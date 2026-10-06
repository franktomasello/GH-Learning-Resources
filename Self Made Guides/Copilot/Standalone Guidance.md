# 🚀 GitHub Copilot Standalone Licensing Guide

> **How to assign Copilot Business licenses without consuming GitHub Enterprise (GHE) licenses**

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [📋 Overview](#-overview)
- [🔧 Phase 1: The Setup (Critical Configuration)](#-phase-1-the-setup-critical-configuration)
- [👥 Phase 2: Add Users to the Team](#-phase-2-add-users-to-the-team)
- [📜 Phase 3: Assign the License](#-phase-3-assign-the-license)
- [✓ Phase 4: Verification & Limits](#-phase-4-verification--limits)
- [⚠️ Common Pitfall to Avoid](#️-common-pitfall-to-avoid)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- Set the default policy: `Enterprise → AI controls → Copilot` → **Policies for enterprise-assigned users**
- Add people to the enterprise (personal accounts): `Enterprise → People → Invite member` — they join as **unaffiliated users**
- Create enterprise team: `Enterprise → People → Enterprise teams → Create Enterprise team` — **no organization access**
- Add users: **Add members** (UI), IdP group sync (EMU only), or `POST /enterprises/{enterprise}/teams/{team}/memberships/add`
- Assign licenses: `Enterprise → Billing and licensing → Licensing → Copilot → Manage → Enterprise Teams tab → Assign licenses → Add licenses`

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
| GitHub Enterprise Cloud enterprise with a Copilot Business plan | GitHub **enterprise owner** | ☐ |
| Users exist in the enterprise — **invited** (personal accounts) or **provisioned by SCIM** (EMU) | GitHub **enterprise owner** / IdP admin | ☐ |
| List of GitHub usernames for the standalone users | Program owner | ☐ |
| For IdP sync: an IdP group, and an **Enterprise Managed Users** enterprise | IdP group owner | ☐ |
| For the REST API: a **classic** PAT with `admin:enterprise` (writes) or `read:enterprise` (reads) | GitHub **enterprise owner** | ☐ |

---

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **GitHub enterprise owner** | Creates the enterprise team (no organization access), adds members, and assigns Copilot Business licenses. | Team: GitHub → Enterprise → People → Enterprise teams → Create Enterprise team → name → leave organization access empty → Create Enterprise team. Licenses: Enterprise → Billing and licensing → Licensing → Copilot → Manage → Enterprise Teams tab → Assign licenses → search the team → Add licenses. Handoff: team slug (`ent:...`) and licensed-user count. |
| **Microsoft Entra or Okta group owner, if team membership is synchronized from the IdP (EMU only)** | Maintains the source group that drives enterprise team membership. | Entra: Microsoft Entra admin center → Entra ID → Groups → [group] → Members → Add members → select users → Select. Okta: Okta Admin Console → Directory → Groups → [group] → People → Assign people → select users → Save. Make sure the group is assigned to the GitHub EMU app so SCIM pushes it. Handoff: group name and member count (5,000 max). |
| **Visual Studio subscriptions administrator, if Visual Studio subscriptions are linked** | Confirms which users have linked Visual Studio subscriptions — linked users consume a bundled Visual Studio license even when unaffiliated. | Visual Studio Admin Portal (`https://manage.visualstudio.com`) → Subscribers → search user → check subscription and GitHub link status. Handoff: list of linked subscribers. |

---

## 📋 Overview

Give Copilot Business to a large group (for example ~1,000 people) **without each person consuming a GitHub Enterprise license**.

**Why it works:** people who are in your enterprise but **not in any organization** are *unaffiliated users*. Unaffiliated users don't consume a GitHub Enterprise license, and an enterprise owner can assign them Copilot Business licenses directly — one at a time or through an enterprise team.

| Approach | Plan | GHE license? |
|----------|------|-------------|
| **Direct assignment** (enterprise → users or enterprise teams) | Copilot **Business** only | ❌ No — for unaffiliated users |
| **Organization assignment** | Copilot Business or Copilot Enterprise | ✅ Yes — organization members consume a license |

---

## 🔧 Phase 1: The Setup (Critical Configuration)

### Step 1: Set the default policy for enterprise-assigned users

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

1. Enterprise → **AI controls** → **Copilot** *(sidebar)*.
2. Find **Policies for enterprise-assigned users** and choose whether policies set to **Let organizations decide** default to **enabled** or **disabled** for these users.

> 📌 Users who get Copilot straight from the enterprise aren't in an organization, so "let organizations decide" policies can't reach them. This policy decides their defaults.

### Step 2: Get the users into the enterprise

- **Personal accounts:** Enterprise → **People** → **Members** page → **Invite member** → search users → **Invite**. They join as **unaffiliated users** after accepting the email (invitations expire after 7 days).
- **Enterprise Managed Users:** provision the users from your IdP with SCIM. Don't add them to any organization.

### Step 3: Create the enterprise team (the "zero-cost" step)

> ⚠️ **CRITICAL**: This step decides whether you pay for ~1,000 extra GHE licenses. Follow it exactly.

1. Enterprise → **People** → in the left sidebar, **Enterprise teams**.
2. Click **Create Enterprise team**.
3. **Name:** a functional name (for example `copilot-standalone-users`). GitHub creates the slug with an `ent:` prefix (`ent:copilot-standalone-users`).
4. **Organization access:** **leave it empty.**
5. Click **Create Enterprise team**.

#### 🛑 WARNING: Organization access
- **DO NOT** give the team access to any organization.
- **Why?** Team members are added directly to every organization the team can access. Unaffiliated users and outside collaborators in the team then become standard enterprise members — with access to internal repositories — and **consume a GitHub Enterprise license** (list price $21/user/month).
- **Correct state:** the team exists at the enterprise level only.

---

## 👥 Phase 2: Add Users to the Team

Choose the method that matches your setup. Each enterprise team holds up to **5,000** members, and an enterprise can have up to **2,500** teams.

### Option A: IdP Group Sync (Best long-term — EMU only)

1. In your IdP (Entra ID or Okta), create a group such as `github-copilot-users` and make sure it's pushed to GitHub through SCIM.
2. Enterprise → **People** → **Enterprise teams** → click your team.
3. Make sure the team has **no manually added members** (remove them with the **⋯** menu next to each name).
4. Next to the team name, click the **Edit** (pencil) icon.
5. Under **Manage members**, click **Identity provider group**.
6. Click **Select group** and choose the IdP group.
7. Click **Update team**.

> ⚠️ If the IdP group grows past **5,000** users, syncing stops until it's back under the limit.

### Option B: REST API (Best for bulk initial setup)

Requires a **classic** personal access token with `admin:enterprise` (fine-grained tokens and GitHub App tokens aren't supported).

**Bulk add (up to many users per call):**

```bash
curl -L -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer <YOUR-TOKEN>" \
  -H "X-GitHub-Api-Version: 2026-03-10" \
  https://api.github.com/enterprises/ENTERPRISE/teams/ent:copilot-standalone-users/memberships/add \
  -d '{"usernames":["monalisa","octocat"]}'
```

**Single user:** `PUT /enterprises/{enterprise}/teams/{enterprise-team}/memberships/{username}`

**Logic:**
1. Read the list of usernames (users must already be in the enterprise).
2. Send them in batches to the bulk add endpoint.
3. Re-check with `GET /enterprises/{enterprise}/teams/{enterprise-team}/memberships` and retry any that are missing.

### Option C: Manual UI (small groups only)

1. Enterprise → **People** → **Enterprise teams** → click the team.
2. Click **Add members**, search for and select users.
3. Click **Add**.

> 💡 Practical for small groups (for example under 50 users). Use Option A or B for hundreds of users.

---

## 📜 Phase 3: Assign the License

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **Billing and licensing** → **Licensing** *(sidebar)* → next to **Copilot**, **Manage**

1. At the top of the enterprise page, click **Billing and licensing**.
2. In the left sidebar, click **Licensing**.
3. Next to **Copilot**, click **Manage**.
4. Click the **Enterprise Teams** tab. *(To license individual people instead, use the **All members** tab.)*
5. Click **Assign licenses**.
6. Search for your team (`copilot-standalone-users`), then click **Add licenses**.

### ✅ Outcome
- Everyone in the team has a **Copilot Business** license. People added to or removed from the team gain or lose Copilot automatically.
- **Cost:** number of users × the Copilot Business rate, plus any metered AI credit usage.
- **GHE license consumption:** none for unaffiliated users (linked Visual Studio subscribers use a bundled Visual Studio license).

> 💡 Optional: to stop organization owners from assigning their own seats, disable Copilot for organizations on the same **Manage** page.

---

## ✓ Phase 4: Verification & Limits

| Check | Fact |
|:------|:-----|
| **Team capacity** | **5,000 members** per enterprise team; up to **2,500** teams per enterprise. |
| **License consumption** | Unaffiliated users (in the enterprise, not in any organization) **don't** consume a GitHub Enterprise license. Exception: users linked to a Visual Studio subscription consume a bundled Visual Studio license. |
| **Plan** | Direct assignment is **Copilot Business only**. Copilot Enterprise is assigned through organizations. |
| **Policies** | Enterprise-assigned users follow enterprise policies, with **Policies for enterprise-assigned users** setting the defaults. |
| **Feature status** | Enterprise teams are **generally available** (since June 2026). |
| **Verify** | Enterprise → **People** → **Members** shows the users as unaffiliated (no organizations). **Billing and licensing** → **Licensing** shows the Copilot license count. |

---

## ⚠️ Common Pitfall to Avoid

### The "Upgrade" Trap

Copilot Enterprise can't be assigned directly to users or enterprise teams — it's assigned through organizations.

- **The moment you upgrade:** the users must become members of an organization.
- **The cost:** each one then consumes a **GitHub Enterprise license** *plus* the Copilot Enterprise rate.
- **Conclusion:** keep standalone users on **Copilot Business** unless the Copilot Enterprise features are worth the extra GHE seat.

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
| **Copilot stops working for a user mid-cycle** | The user's user-level budget is used up, the shared AI credit pool is exhausted with **AI credits paid usage** disabled, or a spending limit with **Stop usage** was reached. | Check the user on **Billing and licensing** → **AI usage** and the budgets on **Budgets and alerts**; raise their budget, approve their budget request (not available for EMU enterprises), or enable AI credits paid usage. |
| **Content exclusions do not apply immediately** | Client policy cache, unsupported surface/mode, symlink/remote filesystem limitation, or indirect IDE context. | Reload the IDE policy, verify the exclusion syntax at enterprise/org/repo scope, and document surfaces where exclusions are limited. |
| **Usage metrics look empty or inconsistent** | Telemetry is disabled, data freshness delay applies, users are unlicensed, or different APIs report different scopes. | Enable the metrics policy, confirm seats and telemetry, wait for data freshness, and avoid comparing dashboards/API endpoints as if they share identical data models. |
| **Cloud agent or MCP action is denied** | Agent policy, MCP policy, repository permissions, secrets, or server allowlist does not permit the operation. | Review Enterprise AI controls > Agents/MCP, repo-level permissions, MCP server configuration, and audit logs for the denied action. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>


### Q: Users in the enterprise team are consuming GHE licenses — what went wrong?
**A:** Most likely the team was given access to an organization, which made every member a standard enterprise member. Check the team's organization access (Enterprise → **People** → **Enterprise teams** → the team → **Edit**) and remove it. Also check whether the users were added to any organization some other way. Under usage-based billing, anyone who consumed a license during the cycle stays billable for that cycle.

---

### Q: We want to upgrade standalone users to Copilot Enterprise — what's the cost impact?
**A:** Copilot Enterprise is only assigned through organizations, so the users must join an organization. Each one then consumes a GitHub Enterprise license (list price $21/user/month) on top of the Copilot Enterprise rate. Budget for the GHE seat before you move them.

---

### Q: Can we use IdP group sync and manual assignment together on the same enterprise team?
**A:** No. A team synced to an IdP group can't have manually added members — remove them before linking the group, and membership is then managed entirely by the IdP. IdP sync is only available with Enterprise Managed Users.

---

### Q: We hit the enterprise team member limit — what's the cap?
**A:** 5,000 members per enterprise team (up to 2,500 teams per enterprise). For more than 5,000 standalone users, create several teams and assign licenses to each.

---

### Q: A user already has Copilot through an org — will adding them to the standalone enterprise team cause double billing?
**A:** GitHub bills a Copilot seat once per unique user per billing cycle within an enterprise, even when several organizations assign one. If a user has both Business and Enterprise seats, only Copilot Enterprise is billed. To keep things clean, don't license the same person both ways — and remember that a user in an organization already consumes a GHE license, so they aren't "standalone" anyway.

</details>

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Seat Assignment & Enablement | `Copilot/Seat Assignment & Enablement.md` |
| Admin Controls (Models, Content Exclusion, Instructions) | `Copilot/Admin Controls (Models, Content Exclusion, Instructions).md` |
| Enterprise Trial (GHEC, EMU, DRUS) | `Setup/Enterprise Trial (GHEC, EMU, DRUS).md` |

---

## 📝 Resources

1. [Creating enterprise teams](https://docs.github.com/en/enterprise-cloud@latest/admin/managing-accounts-and-repositories/managing-users-in-your-enterprise/create-enterprise-teams)
2. [REST API endpoints for enterprise team members](https://docs.github.com/en/enterprise-cloud@latest/rest/enterprise-teams/enterprise-team-members)
3. [Granting users access to GitHub Copilot in your enterprise](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/administer-copilot/manage-for-enterprise/manage-access/grant-access)
4. [People who consume a GitHub Enterprise license](https://docs.github.com/en/enterprise-cloud@latest/billing/reference/github-license-users)
5. [Adding users to your enterprise](https://docs.github.com/en/enterprise-cloud@latest/admin/managing-accounts-and-repositories/managing-users-in-your-enterprise/add-users)
6. [Copilot seat assignment and billing](https://docs.github.com/en/copilot/reference/copilot-billing/seat-assignment)

---

*Last updated: October 2026*
