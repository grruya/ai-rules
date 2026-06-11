## Laravel Type-Safety Standard

### Goal

Make bad data hard to carry across boundaries: lean on types at edges, small immutable objects for structured data, and static analysis when PHPStan is in the project.

### Value object vs DTO vs typed array

| Kind         | Role                                                  | Behavior                                                                                         |
| ------------ | ----------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| Value object | Domain idea with rules (`Money`, `Email`, `Duration`) | Validate in the constructor or static factories; small domain methods are fine; compare by value |
| DTO          | Larger or nested data passed across multiple layers   | No business logic; readonly typed properties; build at the boundary after validation             |
| Typed array  | Validated payload local to one flow                   | Document the shape with PHPDoc and pass it directly where Laravel already handles the data well  |

Do not use Eloquent models as DTOs. Do not attach persistence logic to DTOs.

### Replace mystery arrays with types

- When the shape is known, avoid untyped `$data`; document it with PHPDoc like `array{name: string, email: string}`.
- Prefer PHPDoc typed arrays for validated Laravel request payloads that stay local to one flow, especially when the payload can be passed directly into Eloquent `create()`, `update()`, or `fill()`.
- Use a DTO only when the payload crosses multiple layers, is reused in several places, is very large, deeply nested, or represents a stable API/job/event contract.
- Typed array shapes improve IDE support and static analysis without adding unnecessary mapping code.

### Value objects

- Use a private constructor plus static factories (`Duration::seconds(5)`, `Money::fromMajor(...)`) so every instance is valid.
- Enforce invariants in the constructor or factory and throw domain exceptions or `InvalidArgumentException` when rules break (for example non-negative duration or discount in range).
- Prefer `readonly` properties and return new instances from operations that change state.
- For Eloquent, implement `CastsAttributes`, register it on `$casts`, and make `set()` reject invalid types.

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

- Use `public readonly` properties with scalars or value objects as needed.
- Provide explicit factories such as `fromArray()`, `fromRequest()` (or from a validated array), and optionally `fromModel()` on read paths.
- Call `toArray()` only at boundaries (responses, queue payloads), not throughout domain code.
- Validate first via Form Request, `validate()`, or equivalent. If the data stays local and maps cleanly to Eloquent, keep the validated array. If it crosses multiple layers or gets complex, build a DTO.

Keep each DTO small and single-purpose: one type per direction and use case. Avoid one `UserDTO` for create, update, API responses, and jobs when the fields diverge.

Prefer separate input and output types (for example `CreateUserDTO` and `UpdateUserDTO` for commands, `UserResponseDTO` or `BookingsResponseDTO` for responses).

Name types for intent: action or context plus `DTO` (`CreateUserDTO`, `BookingsResponseDTO`, `SyncInvoicePayloadDTO`). Skip vague names like `UserDTO` when the type serves a single flow.

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

Use DTOs for large or nested controller-to-service payloads, job and event constructors with stable contracts, stable API shapes, and heavy payloads. Skip them when a validated array is clearer, especially for straightforward Eloquent writes.

### DTO vs typed array

- Use a PHPDoc typed associative array by default for validated request data that stays in one flow, even when it has many fields, as long as the structure is still easy to read.
- Use a DTO when the payload is reused across many layers, very large, nested, represents an important contract, or needs named factory methods.

### Immutability defaults

- Default new DTOs and value objects to `readonly` (or readonly classes on PHP 8.2+ when it fits).
- Do not force readonly on Eloquent models, fluent builders, or legacy code that must stay mutable.

### PHPStan

- Add `@template T` on reusable collection- or repository-style abstractions; document `Collection<ItemType>` in `@var` and `@return` when it helps.
- Use literal unions (`'get'|'post'`), `value-of<BackedEnum>`, `key-of`, and `class-string<SomeInterface>` when they catch real defects.
- Skip deep generic annotations if the project does not run PHPStan.

### Domain invariants vs input validation

- Handle format, presence, and HTTP concerns at the boundary as input validation.
- Keep rules that must always hold for the model (`start < end`, cannot pay twice) inside domain types, value objects, or services, and throw domain exceptions there.

### Combining value objects

When complexity justifies it, prefer composed types (`$money->apply($rate)`) over raw arithmetic spread through services.

### When not to over-engineer

- Skip heavy typing for simple key-value maps, highly dynamic external payloads, or hot paths where profiling shows object overhead.
- Do not force readonly on code that the framework or legacy flows must keep mutable.
