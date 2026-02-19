# Effect – Metrics

Counters, gauges, frequency, summaries, and metric tagging.

---

### Create and Use Summary for Response Times (TypeScript)

Source: https://effect.website/docs/observability/metrics

This snippet demonstrates how to create a Summary metric in TypeScript to track response times. It configures the summary with a maximum age, size, error margin, and specific quantiles. The example then applies this summary to an effect that simulates response times and logs the resulting state.

```typescript
import { Metric, Random, Effect } from "effect"

// Define the summary for response times
const responseTimeSummary = Metric.summary({
  name: "response_time_summary", // Name of the summary metric
  maxAge: "1 day", // Maximum sample age
  maxSize: 100, // Maximum number of samples to retain
  error: 0.03, // Error margin for quantile calculation
  quantiles: [0.1, 0.5, 0.9], // Quantiles to observe (10%, 50%, 90%)
  // Optional
  description: "Measures the distribution of response times"
})

const program = Effect.gen(function* () {
  // Record 100 random response times between 1 and 120 ms
  yield* responseTimeSummary(Random.nextIntBetween(1, 120)).pipe(
    Effect.repeatN(99)
  )

  // Retrieve and log the current state of the summary
  const state = yield* Metric.value(responseTimeSummary)
  console.log("%o", state)
})

Effect.runPromise(program)
```

---

### Effect Metrics: Counters

Source: https://context7_llms

Example of using counters for tracking metrics in Effect applications. Counters are suitable for measuring the frequency of events.

```typescript
import * as Metric from "@effect/io/Metric"
import * as Effect from "@effect/io/Effect"

const requestCounter = Metric.counter("http_requests_total").withLabels({"path": "/"})

const incrementRequest = requestCounter.increment

Effect.runSync(incrementRequest)
// The counter 'http_requests_total' would now be incremented.
```

---

### Tagging Multiple Metrics with Effect.js

Source: https://effect.website/docs/observability/metrics

This example shows how to apply tags to multiple metrics within the same context using `Effect.tagMetrics`. This approach is beneficial for applying common tags, like the application environment, across several metrics simultaneously. The `environment: production` tag is applied to both `counter1` and `counter2`.

```typescript
import { Metric, Effect } from "effect"

// Create two separate counters
const counter1 = Metric.counter("counter1")
const counter2 = Metric.counter("counter2")

// Define a task that simulates some work with a slight delay
const task = Effect.succeed(1).pipe(Effect.delay("100 millis"))

// Apply the environment tag to both counters in the same context
Effect.gen(function* () {
  yield* counter1(task)
  yield* counter2(task)
}).pipe(Effect.tagMetrics("environment", "production"))
```

---

### Create and Use a Counter with Effect.js Ref

Source: https://effect.website/docs/state-management/ref

This snippet shows how to create a simple Counter class using Effect.js Ref. It demonstrates initializing a Ref, defining methods to increment, decrement, and get the counter's value, and running a program that manipulates the counter. The `Ref.make` and `Ref.update` operations are effectful.

```typescript
import { Effect, Ref } from "effect"

class Counter {
  readonly inc: Effect.Effect<void, never, never>
  readonly dec: Effect.Effect<void, never, never>
  readonly get: Effect.Effect<number, never, never>

  constructor(private value: Ref.Ref<number>) {
    this.inc = Ref.update(this.value, (n) => n + 1)
    this.dec = Ref.update(this.value, (n) => n - 1)
    this.get = Ref.get(this.value)
  }
}

const make = Effect.andThen(Ref.make(0), (value) => new Counter(value))

const program = Effect.gen(function* () {
  const counter = yield* make
  yield* counter.inc
  yield* counter.inc
  yield* counter.dec
  yield* counter.inc
  const value = yield* counter.get
  console.log(`This counter has a value of ${value}.`)
})

Effect.runPromise(program)
/* 
Output:
This counter has a value of 2.
*/
```

---

### Track Error Occurrences with Frequency (TypeScript)

Source: https://effect.website/docs/observability/metrics

This TypeScript example shows how to create and use a Frequency metric to count the occurrences of different error codes. It defines a frequency metric, simulates random errors using an effect, and repeats this process to populate the frequency count. Finally, it retrieves and logs the state of the frequency metric.

```typescript
import { Metric, Random, Effect } from "effect"

// Define a frequency metric to track errors
const errorFrequency = Metric.frequency("error_frequency", {
  // Optional
  description: "Counts the occurrences of errors."
})

const task = Effect.gen(function* () {
  const n = yield* Random.nextIntBetween(1, 10)
  return `Error-${n}`
})

// Program that simulates random errors and tracks their occurrences
const program = Effect.gen(function* () {
  yield* errorFrequency(task).pipe(Effect.repeatN(99))

  // Retrieve and log the current state of the summary
  const state = yield* Metric.value(errorFrequency)
  console.log("%o", state)
})

Effect.runPromise(program)
```

---

### Concurrent Updates to a Shared Counter with Effect.js Ref

Source: https://effect.website/docs/state-management/ref

This example demonstrates how to manage concurrent updates to a shared counter using Effect.js. It utilizes `Effect.zip` with the `concurrent: true` option to run multiple operations on the counter simultaneously, showcasing how Ref handles state in concurrent scenarios. The output shows the interleaved execution and the final state of the counter.

```typescript
import { Effect, Ref } from "effect"

class Counter {
  readonly inc: Effect.Effect<void, never, never>
  readonly dec: Effect.Effect<void, never, never>
  readonly get: Effect.Effect<number, never, never>

  constructor(private value: Ref.Ref<number>) {
    this.inc = Ref.update(this.value, (n) => n + 1)
    this.dec = Ref.update(this.value, (n) => n - 1)
    this.get = Ref.get(this.value)
  }
}

const make = Effect.andThen(Ref.make(0), (value) => new Counter(value))

const program = Effect.gen(function* () {
  const counter = yield* make

  // Helper to log the counter's value before running an effect
  const logCounter = (
    label: string,
    effect: Effect.Effect<any, any, any>
  ) =>
    Effect.gen(function* () {
      const value = yield* counter.get
      yield* Effect.log(`${label} get: ${value}`)
      return yield* effect
    })

  yield* logCounter("task 1", counter.inc).pipe(
    Effect.zip(logCounter("task 2", counter.inc), { concurrent: true }),
    Effect.zip(logCounter("task 3", counter.dec), { concurrent: true }),
    Effect.zip(logCounter("task 4", counter.inc), { concurrent: true })
  )
  const value = yield* counter.get
  yield* Effect.log(`This counter has a value of ${value}.`)
})

Effect.runPromise(program)
/* 
Output:
timestamp=... fiber=#3 message="task 4 get: 0"
timestamp=... fiber=#6 message="task 3 get: 1"
timestamp=... fiber=#8 message="task 1 get: 0"
timestamp=... fiber=#9 message="task 2 get: 1"
timestamp=... fiber=#0 message="This counter has a value of 2."
*/
```

---

### Use a Counter Metric in Effect-TS

Source: https://effect.website/docs/observability/metrics

Shows how to apply a counter metric to an Effect. The counter can be incremented or decremented by providing a numerical value. The current state of the counter can be retrieved using `Metric.value`. This example demonstrates incrementing by 1, 2, and decrementing by 4.

```typescript
import { Metric, Effect } from "effect"

const requestCount = Metric.counter("request_count")

const program = Effect.gen(function* () {
  // Increment the counter by 1
  const a = yield* requestCount(Effect.succeed(1))
  // Increment the counter by 2
  const b = yield* requestCount(Effect.succeed(2))
  // Decrement the counter by 4
  const c = yield* requestCount(Effect.succeed(-4))

  // Get the current state of the counter
  const state = yield* Metric.value(requestCount)
  console.log(state)

  return a * b * c
})

Effect.runPromise(program).then(console.log)
/*
Output:
CounterState {
  count: -1,
  ...
}
-8
*/
```

---

### Create a Gauge Metric in TypeScript

Source: https://effect.website/docs/observability/metrics

Demonstrates how to initialize a Gauge metric using the Metric.gauge constructor. This metric type is suitable for values that can increase or decrease, such as memory usage.

```typescript
import { Metric } from "effect"

const memory = Metric.gauge("memory_usage", {
  // Optional
  description: "A gauge for memory usage"
})
```

---

### Create a Counter Metric in Effect-TS

Source: https://effect.website/docs/observability/metrics

Demonstrates the creation of a basic counter metric using `Metric.counter`. This metric can track cumulative numerical changes over time. It requires the 'effect' library and accepts a name and an optional description.

```typescript
import { Metric, Effect } from "effect"

const requestCount = Metric.counter("request_count", {
  // Optional
  description: "A counter for tracking requests"
})
```
