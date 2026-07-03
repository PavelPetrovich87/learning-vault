# Adapter Pattern

Why use an Adapter instead of calling an external API directly from business logic? ::
It decouples business logic from vendor-specific APIs, data formats, and error types — enabling swap-ability and testability without rewriting consumers.
#flashcards/system-design

What is a "leaky abstraction" in the context of an adapter, and why is it dangerous? ::
When the adapter exposes vendor-specific details (e.g., raw `S3Error` or `StripeError`) to consumers. It forces upstream code to handle external quirks, defeating the purpose of the abstraction and creating vendor lock-in.
#flashcards/system-design

How should an adapter handle errors from an external service? ::
Map them to domain-specific errors (e.g., `StorageError('BUCKET_NOT_FOUND')`, `PaymentError('INSUFFICIENT_FUNDS')`) so consumers only deal with your unified error taxonomy.
#flashcards/system-design

Why must adapters be stateless? What goes wrong if they store state internally? ::
State makes adapters hard to unit-test and unsafe under concurrency. State (metrics, caching, request counts) should be injected as dependencies, not owned by the adapter.
#flashcards/system-design

If you need caching on top of an adapter, what is the correct design? ::
Use a **Cache Decorator** — a separate wrapper implementing the same interface. This keeps concerns separated, allows TTL and invalidation to be explicit, and lets you compose caching without modifying the core adapter.
#flashcards/system-design

What is the "N+1 problem" in adapters, and how do you prevent it? ::
The adapter fetches a list of IDs, then makes one individual HTTP/API call per item. Prevent it by using **bulk/batch APIs** or exposing pagination explicitly in the interface.
#flashcards/system-design

Why is using `any` for input/output types in an adapter a serious anti-pattern? ::
It defeats type safety and hides schema drift from external APIs. Use strict types with runtime validation (Zod, io-ts) at the boundary to catch breaking changes early.
#flashcards/system-design

What is a "God Adapter" anti-pattern, and how do you fix it? ::
When an adapter does more than fetch and adapt — e.g., runs business logic, persists data, or sends notifications. Fix it by splitting responsibilities: adapter only fetches/adapts, orchestration belongs to the pipeline/service layer.
#flashcards/system-design

How should an adapter handle API pagination to avoid silent data loss? ::
Either auto-paginate and collect all results internally, or expose pagination explicitly in the interface (`PaginatedResult<T>`). Never return only the first page without the consumer knowing.
#flashcards/system-design

When an external API introduces a breaking change, how do you evolve the adapter without breaking existing consumers? ::
Version the interface (`AdapterV1`, `AdapterV2`) and use a **bridge adapter** (e.g., `V2ToV1Adapter`) to let legacy consumers migrate gradually while new code uses the updated contract.
#flashcards/system-design

What is the single responsibility of an adapter? ::
**Translate** data and interface calls between an external system and your domain. Fetch, map types, normalize field names, and return. Nothing else.
#flashcards/system-design

Why should an adapter normalize optional/nullable fields from external APIs even if the docs say they are always present? ::
External APIs lie, change schemas, or return `null` under edge cases. Defensive programming (`?.` / `??`) prevents runtime crashes and makes the adapter resilient to schema drift.
#flashcards/system-design
