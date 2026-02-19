# Effect – Error Handling

`catchAll`, `catchTag`, `catchIf`, `sandbox`, `Cause`, `Either`, and matching on failures.

---

### Handle Failures During Effect Repetition

Source: https://effect.website/docs/micro/new-users

This example demonstrates how `Micro.repeat` handles failures when repeating an effect. If an effect fails during repetition, the process stops, and the failure is reported. The `Micro.runPromiseExit` function is used to observe the final outcome.

```typescript
import { Micro } from "effect"

let count = 0

// Define an async effect that simulates an action with potential failure
const action = Micro.async((resume) => {
  if (count > 1) {
    console.log("failure")
    resume(Micro.fail("Uh oh!"))
  } else {
    count++
    console.log("success")
    resume(Micro.succeed("yay!"))
  }
})

// Define a schedule that repeats the action 2 more times with a delay
const policy = Micro.scheduleAddDelay(Micro.scheduleRecurs(2), () => 100)

// Repeat the action according to the schedule
const program = Micro.repeat(action, { schedule: policy })

// Run the program and observe the result on failure
Micro.runPromiseExit(program).then(console.log)
/* 
Output:
success
success
failure
{
  "_id": "MicroExit",
  "_tag": "Failure",
  "cause": {
    "_tag": "Fail",
    "traces": [],
    "name": "MicroCause.Fail",
    "error": "Uh oh!"
  }
}
*/
```

---

### Pattern Matching on Causes with Cause.match

Source: https://effect.website/docs/data-types/cause

Provides an example of using Cause.match to handle different types of Causes by defining specific callbacks for each potential error scenario, allowing for structured error management.

```typescript
import { Cause } from "effect"

const cause = Cause.parallel(
  Cause.fail(new Error("my fail message")),
  Cause.die("my die message")
)

console.log(
  Cause.match(cause, {
    onEmpty: "(empty)",
    onFail: (error) => `(error: ${error.message})`,
    onDie: (defect) => `(defect: ${defect})`,
    onInterrupt: (fiberId) => `(fiberId: ${fiberId})`,
    onSequential: (left, right) =>
      `(onSequential (left: ${left}) (right: ${right}))`,
    onParallel: (left, right) =>
      `(onParallel (left: ${left}) (right: ${right}))`
  })
)
/* 
Output:
(onParallel (left: (error: my fail message)) (right: (defect: my die message)) 
*/
```

---

### Sandboxing and Handling Errors with Effect.sandbox

Source: https://effect.website/docs/error-management/sandboxing

This example demonstrates how to use Effect.sandbox to capture different types of errors (Defect, Interrupt, Fail) and handle them using Effect.catchTags. It then uses Effect.unsandbox to restore the original error handling before running the program.

```typescript
import { Effect, Console } from "effect"

// Create an effect that fails
const task = Effect.fail(new Error("Oh uh!")).pipe(
  Effect.as("primary result")
)

// Sandbox the task to capture the full cause of the error
const sandboxed = Effect.sandbox(task)

// Handle different error causes using Effect.catchTags
const program = Effect.catchTags(sandboxed, {
  Die: (cause) =>
    Console.log(`Caught a defect: ${cause.defect}`).pipe(
      Effect.as("fallback result on defect")
    ),
  Interrupt: (cause) =>
    Console.log(`Caught a defect: ${cause.fiberId}`).pipe(
      Effect.as("fallback result on fiber interruption")
    ),
  Fail: (cause) =>
    Console.log(`Caught a defect: ${cause.error}`).pipe(
      Effect.as("fallback result on failure")
    )
})

// Restore the original error handling with unsandbox
const main = Effect.unsandbox(program)

// Run the program and log the final result
Effect.runPromise(main).then(console.log)

```

---

### Recovering from All Errors with Effect.catchAllCause

Source: https://effect.website/docs/error-management/expected-errors

This example demonstrates how to recover from any type of error in an Effect program using `Effect.catchAllCause`. It checks the cause of the error and returns a success value, allowing for graceful handling of both regular failures and defects.

```typescript
import * as Effect from "effect/Effect"
import * as Cause from "effect/Cause"

const program = Effect.fail("Something went wrong!")

// Recover from all errors by examining the cause
const recovered = program.pipe(
  Effect.catchAllCause((cause) =>
    Cause.isFailType(cause)
      ? Effect.succeed("Recovered from a regular error")
      : Effect.succeed("Recovered from a defect")
  )
)

Effect.runPromise(recovered).then(console.log)
// Output: "Recovered from a regular error"
```

---

### Handle Errors by Tag with Effect.catchTag

Source: https://effect.website/docs/error-management/expected-errors

Effect.catchTag provides a way to handle errors based on their `_tag` field. This is useful for discriminated union error types. This example shows how to catch a specific HttpError.

```typescript
import { Effect, Random, Data } from "effect"

class HttpError extends Data.TaggedError("HttpError")<{}>

class ValidationError extends Data.TaggedError("ValidationError")<{}>

const program = Effect.gen(function* () {
  const n1 = yield* Random.next
  const n2 = yield* Random.next
  if (n1 < 0.5) {
    return yield* Effect.fail(new HttpError())
  }
  if (n2 < 0.5) {
    return yield* Effect.fail(new ValidationError())
  }
  return "some result"
})

const recovered = program.pipe(
  // Only handle HttpError errors
  Effect.catchTag("HttpError", (_HttpError) => 
    Effect.succeed("Recovering from HttpError")
  )
)

// const recovered: Effect<string, ValidationError, never>

```

---

### Collecting All Errors with combineWithAllErrors in neverthrow and Effect

Source: https://effect.website/docs/additional-resources/effect-vs-neverthrow

This example shows how to collect all errors from a list of asynchronous operations, instead of failing on the first error. It uses `ResultAsync.combineWithAllErrors` in neverthrow and `Effect.validateAll` with `Effect.either` in Effect.

```typescript
import { ResultAsync, okAsync, errAsync } from "neverthrow"

const resultList: ResultAsync<number, string>[] = [
  okAsync(123),
  errAsync("boooom!"),
  okAsync(456),
  errAsync("ahhhhh!")
]

const result = await ResultAsync.combineWithAllErrors(resultList)
// result is Err(['boooom!', 'ahhhhh!'])
```

```typescript
import { Effect, identity } from "effect"

const resultList: Effect.Effect<number, string>[] = [
  Effect.succeed(123),
  Effect.fail("boooom!"),
  Effect.succeed(456),
  Effect.fail("ahhhhh!")
]

const result = await Effect.runPromise(
  Effect.either(Effect.validateAll(resultList, identity))
)
// result is left(['boooom!', 'ahhhhh!'])
```

---

### Handle Multiple Error Types with Effect.catchTag

Source: https://effect.website/docs/error-management/expected-errors

This example demonstrates how to chain multiple Effect.catchTag calls to handle different error types. By catching both HttpError and ValidationError, the resulting effect's error channel is narrowed.

```typescript
import { Effect, Random, Data } from "effect"

class HttpError extends Data.TaggedError("HttpError")<{}>

class ValidationError extends Data.TaggedError("ValidationError")<{}>

const program = Effect.gen(function* () {
  const n1 = yield* Random.next
  const n2 = yield* Random.next
  if (n1 < 0.5) {
    return yield* Effect.fail(new HttpError())
  }
  if (n2 < 0.5) {
    return yield* Effect.fail(new ValidationError())
  }
  return "some result"
})

const recovered = program.pipe(
  Effect.catchTag("HttpError", (_HttpError) => 
    Effect.succeed("Recovering from HttpError")
  ),
  Effect.catchTag("ValidationError", (_ValidationError) => 
    Effect.succeed("Recovering from ValidationError")
  )
)

// const recovered: Effect<string, never, never>

```

---

### Handling Success and Failure with match in neverthrow and Effect

Source: https://effect.website/docs/additional-resources/effect-vs-neverthrow

This example demonstrates how to handle both successful and failed outcomes at the end of a chain of asynchronous operations. It uses the `match` function in neverthrow and Effect's `match` operator.

```typescript
import { ResultAsync } from "neverthrow"

interface User {
  readonly name: string
}
declare function validateUser(user: User): ResultAsync<User, Error>
declare function insertUser(user: User): ResultAsync<User, Error>

const user: User = { name: "John" }

// Handle both cases at the end of the chain using match
const resultMessage = await validateUser(user)
  .andThen(insertUser)
  .match(
    (user: User) => `User ${user.name} has been successfully created`,
    (error: Error) => `User could not be created because ${error.message}`
  )
```

```typescript
import * as Effect from "effect/Effect"

interface User {
  readonly name: string
}
declare function validateUser(user: User): Effect.Effect<User, Error>
declare function insertUser(user: User): Effect.Effect<User, Error>

const user: User = { name: "John" }

// Handle both cases at the end of the chain using match
const resultMessage = await Effect.runPromise(
  validateUser(user).pipe(
    Effect.andThen(insertUser),
    Effect.match({
      onSuccess: (user) =>
        `User ${user.name} has been successfully created`,
      onFailure: (error) =>
        `User could not be created because ${error.message}`
    })
  )
)
```

---

### Handle Failures with matchEffect in Effect.js

Source: https://effect.website/docs/micro/new-users

Demonstrates how to use Micro.matchEffect to handle both successful outcomes and failures in an Effect.js program. It pipes the result through a tap function for logging.

```typescript
import * as Micro from "effect/Cause"

const log = (message: string) => Micro.sync(() => console.log(message))

const program1 = Micro.matchEffect(Micro.succeed(42), {
  onFailure: (error) => Micro.succeed(`failure: ${error.message}`).pipe(Micro.tap(log)),
  onSuccess: (value) => Micro.succeed(`success: ${value}`).pipe(Micro.tap(log))
})

Micro.runSync(program1)
/* 
Output:
success: 42 
*/

const program2 = Micro.matchEffect(Micro.fail(new Error("Uh oh!")), {
  onFailure: (error) => Micro.succeed(`failure: ${error.message}`).pipe(Micro.tap(log)),
  onSuccess: (value) => Micro.succeed(`success: ${value}`).pipe(Micro.tap(log))
})

Micro.runSync(program2)
/* 
Output:
failure: Uh oh! 
*/
```

---

### Effect.gen Error Handling with Effect.fail (TypeScript)

Source: https://effect.website/docs/getting-started/using-generators

Shows how to introduce errors into an Effect.gen workflow using `Effect.fail`. This example executes two console logging tasks and then intentionally fails the program with a custom error message, demonstrating immediate error propagation.

```typescript
import { Effect, Console } from "effect"

const task1 = Console.log("task1...")
const task2 = Console.log("task2...")

const program = Effect.gen(function* () {
  // Perform some tasks
  yield* task1
  yield* task2
  // Introduce an error
  yield* Effect.fail("Something went wrong!")
})

Effect.runPromise(program).then(console.log, console.error)
```

---

### TypeScript: Demonstrate Short-Circuiting Behavior in Effect

Source: https://effect.website/docs/error-management/expected-errors

This example illustrates the short-circuiting behavior of Effect.js when errors occur in sequential operations. It defines three tasks, where the second task is designed to fail. The code shows that subsequent tasks are skipped when an error is encountered, and the error is propagated.

```typescript
import { Effect, Console } from "effect"

const task1 = Console.log("Executing task1...")
const task2 = Effect.fail("Something went wrong!")
const task3 = Console.log("Executing task3...")

const program = Effect.gen(function* () {
  yield* task1
  yield* task2
  yield* task3
})

Effect.runPromiseExit(program).then(console.log)
/* 
Output:
Executing task1...
{ 
  _id: 'Exit', 
  _tag: 'Failure', 
  cause: { _id: 'Cause', _tag: 'Fail', failure: 'Something went wrong!' } 
}
*/
```

---

### Recover from Specific Errors with Effect.catchIf (TypeScript < 5.5)

Source: https://effect.website/docs/error-management/expected-errors

Effect.catchIf allows recovery from errors based on a predicate function. This example demonstrates catching a specific HttpError using a predicate. Note that for TypeScript versions prior to 5.5, the error type is not altered by default.

```typescript
import { Data, Effect, Random } from "effect"

class HttpError extends Data.TaggedError("HttpError")<{}>

class ValidationError extends Data.TaggedError("ValidationError")<{}>

const program = Effect.gen(function* () {
  const n1 = yield* Random.next
  const n2 = yield* Random.next
  if (n1 < 0.5) {
    return yield* Effect.fail(new HttpError())
  }
  if (n2 < 0.5) {
    return yield* Effect.fail(new ValidationError())
  }
  return "some result"
})

const recovered = program.pipe(
  Effect.catchIf(
    // Only handle HttpError errors
    (error) => error._tag === "HttpError",
    () => Effect.succeed("Recovering from HttpError")
  )
)

// const recovered: Effect

```

---

### Handling Specific Errors with Effect.either (Part 2)

Source: https://effect.website/docs/error-management/expected-errors

This example extends the previous one by demonstrating how to handle both `HttpError` and `ValidationError` using `Effect.either`. After capturing the result with `Effect.either`, both error types are explicitly handled, resulting in the error channel of the `recovered` effect becoming `never`, indicating all errors are managed.

```typescript
import { Effect, Random, Either, Data } from "effect"

class HttpError extends Data.TaggedError("HttpError")<{}> {}
class ValidationError extends Data.TaggedError("ValidationError")<{}> {}

const program = Effect.gen(function* () {
  const n1 = yield* Random.next
  const n2 = yield* Random.next
  if (n1 < 0.5) {
    return yield* Effect.fail(new HttpError())
  }
  if (n2 < 0.5) {
    return yield* Effect.fail(new ValidationError())
  }
  return "some result"
})

const recovered = Effect.gen(function* () {
  const failureOrSuccess = yield* Effect.either(program)
  if (Either.isLeft(failureOrSuccess)) {
    const error = failureOrSuccess.left
    // Handle both HttpError and ValidationError
    if (error._tag === "HttpError") {
      return "Recovering from HttpError"
    } else {
      return "Recovering from ValidationError"
    }
  } else {
    return failureOrSuccess.right
  }
})
```

---

### TypeScript: Handle Errors with Effect.either in Effect.js

Source: https://effect.website/docs/error-management/expected-errors

This example demonstrates how to use Effect.either to transform an Effect into one that returns an Either data type, encapsulating both success and failure. This allows for explicit handling of both outcomes within a generator function by pattern matching on the Either. It uses custom error types.

```typescript
import { Effect, Either, Random, Data } from "effect"

class HttpError extends Data.TaggedError("HttpError")<{}> {}

class ValidationError extends Data.TaggedError("ValidationError")<{}> {}

const program = Effect.gen(function* () {
  const n1 = yield* Random.next
  const n2 = yield* Random.next

  if (n1 < 0.5) {
    return yield* Effect.fail(new HttpError())
  }

  if (n2 < 0.5) {
    return yield* Effect.fail(new ValidationError())
  }

  return "some result"
})

const handledProgram = Effect.either(program)

// handledProgram now returns Effect<Either<HttpError | ValidationError, string>, never, never>
```

---

### Promise.allSettled() vs. Effect.forEach with Effect.either

Source: https://effect.website/docs/additional-resources/effect-vs-promise

This snippet showcases how to handle an array of asynchronous tasks, ensuring that all tasks complete regardless of their success or failure. It contrasts the native Promise.allSettled with the Effect library's approach using Effect.forEach and Effect.either for granular result handling.

```javascript
const task1 = new Promise((resolve, reject) => {
  console.log("Executing task1...")
  setTimeout(() => {
    console.log("task1 done")
    resolve(1)
  }, 100)
})

const task2 = new Promise((resolve, reject) => {
  console.log("Executing task2...")
  setTimeout(() => {
    console.log("task2 done")
    reject("Uh oh!")
  }, 200)
})

const task3 = new Promise((resolve, reject) => {
  console.log("Executing task3...")
  setTimeout(() => {
    console.log("task3 done")
    resolve(3)
  }, 300)
})

const program = Promise.allSettled([task1, task2, task3])

program.then(console.log, console.error)
/* 
Output:
Executing task1...
Executing task2...
Executing task3...
task1 done
task2 done
task3 done
[
  { status: 'fulfilled', value: 1 },
  { status: 'rejected', reason: 'Uh oh!' },
  { status: 'fulfilled', value: 3 }
]
*/
```

```typescript
import { Effect } from "effect"

const task1 = Effect.gen(function* () {
  console.log("Executing task1...")
  yield* Effect.sleep("100 millis")
  console.log("task1 done")
  return 1
})

const task2 = Effect.gen(function* () {
  console.log("Executing task2...")
  yield* Effect.sleep("200 millis")
  console.log("task2 done")
  return yield* Effect.fail("Uh oh!")
})

const task3 = Effect.gen(function* () {
  console.log("Executing task3...")
  yield* Effect.sleep("300 millis")
  console.log("task3 done")
  return 3
})

const program = Effect.forEach(
  [task1, task2, task3],
  (task) => Effect.either(task), // or Effect.exit 
  {
    concurrency: "unbounded"
  }
)

Effect.runPromise(program).then(console.log, console.error)
/* 
Output:
Executing task1...
Executing task2...
Executing task3...
task1 done
task2 done
[
  {
    _id: "Either",
    _tag: "Right",
    right: 1 
  }, {
    _id: "Either",
    _tag: "Left",
    left: "Uh oh!"
  }, {
    _id: "Either",
    _tag: "Right",
    right: 3 
  }
]
*/
```

---

### TypeScript: Automatically Track Union Error Types in Effect

Source: https://effect.website/docs/error-management/expected-errors

This example demonstrates how Effect.js automatically tracks multiple potential error types as a union. It defines custom error classes for HTTP and validation errors and shows how Effect.gen composes operations that can fail with these distinct errors, making error handling more predictable.

```typescript
import { Effect, Random, Data } from "effect"

class HttpError extends Data.TaggedError("HttpError")<{}> {}

class ValidationError extends Data.TaggedError("ValidationError")<{}> {}

const program = Effect.gen(function* () {
  const n1 = yield* Random.next
  const n2 = yield* Random.next

  if (n1 < 0.5) {
    return yield* Effect.fail(new HttpError())
  }

  if (n2 < 0.5) {
    return yield* Effect.fail(new ValidationError())
  }

  return "some result"
})

// The inferred type of 'program' will be Effect<string, HttpError | ValidationError, never>
```

---

### Handle Failure: Promise vs. Effect

Source: https://effect.website/docs/additional-resources/effect-vs-promise

Illustrates how to create and handle a failed asynchronous operation using Promise.reject and Effect.fail respectively. Both methods signal an unsuccessful outcome.

```javascript
const failure = Promise.reject("Uh oh!")
```

```typescript
import { Effect } from "effect"

const failure = Effect.fail("Uh oh!")
```
