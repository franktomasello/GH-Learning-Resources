# 🎯 Runbook: Let Only a "Hyper Power User" Group Exceed Copilot Premium Request Allowance

> **Using Cost Centers + Budgets to control who can incur premium request overages**

---

## 📋 Goal

Allow only a specific subset of users ("hyper power users") to use paid premium requests beyond their included allowance, while everyone else is blocked from overages.

> 💡 **Why this matters:** This solves the exact failure mode where org-level $0 budgets with "Stop usage…" enabled block overages for everyone in that org, even if you try to allow spend for a smaller group elsewhere. GitHub explicitly calls out that existing budgets don't get overridden, and conflicting "stop usage" budgets can block premium requests.

---

## ✅ Before You Start

Confirm these items before proceeding:

| Requirement | Status |
|-------------|--------|
| You're on GitHub Enterprise Cloud (GHEC) | ☐ Confirmed |
| You know the list of "hyper power users" | ☐ Confirmed |
| You understand: every user must be covered by a budget to prevent unlimited overages | ☐ Confirmed |

---

## 🏗️ Recommended Design

*This is the cleanest and easiest setup to operate.* Cost centers allocate premium request spending based on users (which is exactly what you need for a user subset).

| Cost Center A: "Default Users" | Cost Center B: "Hyper Power Users" |
|-------------------------------|-----------------------------------|
| **Budget:** $0 | **Budget:** $X (monthly cap you choose) |
| **Stop usage when limit reached:** ON | **Stop usage:** ON (hard cap) or OFF (alert-only) |
| *→ Blocks paid overages* | *→ Allows overages up to cap* |

---

## 1️⃣ Step 1 — Enable the "Premium Request Paid Usage" Policy

> ⚠️ **Important:** If this policy is disabled, no one can exceed their included allowance (budgets won't help).

### Navigation

```
Profile Picture → Enterprise → AI controls → Copilot (sidebar) → Premium request paid usage
```

### Detailed Steps

1. Top-right on GitHub → click your profile picture
2. Click **Enterprise** (or Enterprises → select the enterprise)
3. Top menu → **AI controls**
4. Left sidebar → **Copilot**
5. Find **Premium request paid usage** → set to **Enabled**
6. *Optional:* choose "Enabled for specific products" if you only want overages for certain AI tools

---

## 2️⃣ Step 2 — Remove/Adjust Blocking $0 Budgets

> 🔴 **This is the part that's breaking your customer today.**

GitHub's guidance:
- **Edit or delete budgets** that stop usage for the Premium Request SKU if you want paid usage to work
- **Avoid overlapping/conflicting budgets** because users can be "unexpectedly blocked"

### Navigation

```
Profile Picture → Enterprise → Billing & Licensing → Budgets and alerts (sidebar)
```

### What to Look For

Review budgets for:
- **Copilot premium requests** and/or **Bundled premium requests**
- **Scope:** Enterprise and Organization

### Action Required

For each budget that is $0 AND has "Stop usage when budget limit is reached" enabled:

1. Click the budget's **⋯ menu** → **Edit** or **Delete**
2. If you want to keep it for monitoring, **Edit** and turn "Stop usage…" **OFF** (monitor-only)

> ⚠️ **Important:** You cannot change the scope of a budget after creating it — so if you need a cost-center-scoped budget, create a new one.

---

## 3️⃣ Step 3 — Create Two Cost Centers and Assign Users

### Navigation

```
Enterprise → Billing & Licensing → Cost centers (sidebar) → New cost center
```

### Create the Cost Centers

1. Click **New cost center**
2. Create cost center: **Default users**
3. Create cost center: **Hyper power users**

### Assign Users

| Cost Center | Members |
|-------------|---------|
| **Default users** | All non-hyper users |
| **Hyper power users** | Only the hyper list |

> 💡 **How cost centers work:** Cost centers work by assigning resources (including users) to the cost center.

---

## 4️⃣ Step 4 — Create Cost-Center-Scoped Budgets

**This is what enforces "only hyper can overspend."**

> 💡 **Recommendation:** GitHub recommends a Bundled premium requests budget for most customers (covers premium requests across tools).

### 4A) Default Users Budget (Block Overages)

**Navigation:**
```
Enterprise → Billing & Licensing → Budgets and alerts → New budget
```

**Configuration:**

| Setting | Value |
|---------|-------|
| **Budget Type** | Bundled premium requests budget (recommended) |
| **Budget scope** | Cost center → **Default users** |
| **Budget** | **$0** |
| **Stop usage when budget limit is reached** | ✅ **ON** |
| **Threshold alerts** | *(Optional)* |

Click **Create budget**

---

### 4B) Hyper Power Users Budget (Allow Overages)

Repeat the same steps with these differences:

| Setting | Value |
|---------|-------|
| **Scope** | Cost center → **Hyper power users** |
| **Budget** | **$X** (your monthly cap) |
| **Stop usage** | **ON** (hard cap) or **OFF** (alert-only) |

---

## 5️⃣ Step 5 — Verify It's Working

### Verify Budgets Aren't Conflicting

**Navigation:**
```
Enterprise → Billing & Licensing → Budgets and alerts
```

**Confirm:**
- ✅ No leftover org/enterprise $0 stop-usage budgets for premium requests that apply to everyone
- ✅ Not accidentally using both "Copilot premium requests" and "Bundled premium requests" budgets in overlapping ways

### Verify Attribution by Cost Center

To check whether usage is landing in the right bucket, use the **Usage views** and/or a **detailed usage report** (look for cost center columns like `cost_center_name`).

---

## 💡 Optional: Use a Second Org for Power Users

If the customer's "Enterprise org" exists to raise the included allowance (e.g., Business → Enterprise), GitHub explicitly documents this approach: create a new org, add users, grant Copilot Enterprise to that org.

> 💡 **Use case:** This helps "power users" hit the overage threshold less often, but it does not replace the budget/cost-center setup for "hyper users who must exceed allowance."

---

## 📝 Notes for Accounts with Legacy $0 Budgets

GitHub notes that accounts created **before Aug 22, 2025** may have an auto-created $0 Copilot premium request budget that rejects overages unless edited/deleted, and that these default $0 budgets are being removed **beginning Dec 2, 2025**.

---

*Last updated: December 2025*