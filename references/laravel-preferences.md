## Controllers

- A controller should only hold CRUD actions: `index`, `store`, `show`, `update`, `destroy`.
- Use a single-purpose invocable controller for anything else (e.g., `UserExportController`).
- Do not add private methods, business logic, or validation in controllers.

## Action classes for business logic that modifies data

- Make each action `final readonly` with a single public `handle()` and keep other methods `private`.
- Start the class name with a verb (e.g., `CreateOrder`, `DeleteUser`).
- Actions need to be only around data modification like CreateUser, UpdateUser, DeleteUser or some other cases like ChargeCustomer
- Actions must not be used for read operations, use Query class instead.
- Keep actions free of HTTP and session so any entry point can call them.
- Wrap multi-step database changes in `DB::transaction`, but expect pain on very large tables (e.g., history).

## Support classes for reusable domain logic without side effects

- Place reusable application domain logic in `app/Support` when it does not modify application state to be an Action nor it reads from database to be a Query class.
- Use Support classes for calculations, domain policy decisions, sorting or grouping in-memory data for business rules, or deriving text from trusted inputs.
- Do not use Support classes for API or UI output formatting; use Laravel Resources or frontend presentation code for that.
- Do not use Support classes just to map data to another structure; use DTOs when a named structure is needed.
- Make each Support class `final readonly` with only one public method named handle.
- Support classes must not use HTTP, session, request, response, controllers, database writes, queued jobs, emails, external API calls, cache writes, or file writes.
- Do not use generic names like `Manager`, `Service`, or `Processor`; name the class after the result or operation (e.g., `SortPropertiesByMinimumPrice`, `GenerateTermsOfServiceText`).

## Helper classes for simple non-business logic

- They need to have suffix `Helper`
- Need to be `final readonly`
- Every method should be `public static` no private methods allowed

## Query classes for bigger read logic

- Use Query class when u need bigger read operations since actions should not be used for this purpose
- Place them in `app/Queries` and follow this naming convention. If query gives back a list name it like `UserListQuery` if it does not give a list then just `CartChangeTimelineQuery`
- Query classes have only one public method named "builder" and return an Eloquent Builder the controller can chain.

## Migrations

- Avoid `onDelete('cascade')` in migrations and perform deletes in application code for clearer side effects.
- Avoid database defaults in migrations; set defaults inside Actions.

## Models

- Document every column with `@property` in PHPDoc for full static analysis coverage.

## DTOs, value objects, and typed arrays

- Use a Value Object for a domain concept with rules that must always be valid, like `Money`, `Email`, or `Duration`. Make it `final readonly`, validate in the constructor or static factory, and return new instances instead of mutating state.
- Prefer PHPDoc typed arrays for validated request payloads that stay in one flow, for example `array{name: string, email: string}`. This is fine even for bigger arrays when they map cleanly to Eloquent `create()`, `update()`, or `fill()`.
- Use a DTO only when the payload crosses multiple layers, is reused, is very large or nested, or represents a stable API, job, or event contract. Keep it `final readonly`, with typed public properties and factories like `fromValidated()` or `fromArray()`.
- Do not use Eloquent models as DTOs, and do not put business or persistence logic inside DTOs.
- Validate HTTP input first, then pass a typed array, DTO, or Value Object inward depending on the shape and lifetime of the data.

## Form Requests and Validation

- Use the `after` callback in Form Requests for validation checks that rely on the database or other post-basic-rule logic
- Apply sensible min and max rules on strings (e.g., `min:3` for a name).
- Prefer `#[CurrentUser] User $user` injection over `$request->user()`.

## Jobs

- Keep the job `$tries` value to 0.
- Pass serializable arguments such as a user id instead of heavy objects.
- Use the `DeleteWhenMissingModels` trait when the job references a model that might be deleted.

## File System and Atomic Operations

- If file operations must track a database transaction, commit the database work first and queue the file work.
- Call `deleteFileAfterSend()` on temporary download files so they are removed after the response finishes.
