# 💰 GitHub Copilot Premium Request Budgeting Scenarios

> **A guide to controlling Copilot premium request overage spend in GitHub Enterprise Cloud**

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

**So:** Do not assume a $0 budget exists—check your Budgets & alerts page first.

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

1. Download the **"Copilot premium requests usage report"** from Enterprise → Billing & licensing → Usage
2. Aggregate the report to find users near/over 800 premium requests/month
3. Create a new organization in your enterprise
4. Add the high-usage users to that new org
5. Grant Copilot Enterprise licenses to all users in that new org (including upgrading the enterprise/org to Copilot Enterprise if needed)
6. Re-check usage regularly to ensure the move remains cost-effective

---

## 🎛️ Scenario 2: Separate Spending Limits for Specific Users

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
4. Check **Budgets & alerts** and remove/edit any legacy $0 Copilot premium request budget if present
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

---

*Last updated: December 2025*
