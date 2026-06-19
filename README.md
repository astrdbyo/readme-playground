<!-- <div align="center">

<!-- <p align="center">
  <img src="docs/assets/header.png" alt="Banner" />
</p> -->

# Intcrews Hub Monorepo

A production-ready **enterprise monorepo** containing both frontend and backend applications, designed for internal engineering teams and scalable long-term usage.  
Structured for clarity, maintainability, and fast onboarding.  -->

[← Back to Main README](../README.md)


## Inventory Service

### Overview

Inventory Service is a **downstream business service** in Intcrews Hub for managing company asset lifecycle. It consumes trusted execution context from API Gateway and Access Service.

This service is designed for multiple internal stakeholders:

* IT Department
* General Affairs / GA
* Finance / Accounting

Inventory Service is the **single source of truth for asset lifecycle management**. Company structure, user identity, workspace permissions, and branch master data are managed by other services.
The service is built to support future business growth while maintaining strong data control and a simple user experience with minimal learning curve.

## Relationship With Access Service

Access Service owns platform identity, company structure, workspace access, and execution context.

Access Service owns:

```txt
UserAccount
UserProfile
Company
CompanyBranch
CompanyMember
Department
JobTitle
Workspace
WorkspaceMember
WorkspaceModuleRegistry
Workspace ownership profiles
```

Inventory Service references Access-owned entities using UUID snapshots only.
There is no cross-service relational dependency and no cross-database foreign key.

Common Access references used by Inventory:

```txt
companyId
workspaceId
ownerDepartmentId
userAccountId
branchId
employeeId
```

These values are resolved from trusted execution context or fetched from Access Service options.

---

## Data Ownership Boundary

### Access Service owns

```txt
company master
company branch master
workspace access
workspace routing
department ownership
employee identity bridge
user identity
role context
```

### Inventory Service owns

```txt
asset master data
asset lifecycle
asset location detail
asset assignment
asset movement
asset service log
asset warranty
asset procurement
asset finance
asset depreciation
asset document reference
brand
vendor
floor
zone
floor plan
asset placement
```

---

## Workspace Execution Context

Inventory Service is workspace-centric.

Most Inventory business records are scoped by:

```txt
companyId
workspaceId
ownerDepartmentId
```

Workspace determines data visibility and execution boundary.

Branch is not an auth boundary.

```txt
Company = tenant / legal entity
Workspace = execution context / access boundary / app container
Branch = company location master / operational metadata
```

Example:

```txt
Company: Intcrews
Workspace: IT Asset Inventory HQ
Workspace: IT Asset Inventory Bali
Workspace: HR Management HQ
Workspace: HR Management Bali
```

Multiple workspaces can use the same module.

Each workspace still has isolated downstream data through `workspaceId`.

---

## Branch Strategy

Branch is owned by Access Service as `CompanyBranch`.

Inventory Service does not own Branch.

Inventory stores branch reference as:

```txt
branchId
branchLabel
```

`branchId` comes from Access Service.

`branchLabel` is stored as a display snapshot.

This keeps historical Inventory records readable even if branch data changes later in Access Service.

Branch is used for:

```txt
asset location
filtering
reporting
floor mapping
distribution map
operational grouping
```

Branch is not used for:

```txt
authentication
authorization
workspace membership
role resolution
```

Workspace remains the access and data visibility boundary.

---

## Multi-Stakeholder Design

Inventory Service supports one shared asset lifecycle with multiple stakeholder views.

IT needs:

```txt
assignment
maintenance
service log
QR code
scanner
distribution map
troubleshooting history
```

GA needs:

```txt
furniture tracking
quantity onboarding
location visibility
distribution management
branch/floor/zone tracking
```

Finance needs:

```txt
purchase records
depreciation
audit trail
tax compliance
monthly reporting
```

The system does not split:

```txt
IT Inventory
GA Inventory
Finance Inventory
```

into different systems.

Instead:

```txt
one asset lifecycle
multiple stakeholder views
```

---

## Asset Modeling Principle

Physical assets are tracked per item.

Example:

```txt
Laptop x20
```

becomes:

```txt
20 asset records
20 asset identities
20 histories
20 possible QR labels
```

However, onboarding UX may support quantity-based input.

Example:

```txt
Guest Chair
Qty = 40
```

The system can expand it into 40 physical asset records.

This enables:

```txt
QR generation
maintenance history
assignment tracking
movement tracking
disposal tracking
finance valuation
```

while keeping onboarding simple.

---

## Asset Identification

Asset code is system generated.

Format:

```txt
[DEPARTMENT_CODE][COMPANY_CODE][YEAR]-[CATEGORY]-[SEQUENCE]
```

Example:

```txt
IT012024-NTB-000123
```

Components:

```txt
IT     = owner department
01     = company code
2024   = acquisition year
NTB    = asset code prefix / category-like prefix
000123 = running sequence
```

Asset code is used as the primary operational identity shown to users.

---

## QR Code and Asset Scanner

QR Code image does not need to be stored in the database.

QR can be generated from asset identity.

For MVP, QR payload can remain as `assetCode`.

Example sticker:

```txt
PROPERTY OF IT DEPARTMENT
[QR CODE]
IT012024-NTB-000123
```

QR must never become the source of truth.

The source of truth remains the Asset record.

Recommended scanner flow:

```txt
User opens Global Asset Scanner shortcut
User must be logged in
User scans QR code
Scanner reads assetCode
Backend searches asset by assetCode
Backend returns limited quick summary
Full detail requires workspace context and workspace permission
```

Global scanner may live outside selected workspace.

This allows quick lookup without forcing the user to select company and workspace first.

However, full asset detail must still require workspace access validation.

Quick summary may show:

```txt
assetCode
asset name
status
condition
specification
branch / location summary
remarks
```

Quick summary should not expose sensitive data such as:

```txt
purchase cost
depreciation data
invoice document
assigned employee detail
warranty cost
service cost
sensitive serial detail
```

If the same assetCode exists in multiple company/workspace scopes, scanner should return multiple limited results.

Remarks should help distinguish results:

```txt
Company / Workspace / Branch
```

---

## Location Model

Inventory Service owns asset operational location detail.

Location model:

```txt
Access CompanyBranch
└── Inventory Floor
    └── Inventory Zone / Room
        └── Inventory FloorPlan
            └── Inventory AssetPlacement
```

### CompanyBranch

Owned by Access Service.

Used by Inventory as:

```txt
branchId
branchLabel
```

### Floor

Owned by Inventory Service.

Floor belongs to:

```txt
companyId
workspaceId
branchId
```

Floor represents operational floor data inside a branch.

### Zone

Owned by Inventory Service.

Zone belongs to Floor.

Zone can represent:

```txt
room
area
rack
pantry
meeting room
server room
storage
```

### FloorPlan

Owned by Inventory Service.

FloorPlan stores uploaded visual layout files for distribution map.

It supports future features such as:

```txt
office layout
asset pinning
visual placement
drag and drop position
distribution monitoring
```

### AssetPlacement

Owned by Inventory Service.

AssetPlacement stores current physical location snapshot.

It can store:

```txt
branchId
branchLabel
floorId
floorPlanId
zoneId
mapX
mapY
rotation
displayLabel
note
```

Historical movement is stored separately in `AssetMovement`.

---

## Service Layer Location Rules

Because Branch is owned by Access Service, Inventory Service must validate branch references at service layer.

### FloorService must validate

```txt
branchId exists in Access Service
branchId belongs to current company
branch is active
```

### FloorPlanService must validate

```txt
branchId exists in Access Service
floorId belongs to same companyId
floorId belongs to same workspaceId
floorId belongs to same branchId
only one active floor plan per floor is allowed
```

### ZoneService must validate

```txt
floorId belongs to current company
floorId belongs to current workspace
branch scope is derived from floor.branchId
```

### AssetPlacementService must validate

```txt
branchId exists in Access Service
branchId belongs to current company
branch is active
floorId belongs to same branchId when floorId is provided
floorPlanId belongs to same floorId when floorPlanId is provided
zoneId belongs to same floorId when zoneId is provided
```

### AssetMovementService must validate

```txt
fromBranchId and toBranchId exist in Access Service when provided
fromFloorId and toFloorId match their branch references when provided
fromZoneId and toZoneId match their floor references when provided
labels are stored as immutable historical snapshots
```

---

## Asset Classification

Inventory classification uses this hierarchy:

```txt
AssetCategory
└── AssetType
    └── AssetVariant
```

### AssetCategory

Global baseline category.

Tied to `AssetProfileType`.

Examples:

```txt
IT Equipment
Furniture
Facility Equipment
Vehicle
```

### AssetType

Workspace-scoped type.

Examples:

```txt
Computer
Chair
Air Conditioner
Vehicle
```

### AssetVariant

Workspace-scoped practical asset variant.

Examples:

```txt
Laptop
Desktop PC
Office Chair
Split AC
Car
```

AssetVariant can define asset code prefix and optional depreciation policy override.

---

## Asset Model Catalog

`AssetModelCatalog` supports cleaner user input and better suggestions.

It stores common brand/model combinations.

Examples:

```txt
Lenovo ThinkPad T14
Apple MacBook Air M2
Daikin FTKC25
```

It helps reduce repeated manual typing and improves data consistency.

---

## Capability Profiles

Asset uses polymorphic profile tables to avoid nullable fields in the main Asset table.

Supported profiles:

```txt
AssetITProfile
AssetFurnitureProfile
AssetFacilityProfile
AssetVehicleProfile
```

Example IT profile fields:

```txt
brand
model name
serial number
specification
operating system
```

Example vehicle profile fields:

```txt
brand
model name
plate number
chassis number
engine number
fuel type
```

---

## Brand vs Vendor

Brand and Vendor are different concepts.

### Brand

Brand is the product manufacturer or principal.

Examples:

```txt
Lenovo
Apple
Dell
IKEA
Toyota
Daikin
```

Brand is used in asset profiles and model catalog suggestions.

### Vendor

Vendor is the seller, distributor, service provider, or warranty provider.

Examples:

```txt
PT ABC Komputer
Bhinneka
Metrodata
AC service vendor
Vehicle workshop
```

Vendor is used in:

```txt
AssetProcurement
AssetServiceLog
AssetWarranty
```

Vendor is not assignment.

---

## Assignment Model

Assignment represents internal responsibility or custody.

It answers:

```txt
Who is internally responsible for this asset?
```

Supported assignment types:

```txt
employee
department
room
other
```

Vendor is not an assignment type.

If an asset is sent to a vendor for service, assignment does not automatically change to vendor.

Example:

```txt
Laptop assigned to Budi
Laptop is sent to service vendor for 3 days
Assignment remains Budi
Status becomes maintenance
ServiceLog stores vendor information
Movement stores physical movement history
```

For `other` assignment type:

```txt
assignedToId = null
assignedToLabel = free text
```

Example:

```txt
assignmentType = other
assignedToLabel = Security Lobby
```

---

## Movement Model

AssetMovement stores historical physical movement.

It answers:

```txt
Where did the asset move from and to?
```

Examples:

```txt
procurement arrival
assignment handover
return asset
transfer location
maintenance movement
disposal
retirement
```

Movement stores display snapshots:

```txt
fromLabel
toLabel
```

This keeps history readable even if location data changes later.

---

## Service and Maintenance

AssetServiceLog stores service and maintenance history.

It answers:

```txt
What happened to the asset during service or maintenance?
```

It can store:

```txt
status
vendorId
vendorName
issueDescription
actionTaken
serviceDate
completedAt
cost
note
```

Service to vendor does not mean assignment changes to vendor.

---

## Warranty

AssetWarranty stores warranty information.

It can reference Vendor because warranty claim may go through seller, distributor, or service provider.

It can store:

```txt
vendorId
vendorName
warrantyType
startDate
endDate
note
```

---

## Procurement

AssetProcurement stores purchase traceability.

It can reference Vendor because purchase source is operationally important.

It can store:

```txt
vendorId
vendorName
requisitionNumber
quotationNumber
purchaseOrderNumber
invoiceNumber
taxInvoiceNumber
purchaseDate
note
```

One procurement record can contain many asset items through `AssetProcurementItem`.

---

## Finance and Depreciation

Finance remains the owner of accounting truth.

IT and GA are operational owners.

### DepreciationPolicy

DepreciationPolicy is a finance formula master.

It is not tied strictly to category.

It can be selected by asset finance data.

Example Indonesia tax grouping:

```txt
Group 1 → 4 years
Group 2 → 8 years
Group 3 → 16 years
Group 4 → 20 years
```

### AssetFinance

AssetFinance stores accounting-related asset data.

It can store:

```txt
purchaseDate
purchaseCost
residualValue
depreciationStartDate
depreciationPolicyId
isCapitalized
note
```

### AssetDepreciation

AssetDepreciation stores monthly depreciation results.

It is generated from finance calculation logic.

---

## Historical Tracking

Current state is stored directly on Asset for fast query.

Examples:

```txt
status
condition
current placement
current finance data
```

Historical changes are stored separately.

Examples:

```txt
AssetMovement
AssetAssignment
AssetStatusHistory
AssetServiceLog
AssetDepreciation
AuditLog
```

History must not be overwritten.

New lifecycle events should create new records or update lifecycle-specific records intentionally.

---

## Document Strategy

Inventory Service does not own file storage.

AssetDocument stores references to Drive Service.

Supported document examples:

```txt
requisition form
quotation
purchase order
invoice
tax invoice
receiving form
service invoice
handover form
disposal document
asset photo
warranty card
repair invoice
```

Inventory stores only document metadata and Drive references.

---

## Mandatory Field Principle

Only truly essential fields are mandatory.

Example:

```txt
Furniture:
serial number = optional
warranty = optional

Laptop:
serial number = recommended
```

Goal:

```txt
reduce operational pain
preserve data quality
support historical migration
```

---

## Audit Log

Inventory Service has downstream-scoped AuditLog.

It stores immutable event snapshots for traceability.

Downstream AuditLog should not contain Access Service relations.

Access Service owns canonical user and company relations.

Inventory audit stores IDs and labels as snapshots.

---

## Security Rules

Inventory Service should trust only gateway-injected context.

Inventory Service should not trust these values from frontend body for scoped writes:

```txt
companyId
workspaceId
ownerDepartmentId
userAccountId
```

Scoped values must come from trusted request context.

Branch is not a security boundary.

Workspace is the access and data visibility boundary.

---

## Module Map

### Master Data

```txt
AssetCategory
AssetType
AssetVariant
AssetModelCatalog
Brand
Vendor
DepreciationPolicy
```

### Execution Core

```txt
Asset
AssetBatch
AssetCodeSequence
```

### Capability Profiles

```txt
AssetITProfile
AssetFurnitureProfile
AssetFacilityProfile
AssetVehicleProfile
```

### Location

```txt
Floor
Zone
FloorPlan
AssetPlacement
```

Branch is not an Inventory model.

Branch is owned by Access Service as CompanyBranch.

### Lifecycle

```txt
AssetAssignment
AssetMovement
AssetServiceLog
AssetStatusHistory
```

### Finance / Procurement / Document

```txt
AssetFinance
AssetDepreciation
AssetProcurement
AssetProcurementItem
AssetDocument
AssetWarranty
```

### Audit

```txt
AuditLog
```

---

## Migration Notes

Recommended migration order:

```txt
1. Finalize Access Service CompanyBranch schema.
2. Run Access prisma validate.
3. Migrate Access Service.
4. Remove Branch model from Inventory Service.
5. Store branchId + branchLabel in Inventory location models.
6. Run Inventory prisma validate.
7. Migrate Inventory Service.
8. Regenerate Prisma clients.
9. Adjust repositories and services affected by Branch removal.
10. Add Access branch options endpoint.
11. Consume branch options in Inventory UI/service flow.
```

---

## Future Features

Schema intentionally supports:

```txt
QR code
global asset scanner
office map visualization
distribution map
asset pinning
predictive maintenance
asset automation
cross-department collaboration
bulk import migration
AI reporting
monthly depreciation
procurement traceability
```

without requiring major schema rewrite.

---

## Final Boundary Summary

```txt
Access Service:
- user
- company
- company branch
- department
- job title
- workspace
- workspace access
- workspace module routing

Inventory Service:
- asset
- asset master data
- vendor
- brand
- floor
- zone
- floor plan
- placement
- assignment
- movement
- service
- warranty
- procurement
- finance
- depreciation
- document reference
```

Final rule:

```txt
Access owns company structure and company location master.
Inventory owns asset operational data.
Workspace remains the access and data visibility boundary.
Branch is operational location metadata, not auth boundary.
```
