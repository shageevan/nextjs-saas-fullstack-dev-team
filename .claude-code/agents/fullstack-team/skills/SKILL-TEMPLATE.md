# [Skill Name]

**Category:** [frameworks|libraries|patterns|tools|best-practices]
**Estimated Tokens:** [200-500]
**Last Updated:** [YYYY-MM-DD]
**Official Docs:** [URL or null]
**Status:** [stable|beta|deprecated]

## Metadata

```yaml
skill:
  id: "[category]/[skill-name]"
  version: "1.0.0"
  tags: ["tag1", "tag2", "tag3"]
  related_skills: ["skill1", "skill2"]
  prerequisites: ["skill1", "skill2"]

context:
  tier: 3B  # Individual skill (loaded after index)
  estimated_tokens: 400
  load_when: ["skill_referenced", "use_case_match"]
  cache_duration: "session"
```

## Quick Reference (Load This First - 50 tokens)

**One-sentence description:** Brief description of what this skill is for.

**When to use:** Primary use case in one sentence.

**Key pattern:**
```typescript
// Most common usage in 3-5 lines
```

---

## Overview (100 tokens)

Brief 2-3 paragraph explanation of:
- What this skill covers
- Why it's important
- When to use it

Keep concise - detailed explanations go in later sections.

---

## Key Concepts (100 tokens)

**Concept 1: Name**
- Brief explanation (1-2 sentences)
- Key point to remember

**Concept 2: Name**
- Brief explanation (1-2 sentences)
- Key point to remember

**Concept 3: Name**
- Brief explanation (1-2 sentences)
- Key point to remember

---

## Official Documentation

**Main docs:** [URL]
**API reference:** [URL]
**Guides:** [URL]
**Examples:** [URL]

---

## Common Patterns (150 tokens)

### Pattern 1: [Most Common Use Case]

**When to use:** Brief description

**Quick example:**
```typescript
// Minimal working example (10-15 lines max)
```

**Key points:**
- Important point 1
- Important point 2

### Pattern 2: [Second Most Common]

**When to use:** Brief description

**Quick example:**
```typescript
// Minimal working example
```

---

## Best Practices (50 tokens)

✅ **DO:**
- Best practice 1
- Best practice 2
- Best practice 3

❌ **DON'T:**
- Anti-pattern 1
- Anti-pattern 2
- Anti-pattern 3

---

## Common Pitfalls (50 tokens)

### Pitfall 1: [Name]
**Problem:** What goes wrong
**Fix:** How to avoid it

### Pitfall 2: [Name]
**Problem:** What goes wrong
**Fix:** How to avoid it

---

## Troubleshooting (50 tokens)

### Issue 1: [Common Error]
**Error message:** `error text`
**Cause:** Why it happens
**Fix:**
```bash
# Solution
```

### Issue 2: [Common Problem]
**Symptoms:** What you see
**Fix:** How to solve

---

## Integration (30 tokens)

**Works well with:**
- `@reference [skill1]` - Why
- `@reference [skill2]` - Why

**Depends on:**
- `@reference [skill3]` - Why

**Conflicts with:**
- [approach/pattern] - Why to avoid

---

## Updates Log (20 tokens)

- **2025-01-22:** Initial creation from [source]
- **[YYYY-MM-DD]:** [What changed]

---

## Sources

1. [Official docs title](URL)
2. [GitHub repository](URL)
3. [High-quality article](URL)

---

## Context Optimization Notes

**Target tokens:** 400
**Actual tokens:** [measure after creation]
**Structure:**
- Quick Reference: 50 tokens (load first)
- Core content: 300 tokens
- Extended content: 50 tokens

**Loading strategy:**
- Load Quick Reference first
- Load full content only if needed
- Cache for session duration
- Update if >30 days old
