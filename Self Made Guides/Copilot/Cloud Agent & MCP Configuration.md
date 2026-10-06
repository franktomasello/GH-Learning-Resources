# 🤖 GitHub Copilot Cloud Agent & MCP Configuration Runbook

> Guide to enabling, scoping, securing, and extending Copilot cloud agent (formerly "Copilot coding agent") with Model Context Protocol (MCP) servers in an enterprise

| 🧭 **At a Glance** | |
|---|---|
| **Goal** | Turn on Copilot cloud agent safely and give it extra tools through MCP |
| **Use this when** | Piloting agents that open pull requests on their own |
| **People you need** | Enterprise owner; organization owners; repository admins |
| **Where you click** | GitHub (AI controls, org and repo settings) |
| **End result** | Cloud agent enabled for chosen repos, with MCP, firewall, and review guardrails |
| **New to a term?** | See the [Glossary](../Glossary.md) for plain-English definitions |

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [📋 Overview](#-overview)
- [1️⃣ Enable Cloud Agent at the Enterprise Level](#1️⃣-enable-cloud-agent-at-the-enterprise-level)
- [2️⃣ Enable Cloud Agent for an Organization](#2️⃣-enable-cloud-agent-for-an-organization)
- [3️⃣ Choose Which Repositories Can Use the Agent](#3️⃣-choose-which-repositories-can-use-the-agent)
- [4️⃣ Allow Third-Party MCP Servers](#4️⃣-allow-third-party-mcp-servers)
- [5️⃣ Configure MCP Servers in a Repository](#5️⃣-configure-mcp-servers-in-a-repository)
- [6️⃣ Control the Agent's Internet Access (Firewall)](#6️⃣-control-the-agents-internet-access-firewall)
- [7️⃣ Review Repository Safety Settings](#7️⃣-review-repository-safety-settings)
- [8️⃣ Improve Agent Quality (Instructions & Custom Agents)](#8️⃣-improve-agent-quality-instructions--custom-agents)
- [9️⃣ Monitor Agent Activity and Cost](#9️⃣-monitor-agent-activity-and-cost)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- Enable for the enterprise: `Enterprise → AI controls → Agents → Copilot Cloud Agent` → choose a policy (for example **Enabled for selected organizations**)
- Enable for an org: `Org Settings → Copilot → Policies` → **Copilot cloud agent** = **Enabled**, **MCP servers on GitHub.com** = **Enabled**
- Limit repositories: `Org Settings → Copilot → Cloud agent` → **Repository access** → **Selected repositories** → **Select**
- Allow third-party MCP (enterprise): `Enterprise → AI controls → MCP` → **MCP servers in Copilot**
- Add MCP servers to a repo: `Repo Settings → Copilot → MCP servers` → paste JSON → **Save MCP configuration**
- Firewall: `Org Settings → Copilot → Internet access`
- Watch activity: `Enterprise → AI controls` → **Agent sessions** → **View all**

---

## ✅ Accuracy & Click-Path Notes

<details>
<summary><em>Show click-path conventions</em></summary>

- Reviewed against current public GitHub documentation in October 2026. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
- When a path starts with `Enterprise`, begin at GitHub, click your profile picture, click `Enterprise` (managed/EMU accounts) — or open the `Enterprises` page at github.com/settings/enterprises (standard accounts) —, select the enterprise, then continue with the listed top tab or left-sidebar item.
- When a path starts with `Organization` or `Org`, begin at GitHub, click your profile picture, click `Organizations`, select the organization, click `Settings`, then continue with the listed sidebar item.
- When a path starts with `Repository`, `Repo`, or a repository name, open the repository, click the `Settings` tab, then continue with the listed sidebar item.
- If the expected button is missing, verify you are signed in with the role named in Prerequisites, the feature or license is enabled, and the object is owned by the selected enterprise, organization, or repository. Use page search only to locate the same page, not to skip required confirmation, test, save, or consent clicks.

</details>

---

## ✅ Prerequisites

| Requirement | Who / Role needed | ✓ |
|-------------|-------------------|---|
| Copilot Business or Copilot Enterprise, with seats assigned to the pilot users | GitHub **enterprise owner** or **organization owner** | ☐ |
| Enterprise policy for cloud agent and MCP (Sections 1 and 4) | GitHub **enterprise owner** | ☐ |
| Organization policies, repository access, runner, and firewall (Sections 2, 3, 6) | GitHub **organization owner** | ☐ |
| Repository MCP configuration, Agents secrets, and safety settings (Sections 5, 7) | **Repository administrator** | ☐ |
| Users who delegate tasks | A Copilot license **and** write permission to the repository | ☐ |
| Budget plan — agent sessions consume **AI credits** and **GitHub Actions minutes** | Billing manager / enterprise owner | ☐ |
| Security review of every MCP server before it's added | Security team | ☐ |

> 📌 Cloud agent and third-party MCP servers are **off by default** for Copilot Business and Copilot Enterprise. Nothing happens until an admin turns them on.

---

## 📋 Overview

| Capability | Description |
|-----------|-------------|
| **Copilot cloud agent** | Works on an issue or request in its own GitHub Actions–powered environment, then opens a **draft pull request** and iterates on review feedback |
| **Agent mode (IDE)** | Interactive multi-step coding inside supported IDEs — a separate feature |
| **MCP servers** | Give the agent extra tools (for example Sentry, Azure, Azure DevOps). GitHub and Playwright MCP servers are on by default |
| **Custom agents** | Markdown "agent profiles" with your own prompt, tools, and MCP servers |
| **Automations** | Run the cloud agent on a schedule or on events (private and internal repositories only) |
| **Firewall** | Limits the agent's internet access to reduce data-exfiltration risk |

**How the policy layers fit together:**

| Layer | Who | What it controls |
|-------|-----|------------------|
| Enterprise | Enterprise owner | Which organizations can use cloud agent; third-party MCP; enterprise-wide block |
| Organization | Organization owner | Policy for members, which repositories allow the agent, runner, firewall, automations |
| Repository | Repository administrator | MCP configuration, Agents secrets, firewall allowlist, validation tools, workflow approval |

> ⚠️ **Important:** An organization can't override an enterprise decision. If the enterprise disables cloud agent — or doesn't select the organization — the org setting has no effect.

---

## 1️⃣ Enable Cloud Agent at the Enterprise Level

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **AI controls** → **Agents** *(sidebar)* → **Copilot Cloud Agent**

**Steps:**

1. At the top of the enterprise page, click **AI controls**.
2. In the left sidebar, click **Agents**.
3. Under **Available agents**, click **Copilot Cloud Agent**.
4. Select a policy. For a pilot, choose **Enabled for selected organizations**, then select the pilot organizations.
5. Tell your organization owners what you chose.

> 📌 The policy applies as soon as you select it — there's no Save button. To pick organizations by custom property instead of by name, use the REST API.

> 🛑 **Need to stop it everywhere?** On the same page, turn on **Block Copilot cloud agent in all repositories owned by *[enterprise]***. This blocks the agent for *everyone* in your repositories, including users who get Copilot from a personal plan or another enterprise.

---

## 2️⃣ Enable Cloud Agent for an Organization

**👤 Role:** GitHub **organization owner** · **📍 Portal:** GitHub

**Navigate:** Organization → **Settings** → **Copilot** *(under "Code, planning, and automation")* → **Policies**

**Steps:**

1. In the sidebar, under **Code, planning, and automation**, click **Copilot**, then **Policies**.
2. Next to **Copilot cloud agent**, open the dropdown and select **Enabled**.
3. If the agent should use MCP servers, next to **MCP servers on GitHub.com**, select **Enabled**.

> 📌 If either dropdown is locked, the enterprise has already set that policy (Section 1).

---

## 3️⃣ Choose Which Repositories Can Use the Agent

**👤 Role:** GitHub **organization owner** · **📍 Portal:** GitHub

**Navigate:** Organization → **Settings** → **Copilot** → **Cloud agent**

By default, the agent is available in **all** repositories in an enabled organization.

**Steps — limit repositories:**

1. In the sidebar, under **Code, planning, and automation**, click **Copilot**, then **Cloud agent**.
2. Use the **Repository access** control and choose **Selected repositories**.
3. In the **Select repositories** dialog, check the pilot repositories, then click **Select**.

**Steps — optional organization settings on the same page:**

| Setting | What to click | Default |
|---------|---------------|---------|
| **Allow automations** | Toggle on or off | On |
| **Runner type** | Pencil icon → **Standard GitHub runner** (`ubuntu-latest`) or **Labeled runner** (enter **Runner group name** and/or **Runner label**) → **Save runner selection** | Standard GitHub runner |
| **Allow repositories to customize the runner type** | Toggle → **Save** | On (repos can override with `copilot-setup-steps.yml`) |
| **Partner agents** (Anthropic Claude, OpenAI Codex) | Toggle each agent | Only shown if the enterprise allows third-party agents |

> 💡 **Tip:** Start with a few repositories that have good tests and CI. Expand once you trust the output and your review process.

---

## 4️⃣ Allow Third-Party MCP Servers

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **AI controls** → **MCP** *(sidebar)*

The cloud agent always has the **GitHub** and **Playwright** MCP servers. Any other MCP server needs this policy.

**Steps:**

1. At the top of the enterprise page, click **AI controls**.
2. In the left sidebar, click **MCP**.
3. Set a policy for **MCP servers in Copilot**.

> 📌 The **MCP Registry URL** and **Restrict MCP access to registry servers** policies do **not** apply to Copilot cloud agent.

---

## 5️⃣ Configure MCP Servers in a Repository

**👤 Role:** **Repository administrator** · **📍 Portal:** GitHub

**Navigate:** Repository → **Settings** → **Copilot** *(under "Code, planning, and automation")* → **MCP servers**

> ⚠️ **Important:** Once a server is configured, Copilot uses its tools **on its own, without asking for approval**. Allowlist only the tools you need — read-only tools wherever possible.

### Step A — Store any keys as Agents secrets

1. Repository → **Settings** → in the **Security** section, click **Secrets and variables** → **Agents**.
2. Click the **Secrets** tab → **New repository secret**.
3. Enter a **Name** that starts with `COPILOT_MCP_` (for example `COPILOT_MCP_SENTRY_ACCESS_TOKEN`) and the **Secret**.
4. Click **Add secret**.

> 📌 Only Agents secrets and variables that start with `COPILOT_MCP_` reach your MCP configuration. Organization owners can create shared ones at Organization → **Settings** → **Secrets and variables** → **Agents** → **New organization secret** → choose **Repository access** → **Add secret**.

### Step B — Add the configuration

1. In the sidebar, under **Code, planning, and automation**, click **Copilot**, then **MCP servers**.
2. In the **MCP configuration** section, paste your JSON (example below).
3. Click **Save MCP configuration**. GitHub checks the syntax when you save.

**Example (Sentry).** Replace `READ-ONLY-TOOL-NAME` with tool names from the server's documentation:

```json
{
  "mcpServers": {
    "sentry": {
      "type": "local",
      "command": "npx",
      "args": ["@sentry/mcp-server@latest", "--host=$SENTRY_HOST"],
      "tools": ["READ-ONLY-TOOL-NAME"],
      "env": {
        "SENTRY_HOST": "https://contoso.sentry.io",
        "SENTRY_ACCESS_TOKEN": "$COPILOT_MCP_SENTRY_ACCESS_TOKEN"
      }
    }
  }
}
```

| Key | Required? | Notes |
|-----|-----------|-------|
| `type` | Yes | `local`, `stdio`, `http`, or `sse` |
| `tools` | Yes | List the tool names to allow. `"*"` allows every tool — avoid it |
| `command`, `args` | Yes (local) | How to start the server |
| `env` | No (local) | Literal values or `$COPILOT_MCP_...` references |
| `url` | Yes (remote) | The server's URL |
| `headers` | No (remote) | Literal values or `$COPILOT_MCP_...` references |

> 📌 **Limits:** the cloud agent uses MCP **tools** only (not resources or prompts), and doesn't support remote servers that use **OAuth**. This repository MCP configuration is shared with Copilot code review.

### Security review checklist (before saving)

- [ ] Which tools does the server expose, and which ones are write tools?
- [ ] Is `tools` limited to the tools you actually need?
- [ ] What credentials does it need, and are they least-privilege?
- [ ] What network access does it need?
- [ ] Does it meet your data classification policy?
- [ ] Has your security team approved it?

---

## 6️⃣ Control the Agent's Internet Access (Firewall)

**👤 Role:** GitHub **organization owner** (org-wide) or **repository administrator** (per repo) · **📍 Portal:** GitHub

**Navigate:** Organization → **Settings** → **Copilot** → **Internet access**

The firewall is on by default, with a **recommended allowlist** for common package registries, container registries, and certificate authorities.

**Steps (organization):**

1. In the sidebar, under **Code, planning, and automation**, click **Copilot**, then **Internet access**.
2. In the Copilot cloud agent section, set each setting:
   - **Enable firewall** → **Enabled**, **Disabled**, or **Let repositories decide** *(default)*.
   - **Recommended allowlist** → **Enabled**, **Disabled**, or **Let repositories decide** *(default)*.
   - **Allow repository custom rules** → **Enabled** *(default)* or **Disabled**.
3. To allow internal hosts for every repository, click **Organization custom allowlist**, enter a domain (for example `packages.contoso.corp`) or URL, and click **Add rule**.
4. When your list is complete, click **Save changes**.

**Steps (repository):** Repository → **Settings** → **Copilot** → **Internet access** → toggle **Enable firewall** / **Recommended allowlist** (only if the org chose **Let repositories decide**) → **Custom allowlist** → **Add rule** → **Save changes**.

> ⚠️ **Important:** The firewall only covers processes the agent starts with its Bash tool. It does **not** cover MCP server processes or steps in `copilot-setup-steps.yml`, and it's not a complete security control.

---

## 7️⃣ Review Repository Safety Settings

**👤 Role:** **Repository administrator** · **📍 Portal:** GitHub

**Navigate:** Repository → **Settings** → **Copilot** → **Cloud agent**

| Setting | Default | Recommendation |
|---------|---------|----------------|
| **Validation tools** (security checks and Copilot code review on the agent's own code) | On | Keep on unless they conflict with your own tools |
| **Require approval for workflow runs** (in **Actions workflow approval**) | On — someone must click **Approve and run workflows** in the PR | Keep on. Turning it off lets unreviewed agent code run your workflows and reach Actions secrets |

> 💡 **Tip:** Pair the agent with branch protection or rulesets so its pull requests need human review before merge.

---

## 8️⃣ Improve Agent Quality (Instructions & Custom Agents)

**👤 Role:** Write access to the repository (repo files) · **organization owner** (org files) · **enterprise owner** (enterprise setup)

### Custom instructions

| File | Scope |
|------|-------|
| `.github/copilot-instructions.md` | Whole repository |
| `.github/instructions/NAME.instructions.md` (with `applyTo` front matter) | Matching paths only |
| `AGENTS.md` | Agent instructions, nearest file wins |
| Organization → **Settings** → **Copilot** → **Custom instructions** | Whole organization |

Example `.github/copilot-instructions.md`:

```markdown
## Build & test
- Install with `npm ci`, test with `npm test`, lint with `npm run lint`
- All pull requests must pass CI

## Coding standards
- TypeScript strict mode; follow existing naming conventions
- Add unit tests for new functions
- Do not modify files in /config without approval
```

### Agent environment

Add `.github/workflows/copilot-setup-steps.yml` with a single job named `copilot-setup-steps` to preinstall tools and dependencies before the agent starts. Use **Agents** secrets (Section 5) for private registries — the agent can't see Actions, Codespaces, or Dependabot secrets.

### Custom agents

| Level | Where the agent profile lives |
|-------|-------------------------------|
| Repository | `.github/agents/AGENT-NAME.md` |
| Organization | `/agents/AGENT-NAME.md` in the org's `.github` or `.github-private` repository |
| Enterprise | `/agents/AGENT-NAME.md` in a `.github-private` repository that an enterprise owner designates in enterprise settings |

> 💡 **Enterprise tip:** On Enterprise → **AI controls** → **Agents**, the **Protect agent files using rulesets** section has a **Create ruleset** button. It limits who can edit agent profiles.

---

## 9️⃣ Monitor Agent Activity and Cost

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

| What | Where |
|------|-------|
| Recent sessions | Enterprise → **AI controls** → **Agent sessions** section → **View all** (last 24 hours). Click the search bar and press <kbd>Space</kbd> to filter |
| Audit trail | Enterprise → **AI controls** → **Audit logs** (bottom of the page) |
| Streaming | Audit log streaming of agent session events (public preview, EMU and GHE.com enterprises) |
| Spend | Enterprise → **Billing and licensing** → **AI usage** (AI credits) and **Usage** (Actions minutes) |

**What a session costs:**

- **AI credits** — based on the model and tokens used (billing SKU: *Copilot Cloud Agent*). Covered by AI credit budgets.
- **GitHub Actions minutes** — the agent runs on Actions runners.

> 💡 **Tip:** Clear, detailed issues mean fewer follow-up comments and fewer sessions — the cheapest way to control cost.

---

## 🧯 Known Errors & Resolutions

<details>
<summary><em>Show known errors table</em></summary>

> This section lists the known product errors and admin-facing symptoms that commonly occur with this workflow. Exact message text can vary by product rollout, tenant policy, and provider, so use the log or settings page named in the resolution to confirm the root cause.

| Error or symptom | Likely cause | Resolution |
|------------------|--------------|------------|
| **Page, tab, or button is missing** | Wrong account context, missing admin role, unavailable plan/add-on, or feature rollout not enabled for the selected enterprise/org/repo. | Switch to the correct account and scope, confirm the prerequisite role, verify licensing or add-on activation, then refresh the page. If the control is still absent, use the direct settings URL from the relevant GitHub Docs page and confirm the feature is available for your plan. |
| **Changes appear saved but behavior does not change** | Policy inheritance, cached UI state, propagation delay, or an overlapping enterprise/org/repo policy. | Reopen the settings page, verify the effective policy at the lowest affected scope, wait for propagation where documented, and check for a stricter policy at an enterprise or organization level. |
| **403, forbidden, or resource not accessible** | The signed-in user or token can see the page but lacks the specific permission for the action. | Use an enterprise owner, organization owner, repository admin, or token with the exact scopes/permissions listed in the runbook. For SAML-protected orgs, authorize the token or SSH key for SSO before retrying. |
| **Copilot feature, model, or policy is not visible** | Plan, license assignment, enterprise policy, org delegation, or feature rollout does not permit it. | Check enterprise AI controls, organization Copilot settings, assigned seat status, and the plan requirements for the feature. |
| **Copilot stops working for a user mid-cycle** | The user's user-level budget is used up, the shared AI credit pool is exhausted with **AI credits paid usage** disabled, or a spending limit with **Stop usage** was reached. | Check the user on **Billing and licensing** → **AI usage** and the budgets on **Budgets and alerts**; raise their budget, approve their budget request (not available for EMU enterprises), or enable AI credits paid usage. |
| **Content exclusions do not apply immediately** | Client policy cache, unsupported surface/mode, symlink/remote filesystem limitation, or indirect IDE context. | Reload the IDE policy, verify the exclusion syntax at enterprise/org/repo scope, and document surfaces where exclusions are limited. |
| **Usage metrics look empty or inconsistent** | Telemetry is disabled, data freshness delay applies, users are unlicensed, or different APIs report different scopes. | Enable the metrics policy, confirm seats and telemetry, wait for data freshness, and avoid comparing dashboards/API endpoints as if they share identical data models. |
| **Cloud agent or MCP action is denied** | Agent policy, MCP policy, repository permissions, secrets, or server allowlist does not permit the operation. | Review Enterprise AI controls > Agents/MCP, repo-level permissions, MCP server configuration, and audit logs for the denied action. |
| **PR body or comment warns that the firewall blocked a request** | The agent tried to reach a host that isn't on the allowlist. | Check the blocked address in the warning. If it's legitimate, add it under **Internet access** → **Custom allowlist** (repo) or **Organization custom allowlist**. |
| **MCP server works locally but not for the agent** | Secret name missing the `COPILOT_MCP_` prefix, remote server needs OAuth, or tool not listed in `tools`. | Rename the Agents secret with the prefix, use a token-based server, and add the tool name to `tools`. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>

### Q: Users can't assign issues to Copilot — what should we check?
**A:** Check, in order: (1) the enterprise **Copilot Cloud Agent** policy includes the organization, (2) the organization's **Copilot cloud agent** policy is **Enabled**, (3) the repository is allowed under **Repository access**, (4) the enterprise hasn't turned on **Block Copilot cloud agent in all repositories**, and (5) the user has a Copilot license **and** write permission to the repository.

---

### Q: An MCP server isn't working — how do I debug it?
**A:** Re-save the JSON under **Settings** → **Copilot** → **MCP servers** to confirm it's valid. Make sure every secret it uses is an **Agents** secret whose name starts with `COPILOT_MCP_`, that the tools you need are listed in `tools`, and that the server doesn't need OAuth. Then read the agent's session logs for the connection error.

---

### Q: Can the cloud agent reach private package registries?
**A:** Yes. Store credentials as **Agents** secrets or variables (repository or organization level), install dependencies in `copilot-setup-steps.yml`, and add the registry host to the firewall allowlist if needed. The agent can't use Actions, Codespaces, or Dependabot secrets.

---

### Q: The agent's output is low quality — how can we improve it?
**A:** Add `.github/copilot-instructions.md` with build, test, and style guidance, add path-specific `.github/instructions/*.instructions.md` files, preinstall dependencies with `copilot-setup-steps.yml`, and write issues with clear acceptance criteria. For repeatable work, create a custom agent.

---

### Q: How do we limit which repositories can use the agent?
**A:** Organization → **Settings** → **Copilot** → **Cloud agent** → **Repository access** → **Selected repositories** → check repositories → **Select**. To block the agent across every repository in the enterprise, use the enterprise **Block Copilot cloud agent** toggle.

---

### Q: Agent sessions are using too many AI credits — how do we control this?
**A:** Set user-level budgets and cost center budgets for AI credits, restrict which models are available, keep the pilot to a few repositories, and coach users to write detailed issues so fewer follow-up sessions are needed. Remember that sessions also use Actions minutes.

---

### Q: Is it safe to enable MCP servers from third parties?
**A:** Treat it as a governance decision. Copilot uses MCP tools without asking for approval, so review each server, allow only the tools you need (preferably read-only), give it least-privilege credentials, and have your security team approve it.

</details>

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Admin Controls (Models, Content Exclusion, Instructions) | `Copilot/Admin Controls (Models, Content Exclusion, Instructions).md` |
| Context Management (Spaces, Indexing, Instructions, Exclusions) | `Copilot/Context Management (Spaces, Indexing, Instructions, Exclusions).md` |
| Responsible AI Guardrails | `Copilot/Responsible AI Guardrails.md` |
| AI Credits Budget & Overage Planning | `Copilot/AI Credits Budget & Overage Planning.md` |
| Branch Protection Rules & Rulesets | `Governance/Branch Protection Rules & Rulesets.md` |

---

## 📝 Resources

| Resource | Link |
|----------|------|
| Enable Copilot cloud agent (enterprise) | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/administer-copilot/manage-for-enterprise/manage-agents/enable-copilot-cloud-agent) |
| Add Copilot cloud agent to your organization | [GitHub Docs](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-organization/add-copilot-cloud-agent) |
| MCP and Copilot cloud agent | [GitHub Docs](https://docs.github.com/en/copilot/concepts/agents/cloud-agent/mcp-and-cloud-agent) |
| Configure MCP servers for a repository | [GitHub Docs](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/configure-mcp-servers) |
| Customize the agent firewall | [GitHub Docs](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-the-firewall) |
| Configure secrets and variables for cloud agent | [GitHub Docs](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/configure-secrets-and-variables) |
| About custom agents | [GitHub Docs](https://docs.github.com/en/copilot/concepts/agents/cloud-agent/about-custom-agents) |
| Monitor agentic activity | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/administer-copilot/manage-for-enterprise/manage-agents/monitor-agentic-activity) |

---

*Last updated: October 2026*
