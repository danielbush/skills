# Effect – Scheduling

`Schedule`, retry, repeat, cron, jitter, and schedule combinators.

---

### Spaced Schedule: Recur with Delay Between Executions

Source: https://effect.website/docs/scheduling/built-in-schedules

Demonstrates the `Schedule.spaced` schedule, which repeats indefinitely with a specified duration between the end of one run and the start of the next. The example shows delays when the action takes 100ms and the schedule is spaced by 200ms.

```typescript
import { Array, Chunk, Duration, Effect, Schedule } from "effect"

const log = (
  schedule: Schedule.Schedule,
  delay: Duration.DurationInput = 0
): void => {
  const maxRecurs = 10
  const delays = Chunk.toArray(
    Effect.runSync(
      Schedule.run(
        Schedule.delays(Schedule.addDelay(schedule, () => delay)),
        Date.now(),
        Array.range(0, maxRecurs)
      )
    )
  )
  delays.forEach((duration, i) => {
    console.log(
      i === maxRecurs
        ? "..."
        : i === delays.length - 1
        ? "(end)"
        : `#${i + 1}: ${Duration.toMillis(duration)}ms`
    )
  })
}

const schedule = Schedule.spaced("200 millis")

// ┌─── Simulating an effect that takes
// │ 100 milliseconds to complete
// ▼
log(schedule, "100 millis")
```

---

### EffectJS scheduleExponential Example

Source: https://effect.website/docs/micro/new-users

Illustrates a schedule that recurs using exponential backoff, where each delay increases exponentially. The dryRun helper function visualizes this behavior.

```typescript
import { Micro } from "effect"
import * as Option from "effect/Option"

const dryRun = (
  schedule: Micro.MicroSchedule,
  maxAttempt: number = 7
): Array => {
  let attempt = 1
  let elapsed = 0
  const out: Array = []
  let duration = schedule(attempt, elapsed)
  while (Option.isSome(duration) && attempt <= maxAttempt) {
    const value = duration.value
    attempt++
    elapsed += value
    out.push(value)
    duration = schedule(attempt, elapsed)
  }
  return out
}

const policy = Micro.scheduleExponential(10)

console.log(dryRun(policy))
```

---

### Skip First Execution with Effect.schedule

Source: https://effect.website/docs/scheduling/repetition

This example demonstrates how to use Effect.schedule to avoid the initial execution of an effect, allowing it to run according to a defined schedule. It imports Effect, Schedule, and Console from 'effect'.

```typescript
import { Effect, Schedule, Console } from "effect"

const action = Console.log("success")

const policy = Schedule.addDelay(Schedule.recurs(2), () => "100 millis")

const program = Effect.schedule(action, policy)

Effect.runPromise(program).then((n) => console.log(`repetitions: ${n}`))
```

---

### EffectJS scheduleSpaced Example

Source: https://effect.website/docs/micro/new-users

Demonstrates a schedule that repeats indefinitely, with each repetition spaced a specified duration from the last run. It uses a dryRun helper function to visualize the schedule's behavior up to a maximum number of attempts.

```typescript
import { Micro } from "effect"
import * as Option from "effect/Option"

const dryRun = (
  schedule: Micro.MicroSchedule,
  maxAttempt: number = 7
): Array => {
  let attempt = 1
  let elapsed = 0
  const out: Array = []
  let duration = schedule(attempt, elapsed)
  while (Option.isSome(duration) && attempt <= maxAttempt) {
    const value = duration.value
    attempt++
    elapsed += value
    out.push(value)
    duration = schedule(attempt, elapsed)
  }
  return out
}

const policy = Micro.scheduleSpaced(10)

console.log(dryRun(policy))
```

---

### EffectJS scheduleIntersect Example

Source: https://effect.website/docs/micro/new-users

Shows how to combine two schedules using intersection. The resulting schedule recurs only if both input schedules want to continue, using the maximum delay between them. The dryRun helper function visualizes the intersection.

```typescript
import { Micro } from "effect"
import * as Option from "effect/Option"

const dryRun = (
  schedule: Micro.MicroSchedule,
  maxAttempt: number = 7
): Array => {
  let attempt = 1
  let elapsed = 0
  const out: Array = []
  let duration = schedule(attempt, elapsed)
  while (Option.isSome(duration) && attempt <= maxAttempt) {
    const value = duration.value
    attempt++
    elapsed += value
    out.push(value)
    duration = schedule(attempt, elapsed)
  }
  return out
}

const policy = Micro.scheduleIntersect(
  Micro.scheduleExponential(10),
  Micro.scheduleSpaced(300)
)

console.log(dryRun(policy))
```

---

### EffectJS scheduleUnion Example

Source: https://effect.website/docs/micro/new-users

Demonstrates combining two schedules using a union. The resulting schedule recurs as long as either of the input schedules wants to, using the minimum delay between recurrences. The dryRun helper visualizes the combined schedule.

```typescript
import { Micro } from "effect"
import * as Option from "effect/Option"

const dryRun = (
  schedule: Micro.MicroSchedule,
  maxAttempt: number = 7
): Array => {
  let attempt = 1
  let elapsed = 0
  const out: Array = []
  let duration = schedule(attempt, elapsed)
  while (Option.isSome(duration) && attempt <= maxAttempt) {
    const value = duration.value
    attempt++
    elapsed += value
    out.push(value)
    duration = schedule(attempt, elapsed)
  }
  return out
}

const policy = Micro.scheduleUnion(
  Micro.scheduleExponential(10),
  Micro.scheduleSpaced(300)
)

console.log(dryRun(policy))
```

---

### Retry Failing Effects with Fixed Delay in Effect.js

Source: https://effect.website/docs/micro/new-users

Shows how to use Micro.retry with Micro.scheduleSpaced to automatically retry a failing Effect.js operation after a fixed delay. The example retries an async task until it succeeds.

```typescript
import { Micro } from "effect"

let count = 0

// Simulates an effect with possible failures
const effect = Micro.async((resume) => {
  if (count <= 2) {
    count++
    console.log("failure")
    resume(Micro.fail(new Error()))
  } else {
    console.log("success")
    resume(Micro.succeed("yay!"))
  }
})

// Define a repetition policy using a spaced delay between retries
const policy = Micro.scheduleSpaced(100)

const repeated = Micro.retry(effect, { schedule: policy })

Micro.runPromise(repeated).then(console.log)
/* 
Output:
failure
failure
failure
success
yay!
*/
```

---

### Find Next Cron Run Time

Source: https://effect.website/docs/scheduling/cron

Determines the next date and time that satisfies a given cron schedule using the `next` function. It starts searching from a specified date or the current time if no starting date is provided. The function throws an error if it cannot find a matching date within a reasonable number of iterations to prevent infinite loops.

```typescript
import { Cron } from "effect"

// Define a cron expression for 4:00 AM
// on the 8th to the 14th of every month
const cron = Cron.unsafeParse("0 0 4 8-14 * *", "UTC")

// Specify the starting point for the search
const after = new Date("2025-01-08")

// Find the next matching date
const nextDate = Cron.next(cron, after)

console.log(nextDate)
// Output: 2025-01-08T04:00:00.000Z
```

---

### Create Cron Instance with Specific Constraints

Source: https://effect.website/docs/scheduling/cron

This example demonstrates how to create a Cron instance with specific constraints for seconds, minutes, hours, days, and optionally a time zone. The `make` function requires all fields to be defined, with empty arrays indicating no restrictions for a given field. This allows for precise scheduling, such as triggering at a specific time on a range of days within a month.

```typescript
import { Cron, DateTime } from "effect"

// Build a cron that triggers at 4:00 AM
// on the 8th to the 14th of each month
const cron = Cron.make({
  seconds: [0],
  minutes: [0],
  hours: [4],
  days: [8, 9, 10, 11, 12, 13, 14],
  months: [],
  weekdays: [],
  tz: DateTime.zoneUnsafeMakeNamed("Europe/Rome")
})
```

---

### Sequencing Schedule Combinator Example - Effect.js

Source: https://effect.website/docs/scheduling/schedule-combinators

Shows the 'Sequencing' schedule combinator. This combinator chains two schedules together. The first schedule runs to completion, and only then does the second schedule begin its execution.

```typescript
import { Array, Chunk, Duration, Effect, Schedule } from "effect"

// Assuming the log helper function is defined as above

const firstSchedule = Schedule.recurs(3).pipe(
  Schedule.addDelay(() => "100ms")
);

const secondSchedule = Schedule.spaced("50ms").pipe(
  Schedule.addDelay(() => "20ms")
);

const sequencedSchedule = Schedule.andThen(firstSchedule, secondSchedule);

console.log("--- Sequencing Schedule ---");
log(sequencedSchedule);

```

---

### Repeat Action Multiple Times with Effect.repeatN

Source: https://effect.website/docs/scheduling/repetition

Demonstrates using Effect.repeatN to execute an effect a specified number of times in addition to its initial run. It imports Effect and Console from 'effect'. The example repeats a logging action twice.

```typescript
import { Effect, Console } from "effect"

const action = Console.log("success")

// Repeat the action 2 additional times after the first execution
const program = Effect.repeatN(action, 2)

Effect.runPromise(program)
```

---

### Recurs Schedule: Execute a Task a Fixed Number of Times

Source: https://effect.website/docs/scheduling/built-in-schedules

Illustrates the `Schedule.recurs` schedule, which repeats a task a specified number of times. The example logs the delays for each of the 5 recurrences.

```typescript
import { Array, Chunk, Duration, Effect, Schedule } from "effect"

const log = (
  schedule: Schedule.Schedule,
  delay: Duration.DurationInput = 0
): void => {
  const maxRecurs = 10
  const delays = Chunk.toArray(
    Effect.runSync(
      Schedule.run(
        Schedule.delays(Schedule.addDelay(schedule, () => delay)),
        Date.now(),
        Array.range(0, maxRecurs)
      )
    )
  )
  delays.forEach((duration, i) => {
    console.log(
      i === maxRecurs
        ? "..."
        : i === delays.length - 1
        ? "(end)"
        : `#${i + 1}: ${Duration.toMillis(duration)}ms`
    )
  })
}

const schedule = Schedule.recurs(5)

log(schedule)
```

---

### Combine Schedules with Sequencing (Effect)

Source: https://effect.website/docs/scheduling/schedule-combinators

Demonstrates using Schedule.andThen to combine two schedules sequentially. The first schedule runs fully before the second one takes over. This example switches from fixed retries to periodic execution.

```typescript
import { Array, Chunk, Duration, Effect, Schedule } from "effect"

const log = (
  schedule: Schedule.Schedule,
  delay: Duration.DurationInput = 0
): void => {
  const maxRecurs = 10
  const delays = Chunk.toArray(
    Effect.runSync(
      Schedule.run(
        Schedule.delays(Schedule.addDelay(schedule, () => delay)),
        Date.now(),
        Array.range(0, maxRecurs)
      )
    )
  )
  delays.forEach((duration, i) => {
    console.log(
      i === maxRecurs
        ? "..."
        : i === delays.length - 1
        ? "(end)"
        : `#${i + 1}: ${Duration.toMillis(duration)}ms`
    )
  })
}

const schedule = Schedule.andThen(
  Schedule.recurs(5),
  Schedule.spaced("1 second")
)

log(schedule)
```

---

### Combine Schedules with Union (Effect)

Source: https://effect.website/docs/scheduling/schedule-combinators

Demonstrates using Schedule.union to combine two schedules. The union operator selects the shortest delay at each step. This example combines an exponential backoff with a spaced interval.

```typescript
import { Array, Chunk, Duration, Effect, Schedule } from "effect"

const log = (
  schedule: Schedule.Schedule,
  delay: Duration.DurationInput = 0
): void => {
  const maxRecurs = 10
  const delays = Chunk.toArray(
    Effect.runSync(
      Schedule.run(
        Schedule.delays(Schedule.addDelay(schedule, () => delay)),
        Date.now(),
        Array.range(0, maxRecurs)
      )
    )
  )
  delays.forEach((duration, i) => {
    console.log(
      i === maxRecurs
        ? "..."
        : i === delays.length - 1
        ? "(end)"
        : `#${i + 1}: ${Duration.toMillis(duration)}ms`
    )
  })
}

const schedule = Schedule.union(
  Schedule.exponential("100 millis"),
  Schedule.spaced("1 second")
)

log(schedule)
```

---

### EffectJS: Indefinitely Recurring Schedule (forever)

Source: https://effect.website/docs/scheduling/built-in-schedules

Demonstrates the `Schedule.forever` schedule, which repeats indefinitely. The example uses the `log` helper function to show the execution delays, which will continue without end.

```typescript
import { Array, Chunk, Duration, Effect, Schedule } from "effect"

const log = (
  schedule: Schedule.Schedule,
  delay: Duration.DurationInput = 0
): void => {
  const maxRecurs = 10
  const delays = Chunk.toArray(
    Effect.runSync(
      Schedule.run(
        Schedule.delays(Schedule.addDelay(schedule, () => delay)),
        Date.now(),
        Array.range(0, maxRecurs)
      )
    )
  )
  delays.forEach((duration, i) => {
    console.log(
      i === maxRecurs
        ? "..."
        : i === delays.length - 1
        ? "(end)"
        : `#${i + 1}: ${Duration.toMillis(duration)}ms`
    )
  })
}

const schedule = Schedule.forever

log(schedule)
/* 
Output:
#1: 0ms < forever 
#2: 0ms 
#3: 0ms 
#4: 0ms 
#5: 0ms 
#6: 0ms 
*/
```

---

### Handle Failures During Repeats with Effect.repeatOrElse

Source: https://effect.website/docs/scheduling/repetition

Illustrates how to use Effect.repeatOrElse to repeat an effect based on a schedule and handle failures by executing a fallback handler. It imports Effect and Schedule from 'effect'. The example simulates an action that fails after a few successful attempts.

```typescript
import { Effect, Schedule } from "effect"

let count = 0

// Define an async effect that simulates an action with possible failures
const action = Effect.async((resume) => {
  if (count > 1) {
    console.log("failure")
    resume(Effect.fail("Uh oh!"))
  } else {
    count++
    console.log("success")
    resume(Effect.succeed("yay!"))
  }
})

// Define a schedule that repeats up to 2 times
// with a 100ms delay between attempts
const policy = Schedule.addDelay(Schedule.recurs(2), () => "100 millis")

// Provide a handler to run when failure occurs after the retries
const program = Effect.repeatOrElse(action, policy, () =>
  Effect.sync(() => {
    console.log("orElse")
    return count - 1
  })
)

Effect.runPromise(program).then((n) => console.log(`repetitions: ${n}`))
```

---

### Log Schedule Intervals with Effect.js

Source: https://effect.website/docs/scheduling/cron

This example demonstrates how to create a cron-like schedule using Effect.js and log the intervals at which it triggers. It utilizes `Cron.unsafeParse` to define the schedule and `Schedule.cron` to convert it into a usable schedule. The `log` function, which takes an action and a schedule, is used to output the schedule's execution times.

```typescript
const log = <R, E, A, B>(action: Effect<R, E, A>, schedule: Schedule<R, unknown, B>) => Effect.tapError(Effect.zip(action, schedule).pipe(Effect.fork, Effect.tap(() => Effect.sleep(1000))), () => Effect.unit);

// Build a cron that triggers at 4:00 AM
// on the 8th to the 14th of each month
const cron = Cron.unsafeParse("0 0 4 8-14 * *", "UTC")

// Convert the Cron into a Schedule
const schedule = Schedule.cron(cron)

// Define a dummy action to repeat
const action = Effect.void

// Log the schedule intervals
log(action, schedule)
```

---

### Union Schedule Combinator Example - Effect.js

Source: https://effect.website/docs/scheduling/schedule-combinators

Demonstrates the 'Union' schedule combinator, which combines two schedules. The resulting schedule recurs if either of the combined schedules wants to continue, and it uses the shorter delay between the two.

```typescript
import { Array, Chunk, Duration, Effect, Schedule } from "effect"

// Assuming the log helper function is defined as above

const exponential = Schedule.exponential("100ms").pipe(
  Schedule.addDelay(() => "50ms") // Adds a base delay
);

const spaced = Schedule.spaced("200ms").pipe(
  Schedule.addDelay(() => "20ms") // Adds a base delay
);

const unionSchedule = Schedule.union(exponential, spaced);

console.log("--- Union Schedule ---");
log(unionSchedule);

```

---

### Add Randomness to Delays with Jitter (Effect)

Source: https://effect.website/docs/scheduling/schedule-combinators

Demonstrates using Schedule.jittered to add randomness to retry delays. This helps avoid synchronized retries causing further overload. The example shows jittered exponential backoff.

```typescript
import { Array, Chunk, Duration, Effect, Schedule } from "effect"

const log = (
  schedule: Schedule.Schedule,
  delay: Duration.DurationInput = 0
): void => {
  const maxRecurs = 10
  const delays = Chunk.toArray(
    Effect.runSync(
      Schedule.run(
        Schedule.delays(Schedule.addDelay(schedule, () => delay)),
        Date.now(),
        Array.range(0, maxRecurs)
      )
    )
  )
  delays.forEach((duration, i) => {
    console.log(
      i === maxRecurs
        ? "..."
        : i === delays.length - 1
        ? "(end)"
        : `#${i + 1}: ${Duration.toMillis(duration)}ms`
    )
  })
}

const schedule = Schedule.jittered(Schedule.exponential("10 millis"))

log(schedule)
```

---

### Intersection Schedule Combinator Example - Effect.js

Source: https://effect.website/docs/scheduling/schedule-combinators

Illustrates the 'Intersection' schedule combinator. This combinator merges two schedules, and the resulting schedule recurs only if both input schedules want to continue. It uses the longer delay between the two schedules.

```typescript
import { Array, Chunk, Duration, Effect, Schedule } from "effect"

// Assuming the log helper function is defined as above

const exponential = Schedule.exponential("100ms").pipe(
  Schedule.addDelay(() => "50ms")
);

const spaced = Schedule.spaced("200ms").pipe(
  Schedule.addDelay(() => "20ms")
);

const intersectionSchedule = Schedule.intersect(exponential, spaced);

console.log("--- Intersection Schedule ---");
log(intersectionSchedule);

```

---

### Fixed Schedule: Recur at Fixed Intervals

Source: https://effect.website/docs/scheduling/built-in-schedules

Shows the `Schedule.fixed` schedule, which ensures recurrences happen at regular intervals, regardless of action execution time. This example logs delays when the action takes 100ms and the schedule is fixed at 200ms.

```typescript
import { Array, Chunk, Duration, Effect, Schedule } from "effect"

const log = (
  schedule: Schedule.Schedule,
  delay: Duration.DurationInput = 0
): void => {
  const maxRecurs = 10
  const delays = Chunk.toArray(
    Effect.runSync(
      Schedule.run(
        Schedule.delays(Schedule.addDelay(schedule, () => delay)),
        Date.now(),
        Array.range(0, maxRecurs)
      )
    )
  )
  delays.forEach((duration, i) => {
    console.log(
      i === maxRecurs
        ? "..."
        : i === delays.length - 1
        ? "(end)"
        : `#${i + 1}: ${Duration.toMillis(duration)}ms`
    )
  })
}

const schedule = Schedule.fixed("200 millis")

// ┌─── Simulating an effect that takes
// │ 100 milliseconds to complete
// ▼
log(schedule, "100 millis")
```

---

### Iterate Over Future Cron Dates

Source: https://effect.website/docs/scheduling/cron

Generates an infinite iterator of future dates that match a cron schedule using the `sequence` function. You can specify a starting date for the iteration. This is useful for generating a series of upcoming event times.

```typescript
import { Cron } from "effect"

// Define a cron expression for 4:00 AM
// on the 8th to the 14th of every month
const cron = Cron.unsafeParse("0 0 4 8-14 * *", "UTC")

// Specify the starting date
const start = new Date("2021-01-08")

// Create an iterator for the schedule
const iterator = Cron.sequence(cron, start)

// Get the first matching date after the start date
console.log(iterator.next().value)
// Output: 2021-01-08T04:00:00.000Z

// Get the second matching date after the start date
console.log(iterator.next().value)
// Output: 2021-01-09T04:00:00.000Z
```

---

### Combine Schedules with Intersection (Effect)

Source: https://effect.website/docs/scheduling/schedule-combinators

Demonstrates using Schedule.intersect to combine two schedules. The intersect operator recurs only if both schedules want to continue, using the longer delay. This example limits exponential backoff with a fixed number of retries.

```typescript
import { Array, Chunk, Duration, Effect, Schedule } from "effect"

const log = (
  schedule: Schedule.Schedule,
  delay: Duration.DurationInput = 0
): void => {
  const maxRecurs = 10
  const delays = Chunk.toArray(
    Effect.runSync(
      Schedule.run(
        Schedule.delays(Schedule.addDelay(schedule, () => delay)),
        Date.now(),
        Array.range(0, maxRecurs)
      )
    )
  )
  delays.forEach((duration, i) => {
    console.log(
      i === maxRecurs
        ? "..."
        : i === delays.length - 1
        ? "(end)"
        : `#${i + 1}: ${Duration.toMillis(duration)}ms`
    )
  })
}

const schedule = Schedule.intersect(
  Schedule.exponential("10 millis"),
  Schedule.recurs(5)
)

log(schedule)
```

---

### Retry Effect with Fixed Delay Schedule

Source: https://effect.website/docs/error-management/retrying

The `Effect.retry` function retries an effect based on a provided `Schedule` policy. This example demonstrates retrying an effect with a fixed delay between attempts, suitable for handling intermittent failures.

```typescript
import { Effect, Schedule } from "effect"

let count = 0

// Simulates an effect with possible failures
const task = Effect.async((resume) => {
  if (count <= 2) {
    count++
    console.log("failure")
    resume(Effect.fail(new Error()))
  } else {
    console.log("success")
    resume(Effect.succeed("yay!"))
  }
})

// Define a repetition policy using a fixed delay between retries
const policy = Schedule.delay(2000).lastIndexOf(3)

// Retry the task with the defined policy
const retriedTask = Effect.retry(task, policy)

Effect.runPromise(retriedTask).then(console.log).catch(console.error)
```

---

### Retry API Calls with Timeout in TypeScript

Source: https://effect.website/docs/scheduling/examples

This example demonstrates how to make an API call with a specified number of retries and a timeout duration. The `getJson` function fetches data from a URL, and the `program` function utilizes `Effect.retry` to attempt the call up to two times and `Effect.timeout` to interrupt the operation if it exceeds 4 seconds. `Effect.catchAll` is used to log any errors encountered. Dependencies include the 'effect' library.

```TypeScript
import { Console, Effect } from "effect"

// Function to make the API call
const getJson = (url: string) =>
  Effect.tryPromise(() =>
    fetch(url).then((res) => {
      if (!res.ok) {
        console.log("error")
        throw new Error(res.statusText)
      }
      console.log("ok")
      return res.json() as unknown
    })
  )

// Program that retries the API call twice, times out after 4 seconds,
// and logs errors
const program = (url: string) =>
  getJson(url).pipe(
    Effect.retry({ times: 2 }),
    Effect.timeout("4 seconds"),
    Effect.catchAll(Console.error)
  )

// Test case: successful API response
Effect.runFork(program("https://dummyjson.com/products/1?delay=1000"))
/* 
Output:
ok 
*/

// Test case: API call exceeding timeout limit
Effect.runFork(program("https://dummyjson.com/products/1?delay=5000"))
/* 
Output:
TimeoutException: Operation timed out before the specified duration of '4s' elapsed 
*/

// Test case: API returning an error response
Effect.runFork(program("https://dummyjson.com/auth/products/1?delay=500"))
/* 
Output:
error 
error 
error 
UnknownException: An unknown error occurred 
*/
```

---

### Dynamic Retry Delays with Retry-After Header using Effect.js

Source: https://effect.website/docs/scheduling/examples

This example demonstrates how to implement dynamic retry delays in Effect.js, particularly for handling '429 Too Many Requests' errors. It extracts the delay duration from a custom error's `retryAfter` property and uses it to configure the retry schedule, ensuring the application respects server-specified wait times.

```typescript
import { Duration, Effect, Schedule, Data } from "effect"

// Custom error class representing a "Too Many Requests" response
class TooManyRequestsError extends Data.TaggedError(
"TooManyRequestsError"
)<{ readonly retryAfter: number }> {}

let n = 1
const request = Effect.gen(function* () {
// Simulate failing a particular number of times
if (n < 3) {
const retryAfter = n * 500
console.log(`Attempt #${n++}, retry after ${retryAfter} millis...`)
// Simulate retrieving the retry-after header
return yield* Effect.fail(new TooManyRequestsError({ retryAfter }))
}
console.log("Done")
return "some result"
})

// Retry policy that extracts the retry delay from the error
const policy = Schedule.identity().pipe(
Schedule.addDelay((error) =>
error._tag === "TooManyRequestsError"
? // Wait for the specified retry-after duration
Duration.millis(error.retryAfter)
: Duration.zero
),
// Limit retries to 5 attempts
Schedule.intersect(Schedule.recurs(5))
)

const program = request.pipe(Effect.retry(policy))

Effect.runFork(program)
/* 
Output:
Attempt #1, retry after 500 millis...
Attempt #2, retry after 1000 millis...
Done 
*/
```

---

### Create Schedule from Cron Expression

Source: https://effect.website/docs/scheduling/cron

Creates an Effect Schedule from a cron expression or a Cron instance using `Schedule.cron`. This schedule triggers at the beginning of each interval defined by the cron. When triggered, it yields a tuple `[start, end]` representing the interval's timestamp range in milliseconds.

```typescript
import {
  Effect,
  Schedule,
  TestClock,
  Fiber,
  TestContext,
  Cron,
  Console
} from "effect"

// A helper function to log output at each interval of the schedule
const log = (
  action: Effect.Effect,
  schedule: Schedule.Schedule<[number, number], void>
): void => {
  let i = 0

  Effect.gen(function*() {
    const fiber: Fiber.RuntimeFiber<[[number, number], number]> = yield* Effect.gen(function*() {
      yield* action
      i++
    }).pipe(
      Effect.repeat(
        schedule.pipe(
          // Limit the number of iterations for the example
          Schedule.intersect(
            // ... rest of the code for the helper function ...
          )
        )
      )
    )
    yield* Console.log(fiber.result)
  }).pipe(Effect.provide(TestContext))
}

// Example usage would go here, calling the log helper with a schedule created from a cron expression.
```

---

### Run Scheduled Task Until Another Completes with Effect.js

Source: https://effect.website/docs/scheduling/examples

This example illustrates how to run a periodic task using Effect.js until another, potentially longer-running task, finishes. It utilizes `Effect.race` to combine a repeating action with a delayed effect, ensuring the repeating action stops once the main task is complete. This pattern is useful for polling or periodic logging that should cease upon a specific event.

```typescript
import { Effect, Console, Schedule } from "effect"

// Define a long-running effect 
// (e.g., a task that takes 5 seconds to complete)
const longRunningEffect = Console.log("done").pipe(
Effect.delay("5 seconds")
)

// Define an action to run periodically
const action = Console.log("action...")

// Define a fixed interval schedule
const schedule = Schedule.fixed("1.5 seconds")

// Run the action repeatedly until the long-running task completes
const program = Effect.race(
Effect.repeat(action, schedule),
longRunningEffect
)

Effect.runPromise(program)
/* 
Output:
action...
action...
action...
action...
done 
*/
```

---

### Repeat Effect with Termination using Effect TS

Source: https://effect.website/docs/stream/creating

Repeats the evaluation of an effect and allows for stream termination based on specific conditions. This function is suitable for scenarios where you need to process elements from an effectful source until a certain condition is met, such as draining an iterator. The example demonstrates draining an iterator.

```typescript
import { Stream, Effect, Option } from "effect"

const drainIterator = (it: Iterator): Stream.Stream<unknown, unknown, unknown> => 
  Stream.repeatEffectOption(
    Effect.sync(() => it.next())
  )
```

---

### Retry on Specific Error Codes with Effect.js

Source: https://effect.website/docs/scheduling/examples

This example shows how to configure Effect.js to retry an operation only when a specific error condition is met, such as an HTTP 401 Unauthorized response. It defines a custom error class and uses `Effect.retry` with a condition based on the error status. Errors not matching the condition will not trigger a retry.

```typescript
import { Console, Effect, Data } from "effect"

// Custom error class for handling status codes
class Err extends Data.TaggedError("Err")<{ 
readonly message: string 
readonly status: number 
}> {} 

// Function to make the API call
const getJson = (url: string) =>
Effect.tryPromise({
try: () =>
fetch(url).then((res) => {
if (!res.ok) {
console.log(res.status)
throw new Err({ message: res.statusText, status: res.status })
}
return res.json() as unknown
}),
catch: (e) => e as Err
})

// Program that retries only when the error status is 401 (Unauthorized)
const program = (url: string) =>
getJson(url).pipe(
Effect.retry({ while: (err) => err.status === 401 }),
Effect.catchAll(Console.error)
)

// Test case: API returns 401 (triggers multiple retries)
Effect.runFork(
program("https://dummyjson.com/auth/products/1?delay=1000")
)
/* 
Output:
401
401
401
401
... 
*/ 

// Test case: API returns 404 (no retries)
Effect.runFork(program("https://dummyjson.com/-"))
/* 
Output:
404
Err [Error]: Not Found 
*/
```
