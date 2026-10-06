# 📊 GitHub Copilot Overage Budgets & Cost Monitoring Runbook

> **Cap and monitor Copilot spending past the included AI credits — enterprise-wide, by organization, or by cost center**

> 📌 **Billing changed on June 1, 2026.** Copilot now bills in **AI credits** (1 credit = $0.01) from a shared pool; "overage" means **additional (metered) usage** after that pool is exhausted. Premium-request budgets and the "Premium request paid usage" policy belong to the legacy model.

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [📋 Overview](#-overview)
- [🚨 Critical Warning (Read This First)](#-critical-warning-read-this-first)
- [🤔 Why Budget at Each Level?](#-why-budget-at-each-level)
- [0️⃣ One-Time Prerequisite: Allow (or Block) Additional Usage via Policy](#0️⃣-one-time-prerequisite-allow-or-block-additional-usage-via-policy)
- [📘 Guide A — Enterprise Spending Limit](#-guide-a--enterprise-spending-limit)
- [📗 Guide B — Budget by Organization](#-guide-b--budget-by-organization)
- [📙 Guide C — Budget by Cost Center](#-guide-c--budget-by-cost-center)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Policy gate:** Enterprise → **AI controls** → **Copilot** → **AI credits paid usage** *(enabled by default)*
- **Budgets:** Enterprise → **Billing and licensing** → **Budgets and alerts** → **New budget** → **Bundled AI credits budget** → scope **Enterprise**, **Organization**, or **Cost center** → amount → **Stop usage when budget limit is reached** → **Create budget**
- **Monitor:** Enterprise → **Billing and licensing** → **AI usage** → group/filter by organization, cost center, user, or model
- **Export:** **AI usage** or **Metered usage** → **Get usage report** → **Email me the report**

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
| Set the AI credits paid usage policy | GitHub **enterprise owner** | ☐ |
| Create budgets, cost centers, and usage reports | GitHub **enterprise owner** or **billing manager** | ☐ |
| Organization budgets (Guide B, from the organization side) | GitHub **organization owner** | ☐ |

---

## 📋 Overview

These budgets — called **spending limits** — cap **metered charges after the shared pool of AI credits is exhausted**. They don't limit how much anyone draws from the pool; for that, use user-level budgets (see [AI Credits Budget & Overage Planning](AI%20Credits%20Budget%20%26%20Overage%20Planning.md)).

| Guide | Scope | Best for |
|-------|-------|----------|
| **Guide A** | Enterprise | One global cap on total metered spend |
| **Guide B** | Organization | A cap per org, team, or product line |
| **Guide C** | Cost center | Chargeback and business-unit ownership, including cross-org groups |

> 📝 **Scope note:** This runbook is for GitHub Enterprise Cloud (enterprise billing with Copilot Business or Copilot Enterprise).

---

## 🚨 Critical Warning (Read This First)

- **A spending limit is only an alert unless you select "Stop usage when budget limit is reached."** Without it, you get an email when the limit is exceeded, but usage — and charges — continue.
- **Spending limits don't replace each other.** Creating a new one doesn't override an existing one. Once the pool is exhausted, any applicable spending limit with **Stop usage** that's used up blocks the users it covers.
- **Your maximum monthly bill** is your license fees **plus** the enterprise spending limit — the enterprise limit is not a total monthly budget.

---

## 🤔 Why Budget at Each Level?

| Level | Best when… | Tradeoff |
|-------|------------|----------|
| **Enterprise** | You want a single guardrail on total metered spend — simple for Finance and early rollout | One heavy org or team can use the whole cap and block everyone |
| **Organization** | Orgs map to teams or product lines, or you want to stop one org affecting others | More budgets to manage; only covers users whose Copilot seats are billed to that org |
| **Cost center** | Budgets must follow financial entities (business unit, department, project), including across orgs | Requires cost center setup |

---

## 0️⃣ One-Time Prerequisite: Allow (or Block) Additional Usage via Policy

The **AI credits paid usage** policy decides whether anyone can use Copilot past the shared pool at all.

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **AI controls** → **Copilot** *(sidebar)*

1. At the top of the enterprise page, click **AI controls**.
2. In the sidebar, click **Copilot**.
3. Find **AI credits paid usage** and choose:
   - **Enabled** — usage continues past the pool at $0.01 per credit, within your spending limits (the default).
   - **Disabled** — usage stops when the pool is exhausted; spending limits never come into play.

> 📌 The selection applies immediately — there is no Save button.

---

## 📘 Guide A — Enterprise Spending Limit

### A1) Navigate to Budgets and Alerts

**👤 Role:** GitHub **enterprise owner** or **billing manager** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **Billing and licensing** → **Budgets and alerts**

### A2) Create the Enterprise Spending Limit

1. Click **New budget**.
2. Under **Budget Type**, select **Bundled AI credits budget**.
3. Under **Budget scope**, select **Enterprise**.
4. Under **Budget**, enter the monthly limit for metered charges.
5. Select **Stop usage when budget limit is reached**.
6. Under **Alerts**, select **Receive budget threshold alerts** (75%, 90%, and 100%) and choose the **Alert Recipients**.
7. Click **Create budget**.

### A3) Monitor Costs (Enterprise-Wide)

**Navigate:** Enterprise → **Billing and licensing** → **AI usage** *(under "Metered usage")*

1. Review the chart and table — grouped by model by default.
2. Use **Group by**, the filter, and **Timeframe** to view usage by user, model, organization, or cost center.
3. To export, click **Get usage report** at the top of the page, specify the report details, and click **Email me the report**. *(The download link arrives by email and expires after 24 hours.)*

> 💡 **Also watch:** on **Budgets and alerts**, select **Receive alerts when my included usage reaches 90% and 100%** to know when the shared pool is running low.

---

## 📗 Guide B — Budget by Organization

### B1) Navigate to Budgets and Alerts

**👤 Role:** GitHub **enterprise owner** or **billing manager** (or the **organization owner**, from the organization's settings) · **📍 Portal:** GitHub

**Navigate (enterprise):** Enterprise → **Billing and licensing** → **Budgets and alerts**
**Navigate (organization):** Organization → **Settings** → **Billing and licensing** *(sidebar, under "Access")* → **Budgets and alerts**

### B2) Create an Organization-Scoped Budget

1. Click **New budget**.
2. Under **Budget Type**, select **Bundled AI credits budget**.
3. Under **Budget scope**, select **Organization**, then choose the organization.
4. Under **Budget**, enter the monthly limit.
5. Select **Stop usage when budget limit is reached** for a hard cap.
6. Select **Receive budget threshold alerts** and choose the recipients.
7. Click **Create budget**.
8. Repeat for each organization you want to cap separately.

> 📌 **What it covers:** an organization budget caps metered charges for users whose Copilot seats are billed to **that** organization. Organization owners can only use it to restrict further — it can't override a higher-level budget. If a user has seats in several organizations, GitHub picks one each billing cycle to bill the seat, so their spend may count against a different organization's budget from month to month.

### B3) Monitor Costs by Organization

**Navigate:** Enterprise → **Billing and licensing** → **AI usage**

1. Set **Group by** to organization, or filter to a single organization.
2. Export with **Get usage report** → **Email me the report** if needed.

---

## 📙 Guide C — Budget by Cost Center

### C1) Create a Cost Center

**👤 Role:** GitHub **enterprise owner** or **billing manager** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **Billing and licensing** → **Cost centers** → **New cost center**

1. Click **Cost centers**, then **New cost center** (upper-right).
2. Under **Name**, enter a name (for example, `Payments BU`).
3. If your account is billed to Azure, optionally add an **Azure ID**.
4. Under **Resources**, select the organizations, repositories, users, and/or enterprise teams that belong to it.
5. Click **Create cost center**.

### C2) Create a Cost Center Budget

1. On **Budgets and alerts**, click **New budget**.
2. Under **Budget Type**, select **Bundled AI credits budget**.
3. Under **Budget scope**, select **Cost center**, then choose the cost center.
4. Under **Budget**, enter the monthly limit.
5. Select **Stop usage when budget limit is reached** for a hard cap.
6. Select **Receive budget threshold alerts** and choose the recipients.
7. Click **Create budget**.

> 📌 When a cost center's budget is used up, **only that cost center's users** are blocked — other teams are unaffected. A cost center budget doesn't extend or override anyone's user-level budget.

> 💡 **Also limit the pool draw:** a cost center budget only applies after the pool is exhausted. To stop a team using more than its share of the pool itself, turn on the cost center's included usage control — see [AI Credits Budgeting Scenarios](AI%20Credits%20Budgeting%20Scenarios.md).

### C3) Monitor Costs by Cost Center

**Navigate:** Enterprise → **Billing and licensing** → **AI usage**

1. Filter or group by cost center.
2. Export with **Get usage report** → **Email me the report**.
3. For a cross-product view, open **Metered usage** and search `product:copilot cost_center:<cost-center-name>`.

## 🧯 Known Errors & Resolutions

<details>
<summary><em>Show known errors table</em></summary>


> This section lists the known product errors and admin-facing symptoms that commonly occur with this workflow. Exact message text can vary by product rollout, tenant policy, and provider, so use the log or settings page named in the resolution to confirm the root cause.

| Error or symptom | Likely cause | Resolution |
|------------------|--------------|------------|
| **Page, tab, or button is missing** | Wrong account context, missing admin role, unavailable plan/add-on, or feature rollout not enabled for the selected enterprise/org/repo. | Switch to the correct account and scope, confirm the prerequisite role, verify licensing or add-on activation, then refresh the page. If the control is still absent, use the direct settings URL from the relevant GitHub Docs page and confirm the feature is available for your plan. |
| **Changes appear saved but behavior does not change** | Policy inheritance, cached UI state, propagation delay, or an overlapping enterprise/org/repo policy. | Reopen the settings page, verify the effective policy at the lowest affected scope, wait for propagation where documented, and check for a stricter policy at an enterprise or organization level. |
| **403, forbidden, or resource not accessible** | The signed-in user or token can see the page but lacks the specific permission for the action. | Use an enterprise owner, organization owner, repository admin, or token with the exact scopes/permissions listed in the runbook. For SAML-protected orgs, authorize the token or SSH key for SSO before retrying. |
| **Metered charges kept growing past the limit** | **Stop usage when budget limit is reached** wasn't selected, so the budget only sends alerts. | Edit the budget and select **Stop usage when budget limit is reached**. |
| **Users blocked even though a new budget allows spend** | Another applicable spending limit with **Stop usage** is used up, a **$0** budget applies, or the user's user-level budget is exhausted. | Review every budget on **Budgets and alerts**; raise or delete the one that's blocking. |
| **Copilot feature, model, or policy is not visible** | Plan, license assignment, enterprise policy, org delegation, or feature rollout does not permit it. | Check enterprise AI controls, organization Copilot settings, assigned seat status, and the plan requirements for the feature. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>


### Q: I created a new budget but users are still being blocked — why?
**A:** A new budget doesn't override existing ones. Once the shared pool is exhausted, any applicable spending limit with **Stop usage** that's used up blocks the users it covers — and a user-level budget can block a user at any time. Check every budget on **Budgets and alerts**, including any **$0** budget, which stops usage immediately.

---

### Q: Budget alert emails aren't arriving — what should I check?
**A:** Edit the budget and confirm **Receive budget threshold alerts** is selected and the right people are listed under **Alert Recipients**. Alerts go out at 75%, 90%, and 100% of the budget, by email and as a banner on GitHub. Also check spam filters for GitHub notification emails.

---

### Q: Do we still need to look for a legacy "$0 premium request budget"?
**A:** Premium-request budgets belong to the legacy billing model. What matters now is any **$0** budget of any type that applies to your users — it stops their usage immediately. Review **Budgets and alerts** and edit or delete any you don't intend.

---

### Q: Can I change the scope of an existing budget (for example, from enterprise to organization)?
**A:** Click **…** next to the budget → **Edit** to see what can be changed. If the scope can't be edited, create a new budget with the scope you want, then delete the old one (**…** → **Delete**).

---

### Q: One heavy org used up the enterprise spending limit and blocked everyone — how do we prevent this?
**A:** Add organization or cost center budgets (Guides B and C) so each team has its own cap, and give everyone a universal user-level budget so no single person can run away with the pool. Keep the enterprise spending limit as a final backstop.

</details>

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| AI Credits Budget & Overage Planning | `Copilot/AI Credits Budget & Overage Planning.md` |
| AI Credits Budgeting Scenarios | `Copilot/AI Credits Budgeting Scenarios.md` |
| Power User AI Credit Allowance (Cost Centers + Budgets) | `Copilot/Power User AI Credit Allowance (Cost Centers + Budgets).md` |
| Cost Centers & Department Billing | `Billing/Cost Centers & Department Billing.md` |

---

## 📝 Resources

| Resource | Link |
|----------|------|
| Copilot budget controls | [GitHub Docs](https://docs.github.com/en/copilot/concepts/billing-and-usage/organizations-and-enterprises/budgets) |
| Setting up budgets | [GitHub Docs](https://docs.github.com/en/billing/how-tos/set-up-budgets) |
| Using cost centers | [GitHub Docs](https://docs.github.com/en/billing/how-tos/products/use-cost-centers) |
| Viewing usage and downloading reports | [GitHub Docs](https://docs.github.com/en/billing/how-tos/products/view-productlicense-use) |
| Managing Copilot spending for your company | [GitHub Docs](https://docs.github.com/en/copilot/how-tos/manage-and-track-spending/manage-company-spending) |

---

*Last updated: October 2026*
