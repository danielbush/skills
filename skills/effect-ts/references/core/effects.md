# Effect – Core Effects

Basic effect creation, generators (`Effect.gen`), pipe, running effects, control flow, `Ref`, timeouts, and concurrency primitives.

---

### Full Game Setup and Execution (TypeScript)

Source: https://effect.website/docs/platform/terminal

An Effect that sets up the game by displaying initial instructions and then starting the game loop with a randomly generated secret number. This Effect orchestrates the entire game flow from start to finish.

```typescript
import { Effect, Option, Random } from "effect"
import { NodeRuntime, NodeTerminal } from "@effect/platform-node"
import { Terminal, PlatformError } from "@effect/platform"

// Full game setup and execution
const game = Effect.gen(function* () {
  yield* display(
    `We have selected a random number between 1 and 100. 
See if you can guess it in 10 turns or fewer. 
We'll tell you if your guess was too high or too low.`
  )
  yield* loop(yield* secret)
})
```

---

### Build a Transaction Pipeline with Effect.js

Source: https://effect.website/docs/getting-started/building-pipelines

This example demonstrates building a transaction processing pipeline using Effect.js. It chains multiple effects together using `pipe`, `Effect.all`, and `Effect.andThen` to fetch data, apply discounts, add service charges, and format the final output. Dependencies include the 'effect' library.

```typescript
import { Effect, pipe } from "effect"

const addServiceCharge = (amount: number) => amount + 1

const applyDiscount = (
  total: number,
  discountRate: number
): Effect.Effect =>
  discountRate === 0
    ? Effect.fail(new Error("Discount rate cannot be zero"))
    : Effect.succeed(total - (total * discountRate) / 100);

const fetchTransactionAmount = Effect.promise(() => Promise.resolve(100));
const fetchDiscountRate = Effect.promise(() => Promise.resolve(5));

const program = pipe(
  Effect.all([fetchTransactionAmount, fetchDiscountRate]),
  Effect.andThen(([transactionAmount, discountRate]) => 
    applyDiscount(transactionAmount, discountRate)
  ),
  Effect.andThen(addServiceCharge),
  Effect.andThen(
    (finalAmount) => `Final amount to charge: ${finalAmount}`
  )
);

Effect.runPromise(program).then(console.log);
```

---

### Sequential Effect Execution Example

Source: https://effect.website/docs/concurrency/basic-concurrency

Demonstrates sequential execution of effects using Effect.all. Each effect starts only after the previous one has completed, with logs showing the order of execution. This is the default behavior when no concurrency option is specified.

```typescript
import { Effect, Duration } from "effect"

// Helper function to simulate a task with a delay
const makeTask = (n: number, delay: Duration.DurationInput) =>
  Effect.promise(
    () =>
      new Promise((resolve) => {
        console.log(`start task${n}`)
        setTimeout(() => {
          console.log(`task${n} done`)
          resolve()
        }, Duration.toMillis(delay))
      })
  )

const task1 = makeTask(1, "200 millis")
const task2 = makeTask(2, "100 millis")

const sequential = Effect.all([task1, task2])

Effect.runPromise(sequential)
/*
Output:
start task1
task1 done
start task2 <-- task2 starts only after task1 completes
task2 done
*/
```

---

### Create and Run a Basic EffectJS Program (Node.js/tsx)

Source: https://effect.website/docs/getting-started/installation

Sets up a minimal EffectJS project directory and a simple TypeScript file that logs 'Hello, World!' to the console. The program is executed using tsx.

```bash
mkdir src
touch src/index.ts
```

```typescript
1 import { Effect, Console } from "effect" 
2 
3 const program = Console.log("Hello, World!") 
4 
5 Effect.runSync(program)
```

```bash
npx tsx src/index.ts
```

---

### Create a New Effect App

Source: https://context7_llms

Command to quickly set up a new Effect application using a customizable template. This streamlines the initial project configuration.

```bash
npm create effect-app@latest
```

---

### Effect.gen Example: Transactions with Discounts

Source: https://effect.website/docs/getting-started/using-generators

This example demonstrates using Effect.gen to orchestrate a series of asynchronous operations, including fetching transaction amounts and discount rates, and then applying a discount. It showcases error handling for invalid discount rates. The code relies on the 'effect' library.

```typescript
import { Effect } from "effect"

// Function to add a small service charge to a transaction amount
const addServiceCharge = (amount: number) => amount + 1

// Function to apply a discount safely to a transaction amount
const applyDiscount = (
    total: number,
    discountRate: number
): Effect.Effect =>
    discountRate === 0
        ? Effect.fail(new Error("Discount rate cannot be zero"))
        : Effect.succeed(total - (total * discountRate) / 100)

// Simulated asynchronous task to fetch a transaction amount from a
// database
const fetchTransactionAmount = Effect.promise(() => Promise.resolve(100))

// Simulated asynchronous task to fetch a discount rate from a
// configuration file
const fetchDiscountRate = Effect.promise(() => Promise.resolve(5))

// Assembling the program using a generator function
const program = Effect.gen(function* () {
    // Retrieve the transaction amount
    const transactionAmount = yield* fetchTransactionAmount

    // Retrieve the discount rate
    const discountRate = yield* fetchDiscountRate

    // Calculate discounted
```

---

### Creating Basic Effects (TypeScript)

Source: https://effect.website/docs/stream/introduction

Demonstrates the creation of various Effects, including those that fail, produce a single value, a chunk of values, or an optional value. These examples illustrate the single-result nature of Effects.

```typescript
import { Effect, Chunk, Option } from "effect"

// An Effect that fails with a string error
const failedEffect = Effect.fail("fail!")

// An Effect that produces a single number
const oneNumberValue = Effect.succeed(3)

// An Effect that produces a chunk of numbers
const oneListValue = Effect.succeed(Chunk.make(1, 2, 3))

// An Effect that produces an optional number
const oneOption = Effect.succeed(Option.some(1))
```

---

### Simulating User Retrieval with Effect.succeed and Effect.fail

Source: https://effect.website/docs/getting-started/creating-effects

Provides an example of simulating a user retrieval operation, using `Effect.succeed` for successful retrieval and `Effect.fail` when a user is not found.

```APIDOC
## Simulating User Retrieval

### Description
Models a user retrieval operation where a user can either be found successfully (`Effect.succeed`) or not found (`Effect.fail`), useful for testing or mocking data.

### Method
`Effect.succeed`, `Effect.fail`

### Parameters
#### Path Parameters
None

#### Query Parameters
None

#### Request Body
None

### Request Example
```javascript
import { Effect } from "effect"

// Define a User type
interface User {
  readonly id: number
  readonly name: string
}

// A mocked function to simulate fetching a user from a database
const getUser = (userId: number): Effect.Effect<User, Error, never> => {
  // Normally, you would access a database or API here, but we'll mock it
  const userDatabase: Record<number, User> = {
    1: { id: 1, name: "John Doe" },
    2: { id: 2, name: "Jane Smith" }
  }

  // Check if the user exists in our "database" and return appropriately
  const user = userDatabase[userId]
  if (user) {
    return Effect.succeed(user)
  }
  else {
    return Effect.fail(new Error("User not found"))
  }
}

// When executed, this will successfully return the user with id 1
const exampleUserEffect = getUser(1)
```

### Response
#### Success Response (200)
User object if the user ID exists in the mocked database.

#### Response Example
```json
{
  "id": 1,
  "name": "John Doe"
}
```

#### Failure Response
An `Error` object with the message "User not found" if the user ID does not exist.

#### Response Example
```json
{
  "error": {
    "message": "User not found"
  }
}
```
```

---

### Create Effect Queries for API Interactions (TypeScript)

Source: https://effect.website/docs/batching

Demonstrates creating Effect queries for fetching todos, getting a user by ID, and sending emails, utilizing their respective defined resolvers. It also shows composition of these queries to build more complex operations like `sendEmailToUser` and `notifyOwner`.

```typescript
const getTodos: Effect.Effect<Array<Todo>, GetTodosError> = Effect.request(GetTodos({}), GetTodosResolver)

const getUserById = (id: number) =>
  Effect.request(GetUserById({ id }), GetUserByIdResolver)

const sendEmail = (address: string, text: string) =>
  Effect.request(SendEmail({ address, text }), SendEmailResolver)

const sendEmailToUser = (id: number, message: string) =>
  getUserById(id).pipe(
    Effect.andThen((user) => sendEmail(user.email, message))
  )

const notifyOwner = (todo: Todo) =>
  getUserById(todo.ownerId).pipe(
    Effect.andThen((user) =>
      sendEmailToUser(user.id, `hey ${user.name} you got a todo!`) 
    )
  )
```

---

### Running an Effect Using the Default Runtime (JavaScript)

Source: https://effect.website/docs/runtime

Demonstrates how to execute an Effect using the default runtime via Effect.runPromise and contrasts it with explicitly using Runtime.runPromise(Runtime.defaultRuntime). Both methods achieve the same outcome for running effects.

```javascript
import { Effect, Runtime } from "effect"

const program = Effect.log("Application started!")

// Using the default runtime shortcut
Effect.runPromise(program)
/* 
Output:
timestamp=... level=INFO fiber=#0 message="Application started!"
*/

// Explicitly using the default runtime
Runtime.runPromise(Runtime.defaultRuntime)(program)
/* 
Output: 
timestamp=... level=INFO fiber=#0 message="Application started!"
*/
```

---

### Control Flow: `ifEffect`

Source: https://context7_llms

Example of using the `ifEffect` operator for conditional branching based on the result of another Effect. This ensures type-safe conditional execution.

```typescript
import * as Effect from "@effect/io/Effect"

const condition = Effect.succeed(true)
const onTrue = Effect.succeed("Condition met")
const onFalse = Effect.succeed("Condition not met")

const result = Effect.ifEffect(condition)(
  () => onTrue,
  () => onFalse
)

Effect.runSync(result) // "Condition met"
```

---

### Effect.map data-first variant without pipe

Source: https://effect.website/docs/code-style/dual

Shows the 'data-first' usage of Effect.map, where the 'Effect' is explicitly provided as the first argument. This form is convenient for single operations.

```typescript
const mappedEffect = Effect.map(effect, func)
```

---

### Unbounded Concurrency with Effect.all

Source: https://effect.website/docs/concurrency/basic-concurrency

Shows how to run an unlimited number of effects concurrently by setting `concurrency` to 'unbounded' in `Effect.all`. This allows all tasks to start immediately, potentially consuming more resources.

```typescript
import { Effect, Duration } from "effect"

// Helper function to simulate a task with a delay
const makeTask = (n: number, delay: Duration.DurationInput) =>
  Effect.promise(
    () =>
      new Promise((resolve) => {
        console.log(`start task${n}`)
        setTimeout(() => {
          console.log(`task${n} done`)
          resolve()
        }, Duration.toMillis(delay))
      })
  )

const task1 = makeTask(1, "200 millis")
const task2 = makeTask(2, "100 millis")
const task3 = makeTask(3, "210 millis")
const task4 = makeTask(4, "110 millis")
const task5 = makeTask(5, "150 millis")

const unbounded = Effect.all([task1, task2, task3, task4, task5], {
  concurrency: "unbounded"
})

Effect.runPromise(unbounded)
```

---

### Effect.map dual API signatures

Source: https://effect.website/docs/code-style/dual

Illustrates the two TypeScript overloads for Effect.map, representing the 'data-last' and 'data-first' API variants. These signatures show the positional difference of the 'self' argument.

```typescript
declare const map: {
  // ┌─── data-last
  // ▼ (f: (a: A) => B): (self: Effect) => Effect
  // ┌─── data-first
  // ▼ (self: Effect, f: (a: A) => B): Effect
}
```

---

### Effect.map data-last variant with pipe

Source: https://effect.website/docs/code-style/dual

Demonstrates the 'data-last' usage of Effect.map, where the 'Effect' argument is implicitly passed via the 'pipe' function. This style is ideal for chaining multiple transformations.

```typescript
const mappedEffect = pipe(
  effect,
  Effect.map(func)
)
```

```typescript
pipe(
  effect,
  Effect.map(func1),
  Effect.map(func2),
  ...)
```

---

### Running an Effect (Sync)

Source: https://context7_llms

Shows how to synchronously run an Effect and handle its outcome. This is useful for simple computations where immediate results are desired.

```typescript
import * as Effect from "@effect/io/Effect"

const computation = Effect.sync(() => 10 + 5)

const result = Effect.runSync(computation)

console.log(result) // 15
```

---

### Async/Await Comparison with Effect.gen (TypeScript)

Source: https://effect.website/docs/getting-started/using-generators

Provides a direct comparison to the Effect.gen example using standard async/await syntax in TypeScript. Both snippets perform the same logical operations (fetching data, applying discount, adding service charge) to highlight the syntactical similarities in program flow.

```typescript
const addServiceCharge = (amount: number) => amount + 1

const applyDiscount = (
  total: number,
  discountRate: number
): Promise =>
  discountRate === 0
    ? Promise.reject(new Error("Discount rate cannot be zero"))
    : Promise.resolve(total - (total * discountRate) / 100)

const fetchTransactionAmount = Promise.resolve(100)

const fetchDiscountRate = Promise.resolve(5)

export const program = async function () {
  const transactionAmount = await fetchTransactionAmount
  const discountRate = await fetchDiscountRate
  const discountedAmount = await applyDiscount(
    transactionAmount,
    discountRate
  )
  const finalAmount = addServiceCharge(discountedAmount)
  return `Final amount to charge: ${finalAmount}`
}
```

---

### Custom Timeout Error with Effect.timeoutFail

Source: https://effect.website/docs/error-management/timing-out

The Effect.timeoutFail function allows you to produce a specific error when a timeout occurs. This example demonstrates defining a custom error class MyTimeoutError and using it with timeoutFail.

```typescript
import { Effect, Data } from "effect"

const task = Effect.gen(function* () {
  console.log("Start processing...")
  yield* Effect.sleep("2 seconds") // Simulates a delay in processing
  console.log("Processing complete.")
  return "Result"
})

class MyTimeoutError extends Data.TaggedError("MyTimeoutError")<{}> {}

const program = task.pipe(
  Effect.timeoutFail({
    duration: "1 second",
    onTimeout: () => new MyTimeoutError() // Custom timeout error
  })
)

Effect.runPromiseExit(program).then(console.log)
```

---

### Running an Effect Application with Graceful Teardown (Node.js)

Source: https://effect.website/docs/code-style/guidelines

Demonstrates how to use `NodeRuntime.runMain` to execute an Effect application on Node.js, ensuring graceful teardown upon interruption (e.g., CTRL+C). It includes adding a finalizer to log an exit message and repeating a process until interrupted.

```typescript
import { Effect, Console, Schedule, pipe } from "effect"
import { NodeRuntime } from "@effect/platform-node"

const program = pipe(
  Effect.addFinalizer(() => Console.log("Application is about to exit!")),
  Effect.andThen(Console.log("Application started!")),
  Effect.andThen(
    Effect.repeat(Console.log("still alive..."), {
      schedule: Schedule.spaced("1 second")
    })
  ),
  Effect.scoped
)

// Use NodeRuntime.runMain for graceful teardown on CTRL+C
NodeRuntime.runMain(program)
/* 
Output:
Application started!
still alive...
still alive...
still alive...
still alive...
^C <-- CTRL+C
Application is about to exit!
*/
```

---

### Deriving Custom Object Equivalence with mapInput

Source: https://effect.website/docs/behaviour/equivalence

Shows how to create a custom equivalence for objects by using `Equivalence.mapInput`. This example derives an equivalence for `User` objects that compares them based solely on their `id` property.

```typescript
import { Equivalence } from "effect"

interface User {
  readonly id: number
  readonly name: string
}

// Create an equivalence that compares User objects based only on the id
const equivalence = Equivalence.mapInput(
  Equivalence.number, // Base equivalence
  (user: User) => user.id // Function to extract the value to compare
)
```

---

### Custom Constructor: Promise vs. Effect

Source: https://effect.website/docs/additional-resources/effect-vs-promise

Shows how to create asynchronous operations with custom logic using the Promise constructor and Effect.gen. Both allow for deferred execution and conditional resolution or rejection.

```javascript
const task = new Promise((resolve, reject) => {
  setTimeout(() => {
    Math.random() > 0.5 ? resolve(2) : reject("Uh oh!")
  }, 300)
})
```

```typescript
import { Effect } from "effect"

const task = Effect.gen(function* () {
  yield* Effect.sleep("300 millis")
  return Math.random() > 0.5 ? 2 : yield* Effect.fail("Uh oh!")
})
```

---

### Combine Option and Effect - Example

Source: https://effect.website/docs/data-types/option

Demonstrates how `Option` values can be seamlessly integrated with `Effect` computations. `Option` variants (`None`, `Some`) map directly to `Effect` representations of absence or presence of a value. This allows for safe composition of potentially failing or optional asynchronous operations.

```typescript
import { Effect, Option } from "effect"

// Function to get the head of an array, returning Option
const head = (array: ReadonlyArray): Option.Option =>
  array.length > 0 ? Option.some(array[0]) : Option.none()

// Simulated fetch function that returns Effect
const fetchData = (): Effect.Effect => {
  const success = Math.random() > 0.5
  return success
    ? Effect.succeed("some data")
    : Effect.fail("Failed to fetch data")
}

// Mixing Either and Effect
const program = Effect.all([head([1, 2, 3]), fetchData()])

Effect.runPromise(program).then(console.log)
/*
Example Output:
[
  1,
  'some data'
]
*/
```

---

### Running an Effect (Async)

Source: https://context7_llms

Demonstrates how to asynchronously run an Effect, typically involving promises or other asynchronous operations. It handles potential errors and successes.

```typescript
import * as Effect from "@effect/io/Effect"

const asyncComputation = Effect.promise(() => Promise.resolve(20))

Effect.runPromise(asyncComputation)
  .then(result => console.log(result))
  .catch(error => console.error("An error occurred:", error)) // Outputs: 20
```

---

### Define Request Resolvers for Batching (TypeScript)

Source: https://effect.website/docs/batching

Illustrates how to define `RequestResolver` instances for handling specific Effect-TS requests. This snippet shows the setup for `getUserById` and `getTodos` resolvers, which are fundamental for batching operations by processing multiple requests efficiently.

```typescript
import { Effect, Request, RequestResolver, Data } from "effect"

// ------------------------------
// Model
// ------------------------------

interface User {
  readonly _tag: "User"
  readonly id: number
  readonly name: string
  readonly email: string
}

class GetUserError extends Data.TaggedError("GetUserError")<{}> {}

interface Todo {
  readonly _tag: "Todo"
  readonly id: number
  readonly message: string
  readonly ownerId: number
}

// Placeholder for GetTodosError and SendEmailError, assumed to be defined elsewhere
class GetTodosError extends Data.TaggedError("GetTodosError")<{}> {}
class SendEmailError extends Data.TaggedError("SendEmailError")<{}> {}

// Mock data for demonstration
const mockUsers: Map<number, User> = new Map([
  [1, { _tag: "User", id: 1, name: "Alice", email: "alice@example.com" }]
]);

const mockTodos: Map<number, Array<Todo>> = new Map([
  [1, [
    { _tag: "Todo", id: 1, message: "Buy groceries", ownerId: 1 },
    { _tag: "Todo", id: 2, message: "Walk the dog", ownerId: 1 }
  ]]
]);

// ------------------------------
// Resolvers
// ------------------------------

// Resolver for fetching a single User by ID
const userResolver: RequestResolver.RequestResolver<User | GetUserError, { id: number }>
  = RequestResolver.makeBatched((requests: Array<{ id: number }>) =>
      Effect.succeed([
        ...requests.map(req => {
          const user = mockUsers.get(req.id);
          return user ? Effect.succeed(user) : Effect.fail(new GetUserError());
        })
      ])
    );

// Resolver for fetching multiple Todo items
const todosResolver: RequestResolver.RequestResolver<Array<Todo> | GetTodosError, { _tag: "GetTodos" }>
  = RequestResolver.makeBatched((requests: Array<{ _tag: "GetTodos" }>)=> 
      Effect.succeed([
        ...requests.map(() => {
          // In a real scenario, this would fetch todos based on some criteria
          // For simplicity, returning a fixed list
          const todos = Array.from(mockTodos.values()).flat();
          return Effect.succeed(todos);
        })
      ])
    );

// Placeholder for SendEmail resolver, assuming it's defined elsewhere
// const sendEmailResolver: RequestResolver.RequestResolver<void | SendEmailError, SendEmailRequestType> = ...

// Example of combining resolvers
const combinedResolver = RequestResolver.batch(userResolver, todosResolver);

```

---

### Simulating User Retrieval with Effect.succeed and Effect.fail

Source: https://effect.website/docs/getting-started/creating-effects

Provides an example of modeling a user retrieval operation using Effect.succeed and Effect.fail. This function simulates fetching user data from a mocked database, returning either a User object on success or an Error if the user is not found.

```typescript
import { Effect } from "effect"

// Define a User type
interface User {
  readonly id: number
  readonly name: string
}

// A mocked function to simulate fetching a user from a database
const getUser = (userId: number): Effect.Effect => {
  // Normally, you would access a database or API here, but we'll mock it
  const userDatabase: Record<number, User> = {
    1: { id: 1, name: "John Doe" },
    2: { id: 2, name: "Jane Smith" }
  }

  // Check if the user exists in our "database" and return appropriately
  const user = userDatabase[userId]
  if (user) {
    return Effect.succeed(user)
  } else {
    return Effect.fail(new Error("User not found"))
  }
}

// When executed, this will successfully return the user with id 1
const exampleUserEffect = getUser(1)
```

---

### Handling Success and Timeout with Effect.timeoutTo (Either)

Source: https://effect.website/docs/error-management/timing-out

Effect.timeoutTo provides flexibility to define different outcomes for successful and timed-out operations. This example uses Either to return the result on success or an error message on timeout.

```typescript
import { Effect, Either } from "effect"

const task = Effect.gen(function* () {
  console.log("Start processing...")
  yield* Effect.sleep("2 seconds") // Simulates a delay in processing
  console.log("Processing complete.")
  return "Result"
})

const program = task.pipe(
  Effect.timeoutTo({
    duration: "1 second",
    onSuccess: (result): Either.Either => 
      Either.right(result),
    onTimeout: (): Either.Either => 
      Either.left("Timed out!")
  })
)

Effect.runPromise(program).then(console.log)
```

---

### Handling Successful Exit with Effect-TS

Source: https://effect.website/docs/data-types/exit

Demonstrates how to run an Effect that succeeds and process its Exit value. It uses `Effect.runSyncExit` to get an Exit object and `Exit.match` to handle the success case, extracting the successful value.

```typescript
import * as Effect from "@effect/io/Effect";
import * as Exit from "@effect/io/Exit";
import * as Cause from "@effect/io/Cause";

const simulatedSuccess = Effect.runSyncExit(Effect.succeed(1));

console.log(
  Exit.match(simulatedSuccess, {
    onFailure: (cause) => `Exited with failure state: ${Cause.pretty(cause)}`,
    onSuccess: (value) => `Exited with success value: ${value}`
  })
);
```

---

### Custom Defect on Timeout with Effect.timeoutFailCause

Source: https://effect.website/docs/error-management/timing-out

Effect.timeoutFailCause lets you define a specific defect to throw when a timeout occurs. This is useful for treating timeouts as exceptional cases. The example shows throwing a custom string defect.

```typescript
import { Effect, Cause } from "effect"

const task = Effect.gen(function* () {
  console.log("Start processing...")
  yield* Effect.sleep("2 seconds") // Simulates a delay in processing
  console.log("Processing complete.")
  return "Result"
})

const program = task.pipe(
  Effect.timeoutFailCause({
    duration: "1 second",
    onTimeout: () => Cause.die("Timed out!") // Custom defect for timeout
  })
)

Effect.runPromiseExit(program).then(console.log)
```

---

### Handle Multiple Resume Calls with Effect.async in JavaScript

Source: https://effect.website/docs/getting-started/creating-effects

Demonstrates the behavior of Effect.async when the 'resume' function is called multiple times. Only the first call to 'resume' will take effect; subsequent calls are ignored. This example shows a successful result followed by an ignored second attempt.

```javascript
import { Effect } from "effect"

const program = Effect.async((resume) => {
  resume(Effect.succeed(1))
  resume(Effect.succeed(2)) // This line will be ignored
})

// Run the program
Effect.runPromise(program).then(console.log) // Output: 1
```

---

### Compose Workspace Transactional Effect (Effect)

Source: https://effect.website/docs/resource-management/scope

Composes the `createBucket`, `createIndex`, and `createEntry` effects into a single scoped effect named `make`. This `make` effect ensures that resources are acquired and released in a transactional manner, suitable for workspace setup.

```typescript
import { Effect, Context, Exit, Data } from "effect"

// Assume S3, ElasticSearch, and Database services and their create operations are defined as above.

const make = Effect.scoped(
  Effect.gen(function* () {
    const bucket = yield* createBucket
    const index = yield* createIndex
    return yield* createEntry(bucket, index)
  })
)
```

---

### Wrap Callback API with Effect.async in JavaScript

Source: https://effect.website/docs/getting-started/creating-effects

Illustrates how to convert a Node.js fs module callback-based function (like readFile) into an Effect-compatible asynchronous operation using Effect.async. The 'resume' function is used to signal success with data or failure with an error. Ensure '@types/node' is installed for TypeScript.

```javascript
import { Effect } from "effect"
import * as NodeFS from "node:fs"

const readFile = (filename) =>
  Effect.async((resume) => {
    NodeFS.readFile(filename, (error, data) => {
      if (error) {
        // Resume with a failed Effect if an error occurs
        resume(Effect.fail(error))
      } else {
        // Resume with a succeeded Effect if successful
        resume(Effect.succeed(data))
      }
    })
  })

// ┌─── Effect
// ▼
const program = readFile("example.txt")
```

---

### Creating Effects with Different Causes (TypeScript)

Source: https://effect.website/docs/data-types/cause

This snippet demonstrates how to define effects that fail with specific causes using `Effect.failCause`. It shows examples of creating an effect that 'dies' with an unexpected error (`Cause.die`) and an effect that 'fails' with an expected error (`Cause.fail`).

```typescript
import { Effect, Cause } from "effect"

// Define an effect that dies with an unexpected error
//
// ┌─── Effect
// ▼
const die = Effect.failCause(Cause.die("Boom!"))

// Define an effect that fails with an expected error
//
// ┌─── Effect
// ▼
const fail = Effect.failCause(Cause.fail("Oh no!"))
```

---

### Control Flow: `while` Loop

Source: https://context7_llms

Demonstrates how to implement a `while` loop using Effect's control flow operators. This allows for iterative execution of Effects based on a condition.

```typescript
import * as Effect from "@effect/io/Effect"

let counter = 0
const loop = Effect.whileLoop(
  () => counter < 5,
  () => Effect.sync(() => {
    counter++
    console.log(`Iteration: ${counter}`)
  })
)

Effect.runSync(loop)
```

---

### Run Effect Without Interruption

Source: https://effect.website/docs/concurrency/basic-concurrency

Demonstrates a basic Effect execution that runs to completion without any interruption. It logs the start and end of a sleep operation, returning a success value. This serves as a baseline for understanding interruption.

```typescript
import { Effect } from "effect"

const program = Effect.gen(function* () {
 console.log("start")
 yield* Effect.sleep("2 seconds")
 console.log("done")
 return "some result"
})

Effect.runPromiseExit(program).then(console.log)
/* 
Output:
start
done
{ 
_id: 'Exit', 
_tag: 'Success', 
value: 'some result' 
}
*/
```

---

### Setting a Timeout for an Effect Operation (TypeScript)

Source: https://effect.website/docs/error-management/timing-out

This example demonstrates how to use `Effect.timeout` to set a time limit for an Effect operation. If the operation completes within the specified duration, its result is successfully returned. Otherwise, a TimeoutException is thrown.

```typescript
import { Effect } from "effect"

const task = Effect.gen(function* () {
  console.log("Start processing...")
  yield* Effect.sleep("2 seconds") // Simulates a delay in processing
  console.log("Processing complete.")
  return "Result"
})

// Sets a 3-second timeout for the task
const timedEffect = task.pipe(Effect.timeout("3 seconds"))

// Output will show that the task completes successfully
// as it falls within the timeout duration
Effect.runPromiseExit(timedEffect).then(console.log)
/*
Output:
Start processing...
Processing complete.
{ _id: 'Exit', _tag: 'Success', value: 'Result' }
*/
```

---

### Handle Overlapping Schemas in Union

Source: https://effect.website/docs/schema/basic-usage

Provides an example of defining a union schema with potentially overlapping member schemas. It highlights the importance of evaluation order, demonstrating how placing a more specific schema first ensures correct decoding.

```typescript
import { Schema } from "effect"

const Member1 = Schema.Struct({
  a: Schema.String
})

const Member2 = Schema.Struct({
  a: Schema.String,
  b: Schema.Number
})

// ❌ Define a union where Member1 appears first
const Bad = Schema.Union(Member1, Member2)

console.log(Schema.decodeUnknownSync(Bad)({ a: "a", b: 12 }))
// Output: { a: 'a' } (Member1 matched first, so `b` was ignored)

// ✅ Define a union where Member2 appears first
const Good = Schema.Union(Member2, Member1)

console.log(Schema.decodeUnknownSync(Good)({ a: "a", b: 12 }))
// Output: { a: 'a', b: 12 } (Member2 matched first, so `b` was included)
```

---

### Handle Invalid Input for String to Boolean Transformation (Effect)

Source: https://effect.website/docs/schema/transformations

This example shows how the `Schema.transform` function handles invalid input when decoding. If the input does not conform to the source schema ('on' or 'off' in this case), the transformation fails before the decode function is even called, providing detailed error messages.

```typescript
import { Schema } from "effect"

// Convert "on"/"off" to boolean and back
const BooleanFromString = Schema.transform(
  Schema.Literal("on", "off"),
  Schema.Boolean,
  {
    strict: true,
    decode: (s) => s === "on",
    encode: (bool) => (bool ? "on" : "off")
  }
)

// Providing input not allowed by the source schema
Schema.decodeUnknownSync(BooleanFromString)("wrong")
/*
throws:
ParseError: ("on" | "off" <-> boolean)
└─ Encoded side transformation failure
└─ "on" | "off"
├─ Expected "on", actual "wrong"
└─ Expected "off", actual "wrong"
*/
```

---

### Environment Object for Bundling Services in TypeScript

Source: https://effect.website/docs/requirements-management/services

Illustrates using a context or environment object to group multiple services. This approach simplifies passing services but introduces the complexity of ensuring the environment is correctly initialized with all required services.

```typescript
type Context = {
  databaseService: DatabaseService
  loggingService: LoggingService
}

const processData = (data: Data, context: Context) => {
  // Using multiple services from the context
}
```

---

### Main Effect Execution with Client Provisioning (TypeScript)

Source: https://effect.website/docs/ai/planning-llm-interactions

The main entry point for the application's effects. It pipes a main effect with the provision of both Anthropic and OpenAI clients, then runs the effect to completion using `runPromise`. This ensures the necessary client layers are available.

```typescript
main.pipe(
  Effect.provide([Anthropic, OpenAi]),
  Effect.runPromise
)
```

---

### Observe Asynchronous Effect with Effect.js

Source: https://effect.website/docs/micro/effect-users

Demonstrates how to observe the completion of an asynchronous effect using `Micro.runFork` and attaching an observer. This pattern is useful for handling side effects that occur after an effect has successfully completed. The output shows the logged result of the asynchronous operation.

```typescript
import { Micro } from "effect"

// ┌─── MicroFiber
// ▼
const fiber = Micro.succeed(42).pipe(Micro.delay(1000), Micro.runFork)

// Attach an observer to log the result when the effect completes
fiber.addObserver((result) => {
console.log(result)
})

console.log("observing...")
/* 
Output:
observing...
{
  "_id": "MicroExit",
  "_tag": "Success",
  "value": 42
}
*/
```

---

### Create Successful Operation: Promise vs. Effect

Source: https://effect.website/docs/additional-resources/effect-vs-promise

Demonstrates how to create a successful asynchronous operation using both JavaScript's native Promise and the Effect library. Effect.succeed is used for success cases.

```javascript
const success = Promise.resolve(2)
```

```typescript
import { Effect } from "effect"

const success = Effect.succeed(2)
```

---

### Using the Brand Interface from the Brand Module in TypeScript

Source: https://effect.website/docs/code-style/branded-types

This example demonstrates the practical application of the `effect/Brand` module by importing and using its `Brand` interface. It simplifies the creation of branded types like `ProductId` and `UserId` by providing a standardized and type-safe way to add unique identifiers.

```typescript
import { Brand } from "effect"

// Define a ProductId type branded with a unique identifier
type ProductId = number & Brand.Brand<{
  readonly ProductId: "ProductId"
}>

// Define a UserId type branded similarly
type UserId = number & Brand.Brand<{
  readonly UserId: "UserId"
}>
```

---

### Conditional Effect Execution with Effect.whenEffect in TypeScript

Source: https://effect.website/docs/getting-started/control-flow

Illustrates executing an effect conditionally based on the result of another effect using `Effect.whenEffect`. In this example, `Random.nextInt` is executed only if `Random.nextBoolean` returns true. Requires 'effect' and 'effect/random' libraries.

```typescript
import { Effect, Random } from "effect"

const randomIntOption = Random.nextInt.pipe(
  Effect.whenEffect(Random.nextBoolean)
)

console.log(Effect.runSync(randomIntOption))
/* 
Example Output:
{
  _id: 'Option',
  _tag: 'Some',
  value: 8609104974198840
}
*/
```

---

### Cleanup Interrupted Effect.async Operations in JavaScript

Source: https://effect.website/docs/getting-started/creating-effects

Shows an advanced usage of Effect.async where the returned Effect handles resource cleanup upon fiber interruption. This example wraps file writing and includes a cleanup effect that deletes the file if the operation is interrupted, ensuring resources are released.

```javascript
import { Effect, Fiber } from "effect"
import * as NodeFS from "node:fs"

// Simulates a long-running operation to write to a file
const writeFileWithCleanup = (filename, data) =>
  Effect.async((resume) => {
    const writeStream = NodeFS.createWriteStream(filename)

    // Start writing data to the file
    writeStream.write(data)

    // When the stream is finished, resume with success
    writeStream.on("finish", () => resume(Effect.void))

    // In case of an error during writing, resume with failure
    writeStream.on("error", (err) => resume(Effect.fail(err)))

    // Handle interruption by returning a cleanup effect
    return Effect.sync(() => {
      console.log(`Cleaning up ${filename}`)
      NodeFS.unlinkSync(filename)
    })
  })

const program = Effect.gen(function* () {
  const fiber = yield* Effect.fork(
    writeFileWithCleanup("example.txt", "Some long data...")
  )
  // Simulate interrupting the fiber after 1 second
  yield* Effect.sleep("1 second")
  yield* Fiber.interrupt(fiber) // This will trigger the cleanup
})

// Run the program
Effect.runPromise(program)
/*
Output:
Cleaning up example.txt
*/
```

---

### Effect.gen with Control Flow (TypeScript)

Source: https://effect.website/docs/getting-started/using-generators

Illustrates how to use standard control flow statements like `while` and `if` within an Effect.gen generator function. This example calculates tax for even numbers within a loop until a counter reaches 10, demonstrating complex logic handling.

```typescript
import { Effect, Console } from "effect"

const calculateTax = (
  amount: number,
  taxRate: number
): Effect.Effect =>
  taxRate > 0
    ? Effect.succeed((amount * taxRate) / 100)
    : Effect.fail(new Error("Invalid tax rate"))

const program = Effect.gen(function* () {
  let i = 1

  while (true) {
    if (i === 10) {
      break // Break the loop when counter reaches 10
    } else {
      if (i % 2 === 0) {
        // Calculate tax for even numbers
        console.log(yield* calculateTax(100, i))
      }
      i++
      continue
    }
  }
})

Effect.runPromise(program)
```

---

### Get First Some Value from Iterable - Option.firstSomeOf

Source: https://effect.website/docs/data-types/option

The `Option.firstSomeOf` function retrieves the first `Some` value from an iterable of `Option` values. If no `Some` value is found, it returns `None`. This is useful for efficiently processing collections where you only need the first valid result.

```typescript
import { Option } from "effect"

const first = Option.firstSomeOf([
  Option.none(),
  Option.some(2),
  Option.none(),
  Option.some(3)
])

console.log(first)
// Output: { _id: 'Option', _tag: 'Some', value: 2 }
```

---

### Compose Program for Notifying Todo Owners (TypeScript)

Source: https://effect.website/docs/batching

Illustrates a composed Effect program using `Effect.gen` and `Effect.forEach` to fetch all todos and then notify the owner of each todo. This program leverages the previously defined queries and resolvers for efficient batching.

```typescript
const program = Effect.gen(function* () {
  const todos = yield* getTodos
  yield* Effect.forEach(todos, (todo) => notifyOwner(todo), { batching: true })
})
```

---

### Get Redacted Equivalence - Effect.js

Source: https://effect.website/docs/data-types/redacted

The Redacted.getEquivalence function generates an Equivalence for Redacted values using an Equivalence for the underlying values of type A. This allows for comparing Redacted values based on their underlying content. It requires an Equivalence for type A as input.

```typescript
import { Redacted, Equivalence } from "effect"

const stringEquivalence: Equivalence<string> = (x, y) => x === y
const redactedEquivalence = Redacted.getEquivalence(stringEquivalence)

const redacted1 = Redacted.make("sensitive")
const redacted2 = Redacted.make("sensitive")
const redacted3 = Redacted.make("different")

console.log(redactedEquivalence(redacted1, redacted2)) // true
console.log(redactedEquivalence(redacted1, redacted3)) // false
```

---

### Create Effect Request with Context-Aware Resolver

Source: https://effect.website/docs/batching

Shows how to create an Effect.request that requires the HttpService context, using a resolver that has been explicitly configured to depend on it. The resulting Effect type reflects the dependency on HttpService and the minimal context required by the resolver.

```typescript
const getTodos: Effect.Effect<Array<Todo>, GetTodosError | GetUserError | SendEmailError, HttpService> = Effect.request(GetTodos({}), GetTodosResolver)
```

---

### Async/Await Equivalence: Promise vs. Effect.gen

Source: https://effect.website/docs/additional-resources/effect-vs-promise

Compares the syntax and flow of Promise-based async/await with Effect.gen. While syntactically similar, Effect provides static error and context tracking.

```javascript
const increment = (x: number) => x + 1

const divide = (a: number, b: number): Promise => 
  b === 0 
  ? Promise.reject(new Error("Cannot divide by zero")) 
  : Promise.resolve(a / b)

const task1 = Promise.resolve(10)
const task2 = Promise.resolve(2)

const program = async function () {
  const a = await task1
  const b = await task2
  const n1 = await divide(a, b)
  const n2 = increment(n1)
  return `Result is: ${n2}`
}

program().then(console.log) // Output: "Result is: 6"
```

```typescript
import { Effect } from "effect"

const increment = (x: number) => x + 1

const divide = (a: number, b: number): Effect.Effect => 
  b === 0 
  ? Effect.fail(new Error("Cannot divide by zero")) 
  : Effect.succeed(a / b)

const task1 = Effect.promise(() => Promise.resolve(10))
const task2 = Effect.promise(() => Promise.resolve(2))

const program = Effect.gen(function* () {
  const a = yield* task1
  const b = yield* task2
  const n1 = yield* divide(a, b)
  const n2 = increment(n1)
  return `Result is: ${n2}`
})

Effect.runPromise(program).then(console.log)
// Output: "Result is: 6"
```

---

### Effect.all for Combining Effects in TypeScript

Source: https://effect.website/docs/getting-started/building-pipelines

Demonstrates using Effect.all to run multiple effects concurrently or sequentially and combine their results into a single structure like a tuple or object. If any effect fails, Effect.all short-circuits and propagates the error.

```typescript
import { Effect } from "effect"

// Simulated function to read configuration from a file
const webConfig = Effect.promise(() => 
  Promise.resolve({ dbConnection: "localhost", port: 8080 })
)

// Simulated function to check database status
const checkDatabase = Effect.promise(() => 
  Promise.resolve(true)
)

// Combining configuration and database check effects
const combinedEffects = Effect.all([
  webConfig,
  checkDatabase
])

// If webConfig was an object and checkDatabase was another object, you could use Effect.all with an object structure too:
// const combinedEffectsObject = Effect.all({
//   config: webConfig,
//   dbStatus: checkDatabase
// })

```

---

### Default Value When Field is Missing (Exact)

Source: https://effect.website/docs/schema/advanced-usage

This example demonstrates using the 'exact: true' option with 'optionalWith'. The default value (1) is applied only if the 'quantity' field is completely absent in the input. If 'quantity' is 'undefined', it results in a parsing error, as shown in the commented-out output.

```typescript
import { Schema } from "effect"

const Product = Schema.Struct({
  quantity: Schema.optionalWith(Schema.NumberFromString, {
    default: () => 1, // Default value for quantity
    exact: true // Only apply default if quantity is not provided
  })
})

// ┌─── { readonly quantity?: string; }
// ▼
type Encoded = typeof Product.Encoded

// ┌─── { readonly quantity: number; }
// ▼
type Type = typeof Product.Type

console.log(Schema.decodeUnknownSync(Product)({}))
// Output: { quantity: 1 }

console.log(Schema.decodeUnknownSync(Product)({ quantity: "2" }))
// Output: { quantity: 2 }

console.log(Schema.decodeUnknownSync(Product)({ quantity: undefined }))
/* 
throws:
ParseError: (Struct (Encoded side) <-> Struct (Type side))
└─ Encoded side transformation failure
└─ Struct (Encoded side)
└─ ["quantity"]
└─ NumberFromString
└─ Encoded side transformation failure
└─ Expected string, actual undefined
*/
```

---

### Default Value When Field is Missing, Null, or Undefined (Nullable)

Source: https://effect.website/docs/schema/advanced-usage

This example uses the 'nullable: true' option with 'optionalWith'. The default value (1) is applied if the 'quantity' field is missing, 'null', or 'undefined'. This provides flexibility in handling various forms of absent or nullish input.

```typescript
import { Schema } from "effect"

const Product = Schema.Struct({
  quantity: Schema.optionalWith(Schema.NumberFromString, {
    default: () => 1, // Default value for quantity
    nullable: true // Apply default if quantity is null
  })
})

// ┌─── { readonly quantity?: string | null | undefined; }
// ▼
type Encoded = typeof Product.Encoded

// ┌─── { readonly quantity: number; }
// ▼
type Type = typeof Product.Type

console.log(Schema.decodeUnknownSync(Product)({}))
// Output: { quantity: 1 }

console.log(Schema.decodeUnknownSync(Product)({ quantity: undefined }))
// Output: { quantity: 1 }

console.log(Schema.decodeUnknownSync(Product)({ quantity: null }))
// Output: { quantity: 1 }

console.log(Schema.decodeUnknownSync(Product)({ quantity: "2" }))
// Output: { quantity: 2 }
```

---

### Sequencing Operations with Effect.andThen

Source: https://effect.website/docs/getting-started/building-pipelines

Illustrates Effect.andThen for executing operations sequentially, where the second operation can depend on the result of the first. It supports various types for the second operation, including values, Promises, and Effects, similar to Effect.as, Effect.map, and Effect.flatMap. This method is ideal for ordered computations.

```typescript
import { pipe, Effect } from "effect"

// Function to apply a discount safely to a transaction amount
const applyDiscount = (
  total: number,
  discountRate: number
): Effect.Effect =>
  discountRate === 0
    ? Effect.fail(new Error("Discount rate cannot be zero"))
    : Effect.succeed(total - (total * discountRate) / 100)

// Simulated asynchronous task to fetch a transaction amount from database
const fetchTransactionAmount = Effect.promise(() => Promise.resolve(100))

// Using Effect.map and Effect.flatMap for comparison
const result1 = pipe(
  fetchTransactionAmount,
  Effect.map((amount) => amount * 2),
  Effect.flatMap((amount) => applyDiscount(amount, 5))
)

Effect.runPromise(result1).then(console.log) // Output: 190

// Using Effect.andThen for sequencing
const result2 = pipe(
  fetchTransactionAmount,
  Effect.andThen((amount) => amount * 2),
  Effect.andThen((amount) => applyDiscount(amount, 5))
)

Effect.runPromise(result2).then(console.log) // Output: 190
```

---

### Concurrent Execution: Promise.all vs. Effect

Source: https://effect.website/docs/additional-resources/effect-vs-promise

Compares running multiple asynchronous operations concurrently using Promise.all and Effect's concurrency primitives. Demonstrates handling of both success and failure in parallel tasks.

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

const program = Promise.all([task1, task2, task3])

program.then(console.log, console.error)
/* 
Output:
Executing task1...
Executing task2...
Executing task3...
task1 done
task2 done
Uh oh!
task3 done 
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

const program = Effect.all([task1, task2, task3])

Effect.runPromise(program).then(console.log).catch(console.error)
```

---

### Using Generators for Effects

Source: https://context7_llms

Shows how to use generators within Effect for a more sequential and readable way to write effectful code, similar to async/await syntax.

```typescript
import * as Effect from "@effect/io/Effect"

const myGeneratorEffect = Effect.gen(function* (_) {
  const result1 = yield* _(Effect.succeed(10))
  const result2 = yield* _(Effect.succeed(20))
  return result1 + result2
})

Effect.runSync(myGeneratorEffect) // 30
```

---

### Default Value When Field is Missing or Null (Exact and Nullable)

Source: https://effect.website/docs/schema/advanced-usage

This example combines both 'exact: true' and 'nullable: true' options. The default value (1) is applied if the 'quantity' field is missing or explicitly 'null'. However, if 'quantity' is set to 'undefined', it will result in a parsing error, enforcing stricter input validation.

```typescript
import { Schema } from "effect"

const Product = Schema.Struct({
  quantity: Schema.optionalWith(Schema.NumberFromString, {
    default: () => 1, // Default value for quantity
    exact: true, // Only apply default if quantity is not provided
    nullable: true // Apply default if quantity is null
  })
})

// ┌─── { readonly quantity?: string | null; }
// ▼
type Encoded = typeof Product.Encoded

// ┌─── { readonly quantity: number; }
// ▼
type Type = typeof Product.Type

console.log(Schema.decodeUnknownSync(Product)({}))
// Output: { quantity: 1 }

console.log(Schema.decodeUnknownSync(Product)({ quantity: null }))
// Output: { quantity: 1 }

console.log(Schema.decodeUnknownSync(Product)({ quantity: "2" }))
// Output: { quantity: 2 }

console.log(Schema.decodeUnknownSync(Product)({ quantity: undefined }))
/* 
throws:
ParseError: (Struct (Encoded side) <-> Struct (Type side))
└─ Encoded side transformation failure
└─ Struct (Encoded side)
└─ ["quantity"]
└─ NumberFromString
└─ Encoded side transformation failure
└─ Expected string, actual undefined
*/
```

---

### Basic Effect Creation

Source: https://context7_llms

Demonstrates the fundamental way to create an Effect in TypeScript. Effects represent lazy, immutable computations that can encapsulate success, failure, and side effects.

```typescript
import * as Effect from "@effect/io/Effect"

const myEffect = Effect.sync(() => {
  console.log("This is a side effect!")
  return 42
})

console.log(myEffect) // Effect { _id: "Effect", ... }
```
