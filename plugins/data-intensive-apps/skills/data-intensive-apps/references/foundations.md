# Foundations of Data Systems

## Reliability

**Fault vs failure**: fault = one component deviates from spec; failure = system stops serving users. Goal: prevent faults from causing failures.

**Chaos engineering**: deliberately induce faults (Netflix Chaos Monkey) to verify fault-tolerance machinery works in production.

| Fault Category | Characteristics | Mitigations |
|---|---|---|
| **Hardware** | Disk MTTF 10-50 years; RAM errors; power failures; independent failures | RAID, dual power supplies, hot-swap CPUs; moving to software fault-tolerance (cloud VMs disappear without warning) |
| **Software** | Systematic, correlated across nodes; leap-second bugs; runaway processes; cascading failures | Careful assumption-checking, process isolation, crash-and-restart, monitoring, self-checking invariants |
| **Human** | Configuration errors = leading cause of outages (hardware = only 10-25%) | APIs that make "right thing" easy, sandbox environments, thorough testing, quick rollback, detailed telemetry |

- Never sacrifice reliability without a conscious decision (prototype/unproven market only).
- Software errors are harder to anticipate than hardware faults — they are correlated, not independent.

---

## Scalability

**Definition**: not a one-dimensional label — ask "if the system grows in X way, what are our options?"

**Load parameters**: requests/sec, read/write ratio, concurrent users, cache hit rate, fan-out. Choice depends on architecture.

**Twitter fan-out case study**:
- Query-on-read: simple writes, expensive reads (JOIN at read time)
- Fan-out-on-write: expensive writes, cheap reads (pre-compute per-user timelines)
- Hybrid: fan-out for regular users, query-on-read for celebrities (high follower count)
- Key load parameter = distribution of followers per user weighted by tweet frequency

### Latency & Percentiles

| Metric | Definition | When to use |
|---|---|---|
| Response time | Service time + network delays + queueing | Client-side measurement (always) |
| Latency | Duration request waits before being handled | Distinguishing wait from processing |
| p50 (median) | Typical user experience threshold | Baseline performance |
| p95 / p99 | Tail latencies affecting most valuable users | SLA targets |
| p999 | 1-in-1000 worst case | Amazon-style high-value customer protection |

- Amazon: +100ms → -1% sales; +1s → -16% satisfaction.
- **Head-of-line blocking**: slow requests hold up fast ones — measure on client side, not server.
- **Tail latency amplification**: fan-out to N backends means P(at least one slow) compounds quickly.
- Never average percentiles across machines — add histograms (forward decay, t-digest, HdrHistogram).

### Scaling Approaches

| Approach | Description | When to prefer |
|---|---|---|
| **Vertical (scale up)** | More powerful single machine | Stateful data; simpler ops; early stage |
| **Horizontal (scale out)** | Shared-nothing, distribute across machines | Stateless services; cost at scale |
| **Elastic** | Auto-add resources when load spikes | Unpredictable load; cloud environments |

- Pragmatic reality: a few powerful machines is often simpler than many small VMs.
- Anti-pattern: premature scaling architecture for unproven products — iterate on features first.

---

## Maintainability

| Principle | Goal | Key practices |
|---|---|---|
| **Operability** | Easy for ops teams to keep system running | Good monitoring, automation support, no single-machine dependencies, predictable behavior, self-healing with manual override |
| **Simplicity** | Remove accidental complexity | Abstractions that hide implementation detail behind clean facades |
| **Evolvability** | Adapt system to changing requirements | Simple systems are easier to modify; linked to simplicity |

- **Accidental complexity**: complexity from implementation, not inherent in the problem — eliminate it via abstraction.

---

## Data Models

### Decision Table

| Data characteristic | Best model | Why |
|---|---|---|
| Self-contained documents, 1:many tree, rare cross-doc relations | **Document** | Locality, schema flexibility, closer to app objects |
| Many-to-many relationships, normalized data, joins needed | **Relational** | Native joins, mature query optimizers, normalization |
| Highly interconnected, variable-length traversals, heterogeneous types | **Graph** | Natural traversal, evolvable schema, no structural constraint |
| Heterogeneous types, externally-determined structure | **Document** (schema-on-read) | No rigid schema to maintain |
| Uniform structure, all records same shape | **Relational** (schema-on-write) | Schema documents and enforces structure |
| Need data locality (load whole entity in one query) | **Document** | Single continuous storage |
| Many-to-one references, deduplication | **Relational** | IDs + joins are native |

### Schema-on-Read vs Schema-on-Write

| | Schema-on-Read | Schema-on-Write |
|---|---|---|
| **Enforcement** | Implicit, at read time | Explicit, on write |
| **Analogy** | Dynamic/runtime type checking | Static/compile-time type checking |
| **Best for** | Heterogeneous data, externally-determined structure | Uniform data, documentation via schema |
| **Migration** | Write new format, handle old in app code | `ALTER TABLE` + `UPDATE` |
| **MySQL caveat** | N/A | Copies entire table on ALTER (slow) |

### Key Concepts

- **Impedance mismatch**: disconnect between OOP objects and relational tables — ORMs reduce but don't eliminate.
- **Normalization**: store ID referencing single canonical copy; IDs never change, human-readable strings do. Requires joins.
- **Data locality**: documents stored as single continuous string (JSON/BSON); advantage for loading entire entity; rewrite required for size-changing updates — keep documents small.
- **Convergence trend**: PostgreSQL adds JSON/XML; RethinkDB adds joins — hybrid models becoming standard.
- **Polyglot persistence**: use multiple DB technologies alongside each other for different needs.

---

## Query Languages

| | Declarative | Imperative |
|---|---|---|
| **Style** | Specify WHAT result you want | Specify HOW to traverse step-by-step |
| **Examples** | SQL, CSS, XPath, Cypher, SPARQL | IMS, CODASYL |
| **Parallelism** | Engine decides — lends itself to parallelization | Hard to parallelize |
| **Optimization** | Engine can change strategy without rewriting query | Fixed access path |
| **Conciseness** | Higher | Lower |

**MapReduce**: neither fully declarative nor imperative. Map + reduce functions must be pure (no side effects). MongoDB evolved to aggregation pipeline (declarative, JSON-based) — prefer it over MapReduce.

### Graph Query Languages

| Language | Model | Style | Notable |
|---|---|---|---|
| **Cypher** | Property graph (Neo4j) | Declarative, pattern-matching with `(a)-[:EDGE]->(b)` | Most concise for graph traversal |
| **SPARQL** | RDF triple-store | Declarative, predates Cypher, similar pattern-matching | Used with linked-data / semantic web |
| **Datalog** | Triple-store (Datomic) | Rule-based, define predicates from other predicates | Most powerful for complex recursive queries |

- Graph queries in SQL via `WITH RECURSIVE` — possible but extremely verbose (29 lines vs 4 in Cypher for same query).

---

## Storage Engines

### Hash Indexes

- In-memory hash map: key → byte offset in append-only disk file (Bitcask / Riak).
- **Use when**: key count fits in RAM, high write volume per key, frequent updates.
- **Anti-pattern**: too many distinct keys (won't fit in memory); range queries are inefficient.
- Compaction: throw away duplicate keys, keep most recent. Merge segments simultaneously. Write to new file — never modify existing.
- Append-only is good: sequential writes >> random writes; simpler crash recovery; merging avoids fragmentation.

### SSTables & LSM-Trees

**SSTable** (Sorted String Table): log segments with keys sorted within each segment.
- Advantages over hash indexes: mergesort merging (even beyond RAM); sparse in-memory index (1 key per few KB); compressible blocks between index entries.

**Memtable**: in-memory balanced tree (Red-Black tree) accepting writes in sorted order. Flush to SSTable when threshold (~few MB) exceeded. WAL on disk prevents loss before flush.

**LSM-Tree** (Log-Structured Merge-Tree): memtable → SSTable segments → background compaction.
- Implementations: LevelDB, RocksDB, Cassandra, HBase, Lucene (term dictionary).
- **Bloom filters**: skip checking segments for non-existent keys — critical optimization.
- High write throughput (sequential disk writes). Efficient range queries (sorted data).

### B-Trees

- Standard in almost all relational databases.
- Fixed-size **pages** (4 KB); tree of pages with O(log n) height; updated **in-place**.
- **Branching factor** ≈ several hundred child references per page.
- **WAL (write-ahead log)**: every modification logged before applying to tree — crash recovery.
- **Write amplification**: 1 logical write → WAL write + page write + possible page split.
- Concurrency via **latches** (lightweight locks).

### LSM vs B-Tree Decision

| Dimension | LSM-Tree | B-Tree |
|---|---|---|
| Write throughput | Higher (sequential writes) | Lower (random writes, WAL + page) |
| Read speed | Slower (check multiple levels) | Faster (single lookup path) |
| Write amplification | Can be lower | Higher (WAL + pages + splits) |
| Latency predictability | Can stall at high percentiles (compaction) | More predictable |
| Space usage | Lower (compaction removes fragmentation) | Can have page fragmentation |
| Transaction isolation | Harder (key in multiple segments) | Easier (lock key range in index) |
| Maturity | Newer, gaining adoption | Battle-tested decades |

**Rule**: LSM faster for writes; B-tree faster for reads. Benchmarks are workload-sensitive — test empirically.

### Index Types

| Index Type | Description | Use case |
|---|---|---|
| **Heap file** | Index points to row in unordered file | Multiple secondary indexes without duplicating data |
| **Clustered index** | Row data stored inside the index | Fast reads by primary key (MySQL InnoDB default) |
| **Covering index** | Some columns stored in index | Answer queries from index alone (index-only scan) |
| **Concatenated index** | Fields appended into one key | Queries filtering on leading columns (order matters!) |
| **Multi-dimensional (R-tree)** | Index across multiple dimensions | Geospatial queries (PostGIS), multi-field range queries |
| **Fuzzy / full-text** | Finite state automaton / Levenshtein | Misspelling search, edit-distance queries (Lucene) |

### In-Memory Databases

- Examples: VoltDB, MemSQL (relational); Redis, Couchbase (key-value).
- Performance gain is NOT from avoiding disk reads (OS cache handles that) — it's from eliminating the overhead of encoding data structures for disk.
- Enables data models hard on disk: Redis priority queues, sorted sets.
- Durability options: WAL, snapshots, replication, battery-backed RAM.
- **Anti-caching**: evict LRU records to disk, reload on access — more efficient than OS virtual memory paging.

---

## OLTP vs OLAP

| Property | OLTP | OLAP |
|---|---|---|
| Read pattern | Few records per query, by key | Aggregate over millions of records |
| Write pattern | Random-access, low-latency (user input) | Bulk ETL or event stream |
| Users | End users via web app | Internal analysts, decision support |
| Data represents | Latest state | History of events |
| Dataset size | GB–TB | TB–PB |
| Storage engine | Row-oriented | Column-oriented |

### Data Warehousing

- Separate DB from OLTP systems, populated via **ETL** (Extract-Transform-Load).
- **Star schema**: central **fact table** (events with FK to dimensions) surrounded by **dimension tables** (who, what, where, when, how, why).
- **Snowflake schema**: dimensions broken into sub-dimensions (more normalized). Star preferred for analyst simplicity.
- Fact tables: 100+ columns, petabytes of rows.

### Column-Oriented Storage

- Store all values from each column together — load only columns needed by query → massive I/O savings.
- Consistent row ordering across all column files: k-th item in column A = k-th row overall.
- **Column compression**: bitmap per distinct value, 1 bit per row. Run-length encoding for sparse bitmaps.
- **Vectorized processing**: operate on compressed column chunks fitting in CPU L1 cache.
- **Sort order**: sorting by chosen key (e.g., `date_key`) dramatically compresses the first sort column (long runs). Vertica/C-Store: different replicas sorted differently for different query patterns.
- **Writes**: LSM-tree approach — in-memory store eventually merged with column files on disk.

### Materialized Views & OLAP Cubes

- **Materialized view**: pre-computed query result on disk, updated when data changes. Common in OLAP, rare in OLTP (makes writes expensive).
- **Data cube / OLAP cube**: grid of aggregates grouped by all dimension combinations. Fast for known aggregates; inflexible for ad-hoc queries.
- Best practice: keep raw data; use cubes only as targeted performance optimization.

---

## Encoding Formats

**Encoding** = in-memory representation → byte sequence (file/network). **Decoding** = reverse.

### Format Comparison

| Criterion | JSON / XML / CSV | Thrift / Protobuf | Avro |
|---|---|---|---|
| Human-readable | Yes | No | No |
| Schema required | No (optional) | Yes (IDL + field tags) | Yes (IDL or JSON, no tags) |
| Compactness | Low (81 bytes sample) | High (33–59 bytes) | Highest (32 bytes) |
| Dynamic schema generation | N/A | Awkward (manual tag assignment) | Excellent (no tags needed) |
| Cross-org interchange | Best | Good for internal services | Good for Hadoop ecosystem |
| Static type codegen | No | Excellent | Optional |
| Dynamic languages | Native | Codegen friction | Self-describing files, no codegen |

**Anti-pattern**: never use language-specific serialization (Java `Serializable`, Python `pickle`) outside of transient in-process data. Reasons: tied to one language, security risk (remote code execution via arbitrary class instantiation), versioning as afterthought, poor efficiency.

### Schema Evolution Rules

| Format | Add field | Remove field | Rename field | Change type |
|---|---|---|---|---|
| **Protobuf / Thrift** | Safe if `optional` or has default | Safe if `optional`; NEVER reuse tag number | Unsafe (breaks wire format) | Widening safe (int32→int64); narrowing risks truncation |
| **Avro** | Safe only if field has default | Safe only if field has default | Backward-compatible only (use alias) | Only via union type changes |
| **JSON / XML** | Safe (parser ignores unknown) | Safe (missing = null/default) | Unsafe (all consumers must update) | Risky (no type enforcement) |

- **Forward compatibility**: old code reads data written by new code → must ignore unknown fields.
- **Backward compatibility**: new code reads data written by old code → tag numbers / field names must not change.
- Rolling upgrades require **both** directions simultaneously.

### Avro-Specific

- No field tags or identifiers in encoded data — most compact.
- Reader's schema and writer's schema matched by **field name** (not position, not tag).
- Writer's schema delivery: (1) large files → once at file header; (2) DB records → version number + schema registry; (3) network → negotiated at connection setup.
- Best for dynamically generated schemas (e.g., mirror a relational DB schema to Avro automatically).

---

## Data Flow Modes

### Through Databases

- Writer encodes; reader decodes — reader may be a future version ("message to your future self").
- **Trap**: old code that reads → modifies → writes back may silently strip unknown fields added by newer code. Application layer must preserve unknown fields during re-encoding.
- **Data outlives code**: values written years ago remain in original encoding; schema migration via `ALTER TABLE` fills nulls on read without rewriting (except MySQL, which rewrites entire table).
- Archival dumps: use latest schema; Parquet or Avro object container files are ideal.

### REST vs RPC

| | REST | RPC |
|---|---|---|
| **Model** | Resources + HTTP verbs | Local function call abstraction |
| **Best for** | Public APIs, external consumers | Internal inter-service communication |
| **Debugging** | Easy (curl, browser) | Harder |
| **Schema** | Optional (OpenAPI/Swagger) | IDL required (Protobuf, Thrift) |
| **Streaming** | Limited (SSE / WebSockets) | First-class in gRPC |
| **Fundamental flaw** | None | Pretends network = local call |

**RPC problems**: unpredictable failures, retries risk duplication (idempotency needed), variable latency, parameter encoding overhead, cross-language type translation. Modern RPC (gRPC, Finagle) is more honest about network but doesn't eliminate the mismatch.

**RPC compatibility**: server updated first → backward-compat on requests, forward-compat on responses. Multiple API versions maintained side-by-side.

### Message Brokers

- Asynchronous, one-way — sender doesn't wait for response.
- Sits between RPC (low latency) and databases (durable storage).
- Examples: RabbitMQ, ActiveMQ, NATS, Apache Kafka.

| Advantage over RPC | Explanation |
|---|---|
| Buffering | Works even if recipient is temporarily unavailable |
| Auto-redelivery | Resends on recipient crash |
| Address decoupling | Sender doesn't need to know recipient's address |
| Fan-out | One message → multiple recipients |
| Logical decoupling | Producer and consumer deployable independently |

- Message = byte sequence → any encoding format works. Use format with forward/backward compatibility for independent deployments.

### Distributed Actor Frameworks

- Actor model: local state per actor, async messages, single-threaded (no locks).
- Distributed actors (Akka, Orleans, Erlang OTP): messages encoded and sent transparently over network.
- **Danger**: rolling upgrades require forward/backward compatibility on all messages.
- Akka defaults to Java serialization (no compat guarantees) — replace with Protobuf.
- Erlang OTP: schema changes are hard due to record encoding approach.
