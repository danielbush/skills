# Effect Platform – KeyValueStore

`KeyValueStore` service: basic operations, in-memory implementation, and custom backends.

---

### Basic Key-Value Store Operations

Source: https://effect.website/docs/platform/key-value-store

Illustrates fundamental operations of the KeyValueStore, including setting, getting, removing, and checking the size of the store. This example uses an in-memory implementation.

```typescript
import {
  KeyValueStore,
  layerMemory
} from "@effect/platform/KeyValueStore"
import { Effect } from "effect"

const program = Effect.gen(function* () {
  const kv = yield* KeyValueStore

  // Store is initially empty
  console.log(yield* kv.size)

  // Set a key-value pair
  yield* kv.set("key", "value")
  console.log(yield* kv.size)

  // Retrieve the value
  const value = yield* kv.get("key")
  console.log(value)

  // Remove the key
  yield* kv.remove("key")
  console.log(yield* kv.size)
})

// Run the p
```

---

### Accessing KeyValueStore Service

Source: https://effect.website/docs/platform/key-value-store

Demonstrates how to access the KeyValueStore service within an Effect program. This is the entry point for interacting with any KeyValueStore implementation.

```typescript
import { KeyValueStore } from "@effect/platform"
import { Effect } from "effect"

const program = Effect.gen(function* () {
  const kv = yield* KeyValueStore.KeyValueStore

  // Use `kv` to perform operations on the store
})
```

---

### KeyValueStore API

Source: https://effect.website/docs/platform/key-value-store

The KeyValueStore module provides a service for managing key-value pairs. It includes operations for getting, setting, removing, and managing entries, as well as creating schema-validated stores.

```APIDOC
## KeyValueStore API

### Description

The `@effect/platform/KeyValueStore` module provides a robust and effectful interface for managing key-value pairs. It supports asynchronous operations, ensuring data integrity and consistency, and includes built-in implementations for in-memory, file system-based, and schema-validated stores.

### Service

The module exposes a single service, `KeyValueStore`, which acts as the gateway for interacting with the store.

### Accessing the KeyValueStore Service

```md
1 import { KeyValueStore } from "@effect/platform"
2 import { Effect } from "effect"
3 
4 const program = Effect.gen(function* () {
5   const kv = yield* KeyValueStore.KeyValueStore
6 
7   // Use `kv` to perform operations on the store
8 })
```

### Operations

| Operation | Description |
|---|---|
| **get** | Returns the value as `string` of the specified key if it exists. |
| **getUint8Array** | Returns the value as `Uint8Array` of the specified key if it exists. |
| **set** | Sets the value of the specified key. |
| **remove** | Removes the specified key. |
| **clear** | Removes all entries. |
| **size** | Returns the number of entries. |
| **modify** | Updates the value of the specified key if it exists. |
| **modifyUint8Array** | Updates the value of the specified key if it exists. |
| **has** | Check if a key exists. |
| **isEmpty** | Check if the store is empty. |
| **forSchema** | Create a `SchemaStore` for the specified schema. |

### Basic Operations Example

```md
1 import {
2 KeyValueStore,
3 layerMemory
4 } from "@effect/platform/KeyValueStore"
5 import { Effect } from "effect"
6 
7 const program = Effect.gen(function* () {
8   const kv = yield* KeyValueStore
9 
10   // Store is initially empty
11   console.log(yield* kv.size)
12 
13   // Set a key-value pair
14   yield* kv.set("key", "value")
15   console.log(yield* kv.size)
16 
17   // Retrieve the value
18   const value = yield* kv.get("key")
19   console.log(value)
20 
21   // Remove the key
22   yield* kv.remove("key")
23   console.log(yield* kv.size)
24 })
25 
26 // Run the program with an in-memory layer
27 Effect.runPromise(program.pipe(KeyValueStore.layerMemory))
```
```

---

### Platform: KeyValueStore Interface

Source: https://context7_llms

Details on the KeyValueStore interface provided by @effect/platform, enabling asynchronous and consistent storage for key-value pairs across different backends.

```typescript
import * as KeyValueStore from "@effect/platform/KeyValueStore"
import * as Effect from "@effect/io/Effect"

// Assume an in-memory implementation
const store: KeyValueStore.KeyValueStore<string, string> = KeyValueStore.inMemory()

const putEffect = store.put("mykey", "myvalue")
const getEffect = store.get("mykey")

Effect.runSync(Effect.zip(putEffect, getEffect)) // [void, Maybe<string>]
```
