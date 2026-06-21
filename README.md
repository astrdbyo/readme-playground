[← Back to Main README](../README.md)

# Docker & Server Operations

This document contains common Docker and server operation notes used for deployment and maintenance.

It covers:

```txt
Postgres Docker Compose sample
Server folder structure
Container recreation without rebuild
Nginx validation and reload
Nginx basic-auth password update
```

## 🐳 Postgres Docker Compose Sample

```yml
# infrastructure data layer; postgres server for production environment
# naming format: <org>-<layer>-<domain>-<service>-<env?>-<variant?>
# use the format to prevent orphan containers; deterministic stack identity
# purpose: production database server; private on app-network only

name: ics-infra-data-postgres-production

services:
  postgres:
    image: postgres:16-alpine
    container_name: ics-infra-data-postgres-production
    restart: unless-stopped

    # load production credentials; keep secrets out of compose file
    env_file:
      - .env.production

    # persistent data volume; do not delete
    volumes:
      - ./data:/var/lib/postgresql/data
      - ./backup:/backup

    networks:
      - app-network

networks:
  app-network:
    external: true
```

Run the stack:

```sh
docker compose --env-file .env.production up -d
```

## 📁 Server Folder Structure for Applications

```sh
/opt/docker
├── applications-layer
│   ├── intcrews-cdn
│   └── intcrews-hub
│       ├── api
│       │   ├── prod
│       │   └── staging
│       └── web
│           ├── prod
│           └── staging
```

## 🔁 Recreate Container With Existing Image

Use this when you want to recreate a container using an existing image without rebuilding.

List available images:

```sh
docker images | grep ops-automation
```

Choose the image tag you want to use:

```sh
export IMAGE_TAG=sha-3dfd762
```

Recreate the API container:

```sh
docker compose up -d --no-deps --force-recreate api
```

For staging environment:

```sh
docker compose --env-file .env.staging up -d --no-deps --force-recreate api
```

## 🌐 Nginx Validation and Reload

Validate Nginx configuration:

```sh
docker exec -it ics-infra-edge-nginx nginx -t
```

Reload Nginx:

```sh
docker exec -it ics-infra-edge-nginx nginx -s reload
```
## 🔐 Update Nginx Basic Auth Password

Go to the reverse proxy folder:

```sh
cd /opt/docker/infrastructures-layer/edge/reverse-proxy
```

Check available auth files:

```sh
ls nginx/nginx-auth
```

Update or revoke Nginx basic-auth password:

```sh
docker run --rm \
  -v $(pwd)/nginx/nginx-auth:/output \
  httpd:2.4-alpine \
  sh -c "htpasswd -bB /output/staging-aii-hub-api.htpasswd qauser NewPassword123"
```

## ✅ Quick Checklist

Before running Docker or Nginx operations, verify:

```txt
1. You are in the correct server folder.
2. You are using the correct environment file.
3. You are targeting the correct service name.
4. You already checked the current running containers.
5. You already validated Nginx config before reload.
```

Useful commands:

```sh
docker ps
docker compose ps
docker images
```
