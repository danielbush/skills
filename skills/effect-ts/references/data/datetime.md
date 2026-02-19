# Effect – DateTime, Duration & Clock

`DateTime` (UTC and Zoned), `Duration`, `Clock`, `Cron`, and time utilities.

---

### Using Default Clock and Console Services - Effect.js

Source: https://effect.website/docs/requirements-management/default-services

This example demonstrates how to use the default Clock and Console services within an Effect.js program. Effect automatically supplies live versions of these services, simplifying setup. The program retrieves the current time in milliseconds and logs it to the console.

```typescript
import { Effect, Clock, Console } from "effect"

const program = Effect.gen(function* () {
  const now = yield* Clock.currentTimeMillis
  yield* Console.log(`Application started at ${new Date(now)}`)
})

Effect.runFork(program)
```

---

### Get Current UTC Time Immediately

Source: https://effect.website/docs/data-types/datetime

Retrieves the current UTC time immediately using Date.now(), without relying on the Clock service. This is a direct and potentially less controlled method for getting the current time. Requires 'effect' library.

```typescript
import { DateTime } from "effect"

const currentTime = DateTime.unsafeNow()
```

---

### UTC Constructors

Source: https://effect.website/docs/data-types/datetime

Demonstrates how to create UTC DateTimes from various inputs like JavaScript Date objects, partial date parts, and strings.

```APIDOC
## UTC Constructors

### Description

Creates a UTC `DateTime` from various inputs. The input is interpreted as a local time and then converted to UTC by subtracting the timezone offset. If the input is invalid, an `IllegalArgumentException` is thrown.

### Method

`DateTime.make(input)`

### Parameters

#### Request Body

- **input** (Date | { year: number, month?: number, date?: number, hours?: number, minutes?: number, seconds?: number, milliseconds?: number } | string) - Required - The input value to create a DateTime from.

### Request Example

```javascript
import { DateTime } from "effect"

// From a JavaScript Date
const maybeUtc1 = DateTime.make(new Date("2025-01-01 04:00:00"))
console.log(maybeUtc1)
/*
Output:
{
  _id: 'Option',
  _tag: 'Some',
  value: '2025-01-01T03:00:00.000Z'
}
*/

// From partial date parts
const maybeUtc2 = DateTime.make({ year: 2025 })
console.log(maybeUtc2)
/*
Output:
{
  _id: 'Option',
  _tag: 'Some',
  value: '2025-01-01T00:00:00.000Z'
}
*/

// From a string
const maybeUtc3 = DateTime.make("2025-01-01")
console.log(maybeUtc3)
/*
Output:
{
  _id: 'Option',
  _tag: 'Some',
  value: '2025-01-01T00:00:00.000Z'
}
*/
```

### Response

#### Success Response (200)

- **_id** (string) - The type of the Option, either 'Some' or 'None'.
- **_tag** (string) - Indicates if the value is 'Some' or 'None'.
- **value** (string) - The UTC DateTime string in ISO 8601 format if successful.

#### Response Example

```json
{
  "_id": "Option",
  "_tag": "Some",
  "value": "2025-01-01T03:00:00.000Z"
}
```
```

---

### Get Duration Value in Milliseconds (TypeScript)

Source: https://effect.website/docs/data-types/duration

Retrieves the value of a duration in milliseconds using `Duration.toMillis`. This function is useful for obtaining a numerical representation of time for calculations or logging.

```typescript
import { Duration } from "effect"

console.log(Duration.toMillis(Duration.seconds(30)))
// Output: 30000
```

---

### Get Current UTC Time with Effect Clock

Source: https://effect.website/docs/data-types/datetime

Retrieves the current UTC time as an Effect using the Clock service. This ensures consistent time across the application, especially useful for testing. Requires 'effect' library.

```typescript
import { DateTime, Effect } from "effect"

const program = Effect.gen(function* () {
  const currentTime = yield* DateTime.now
})
```

---

### Get Duration Value in Nanoseconds (TypeScript)

Source: https://effect.website/docs/data-types/duration

Retrieves the value of a duration in nanoseconds using `Duration.toNanos`. This function returns an `Option` because durations can be infinite. Use this for high-precision time measurements.

```typescript
import { Duration } from "effect"

console.log(Duration.toNanos(Duration.millis(100)))
/*
Output:
{
  _id: 'Option',
  _tag: 'Some',
  value: 100000000n
}
*/
```

---

### Handling Operation Exceeding Timeout Duration (TypeScript)

Source: https://effect.website/docs/error-management/timing-out

This example shows what happens when an Effect operation exceeds the specified timeout duration. A TimeoutException is raised, indicating that the operation did not complete within the allocated time.

```typescript
import { Effect } from "effect"

const task = Effect.gen(function* () {
  console.log("Start processing...")
  yield* Effect.sleep("2 seconds") // Simulates a delay in processing
  console.log("Processing complete.")
  return "Result"
})

// Output will show a TimeoutException as the task takes longer
// than the specified timeout duration
const timedEffect = task.pipe(Effect.timeout("1 second"))

Effect.runPromiseExit(timedEffect).then(console.log)
/*
Output:
Start processing...
{ _id: 'Exit', _tag: 'Failure', cause: { _id: 'Cause', _tag: 'Fail', error: { _id: 'TaggedError', _tag: 'TimeoutException' } } }
*/
```

---

### Unsafely Get Duration Value in Nanoseconds (TypeScript)

Source: https://effect.website/docs/data-types/duration

Retrieves the value of a duration in nanoseconds as a `bigint` using `Duration.unsafeToNanos`. This method throws an error for infinite durations, so use it only when you are certain the duration is finite.

```typescript
import { Duration } from "effect"

console.log(Duration.unsafeToNanos(Duration.millis(100)))
// Output: 100000000n

console.log(Duration.unsafeToNanos(Duration.infinity))
/*
throws:
Error: Cannot convert infinite duration to nanos
...
*/
```

---

### Create Utc from Various Inputs (Effect-TS)

Source: https://effect.website/docs/data-types/datetime

Shows how to create Utc DateTime instances using `DateTime.unsafeMake` with different input types: a JavaScript Date, partial date parts, and a string.

```typescript
import { DateTime } from "effect"

// From a JavaScript Date
const utc1 = DateTime.unsafeMake(new Date("2025-01-01 04:00:00"))
console.log(utc1)
// Output: DateTime.Utc(2025-01-01T03:00:00.000Z)

// From partial date parts
const utc2 = DateTime.unsafeMake({ year: 2025 })
console.log(utc2)
// Output: DateTime.Utc(2025-01-01T00:00:00.000Z)

// From a string
const utc3 = DateTime.unsafeMake("2025-01-01")
console.log(utc3)
// Output: DateTime.Utc(2025-01-01T00:00:00.000Z)
```

---

### Create UTC DateTime from JavaScript Date

Source: https://effect.website/docs/data-types/datetime

Converts a JavaScript Date object to a UTC DateTime object using the 'effect' library. It handles potential `None` results if the input is invalid. The example demonstrates conversion from a specific date and time, showing the resulting UTC value.

```javascript
import { DateTime } from "effect"

// From a JavaScript Date
const maybeUtc1 = DateTime.make(new Date("2025-01-01 04:00:00"))
console.log(maybeUtc1)
/* 
Output:
{ _id: 'Option', _tag: 'Some', value: '2025-01-01T03:00:00.000Z' } 
*/
```

---

### Do Simulation: Measure Elapsed Time with Effect.js

Source: https://effect.website/docs/code-style/do

This snippet demonstrates how to measure the elapsed time of an Effect using the Do Simulation pattern. It binds the start and end timestamps and calculates the difference, logging it to the console. It requires the 'effect' library, specifically Effect and Console.

```typescript
import { Effect, Console } from "effect"

// Get the current timestamp
const now = Effect.sync(() => new Date().getTime())

const elapsed = (
  self: Effect.Effect
): Effect.Effect =>
Effect.Do.pipe(
  Effect.bind("startMillis", () => now),
  Effect.bind("result", () => self),
  Effect.bind("endMillis", () => now),
  Effect.let(
    "elapsed",
    // Calculate the elapsed time in milliseconds
    ({ startMillis, endMillis }) => endMillis - startMillis
  ),
  // Log the elapsed time
  Effect.tap(({ elapsed }) => Console.log(`Elapsed: ${elapsed}`)),
  Effect.map(({ result }) => result)
)

// Simulates a successful computation with a delay of 200 milliseconds
const task = Effect.succeed("some task").pipe(Effect.delay("200 millis"))

const program = elapsed(task)

Effect.runPromise(program).then(console.log)
/* 
Output:
Elapsed: 204
some task 
*/
```

---

### Effect.gen: Measure Elapsed Time with Effect.js

Source: https://effect.website/docs/code-style/do

This snippet shows how to measure the elapsed time of an Effect using the Effect.gen pattern, which leverages generator functions for a more concise and readable syntax. It binds start and end timestamps using yield* and calculates the duration. It requires the 'effect' library.

```typescript
import { Effect } from "effect"

// Get the current timestamp
const now = Effect.sync(() => new Date().getTime())

// Prints the elapsed time occurred to `self` to execute
const elapsed = (
  self: Effect.Effect
): Effect.Effect =>
Effect.gen(function* () {
  const startMillis = yield* now
  const result = yield* self
  const endMillis = yield* now
  // Calculate the elapsed time in milliseconds
  const elapsed = endMillis - startMillis
  // Log the elapsed time
  console.log(`Elapsed: ${elapsed}`)
  return result
})

// Simulates a successful computation with a delay of 200 milliseconds
const task = Effect.succeed("some task").pipe(Effect.delay("200 millis"))

const program = elapsed(task)

Effect.runPromise(program).then(console.log)
/* 
Output:
Elapsed: 204
some task 
*/
```

---

### Zoned Constructors

Source: https://effect.website/docs/data-types/datetime

Provides methods for creating Zoned DateTimes, which include epoch milliseconds and a TimeZone.

```APIDOC
## Zoned Constructors

A `Zoned` includes `epochMillis` along with a `TimeZone`, allowing you to represent a specific point in time with an associated time zone.

### unsafeMakeZoned

Creates a `Zoned` by combining a DateTime.Input with an optional `TimeZone`. The time zone can be provided as a `TimeZone` object, a string identifier (e.g., `"Europe/London"`), or a numeric offset in milliseconds. Throws `IllegalArgumentException` if the input or time zone is invalid.

#### Parameters

- **input** (Date | string | { year: number, month?: number, date?: number, hours?: number, minutes?: number, seconds?: number, milliseconds?: number }) - Required - The input value to create a Zoned DateTime from.
- **options** (object) - Optional - Configuration options.
  - **timeZone** (TimeZone | string | number) - The time zone to associate with the Zoned DateTime.
  - **adjustForTimeZone** (boolean) - If true, interprets the input date as being in the specified time zone. Defaults to false.

#### Request Example (System's Local Time Zone)

```javascript
import { DateTime } from "effect"

// Assuming execution in Italy (CET timezone, UTC+1 in January)
const zoned = DateTime.unsafeMakeZoned(new Date("2025-01-01 04:00:00"))
console.log(zoned)
// Output: DateTime.Zoned(2025-01-01T04:00:00.000+01:00)
console.log(zoned.zone)
// Output: TimeZone.Offset(+01:00)
```

#### Request Example (Specified Named Time Zone)

```javascript
import { DateTime } from "effect"

// Assuming execution in Italy (CET timezone)
const zoned = DateTime.unsafeMakeZoned(new Date("2025-01-01 04:00:00"), {
  timeZone: "Europe/Rome"
})
console.log(zoned)
// Output: DateTime.Zoned(2025-01-01T04:00:00.000+01:00[Europe/Rome])
console.log(zoned.zone)
// Output: TimeZone.Named(Europe/Rome)
```

#### Request Example (Adjusting for Time Zone Interpretation)

```javascript
import { DateTime } from "effect"

// Assuming execution in Italy (CET timezone)
const zoned = DateTime.unsafeMakeZoned(new Date("2025-01-01 04:00:00"), {
  timeZone: "Europe/Rome",
  adjustForTimeZone: true
})
console.log(zoned)
// Output: DateTime.Zoned(2025-01-01T03:00:00.000+01:00[Europe/Rome])
console.log(zoned.zone)
// Output: TimeZone.Named(Europe/Rome)
```

### makeZoned

Works like `unsafeMakeZoned` but returns an `Option`. Returns `None` if the input is invalid, otherwise returns `Some` containing the `Zoned` DateTime.

#### Parameters

- **input** (Date | string | { year: number, month?: number, date?: number, hours?: number, minutes?: number, seconds?: number, milliseconds?: number }) - Required - The input value to create a Zoned DateTime from.
- **options** (object) - Optional - Configuration options.
  - **timeZone** (TimeZone | string | number) - The time zone to associate with the Zoned DateTime.
  - **adjustForTimeZone** (boolean) - If true, interprets the input date as being in the specified time zone. Defaults to false.

#### Request Example

```javascript
import { DateTime, Option } from "effect"

const zoned = DateTime.makeZoned(new Date("2025-01-01 04:00:00"), {
  timeZone: "Europe/Rome"
})

if (Option.isSome(zoned)) {
  console.log("The DateTime is valid")
}
```

### makeZonedFromString

Creates a `Zoned` by parsing a string in the format `YYYY-MM-DDTHH:mm:ss.sss+HH:MM[IANA timezone identifier]`. Returns `Some` with the `Zoned` if valid, otherwise returns `None`.

#### Parameters

- **input** (string) - Required - The string to parse.

#### Request Example

```javascript
import { DateTime, Option } from "effect"

// Example of a valid zoned string
const zonedFromString = DateTime.makeZonedFromString("2025-01-01T04:00:00.000+01:00[Europe/Rome]")

if (Option.isSome(zonedFromString)) {
  console.log("Successfully parsed Zoned DateTime from string.")
}
```
```

---

### Measure Elapsed Time of Effect with pipe

Source: https://effect.website/docs/code-style/do

This code snippet demonstrates how to measure the elapsed time of an Effect using the standard 'pipe' method. It shows the initial approach which can lead to excessive nesting and verbosity. The function 'elapsed' wraps another Effect, recording the start and end times to calculate and log the duration.

```typescript
import { Effect, Console } from "effect"

// Get the current timestamp
const now = Effect.sync(() => new Date().getTime())

// Prints the elapsed time occurred to `self` to execute
const elapsed = (
  self: Effect.Effect
): Effect.Effect =>
  now.pipe(
    Effect.andThen((startMillis) =>
      self.pipe(
        Effect.andThen((result) =>
          now.pipe(
            Effect.andThen((endMillis) => {
              // Calculate the elapsed time in milliseconds
              const elapsed = endMillis - startMillis
              // Log the elapsed time
              return Console.log(`Elapsed: ${elapsed}`).pipe(
                Effect.map(() => result)
              )
            })
          )
        )
      )
    )
  )

// Simulates a successful computation with a delay of 200 milliseconds
const task = Effect.succeed("some task").pipe(Effect.delay("200 millis"))

const program = elapsed(task)

Effect.runPromise(program).then(console.log)
/* 
Output:
Elapsed: 204 
some task 
*/
```
