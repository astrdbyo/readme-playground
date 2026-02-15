<div align="center">

<p align="center">
  <img src="docs/assets/header.png" alt="Banner" />
</p>

# Intcrews Hub Monorepo

A production-ready **enterprise monorepo** containing both frontend and backend applications, designed for internal engineering teams and scalable long-term usage.  
Structured for clarity, maintainability, and fast onboarding. 

### Built With

[![Node][Node.js]][Node-url]
[![TypeScript][TypeScript.js]][TypeScript-url]
[![Express][Express.js]][Express-url]
[![PNPM][PNPM.js]][PNPM-url]
[![Docker][Docker.js]][Docker-url]
[![Postgres][Postgres.js]][Postgres-url]
[![Redis][Redis.js]][Redis-url]

This repository eliminates the typical multi-day setup required for a clean, maintainable backend foundation.  
Simply **clone → init → start building**.

</div>

## &nbsp;

## 🧩 Key Features

- **Enterprise-ready monorepo architecture**
  - Clear separation between `apps/` and `packages/`
  - Shared tooling and consistent standards
- **Packages-based repository layer**
  - Database repositories are implemented as reusable workspace packages
  - Applications consume repositories via package imports (e.g. `@repo/...`)
- **Layered backend design**
  - Request → Routes → Controller → Service → Repository → Database
- **Service / Repository pattern**
  - Services are business-logic-only (DTO contracts)
  - Repositories are the only layer allowed to touch ORM/SQL
- **Standardized API response format**
  - Unified success, list, and error structures
  - Built-in request id for easier tracing and debugging
- **Validation-first request handling**
  - Zod schemas at route level
  - Controllers trust validated payloads
- **Production-grade operational defaults**
  - Docker-ready workflows
  - Environment-based configuration (dev/staging/prod/test)
  - Centralized logging & error handling
- **Built for teams**
  - Predictable patterns
  - Fast onboarding
  - Long-term maintainability

---

## 📚 Table of Contents

- [Repository Overview](#-repository-overview)
- [Monorepo Structure](#-monorepo-structure)
- [Quick Start](#-quick-start)
- [Docker](#-docker)
- [Documentation](#-documentation)
- [Development Rules](#-development-rules)
- [Contributing](#-contributing)

---

## 🧭 Repository Overview

This is a monorepo that contains:

- **Applications** (deployable runtimes)
- **Shared packages** (database layer, validation, shared utilities, configs)
- **Centralized tooling** (linting, formatting, TS configs, hooks)

A key design decision in this repository is that the **repository layer lives in workspace packages**, not inside the application source folder.  
This ensures the database access layer is:

- reusable across multiple applications
- consistent across the codebase
- easier to test and maintain independently

---

## 🗂 Monorepo Structure

```txt
apps/                     # deployable applications (runtime entry points)
packages/                 # shared libraries (db repositories, shared modules)
docs/                     # documentation (architecture, setup, operations)

```

[Node.js]: https://img.shields.io/badge/Node.js-000000?style=for-the-badge&logo=node.js&logoColor=white
[Node-url]: https://nodejs.org

[TypeScript.js]: https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white
[TypeScript-url]: https://www.typescriptlang.org

[Express.js]: https://img.shields.io/badge/Express-000000?style=for-the-badge&logo=express&logoColor=white
[Express-url]: https://expressjs.com

[PNPM.js]: https://img.shields.io/badge/PNPM-F69220?style=for-the-badge&logo=pnpm&logoColor=white
[PNPM-url]: https://pnpm.io

[Docker.js]: https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white
[Docker-url]: https://www.docker.com

[Postgres.js]: https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white
[Postgres-url]: https://www.postgresql.org

[Redis.js]: https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white
[Redis-url]: https://redis.io


