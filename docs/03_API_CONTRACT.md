> The external contract exposed by the Core KV store.

## Store

A `Store` represents one logical key-value store. The initial single-process store uses a fixed number of execution shards:

```cpp
Store store(shard_count);
```

`shard_count` must be greater than zero. Shard placement, ownership, and request routing are internal implementation 
details and are not exposed through the store API.

## Key

A **key** uniquely identifies an entry within the store. Each key can be associated with at most one value at any point 
in time.

## Value

A **value** is the data associated with a key. Core KV treats values as opaque data; their interpretation belongs to the caller.

## Put

```cpp
store.put(key, value);
```

`put` associates `value` with `key`. If the key does not exist, a new entry is created. If the key already exists, its 
existing value is replaced. After a successful `put(key, value)`, a subsequent `get(key)` returns `value` unless the key 
is subsequently modified or removed.

## Get

```cpp
store.get(key);
```

`get` retrieves the value currently associated with `key`. If the key exists, the associated value is returned. If the 
key does not exist, the operation returns `NOT_FOUND`. `get` does not modify the store.

## Remove

```cpp
store.remove(key);
```

`remove` removes the entry associated with `key`. If the key exists, the entry is removed. If the key does not exist, 
the operation returns `NOT_FOUND`. After a successful `remove(key)`, a subsequent `get(key)` returns `NOT_FOUND` unless 
the key is subsequently inserted again.

## Initial Consistency Model

The initial implementation provides **linearizable semantics within a single Core KV process**. Once an operation 
completes successfully, every subsequently invoked operation observes its effect. Distributed consistency semantics will 
be defined when Core KV introduces replication and multiple compute nodes.

This is Core KV's initial contract. ScyllaDB's ordinary table operations use tunable consistency and timestamp-based
conflict resolution; linearizable operations use explicit consensus paths (Paxos/LWT or Raft for strongly consistent tablets).

## Initial Scope

The initial API intentionally exposes only:

```text
PUT
GET
REMOVE
```

Partitioning, shard ownership, request routing, concurrency, persistence, replication, and failure recovery are 
implementation concerns and are not part of the client-facing API contract.
