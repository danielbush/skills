# Effect Platform – FileSystem & Path

Reading/writing files, mocking the file system, and cross-platform path handling.

---

### Basic Cross-Platform Path Handling with Effect Platform

Source: https://effect.website/docs/platform/introduction

A fundamental example using Effect Platform's Path module to join path segments. This code is designed to run across different JavaScript runtimes without modification.

```typescript
1 import { Path } from "@effect/platform"
2 import { Effect } from "effect"
3 
4 const program = Effect.gen(function* () {
5 // Access the Path service
6 const path = yield* Path.Path
7 
8 // Join parts of a path to create a complete file path
9 const mypath = path.join("tmp", "file.txt")
10 
11 console.log(mypath)
12 })

```

---

### Read File Content as String using Effect Platform

Source: https://effect.website/docs/platform/file-system

Demonstrates how to read the content of a file as a string using the FileSystem service from @effect/platform. This example uses Effect.gen for asynchronous operations and NodeRuntime for execution in a Node.js environment. It requires importing FileSystem, NodeContext, NodeRuntime, and Effect.

```typescript
import { FileSystem } from "@effect/platform"
import { NodeContext, NodeRuntime } from "@effect/platform-node"
import { Effect } from "effect"

const program = Effect.gen(function* () {
  const fs = yield* FileSystem.FileSystem

  const content = yield* fs.readFileString("./index.ts", "utf8")
  console.log(content)
})

NodeRuntime.runMain(program.pipe(Effect.provide(NodeContext.layer)))
```

---

### Joining Path Segments with @effect/platform/Path

Source: https://effect.website/docs/platform/path

Shows an example of using the `join` method from the Path service to combine multiple path segments into a single path string. It utilizes Node.js specific runtime context.

```typescript
import { Path } from "@effect/platform"
import { Effect } from "effect"
import { NodeContext, NodeRuntime } from "@effect/platform-node"

const program = Effect.gen(function* () {
  const path = yield* Path.Path

  const mypath = path.join("tmp", "file.txt")
  console.log(mypath)
})

NodeRuntime.runMain(program.pipe(Effect.provide(NodeContext.layer)))
// Output: "tmp/file.txt"
```

---

### Accessing File System Operations with Effect

Source: https://effect.website/docs/platform/file-system

Demonstrates how to access the FileSystem service within an Effect program. This pattern is used to obtain an instance of the FileSystem interface to perform subsequent file operations. It requires the 'effect' and '@effect/platform' libraries.

```typescript
import { FileSystem } from "@effect/platform"
import { Effect } from "effect"

const program = Effect.gen(function* () {
  const fs = yield* FileSystem.FileSystem

  // Use `fs` to perform file system operations
})
```

---

### Platform: File System Operations

Source: https://context7_llms

Explore Effect's FileSystem module for performing various file operations such as reading, writing, and managing files and directories in a type-safe manner.

```typescript
import * as FileSystem from "@effect/platform/FileSystem"
import * as Effect from "@effect/io/Effect"

const readFileEffect = FileSystem.readFile("./myfile.txt")

Effect.runSync(readFileEffect)
// Reads the content of './myfile.txt'.
```

---

### FileSystem Operations

Source: https://effect.website/docs/platform/file-system

This section details various file system operations available through the Effect Platform's FileSystem API.

```APIDOC
## FileSystem API Operations

### Description
Provides a collection of functions to interact with the file system, including reading, writing, removing, and managing files and directories.

### Core Operations

*   **copy** (fromPath, toPath, options?): Copies a file or directory.
*   **createDir** (path): Creates a directory.
*   **deleteDir** (path, recursive?): Deletes a directory. If `recursive` is set to `true`, it will recursively delete nested directories.
*   **exists** (path): Checks if a file or directory exists at the specified path.
*   **glob** (pattern, options?): Returns a list of files matching the glob pattern.
*   **list** (directory, recursive?): Lists the contents of a directory. Recursively lists the contents of nested directories by setting the `recursive` option.
*   **mkdir** (path): Creates a directory.
*   **move** (fromPath, toPath): Moves a file or directory.
*   **read** (path): Reads the contents of a file.
*   **readFile** (path, encoding?): Reads the contents of a file.
*   **readFileString** (path, encoding?): Reads the contents of a file as a string.
*   **readLink** (path): Reads the destination of a symbolic link.
*   **realPath** (path): Resolves a path to its canonicalized absolute pathname.
*   **remove** (path, recursive?): Removes a file or directory. By setting the `recursive` option to `true`, you can recursively remove nested directories.
*   **rename** (fromPath, toPath): Renames a file or directory.
*   **sink** (path): Creates a writable `Sink` for the specified `path`.
*   **stat** (path): Gets information about a file at `path`.
*   **stream** (path): Creates a readable `Stream` for the specified `path`.
*   **symlink** (fromPath, toPath): Creates a symbolic link from `fromPath` to `toPath`.
*   **truncate** (path, length?): Truncates a file to a specified length. If the `length` is not specified, the file will be truncated to length `0`.
*   **utimes** (path, atime, mtime): Changes the file system timestamps of the file at `path`.
*   **watch** (path): Watches a directory or file for changes.
*   **writeFile** (path, data, encoding?): Writes data to a file at `path`.
*   **writeFileString** (path, data, encoding?): Writes a string to a file at `path`.
```
