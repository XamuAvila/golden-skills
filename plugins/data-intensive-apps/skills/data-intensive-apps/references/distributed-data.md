# Distributed Data: Replication & Partitioning

## Replication Models

**Replication**: Keeping a copy of the same data on multiple machines. Goals: reduce latency (geo-proximity), increase availability (fault tolerance), scale read throughput.

**Replica**: Any node storing a copy of the database.
**Leader (master/primary)**: The designated replica that accepts writes.
**Follower (read replica/slave/hot standby)**: Receives a replication log from the leader and serves read-only queries.
**Replication log (change stream)**: Data change feed sent from leader to followers for replay.

### Three Replication Approaches

| Model | Writes accepted by | Reads served by | Conflict handling | Used by |
|---|---|---|---|---|
| **Single-leader** | Leader only | Any replica | N/A — single write path | PostgreSQL, MySQL, Oracle Data Guard, SQL Server, MongoDB, RethinkDB, Kafka, RabbitMQ |
| **Multi-leader** | Any leader | Any replica | Required — concurrent writes possible | CouchDB, multi-DC setups, offline-capable apps |
| **Leaderless (Dynamo-style)** | Any replica (quorum) | Any replica (quorum) | Required — concurrent writes possible | Riak, Cassandra, Voldemort |

### Decision: Which Replication Model?

| Need | Recommendation |
|---|---|
| Simple setup, most workloads | Single-leader |
| Multi-datacenter, low write latency per DC | Multi-leader (accept conflict complexity) |
| Offline-first / mobile sync | Multi-leader (each device = leader) |
| High availability, tolerate partitions, eventual consistency OK | Leaderless |
| Strong consistency required | Single-leader + synchronous replication (or transactions) |

### Synchrony Modes

| Mode | Guarantee | Risk | Typical use |
|---|---|---|---|
| **Synchronous** | Follower confirmed before write ack to client | Blocks writes if follower is unavailable | One follower only (semi-sync) |
| **Asynchronous** | Leader proceeds without waiting for follower | Data lost if leader crashes before replication | All-but-one followers |
| **Semi-synchronous** | One follower sync, rest async | Balanced | Default in many deployments |

> **Replication lag**: Delay between leader write and follower visibility. Sub-second normally; can reach minutes under load or network issues. **Eventual consistency**: followers catch up *eventually* — no upper bound on lag.

---

## Replication Log Methods

How the leader communicates changes to followers matters significantly for compatibility and correctness.

| Method | Mechanism | Pros | Cons |
|---|---|---|---|
| **Statement-based** | Forward SQL verbatim (INSERT, UPDATE, DELETE) | Compact; readable | Breaks on NOW(), RAND(), auto-increment, stored procedures with side effects |
| **WAL shipping** | Ship write-ahead log (byte-level storage changes) | Exact replica; no interpretation needed | Coupled to storage engine version — can't run different DB versions across replicas |
| **Logical log (row-based)** | Row-granularity records: inserts→all columns; deletes→PK; updates→changed columns | Decoupled from storage engine; supports different versions; CDC-friendly | Larger payloads for wide tables |
| **Trigger-based** | Application-layer via triggers/stored procedures writing to a replication table | Most flexible: subset, transformation, cross-database | Highest overhead; most error-prone; must be maintained manually |

> **Default choice**: Logical/row-based. Best balance of flexibility, compatibility, and correctness.
> **CDC (Change Data Capture)**: Logical logs are the foundation — external consumers (Kafka, Debezium) parse them to build derived data systems.

---

## Replication Lag & Consistency

Three specific consistency guarantees address common lag-related anomalies:

| Guarantee | Problem it solves | Symptom without it | Implementation |
|---|---|---|---|
| **Read-after-write (read-your-writes)** | User sees stale data after their own write | Profile update disappears on reload | Read own data from leader; or track write timestamp and block read until replica catches up |
| **Monotonic reads** | User reads newer state then older state (time reversal) | Comment posted, then disappears, then reappears | Route each user to the same replica (e.g., `hash(user_id) mod replica_count`) |
| **Consistent prefix reads** | Observer sees causally dependent writes out of order | Answer appears before question | Write causally related records to same partition; or use client-side version tracking |

> These are **not** strong consistency. They address specific anomalies within eventual consistency.
> **Transaction isolation** is the solution when these per-guarantee patches are insufficient.

---

## Failover

Failover = promoting a follower to leader when the current leader fails.

### Steps

1. **Detect failure**: Monitor leader heartbeat; trigger on timeout
2. **Choose new leader**: Election among followers, or decision by controller node; prefer replica with most up-to-date replication log
3. **Reconfigure system**: Redirect client writes to new leader; demote old leader to follower on recovery

### Timeout Trade-off

| Timeout setting | Effect |
|---|---|
| Too short | False positives under transient load spikes → unnecessary failover → extra disruption |
| Too long | Extended unavailability during real failures |

### Pitfalls Checklist

- [ ] **Data loss from async lag**: New leader may not have all writes the old leader acknowledged — conflicting with application expectations
- [ ] **External system divergence**: Databases coupled to external state (e.g., Redis counters, MySQL auto-increment IDs) can diverge — GitHub incident: MySQL primary key reuse invalidated Redis cache entries
- [ ] **Split brain**: Two nodes both believe they are leader → concurrent conflicting writes → data corruption
  - **Mitigation**: Fencing (STONITH) — proactively shut down suspected stale leader before promoting new one
- [ ] **Cascading failures**: Failover under load can increase load on remaining nodes, triggering further failures
- [ ] **In-flight writes**: New leader may receive writes originally sent to old leader — must detect and discard to prevent double-apply

---

## Conflict Resolution

Applies when **multi-leader** or **leaderless** systems receive concurrent writes to the same record.

| Strategy | Mechanism | Data loss? | Complexity | Recommendation |
|---|---|---|---|---|
| **Conflict avoidance** | Route all writes for a record through the same leader | None | Low | **First choice** — eliminates the problem |
| **Last Write Wins (LWW)** | Timestamp determines winner; concurrent write silently discarded | **Yes** | Low | Avoid unless loss is acceptable (e.g., caches) |
| **Replica priority** | Higher-numbered replica always wins | **Yes** | Low | Rarely appropriate |
| **Merge values** | Concatenate, union, or set-merge concurrent values | None | Medium | Works for commutative operations |
| **Record and resolve** | Preserve all conflicting versions (siblings); surface to application or user | None | High | Correct but shifts burden to application |
| **CRDTs** | Conflict-free replicated data types (sets, counters, registers) that mathematically auto-merge | None | Medium | Requires data structure redesign |
| **Operational transformation** | Preserve intent of concurrent operations (sequential correctness) | None | Very high | Google Docs model; specialized |

> **Anti-pattern — LWW**: Silent data loss. Any system that cannot tolerate data loss must not use LWW as a conflict resolution strategy.
> **Multi-leader danger**: Even "safe" write paths can produce conflicts when a network partition heals. Design for conflict resolution from the start.

---

## Quorums

Used in **leaderless** replication to determine how many replicas must acknowledge a read or write.

**Core formula**: `w + r > n`
- `n` = total replicas
- `w` = write acknowledgments required
- `r` = replicas queried per read
- Overlap guarantees at least one read node has the latest write

### Common Configurations

| Config | Tolerates | Trade-off |
|---|---|---|
| `n=3, w=2, r=2` | 1 unavailable node | Standard balanced setup |
| `n=5, w=3, r=3` | 2 unavailable nodes | Higher redundancy |
| `w=n, r=1` | Any write node failure | Fast reads; any write failure blocks all writes |
| `w=1, r=n` | Any read node failure | Fast writes; reads expensive; high durability |
| `w=1, r=1` | Nothing — no overlap guarantee | Maximum throughput, no consistency |

### Staleness Management Mechanisms

| Mechanism | What it does | Consistency impact |
|---|---|---|
| **Sloppy quorum** | Accept writes on *any* reachable nodes when home nodes unavailable | Durability without consistency — w+r>n overlap no longer guaranteed |
| **Hinted handoff** | After sloppy quorum, forward "hinted" writes to home nodes once they recover | Eventual consistency restored |
| **Read repair** | Client detects stale value during quorum read; writes newer value back to stale replica | Lazily corrects divergence; misses items not read |
| **Anti-entropy process** | Background daemon scans replicas for differences; copies missing data | No ordering guarantee; covers items read repair misses |

> **Quorum edge cases**: Even with w+r>n, stale reads can still occur during concurrent writes, sloppy quorums, or if the implementation uses LWW with clock skew.

---

## Concurrent Writes

### Core Concepts

| Concept | Definition |
|---|---|
| **Happens-before** | A happens before B if B knows about A or causally depends on A |
| **Concurrent** | Neither A happens before B nor B before A — both unaware of each other at write time |
| **Version vectors** | Collection of `(replica_id, version_number)` pairs per key; captures causal history across all replicas |
| **Siblings** | Multiple concurrent values for the same key that must be merged — the database stores all of them |
| **Tombstone** | Deletion marker — required instead of simple removal when using CRDTs or merging siblings (simple delete would be overwritten by concurrent add) |

### Version Vector Algorithm

1. Server increments its own counter per key on each write
2. Returns `{value, version_vector}` to client on read
3. Client writes `{new_value, version_vector_from_read}` to server
4. Server determines: if incoming vector descends from stored → overwrite; if concurrent → keep both as siblings
5. Client must merge siblings before next write

> **Version vectors vs vector clocks**: Version vectors are per-key structures tracking per-replica versions. Vector clocks are similar but applied at operation level. The terms are often conflated.

### Merging Siblings

| Data type | Merge strategy |
|---|---|
| Counter | Sum values |
| Set (add only) | Union |
| Set with deletes | Union of adds; union of tombstones; subtract tombstoned items |
| String / arbitrary | Application-defined; last-writer-wins as fallback (with data loss) |

---

## Partitioning Strategies

**Partitioning (sharding)**: Splitting a dataset into subsets assigned to different nodes. Goal: horizontal scalability — spread data and query load.

**Terminology**: partition = shard (MongoDB/Elasticsearch) = region (HBase) = tablet (BigTable) = vnode (Cassandra/Riak) = vBucket (Couchbase)

**Skewed partitioning**: Uneven distribution — some partitions get disproportionate traffic.
**Hot spot**: A single partition with overload. Can occur even with hash partitioning when millions of writes target the same key (celebrity pattern).

### Strategies Comparison

| Strategy | Mechanism | Advantages | Disadvantages | Used by |
|---|---|---|---|---|
| **Key range** | Each partition owns a contiguous sorted key range | Efficient range queries; adjacent keys co-located | Hot spots when access clusters on current range (e.g., today's timestamp prefix) | HBase, BigTable, RethinkDB, MongoDB <2.4 |
| **Hash** | Hash(key) → assign key to partition owning that hash range | Even distribution; reduces skew | Destroys sort order; range queries scatter to all partitions | Cassandra, MongoDB (hash mode), Riak, Couchbase, Voldemort |
| **Compound key** | First key part hashed → selects partition; remaining parts sort within partition | Partitions by dimension AND sorts within partition | Identical first-parts still hot-spot | Cassandra |

### Decision: Which Partitioning Strategy?

| Access pattern | Recommendation |
|---|---|
| Range queries critical (time-series, sorted scans) | Key range — mitigate hot spots with multi-dimension prefix (`sensor_id + timestamp`) |
| Uniform distribution, no range queries | Hash partitioning |
| Partition by user, sort by recency within user | Compound key: `(user_id, timestamp)` |
| Celebrity / hot-key problem | Application-level: append random 2-digit suffix to hot key → distribute across 100 partitions; read fan-out of 100 on reads |

### Key Range vs Hash

| Dimension | Key range | Hash |
|---|---|---|
| Distribution | Hot-spot risk if access pattern clusters | Even by design |
| Range queries | Efficient (small partition set) | Scatter/gather to all partitions |
| Rebalancing | Dynamic split/merge | Fixed partition count, whole-partition reassignment |
| Hot key mitigation | Application key design | Vulnerable to identical hot keys |

### Anti-pattern: `hash(key) mod N`

`hash(key) mod N` (N = number of nodes) fails when N changes: nearly every key must move to a new node. Use **fixed partition count** (reassign whole partitions) or **dynamic splitting** instead.

---

## Secondary Index Partitioning

Secondary indexes don't map neatly to partition boundaries, requiring one of two approaches:

| Approach | Also known as | Write cost | Read cost | Consistency | Used by |
|---|---|---|---|---|---|
| **Document-partitioned** | Local index | Single partition update (index co-located with document) | **Scatter/gather** — query all partitions, merge results | Immediate within partition | MongoDB, Riak, Cassandra, Elasticsearch, VoltDB |
| **Term-partitioned** | Global index | Multiple partitions (each index term may live on different node) | **Single partition** — only the node holding the term | Often asynchronously updated | DynamoDB (some modes) |

> **Scatter/gather tail latency**: Document-partitioned reads fan out to all N partitions. The overall response time = slowest partition. High N amplifies tail latency.
> **Term-partitioned write overhead**: A single document write can require index updates on several nodes → higher write latency and partial failure risk.

---

## Rebalancing

**Rebalancing**: Moving partitions between nodes when topology changes (nodes added/removed) or data grows.
Requirements: fair load distribution after rebalancing, no downtime during, minimize data movement.

### Strategies

| Strategy | Mechanism | Pros | Cons |
|---|---|---|---|
| **Fixed partition count** | Create many more partitions than nodes at setup; reassign whole partitions when nodes change | Simple; only partition→node mapping updates; no intra-partition reshuffling | Must choose count upfront; too few = large partitions; too many = per-partition overhead |
| **Dynamic partitioning** | Split when partition exceeds size threshold; merge when too small | Adapts to data volume; partition count proportional to data size | Empty DB starts as single partition → single-node bottleneck; mitigate with **pre-splitting** |
| **Proportional to nodes** | Fixed partitions per node; new node steals random splits from existing partitions | Partition size proportional to dataset as cluster scales | Split partitions can be uneven in size |

### Automatic vs Manual

| Mode | Benefit | Risk |
|---|---|---|
| **Fully automatic** | Less operational work | Overloaded node → detected as dead → rebalancing → more load → cascade failure |
| **Human in the loop** | Prevents cascading surprises | Slower reaction time to topology changes |

> **Recommendation**: Generate rebalancing plans automatically; require administrator approval before executing. Slower, but prevents runaway cascade failures.
> **Pre-splitting**: Configure initial partition boundaries on an empty database to avoid starting with a single-partition bottleneck.

---

## Request Routing

After partitioning, clients must locate the correct node for a given key.

### Three Approaches

| Approach | How it works | Pros | Cons |
|---|---|---|---|
| **Contact any node** | Round-robin LB; receiving node forwards request if it doesn't own the key | Simple client; no topology knowledge needed | Extra hop on every miss |
| **Routing tier** | Partition-aware proxy (e.g., mongos) routes to correct node directly | Clients fully decoupled from topology | Routing tier is a new operational component |
| **Client-aware** | Client maintains partition→node map; connects directly | Zero extra hops | Clients must track topology changes |

### Coordination Mechanisms

| Mechanism | Used by | Approach | Dependency |
|---|---|---|---|
| **ZooKeeper** | HBase, Kafka, SolrCloud | Nodes register; routing tiers/clients subscribe to changes | External ZooKeeper cluster required |
| **Gossip protocol** | Cassandra, Riak | Peer-to-peer state dissemination; each node knows full cluster map | None — self-contained |
| **Config server + mongos** | MongoDB | mongos daemons act as routing tier; config servers are authoritative | MongoDB config servers |

> **ZooKeeper**: Strongly consistent partition map; adds external operational dependency and single-point-of-care.
> **Gossip**: No external dependency; eventually consistent — partition map may lag briefly after topology change.
> **MPP (Massively Parallel Processing)**: Partition-aware query execution used in analytics databases (Teradata, etc.) — queries execute in parallel across partitions and results merged.

---

## Quick Reference: System Characteristics

| System | Replication model | Partitioning | Secondary index | Routing |
|---|---|---|---|---|
| PostgreSQL / MySQL | Single-leader | N/A (single node or app-level) | N/A | N/A |
| MongoDB | Single-leader | Hash or key range | Document-partitioned (local) | mongos + config server |
| Cassandra | Leaderless | Hash + compound key | Term-partitioned (global) | Gossip |
| HBase | Single-leader | Key range (dynamic split) | N/A (manual) | ZooKeeper |
| Riak | Leaderless | Hash | Document-partitioned (local) | Gossip |
| Kafka | Single-leader per partition | Key-based (hash) | N/A | ZooKeeper / KRaft |
| Elasticsearch | Single-leader per shard | Hash | Document-partitioned (local) | Routing tier |

### Danger Zones Summary

| Situation | Risk | Mitigation |
|---|---|---|
| Async single-leader failover | Write data loss on new leader promotion | Accept loss OR use sync replication for at least one follower |
| Multi-leader without conflict strategy | Silent data corruption | Design conflict resolution before deploying multi-leader |
| LWW as conflict resolution | Silent data loss on concurrent writes | Use CRDTs or record-and-resolve for loss-intolerant data |
| `hash(key) mod N` partitioning | Mass key migration on any topology change | Use fixed partition count or dynamic splitting |
| Fully automatic rebalancing | Cascade failure under load | Always require human approval before rebalancing executes |
| Scatter/gather on all partitions | Tail latency amplification | Consider term-partitioned (global) index or denormalization |
