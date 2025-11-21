# Orchestrator Agent - Modular Structure

This agent has been restructured using the **tiered loading architecture** for optimal context efficiency.

## Structure

```
orchestrator/
├── core.yaml                    # Tier 1: Always loaded (~500 tokens)
├── operations.yaml              # Tier 2: On-demand (~800 tokens)
├── examples/                    # Tier 3: Reference only
│   └── coordination.md          # (~400 tokens when needed)
└── README.md                    # This file
```

## Loading Strategy

### Tier 1: Core (Always Loaded)
**File:** `core.yaml`
**When:** Every interaction where orchestrator is active
**Contains:**
- Agent metadata (id, name, title, icon)
- Persona (role, identity, communication style, principles)
- Critical actions (must-dos)
- Role boundaries enforcement (agent boundaries + handoff protocol)
- Team members list
- References to other tiers

**Tokens:** ~500
**Why Always Load:** Essential for orchestrator to function and enforce boundaries

### Tier 2: Operations (On-Demand)
**File:** `operations.yaml`
**When:** Orchestrator is actively coordinating work
**Contains:**
- Coordination phases (prerequisites → deployment)
- Delegation rules (who does what)
- Parallel execution strategies
- Best practices checklist

**Tokens:** ~800
**Why On-Demand:** Only needed when actively coordinating, not for simple queries

### Tier 3: Reference (Load When Needed)
**Directory:** `examples/`
**When:** Explicitly referenced or needed for complex scenarios
**Contains:**
- `coordination.md` - Complete coordination examples
- `handoffs.md` - Detailed handoff patterns (to be created)
- `parallel-execution.md` - Parallel execution scenarios (to be created)

**Tokens:** ~400 per file
**Why Reference:** Detailed examples rarely needed, only for complex cases

## Loading Examples

### Example 1: Simple Question
```
User: "What does the orchestrator do?"

Load:
- orchestrator/core.yaml (500 tokens)

Total: 500 tokens
```

### Example 2: Coordinating Feature Build
```
User: "Build user management feature"

Turn 1 - Initial:
- orchestrator/core.yaml (500 tokens)

Turn 2 - Coordination:
- orchestrator/core.yaml (already loaded)
- orchestrator/operations.yaml (800 tokens)

Total: 1,300 tokens (vs 1,500 tokens for monolithic)
```

### Example 3: Complex Multi-Agent Coordination
```
User: "Build Stripe subscription feature with 8 agents"

Turn 1:
- orchestrator/core.yaml (500 tokens)
- orchestrator/operations.yaml (800 tokens)

Turn 5 - Need detailed example:
- orchestrator/examples/coordination.md (400 tokens)

Total: 1,700 tokens (vs 2,000 tokens for monolithic)
```

## Context Efficiency

### Before Optimization (Monolithic)
- Single file: `orchestrator.agent.yaml`
- Size: 1,232 lines
- Tokens: ~1,500
- **Always loaded all content**

### After Optimization (Modular)
- Typical interaction: core.yaml only
- Size: 200 lines
- Tokens: ~500
- **Load only what's needed**
- **Savings: 67% reduction**

### Best Case Scenario
- Complex coordination: core.yaml + operations.yaml
- Size: 500 lines
- Tokens: ~1,300
- **Savings: 13% reduction**

### With Examples
- Very complex: all tiers
- Size: 700 lines
- Tokens: ~1,700
- **Savings: Still better than loading everything always**

## References

### From core.yaml
```yaml
references:
  operations: "@load orchestrator/operations.yaml"
  coordination_examples: "@reference orchestrator/examples/coordination.md"
```

### From operations.yaml
```yaml
references:
  detailed_examples: "@reference orchestrator/examples/"
```

## Integration with Context Manifest

This structure follows the rules defined in `context-manifest.yaml`:

```yaml
orchestrator:
  always_load: ["core.yaml"]
  load_when_active: ["operations.yaml"]
  pre_fetch_with: ["resource-verifier"]
  estimated_tokens_core: 500
  estimated_tokens_operations: 800
```

## Migration Notes

This orchestrator serves as the **reference implementation** for the tiered loading architecture.

Other agents should follow this pattern:
1. Split into core.yaml + operations.yaml + examples/
2. Core contains: metadata, persona, role_boundaries
3. Operations contains: detailed workflows, patterns
4. Examples contains: code samples, detailed scenarios

See `docs/CONTEXT-OPTIMIZATION.md` for full migration guide.

## Benefits

✅ **67% token reduction** for simple interactions
✅ **Load only what's needed** based on conversation depth
✅ **Faster response times** with smaller context
✅ **Better focus** with relevant context only
✅ **Scalable** as agent knowledge grows
✅ **Cacheable** core definitions across conversations

## Related Documentation

- `context-manifest.yaml` - Loading rules for all agents
- `docs/CONTEXT-OPTIMIZATION.md` - Full optimization strategy
- `docs/core/PARALLEL-CORE.md` - Core parallel execution concepts
- `docs/core/ROLE-BOUNDARIES-CORE.md` - Core boundary enforcement
