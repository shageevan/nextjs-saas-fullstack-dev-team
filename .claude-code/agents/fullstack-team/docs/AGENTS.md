# Agent Guide

Complete guide to all 12 specialists in the Fullstack AI Tech Team.

## 🎯 Orchestrator - Kai

**Role:** Tech Team Orchestrator

**When to use:**
- Starting any new project or feature
- Coordinating multiple specialists
- Complex tasks requiring team coordination
- When unsure which agent to use

**Triggers:**
- `analyze-requirements` - Analyze and plan project
- `create-app` - Full app creation workflow
- `add-feature` - Add feature to existing app
- `security-review` - Comprehensive security audit
- `deploy-production` - Production deployment
- `party-mode` - Group discussion with all agents
- `status` - Project status and next steps

**Example:**
```
Load: orchestrator
Trigger: analyze-requirements

"Build a project management SaaS with teams, projects, tasks, and Stripe billing"
```

**Specialty:**
- Breaks down requirements
- Delegates to appropriate specialists
- Enforces quality gates
- Coordinates parallel workstreams

---

## ✅ Resource Verifier - Reese

**Role:** Prerequisites & Resource Verification

**When to use:**
- Before starting any new project
- When setting up development environment
- Before major deployments
- When troubleshooting missing dependencies
- Verifying team readiness

**Triggers:**
- `verify-prerequisites` - Complete prerequisites check
- `verify-mcp-servers` - Check MCP server availability
- `verify-environment` - Check environment variables
- `verify-database` - Test database connectivity
- `verify-npm-packages` - Check installed packages
- `setup-missing-resources` - Coordinate resource setup
- `create-readiness-report` - Generate GO/NO-GO report

**Example:**
```
Load: resource-verifier
Trigger: verify-prerequisites

(Reese will check everything and coordinate setup of missing resources)
```

**Verifies:**
- **MCP Servers**: postgres, github, stripe, docker, k8s, redis
- **Environment Variables**: DATABASE_URL, auth secrets, Stripe keys, etc.
- **NPM Packages**: next, react, typescript, prisma, stripe, etc.
- **Configuration Files**: package.json, tsconfig.json, next.config.js, etc.
- **Database**: Connection, migrations, Redis cache
- **External Services**: Stripe API, auth providers, monitoring

**Coordinates with:**
- DevOps (Casey) - Infrastructure and MCP servers
- Security (Riley) - Secrets and authentication
- Database (Taylor) - Database and cache setup
- Payment (Avery) - Stripe configuration
- Architect (Jordan) - Configuration files

**Critical function:**
- **BLOCKS development** if critical resources missing
- Ensures team has everything needed before starting
- Prevents "it doesn't work on my machine" issues
- Creates clear audit trail of what's set up

**Output:**
- Prerequisites Verification Report
- MCP Server Status
- Missing Resources Report
- Readiness Report (GO/NO-GO)

---

## 📋 Product Manager - Alex

**Role:** Requirements & User Stories

**When to use:**
- Need to clarify requirements
- Create Product Requirements Document (PRD)
- Break epics into user stories
- Prioritize features
- Define acceptance criteria

**Triggers:**
- `create-prd` - Create comprehensive PRD
- `create-user-stories` - Break down into user stories
- `validate-requirements` - Check requirements completeness
- `prioritize-features` - Prioritize with RICE/MoSCoW
- `refine-story` - Refine specific user story

**Example:**
```
Load agent: pm
Trigger: create-prd

"We need a customer support ticketing system"
```

**Delivers:**
- Product Requirements Document
- User stories with acceptance criteria
- Feature prioritization
- Epic breakdown

**Works with:**
- Orchestrator (Kai) - For coordination
- Architect (Jordan) - For feasibility
- Designer (Drew) - For UX

---

## 🏗️ Technical Architect - Jordan

**Role:** System Architecture & Design

**When to use:**
- Design system architecture
- Choose technology stack
- Multi-tenant strategy design
- API design
- Architecture reviews
- Performance planning

**Triggers:**
- `create-architecture` - Design system architecture
- `review-architecture` - Review existing architecture
- `create-adr` - Architecture Decision Record
- `design-api` - Design API contracts
- `design-database` - Work with DB architect
- `performance-review` - Performance audit

**Example:**
```
Load agent: architect
Trigger: create-architecture

"Design architecture for a SaaS with 1000+ tenants, each with 10-100 users"
```

**Delivers:**
- System Architecture Document
- Architecture Decision Records (ADRs)
- API Design Document
- Technology recommendations

**Specialties:**
- Multi-tenant architectures
- Next.js App Router patterns
- Scalability planning
- Performance architecture

**Works with:**
- PM (Alex) - For requirements
- Database (Taylor) - For data architecture
- Security (Riley) - For security architecture

---

## 💻 Frontend Developer - Sam

**Role:** Next.js + Shadcn UI

**When to use:**
- Implement UI pages
- Build components
- Create forms
- Optimize performance
- Accessibility implementation

**Triggers:**
- `implement-page` - Implement complete page
- `create-component` - Create reusable component
- `implement-form` - Form with validation
- `optimize-performance` - Performance audit
- `accessibility-audit` - Accessibility review

**Example:**
```
Load agent: frontend
Trigger: implement-page

"Create a dashboard page with:
- Stats cards
- Recent activity table
- Quick actions
- Responsive layout"
```

**Expertise:**
- Next.js 15 App Router
- Server Components & Server Actions
- Shadcn UI + Tailwind
- React Hook Form + Zod
- Performance (Lighthouse >90)
- Accessibility (WCAG 2.1 AA)

**Delivers:**
- Next.js pages
- Reusable components
- Forms with validation
- Responsive layouts

**Works with:**
- Designer (Drew) - For design specs
- Backend (Morgan) - For API integration
- QA (Quinn) - For testing

---

## ⚙️ Backend Developer - Morgan

**Role:** Server Actions + API Routes

**When to use:**
- Implement Server Actions
- Build API routes
- Business logic
- Database queries
- Webhook handling

**Triggers:**
- `implement-server-action` - Create Server Action
- `implement-api-route` - Create API route
- `implement-background-job` - Background job
- `optimize-queries` - Query optimization
- `add-logging` - Structured logging

**Example:**
```
Load agent: backend
Trigger: implement-server-action

"Create user management actions:
- Create user
- Update user
- Delete user
- With tenant isolation and RBAC"
```

**Expertise:**
- Server Actions (mutations)
- API Routes (webhooks, external)
- Multi-tenant isolation
- Zod validation
- Error handling
- Rate limiting

**Delivers:**
- Server Actions
- API routes
- Business logic
- Background jobs

**Works with:**
- Frontend (Sam) - For API contracts
- Database (Taylor) - For queries
- Security (Riley) - For auth/permissions

---

## 🗄️ Database Architect - Taylor

**Role:** Database Design & Optimization

**When to use:**
- Design database schema
- Choose multi-tenant strategy
- Create migrations
- Optimize queries
- Set up caching

**Triggers:**
- `design-schema` - Design database schema
- `create-migration` - Create migration
- `optimize-query` - Optimize slow query
- `setup-caching` - Redis caching strategy
- `review-performance` - Database performance audit
- `implement-audit-log` - Audit logging

**Example:**
```
Load agent: database
Trigger: design-schema

"Design schema for e-commerce:
- Products with variants
- Orders and order items
- Customers
- Inventory tracking
- Multi-tenant with row-level security"
```

**Expertise:**
- Multi-tenant strategies
- Prisma & Drizzle ORM
- PostgreSQL optimization
- Redis caching
- Query performance
- Database migrations

**Delivers:**
- Database schema (Prisma/Drizzle)
- Migration files
- Indexes and optimizations
- Caching layer

**Works with:**
- Architect (Jordan) - For data architecture
- Backend (Morgan) - For query implementation
- Security (Riley) - For tenant isolation

---

## 🔒 Security Expert - Riley

**Role:** Application Security

**When to use:**
- Implement authentication
- Set up authorization/RBAC
- Security audits
- Tenant isolation review
- PCI compliance (payments)
- Security headers

**Triggers:**
- `security-audit` - Full security audit
- `implement-auth` - Authentication system
- `implement-rbac` - Role-based access control
- `review-tenant-isolation` - Tenant isolation audit
- `configure-security-headers` - Security headers
- `implement-rate-limiting` - Rate limiting

**Example:**
```
Load agent: security
Trigger: security-audit

"Perform comprehensive security audit before production launch"
```

**Expertise:**
- NextAuth.js / Clerk / Auth.js
- RBAC & permissions
- Multi-tenant security
- OWASP Top 10
- PCI compliance
- Security headers

**Delivers:**
- Authentication implementation
- RBAC system
- Security audit reports
- Security configurations

**Critical for:**
- Any auth/permission features
- Payment integrations
- Tenant isolation
- Production deployments

**Works with:**
- Backend (Morgan) - For auth implementation
- Database (Taylor) - For tenant isolation
- Payment (Avery) - For PCI compliance

---

## 💳 Payment Integration Specialist - Avery

**Role:** Stripe Integration

**When to use:**
- Set up Stripe Checkout
- Implement subscriptions
- Handle webhooks
- Usage-based billing
- Customer Portal

**Triggers:**
- `implement-stripe-checkout` - Stripe Checkout
- `implement-webhooks` - Webhook handling
- `implement-customer-portal` - Customer Portal
- `implement-usage-billing` - Usage-based billing
- `test-payment-flow` - End-to-end payment testing

**Example:**
```
Load agent: payment
Trigger: implement-stripe-checkout

"Set up Stripe with three tiers:
- Starter: $19/month
- Pro: $49/month
- Enterprise: $149/month
Include trial period and annual discount"
```

**Expertise:**
- Stripe Checkout
- Subscriptions & invoices
- Webhook handling
- Usage-based billing
- Customer Portal
- PCI compliance

**Delivers:**
- Stripe Checkout flow
- Webhook handlers
- Subscription management
- Billing portal

**Critical checks:**
- Webhook signature verification
- Idempotency
- Failed payment handling
- PCI compliance

**Works with:**
- Backend (Morgan) - For webhook implementation
- Security (Riley) - For PCI compliance
- Database (Taylor) - For subscription data

---

## 🚀 DevOps Engineer - Casey

**Role:** Infrastructure & Deployment

**When to use:**
- Containerize application
- Set up CI/CD
- Deploy to production
- Configure monitoring
- Database migrations
- Environment management

**Triggers:**
- `setup-docker` - Dockerize app
- `setup-cicd` - CI/CD pipeline
- `setup-monitoring` - Monitoring & alerts
- `setup-environments` - Dev/staging/prod
- `database-backup` - Backup strategy
- `deploy-production` - Production deployment

**Example:**
```
Load agent: devops
Trigger: setup-cicd

"Set up GitHub Actions CI/CD with:
- Tests on every PR
- Deploy to staging on develop
- Deploy to production on main
- Run migrations automatically"
```

**Expertise:**
- Docker & Kubernetes
- GitHub Actions
- Vercel deployment
- Monitoring (Sentry, Datadog)
- Database migrations
- Secrets management

**Delivers:**
- Dockerfile & docker-compose
- CI/CD pipelines
- Monitoring setup
- Deployment scripts

**Works with:**
- Architect (Jordan) - For infrastructure design
- Security (Riley) - For secrets management
- Database (Taylor) - For migrations
- QA (Quinn) - For CI testing

---

## 🧪 QA/Test Engineer - Quinn

**Role:** Testing & Quality Assurance

**When to use:**
- Write tests
- Test tenant isolation
- E2E testing
- Test coverage check
- CI test setup

**Triggers:**
- `write-unit-tests` - Unit tests
- `write-integration-tests` - Integration tests
- `write-e2e-tests` - E2E tests
- `test-tenant-isolation` - Tenant isolation testing
- `run-test-suite` - Run all tests
- `setup-ci-tests` - CI test configuration

**Example:**
```
Load agent: qa
Trigger: test-tenant-isolation

"Test that tenant A users cannot access tenant B data"
```

**Expertise:**
- Vitest (unit tests)
- Playwright (E2E)
- Multi-tenant testing
- Test fixtures
- Code coverage
- CI integration

**Delivers:**
- Unit test suite
- Integration tests
- E2E tests
- Coverage reports
- CI test configuration

**Test types:**
- **Unit**: Business logic, utilities
- **Integration**: APIs, Server Actions
- **E2E**: User flows, critical paths

**Works with:**
- All developers - For test requirements
- DevOps (Casey) - For CI integration
- Security (Riley) - For security testing

---

## 🎨 UI/UX Designer - Drew

**Role:** Design & User Experience

**When to use:**
- Design user flows
- Create wireframes
- Customize Shadcn theme
- Design components
- Accessibility review
- Responsive design

**Triggers:**
- `design-user-flow` - Design user flow
- `customize-shadcn-theme` - Customize theme
- `design-component` - Design custom component
- `design-page-layout` - Page layout design
- `accessibility-review` - Accessibility audit
- `create-design-system-docs` - Design system docs

**Example:**
```
Load agent: designer
Trigger: design-user-flow

"Design the onboarding flow:
1. Company/workspace setup
2. Invite team members
3. Connect integrations
4. First project creation"
```

**Expertise:**
- User flow design
- Shadcn UI customization
- Responsive design
- Accessibility (WCAG 2.1 AA)
- Design systems
- Component specifications

**Delivers:**
- User flows
- Wireframes
- Component specs
- Custom themes
- Design system docs

**Design focus:**
- Mobile-first responsive
- Accessibility baked in
- Shadcn component consistency
- Performance-conscious

**Works with:**
- Frontend (Sam) - For implementation
- PM (Alex) - For requirements
- QA (Quinn) - For accessibility testing

---

## Choosing the Right Agent

### Start Here
- **Orchestrator (Kai)** - When in doubt, start here

### By Phase

**Planning:**
- PM (Alex) - Requirements
- Architect (Jordan) - Architecture
- Designer (Drew) - UX design

**Development:**
- Frontend (Sam) - UI implementation
- Backend (Morgan) - Server logic
- Database (Taylor) - Schema & queries

**Specialized:**
- Security (Riley) - Auth, permissions, audits
- Payment (Avery) - Stripe integration
- DevOps (Casey) - Deployment & infrastructure
- QA (Quinn) - Testing

### By Task Type

**Authentication:**
- Security (Riley) - Primary
- Backend (Morgan) - Implementation
- Frontend (Sam) - UI

**Database:**
- Database (Taylor) - Primary
- Architect (Jordan) - Strategy
- Backend (Morgan) - Queries

**Payments:**
- Payment (Avery) - Primary
- Security (Riley) - Compliance
- Backend (Morgan) - Integration

**Deployment:**
- DevOps (Casey) - Primary
- Security (Riley) - Security check
- QA (Quinn) - Testing

**Performance:**
- Architect (Jordan) - Strategy
- Frontend (Sam) - UI performance
- Backend (Morgan) - API performance
- Database (Taylor) - Query optimization

---

## Agent Collaboration Patterns

### Pattern 1: Full Stack Feature
```
Orchestrator → PM (stories) → Architect (design) →
Frontend + Backend + Database (implementation) →
QA (testing) → DevOps (deploy)
```

### Pattern 2: Security Feature
```
Orchestrator → Security (design) →
Backend (implementation) →
Database (schema updates) →
QA (security testing)
```

### Pattern 3: Payment Integration
```
Orchestrator → Payment (Stripe setup) →
Backend (webhooks) →
Frontend (checkout UI) →
Security (compliance check) →
QA (payment flow testing)
```

### Pattern 4: Performance Optimization
```
Orchestrator →
Frontend (UI optimization) +
Backend (API optimization) +
Database (query optimization) +
DevOps (infrastructure tuning)
```

---

## Tips for Working with Agents

### 1. Be Specific
Each agent appreciates detail:
```
❌ "Add authentication"
✅ "Implement Google OAuth and email/password auth with NextAuth.js.
    Include password reset, email verification, and session management."
```

### 2. Use the Right Agent
Don't ask Frontend Sam to set up CI/CD—that's DevOps Casey's job.

### 3. Let Them Collaborate
The Orchestrator knows when to bring in other agents.

### 4. Trust Their Expertise
Agents follow best practices, but you can override:
```
"I know you recommend NextAuth, but we'll use Clerk for this project"
```

### 5. Review Their Work
Agents are experts but not infallible:
- Review code
- Test functionality
- Provide feedback

---

## Next Steps

1. **Load the Orchestrator** - Your starting point for most tasks
2. **Try a specific agent** - Load Frontend, Backend, etc. directly
3. **Load a team bundle** - Get multiple agents for a phase
4. **Customize agents** - Modify YAML files to fit your needs

Ready to build? Load an agent and start!
