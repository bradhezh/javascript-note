---
title: 2.8.2 - The Type System
---

TypeScript types are structural rather than nominal. For example, if an object type has all members required by another, it is assignable to that one even if it possesses additional members. In fact, _A_ is assignable to _B_ means that any instance of _A_ meets (satisfies) the requirements of _B_ and then can be regarded as an instance of _B_, and _A_ can actually be more specific (narrow) than _B_. For example, a subclass is assignable to its superclass. Types can either be implicitly inferred by TypeScript or explicitly annotated by programmers. If left unannotated, a type that cannot be inferred implicitly becomes `any`, which disables type checking. Implicit `any` can be prevented by the `noImplicitAny` compiler option, and `unknown` can be used for better type safety. Everything is assignable to `unknown`, but `unknown` is not assignable to anything except itself and `any`, meaning that `unknown` can be regarded as the most general (wide) type.

<table>
  <caption>Type Constructs</caption>
  <thead>
    <tr><th>Concept</th><th>Description</th><th style={{ width: "45%" }}>Example</th></tr>
  </thead>
  <tbody>
    <tr>
      <td>Type Alias</td>
      <td>Custom name for any type (primitives, objects, unions, intersections, etc.)</td>
      <td>`type User = { id: number; name: string };`</td>
    </tr>
    <tr>
      <td>Interface</td>
      <td>Structure (shape) for objects, supporting extension and declaration merging</td>
      <td>`interface User { id: number; name: string }`</td>
    </tr>
    <tr>
      <td>Class</td>
      <td>For creating objects. Note that TypeScript classes will be compiled to JavaScript classes, whereas TypeScript interfaces or object types only exist at compile time without any data or code at runtime.</td>
      <td>`class User { constructor(public name: string) {} }`</td>
    </tr>
    <tr>
      <td>Array</td>
      <td>Ordered list of elements of one type</td>
      <td>`type Nums = number[];`</td>
    </tr>
    <tr>
      <td>Tuple</td>
      <td>Array with fixed number of elements and fixed types</td>
      <td>`type Point = [number, number];`</td>
    </tr>
    <tr>
      <td>Union</td>
      <td>Assignable to _A_ or assignable to _B_, for values of any one of multiple types</td>
      <td>`type ID = number | string;`</td>
    </tr>
    <tr>
      <td>Intersection</td>
      <td>Assignable to _A_ and assignable to _B_, e.g. to merge multiple object types</td>
      <td>`type User = { id: ID } & { name: string };`</td>
    </tr>
    <tr>
      <td>Literal</td>
      <td>Limited to specific values</td>
      <td>`type Direction = "left" | "right";`</td>
    </tr>
    <tr>
      <td>Enum</td>
      <td>A set of named constants (`number`/`string`), which remain at runtime. In contrast, TypeScript literals are types only existing at compile time.</td>
      <td>`enum Role { Admin, User }`</td>
    </tr>
    <tr>
      <td>Function</td>
      <td>Type signature for functions, with parameter and return types</td>
      <td>`type Hello = (user: User) => string;`</td>
    </tr>
    <tr>
      <td>Any</td>
      <td>Any type without type checking</td>
      <td>`any`</td>
    </tr>
    <tr>
      <td>Unknown</td>
      <td>Unknown type to be narrowed or asserted before using</td>
      <td>`unknown`</td>
    </tr>
  </tbody>
</table>

```typescript title="Interfaces, Classes, and Assignability"
interface Greetable {
  name: string;
  greet(): void;
}

class Person implements Greetable {
  /* Equivalent to
  public name: string;
  constructor(name: string) {
    this.name = name;
  }
  */
  constructor(public name: string) {}

  greet() {
    console.log(`Hello, my name is ${this.name}.`);
  }
}

class Employee extends Person {
  constructor(
    name: string,
    public role: string,
  ) {
    super(name);
  }

  greet() {
    super.greet();
    console.log(`I work as a ${this.role}.`);
  }
}

//Type: Employee
const e = new Employee("Brad", "developer");
e.greet();
// A subclass is assignable to its superclass.
((x: Person) => x.greet())(e);
((x: Greetable) => x.greet())(e);
```

```typescript title="Type Inference and Annotations"
// Inference from values
//Type: { a: string; b: string }
const v = { a: "a", b: "b" };
// Inference as consts
//Type: { readonly a: "a"; readonly b: "b" }
const c = { a: "a", b: "b" } as const;
// Inference with `satisfies`, kept as much as possible to just satisfy (be assignable to) the type
//Type: { a: string; b: "b" }
const satisfied = { a: "a", b: "b" } satisfies { a: string; b: "b" | "B" };
// Annotation
//Type: { a: string; b: "b" | "B" }
const annotated: { a: string; b: "b" | "B" } = { a: "a", b: "b" };

// An object with more members is assignable to a wider one,
const o1 = { a: "a", b: "b" };
const o: { a: string } = o1;
// but error: Object literal may only specify known properties, and 'b' does not exist in type
// '{ a: string; }'.
const fromObjLiteral: { a: string } = { a: "a", b: "b" };
```

In addition to inferring types from values, types can also be inferred from certain type operations such as `typeof`, `instanceof`, or custom type guards, known as type narrowing.

```typescript title="Type Narrowing"
// Type guard (function) with a type predicate as the return type
const isString = (value: unknown): value is string => {
  return typeof value === "string" || value instanceof String;
};

const printLength = (value: unknown) => {
  /* Equivalent to
  if (typeof value === "string" || value instanceof String) {
  */
  if (isString(value)) {
    // Narrowed to `string` now
    return console.log(`String length: ${value.length}`);
  }
  if (Array.isArray(value)) {
    // Narrowed to any[] now
    return console.log(`Array length: ${value.length}`);
  }

  console.log("Neither a string nor an array");
};
```

Note that in function assignability, the rule for parameters is reversed, meaning that when function type _A_ is assignable to function type _B_, parameter types of _B_ must be assignable to corresponding ones of _A_. This is because functions can actually take more general but not more specific parameter types to satisfy a function type.

```typescript title="Inversion for Parameters of Function Assignability"
// Error: Type '(x: "a") => void' is not assignable to type '(x: string) => unknown'. Types of
// parameters 'x' and 'x' are incompatible. Type 'string' is not assignable to type '"a"'.
const foo: (x: string) => unknown = (x: "a") => console.log(x);
// Functions with wider param types still satisfy the function type.
const bar = ((x: string) => console.log(x)) satisfies (x: "a") => unknown;
bar("a");
bar("b");
```

Apart from TypeScript compiler options like `noImplicitAny`, [ESLint](/tool/toolchain/eslint) can also be used to restrict `any` for stronger type safety.

```typescript title="Type Safety Practice"
// ESLint `no-explicit-any` can prevent `any` from being explicitly used. As for `any` implicitly
// returned from others, ESLint `no-unsafe-assignment`, `no-unsafe-member-access`, and
// `no-unsafe-call` can prevent it from being implicitly used, to make the unsafe use noticeable.
// ESLint: Unsafe assignment of an `any` value. (eslint @typescript-eslint/no-unsafe-assignment)
const unsafe = JSON.parse("{}");
const safe = JSON.parse("{}") as unknown;

// any[] here can't be replaced by unknown[] due to param assignability inversion, as no function
// can take param types wider than `unknown`.
// eslint-disable-next-line @typescript-eslint/no-explicit-any
type GeneralFunc = (...params: any[]) => unknown;
// No type checking for params
((f: GeneralFunc) => f())((x: string) => console.log(x));
```
