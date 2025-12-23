[[← Back to Main README](../../README.md)

## ⛓️‍💥 API Layer Overview

### 🎲 Controller-Based API Versioning# CI / CD Workflow Guide

This document explains **why these workflows exist**, **what each one does**, and **how to use them**.  
It is written for future reference, so you do not need to re-learn the entire CI/CD concept again.

---

## Purpose

This repository uses **GitHub Actions** to automate the full software delivery lifecycle:

1. **Validate code before merge**
2. **Build Docker images**
3. **Deploy safely to staging**
4. **Deploy explicitly to production**
5. **Create a production release record**

Each step has **one responsibility only**.  
Nothing is automatic in production by design.

---

## Workflow Overview



```sh
src/
├── controllers/ 
│   ├── v1/                
│   └── v2/   
```

API versioning is required to ensure backward compatibility. It allows developers to deploy new logic safely and roll back API behavior by switching versions, rather than reverting Git commits.

### 📦 Standardized API Response Format

To ensure consistency across all endpoints, this project uses a **centralized response utility** to standardize API responses.  
All controllers **must return responses using `HttpResponse`**, rather than manually shaping JSON objects.

This approach helps to:
- Keep response formats consistent
- Make APIs easier to consume
- Simplify future changes to response structure
- Reduce duplicated logic in controllers


### ⚙️ Controller Usage

Controllers should focus only on:
- Handling incoming requests
- Calling the service layer
- Returning standardized responses

Use the following patterns:

- **Single resource**
  - `HttpResponse.success(req, data)`
- **List / collection**
  - `HttpResponse.list(req, items, meta)`

Example:

```ts
return res.status(200).json(
  HttpResponse.list(req, users, { total: users.length })
)
```

### 📄 Response Utility Location

All response formatting logic is centralized in:

```txt
src/lib/utils/response.util.ts
```

If you need to:
- Adjust the response structure
- Add or remove fields
- Modify metadata or error format
<br>👉 **Update the implementation in** `response.util.ts`, not inside controllers.

This ensures consistent behavior across all endpoints and API versions


🚫 What to Avoid

- Returning raw objects directly from controllers
- Manually constructing JSON responses per endpoint
- Creating inconsistent response formats

✅ Why This Matters

Standardized API responses make it easier to:
- Debug requests using shared identifiers (e.g. requestId)
- Integrate frontend or external consumers
- Maintain backward compatibility across API versions


This keeps the structure **readable, clean, and Markdown-friendly** for your README
](https://github.com/astrdbyo/readme-playground/tree/main)
