# Outerweb Laravel Guidelines

## Table of Contents

- [Core](#core)
  - [Follow PHP conventions](#follow-php-conventions)
  - [Follow Laravel conventions](#follow-laravel-conventions)
  - [Formatting](#formatting)
  - [File names](#file-names)
  - [Imports](#imports)
  - [Typing](#typing)
  - [Comments / Docblocks](#comments--docblocks)
  - [Control flow](#control-flow)
  - [Strings](#strings)
  - [Localization](#localization)
- [Actions](#actions)
- [Commands](#commands)
- [Configurations](#configurations)
- [Concerns](#concerns)
- [Contracts](#contracts)
- [Controllers](#controllers)
- [Enums](#enums)
- [Factories](#factories)
- [Jobs](#jobs)
- [Middleware](#middleware)
- [Migrations](#migrations)
- [Models](#models)
- [Policies](#policies)
- [Routes](#routes)
- [Seeders](#seeders)
- [Services](#services)
- [Tests](#tests)
- [Traits](#traits)
- [Validation](#validation)
- [Views](#views)

## Core

### Follow PHP conventions

- Use `composer.json` to determine the used version of PHP.
- Follow PSR-1 and PSR-12 recommendations.

### Follow Laravel conventions

- Always follow Laravel conventions first. If the Laravel documentation documents something in a certain way, use it.
- Only deviate if there is a specific convention mentioned in this skill's guidelines.

### Formatting

- Let code breathe. Avoid cramped formatting.
- Add blank lines between statements.
- Place chained method calls on new lines instead of keeping them inline.
    ```php
    // Do this
    User::query()
        ->where('active', true)
        ->orderBy('name')
        ->get();

    // Not this
    User::query()->where('active', true)->orderBy('name')->get();
    ```

### File names

- Use Laravel conventions for file names.
- Add suffixes to file names when they communicate the artifact type clearly. ( e.g. `UserController.php`, `UserService.php`, `UserRepository.php` )

### Imports

- Always import class names for code and comments instead of using FQCNs inline.

### Typing

- Use short nullable notation `?User $user`. Not `User|null $user`.
- Always use typed properties over docblocks.
- Always use return types over docblocks.
- Always document iterable types with docblocks. like `/** @var Collection<int, User> $users */`.
- Always document key and value types for iterable types.
- Always order a multi-type declaration with the most common type first.
- Use constructor property promotion when all properties can be promoted

### Comments / Docblocks

- Never write docblocks when the code is self-explanatory.
- Always document iterable types with docblocks. like `/** @var Collection<int, User> $users */`.
- Always document key and value types for iterable types.
- Use array shape notation for fixed keys
    ```php
    /**
     * @return array{
     *     name: string,
     *     age: int,
     * }
     */
    ```

### Control flow

- Always place the happy path last. First handle error / false cases and then handle the happy path.
- Prefer early return statements.
- Prefer separate readable conditions over one-liners.
- Always use curly braces for control flow.
- Always write ternary operators on multiple lines.
    ```php
    $status = $user->isAdmin()
        ? 'admin'
        : 'user';
    ```

### Strings

- Use string interpolation instead of concatenation.

### Localization

- Use `__()` for translating strings.
- Use `trans_choice()` for pluralizing strings.
- Always use dot notation for translating strings.
- Structure translation files by scope. ( e.g. `users.php`, `roles.php`, `permissions.php`)

## Actions

### General

- Use the Action pattern by default for business logic that can be reused, tested independently, or shared between controllers, commands, jobs, and services.
- Use the action to perform a single business logic operation.
- Use actions from controllers, commands, services, and jobs when they need shared business logic.

### Examples

```php
final readonly class CreateUserAction
{
    public function execute(array $data): User
    {
        return User::query()->create($data);
    }
}
```

### Naming conventions

- Use PascalCase for class names
- Suffix action names with `Action`
- Place actions in the `App\Actions` namespace.
- Group actions by scope in subfolders when it improves discoverability. ( e.g. `App\Actions\Users\CreateUserAction.php`, `App\Actions\Users\DeleteUserAction.php` )

## Commands

### General

- Keep commands focused on orchestration and console interaction.
- Delegate business logic to actions.

### Examples

```php
final class SyncUsersCommand extends Command
{
    protected $signature = 'users:sync';

    public function handle(SyncUsersAction $syncUsersAction): int
    {
        $syncUsersAction->execute();

        return self::SUCCESS;
    }
}
```

### Naming conventions

- Always use the `Command` suffix for class names.

## Configurations

### General

- Only use `env()` in config files.
- Always use the `Config` facade and its typed methods to access the config.

### Examples

```php
// config/services.php
return [
    'postmark' => [
        'token' => env('POSTMARK_TOKEN'),
    ],
];

// App\Services\PostmarkService.php
$token = Config::string('services.postmark.token');
```

### Naming conventions

- Strictly use `kebab-case` for file names.
- Use `snake_case` for config keys.
- Place config files in the `config` folder.

## Concerns

### General

- Use `Concerns` naming instead of `Traits`.
- Place concern traits in the `App\Concerns` namespace.

### Examples

```php
trait InteractsWithSeoData
{
    public function getSeoTitle(): string
    {
        return $this->seo_title ?? $this->title;
    }
}
```

### Naming conventions

- Use PascalCase for class names.
- Place concerns in the `App\Concerns` namespace.
- Group concerns by scope in subfolders when it improves discoverability. ( e.g. `App\Concerns\Models`, `App\Concerns\Filament\Forms` )
- Prefer names with a verb and noun. ( e.g. `InteractsWithSeoData`, `SerializesModels` )
- Choose names that do not clash with contracts or interfaces. Some examples:
    - `App\Concerns\InteractsWithSeoData`
    - `App\Concerns\SerializesModels`

## Contracts

### General

- Only add `interface` classes to the `App\Contracts` namespace.

### Examples

```php
interface HasSeoData
{
    public function getSeoTitle(): string;
}
```

### Naming conventions

- Use PascalCase for class names.
- Place contracts in the `App\Contracts` namespace.
- Group contracts by scope in subfolders when it improves discoverability. ( e.g. `App\Contracts\Models`, `App\Contracts\Filament\Forms` )
- Prefer names starting with `Has` when the contract expresses a capability. ( e.g. `HasSeoData`, `HasModelSerialization` )
- Choose names that do not clash with concerns or traits. Some examples:
    - `App\Contracts\HasSeoData`
    - `App\Contracts\HasModelSerialization`

## Controllers

### General

- Keep controllers as thin as possible by using `Requests`, `Actions`, and `Resources`.
- Delegate business logic to actions by default.
- Use `Concerns` to avoid code duplication if absolutely necessary, but prefer `Actions` over `Concerns` to keep the code more maintainable and testable.
- Use `Resources` to transform the data before sending it to the client when working with APIs.
- Strictly keep controllers to the resource methods. ( `index`, `create`, `store`, `show`, `edit`, `update`, `destroy` ).
- Use invokable controllers for other actions that do not fit in the resource methods. (e.g. `PublishPostController`, `UploadAvatarController` )

### Examples

```php
final class PostsController extends Controller
{
    public function store(StorePostRequest $request, CreatePostAction $createPostAction): PostResource
    {
        $post = $createPostAction->execute($request->validated());

        return new PostResource($post);
    }
}
```

### Naming conventions

- Always use `Controller` suffix
- Place app controllers in `app/Http/Controllers`
- Place API controllers in `app/Http/Controllers/Api` and versioned controllers in `app/Http/Controllers/Api/V1`
- Use plural names for controllers that handle resource methods. (e.g. `PostsController`, `UsersController` )
- Use singular names for invokable controllers. (e.g. `PublishPostController`, `UploadAvatarController` )

## Enums

### General

- Place enum cases in alphabetical order.

### Examples

```php
enum PostStatus: string
{
    case Archived = 'archived';
    case Draft = 'draft';
    case Published = 'published';
}
```

### Naming conventions

- Use `PascalCase` for class names.
- Use `PascalCase` for enum cases.
- Place enums in the `App\Enums` namespace.
- Place enums used for `Filament` in the `App\Enums\Filament` namespace.
- Use the `get` suffix for enum methods. ( e.g. `getLabel()`, `getColor()`, `getIcon()`, `getDescription()` )

## Factories

### General

- Always use the `fake()` method instead of `$this->faker`.
- Always add all possible `state` methods that fit the model. (e.g. `active()`, `inactive()`, `archived()`, `published()`, `unpublished()` )
- Use the factory of the corresponding model for belongsTo columns
- Place chained factory modifiers on new lines instead of keeping them inline.

### Examples

```php
public function definition(): array
{
    return [
        'name' => fake()->name(),
        'email' => fake()->unique()->safeEmail(),
    ];
}

User::factory()
    ->count(10)
    ->create();
```

### Naming conventions

- Always use the `{Model}Factory` naming convention.

## Jobs

### General

- Keep jobs focused on queue concerns and orchestration.
- Delegate business logic to actions.
- Jobs should almost never have more than 1 tries configured.
- Always use model keys instead of full models as parameters to keep the payload small and avoid serialization issues.

### Examples

```php
final readonly class SendWelcomeEmailJob implements ShouldQueue
{
    public function __construct(public int $userId) {}

    public function handle(SendWelcomeEmailAction $sendWelcomeEmailAction): void
    {
        $sendWelcomeEmailAction->execute($this->userId);
    }
}
```

### Naming conventions

- Always use the `Job` suffix for class names.

## Middleware

### General

- Always place happy path last.

### Examples

```php
public function handle(Request $request, Closure $next): Response
{
    if (! $request->user()?->is_active) {
        abort(403);
    }

    return $next($request);
}
```

### Naming conventions

- Always use the `Middleware` suffix for middleware classes.
- All middleware classes should live in the `App\Http\Middleware` namespace.
- Middleware for usage in `Filament` should live in the `App\Http\Middleware\Filament` namespace.

## Migrations

### General

- Never write `down` migrations.
- When creating or modifying a migration, remove the `down()` method if it is present.
- Place every chained modifier on a new line in migrations. Do not keep modifiers inline on the same line as the column definition or with each other.

### Examples

```php
// Do this
$table->string('name')
    ->nullable()
    ->default('John Doe');

$table->boolean('completed')
    ->default(false)
    ->index();

// Not this
$table->string('name')->nullable()->default('John Doe');
$table->boolean('completed')->default(false)->index();
```

### Naming conventions

- Follow Laravel naming conventions.
- Use the file name to describe the migration. ( e.g. `2024_01_01_000000_create_users_table.php`, `2024_01_01_000001_add_email_field_to_users_table.php`, `2024_01_01_000002_remove_email_field_from_users_table.php` )

## Models

### General

- Make sure the application unguards models using `Model::unguard()`.
- Do not define `$fillable` on models.
- Use the `#[UseFactory]` attribute on models instead of the `HasFactory` trait.
- Always use the `casts` function to define casts and add an array shape docblock to help IDEs and PHPStan understand the type.
- Always use `Attribute` return types for accessors and mutators.
- Always use `#[Scope]` attributes for scopes and do not prefix them with `scope`.
- Keep business logic in models to a minimum. Prefer actions or dedicated classes for larger operations.
- Always use `Enums` for model attributes with a limited set of values.
- Always start Eloquent model queries with `Model::query()` instead of calling query builder methods directly on the model.

### Examples

```php
#[UseFactory(UserFactory::class)]
final class User extends Model
{
    /**
     * @return array{
     *     completed: 'boolean',
     * }
     */
    protected function casts(): array
    {
        return [
            'completed' => 'boolean',
        ];
    }
}

User::query()
    ->where('active', true)
    ->first();

// Not this
User::where('active', true)->first();
```

### Naming conventions

- Models should live in the `app/Models` namespace.
- Pivot models should live in the `app/Models/Pivots` namespace.

## Policies

### General

- Always add the following methods:
    - `viewAny()`
    - `view()`
    - `create()`
    - `update()`
    - `deleteAny()`
    - `delete()`
- Always use `Authenticatable $authenticatable` instead of `User $user` as the first parameter of the methods so that we can support multiple auth models.

### Examples

```php
final class PostPolicy
{
    public function viewAny(Authenticatable $authenticatable): bool
    {
        if (! $authenticatable instanceof User) {
            return false;
        }

        return $authenticatable->can('viewAny posts');
    }
}
```

### Naming conventions

- Always use `{Model}Policy` as the naming convention.
- All policies should live in the `App\Policies` namespace.

## Routes

### General

- Never use the `Route::resource` method.
- Always place the routes in alphabetical order.
- Group routes by middleware and resource.
- Place chained route modifiers on new lines instead of keeping them inline.

### Examples

```php
Route::get('posts', [PostsController::class, 'index'])
    ->name('posts.index');

Route::post('posts/{post}/publish', PublishPostController::class)
    ->name('posts.publish');
```

### Naming conventions

- Use `snake_case` for route names.
- Use `kebab-case` for route segments.
- Use `camelCase` for route parameters.
- Always use tuple notation (`[Controller::class, 'method']`) for non-invokable controllers.
- Use `::class` for invokable controllers.
- Use plural names for controllers that handle resource methods. (e.g. `PostsController`, `UsersController` )
- Use singular names for invokable controllers. (e.g. `PublishPostController`, `UploadAvatarController` )

## Seeders

### General

- Always use the provided factories
- Always create a seeder per model
- Always call that seeder from the `DatabaseSeeder` class
- Place chained factory modifiers on new lines instead of keeping them inline.

### Examples

```php
final class UserSeeder extends Seeder
{
    public function run(): void
    {
        User::factory()
            ->count(10)
            ->create();
    }
}
```

### Naming conventions

- Always use the `{Model}Seeder` naming convention.

## Services

### General

- Prefer actions for single business operations.
- Use services for integrations, shared infrastructure, or long-lived abstractions that do not fit a single action cleanly.
- Add a `Facade` when it improves ergonomics for a widely used service.
- Always use a singleton pattern when possible. Using the `#[Singleton]` attribute is recommended.

### Examples

```php
#[Singleton]
final class StripeService
{
    public function createCheckoutSession(array $data): array
    {
        return [];
    }
}
```

### Naming conventions

- Always use `Service` suffix so that the Facade can be the class name without the suffix.

## Tests

### General

- Always use `pest` as the testing framework.
- Keep test classes in the same file when possible.
- Follow the arrange-act-assert pattern.

### Examples

```php
it('creates a post', function () {
    // Arrange, act, assert.
});
```

### Naming conventions

- Always mimic the app folder structure in feature and unit tests.

## Traits

### General

- We use `Concerns` instead of `Traits` naming.

## Validation

### General

- Always use `Request` classes for validation. Even when using Livewire.
- Use policies or gates in the `authorize` method when authorization is required.
- Place each validation rule on its own line inside the rules array.

### Examples

```php
final class StoreUserRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'name' => [
                'required',
                'string',
            ],
        ];
    }
}
```

### Naming conventions

- Always use the `Request` suffix for class names.
- Requests should live in the `App\Http\Requests` namespace.
- Name requests after the action they validate. (e.g. `StoreUserRequest`, `UpdateUserSettingsRequest`)

## Views

### General

- Make components reusable when the same UI pattern appears in multiple contexts.

### Examples

```blade
<x-layouts.app>
    <x-users.table :users="$users" />
</x-layouts.app>
```

### Naming conventions

- Use `kebab-case` for view file names.
- Place views in the `resources/views` folder.
