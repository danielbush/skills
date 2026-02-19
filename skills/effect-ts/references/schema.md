# Effect – Schema

Validation, encoding/decoding, branded types, JSON Schema generation, and arbitrary data.

---

### Define Schema for Configuration (Effect/TypeScript)

Source: https://effect.website/docs/configuration

Illustrates how to define a configuration schema using Effect's Schema module. This example creates a configuration expecting a string named 'Foo' with a minimum length of 4 characters.

```typescript
import { Effect, Schema } from "effect"

// Define a config that expects a string with at least 4 characters
const myConfig = Schema.Config(
  "Foo",
  Schema.String.pipe(Schema.minLength(4))
)
```

---

### Decoding and Encoding Examples with Product Schema

Source: https://effect.website/docs/schema/advanced-usage

Demonstrates decoding unknown data synchronously and encoding a Product schema, including handling null values and empty objects. It shows how 'Schema.NumberFromString' interacts with optional fields.

```javascript
console.log(Schema.decodeUnknownSync(Product)({ quantity: null }))
// Output: {}
```

```javascript
console.log(Schema.encodeSync(Product)({ quantity: 1 }))
// Output: { quantity: "1" }
```

```javascript
console.log(Schema.encodeSync(Product)({}))
// Output: {}
```

---

### Example: Retain Initial Refinements with encodedBoundSchema - TypeScript

Source: https://effect.website/docs/schema/projections

This example demonstrates the usage of `Schema.encodedBoundSchema`. It shows how the original schema `Original` with a `minLength` refinement and a `Trim` transformation is processed. The resulting `EncodedBoundSchema` retains the `minLength(3)` but omits the `Schema.Trim` transformation, as intended.

```typescript
import { Schema } from "effect"

const Original = Schema.Struct({
  foo: Schema.String.pipe(
    Schema.minLength(3),
    Schema.compose(Schema.Trim)
  )
})

const EncodedBoundSchema = Schema.encodedBoundSchema(Original)

const EncodedBoundSchema2 = Schema.Struct({
  foo: Schema.String.pipe(Schema.minLength(3))
})
```

---

### Trim Whitespace from String using Schema.Trim

Source: https://effect.website/docs/schema/transformations

Illustrates the `Schema.Trim` utility, which removes leading and trailing whitespace from a string. The examples show its effectiveness on strings with spaces at the beginning, end, or both.

```typescript
import { Schema } from "effect"

const decode = Schema.decodeUnknownSync(Schema.Trim)

console.log(decode("a")) // "a"
console.log(decode(" a")) // "a"
console.log(decode("a ")) // "a"
console.log(decode(" a ")) // "a"
```

---

### Customizing Schema with Annotations (TypeScript)

Source: https://effect.website/docs/schema/annotations

This example demonstrates how to use annotations to customize a 'Password' schema. It adds custom error messages for non-string values, required fields, and string length limits. It also includes metadata like 'identifier', 'title', 'description', 'examples', and 'documentation'.

```typescript
import { Schema } from "effect"

// Define a Password schema, starting with a string type
const Password = Schema.String
// Add a custom error message for non-string values
.annotations({ message: () => "not a string" })
.pipe(
// Enforce non-empty strings and provide a custom error message
Schema.nonEmptyString({ message: () => "required" }),
// Restrict the string length to 10 characters or fewer
// with a custom error message for exceeding length
Schema.maxLength(10, {
message: (issue) => `${issue.actual} is too long`
})
)
.annotations({
// Add a unique identifier for the schema
identifier: "Password",
// Provide a title for the schema
title: "password",
// Include a description explaining what this schema represents
description:
"A password is a secret string used to authenticate a user",
// Add examples for better clarity
examples: ["1Ki77y", "jelly22fi$h"],
// Include any additional documentation
documentation: `...technical information on Password schema...`
})
```

---

### Example of Unintended Type Compatibility (TypeScript)

Source: https://effect.website/docs/code-style/branded-types

Illustrates a common issue with TypeScript's structural typing where types with the same underlying structure are treated as interchangeable, potentially leading to bugs. This example shows a UserId being incorrectly passed to a function expecting a ProductId.

```typescript
type UserId = number;
type ProductId = number;

const getUserById = (id: UserId) => {
  // Logic to retrieve user
};

const getProductById = (id: ProductId) => {
  // Logic to retrieve product
};

const id: UserId = 1;

getProductById(id) // No type error, but incorrect usage
```

---

### MutableHashSet Example with Schema.Data

Source: https://effect.website/docs/data-types/hash-set

Illustrates the use of Effect's MutableHashSet in conjunction with Schema.Data. It shows how to create a mutable set and add elements decoded from plain objects, demonstrating that duplicate values based on schema equality are automatically handled.

```typescript
import { Schema, MutableHashSet } from "effect"

// Define a schema that describes the structure of a Person
const PersonSchema = Schema.Data(
  Schema.Struct({
    id: Schema.Number,
    name: Schema.String,
    age: Schema.Number
  })
)

// Decode values from plain objects
const Person = Schema.decodeSync(PersonSchema)

const person1 = Person({ id: 1, name: "Alice", age: 30 })
const person2 = Person({ id: 1, name: "Alice", age: 30 })

// Add both to a MutableHashSet — only one will be stored
const set = MutableHashSet.empty().pipe(
  MutableHashSet.add(person1),
  MutableHashSet.add(person2)
)

console.log(MutableHashSet.size(set))
// Output: 1
```

---

### Simple Custom Message for Scalar Schema (Effect)

Source: https://effect.website/docs/schema/error-messages

Shows a basic example of adding a custom error message to a scalar schema, specifically `Schema.String`, using the `annotations` method. The custom message is defined within the `message` property of the annotation object. The example demonstrates catching the error and logging its message.

```typescript
import { Schema } from "effect"

const MyString = Schema.String.annotations({
  message: () => "my custom message"
})

const decode = Schema.decodeUnknownSync(MyString)

try {
  decode(null)
} catch (e: any) {
  console.log(e.message) // "my custom message"
}
```

---

### Default ParseError (Missing Properties)

Source: https://effect.website/docs/schema/error-messages

This example illustrates the default ParseError when required properties are missing from the input data during schema decoding. It details which properties are expected but not found.

```typescript
import { Schema } from "effect"

const Person = Schema.Struct({
  name: Schema.String,
  age: Schema.Number
})

Schema.decodeUnknownSync(Person)({}, { errors: "all" })
/* 
throws:
ParseError: { readonly name: string; readonly age: number }
├─ ["name"]
│  └─ is missing
└─ ["age"]
   └─ is missing
*/
```

---

### Create User Instances with/without Optional Email in TypeScript

Source: https://effect.website/docs/data-types/option

Provides examples of creating instances of the `User` interface, demonstrating how to assign values to the optional `email` property using `Option.some` and `Option.none`.

```typescript
import { Option } from "effect"

interface User {
  readonly id: number
  readonly username: string
  readonly email: Option.Option
}

const withEmail: User = {
  id: 1,
  username: "john_doe",
  email: Option.some("john.doe@example.com")
}

const withoutEmail: User = {
  id: 2,
  username: "jane_doe",
  email: Option.none()
}
```

---

### Create Instances of a Schema Class (TypeScript)

Source: https://effect.website/docs/schema/classes

Demonstrates creating instances of the `Person` schema class. It shows instantiation using the `new` keyword with an object literal and using the `make` factory function provided by `Schema.Class`. Both methods ensure the created instances conform to the schema.

```typescript
import { Schema } from "effect"

class Person extends Schema.Class("Person")({
  id: Schema.Number,
  name: Schema.NonEmptyString
}) {}

console.log(new Person({ id: 1, name: "John" }))
/* 
Output: 
Person { id: 1, name: 'John' } 
*/

// Using the factory function
console.log(Person.make({ id: 1, name: "John" }))
/* 
Output: 
Person { id: 1, name: 'John' } 
*/

```

---

### Split String by Delimiter using Schema.split

Source: https://effect.website/docs/schema/transformations

Demonstrates the `Schema.split` function, which splits a string into an array of substrings based on a specified delimiter. Examples show splitting an empty string, a string with only a delimiter, and strings with leading/trailing delimiters.

```typescript
import { Schema } from "effect"

const schema = Schema.split(",")

const decode = Schema.decodeUnknownSync(schema)

console.log(decode("")) // [""].
console.log(decode(",")) // ["", ""].
console.log(decode("a,")) // ["a", ""].
console.log(decode("a,b")) // ["a", "b"]
```

---

### Trim Record Keys During Decoding (Effect Schema)

Source: https://effect.website/docs/schema/basic-usage

Provides an example of how to trim keys while decoding a record by using 'Schema.transform' and a custom 'decode' function that maps keys after trimming.

```typescript
import { Schema, Record, identity } from "effect"

const schema = Schema.transform(
  // Define the input schema with unprocessed keys
  Schema.Record({
    key: Schema.String,
    value: Schema.NumberFromString
  }),
  // Define the output schema with transformed keys
  Schema.Record({
    key: Schema.Trimmed,
    value: Schema.Number
  }),
  {
    strict: true,
    // Trim keys during decoding
    decode: (record) => Record.mapKeys(record, (key) => key.trim()),
    encode: identity
  }
)

console.log(
  Schema.decodeUnknownSync(schema)({" key1 ": "1", key2: "2"})
)
// Output: { key1: 1, key2: 2 }
```

---

### Effect Schema: Number Comparison Filters

Source: https://effect.website/docs/schema/filters

Provides examples of number filters for validating if a number is greater than, less than, greater than or equal to, or less than or equal to a specified value.

```typescript
import { Schema } from "effect"

// Specifies a number greater than 5
Schema.Number.pipe(Schema.greaterThan(5))

// Specifies a number greater than or equal to 5
Schema.Number.pipe(Schema.greaterThanOrEqualTo(5))

// Specifies a number less than 5
Schema.Number.pipe(Schema.lessThan(5))

// Specifies a number less than or equal to 5
Schema.Number.pipe(Schema.lessThanOrEqualTo(5))

// Specifies a number between -2 and 2, inclusive
Schema.Number.pipe(Schema.between(-2, 2))
```

---

### Decoding with decodeUnknown

Source: https://effect.website/docs/schema/getting-started

Decodes a value and returns an `Effect`. This is suitable for schemas involving asynchronous transformations. The `Effect` can be run synchronously or asynchronously to get the result or handle errors.

```APIDOC
## POST /decodeUnknown

### Description
Decodes a value using a schema and returns an `Effect`. This function is specifically designed for schemas that include asynchronous operations or transformations. The resulting `Effect` encapsulates the decoding process, allowing for integrated error handling and asynchronous execution.

### Method
POST

### Endpoint
`/decodeUnknown`

### Parameters
#### Request Body
- **schema** (Schema) - Required - The schema to use for decoding, potentially including async transformations.
- **unknownValue** (unknown) - Required - The value to decode.

### Request Example
```json
{
  "schema": "Schema.transformOrFail(Schema.Number, Schema.Struct({\"id\": Schema.Number, \"name\": Schema.String, \"age\": Schema.Number}), {\n  decode: (id) => Effect.succeed({ id, name: \"name\", age: 18 }).pipe(Effect.delay(\"10 millis\"))\n})",
  "unknownValue": 1
}
```

### Response
#### Success Response (200)
- **effectResult** (object) - The result of running the `Effect`, containing the decoded value.

#### Response Example
```json
{
  "effectResult": {
    "id": 1,
    "name": "name",
    "age": 18
  }
}
```

#### Error Response (400)
- **error** (object) - Contains details about the error during effect execution or decoding.
  - **message** (string) - Description of the error.

#### Error Response Example
```json
{
  "error": {
    "message": "An error occurred during asynchronous decoding."
  }
}
```
```

---

### Create Effect Schema Class Instance with Validation

Source: https://effect.website/docs/schema/classes

Demonstrates creating an instance of a Person class defined with Effect Schema. The example shows how validation fails if the 'name' property is an empty string, as it requires a NonEmptyString. The expected output is a ParseError.

```typescript
import { Schema } from "effect"

class Person extends Schema.Class("Person")({
  id: Schema.Number,
  name: Schema.NonEmptyString
}) {}

// Attempt to create an instance with an invalid `name`
new Person({ id: 1, name: "" })
/* 
throws:
ParseError: Person (Constructor) 
└─ ["name"]
└─ NonEmptyString
└─ Predicate refinement failure
└─ Expected NonEmptyString, actual ""
*/
```

---

### Effect Schema: Accessing Base Schema with 'from'

Source: https://effect.website/docs/schema/filters

This example shows how to access the original schema before a filter was applied using the 'from' property. This is useful when you need to refer to the unfiltered schema for further operations.

```typescript
import { Schema } from "effect"

const LongString = Schema.String.pipe(
  Schema.filter((s) => s.length >= 10)
)

// Access the base schema, which is the string schema
// before the filter was applied
const From = LongString.from
```

---

### Duration Filters in Effect Schema

Source: https://effect.website/docs/schema/filters

Explains filters for validating Duration values, including comparisons like greater than, less than, and greater than or equal to, with examples using string representations of durations.

```typescript
import { Schema } from "effect"

// Specifies a duration greater than 5 seconds
Schema.Duration.pipe(Schema.greaterThanDuration("5 seconds"))

// Specifies a duration greater than or equal to 5 seconds
Schema.Duration.pipe(Schema.greaterThanOrEqualToDuration("5 seconds"))

// Specifies a duration less than 5 seconds
Schema.Duration.pipe(Schema.lessThanDuration("5 seconds"))
```

---

### Scoped Annotations with Effect.annotateLogsScoped

Source: https://effect.website/docs/observability/logging

This example demonstrates how to use `Effect.annotateLogsScoped` to apply annotations to log messages within a specific scope. Annotations are only applied to logs generated within the scope defined by `Effect.annotateLogsScoped` and are not present in logs outside this scope.

```typescript
import { Effect } from "effect"

const program = Effect.gen(function* () {
  yield* Effect.log("no annotations") // No annotations
  yield* Effect.annotateLogsScoped({ key: "value" }) // Scoped annotation
  yield* Effect.log("message1") // Annotation applied
  yield* Effect.log("message2") // Annotation applied
}).pipe(
  Effect.scoped,
  // Outside scope, no annotations
  Effect.andThen(Effect.log("no annotations again"))
)

Effect.runFork(program)
```

---

### Define and Compare Recursive Case Classes with Data.case

Source: https://effect.website/docs/data-types/data

Illustrates the creation of a recursive data structure, a binary tree, using Data.case. This example shows how to define a type that can contain itself and how to compare instances for equality.

```typescript
import { Data, Equal } from "effect"

interface BinaryTree<T> {
  readonly value: T
  readonly left: BinaryTree<T> | null
  readonly right: BinaryTree<T> | null
}

const BinaryTree = Data.case<BinaryTree<number>>()

const tree1 = BinaryTree({
  value: 0,
  left: BinaryTree({ value: 1, left: null, right: null }),
  right: null
})

const tree2 = BinaryTree({
  value: 0,
  left: BinaryTree({ value: 1, left: null, right: null }),
  right: null
})

console.log(Equal.equals(tree1, tree2)) // Output: true
```

---

### Defining Tuples with Optional Elements in Effect Schema

Source: https://effect.website/docs/schema/basic-usage

Explains how to define a tuple that includes optional elements using Schema.optionalElement. The example shows a tuple with a required string and an optional number.

```typescript
import { Schema } from "effect"

// Define a tuple with a required string and an optional number
const schema = Schema.Tuple(
  Schema.String, // required element
  Schema.optionalElement(Schema.Number) // optional element
)

// ┌─── readonly [string, number?]
// ▼
type Type = typeof schema.Type
```

---

### Default ParseError (Type Mismatch)

Source: https://effect.website/docs/schema/error-messages

This example shows the default ParseError when a data type mismatch occurs during schema decoding. It highlights the expected structure versus the actual received input.

```typescript
import { Schema } from "effect"

const Person = Schema.Struct({
  name: Schema.String,
  age: Schema.Number
})

Schema.decodeUnknownSync(Person)(null)
// Output: ParseError: Expected { readonly name: string; readonly age: number }, actual null
```

---

### Using Exactness with Optional Field

Source: https://effect.website/docs/schema/advanced-usage

This example shows how to use `exact: true` with Schema.optionalWith to enforce strict typing for an optional field. Decoding `undefined` will result in a `ParseError`.

```typescript
import { Schema } from "effect"

const Product = Schema.Struct({
  quantity: Schema.optionalWith(Schema.NumberFromString, {
    exact: true
  })
})

// Decoding examples
console.log(Schema.decodeUnknownSync(Product)({ quantity: "1" }))
// Output: { quantity: 1 }
console.log(Schema.decodeUnknownSync(Product)({}))
// Output: {}
console.log(Schema.decodeUnknownSync(Product)({ quantity: undefined }))
/* 
throws:
ParseError: { readonly quantity?: NumberFromString } 
└─ ["quantity"]
└─ NumberFromString
└─ Encoded side transformation failure
└─ Expected string, actual undefined 
*/

// Encoding examples
console.log(Schema.encodeSync(Product)({ quantity: 1 }))
// Output: { quantity: "1" }
console.log(Schema.encodeSync(Product)({}))
// Output: {}
```

---

### Add Custom Method to Effect Schema Class

Source: https://effect.website/docs/schema/classes

Illustrates adding a custom method to an Effect Schema class. The example defines a 'greet' method within the Person class that returns a greeting string incorporating the person's name.

```typescript
import { Schema } from "effect"

class Person extends Schema.Class("Person")({
  id: Schema.Number,
  name: Schema.NonEmptyString
}) {
  // Custom method to return a greeting
  greet() {
    return `Hello, my name is ${this.name}`
  }
}

const john = new Person({ id: 1, name: "John" })

// Use the custom method
console.log(john.greet())
// Output: "Hello, my name is John"
```

---

### Define Discriminated Union in TypeScript

Source: https://effect.website/docs/schema/basic-usage

This example shows the basic TypeScript syntax for defining a discriminated union with a common discriminant property ('kind') for different shape types (Circle, Square).

```typescript
type Circle = {
  readonly kind: "circle"
  readonly radius: number
}

type Square = {
  readonly kind: "square"
  readonly sideLength: number
}

type Shape = Circle | Square
```

---

### Define TaggedStruct with Multiple Tags

Source: https://effect.website/docs/schema/basic-usage

Illustrates how to define a schema using Schema.TaggedStruct that includes multiple tags. This example adds a 'category' tag alongside the primary '_tag' for more specific data structuring.

```typescript
import { Schema } from "effect"

const Product = Schema.TaggedStruct("Product", {
  category: Schema.tag("Electronics"),
  name: Schema.String,
  price: Schema.Number
})

// `_tag` and `category` are optional when creating an instance
console.log(Product.make({ name: "Smartphone", price: 999 }))
/* 
Output: 
{
  _tag: 'Product',
  category: 'Electronics',
  name: 'Smartphone',
  price: 999
}
*/
```

---

### Combining Nullability and Exactness with Optional Field

Source: https://effect.website/docs/schema/advanced-usage

This example combines `exact: true` and `nullable: true` in Schema.optionalWith. It enforces strict typing while treating `null` as a missing value during decoding.

```typescript
import { Schema } from "effect"

const Product = Schema.Struct({
  quantity: Schema.optionalWith(Schema.NumberFromString, {
    exact: true,
    nullable: true
  })
})

// Decoding examples
console.log(Schema.decodeUnknownSync(Product)({ quantity: "1" }))
// Output: { quantity: 1 }
console.log(Schema.decodeUnknownSync(Product)({}))
// Output: {}
console.log(Schema.decodeUnknownSync(Product)({ quantity: undefined }))
/* 
throws:
ParseError: (Struct (Encoded side) <-> Struct (Type side))
└─ Encod
```

---

### Asynchronous Schema Validation Example

Source: https://effect.website/docs/schema/standard-schema

Demonstrates asynchronous validation of a schema. It shows how to validate an object against a standard schema and the expected output when validation fails due to an empty string for the 'name' field. This function relies on the 'effect' library.

```typescript
import { Effect, Schema } from "effect"

// Assuming asyncStandardSchema is defined elsewhere and contains a "~standard" property for validation.
// Example structure for asyncStandardSchema["~standard"]: Schema.Schema<any, any>

// Validate asynchronously
console.log(asyncStandardSchema["~standard"].validate({ name: "" }))
/* 
Output:
Promise { 
"issues": [
  {
    "path": [
      "name"
    ],
    "message": "Expected a non empty string, actual \"\""
  }
]
}
*/
```

---

### Simplifying Error Handling with Generators and safeTry

Source: https://effect.website/docs/additional-resources/effect-vs-neverthrow

This example shows how generators can simplify complex error handling chains by abstracting away repetitive error checking. It uses the `safeTry` function from neverthrow.

```typescript
import { Result, ok, safeTry } from "neverthrow"

declare function mayFail1(): Result<any, any>
declare function mayFail2(): Result<any, any>

function myFunc(): Result {
  return safeTry(function* () {
    return ok(
      (yield* mayFail1().mapErr(

```

---

### Effect Schema: Pick Properties from Union

Source: https://effect.website/docs/schema/basic-usage

This example shows how to apply the `pick` function more broadly to a union of schemas in Effect. It creates a new schema by selecting specific properties ('a' and 'b') that are common across the schemas within the union.

```typescript
import { Schema } from "effect"

const MyUnion = Schema.Union(
  Schema.Struct({
    a: Schema.String,
    b: Schema.String,
    c: Schema.String
  }),
  Schema.Struct({
    a: Schema.Number,
    b: Schema.Number,
    d: Schema.Number
  })
)

const PickedSchema = MyUnion.pipe(Schema.pick("a", "b"))
```

---

### Handling Null as Missing Value with Optional Field

Source: https://effect.website/docs/schema/advanced-usage

This example demonstrates how Schema.optionalWith treats null as a missing value during decoding and encoding. It uses Schema.NumberFromString for the field's type.

```typescript
import { Schema } from "effect"

const Product = Schema.Struct({
  quantity: Schema.optionalWith(Schema.NumberFromString, {
    nullable: true
  })
})

// Decoding examples
console.log(Schema.decodeUnknownSync(Product)({ quantity: "1" }))
// Output: { quantity: 1 }
console.log(Schema.decodeUnknownSync(Product)({}))
// Output: {}
console.log(Schema.decodeUnknownSync(Product)({ quantity: undefined }))
// Output: { quantity: undefined }
console.log(Schema.decodeUnknownSync(Product)({ quantity: null }))
// Output: {}

// Encoding examples
console.log(Schema.encodeSync(Product)({ quantity: 1 }))
// Output: { quantity: "1" }
console.log(Schema.encodeSync(Product)({}))
// Output: {}
console.log(Schema.encodeSync(Product)({ quantity: undefined }))
// Output: { quantity: undefined }
```

---

### Decode Hex Encoded String to UTF-8 using Effect Schema

Source: https://effect.website/docs/schema/transformations

Decodes a hex encoded string into a UTF-8 string using `Schema.StringFromHex`. The example demonstrates converting hex pairs to their corresponding byte values using `TextEncoder`.

```typescript
import { Schema } from "effect"

const decode = Schema.decodeUnknownSync(Schema.StringFromHex)

console.log(new TextEncoder().encode(decode("0001020304050607")))
/*
Output:
Uint8Array(8) [
  0,
  1,
  2,
  3,
  4,
  5,
  6,
  7
]
*/
```

---

### Handling Shallow Defaults in Nested Structs (Effect Schema)

Source: https://effect.website/docs/schema/default-constructors

Shows an example where default values in nested structs do not automatically apply to the top-level constructor. This leads to a type error if the nested property is missing.

```typescript
import { Schema } from "effect"

const Config = Schema.Struct({
  // Define a nested struct with a default value
  web: Schema.Struct({
    application_url: Schema.String.pipe(
      Schema.propertySignature,
      Schema.withConstructorDefault(() => "http://localhost")
    ),
    application_port: Schema.Number
  })
})

// This will cause a type error because 'application_url'
// is missing in the nested struct
// Config.make({ web: { application_port: 3000 } })
```

---

### Conflicting Index Signature Example (Effect Schema)

Source: https://effect.website/docs/schema/basic-usage

Demonstrates a conflicting index signature scenario where a fixed property 'a' has a different type (string) than the values allowed by the index signature (number). This leads to incorrect TypeScript typing.

```typescript
import { Schema } from "effect"

// Attempting to define a struct with a conflicting index signature
// - The fixed property "a" is a string
// - The index signature requires all values to be numbers
const schema = Schema.Struct(
  {
    a: Schema.String
  },
  {
    key: Schema.String,
    value: Schema.Number
  }
)

// ❌ Incorrect TypeScript type:
// type Type = typeof schema.Type
```

---

### Effect Schema: Omit Properties from Union

Source: https://effect.website/docs/schema/basic-usage

This example demonstrates applying the `omit` function to a union of schemas in Effect. It creates a new schema by excluding a specific property ('b') that exists in the schemas within the union.

```typescript
import { Schema } from "effect"

const MyUnion = Schema.Union(
  Schema.Struct({
    a: Schema.String,
    b: Schema.String,
    c: Schema.String
  }),
  Schema.Struct({
    a: Schema.Number,
    b: Schema.Number,
    d: Schema.Number
  })
)

const PickedSchema = MyUnion.pipe(Schema.omit("b"))
```

---

### Default ParseError (Incorrect Property Type)

Source: https://effect.website/docs/schema/error-messages

This example demonstrates the default ParseError when a property has an incorrect data type compared to what the schema expects. It specifies the expected type and the actual received type for each erroneous property.

```typescript
import { Schema } from "effect"

const Person = Schema.Struct({
  name: Schema.String,
  age: Schema.Number
})

Schema.decodeUnknownSync(Person)(
  { name: null, age: "age" },
  { errors: "all" }
)
/* 
throws:
ParseError: { readonly name: string; readonly age: number }
├─ ["name"]
│  └─ Expected string, actual null
└─ ["age"]
   └─ Expected number, actual "age"
*/
```

---

### Parse and Validate JSON Structure using Effect Schema

Source: https://effect.website/docs/schema/transformations

Parses a JSON string and validates its structure against a provided schema using `Schema.parseJson` with a nested schema. This example uses a `Schema.Struct` to ensure the parsed JSON is an object with a numeric property `a`.

```typescript
import { Schema } from "effect"

// ┌─── SchemaClass<{ readonly a: number; }, string, never>
// ▼ 
const schema = Schema.parseJson(Schema.Struct({ a: Schema.Number }))
```

---

### Invalid Struct with Duplicate String Index Signatures (Effect Schema)

Source: https://effect.website/docs/schema/basic-usage

Shows an example of an invalid struct definition where multiple string index signatures are attempted. This will result in a TypeScript error indicating a duplicate index signature.

```typescript
import { Schema } from "effect"

Schema.Struct(
  {
    a: Schema.Number
  },
  // Attempting to define multiple string index signatures
  {
    key: Schema.String,
    value: Schema.Number
  },
  {
    key: Schema.String,
    value: Schema.Boolean
  }
)
/*
throws:
Error: Duplicate index signature 
details: string index signature
*/
```

---

### Refinement Default Constructor Example - Effect Schema

Source: https://effect.website/docs/schema/default-constructors

Demonstrates using a default constructor with a Schema.NumberFromString refined by Schema.between. The constructor accepts numbers within the valid range (1-10) and throws an error for out-of-range numbers.

```typescript
import { Schema } from "effect"

const schema = Schema.NumberFromString.pipe(
  Schema.between(1, 10)
)

// The constructor only accepts numbers
console.log(schema.make(5)) // Output: 5

// This will throw an error because the number is outside the valid range
console.log(schema.make(20))
/*
throws:
ParseError: between(1, 10)
└─ Predicate refinement failure
└─ Expected a number between 1 and 10, actual 20
*/
```

---

### Asynchronous API Data Validation with Effect-TS Schema

Source: https://effect.website/docs/schema/transformations

Shows how to perform asynchronous transformations and validations using `Schema.transformOrFail` by returning an `Effect`. This example validates a person's ID by making an API call. Failures in the API call are caught and converted into `ParseResult.Type` errors.

```typescript
import { Effect, Schema, ParseResult } from "effect"

// Define a function to make API requests
const get = (url: string): Effect.Effect =>
  Effect.tryPromise({
    try: () =>
      fetch(url).then((res) => {
        if (res.ok) {
          return res.json() as Promise
        }
        throw new Error(String(res.status))
      }),
    catch: (e) => new Error(String(e))
  })

// Create a branded schema for a person's ID
const PeopleId = Schema.String.pipe(Schema.brand("PeopleId"))

// Define a schema with async transformation
const PeopleIdFromString = Schema.transformOrFail(
  Schema.String,
  PeopleId,
  {
    strict: true,
    decode: (s, _, ast) =>
      // Make an API call to validate the ID
      Effect.mapBoth(get(`https://swapi.dev/api/people/${s}`), {
        // Error handling for failed API call
        onFailure: (e) => new ParseResult.Type(ast, s, e.message),
        // Return the ID if the API call succeeds
        onSuccess: () => s
      }),
    encode: ParseResult.succeed
  }
)

// ┌─── string 
// ▼ 
type Encoded = typeof PeopleIdFromString.Encoded

// ┌─── string & Brand<"PeopleId"> 
// ▼ 
type Type = typeof PeopleIdFromString.Type

// ┌─── never 
// ▼ 
type Context = typeof PeopleIdFromString.Context

// Run a successful decode operation
Effect.runPromiseExit(Schema.decodeUnknown(PeopleIdFromString)("1")).then(
  console.log
)
/* 
Output: 
{ _id: 'Exit', _tag: 'Success', value: '1' } 
*/

// Run a decode operation that will fail
Effect.runPromiseExit(
  Schema.decodeUnknown(PeopleIdFromString)("fail")
).then(console.log)
```

---

### Bypass Validation in Effect Schema Class Instantiation

Source: https://effect.website/docs/schema/classes

Illustrates how to bypass validation when creating an instance of a Person class using Effect Schema. The example shows two methods: passing 'true' as the second argument to the constructor, or passing an options object with 'disableValidation: true'.

```typescript
import { Schema } from "effect"

class Person extends Schema.Class("Person")({
  id: Schema.Number,
  name: Schema.NonEmptyString
}) {}

// Bypass validation during instantiation
const john = new Person({ id: 1, name: "" }, true)

// Or use the `disableValidation` option explicitly
new Person({ id: 1, name: "" }, { disableValidation: true })
```

---

### Literal Schemas in Effect

Source: https://effect.website/docs/schema/basic-usage

Illustrates the creation of literal schemas for exact values of string, number, boolean, null, and bigint. Examples show defining single literals and unions of literals.

```typescript
import { Schema } from "effect"

// Define various literal schemas
Schema.Null // Same as S.Literal(null)
Schema.Literal("a") // string literal
Schema.Literal(1) // number literal
Schema.Literal(true) // boolean literal
Schema.Literal(2n) // BigInt literal
```

```typescript
import { Schema } from "effect"

// ┌─── Literal<["a"]>
// ▼
const schema = Schema.Literal("a")

// ┌─── "a"
// ▼
type Type = typeof schema.Type

console.log(Schema.decodeUnknownSync(schema)("a"))
// Output: "a"

console.log(Schema.decodeUnknownSync(schema)("b"))
/* 
throws:
ParseError: Expected "a", actual "b"
*/
```

```typescript
import { Schema } from "effect"

// ┌─── Literal<["a", "b", "c"]>
// ▼
const schema = Schema.Literal("a", "b", "c")

// ┌─── "a" | "b" | "c"
// ▼
type Type = typeof schema.Type

Schema.decodeUnknownSync(schema)(null)
/* 
throws:
ParseError: "a" | "b" | "c"
├─ Expected "a", actual null
├─ Expected "b", actual null
└─ Expected "c", actual null
*/
```

```typescript
import { Schema } from "effect"

// Schema with individual messages for each literal
const individualMessages = Schema.Literal("a", "b", "c")

console.log(Schema.decodeUnknownSync(individualMessages)(null))
/* 
throws:
ParseError: "a" | "b" | "c"
├─ Expected "a", actual null
├─ Expected "b", actual null
└─ Expected "c", actual null
*/

// Schema with a unified custom message for all literals
const unifiedMessage = Schema.Literal("a", "b", "c").annotations({
  message: () => ({ message: "Not a valid code", override: true })
})

console.log(Schema.decodeUnknownSync(unifiedMessage)(null))
/* 
throws:
ParseError: Not a valid code
*/
```

```typescript
import { Schema } from "effect"

const schema = Schema.Literal("a", "b", "c")

// ┌─── readonly ["a", "b", "c"]
// ▼
const literals = schema.literals
```

---

### Use the `Int` Constructor with Validation (TypeScript)

Source: https://effect.website/docs/code-style/branded-types

This example shows how to use the `Int` constructor created with `Brand.refined`. It demonstrates creating a valid `Int` value and how attempting to create an `Int` with a non-integer value (e.g., a float) will result in a runtime error with a descriptive message.

```typescript
import { Brand } from "effect"

type Int = number & Brand.Brand<"Int">

const Int = Brand.refined<number, "Int">( 
  (n) => Number.isInteger(n),
  (n) => Brand.error(`Expected ${n} to be an integer`)
)

// Create a valid Int value
const x: Int = Int(3)
console.log(x) // Output: 3

// Attempt to create an Int with an invalid value
// const y: Int = Int(3.14)
// throws [ { message: 'Expected 3.14 to be an integer' } ]

```

---

### Defining Environment with Union of Services in Effect

Source: https://effect.website/docs/additional-resources/coming-from-zio

This example demonstrates how to define the environment required for an Effect workflow using a union of services in TypeScript. It shows the structure for `IOError`, `HttpError`, `Console`, and `Logger` interfaces, culminating in a `Http` type that is a union of `Console` and `Logger`. This approach differs from ZIO's intersection-based environment representation.

```typescript
import { Effect } from "effect"

interface IOError {
  readonly _tag: "IOError"
}

interface HttpError {
  readonly _tag: "HttpError"
}

interface Console {
  readonly log: (msg: string) => void
}

interface Logger {
  readonly log: (msg: string) => void
}

type Response = Record

// `R` is a union of `Console` and `Logger`
type Http = Effect.Effect<never, IOError | HttpError, Response>
```

---

### Defining a Person Schema with Optional Properties

Source: https://effect.website/docs/schema/introduction

This example demonstrates how to define a schema for a 'Person' object using effect/Schema. It utilizes 'Schema.optionalWith' to specify an optional 'name' property, leveraging the 'exact: true' option for precise handling of optional properties when 'exactOptionalPropertyTypes' is enabled in TypeScript.

```typescript
import { Schema } from "effect"

const Person = Schema.Struct({
  name: Schema.optionalWith(Schema.NonEmptyString, { exact: true 

```

---

### Handle Sync and Async Validation with Standard Schema V1

Source: https://effect.website/docs/schema/standard-schema

Illustrates how the `validate` method of a Standard Schema V1 object can return either a direct value (for synchronous validation) or a `Promise` (for asynchronous validation), depending on the underlying Effect Schema. Includes examples for both scenarios.

```typescript
import { Effect, Schema } from "effect"

// Utility function to display sync and async results
const print = (t: T) =>
 t instanceof Promise
 ? t.then((x) => console.log("Promise", JSON.stringify(x, null, 2)))
 : console.log("Value", JSON.stringify(t, null, 2))

// Define a synchronous schema
const sync = Schema.Struct({
 name: Schema.String
})

// Generate a Standard Schema V1 object
const syncStandardSchema = Schema.standardSchemaV1(sync)

// Validate synchronously
print(syncStandardSchema["~standard"].validate({ name: null }))
/*
Output:
{
  "issues": [
    {
      "path": [
        "name"
      ],
      "message": "Expected string, actual null"
    }
  ]
}
*/

// Define an asynchronous schema with a transformation
const async = Schema.transformOrFail(
 sync,
 Schema.Struct({
  name: Schema.NonEmptyString
 }),
 {
  // Simulate an asynchronous validation delay
  decode: (x) => Effect.sleep("100 millis").pipe(Effect.as(x)),
  encode: Effect.succeed
 }
)

// Generate a Standard Schema V1 object
const asyncStandardSchema = Schema.standardSchemaV1(async)
```

---

### Map Property Signature from a Different Key in Effect Schema

Source: https://effect.website/docs/schema/advanced-usage

This example shows how to map a property signature from a source field with a different key name to a target field in your internal model. The Schema.fromKey function is used to establish this mapping.

```typescript
import { Schema } from "effect"

const Person = Schema.Struct({
  name: Schema.String,
  age: Schema.propertySignature(Schema.NumberFromString).pipe(
    Schema.fromKey("AGE") // Maps from "AGE" to "age"
  )
})

console.log(Schema.decodeUnknownSync(Person)({" name": "name", AGE: "18" }))
// Output: { name: 'name', age: 18 }
```

---

### Covariant Type Parameter in EffectJS

Source: https://effect.website/docs/additional-resources/coming-from-zio

Demonstrates how EffectJS uses a covariant R type parameter, utilizing unions for composing services. This differs from ZIO's approach and improves type signature clarity. The example shows assigning a union type to a variable.

```typescript
interface A {
  readonly prop: string
}

interface B {
  readonly prop: number
}

// ok
const ab: A | B = {
  prop: ""
}
```

---

### Using a Rest Element in Effect Schema Tuples

Source: https://effect.website/docs/schema/basic-usage

Illustrates the use of a rest element in Effect's Schema.Tuple constructor, allowing the tuple to accept any number of additional elements of a specified type. The example defines a tuple with required and optional elements followed by a rest element of booleans.

```typescript
import { Schema } from "effect"

// Define a tuple with required elements and a rest element of type boolean
const schema = Schema.Tuple(
  [Schema.String, Schema.optionalElement(Schema.Number)], // elements
  Schema.Boolean // rest element
)

// ┌─── readonly [string, number?, ...boolean[]]
// ▼
type Type = typeof schema.Type
```

---

### Encode/Decode HashSetFromSelf with Effect-TS Schema

Source: https://effect.website/docs/schema/effect-data-types

This snippet illustrates using Schema.HashSetFromSelf for scenarios where HashSets are already in the correct format but inner values need transformation. It demonstrates decoding a HashSet of strings to a HashSet of numbers and encoding vice-versa. The examples show the resulting HashSets after decoding and encoding.

```typescript
import { Schema } from "effect"
import { HashSet } from "effect"

const schema = Schema.HashSetFromSelf(Schema.NumberFromString)

// ┌─── HashSet
// ▼
type Encoded = typeof schema.Encoded

// ┌─── HashSet
// ▼
type Type = typeof schema.Type

const decode = Schema.decodeUnknownSync(schema)
const encode = Schema.encodeSync(schema)

// Decoding examples

console.log(decode(HashSet.fromIterable(["1", "2", "3"])))
// Output: { _id: 'HashSet', values: [ 1, 2, 3 ] }

// Encoding examples

console.log(encode(HashSet.fromIterable([1, 2, 3])))
// Output: { _id: 'HashSet', values: [ '1', '3', '2' ] }
```

---

### Get and Validate User Guess (TypeScript)

Source: https://effect.website/docs/platform/terminal

An Effect that orchestrates prompting the user for input and validating it using `parseGuess`. If the input is invalid, it displays an error message and recursively calls itself to re-prompt. This ensures the game proceeds only with valid guesses.

```typescript
import { Effect, Option, Random } from "effect"
import { NodeRuntime, NodeTerminal } from "@effect/platform-node"
import { Terminal, PlatformError } from "@effect/platform"

// Get the user's guess, validating it as an integer between 1 and 100
const answer: Effect.Effect<
  number,
  Terminal.QuitException | PlatformError,
  Terminal.Terminal
> = Effect.gen(function* () {
  const input = yield* prompt
  const guess = parseGuess(input)
  if (Option.isNone(guess)) {
    yield* display("You must enter an integer from 1 to 100")
    return yield* answer
  }
  return guess.value
})
```

---

### Custom Pretty Printer for Numbers with Annotation (TypeScript)

Source: https://effect.website/docs/schema/pretty

This example shows how to define a custom pretty printer for a schema by using the 'pretty' annotation. The annotation provides a function that takes the value and returns its formatted string representation. This allows for specific formatting logic for different data types within a schema. Dependencies include 'effect'.

```typescript
import { Pretty, Schema } from "effect"

// Define a schema with a custom pretty annotation
const schema = Schema.Number.annotations({
  pretty: (/\*typeParameters*\/) => (value) => `my format: ${value}`
})

// Create the pretty printer
const customPrettyPrinter = Pretty.make(schema)

// Format and print a value
console.log(customPrettyPrinter(1))
// Output: "my format: 1"
```

---

### Create a Valid Schema Class Instance (TypeScript)

Source: https://effect.website/docs/schema/classes

Shows the creation of a valid `Person` instance using its constructor. The constructor automatically validates that the provided properties (`id` and `name`) adhere to the schema's rules, ensuring `id` is a number and `name` is a non-empty string.

```typescript
import { Schema } from "effect"

class Person extends Schema.Class("Person")({
  id: Schema.Number,
  name: Schema.NonEmptyString
}) {}

// Create an instance with valid properties
const john = new Person({ id: 1, name: "John" })

```

---

### Defining Template Literals with Schema.TemplateLiteral - Effect Schema

Source: https://effect.website/docs/schema/basic-usage

Illustrates how to use Schema.TemplateLiteral to create schemas for TypeScript template literal types. Examples include basic string interpolation and unions of literal string endings.

```typescript
import { Schema } from "effect"

// This creates a schema for: `a${string}`
// 
// ┌─── TemplateLiteral<`a${string}`>
// ▼
const schema1 = Schema.TemplateLiteral("a", Schema.String)

// This creates a schema for:
// `https://${string}.com` | `https://${string}.net`
const schema2 = Schema.TemplateLiteral(
  "https://",
  Schema.String,
  ".",
  Schema.Literal("com", "net")
)
```

---

### Omitting Empty Strings with `optionalToOptional` in Effect Schema

Source: https://effect.website/docs/schema/advanced-usage

Shows how to use `Schema.optionalToOptional` to transform an optional string field. This example specifically demonstrates how to omit fields containing empty strings from the output during decoding, treating them as absent.

```typescript
import { Option, Schema } from "effect"

const schema = Schema.Struct({
  nonEmpty: Schema.optionalToOptional(Schema.String, Schema.String, {
    // ┌─── Option
    // ▼
    decode: (maybeString) => {
      if (Option.isNone(maybeString)) {
        // If `maybeString` is `None`, the field is absent in the input.
        // Return O
```

---

### Adding Metadata with Annotations in Effect Schema

Source: https://effect.website/docs/schema/filters

Example of how to add metadata such as identifiers, JSON schema specifications, and descriptions to a schema using annotations in Effect. This enhances schema understanding and analysis. It uses Schema.filter to apply a length constraint and associates metadata with the schema.

```typescript
import { Schema, JSONSchema } from "effect"

const LongString = Schema.String.pipe(
  Schema.filter(
    (s) =>
      s.length >= 10 ? undefined : "a string at least 10 characters long",
    {
      identifier: "LongString",
      jsonSchema: { minLength: 10 },
      description: "Lorem ipsum dolor sit amet, ..."
    }
  )
)

console.log(Schema.decodeUnknownSync(LongString)("a"))
/*
throws:
ParseError: LongString
└─ Predicate refinement failure
└─ a string at least 10 characters long
*/

console.log(JSON.stringify(JSONSchema.make(LongString), null, 2))
/*
Output:
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$defs": {
    "LongString": {
      "type": "string",
      "description": "Lorem ipsum dolor sit amet, ...",
      "minLength": 10
    }
  },
  "$ref": "#/\$defs/LongString"
}
*/
```

---

### Generate JSON Schema with Standard Annotations (Effect-TS)

Source: https://effect.website/docs/schema/json-schema

This snippet demonstrates how to use standard JSON Schema annotations such as 'description', 'title', 'default', and 'examples' with Effect-TS Schemas. These annotations enrich the schema with metadata for better readability and information. It imports Schema and JSONSchema from 'effect' and generates a JSON schema for a string type.

```typescript
import { JSONSchema, Schema } from "effect"

const schema = Schema.String.annotations({
  description: "my custom description",
  title: "my custom title",
  default: "",
  examples: ["a", "b"]
})

const jsonSchema = JSONSchema.make(schema)

console.log(JSON.stringify(jsonSchema, null, 2))
```

---

### Encode/Decode HashMapFromSelf with Effect-TS Schema

Source: https://effect.website/docs/schema/effect-data-types

This snippet demonstrates Schema.HashMapFromSelf for direct HashMap transformations. It decodes a HashMap of stringified numbers into a HashMap of actual numbers and encodes vice-versa. The examples display the resulting HashMaps after decoding and encoding operations.

```typescript
import { Schema } from "effect"
import { HashMap } from "effect"

const schema = Schema.HashMapFromSelf({
  key: Schema.String,
  value: Schema.NumberFromString
})

// ┌─── HashMap
// ▼
type Encoded = typeof schema.Encoded

// ┌─── HashMap
// ▼
type Type = typeof schema.Type

const decode = Schema.decodeUnknownSync(schema)
const encode = Schema.encodeSync(schema)

// Decoding examples

console.log(
  decode(
    HashMap.fromIterable([
      ["a", "2"],
      ["b", "2"],
      ["c", "3"]
    ])
  )
)
// Output: { _id: 'HashMap', values: [ [ 'a', 2 ], [ 'c', 3 ], [ 'b', 2 ] ] }

// Encoding examples

console.log(
  encode(
    HashMap.fromIterable([
      ["a", 1],
      ["b", 2],
      ["c", 3]
    ])
  )
)
// Output: { _id: 'HashMap', values: [ [ 'a', 1 ], [ 'c', 3 ], [ 'b', 2 ] ] }
```

---

### Generate Arbitrary Data from Effect Schema

Source: https://effect.website/docs/schema/arbitrary

This snippet demonstrates how to use `Arbitrary.make` to create an `Arbitrary` instance from an Effect `Schema` and then generate random samples using `fast-check.sample`. It defines a `Person` schema with constraints on `name` and `age` and shows example output.

```typescript
import { Arbitrary, FastCheck, Schema } from "effect"

// Define a Person schema with constraints
const Person = Schema.Struct({
  name: Schema.NonEmptyString,
  age: Schema.Int.pipe(Schema.between(1, 80))
})

// Create an Arbitrary based on the schema
const arb = Arbitrary.make(Person)

// Generate random samples from the Arbitrary
console.log(FastCheck.sample(arb, 2))
/*
Example Output:
[ { name: 'q r', age: 3 }, { name: '&|', age: 6 } ]
*/
```

---

### Reporting Multiple Validation Errors with Effect Schema

Source: https://effect.website/docs/schema/filters

Demonstrates how the Schema.filter API in Effect can report multiple validation issues simultaneously. This is beneficial for form validation where several checks might fail at once. The example defines a form with multiple fields and applies filters.

```typescript
import { Either, Schema, ParseResult } from "effect"

const Password = Schema.Trim.pipe(Schema.minLength(2))
const OptionalString = Schema.optional(Schema.String)

const MyForm = Schema.Struct({
  password: Password,
  confirm_password: Password,
  name: OptionalString,
  surname: OptionalString
}).pipe(
  Schema.filter
)
```

---

### Generate JSON Schema from Effect Struct

Source: https://effect.website/docs/schema/json-schema

This example demonstrates how to create a JSON Schema from an Effect Schema representing a 'Person' struct. The `JSONSchema.make` function takes the struct schema and generates a corresponding JSON Schema object. The output is then stringified for display.

```typescript
import { JSONSchema, Schema } from "effect"

const Person = Schema.Struct({
  name: Schema.String,
  age: Schema.Number
})

const jsonSchema = JSONSchema.make(Person)

console.log(JSON.stringify(jsonSchema, null, 2))
```

---

### Applying Filters to Schema Constraints for Arbitrary Generation

Source: https://effect.website/docs/schema/arbitrary

This example shows how Effect translates schema constraints, like `Schema.between`, into corresponding `fast-check` primitives. For instance, `Schema.Int.pipe(Schema.between(1, 80))` is mapped to `FastCheck.integer({ min: 1, max: 80 })` during arbitrary generation.

```typescript
import { Schema, FastCheck } from "effect"

// Schema with a numerical constraint
const constrainedInt = Schema.Int.pipe(Schema.between(1, 80))

// When Arbitrary.make is used with this schema, it will utilize:
// FastCheck.integer({ min: 1, max: 80 })

// Example of generating a value using the underlying primitive:
const arbInt = Arbitrary.make(constrainedInt)
console.log(FastCheck.sample(arbInt, 1))
```

---

### Effect Schema: Validate Class Instance Fields

Source: https://effect.website/docs/schema/basic-usage

This example shows how to validate the fields of a class instance using Effect's Schema. It combines instance validation with additional checks on the instance's fields using `Schema.filter` and `ParseResult.validateEither` to ensure specific field constraints are met.

```typescript
import { Either, ParseResult, Schema } from "effect"

class MyData {
  constructor(readonly name: string) {}
}

const MyDataFields = Schema.Struct({
  name: Schema.NonEmptyString
})

const MyDataSchema = Schema.instanceOf(MyData).pipe(
  Schema.filter((a, options) => 
    ParseResult.validateEither(MyDataFields)(a, options).pipe(
      Either.flip,
      Either.getOrUndefined
    )
  )
)

// Example: Valid instance
console.log(Schema.validateSync(MyDataSchema)(new MyData("John")))
// Output: MyData { name: 'John' }

// Example: Invalid instance (empty name)
console.log(Schema.validateSync(MyDataSchema)(new MyData("")))
/* 
throws:
ParseError: { MyData | filter }
└─ Predicate refinement failure
└─ { readonly name: NonEmptyString }
└─ ["name"]
└─ NonEmptyString
└─ Predicate refinement failure
└─ Expected a non empty string, actual ""
*/
```

---

### Customize Error Handling with parseOptions in Effect Schema

Source: https://effect.website/docs/schema/getting-started

Demonstrates how to use parseOptions within schema annotations to control error reporting. This allows for granular control over whether the first or all errors are reported for a given schema or sub-schema. The example shows a main schema configured for 'all' errors and a nested schema configured for 'first' errors.

```typescript
import { Schema } from "effect"
import { Either } from "effect"

const schema = Schema.Struct({
  a: Schema.Struct({
    b: Schema.String,
    c: Schema.String
  }).annotations({
    title: "first error only",
    // Limit errors to the first in this sub-schema
    parseOptions: { errors: "first" }
  }),
  d: Schema.String
}).annotations({
  title: "all errors",
  // Capture all errors for the main schema
  parseOptions: { errors: "all" }
})

// Decode input with custom error-handling behavior
const result = Schema.decodeUnknownEither(schema)(
  {
    a: {}
  },
  {
    errors: "first"
  }
)

if (Either.isLeft(result)) {
  console.log(result.left.message)
}
/* 
all errors 
├─ ["a"]
│ └─ first error only
│ └─ ["b"]
│ └─ is missing
└─ ["d"]
└─ is missing
*/
```

---

### Define Optional Number Field in Effect Schema

Source: https://effect.website/docs/schema/advanced-usage

This code defines a Product schema with an optional 'quantity' field using Schema.optional. It illustrates decoding and encoding examples for cases where the quantity is provided, omitted, or set to undefined.

```typescript
import { Schema } from "effect"

const Product = Schema.Struct({
  quantity: Schema.optional(Schema.NumberFromString)
})

// Decoding examples
console.log(Schema.decodeUnknownSync(Product)({" quantity": "1" }))
// Output: { quantity: 1 }
console.log(Schema.decodeUnknownSync(Product)({}))
// Output: {}
console.log(Schema.decodeUnknownSync(Product)({" quantity": undefined }))
// Output: { quantity: undefined }

// Encoding examples
console.log(Schema.encodeSync(Product)({" quantity": 1 }))
// Output: { quantity: "1" }
console.log(Schema.encodeSync(Product)({}))
// Output: {}
console.log(Schema.encodeSync(Product)({" quantity": undefined }))
// Output: { quantity: undefined }
```

---

### Define and Use PositiveInt Type (TypeScript)

Source: https://effect.website/docs/code-style/branded-types

This snippet defines a custom type `PositiveInt` using `Brand.Brand.FromConstructor`. It illustrates how to create instances of this type with valid positive integers and shows examples that would throw errors for negative or non-integer values.

```typescript
import { Brand } from 'effect/Brand'

type PositiveInt = Brand.Brand<number, 'PositiveInt'>

const PositiveInt = Brand.fromConstructor<PositiveInt>(
  (n: number) => {
    if (n <= 0) {
      throw new Error(`Expected ${n} to be positive`)
    }
    if (!Number.isInteger(n)) {
      throw new Error(`Expected ${n} to be an integer`)
    }
    return n
  }
)

// Usage example

// Valid positive integer
const good: PositiveInt = PositiveInt(10)

// throws [ { message: 'Expected -5 to be positive' } ]
// const bad1: PositiveInt = PositiveInt(-5)

// throws [ { message: 'Expected 3.14 to be an integer' } ]
// const bad2: PositiveInt = PositiveInt(3.14)
```

---

### Declare File Schema for Arbitrary Generation (TypeScript)

Source: https://effect.website/docs/schema/advanced-usage

Shows an example of declaring a schema for the `File` type intended for use with the `Arbitrary` compiler. This requires an `arbitrary` annotation, which is missing in this snippet, leading to an error. This highlights the need for compiler-specific annotations when defining custom schemas.

```typescript
import { Arbitrary, Schema } from "effect"

// Define a schema for the File type
const FileFromSelf = Schema.declare(
  (input: unknown): input is File => input instanceof File,
  {
    identifier: "FileFromSelf"
  }
)

// Try creating an Arbitrary instance for the schema
const arb = Arbitrary.make(FileFromSelf)
/*
throws:
Error: Missing annotation 
       details: Generating an Arbitrary for this schema requires an "arbitrary" ann
```

---

### Create and Use an Assertion with Effect Schema

Source: https://effect.website/docs/schema/getting-started

Shows how to create an assertion function using Schema.asserts. This function validates an input against a schema and throws a detailed ParseError if the input does not conform. It's useful for runtime validation where non-conforming data should halt execution. The example defines a Person schema and uses Schema.asserts to create an `assertsPerson` function.

```typescript
import { Schema } from "effect"

// Define a schema for a Person object
const Person = Schema.Struct({
  name: Schema.String,
  age: Schema.Number
})

// Generate an assertion function from the schema
const assertsPerson: Schema.Schema.ToAsserts = Schema.asserts(Person)

try {
  // Attempt to assert that the input matches the Person schema
  assertsPerson({ name: "Alice", age: "30" })
} catch (e) {
  console.error("The input does not match the schema:")
  console.error(e)
}
/* 
throws:
The input does not match the schema:
{
  _id: 'ParseError',
  message: '{ readonly name: string; readonly age: number }\n' +
    '└─ ["age"]\n' +
    ' └─ Expected number, actual "30"'
}
*/

// This input matches the schema and will not throw an error
assertsPerson({ name: "Alice", age: 30 })
```

---

### Clamp Number within Range using Schema.transform (Strict with Type Assertion)

Source: https://effect.website/docs/schema/transformations

This example demonstrates clamping a number within a specified range using Schema.transform with strict type checking. To resolve the potential type mismatch issue where the clamped number might not strictly conform to the original type, a type assertion `as A` is used in the decode function. The encode function returns the value unchanged.

```typescript
import { Schema, Number } from "effect"

const clamp = <A extends number>( 
  minimum: number,
  maximum: number
) =>
(self: Schema.Schema<A, A>) =>
  Schema.transform<A, number>(
    // Source schema
    self,
    // Target schema: filter based on min/max range
    self.pipe(
      Schema.typeSchema,
      Schema.filter((a) => a <= minimum || a >= maximum)
    ),
    {
      strict: true,
      // Clamp the number within the specified range and assert type
      decode: (a) => Number.clamp(a, { minimum, maximum }) as A,
      encode: (a) => a
    }
  )
```

---

### Merging JSON Schema Annotations from Multiple Refinements in Effect

Source: https://effect.website/docs/schema/json-schema

Illustrates how to combine JSON Schema annotations from multiple `Schema.filter` refinements in Effect. Each refinement can add its own `jsonSchema` fragment, which are then merged into the final generated schema. This example shows merging `minimum` and `maximum` constraints.

```typescript
import { JSONSchema, Schema } from "effect"

// Define a schema with a refinement for positive numbers
const Positive = Schema.Number.pipe(
  Schema.filter((n) => n > 0, {
    jsonSchema: { minimum: 0 }
  })
)

// Add an upper bound refinement to the schema
const schema = Positive.pipe(
  Schema.filter((n) => n <= 10, {
    jsonSchema: { maximum: 10 }
  })
)

const jsonSchema = JSONSchema.make(schema)

console.log(JSON.stringify(jsonSchema, null, 2))
```

---

### Combine Multiple Branded Types (TypeScript)

Source: https://effect.website/docs/code-style/branded-types

This example demonstrates how to combine multiple branded types using `Brand.all` in Effect-TS. It defines `Int` and `Positive` branded types and then uses `Brand.all` to create a new constructor `PositiveInt` that enforces both integer and positive value constraints.

```typescript
import { Brand } from "effect"

type Int = number & Brand.Brand<"Int">

const Int = Brand.refined<number, "Int">( 
  (n) => Number.isInteger(n),
  (n) => Brand.error(`Expected ${n} to be an integer`)
)

type Positive = number & Brand.Brand<"Positive">

const Positive = Brand.refined<number, "Positive">( 
  (n) => n > 0,
  (n) => Brand.error(`Expected ${n} to be positive`)
)

// Combine the Int and Positive constructors 
// into a new branded constructor PositiveInt
const PositiveInt = Brand.all(Int, Positive)

// Extract the branded type from the PositiveInt constructor
type PositiveIntType = Brand.TypeOf<typeof PositiveInt>;

// Example usage:
// const validPositiveInt: PositiveIntType = PositiveInt(5);
// const invalidPositiveInt: PositiveIntType = PositiveInt(-2);
// const invalidFloat: PositiveIntType = PositiveInt(3.14);

```

---

### Type Safety with Branded Identifiers (TypeScript)

Source: https://effect.website/docs/code-style/branded-types

This example illustrates the type safety provided by branded identifiers in TypeScript. It shows how attempting to assign a raw number or a `UserId` to a `ProductId` parameter results in a compile-time error, enforcing correct usage and preventing bugs.

```typescript
import { Brand } from "effect"

type UserId = number & Brand.Brand<"UserId">
const UserId = Brand.nominal<number, "UserId">()

const getUserById = (id: UserId) => {
  // Logic to retrieve user
}

type ProductId = number & Brand.Brand<"ProductId">
const ProductId = Brand.nominal<number, "ProductId">()

const getProductById = (id: ProductId) => {
  // Logic to retrieve product
}

// Correct usage
getProductById(ProductId(1))

// Incorrect, will result in an error
// getProductById(1)
// Error ts(2345) ― Argument of type 'number' is not assignable to parameter of type 'ProductId'.

// Also incorrect, will result in an error
// getProductById(UserId(1))
// Error ts(2345) ― Argument of type 'UserId' is not assignable to parameter of type 'ProductId'.

```

---

### Enforcing Type Safety with Branded Types in TypeScript

Source: https://effect.website/docs/code-style/branded-types

This example illustrates the type safety provided by branded types. It shows that a function expecting a `ProductId` will reject a plain `number` (or a `UserId` without the correct brand), as indicated by the TypeScript compile-time error.

```typescript
const BrandTypeId: unique symbol = Symbol.for("effect/Brand")

type ProductId = number & {
  readonly [BrandTypeId]: {
    readonly ProductId: "ProductId"
  }
}

const getProductById = (id: ProductId) => {
  // Logic to retrieve product
}

type UserId = number

const id: UserId = 1

getProductById(id)
// Error ts(2345) ― Argument of type 'number' is not assignable to parameter of type 'ProductId'.
// Type 'number' is not assignable to type '{ readonly [BrandTypeId]: { readonly ProductId: "ProductId"; }; }'.
// Type 'number' is not assignable to type '{ readonly [BrandTypeId]: { readonly ProductId: "ProductId"; }; }'.
```

---

### Create and Use a Type Guard with Effect Schema

Source: https://effect.website/docs/schema/getting-started

Illustrates how to generate a type guard function using Schema.is. This function can be used at runtime to check if an unknown value conforms to a defined schema, effectively acting as a type guard for TypeScript. The example defines a Person schema and then uses Schema.is to create an `isPerson` type guard.

```typescript
import { Schema } from "effect"

// Define a schema for a Person object
const Person = Schema.Struct({
  name: Schema.String,
  age: Schema.Number
})

// Generate a type guard from the schema
const isPerson = Schema.is(Person)

// Test the type guard with various inputs
console.log(isPerson({ name: "Alice", age: 30 }))
// Output: true

console.log(isPerson(null))
// Output: false

console.log(isPerson({}))
// Output: false
```

---

### Customize JSON Schema for Unsupported Types (Effect-TS)

Source: https://effect.website/docs/schema/json-schema

This example demonstrates how to handle unsupported types like 'bigint' during JSON Schema generation with Effect-TS. When a type lacks native JSON Schema support, an error occurs. This snippet shows how to resolve this by providing a custom 'jsonSchema' annotation to define the desired representation.

```typescript
import { JSONSchema, Schema } from "effect"

const schema = Schema.Struct({
  // Adding a custom JSON Schema annotation for the `bigint` type
  a_bigint_field: Schema.BigIntFromSelf.annotations({
    jsonSchema: {
      type: "some cus"
```

---

### Specifying Error Paths for Form Validation in Effect Schema

Source: https://effect.website/docs/schema/filters

Illustrates how to associate specific error messages with particular fields or paths when validating structured data, such as forms, using Effect's Schema.filter. This example ensures that password fields match and specifies the error path for the 'confirm_password' field.

```typescript
import { Either, Schema, ParseResult } from "effect"

const Password = Schema.Trim.pipe(Schema.minLength(2))

const MyForm = Schema.Struct({
  password: Password,
  confirm_password: Password
}).pipe(
  // Add a filter to ensure that passwords match
  Schema.filter((input) => {
    if (input.password !== input.confirm_password) {
      // Return an error message associated
      // with the "confirm_password" field
      return {
        path: ["confirm_password"],
        message: "Passwords do not match"
      }
    }
  })
)

console.log(
  JSON.stringify(
    Schema.decodeUnknownEither(MyForm)({
      password: "abc",
      confirm_password: "abd" // Confirm password does not match
    }).pipe(
      Either.mapLeft((error) =>
        ParseResult.ArrayFormatter.formatErrorSync(error)
      )
    ),
    null,
    2
  )
)
/*
"_id": "Either",
"_tag": "Left",
"left": [
  {
    "_tag": "Type",
    "path": [
      "confirm_password"
    ],
    "message": "Passwords do not match"
  }
]
}
*/
```

---

### Define String Length Filter with Schema.filter

Source: https://effect.website/docs/schema/filters

This example demonstrates how to create a custom filter for a string schema using `Schema.filter`. It ensures that the input string is at least 10 characters long, providing a specific error message if the condition is not met. The filter adds a validation constraint without altering the original schema's type.

```typescript
import { Schema } from "effect"

// Define a string schema with a filter to ensure the string
// is at least 10 characters long
const LongString = Schema.String.pipe(
  Schema.filter(
    // Custom error message for strings shorter than 10 characters
    (s) => s.length >= 10 || "a string at least 10 characters long"
  )
)

// ┌─── string
// ▼
type Type = typeof LongString.Type

console.log(Schema.decodeUnknownSync(LongString)("a"))
/* 
throws: 
ParseError: { string | filter } 
└─ Predicate refinement failure 
└─ a string at least 10 characters long 
*/
```

---

### Define a Schema Class Without Fields (TypeScript)

Source: https://effect.website/docs/schema/classes

Shows how to define a schema class `NoArgs` using `Schema.Class` when no fields are required. It demonstrates creating instances using the default constructor and by explicitly passing an empty object.

```typescript
import { Schema } from "effect"

// Define a class with no fields
class NoArgs extends Schema.Class("NoArgs")({})

// Create an instance using the default constructor
const noargs1 = new NoArgs()

// Alternatively, create an instance by explicitly passing an empty object
const noargs2 = new NoArgs({})

```

---

### Importing Effect Schema Module

Source: https://effect.website/docs/schema/getting-started

Demonstrates how to import the Schema module from the 'effect' package, supporting both namespace and named imports.

```typescript
import * as Schema from "effect/Schema"

```

```typescript
import { Schema } from "effect"

```

---

### Extract Keys from Object Schema with Schema.keyof

Source: https://effect.website/docs/schema/basic-usage

The Schema.keyof operation generates a schema that represents the keys of an input object schema. This is helpful when you need to work with the names of properties within a schema, for example, to create a schema for a set of allowed property names. It takes an object schema and returns a new schema representing its keys.

```typescript
import { Schema } from "effect"

const schema = Schema.Struct({
  a: Schema.String,
  b: Schema.Number
})

const keys = Schema.keyof(schema)

// ┌─── "a" | "b"
// ▼
type Type = typeof keys.Type
```

---

### Handle Type Inference Error in Recursive Schemas

Source: https://effect.website/docs/schema/advanced-usage

Illustrates a common TypeScript error (ts(7022) and ts(7024)) that occurs when defining recursive schemas without explicit type annotations. This example shows the problematic code that leads to implicit 'any' types and incorrect inference.

```typescript
import { Schema } from "effect"

const Category = Schema.Struct({
  // Error ts(7022) — 'Category' implicitly has type 'any' because it does not have a type annotation and is referenced directly or indirectly in its own initializer.
  name: Schema.String,
  subcategories: Schema.Array(Schema.suspend(() => Category))
  // Error ts(7024) — Function implicitly has return type 'any' because it does not have a return type annotation and is referenced directly or indirectly in one of its return expressions.
})
```

---

### Annotate Struct Properties for JSON Schema (Effect-TS)

Source: https://effect.website/docs/schema/json-schema

This example shows how to add annotations directly to struct property signatures in Effect-TS for more semantically accurate JSON Schema generation. This approach links metadata like 'title' specifically to the properties they describe. It generates a JSON schema for a 'Person' struct with 'firstName' and 'lastName' properties.

```typescript
import { JSONSchema, Schema } from "effect"

const Person = Schema.Struct({
  firstName: Schema.propertySignature(Schema.String).annotations({
    title: "First name"
  }),
  lastName: Schema.propertySignature(Schema.String).annotations({
    title: "Last Name"
  })
})

const jsonSchema = JSONSchema.make(Person)

console.log(JSON.stringify(jsonSchema, null, 2))
```

---

### Add Custom Error Message to String Schema (Effect)

Source: https://effect.website/docs/schema/error-messages

Demonstrates how to define a string schema with a custom error message using `Schema.String.annotations({ message: ... })`. This custom message replaces the default error when decoding fails. The example shows decoding `null` against a string schema with and without a custom message.

```typescript
import { Schema } from "effect"

// Define a string schema without a custom message
const MyString = Schema.String

// Attempt to decode `null`, resulting in a default error message
// Schema.decodeUnknownSync(MyString)(null)
/* 
throws:
ParseError: Expected string, actual null 
*/

// Define a string schema with a custom error message
const MyStringWithMessage = Schema.String.annotations({
  message: () => "not a string"
})

// Decode with the custom schema, showing the new error message
// Schema.decodeUnknownSync(MyStringWithMessage)(null)
/* 
throws:
ParseError: not a string 
*/
```

---

### Define Person Schema and Generate Arbitrary Data with FastCheck (TypeScript)

Source: https://effect.website/docs/schema/arbitrary

This snippet defines a 'Person' schema including 'Name' and a constrained 'Age' (between 1 and 80). It then uses 'Arbitrary.make' to create an arbitrary generator for the 'Person' schema and samples it using 'FastCheck.sample'. The example output shows randomly generated person objects.

```typescript
const Age = Schema.Int.pipe(Schema.between(1, 80))
const Person = Schema.Struct({
  name: Name,
  age: Age
})

const arb = Arbitrary.make(Person)

console.log(FastCheck.sample(arb, 2))
```

---

### Generate Recursive JSON Schema with Identifier (Effect-TS)

Source: https://effect.website/docs/schema/json-schema

This snippet illustrates how to create and generate JSON Schemas for recursive data structures using Effect-TS. It highlights the mandatory use of 'identifier' annotations to ensure correct self-references within the generated schema. The example defines a 'Category' schema that can contain an array of other categories.

```typescript
import { JSONSchema, Schema } from "effect"

// Define the interface representing a category structure 
interface Category {
  readonly name: string
  readonly categories: ReadonlyArray<Category>
}

// Define a recursive schema with a required identifier annotation 
const Category = Schema.Struct({
  name: Schema.String,
  categories: Schema.Array(
    // Recursive reference to the Category schema 
    Schema.suspend((): Schema.Schema => Category)
  )
}).annotations({ identifier: "Category" })

const jsonSchema = JSONSchema.make(Category)

console.log(JSON.stringify(jsonSchema, null, 2))
```

---

### Using Identifiers for Schema Clarity (TypeScript)

Source: https://effect.website/docs/schema/error-messages

This snippet demonstrates how to use the 'identifier' annotation to provide clear names for schema components like 'Name', 'Age', and 'Person'. This improves the readability of error messages when decoding fails, making it easier to pinpoint the source of the issue. It shows examples of decoding null, an empty object, and an object with null values, all resulting in specific 'ParseError' messages.

```typescript
import { Schema } from "effect"

const Name = Schema.String.annotations({ identifier: "Name" })

const Age = Schema.Number.annotations({ identifier: "Age" })

const Person = Schema.Struct({
    name: Name,
    age: Age
}).annotations({ identifier: "Person" })

// Example of decoding null
// Schema.decodeUnknownSync(Person)(null)
/* 
throws:
ParseError: Expected Person, actual null 
*/ 

// Example of decoding an empty object
// Schema.decodeUnknownSync(Person)({}, { errors: "all" })
/* 
throws:
ParseError: Person 
└─ ["name"]
   └─ is missing
└─ ["age"]
   └─ is missing
*/ 

// Example of decoding an object with null values
// Schema.decodeUnknownSync(Person)(
//     { name: null, age: null },
//     { errors: "all" }
// )
/* 
throws:
ParseError: Person 
├─ ["name"]
│  └─ Expected Name, actual null
└─ ["age"]
   └─ Expected Age, actual null
*/
```

---

### Configure Optional Field with Exactness as Option in TypeScript

Source: https://effect.website/docs/schema/advanced-usage

This example uses optionalWith with exact: true to ensure Option.none() only on missing fields, treating undefined as an error. Requires the Effect library for parsing, handling string-to-number conversion, and throwing ParseError for undefined inputs. Outputs Option types or errors, limited by strict exactness that disallows undefined values.

```typescript
import { Schema } from "effect"

const Product = Schema.Struct({
  quantity: Schema.optionalWith(Schema.NumberFromString, {
    as: "Option",
    exact: true
  })
})

// ┌─── { readonly quantity?: string; }
// ▼
type Encoded = typeof Product.Encoded

// ┌─── { readonly quantity: Option; }
// ▼
type Type = typeof Product.Type

console.log(Schema.decodeUnknownSync(Product)({}))
// Output: { quantity: { _id: 'Option', _tag: 'None' } }

console.log(Schema.decodeUnknownSync(Product)({ quantity: "2" }))
// Output: { quantity: { _id: 'Option', _tag: 'Some', value: 2 } }

console.log(Schema.decodeUnknownSync(Product)({ quantity: undefined }))
/*
throws:
ParseError: (Struct (Encoded side) <-> Struct (Type side))
└─ Encoded side transformation failure
└─ Struct (Encoded side)
└─ ["quantity"]
└─ NumberFromString
└─ Encoded side transformation failure
└─ Expected string, actual undefined
*/
```

---

### Create Branded Values with Effect Schema

Source: https://effect.website/docs/schema/default-constructors

Shows how to use `Schema.brand` to add metadata to a value, creating a branded type in Effect Schema. It illustrates successful branding and how validation rules defined before branding are still enforced.

```typescript
import { Schema } from "effect"

const BrandedNumberSchema = Schema.Number.pipe(
  Schema.between(1, 10),
  Schema.brand("MyNumber")
)

// Successful creation
const n = BrandedNumberSchema.make(5)

// This will throw an error because the number is outside the valid range
BrandedNumberSchema.make(20)
/* 
throws 
ParseError: a number between 1 and 10 & Brand<"MyNumber">
└─ Predicate refinement failure
└─ Expected a number between 1 and 10 & Brand<"MyNumber">, actual 20
*/

// Bypasses validation
BrandedNumberSchema.make(20, { disableValidation: true })
```

---

### Custom Message on Last Refinement in Chain (Effect)

Source: https://effect.website/docs/schema/error-messages

Demonstrates applying a custom error message to the last refinement in a chain of schema refinements using `pipe` and `annotations`. The custom message is only triggered if the final refinement (`maxLength` in this case) fails. The example includes scenarios for different decoding failures (`null`, empty string, string exceeding max length).

```typescript
import { Schema } from "effect"

const MyString = Schema.String.pipe(
  Schema.minLength(1),
  Schema.maxLength(2)
).annotations({
  // This message is displayed only if the last filter (`maxLength`) fails
  message: () => "my custom message"
})

const decode = Schema.decodeUnknownSync(MyString)

try {
  decode(null)
} catch (e: any) {
  console.log(e.message)
  /* 
  minLength(1) & maxLength(2) 
  └─ From side refinement failure 
  └─ minLength(1) 
  └─ From side refinement failure 
  └─ Expected string, actual null 
  */
}

try {
  decode("")
} catch (e: any) {
  console.log(e.message)
  /* 
  minLength(1) & maxLength(2) 
  └─ From side refinement failure 
  └─ minLength(1) 
  └─ Predicate refinement failure 
  └─ Expected a string at least 1 character(s) long, actual "" 
  */
}

try {
  decode("abc")
} catch (e: any) {
  console.log(e.message)
  // "my custom message"
}
```

---

### Handling Refinement Errors in Effect Schemas (TypeScript)

Source: https://effect.website/docs/schema/error-messages

This example illustrates how Effect Schemas report errors when data fails refinement checks. It distinguishes between 'From side failure', occurring when the initial data type is incorrect (e.g., null instead of string), and 'Predicate refinement failure', where the data conforms to the initial type but violates the refinement condition (e.g., an empty string when a non-empty string is required).

```typescript
import { Schema } from "effect"

const Name = Schema.NonEmptyString.annotations({ identifier: "Name" })

const Age = Schema.Positive.pipe(Schema.int({ identifier: "Age" }))

const Person = Schema.Struct({
    name: Name,
    age: Age
}).annotations({ identifier: "Person" })

// From side failure
// Schema.decodeUnknownSync(Person)({ name: null, age: 18 })
/* 
throws:
ParseError: Person
└─ ["name"]
   └─ Name
   └─ From side refinement failure
   └─ Expected string, actual null
*/ 

// Predicate refinement failure
// Schema.decodeUnknownSync(Person)({ name: "", age: 18 })
/* 
throws:
ParseError: Person
└─ ["name"]
   └─ Name
   └─ Predicate refinement failure
   └─ Expected a non empty string, actual ""
*/
```

---

### Create Struct Instances with Effect Schema

Source: https://effect.website/docs/schema/default-constructors

Demonstrates how to create instances of a struct schema using the make function. Shows successful creation and validation failure when constraints are violated. Includes ParseError details for empty string validation failures.

```typescript
import { Schema } from "effect"

const Struct = Schema.Struct({
  name: Schema.NonEmptyString
})

// Successful creation
Struct.make({ name: "a" })

// This will throw an error because the name is empty
Struct.make({ name: "" })
/*
throws
ParseError: { readonly name: NonEmptyString }
└─ ["name"]
  └─ NonEmptyString
    └─ Predicate refinement failure
      └─ Expected NonEmptyString, actual ""
*/
```

---

### Custom Error Message for Union Schema with Override (Effect)

Source: https://effect.website/docs/schema/error-messages

Illustrates how to apply a custom error message to a union schema using `Schema.Union(...).annotations({ message: ... })`. The `override: true` option ensures this custom message replaces all nested or default error messages when decoding fails. The example decodes `null` against a union of string and number.

```typescript
import { Schema } from "effect"

// Define a union schema without a custom message
const MyUnion = Schema.Union(Schema.String, Schema.Number)

// Decode `null`, resulting in default union error messages
// Schema.decodeUnknownSync(MyUnion)(null)
/* 
throws:
ParseError: string | number 
├─ Expected string, actual null 
└─ Expected number, actual null 
*/

// Define a union schema with a custom message and override flag
const MyUnionWithMessage = Schema.Union(
  Schema.String,
  Schema.Number
).annotations({
  message: () => ({
    message: "Please provide a string or a number",
    // Ensures this message replaces all nested messages
    override: true
  })
})

// Decode with the custom schema, showing the new error message
// Schema.decodeUnknownSync(MyUnionWithMessage)(null)
/* 
throws:
ParseError: Please provide a string or a number 
*/
```
