# Core KV

**A distributed key-value store built around shard-per-core ownership.**

Core KV is an open-source C++ project exploring a distributed key-value store whose data is partitioned across machines
and, within each machine, across CPU cores.

The goal is to coordinate **partitioning, routing, replication, persistence, failure recovery, and rebalancing** while
using per-core state ownership, CPU affinity, and explicit message passing to minimize shared-memory contention.

## Status

🚧 **Early development.**

CoreKV is currently in the design and prototyping stage. APIs, architecture, and implementation details are expected to 
change.

## Documentation

Start here:

**[Problem & Vision](docs/01_PROBLEM_AND_VISION.md)** — the problem Core KV aims to solve, the proposed approach, and how it differs from 
conventional shared-state and distributed key-value-store architectures.

## License

See [LICENSE](LICENSE).
