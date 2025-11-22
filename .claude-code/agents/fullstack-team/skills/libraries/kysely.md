# Kysely

**Type:** Type-safe SQL query builder
**Best for:** SQL experts, performance-critical apps, teams wanting raw SQL with type safety
**Official Docs:** https://kysely.dev

## Installation

```bash
npm install kysely
npm install pg  # PostgreSQL driver
```

## Type Definitions (Manual Setup)

```typescript
// db/types.ts
import { ColumnType } from 'kysely'

// Type for auto-generated timestamps
type Generated<T> = T extends ColumnType<infer S, infer I, infer U>
  ? ColumnType<S, I | undefined, U>
  : ColumnType<T, T | undefined, T>

// Type for nullable timestamps
type Timestamp = ColumnType<Date, Date | string, Date | string>

// Database schema types
export interface Database {
  tenants: TenantsTable
  users: UsersTable
  projects: ProjectsTable
}

export interface TenantsTable {
  id: Generated<string>
  name: string
  slug: string
  created_at: Generated<Timestamp>
}

export interface UsersTable {
  id: Generated<string>
  email: string
  name: string | null
  tenant_id: string
  created_at: Generated<Timestamp>
  updated_at: Generated<Timestamp>
  deleted_at: Timestamp | null
}

export interface ProjectsTable {
  id: Generated<string>
  name: string
  description: string | null
  tenant_id: string
  owner_id: string
  created_at: Generated<Timestamp>
  updated_at: Generated<Timestamp>
}
```

## Database Connection

```typescript
// db/index.ts
import { Kysely, PostgresDialect } from 'kysely'
import { Pool } from 'pg'
import type { Database } from './types'

const dialect = new PostgresDialect({
  pool: new Pool({
    connectionString: process.env.DATABASE_URL,
    max: 10,
  })
})

export const db = new Kysely<Database>({
  dialect,
})
```

## Multi-Tenant Queries

### Basic CRUD with Tenant Isolation

```typescript
import { db } from '@/db'

// Find all users in a tenant
export async function getUsersByTenant(tenantId: string) {
  return db
    .selectFrom('users')
    .selectAll()
    .where('tenant_id', '=', tenantId)
    .where('deleted_at', 'is', null) // Soft delete filter
    .orderBy('created_at', 'desc')
    .execute()
}

// Find specific user with tenant check
export async function getUserById(userId: string, tenantId: string) {
  const user = await db
    .selectFrom('users')
    .selectAll()
    .where('id', '=', userId)
    .where('tenant_id', '=', tenantId) // CRITICAL: Always filter by tenant
    .where('deleted_at', 'is', null)
    .executeTakeFirst()

  if (!user) {
    throw new Error('User not found or access denied')
  }

  return user
}

// Create with automatic tenant
export async function createProject(
  data: { name: string; description?: string; owner_id: string },
  tenantId: string
) {
  return db
    .insertInto('projects')
    .values({
      ...data,
      tenant_id: tenantId // Always set tenant_id
    })
    .returningAll()
    .executeTakeFirstOrThrow()
}

// Update with tenant verification
export async function updateProject(
  projectId: string,
  tenantId: string,
  data: { name?: string; description?: string }
) {
  const updated = await db
    .updateTable('projects')
    .set({
      ...data,
      updated_at: new Date()
    })
    .where('id', '=', projectId)
    .where('tenant_id', '=', tenantId) // Ensures project belongs to tenant
    .returningAll()
    .executeTakeFirst()

  if (!updated) {
    throw new Error('Project not found or access denied')
  }

  return updated
}

// Soft delete
export async function deleteUser(userId: string, tenantId: string) {
  return db
    .updateTable('users')
    .set({ deleted_at: new Date() })
    .where('id', '=', userId)
    .where('tenant_id', '=', tenantId)
    .returningAll()
    .executeTakeFirstOrThrow()
}
```

### Complex Queries

```typescript
import { sql } from 'kysely'

// Search users with OR conditions
export async function searchUsers(tenantId: string, search: string) {
  return db
    .selectFrom('users')
    .selectAll()
    .where('tenant_id', '=', tenantId)
    .where('deleted_at', 'is', null)
    .where((eb) =>
      eb.or([
        eb('name', 'ilike', `%${search}%`),
        eb('email', 'ilike', `%${search}%`)
      ])
    )
    .limit(20)
    .execute()
}

// Joins
export async function getProjectsWithTenant(tenantId: string) {
  return db
    .selectFrom('projects')
    .innerJoin('tenants', 'tenants.id', 'projects.tenant_id')
    .select([
      'projects.id',
      'projects.name',
      'projects.description',
      'tenants.name as tenant_name',
      'tenants.slug as tenant_slug',
    ])
    .where('projects.tenant_id', '=', tenantId)
    .orderBy('projects.created_at', 'desc')
    .execute()
}

// Aggregation
export async function getUserCount(tenantId: string) {
  const result = await db
    .selectFrom('users')
    .select((eb) => eb.fn.count<number>('id').as('count'))
    .where('tenant_id', '=', tenantId)
    .where('deleted_at', 'is', null)
    .executeTakeFirst()

  return result?.count ?? 0
}

// Subqueries
export async function getActiveUsersWithProjectCount(tenantId: string) {
  return db
    .selectFrom('users')
    .select((eb) => [
      'users.id',
      'users.name',
      'users.email',
      eb
        .selectFrom('projects')
        .select((eb) => eb.fn.count<number>('id').as('count'))
        .whereRef('projects.owner_id', '=', 'users.id')
        .as('project_count')
    ])
    .where('users.tenant_id', '=', tenantId)
    .where('users.deleted_at', 'is', null)
    .execute()
}

// Window functions
export async function getUsersWithRanking(tenantId: string) {
  return db
    .selectFrom('users')
    .select((eb) => [
      'id',
      'name',
      'created_at',
      sql<number>`ROW_NUMBER() OVER (ORDER BY created_at)`.as('rank')
    ])
    .where('tenant_id', '=', tenantId)
    .where('deleted_at', 'is', null)
    .execute()
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
  return db.transaction().execute(async (trx) => {
    // Verify new owner exists in tenant
    const newOwner = await trx
      .selectFrom('users')
      .selectAll()
      .where('id', '=', newOwnerId)
      .where('tenant_id', '=', tenantId)
      .executeTakeFirst()

    if (!newOwner) {
      throw new Error('New owner not found in tenant')
    }

    // Update project
    return trx
      .updateTable('projects')
      .set({ owner_id: newOwnerId })
      .where('id', '=', projectId)
      .where('tenant_id', '=', tenantId)
      .returningAll()
      .executeTakeFirstOrThrow()
  })
}

// Batch insert with transaction
export async function createMultipleUsers(
  users: Array<{ email: string; name: string }>,
  tenantId: string
) {
  return db.transaction().execute(async (trx) => {
    const results = []

    for (const user of users) {
      const created = await trx
        .insertInto('users')
        .values({
          ...user,
          tenant_id: tenantId
        })
        .returningAll()
        .executeTakeFirstOrThrow()

      results.push(created)
    }

    return results
  })
}
```

## Raw SQL (Full Control)

```typescript
import { sql } from 'kysely'

// Execute completely raw SQL
const result = await sql<{ total: number }>`
  SELECT COUNT(*) as total
  FROM users
  WHERE tenant_id = ${tenantId}
  AND deleted_at IS NULL
`.execute(db)

// Raw SQL with Kysely helpers
const complexQuery = await db
  .selectFrom('users')
  .select((eb) => [
    'id',
    'name',
    sql<string>`CONCAT(name, ' (', email, ')')`.as('display_name'),
    sql<number>`EXTRACT(YEAR FROM created_at)`.as('created_year')
  ])
  .where('tenant_id', '=', tenantId)
  .execute()
```

## Migrations

Kysely doesn't have built-in migrations, but provides a migration system:

```typescript
// migrations/001_create_tables.ts
import { Kysely, sql } from 'kysely'

export async function up(db: Kysely<any>): Promise<void> {
  await db.schema
    .createTable('tenants')
    .addColumn('id', 'uuid', (col) =>
      col.primaryKey().defaultTo(sql`gen_random_uuid()`)
    )
    .addColumn('name', 'text', (col) => col.notNull())
    .addColumn('slug', 'text', (col) => col.notNull().unique())
    .addColumn('created_at', 'timestamp', (col) =>
      col.notNull().defaultTo(sql`NOW()`)
    )
    .execute()

  await db.schema
    .createTable('users')
    .addColumn('id', 'uuid', (col) =>
      col.primaryKey().defaultTo(sql`gen_random_uuid()`)
    )
    .addColumn('email', 'text', (col) => col.notNull())
    .addColumn('name', 'text')
    .addColumn('tenant_id', 'uuid', (col) =>
      col.notNull().references('tenants.id').onDelete('cascade')
    )
    .addColumn('created_at', 'timestamp', (col) =>
      col.notNull().defaultTo(sql`NOW()`)
    )
    .addColumn('updated_at', 'timestamp', (col) =>
      col.notNull().defaultTo(sql`NOW()`)
    )
    .addColumn('deleted_at', 'timestamp')
    .execute()

  // Create indexes
  await db.schema
    .createIndex('users_tenant_idx')
    .on('users')
    .column('tenant_id')
    .execute()

  await db.schema
    .createIndex('users_tenant_email_unique')
    .on('users')
    .columns(['tenant_id', 'email'])
    .unique()
    .execute()
}

export async function down(db: Kysely<any>): Promise<void> {
  await db.schema.dropTable('users').execute()
  await db.schema.dropTable('tenants').execute()
}
```

### Migration Runner

```typescript
// migrate.ts
import * as path from 'path'
import { promises as fs } from 'fs'
import { Migrator, FileMigrationProvider } from 'kysely'
import { db } from './db'

async function migrateToLatest() {
  const migrator = new Migrator({
    db,
    provider: new FileMigrationProvider({
      fs,
      path,
      migrationFolder: path.join(__dirname, 'migrations'),
    }),
  })

  const { error, results } = await migrator.migrateToLatest()

  results?.forEach((it) => {
    if (it.status === 'Success') {
      console.log(`Migration "${it.migrationName}" executed successfully`)
    } else if (it.status === 'Error') {
      console.error(`Migration "${it.migrationName}" failed`)
    }
  })

  if (error) {
    console.error('Migration failed')
    console.error(error)
    process.exit(1)
  }

  await db.destroy()
}

migrateToLatest()
```

## Pros & Cons

### Pros ✅
- **Type-safe SQL** - IntelliSense for SQL queries
- **Zero runtime overhead** - No abstraction layer
- **Full SQL control** - Write any SQL you want
- **Lightweight** - Tiny bundle size
- **Perfect type inference** - TypeScript infers everything
- **SQL-first** - Queries look like SQL
- **No code generation** - Types from schema definitions
- **Flexible** - Query builder, not ORM

### Cons ⚠️
- **Manual type definitions** - Must define database types yourself
- **No schema management** - Must write migrations manually
- **SQL knowledge required** - Not beginner-friendly
- **More verbose** - More code than ORM abstractions
- **No relations helper** - Must write joins manually
- **Smaller ecosystem** - Fewer plugins than Prisma

## When to Use Kysely

✅ **Use Kysely when:**
- Team is SQL-expert level
- Need maximum performance (zero overhead)
- Want complete SQL control
- Building performance-critical systems
- Need complex queries (window functions, CTEs, etc.)
- Prefer compile-time safety over runtime abstractions
- Want raw SQL with TypeScript safety

❌ **Consider alternatives when:**
- Team is less comfortable with SQL (use Prisma)
- Want managed schema/migrations (use Prisma/Drizzle)
- Need quick development velocity
- Prefer ORM abstractions

## Best Practices

1. **Always filter by tenant_id** - Use `.where('tenant_id', '=', tenantId)`
2. **Type your database** - Maintain accurate Database interface
3. **Use transactions** - Wrap multi-step operations
4. **Soft deletes** - Filter with `.where('deleted_at', 'is', null)`
5. **Create indexes** - Define in migrations
6. **Prepare queries** - Use `.compile()` for repeated queries
7. **Use expression builders** - For dynamic queries

## Performance

Kysely has **zero runtime overhead**:
- No query parsing at runtime
- Direct SQL generation
- Connection pooling built-in
- Prepared statements automatic
- Type checking at compile time only

**Benchmark (approximate):**
- Kysely: 100% (baseline)
- Drizzle: 95-98%
- Prisma: 80-85%

## Type Generation Helper

```typescript
// scripts/generate-types.ts
// Use kysely-codegen for auto-generating types from database
import { defineConfig } from 'kysely-codegen'

export default defineConfig({
  connectionString: process.env.DATABASE_URL!,
  out: './db/types.ts'
})
```

```bash
npm install -D kysely-codegen
npx kysely-codegen
```

## Comparison with Prisma & Drizzle

| Feature | Kysely | Drizzle | Prisma |
|---------|--------|---------|--------|
| Bundle Size | Tiny | Small | Large |
| Query Style | Raw SQL | SQL-like | Methods |
| Type Safety | Perfect | Excellent | Excellent |
| Performance | Fastest | Fast | Good |
| Migrations | Manual | Good | Excellent |
| Learning Curve | High | Medium | Low |
| DX | Good | Good | Excellent |
| SQL Control | Full | High | Limited |

**Use Kysely if:** You're a SQL expert and need maximum control/performance
**Use Drizzle if:** You want balance of control and convenience
**Use Prisma if:** You want best DX and don't need raw SQL control
