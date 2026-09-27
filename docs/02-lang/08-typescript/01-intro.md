---
title: 2.8.1 - Introduction
---

TypeScript is a typed superset of JavaScript, eventually compiled (actually transpiled, from source code to source code rather than binary) to plain JavaScript. TypeScript introduces a static type system via inference and annotations, to enforce type safety at compile time. The type system is also helpful for IDE IntelliSense.

```bash
npm init -y
npm install --save-dev typescript
# Generate tsconfig.json in the CWD. Note that tsc can then run in sub-dirs and find tsconfig.json
# upwards.
npx tsc --init
```

```json title="tsconfig.json"
{
  "compilerOptions": {
    "target": "ES2023",
    // Adopt the latest Node module detection and emission
    "module": "nodenext",
    "moduleResolution": "nodenext",
    "esModuleInterop": true,
    // `exports` in package.json of libs
    "resolvePackageJsonExports": true,
    "resolveJsonModule": true,
    // `import <name> from "<module>"` for `import * as <name> from "<module>"`
    "allowSyntheticDefaultImports": true,
    "forceConsistentCasingInFileNames": true,
    // Ensure other compilers like SWC can operate on a single file at a time
    "isolatedModules": true,
    "skipLibCheck": true,

    // .tsbuildinfo
    "incremental": true,
    // Generate type declaration files (.d.ts)
    "declaration": true,
    // Generate source map files (.js.map) for debuggers or dev tools to report source info like
    // filenames and line numbers for stack traces
    "sourceMap": true,
    "removeComments": true,
    // Input root. Note that paths are relative to the project root where tsconfig.json is.
    "rootDir": "src",
    // Output root. Note that the input dir structure under the input root will be kept under the
    // output root.
    "outDir": "dist",

    // alwaysStrict, strictNullChecks, strictBindCallApply, strictBuiltinIteratorReturn,
    // strictFunctionTypes, strictPropertyInitialization,
    // noImplicitAny, noImplicitThis, useUnknownInCatchVariables
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedSideEffectImports": true
  },
  // Source files to be compiled
  "include": ["src"]
}
```

```typescript title="src/app.ts"
interface User {
  id: number;
  name: string;
}

function hello(user: User) {
  return `Hello, ${user.name}!`;
}

const brad = { id: 0, name: "Brad" };
console.log(hello(brad));
```

```bash
npm install --save-dev @types/node
npx tsc
node dist/app.js
node --enable-source-maps dist/app.js
```
