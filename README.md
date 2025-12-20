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
3. Log files are generated only in the production environment. See the initial setup documentation for activation details.

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
