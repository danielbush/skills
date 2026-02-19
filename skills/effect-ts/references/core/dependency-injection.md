# Effect – Dependency Injection

`Context.Tag`, `Effect.Service`, `Layer`, service composition, memoization, and providing dependencies.

---

### Recovering from Layer Construction Errors with Layer.catchAll (TypeScript)

Source: https://effect.website/docs/requirements-management/layers

This example shows how to use Layer.catchAll to recover from errors during layer construction. If the initial server layer fails (e.g., due to a missing configuration), it provides a fallback layer that logs the error and starts a server on a default port.

```typescript
import { Config, Context, Effect, Layer } from "effect"

class HTTPServer extends Context.Tag("HTTPServer")() {}

// Simulating an HTTP server
const server = Layer.effect(
  HTTPServer,
  Effect.gen(function* () {
    const host = yield* Config.string("HOST")
    console.log(`Listening on http://localhost:${host}`)
  })
).pipe(
  // Recover from errors during layer construction
  Layer.catchAll((configError) =>
    Layer.effect(
      HTTPServer,
      Effect.gen(function* () {
        console.log(`Recovering from error:\n${configError}`)
        console.log(`Listening on http://localhost:3000`)
      })
    )
  )
)

Effect.runFork(Layer.launch(server))
```

---

### Compose and Provide Layers (Effect.js)

Source: https://effect.website/docs/requirements-management/layers

Demonstrates composing multiple layers using Layer.merge and Layer.provideMerge, then providing the composed layer to an effect. This allows for flexible dependency management and application setup.

```typescript
import { Effect, Context, Layer } from "effect"

// Assuming Config, Logger, and Database layers are defined as above

const ConfigLive = Layer.succeed(class Config extends Context.Tag("Config") {}, {
  getConfig: Effect.succeed({
    logLevel: "INFO",
    connection: "mysql://username:password@hostname:port/database_name"
  })
})

class Logger extends Context.Tag("Logger")<Logger, { readonly log: (message: string) => Effect.Effect }>() {}
const LoggerLive = Layer.effect(Logger, Effect.gen(function* () {
  const config = yield* Config
  return {
    log: (message) => Effect.gen(function* () {
      const { logLevel } = yield* config.getConfig
      console.log(`[${logLevel}] ${message}`)
    })
  }
}))

class Database extends Context.Tag("Database")<Database, { readonly query: (sql: string) => Effect.Effect }>() {}
const DatabaseLive = Layer.effect(Database, Effect.gen(function* () {
  const config = yield* Config
  const logger = yield* Logger
  return {
    query: (sql: string) => Effect.gen(function* () {
      yield* logger.log(`Executing query: ${sql}`)
      const { connection } = yield* config.getConfig
      return { result: `Results from ${connection}` }
    })
  }
}))

const AppConfigLive = Layer.merge(ConfigLive, LoggerLive)

const MainLive = DatabaseLive.pipe(
  Layer.provide(AppConfigLive),
  Layer.provideMerge(ConfigLive)
)

// Example of providing the MainLive layer to an effect
const program = Effect.gen(function* () {
  const db = yield* Database
  return yield* db.query("SELECT * FROM users")
})

Effect.runPromise(Effect.provide(program, MainLive))
```

---

### Accessing and Running a Service with Effect.Service

Source: https://effect.website/docs/requirements-management/layers

Demonstrates how to access a service defined with Effect.Service within an Effect program and run it with provided dependencies. This example shows accessing the Cache service and handling potential errors during file lookup.

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

const runnable = program.pipe(Effect.provide(Cache.Default))

Effect.runFork(runnable)
```

---

### Running Effect-TS Notifications Service Example

Source: https://effect.website/docs/runtime

This snippet demonstrates how to set up an Effect-TS runtime with the live Notifications service and execute an effect that uses it. It shows the lifecycle management of the runtime, including creation, running a promise-based effect, and disposal.

```typescript
// Example entry point for an external framework
async function main() {
  // Create a custom runtime using the Notifications layer
  const runtime = ManagedRuntime.make(NotificationsLive.Live)

  // Run the effect
  await runtime.runPromise(Notifications.notify("Hello, world!"))

  // Dispose of the runtime, cleaning up resources
  await runtime.dispose()
}

```

---

### Simulating Database Connections with Effect Layers

Source: https://effect.website/docs/requirements-management/layers

Demonstrates how to create Effect layers to simulate database connections, including a PostgreSQL connection and an in-memory fallback. This approach allows for flexible dependency management and error handling.

```typescript
import { Effect, Layer, Config, Context } from "effect"

class Database extends Context.Tag("Database")() {}

// Simulating a database connection
const postgresDatabaseLayer = Layer.effect(
  Database,
  Effect.gen(function* () {
    const databaseConnectionString = yield* Config.string(
      "CONNECTION_STRING"
    )
    console.log(
      `Connecting to database with: ${databaseConnectionString}`
    )
  })
)

// Simulating an in-memory database connection
const inMemoryDatabaseLayer = Layer.effect(
  Database,
  Effect.gen(function* () {
    console.log(`Connecting to in-memory database`)
  })
)

// Fallback to in-memory database if PostgreSQL connection fails
const database = postgresDatabaseLayer.pipe(
  Layer.orElse(() => inMemoryDatabaseLayer)
)

Effect.runFork(Layer.launch(database))
```

---

### Use Effect.js Ref as a Service

Source: https://effect.website/docs/state-management/ref

This example demonstrates how to use Effect.js Ref as a service, allowing state to be shared across different parts of a program. It defines a `MyState` tag, creates subprograms that interact with the state via the service, and composes them into a main program. `Effect.provideServiceEffect` is used to provide the effectful Ref implementation.

```typescript
import { Effect, Context, Ref } from "effect"

// Create a Tag for our state
class MyState extends Context.Tag("MyState")<MyState, Ref.Ref<number>>() {}

// Subprogram 1: Increment the state value twice
const subprogram1 = Effect.gen(function* () {
  const state = yield* MyState
  yield* Ref.update(state, (n) => n + 1)
  yield* Ref.update(state, (n) => n + 1)
})

// Subprogram 2: Decrement the state value and then increment it
const subprogram2 = Effect.gen(function* () {
  const state = yield* MyState
  yield* Ref.update(state, (n) => n - 1)
  yield* Ref.update(state, (n) => n + 1)
})

// Subprogram 3: Read and log the current value of the state
const subprogram3 = Effect.gen(function* () {
  const state = yield* MyState
  const value = yield* Ref.get(state)
  console.log(`MyState has a value of ${value}.`)
})

// Compose subprograms 1, 2, and 3 to create the main program
const program = Effect.gen(function* () {
  yield* subprogram1
  yield* subprogram2
  yield* subprogram3
})

// Create a Ref instance with an initial value of 0
const initialState = Ref.make(0)

// Provide the Ref as a service
const runnable = program.pipe(
  Effect.provideServiceEffect(MyState, initialState)
)

// Run the program and observe the output
Effect.runPromise(runnable)
/* 
Output:
MyState has a value of 2.
*/
```

---

### Manual Service Passing Example in TypeScript

Source: https://effect.website/docs/requirements-management/services

Demonstrates the traditional approach of manually passing a service object to a function. This method becomes unmanageable in larger applications due to the need to pass services through multiple layers.

```typescript
const processData = (data: Data, databaseService: DatabaseService) => {
  // Operations using the database service
}
```

---

### Define Service with Synchronous Constructor (Effect.js)

Source: https://effect.website/docs/requirements-management/layers

Shows how to define a service using a synchronous constructor with `Effect.Service`. This is useful for services that can be instantiated immediately without asynchronous operations.

```typescript
import { Effect, Random } from "effect"

class Sync extends Effect.Service()("Sync", {
  sync: () => ({
    next: Random.nextInt
  })
}) {}

// Accessing the Service
const program = Effect.gen(function* () {
  const sync = yield* Sync
  const n = yield* sync.next
  console.log(`The number is ${n}`)
})

Effect.runPromise(program.pipe(Effect.provide(Sync.Default)))
// Example Output: The number is 3858843290019673
```

---

### Create Fresh Layer Instance with Layer.fresh (Effect-TS)

Source: https://effect.website/docs/requirements-management/layer-memoization

Demonstrates how to create a fresh, non-shared instance of a layer using `Layer.fresh`. This is useful when you don't want to share a module across different parts of your application. The example shows how `ALive` is initialized twice when provided locally to `BLive` and `CLive` respectively.

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
  Layer.merge(
    Layer.provide(BLive, Layer.fresh(ALive)),
    Layer.provide(CLive, Layer.fresh(ALive))
  )
)

Effect.runPromise(runnable)
```

---

### Provide Random Service Implementation - Effect.js

Source: https://effect.website/docs/requirements-management/services

Demonstrates providing a custom implementation for the `Random` service to a program. The `Random` service is defined with a `next` operation that generates a random number. This example shows how to use `Effect.provideService` to inject this custom service implementation.

```typescript
import * as Effect from "effect/Effect";
import * as Random from "effect/Random";

const program = Effect.gen(function* (_) {
  const random = yield* Random.Random;
  return yield* random.next;
});

Effect.runPromise(Effect.provideService(program, Random, {
  next: Effect.sync(() => Math.random())
})).then(console.log);
// Example Output: 0.9957979486841035
```

---

### Provide and Use Custom Service in Effect-TS

Source: https://effect.website/docs/micro/new-users

Demonstrates how to provide an implementation for a custom service and use it within an Effect-TS program. The `Micro.provideService` function is used to associate a concrete implementation (e.g., an object with a `next` method returning `Math.random()`) with the `Random` service tag. The program can then be executed.

```typescript
import { Micro, Context } from "effect"

// Declaring a tag for a service that generates random numbers
class Random extends Context.Tag("MyRandomService")< 
Random,
 {
  readonly next: Micro.Micro
 }
>() {}

// Using the service
const program = Micro.gen(function* () {
  // Access the Random service
  const random = yield* Micro.service(Random)

  // Retrieve a random number from the service
  const randomNumber = yield* random.next

  console.log(`random number: ${randomNumber}`)
})

// Providing the implementation
// 
// // 
// // ┌─── Micro
// // ▼
const runnable = Micro.provideService(program, Random, {
  next: Micro.sync(() => Math.random())
})

Micro.runPromise(runnable)
/* 
Example Output:
random number: 0.8241872233134417 
*/
```

---

### Executing Database Queries with Effect and Layers (TypeScript)

Source: https://effect.website/docs/requirements-management/layers

This snippet shows how to define a query function using Effect.gen, which logs the SQL query and retrieves connection details from configuration. It then defines live implementations for configuration and logging, and a main live layer that provides these services. Finally, it constructs and runs a program that queries the database.

```typescript
const query = (sql: string) =>
  Effect.gen(function* () {
    yield* logger.log(`Executing query: ${sql}`)
    const { connection } = yield* config.getConfig
    return { result: `Results from ${connection}` }
  })

const AppConfigLive = Layer.merge(ConfigLive, LoggerLive)

const MainLive = DatabaseLive.pipe(
  Layer.provide(AppConfigLive),
  Layer.provide(ConfigLive)
)

const program = Effect.gen(function* () {
  const database = yield* Database
  const result = yield* database.query("SELECT * FROM users")
  return result
})

const runnable = Effect.provide(program, MainLive)

Effect.runPromise(runnable).then(console.log)
```

---

### Direct Method Access with Effect.Service

Source: https://effect.website/docs/requirements-management/layers

Demonstrates how to define a service using Effect.Service and access its methods directly without explicit service extraction. This method simplifies code by enabling direct calls like `Sync.next`. It requires a default implementation provided via `Effect.provide`.

```typescript
import { Effect, Random } from "effect"

class Sync extends Effect.Service("Sync", {
  sync: () => ({
    next: Random.nextInt
  }),
  accessors: true // Enables direct method access via the tag
}) {}

const program = Effect.gen(function* () {
  // const sync = yield* Sync
  // const n = yield* sync.next
  const n = yield* Sync.next // No need to extract the service first
  console.log(`The number is ${n}`)
})

Effect.runPromise(program.pipe(Effect.provide(Sync.Default)))
// Example Output: The number is 3858843290019673
```

---

### Using a Service within an Effect with EffectJS

Source: https://effect.website/docs/requirements-management/services

Demonstrates how to use a previously declared service (Random Number Generator) within an EffectJS program. This example utilizes Effect.gen and pipe for composing effects and accessing service methods.

```typescript
import { Effect, Context } from "effect"

// Declaring a tag for a service that generates random numbers
class Random extends Context.Tag("MyRandomService")< 
Random, 
{
  readonly next: Effect.Effect<number>
}
>() {}

// Example of using the Random service
const randomNumberEffect = Effect.gen(function* (_) {
  const random = yield* Random
  const number = yield* random.next
  return number
})

// Alternatively using pipe
const randomNumberEffectWithPipe = pipe(
  Random,
  Context.flatMap(random => random.next)
)
```

---

### Define Service with Static Implementation (Effect.js)

Source: https://effect.website/docs/requirements-management/layers

Illustrates defining a service using a static implementation with `Effect.Service`. This method is suitable for providing a constant value as a service.

```typescript
import { Effect } from "effect"

class MagicNumber extends Effect.Service()("MagicNumber", {
  succeed: { value: 42 }
}) {}

// Accessing the Service
const program = Effect.gen(function* () {
  const magicNumber = yield* MagicNumber
  console.log(`The magic number is ${magicNumber.value}`)
})

Effect.runPromise(program.pipe(Effect.provide(MagicNumber.Default)))
// The magic number is 42
```

---

### Provide Random Service Implementation

Source: https://effect.website/docs/requirements-management/services

Demonstrates how to provide an actual implementation for a declared service (Random) using Effect.provideService. This allows the effect to be run successfully, as the required service is now available.

```typescript
import { Effect, Context } from "effect"

// Declaring a tag for a service that generates random numbers
class Random extends Context.Tag("MyRandomService")<Random, {
  readonly next: Effect.Effect
}>() {}

// Using the service
const program = Effect.gen(function* () {
  const random = yield* Random
  const randomNumber = yield* random.next
  console.log(`random number: ${randomNumber}`)
})

// Providing the implementation
const runnable = Effect.provideService(program, Random, {
  next: Effect.sync(() => Math.random())
})

// Run successfully
Effect.runPromise(runnable)
```

---

### Combining and Providing Multiple Services with Effect-TS

Source: https://effect.website/docs/requirements-management/services

Illustrates how to combine multiple service implementations into a single Context object using Context.empty and Context.add, and then provide this combined context to an effect using Effect.provide.

```typescript
import { Effect, Context } from "effect"

class Random extends Context.Tag("MyRandomService")<Random, {
  readonly next: Effect.Effect
}>() {}

class Logger extends Context.Tag("MyLoggerService")<Logger, {
  readonly log: (message: string) => Effect.Effect
}>() {}

const program = Effect.gen(function* () {
  const random = yield* Random
  const logger = yield* Logger
  const randomNumber = yield* random.next
  return yield* logger.log(String(randomNumber))
})

// Combine service implementations into a single 'Context'
const context = Context.empty().pipe(
  Context.add(Random, { next: Effect.sync(() => Math.random()) }),
  Context.add(Logger, {
    log: (message) => Effect.sync(() => console.log(message))
  })
)

// Provide the entire context
const runnable = Effect.provide(program, context)
```

---

### Providing Bun Context for Effect Platform

Source: https://effect.website/docs/platform/introduction

Adapts the Effect Platform program to run within a Bun environment by providing the Bun-specific context. This allows the program to leverage Bun APIs.

```typescript
1 import { Path } from "@effect/platform"
2 import { Effect } from "effect"
3 import { BunContext, BunRuntime } from "@effect/platform-bun"
4 
5 const program = Effect.gen(function* () {
6 // Access the Path service
7 const path = yield* Path.Path
8 
9 // Join parts of a path to create a complete file path
10 const mypath = path.join("tmp", "file.txt")
11 
12 console.log(mypath)
13 })
14 
15 BunRuntime.runMain(program.pipe(Effect.provide(BunContext.layer)))

```

---

### Define Custom Service Tag with Context.Tag in Effect-TS

Source: https://effect.website/docs/micro/new-users

Illustrates how to declare a service tag using `Context.Tag` in Effect-TS. A service tag defines a unique identifier and the interface for a service. This example defines a `Random` service tag with a `next` method that returns a `Micro` effect.

```typescript
import { Micro, Context } from "effect"

// Declaring a tag for a service that generates random numbers
class Random extends Context.Tag("MyRandomService")<
Random,
{
  readonly next: Micro.Micro
}
>() {}
```

---

### Tapping Layer Acquisition Success and Failure (TypeScript)

Source: https://effect.website/docs/requirements-management/layers

This snippet illustrates using Layer.tap and Layer.tapError to perform side effects during layer acquisition. It logs a success message with the context upon successful acquisition and an error message if acquisition fails, without altering the layer's signature.

```typescript
import { Config, Context, Effect, Layer, Console } from "effect"

class HTTPServer extends Context.Tag("HTTPServer")() {}

// Simulating an HTTP server
const server = Layer.effect(
  HTTPServer,
  Effect.gen(function* () {
    const host = yield* Config.string("HOST")
    console.log(`Listening on http://localhost:${host}`)
  })
).pipe(
  // Log a message if the layer acquisition succeeds
  Layer.tap((ctx) =>
    Console.log(`layer acquisition succeeded with:\n${ctx}`)
  ),
  // Log a message if the layer acquisition fails
  Layer.tapError((err) =>
    Console.log(`layer acquisition failed with:\n${err}`)
  )
)

Effect.runFork(Layer.launch(server))
```

---

### Providing Multiple Services to an Effect

Source: https://effect.website/docs/requirements-management/services

Shows how to provide implementations for multiple services (Random and Logger) to an effect using chained Effect.provideService calls. Each service is provided individually.

```typescript
import { Effect, Context } from "effect"

class Random extends Context.Tag("MyRandomService")<Random, {
  readonly next: Effect.Effect
}>() {}

class Logger extends Context.Tag("MyLoggerService")<Logger, {
  readonly log: (message: string) => Effect.Effect
}>() {}

const program = Effect.gen(function* () {
  const random = yield* Random
  const logger = yield* Logger
  const randomNumber = yield* random.next
  return yield* logger.log(String(randomNumber))
})

// Provide service implementations for 'Random' and 'Logger'
const runnable = program.pipe(
  Effect.provideService(Random, {
    next: Effect.sync(() => Math.random())
  }),
  Effect.provideService(Logger, {
    log: (message) => Effect.sync(() => console.log(message))
  })
)
```
