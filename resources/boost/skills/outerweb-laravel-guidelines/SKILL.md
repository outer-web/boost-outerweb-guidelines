---
name: outerweb-laravel-guidelines
description: Apply Outerweb's Laravel and PHP guidelines when working with Laravel. Use this skill when creating, editing, reviewing and/or formatting Laravel/PHP code. This is important to keep the codebase consistent and easier to maintain.
metadata:
  author: Outerweb
---

# Outerweb Laravel Guidelines

## When to use this skill

- Use this skill when creating, editing, reviewing and/or formatting Laravel/PHP code.
- Assume that, when installed, the developer wants to follow the Outerweb Laravel guidelines. Even when they did not specifically ask for it.

## Scope

- Any `.php` file in the codebase, including but not limited to:
  - Models
  - Views
  - Controllers
  - Actions
  - Middleware
  - Services
  - Contracts
  - Concerns
  - Enums
  - Factories
  - Migrations
  - Seeders
  - Routes
  - Configurations
  - Tests
- Any `.blade.php` file in the codebase

## Critical rules

- Use the Action pattern by default for business logic in Laravel applications.
- Keep controllers, commands, and jobs thin by delegating business logic to actions.
- Only skip creating an action when the code is trivial glue code with no meaningful business logic or reuse.
- When building features, assume the business operations should be implemented as actions unless the request clearly calls for another pattern.

## Workflow

1. Identify the artifact (e.g. model, controller, view, etc.) and its purpose. 
2. Read and follow `references/GUIDELINES.md`.
3. Apply the relevant sections for the artifact type you are creating or editing.

## References

- `references/GUIDELINES.md`
