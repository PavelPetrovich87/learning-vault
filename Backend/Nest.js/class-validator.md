---
status: stable
mastery: beginner
tags: [nest-js, validation, class-validator, decorators, dto]
source: https://github.com/typestack/class-validator
created: 2026-06-30
updated: 2026-10-21
difficulty: intermediate
---

# 🧠 Topic: class-validator in NestJS

> [!abstract] Executive Summary (ELI5)
> `class-validator` is a library that lets you attach validation rules to class properties using decorators. When paired with NestJS's `ValidationPipe`, it automatically checks incoming request data and rejects invalid payloads before they reach your business logic.

---

## 🎯 The "Why"
- **Problem Solved:** Manually checking every field of incoming data is repetitive and error-prone. `class-validator` centralizes validation rules directly on the data model.
- **Alternatives:**
  - **Joi / Zod:** Schema-based validators, not tied to class decorators.
  - **Manual `if` checks inside handlers:** Breaks SRP and is hard to maintain.
  - **JSON Schema validation:** Works but requires maintaining a separate schema file.
- **Trade-offs:** `class-validator` relies on classes and decorators, so it can't validate TypeScript interfaces. It also requires `class-transformer` for NestJS integration.

## 💡 Mental Model & Analogy
- **Analogy:** "Think of `class-validator` as a set of sticky labels you put on each field of a form. Each label says what's allowed (`@IsEmail`, `@IsInt`, `@Length(10, 20)`). When a form is submitted, the `ValidationPipe` scans all labels and rejects the form if any rule is broken."
- **Visual/Diagram:**
```mermaid
graph LR
    A[CreateUserDto Class] --> B[@IsEmail]
    A --> C[@MinLength]
    A --> D[@IsOptional]
    B --> E[ValidationPipe]
    C --> E
    D --> E
    E -->|all pass| F[Handler Receives Valid DTO]
    E -->|failure| G[400 Bad Request]
```

## 🛠️ Core Principles & Code Snippets

### 1. Installation
`class-validator` is used by NestJS's built-in `ValidationPipe`. You also need `class-transformer` for the pipe to transform plain objects into class instances.

```bash
npm i --save class-validator class-transformer
```

### 2. Declaring Rules with Decorators
Attach decorators to class properties. The most common ones are:

| Category | Decorators |
|----------|------------|
| **Type** | `@IsString()`, `@IsNumber()`, `@IsInt()`, `@IsBoolean()`, `@IsDate()`, `@IsArray()`, `@IsEnum()` |
| **Presence** | `@IsDefined()`, `@IsOptional()`, `@IsNotEmpty()`, `@IsEmpty()` |
| **String** | `@Length(min, max)`, `@MinLength()`, `@MaxLength()`, `@IsEmail()`, `@IsURL()`, `@Matches()` |
| **Number** | `@Min()`, `@Max()`, `@IsPositive()`, `@IsNegative()` |
| **Date** | `@MinDate()`, `@MaxDate()`, `@IsDateString()` |
| **Identifier** | `@IsUUID()`, `@IsMongoId()`, `@IsNumberString()` |
| **Other** | `@IsIn(values)`, `@IsNotIn(values)`, `@Equals()`, `@NotEquals()` |

```typescript
import {
  IsString,
  IsInt,
  IsEmail,
  IsOptional,
  Length,
  Min,
  Max,
} from 'class-validator';

export class CreateUserDto {
  @IsEmail()
  email: string;

  @IsString()
  @Length(8, 128)
  password: string;

  @IsString()
  @IsOptional()
  nickname?: string;

  @IsInt()
  @Min(13)
  @Max(120)
  age: number;
}
```

> [!tip] Implementation Note
> Prefer `@IsInt()` over `@IsNumber()` for integer fields. `@IsNumber()` accepts decimals; `@IsInt()` enforces whole numbers.

### 3. How ValidationPipe Uses It
When `ValidationPipe` receives a plain object, it transforms it into a class instance using `class-transformer`, then runs `class-validator` on it. If validation fails, it throws a `BadRequestException`.

```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe({
    whitelist: true,
    forbidNonWhitelisted: true,
    transform: true,
  }));
  await app.listen(3000);
}
```

### 4. Nested Object Validation
Use `@ValidateNested()` to validate nested objects. The nested value must be an instance of a class, not a plain object.

```typescript
import { ValidateNested } from 'class-validator';
import { Type } from 'class-transformer';

export class AddressDto {
  @IsString()
  city: string;

  @IsString()
  zipCode: string;
}

export class CreateUserDto {
  @ValidateNested()
  @Type(() => AddressDto)
  address: AddressDto;
}
```

> [!tip] Implementation Note
> Always pair `@ValidateNested()` with `@Type(() => Class)` from `class-transformer`. Otherwise the nested object stays a plain object and validation won't run.

### 5. Validating Arrays
Use `each: true` to validate every item in an array.

```typescript
export class CreatePostDto {
  @IsString({ each: true })
  @ArrayNotEmpty()
  tags: string[];
}
```

### 6. Conditional Validation
Use `@ValidateIf()` to run decorators only when a condition is met.

```typescript
export class UpdateUserDto {
  @IsOptional()
  @IsString()
  country?: string;

  @ValidateIf(o => o.country === 'USA')
  @IsString()
  @IsNotEmpty()
  state?: string;
}
```

### 7. Custom Validation Messages
Override default messages with the `message` option. You can use special tokens: `$value`, `$property`, `$constraint1`, `$constraint2`, etc.

```typescript
export class CreateUserDto {
  @IsEmail({}, { message: '$property must be a valid email, got $value' })
  email: string;

  @MinLength(8, { message: 'Password is too short. Minimum $constraint1 characters.' })
  password: string;
}
```

### 8. Custom Validators
For domain-specific rules, create a custom validator class or decorator.

```typescript
import { ValidatorConstraint, ValidatorConstraintInterface, ValidationArguments } from 'class-validator';

@ValidatorConstraint({ name: 'isPasswordMatch', async: false })
export class IsPasswordMatchConstraint implements ValidatorConstraintInterface {
  validate(value: string, args: ValidationArguments) {
    const object = args.object as any;
    return value === object.password;
  }

  defaultMessage(args: ValidationArguments) {
    return `Passwords do not match`;
  }
}
```

Use it in your DTO:
```typescript
import { Validate } from 'class-validator';

export class CreateUserDto {
  @IsString()
  password: string;

  @Validate(IsPasswordMatchConstraint)
  confirmPassword: string;
}
```

## ⚠️ Pitfalls & Anti-patterns
- **Common Mistakes:**
  - Using `class-validator` on TypeScript interfaces — interfaces have no runtime metadata.
  - Forgetting `@IsOptional()` on optional fields and then wondering why `undefined` fails validation.
  - Using `@ValidateNested()` without `@Type()` from `class-transformer`.
  - Forgetting `each: true` when validating arrays.
- **When NOT to use:**
  - For very dynamic validation rules that are better expressed as schemas (Joi/Zod may fit better).
  - When you don't want runtime overhead and TypeScript compile-time checks are enough.
- **Performance/Security considerations:**
  - Validating large nested objects can be CPU-intensive. Avoid exposing `target` in errors in production (`validationError: { target: false }`).
  - `forbidNonWhitelisted` + `whitelist` prevents mass-assignment vulnerabilities by rejecting unknown properties.

## 🔗 Deep Connections
- **Parent Topic:** [[Data Transfer Object (DTO)]]
- **Related Concepts:** [[Validating Request Data with Pipes]], [[class-transformer]], [[NestJS Routing Decorators]]
- **Prerequisites:** TypeScript decorators, NestJS controllers, DTOs.

## ❓ Active Recall Questions
> [!faq]- Why does `class-validator` require classes instead of interfaces?
> Interfaces are erased at runtime. Decorators are attached to class prototypes and preserved at runtime, which `class-validator` and `ValidationPipe` rely on.

> [!faq]- What happens if you use `@ValidateNested()` without `@Type(() => NestedDto)` from `class-transformer`?
> The nested object remains a plain object, so `class-validator` won't know which class to validate against and the nested validation will not run.

> [!faq]- What is the purpose of `each: true` in a decorator option?
> It applies the validation rule to every item in an array, set, or map instead of the collection itself.

---
*This note follows the Technical-Deep-Dive template. Status: new → update to `growing` after hands-on practice.*
