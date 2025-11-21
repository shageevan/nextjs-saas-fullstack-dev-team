# Fullstack AI Tech Team

> A complete AI-powered development team for building production-ready Next.js SaaS applications with multi-tenant architecture, Stripe payments, and API-first design.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code](https://img.shields.io/badge/Claude_Code-Compatible-blue.svg)](https://claude.com/claude-code)
[![Cursor 2.1+](https://img.shields.io/badge/Cursor_2.1+-Compatible-green.svg)](https://cursor.sh)
[![MCP Enabled](https://img.shields.io/badge/MCP-Enabled-purple.svg)](https://modelcontextprotocol.io)

---

## Overview

The **Fullstack AI Tech Team** is a specialized collection of 12 AI agents designed to work together seamlessly, handling everything from requirements gathering to production deployment. Built for **Next.js 15+**, **Shadcn UI**, and **multi-tenant SaaS** applications.

### Key Features

- **12 Specialized Agents** - PM, Architect, Designers, Developers, Security, QA, DevOps
- **API-First Architecture** - All apps built with secure external APIs
- **Multi-Tenant by Default** - Row-level security, schema isolation, or database-per-tenant
- **Context Optimized** - 72% token reduction (12,300 → 3,500 tokens)
- **Skills Knowledge Base** - 33 curated skills with 90% token efficiency
- **MCP Integration** - 5 tools with direct operations (PostgreSQL, GitHub, Docker, Stripe, Redis)
- **Dual IDE Support** - Native support in Claude Code and Cursor 2.1+

---

## Performance & ROI

**Optimized for maximum efficiency:**

| Metric | Value |
|--------|-------|
| Token Reduction | **72%** (12,300 → 3,500 tokens) |
| Context Efficiency | **95%** (only load what's needed) |
| Annual Cost Savings | **$96K+** (at 1K interactions/day) |
| Response Speed | **40% faster** (smaller context) |
| Skills Efficiency | **90%** (vs loading all 33 skills) |

See [OPTIMIZATION-SUMMARY.md](.claude-code/agents/fullstack-team/OPTIMIZATION-SUMMARY.md) for detailed metrics.

---

## Team Structure

### Orchestrator
- **Kai** - Tech Team Orchestrator - Coordinates all specialists, enforces role boundaries

### Resource Verification
- **Reese** - Resource Verifier - Checks prerequisites before development starts (MCP servers, environment, database)

### Planning & Design
- **Alex** - Product Manager - Requirements, PRDs, user stories
- **Jordan** - Technical Architect - System architecture, API design, ADRs
- **Drew** - UI/UX Designer - User flows, Shadcn UI, accessibility

### Development
- **Sam** - Frontend Developer - Next.js 15+, Server Components, Shadcn UI
- **Morgan** - Backend Developer - Server Actions, API routes, business logic
- **Taylor** - Database Architect - Multi-tenant schema, Prisma/Drizzle, query optimization

### Specialized
- **Riley** - Security Expert - Authentication, RBAC, OWASP Top 10, PCI compliance
- **Avery** - Payment Specialist - Stripe integration, webhooks, subscriptions
- **Casey** - DevOps Engineer - Docker, Kubernetes, CI/CD, monitoring
- **Quinn** - QA Engineer - Unit tests (Vitest), integration tests, E2E (Playwright)

---

## Tech Stack

### Core
- **Framework:** Next.js 15+ (App Router)
- **UI Library:** Shadcn UI + Tailwind CSS + Radix UI
- **Language:** TypeScript

### Backend
- **Database:** PostgreSQL (multi-tenant patterns)
- **ORM:** Prisma or Drizzle
- **Cache:** Redis (Upstash)
- **API:** Server Actions + API Routes (versioned, secured)

### Authentication
- NextAuth.js / Clerk / Auth.js

### Payments
- Stripe (Checkout, Subscriptions, Webhooks)

### Testing
- **Unit:** Vitest
- **E2E:** Playwright
- **Coverage:** 80%+ target

### Deployment
- **Hosting:** Vercel / Docker + Kubernetes
- **CI/CD:** GitHub Actions
- **Monitoring:** Sentry + OpenTelemetry

---

## Quick Start

### Prerequisites

- Node.js 18+ installed
- Claude Code or Cursor 2.1+ IDE
- Git installed

### Installation

1. **Clone this repository:**
   ```bash
   git clone <your-repo-url>
   cd fullstack
   ```

2. **Open in your IDE:**
   ```bash
   # For Claude Code
   claude-code .

   # For Cursor
   cursor .
   ```

3. **Set up MCP servers (optional but recommended):**

   See [MCP-SETUP.md](.claude-code/agents/fullstack-team/MCP-SETUP.md) for detailed instructions.

   Quick setup:
   ```bash
   # Create MCP config
   mkdir -p .cursor
   cp .claude-code/agents/fullstack-team/MCP-SETUP.md .cursor/

   # Add environment variables
   cp .env.example .env.local
   ```

4. **Start using the team:**

   In Claude Code or Cursor, simply start a conversation:
   ```
   "I want to build a project management SaaS with team collaboration
   and subscription billing"
   ```

---

## Usage

### Method 1: Start with Orchestrator

The orchestrator (Kai) analyzes your requirements and coordinates the team:

```
Load: orchestrator

User: "Build a CRM application with:
- Contact management
- Deal pipeline tracking
- Team collaboration
- Stripe subscriptions (3 tiers)
- Mobile responsive
- Multi-tenant (company isolation)"

Orchestrator will:
1. Verify prerequisites (Reese)
2. Create plan (Alex, Jordan, Drew)
3. Coordinate implementation (Sam, Morgan, Taylor)
4. Security review (Riley)
5. Payment integration (Avery)
6. Testing (Quinn)
7. Deployment (Casey)
```

### Method 2: Load Planning Team

For detailed architecture and planning:

```
Load: team-planning

User: "Design architecture for a real-time collaboration tool"

Planning team (PM + Architect + Designer + Security + Database):
- Creates PRD
- Designs system architecture
- Plans multi-tenant strategy
- Designs database schema
- Security assessment
```

### Method 3: Load Development Team

For implementation only:

```
Load: team-development

User: "Implement user authentication with NextAuth and Google OAuth"

Development team (Frontend + Backend + Database + QA):
- Implements auth flow
- Creates database schema
- Builds UI components
- Writes tests
```

---

## Features

### API-First Architecture

Every application is built API-first with secure external access:

- **RESTful APIs** designed before UI implementation
- **Versioned** (`/api/v1/`, `/api/v2/`)
- **Authenticated** (OAuth 2.0, API keys, JWT)
- **Rate limited** to prevent abuse
- **Documented** with OpenAPI/Swagger
- **CORS configured** for cross-origin access
- **External consumption ready**

Example API route:
```typescript
// app/api/v1/users/route.ts
export async function GET(request: NextRequest) {
  // 1. Authenticate
  const tenant = await verifyApiKey(request.headers.get('x-api-key'))
  if (!tenant) return Response.json({ error: 'Unauthorized' }, { status: 401 })

  // 2. Rate limit
  const rateLimit = await checkRateLimit(tenant.id)
  if (!rateLimit.allowed) {
    return Response.json({ error: 'Rate limit exceeded' }, { status: 429 })
  }

  // 3. Query with tenant isolation
  const users = await db.user.findMany({
    where: { tenantId: tenant.id }
  })

  return Response.json({ data: users })
}
```

### Multi-Tenant by Default

Every application includes tenant isolation:

- **Row-level security** (shared DB, tenant_id column) - Default
- **Schema per tenant** (shared DB, separate schemas)
- **Database per tenant** (maximum isolation)
- Automatic tenant context resolution
- Tenant-scoped queries enforced

Example query:
```typescript
// ✅ GOOD - Includes tenant isolation
const projects = await db.project.findMany({
  where: { tenantId: tenant.id }
})

// ❌ BAD - Data leak!
const projects = await db.project.findMany()
```

### Stripe Integration Built-In

- Stripe Checkout for subscriptions
- Webhook handling with signature verification
- Subscription lifecycle management
- Customer Portal for self-service
- Usage-based billing support

### Security First

- Authentication with battle-tested providers
- Role-Based Access Control (RBAC)
- OWASP Top 10 prevention
- Security headers configured
- Rate limiting
- PCI compliance for payments

### Skills Knowledge Base

33 curated skills across 5 categories:

```
skills/
├── frameworks/      # Next.js, React, TypeScript (3 skills)
├── libraries/       # Shadcn UI, Prisma, Stripe, Zod, NextAuth (5 skills)
├── patterns/        # Server Actions, multi-tenant, RBAC, API-first (8 skills)
├── tools/           # PostgreSQL, GitHub, Docker, Stripe CLI, Redis, Vitest, Playwright (11 skills)
└── best-practices/  # Security, performance, accessibility, testing (6 skills)
```

**Self-Learning System:**
- Agents search skills folder before internet
- Create skills on-the-fly when not found
- 90% faster than web searches
- Consistent, curated patterns

### MCP Integration

5 tools with Model Context Protocol servers for direct operations:

| Tool | MCP Server | Capabilities |
|------|------------|--------------|
| PostgreSQL | `@modelcontextprotocol/server-postgres` | Execute queries, inspect schema, create migrations |
| GitHub | `@modelcontextprotocol/server-github` | Manage PRs, issues, branches |
| Docker | `@modelcontextprotocol/server-docker` | Build images, manage containers |
| Stripe CLI | `stripe-mcp-server` | Test payments, manage customers, trigger webhooks |
| Redis | `redis-mcp-server` | Inspect cache, manage keys, check TTLs |

**Benefit:** Agents perform direct operations instead of generating code.

---

## IDE Support

### Claude Code

Native support with full agent coordination:
- Automatic agent loading and role enforcement
- 3-tier context loading (Core/Operations/Reference)
- Skills management and creation
- MCP integration (native)

### Cursor 2.1+

Integrated via `.cursorrules` file:
- Automatic context loading via `.cursorrules`
- Explicit agent loading with `@` file mentions
- Skills folder access
- MCP integration (native in v2.1+)
- Real-time editing and suggestions

**Both IDEs have identical MCP capabilities!**

---

## Project Structure

```
fullstack/
├── README.md                           # This file
├── .cursorrules                        # Cursor integration
├── .env.example                        # Environment template
│
└── .claude-code/
    └── agents/
        └── fullstack-team/
            ├── README.md               # Team documentation
            ├── MCP-SETUP.md            # MCP installation guide
            ├── CURSOR-INTEGRATION.md   # Cursor usage guide
            ├── OPTIMIZATION-SUMMARY.md # Performance metrics
            │
            ├── agents/                 # 12 agent definitions
            │   ├── orchestrator/
            │   ├── resource-verifier.agent.yaml
            │   ├── pm.agent.yaml
            │   ├── architect.agent.yaml
            │   ├── designer.agent.yaml
            │   ├── frontend.agent.yaml
            │   ├── backend.agent.yaml
            │   ├── database.agent.yaml
            │   ├── security.agent.yaml
            │   ├── payment.agent.yaml
            │   ├── devops.agent.yaml
            │   └── qa.agent.yaml
            │
            ├── skills/                 # 33 curated skills
            │   ├── index.yaml
            │   ├── frameworks/
            │   ├── libraries/
            │   ├── patterns/
            │   ├── tools/
            │   └── best-practices/
            │
            ├── docs/                   # Guides and references
            │   ├── CONTEXT-OPTIMIZATION.md
            │   ├── MIGRATION-GUIDE.md
            │   ├── ROLE-BOUNDARIES.md
            │   └── PARALLEL-EXECUTION.md
            │
            └── context-manifest.yaml   # Context loading rules
```

---

## Common Workflows

### Starting a New SaaS Project

1. **Load Orchestrator:**
   ```
   Load: orchestrator
   ```

2. **Describe Your Project:**
   ```
   "Build a SaaS for project management with:
   - Team workspaces (multi-tenant)
   - Task management with Kanban boards
   - Time tracking
   - File attachments
   - Real-time collaboration
   - Stripe subscriptions (Free, Pro, Enterprise)
   - Mobile responsive
   - Public API for integrations"
   ```

3. **Orchestrator Coordinates:**
   - **Reese:** Verifies prerequisites (database, MCPs, environment)
   - **Alex:** Creates PRD with user stories
   - **Jordan:** Designs architecture (multi-tenant, API-first)
   - **Taylor:** Designs database schema
   - **Drew:** Designs UI/UX flows
   - **Riley:** Security review
   - **Sam + Morgan:** Implement features in parallel
   - **Avery:** Stripe integration
   - **Quinn:** Writes tests
   - **Casey:** Sets up deployment

### Adding a Feature

1. **Load Orchestrator:**
   ```
   Load: orchestrator
   Trigger: add-feature
   ```

2. **Describe Feature:**
   ```
   "Add user onboarding flow:
   1. Company setup (name, subdomain, logo)
   2. Team invites (email invitations)
   3. Integration connections (Slack, Google Calendar)
   4. Welcome tour"
   ```

3. **Orchestrator Determines Complexity:**
   - Level 0-1: Quick implementation (Sam + Morgan)
   - Level 2: Architecture review (Jordan)
   - Level 3-4: Full planning cycle

### Deploying to Production

1. **Load Deployment Team:**
   ```
   Load: team-deployment
   Trigger: deploy-production
   ```

2. **Pre-deployment Checks:**
   - All tests passing (Quinn)
   - Security audit passed (Riley)
   - Database backup (Taylor)
   - Environment variables verified (Casey)

3. **Deployment:**
   - Run migrations
   - Deploy application
   - Smoke tests
   - Monitor errors

---

## Documentation

### Core Guides
- [Team Documentation](.claude-code/agents/fullstack-team/README.md) - Complete team overview
- [Quick Start Guide](.claude-code/agents/fullstack-team/docs/QUICKSTART.md) - 5-minute setup
- [MCP Setup](.claude-code/agents/fullstack-team/MCP-SETUP.md) - Install MCP servers
- [Cursor Integration](.claude-code/agents/fullstack-team/CURSOR-INTEGRATION.md) - Use in Cursor

### Optimization
- [Optimization Summary](.claude-code/agents/fullstack-team/OPTIMIZATION-SUMMARY.md) - Performance & ROI
- [Context Optimization](.claude-code/agents/fullstack-team/docs/CONTEXT-OPTIMIZATION.md) - Full strategy
- [Skills Optimization](.claude-code/agents/fullstack-team/skills/SKILLS-OPTIMIZATION.md) - Skills efficiency

### Agent Guides
- [Agent Details](.claude-code/agents/fullstack-team/docs/AGENTS.md) - Detailed agent guide
- [Role Boundaries](.claude-code/agents/fullstack-team/docs/ROLE-BOUNDARIES.md) - Role separation
- [Parallel Execution](.claude-code/agents/fullstack-team/docs/PARALLEL-EXECUTION.md) - Concurrent workflows

### Skills
- [Skills Overview](.claude-code/agents/fullstack-team/skills/README.md) - Skills system
- [Skills Index](.claude-code/agents/fullstack-team/skills/index.yaml) - All 33 skills
- [Skill Template](.claude-code/agents/fullstack-team/skills/SKILL-TEMPLATE.md) - Create new skills

---

## Examples

### Example 1: E-commerce Platform

```
User: "Build an e-commerce platform with:
- Product catalog with categories
- Shopping cart
- Checkout with Stripe
- Order management
- Customer accounts
- Admin dashboard
- Multi-tenant (store owners)
- Public API"

Result:
✅ Complete Next.js application
✅ Multi-tenant architecture (store per tenant)
✅ Stripe Checkout integration
✅ Admin dashboard with Shadcn UI
✅ RESTful API with authentication
✅ 80%+ test coverage
✅ Docker deployment ready
```

### Example 2: Internal HR Tool

```
User: "Create an HR management system:
- Employee directory
- Leave management
- Performance reviews
- Document storage
- Single tenant (our company)
- SSO with Google Workspace"

Result:
✅ Next.js application with NextAuth
✅ Google OAuth integration
✅ File upload to S3
✅ Email notifications
✅ Role-based permissions
✅ Responsive design
✅ Unit + E2E tests
```

### Example 3: API-First Platform

```
User: "Build a customer support ticketing API:
- Create/update/close tickets
- Assign to agents
- Add comments
- Search and filter
- Webhook notifications
- Rate limiting
- API documentation
- Multi-tenant"

Result:
✅ RESTful API with versioning
✅ API key authentication
✅ Rate limiting (1000 req/hour)
✅ OpenAPI documentation
✅ Webhook system
✅ Multi-tenant isolation
✅ Comprehensive tests
```

---

## Architecture Patterns

### Multi-Tenant Isolation

**Row-Level Security (Default):**
```typescript
// Every query includes tenantId
const users = await db.user.findMany({
  where: { tenantId: tenant.id }
})

// Middleware sets tenant context
export async function middleware(request: NextRequest) {
  const subdomain = getSubdomain(request)
  const tenant = await resolveTenant(subdomain)
  // Set in headers for downstream use
}
```

### Server Components First

```typescript
// app/dashboard/page.tsx (Server Component)
export default async function DashboardPage() {
  const tenant = await getCurrentTenant()
  const data = await db.getData(tenant.id)

  return <DashboardView data={data} />
}
```

### Server Actions for Mutations

```typescript
// app/actions/users.ts
'use server'

export async function createUser(formData: FormData) {
  const tenant = await getCurrentTenant()
  const user = await requireAuth()

  // Validate with Zod
  const parsed = schema.safeParse(...)

  // Create with tenant isolation
  await db.user.create({
    data: { ...parsed.data, tenantId: tenant.id }
  })

  revalidatePath('/users')
}
```

---

## Best Practices

### Database Queries
✅ **ALWAYS include tenantId**
```typescript
// ✅ GOOD
const user = await db.user.findFirst({
  where: { id: userId, tenantId: tenant.id }
})

// ❌ BAD - Missing tenant check!
const user = await db.user.findUnique({
  where: { id: userId }
})
```

### Validation
✅ **Always validate with Zod**
```typescript
const schema = z.object({
  email: z.string().email(),
  name: z.string().min(1),
})

const result = schema.safeParse(data)
if (!result.success) {
  return { error: result.error }
}
```

### API Security
✅ **Authenticate, rate limit, document**
```typescript
export async function GET(request: NextRequest) {
  // 1. Authenticate
  const tenant = await verifyApiKey(...)

  // 2. Rate limit
  await checkRateLimit(tenant.id)

  // 3. Query with tenant isolation
  const data = await db.query({ tenantId: tenant.id })

  return Response.json({ data })
}
```

### Stripe Webhooks
✅ **ALWAYS verify signatures**
```typescript
const event = stripe.webhooks.constructEvent(
  body,
  signature,
  webhookSecret
)
// Only process verified events
```

---

## Contributing

This is a living system designed to grow and improve. Contributions are welcome!

### How to Contribute

1. **Report Issues:** Found a bug or have a suggestion? Open an issue.
2. **Improve Skills:** Add new skills or enhance existing ones.
3. **Add Agents:** Create specialized agents for specific domains.
4. **Enhance Documentation:** Improve guides and examples.
5. **Share Success Stories:** Tell us what you built!

### Creating New Skills

1. Use the template: [skills/SKILL-TEMPLATE.md](.claude-code/agents/fullstack-team/skills/SKILL-TEMPLATE.md)
2. Follow the 400-token target
3. Include practical examples
4. Add to [skills/index.yaml](.claude-code/agents/fullstack-team/skills/index.yaml)
5. Submit a PR

---

## Roadmap

### Current
- ✅ 12 specialized agents
- ✅ 33 curated skills
- ✅ MCP integration (5 tools)
- ✅ Claude Code + Cursor support
- ✅ Context optimization (72% reduction)

### Coming Soon
- [ ] Workflow YAML files for common tasks
- [ ] Pre-built templates (Auth, Dashboard, Admin)
- [ ] Video walkthrough tutorials
- [ ] Advanced patterns (WebSockets, Background Jobs)
- [ ] Migration from other frameworks

### Under Consideration
- [ ] Mobile app agent (React Native/Flutter)
- [ ] Email marketing agent
- [ ] Analytics agent
- [ ] AI/ML integration agent

---

## Support

### Getting Help

1. **Read the docs:** Check [Team Documentation](.claude-code/agents/fullstack-team/README.md)
2. **Check examples:** Review common workflows above
3. **Ask the orchestrator:** Load Kai and ask questions
4. **Open an issue:** Report bugs or request features

### Community

- **GitHub Issues:** Bug reports and feature requests
- **Discussions:** Questions and community help
- **Pull Requests:** Code contributions

---

## License

MIT License - See LICENSE file for details

---

## Acknowledgments

- Inspired by **BMAD-METHOD** architecture
- Built for **Claude Code** and **Cursor**
- Powered by **Anthropic's Claude**
- Uses **Model Context Protocol** for tool integration

---

## Get Started Now

Ready to build production-ready SaaS applications with AI?

```bash
# 1. Open this folder in your IDE
claude-code .  # or cursor .

# 2. Start a conversation
"I want to build a [describe your SaaS idea]"

# 3. Let the team handle the rest!
```

The team will coordinate planning, architecture, development, security, testing, and deployment - all optimized for context efficiency and maximum ROI.

**Happy building!** 🚀

---

<p align="center">
  <sub>Built with ❤️ for developers who want to ship faster with AI assistance</sub>
</p>
