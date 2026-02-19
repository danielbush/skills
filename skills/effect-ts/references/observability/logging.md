# Effect – Logging

Log levels, structured loggers, log spans, annotations, custom loggers, and file logging.

---

### Log at Info Level with Effect.js

Source: https://effect.website/docs/observability/logging

Provides an example of logging messages specifically at the `INFO` level using `Effect.logInfo`. This is the default level and is typically used for general application events.

```typescript
import { Effect } from "effect"

const program = Effect.gen(function* () {
  yield* Effect.logInfo("start")
  yield* Effect.sleep("2 seconds")
  yield* Effect.sleep("1 second")
  yield* Effect.logInfo("done")
})

Effect.runFork(program)
/* 
Output:
timestamp=... level=INFO message=start 
timestamp=... level=INFO message=done <-- 3 seconds later 
*/
```

---

### Disabling Logging with Logger.withMinimumLogLevel

Source: https://effect.website/docs/observability/logging

This example demonstrates how to disable all logging output by setting the minimum log level to `LogLevel.None` using `Logger.withMinimumLogLevel`. This is useful during testing or when no log output is desired.

```typescript
import { Effect, Logger, LogLevel } from "effect"

const program = Effect.gen(function* () {
  yield* Effect.log("Executing task...")
  yield* Effect.sleep("100 millis")
  console.log("task done")
})

// Default behavior: logging enabled
// Effect.runFork(program)

// Disable logging by setting minimum log level to 'None'
Effect.runFork(program.pipe(Logger.withMinimumLogLevel(LogLevel.None)))
```

---

### Effect Logging with structuredLogger in TypeScript

Source: https://effect.website/docs/observability/logging

This example demonstrates the structuredLogger for detailed, object-based logs, useful for traceability and analysis. The Effect program is configured with Logger.structured, outputting logs as JSON objects.

```typescript
import { Effect, Logger } from "effect"

const program = Effect.log("msg1", "msg2", ["msg3", "msg4"]).pipe(
  Effect.delay("100 millis"),
  Effect.annotateLogs({ key1: "value1", key2: "value2" }),
  Effect.withLogSpan("myspan")
)

Effect.runFork(program.pipe(Effect.provide(Logger.structured)))
```

---

### Effect Logging with logfmtLogger in TypeScript

Source: https://effect.website/docs/observability/logging

This example showcases the logfmtLogger for compact, human-readable logs. It configures the Effect program to use logfmt formatting by providing Logger.logFmt. The output is a key-value string format without extra spaces.

```typescript
import { Effect, Logger } from "effect"

const program = Effect.log("msg1", "msg2", ["msg3", "msg4"]).pipe(
  Effect.delay("100 millis"),
  Effect.annotateLogs({ key1: "value1", key2: "value2" }),
  Effect.withLogSpan("myspan")
)

Effect.runFork(program.pipe(Effect.provide(Logger.logFmt)))
```

---

### Measuring Task Duration with Log Spans

Source: https://effect.website/docs/observability/logging

This example shows how to use `Effect.withLogSpan` to measure and log the duration of a specific task. A log span is applied to a section of the program, and the duration of that section is automatically logged.

```typescript
import { Effect } from "effect"

const program = Effect.gen(function* () {
  // Simulate a delay to represent a task taking time
  yield* Effect.sleep("1 second")
  // Log a message indicating the job is done
  yield* Effect.log("The job is finished!")
}).pipe(
  // Apply a log span labeled "myspan" to measure
  // the duration of this operation
  Effect.withLogSpan("myspan")
)

Effect.runFork(program)
```

---

### Directing Logs to a File with PlatformLogger

Source: https://effect.website/docs/platform/platformlogger

This example demonstrates how to use PlatformLogger.toFile to direct log output to a file. It initializes a logfmtLogger, pipes it through toFile to specify the output file path, and then replaces the default logger with this new file logger. The NodeFileSystem.layer is provided to enable file system access. The program then logs a message, which is written to the specified file instead of the console.

```typescript
import { PlatformLogger } from "@effect/platform"
import { NodeFileSystem } from "@effect/platform-node"
import { Effect, Layer, Logger } from "effect"

// Create a string-based logger (logfmtLogger in this case)
const myStringLogger = Logger.logfmtLogger

// Apply toFile to write logs to "/tmp/log.txt"
const fileLogger = myStringLogger.pipe(
  PlatformLogger.toFile("/tmp/log.txt")
)

// Replace the default logger, providing NodeFileSystem
// to access the file system
const LoggerLive = Logger.replaceScoped(
  Logger.defaultLogger,
  fileLogger
).pipe(Layer.provide(NodeFileSystem.layer))

const program = Effect.log("Hello")

// Run the program, writing logs to /tmp/log.txt
Effect.runFork(program.pipe(Effect.provide(LoggerLive)))
/* 
Logs will be written to "/tmp/log.txt" in the logfmt format,
and won't appear on the console. 
*/
```

---

### Combining Loggers with Logger.zip in TypeScript

Source: https://effect.website/docs/observability/logging

This example demonstrates combining multiple loggers using Logger.zip. A custom console logger is created and then zipped with the default logger. When the program logs a message, it's processed by both loggers.

```typescript
import { Effect, Logger } from "effect"

// Define a custom logger that logs to the console
const logger = Logger.make(({ logLevel, message }) => {
  globalThis.console.log(`[${logLevel.label}] ${message}`)
})

// Combine the default logger and the custom logger
const combined = Logger.zip(Logger.defaultLogger, logger)

const program = Effect.log("something")

Effect.runFork(
  program.pipe(
    // Replace the default logger with the combined logger
    Effect.provide(Logger.withMinimumLogLevel(combined, 1))
  )
)
```

---

### Disabling Logging with a Custom Runtime

Source: https://effect.website/docs/observability/logging

This example illustrates how to disable logging by creating a custom runtime that is configured to turn off logging. This method provides a global way to control logging behavior for the entire program execution.

```typescript
import { Effect, Logger, LogLevel, ManagedRuntime } from "effect"

const program = Effect.gen(function* () {
  yield* Effect.log("Executing task...")
  yield* Effect.sleep("100 millis")
  console.log("task done")
})

// Create a custom runtime that disables logging
const customRuntime = ManagedRuntime.make(
  Logger.minimumLogLevel(LogLevel.None)
)

// Run the program using the custom runtime
customRuntime.runFork(program)
```

---

### Log Error with Effect

Source: https://effect.website/docs/observability/logging

Shows how to log a message at the ERROR level using Effect.logError. This level is for issues that require attention. The example handles a failing task and logs the error.

```typescript
import { Effect, Either } from "effect"

const task = Effect.fail("Oh uh!").pipe(Effect.as(2))

const program = Effect.gen(function*() {
  const failureOrSuccess = yield* Effect.either(task)
  if (Either.isLeft(failureOrSuccess)) {
    yield* Effect.logError(failureOrSuccess.left)
    return 0
  } else {
    return failureOrSuccess.right
  }
})

Effect.runFork(program)
```

---

### Disabling Logging with a Layer

Source: https://effect.website/docs/observability/logging

This example shows how to disable logging by providing a layer that sets the minimum log level to `LogLevel.None`. This approach is effective for disabling logging across a larger part of an application or within specific contexts.

```typescript
import { Effect, Logger, LogLevel } from "effect"

const program = Effect.gen(function* () {
  yield* Effect.log("Executing task...")
  yield* Effect.sleep("100 millis")
  console.log("task done")
})

// Create a layer that disables logging
const layer = Logger.minimumLogLevel(LogLevel.None)

// Apply the layer to disable logging
Effect.runFork(program.pipe(Effect.provide(layer)))
```

---

### Custom Logger with Effect.js

Source: https://effect.website/docs/runtime

Demonstrates replacing the default logger with a custom implementation using Logger.replace and applying it temporarily using Effect.provide. This allows for specific logging configurations within certain parts of an Effect program.

```typescript
import { Effect, Logger } from "effect"

// Custom logger implementation
const addSimpleLogger = Logger.make(({ message }) => console.log(message))

const removeDefaultLogger = Logger.remove(Logger.defaultLogger)

const program = Effect.gen(function* () {
  // Logs with default logger
  yield* Effect.log("Application started!")

  yield* Effect.gen(function* () {
    // This log is suppressed
    yield* Effect.log("I'm not going to be logged!")

    // Custom logger applied here
    yield* Effect.log("I will be logged by the simple logger.").pipe(
      Effect.provide(addSimpleLogger)
    )

    // This log is suppressed
    yield* Effect.log("Reset back to the previous configuration, so I won't be logged.")
  }).pipe(
    // Remove the default logger temporarily
    Effect.provide(removeDefaultLogger)
  )

  // Logs with default logger again
  yield* Effect.log("Application is about to exit!")
})

Effect.runSync(program)
```

---

### Log a Simple Message with Effect.js

Source: https://effect.website/docs/observability/logging

Demonstrates how to log a single message at the default INFO level using `Effect.log`. The output includes timestamp, log level, fiber ID, and the message content.

```typescript
import { Effect } from "effect"

const program = Effect.log("Application started")

Effect.runFork(program)
/* 
Output:
timestamp=... level=INFO fiber=#0 message="Application started"
*/
```

---

### Define a Simple Custom Logger

Source: https://effect.website/docs/observability/logging

Illustrates how to create a custom logger using Logger.make. This logger formats log messages to include the log level's label and the message itself, then logs them to the console.

```typescript
import { Logger } from "effect"

// Custom logger that outputs log messages to the console
const logger = Logger.make(({ logLevel, message }) => {
  globalThis.console.log(`[${logLevel.label}] ${message}`)
})
```

---

### Log Multiple Messages with Effect.js

Source: https://effect.website/docs/observability/logging

Shows how to log multiple string messages simultaneously using `Effect.log`. All provided messages are appended to the log entry with their respective keys.

```typescript
import { Effect } from "effect"

const program = Effect.log("message1", "message2", "message3")

Effect.runFork(program)
/* 
Output:
timestamp=... level=INFO fiber=#0 message=message1 message=message2 message=message3 
*/
```

---

### PlatformLogger.toFile

Source: https://effect.website/docs/platform/platformlogger

Creates a new logger from an existing string-based logger, writing its output to the specified file. It supports optional batching with `batchWindow`.

```APIDOC
## PlatformLogger.toFile

### Description
Creates a new logger from an existing string-based logger, writing its output to the specified file. If a `batchWindow` duration is provided, logs are batched before writing, which can reduce overhead. Without `batchWindow`, logs are written as they arrive. This function returns an `Effect` that may fail with a `PlatformError` if the file cannot be opened or written to.

### Method
Pipeable function (part of Effect's pipeable API)

### Endpoint
N/A (This is a library function, not a network endpoint)

### Parameters
#### Path Parameters
N/A

#### Query Parameters
N/A

#### Request Body
N/A

### Request Example
(Example for directing logs to a file)
```javascript
import { PlatformLogger } from "@effect/platform"
import { NodeFileSystem } from "@effect/platform-node"
import { Effect, Layer, Logger } from "effect"

// Create a string-based logger
const myStringLogger = Logger.logfmtLogger

// Apply toFile to write logs to "/tmp/log.txt"
const fileLogger = myStringLogger.pipe(
  PlatformLogger.toFile("/tmp/log.txt")
)

// Replace the default logger, providing NodeFileSystem
const LoggerLive = Logger.replaceScoped(
  Logger.defaultLogger,
  fileLogger
).pipe(Layer.provide(NodeFileSystem.layer))

const program = Effect.log("Hello")

// Run the program, writing logs to /tmp/log.txt
Effect.runFork(program.pipe(Effect.provide(LoggerLive)))
/*
Logs will be written to "/tmp/log.txt" in the logfmt format,
and won't appear on the console.
*/
```

(Example for directing logs to both a file and the console)
```javascript
import { PlatformLogger } from "@effect/platform"
import { NodeFileSystem } from "@effect/platform-node"
import { Effect, Layer, Logger } from "effect"

// Create a string-based logger for the file
const fileLogger = Logger.logfmtLogger.pipe(
  PlatformLogger.toFile("/tmp/app.log")
)

// Create a pretty logger for the console
const consoleLogger = Logger.prettyLogger

// Combine loggers: fileLogger for file output, consoleLogger for console
const combinedLogger = Logger.combine(fileLogger, consoleLogger)

// Provide FileSystem for file operations and replace the default logger
const LoggerLive = Logger.replaceScoped(
  Logger.defaultLogger,
  combinedLogger
).pipe(Layer.provide(NodeFileSystem.layer))

const program = Effect.all([
  Effect.log("This log goes to both file and console."),
  Effect.logError("This error goes to both file and console.")
])

// Run the program
Effect.runFork(program.pipe(Effect.provide(LoggerLive)))
/*
Logs will be written to "/tmp/app.log" in logfmt format
AND displayed on the console using the pretty format.
*/
```

### Response
#### Success Response (200)
N/A (This function returns an Effect that produces a Logger)

#### Response Example
N/A
```

---

### Configure Combined Console and File Logging in Effect-TS

Source: https://effect.website/docs/platform/platformlogger

This snippet demonstrates how to create a logger that writes to both the console (using a pretty format) and a file (using logfmt format). It utilizes Effect-TS's Layer system to provide the configured logger to the program. Dependencies include 'effect' and 'node:fs'.

```typescript
import { Effect, Layer, Logger } from "effect"
import * as PlatformLogger from "@effect/platform/Logger"
import * as NodeFileSystem from "@effect/platform-node/FileSystem"

const fileLogger = Logger.logfmtLogger.pipe(
  PlatformLogger.toFile("/tmp/log.txt")
)

const bothLoggers = Effect.map(fileLogger, (fileLogger) =>
  Logger.zip(Logger.prettyLoggerDefault, fileLogger)
)

const LoggerLive = Logger.replaceScoped(
  Logger.defaultLogger,
  bothLoggers
).pipe(Layer.provide(NodeFileSystem.layer))

const program = Effect.log("Hello")

// Run the program, writing logs to both the console (pretty format)
// and "/tmp/log.txt" (logfmt)
Effect.runFork(program.pipe(Effect.provide(LoggerLive)))
```

---

### Provide Combined Logger in Effect.js

Source: https://effect.website/docs/observability/logging

This snippet demonstrates how to replace the default logger with a combined logger in Effect.js. It utilizes Effect.provide to set up the logger context for the application. Ensure the `combined` logger is correctly instantiated before this step.

```typescript
Effect.provide(Logger.replace(Logger.defaultLogger, combined))
```

---

### Effect Logging with prettyLogger in TypeScript

Source: https://effect.website/docs/observability/logging

This snippet illustrates the use of prettyLogger for enhanced readability with colors and indentation, ideal for development. The Effect program is set up to use Logger.pretty, producing visually appealing console output.

```typescript
import { Effect, Logger } from "effect"

const program = Effect.log("msg1", "msg2", ["msg3", "msg4"]).pipe(
  Effect.delay("100 millis"),
  Effect.annotateLogs({ key1: "value1", key2: "value2" }),
  Effect.withLogSpan("myspan")
)

Effect.runFork(program.pipe(Effect.provide(Logger.pretty)))
```

---

### Effect Logging Utilities

Source: https://context7_llms

Overview of Effect's logging capabilities, including dynamic log levels, custom output destinations, and fine-grained control over log messages.

```typescript
import * as Logger from "@effect/io/Logger"
import * as Effect from "@effect/io/Effect"

const myLogger = Logger.make(({ message }) => {
  console.log(`[INFO] ${message}`)
})

const logEffect = Logger.log("Hello from Effect logging!")

Effect.provide(logEffect, Logger.using(myLogger))
// Output: [INFO] Hello from Effect logging!
```
