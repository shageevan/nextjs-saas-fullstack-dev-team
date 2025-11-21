# Stripe CLI & MCP

**Category:** Tools
**Estimated Tokens:** 400
**MCP Server:** `stripe-mcp-server`
**Official Docs:** https://stripe.com/docs/stripe-cli

## Quick Reference (50 tokens)

Stripe CLI for local webhook testing and Stripe operations. MCP server enables direct Stripe API access from Claude Code for managing customers, subscriptions, and webhooks.

## Overview (80 tokens)

Stripe CLI provides:
- **Webhook testing** locally (forward events)
- **API operations** from terminal
- **Event triggering** for testing
- **Log streaming** for debugging
- **Sample data** for development

**When to use:** Every project with payments, from development through production.

**MCP Integration:** Install `stripe-mcp-server` for Stripe operations via Claude Code.

## MCP Server Setup (80 tokens)

```json
// Add to Claude Code MCP settings
{
  "mcpServers": {
    "stripe": {
      "command": "npx",
      "args": [
        "-y",
        "stripe-mcp-server"
      ],
      "env": {
        "STRIPE_API_KEY": "<your-stripe-secret-key>"
      }
    }
  }
}
```

**API Key:** Use test mode key (starts with `sk_test_`)

## Stripe CLI Installation (60 tokens)

```bash
# macOS
brew install stripe/stripe-cli/stripe

# Windows (Scoop)
scoop install stripe

# Linux
wget https://github.com/stripe/stripe-cli/releases/latest/download/stripe_linux_amd64.tar.gz
tar -xvf stripe_linux_amd64.tar.gz
sudo mv stripe /usr/local/bin

# Login
stripe login
```

## Key Concepts (100 tokens)

### 1. Webhook Forwarding for Local Dev

```bash
# Forward webhooks to local endpoint
stripe listen --forward-to localhost:3000/api/webhooks/stripe

# Output shows webhook signing secret
# Add to .env.local:
# STRIPE_WEBHOOK_SECRET=whsec_...

# Test specific events
stripe trigger payment_intent.succeeded
stripe trigger customer.subscription.created
stripe trigger checkout.session.completed
```

### 2. Common Stripe CLI Commands

```bash
# View customers
stripe customers list --limit 10

# Create customer
stripe customers create --email="test@example.com" --name="Test User"

# View subscriptions
stripe subscriptions list

# Cancel subscription
stripe subscriptions cancel sub_xxxxx

# View products and prices
stripe products list
stripe prices list

# View recent events
stripe events list --limit 20

# Retrieve specific event
stripe events retrieve evt_xxxxx
```

### 3. Testing Webhooks

```bash
# Start local server
npm run dev

# In another terminal, forward webhooks
stripe listen --forward-to localhost:3000/api/webhooks/stripe

# Trigger test event
stripe trigger payment_intent.succeeded

# View webhook logs
stripe listen --print-json
```

## Common Patterns (150 tokens)

### Pattern 1: Local Development Workflow

```bash
# Terminal 1: Start Next.js
npm run dev

# Terminal 2: Forward webhooks
stripe listen --forward-to localhost:3000/api/webhooks/stripe

# Terminal 3: Trigger events for testing
stripe trigger checkout.session.completed
stripe trigger customer.subscription.updated
stripe trigger invoice.payment_succeeded

# View logs in both terminals
```

### Pattern 2: Testing Webhook Endpoint

```typescript
// app/api/webhooks/stripe/route.ts
import { headers } from 'next/headers'
import { stripe } from '@/lib/stripe'

export async function POST(req: Request) {
  const body = await req.text()
  const signature = headers().get('stripe-signature')!

  let event
  try {
    event = stripe.webhooks.constructEvent(
      body,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET!
    )
  } catch (err) {
    console.error('Webhook signature verification failed')
    return new Response('Invalid signature', { status: 400 })
  }

  // Handle event
  switch (event.type) {
    case 'checkout.session.completed':
      const session = event.data.object
      await handleCheckoutComplete(session)
      break

    case 'customer.subscription.updated':
      const subscription = event.data.object
      await handleSubscriptionUpdate(subscription)
      break
  }

  return new Response(JSON.stringify({ received: true }))
}
```

```bash
# Test the endpoint
stripe trigger checkout.session.completed

# Verify in logs
# ✅ Should see: Event received: checkout.session.completed
# ❌ If error, check STRIPE_WEBHOOK_SECRET
```

### Pattern 3: Create Test Data

```bash
# Create customer with subscription
CUSTOMER=$(stripe customers create \
  --email="test@example.com" \
  --name="Test User" \
  -q)

echo "Customer created: $CUSTOMER"

stripe subscriptions create \
  --customer="$CUSTOMER" \
  --items='[{"price": "price_xxxxx"}]'

# Create checkout session
stripe checkout sessions create \
  --customer="$CUSTOMER" \
  --mode=subscription \
  --line-items='[{"price": "price_xxxxx", "quantity": 1}]' \
  --success-url="http://localhost:3000/success" \
  --cancel-url="http://localhost:3000/cancel"
```

## Best Practices (50 tokens)

**DO:**
- ✅ Use `stripe listen` for local webhook testing
- ✅ Store webhook secret in .env.local
- ✅ Test all webhook events locally before production
- ✅ Use test mode for development (sk_test_)
- ✅ Verify webhook signatures ALWAYS
- ✅ Use Stripe CLI for quick API operations

**DON'T:**
- ❌ Commit webhook secrets
- ❌ Use production keys in development
- ❌ Skip webhook signature verification
- ❌ Process webhooks without idempotency

## Troubleshooting (50 tokens)

**Webhook Not Receiving Events:**
```bash
# Verify forwarding is active
stripe listen --forward-to localhost:3000/api/webhooks/stripe

# Check endpoint is correct
curl http://localhost:3000/api/webhooks/stripe

# Verify webhook secret
echo $STRIPE_WEBHOOK_SECRET
```

**Signature Verification Failed:**
```typescript
// Common issue: Reading body twice
// ❌ BAD
const json = await req.json() // Consumes stream
const body = await req.text() // Empty!

// ✅ GOOD
const body = await req.text() // Read once
const event = stripe.webhooks.constructEvent(body, sig, secret)
```

**CLI Not Authenticated:**
```bash
# Re-login
stripe login

# Verify
stripe customers list --limit 1
```

**Wrong API Key:**
```bash
# Check which key you're using
stripe config --list

# Use specific key
stripe customers list --api-key sk_test_xxxxx
```

## Related Skills
- `libraries/stripe.md` - Stripe SDK integration
- `patterns/webhooks.md` - Webhook patterns

## Tags
stripe, payments, cli, webhooks, testing, mcp
