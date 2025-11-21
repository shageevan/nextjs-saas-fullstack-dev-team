# Orchestrator Coordination Examples (Tier 3)
# LOAD ON-DEMAND: Detailed coordination patterns and examples
# Estimated tokens: ~400

## Example 1: Complete Feature Request Flow

**User Request:** "Build a user management feature with roles and permissions"

### Phase 1: Prerequisites (Reese - Resource Verifier)
```
Kai (Orchestrator): "Starting with prerequisites verification."

@resource-verifier Reese, verify we have:
- Database (PostgreSQL)
- Auth MCP server
- Environment variables
- Required NPM packages

Reese: "Running verification..."
✓ PostgreSQL available
✓ NextAuth MCP server configured
✓ DATABASE_URL set
✓ All NPM packages installed

"Prerequisites verified. Ready to proceed."

Kai: "GO signal received. Moving to discovery phase."
```

### Phase 2: Discovery (Parallel - Alex, Jordan, Drew)
```
Kai: "Starting discovery phase with 3 agents in PARALLEL:"

@pm Alex: Clarify requirements for user management
@architect Jordan: Assess technical feasibility
@designer Drew: Analyze user flows

[ALL THREE WORK SIMULTANEOUSLY]

Alex: "Requirements clarified:
- Users belong to tenants
- Roles: Admin, Manager, User
- Permissions: CRUD on resources
- Must integrate with existing auth"

Jordan: "Technical assessment:
- Next.js Server Actions for mutations
- RBAC with permission checks
- Row-level security for multi-tenant
- Compatible with current architecture"

Drew: "User flows mapped:
- Admin invites users
- Users accept invitation
- Role assignment flow
- Permission management UI"

Kai: "Discovery complete. All teams report ready for planning."
```

### Phase 3: Planning (Parallel - Multiple Specialists)
```
Kai: "Starting planning phase with 5 agents in PARALLEL:"

@pm Alex: Create full PRD
@architect Jordan: Design system architecture
@database Taylor: Design multi-tenant user schema
@security Riley: Design RBAC system
@devops Casey: Plan infrastructure needs

[ALL FIVE WORK SIMULTANEOUSLY]

Alex: "PRD complete with user stories and acceptance criteria"
Jordan: "Architecture designed with API contracts"
Taylor: "Schema ready with tenant isolation and role relationships"
Riley: "RBAC design with permission middleware"
Casey: "Infrastructure ready, no changes needed"

Kai: "Planning complete. Ready for implementation."
```

### Phase 4: Implementation (Maximum Parallel with Handoffs)
```
Kai: "Starting implementation with PARALLEL streams and HANDOFFS:"

Stream 1 - Database (Taylor → Morgan handoff):
@database Taylor: Create schema and migrations

Taylor: "Schema and migrations complete. Handing off to Morgan for queries."

@backend Morgan: Implement user queries with tenant filtering

Morgan: "Queries implemented. Server Actions ready."

Stream 2 - Backend (Morgan with Security guidance):
@security Riley: Provide RBAC middleware pattern

Riley: "RBAC middleware pattern ready. Handing to Morgan."

@backend Morgan: Implement RBAC middleware and permission checks

Morgan: "RBAC implemented. All endpoints protected."

Stream 3 - Frontend (Sam needs Morgan's Server Actions):
@frontend Sam: Build user management UI

Sam: "Building UI. Need Server Actions for user CRUD."
[Sam continues building other components while waiting]

Morgan: "Server Actions ready: createUser, updateUser, deleteUser, assignRole"

Sam: "Integrating Server Actions now. User management UI complete."

Stream 4 - Testing (Quinn - continuous parallel):
@qa Quinn: Write tests for user management

Quinn: "Unit tests written. Integration tests added. E2E tests implemented."

[All streams complete]

Kai: "Implementation complete. All tests passing. Ready for deployment."
```

### Phase 5: Deployment
```
Kai: "Prerequisites for deployment:"

@qa Quinn: Confirm all tests passing
@security Riley: Security review for RBAC
@devops Casey: Deploy to production

Quinn: "✓ All tests passing"
Riley: "✓ Security review passed"
Casey: "✓ Deployed to production. Monitoring active."

Kai: "User management feature deployed successfully."
```

---

## Example 2: Handling Role Boundary Violations

**Scenario:** Frontend Developer attempts to implement Server Action

```
User: "Add a createUser Server Action"

Sam (Frontend): "I'll create the Server Action at app/actions/users.ts..."

Kai (Orchestrator): "STOP: Sam, this task is outside your role.

Server Actions are backend logic and require Backend Developer expertise because:
- Business logic implementation
- Database queries with tenant filtering
- Input validation
- Error handling

I'm redirecting this to Morgan (Backend Developer).

Your role is to:
- Build the UI that will call this Server Action
- Handle loading/error states
- Implement form components

@backend Morgan, you're now receiving this handoff from Sam.
Implement createUser Server Action with:
- Zod validation
- Tenant isolation
- RBAC checks
- Error handling

Sam, you can start building the user creation form while Morgan works on the Server Action."
```

---

## Example 3: Parallel Execution with Dependencies

**Scenario:** Building authentication feature

### Dependency Graph:
```
Level 0: Prerequisites (BLOCKING)
└─ Reese: Verify NextAuth MCP server

Level 1: Design (PARALLEL)
├─ Riley: Auth strategy design
├─ Drew: Login/signup UI design
└─ Taylor: User/session schema design

Level 2: Implementation (PARALLEL with handoffs)
├─ Taylor: Create schema → [HANDOFF] → Morgan: Implement auth queries
├─ Morgan: Build auth Server Actions (needs Riley's design)
└─ Sam: Build login UI (needs Drew's design, Morgan's Server Actions)

Level 3: Testing + Deployment
├─ Quinn: Auth tests
└─ Casey: Deploy
```

### Execution:
```
Kai: "Building authentication feature with optimized parallelization."

[Level 0 - Blocking]
@resource-verifier Reese: Verify NextAuth MCP available
Reese: "✓ NextAuth MCP configured. GO."

[Level 1 - All parallel]
Kai: "Starting design phase with 3 agents in PARALLEL:"

@security Riley: Design auth strategy
@designer Drew: Design login/signup UI
@database Taylor: Design user/session schema

[ALL THREE WORK SIMULTANEOUSLY - NO BLOCKING]

Riley: "Auth strategy complete: NextAuth + JWT + RBAC"
Drew: "Login/signup design complete with Shadcn components"
Taylor: "User/session schema designed with tenant isolation"

[Level 2 - Parallel with strategic handoffs]
Kai: "Starting implementation with coordinated handoffs:"

Stream 1 (Database → Backend handoff):
@database Taylor: Create migrations

Taylor: "Migrations complete. Handing off to Morgan."

@backend Morgan: Implement auth queries + Server Actions
[Uses Riley's auth strategy]

Morgan: "Auth implementation complete. Ready for Sam."

Stream 2 (Frontend - works in parallel):
@frontend Sam: Build login UI components
[Uses Drew's design]
[Continues building static components while waiting for Morgan]

Morgan: "Server Actions ready"

Sam: "Integrating Server Actions now. Auth UI complete."

[Level 3 - Testing and deployment]
@qa Quinn: Test auth flows
@devops Casey: Deploy auth feature

Kai: "Authentication feature deployed successfully with 3 hours of work
instead of 8 hours (sequential). 62% faster through parallelization."
```

---

## Example 4: Handling Complex Multi-Agent Dependencies

**Scenario:** Stripe subscription feature

### Agents Involved: 8
- Orchestrator (Kai)
- Payment Specialist (Avery)
- Database Architect (Taylor)
- Backend Developer (Morgan)
- Frontend Developer (Sam)
- Security Expert (Riley)
- QA Engineer (Quinn)
- DevOps Engineer (Casey)

### Coordination:
```
Kai: "Building Stripe subscription feature. This requires coordination of 8 specialists."

[Phase 1 - Design (Parallel)]
@payment Avery: Design Stripe integration (webhooks, subscriptions)
@security Riley: Security review of payment flow
@database Taylor: Design subscription schema
@designer Drew: Design pricing page and checkout flow

[ALL FOUR PARALLEL]

Avery: "Stripe integration designed. Webhook events mapped."
Riley: "Security requirements defined. PCI compliance checklist ready."
Taylor: "Subscription schema ready with tenant linking."
Drew: "Pricing page and checkout UI designed."

[Phase 2 - Implementation (Coordinated handoffs)]
Taylor: "Creating subscription migrations..."
Taylor: "Migrations complete. Handing to Morgan."

@backend Morgan: Implement Stripe webhook handlers
[Morgan coordinates with Avery for webhook verification]

Avery: "Webhook signature verification pattern ready."
Morgan: "Webhook handlers implemented with Avery's pattern."

@frontend Sam: Build pricing page and checkout UI
[Sam uses Drew's design and Morgan's Server Actions]

Sam: "Pricing page complete. Need checkout Server Action."

Morgan: "Checkout Server Action ready with Stripe integration."

Sam: "Checkout flow integrated and working."

[Phase 3 - Testing and Security Review]
@qa Quinn: Test subscription flows
@security Riley: Security audit of payment implementation

Quinn: "All subscription tests passing."
Riley: "Security audit complete. PCI compliant. Approved for production."

[Phase 4 - Deployment]
@devops Casey: Configure Stripe webhook endpoint in production

Casey: "Stripe webhooks configured. Subscription feature deployed."

Kai: "Stripe subscription feature complete.
8 specialists coordinated with proper handoffs.
All security and compliance requirements met."
```

---

## Coordination Patterns Summary

### Pattern 1: Prerequisites Gate
```
Always: Reese (Resource Verifier) → GO/NO-GO → Team proceeds
```

### Pattern 2: Parallel Discovery
```
Alex (PM) + Jordan (Architect) + Drew (Designer) work simultaneously
No blocking - all independent work
```

### Pattern 3: Sequential Handoff
```
Taylor (DB) designs schema → Morgan (Backend) implements queries
MUST happen in order - blocking handoff
```

### Pattern 4: Parallel with Coordination Point
```
Multiple agents work independently →
All complete →
Coordination point (integration) →
Continue
```

### Pattern 5: Continuous Parallel Stream
```
Quinn (QA) and Casey (DevOps) run continuously in parallel with development
Only block at deployment gate
```

---

## Handoff Communication Templates

### Template 1: Clean Handoff
```
[Agent A]: "[Task X] complete. Here's the deliverable:
[Detailed output]

Handing off to [Agent B] for [next phase].

I'm available if [Agent B] has questions."
```

### Template 2: Handoff with Context
```
[Agent A]: "[Task X] requires [Agent B] expertise.

This involves:
- [Requirement 1]
- [Requirement 2]
- [Requirement 3]

Context: [Why this handoff is needed]

Handing off to [Agent B].

I'll continue with [other independent work]."
```

### Template 3: Orchestrator Redirect
```
Kai: "STOP: [Agent], this task is outside your role.

[Task] requires [Specialist] because:
- [Reason 1]
- [Reason 2]

Redirecting to [Specialist].

[Agent], please [what they should do instead]."
```

---

## Best Practices for Coordination

1. **Always Verify Prerequisites First** - Reese runs before anyone else
2. **Maximize Parallelization** - Run simultaneously until dependencies hit
3. **Clear Handoffs** - Explicit communication when passing work
4. **Continue Other Work** - Don't idle while waiting for handoffs
5. **Track All Streams** - Orchestrator monitors all parallel work
6. **Enforce Role Boundaries** - Intercept violations immediately
7. **Batch Handoffs** - Collect multiple needs before handing off
8. **Define Contracts Early** - Clear interfaces enable parallel work
