# Day 2 — JavaScript & TypeScript Revision

**Topics:** Prototypes & Inheritance · Class vs `Object.create` · Union & Intersection Types · Type Guards

---

## Table of Contents

1. [Prototypes — Foundation](#1-prototypes--foundation)
2. [Prototype Chain & Inheritance](#2-prototype-chain--inheritance)
3. [Class vs Object.create vs Constructor Functions](#3-class-vs-objectcreate-vs-constructor-functions)
4. [TypeScript Union Types](#4-typescript-union-types)
5. [TypeScript Intersection Types](#5-typescript-intersection-types)
6. [Type Guards](#6-type-guards)
7. [Interview Quick Index](#7-interview-quick-index)

---

## 1. Prototypes — Foundation

### Every object has a hidden link

In JavaScript, objects inherit from other objects via the **prototype chain**. Every object has an internal slot `[[Prototype]]` (exposed as `__proto__` in browsers — avoid in production; use `Object.getPrototypeOf`).

```js
const obj = { name: "Alice" };
console.log(Object.getPrototypeOf(obj) === Object.prototype); // true
```

### Functions have `.prototype`; objects don't (usually)

When you call a function with `new`, the created object's `[[Prototype]]` is set to the function's `.prototype`.

```js
function Person(name) {
  this.name = name;
}
Person.prototype.greet = function () {
  return `Hi, ${this.name}`;
};

const p = new Person("Bob");
console.log(p.greet());           // "Hi, Bob"
console.log(p.hasOwnProperty("name"));   // true — own property
console.log(p.hasOwnProperty("greet"));  // false — on prototype
```

### Property lookup order

1. **Own properties** on the object
2. Walk up **prototype chain** until `null`
3. Return `undefined` if not found

```js
const animal = { eats: true };
const rabbit = Object.create(animal);
rabbit.jumps = true;

console.log(rabbit.jumps);  // true (own)
console.log(rabbit.eats);   // true (inherited from animal)
console.log(rabbit.toString); // function (from Object.prototype)
```

### `Object.create(proto)` — explicit prototype

Creates a new object with `[[Prototype]]` set to `proto`.

```js
const proto = {
  speak() { return "woof"; }
};
const dog = Object.create(proto);
dog.name = "Rex";
console.log(dog.speak()); // "woof" — inherited method
```

---

## 2. Prototype Chain & Inheritance

### Classic inheritance pattern (pre-ES6)

```js
function Animal(name) {
  this.name = name;
}
Animal.prototype.speak = function () {
  return `${this.name} makes a sound`;
};

function Dog(name, breed) {
  Animal.call(this, name);  // "super" constructor
  this.breed = breed;
}
// Link prototypes — Dog.prototype inherits from Animal.prototype
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

Dog.prototype.speak = function () {
  return `${this.name} barks`;
};

const d = new Dog("Max", "Lab");
console.log(d.speak()); // "Max barks"
console.log(d instanceof Dog);    // true
console.log(d instanceof Animal); // true
```

### ES6 `class` — syntactic sugar

```js
class Animal {
  constructor(name) {
    this.name = name;
  }
  speak() {
    return `${this.name} makes a sound`;
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);
    this.breed = breed;
  }
  speak() {
    return `${this.name} barks`;
  }
}

const d = new Dog("Max", "Lab");
```

> Under the hood: still prototype-based. `class` does NOT create a new inheritance model.

### `instanceof` — prototype chain check

```js
console.log(d instanceof Dog);     // true
console.log(d instanceof Animal);  // true
console.log(d instanceof Object);  // true

// Custom check without instanceof
function isDog(obj) {
  return Object.getPrototypeOf(obj) === Dog.prototype;
}
```

### Mixins via Object.assign on prototype

```js
const CanSwim = {
  swim() { return `${this.name} swims`; }
};
Object.assign(Dog.prototype, CanSwim);
```

---

## 3. Class vs Object.create vs Constructor Functions

| Approach | Pros | Cons |
|----------|------|------|
| **Constructor + prototype** | Full control, interview classic | Verbose inheritance setup |
| **`Object.create`** | Explicit prototype, no `new` required | Less familiar syntax |
| **`class`** | Readable, `extends`/`super` | Can mislead into "class-based OOP" thinking |

### Object.create for inheritance (no `class`)

```js
const personProto = {
  greet() { return `Hello, ${this.name}`; }
};

function createPerson(name) {
  const obj = Object.create(personProto);
  obj.name = name;
  return obj;
}

const alice = createPerson("Alice");
console.log(alice.greet()); // "Hello, Alice"
```

### Factory + closure (alternative to inheritance)

When behavior differs by **composition** not **is-a**:

```js
function createCounter(initial = 0) {
  let count = initial;
  return {
    increment() { count++; return count; },
    get value() { return count; }
  };
}
// No prototype chain — data hidden in closure
```

### Interview: when to use what?

| Scenario | Recommendation |
|----------|----------------|
| React components | Functional + hooks (no classical inheritance) |
| Shared methods on many instances | Prototype or `class` |
| One-off objects with shared behavior | `Object.create` |
| Private state | Closure / `#private` fields (ES2022) |
| Library API design | Factory functions or classes depending on team style |

### Private fields (modern alternative)

```js
class BankAccount {
  #balance = 0;
  deposit(amount) { this.#balance += amount; }
  get balance() { return this.#balance; }
}
```

---

## 4. TypeScript Union Types

### Definition

A value can be **one of several types** — written with `|`.

```ts
type Status = "idle" | "loading" | "success" | "error";
type Id = string | number;

let s: Status = "idle";
// s = "pending"; // Error — not in union
```

### Narrowing with conditionals

```ts
function printId(id: string | number) {
  if (typeof id === "string") {
    console.log(id.toUpperCase()); // id is string here
  } else {
    console.log(id.toFixed(2));    // id is number here
  }
}
```

### Discriminated unions (tagged unions)

Add a **literal field** so TypeScript can narrow reliably.

```ts
type ApiResult =
  | { status: "success"; data: User[] }
  | { status: "error"; message: string };

function handle(result: ApiResult) {
  switch (result.status) {
    case "success":
      return result.data.length;  // data exists
    case "error":
      return result.message;      // message exists
  }
}
```

### Union with `null` / `undefined`

```ts
function greet(name: string | null) {
  if (name === null) return "Guest";
  return `Hello, ${name}`;
}
```

---

## 5. TypeScript Intersection Types

### Definition

Combine **multiple types** — value must satisfy **all** of them. Written with `&`.

```ts
type Named = { name: string };
type Aged = { age: number };
type Person = Named & Aged;

const p: Person = { name: "Alice", age: 30 };
```

### Intersection vs extends

```ts
interface Employee extends Person {
  employeeId: string;
}
// Similar to: type Employee = Person & { employeeId: string };
```

### Mixing interfaces and types

```ts
type Timestamped = { createdAt: Date };
type User = { id: string; email: string };
type UserRecord = User & Timestamped;
```

### Conflict resolution

If two intersected types have the same property with **incompatible** types → `never` for that property.

```ts
type A = { x: string };
type B = { x: number };
type C = A & B; // x: never — impossible to satisfy both
```

### Practical use: React props

```ts
type ButtonProps = React.ButtonHTMLAttributes<HTMLButtonElement> & {
  variant: "primary" | "secondary";
  loading?: boolean;
};
```

---

## 6. Type Guards

### `typeof` guard

```ts
function padLeft(value: string, padding: string | number) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + value;
  }
  return padding + value;
}
```

### `instanceof` guard

```ts
class Dog { bark() { return "woof"; } }
class Cat { meow() { return "meow"; } }

function speak(pet: Dog | Cat) {
  if (pet instanceof Dog) {
    return pet.bark();
  }
  return pet.meow();
}
```

### `in` operator guard

```ts
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    animal.swim();
  } else {
    animal.fly();
  }
}
```

### User-defined type predicates

```ts
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function process(input: unknown) {
  if (isString(input)) {
    console.log(input.toUpperCase()); // input: string
  }
}
```

### Assertion functions (TS 3.7+)

```ts
function assertIsNumber(val: unknown): asserts val is number {
  if (typeof val !== "number") {
    throw new Error("Not a number");
  }
}

function double(x: unknown) {
  assertIsNumber(x);
  return x * 2; // x is number
}
```

### Exhaustiveness checking with `never`

```ts
type Shape = { kind: "circle"; radius: number } | { kind: "square"; side: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.side ** 2;
    default:
      const _exhaustive: never = shape;
      return _exhaustive;
  }
}
```

### Interview snippets

| Question | Answer |
|----------|--------|
| Prototype vs `__proto__` vs `.prototype` | `.prototype` on functions; `[[Prototype]]` on objects; `__proto__` is legacy accessor |
| Is JS class-based? | No — prototype-based; `class` is syntax sugar |
| Union vs intersection | `\|` = OR; `&` = AND (merge requirements) |
| Type guard purpose | Narrow union to specific type in a branch |
| `value is T` | Return type of predicate — tells TS the branch type |

---

## 7. Interview Quick Index

### Prototypes & Inheritance

| Question | Section |
|----------|---------|
| What is prototype chain? | [§1](#1-prototypes--foundation) |
| `Object.create` vs `new` | [§1](#1-prototypes--foundation), [§3](#3-class-vs-objectcreate-vs-constructor-functions) |
| How does `instanceof` work? | [§2](#2-prototype-chain--inheritance) |
| ES6 class vs prototype inheritance | [§2](#2-prototype-chain--inheritance), [§3](#3-class-vs-objectcreate-vs-constructor-functions) |
| Factory vs inheritance | [§3](#3-class-vs-objectcreate-vs-constructor-functions) |

### TypeScript Unions & Guards

| Question | Section |
|----------|---------|
| Union type syntax | [§4](#4-typescript-union-types) |
| Discriminated union | [§4](#4-typescript-union-types) |
| Intersection type | [§5](#5-typescript-intersection-types) |
| `typeof` / `instanceof` / `in` guards | [§6](#6-type-guards) |
| Custom type predicate | [§6](#6-type-guards) |
| Exhaustive switch with `never` | [§6](#6-type-guards) |

---

## Day 2 Cheat Sheet (30-second recall)

```
Prototype     → objects inherit via [[Prototype]] chain
Object.create → new object with given proto (no class needed)
class         → syntax sugar over prototypes; extends/super
Union (|)     → value is ONE of several types
Intersection (&) → value satisfies ALL types
Type guard    → narrow union: typeof, instanceof, in, is predicate
Next TS      → Pick/Omit/Partial/Record — see TS utilities supplement
```

**Next:** [TypeScript Utility Types](../supplements/sde1_typescript_utilities.md) · Day 4 mapped types · Day 5 conditional types

---

*End of Day 2 revision*
