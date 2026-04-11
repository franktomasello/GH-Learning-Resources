# 📊 Measuring GitHub Copilot Adoption & ROI Runbook

> **Complete guide to tracking Copilot metrics, running pilots, building dashboards, and reporting ROI to leadership**

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
Enterprise → Billing & Licensing → Usage
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
   Enterprise → Billing & Licensing → Usage
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

---

## 📚 Resources

- [Analyzing Copilot usage in your organization](https://docs.github.com/en/copilot/rolling-out-github-copilot-at-scale/analyzing-usage-and-impact/analyzing-copilot-usage-in-your-organization)
- [Copilot metrics API](https://docs.github.com/en/rest/copilot/copilot-metrics)

---

*Last updated: April 2026*
