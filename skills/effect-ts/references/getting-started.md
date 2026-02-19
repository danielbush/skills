# Effect – Getting Started

Installation, project scaffolding, tooling setup, and running programs.

---

### Create Effect App using Bun

Source: https://effect.website/docs/getting-started/create-effect-app

Initiates a new Effect application using the create-effect-app CLI via Bun. This command starts an interactive setup process to guide the user through project bootstrapping.

```bash
bunx create-effect-app@latest
```

---

### Create Effect App using Deno

Source: https://effect.website/docs/getting-started/create-effect-app

Initiates a new Effect application using the create-effect-app CLI via Deno. This command starts an interactive setup process to guide the user through project bootstrapping.

```bash
deno init --npm effect-app@latest
```

---

### Install Effect AI and OpenAI Provider Packages (bun)

Source: https://effect.website/docs/ai/getting-started

Installs the core @effect/ai package for AI abstractions and the @effect/ai-openai package for OpenAI integration, along with the core Effect package using bun. This is the initial setup for using Effect AI with OpenAI.

```bash
# Install the base package for the core abstractions (always required)
bun add @effect/ai

# Install one (or more) provider integrations
bun add @effect/ai-openai

# Also add the core Effect package (if not already installed)
bun add effect
```

---

### Create Effect App using npm

Source: https://effect.website/docs/getting-started/create-effect-app

Initiates a new Effect application using the create-effect-app CLI via npm. This command starts an interactive setup process to guide the user through project bootstrapping.

```bash
npx create-effect-app@latest
```

---

### Install Effect AI and OpenAI Provider Packages (npm)

Source: https://effect.website/docs/ai/getting-started

Installs the core @effect/ai package for AI abstractions and the @effect/ai-openai package for OpenAI integration, along with the core Effect package. This is the initial setup for using Effect AI with OpenAI.

```bash
# Install the base package for the core abstractions (always required)
npm install @effect/ai

# Install one (or more) provider integrations
npm install @effect/ai-openai

# Also add the core Effect package (if not already installed)
npm install effect
```

---

### Install Effect AI and OpenAI Provider Packages (pnpm)

Source: https://effect.website/docs/ai/getting-started

Installs the core @effect/ai package for AI abstractions and the @effect/ai-openai package for OpenAI integration, along with the core Effect package using pnpm. This is the initial setup for using Effect AI with OpenAI.

```bash
# Install the base package for the core abstractions (always required)
pnpm add @effect/ai

# Install one (or more) provider integrations
pnpm add @effect/ai-openai

# Also add the core Effect package (if not already installed)
pnpm add effect
```

---

### Install OpenTelemetry Dependencies (Bun)

Source: https://effect.website/docs/observability/tracing

Installs the core library for integrating OpenTelemetry with Effect, along with the necessary OpenTelemetry SDKs for tracing and metrics using Bun.

```bash
# Install the main library for integrating OpenTelemetry with Effect
bun add @effect/opentelemetry

# Install the required OpenTelemetry SDKs for tracing and metrics
bun add @opentelemetry/sdk-trace-base
bun add @opentelemetry/sdk-trace-node
bun add @opentelemetry/sdk-trace-web
bun add @opentelemetry/sdk-metrics
```

---

### Create Effect App using pnpm

Source: https://effect.website/docs/getting-started/create-effect-app

Initiates a new Effect application using the create-effect-app CLI via pnpm. This command starts an interactive setup process to guide the user through project bootstrapping.

```bash
pnpm create effect-app@latest
```

---

### Install Effect AI and OpenAI Provider Packages (yarn)

Source: https://effect.website/docs/ai/getting-started

Installs the core @effect/ai package for AI abstractions and the @effect/ai-openai package for OpenAI integration, along with the core Effect package using yarn. This is the initial setup for using Effect AI with OpenAI.

```bash
# Install the base package for the core abstractions (always required)
yarn add @effect/ai

# Install one (or more) provider integrations
yarn add @effect/ai-openai

# Also add the core Effect package (if not already installed)
yarn add effect
```

---

### Create Effect App using Yarn

Source: https://effect.website/docs/getting-started/create-effect-app

Initiates a new Effect application using the create-effect-app CLI via Yarn. This command starts an interactive setup process to guide the user through project bootstrapping.

```bash
yarn create effect-app@latest
```

---

### Install Effect Library using npm, pnpm, Yarn, Bun, or Deno

Source: https://effect.website/docs/micro/new-users

Commands to install the Effect library using various package managers. This is a prerequisite for using the Micro module.

```bash
npm install effect
```

```bash
pnpm add effect
```

```bash
yarn add effect
```

```bash
bun add effect
```

```bash
deno add npm:effect
```

---

### Setup EffectJS Project for Deno

Source: https://effect.website/docs/getting-started/installation

Initializes a new Deno project, adds the EffectJS package as a dependency, and creates a basic 'Hello, World!' program to verify the setup. The program is then executed using the Deno runtime.

```bash
mkdir hello-effect
cd hello-effect
deno init
deno add npm:effect
```

```typescript
1 import { Effect, Console } from "effect" 
2 
3 const program = Console.log("Hello, World!") 
4 
5 Effect.runSync(program)
```

```bash
deno run main.ts
```

---

### Development Server Start Commands (Shell)

Source: https://effect.website/docs/getting-started/installation

This section provides commands to start the development server for the project using different package managers. These commands are essential for running the application locally and testing changes. Supported package managers include npm, pnpm, Yarn, Bun, and Deno.

```bash
npm run dev
```

```bash
pnpm run dev
```

```bash
yarn run dev
```

```bash
bun run dev
```

```bash
deno run dev
```

---

### Install Effect and OpenTelemetry Dependencies with Bun

Source: https://effect.website/docs/observability/tracing

Installs the Effect library, the Effect-OpenTelemetry integration, and necessary OpenTelemetry packages for trace export via HTTP (OTLP) and Node.js/web SDKs using Bun.

```bash
# If not already installed
bun add effect
# Required to integrate Effect with OpenTelemetry
bun add @effect/opentelemetry
# Required to export traces over HTTP in OTLP format
bun add @opentelemetry/exporter-trace-otlp-http
# Required by all applications
bun add @opentelemetry/sdk-trace-base
# For NodeJS applications
bun add @opentelemetry/sdk-trace-node
# For browser applications
bun add @opentelemetry/sdk-trace-web
# If you also need to export metrics
bun add @opentelemetry/sdk-metrics
```

---

### Install Effect Package (npm)

Source: https://context7_llms

Instructions for installing the Effect TypeScript package using npm. This is the first step to start building type-safe applications with Effect.

```bash
npm install @effect/io
```

---

### Install OpenTelemetry Dependencies (npm)

Source: https://effect.website/docs/observability/tracing

Installs the core library for integrating OpenTelemetry with Effect, along with the necessary OpenTelemetry SDKs for tracing and metrics using npm.

```bash
# Install the main library for integrating OpenTelemetry with Effect
npm install @effect/opentelemetry

# Install the required OpenTelemetry SDKs for tracing and metrics
npm install @opentelemetry/sdk-trace-base
npm install @opentelemetry/sdk-trace-node
npm install @opentelemetry/sdk-trace-web
npm install @opentelemetry/sdk-metrics
```

---

### Install OpenTelemetry Dependencies (pnpm)

Source: https://effect.website/docs/observability/tracing

Installs the core library for integrating OpenTelemetry with Effect, along with the necessary OpenTelemetry SDKs for tracing and metrics using pnpm.

```bash
# Install the main library for integrating OpenTelemetry with Effect
pnpm add @effect/opentelemetry

# Install the required OpenTelemetry SDKs for tracing and metrics
pnpm add @opentelemetry/sdk-trace-base
pnpm add @opentelemetry/sdk-trace-node
pnpm add @opentelemetry/sdk-trace-web
pnpm add @opentelemetry/sdk-metrics
```

---

### Install Effect and OpenTelemetry Dependencies with npm

Source: https://effect.website/docs/observability/tracing

Installs the Effect library, the Effect-OpenTelemetry integration, and necessary OpenTelemetry packages for trace export via HTTP (OTLP) and Node.js/web SDKs using npm.

```bash
# If not already installed
npm install effect
# Required to integrate Effect with OpenTelemetry
npm install @effect/opentelemetry
# Required to export traces over HTTP in OTLP format
npm install @opentelemetry/exporter-trace-otlp-http
# Required by all applications
npm install @opentelemetry/sdk-trace-base
# For NodeJS applications
npm install @opentelemetry/sdk-trace-node
# For browser applications
npm install @opentelemetry/sdk-trace-web
# If you also need to export metrics
npm install @opentelemetry/sdk-metrics
```

---

### Install OpenTelemetry Dependencies (Yarn)

Source: https://effect.website/docs/observability/tracing

Installs the core library for integrating OpenTelemetry with Effect, along with the necessary OpenTelemetry SDKs for tracing and metrics using Yarn.

```bash
# Install the main library for integrating OpenTelemetry with Effect
yarn add @effect/opentelemetry

# Install the required OpenTelemetry SDKs for tracing and metrics
yarn add @opentelemetry/sdk-trace-base
yarn add @opentelemetry/sdk-trace-node
yarn add @opentelemetry/sdk-trace-web
yarn add @opentelemetry/sdk-metrics
```

---

### Install Effect Platform

Source: https://effect.website/docs/platform/introduction

Installs the core Effect Platform package, enabling cross-platform capabilities. This is the first step before using any Effect Platform modules.

```bash
yarn add @effect/platform
```

```bash
bun add @effect/platform
```

```bash
deno add npm:@effect/platform
```

---

### Non-Interactive Create Effect App from example

Source: https://effect.website/docs/getting-started/create-effect-app

Demonstrates non-interactive usage of create-effect-app to create a project directly from a public GitHub example, such as 'http-server'.

```bash
create-effect-app (-e, --example http-server)
```

---

### Setup EffectJS Project for Bun

Source: https://effect.website/docs/getting-started/installation

Initializes a new Bun project, configures TypeScript with strict mode, adds the EffectJS package, and creates a simple 'Hello, World!' program. The program is executed using the Bun runtime.

```bash
mkdir hello-effect
cd hello-effect
bun init
```

```json
{
  "compilerOptions": {
    "strict": true
  }
}
```

```bash
bun add effect
```

```typescript
1 import { Effect, Console } from "effect" 
2 
3 const program = Console.log("Hello, World!") 
4 
5 Effect.runSync(program)
```

```bash
bun index.ts
```

---

### Install Effect and OpenTelemetry Dependencies with pnpm

Source: https://effect.website/docs/observability/tracing

Installs the Effect library, the Effect-OpenTelemetry integration, and necessary OpenTelemetry packages for trace export via HTTP (OTLP) and Node.js/web SDKs using pnpm.

```bash
# If not already installed
pnpm add effect
# Required to integrate Effect with OpenTelemetry
pnpm add @effect/opentelemetry
# Required to export traces over HTTP in OTLP format
pnpm add @opentelemetry/exporter-trace-otlp-http
# Required by all applications
pnpm add @opentelemetry/sdk-trace-base
# For NodeJS applications
pnpm add @opentelemetry/sdk-trace-node
# For browser applications
pnpm add @opentelemetry/sdk-trace-web
# If you also need to export metrics
pnpm add @opentelemetry/sdk-metrics
```

---

### Install Effect and OpenTelemetry Dependencies with Yarn

Source: https://effect.website/docs/observability/tracing

Installs the Effect library, the Effect-OpenTelemetry integration, and necessary OpenTelemetry packages for trace export via HTTP (OTLP) and Node.js/web SDKs using Yarn.

```bash
# If not already installed
yarn add effect
# Required to integrate Effect with OpenTelemetry
yarn add @effect/opentelemetry
# Required to export traces over HTTP in OTLP format
yarn add @opentelemetry/exporter-trace-otlp-http
# Required by all applications
yarn add @opentelemetry/sdk-trace-base
# For NodeJS applications
yarn add @opentelemetry/sdk-trace-node
# For browser applications
yarn add @opentelemetry/sdk-trace-web
# If you also need to export metrics
yarn add @opentelemetry/sdk-metrics
```

---

### Install Effect Platform Bun Package

Source: https://effect.website/docs/platform/introduction

Installs the Bun-specific package for Effect Platform, enabling the use of Bun-specific services and contexts.

```bash
bun add @effect/platform-bun
```

---

### Start OpenTelemetry Backend with Docker

Source: https://effect.website/docs/observability/tracing

This command starts the OpenTelemetry backend using a preconfigured Grafana Docker image. It exposes ports for OTLP trace and metric collection, and the Grafana UI.

```bash
docker run -p 3000:3000 -p 4317:4317 -p 4318:4318 --rm -it docker.io/grafana/otel-lgtm
```

---

### Install Effect Package (yarn)

Source: https://context7_llms

Instructions for installing the Effect TypeScript package using yarn. This command ensures you have the necessary Effect modules for your project.

```bash
yarn add @effect/io
```

---

### Install Effect Platform Node.js Package

Source: https://effect.website/docs/platform/introduction

Installs the Node.js-specific package for Effect Platform, enabling the use of Node.js-specific services and contexts.

```bash
npm install @effect/platform-node
```

```bash
pnpm add @effect/platform-node
```

```bash
yarn add @effect/platform-node
```

```bash
deno add npm:@effect/platform-node
```

---

### Install Effect Package with Bun

Source: https://effect.website/docs/getting-started/importing-effect

Installs the 'effect' package using Bun. This command is suitable for projects using the Bun runtime and package manager.

```bash
bun add effect
```

---

### Install Effect Package with Yarn

Source: https://effect.website/docs/getting-started/importing-effect

Installs the 'effect' package using Yarn. Use this command if your project utilizes Yarn for package management.

```bash
yarn add effect
```

---

### Install EffectJS with npm/yarn/pnpm

Source: https://effect.website/docs/getting-started/installation

Installs the core EffectJS package using package managers like npm, pnpm, or yarn. This is the foundational step for any EffectJS project.

```bash
pnpm add effect
```

```bash
yarn add effect
```

```bash
npm install effect
```

---

### Install Effect Package with pnpm

Source: https://effect.website/docs/getting-started/importing-effect

Installs the 'effect' package using pnpm. This command is an alternative for projects managed with pnpm.

```bash
pnpm add effect
```

---

### Install @effect/experimental using npm, pnpm, or yarn

Source: https://effect.website/docs/getting-started/devtools

Commands to install the @effect/experimental package using different JavaScript package managers. This package is necessary for utilizing the experimental DevTools.

```bash
npm install @effect/experimental
```

```bash
pnpm install @effect/experimental
```

```bash
yarn add @effect/experimental
```

```bash
bun add @effect/experimental
```

---

### Install Effect Package with npm

Source: https://effect.website/docs/getting-started/importing-effect

Installs the 'effect' package using npm. This is the primary command for integrating the Effect library into your project.

```bash
npm install effect
```

---

### Install Effect Package with Deno

Source: https://effect.website/docs/getting-started/importing-effect

Installs the 'effect' package for Deno using npm compatibility. This command allows Deno projects to leverage npm packages.

```bash
deno add npm:effect
```

---

### Install Effect Language Service (Bun)

Source: https://effect.website/docs/getting-started/devtools

Installs the Effect Language Service as a development dependency using Bun. This command should be run in the terminal within your project directory.

```bash
bun add --dev @effect/language-service
```

---

### Scaffold Vite + React Project with EffectJS

Source: https://effect.website/docs/getting-started/installation

Creates a new Vite project with the React and TypeScript template. It then installs project dependencies and adds EffectJS as a library. The tsconfig.json is verified for strict mode.

```bash
# npm 6.x
npm create vite@latest hello-effect --template react-ts

# npm 7+, extra double-dash is needed
npm create vite@latest hello-effect -- --template react-ts

pnpm create vite@latest hello-effect -- --template react-ts

yarn create vite@latest hello-effect -- --template react-ts

bun create vite@latest hello-effect -- --template react-ts

denom init --npm vite@latest hello-effect -- --template react-ts
```

```bash
cd hello-effect
npm install

pnpm install

yarn install

bun install

denom install
```

```json
{
  "compilerOptions": {
    "strict": true
  }
}
```

```bash
npm install effect

pnpm add effect

yarn add effect

bun add effect

denom add effect
```

---

### Install Effect Language Service (pnpm)

Source: https://effect.website/docs/getting-started/devtools

Installs the Effect Language Service as a development dependency using pnpm. This command should be run in the terminal within your project directory.

```bash
pnpm add -D @effect/language-service
```

---

### Install Effect Language Service (npm)

Source: https://effect.website/docs/getting-started/devtools

Installs the Effect Language Service as a development dependency using npm. This command should be run in the terminal within your project directory.

```bash
npm install @effect/language-service --save-dev
```

---

### Create Effect App with specific template and ESLint

Source: https://effect.website/docs/getting-started/create-effect-app

Creates a new Effect project in a specified directory using a 'basic' template and enables ESLint integration for code quality. This is an example of non-interactive usage.

```bash
npx create-effect-app --template basic --eslint my-effect-app
```

---

### Install Effect Language Service (Yarn)

Source: https://effect.website/docs/getting-started/devtools

Installs the Effect Language Service as a development dependency using Yarn. This command should be run in the terminal within your project directory.

```bash
yarn add --dev @effect/language-service
```

---

### Effect TS: Automating TypeScript Patching with package.json

Source: https://effect.website/docs/getting-started/devtools

Illustrates how to automatically apply the TypeScript patch for Effect diagnostics during the package installation process by adding a 'prepare' script to package.json.

```json
{
  "scripts": {
    "prepare": "effect-language-service patch"
  }
}
```

---

### Run a Successful Program with runMain (Node.js)

Source: https://effect.website/docs/platform/runtime

This example demonstrates how to use `runMain` from `@effect/platform-node` to execute a successful Effect. `runMain` handles the execution and ensures proper finalization. No output is produced for a successful run.

```typescript
import { NodeRuntime } from "@effect/platform-node"
import { Effect } from "effect"

const success = Effect.succeed("Hello, World!")

NodeRuntime.runMain(success)
// No Output
```

---

### Non-Interactive Create Effect App with template and options

Source: https://effect.website/docs/getting-started/create-effect-app

Demonstrates non-interactive usage of create-effect-app, allowing users to specify templates (e.g., 'basic', 'cli', 'monorepo') and additional configurations like '--changesets', '--flake', '--eslint', and '--workflows'.

```bash
create-effect-app (-t, --template basic | cli | monorepo) [--changesets] [--flake] [--eslint] [--workflows]
```

---

### Run a Failing Program with runMain (Node.js)

Source: https://effect.website/docs/platform/runtime

This example shows how `runMain` handles a failing Effect. By default, it logs the error in a pretty format to the console, indicating the timestamp, error level, and the error message itself.

```typescript
import { NodeRuntime } from "@effect/platform-node"
import { Effect } from "effect"

const failure = Effect.fail("Uh oh!")

NodeRuntime.runMain(failure)
/* 
Output: 
[12:43:07.186] ERROR (#0):
Error: Uh oh!
*/
```

---

### Integrate Effect DevTools with NodeRuntime in JavaScript

Source: https://effect.website/docs/getting-started/devtools

Example demonstrating how to import and use the DevTools module in an Effect application with NodeRuntime. It shows logging, delaying, and applying spans within an Effect program, then providing the DevTools layer before running the main program.

```javascript
import { DevTools } from "@effect/experimental"
import { NodeRuntime, NodeSocket } from "@effect/platform-node"
import { Effect, Layer } from "effect"

const program = Effect.log("Hello!").pipe(
  Effect.delay(2000),
  Effect.withSpan("Hi", { attributes: { foo: "bar" } }),
  Effect.forever,
)

const DevToolsLive = DevTools.layer()

program.pipe(Effect.provide(DevToolsLive), NodeRuntime.runMain)
```

---

### Effect TS: Patching TypeScript for Build-Time Diagnostics

Source: https://effect.website/docs/getting-started/devtools

Shows how to patch the local TypeScript installation to enable Effect diagnostics during the build process. This ensures that type-aware linting catches issues by performing type checking.

```bash
effect-language-service patch
```

---

### Run Program with In-Memory KeyValueStore

Source: https://effect.website/docs/platform/key-value-store

Demonstrates executing an Effect program that utilizes the in-memory KeyValueStore implementation. This is useful for testing or lightweight applications.

```typescript
import { Effect, layerMemory } from "@effect/platform/KeyValueStore"

const program = Effect.gen(function* () {
  const kv = yield* KeyValueStore
  yield* kv.set("key", "value")
  console.log(yield* kv.size)
  console.log(yield* kv.get("key"))
})

Effect.runPromise(program.pipe(Effect.provide(layerMemory)))
```

---

### Run Bun Program

Source: https://effect.website/docs/platform/introduction

Executes a JavaScript or TypeScript file directly using the Bun runtime. Bun is known for its speed and built-in tooling.

```bash
bun index.ts
```

---

### Run a Failing Program without Pretty Logger (Node.js)

Source: https://effect.website/docs/platform/runtime

This example demonstrates disabling the default pretty logger when running a failing Effect with `runMain`. The error will still be logged, but in a more basic format, useful for environments where detailed formatting is not desired.

```typescript
import { NodeRuntime } from "@effect/platform-node"
import { Effect } from "effect"

const failure = Effect.fail("Uh oh!")

NodeRuntime.runMain(failure, {
  disablePrettyLogger: true
})
```

---

### Run Node.js Program with Deno

Source: https://effect.website/docs/platform/introduction

Executes a TypeScript file directly using Deno. Deno can run Node.js-compatible code by importing npm packages, requiring specific permission flags for certain operations.

```bash
deno run index.ts
```

```bash
deno run -RE index.ts
```

---

### Run Failing Program with Pretty Logger Disabled (TypeScript)

Source: https://effect.website/docs/platform/runtime

This example demonstrates running a failing Effect.js program with the pretty logger disabled. It uses NodeRuntime to execute an Effect that fails with an error message. The output will include timestamp and level information but with disabled pretty logging.

```typescript
import { NodeRuntime } from "@effect/platform-node"
import { Effect } from "effect"

const failure = Effect.fail("Uh oh!")

NodeRuntime.runMain(failure, { disablePrettyLogger: true }) 
/* 
Output:
timestamp=2025-01-14T11:43:46.276Z level=ERROR fiber=#0 cause="Error: Uh oh!"
*/
```

---

### Run Failing Program Without Error Reporting (TypeScript)

Source: https://effect.website/docs/platform/runtime

This example shows how to run a failing Effect.js program while explicitly disabling error reporting. It uses NodeRuntime to execute a failing Effect. When error reporting is disabled, no output related to the error is produced.

```typescript
import { NodeRuntime } from "@effect/platform-node"
import { Effect } from "effect"

const failure = Effect.fail("Uh oh!")

NodeRuntime.runMain(failure, { disableErrorReporting: true })
// No Output
```

---

### Run Failing Program with Custom Teardown (TypeScript)

Source: https://effect.website/docs/platform/runtime

This example illustrates running a failing Effect.js program with a custom teardown function. The teardown logic checks the exit status of the program and logs a specific message if the program ended with an error. It then calls the provided `onExit` callback with an appropriate exit code.

```typescript
import { NodeRuntime } from "@effect/platform-node"
import { Effect } from "effect"

const failure = Effect.fail("Uh oh!")

NodeRuntime.runMain(failure, {
  teardown: function customTeardown(exit, onExit) {
    if (exit._tag === "Failure") {
      console.error("Program ended with an error.")
      onExit(1)
    } else {
      console.log("Program finished successfully.")
      onExit(0)
    }
  }
})
/* 
Output:
[12:46:39.871] ERROR (#0):
Error: Uh oh!
Program ended with an error.
*/
```
