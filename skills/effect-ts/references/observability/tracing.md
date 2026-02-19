# Effect – Tracing & OpenTelemetry

Spans, OTLP export, distributed tracing, and DevTools integration.

---

### Setting Up and Printing a Span (Effect/TypeScript)

Source: https://effect.website/docs/observability/tracing

This example demonstrates setting up OpenTelemetry tracing and printing spans to the console. It configures `NodeSdk` to use `ConsoleSpanExporter` and `BatchSpanProcessor`, then runs an instrumented effect, providing the tracing layer.

```typescript
import { Effect } from "effect"
import { NodeSdk } from "@effect/opentelemetry"
import {
 ConsoleSpanExporter,
 BatchSpanProcessor
} from "@opentelemetry/sdk-trace-base"

// Define an effect that delays for 100 milliseconds
const program = Effect.void.pipe(Effect.delay("100 millis"))

// Instrument the effect with a span for tracing
const instrumented = program.pipe(Effect.withSpan("myspan"))

// Set up tracing with the OpenTelemetry SDK
const NodeSdkLive = NodeSdk.layer(() => ({
 resource: { serviceName: "example" },
 // Export span data to the console
 spanProcessor: new BatchSpanProcessor(new ConsoleSpanExporter())
}))

// Run the effect, providing the tracing layer
Effect.runPromise(instrumented.pipe(Effect.provide(NodeSdkLive)))
```

---

### Effect Tracing in Distributed Systems

Source: https://context7_llms

Introduction to tracing in Effect for distributed systems. It uses spans and traces to track request lifecycles across multiple services for debugging and performance analysis.

```markdown
Effect's tracing capabilities help visualize the flow of requests through complex distributed architectures, identifying bottlenecks and errors.
```

---

### Export Spans to Console with Effect.fn and OpenTelemetry (Effect.js)

Source: https://effect.website/docs/error-management/expected-errors

Configures Effect.js to export traced spans to the console using OpenTelemetry. This example sets up a NodeSdk with a ConsoleSpanExporter and BatchSpanProcessor, allowing visibility into function execution, metadata, and errors.

```typescript
import { Effect } from "effect"
import { NodeSdk } from "@effect/opentelemetry"
import {
  ConsoleSpanExporter,
  BatchSpanProcessor
} from "@opentelemetry/sdk-trace-base"

const myfunc = Effect.fn("myspan")(function* (n: number) {
  yield* Effect.annotateCurrentSpan("n", n)
  console.log(`got: ${n}`)
  yield* Effect.fail(new Error("Boom!"))
})

const program = myfunc(100)

const NodeSdkLive = NodeSdk.layer(() => ({
  resource: { serviceName: "example" },
  // Export span data to the console
  spanProcessor: new BatchSpanProcessor(new ConsoleSpanExporter())
}))

// Effect.runFork(program.pipe(Effect.provide(NodeSdkLive)))
// Note: The above line is commented out as it's an incomplete snippet for demonstration.
```

---

### Nesting Spans for Hierarchical Operations with Effect.js and OpenTelemetry

Source: https://effect.website/docs/observability/tracing

This example illustrates how to create nested spans using Effect.js and OpenTelemetry to represent hierarchical operations. It defines a 'child' effect with its own span and a 'parent' effect that includes the 'child' effect and additional delays, showcasing the parent-child relationship in the trace output.

```typescript
import { Effect } from "effect"
import { NodeSdk } from "@effect/opentelemetry"
import {
  ConsoleSpanExporter,
  BatchSpanProcessor
} from "@opentelemetry/sdk-trace-base"

const child = Effect.void.pipe(
  Effect.delay("100 millis"),
  Effect.withSpan("child")
)

const parent = Effect.gen(function* () {
  yield* Effect.sleep("20 millis")
  yield* child
  yield* Effect.sleep("10 millis")
}).pipe(Effect.withSpan("parent"))

// Set up tracing with the OpenTelemetry SDK
const NodeSdkLive = NodeSdk.layer(() => ({
  resource: { serviceName: "example" },
  spanProcessor: new BatchSpanProcessor(new ConsoleSpanExporter())
}))

// Run the effect, providing the tracing layer
Effect.runPromise(parent.pipe(Effect.provide(NodeSdkLive)))
```

---

### Export Traces via OTLP to OpenTelemetry Backend (TypeScript)

Source: https://effect.website/docs/observability/tracing

This snippet demonstrates how to configure Effect-TS to export application traces to an OpenTelemetry backend using the OTLP format over HTTP. It utilizes NodeSdk, BatchSpanProcessor, and OTLPTraceExporter. The program defines a recursive task structure with logging and sleep operations, then runs it with the OpenTelemetry SDK layer provided.

```typescript
import { Effect } from "effect"
import { NodeSdk } from "@effect/opentelemetry"
import { BatchSpanProcessor } from "@opentelemetry/sdk-trace-base"
import { OTLPTraceExporter } from "@opentelemetry/exporter-trace-otlp-http"

// Function to simulate a task with possible subtasks
const task = (
  name: string,
  delay: number,
  children: ReadonlyArray<Effect.Effect<any, any, any>> = []
) =>
  Effect.gen(function* () {
    yield* Effect.log(name)
    yield* Effect.sleep(`${delay} millis`)
    for (const child of children) {
      yield* child
    }
    yield* Effect.sleep(`${delay} millis`)
  }).pipe(Effect.withSpan(name))

const poll = task("/poll", 1)

// Create a program with tasks and subtasks
const program = task("client", 2, [
  task("/api", 3, [
    task("/authN", 4, [task("/authZ", 5)]),
    task("/payment Gateway", 6, [
      task("DB", 7),
      task("Ext. Merchant", 8)
    ]),
    task("/dispatch", 9, [
      task("/dispatch/search", 10),
      Effect.all([poll, poll, poll], { concurrency: "inherit" }),
      task("/pollDriver/{id}", 11)
    ])
  ])
])

const NodeSdkLive = NodeSdk.layer(() => ({
  resource: { serviceName: "example" },
  spanProcessor: new BatchSpanProcessor(new OTLPTraceExporter())
}))

Effect.runPromise(
  program.pipe(
    Effect.provide(NodeSdkLive),
    Effect.catchAllCause(Effect.logError)
  )
)
```
