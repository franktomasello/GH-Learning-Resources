# 🛡️ GitHub Branch Protection & Rulesets Runbook

> **Complete guide to configuring branch protection at the repo, org, and enterprise level using classic rules and modern rulesets**

| 🧭 **At a Glance** | |
|---|---|
| **Goal** | Protect important branches with reviews and checks |
| **Use this when** | Setting up governance, or a merge is unexpectedly blocked |
| **People you need** | Repository admins; organization owners; enterprise owners |
| **Where you click** | GitHub (repo, org, and enterprise settings) |
| **End result** | Rulesets that require reviews and checks, with a controlled bypass list |
| **New to a term?** | See the [Glossary](../Glossary.md) for plain-English definitions |

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [📋 Overview](#-overview)
- [1️⃣ Classic Branch Protection (Repo-Level)](#1️⃣-classic-branch-protection-repo-level)
- [2️⃣ Repo-Level Rulesets (Modern, Recommended)](#2️⃣-repo-level-rulesets-modern-recommended)
- [3️⃣ Org-Level Rulesets (Enforce Across Repos)](#3️⃣-org-level-rulesets-enforce-across-repos)
- [4️⃣ Enterprise-Level Rulesets](#4️⃣-enterprise-level-rulesets)
- [5️⃣ Finding Which Rule Is Enforcing Approval Requirements](#5️⃣-finding-which-rule-is-enforcing-approval-requirements)
- [6️⃣ CODEOWNERS for Path-Specific Approvals](#6️⃣-codeowners-for-path-specific-approvals)
- [7️⃣ Recommended Baseline Configuration](#7️⃣-recommended-baseline-configuration)
- [8️⃣ Bypass Lists for Admins & Automation (Rulesets Feature)](#8️⃣-bypass-lists-for-admins--automation-rulesets-feature)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Repo ruleset:** `Repo → Settings → Rulesets → Rulesets → New ruleset → New branch ruleset` → name → **Enforcement status: Active** → targets → rules → **Create**
- **Org ruleset:** `Org → Settings → Repository → Rulesets → New ruleset → New branch ruleset`
- **Enterprise ruleset:** `Enterprise → Policies → Code → New ruleset → New branch ruleset`
- **Classic rule (repo):** `Repo → Settings → Branches → Add classic branch protection rule` → **Create**
- **CODEOWNERS:** commit `.github/CODEOWNERS`, then turn on **Require review from Code Owners**

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
| Repository rulesets and classic branch protection | **Repository administrator** (or a custom role with "edit repository rules") | ☐ |
| Organization rulesets | **Organization owner** — GitHub Team or GitHub Enterprise Cloud | ☐ |
| Enterprise rulesets | **Enterprise owner** — GitHub Enterprise Cloud | ☐ |
| Branch naming conventions | Platform team | ☐ |
| Exact names of the CI checks you'll require | CI owners | ☐ |

---

## 📋 Overview

| Method | Scope | Best for |
|--------|-------|----------|
| **Repository rulesets** | One repository | Modern, flexible protection with bypass lists and statuses |
| **Organization rulesets** | Many or all repositories in an organization | Consistent policy across teams |
| **Enterprise rulesets** | Many or all organizations in the enterprise | Enterprise-wide governance |
| **Classic branch protection** | One repository | Existing setups and simple rules |
| **CODEOWNERS** | Paths within a repository | Requiring domain experts to approve specific files |

> 💡 **Tip:** Prefer **rulesets**. They support statuses (Active / Evaluate / Disabled), bypass lists, layering, and organization- and enterprise-level targeting — and anyone with read access can see them.

> ⚠️ **Default status:** a new ruleset starts as **Disabled**. Change **Enforcement status** to **Active**, or it won't enforce anything.

---

## 1️⃣ Classic Branch Protection (Repo-Level)

*Still supported, but rulesets are preferred.*

**👤 Role:** **Repository administrator** · **📍 Portal:** GitHub

**Navigate:** Repository → **Settings** → **Branches** *(under "Code, planning, and automation")*

**Steps:**

1. Under **Branch protection rules**, click **Add classic branch protection rule**.
2. Under **Branch name pattern**, enter the branch or pattern (for example `main` or `release/*`).
3. Select **Require a pull request before merging**, then **Require approvals**, and choose the **Required number of approvals before merging**.
4. *(Optional)* Select any of these:

| Setting | What it does |
|---------|-------------|
| **Dismiss stale pull request approvals when new commits are pushed** | New commits reset approvals |
| **Require review from Code Owners** | Code owners must approve changes to their paths |
| **Require approval of the most recent reviewable push** | Someone other than the last pusher must approve |
| **Require status checks to pass before merging** | Search for and select the checks; optionally **Require branches to be up to date before merging** |
| **Require conversation resolution before merging** | All review threads must be resolved |
| **Require signed commits** | Commits must be signed |
| **Require linear history** | No merge commits |
| **Do not allow bypassing the above settings** | Apply the rules to administrators too |
| **Restrict who can push to matching branches** | Only listed people, teams, or apps can push |

5. Leave **Allow force pushes** and **Allow deletions** unchecked (the safe default).
6. Click **Create**.

> ✅ **Result:** branches matching the pattern are protected.

---

## 2️⃣ Repo-Level Rulesets (Modern, Recommended)

**👤 Role:** **Repository administrator** · **📍 Portal:** GitHub

**Navigate:** Repository → **Settings** → under "Code, planning, and automation", **Rulesets** → **Rulesets**

**Steps:**

1. Click **New ruleset**, then **New branch ruleset** (or **New tag ruleset**).
2. Under **Ruleset name**, type a name (for example `main-protection`).
3. Click **Disabled ▾** next to **Enforcement status** and choose **Active** (or **Evaluate** to test first).
4. *(Optional)* In **Bypass list**, click **Add bypass** — see Section 8.
5. Under **Target branches**, click **Add target** → **Include default branch** (or include/exclude by pattern).
6. Under **Branch rules**, select:

| Rule | Configuration |
|------|--------------|
| **Restrict deletions** | On by default — keep it |
| **Require a pull request before merging** | Set **Required approvals** (for example `1` or `2`); optionally **Require review from Code Owners** |
| **Require status checks to pass before merging** | Type each check name and click **+** to add it |
| **Block force pushes** | On by default — keep it |

7. Click **Create**.

> ✅ **Result:** with **Active** status the ruleset takes effect immediately.

---

## 3️⃣ Org-Level Rulesets (Enforce Across Repos)

**👤 Role:** **Organization owner** · **📍 Portal:** GitHub

**Navigate:** Organization → **Settings** → under "Code, planning, and automation", **Repository** → **Rulesets**

**Steps:**

1. Click **New ruleset** → **New branch ruleset**.
2. Enter a **Ruleset name** and set **Enforcement status** to **Active**.
3. Under **Target repositories**, next to **Repository targeting criteria**, choose:

| Option | Effect |
|--------|--------|
| **All repositories** | Every repository in the organization |
| **Repositories matching a name** | **Add a target** → **Include by pattern** / **Exclude by pattern** (`fnmatch`) |
| **Repositories matching a filter** | Query by custom or system properties, for example `props.team:infra visibility:private` |
| **Only selected repositories** | **Select repositories**, then pick each one |

4. Under **Target branches**, click **Add target** → **Include default branch** (or patterns).
5. Select the branch rules (pull request, approvals, status checks, and so on).
6. *(Optional)* Add a **Bypass list** for admin roles or automation.
7. Click **Create**.

> 📌 Only organization owners can edit an organization ruleset. Repository admins can still add their own repository rulesets on top.

---

## 4️⃣ Enterprise-Level Rulesets

**👤 Role:** **Enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **Policies** tab → **Code**

**Steps:**

1. At the top of the enterprise page, click **Policies**, then under **Policies**, click **Code**.
2. Click **New ruleset** → **New branch ruleset**.
3. Enter a **Ruleset name** and set **Enforcement status** to **Active**.
4. *(Optional)* Add a **Bypass list** (enterprise teams, roles, apps, repository admins, org owners, enterprise owners…).
5. **Target organizations:** all, a selected list, a name pattern (`fnmatch`), or organization custom properties. *(With EMU, you can also target repositories owned by users.)*
6. **Target repositories:** all, or a filter by custom property or deployment context.
7. **Target branches:** **Add target** → **Include default branch** (or patterns).
8. Select the branch rules and click **Create**.

> ⚠️ **Important:** organization and repository admins can't edit or remove an enterprise ruleset. Use it for non-negotiable policies.

---

## 5️⃣ Finding Which Rule Is Enforcing Approval Requirements

**👤 Role:** Anyone with **read** access (to view rules); repository admin, organization owner, or enterprise owner to change them · **📍 Portal:** GitHub

**Fastest way (anyone with read access):**

1. Open the repository's branch dropdown → **View all branches**.
2. Next to the branch, click the **shield-lock** icon to see every ruleset that targets it — including ones in **Evaluate** mode.

**Then check, in any order (all applicable rules combine):**

| Where to check | Click path |
|---------------|-----------|
| **Repository rulesets** | Repo → **Settings** → **Rulesets** → **Rulesets** |
| **Classic branch protection** | Repo → **Settings** → **Branches** |
| **Organization rulesets** | Org → **Settings** → **Repository** → **Rulesets** |
| **Enterprise rulesets** | Enterprise → **Policies** → **Code** |
| **What actually happened** | Repo → **Settings** → **Rulesets** → **Rule Insights** (passes, failures, bypasses) |

> 💡 **Tip:** On the pull request, the merge box lists the rules that are blocking the merge.

---

## 6️⃣ CODEOWNERS for Path-Specific Approvals

**👤 Role:** **Write** access to commit the file; **repository administrator** to require it

1. Create a `CODEOWNERS` file in `.github/`, the repository root, or `docs/`.
2. Add ownership rules:

```
# Global owners (fallback)
*                     @org/platform-team

# Frontend paths
/src/frontend/        @org/frontend-team

# Infrastructure
/terraform/           @org/infra-team

# Sensitive configs
/.github/workflows/   @org/devops-leads
```

3. Commit the file to the default branch.
4. In your ruleset (**Require a pull request before merging** → **Require review from Code Owners**) or classic rule (**Require review from Code Owners**), turn it on.

> ⚠️ **Prerequisite:** the branch needs a ruleset or classic rule with **Require a pull request before merging** for CODEOWNERS approval to be enforced. Teams listed as owners need write access to the repository.

---

## 7️⃣ Recommended Baseline Configuration

| Setting | Recommended value |
|---------|-------------------|
| **Method** | Organization ruleset (for consistency) |
| **Required approvals** | 1–2 depending on team size |
| **Dismiss stale approvals** | Enabled |
| **Require Code Owners review** | Enabled for critical paths |
| **Require status checks** | Enabled (CI must pass) |
| **Block force pushes** | Enabled |
| **Restrict deletions** | Enabled |

> 💡 **Tip:** Start with 1 approval for most repositories and 2 for critical or production repositories. Roll out new rulesets in **Evaluate** mode first and watch **Rule Insights**.

---

## 8️⃣ Bypass Lists for Admins & Automation (Rulesets Feature)

**👤 Role:** Whoever owns the ruleset — **repository administrator**, **organization owner**, or **enterprise owner** · **📍 Portal:** GitHub

**Navigate (inside any ruleset):** **Bypass list** → **Add bypass**

**Steps:**

1. In the **Bypass list** section, click **Add bypass**.
2. Search for the role, team, or app, select it under **Suggestions**, and click **Add Selected**.
3. *(Optional)* Next to **Always allow**, click **⋯** → **For pull requests only**.

**Who can be added:**

| Actor | Example use case |
|-------|-----------------|
| **Repository admin / organization owner / enterprise owner** | Emergency hotfixes |
| **Maintain or write role** (or custom roles based on write) | Release managers |
| **Teams** (not secret teams) | Release engineering team |
| **Enterprise teams, apps, and roles** *(public preview)* | Central platform team |
| **GitHub Apps** | CI/CD or deploy bots |
| **Dependabot** | Dependency updates |
| **Copilot cloud agent** | Agent pull requests |
| **Deploy keys** *(enterprise rulesets)* | Automated deploys |

**Bypass modes:**

| Mode | Description |
|------|-------------|
| **Always allow** | Can bypass without a pull request |
| **For pull requests only** | Must open a pull request, then can bypass the rules to merge it — leaving an audit trail |

> ⚠️ **Important:** Keep bypass lists short and review bypasses regularly in **Rule Insights** and the audit log.

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
| **Audit log search returns no events** | The date range, action qualifier, actor, or retention window excludes the event. | Widen the query, search by known action names, and use exported/streamed logs for events older than the UI retention window. |
| **Audit log stream is configured but SIEM receives no events** | Destination credentials, network allow lists, event hub/topic configuration, or stream status is wrong. | Check stream health in GitHub, rotate destination credentials if needed, allow GitHub source IPs, and pause/resume only within documented retention limits. |
| **Ruleset blocks a push or merge unexpectedly** | A branch/tag/push ruleset or legacy branch protection rule targets the ref. | Open the repository rules view for the affected branch/tag, identify the active rule, and either comply with the rule or request a bypass from the owner. |
| **Repository transfer or org rename leaves broken references** | Profile URLs, marketplace/action namespaces, webhooks, secrets, environments, and external integrations may not redirect or transfer. | Inventory dependent systems before the change, update remote URLs and integration settings after the change, and validate webhooks, Actions, Apps, and security configurations. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>

### Q: I created a branch protection rule or ruleset, but it does not seem to be enforcing. What should I check?
**A:** Check that the target matches the branch (`main` vs `master`; `release/*` doesn't match `release/a/b` because `*` stops at `/`). For rulesets, confirm **Enforcement status** is **Active** — new rulesets start as **Disabled**, and **Evaluate** only logs results. For organization rulesets, confirm the repository is in **Target repositories**. Use the branch's **shield-lock** icon to see which rulesets actually apply.

---

### Q: Our admin can bypass branch protection and push directly to the protected branch. How do we prevent this?
**A:** In a classic rule, administrators can bypass unless you select **Do not allow bypassing the above settings**. In rulesets, only actors on the **Bypass list** can bypass — leave admins off it, or add them **For pull requests only** so they must still open a pull request.

---

### Q: Should we use rulesets or classic branch protection rules? What is the difference?
**A:** Use rulesets for new work. They add enforcement statuses (including **Evaluate** for testing), explicit bypass lists, layering, organization- and enterprise-level targeting (by name, property, or selection), and visibility to anyone with read access. Classic rules still work and combine with rulesets — GitHub provides a way to convert branch protection rules to rulesets.

---

### Q: How do I require CODEOWNERS approval for changes to specific file paths?
**A:** Commit a `CODEOWNERS` file in `.github/`, the repository root, or `docs/`. Then turn on **Require review from Code Owners** under **Require a pull request before merging** in your ruleset (or in the classic rule). When a pull request changes owned files, an owner must approve — any one owner is enough if several are listed.

---

### Q: CI status checks are passing but not blocking the merge when they fail. What is wrong?
**A:** The required check name must exactly match the check name shown on the pull request's **Checks** tab (usually the job name). In a ruleset, make sure you clicked **+** after typing each name — otherwise it isn't saved. If **Require branches to be up to date before merging** is on, at least one check must be defined.

---

### Q: Can multiple rulesets apply to the same branch at the same time?
**A:** Yes. Rulesets have no priority — all rulesets targeting a branch are aggregated, along with any classic branch protection rule. If the same rule is set differently, the most restrictive version applies (for example, 1 vs 2 required approvals → 2). This works across repository, organization, and enterprise rulesets.

</details>

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Responsible AI Guardrails | `Copilot/Responsible AI Guardrails.md` |
| Enterprise Environment Scaffolding Checklist | `Setup/Enterprise Environment Scaffolding Checklist.md` |
| Code Scanning (CodeQL) Enablement & Troubleshooting | `Security/Code Scanning (CodeQL) Enablement & Troubleshooting.md` |
| Copilot Admin Controls (Models, Content Exclusion, Instructions) | `Copilot/Admin Controls (Models, Content Exclusion, Instructions).md` |

---

## 📝 Resources

| Resource | Link |
|----------|------|
| **About rulesets** | [GitHub Docs](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets) |
| **Creating rulesets for a repository** | [GitHub Docs](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/creating-rulesets-for-a-repository) |
| **Org-level rulesets** | [GitHub Docs](https://docs.github.com/en/organizations/managing-organization-settings/creating-rulesets-for-repositories-in-your-organization) |
| **Enterprise rulesets (code governance)** | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/admin/enforcing-policies/enforcing-policies-for-your-enterprise/enforcing-policies-for-code-governance) |
| **Available rules for rulesets** | [GitHub Docs](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets) |
| **Managing a branch protection rule** | [GitHub Docs](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/managing-a-branch-protection-rule) |

---

*Last updated: October 2026*
