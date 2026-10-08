---
title: 2.8.3. Type Assertions
---

At compile time, types can be inferred from values, but in function definitions, parameter types normally need to be annotated, except for inline callbacks, where parameter types can still be inferred from calling. Parameters might be annotated with wider types than the actual arguments passed to them, especially when working with general types like `object`, with the expected return type widened accordingly. Moreover, compile-time inference cannot always reflect real runtime types, which could depend on different runtime values or runtime logic that cannot be inferred at compile time. Type assertions then provide a resort for programmers to explicitly assert types, relying on runtime logic for type safety. Nevertheless, assertions retain partial compile-time type safety, where assignability must exist in at least one direction. For example, `unknown` can be asserted as anything because anything is assignable to `unknown`.

```typescript title="Type Assertions"
//Type: { id: number; name: string }
//Key: "id" | "name"
const brad = { id: 0, name: "Brad" };
// Param annotated with a wider type (typeof brad -> {}), with the expected result widened
// accordingly ((keyof typeof brad)[] -> string[]). Note that {} is wider than `object`, for any
// value with properties, including primitives (via auto-wrapping, e.g. "hello".length), so it's
// actually for any value except `null` and `undefined`.
//Object.keys: (o: {}) => string[]
const keys = Object.keys(brad);
// Then error: ... expression of type 'string' can't be used to index type
// '{ id: number; name: string; }' ...
console.log(brad[keys[0]]);
// Assert based on runtime logic
console.log(brad[keys[0] as keyof typeof brad]);

//Type: { a: { a: string }; b: { b: string } }
//Key: "a" | "b"
//Value: { a: string } | { b: string }
//Key of Value: never (The real type depends on different runtime values.)
const o = { a: { a: "a" }, b: { b: "b" } };
//Type: [string, { a: string } | { b: string }][]
const es = Object.entries(o);
// Then error: ... expression of type 'string' can't be used to index type
// '{ a: string; } | { b: string; }' ...
es.map(([k, v]) => console.log(v[k]));
// Or error: ... expression of type '"a" | "b"' can't be used to index type
// '{ a: string; } | { b: string; }' ...
(es as [keyof typeof o, (typeof o)[keyof typeof o]][]).map(([k, v]) => console.log(v[k]));
// Assert based on runtime logic
es.map(([k, v]) => console.log((v as { [K in typeof k]: unknown })[k]));

// The real type depends on runtime logic unable to be inferred at compile time.
(document.getElementById("name") as HTMLInputElement).value = "Brad";
```
