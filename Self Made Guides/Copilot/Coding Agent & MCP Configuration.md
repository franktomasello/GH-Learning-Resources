# 🤖 Copilot Coding Agent & MCP Configuration Runbook

> Guide to enabling, scoping, and extending GitHub Copilot coding agent with Model Context Protocol (MCP) in enterprise

---

## 📋 Overview

| Capability | Description |
|-----------|-------------|
| **Agent mode** | Interactive multi-step coding in supported IDEs |
| **Copilot coding agent** | Repo-scoped autonomous work that opens PRs |
| **MCP (Model Context Protocol)** | Extend agent with external tools and data sources |
| **Custom agents** | Package approved workflows as reusable agents |

---

## 1️⃣ Enable Copilot Coding Agent

### Enterprise Level

```
Enterprise → AI controls → Copilot → Policies
  → Copilot coding agent → Enable (or "Enabled for selected organizations")
```

### Organization Level

```
Organization → Settings → Copilot → Policies
  → Copilot coding agent → Enable
```

> ⚠️ **Important:** Coding agent always opens PRs — it never commits directly to branches. This ensures all AI-generated code goes through review.

---

## 2️⃣ Scope Agent to Approved Repositories

The coding agent can be scoped to specific repos:

```
Organization → Settings → Copilot → Policies
  → Copilot coding agent → Select repositories
```

### Best Practice
- Start with a small set of approved repos and teams
- Expand after validating output quality and review processes
- Do NOT enable on repos containing sensitive data without content exclusions

---

## 3️⃣ Configure MCP (Model Context Protocol)

MCP lets the coding agent connect to external tools and data sources.

### Repository-Level MCP Configuration

Create `.github/copilot/mcp.json` in the repository:

```json
{
  "mcpServers": {
    "server-name": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-name"]
    }
  }
}
```

### Available MCP Integrations

| MCP Server | Purpose |
|-----------|---------|
| GitHub MCP Server | Access GitHub APIs (issues, PRs, repos) |
| Filesystem | Read/write local files |
| Database connectors | Query databases for context |
| API connectors | Call external APIs |
| Custom servers | Build your own for internal tools |

---

## 4️⃣ Security Review for MCP

> ⚠️ **Treat MCP server/tool approval as a governance decision, not just a developer convenience setting.**

### Review Checklist Before Enabling MCP

- [ ] What tools/APIs does the MCP server access?
- [ ] What secrets or credentials does it require?
- [ ] What network access does it need?
- [ ] Does it comply with your data classification policy?
- [ ] Has the MCP server been reviewed by your security team?
- [ ] Are there content exclusions in place for sensitive files?

---

## 5️⃣ Custom Instructions for Agent Quality

Improve agent output with repository instructions:

### Repository Instructions

Create `.github/copilot-instructions.md`:

```markdown
## Build & Test
- Run tests with: `npm test`
- Run linting with: `npm run lint`
- All PRs must pass CI before merge

## Coding Standards
- Use TypeScript strict mode
- Follow existing naming conventions
- Add unit tests for new functions
- Do not modify files in /config without approval
```

### Organization Instructions

```
Organization → Settings → Copilot → Custom instructions
  → Add instructions that apply across all repos in the org
```

---

## 6️⃣ Monitor Agent Usage

```
Enterprise → AI controls → Insights
  → Filter by "Coding agent" to see usage metrics
```

- Coding agent sessions consume premium requests
- Each steering comment (follow-up instruction) consumes additional premium requests
- Monitor via Enterprise → Billing & Licensing → Usage

---

## 📝 Resources

| Resource | Link |
|----------|------|
| About Copilot coding agent | [GitHub Docs](https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-coding-agent) |
| Extending coding agent with MCP | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/use-copilot-agents/coding-agent/extend-coding-agent-with-mcp) |
| Best practices for coding agent | [GitHub Docs](https://docs.github.com/en/copilot/how-tos/agents/copilot-coding-agent/best-practices-for-using-copilot-to-work-on-tasks) |
| Copilot features overview | [GitHub Docs](https://docs.github.com/en/copilot/get-started/features) |
| Custom instructions | [GitHub Docs](https://docs.github.com/en/copilot/customizing-copilot/adding-custom-instructions-for-github-copilot) |

---

*Last updated: April 2026*
