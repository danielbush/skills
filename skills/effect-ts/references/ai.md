# Effect – AI Integration

`@effect/ai`, OpenAI and Anthropic providers, `LanguageModel` service.

---

### Creating a Client Layer for OpenAI Provider

Source: https://effect.website/docs/ai/getting-started

Shows how to create a Layer that provides an OpenAiClient. This layer requires an HttpClient and configuration (like an API key) to initialize the client, making the program executable.

```typescript
import { OpenAiClient, OpenAiLanguageModel } from "@effect/ai-openai"
import { LanguageModel } from "@effect/ai"
import { Config, Effect } from "effect"

const generateDadJoke = Effect.gen(function* () {
  const response = yield* LanguageModel.generateText({
    prompt: "Generate a dad joke"
  })
  console.log(response.text)
  return response
})

const Gpt4o = OpenAiLanguageModel.model("gpt-4o")

const main = generateDadJoke.pipe(
  Effect.provide(Gpt4o)
)

const OpenAi = OpenAiClient.layerConfig({
  apiKey: Config.redacted("OPENAI_API_KEY")
})
```

---

### Provide OpenAI Model to LanguageModel Service (Effect)

Source: https://effect.website/docs/ai/getting-started

Shows how to satisfy the `LanguageModel` requirement by providing a concrete implementation using `OpenAiLanguageModel` from `@effect/ai-openai`. This example defines a GPT-4o model and then pipes it into the `generateDadJoke` program using `Effect.provide`.

```typescript
import { OpenAiLanguageModel } from "@effect/ai-openai"
import { LanguageModel } from "@effect/ai"
import { Effect } from "effect"

const generateDadJoke = Effect.gen(function*() {
  const response = yield* LanguageModel.generateText({
    prompt: "Generate a dad joke"
  })
  console.log(response.text)
  return response
})

// Create a `Model` which provides a concrete implementation of
// `LanguageModel` and requires an `OpenAiClient`
const Gpt4o = OpenAiLanguageModel.model("gpt-4o")

// Provide the `Model` to the program
const main = generateDadJoke.pipe(
  Effect.provide(Gpt4o)
)
```

---

### Provide Model to Effect Program (Effect)

Source: https://effect.website/docs/ai/getting-started

Demonstrates how to provide a created `Model` (in this case, `Gpt4o` for OpenAI) directly to an Effect program using `Effect.provide`. This makes the services provided by the model available to the program.

```typescript
import { OpenAiLanguageModel } from "@effect/ai-openai"
import { LanguageModel } from "@effect/ai"
import { Effect } from "effect"

// ┌─── Model<"openai", LanguageModel | ProviderName, OpenAiClient>
// ▼
const Gpt4o = OpenAiLanguageModel.model("gpt-4o")

// ┌─── Effect, AiError, OpenAiClient>
// ▼
const program = LanguageModel.generateText({
  prompt: "Generate a dad joke"
}).pipe(Effect.provide(Gpt4o))
```

---

### Provide Model to Multiple Programs (Effect)

Source: https://effect.website/docs/ai/getting-started

Shows the reusability of a `Model` by providing it to multiple Effect programs. In this example, the `Gpt4o` model is provided once to a parent effect that executes `generateDadJoke` multiple times, demonstrating flexibility.

```typescript
import { OpenAiLanguageModel } from "@effect/ai-openai"
import { LanguageModel } from "@effect/ai"
import { Effect } from "effect"

const generateDadJoke = Effect.gen(function*() {
  const response = yield* LanguageModel.generateText({
    prompt: "Generate a dad joke"
  })
  console.log(response.text)
  return response
})

const Gpt4o = OpenAiLanguageModel.model("gpt-4o")

const main = Effect.gen(function*() {
  // You can provide the `Model` individually to each
  // program, or to all of them at once (as we do here)
  const res1 = yield* generateDadJoke
  const res2 = yield* generateDadJoke
  const res3 = yield* generateDadJoke
}).pipe(Effect.provide(Gpt4o))
```

---

### Abstracting LLM Interactions into a Service

Source: https://effect.website/docs/ai/getting-started

Illustrates how to abstract Language Model dependencies into a service, allowing consuming programs to depend only on the service interface, not the underlying AI requirements. This is achieved using Effect's Layer composition.

```typescript
import { AnthropicLanguageModel } from "@effect/ai-anthropic"
import { OpenAiLanguageModel } from "@effect/ai-openai"
import { LanguageModel } from "@effect/ai"
import { Effect } from "effect"

const Gpt4o = OpenAiLanguageModel.model("gpt-4o")
const Claude37 = AnthropicLanguageModel.model("claude-3-7-sonnet-latest")

class DadJokes extends Effect.Service()("app/DadJokes", {
  effect: Effect.gen(function* () {
    const gpt = yield* Gpt4o
    const claude = yield* Claude37

    const generateDadJoke = Effect.gen(function* () {
      const response = yield* LanguageModel.generateText({
        prompt: "Generate a dad joke"
      })
      console.log(response.text)
      return response
    })

    return {
      generateDadJoke: Effect.provide(generateDadJoke, gpt),
      generateBetterDadJoke: Effect.provide(generateDadJoke, claude)
    }
  })
}) {}

const main = Effect.gen(function* () {
  const dadJokes = yield* DadJokes
  const res1 = yield* dadJokes.generateDadJoke
  const res2 = yield* dadJokes.generateBetterDadJoke
})

DadJokes.Default
```

---

### Mixing Providers and Models with Effect. AI

Source: https://effect.website/docs/ai/getting-started

Demonstrates how to combine different language models from various providers (Anthropic, OpenAI) within a single Effect program. It shows how Effect's type-level dependency tracking helps manage these integrations.

```typescript
import { AnthropicLanguageModel } from "@effect/ai-anthropic"
import { OpenAiLanguageModel } from "@effect/ai-openai"
import { LanguageModel } from "@effect/ai"
import { Effect } from "effect"

const generateDadJoke = Effect.gen(function* () {
  const response = yield* LanguageModel.generateText({
    prompt: "Generate a dad joke"
  })
  console.log(response.text)
  return response
})

const Gpt4o = OpenAiLanguageModel.model("gpt-4o")
const Claude37 = AnthropicLanguageModel.model("claude-3-7-sonnet-latest")

const main = Effect.gen(function* () {
  const res1 = yield* generateDadJoke
  const res2 = yield* generateDadJoke
  const res3 = yield* Effect.provide(generateDadJoke, Claude37)
}).pipe(Effect.provide(Gpt4o))
```

---

### Define OpenAI Model for LanguageModel Service (Effect)

Source: https://effect.website/docs/ai/getting-started

Illustrates the creation of a specific `Model` instance for interacting with OpenAI's GPT-4o model. This `Model` provides the `LanguageModel` service and requires an `OpenAiClient`.

```typescript
import { OpenAiLanguageModel } from "@effect/ai-openai"

// ┌─── Model<"openai", LanguageModel | ProviderName, OpenAiClient>
// ▼
const Gpt4o = OpenAiLanguageModel.model("gpt-4o")
```

---

### Configure OpenAI Client Layer with EffectJS

Source: https://effect.website/docs/ai/getting-started

This snippet demonstrates how to create a configurable `OpenAiClient` layer using EffectJS. It utilizes `layerConfig` to read the API key from configuration, ensuring it's handled securely. This layer can then be provided to other effects.

```typescript
import { OpenAiClient } from "@effect/ai-openai"
import { Config, Layer } from "effect"

// Create a `Layer` which produces an `OpenAiClient` and requires
// an `HttpClient`
const OpenAi = OpenAiClient.layerConfig({
  apiKey: Config.redacted("OPENAI_API_KEY")
})
```

---

### Provide NodeHttpClient to OpenAI Layer in EffectJS

Source: https://effect.website/docs/ai/getting-started

This code illustrates how to provide a platform-specific `HttpClient` implementation to the `OpenAi` layer. It uses `Layer.provide` to integrate `NodeHttpClient.layerUndici`, making the OpenAI client ready for use in a Node.js environment.

```typescript
import { OpenAiClient } from "@effect/ai-openai"
import { NodeHttpClient } from "@effect/platform-node"
import { Layer } from "effect"

// Assume OpenAi layer is defined as above
const OpenAi = OpenAiClient.layerConfig({
  apiKey: Config.redacted("OPENAI_API_KEY")
})

// Provide a platform-specific implementation of `HttpClient` to our
// OpenAi layer
const OpenAiWithHttp = Layer.provide(OpenAi, NodeHttpClient.layerUndici)
```

---

### Generate Dad Joke with LanguageModel Service (Effect)

Source: https://effect.website/docs/ai/getting-started

Demonstrates generating text using the abstract `LanguageModel` service from `@effect/ai`. This code defines the business logic for generating a dad joke without specifying a particular LLM provider. It relies on the `LanguageModel` service being provided to the program.

```typescript
import { LanguageModel } from "@effect/ai"
import { Effect } from "effect"

// Using `LanguageModel` will add it to your program's requirements
const generateDadJoke = Effect.gen(function*() {
  // Use the `LanguageModel` to generate some text
  const response = yield* LanguageModel.generateText({
    prompt: "Generate a dad joke"
  })
  // Log the generated text to the console
  console.log(response.text)
  // Return the response
  return response
})
```

---

### Generate Dad Joke with OpenAI and EffectJS

Source: https://effect.website/docs/ai/getting-started

This code defines a function to generate a dad joke using OpenAI's language model through the EffectJS framework. It sets up the necessary layers and provides the specific model to use. The final program executes this generation and runs it as a promise.

```typescript
import { OpenAiClient, OpenAiLanguageModel } from "@effect/ai-openai"
import { LanguageModel } from "@effect/ai"
import { NodeHttpClient } from "@effect/platform-node"
import { Config, Effect, Layer } from "effect"

const generateDadJoke = Effect.gen(function*() {
  const response = yield* LanguageModel.generateText({
    prompt: "Generate a dad joke"
  })
  console.log(response.text)
  return response
})

const Gpt4o = OpenAiLanguageModel.model("gpt-4o")

const main = generateDadJoke.pipe(
  Effect.provide(Gpt4o)
)

const OpenAi = OpenAiClient.layerConfig({
  apiKey: Config.redacted("OPENAI_API_KEY")
})

const OpenAiWithHttp = Layer.provide(OpenAi, NodeHttpClient.layerUndici)

main.pipe(
  Effect.provide(OpenAiWithHttp),
  Effect.runPromise
)
```

---

### Configure Anthropic Client with NodeHttpClient (TypeScript)

Source: https://effect.website/docs/ai/planning-llm-interactions

Configures the Anthropic client by providing an API key from a redacted environment variable and specifying the Node.js HTTP client (Undici). This setup is essential for making requests to the Anthropic API.

```typescript
const Anthropic = AnthropicClient.layerConfig({
  apiKey: Config.redacted("ANTHROPIC_API_KEY")
}).pipe(Layer.provide(NodeHttpClient.layerUndici))
```

---

### Implement Dad Joke Tool Handlers - Effect AI

Source: https://effect.website/docs/ai/tool-use

Implements the logic for the 'GetDadJoke' tool within a Toolkit using '.toLayer'. This example demonstrates how to access Effect platform services like HttpClient to make API calls and handle responses, returning the joke string.

```typescript
import { Tool, Toolkit } from "@effect/ai"
import { HttpClient, HttpClientRequest, HttpClientResponse } from "@effect/platform"
import { NodeHttpClient } from "@effect/platform-node"
import { Array, Effect, Schema } from "effect"

class DadJoke extends Schema.Class("DadJoke")({
  id: Schema.String,
  joke: Schema.String
}) {}

class SearchResponse extends Schema.Class("SearchResponse")({
  results: Schema.Array(DadJoke)
}) {}

class ICanHazDadJoke extends Effect.Service()("ICanHazDadJoke", {
  dependencies: [NodeHttpClient.layerUndici],
  effect: Effect.gen(function*() {
    const httpClient = yield* HttpClient.HttpClient
    const httpClientOk = httpClient.pipe(
      HttpClient.filterStatusOk,
      HttpClient.mapRequest(HttpClientRequest.prependUrl("https://icanhazdadjoke.com"))
    )

    const search = Effect.fn("ICanHazDadJoke.search")(
      function*(searchTerm: string) {
        return yield* httpClientOk.get("/search", {
          acceptJson: true,
          urlParams: { searchTerm }
        }).pipe(
          Effect.flatMap(HttpClientResponse.schemaBodyJson(SearchResponse)),
          Effect.flatMap(({ results }) => Array.head(results)),
          Effect.map((joke) => joke.joke),
          Effect.orDie
        )
      }
    )

    return {
      search
    } as const
  })
}) {}

const GetDadJoke = Tool.make("GetDadJoke", {
  description: "Get a hilarious dad joke from the ICanHazDadJoke API",
  success: Schema.String,
  failure: Schema.Never,
  parameters: {
    searchTerm: Schema.String.annotations({
      description: "The search term to use to find dad jokes"
    })
  }
})

const DadJokeTools = Toolkit.make(GetDadJoke)

const DadJokeToolHandlers = DadJokeTools.toLayer(
  Effect.gen(function*() {
    // Access the `ICanHazDadJoke` service
    const icanhazdadjoke = yield* ICanHazDadJoke
    return {
      // Implement the handler for the `GetDadJoke` tool call request
      GetDadJoke: ({ searchTerm }) => icanhazdadjoke.search(searchTerm)
    }
  })
)
```

---

### Configure OpenAI Client with NodeHttpClient (TypeScript)

Source: https://effect.website/docs/ai/planning-llm-interactions

Sets up the OpenAI client, similar to the Anthropic client. It retrieves the API key from a redacted environment variable and configures it to use the Node.js Undici HTTP client for making API calls.

```typescript
const OpenAi = OpenAiClient.layerConfig({
  apiKey: Config.redacted("OPENAI_API_KEY")
}).pipe(Layer.provide(NodeHttpClient.layerUndici))
```

---

### Create an ExecutionPlan for LLM Interactions

Source: https://effect.website/docs/ai/planning-llm-interactions

Demonstrates the creation of an ExecutionPlan using the ExecutionPlan.make constructor. This plan is designed to handle LLM interactions, including potential errors, retries, and fallback strategies, in a declarative manner.

```typescript
import type { LanguageModel } from "@effect/ai"
import { OpenAiLanguageModel } from "@effect/ai-openai"
import { Data, Effect, ExecutionPla
```
