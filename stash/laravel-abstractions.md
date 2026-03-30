---
alwaysApply: false
---

## Action Classes

**Preferred approach for complex business logic** - Use Action classes instead of observers or complex events.

- **Reduces cognitive load**: All related logic is kept top-to-bottom in one file, making functionality easier to understand and test.
- **Better than observers**: Actions orchestrate functionality without hiding side effects or scattering logic across the application.
- **Better than services**: Use Action classes over Service unless the project already uses Services.
- **Transaction safety**: For larger actions, wrap all logic inside a database transaction.
- **Command pattern**: Actions follow the Command pattern for executing functionality and orchestrating other functions.

### Action Setup

- Implement the `execute()` method with all business logic
- if logic is bigger abstract in couple of private methods
- Dispatch from controllers: `CreateUserAction::execute($data);`

---

## Request Classes

- In `authorize()`, use the **Gate** facade to check permissions instead of returning `true`.
- In `rules()`, always define **min** and **max** values and the **data type** for each column.
- You can add an `after()` method to perform extra validation (only if the initial validation passes).
  Example: limit each user to a maximum of 5 products.

---

## Observers

**⚠️ Avoid observers for complex business logic** - They introduce significant risks and should be used sparingly.

### When NOT to Use Observers

- **Avoid for core business logic**: Observers scatter logic away from the main execution path, making it difficult to discover hidden side effects.
- **Testing difficulties**: Observers make tests brittle and tightly coupled to implementation details.
- **Lack of meaning**: Generic model events (UserUpdated, UserCreated) don't explain what actually happened in business terms.

### When to Use Observers (Limited Cases)

- **Simple middleware logic**: Only for logging, auditing, or basic data validation.
- **Guard invariants**: Use `saving()` event to ensure data is valid before persistence.
- **Avoid complex side effects**: Don't send emails, update counters, or dispatch jobs from observers.

### Observer Setup and Registration

- Create observers using `php artisan make:observer UserObserver --model=User` which places them in `app/Observers/`.
- Register observers using the `#[ObservedBy([UserObserver::class])]` attribute on the model class, or manually in `AppServiceProvider::boot()` with `User::observe(UserObserver::class)`.
- Observer methods receive the affected model as their only argument and should be named after Eloquent events: `created()`, `updated()`, `deleted()`, `restored()`, `forceDeleted()`, `saving()`, `retrieved()`.

### Transaction Safety

- Implement `ShouldHandleEventsAfterCommit` interface on observers when you need event handlers to execute only after database transactions are committed.
- This prevents side effects from running if the transaction is rolled back, ensuring data consistency.
- If no transaction is in progress, events execute immediately.

### Event Control

- Use `withoutEvents()` method to temporarily mute all model events during bulk operations or when you need to bypass observers.
- Use `saveQuietly()`, `deleteQuietly()`, `forceDeleteQuietly()`, `restoreQuietly()` methods to perform operations without triggering any events.
- These methods are useful for data migrations, seeding, or when you need to perform operations that shouldn't trigger business logic.

---

## Mutators / Accessors / Casts

- Ensure stored and retrieved data are consistently shaped.
- Examples:
  - Hash passwords on set; format dates on get.
  - Convert JSON or database primitives to value objects (via custom casts).
  - Expose computed attributes without adding columns.
- Keep attribute-level logic close to the model for type-safety and predictable transformations.
- Avoid triggering external systems from these methods.

---

## Jobs

- Use Jobs for background/async work; prefer Actions for immediate in-process orchestration.
- Create with `php artisan make:job NameJob`. Implement `ShouldQueue` and use traits: `Dispatchable, InteractsWithQueue, Queueable, SerializesModels`.
- Pass small, serializable data (IDs/primitives). If passing models, rely on `SerializesModels`, but prefer IDs for clarity and resilience.
- Dispatch from controllers/services: `NameJob::dispatch($userId);` Optionally set queue, delay: `->onQueue('high')->delay(now()->addSeconds(5));`
- Ensure DB writes are committed before execution: set `$afterCommit = true;` on the Job for transaction safety.
- Control reliability/perf: set `$tries`, `$backoff`, and `$timeout` on the Job; handle `fail()` for cleanup/alerts.
- For dependent work, use chains (`Bus::chain([...])`). For grouped work or progress, use batches (`Bus::batch([...])`).
- Keep handlers idempotent (safe to retry) and side-effect aware; guard against duplicates.
- Example flow: after user registration, dispatch `NewUserDataJob` to create demo data without blocking the response.

---

## Events and Listeners

**Use behavior-driven events instead of generic model events** for meaningful business communication.

### Event Naming: Behavioral vs Generic

**✅ Recommended (Behavioral Events):**

- `OrderPlaced` (not `OrderUpdated`)
- `PaymentSucceeded` (not `PaymentUpdated`)
- `UserDisabled` (not `UserSaved`)
- `OrderShippingAddressChanged` (not `OrderUpdated`)

**❌ Avoid (Generic/Data Events):**

- Generic model events like `UserUpdated`, `OrderCreated` - they lack business meaning
- Events tied to Active Record changes don't explain what actually happened

### Event Execution: Always Asynchronous

- **Always queue events**: Make listeners implement `ShouldQueue` for asynchronous execution.
- **Prevents request failure**: If a listener fails, it won't break the HTTP request or leave the database in an invalid state.
- **Decouples actors**: Allows new logic to be added without modifying the dispatcher.

### Atomic State Changes

- **Dispatch within transactions**: Event dispatch must be associated with state changes as an atomic unit.
- **Guarantee persistence**: Listeners only run after the state has been successfully persisted.
- Use `DB::transaction()` to wrap both the state change and event dispatch.

### Event Setup

- Create with `php artisan make:event OrderPlaced` and `php artisan make:listener SendOrderConfirmation --event=OrderPlaced`.
- Keep Event classes small; pass only the data listeners need (e.g., `public function __construct(public Order $order) {}`).
- Dispatch from controllers/services: `OrderPlaced::dispatch($order);` alongside actions to keep controllers thin.
- Listeners receive the typed event in `handle(Event $event)`. Move controller side‑effects into listeners (reports, notifications, etc.).
- Auto-registration: listeners in `app/Listeners` are discovered; otherwise register in `EventServiceProvider`.
- Keep listeners idempotent and side‑effect aware; avoid heavy logic in the event itself.

---

## Helpers (Classes and Static Methods)

- Use Helper classes for small, reusable, stateless utilities shared across layers (e.g., `DateHelper`, `CurrencyHelper`).
- Prefer class-based helpers with `public static` methods over global functions to keep code OOP and avoid name collisions.
- Keep helpers pure: depend only on inputs, not app state; make them easily unit-testable.
- Organize under `app/Helpers/NameHelper.php`; no Artisan command needed. Group by domain/topic, not by caller.
- Example: move date normalization out of models into `DateHelper::convertToDB($date)` and reuse in mutators, actions, or requests.
- Don't overuse helpers: if logic belongs to a Value Object or Domain Service, put it there instead.
- Type-hint parameters/returns and avoid hidden side-effects or I/O inside helpers.

---

## Query Scopes (Global and Local)

- Use scopes to encapsulate reusable query constraints and keep queries DRY across the application.
- **Global Scopes**: Auto-apply to all queries for a model (e.g., soft deletes, tenant filtering). Create with `php artisan make:scope NameScope`, implement `Scope` interface, and register via `#[ScopedBy([NameScope::class])]` or in `booted()`.
- **Local Scopes**: Explicit, chainable filters invoked when building queries. Define with `#[Scope]` attribute on protected methods; return `void` and modify `$query` in place.
- Prefer global scopes for model-wide invariants that must always apply (multi-tenancy, published-only). Opt out with `withoutGlobalScope(NameScope::class)` when needed.
- Prefer local scopes for common, optional filters reused across queries (`popular()`, `active()`, `ofType($type)`). Chain them for readability: `User::active()->verified()->get()`.
- Dynamic scopes accept parameters after `$query`: `protected function ofType(Builder $query, string $type): void`.
- Use `addSelect()` in global scopes to avoid overwriting existing select clauses.
- Keep scopes pure: no side-effects, only query building. Don't hide authorization in scopes; use policies/gates.
- Avoid scopes for one-off filters (inline where clause is clearer) or complex joins (use query builder methods or repositories).
