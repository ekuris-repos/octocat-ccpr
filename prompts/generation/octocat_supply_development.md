---
title: "OctoCAT Supply Chain Application Development"
category: "generation"
subcategory: "full_stack_application_development"
author: "Erik Kuris"
created_date: "2026-03-20"
last_modified: "2026-03-20"
version: "1.0.0"
status: "draft"
compliance_frameworks: []
tags: ["typescript", "react", "express", "sqlite", "supply-chain", "copilot-demo", "monorepo"]
difficulty_level: "intermediate"
estimated_time: "5-15 minutes"
confidence_score: 0.90
dependencies: ["Node.js 18+", "npm", "Make"]
related_prompts: ["generate_code.md", "generate_summary.md"]
---

# OctoCAT Supply Chain Application Development

## Purpose

Guide AI-assisted development of the OctoCAT Supply Chain Management application, a TypeScript monorepo consisting of an Express REST API with SQLite persistence and a React + Vite + Tailwind frontend. This prompt provides context, conventions, and instructions so that code generation stays consistent with the project's established architecture and patterns.

### Primary Use Cases
- Generating new API endpoints (routes, repositories, models) following the repository pattern
- Creating or modifying React components with Tailwind CSS styling
- Writing SQLite migration and seed scripts
- Extending Swagger/OpenAPI documentation
- Writing unit tests for repository logic and component tests for UI

### Expected Outcomes
- Code that follows existing conventions (camelCase models, snake_case DB columns, repository pattern)
- Proper error handling using custom error types (NotFound, Validation, Conflict)
- Type-safe TypeScript with no untyped `any` unless justified
- Consistent REST API patterns with correct HTTP status codes

### Target Audience
- **Solution Engineers**: Preparing and running Copilot demos with this application
- **Developers**: Extending the application with new features or entities
- **Reviewers**: Validating generated code against project standards

## Prompt Text

```
You are working on the OctoCAT Supply Chain Management application, a TypeScript monorepo.

**Project Structure:**
- api/          Express REST API (SQLite, repository pattern, Swagger docs)
- frontend/     React 18+ with Vite and Tailwind CSS
- docs/         Architecture and integration documentation
- infra/        Azure Container Apps Bicep templates

**Backend Conventions:**
- Entity models in api/src/models/ use camelCase properties
- Database columns use snake_case naming
- Repositories in api/src/repositories/ handle all data access via raw SQL
- Routes in api/src/routes/ wire Express handlers to repository methods
- Swagger annotations are inline with route definitions
- Database file: api/data/app.db (configurable via DB_FILE env var)
- Migrations: api/database/migrations/ (SQL files, executed in order)
- Seed data: api/database/seed/ (SQL files, ordered execution)

**Frontend Conventions:**
- Components live in frontend/src/ with TypeScript (.tsx)
- Styling uses Tailwind CSS utility classes
- Vite handles bundling and dev server (port 5173)
- API proxy configured to route /api requests to the backend

**Database Entities:**
Headquarters, Branch, Order, OrderDetail, Product, Supplier, Delivery, OrderDetailDelivery

**Error Handling:**
Use custom error types (NotFound, Validation, Conflict) that propagate consistent
HTTP status codes through Express middleware.

**Testing:**
- API tests use Vitest with in-memory SQLite (:memory:)
- Frontend tests use Vitest with React Testing Library
- E2E tests use Playwright

When generating code for this project:
1. Follow existing file naming and structure patterns
2. Use TypeScript with strict typing (no untyped any)
3. Include Swagger/OpenAPI annotations for new API routes
4. Write repository methods with parameterized SQL queries
5. Map between camelCase model properties and snake_case DB columns
6. Add unit tests for new repository logic
7. Use Tailwind utility classes for frontend styling
8. Prefer environment variable configuration over hard-coded values
```

## Example Usage

### Input:
```
Create a new API endpoint for managing warehouse locations. Each warehouse has an id,
name, address, capacity, and is linked to a branch.
```

### Expected Output:

A set of files following the established pattern:
- `api/src/models/warehouse.ts` with a TypeScript interface
- `api/src/repositories/warehouseRepository.ts` with CRUD SQL operations
- `api/src/routes/warehouseRoutes.ts` with Express route handlers and Swagger docs
- `api/database/migrations/XXX_create_warehouses.sql` with the CREATE TABLE statement
- `api/src/repositories/__tests__/warehouseRepository.test.ts` with Vitest tests

### Example 2: Frontend Component
**Input:**
```
Add a warehouse list page to the frontend that displays all warehouses in a table
with columns for name, address, capacity, and branch name.
```

**Expected Output:**
- A React component using Tailwind CSS table styling
- Fetch call to the `/api/warehouses` endpoint
- Loading and error states
- TypeScript interface for the warehouse data shape

## Variables

| Variable | Description | Required | Default |
|----------|-------------|----------|---------|
| `{ENTITY_NAME}` | Name of the entity being created or modified | Yes | N/A |
| `{DB_FILE}` | Path to the SQLite database file | No | `api/data/app.db` |
| `{API_PORT}` | Port for the Express API server | No | `3000` |
| `{FRONTEND_PORT}` | Port for the Vite dev server | No | `5173` |
| `{DB_ENABLE_WAL}` | Enable Write-Ahead Logging for SQLite | No | `true` |
| `{DB_FOREIGN_KEYS}` | Enforce foreign key constraints | No | `true` |

## Guidelines

### Best Practices
1. **Repository Pattern**: All database access goes through repository classes. Never write SQL directly in route handlers.
2. **Parameterized Queries**: Always use parameterized SQL to prevent injection. The repository layer handles this via `better-sqlite3` prepared statements.
3. **Minimal Diffs**: Prefer incremental changes that preserve existing style and naming rather than large rewrites.
4. **Type Safety**: Define TypeScript interfaces for all models and DTOs. Avoid `any` unless there is a documented justification.

### Quality Criteria
- **Correctness**: Generated SQL must respect foreign key relationships defined in the ERD
- **Consistency**: New files must follow the naming pattern of existing files in the same directory
- **Completeness**: New endpoints need a model, repository, route, migration, and test at minimum

### Common Pitfalls
- **N+1 Queries**: When loading related entities, use JOINs in the repository rather than multiple sequential queries
- **Missing Migrations**: Every schema change requires a new migration file; never modify existing migration files
- **Hard-coded Config**: Use environment variables (DB_FILE, ports) instead of hard-coded paths or values
- **Ignoring Seed Order**: Seed scripts must respect foreign key dependencies. Parent tables are seeded before child tables.

### Performance Optimization
- SQLite WAL mode is enabled by default for concurrent read performance
- Add indexes to columns used in WHERE clauses and JOIN conditions
- Use `SELECT` with specific columns rather than `SELECT *` in production queries

## Related Prompts

- [Generate Code](generate_code.md) for general code generation patterns
- [Generate Summary](generate_summary.md) for summarizing changes and documentation

## Compliance Notes

- This is a demo/showcase application. No PII or sensitive customer data is stored.
- API endpoints do not currently implement authentication or authorization. For production use, add appropriate auth middleware.
- SQLite database files should not be committed to version control. The `.gitignore` excludes `api/data/*.db`.
- Environment variables should be used for all configuration. Never commit secrets or API keys.
