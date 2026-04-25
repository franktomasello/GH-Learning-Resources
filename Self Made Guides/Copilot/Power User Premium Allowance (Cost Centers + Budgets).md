# 🎯 GitHub Copilot Power User Premium Allowance Runbook

> **Use the "Premium request paid usage" policy + cost-center budgets to allow overages for only a subset of users, while blocking everyone else.**

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [📋 Overview](#-overview)
- [✅ Prerequisites](#-prerequisites)
- [🏗️ Recommended Design (Most Operable)](#-recommended-design-most-operable)
- [1️⃣ Step 1 — Enable "Premium request paid usage" (Enterprise Policy)](#1-step-1--enable-premium-request-paid-usage-enterprise-policy)
- [2️⃣ Step 2 — Remove or Fix Any Budgets That Are Blocking Premium Request Overages](#2-step-2--remove-or-fix-any-budgets-that-are-blocking-premium-request-overages)
- [3️⃣ Step 3 — Create Two Cost Centers and Assign Users](#3-step-3--create-two-cost-centers-and-assign-users)
- [4️⃣ Step 4 — Create Cost-Center-Scoped Budgets (The Enforcement)](#4-step-4--create-cost-center-scoped-budgets-the-enforcement)
- [5️⃣ Step 5 — Verify It's Working (No Conflicts + Correct Attribution)](#5-step-5--verify-its-working-no-conflicts--correct-attribution)
- [💡 Optional Pattern: Use a Second Org to Increase Included Allowance (Power Users)](#-optional-pattern-use-a-second-org-to-increase-included-allowance-power-users)
- [📝 Notes on Legacy Auto-Created $0 Copilot Premium Request Budgets (Verified)](#-notes-on-legacy-auto-created-0-copilot-premium-request-budgets-verified)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- Enable paid overages: `Enterprise → AI controls → Copilot → Premium request paid usage → Enabled`
- Review/delete legacy budgets: `Enterprise → Billing and licensing → Budgets and alerts`
- Create cost centers: `Enterprise → Billing and licensing → Cost centers → New cost center`
- Create $0 budget for default users: `Budgets and alerts → New budget → Cost center: Default users → $0 → Stop usage: ON`
- Create allowance budget for power users: `Budgets and alerts → New budget → Cost center: Hyper power users → $X`

---

## ✅ Accuracy & Click-Path Notes

<details>
<summary><em>Show click-path conventions</em></summary>


- Reviewed against current public GitHub and Microsoft documentation in April 2026 where public documentation is available. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
- When a path starts with `Enterprise`, begin at GitHub, click your profile photo, click `Your enterprises` or `Enterprise`, select the enterprise, then continue with the listed top tab or left-sidebar item.
- When a path starts with `Organization` or `Org`, begin at GitHub, click your profile photo, click `Your organizations`, select the organization, click `Settings`, then continue with the listed sidebar item.
- When a path starts with `Repository`, `Repo`, or a repository name, open the repository, click the `Settings` tab, then continue with the listed sidebar item.
- When a path starts with a vendor portal such as `Microsoft Entra admin center`, `Azure portal`, `Okta Admin Console`, `PingFederate`, `PingOne`, `OneLogin`, `AD FS Management`, `Visual Studio Admin Portal`, or `Azure DevOps`, sign in to that admin portal first, select the tenant, application, or project named in the step, then follow each listed blade, tab, button, and confirmation in order.
- If the expected button is missing, verify you are signed in with the role named in Prerequisites, the feature or license is enabled, and the object is owned by the selected enterprise, organization, or repository. Use page search only to locate the same page, not to skip required confirmation, test, save, or consent clicks.

</details>

---

## 📋 Overview

Allow only a specific subset of users (“hyper power users”) to use paid premium requests beyond their included allowance, while everyone else is blocked from overages using budgets that stop usage at the limit.

Key mechanics to keep in mind:
- Budgets don't override each other. If any applicable budget with "Stop usage when budget limit is reached" is exhausted, premium requests are blocked, even if another budget "should allow" spend [[1]](#source-1).
- Every user must be covered by a budget or they may have unlimited premium-request overage spend [[1]](#source-1).

---

## ✅ Prerequisites

Before starting, confirm these items are in place:

| Requirement | Status |
|-------------|--------|
| GitHub Enterprise Cloud enterprise with Copilot Business or Copilot Enterprise active | ☐ |
| Enterprise owner or billing admin access for AI controls, budgets, and cost centers | ☐ |
| Premium request paid usage policy decision approved by platform/finance owners | ☐ |
| List of hyper power users identified by GitHub username or managed user account | ☐ |
| Cost center model agreed on for default users and hyper power users | ☐ |
| Budget approach selected: bundled premium requests budget or SKU-level budgets | ☐ |

> 💡 **Tip:** Premium requests are metered, so budgets can optionally stop usage at the limit [[1]](#source-1). Use bundled premium request budgets for most organizations, and SKU-level budgets only when you need separate controls for Copilot, Spark, or coding agent spend [[2]](#source-2).

---

## 🏗️ Recommended Design (Most Operable)

Create two cost centers and two cost-center-scoped budgets:

| Cost Center | Who’s in it | Budget | “Stop usage…” |
|-------------|-------------|--------|---------------|
| Default users | All organizations (safety net) | $0 | ON (hard block) |
| Hyper power users | Only hyper users (direct assignment) | $X | ON (hard cap) or OFF (alert-only) |

This matches GitHub's recommended pattern for "budget for specific members" + "separate restrictive budget for everyone else" [[2]](#source-2).

---

## 1️⃣ Step 1 — Enable "Premium request paid usage" (Enterprise Policy)

If this policy is disabled, users cannot surpass included allowance, regardless of budgets [[1]](#source-1).

### Exact click-by-click navigation (Enterprise)
1. In the top-right of GitHub, click your profile picture.
2. Click Enterprise
   - Or click Enterprises, then click your enterprise name.
3. At the top of the enterprise page, click AI controls.
4. In the left sidebar, click Copilot.
5. Find Premium request paid usage.
6. Set it to Enabled [[1]](#source-1).

---

## 2️⃣ Step 2 — Remove or Fix Any Budgets That Are Blocking Premium Request Overages

This is the most common "why is it still blocked?" issue: an existing budget with "Stop usage…" enabled is still applying [[1]](#source-1).

### Exact click-by-click navigation (Enterprise budgets list)
1. Top-right → profile picture
2. Enterprise (or Enterprises → select enterprise)
3. At the top of the page, click Billing and licensing.
4. In the billing left sidebar, click Budgets and alerts.

### What to do on "Budgets and alerts"
- Look for any budget that applies to:
  - Bundled premium requests, and/or
  - SKU-level premium request budgets (e.g., Copilot premium requests / Spark premium requests / Copilot coding agent premium requests) [[2]](#source-2)
- For any budget that is $0 and has “Stop usage when budget limit is reached” = ON (or any other exhausted stop-usage budget that applies to your users):
  1. Click the ⋯ (kebab) menu next to the budget
  2. Click Edit (or Delete)
  3. If keeping it, either:
     - Increase the budget above $0, or
     - Turn Stop usage when budget limit is reached OFF (monitor-only)

### Important accuracy notes (verified)
- Creating a new budget does not override old ones; overlapping budgets can block usage unexpectedly [[1]](#source-1).
- You cannot change the scope of a budget after it's created (you must create a new one) [[3]](#source-3).

---

## 3️⃣ Step 3 — Create Two Cost Centers and Assign Users

### Exact click-by-click navigation (Create cost centers)
1. Top-right → profile picture
2. Enterprise (or Enterprises → select enterprise)
3. Top tab → Billing and licensing
4. Left sidebar → Cost centers
5. Click New cost center (upper-right) [[4]](#source-4)

### Create cost center: "Default users"
- Name: Default users
- Under Resources, select your **Organizations** (e.g., `acme-corp`, `acme-engineering`)
- Save
- *Crucial: This acts as a safety net for all current and future members* [[5]](#source-5)

### Create cost center: "Hyper power users"
- Name: Hyper power users
- Under Resources, add only the hyper users (search by individual username)
- Save
- *Note: Direct user assignment automatically moves them out of the Default cost center* [[6]](#source-6)

---

## 4️⃣ Step 4 — Create Cost-Center-Scoped Budgets (The Enforcement)

### Exact click-by-click navigation (Create a budget)
1. Top-right → profile picture
2. Enterprise (or Enterprises → select enterprise)
3. Top tab → Billing and licensing
4. Left sidebar → Budgets and alerts
5. Click New budget [[3]](#source-3)

### 4A) Budget for "Default users" (Block Overages)

On the New budget flow:
- Budget Type
  - Choose Bundled premium requests budget (recommended for most) [[2]](#source-2)
- Budget scope
  - Choose Cost center
  - Select Default users [[2]](#source-2)
- Budget amount: $0
- Enable: Stop usage when budget limit is reached ✅ (hard block)
- Click New budget [[2]](#source-2)

### 4B) Budget for "Hyper power users" (Allow Overages)

Repeat the same flow, but set:
- Budget scope: Cost center → Hyper power users
- Budget amount: $X
- Stop usage when budget limit is reached:
  - ON = hard cap
  - OFF = alert-only (spend can exceed $X) [[2]](#source-2)

---

## 5️⃣ Step 5 — Verify It's Working (No Conflicts + Correct Attribution)

### A) Verify no conflicting budgets

#### Navigation
1. Top-right → profile picture
2. Enterprise → Billing and licensing → Budgets and alerts

#### Confirm
- No leftover Enterprise-level premium-request budgets with Stop usage enabled that unintentionally apply to the hyper users [[3]](#source-3).

### B) Verify usage lands in the right cost center

#### GitHub-supported ways
- On the Usage page, group/filter by cost center [[7]](#source-7)
- Download a detailed usage report and verify columns like cost_center_name [[7]](#source-7)

---

## 💡 Optional Pattern: Use a Second Org to Increase Included Allowance (Power Users)

If you also need a middle tier ("power users" who just need a bigger included allowance), GitHub documents upgrading selected users to Copilot Enterprise by placing them in a separate org and granting Copilot Enterprise licenses there [[8]](#source-8).

---

## 📝 Notes on Legacy Auto-Created $0 Copilot Premium Request Budgets (Verified)
- Accounts created before Aug 22, 2025 may have had a default $0 Copilot premium request budget that rejects overages unless edited/deleted [[1]](#source-1).
- GitHub announced that starting Dec 2, 2025, those $0 budgets for enterprise/team accounts would be removed, and paid usage would be governed by the Premium request paid usage policy instead [[9]](#source-9).

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
| **Premium requests are rejected after the included allowance** | Paid usage is disabled, no billing entity is selected, or a stop-usage budget is exhausted. | Enable Premium request paid usage where appropriate, set or delete conflicting budgets, and have users with multiple licenses choose a billing entity. |
| **Content exclusions do not apply immediately** | Client policy cache, unsupported surface/mode, symlink/remote filesystem limitation, or indirect IDE context. | Reload the IDE policy, verify the exclusion syntax at enterprise/org/repo scope, and document surfaces where exclusions are limited. |
| **Usage metrics look empty or inconsistent** | Telemetry is disabled, data freshness delay applies, users are unlicensed, or different APIs report different scopes. | Enable the metrics policy, confirm seats and telemetry, wait for data freshness, and avoid comparing dashboards/API endpoints as if they share identical data models. |
| **Coding agent or MCP action is denied** | Agent policy, MCP policy, repository permissions, secrets, or server allowlist does not permit the operation. | Review Enterprise AI controls > Agents/MCP, repo-level permissions, MCP server configuration, and audit logs for the denied action. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>


### Q: I assigned users to the "Hyper power users" cost center but they are still blocked from overages — why?
**A:** Check for conflicting budgets. If an enterprise-wide or org-level budget with "Stop usage when budget limit is reached" is exhausted, it will block those users even though they have a separate cost center budget. Review all budgets on the Budgets and alerts page and remove or edit any that conflict.

---

### Q: A new user joined the org but is not covered by any cost center — what happens?
**A:** If a user is not assigned to any cost center and there is no enterprise-wide budget covering them, they may have unlimited premium request overage spend. Always ensure a "Default users" cost center with a $0 hard-block budget covers all organizations as a safety net.

---

### Q: I moved a user from "Default users" to "Hyper power users" but the change is not reflected — how long does it take?
**A:** Direct user assignment to a cost center automatically moves them out of any org-based cost center assignment. The change should take effect within minutes. If it does not, verify the assignment on the Cost centers page and check that you are using the correct GitHub username.

---

### Q: Can I use the API to manage cost center assignments for hyper power users?
**A:** Yes, adding users to cost centers is supported via the enterprise billing REST API. However, note that the endpoint does not work with fine-grained PATs (and some GitHub App token types). Use a classic PAT with appropriate enterprise billing permissions.

---

### Q: How do I verify that usage is landing in the correct cost center?
**A:** Navigate to Enterprise > Billing and licensing > Usage > Premium request analytics and group or filter by cost center. You can also download a detailed usage report (CSV) and check the `cost_center_name` column to confirm attribution.

</details>

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Premium Request Budget & Overage Planning | `Copilot/Premium Request Budget & Overage Planning.md` |
| Overage Budgets & Cost Monitoring (GHEC Enterprise) | `Copilot/Overage Budgets & Cost Monitoring (GHEC Enterprise).md` |
| Cost Centers & Department Billing | `Billing/Cost Centers & Department Billing.md` |

---

## 📝 Resources

| # | Title | Link |
|:-:|-------|------|
| <a id="source-1"></a>1 | Managing the premium request allowance for your enterprise | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/manage-and-track-spending/manage-request-allowances) |
| <a id="source-2"></a>2 | Controlling and tracking costs at scale | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/billing/tutorials/control-costs-at-scale) |
| <a id="source-3"></a>3 | Setting up budgets to control spending on metered products | [GitHub Docs](https://docs.github.com/enterprise-cloud@latest/billing/tutorials/set-up-budgets) |
| <a id="source-4"></a>4 | Using cost centers to allocate costs to business units | [GitHub Docs](https://docs.github.com/enterprise-cloud@latest/billing/tutorials/use-cost-centers) |
| <a id="source-5"></a>5 | Cost center allocation for different products | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/billing/reference/cost-center-allocation) |
| <a id="source-6"></a>6 | Customers can now add users to a cost center from the UI and API | [GitHub Blog](https://github.blog/changelog/2025-08-18-customers-can-now-add-users-to-a-cost-center-from-both-the-ui-and-api-2/) |
| <a id="source-7"></a>7 | Billing reports reference | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/billing/reference/billing-reports) |
| <a id="source-8"></a>8 | Choosing your enterprise's plan for GitHub Copilot | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/copilot/get-started/choose-enterprise-plan) |
| <a id="source-9"></a>9 | Upcoming removal of Copilot premium request $0 budgets | [GitHub Blog](https://github.blog/changelog/2025-09-17-upcoming-removal-of-copilot-premium-request-0-budgets-for-enterprise-and-team-accounts/) |

---

*Last updated: April 2026*
