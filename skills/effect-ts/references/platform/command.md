# Effect Platform – Command

Shell command execution, stdin/stdout streaming, exit codes, and environment variables.

---

### Run a Command and Print Output as String

Source: https://effect.website/docs/platform/command

This example shows how to run a command and capture its output as a string using `Command.string`. It requires a `CommandExecutor` (provided by `@effect/platform-node/NodeContext`) and uses `NodeRuntime.runMain` to execute the Effect program. The output is then logged to the console.

```typescript
import { Command } from "@effect/platform"
import { NodeContext, NodeRuntime } from "@effect/platform-node"
import { Effect } from "effect"

const command = Command.make("ls", "-al")

// The program depends on a CommandExecutor
const program = Effect.gen(function* () {
  // Runs the command returning the output as a string
  const output = yield* Command.string(command)
  console.log(output)
})

// Provide the necessary CommandExecutor
NodeRuntime.runMain(program.pipe(Effect.provide(NodeContext.layer)))
```

---

### Feed Input to Command Standard Input

Source: https://effect.website/docs/platform/command

Illustrates how to provide data to a command's standard input using `Command.feed`. The example uses `cat` and feeds it the string "Hello", which is then echoed back as output.

```typescript
import { Command } from "@effect/platform"
import { NodeContext, NodeRuntime } from "@effect/platform-node"
import { Effect } from "effect"

const command = Command.make("cat").pipe(Command.feed("Hello"))

const program = Effect.gen(function* () {
  console.log(yield* Command.string(command))
})

NodeRuntime.runMain(program.pipe(Effect.provide(NodeContext.layer)))
```

---

### Access Process Details (Exit Code, Stdout, Stderr)

Source: https://effect.website/docs/platform/command

Demonstrates how to start a command using `Command.start` and then access its `exitCode`, `stdout`, and `stderr` streams. The `stdout` and `stderr` streams are collected into strings for display.

```typescript
import { Command } from "@effect/platform"
import { NodeContext, NodeRuntime } from "@effect/platform-node"
import { Effect, Stream, String, pipe } from "effect"

// Helper function to collect stream output as a string
const runString = (
  stream: Stream.Stream
):
  Effect.Effect =>
  stream.pipe(
    Stream.decodeText(),
    Stream.runFold(String.empty, String.concat)
  )

const program = Effect.gen(function* () {
  const command = Command.make("ls")

  const [exitCode, stdout, stderr] = yield* pipe(
    // Start running the command and return a handle to the running process
    Command.start(command),
    Effect.flatMap((process) =>
      Effect.all (
        [
          // Waits for the process to exit and returns
          // the ExitCode of the command that was run
          process.exitCode,
          // The standard output stream of the process
          runString(process.stdout),
          // The standard error stream of the process
          runString(process.stderr)
        ],
        { concurrency: 3 }
      )
    )
  )
  console.log({ exitCode, stdout, stderr })
})

NodeRuntime.runMain(
  Effect.scoped(program).pipe(Effect.provide(NodeContext.layer))
)
```

---

### Platform: Command Execution

Source: https://context7_llms

Learn how to create, run, and manage command-line commands within Effect applications, including handling arguments, environment variables, and I/O.

```typescript
import * as Command from "@effect/platform/Command"
import * as Effect from "@effect/io/Effect"

const listDirCommand = Command.make("ls", ["-l"])

const runCommand = Command.run(listDirCommand)

Effect.runSync(runCommand)
// Executes the 'ls -l' command and streams its output.
```

---

### Writing to Standard Output with Effect

Source: https://effect.website/docs/platform/terminal

Shows how to display a message to the terminal using the `display` method of the `Terminal` service. This example utilizes `Effect.provide` with `NodeTerminal.layer` to run the program in a Node.js environment.

```typescript
import { Terminal } from "@effect/platform"
import { NodeRuntime, NodeTerminal } from "@effect/platform-node"
import { Effect } from "effect"

const program = Effect.gen(function* () {
  const terminal = yield* Terminal.Terminal
  yield* terminal.display("a message\n")
})

NodeRuntime.runMain(program.pipe(Effect.provide(NodeTerminal.layer)))
// Output: "a message"
```

---

### Reading from Standard Input with Effect

Source: https://effect.website/docs/platform/terminal

Illustrates how to read a line of input from the user via standard input using the `readLine` method of the `Terminal` service. The example logs the received input to the console and is configured to run in a Node.js environment.

```typescript
import { Terminal } from "@effect/platform"
import { NodeRuntime, NodeTerminal } from "@effect/platform-node"
import { Effect } from "effect"

const program = Effect.gen(function* () {
  const terminal = yield* Terminal.Terminal
  const input = yield* terminal.readLine
  console.log(`input: ${input}`)
})

NodeRuntime.runMain(program.pipe(Effect.provide(NodeTerminal.layer)))
// Input: "hello"
// Output: "input: hello"
```

---

### Get Command Exit Code with Effect Platform

Source: https://effect.website/docs/platform/command

Demonstrates how to execute a command and retrieve only its exit code using `Command.exitCode`. This is useful when you only need to know if a command succeeded or failed without processing its output.

```typescript
import { Command } from "@effect/platform"
import { NodeContext, NodeRuntime } from "@effect/platform-node"
import { Effect } from "effect"

const command = Command.make("ls", "-al")

const program = Effect.gen(function* () {
  const exitCode = yield* Command.exitCode(command)
  console.log(exitCode)
})

NodeRuntime.runMain(program.pipe(Effect.provide(NodeContext.layer)))
```

---

### Define a Command for Directory Listing

Source: https://effect.website/docs/platform/command

This snippet demonstrates how to create a command object using `Command.make`. The command object represents the process name and its arguments but does not execute the command. It includes properties like command, args, env, cwd, shell, gid, and uid.

```typescript
import { Command } from "@effect/platform"

const command = Command.make("ls", "-al")
console.log(command)
/*
{
_id: '@effect/platform/Command',
_tag: 'StandardCommand',
command: 'ls',
args: [ '-al' ],
env: {},
cwd: { _id: 'Option', _tag: 'None' },
shell: false,
gid: { _id: 'Option', _tag: 'None' },
uid: { _id: 'Option', _tag: 'None' }
}
*/
```
