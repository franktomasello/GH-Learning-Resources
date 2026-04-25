# 🚀 GitHub Copilot Standalone Licensing Guide

> **How to assign Copilot Business licenses without consuming GitHub Enterprise (GHE) licenses**

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- Create enterprise team: `Enterprise → People → Enterprise teams → Create enterprise team` (leave org access blank)
- Add users: IdP group sync, REST API bulk add, or manual UI
- Assign licenses: `Enterprise → Billing and licensing → Licensing → Copilot → Manage → Enterprise teams tab → Assign licenses`

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
| Copilot Business subscription purchased | ☐ |
| List of GitHub usernames for standalone users | ☐ |
| IdP group configured (if using IdP sync) or API access (if using REST API) | ☐ |

---

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **GitHub enterprise or organization owner** | Creates or selects the GitHub team used for Copilot assignment, then assigns Copilot seats. | Team setup: GitHub → profile photo → Your organizations → [organization] → Teams → New team or [team] → Members → Add members. Enterprise Copilot assignment: GitHub → profile photo → Your enterprises → [enterprise] → Billing and licensing → Licensing → Copilot → Manage → Enterprise teams tab → Assign licenses. Organization Copilot assignment: GitHub → profile photo → Your organizations → [organization] → Settings → Copilot → Access → Add seats or Assign seats → choose users or teams → Save. Handoff: team slug and assigned-seat count. |
| **Microsoft Entra or Okta group owner, if team membership is synchronized from the IdP** | Maintains the source group that drives GitHub team membership. | Entra: Microsoft Entra admin center → Entra ID → Groups → [group] → Members → Add members → select users → Add. Okta: Okta Admin Console → Directory → Groups → [group] → People → Assign people → select users → Save. Handoff: group name, object ID or group ID, and pilot members. |
| **Visual Studio subscriptions administrator, if Visual Studio benefits are part of the license plan** | Confirms Visual Studio Enterprise subscribers are assigned to the correct corporate identity before GitHub seat assignment. | Visual Studio Admin Portal (`https://manage.visualstudio.com`) → Subscribers → search user → verify Visual Studio Enterprise, active status, corporate email, and tenant. Handoff: eligible subscriber list. |

---

## 📋 Overview

Assign Copilot Business licenses to ~1,000 users **without consuming GitHub Enterprise (GHE) licenses**.

---

## 🔧 Phase 1: The Setup (Critical Configuration)

### Step 1: Navigate to Enterprise Settings

1. Click your profile photo (top-right) → **Your enterprises**
2. Select the target enterprise

### Step 2: Create the Enterprise Team (The "Zero-Cost" Step)

> ⚠️ **CRITICAL**: This step determines if you pay for 1,000 extra GHE licenses or not. Follow exactly.

1. Go to **People** → **Enterprise teams**
2. Click **Create enterprise team**
3. **Name:** Use a functional name (e.g., `copilot-standalone-users`, `contractor-devs`)
4. **Organizations:** **LEAVE THIS BLANK**

#### 🛑 WARNING: Organization Access
- **DO NOT** select any organization
- **Why?** If you grant this team access to an organization (even "Read" access), every user in it becomes a "standard enterprise member" and immediately consumes a full GitHub Enterprise license (approx. $21/mo or $231/yr depending on contract)
- **Correct State:** The team exists at the *enterprise* level only, with no org affiliation

---

## 👥 Phase 2: Add Users to the Team

Choose the method that matches your infrastructure.

### Option A: IdP Group Sync (Best for Long-Term/EMU)

*Requires Enterprise Managed Users (EMU) or Team Sync configured.*

1. In your IdP (Okta, Azure AD, etc.), create a group (e.g., `github-copilot-users`)
2. In GitHub **Enterprise teams**, select your team (`copilot-standalone-users`)
3. Go to **Settings** → **Identity Provider Groups**
4. Connect the IdP group
   - **Constraint:** The GitHub team must have **no manually assigned users** before you link the group
   - **Result:** Membership is now 100% automated by your IdP

### Option B: Scripted REST API (Best for Bulk Initial Setup)

*Use this if you have a CSV of usernames and don't use IdP sync.*

**Correct Endpoint:**
```
PUT /enterprises/{enterprise}/teams/{enterprise-team}/memberships/{username}
```

**Bulk Add Option (Recommended for 1,000 users):**
```
POST /enterprises/{enterprise}/teams/{enterprise-team}/memberships/add
```
*(accepts an array of usernames)*

**Important Note:** Enterprise team slugs receive an `ent:` prefix. Use the full slug format in API calls.

**Logic:**
1. Read list of usernames
2. Loop through list: Call endpoint for each user (or use bulk add)
3. Handle 404s (user not found) or 422s (already member)

### Option C: Manual UI (Not Recommended)

- Valid only for <50 users
- Do not attempt for 1,000 users

---

## 📜 Phase 3: Assign the License

1. In the Enterprise account, go to **Billing and licensing**
2. Select **Licensing** → **Copilot**
3. Click **Manage** (next to Copilot Business)
4. Switch to the **Enterprise teams** tab (not the default "Organizations" tab)
5. Click **Assign licenses**
6. Select your team (`copilot-standalone-users`) and confirm

### ✅ Outcome
- **All 1,000 users now have a Copilot Business license**
- **Cost:** 1,000 × Copilot Business rate
- **GHE Seat Consumption:** 0 (Verified)

---

## ✓ Phase 4: Verification & Limits (Triple-Checked)

| Check | Verified Fact |
|:------|:--------------|
| **Team Capacity** | **5,000 Members.** The limit was increased from 500 to 5,000 per enterprise team in December 2025. |
| **License Consumption** | **Copilot Only.** Users in an enterprise team *without* org access do NOT consume a GHE/Visual Studio license. |
| **Billing Deduplication** | **Unique User.** If a user is already licensed for Copilot in an Org, adding them here generally does not double-bill; the system bills per unique user based on GitHub's combined billing model. |
| **Scope** | **Copilot Business Only.** This method does *not* grant Copilot Enterprise features (which require repo context/org access). |
| **Feature Status** | **Public Preview.** Enterprise Teams remain in Public Preview, though widely stable for this use case. |

---

## ⚠️ Common Pitfall to Avoid

### The "Upgrade" Trap

If you later decide to upgrade these users to **Copilot Enterprise** (to use Chat with your codebase), you will need to grant them access to repositories.

- **The Moment you do this:** You must add them to an Organization
- **The Cost:** They will immediately consume a **GitHub Enterprise license** + the **Copilot Enterprise upgrade** cost
- **Conclusion:** Keep them on **Copilot Business** if the goal is "standalone" usage (IDE completion/chat without repo context)

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

### Q: Users in the enterprise team are consuming GHE licenses — what went wrong?
**A:** The enterprise team was likely granted access to an organization. If the team has any org affiliation (even "Read" access), every member becomes a standard enterprise member and consumes a GHE license. Verify the team has no organization access under Enterprise > People > Enterprise teams > [your team] > Settings.

---

### Q: We want to upgrade standalone users to Copilot Enterprise — what's the cost impact?
**A:** Copilot Enterprise features require repository and org access. The moment you add standalone users to an organization, they consume a GitHub Enterprise license (approximately $21/month) in addition to the Copilot Enterprise upgrade cost. This is the "upgrade trap" — factor in the GHE seat cost before migrating.

---

### Q: Can we use IdP group sync and manual assignment together on the same enterprise team?
**A:** No. If you link an IdP group to an enterprise team, the team must have no manually assigned users. Membership becomes 100% automated by your IdP. Choose one method: IdP sync for long-term management or API/manual for initial setup.

---

### Q: We hit the enterprise team member limit — what's the cap?
**A:** The limit is 5,000 members per enterprise team (increased from 500 in December 2025). If you need to assign Copilot to more than 5,000 users in standalone mode, create multiple enterprise teams and assign licenses to each.

---

### Q: A user already has Copilot through an org — will adding them to the standalone enterprise team cause double billing?
**A:** No. GitHub's combined billing model bills per unique user. If a user is already licensed for Copilot in an org, adding them to the enterprise team generally does not double-bill. The system deduplicates based on unique user identity.

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Seat Assignment & Enablement | `Copilot/Seat Assignment & Enablement.md` |
| Admin Controls (Models, Content Exclusion, Instructions) | `Copilot/Admin Controls (Models, Content Exclusion, Instructions).md` |
| Enterprise Trial (GHEC, EMU, DRUS) | `Setup/Enterprise Trial (GHEC, EMU, DRUS).md` |

---

## 📝 Resources

1. [Creating enterprise teams](https://docs.github.com/en/enterprise-cloud@latest/enterprise-onboarding/setting-up-organizations-and-teams/creating-teams)
2. [REST API endpoints for enterprise team memberships](https://docs.github.com/en/enterprise-cloud@latest/rest/enterprise-teams/enterprise-team-members)
3. [Granting users access to GitHub Copilot in your enterprise](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/administer-copilot/manage-for-enterprise/manage-access/grant-access)
4. [Combined GitHub Enterprise cloud and server use](https://docs.github.com/en/enterprise-cloud@latest/billing/concepts/enterprise-billing/combined-enterprise-use)
5. [Enterprise teams product limits increased by over 10x](https://github.blog/changelog/2025-12-08-enterprise-teams-product-limits-increased-by-over-10x/)

---

---

*Last updated: April 2026*
