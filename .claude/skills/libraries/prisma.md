# Prisma ORM

**Type:** Type-safe ORM with schema-first approach
**Best for:** Teams prioritizing developer experience and type safety
**Official Docs:** https://www.prisma.io/docs

## Installation

```bash
npm install prisma @prisma/client
npx prisma init
```

## Schema Definition (schema.prisma)

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// Multi-tenant models
model Tenant {
  id        String   @id @default(cuid())
  name      String
  slug      String   @unique
  createdAt DateTime @default(now())

  users     User[]
  projects  Project[]
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  tenantId  String
  tenant    Tenant   @relation(fields: [tenantId], references: [id], onDelete: Cascade)

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  deletedAt DateTime? // Soft delete

  @@unique([tenantId, email])
  @@index([tenantId])
  @@index([tenantId, deletedAt])
}

model Project {
  id          String   @id @default(cuid())
  name        String
  description String?
  tenantId    String
  tenant      Tenant   @relation(fields: [tenantId], references: [id], onDelete: Cascade)
  ownerId     String

  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  @@index([tenantId])
  @@index([tenantId, ownerId])
}
```

## Client Generation

```bash
# Generate Prisma Client after schema changes
npx prisma generate

# Create and run migrations
npx prisma migrate dev --name init

# Open Prisma Studio (GUI)
npx prisma studio
```

## Multi-Tenant Queries

### Basic Queries with Tenant Isolation

```typescript
import { db } from '@/lib/db'

// Find all users in a tenant
export async function getUsersByTenant(tenantId: string) {
  return db.user.findMany({
    where: {
      tenantId,
      deletedAt: null // Soft delete filter
    },
    orderBy: { createdAt: 'desc' }
  })
}

// Find specific user with tenant check
export async function getUserById(userId: string, tenantId: string) {
  const user = await db.user.findFirst({
    where: {
      id: userId,
      tenantId, // CRITICAL: Always include tenantId
      deletedAt: null
    }
  })

  if (!user) {
    throw new Error('User not found or access denied')
  }

  return user
}

// Create with automatic tenant
export async function createProject(
  data: { name: string; description?: string; ownerId: string },
  tenantId: string
) {
  return db.project.create({
    data: {
      ...data,
      tenantId // Always set tenantId
    }
  })
}

// Update with tenant verification
export async function updateProject(
  projectId: string,
  tenantId: string,
  data: { name?: string; description?: string }
) {
  // Prisma will throw if project doesn't exist or wrong tenant
  return db.project.update({
    where: {
      id: projectId,
      tenantId // Ensures project belongs to tenant
    },
    data
  })
}

// Soft delete
export async function deleteUser(userId: string, tenantId: string) {
  return db.user.update({
    where: {
      id: userId,
      tenantId
    },
    data: {
      deletedAt: new Date()
    }
  })
}
```

### Relations and Includes

```typescript
// Get projects with owner info
export async function getProjects(tenantId: string) {
  return db.project.findMany({
    where: { tenantId },
    include: {
      tenant: true,
      // Note: If User is related, must ensure user also belongs to tenant
    },
    orderBy: { createdAt: 'desc' }
  })
}

// Complex query with filters
export async function searchUsers(
  tenantId: string,
  search: string
) {
  return db.user.findMany({
    where: {
      tenantId,
      deletedAt: null,
      OR: [
        { name: { contains: search, mode: 'insensitive' } },
        { email: { contains: search, mode: 'insensitive' } }
      ]
    },
    take: 20
  })
}
```

### Transactions

```typescript
// Transfer project ownership atomically
export async function transferProject(
  projectId: string,
  newOwnerId: string,
  tenantId: string
) {
  return db.$transaction(async (tx) => {
    // Verify new owner exists in tenant
    const newOwner = await tx.user.findFirst({
      where: { id: newOwnerId, tenantId }
    })

    if (!newOwner) {
      throw new Error('New owner not found in tenant')
    }

    // Update project
    return tx.project.update({
      where: { id: projectId, tenantId },
      data: { ownerId: newOwnerId }
    })
  })
}
```

## Middleware for Auto-Tenant Filtering

```typescript
// lib/db.ts
import { PrismaClient } from '@prisma/client'

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined
}

export const db = globalForPrisma.prisma ?? new PrismaClient()

// WARNING: Global middleware can be risky if not careful
// Only use if you have consistent tenant context
db.$use(async (params, next) => {
  // Example: Auto-add tenantId to all queries
  // This requires storing tenant in AsyncLocalStorage or similar

  const tenantId = getTenantFromContext() // Your implementation

  if (params.model && tenantId) {
    if (params.action === 'findMany' || params.action === 'findFirst') {
      params.args.where = {
        ...params.args.where,
        tenantId
      }
    }
  }

  return next(params)
})

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = db
}
```

## Migrations

```bash
# Development
npx prisma migrate dev --name add_projects_table

# Production
npx prisma migrate deploy

# Reset database (DEV ONLY)
npx prisma migrate reset

# View migration status
npx prisma migrate status
```

## Seeding

```typescript
// prisma/seed.ts
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

async function main() {
  // Create test tenant
  const tenant = await prisma.tenant.create({
    data: {
      name: 'Acme Corp',
      slug: 'acme'
    }
  })

  // Create test users
  await prisma.user.createMany({
    data: [
      { email: 'admin@acme.com', name: 'Admin', tenantId: tenant.id },
      { email: 'user@acme.com', name: 'User', tenantId: tenant.id }
    ]
  })
}

main()
  .catch(console.error)
  .finally(() => prisma.$disconnect())
```

```json
// package.json
{
  "prisma": {
    "seed": "tsx prisma/seed.ts"
  }
}
```

## Pros & Cons

### Pros ✅
- **Excellent TypeScript support** - Auto-generated types
- **Great DX** - Intuitive API, autocomplete
- **Prisma Studio** - Visual database browser
- **Migration system** - Robust, version controlled
- **Strong community** - Large ecosystem, good docs
- **Auto-completion** - VSCode extension
- **Type-safe relations** - Includes/joins are typed

### Cons ⚠️
- **Larger bundle size** - Generates client code
- **Less SQL control** - Abstraction layer
- **Learning curve** - Schema syntax, migrations
- **Query limitations** - Some complex SQL harder to express
- **Generated code** - Must regenerate after schema changes

## When to Use Prisma

✅ **Use Prisma when:**
- Building new projects from scratch
- Team prioritizes DX and type safety
- Need robust migration system
- Want visual database browser (Studio)
- Building standard CRUD operations
- Team is less familiar with SQL

❌ **Consider alternatives when:**
- Need maximum performance (use Drizzle)
- Want raw SQL control (use Kysely)
- Building microservices with tiny bundles
- Complex queries not well-supported by Prisma
- Need to optimize every query manually

## Best Practices

1. **Always include tenantId in WHERE** - Prevent data leaks
2. **Use soft deletes** - Add deletedAt instead of hard delete
3. **Index tenant columns** - Add @@index([tenantId])
4. **Use transactions** - For multi-step operations
5. **Generate after changes** - Run `prisma generate` after schema edits
6. **Version migrations** - Commit migration files to git
7. **Seed for development** - Create seed script for local dev
8. **Use enums** - Define enums in schema for type safety

## Common Patterns

### Unique Constraints per Tenant
```prisma
model Product {
  sku      String
  tenantId String

  @@unique([tenantId, sku]) // SKU unique per tenant
}
```

### Audit Fields
```prisma
model AuditLog {
  id        String   @id @default(cuid())
  tenantId  String
  action    String
  userId    String
  metadata  Json?
  createdAt DateTime @default(now())

  @@index([tenantId, createdAt])
}
```

### JSON Fields
```prisma
model Settings {
  id       String @id
  tenantId String
  config   Json   // Store arbitrary config

  @@unique([tenantId])
}
```

```typescript
// Usage
await db.settings.create({
  data: {
    tenantId: 'tenant_123',
    config: {
      theme: 'dark',
      notifications: true
    }
  }
})
```

## Performance Tips

- Use `select` to fetch only needed fields
- Use `include` sparingly (creates JOINs)
- Add indexes on frequently queried columns
- Use pagination with `skip`/`take`
- Consider `findFirst` over `findMany` when expecting one result
- Use `$queryRaw` for complex queries Prisma doesn't optimize well
