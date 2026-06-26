# Derived Data: Batch & Stream Processing

## 1. Three System Types

| Type | Definition | Performance Metric | Example |
|---|---|---|---|
| **Services (online)** | Waits for requests, handles immediately | Response time | REST API, database query |
| **Batch processing (offline)** | Takes bounded input, runs job, produces output | Throughput | MapReduce, ETL pipeline |
| **Stream processing (near-real-time)** | Processes events shortly after they arrive | Latency | Kafka Streams, Flink job |

- **Bounded input**: Finite, known-size dataset (batch)
- **Unbounded input**: Never-ending event stream (stream)

---

## 2. Unix Philosophy & Batch Foundations

**4 Rules:**
1. Each program does one thing well — build new programs, don't add features
2. Output of every program should be input to another — no binary formats, no extraneous output
3. Design to be tried early — don't hesitate to rebuild
4. Use tools over unskilled help — even throwaway tools count

**Composability properties:**

| Property | Mechanism | Benefit |
|---|---|---|
| **Uniform interface** | Files (bytes) as common medium | Any tool can connect to any other |
| **Logic/wiring separation** | Programs use stdin/stdout; shell wires them | Loose coupling, inversion of control |
| **Transparency** | Inputs immutable; inspect at any stage | Debuggable, restartable from any point |

**Immutability principle:** Inputs never mutated → automatic retry safe → human fault tolerance enabled.

---

## 3. MapReduce

**HDFS basics:**
- **Shared-nothing** principle (vs NAS/SAN)
- NameNode tracks block locations; blocks replicated across machines (or erasure-coded)
- Object stores (S3, Azure Blob) similar but separate storage from compute

**Execution model (4 steps):**
1. Read input files → break into records
2. **Map**: extract key-value pair from each record
3. **Shuffle/Sort**: partition by hash(key), sort — the most critical step
4. **Reduce**: iterate over sorted pairs with same key, emit output

**"Putting computation near data":** Scheduler tries to run each mapper on a machine that stores a replica of the input block.

---

## 4. Join Algorithms in Batch

| Join Type | Condition Required | Mechanism | When to Use |
|---|---|---|---|
| **Sort-merge (reduce-side)** | No assumptions | Both inputs keyed on join field; framework sorts; reducer joins | General case, any input |
| **Broadcast hash join** | One input fits in memory | Each mapper loads small input as hash table; scans large input | Small dimension table |
| **Partitioned hash join** | Both inputs partitioned identically | Each mapper loads one partition of small input only | Medium dimension, same partition key |
| **Map-side merge join** | Both partitioned AND sorted on join key | Mapper streams both partitions in sorted order | Pre-sorted inputs available |
| **Skew join (two-stage)** | Hot keys / data skew detected | Send hot-key records to random reducers; replicate other input | Celebrity users, viral products |

**Secondary sort:** Arrange reducer input so dimension records arrive before event records (e.g., user profile before all user events).

---

## 5. Handling Skew

**Skew join (Pig/Hive):**
- Hot-key records distributed to random reducers
- Opposite input replicated to ALL reducers handling that hot key
- Cost: replication of the "other" side

**Two-stage grouping for hot keys:**
1. First MapReduce: send skewed records to random reducers → partial aggregation
2. Second MapReduce: combine partial results

**Linchpin objects:** Database records disproportionately referenced (hot keys). Skew breaks the "work distributed evenly" assumption of vanilla MapReduce.

---

## 6. Batch Output Patterns

| Pattern | Mechanism | Key Property |
|---|---|---|
| **Search indexes** | Mappers partition docs; reducers build per-partition Lucene indexes | Immutable, bulk-loaded, read-only at serve time |
| **KV store bulk load** | Build DB files in batch job; atomic switch in serving layer | Zero-downtime swap (Voldemort, HBase bulk load) |
| **Analytics warehouse** | Write Parquet/ORC to data lake | Schema-on-read, multiple consumers |

**Philosophy of batch outputs:**
- Inputs **immutable** → no side effects during job
- **Human fault tolerance**: roll back code, re-run job → output correct again
- **Minimizing irreversibility**: easy rollback → faster feature development
- Same input files can feed multiple independent jobs

**Sushi principle:** "Raw data is better" — dump first, model later. Shifts interpretation to consumers (schema-on-read). Enables data lake / enterprise data hub pattern.

---

## 7. MapReduce vs MPP Databases

| Dimension | MapReduce / Hadoop | MPP Database |
|---|---|---|
| Storage | Files, any format | Structured, proprietary format |
| Processing | Arbitrary code (JVM, Python, etc.) | SQL queries |
| Schema | Schema-on-read | Schema-on-write |
| Fault tolerance | Task-level retry (writes to HDFS) | Query-level abort and restart (in memory) |
| Data model | Diversity: images, logs, text, binary | Relational / columnar |
| Use case fit | Large jobs, ML, mixed workloads | Analytic SQL, interactive queries |

**Anti-pattern:** Using MPP for raw, heterogeneous data — forces premature schema commitment.

---

## 8. Dataflow Engines

**MapReduce weakness — materialization of intermediate state:**
- Must wait for ALL preceding tasks to finish before next stage starts
- Intermediate state written to HDFS (replicated, slow, expensive)
- Mapper stages often redundant (read what reducer just wrote)

**Dataflow engines (Spark, Tez, Flink) advantages over MapReduce:**
- Entire workflow is ONE job (not chained independent jobs)
- Functions called **operators** (generalization of map/reduce)
- Sort only when required by the operator
- No unnecessary mapper stages
- Co-locate operator with data producer (locality)
- Intermediate state in memory/local disk (not HDFS)
- Operators start as soon as input is ready (pipelined)
- JVM reuse across operators

**Fault tolerance comparison:**

| Engine | Fault Tolerance Mechanism | Requirement |
|---|---|---|
| MapReduce | Writes all intermediate state to HDFS | None — restart from disk |
| Spark | Recompute from RDD lineage | Operators must be **deterministic** |
| Flink | Periodic checkpoints to durable storage | Operators must be **deterministic** |

**Deterministic operators:** Non-determinism (random numbers, clock reads) breaks recomputation-based fault tolerance — the recomputed result differs from the lost result.

---

## 9. Graph Processing

**Pregel / BSP (Bulk Synchronous Parallel) model:**
- Each vertex sends messages along edges
- Processed in synchronous rounds (supersteps)
- Each vertex maintains state across iterations (unlike stateless MapReduce)
- Fault tolerance via periodic checkpointing of ALL vertex state

**Rule of thumb:** If the graph fits on a single machine, single-machine algorithms (GraphChi for disk-based) typically outperform distributed Pregel — communication overhead dominates.

---

## 10. Event Streams & Messaging

**Event:** Small, self-contained, immutable object with timestamp. Produced by a **producer**; consumed by **consumers**.

**Two key questions for any messaging system:**

| Question | Options |
|---|---|
| Producer faster than consumer? | Drop messages / buffer in queue / **backpressure** (block producer) |
| Node crashes? | Write to disk and/or replicate (costs throughput/latency) |

**Consumer patterns:**

| Pattern | How | Use When |
|---|---|---|
| **Load balancing** | Each message → ONE consumer (AMQP queue / JMS shared sub) | Processing is expensive, parallelizable |
| **Fan-out** | Each message → ALL consumers (topic subscription) | Multiple independent systems need same events |
| **Combined** | Consumer groups: each group gets all messages; within group, load-balanced | Common Kafka pattern |

**Backpressure:** Producer is blocked when consumer queue is full. Prevents unbounded memory growth.

---

## 11. Message Brokers vs Databases

| Dimension | Message Broker | Database |
|---|---|---|
| Retention | Delete after delivery/ack | Keep until explicitly deleted |
| Working set | Small (short queues) | Arbitrary size |
| Querying | Subscribe to topic or pattern | Secondary indexes, arbitrary queries |
| Data view | Notify on new data (push) | Point-in-time snapshot (pull) |

---

## 12. Log-based Brokers

**Kafka / Kinesis / DistributedLog model:**
- **Partitioned log**: append-only, each partition totally ordered by offset
- **Consumer offsets**: consumer tracks its own position (like a replication log LSN)
- Supports fan-out (independent consumers at different offsets) AND replay
- Disk as **circular buffer**: old segments deleted or archived when space needed
- **Ordering guarantee**: within a partition only. Max parallelism = number of partitions.

**Decision: AMQP/JMS-style vs Log-based broker:**

| Use When | Broker Type |
|---|---|
| Message order NOT important; expensive per-message work; need per-message parallelism | AMQP / JMS |
| Message order important; fast per-message processing; high throughput; need replay | Log-based (Kafka) |

---

## 13. Change Data Capture (CDC)

**Concept:** Observe all writes to a database, extract as a stream → replicate to derived systems.

- Makes one database the **leader**; all derived systems are **followers**
- Avoids **dual writes problem**: race conditions when writing to DB + search index independently cause permanent inconsistency
- Implementation options: database triggers (fragile), parse replication log (robust)

**Tools:**

| Tool | Source DB |
|---|---|
| Debezium, Maxwell | MySQL |
| Bottled Water | PostgreSQL |
| Mongoriver | MongoDB |
| LinkedIn Databus, Facebook Wormhole | Generic / internal |

**Initial snapshot:** When adding a new derived system, take a DB snapshot + apply subsequent CDC stream from that snapshot's log position.

**Log compaction:** Keep only latest value per key → new consumer can rebuild full state from offset 0 without needing historical snapshots.

---

## 14. Event Sourcing & CQRS

**CDC vs Event Sourcing:**

| | CDC | Event Sourcing |
|---|---|---|
| Level | Low-level (row changes) | High-level (user actions / domain events) |
| Storage | Mutable DB; replication log extracted | Append-only event store; events are primary source |
| Log compaction | Works (latest value per key) | Does NOT work (need full history, events don't override) |
| Community | Database / infrastructure | DDD / application architecture |

**Commands vs Events:**
- **Command**: a request that may be rejected after validation (intent)
- **Event**: an accepted, immutable fact (durable, happened)

**State-Stream duality:**
- `state(now) = ∫ stream(t) dt` — state is the integral of the event stream
- `stream(t) = d state(t) / dt` — stream is the derivative of state changes

> "The truth is the log. The database is a cache of the latest record values." — Pat Helland

**CQRS (Command Query Responsibility Segregation):**
- Separate write-optimized event log from read-optimized views
- Derive multiple read views from the same event log
- Normalization debates become irrelevant — translate between write-optimized and read-optimized forms

**Advantages of immutable events:**
- Auditability (like accounting ledgers — append corrections, never erase)
- Captures MORE info than current state (cart add + remove reveals customer intent)
- Human fault tolerance: replay from immutable log to recover from buggy code

**Excision:** True deletion from an immutable log (required for privacy regulations). Datomic's term. Hard in practice — data exists in storage engines, filesystems, backups, and caches.

---

## 15. Stream Processing Uses

| Use Case | What It Does | Tools |
|---|---|---|
| **Complex Event Processing (CEP)** | Search for event patterns in stream (like regex over events); queries stored long-term, data flows past | Esper, IBM InfoSphere Streams |
| **Stream analytics** | Aggregations and metrics over time windows; probabilistic algorithms for cardinality | Flink, Kafka Streams, Dataflow |
| **Materialized views** | Keep derived data up-to-date via stream; must consider ALL events, not just a window | Kafka Streams, Samza |
| **Search on streams** | Invert the model — index queries, stream documents past them | Percolator (Elasticsearch) |

---

## 16. Time in Streams

**Event time vs processing time — critical distinction:**

| | Event Time | Processing Time |
|---|---|---|
| Definition | When the event actually occurred | When the stream processor processes it |
| Source | Embedded in event payload | Stream processor's local clock |
| Problem | Device clocks drift / are untrusted | Shows fake spikes during restart catchup |

**Straggler events** (late arrivals after window declared complete):

| Option | How | Cost |
|---|---|---|
| Ignore | Drop straggler; track dropped count as metric | Slight inaccuracy |
| Publish correction | Retract previous output; issue updated value | Consumers must handle retractions |

**Watermark:** Signal from the processor indicating "no more messages with timestamp < t." Not a hard guarantee when timestamps come from untrusted client clocks.

**Three-timestamp approach (untrusted device clocks):**
1. Time event occurred (device clock — potentially skewed)
2. Time event was sent (device clock)
3. Time event was received (server clock)

→ Estimate clock offset from (2) and (3); infer true event time from (1) corrected by offset.

---

## 17. Window Types

| Window | Description | Overlap | Use Case |
|---|---|---|---|
| **Tumbling** | Fixed length, non-overlapping; each event in exactly one window | None | Count per fixed interval (e.g., requests per minute) |
| **Hopping** | Fixed length, overlapping; hop size < window size | Partial | Smoothed metrics (5-min window, 1-min hop) |
| **Sliding** | All events within a fixed interval of each other | Continuous | Moving aggregation; every event shifts the window |
| **Session** | No fixed duration; groups events by user activity; ends after inactivity gap | None (per user) | Website analytics, user session tracking |

---

## 18. Stream Joins

| Join Type | What It Joins | State Maintained | Notes |
|---|---|---|---|
| **Stream-stream (window join)** | Two event streams within a time window | All events in the window, indexed by join key | Non-deterministic ordering → joins can vary unless interleaving is logged |
| **Stream-table (enrichment)** | Event stream + database table | Local copy of the DB table (kept current via CDC) | Table updates propagate to join |
| **Table-table (materialized view)** | Two database changelogs | Full materialized state of both tables | Output = new state of the join whenever either table changes |

**Time-dependence of joins:** Ordering across streams is undetermined → stream joins are inherently non-deterministic unless the interleaving of events is explicitly logged to enable replay.

---

## 19. Fault Tolerance in Streams

**Exactly-once semantics:** Visible output behaves as if each record was processed exactly once, even though internally retries may reprocess records.

| Approach | Mechanism | Used By | Trade-off |
|---|---|---|---|
| **Microbatching** | Break stream into small fixed-time batches; batch semantics apply | Spark Streaming | Minimum latency = batch interval size |
| **Checkpointing** | Periodically snapshot all operator state to durable storage | Apache Flink | Reprocesses records since last checkpoint on failure |
| **Atomic commit** | Output + state change + offset advance happen atomically or not at all | Google Cloud Dataflow | Higher overhead; strongest guarantee |
| **Idempotence** | Include message offset with writes; detect and skip duplicates on retry | Storm Trident, general pattern | Requires idempotent operations or dedup logic |

**Idempotent vs non-idempotent:**

| Operation | Idempotent? | Why |
|---|---|---|
| Set key to fixed value | Yes | Repeating has same effect |
| Increment counter | No | Retry doubles the count |
| Write with dedup via offset | Yes (engineered) | Include offset → detect duplicate on retry |

**Rebuilding state after failure:**
- Remote datastore: safe but slow (per-message round-trips)
- Local state + periodic replication to durable log (Flink → HDFS; Samza/Kafka Streams → compacted Kafka topic)
- Rebuild from input: viable only when state is a short-window aggregation or maintained via CDC
