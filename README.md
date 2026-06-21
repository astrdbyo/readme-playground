[← Back to Main README](../README.md)

# CI/CD Workflow Guide

This document explains how GitHub Actions workflows are used to validate code, build Docker images, deploy to staging, promote to production, switch traffic, and support rollback.

The main idea:

```txt
Build Once
Tag Clearly
Deploy Explicitly
Rollback Safely
```

Production deployment is manual by design.

## 1. Purpose

This repository uses GitHub Actions to support a predictable delivery flow:

```txt
validate code before merge
build Docker images
publish images to GHCR
deploy specific image tags to staging
promote verified images to production
switch production traffic using Nginx
rollback to known-good images when needed
```

The workflows are also connected to release and versioning semantics.

```txt
CI creates the image tag
deployment uses the image tag
rollback returns to an older image tag
```

Without clear image tags, rollback becomes guesswork.

## 2. High-Level Workflow

```txt
Developer
  │
  ▼
Pull Request
  │
  ▼
01 - Pre-Merge Code Validation Checks
  │
  ▼
Merge to staging
  │
  ▼
02 - Build & Publish Images to GHCR
  │
  ▼
Manual staging deploy
  │
  ├── 03 - Deploy API to Staging
  └── 04 - Deploy Web to Staging
  │
  ▼
Manual promote to production
  │
  ├── 05 - Promote API to Production
  └── 06 - Promote Web to Production
  │
  ▼
Switch production traffic
  │
  └── 96 - Switch Production Traffic via Nginx
  │
  ▼
Rollback when needed
  │
  ├── 99 - Rollback API Production
  └── 100 - Rollback Web Production
```

## 3. CI/CD and Image Tag Semantics

Workflow `02` builds Docker images and publishes them to GHCR.

Each image gets two types of tags:

```txt
moving tag     -> staging
immutable tag  -> sha-xxxxxxx
```

Example:

```txt
ghcr.io/<owner>/<repo>/api:staging
ghcr.io/<owner>/<repo>/api:sha-3dfd762

ghcr.io/<owner>/<repo>/web:staging
ghcr.io/<owner>/<repo>/web:sha-3dfd762
```

Meaning:

```txt
staging
-> moving tag
-> points to latest staging build

sha-3dfd762
-> immutable tag
-> points to a specific commit
-> useful for deployment, promotion, and rollback
```

Use SHA tags when deploying to staging or production if you need traceability.

Avoid using `latest` for staging or production.

## 4. Workflow List

| No  | Workflow                         | Purpose                                         | Trigger                         |
| --- | -------------------------------- | ----------------------------------------------- | ------------------------------- |
| 01  | Pre-Merge Code Validation Checks | Validate code before merge                      | Pull request / protected branch |
| 02  | Build & Publish Images to GHCR   | Build and push Docker images                    | Push to `staging`               |
| 03  | Deploy API to Staging            | Deploy specific API image to staging            | Manual                          |
| 04  | Deploy Web to Staging            | Deploy specific Web image to staging            | Manual                          |
| 05  | Promote API to Production        | Deploy verified API image to production slot    | Manual                          |
| 06  | Promote Web to Production        | Deploy verified Web image to production slot    | Manual                          |
| 96  | Switch Production Traffic        | Switch active production slot through Nginx     | Manual                          |
| 98  | Cleanup Staging Environment      | Stop staging containers and clean unused images | Manual                          |
| 99  | Rollback API Production          | Rollback API to older image tag                 | Manual                          |
| 100 | Rollback Web Production          | Rollback Web to older image tag                 | Manual                          |

## 5. Workflow Responsibilities

### 01 - Pre-Merge Code Validation Checks

Purpose:

```txt
make sure only valid code is merged
```

Usually runs:

```txt
install dependencies
lint
typecheck
test
build
```

Result:

```txt
fail -> merge should be blocked
pass -> code is safe to merge
```

### 02 - Build & Publish Images to GHCR

Purpose:

```txt
create Docker images that can be deployed later
```

Trigger:

```txt
push to staging branch
```

What it does:

```txt
checkout repository
setup Docker Buildx
login to GHCR
generate short SHA tag
build Web image
build API image
push staging tag
push SHA tag
```

Example output:

```txt
api:staging
api:sha-a1b2c3d
web:staging
web:sha-a1b2c3d
```

The SHA tag is the important part for rollback.

### 03 - Deploy API to Staging

Purpose:

```txt
deploy a specific API image to staging
```

Trigger:

```txt
manual workflow_dispatch
```

Input:

```txt
image_sha
```

Example:

```txt
sha-a1b2c3d
```

What it does:

```txt
connect to server through Tailscale
login to GHCR
go to staging API compose folder
update IMAGE_TAG in .env.staging
pull selected image
restart API container
cleanup unused images
```

### 04 - Deploy Web to Staging

Same idea as workflow `03`, but for Web.

### 05 - Promote API to Production

Purpose:

```txt
deploy a verified API image into a production blue/green slot
```

Trigger:

```txt
manual workflow_dispatch
```

Inputs:

```txt
image_sha
slot: blue or green
```

What it does:

```txt
connect to production server
login to GHCR
go to production API compose folder
update IMAGE_TAG_BLUE or IMAGE_TAG_GREEN
pull selected image
start selected slot container
cleanup unused images
```

Important:

```txt
promotion deploys the container
traffic switching is handled separately by workflow 96
```

### 06 - Promote Web to Production

Same idea as workflow `05`, but for Web.

### 96 - Switch Production Traffic via Nginx

Purpose:

```txt
switch live traffic to blue or green
```

Trigger:

```txt
manual workflow_dispatch
```

Input:

```txt
slot: blue or green
```

What it does:

```txt
connect to production server
write active slot config
validate Nginx config
reload Nginx
```

This separates deployment from traffic switching.

That separation is important because a container can be deployed and checked before receiving live traffic.

### 98 - Cleanup Staging Environment

Purpose:

```txt
clean staging containers and unused images safely
```

This should not touch production.

### 99 - Rollback API Production

Purpose:

```txt
rollback API production to a previous known-good image tag
```

Rollback uses the same deployment mechanism as promotion, but with an older SHA.

Example:

```txt
current broken image: sha-bad1234
previous stable image: sha-good567
rollback target: sha-good567
```

### 100 - Rollback Web Production

Same idea as workflow `99`, but for Web.

## 6. Blue/Green Deployment Support

The current workflow and Docker Compose structure already support a blue/green deployment model.

Concept:

```txt
blue  -> one production slot
green -> another production slot
```

Example:

```txt
api-blue  -> sha-oldgood
api-green -> sha-newcandidate
```

The purpose of blue/green deployment is:

```txt
deploy new version into inactive slot
verify the new slot
switch traffic when ready
rollback by switching traffic back
```

In this repository:

```txt
05 / 06 deploy image to selected production slot
96 switches live traffic through Nginx
99 / 100 rollback by deploying older image tag
```

Important note:

```txt
full blue/green requires clear traffic switching
```

If the promotion workflow stops the opposite slot immediately, the setup is closer to slot-based deployment with blue/green naming.

For full zero-downtime blue/green later:

```txt
keep old slot running
start new slot
verify new slot
switch traffic using workflow 96
stop old slot only after confirmation
```

The current structure is already a good foundation. The operational procedure can be improved later when monitoring, resources, and team readiness are stronger.

## 7. Staging vs Production Behavior

| Environment | Behavior                                        |
| ----------- | ----------------------------------------------- |
| Staging     | Used for integration testing                    |
| Staging     | Can deploy latest staging image or specific SHA |
| Staging     | Safe place to verify image before production    |
| Production  | Manual only                                     |
| Production  | Should use verified SHA                         |
| Production  | Should not deploy automatically from push       |
| Production  | Traffic switching should be explicit            |

Simple rule:

```txt
staging proves the image
production runs the approved image
```

## 8. Deployment and Rollback Relationship

Deployment and rollback use the same basic mechanism:

```txt
choose image tag
update env value
pull image
recreate container
verify service
```

Deployment usually uses a new SHA.

Rollback uses an older known-good SHA.

Example:

```sh
export IMAGE_TAG=sha-good567
docker compose --env-file .env.production up -d --no-deps --force-recreate api
```

This is why workflow `02` must always push immutable SHA tags.

## 9. Key Principles

```txt
CI is not the same as deployment
build once, deploy many times
production deployment is manual
SHA is the source of truth for build identity
staging is for verification
production is for approved images
blue/green separates deployment from traffic
Nginx controls live traffic switching
rollback requires known-good image tags
one workflow should have one responsibility
```

## 10. When to Use Which Workflow

| Situation                            | Workflow |
| ------------------------------------ | -------- |
| Validate pull request                | 01       |
| Build Docker images                  | 02       |
| Deploy API to staging                | 03       |
| Deploy Web to staging                | 04       |
| Promote API image to production slot | 05       |
| Promote Web image to production slot | 06       |
| Switch live production traffic       | 96       |
| Cleanup staging                      | 98       |
| Rollback API production              | 99       |
| Rollback Web production              | 100      |

## 11. Recommended Operator Checklist

Before deploying:

```txt
confirm image SHA
confirm target environment
confirm target service
confirm slot if production
confirm database migration risk
confirm rollback image
```

After deploying:

```txt
check container status
check logs
check health endpoint
check Nginx config if traffic switch is involved
confirm application behavior
```

Useful commands:

```sh
docker ps
docker compose ps
docker compose logs -f
docker images
```

## 12. Quick Summary

```txt
workflow 01 protects code quality
workflow 02 creates image artifacts
workflow 03 and 04 deploy staging
workflow 05 and 06 promote to production slots
workflow 96 switches production traffic
workflow 99 and 100 rollback using older SHA
image SHA connects CI/CD with versioning
blue/green is already supported structurally
full blue/green operation can be improved over time
```
