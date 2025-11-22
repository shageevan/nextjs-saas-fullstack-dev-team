# Postmark

**MCP Server:** `postmark-mcp` (https://github.com/ActiveCampaign/postmark-mcp)
**Type:** Transactional email service
**Best for:** Reliable, fast email delivery with detailed analytics
**Official Docs:** https://postmarkapp.com/developer

## Installation

```bash
npm install postmark
```

## Setup

```typescript
// lib/email.ts
import { ServerClient } from 'postmark'

export const postmark = new ServerClient(process.env.POSTMARK_API_TOKEN!)

// For multi-tenant with different from addresses
export function getPostmarkClient(apiToken?: string) {
  return new ServerClient(apiToken || process.env.POSTMARK_API_TOKEN!)
}
```

### Environment Variables

```bash
# .env.local
POSTMARK_API_TOKEN=your-server-token
POSTMARK_FROM_EMAIL=noreply@yourdomain.com
POSTMARK_FROM_NAME="Your SaaS Name"

# For testing
POSTMARK_SANDBOX_TOKEN=your-sandbox-token
```

## Basic Email Sending

### Simple Email

```typescript
import { postmark } from '@/lib/email'

export async function sendWelcomeEmail(
  to: string,
  userName: string
) {
  const result = await postmark.sendEmail({
    From: process.env.POSTMARK_FROM_EMAIL!,
    To: to,
    Subject: 'Welcome to Our Platform!',
    TextBody: `Hi ${userName},\n\nWelcome to our platform!`,
    HtmlBody: `
      <h1>Welcome ${userName}!</h1>
      <p>We're excited to have you on board.</p>
    `,
    MessageStream: 'outbound'
  })

  return result
}
```

### With Tracking

```typescript
export async function sendEmailWithTracking(
  to: string,
  subject: string,
  html: string
) {
  return postmark.sendEmail({
    From: process.env.POSTMARK_FROM_EMAIL!,
    To: to,
    Subject: subject,
    HtmlBody: html,
    TextBody: html.replace(/<[^>]*>/g, ''), // Strip HTML for text version
    TrackOpens: true,
    TrackLinks: 'HtmlOnly',
    MessageStream: 'outbound',
    Tag: 'notification', // For analytics
    Metadata: {
      userId: 'user_123',
      tenantId: 'tenant_456'
    }
  })
}
```

## Template Emails

### Create Template in Postmark UI
1. Go to Templates in Postmark dashboard
2. Create template with variables: `{{name}}`, `{{action_url}}`
3. Get Template ID or Alias

### Send with Template

```typescript
export async function sendInvitationEmail(
  email: string,
  inviterName: string,
  tenantName: string,
  inviteToken: string
) {
  const inviteUrl = `${process.env.NEXT_PUBLIC_APP_URL}/invitations/accept?token=${inviteToken}`

  return postmark.sendEmailWithTemplate({
    From: process.env.POSTMARK_FROM_EMAIL!,
    To: email,
    TemplateAlias: 'user-invitation', // Or use TemplateId: 123456
    TemplateModel: {
      inviter_name: inviterName,
      tenant_name: tenantName,
      invite_url: inviteUrl,
      product_name: 'Your SaaS'
    },
    MessageStream: 'outbound',
    Tag: 'invitation',
    Metadata: {
      tenantName,
      inviteToken
    }
  })
}
```

## Multi-Tenant Email Patterns

### Tenant-Specific From Addresses

```typescript
// db/schema.ts (Prisma)
model Tenant {
  id          String  @id
  name        String

  // Email configuration
  emailFrom   String  @default("noreply@yourdomain.com")
  emailName   String?
  postmarkToken String? // Optional: tenant-specific Postmark account
}

// lib/email.ts
export async function sendTenantEmail(
  tenantId: string,
  to: string,
  subject: string,
  html: string
) {
  const tenant = await db.tenant.findUniqueOrThrow({
    where: { id: tenantId }
  })

  // Use tenant-specific token if available
  const client = tenant.postmarkToken
    ? new ServerClient(tenant.postmarkToken)
    : postmark

  return client.sendEmail({
    From: `${tenant.emailName || tenant.name} <${tenant.emailFrom}>`,
    To: to,
    Subject: subject,
    HtmlBody: html,
    TextBody: html.replace(/<[^>]*>/g, ''),
    MessageStream: 'outbound',
    Tag: 'tenant-notification',
    Metadata: {
      tenantId,
      tenantName: tenant.name
    }
  })
}
```

### Branded Email Templates per Tenant

```typescript
export async function sendBrandedEmail(
  tenantId: string,
  to: string,
  templateAlias: string,
  model: Record<string, any>
) {
  const tenant = await db.tenant.findUniqueOrThrow({
    where: { id: tenantId },
    include: { branding: true }
  })

  return postmark.sendEmailWithTemplate({
    From: `${tenant.name} <${tenant.emailFrom}>`,
    To: to,
    TemplateAlias: templateAlias,
    TemplateModel: {
      ...model,
      // Inject tenant branding
      brand_color: tenant.branding?.primaryColor || '#3B82F6',
      brand_logo: tenant.branding?.logoUrl,
      company_name: tenant.name,
      company_address: tenant.address
    },
    MessageStream: 'outbound',
    Metadata: { tenantId }
  })
}
```

## Common Transactional Emails

### Password Reset

```typescript
export async function sendPasswordResetEmail(
  email: string,
  resetToken: string,
  userName: string
) {
  const resetUrl = `${process.env.NEXT_PUBLIC_APP_URL}/auth/reset-password?token=${resetToken}`

  return postmark.sendEmailWithTemplate({
    From: process.env.POSTMARK_FROM_EMAIL!,
    To: email,
    TemplateAlias: 'password-reset',
    TemplateModel: {
      user_name: userName,
      reset_url: resetUrl,
      reset_token: resetToken
    },
    MessageStream: 'outbound',
    Tag: 'password-reset',
    Metadata: { email }
  })
}
```

### Email Verification

```typescript
export async function sendVerificationEmail(
  email: string,
  verificationToken: string
) {
  const verifyUrl = `${process.env.NEXT_PUBLIC_APP_URL}/auth/verify?token=${verificationToken}`

  return postmark.sendEmail({
    From: process.env.POSTMARK_FROM_EMAIL!,
    To: email,
    Subject: 'Verify your email address',
    HtmlBody: `
      <h2>Verify Your Email</h2>
      <p>Click the link below to verify your email address:</p>
      <a href="${verifyUrl}">Verify Email</a>
      <p>Or copy this link: ${verifyUrl}</p>
      <p>This link expires in 24 hours.</p>
    `,
    MessageStream: 'outbound',
    Tag: 'verification'
  })
}
```

### Batch Emails (Notifications)

```typescript
export async function sendBatchNotifications(
  notifications: Array<{
    to: string
    subject: string
    html: string
    userId: string
  }>
) {
  const messages = notifications.map(n => ({
    From: process.env.POSTMARK_FROM_EMAIL!,
    To: n.to,
    Subject: n.subject,
    HtmlBody: n.html,
    TextBody: n.html.replace(/<[^>]*>/g, ''),
    MessageStream: 'outbound',
    Tag: 'batch-notification',
    Metadata: {
      userId: n.userId
    }
  }))

  // Send up to 500 emails in one batch
  return postmark.sendEmailBatch(messages)
}
```

## Webhooks (Bounces, Opens, Clicks)

### Setup Webhook Handler

```typescript
// app/api/webhooks/postmark/route.ts
import { NextRequest } from 'next/server'

export async function POST(req: NextRequest) {
  const body = await req.json()

  // Postmark sends different webhook types
  const { RecordType, Email, MessageID, Metadata } = body

  switch (RecordType) {
    case 'Bounce':
      await handleBounce({
        email: Email,
        bounceType: body.Type,
        messageId: MessageID,
        metadata: Metadata
      })
      break

    case 'Open':
      await handleEmailOpen({
        email: Email,
        messageId: MessageID,
        openedAt: body.ReceivedAt,
        metadata: Metadata
      })
      break

    case 'Click':
      await handleLinkClick({
        email: Email,
        messageId: MessageID,
        clickedUrl: body.OriginalLink,
        metadata: Metadata
      })
      break

    case 'SpamComplaint':
      await handleSpamComplaint({
        email: Email,
        messageId: MessageID
      })
      break
  }

  return Response.json({ received: true })
}

async function handleBounce(data: any) {
  // Mark email as bounced, suspend user notifications
  await db.user.update({
    where: { email: data.email },
    data: {
      emailBounced: true,
      emailBouncedAt: new Date()
    }
  })
}

async function handleEmailOpen(data: any) {
  // Track email opens for analytics
  await db.emailEvent.create({
    data: {
      type: 'open',
      email: data.email,
      messageId: data.messageId,
      openedAt: data.openedAt,
      tenantId: data.metadata?.tenantId
    }
  })
}
```

## Testing

### Sandbox Mode (Development)

```typescript
// Use sandbox token for testing
const testClient = new ServerClient(process.env.POSTMARK_SANDBOX_TOKEN!)

export async function sendTestEmail() {
  // Emails in sandbox won't actually send
  return testClient.sendEmail({
    From: 'sender@example.com',
    To: 'recipient@example.com',
    Subject: 'Test Email',
    TextBody: 'This is a test'
  })
}
```

### Email Preview Helper

```typescript
// app/api/email-preview/route.ts
import { sendWelcomeEmail } from '@/lib/email'

export async function GET(req: NextRequest) {
  // Preview email HTML without sending
  const html = `
    <h1>Welcome Test User!</h1>
    <p>This is a preview of the welcome email.</p>
  `

  return new Response(html, {
    headers: { 'Content-Type': 'text/html' }
  })
}
```

## Error Handling

```typescript
import { ServerClient } from 'postmark'

export async function sendEmailSafely(
  to: string,
  subject: string,
  html: string
) {
  try {
    const result = await postmark.sendEmail({
      From: process.env.POSTMARK_FROM_EMAIL!,
      To: to,
      Subject: subject,
      HtmlBody: html,
      MessageStream: 'outbound'
    })

    return { success: true, messageId: result.MessageID }
  } catch (error: any) {
    console.error('Postmark error:', error)

    // Handle specific Postmark errors
    if (error.statusCode === 422) {
      // Invalid email format
      return { error: 'Invalid email address' }
    }

    if (error.statusCode === 406) {
      // Inactive recipient (bounced previously)
      return { error: 'Email address is inactive' }
    }

    return { error: 'Failed to send email' }
  }
}
```

## MCP Integration

With the Postmark MCP server, agents can:
- Send emails directly through MCP
- Check email delivery status
- View bounce/spam reports
- Test email templates
- Monitor email analytics

### Setup in Claude Code

```json
// .claude-code/mcp_config.json
{
  "mcpServers": {
    "postmark": {
      "command": "npx",
      "args": ["-y", "postmark-mcp"],
      "env": {
        "POSTMARK_API_TOKEN": "${POSTMARK_API_TOKEN}"
      }
    }
  }
}
```

### MCP Usage

Agents can now:
- "Send a welcome email to user@example.com"
- "Check delivery status for message ID xyz"
- "Show bounce statistics for last 7 days"
- "Test the password-reset template"

## Best Practices

### 1. Use Message Streams
Separate transactional from broadcast emails:
- `outbound` - Transactional (password resets, receipts)
- `broadcasts` - Marketing emails (if enabled)

### 2. Always Set Tags
```typescript
Tag: 'invitation' | 'notification' | 'receipt' | 'password-reset'
```
Helps with analytics and debugging

### 3. Include Metadata
```typescript
Metadata: {
  tenantId: 'tenant_123',
  userId: 'user_456',
  feature: 'invitation'
}
```
Track business context for each email

### 4. Handle Bounces
Monitor webhook for bounces and update user records

### 5. Use Templates
Store email design in Postmark templates, not in code

### 6. Text Version
Always include TextBody or Postmark generates it poorly

### 7. Rate Limiting
Postmark handles 10,000+ emails/sec, but implement your own limits:
```typescript
// Rate limit per tenant
await checkEmailRateLimit(tenantId, 100) // 100 emails/hour
```

## Common Patterns

### Queue System for Bulk Emails

```typescript
// Use job queue for large batches
import { Queue } from 'bullmq'

const emailQueue = new Queue('emails')

export async function queueEmail(
  to: string,
  subject: string,
  html: string
) {
  await emailQueue.add('send-email', {
    to,
    subject,
    html
  })
}

// Worker processes emails
emailQueue.process('send-email', async (job) => {
  const { to, subject, html } = job.data
  return sendEmailSafely(to, subject, html)
})
```

### Retry Logic

```typescript
export async function sendEmailWithRetry(
  to: string,
  subject: string,
  html: string,
  maxRetries = 3
) {
  for (let i = 0; i < maxRetries; i++) {
    const result = await sendEmailSafely(to, subject, html)

    if (result.success) {
      return result
    }

    // Wait before retry (exponential backoff)
    await new Promise(resolve => setTimeout(resolve, 1000 * Math.pow(2, i)))
  }

  throw new Error('Failed to send email after retries')
}
```

## Monitoring & Analytics

```typescript
// Get email statistics
export async function getEmailStats(tag?: string) {
  const outbound = await postmark.getOutboundStats({
    tag,
    fromdate: '2024-01-01',
    todate: '2024-12-31'
  })

  return {
    sent: outbound.Sent,
    bounced: outbound.Bounced,
    spamComplaints: outbound.SpamComplaints,
    opened: outbound.Opened,
    clicked: outbound.Clicked
  }
}
```

## Comparison with Alternatives

| Feature | Postmark | SendGrid | AWS SES |
|---------|----------|----------|---------|
| Reliability | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Speed | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| DX | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| Price | Mid | Low | Lowest |
| Analytics | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ |
| Templates | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ |

**Why Postmark:**
- Best deliverability rates
- Fastest delivery speed
- Excellent dashboard and analytics
- Great developer experience
- Dedicated to transactional email
