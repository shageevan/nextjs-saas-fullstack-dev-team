# GitHub & Git

**Category:** Tools
**Estimated Tokens:** 400
**MCP Server:** `@modelcontextprotocol/server-github`
**Official Docs:** https://docs.github.com

## Quick Reference (50 tokens)

Git for version control, GitHub for collaboration, CI/CD, and project management. Use conventional commits and protect main branch with required reviews.

## Overview (80 tokens)

GitHub provides:
- **Version control** with Git
- **CI/CD** via GitHub Actions
- **Code review** and pull requests
- **Issue tracking** and project boards
- **Branch protection** and security scanning

**When to use:** Every project, from day one.

**MCP Integration:** Install `@modelcontextprotocol/server-github` for repository operations via Claude Code.

## MCP Server Setup (80 tokens)

```json
// Add to Claude Code MCP settings
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

**Token Permissions:** repo, workflow, read:org

## Key Concepts (100 tokens)

### 1. Conventional Commits

```bash
# Format: type(scope): description
feat(auth): add OAuth login
fix(api): resolve rate limiting bug
docs(readme): update installation steps
refactor(db): optimize tenant queries
test(users): add E2E tests for signup

# Types: feat, fix, docs, style, refactor, test, chore
```

### 2. Branch Strategy

```bash
main          # Production-ready code
├── develop   # Integration branch
├── feature/user-auth
├── fix/payment-webhook
└── release/v1.2.0

# Create feature branch
git checkout -b feature/new-feature

# Keep updated
git fetch origin
git rebase origin/develop
```

### 3. Branch Protection

- Require pull request reviews (min 1)
- Require status checks to pass
- Require branches to be up to date
- No direct pushes to main
- Require signed commits

## Common Patterns (150 tokens)

### Pattern 1: Feature Development Workflow

```bash
# 1. Create feature branch
git checkout -b feature/subscription-billing

# 2. Make changes, commit frequently
git add .
git commit -m "feat(billing): add Stripe subscription model"

# 3. Push and create PR
git push -u origin feature/subscription-billing
gh pr create --title "Add subscription billing" --body "Implements Stripe subscriptions"

# 4. Address review feedback
git add .
git commit -m "fix(billing): handle webhook signature validation"
git push

# 5. Merge via GitHub UI (squash merge preferred)
```

### Pattern 2: GitHub Actions CI/CD

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - run: npm ci

      - run: npm run lint

      - run: npm run test
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test

      - run: npm run build
```

### Pattern 3: Using GitHub CLI

```bash
# Install gh CLI
brew install gh
gh auth login

# Create PR
gh pr create --title "Feature" --body "Description"

# Check PR status
gh pr status

# Merge PR
gh pr merge --squash --delete-branch

# View issues
gh issue list --assignee @me

# Create issue
gh issue create --title "Bug" --body "Description"
```

## Best Practices (50 tokens)

**DO:**
- ✅ Write clear commit messages (conventional commits)
- ✅ Keep commits atomic and focused
- ✅ Review diffs before committing
- ✅ Use pull requests for all changes
- ✅ Protect main branch
- ✅ Run tests in CI before merge
- ✅ Use GitHub Actions for automation

**DON'T:**
- ❌ Commit directly to main
- ❌ Commit secrets or .env files
- ❌ Create massive PRs (> 400 lines)
- ❌ Force push to shared branches
- ❌ Merge without review

## Troubleshooting (50 tokens)

**Merge Conflicts:**
```bash
# Update your branch
git fetch origin
git rebase origin/main

# Resolve conflicts in files, then
git add <resolved-files>
git rebase --continue
```

**Undo Last Commit:**
```bash
# Keep changes
git reset --soft HEAD~1

# Discard changes
git reset --hard HEAD~1
```

**PR Not Triggering CI:**
```yaml
# Check workflow syntax
gh workflow view ci

# Re-run workflow
gh workflow run ci
```

**Large File Error:**
```bash
# Remove from history
git rm --cached large-file.zip
echo "large-file.zip" >> .gitignore
git commit --amend
```

## Related Skills
- `tools/docker.md` - Container workflows
- `patterns/ci-cd.md` - Deployment pipelines

## Tags
git, github, version-control, ci-cd, github-actions, collaboration
