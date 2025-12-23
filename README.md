## CI / CD Workflow Guide

This document explains **why these workflows exist**, **what each one does**, and **how to use them**.  
It is written for future reference, so you do not need to re-learn the entire CI/CD concept again.

### Purpose

This repository uses **GitHub Actions** to automate the full software delivery lifecycle:

1. **Validate code before merge**
2. **Build Docker images**
3. **Deploy safely to staging**
4. **Deploy explicitly to production**
5. **Create a production release record**

Each step has **one responsibility only**.  
Nothing is automatic in production by design.

### Workflow Overview

```sh
Developer → Pull Request
↓
01 - Pre-Merge Code Validation Checks
↓
Merge to staging
↓
02 - Build & Publish Image to GHCR (staging + SHA)
↓
03 - Deploy to Staging
↓
Manual approval
↓
04 - Deploy to Production
↓
05 - Create Production Release
```

> [!TIP]
> A quick tip to help you understand and work with this project more easily.

### 01 – Pre-Merge Code Validation Checks

### Purpose
Ensure **only correct code** is merged.

### Trigger
- Pull Request to `main`, `staging`, or `development`
- Direct push to protected branches

### What it does
1. Checkout code
2. Install dependencies
3. Lint source code
4. Type-check TypeScript
5. Run automated tests
6. Build the application

### Result
- ❌ Fail → Merge is blocked
- ✅ Pass → Safe to merge

### 02 – Build & Publish Image to GHCR (Staging + SHA)

### Purpose
Create **immutable Docker images** for staging and production.

### Trigger
- Push to `staging` branch

### What it does
1. Checkout code
2. Setup Docker Buildx
3. Login to GitHub Container Registry (GHCR)
4. Build Docker image
5. Push two tags:
   - `:staging` → for staging runtime
   - `:sha-<commit>` → for traceability

### Result
- Image is stored in GHCR
- Every build is uniquely identifiable

### 03 – Deploy to Staging

### Purpose
Run the **exact built image** in a staging environment.

### Trigger
- Manual (`workflow_dispatch`)

### What it does
1. SSH into staging server
2. Pull `:staging` image from GHCR
3. Restart containers using Docker Compose
4. Clean unused images

### Result
- Staging always reflects latest tested code
- Safe environment for QA and verification


### 04 – Deploy to Production Environment

### Purpose
Deploy a **specific, verified image** to production.

### Trigger
- Manual (`workflow_dispatch`)

### Required Input
- `image_sha` (example: `sha-a1b2c3d`)

### What it does
1. SSH into production server
2. Pull the exact SHA image from GHCR
3. Deploy using Docker Compose
4. Clean unused images

### Why SHA matters
- Full traceability
- Easy rollback
- No “latest” surprises

### 05 – Create Production Release

### Purpose
Record **what was released** and **when**.

### Trigger
- Manual (`workflow_dispatch`)

### Required Inputs
- `version` (example: `v1.2.0`)
- `image_sha` (deployed SHA)

### What it does
1. Create a GitHub Release
2. Tag the repository
3. Store release notes
4. Link deployed image SHA

### Result
- Clear audit trail
- Human-readable release history

---

## Key Principles

- **CI ≠ Deployment**
- **Build once, deploy many times**
- **Production is always manual**
- **SHA = truth**
- **One workflow = one responsibility**

---

## When to Use Which Workflow

| Situation | Workflow |
|--------|--------|
| Validate PR | 01 |
| Prepare artifact | 02 |
| Test runtime | 03 |
| Go live | 04 |
| Record release | 05 |

---

## Final Notes

This setup is:
- Beginner-friendly
- Production-safe
- Industry-aligned
- Easy to maintain
