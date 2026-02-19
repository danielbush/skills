# Effect – Collections & Data Structures

`HashSet`, `HashMap`, `MutableHashSet`, `Chunk`, `Order`, `Data`, and `BigDecimal`.

---

### HashSet - Operations and Examples

Source: https://effect.website/docs/data-types/hash-set

Details the various operations available for HashSet, including construction, element management, set operations, and transformations, with practical code examples.

```APIDOC
## HashSet - Operations and Examples

This section outlines the common operations for `HashSet` and provides illustrative examples.

### Operations

| Category      | Operation    | Description                                     | Time Complexity |
|---------------|--------------|-------------------------------------------------|-----------------|
| Constructors  | `empty`      | Creates an empty HashSet                        | O(1)            |
| Constructors  | `fromIterable`| Creates a HashSet from an iterable              | O(n)            |
| Constructors  | `make`       | Creates a HashSet from multiple values          | O(n)            |
| Elements      | `has`        | Checks if a value exists in the set             | O(1) avg        |
| Elements      | `some`       | Checks if any element satisfies a predicate     | O(n)            |
| Elements      | `every`      | Checks if all elements satisfy a predicate      | O(n)            |
| Elements      | `isSubset`   | Checks if a set is a subset of another          | O(n)            |
| Getters       | `values`     | Gets an `Iterator` of all values                | O(1)            |
| Getters       | `toValues`   | Gets an `Array` of all values                   | O(n)            |
| Getters       | `size`       | Gets the number of elements                     | O(1)            |
| Mutations     | `add`        | Adds a value to the set (returns new set)       | O(1) avg        |
| Mutations     | `remove`     | Removes a value from the set (returns new set)  | O(1) avg        |
| Mutations     | `toggle`     | Toggles a value’s presence (returns new set)    | O(1) avg        |
| Operations    | `difference` | Computes set difference (A - B)                 | O(n)            |
| Operations    | `intersection`| Computes set intersection (A ∩ B)               | O(n)            |
| Operations    | `union`      | Computes set union (A ∪ B)                      | O(n)            |
| Mapping       | `map`        | Transforms each element (returns new set)       | O(n)            |
| Sequencing    | `flatMap`    | Transforms and flattens elements (returns new set)| O(n)            |
| Traversing    | `forEach`    | Applies a function to each element              | O(n)            |
| Folding       | `reduce`     | Reduces the set to a single value               | O(n)            |
| Filtering     | `filter`     | Keeps elements that satisfy a predicate (returns new set)| O(n)            |
| Partitioning  | `partition`  | Splits into two sets by a predicate (returns new sets)| O(n)            |

### Example (Basic Creation and Operations)

```typescript
import { HashSet } from "effect"

// Create an initial set with 3 values
const set1 = HashSet.make(1, 2, 3)

// Add a value (returns a new set)
const set2 = HashSet.add(set1, 4)

// The original set is unchanged
console.log(HashSet.toValues(set1)) // Output: [1, 2, 3]
console.log(HashSet.toValues(set2)) // Output: [1, 2, 3, 4]

// Perform set operations with another set
const set3 = HashSet.make(3, 4, 5)

// Combine both sets
const union = HashSet.union(set2, set3)
console.log(HashSet.toValues(union)) // Output: [1, 2, 3, 4, 5]

// Shared values
const intersection = HashSet.intersection(set2, set3)
console.log(HashSet.toValues(intersection)) // Output: [3, 4]

// Values only in set2
const difference = HashSet.difference(set2, set3)
console.log(HashSet.toValues(difference)) // Output: [1, 2]
```

### Example (Chaining with `pipe`)

```typescript
import { HashSet, pipe } from "effect"

const result = pipe(
  // Duplicates are ignored
  HashSet.make(1, 2, 2, 3, 4, 5, 5),
  // Keep even numbers
  HashSet.filter((n) => n % 2 === 0),
  // Double each value
  HashSet.map((n) => n * 2),
  // Convert to array
  HashSet.toValues
)

console.log(result) // Output: [4, 8]
```
```

---

### HashSet - Core Concepts and Usage

Source: https://effect.website/docs/data-types/hash-set

Demonstrates the core concepts of HashSet, including value equality, immutability, and usage with collections like HashSet.

```APIDOC
## HashSet - Core Concepts and Usage

This example illustrates how `HashSet` in Effect-TS handles value equality and immutability.

### Description

`HashSet` ensures that only unique values are stored based on their content, even if they are different object instances. Operations on `HashSet` return new instances, leaving the original unchanged.

### Example (Value Equality and Immutability)

```typescript
import { Equal, HashSet, pipe } from "effect"

// Using Schema.Data for structured data
import { Schema } from "effect"

const PersonSchema = Schema.Data(
  Schema.Struct({
    id: Schema.Number,
    name: Schema.String,
    age: Schema.Number
  })
)

const Person = Schema.decodeSync(PersonSchema)

const person1 = Person({ id: 1, name: "Alice", age: 30 })
const person2 = Person({ id: 1, name: "Alice", age: 30 })

// They are different object references
console.log(Object.is(person1, person2)) // Output: false

// But they are equal in value (based on content)
console.log(Equal.equals(person1, person2)) // Output: true

// Add both to a HashSet — only one will be stored
const set = pipe(
  HashSet.empty<typeof person1>(),
  HashSet.add(person1),
  HashSet.add(person2)
)

console.log(HashSet.size(set)) // Output: 1
```

### Key Features of HashSet

- **Immutable**: Operations return new sets.
- **Unique Values**: Stores only distinct values based on `Equal.equals`.
- **Efficient Operations**: Supports fast lookups, insertions, and deletions.
```

---

### Chaining HashSet Operations with pipe

Source: https://effect.website/docs/data-types/hash-set

Demonstrates a functional approach to manipulating an immutable HashSet using the `pipe` function. This example shows how to chain multiple operations like filtering, mapping, and converting to an array, processing data immutably.

```typescript
import { HashSet, pipe } from "effect"

const result = pipe(
  // Duplicates are ignored
  HashSet.make(1, 2, 2, 3, 4, 5, 5),
  // Keep even numbers
  HashSet.filter((n) => n % 2 === 0),
  // Double each value
  HashSet.map((n) => n * 2),
  // Convert to array
  HashSet.toValues
)

console.log(result)
// Output: [4, 8]
```

---

### Reverse Order Comparison with Effect-TS Order

Source: https://effect.website/docs/behaviour/order

Shows how to invert the sorting direction of an existing `Order` using `Order.reverse`. This example creates a descending order from a default ascending `Order.number`.

```typescript
import { Order } from "effect"

const ascendingOrder = Order.number

const descendingOrder = Order.reverse(ascendingOrder)

console.log(ascendingOrder(1, 3)) 
// Output: -1 (1 < 3 in ascending order)
console.log(descendingOrder(1, 3))
// Output: 1 (1 > 3 in descending order)
```

---

### Combine Multiple Orders for Complex Sorting in Effect-TS

Source: https://effect.website/docs/behaviour/order

Demonstrates how to sort an array of `Person` objects by multiple criteria using `Order.combine`. This example sorts first by `name` and then by `age` for individuals with the same name.

```typescript
import { Order, Array } from "effect"

// Define the Person interface
interface Person {
  readonly name: string
  readonly age: number
}

// Create an Order to sort people by their names in ascending order
const byName = Order.mapInput(
  Order.string,
  (person: Person) => person.name
)

// Create an Order to sort people by their ages in ascending order
const byAge = Order.mapInput(Order.number, (person: Person) => person.age)

// Combine orders to sort by name, then by age
const byNameAge = Order.combine(byName, byAge)

const result = Array.sort(
  [
    { name: "Bob", age: 20 },
    { name: "Alice", age: 18 },
    { name: "Bob", age: 18 }
  ],
  byNameAge
)

console.log(result)
/* 
Output:
[
  { name: 'Alice', age: 18 }, // Sorted by name
  { name: 'Bob', age: 18 }, // Sorted by age within the same name
  { name: 'Bob', age: 20 }
]
*/
```

---

### Create an Empty Chunk using Chunk.empty

Source: https://effect.website/docs/data-types/chunk

Demonstrates how to create an empty Chunk using the `Chunk.empty` static method. This is useful for initializing a Chunk when no elements are present initially.

```typescript
import { Chunk } from "effect"

// ┌─── Chunk
// ▼
const chunk = Chunk.empty()
```

---

### Derive Custom Order for Objects with Effect-TS

Source: https://effect.website/docs/behaviour/order

Shows how to create a custom sorting order for an array of objects using `Order.mapInput`. This example defines an order to sort `Person` objects by their `name` property.

```typescript
import { Order } from "effect"

// Define the Person interface
interface Person {
  readonly name: string
  readonly age: number
}

// Create a custom order to sort Person objects by name in ascending order
const byName = Order.mapInput(
  Order.string,
  (person: Person) => person.name
)
```

---

### BigDecimal Creation

Source: https://effect.website/docs/data-types/bigdecimal

Demonstrates various methods for creating BigDecimal instances, including from BigInt, strings, and numbers.

```APIDOC
## BigDecimal Creation

### fromBigInt

The `fromBigInt` function creates a `BigDecimal` from a `bigint`. The `scale` defaults to `0`, meaning the number has no fractional part.

**Example** (Creating a BigDecimal from a BigInt)
```
1 import { BigDecimal } from "effect"
2 
3 const decimal = BigDecimal.fromBigInt(10n)
4 
5 console.log(decimal)
6 // Output: { _id: 'BigDecimal', value: '10', scale: 0 }
```

### fromString

Parses a numerical string into a `BigDecimal`. Returns an `Option`:
* `Some(BigDecimal)` if the string is valid.
* `None` if the string is invalid.

**Example** (Parsing a String into a BigDecimal)
```
1 import { BigDecimal } from "effect"
2 
3 const decimal = BigDecimal.fromString("0.02")
4 
5 console.log(decimal)
6 /*
7 Output:
8 {
9  _id: 'Option',
10  _tag: 'Some',
11  value: { _id: 'BigDecimal', value: '2', scale: 2 }
12 }
13 */
```

### unsafeFromString

The `unsafeFromString` function is a variant of `fromString` that throws an error if the input string is invalid. Use this only when you are confident that the input will always be valid.

**Example** (Unsafe Parsing of a String)
```
1 import { BigDecimal } from "effect"
2 
3 const decimal = BigDecimal.unsafeFromString("0.02")
4 
5 console.log(decimal)
6 // Output: { _id: 'BigDecimal', value: '2', scale: 2 }
```

### unsafeFromNumber

Creates a `BigDecimal` from a JavaScript `number`. Throws a `RangeError` for non-finite numbers (`NaN`, `+Infinity`, or `-Infinity`).

**Example** (Unsafe Parsing of a Number)
```
1 import { BigDecimal } from "effect"
2 
3 console.log(BigDecimal.unsafeFromNumber(123.456))
4 // Output: { _id: 'BigDecimal', value: '123456', scale: 3 }
```

**Avoid Direct Conversion**
Avoid converting floating-point numbers directly to `BigDecimal`, as their representation may already introduce precision issues.
```

---

### Perform Basic Arithmetic with BigDecimal in TypeScript

Source: https://effect.website/docs/data-types/bigdecimal

Provides examples of performing common arithmetic operations (addition, multiplication, subtraction, division, negation, modulus) using the BigDecimal module. These operations ensure precision and avoid floating-point errors. Requires the 'effect' package.

```typescript
import { BigDecimal } from "effect"

const dec1 = BigDecimal.unsafeFromString("1.05")
const dec2 = BigDecimal.unsafeFromString("2.10")

// Addition
console.log(String(BigDecimal.sum(dec1, dec2)))
// Output: BigDecimal(3.15)

// Multiplication
console.log(String(BigDecimal.multiply(dec1, dec2)))
// Output: BigDecimal(2.205)

// Subtraction
console.log(String(BigDecimal.subtract(dec2, dec1)))
// Output: BigDecimal(1.05)

// Division (safe, returns Option)
console.log(BigDecimal.divide(dec2, dec1))
/* 
Output: 
{
  _id: 'Option',
  _tag: 'Some',
  value: { _id: 'BigDecimal', value: '2', scale: 0 }
}
*/

// Division (unsafe, throws if divisor is zero)
console.log(String(BigDecimal.unsafeDivide(dec2, dec1)))
// Output: BigDecimal(2)

// Negation
console.log(String(BigDecimal.negate(dec1)))
// Output: BigDecimal(-1.05)

// Modulus (unsafe, throws if divisor is zero)
console.log(
  String(
    BigDecimal.unsafeRemainder(dec2, BigDecimal.unsafeFromString("0.6") )
  )
)
// Output: BigDecimal(0.3)
```

---

### HashSet Basic Creation and Operations

Source: https://effect.website/docs/data-types/hash-set

Illustrates the fundamental operations of Effect's immutable HashSet, including creation, adding elements, and performing set algebra like union, intersection, and difference. It highlights that adding an element to an immutable set returns a new set, leaving the original unchanged.

```typescript
import { HashSet } from "effect"

// Create an initial set with 3 values
const set1 = HashSet.make(1, 2, 3)

// Add a value (returns a new set)
const set2 = HashSet.add(set1, 4)

// The original set is unchanged
console.log(HashSet.toValues(set1))
// Output: [1, 2, 3]

console.log(HashSet.toValues(set2))
// Output: [1, 2, 3, 4]

// Perform set operations with another set
const set3 = HashSet.make(3, 4, 5)

// Combine both sets
const union = HashSet.union(set2, set3)

console.log(HashSet.toValues(union))
// Output: [1, 2, 3, 4, 5]

// Shared values
const intersection = HashSet.intersection(set2, set3)

console.log(HashSet.toValues(intersection))
// Output: [3, 4]

// Values only in set2
const difference = HashSet.difference(set2, set3)

console.log(HashSet.toValues(difference))
// Output: [1, 2]
```

---

### MutableHashSet - Concepts and Operations

Source: https://effect.website/docs/data-types/hash-set

Explains the concept of MutableHashSet, highlighting its mutability and contrasting it with HashSet, and lists its available operations.

```APIDOC
## MutableHashSet - Concepts and Operations

A `MutableHashSet` allows for in-place modifications, offering performance benefits for frequent updates.

### Description

A `MutableHashSet` is a **mutable**, **unordered** collection of **unique** values. Unlike `HashSet`, operations like `add`, `remove`, and `clear` modify the original set directly. This mutability is beneficial for scenarios requiring repeated modifications within a contained scope.

### Example (Using `Schema.Data` with `MutableHashSet`)

```typescript
import { Equal, MutableHashSet, Schema } from "effect"

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

// person1 and person2 are different instances but equal in value
console.log(Equal.equals(person1, person2)) // Output: true

// Add both to a MutableHashSet — only one will be stored
const set = MutableHashSet.empty<typeof person1>().pipe(
  MutableHashSet.add(person1),
  MutableHashSet.add(person2)
)

console.log(MutableHashSet.size(set)) // Output: 1
```

### Operations

| Category      | Operation    | Description                                     | Complexity |
|---------------|--------------|-------------------------------------------------|------------|
| Constructors  | `empty`      | Creates an empty MutableHashSet                 | O(1)       |
| Constructors  | `fromIterable`| Creates a MutableHashSet from an iterable       | O(n)       |
| Constructors  | `make`       | Creates a MutableHashSet from multiple values   | O(n)       |
| Elements      | `has`        | Checks if a value exists in the set             | O(1) avg   |
| Elements      | `some`       | Checks if any element satisfies a predicate     | O(n)       |
| Elements      | `every`      | Checks if all elements satisfy a predicate      | O(n)       |
| Getters       | `values`     | Gets an `Iterator` of all values                | O(1)       |
| Getters       | `toValues`   | Gets an `Array` of all values                   | O(n)       |
| Getters       | `size`       | Gets the number of elements                     | O(1)       |
| Mutations     | `add`        | Adds a value to the set (modifies in-place)     | O(1) avg   |
| Mutations     | `remove`     | Removes a value from the set (modifies in-place)| O(1) avg   |
| Mutations     | `clear`      | Removes all elements from the set               | O(n)       |
| Operations    | `difference` | Computes set difference (A - B) (returns new set)| O(n)       |
| Operations    | `intersection`| Computes set intersection (A ∩ B) (returns new set)| O(n)       |
| Operations    | `union`      | Computes set union (A ∪ B) (returns new set)    | O(n)       |
| Mapping       | `map`        | Transforms each element (returns new set)       | O(n)       |
| Filtering     | `filter`     | Keeps elements that satisfy a predicate (returns new set)| O(n)       |
```

---

### TypeScript: Using Built-in Order Comparators

Source: https://effect.website/docs/behaviour/order

Demonstrates how to use the built-in Order comparators for string, number, and bigint types. These functions simplify comparisons without needing to manually implement the logic.

```typescript
import { Order } from "effect"

console.log(Order.string("apple", "banana"))
// Output: -1, as "apple" < "banana"

console.log(Order.number(1, 1))
// Output: 0, as 1 = 1

console.log(Order.bigint(2n, 1n))
// Output: 1, as 2n > 1n
```

---

### Create and Compare Objects with Effect Data

Source: https://effect.website/docs/data-types/hash-set

Demonstrates how to define a data structure using Effect's Schema.Data and compare instances for value equality. It shows that even with different object references, value-equal objects are treated as identical by Effect's Equal.equals and can be stored uniquely in a HashSet.

```typescript
import { Equal, Schema } from "effect"

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

// person1 and person2 are different instances but equal in value
console.log(Equal.equals(person1, person2))
// Output: true

// Add both to a MutableHashSet — only one will be stored
import { MutableHashSet } from "effect"

const set = MutableHashSet.empty().pipe(
  MutableHashSet.add(person1),
  MutableHashSet.add(person2)
)

console.log(MutableHashSet.size(set))
// Output: 1
```

---

### MutableHashSet Constructors

Source: https://effect.website/docs/data-types/hash-set

Functions to create and initialize MutableHashSet instances.

```APIDOC
## MutableHashSet Constructors

### empty

Creates an empty MutableHashSet.

### Method

constructor

### Endpoint

N/A (Constructor)

### Parameters

None

### Request Example

```json
{}
```

### Response

#### Success Response (200)

- **MutableHashSet** - An empty mutable hash set.

### Response Example

```json
{
  "instance": "MutableHashSet"
}
```

## MutableHashSet Constructors

### fromIterable

Creates a set from an iterable.

### Method

constructor

### Endpoint

N/A (Constructor)

### Parameters

- **iterable** (Iterable<A>) - Required - An iterable collection of values.

### Request Example

```json
{
  "iterable": [
    1,
    2,
    3
  ]
}
```

### Response

#### Success Response (200)

- **MutableHashSet** - A mutable hash set initialized with values from the iterable.

### Response Example

```json
{
  "instance": "MutableHashSet"
}
```

## MutableHashSet Constructors

### make

Creates a set from multiple values.

### Method

constructor

### Endpoint

N/A (Constructor)

### Parameters

- **elements** (...A) - Required - A variable number of values to add to the set.

### Request Example

```json
{
  "elements": [
    1,
    2,
    3
  ]
}
```

### Response

#### Success Response (200)

- **MutableHashSet** - A mutable hash set initialized with the provided values.

### Response Example

```json
{
  "instance": "MutableHashSet"
}
```
```

---

### Create a Non-Empty Chunk using Chunk.make

Source: https://effect.website/docs/data-types/chunk

Illustrates how to create a non-empty Chunk with specific values using the `Chunk.make` static method. The resulting Chunk is typed as `NonEmptyChunk`.

```typescript
import { Chunk } from "effect"

// ┌─── NonEmptyChunk
// ▼
const chunk = Chunk.make(1, 2, 3)
```
