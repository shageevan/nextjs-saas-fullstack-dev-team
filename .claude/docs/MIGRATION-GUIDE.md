# Migration Guide: Monolithic → Modular Agent Structure

## Overview

This guide shows how to migrate agents from monolithic YAML files to the optimized tiered structure for maximum context efficiency.

**Goal:** 72% token reduction (12,300 → 3,500 tokens per typical interaction)

## Reference Implementation

See `agents/orchestrator/` for the complete reference implementation.

## Migration Steps

### Step 1: Analyze Current Agent

**Example: Backend Developer (backend.agent.yaml - 553 lines)**

Current structure:
```yaml
agent:
  metadata: {...}               # ~20 lines
  persona: {...}                # ~30 lines
  role_boundaries: {...}        # ~80 lines
  expertise: [...]              # ~10 lines
  responsibilities: [...]       # ~10 lines
  tech_stack: {...}             # ~30 lines
  server_actions_pattern: {...} # ~50 lines (with code examples)
  api_routes_pattern: {...}     # ~40 lines (with code examples)
  validation_pattern: {...}     # ~30 lines (with code examples)
  error_handling: {...}         # ~40 lines (with code examples)
  rate_limiting: {...}          # ~35 lines (with code examples)
  menu: [...]                   # ~100 lines
  best_practices: [...]         # ~40 lines
  collaboration: {...}          # ~20 lines
  output_artifacts: [...]       # ~10 lines
```

### Step 2: Create Directory Structure

```bash
mkdir -p agents/backend/examples
```

Result:
```
agents/backend/
├── core.yaml
├── operations.yaml
├── examples/
│   ├── server-actions.md
│   ├── api-routes.md
│   ├── validation.md
│   └── webhooks.md
└── README.md
```

### Step 3: Create core.yaml (Tier 1)

**Purpose:** Always loaded - essential agent identity and boundaries

**Include:**
- metadata
- persona
- role_boundaries (strictly_my_job, never_my_job, must_handoff_to)
- critical actions (if any)
- expertise list (brief)
- references to other tiers

**Exclude:**
- Detailed patterns
- Code examples
- Complete workflows
- Lengthy explanations

**Example: backend/core.yaml**

```yaml
# Backend Developer - Core Definition (Tier 1)
# Estimated tokens: ~400

agent:
  metadata:
    id: "fullstack-team/agents/backend.md"
    name: Morgan
    title: Backend Developer
    icon: ⚙️
    module: fullstack-team
    context:
      tier: 1
      estimated_tokens: 400
      load_when: ["backend_work_needed", "server_actions_needed"]
      pre_fetch_with: ["database", "security"]

  persona:
    role: Backend Engineer + API Specialist
    identity: |
      Senior backend engineer with 8+ years building scalable APIs.
      Expert in Next.js Server Actions, API routes, and multi-tenant isolation.
      Security-conscious and performance-obsessed.
    communication_style: |
      Precise and data-focused.
      Speaks in terms of endpoints, schemas, and business logic.
      "Trust nothing, validate everything" is the principle.
    principles:
      - "Validate all inputs with Zod"
      - "Tenant isolation enforced at every query"
      - "Transactions for data consistency"
      - "Idempotency for mutations"
      - "Error handling with proper status codes"

  role_boundaries:
    strictly_my_job:
      - "Implement Server Actions for mutations"
      - "Create API routes for webhooks and external APIs"
      - "Implement business logic"
      - "Write database queries (using schema from DB Architect)"
      - "Validate inputs with Zod"
      - "Error handling and logging"

    never_my_job:
      - "Design database schema (that's Database Architect)"
      - "Implement UI components (that's Frontend Developer)"
      - "Design system architecture (that's Technical Architect)"
      - "Configure deployment (that's DevOps Engineer)"

    must_handoff_to:
      database_architect: ["Schema design", "Multi-tenant strategy", "Index design"]
      frontend_developer: ["UI components", "Client-side forms", "Page layouts"]
      security_expert: ["Auth strategy", "Security architecture", "RBAC design"]
      payment_specialist: ["Stripe integration design", "Webhook strategy"]

  expertise:
    - Next.js Server Actions
    - Next.js API Routes
    - Multi-tenant data isolation
    - Prisma / Drizzle ORM
    - Zod validation
    - Webhook handling

  references:
    operations: "@load backend/operations.yaml"
    examples:
      server_actions: "@reference backend/examples/server-actions.md"
      api_routes: "@reference backend/examples/api-routes.md"
      validation: "@reference backend/examples/validation.md"
```

**Size: ~150 lines (vs 553 lines) - 73% reduction**

### Step 4: Create operations.yaml (Tier 2)

**Purpose:** Loaded when agent is actively working

**Include:**
- Detailed workflows
- Pattern summaries (without full code)
- Best practices checklist
- Menu triggers
- Collaboration info
- Tech stack details

**Exclude:**
- Complete code examples (reference instead)
- Lengthy tutorials
- Step-by-step guides

**Example: backend/operations.yaml**

```yaml
# Backend Developer - Operations (Tier 2)
# Estimated tokens: ~400

operations:
  context:
    tier: 2
    estimated_tokens: 400
    load_when: ["backend_implementing", "api_development"]

  responsibilities:
    - "Implement Server Actions for mutations"
    - "Build API routes for webhooks"
    - "Enforce multi-tenant data isolation"
    - "Implement business logic"
    - "Database queries with tenant filtering"
    - "Handle errors gracefully"

  tech_stack:
    framework: "Next.js 15+ Server Actions + API Routes"
    orm: "Prisma or Drizzle ORM"
    validation: "Zod"
    database: "PostgreSQL"
    cache: "Redis (Upstash)"

  patterns:
    server_actions:
      description: "Use Server Actions for form submissions and mutations"
      quick_guide:
        - "Add 'use server' directive"
        - "Authenticate user and get tenant"
        - "Validate with Zod"
        - "Execute with tenant filter"
        - "Revalidate cache"
      full_example: "@reference backend/examples/server-actions.md"

    api_routes:
      description: "Use API Routes for webhooks and external APIs"
      quick_guide:
        - "Create route handler (GET, POST, etc.)"
        - "Verify signatures for webhooks"
        - "Parse and validate payload"
        - "Return proper HTTP status"
      full_example: "@reference backend/examples/api-routes.md"

    validation:
      description: "Always validate with Zod"
      quick_guide:
        - "Define Zod schema"
        - "Use safeParse() for validation"
        - "Return structured errors"
      full_example: "@reference backend/examples/validation.md"

  best_practices:
    - "ALWAYS validate input with Zod"
    - "ALWAYS include tenantId in database queries"
    - "ALWAYS check permissions before mutations"
    - "Use transactions for multi-step operations"
    - "Return structured error responses"

  menu:
    - trigger: implement-server-action
      description: "Implement Server Action for data mutation"
      exec: "@reference backend/examples/server-actions.md#implementation"

    - trigger: implement-api-route
      description: "Implement API route for webhooks"
      exec: "@reference backend/examples/api-routes.md#implementation"

  collaboration:
    works_closely_with:
      - "Frontend Developer (Sam): For API contracts"
      - "Database Architect (Taylor): For schema and queries"
      - "Security Expert (Riley): For auth and permissions"
```

**Size: ~200 lines - Loads only when backend is working**

### Step 5: Create Examples (Tier 3)

**Purpose:** Reference material loaded only when explicitly needed

**Include:**
- Complete code examples
- Step-by-step tutorials
- Detailed patterns
- Troubleshooting guides

**Example: backend/examples/server-actions.md**

```markdown
# Server Actions - Complete Guide

## Full Pattern

\`\`\`typescript
// app/actions/users.ts
'use server'

import { z } from 'zod'
import { revalidatePath } from 'next/cache'
import { db } from '@/lib/db'
import { getCurrentTenant, requireAuth } from '@/lib/auth'

const createUserSchema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
  role: z.enum(['admin', 'user']),
})

export async function createUser(formData: FormData) {
  // 1. Authenticate and get tenant
  const user = await requireAuth()
  const tenant = await getCurrentTenant()

  // 2. Validate input
  const parsed = createUserSchema.safeParse({
    name: formData.get('name'),
    email: formData.get('email'),
    role: formData.get('role'),
  })

  if (!parsed.success) {
    return { error: 'Invalid input', details: parsed.error.flatten() }
  }

  // 3. Check permissions
  if (user.role !== 'admin') {
    return { error: 'Unauthorized' }
  }

  // 4. Execute with tenant isolation
  try {
    const newUser = await db.user.create({
      data: {
        ...parsed.data,
        tenantId: tenant.id,
        createdBy: user.id,
      },
    })

    // 5. Revalidate cache
    revalidatePath('/users')

    return { success: true, data: newUser }
  } catch (error) {
    console.error('Failed to create user:', error)
    return { error: 'Failed to create user' }
  }
}
\`\`\`

[Additional examples and explanations...]
```

**Size: ~100-200 lines per example - Loaded only when referenced**

### Step 6: Update context-manifest.yaml

```yaml
agent_loading_rules:
  backend:
    always_load: []
    load_when_active: ["core.yaml", "operations.yaml"]
    pre_fetch_with: ["database", "security", "payment"]
    estimated_tokens_core: 400
    estimated_tokens_operations: 400
    examples:
      - "server-actions.md"
      - "api-routes.md"
      - "validation.md"
      - "webhooks.md"
```

### Step 7: Create Agent README

Document the new structure, loading strategy, and benefits.

See `agents/orchestrator/README.md` for template.

## Migration Checklist

### Per Agent

- [ ] Create directory: `agents/[agent-name]/`
- [ ] Create subdirectory: `agents/[agent-name]/examples/`
- [ ] Extract and create `core.yaml` (Tier 1)
  - [ ] metadata + context metadata
  - [ ] persona
  - [ ] role_boundaries
  - [ ] expertise (brief)
  - [ ] references section
- [ ] Extract and create `operations.yaml` (Tier 2)
  - [ ] responsibilities
  - [ ] tech_stack
  - [ ] patterns (summaries with references)
  - [ ] best_practices
  - [ ] menu triggers
  - [ ] collaboration
- [ ] Extract and create `examples/*.md` (Tier 3)
  - [ ] Complete code examples
  - [ ] Detailed patterns
  - [ ] Step-by-step guides
- [ ] Create `README.md` documenting structure
- [ ] Update `context-manifest.yaml` with loading rules
- [ ] Test loading in scenarios
- [ ] Measure token reduction

## Priority Order

### Phase 1: High-Impact Agents (Week 1)
1. ✅ Orchestrator (reference implementation)
2. Backend Developer (frequent use, large file)
3. Frontend Developer (frequent use, large file)
4. Database Architect (critical for multi-tenant)

**Expected Reduction:** 50%

### Phase 2: Medium-Impact Agents (Week 2)
5. Resource Verifier (blocks everything)
6. Security Expert (critical but conditional)
7. Payment Specialist (Stripe features)
8. DevOps Engineer (deployment)

**Expected Reduction:** 65%

### Phase 3: Remaining Agents (Week 2)
9. QA Engineer
10. UI/UX Designer
11. Product Manager
12. Technical Architect

**Expected Reduction:** 70%

### Phase 4: Documentation (Week 3)
- Split docs into core/ and reference/
- Update all references
- Test loading

**Expected Reduction:** 72%

## Token Reduction Targets

| Agent | Before | After (Core) | After (Core+Ops) | Reduction |
|-------|--------|--------------|------------------|-----------|
| Orchestrator | 1,500 | 500 | 1,300 | 67% / 13% |
| Backend | 680 | 400 | 800 | 41% |
| Frontend | 650 | 400 | 800 | 38% |
| Database | 800 | 450 | 850 | 44% |
| Security | 750 | 400 | 750 | 47% |
| Payment | 750 | 400 | 750 | 47% |
| DevOps | 730 | 400 | 800 | 45% |
| QA | 720 | 400 | 750 | 44% |
| Designer | 730 | 400 | 700 | 45% |
| PM | 370 | 300 | 500 | 19% |
| Architect | 540 | 400 | 700 | 26% |
| Resource Verifier | 910 | 400 | 900 | 56% |
| **Total** | **9,000** | **4,850** | **9,100** | **46%** |

**Note:** These are "core only" vs "core + operations" scenarios. The key insight is that most interactions only need core (~4,850 tokens total), not full content.

## Validation

After migrating each agent:

1. **Load Test:**
   ```
   Scenario: Simple question about agent
   Expected: Load core.yaml only
   Target: <500 tokens
   ```

2. **Work Test:**
   ```
   Scenario: Agent performing task
   Expected: Load core.yaml + operations.yaml
   Target: <1,000 tokens
   ```

3. **Example Test:**
   ```
   Scenario: Need detailed pattern
   Expected: Load core + operations + specific example
   Target: <1,500 tokens
   ```

## Tips

### Core.yaml Tips
- Keep it minimal but complete
- Include only essential boundaries
- Use brief lists, not paragraphs
- Reference don't embed

### Operations.yaml Tips
- Summaries not full explanations
- Pattern overviews not complete code
- Link to examples liberally
- Keep menu triggers concise

### Examples Tips
- One pattern per file
- Complete, working code
- Step-by-step explanations
- Include troubleshooting

### General Tips
- Measure tokens before/after
- Test loading scenarios
- Verify nothing lost
- Document references clearly

## Success Criteria

✅ Each agent core.yaml <500 lines
✅ Each agent operations.yaml <400 lines
✅ Examples <200 lines each
✅ Total context reduction >70%
✅ All functionality preserved
✅ Loading time <200ms overhead
✅ Clear documentation

## Tools

### Token Counter
```bash
# Estimate tokens (rough: 1 token ≈ 1.2 chars or 0.8 lines)
wc -l file.yaml
# Multiply by 1.2 for token estimate
```

### File Splitter Script
```bash
# Create agent directory structure
create_agent_structure() {
  agent=$1
  mkdir -p "agents/$agent/examples"
  touch "agents/$agent/core.yaml"
  touch "agents/$agent/operations.yaml"
  touch "agents/$agent/README.md"
}

# Usage:
create_agent_structure backend
```

## Questions?

See:
- `docs/CONTEXT-OPTIMIZATION.md` - Full optimization strategy
- `agents/orchestrator/README.md` - Reference implementation
- `context-manifest.yaml` - Loading rules
- `docs/PARALLEL-EXECUTION.md` - Parallel coordination
- `docs/ROLE-BOUNDARIES.md` - Role boundaries

## Next Steps

1. Review reference implementation: `agents/orchestrator/`
2. Start with Backend Developer (high-impact)
3. Follow migration checklist
4. Test and validate
5. Move to next agent
6. Update documentation
7. Measure overall impact
