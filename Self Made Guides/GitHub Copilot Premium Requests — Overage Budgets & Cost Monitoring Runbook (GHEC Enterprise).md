# 📊 GitHub Copilot Premium Requests — Overage Budgets & Cost Monitoring Runbook

> **Three approaches to budgeting: Enterprise-wide, By Organization, and By Cost Center (GHEC)**

---

## 📋 Overview

This runbook covers three approaches for setting up premium request budgets:

| Guide | Scope | Best For |
|-------|-------|----------|
| **Guide A** | Enterprise-wide | Single global cap across the whole company |
| **Guide B** | By Organization | Cap per org/team/product line |
| **Guide C** | By Cost Center | Chargeback and BU ownership |

> 📝 **Scope note:** This runbook is for GitHub Enterprise Cloud (enterprise billing + Copilot Business/Enterprise).

---

## 🚨 Critical Warning (Read This First)

**Creating new budgets without deleting or editing existing budgets does not override them.** If any applicable budget with "Stop usage when budget limit is reached" enabled is exhausted, additional premium requests are blocked.

---

## 🤔 Why Budget at Each Level?

| Level | Best When... | Tradeoff |
|-------|--------------|----------|
| **Enterprise-wide** | Single guardrail on total spend—simple for Finance and early rollout | One heavy-consuming org can burn the shared cap and block everyone |
| **Org-level** | Orgs map to different teams/product lines, phased rollout, or preventing cross-org impact | More budgets to manage |
| **Cost-center** | Budgets aligned to financial entities (BU/department/project), including cross-org scenarios | Requires cost center setup |

---

## 0️⃣ One-Time Prerequisite: Allow (or Block) Overages via Policy

The "Premium request paid usage" policy is the gate for whether users can go past their included allowance (and incur overage charges).

> 💡 **Quick sanity check (Dec 2025):** Accounts created before Aug 22, 2025 may have had a default $0 Copilot premium request budget; beginning Dec 2, 2025, GitHub started removing those account-level $0 budgets for Enterprise/Team. Still: check Budgets & alerts and delete/edit any $0 budget if present.

### Step 0.1 — Open Copilot Policy Controls

**Navigation:**
```
Profile Picture → Enterprise → AI controls → Copilot (sidebar)
```

### Step 0.2 — Set Premium Request Paid Usage

| Option | Effect |
|--------|--------|
| **Enabled** | Allow overages, subject to budgets |
| **Disabled** | Block overages entirely |

> 💡 **Tip:** If your goal is "allow overages but cap spend," use **Enabled** + a budget with **Stop usage when budget limit is reached**.

---

## 📘 Guide A — Enterprise-Wide Overage Budget

*Single cap for the whole enterprise*

### A1) Navigate to Budgets and Alerts

**Navigation:**
```
Profile Picture → Enterprise → Billing and licensing → Budgets and alerts
```

### A2) Create the Enterprise-Wide Premium Request Budget

1. Click **New budget**
2. **Budget type:** Choose one:
   - **Bundled premium requests budget** (recommended for most)
   - **Individual/SKU-level budgets** (separate budgets per AI tool/SKU)
3. **Budget scope:** Select **Whole enterprise**
4. Set the **Budget** ($ amount)
5. Enable **Stop usage when budget limit is reached** (hard cap)
6. Configure alerts (e.g., 75/90/100%) + recipients
7. Click **Create budget**

> ⚠️ **Re-emphasis:** A new enterprise-wide budget does not cancel older applicable budgets.

### A3) Monitor Costs (Enterprise-Wide)

#### Premium Request Analytics

**Navigation:**
```
Enterprise → Billing and licensing → Usage → Premium request analytics
```

#### Download a Cost/Usage Report (CSV)

1. From **Metered Usage** or **Premium request analytics**, click **Get usage report**
2. Specify report details
3. Click **Email me the report**

---

## 📗 Guide B — Budget By Organization

*Cap premium request spend per organization*

### B1) Navigate to Budgets and Alerts

**Navigation:**
```
Enterprise → Billing and licensing → Budgets and alerts
```

### B2) Create an Org-Scoped Premium Request Budget

1. Click **New budget**
2. **Budget type:** Bundled premium requests or Individual/SKU-level
3. **Budget scope:** Select **Organization** → choose the org
4. Set **Budget** ($)
5. Enable **Stop usage when budget limit is reached**
6. Set alerts → Click **Create budget**

> ⚠️ **Re-emphasis:** Creating a new org budget does not override an existing enterprise/org/cost-center budget.

### B3) Monitor Costs By Org

**Navigation:**
```
Enterprise → Billing and licensing → Usage → Premium request analytics
```

Use filters/grouping for org-level breakdowns.

---

## 📙 Guide C — Budget By Cost Center

*Best for chargeback + BU ownership*

Cost centers let you attribute spend to BUs and apply budgets to that grouping.

### C1) Create a Cost Center

**Navigation:**
```
Profile Picture → Enterprise → Billing and licensing → Cost centers → New cost center
```

**Configuration:**
1. Enter **Name**
2. Under **Resources**, add organizations, repositories, and/or users

> 📝 **Note:** A resource (org/repo/user) can only be assigned to one cost center at a time; adding it elsewhere moves it.

3. Click **Create cost center**

### C2) Create a Premium Request Budget for the Cost Center

**Navigation:**
```
Enterprise → Billing and licensing → Budgets and alerts → New budget
```

**Configuration:**
1. **Budget type:** Bundled premium requests or Individual/SKU-level
2. **Budget scope:** Select **Cost center** → choose the cost center
3. Set **Budget** ($)
4. Enable **Stop usage when budget limit is reached**
5. Set alerts → Click **Create budget**

> ⚠️ **Re-emphasis:** Adding a cost-center budget does not override existing enterprise/org budgets.

### C3) Monitor Costs By Cost Center

**Navigation:**
```
Enterprise → Billing and licensing → Usage → Premium request analytics
```

- Group/filter by cost center
- Optional export: **Get usage report** → **Email me the report**

---

*Last updated: December 2025*