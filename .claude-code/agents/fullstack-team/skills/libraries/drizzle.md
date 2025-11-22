# Drizzle ORM

**Type:** Lightweight, SQL-like TypeScript ORM
**Best for:** Performance-critical apps, teams comfortable with SQL
**Official Docs:** https://orm.drizzle.team/docs

## Installation

```bash
npm install drizzle-orm postgres
npm install -D drizzle-kit

# For PostgreSQL
npm install postgres
```

## Schema Definition

```typescript
// db/schema.ts
import { pgTable, text, timestamp, uuid, index, unique } from 'drizzle-orm/pg-core'
import { relations } from 'drizzle-orm'

// Tenants table
export const tenants = pgTable('tenants', {
  id: uuid('id').defaultRandom().primaryKey(),
  name: text('name').notNull(),
  slug: text('slug').notNull().unique(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
})

// Users table with multi-tenant isolation
export const users = pgTable('users', {
  id: uuid('id').defaultRandom().primaryKey(),
  email: text('email').notNull(),
  name: text('name'),
  tenantId: uuid('tenant_id').notNull().references(() => tenants.id, { onDelete: 'cascade' }),

  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
  deletedAt: timestamp('deleted_at'), // Soft delete
}, (table) => ({
  tenantIdx: index('users_tenant_idx').on(table.tenantId),
  tenantEmailIdx: unique('users_tenant_email_unique').on(table.tenantId, table.email),
  tenantDeletedIdx: index('users_tenant_deleted_idx').on(table.tenantId, table.deletedAt),
}))

// Projects table
export const projects = pgTable('projects', {
  id: uuid('id').defaultRandom().primaryKey(),
  name: text('name').notNull(),
  description: text('description'),
  tenantId: uuid('tenant_id').notNull().references(() => tenants.id, { onDelete: 'cascade' }),
  ownerId: uuid('owner_id').notNull(),

  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
}, (table) => ({
  tenantIdx: index('projects_tenant_idx').on(table.tenantId),
  tenantOwnerIdx: index('projects_tenant_owner_idx').on(table.tenantId, table.ownerId),
}))

// Relations (optional but helpful for joins)
export const tenantsRelations = relations(tenants, ({ many }) => ({
  users: many(users),
  projects: many(projects),
}))

export const usersRelations = relations(users, ({ one }) => ({
  tenant: one(tenants, {
    fields: [users.tenantId],
    references: [tenants.id],
  }),
}))

export const projectsRelations = relations(projects, ({ one }) => ({
  tenant: one(tenants, {
    fields: [projects.tenantId],
    references: [tenants.id],
  }),
}))
```

## Database Connection

```typescript
// db/index.ts
import { drizzle } from 'drizzle-orm/postgres-js'
import postgres from 'postgres'
import * as schema from './schema'

const connectionString = process.env.DATABASE_URL!

// Disable prefetch as it's not supported for "Transaction" pool mode
const client = postgres(connectionString, { prepare: false })

export const db = drizzle(client, { schema })
```

## Multi-Tenant Queries

### Basic CRUD with Tenant Isolation

```typescript
import { db } from '@/db'
import { users, projects } from '@/db/schema'
import { eq, and, isNull, ilike, or } from 'drizzle-orm'

// Find all users in a tenant
export async function getUsersByTenant(tenantId: string) {
  return db
    .select()
    .from(users)
    .where(
      and(
        eq(users.tenantId, tenantId),
        isNull(users.deletedAt) // Soft delete filter
      )
    )
    .orderBy(users.createdAt)
}

// Find specific user with tenant check
export async function getUserById(userId: string, tenantId: string) {
  const [user] = await db
    .select()
    .from(users)
    .where(
      and(
        eq(users.id, userId),
        eq(users.tenantId, tenantId), // CRITICAL: Always filter by tenant
        isNull(users.deletedAt)
      )
    )
    .limit(1)

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
  const [project] = await db
    .insert(projects)
    .values({
      ...data,
      tenantId // Always set tenantId
    })
    .returning()

  return project
}

// Update with tenant verification
export async function updateProject(
  projectId: string,
  tenantId: string,
  data: { name?: string; description?: string }
) {
  const [updated] = await db
    .update(projects)
    .set({
      ...data,
      updatedAt: new Date()
    })
    .where(
      and(
        eq(projects.id, projectId),
        eq(projects.tenantId, tenantId) // Ensures project belongs to tenant
      )
    )
    .returning()

  if (!updated) {
    throw new Error('Project not found or access denied')
  }

  return updated
}

// Soft delete
export async function deleteUser(userId: string, tenantId: string) {
  const [deleted] = await db
    .update(users)
    .set({ deletedAt: new Date() })
    .where(
      and(
        eq(users.id, userId),
        eq(users.tenantId, tenantId)
      )
    )
    .returning()

  return deleted
}
```

### Complex Queries

```typescript
import { sql, desc } from 'drizzle-orm'

// Search users with OR conditions
export async function searchUsers(tenantId: string, search: string) {
  return db
    .select()
    .from(users)
    .where(
      and(
        eq(users.tenantId, tenantId),
        isNull(users.deletedAt),
        or(
          ilike(users.name, `%${search}%`),
          ilike(users.email, `%${search}%`)
        )
      )
    )
    .limit(20)
}

// Joins with relations
export async function getProjectsWithTenant(tenantId: string) {
  return db.query.projects.findMany({
    where: eq(projects.tenantId, tenantId),
    with: {
      tenant: true
    },
    orderBy: [desc(projects.createdAt)]
  })
}

// Aggregation
export async function getUserCount(tenantId: string) {
  const [result] = await db
    .select({ count: sql<number>`count(*)` })
    .from(users)
    .where(
      and(
        eq(users.tenantId, tenantId),
        isNull(users.deletedAt)
      )
    )

  return result.count
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
  return db.transaction(async (tx) => {
    // Verify new owner exists in tenant
    const [newOwner] = await tx
      .select()
      .from(users)
      .where(
        and(
          eq(users.id, newOwnerId),
          eq(users.tenantId, tenantId)
        )
      )
      .limit(1)

    if (!newOwner) {
      throw new Error('New owner not found in tenant')
    }

    // Update project
    const [updated] = await tx
      .update(projects)
      .set({ ownerId: newOwnerId })
      .where(
        and(
          eq(projects.id, projectId),
          eq(projects.tenantId, tenantId)
        )
      )
      .returning()

    return updated
  })
}
```

## Raw SQL (When Needed)

```typescript
import { sql } from 'drizzle-orm'

// Execute raw SQL with parameters
const result = await db.execute(
  sql`
    SELECT * FROM users
    WHERE tenant_id = ${tenantId}
    AND email ILIKE ${`%${search}%`}
  `
)

// For complex analytics
const stats = await db.execute(sql`
  SELECT
    COUNT(*) as total_users,
    COUNT(CASE WHEN deleted_at IS NULL THEN 1 END) as active_users
  FROM users
  WHERE tenant_id = ${tenantId}
`)
```

## Migrations

### drizzle.config.ts

```typescript
import type { Config } from 'drizzle-kit'

export default {
  schema: './db/schema.ts',
  out: './db/migrations',
  driver: 'pg',
  dbCredentials: {
    connectionString: process.env.DATABASE_URL!,
  }
} satisfies Config
```

### Migration Commands

```bash
# Generate migration
npx drizzle-kit generate:pg

# Apply migrations
npx drizzle-kit push:pg

# Open Drizzle Studio (visual browser)
npx drizzle-kit studio
```

## Seeding

```typescript
// db/seed.ts
import { db } from './index'
import { tenants, users } from './schema'

async function main() {
  // Create test tenant
  const [tenant] = await db
    .insert(tenants)
    .values({
      name: 'Acme Corp',
      slug: 'acme'
    })
    .returning()

  // Create test users
  await db.insert(users).values([
    { email: 'admin@acme.com', name: 'Admin', tenantId: tenant.id },
    { email: 'user@acme.com', name: 'User', tenantId: tenant.id }
  ])

  console.log('Seed completed')
}

main().catch(console.error)
```

## Pros & Cons

### Pros ✅
- **Lightweight** - Minimal bundle size (~7kb)
- **SQL-like syntax** - Familiar to SQL developers
- **Better performance** - Less abstraction overhead
- **Full type safety** - IntelliSense for everything
- **Raw SQL escape hatch** - Easy to drop down when needed
- **No code generation** - Types from schema directly
- **Flexible** - Use as query builder or full ORM

### Cons ⚠️
- **Newer** - Smaller community than Prisma
- **Less tooling** - Fewer plugins/extensions
- **More manual** - More code for complex relations
- **Learning curve** - If not familiar with SQL
- **Migration system** - Less mature than Prisma's

## When to Use Drizzle

✅ **Use Drizzle when:**
- Performance is critical
- Want lightweight bundle size
- Team is comfortable with SQL
- Need fine-grained query control
- Building microservices
- Want SQL-like syntax with TypeScript
- Need to optimize queries manually

❌ **Consider alternatives when:**
- Team prefers abstraction over SQL (use Prisma)
- Need mature ecosystem and tooling
- Want visual schema designer
- Team is less SQL-comfortable

## Best Practices

1. **Always filter by tenantId** - Use `and()` helper
2. **Use indexes** - Define in schema with `index()`
3. **Soft deletes** - Filter with `isNull(table.deletedAt)`
4. **Type safety** - Use `eq()`, `and()`, not raw comparisons
5. **Transactions** - Use `db.transaction()` for atomic operations
6. **Prepared statements** - Drizzle auto-prepares queries
7. **Relations** - Define for easier joins with `.with()`

## Performance Tips

- Drizzle generates efficient SQL automatically
- Use `.prepare()` for repeated queries
- Select only needed columns: `.select({ id: users.id })`
- Use indexes on frequently queried columns
- Leverage raw SQL for complex analytics
- Connection pooling built-in with postgres.js

## Comparison with Prisma

| Feature | Drizzle | Prisma |
|---------|---------|--------|
| Bundle Size | ~7kb | ~500kb+ |
| Query Style | SQL-like | Method chaining |
| Performance | Faster | Good |
| Type Safety | Excellent | Excellent |
| Migrations | Good | Excellent |
| Community | Growing | Large |
| Learning Curve | Medium | Low |
| DX | Good | Excellent |

**Migration from Prisma:**
Straightforward - schemas translate easily, queries need rewriting but patterns similar.
