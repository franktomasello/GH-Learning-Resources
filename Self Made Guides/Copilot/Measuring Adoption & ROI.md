# 📊 GitHub Copilot Adoption & ROI Measurement Runbook

> **Complete guide to tracking Copilot metrics, running pilots, building dashboards, and reporting ROI to leadership**

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [📋 Overview](#-overview)
- [1️⃣ Turn On Copilot Usage Metrics](#1️⃣-turn-on-copilot-usage-metrics)
- [2️⃣ Use the Built-In Dashboards](#2️⃣-use-the-built-in-dashboards)
- [3️⃣ Track Spend and Seat Activity](#3️⃣-track-spend-and-seat-activity)
- [4️⃣ Key Metrics to Track](#4️⃣-key-metrics-to-track)
- [5️⃣ Copilot Usage Metrics API](#5️⃣-copilot-usage-metrics-api)
- [6️⃣ ROI Indicators](#6️⃣-roi-indicators)
- [7️⃣ Running a Copilot Pilot](#7️⃣-running-a-copilot-pilot)
- [8️⃣ Executive Reporting Framework](#8️⃣-executive-reporting-framework)
- [🚀 Quick Metrics Setup Recipe](#-quick-metrics-setup-recipe)
- [📝 Additional Notes](#-additional-notes)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📚 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- Turn on metrics: `Enterprise → AI controls → Copilot` → **Copilot usage metrics** policy → **Enabled everywhere**
- Dashboards: `Enterprise → Insights` → **Copilot usage** · **Code generation** · **Copilot impact**
- AI credit spend: `Enterprise → Billing and licensing → AI usage`
- Seat activity CSV: `Enterprise → Billing and licensing → Licensing` → next to Copilot, **Get activity report**
- API: `GET /enterprises/{enterprise}/copilot/metrics/reports/enterprise-28-day/latest` (returns download links to NDJSON reports)

---

## ✅ Accuracy & Click-Path Notes

<details>
<summary><em>Show click-path conventions</em></summary>


- Reviewed against current public GitHub documentation in October 2026. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
- When a path starts with `Enterprise`, begin at GitHub, click your profile picture, click `Enterprise` (managed/EMU accounts) — or open the `Enterprises` page at github.com/settings/enterprises (standard accounts) —, select the enterprise, then continue with the listed top tab or left-sidebar item.
- When a path starts with `Organization` or `Org`, begin at GitHub, click your profile picture, click `Organizations`, select the organization, click `Settings`, then continue with the listed sidebar item.
- When a path starts with `Repository`, `Repo`, or a repository name, open the repository, click the `Settings` tab, then continue with the listed sidebar item.
- When a path starts with a vendor portal such as `Microsoft Entra admin center`, `Azure portal`, `Okta Admin Console`, `PingFederate`, `PingOne`, `OneLogin`, `AD FS Management`, `Visual Studio Admin Portal`, or `Azure DevOps`, sign in to that admin portal first, select the tenant, application, or project named in the step, then follow each listed blade, tab, button, and confirmation in order.
- If the expected button is missing, verify you are signed in with the role named in Prerequisites, the feature or license is enabled, and the object is owned by the selected enterprise, organization, or repository. Use page search only to locate the same page, not to skip required confirmation, test, save, or consent clicks.

</details>

---

## ✅ Prerequisites

| Requirement | Who / Role needed | ✓ |
|-------------|-------------------|---|
| Copilot Business or Copilot Enterprise with assigned seats | GitHub **enterprise owner** or **organization owner** | ☐ |
| **Copilot usage metrics** policy enabled (Section 1) | GitHub **enterprise owner** (or **organization owner** if delegated) | ☐ |
| View the dashboards and reports | **Enterprise owner**, **enterprise billing manager**, **organization owner**, or a custom role with **View Enterprise Copilot Metrics** / **View Organization Copilot Metrics** | ☐ |
| API access | A token for one of the roles above. Classic PAT scopes: `manage_billing:copilot` or `read:enterprise` (enterprise reports), `read:org` (organization reports) | ☐ |
| Users send IDE telemetry | Developers keep telemetry on in their IDE, using a supported IDE and extension version | ☐ |
| Survey tool for qualitative feedback (optional) | Program owner | ☐ |

---

## 📋 Overview

| Method | Scope | Best for |
|--------|-------|----------|
| **Copilot usage dashboard** | Enterprise or organization (28-day trends) | Adoption and engagement tracking |
| **Code generation dashboard** | Enterprise or organization | User- vs agent-initiated code changes by model and language |
| **Copilot impact dashboard** | Enterprise | Adoption cohorts, pull request output, and a potential-ROI estimate |
| **Usage metrics API / NDJSON export** | Enterprise, organization, repository, and user reports | Power BI, Tableau, data warehouse |
| **Activity report (CSV)** | Enterprise or organization | Seat activity and license cleanup |
| **Pilot measurement and surveys** | Controlled group | Before/after comparison and sentiment |

> 📌 **What's counted:** metrics come mainly from **IDE telemetry**, plus server-side signals that catch active users whose telemetry is blocked. Copilot Chat on GitHub.com and GitHub Mobile are **not** included. Data arrives within about **two full UTC days** (the dashboard can lag up to three).

---

## 1️⃣ Turn On Copilot Usage Metrics

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **AI controls** → **Copilot** *(sidebar)*

**Steps:**

1. At the top of the enterprise page, click **AI controls**.
2. In the left sidebar, click **Copilot**.
3. Find the **Copilot usage metrics** policy and select **Enabled everywhere**. *(The API endpoints need **Enabled everywhere**.)*

> 📌 The policy applies as soon as you select it. If you delegate it to organizations, an organization owner enables it at Organization → **Settings** → **Copilot** → **Policies**.

> 💡 **Give someone metrics access without making them an owner:** create a custom organization role that includes **View organization Copilot metrics** and assign it — or use the enterprise equivalent.

---

## 2️⃣ Use the Built-In Dashboards

**👤 Role:** See Prerequisites · **📍 Portal:** GitHub

**Navigate:** Enterprise → **Insights** tab → choose a dashboard in the left sidebar

| Dashboard | Sidebar item | What it shows |
|-----------|--------------|---------------|
| **Copilot usage** | **Copilot usage** | 28-day trends: active users, engagement, acceptance, feature, model, and language use. Can export NDJSON |
| **Code generation** | **Code generation** | Lines changed with AI, split into user-initiated and agent-initiated, by model and language |
| **Copilot impact** | **Copilot impact** | Users grouped into adoption phases, an **adoption multiplier** for pull requests merged, and a **Potential return on investment** section |

**Steps (impact dashboard ROI estimate):**

1. Enterprise → **Insights** → **Copilot impact**.
2. Under **Average developer cost in your organization**, select a compensation band.
3. In the **Transition your developers to be agent-first** card, compare cost, payroll %, and pull requests per developer for **Phase 0-1 Passive and Code First Users** vs **Phase 2-3 Agent First Users**.

> ⚠️ Treat the ROI figures as **directional estimates**, not financial results. They're dashboard-only — not in the API.

**Adoption phases used by the impact dashboard:**

| Phase | Meaning (at least 2 active days in the trailing 28 days) |
|-------|--------------------------------------------------------|
| **Passive users** (API: `No Cohort`) | Hasn't reached a phase threshold yet — not the same as inactive |
| **Phase 1: Code first** | Code completions and/or agent edits in the IDE |
| **Phase 2: Agent first** | One GitHub agent surface — cloud agent, code review, or Copilot CLI |
| **Phase 3: Multi-agent** | Two or more agent surfaces, or the GitHub Copilot app |

> 📌 Organization owners see the same dashboards for their organization. A user's usage shows up in **every** organization they belong to, as long as they hold a Copilot seat somewhere in the enterprise. Enterprise totals de-duplicate users; organization totals don't.

---

## 3️⃣ Track Spend and Seat Activity

**👤 Role:** GitHub **enterprise owner** or **billing manager** · **📍 Portal:** GitHub

| What | Click path |
|------|-----------|
| AI credit consumption | Enterprise → **Billing and licensing** → **AI usage** |
| Budgets and alerts | Enterprise → **Billing and licensing** → **Budgets and alerts** |
| Seat activity (CSV) | Enterprise → **Billing and licensing** → **Licensing** → next to **Copilot**, click **Get activity report** |
| Seat activity (org CSV) | Organization → **Settings** → **Copilot** → **Access** → **Get usage report** → **Get activity report** |

> 💡 **Tip:** The per-user usage report includes `ai_credits_used`, so you can compare credits against adoption phase and pull request output.

---

## 4️⃣ Key Metrics to Track

### Adoption and engagement

| Metric | API field | What it tells you |
|--------|-----------|-------------------|
| Daily / weekly / monthly active users | `daily_active_users`, `weekly_active_users`, `monthly_active_users` | Is Copilot used regularly? |
| Active seat rate | Monthly active users ÷ assigned seats | How many paid seats are really used |
| Chat and agent users | `monthly_active_chat_users`, `monthly_active_agent_users` | Breadth beyond completions |
| Cloud agent users | `monthly_active_copilot_cloud_agent_users` | Agent adoption |
| Code review users | `monthly_active_copilot_code_review_users` (active), `monthly_passive_copilot_code_review_users` | Code review adoption |
| Acceptance | `code_acceptance_activity_count` ÷ `code_generation_activity_count` | Do developers trust the output? |
| Lines added with Copilot | `loc_added_sum` | Directional view of output |

### Cost

| Metric | Source | Action |
|--------|--------|--------|
| AI credits per user | `ai_credits_used` (per-user report) or **AI usage** page | Find heavy users; set user-level budgets |
| AI credits by model | **AI usage** page | Inform model policy decisions |
| Unused seats | Activity report / user management API (`last_activity_at`) | Reclaim seats |
| Cost per active user | (Seat cost + metered AI credits) ÷ monthly active users | Compare against ROI |

> 💡 **Tip:** Set targets only after you have a baseline from your own data. GitHub recommends looking at patterns across several signals rather than any single number — for example, steady daily active users plus a rising acceptance rate means growing trust.

---

## 5️⃣ Copilot Usage Metrics API

*Build custom dashboards in Power BI, Tableau, or a data warehouse*

Each endpoint returns `download_links` (short-lived signed URLs) and a `report_day`. Download the NDJSON files from the links.

| Endpoint | Report |
|----------|--------|
| `GET /enterprises/{enterprise}/copilot/metrics/reports/enterprise-1-day?day=YYYY-MM-DD` | Enterprise totals for one day |
| `GET /enterprises/{enterprise}/copilot/metrics/reports/enterprise-28-day/latest` | Latest 28-day enterprise totals |
| `GET /enterprises/{enterprise}/copilot/metrics/reports/users-1-day?day=YYYY-MM-DD` | Per-user, one day |
| `GET /enterprises/{enterprise}/copilot/metrics/reports/users-28-day/latest` | Per-user, latest 28 days |
| `GET /enterprises/{enterprise}/copilot/metrics/reports/repos-1-day?day=YYYY-MM-DD` | Per-repository pull request activity |
| `GET /enterprises/{enterprise}/copilot/metrics/reports/user-teams-1-day?day=YYYY-MM-DD` | User-to-team mapping (join with the per-user report for team metrics) |
| `GET /orgs/{org}/copilot/metrics/reports/organization-1-day` · `organization-28-day/latest` · `users-1-day` · `users-28-day/latest` · `repos-1-day` · `user-teams-1-day` | Same reports for one organization |

**Example:**

```bash
curl -L \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer <YOUR-TOKEN>" \
  -H "X-GitHub-Api-Version: 2026-03-10" \
  https://api.github.com/enterprises/ENTERPRISE/copilot/metrics/reports/enterprise-28-day/latest
```

**Power BI / Tableau pattern:**

1. Run a daily job that calls the `*-1-day` endpoints for the day that finished two days ago.
2. Download each file in `download_links` right away — the links expire.
3. Load the NDJSON into your warehouse, keyed by `day` (and `user_id` for per-user reports).
4. Join `user-teams` with per-user data to build team views.
5. Point Power BI or Tableau at the warehouse.

> 📌 Reports go back to **October 10, 2025** for enterprises (organization reports start **December 12, 2025**) and keep up to **one year** of history. For seat and license data, use the **Copilot user management** API instead — it's the source of truth for seats.

> ⚠️ The older `GET /orgs/{org}/copilot/metrics` endpoint and its fields (`total_active_users`, `total_engaged_users`, …) are no longer documented. Move integrations to the usage metrics reports above.

---

## 6️⃣ ROI Indicators

### Quantitative indicators

| Indicator | How to measure |
|-----------|---------------|
| **Adoption depth** | Share of users in Phase 2–3 on the impact dashboard |
| **Adoption multiplier** | Impact dashboard: pull requests merged by engaged vs passive users |
| **Pull request throughput and time to merge** | Usage metrics pull request lifecycle data, or before/after comparison |
| **Active seat rate** | Monthly active users ÷ assigned seats |
| **Cost per active user** | Seat cost + metered AI credits, divided by active users |
| **Code review turnaround** | Time from PR open to first review, before/after |

### Qualitative indicators

| Indicator | How to measure |
|-----------|---------------|
| **Developer satisfaction** | Pre/post surveys (see the pilot section) |
| **Perceived productivity** | "Do you feel more productive with Copilot?" (1–5) |
| **Task confidence** | "Do you feel more confident in unfamiliar codebases?" (1–5) |
| **Onboarding speed** | New hire time-to-first-PR, before/after |

---

## 7️⃣ Running a Copilot Pilot

*A structured way to measure impact before a full rollout*

### Pilot parameters

| Parameter | Suggested |
|-----------|-----------|
| **Duration** | 30–60 days |
| **Group size** | 50–100 users |
| **Composition** | A mix of junior, mid, and senior developers across teams |
| **Control group** | Optional: a similar group without Copilot |

### Before the pilot

1. **Turn on the Copilot usage metrics policy** (Section 1) so data is collected from day one.
2. **Collect baseline metrics** for 2–4 weeks:

| Metric | Source |
|--------|--------|
| Median time to merge | Repository insights or the REST API |
| PRs per developer per week | REST API |
| Developer satisfaction | Survey tool |

3. **Send the pre-pilot survey:**

| Question | Scale |
|----------|-------|
| "How productive do you feel in your daily coding work?" | 1–5 |
| "How confident are you working in unfamiliar codebases?" | 1–5 |
| "How much time do you spend searching for code examples?" | Hours/week |
| "How satisfied are you with your development tooling?" | 1–5 |

### During the pilot

- Check the **Copilot usage** dashboard weekly (active users, acceptance)
- Hold check-ins every two weeks
- Log blockers and feedback

### After the pilot

1. **Post-pilot survey** (same questions, plus):

| Question | Scale |
|----------|-------|
| "How useful is GitHub Copilot for your daily work?" | 1–5 |
| "Would you recommend Copilot to a colleague?" | Yes/No |
| "What tasks does Copilot help you most with?" | Open text |

2. **Compare:**

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| Median time to merge | ___ | ___ | ___% |
| PRs per developer per week | ___ | ___ | ___% |
| Developer satisfaction | ___ | ___ | +/- ___ |
| AI credits per active user | — | ___ | — |

---

## 8️⃣ Executive Reporting Framework

### Recommended report structure

| Section | Content |
|---------|---------|
| **Adoption summary** | Active users, active seat rate, adoption phases, trend |
| **Productivity indicators** | Acceptance trend, adoption multiplier, pull request throughput and time to merge |
| **Cost analysis** | Seat cost, AI credits consumed, cost per active user |
| **Developer sentiment** | Survey results and quotes |
| **Recommendations** | Expand rollout, adjust policies and budgets, reclaim unused seats |

### Reporting cadence

| Audience | Frequency | Focus |
|----------|-----------|-------|
| **Engineering leadership** | Monthly | Adoption trends, team-level metrics |
| **Executive sponsors** | Quarterly | ROI summary, cost, strategy |
| **Finance** | Monthly or quarterly | AI credit spend, seat use, forecast |
| **Pilot stakeholders** | Weekly (during pilot) | Participation, early feedback |

> 💡 **Tip:** Lead with adoption in the first 90 days, then shift to outcomes (adoption depth, pull request flow, cost per active user) once usage settles.

---

## 🚀 Quick Metrics Setup Recipe

*Get visibility into Copilot usage in under 30 minutes:*

### Steps

1. **Enable the policy:** Enterprise → **AI controls** → **Copilot** → **Copilot usage metrics** → **Enabled everywhere**.
2. **Open the dashboards:** Enterprise → **Insights** → **Copilot usage**, then **Code generation** and **Copilot impact**.
3. **Check spend:** Enterprise → **Billing and licensing** → **AI usage**.
4. **Download seat activity:** **Billing and licensing** → **Licensing** → **Get activity report** (next to Copilot).
5. **Automate reporting:** schedule daily calls to the `*-1-day` report endpoints and load the NDJSON into your warehouse.
6. **Set a monthly reporting cadence** using the framework above.

---

## 📝 Additional Notes

> 💡 **Data delay:** metrics land within about two full UTC days, so a brand-new rollout won't show data immediately.

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
| **Copilot stops working for a user mid-cycle** | The user's user-level budget is used up, the shared AI credit pool is exhausted with **AI credits paid usage** disabled, or a spending limit with **Stop usage** was reached. | Check the user on **Billing and licensing** → **AI usage** and the budgets on **Budgets and alerts**; raise their budget, approve their budget request, or enable AI credits paid usage. |
| **Content exclusions do not apply immediately** | Client policy cache, unsupported surface/mode, symlink/remote filesystem limitation, or indirect IDE context. | Reload the IDE policy, verify the exclusion syntax at enterprise/org/repo scope, and document surfaces where exclusions are limited. |
| **Usage metrics look empty or inconsistent** | Telemetry is disabled, data freshness delay applies, users are unlicensed, or different APIs report different scopes. | Enable the metrics policy, confirm seats and telemetry, wait for data freshness, and avoid comparing dashboards/API endpoints as if they share identical data models. |
| **Cloud agent or MCP action is denied** | Agent policy, MCP policy, repository permissions, secrets, or server allowlist does not permit the operation. | Review Enterprise AI controls > Agents/MCP, repo-level permissions, MCP server configuration, and audit logs for the denied action. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>


### Q: Our acceptance rate seems low — is that normal?
**A:** GitHub doesn't publish a "normal" acceptance rate, so compare against your own baseline and look at the trend. Low acceptance often points to weak context (no `.github/copilot-instructions.md`), little training, or languages and tasks where Copilot is less useful. Add instructions, run enablement sessions, and watch whether the trend improves.

---

### Q: The usage metrics API returns no data or a 404 — what's wrong?
**A:** Check that (1) the **Copilot usage metrics** policy is **Enabled everywhere**, (2) your token belongs to an enterprise owner, billing manager, or someone with the **View Enterprise Copilot Metrics** permission, with the right scope (`manage_billing:copilot` or `read:enterprise`; `read:org` for organization reports), (3) the day you asked for is at least two full UTC days old and not before the report start date, and (4) the enterprise or organization slug is correct.

---

### Q: How do we attribute productivity gains to Copilot vs other factors?
**A:** Use a before/after pilot with a control group. Collect baseline metrics (time to merge, PRs per developer, satisfaction) for 2–4 weeks, then compare over 30–60 days. The impact dashboard's adoption multiplier adds a within-company comparison of engaged vs passive users. Pair the numbers with surveys.

---

### Q: Can we export Copilot metrics to Power BI or Tableau?
**A:** Yes. Call the usage metrics report endpoints (for example `GET /enterprises/{enterprise}/copilot/metrics/reports/enterprise-1-day`), download the NDJSON files from `download_links` before they expire, load them into a warehouse, and connect Power BI or Tableau. You can also export NDJSON from the **Copilot usage** dashboard.

---

### Q: Seat utilization is low — many assigned users aren't using Copilot. What should we do?
**A:** Download the activity report to find inactive seats, run enablement sessions or office hours, and reclaim seats from people who still don't use Copilot after a set period (for example 30 days). Assigning seats through teams makes this easier to manage.

---

### Q: How long should we wait before reporting ROI to leadership?
**A:** Report adoption in the first 90 days (active users, active seat rate, adoption phases). Move to outcome metrics — adoption multiplier, pull request flow, cost per active user — after about 3–6 months, once usage has settled and you have before/after data.

---

### Q: Leadership wants a dollar-value ROI — how do we calculate it?
**A:** Start with the impact dashboard's **Potential return on investment** section, which compares Copilot cost (from actual AI credit use) with pull request output per developer for your chosen compensation band. Treat it as directional. Add your own before/after data and survey results, and compare against total cost: seats plus metered AI credits.

</details>

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Seat Assignment & Enablement | `Copilot/Seat Assignment & Enablement.md` |
| AI Credits Budget & Overage Planning | `Copilot/AI Credits Budget & Overage Planning.md` |
| Admin Controls (Models, Content Exclusion, Instructions) | `Copilot/Admin Controls (Models, Content Exclusion, Instructions).md` |

---

## 📚 Resources

- [About Copilot usage metrics](https://docs.github.com/en/copilot/concepts/billing-and-usage/copilot-usage-metrics/copilot-metrics)
- [View the Copilot usage metrics dashboard](https://docs.github.com/en/copilot/how-tos/administer-copilot/view-usage-and-adoption)
- [View the Copilot impact dashboard](https://docs.github.com/en/copilot/how-tos/administer-copilot/view-impact-dashboard)
- [Copilot usage metrics reference](https://docs.github.com/en/copilot/reference/copilot-usage-metrics/copilot-usage-metrics)
- [REST API: Copilot usage metrics](https://docs.github.com/en/enterprise-cloud@latest/rest/copilot/copilot-usage-metrics)
- [Download the Copilot activity report](https://docs.github.com/en/copilot/how-tos/administer-copilot/download-activity-report)

---

*Last updated: October 2026*
