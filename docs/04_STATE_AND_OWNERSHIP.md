> State ownership rules and invariants for the initial Core KV execution model.

## Store State

The initial single-process store uses a fixed number of node-local execution shards. Each stored key-value entry belongs
to exactly one shard.

```text
Store
  |
  +-- Shard 0
  +-- Shard 1
  +-- Shard 2
  +-- ...
```

The initial implementation does not replicate state: a key has one logical owner and one in-memory copy.

## Key Ownership

Every key deterministically maps to exactly one shard. The initial implementation uses a simplified mapping:

```text
owner(key) = key % shard_count
```

for non-negative integer keys. Given the same key and shard count, ownership resolution must always return the same 
shard. The ownership function is an initial implementation choice and may be replaced later without changing the 
client-facing API.

ScyllaDB hashes partition keys to tokens, then selects local shards through token sharding or tablet replica placement.

## Shard State

Each shard owns its own key-value state. Conceptually:

```text
Shard 0 -> Map 0
Shard 1 -> Map 1
Shard 2 -> Map 2
```

A key may exist only in the map belonging to its owner shard. No shard may directly store or mutate a key owned by 
another shard.

## Core Ownership

In the initial concurrent execution model, each shard runs on one event-loop thread associated with one CPU core.
That execution context is the exclusive owner of the shard's mutable state.

```text
Core 0 -> Shard 0 -> Map 0
Core 1 -> Shard 1 -> Map 1
Core 2 -> Shard 2 -> Map 2
```

The first implementation may establish and test logical shard ownership before CPU affinity is introduced. Physical 
pinning of a thread to a specific CPU core is therefore not required to preserve the ownership model.

## Exclusive Access

Only the execution context that owns a shard may directly access that shard's mutable key-value state. Other execution 
contexts must not directly read or mutate the shard's internal map. Cross-owner operations must eventually cross an 
explicit communication boundary rather than bypass ownership. The communication mechanism is an architectural concern 
and is intentionally not defined in this document.

Asynchronous operations on one shard may interleave at suspension points. Exclusive ownership alone does not make an
entire operation atomic; the API's consistency guarantees must also hold across those suspension points.
