---
title: 2.8.5. JavaScript and Type Declarations
---

TypeScript supports `import` and `export` syntax, which will be compiled according to the `module` compiler option. For example, `nodenext` indicates the latest Node module detection and emission, i.e. depending on `"type": "module"` in `package.json` and module file extensions. Although with the `allowJs` compiler option enabled, JavaScript files can also be imported in TypeScript, with JSDoc comments for type inference, type declaration files (`.d.ts`) are normally used to provide type support for JavaScript files. When importing a JavaScript file, the corresponding type declaration file (e.g. `lib/hello.d.ts` for `lib/hello.js`) will be included as well. A package (installed in `node_modules/`) can specify its default type declaration file via `types` in its `package.json`. Note that even TypeScript projects are released and used as compiled JavaScript packages, and corresponding type declaration files can be generated via the `declaration` compiler option. As for packages written in JavaScript, `node_modules/@types/` (the `typeRoots` compiler option) can be used for corresponding type declaration files (e.g. `node_modules/@types/hello/index.d.ts` for `node_modules/hello/index.js`). The `@types` namespace is maintained by the Definitely Typed community to provide type declaration files for many commonly used JavaScript packages without their own type support.

```typescript title="*/hello.js or */hello/index.js"
function hello(user) {
  return `Hello, ${user.name}!`;
}

module.exports = { hello };
```

```typescript title="*/hello.d.ts or */hello/index.d.ts"
export interface User {
  id: number;
  name: string;
}

export function hello(user: User): string;
```

```typescript title="src/app.ts"
/* `lib/hello.js` with `lib/hello.d.ts`
import { hello } from "../lib/hello";
*/
/* `hello/index.js` with `hello/index.d.ts`
import { hello } from "../hello";
*/
/* `node_modules/lib/hello.js` with `node_modules/@types/lib/hello.d.ts`
import { hello } from "lib/hello";
*/
/* `node_modules/hello/index.js` with `node_modules/@types/hello/index.d.ts` */
import { hello } from "hello";

const brad = { id: 0, name: "Brad" };
console.log(hello(brad));
```

While some TypeScript definitions like `enum` and `class` will be compiled to runtime JavaScript data or code, declarations only exist at compile time without affecting any runtime behaviour. Declarations can also be used to augment existing types that support declaration merging, such as interfaces. Additionally, types can be declared globally, and if there is no top-level `import` or `export` used, all top-level types will be global by default. Global types can be used without being imported, called ambient types. Note that all type declaration files included in a TypeScript project are compiled as source files, and the global types that they declare then become ambient automatically. Furthermore, all packages in `node_modules/@types/` are included in compilation as well by default, or only those specified by the `types` compiler option will be included. Note that all those packages are actually included via their default type declaration files (`index.d.ts` or specified by `types` in `package.json`). In addition, `/// <reference types="..." />` or `/// <reference path="..." />` can be used to explicitly include packages or type declaration files.

```typescript title="src/global.d.ts"
import Hello from "hello";

// For global ones since there's already top-level `import`
declare global {
  // New ambient type as there's no `User` as a global one yet
  interface User extends Hello.User {
    password?: string;
  }
}

// For a module other than the corresponding one of this
declare module "hello" {
  // Augment (via declaration merging) the one already existing in the module
  interface User {
    password?: string;
  }
}
```

```typescript title="src/app.ts"
/* There's an ambient `User` now, but the `User` from module `hello` can still be imported.
import { User } from "hello";
*/

// Both the ambient and module `User` are augmented with `password`.
((x: User) => console.log(x.id, x.name, x.password))({ id: 0, name: "Brad" });
```

:::note
Globals from packages in `node_modules/@types/`

```typescript title="node_modules/@types/hello/index.d.ts"
// Include `node_modules/@types/hello/global.d.ts` from the compilation entry of this package
/// <reference path="global.d.ts" />
...
```

:::
