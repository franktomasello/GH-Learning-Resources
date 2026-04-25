# 💰 GitHub Enterprise Cost Centers & Department Billing Runbook

> **Complete guide to configuring cost centers, budgets, Azure billing, and usage reporting for GitHub Enterprise Cloud**

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [📋 Overview](#-overview)
- [1️⃣ Create Cost Centers](#1-create-cost-centers)
- [2️⃣ Assign Resources to Cost Centers](#2-assign-resources-to-cost-centers)
- [3️⃣ Cost Center Scope and Limitations](#3-cost-center-scope-and-limitations)
- [4️⃣ Patterns for Team-Level Billing](#4-patterns-for-team-level-billing)
- [5️⃣ SKU-Level Budgets for Premium Requests](#5-sku-level-budgets-for-premium-requests)
- [6️⃣ Set Budgets and Hard Stops](#6-set-budgets-and-hard-stops)
- [7️⃣ Connect an Azure Subscription](#7-connect-an-azure-subscription)
- [8️⃣ Export Usage Reports for Finance Reconciliation](#8-export-usage-reports-for-finance-reconciliation)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Create cost center:** `Enterprise → Billing and licensing → Cost centers → New cost center`
- **Assign resources:** `Enterprise → Billing and licensing → Cost centers → [Cost center] → Resources → Add resources`
- **Create SKU budget:** `Enterprise → Billing and licensing → Budgets and alerts → New budget → Scope: cost center`
- **Set hard-stop budget:** `Enterprise → Billing and licensing → Budgets and alerts → New budget → Stop usage when budget limit is reached`
- **Export usage report:** `Enterprise → Billing and licensing → Usage report → Download CSV`

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

## ✅ Prerequisites

| Requirement | Status |
|-------------|--------|
| Enterprise owner role | ☐ |
| GitHub Enterprise Cloud account | ☐ |
| Azure subscription linked (for Azure billing) | ☐ |
| Organizations, repositories, or users identified for cost allocation | ☐ |

---

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **GitHub enterprise or organization owner** | Starts the Azure metered billing connection from GitHub. | Enterprise path: GitHub → profile photo → Your enterprises → [enterprise] → Billing and licensing → Payment information → Metered billing via Azure → Add Azure Subscription. Organization path: GitHub → profile photo → Your organizations → [organization] → Settings → Billing and licensing → Payment information → Metered billing via Azure → Add Azure Subscription. Then sign in to Microsoft → Permissions requested → Accept → Select a subscription → Connect. Handoff: the subscription ID is visible on Payment information. |
| **Azure subscription Owner** | Provides the Azure subscription that GitHub will bill against, or grants another signer the required Azure RBAC rights. | Azure portal → Subscriptions → [subscription] → Access control (IAM) → Role assignments → confirm the signer is listed under Owner. To grant access: Add → Add role assignment → Privileged administrator roles → Owner → Members → Select members → [user] → Select → Review + assign. Handoff: subscription ID and tenant ID. |
| **Microsoft Entra Global Administrator or consent approver** | Approves tenant-wide consent when the Microsoft consent prompt blocks the GitHub billing app. | Microsoft Entra admin center → Entra ID → Enterprise apps → Activity → Admin consent requests → My Pending → [GitHub request] → Review permissions and consent → Approve. If the Global Administrator completes the GitHub flow directly, approve the Permissions requested prompt by clicking Accept. |

---

## 📋 Overview

This runbook covers how to set up cost tracking and billing for enterprise departments:

| Capability | Description |
|------------|-------------|
| **Cost centers** | Group organizations, repositories, or users for billing attribution |
| **Budgets and alerts** | Monitor metered spend and hard-stop usage where supported |
| **SKU-level budgets** | Set budgets for premium requests and specific SKUs |
| **Azure subscription** | Connect Azure for consolidated billing |
| **Usage reports** | Export data for finance reconciliation |

---

## 1️⃣ Create Cost Centers

**Navigation:**

```
Enterprise → Billing and licensing → Cost centers
  → New cost center
```

**Steps:**

1. Click **New cost center**
2. Enter a **Name** (e.g., `Engineering`, `Marketing`, `Platform Team`)
3. Add one or more **resources** to the cost center: organizations, repositories, or users
4. If the cost center should bill to a different Azure subscription than the enterprise default, add that Azure subscription during creation
5. Click **Create**

> ✅ **Result:** Spending by the assigned resources is attributed to this cost center for reporting purposes.

---

## 2️⃣ Assign Resources to Cost Centers

**Navigation:**

```
Enterprise → Billing and licensing → Cost centers
  → [Select cost center] → Resources → Add resources
```

**Steps:**

1. Click on an existing cost center
2. Open the **Resources** area
3. Click **Add resources**
4. Select organizations, repositories, or users from the list
5. Save changes

| Rule | Detail |
|------|--------|
| **Resources are allocated to one active cost center** | A resource should be assigned to the cost center that finance expects to own the resulting spend |
| **Unassigned resources** | Usage that is not covered by a cost center is attributed to the enterprise or owning account instead of a department-specific cost center |
| **User-based allocation matters for Copilot** | License-based products and premium-request usage are allocated by user, so user-scoped cost centers are the right pattern for role-based Copilot budgets |

> ✅ **Result:** Usage from assigned resources rolls up to the cost center according to GitHub's product-specific cost center allocation rules.

---

## 3️⃣ Cost Center Scope and Limitations

> ⚠️ **Important:** Current GitHub cost centers can include organizations, repositories, and users. They do **not** support GitHub teams as a first-class cost center resource.

| Scope | Supported |
|-------|-----------|
| **Organization** | Yes — useful for business-unit or org-based allocation |
| **Repository** | Yes — useful when usage should follow specific repositories |
| **User** | Yes — useful for Copilot license and premium-request allocation |
| **Team** | No — teams are not a first-class cost center resource |

---

## 4️⃣ Patterns for Team-Level Billing

*If you need to track costs by department or team within an organization:*

### Strategy A: Use User-Scoped Cost Centers

Use this when the team cost is driven mostly by Copilot seats, Copilot premium requests, or other user-attributed products.

**Steps:**

1. Create a cost center for the team or department
2. Add the team members as user resources
3. Create budgets scoped to that cost center
4. Keep the user list synchronized as people move teams

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

```
Enterprise → Settings → Organizations → New organization
```

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

## 5️⃣ SKU-Level Budgets for Premium Requests

*Set budgets per cost center for specific product SKUs (e.g., Copilot premium requests).*

**Navigation:**

```
Enterprise → Billing and licensing → Cost centers
  → [Select cost center] → Budgets → New budget
```

**Steps:**

1. Select the cost center
2. Click **New budget**
3. Choose the **SKU** (e.g., `Copilot Premium Requests`)
4. Set the **budget amount** (USD per month)
5. Configure **alert thresholds** (e.g., 75%, 90%, 100%)
6. Save the budget

| Setting | Description |
|---------|-------------|
| **Budget amount** | Monthly spending cap for the selected SKU |
| **Alert thresholds** | Email notifications when spending hits a percentage |
| **Enforcement** | Budgets are informational (alerts only) unless hard limits are configured |

> 💡 **Tip:** Use hard-stop budgets where GitHub supports enforcement, and alert-only budgets where you only need notification.

---

## 6️⃣ Set Budgets and Hard Stops

**Navigation:**

```
Enterprise → Billing and licensing → Budgets and alerts
```

**Steps:**

1. Click **New budget** or edit an existing budget for the product (e.g., **GitHub Actions**, **GitHub Packages**, **Copilot premium requests**)
2. Set a **monthly budget** (USD per month):

| Option | Effect |
|--------|--------|
| **$0** | No metered spending allowed (included minutes/storage only) |
| **Set a specific budget** | Spending is tracked against that monthly amount |
| **Stop usage enabled** | Metered usage stops when the budget limit is reached, where GitHub supports hard stops for that product |
| **Stop usage disabled** | Budget sends alerts only; usage can continue beyond the budget |

3. Configure alert thresholds and save

> ⚠️ **Important:** Hard-stop behavior applies to metered products that support stopping usage. License-based budgets generally alert but do not prevent license assignment.

---

## 7️⃣ Connect an Azure Subscription

*Route GitHub Enterprise billing through your existing Azure EA or pay-as-you-go subscription.*

### A) Add an Azure Subscription

**Navigation:**

```
Enterprise → Billing and licensing → Payment information
  → Add Azure subscription
```

**Steps:**

1. Click **Add Azure subscription**
2. Sign in to your Azure account when prompted
3. Select the Azure subscription to link
4. Authorize the connection
5. Confirm the subscription details

> ✅ **Result:** GitHub usage charges appear on your Azure invoice.

### B) EA vs. Metered Billing Comparison

| Feature | EA (Enterprise Agreement) | Metered (Pay-as-you-go) |
|---------|--------------------------|------------------------|
| **Billing model** | Pre-committed annual spend | Usage-based monthly billing |
| **Seat licensing** | Covered by EA commitment | Billed per-seat monthly |
| **Metered usage** | Overage billed against EA | Billed at list price |
| **Invoice** | Consolidated Azure invoice | Consolidated Azure invoice |
| **Discount eligibility** | EA discount rates apply | Standard pricing |
| **Commitment** | Annual (1-3 year terms) | Month-to-month |

### C) Cross-Tenant Azure Subscription Handling

> ⚠️ **Important:** The Azure subscription must be in the same Azure AD tenant that your GitHub enterprise billing admin has access to.

| Scenario | Solution |
|----------|----------|
| **Same tenant** | Connect directly via the UI |
| **Different tenant** | The billing admin must be invited as a guest user in the Azure AD tenant that owns the subscription, or use a service principal with cross-tenant access |
| **Multiple subscriptions** | The enterprise can have a default Azure subscription, and cost centers can route their usage to different Azure subscriptions when configured in the cost center UI |

---

## 8️⃣ Export Usage Reports for Finance Reconciliation

**Navigation:**

```
Enterprise → Billing and licensing → Usage report
  → Download usage report (CSV)
```

**Steps:**

1. Select the **date range** for the report
2. Click **Download** to get a CSV export
3. The report includes:

| Column | Description |
|--------|-------------|
| **Organization** | Which org incurred the cost |
| **Product** | GitHub Actions, Copilot, Packages, etc. |
| **SKU** | Specific line item (e.g., premium requests) |
| **Quantity** | Units consumed |
| **Cost** | Dollar amount |
| **Cost center** | Attributed cost center (if assigned) |

> 💡 **Tip:** Schedule monthly exports and share with your finance team for reconciliation against Azure invoices.

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
**A:** First, verify that the expected organizations, repositories, or users are assigned to the cost center (Enterprise > Billing and licensing > Cost centers > [Your cost center] > Resources). Usage data can take time to appear in cost center reports after resources are assigned. If the resources were just added, check again after the next usage/billing data refresh.

---

### Q: I cannot create cost centers. The option does not appear. What do I need?
**A:** Creating cost centers requires the enterprise owner role. Billing managers can view billing data but cannot create or modify cost centers. Contact your enterprise owner to create cost centers, or request the enterprise owner role if your responsibilities require it.

---

### Q: I set up a budget alert but it never fired even though spending exceeded the threshold. What happened?
**A:** Budget alerts are based on projected usage trends, not real-time actual spending. If spending spiked suddenly rather than gradually increasing, the projection may not have triggered the alert before the threshold was crossed. Also verify the alert thresholds are configured correctly and that the notification email is going to a monitored inbox (check spam filters).

---

### Q: Our Azure subscription for billing is in a different tenant than our SSO/EMU identity tenant. Is that a problem?
**A:** Billing and identity are independent concerns. The Azure subscription used for GitHub billing does not need to be in the same Entra ID tenant as your SAML/SSO or EMU identity configuration. However, the billing admin who links the Azure subscription must have access to that subscription -- if it is in a different tenant, the admin may need to be invited as a guest user in that tenant.

---

### Q: Can I split billing by team within a single organization?
**A:** Not directly by GitHub team object, but you can usually model this with user-scoped or repository-scoped cost centers. Use user resources for Copilot seat and premium-request allocation, and repository resources for repository-driven metered usage such as Actions. Only create separate organizations when you also need a separate admin boundary, policy boundary, or clear business-unit ownership model.

---

### Q: How often are usage reports updated, and can I automate the export?
**A:** Usage reports in the GitHub enterprise billing dashboard are updated daily. You can manually download CSV exports from Enterprise > Billing and licensing > Usage report. For automation, use the GitHub billing REST API to programmatically retrieve usage data and feed it into your finance systems or data warehouse.

</details>

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Premium Request Budget & Overage Planning | `Copilot/Premium Request Budget & Overage Planning.md` |
| Overage Budgets & Cost Monitoring (GHEC Enterprise) | `Copilot/Overage Budgets & Cost Monitoring (GHEC Enterprise).md` |
| Visual Studio Subscription to GitHub Enterprise Linking | `Billing/Visual Studio Subscription to GitHub Enterprise Linking.md` |
| Enterprise Environment Scaffolding Checklist | `Setup/Enterprise Environment Scaffolding Checklist.md` |
| Minutes Governance & Runner Strategy | `Actions/Minutes Governance & Runner Strategy.md` |

---

## 📝 Resources

| Resource | Link |
|----------|------|
| **Cost centers** | [GitHub Docs](https://docs.github.com/en/billing/concepts/cost-centers) |
| **Budgets and alerts** | [GitHub Docs](https://docs.github.com/en/billing/concepts/budgets-and-alerts) |
| **Connecting an Azure subscription** | [GitHub Docs](https://docs.github.com/en/billing/how-tos/set-up-payment/connect-azure-sub) |

---

*Last updated: April 2026*
