[← Back to Main README](../../README.md)

## 📘 Architecture Overview

This project follows a layered architecture to keep concerns separated, improve maintainability, and make the codebase easy to scale
At a high level, the request flow is:
Request → Routes → Controller → Service → Repository → Database

### Project Structure

```sh
.github/                    # GitHub workflows, templates, CI/CD configs
src/
├── controllers/            # HTTP request handlers (API layer)
│   ├── v1/                 # Version 1 API controllers
│   └── v2/                 # Version 2 API controllers (future-safe)
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
│   ├── logs/               # Logging utilities and configuration
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
│   └── unit/               # Unit tests
│
├── main.ts                 # Application entry point
└── server.ts               # HTTP server bootstrap

````
