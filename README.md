[← Back to Main README](../README.md)

# Route Usage Guideline

This guide defines how backend routes should be designed, authorized, and consumed.

The goal is simple:

```txt
Route file should explain the route behavior clearly.
```

A developer should be able to read a route file and quickly understand:

```txt
who can access it
what context is required
what schema is validated
which controller handles it
```

## 1. Route file must be readable

A route should show authorization, validation, and controller flow in one place.

Example:

```ts
router.get(
  '/:companyId/detail',
  authorize({
    accessScope: 'company',
    systemRoles: SYSTEM_ROLES.ALL,
    companyRoles: COMPANY_ROLES.ALL,
    allowPrivilegedGlobalAccess: true,
    enforceCompanyScope: true,
  }),
  validate({ params: CompanyParamSchema }),
  controller.detail.bind(controller),
)
```

Avoid hiding route permission inside service logic unless it is a defensive business check.

## 2. Always choose the route scope first

Every protected route must clearly declare its scope:

```ts
accessScope: 'global' | 'company' | 'workspace' | 'company-or-workspace'
```

Use this mental model:

```txt
global               -> no selected company/workspace context
company              -> selected company context required
workspace            -> selected company + workspace context required
company-or-workspace -> selected company required, workspace optional
```

Examples:

```ts
// platform/global route
authorize({
  accessScope: 'global',
  systemRoles: SYSTEM_ROLES.ADMIN_AND_ABOVE,
})
```

```ts
// company route
authorize({
  accessScope: 'company',
  systemRoles: SYSTEM_ROLES.ALL,
  companyRoles: COMPANY_ROLES.ALL,
})
```

```ts
// workspace runtime route
authorize({
  accessScope: 'workspace',
  systemRoles: SYSTEM_ROLES.ALL,
  companyRoles: COMPANY_ROLES.ALL,
  workspaceRoles: WORKSPACE_ROLES.MEMBER_AND_ABOVE,
})
```

## 3. Role checks are layered

Authorization uses layered roles:

```txt
systemRole    -> platform-level role
companyRole   -> role inside selected company
workspaceRole -> role inside selected workspace
```

They are checked from outside to inside.

```txt
systemRoles + companyRoles + workspaceRoles = layered access rule
```

They are not random OR conditions.

Example:

```ts
authorize({
  accessScope: 'workspace',
  systemRoles: SYSTEM_ROLES.ALL,
  companyRoles: COMPANY_ROLES.ALL,
  workspaceRoles: WORKSPACE_ROLES.MANAGER_AND_ABOVE,
})
```

This means:

```txt
User must be active.
User must have selected company access.
User must have selected workspace access.
User must have enough workspace role.
```

## 4. Use privileged access intentionally

Use `allowPrivilegedGlobalAccess` only when admin/superuser should be allowed to open a company route without selected company execution.

Example:

```ts
authorize({
  accessScope: 'company',
  systemRoles: SYSTEM_ROLES.ALL,
  companyRoles: COMPANY_ROLES.ALL,
  allowPrivilegedGlobalAccess: true,
  enforceCompanyScope: true,
})
```

Common use cases:

```txt
company detail from admin panel
company update from admin panel
company logo update from admin panel
```

Do not use it for workspace runtime routes.

Workspace routes should require workspace context.

## 5. Use `enforceCompanyScope` when route has `:companyId`

If a route has `:companyId`, consider using:

```ts
enforceCompanyScope: true
```

Example:

```ts
router.patch(
  '/:companyId',
  authorize({
    accessScope: 'company',
    systemRoles: SYSTEM_ROLES.ALL,
    companyRoles: COMPANY_ROLES.OWNER_ONLY,
    allowPrivilegedGlobalAccess: true,
    enforceCompanyScope: true,
  }),
  validate({ params: CompanyParamSchema, body: UpdateCompanySchema }),
  controller.update.bind(controller),
)
```

This prevents:

```txt
User selected company A
but tries to access /company/company-B
```

Rule:

```txt
params.companyId must match execution.companyId
```

Do not use `enforceCompanyScope` on routes without `:companyId`.

## 6. Use `superuserOnly` for dangerous actions

Use `superuserOnly` for irreversible or highly sensitive actions.

Examples:

```txt
hard delete
system maintenance
dangerous cleanup
irreversible admin action
```

Example:

```ts
router.delete(
  '/hard/:companyId',
  authorize({
    accessScope: 'global',
    superuserOnly: true,
  }),
  validate({ params: CompanyParamSchema }),
  controller.hardDelete.bind(controller),
)
```

Do not treat admin and superuser as always equal.

```txt
admin      -> platform administrator
superuser  -> highest system authority
```

## 7. Keep read, write, detail, and options routes clear

Read route may return combined data for UI convenience.

Example:

```txt
GET /company/:companyId/detail
```

May return:

```txt
company info
summary
branches
```

But write routes should stay specific.

Good:

```txt
PATCH  /company/:companyId
PATCH  /company/:companyId/logo
DELETE /company/:companyId/logo
PUT    /company/:companyId/branches/information
```

Avoid:

```txt
POST /company/action
PATCH /company/data
POST /company/update-detail
```

Options endpoints should stay small and stable.

Example:

```json
{
  "value": "branch-uuid",
  "label": "Jakarta Head Office",
  "branchCode": "HQ",
  "isHeadOffice": true
}
```

Avoid returning duplicated fields like `value` and `branchId`, or `label` and `displayName`, unless there is a strong reason.

## 8. Keep responsibility boundaries clean

Controller is only an HTTP adapter.

Controller should:

```txt
read validated params/query/body
read request context
call service
return response
```

Service owns business rules.

Service should:

```txt
validate business rules
run defensive scope checks
call repository
map response DTO
record audit log when needed
```

Repository owns database access only.

Repository should:

```txt
query database
update database
use transactions when needed
return database result
```

Repository should not know about Express, frontend behavior, or route authorization.

## Simple route checklist

Before adding a route, answer this:

```txt
1. Is this global, company, workspace, or company-or-workspace?
2. Does it need selected company context?
3. Does it need selected workspace context?
4. Does the route have :companyId?
5. Should enforceCompanyScope be true?
6. Can admin/superuser access it without selected company?
7. Is this a read route, write route, detail route, or options route?
8. Should this action be superuser only?
```

If the route is hard to explain, the route is probably too vague.
