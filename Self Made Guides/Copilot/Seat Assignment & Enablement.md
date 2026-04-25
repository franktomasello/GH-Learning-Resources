# 💺 GitHub Copilot Seat Assignment & Enablement Runbook

> **Complete guide to assigning Copilot seats across enterprise, organization, and team levels including pilot rollouts and mixed-plan scenarios**

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- Enable orgs: `Enterprise → AI controls → Copilot → Access → Enabled for selected organizations`
- Assign individual seats: `Org Settings → Copilot → Access → Enabled for selected members → Add members`
- Team-based assignment: `Org Settings → Copilot → Access → Enabled for selected members → Add teams`
- Enable all members: `Org Settings → Copilot → Access → All members`
- Assign Business vs Enterprise tier: `Enterprise → AI controls → Copilot → Access` — set tier per org

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
| Copilot Business or Copilot Enterprise subscription purchased | ☐ |
| Organization admin access (for org-level seat assignment) | ☐ |
| GitHub Teams created (for team-based pilot rollouts) | ☐ |

---

## 📋 Overview

This runbook covers every way to assign and manage GitHub Copilot seats:

| Method | Scope | Best For |
|--------|-------|----------|
| **Enterprise-level org selection** | Entire enterprise | Choosing which orgs get Copilot |
| **Org-level selected members** | Specific users | Controlled rollouts, budget management |
| **Team-based assignment** | GitHub Team | Pilot programs, department rollouts |
| **Org-level all members** | Entire organization | Full org enablement |
| **Enterprise tier assignment** | Enterprise | Assigning Business vs Enterprise plans to orgs |

---

## 1️⃣ Enable Copilot for Specific Organizations (Enterprise Level)

*Choose which organizations under your enterprise have access to Copilot*

**Navigation:**

```
Enterprise → AI controls → Copilot → Access
  → "Enabled for selected organizations"
```

**Steps:**

1. Select **Enabled for selected organizations**
2. Use the search box to find organizations
3. Check the organizations you want to enable
4. Save changes

> 💡 **Tip:** Start with a single organization for your pilot, then expand as adoption grows.

---

## 2️⃣ Assign Seats to Specific Users (Organization Level)

*Grant Copilot access to individual members within an organization*

**Navigation:**

```
Profile Picture → Organizations → [Your Organization] → Settings
  → Copilot → Access → "Enabled for selected members"
```

**Steps:**

1. Select **Enabled for selected members**
2. Click **Add members**
3. Search for and select the users to grant access
4. Confirm the seat assignments

> ⚠️ **Important:** Each assigned seat counts toward your Copilot license total. Users who are assigned but never activate Copilot still consume a seat.

---

## 3️⃣ Team-Based Assignment for Pilots

*Recommended approach for running a Copilot pilot program*

### A) Create a GitHub Team

**Navigation:**

```
Profile Picture → Organizations → [Your Organization]
  → Teams → New team
```

**Steps:**

1. Enter a team name (e.g., `copilot-pilot`)
2. Add a description (e.g., "Copilot pilot participants - Q2 2026")
3. Set visibility to **Visible** or **Secret** depending on preference
4. Click **Create team**

### B) Add Pilot Users to the Team

**Navigation:**

```
Profile Picture → Organizations → [Your Organization]
  → Teams → [copilot-pilot] → Members → Add member
```

### C) Grant Copilot Access to the Team

**Navigation:**

```
Profile Picture → Organizations → [Your Organization] → Settings
  → Copilot → Access → "Enabled for selected members"
    → Add teams → Search for [copilot-pilot] → Select → Save
```

> ✅ **Result:** Only members of the `copilot-pilot` team will have Copilot access. New members added to the team automatically receive a seat.

> 💡 **Tip:** Team-based assignment makes it easy to add or remove pilot participants without managing individual seats. It also simplifies reporting because you can track usage by team.

---

## 4️⃣ Enable Copilot for All Members (Organization Level)

*Grant Copilot access to every member of the organization*

**Navigation:**

```
Profile Picture → Organizations → [Your Organization] → Settings
  → Copilot → Access → "All members"
```

**Steps:**

1. Select **All members**
2. Confirm the enablement

> ⚠️ **Important:** This will assign a seat to every current member and automatically assign seats to new members as they join the organization. Monitor your seat count against your license total.

---

## 5️⃣ Mixed Plans: Business + Enterprise in the Same Enterprise

*Assign different Copilot tiers to different organizations*

### Understanding Mixed Plans

| Tier | Features | Typical Use |
|------|----------|-------------|
| **Copilot Business** | Code completions, Chat, CLI | Standard developer productivity |
| **Copilot Enterprise** | Business features + knowledge bases, Bing search, PR summaries | Teams needing advanced features |

### Assign Tiers at the Enterprise Level

**Navigation:**

```
Enterprise → AI controls → Copilot → Access
```

**Steps:**

1. For each enabled organization, select the Copilot tier:
   - **Copilot Business**
   - **Copilot Enterprise**
2. Save changes

> 💡 **Tip:** You can run some organizations on Copilot Business and others on Copilot Enterprise under the same enterprise. This lets you control costs while giving advanced features to teams that need them.

---

## 6️⃣ Resolving Duplicate Tier Assignments

*When a user has both Business and Enterprise assignments*

### How Duplicates Happen

A user can end up with multiple Copilot assignments if they belong to more than one organization under the enterprise, and those organizations are assigned different Copilot tiers.

### Resolution Rules

| Scenario | Result |
|----------|--------|
| User has Business in Org A + Enterprise in Org B | User gets **Enterprise** (highest tier wins) |
| User has Business in Org A + Business in Org B | User consumes **one** Business seat |

### Identifying Duplicate Assignments

**Navigation:**

```
Enterprise → AI controls → Copilot → Access
```

Review the seat assignment summary. Users with multiple assignments will be flagged.

> 💡 **Tip:** To avoid unnecessary costs, audit seat assignments periodically. Remove users from the lower-tier organization's Copilot access if they already have a higher tier through another org.

---

## 7️⃣ Copilot Enterprise vs Business Assignment at the Enterprise Level

*Set the default tier and manage organization-level plan assignments*

**Navigation:**

```
Enterprise → AI controls → Copilot → Access
```

**Options:**

| Setting | Effect |
|---------|--------|
| **Copilot Business** for an org | All seats in that org are Business tier |
| **Copilot Enterprise** for an org | All seats in that org are Enterprise tier |

> ⚠️ **Important:** Changing an organization's tier from Enterprise to Business will immediately remove access to Enterprise-only features (knowledge bases, Bing search in Chat, PR summaries) for all users in that org.

---

## 🚀 Quick Pilot Rollout Recipe

*Recommended steps for a controlled Copilot pilot:*

### Steps

1. **Enable Copilot for the pilot organization:**
   ```
   Enterprise → AI controls → Copilot → Access
     → Enabled for selected organizations → [Pilot Org]
   ```

2. **Create a pilot team:**
   ```
   Org → Teams → New team → "copilot-pilot"
   ```

3. **Add pilot users to the team** (50-100 recommended)

4. **Grant Copilot to the team:**
   ```
   Org Settings → Copilot → Access
     → Enabled for selected members → Add teams → copilot-pilot
   ```

5. **Set policies** (model access, public code filter, content exclusions)

6. **Run for 30-60 days**, then review metrics

---

## 📝 Additional Notes

> 💡 **Customization:** The exact UI labels and options may vary slightly depending on your enterprise agreement and whether you are on Copilot Business, Copilot Enterprise, or a trial. The navigation paths above reflect the current UI at time of writing.

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

### Q: A user says Copilot isn't working even though we assigned them a seat — what should we check?
**A:** Verify four things: (1) the user's IDE extension is up to date, (2) their SSO/SAML session is active and not expired, (3) the user has accepted the Copilot terms of service, and (4) they are signed into the correct GitHub account in their IDE. An assigned seat does not activate until the user completes these steps.

---

### Q: Can we auto-assign Copilot to all new org members?
**A:** Yes, set the access policy to "All members" under Org Settings > Copilot > Access. New members joining the organization will automatically receive a Copilot seat. Monitor your seat count against your license total to avoid exceeding your purchased seats.

---

### Q: A user has both Copilot Business and Enterprise assigned — what happens?
**A:** Enterprise takes precedence. The user gets Enterprise-tier features. To avoid unnecessary cost, remove the lower-tier (Business) assignment by removing them from the Business-tier organization's Copilot access.

---

### Q: How do we revoke a seat immediately?
**A:** Remove the user from the team or individual seat assignment under Org Settings > Copilot > Access. The revocation takes effect within minutes. The user will lose Copilot functionality in their IDE on their next session refresh.

---

### Q: Seats are showing as "pending" — what does that mean?
**A:** A pending seat means the user has been assigned but has not yet activated Copilot. The user needs to sign into an IDE with the Copilot extension installed and accept the Copilot terms of service. Pending seats still consume a license.

---

### Q: We removed a user from the org but they still appear to have a Copilot seat — why?
**A:** Seat assignment changes may take a few minutes to reflect in the UI. If the user belongs to multiple organizations under the same enterprise, they may still have a seat through another org. Check all org-level assignments in the enterprise access view.

---

### Q: How many seats can we assign during a pilot without over-purchasing?
**A:** Use team-based assignment with a dedicated pilot team (e.g., `copilot-pilot`). This lets you control exactly who has access. Seats are counted at the enterprise level, so you only need enough total licenses to cover all assigned users across all orgs.

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Admin Controls (Models, Content Exclusion, Instructions) | `Copilot/Admin Controls (Models, Content Exclusion, Instructions).md` |
| Premium Request Budget & Overage Planning | `Copilot/Premium Request Budget & Overage Planning.md` |
| Measuring Adoption & ROI | `Copilot/Measuring Adoption & ROI.md` |
| Enterprise Trial (GHEC, EMU, DRUS) | `Setup/Enterprise Trial (GHEC, EMU, DRUS).md` |

---

## 📚 Resources

- [Managing access to GitHub Copilot in your organization](https://docs.github.com/en/copilot/managing-copilot/managing-github-copilot-in-your-organization/managing-access-to-github-copilot-in-your-organization)
- [Managing policies and features for Copilot in your enterprise](https://docs.github.com/en/copilot/managing-copilot/managing-copilot-for-your-enterprise/managing-policies-and-features-for-copilot-in-your-enterprise)

---

*Last updated: April 2026*
