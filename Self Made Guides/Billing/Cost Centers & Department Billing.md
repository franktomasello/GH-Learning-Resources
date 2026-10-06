# 💰 GitHub Enterprise Cost Centers & Department Billing Runbook

> **Complete guide to configuring cost centers, budgets, Azure billing, and usage reporting for GitHub Enterprise Cloud**

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [📋 Overview](#-overview)
- [1️⃣ Create Cost Centers](#1️⃣-create-cost-centers)
- [2️⃣ Assign Resources to Cost Centers](#2️⃣-assign-resources-to-cost-centers)
- [3️⃣ Cost Center Scope and Limitations](#3️⃣-cost-center-scope-and-limitations)
- [4️⃣ Patterns for Team-Level Billing](#4️⃣-patterns-for-team-level-billing)
- [5️⃣ SKU-Level and AI Credit Budgets for a Cost Center](#5️⃣-sku-level-and-ai-credit-budgets-for-a-cost-center)
- [6️⃣ Set Budgets and Hard Stops](#6️⃣-set-budgets-and-hard-stops)
- [7️⃣ Connect an Azure Subscription](#7️⃣-connect-an-azure-subscription)
- [8️⃣ Export Usage Reports for Finance Reconciliation](#8️⃣-export-usage-reports-for-finance-reconciliation)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Create a cost center:** `Enterprise → Billing and licensing → Cost centers → New cost center` → name → (optional Azure ID) → resources → **Create cost center**
- **Edit resources:** `Cost centers` → **⋯** next to the cost center → **Edit**
- **Cost center budget:** `Enterprise → Billing and licensing → Budgets and alerts → New budget` → type → **Budget scope: Cost center** → amount → **Create budget**
- **Hard stop:** in the budget, select **Stop usage when budget limit is reached**
- **Usage report:** `Enterprise → Billing and licensing → Usage` → **Get usage report** (emailed CSV)
- **Azure billing:** `Billing and licensing → Payment information` → **Metered billing via Azure** → **Add Azure Subscription**

---

## ✅ Accuracy & Click-Path Notes

<details>
<summary><em>Show click-path conventions</em></summary>


- Reviewed against current public GitHub and Microsoft documentation in October 2026 where public documentation is available. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
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
| GitHub Enterprise Cloud on metered (usage-based) billing — cost centers don't apply to volume or subscription billing | — | ☐ |
| Create and edit cost centers for any resource; create budgets | **Enterprise owner** or **billing manager** | ☐ |
| Create cost centers for resources in their own organization | **Organization owner** | ☐ |
| Connect an Azure subscription | GitHub **enterprise owner** + an Azure user who is subscription **Owner** and can give tenant-wide admin consent (or a Global Administrator) | ☐ |
| A mapping of organizations, repositories, users, or enterprise teams to finance cost codes | Finance + platform team | ☐ |

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **GitHub enterprise or organization owner** | Starts the Azure metered billing connection from GitHub. | Enterprise path: GitHub → Enterprise → Billing and licensing → Payment information → scroll to Metered billing via Azure → Add Azure Subscription. Organization path: GitHub → Organizations → [organization] → Settings → Billing & Licensing → Payment information → Metered billing via Azure → Add Azure Subscription. Then sign in to Microsoft → Permissions requested → Accept → Select a subscription → tick the confirmation checkbox → Connect. Handoff: the subscription ID is visible on Payment information. |
| **Azure subscription Owner** | Provides the Azure subscription that GitHub will bill against, or grants another signer the required Azure RBAC rights. | Azure portal → Subscriptions → [subscription] → Access control (IAM) → Role assignments → confirm the signer is listed under Owner. To grant access: Add → Add role assignment → Privileged administrator roles → Owner → Members → Select members → [user] → Select → Review + assign. Handoff: subscription ID and tenant ID. |
| **Microsoft Entra Global Administrator or consent approver** | Approves tenant-wide consent when the Microsoft consent prompt blocks the GitHub billing app. | Microsoft Entra admin center → Entra ID → Enterprise apps → Activity → Admin consent requests → My Pending → [GitHub request] → Review permissions and consent → Approve. If the Global Administrator completes the GitHub flow directly, approve the Permissions requested prompt by clicking Accept. |

---

## 📋 Overview

| Capability | Description |
|------------|-------------|
| **Cost centers** | Attribute spend to business units using organizations, repositories, users, or enterprise teams |
| **Budgets and alerts** | Alert at 75%, 90%, and 100%, and optionally stop metered usage at the limit |
| **SKU-level and AI credit budgets** | Budgets for one SKU, or for AI credits across Copilot, cloud agent, and Spark |
| **Azure subscription** | Pay for metered usage through Azure — one per enterprise and, optionally, one per cost center |
| **Usage reports** | Summarized, detailed, and AI usage CSVs for finance reconciliation |

---

## 1️⃣ Create Cost Centers

**👤 Role:** **Enterprise owner** or **billing manager** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **Billing and licensing** → **Cost centers**

**Steps:**

1. Click **New cost center** (upper right).
2. Under **Name**, enter a name (for example `Engineering` or `Platform Team`).
3. *(Optional — Azure billing only)* Add an **Azure ID** to bill this cost center to a different subscription than the enterprise default. GitHub verifies it against Azure.
4. Under **Resources**, select the **organizations**, **repositories**, **users**, and/or **enterprise teams** to include.
5. Click **Create cost center**.

> ✅ **Result:** future metered spend for those resources is attributed to the cost center.

> 📌 **Limits:** up to 1,000 active cost centers per enterprise and 25,000 resources per cost center; add or remove up to 50 resources at a time. Outside collaborators and unaffiliated users can only be added through the cost center API.

---

## 2️⃣ Assign Resources to Cost Centers

**Navigate:** Enterprise → **Billing and licensing** → **Cost centers** → **⋯** next to the cost center → **Edit**

**Steps:**

1. Click **⋯** to the right of the cost center, then **Edit**.
2. Add or remove organizations, repositories, users, or enterprise teams.
3. Save your changes.

| Rule | Detail |
|------|--------|
| **One cost center per resource** | A resource belongs to only one cost center. Adding it elsewhere **moves** it. |
| **Usage-based products** (Actions, Codespaces, Packages, LFS) | Charged by the **repository or organization** where the usage happens |
| **License-based products** (Copilot, GitHub Enterprise, GHAS) | Charged by the **user** first, otherwise the organization that's billed for the license |
| **AI credits** | Charged by the **user** who used them, otherwise the organization that granted their Copilot license |
| **Enterprise teams** | Members are added automatically as they join or leave. A direct user assignment wins over team membership |
| **Unassigned usage** | Shows as **Enterprise Only** when you group usage by cost center |

> 💡 Deleting a cost center sends future usage to the enterprise; past usage stays with the cost center (see the **Deleted** tab).

---

## 3️⃣ Cost Center Scope and Limitations

| Resource | Supported |
|----------|-----------|
| **Organization** | ✅ Business-unit or org-based allocation |
| **Repository** | ✅ When usage should follow specific repositories |
| **User** | ✅ Copilot licenses, AI credits, and other per-user products |
| **Enterprise team** | ✅ Membership stays in sync automatically |
| **Organization team** | ❌ Not a cost center resource — use an enterprise team or user list |

> ⚠️ **Important:** a budget applies to the whole cost center. If two teams need separate budgets, give each its own cost center (they can share an Azure subscription).

---

## 4️⃣ Patterns for Team-Level Billing

*If you need to track costs by department or team within an organization:*

### Strategy A: Use User- or Enterprise-Team-Scoped Cost Centers

Use this when the cost is driven mostly by Copilot seats, AI credits, or other user-attributed products.

**Steps:**

1. Create a cost center for the team or department.
2. Add the matching **enterprise team** (best — stays in sync) or the users.
3. Create budgets scoped to that cost center.
4. If you add users directly, keep the list current as people move teams.

### Strategy B: Use Repository-Scoped Cost Centers

Use this when the team cost is driven mostly by Actions, Packages, Codespaces, or other repository-attributed metered usage.

**Steps:**

1. Create a cost center for the workload or team
2. Add the repositories that generate the usage
3. Create budgets scoped to that cost center
4. Review repository ownership regularly so the allocation stays correct

### Strategy C: Create Separate Organizations per Cost Center

**Steps:**

1. Create a new organization for each billing group:

Enterprise → **Organizations** → **New organization**

2. Name organizations to reflect departments (e.g., `acme-engineering`, `acme-data-science`)
3. Move relevant repositories to the appropriate org only if org-level governance and cost ownership should be separated
4. Create cost centers and assign each org to its respective cost center

| Pros | Cons |
|------|------|
| True cost separation by department | More orgs to manage |
| Clean billing reports per group | Teams may need cross-org access |
| Works well for hard business boundaries | Repository transfers required |

> 💡 **Tip:** Use inner source (internal repository visibility) to maintain cross-org collaboration even when repos are split across organizations.

---

## 5️⃣ SKU-Level and AI Credit Budgets for a Cost Center

**👤 Role:** **Enterprise owner** or **billing manager** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **Billing and licensing** → **Budgets and alerts** → **New budget**

**Steps:**

1. Under **Budget Type**, choose:
   - **SKU-level budget** — then a product and a SKU (for example Copilot AI credits or Copilot cloud agent), or
   - **Bundled AI credits budget** — all AI credit SKUs (Copilot, cloud agent, Spark).
2. Under **Budget scope**, choose **Cost center** and select it.
3. Under **Budget**, enter the monthly amount.
4. *(Optional)* Select **Stop usage when budget limit is reached**.
5. Under **Alerts**, select **Receive budget threshold alerts** and choose **Alert Recipients**.
6. Click **Create budget**.

| Setting | Description |
|---------|-------------|
| **Budget amount** | Monthly cap for the selected SKU or AI credits |
| **Threshold alerts** | Email and banner at 75%, 90%, and 100% |
| **Stop usage** | Without it, the budget only alerts |

> 💡 **Copilot cost centers:** you can also turn on **included usage controls**, which cap the cost center's share of included AI credits to what its own licenses fund. See `Copilot/Power User AI Credit Allowance (Cost Centers + Budgets).md`.

---

## 6️⃣ Set Budgets and Hard Stops

**Navigate:** Enterprise → **Billing and licensing** → **Budgets and alerts**

**Steps:**

1. Click **New budget** (or **⋯** → **Edit** on an existing one).
2. Choose **Product-level budget** (for example Actions, Packages, Codespaces), **SKU-level budget**, or **Bundled AI credits budget**.
3. Choose the **Budget scope**: **Enterprise**, **Organization**, **Repository**, or **Cost center**.
4. Enter the amount and choose whether to stop usage:

| Option | Effect |
|--------|--------|
| **$0 + stop usage** | No metered spending (included usage only) |
| **Amount + stop usage** | Metered usage stops at the limit |
| **Amount, no stop usage** | Alerts only — usage continues |

5. Select **Receive budget threshold alerts**, pick recipients, and click **Create budget**.

> ⚠️ **Important:** **Stop usage** works for metered products (and Advanced Security licenses, where offered as **Limit usage**). Without it, you get emails but usage isn't stopped.

---

## 7️⃣ Connect an Azure Subscription

*Pay for metered GitHub usage on your Azure invoice.*

### A) Add an Azure subscription

**👤 Role:** GitHub **enterprise owner** + Azure subscription **Owner** who can give tenant-wide admin consent · **📍 Portal:** GitHub + Microsoft sign-in

**Navigate:** Enterprise → **Billing and licensing** → **Payment information**

**Steps:**

1. Scroll to the bottom. Next to **Metered billing via Azure**, click **Add Azure Subscription**.
2. Sign in to your Microsoft account.
3. On **Permissions requested**, click **Accept**. *(If you see "Need admin approval", a Global Administrator must approve GitHub's Subscription Permission Validation app — see the Action Matrix.)*
4. Under **Select a subscription**, choose the subscription ID. If it's not listed, enter the correct **tenant ID**.
5. Select **By clicking "Connect", you are confirming that you want to be billed for metered services via the selected Azure subscription**.
6. Click **Connect**.

> ✅ **Result:** metered usage from that point on is billed through Azure on the 1st of each month. Charges before the connection are billed by GitHub as usual.

### B) How Azure billing works

| Topic | Detail |
|-------|--------|
| **What's billed through Azure** | Metered (usage-based) charges, shown by product family and by `enterprise:sku` or `costcenter:sku` |
| **Billing date** | The 1st of each month |
| **Prepaid usage** | Not available through Azure |
| **MACC** | GitHub products on the Azure invoice count toward a Microsoft Azure Consumption Commitment |
| **Existing volume or prepaid agreements** | Continue until they expire or you're invited to switch |
| **Microsoft Enterprise Agreement customers** | Connecting Azure is the only way to use GHAS, Codespaces, Copilot, or extra Actions/LFS/Packages |

### C) Tenant and multi-subscription handling

| Scenario | What to do |
|----------|------------|
| **Subscription in another tenant** | The person connecting must be **Owner** of the subscription in that tenant; enter that tenant ID during **Select a subscription** |
| **Consent blocked** | A Global Administrator approves the GitHub app's admin consent request, or completes the flow themselves |
| **Multiple subscriptions** | Keep one default for the enterprise and add an **Azure ID** to individual cost centers. Usage of cost centers without one goes to the default |

> 📌 Billing and identity are separate: the Azure subscription doesn't need to be in the same Entra tenant as your SAML/OIDC or EMU configuration.

---

## 8️⃣ Export Usage Reports for Finance Reconciliation

**👤 Role:** **Enterprise owner** or **billing manager** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **Billing and licensing** → **Usage**

**Steps:**

1. Open **Metered usage** (or **AI usage**).
2. *(Optional)* Group or filter by **cost center**, organization, or product.
3. Click **Get usage report** and choose the report type and date range.
4. GitHub emails the CSV to your default email address (one report request at a time).

| Report | Covers |
|--------|--------|
| **Summarized usage report** | All paid products, up to one year |
| **Detailed usage report** | Adds `username` and `workflow_path`, up to 31 days (web UI only) |
| **AI usage report** | Per-user AI credits with `model` and token counts, up to 31 days |

**Key columns:** `date`, `product`, `sku`, `quantity`, `unit_type`, `gross_amount`, `discount_amount`, `net_amount`, `organization`, `repository`, `cost_center_name`.

> 💡 **Tip:** reconcile monthly with finance. For automation, use `GET /enterprises/{enterprise}/settings/billing/usage` (summarized data).

## 🧯 Known Errors & Resolutions

<details>
<summary><em>Show known errors table</em></summary>


> This section lists the known product errors and admin-facing symptoms that commonly occur with this workflow. Exact message text can vary by product rollout, tenant policy, and provider, so use the log or settings page named in the resolution to confirm the root cause.

| Error or symptom | Likely cause | Resolution |
|------------------|--------------|------------|
| **Page, tab, or button is missing** | Wrong account context, missing admin role, unavailable plan/add-on, or feature rollout not enabled for the selected enterprise/org/repo. | Switch to the correct account and scope, confirm the prerequisite role, verify licensing or add-on activation, then refresh the page. If the control is still absent, use the direct settings URL from the relevant GitHub Docs page and confirm the feature is available for your plan. |
| **Changes appear saved but behavior does not change** | Policy inheritance, cached UI state, propagation delay, or an overlapping enterprise/org/repo policy. | Reopen the settings page, verify the effective policy at the lowest affected scope, wait for propagation where documented, and check for a stricter policy at an enterprise or organization level. |
| **403, forbidden, or resource not accessible** | The signed-in user or token can see the page but lacks the specific permission for the action. | Use an enterprise owner, organization owner, repository admin, or token with the exact scopes/permissions listed in the runbook. For SAML-protected orgs, authorize the token or SSH key for SSO before retrying. |
| **Budget does not block usage** | Budget is alert-only, scoped to the wrong account/resource/SKU, or another budget is the one applying. | Edit or recreate the budget with the correct scope and SKU. For metered products, enable `Stop usage when budget limit is reached` if you need a hard stop, then verify there are no overlapping budgets with a different behavior. |
| **Usage report or cost center shows zero usage** | Usage has not processed yet, resources are not assigned to the cost center, or the report date range is wrong. | Confirm the cost center resources, select a date range after assignment, and wait for normal billing data latency before reconciling. |
| **Azure subscription is not listed or connection fails** | The Azure user lacks subscription owner rights or tenant-wide consent is required. | Sign in with an Azure subscription owner who can grant consent, or have an Entra global administrator approve the GitHub Subscription Permission Validation app, then repeat the Add Azure Subscription flow. |
| **Admin approval required during Azure billing connection** | The tenant blocks user consent for the GitHub billing app. | Use the tenant admin consent workflow or have a Global Administrator grant consent, then return to GitHub and select the subscription again. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>


### Q: My cost center is showing $0 usage even though the assigned organizations are actively using GitHub. Why?
**A:** Check the cost center's resources (**Cost centers** → **⋯** → **View details**) and remember the allocation rules: Actions and other usage-based products follow the **repository or organization**, while Copilot and other licenses follow the **user**. Assignments only affect usage from that point on. Also confirm you're on metered billing — cost centers don't apply to volume or subscription agreements.

---

### Q: I cannot create cost centers. The option does not appear. What do I need?
**A:** Enterprise owners and **billing managers** can create cost centers for any resource; organization owners can create them for resources in their own organization. If **Cost centers** is missing, check your role and that the enterprise is on metered (usage-based) billing.

---

### Q: I set up a budget alert but it never fired even though spending exceeded the threshold. What happened?
**A:** Threshold alerts (75%, 90%, 100%) only go out if **Receive budget threshold alerts** is selected, and only to the chosen **Alert Recipients**. Check that the budget's type and scope actually cover the spend (for example, a cost center budget won't see usage from repositories outside it), and check spam filters. Usage data can lag by up to a day.

---

### Q: Our Azure subscription for billing is in a different tenant than our SSO/EMU identity tenant. Is that a problem?
**A:** No. Billing and identity are independent — the subscription doesn't have to be in your SSO/EMU tenant. The person connecting it must be an **Owner** of the subscription and able to give (or get) tenant-wide admin consent in **its** tenant; during **Select a subscription**, enter that tenant's ID if the subscription isn't listed.

---

### Q: Can I split billing by team within a single organization?
**A:** Yes. Organization teams can't be added, but **enterprise teams** can — membership stays in sync. Otherwise, use user resources for Copilot and AI credits and repository resources for Actions and other usage-based products. Create separate organizations only when you also need a separate admin or policy boundary.

---

### Q: How often are usage reports updated, and can I automate the export?
**A:** Request CSVs from Enterprise → **Billing and licensing** → **Usage** → **Get usage report** (emailed). For automation, call `GET /enterprises/{enterprise}/settings/billing/usage` or `/usage/summary` — the API returns summarized data; the detailed report is web-only.

</details>

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| AI Credits Budget & Overage Planning | `Copilot/AI Credits Budget & Overage Planning.md` |
| Overage Budgets & Cost Monitoring (GHEC Enterprise) | `Copilot/Overage Budgets & Cost Monitoring (GHEC Enterprise).md` |
| Visual Studio Subscription to GitHub Enterprise Linking | `Billing/Visual Studio Subscription to GitHub Enterprise Linking.md` |
| Enterprise Environment Scaffolding Checklist | `Setup/Enterprise Environment Scaffolding Checklist.md` |
| Minutes Governance & Runner Strategy | `Actions/Minutes Governance & Runner Strategy.md` |

---

## 📝 Resources

| Resource | Link |
|----------|------|
| **Cost centers** | [GitHub Docs](https://docs.github.com/en/billing/concepts/cost-centers) |
| **Using cost centers** | [GitHub Docs](https://docs.github.com/en/billing/how-tos/products/use-cost-centers) |
| **Cost center allocation** | [GitHub Docs](https://docs.github.com/en/billing/reference/cost-center-allocation) |
| **Setting up budgets** | [GitHub Docs](https://docs.github.com/en/billing/how-tos/set-up-budgets) |
| **Connecting an Azure subscription** | [GitHub Docs](https://docs.github.com/en/billing/how-tos/set-up-payment/connect-azure-sub) |
| **Billing reports** | [GitHub Docs](https://docs.github.com/en/billing/reference/billing-reports) |

---

*Last updated: October 2026*
