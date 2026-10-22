---
status: stable
mastery: beginner
tags: [nestjs, backend, initialization, learning]
created: 2026-06-23
updated: 2026-10-21
---

# NestJS Project Initialization

---
title: NestJS Project Initialization
date: 2026-06-23
tags:
  - nestjs
  - backend
  - initialization
  - learning
source: NestJS tutorial — scratch project setup
---

## Why we start here

Before writing any controller or service, NestJS needs a runtime foundation. This step installs the smallest set of packages that let a NestJS application boot: the framework core, the HTTP adapter, metadata reflection, reactive streams, and TypeScript support.

## What each package does

### 1. `@nestjs/common`

The public face of NestJS. It provides the decorators and building blocks you use every day:

- `@Controller`, `@Get`, `@Post`, `@Body`, `@Param`, `@Query`, `@Module`, `@Injectable`
- Pipes, guards, interceptors, filters, exceptions (`HttpException`, `BadRequestException`)
- Utility types like `NestMiddleware`, `ExecutionContext`, `Logger`

**What to pay attention to:** Most of your application code will import from this package. If you cannot import a common decorator, you probably forgot to install or import `@nestjs/common`.

---

### 2. `@nestjs/core`

The framework engine. It wires everything together at runtime:

- `NestFactory` — creates the application instance (`NestFactory.create(AppModule)`)
- Module scanner and dependency injection container
- Lifecycle hooks (`OnModuleInit`, `OnApplicationBootstrap`, `OnModuleDestroy`)
- Exception zones and application context

**What to pay attention to:** You rarely import from `@nestjs/core` directly in feature code, but `main.ts` always starts with `NestFactory.create(...)`. Errors during startup usually originate here.

---

### 3. `@nestjs/platform-express`

The default HTTP server adapter. NestJS is platform-agnostic at the core; this package bridges NestJS to Express:

- Creates the underlying Express instance
- Handles HTTP requests/responses
- Enables Express middleware (`app.use(...)`)

**What to pay attention to:** Other adapters exist (`@nestjs/platform-fastify`, `@nestjs/platform-ws`). For most tutorials and APIs, Express is the default and safest choice. If you need Fastify later, you swap only this adapter.

---

### 4. `reflect-metadata`

A runtime polyfill that makes TypeScript decorators store and expose metadata:

- Required for `@Module`, `@Controller`, `@Injectable`, and parameter decorators (`@Body`, `@Param`)
- Without it, NestJS cannot read constructor parameter types or decorator metadata

**What to pay attention to:** You must import it once at the very top of `main.ts` (or enable `emitDecoratorMetadata` in `tsconfig.json`). Missing `reflect-metadata` produces cryptic dependency-injection errors.

---

### 5. `rxjs`

The reactive programming library NestJS uses for streams and async operations:

- `Observable`, `Subject`, `BehaviorSubject`
- Used by interceptors, gateways, and some internal APIs
- A peer dependency of `@nestjs/common` and `@nestjs/core`

**What to pay attention to:** Many NestJS APIs return Promises, but some (interceptor `next.handle()`, WebSocket gateways) return Observables. If you forget to subscribe or convert with `lastValueFrom`, the code may silently do nothing.

---

### 6. `typescript`

The TypeScript compiler and language services:

- Compiles `.ts` files to JavaScript
- Enables decorator syntax (`experimentalDecorators`, `emitDecoratorMetadata`)
- Provides type checking during development

**What to pay attention to:** TypeScript is a dev dependency. It is not needed at runtime. A proper `tsconfig.json` is required next so the compiler understands decorators and Node.js module resolution.

---

### 7. `@types/node` (supporting package)

Type definitions for Node.js built-in modules (`http`, `fs`, `process`, `__dirname`):

- Allows TypeScript to understand Node.js globals
- Required whenever you use TypeScript with a Node.js runtime

**What to pay attention to:** Without it, TypeScript will complain about `process`, `Buffer`, or `__dirname` even though they exist at runtime.

## Commands used

```bash
npm init -y
npm install @nestjs/common @nestjs/core @nestjs/platform-express reflect-metadata rxjs
npm install -D typescript @types/node
```

## What to pay attention to next

- `tsconfig.json` must enable decorators:
  - `experimentalDecorators: true`
  - `emitDecoratorMetadata: true`
- `reflect-metadata` must be imported in `main.ts` before `NestFactory.create`.
- Keep `@nestjs/*` versions in sync to avoid peer-dependency warnings.

## Related notes

- [[NestJS — Архитектура и Точка Входа (Auth Academy Stage 0)]]

