---
title: 2.8.6. Decorators
---

Decorators are inherent in functional languages including JavaScript, also known as Higher-Order Functions (HOF), i.e. functions taking other functions as parameters, which forms a design pattern — decorators provide certain logic that is additional to and reusable across different functions. In functional languages, a decorator typically returns a new function with the same interface as the one to be decorated. In the class-based paradigm, decorators are reusable across classes, implemented via metaprogramming in compiled languages. Compared to templates — another typical metaprogramming technique that is completely static with heavy rigid type contracts and potential code bloat, compilers only emit metadata for decorators (annotations in Java or attributes in C#), while the binding can still be dynamically executed at runtime by frameworks. Furthermore, in modern Java and C#, frameworks can leverage compile-time tools (Java Annotation Processors or Roslyn Source Generators in C#) to generate hardcoded wired-up code at compile time, while compilers use those framework-provided generators to complete the binding entirely during compilation, retaining declarative elegance while enabling reflection-free execution and native AOT compilation.

JavaScript also supports the class-based paradigm, but JavaScript classes are essentially functions defined and executed at runtime. In TypeScript, therefore, decorators are still functions applied on classes (properties, methods, or parameters). Frameworks can still use decorator functions to wire the decorated entities up, while the compiler only generates the calling code, which gets executed with the class definition. Although modern TypeScript has supported ECMAScript Stage 3 decorators natively, the legacy one with the `experimentalDecorators` compiler option is still used in major frameworks such as Angular and NestJS since the native ECMAScript one adopts different function signatures and lacks parameter decorators.

```json title="tsconfig.json"
{
  "compilerOptions": {
    ...
    "experimentalDecorators": true,
    ...
  },
  ...
}
```

```typescript title="src/decorator.ts"
// Class decorator
/* eslint-disable @typescript-eslint/no-explicit-any */
function Logger<T extends { new (...args: any[]): {} }>(constructor: T) {
  console.log(constructor, "defined");
  return class extends constructor {
    constructor(...args: any[]) {
      // eslint-disable-next-line @typescript-eslint/no-unsafe-argument
      super(...args);
      console.log(`Object of`, constructor, "constructed");
    }
  };
}
/* eslint-enable @typescript-eslint/no-explicit-any */

/* Usage
@Logger
class MyClass {}
// Compiled like
//let MyClass = class MyClass {};
//MyClass = Logger(MyClass) || MyClass;
*/

// Property decorator
function LogProperty(target: object, key: string) {
  console.log(`Property "${key}" defined on`, target.constructor);
}

// Method decorator
function LogMethod(target: object, key: string, descriptor: PropertyDescriptor) {
  console.log(`Method "${key}" defined on`, target.constructor);
  const original = descriptor.value as Function;
  descriptor.value = function (...args: unknown[]) {
    console.log(`Method "${key}" called with`, args);
    return original.apply(this, args) as unknown;
  };
}

// Param decorator
function LogParam(target: object, key: string, paramIndex: number) {
  console.log(`Param ${paramIndex} defined in method "${key}" on`, target.constructor);
}

@Logger
class MyClass {
  @LogProperty
  prop: string = "";

  @LogMethod
  greet(@LogParam name: string) {
    console.log(`Hello, ${name}!`);
  }
}

((x: MyClass) => x.greet("decorators"))(new MyClass());
```

```bash
npx tsc
node dist/decorator.js
# Output
Property "prop" defined on [class MyClass]
Param 0 defined in method "greet" on [class MyClass]
Method "greet" defined on [class MyClass]
[class MyClass] defined
Object of [class MyClass] constructed
Method "greet" called with [ 'decorators' ]
Hello, decorators!
```

Reusability across classes makes decorators ideal for framework customisation with declarative elegance. Dependency Injection (DI) is the quintessential pattern in decorator-based frameworks, responsible for instantiation, lifecycle management, and wiring-up of all injectable objects. Although it is not necessary in principle, TypeScript DI frameworks like NestJS typically rely on the `emitDecoratorMetadata` compiler option, which makes the compiler emit calls like `Reflect.metadata(...)` for decorated entities. Note that `Reflect.metadata(...)` relies on the `reflect-metadata` polyfill package, which must be enabled (`import "reflect-metadata"`) at application entry. As a result, those emitted calls will generate corresponding metadata (e.g. `design:paramtypes` for constructor parameter types) at runtime. Based on this, frameworks can inspect dependency signatures from constructor metadata via `Reflect.getMetadata(...)`, recursively instantiate required dependencies and wire them up. In contrast, with its own AOT compiler handling DI metadata and wiring-up at build time, modern Angular has abandoned `emitDecoratorMetadata` and runtime reflection to improve startup performance and tree-shaking.

Note that TypeScript touches neither JavaScript's value-based dynamic typing nor imposes the class-based paradigm. In other words, TypeScript does not create a new compiled language, and "transpile" might be less misleading than "compile" in this term.
