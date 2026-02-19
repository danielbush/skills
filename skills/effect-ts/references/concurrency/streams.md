# Effect – Streams

`Stream` creation, transformation, merging, broadcasting, sinks, and stream resource management.

---

### Transforming Stream Elements with Stream.map (TypeScript)

Source: https://effect.website/docs/stream/operations

Demonstrates the basic usage of Stream.map to apply a function to each element of a stream, producing a new stream with the transformed values. The provided example increments each element by one.

```typescript
import { Stream, Effect } from "effect"

const stream = Stream.make(1, 2, 3).pipe(
  Stream.map((n) => n + 1) // Increment each element by 1
)

Effect.runPromise(Stream.runCollect(stream)).then(console.log)
/*
Output:
{ _id: 'Chunk', values: [ 2, 3, 4 ] }
*/
```

---

### Buffering Effect Streams for Speed Mismatch Handling

Source: https://effect.website/docs/stream/operations

This example shows how to use `Stream.buffer` in Effect JS to manage speed differences between a producer and consumer. It buffers elements up to a capacity of 4, logs elements before and after buffering, and introduces a 5-second delay between emissions to illustrate the buffering effect.

```typescript
const stream = Stream.range(1, 10).pipe(
  // Log each element before buffering
  Stream.tap((n) => Console.log(`before buffering: ${n}`)),
  // Buffer with a capacity of 4 elements
  Stream.buffer({ capacity: 4 }),
  // Log each element after buffering
  Stream.tap((n) => Console.log(`after buffering: ${n}`)),
  // Add a 5-second delay between each emission
  Stream.schedule(Schedule.spaced("5 seconds"))
);

Effect.runPromise(Stream.runCollect(stream)).then(console.log);
```

---

### Index Stream Elements with zipWithIndex

Source: https://effect.website/docs/stream/operations

The `Stream.zipWithIndex` operator attaches an index to each element of a stream. It pairs each element with its sequential position, starting from 0. This operator is helpful for maintaining order and referencing elements by their position.

```typescript
import { Stream, Effect } from "effect"

const stream = Stream.zipWithIndex(
  Stream.make("Mary", "James", "Robert", "Patricia")
)

Effect.runPromise(Stream.runCollect(stream)).then(console.log)
/* 
Output:
{
  _id: 'Chunk',
  values: [
    [ 'Mary', 0 ],
    [ 'James', 1 ],
    [ 'Robert', 2 ],
    [ 'Patricia', 3 ]
  ]
}
*/
```

---

### Add Affixes with Stream.intersperseAffixes

Source: https://effect.website/docs/stream/operations

Stream.intersperseAffixes provides more control by allowing different elements to be added at the start, between elements, and at the end of the stream. This is valuable for creating structured output like arrays or delimited strings with custom formatting.

```typescript
import { Stream, Effect } from "effect"

// Create a stream and add affixes:
// - `[` at the start
// - `|` between elements
// - `]` at the end
const stream = Stream.make(1, 2, 3, 4, 5).pipe(
  Stream.intersperseAffixes({
    start: "[",
    middle: "|",
    end: "]"
  })
)

Effect.runPromise(Stream.runCollect(stream)).then(console.log)
/* 
Output: 
{ 
_id: 'Chunk', 
values: [ 
'[', 1, '|', 2, '|', 
3, '|', 4, '|', 5, 
']' 
] 
} 
*/
```

---

### Debounce Stream Values in Effect-TS

Source: https://effect.website/docs/stream/operations

This example shows how to use `Stream.debounce` to only emit values after a certain quiet period. It's useful for preventing excessive processing when data arrives rapidly. It requires the 'effect' library.

```typescript
import { Stream, Effect } from "effect"

// Helper function to log with elapsed time since the last log
let last = Date.now()
const log = (message: string) =>
  Effect.sync(() => {
    const end = Date.now()
    console.log(`${message} after ${end - last}ms`)
    last = end
  })

const stream = Stream.make(1, 2, 3).pipe(
  // Emit the value 4 after 200 ms
  Stream.concat(
    Stream.fromEffect(Effect.sleep("200 millis").pipe(Effect.as(4)))
  ),
  // Continue with more rapid values
  Stream.concat(Stream.make(5, 6)),
  // Emit 7 after 150 ms
  Stream.concat(
    Stream.fromEffect(Effect.sleep("150 millis").pipe(Effect.as(7)))
  ),
  Stream.concat(Stream.make(8)),
  Stream.tap((n) => log(`Received ${n}`)),
  // Only emit values after a pause of at least 100 milliseconds
  Stream.debounce("100 millis"),
  Stream.tap((n) => log(`> Emitted ${n}`))
)

Effect.runPromise(Stream.runCollect(stream)).then(console.log)
```

---

### Partition Stream with Effectful Predicate using Stream.partitionEither

Source: https://effect.website/docs/stream/operations

This example demonstrates how to use Stream.partitionEither to split a stream into two substreams based on an effectful predicate. It separates elements into those producing Either.left and Either.right values. The input is a stream of integers, and the output is two streams, one for odd numbers and one for even numbers.

```typescript
import { Stream, Effect, Either } from "effect"

// ┌─── Effect<[Stream, Stream], never, Scope>
// ▼
const program = Stream.range(1, 9).pipe(
  Stream.partitionEither(
    // Simulate an effectful computation
    (n) => Effect.succeed(n % 2 === 0 ? Either.right(n) : Either.left(n)),
    { bufferSize: 5 }
  )
)

Effect.runPromise(
  Effect.scoped(
    Effect.gen(function*() {
      const [odds, evens] = yield* program
      console.log(yield* Stream.runCollect(odds))
      console.log(yield* Stream.runCollect(evens))
    })
  )
)

/* 
Output:
{
  _id: 'Chunk', values: [ 1, 3, 5, 7, 9 ]
}
{
  _id: 'Chunk', values: [ 2, 4, 6, 8 ]
}
*/
```

---

### Using a Sink to Collect Stream Elements

Source: https://effect.website/docs/sink/introduction

This example demonstrates how to use a Sink to consume elements from a Stream. It creates a stream of numbers and a Sink that takes the first two elements. The Stream.run function is then used to process the stream with the sink, and the result is logged to the console. This showcases a basic application of Sinks for data collection.

```typescript
import { Stream, Sink, Effect } from "effect"

// ┌─── Stream
// ▼
const stream = Stream.make(1, 2, 3)

// Create a sink to take the first 2 elements of the stream
//
// ┌─── Sink, number, number, never, never>
// ▼
const sink = Sink.take(2)

// Run the stream through the sink to collect the elements
//
// ┌─── Effect
// ▼
const sum = Stream.run(stream, sink)

Effect.runPromise(sum).then(console.log)
/*
Output:
{
  _id: 'Chunk',
  values: [ 1, 2 ]
}
*/
```

---

### Simulate Paginated API Fetch with Effect

Source: https://effect.website/docs/stream/creating

Demonstrates fetching data from a simulated paginated API using Effect. Defines a type for paginated results and a function to fetch pages. Useful for understanding pagination patterns. Requires 'effect' library.

```typescript
import { Chunk, Effect } from "effect"

type RawData = string 

class PageResult {
  constructor(
    readonly results: Chunk.Chunk,
    readonly isLast: boolean
  ) {}
}

const pageSize = 2

const listPaginated = (
  pageNumber: number 
): Effect.Effect => {
  return Effect.succeed(
    new PageResult(
      Chunk.map(
        Chunk.range(1, pageSize),
        (index) => `Result ${pageNumber}-`
        }
      ]
    }
  ]
}
```
```

---

### Create Stream with Stream.iterate

Source: https://effect.website/docs/stream/creating

Generates a stream by iteratively applying a function to an initial value. The stream starts with the initial value, followed by subsequent values produced by repeatedly applying the function.

```typescript
import { Stream, Effect } from "effect"

// Creating a stream of incrementing numbers
const stream = Stream.iterate(1, (n) => n + 1) // Produces 1, 2, 3, ...

Effect.runPromise(Stream.runCollect(stream.pipe(Stream.take(5)))).then(
  console.log
)
// { _id: 'Chunk', values: [ 1, 2, 3, 4, 5 ] }
```

---

### Group Stream Elements by Key using Stream.groupByKey

Source: https://effect.website/docs/stream/operations

This example illustrates how to group elements of a stream by a key derived from each element using Stream.groupByKey. The function is non-effectful and uses a provided key function. The result is a GroupBy data type that can be evaluated. The input is a stream of Exam objects, and the output groups them by the tens place of their scores and counts the number of exams in each group.

```typescript
import { Stream, GroupBy, Effect, Chunk } from "effect"

class Exam {
  constructor(
    readonly person: string,
    readonly score: number
  ) {}
}

// Define a list of exam results
const examResults = [
  new Exam("Alex", 64),
  new Exam("Michael", 97),
  new Exam("Bill", 77),
  new Exam("John", 78),
  new Exam("Bobby", 71)
]

// Group exam results by the tens place in the score
const groupByKeyResult = Stream.fromIterable(examResults).pipe(
  Stream.groupByKey((exam) => Math.floor(exam.score / 10) * 10)
)

// Count the number of exam results in each group
const stream = GroupBy.evaluate(groupByKeyResult, (key, stream) =>
  Stream.fromEffect(
    Stream.runCollect(stream).pipe(
      Effect.andThen((chunk) => [key, Chunk.size(chunk)] as const)
    )
  )
)

Effect.runPromise(Stream.runCollect(stream)).then(console.log)

/* 
Output:
{
  _id: 'Chunk', values: [ [ 60, 1 ], [ 90, 1 ], [ 70, 3 ] ]
}
*/
```

---

### Timeout Stream and Fail with Message

Source: https://effect.website/docs/stream/error-handling

This example uses `Stream.timeoutFail` to terminate a stream after a set duration and fail with a custom error message. The stream `Effect.never` will time out, resulting in a failure with the provided message.

```typescript
import { Stream, Effect } from "effect"

const stream = Stream.fromEffect(Effect.never).pipe(
  Stream.timeoutFail(() => "timeout", "2 seconds")
)

Effect.runPromiseExit(Stream.runCollect(stream)).then(console.log)
```

---

### Collecting Leftovers with Sink.collectLeftover

Source: https://effect.website/docs/sink/leftovers

This example demonstrates how to use `Sink.collectLeftover` to capture elements that remain unconsumed after a sink operation. It returns a tuple containing the sink's result and the leftover elements. This is useful when you need to process or inspect all elements from a stream, even if the sink only acts on a subset.

```typescript
import { Stream, Sink, Effect } from "effect"

const stream = Stream.make(1, 2, 3, 4, 5)

// Take the first 3 elements and collect any leftovers
const sink1 = Sink.take(3).pipe(Sink.collectLeftover)

Effect.runPromise(Stream.run(stream, sink1)).then(console.log)
/*
Output:
[
  { _id: 'Chunk', values: [ 1, 2, 3 ] },
  { _id: 'Chunk', values: [ 4, 5 ] }
]
*/

// Take only the first element and collect the rest as leftovers
const sink2 = Sink.head().pipe(Sink.collectLeftover)

Effect.runPromise(Stream.run(stream, sink2)).then(console.log)
/*
Output:
[
  { _id: 'Option', _tag: 'Some', value: 1 },
  { _id: 'Chunk', values: [ 2, 3, 4, 5 ] }
]
*/
```

---

### Generate Natural Numbers with Stream.unfold

Source: https://effect.website/docs/stream/creating

Generates a stream of natural numbers starting from an initial state. Uses a step function that returns Option.some([value, nextState]) to continue or Option.none() to end. Requires 'effect' library. The output is a chunk of numbers.

```typescript
import { Stream, Effect, Option } from "effect"

const stream = Stream.unfold(1, (n) => Option.some([n, n + 1]))

Effect.runPromise(Stream.runCollect(stream.pipe(Stream.take(5)))).then(
  console.log
)
// { _id: 'Chunk', values: [ 1, 2, 3, 4, 5 ] }
```

---

### Group Stream Elements with Effectful Function using Stream.groupBy

Source: https://effect.website/docs/stream/operations

This example shows how to group stream elements using an effectful partitioning function with Stream.groupBy. It returns a GroupBy data type that can be evaluated. The input is a stream of names, and the output groups them by their first letter and counts the names in each group, simulating an effectful operation.

```typescript
import { Stream, GroupBy, Effect, Chunk } from "effect"

// Group names by their first letter
const groupByKeyResult = Stream.fromIterable([
  "Mary",
  "James",
  "Robert",
  "Patricia",
  "John",
  "Jennifer",
  "Rebecca",
  "Peter"
]).pipe(
  // Simulate an effectful groupBy operation
  Stream.groupBy((name) => Effect.succeed([name.substring(0, 1), name]))
)

// Count the number of names in each group and display results
const stream = GroupBy.evaluate(groupByKeyResult, (key, stream) =>
  Stream.fromEffect(
    Stream.runCollect(stream).pipe(
      Effect.andThen((chunk) => [key, Chunk.size(chunk)] as const)
    )
  )
)

Effect.runPromise(Stream.runCollect(stream)).then(console.log)

/* 
Output:
{
  _id: 'Chunk',
  values: [ [ 'M', 1 ], [ 'J', 3 ], [ 'R', 2 ], [ 'P', 2 ] ]
}
*/
```

---

### Repeat Single Value Stream using Effect TS

Source: https://effect.website/docs/stream/creating

Creates a stream that endlessly repeats a specified value. This is useful for generating streams with a constant value. The example demonstrates taking the first 5 elements of such a stream.

```typescript
import { Stream, Effect } from "effect"

const stream = Stream.repeatValue(0)

Effect.runPromise(Stream.runCollect(stream.pipe(Stream.take(5)))).then(
  console.log
)
// { _id: 'Chunk', values: [ 0, 0, 0, 0, 0 ] }
```

---

### Refine or Die Stream Errors by Type

Source: https://effect.website/docs/stream/error-handling

This example shows how to selectively keep certain errors while terminating the stream with others using `Stream.refineOrDie`. It filters errors, retaining only `SyntaxError` types and discarding all other error types.

```typescript
import { Stream, Option } from "effect"

const stream = Stream.fail(new Error())

const res = Stream.refineOrDie(stream, (error) => {
  if (error instanceof SyntaxError) {
    return Option.some(error)
  }
  return Option.none()
})
```

---

### Stream Throttling with Burst Capacity (TypeScript)

Source: https://effect.website/docs/stream/operations

Demonstrates how to throttle a stream using 'effect' library, allowing temporary increases in data throughput beyond set rate limits by configuring burst capacity. This is useful for handling uneven data flows.

```typescript
import { Effect, Schedule, Stream, Chunk } from "effect"

// Helper function to log with elapsed time since last log
let last = Date.now()
const log = (message: string) =>
  Effect.sync(() => {
    const end = Date.now()
    console.log(`${message} after ${end - last}ms`)
    last = end
  })

const stream = Stream.fromSchedule(Schedule.spaced("10 millis")).pipe(
  Stream.take(20),
  Stream.tap((n) => log(`Received ${n}`)),
  Stream.throttle({
    cost: Chunk.size,
    duration: "200 millis",
    units: 5,
    strategy: "enforce",
    burst: 2
  }),
  Stream.tap((n) => log(`> Emitted ${n}`))
)

Effect.runPromise(Stream.runCollect(stream)).then(console.log)
```

---

### Timeout Stream and Switch to Another

Source: https://effect.website/docs/stream/error-handling

This example demonstrates `Stream.timeoutTo`, where if a stream doesn't produce a value within a specified time, it switches to a different stream. The `Effect.never` stream times out, and execution continues with `Stream.make(1, 2, 3)`.

```typescript
import { Stream, Effect } from "effect"

const stream = Stream.fromEffect(Effect.never).pipe(
  Stream.timeoutTo("2 seconds", Stream.make(1, 2, 3))
)

Effect.runPromise(Stream.runCollect(stream)).then(console.log)
```

---

### Zip Streams with Custom Pairing Logic (EffectJS)

Source: https://effect.website/docs/stream/operations

Demonstrates using `Stream.zipWith` to combine elements from two streams by applying a custom function to each pair. This allows for flexible transformations on the zipped elements.

```typescript
import { Stream, Effect } from "effect"

const stream = Stream.zipWith(
  Stream.make(1, 2, 3, 4, 5, 6),
  Stream.make("a", "b", "c"),
  (n, s) => [n + 10, s + "!"]
)

Effect.runPromise(Stream.runCollect(stream)).then(console.log)
```

---

### Effect-TS: Control Stream Termination with 'haltStrategy'

Source: https://effect.website/docs/stream/operations

Illustrates how to control the termination of a merged stream using the `haltStrategy` option in `Stream.merge`. The example specifically uses `haltStrategy: "left"` to terminate the merged stream as soon as the left-hand stream finishes, even if the right-hand stream is still active.

```typescript
import { Stream, Schedule, Effect } from "effect"

const s1 = Stream.range(1, 5).pipe(
  Stream.schedule(Schedule.spaced("100 millis"))
)
const s2 = Stream.repeatValue(0).pipe(
  Stream.schedule(Schedule.spaced("200 millis"))
)

const merged = Stream.merge(s1, s2, { haltStrategy: "left" })

Effect.runPromise(Stream.runCollect(merged)).then(console.log)
```

---

### Ignoring Leftovers with Sink.ignoreLeftover

Source: https://effect.website/docs/sink/leftovers

This example shows how to use `Sink.ignoreLeftover` to discard any elements that are not consumed by the sink. This is beneficial when you only care about the result of the sink's intended operation and want to ensure that any remaining elements do not interfere with subsequent processing or cause unexpected behavior. The `Sink.collectLeftover` is still used here to demonstrate that the leftovers are indeed empty.

```typescript
import { Stream, Sink, Effect } from "effect"

const stream = Stream.make(1, 2, 3, 4, 5)

// Take the first 3 elements and ignore any remaining elements
const sink = Sink.take(3).pipe(
  Sink.ignoreLeftover,
  Sink.collectLeftover
)

Effect.runPromise(Stream.run(stream, sink)).then(console.log)
/*
Output:
[
  { _id: 'Chunk', values: [ 1, 2, 3 ] },
  { _id: 'Chunk', values: [] }
]
*/
```

---

### Effect-TS: Interleaving Streams with 'Stream.merge'

Source: https://effect.website/docs/stream/operations

Shows how to combine elements from two streams into a single stream using `Stream.merge`. This operation interleaves elements as they become available, unlike `Stream.concat` which waits for the first stream to complete. The example demonstrates merging two streams with different emission intervals.

```typescript
import { Schedule, Stream, Effect } from "effect"

// Create two streams with different emission intervals
const s1 = Stream.make(1, 2, 3).pipe(
  Stream.schedule(Schedule.spaced("100 millis"))
)
const s2 = Stream.make(4, 5, 6).pipe(
  Stream.schedule(Schedule.spaced("200 millis"))
)

// Merge s1 and s2 into a single stream that interleaves their values
const merged = Stream.merge(s1, s2)

Effect.runPromise(Stream.runCollect(merged)).then(console.log)
```

---

### Repeat Stream Content using Effect TS

Source: https://effect.website/docs/stream/creating

Constructs a new stream that repeats the content of a given stream according to a specified schedule. This enables the creation of recurring events or values based on a defined pattern. The example uses `Schedule.forever` to repeat indefinitely.

```typescript
import { Stream, Effect, Schedule } from "effect"

// Creating a stream that repeats a value indefinitely
const stream = Stream.repeat(Stream.succeed(1), Schedule.forever)

Effect.runPromise(Stream.runCollect(stream.pipe(Stream.take(5)))).then(
  console.log
)
// { _id: 'Chunk', values: [ 1, 1, 1, 1, 1 ] }
```

---

### Stream.ensuring for Post-Finalization Actions (TypeScript)

Source: https://effect.website/docs/stream/resourceful-streams

Illustrates the use of Stream.ensuring to execute actions after a stream has been finalized. This operator is useful for performing secondary tasks that should run once the primary stream and its finalizers have completed. The example shows application logic, finalization, and then post-finalization work.

```typescript
import { Stream, Console, Effect } from "effect"

const program = Stream.fromEffect(Console.log("Application Logic.")).pipe(
  Stream.concat(Stream.finalizer(Console.log("Finalizing the stream"))),
  Stream.ensuring(
    Console.log("Doing some other works after stream's finalization")
  )
)

Effect.runPromise(Stream.runCollect(program)).then(console.log)
```

---

### Zip Streams with Custom Logic for Uneven Lengths (EffectJS)

Source: https://effect.website/docs/stream/operations

Demonstrates `Stream.zipAllWith` for zipping streams with custom logic that defines how to handle elements when one stream is shorter than the other. This provides fine-grained control over combining elements in all scenarios.

```typescript
import { Stream, Effect } from "effect"

const stream = Stream.zipAllWith(Stream.make(1, 2, 3, 4, 5, 6), {
  other: Stream.make("a", "b", "c"),
  onSelf: (n) => [n, "x"],
  onOther: (s) => [-1, s],
  onBoth: (n, s) => [n + 10, s + "!"]
})

Effect.runPromise(Stream.runCollect(stream)).then(console.log)
```

---

### Broadcast Stream to Multiple Consumers with Stream.broadcast

Source: https://effect.website/docs/stream/operations

The Stream.broadcast operator duplicates a source stream, sending all its elements to multiple downstream consumers. It includes a 'maximumLag' parameter to manage backpressure, ensuring the upstream stream doesn't get too far ahead of the slowest consumer. This is useful for distributing stream data to parallel processing units.

```typescript
import { Effect, Stream, Console, Schedule, Fiber } from "effect"

const numbers = Effect.scoped(
  Stream.range(1, 20).pipe(
    Stream.tap((n) =>
      Console.log(`Emit ${n} element before broadcasting`)
    ),
    // Broadcast to 2 downstream consumers with max lag of 5
    Stream.broadcast(2, 5)
  )
)
```

---

### Repeat Effect Result Stream using Effect TS

Source: https://effect.website/docs/stream/creating

Generates a stream by repeatedly evaluating an Effect. This is useful for creating streams of values produced by effectful operations, such as random number generation. The stream's results are collected into a Chunk. The example shows generating a stream of random integers.

```typescript
import { Stream, Effect, Random } from "effect"

const stream = Stream.repeatEffect(Random.nextInt)

Effect.runPromise(Stream.runCollect(stream.pipe(Stream.take(5)))).then(
  console.log
)
/* 
Example Output:
{
  _id: 'Chunk',
  values: [ 1666935266, 604851965, 2194299958, 3393707011, 4090317618 ]
}
*/
```

---

### Distinguish Stream Elements with Stream.orElseEither (JavaScript)

Source: https://effect.website/docs/stream/error-handling

This example showcases `Stream.orElseEither`, a variant of `Stream.orElse`, which uses the Either data type to clearly distinguish elements originating from the primary stream (`s1`) or the fallback stream (`s2`). This is useful when you need to differentiate the source of each element, especially after a potential failure. It requires the 'effect' library.

```javascript
import { Stream, Effect } from "effect"

const s1 = Stream.make(1, 2, 3).pipe(
  Stream.concat(Stream.fail("Oh! Error!")),
  Stream.concat(Stream.make(4, 5))
)

const s2 = Stream.make("a", "b", "c")

const stream = Stream.orElseEither(s1, () => s2)

Effect.runPromise(Stream.runCollect(stream)).then(console.log)
/* 
Output:
{
  _id: "Chunk",
  values: [
    {
      _id: "Either",
      _tag: "Left",
      left: 1
    },
    {
      _id: "Either",
      _tag: "Left",
      left: 2
    },
    {
      _id: "Either",
      _tag: "Left",
      left: 3
    },
    {
      _id: "Either",
      _tag: "Right",
      right: "a"
    },
    {
      _id: "Either",
      _tag: "Right",
      right: "b"
    },
    {
      _id: "Either",
      _tag: "Right",
      right: "c"
    }
  ]
}
*/
```

---

### Extracting Elements from Streams with take variations (TypeScript)

Source: https://effect.website/docs/stream/operations

Shows how to use various 'take' operations on streams: `take` for a fixed number of elements, `takeWhile` for elements meeting a condition, `takeUntil` for elements until a condition is met, and `takeRight` for elements from the end. These functions help in selectively extracting data from a stream.

```typescript
import { Stream, Effect } from "effect"

const stream = Stream.iterate(0, (n) => n + 1)

// Using `take` to extract a fixed number of elements:
const s1 = Stream.take(stream, 5)
Effect.runPromise(Stream.runCollect(s1)).then(console.log)
/*
Output:
{ _id: 'Chunk', values: [ 0, 1, 2, 3, 4 ] }
*/

// Using `takeWhile` to extract elements while a condition is met:
const s2 = Stream.takeWhile(stream, (n) => n < 5)
Effect.runPromise(Stream.runCollect(s2)).then(console.log)
/*
Output:
{ _id: 'Chunk', values: [ 0, 1, 2, 3, 4 ] }
*/

// Using `takeUntil` to extract elements until a condition is met:
const s3 = Stream.takeUntil(stream, (n) => n === 5)
Effect.runPromise(Stream.runCollect(s3)).then(console.log)
/*
Output:
{ _id: 'Chunk', values: [ 0, 1, 2, 3, 4, 5 ] }
*/

// Using `takeRight` to take elements from the end of the stream:
const s4 = Stream.takeRight(s3, 3)
Effect.runPromise(Stream.runCollect(s4)).then(console.log)
/*
Output:
{ _id: 'Chunk', values: [ 3, 4, 5 ] }
*/
```

---

### Sum Stream Elements Using Fold Left (Effect-TS)

Source: https://effect.website/docs/sink/creating

The Sink.foldLeft function reduces a stream to a single cumulative value by applying a given operation sequentially to each element, starting with an initial value. This is efficient for operations like summing or concatenating. Dependencies include Stream, Sink, and Effect from 'effect'. Input is a stream and an initial value with an accumulator function; output is the final reduced value.

```typescript
import { Stream, Sink, Effect } from "effect"

const stream = Stream.make(1, 2, 3, 4)

Effect.runPromise(
  Stream.run(
    stream,
    // Use foldLeft to sequentially add each element, starting with 0
    Sink.foldLeft(0, (a, b) => a + b)
  )
).then(console.log)
// Output: 10
```

---

### Stream Command stdout Directly to Process stdout

Source: https://effect.website/docs/platform/command

Explains how to pipe a command's standard output directly to the parent process's standard output using `Command.stdout("inherit")`. This is efficient for commands that produce large amounts of output.

```typescript
import { Command } from "@effect/platform"
import { NodeContext, NodeRuntime } from "@effect/platform-node"
import { Effect } from "effect"

// Create a command to run `cat` on a file and inherit stdout
const program = Command.make("cat", "./some-file.txt").pipe(
  Command.stdout("inherit"), // Stream stdout to process.stdout
  Command.exitCode // Get the exit code
)

NodeRuntime.runMain(program.pipe(Effect.provide(NodeContext.layer)))
```
