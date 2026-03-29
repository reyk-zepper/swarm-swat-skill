---
name: Backend API
triggers:
  keywords: [API, endpoint, route, REST, GraphQL, database, schema, migration, query, ORM, Prisma, Drizzle, controller, service, middleware, CRUD, webhook, server action]
  file_patterns: ["**/api/**", "**/routes/**", "**/server/**", "**/db/**", "**/schema/**", "**/models/**", "**/services/**", "**/prisma/**", "**/drizzle/**"]
  task_patterns: ["build.*api", "create.*endpoint", "implement.*backend", "add.*route", "database.*schema", "server.*action"]
quality_mode: standard
max_workers: 4
---

# Backend API SWAT Team

Deployed when the task involves building or modifying backend API endpoints, database schemas, or server-side logic.

## Workflow

```
Schema-Designer ──────┐
                      ├──→ Test-Writer ──→ Integration-Tester
Endpoint-Builder ─────┘
```

Schema-Designer and Endpoint-Builder run in parallel. Test-Writer starts only after both complete. Integration-Tester starts only after Test-Writer completes.

---

## Role 1: Schema-Designer

```yaml
model: sonnet
subagent_type: general-purpose
mode: bypassPermissions
```

### Prompt

```
## Objective

Design and implement the database schema changes required for the following task:

{task_context}

## Your Responsibilities

- Identify which entities and relations need to be created or modified
- Write the complete schema definition using the ORM or SQL dialect already present in this project (Prisma, Drizzle, raw SQL — match what exists)
- Create reversible migrations if the project uses a migration system
- Define all necessary indices, constraints, and relations
- Document the entity-relationship structure as ASCII art

## How to Proceed

1. Read the existing schema files to understand current conventions, naming style, and ORM version
2. Identify what needs to change: new tables/models, new columns, changed relations
3. Apply these principles:
   - Stick to the ORM and migration tooling already in use — do not introduce new tools
   - Default to 3rd Normal Form (3NF); only denormalize if you can state a concrete performance reason
   - Add indices for every foreign key column and every column that appears in WHERE or ORDER BY clauses in expected queries
   - Add DB-level constraints (NOT NULL, UNIQUE, CHECK) wherever business logic requires them — do not rely solely on application-level validation
   - Follow the naming conventions already established in the existing schema (snake_case vs camelCase, singular vs plural table names, etc.)
   - Write migrations that are reversible (up + down); if a migration cannot be reversed, state why explicitly
   - Only introduce soft deletes (deleted_at column) if the pattern already exists in the project

## Expected Output

For each entity created or modified, provide:

1. **Schema code** — complete, copy-pasteable schema definition (Prisma model, Drizzle table, SQL CREATE TABLE, etc.)
2. **Migration file** — if the project uses migrations, the complete migration code (up and down)
3. **Entity-Relationship diagram** — ASCII representation showing entities, their fields, and relations
4. **Index rationale** — list every index you added with a one-line reason per index
5. **Constraints** — list all DB-level constraints (NOT NULL, UNIQUE, CHECK, FK) and why each is needed

## Quality Criteria

- Schema compiles without errors against the existing ORM version
- Every foreign key has an index
- No nullable column that the application will always require
- Migrations are reversible
- Naming is consistent with the existing schema
```

### Output Contract

| Artifact | Format | Required |
|----------|--------|----------|
| Schema code | ORM-native syntax (Prisma/Drizzle/SQL) | Yes |
| Migration files | up + down, ORM-native | If project uses migrations |
| ER diagram | ASCII | Yes |
| Index list | `column_name — reason` per line | Yes |
| Constraint list | `constraint_type column — reason` per line | Yes |

---

## Role 2: Endpoint-Builder

```yaml
model: sonnet
subagent_type: general-purpose
mode: bypassPermissions
isolation: worktree
```

### Prompt

```
## Objective

Implement the API endpoints required for the following task:

{task_context}

## Your Responsibilities

- Identify which HTTP endpoints (or Server Actions) need to be created or modified
- Implement each endpoint completely: routing, validation, business logic delegation, response formatting, error handling
- Define TypeScript types for all inputs and outputs
- Use the existing framework and patterns already in the codebase

## How to Proceed

1. Read existing endpoint/route files to understand the framework in use (Next.js Route Handlers, Express, Hono, tRPC, etc.) and current conventions
2. Read existing service files, middleware, and auth helpers to understand how to reuse them
3. For each endpoint, implement the following layers:
   - **Route definition** — correct HTTP method and path, registered in the right place
   - **Input validation** — validate and parse all inputs at the system boundary using the existing validation library (Zod, Yup, Valibot, etc.); never trust raw request data
   - **Auth check** — apply the existing auth middleware or guard; every endpoint must explicitly handle the unauthenticated and unauthorized cases
   - **Business logic** — delegate to a service function, do not put business logic directly in the endpoint handler
   - **Response** — return typed responses with correct HTTP status codes
   - **Error handling** — catch and convert errors into consistent error responses using the project's established error format

4. Apply these rules:
   - Use the framework already in the project — do not introduce a new router or server library
   - All TypeScript types must be explicit; no `any`, no untyped `request.body`
   - HTTP semantics: GET for reads, POST for creates, PUT/PATCH for updates, DELETE for deletes; 200 for success, 201 for created, 204 for no-content deletes, 400 for validation errors, 401 for unauthenticated, 403 for unauthorized, 404 for not found, 409 for conflicts, 500 for unexpected errors
   - Business logic belongs in service functions, not endpoint handlers; create or extend service files as needed
   - Reuse existing middleware for auth, rate limiting, logging — do not duplicate it

## Expected Output

For each endpoint, provide:

1. **Route summary** — `METHOD /path/to/endpoint` in one line
2. **File path** — where the endpoint is implemented
3. **Auth requirement** — which auth level is required (public, authenticated, specific role/permission)
4. **Input types** — TypeScript types for request body, query params, path params
5. **Output types** — TypeScript type for the success response body
6. **Error codes** — table of possible HTTP error codes with description for each
7. **Complete implementation** — the full, copy-pasteable code

## Quality Criteria

- Every endpoint validates all inputs before use
- Every endpoint has an explicit auth check
- No `any` types in input or output interfaces
- Error responses follow a consistent format matching the project's existing pattern
- Business logic is in service functions, not inline in route handlers
- HTTP methods and status codes match REST semantics
```

### Output Contract

| Artifact | Per Endpoint | Required |
|----------|-------------|----------|
| Route summary | `METHOD /path` | Yes |
| File path | absolute path | Yes |
| Auth requirement | public / authenticated / role:X | Yes |
| Input types | TypeScript interface/type | Yes |
| Output types | TypeScript interface/type | Yes |
| Error codes | table: code — description | Yes |
| Full implementation | complete file content | Yes |

---

## Role 3: Test-Writer

```yaml
model: sonnet
subagent_type: general-purpose
mode: bypassPermissions
isolation: worktree
consumes: [Schema-Designer.output, Endpoint-Builder.output]
```

### Prompt

```
## Objective

Write comprehensive tests for the API endpoints produced in this task:

{task_context}

## Schema Output (from Schema-Designer)

{Schema-Designer.output}

## Endpoint Output (from Endpoint-Builder)

{Endpoint-Builder.output}

## Your Responsibilities

- Write test files for every endpoint the Endpoint-Builder produced
- Cover the full test matrix per endpoint (see below)
- Use the test framework already present in the project
- Structure tests for a real test database, not mocked DB calls

## How to Proceed

1. Read existing test files to understand the test framework (Jest, Vitest, Playwright, supertest, etc.), test utilities, and database setup/teardown patterns
2. Read the endpoint implementations from Endpoint-Builder.output to understand exact routes, input shapes, and expected responses
3. Read the schema from Schema-Designer.output to understand the data model
4. For each endpoint write tests covering:

   **Happy Path**
   - 200/201 response with valid input and authorized user
   - Correct response body shape matching the endpoint's output type
   - DB state is correct after the operation (for mutations)

   **Auth Failures**
   - 401 when no auth token / session is provided
   - 403 when authenticated but not authorized (wrong role, wrong ownership)

   **Validation Errors**
   - 400 when required fields are missing
   - 400 when field types are wrong
   - 400 when values violate business constraints (e.g., negative quantity, future date where past required)

   **Not Found**
   - 404 when the requested resource does not exist
   - 404 when the resource exists but belongs to a different user (ownership check)

   **Edge Cases**
   - Empty collections for list endpoints (return [] not 404)
   - Boundary values (max string length, 0, -1, very large numbers)
   - Unicode and special characters in string inputs
   - Duplicate/idempotency scenarios for POST endpoints

5. Apply these rules:
   - Use Arrange-Act-Assert structure for every test
   - Use descriptive test names: `it('returns 401 when request has no auth token')`
   - Prefer a real test database over DB mocks; use transactions or truncation for cleanup
   - Each test must clean up after itself — no test should depend on state from a previous test
   - Do not test framework internals — test your endpoint's behavior, not Express or Next.js

## Expected Output

For each endpoint, provide:

1. **Test file path** — where the test file should be created
2. **Coverage description** — one-line list of all test cases in this file
3. **Complete test code** — full, copy-pasteable test file content

## Quality Criteria

- Every endpoint has a test file
- Every test file covers: happy path, 401, 403, 400 validation, 404, at least one edge case
- Tests use Arrange-Act-Assert
- Tests are independent (no shared mutable state between tests)
- Test names describe the scenario, not the implementation
```

### Output Contract

| Artifact | Per Endpoint | Required |
|----------|-------------|----------|
| Test file path | absolute path | Yes |
| Coverage description | list of test case names | Yes |
| Complete test code | full file content | Yes |

Minimum test coverage per endpoint:

| Scenario | Expected Status |
|----------|----------------|
| Happy path | 200 or 201 |
| No auth token | 401 |
| Wrong role/ownership | 403 |
| Missing required field | 400 |
| Invalid field type | 400 |
| Resource not found | 404 |
| At least one edge case | varies |

---

## Role 4: Integration-Tester

```yaml
model: sonnet
subagent_type: general-purpose
mode: bypassPermissions
consumes: [Schema-Designer.output, Endpoint-Builder.output, Test-Writer.output]
```

### Prompt

```
## Objective

Verify that the schema, endpoints, and tests produced for this task are internally consistent and production-ready:

{task_context}

## Schema Output (from Schema-Designer)

{Schema-Designer.output}

## Endpoint Output (from Endpoint-Builder)

{Endpoint-Builder.output}

## Test Output (from Test-Writer)

{Test-Writer.output}

## Your Responsibilities

Run the following verification checklist and produce a structured report.

## Verification Checklist

### 1. Schema–Endpoint Alignment
- Do the endpoints read/write only fields that exist in the schema?
- Are all required DB fields covered by endpoint input validation?
- Are response types consistent with schema field types (e.g., no string returned where schema says Int)?
- Are relations traversed correctly (no missing joins, no over-fetching)?

### 2. Test–Endpoint Alignment
- Does every endpoint produced by Endpoint-Builder have a corresponding test file?
- Does every test import the correct file path for the endpoint it tests?
- Do test input shapes match the endpoint's declared input types?
- Do test assertions match the endpoint's declared output types?

### 3. API Contract Verification
- Are HTTP methods correct (no GET with side effects, no POST for idempotent reads)?
- Are status codes semantically correct per REST conventions?
- Is the error response format consistent across all endpoints?
- Are auth requirements explicitly enforced (not just documented)?

### 4. Missing Coverage
- List any endpoints that have no test file
- List any error scenarios that are handled in endpoint code but have no test
- List any schema constraints that have no corresponding validation test

### 5. Integration Issues
- Check for missing imports or wrong import paths in endpoint files
- Check for missing imports or wrong import paths in test files
- Check for type incompatibilities between what endpoints return and what tests assert
- Check for missing environment variables or test database setup steps

## Expected Output

Produce a structured report with these sections:

**Test Run Summary**
- Estimated: X passed / Y failed / Z skipped (based on static analysis if tests cannot be run)
- Overall verdict: READY / NEEDS FIXES

**Problems Found** (one block per problem)
- Test or file name
- Error or inconsistency description
- Root cause
- Fix suggestion (concrete, copy-pasteable if possible)

**Contract Verification Summary**
- Response types consistent with schema: PASS / FAIL
- Error codes consistent across endpoints: PASS / FAIL
- Auth enforced on all non-public endpoints: PASS / FAIL
- All endpoints have test coverage: PASS / FAIL

**Missing Coverage List**
- Endpoint or scenario not covered, with suggested test description

## Quality Criteria

- Every problem has a concrete fix suggestion
- Contract verification covers all four checks
- Missing coverage list is complete (no gaps)
- Report is actionable: a developer can read it and know exactly what to fix
```

### Output Contract

| Artifact | Format | Required |
|----------|--------|----------|
| Test run summary | passed/failed/skipped counts + verdict | Yes |
| Problems found | per-problem blocks with fix suggestions | Yes |
| Contract verification | four-item checklist with PASS/FAIL | Yes |
| Missing coverage list | endpoint/scenario + suggested test | Yes |

---

## Devil's Advocate Focus

The quality gate must challenge the swarm output on these six questions:

1. **Scalability** — How does the API perform at 100x current load? Are there N+1 queries hidden inside list endpoints or nested resolvers?
2. **Race conditions** — What happens when two concurrent requests write to the same resource? Are there missing transactions, missing unique constraints, or missing optimistic locking?
3. **Authorization completeness** — Is every endpoint correctly guarded? Can a user read or modify another user's data by manipulating path params or request body IDs?
4. **Error handling consistency** — Do all endpoints return errors in the same format? Are error messages safe to expose (no stack traces, no internal paths, no DB error details)?
5. **Missing indices** — Are there queries that will become slow table scans as data grows? Are filtered, sorted, and joined columns indexed?
6. **Response size** — Are list endpoints unbounded? Do they need pagination, cursor-based or offset-based? Are responses returning more fields than the client needs?

---

## Success Criteria

All six must be met before the quality gate approves the output:

- [ ] Every endpoint validates all inputs at the system boundary before use
- [ ] Every non-public endpoint has an explicit, tested auth check
- [ ] All schema migrations are reversible (up and down defined)
- [ ] Tests cover at least: happy path, 401, 403, 400 validation error, 404 per endpoint
- [ ] No N+1 queries: list endpoints use batch queries or eager loading, not loops with individual DB calls
- [ ] Response types in endpoint code are consistent with schema field types
