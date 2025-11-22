# Next.js Server Actions

**Category:** patterns
**Estimated Tokens:** 400
**Last Updated:** 2025-01-22
**Official Docs:** https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations
**Status:** stable

## Metadata

```yaml
skill:
  id: "patterns/server-actions"
  version: "1.0.0"
  tags: ["nextjs", "mutations", "forms", "server"]
  related_skills: ["libraries/zod", "patterns/error-handling", "patterns/multi-tenant"]
  prerequisites: ["frameworks/nextjs"]

context:
  tier: 3B
  estimated_tokens: 400
  load_when: ["implementing_mutations", "form_handling", "backend_logic"]
  cache_duration: "session"
```

## Quick Reference

**One-sentence description:** Server Actions are async functions that run on the server for handling form submissions and data mutations in Next.js.

**When to use:** For form submissions, data mutations, and any operation that modifies server-side state.

**Key pattern:**
```typescript
'use server'
export async function createItem(formData: FormData) {
  const data = await db.create({ data: {...} })
  revalidatePath('/items')
  return { success: true, data }
}
```

---

## Overview

Server Actions are asynchronous server functions that can be called directly from React components. They eliminate the need for API routes for most mutations.

**Benefits:**
- No API route boilerplate
- Type-safe end-to-end
- Automatic request deduplication
- Built-in error handling

**Use for:** Form submissions, database mutations, revalidating cache, file uploads

---

## Key Concepts

**Concept 1: 'use server' directive**
- Marks function as server-only
- Can be at file or function level
- Never sent to client

**Concept 2: Progressive Enhancement**
- Works without JavaScript
- Form submits even if JS fails
- Graceful degradation built-in

**Concept 3: Revalidation**
- `revalidatePath()` clears cached data
- `revalidateTag()` for granular control
- Automatically triggers re-render

---

## Official Documentation

**Main docs:** https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations
**API reference:** https://nextjs.org/docs/app/api-reference/functions/server-actions
**Guides:** https://nextjs.org/docs/app/building-your-application/data-fetching
**Examples:** https://github.com/vercel/next.js/tree/canary/examples/next-forms

---

## Common Patterns

### Pattern 1: Basic Form Submission with Validation

**When to use:** Standard form with validation

**Quick example:**
```typescript
'use server'
import { z } from 'zod'
import { revalidatePath } from 'next/cache'

const schema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
})

export async function createUser(formData: FormData) {
  // 1. Parse and validate
  const parsed = schema.safeParse({
    name: formData.get('name'),
    email: formData.get('email'),
  })

  if (!parsed.success) {
    return { error: 'Invalid input', details: parsed.error.flatten() }
  }

  // 2. Execute mutation
  const user = await db.user.create({ data: parsed.data })

  // 3. Revalidate
  revalidatePath('/users')

  return { success: true, data: user }
}
```

**Key points:**
- Always validate with Zod
- Return structured response
- Revalidate relevant paths

### Pattern 2: Multi-Tenant with Auth

**When to use:** SaaS with tenant isolation

**Quick example:**
```typescript
'use server'
import { getCurrentTenant, requireAuth } from '@/lib/auth'

export async function createItem(formData: FormData) {
  // 1. Auth & tenant
  const user = await requireAuth()
  const tenant = await getCurrentTenant()

  // 2. Validate
  const parsed = schema.safeParse({...})
  if (!parsed.success) return { error: 'Invalid' }

  // 3. Tenant isolation
  const item = await db.item.create({
    data: {
      ...parsed.data,
      tenantId: tenant.id,
      createdBy: user.id,
    }
  })

  revalidatePath('/items')
  return { success: true, data: item }
}
```

---

## Best Practices

✅ **DO:**
- Validate all inputs with Zod
- Include tenant/user context
- Return structured responses
- Revalidate after mutations
- Handle errors gracefully

❌ **DON'T:**
- Skip input validation
- Return sensitive data to client
- Forget to revalidate
- Use for read operations (use Server Components)
- Include 'use server' in client components

---

## Common Pitfalls

### Pitfall 1: Not Revalidating Cache
**Problem:** UI shows stale data after mutation
**Fix:** Always call `revalidatePath()` or `revalidateTag()` after successful mutation

### Pitfall 2: Missing Tenant Check
**Problem:** Data leaks between tenants
**Fix:** Always include `tenantId` in queries and mutations

### Pitfall 3: Returning Full Error Objects
**Problem:** Exposing internal details to client
**Fix:** Return sanitized error messages only

---

## Troubleshooting

### Issue 1: "Error: Invariant: headers() expects to have requestAsyncStorage"
**Cause:** Using Server Action in wrong context
**Fix:** Ensure 'use server' is properly placed and function is called from Server Component or form action

### Issue 2: FormData returns null
**Cause:** Input missing `name` attribute
**Fix:**
```tsx
<input name="email" /> {/* name is required */}
```

### Issue 3: Revalidation not working
**Cause:** Wrong path or tag specified
**Fix:** Use exact path from `app/` directory, e.g., `revalidatePath('/dashboard/users')`

---

## Integration

**Works well with:**
- `@reference libraries/zod` - Input validation
- `@reference patterns/error-handling` - Structured error responses
- `@reference patterns/multi-tenant` - Tenant isolation

**Depends on:**
- `@reference frameworks/nextjs` - Next.js App Router

**Conflicts with:**
- Client Components (can call, but can't define in them)

---

## Updates Log

- **2025-01-22:** Initial creation from Next.js official docs

---

## Sources

1. [Next.js Server Actions Documentation](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations)
2. [Next.js API Reference](https://nextjs.org/docs/app/api-reference/functions/server-actions)
3. [Next.js Examples](https://github.com/vercel/next.js/tree/canary/examples/next-forms)
