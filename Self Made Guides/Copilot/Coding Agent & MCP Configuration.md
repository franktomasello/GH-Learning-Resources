# 🤖 Copilot Coding Agent & MCP Configuration Runbook

> Guide to enabling, scoping, and extending GitHub Copilot coding agent with Model Context Protocol (MCP) in enterprise

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- Enable coding agent (enterprise): `Enterprise → AI controls → Copilot → Policies → Copilot coding agent → Enable`
- Enable coding agent (org): `Org Settings → Copilot → Policies → Copilot coding agent → Enable`
- Scope to repos: `Org Settings → Copilot → Policies → Copilot coding agent → Select repositories`
- Configure MCP: create `.github/copilot/mcp.json` in the repository
- Custom instructions: create `.github/copilot-instructions.md` in repo root

---

## ✅ Prerequisites

| Requirement | Status |
|-------------|--------|
| GitHub Enterprise Cloud with enterprise owner access | ☐ |
| Copilot Business or Copilot Enterprise subscription active | ☐ |
| Coding agent enabled at enterprise level (org cannot override if disabled) | ☐ |
| Premium request budget configured (coding agent sessions consume premium requests) | ☐ |
| Security review completed for any MCP servers before enabling | ☐ |

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

## ❓ Common Questions & Troubleshooting

### Q: The coding agent is not creating PRs — what should we check?
**A:** Verify three things: (1) the coding agent is enabled at both the enterprise AND organization level (enterprise policies act as a ceiling), (2) the required premium model is available and not disabled in model settings, and (3) the repository has proper permissions — the agent needs write access to create branches and PRs.

---

### Q: MCP server is not connecting — how do I debug this?
**A:** Check the `.github/copilot/mcp.json` file for valid JSON syntax. Verify the MCP server command (e.g., `npx`) is accessible in the agent's runtime environment. Check network and firewall rules if the MCP server needs to reach external endpoints. Review the agent session logs for specific connection error messages.

---

### Q: Can the coding agent access private dependencies?
**A:** Only if the repository has the necessary credentials configured. Set up repository secrets or tokens for private registries (npm, Maven, etc.) and reference them in your build configuration. The agent runs in the context of the repository and can use configured secrets.

---

### Q: The agent is producing low-quality output — how can we improve it?
**A:** Add a `.github/copilot-instructions.md` file with project conventions, build commands, test commands, and architectural guidance. The more context the agent has about your project's patterns and standards, the better its output. Also consider adding path-specific instruction files in `.github/instructions/`.

---

### Q: How do we limit which repos can use the coding agent?
**A:** Navigate to Org Settings > Copilot > Policies > Copilot coding agent and select "Select repositories" to choose specific repos. Start with a small set of approved repos and expand after validating output quality and review processes.

---

### Q: Coding agent sessions are consuming too many premium requests — how do we control this?
**A:** Each agent session and steering comment consumes premium requests. Limit available premium models via enterprise/org model settings, set premium request budgets, and educate users to write clear, detailed issue descriptions to reduce back-and-forth steering comments.

---

### Q: Is it safe to enable MCP servers from third-party sources?
**A:** Treat MCP server approval as a governance decision. Run through the security review checklist before enabling any MCP server: assess what tools/APIs it accesses, what credentials it requires, what network access it needs, and whether it complies with your data classification policy. Have your security team review before approving.

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Admin Controls (Models, Content Exclusion, Instructions) | `Copilot/Admin Controls (Models, Content Exclusion, Instructions).md` |
| Context Management (Spaces, Indexing, Instructions, Exclusions) | `Copilot/Context Management (Spaces, Indexing, Instructions, Exclusions).md` |
| Responsible AI Guardrails | `Copilot/Responsible AI Guardrails.md` |
| Branch Protection Rules & Rulesets | `Governance/Branch Protection Rules & Rulesets.md` |

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
