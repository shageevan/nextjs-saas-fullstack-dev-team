# Skills Knowledge Base

## Overview

The Skills Knowledge Base is a self-learning system where agents accumulate and maintain technical knowledge. Instead of searching the internet repeatedly for the same information, agents build a curated knowledge repository that grows over time.

## Purpose

- **Reduce redundant internet searches** - Knowledge is captured once, reused many times
- **Faster responses** - Local knowledge base is faster than web searches
- **Curated content** - Agents extract and structure the most relevant information
- **Context efficiency** - Skills are Tier 3 reference content, loaded only when needed
- **Team learning** - All agents benefit from knowledge gathered by any agent

## Structure

```
skills/
├── frameworks/          # Framework-specific knowledge (Next.js, React, etc.)
├── libraries/           # Library documentation (Shadcn, Prisma, Stripe, etc.)
├── patterns/            # Design patterns (Server Actions, multi-tenant, etc.)
├── tools/               # Development tools (Docker, Kubernetes, etc.)
├── best-practices/      # Best practices and guidelines
└── README.md            # This file
```

## Skill File Format

Each skill file follows this structure:

```markdown
# [Skill Name]

**Category:** [frameworks|libraries|patterns|tools|best-practices]
**Last Updated:** [YYYY-MM-DD]
**Official Docs:** [URL]
**Sources:** [List of URLs used to compile this skill]

## Overview

Brief description of what this skill covers.

## Key Concepts

- Concept 1: Explanation
- Concept 2: Explanation
- Concept 3: Explanation

## Official Documentation

Links to official documentation:
- [Main docs](url)
- [API reference](url)
- [Guides](url)

## Common Patterns

### Pattern 1: [Name]

**When to use:** Description

**Implementation:**
\`\`\`typescript
// Code example
\`\`\`

**Notes:**
- Important note 1
- Important note 2

### Pattern 2: [Name]

[Same structure as Pattern 1]

## Code Examples

### Example 1: [Description]

\`\`\`typescript
// Complete working example
\`\`\`

**Explanation:**
Step-by-step explanation of the code.

## Best Practices

- ✅ DO: Best practice 1
- ✅ DO: Best practice 2
- ❌ DON'T: Anti-pattern 1
- ❌ DON'T: Anti-pattern 2

## Common Pitfalls

### Pitfall 1: [Name]

**Problem:** Description of the issue

**Solution:** How to avoid or fix it

## Troubleshooting

### Issue 1: [Error or Problem]

**Symptoms:** What you see

**Cause:** Why it happens

**Fix:**
\`\`\`bash
# Commands or code to fix
\`\`\`

## Integration with Other Skills

- Related to: [Link to other skill files]
- Works well with: [Link to other skill files]
- Depends on: [Link to other skill files]

## Updates Log

- **[YYYY-MM-DD]:** Initial creation from [source]
- **[YYYY-MM-DD]:** Updated with [what changed]
```

## Agent Workflow

### When Agent Needs Knowledge

1. **Search Skills Folder First**
   ```
   Agent: "I need to implement [X]. Let me check skills folder."
   → Search: skills/**/*[X]*.md
   ```

2. **If Skill Found**
   ```
   → Load skill file
   → Use knowledge to implement
   → If skill is outdated (>30 days), consider updating
   ```

3. **If Skill NOT Found**
   ```
   → Search internet for official documentation
   → Search for other reliable sources (Stack Overflow, blogs, etc.)
   → Extract and structure knowledge
   → Create new skill file in appropriate category
   → Use knowledge to implement
   ```

4. **If Skill Found but Incomplete**
   ```
   → Search internet for missing information
   → Update skill file with new knowledge
   → Use updated knowledge to implement
   ```

### Creating a New Skill

```yaml
steps:
  1. Search internet:
     - Official documentation (PRIORITY)
     - GitHub repositories
     - High-quality blog posts
     - Stack Overflow discussions

  2. Extract knowledge:
     - Key concepts
     - Common patterns
     - Code examples
     - Best practices
     - Common pitfalls

  3. Structure content:
     - Use skill file format above
     - Include working code examples
     - Add troubleshooting section
     - Link to official docs

  4. Save skill:
     - Choose appropriate category
     - Use descriptive filename (kebab-case)
     - Include all sources used
     - Add creation date

  5. Reference skill:
     - Use skill in current implementation
     - Add to agent's knowledge base
```

### Updating an Existing Skill

```yaml
when_to_update:
  - Skill is >30 days old
  - Agent finds outdated information
  - New version of framework/library released
  - Agent discovers missing information
  - Error or issue not covered in troubleshooting

how_to_update:
  1. Read existing skill file
  2. Search for updated information
  3. Compare with existing content
  4. Add new patterns/examples/fixes
  5. Update "Last Updated" date
  6. Add entry to "Updates Log"
  7. Update version numbers if applicable
```

## Skill Categories

### frameworks/

Framework-specific knowledge:
- `nextjs.md` - Next.js App Router, Server Components, Server Actions
- `react.md` - React hooks, component patterns
- `typescript.md` - TypeScript patterns, types, generics

### libraries/

Library-specific documentation:
- `shadcn-ui.md` - Shadcn UI components, customization
- `prisma.md` - Prisma ORM, schema, queries
- `drizzle.md` - Drizzle ORM, migrations
- `stripe.md` - Stripe API, webhooks, subscriptions
- `nextauth.md` - Authentication with NextAuth
- `zod.md` - Zod validation schemas

### patterns/

Design patterns and architectures:
- `server-actions.md` - Next.js Server Actions pattern
- `multi-tenant.md` - Multi-tenant architecture patterns
- `rbac.md` - Role-based access control
- `api-routes.md` - Next.js API routes
- `middleware.md` - Next.js middleware patterns
- `caching.md` - Caching strategies (Redis, Next.js)
- `error-handling.md` - Error handling patterns

### tools/

Development and deployment tools:
- `docker.md` - Docker containerization
- `kubernetes.md` - Kubernetes orchestration
- `github-actions.md` - CI/CD pipelines
- `vercel.md` - Vercel deployment
- `sentry.md` - Error tracking and monitoring
- `playwright.md` - E2E testing
- `vitest.md` - Unit testing

### best-practices/

Guidelines and best practices:
- `security.md` - Security best practices (OWASP, PCI)
- `performance.md` - Performance optimization
- `accessibility.md` - WCAG 2.1 AA compliance
- `testing.md` - Testing strategies
- `code-quality.md` - Code quality standards
- `database.md` - Database design and optimization

## Context Optimization Integration

Skills follow the Tier 3 (Reference) pattern:

```yaml
# In context-manifest.yaml
skills:
  tier: 3
  estimated_tokens: 200-500 per skill
  load_when: ["skill_referenced", "knowledge_needed"]
  cache_duration: "session"

  loading_strategy:
    - Search local skills first
    - Load only requested skill
    - Create/update if not found
    - Cache for session duration
```

**Token Efficiency:**
- **Before:** Agent searches internet every time → No token savings
- **After:** Agent loads skill once → Reused across conversation
- **Savings:** ~70% reduction in knowledge gathering time

## Naming Conventions

### File Names
- Use kebab-case: `server-actions.md`, `multi-tenant-architecture.md`
- Be specific: `nextjs-app-router.md` not `nextjs.md`
- Include version if applicable: `nextjs-15-server-actions.md`

### Headers
- Use clear, descriptive headers
- Start with H1 for skill name
- Use H2 for main sections
- Use H3 for subsections

### Code Examples
- Always include language identifier: \`\`\`typescript
- Use complete, working examples
- Add inline comments for clarity
- Show imports and setup

## Quality Standards

### Official Documentation Priority
Always prioritize official documentation:
1. Official docs (e.g., nextjs.org, stripe.com/docs)
2. Official GitHub repositories
3. Official blog posts and guides
4. High-quality community content (secondary)

### Code Example Standards
- ✅ Complete and working
- ✅ Follow best practices
- ✅ Include error handling
- ✅ Add explanatory comments
- ✅ Show imports and dependencies

### Freshness
- Update skills when >30 days old
- Check for version updates
- Remove deprecated patterns
- Add new features and patterns

## Example: Using Skills

### Scenario 1: Backend Developer Needs Server Actions Pattern

```
Morgan (Backend): "I need to implement a Server Action for user creation."

Step 1: Search skills
→ Found: skills/patterns/server-actions.md

Step 2: Load skill
→ Read Server Actions pattern
→ Load code examples
→ Check best practices

Step 3: Implement
→ Use pattern from skill
→ Adapt to specific use case
→ Follow best practices from skill
```

### Scenario 2: Frontend Developer Needs Shadcn UI Knowledge

```
Sam (Frontend): "I need to implement a data table with Shadcn UI."

Step 1: Search skills
→ Not found: skills/libraries/shadcn-ui.md

Step 2: Search internet
→ Official docs: ui.shadcn.com
→ Data table component docs
→ Examples and patterns

Step 3: Create skill
→ Create: skills/libraries/shadcn-ui.md
→ Include: Components, patterns, examples
→ Add: Data table specific information

Step 4: Implement
→ Use skill for implementation
→ Skill now available for future use
```

### Scenario 3: Security Expert Needs OWASP Guidelines

```
Riley (Security): "I need to review for OWASP Top 10 vulnerabilities."

Step 1: Search skills
→ Found: skills/best-practices/security.md
→ Last updated: 45 days ago

Step 2: Check for updates
→ Search for OWASP Top 10 updates
→ Found: New 2023 version released

Step 3: Update skill
→ Update: skills/best-practices/security.md
→ Add: 2023 changes
→ Update: Examples and mitigations
→ Add to Updates Log

Step 4: Use updated skill
→ Perform security review
→ Check against latest OWASP Top 10
```

## Benefits

### For Individual Agents
- Faster knowledge access
- No repeated internet searches
- Curated, reliable information
- Working code examples ready to use

### For the Team
- Shared knowledge base
- Consistent patterns across agents
- Accumulated expertise over time
- Reduced external dependencies

### For Context Efficiency
- Skills loaded on-demand (Tier 3)
- Typical skill: 200-500 tokens
- Cached for session duration
- Much faster than web searches

### For Users
- Faster implementations
- More consistent results
- Better quality code
- Growing agent intelligence

## Maintenance

### Regular Maintenance (Weekly)
- Check for outdated skills (>30 days)
- Update version-specific skills
- Add new patterns discovered during work
- Remove deprecated information

### Major Updates (When Released)
- Framework version updates (Next.js, React)
- Library major versions (Stripe, Prisma)
- Best practice changes (OWASP, WCAG)
- Tool updates (Docker, Kubernetes)

## Getting Started

### For Agents

1. **Before implementing anything:**
   - Search skills folder: `skills/[category]/[topic].md`
   - If found: Use the knowledge
   - If not found: Create it

2. **When creating a skill:**
   - Search official docs first
   - Structure using skill file format
   - Include working examples
   - Add troubleshooting section

3. **When using a skill:**
   - Check "Last Updated" date
   - Verify information is current
   - Update if >30 days old
   - Reference skill in your work

### For Users

Skills are automatic - agents will:
- Build knowledge base as they work
- Reuse knowledge across conversations
- Keep information up-to-date
- Share knowledge across the team

You can also:
- Browse skills folder to see accumulated knowledge
- Request specific skills be created
- Ask agents to update outdated skills
- Contribute your own skill files

## Related Documentation

- `context-manifest.yaml` - Skills loading rules
- `docs/CONTEXT-OPTIMIZATION.md` - Context optimization strategy
- `OPTIMIZATION-SUMMARY.md` - Context efficiency metrics
- `docs/AGENTS.md` - Agent capabilities and roles

---

**The Skills Knowledge Base grows smarter with every task.**

Start small, build incrementally, and watch your team's intelligence compound over time.
