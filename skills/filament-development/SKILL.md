---
name: filament-development
description: >-
  Develops admin panels and server-driven UI using Filament v4. Activates when
  creating or editing Filament resources, forms, tables, infolists, actions,
  or panels; working with Livewire in Filament; or when the user mentions
  Filament, admin panel, resource, form schema, or table columns.
---

# Filament Development

## When to Apply

Activate this skill when:

- Creating or modifying Filament resources (CRUD, forms, tables, infolists)
- Working with form components, schemas, or layout (Grid, Section, Tabs, Wizard)
- Adding or testing table columns, filters, or actions
- Building Filament panels or custom pages
- Testing Filament with Livewire

## Documentation

Use the `search-docs` tool for official Filament documentation on Artisan commands, code examples, testing, relationships, and idiomatic practices.

## Basic Usage

### Artisan

- Use Filament-specific Artisan commands to create files. Find them with `list-artisan-commands` or `php artisan --help`.
- Inspect required options and always pass `--no-interaction`.

### Conventions

- Follow existing Filament conventions in this application for where and how resources, forms, and tables are implemented.
- Filament is a Server-Driven UI (SDUI) framework for Laravel: define interfaces in PHP using structured configuration objects. It is built on Livewire, Alpine.js, and Tailwind CSS.

### Patterns

Use static `make()` methods to initialize components. Most configuration methods accept a `Closure` for dynamic values.

Use `Get $get` to read other form field values for conditional logic:

<code-snippet name="Conditional form field" lang="php">
use Filament\Forms\Components\Select;
use Filament\Forms\Components\TextInput;
use Filament\Schemas\Components\Utilities\Get;

Select::make('type')
->options(CompanyType::class)
->required()
->live(),

TextInput::make('company_name')
->required()
->visible(fn (Get $get): bool => $get('type') === 'business'),
</code-snippet>

Use `state()` with a `Closure` to compute derived column values:

<code-snippet name="Computed table column" lang="php">
use Filament\Tables\Columns\TextColumn;

TextColumn::make('full_name')
->state(fn (User $record): string => "{$record->first_name} {$record->last_name}"),
</code-snippet>

Actions encapsulate a button with optional modal form and logic:

<code-snippet name="Action with modal form" lang="php">
use Filament\Actions\Action;
use Filament\Forms\Components\TextInput;

Action::make('updateEmail')
->form([
TextInput::make('email')->email()->required(),
])
->action(fn (array $data, User $record): void => $record->update($data)),
</code-snippet>

## Testing

Authenticate before testing panel functionality. Filament uses Livewire, so use `livewire()` or `Livewire::test()`:

<code-snippet name="Filament Table Test" lang="php">
    livewire(ListUsers::class)
        ->assertCanSeeTableRecords($users)
        ->searchTable($users->first()->name)
        ->assertCanSeeTableRecords($users->take(1))
        ->assertCanNotSeeTableRecords($users->skip(1));
</code-snippet>

<code-snippet name="Filament Create Resource Test" lang="php">
    livewire(CreateUser::class)
        ->fillForm([
            'name' => 'Test',
            'email' => 'test@example.com',
        ])
        ->call('create')
        ->assertNotified()
        ->assertRedirect();

    assertDatabaseHas(User::class, [
        'name' => 'Test',
        'email' => 'test@example.com',
    ]);

</code-snippet>

<code-snippet name="Testing Validation" lang="php">
    livewire(CreateUser::class)
        ->fillForm([
            'name' => null,
            'email' => 'invalid-email',
        ])
        ->call('create')
        ->assertHasFormErrors([
            'name' => 'required',
            'email' => 'email',
        ])
        ->assertNotNotified();
</code-snippet>

<code-snippet name="Calling Actions" lang="php">
    use Filament\Actions\DeleteAction;
    use Filament\Actions\Testing\TestAction;

    livewire(EditUser::class, ['record' => $user->id])
        ->callAction(DeleteAction::class)
        ->assertNotified()
        ->assertRedirect();

    livewire(ListUsers::class)
        ->callAction(TestAction::make('promote')->table($user), [
            'role' => 'admin',
        ])
        ->assertNotified();

</code-snippet>

## Common Pitfalls

- **Wrong namespaces:** Form fields (TextInput, Select, etc.) and infolist entries (TextEntry, IconEntry) use `Filament\Forms\Components\`. Layout (Grid, Section, Fieldset, Tabs, Wizard) and schema utilities (Get, Set) use `Filament\Schemas\Components\` and `Filament\Schemas\Components\Utilities\`. Actions use `Filament\Actions\` (not `Filament\Tables\Actions\`). Icons: `Filament\Support\Icons\Heroicon` enum (e.g. `Heroicon::PencilSquare`).
- **File visibility:** Default is `private`. Use `->visibility('public')` for public access.
- **Layout spanning:** In Filament v4, `Grid`, `Section`, and `Fieldset` no longer span all columns by default; set column span when needed.
