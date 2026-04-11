## Controllers

- **Slim Controllers:** Controllers must only contain CRUD methods (`index`, `store`, `show`, `update`, `destroy`).
- **Invocable Controllers:** For non-CRUD logic, use a single-purpose invocable controller (e.g., `UserExportController`).
- **No Private Logic:** Controllers must not contain private methods, business logic or validations. Delegate to **Action** classes and **FormRequests**.

## Action Classes

- **Structure:** Must be `final` and contain exactly one public method: `handle()`. If more methods are needed, they must be `private`.
- **Naming:** Must start with a verb (e.g., `CreateOrder`, `ProcessPayment`).
- **Agnosticism:** Actions must be agnostic of the entry point. Do not handle HTTP requests or sessions inside an Action.
- **Transactions:** Wrap multi-step database operations in `DB::transaction`, to ensure all related operations are atomic Be aware that transactions can become problematic when dealing with tables containing billions of records (e.g., history tables)

## Migrations

- **Avoid `down()` method:** Its a waste of time as production data only moves forward
- **Avoid `onDelete('cascade')`:** Its a best practice to avoid in migrations. Keep deletion logic on the application side This ensures you have full control over side effects
- **Avoid default in migrations:** Instead set the default value in Actions

## Models

- **`@property` PHPDoc blocks:** Use to ensure full type safety for every database column

## Complex queries

- **Query class:** For complex queries the recommended approach is to create a dedicated Query Class (e.g., `UserListQuery`) within an `app/Queries` folder. This class should return an Eloquent Builder instance, allowing the controller to easily chain further operations.

## Form Requests and Validation

- **Strict validation:** Setting sensible minimum and maximum constraints on strings (e.g., `min:3` for a name)
- **`after` callback:** Use it in Form Requests for validation checks that rely on the database or other post-basic-rule logic
- **`#[CurrentUser] User $user`:** Use this attribute method injection to access the authenticated user easily, rather than relying on `$request->user()`

## Jobs

- **Low tries:** Keep the job’s `$tries` property low, ideally one
- **Pass necessary context:** like user ID to the job as a serializable argument
- **`DeleteWhenMissingModels` trait** Use it to automatically ignore a job if the referenced model no longer exists

## File System and Atomic Operations

- **Asynchronous IO**: For File System operations that need to be “atomic” with a database transaction, perform the database operation first and then queue the File IO operation as a job
- **Temporary File Cleanup**: Use Laravel’s `deleteFileAfterSend()` method to ensure temporary files created for downloads are deleted after the response is sent
