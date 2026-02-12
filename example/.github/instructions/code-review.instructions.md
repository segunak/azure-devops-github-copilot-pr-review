---
name: Code Review Standards
description: Review standards and criteria for the Pied Piper API. Referenced by the PR Review agent.
---

# Code Review Standards

These are the review standards for the Pied Piper API. Apply these criteria when reviewing pull requests.

## TypeScript

- Use strict mode (`"strict": true` in tsconfig.json). Do not use `any` unless absolutely necessary and justified with a comment.
- Prefer `interface` over `type` for object shapes that may be extended. Use `type` for unions, intersections, and mapped types.
- Use `async/await` over raw Promises. Never mix `.then()` chains with `await` in the same function.
- All functions should have explicit return types. Do not rely on type inference for public APIs.
- Use `readonly` for properties and parameters that should not be mutated.
- Prefer `unknown` over `any` for values of uncertain type. Narrow with type guards before use.

## Node.js and Express

- All route handlers must have error handling. Use `try/catch` in async handlers or a centralized error middleware.
- Never swallow errors silently. Log them with sufficient context (request ID, user ID, operation name) before returning an error response.
- Use environment variables for configuration (port, database URLs, feature flags). Access them through the config module (`src/config/`), not directly via `process.env` in business logic.
- Validate all request inputs at the controller layer using the validation middleware. Do not trust client data in service or repository layers.
- Use proper HTTP status codes: 200 for success, 201 for creation, 400 for bad input, 401 for unauthenticated, 403 for unauthorized, 404 for not found, 500 for server errors. Do not return 200 with an error body.

## API Design

- Follow RESTful conventions: plural nouns for resource endpoints (`/users`, `/orders`), HTTP verbs for actions.
- Return consistent error response shapes: `{ "error": { "code": "string", "message": "string" } }`.
- Paginated endpoints must use `limit` and `offset` query parameters with sensible defaults.
- Breaking changes to API contracts require a version bump or a new endpoint version.

## Database

- All database queries must use parameterized statements. Never concatenate user input into query strings.
- Database migrations must be backward-compatible. A migration that breaks the previous version of the application is a critical issue.
- Use transactions for operations that modify multiple tables. Partial writes are data integrity bugs.

## Testing

- New endpoints must have integration tests covering the happy path and at least one error case.
- Business logic in service classes should have unit tests.
- Tests must not depend on external services. Use mocks or test doubles for database and API dependencies.
- Test file naming convention: `*.test.ts` colocated with the source file or in a `__tests__/` directory.

## Security

- No secrets, API keys, connection strings, or tokens hardcoded anywhere in the codebase.
- Sensitive configuration must come from environment variables or a secrets manager, never checked into source control.
- All authenticated endpoints must validate the JWT token via the auth middleware. Do not implement custom token parsing in individual routes.
- Validate and sanitize all user inputs. Be especially careful with inputs used in database queries, file paths, or shell commands.
- CORS configuration must be explicit. Do not use `origin: '*'` in production.

## Documentation

- Changes to API endpoints must be reflected in the OpenAPI spec or API documentation.
- New modules or services should have a brief JSDoc comment explaining their purpose.
- Changes to environment variables or configuration must update the README or relevant config documentation.

## General Code Quality

- Prefer readability over cleverness. Code is read far more often than it is written.
- Remove dead code rather than commenting it out. Git has history.
- Variable and function names should be descriptive and use camelCase. Class names use PascalCase. Constants use UPPER_SNAKE_CASE.
- Avoid duplicating logic that already exists elsewhere in the codebase. Extract shared logic into utility functions or services.
- Keep functions focused on a single responsibility. If a function does too many things, split it.
- No emojis or em/en dashes in code, comments, or documentation.
