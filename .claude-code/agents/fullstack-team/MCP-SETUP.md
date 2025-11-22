# MCP Server Setup Guide

Model Context Protocol (MCP) servers enhance **both Claude Code and Cursor 2.1+** with direct access to tools and services. This guide covers installing and configuring MCPs for the Fullstack AI Tech Team.

## Quick Start

MCP servers are configured in your IDE's settings file. Each server runs as a separate process and communicates via stdio.

### Configuration Locations

**Claude Code:**
- Project-level: `.claude-code/mcp_settings.json`
- Global: Claude Code settings

**Cursor 2.1+:**
- Settings → Features → Model Context Protocol
- Or: `.cursor/mcp_config.json` (project-level)
- Or: `~/.cursor/mcp_config.json` (global)

Both IDEs use the same MCP server format and configuration.

## Required MCP Servers

These servers are essential for default team functionality:

### 1. PostgreSQL Server

**Package:** `@modelcontextprotocol/server-postgres`
**Purpose:** Direct database operations, schema inspection, query execution
**Skills:** `tools/postgresql.md`

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-postgres",
        "postgresql://user:password@localhost:5432/database"
      ]
    }
  }
}
```

**Environment Variable Approach:**
```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-postgres",
        "${DATABASE_URL}"
      ]
    }
  }
}
```

**Features:**
- Execute SQL queries
- Inspect schema and tables
- Create migrations
- Query execution plans
- Database performance analysis

---

### 2. GitHub Server

**Package:** `@modelcontextprotocol/server-github`
**Purpose:** Repository operations, PR management, issue tracking
**Skills:** `tools/github.md`

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-github"
      ],
      "env": {
        "GITHUB_TOKEN": "<your-github-personal-access-token>"
      }
    }
  }
}
```

**Generate Token:** https://github.com/settings/tokens
**Required Permissions:** `repo`, `workflow`, `read:org`

**Features:**
- Create and manage pull requests
- Create and update issues
- View repository contents
- Manage branches
- View workflows and runs

---

### 3. Stripe Server

**Package:** `stripe-mcp-server` (or official Stripe MCP when available)
**Purpose:** Payment operations, customer management, subscription handling
**Skills:** `tools/stripe-cli.md`, `libraries/stripe.md`

```json
{
  "mcpServers": {
    "stripe": {
      "command": "npx",
      "args": [
        "-y",
        "stripe-mcp-server"
      ],
      "env": {
        "STRIPE_API_KEY": "<your-stripe-secret-key>"
      }
    }
  }
}
```

**API Key:** Use test mode key (starts with `sk_test_`)
**Get Key:** https://dashboard.stripe.com/test/apikeys

**Features:**
- Manage customers and subscriptions
- Create checkout sessions
- View payment history
- Manage products and prices
- Trigger test webhooks

---

## Recommended MCP Servers

These servers enhance development workflow:

### 4. Docker Server

**Package:** `@modelcontextprotocol/server-docker`
**Purpose:** Container management, image operations
**Skills:** `tools/docker.md`

```json
{
  "mcpServers": {
    "docker": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-docker"
      ]
    }
  }
}
```

**Prerequisites:** Docker Desktop or Docker Engine running

**Features:**
- List and manage containers
- Build images
- View logs
- Execute commands in containers
- Manage networks and volumes

---

### 5. Redis Server

**Package:** `redis-mcp-server` (community or build custom)
**Purpose:** Cache operations, data inspection
**Skills:** `tools/redis.md`

```json
{
  "mcpServers": {
    "redis": {
      "command": "npx",
      "args": [
        "-y",
        "redis-mcp-server"
      ],
      "env": {
        "REDIS_URL": "redis://localhost:6379"
      }
    }
  }
}
```

**Features:**
- Inspect cache keys
- Get/set values
- Check TTLs
- Monitor performance
- Flush cache

---

### 6. Postmark Server

**Package:** `postmark-mcp`
**Repository:** https://github.com/ActiveCampaign/postmark-mcp
**Purpose:** Transactional email operations, delivery tracking
**Skills:** `tools/postmark.md`

```json
{
  "mcpServers": {
    "postmark": {
      "command": "npx",
      "args": [
        "-y",
        "postmark-mcp"
      ],
      "env": {
        "POSTMARK_API_TOKEN": "<your-postmark-server-token>"
      }
    }
  }
}
```

**API Token:** Get from Postmark dashboard → Servers → API Tokens

**Features:**
- Send transactional emails
- Check email delivery status
- View bounce and spam reports
- Test email templates
- Monitor email analytics
- Track opens and clicks

**Common Operations:**
```
"Send a password reset email to user@example.com"
"Check delivery status for message ID abc123"
"Show bounce statistics for last 7 days"
"Test the welcome-email template"
```

---

## Optional MCP Servers

Enhance team capabilities based on deployment platform:

### AWS Server
```json
{
  "mcpServers": {
    "aws": {
      "command": "npx",
      "args": ["-y", "aws-mcp-server"],
      "env": {
        "AWS_ACCESS_KEY_ID": "${AWS_ACCESS_KEY_ID}",
        "AWS_SECRET_ACCESS_KEY": "${AWS_SECRET_ACCESS_KEY}",
        "AWS_REGION": "us-east-1"
      }
    }
  }
}
```

### Vercel Server
```json
{
  "mcpServers": {
    "vercel": {
      "command": "npx",
      "args": ["-y", "vercel-mcp-server"],
      "env": {
        "VERCEL_TOKEN": "${VERCEL_TOKEN}"
      }
    }
  }
}
```

### Sentry Server
```json
{
  "mcpServers": {
    "sentry": {
      "command": "npx",
      "args": ["-y", "sentry-mcp-server"],
      "env": {
        "SENTRY_AUTH_TOKEN": "${SENTRY_AUTH_TOKEN}",
        "SENTRY_ORG": "your-org",
        "SENTRY_PROJECT": "your-project"
      }
    }
  }
}
```

---

## Complete Configuration Example

**File:** `.claude-code/mcp_settings.json` (project root)

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-postgres",
        "${DATABASE_URL}"
      ]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "stripe": {
      "command": "npx",
      "args": ["-y", "stripe-mcp-server"],
      "env": {
        "STRIPE_API_KEY": "${STRIPE_API_KEY}"
      }
    },
    "docker": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-docker"]
    },
    "redis": {
      "command": "npx",
      "args": ["-y", "redis-mcp-server"],
      "env": {
        "REDIS_URL": "${REDIS_URL}"
      }
    },
    "postmark": {
      "command": "npx",
      "args": ["-y", "postmark-mcp"],
      "env": {
        "POSTMARK_API_TOKEN": "${POSTMARK_API_TOKEN}"
      }
    }
  }
}
```

**Environment Variables File:** `.env.local`

```bash
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/myapp

# GitHub
GITHUB_TOKEN=ghp_xxxxxxxxxxxxxxxxxxxxx

# Stripe (use test keys)
STRIPE_API_KEY=sk_test_xxxxxxxxxxxxxxxxxxxxx

# Redis
REDIS_URL=redis://localhost:6379

# Postmark
POSTMARK_API_TOKEN=your-postmark-server-token
```

**Important:** Add `.env.local` to `.gitignore` to prevent committing secrets!

---

## Cursor 2.1+ Configuration

Cursor 2.1+ has native MCP support. Configuration is identical to Claude Code.

### Method 1: Via Settings UI (Recommended)

1. Open Cursor Settings (Cmd/Ctrl + ,)
2. Search for "Model Context Protocol" or "MCP"
3. Click "Edit in settings.json"
4. Add your MCP servers using the same format as above

### Method 2: Project-Level Config File

**File:** `.cursor/mcp_config.json` (create in project root)

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-postgres",
        "${DATABASE_URL}"
      ]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "stripe": {
      "command": "npx",
      "args": ["-y", "stripe-mcp-server"],
      "env": {
        "STRIPE_API_KEY": "${STRIPE_API_KEY}"
      }
    },
    "docker": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-docker"]
    },
    "redis": {
      "command": "npx",
      "args": ["-y", "redis-mcp-server"],
      "env": {
        "REDIS_URL": "${REDIS_URL}"
      }
    }
  }
}
```

### Method 3: Global Config

**File:** `~/.cursor/mcp_config.json` (user home directory)

Use same format as above. Global config applies to all projects.

### Environment Variables in Cursor

Cursor reads environment variables from:
1. `.env.local` (project root)
2. `.env` (project root)
3. System environment variables

Same `.env.local` file works for both Claude Code and Cursor.

### Verifying MCP in Cursor

1. Open Cursor
2. Look for MCP indicator in status bar (bottom right)
3. Click to see connected MCP servers
4. Green checkmarks = servers running successfully

Or ask Cursor directly:
```
"What MCP servers are currently connected?"
```

---

## Verification

After configuring MCP servers, verify they're working in your IDE:

### 1. Check MCP Status

**Claude Code:**
- Look for MCP server status indicators
- ✅ Green checkmarks = running servers
- ⚠️ Warnings = misconfigured servers
- ❌ Errors = failed servers

**Cursor 2.1+:**
- Check MCP indicator in status bar (bottom right)
- Click to see list of connected servers
- Green checkmarks = servers running successfully

### 2. Test Each Server

Test in either IDE by asking the AI:

**PostgreSQL:**
```
Ask Claude: "Show me the schema for the users table"
```

**GitHub:**
```
Ask Claude: "List open pull requests in this repository"
```

**Stripe:**
```
Ask Claude: "Show me the last 5 customers in Stripe"
```

**Docker:**
```
Ask Claude: "List all running Docker containers"
```

**Redis:**
```
Ask Claude: "Show me all cache keys starting with 'user:'"
```

---

## Troubleshooting

### Server Won't Start

**Issue:** MCP server fails to start

**Solutions:**
1. Check package exists: `npm info @modelcontextprotocol/server-postgres`
2. Verify Node.js version: `node --version` (requires v18+)
3. Check command syntax in mcp_settings.json
4. View Claude Code logs for error details

### Authentication Errors

**Issue:** "Unauthorized" or "Invalid credentials"

**Solutions:**
1. Verify environment variables are set: `echo $GITHUB_TOKEN`
2. Check token permissions (GitHub needs `repo`, `workflow`)
3. Test API key directly: `curl -H "Authorization: Bearer $GITHUB_TOKEN" https://api.github.com/user`
4. Regenerate token if expired

### Connection Errors

**Issue:** "Connection refused" or "Timeout"

**Solutions:**
1. Verify service is running (PostgreSQL, Redis, Docker)
2. Check connection string (host, port, credentials)
3. Test connection manually:
   - PostgreSQL: `psql $DATABASE_URL`
   - Redis: `redis-cli ping`
   - Docker: `docker ps`

### Performance Issues

**Issue:** MCP operations are slow

**Solutions:**
1. Check network latency (especially for cloud databases)
2. Use connection pooling for databases
3. Limit query results (use LIMIT in SQL queries)
4. Consider caching frequently accessed data

---

## Security Best Practices

### 1. Use Environment Variables

✅ **DO:**
```json
"env": {
  "GITHUB_TOKEN": "${GITHUB_TOKEN}"
}
```

❌ **DON'T:**
```json
"env": {
  "GITHUB_TOKEN": "ghp_abc123hardcoded"
}
```

### 2. Use Test Keys in Development

- Stripe: Use `sk_test_` keys, not `sk_live_`
- GitHub: Use personal access tokens with minimum required permissions
- Databases: Use separate dev/staging databases

### 3. Restrict Permissions

- GitHub token: Only `repo` and `workflow` (not admin)
- Database user: Read/write only, not superuser
- Stripe: Test mode only in development

### 4. Rotate Keys Regularly

- GitHub: Regenerate tokens every 90 days
- Stripe: Rotate API keys quarterly
- Database: Update passwords regularly

### 5. Never Commit Secrets

```bash
# Add to .gitignore
.env
.env.local
.env.*.local
.claude-code/mcp_settings.json  # If it contains secrets
```

---

## Agent Workflow with MCPs

When agents need tool access:

1. **Agent checks skill** → Load `tools/postgresql.md`
2. **Skill mentions MCP** → "MCP Server: @modelcontextprotocol/server-postgres"
3. **Agent verifies MCP available** → Check if postgres MCP is running
4. **Agent uses MCP** → Execute operations via MCP
5. **Agent updates skill if needed** → Document new patterns discovered

**Example: Database Schema Design**

```
User: "Design a database schema for multi-tenant blog posts"

Taylor (Database Architect):
1. Load skills/tools/postgresql.md
2. See MCP available: @modelcontextprotocol/server-postgres
3. Use MCP to inspect current database schema
4. Design new tables with tenant_id
5. Use MCP to create migration
6. Verify schema created successfully
```

---

## MCP Server Development

Building custom MCP servers for team-specific tools:

### Structure
```typescript
// server.ts
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';

const server = new Server({
  name: 'my-custom-server',
  version: '1.0.0',
});

// Register tools
server.setRequestHandler('tools/list', async () => ({
  tools: [
    {
      name: 'my_tool',
      description: 'Does something useful',
      inputSchema: {
        type: 'object',
        properties: {
          param: { type: 'string' }
        }
      }
    }
  ]
}));

// Handle tool calls
server.setRequestHandler('tools/call', async (request) => {
  // Implementation
});

const transport = new StdioServerTransport();
await server.connect(transport);
```

### Publishing
```bash
# Build
npm run build

# Publish to npm
npm publish

# Users install via npx
npx -y my-custom-mcp-server
```

---

## Support & Resources

- **MCP Specification:** https://modelcontextprotocol.io
- **Official MCP Servers:** https://github.com/modelcontextprotocol
- **Claude Code Docs:** Check for latest MCP integration docs
- **Team Skills:** All MCP-enabled tools have `mcp_server` field in skills/index.yaml

---

## Summary

✅ **Required:** PostgreSQL, GitHub, Stripe
✅ **Recommended:** Docker, Redis
✅ **Optional:** AWS, Vercel, Sentry

Configure once, benefit across all projects. MCPs significantly enhance agent capabilities by providing direct tool access.
