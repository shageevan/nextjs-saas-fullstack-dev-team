# PostgreSQL

**Category:** Tools
**Estimated Tokens:** 420
**MCP Server:** `@modelcontextprotocol/server-postgres`
**Official Docs:** https://www.postgresql.org/docs/

## Quick Reference (50 tokens)

PostgreSQL is the default database for all multi-tenant SaaS applications. Use with Prisma/Drizzle ORM and enforce tenant isolation on every query.

## Overview (100 tokens)

PostgreSQL provides:
- **ACID compliance** for data integrity
- **Multi-tenant patterns** (row-level security, schema-per-tenant)
- **JSON support** for flexible data
- **Full-text search** built-in
- **Robust indexing** for performance

**When to use:** Default choice for all applications. Switch only if specific requirements (e.g., time-series data) demand it.

**MCP Integration:** Install `@modelcontextprotocol/server-postgres` for direct database operations via Claude Code.

## MCP Server Setup (80 tokens)

```json
// Add to Claude Code MCP settings
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-postgres",
        "postgresql://user:password@localhost:5432/dbname"
      ]
    }
  }
}
```

Use environment variable: `postgresql://${DB_USER}:${DB_PASSWORD}@${DB_HOST}:${DB_PORT}/${DB_NAME}`

## Key Concepts (100 tokens)

### 1. Multi-Tenant Isolation Strategies

**Row-Level Security (Default):**
```sql
-- Every table includes tenantId
CREATE TABLE users (
  id UUID PRIMARY KEY,
  tenant_id UUID NOT NULL,
  email VARCHAR(255) NOT NULL,
  CONSTRAINT fk_tenant FOREIGN KEY (tenant_id) REFERENCES tenants(id)
);

-- Index for performance
CREATE INDEX idx_users_tenant ON users(tenant_id);
```

**PostgreSQL RLS (Advanced):**
```sql
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation ON users
  USING (tenant_id = current_setting('app.tenant_id')::uuid);
```

### 2. Indexes for Performance

```sql
-- Tenant-scoped queries
CREATE INDEX idx_users_tenant_email ON users(tenant_id, email);

-- Full-text search
CREATE INDEX idx_posts_search ON posts USING GIN(to_tsvector('english', title || ' ' || content));
```

### 3. JSON Support

```sql
-- JSONB for flexible metadata
CREATE TABLE settings (
  id UUID PRIMARY KEY,
  tenant_id UUID NOT NULL,
  config JSONB NOT NULL DEFAULT '{}'::jsonb
);

-- Query JSON
SELECT * FROM settings WHERE config->>'theme' = 'dark';
```

## Common Patterns (150 tokens)

### Pattern 1: Connection with Prisma

```prisma
// schema.prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(uuid())
  tenantId  String   @map("tenant_id")
  email     String
  createdAt DateTime @default(now()) @map("created_at")

  tenant    Tenant   @relation(fields: [tenantId], references: [id])

  @@index([tenantId])
  @@index([tenantId, email])
  @@map("users")
}
```

### Pattern 2: Raw SQL with Tenant Isolation

```typescript
// Always use parameterized queries
const users = await db.$queryRaw`
  SELECT * FROM users
  WHERE tenant_id = ${tenantId}
  AND status = 'active'
`
```

### Pattern 3: Migrations

```sql
-- migrations/001_initial.sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

CREATE TABLE tenants (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name VARCHAR(255) NOT NULL,
  subdomain VARCHAR(63) UNIQUE NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  email VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  CONSTRAINT unique_email_per_tenant UNIQUE (tenant_id, email)
);
```

## Best Practices (50 tokens)

**DO:**
- ✅ Always filter by `tenant_id`
- ✅ Use indexes on tenant_id columns
- ✅ Use connection pooling (PgBouncer)
- ✅ Use parameterized queries (prevent SQL injection)
- ✅ Enable UUID extension for IDs

**DON'T:**
- ❌ Query without tenant_id filter
- ❌ Store sensitive data unencrypted
- ❌ Use string concatenation for SQL
- ❌ Create indexes without considering tenant isolation

## Troubleshooting (50 tokens)

**Connection Issues:**
```bash
# Test connection
psql -h localhost -U username -d database_name

# Check connection string
echo $DATABASE_URL
```

**Slow Queries:**
```sql
-- Analyze query performance
EXPLAIN ANALYZE SELECT * FROM users WHERE tenant_id = '...';

-- Check missing indexes
SELECT schemaname, tablename, indexname
FROM pg_indexes
WHERE tablename = 'users';
```

**Tenant Data Leak:**
```typescript
// ❌ BAD - Missing tenant check
const user = await db.user.findUnique({ where: { id: userId } })

// ✅ GOOD - Includes tenant isolation
const user = await db.user.findFirst({
  where: { id: userId, tenantId: tenant.id }
})
```

## Related Skills
- `libraries/prisma.md` - ORM integration
- `libraries/drizzle.md` - Alternative ORM
- `patterns/multi-tenant.md` - Isolation patterns

## Tags
postgresql, database, sql, multi-tenant, orm, rls
