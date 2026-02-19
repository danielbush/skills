# Effect – Resource Management

`Scope`, finalizers, `acquireUseRelease`, `Effect.scoped`, and `Effect.ensuring`.

---

### Define a Simple Resource with Acquire and Release

Source: https://effect.website/docs/micro/new-users

Demonstrates defining a resource using `Micro.acquireRelease`. It includes acquiring the resource, simulating its use, and releasing it. The `acquire` operation is uninterruptible, guaranteeing the `release` operation will run if acquisition succeeds.

```typescript
import { Micro } from "effect"

interface MyResource {
  readonly contents: string
  readonly close: () => Promise<void>
}

const getMyResource = (): Promise<MyResource> =>
  Promise.resolve({
    contents: "lorem ipsum",
    close: () =>
      new Promise((resolve) => {
        console.log("Resource released")
        resolve()
      })
  })

const acquire = Micro.tryPromise({
  try: () =>
    getMyResource().then((res) => {
      console.log("Resource acquired")
      return res
    }),
  catch: () => new Error("getMyResourceError")
})

const release = (res: MyResource) => Micro.promise(() => res.close())

const resource = Micro.acquireRelease(acquire, release)

const program = Micro.scoped(
  Micro.gen(function* () {
    const res = yield* resource
    console.log(`content is ${res.contents}`)
  })
)

Micro.runPromise(program)
/* 
Resource acquired
content is lorem ipsum
Resource released
*/
```

---

### Add Finalizer on Interruption using Effect.gen and pipe

Source: https://effect.website/docs/resource-management/scope

This example demonstrates how to add a finalizer that is executed when an Effect is interrupted. It presents the Effect.gen and pipe implementations, showing that the finalizer logs the interruption status. Similar to other examples, Effect.scoped is used to manage the scope and ensure the finalizer runs.

```typescript
import { Effect, Console } from "effect"

// Using Effect.gen
const programGen = Effect.gen(function* () {
  yield* Effect.addFinalizer((exit) =>
    Console.log(`Finalizer executed. Exit status: ${exit._tag}`)
  )
  return yield* Effect.interrupt
})

const runnableGen = Effect.scoped(programGen)
Effect.runPromiseExit(runnableGen).then(console.log)

// Using pipe
const programPipe = Effect.addFinalizer((exit) =>
  Console.log(`Finalizer executed. Exit status: ${exit._tag}`)
).pipe(
  Effect.andThen(Effect.interrupt)
)

const runnablePipe = Effect.scoped(programPipe)
Effect.runPromiseExit(runnablePipe).then(console.log)
```

---

### Fork Scoped Fiber with Effect.forkScoped

Source: https://effect.website/docs/concurrency/fibers

This example demonstrates using `Effect.forkScoped` to create a fiber that is tied to a local scope. Fibers forked with `Effect.forkScoped` can outlive their parent fibers and are only terminated when the local scope closes. It requires the 'effect' library.

```typescript
import { Effect, Console, Schedule } from "effect"

// Child fiber that logs a message repeatedly every second
const child = Effect.repeat(
  Console.log("child: still running!"),
  Schedule.fixed("1 second")
)

// ┌─── Effect
// ▼
const parent = Effect.gen(function* () {
  console.log("parent: started!")
  // Child fiber attached to local scope
  yield* Effect.forkScoped(child)
  yield* Effect.sleep("3 seconds")
  console.log("parent: finished!")
})

// Program runs within a local scope
const program = Effect.scoped(
  Effect.gen(function* () {
    console.log("Local scope started!")
    yield* Effect.fork(parent)
    // Scope lasts for 5 seconds
    yield* Effect.sleep("5 seconds")
    console.log("Leaving the local scope!")
  })
)

Effect.runFork(program)
```

---

### Fork Fiber into Specific Scope with Effect.forkIn

Source: https://effect.website/docs/concurrency/fibers

This example illustrates using `Effect.forkIn` to fork a fiber into a specific scope. This allows for more fine-grained control over the fiber's lifecycle, enabling it to outlive inner scopes but still be terminated when the target outer scope is closed. It requires the 'effect' library.

```typescript
import { Console, Effect, Schedule } from "effect"

// Child fiber that logs a message repeatedly every second
const child = Effect.repeat(
  Console.log("child: still running!"),
  Schedule.fixed("1 second")
)

const program = Effect.scoped(
  Effect.gen(function* () {
    yield* Effect.addFinalizer(() =>
      Console.log("The outer scope is about to be closed!")
    )

    // Capture the outer scope
    const outerScope = yield* Effect.scope

    // Create an inner scope
    yield* Effect.scoped(
      Effect.gen(function* () {
        yield* Effect.addFinalizer(() =>
          Console.log("The inner scope is ab
```

---

### Managing a Scope with Finalizers

Source: https://effect.website/docs/resource-management/scope

This example demonstrates how to create a new scope, add multiple finalizers to it, and then close the scope. The finalizers are executed in the reverse order of their addition when the scope is closed, ensuring proper resource cleanup.

```typescript
import { Scope, Effect, Console, Exit } from "effect"

const program = 
  // create a new scope 
  Scope.make().pipe(
    // add finalizer 1
    Effect.tap((scope) => 
      Scope.addFinalizer(scope, Console.log("finalizer 1")) 
    ),
    // add finalizer 2
    Effect.tap((scope) => 
      Scope.addFinalizer(scope, Console.log("finalizer 2")) 
    ),
    // close the scope 
    Effect.andThen((scope) => 
      Scope.close(scope, Exit.succeed("scope closed successfully")) 
    )
  )

Effect.runPromise(program)
/* 
Output:
finalizer 2 <-- finalizers are closed in reverse order
finalizer 1 
*/
```

---

### Simplify Resource Management with acquireUseRelease

Source: https://effect.website/docs/micro/new-users

The `acquireUseRelease` function simplifies resource management by automating the scope and release of resources. It takes `acquire`, `use`, and `release` functions as arguments, eliminating the need for manual scope management.

```typescript
import { Micro } from "effect"

// Define the interface for the resource
interface MyResource {
  readonly contents: string
  readonly close: () => Promise
}

// Simulate getting the resource
const getMyResource = (): Promise =>
  Promise.resolve({
    contents: "lorem ipsum",
    close: () =>
      new Promise((resolve) => {
        console.log("Resource released")
        resolve()
      })
  })

// Define the acquisition of the resource with error handling
const acquire = Micro.tryPromise({
  try: () =>
    getMyResource().then((res) => {
      console.log("Resource acquired")
      return res
    }),
  catch: () => new Error("getMyResourceError")
})

// Define the release of the resource
const release = (res: MyResource) => Micro.promise(() => res.close())

const use = (res: MyResource) =>
  Micro.sync(() => console.log(`content is ${res.contents}`)) 

// ┌─── Micro 
// ▼ 
const program = Micro.acquireUseRelease(acquire, use, release)

Micro.runPromise(program)
/* 
Resource acquired
content is lorem ipsum
Resource released 
*/
```

---

### Manage Scope and Add Finalizers in Effect-TS

Source: https://effect.website/docs/micro/new-users

Explains and demonstrates the use of `MicroScope` in Effect-TS for resource management. `MicroScope` allows you to add finalizers, which are cleanup functions guaranteed to execute when the scope is closed. This example shows how to create a scope, add two finalizers that log messages, and then close the scope.

```typescript
import { Micro } from "effect"

// Helper function to log a message
const log = (message: string) => Micro.sync(() => console.log(message))

const program = 
  // create a new scope
  Micro.scopeMake.pipe(
    // add finalizer
    Micro.tap((scope) => scope.addFinalizer(() => log("finalizer 1"))),
    // add finalizer
    Micro.tap((scope) => scope.addFinalizer(() => log("finalizer 2"))),
    // close the scope
    Micro.andThen((scope) => 
      // You can perform operations within the scope here if needed
      // For this example, we just close it implicitly by ending the pipeline
      scope
    )
  )

// To actually run and see the finalizers execute, you would typically 
// integrate this into a larger Effect program or use Micro.runPromise.
// For demonstration purposes, let's simulate closing:
// Micro.runPromise(program.pipe(Micro.provideService(Micro.rootScope, ???)))
// However, the core concept is adding finalizers to the scope.

/* 
Example Output when scope is closed:
finalizer 1
finalizer 2
*/
```

---

### Register Cleanup Action on Fiber Interruption

Source: https://effect.website/docs/concurrency/basic-concurrency

Demonstrates the use of `Effect.onInterrupt` to register a cleanup effect that runs when a fiber is interrupted. This example shows how a cleanup action is executed for an interrupted effect, but not for successful or failing effects.

```typescript
import { Console, Effect } from "effect"

// This handler is executed when the fiber is interrupted
const handler = Effect.onInterrupt((\_fibers) =>
 Console.log("Cleanup completed")
)

const success = Console.log("Task completed").pipe(
 Effect.as("some result"),
 handler
)

Effect.runFork(success)
/* 
Output:
Task completed
*/

const failure = Console.log("Task failed").pipe(
 Effect.andThen(Effect.fail("some error")),
 handler
)

Effect.runFork(failure)
/* 
Output:
Task failed
*/

const interruption = Console.log("Task interrupted").pipe(
 Effect.andThen(Effect.interrupt),
 handler
)

Effect.runFork(interruption)
/* 
Output:
Task interrupted
Cleanup completed
*/
```

---

### Sequential Errors with Effect.ensuring

Source: https://effect.website/docs/error-management/parallel-and-sequential-errors

This example illustrates handling multiple sequential errors that arise from resource-safety operators like Effect.ensuring. It shows that both the original effect's failure and the finalizer's defect are captured.

```typescript
import { Effect } from "effect"

// Simulate an effect that fails
const fail = Effect.fail("Oh uh!")

// Simulate a finalizer that causes a defect
const die = Effect.dieMessage("Boom!")

// The finalizer 'die' will always run, even if 'fail' fails
const program = fail.pipe(
  Effect.ensuring(die)
)

Effect.runPromiseExit(program).then(console.log)

/*
Output:
{
  _id: 'Exit',
  _tag: 'Failure',
  cause: {
    _id: 'Cause',
    _tag: 'Sequential',
    left: {
      _id: 'Cause',
      _tag: 'Fail',
      failure: 'Oh uh!'
    },
    right: {
      _id: 'Cause',
      _tag: 'Die',
      defect: [Object]
    }
  }
}
*/
```

---

### Manage Resource Lifetime with Effect.acquireUseRelease (TypeScript)

Source: https://effect.website/docs/resource-management/introduction

Effect.acquireUseRelease ensures resources are properly acquired, used, and released, even if errors occur. It takes three functions: `acquire` to get the resource, `use` to operate on it, and `release` to clean it up. This pattern is crucial for managing external resources like database connections or file handles.

```typescript
import { Effect, Console } from "effect"

// Define an interface for a resource
interface MyResource {
  readonly contents: string
  readonly close: () => Promise<void>
}

// Simulate resource acquisition
const getMyResource = (): Promise<MyResource> =>
  Promise.resolve({
    contents: "lorem ipsum",
    close: () =>
      new Promise((resolve) => {
        console.log("Resource released")
        resolve()
      })
  })

// Define how the resource is acquired
const acquire = Effect.tryPromise({
  try: () =>
    getMyResource().then((res) => {
      console.log("Resource acquired")
      return res
    }),
  catch: () => new Error("getMyResourceError")
})

// Define how the resource is released
const release = (res: MyResource) => Effect.promise(() => res.close())

const use = (res: MyResource) => Console.log(`content is ${res.contents}`)

// Effect
const program = Effect.acquireUseRelease(acquire, use, release)

Effect.runPromise(program)
/*
Output:
Resource acquired
content is lorem ipsum
Resource released
*/
```

---

### Manage Service Lifecycle with Scoped (Effect.js)

Source: https://effect.website/docs/requirements-management/layers

Demonstrates defining a service with a `scoped` constructor for lifecycle management. It uses `Effect.acquireRelease` to manage resource acquisition and release, and `Effect.addFinalizer` for shutdown tasks.

```typescript
import { Effect, Console } from "effect"

class Scoped extends Effect.Service()("Scoped", {
  scoped: Effect.gen(function* () {
    // Acquire the resource and ensure it is properly released
    const resource = yield* Effect.acquireRelease(
      Console.log("Aquiring...").pipe(Effect.as("foo")),
      () => Console.log("Releasing...")
    )
    // Register a finalizer to run when the effect is completed
    yield* Effect.addFinalizer(() => Console.log("Shutting down"))
    return { resource }
  })
}) {}

// Accessing the Service
const program = Effect.gen(function* () {
  const resource = (yield* Scoped).resource
  console.log(`The resource is ${resource}`)
})

Effect.runPromise(
  program.pipe(
    Effect.provide(
      Scoped.Default
    )
  )
)
/* 
Aquiring...
 The resource is foo 
 Shutting down
 Releasing...
*/
```

---

### Effect.js: Closing a Scope with Pending Tasks

Source: https://effect.website/docs/resource-management/scope

Demonstrates that closing a scope in Effect.js does not interrupt tasks within that scope that are still pending. The example shows a task with a sleep duration and a finalizer, where the task and its finalizer execute even after the scope has been closed.

```typescript
import { Console, Effect, Exit, Scope } from "effect"

const task = Effect.gen(function* () {
  yield* Effect.sleep("1 second")
  console.log("Executed")
  yield* Effect.addFinalizer(() => Console.log("Task Finalizer"))
})

const program = Effect.gen(function* () {
  const scope = yield* Scope.make()

  // Close the scope immediately
  yield* Scope.close(scope, Exit.void)
  console.log("Scope closed")

  // This task will be executed even if the scope is closed
  yield* task.pipe(Scope.extend(scope))
})

Effect.runPromise(program)
/* 
Output:
Scope closed
Executed <-- after 1 second
Task Finalizer 
*/
```

---

### Temporarily Overriding Console Service with Effect.withConsoleScoped - Effect.js

Source: https://effect.website/docs/requirements-management/default-services

This example shows how to temporarily override the Console service within a defined scope using `Effect.withConsoleScoped`. This is beneficial for isolating console interactions during tests or specific operational contexts, ensuring the original Console service is reinstated afterward.

```typescript
import { Effect, Clock, Console } from "effect"

// Assume customConsole is an implementation of the Console service
// const customConsole = { log: (message: string) => Effect.unit };

const program = Effect.gen(function* () {
  const now = yield* Clock.currentTimeMillis
  yield* Console.log(`Application started at ${new Date(now)}`)
})

// Effect.withConsoleScoped(customConsole)(program)
```

---

### Run Finalizer in All Outcomes with Effect.ensuring

Source: https://effect.website/docs/resource-management/introduction

Demonstrates how `Effect.ensuring` guarantees a finalizer effect runs regardless of whether the main effect succeeds, fails, or is interrupted. This is crucial for ensuring cleanup actions like closing file handles or logging messages are always executed. The examples cover successful, failing, and interrupted scenarios.

```typescript
import { Console, Effect } from "effect"

// Define a cleanup effect
const handler = Effect.ensuring(Console.log("Cleanup completed"))

// Define a successful effect
const success = Console.log("Task completed").pipe(
    Effect.as("some result"),
    handler
)

Effect.runFork(success)
/*
Output:
Task completed
Cleanup completed
*/

// Define a failing effect
const failure = Console.log("Task failed").pipe(
    Effect.andThen(Effect.fail("some error")),
    handler
)

Effect.runFork(failure)
/*
Output:
Task failed
Cleanup completed
*/

// Define an interrupted effect
const interruption = Console.log("Task interrupted").pipe(
    Effect.andThen(Effect.interrupt),
    handler
)

Effect.runFork(interruption)
/*
Output:
Task interrupted

```

---

### Temporarily Overriding Clock Service with Effect.withClockScoped - Effect.js

Source: https://effect.website/docs/requirements-management/default-services

This example illustrates overriding the Clock service within a specific scope using `Effect.withClockScoped`. The custom Clock implementation is applied only within the scoped effect, and the original Clock service is automatically restored when the scope ends, ensuring predictable behavior.

```typescript
import { Effect, Clock, Console } from "effect"

// Assume customClock is an implementation of the Clock service
// const customClock = { currentTimeMillis: () => Effect.succeed(1678886400000) };

const program = Effect.gen(function* () {
  const now = yield* Clock.currentTimeMillis
  yield* Console.log(`Application started at ${new Date(now)}`)
})

// Effect.withClockScoped(customClock)(program)
```

---

### Demonstrate Nested Scope Closure with Effect.forkIn

Source: https://effect.website/docs/concurrency/fibers

Illustrates the behavior of nested scopes and child fibers when using Effect.forkIn. It shows how a child fiber continues to run even after its inner scope is closed, until the outer scope is also closed. Dependencies include Effect.forkIn, Effect.sleep, and Effect.runFork.

```typescript
import { Effect, Fiber, Scope, Console } from "effect"

const program = Effect.gen(function* () {
  const outerScope = yield* Scope.make
  const child = yield* Effect.gen(function* () {
    yield* Effect.addFinalizer(() =>
      Console.log("The inner scope is about to be closed!")
    )
    yield* Effect.sleep("100 millis")
    Console.log("child: still running!")
  })

  yield* outerScope.addFinalizer(yield* Effect.gen(function* () {
    yield* Effect.sleep("100 millis")
    Console.log("The outer scope is about to be closed!")
  }))

  // Fork the child fiber in the outer scope
  yield* Effect.forkIn(child, outerScope)
  yield* Effect.sleep("3 seconds")
})

Effect.runFork(program)
```
