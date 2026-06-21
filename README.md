[← Back to Main README](../README.md)

# Development Rules

This document defines the daily engineering agreement used in this repository.

These rules are not here to make development feel harder. Some of them may feel strict at first, but the goal is simple: make the codebase easier to read, easier to continue, and easier to maintain after weeks or months without touching it.

> [!TIP]
> A good system gives people certainty.

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

Use this document as a shared team agreement.

Some rules are mandatory. Some are recommended. Some are things we should avoid. The purpose is not to blame people for small mistakes, but to keep the project comfortable for everyone who has to work with it later.

## 1. Rule Levels

| Level       | Meaning                                                                     |
| ----------- | --------------------------------------------------------------------------- |
| Mandatory   | Must be followed because it affects consistency, safety, or maintainability |
| Recommended | Strongly preferred, but can be adjusted when there is a clear reason        |
| Avoid       | Not always forbidden, but should not become the default pattern             |

Simple rule:

```txt
if a pattern makes the code easier to read later, follow it
if a pattern creates confusion later, avoid it
```

## 2. Commit Message Rules

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

| Rule                         | Level       |
| ---------------------------- | ----------- |
| Use allowed commit type      | Mandatory   |
| Include subject              | Mandatory   |
| Do not use scope             | Mandatory   |
| Keep subject short and clear | Recommended |

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

## 3. Naming Conventions

Naming should make the project easy to scan.

A developer should be able to understand the responsibility of a folder, file, class, or function without opening too many files.

| Category       | Rule                                                | Level       | Example / Notes                                                                                  |
| -------------- | --------------------------------------------------- | ----------- | ------------------------------------------------------------------------------------------------ |
| Folder Naming  | Use plural names for grouped responsibility folders | Recommended | `controllers/`, `services/`, `repositories/`, `schemas/`, `middlewares/`                         |
| Folder Naming  | Use lowercase                                       | Mandatory   | `middleware/`, `database/`, `shared/`                                                            |
| Folder Naming  | Use hyphen for multi-word folders                   | Recommended | `access-service/`, `inventory-service/`                                                          |
| Folder Naming  | Use `lib/` for local shared modules inside one app  | Recommended | `lib/dto/`, `lib/helper/`, `lib/utils/`                                                          |
| App Naming     | Use domain or deployable service name               | Mandatory   | `api-gateway`, `access-service`, `inventory-service`                                             |
| Package Naming | Use clear internal package names                    | Mandatory   | `@repo/shared`, `@repo/config`, `@repo/database-access`                                          |
| File Naming    | Use lowercase                                       | Mandatory   | `user.controller.ts`, `company.schema.ts`                                                        |
| File Naming    | Use suffix based on responsibility                  | Mandatory   | `.controller.ts`, `.svc.ts`, `.repository.ts`, `.schema.ts`, `.dto.ts`, `.helper.ts`, `.util.ts` |
| File Naming    | Use `svc` for service files                         | Mandatory   | `user.svc.ts`, `company.svc.ts`                                                                  |
| File Naming    | Use hyphen when file names become long              | Recommended | `company-member.dto.ts`, `workspace-member.repository.ts`                                        |
| File Naming    | Avoid excessive generic `index.ts` usage            | Recommended | Prefer `index.shared.ts`, `index.database.access.ts` when clarity helps                          |
| Variables      | Use explicit names                                  | Mandatory   | `userAccountId`, `companyId`, `isActive`, `accessToken`                                          |
| Variables      | Avoid unclear single-letter names in business logic | Avoid       | Avoid `x`, `e`, `r`                                                                              |
| Variables      | Short names are allowed in small local scope        | Recommended | `i` in loops, `tx` in transactions, `item` in callbacks                                          |
| Functions      | Use camelCase                                       | Mandatory   | `createUser()`, `getCompanyDetail()`                                                             |
| Functions      | Use verbs for actions                               | Recommended | `validatePayload()`, `resolveExecution()`, `mapToResponse()`                                     |
| Classes        | Use PascalCase                                      | Mandatory   | `UserController`, `CompanyService`, `UserRepository`                                             |
| Classes        | One class should represent one responsibility       | Recommended | Avoid generic names like `Manager` or `Handler` without context                                  |
| Types          | Use PascalCase                                      | Mandatory   | `CreateUserDTO`, `CompanyMemberPayload`                                                          |
| Types          | Use semantic suffixes                               | Recommended | `DTO`, `Payload`, `Response`, `Result`, `Context`                                                |
| Interfaces     | Interface prefix `I` is allowed when useful         | Recommended | `IUserPublic`, `IUserPrivate`                                                                    |
| Constants      | Use UPPER_SNAKE_CASE for true constants             | Mandatory   | `MAX_RETRY_COUNT`, `DEFAULT_TIMEOUT`                                                             |
| Constants      | Use named constant objects for domain values        | Recommended | `SYSTEM_ROLE`, `COMPANY_STATUS`, `WORKSPACE_ROLE`                                                |
| Unused Params  | Prefix with underscore                              | Recommended | `_req`, `_res`, `_next`, `_context`                                                              |

## 4. Prisma and Database Naming

Database naming must stay predictable because schema changes are expensive and long-lived.

| Category          | Rule                             | Level       | Example / Notes                             |
| ----------------- | -------------------------------- | ----------- | ------------------------------------------- |
| Prisma Model      | Use PascalCase and singular name | Mandatory   | `UserAccount`, `Company`, `WorkspaceMember` |
| Prisma Enum       | Use PascalCase for enum name     | Mandatory   | `SystemRole`, `UserStatus`, `WorkspaceRole` |
| Prisma Enum Value | Use lowercase values             | Mandatory   | `superuser`, `admin`, `active`, `disabled`  |
| Database Field    | Use camelCase in Prisma schema   | Mandatory   | `companyId`, `createdAt`, `isActive`        |
| ID Field          | Use explicit domain id           | Mandatory   | `userAccountId`, `companyId`, `workspaceId` |
| Join Table Model  | Use domain membership name       | Recommended | `CompanyMember`, `WorkspaceMember`          |
| Repository File   | Match domain model name          | Recommended | `company-member.repository.ts`              |
| Seed File         | Use timestamp and domain name    | Recommended | `20260415T165240.company.seed.ts`           |

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

## 5. Comment Style

Comments should help future developers understand intent quickly.

Use English for day-to-day comments.

| Rule                                                           | Level       |
| -------------------------------------------------------------- | ----------- |
| Keep comments short and clear                                  | Recommended |
| Use lowercase comments                                         | Recommended |
| Prefer inline comments when possible                           | Recommended |
| Explain why, not only what                                     | Recommended |
| Use `;` to separate related statements in one comment          | Recommended |
| Do not use emoji or emoticon in code comments                  | Avoid       |
| Do not write noisy paragraph comments                          | Avoid       |
| Do not leave unclear comments like `fix later` without context | Avoid       |

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

Better replacement:

```ts
// temporary compatibility layer; remove after inventory service migrates to new context contract
```

## 6. Helper Rules

Helpers are allowed, but they must stay clean.

Helpers should reduce service complexity, not hide business flow.

Good helper use cases:

```txt
mapper
transform
payload builder
normalization
merge state
pure validation
small business rule extraction
```

Avoid using helpers for:

```txt
repository access
orm access
external sdk call
transaction
audit log
http request
side effect
```

Rule of thumb:

```txt
if it talks to database, network, filesystem, or external service, it is not a helper
```

Put that logic in service, repository, or utility based on responsibility.

## 7. Controller, Service, Repository Boundary

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

## 8. Shared Package Rule

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

## 9. Typecheck Rule

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

## 10. Environment and Safety Rules

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

## Quick Rules

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
