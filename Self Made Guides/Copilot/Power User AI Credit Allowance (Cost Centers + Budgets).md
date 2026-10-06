# 🎯 GitHub Copilot Power User AI Credit Allowance Runbook

> **Give a defined group of "hyper power users" more Copilot AI credits — including paid usage past the shared pool — while everyone else stays capped**

> 📌 **Billing changed on June 1, 2026.** Premium requests and model multipliers were replaced by **AI credits** (1 credit = $0.01). The old pattern — a "$0 overage budget for everyone" plus an "allowed overage" cost center — is replaced by **user-level budgets**, which cap each person directly.

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [📋 Overview](#-overview)
- [✅ Prerequisites](#-prerequisites)
- [🏗️ Recommended Design (Most Operable)](#️-recommended-design-most-operable)
- [1️⃣ Step 1 — Enable "AI credits paid usage" (Enterprise Policy)](#1️⃣-step-1--enable-ai-credits-paid-usage-enterprise-policy)
- [2️⃣ Step 2 — Remove or Fix Any Budgets That Would Block Power Users](#2️⃣-step-2--remove-or-fix-any-budgets-that-would-block-power-users)
- [3️⃣ Step 3 — Create the "Hyper power users" Cost Center](#3️⃣-step-3--create-the-hyper-power-users-cost-center)
- [4️⃣ Step 4 — Create the Budgets (The Enforcement)](#4️⃣-step-4--create-the-budgets-the-enforcement)
- [5️⃣ Step 5 — Verify It's Working](#5️⃣-step-5--verify-its-working)
- [💡 Optional: Temporary Boosts and Budget Requests](#-optional-temporary-boosts-and-budget-requests)
- [📝 What No Longer Works Under AI Credits](#-what-no-longer-works-under-ai-credits)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Policy:** Enterprise → **AI controls** → **Copilot** → **AI credits paid usage** → **Enabled**
- **Cost center:** Enterprise → **Billing and licensing** → **Cost centers** → **New cost center** → name `Hyper power users` → add users or an enterprise team → **Create cost center**
- **Everyone's cap:** **Budgets and alerts** → **New budget** → **Bundled AI credits budget** → scope **Users** (leave empty) → amount → **Create budget**
- **Power users' cap:** **New budget** → **Bundled AI credits budget** → scope **Users** → select the `Hyper power users` cost center → higher amount → **Create budget**
- **Hard ceiling on total metered spend:** **New budget** → **Bundled AI credits budget** → scope **Enterprise** → amount → **Stop usage when budget limit is reached** → **Create budget**

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

## 📋 Overview

Every Copilot license adds AI credits to one **shared pool** for your enterprise. Without per-person limits, a few heavy users can drain it early in the month. This runbook gives everyone a sensible per-person cap and gives a named group of power users a higher one [[1]](#source-1).

Key mechanics to keep in mind:

- **User-level budgets cap each person's total consumption** — from the shared pool *and* from paid usage after the pool runs out. They're always a hard stop [[1]](#source-1).
- **The most specific user-level budget wins:** individual → cost center → universal. An individual budget wins even if it's **lower** than the cost center's [[1]](#source-1).
- **Spending limits** (cost center, organization, enterprise budgets) only apply after the pool is exhausted, and only stop usage if **Stop usage when budget limit is reached** is selected [[1]](#source-1).

---

## ✅ Prerequisites

| Requirement | Who / Role needed | ✓ |
|-------------|-------------------|---|
| GitHub Enterprise Cloud enterprise with Copilot Business or Copilot Enterprise | GitHub **enterprise owner** | ☐ |
| Set the AI credits paid usage policy | GitHub **enterprise owner** | ☐ |
| Create cost centers and budgets | GitHub **enterprise owner** or **billing manager** | ☐ |
| Decision approved by platform and finance owners: the per-person caps and the enterprise spending limit | Finance / platform owners | ☐ |
| List of hyper power users (GitHub usernames or managed user accounts), or an enterprise team that contains them | Platform owner | ☐ |

---

## 🏗️ Recommended Design (Most Operable)

| Control | Who it covers | Example amount | Hard stop? |
|---------|---------------|----------------|------------|
| **Universal user-level budget** | Every licensed user | $25 / person / month (2,500 credits) | Always |
| **Cost center user-level budget** on `Hyper power users` | Members of that cost center | $150 / person / month (15,000 credits) | Always |
| **Cost center budget** on `Hyper power users` *(optional)* | That team's metered charges after the pool runs out | $2,000 / month | With **Stop usage** |
| **Enterprise spending limit** | All metered charges after the pool runs out | $5,000 / month | With **Stop usage** |

> 💡 **Sizing:** GitHub recommends setting the universal budget **above** the per-license value ($19 for Copilot Business, $39 for Copilot Enterprise) so typical users aren't blocked [[2]](#source-2). Your maximum monthly bill is your license fees plus the enterprise spending limit [[1]](#source-1).

---

## 1️⃣ Step 1 — Enable "AI credits paid usage" (Enterprise Policy)

If this policy is disabled, nobody can use Copilot past the shared pool — regardless of budgets [[6]](#source-6).

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **AI controls** → **Copilot** *(sidebar)*

1. At the top of the enterprise page, click **AI controls**.
2. In the sidebar, click **Copilot**.
3. Set **AI credits paid usage** to **Enabled**. *(It's enabled by default; the selection applies immediately — there is no Save button.)*

---

## 2️⃣ Step 2 — Remove or Fix Any Budgets That Would Block Power Users

**👤 Role:** GitHub **enterprise owner** or **billing manager** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **Billing and licensing** → **Budgets and alerts**

1. Review every budget in the list.
2. For any budget that would block power users — for example a **$0** budget, or a low cost center or organization budget with **Stop usage when budget limit is reached** — click **…** next to it, then **Edit** (to raise it) or **Delete**.
3. Follow the prompts to confirm.

> ⚠️ **Any applicable $0 budget stops usage immediately** [[1]](#source-1). Clear these out before testing.

---

## 3️⃣ Step 3 — Create the "Hyper power users" Cost Center

**👤 Role:** GitHub **enterprise owner** or **billing manager** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **Billing and licensing** → **Cost centers** → **New cost center**

1. Click **Cost centers**, then **New cost center** (upper-right).
2. Under **Name**, enter `Hyper power users`.
3. If your account is billed to Azure, optionally add an **Azure ID**.
4. Under **Resources**, add the users — or an **enterprise team** that contains them, so membership stays current as people join or leave [[4]](#source-4).
5. Click **Create cost center**.

> 📌 You **don't** need a "Default users" cost center anymore — the universal user-level budget in Step 4 covers everyone automatically.

---

## 4️⃣ Step 4 — Create the Budgets (The Enforcement)

**👤 Role:** GitHub **enterprise owner** or **billing manager** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **Billing and licensing** → **Budgets and alerts** → **New budget**

Each budget starts the same way: click **New budget**, then under **Budget Type** select **Bundled AI credits budget** [[3]](#source-3).

### 4A) Universal user-level budget (everyone)

1. Under **Budget scope**, select **Users** and leave the user field **empty**.
2. Under **Budget**, enter the per-person amount (e.g., **$25**).
3. Under **Alerts**, select **Receive budget threshold alerts** (75% / 90% / 100%) and choose the **Alert Recipients**.
4. Click **Create budget**.

### 4B) Cost center user-level budget (hyper power users)

1. Under **Budget scope**, select **Users**, then select the **Hyper power users** cost center.
2. Under **Budget**, enter the higher per-person amount (e.g., **$150**).
3. Select **Receive budget threshold alerts** and choose the recipients.
4. Click **Create budget**.

### 4C) Cost center budget for the team's metered spend (optional)

1. Under **Budget scope**, select **Cost center**, then the **Hyper power users** cost center.
2. Enter the team's monthly metered limit (e.g., **$2,000**).
3. Select **Stop usage when budget limit is reached** for a hard cap — or leave it clear for alerts only.
4. Select **Receive budget threshold alerts**, then click **Create budget**.

### 4D) Enterprise spending limit (everyone's metered spend)

1. Under **Budget scope**, select **Enterprise**.
2. Enter the monthly limit (e.g., **$5,000**).
3. Select **Stop usage when budget limit is reached** — without it, this is an alert, not a guardrail [[2]](#source-2).
4. Select **Receive budget threshold alerts**, then click **Create budget**.

---

## 5️⃣ Step 5 — Verify It's Working

### A) Verify no conflicting budgets

1. On **Budgets and alerts**, confirm you see the universal budget, the `Hyper power users` user-level budget, and your spending limits — and no unexpected $0 or low budgets.
2. Check whether any power user has an **individual** user-level budget — it overrides the cost center budget, even if it's lower.

### B) Verify usage lands in the right place

1. Go to Enterprise → **Billing and licensing** → **AI usage** *(under "Metered usage")*.
2. Group or filter by **cost center** and by **user**, and confirm power users' consumption appears under `Hyper power users` [[5]](#source-5).
3. Ask a power user to check **Your Copilot** → **Usage** → **Usage this cycle** — it shows credits used out of their budget (for example, "450 / 15,000 AI credits used").

---

## 💡 Optional: Temporary Boosts and Budget Requests

- **Temporary boost for one person:** create an **individual** user-level budget (scope **Users** → select the user) and set **Expiration** to **End of current billing cycle** or a **Specific date**. When it expires, the user falls back to their cost center or universal budget [[1]](#source-1).
- **Requests from users:** a user who runs out can request more. Go to the enterprise settings → **Requests from members**, enter a new amount, select the request, and click **Approve and increase** [[7]](#source-7).

---

## 📝 What No Longer Works Under AI Credits

- **"A second organization increases the included allowance."** Included credits are pooled across the billing entity, so moving users to another organization in the same enterprise doesn't add credits.
- **"Copilot Enterprise is cheaper for heavy users."** Under AI credits, Copilot Enterprise's extra included credits cost exactly what you pay for them, and extra usage is $0.01 per credit on both plans — so it's never cheaper for the same usage. See [AI Credits Budgeting Scenarios](AI%20Credits%20Budgeting%20Scenarios.md).
- **"Premium request paid usage" policy and premium-request budgets.** These belong to the legacy model, which now applies only to existing *annual* Copilot Pro and Pro+ subscriptions until they end.

## 🧯 Known Errors & Resolutions

<details>
<summary><em>Show known errors table</em></summary>


> This section lists the known product errors and admin-facing symptoms that commonly occur with this workflow. Exact message text can vary by product rollout, tenant policy, and provider, so use the log or settings page named in the resolution to confirm the root cause.

| Error or symptom | Likely cause | Resolution |
|------------------|--------------|------------|
| **Page, tab, or button is missing** | Wrong account context, missing admin role, unavailable plan/add-on, or feature rollout not enabled for the selected enterprise/org/repo. | Switch to the correct account and scope, confirm the prerequisite role, verify licensing or add-on activation, then refresh the page. If the control is still absent, use the direct settings URL from the relevant GitHub Docs page and confirm the feature is available for your plan. |
| **Changes appear saved but behavior does not change** | Policy inheritance, cached UI state, propagation delay, or an overlapping enterprise/org/repo policy. | Reopen the settings page, verify the effective policy at the lowest affected scope, wait for propagation where documented, and check for a stricter policy at an enterprise or organization level. |
| **403, forbidden, or resource not accessible** | The signed-in user or token can see the page but lacks the specific permission for the action. | Use an enterprise owner, organization owner, repository admin, or token with the exact scopes/permissions listed in the runbook. For SAML-protected orgs, authorize the token or SSH key for SSO before retrying. |
| **A power user is still capped at the universal amount** | They're not in the cost center's **Resources**, the cost center user-level budget wasn't created at scope **Users** → cost center, or they have a lower **individual** budget (which wins). | Check the cost center's members, the budget's scope, and any individual budget for that user. |
| **Power users are blocked after the pool runs out** | **AI credits paid usage** is disabled, or a cost center or enterprise spending limit with **Stop usage** was reached. | Enable the policy, or raise the spending limit that was hit. |
| **Copilot feature, model, or policy is not visible** | Plan, license assignment, enterprise policy, org delegation, or feature rollout does not permit it. | Check enterprise AI controls, organization Copilot settings, assigned seat status, and the plan requirements for the feature. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>


### Q: I added users to the "Hyper power users" cost center but they're still capped at the universal amount — why?
**A:** Check three things: (1) the cost center user-level budget exists and was created with scope **Users** → the cost center (not scope **Cost center**, which is a spending limit instead); (2) the users are listed in the cost center's **Resources**; and (3) they don't have an individual user-level budget — an individual budget always wins, even if it's lower.

---

### Q: A new user joined — are they covered?
**A:** Yes. The universal user-level budget applies automatically to every Copilot-licensed user, including new ones. If you add power users through an enterprise team, they also join the cost center automatically.

---

### Q: I moved a user between cost centers — when does it take effect?
**A:** User-level budgets follow cost center membership, so the new cost center's per-person budget applies to the user once they're a member. If you use included usage controls, the cost centers' pool caps are recalculated at the start of the next billing cycle.

---

### Q: Can I manage this with the API?
**A:** Yes. Use the [Budgets REST API](https://docs.github.com/en/rest/billing/budgets) to create user-level budgets (including an `expires_at` date for individual budgets) and the enterprise billing REST API to manage cost centers [[8]](#source-8).

---

### Q: How do I confirm usage is landing in the right cost center?
**A:** Go to Enterprise → **Billing and licensing** → **AI usage** and filter or group by cost center. If a user belongs to more than one cost center (for example, through an organization and an enterprise team), see GitHub's cost center allocation rules [[5]](#source-5).

</details>

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| AI Credits Budget & Overage Planning | `Copilot/AI Credits Budget & Overage Planning.md` |
| AI Credits Budgeting Scenarios | `Copilot/AI Credits Budgeting Scenarios.md` |
| Overage Budgets & Cost Monitoring (GHEC Enterprise) | `Copilot/Overage Budgets & Cost Monitoring (GHEC Enterprise).md` |
| Cost Centers & Department Billing | `Billing/Cost Centers & Department Billing.md` |

---

## 📝 Resources

| # | Title | Link |
|:-:|-------|------|
| <a id="source-1"></a>1 | Copilot budget controls | [GitHub Docs](https://docs.github.com/en/copilot/concepts/billing-and-usage/organizations-and-enterprises/budgets) |
| <a id="source-2"></a>2 | Getting started with budget controls | [GitHub Docs](https://docs.github.com/en/copilot/tutorials/budgets/getting-started-with-budget-controls) |
| <a id="source-3"></a>3 | Setting up budgets | [GitHub Docs](https://docs.github.com/en/billing/how-tos/set-up-budgets) |
| <a id="source-4"></a>4 | Using cost centers | [GitHub Docs](https://docs.github.com/en/billing/how-tos/products/use-cost-centers) |
| <a id="source-5"></a>5 | Cost center allocation | [GitHub Docs](https://docs.github.com/en/billing/reference/cost-center-allocation) |
| <a id="source-6"></a>6 | Usage-based billing for organizations and enterprises | [GitHub Docs](https://docs.github.com/en/copilot/concepts/billing-and-usage/organizations-and-enterprises/billing) |
| <a id="source-7"></a>7 | Managing requests for additional budget | [GitHub Docs](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-budget-requests) |
| <a id="source-8"></a>8 | Budgets REST API | [GitHub Docs](https://docs.github.com/en/rest/billing/budgets) |

---

*Last updated: October 2026*
