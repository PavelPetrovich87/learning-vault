# Мысленный Эксперимент: Урок по To-Do List с AI-Преподавателем Alex

## Демонстрация Методологии в Действии

---

> **Контекст**: Ученик — Senior Frontend (React, Next.js, TypeScript, 5+ лет), Junior Backend (базовый NestJS, REST API). Преподаватель — Alex, Senior Backend Engineer с бывшим frontend-опытом. Урок — построение простейшего To-Do list с NestJS backend + React frontend. Длительность — ~70 минут.

---

## Phase 1: Theory — Архитектура To-Do List через Frontend-Аналогии

**Время**: 0-15 минут

---

**Alex**: Окей, давай разберём архитектуру To-Do приложения. Не будем сразу писать код — сначала **подумай, как senior frontend инженер**, а потом перенесём логику на backend.

**Вопрос**: Ты строишь React-приложение с To-Do листом. У тебя есть state с массивом todos. Какие **операции** (CRUD) должен поддерживать этот стейт?

**Ученик**: Ну, очевидно — Create (добавить todo), Read (отобразить список), Update (отметить выполненным, изменить текст), Delete (удалить). Классический CRUD.

**Alex**: Идеально. Во frontend'е эти операции меняют **локальный стейт** — `useState`, Context, или что ты используешь. Но в нашем приложении данные должны **сохраняться** после перезагрузки страницы. Значит, state уезжает на сервер. Как ты думаешь, как это влияет на архитектуру?

**Ученик**: Нужен API, через который фронт будет говорить с сервером. Но если я правильно понимаю, просто API недостаточно — нужна ещё база данных, потому что сервер тоже stateless.

**Alex**: **В точку.** Вот ключевое отличие — и оно фундаментально. На фронтенде ты привык, что "стейт живёт в памяти браузера" — один пользователь, один процесс, всё мгновенно. На backend'е у тебя **три слоя**, которые между собой общаются через границы:

```
[React Frontend] ←→ [REST API (NestJS)] ←→ [PostgreSQL DB]
```

Каждая стрелка — это **сетевая граница**. И каждая граница добавляет сложность, которой на фронтенде просто нет:

| Аспект | Frontend | Backend |
|:---|:---|:---|
| **State location** | Memory браузера | PostgreSQL — диск, persistence |
| **Конкурентность** | Один пользователь, один тред | Множество запросов одновременно |
| **Валидация** | Schema (Zod, Yup) | **Двойная**: входящая + на уровне БД |
| **Ошибки** | Catch в компоненте | **Propagation через 3 слоя**: controller → service → DB |
| **Type safety** | TS компиляция | **Runtime**: class-validator, DTO |

**Alex**: Давай пройдём по каждому CRUD-операцию и подумаем, что меняется при переносе на backend. Начнём с **Create**.

**Ученик**: На фронте я бы сделал `setTodos([...todos, newTodo])`. На бэкенде... POST-запрос создаёт запись в БД. Но нужно валидировать входные данные, потому что клиент может отправить что угодно.

**Alex**: **Именно.** И вот тут мы встречаем первый backend-паттерн: **DTO + Validation Pipe**. В NestJS это работает так — `ValidationPipe` автоматически проверяет входящие данные по декораторам DTO. Это как Zod на фронте, но **на уровне фреймворка**, а не ручной валидации. Покажу на диаграмме:

```
Client sends: { title: "", completed: "yes" }  // invalid!
     ↓
ValidationPipe checks @IsNotEmpty(), @IsBoolean()
     ↓
If invalid → 400 Bad Request с деталями ошибки
If valid → передаётся в service → сохраняется в DB
```

**Alex**: Твой frontend-опыт поможет здесь. Помнишь, когда в React Form ты делаешь валидацию перед отправкой? Здесь то же самое, но **сервер не доверяет клиенту никогда**. Клиент может быть твоим React-приложением, а может быть curl, Postman, или злоумышленник. Поэтому валидация — обязательна на сервере, даже если фронт уже провалидировал.

**Ученик**: Логично. То есть DTO — это как interface/тип, но с runtime-валидацией.

**Alex**: **В точку.** DTO — это как TypeScript interface + Zod schema в одном. Только декларативный: `@IsString()`, `@MinLength(1)`, `@IsBoolean()` — и NestJS сам всё проверяет.

**Alex**: Переходим к **Update**. На фронте ты меняешь стейт по индексу. На бэкенде — PUT или PATCH-запрос. Какую HTTP-метод выберешь для "отметить todo выполненным"?

**Ученик**: PUT — заменяет весь ресурс, PATCH — частичное обновление. Для одного поля `completed` логичнее PATCH.

**Alex**: **Верно.** Но есть тонкость: в NestJS `@Patch(':id')` с `@Body() updateDto: UpdateTodoDto` — updateDto содержит **только изменяемые поля**. Это как partial type в TypeScript. И вот тут скрыта ошибка, которую делают 70% junior-разработчиков. Какая?

**Ученик**: Хм... может быть, если не провалидировать, что id существует, то обновишь несуществующую запись?

**Alex**: **Частично.** Правильный ответ: нужно проверить, что todo с таким id **существует**, прежде чем обновлять. Иначе получишь "тихий успех" — 200 OK, но ничего не изменилось. Это как на фронте делать `todos.find(t => t.id === id)` — если не нашёл, показывать ошибку, а не молча ничего не делать. На бэкенде мы обычно бросаем **NotFoundException** → 404.

**Alex**: Теперь про **Delete**. На фронте `todos.filter(t => t.id !== id)`. На бэкенде — `DELETE /todos/:id`. Но опять же: что если id не существует? 404. А что если удаление CASCADE — например, todo связан с категориями? Тут мы упираемся в **database relations**, но это для следующих недель.

---

**[Phase 1 завершена: пройдены CRUD операции, введены паттерны DTO, ValidationPipe, error handling границы. Scaffolding: высокий — Alex объяснил архитектуру полностью.]**

---

## Phase 2: Socratic Discussion — REST API Design

**Время**: 15-25 минут

---

**Alex**: Давай спроектируем API. У нас есть сущность Todo с полями: `id`, `title`, `completed`, `createdAt`. Нарисуй в голове endpoints. Какие HTTP-методы и пути?

**Ученик**: Ну... `GET /todos` — список, `GET /todos/:id` — одна, `POST /todos` — создать, `PATCH /todos/:id` — обновить, `DELETE /todos/:id` — удалить.

**Alex**: Отлично. Это RESTful ресурсно-ориентированный подход — и он **идеален** для такого простого CRUD. Но давай углубим. Что будет, если `GET /todos` вернёт 10,000 записей?

**Ученик**: Нужна пагинация! `?page=1&limit=20` или cursor-based. Для To-Do это может быть перебор, но для Social Feed — критично.

**Alex**: **В точку.** И вот мы уже готовимся к W1 — Social Feed с infinite scroll. Для Todo пагинация не критична, но давай заложим её в архитектуре, чтобы потом просто скопировать паттерн. Какую пагинацию выберешь — offset или cursor?

**Ученик**: Для Todo — offset, потому что порядок фиксированный и записи не удаляются часто. Cursor — для feeds, где порядок меняется.

**Alex**: **Правильно.** Но есть edge case: что если между запросами `?page=1&limit=10` и `?page=2&limit=10` добавили новую запись? Страница 2 "сдвинется" — пропустишь запись или получишь дубликат. Это называется **offset drift**. Cursor-based решает это. Для Todo — допустимо, но знать об edge case важно.

**Alex**: Следующий вопрос: что возвращает `POST /todos` — только id, или полный объект?

**Ученик**: Полный объект, чтобы фронт сразу мог добавить его в список без дополнительного запроса.

**Alex**: **Идеально.** Это называется **"optimistic creation"** — фронт получает полный объект и сразу добавляет в UI. Знакомо? Как в TanStack Query — `mutation.onSuccess` добавляет в кэш. Но опять backend-версия: сервер отвечает полным объектом, потому что у него есть `createdAt`, `id` (автоинкремент), которые клиент не знает заранее.

**Alex**: Ещё один вопрос: `createdAt` — кто генерирует — клиент или сервер?

**Ученик**: Сервер, однозначно. Клиент может быть в другом часовом поясе, или вообще с подменённым временем.

**Alex**: **В точку.** Это принцип **"server is source of truth for server-generated fields"**. На фронте ты бы использовал `new Date().toISOString()` — но это client time, ненадёжно. База данных через `@CreateDateColumn()` генерирует timestamp — это единственная истина.

---

**[Phase 2 завершена: спроектирован API с пагинацией, обсуждены edge cases, введён принцип server as source of truth. Scaffolding: средний — Alex задавал вопросы, но давал hints.]**

---

## Phase 3: Implementation — Skeleton Code

**Время**: 25-55 минут

---

**Alex**: Давай писать код. Начнём с backend. Я дам skeleton — ты заполнишь логику. Помни explain-then-implement: **перед каждым TODO объясни, что будешь делать**.

### Step 1: DTO (Alex предоставляет, ученик объясняет)

```typescript
// create-todo.dto.ts
import { IsString, IsNotEmpty, IsBoolean, IsOptional } from 'class-validator';

export class CreateTodoDto {
  @IsString()
  @IsNotEmpty()
  title: string;

  @IsBoolean()
  @IsOptional()
  completed?: boolean;  // default false
}
```

**Alex**: Объясни, что делает каждый декоратор.

**Ученик**: `@IsString()` — проверяет, что title — строка. `@IsNotEmpty()` — не пустая строка. `@IsBoolean()` — completed должен быть boolean. `@IsOptional()` — поле не обязательное. Если клиент отправит `{ title: 123 }` — `ValidationPipe` вернёт 400 с ошибкой.

**Alex**: **Идеально.** Теперь UpdateTodoDto. Каким он должен быть — похожим или отличаться?

**Ученик**: Все поля optional — потому что PATCH обновляет частично. Может прийти только `{ completed: true }` без title.

**Alex**: **Верно.** И вот тут NestJS помогает:

```typescript
// update-todo.dto.ts
import { PartialType } from '@nestjs/mapped-types';
import { CreateTodoDto } from './create-todo.dto';

export class UpdateTodoDto extends PartialType(CreateTodoDto) {}
```

`PartialType` автоматически делает все поля optional — как `Partial<T>` в TypeScript, но для DTO. Не нужно дублировать декораторы.

### Step 2: Entity (Alex предоставляет)

```typescript
// todo.entity.ts
import { Entity, PrimaryGeneratedColumn, Column, CreateDateColumn } from 'typeorm';

@Entity()
export class Todo {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column({ default: false })
  completed: boolean;

  @CreateDateColumn()
  createdAt: Date;
}
```

**Alex**: Это как TypeScript interface, но с ORM-декораторами. `@Column()` говорит TypeORM: "это поле в таблице todos". `@CreateDateColumn()` — автоматически ставит timestamp при создании. Ты когда-нибудь работал с Prisma или Drizzle?

**Ученик**: Немного с Prisma. Похоже, только декларативное, а не через schema-файл.

**Alex**: **Точно.** TypeORM — "active record" подход, декораторы прямо на классе. Prisma — "data mapper", schema отдельно. Оба подхода валидны — в курсе W1-W3 используем TypeORM (проще для монолита), потом перейдём на Prisma для микросервисов.

### Step 3: Service — Skeleton с TODO

```typescript
// todo.service.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Todo } from './todo.entity';
import { CreateTodoDto } from './dto/create-todo.dto';
import { UpdateTodoDto } from './dto/update-todo.dto';

@Injectable()
export class TodoService {
  constructor(
    @InjectRepository(Todo)
    private todoRepository: Repository<Todo>,
  ) {}

  // TODO 1: Find all todos with pagination
  // HINT: Use this.todoRepository.find({ skip, take })
  // skip = (page - 1) * limit, take = limit
  async findAll(page: number = 1, limit: number = 10): Promise<Todo[]> {
    // Ученик пишет:
    const skip = (page - 1) * limit;
    return this.todoRepository.find({
      skip,
      take: limit,
      order: { createdAt: 'DESC' },
    });
  }

  // TODO 2: Find one todo by id
  // HINT: Use this.todoRepository.findOne({ where: { id } })
  // IMPORTANT: If not found, throw new NotFoundException()
  async findOne(id: number): Promise<Todo> {
    // Ученик пишет:
    const todo = await this.todoRepository.findOne({ where: { id } });
    if (!todo) {
      throw new NotFoundException(`Todo #${id} not found`);
    }
    return todo;
  }

  // TODO 3: Create todo
  // HINT: Use this.todoRepository.save(createTodoDto)
  // The DTO already validated — just save it
  async create(createTodoDto: CreateTodoDto): Promise<Todo> {
    // Ученик пишет:
    const todo = this.todoRepository.create(createTodoDto);
    return this.todoRepository.save(todo);
  }

  // TODO 4: Update todo — THIS CONTAINS AN INTENTIONAL MISTAKE
  // HINT: Find todo first, then merge changes, then save
  async update(id: number, updateTodoDto: UpdateTodoDto): Promise<Todo> {
    // Ученик получает этот код с багом:
    const todo = await this.todoRepository.findOne({ where: { id } });
    // BUG: No check if todo exists! No NotFoundException thrown
    Object.assign(todo, updateTodoDto);
    return this.todoRepository.save(todo);
  }

  // TODO 5: Delete todo
  // HINT: Use this.todoRepository.delete(id)
  // Check result.affected === 0 → NotFoundException
  async remove(id: number): Promise<void> {
    // Ученик пишет:
    const result = await this.todoRepository.delete(id);
    if (result.affected === 0) {
      throw new NotFoundException(`Todo #${id} not found`);
    }
  }
}
```

**Ученик**: Написал все TODO. Но в `update` что-то странное — ты не добавил проверку на существование, хотя в `findOne` и `remove` она есть.

**Alex**: **Ты нашёл баг!** Отличный глаз. Это intentional mistake — я специально пропустил проверку, чтобы ты заметил несоответствие. Почему это критично?

**Ученик**: Если передать несуществующий id — `findOne` вернёт `null`, `Object.assign(null, ...)` упадёт с ошибкой, или если todo undefined...

**Alex**: **Правильно.** `Object.assign(null, updateTodoDto)` выбросит `TypeError: Cannot convert undefined or null to object`. Это **500 Internal Server Error** вместо **404 Not Found** — ужасный UX. Клиент получит "что-то сломалось" вместо "ресурс не найден". Исправь:

```typescript
async update(id: number, updateTodoDto: UpdateTodoDto): Promise<Todo> {
  const todo = await this.todoRepository.findOne({ where: { id } });
  if (!todo) {
    throw new NotFoundException(`Todo #${id} not found`);
  }
  Object.assign(todo, updateTodoDto);
  return this.todoRepository.save(todo);
}
```

**Alex**: Обрати внимание на паттерн: **"find → check → modify → save"**. Он повторяется в `findOne`, `update`, `remove`. Это как на фронте: "проверяем props → рендерим → обрабатываем события". На бэкене — **"проверяем существование → обрабатываем → возвращаем результат"**.

### Step 4: Controller (Alex предоставляет полностью)

```typescript
// todo.controller.ts
import { Controller, Get, Post, Patch, Delete, Body, Param, Query, ParseIntPipe } from '@nestjs/common';
import { TodoService } from './todo.service';
import { CreateTodoDto } from './dto/create-todo.dto';
import { UpdateTodoDto } from './dto/update-todo.dto';

@Controller('todos')
export class TodoController {
  constructor(private readonly todoService: TodoService) {}

  @Get()
  findAll(@Query('page') page = 1, @Query('limit') limit = 10) {
    return this.todoService.findAll(+page, +limit);
  }

  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number) {
    return this.todoService.findOne(id);
  }

  @Post()
  create(@Body() createTodoDto: CreateTodoDto) {
    return this.todoService.create(createTodoDto);
  }

  @Patch(':id')
  update(@Param('id', ParseIntPipe) id: number, @Body() updateTodoDto: UpdateTodoDto) {
    return this.todoService.update(id, updateTodoDto);
  }

  @Delete(':id')
  remove(@Param('id', ParseIntPipe) id: number) {
    return this.todoService.remove(id);
  }
}
```

**Alex**: Controller — это как **React Router + обработчики событий**. `@Controller('todos')` — базовый путь, как `<Route path="/todos">`. `@Get()`, `@Post()` — HTTP-методы, как обработчики onClick. `@Param()`, `@Body()`, `@Query()` — извлечение данных из request, как `useParams()`, `event.target.value`.

**Alex**: `ParseIntPipe` — это гарантия типобезопасности: если клиент отправит `GET /todos/abc`, NestJS автоматически вернёт **400 Bad Request** с "Validation failed (numeric string is expected)". Это как PropTypes в React — runtime type checking.

---

### Step 5: React Frontend — Connecting to API

**Alex**: Теперь фронт. Ты уже senior во фронте, так что код ты напишешь сам. Но давай обсудим архитектурное решение. Как будешь работать с API?

**Ученик**: TanStack Query — useQuery для GET, useMutation для POST/PATCH/DELETE. Optimistic updates для create и toggle completed.

**Alex**: **Идеально.** И вот здесь красота: **те же паттерны, другая сторона**. На фронте — optimistic update через TanStack Query. На бэкене — тот же CRUD, но с persistence. Давай skeleton:

```typescript
// hooks/useTodos.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { todoApi } from '../api/todoApi';

export const useTodos = (page = 1, limit = 10) => {
  return useQuery({
    queryKey: ['todos', page, limit],
    queryFn: () => todoApi.getAll(page, limit),
  });
};

export const useCreateTodo = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: todoApi.create,
    onSuccess: (newTodo) => {
      // TODO: Implement optimistic update
      // HINT: queryClient.setQueryData(['todos', 1, 10], (old) => [newTodo, ...old])
      // Don't forget to invalidate on error!
      
      // Ученик пишет:
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });
};
```

**Alex**: Хорошо, но `invalidateQueries` — это **pessimistic** update. Давай сделаем **optimistic** — как ты бы сделал для инстантного UX?

**Ученик**: А, точно. Нужно `onMutate` — сразу добавить в кэш, потом `onError` — откатить:

```typescript
return useMutation({
  mutationFn: todoApi.create,
  onMutate: async (newTodo) => {
    await queryClient.cancelQueries({ queryKey: ['todos'] });
    const previous = queryClient.getQueryData(['todos', 1, 10]);
    queryClient.setQueryData(['todos', 1, 10], (old) => [newTodo, ...old]);
    return { previous };
  },
  onError: (err, newTodo, context) => {
    queryClient.setQueryData(['todos', 1, 10], context?.previous);
  },
  onSettled: () => {
    queryClient.invalidateQueries({ queryKey: ['todos'] });
  },
});
```

**Alex**: **Отлично!** Ты только что продемонстрировал критический паттерн: **optimistic updates с rollback**. Это то, что делает ваш Social Feed (W1) — лайк мгновенно появляется, откатывается при ошибке. Но в W1 backend ещё сложнее: нужно idempotency, чтобы при retry не создать дубликат.

---

**[Phase 3 завершена: написан NestJS backend + React frontend. 1 intentional mistake найден и исправлен. Scaffolding: 60% — skeleton с TODO и hints.]**

---

## Phase 4: Review — Code Review по Рубрике

**Время**: 55-70 минут

---

**Alex**: Давай проведём code review. Я буду по рубрике, но не просто "правильно/неправильно" — с объяснением **почему**.

### Code Review Session

```
## Code Review: To-Do List (W0 — Warm-up Project)

### Что сделано хорошо ✅

1. **RESTful API design** — ресурсно-ориентированный подход, правильные HTTP методы.
   CRUD через /todos — canonical REST. Это покрывает паттерн P01 из нашего курса.

2. **Separation of concerns** — controller handles HTTP, service handles business logic.
   Как React container/presentational components: controller = container (data fetching),
   service = reducer (business logic).

3. **DTO + ValidationPipe** — runtime валидация через декораторы.
   Это как Zod на фронте, но на уровне фреймворка. Покрывает security-тему
   "Input Validation" для W1.

4. **Error handling** — NotFoundException для несуществующих ресурсов.
   Правильные HTTP статусы: 404 для missing, 400 для validation errors.
   Как Error Boundary на фронте — каждый слой обрабатывает свои ошибки.

5. **Optimistic updates** на фронте — мгновенный UX с rollback при ошибке.
   Это preview паттерна для W1 (optimistic UI in Social Feed).

### Что можно улучшить ⚠️

1. [MEDIUM] **No database transaction for update.** 
   Сейчас: find → check → assign → save — 2 запроса к БД.
   Можно оптимизировать через `this.todoRepository.update(id, updateTodoDto)`
   — один запрос, но тогда нужно отдельно проверять существование.
   Trade-off: 1 vs 2 запросов, но `update()` не возвращает обновлённый объект.

2. [LOW] **Нет сортировки по умолчанию в findAll.**
   `order: { createdAt: 'DESC' }` — хорошо, но если БД большая,
   нужен index на createdAt. Без index — full table scan.
   Это preview для W5 (Video) — indexing стратегии.

3. [LOW] **No rate limiting.**
   Сейчас клиент может спамить POST /todos бесконечно.
   Для W1 мы добавим `@nestjs/throttler` — token bucket per-IP.

### Критический баг найден и исправлен 🐛

- **Missing null check in update()** — intentional mistake.
  Исправлено: добавлен NotFoundException перед Object.assign().
  Это типичная ошибка junior-разработчиков: "если я нашёл в одном методе,
  значит и тут найду". Но каждый метод — независимый entry point.

### Архитектурные мысли на будущее 🤔

- Если Todo привязан к User — добавляем @ManyToOne(User).
  Тогда findAll должен фильтровать по `where: { userId }`.
  Auth — W2 (E-commerce).

- Если нужна full-text search по title — PostgreSQL `@Index({ fulltext: true })`
  или Elasticsearch. Search — W2 (E-commerce autocomplete).

- Если нужна real-time sync между устройствами — WebSocket + SSE.
  Real-time — W3 (Chat).
```

**Alex**: Вопрос для reflection: объясни своими словами, почему сервер не может доверять клиенту — даже если клиент — твоё React-приложение, которое ты сам написал.

**Ученик**: Потому что клиент — это не только React-приложение. Любой может отправить HTTP-запрос через curl или Postman. Валидация на клиенте — для UX, валидация на сервере — для security.

**Alex**: **В точку.** Это один из ключевых принципов backend-разработки: **"never trust the client"**. На фронте ты контролируешь весь код — это твой домен. На бэкене API публичен — любой может его вызвать. Это фундаментально меняет подход к безопасности.

---

**[Phase 4 завершена: code review по рубрике, reflection, articulation. Scaffolding: fading — ученик самостоятельно артикулировал принцип.]**

---

## Мета-Рефлексия: Что Мы Только Что Увидели

### Как Методология Проявилась в Уроке

| Метод Когнитивной Аппрентисшип | Где Проявился | Пример из Диалога |
|:---|:---|:---|
| **Modeling** | Phase 1: Theory | Alex verbalizes мышление: "Мой первый вопрос при виде multi-step process..." |
| **Coaching** | Phase 2: Socratic | Вопросы на диагностику: "Что если 10,000 записей?" |
| **Scaffolding** | Phase 3: Implementation | Skeleton code с TODO и hints |
| **Intentional Mistake** | Phase 3: Update method | Пропущен null check — ученик нашёл сам |
| **Articulation** | Phase 4: Review | "Объясни своими словами, почему сервер не доверяет клиенту" |
| **Reflection** | Phase 4: Review | Сравнение с expert solution, side-by-side comments |
| **Analogical Reasoning** | Везде | Middleware = HOC, DTO = Zod, Optimistic updates = TanStack Query |

### Что Ученик Освоил за 70 Минут

1. **NestJS архитектура**: Controller → Service → Repository → Entity
2. **DTO + ValidationPipe**: runtime валидация декларативно
3. **REST API design**: ресурсно-ориентированный подход, HTTP методы
4. **Error handling**: NotFoundException, 404 vs 500
5. **CRUD с persistence**: отличие от in-memory frontend state
6. **"Never trust the client"**: фундаментальный принцип backend security
7. **Optimistic updates**: bridge между frontend и backend patterns
8. **Pagination**: offset-based, edge case с drift

---

## Критика: Что Не Сработало и Что Нужно Доработать

### 1. Скорость: Урок Занял 70 Минут на To-Do List

**Проблема**: Для самого простого CRUD 70 минут — это много. В вашем курсе W1 — Social Feed с GraphQL, Redis, CDN, Rate Limiting. Если To-Do занял 70 минут, сколько займёт W1?

**Решение**: Для warm-up (W0) 70 минут нормально — это **calibration** скорости. Для W1+ Alex должен **ускорять** scaffolding — меньше объяснять, что такое DTO, больше фокусироваться на новых паттернах (GraphQL, Caching). На W3 skeleton code для нового сервиса — 20 строк с 3 TODO, не 60 строк с 8 TODO.

### 2. Риск: Ученик Может Застрять на Обсуждении

**Проблема**: В Phase 2 Alex задавал вопросы про пагинацию и server-generated fields. Ученик ответил правильно, но если бы запутался — диалог мог растянуться. Критерий переключения с вопроса на hint — **нечёткий**.

**Решение**: Нужен **явный таймбокс**: "2 минуты на обсуждение, потом hint". Или явный критерий: "если ученик не даёт направление ответа за 30 секунд — switch to scaffolding".

### 3. Intentional Mistake: Не Все Будут Найдены

**Проблема**: В примере ученик нашёл баг сразу — потому что паттерн "find → check" повторялся в двух других методах. Если баг был бы **изолирован** — например, race condition в concurrent update — ученик бы его пропустил.

**Решение**: Нужна **иерархия сложности** intentional mistakes:

| Уровень | Тип | Как Найти | Когда Внедрять |
|:---|:---|:---|:---:|
| **Level 1** | Missing null check | Паттерн recognition (другие методы делают check) | W1-W2 |
| **Level 2** | Wrong HTTP status | Например, 200 вместо 201 для POST | W2-W3 |
| **Level 3** | Missing auth check | Нет guard на endpoint | W2-W3 |
| **Level 4** | Race condition | Concurrent update без optimistic lock | W4-W5 |
| **Level 5** | N+1 query | findOne внутри цикла | W5-W6 |

### 4. Frontend-Аналогии: Риск Overfitting

**Проблема**: "DTO = Zod" — хорошая аналогия, но у DTO есть аспекты, которых у Zod нет: `@nestjs/swagger` автоматически генерирует OpenAPI spec, DTO может включать transformation (`@Transform()`). Если ученик думает "DTO — это просто Zod на сервере", он упустит эти возможности.

**Решение**: Каждая аналогия должна сопровождаться **"границей применимости"**: "DTO похож на Zod, но также включает transformation и Swagger-интеграцию — это то, чего Zod не делает".

### 5. State Injection: YAML Будет Огромным к W10

**Проблема**: К W10 state.yml будет содержать 12 паттернов × 3 языка × 10 проектов — YAML разрастается. Context window LLM имеет лимит, и огромный state будет "съедать" токены.

**Решение**: **Иерархический state** — только текущий проект в полном виде, остальные — summary:

```yaml
# W10 — полный контекст текущего проекта
current_project: { ...full details... }

# W1-W9 — summary только
previous_projects:
  w1_social_feed: { patterns: [P01, P03, P10], mastered: true }
  w2_ecommerce: { patterns: [P02, P06], mastered: true }
  # ... и т.д.
```

### 6. Эмоциональный Фактор: "Просто Дай Мне Код"

**Проблема**: В реальной сессии ученик может разозлиться: "Я senior, я знаю, что делаю, просто покажи код!". Alex по методологии должен **твёрдо отказать** — но это создаёт friction.

**Решение**: Нужен **"escape hatch"**: если ученик настаивает 3+ раза — Alex даёт worked example, но с условием: "Окей, вот полный код. Но обещай: после реализации ты объяснишь, как он работает, своими словами. Deal?"

### 7. Непрерывное Взаимодействие: Fatigue

**Проблема**: Методология предполагает непрерывное взаимодействие в течение 2-4 часов. Но даже 70-минутный урок на To-Do list потребовал **интенсивного внимания** — вопросы, объяснения, написание кода, review. 4 часа подряд — cognitive overload.

**Решение**: **Micro-sessions**: 45-60 минут активного взаимодействия, затем 15-минутный перерыв. Alex может генерировать **"homework"** на перерыв: "Прочитай про REST vs GraphQL trade-offs, приходи с вопросами".

---

## Итог: Методология Валидна, Но Требует Калибровки

### Что Работает Идеально

1. **Frontend аналогии** — мощный мост. Ученик мгновенно понимает DTO через Zod, optimistic updates через TanStack Query.
2. **Explain-then-implement** — предотвращает слепое копирование. Ученик думает перед кодом.
3. **Intentional mistakes** — найденный баг оставляет **эмоциональный след**. Ученик запомнит: "А, в update всегда проверяй существование".
4. **Code review по рубрике** — структурирует feedback и создаёт ощущение прогресса.

### Что Требует Доработки

1. **Таймбоксы** — явные лимиты на каждую фазу. Без них Socratic discussion растягивается.
2. **Escape hatch** — для случаев, когда ученик настаивает на прямом ответе.
3. **Hierarchical state** — чтобы state.yml не разрастался до неуправляемых размеров.
4. **Fatigue management** — micro-sessions вместо 4-часовых марафонов.
5. **Edge case диаграмма** — когда intentional mistake не найден, Alex должен иметь план B: "Давай я покажу, что пропустили, и объясню, почему это важно".
