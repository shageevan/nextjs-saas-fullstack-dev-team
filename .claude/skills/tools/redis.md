# Redis (Upstash)

**Category:** Tools
**Estimated Tokens:** 400
**MCP Server:** `redis-mcp-server` (optional)
**Official Docs:** https://redis.io/docs/ | https://upstash.com/docs/redis

## Quick Reference (50 tokens)

Redis for caching, session storage, and rate limiting. Use Upstash for serverless Redis with per-request pricing. Essential for API rate limiting and multi-tenant caching.

## Overview (80 tokens)

Redis provides:
- **Caching** for fast data access
- **Session storage** for authentication
- **Rate limiting** counters
- **Job queues** for background tasks
- **Pub/Sub** for real-time features

**When to use:** All projects requiring caching, rate limiting, or sessions.

**Upstash:** Serverless Redis with HTTP API, perfect for Vercel/serverless environments.

**MCP Integration:** `redis-mcp-server` available for Redis operations.

## Setup (100 tokens)

### Option 1: Upstash (Recommended for Serverless)

```bash
# Create account at upstash.com
# Create Redis database

# Install SDK
npm install @upstash/redis

# Add to .env
UPSTASH_REDIS_REST_URL=https://xxx.upstash.io
UPSTASH_REDIS_REST_TOKEN=your-token
```

```typescript
// lib/redis.ts
import { Redis } from '@upstash/redis'

export const redis = new Redis({
  url: process.env.UPSTASH_REDIS_REST_URL!,
  token: process.env.UPSTASH_REDIS_REST_TOKEN!,
})
```

### Option 2: Local/Self-Hosted Redis

```bash
# Install Redis
brew install redis  # macOS
apt-get install redis  # Ubuntu

# Start Redis
redis-server

# Install Node client
npm install ioredis
```

```typescript
// lib/redis.ts
import Redis from 'ioredis'

export const redis = new Redis(process.env.REDIS_URL || 'redis://localhost:6379')
```

## Key Concepts (120 tokens)

### 1. Caching Patterns

**Cache-Aside (Lazy Loading):**
```typescript
async function getUser(userId: string) {
  // Try cache first
  const cached = await redis.get(`user:${userId}`)
  if (cached) return JSON.parse(cached)

  // Cache miss - fetch from DB
  const user = await db.user.findUnique({ where: { id: userId } })

  // Store in cache (1 hour TTL)
  await redis.setex(`user:${userId}`, 3600, JSON.stringify(user))

  return user
}
```

**Write-Through (Update cache on write):**
```typescript
async function updateUser(userId: string, data: any) {
  // Update database
  const user = await db.user.update({
    where: { id: userId },
    data,
  })

  // Update cache
  await redis.setex(`user:${userId}`, 3600, JSON.stringify(user))

  return user
}
```

### 2. Rate Limiting

**Fixed Window:**
```typescript
export async function checkRateLimit(
  identifier: string,
  limit = 100,
  window = 60
) {
  const key = `rate_limit:${identifier}:${Math.floor(Date.now() / 1000 / window)}`

  const count = await redis.incr(key)

  if (count === 1) {
    await redis.expire(key, window)
  }

  return {
    allowed: count <= limit,
    remaining: Math.max(0, limit - count),
    reset: Math.ceil(Date.now() / 1000 / window) * window,
  }
}
```

### 3. Session Storage

```typescript
export async function createSession(userId: string) {
  const sessionId = crypto.randomUUID()

  await redis.setex(
    `session:${sessionId}`,
    86400, // 24 hours
    JSON.stringify({ userId, createdAt: Date.now() })
  )

  return sessionId
}

export async function getSession(sessionId: string) {
  const data = await redis.get(`session:${sessionId}`)
  return data ? JSON.parse(data) : null
}

export async function deleteSession(sessionId: string) {
  await redis.del(`session:${sessionId}`)
}
```

## Common Patterns (150 tokens)

### Pattern 1: Tenant-Scoped Caching

```typescript
// Cache with tenant isolation
export async function getCachedData<T>(
  tenantId: string,
  key: string,
  fetcher: () => Promise<T>,
  ttl = 3600
): Promise<T> {
  const cacheKey = `tenant:${tenantId}:${key}`

  // Try cache
  const cached = await redis.get(cacheKey)
  if (cached) return JSON.parse(cached) as T

  // Fetch and cache
  const data = await fetcher()
  await redis.setex(cacheKey, ttl, JSON.stringify(data))

  return data
}

// Usage
const projects = await getCachedData(
  tenant.id,
  'projects',
  () => db.project.findMany({ where: { tenantId: tenant.id } }),
  1800 // 30 minutes
)
```

### Pattern 2: API Rate Limiting Middleware

```typescript
// middleware.ts
import { NextRequest, NextResponse } from 'next/server'
import { redis } from '@/lib/redis'

export async function middleware(request: NextRequest) {
  // Skip rate limiting for non-API routes
  if (!request.nextUrl.pathname.startsWith('/api/')) {
    return NextResponse.next()
  }

  // Get identifier (API key or IP)
  const apiKey = request.headers.get('x-api-key')
  const identifier = apiKey || request.ip || 'anonymous'

  // Check rate limit (100 requests per minute)
  const { allowed, remaining, reset } = await checkRateLimit(identifier, 100, 60)

  if (!allowed) {
    return NextResponse.json(
      { error: 'Rate limit exceeded' },
      {
        status: 429,
        headers: {
          'X-RateLimit-Limit': '100',
          'X-RateLimit-Remaining': '0',
          'X-RateLimit-Reset': reset.toString(),
          'Retry-After': (reset - Math.floor(Date.now() / 1000)).toString(),
        },
      }
    )
  }

  const response = NextResponse.next()
  response.headers.set('X-RateLimit-Limit', '100')
  response.headers.set('X-RateLimit-Remaining', remaining.toString())
  response.headers.set('X-RateLimit-Reset', reset.toString())

  return response
}
```

### Pattern 3: Cache Invalidation

```typescript
// Invalidate user cache on update
export async function invalidateUserCache(userId: string) {
  await redis.del(`user:${userId}`)
}

// Invalidate tenant cache on update
export async function invalidateTenantCache(tenantId: string) {
  // Get all keys matching pattern
  const keys = await redis.keys(`tenant:${tenantId}:*`)

  if (keys.length > 0) {
    await redis.del(...keys)
  }
}

// Use in Server Action
'use server'
export async function updateProject(projectId: string, data: any) {
  const tenant = await getCurrentTenant()

  const project = await db.project.update({
    where: { id: projectId, tenantId: tenant.id },
    data,
  })

  // Invalidate cache
  await invalidateTenantCache(tenant.id)

  return project
}
```

## Best Practices (50 tokens)

**DO:**
- ✅ Use tenant-scoped keys for multi-tenant apps
- ✅ Set appropriate TTLs (don't cache forever)
- ✅ Invalidate cache on updates
- ✅ Use Upstash for serverless (no connection pooling issues)
- ✅ Handle cache misses gracefully
- ✅ Monitor cache hit rates

**DON'T:**
- ❌ Cache without TTL (use `setex` not `set`)
- ❌ Store sensitive data without encryption
- ❌ Forget to invalidate on updates
- ❌ Cache too much data (> 1MB per key)

## Troubleshooting (50 tokens)

**Connection Issues:**
```typescript
// Test connection
await redis.ping() // Should return "PONG"

// Check URL
console.log(process.env.UPSTASH_REDIS_REST_URL)
```

**Cache Not Working:**
```typescript
// Debug cache
const cached = await redis.get(key)
console.log('Cache hit:', cached !== null)

// Check TTL
const ttl = await redis.ttl(key)
console.log('TTL remaining:', ttl, 'seconds')
```

**Rate Limit Not Resetting:**
```typescript
// Check current count
const key = `rate_limit:${identifier}:${Math.floor(Date.now() / 1000 / window)}`
const count = await redis.get(key)
console.log('Current count:', count)

// Manually reset
await redis.del(key)
```

## Related Skills
- `patterns/caching.md` - Caching strategies
- `patterns/rate-limiting.md` - Rate limiting patterns

## Tags
redis, caching, rate-limiting, upstash, sessions, performance
