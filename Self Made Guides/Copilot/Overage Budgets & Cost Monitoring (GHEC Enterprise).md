# 📊 GitHub Copilot Overage Budgets & Cost Monitoring Runbook

> **Three approaches to budgeting: Enterprise-wide, By Organization, and By Cost Center (GHEC)**

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- Set overage policy: `Enterprise → AI controls → Copilot → Premium request paid usage → Enable/Disable`
- Create budget: `Enterprise → Billing and licensing → Budgets and alerts → New budget`
- Create cost center: `Enterprise → Billing and licensing → Cost centers → New cost center`
- Monitor usage: `Enterprise → Billing and licensing → Usage → Premium request analytics`
- Download report: from Usage page, click `Get usage report → Email me the report`

---

## ✅ Accuracy & Click-Path Notes

- Reviewed against current public GitHub and Microsoft documentation in April 2026 where public documentation is available. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
- When a path starts with `Enterprise`, begin at GitHub, click your profile photo, click `Your enterprises` or `Enterprise`, select the enterprise, then continue with the listed top tab or left-sidebar item.
- When a path starts with `Organization` or `Org`, begin at GitHub, click your profile photo, click `Your organizations`, select the organization, click `Settings`, then continue with the listed sidebar item.
- When a path starts with `Repository`, `Repo`, or a repository name, open the repository, click the `Settings` tab, then continue with the listed sidebar item.
- When a path starts with a vendor portal such as `Microsoft Entra admin center`, `Azure portal`, `Okta Admin Console`, `PingFederate`, `PingOne`, `OneLogin`, `AD FS Management`, `Visual Studio Admin Portal`, or `Azure DevOps`, sign in to that admin portal first, select the tenant, application, or project named in the step, then follow each listed blade, tab, button, and confirmation in order.
- If the expected button is missing, verify you are signed in with the role named in Prerequisites, the feature or license is enabled, and the object is owned by the selected enterprise, organization, or repository. Use page search only to locate the same page, not to skip required confirmation, test, save, or consent clicks.

---

## ✅ Prerequisites

| Requirement | Status |
|-------------|--------|
| GitHub Enterprise Cloud (GHEC) with enterprise billing enabled | ☐ |
| Enterprise owner or billing admin access | ☐ |
| Copilot Business or Enterprise subscription active | ☐ |
| Legacy $0 budgets reviewed and deleted/edited if present | ☐ |

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

> 💡 **Quick sanity check (Dec 2025):** Accounts created before Aug 22, 2025 may have had a default $0 Copilot premium request budget; beginning Dec 2, 2025, GitHub started removing those account-level $0 budgets for Enterprise/Team. Still: check Budgets and alerts and delete/edit any $0 budget if present.

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
7. Click **New budget**

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
6. Set alerts → Click **New budget**

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

3. Click **New cost center**

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
5. Set alerts → Click **New budget**

> ⚠️ **Re-emphasis:** Adding a cost-center budget does not override existing enterprise/org budgets.

### C3) Monitor Costs By Cost Center

**Navigation:**
```
Enterprise → Billing and licensing → Usage → Premium request analytics
```

- Group/filter by cost center
- Optional export: **Get usage report** → **Email me the report**

## 🧯 Known Errors & Resolutions

> This section lists the known product errors and admin-facing symptoms that commonly occur with this workflow. Exact message text can vary by product rollout, tenant policy, and provider, so use the log or settings page named in the resolution to confirm the root cause.

| Error or symptom | Likely cause | Resolution |
|------------------|--------------|------------|
| **Page, tab, or button is missing** | Wrong account context, missing admin role, unavailable plan/add-on, or feature rollout not enabled for the selected enterprise/org/repo. | Switch to the correct account and scope, confirm the prerequisite role, verify licensing or add-on activation, then refresh the page. If the control is still absent, use the direct settings URL from the relevant GitHub Docs page and confirm the feature is available for your plan. |
| **Changes appear saved but behavior does not change** | Policy inheritance, cached UI state, propagation delay, or an overlapping enterprise/org/repo policy. | Reopen the settings page, verify the effective policy at the lowest affected scope, wait for propagation where documented, and check for a stricter policy at an enterprise or organization level. |
| **403, forbidden, or resource not accessible** | The signed-in user or token can see the page but lacks the specific permission for the action. | Use an enterprise owner, organization owner, repository admin, or token with the exact scopes/permissions listed in the runbook. For SAML-protected orgs, authorize the token or SSH key for SSO before retrying. |
| **Copilot feature, model, or policy is not visible** | Plan, license assignment, enterprise policy, org delegation, or feature rollout does not permit it. | Check enterprise AI controls, organization Copilot settings, assigned seat status, and the plan requirements for the feature. |
| **Premium requests are rejected after the included allowance** | Paid usage is disabled, no billing entity is selected, or a stop-usage budget is exhausted. | Enable Premium request paid usage where appropriate, set or delete conflicting budgets, and have users with multiple licenses choose a billing entity. |
| **Content exclusions do not apply immediately** | Client policy cache, unsupported surface/mode, symlink/remote filesystem limitation, or indirect IDE context. | Reload the IDE policy, verify the exclusion syntax at enterprise/org/repo scope, and document surfaces where exclusions are limited. |
| **Usage metrics look empty or inconsistent** | Telemetry is disabled, data freshness delay applies, users are unlicensed, or different APIs report different scopes. | Enable the metrics policy, confirm seats and telemetry, wait for data freshness, and avoid comparing dashboards/API endpoints as if they share identical data models. |
| **Coding agent or MCP action is denied** | Agent policy, MCP policy, repository permissions, secrets, or server allowlist does not permit the operation. | Review Enterprise AI controls > Agents/MCP, repo-level permissions, MCP server configuration, and audit logs for the denied action. |

---

## ❓ Common Questions & Troubleshooting

### Q: I created a new budget but users are still being blocked — why?
**A:** Creating a new budget does not override existing budgets. If any applicable budget with "Stop usage when budget limit is reached" enabled is exhausted, premium requests are blocked. Check the Budgets and alerts page for all active budgets and delete or edit any conflicting ones (including legacy $0 budgets).

---

### Q: Budget alert emails are not firing — what should I check?
**A:** Verify that alert recipients are configured on the budget (email addresses must be set during budget creation). Check that the alert thresholds (e.g., 75%, 90%, 100%) are configured. Also confirm that the budget scope matches the usage you expect — an org-scoped budget will not alert on usage from a different org.

---

### Q: How do I know if my enterprise has a legacy $0 Copilot premium request budget?
**A:** Navigate to Enterprise > Billing and licensing > Budgets and alerts. Look for any budget with a $0 amount and "Stop usage when budget limit is reached" enabled that covers premium requests. Accounts created before Aug 22, 2025 may have had one auto-created. Delete or edit it if present.

---

### Q: Can I change the scope of an existing budget (e.g., from enterprise-wide to org-level)?
**A:** No, you cannot change the scope of a budget after it is created. You must create a new budget with the desired scope and then delete the old one.

---

### Q: One heavy-consuming org burned through the enterprise-wide budget and blocked everyone — how do we prevent this?
**A:** Switch from a single enterprise-wide budget to per-org or per-cost-center budgets. This isolates spend so one team's heavy usage does not block other teams. See Guide B (by Organization) or Guide C (by Cost Center) in this runbook.

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Premium Request Budget & Overage Planning | `Copilot/Premium Request Budget & Overage Planning.md` |
| Premium Request Budgeting Scenarios | `Copilot/Premium Request Budgeting Scenarios.md` |
| Cost Centers & Department Billing | `Billing/Cost Centers & Department Billing.md` |
| Power User Premium Allowance (Cost Centers + Budgets) | `Copilot/Power User Premium Allowance (Cost Centers + Budgets).md` |

---

## 📝 Resources

| Resource | Link |
|----------|------|
| Managing the premium request allowance | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/manage-and-track-spending/manage-request-allowances) |
| Setting up budgets to control spending | [GitHub Docs](https://docs.github.com/enterprise-cloud@latest/billing/tutorials/set-up-budgets) |
| Using cost centers to allocate costs | [GitHub Docs](https://docs.github.com/enterprise-cloud@latest/billing/tutorials/use-cost-centers) |
| Controlling and tracking costs at scale | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/billing/tutorials/control-costs-at-scale) |

---

*Last updated: April 2026*
