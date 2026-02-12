# GitHub Copilot Instructions

## Overview

This repository (`pied-piper-api`) is a Node.js REST API built with TypeScript and Express. It provides backend services for the Pied Piper platform, including user management, order processing, and reporting endpoints. The API uses PostgreSQL for persistence and JWT-based authentication.

## Code Style

- TypeScript strict mode is enabled. Avoid `any` types.
- Use camelCase for variables and functions, PascalCase for classes and interfaces, UPPER_SNAKE_CASE for constants.
- Files are named in kebab-case (e.g., `user-controller.ts`, `order-service.ts`).
- Prefer `async/await` over `.then()` chains. All async functions must have error handling.

## Architecture

The project follows a layered architecture:

- `src/controllers/` - HTTP request handling, input validation, response formatting.
- `src/services/` - Business logic. Controllers call services, never the database directly.
- `src/repositories/` - Database access. Services call repositories for data operations.
- `src/middleware/` - Express middleware for auth, validation, error handling, and logging.
- `src/config/` - Centralized configuration. All environment variables are accessed through this module.
- `src/types/` - Shared TypeScript interfaces and types.

## Build and Test

```bash
npm install          # Install dependencies
npm run build        # Compile TypeScript
npm run test         # Run unit and integration tests
npm run lint         # Run ESLint
npm run dev          # Start development server with hot reload
```

## Project Conventions

- Environment-specific configuration is accessed through `src/config/index.ts`. Do not use `process.env` directly in business logic.
- All route handlers use the centralized error middleware. Individual handlers should throw typed errors, not send responses directly for error cases.
- Database queries use parameterized statements exclusively. String concatenation in queries is a critical violation.
- API responses follow a consistent shape: `{ "data": ... }` for success, `{ "error": { "code": "...", "message": "..." } }` for errors.

## Security

- Secrets and API keys come from environment variables, never hardcoded.
- JWT authentication is handled by the auth middleware in `src/middleware/auth.ts`. Individual routes must not implement custom token parsing.
- CORS is configured explicitly in `src/config/cors.ts`. Wildcard origins are not allowed in production.
- All user inputs are validated at the controller layer before reaching services.

## Azure DevOps Integration

This project uses Azure DevOps (organization: YourOrg, project: YourProject, repository: pied-piper-api).

- Repository: https://dev.azure.com/YourOrg/YourProject/_git/pied-piper-api
- Active Pull Requests: https://dev.azure.com/YourOrg/YourProject/_git/pied-piper-api/pullrequests?_a=active
- Individual PR link format: https://dev.azure.com/YourOrg/YourProject/_git/pied-piper-api/pullrequest/{prId}

## PR Review Agent

A custom PR Review agent is available for reviewing pull requests. Engineers can switch to the "PR Review" agent in the chat dropdown or type `/prReview` to run a full review of a PR against `main`. The engineer must provide a PR number or link, the agent does not use the local branch. It fetches the remote diff via the ADO MCP server, reviews against team standards, and provides a summary in chat. It then asks for explicit confirmation before posting inline comments on the PR in Azure DevOps. It will never spam comments without the engineer approving first. See [agents/pr-review.agent.md](agents/pr-review.agent.md) for the agent definition and [instructions/code-review.instructions.md](instructions/code-review.instructions.md) for the review standards.

## Guidelines

* Be concise in your responses, informative and to the point, avoiding unnecessary elaboration.
* When providing code, ensure it is relevant to the context of the repository and follows best practices for readability and maintainability.
* Never use emojis or em or en dashes in any of your suggestions.
