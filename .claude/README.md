# Claude Code Configuration

This directory contains the Claude Code configuration for the Fullstack SaaS project.

## Directory Structure

```
.claude/
├── settings.json          # Project settings (version controlled)
├── settings.local.json    # Personal overrides (git-ignored)
├── agents/                # Custom AI agents
│   ├── orchestrator.md    # Main coordinator agent
│   ├── pm.md             # Product Manager
│   ├── architect.md      # Technical Architect
│   ├── frontend.md       # Frontend Developer
│   ├── backend.md        # Backend Developer
│   ├── database.md       # Database Architect
│   ├── security.md       # Security Expert
│   ├── payment.md        # Payment Specialist
│   ├── devops.md         # DevOps Engineer
│   ├── qa.md             # QA Engineer
│   ├── designer.md       # UI/UX Designer
│   └── resource-verifier.md  # Resource Verifier
├── commands/             # Custom slash commands
├── hooks/                # Pre/post command hooks
├── workflows/            # Custom workflows
├── teams/                # Team configurations
├── skills/               # Reusable skills
└── docs/                 # Documentation
```

## Official vs Custom Directories

### Official (Claude Code Standard)
- **agents/**: Custom AI agents with YAML frontmatter
- **commands/**: Slash commands as markdown files
- **hooks/**: Shell scripts for command hooks
- **settings.json**: Shared project settings
- **settings.local.json**: Personal settings (not version controlled)

### Custom Extensions
- **workflows/**: Custom workflow definitions
- **teams/**: Team composition and coordination
- **skills/**: Reusable agent skills
- **docs/**: Project-specific documentation

## Agent Modules

The `fullstack-team` module includes:

- **Orchestrator (Kai)**: Coordinates all agents and manages workflows
- **Product Manager (Alex)**: Requirements and user stories
- **Technical Architect (Jordan)**: System architecture
- **Frontend Developer (Sam)**: Next.js + Shadcn UI
- **Backend Developer (Morgan)**: Server Actions + API Routes
- **Database Architect (Taylor)**: Multi-tenant database design
- **Security Expert (Riley)**: Authentication & security
- **Payment Specialist (Avery)**: Stripe integration
- **DevOps Engineer (Casey)**: Infrastructure & deployment
- **QA Engineer (Quinn)**: Testing & quality assurance
- **UI/UX Designer (Drew)**: Design system & user experience
- **Resource Verifier (Reese)**: Prerequisites verification

## Usage

All agents are available in Claude Code and can be invoked through the orchestrator or directly.

## Tech Stack

- **Framework**: Next.js 15+ (App Router)
- **UI**: Shadcn UI + Tailwind CSS
- **Database**: PostgreSQL with Prisma
- **Cache**: Redis
- **Auth**: NextAuth.js
- **Payments**: Stripe
- **Deployment**: Vercel / Docker

## MCP Servers

Required:
- `@modelcontextprotocol/server-postgres`
- `@modelcontextprotocol/server-github`
- `stripe-mcp-server`

## Migration Note

This directory structure follows the official Claude Code specification. It was migrated from `.claude-code/` to align with Claude Code standards while preserving custom extensions (workflows, teams, skills).
