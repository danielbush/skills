# Effect Platform – HTTP

`HttpClient`, HTTP server setup, and `RequestResolver`.

---

### Launching an HTTP Server Layer with Layer.launch (TypeScript)

Source: https://effect.website/docs/requirements-management/layers

This example demonstrates how to convert a Layer into an Effect that keeps the layer alive using Layer.launch. It simulates an HTTP server Layer that logs a startup message and then runs this layer indefinitely.

```typescript
import { Console, Context, Effect, Layer } from "effect"

class HTTPServer extends Context.Tag("HTTPServer")() {}

// Simulating an HTTP server
const server = Layer.effect(
  HTTPServer,
  // Log a message to simulate a server starting
  Console.log("Listening on http://localhost:3000")
)

// Converts the layer to an effect and runs it
Effect.runFork(Layer.launch(server))
```

---

### GET /todos

Source: https://effect.website/docs/batching

Fetches a list of todos from an external API.

```APIDOC
## GET /todos

### Description
Fetches a list of todos from an external API.

### Method
GET

### Endpoint
https://api.example.demo/todos

### Parameters

### Request Body
None

### Request Example
None

### Response
#### Success Response (200)
- **Array of Todo objects** (Array<Todo>) - A list of todo items.

#### Response Example
```json
[
  {
    "_tag": "Todo",
    "id": 1,
    "message": "Buy groceries",
    "ownerId": 101
  }
]
```
```

---

### GET /getUserById

Source: https://effect.website/docs/batching

Retrieves a user by their ID from an external API.

```APIDOC
## GET /getUserById

### Description
Retrieves a user by their ID from an external API.

### Method
GET

### Endpoint
https://api.example.demo/getUserById?id={id}

### Parameters
#### Query Parameters
- **id** (number) - Required - The ID of the user to retrieve.

### Request Body
None

### Request Example
None

### Response
#### Success Response (200)
- **User object** (User) - The user object containing id, name, and email.

#### Response Example
```json
{
  "_tag": "User",
  "id": 1,
  "name": "John Doe",
  "email": "john.doe@example.com"
}
```
```

---

### Define HTTP Service and RequestResolver with Context

Source: https://effect.website/docs/batching

Sets up an HTTP service using Effect.Context.Tag and defines a RequestResolver for fetching todos. The resolver uses Effect.andThen to access the HttpService and Effect.tryPromise for making the actual fetch call. It specifies the HttpService as part of the resolver's context dependencies.

```typescript
1 import { Effect, Context, RequestResolver, Request, Data } from "effect"
2 
3 // ------------------------------ 
4 // Model 
5 // ------------------------------ 
6 
19 collapsed lines 
7 interface User {
8 readonly _tag: "User"
9 readonly id: number
10 readonly name: string
11 readonly email: string
12 }
13 
14 class GetUserError extends Data.TaggedError("GetUserError")<{}> {}
15 
16 interface Todo {
17 readonly _tag: "Todo"
18 readonly id: number
19 readonly message: string
20 readonly ownerId: number
21 }
22 
23 class GetTodosError extends Data.TaggedError("GetTodosError")<{}> {}
24 
25 class SendEmailError extends Data.TaggedError("SendEmailError")<{}> {}
26 
27 // ------------------------------ 
28 // Requests 
29 // ------------------------------ 
30 
29 collapsed lines 
31 // Define a request to get multiple Todo items which might
32 // fail with a GetTodosError
33 interface GetTodos extends Request.Request, GetTodosError> {
34 readonly _tag: "GetTodos"
35 }
36 
37 // Create a tagged constructor for GetTodos requests
38 const GetTodos = Request.tagged("GetTodos")
39 
40 // Define a request to fetch a User by ID which might
41 // fail with a GetUserError
42 interface GetUserById extends Request.Request {
43 readonly _tag: "GetUserById"
44 readonly id: number
45 }
46 
47 // Create a tagged constructor for GetUserById requests
48 const GetUserById = Request.tagged("GetUserById")
49 
50 // Define a request to send an email which might
51 // fail with a SendEmailError
52 interface SendEmail extends Request.Request {
53 readonly _tag: "SendEmail"
54 readonly address: string
55 readonly text: string
56 }
57 
58 // Create a tagged constructor for SendEmail requests
59 const SendEmail = Request.tagged("SendEmail")
60 
61 // ------------------------------ 
62 // Resolvers With Context 
63 // ------------------------------ 
64 
65 class HttpService extends Context.Tag("HttpService")< 
66 HttpService,
67 { fetch: typeof fetch }
68 >()
69 
70 const GetTodosResolver =
71 // we create a normal resolver like we did before
72 RequestResolver.fromEffect((_: GetTodos) =>
73 Effect.andThen(HttpService, (http) =>
74 Effect.tryPromise({
75 try: () =>
76 http
77 .fetch("https://api.example.demo/todos")
78 .then((res) => res.json() as Promise<Array<Todo>>),
79 catch: () => new GetTodosError()
80 })
81 )
82 ).pipe(
83 // we list the tags that the resolver can access
84 RequestResolver.contextFromServices(HttpService)
85 )
```

---

### HttpError with Tagged Discriminant - TypeScript

Source: https://effect.website/docs/error-management/expected-errors

This example illustrates the use of `Data.TaggedError` to define a custom error type. It shows how a `_tag` field is automatically added to instances of `HttpError`, serving as a discriminant for distinguishing between different error types, which is crucial for error handling mechanisms like `Effect.catchTag`.

```typescript
// Define a custom error type using Data.TaggedError
class HttpError extends Data.TaggedError("HttpError")<{}> {}

// This field serves as a discriminant for the error
console.log(new HttpError()._tag) // Output: "HttpError"
```

---

### Define TodosService Layer in TypeScript

Source: https://effect.website/docs/batching

Defines a live implementation layer for 'TodosService'. This layer sets up the 'getTodos' functionality by using 'HttpService' to fetch data and returning an Effect that requests 'GetTodos'.

```typescript
class TodosService extends Context.Tag("TodosService")< 
  TodosService,
  { getTodos: Effect.Effect<Array, GetTodosError, TodosService> }
>() {}

const TodosServiceLive = Layer.effect(
  TodosService,
  Effect.gen(function* () {
    const http = yield* HttpService
    const resolver = RequestResolver.fromEffect((_: GetTodos) =>
      Effect.tryPromise({
        try: () =>
          http
            .fetch("https://api.example.demo/todos")
            .then((res) => res.json()),
        catch: () => new GetTodosError()
      })
    )
    return {
      getTodos: Effect.request(GetTodos({}), resolver)
    }
  })
)
```
