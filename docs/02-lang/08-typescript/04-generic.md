---
title: 2.8.4. Type Logic and Generics
---

In addition to implicit type inference, TypeScript also supports explicit type manipulation and transformation with various logic. TypeScript provides structures including conditional types, mapped types, and template literal types, with operations like `typeof`, `keyof`, indexed access, key remapping, and `infer`, as well as generics, where type parameters (instead of general types like `object`) can be used for types or functions without widening.

```typescript title="Type Logic and Generics"
// Conditional types (X extends Y ? T1 : T2)
// Actually, built-in NonNullable<T> is defined as `T & {}`.
type NonNullable<T> = T extends null | undefined ? never : T;
//Type: string | number
type NonNullableT = NonNullable<string | number | null | undefined>;

// Mapped types ({ [K in U]: V })
// Note that `{ a?: "a" }` is the same as `{ a?: "a" | undefined }`.
type Partial<T> = { [K in keyof T]?: T[K] };
//Type: { a?: "a" | undefined; b?: "b" | undefined }
type PartialT = Partial<{ a: "a"; b: "b" }>;
// Note that `-?` removes `?` and only removes `undefined` for `?`.
type Required<T> = { [K in keyof T]-?: T[K] };
//Type: { a: "a"; b: "b" | undefined }
type RequiredT = Required<{ a?: "a"; b: "b" | undefined }>;

// Template literal types
type Event = "Click" | "Hover";
type Target = "Button" | "Link";
//Type: "onClickButton" | "onClickLink" | "onHoverButton" | "onHoverLink"
type Handler = `on${Event}${Target}`;

// Key remapping ({ [K in U as ...]: V })
type Getters<T extends object> = {
  [K in keyof T as T[K] extends Function ? never : `get_${string & K}`]: () => T[K];
};
//Type: { get_a: () => "a"; get_b: () => "b" }
type GettersT = Getters<{ a: "a"; b: "b"; foo(): void }>;

// infer (destructuring) (T extends ... infer X ... ? Y : Z)
// eslint-disable-next-line @typescript-eslint/no-explicit-any
type ReturnType<T> = T extends (...params: any[]) => infer R ? R : never;
//Type: string | undefined
type Return = ReturnType<(x: number) => string | undefined>;

// Generic function
const keys = <T extends object>(o: T) => {
  return Object.keys(o) as (keyof T)[];
};
const brad = { id: 0, name: "Brad", role: "developer" };
// Type args can be implicitly inferred from function args,
//Type: ("id" | "name" | "role")[]
const bradKeys = keys(brad);
// or explicitly specified.
//Type: ("id" | "name")[]
const userKeys = keys<{ id: number; name: string }>(brad);
// No type widening
console.log(brad[bradKeys[0]]);
console.log(brad[userKeys[0]]);

// Note that Array.isArray narrows the value into any[], losing elem types, tuples, and `readonly`.
const isArray = <T>(value: T): value is T => {
  return Array.isArray(value);
};
((x: number | readonly [number, string]) => {
  if (typeof x === "number") {
    //Type: number
    return console.log(x.toFixed());
  }
  if (isArray(x)) {
    //Type: readonly [number, string]
    console.log(x.length);
    console.log(x[1].length);
    // Error: Cannot assign to '0' because it is a read-only property.
    return (x[0] = 1);
  }
})([0, "hello"]);
```

TypeScript provides built-in utility types for common type logic.

<table>
  <caption>Utility Types</caption>
  <thead>
    <tr>
      <th>Utility Type</th><th>Example</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>`Record<K, V>`</td>
      <td>`Record<"a" | "b", string>` -> `{ a: string; b: string }`</td>
    </tr>
    <tr>
      <td>`Readonly<T>`</td>
      <td>`Readonly<{ a: "a"; b: "b" }>` -> `{ readonly a: "a"; readonly b: "b" }`</td>
    </tr>
    <tr>
      <td>`ReadonlyArray<T>`</td>
      <td>`ReadonlyArray<number>` -> `readonly number[]`</td>
    </tr>
    <tr>
      <td>`Map<K, V>`</td>
      <td>`((x: Map<string, number>) => console.log(x.set("a", 1).get("a")))(new Map([["a", 0]]));`</td>
    </tr>
    <tr>
      <td>`ReadonlyMap<K, V>`</td>
      <td>`((x: ReadonlyMap<string, number>) => console.log(x.get("a")))(new Map([["a", 0]]));`</td>
    </tr>
    <tr>
      <td>`Set<T>`</td>
      <td>`((x: Set<number>) => console.log(x.add(1).has(1)))(new Set([0]));`</td>
    </tr>
    <tr>
      <td>`ReadonlySet<T>`</td>
      <td>`((x: ReadonlySet<number>) => console.log(x.has(0)))(new Set([0]));`</td>
    </tr>
    <tr>
      <td>`Partial<T>`</td>
      <td>`Partial<{ a: "a"; b: "b" }>` -> `{ a?: "a" | undefined; b?: "b" | undefined }`</td>
    </tr>
    <tr>
      <td>`Required<T>`</td>
      <td>`Required<{ a?: "a"; b: "b" }>` -> `{ a: "a"; b: "b" }`</td>
    </tr>
    <tr>
      <td>`Pick<T, K>`</td>
      <td>`Pick<{ a: "a"; b: "b"; c: "c" }, "a" | "b">` -> `{ a: "a"; b: "b" }`</td>
    </tr>
    <tr>
      <td>`Omit<T, K>`</td>
      <td>`Omit<{ a: "a"; b: "b"; c: "c" }, "a" | "b" | "x">` -> `{ c: "c" }`</td>
    </tr>
    <tr>
      <td>`Extract<T1, T2>`</td>
      <td>`Extract<"a" | "b" | "c", "a" | "b" | "x">` -> `"a" | "b"`</td>
    </tr>
    <tr>
      <td>`Exclude<T1, T2>`</td>
      <td>`Exclude<"a" | "b" | "c", "a" | "b" | "x">` -> `"c"`</td>
    </tr>
    <tr>
      <td>`NonNullable<T>`</td>
      <td>`NonNullable<"a" | "b" | null | undefined>` -> `"a" | "b"`</td>
    </tr>
    <tr>
      <td>`Uppercase<S>`<br />`Lowercase<S>`</td>
      <td>`Uppercase<"click" | "hover">` -> `"CLICK" | "HOVER"`</td>
    </tr>
    <tr>
      <td>`Capitalize<S>`<br />`Uncapitalize<S>`</td>
      <td>`Capitalize<"click" | "hover">` -> `"Click" | "Hover"`</td>
    </tr>
    <tr>
      <td>`ReturnType<F>`</td>
      <td>`ReturnType<(x: number) => string | undefined>` -> `string | undefined`</td>
    </tr>
  </tbody>
</table>

Generally, wider types enforce less requirements while support less operations, and union can make types wider while intersection can make types narrower. Note that a readonly array (or map/set) is wider than and then not assignable to its writeable equivalent, but an object with readonly members and its writeable equivalent are mutually assignable to each other. Type logic will normally be distributed across each member of a union in a reasonable way. For example, `T[K1 | K2]` returns `T[K1] | T[K2]`. Note that `keyof (T1 | T2)` returns `keyof T1 & keyof T2`, and from a function union like `((x: T1) => unknown) | ((x: T2) => unknown)`, the parameter type inferred in `(x: infer T) => unknown` is `T1 & T2`. TypeScript also supports recursion in type logic.

```typescript title="Union, Intersection, and readonly"
// Note that `number & string` is `never`, but `number & unknown[]` isn't, although nothing can
// actually satisfy it as well.
const foo = (x: number & string) => {
  // Error: Property 'toFixed' does not exist on type 'never'.
  console.log(x.toFixed());
  // Error: Property 'length' does not exist on type 'never'.
  console.log(x.length);
};
const bar = (x: number & unknown[]) => {
  // With all operations on `number` or unknown[]
  console.log(x.toFixed());
  console.log(x.length);
};

type Extend<T1, T2> = T1 extends T2 ? true : false;
/* eslint-disable @typescript-eslint/no-duplicate-type-constituents */
//Type: false
((
  x:
    | Extend<number[], [number, number]>
    | Extend<readonly number[], number[]>
    | Extend<readonly [number, number], [number, number]>
    | Extend<ReadonlyMap<string, number>, Map<string, number>>
    | Extend<ReadonlySet<number>, Set<number>>,
) => console.log(x))(false);
//Type: true
((
  x: Extend<[number, number], number[]> &
    Extend<number[], readonly number[]> &
    Extend<[number, number], readonly [number, number]> &
    Extend<Map<string, number>, ReadonlyMap<string, number>> &
    Extend<Set<number>, ReadonlySet<number>>,
) => console.log(x))(true);
//Type: true
((
  x: Extend<{ readonly a: "a"; b: "b" }, { a: "a"; readonly b: "b" }> &
    Extend<{ a: "a"; readonly b: "b" }, { readonly a: "a"; b: "b" }>,
) => console.log(x))(true);
/* eslint-enable @typescript-eslint/no-duplicate-type-constituents */

// Note that `readonly number[]` isn't narrower than unknown[] as `readonly` makes it wider than a
// writeable one. It seems like `& unknown[]` removes `readonly` from arrays.
//Type -> number[]
((x: readonly number[] & unknown[]) => (x[0] = 1))([0, 1]);
// Note that `& unknown[]` doesn't completely remove `readonly` from tuples.
((x: readonly [number, number] & unknown[]) => {
  // Writeable
  x.push(2);
  // Error: Cannot assign to '0' because it is a read-only property.
  x[0] = 1;
  console.log(x);
})([0, 1]);

type Writeable<T> = { -readonly [K in keyof T]: T[K] };
//Type: { a?: "a" | undefined; b?: "b" | undefined }
type WriteableObj = Writeable<{ readonly a?: "a"; b?: "b" }>;
// Also applicable to arrays, like Readonly<T>
//Type: number[]
type WriteableArr = Writeable<readonly number[]>;
//Type: readonly number[]
type ReadonlyArr = Readonly<number[]>;
//Type: [number, number]
type WriteableTpl = Writeable<readonly [number, number]>;
//Type: readonly [number, number]
type ReadonlyTpl = Readonly<[number, number]>;
// Readonly<T> and Writeable<T> aren't applicable to maps and sets.
type WriteableMap<T> = T extends Map<infer K, infer V> | ReadonlyMap<infer K, infer V>
  ? Map<K, V>
  : never;
type WriteableSet<T> = T extends Set<infer E> | ReadonlySet<infer E> ? Set<E> : never;
//Type: Map<string, number>
type WriteableMapT = WriteableMap<ReadonlyMap<string, number>>;
//Type: Set<number>
type WriteableSetT = WriteableSet<ReadonlySet<number>>;
```

```typescript title="Union Distribution"
// Indexed access
type T1 = { a: "a1"; b: "b1"; x: "x" };
type T2 = { a: "a2"; b: "b2"; y: "y" };
//Type: "a1" | "b1" | "a2" | "b2"
type V = (T1 | T2)["a" | "b"];
type V1 = T1["a"] | T1["b"] | T2["a"] | T2["b"];
// keyof
//Type: "a" | "b"
type K = keyof (T1 | T2);
type K1 = keyof T1 & keyof T2;

// Mapped types
// No union, no distribution
//Type: { [x: string]: { [x: string]: string } }
type CascadeFailed = { [K in string]: { [K1 in K]: K } };
// Distribution on union
type Cascade<K extends string> = { [K1 in K]: { [K2 in K1]: K1 } };
//Type: { a: { a: "a" }; b: { b: "b" } }
type CascadeT = Cascade<"a" | "b">;

// Conditional types
type AllKeys<U> = U extends unknown ? keyof U : never;
type AllVals<U> = U extends unknown ? U[keyof U] : never;
type U = { readonly a: "a" } | { b?: "b" };
//Type: never
type KofU = keyof U;
type VofU = U[KofU];
//Type: "a" | "b"
type KinU = AllKeys<U>;
//Type: "a" | "b" | undefined
type VinU = AllVals<U>;

// Function union
// Note that `U extends unknown` makes the function union via union distribution of conditional
// types. Without it, the function will take a union as the param rather than being a function
// union.
type UnionToIntersection<U> = (U extends unknown ? (x: U) => void : never) extends (
  x: infer I,
) => void
  ? I
  : never;
//Type: { readonly a: "a"; b?: "b" | undefined }
type I = UnionToIntersection<{ readonly a: "a" } | { b?: "b" }>;

// Support members with the same name but no intersection type, but remove `readonly` and `?`
type MergeUnion<U extends object> = {
  [K in AllKeys<U>]: U extends { [K1 in K]?: infer V } ? V : never;
};
//Type: { a: "a1" | "a2"; b: "b" }
type Merged = MergeUnion<{ readonly a: "a1" } | { b?: "b"; a: "a2" }>;
```

```typescript title="Recursive Types"
// `| undefined` for `?`. Note that arrays are objects as well, and the logic is also applicable to
// arrays.
type DeepWriteable<T> = T extends object
  ? { -readonly [K in keyof T]: T[K] extends object | undefined ? DeepWriteable<T[K]> : T[K] }
  : T;
//Type: { a?: number[] | undefined; b: { c?: { d?: [number, number] | undefined } | undefined } }
type WriteableT = DeepWriteable<{
  readonly a?: readonly number[];
  b: { readonly c?: { readonly d?: readonly [number, number] } };
}>;

type Flatten<T> = T extends object
  ? UnionToIntersection<
      Exclude<
        | {
            [
              // Note that any array `extends readonly unknown[]`.
              K in keyof T as T[K] extends readonly unknown[] | undefined
                ? K
                : // Then objects except arrays
                  T[K] extends object | undefined
                  ? never
                  : K
            ]: T[K];
          }
        | {
            [K in keyof T]: T[K] extends readonly unknown[] | undefined
              ? object
              : T[K] extends object | undefined
                ? Flatten<T[K]>
                : object;
          }[keyof T],
        // Exclude `undefined` from `object | undefined` in the value union
        undefined
      >
    >
  : T;
//Type: {
//  readonly a1?: "a1" | undefined;
//  b: number[] | undefined;
//  readonly c?: readonly [number, number] | undefined;
//  a2: "a2";
//}
type Flattened = Flatten<{
  readonly a1?: "a1";
  x: { b: number[] | undefined; readonly y?: { readonly c?: readonly [number, number]; a2: "a2" } };
}>;
```
