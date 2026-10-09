> Fundamental terminology used throughout Core KV.

## Store

The **store** is the complete logical key-value dataset managed by Core KV. The store may span multiple compute nodes, 
but clients interact with it as one logical key-value store.

## Shard

A **shard** is a node-local execution context, consisting of one event-loop thread associated with one CPU core and
owning a subset of the node's local state. It is distinct from a distributed data partition or tablet, whose replicas
may be assigned to shards on multiple nodes. In the initial single-process store, every key belongs to exactly one shard.

## Compute Node

A **compute node** is one physical or virtual machine participating in the Core KV cluster. Each compute node provides 
CPU cores, memory, local storage, and network connectivity and may host multiple shards.

## Core

A **core** is an independently executing processing unit within a CPU. Within a Core KV compute node, each shard's
execution thread is associated with one core and owns and executes operations against that shard's state.
