# 💰 GitHub Copilot Premium Request Budget & Overage Planning Runbook

> **Complete guide to understanding, budgeting, and controlling GitHub Copilot premium request consumption and overages**

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- Enable/disable overages: `Enterprise → AI controls → Copilot → Policies → Premium request overages`
- Set spending budgets: `Enterprise → Billing and licensing → Budgets and alerts → New budget → Set monthly limit`
- Create cost centers: `Enterprise → Billing and licensing → Cost centers → New cost center`
- Control models: `Enterprise → AI controls → Copilot → Models` — disable high-multiplier models
- Monitor consumption: `Enterprise → Billing and licensing → Usage → Filter by Copilot`

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
| GitHub Enterprise Cloud account with enterprise owner access | ☐ |
| Copilot Business (300 requests/user/month) or Enterprise (1,000 requests/user/month) active | ☐ |
| Enterprise billing admin access (for budgets and cost centers) | ☐ |
| Understanding of model multipliers for budget projections | ☐ |

---

## 📋 Overview

GitHub Copilot includes a monthly allowance of premium requests per user. When users interact with premium models that are not included in the base allowance, each interaction consumes premium requests at a model-specific multiplier rate. Once the allowance is exhausted, overages are billed per request.

| Plan | Monthly Premium Requests per User |
|------|----------------------------------|
| **Copilot Business** | 300 |
| **Copilot Enterprise** | 1,000 |

---

## 1️⃣ Included Models (No Premium Request Cost)

*These models do not consume premium requests — they are included in the base subscription*

| Model | Premium Cost |
|-------|-------------|
| **GPT-5 mini** | None (included) |
| **GPT-4.1** | None (included) |
| **GPT-4o** | None (included) |

> 💡 **Tip:** If your goal is to minimize premium request consumption, encourage users to default to these included models for everyday tasks.

---

## 2️⃣ Overage Rate

| Metric | Value |
|--------|-------|
| **Cost per premium request over allowance** | $0.04 |

> ⚠️ **Important:** Overages are billed at the enterprise level. Without spending controls, a large user base using premium models can generate significant unexpected costs.

---

## 3️⃣ Model Multiplier Examples

*Each premium model interaction consumes a different number of premium requests based on its multiplier*

| Model | Multiplier | Premium Requests per Interaction |
|-------|-----------|--------------------------------|
| **Claude Sonnet 4.6** | 1x | 1 |
| **GPT-5.4** | 1x | 1 |
| **Claude Opus 4.6** | 3x | 3 |
| **Gemini 3 Flash** | 0.33x | 0.33 |
| **Grok Code Fast 1** | 0.25x | 0.25 |

> 💡 **Tip:** A single interaction with Claude Opus 4.6 (3x) consumes the same number of premium requests as three interactions with Claude Sonnet 4.6 (1x). Model choice has a significant impact on budget.

---

## 4️⃣ Budget Planning Formula

*Use this formula to estimate monthly premium request consumption and overage costs*

### Step 1: Calculate Monthly Consumption

```
Monthly Consumption = Sum of (interactions per model × model multiplier)
```

### Step 2: Calculate Overage

```
Overage Requests = max(0, Monthly Consumption - Monthly Allowance)
Overage Cost     = Overage Requests × $0.04
```

### Example Scenario

| Input | Value |
|-------|-------|
| **Plan** | Copilot Enterprise (1,000 premium requests/user/month) |
| **User count** | 50 |
| **Avg interactions with Claude Sonnet 4.6 (1x)** | 400 per user |
| **Avg interactions with Claude Opus 4.6 (3x)** | 100 per user |

| Calculation | Result |
|-------------|--------|
| Sonnet consumption | 400 x 1 = 400 |
| Opus consumption | 100 x 3 = 300 |
| Total per user | 700 premium requests |
| Overage per user | max(0, 700 - 1,000) = 0 |
| **Monthly overage cost** | **$0** |

> 💡 **Tip:** In this example, usage is within the allowance. If each user added 200 more Opus interactions: total = 400 + (300 x 3) = 1,300, overage = 300 x $0.04 = **$12/user/month**.

---

## 5️⃣ Enterprise Controls

### A) Enable or Disable Overages

**Navigation:**

```
Enterprise → AI controls → Copilot → Policies
  → Premium request overages → Enable / Disable
```

| Setting | Effect |
|---------|--------|
| **Overages enabled** | Users can continue using premium models after exhausting their allowance; enterprise is billed for overages |
| **Overages disabled** | Users are throttled or fall back to included models once the allowance is reached |

> ⚠️ **Warning:** If overages are enabled without budgets and alerts, there is no cap on how much can be billed. Always pair overages with budgets.

---

### B) Control Available Models

**Navigation:**

```
Enterprise → AI controls → Copilot → Models
  → Enable or disable specific models
```

> 💡 **Tip:** Disable high-multiplier models (e.g., 3x models) enterprise-wide if budget is a concern, then selectively enable them for teams that need them.

---

### C) Set Budgets and Hard Stops

**Navigation:**

```
Enterprise → Billing and licensing → Budgets and alerts
  → New budget → Set monthly limit
    → Assign to cost center or enterprise-wide
      → Configure alert thresholds (50%, 75%, 100%)
```

---

### D) Create Cost Centers for Department Allocation

**Navigation:**

```
Enterprise → Billing and licensing → Cost centers
  → New cost center → Name (e.g., "Engineering", "Data Science")
    → Assign organizations or specific users
```

| Cost Center | Assigned Users | Budget |
|-------------|---------------|--------|
| Engineering | platform-eng org | $2,000/month |
| Data Science | data-science org | $5,000/month |
| Copilot Pilot | pilot-group users | $500/month |

---

## 6️⃣ Best-Practice Rollout

*A phased approach to introducing premium models without budget surprises*

| Phase | Action | Duration |
|-------|--------|----------|
| **1. Pilot** | Enable premium models for a small cohort (10-20 users) | 2-4 weeks |
| **2. Isolate costs** | Create a dedicated cost center for the pilot group | At pilot start |
| **3. Set budgets** | Apply SKU-level budgets and alert thresholds to the pilot cost center | At pilot start |
| **4. Monitor** | Review per-user and per-model consumption in billing reports | Weekly during pilot |
| **5. Evaluate** | Compare productivity gains against cost; adjust model availability | End of pilot |
| **6. Expand** | Gradually roll out to broader teams with established budgets and guardrails | After evaluation |

> 💡 **Tip:** Start with overages **disabled** during the pilot. Once you understand consumption patterns, enable overages with budgets for the broader rollout.

---

## 7️⃣ Monitoring Consumption

**Navigation:**

```
Enterprise → Billing and licensing → Usage
  → Filter by: Copilot → View per-user, per-model consumption
```

| Report View | What It Shows |
|-------------|--------------|
| **Per-user** | Total premium requests consumed by each user |
| **Per-model** | Which models are driving the most consumption |
| **Per-cost-center** | Consumption and cost allocated to each department |
| **Trend** | Month-over-month usage growth |

> 💡 **Tip:** Review usage reports weekly during the first month of any rollout expansion. Usage patterns stabilize after 2-4 weeks, after which monthly reviews are sufficient.

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

### Q: Users hit the 300 premium request limit mid-month — what happens?
**A:** If overages are disabled, users fall back to the included base models (GPT-4o, GPT-4.1, GPT-5 mini) and cannot use premium models until the next billing cycle. If overages are enabled, they can continue using premium models and the enterprise is billed at $0.04 per additional premium request.

---

### Q: How do we see which users are consuming the most premium requests?
**A:** Navigate to Enterprise > Billing and licensing > Usage and filter by Copilot. This shows per-user consumption of premium requests. You can also download a CSV usage report for deeper analysis.

---

### Q: Can we set a hard spending cap on premium request overages?
**A:** Yes. Navigate to Enterprise > Billing and licensing > Budgets and alerts, create a budget scoped to premium requests, set a dollar amount, and enable "Stop usage when budget limit is reached." Configure alert thresholds (e.g., 50%, 75%, 100%) to get notifications before hitting the cap.

---

### Q: The multipliers changed since we did our cost estimate — what should we do?
**A:** Model multipliers can change over time. Always check the current values on the GitHub supported models documentation page before finalizing budget estimates. Recalculate your projections whenever multipliers are updated and adjust budgets accordingly.

---

### Q: Do premium requests reset monthly? Do unused requests roll over?
**A:** Yes, premium requests reset on each billing cycle. Unused requests do not roll over to the next month. Each user starts fresh with their full allowance (300 for Business, 1,000 for Enterprise) at the beginning of each billing period.

---

### Q: We enabled overages but users are still being blocked — why?
**A:** Check for existing budgets under Enterprise > Billing and licensing > Budgets and alerts. If any applicable budget with "Stop usage when budget limit is reached" is exhausted (including legacy $0 budgets), premium requests will be blocked even if the overage policy is enabled. Delete or increase any blocking budgets.

---

### Q: How do we estimate costs before enabling premium models for the whole enterprise?
**A:** Run a pilot with 10-20 users for 2-4 weeks with overages disabled. Review per-user and per-model consumption in the billing reports, then multiply by your total user count. Use the budget planning formula in this guide to project monthly costs before expanding.

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Seat Assignment & Enablement | `Copilot/Seat Assignment & Enablement.md` |
| Admin Controls (Models, Content Exclusion, Instructions) | `Copilot/Admin Controls (Models, Content Exclusion, Instructions).md` |
| Cost Centers & Department Billing | `Billing/Cost Centers & Department Billing.md` |
| Overage Budgets & Cost Monitoring (GHEC Enterprise) | `Copilot/Overage Budgets & Cost Monitoring (GHEC Enterprise).md` |
| Premium Request Budgeting Scenarios | `Copilot/Premium Request Budgeting Scenarios.md` |

---

## 📚 Resources

- [Copilot premium requests](https://docs.github.com/en/copilot/concepts/billing/copilot-requests)
- [Supported AI models for Copilot](https://docs.github.com/en/copilot/reference/ai-models/supported-models)
- [Manage Copilot request allowances](https://docs.github.com/en/copilot/how-tos/manage-and-track-spending/manage-request-allowances)

---

*Last updated: April 2026*
