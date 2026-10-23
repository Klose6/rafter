# GoKV

GoKV is a distributed transactional key-value store written in Go. It aims at the same core capabilities as [TiKV](https://github.com/tikv/tikv): a sharded key space, snapshot-isolated transactions, and strongly replicated storage.

Three pieces carry that design:

- **Percolator**, implemented in GoKV, for distributed transactions.
- **[HashiCorp Raft](https://github.com/hashicorp/raft)** for replicating each shard’s log so committed data survives node failure.
- **RocksDB** as the local storage engine on every node.

The store is the source of truth. Clients talk to it with a small KV API (`Get`, `Scan`, `Begin`, `Commit`, `Rollback`). Cross-shard transactions are coordinated by the Percolator protocol, not by a central transaction manager that holds locks.

## How the pieces fit

```
                         Client
                           |
                    Percolator client
              (prewrite / commit / get)
                           |
          +----------------+----------------+
          |                |                |
     Region A         Region B         Region C
     Raft group       Raft group       Raft group
          |                |                |
       RocksDB          RocksDB          RocksDB
     (CF: default,     ...              ...
      lock, write)
```

The key space is split into **regions**. Each region is a contiguous key range and is the unit of sharding and replication. A region is owned by one Raft group: the leader applies writes, followers replicate the log, and any replica can be promoted if the leader is lost.

A single-region write is a Raft log entry applied to RocksDB. A transaction that touches several regions runs Percolator on top of those Raft groups. Raft makes each region’s state durable and linearizable. Percolator stitches regions into one snapshot-isolated transaction.

## Storage engine

Each node keeps region data in RocksDB. Percolator needs three column families, following the same layout TiKV uses:

| Column family | What it stores |
| --- | --- |
| `default` | The value of a key at a given version. |
| `lock` | An in-progress transaction’s lock on a key. At most one lock per key. |
| `write` | The commit record: which `start_ts` became visible at which `commit_ts`. |

A key is stored as `user_key` plus a version timestamp, so RocksDB’s ordered keys give multi-version concurrency control for free. Readers pick the newest version whose commit timestamp is less than or equal to their snapshot timestamp. A missing lock column on that version means the write is committed and visible.

RocksDB is local. It does not replicate. Durability across machines comes from Raft: a write is acknowledged only after the Raft log entry is committed, then applied into RocksDB.

## Replication

Each region runs its own Raft group through `github.com/hashicorp/raft`.

- The **leader** serializes proposals. A client write becomes one log entry (or a small batch), is replicated to a quorum, and is applied to RocksDB in log order.
- **Followers** replay the same log, so every replica of a region converges to the same state machine.
- **Snapshots** compact the log. A lagging peer installs a RocksDB snapshot plus a log tail instead of replaying from the beginning.

Raft’s job is the region state machine: put, delete, and the Percolator mutations (`Prewrite`, `Commit`, `Rollback`, resolve lock). It does not know about multi-region transactions. That split keeps replication local to the shard and leaves atomicity across shards to Percolator.

## Transactions

GoKV implements [Percolator](https://research.google/pubs/large-scale-incremental-processing-using-distributed-transactions-and-notifications/) itself, rather than calling an external transaction library.

Percolator gives snapshot isolation with optimistic concurrency control. There is no central lock manager. The client drives a two-phase commit, and the locks live in the `lock` column family next to the data.

1. **Start.** The client obtains a `start_ts` from a timestamp oracle. That timestamp is the snapshot: reads see every commit with `commit_ts <= start_ts` and nothing newer.
2. **Read and buffer.** Gets and scans run at `start_ts`. Writes stay in the client until commit.
3. **Prewrite.** For every key in the write set, the client writes the value into `default` and a lock into `lock`, both tagged with `start_ts`. One key is chosen as the **primary**. Secondary locks record the primary’s location. Prewrite fails if another lock is present or if a newer committed version exists (`commit_ts > start_ts`), which is how write-write conflicts are detected.
4. **Commit.** The client takes a `commit_ts` strictly greater than `start_ts` and commits the primary first: remove its lock, write a record into `write`. After the primary commits, the transaction is decided. Secondaries are then committed the same way and can be retried safely.
5. **Rollback.** If prewrite or the primary commit fails, the client removes locks and writes a rollback record so a later retry of the same `start_ts` cannot commit.

A reader that hits a lock compares the lock’s `start_ts` with its own snapshot. A lock from a transaction that started after the reader is ignored. A lock that could affect the snapshot must be resolved: look at the primary, and either roll the transaction forward (primary already committed) or abort it (primary rolled back or timed out).

Crash safety comes from that primary. Any replica can finish a transaction it did not start, because the primary lock is the single decision point and Raft has already made each region’s lock and write records durable.

## What “similar to TiKV” means here

GoKV follows TiKV’s shape, not its full feature set.

| | TiKV | GoKV |
| --- | --- | --- |
| Language | Rust | Go |
| Transactions | Percolator | Percolator, implemented in this repo |
| Replication | raft-rs | hashicorp/raft |
| Engine | RocksDB | RocksDB |
| Shard | Region + Raft group | Region + Raft group |
| SQL layer | TiDB, separate | None; KV API only |

Scheduling, a placement driver, coprocessors, and a SQL frontend are out of scope until the KV path is solid. A timestamp oracle is in scope, because Percolator needs monotonically increasing timestamps for `start_ts` and `commit_ts`.

## Features

- Multi-version key-value storage on RocksDB (`default`, `lock`, `write`).
- Region-level replication with HashiCorp Raft, including quorum commit and snapshots.
- Distributed transactions via a built-in Percolator implementation: snapshot reads, two-phase commit, primary locks, and lock resolution.
- Optimistic concurrency control with write-write conflict detection at prewrite.

## Performance tests

Not yet. Benchmarks will cover single-region reads and writes, replicated write latency under Raft, and multi-region transaction throughput once those paths exist.

## Status

Early design. The repository currently describes the system; the storage, Raft, and Percolator layers are still to be built.
