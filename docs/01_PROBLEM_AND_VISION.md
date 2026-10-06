# Core KV

> A distributed key-value store built around shard-per-core ownership.

## The Problem: Distributed State Must Scale Across Machines and CPU Cores

Stateful services are constrained not only by aggregate compute capacity, but by **state ownership and data locality**.
Once a dataset or request rate exceeds the practical capacity of one machine, state must be partitioned across multiple
machines. Requests can no longer be sent arbitrarily, each key must reach a node that owns or replicates the relevant
state, while failures, membership changes, and rebalancing can change that ownership over time.

Adding more machines solves only one level of the problem. Inside each machine, conventional multithreaded designs often
allow many threads to access shared data structures protected by locks or atomics. As core counts and request rates 
grow, shared mutable state can introduce lock contention, cache-line bouncing, synchronization overhead, and poor 
locality. The distributed system may scale across machines while failing to efficiently use the hardware within each 
machine.

For a distributed key-value store, the hard problem is therefore not simply:

> **How do we partition keys across multiple machines?**

It is:

> **How do we assign each key to a clear owner across both machines and CPU cores, while preserving durability,
> replication, availability, and efficient request processing as the system changes?**

A request may arrive at the wrong machine or the wrong core. A leader may fail after accepting a write. Replicas may 
lag. A new node may require partitions to move. And cross-core communication may become expensive even when no network 
is involved. The target is not zero coordination cost (that is impossible) but **explicit ownership and locality**, so 
that coordination occurs only where the system's correctness requires it.

## Proposal

**Core KV** is a C++ distributed key-value store organized around shard-per-core state ownership.

Keys map to logical partitions, partitions map to machines, and each machine maps its local partitions to dedicated CPU
shards. A core owns and mutates its local state, requests for another shard are forwarded through explicit message 
passing rather than allowing arbitrary threads to concurrently mutate the same data structures.

``` text
Key <-> Partition <-> Machine <-> CPU Core <-> Local State
```

The data plane should keep the common request path local to the owning core while supporting explicit cross-core and
cross-node communication when required. The control plane maintains cluster membership, partition ownership, replica
placement, leadership/epochs, and rebalancing state.

A critical requirement is **ownership-safe concurrency**. A key cannot be concurrently mutated by arbitrary worker
threads merely because they share an address space. Core KV therefore makes ownership explicit: state is mutated by its
owning shard, while other shards communicate with that owner through queues/messages, making synchronization boundaries
visible in the architecture.

The long-term objective is simple:

> Given a distributed key space and a multicore cluster, route each operation to the state owner while keeping the 
> common path local, replicated, durable, and recoverable.

Core KV does **not** claim that shard-per-core eliminates coordination or is optimal for every workload. Performance
remains bounded by workload skew, replication cost, storage latency, network bandwidth, cross-core communication, 
rebalancing, and the consistency guarantees the system provides.

## Why Existing Solutions Do Not Fully Provide This Abstraction

| Technology | What It Solves | Remaining Gap For Core KV |
|-|-|-|
| **External Distributed Databases** | Durable, replicated, horizontally scalable application state behind a network-accessible service. | Hide most state ownership from the application tier. They do not by themselves expose the interaction between distributed ownership and per-core execution that Core KV is intended to explore. |
| **ZooKeeper / etcd** | Strongly consistent coordination metadata, membership/configuration primitives, watches, and leader/lease-style coordination. | Coordination systems, not general high-throughput storage engines for application data. They do not implement Core KV's data path, per-core ownership, WAL, partition replication, or rebalancing. |
| **Raft Libraries** | Reusable consensus and replicated-log machinery. | Solve consensus, not the complete storage architecture: key partitioning, core-local execution, persistence, routing, membership integration, state transfer, and rebalancing still remain. |
| **Conventional Thread-Pool KV Stores** | Parallel request processing over shared in-process state. | Shared mutable structures can require locks/atomics and induce contention or cache-coherence traffic. Core KV explicitly investigates ownership-oriented, shard-per-core execution. |
| **Production Shard-Per-Core Databases** | Mature distributed storage with sophisticated replication, scheduling, persistence, and operational tooling. | Already solve the production problem. Core KV is intentionally smaller and educational: it exposes the mechanisms so their distributed-systems and hardware-level tradeoffs can be implemented, measured, and understood directly. |

Core KV should **reuse commodity mechanisms where they are not the subject of study rather than reproduce an entire
production database ecosystem**. Kubernetes can provide process orchestration, standard observability tooling can expose
metrics, and established serialization or networking libraries can eventually provide plumbing. Core KV's intended
contribution is the ownership-oriented storage engine connecting distributed state management to multicore hardware
behavior.

## Closing

Modern multicore machines provide enormous aggregate compute capacity, but that capacity is not automatically useful 
when threads contend over shared state. Likewise, adding machines creates aggregate capacity, but turns state placement,
replication, routing, failure recovery, and rebalancing into distributed-systems problems.

Core KV aims to make both levels of ownership explicit in one system.

The initial milestone should be deliberately narrow: **one process, multiple CPU cores, an in-memory key-value API,
deterministic key-to-shard mapping, persistent worker threads, CPU affinity where supported, per-core state ownership,
and explicit cross-shard message passing**. WAL persistence, networking, multi-node partitioning, replication, Raft,
failure recovery, dynamic membership, and rebalancing can then extend the same ownership model across machines.

The project's success criterion is not "build another key-value store", mature systems already provide that.

**It is understanding and demonstrating how distributed state ownership interacts with multicore hardware.**
