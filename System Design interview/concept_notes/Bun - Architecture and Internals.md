---
status: new
tags: [bun, javascript, typescript, runtime, zig, javascriptcore, backend, nodejs]
source: Bun official docs (bun.sh), GitHub bun repository, various benchmarks and deep-dive articles
created: 2026-06-05
difficulty: intermediate
---

# 🧠 Topic: Bun - Architecture and Internals

> [!abstract] Executive Summary (ELI5)
> Imagine you want to build a website. Normally you need a whole toolbox: a screwdriver (runtime), a hammer (package manager), a wrench (bundler), and a level (test runner). **Bun is a Swiss Army knife** — one tool that does all of these jobs, and it is forged from lighter, sharper metal (the Zig programming language and Apple's JavaScriptCore engine) so it starts instantly and cuts through tasks faster than the older, heavier toolbox.

---

## 🎯 The "Why"

### Problem Solved
By 2022, the JavaScript ecosystem had accumulated decades of technical debt:

1. **Tooling sprawl:** To build a modern JS project you needed: Node.js (runtime) + npm/yarn/pnpm (package manager) + webpack/rollup/esbuild/vite (bundler) + jest/vitest/mocha (test runner) + tsc/ts-node (TypeScript compiler) + eslint/prettier (linting/formatting). Each tool has its own config file, dependencies, and startup overhead.

2. **Slow startup:** Node.js + V8 take 100-300ms just to boot. For serverless functions (AWS Lambda, Cloudflare Workers, Vercel Edge) where you pay per millisecond and cold starts matter, this is expensive.

3. **Configuration hell:** Getting TypeScript, JSX, and modern ESM to work together in Node.js requires tsconfig.json, package.json type fields, possibly babel or swc, and careful module resolution dance between CommonJS and ESM.

4. **npm install is slow:** Even with pnpm's optimizations, installing 1000+ dependencies from a complex package.json takes significant time in CI/CD.

**Bun attacks all four problems simultaneously:**
- One binary replaces 5-7 separate tools
- Startup in ~20-50ms (vs 100-300ms for Node)
- TypeScript and JSX work out of the box with zero configuration
- `bun install` is often 20-100x faster than `npm install`

### Alternatives
- **Node.js** — the incumbent; largest ecosystem, 15+ years of maturity
- **Deno** — another modern runtime by Ryan Dahl (creator of Node.js); also TypeScript-native but stricter security model and smaller ecosystem
- **WinterCG** (Web-interoperable Runtimes Community Group) — not a runtime but a standard that both Node.js, Deno, and Bun implement for edge compatibility
- **Just-in-time compiled runtimes** like QuickJS (embedded, very lightweight but much slower)

### Trade-offs
| Aspect | Bun | Node.js | Deno |
|--------|-----|---------|------|
| **Speed** | Fastest startup & runtime | Good, well-optimized | Good, comparable to Node |
| **Ecosystem** | npm-compatible (growing) | Massive (de-facto standard) | Smaller, own registry |
| **Maturity** | v1.0 in 2023, evolving fast | 15+ years, rock-solid | 5+ years, stable |
| **TypeScript** | Native, zero config | Requires ts-node/tsc | Native, zero config |
| **Security** | Standard | Standard | Permission model by default |
| **Compatibility** | ~90-95% Node API | 100% (obviously) | Different module system |
| **Implementation** | Zig (modern systems lang) | C++ | Rust |
| **JS Engine** | JavaScriptCore (JSC) | V8 | V8 |

> [!tip] Senior Insight
> Bun's primary value proposition is **developer velocity and reduced toolchain complexity**, not just raw speed. The speed is the hook; the unified toolchain is the real game-changer.

---

## 📜 History

| Year | Milestone |
|------|-----------|
| **2021** | Jarred Sumner starts working on Bun as a side project. Frustrated with slow JS tooling, he decides to rebuild the entire toolchain from scratch using Zig and JavaScriptCore. |
| **July 2022** | Bun is publicly released (v0.1.0) with a simple HTTP server benchmark showing 4x speedup over Node.js. Goes viral in the JS community. |
| **2022-2023** | Rapid iteration: package manager (`bun install`), bundler (`bun build`), test runner (`bun test`), SQLite support (`bun:sqlite`), WebSocket server, file watcher, hot reload. |
| **September 2023** | **Bun v1.0** — declared production-ready. Full npm compatibility, stable APIs, and enterprise support offerings announced. |
| **2024-2025** | Maturation: improved Node.js compatibility, Windows support, package workspace support, Bun APIs for embedding, enterprise features, and continued performance improvements. |
| **2025+** | Positioned as the " batteries-included" JavaScript runtime. Growing adoption in edge computing, serverless, and developer tooling. |

---

## 🏗️ What Bun Is Made Of

Bun is fundamentally different from Node.js in its implementation choices. Where Node.js is a C++ application embedding V8 and using libuv, Bun is a **Zig application** embedding JavaScriptCore with a **custom I/O layer**.

```
┌─────────────────────────────────────────────────────────────┐
│                    Your JavaScript/TypeScript Code           │
│              (.js, .ts, .jsx, .tsx — all native)            │
├─────────────────────────────────────────────────────────────┤
│  Bun Standard Library (JS/TS layer)                          │
│  - bun:sqlite, bun:ffi, bun:jsc, WebSocket, file I/O        │
├─────────────────────────────────────────────────────────────┤
│  Bun Runtime Core (Zig + C layer)                            │
│  - Module resolution, bundler, package manager logic          │
│  - HTTP server, WebSocket implementation                      │
│  - Transpiler (TS → JS, JSX → JS)                            │
├─────────────────────────────────────────────────────────────┤
│  ┌────────────────────┐    ┌─────────────────────────────┐ │
│  │ JavaScriptCore     │    │    Bun's Custom I/O Layer   │ │
│  │ (JSC / WebKit)     │    │    (Zig, NOT libuv)         │ │
│  │                    │    │                              │ │
│  │ - Parses & JITs    │    │  - Event Loop                │ │
│  │   JS/TS to         │    │  - Network I/O (epoll,       │ │
│  │   machine code     │    │    kqueue, IOCP)             │ │
│  │ - 3-tier GC        │    │  - File I/O                  │ │
│  │   (Eden,           │    │  - Timers                    │ │
│  │    Survivor,       │    │  - Thread Pool               │ │
│  │    Old/Marked)     │    │  - DNS resolution            │ │
│  └────────────────────┘    └─────────────────────────────┘ │
├─────────────────────────────────────────────────────────────┤
│  Operating System (Linux/macOS/Windows)                      │
└─────────────────────────────────────────────────────────────┘
```

### 1. JavaScriptCore (JSC) — The Engine
Bun uses **JavaScriptCore** (also known as Nitro, SquirrelFish, or simply JSC), the engine that powers Safari and all WebKit-based browsers.

**Why not V8?**
- JSC has a **faster startup time** than V8 because its baseline JIT (LLInt / Baseline JIT) is lighter and quicker to initialize
- JSC's architecture is more amenable to embedding in applications that need fast cold starts
- Jarred Sumner specifically chose JSC for edge computing scenarios where startup latency matters more than sustained peak throughput
- V8 is heavily optimized for long-running browser tabs; JSC is optimized for a broader range of execution profiles

**JSC Architecture:**
```
JS Source Code
      │
      ▼
┌─────────────┐
│   Parser    │ → Abstract Syntax Tree (AST)
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Bytecode   │ → LLInt (Low Level Interpreter)
│  Generation │   Fast startup, low memory
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Baseline JIT│ → First-tier compilation, moderate speed
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  DFG JIT    │ → Data Flow Graph JIT, speculative optimization
│ (Optimizing)│   Type profiling, inlining, loop unrolling
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  FTL JIT    │ → Faster Than Light JIT, LLVM-based
│ (Max Opt)   │   Aggressive optimization for hot code
└─────────────┘
```

JSC uses a **4-tier execution strategy**: LLInt (interpreter) → Baseline JIT → DFG JIT → FTL JIT. This is similar to V8's Ignition → TurboFan but with an extra tier and different optimization tradeoffs.

**JSC Garbage Collector:**
- **Generational** like V8 but with different heuristics
- **Concurrent GC**: JSC can collect garbage while JavaScript is executing, reducing pause times
- **Incremental marking**: Spreads GC work across time slices instead of stopping the world

### 2. Zig — The Implementation Language
Bun is written primarily in **Zig**, a modern systems programming language created by Andrew Kelley.

**Why Zig instead of C++ or Rust?**
- **Explicit memory control**: Zig has no hidden allocations. Every memory allocation is explicit and can be tracked, which is critical for a runtime that needs predictable performance
- **Comptime**: Zig's metaprogramming is done at compile time (comptime), generating optimized code without runtime overhead
- **C interoperability**: Zig can import C headers directly (`@cImport`) and compile alongside C/C++ code seamlessly
- **Simplicity**: Zig is a smaller language than C++ or Rust, with fewer abstractions getting in the way of low-level optimization
- **No hidden control flow**: No operator overloading, no exceptions, no default destructors — what you see is what executes

**Zig code example (simplified):**
```zig
// Explicit allocator — no hidden heap allocations
pub fn readFile(allocator: std.mem.Allocator, path: []const u8) ![]u8 {
    const file = try std.fs.cwd().openFile(path, .{});
    defer file.close(); // deterministic cleanup, no RAII magic
    
    const stat = try file.stat();
    const contents = try allocator.alloc(u8, stat.size);
    _ = try file.readAll(contents);
    return contents;
}
```

### 3. Custom I/O Layer — Not libuv
Unlike Node.js which uses libuv, Bun implements its own I/O layer in Zig.

**Why build from scratch?**
- libuv was designed for Node.js's specific needs and carries compatibility constraints
- Bun's I/O layer is optimized for Bun's specific architecture and can make different tradeoffs
- Direct control over epoll (Linux), kqueue (macOS/BSD), and IOCP (Windows) allows finer optimization
- The I/O layer is tightly integrated with JSC's event loop rather than being a separate abstraction

**Architecture:**
- **Event Loop**: Single-threaded, similar in concept to Node.js but implemented differently
- **Thread Pool**: For file system operations and CPU-intensive tasks
- **Network I/O**: Direct async socket operations using OS-specific APIs
- **File I/O**: Uses `io_uring` on modern Linux (when available), falling back to thread pool on older systems

> [!tip] Key Insight
> Bun's use of **io_uring** (Linux's new async I/O API) for file operations on modern kernels gives it a significant performance advantage over libuv-based file I/O, which cannot use io_uring directly.

---

## 💡 Mental Model & Analogy

### Analogy: Bun Is a Formula 1 Pit Crew

Imagine a Formula 1 race where every millisecond counts.

**Node.js** is like a traditional garage with separate specialists:
- Engine tuner (V8)
- Tire specialist (npm)
- Fuel engineer (webpack)
- Aerodynamicist (jest)
- Data analyst (tsc)

Each expert is world-class at their job, but they work in separate bays, pass papers between them, and have different toolboxes. A pit stop takes 3-4 seconds.

**Bun** is like a **revolutionary single-pit-crew unit**:
- One team trained to do ALL jobs
- One unified set of tools optimized to work together
- Pre-positioned everything — tires, fuel, adjustments happen simultaneously
- Pit stop: 1.8 seconds

The individual specialists (V8, npm, webpack) are not bad — they are excellent. But the **systemic overhead of coordination** between them is what Bun eliminates. Bun's speed comes not just from a faster engine (JSC vs V8) but from removing all the handoff delays between tools.

**When it breaks:**
If your car needs a very specific, rare part that only the traditional garage's specialist knows how to install (a niche npm package with native C++ addons that only work with Node.js/V8), the unified crew might struggle because they optimized for the common case, not the exotic one.

---

## 🛠️ Core Principles & How It Actually Works

### The Unified Toolchain

Bun's biggest architectural difference from Node.js is that it is not just a runtime — it is a **complete development environment**.

```bash
# What used to require 5+ separate tools:
node server.js              # Node runtime
npm install                 # Package manager
npx tsc                     # TypeScript compiler
npx jest                    # Test runner
npx webpack                 # Bundler

# In Bun, it's one binary:
bun run server.ts           # Run TypeScript directly
bun install                 # Install dependencies (uses bun.lockb)
bun test                    # Run tests
bun build index.ts          # Bundle for production
```

### Native TypeScript & JSX

Bun's **transpiler** is built into the runtime and executes at startup:

```typescript
// This file is server.ts — Bun runs it directly, no tsconfig needed
import { serve } from "bun";

serve({
  port: 3000,
  fetch(req: Request): Response {
    return new Response("Hello from TypeScript!");
  },
});
```

Bun's transpiler:
1. Parses TypeScript/JSX into an AST
2. Strips type annotations (TypeScript types are erased, not checked)
3. Transforms JSX (`<div />` → `React.createElement('div')` or custom pragma)
4. Outputs JavaScript that JSC executes

> [!warning] Important Distinction
> Bun **transpiles** TypeScript but does **NOT type-check** it. Type errors do not prevent execution. For type checking, you still need `tsc --noEmit`.

```bash
# Run (transpiles and executes, ignores type errors)
bun run app.ts

# Type-check separately
bun x tsc --noEmit
```

### bun: APIs — Bun's Native Modules

Bun provides its own set of built-in modules, prefixed with `bun:`:

```typescript
import { Database } from "bun:sqlite";
import { dlopen, FFIType } from "bun:ffi";
import { gzipSync } from "bun";

// SQLite — built-in, no npm install needed
const db = new Database("app.db");
const users = db.query("SELECT * FROM users").all();

// FFI — call C libraries directly
const lib = dlopen("./libfoo.so", {
  add: { args: [FFIType.i32, FFIType.i32], returns: FFIType.i32 },
});
console.log(lib.symbols.add(1, 2)); // 3

// Compression — native, fast
const compressed = gzipSync(Buffer.from("hello world"));
```

Key `bun:` modules:
- **`bun:sqlite`** — high-performance SQLite wrapper (faster than `better-sqlite3`)
- **`bun:ffi`** — Foreign Function Interface to call C libraries without writing C++ addons
- **`bun:jsc`** — Access to JavaScriptCore internals (advanced use cases)
- **`bun:test`** — Built-in test runner (Jest-compatible API)

### Package Manager: bun install

Bun's package manager is dramatically faster than npm/yarn/pnpm:

```bash
# Typical install times for a medium project (~500 deps):
npm install     # 30-60 seconds
yarn install    # 20-40 seconds
pnpm install    # 10-20 seconds
bun install     # 0.5-3 seconds
```

**Why so fast?**
1. **Symlinked `node_modules`** — Bun uses a flat `node_modules` with symlinks, similar to pnpm but even more optimized
2. **Binary lockfile (`bun.lockb`)** — Instead of JSON/YAML, Bun uses a binary format for faster parsing
3. **Parallel downloads** — Aggressive parallelization of HTTP requests
4. **Native code** — The entire package manager is native Zig code, not JavaScript
5. **Global module cache** — Downloaded packages are cached globally and hard-linked into projects

```bash
# Install dependencies
bun install

# Add a dependency
bun add react

# Add dev dependency
bun add -d @types/react

# Run a package script
bun run build

# Execute a package binary
bun x prisma generate   # equivalent to npx
```

### Bun Workspaces

Bun Workspaces is Bun's built-in monorepo support — a way to manage multiple packages (projects) in a single repository with a shared lockfile and dependency tree.

**Why workspaces matter:**
In modern development, you rarely have just one package. A typical company has:
- `apps/web` — Next.js frontend
- `apps/api` — Express/Fastify backend
- `packages/shared-types` — shared TypeScript definitions
- `packages/ui` — shared React component library
- `packages/utils` — common utilities

Without workspaces, each package has its own `node_modules`, lockfile, and install step. CI becomes slow. Version mismatches creep in. Refactoring across packages is painful.

**How Bun Workspaces work:**

```json
// Root package.json
{
  "name": "my-company",
  "private": true,
  "workspaces": [
    "apps/*",
    "packages/*"
  ],
  "devDependencies": {
    "typescript": "^5.0.0"
  }
}
```

```
my-company/
├── package.json          # Root workspace config
├── bun.lockb             # Single lockfile for ALL packages
├── apps/
│   ├── web/
│   │   ├── package.json  # { "name": "@mycompany/web", "dependencies": { "@mycompany/ui": "workspace:*" } }
│   │   └── src/
│   └── api/
│       ├── package.json  # { "name": "@mycompany/api", "dependencies": { "@mycompany/shared-types": "workspace:*" } }
│       └── src/
└── packages/
    ├── ui/
    │   ├── package.json  # { "name": "@mycompany/ui" }
    │   └── src/
    ├── utils/
    │   ├── package.json  # { "name": "@mycompany/utils" }
    │   └── src/
    └── shared-types/
        ├── package.json  # { "name": "@mycompany/shared-types" }
        └── src/
```

**Key features:**

1. **Single lockfile (`bun.lockb`)** — All workspace dependencies are resolved together, eliminating version conflicts
2. **Hoisted `node_modules`** — Common dependencies are installed once at the root, saving disk space and install time
3. **Workspace protocol (`workspace:*`)** — Links packages locally instead of publishing to npm
4. **Cross-package scripts** — Run commands across all workspaces

```bash
# Install ALL dependencies for ALL workspaces
bun install

# Add a dependency to a specific workspace
bun add react --cwd apps/web

# Add a workspace-internal dependency
bun add @mycompany/ui --cwd apps/web
# Automatically resolves to local package, not npm

# Run a script in all workspaces
bun run build --filter '*'

# Run a script in a specific workspace
bun run dev --cwd apps/api

# Run tests across all workspaces
bun test

# Filter workspaces by name pattern
bun run lint --filter '@mycompany/*'
```

**Workspace dependency resolution:**

```json
// apps/web/package.json
{
  "name": "@mycompany/web",
  "dependencies": {
    "react": "^18.2.0",                    // Regular npm package
    "@mycompany/ui": "workspace:*",         // Links to packages/ui locally
    "@mycompany/utils": "workspace:^1.0.0"  // Local + semver constraint
  }
}
```

The `workspace:*` protocol means: "Use the exact version from the local workspace, ignore npm." This is how you develop multiple interdependent packages without publishing them.

**Use cases for Bun Workspaces:**

| Use Case | Description |
|----------|-------------|
| **Monorepo with shared libraries** | Frontend, backend, and shared packages in one repo |
| **Design system development** | Build UI components in isolation, consume in apps |
| **Microservices with shared types** | API contracts defined in `packages/shared-types`, used by all services |
| **Tooling consolidation** | One `bun install` installs everything; one CI job tests everything |
| **Version synchronization** | Single source of truth for dependency versions across packages |

**Comparison with alternatives:**

| Feature | Bun Workspaces | npm Workspaces | Yarn Workspaces | pnpm Workspaces |
|---------|---------------|----------------|-----------------|-----------------|
| **Speed** | Fastest (native Zig) | Slow (JS-based) | Medium | Fast |
| **Lockfile** | Binary (`bun.lockb`) | JSON (`package-lock.json`) | YAML (`yarn.lock`) | YAML (`pnpm-lock.yaml`) |
| **Disk usage** | Hoisted + deduped | Hoisted | Hoisted | Content-addressable store (most efficient) |
| **Workspace protocol** | `workspace:*` | `workspace:*` | `workspace:*` | `workspace:*` |
| **Filter syntax** | `--filter` | `--workspace` | `yarn workspaces foreach` | `--filter` |
| **Runs without install** | No | No | No | No |

> [!tip] Pro Tip
> Bun Workspaces shines when combined with Bun's other features: `bun install` resolves all workspaces in seconds, `bun run` executes scripts across workspaces, and `bun build` can bundle workspace packages for production.

### HTTP Server

Bun's built-in HTTP server is a core API, not an external package like Express:

```typescript
import { serve } from "bun";

// High-performance HTTP server
serve({
  port: 3000,
  hostname: "0.0.0.0",
  
  // Standard Web API Request/Response
  fetch(request: Request): Response | Promise<Response> {
    const url = new URL(request.url);
    
    if (url.pathname === "/api/users") {
      return Response.json({ users: [] });
    }
    
    return new Response("Not Found", { status: 404 });
  },
  
  // WebSocket support built-in
  websocket: {
    open(ws) {
      ws.subscribe("chat");
    },
    message(ws, message) {
      ws.publish("chat", message);
    },
  },
});
```

**Key features:**
- Uses standard **Web API** `Request`/`Response` objects (WinterCG compatible)
- **WebSocket server** built-in (no `ws` package needed)
- **TLS** built-in (no need for external reverse proxy for simple cases)
- **Hot reload** in development: `bun --hot run server.ts`

### Bundler: bun build

Bun's bundler is designed to replace webpack, Rollup, esbuild, and Vite for many use cases:

```bash
# Bundle for browser
bun build ./src/index.tsx --outdir ./dist

# Bundle for Bun runtime (tree-shaken, optimized)
bun build ./src/index.ts --outfile ./dist/bundle.js --target bun

# Bundle for Node.js compatibility
bun build ./src/index.ts --outfile ./dist/bundle.js --target node
```

**Features:**
- Tree shaking
- Minification
- Source maps
- CSS bundling (experimental)
- Code splitting (experimental)
- ESM and CommonJS output

---

## ⚔️ Bun vs Node.js — Detailed Comparison

### Runtime Performance

| Metric | Bun | Node.js | Notes |
|--------|-----|---------|-------|
| **Startup time** | ~20-50ms | ~100-300ms | Bun's advantage is most pronounced in serverless/edge |
| **HTTP throughput** | Often 2-4x | Baseline | Varies by workload; Bun wins on simple routing |
| **File I/O** | Faster (io_uring) | Good | Bun uses io_uring on Linux 5.1+ |
| **JSON parsing** | Native optimization | JSON.parse() | Bun has a highly optimized JSON parser |
| **Crypto** | Native (BoringSSL) | Native (OpenSSL) | Comparable |
| **Memory footprint** | Lower | Higher | Bun's unified binary shares more code |

### API Compatibility

Bun aims for **drop-in replacement** status:

```javascript
// Code that works in BOTH Node.js and Bun:
const fs = require("fs");
const http = require("http");
const path = require("path");

http.createServer((req, res) => {
  fs.readFile(path.join(__dirname, "index.html"), (err, data) => {
    res.end(data);
  });
}).listen(3000);
```

**Compatibility status (as of 2025):**
- **Core modules**: ~95% compatible (`fs`, `path`, `http`, `crypto`, `os`, etc.)
- **npm packages**: ~90-95% work without modification
- **Native C++ addons (node-gyp)**: Partial — many work, some don't. Bun prefers `bun:ffi` for native interop.
- **Worker Threads**: Supported
- **ESM**: First-class, native support
- **CommonJS**: Fully supported for backward compatibility

### Ecosystem Lock-in

```
Node.js ecosystem:
┌─────────────────────────────────────────┐
│  Your App                                │
│  ├─ depends on Express (npm package)    │
│  ├─ depends on Prisma (npm + engines)   │
│  ├─ depends on Sharp (native C++ addon) │
│  └─ depends on Jest (test runner)       │
│                                          │
│  All managed by npm/node-gyp/tsc/jest   │
└─────────────────────────────────────────┘

Bun ecosystem:
┌─────────────────────────────────────────┐
│  Your App                                │
│  ├─ depends on Express (npm-compatible) │
│  ├─ depends on Prisma (usually works)   │
│  ├─ depends on Sharp (may need FFI)     │
│  └─ bun:test (built-in, replaces Jest)  │
│                                          │
│  Managed by bun (single tool)           │
└─────────────────────────────────────────┘
```

---

## ⚔️ Bun vs Deno

Both are "modern alternatives to Node.js" but with different philosophies:

| Aspect | Bun | Deno |
|--------|-----|------|
| **Creator** | Jarred Sumner | Ryan Dahl (creator of Node.js) |
| **Philosophy** | Fast, compatible, batteries-included | Secure, standards-compliant, explicit |
| **npm compatibility** | Primary goal | Added later (via `npm:` specifiers) |
| **TypeScript** | Native, no config | Native, no config |
| **Security** | Standard Node.js model | Permission model by default (file, net, env access must be granted) |
| **Standard library** | Growing `bun:` APIs | Extensive, browser-compatible `std` library |
| **Package management** | `bun install` (npm-compatible) | `deno add` (JSR + npm) |
| **Bundler** | Built-in | Built-in (`deno bundle`) |
| **Test runner** | Built-in (`bun:test`) | Built-in (`Deno.test`) |
| **FFI** | `bun:ffi` (simplified) | `Deno.dlopen` |
| **Config files** | Minimal (bunfig.toml optional) | None by default (explicit permissions via CLI flags) |

**Key philosophical difference:**
- **Bun**: "Make JavaScript tooling faster and simpler while staying compatible"
- **Deno**: "Fix the mistakes of Node.js (security, module system) even if it breaks compatibility"

---

## 🚀 How Bun Is Used

| Use Case | Why Bun Fits |
|----------|--------------|
| **Serverless / Edge functions** | Cold start is critical; Bun's ~20ms startup vs Node's ~200ms is a 10x improvement |
| **Development servers** | `bun --hot` provides instant hot reload; `bun install` keeps CI fast |
| **CLI tools** | Single binary distribution; fast execution; built-in bundling |
| **API backends** | Built-in HTTP server with WebSockets; often faster than Express on Node |
| **Build pipelines** | Replaces webpack/esbuild + tsc in many projects |
| **Testing** | `bun:test` with Jest-compatible API; much faster test execution |
| **SQLite-backed apps** | `bun:sqlite` is faster and simpler than external SQLite libraries |
| **Scripting** | TypeScript as a scripting language with zero setup |

---

## ⚠️ Pitfalls & Anti-patterns

### 1. Assuming 100% Node.js Compatibility
```javascript
// ❌ This might not work in Bun
const addon = require("./native-addon.node"); // node-gyp compiled

// ✅ Check Bun compatibility or use bun:ffi
import { dlopen } from "bun:ffi";
const lib = dlopen("./lib.so", { ... });
```

### 2. Relying on Bun's TypeScript Without Type Checking
```typescript
// ❌ Bun runs this fine, but it's wrong
function add(a: number, b: number): number {
  return a + b;
}
add("1", "2"); // Runtime: "12", TypeScript would catch this

// ✅ Always run tsc in CI
// package.json script:
// "typecheck": "tsc --noEmit",
// "test": "bun test && npm run typecheck"
```

### 3. Ignoring the Lockfile
```bash
# ❌ Don't commit only package-lock.json
# Bun uses bun.lockb, not package-lock.json

# ✅ Commit both for mixed teams
# .gitignore should NOT ignore bun.lockb
git add bun.lockb
```

### 4. Using Bun APIs in Shared Code
```typescript
// ❌ This won't work in Node.js or Deno
import { Database } from "bun:sqlite";

// ✅ Use standard Web APIs or provide abstraction
// For cross-runtime code, use standard fetch, Web Streams, etc.
```

### 5. Overlooking Memory Characteristics
```javascript
// ❌ JSC's garbage collector behaves differently than V8's
// Objects might be collected at different times
// Don't rely on GC timing for application logic

// ✅ Explicitly manage resources
const file = Bun.file("data.txt");
const text = await file.text(); // reads entire file into memory
// For large files, use streaming
```

### 6. Production Without Fallback Testing
```bash
# ❌ Deploying to production without verifying on target runtime
bun build && deploy

# ✅ Test on both runtimes if migrating from Node.js
bun test       # Bun tests
node --test    # Node tests (if applicable)
```

---

## 🔗 Deep Connections

- **Parent Topic:** [[JavaScript Runtimes]], [[Server-Side Development]]
- **Related Concepts:**
  - [[Node.js - Architecture and Internals]] — the incumbent runtime Bun challenges
  - [[Deno]] — the other modern JS runtime by Ryan Dahl
  - [[JavaScriptCore]] — the engine inside Bun and Safari
  - [[V8 Engine]] — the engine inside Node.js and Chrome
  - [[Zig]] — the systems language Bun is written in
  - [[libuv]] — the I/O library Node.js uses (contrast with Bun's custom layer)
  - [[Web-interoperable Runtimes (WinterCG)]] — standards for edge-compatible JS
  - [[Serverless Architecture]] — primary use case for Bun's speed advantage
  - [[Edge Computing]] — where Bun's startup time matters most
  - [[FFI (Foreign Function Interface)]] — Bun's approach to native interop
- **Prerequisites:**
  - Understanding of Node.js architecture (event loop, V8, libuv)
  - Basics of JavaScript/TypeScript
  - Understanding of npm and package management
- **Next Steps:**
  - [[io_uring]] — Linux's modern async I/O API that Bun leverages
  - [[JavaScript JIT Compilation]] — how JSC and V8 optimize JS execution
  - [[Edge Runtimes]] — Cloudflare Workers, Vercel Edge, Deno Deploy

---

## ❓ Active Recall Questions

> [!faq]- Why did Bun choose JavaScriptCore instead of V8?
> JSC has faster cold-start times and different optimization characteristics that better suit serverless and edge computing scenarios. V8 is optimized for long-running browser tabs; JSC is more versatile for short-lived executions.

> [!faq]- What language is Bun written in, and why was that choice made?
> Zig. It offers explicit memory control (no hidden allocations), compile-time metaprogramming (`comptime`), seamless C interoperability, and a simpler mental model than C++ or Rust for systems programming.

> [!faq]- Does Bun type-check TypeScript at runtime?
> No. Bun transpiles TypeScript (strips types, transforms JSX) but does not perform type checking. You still need `tsc --noEmit` for type validation. This is intentional for speed.

> [!faq]- What is `bun:sqlite` and why is it significant?
> A built-in, high-performance SQLite wrapper. It eliminates the need for `better-sqlite3` or similar npm packages, is faster due to native integration, and requires zero installation.

> [!faq]- How does `bun install` achieve 20-100x speed over `npm install`?
> Native Zig implementation (not JS), binary lockfile format, aggressive parallel HTTP downloads, global module cache with hard links, and optimized symlinking strategy for `node_modules`.

> [!faq]- What is Bun's custom I/O layer, and why didn't they use libuv?
> Bun wrote its own async I/O layer in Zig to avoid libuv's design constraints and Node.js compatibility baggage. It directly uses epoll/kqueue/IOCP and leverages io_uring on modern Linux for file operations.

> [!faq]- What is the philosophical difference between Bun and Deno?
> Bun prioritizes speed and npm compatibility ("make JS tooling faster"). Deno prioritizes security and standards compliance, often at the cost of breaking changes from Node.js ("fix Node's mistakes").

> [!faq]- When should you NOT use Bun?
> When you need guaranteed 100% Node.js API compatibility (some native addons don't work), when your team requires maximum ecosystem maturity and LTS guarantees, or when your organization mandates specific security/compliance tools that only support Node.js.

> [!faq]- What does Bun's `--hot` flag do?
> Enables hot module reloading in development. Bun watches files and reloads changed modules without restarting the process, preserving server state where possible.

> [!faq]- What is `bun:ffi` and how does it differ from Node.js's N-API?
> `bun:ffi` is Bun's Foreign Function Interface that lets you call C libraries directly from JavaScript without writing C++ wrapper code. Node.js's N-API requires writing and compiling C++ addons. `bun:ffi` is simpler but potentially less performant for complex interop.

> [!faq]- What file does Bun use for lockfile, and why is it binary?
> `bun.lockb` — a binary format that parses faster than JSON/YAML. The `.lockb` extension signals "lockfile binary." It is designed for speed in large projects.

> [!faq]- What are Bun Workspaces and what problem do they solve?
> Bun Workspaces is built-in monorepo support that lets you manage multiple packages in a single repository with one shared lockfile and dependency tree. It solves the problem of version mismatches, duplicate installs, and slow CI when you have multiple interdependent packages (e.g., frontend + backend + shared libraries).

> [!faq]- What does the `workspace:*` protocol mean in Bun?
> It tells Bun to use the local workspace package instead of fetching from npm. For example, `"@mycompany/ui": "workspace:*"` links directly to the `packages/ui` folder in your monorepo, enabling development of interdependent packages without publishing to npm.

> [!faq]- How do you add a dependency to a specific workspace in Bun?
> Use the `--cwd` flag: `bun add react --cwd apps/web`. This installs `react` into the `apps/web` workspace specifically, not the root.

> [!faq]- How do you run a script across all Bun workspaces?
> Use the `--filter` flag with a glob pattern: `bun run build --filter '*'`. You can also filter by package name: `bun run lint --filter '@mycompany/*'`.

> [!faq]- What are the main use cases for Bun Workspaces?
> Monorepos with shared libraries, design system development (build UI components in isolation and consume in apps), microservices with shared types (API contracts in a common package), tooling consolidation (one install for everything), and version synchronization across packages.

> [!faq]- How does Bun Workspaces compare to pnpm Workspaces?
> Both support the `workspace:*` protocol and filtering. Bun is faster at install due to native Zig implementation and binary lockfile. pnpm uses a content-addressable store for disk efficiency, which is more space-efficient than Bun's hoisting approach. Both are good; Bun wins on speed, pnpm on disk usage.

---

## 📝 References & Further Reading

- [Bun Official Documentation](https://bun.sh/docs)
- [Bun GitHub Repository](https://github.com/oven-sh/bun)
- [Zig Programming Language](https://ziglang.org/)
- [JavaScriptCore / WebKit Documentation](https://trac.webkit.org/wiki/JavaScriptCore)
- [io_uring Linux Documentation](https://kernel.dk/io_uring.pdf)
- [WinterCG Web-interoperable Runtimes](https://wintercg.org/)
- "Bun: A fast all-in-one JavaScript runtime" — Jarred Sumner's original blog posts
- [Node.js - Architecture and Internals]] — for contrast and comparison
- [Deno]] — the other modern runtime
