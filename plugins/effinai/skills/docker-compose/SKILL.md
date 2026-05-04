---
name: docker-compose
description: "Generate Docker Compose configurations for multi-service applications"
version: "1.0.0"
tags: [engineering, docker, devops, containerization]
createdBy: "built-in"
status: "active"
---

# Docker Compose Generator

## Activation
When the user asks to dockerize an application, create docker-compose files, or containerize services.

## Workflow

### Step 1: Analyze the Application
1. Read the project structure to identify services
2. Check for existing Dockerfile(s)
3. Identify the tech stack (Node, Python, Go, etc.)
4. Determine dependencies (database, cache, queue, etc.)

### Step 2: Generate Dockerfile (if missing)

**Node.js pattern:**
```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --production=false
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./
USER node
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

Best practices:
- Multi-stage builds to reduce image size
- Use alpine base images
- Copy package files first for layer caching
- Run as non-root user
- Use .dockerignore to exclude node_modules, .git, etc.

### Step 3: Generate docker-compose.yml

```yaml
version: "3.9"
services:
  app:
    build: .
    ports:
      - "${PORT:-3000}:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:pass@db:5432/appdb
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  db:
    image: postgres:16-alpine
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: appdb
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    volumes:
      - redisdata:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]

volumes:
  pgdata:
  redisdata:
```

### Step 4: Generate Supporting Files

1. **.dockerignore** — Exclude unnecessary files
2. **docker-compose.override.yml** — Development overrides (volumes, debug ports)
3. **.env.example** — Document required environment variables

### Step 5: Development vs Production

**Development override (docker-compose.override.yml):**
- Mount source code as volume for hot reload
- Expose debug ports
- Use development environment variables
- Add admin tools (pgAdmin, Redis Commander)

**Production considerations:**
- No source mounts
- Proper secrets management (Docker secrets or external vault)
- Resource limits (memory, CPU)
- Logging driver configuration
- Network isolation

## Rules
- Always use health checks for service dependencies
- Never hardcode secrets — use environment variables
- Use named volumes for persistent data
- Pin image versions (not `latest`)
- Generate .dockerignore alongside Dockerfile
- Include restart policies for production
- Use depends_on with condition: service_healthy
