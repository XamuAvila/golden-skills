---
name: data-intensive-apps
description: >
  Practical reference for designing data-intensive applications based on Martin Kleppmann's
  "Designing Data-Intensive Applications" (O'Reilly). Use when choosing databases (relational vs document
  vs graph), designing storage engines (LSM-trees vs B-trees, OLTP vs OLAP), selecting replication
  strategies (single-leader vs multi-leader vs leaderless), partitioning/sharding data, understanding
  transactions and isolation levels (read committed, snapshot isolation, serializability), reasoning
  about distributed systems problems (network faults, clock skew, split brain, Byzantine faults),
  implementing consensus (2PC, Paxos, Raft, ZooKeeper), or designing batch/stream processing
  pipelines (MapReduce, Spark, Flink, Kafka, CDC, event sourcing, CQRS). Also trigger on:
  reliability/scalability/maintainability trade-offs, CAP theorem, linearizability vs causal consistency,
  quorum reads/writes, encoding formats (Protobuf, Avro, Thrift), schema evolution, data warehouse
  star/snowflake schemas, column-oriented storage, change data capture, exactly-once semantics,
  stream joins, windowing (tumbling/hopping/sliding/session), total order broadcast, Lamport timestamps,
  version vectors, conflict resolution (CRDTs, LWW), failover pitfalls, write skew, phantom reads,
  fencing tokens, or any question about data system architecture, trade-offs, and design decisions.
---

# Designing Data-Intensive Applications — Skill Guide

**Source**: "Designing Data-Intensive Applications" by Martin Kleppmann (O'Reilly, 2017)

## Overview

This skill encapsulates the core principles, decision frameworks, and trade-offs from the book. It's organized into four reference domains matching the book's three Parts (Part II is split due to content density). Read ONLY the relevant reference file for the domain the user is asking about.

## Quick Navigation

| User is asking about... | Read this reference |
|---|---|
| Reliability, scalability, maintainability, data models (relational vs document vs graph), query languages, storage engines (LSM-trees, B-trees), OLTP vs OLAP, data warehousing, column storage, encoding formats (JSON, Protobuf, Avro, Thrift), schema evolution, data flow modes (REST, RPC, message brokers) | `references/foundations.md` |
| Replication (single-leader, multi-leader, leaderless), replication lag, quorums, conflict resolution, failover, partitioning/sharding strategies, rebalancing, secondary indexes, request routing | `references/distributed-data.md` |
| Transactions, ACID, isolation levels (read committed, snapshot isolation, serializability), race conditions (dirty reads, lost updates, write skew, phantoms), 2PL, SSI, distributed systems problems (unreliable networks, clocks, process pauses), CAP theorem, linearizability, causal consistency, consensus (2PC, Raft, Paxos), ZooKeeper, total order broadcast | `references/transactions-consistency.md` |
| Batch processing (Unix philosophy, MapReduce, Spark, Flink), join algorithms, stream processing (Kafka, event streams, CDC, event sourcing, CQRS), windowing, stream joins, exactly-once semantics, fault tolerance in streams | `references/derived-data.md` |

## Master Decision Framework

### Choosing a Data Model

| Data characteristic | Best model |
|---|---|
| Self-contained documents, one-to-many trees | Document (MongoDB, CouchDB) |
| Many-to-many relationships, joins, normalized data | Relational (PostgreSQL, MySQL) |
| Highly interconnected data, variable-length traversals | Graph (Neo4j, Datomic) |
| Heterogeneous data types, schema-on-read | Document |
| Uniform structure, schema-on-write | Relational |

### Choosing a Replication Strategy

| Need | Strategy |
|---|---|
| Simple setup, most workloads | Single-leader |
| Multi-datacenter, low write latency per DC | Multi-leader |
| Offline-first / mobile sync | Multi-leader (each device = leader) |
| High availability, eventual consistency OK | Leaderless (Dynamo-style) |
| Strong consistency required | Single-leader + synchronous replication |

### Choosing a Partitioning Strategy

| Access pattern | Strategy |
|---|---|
| Range queries critical (time-series, sorted data) | Key range partitioning |
| Uniform distribution, no range queries | Hash partitioning |
| Both: partition by one dimension, sort by another | Compound key (hash + sort) |
| Celebrity/hot-key problem | Application-level: random suffix on hot keys |

### Choosing an Isolation Level

| Situation | Isolation level |
|---|---|
| Simple read-write, no complex invariants | Read committed |
| Long-running read queries, analytics, backups | Snapshot isolation (MVCC) |
| Write skew or phantom problems | Serializable (SSI, 2PL, or serial execution) |
| Maximum throughput, single-threaded fits | Actual serial execution |

### Choosing a Processing Paradigm

| Characteristic | Paradigm |
|---|---|
| Bounded input, throughput-oriented | Batch (MapReduce, Spark) |
| Unbounded input, low-latency needed | Stream (Kafka Streams, Flink) |
| Both: batch for history + stream for real-time | Lambda architecture (or unified with Flink) |

## Three Pillars of Data System Design

Every data system design balances three concerns:

1. **Reliability** — continues working correctly under faults (hardware, software, human error)
2. **Scalability** — copes with growth in data volume, traffic, or complexity
3. **Maintainability** — operability, simplicity, evolvability for current and future teams

## How to Use This Skill

1. Identify which domain(s) the question falls into using the Quick Navigation table above.
2. Read the relevant reference file(s) for specific guidance, trade-off tables, and decision frameworks.
3. Provide concrete, actionable answers grounded in the book's principles — always state trade-offs, not just recommendations.
4. When the user faces a design decision, use the Master Decision Framework tables above for quick guidance, then dive into the reference file for nuance.
5. Adapt advice to context — team size, scale requirements, consistency needs, and operational maturity matter.
