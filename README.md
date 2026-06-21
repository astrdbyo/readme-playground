[← Back to Main README](../README.md)

# Monorepo Structure

This document explains the high-level structure of the Intcrews Hub monorepo.

The goal is to make the repository easy to understand, easy to typecheck, and safe to deploy across development, staging, and production environments.

## 1. High-Level Architecture

```txt
CLIENT
  │
  ▼
NGINX
  │
  ▼
API GATEWAY
  │
  ├── resolve auth/context through Access Service
  ├── inject trusted internal headers
  └── route request to target service
          │
          ├── Access Service
          ├── Inventory Service
          ├── Seafarer Service
          └── Future Services
```

The API Gateway is the public entry point for backend traffic.

Access Service is the source of truth for identity, company access, workspace access, and execution context.

Downstream services consume trusted context from the Gateway and should not resolve user authorization directly from the frontend.

## 2. Current Monorepo Skeleton

```txt
apps/
  api-gateway/
  access-service/
  inventory-service/
  seafarer-service/
  crew-operations-api/     # legacy reference app
  web/

packages/
  shared/
  config/
  redis/
  kafka/
  swagger/
  vitest/
  eslint/
  typescript/
  database/
    access/
    inventory/
    seafarer/
  crew-operations-db/      # legacy database package

docs/
  01-quick-start.md
  02-monorepo.md
  03-rules.md
  04-docker.md
  05-workflows.md
  06-route.md
```

## 3. Apps vs Packages

Use this simple rule:

```txt
apps/     -> deployable applications
packages/ -> reusable internal packages
```

Examples of deployable applications:

```txt
api-gateway
access-service
inventory-service
web
```

Examples of reusable packages:

```txt
@repo/shared
@repo/config
@repo/redis
@repo/database-access
@repo/database-inventory
```

Apps may depend on packages.

Packages should not depend on apps.

## 4. Shared Package as Internal SDK

`packages/shared` is the internal SDK of the platform.

It should contain reusable contracts and utilities such as:

```txt
types
constants
validation schemas
error helpers
response helpers
normalization utilities
request context contracts
role mapping helpers
webhook utilities
```

Design rule:

```txt
shared must be framework-agnostic
shared must not understand app architecture
shared must not depend on Express, Next.js, Prisma, or service-specific logic
```

Good shared content:

```txt
Role constants
RequestContext type
Http response helper
Zod primitive schemas
ErrorResponse helper
Normalization utility
```

Avoid putting this into shared:

```txt
Access Service business rules
Inventory Service business rules
Express route handlers
Prisma repository logic
Frontend component logic
```

If a utility only makes sense for one service, keep it inside that service.

If it is reusable across multiple apps/packages, move it into shared.

## 5. Database Packages

Database access is separated by domain.

```txt
packages/database/access
packages/database/inventory
packages/database/seafarer
```

Each database package owns:

```txt
Prisma schema
migrations
seeders
Prisma client
repositories
database-specific access logic
```

Services should call repositories from the correct database package.

Example:

```txt
Access Service
-> uses packages/database/access

Inventory Service
-> uses packages/database/inventory
```

Avoid cross-database foreign keys between service databases.

Use IDs and trusted context snapshots when a downstream service needs references from Access Service.

## 6. Typecheck Rule

We use Turbo-orchestrated typecheck.

Every workspace that contains TypeScript runtime code must have a `typecheck` script.

Add `typecheck` when the package has:

```txt
tsconfig.json
src/
.ts or .tsx files
runtime code
code consumed by another app/package
```

Example package script:

```json
{
  "scripts": {
    "typecheck": "tsc --noEmit"
  }
}
```

For buildable packages, use the package’s own TypeScript config if needed:

```json
{
  "scripts": {
    "typecheck": "tsc -p tsconfig.json --noEmit"
  }
}
```

Root typecheck should be orchestrated by Turbo:

```sh
pnpm turbo typecheck
```

This keeps type safety consistent across apps and packages.

## 7. Environment Rules

The project uses environment-specific `.env` files.

```txt
.env.development
.env.staging
.env.production
.env.test
```

Developers should not use production to test changes.

Use staging for real integration testing.

```txt
development -> local development
staging     -> real integration testing
production  -> real users and real data
test        -> automated tests
```

Production credentials, production database, and production services should not be used for experiments.

## 8. Auth Mode

Backend services may use `AUTH_MODE` to control authentication behavior per environment.

```ts
AUTH_MODE: 'full-bypass' | 'semi-bypass' | 'strict'
```

Recommended usage:

```txt
development -> full-bypass
staging     -> semi-bypass
production  -> strict
```

Meaning:

```txt
full-bypass
-> local development only
-> useful when developer needs to work without real auth flow

semi-bypass
-> staging only
-> useful for controlled testing while keeping service boundaries closer to real behavior

strict
-> production
-> real authentication and authorization flow
```

Production must use strict authentication.

Do not use bypass mode in production.

## 9. Server Deployment Layout

Server deployment folders are separated by layer.

```txt
/opt/docker
├── infrastructures-layer
│   └── edge
│       └── reverse-proxy
│
└── applications-layer
    ├── intcrews-hub
    │   ├── api
    │   │   ├── prod
    │   │   └── staging
    │   └── web
    │       ├── prod
    │       └── staging
    └── intcrews-cdn
```

Simple rule:

```txt
infrastructures-layer -> shared infrastructure
applications-layer    -> deployable apps
```

Keep staging and production separated.

Do not reuse production folders for staging tests.

## 10. Quick Rules

```txt
apps are deployable
packages are reusable
shared is internal SDK
shared must stay framework-agnostic
database packages own Prisma and repositories
each TypeScript workspace needs typecheck
development uses full-bypass
staging uses semi-bypass
production uses strict auth
never test using production data or production credentials
```
