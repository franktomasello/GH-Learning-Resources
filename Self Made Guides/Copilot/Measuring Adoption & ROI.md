# 📊 GitHub Copilot Adoption & ROI Measurement Runbook

> **Complete guide to tracking Copilot metrics, running pilots, building dashboards, and reporting ROI to leadership**

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [📋 Overview](#-overview)
- [1️⃣ Built-In Metrics at the Enterprise Level](#1-built-in-metrics-at-the-enterprise-level)
- [2️⃣ Organization-Level Metrics](#2-organization-level-metrics)
- [3️⃣ Key Metrics to Track](#3-key-metrics-to-track)
- [4️⃣ Copilot Metrics API](#4-copilot-metrics-api)
- [5️⃣ ROI Indicators](#5-roi-indicators)
- [6️⃣ Running a Copilot Pilot](#6-running-a-copilot-pilot)
- [7️⃣ Executive Reporting Framework](#7-executive-reporting-framework)
- [🚀 Quick Metrics Setup Recipe](#-quick-metrics-setup-recipe)
- [📝 Additional Notes](#-additional-notes)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📚 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- Enterprise adoption metrics: `Enterprise → AI controls → Insights`
- Premium request spend: `Enterprise → Billing and licensing → Usage`
- Org-level usage: `Org Settings → Copilot → Usage`
- API for custom dashboards: `GET /orgs/{org}/copilot/metrics` or `/enterprises/{enterprise}/copilot/metrics`

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
| GitHub Enterprise Cloud account with enterprise owner or billing admin access | ☐ |
| Copilot Business or Copilot Enterprise subscription active with assigned seats | ☐ |
| API token with `manage_billing:copilot` scope (for metrics API) | ☐ |
| Survey tool for qualitative developer feedback (optional) | ☐ |

---

## 📋 Overview

This runbook covers how to measure and report on GitHub Copilot adoption and return on investment:

| Method | Scope | Best For |
|--------|-------|----------|
| **Built-in enterprise metrics** | Enterprise-wide | Executive dashboards, adoption tracking |
| **Org-level usage metrics** | Single organization | Team-level insights |
| **Copilot metrics API** | Custom dashboards | Power BI, Tableau, automated reporting |
| **Pilot measurement** | Controlled group | Pre-purchase justification, before/after comparison |
| **Developer surveys** | Qualitative | Sentiment, productivity perception |

---

## 1️⃣ Built-In Metrics at the Enterprise Level

*Access adoption metrics and premium request spend from the enterprise dashboard*

### A) Adoption Metrics

**Navigation:**

```
Enterprise → AI controls → Insights
```

**Available Metrics:**

| Metric | Description |
|--------|-------------|
| **Active users** | Number of users who used Copilot in the selected period |
| **Acceptance rate** | Percentage of Copilot suggestions accepted by users |
| **Lines suggested** | Total lines of code suggested by Copilot |
| **Lines accepted** | Total lines of code accepted by users |
| **Language breakdown** | Usage by programming language |
| **IDE breakdown** | Usage by editor (VS Code, JetBrains, Neovim, etc.) |

### B) Premium Request Spend

**Navigation:**

```
Enterprise → Billing and licensing → Usage
```

**Available Metrics:**

| Metric | Description |
|--------|-------------|
| **Premium requests consumed** | Total premium model requests across the enterprise |
| **Per-user consumption** | Premium requests broken down by user |
| **Per-model consumption** | Premium requests broken down by AI model |
| **Spending trends** | Historical consumption over time |

> 💡 **Tip:** Use the date range selector to compare adoption across months. Look for trends in the first 30, 60, and 90 days after rollout.

---

## 2️⃣ Organization-Level Metrics

*View usage data scoped to a specific organization*

**Navigation:**

```
Profile Picture → Organizations → [Your Organization] → Settings
  → Copilot → Usage
```

**Available Metrics:**

| Metric | Description |
|--------|-------------|
| **Active users** | Members who used Copilot in the period |
| **Seat utilization** | Assigned seats vs active seats |
| **Acceptance rate** | Suggestion acceptance percentage |
| **Language breakdown** | Top languages by Copilot usage |
| **IDE breakdown** | Top editors by Copilot usage |

> 💡 **Tip:** Compare seat utilization against assigned seats to identify unused licenses. If utilization is below 70%, consider reassigning idle seats or running enablement sessions.

---

## 3️⃣ Key Metrics to Track

*The metrics that matter most for adoption and ROI reporting*

### Adoption Metrics

| Metric | What It Tells You | Healthy Range |
|--------|-------------------|---------------|
| **Active user rate** | Percentage of assigned seats actively used | 70-90% |
| **Acceptance rate** | How useful suggestions are to developers | 25-40% typical |
| **Lines accepted per user per day** | Volume of AI-assisted code | Varies by role |
| **IDE coverage** | Which editors are being used with Copilot | Broad adoption preferred |
| **Language coverage** | Which languages benefit most | Compare to your tech stack |

### Cost Metrics

| Metric | What It Tells You | Action |
|--------|-------------------|--------|
| **Premium requests per user** | Who is using premium models heavily | Monitor for budget planning |
| **Premium requests per model** | Which models drive the most cost | Inform model restriction decisions |
| **Seat utilization** | Assigned vs active seats | Reclaim unused seats |
| **Cost per active user** | Effective cost of Copilot per productive user | Compare against ROI |

---

## 4️⃣ Copilot Metrics API

*Build custom dashboards in Power BI, Tableau, or other reporting tools*

### API Endpoint

The Copilot metrics API provides programmatic access to usage data:

| Endpoint | Scope | Data |
|----------|-------|------|
| `GET /orgs/{org}/copilot/metrics` | Organization | Daily usage metrics |
| `GET /enterprises/{enterprise}/copilot/metrics` | Enterprise | Daily usage metrics |

### Available Data Fields

| Field | Description |
|-------|-------------|
| `total_active_users` | Users who used Copilot on that day |
| `total_engaged_users` | Users who accepted at least one suggestion |
| `total_code_acceptances` | Number of accepted suggestions |
| `total_code_suggestions` | Number of suggestions shown |
| `total_code_lines_accepted` | Lines of code accepted |
| `total_code_lines_suggested` | Lines of code suggested |
| `breakdown` | Per-language and per-editor breakdown |

### Example: Exporting to Power BI

1. Use the GitHub REST API to pull daily metrics
2. Store in a data warehouse or CSV
3. Connect Power BI to the data source
4. Build dashboards with adoption trends, acceptance rates, and language breakdowns

> 💡 **Tip:** Schedule API calls daily to build a historical dataset. The API returns daily granularity, so pulling once per day ensures complete data.

---

## 5️⃣ ROI Indicators

*Metrics and signals that demonstrate return on investment*

### Quantitative Indicators

| Indicator | How to Measure | Benchmark |
|-----------|---------------|-----------|
| **Acceptance rate** | Built-in metrics | 25-40% is typical for mature rollouts |
| **Lines accepted per user per day** | API or built-in metrics | Track trend over time |
| **Active user rate** | Assigned seats vs active users | Target 70%+ |
| **PR velocity** | Compare PR merge time before/after Copilot | 10-30% improvement common |
| **Code review turnaround** | Measure time from PR open to first review | Track for improvement |

### Qualitative Indicators

| Indicator | How to Measure |
|-----------|---------------|
| **Developer satisfaction** | Pre/post surveys (see pilot section below) |
| **Perceived productivity** | "Do you feel more productive with Copilot?" (1-5 scale) |
| **Task confidence** | "Do you feel more confident tackling unfamiliar codebases?" |
| **Onboarding speed** | New hire time-to-first-PR before/after Copilot |

---

## 6️⃣ Running a Copilot Pilot

*Structured approach to measuring Copilot impact before full rollout*

### Pilot Parameters

| Parameter | Recommended |
|-----------|-------------|
| **Duration** | 30-60 days |
| **Group size** | 50-100 users |
| **Composition** | Mix of junior, mid, and senior developers across teams |
| **Control group** | Optional: equal-sized group without Copilot for comparison |

### Before the Pilot

1. **Baseline metrics** (collect 2-4 weeks before pilot starts):

| Metric | Source |
|--------|--------|
| Average PR merge time | GitHub Insights or API |
| PRs per developer per week | GitHub API |
| Lines of code per PR | GitHub API |
| Developer satisfaction survey | Survey tool |

2. **Pre-pilot survey** (send to all pilot participants):

| Question | Scale |
|----------|-------|
| "How productive do you feel in your daily coding work?" | 1-5 |
| "How confident are you working in unfamiliar codebases?" | 1-5 |
| "How much time do you spend searching for code examples?" | Hours/week |
| "How satisfied are you with your development tooling?" | 1-5 |

### During the Pilot

- Track Copilot metrics weekly (active users, acceptance rate)
- Hold bi-weekly check-ins with pilot participants
- Document blockers and feedback

### After the Pilot

1. **Post-pilot survey** (same questions as pre-pilot, plus):

| Question | Scale |
|----------|-------|
| "How useful is GitHub Copilot for your daily work?" | 1-5 |
| "Would you recommend Copilot to a colleague?" | Yes/No |
| "What tasks does Copilot help you most with?" | Open text |

2. **Compare metrics:**

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| PR merge time | ___ | ___ | ___% |
| PRs per developer per week | ___ | ___ | ___% |
| Developer satisfaction | ___ | ___ | +/- ___ |

---

## 7️⃣ Executive Reporting Framework

*Structure for presenting Copilot ROI to leadership*

### Recommended Report Structure

| Section | Content |
|---------|---------|
| **Adoption summary** | Active users, seat utilization, trend over time |
| **Productivity indicators** | Acceptance rate, lines accepted, PR velocity changes |
| **Cost analysis** | Cost per seat, cost per active user, premium request spend |
| **Developer sentiment** | Survey results, satisfaction scores, qualitative feedback |
| **Recommendations** | Expand rollout, adjust policies, reclaim unused seats |

### Reporting Cadence

| Audience | Frequency | Focus |
|----------|-----------|-------|
| **Engineering leadership** | Monthly | Adoption trends, team-level metrics |
| **Executive sponsors** | Quarterly | ROI summary, cost analysis, strategic recommendations |
| **Finance** | Quarterly | Seat utilization, cost per active user, budget forecasting |
| **Pilot stakeholders** | Weekly (during pilot) | Participation rates, early feedback |

> 💡 **Tip:** Lead with adoption metrics in early reports (first 90 days) and shift to ROI metrics as usage matures. Acceptance rates and developer sentiment are the strongest early indicators of long-term value.

---

## 🚀 Quick Metrics Setup Recipe

*Get visibility into Copilot usage in under 30 minutes:*

### Steps

1. **Check enterprise-level adoption:**
   ```
   Enterprise → AI controls → Insights
   ```

2. **Check premium request spend:**
   ```
   Enterprise → Billing and licensing → Usage
   ```

3. **Review org-level details:**
   ```
   Org Settings → Copilot → Usage
   ```

4. **Set up API-based reporting** for custom dashboards:
   - Use `GET /orgs/{org}/copilot/metrics` for daily data
   - Schedule daily exports to your data warehouse

5. **Establish a monthly reporting cadence** using the executive framework above

---

## 📝 Additional Notes

> 💡 **Customization:** Metrics availability may vary based on your Copilot plan (Business vs Enterprise) and enterprise agreement. Some advanced metrics and API endpoints may require Copilot Enterprise. The navigation paths above reflect the current UI at time of writing.

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


### Q: Our acceptance rate seems low (15-20%) — is that normal?
**A:** Typical mature deployments see 25-40% acceptance rates. Low rates may indicate poor context setup (missing custom instructions or repository indexing), lack of developer training, or users working in languages where Copilot is less effective. Add `.github/copilot-instructions.md` files and run enablement sessions to improve.

---

### Q: The Copilot metrics API is returning empty data — what's wrong?
**A:** Check three things: (1) your API token has the correct permissions (requires `manage_billing:copilot` or admin scope), (2) ensure Copilot has been active for at least 24 hours — the API returns daily granularity and may not have data yet, and (3) verify you are using the correct org or enterprise slug in the endpoint URL.

---

### Q: How do we attribute productivity gains to Copilot vs other factors?
**A:** Use a before/after pilot methodology with a control group. Collect baseline metrics (PR merge time, PRs per developer, developer satisfaction) for 2-4 weeks before the pilot, then compare against the pilot group over 30-60 days. Pair quantitative metrics with developer surveys to capture both measurable and perceived impact.

---

### Q: Can we export Copilot metrics to Power BI or Tableau?
**A:** Yes. Use the Copilot metrics API endpoints (`GET /orgs/{org}/copilot/metrics` or `/enterprises/{enterprise}/copilot/metrics`) to pull daily data. Schedule daily exports to a data warehouse or CSV, then connect Power BI or Tableau to the data source to build custom dashboards.

---

### Q: Seat utilization is low — many assigned users are not using Copilot. What should we do?
**A:** If utilization is below 70%, consider running enablement sessions or office hours to help users get started. Reclaim unused seats from users who have not activated Copilot after 30 days. Use team-based assignment to ensure seats go to engaged users.

---

### Q: How long should we wait before reporting ROI to leadership?
**A:** Lead with adoption metrics in the first 90 days (active users, seat utilization, acceptance rate trends). Shift to ROI metrics (PR velocity improvements, developer satisfaction changes, cost per active user) after 3-6 months when usage patterns have stabilized and you have meaningful before/after comparisons.

---

### Q: Our metrics show high usage but leadership wants dollar-value ROI — how do we calculate that?
**A:** Estimate developer time saved by multiplying accepted suggestions by average time-per-task saved (typically 30-60 seconds per accepted suggestion). Convert to hourly developer cost. Compare this against total Copilot licensing and premium request costs. Supplement with survey data on perceived productivity gains.

</details>

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Seat Assignment & Enablement | `Copilot/Seat Assignment & Enablement.md` |
| Premium Request Budget & Overage Planning | `Copilot/Premium Request Budget & Overage Planning.md` |
| Admin Controls (Models, Content Exclusion, Instructions) | `Copilot/Admin Controls (Models, Content Exclusion, Instructions).md` |

---

## 📚 Resources

- [GitHub Copilot usage metrics](https://docs.github.com/en/copilot/concepts/copilot-metrics)
- [Copilot metrics API](https://docs.github.com/en/rest/copilot/copilot-metrics)

---

*Last updated: April 2026*
