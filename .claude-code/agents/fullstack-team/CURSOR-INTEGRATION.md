# Cursor Integration Guide

This guide explains how to use the Fullstack AI Tech Team within Cursor IDE.

## Quick Setup

### 1. Automatic Integration (via .cursorrules)

The `.cursorrules` file at the project root automatically loads the team context into Cursor. No additional setup needed!

### 2. How It Works

When you open this project in Cursor:
1. Cursor reads `.cursorrules` file
2. Loads team principles, agent roles, and patterns
3. Understands API-first architecture requirements
4. Knows to check skills folder before implementing
5. Enforces role boundaries and multi-tenant security

## Using the Team in Cursor

### Method 1: Ask for Specific Agent

```
You: "I need Morgan (Backend Developer) to create an API endpoint for user management with authentication"

Cursor: [Applies Morgan's expertise from backend.agent.yaml]
- Designs RESTful API endpoint
- Implements API key authentication
- Adds rate limiting
- Enforces tenant isolation
- Includes Zod validation
```

### Method 2: Request Team Coordination

```
You: "I need to build a subscription billing feature with Stripe"

Cursor: [Uses orchestrator coordination patterns]
1. Jordan (Architect) - Designs system architecture
2. Taylor (Database) - Creates subscription schema
3. Morgan (Backend) - Implements Stripe webhooks
4. Sam (Frontend) - Builds pricing page
5. Riley (Security) - Reviews payment security
6. Avery (Payment) - Configures Stripe
7. Quinn (QA) - Tests subscription flows
```

### Method 3: Reference Skills

```
You: "Show me the pattern for implementing Server Actions"

Cursor: [Loads skills/patterns/server-actions.md]
- Provides complete pattern with code examples
- Shows validation, authentication, revalidation
- Includes best practices and pitfalls
```

### Method 4: Check Prerequisites

```
You: "What do I need before starting development?"

Cursor: [Uses resource-verifier patterns]
- Checks database connection
- Verifies MCP servers
- Lists missing environment variables
- Confirms NPM packages installed
```

## Cursor Composer Integration

### Using Composer with Agent Context

Cursor Composer can leverage the agent context automatically:

1. **Open Composer** (Cmd/Ctrl + Shift + I)
2. **Mention specific files** to load agent context:
   ```
   @agents/orchestrator/core.yaml
   @agents/backend.agent.yaml
   @skills/patterns/api-first-architecture.md
   ```
3. **Ask for implementation** and Composer will follow agent patterns

### Example Composer Usage

```
Composer Query:
"@agents/backend.agent.yaml
Implement a user management API with:
- GET /api/v1/users (list users)
- POST /api/v1/users (create user)
- PUT /api/v1/users/:id (update user)
- DELETE /api/v1/users/:id (delete user)

Requirements:
- API key authentication
- Rate limiting (1000 req/hour)
- Multi-tenant isolation
- OpenAPI documentation"

Composer Response:
[Follows Morgan's patterns from backend.agent.yaml]
- Creates versioned API routes
- Implements authentication
- Adds rate limiting
- Enforces tenant isolation
- Generates OpenAPI spec
```

## Cursor Chat Integration

### Using Chat with Skills

```
You: "What's the best way to implement multi-tenant isolation?"

Chat: [References skills/patterns/multi-tenant.md]
"Based on the team's multi-tenant patterns, here are three approaches:

1. Row-Level Security (Recommended for most cases)
2. Schema per Tenant (For data isolation needs)
3. Database per Tenant (Maximum isolation)

Let me show you the Row-Level Security pattern..."
```

### Using Chat with Agent Roles

```
You: "Should I create Server Actions in the frontend component?"

Chat: [Uses role boundaries from .cursorrules]
"No. According to the role boundaries:

- Sam (Frontend): UI components, client state
- Morgan (Backend): Server Actions, business logic

Server Actions should be created by Morgan in separate files
(e.g., app/actions/users.ts), then imported by Sam in frontend
components.

Would you like me to show you the proper pattern?"
```

## Context Loading in Cursor

### When Cursor Loads Agent Context

Cursor automatically loads context when:
1. You reference agent files with `@` mentions
2. You ask questions about architecture or patterns
3. You request specific agent expertise
4. You're implementing features that match agent roles

### Context Optimization

Cursor follows the team's 3-tier loading architecture:
- **Tier 1 (Core)**: Always available via .cursorrules
- **Tier 2 (Operations)**: Loaded when you ask for implementations
- **Tier 3 (Examples/Skills)**: Loaded when explicitly referenced

This keeps Cursor's context efficient while maintaining access to all knowledge.

## Practical Examples

### Example 1: Building Authentication

```
You: "I need to add authentication to the app"

Cursor will:
1. Load Security Expert (Riley) patterns
2. Load skills/libraries/nextauth.md
3. Load skills/patterns/rbac.md
4. Suggest architecture from Jordan (Architect)
5. Implement with Morgan (Backend) + Sam (Frontend)
6. Add tests with Quinn (QA)
```

### Example 2: Creating API Endpoints

```
You: "Create a RESTful API for products"

Cursor will:
1. Follow API-first architecture from .cursorrules
2. Apply Morgan (Backend) patterns
3. Implement versioning (/api/v1/products)
4. Add authentication and rate limiting
5. Enforce tenant isolation
6. Generate OpenAPI documentation
7. Suggest Riley (Security) review
```

### Example 3: Database Schema Design

```
You: "Design a database schema for orders"

Cursor will:
1. Consult Taylor (Database Architect) patterns
2. Include multi-tenancy (tenantId column)
3. Add proper indexes
4. Create Prisma/Drizzle schema
5. Generate migrations
6. Show query patterns with tenant isolation
```

## Best Practices for Cursor

### 1. Always Start with Context

Before implementing, ask Cursor to:
```
"Check skills folder for [topic] patterns"
"What does the team's architecture say about [feature]?"
"Show me the agent role boundaries for this task"
```

### 2. Reference Specific Agents

```
"Using Morgan (Backend) patterns, implement..."
"Following Sam (Frontend) guidelines, create..."
"Apply Riley (Security) best practices to..."
```

### 3. Check Skills Before Implementing

```
"Search skills/index.yaml for authentication patterns"
"Load skills/patterns/api-first-architecture.md"
"Show me skills/libraries/prisma.md examples"
```

### 4. Use Composer for Complex Tasks

For features requiring multiple agents:
```
Composer: "Build a subscription feature with the full team"
(Automatically coordinates Architect → Database → Backend → Frontend → Security → Payment → QA)
```

### 5. Validate with Role Boundaries

If unsure who should do what:
```
You: "Should Frontend create database queries?"
Cursor: "No. According to role boundaries:
- Frontend (Sam): UI only
- Backend (Morgan): Database queries
- Database Architect (Taylor): Schema design"
```

## Advanced Usage

### Custom Workflows

Create custom Cursor commands that load specific agent combinations:

```json
// .vscode/settings.json
{
  "cursor.customInstructions": [
    {
      "name": "New API Endpoint",
      "instruction": "Follow Morgan (Backend) patterns to create a new API endpoint with authentication, rate limiting, and tenant isolation. Check skills/patterns/api-first-architecture.md first."
    },
    {
      "name": "New UI Component",
      "instruction": "Follow Sam (Frontend) patterns to create a Shadcn UI component. Check skills/libraries/shadcn-ui.md for component patterns."
    }
  ]
}
```

### Agent Chaining

Request sequential agent work:
```
"First, have Jordan (Architect) design the system.
Then Taylor (Database) create the schema.
Then Morgan (Backend) implement the API.
Finally, Sam (Frontend) build the UI."
```

## Troubleshooting

### Issue: Cursor not following agent patterns
**Solution**: Explicitly reference the agent file:
```
@.claude-code/agents/fullstack-team/agents/backend.agent.yaml
"Using Morgan's patterns, implement..."
```

### Issue: Skills not being used
**Solution**: Explicitly reference the skills folder:
```
@.claude-code/agents/fullstack-team/skills/patterns/
"Check skills folder first for patterns"
```

### Issue: Role boundaries being violated
**Solution**: Reference role boundaries doc:
```
@.claude-code/agents/fullstack-team/docs/ROLE-BOUNDARIES.md
"Ensure this follows role boundaries"
```

## Key Differences: Claude Code vs Cursor

| Feature | Claude Code | Cursor |
|---------|-------------|--------|
| Agent Loading | Automatic via agent files | Via .cursorrules + @ mentions |
| Context Management | 3-tier loading system | Manual file references |
| Team Coordination | Orchestrator built-in | You coordinate via prompts |
| Skills Loading | Automatic index search | Manual file references |
| Role Enforcement | Strict automatic | Via .cursorrules guidance |

## Making the Most of Both

### Use Claude Code For:
- Initial project setup
- Complex team coordination
- Automatic role enforcement
- Skills creation and management

### Use Cursor For:
- Day-to-day development
- Quick implementations
- Code editing and refactoring
- Real-time suggestions

### Best Workflow:
1. **Plan with Claude Code** (load orchestrator, coordinate team)
2. **Implement with Cursor** (follow patterns from .cursorrules)
3. **Refine with Claude Code** (complex multi-agent tasks)

## Resources

- **Agent Files**: `.claude-code/agents/fullstack-team/agents/`
- **Skills**: `.claude-code/agents/fullstack-team/skills/`
- **Documentation**: `.claude-code/agents/fullstack-team/docs/`
- **Cursor Rules**: `.cursorrules` (root of project)

## Summary

The Fullstack AI Tech Team integrates with Cursor through:
1. ✅ `.cursorrules` file (automatic context loading)
2. ✅ `@` file mentions (explicit agent loading)
3. ✅ Skills folder (pattern reference library)
4. ✅ Documentation (comprehensive guides)

**You now have the full team available in both Claude Code and Cursor!** 🎉
