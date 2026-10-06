# 🛡️ GitHub Copilot Responsible AI Guardrails Runbook

> Platform controls and organizational policies to govern responsible AI use with GitHub Copilot

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [📋 Overview](#-overview)
- [1️⃣ Platform Controls (Admin-Configurable)](#1️⃣-platform-controls-admin-configurable)
- [2️⃣ Code Governance Controls](#2️⃣-code-governance-controls)
- [3️⃣ Custom Instructions for Standards](#3️⃣-custom-instructions-for-standards)
- [4️⃣ Mitigating "Vibe Coding"](#4️⃣-mitigating-vibe-coding)
- [5️⃣ Monitoring & Audit](#5️⃣-monitoring--audit)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- Content exclusions: `Enterprise → AI controls → Copilot → Content exclusion`
- Approved models only: `Enterprise → AI controls → Copilot → Configure models`
- Public code filter: `Enterprise → AI controls → Copilot` → **Privacy** → **Suggestions matching public code** → **Blocked**
- Agents and MCP: `Enterprise → AI controls → Agents` and `→ MCP`
- Required reviews and checks: `Org Settings → Repository → Rulesets → New ruleset → New branch ruleset`
- Audit: `Enterprise → Settings → Audit log` → search `action:copilot` or `actor:Copilot`

---

## ✅ Accuracy & Click-Path Notes

<details>
<summary><em>Show click-path conventions</em></summary>


- Reviewed against current public GitHub documentation in October 2026. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
- When a path starts with `Enterprise`, begin at GitHub, click your profile picture, click `Enterprise` (managed/EMU accounts) — or open the `Enterprises` page at github.com/settings/enterprises (standard accounts) —, select the enterprise, then continue with the listed top tab or left-sidebar item.
- When a path starts with `Organization` or `Org`, begin at GitHub, click your profile picture, click `Organizations`, select the organization, click `Settings`, then continue with the listed sidebar item.
- When a path starts with `Repository`, `Repo`, or a repository name, open the repository, click the `Settings` tab, then continue with the listed sidebar item.
- When a path starts with a vendor portal such as `Microsoft Entra admin center`, `Azure portal`, `Okta Admin Console`, `PingFederate`, `PingOne`, `OneLogin`, `AD FS Management`, `Visual Studio Admin Portal`, or `Azure DevOps`, sign in to that admin portal first, select the tenant, application, or project named in the step, then follow each listed blade, tab, button, and confirmation in order.
- If the expected button is missing, verify you are signed in with the role named in Prerequisites, the feature or license is enabled, and the object is owned by the selected enterprise, organization, or repository. Use page search only to locate the same page, not to skip required confirmation, test, save, or consent clicks.

</details>

---

## ✅ Prerequisites

| Requirement | Who / Role needed | ✓ |
|-------------|-------------------|---|
| Copilot Business or Copilot Enterprise | GitHub **enterprise owner** | ☐ |
| Enterprise Copilot policies, models, MCP, agents, and audit log | GitHub **enterprise owner** | ☐ |
| Organization policies, custom instructions, rulesets, and security configurations | GitHub **organization owner** | ☐ |
| Code scanning and secret scanning on private repositories | **GitHub Code Security** and **GitHub Secret Protection** (or GitHub Advanced Security) licenses | ☐ |
| A written AI acceptable-use policy | Leadership, legal, and security | ☐ |

---

## 📋 Overview

Responsible AI governance needs **both** platform controls (what GitHub provides) **and** organizational controls (what you enforce through policy and process).

| Layer | Who manages it | Examples |
|-------|---------------|---------|
| **Platform controls** | Enterprise and org owners | Content exclusion, approved models, public code filter, agent and MCP policies |
| **Code governance** | Org owners and repo admins | Rulesets, required reviews, status checks, CODEOWNERS, code scanning |
| **Organizational policy** | Leadership and security | Acceptable use, training, approval process |
| **Developer practice** | Individual developers | Prompt hygiene, careful review of AI output |

---

## 1️⃣ Platform Controls (Admin-Configurable)

**👤 Role:** GitHub **enterprise owner** (or **organization owner** where delegated) · **📍 Portal:** GitHub

| Control | Click path | What to set |
|---------|-----------|-------------|
| **Content exclusion** | Enterprise → **AI controls** → **Copilot** → **Content exclusion** *(also at Org → Settings → Copilot → Content exclusion)* | Exclude `**/.env`, `**/secrets/**`, key files, and sensitive repositories |
| **Approved models** | Enterprise → **AI controls** → **Copilot** → **Configure models** | Enable only models your security team approved |
| **Public code filter** | Enterprise → **AI controls** → **Copilot** → **Privacy** section → **Suggestions matching public code** *(also at Org → Settings → Copilot → Policies)* | **Blocked** — Copilot won't show suggestions that match public code |
| **Cloud agent** | Enterprise → **AI controls** → **Agents** → **Copilot Cloud Agent** | Enable for selected organizations only, after a pilot |
| **MCP servers** | Enterprise → **AI controls** → **MCP** → **MCP servers in Copilot** | Allow only after a security review of each server |
| **Third-party coding agents and agent apps** | Enterprise → **AI controls** → **Agents** | Enable only the agents you've approved |
| **Default availability** | Enterprise → **AI controls** → **Copilot** → default availability policies | Decide before **October 22, 2026**: unconfigured GA features turn on then unless you disable the **Default policy for new features** or set each feature explicitly |
| **Session syncing** | Enterprise → **AI controls** → **Copilot** policy pages → **Store local sessions in the Cloud** (set per client, such as Copilot CLI and VS Code) | Decide whether local sessions sync to GitHub. Unconfigured = local only |

> 📌 **Defaults to know:** **Suggestions matching public code** is **Allowed** by default for Copilot Business users. Cloud agent and third-party MCP servers are **off** by default.

> ℹ️ **Copilot Extensions** (the GitHub App–based extensions) were retired on November 10, 2025. Govern tool access through the **MCP** and **Agents** policies instead.

### Data handling — what to tell your legal and privacy teams

| Topic | What GitHub documents |
|-------|----------------------|
| **Terms** | Direct purchases: GitHub Generative AI Services Terms. Through Microsoft: Microsoft Product Terms. Plus the GitHub Data Protection Agreement |
| **Model training** | GitHub's Trust Center FAQ covers training on Copilot Business and Enterprise data. Repository indexes are explicitly **not** used for model training |
| **Chat history on GitHub.com** | Up to 100 recent conversations. Messages are deleted after **28 days** |
| **Cloud agent sessions** | Logs stay on GitHub.com and are visible to people with access to the repository |
| **Copilot CLI and app sessions** | Stored locally. Synced to the user's GitHub account only when **Store local sessions in the Cloud** is set to at least **View from cloud** (disabled or unconfigured = local only) |
| **BYOK models** | Prompts go to your provider and follow its retention terms |

> 💡 Send legal, privacy, and compliance teams the GitHub Enterprise Trust Center (`ghec.github.trust.page`) for attestations and FAQs.

---

## 2️⃣ Code Governance Controls

**👤 Role:** GitHub **organization owner** · **📍 Portal:** GitHub

**Navigate:** Organization → **Settings** → under **Code, planning, and automation**, click **Repository** → **Rulesets**

### Create a ruleset for AI-assisted code

1. Click **New ruleset** → **New branch ruleset**.
2. Enter a **Ruleset name** (for example `ai-guardrails`) and set **Enforcement status** to **Active**.
3. Under **Target repositories**, choose the repositories. Under **Target branches**, click **Add target** → **Include default branch**.
4. Under **Branch rules**, select:
   - **Require a pull request before merging** → set **Required approvals** to 1 or more. Optionally turn on **Require review from Code Owners**.
   - **Require status checks to pass before merging** → **Add checks** → add your test and lint checks.
   - **Require code scanning results** → add **CodeQL** with the alert thresholds you want.
   - **Block force pushes**.
   - *(Optional)* **Automatically request Copilot code review**.
5. Click **Create**.

### CODEOWNERS for critical paths

```
# .github/CODEOWNERS
/src/auth/          @octo-org/security-team
/config/            @octo-org/platform-team
/.github/workflows/ @octo-org/devops-team
```

### Built-in protections for Copilot cloud agent

- It opens **draft** pull requests and can't mark them ready, approve, or merge them — a human must.
- The person who asked for the PR can't approve it.
- When the PR isn't attributed to a person, one **extra approval** is required (if the repo already requires approvals).
- Actions workflows don't run on its PRs until someone clicks **Approve and run workflows** (unless a repo admin turns that off).

> ⚠️ **Copilot approvals (public preview):** at Repository → **Settings** → **Copilot** → **Code review** → **Auto-approval**, you can let Copilot approve pull requests and count those approvals toward merge requirements. For guardrails, leave **Allow Copilot approvals to count toward merge requirements** **off** unless you scope it with **File paths**.

---

## 3️⃣ Custom Instructions for Standards

| Level | Where | Applies to |
|-------|-------|-----------|
| **Organization** | Org → **Settings** → **Copilot** → **Custom instructions** → **Save changes** | Copilot Chat, code review, and cloud agent **on GitHub.com** only |
| **Repository** | `.github/copilot-instructions.md` | Copilot on GitHub.com and in IDEs |
| **Path-specific** | `.github/instructions/NAME.instructions.md` with `applyTo` | Cloud agent and code review on GitHub.com, and supported IDEs |

Example security instructions:
- "Never use `eval()` or `exec()`."
- "Always sanitize user input before database queries — use parameterized queries, never string concatenation."
- "Follow OWASP Top 10 secure coding practices."
- "Never hard-code secrets, API keys, or credentials — read them from environment variables or a secrets manager."

> 📌 Instructions guide Copilot; they don't enforce anything. Pair them with rulesets and scanning.

---

## 4️⃣ Mitigating "Vibe Coding"

"Vibe coding" means letting AI generate code quickly with little human review. Risks: technical debt, security vulnerabilities, and code nobody understands.

### Governance controls

| Control | Click path | Effect |
|---------|-----------|--------|
| Required PR reviews | Org → **Settings** → **Repository** → **Rulesets** | AI code must be reviewed by a human |
| Code scanning (CodeQL) | Org → **Settings** → **Advanced Security** → **Configurations** → apply a configuration with code scanning | Finds vulnerabilities regardless of who wrote the code |
| Secret scanning + push protection | Same configuration page — enable **Secret Protection** features and **Push protection** | Blocks leaked credentials before they're pushed |
| Block direct pushes | Ruleset rules **Require a pull request before merging** + **Block force pushes** | No unreviewed changes reach the default branch |
| Content exclusion | Org → **Settings** → **Copilot** → **Content exclusion** | Keeps sensitive files out of AI context |

### Additional best practices

- Use the **Copilot impact** and **Code generation** dashboards to see where agent-generated code is growing, and spot-check those teams' review quality.
- Train developers: Copilot is an accelerator, not a replacement for understanding.
- Run periodic code quality reviews on AI-heavy pull requests.

---

## 5️⃣ Monitoring & Audit

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

| What | Click path |
|------|-----------|
| Usage and adoption | Enterprise → **Insights** → **Copilot usage** / **Code generation** / **Copilot impact** |
| Copilot settings and license changes | Enterprise → **Settings** → **Audit log** → search `action:copilot` |
| Agent activity | Same audit log → search `actor:Copilot`, or Enterprise → **AI controls** → **Audit logs** |
| Recent agent sessions | Enterprise → **AI controls** → **Agent sessions** → **View all** |
| Security findings | Organization → **Security and quality** tab → **Overview** |

> 📌 The audit log keeps **180 days** of events and does **not** include client prompts. Stream it to your SIEM for long-term history and alerting.

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
| **Copilot stops working for a user mid-cycle** | The user's user-level budget is used up, the shared AI credit pool is exhausted with **AI credits paid usage** disabled, or a spending limit with **Stop usage** was reached. | Check the user on **Billing and licensing** → **AI usage** and the budgets on **Budgets and alerts**; raise their budget, approve their budget request, or enable AI credits paid usage. |
| **Content exclusions do not apply immediately** | Client policy cache, unsupported surface/mode, symlink/remote filesystem limitation, or indirect IDE context. | Reload the IDE policy, verify the exclusion syntax at enterprise/org/repo scope, and document surfaces where exclusions are limited. |
| **Usage metrics look empty or inconsistent** | Telemetry is disabled, data freshness delay applies, users are unlicensed, or different APIs report different scopes. | Enable the metrics policy, confirm seats and telemetry, wait for data freshness, and avoid comparing dashboards/API endpoints as if they share identical data models. |
| **Cloud agent or MCP action is denied** | Agent policy, MCP policy, repository permissions, secrets, or server allowlist does not permit the operation. | Review Enterprise AI controls > Agents/MCP, repo-level permissions, MCP server configuration, and audit logs for the denied action. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>


### Q: Developers are accepting all Copilot suggestions without review — how do we address this?
**A:** Require pull request reviews with a ruleset (Org → **Settings** → **Repository** → **Rulesets**), require CodeQL results and status checks, and use the **Code generation** and **Copilot impact** dashboards to find teams where AI-generated code is growing fast. Follow up with coaching, not punishment.

---

### Q: Copilot is suggesting code that matches public repositories — how do we prevent this?
**A:** Set **Suggestions matching public code** to **Blocked** — at Enterprise → **AI controls** → **Copilot** → **Privacy**, or at Org → **Settings** → **Copilot** → **Policies** if the enterprise lets organizations decide. It's **Allowed** by default for Copilot Business.

---

### Q: How do we enforce responsible AI policy across the entire enterprise?
**A:** Layer the controls: enterprise policies for models, agents, and MCP; content exclusion; organization custom instructions; rulesets for reviews and checks; and code scanning plus secret scanning through security configurations. No single control is enough.

---

### Q: Copilot is suggesting secrets or credentials in code — how do we stop this?
**A:** Turn on secret scanning with **push protection** through a security configuration (Org → **Settings** → **Advanced Security** → **Configurations**). Exclude sensitive files (`.env`, key files) with content exclusion, and add an instruction such as "Never hard-code secrets — read them from environment variables."

---

### Q: How do we audit what Copilot is being used for across the enterprise?
**A:** Use the enterprise audit log (Enterprise → **Settings** → **Audit log**) with `action:copilot` for settings and license changes and `actor:Copilot` for agent activity, and stream it to your SIEM. Use **Insights** dashboards for adoption. The audit log doesn't contain users' prompts.

---

### Q: Does GitHub use our code or prompts to train AI models?
**A:** Point your legal team to the GitHub Generative AI Services Terms (or Microsoft Product Terms), the GitHub Data Protection Agreement, and the GitHub Enterprise Trust Center FAQ, which cover how Copilot Business and Enterprise data is used. GitHub's docs state that repository indexes aren't used for model training. Note that some data **is** stored — for example, Chat messages on GitHub.com for 28 days and cloud agent session logs.

---

### Q: How do we prevent "vibe coding" from introducing security vulnerabilities?
**A:** Require human pull request reviews, require CodeQL results and status checks in rulesets, block force pushes, and keep Copilot approvals from counting toward merge requirements. Track AI-generated code volume on the **Code generation** dashboard.

---

### Q: Can we restrict which third-party tools Copilot can use?
**A:** Yes. GitHub App–based Copilot Extensions were retired on November 10, 2025. Use Enterprise → **AI controls** → **MCP** → **MCP servers in Copilot** to control MCP servers, and **AI controls** → **Agents** to control third-party agents and agent apps. Review each one with your security team before enabling it.

</details>

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Admin Controls (Models, Content Exclusion, Instructions) | `Copilot/Admin Controls (Models, Content Exclusion, Instructions).md` |
| Context Management (Spaces, Indexing, Instructions, Exclusions) | `Copilot/Context Management (Spaces, Indexing, Instructions, Exclusions).md` |
| Branch Protection Rules & Rulesets | `Governance/Branch Protection Rules & Rulesets.md` |
| Code Scanning (CodeQL) Enablement & Troubleshooting | `Security/Code Scanning (CodeQL) Enablement & Troubleshooting.md` |

---

## 📝 Resources

| Resource | Link |
|----------|------|
| GitHub Enterprise Trust Center | [Trust Center](https://ghec.github.trust.page) |
| Resources for legal, compliance, and security approval | [GitHub Docs](https://docs.github.com/en/copilot/tutorials/roll-out-at-scale/govern-at-scale/resources-for-approval) |
| Content exclusion | [GitHub Docs](https://docs.github.com/en/copilot/how-tos/configure-content-exclusion/exclude-content-from-copilot) |
| Managing Copilot policies for your enterprise | [GitHub Docs](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-enterprise/manage-enterprise-policies) |
| Risks and mitigations for Copilot agents | [GitHub Docs](https://docs.github.com/en/copilot/concepts/security-governance-and-network-settings/risks-and-mitigations) |
| Review Copilot audit logs | [GitHub Docs](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-enterprise/review-audit-logs) |
| Available rules for rulesets | [GitHub Docs](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets) |

---

*Last updated: October 2026*
