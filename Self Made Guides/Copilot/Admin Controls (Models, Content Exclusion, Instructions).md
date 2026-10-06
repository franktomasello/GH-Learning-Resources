# 🤖 GitHub Copilot Admin Controls Runbook

> **Complete guide to managing Copilot AI models, content exclusions, custom instructions, and policy controls across enterprise and organization levels**

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [📋 Overview](#-overview)
- [🏗️ Policy Hierarchy](#️-policy-hierarchy)
- [1️⃣ Restrict AI Models at the Enterprise Level](#1️⃣-restrict-ai-models-at-the-enterprise-level)
- [2️⃣ Restrict AI Models at the Organization Level](#2️⃣-restrict-ai-models-at-the-organization-level)
- [3️⃣ Set Up Content Exclusions at the Enterprise Level](#3️⃣-set-up-content-exclusions-at-the-enterprise-level)
- [4️⃣ Set Up Content Exclusions at the Organization Level](#4️⃣-set-up-content-exclusions-at-the-organization-level)
- [5️⃣ Create Organization-Level Custom Instructions](#5️⃣-create-organization-level-custom-instructions)
- [6️⃣ Create Repository-Level Custom Instructions](#6️⃣-create-repository-level-custom-instructions)
- [7️⃣ Enable or Disable the Public Code Filter](#7️⃣-enable-or-disable-the-public-code-filter)
- [8️⃣ Manage MCP Server Policy at the Enterprise Level](#8️⃣-manage-mcp-server-policy-at-the-enterprise-level)
- [📝 Additional Notes](#-additional-notes)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📚 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- Restrict models: `Enterprise → AI controls → Copilot → Configure models` — set each model to **Enabled**, **Disabled**, or **Delegate**
- Content exclusions: `Enterprise → AI controls → Copilot → Content exclusion` (or `Org Settings → Copilot → Content exclusion`) — enter repositories and paths
- Org custom instructions: `Org Settings → Copilot → Custom instructions` → **Save changes**
- Repo instructions: `.github/copilot-instructions.md`, path-specific `.github/instructions/**/*.instructions.md` (with `applyTo`), or `AGENTS.md`
- MCP: `Enterprise → AI controls → MCP` → **MCP servers in Copilot**

---

## ✅ Accuracy & Click-Path Notes

<details>
<summary><em>Show click-path conventions</em></summary>


- Reviewed against current public GitHub documentation in October 2026 where public documentation is available. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
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
| GitHub Enterprise Cloud enterprise with Copilot Business or Copilot Enterprise | GitHub **enterprise owner** | ☐ |
| Enterprise-level policies, models, content exclusion, and MCP | GitHub **enterprise owner** | ☐ |
| Organization-level policies, models, content exclusion, and custom instructions | GitHub **organization owner** | ☐ |
| Repository custom instructions | Write access to the repository (to commit the files) | ☐ |

---

## 📋 Overview

This runbook covers the admin controls that govern which models people can use, what content Copilot can see, and what instructions guide it:

| Control | Scope | Purpose |
|---------|-------|---------|
| **Model availability** | Enterprise / Org | Control which AI models people can use |
| **Content exclusion** | Enterprise / Org / Repo | Keep sensitive files out of Copilot's context |
| **Custom instructions** | Org / Repo / Personal | Guide Copilot's behavior with standards and project context |
| **Suggestions matching public code** | Enterprise / Org | Block suggestions that match public code |
| **MCP servers** | Enterprise / Org | Control use of Model Context Protocol servers |

> 📌 **Copilot Extensions are retired.** GitHub App-based Copilot Extensions were sunset on November 10, 2025 in favor of MCP servers, so this runbook covers MCP policy instead.

---

## 🏗️ Policy Hierarchy

Understanding where each control lives and how they cascade:

| Level | Role | Example |
|-------|------|---------|
| **Enterprise** | Guardrails | Disable models, set content exclusions, set MCP policy |
| **Organization** | Standards | Restrict models further, add org exclusions, set org custom instructions |
| **Repository** | Project truth | Repository instructions that reflect the actual codebase |
| **Content exclusion** | Sensitive paths | Keep secrets, configs, and proprietary logic out of Copilot's context |

> 💡 **Tip:** Enterprise policies act as a ceiling. If the enterprise sets a policy, organizations can't change it — only policies left at **No policy** (or models set to **Delegate**) are decided by organizations.

> 📌 **No Save button for policies and models:** dropdowns and toggles on these pages apply as soon as you select them.

> ⏰ **Act before October 22, 2026 — default availability policies.** Two policies decide what happens to anything left **Unconfigured**:
> - **Default availability for released models** (already active): new GA models and models shown as **Delegate to Default Policy** follow it. Pre-GA models, open-weight models, and models outside GitHub's data retention agreement stay off regardless.
> - **Default policy for new features** (applies from **October 22, 2026**, **enabled by default**): unconfigured GA features on **AI controls** → **Copilot** → **Features & clients** — plus **Copilot code review** and **MCP servers in Copilot** — will turn on.
>
> To keep control, either disable these default policies, or explicitly set every feature and model you care about to **Enabled** or **Disabled**. The GHE.com restrictive model policies and **Store local sessions in the Cloud** aren't affected.

---

## 1️⃣ Restrict AI Models at the Enterprise Level

*Control which models are available across all organizations*

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **AI controls** → **Copilot** *(sidebar)* → **Configure models**

**Steps:**

1. At the top of the enterprise page, click **AI controls**, then **Copilot** in the sidebar.
2. Click **Configure models**.
3. Set each model to:
   - **Enabled** — available to every user
   - **Disabled** — unavailable everywhere in the enterprise
   - **Delegate** — each organization decides

> ⚠️ **Important:** Organization owners can't re-enable a model the enterprise has set to **Disabled**. Disabled models don't appear in users' model pickers.

> 💡 **Tip:** Model prices differ a lot under usage-based billing — see [Models and pricing](https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing) before enabling expensive models broadly.

---

## 2️⃣ Restrict AI Models at the Organization Level

*Decide models the enterprise has delegated to organizations*

**👤 Role:** GitHub **organization owner** · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Organizations** → *[organization]* → **Settings** → **Copilot** *(sidebar, under "Code, planning, and automation")* → **Models**

**Steps:**

1. Click **Models**.
2. For each model you can control, open its dropdown and choose **Enabled** or **Disabled**.

> 💡 **Tip:** Organizations can only decide models the enterprise set to **Delegate**. Models not configured by the organization follow its **Default availability for released models** policy.

---

## 3️⃣ Set Up Content Exclusions at the Enterprise Level

*Keep specific files and repositories out of Copilot's context across all organizations*

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **AI controls** → **Copilot** *(sidebar)* → **Content exclusion**

**Steps:**

1. Click **Content exclusion**.
2. In the text box, enter the repositories and paths to exclude, one pattern per line. Use `"*":` to match any repository, or a repository reference — its full clone URL (or, in organization settings, just the repository name) — followed by its paths:

```yaml
"*":
  - "**/.env"
  - "**/secrets/**"
https://github.com/octo-org/payments-service.git:
  - "/config/production.*"
  - "**/*.pem"
```

3. Save your changes.

> ⚠️ **Where it applies:** content exclusion works for inline suggestions and Copilot Chat. It is **not** supported in Edit and Agent modes of Copilot Chat in IDEs, and support in other surfaces has changed over time — check GitHub's [content exclusion](https://docs.github.com/en/copilot/concepts/context/content-exclusion) page for the current list. An IDE can also pass Copilot indirect information from excluded files, such as type information and hover definitions.

---

## 4️⃣ Set Up Content Exclusions at the Organization Level

*Add organization-specific exclusions on top of enterprise exclusions*

**👤 Role:** GitHub **organization owner** · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Organizations** → *[organization]* → **Settings** → **Copilot** → **Content exclusion**

**Steps:**

1. Click **Content exclusion**.
2. In the box under **Repositories and paths to exclude**, enter the patterns:

| Pattern | What it excludes |
|---------|------------------|
| `"*": ["**/.env"]` | Every `.env` file, in any repository or location |
| `"*": ["**/secrets/**"]` | Everything under any `secrets` directory |
| `my-repo: ["/config/production.*"]` | Production config files in `my-repo` |
| `"*": ["**/*.pem"]` | All PEM certificate files |

3. Save your changes.

> 💡 **Tip:** Organization exclusions add to enterprise exclusions — you don't need to repeat enterprise patterns. Repository admins can also exclude paths in a single repository at **Repo** → **Settings** → **Copilot** → **Content exclusion**.

---

## 5️⃣ Create Organization-Level Custom Instructions

*Set standards that apply to Copilot interactions in the organization's context on GitHub.com*

**👤 Role:** GitHub **organization owner** · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Organizations** → *[organization]* → **Settings** → **Copilot** → **Custom instructions**

**Steps:**

1. Click **Custom instructions**.
2. Under **Preferences and instructions**, write your instructions in plain language — for example, one per line.
3. Click **Save changes**.

**Example instructions:**

- "Always use TypeScript strict mode"
- "Follow the Google Java Style Guide"
- "Include JSDoc comments on all public functions"
- "Use snake_case for Python variables and function names"

> 📌 **Where they apply:** organization custom instructions are used by Copilot Chat, Copilot code review, and Copilot cloud agent **on GitHub.com**. They're available with Copilot Business and Copilot Enterprise.

---

## 6️⃣ Create Repository-Level Custom Instructions

*Provide project-specific context that lives alongside the code*

### A) General Repository Instructions

Create this file in the repository:

```
.github/copilot-instructions.md
```

It applies to all Copilot requests made in the context of the repository.

### B) Task-Specific Instruction Files

Create path-specific instruction files anywhere under:

```
.github/instructions/**/*.instructions.md
```

Use the `applyTo` front matter to choose which files they apply to:

```markdown
---
applyTo: "**/*.test.ts"
---

When writing tests, use Vitest as the test framework.
Always include edge case tests.
Use descriptive test names following the pattern: "should [expected behavior] when [condition]".
```

> 💡 **Tip:** Repository instructions are version-controlled and travel with the code. Agents also read `AGENTS.md` files.

### Instruction Precedence

All relevant instructions are sent to Copilot together. When they conflict, higher entries win:

| Priority | Source | Scope |
|----------|--------|-------|
| 1 (highest) | **Personal** instructions | Just you |
| 2 | Path-specific `.github/instructions/**/*.instructions.md` | Matching files in the repository |
| 3 | Repository-wide `.github/copilot-instructions.md` | The whole repository |
| 4 | Agent instructions (e.g., `AGENTS.md`) | The repository, for agents |
| 5 | **Organization** custom instructions | The organization (on GitHub.com) |

---

## 7️⃣ Enable or Disable the Public Code Filter

*Control whether Copilot suggests code that matches public code*

**👤 Role:** GitHub **enterprise owner** (enterprise) or **organization owner** (organization) · **📍 Portal:** GitHub

**Navigate (enterprise):** Enterprise → **AI controls** → **Copilot** *(sidebar)* → **Suggestions matching public code**
**Navigate (organization):** Organization → **Settings** → **Copilot** → **Policies** → **Suggestions matching public code**

**Options:**

| Setting | Effect |
|---------|--------|
| **Blocked** | Copilot filters out suggestions that match public code |
| **Allowed** | Copilot can show matching suggestions, with references to the matching code |
| **No policy** *(enterprise only)* | Each organization decides |

> 💡 **Tip:** Agree this setting with your legal team. The selection applies immediately — there is no Save button.

---

## 8️⃣ Manage MCP Server Policy at the Enterprise Level

*Control whether and how Model Context Protocol (MCP) servers can be used with Copilot*

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **AI controls** → **MCP** *(sidebar)*

**Steps:**

1. Set the **MCP servers in Copilot** policy (for example, **Enabled everywhere**, or disabled).
2. *(Optional)* To limit which servers can run, enter your registry in **MCP Registry URL** and click **Save**, then under **Restrict MCP access to registry servers** choose an option such as **Registry only**.

> 📌 **What this policy covers:** it controls MCP use in Copilot where MCP support is generally available. It doesn't control access to the GitHub MCP server from third-party tools such as Cursor or Claude.

> 💡 **Tip:** GitHub recommends enforcing an MCP allowlist with your enterprise's `managed-settings.json` file, which users can't override; custom-registry allowlists are in public preview. Allowlist controls require Copilot Business or Copilot Enterprise.

---

## 📝 Additional Notes

> 💡 **Customization:** Settings wording and layout can vary by plan and rollout. The paths above reflect GitHub's documentation as of October 2026.

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


### Q: Can we restrict models at the org level if the enterprise allows them?
**A:** Only if the enterprise **delegates** the decision. In **Configure models**, a model set to **Enabled** or **Disabled** at the enterprise is enforced for everyone; a model set to **Delegate** lets each organization choose **Enabled** or **Disabled** under **Settings** → **Copilot** → **Models**.

---

### Q: My content exclusion patterns are not working — what's wrong?
**A:** Check four things: (1) the syntax — `"*":` for any repository, a bare repository name (organization settings) or full clone URL as the key, and quoted path patterns; (2) paths that start with `/` are relative to the repository root, while patterns like `**/.env` match anywhere; (3) where you're testing — exclusions aren't supported in Edit and Agent modes of Copilot Chat in IDEs; and (4) timing — IDEs that already loaded the settings can take up to 30 minutes to pick up changes, and reloading the IDE applies them sooner.

---

### Q: Can we see which files are currently excluded?
**A:** There is no dashboard or report that lists excluded files. The best way to test is to ask Copilot Chat about content in an excluded file — if the exclusion is working, Copilot will not reference that file's content.

---

### Q: Custom instructions are not being followed — what should I check?
**A:** Confirm the repository file is exactly `.github/copilot-instructions.md`, and that path-specific files live under `.github/instructions/` with an `applyTo` pattern that matches the files you're working on. Remember that **personal** instructions take precedence over repository and organization instructions when they conflict, and organization instructions only apply on GitHub.com (Copilot Chat, code review, and cloud agent). Also check that the Copilot feature or IDE you're using supports that instruction type.

---

### Q: Can we enforce custom instructions across all repos in the org?
**A:** Organization custom instructions cover every repository in the organization, but only for Copilot Chat, Copilot code review, and Copilot cloud agent **on GitHub.com**. For IDE use, add standards to each repository's `.github/copilot-instructions.md` (a repository template helps new repos start with one).

---

### Q: We disabled a model at the enterprise level but users still see it — why?
**A:** Confirm the model is set to **Disabled** (not **Delegate**) in Enterprise → **AI controls** → **Copilot** → **Configure models**. Then ask users to reload their IDE or start a new Copilot Chat session — clients can take a little while to pick up policy changes.

---

### Q: Can content exclusions block Copilot code completions and Chat separately?
**A:** No. An exclusion applies to both inline suggestions and Copilot Chat; you can't exclude a file from one but not the other. Note that exclusions aren't supported in Edit and Agent modes of Copilot Chat in IDEs.

---

### Q: What happens if org-level and enterprise-level exclusions overlap?
**A:** Organization-level exclusions are additive to enterprise-level exclusions. You do not need to duplicate enterprise exclusions at the org level. The most restrictive combination of all applicable exclusions applies.

</details>

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Seat Assignment & Enablement | `Copilot/Seat Assignment & Enablement.md` |
| BYOK (Bring Your Own Key) Configuration | `Copilot/BYOK (Bring Your Own Key) Configuration.md` |
| Context Management (Spaces, Indexing, Instructions, Exclusions) | `Copilot/Context Management (Spaces, Indexing, Instructions, Exclusions).md` |
| Responsible AI Guardrails | `Copilot/Responsible AI Guardrails.md` |
| Cloud Agent & MCP Configuration | `Copilot/Cloud Agent & MCP Configuration.md` |

---

## 📚 Resources

- [Managing policies and features for Copilot in your enterprise](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-enterprise/manage-enterprise-policies)
- [Managing policies and features for Copilot in your organization](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-organization/manage-policies)
- [Default availability of features and models](https://docs.github.com/en/copilot/concepts/enterprise/default-availability)
- [Excluding content from GitHub Copilot](https://docs.github.com/en/copilot/how-tos/configure-content-exclusion/exclude-content-from-copilot)
- [Content exclusion for GitHub Copilot](https://docs.github.com/en/copilot/concepts/context/content-exclusion)
- [Adding organization custom instructions](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-organization-instructions)
- [About customizing Copilot responses](https://docs.github.com/en/copilot/concepts/prompting/response-customization)
- [Restrict MCP server access to a custom registry](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-mcp-usage/configure-mcp-server-access)
- [Sunset notice: GitHub App-based Copilot Extensions](https://github.blog/changelog/2025-09-24-deprecate-github-copilot-extensions-github-apps/)

---

*Last updated: October 2026*
