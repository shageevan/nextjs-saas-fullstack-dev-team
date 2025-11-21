# Quick Start Guide

Get up and running with the Fullstack AI Tech Team in 5 minutes.

## Prerequisites

- Claude Code or compatible IDE with agent support
- Basic understanding of Next.js and TypeScript
- (Optional) MCP servers configured

## Step 1: Load the Orchestrator

The Orchestrator (Kai) is your entry point to the team.

In Claude Code:
```
Load agent: .claude-code/agents/fullstack-team/agents/orchestrator.agent.yaml
```

Or simply say:
```
"Load the Orchestrator agent"
```

## Step 2: Describe Your Project

Tell Kai what you want to build:

### Example 1: New SaaS Application
```
"I want to build a project management SaaS application.

Features needed:
- User authentication (Google OAuth + email/password)
- Teams/workspaces (multi-tenant)
- Projects with tasks
- File attachments
- Comments and activity feeds
- Subscription billing with Stripe (Free, Pro, Enterprise tiers)

Expected scale: 100-1000 companies, 10-50 users per company"
```

### Example 2: Add Feature to Existing App
```
"I need to add a real-time notification system to our existing Next.js app.

Requirements:
- Toast notifications for user actions
- Bell icon with unread count
- Notification center with history
- Email notifications for important events
- Per-tenant notification preferences"
```

### Example 3: Fix and Improve
```
"Our app is slow and has some security issues.

Problems:
- Dashboard takes 3+ seconds to load
- Database queries are slow
- No rate limiting on API endpoints
- Missing tenant isolation on some endpoints"
```

## Step 3: Answer Clarifying Questions

Kai will ask questions to understand your needs:

**Example Questions:**
- "What authentication provider would you prefer?"
- "Do you need social login?"
- "What's your expected scale?"
- "Any compliance requirements (SOC2, GDPR)?"
- "Preferred deployment platform?"

## Step 4: Review the Plan

Kai will create a detailed execution plan:

```markdown
## Execution Plan

### Phase 1: Discovery & Requirements
- Alex (PM): Create PRD with user stories
- Jordan (Architect): Assess technical feasibility
- Drew (Designer): Map user flows

Deliverables:
- Product Requirements Document
- Initial architecture assessment

### Phase 2: Architecture & Planning
- Jordan (Architect): Design system architecture
- Taylor (Database): Design multi-tenant database schema
- Riley (Security): Threat modeling and auth design
- Casey (DevOps): Plan infrastructure and deployment

Deliverables:
- System Architecture Document
- Database schema
- Security architecture
- Infrastructure plan

### Phase 3: Implementation
Parallel workstreams:

**Infrastructure** (Casey)
- Docker setup
- CI/CD pipeline
- Database provisioning

**Backend** (Morgan + Taylor + Avery)
- Authentication
- Multi-tenant middleware
- Database models
- API endpoints
- Stripe integration

**Frontend** (Sam + Drew)
- Next.js setup
- Shadcn UI components
- Pages and layouts
- Forms and validation

**Testing** (Quinn)
- Test framework setup
- Unit tests
- E2E tests

### Phase 4: Deployment
- Casey: Deploy to production
- Quinn: Smoke tests
- Riley: Security verification
```

**Approve the plan** or request changes:
- "Looks good, proceed!"
- "Can we start with authentication first?"
- "Let's use Clerk instead of NextAuth"

## Step 5: Team Executes

Kai coordinates the team through each phase:

### Phase 0: Prerequisites (Automatic)
Before anything else, **Reese (Resource Verifier)** runs:
- ✅ Checks MCP servers (postgres, github, stripe, etc.)
- ✅ Verifies environment variables (DATABASE_URL, secrets, etc.)
- ✅ Tests database connectivity
- ✅ Validates NPM packages installed
- ✅ Checks configuration files exist

**If resources missing:**
- Reese coordinates with specialists to set them up
- DevOps (Casey) for infrastructure
- Security (Riley) for secrets
- Database (Taylor) for database setup

**BLOCKS until GO signal** - Won't proceed without critical resources

### You'll see:
- **Prerequisites check**: "Reese is verifying prerequisites..."
- **Setup coordination**: "Missing DATABASE_URL, coordinating with Taylor..."
- **Delegation**: "Delegating to Alex (PM) to create PRD..."
- **Progress updates**: "Alex has completed the PRD. Moving to Jordan for architecture..."
- **Questions**: Specialists may ask clarifying questions
- **Deliverables**: Documents, code, configurations
- **Reviews**: Quality gates and approvals

### Your role:
- **Answer questions** when asked
- **Provide feedback** on deliverables
- **Make decisions** when options are presented
- **Approve** at quality gates

## Common Use Cases

### Use Case 1: Build from Scratch

```
Load: orchestrator
Trigger: create-app

"Build a SaaS for [problem] with [features]"
```

**Kai will:**
1. Gather requirements (with Alex)
2. Design architecture (with Jordan, Taylor, Riley)
3. Coordinate implementation (all devs)
4. Deploy (with Casey)

**Timeline:** Hours to days depending on complexity

### Use Case 2: Add Major Feature

```
Load: orchestrator
Trigger: add-feature

"Add [feature description]"
```

**Kai will:**
1. Assess complexity
2. Update architecture if needed
3. Coordinate implementation
4. Ensure tests pass

**Timeline:** Minutes to hours

### Use Case 3: Fix Issues

```
Load: orchestrator

"We have [issue description]. Please fix."
```

**Kai will:**
1. Analyze the problem
2. Delegate to appropriate specialist
3. Implement fix
4. Add tests to prevent regression

### Use Case 4: Security Audit

```
Load: orchestrator
Trigger: security-review
```

**Riley will:**
1. Audit authentication/authorization
2. Check tenant isolation
3. Review for OWASP Top 10
4. Test security headers
5. Verify Stripe security
6. Provide detailed report

### Use Case 5: Performance Optimization

```
Load: orchestrator

"Our app is slow, please optimize"
```

**Kai coordinates:**
- Sam: Frontend performance audit
- Morgan: Backend optimization
- Taylor: Database query optimization
- Casey: Infrastructure tuning

## Loading Specific Agents

If you know which specialist you need:

### Frontend Work
```
Load agent: frontend

"Implement a data table with Shadcn UI, including:
- Sorting
- Filtering
- Pagination
- Row selection"
```

### Database Work
```
Load agent: database

"Design the database schema for a course management system with:
- Courses
- Lessons
- Enrollments
- Progress tracking
- Multi-tenant isolation"
```

### Payment Integration
```
Load agent: payment

"Set up Stripe with three subscription tiers:
- Free: $0, limited features
- Pro: $29/month
- Enterprise: $99/month, custom features"
```

## Loading Team Bundles

For specific phases, load the appropriate team:

### Planning Phase
```
Load team: team-planning

"Let's plan a CRM application"
```

Includes: Orchestrator, PM, Architect, Designer, Security, Database

### Implementation Phase
```
Load team: team-development

"Implement the user dashboard"
```

Includes: Orchestrator, Frontend, Backend, Database, QA

### Deployment Phase
```
Load team: team-deployment

"Deploy to production"
```

Includes: Orchestrator, DevOps, Security, QA, Database

## Tips for Success

### 1. Be Specific
❌ "Build a CRM"
✅ "Build a CRM for real estate agents with property listings, client management, and showing scheduling. Multi-tenant with subscription billing."

### 2. Mention Constraints
- Expected scale (users, tenants)
- Budget considerations
- Timeline requirements
- Compliance needs (GDPR, SOC2)

### 3. Iterate
Don't expect perfection on first try:
- Review deliverables
- Ask questions
- Request changes
- Provide feedback

### 4. Trust the Process
The team follows best practices:
- Multi-tenant by default
- Security-first
- Performance-optimized
- Well-tested

But you can override:
- "Skip authentication for now, we'll add it later"
- "Use MongoDB instead of PostgreSQL"
- "Deploy to AWS instead of Vercel"

### 5. Use Quality Gates
Before production:
```
Load: orchestrator

"Run full quality check before deployment"
```

**Kai will coordinate:**
- Quinn: Run all tests
- Riley: Security audit
- Sam: Performance audit
- Casey: Infrastructure check

## Troubleshooting

### Agent Not Found
Ensure the agent file exists:
```
ls .claude-code/agents/fullstack-team/agents/
```

### Unclear Responses
Be more specific:
- Provide more context
- Share existing code
- Clarify requirements

### Need Different Approach
You can override decisions:
- "Actually, let's use Clerk for auth"
- "Use row-level security instead of schema-per-tenant"
- "Skip the design phase, I have wireframes"

## Next Steps

1. **Try a Simple Task**
   ```
   "Create a simple authentication page with email and password"
   ```

2. **Build Something Real**
   ```
   "Let's build [your actual project idea]"
   ```

3. **Explore Agent Capabilities**
   - Load each agent individually
   - Try their specific triggers
   - Read their documentation

4. **Customize**
   - Modify agent personas
   - Adjust best practices
   - Add your own agents

## Getting Help

### From the Orchestrator
```
"Kai, I'm not sure how to proceed with [issue]"
```

### From Specific Agents
```
Load agent: security

"Riley, can you explain the best way to implement RBAC?"
```

### Documentation
- `README.md` - Overview
- `docs/AGENTS.md` - Agent details
- `docs/ARCHITECTURE.md` - Architecture patterns
- Agent YAML files - Full specifications

---

**Ready?** Load the Orchestrator and start building!

```
Load: orchestrator

"I want to build..."
```
