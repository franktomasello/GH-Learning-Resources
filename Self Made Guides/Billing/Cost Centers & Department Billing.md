# 💰 Cost Centers & Department Billing Setup Runbook

> **Complete guide to configuring cost centers, budgets, Azure billing, and usage reporting for GitHub Enterprise Cloud**

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Create cost center:** `Enterprise → Settings → Billing → Cost centers → Create cost center`
- **Assign orgs:** `Enterprise → Settings → Billing → Cost centers → [Cost center] → Manage organizations`
- **Create SKU budget:** `Enterprise → Settings → Billing → Cost centers → [Cost center] → Budgets → Create budget`
- **Set spending limit:** `Enterprise → Settings → Billing → Spending limits → [Product] → Set limit`
- **Export usage report:** `Enterprise → Settings → Billing → Usage report → Download CSV`

---

## ✅ Prerequisites

| Requirement | Status |
|-------------|--------|
| Enterprise owner role | ☐ |
| GitHub Enterprise Cloud account | ☐ |
| Azure subscription linked (for Azure billing) | ☐ |
| Organizations created and assigned to the enterprise | ☐ |

---

## 📋 Overview

This runbook covers how to set up cost tracking and billing for enterprise departments:

| Capability | Description |
|------------|-------------|
| **Cost centers** | Group organizations for billing attribution |
| **Spending limits** | Cap metered usage spending |
| **SKU-level budgets** | Set budgets for premium requests and specific SKUs |
| **Azure subscription** | Connect Azure for consolidated billing |
| **Usage reports** | Export data for finance reconciliation |

---

## 1️⃣ Create Cost Centers

**Navigation:**

```
Enterprise → Settings → Billing (sidebar) → Cost centers
  → Create cost center
```

**Steps:**

1. Click **Create cost center**
2. Enter a **Name** (e.g., `Engineering`, `Marketing`, `Platform Team`)
3. Assign one or more **organizations** to the cost center
4. Click **Create**

> ✅ **Result:** Spending by the assigned organizations is attributed to this cost center for reporting purposes.

---

## 2️⃣ Assign Organizations to Cost Centers

**Navigation:**

```
Enterprise → Settings → Billing (sidebar) → Cost centers
  → [Select cost center] → Manage organizations
```

**Steps:**

1. Click on an existing cost center
2. Click **Add organization**
3. Select organizations from the list
4. Save changes

| Rule | Detail |
|------|--------|
| **One org per cost center** | An organization can only belong to one cost center at a time |
| **Unassigned orgs** | Organizations not in a cost center appear under "Unassigned" in reports |

> ✅ **Result:** All usage from assigned organizations rolls up to the cost center.

---

## 3️⃣ Cost Center Scope Limitations

> ⚠️ **Important:** Cost centers operate at the **organization level only** — not at the team level.

| Scope | Supported |
|-------|-----------|
| **Organization** | Yes — assign entire orgs to cost centers |
| **Team** | No — teams within an org cannot be split across cost centers |
| **Repository** | No — individual repos cannot be assigned to cost centers |

---

## 4️⃣ Workaround for Team-Level Billing

*If you need to track costs by department or team within an organization:*

### Strategy: Create Separate Organizations per Cost Center

**Steps:**

1. Create a new organization for each billing group:

```
Enterprise → Settings → Organizations → New organization
```

2. Name organizations to reflect departments (e.g., `acme-engineering`, `acme-data-science`)
3. Move relevant repositories to the appropriate org
4. Create cost centers and assign each org to its respective cost center

| Pros | Cons |
|------|------|
| True cost separation by department | More orgs to manage |
| Clean billing reports per group | Teams may need cross-org access |
| Works with existing cost center features | Repository transfers required |

> 💡 **Tip:** Use inner source (internal repository visibility) to maintain cross-org collaboration even when repos are split across organizations.

---

## 5️⃣ SKU-Level Budgets for Premium Requests

*Set budgets per cost center for specific product SKUs (e.g., Copilot premium requests).*

**Navigation:**

```
Enterprise → Settings → Billing (sidebar) → Cost centers
  → [Select cost center] → Budgets → Create budget
```

**Steps:**

1. Select the cost center
2. Click **Create budget**
3. Choose the **SKU** (e.g., `Copilot Premium Requests`)
4. Set the **budget amount** (USD per month)
5. Configure **alert thresholds** (e.g., 75%, 90%, 100%)
6. Save the budget

| Setting | Description |
|---------|-------------|
| **Budget amount** | Monthly spending cap for the selected SKU |
| **Alert thresholds** | Email notifications when spending hits a percentage |
| **Enforcement** | Budgets are informational (alerts only) unless hard limits are configured |

> 💡 **Tip:** Use budgets in combination with spending limits for hard enforcement of caps.

---

## 6️⃣ Set Spending Limits

**Navigation:**

```
Enterprise → Settings → Billing (sidebar) → Spending limits
```

**Steps:**

1. Locate the product (e.g., **GitHub Actions**, **GitHub Packages**, **Copilot**)
2. Set a **spending limit** (USD per month):

| Option | Effect |
|--------|--------|
| **$0** | No metered spending allowed (included minutes/storage only) |
| **Set a specific limit** | Spending stops when the limit is reached |
| **Unlimited** | No cap on metered spending |

3. Click **Update**

> ⚠️ **Important:** When a spending limit is reached, metered services stop working (e.g., Actions workflows will queue but not run).

---

## 7️⃣ Connect an Azure Subscription

*Route GitHub Enterprise billing through your existing Azure EA or pay-as-you-go subscription.*

### A) Add an Azure Subscription

**Navigation:**

```
Enterprise → Settings → Billing (sidebar) → Payment information
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
| **Multiple subscriptions** | Only one Azure subscription can be linked per GitHub enterprise at a time |

---

## 8️⃣ Export Usage Reports for Finance Reconciliation

**Navigation:**

```
Enterprise → Settings → Billing (sidebar) → Usage report
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

---

## ❓ Common Questions & Troubleshooting

### Q: My cost center is showing $0 usage even though the assigned organizations are actively using GitHub. Why?
**A:** First, verify that the organizations are correctly assigned to the cost center (Enterprise > Settings > Billing > Cost centers > [Your cost center] > Manage organizations). Usage data can take 24-48 hours to appear in cost center reports after organizations are assigned. If the orgs were just added, check again the next day.

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
**A:** Cost centers operate at the organization level only -- you cannot assign individual teams or repositories to different cost centers within the same org. The recommended workaround is to create separate organizations for each billing group (e.g., `acme-engineering`, `acme-data-science`), then assign each org to its own cost center. Use internal repository visibility for cross-org collaboration.

---

### Q: How often are usage reports updated, and can I automate the export?
**A:** Usage reports in the GitHub enterprise billing dashboard are updated daily. You can manually download CSV exports from Enterprise > Settings > Billing > Usage report. For automation, use the GitHub billing REST API to programmatically retrieve usage data and feed it into your finance systems or data warehouse.

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
| **Creating cost centers** | [GitHub Docs](https://docs.github.com/en/billing/using-the-enhanced-billing-platform-for-enterprises/gathering-insights-on-your-spending#creating-cost-centers) |
| **Connecting an Azure subscription** | [GitHub Docs](https://docs.github.com/en/billing/managing-the-plan-for-your-github-account/connecting-an-azure-subscription) |

---

*Last updated: April 2026*