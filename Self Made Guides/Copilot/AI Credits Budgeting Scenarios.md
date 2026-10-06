# 💰 GitHub Copilot AI Credits Budgeting Scenarios Guide

> **Worked, math-checked scenarios for controlling Copilot spend under usage-based billing (AI credits)**

> 📌 **Billing changed on June 1, 2026.** Premium requests and model multipliers were replaced by **AI credits** (1 credit = $0.01), charged by model and tokens used. These scenarios use the current model. See [AI Credits Budget & Overage Planning](AI%20Credits%20Budget%20%26%20Overage%20Planning.md) for the fundamentals.

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [📋 Overview](#-overview)
- [🚨 What Actually Controls Spending (Don't Skip)](#-what-actually-controls-spending-dont-skip)
- [📊 Scenario 1: Copilot Business vs Copilot Enterprise](#-scenario-1-copilot-business-vs-copilot-enterprise)
- [🎛️ Scenario 2: Higher Limits for Specific Users](#️-scenario-2-higher-limits-for-specific-users)
- [🛡️ Scenario 3: Stop One Team From Draining the Shared Pool](#️-scenario-3-stop-one-team-from-draining-the-shared-pool)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the decisions:**

- **Business vs Enterprise:** Enterprise is never cheaper for the same usage — choose it for its features, not to save on credits (Scenario 1).
- **Higher limits for some users:** use an **individual user-level budget** (optionally expiring) or a **cost center user-level budget** — both override the universal budget (Scenario 2).
- **Stop one team draining the pool:** put the team in a cost center and turn on its **included usage control** (Scenario 3).
- **All budgets:** Enterprise → **Billing and licensing** → **Budgets and alerts** → **New budget** → **Bundled AI credits budget**.

---

## ✅ Accuracy & Click-Path Notes

<details>
<summary><em>Show click-path conventions</em></summary>


- Reviewed against current public GitHub documentation in October 2026, including the June 1, 2026 move to usage-based billing. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
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
| Create budgets and cost centers | GitHub **enterprise owner** or **billing manager** | ☐ |
| Usage reviewed on Enterprise → **Billing and licensing** → **AI usage** | GitHub **enterprise owner** or **billing manager** | ☐ |
| Existing budgets reviewed on **Budgets and alerts** (any **$0** budget blocks usage immediately) | GitHub **enterprise owner** or **billing manager** | ☐ |

---

## 📋 Overview

| Scenario | Question it answers |
|----------|---------------------|
| **Scenario 1** | Should heavy users move to Copilot Enterprise to save money? |
| **Scenario 2** | How do we give some users higher limits while keeping everyone else restricted? |
| **Scenario 3** | How do we keep one heavy team from using up the shared pool for everyone else? |

---

## 🚨 What Actually Controls Spending (Don't Skip)

### 1) The AI credits paid usage policy

Enterprise → **AI controls** → **Copilot** → **AI credits paid usage**. It's **enabled by default**. When enabled, usage continues past the shared pool at $0.01 per credit; when disabled, usage stops once the pool is exhausted.

### 2) User-level budgets — always active, always a hard stop

They cap each user's total consumption (pool **and** metered). When more than one applies, **the most specific wins**:

1. **Individual** user-level budget (one named user) — wins over everything below
2. **Cost center** user-level budget (one per-person amount for every member of a cost center)
3. **Universal** user-level budget (every licensed user)

### 3) Spending limits — metered phase only

**Cost center budgets**, **organization budgets**, and the **enterprise spending limit** only apply after the shared pool is exhausted. They're hard stops **only** if **Stop usage when budget limit is reached** is selected — otherwise they just send alerts. A cost center budget does **not** extend or override a user's user-level budget.

### 4) $0 budgets

Any budget set to **$0** stops usage immediately for everyone it applies to. Check **Budgets and alerts** before troubleshooting anything else.

---

## 📊 Scenario 1: Copilot Business vs Copilot Enterprise

**Situation:** Some users consume a lot of AI credits. Under the old premium-request model, moving heavy users to Copilot Enterprise often saved money. Does it still?

### Verified plan entitlements & pricing

| Plan | Seat price | Included AI credits per user | Value of included credits | Additional usage |
|------|-----------|------------------------------|---------------------------|------------------|
| **Copilot Business** | $19/month | 1,900 | $19 | $0.01 per credit |
| **Copilot Enterprise** | $39/month | 3,900 | $39 | $0.01 per credit |

### Cost comparison per user (math-checked)

Assumes the user's usage isn't offset by other users' unused credits in the shared pool.

| Credits used per month | Copilot Business total | Copilot Enterprise total | Enterprise minus Business |
|------------------------|------------------------|--------------------------|---------------------------|
| 1,000 | $19 | $39 | **+$20** |
| 1,900 | $19 | $39 | **+$20** |
| 3,000 | $19 + $11 = **$30** | $39 | **+$9** |
| 3,900 | $19 + $20 = **$39** | $39 | **$0** |
| 6,000 | $19 + $41 = **$60** | $39 + $21 = **$60** | **$0** |

> 💡 **Key rule:** Under usage-based billing, **Copilot Enterprise is never cheaper than Copilot Business for the same usage.** Each dollar of seat price buys exactly one dollar of credits, and extra usage costs the same $0.01 per credit on either plan. At best (3,900+ credits per user) the totals are equal. Choose Copilot Enterprise for its **additional features**, not to save on usage.

> 📌 **Pooling changes the math in your favor on Business, too:** light users' unused credits cover heavy users, so a team's metered spend depends on the team's total, not each person's.

### If you move users to Copilot Enterprise for its features

**👤 Role:** GitHub **enterprise owner**, then **organization owner** · **📍 Portal:** GitHub

1. Confirm your enterprise has a **Copilot Enterprise** plan.
2. Create (or choose) an organization for those users and add them to it.
3. Go to Enterprise → **Billing and licensing** → **Licensing** → **Manage** *(in the "Copilot" section)* → **Organizations** tab → open that organization's **Copilot** dropdown → click **Copilot: Enterprise**. *(Applies immediately.)*
4. In that organization: **Settings** → **Copilot** → **Access** → **Start adding seats** → **Purchase for selected members** → add the users → **Continue to purchase** → **Purchase seats**.
5. Re-check usage monthly on Enterprise → **Billing and licensing** → **AI usage**.

> 📌 Direct enterprise-level license assignment (without an organization) is available for **Copilot Business** only.

---

## 🎛️ Scenario 2: Higher Limits for Specific Users

**Situation:** Everyone should have a sensible per-person cap, but a handful of power users — or a whole team — need more.

### Choose an approach

| Approach | Best for | Scope setting on the budget |
|----------|----------|-----------------------------|
| **Individual user-level budget** | A few named people, or a temporary boost (with an expiration) | **Users** → select the user |
| **Cost center user-level budget** | A whole team gets a higher per-person cap | **Users** → select the cost center |
| **Cost center budget** (spending limit) | Cap a team's total metered charges after the pool runs out | **Cost center** → select the cost center |

### Step 1 — Set the universal user-level budget (everyone)

**👤 Role:** GitHub **enterprise owner** or **billing manager** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **Billing and licensing** → **Budgets and alerts**

1. Click **New budget**.
2. Under **Budget Type**, select **Bundled AI credits budget**.
3. Under **Budget scope**, select **Users** and leave the user field **empty** (this makes it universal).
4. Under **Budget**, enter an amount **above** the license value — for example **$25** (2,500 credits) for Copilot Business users.
5. Under **Alerts**, select **Receive budget threshold alerts** and choose the **Alert Recipients**.
6. Click **Create budget**.

> ✅ Once a universal user-level budget exists, **every** licensed user is covered — there's no "uncovered user" gap.

### Step 2a — Raise the limit for named users (individual budget)

1. Click **New budget** → **Bundled AI credits budget** → **Budget scope**: **Users** → select the user.
2. Under **Expiration**, choose **No expiration**, **End of current billing cycle**, or a **Specific date** (useful for a one-sprint boost).
3. Enter the amount (for example **$80**), select **Receive budget threshold alerts**, and click **Create budget**.

### Step 2b — Raise the limit for a whole team (cost center user-level budget)

1. Create the cost center: **Billing and licensing** → **Cost centers** → **New cost center** → enter a **Name** → under **Resources** add the users, organizations, or enterprise teams → **Create cost center**.
2. Back on **Budgets and alerts**, click **New budget** → **Bundled AI credits budget** → **Budget scope**: **Users** → select the **cost center**.
3. Enter the per-person amount (for example **$60**), select **Receive budget threshold alerts**, and click **Create budget**.

### Step 3 — (Optional) Cap the team's metered spend

1. Click **New budget** → **Bundled AI credits budget** → **Budget scope**: **Cost center** → select the cost center.
2. Enter the monthly limit, select **Stop usage when budget limit is reached**, select **Receive budget threshold alerts**, and click **Create budget**.

| Example result | User-level cap that applies |
|----------------|-----------------------------|
| A typical developer | $25 (universal) |
| A member of the `ml-platform` cost center | $60 (cost center user-level budget) |
| Priya, with an individual budget | $80 (individual — wins over the other two) |

> 📝 **API note:** You can manage budgets with the REST API, including an `expires_at` field for individual budgets. See [Budgets REST API](https://docs.github.com/en/rest/billing/budgets).

---

## 🛡️ Scenario 3: Stop One Team From Draining the Shared Pool

**Situation:** One team runs long Copilot cloud agent sessions and uses a large share of the shared pool, so everyone else starts hitting metered charges early in the month.

**Solution:** put the team in a cost center and turn on that cost center's **included usage control**. GitHub then caps the team's draw from the pool at the credits funded by **its own licenses**.

### How the cap is calculated (math-checked)

| Licenses in the cost center | Credits each | Subtotal |
|-----------------------------|-------------:|---------:|
| 10 × Copilot Business | 1,900 | 19,000 |
| 5 × Copilot Enterprise | 3,900 | 19,500 |
| **Cost center cap** | | **38,500** |

- Adding a licensed user (or upgrading Business → Enterprise) raises the cap **right away**; removing or downgrading lowers it at the **start of the next billing cycle**.
- When the cap is reached, you choose whether the team's members are **blocked** or **roll into paid overage**.

### Steps

**👤 Role:** GitHub **enterprise owner** or **billing manager** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **Billing and licensing** → **Cost centers**

1. Create the team's cost center if it doesn't exist (**New cost center** → **Name** → **Resources** → **Create cost center**).
2. Next to the cost center, click **…** → **Edit**, and turn on its included usage control, choosing whether members are blocked or roll into paid overage at the cap. *(The exact label may vary — see GitHub's [cost centers documentation](https://docs.github.com/en/billing/concepts/cost-centers).)*
3. Open the cost center and confirm its home page shows **AI credit pool enabled** with the credits consumed so far.
4. Optionally add a cost center budget (Scenario 2, Step 3) to cap the team's metered charges too.

> ⚠️ **Not retroactive:** turning on included usage controls doesn't redistribute credits already used this cycle. From then on, the cost center's members share only the credits funded by licenses attributed to that cost center.

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
| **A power user is still capped at the universal amount** | Their individual budget expired, was created at the wrong scope, or they're not in the cost center you expected. | Check **Budgets and alerts** for an active individual budget for that user, and the cost center's **Resources** list. |
| **A whole team is blocked even though the pool has credits** | Their cost center user-level budget, or a cost center included usage cap, has been reached. | Raise the cost center user-level budget, or choose to roll members into paid overage at the included usage cap. |
| **Content exclusions do not apply immediately** | Client policy cache, unsupported surface/mode, symlink/remote filesystem limitation, or indirect IDE context. | Reload the IDE policy, verify the exclusion syntax at enterprise/org/repo scope, and document surfaces where exclusions are limited. |
| **Cloud agent or MCP action is denied** | Agent policy, MCP policy, repository permissions, secrets, or server allowlist does not permit the operation. | Review Enterprise AI controls > Agents/MCP, repo-level permissions, MCP server configuration, and audit logs for the denied action. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>


### Q: We moved heavy users to Copilot Enterprise but costs went up — what happened?
**A:** Under usage-based billing, Copilot Enterprise can't lower usage costs: its extra included credits cost exactly what you pay for them ($20 more per seat buys 2,000 more credits), and extra usage is billed at the same $0.01 per credit on both plans. If those users weren't already using more than 3,900 credits a month, the higher seat price is pure added cost. Keep Copilot Enterprise only where you need its features.

---

### Q: Do we still need a "restrictive budget for everyone else"?
**A:** Not in the old sense. A **universal user-level budget** automatically covers every licensed user, so nobody is uncovered. Give specific people or teams more with individual or cost center user-level budgets, which override the universal one.

---

### Q: Budget interaction is confusing — which budget applies?
**A:** User-level budgets follow a precedence order: individual, then cost center, then universal — the most specific one applies, and it's always a hard stop. Spending limits (cost center, organization, enterprise) are different: they only apply once the shared pool is exhausted, they apply side by side, and any one of them with **Stop usage when budget limit is reached** that's used up blocks the users in its scope.

---

### Q: How do we find our heaviest users?
**A:** Go to Enterprise → **Billing and licensing** → **AI usage**, group by user, and export the data. Look for users consistently near their user-level budget, and check which models they use — switching everyday work to a lighter model often helps more than a bigger budget.

---

### Q: Can we use cost center budgets and organization budgets together?
**A:** Yes. Both are spending limits that apply after the pool is exhausted, side by side. If either one with **Stop usage** is used up, the users it covers are blocked. Organization owners can only use organization budgets, and those can only restrict further — they can't override an enterprise budget.

</details>

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| AI Credits Budget & Overage Planning | `Copilot/AI Credits Budget & Overage Planning.md` |
| Overage Budgets & Cost Monitoring (GHEC Enterprise) | `Copilot/Overage Budgets & Cost Monitoring (GHEC Enterprise).md` |
| Power User AI Credit Allowance (Cost Centers + Budgets) | `Copilot/Power User AI Credit Allowance (Cost Centers + Budgets).md` |
| Seat Assignment & Enablement | `Copilot/Seat Assignment & Enablement.md` |

---

## 📝 Resources

| Resource | Link |
|----------|------|
| Usage-based billing for organizations and enterprises | [GitHub Docs](https://docs.github.com/en/copilot/concepts/billing-and-usage/organizations-and-enterprises/billing) |
| Copilot budget controls | [GitHub Docs](https://docs.github.com/en/copilot/concepts/billing-and-usage/organizations-and-enterprises/budgets) |
| Setting up budgets | [GitHub Docs](https://docs.github.com/en/billing/how-tos/set-up-budgets) |
| Using cost centers | [GitHub Docs](https://docs.github.com/en/billing/how-tos/products/use-cost-centers) |
| Plans for GitHub Copilot | [GitHub Docs](https://docs.github.com/en/copilot/get-started/plans) |

---

*Last updated: October 2026*
