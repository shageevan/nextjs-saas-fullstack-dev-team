# API-First Architecture

**Category:** patterns
**Estimated Tokens:** 450
**Last Updated:** 2025-01-22
**Official Docs:** https://swagger.io/resources/articles/adopting-an-api-first-approach/
**Status:** stable

## Metadata

```yaml
skill:
  id: "patterns/api-first-architecture"
  version: "1.0.0"
  tags: ["api", "architecture", "design", "rest", "external-access"]
  related_skills: ["patterns/api-routes", "best-practices/security", "patterns/rbac"]
  prerequisites: ["frameworks/nextjs"]

context:
  tier: 3B
  estimated_tokens: 450
  load_when: ["api_design", "architecture_planning", "external_integration"]
  cache_duration: "session"
```

## Quick Reference

**One-sentence description:** API-First means designing and building secure, well-documented APIs before implementing the UI, enabling external parties to consume your services.

**When to use:** For all SaaS applications, especially those requiring third-party integrations, mobile apps, or external consumption.

**Key pattern:**
```typescript
// Design API contract first, implement UI second
// All endpoints secured, versioned, and documented
/api/v1/users        → Public API (authenticated)
/api/v1/organizations → Public API (authenticated)
/api/webhooks/*      → Webhook endpoints (signature verified)
```

---

## Overview

API-First architecture means designing and building APIs as the foundation of your application, before implementing the user interface. This approach ensures:

**Benefits:**
- **External consumption** - Third parties can integrate easily
- **Mobile-ready** - Same APIs power web, mobile, desktop
- **Better design** - Forces clear contracts and separation of concerns
- **Parallel development** - Frontend and backend teams work independently
- **Scalability** - APIs can serve multiple clients efficiently

**Use for:** All SaaS applications, especially multi-tenant systems requiring external integrations.

---

## Key Concepts

**Concept 1: Design Before Build**
- Define API contracts first (OpenAPI spec)
- UI implementation comes after
- APIs are the product, UI is a client

**Concept 2: Security First**
- Authentication required for all endpoints
- API keys, OAuth 2.0, or JWT tokens
- Rate limiting to prevent abuse
- Tenant isolation enforced

**Concept 3: External-Ready**
- Versioning for backward compatibility
- Comprehensive documentation
- SDKs for popular languages (optional)
- Webhook support for real-time events

---

## Official Documentation

**Main docs:** https://swagger.io/resources/articles/adopting-an-api-first-approach/
**OpenAPI Spec:** https://swagger.io/specification/
**Next.js API Routes:** https://nextjs.org/docs/app/building-your-application/routing/route-handlers
**API Design:** https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design

---

## Common Patterns

### Pattern 1: RESTful API with Authentication

**When to use:** Standard CRUD operations for resources

**Quick example:**
```typescript
// app/api/v1/users/route.ts
import { NextRequest } from 'next/server'
import { verifyApiKey, getCurrentTenant } from '@/lib/api-auth'
import { db } from '@/lib/db'

export async function GET(request: NextRequest) {
  // 1. Authenticate API request
  const apiKey = request.headers.get('x-api-key')
  const tenant = await verifyApiKey(apiKey)

  if (!tenant) {
    return Response.json(
      { error: 'Unauthorized' },
      { status: 401 }
    )
  }

  // 2. Rate limiting check
  const rateLimit = await checkRateLimit(tenant.id)
  if (!rateLimit.allowed) {
    return Response.json(
      { error: 'Rate limit exceeded' },
      { status: 429, headers: { 'Retry-After': '60' } }
    )
  }

  // 3. Fetch data with tenant isolation
  const users = await db.user.findMany({
    where: { tenantId: tenant.id }
  })

  // 4. Return formatted response
  return Response.json({
    data: users,
    meta: {
      total: users.length,
      tenant: tenant.id
    }
  })
}

export async function POST(request: NextRequest) {
  const apiKey = request.headers.get('x-api-key')
  const tenant = await verifyApiKey(apiKey)

  if (!tenant) {
    return Response.json({ error: 'Unauthorized' }, { status: 401 })
  }

  const body = await request.json()

  // Validate with Zod
  const parsed = createUserSchema.safeParse(body)
  if (!parsed.success) {
    return Response.json(
      { error: 'Validation failed', details: parsed.error },
      { status: 400 }
    )
  }

  // Create user with tenant isolation
  const user = await db.user.create({
    data: {
      ...parsed.data,
      tenantId: tenant.id
    }
  })

  return Response.json(
    { data: user },
    { status: 201 }
  )
}
```

**Key points:**
- Authenticate every request
- Enforce tenant isolation
- Rate limiting
- Consistent response format

### Pattern 2: API Versioning

**When to use:** All external APIs (for backward compatibility)

**Quick example:**
```typescript
// Directory structure for versioning
app/api/
├── v1/
│   ├── users/route.ts
│   └── organizations/route.ts
├── v2/
│   ├── users/route.ts      # New version with breaking changes
│   └── organizations/route.ts
└── route.ts                # Latest version redirect

// app/api/route.ts (redirect to latest)
export async function GET() {
  return Response.redirect('/api/v2')
}

// app/api/v1/users/route.ts (old version, maintained)
export async function GET() {
  // Old response format maintained for backward compatibility
  return Response.json({ users: [...] })
}

// app/api/v2/users/route.ts (new version)
export async function GET() {
  // New response format with additional fields
  return Response.json({
    data: { users: [...] },
    meta: { version: 'v2', ... }
  })
}
```

### Pattern 3: API Key Authentication

**When to use:** Simple API authentication for external clients

**Quick example:**
```typescript
// lib/api-auth.ts
import { db } from '@/lib/db'
import { createHash } from 'crypto'

export async function verifyApiKey(apiKey: string | null) {
  if (!apiKey) return null

  // Hash the API key
  const hashedKey = createHash('sha256')
    .update(apiKey)
    .digest('hex')

  // Look up API key
  const apiKeyRecord = await db.apiKey.findUnique({
    where: { hashedKey },
    include: { tenant: true }
  })

  if (!apiKeyRecord || !apiKeyRecord.active) {
    return null
  }

  // Update last used timestamp
  await db.apiKey.update({
    where: { id: apiKeyRecord.id },
    data: { lastUsedAt: new Date() }
  })

  return apiKeyRecord.tenant
}

// Generate API key for tenant
export async function generateApiKey(tenantId: string) {
  const apiKey = `sk_${generateRandomString(32)}`

  const hashedKey = createHash('sha256')
    .update(apiKey)
    .digest('hex')

  await db.apiKey.create({
    data: {
      tenantId,
      hashedKey,
      active: true,
      createdAt: new Date()
    }
  })

  // Return plain API key ONCE (never shown again)
  return apiKey
}
```

### Pattern 4: Rate Limiting

**When to use:** All external APIs to prevent abuse

**Quick example:**
```typescript
// lib/rate-limit.ts
import { Redis } from '@upstash/redis'

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL!,
  token: process.env.UPSTASH_REDIS_TOKEN!
})

export async function checkRateLimit(
  tenantId: string,
  limit = 1000,
  window = 3600 // 1 hour
) {
  const key = `rate_limit:${tenantId}:${Math.floor(Date.now() / 1000 / window)}`

  const current = await redis.incr(key)

  if (current === 1) {
    await redis.expire(key, window)
  }

  return {
    allowed: current <= limit,
    remaining: Math.max(0, limit - current),
    reset: Math.ceil(Date.now() / 1000 / window) * window
  }
}

// Usage in API route
export async function GET(request: NextRequest) {
  const tenant = await verifyApiKey(request.headers.get('x-api-key'))

  const rateLimit = await checkRateLimit(tenant.id)

  if (!rateLimit.allowed) {
    return Response.json(
      { error: 'Rate limit exceeded' },
      {
        status: 429,
        headers: {
          'X-RateLimit-Limit': '1000',
          'X-RateLimit-Remaining': rateLimit.remaining.toString(),
          'X-RateLimit-Reset': rateLimit.reset.toString()
        }
      }
    )
  }

  // Process request...
}
```

---

## Best Practices

✅ **DO:**
- Design API contracts first (OpenAPI spec)
- Version all external APIs (/api/v1, /api/v2)
- Authenticate every API request
- Enforce rate limiting
- Use consistent response formats
- Document all endpoints thoroughly
- Include pagination for list endpoints
- Return proper HTTP status codes
- Add CORS headers for cross-origin access
- Monitor API usage and errors

❌ **DON'T:**
- Skip authentication on any endpoint
- Expose internal implementation details
- Return sensitive data in errors
- Break backward compatibility without versioning
- Forget tenant isolation in multi-tenant APIs
- Skip rate limiting (leads to abuse)
- Use generic error messages (be specific but safe)
- Return stack traces in production

---

## Common Pitfalls

### Pitfall 1: Missing Tenant Isolation in APIs
**Problem:** API endpoints leak data between tenants
**Fix:** Always include tenant ID check in all queries
```typescript
// ❌ BAD
const users = await db.user.findMany()

// ✅ GOOD
const tenant = await verifyApiKey(apiKey)
const users = await db.user.findMany({
  where: { tenantId: tenant.id }
})
```

### Pitfall 2: No Rate Limiting
**Problem:** API gets abused, costs skyrocket
**Fix:** Implement rate limiting on all endpoints
```typescript
const rateLimit = await checkRateLimit(tenant.id)
if (!rateLimit.allowed) {
  return Response.json({ error: 'Rate limit exceeded' }, { status: 429 })
}
```

### Pitfall 3: Breaking Changes Without Versioning
**Problem:** API updates break existing integrations
**Fix:** Use versioning for all external APIs
```typescript
// Keep v1 stable
/api/v1/users → Old format

// New features in v2
/api/v2/users → New format
```

---

## Troubleshooting

### Issue 1: CORS Errors
**Symptoms:** API works in Postman but fails in browser
**Cause:** Missing CORS headers
**Fix:**
```typescript
export async function GET(request: NextRequest) {
  const response = Response.json({ data: [...] })

  response.headers.set('Access-Control-Allow-Origin', '*')
  response.headers.set('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE')
  response.headers.set('Access-Control-Allow-Headers', 'Content-Type, Authorization')

  return response
}
```

### Issue 2: 401 Unauthorized Despite Valid API Key
**Cause:** API key not being sent correctly
**Fix:** Check header format
```bash
# Correct format
curl -H "x-api-key: sk_your_key_here" https://api.example.com/v1/users

# NOT: Authorization: Bearer sk_...
# Use x-api-key header specifically
```

### Issue 3: Rate Limit Not Working
**Cause:** Redis not configured or key collision
**Fix:** Verify Redis connection and use unique keys per tenant and time window

---

## Integration

**Works well with:**
- `@reference patterns/api-routes` - API route implementation
- `@reference best-practices/security` - API security patterns
- `@reference patterns/rbac` - Permission checks for API endpoints

**Depends on:**
- `@reference frameworks/nextjs` - Next.js API routes

**Requires:**
- Authentication system (NextAuth, Clerk, or custom)
- Rate limiting (Redis/Upstash)
- API documentation (OpenAPI/Swagger)

---

## API Documentation Example

```yaml
# openapi.yaml
openapi: 3.0.0
info:
  title: Your SaaS API
  version: 1.0.0
  description: Public API for external integrations

servers:
  - url: https://api.yoursaas.com/v1

security:
  - ApiKeyAuth: []

paths:
  /users:
    get:
      summary: List users
      parameters:
        - name: x-api-key
          in: header
          required: true
          schema:
            type: string
      responses:
        '200':
          description: List of users
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/User'
        '401':
          description: Unauthorized
        '429':
          description: Rate limit exceeded

components:
  securitySchemes:
    ApiKeyAuth:
      type: apiKey
      in: header
      name: x-api-key

  schemas:
    User:
      type: object
      properties:
        id:
          type: string
        name:
          type: string
        email:
          type: string
```

---

## Updates Log

- **2025-01-22:** Initial creation from API design best practices

---

## Sources

1. [API-First Design - Swagger](https://swagger.io/resources/articles/adopting-an-api-first-approach/)
2. [OpenAPI Specification](https://swagger.io/specification/)
3. [Microsoft API Design Guidelines](https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design)
4. [Next.js Route Handlers](https://nextjs.org/docs/app/building-your-application/routing/route-handlers)
