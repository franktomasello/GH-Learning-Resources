# 💰 GitHub Copilot Premium Request Budgeting Scenarios Guide

> **A guide to controlling Copilot premium request overage spend in GitHub Enterprise Cloud**

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [📋 Overview](#-overview)
- [🚨 What Actually Controls Overages (Don't Skip)](#-what-actually-controls-overages-dont-skip)
- [📊 Scenario 1: Selective Onboarding to Copilot Enterprise](#-scenario-1-selective-onboarding-to-copilot-enterprise)
- [🎛️ Scenario 2: Separate Spending Budgets for Specific Users](#-scenario-2-separate-spending-budgets-for-specific-users)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- Download usage report: `Enterprise → Billing and licensing → Usage → Copilot premium requests usage report`
- Check overage policy: `Enterprise → AI controls → Copilot → Premium request paid usage`
- Review/clean budgets: `Enterprise → Billing and licensing → Budgets and alerts`
- Create org-scoped budget: `Budgets and alerts → New budget → Scope: Organization`
- Create cost center budget: `Billing and licensing → Cost centers → New cost center` then scope a budget to it

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
| GitHub Enterprise Cloud (GHEC) with enterprise billing enabled | ☐ |
| Enterprise owner or billing admin access | ☐ |
| Copilot Business or Enterprise subscription active | ☐ |
| Copilot premium requests usage report downloaded for analysis | ☐ |
| Legacy $0 budgets reviewed and deleted/edited if present | ☐ |

---

## 📋 Overview

This guide covers two common scenarios for controlling Copilot premium request overage spend:

| Scenario | Description |
|----------|-------------|
| **Scenario 1** | Put heavy premium-request users on Copilot Enterprise (often cheaper at high usage) |
| **Scenario 2** | Give some users a higher overage budget while keeping everyone else restricted |

---

## 🚨 What Actually Controls Overages (Don't Skip)

### 1) Premium Request Paid Usage Policy

For organizations/enterprises, the **"Premium request paid usage"** policy determines whether users can go beyond their included monthly allowance and incur overage charges (it's enabled by default).

### 2) Premium Request Budgets

Budgets can monitor or block overages. If any applicable budget has **"Stop usage when budget limit is reached"** enabled and becomes exhausted, then additional premium requests are blocked. Budgets also do not override each other automatically.

### 3) Legacy "$0 Copilot Premium Request Budget"

> ⚠️ **Historical Context:**
> - Accounts created before Aug 22, 2025 had a default $0 budget for Copilot premium requests that would reject overage requests unless you edited/deleted it
> - Beginning Dec 2, 2025, GitHub began removing account-level $0 Copilot premium request budgets for Enterprise/Team accounts created before Aug 22, 2025

**So:** Do not assume a $0 budget exists—check your Budgets and alerts page first.

---

## 📊 Scenario 1: Selective Onboarding to Copilot Enterprise

**Situation:** You want to move a subset of users to Copilot Enterprise because it may be more cost-effective for heavy premium-request users.

### Verified Plan Entitlements & Pricing

| Plan | Included Requests | Seat Cost | Overage Rate |
|------|-------------------|-----------|--------------|
| **Copilot Business** | 300/user/month | $19/month | $0.04/request |
| **Copilot Enterprise** | 1,000/user/month | $39/month | $0.04/request |

### Cost Comparison (Math-Checked)

| Monthly Premium Requests | Copilot Business Total | Copilot Enterprise Total | Savings (CE vs CB) |
|--------------------------|------------------------|--------------------------|-------------------|
| 300 | $19 + $0 = **$19** | $39 + $0 = **$39** | **-$20** |
| 500 | $19 + $8 = **$27** | $39 + $0 = **$39** | **-$12** |
| 800 | $19 + $20 = **$39** | $39 + $0 = **$39** | **$0** |
| 1,000 | $19 + $28 = **$47** | $39 + $0 = **$39** | **$8** |
| 2,000 | $19 + $68 = **$87** | $39 + $40 = **$79** | **$8** |

> 💡 **Key Rule:** GitHub's own guidance notes that Business users making **>800 premium requests/month** would save money on Copilot Enterprise.

### Implementation Steps

1. Download the **"Copilot premium requests usage report"** from Enterprise → Billing and licensing → Usage
2. Aggregate the report to find users near/over 800 premium requests/month
3. Create a new organization in your enterprise
4. Add the high-usage users to that new org
5. Grant Copilot Enterprise licenses to all users in that new org (including upgrading the enterprise/org to Copilot Enterprise if needed)
6. Re-check usage regularly to ensure the move remains cost-effective

---

## 🎛️ Scenario 2: Separate Spending Budgets for Specific Users

**Situation:** You want some users to be able to incur paid overages, while keeping everyone else restricted.

### Two Supported Approaches

| Approach | Best For |
|----------|----------|
| **Separate Organization + budget** | Scoped to that org |
| **Cost Center + budget** | Scoped to that cost center |

> ⚠️ **Non-negotiable requirement:** Every Copilot user must be covered by some budget, otherwise uncovered users can have unlimited spending on premium requests.

> ⚠️ **Budget interaction gotcha:** Creating a new budget does not override an existing one; "Stop usage…" on any applicable budget can block overages once exhausted.

---

### Option 1: Use a Separate Organization Budget

#### Steps

1. Download the Copilot premium requests usage report
2. Identify the users who need paid overages
3. Ensure **Premium request paid usage policy** is configured as intended (enabled if you want paid overages)
4. Check **Budgets and alerts** and remove/edit any legacy $0 Copilot premium request budget if present
5. Create a new org for "allowed overage" users and add them
6. Create a premium requests budget scoped to that new org:

| Setting | Value |
|---------|-------|
| **Budget Type** | Bundled premium requests budget (recommended) or SKU-level |
| **Scope** | New organization |
| **Amount** | Your monthly $ limit |
| **Stop usage when budget limit is reached** | Enable for hard enforcement |

7. Create a separate restrictive budget that covers all other Copilot users

---

### Option 2: Use a Cost Center Budget

#### Steps

1. Download the Copilot premium requests usage report
2. Ensure the **Premium request paid usage policy** is configured as intended
3. Create a Cost Center for "allowed overage" users and assign those users to it (UI and API are supported)
4. Create a premium requests budget scoped to that cost center (Bundled premium requests budget or SKU-level)
5. Create a second budget that covers everyone else

> 📝 **API Note:** Adding users to cost centers is supported via the enterprise billing REST API, but the endpoint does not work with fine-grained PATs (and some GitHub App token types).

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
| **Premium requests are rejected after the included allowance** | Paid usage is disabled, no billing entity is selected, or a stop-usage budget is exhausted. | Enable Premium request paid usage where appropriate, set or delete conflicting budgets, and have users with multiple licenses choose a billing entity. |
| **Content exclusions do not apply immediately** | Client policy cache, unsupported surface/mode, symlink/remote filesystem limitation, or indirect IDE context. | Reload the IDE policy, verify the exclusion syntax at enterprise/org/repo scope, and document surfaces where exclusions are limited. |
| **Usage metrics look empty or inconsistent** | Telemetry is disabled, data freshness delay applies, users are unlicensed, or different APIs report different scopes. | Enable the metrics policy, confirm seats and telemetry, wait for data freshness, and avoid comparing dashboards/API endpoints as if they share identical data models. |
| **Coding agent or MCP action is denied** | Agent policy, MCP policy, repository permissions, secrets, or server allowlist does not permit the operation. | Review Enterprise AI controls > Agents/MCP, repo-level permissions, MCP server configuration, and audit logs for the denied action. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>


### Q: We moved users to Copilot Enterprise but costs went up instead of down — what happened?
**A:** The cost savings from Copilot Enterprise only apply when users consume more than approximately 800 premium requests per month. If the moved users are not heavy premium request consumers, the higher seat cost ($39 vs $19) will outweigh the savings from the larger included allowance. Re-check per-user consumption in the usage report.

---

### Q: We created a budget for the "allowed overage" org but users in other orgs can still incur overages — why?
**A:** Every Copilot user must be covered by some budget, otherwise uncovered users can have unlimited premium request spending. Create a separate restrictive budget (e.g., $0 with "Stop usage when budget limit is reached") that covers all other orgs or users.

---

### Q: Budget interaction is confusing — a new budget did not override the old one. How do budgets interact?
**A:** Budgets never override each other. They are additive. If any applicable budget with "Stop usage when budget limit is reached" is exhausted, premium requests are blocked — even if another budget would still allow spend. Always review all active budgets on the Budgets and alerts page when troubleshooting.

---

### Q: How do we identify which users should be moved to Copilot Enterprise?
**A:** Download the "Copilot premium requests usage report" from Enterprise > Billing and licensing > Usage. Aggregate by user and look for users consistently near or over 800 premium requests per month. These users will save money on Enterprise vs Business with overages.

---

### Q: Can we use cost center budgets and org budgets together?
**A:** Yes, but be careful. If a user is covered by both an org-level budget and a cost-center budget, both apply. If either budget is exhausted and has "Stop usage when budget limit is reached" enabled, the user is blocked. Plan your budget structure to avoid overlapping stop-usage budgets.

</details>

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Premium Request Budget & Overage Planning | `Copilot/Premium Request Budget & Overage Planning.md` |
| Overage Budgets & Cost Monitoring (GHEC Enterprise) | `Copilot/Overage Budgets & Cost Monitoring (GHEC Enterprise).md` |
| Seat Assignment & Enablement | `Copilot/Seat Assignment & Enablement.md` |

---

## 📝 Resources

| Resource | Link |
|----------|------|
| Managing the premium request allowance | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/manage-and-track-spending/manage-request-allowances) |
| Setting up budgets to control spending | [GitHub Docs](https://docs.github.com/enterprise-cloud@latest/billing/tutorials/set-up-budgets) |
| Choosing your enterprise's plan for GitHub Copilot | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/copilot/get-started/choose-enterprise-plan) |
| Using cost centers to allocate costs | [GitHub Docs](https://docs.github.com/enterprise-cloud@latest/billing/tutorials/use-cost-centers) |

---

*Last updated: April 2026*
