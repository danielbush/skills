# Effect – Caching

`Cache`, time-to-live (TTL), and `Layer.memoize` patterns.

---

### Apply Custom Cache Directly to a Program

Source: https://effect.website/docs/batching

This example shows how to create a custom cache using `Request.makeCache` and then apply it directly to a specific program segment using `Effect.withRequestCache`. This ensures that requests within that segment are managed by the custom cache, assuming caching is enabled.

```typescript
// Assuming 'myProgram' is a defined Effect program
const customCache = Request.makeCache({
  capacity: 128,
  timeToLive: "30 minutes"
});

const programWithDirectCache = Effect.withRequestCache(customCache)(myProgram);
```

---

### Get Value from Cache (Compute if Missing)

Source: https://effect.website/docs/caching/cache

Illustrates the idiomatic way to retrieve a value from a cache using the `get` method. If the value for the given key exists in the cache and is still valid, it's returned directly. Otherwise, the `lookup` function is invoked to compute the value, which is then stored in the cache before being returned. This method ensures that even with multiple concurrent requests for the same key, the value is computed only once.

```typescript
interface Cache<Key, Value, Requirements, Error> {
  readonly get: (key: Key) => Effect<Value, Error, Requirements>
}
```

---

### Defining a Cache Service with Effect.Service and Dependencies

Source: https://effect.website/docs/requirements-management/layers

Shows how to define a Cache service using Effect.Service, specifying its effectful creation logic and its dependency on the FileSystem service. This simplifies service definition by combining tag, layer, and dependencies.

```typescript
import { FileSystem } from "@effect/platform"
import { NodeFileSystem } from "@effect/platform-node"
import { Effect } from "effect"

// Define a Cache service
class Cache extends Effect.Service()("app/Cache", {
  // Define how to create the service
  effect: Effect.gen(function* () {
    const fs = yield* FileSystem.FileSystem
    const lookup = (key: string) => fs.readFileString(`cache/${key}`)
    return { lookup } as const
  }),
  // Specify dependencies
  dependencies: [NodeFileSystem.layer]
}) {}
```

---

### Create and Apply Custom Cache Globally

Source: https://effect.website/docs/batching

This snippet demonstrates how to create a custom request cache with a specified capacity and time-to-live, and then apply it globally to a program using `Effect.provide` and `Layer.setRequestCache`. The program fetches todos and processes them, repeating every 10 seconds.

```typescript
const program = Effect.gen(function* () {
  const todos = yield* getTodos;
  yield* Effect.forEach(todos, (todo) => notifyOwner(todo), {
    concurrency: "unbounded"
  });
}).pipe(
  Effect.repeat(Schedule.fixed("10 seconds")),
  Effect.provide(
    Layer.setRequestCache(
      Request.makeCache({
        capacity: 256,
        timeToLive: "60 minutes"
      })
    )
  )
);
```

---

### No Memoization with Local Layer Provision (Effect-TS)

Source: https://effect.website/docs/requirements-management/layer-memoization

Illustrates that when layers are provided locally using `Effect.provide`, they do not support memoization by default. The example shows `ALive` being initialized twice because it's provided locally twice within the same program scope. This results in the log message appearing two times.

```typescript
import { Effect, Context, Layer } from "effect"

class A extends Context.Tag("A")() {}

const Alive = Layer.effect(
  A,
  Effect.succeed({ a: 5 }).pipe(
    Effect.tap(() => Effect.log("initialized"))
  )
)

const program = Effect.gen(function* () {
  yield* Effect.provide(A, Alive)
  yield* Effect.provide(A, Alive)
})

Effect.runPromise(program)
```

---

### Memoization When Providing Globally - Effect

Source: https://effect.website/docs/requirements-management/layer-memoization

Demonstrates how Effect layers are shared by default when provided globally. The example shows that even if a layer (ALive) is depended upon by multiple other layers (BLive, CLive), it is only initialized once, as evidenced by the single 'initialized' log message.

```typescript
import { Effect, Context, Layer } from "effect"

class A extends Context.Tag("A")() {}

class B extends Context.Tag("B")() {}

class C extends Context.Tag("C")() {}

const ALive = Layer.effect(
  A,
  Effect.succeed({ a: 5 }).pipe(
    Effect.tap(() => Effect.log("initialized"))
  )
)

const BLive = Layer.effect(
  B,
  Effect.gen(function* () {
    const { a } = yield* A
    return { b: String(a) }
  })
)

const CLive = Layer.effect(
  C,
  Effect.gen(function* () {
    const { a } = yield* A
    return { c: a > 0 }
  })
)

const program = Effect.gen(function* () {
  yield* B
  yield* C
})

const runnable = Effect.provide(
  program,
  Layer.merge(Layer.provide(BLive, ALive), Layer.provide(CLive, ALive))
)

Effect.runPromise(runnable)
/* 
Output:
timestamp=... level=INFO fiber=#2 message=initialized
*/
```

---

### Define and Use Cache Service with NodeFileSystem

Source: https://effect.website/docs/requirements-management/layers

Defines a Cache service that uses `NodeFileSystem` for reading files. It then shows how to access this service within a program to look up data and provides a mock file system for testing.

```typescript
import { FileSystem } from "@effect/platform"
import { NodeFileSystem } from "@effect/platform-node"
import { Effect, Console } from "effect"

// Define a Cache service
class Cache extends Effect.Service()("app/Cache", {
  effect: Effect.gen(function* () {
    const fs = yield* FileSystem.FileSystem
    const lookup = (key: string) => fs.readFileString(`cache/${key}`)
    return { lookup } as const
  }),
  dependencies: [NodeFileSystem.layer]
}) {}

// Accessing the Cache Service
const program = Effect.gen(function* () {
  const cache = yield* Cache
  const data = yield* cache.lookup("my-key")
  console.log(data)
}).pipe(Effect.catchAllCause((cause) => Console.log(cause)))

// Create a test file system that always returns a fixed value
const FileSystemTest = FileSystem.layerNoop({
  readFileString: () => Effect.succeed("File Content...")
})

const runnable = program.pipe(
  Effect.provide(Cache.DefaultWithoutDependencies),
  // Provide the mock file system
  Effect.provide(FileSystemTest)
)

Effect.runFork(runnable)
// Output: File Content...
```

---

### Create a Cache with Options

Source: https://effect.website/docs/caching/cache

Demonstrates the creation of a cache using the `make` function. This function requires options specifying the cache's `capacity` (maximum number of entries) and `timeToLive` (duration for entries to remain valid). It also takes the `lookup` function, which defines how to compute values for given keys.

```typescript
declare const make: <Key, Value, Requirements, Error>(
  options: {
    readonly capacity: number
    readonly timeToLive: Duration.DurationInput
    readonly lookup: Lookup<Key, Value, Requirements, Error>
  }
) => Effect<Cache<Key, Value, Requirements, Error>, never, Requirements | Scope>
```

---

### Effect Caching with Time-to-Live (TTL) in Effect.js

Source: https://effect.website/docs/caching/caching-effects

Explains how to use Effect.cachedWithTTL to cache an effect's result for a specific duration. After the TTL expires, the effect is recomputed on the next evaluation. This example shows caching for 150 milliseconds and recomputation after expiration.

```typescript
import { Effect, Console } from "effect"

let i = 1

// Simulating an expensive task with a delay
const expensiveTask = Effect.promise(() => {
  console.log("expensive task...")
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(`result ${i++}`)
    }, 100)
  })
})

const program = Effect.gen(function* () {
  // Caches the result for 150 milliseconds
  const cached = yield* Effect.cachedWithTTL(expensiveTask, "150 millis")

  // First evaluation triggers the task
  yield* cached.pipe(Effect.andThen(Console.log))

  // Second evaluation returns the cached result
  yield* cached.pipe(Effect.andThen(Console.log))

  // Wait for 100 milliseconds, ensuring the cache expires
  yield* Effect.sleep("100 millis")

  // Recomputes the task after cache expiration
  yield* cached.pipe(Effect.andThen(Console.log))
})

Effect.runFork(program)
```

---

### Manual Memoization with Layer.memoize (Effect-TS)

Source: https://effect.website/docs/requirements-management/layer-memoization

Shows how to manually memoize a layer using `Layer.memoize`. This function returns a scoped effect that, when evaluated, provides the lazily computed result of the layer. The example demonstrates that even when the memoized layer is provided multiple times locally, the underlying effect is only executed once, as evidenced by the single log message.

```typescript
import { Effect, Context, Layer } from "effect"

class A extends Context.Tag("A")() {}

const ALive = Layer.effect(
  A,
  Effect.succeed({ a: 5 }).pipe(
    Effect.tap(() => Effect.log("initialized"))
  )
)

const program = Effect.scoped(
  Layer.memoize(ALive).pipe(
    Effect.andThen((memoized) =>
      Effect.gen(function* () {
        yield* Effect.provide(A, memoized)
        yield* Effect.provide(A, memoized)
      })
    )
  )
)

Effect.runPromise(program)
```

---

### Mock Cache Service Directly

Source: https://effect.website/docs/requirements-management/layers

Demonstrates mocking the `Cache` service directly instead of its dependencies. A new instance of `Cache` is created with a custom `lookup` implementation that returns a hardcoded string.

```typescript
import { FileSystem } from "@effect/platform"
import { NodeFileSystem } from "@effect/platform-node"
import { Effect, Console } from "effect"

// Define a Cache service
class Cache extends Effect.Service()("app/Cache", {
  effect: Effect.gen(function* () {
    const fs = yield* FileSystem.FileSystem
    const lookup = (key: string) => fs.readFileString(`cache/${key}`)
    return { lookup } as const
  }),
  dependencies: [NodeFileSystem.layer]
}) {}

// Accessing the Cache Service
const program = Effect.gen(function* () {
  const cache = yield* Cache
  const data = yield* cache.lookup("my-key")
  console.log(data)
}).pipe(Effect.catchAllCause((cause) => Console.log(cause)))

// Create a mock implementation of Cache
const cache = new Cache({
  lookup: () => Effect.succeed("Cache Content...")
})

// Provide the mock Cache service
const runnable = program.pipe(Effect.provideService(Cache, cache))

Effect.runFork(runnable)
// Output: Cache Content...
```
