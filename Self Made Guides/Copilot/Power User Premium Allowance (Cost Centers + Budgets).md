# 🎯 Runbook: Let Only a "Hyper Power User" Group Exceed Copilot Premium Request Allowance (GHEC)

> **Use the "Premium request paid usage" policy + cost-center budgets to allow overages for only a subset of users, while blocking everyone else.**

---

## 📋 Overview

Allow only a specific subset of users (“hyper power users”) to use paid premium requests beyond their included allowance, while everyone else is blocked from overages using budgets that stop usage at the limit.

Key mechanics to keep in mind:
- Budgets don't override each other. If any applicable budget with "Stop usage when budget limit is reached" is exhausted, premium requests are blocked, even if another budget "should allow" spend [[1]](#source-1).
- Every user must be covered by a budget or they may have unlimited premium-request overage spend [[1]](#source-1).

---

## ✅ Before You Start (Confirm)
- You are on GitHub Enterprise Cloud (GHEC).
- You have the list of hyper power users (GitHub usernames / managed users).
- You understand budget structure:
  - Premium requests are metered, so budgets can optionally stop usage [[1]](#source-1).
  - You can use either:
    - Bundled premium requests budget (recommended for most orgs) [[2]](#source-2)
    - SKU-level budgets (Copilot vs Spark vs Copilot coding agent) [[2]](#source-2)

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
- Click Create budget [[2]](#source-2)

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

---

## ❓ Common Questions & Troubleshooting

### Q: I assigned users to the "Hyper power users" cost center but they are still blocked from overages — why?
**A:** Check for conflicting budgets. If an enterprise-wide or org-level budget with "Stop usage when budget limit is reached" is exhausted, it will block those users even though they have a separate cost center budget. Review all budgets on the Budgets & alerts page and remove or edit any that conflict.

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

*Last updated: December 2025*
