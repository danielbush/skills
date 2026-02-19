# Effect – Micro Module

Lightweight `Micro` runtime for size-constrained environments.

---

### Await Fiber Completion with Micro.fiberAwait (TypeScript)

Source: https://effect.website/docs/micro/new-users

This example shows how to use `Micro.fiberAwait` to get detailed information about a fiber's completion. `Micro.fiberAwait` returns an effect that yields a `MicroExit` value, which describes whether the fiber succeeded, failed, or was interrupted.

```typescript
import { Micro } from "effect"

const fib = (n: number): Micro.Micro =>
  n < 2 
  ? Micro.succeed(n)
  : Micro.zipWith(fib(n - 1), fib(n - 2), (a, b) => a + b)

// ┌─── Micro, never, never>
// ▼
const fib10Fiber = Micro.fork(fib(10))

const program = Micro.gen(function* () {
  // Retrieve the fiber
  const fiber = yield* fib10Fiber
  // Await its completion and get the MicroExit result
  const exit = yield* Micro.fiberAwait(fiber)
  console.log(exit)
})

Micro.runPromise(program)
/*
Output:
{
  "_id": "MicroExit",
  "_tag": "Success",
  "value": 55
}
*/
```

---

### Join a Fiber and Retrieve its Result with Micro.fiberJoin (TypeScript)

Source: https://effect.website/docs/micro/new-users

This example demonstrates joining a forked fiber using `Micro.fiberJoin`. The `Micro.fiberJoin` function returns a `Micro` that succeeds or fails based on the outcome of the fiber it joins, allowing you to retrieve the computed value.

```typescript
import { Micro } from "effect"

const fib = (n: number): Micro.Micro =>
  n < 2 
  ? Micro.succeed(n)
  : Micro.zipWith(fib(n - 1), fib(n - 2), (a, b) => a + b)

// ┌─── Micro, never, never>
// ▼
const fib10Fiber = Micro.fork(fib(10))

const program = Micro.gen(function* () {
  // Retrieve the fiber
  const fiber = yield* fib10Fiber
  // Join the fiber and get the result
  const n = yield* Micro.fiberJoin(fiber)
  console.log(n)
})

Micro.runPromise(program)
// Output: 55
```

---

### Terminate on Unexpected Errors with Micro.die in Effect.js

Source: https://effect.website/docs/micro/new-users

This example shows how to use Micro.die to immediately terminate a program when an unexpected and critical error (a 'defect') is detected. It's typically used for unrecoverable situations.

```typescript
import { Micro } from "effect"

const divide = (a: number, b: number)
```

---

### Interrupt a Running Fiber with Micro.fiberInterrupt (TypeScript)

Source: https://effect.website/docs/micro/new-users

This example demonstrates interrupting a long-running fiber using `Micro.fiberInterrupt`. The fiber is forked to run indefinitely, and after a delay, it is interrupted, safely terminating its execution and releasing resources.

```typescript
import { Micro } from "effect"

const program = Micro.gen(function* () {
  // Fork a fiber that runs indefinitely, printing "Hi!"
  const fiber = yield* Micro.fork(
    Micro.forever(
      Micro.sync(() => console.log("Hi!")).pipe(Micro.delay(10))
    )
  )
  yield* Micro.sleep(30)
  // Interrupt the fiber
  yield* Micro.fiberInterrupt(fiber)
})

Micro.runPromise(program)
/*
Output:
Hi!
Hi!
*/
```

---

### Catch All Errors with Micro.catchAll in Effect.js

Source: https://effect.website/docs/micro/new-users

This example illustrates how to use Micro.catchAll to intercept any error originating from a program and replace it with a successful outcome. It's useful for ensuring a program always completes, even when unexpected errors occur.

```typescript
import { Micro } from "effect"

class HttpError {
  readonly _tag = "HttpError"
}

class ValidationError {
  readonly _tag = "ValidationError"
}

const program = Micro.gen(function* () {
  // Simulate http and validation errors
  if (Math.random() > 0.5) yield* Micro.fail(new HttpError())
  if (Math.random() > 0.5) yield* Micro.fail(new ValidationError())
  return "some result"
})

const recovered = program.pipe(
  Micro.catchAll((error) =>
    Micro.succeed(`Recovering from ${error._tag}`)
  )
)

Micro.runPromiseExit(recovered).then(console.log)
/* 
Example Output:
{
  "_id": "MicroExit",
  "_tag": "Success",
  "value": "Recovering from HttpError"
}
*/
```

---

### Access Service in Micro.gen with Effect.js

Source: https://effect.website/docs/micro/effect-users

Shows how to access a service within a `Micro.gen` block by using `Micro.service`. This is the correct way to yield services in `Micro.gen`, as directly yielding the service tag will not work. The example demonstrates providing a `Random` service and then using it within the generated program.

```typescript
import { Micro, Context } from "effect"

class Random extends Context.Tag("MyRandomService")< 
Random, 
{ readonly next: Micro.Micro }
>() {}

const program = Micro.gen(function* () {
  // const random = yield* Random // this doesn't work
  const random = yield* Micro.service(Random)
  const randomNumber = yield* random.next
  console.log(`random number: ${randomNumber}`)
})

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

### Importing the Micro Module in Effect.js

Source: https://effect.website/docs/micro/effect-users

Demonstrates how to import the 'Micro' module from the 'effect' library using both default and namespace imports. It also touches upon tree shaking considerations with bundlers.

```typescript
import { Micro } from "effect"

// Or using a namespace import:
import * as Micro from "effect/Micro"
```

---

### Inspect Micro Effect Exit Status with Micro.runPromiseExit

Source: https://effect.website/docs/micro/new-users

This example demonstrates how to use Micro.runPromiseExit to execute a Micro effect and retrieve its detailed exit status. Unlike Micro.runPromise, this function returns a Promise that resolves with a structured exit object, which can be a 'Success' containing the value or a 'Fail' containing the error. This provides more granular control over error inspection.

```typescript
import { Micro } from "effect"

// Simulate fetching weather data
function fetchWeather(city: string): Promise {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (city === "London") {
        resolve("Sunny")
      } else {
        reject(new Error("Weather data not found for this location"))
      }
    }, 1_000)
  })
}

function getWeather(city: string) {
  return Micro.promise(() => fetchWeather(city))
}

const weatherEffect = getWeather("London")

Micro.runPromiseExit(weatherEffect).then(
  (exit) => console.log(exit)
)
```

---

### Wrap Promise with Micro Effect

Source: https://effect.website/docs/micro/new-users

Shows how to wrap a Promise-based function (`fetchWeather`) using the Micro library from Effect. This converts the Promise into a Micro effect, enabling structured handling of success and failure.

```typescript
import { Micro } from "effect"

// Simulate fetching weather data
function fetchWeather(city: string): Promise<string> {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      // ... (rest of the function implementation)
    }, 1000)
  })
}

// Wrap the Promise with Micro
const getWeatherEffect = Micro.fromPromise(fetchWeather("London"))

```

---

### Fiber Interruption with Micro.interrupt (TypeScript)

Source: https://effect.website/docs/micro/new-users

This example illustrates interrupting a fiber using `Micro.interrupt` within the fiber's own execution context. The `Micro.interrupt` effect causes the fiber to terminate with an interruption cause, preventing subsequent operations within that fiber from executing.

```typescript
import { Micro } from "effect"

const program = Micro.gen(function* () {
  console.log("start")
  yield* Micro.sleep(2000)
  yield* Micro.interrupt
  console.log("done")
})

Micro.runPromiseExit(program).then(console.log)
/*
Output:
start
{
  "_id": "MicroExit",
  "_tag": "Failure",
  "cause": {
    "_tag": "Interrupt",
    "traces": [],
    "name": "MicroCause.Interrupt"
  }
}
*/
```

---

### Executing Effects and Obtaining a MicroFiber with Micro.runFork

Source: https://effect.website/docs/micro/effect-users

Explains the usage of `Micro.runFork` to execute a 'Micro' effect and obtain a `MicroFiber`. This fiber can be managed (awaited, joined, aborted), and results can be observed using `addObserver`.

```typescript
import { Micro } from "effect"

// Example usage:
// const fiber = Micro.runFork(myMicroEffect)
// fiber.addObserver(result => {
//   console.log("Effect finished:", result)
// })

```

---

### Micro module for Effect Users

Source: https://context7_llms

Introduction to the Micro module, a lightweight alternative to Effect designed to reduce bundle size while maintaining compatibility for TypeScript applications.

```markdown
The Micro module offers a more minimal API surface compared to the full Effect library, suitable for scenarios where bundle size is critical.
```

---

### Running Effects Synchronously with Micro.runSyncExit

Source: https://effect.website/docs/micro/effect-users

Demonstrates the synchronous execution of a 'Micro' effect using `Micro.runSyncExit`. It shows how the function returns a `MicroExit` object, indicating either success with a value or failure with a `MicroCause`.

```typescript
import { Micro } from "effect"

const result1 = Micro.runSyncExit(Micro.succeed(1))
console.log(result1)
/* 
Output:
{
  "_id": "MicroExit",
  "_tag": "Success",
  "value": 1
}
*/

const result2 = Micro.runSyncExit(Micro.fail("my error"))
console.log(result2)
/* 
Output:
{
  "_id": "MicroExit",
  "_tag": "Failure",
  "cause": {
    "_tag": "Fail",
    "traces": [],
    "name": "MicroCause.Fail",
    "error": "my error"
  }
}
*/
```

---

### Import Micro Module from Effect

Source: https://effect.website/docs/micro/new-users

Demonstrates two ways to import the Micro module from the Effect library: a named import and a namespace import. Both methods provide access to Micro functionalities.

```typescript
import { Micro } from "effect"

```

```typescript
import * as Micro from "effect/Micro"

```

---

### Handle Various Error Types with Effect Micro

Source: https://effect.website/docs/micro/new-users

Demonstrates how to use Micro.sandbox and catchTag to handle different error types (Fail, Interrupt, Die) in an Effect, providing fallback results for each. It logs specific error details based on the caught error type.

```typescript
import { Micro } from "effect"

// Helper function to log a message
const log = (message: string) => Micro.sync(() => console.log(message))

// ┌─── Micro
// ▼
const task = Micro.fail(new Error("Oh uh!")).pipe(
  Micro.as("primary result")
)

// ┌─── Effect, never>
// ▼
const sandboxed = Micro.sandbox(task)

const program = sandboxed.pipe(
  Micro.catchTag("Fail", (cause) =>
    log(`Caught a defect: ${cause.error}`).pipe(
      Micro.as("fallback result on expected error")
    )
  ),
  Micro.catchTag("Interrupt", () =>
    log(`Caught a defect`).pipe(
      Micro.as("fallback result on fiber interruption")
    )
  ),
  Micro.catchTag("Die", (cause) =>
    log(`Caught a defect: ${cause.defect}`).pipe(
      Micro.as("fallback result on unexpected error")
    )
  )
)

Micro.runPromise(program).then(console.log)
/* 
Output:
Caught a defect: Error: Oh uh!
fallback result on expected error 
*/
```

---

### Run Micro Effect and Handle Results with Micro.runPromise

Source: https://effect.website/docs/micro/new-users

This code shows how to execute a Micro effect and handle its success or failure using Micro.runPromise. This function converts the Micro effect back into a standard JavaScript Promise, allowing for the use of familiar .then() and .catch() methods for asynchronous operations. It logs the weather data or any errors encountered during the fetch.

```typescript
import { Micro } from "effect"

// Simulate fetching weather data
function fetchWeather(city: string): Promise {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (city === "London") {
        resolve("Sunny")
      } else {
        reject(new Error("Weather data not found for this location"))
      }
    }, 1_000)
  })
}

function getWeather(city: string) {
  return Micro.promise(() => fetchWeather(city))
}

const weatherEffect = getWeather("London")

Micro.runPromise(weatherEffect)
  .then((data) => console.log(`The weather in London is: ${data}`))
  .catch((error) =>
    console.error(`Failed to fetch weather data: ${error.message}`)
  )
```
