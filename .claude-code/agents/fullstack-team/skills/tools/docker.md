# Docker

**Category:** Tools
**Estimated Tokens:** 400
**MCP Server:** `@modelcontextprotocol/server-docker`
**Official Docs:** https://docs.docker.com

## Quick Reference (50 tokens)

Docker containerizes applications for consistent deployment across environments. Use for local development, staging, and production with multi-stage builds for optimization.

## Overview (80 tokens)

Docker provides:
- **Environment consistency** (dev = staging = prod)
- **Dependency isolation** (no version conflicts)
- **Easy scaling** with container orchestration
- **Fast deployment** with image caching
- **Resource efficiency** vs VMs

**When to use:** All projects for local dev and production deployment.

**MCP Integration:** Install `@modelcontextprotocol/server-docker` for container management via Claude Code.

## MCP Server Setup (60 tokens)

```json
// Add to Claude Code MCP settings
{
  "mcpServers": {
    "docker": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-docker"
      ]
    }
  }
}
```

Requires Docker Desktop or Docker Engine running.

## Key Concepts (100 tokens)

### 1. Multi-Stage Builds

Optimize image size by separating build and runtime stages:

```dockerfile
# Build stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Runtime stage
FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./

EXPOSE 3000
CMD ["npm", "start"]
```

### 2. Docker Compose for Services

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgresql://postgres:postgres@db:5432/myapp
      REDIS_URL: redis://redis:6379
    depends_on:
      - db
      - redis

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: myapp
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

### 3. .dockerignore

```
node_modules
.next
.git
.env*.local
*.log
.DS_Store
README.md
```

## Common Patterns (150 tokens)

### Pattern 1: Next.js Production Dockerfile

```dockerfile
FROM node:20-alpine AS base

# Dependencies
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app
COPY package*.json ./
RUN npm ci

# Builder
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
ENV NEXT_TELEMETRY_DISABLED 1
RUN npm run build

# Runner
FROM base AS runner
WORKDIR /app
ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs
EXPOSE 3000
ENV PORT 3000

CMD ["node", "server.js"]
```

```js
// next.config.js - Enable standalone output
module.exports = {
  output: 'standalone',
}
```

### Pattern 2: Development Workflow

```bash
# Build and run locally
docker-compose up --build

# Run in background
docker-compose up -d

# View logs
docker-compose logs -f app

# Execute commands in container
docker-compose exec app npm run prisma:migrate

# Stop and remove containers
docker-compose down

# Remove volumes (reset database)
docker-compose down -v
```

### Pattern 3: Database Migrations in Docker

```dockerfile
# Add to Dockerfile for migrations
FROM builder AS migrator
RUN npx prisma generate
CMD ["npx", "prisma", "migrate", "deploy"]
```

```yaml
# docker-compose.yml - Run migrations before app starts
services:
  migrate:
    build:
      context: .
      target: migrator
    environment:
      DATABASE_URL: postgresql://postgres:postgres@db:5432/myapp
    depends_on:
      - db
    command: npx prisma migrate deploy

  app:
    depends_on:
      migrate:
        condition: service_completed_successfully
```

## Best Practices (50 tokens)

**DO:**
- ✅ Use multi-stage builds for smaller images
- ✅ Use .dockerignore to exclude unnecessary files
- ✅ Run containers as non-root user
- ✅ Use specific image tags (not `latest`)
- ✅ Leverage build cache (order COPY commands correctly)
- ✅ Use Docker Compose for multi-container apps

**DON'T:**
- ❌ Copy node_modules into image
- ❌ Store secrets in Dockerfile
- ❌ Run as root user
- ❌ Use `latest` tag in production
- ❌ Skip .dockerignore

## Troubleshooting (50 tokens)

**Build Failures:**
```bash
# Clear build cache
docker builder prune -a

# Build without cache
docker build --no-cache .
```

**Container Won't Start:**
```bash
# Check logs
docker logs <container-id>

# Inspect container
docker inspect <container-id>

# Run interactively
docker run -it <image> /bin/sh
```

**Port Already in Use:**
```bash
# Find process using port
lsof -i :3000

# Use different port
docker run -p 3001:3000 <image>
```

**Database Connection Issues:**
```yaml
# Use service name as hostname
DATABASE_URL: postgresql://user:pass@db:5432/myapp
# NOT localhost (localhost = container itself)
```

## Related Skills
- `tools/kubernetes.md` - Container orchestration
- `patterns/ci-cd.md` - Docker in pipelines

## Tags
docker, containers, deployment, devops, docker-compose
