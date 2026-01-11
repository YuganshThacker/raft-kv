raft-kv

A fault-tolerant, strongly consistent key-value store built in Go using the Raft consensus algorithm.

Overview

raft-kv is a distributed key-value store designed to explore how modern systems maintain consistency and availability in the presence of failures.

The project focuses on:

Leader-based replication

Consensus via Raft

Crash recovery

Linearizable reads and writes

Operational observability

It is inspired by systems such as etcd and Consul, with an emphasis on clarity and correctness over raw throughput.

Design Goals

Strong consistency (linearizable operations)

Single active leader at any time

Fault tolerance under node crashes and restarts

Deterministic recovery using persisted logs

Operational visibility via metrics

Non-goals:

Geo-replication

Exactly-once semantics

High-throughput streaming workloads

Architecture
Client
  |
 gRPC
  |
┌──────────────┐
│ Leader Node  │
└─────┬────────┘
      │ Raft log replication
┌─────▼────────┐   ┌─────────────┐
│ Follower     │   │ Follower    │
└──────────────┘   └─────────────┘


Writes are accepted only by the leader

Log entries are replicated to followers

Entries are committed after majority agreement

Committed entries are applied to the state machine

Consistency Model

Writes: Linearizable

Reads: Served by the leader to guarantee freshness

Replication: Majority quorum required for commit

This design avoids split-brain scenarios and ensures correctness under network partitions.

API

The system exposes a simple gRPC interface:

Operation	Description
Put(key, value)	Store a value
Get(key)	Retrieve a value
Delete(key)	Remove a key

All write operations are routed through the current leader.

Internal Components
Raft

Leader election

Heartbeats

Log replication

Commit index tracking

Term management

State Machine

In-memory key-value store

Applies committed log entries deterministically

Write-Ahead Log (WAL)

Persists log entries before commit

Enables crash-safe recovery

Snapshotting

Periodic snapshots of state

Log compaction to bound storage growth

Observability

Commit latency

Replication lag

Leader changes

Request throughput

Metrics are exported in Prometheus format.

Failure Handling

The system is designed to recover from common failure modes:

Leader crash: A new leader is elected automatically

Follower crash: The node catches up via log replication

Network partition: Only the majority partition remains writable

Process restart: State is reconstructed from WAL and snapshots

Repository Layout
raft-kv/
├── cmd/
│   ├── node/        # KV node entry point
│   └── client/      # CLI client
├── internal/
│   ├── raft/        # Consensus logic
│   ├── store/       # State machine
│   ├── wal/         # Write-ahead log
│   ├── snapshot/    # Snapshot management
│   ├── api/         # gRPC handlers
│   └── metrics/     # Prometheus metrics
├── proto/
├── tests/
├── docker-compose.yml
└── README.md

Running Locally
Requirements

Go 1.21+

Docker & Docker Compose

Start a local cluster
docker-compose up

Example usage
raftkv put user:1 Alice
raftkv get user:1
raftkv delete user:1

Project Status

Status: Active development

Implemented

Leader election

Log replication

Majority-based commits

Basic KV operations

Crash recovery

Metrics export

Planned

Snapshot installation on lagging nodes

Read optimizations

Fault-injection testing

Stress and soak testing

Trade-offs

Prioritizes correctness over throughput

Uses leader-based reads to simplify consistency

Accepts higher latency under failures for stronger guarantees

These decisions are intentional and documented.

Motivation & Learning

This project was built to gain hands-on experience with:

Distributed consensus

Failure recovery

Consistency models

Designing observable backend systems

Contributing

Issues, discussions, and pull requests are welcome.
The project aims to remain readable, well-documented, and educational.

License

MIT