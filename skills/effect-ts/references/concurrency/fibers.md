# Effect – Fibers

Forking, joining, racing, `Deferred`, `Latch`, `Supervisor`, and fiber interruption.

---

### EffectJS Forking a Fiber Example

Source: https://effect.website/docs/micro/new-users

Demonstrates the fundamental way to create a new fiber using `Micro.fork`. This starts an effect on a separate fiber, returning a reference to the new fiber. The example shows forking a recursive Fibonacci calculation.

```typescript
import { Micro } from "effect"

const fib = (n: number): Micro.Micro =>
  n < 2 
  ? Micro.succeed(n)
  : Micro.zipWith(fib(n - 1), fib(n - 2), (a, b) => a + b)

const fib10Fiber = Micro.fork(fib(10))
```

---

### Delay Allows Fiber to Capture All Values - Effect-TS

Source: https://effect.website/docs/concurrency/fibers

This example demonstrates how introducing a delay with Effect.sleep allows a forked fiber to start in time to capture all values from a SubscriptionRef. It contrasts with the previous example where the fiber started too late. Dependencies include Effect, SubscriptionRef, Stream, and Console.

```typescript
import { Effect, SubscriptionRef, Stream, Console } from "effect"

const program = Effect.gen(function* () {
  const ref = yield* SubscriptionRef.make(0)
  yield* ref.changes.pipe(
    // Log each change in SubscriptionRef
    Stream.tap((n) => Console.log(`SubscriptionRef changed to ${n}`)),
    Stream.runDrain,
    // Fork a fiber to run the stream
    Effect.fork
  )

  // Allow the fiber a chance to start
  yield* Effect.sleep("100 millis")

  yield* SubscriptionRef.set(ref, 1)
  yield* SubscriptionRef.set(ref, 2)
})

Effect.runFork(program)
```

---

### Async Promise Equivalent with Effect-TS Fork

Source: https://effect.website/docs/additional-resources/effect-vs-promise

This example shows how to replicate the behavior of starting a Promise without immediately awaiting it in Effect-TS. It uses `Effect.fork` to run an effect concurrently and `Fiber.join` to retrieve its result later, mimicking the non-blocking nature of `new Promise()`.

```typescript
import { Effect, Fiber } from "effect"

const task = (delay: number, name: string) =>
  Effect.gen(function* () {
    yield* Effect.sleep(delay)
    console.log(`${name} done`)
    return name
  })

const program = Effect.gen(function* () {
  const r0 = yield* Effect.fork(task(2000, "long running task"))
  const r1 = yield* task(200, "task 2")
  const r2 = yield* task(100, "task 3")
  return {
    r1,
    r2,
    r0: yield* Fiber.join(r0)
  }
})

Effect.runPromise(program).then(console.log)
```

---

### Late Fiber Start Captures Only One Value - Effect-TS

Source: https://effect.website/docs/concurrency/fibers

This example demonstrates a scenario where a forked fiber, started after a value update, only captures a single value from a SubscriptionRef. It highlights the importance of fiber timing. Dependencies include Effect, SubscriptionRef, Stream, and Console.

```typescript
import { Effect, SubscriptionRef, Stream, Console } from "effect"

const program = Effect.gen(function* () {
  const ref = yield* SubscriptionRef.make(0)
  yield* ref.changes.pipe(
    // Log each change in SubscriptionRef
    Stream.tap((n) => Console.log(`SubscriptionRef changed to ${n}`)),
    Stream.runDrain,
    // Fork a fiber to run the stream
    Effect.fork
  )
  yield* SubscriptionRef.set(ref, 1)
  yield* SubscriptionRef.set(ref, 2)
})

Effect.runFork(program)
```

---

### Observe Fiber Completion Without Interruption (TypeScript)

Source: https://effect.website/docs/micro/new-users

This example shows a program running to completion without any interruption. `Micro.runPromiseExit` is used to execute the program and log its final `MicroExit` status, which in this case is 'Success'.

```typescript
import { Micro } from "effect"

const program = Micro.gen(function* () {
  console.log("start")
  yield* Micro.sleep(2000)
  console.log("done")
})

Micro.runPromiseExit(program).then(console.log)
/*
Output:
start
done
{
  "_id": "MicroExit",
  "_tag": "Success"
}
*/
```

---

### Forking an Effect into a New Fiber (TypeScript)

Source: https://effect.website/docs/concurrency/fibers

Demonstrates how to use `Effect.fork` to start an Effect computation in a new, independent fiber. This allows the computation to run concurrently and returns a `Fiber` object that can be used to interact with the running effect.

```typescript
import { Effect } from "effect"

const fib = (n: number): Effect.Effect =>
  n < 2 
    ? Effect.succeed(n)
    : Effect.zipWith(fib(n - 1), fib(n - 2), (a, b) => a + b)

// Fork the fib(10) computation into a new fiber
const fib10Fiber = Effect.fork(fib(10))
```

---

### Coordinating Two Fibers with Deferred

Source: https://effect.website/docs/concurrency/deferred

An example demonstrating how to use a Deferred to coordinate two fibers, allowing one fiber to signal completion and pass a value to another fiber that is waiting.

```typescript
import { Effect, Deferred, Fiber } from "effect"

const program = Effect.gen(function* () {
  const deferred = yield* Deferred.make()

  // Completes the Deferred with a value after a delay
  const taskA = Effect.gen(function* () {
    console.log("Starting task to complete the Deferred")
    yield* Effect.sleep("1 second")
    console.log("Completing the Deferred")
    return yield* Deferred.succeed(deferred, "hello world")
  })

  // Waits for the Deferred and prints the value
  const taskB = Effect.gen(function* () {
```

---

### Run Daemon Fiber Independently with Effect.forkDaemon

Source: https://effect.website/docs/concurrency/fibers

This example demonstrates how to fork a daemon fiber using `Effect.forkDaemon`. Daemon fibers run independently of their parent fiber and continue to execute even after the parent has finished or been interrupted. It requires the 'effect' library.

```typescript
import { Effect, Console, Schedule } from "effect"

// Daemon fiber that logs a message repeatedly every second
const daemon = Effect.repeat(
  Console.log("daemon: still running!"),
  Schedule.fixed("1 second")
)

const parent = Effect.gen(function* () {
  console.log("parent: started!")
  // Daemon fiber running independently
  yield* Effect.forkDaemon(daemon)
  yield* Effect.sleep("3 seconds")
  console.log("parent: finished!")
})

Effect.runFork(parent)
```

---

### Awaiting Fiber Completion with Exit Status (TypeScript)

Source: https://effect.website/docs/concurrency/fibers

Shows how to use `Fiber.await` to wait for a fiber to finish and get an `Exit` value. The `Exit` type provides detailed information about whether the fiber succeeded, failed, or was interrupted.

```typescript
import { Effect, Fiber } from "effect"

const fib = (n: number): Effect.Effect =>
  n < 2 
    ? Effect.succeed(n)
    : Effect.zipWith(fib(n - 1), fib(n - 2), (a, b) => a + b)

const fib10Fiber = Effect.fork(fib(10))

const program = Effect.gen(function* () {
  // Retrieve the fiber reference
  const fiber = yield* fib10Fiber
  // Await its completion and get the Exit result
  const exit = yield* Fiber.await(fiber)
  console.log(exit)
})

Effect.runFork(program) 
/* 
Output:
{
  _id: 'Exit',
  _tag: 'Success',
  value: 55
}
*/
```

---

### Deferred Status Checks

Source: https://effect.website/docs/concurrency/deferred

Explains how to check the status of a Deferred without suspending, using `Deferred.poll` to get its current state or `Deferred.isDone` for a simple boolean check.

```APIDOC
## Deferred Status Checks

### Description

This section covers methods for inspecting the state of a `Deferred` without causing the current fiber to suspend. This is useful for scenarios where you need to check if a `Deferred` has been completed or retrieve its result non-blockingly.

### Methods

- `Deferred.poll(deferred)`: Returns an `Option<Exit>` of the `Deferred`'s result. Returns `None` if the `Deferred` is incomplete, and `Some(exit)` if it is complete.
- `Deferred.isDone(deferred)`: Returns an `Effect<boolean>` that evaluates to `true` if the `Deferred` has been completed, and `false` otherwise.

### Example (Polling and Checking Completion Status)

```typescript
import { Effect, Deferred } from "effect";

const program = Effect.gen(function* () {
  const deferred = yield* Deferred.make<number, string>();

  // Polling the Deferred to check if it's completed
  const done1 = yield* Deferred.poll(deferred);

  // Checking if the Deferred has been completed
  const done2 = yield* Deferred.isDone(deferred);

  console.log([done1, done2]);
});

Effect.runFork(program);
/*
Output:
[
  { _id: 'Option', _tag: 'None' },
  false
]
*/
```
```

---

### Forking Effects

Source: https://effect.website/docs/micro/new-users

Demonstrates how to create a new fiber by forking an existing effect using `Micro.fork`.

```APIDOC
## Forking Effects

### Description
One of the fundamental ways to create a fiber is by forking an existing effect. When you fork an effect, it starts executing the effect on a new fiber, giving you a reference to this newly-created fiber.

### Method
`Micro.fork(effect)`

### Parameters
#### Path Parameters
None

#### Query Parameters
None

#### Request Body
None

### Request Example
```typescript
import { Micro } from "effect"

const fib = (n: number): Micro.Micro<number, never, never> =>
  n < 2
    ? Micro.succeed(n)
    : Micro.zipWith(fib(n - 1), fib(n - 2), (a, b) => a + b)

// ┌─── Micro, never, never>
// ▼
const fib10Fiber = Micro.fork(fib(10))
```

### Response
#### Success Response (200)
- **Fiber<any, any>** - A reference to the newly created fiber.

#### Response Example
(No specific JSON response for this operation as it returns a Fiber object)
```

---

### Supervise and Track Fibers with Supervisor

Source: https://effect.website/docs/observability/supervisor

This example demonstrates how to use `Supervisor.track` to create a supervisor that tracks child fibers. The `Effect.supervised` function then applies this supervisor to an effect, ensuring that all fibers forked within it are managed by the supervisor. This allows for monitoring of fiber status and count.

```typescript
import { Effect, Supervisor, Schedule, Fiber, FiberStatus } from "effect"

// Main program that monitors fibers while calculating a Fibonacci number
const program = Effect.gen(function* () {
  // Create a supervisor to track child fibers
  const supervisor = yield* Supervisor.track

  // Start a Fibonacci calculation, supervised by the supervisor
  const fibFiber = yield* fib(20).pipe(
    Effect.supervised(supervisor),
    // Fork the Fibonacci effect into a fiber
    Effect.fork
  )

  // Define a schedule to periodically monitor the fiber count every 500ms
  const policy = Schedule.spaced("500 millis").pipe(
    Schedule.whileInputEffect((_) =>
      Fiber.status(fibFiber).pipe(
        // Continue while the Fibonacci fiber is not done
        Effect.andThen((status) => status !== FiberStatus.done)
      )
    )
  )

  // Start monitoring the fibers, using the supervisor to track the count
  const monitorFiber = yield* monitorFibers(supervisor).pipe(
    // Repeat the monitoring according to the schedule
    Effect.repeat(policy)
    // Fork the monitoring effect into a fiber
    // Effect.fork
  )
})

// Placeholder for fib function (implementation not provided in the original text)
const fib = (n: number): Effect<number, never, never> => {
  return Effect.succeed(0);
}

// Placeholder for monitorFibers function (implementation not provided in the original text)
const monitorFibers = (supervisor: Supervisor<unknown>): Effect<void, never, never> => {
  return Effect.gen(function* () {
    const fiberCount = yield* supervisor.count
    console.log(`Current fiber count: ${fiberCount}`)
  })
}
```

---

### Run Effect-TS RaceAll with Either Error Handling

Source: https://effect.website/docs/additional-resources/effect-vs-promise

Demonstrates how to use `Effect.raceAll` to run multiple effects concurrently and handle their outcomes using `Effect.either`. This is useful for scenarios where you want to get the result of the first effect to complete, while also capturing any errors from other effects.

```typescript
import { Effect } from "effect"

const task1 = Effect.gen(function* () {
  console.log("Executing task1...")
  yield* Effect.sleep("100 millis")
  console.log("task1 done")
  return yield* Effect.fail("Something went wrong!")
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

const program = Effect.raceAll([task1, task2, task3].map(Effect.either)) // or Effect.exit

Effect.runPromise(program).then(console.log, console.error)
```

---

### Promise.any() vs. Effect.raceAll()

Source: https://effect.website/docs/additional-resources/effect-vs-promise

This snippet demonstrates how to handle an array of asynchronous tasks and return the result of the first one that fulfills. It compares the native Promise.any with the Effect library's Effect.raceAll, which achieves a similar outcome by returning the result of the first task to succeed.

```javascript
const task1 = new Promise((resolve, reject) => {
  console.log("Executing task1...")
  setTimeout(() => {
    console.log("task1 done")
    reject("Something went wrong!")
  }, 100)
})

const task2 = new Promise((resolve, reject) => {
  console.log("Executing task2...")
  setTimeout(() => {
    console.log("task2 done")
    resolve(2)
  }, 200)
})

const task3 = new Promise((resolve, reject) => {
  console.log("Executing task3...")
  setTimeout(() => {
    console.log("task3 done")
    reject("Uh oh!")
  }, 300)
})

const program = Promise.any([task1, task2, task3])

program.then(console.log, console.error)
/* 
Output:
Executing task1...
Executing task2...
Executing task3...
task1 done
task2 done
2
task3 done
*/
```

```typescript
import { Effect } from "effect"

const task1 = Effect.gen(function* () {
  console.log("Executing task1...")
  yield* Effect.sleep("100 millis")
  console.log("task1 done")
  return yield* Effect.fail("Something went wrong!")
})

const task2 = Effect.gen(function* () {
  console.log("Executing task2...")
  yield* Effect.sleep("200 millis")
  console.log("task2 done")
  return 2
})

const task3 = Effect.gen(function* () {
  console.log("Executing task3...")
  yield* Effect.sleep("300 millis")
  console.log("task3 done")
  return yield* Effect.fail("Uh oh!")
})

const program = Effect.raceAll([task1, task2, task3])

Effect.runPromise(program).then(console.log, console.error)
/* 
Output:
Executing task1...
Executing task2...
Executing task3...
task1 done
task2 done
2
*/
```

---

### Promise.race() vs. Effect.race()

Source: https://effect.website/docs/additional-resources/effect-vs-promise

This snippet demonstrates the behavior of Promise.race and its Effect library equivalent, Effect.race. Both methods return the result of the first promise/effect to settle, whether it fulfills or rejects. This is useful for scenarios where you want to get a result as quickly as possible and don't need to wait for all operations.

```javascript
const task1 = new Promise((resolve, reject) => {
  console.log("Executing task1...")
  setTimeout(() => {
    console.log("task1 done")
    reject("Something went wrong!")
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

const program = Promise.race([task1, task2, task3])

program.then(console.log, console.error)
/* 
Output:
Executing task1...
Executing task2...
Executing task3...
task1 done
Something went wrong!
*/
```

```typescript
import { Effect } from "effect"

const task1 = Effect.gen(function* () {
  console.log("Executing task1...")
  yield* Effect.sleep("100 millis")
  console.log("task1 done")
  return yield* Effect.fail("Something went wrong!")
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

const program = Effect.race([task1, task2, task3])

Effect.runPromise(program).then(console.log, console.error)
/* 
Output:
Executing task1...
Executing task2...
Executing task3...
task1 done
Something went wrong!
*/
```

---

### Deferred Common Use Cases

Source: https://effect.website/docs/concurrency/deferred

Illustrates practical applications of the Deferred primitive, such as coordinating concurrent fibers, achieving synchronization, handing over work between fibers, and suspending execution.

```APIDOC
## Common Use Cases

### Description

The `Deferred` primitive is particularly useful for managing asynchronous operations and coordinating the execution of concurrent fibers. It acts as a synchronization point, allowing one fiber to signal completion or the availability of a result to other fibers.

### Use Cases

- **Coordinating Fibers**: Facilitates communication and synchronization between multiple concurrent tasks, enabling one fiber to signal another when a specific operation is complete.
- **Synchronization**: Ensures that a piece of code does not proceed until another piece of code has finished its work, providing necessary synchronization.
- **Handing Over Work**: Allows one fiber to prepare data or perform an initial task, and then hand over the continuation of the work to another fiber.
- **Suspending Execution**: Enables a fiber to pause its execution until a certain condition is met or a specific event occurs, signaled via a `Deferred`.

### Example (Using Deferred to Coordinate Two Fibers)

This example demonstrates how to use a `Deferred` to pass a value between two concurrently running fibers, ensuring that `fiberB` waits for `fiberA` to complete its task.

```typescript
import { Effect, Deferred, Fiber } from "effect";

const program = Effect.gen(function* () {
  const deferred = yield* Deferred.make<string>();

  // Completes the Deferred with a value after a delay
  const taskA = Effect.gen(function* () {
    console.log("Starting task to complete the Deferred");
    yield* Effect.sleep("1 second");
    console.log("Completing the Deferred");
    return yield* Deferred.succeed(deferred, "hello world");
  });

  // Waits for the Deferred and prints the value
  const taskB = Effect.gen(function* () {
    console.log("Waiting for the Deferred to be completed...");
    const value = yield* Deferred.await(deferred);
    console.log("Deferred completed with value:", value);
    return value;
  });

  // Run both tasks concurrently
  const fiberA = yield* Fiber.fork(taskA);
  const fiberB = yield* Fiber.fork(taskB);

  // Optionally, await both fibers to ensure they complete
  yield* Fiber.join(fiberA);
  yield* Fiber.join(fiberB);
});

Effect.runFork(program);
```
```

---

### Effect.js: Forking and Waiting on a Latch

Source: https://effect.website/docs/concurrency/latch

This snippet demonstrates forking an Effect into a new fiber, yielding to allow it to run, and then waiting for a latch to open before proceeding. It also shows how to wait for the forked fiber to complete.

```javascript
import * as Effect from "@effect/core/io/Effect"
import * as Fiber from "@effect/core/io/Fiber"
import * as Ref from "@effect/core/io/Ref"
import * as Layer from "@effect/core/io/Layer"
import { pipe } from "@effect/data/Function"
import * as Synchronized from "@effect/core/io/Cause/errors"
import { Clock } from "@effect/core/io/Clock"

// Assume latch and fiber are defined elsewhere, e.g.:
// const latch = await Effect.runPromise(Synchronized.newLatch(0))
// const fiber = await Effect.runPromise(Effect.fork(someEffect))

const program = Effect.gen(function* (_) {
  const latch = yield* _(Synchronized.newLatch(0))
  const fiber = yield* _(Effect.fork(Effect.delay(1000)(Effect.succeed("open sesame"))))

  // Wait for the latch to open
  yield* _(latch.await)

  // Wait for the forked fiber to finish
  const result = yield* _(fiber.await)
  console.log(result)
})

// To run this example, you would need to implement the latch and fiber creation
// For demonstration purposes, let's simulate the latch opening after a delay

const runExample = Effect.gen(function* (_)
{
  const latch = yield* _(Synchronized.newLatch(0))
  const fiber = yield* _(Effect.fork(Effect.delay(1000)(Effect.succeed("open sesame"))))

  // Simulate opening the latch after a delay
  yield* _(Effect.delay(500)(latch.open))

  yield* _(fiber.await)
})

// Effect.runFork(runExample)

```

---

### Combine Fibers with Fiber.zip in Effect.js

Source: https://effect.website/docs/concurrency/fibers

Demonstrates how to fork two fibers concurrently and combine their results into a tuple using Fiber.zip. This is useful when you need to run multiple independent effects in parallel and await all their results.

```typescript
import { Effect, Fiber } from "effect"

const program = Effect.gen(function* () {
  // Fork two fibers that each produce a string
  const fiber1 = yield* Effect.fork(Effect.succeed("Hi!"))
  const fiber2 = yield* Effect.fork(Effect.succeed("Bye!"))

  // Combine the two fibers using Fiber.zip
  const fiber = Fiber.zip(fiber1, fiber2)

  // Join the combined fiber and get the result as a tuple
  const tuple = yield* Fiber.join(fiber)
  console.log(tuple)
})

Effect.runFork(program)
/* 
Output: 
[ 'Hi!', 'Bye!' ] 
*/
```

---

### Interrupt Parent Fiber Without Affecting Daemon Fiber

Source: https://effect.website/docs/concurrency/fibers

This example shows how interrupting the parent fiber does not affect the daemon fiber, which continues to run in the background. It uses `Effect.forkDaemon` for the background task and `Effect.onInterrupt` to handle parent fiber interruption. It requires the 'effect' library.

```typescript
import { Effect, Console, Schedule, Fiber } from "effect"

// Daemon fiber that logs a message repeatedly every second
const daemon = Effect.repeat(
  Console.log("daemon: still running!"),
  Schedule.fixed("1 second")
)

const parent = Effect.gen(function* () {
  console.log("parent: started!")
  // Daemon fiber running independently
  yield* Effect.forkDaemon(daemon)
  yield* Effect.sleep("3 seconds")
  console.log("parent: finished!")
}).pipe(Effect.onInterrupt(() => Console.log("parent: interrupted!")))

// Program that interrupts the parent fiber after 2 seconds
const program = Effect.gen(function* () {
  const fiber = yield* Effect.fork(parent)
  yield* Effect.sleep("2 seconds")
  yield* Fiber.interrupt(fiber) // Interrupt the parent fiber
})

Effect.runFork(program)
```

---

### Create and Use Latch - Effect TypeScript

Source: https://effect.website/docs/concurrency/latch

Demonstrates creating a Latch in a closed state using Effect.makeLatch and then opening it to release a waiting fiber. The fiber logs a message only after the latch is opened.

```typescript
import { Console, Effect } from "effect"

// A generator function that demonstrates latch usage
const program = Effect.gen(function* () {
  // Create a latch, starting in the closed state
  const latch = yield* Effect.makeLatch()

  // Fork a fiber that logs "open sesame" only when the latch is open
  const fiber = yield* Console.log("open sesame").pipe(
    latch.await
  )

  // Wait for one second before opening the latch
  yield* Effect.sleep(1000)
  yield* latch.open()

  yield* Console.log("Latch opened!")
})

Effect.runFork(program)
```

---

### Handle Full Queue with Effect.fork (TypeScript)

Source: https://effect.website/docs/concurrency/queue

Demonstrates handling a full queue by forking the `Queue.offer` operation. This prevents blocking the main fiber when the queue is at capacity.

```typescript
import { Effect, Queue, Fiber } from "effect"

const program = Effect.gen(function* () {
  const queue = yield* Queue.bounded(1)
  // Fill the queue with one item
  yield* Queue.offer(queue, 1)
  // Attempting to add a second item will suspend as the queue is full
  const fiber = yield* Effect.fork(Queue.offer(queue, 2))
  // Empties the queue to make space
  yield* Queue.take(queue)
  // Joins the fiber, completing the suspended offer
  yield* Fiber.join(fiber)
  // Returns the size of the queue after additions
  return yield* Queue.size(queue)
})

Effect.runPromise(program).then(console.log)
// Output: 1
```

---

### Daemon Fiber with Effect.forkDaemon in Effect.js

Source: https://effect.website/docs/concurrency/fibers

Demonstrates creating a daemon fiber using `Effect.forkDaemon`. This fiber runs in the global scope, independent of its parent, and continues execution until it completes naturally or the application terminates.

```typescript
// Example for Effect.forkDaemon would go here, demonstrating a long-running background task.
```

---

### Run Concurrent Fibonacci Calculation with Fiber Monitoring

Source: https://effect.website/docs/observability/supervisor

Executes a concurrent Fibonacci calculation using Effect-TS fibers. It forks two fibers for recursive calls, joins them to get results, and logs the final Fibonacci number. A monitor function is used to log the number of active fibers at different stages, illustrating the dynamic nature of concurrency.

```typescript
import * as Effect from "@effect/io/Effect";
import * as Fiber from "@effect/io/Fiber";
import * as Supervisor from "@effect/io/Supervisor";

// Program to orchestrate the Fibonacci calculation and monitoring
const program = Effect.gen(function* () {
  // Create a supervisor to manage fibers
  const supervisor = yield* Supervisor.track();

  // Start a monitor fiber that periodically logs the number of active fibers
  const monitorFiber = yield* Effect.fork(monitorFibers(supervisor));

  // Start the Fibonacci calculation fiber, supervised
  const fibFiber = yield* Effect.fork(fib(40).pipe(Effect.supervised(supervisor)));

  // Join the monitor and Fibonacci fibers to ensure they complete
  yield* Fiber.join(monitorFiber);
  const result = yield* Fiber.join(fibFiber);

  console.log(`fibonacci result: ${result}`);
});

// Function to monitor and log the number of active fibers
const monitorFibers = (supervisor: Supervisor.Supervisor) =>
  Effect.gen(function* () {
    // Loop indefinitely to monitor fibers
    for (let i = 0; i < 10; i++) {
      const fibers = yield* supervisor.value; // Get the current set of fibers
      console.log(`number of fibers: ${fibers.length}`);
      yield* Effect.sleep("500 millis"); // Wait before checking again
    }
  });

// Recursive Fibonacci calculation, spawning fibers for each recursive step
const fib = (n: number): Effect.Effect<number> =>
  Effect.gen(function* () {
    if (n <= 1) {
      return 1;
    }
    yield* Effect.sleep("500 millis"); // Simulate work by delaying

    // Fork two fibers for the recursive Fibonacci calls
    const fiber1 = yield* Effect.fork(fib(n - 2));
    const fiber2 = yield* Effect.fork(fib(n - 1));

    // Join the fibers to retrieve their results
    const v1 = yield* Fiber.join(fiber1);
    const v2 = yield* Fiber.join(fiber2);

    return v1 + v2; // Combine the results
  });

// Run the program
Effect.runPromise(program);
```

---

### Handle results of concurrent tasks with Effect.raceWith

Source: https://effect.website/docs/concurrency/basic-concurrency

This example illustrates using Effect.raceWith to run two tasks concurrently and handle their results individually as they complete. The 'onSelfDone' and 'onOtherDone' callbacks in the finisher object allow for specific actions based on which task finishes first and its outcome. The program logs messages during task execution and upon the completion of either task.

```typescript
import { Effect, Console } from "effect"

const task1 = Effect.succeed("task1").pipe(
  Effect.delay("100 millis"),
  Effect.tap(Console.log("task1 done")),
  Effect.onInterrupt(() =>
    Console.log("task1 interrupted").pipe(Effect.delay("100 millis"))
  )
)

const task2 = Effect.succeed("task2").pipe(
  Effect.delay("200 millis"),
  Effect.tap(Console.log("task2 done")),
  Effect.onInterrupt(() =>
    Console.log("task2 interrupted").pipe(Effect.delay("100 millis"))
  )
)

const program = Effect.raceWith(task1, task2, {
  onSelfDone: (exit) => Console.log(`task1 exited with ${exit}`),
  onOtherDone: (exit) => Console.log(`task2 exited with ${exit}`)
})

Effect.runFork(program)
```
