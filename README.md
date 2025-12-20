[← Back to Main README](../../README.md)

## 📘 Architecture Overview

This project follows a layered architecture to keep concerns separated, improve maintainability, and make the codebase easy to scale.<br>

<div align="center">

**When a request comes in, it goes through these layers**<br/>
Request → Routes → Controller → Service → Repository → Database

</div>

<br>
<br>

> [!TIP]
> A quick tip to help you understand and work with this project more easily.

1. Review the scripts in `package.json`, as some of them are already set up to make your work easier.
2. Run all tests before committing to reduce the risk of bugs or failed deployments.
3. Log files are generated only in production. See the `initial setup` docs to enable them.
4. Follow the [Naming Conventions](#naming-conventions) to maintain consistency and improve readability.

<br>
<br>

### Project Structure

```sh
.github/                    # GitHub workflows, actions
src/
├── controllers/            # HTTP request handlers (API layer)
│   ├── v1/                
│   └── v2/                 
│
├── database/               # Database-related logic & configuration
│   ├── migrations/         # Database migrations
│   ├── schema/             # Database schema definitions
│   ├── scripts/            # Utility scripts (DB setup, maintenance)
│   ├── seeders/            # Seed data for development/testing
│   ├── db-connection.ts    # Database connection setup
│   ├── drizzle-config.js   # Drizzle ORM configuration for npm, pnpm
│   └── drizzle-config.ts   # Drizzle ORM configuration for bun
│
├── lib/                    # Shared libraries and reusable modules
│   ├── assets/             # Static or shared assets
│   ├── helper/             # Helper functions
│   ├── logs/               # Application logs
│   ├── swagger/            # Swagger / OpenAPI documentation setup
│   ├── types/              # Global TypeScript types and interfaces
│   ├── utils/              # Generic utility functions
│   └── validations/        # Request Zod validation schemas
│
├── middleware/             # Express middleware (auth, error handling, etc.)
│
├── repositories/           # Data access layer (DB queries)
│
├── routes/                 # API route definitions and versioning
│
├── services/               # Business logic layer
│
├── test/                   # Automated tests
│   ├── factories/          # Test data factories
│   ├── integration/        # Integration tests
│   ├── repositories/       # Repository-level tests
│   ├── setup/              # Test environment setup
│   └── unit/               # Logic tests
│
├── main.ts                 # Application entry point
└── server.ts               # HTTP server bootstrap

````

### Naming Conventions

### 📁 Folder & File Naming Conventions

### 📁 Folder, File, and Comment Naming Conventions

| Category | Rule | Example / Notes |
|---------|------|------------------|
| **Folder Naming** | Always plural | `controllers/`, `services/`, `repositories/` |
| | Always lowercase | `database/`, `middleware/` |
| | Use hyphens for multi-word names | `user-profile/`, `auth-service/` |
| | Shared, non-domain modules go here | `lib/` |
| **Database Schema** | Use singular names | `user.schema.ts`, `role.schema.ts` |
| | Group complex domains in subfolders | `user/user.schema.ts` |
| **File Naming** | Always lowercase | `auth.middleware.ts`, `user.svc.ts` |
| | Use descriptive suffixes based on responsibility | `.controller.ts`, `.util.ts`, `.repository.ts` |
| | Use `svc` suffix for services | `user.svc.ts` |
| | Use hyphens only when file names become too long | `app-helper.ts` |
| | Avoid excessive `index.ts` usage | `index.swagger.ts` |
| **Comments** | Keep comments clear and concise | Short, readable explanations |
| | Lowercase comments are allowed | Maintain consistency across the codebase |
| | Semicolons may separate related ideas | `// enable cache; improves performance` |
| **Variables** | Use explicit, descriptive names | `userId`, `isActive`, `authToken` |
| | Avoid unclear single-letter names | Avoid `x`, `e` in business logic |
| | Short names allowed in limited scope | `i` in loops, `item` in callbacks |
| **Functions** | Use camelCase | `createUser()`, `getUserById()` |
| | Use verbs for actions | `validateInput()`, `fetchUsers()` |
| **Classes** | Use PascalCase | `UserController`, `AuthService` |
| | One class per responsibility | Avoid generic names like `Manager` |
| **Types & Interfaces** | Use PascalCase | `User`, `CreateUserDto` |
| **Constants** | Use UPPER_SNAKE_CASE | `MAX_RETRY_COUNT`, `DEFAULT_TIMEOUT` |
| **Unused Parameters** | Prefix with underscore (`_`) | `_req`, `_res`, `_next` |




