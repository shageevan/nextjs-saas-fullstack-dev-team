# Context Optimization Strategy

## Executive Summary

**Current Token Usage:** ~12,300 tokens (if all content loaded)
**Optimized Token Usage:** ~3,500 tokens (72% reduction)
**ROI Impact:** 3.5x more efficient context utilization

## Problem Statement

The current agent system contains comprehensive knowledge that, while valuable, creates context inefficiency:

- **12 agent files**: 7,461 lines (~9,000 tokens)
- **4 documentation files**: 2,227 lines (~3,300 tokens)
- **Total baseline**: ~12,300 tokens

Loading all content for every interaction is wasteful. Most interactions need only 20-30% of available context.

## Optimization Strategy

### 1. Tiered Loading Architecture

**Tier 1: Core Definition (Always Loaded)**
- Agent metadata (id, name, title, icon)
- Persona (role, identity, communication style)
- Role boundaries (strictly_my_job, never_my_job, must_handoff_to)
- Critical actions
- **Estimated**: ~500 tokens per agent

**Tier 2: Operational Context (Load on Demand)**
- Coordination phases
- Best practices summaries
- Handoff protocols
- Collaboration info
- **Estimated**: ~300 tokens per agent

**Tier 3: Reference Materials (Load as Needed)**
- Detailed examples
- Complete patterns
- Full documentation
- Code snippets
- **Estimated**: ~400 tokens per agent

### 2. File Structure Refactoring

#### Current Structure (Monolithic)
```
agents/
  orchestrator.agent.yaml    # 1,232 lines - everything in one file
  backend.agent.yaml         # 553 lines - everything in one file
  ...
```

#### Optimized Structure (Modular)
```
agents/
  orchestrator/
    core.yaml                # Tier 1: Always loaded (~200 lines)
    operations.yaml          # Tier 2: On-demand (~300 lines)
    examples/
      coordination.md        # Tier 3: Reference (~250 lines)
      handoffs.md           # Tier 3: Reference (~250 lines)
      parallel.md           # Tier 3: Reference (~230 lines)

  backend/
    core.yaml               # Tier 1: Always loaded (~150 lines)
    operations.yaml         # Tier 2: On-demand (~200 lines)
    examples/
      server-actions.md     # Tier 3: Reference (~100 lines)
      api-routes.md         # Tier 3: Reference (~100 lines)
```

### 3. Context Loading Rules

**Rule 1: Load Core Always**
```yaml
# Every interaction loads:
- agent.core (metadata + persona + role_boundaries)
- Total: ~500 tokens per active agent
```

**Rule 2: Load Operations on Trigger**
```yaml
# Load operations.yaml when:
- Agent is actively performing task
- Agent receives handoff
- Orchestrator coordinates agent

# Example: Frontend agent starts work
Load: frontend/core.yaml + frontend/operations.yaml
Skip: frontend/examples/* (unless referenced)
```

**Rule 3: Load Examples on Reference**
```yaml
# Load examples/*.md only when:
- Agent explicitly needs pattern
- User requests example
- Error occurs and example aids debugging

# Example: Backend implements Server Action
Load: backend/examples/server-actions.md
Skip: backend/examples/api-routes.md (not needed yet)
```

**Rule 4: Smart Pre-fetching**
```yaml
# Pre-fetch likely next agents based on workflow:
Orchestrator active → Pre-fetch: resource-verifier/core.yaml
Resource Verifier done → Pre-fetch: pm/core.yaml, architect/core.yaml
Planning complete → Pre-fetch: backend/core.yaml, frontend/core.yaml, database/core.yaml
```

### 4. Documentation Optimization

**Current Docs:**
- AGENTS.md: 745 lines (all agent descriptions)
- ROLE-BOUNDARIES.md: 541 lines (complete handoff guide)
- PARALLEL-EXECUTION.md: 501 lines (full parallel guide)
- QUICKSTART.md: 440 lines (getting started)

**Optimization:**

```markdown
docs/
  core/
    AGENTS-SUMMARY.md         # 150 lines - Quick reference only
    ROLE-BOUNDARIES-CORE.md   # 100 lines - Core principles only
    PARALLEL-CORE.md          # 100 lines - Key concepts only
    QUICKSTART.md            # 440 lines - Keep as-is (entry point)

  reference/
    AGENTS-DETAILED.md        # 595 lines - Full descriptions
    ROLE-BOUNDARIES-EXAMPLES.md # 441 lines - All examples
    PARALLEL-EXAMPLES.md      # 401 lines - Detailed scenarios
```

**Loading Strategy:**
1. Load `core/` docs first (~350 lines / ~525 tokens)
2. Load `reference/` docs only when explicitly needed
3. Reduction: 2,227 lines → 790 lines loaded normally (65% reduction)

### 5. Context Compression Techniques

#### Technique 1: Reference Pointers
**Before:**
```yaml
server_actions_pattern:
  description: "Use Server Actions for form submissions and mutations"
  example: |
    // app/actions/users.ts
    'use server'

    import { z } from 'zod'
    import { revalidatePath } from 'next/cache'
    # ... 50 more lines of example code
```

**After:**
```yaml
server_actions_pattern:
  description: "Use Server Actions for form submissions and mutations"
  example: "@reference backend/examples/server-actions.md"
  quick_example: "See: app/actions/users.ts with Zod validation + tenant isolation"
```

**Savings:** 50 lines → 3 lines (94% reduction per pattern)

#### Technique 2: Summary + Details
**Before:**
```yaml
best_practices:
  - "ALWAYS validate input with Zod"
  - "ALWAYS include tenantId in database queries"
  # ... 15 more practices with explanations (200 lines)
```

**After (Tier 1):**
```yaml
best_practices:
  summary: "10 critical practices: input validation, tenant isolation, auth checks, transactions, error handling, logging, rate limiting, webhook verification"
  details: "@reference backend/operations.yaml#best_practices_detailed"
```

**After (Tier 2 - operations.yaml):**
```yaml
best_practices_detailed:
  - practice: "Input Validation"
    rule: "ALWAYS validate with Zod"
    why: "Prevent injection attacks and data corruption"
    example: "@reference backend/examples/validation.md"
  # ... compact list
```

#### Technique 3: Pattern Templates
**Before:** Full code examples inline (100+ lines each)

**After:** Pattern templates with placeholders
```yaml
pattern_template: "server_action"
template_url: "@template server-action.ts"
customizations:
  - "Replace {{ENTITY}} with your entity name"
  - "Add tenant filtering: where: { tenantId }"
  - "Validate with Zod schema"
```

### 6. Implementation Plan

#### Phase 1: Restructure Files (High Impact)
```bash
# For each agent, split into:
agents/[agent-name]/
  core.yaml           # Tier 1: ~150-200 lines
  operations.yaml     # Tier 2: ~200-300 lines
  examples/          # Tier 3: Multiple small files

# Savings per agent: Load 150 lines instead of 550 lines (73% reduction)
```

#### Phase 2: Optimize Documentation (Medium Impact)
```bash
docs/
  core/              # Always loaded: ~790 lines total
  reference/         # Load on-demand: ~1,437 lines total

# Savings: Load 790 lines instead of 2,227 lines (65% reduction)
```

#### Phase 3: Add Context Metadata (Enables Smart Loading)
```yaml
# Add to each file:
context:
  tier: 1               # 1 = always, 2 = on-demand, 3 = reference
  estimated_tokens: 500
  pre_fetch_with: ["resource-verifier", "pm"]
  load_when:
    - "agent starts work"
    - "receives handoff"
```

#### Phase 4: Create Loading Manifest
```yaml
# context-manifest.yaml
loading_strategy:
  on_conversation_start:
    load: ["orchestrator/core.yaml"]
    estimated_tokens: 500

  on_orchestrator_active:
    load: ["orchestrator/operations.yaml"]
    pre_fetch: ["resource-verifier/core.yaml"]
    estimated_tokens: 800

  on_agent_handoff:
    load: ["[target-agent]/core.yaml", "[target-agent]/operations.yaml"]
    estimated_tokens: 400
```

### 7. ROI Analysis

#### Current State (Worst Case)
```
Scenario: User asks "Build a user management feature"

Context Loaded:
- All 12 agent files: ~9,000 tokens
- All 4 docs: ~3,300 tokens
- Total: ~12,300 tokens

Agents Actually Used: 5 (Orchestrator, PM, Architect, Backend, Frontend)
Wasted Context: 58% (7 unused agents + full docs)
```

#### Optimized State (Best Case)
```
Scenario: User asks "Build a user management feature"

Context Loaded:
Phase 1 (Planning):
- orchestrator/core.yaml: 500 tokens
- orchestrator/operations.yaml: 300 tokens
- pm/core.yaml: 400 tokens
- architect/core.yaml: 400 tokens
- core docs: 525 tokens
Total Phase 1: 2,125 tokens

Phase 2 (Implementation):
- backend/core.yaml: 400 tokens
- backend/operations.yaml: 300 tokens
- frontend/core.yaml: 400 tokens
- frontend/operations.yaml: 300 tokens
Total Phase 2: 1,400 tokens

Total Conversation: ~3,500 tokens average
Reduction: 72% (12,300 → 3,500)
```

#### ROI Metrics

| Metric | Current | Optimized | Improvement |
|--------|---------|-----------|-------------|
| Average tokens/interaction | 12,300 | 3,500 | **72% reduction** |
| Context efficiency | 42% | 95% | **2.3x better** |
| Cost per 1M tokens (GPT-4) | $30 | $8.50 | **$21.50 savings** |
| Response latency | Baseline | -40% | **Faster** |
| Unused context | 58% | 15% | **43% improvement** |

### 8. Additional Optimizations

#### A. Incremental Context Loading
```yaml
# Load progressively as conversation deepens
Turn 1: Load orchestrator/core.yaml (500 tokens)
Turn 2: Add orchestrator/operations.yaml (300 tokens)
Turn 3: Add active agent cores (400 tokens each)
Turn 4: Add examples if referenced (100-200 tokens)
```

#### B. Context Caching (with LLM prompt caching)
```yaml
# Cache stable content across conversations
Cacheable:
  - All agent core.yaml files (Tier 1)
  - Core documentation
  - Pattern templates

Non-cacheable:
  - User-specific context
  - Project files
  - Conversation history

# With caching: 50-90% cost reduction on cached content
```

#### C. Smart Context Eviction
```yaml
# Remove context when no longer needed
Rules:
  - Remove agent context when handoff completes
  - Keep orchestrator always
  - Remove examples after use
  - Compress history after 10 turns
```

#### D. Compression via Summarization
```yaml
# After agent completes work, compress to summary
Before (full context): 800 tokens
After (summary): 100 tokens

Example:
Full: "Backend Developer implemented 5 Server Actions with Zod validation,
       tenant filtering, error handling, and rate limiting. Code at..."
Summary: "✓ Backend: 5 Server Actions implemented with security checks"
```

### 9. Implementation Checklist

**Week 1: Restructure**
- [ ] Split orchestrator into core + operations + examples
- [ ] Split backend into core + operations + examples
- [ ] Split frontend into core + operations + examples
- [ ] Test tiered loading with orchestrator

**Week 2: Optimize Remaining Agents**
- [ ] Split database, security, payment agents
- [ ] Split devops, qa, designer agents
- [ ] Split pm, architect, resource-verifier agents
- [ ] Verify all agents work with new structure

**Week 3: Documentation + Metadata**
- [ ] Split docs into core/ and reference/
- [ ] Add context metadata to all files
- [ ] Create context-manifest.yaml
- [ ] Create loading strategy rules

**Week 4: Testing + Refinement**
- [ ] Test with real scenarios
- [ ] Measure token reduction
- [ ] Optimize further based on usage
- [ ] Document best practices

### 10. Success Metrics

**Target Metrics:**
- ✅ 70%+ reduction in average context size
- ✅ 90%+ context utilization efficiency
- ✅ <4,000 tokens per typical interaction
- ✅ <200ms additional latency for context loading
- ✅ 50%+ cost savings on API calls

**Monitoring:**
```yaml
# Add to each agent interaction
metrics:
  tokens_loaded: 3,420
  tokens_used: 3,250
  efficiency: 95%
  agents_active: 3
  context_wasted: 5%
```

## Summary

**Current State:**
- Monolithic agent files
- All content loaded always
- ~12,300 tokens per interaction
- 58% wasted context

**Optimized State:**
- Modular agent files (core + operations + examples)
- Tiered loading (load only what's needed)
- ~3,500 tokens per interaction
- 15% wasted context

**Impact:**
- **72% token reduction**
- **3.5x better efficiency**
- **$21.50 saved per 1M tokens**
- **Faster response times**
- **Better context focus**

**Next Steps:**
1. Implement file restructuring (Phase 1)
2. Add context metadata (Phase 3)
3. Test and refine loading strategy
4. Monitor and optimize based on real usage

This optimization ensures maximum ROI on context size while maintaining the comprehensive knowledge base that makes the agents effective.
