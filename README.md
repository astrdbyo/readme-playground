[Home](../../README.md) / Database Guide

**[Home](../../README.md)** / Database Guide

[← Back to Home](../../README.md)

---

## 📘 Database Setup & Management Guide

This project uses Drizzle ORM as the SQL toolkit and migration system. It was chosen because:
- ⚡ Faster than Prisma
- 🧩 Schema structure similar to Sequelize models
- 🧱 Simple & predictable migrations
- 🛡️ Safe environment loader (app will not crash even if DB is missing)
- 🚀 Clear workflow for schema → migration → database

### 🛠 Drizzle Workflow

1. Create schema
   Write your table definitions in src/database/schema/.

2. Generate migration files
&nbsp;

```sh
   pnpm db:generate
```

3. Apply migration to the database
&nbsp;

```sh
pnpm db:migrate
```

4. Run seeders
&nbsp;

```sh
pnpm db:seed --file=roles
pnpm db:seed --file=users
pnpm db:seed --file=20251126T120612.roles.seed.ts
```


### ♻️ Database Reset

When running:
&nbsp;

```sh
pnpm db:reset
```
This will drop & recreate your schema.
<br>
After resetting, always re-run:
&nbsp;

```sh
pnpm db:generate
```
&nbsp;
Otherwise Drizzle will not regenerate fresh SQL migration files

### 🧱 RDBMS Design (ERD)
We provide a sample mockup database design to illustrate a clean and scalable relational structure. This example helps guide your development process, especially as your application grows into a larger and more complex system.
If you prefer visual modeling, you can also recreate or extend this ERD using tools like **Lucidchart**, which makes planning and scaling your database architecture much easier
<br>

```sh
┌──────────────┐           ┌────────────────┐
│    roles     │ 1       ∞ │     users      │
├──────────────┤───────────┤────────────────┤
│ id (uuid)    │           │ id (uuid)      │
│ name (text)  │           │ full_name      │
│ admin        │           │ email          │
│ user         │           │ password       │
│              │           │ role_id (uuid) │ (FK → roles.id)
└──────────────┘           └────────────────┘
```

### 🌐 pgAdmin Setup

We include pgAdmin so you can view, edit, and browse your database visually, similar to a GUI client.

1. Open pgAdmin: http://localhost:5050
   - Login using credentials from docker-compose.yml:</br>
      - Email:    admin@docker.com
      - Password: lorem_ipsum
        
2. Register a new server
   - 🔧 Servers → Register → Server

3. Fill the details
   
**General tab**
| Field | Value |
|-------|-------|
| Name | express-postgres |


**Connection tab**
| Field | Value |
|------|-------|
| Host name / Address | postgres |
| Port | 5432 |
| Username | admin |
| Password | admin |

