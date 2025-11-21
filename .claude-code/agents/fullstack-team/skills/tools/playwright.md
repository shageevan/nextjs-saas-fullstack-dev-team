# Playwright

**Category:** Tools
**Estimated Tokens:** 400
**Official Docs:** https://playwright.dev

## Quick Reference (50 tokens)

Playwright is a modern E2E testing framework supporting Chrome, Firefox, Safari, and mobile. Use for critical user flows, multi-tenant isolation tests, and cross-browser verification.

## Overview (80 tokens)

Playwright provides:
- **Multi-browser** testing (Chromium, Firefox, WebKit)
- **Auto-wait** for elements (no flaky tests)
- **Network interception** for API mocking
- **Parallel execution** for speed
- **Visual regression** testing
- **Mobile emulation** for responsive tests

**When to use:** E2E tests for critical user flows (signup, payment, multi-tenant access).

**Target:** Cover top 10 user flows with E2E tests.

## Setup (100 tokens)

```bash
# Install Playwright
npm init playwright@latest

# Install browsers
npx playwright install

# Run tests
npx playwright test

# Run with UI
npx playwright test --ui

# Run specific browser
npx playwright test --project=chromium

# Debug mode
npx playwright test --debug
```

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test'

export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',

  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },

  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'mobile',
      use: { ...devices['iPhone 13'] },
    },
  ],

  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
})
```

## Key Concepts (100 tokens)

### 1. Page Object Model

```typescript
// tests/e2e/pages/login.page.ts
import { Page, Locator } from '@playwright/test'

export class LoginPage {
  readonly page: Page
  readonly emailInput: Locator
  readonly passwordInput: Locator
  readonly submitButton: Locator

  constructor(page: Page) {
    this.page = page
    this.emailInput = page.getByLabel('Email')
    this.passwordInput = page.getByLabel('Password')
    this.submitButton = page.getByRole('button', { name: 'Sign in' })
  }

  async goto() {
    await this.page.goto('/login')
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email)
    await this.passwordInput.fill(password)
    await this.submitButton.click()
    await this.page.waitForURL('/dashboard')
  }
}
```

### 2. Test Fixtures

```typescript
// tests/e2e/fixtures.ts
import { test as base } from '@playwright/test'
import { LoginPage } from './pages/login.page'

type MyFixtures = {
  loginPage: LoginPage
  authenticatedPage: Page
}

export const test = base.extend<MyFixtures>({
  loginPage: async ({ page }, use) => {
    const loginPage = new LoginPage(page)
    await use(loginPage)
  },

  authenticatedPage: async ({ page }, use) => {
    const loginPage = new LoginPage(page)
    await loginPage.goto()
    await loginPage.login('test@example.com', 'password')
    await use(page)
  },
})

export { expect } from '@playwright/test'
```

### 3. API Mocking

```typescript
await page.route('**/api/users', async (route) => {
  await route.fulfill({
    status: 200,
    contentType: 'application/json',
    body: JSON.stringify({
      data: [{ id: '1', name: 'Test User' }],
    }),
  })
})
```

## Common Patterns (150 tokens)

### Pattern 1: Testing Multi-Tenant Isolation

```typescript
// tests/e2e/multi-tenant.spec.ts
import { test, expect } from './fixtures'

test.describe('Multi-Tenant Isolation', () => {
  test('tenant A cannot access tenant B data', async ({ page }) => {
    // Login as Tenant A user
    await page.goto('/login')
    await page.getByLabel('Email').fill('tenant-a@example.com')
    await page.getByLabel('Password').fill('password')
    await page.getByRole('button', { name: 'Sign in' }).click()

    // Get Tenant A project ID from URL
    await page.goto('/projects')
    const tenantAProjectLink = page.locator('a[href^="/projects/"]').first()
    const tenantAProjectId = await tenantAProjectLink.getAttribute('href')

    // Logout and login as Tenant B
    await page.getByRole('button', { name: 'Logout' }).click()
    await page.getByLabel('Email').fill('tenant-b@example.com')
    await page.getByLabel('Password').fill('password')
    await page.getByRole('button', { name: 'Sign in' }).click()

    // Try to access Tenant A's project
    await page.goto(tenantAProjectId!)

    // Should see 404 or access denied
    await expect(page.getByText(/not found|access denied/i)).toBeVisible()
  })
})
```

### Pattern 2: Testing Stripe Checkout Flow

```typescript
// tests/e2e/checkout.spec.ts
import { test, expect } from './fixtures'

test.describe('Stripe Checkout', () => {
  test('completes subscription purchase', async ({ authenticatedPage: page }) => {
    // Navigate to pricing page
    await page.goto('/pricing')

    // Select Pro plan
    await page.getByRole('button', { name: /upgrade to pro/i }).click()

    // Redirects to Stripe Checkout
    await page.waitForURL(/checkout.stripe.com/)

    // Fill in test card (use Stripe test mode)
    const frame = page.frameLocator('iframe[name^="__privateStripeFrame"]')
    await frame.locator('[name="cardnumber"]').fill('4242424242424242')
    await frame.locator('[name="exp-date"]').fill('12/34')
    await frame.locator('[name="cvc"]').fill('123')
    await frame.locator('[name="postal"]').fill('12345')

    // Complete purchase
    await page.getByRole('button', { name: /pay/i }).click()

    // Wait for success redirect
    await page.waitForURL(/\/success/)

    // Verify subscription active
    await page.goto('/settings/billing')
    await expect(page.getByText('Pro Plan')).toBeVisible()
    await expect(page.getByText('Active')).toBeVisible()
  })
})
```

### Pattern 3: Testing Critical User Flow

```typescript
// tests/e2e/onboarding.spec.ts
import { test, expect } from '@playwright/test'

test('complete user onboarding flow', async ({ page }) => {
  // 1. Signup
  await page.goto('/signup')
  await page.getByLabel('Email').fill('newuser@example.com')
  await page.getByLabel('Password').fill('SecurePassword123!')
  await page.getByRole('button', { name: 'Create account' }).click()

  // 2. Email verification (mock)
  await page.route('**/api/verify-email', (route) => route.fulfill({ status: 200 }))
  await page.goto('/verify?token=mock-token')
  await expect(page.getByText('Email verified')).toBeVisible()

  // 3. Company setup
  await page.getByLabel('Company name').fill('Acme Corp')
  await page.getByLabel('Subdomain').fill('acme')
  await page.getByRole('button', { name: 'Continue' }).click()

  // 4. Team invites
  await page.getByLabel('Email').fill('teammate@example.com')
  await page.getByRole('button', { name: 'Invite' }).click()
  await expect(page.getByText('Invitation sent')).toBeVisible()
  await page.getByRole('button', { name: 'Skip for now' }).click()

  // 5. Complete onboarding
  await expect(page).toHaveURL('/dashboard')
  await expect(page.getByText('Welcome to Acme Corp')).toBeVisible()
})
```

### Pattern 4: Visual Regression Testing

```typescript
// tests/e2e/visual.spec.ts
import { test, expect } from './fixtures'

test('pricing page visual regression', async ({ page }) => {
  await page.goto('/pricing')

  // Wait for all images to load
  await page.waitForLoadState('networkidle')

  // Take screenshot
  await expect(page).toHaveScreenshot('pricing-page.png', {
    fullPage: true,
    maxDiffPixels: 100,
  })
})
```

## Best Practices (50 tokens)

**DO:**
- ✅ Use Page Object Model for reusability
- ✅ Test critical user flows only
- ✅ Use auto-wait (don't add manual waits)
- ✅ Test tenant isolation thoroughly
- ✅ Run in CI before deployment
- ✅ Use data-testid for stable selectors
- ✅ Test mobile responsive layouts

**DON'T:**
- ❌ Test every single feature with E2E
- ❌ Use `waitForTimeout` (use `waitForLoadState`)
- ❌ Hardcode selectors that change often
- ❌ Skip cross-browser testing

## Troubleshooting (50 tokens)

**Flaky Tests:**
```typescript
// Use proper waits
await page.waitForLoadState('networkidle')
await page.waitForURL('/dashboard')

// Not waitForTimeout
```

**Element Not Found:**
```typescript
// Use role-based selectors (accessible)
await page.getByRole('button', { name: 'Submit' })

// Not CSS selectors
await page.locator('button.submit-btn')
```

**CI Failures:**
```typescript
// playwright.config.ts
retries: process.env.CI ? 2 : 0,
workers: process.env.CI ? 1 : 4,
```

**Slow Tests:**
```bash
# Run in parallel
npx playwright test --workers=4

# Run specific test
npx playwright test onboarding.spec.ts
```

## Related Skills
- `tools/vitest.md` - Unit testing
- `patterns/testing.md` - Testing strategies

## Tags
playwright, e2e, testing, automation, multi-browser, end-to-end
