# Parallel Execution Model

## Core Principle

**Agents work SIMULTANEOUSLY until they need each other.**

**ONLY block when a dependency requires handoff. Otherwise, EVERYTHING runs in PARALLEL.**

## Why Parallel Execution?

### ❌ Sequential (Slow):
```
Reese → Alex → Jordan → Taylor → Morgan → Sam → Quinn → Casey
Total: 8 hours (1 hour each)
```

### ✅ Parallel (Fast):
```
Reese (blocks everyone) → 1 hour

Then simultaneously:
├─ Alex (1 hour)
├─ Jordan (1 hour)
└─ Drew (1 hour)

Then simultaneously:
├─ Morgan (1 hour)
├─ Sam (1 hour)
├─ Casey (1 hour)
└─ Quinn (1 hour)

Total: 3 hours (vs 8 hours!)
```

## Execution Rules

### ✅ Run in Parallel:
- Tasks are independent
- No dependencies between agents
- Different features being built
- Different parts of the system
- Testing while developing
- Documentation while coding

### 🚫 Block Only When:
- Agent A needs Agent B's output to proceed
- Security review required before production
- Architecture must be approved before implementation
- Schema must be designed before queries
- Prerequisites must pass before development

### ⚡ Never Block For:
- Status updates
- Progress reports
- Independent work
- Continuous testing
- Infrastructure maintenance

## Dependency Graph

### Level 0: Prerequisites (BLOCKING)
```
└─ Reese (Resource Verifier)
   Verifies: MCP servers, env vars, database
   Blocks: EVERYTHING
   When done: Gives GO signal → Level 1 starts
```

### Level 1: Discovery (PARALLEL)
```
├─ Alex (PM) - PRD creation [INDEPENDENT]
├─ Jordan (Architect) - Technical assessment [INDEPENDENT]
└─ Drew (Designer) - User flows [INDEPENDENT]

All 3 run simultaneously!
```

### Level 2: Planning (PARALLEL)
```
Triggered by: Alex's PRD completion

├─ Jordan - System architecture [INDEPENDENT]
├─ Taylor - Database design [INDEPENDENT]
├─ Riley - Security design [INDEPENDENT]
├─ Casey - Infrastructure plan [INDEPENDENT]
└─ Avery - Stripe plan (if needed) [INDEPENDENT]

All 5 run simultaneously!
```

### Level 3: Implementation (MAXIMUM PARALLEL)
```
Triggered by: Planning completion

ALL AGENTS WORKING AT ONCE:

Infrastructure Stream (Casey):
├─ Docker setup
├─ CI/CD pipeline
├─ Environment config
└─ Database provisioning
[RUNS ENTIRE TIME - NEVER BLOCKS]

Database Stream (Taylor → Morgan handoff):
├─ Taylor: Create schema
├─ Taylor: Generate migrations
└─ Morgan: Implement queries (after schema ready)
[HANDOFF when schema complete]

Backend Stream (Morgan):
├─ Server Actions [INDEPENDENT]
├─ API routes [INDEPENDENT]
├─ Validation [INDEPENDENT]
├─ Auth (needs Riley's design) [HANDOFF]
└─ Stripe (needs Avery's design) [HANDOFF]
[MOSTLY PARALLEL, occasional handoffs]

Frontend Stream (Sam):
├─ Next.js setup [INDEPENDENT]
├─ Shadcn UI install [INDEPENDENT]
├─ Layouts [INDEPENDENT]
├─ Components (needs Drew's designs) [HANDOFF]
└─ Integration (needs Morgan's actions) [HANDOFF]
[MOSTLY PARALLEL, occasional handoffs]

Testing Stream (Quinn):
├─ Test setup [INDEPENDENT]
├─ Unit tests (as code completes)
├─ Integration tests (as APIs complete)
└─ E2E tests (as flows complete)
[CONTINUOUS PARALLEL - NEVER BLOCKS DEV]

Security Stream (Riley):
└─ Reviews (as features complete)
[CONTINUOUS PARALLEL - ONLY BLOCKS DEPLOYMENT]
```

### Level 4: QA (PARALLEL WITH DEVELOPMENT)
```
Quinn writes tests while Morgan/Sam develop
Riley reviews security while development continues
Only blocks at production deployment gate
```

### Level 5: Deployment (COORDINATED)
```
Prerequisites (must complete):
├─ All tests passing (Quinn)
├─ Security review passed (Riley)
└─ Code review complete

Then:
└─ Casey deploys + monitors
```

## Handoff Optimization

### 1. Continue Other Work
**Principle:** Don't stop when waiting for handoff

❌ **WRONG:**
```
Sam: "I need Morgan's auth Server Action"
Sam: *stops all work and waits*
```

✅ **RIGHT:**
```
Sam: "I need Morgan's auth Server Action"
Sam: "Handing off to Morgan"
Sam: *continues building other UI components*
Morgan: *implements auth action*
Morgan: "Auth action ready for Sam"
Sam: *integrates when convenient*
```

### 2. Batch Handoffs
**Principle:** Collect multiple needs before handing off

❌ **WRONG:**
```
Sam → Morgan: "Need createUser action"
*waits*
Sam → Morgan: "Need updateUser action"
*waits*
Sam → Morgan: "Need deleteUser action"
```

✅ **RIGHT:**
```
Sam → Morgan: "Need complete user CRUD:
- createUser
- getUser
- updateUser
- deleteUser
- listUsers

All with tenant isolation and RBAC checks."

Morgan: *implements all at once*
Morgan → Sam: "Complete user CRUD ready"
Sam: *integrates all together*
```

### 3. Async Communication
**Principle:** Hand off and keep working, don't wait synchronously

```
09:00 Sam: "Morgan, I need these 5 Server Actions [list]"
09:01 Sam: *continues building layouts*
09:05 Sam: *builds navigation*
09:10 Sam: *implements forms*
09:15 Morgan: "Server Actions complete"
09:20 Sam: "Thanks! Integrating now"
```

### 4. Clear Interfaces
**Principle:** Define contracts early so parallel work is possible

```
Jordan defines API contracts:
- POST /api/users → { name, email, role }
- GET /api/users/:id → User object
- PUT /api/users/:id → Updated user
- DELETE /api/users/:id → Success

Morgan builds backend to contract
Sam builds frontend to contract
↓
Both work in parallel
↓
Integration is seamless (contract was clear)
```

## Real-World Example

### Scenario: Add User Management Feature

#### Sequential (Slow) - 8 hours:
```
1. Alex writes PRD (1 hour) → waits
2. Jordan designs API (1 hour) → waits
3. Taylor designs schema (1 hour) → waits
4. Morgan implements backend (2 hours) → waits
5. Sam implements frontend (2 hours) → waits
6. Quinn writes tests (1 hour)

Total: 8 hours
```

#### Parallel (Fast) - 3 hours:
```
HOUR 1 (Parallel):
├─ Alex: PRD ✓
├─ Jordan: API design ✓
└─ Drew: UI mockups ✓

HOUR 2 (Parallel):
├─ Taylor: Schema design ✓
├─ Casey: Infrastructure ✓
└─ Quinn: Test setup ✓

HOUR 3 (Parallel):
├─ Morgan: Backend implementation
│   ├─ Gets schema from Taylor ← handoff
│   ├─ Implements CRUD
│   └─ Adds validation
│
├─ Sam: Frontend implementation
│   ├─ Builds UI structure
│   ├─ Gets Server Actions from Morgan ← handoff
│   └─ Integrates
│
└─ Quinn: Writes tests (continuous)

Total: 3 hours
```

## Parallel Coordination Strategies

### Strategy 1: Independent Features
```
Feature A: User Profiles
Feature B: Dashboard Analytics

These have NO dependencies!

Parallel execution:
├─ Sam builds Profile UI
├─ Sam builds Dashboard UI
├─ Morgan builds Profile API
└─ Morgan builds Dashboard API

All can happen simultaneously.
No handoffs between features.
```

### Strategy 2: Layered Architecture
```
Layer 1 (Foundation):
└─ Taylor: Complete schema for entire app [BLOCKS]

Layer 2 (Parallel):
├─ Morgan: ALL Server Actions
├─ Sam: ALL UI components (mock data)
├─ Casey: ALL infrastructure
└─ Drew: ALL design refinements

Layer 3 (Integration):
└─ Sam: Integrate Morgan's actions with UI
```

### Strategy 3: Feature Slicing
```
Feature: Authentication

Planning Phase (Parallel):
├─ Riley: Auth strategy
├─ Drew: Login UI design
└─ Casey: Auth infrastructure

Implementation Phase (Parallel):
├─ Morgan: Auth backend (using Riley's design)
├─ Sam: Login UI (using Drew's design)
└─ Quinn: Auth tests

Integration Phase:
└─ Team: Connect backend + frontend
```

### Strategy 4: Continuous Integration
```
Stream 1: Development (Morgan + Sam)
├─ Building features
└─ Never blocked by testing

Stream 2: Testing (Quinn)
├─ Writing tests
├─ Running tests
└─ Never blocks development

Stream 3: Infrastructure (Casey)
├─ Maintaining systems
├─ Preparing deployment
└─ Never blocks development

Only synchronize at deployment gate.
```

## Orchestrator's Parallel Management

### Responsibilities:
1. **Identify** all independent work
2. **Start** all parallel streams simultaneously
3. **Track** progress of each stream
4. **Coordinate** handoffs when needed
5. **Keep agents working** - never idle
6. **Block only when required**

### Communication Pattern:
```
Kai: "Starting parallel execution:

Currently working:
├─ Sam: Building dashboard UI [NO BLOCKERS]
├─ Morgan: Implementing CRUD actions [NO BLOCKERS]
├─ Taylor: Creating migrations [NO BLOCKERS]
├─ Casey: Setting up CI/CD [NO BLOCKERS]
└─ Quinn: Writing test utilities [NO BLOCKERS]

All agents: Proceed independently.
Hand off when you need each other.
I'll coordinate handoffs as they occur."
```

### Tracking Status:
```
Kai monitors:

[IN PROGRESS - PARALLEL]
- Sam: Dashboard UI (70% complete)
- Morgan: User CRUD (50% complete)
- Casey: Docker setup (90% complete)

[HANDOFF PENDING]
- Sam needs Morgan's auth action
  → Morgan working on it, ETA 10 mins
  → Sam continues other components

[COMPLETED]
- Taylor: Schema ✓
- Drew: Dashboard design ✓
```

## Handoff Examples

### Example 1: Schema → Queries
```
Taylor: "Schema design complete. Here's the Prisma schema:

[full schema with tenant isolation, indexes, relationships]

Handing off to Morgan for query implementation."

Taylor: *continues working on caching strategy*

Morgan: "Received schema. Implementing queries now."

Morgan: *implements all queries*

Morgan: "Queries complete. Available for Sam to use in Server Actions."
```

### Example 2: Backend → Frontend
```
Morgan: "Auth Server Actions complete:

- signIn(email, password)
- signOut()
- signUp(userData)
- resetPassword(email)

All include tenant context and RBAC checks.

Handing off to Sam for UI integration."

Morgan: *continues working on user CRUD*

Sam: "Received auth actions. Integrating into login UI."

Sam: *integrates while Morgan works on other features*
```

### Example 3: Design → Implementation
```
Drew: "Dashboard design complete. Here's the full spec:

[Detailed component specs, colors, spacing, interactions]

Handing off to Sam for implementation."

Drew: *continues refining other designs*

Sam: "Received dashboard design. Implementing now."

Sam: *builds dashboard while Drew works on other pages*
```

## Parallel Execution Checklist

### Before Starting:
- [ ] Reese verified all prerequisites ✓
- [ ] Identified all independent tasks
- [ ] Created dependency map
- [ ] Planned handoff points

### During Execution:
- [ ] All agents working simultaneously (when possible)
- [ ] No unnecessary blocking
- [ ] Handoffs coordinated smoothly
- [ ] Agents continue other work during waits
- [ ] Progress tracked for all streams

### At Handoff Points:
- [ ] Clear communication of what's being handed off
- [ ] Receiving agent has context needed
- [ ] Sending agent continues other work
- [ ] Orchestrator tracks handoff completion

### At Completion:
- [ ] All parallel streams completed
- [ ] All handoffs successful
- [ ] No work was unnecessarily sequential
- [ ] Team achieved maximum efficiency

## Benefits

✅ **Speed**: Features delivered 2-5x faster
✅ **Efficiency**: No idle agents waiting
✅ **Coordination**: Handoffs are smooth and intentional
✅ **Quality**: Specialists do their best work without rushing
✅ **Predictability**: Clear dependency graph, no surprises

## Summary

**The team operates like a well-orchestrated symphony:**

- Each musician (agent) plays their part
- Most play simultaneously (parallel)
- Some wait for cues (handoffs)
- Conductor (Orchestrator) coordinates
- Result: Beautiful music (great software) delivered fast

**Key Rules:**
1. Work in parallel by default
2. Block only when dependency requires it
3. Continue other work while waiting for handoffs
4. Orchestrator coordinates, doesn't micromanage
5. Maximum efficiency, minimum idle time

**Your team is now optimized for speed through parallelization!** 🚀
