---
title: NestJS TypeScript Configuration
date: 2026-06-23
tags:
  - nestjs
  - typescript
  - tsconfig
  - configuration
  - learning
source: NestJS tutorial — educational reference
---

## Why `tsconfig.json` matters

TypeScript cannot run directly in Node.js. `tsconfig.json` tells the TypeScript compiler how to transform `.ts` source files into executable JavaScript. For NestJS it is especially important because decorators (`@Controller`, `@Injectable`, `@Module`) rely on two specific compiler flags: `experimentalDecorators` and `emitDecoratorMetadata`.

## Main approaches

### 1. Module system: CommonJS vs ESM

NestJS supports both module systems. The choice affects `tsconfig.json`, `package.json`, import syntax, and runtime behavior.

| Aspect | CommonJS (legacy) | ESM / NodeNext (modern) |
|--------|-------------------|-------------------------|
| Output syntax | `require` / `module.exports` | `import` / `export` |
| `tsconfig` module | `"commonjs"` | `"NodeNext"` |
| `tsconfig` moduleResolution | `"node"` (or omitted) | `"NodeNext"` |
| `package.json` | `"type": "commonjs"` or omitted | `"type": "module"` |
| `__dirname` / `require` | Available | Not available by default |
| Path aliases | `tsconfig-paths` at runtime | Subpath imports or loader |
| Top-level await | No | Yes |
| Package interop | Easier with older packages | May need named imports |

**When to choose CommonJS:** legacy codebases, packages without ESM support, or when you want the simplest default setup.

**When to choose ESM:** modern Node.js projects, libraries you plan to publish, or when you want strict module resolution and top-level await.

### 2. Strictness: lenient vs strict

#### Lenient style

This is what `nest new` generates. It prioritizes getting started quickly over type safety.

```json
{
  "strict": false,
  "strictNullChecks": false,
  "noImplicitAny": false,
  "strictBindCallApply": false
}
```

**When to use it:** learning, prototypes, migrating from JavaScript.

**Trade-off:** TypeScript catches fewer errors at compile time. Runtime surprises are more likely.

#### Strict style

Enables the full `strict` family of checks.

```json
{
  "strict": true,
  "strictNullChecks": true,
  "noImplicitAny": true,
  "strictBindCallApply": true
}
```

**When to use it:** production applications, team projects, or whenever you want the compiler to act as a safety net.

**Trade-off:** More upfront type annotations, but fewer bugs reach production.

### 3. Monorepo / composite projects

Uses `composite: true`, project references, and shared base configs. Useful when the repository contains multiple NestJS apps or libraries.

**When to use it:** NX/Nest monorepos, large teams, or shared internal packages.

**Trade-off:** More complex configuration and build orchestration.

## Key compiler options explained

### `target`

The JavaScript language version the compiler emits.

- `"ES2021"` — safe for older Node.js LTS versions
- `"ES2022"` — pairs well with `"module": "NodeNext"`
- `"ESNext"` — newest features; use only if you control the runtime

**What to pay attention to:** Newer targets produce cleaner output but may not run on older Node versions.

### `module` and `moduleResolution`

Control the emitted module system and how TypeScript resolves imports.

| `module` | `moduleResolution` | Use case |
|----------|-------------------|----------|
| `"commonjs"` | `"node"` | Legacy Node.js projects |
| `"NodeNext"` | `"NodeNext"` | Native ESM with Node.js |
| `"ES2022"` | `"bundler"` | Bundled frontends or libraries |

**What to pay attention to:** With `"module": "NodeNext"`, you must also set `"moduleResolution": "NodeNext"` and `"type": "module"` in `package.json`.

### `outDir` and `rootDir`

- `outDir`: where compiled `.js` files are placed (commonly `"./dist"`).
- `rootDir`: the root folder of the TypeScript source (commonly `"./src"`).

**What to pay attention to:** Keep `dist/` out of version control. Files outside `rootDir` can cause compiler errors if they are included.

### `baseUrl` and `paths`

Used for compile-time path aliases:

```json
"baseUrl": "./",
"paths": {
  "@/*": ["src/*"]
}
```

With this alias, `import { UserService } from '@/users/user.service'` resolves to `src/users/user.service` during type checking.

**What to pay attention to:** `paths` does not rewrite imports in the emitted JavaScript. At runtime you need:
- `tsconfig-paths` or `module-alias` for CommonJS
- Node.js subpath imports or a loader for ESM

### `experimentalDecorators` and `emitDecoratorMetadata`

- `experimentalDecorators`: enables decorator syntax (`@Controller()`, `@Injectable()`)
- `emitDecoratorMetadata`: emits type metadata so NestJS can resolve dependencies by type

**What to pay attention to:** `emitDecoratorMetadata` is useless without `import 'reflect-metadata'` at the top of `main.ts`.

### `esModuleInterop` and `allowSyntheticDefaultImports`

Allows default imports from CommonJS modules:

```ts
import express from 'express'; // instead of import * as express from 'express'
```

**What to pay attention to:** Most modern NestJS projects enable both. They reduce friction when importing older packages.

### `strict`

A shorthand that enables all strict type-checking options.

**What to pay attention to:** Enable `strict` as early as possible. Switching it on later in a large codebase can produce many errors at once.

### `sourceMap`

Generates `.js.map` files that map compiled JavaScript back to TypeScript source.

**What to pay attention to:** Essential for debugging with breakpoints in original `.ts` files.

### `skipLibCheck`

Skips type checking of `.d.ts` files in `node_modules`.

**What to pay attention to:** Speeds up compilation and avoids unrelated errors from dependencies. Keep it on unless you need to debug a specific declaration file.

### `resolveJsonModule`

Allows importing JSON files:

```ts
import config from './config.json';
```

**What to pay attention to:** Useful for configuration files. Required in ESM when importing JSON.

## Path alias strategies

### Strategy 1: TypeScript `paths` + `tsconfig-paths` (CommonJS)

`tsconfig.json`:
```json
"baseUrl": "./",
"paths": {
  "@/*": ["src/*"]
}
```

`package.json`:
```json
"scripts": {
  "start:dev": "ts-node -r tsconfig-paths/register src/main.ts",
  "start": "node -r tsconfig-paths/register dist/main.js"
}
```

Install: `npm install -D tsconfig-paths`

### Strategy 2: Node.js subpath imports (ESM)

`package.json`:
```json
"type": "module",
"imports": {
  "#src/*": "./dist/*"
}
```

Usage:
```ts
import { UserService } from '#src/users/user.service';
```

**What to pay attention to:** Subpath imports must start with `#`. TypeScript 5+ with `"moduleResolution": "NodeNext"` can resolve them from `package.json`.

### Strategy 3: Relative imports

No configuration needed:

```ts
import { UserService } from '../users/user.service';
```

**What to pay attention to:** Simplest for learning and small projects. Refactoring becomes harder as the project grows.

## Example configurations

### Example A: CommonJS lenient (closest to `nest new`)

```json
{
  "compilerOptions": {
    "target": "ES2021",
    "module": "commonjs",
    "lib": ["ES2021"],
    "outDir": "./dist",
    "rootDir": "./src",
    "baseUrl": "./",
    "paths": { "@/*": ["src/*"] },
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "removeComments": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
    "strict": false,
    "strictNullChecks": false,
    "noImplicitAny": false,
    "strictBindCallApply": false,
    "forceConsistentCasingInFileNames": false,
    "noFallthroughCasesInSwitch": false,
    "skipLibCheck": true,
    "resolveJsonModule": true,
    "incremental": true,
    "tsBuildInfoFile": "./dist/.tsbuildinfo"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.spec.ts", "**/*.test.ts"]
}
```

`package.json`:
```json
{
  "type": "commonjs",
  "main": "dist/main.js"
}
```

### Example B: CommonJS strict

Same as Example A, but with:

```json
"strict": true,
"strictNullChecks": true,
"noImplicitAny": true,
"strictBindCallApply": true,
"forceConsistentCasingInFileNames": true,
"noFallthroughCasesInSwitch": true
```

### Example C: ESM / NodeNext strict

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "lib": ["ES2022"],
    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "removeComments": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
    "strict": true,
    "strictNullChecks": true,
    "noImplicitAny": true,
    "strictBindCallApply": true,
    "forceConsistentCasingInFileNames": true,
    "noFallthroughCasesInSwitch": true,
    "skipLibCheck": true,
    "resolveJsonModule": true,
    "incremental": true,
    "tsBuildInfoFile": "./dist/.tsbuildinfo"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.spec.ts", "**/*.test.ts"]
}
```

`package.json`:
```json
{
  "type": "module",
  "main": "dist/main.js"
}
```

### Example D: ESM with subpath imports

`tsconfig.json`:
Same as Example C.

`package.json`:
```json
{
  "type": "module",
  "main": "dist/main.js",
  "imports": {
    "#src/*": "./dist/*"
  }
}
```

Usage in code:
```ts
import { UserService } from '#src/users/user.service';
```

## How to choose

| Scenario | Recommended config |
|----------|-------------------|
| Learning NestJS basics | Example A (CommonJS lenient) |
| Production API | Example B or C (strict) |
| Modern Node.js, library publishing | Example C or D (ESM) |
| Large monorepo | Composite projects with shared base configs |

## Common pitfalls

1. **Missing `reflect-metadata` import** — `emitDecoratorMetadata` only works if `reflect-metadata` is imported in `main.ts`.
2. **Path aliases work at compile time but fail at runtime** — always configure a runtime resolver.
3. **Forgetting `"type": "module"`** — with `"module": "NodeNext"`, Node.js still treats `.js` files as CommonJS unless `type` is set.
4. **ESM and `__dirname`** — use `import.meta.url` or `createRequire(import.meta.url)` instead.
5. **`moduleResolution` mismatch** — `"module": "NodeNext"` requires `"moduleResolution": "NodeNext"`.

## What to pay attention to next

1. Create the `src/` directory.
2. Add `import 'reflect-metadata'` at the top of `src/main.ts`.
3. Choose a module system and stick to it for the whole project.
4. Add build/start scripts to `package.json`:
   - CommonJS: `"build": "tsc"`, `"start": "node dist/main.js"`
   - ESM: `"build": "tsc"`, `"start": "node dist/main.js"`
5. For dev hot-reload, consider `tsx`, `ts-node` with ESM loader, or Nest CLI.

## Related notes

- [[NestJS Project Initialization]]
