# Effect – Configuration

`Config`, `ConfigProvider`, environment variables, nested namespaces, and mocking config in tests.

---

### Loading Log Level from Configuration

Source: https://effect.website/docs/observability/logging

This example demonstrates how to dynamically load the desired log level from configuration and apply it to the program using the `Logger.minimumLogLevel` layer. This allows for flexible control over logging verbosity based on external settings.

```typescript
import {
  Effect,
  Config,
  Logger,
  Layer,
  ConfigProvider,
  LogLevel
} from "effect"

// Simulate a program with logs
const program = Effect.gen(function* () {
  yield* Effect.logError("ERROR!")
  yield* Effect.logWarning("WARNING!")
  yield* Effect.logInfo("INFO!")
  yield* Effect.logDebug("DEBUG!")
})

// Load the log level from the configuration and apply it as a layer
```

---

### Basic Configuration Retrieval (Effect-TS)

Source: https://effect.website/docs/configuration

Retrieves 'HOST' and 'PORT' environment variables and logs the application start URL. Requires 'HOST' and 'PORT' to be set in the environment.

```typescript
import { Effect, Config } from "effect"

const program = Effect.gen(function* () {
  const hostPort = yield* Config.all([
    Config.string("HOST"),
    Config.number("PORT")
  ]);
  console.log(`Application started: ${hostPort.url}`)
});

Effect.runPromise(program)
```

---

### Providing Configuration to a Nested Workflow (JavaScript)

Source: https://effect.website/docs/runtime

Demonstrates applying a custom logger configuration only to a specific nested section of a program using Effect.provide. The example highlights how the configuration is localized and reverts after the nested block.

```javascript
import { Logger, Effect } from "effect"

const addSimpleLogger = Logger.replace(
  Logger.defaultLogger,
  Logger.make(({ message }) => console.log(message))
)

const nestedProgram = Effect.gen(function* () {
  yield* Effect.log("Inside nested program")
})

const mainProgram = Effect.gen(function* () {
  yield* Effect.log("Before nested program")
  yield* nestedProgram.pipe(Effect.provide(addSimpleLogger))
  yield* Effect.log("After nested program")
})

Effect.runFork(mainProgram)
/* 
Output:
timestamp=... level=INFO fiber=#0 message="Before nested program"
[ 'Inside nested program' ]
timestamp=... level=INFO fiber=#0 message="After nested program"
*/
```

---

### Overriding Logger Configuration with Effect.provide (JavaScript)

Source: https://effect.website/docs/runtime

Illustrates how to temporarily override the default logger with a custom implementation using Effect.provide. The example shows a program logging messages with and without timestamps/levels, demonstrating the effect of the provided configuration.

```javascript
import { Logger, Effect } from "effect"

const addSimpleLogger = Logger.replace(
  Logger.defaultLogger,
  // Custom logger implementation
  Logger.make(({ message }) => console.log(message))
)

const program = Effect.gen(function* () {
  yield* Effect.log("Application started!")
  yield* Effect.log("Application is about to exit!")
})

// Running with the default logger
Effect.runFork(program)
/* 
Output:
timestamp=... level=INFO fiber=#0 message="Application started!"
timestamp=... level=INFO fiber=#0 message="Application is about to exit!"
*/

// Overriding the default logger with a custom one
Effect.runFork(program.pipe(Effect.provide(addSimpleLogger)))
/* 
Output:
[ 'Application started!' ]
[ 'Application is about to exit!' ]
*/
```

---

### Provide Default Configuration Values (Effect/TypeScript)

Source: https://effect.website/docs/configuration

Shows how to provide default values for configuration parameters using Config.withDefault. If an environment variable is not set, the specified default value is used. This example sets a default port of 8080.

```typescript
import { Effect, Config } from "effect"

const program = Effect.gen(function* () {
  const host = yield* Config.string("HOST")
  // Use default 8080 if PORT is not set
  const port = yield* Config.number("PORT").pipe(Config.withDefault(8080))
  console.log(`Application started: ${host}:${port}`)
})

Effect.runPromise(program)
```

---

### Load Config from JSON with Nested Namespaces (Effect-TS)

Source: https://effect.website/docs/configuration

Illustrates how to load configuration values from a JSON object using ConfigProvider.fromJson. This example demonstrates reading nested configuration values like 'SERVER.PORT' and 'SERVER.HOST'.

```typescript
import { Config, ConfigProvider, Effect } from "effect"

const program = Effect.gen(function* () {
  // Read SERVER_HOST and SERVER_PORT as nested configuration values
  const port = yield* Config.nested(Config.number("PORT"), "SERVER")
  const host = yield* Config.nested(Config.string("HOST"), "SERVER")
  console.log(`Application started: ${host}:${port}`)
})

Effect.runPromise(
  Effect.withConfigProvider(
    program,
    ConfigProvider.fromJson(
      JSON.parse(`{"SERVER":{"PORT":8080,"HOST":"localhost"}}`)
    )
  )
)
```

---

### Load Basic Environment Variables (Effect/TypeScript)

Source: https://effect.website/docs/configuration

Demonstrates loading 'HOST' as a string and 'PORT' as a number from environment variables using Effect's Config module. Requires environment variables to be set for successful execution. Errors are thrown if variables are missing.

```typescript
import { Effect, Config } from "effect"

// Define a program that loads HOST and PORT configuration
const program = Effect.gen(function* () {
  const host = yield* Config.string("HOST") // Read as a string
  const port = yield* Config.number("PORT") // Read as a number

  console.log(`Application started: ${host}:${port}`)
})

Effect.runPromise(program)
```

---

### Use Nested Configuration Namespaces with Environment Variables (Effect-TS)

Source: https://effect.website/docs/configuration

Explains how to use ConfigProvider.nested to group configuration values under a specific namespace, such as 'SERVER'. This example reads 'PORT' and 'HOST' from environment variables prefixed with 'SERVER_'.

```typescript
import { Config, ConfigProvider, Effect } from "effect"

const program = Effect.gen(function* () {
  const port = yield* Config.number("PORT") // Reads SERVER_PORT
  const host = yield* Config.string("HOST") // Reads SERVER_HOST
  console.log(`Application started: ${host}:${port}`)
})

Effect.runPromise(
  Effect.withConfigProvider(
    program,
    ConfigProvider.fromEnv().pipe(
      // Uses SERVER as a namespace
      ConfigProvider.nested("SERVER")
    )
  )
)
```

---

### Create Configuration for a HashMap

Source: https://effect.website/docs/configuration

Shows how to use the `Config.hashMap` combinator to define a configuration that accepts key-value pairs, typically read from environment variables with a prefix.

```typescript
import { Config, Effect } from "effect"

const program = Effect.gen(function* () {
  const config = yield* Config.hashMap(Config.string(), "MYMAP")
  console.log(config)
})

Effect.runPromise(program)
// Run:
// MYMAP_A=a MYMAP_B=b npx tsx index.ts
// Output:
// { _id: 'HashMap', values: [ [ 'A', 'a' ], [ 'B', 'b' ] ] }
```

---

### Create Configuration for a HashSet

Source: https://effect.website/docs/configuration

Demonstrates using the `Config.hashSet` combinator to create a configuration that expects a set of unique values. Duplicate inputs are handled appropriately.

```typescript
import { Config, Effect } from "effect"

const program = Effect.gen(function* () {
  const config = yield* Config.hashSet(Config.string(), "MYSET")
  console.log(config)
})

Effect.runPromise(program)
// Run:
// MYSET=a,"b c",d,a npx tsx index.ts
// Output:
// { _id: 'HashSet', values: [ 'd', 'a', 'b c' ] }
```

---

### Parse Environment Variables to Constant Case with Effect.js

Source: https://effect.website/docs/configuration

This snippet shows how to load configuration from environment variables and convert their keys to constant case using Effect.js's ConfigProvider. It requires setting environment variables like HOST and PORT. The output demonstrates the application starting with the provided host and port.

```typescript
import { ConfigProvider } from "effect/ConfigProvider"

const program = ConfigProvider.fromEnv().pipe(ConfigProvider.constantCase)

// Example environment variables setup:
// HOST=localhost PORT=8080 npx tsx your-script.ts

// Example output:
// Application started: localhost:8080
```

---

### Define Config Service Layer (Effect.js)

Source: https://effect.website/docs/requirements-management/layers

Defines a Layer for the Config service using Layer.succeed. This layer provides the 'getConfig' effect, which returns application configuration including logLevel and connection string. It's a basic setup for configuration management.

```typescript
import { Effect, Context, Layer } from "effect"

class Config extends Context.Tag("Config")<Config, {
  readonly getConfig: Effect.Effect<{
    readonly logLevel: string
    readonly connection: string
  }>
}>() {}

const ConfigLive = Layer.succeed(Config, {
  getConfig: Effect.succeed({
    logLevel: "INFO",
    connection: "mysql://username:password@hostname:port/database_name"
  })
})
```

---

### Set Custom Environment Variables for Commands

Source: https://effect.website/docs/platform/command

Shows how to set custom environment variables for a command using `Command.env`. It also demonstrates using `Command.runInShell(true)` to ensure that shell-specific variable expansions are correctly processed.

```typescript
import { Command } from "@effect/platform"
import { NodeContext, NodeRuntime } from "@effect/platform-node"
import { Effect } from "effect"

const command = Command.make("echo", "-n", "$MY_CUSTOM_VAR").pipe(
  Command.env({
    MY_CUSTOM_VAR: "Hello, this is a custom environment variable!"
  }),
  // Use shell to interpret variables correctly
  // on Windows and Unix-like systems
  Command.runInShell(true)
)

const program = Effect.gen(function* () {
  const output = yield* Command.string(command)
  console.log(output)
})

NodeRuntime.runMain(program.pipe(Effect.provide(NodeContext.layer)))
```

---

### Load Environment Variable as Array of Strings

Source: https://effect.website/docs/configuration

Illustrates using the `Config.array` combinator to parse an environment variable into an array of strings. The input format is typically comma-separated.

```typescript
import { Config, Effect } from "effect"

const program = Effect.gen(function* () {
  const config = yield* Config.array(Config.string(), "MYARRAY")
  console.log(config)
})

Effect.runPromise(program)
// Run:
// MYARRAY=a,b,c,a npx tsx index.ts
// Output:
// [ 'a', 'b', 'c', 'a' ]
```

---

### Temporarily Overriding ConfigProvider Service with Effect.withConfigProviderScoped - Effect.js

Source: https://effect.website/docs/requirements-management/default-services

This example demonstrates temporarily overriding the ConfigProvider service within a scope using `Effect.withConfigProviderScoped`. This ensures that a specific configuration is used only within the designated scope, and the original ConfigProvider is restored upon exiting the scope, preventing unintended side effects.

```typescript
import { Effect, Clock, Console, ConfigProvider } from "effect"

// Assume customConfigProvider is an implementation of the ConfigProvider service
// const customConfigProvider = ConfigProvider.fromMap(new Map([['key', 'value']]));

const program = Effect.gen(function* () {
  const config = yield* ConfigProvider.get("key")
  yield* Console.log(`Configuration value: ${config}`)
})

// Effect.withConfigProviderScoped(customConfigProvider)(program)
```

---

### Convert Configuration Keys to Constant Case (Effect-TS)

Source: https://effect.website/docs/configuration

Demonstrates the use of ConfigProvider.constantCase to transform configuration keys into uppercase with underscores. This is helpful for aligning configuration keys with environment variable naming conventions.

```typescript
import { Config, ConfigProvider, Effect } from "effect"

const program = Effect.gen(function* () {
  const port = yield* Config.number("Port") // Reads PORT
  const host = yield* Config.string("Host") // Reads HOST
  console.log(`Application started: ${host}:${port}`)
})

Effect.runPromise(
  Effect.withConfigProvider(
    program,
    ConfigProvider.fromEnv().pipe(
      ConfigProvider.constantCase()
    )
  )
)
```

---

### Load Config from Environment Variables with Custom Delimiters (Effect-TS)

Source: https://effect.website/docs/configuration

Shows how to configure ConfigProvider.fromEnv to use custom path and sequence delimiters for reading environment variables. This is essential when environment variable naming conventions differ from the defaults.

```typescript
import { Config, ConfigProvider, Effect } from "effect"

const program = Effect.gen(function* () {
  // Read SERVER_HOST and SERVER_PORT as nested configuration values
  const port = yield* Config.nested(Config.number("PORT"), "SERVER")
  const host = yield* Config.nested(Config.string("HOST"), "SERVER")
  console.log(`Application started: ${host}:${port}`)
})

Effect.runPromise(
  Effect.withConfigProvider(
    program,
    // Custom delimiters
    ConfigProvider.fromEnv({ pathDelim: "__", seqDelim: "|" })
  )
)
```

---

### Effect.js Custom Configuration Type Definition

Source: https://effect.website/docs/configuration

Shows how to define and use custom configuration types in Effect.js. It involves combining primitive configurations (like string and number) using `Config.all` and then mapping the combined values to a custom class instance.

```typescript
import { Config } from "effect"

class HostPort {
  constructor(readonly host: string, readonly port: number) {}
  get url() {
    return `${this.host}:${this.port}`
  }
}

// Combine the configuration for 'HOST' and 'PORT'
const both = Config.all([Config.string("HOST"), Config.number("PORT")])

// Map the configuration values into a HostPort instance
const config = Config.map(
  both,
  ([host, port]) => new HostPort(host, port)
)
```
