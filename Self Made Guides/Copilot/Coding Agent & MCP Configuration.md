# 🤖 GitHub Copilot Coding Agent & MCP Configuration Runbook

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

## ✅ Accuracy & Click-Path Notes

- Reviewed against current public GitHub and Microsoft documentation in April 2026 where public documentation is available. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
- When a path starts with `Enterprise`, begin at GitHub, click your profile photo, click `Your enterprises` or `Enterprise`, select the enterprise, then continue with the listed top tab or left-sidebar item.
- When a path starts with `Organization` or `Org`, begin at GitHub, click your profile photo, click `Your organizations`, select the organization, click `Settings`, then continue with the listed sidebar item.
- When a path starts with `Repository`, `Repo`, or a repository name, open the repository, click the `Settings` tab, then continue with the listed sidebar item.
- When a path starts with a vendor portal such as `Microsoft Entra admin center`, `Azure portal`, `Okta Admin Console`, `PingFederate`, `PingOne`, `OneLogin`, `AD FS Management`, `Visual Studio Admin Portal`, or `Azure DevOps`, sign in to that admin portal first, select the tenant, application, or project named in the step, then follow each listed blade, tab, button, and confirmation in order.
- If the expected button is missing, verify you are signed in with the role named in Prerequisites, the feature or license is enabled, and the object is owned by the selected enterprise, organization, or repository. Use page search only to locate the same page, not to skip required confirmation, test, save, or consent clicks.

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
- Monitor via Enterprise → Billing and licensing → Usage

## 🧯 Known Errors & Resolutions

> This section lists the known product errors and admin-facing symptoms that commonly occur with this workflow. Exact message text can vary by product rollout, tenant policy, and provider, so use the log or settings page named in the resolution to confirm the root cause.

| Error or symptom | Likely cause | Resolution |
|------------------|--------------|------------|
| **Page, tab, or button is missing** | Wrong account context, missing admin role, unavailable plan/add-on, or feature rollout not enabled for the selected enterprise/org/repo. | Switch to the correct account and scope, confirm the prerequisite role, verify licensing or add-on activation, then refresh the page. If the control is still absent, use the direct settings URL from the relevant GitHub Docs page and confirm the feature is available for your plan. |
| **Changes appear saved but behavior does not change** | Policy inheritance, cached UI state, propagation delay, or an overlapping enterprise/org/repo policy. | Reopen the settings page, verify the effective policy at the lowest affected scope, wait for propagation where documented, and check for a stricter policy at an enterprise or organization level. |
| **403, forbidden, or resource not accessible** | The signed-in user or token can see the page but lacks the specific permission for the action. | Use an enterprise owner, organization owner, repository admin, or token with the exact scopes/permissions listed in the runbook. For SAML-protected orgs, authorize the token or SSH key for SSO before retrying. |
| **Copilot feature, model, or policy is not visible** | Plan, license assignment, enterprise policy, org delegation, or feature rollout does not permit it. | Check enterprise AI controls, organization Copilot settings, assigned seat status, and the plan requirements for the feature. |
| **Premium requests are rejected after the included allowance** | Paid usage is disabled, no billing entity is selected, or a stop-usage budget is exhausted. | Enable Premium request paid usage where appropriate, set or delete conflicting budgets, and have users with multiple licenses choose a billing entity. |
| **Content exclusions do not apply immediately** | Client policy cache, unsupported surface/mode, symlink/remote filesystem limitation, or indirect IDE context. | Reload the IDE policy, verify the exclusion syntax at enterprise/org/repo scope, and document surfaces where exclusions are limited. |
| **Usage metrics look empty or inconsistent** | Telemetry is disabled, data freshness delay applies, users are unlicensed, or different APIs report different scopes. | Enable the metrics policy, confirm seats and telemetry, wait for data freshness, and avoid comparing dashboards/API endpoints as if they share identical data models. |
| **Coding agent or MCP action is denied** | Agent policy, MCP policy, repository permissions, secrets, or server allowlist does not permit the operation. | Review Enterprise AI controls > Agents/MCP, repo-level permissions, MCP server configuration, and audit logs for the denied action. |

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
