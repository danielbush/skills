# Effect – Queues & PubSub

`Queue`, `PubSub`, `SubscriptionRef`, `Semaphore`, and concurrent message passing.

---

### Create and Use Bounded PubSub - Effect.js

Source: https://effect.website/docs/concurrency/pubsub

Demonstrates creating a bounded PubSub and subscribing to it. Messages published to the PubSub are received by all subscribers. This example uses Effect.js's PubSub and Queue APIs.

```typescript
import { Effect, PubSub, Queue } from "effect"

const program = Effect.scoped(
  Effect.gen(function* () {
    const pubsub = yield* PubSub.bounded(2)

    // Two subscribers
    const dequeue1 = yield* PubSub.subscribe(pubsub)
    const dequeue2 = yield* PubSub.subscribe(pubsub)

    // Publish a message to the pubsub
    yield* PubSub.publish(pubsub, "Hello from a PubSub!")

    // Each subscriber receives the message
    console.log("Subscriber 1: " + (yield* Queue.take(dequeue1)))
    console.log("Subscriber 2: " + (yield* Queue.take(dequeue2)))
  })
)

Effect.runFork(program)
/* 
Output:
Subscriber 1: Hello from a PubSub!
Subscriber 2: Hello from a PubSub!
*/
```

---

### Using Enqueue and Dequeue Queues Together in Effect-TS

Source: https://effect.website/docs/concurrency/queue

Demonstrates how to combine `Enqueue` and `Dequeue` interfaces for a queue within Effect-TS. This example shows a program that adds values to a queue using an `Enqueue` interface and retrieves them using a `Dequeue` interface, illustrating the flexibility of Effect-TS Queues.

```typescript
import { Effect, Queue } from "effect"

const send = (offerOnlyQueue: Queue.Enqueue, value: number) => {
  return Queue.offer(offerOnlyQueue, value)
}

const receive = (takeOnlyQueue: Queue.Dequeue) => {
  return Queue.take(takeOnlyQueue)
}

const program = Effect.gen(function* () {
  const queue = yield* Queue.unbounded()

  // Add values to the queue
  yield* send(queue, 1)
  yield* send(queue, 2)

  // Retrieve values from the queue
  console.log(yield* receive(queue))
  console.log(yield* receive(queue))
})

Effect.runFork(program)
/* 
Output:
1
2
*/
```

---

### Server and Client Interaction with SubscriptionRef in Effect.js

Source: https://effect.website/docs/state-management/subscriptionref

This snippet defines the core components for a real-time update system. The `server` function continuously increments a shared `Ref`, while the `client` function processes chunks of these updates from a `Stream`. The `program` orchestrates the setup by creating a `SubscriptionRef`, forking the server, launching multiple clients concurrently, and then collecting and displaying their results.

```typescript
import { Ref, Effect, Stream, Random, SubscriptionRef, Fiber } from "effect"

// Server function that increments a shared value forever
const server = (ref: Ref.Ref) =>
  Ref.update(ref, (n) => n + 1).pipe(Effect.forever)

// Client function that observes the stream of changes
const client = (changes: Stream.Stream) =>
  Effect.gen(function* () {
    const n = yield* Random.nextIntBetween(1, 10)
    const chunk = yield* Stream.runCollect(Stream.take(changes, n))
    return chunk
  })

const program = Effect.gen(function* () {
  // Create a SubscriptionRef with an initial value of 0
  const ref = yield* SubscriptionRef.make(0)

  // Fork the server to run concurrently
  const serverFiber = yield* Effect.fork(server(ref))

  // Create 5 clients that subscribe to the changes stream
  const clients = new Array(5).fill(null).map(() => client(ref.changes))

  // Run all clients in concurrently and collect their results
  const chunks = yield* Effect.all(clients, { concurrency: "unbounded" })

  // Interrupt the server when clients are done
  yield* Fiber.interrupt(serverFiber)

  // Output the results collected by each client
  for (const chunk of chunks) {
    console.log(chunk)
  }
})

Effect.runPromise(program)
```

---

### Create SubscriptionRef - Effect

Source: https://effect.website/docs/state-management/subscriptionref

Demonstrates how to create a new SubscriptionRef with an initial value using the SubscriptionRef.make constructor. This is the fundamental step to begin observing state changes.

```typescript
import { SubscriptionRef } from "effect"

const ref = SubscriptionRef.make(0)
```

---

### Get PubSub Capacity and Size

Source: https://effect.website/docs/concurrency/pubsub

Retrieves the fixed capacity of a PubSub using PubSub.capacity and its current dynamic size using PubSub.size. PubSub.capacity returns a number, while PubSub.size returns an Effect, reflecting that the size can change over time.

```typescript
import { Effect, PubSub } from "effect"

const program = Effect.gen(function* () {
  const pubsub = yield* PubSub.bounded(2)
  console.log(`capacity: ${PubSub.capacity(pubsub)}`)
  console.log(`size: ${yield* PubSub.size(pubsub)}`)
})

Effect.runFork(program)
/* 
Output:
capacity: 2
size: 0
*/
```

---

### Create and Use a Bounded Queue in Effect.js

Source: https://effect.website/docs/concurrency/queue

Demonstrates how to create a bounded queue with a specific capacity and perform basic operations like offering and taking elements. This showcases the back-pressure mechanism where offers suspend if the queue is full.

```typescript
import { Effect, Queue } from "effect"

const program = Effect.gen(function* () {
  // Creates a bounded queue with capacity 100
  const queue = yield* Queue.bounded(100)
  // Adds 1 to the queue
  yield* Queue.offer(queue, 1)
  // Retrieves and removes the oldest value
  const value = yield* Queue.take(queue)
  return value
})

Effect.runPromise(program).then(console.log)
// Output: 1
```

---

### Sequential Task Execution with One-Permit Semaphore - Effect.js

Source: https://effect.website/docs/concurrency/semaphore

Demonstrates how to use `withPermits(1)` to force sequential execution of concurrent tasks. The one-permit semaphore ensures that only one task can proceed at a time, even when multiple tasks are started concurrently.

```typescript
import { Effect } from "effect"

const task = Effect.gen(function* () {
  yield* Effect.log("start")
  yield* Effect.sleep("2 seconds")
  yield* Effect.log("end")
})

const program = Effect.gen(function* () {
  const mutex = yield* Effect.makeSemaphore(1)

  // Wrap the task to require one permit, forcing sequential execution
  const semTask = mutex
    .withPermits(1)(task)
    .pipe(Effect.withLogSpan("elapsed"))

  // Run 3 tasks concurrently, but they execute sequentially
  // due to the one-permit semaphore
  yield* Effect.all([semTask, semTask, semTask], {
    concurrency: "unbounded"
  })
})

Effect.runFork(program)
```

---

### Server-Client Model with SubscriptionRef - Effect

Source: https://effect.website/docs/state-management/subscriptionref

Illustrates a server-client pattern using SubscriptionRef. The server continuously updates a shared Ref, while clients subscribe to the 'changes' stream of a SubscriptionRef to react to these updates. This showcases real-time state synchronization.

```typescript
import { Ref, Effect } from "effect"

// Server function that increments a shared value forever
const server = (ref: Ref.Ref) => 
  Ref.update(ref, (n) => n + 1).pipe(Effect.forever)
```

```typescript
import { Ref, Effect, Stream, Random } from "effect"

// Server function that increments a shared value forever
const server = (ref: Ref.Ref) => 
  Ref.update(ref, (n) => n + 1).pipe(Effect.forever)

// Client function that observes changes
```
