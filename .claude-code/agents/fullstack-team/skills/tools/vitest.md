# Vitest

**Category:** Tools
**Estimated Tokens:** 400
**Official Docs:** https://vitest.dev

## Quick Reference (50 tokens)

Vitest is a fast unit testing framework for JavaScript/TypeScript. Compatible with Jest API, optimized for Vite projects, but works with any setup. Use for unit and integration tests.

## Overview (80 tokens)

Vitest provides:
- **Blazing fast** test execution
- **Jest-compatible** API (easy migration)
- **Native TypeScript** support
- **Watch mode** with smart re-runs
- **Coverage** reporting built-in
- **Parallel** test execution

**When to use:** Unit tests, integration tests, and testing utilities/helpers.

**Target:** 80%+ code coverage for critical paths.

## Setup (100 tokens)

```bash
# Install Vitest
npm install -D vitest @vitest/ui

# Install testing utilities
npm install -D @testing-library/react @testing-library/jest-dom
npm install -D @testing-library/user-event

# For Next.js
npm install -D @vitejs/plugin-react
```

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: './tests/setup.ts',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'tests/',
        '**/*.config.ts',
        '**/*.d.ts',
      ],
    },
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './'),
    },
  },
})
```

```typescript
// tests/setup.ts
import '@testing-library/jest-dom'
import { expect, afterEach } from 'vitest'
import { cleanup } from '@testing-library/react'

afterEach(() => {
  cleanup()
})
```

```json
// package.json
{
  "scripts": {
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest --coverage"
  }
}
```

## Key Concepts (100 tokens)

### 1. Test Structure

```typescript
import { describe, it, expect, beforeEach, afterEach } from 'vitest'

describe('Feature Name', () => {
  beforeEach(() => {
    // Setup before each test
  })

  afterEach(() => {
    // Cleanup after each test
  })

  it('should do something', () => {
    // Arrange
    const input = 'test'

    // Act
    const result = doSomething(input)

    // Assert
    expect(result).toBe('expected')
  })
})
```

### 2. Mocking

```typescript
import { vi } from 'vitest'

// Mock function
const mockFn = vi.fn()

// Mock module
vi.mock('@/lib/db', () => ({
  db: {
    user: {
      findMany: vi.fn(),
      create: vi.fn(),
    },
  },
}))

// Mock return value
mockFn.mockReturnValue('result')
mockFn.mockResolvedValue('async result')

// Clear mocks
mockFn.mockClear()
vi.clearAllMocks()
```

### 3. React Component Testing

```typescript
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'

it('renders and handles click', async () => {
  const user = userEvent.setup()

  render(<Button onClick={mockFn}>Click me</Button>)

  const button = screen.getByRole('button', { name: /click me/i })
  expect(button).toBeInTheDocument()

  await user.click(button)
  expect(mockFn).toHaveBeenCalledTimes(1)
})
```

## Common Patterns (150 tokens)

### Pattern 1: Testing Server Actions

```typescript
// app/actions/users.ts
'use server'
export async function createUser(formData: FormData) {
  const name = formData.get('name') as string
  const email = formData.get('email') as string

  const user = await db.user.create({
    data: { name, email, tenantId: 'test-tenant' },
  })

  return { success: true, user }
}

// tests/actions/users.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { createUser } from '@/app/actions/users'
import { db } from '@/lib/db'

vi.mock('@/lib/db', () => ({
  db: {
    user: {
      create: vi.fn(),
    },
  },
}))

describe('createUser', () => {
  beforeEach(() => {
    vi.clearAllMocks()
  })

  it('creates user with form data', async () => {
    const mockUser = { id: '1', name: 'Test', email: 'test@example.com' }
    vi.mocked(db.user.create).mockResolvedValue(mockUser)

    const formData = new FormData()
    formData.set('name', 'Test')
    formData.set('email', 'test@example.com')

    const result = await createUser(formData)

    expect(result.success).toBe(true)
    expect(result.user).toEqual(mockUser)
    expect(db.user.create).toHaveBeenCalledWith({
      data: {
        name: 'Test',
        email: 'test@example.com',
        tenantId: 'test-tenant',
      },
    })
  })
})
```

### Pattern 2: Testing API Routes

```typescript
// app/api/users/route.ts
export async function GET(request: NextRequest) {
  const users = await db.user.findMany()
  return Response.json({ data: users })
}

// tests/api/users.test.ts
import { describe, it, expect, vi } from 'vitest'
import { GET } from '@/app/api/users/route'
import { db } from '@/lib/db'

vi.mock('@/lib/db')

describe('GET /api/users', () => {
  it('returns users list', async () => {
    const mockUsers = [
      { id: '1', name: 'User 1' },
      { id: '2', name: 'User 2' },
    ]
    vi.mocked(db.user.findMany).mockResolvedValue(mockUsers)

    const request = new Request('http://localhost:3000/api/users')
    const response = await GET(request as any)

    expect(response.status).toBe(200)

    const data = await response.json()
    expect(data.data).toEqual(mockUsers)
  })
})
```

### Pattern 3: Testing Utilities with Tenant Isolation

```typescript
// lib/tenant.ts
export async function getTenantData<T>(
  tenantId: string,
  key: string,
  fetcher: () => Promise<T>
): Promise<T> {
  // Implementation
}

// tests/lib/tenant.test.ts
import { describe, it, expect, vi } from 'vitest'
import { getTenantData } from '@/lib/tenant'
import { redis } from '@/lib/redis'

vi.mock('@/lib/redis', () => ({
  redis: {
    get: vi.fn(),
    setex: vi.fn(),
  },
}))

describe('getTenantData', () => {
  it('returns cached data if available', async () => {
    const cachedData = { id: '1', name: 'Test' }
    vi.mocked(redis.get).mockResolvedValue(JSON.stringify(cachedData))

    const fetcher = vi.fn()
    const result = await getTenantData('tenant-1', 'key', fetcher)

    expect(result).toEqual(cachedData)
    expect(fetcher).not.toHaveBeenCalled()
    expect(redis.get).toHaveBeenCalledWith('tenant:tenant-1:key')
  })

  it('fetches and caches on cache miss', async () => {
    const freshData = { id: '1', name: 'Fresh' }
    vi.mocked(redis.get).mockResolvedValue(null)

    const fetcher = vi.fn().mockResolvedValue(freshData)
    const result = await getTenantData('tenant-1', 'key', fetcher)

    expect(result).toEqual(freshData)
    expect(fetcher).toHaveBeenCalled()
    expect(redis.setex).toHaveBeenCalledWith(
      'tenant:tenant-1:key',
      3600,
      JSON.stringify(freshData)
    )
  })
})
```

## Best Practices (50 tokens)

**DO:**
- ✅ Follow AAA pattern (Arrange, Act, Assert)
- ✅ Test behavior, not implementation
- ✅ Mock external dependencies (DB, APIs)
- ✅ Clear mocks between tests
- ✅ Use descriptive test names
- ✅ Test tenant isolation in multi-tenant code
- ✅ Aim for 80%+ coverage on critical paths

**DON'T:**
- ❌ Test implementation details
- ❌ Share state between tests
- ❌ Mock everything (over-mocking)
- ❌ Write tests that depend on order

## Troubleshooting (50 tokens)

**Import Errors:**
```typescript
// vitest.config.ts - Add alias
resolve: {
  alias: {
    '@': path.resolve(__dirname, './')
  }
}
```

**Module Not Mocked:**
```typescript
// Use vi.mock at top of file (hoisted)
vi.mock('@/lib/db')

// NOT inside describe/it blocks
```

**React Component Not Rendering:**
```typescript
// Install jsdom and set environment
// vitest.config.ts
test: {
  environment: 'jsdom'
}
```

**Coverage Not Accurate:**
```bash
# Clear cache
rm -rf node_modules/.vitest

# Run with coverage
npm run test:coverage
```

## Related Skills
- `tools/playwright.md` - E2E testing
- `patterns/testing.md` - Testing strategies

## Tags
vitest, testing, unit-tests, jest, javascript, typescript
