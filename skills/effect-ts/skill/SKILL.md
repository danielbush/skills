---
name: effect-ts
description: >
  Comprehensive guide for building TypeScript applications with Effect, 
  a powerful functional programming library. Use when: (1) Creating new 
  Effect projects, (2) Working with Effect's core types (Effect, Layer, 
  Context, Config), (3) Building pipelines and generators, (4) Implementing 
  services and dependency injection, (5) Adding observability (logging, 
  tracing, metrics), (6) Using Effect Platform for cross-platform operations, 
  (7) Integrating AI providers with @effect/ai, (8) Working with concurrency 
  (fibers, streams, pubsub), or (9) Any TypeScript project using the `effect` package.
---

# Effect TypeScript

Effect is a powerful TypeScript library for building type-safe, composable applications with excellent error handling, dependency injection, and concurrency support.

## Quick Start

### Create new project
```bash
npx create-effect-app@latest    # npm
pnpm create effect-app@latest   # pnpm
yarn create effect-app@latest   # yarn
bunx create-effect-app@latest   # bun
```

### Add to existing project
```bash
npm install effect    # npm
pnpm add effect       # pnpm
yarn add effect       # yarn
bun add effect        # bun
deno add npm:effect   # deno
```

## Core Concepts

### Effect<A, E, R>
The core type representing a computation that:
- Succeeds with value `A`
- Fails with error `E`  
- Requires dependencies `R`

```typescript
import { Effect } from "effect"

// Succeed with a value
const success = Effect.succeed(42)

// Fail with an error
const failure = Effect.fail(new Error("Something went wrong"))

// Wrap a Promise
const fromPromise = Effect.tryPromise(() => fetch("/api/data"))
```

### Running Effects

```typescript
// Sync (throws on error)
Effect.runSync(effect)

// Promise
Effect.runPromise(effect)

// With exit status
Effect.runPromiseExit(effect)

// Fire and forget
Effect.runFork(effect)
```

## Generator Pattern (Effect.gen)

The recommended way to compose Effects using generators:

```typescript
import { Effect } from "effect"

const program = Effect.gen(function* () {
  const a = yield* Effect.succeed(1)
  const b = yield* Effect.succeed(2)
  return a + b
})
```

With error handling:
```typescript
const program = Effect.gen(function* () {
  const config = yield* loadConfig()
  const db = yield* connectDatabase(config)
  const users = yield* db.query("SELECT * FROM users")
  return users
})
```

## Pipe Pattern

Compose operations using `pipe`:

```typescript
import { Effect, pipe } from "effect"

const result = pipe(
  Effect.succeed(5),
  Effect.map((n) => n * 2),
  Effect.flatMap((n) => Effect.succeed(n + 1))
)
```

## Error Handling

```typescript
// Catch all errors
Effect.catchAll(effect, (error) => Effect.succeed("fallback"))

// Catch specific errors
Effect.catchTag(effect, "NotFound", () => Effect.succeed(null))

// Map errors
Effect.mapError(effect, (e) => new CustomError(e))

// Provide fallback
Effect.orElse(effect, () => fallbackEffect)
```

## Dependency Injection with Layer

### Define a Service
```typescript
import { Effect, Context, Layer } from "effect"

class Database extends Context.Tag("Database")<
  Database,
  { query: (sql: string) => Effect.Effect<unknown[]> }
>() {}
```

### Create a Layer
```typescript
const DatabaseLive = Layer.succeed(Database, {
  query: (sql) => Effect.succeed([{ id: 1 }])
})
```

### Use the Service
```typescript
const program = Effect.gen(function* () {
  const db = yield* Database
  return yield* db.query("SELECT * FROM users")
})

// Provide the layer
Effect.runPromise(Effect.provide(program, DatabaseLive))
```

### Effect.Service (simplified pattern)
```typescript
class Cache extends Effect.Service<Cache>()("app/Cache", {
  effect: Effect.gen(function* () {
    const fs = yield* FileSystem.FileSystem
    return {
      lookup: (key: string) => fs.readFileString(`cache/${key}`)
    }
  }),
  dependencies: [NodeFileSystem.layer]
}) {}
```

## Configuration

```typescript
import { Effect, Config } from "effect"

const program = Effect.gen(function* () {
  const host = yield* Config.string("HOST")
  const port = yield* Config.number("PORT")
  const apiKey = yield* Config.redacted("API_KEY") // Sensitive
  return { host, port, apiKey }
})

// With defaults
const port = Config.number("PORT").pipe(Config.withDefault(8080))
```

## Packages Reference

| Package | Use Case |
|---------|----------|
| `effect` | Core library |
| `@effect/platform` | Cross-platform abstractions (HTTP, FileSystem, Terminal) |
| `@effect/platform-node` | Node.js implementations |
| `@effect/platform-bun` | Bun implementations |
| `@effect/schema` | Data validation and transformation |
| `@effect/ai` | AI/LLM abstractions |
| `@effect/ai-openai` | OpenAI provider |
| `@effect/ai-anthropic` | Anthropic provider |
| `@effect/opentelemetry` | Tracing and metrics |

## Platform: FileSystem

```typescript
import { FileSystem } from "@effect/platform"
import { NodeContext, NodeRuntime } from "@effect/platform-node"
import { Effect } from "effect"

const program = Effect.gen(function* () {
  const fs = yield* FileSystem.FileSystem
  const content = yield* fs.readFileString("./file.txt", "utf8")
  yield* fs.writeFileString("./output.txt", content.toUpperCase())
})

NodeRuntime.runMain(program.pipe(Effect.provide(NodeContext.layer)))
```

## Platform: HTTP Client

```typescript
import { HttpClient } from "@effect/platform"
import { NodeRuntime } from "@effect/platform-node"
import { Effect } from "effect"

const program = Effect.gen(function* () {
  const client = yield* HttpClient.HttpClient
  const response = yield* client.get("https://api.example.com/data")
  return yield* response.json
})
```

## Platform: Command

```typescript
import { Command } from "@effect/platform"
import { NodeContext, NodeRuntime } from "@effect/platform-node"
import { Effect } from "effect"

const program = Effect.gen(function* () {
  const output = yield* Command.string(Command.make("ls", "-la"))
  console.log(output)
})

NodeRuntime.runMain(program.pipe(Effect.provide(NodeContext.layer)))
```

## AI Integration

```typescript
import { OpenAiClient, OpenAiLanguageModel } from "@effect/ai-openai"
import { LanguageModel } from "@effect/ai"
import { NodeHttpClient } from "@effect/platform-node"
import { Config, Effect, Layer } from "effect"

const generateText = Effect.gen(function* () {
  const response = yield* LanguageModel.generateText({
    prompt: "Generate a greeting"
  })
  return response.text
})

const Gpt4o = OpenAiLanguageModel.model("gpt-4o")

const OpenAi = OpenAiClient.layerConfig({
  apiKey: Config.redacted("OPENAI_API_KEY")
}).pipe(Layer.provide(NodeHttpClient.layerUndici))

const main = generateText.pipe(
  Effect.provide(Gpt4o),
  Effect.provide(OpenAi)
)
```

## Observability: Logging

```typescript
import { Effect } from "effect"

Effect.gen(function* () {
  yield* Effect.log("Info message")
  yield* Effect.logDebug("Debug message")
  yield* Effect.logWarning("Warning message")
  yield* Effect.logError("Error message")
})

// With annotations
Effect.log("Processing").pipe(
  Effect.annotateLogs({ userId: "123", action: "login" })
)

// With spans (timing)
Effect.gen(function* () {
  yield* Effect.sleep("1 second")
  yield* Effect.log("Done")
}).pipe(Effect.withLogSpan("operation"))
```

## Concurrency: Fibers

```typescript
import { Effect, Fiber } from "effect"

const program = Effect.gen(function* () {
  // Fork a fiber (runs concurrently)
  const fiber = yield* Effect.fork(longRunningTask)
  
  // Do other work...
  yield* otherWork
  
  // Wait for fiber result
  const result = yield* Fiber.join(fiber)
  return result
})
```

## Concurrency: Parallel Execution

```typescript
import { Effect } from "effect"

// Run effects in parallel
const results = yield* Effect.all([task1, task2, task3], {
  concurrency: "unbounded"
})

// With limited concurrency
const results = yield* Effect.all(tasks, { concurrency: 5 })
```

## Concurrency: Streams

```typescript
import { Stream, Effect } from "effect"

const stream = Stream.make(1, 2, 3).pipe(
  Stream.map((n) => n * 2),
  Stream.filter((n) => n > 2)
)

const result = yield* Stream.runCollect(stream)
```

## Schema (Validation)

```typescript
import { Schema } from "effect"

const User = Schema.Struct({
  id: Schema.Number,
  name: Schema.String.pipe(Schema.minLength(1)),
  email: Schema.String
})

type User = Schema.Schema.Type<typeof User>

// Decode unknown data
const user = Schema.decodeUnknownSync(User)({
  id: 1,
  name: "Alice",
  email: "alice@example.com"
})
```

## Reference Documentation

For detailed examples and API documentation, search `references/effect-docs.txt`:

| Topic | Search Pattern |
|-------|---------------|
| Layers & DI | `Layer` |
| File operations | `FileSystem` |
| HTTP client | `HttpClient` |
| AI integration | `@effect/ai` |
| OpenTelemetry | `OpenTelemetry` |
| Streams | `Stream.make` |
| PubSub | `PubSub` |
| Scheduling | `Schedule` |
| Metrics | `Metric` |
| Terminal | `Terminal` |
| Commands | `Command.make` |
