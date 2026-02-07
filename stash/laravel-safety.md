---
alwaysApply: false
---

## 🧩 Objects as Pseudo-Structs: Replace Arrays with Typed Objects

**Rule:** Use objects instead of associative arrays when working with structured data to gain type safety, better tooling, and clearer domain representation.

**How to use:**
When you have structured data that follows a consistent pattern, create a class with typed properties that represent the data structure.
Instead of passing around arrays with keys like `invoice_due_date` and `total_amount`, create an object with properly typed properties and meaningful names.
This gives you IDE autocompletion, type checking, and self-documenting structure.

**When to use:**

- Structured data with known keys and types
- Data transfer between application layers
- When you need type safety and IDE support
- Representing domain concepts with clear structure
- Enforcing required fields and data types
- API responses needing consistent formatting

**When NOT to use:**

- Simple key-value mappings
- Highly variable data
- Performance-critical code
- External API requirements

**Key:** Objects provide structure, type safety, and domain meaning that arrays cannot — making your code more maintainable and less error-prone.

---

## 🏗️ Static Factory Methods: Control Object Creation with Named Constructors

**Rule:** Use private constructors with static factory methods to control instantiation and provide clear, named ways to create objects.

**How to use:**
Make your constructor private so the class can only be instantiated from within itself.
Then create static methods like `fromInvoice()`, `createWithDefaults()`, or `forUser()` that act as named constructors, ensuring valid state creation.

**When to use:**

- You need validation or transformation before creation
- Objects can be created in multiple ways
- To prevent direct instantiation
- When objects need setup or configuration
- To centralize creation logic for easier refactoring

**When NOT to use:**

- Simple constructors
- Easy-to-track instantiation
- Rarely created objects
- Dynamic creation needs

**Key:** Static factories create predictable, refactor-friendly instantiation while enforcing valid states.

---

## 🧱 Immutability: Use `readonly` for Predictable Object State

**Rule:** Make classes and properties `readonly` by default to prevent accidental mutations.

**How to use:**
Add the `readonly` keyword before properties or class declarations to ensure properties cannot change after construction.

**When to use:**

- New classes (default to immutable)
- Value objects and DTOs
- Configuration objects
- Predictable state management
- Classes using immutable dependencies (`CarbonImmutable`, etc.)

**When NOT to use:**

- Eloquent models
- Fluent APIs
- Legacy code
- Mutable workflows

**Key:** `readonly` prevents mutation but doesn't freeze object dependencies — use immutable variants for full safety.

---

## 💎 Value Objects: Immutable, Self-Validating Domain Concepts

**Rule:** Represent domain concepts as immutable, self-validating objects compared by value, not identity.

**How to use:**
Use a private constructor that validates inputs.
Add static factory methods like `fromMajor()` or `fromEmail()`.
Use `readonly` properties or omit setters.
Return new instances from operations instead of mutating state.
Compare by value, not by instance.

**When to use:**

- Domain concepts with validation (`Email`, `Money`, `Address`)
- Immutable and value-based equality needs
- Replacing primitive obsession with meaningful types

**When NOT to use:**

- Entities with identity
- Large mutable state
- Simple data containers

**Key:** Value objects encapsulate domain rules and express behavior in a small, immutable unit.

---

## 📦 Data Transfer Objects (DTOs): Structured, Immutable Data Carriers

**Rule:** Use DTOs to carry validated, typed data between layers without behavior, keeping controllers thin and services decoupled from HTTP and persistence.

**How to use:**
Create small classes with `readonly` typed properties and named constructors like `fromRequest()` and `fromArray()`. Do validation and transformation at the boundary, then pass DTOs inward. Prefer `toArray()` for serialization only at edges.

**When to use:**

- Crossing layer boundaries (HTTP → domain/service, domain → response)
- Mapping `Request` data to use-cases without leaking HTTP into core code
- Aggregating values from multiple sources into a single input object
- Improving testability by constructing inputs without framework dependencies
- Freezing inputs to enforce immutability and predictable flows

**When NOT to use:**

- When a plain value object with behavior is more appropriate
- For trivial one-off parameters (keep it simple)
- Inside Eloquent models or as active records with persistence logic
- For frequently changing, unstructured payloads where arrays are clearer

**Key:** DTOs are simple, immutable containers for data transfer. Keep them behavior-free, construct them with named factories, and use typed properties for safety and clarity.

---

## 🔗 Value Object Interactions: Combine Rich Objects for Domain Logic

**Rule:** Combine value objects to express domain logic clearly, replacing primitives with meaningful types.

**How to use:**
Design objects that interact through well-defined methods — e.g., `Money` and `InterestRate` objects that collaborate via `money.applyInterest(rate)`.
All objects should be immutable and return new instances.

**When to use:**

- Domain concepts with behavior
- Replacing unclear primitive operations
- Multi-object calculations or transformations

**When NOT to use:**

- Simple data transfer
- Performance-critical calculations
- External API boundaries
- When primitives are sufficient

**Key:** Value objects make domain logic expressive, type-safe, and predictable.

---

## 🧮 Eloquent Custom Casts: Convert Primitives to Value Objects

**Rule:** Use custom casts to convert database primitives into rich value objects for type-safe domain models.

**How to use:**
Implement `CastsAttributes` with `get()` and `set()` to handle conversion automatically.
Register your cast in `$casts` on the model.

**When to use:**

- Converting primitives to domain types (`Money`, `Email`, etc.)
- Adding behavior to database fields
- Enforcing type safety across layers

**When NOT to use:**

- Simple primitive fields
- One-off conversions
- Frequently changing external data

**Key:** Custom casts bridge the gap between DB primitives and expressive, type-safe objects.

---

## 🧠 PHPStan Generics: Type-Safe Collections and Data Structures

**Rule:** Use `@template` annotations to create generic classes that enforce compile-time type safety.

**How to use:**
Add `@template T` in your class DocBlock and apply it to properties, parameters, and returns.
When instantiating, declare types with `@var Collection<Invoice>`.

**When to use:**

- Custom collections
- Generic data structures
- Type-safe reusable classes
- Compile-time error detection

**When NOT to use:**

- Simple arrays
- Single-type fixed classes
- Codebases without PHPStan adoption

**Key:** Generics prevent runtime type errors and make collections type-safe and self-documenting.

---

## ⚙️ PHPStan Advanced Typing: Generics & Literal Types

**Rule:** Use PHPStan's advanced typing features to restrict and validate values at static analysis time.

**How to use:**

- Use `@template` for generics
- Use `'get'|'post'|'put'` literal types
- Use `value-of<Enum>` and `key-of` for constants
- Use `class-string<Interface>` for type-safe class names

**When to use:**

- Type-safe collections
- Enum and literal constraints
- Class string validation
- Constant key/value enforcement

**When NOT to use:**

- Simple mixed-type arrays
- Codebases without PHPStan adoption

**Key:** Advanced types catch typos and invalid values _before_ runtime, increasing maintainability.

---

## 🧱 Domain Invariants: Make Illegal States Unrepresentable

**Rule:** Enforce rules that must always be true within your domain model to prevent invalid states.

**How to use:**
Embed business rule validation directly in constructors or domain methods.
Throw domain-specific exceptions when rules are violated.
Examples:

- Prevent paying an already paid invoice
- Enforce start time < end time for events

**When to use:**

- Business rules that must _never_ be broken
- Data integrity constraints
- Domain-specific invariants

**When NOT to use:**

- Input validation
- External API constraints

**Key:** Domain invariants live inside your business logic — not just validation — ensuring impossible states stay impossible.
