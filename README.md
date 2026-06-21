[← Back to Main README](../README.md)

# Development Rules

This document defines the daily engineering rules used in this repository.

Some rules may feel a little strict or slower at first. That is normal. The goal is not to make development harder, but to make the codebase easier to read, easier to continue, and easier to maintain after weeks or months without touching it.

A good system is a system with certainty.

When naming, structure, comments, commits, and patterns are predictable, developers spend less energy remembering old context and more energy solving the actual problem.

These rules exist to help the team:

```txt
Read Code Faster
Remember Old Code Faster
Reduce Unclear Patterns
Reduce Maintenance Stress
Avoid Repeated Decision-Making
Make Onboarding Easier
```

## 1. Commit Message Rules

Commit messages must use this format:

```sh
type: subject
```

Allowed commit types:

```txt
feat
fix
chore
refactor
docs
test
```

Rules:

```txt
scope is not allowed
subject is required
subject should be short and clear
```

Valid examples:

```sh
feat: add company member endpoint
fix: prevent workspace member role mismatch
docs: update route usage guideline
refactor: simplify user repository scope
test: add company service unit tests
chore: update dependencies
```

Invalid examples:

```sh
feat(api): add new endpoint
fix
docs:
```

## 2. Naming Conventions

Naming should make the project easy to scan.

Use predictable names so developers can understand the responsibility of a file without opening it first.

| Category       | Rule                                                | Example / Notes                                                                                  |
| -------------- | --------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| Folder Naming  | Use plural names for grouped responsibility folders | `controllers/`, `services/`, `repositories/`, `schemas/`, `middlewares/`                         |
| Folder Naming  | Use lowercase                                       | `middleware/`, `database/`, `shared/`                                                            |
| Folder Naming  | Use hyphen for multi-word folders                   | `access-service/`, `inventory-service/`                                                          |
| Folder Naming  | Use `lib/` for local shared modules inside one app  | `lib/dto/`, `lib/helper/`, `lib/utils/`                                                          |
| App Naming     | Use domain or deployable service name               | `api-gateway`, `access-service`, `inventory-service`                                             |
| Package Naming | Use clear internal package names                    | `@repo/shared`, `@repo/config`, `@repo/database-access`                                          |
| File Naming    | Use lowercase                                       | `user.controller.ts`, `company.schema.ts`                                                        |
| File Naming    | Use suffix based on responsibility                  | `.controller.ts`, `.svc.ts`, `.repository.ts`, `.schema.ts`, `.dto.ts`, `.helper.ts`, `.util.ts` |
| File Naming    | Use `svc` for service files                         | `user.svc.ts`, `company.svc.ts`                                                                  |
| File Naming    | Use hyphen when file names become long              | `company-member.dto.ts`, `workspace-member.repository.ts`                                        |
| File Naming    | Avoid excessive generic `index.ts` usage            | Prefer `index.shared.ts`, `index.database.access.ts` when clarity helps                          |
| Variables      | Use explicit names                                  | `userAccountId`, `companyId`, `isActive`, `accessToken`                                          |
| Variables      | Avoid unclear single-letter names in business logic | Avoid `x`, `e`, `r`                                                                              |
| Variables      | Short names are allowed in small local scope        | `i` in loops, `tx` in transactions, `item` in callbacks                                          |
| Functions      | Use camelCase                                       | `createUser()`, `getCompanyDetail()`                                                             |
| Functions      | Use verbs for actions                               | `validatePayload()`, `resolveExecution()`, `mapToResponse()`                                     |
| Classes        | Use PascalCase                                      | `UserController`, `CompanyService`, `UserRepository`                                             |
| Classes        | One class should represent one responsibility       | Avoid generic names like `Manager` or `Handler` without context                                  |
| Types          | Use PascalCase                                      | `CreateUserDTO`, `CompanyMemberPayload`                                                          |
| Types          | Use semantic suffixes                               | `DTO`, `Payload`, `Response`, `Result`, `Context`                                                |
| Interfaces     | Interface prefix `I` is allowed when useful         | `IUserPublic`, `IUserPrivate`                                                                    |
| Constants      | Use UPPER_SNAKE_CASE for true constants             | `MAX_RETRY_COUNT`, `DEFAULT_TIMEOUT`                                                             |
| Constants      | Use named constant objects for domain values        | `SYSTEM_ROLE`, `COMPANY_STATUS`, `WORKSPACE_ROLE`                                                |
| Unused Params  | Prefix with underscore                              | `_req`, `_res`, `_next`, `_context`                                                              |

## 3. Prisma and Database Naming

Database naming must stay predictable.

| Category          | Rule                             | Example / Notes                             |
| ----------------- | -------------------------------- | ------------------------------------------- |
| Prisma Model      | Use PascalCase and singular name | `UserAccount`, `Company`, `WorkspaceMember` |
| Prisma Enum       | Use PascalCase for enum name     | `SystemRole`, `UserStatus`, `WorkspaceRole` |
| Prisma Enum Value | Use lowercase values             | `superuser`, `admin`, `active`, `disabled`  |
| Database Field    | Use camelCase in Prisma schema   | `companyId`, `createdAt`, `isActive`        |
| ID Field          | Use explicit domain id           | `userAccountId`, `companyId`, `workspaceId` |
| Join Table Model  | Use domain membership name       | `CompanyMember`, `WorkspaceMember`          |
| Repository File   | Match domain model name          | `company-member.repository.ts`              |
| Seed File         | Use timestamp and domain name    | `20260415T165240.company.seed.ts`           |

Good enum example:

```prisma
enum UserStatus {
  pending
  active
  disabled
}
```

Avoid enum values like this:

```prisma
enum UserStatus {
  PENDING
  ACTIVE
  DISABLED
}
```

Reason:

```txt
database enum values should stay simple, stable, and lowercase
application constants can map them into readable code
```

## 4. Comment Style

Comments should help future developers understand intent quickly.

Use English for day-to-day comments.

Keep comments:

```txt
short
lowercase
clear
inline when possible
focused on why, not only what
```

Use `;` to separate related statements in one comment.

Good:

```ts
// validate company scope; prevents cross-company access
```

```ts
// normalize payload; empty string should not be stored as value
```

```ts
// helper exists to reduce service complexity and noise; pure business logic
// allowed for mapper, transform, payload builder, normalization, merge state, validation
// forbidden for side effects, repository and orm access, external sdk, transaction
// avoid coupling helper to repository return types; prefer explicit local types to reduce maintenance cost
```

Avoid:

```ts
// This function is used to validate the company scope because sometimes the user might access another company from the frontend and that is dangerous because the backend needs to make sure the selected company is correct.
```

Also avoid:

```ts
// IMPORTANT!!! DO NOT CHANGE THIS 😭🔥
```

Rules:

```txt
no emoji
no emoticon
no noisy paragraph comments
no emotional comments
no unclear comments like "fix later" without context
```

Better replacement:

```ts
// temporary compatibility layer; remove after inventory service migrates to new context contract
```

## 5. Helper Rules

Helpers are allowed, but they must stay clean.

Helpers should reduce service complexity, not hide business flow.

Good use cases:

```txt
mapper
transform
payload builder
normalization
merge state
pure validation
small business rule extraction
```

Forbidden use cases:

```txt
repository access
orm access
external sdk call
transaction
audit log
http request
side effect
```

Good helper comment:

```ts
// helper exists to reduce service complexity and noise; pure business logic
// allowed for mapper, transform, payload builder, normalization, merge state, validation
// forbidden for side effects, repository and orm access, external sdk, transaction
// avoid coupling helper to repository return types; prefer explicit local types to reduce maintenance cost
```

Rule of thumb:

```txt
if it talks to database, network, filesystem, or external service, it is not a helper
```

Put that logic in service, repository, or utility based on responsibility.

## 6. Controller, Service, Repository Boundary

Keep backend layers predictable.

| Layer      | Responsibility                                                          | Should Avoid                                         |
| ---------- | ----------------------------------------------------------------------- | ---------------------------------------------------- |
| Route      | authorization, validation, controller binding                           | business logic                                       |
| Controller | read validated input, read context, call service, return response       | repository access, complex branching                 |
| Service    | business rules, defensive checks, orchestration, DTO mapping, audit log | raw SQL or ORM query details                         |
| Repository | database query, mutation, transaction                                   | Express request, frontend behavior, route permission |
| DTO        | response mapping                                                        | business validation                                  |
| Schema     | request validation                                                      | database query logic                                 |
| Helper     | pure business utility                                                   | side effects                                         |

Simple backend flow:

```txt
route -> controller -> service -> repository -> database
```

Controller example:

```ts
async detail(req: Request, res: Response) {
  const request = req as ExpressRequestWithContext
  const { params } = res.locals.validated

  const data = await this.service.detail(params.companyId, request.context)

  return res.json(HttpResponse.success(req, data))
}
```

Service may still perform defensive checks even when middleware already validates access.

This protects the code when routes change later.

## 7. Shared Package Rule

`packages/shared` is the internal SDK.

It should be reusable, modular, and framework-agnostic.

Shared may contain:

```txt
types
constants
validation primitives
error helpers
response helpers
normalization utilities
request context contracts
role mapping helpers
webhook utilities
```

Shared must not depend on:

```txt
Express route behavior
Next.js page behavior
Prisma repositories
service-specific business logic
frontend-only components
```

Good shared content:

```txt
RequestContext type
SYSTEM_ROLE constant
PaginationQuerySchema
ErrorResponse
Normalization.email()
```

Bad shared content:

```txt
CompanyService business rule
Inventory repository query
React component
Express controller
```

Rule:

```txt
if it only makes sense for one app, keep it inside that app
if it is reusable across apps/packages, consider shared
```

## 8. Typecheck Rule

Every workspace that contains TypeScript runtime code must have typecheck.

Add `typecheck` when the workspace has:

```txt
tsconfig.json
src/
.ts or .tsx files
runtime code
code consumed by another app/package
```

Example:

```json
{
  "scripts": {
    "typecheck": "tsc --noEmit"
  }
}
```

For buildable packages:

```json
{
  "scripts": {
    "typecheck": "tsc -p tsconfig.json --noEmit"
  }
}
```

Root command:

```sh
pnpm turbo typecheck
```

Do not skip typecheck just because the package is small.

Small packages can still break the monorepo.

## 9. Environment and Safety Rules

Use the correct environment for the correct purpose.

```txt
development -> local development
staging     -> real integration testing
production  -> real users and real data
test        -> automated tests
```

Developers should not use production for testing changes.

Production credentials, production database, and production services should not be used for experiments.

Auth mode recommendation:

```txt
development -> full-bypass
staging     -> semi-bypass
production  -> strict
```

Production must use strict authentication.

Do not enable bypass mode in production.

## 10. Quick Rules

```txt
make route files readable
keep naming predictable
use english comments
keep comments short and lowercase
do not use emoji or emoticon in code comments
use lowercase prisma enum values
keep helpers pure
keep shared framework-agnostic
use service for business rules
use repository for database access
typecheck every TypeScript workspace
never test with production data or production credentials
```
