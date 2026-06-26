# Transactions, Consistency & Consensus

## Transactions & ACID

| Letter | Name | Precise meaning | Common misconception |
|---|---|---|---|
| **A** | Atomicity | Abortability — if fault occurs mid-tx, all writes are discarded | NOT about concurrency |
| **C** | Consistency | Application-level invariants remain satisfied | "The C doesn't really belong in ACID" — it's the **app's job**, not the DB's |
| **I** | Isolation | Concurrently executing transactions don't interfere; formally: serializability | Often weakened in practice |
| **D** | Durability | Committed data survives hardware/software failures; implemented via WAL or replication | Perfect durability doesn't exist |

- **BASE** (Basically Available, Soft state, Eventual consistency): "The only sensible definition of BASE is 'not ACID'." — deliberately vague marketing
- **Multi-object transactions**: required for foreign keys, denormalized data, secondary index consistency. Many NoSQL stores abandoned these.
- **Single-object atomic ops** (`atomic increment`, `compare-and-set`): NOT "transactions" in the full sense — only cover one object at a time

---

## Isolation Levels

| Level | Prevents | Does NOT prevent | Implementation |
|---|---|---|---|
| **Read committed** (PostgreSQL/Oracle/SQL Server default) | Dirty reads, dirty writes | Lost updates, read skew | Write locks until commit; reads see old committed value |
| **Snapshot isolation / MVCC** | Read skew (non-repeatable reads) | Write skew, phantoms; lost updates in some DBs | Each row: `created_by` + `deleted_by` tx IDs; update = delete + create |
| **Serializable** | All race conditions | — | See Serializability section |

**MVCC visibility rule**: ignore writes from (a) in-progress transactions, (b) aborted transactions, (c) transactions that started after this one.

**Key principle**: _readers never block writers, writers never block readers_

**Naming confusion**: Oracle calls snapshot isolation "serializable"; PostgreSQL/MySQL call it "repeatable read". Neither is wrong per their own specs — the SQL standard is ambiguous.

---

## Race Conditions

| Race Condition | Definition | Minimum prevention |
|---|---|---|
| **Dirty read** | Read uncommitted data from another tx | Read committed |
| **Dirty write** | Overwrite uncommitted data from another tx | Read committed (write locks) |
| **Read skew** | See inconsistent snapshot across two reads of different objects | Snapshot isolation (MVCC) |
| **Lost update** | Read-modify-write cycle clobbered by a concurrent write | Atomic ops, explicit locking, auto-detection, or compare-and-set |
| **Write skew** | Two txs read overlapping data, update *different* objects, violating a constraint | Serializable isolation or `SELECT FOR UPDATE` on the read set |
| **Phantom** | A write in one tx changes the result set of a search query in another tx | Serializable isolation; index-range locks |

> **Write skew anatomy**: `SELECT` checks condition → app decides → `UPDATE`/`INSERT` based on decision → but the checked condition changed concurrently between read and write.

---

## Preventing Lost Updates

Ranked from best to worst when applicable:

| Rank | Approach | Mechanism | Caveat |
|---|---|---|---|
| 1 | **Atomic write operations** | `UPDATE counter SET val = val + 1` | Only works for commutative ops |
| 2 | **Automatic detection** | DB detects lost update under snapshot isolation, aborts loser | PostgreSQL, Oracle, SQL Server support this; **MySQL/InnoDB does NOT** |
| 3 | **Explicit locking** | `SELECT ... FOR UPDATE` | App must remember to lock every relevant read |
| 4 | **Compare-and-set** | `UPDATE ... WHERE val = old_val` | **Unsafe under snapshot isolation** — reads old snapshot, not current |

**Replication caveat**: In multi-leader or leaderless replication, commutative operations (increment, add-to-set) handle conflicts; LWW (last-write-wins) silently drops updates.

---

## Write Skew

**Pattern** (all 6 examples follow this shape):

```
1. SELECT — read shared data to check a condition
2. Decide — application logic makes a choice based on that condition
3. UPDATE/INSERT — modify a *different* object than the one read
4. Race — the condition changed while you were deciding
```

| Example | Read | Write | Constraint |
|---|---|---|---|
| Doctor on-call | Count of on-call doctors | Own on-call status | At least 1 doctor always on call |
| Meeting room booking | Existing bookings for room/time | New booking | No double-booking |
| Username claiming | Existence of username | New user row | Uniqueness |
| Double-spending | Account balance | New transaction | Balance ≥ 0 |
| Multiplayer game | Player positions | Own position | No two players occupy same square |

**Prevention**: Only serializable isolation fully prevents write skew. `SELECT FOR UPDATE` helps when you can identify the rows to lock upfront; doesn't help when the "phantom" set is empty (nothing to lock).

---

## Serializability

| Approach | Mechanism | Pros | Cons |
|---|---|---|---|
| **Actual serial execution** | Single-threaded event loop, stored procedures, in-memory dataset | Zero concurrency bugs, simple | Single CPU core limit; txs must be short; dataset must fit in RAM |
| **Two-phase locking (2PL)** | Shared locks (reads), exclusive locks (writes); readers block writers AND writers block readers | True serializability; prevents everything | Poor throughput, high tail latency, deadlocks |
| **Serializable Snapshot Isolation (SSI)** | Optimistic: run concurrently under MVCC, detect serialization conflicts at commit, abort offenders | High throughput, readers don't block writers | More aborts under contention; txs should be short-lived |

### 2PL Details
- **Predicate locks**: lock based on search condition, not just rows — expensive, so approximated by **index-range locks** (next-key locking)
- **Deadlock detection**: abort one transaction, let the other proceed; caller must retry

### SSI Detection
1. Detect stale MVCC reads: an uncommitted write existed before the read — if that write commits, abort
2. Detect writes affecting prior reads: tracked via index-range bookkeeping per transaction

---

## Transaction Retry Pitfalls

- **Duplicate on lost ack**: server committed, network dropped the confirmation → retry without idempotency key = double effect
- **Overload amplification**: error was caused by load → retry immediately makes it worse; use exponential backoff + jitter
- **Permanent errors**: constraint violation will fail every time → don't retry
- **Out-of-transaction side effects**: sending an email, charging a card — these happen even if the DB tx aborts; not rolled back by retry
- **Client crash mid-retry**: in-flight data lost entirely; client must re-derive state on restart

---

## Unreliable Networks

- Data networks are **asynchronous packet networks**: no bounded delivery time, no delivery guarantee
- When no response arrives, it is **impossible** to distinguish: request lost / remote node down / response lost
- **Only practical failure detector**: timeouts — but no "correct" value

| Timeout trade-off | Short | Long |
|---|---|---|
| Detection speed | Fast | Slow |
| False positive rate | High (slow node declared dead) | Low |
| Cascading risk | High (unnecessary failover under load) | Low |

**Sources of variable delay**: network switch queues → OS receive buffers → VM scheduling (steal time) → TCP retransmission/flow control → "noisy neighbor" in cloud

**Phi Accrual failure detector**: continuously measures response time distribution, dynamically adjusts threshold — better than static timeouts.

**Why not circuit-switched like telephony?**: Telephone networks reserve bandwidth per call → bounded latency. Packet switching uses spare capacity for burst traffic → better utilization, unbounded latency. This is a deliberate engineering trade-off.

---

## Unreliable Clocks

| Type | Behavior | Use for | Do NOT use for |
|---|---|---|---|
| **Time-of-day** | Wall-clock (epoch), NTP-synced, can jump backwards | Timestamps shown to users | Measuring elapsed time, ordering events |
| **Monotonic** | Always forward, not comparable across nodes | Measuring durations, timeouts | Cross-machine ordering |

**Clock skew sources**: quartz drift (~200ppm), NTP accuracy (~35ms best case over internet), leap second smearing, VM clock virtualization, firewalled NTP.

**Danger — LWW with timestamps**: Clock skew means causally-later writes can get earlier timestamps → **silently dropped data**. Never use physical timestamps for event ordering across nodes.

| Ordering mechanism | Properties | Trade-off |
|---|---|---|
| **Lamport timestamps** `(counter, nodeID)` | Total order consistent with causality | Order only known after the fact — can't answer "is this username taken right now?" |
| **Version vectors** | Detect concurrent vs. causal relationships | Per-key state, not global |
| **Google TrueTime** `[earliest, latest]` | Bounded uncertainty interval | Requires GPS/atomic clocks in every datacenter; commits wait for intervals to not overlap |

---

## Process Pauses

**Sources of arbitrary pauses**: GC stop-the-world, VM suspension/live migration, disk I/O wait, memory swapping/paging, OS SIGSTOP, CPU scheduler preemption.

During any pause: leases expire, other nodes take over, the world moves on. A node cannot distinguish "I paused briefly" from "I was dead for 10 minutes."

**Fencing tokens** — the solution:
- Every lock/lease grant includes a **monotonically increasing token** number
- Storage server **rejects writes with a token ≤ current max token seen**
- A node that woke up from a pause will have an old token → its writes are rejected → safe

---

## System Models

### Timing Models

| Model | Assumption | Realistic? |
|---|---|---|
| **Synchronous** | Bounded network delay, bounded pauses, bounded clock drift | No |
| **Partially synchronous** | Usually bounded, occasionally unbounded | Yes — practical default |
| **Asynchronous** | No timing assumptions whatsoever | Extremely restrictive |

### Node Failure Models

| Model | Behavior | When to assume |
|---|---|---|
| **Crash-stop** | Node crashes, never returns | Simple analysis |
| **Crash-recovery** | Crashes, may restart; stable storage intact | Practical default |
| **Byzantine** | Node may send arbitrary/incorrect messages | Aerospace, blockchain |

**Practical default**: Partially synchronous + crash-recovery.

### Safety vs. Liveness

| Property | Meaning | Violation |
|---|---|---|
| **Safety** | "Nothing bad happens" | Permanent; identifiable at a point in time; must hold even during faults |
| **Liveness** | "Something good eventually happens" | Temporary; may have caveats ("if majority alive") |

---

## Linearizability

**Definition**: Make a distributed system appear as if there's only one copy of data, and all operations are atomic. A **recency guarantee**.

**Key rule**: Once any read (on any client, any node) returns the new value, all subsequent reads must also return the new value. No going back.

| Property | Scope | Guarantee |
|---|---|---|
| **Serializability** | Transactions (multi-object) | Behave as if executed in *some* serial order — order can be historical, not current |
| **Linearizability** | Single object/register | Recency — reads always see the latest committed write |
| **Strict serializability** | Both | Serial order + recency. Provided by 2PL and serial execution; **SSI does NOT provide this** |

### When Linearizability is Needed vs. Avoidable

| Need it | Can avoid it |
|---|---|
| Leader election (only one leader) | Compensating transactions acceptable (duplicate username → email apology) |
| Uniqueness constraints (one username, one booking) | Business already handles inconsistency (overselling → order more stock) |
| Cross-channel timing deps (DB replication + message queue racing) | Multi-datacenter performance-sensitive deployment |

### Replication Method → Linearizable?

| Replication | Linearizable? |
|---|---|
| Single-leader | Potentially — only if reads come from leader or sync followers |
| Consensus algorithms (Raft, Paxos) | Yes |
| Multi-leader | No — concurrent writes on different leaders |
| Leaderless Dynamo-style | Probably not — LWW and network delays cause anomalies even with quorums |

---

## CAP Theorem

**Precise formulation**: "Consistent **or** Available when **Partitioned**"

During a network partition, you must choose:
- **Linearizable**: refuse requests on replicas that can't confirm they have latest data → unavailable
- **Available**: serve all requests, but risk returning stale data → not linearizable

**CAP limitations**:
- Only models one fault type (network partition)
- Only models one consistency model (linearizability)
- In practice, most systems drop linearizability for **performance**, not for partition tolerance
- Network partitions are rare in a datacenter; latency and throughput are everyday concerns

---

## Ordering & Causality

- **Causality**: cause must precede effect; a partial order — concurrent events are incomparable (neither happened-before the other)
- **Causal consistency**: strongest model achievable without sacrificing performance to network delays; weaker than linearizability but often sufficient
- **Linearizability implies causality** — total order is stronger than partial causal order

| Mechanism | What it provides | Limitation |
|---|---|---|
| **Lamport timestamps** `(counter, nodeID)` | Total order consistent with causality | Total order emerges only after the fact — real-time "is X taken?" requires consensus |
| **Total order broadcast** | Reliable + totally-ordered delivery to all nodes | Equivalent to consensus |
| **Linearizable storage** | Single-copy semantics, recency | Requires consensus |

**Equivalence chain** (solving any one solves all others):

```
Linearizable storage  ⟺  Total order broadcast  ⟺  Consensus
```

---

## Distributed Transactions (2PC)

**Two-phase commit mechanism**:
1. Coordinator sends `prepare` to all participants
2. All participants vote `yes` (durable promise — cannot unilaterally abort after this)
3. Coordinator writes decision to its WAL, then sends `commit` (or `abort`)

| Phase | Coordinator failure effect |
|---|---|
| Before sending prepare | Safe: participants haven't committed; coordinator restarts and aborts |
| After some voted yes, before commit | **STUCK**: participants hold locks forever "in doubt" — blocking protocol |
| After sending commit | Safe: participants eventually receive commit |

**Three-phase commit (3PC)**: Non-blocking in theory, but assumes bounded delays → not practical in real asynchronous networks.

**XA transactions** (cross-heterogeneous-system 2PC):
- Coordinator is stateful single point of failure
- Cannot detect cross-system deadlocks
- Incompatible with SSI (optimistic concurrency)

---

## Consensus

**Properties** (all four required):

| Property | Definition | Type |
|---|---|---|
| **Uniform agreement** | No two nodes decide different values | Safety |
| **Integrity** | Each node decides at most once | Safety |
| **Validity** | Decided value was actually proposed | Safety |
| **Termination** | Every non-crashed node eventually decides | Liveness |

**Key algorithms**: Paxos, Raft, Viewstamped Replication (VSR), Zab. All share the same shape: one leader per epoch, quorum voting.

**Epoch numbering**:
- Each leader election increments the epoch number
- Higher epoch supersedes lower epoch
- Before a leader can decide, it must get quorum approval — confirming it hasn't been superseded

### 2PC vs. Consensus

| Dimension | 2PC | Consensus (Raft/Paxos) |
|---|---|---|
| Who must vote yes | **All** participants | **Majority** of nodes |
| Leader failure | Participants stuck in doubt | New leader elected, epoch incremented |
| Fault tolerance | No (coordinator is SPOF) | Yes (tolerates minority failures) |
| Blocking | Yes | No (with correct implementation) |

**Consensus limitations**:
- Requires synchronous replication between nodes (performance cost)
- Strict majority requirement — sensitive to simultaneous failures
- Frequent leader elections under network instability waste time
- Dynamic cluster membership (adding/removing nodes) is poorly understood and error-prone

---

## ZooKeeper / etcd

**Features provided**:

| Feature | Mechanism | Use case |
|---|---|---|
| Linearizable atomic ops | Consensus (Zab/Raft) | Compare-and-set, leader election |
| Total ordering | `zxid` monotonic ID | Fencing tokens, log replication |
| Failure detection | Session heartbeats + timeout | Detect dead nodes |
| Change notifications | Watch API | React to config or membership changes |
| Ephemeral nodes | Deleted on session expiry | Service discovery, presence detection |

**Usage pattern**: Outsource consensus to a **small ZooKeeper/etcd cluster (3–5 nodes)**; the application tier can then scale to thousands of nodes without implementing consensus itself.

> "The truth is defined by the majority." A node declared dead by quorum must step down, even if it believes itself alive.

ZooKeeper manages **slow-changing coordination data** (leader identity, config, locks) — not high-throughput application state.

---

## Quick Decision Framework

| Situation | Recommendation |
|---|---|
| Multi-object writes with integrity constraints | Use transactions |
| Write skew or phantom risk identified | Use serializable isolation |
| Read-heavy, eventual-consistency acceptable | Snapshot isolation sufficient |
| Need guaranteed uniqueness (username, seat) | Linearizable storage or consensus |
| Multi-datacenter, low-latency priority | Accept causal consistency; skip linearizability |
| Leader election needed | ZooKeeper/etcd — don't roll your own |
| Physical timestamps for event ordering | **Never** — use Lamport timestamps or version vectors |
| 2PC coordinator crashed with in-doubt participants | Operator must manually intervene or wait for coordinator recovery |
