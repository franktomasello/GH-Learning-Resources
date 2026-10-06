# 💰 GitHub Copilot AI Credits Budget & Overage Planning Runbook

> **Plan, budget, and control GitHub Copilot spending under usage-based billing (AI credits)**

> 📌 **Billing changed on June 1, 2026.** GitHub replaced premium requests and model multipliers with **usage-based billing**: Copilot usage is now measured in **AI credits** based on the model and the tokens consumed. This guide covers the current model. Only existing *annual* Copilot Pro and Pro+ plans stay on premium requests until they expire.

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [📋 Overview](#-overview)
- [1️⃣ What Uses AI Credits (and What Doesn't)](#1️⃣-what-uses-ai-credits-and-what-doesnt)
- [2️⃣ Included Credits and the Shared Pool](#2️⃣-included-credits-and-the-shared-pool)
- [3️⃣ Model Pricing](#3️⃣-model-pricing)
- [4️⃣ Budget Planning Formula](#4️⃣-budget-planning-formula)
- [5️⃣ Enterprise Controls](#5️⃣-enterprise-controls)
- [6️⃣ Best-Practice Rollout](#6️⃣-best-practice-rollout)
- [7️⃣ Monitoring Consumption](#7️⃣-monitoring-consumption)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📚 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Allow or block spending past the included credits:** Enterprise → **AI controls** → **Copilot** → **AI credits paid usage** *(enabled by default)*
- **Cap every user (universal budget):** Enterprise → **Billing and licensing** → **Budgets and alerts** → **New budget** → **Bundled AI credits budget** → scope **Users** (leave the user field empty) → amount → **Create budget**
- **Cap metered spend (enterprise spending limit):** same page → **New budget** → **Bundled AI credits budget** → scope **Enterprise** → amount → check **Stop usage when budget limit is reached** → **Create budget**
- **Group spend by team:** Enterprise → **Billing and licensing** → **Cost centers** → **New cost center**
- **Control which models are available:** Enterprise → **AI controls** → **Copilot** → **Configure models**
- **Monitor consumption:** Enterprise → **Billing and licensing** → **AI usage**

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
| Set Copilot policies (AI credits paid usage, models) | GitHub **enterprise owner** | ☐ |
| Create budgets and cost centers | GitHub **enterprise owner** or **billing manager** | ☐ |
| Current per-token model prices for your estimates | — ([models and pricing](https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing)) | ☐ |

---

## 📋 Overview

Every Copilot Business and Copilot Enterprise license includes a monthly amount of **AI credits** (1 AI credit = **$0.01 USD**). Each interaction costs credits based on **which model** handled it and **how many tokens** it used — input tokens (what's sent to the model), output tokens (what the model generates), and cached tokens (context it reuses). A quick chat question on a lightweight model can cost a fraction of a credit; a long agent session on a frontier model costs much more.

Included credits are **pooled** across your billing entity. When the pool runs out, usage either continues at per-credit rates (the default) or stops, depending on your policy and budgets.

| Plan | Price | Included AI credits per user per month | Value of included credits |
|------|-------|----------------------------------------|---------------------------|
| **Copilot Business** | $19 per seat | 1,900 | $19 |
| **Copilot Enterprise** | $39 per seat | 3,900 | $39 |

---

## 1️⃣ What Uses AI Credits (and What Doesn't)

| Uses AI credits | Doesn't use AI credits |
|-----------------|------------------------|
| Copilot Chat | Code completions (inline suggestions) — **unlimited on paid plans** |
| Copilot CLI | Next edit suggestions — **unlimited on paid plans** |
| Copilot cloud agent (formerly Copilot coding agent) | |
| Copilot Spaces | |
| Spark | |
| Third-party coding agents | |

> 💡 **Tip:** Copilot code review also consumes **GitHub Actions minutes** (since June 1, 2026), so include Actions in your cost planning if you use it heavily.

---

## 2️⃣ Included Credits and the Shared Pool

- **Pooled, not per user.** Included credits are shared at the billing-entity level. Example: an enterprise with 100 Copilot Business users has one shared pool of **190,000** credits, not 100 separate buckets of 1,900.
- **Adding licenses mid-cycle** increases the pool immediately. **Removing licenses** doesn't shrink it until the next billing cycle.
- **No rollover.** Unused credits are forfeited. The pool resets to its full monthly amount at **00:00:00 UTC on the first day of each calendar month**, regardless of when licenses were added.
- **After the pool is exhausted:**
  - **Additional usage allowed** (the default): usage continues at published per-credit rates and is charged to your organization or enterprise. Additional usage may be capped.
  - **Additional usage not allowed:** usage is blocked until the pool refreshes next month.

> ⚠️ **Additional usage is enabled by default.** To prevent any spending beyond the included credits, an administrator must explicitly disable the **AI credits paid usage** policy (step 5A).

---

## 3️⃣ Model Pricing

Prices are **per 1 million tokens** and are charged in AI credits at $0.01 per credit. Examples from GitHub's published table (October 2026):

| Model | Input | Cached input | Output |
|-------|------:|-------------:|-------:|
| **GPT-5 mini** | $0.25 | $0.025 | $2.00 |
| **Claude Haiku 4.5** | $1.00 | $0.10 | $5.00 |
| **Claude Sonnet 5.5** | $2.00 | $0.20 | $10.00 |
| **GPT-5.4** (≤ 272K input tokens) | $2.50 | $0.25 | $15.00 |
| **Claude Sonnet 4.6** | $3.00 | $0.30 | $15.00 |
| **Claude Opus 5.5** | $4.00 | $0.20 | $20.00 |

> 📌 **Prices change as models are added and retired.** Always pull current rates from [Models and pricing](https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing) before finalizing an estimate. Some models also have a cache-write price or a higher rate above an input-token threshold.

---

## 4️⃣ Budget Planning Formula

*Use these formulas to estimate monthly consumption and metered (additional) spend.*

### Step 1: Calculate the shared pool

```
Pool (credits) = (Business licenses × 1,900) + (Enterprise licenses × 3,900)
```

### Step 2: Estimate credits per interaction

```
Cost ($)    = (input tokens × input price + cached tokens × cached price + output tokens × output price) ÷ 1,000,000
Credits     = Cost ÷ $0.01
```

### Step 3: Calculate metered spend

```
Metered credits = max(0, total monthly consumption − pool)
Metered cost    = metered credits × $0.01
Maximum bill    = license fees + enterprise spending limit   (when "Stop usage when budget limit is reached" is on)
```

### Example: what one interaction costs

A chat turn that sends 20,000 input tokens and gets 2,000 output tokens back:

| Model | Calculation | Cost | AI credits |
|-------|-------------|-----:|-----------:|
| **GPT-5 mini** | (20,000 × $0.25 + 2,000 × $2.00) ÷ 1,000,000 | $0.009 | 0.9 |
| **GPT-5.4** | (20,000 × $2.50 + 2,000 × $15.00) ÷ 1,000,000 | $0.08 | 8 |

> 💡 **Model choice matters:** the same interaction costs roughly **9× more** on GPT-5.4 than on GPT-5 mini. Default everyday work to lighter models and reserve frontier models for hard tasks.

### Example: monthly estimate for a team

| Input | Value |
|-------|-------|
| **Plan** | Copilot Enterprise (3,900 credits per user per month) |
| **Users** | 50 |
| **Average consumption** | 4,500 credits per user per month |

| Calculation | Result |
|-------------|--------|
| Pool | 50 × 3,900 = **195,000** credits |
| Consumption | 50 × 4,500 = **225,000** credits |
| Metered credits | max(0, 225,000 − 195,000) = **30,000** |
| **Metered cost** | 30,000 × $0.01 = **$300 per month** (on top of 50 × $39 = $1,950 in license fees) |

> 💡 Because the pool is shared, light users offset heavy users. If half the team used 2,000 credits and half used 7,000, the total (225,000) — and the metered cost — would be the same.

---

## 5️⃣ Enterprise Controls

### A) Allow or Block Additional Usage (AI credits paid usage)

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **AI controls** → **Copilot** *(sidebar)*

1. At the top of the enterprise page, click **AI controls**.
2. In the sidebar, click **Copilot**.
3. Find the **AI credits paid usage** policy and choose **Enabled** (usage continues past the pool at per-credit rates) or **Disabled** (usage stops when the pool is exhausted). *(The selection applies immediately — there is no Save button.)*

> ⚠️ **Warning:** With paid usage enabled and no enterprise spending limit set to stop usage, there is no cap on metered charges. Always pair it with the budgets in 5C.

### B) Control Which Models Are Available

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **AI controls** → **Copilot** → **Configure models**

1. Click **Configure models**.
2. Set each model to **Enabled**, **Disabled**, or **Delegate** (lets organizations decide).

> 💡 **Tip:** If budget is a concern, disable the most expensive models enterprise-wide, then enable them for the teams that need them.

### C) Set Budgets (GitHub's Recommended Setup)

**👤 Role:** GitHub **enterprise owner** or **billing manager** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **Billing and licensing** → **Budgets and alerts** → **New budget**

Every budget is created the same way:

1. Click **New budget**.
2. Under **Budget Type**, select **Bundled AI credits budget** (covers every feature that uses AI credits).
3. Under **Budget scope**, choose the scope — see the table below.
4. Under **Budget**, enter the amount in US dollars.
5. For spending limits, select **Stop usage when budget limit is reached**. *(Without it, the budget only sends an email — usage is **not** stopped.)*
6. Under **Alerts**, select **Receive budget threshold alerts** (notifies at 75%, 90%, and 100%) and choose the **Alert Recipients**.
7. Click **Create budget**.

Create them in this order:

| # | Budget | Scope setting | What it does |
|---|--------|---------------|--------------|
| 1 | **Universal user-level budget** | **Users** — leave the user field empty | Caps each user's total consumption (pool + metered). Set it **above** the license value ($19 Business, $39 Enterprise). Always a hard stop. |
| 2 | **Individual overrides** for power users | **Users** — pick the user; optionally set **Expiration** | Raises or lowers one person's cap; overrides the universal budget. Use an expiration for temporary boosts. |
| 3 | **Enterprise spending limit** | **Enterprise** | Caps total metered charges after the pool is exhausted. |
| 4 | **Stop usage** on every spending limit | (step 5 above) | Turns spending limits from alerts into hard stops. |

> 📌 Any budget set to **$0** stops usage immediately for everyone it applies to.

### D) Create Cost Centers for Department Allocation

**👤 Role:** GitHub **enterprise owner** or **billing manager** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **Billing and licensing** → **Cost centers** → **New cost center**

1. Click **Cost centers**, then **New cost center** (upper-right).
2. Under **Name**, enter a name (e.g., `Engineering`).
3. If your account is billed to Azure, optionally add an **Azure ID**.
4. Under **Resources**, select the organizations, repositories, users, and/or enterprise teams that belong to it.
5. Click **Create cost center**.

| Cost center | Members | Example budget |
|-------------|---------|----------------|
| Engineering | `platform-eng` org | $2,000/month cost center budget |
| Data Science | `data-science` org | $5,000/month cost center budget |
| Copilot Pilot | pilot enterprise team | $500/month cost center budget |

> 💡 Add a **cost center budget** (scope **Cost center**) to cap that team's metered charges, or a **cost center user-level budget** (scope **Users** → pick the cost center) to give every member the same per-person cap.

### E) Handle Requests for More Budget

When a user exhausts their budget, they can ask for more.

1. Go to the enterprise (or organization) settings and click **Requests from members**.
2. Set a new amount for each request, select the requests, and click **Approve and increase**.

---

## 6️⃣ Best-Practice Rollout

| Phase | Action | When |
|-------|--------|------|
| **1. Pilot** | License a small cohort (10–20 users) and put them in a dedicated cost center | 2–4 weeks |
| **2. Guardrails first** | Set a universal user-level budget and an enterprise spending limit with **Stop usage** on | Before the pilot starts |
| **3. Monitor** | Review the **AI usage** page by user and model | Weekly during the pilot |
| **4. Tune** | Add individual overrides for genuine power users; restrict expensive models if needed | End of pilot |
| **5. Expand** | Roll out to broader teams with budgets already in place | After evaluation |

> 💡 **Tip:** If you have no usage data yet, start with a universal user-level budget that feels reasonable and revisit it after the first billing cycle.

---

## 7️⃣ Monitoring Consumption

**👤 Role:** GitHub **enterprise owner** or **billing manager** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **Billing and licensing** → **AI usage** *(under "Metered usage")*

1. Open **AI usage**. By default, the chart and table group consumption by model.
2. Use the filter, **Group by**, and **Timeframe** controls to view usage by user, model, organization, or cost center.
3. Export the data for deeper analysis.

| What to look for | What it means |
|------------------|---------------|
| Users blocked early in the cycle | The universal user-level budget is too tight, or a power user needs an override |
| Metered charges appearing | The pool is running out before month end — check usage trends and spending limits |
| Pool lasts all month, nobody blocked | Target state — budgets are well sized |

> 💡 **Alerts:** besides budget threshold alerts (75% / 90% / 100%), you can opt in to **included usage alerts** at 90% and 100% of the pool on the **Budgets and alerts** page. Users see their own consumption under **Your Copilot** → **Usage** → **Usage this cycle**.

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
| **A user is blocked from Copilot mid-cycle** | Their user-level budget is used up, the pool is exhausted with **AI credits paid usage** disabled, or a cost center or enterprise spending limit with **Stop usage** was reached. | Check the user on the **AI usage** page and the budgets on **Budgets and alerts**; raise or add an individual budget, approve their budget request, or enable paid usage. |
| **Metered charges with no budget alerts** | Budget threshold alerts weren't selected, or no spending limit exists. | Edit each budget, select **Receive budget threshold alerts**, and add alert recipients. |
| **Content exclusions do not apply immediately** | Client policy cache, unsupported surface/mode, symlink/remote filesystem limitation, or indirect IDE context. | Reload the IDE policy, verify the exclusion syntax at enterprise/org/repo scope, and document surfaces where exclusions are limited. |
| **Cloud agent or MCP action is denied** | Agent policy, MCP policy, repository permissions, secrets, or server allowlist does not permit the operation. | Review Enterprise AI controls > Agents/MCP, repo-level permissions, MCP server configuration, and audit logs for the denied action. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>


### Q: The shared pool ran out mid-month — what happens?
**A:** It depends on the **AI credits paid usage** policy. If it's enabled (the default), Copilot keeps working and usage is charged at $0.01 per credit, until a spending limit with **Stop usage when budget limit is reached** is hit. If it's disabled, Copilot features that use AI credits are blocked until the pool resets at 00:00 UTC on the first of the month. Code completions and next edit suggestions keep working either way.

---

### Q: How do we see which users and models consume the most credits?
**A:** Go to Enterprise → **Billing and licensing** → **AI usage**. Group or filter by user, model, organization, or cost center, and export the data for analysis.

---

### Q: Can we set a hard cap on spending?
**A:** Yes. Create an enterprise-scope **Bundled AI credits budget** and select **Stop usage when budget limit is reached** — your maximum monthly bill is then your license fees plus that limit. User-level budgets are always hard stops for the individual. Without **Stop usage**, a spending limit only sends alerts.

---

### Q: Model prices changed since we did our estimate — what should we do?
**A:** Re-run your estimate with the current rates from [Models and pricing](https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing), compare against your actual usage on the **AI usage** page, and adjust your budgets.

---

### Q: Do AI credits reset monthly? Do unused credits roll over?
**A:** The pool resets at 00:00:00 UTC on the first day of each calendar month. Unused credits do not roll over.

---

### Q: Paid usage is enabled, but users are still being blocked — why?
**A:** A budget is stopping them. Check, in order: the user's individual or cost center user-level budget, the universal user-level budget, any cost center budget with **Stop usage**, and the enterprise spending limit. A **$0** budget at any applicable level blocks usage immediately. The user can also send a budget request, which you approve under **Requests from members**.

---

### Q: How do we estimate costs before rolling out to the whole enterprise?
**A:** Run a 2–4 week pilot in a dedicated cost center with a universal user-level budget and an enterprise spending limit in place. Use the **AI usage** page to get average credits per user, then apply the formula in step 4 to your full user count.

---

### Q: Are code completions billed in AI credits?
**A:** No. Code completions and next edit suggestions are unlimited on all paid plans and don't consume AI credits.

---

### Q: Some of our developers still see "premium requests" — why?
**A:** Premium requests and model multipliers are the legacy billing model. They only still apply to existing **annual** Copilot Pro and Pro+ subscriptions until those plans end. Copilot Business and Copilot Enterprise moved to AI credits on June 1, 2026. Older IDE extensions can also show outdated billing terms — update to the latest version.

</details>

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Seat Assignment & Enablement | `Copilot/Seat Assignment & Enablement.md` |
| Admin Controls (Models, Content Exclusion, Instructions) | `Copilot/Admin Controls (Models, Content Exclusion, Instructions).md` |
| Cost Centers & Department Billing | `Billing/Cost Centers & Department Billing.md` |
| Overage Budgets & Cost Monitoring (GHEC Enterprise) | `Copilot/Overage Budgets & Cost Monitoring (GHEC Enterprise).md` |
| AI Credits Budgeting Scenarios | `Copilot/AI Credits Budgeting Scenarios.md` |
| Power User AI Credit Allowance (Cost Centers + Budgets) | `Copilot/Power User AI Credit Allowance (Cost Centers + Budgets).md` |

---

## 📚 Resources

- [Usage-based billing for organizations and enterprises](https://docs.github.com/en/copilot/concepts/billing-and-usage/organizations-and-enterprises/billing)
- [Copilot budget controls](https://docs.github.com/en/copilot/concepts/billing-and-usage/organizations-and-enterprises/budgets)
- [Getting started with budget controls](https://docs.github.com/en/copilot/tutorials/budgets/getting-started-with-budget-controls)
- [Models and pricing](https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing)
- [Setting up budgets](https://docs.github.com/en/billing/how-tos/set-up-budgets)
- [What changed with Copilot billing (legacy)](https://docs.github.com/en/copilot/reference/copilot-billing/request-based-billing-legacy/what-changed-with-billing)
- [GitHub Copilot is moving to usage-based billing (GitHub Blog)](https://github.blog/news-insights/company-news/github-copilot-is-moving-to-usage-based-billing/)

---

*Last updated: October 2026*
