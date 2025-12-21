[← Back to Main README](../../README.md)

## ⛓️‍💥 API Layer Overview

### 🎲 Controller-Based API Versioning

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
👉 **Update the implementation in** `response.util.ts`, not inside controllers.

This ensures consistent behavior across all endpoints and API versions
