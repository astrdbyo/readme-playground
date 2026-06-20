[← Back to Main README](../README.md)

# Route Usage Guideline

This document defines how routes should be designed and consumed.

The goal is to prevent:

* route ambiguity
* duplicated endpoints
* inconsistent authorization
* hidden business behavior
* frontend coupling
* high learning curve when reading route permission setup

This guide applies to routes inside backend services such as:

* Access Service
* Inventory Service
* future downstream services

## &nbsp;

**1. Route file must be readable**

A route file should clearly show:

* who can access the route
* what context is required
* what schema is validated
* which controller handles the request

Example:

```ts
router.get(
  '/:companyId/detail',
  authorize({
    systemRoles: SYSTEM_ROLES.ALL,
    companyRoles: COMPANY_ROLES.ALL,
    allowWithoutExecution: true,
    enforceCompanyScope: true,
  }),
  validate({ params: CompanyParamSchema }),
  controller.detail.bind(controller),
)
```

A developer should be able to understand the access rule by reading the route file.

Do not hide route permission inside service logic unless it is a defensive check.

## &nbsp;

**2. Authorization has 3 layers**

Authorization uses three layers:

```txt
systemRole
companyRole
workspaceRole
```

These layers are checked from outside to inside.

```txt
systemRole    -> user level in the platform
companyRole   -> user role in the selected company
workspaceRole -> user role in the selected workspace
```

When a route defines more than one role layer, they are checked together.

This means:

```txt
systemRoles + companyRoles + workspaceRoles = layered checks
```

They are not random OR rules.

## &nbsp;

**3. System role**

`systemRoles` controls access at platform level.

Use it for:

* platform pages
* admin pages
* company selector
* global list
* system-level actions

Example:

```ts
authorize({
  systemRoles: SYSTEM_ROLES.ADMIN_AND_ABOVE,
  allowWithoutExecution: true,
})
```

This means:

```txt
Only admin and superuser can access.
No selected company/workspace is required.
```

## &nbsp;

**4. Company role**

`companyRoles` controls access inside selected company context.

Use it when the route is related to a specific company.

Example:

```ts
authorize({
  systemRoles: SYSTEM_ROLES.ALL,
  companyRoles: COMPANY_ROLES.OWNER_ONLY,
  allowWithoutExecution: true,
  enforceCompanyScope: true,
})
```

This means:

```txt
All active users can pass system role check.
Normal user must have selected company context.
User must have owner role in that company.
params.companyId must match selected companyId.
Admin/superuser may access from admin mode without selected company.
```

## &nbsp;

**5. Workspace role**

`workspaceRoles` controls access inside selected workspace context.

Use it when the route is used by a workspace app or downstream service.

Example:

```ts
authorize({
  systemRoles: SYSTEM_ROLES.ALL,
  companyRoles: COMPANY_ROLES.ALL,
  workspaceRoles: WORKSPACE_ROLES.ALL,
  enforceCompanyScope: true,
})
```

This means:

```txt
User must be active.
User must have valid selected company.
User must have valid selected workspace.
Company param must match selected company.
```

Do not use `allowWithoutExecution` for workspace runtime routes unless there is a very clear reason.

Workspace runtime routes should require workspace context.

## &nbsp;

**6. Admin and superuser override**

Admin and superuser have special behavior.

When admin/superuser selects a company or workspace, Access Service resolves effective roles for compatibility:

```txt
admin/superuser in company context
-> owner-equivalent company role

admin/superuser in workspace context
-> manager-equivalent workspace role
```

This allows downstream services to use the same role validation flow.

However:

```txt
superuser and admin are not always the same.
```

Use `superuserOnly` for dangerous actions such as hard delete.

Example:

```ts
authorize({
  superuserOnly: true,
})
```

Use this for:

* hard delete
* irreversible system action
* dangerous maintenance action

## &nbsp;

**7. `allowWithoutExecution`**

`allowWithoutExecution` means:

```txt
This route can run without selected company/workspace context.
```

Use it for:

* platform admin page
* selector page
* global fetch list
* company list
* create company from admin panel

Example:

```ts
authorize({
  systemRoles: SYSTEM_ROLES.ALL,
  allowWithoutExecution: true,
})
```

Do not use it blindly.

If a route also defines `companyRoles` or `workspaceRoles`, normal users still need selected context.

Example:

```ts
authorize({
  systemRoles: SYSTEM_ROLES.ALL,
  companyRoles: COMPANY_ROLES.ALL,
  allowWithoutExecution: true,
})
```

This means:

```txt
Admin/superuser can access without selected company.
Normal user must have selected company.
```

## &nbsp;

**8. `enforceCompanyScope`**

Use `enforceCompanyScope` when the route has `:companyId` param.

Example:

```ts
router.patch(
  '/:companyId',
  authorize({
    systemRoles: SYSTEM_ROLES.ALL,
    companyRoles: COMPANY_ROLES.OWNER_ONLY,
    allowWithoutExecution: true,
    enforceCompanyScope: true,
  }),
  validate({ params: CompanyParamSchema, body: UpdateCompanySchema }),
  controller.update.bind(controller),
)
```

This protects against this case:

```txt
User selected company A
but tries to hit /company/company-B
```

Rule:

```txt
params.companyId must match execution.companyId
```

Use this for company-scoped routes.

## &nbsp;

**9. Route types and examples**

**9.1 Global list route**

Used by admin page and user selector page.

```ts
router.get(
  '/',
  authorize({
    systemRoles: SYSTEM_ROLES.ALL,
    allowWithoutExecution: true,
  }),
  validate({ query: ListCompanyQuerySchema }),
  controller.list.bind(controller),
)
```

Service must still filter data for normal users.

Normal users must not see all companies by accident.

**9.2 Company detail route**

Used to show company detail page.

```ts
router.get(
  '/:companyId/detail',
  authorize({
    systemRoles: SYSTEM_ROLES.ALL,
    companyRoles: COMPANY_ROLES.ALL,
    allowWithoutExecution: true,
    enforceCompanyScope: true,
  }),
  validate({ params: CompanyParamSchema }),
  controller.detail.bind(controller),
)
```

This route may return a composite response.

Example:

```txt
company info
summary
branches
```

Read response can be combined for page needs.

**9.3 Workspace runtime options route**

Used by downstream services or workspace apps.

```ts
router.get(
  '/:companyId/branches/options',
  authorize({
    systemRoles: SYSTEM_ROLES.ALL,
    companyRoles: COMPANY_ROLES.ALL,
    workspaceRoles: WORKSPACE_ROLES.ALL,
    enforceCompanyScope: true,
  }),
  validate({
    params: CompanyParamSchema,
    query: CompanyBranchOptionsQuerySchema,
  }),
  controller.branchOptions.bind(controller),
)
```

This route requires workspace context.

Do not use `allowWithoutExecution`.

This prevents random company-level access outside workspace runtime.

**9.4 Company owner update route**

Used by admin/superuser or company owner.

```ts
router.patch(
  '/:companyId',
  authorize({
    systemRoles: SYSTEM_ROLES.ALL,
    companyRoles: COMPANY_ROLES.OWNER_ONLY,
    allowWithoutExecution: true,
    enforceCompanyScope: true,
  }),
  validate({ params: CompanyParamSchema, body: UpdateCompanySchema }),
  controller.update.bind(controller),
)
```

This means:

```txt
admin/superuser can update from admin mode
company owner can update selected company
other company roles cannot update
```

Service may still protect sensitive fields such as status.

Example:

```txt
Company owner can update company profile.
Only admin/superuser can update company status.
```

**9.5 Hard delete route**

Used only by superuser.

```ts
router.delete(
  '/hard/:companyId',
  authorize({
    superuserOnly: true,
  }),
  validate({ params: CompanyParamSchema }),
  controller.hardDelete.bind(controller),
)
```

Use `superuserOnly` for irreversible actions.

## &nbsp;

**10. Read model and write model can be different**

A detail endpoint may return combined data.

Example:

```txt
GET /company/:companyId/detail
```

Response may include:

```txt
company
summary
branches
```

But update routes should still follow the owner entity.

Example:

```txt
PATCH /company/:companyId
-> update company profile

PUT /company/:companyId/branches/information
-> create/update branch information

PATCH /company/:companyId/logo
-> update company logo
```

Do not put all update behavior into one large endpoint just because the detail response is combined.

Read response is for UI convenience.

Write route is for clear data ownership.

## &nbsp;

**11. Options endpoint**

Options endpoints are for select/dropdown/reference usage.

Keep response small and stable.

Example branch option response:

```json
{
  "value": "branch-uuid",
  "label": "Jakarta Head Office",
  "branchCode": "MTEN",
  "isHeadOffice": true,
  "locationLabel": "Jakarta, Indonesia"
}
```

Do not return duplicated fields.

Avoid this:

```json
{
  "value": "branch-uuid",
  "branchId": "branch-uuid",
  "label": "Jakarta Head Office",
  "displayName": "Jakarta Head Office",
  "isHeadOffice": true,
  "badgeLabel": "HQ"
}
```

Reason:

```txt
value already means id
label already means display name
badge can be derived from isHeadOffice
```

Frontend can render:

```ts
const badge = option.isHeadOffice ? 'HQ' : 'Branch'
```

## &nbsp;

**12. Controller responsibility**

Controller is only an HTTP adapter.

Controller should:

* read validated params/query/body
* read request context
* call service
* return response

Controller should not:

* run business validation
* access repository directly
* check membership manually
* contain ORM logic
* contain complex branching

Example:

```ts
async branchOptions(req: Request, res: Response) {
  const request = req as ExpressRequestWithContext
  const { params, query } = res.locals.validated

  const data = await this.service.branchOptions(params.companyId, query, request.context)

  return res.json(HttpResponse.success(req, data))
}
```

## &nbsp;

**13. Service responsibility**

Service owns business rules.

Service should:

* validate business rules
* run defensive scope checks
* decide what error should be thrown
* call repository
* create audit logs
* map result to response DTO

Example defensive guard:

```ts
this.assertCompanyScope({
  companyId,
  systemRole: context?.user.systemRole,
  execution: context?.execution,
})
```

Even if middleware already checks access, service may still protect scoped data.

This prevents mistakes when a route changes later.

## &nbsp;

**14. Repository responsibility**

Repository owns database access only.

Repository should:

* query database
* update database
* use transaction when needed
* return raw database result

Repository should not:

* check route permission
* know about Express request
* know about frontend behavior
* throw user-facing authorization decisions unless needed for low-level DB errors

Business validation belongs in service.

## &nbsp;

**15. Frontend consumption rule**

Frontend may send selected context to Gateway:

```txt
x-company-id
x-workspace-id
```

Frontend must not send trusted roles.

Do not trust frontend for:

```txt
systemRole
companyRole
workspaceRole
departmentId
serviceKey
```

Gateway and Access Service resolve trusted context.

Downstream services consume trusted headers from Gateway.

## &nbsp;

**16. Route design checklist**

Before adding a route, answer these questions:

```txt
1. Is this route platform-level, company-level, or workspace-level?
2. Does it need selected company context?
3. Does it need selected workspace context?
4. Does it have :companyId param?
5. Should enforceCompanyScope be true?
6. Can normal user access this without selected context?
7. Is this route for admin page, company page, or workspace runtime?
8. Is this a read route or write route?
9. Is the response for page detail or dropdown options?
10. Should this action be superuser only?
```

If the route is hard to explain, the route is probably too vague.

## &nbsp;

**17. Naming guideline**

Use clear route names.

Good:

```txt
GET    /company/:companyId/detail
GET    /company/:companyId/branches/options
PUT    /company/:companyId/branches/information
PATCH  /company/:companyId/logo
DELETE /company/:companyId/logo
DELETE /company/hard/:companyId
```

Avoid vague routes:

```txt
POST /company/action
PATCH /company/data
GET /company/all-info
POST /company/update-detail
```

Route name should show intent.

## &nbsp;

**18. Common mistakes**

**Mistake 1: Using `allowWithoutExecution` everywhere**

Do not do this.

Use it only when a route really can run without selected company/workspace.

Workspace runtime routes should usually not use it.



**Mistake 2: Missing `enforceCompanyScope`**

If route has `:companyId`, consider using:

```ts
enforceCompanyScope: true
```

Especially for update/delete/detail routes.



**Mistake 3: Returning too much data from options endpoint**

Options endpoint should be small.

Use detail endpoint for full data.



**Mistake 4: Mixing unrelated write behavior**

Do not update company profile, logo, branch, and status in one endpoint.

Separate by action and data ownership.



**Mistake 5: Assuming frontend is trusted**

Frontend is not trusted for authorization.

Frontend only sends selected context.

Gateway and Access Service resolve trusted context.

## &nbsp;

**19. Simple rule summary**

```txt
Platform route
-> systemRoles + allowWithoutExecution

Company route
-> systemRoles + companyRoles + enforceCompanyScope

Workspace route
-> systemRoles + companyRoles + workspaceRoles + enforceCompanyScope

Dangerous route
-> superuserOnly

Dropdown route
-> small response shape

Detail route
-> may return combined page data

Write route
-> update one clear business area
```

