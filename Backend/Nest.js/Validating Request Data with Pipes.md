---
status: new
tags: [nest-js, pipes, validation, decorators, dto]
source: https://docs.nestjs.com/pipes
created: 2026-06-30
updated: 2026-06-30
difficulty: intermediate
---

# 🧠 Topic: Validating Request Data with Pipes

> [!abstract] Executive Summary (ELI5)
> NestJS pipes are checkpoint guards that sit between the incoming HTTP request and your controller method. They extract request data (body, query, params, headers) using decorators, validate it, and either pass it through or throw an error before your handler runs.

---

## 🎯 The "Why"
- **Problem Solved:** Without validation, your route handlers receive raw, untrusted data from clients. Pipes enforce data correctness at the system boundary, keeping controllers clean and focused on business logic.
- **Alternatives:**
  - **Manual validation inside handlers:** Breaks single responsibility principle, repeated boilerplate.
  - **Middleware:** Doesn't know which handler or parameter is being processed.
  - **Joi/Zod at route level manually:** Works, but pipes integrate with DI, DTO metatypes, and can be reused globally.
- **Trade-offs:** Built-in `ValidationPipe` requires `class-validator` + `class-transformer` (extra deps). Pipes add a small runtime overhead, but it's negligible compared to the safety gained.

## 💡 Mental Model & Analogy
- **Analogy:** "A pipe is like the security checkpoint at an airport. The decorator (`@Body()`, `@Param()`) is the gate that picks which bag to inspect, and the pipe is the scanner that checks the contents. If the bag is fine, it passes through. If not, it's rejected before it ever reaches the plane (your handler)."
- **Visual/Diagram:**
```mermaid
graph LR
    A[HTTP Request] --> B[@Body / @Query / @Param]
    B --> C[Pipe]
    C -->|valid| D[Controller Handler]
    C -->|invalid| E[BadRequestException]
```

## 🛠️ Core Principles & Code Snippets

### 1. Accessing Request Data with Decorators
Nest provides decorators to extract specific parts of the request. They're imported from `@nestjs/common`.

| Decorator | Accesses |
|-----------|----------|
| `@Body(key?)` | `req.body` or a specific property |
| `@Query(key?)` | `req.query` or a specific property |
| `@Param(key?)` | `req.params` or a specific property |
| `@Headers(name?)` | `req.headers` or a specific header |
| `@Req()` / `@Request()` | The full request object |
| `@Res()` / `@Response()` | The full response object (library-specific mode) |

```typescript
@Controller('cats')
export class CatsController {
  @Get(':id')
  findOne(@Param('id') id: string) {
    return `Cat #${id}`;
  }

  @Get()
  findAll(@Query('limit') limit: number) {
    return `Returning ${limit} cats`;
  }

  @Post()
  create(@Body() createCatDto: CreateCatDto) {
    return this.catsService.create(createCatDto);
  }
}
```

> [!tip] Implementation Note
> Use `@Body()`, `@Query()`, and `@Param()` instead of `@Req()` whenever possible. They make your code platform-agnostic and easier to test.

### 2. What is a Pipe?
A pipe is a class annotated with `@Injectable()` that implements `PipeTransform`.

```typescript
@Injectable()
export class ValidationPipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    return value;
  }
}
```

Pipes have two use cases:
- **Transformation:** convert input data (e.g., string → number).
- **Validation:** evaluate input and throw if it doesn't meet rules.

### 3. Built-in Pipes
Nest ships with these pipes in `@nestjs/common`:

| Pipe | Purpose |
|------|---------|
| `ParseIntPipe` | Parses/validates a numeric string to integer |
| `ParseFloatPipe` | Parses to float |
| `ParseBoolPipe` | Parses to boolean |
| `ParseUUIDPipe` | Validates UUID format |
| `ParseEnumPipe` | Validates enum values |
| `ParseArrayPipe` | Validates/parses arrays |
| `ParseDatePipe` | Parses to Date |
| `DefaultValuePipe` | Supplies default for missing params |
| `ValidationPipe` | Validates DTOs using `class-validator` |
| `ParseFilePipe` | Validates uploaded files |

### 4. Binding Pipes at the Parameter Level
```typescript
@Get(':id')
async findOne(@Param('id', ParseIntPipe) id: number) {
  return this.catsService.findOne(id);
}
```

Passing the class lets Nest instantiate it (enables DI). Pass an instance to customize options:
```typescript
@Get(':id')
async findOne(
  @Param('id', new ParseIntPipe({ errorHttpStatusCode: HttpStatus.NOT_ACCEPTABLE }))
  id: number,
) {
  return this.catsService.findOne(id);
}
```

### 5. Validating DTOs with `ValidationPipe`
Install dependencies:
```bash
npm i --save class-validator class-transformer
```

Define the DTO with decorators:
```typescript
import { IsString, IsInt } from 'class-validator';

export class CreateCatDto {
  @IsString()
  name: string;

  @IsInt()
  age: number;

  @IsString()
  breed: string;
}
```

Bind the built-in `ValidationPipe` globally:
```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe());
  await app.listen(3000);
}
```

Or via a module provider for DI support:
```typescript
@Module({
  providers: [
    {
      provide: APP_PIPE,
      useClass: ValidationPipe,
    },
  ],
})
export class AppModule {}
```

### 6. Common ValidationPipe Options
```typescript
new ValidationPipe({
  whitelist: true,              // strip properties without decorators
  forbidNonWhitelisted: true,   // throw on unexpected properties
  transform: true,              // auto-transform payloads to DTO instances
  disableErrorMessages: false,  // set true in production to hide details
})
```

## ⚠️ Pitfalls & Anti-patterns
- **Common Mistakes:**
  - Using TypeScript interfaces for DTOs — interfaces disappear at runtime, so `class-validator` decorators and `metatype` inspection won't work. Always use classes.
  - Forgetting to register a global pipe via `APP_PIPE` when it needs dependencies.
  - Passing `@Res()` without `passthrough: true` and then trying to use Nest's standard response features.
- **When NOT to use:**
  - For trivial routes where manual type coercion is simpler.
  - When you need complex cross-field validation that is better handled by a dedicated service or Zod schema pipe.
- **Performance/Security considerations:**
  - `ValidationPipe` with `transform: true` creates class instances for every request — acceptable for most APIs, but measure if you're under extreme load.
  - Don't expose raw `class-validator` error details to clients in production; use an exception filter to sanitize.

## 🔗 Deep Connections
- **Parent Topic:** [[NestJS Controller and Request Lifecycle]]
- **Related Concepts:** [[NestJS Routing Decorators]], [[Dependency Injection]], [[Data Transfer Object Pattern]], [[class-validator]]
- **Prerequisites:** TypeScript decorators, basic NestJS controllers, DTOs.

## ❓ Active Recall Questions
> [!faq]- What is the difference between `@Body()` and `@Body('name')`?
> `@Body()` returns the entire request body object, while `@Body('name')` extracts only the `name` property.

> [!faq]- Why must DTOs be classes instead of TypeScript interfaces?
> Interfaces are erased during transpilation. Pipes need runtime access to the DTO metatype, which only classes provide.

> [!faq]- What happens if `ValidationPipe` is registered globally with `whitelist: true` and a request contains an extra property?
> The extra property is automatically stripped before reaching the handler. If `forbidNonWhitelisted: true` is also set, the request throws a `BadRequestException` instead.

---
*This note follows the Technical-Deep-Dive template. Status: new → update to `growing` after hands-on practice.*
