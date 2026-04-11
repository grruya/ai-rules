## Laravel Type-Safety Standard

### Goal

Make invalid data hard to pass: use types at boundaries, small immutable objects for structured data, and static analysis where the project uses PHPStan.

### Value object vs DTO

| Kind             | Role                                                     | Behavior                                                                                        |
| ---------------- | -------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| **Value object** | Domain concept with rules (`Money`, `Email`, `Duration`) | Validates in constructor or static factories; may expose small domain methods; compare by value |
| **DTO**          | Data between layers (HTTP -> service, service -> job)    | No business logic; readonly typed properties; build at boundaries after validation              |

Do not use Eloquent models as DTOs. Do not put persistence logic on DTOs.

### Replace mystery arrays with types

- Avoid untyped `import($data)` or parameters named `$data` when the shape is known; use a named type (`ImportTransaction`, `TransactionImportRow`, etc.).
- Prefer `array $things` only when the structure is genuinely variable; otherwise use a class with typed properties.
- Typed parameters give IDE completion, refactors, and clearer contracts.

### Value objects

- **Construction:** Private constructor + static factories (`Duration::seconds(5)`, `Money::fromMajor(...)`) so every instance is valid.
- **Validation:** Enforce invariants in constructor or factory; throw domain- or `InvalidArgumentException` when rules break (e.g. non-negative duration, discount in range).
- **Immutability:** `readonly` properties; operations return new instances.
- **Eloquent:** Implement `CastsAttributes` and register on `$casts` to map DB columns to/from the value object; `set()` must reject wrong types.

Example shape:

```php
final readonly class Duration
{
    private function __construct(public int $seconds) {}

    public static function seconds(int $seconds): self
    {
        if ($seconds < 0) {
            throw new InvalidArgumentException('Duration must be non-negative.');
        }

        return new self($seconds);
    }
}
```

### DTOs

- **Properties:** `public readonly` with scalar or value-object types as appropriate.
- **Factories:** `fromArray()`, `fromRequest()` (or from validated array), optionally `fromModel()` for read paths; keep mapping explicit.
- **Serialization:** `toArray()` only at edges (response, queue payload), not scattered through domain code.
- **Flow:** Validate first (Form Request, `validate()`, etc.), then construct the DTO, then pass inward to services.

**Small and single-purpose:** One DTO per direction and use case. Do not reuse one `UserDTO` for create, update, API responses, and internal jobs if fields differ.

**Input vs output:** Prefer separate types (e.g. `CreateUserDTO` / `UpdateUserDTO` for commands in, `UserResponseDTO` or `BookingsResponseDTO` for responses out).

**Naming:** Use intent in the class name: action or context + `DTO` (`CreateUserDTO`, `BookingsResponseDTO`, `SyncInvoicePayloadDTO`). Avoid generic names like `UserDTO` when the type serves one flow.

```php
final readonly class CreateUserDTO
{
    public static function fromValidated(array $v): self
    {
        return new self(
            name: $v['name'],
            email: $v['email'],
        );
    }

    public function __construct(
        public string $name,
        public string $email,
    ) {}
}
```

Use DTOs for controller -> service, job/event constructors, consistent API shapes, and complex payloads. Skip for trivial one-off parameters.

### Immutability defaults

- Default new DTOs and value objects to `readonly` (PHP 8.2+ readonly classes where suitable).
- Do not force readonly on Eloquent models, fluent builders, or legacy mutable flows.

### PHPStan

- Use `@template T` on reusable collection-like or repository-style abstractions; document `Collection<ItemType>` in `@var` / `@return` where needed.
- Use literal unions (`'get'|'post'`), `value-of<BackedEnum>`, `key-of`, and `class-string<SomeInterface>` when it prevents real bugs.
- Skip heavy generic markup if the project does not run PHPStan.

### Domain invariants vs input validation

- **Input validation:** Format, presence, HTTP concerns at the boundary.
- **Domain invariants:** Rules that must always hold for the model (`start < end`, cannot pay twice) inside domain/value objects/services; throw domain exceptions.

### Combining value objects

Prefer expressive types working together (`$money->apply($rate)`) over primitive math scattered in services, when complexity warrants it.

### When not to over-engineer

- Simple key-value maps, highly dynamic external payloads, or hot paths where profiling shows object overhead matters.
- Readonly everywhere on code that must stay mutable for framework or legacy reasons.
