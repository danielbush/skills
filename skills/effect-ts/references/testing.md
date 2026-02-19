# Effect – Testing

Mocking services, `ConfigProvider`, file system, and test dependency injection.

---

### Creating a Test Instance of Database Service in Effect.js

Source: https://effect.website/docs/requirements-management/layers

This example demonstrates creating a test instance of the Database service in Effect.js. It simulates a 'query' function that returns an empty array, illustrating how to provide a concrete implementation for a service tag.

```typescript
import { Effect, Context } from "effect"

// Assuming Config, Logger, and Database tags are already declared
class Config extends Context.Tag("Config")() {}
class Logger extends Context.Tag("Logger")() {}
class Database extends Context.Tag("Database")<Database, {
  readonly query: (sql: string) => Effect.Effect
}>() {}

// Declaring a test instance of the Database service
const DatabaseTest = Database.of({
  // Simulating a simple response
  query: (sql: string) => Effect.succeed([])
})
```

---

### Test Database Connectivity and Run Startup Checks

Source: https://effect.website/docs/getting-started/building-pipelines

This snippet demonstrates how to test database connectivity using Effect.promise and combine it with other effects like web configuration checks using Effect.all. It then runs these startup checks and logs the results. Dependencies include the 'effect' library.

```typescript
import { Effect } from "effect"

// Assume webConfig is defined elsewhere and is an Effect
const webConfig = Effect.succeed({"dbConnection": "localhost", "port": 8080});

const checkDatabaseConnectivity = Effect.promise(() => 
  Promise.resolve("Connected to Database")
);

const startupChecks = Effect.all([webConfig, checkDatabaseConnectivity]);

Effect.runPromise(startupChecks).then(([config, dbStatus]) => {
  console.log(
    `Configuration: ${JSON.stringify(config)}\nDB Status: ${dbStatus}`
  );
});
```

---

### Configure Logging Level and Test with Mock ConfigProvider

Source: https://effect.website/docs/observability/logging

Demonstrates how to set a minimum log level and provide a mock configuration using ConfigProvider.fromMap for testing purposes. The output shows log messages filtered by the specified log level.

```typescript
const LogLevelLive = Config.logLevel("LOG_LEVEL").pipe(
  Effect.andThen((level) =>
    // Set the minimum log level
    Logger.minimumLogLevel(level)
  ),
  Layer.unwrapEffect // Convert the effect into a layer
)

// Provide the loaded log level to the program
const configured = Effect.provide(program, LogLevelLive)

// Test the program using a mock configuration provider
const test = Effect.provide(
  configured,
  Layer.setConfigProvider(
    ConfigProvider.fromMap(
      new Map([["LOG_LEVEL", LogLevel.Warning.label]])
    )
  )
)

Effect.runFork(test)
/*
Output:
... level=ERROR fiber=#0 message=ERROR!
... level=WARN fiber=#0 message=WARNING!
*/
```

---

### Mocking the File System

Source: https://effect.website/docs/platform/file-system

Demonstrates how to mock the FileSystem service in testing environments using `FileSystem.layerNoop` for isolated testing.

```APIDOC
## Mocking the File System

### Description
In testing environments, you may want to mock the file system to avoid performing actual disk operations. The `FileSystem.layerNoop` provides a no-operation implementation of the `FileSystem` service. Most operations in `FileSystem.layerNoop` return a **failure** (e.g., `Effect.fail` for missing files) or a **defect** (e.g., `Effect.die` for unimplemented features). However, you can override specific behaviors by passing an object to `FileSystem.layerNoop` to define custom return values for selected methods.

### Example (Mocking File System with Custom Behavior)

```typescript
import { FileSystem } from "@effect/platform"
import { Effect } from "effect"

const program = Effect.gen(function* () {
  const fs = yield* FileSystem.FileSystem

  const exists = yield* fs.exists("/some/path")
  console.log(exists)

  const content = yield* fs.readFileString("/some/path")
  console.log(content)
})

// ┌─── Layer
// ▼
const customMock = FileSystem.layerNoop({
  readFileString: () => Effect.succeed("mocked content"),
  exists: (path) => Effect.succeed(path === "/some/path")
})

// Provide the customized FileSystem mock implementation
Effect.runPromise(program.pipe(Effect.provide(customMock)))

/* 
Output:
true 
mocked content 
*/
```
```

---

### Testing Database Service with Leaked Dependencies in Effect.js

Source: https://effect.website/docs/requirements-management/layers

This snippet illustrates a problematic test setup in Effect.js where the Database service interface directly requires Config and Logger. This forces the test to provide these unnecessary dependencies, complicating isolation.

```typescript
import { Effect, Context } from "effect"
import * as assert from "node:assert"

// Assuming Config, Logger, and Database tags and DatabaseTest instance are declared
class Config extends Context.Tag("Config")() {}
class Logger extends Context.Tag("Logger")() {}
class Database extends Context.Tag("Database")<Database, {
  readonly query: (sql: string) => Effect.Effect
}>() {}
const DatabaseTest = Database.of({
  query: (sql: string) => Effect.succeed([])
})

// A test that uses the Database service
const test = Effect.gen(function* () {
  const database = yield* Database
  const result = yield* database.query("SELECT * FROM users")
  assert.deepStrictEqual(result, [])
})

// Attempt to provide only the Database service without Config and Logger
const incompleteTestSetup = test.pipe(
  Effect.provideService(Database, DatabaseTest)
)
```

---

### Mocking Configuration Provider for Tests (Effect-TS)

Source: https://effect.website/docs/configuration

Demonstrates mocking a configuration provider using ConfigProvider.fromMap for testing. Allows running Effect programs with predefined configuration values instead of environment variables.

```typescript
import { Config, ConfigProvider, Effect } from "effect"

class HostPort {
  constructor(readonly host: string, readonly port: number) {}
  get url() {
    return `${this.host}:${this.port}`
  }
}

const config = Config.map(
  Config.all([Config.string("HOST"), Config.number("PORT")]),
  ([host, port]) => new HostPort(host, port)
)

const program = Effect.gen(function* () {
  const hostPort = yield* config
  console.log(`Application started: ${hostPort.url}`)
})

// Create a mock config provider using a map with test data
const mockConfigProvider = ConfigProvider.fromMap(
  new Map([
    ["HOST", "localhost"],
    ["PORT", "8080"]
  ])
)

// Run the program using the mock config provider
Effect.runPromise(Effect.withConfigProvider(program, mockConfigProvider))
```

---

### Mock File System with Custom Behavior using Effect Platform

Source: https://effect.website/docs/platform/file-system

Illustrates how to mock the FileSystem service in testing environments using FileSystem.layerNoop. This allows for simulating file system interactions without actual disk I/O. Custom return values for methods like readFileString and exists can be provided to the layerNoop function.

```typescript
import { FileSystem } from "@effect/platform"
import { Effect } from "effect"

const program = Effect.gen(function* () {
  const fs = yield* FileSystem.FileSystem

  const exists = yield* fs.exists("/some/path")
  console.log(exists)

  const content = yield* fs.readFileString("/some/path")
  console.log(content)
})

const customMock = FileSystem.layerNoop({
  readFileString: () => Effect.succeed("mocked content"),
  exists: (path) => Effect.succeed(path === "/some/path")
})

Effect.runPromise(program.pipe(Effect.provide(customMock)))
```

---

### Providing a Mock Service Implementation with Effect.Service

Source: https://effect.website/docs/requirements-management/layers

Illustrates how to provide a custom mock implementation for a service defined with Effect.Service. This is useful for testing or scenarios where a default implementation is not suitable. The mock object must conform to the service's interface.

```typescript
const mock = new MyService({
  /* mocked methods */
})
program.pipe(Effect.provideService(MyService, mock))
```

---

### Test Effect with ElasticSearch Failure

Source: https://effect.website/docs/resource-management/scope

This example illustrates testing an effect with a simulated 'ElasticSearch' failure during index creation. The FailureCase service is set to 'ElasticSearch'. The output shows S3 bucket creation, followed by an ElasticSearch index creation failure and subsequent rollback of the S3 bucket.

```typescript
const runnable = make.pipe(
  Effect.provide(layer),
  Effect.provideService(FailureCase, "ElasticSearch")
)
```

---

### Injecting Test Dependencies with Effect.Service

Source: https://effect.website/docs/requirements-management/layers

Illustrates how to use the Cache.DefaultWithoutDependencies layer with a mock FileSystem to test service logic without interacting with the actual file system. This is crucial for unit testing Effect applications.

```typescript
import { FileSystem } from "@effect/platform"
import { NodeFileSystem } from "@effect/platform-node"
import { Effect, Console } from "effect"

// Define a Cache service
class Cache extends Effect.Service()("app/Cache", {
  effect: Effect.gen(function* () {
    const fs = yield* FileSystem.FileSystem
    const lookup = (key: string) => fs.readFileString(`cache/${key}`)
    return { lookup } as const
  }),
  dependencies: [NodeFileSystem.layer]
}) {}

// Example of using a test file system (requires a mock implementation of FileSystem)
// const testFileSystemLayer = FileSystem.layer.use(() => ({ ...mockFileSystemImplementation }))
// const runnable = program.pipe(Effect.provide(Cache.DefaultWithoutDependencies, testFileSystemLayer))
```

---

### Handling Nested Configuration Values with Mocking (Effect-TS)

Source: https://effect.website/docs/configuration

Shows how to provide nested configuration values using ConfigProvider.fromMap with a dot notation separator. Useful for testing applications with nested configurations.

```typescript
import { Config, ConfigProvider, Effect } from "effect"

const config = Config.nested(Config.number("PORT"), "SERVER")

const program = Effect.gen(function* () {
  const port = yield* config
  console.log(`Server is running on port ${port}`)
})

// Mock configuration using '.' as the separator for nested keys
const mockConfigProvider = ConfigProvider.fromMap(
  new Map([["SERVER.PORT", "8080"]])
)

Effect.runPromise(Effect.withConfigProvider(program, mockConfigProvider))
```
