# 💺 GitHub Copilot Seat Assignment & Enablement Runbook

> **Complete guide to assigning Copilot seats across enterprise, organization, and team levels including pilot rollouts and mixed-plan scenarios**

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [📋 Overview](#-overview)
- [1️⃣ Enable Copilot for Specific Organizations (Enterprise Level)](#1️⃣-enable-copilot-for-specific-organizations-enterprise-level)
- [2️⃣ Assign Seats to Specific Users (Organization Level)](#2️⃣-assign-seats-to-specific-users-organization-level)
- [3️⃣ Team-Based Assignment for Pilots](#3️⃣-team-based-assignment-for-pilots)
- [4️⃣ Enable Copilot for All Members (Organization Level)](#4️⃣-enable-copilot-for-all-members-organization-level)
- [5️⃣ Mixed Plans: Business + Enterprise in the Same Enterprise](#5️⃣-mixed-plans-business--enterprise-in-the-same-enterprise)
- [6️⃣ Resolving Duplicate Tier Assignments](#6️⃣-resolving-duplicate-tier-assignments)
- [7️⃣ Copilot Enterprise vs Business Assignment at the Enterprise Level](#7️⃣-copilot-enterprise-vs-business-assignment-at-the-enterprise-level)
- [🚀 Quick Pilot Rollout Recipe](#-quick-pilot-rollout-recipe)
- [📝 Additional Notes](#-additional-notes)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📚 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Turn Copilot on for orgs (and pick the plan per org):** Enterprise → **Billing and licensing** → **Licensing** → **Manage** *(in the "Copilot" section)* → **Organization access** → **Allow for specific organizations** → **Organizations** tab → org's **Copilot** dropdown
- **Seats for specific users or teams:** Organization → **Settings** → **Copilot** *(sidebar, under "Code, planning, and automation")* → **Access** → **Start adding seats** → **Purchase for selected members** → **Users and teams** → **Continue to purchase** → **Purchase seats**
- **Seats for everyone in an org:** Organization → **Settings** → **Copilot** *(sidebar, under "Code, planning, and automation")* → **Access** → **Start adding seats** → **Purchase for all members** → **Purchase seats**
- **License users directly at the enterprise (Copilot Business):** Enterprise → **Billing and licensing** → **Licensing** → **Manage** *(in the "Copilot" section)* → **All members** or **Enterprise Teams** → **Assign licenses** → **Add licenses**

---

## ✅ Accuracy & Click-Path Notes

<details>
<summary><em>Show click-path conventions</em></summary>


- Reviewed against current public GitHub documentation in October 2026 where public documentation is available. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
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
| GitHub Enterprise Cloud enterprise with Copilot Business or Copilot Enterprise | GitHub **enterprise owner** | ☐ |
| Turn Copilot on for organizations and choose each org's plan | GitHub **enterprise owner** | ☐ |
| Assign seats inside an organization | GitHub **organization owner** | ☐ |
| GitHub teams created (for team-based pilots) | GitHub **organization owner** or team maintainer | ☐ |

---

## 📋 Overview

This runbook covers every way to give people GitHub Copilot:

| Method | Who does it | Best for |
|--------|-------------|----------|
| **Turn Copilot on for selected organizations** | Enterprise owner | Choosing which orgs can use Copilot, and on which plan |
| **Seats for selected members** | Organization owner | Controlled rollouts, budget management |
| **Seats for a team** | Organization owner | Pilots and department rollouts |
| **Seats for all members** | Organization owner | Full-organization enablement |
| **Direct enterprise licenses** | Enterprise owner | Copilot Business for people with no org access needed |

> 💡 **Billing:** a seat is billed from the moment it's granted (prorated mid-cycle), whether or not the person uses Copilot. Since **October 1, 2026**, customers who pay by **credit card or PayPal** must pay for each new seat before the user gets access, and assigned seats incur an upfront charge each billing cycle.

---

## 1️⃣ Enable Copilot for Specific Organizations (Enterprise Level)

*Choose which organizations in your enterprise can use Copilot*

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **Billing and licensing** → **Licensing** → **Manage** *(in the "Copilot" section)*

**Steps:**

1. At the top of the enterprise page, click **Billing and licensing**.
2. In the sidebar, click **Licensing**.
3. In the "Copilot" section, click **Manage**.
4. Next to **Organization access**, open the dropdown and select **Allow for specific organizations** (or enable it for all organizations).
5. Click the **Organizations** tab.
6. Find each organization and, to the right of its name, open the **Copilot** dropdown and click **Enabled** (or **Copilot: Business** / **Copilot: Enterprise** if your enterprise has a Copilot Enterprise plan).

> 📌 These selections apply immediately — there is no Save button.

> 💡 **Tip:** Start with a single organization for your pilot, then expand as adoption grows.

---

## 2️⃣ Assign Seats to Specific Users (Organization Level)

*Grant Copilot to individual members of an organization*

**👤 Role:** GitHub **organization owner** · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Organizations** → *[organization]* → **Settings** → **Copilot** → **Access**

**Steps:**

1. If you see **Allow this organization to assign seats**, click it.
2. Click **Start adding seats**.
3. Select **Purchase for selected members**.
4. In the "Enable Copilot access for users and teams" dialog, on the **Users and teams** tab, search for and select the users. *(To add many at once, use the **Upload CSV** tab.)*
5. Click **Continue to purchase**, then **Purchase seats**.

> ⚠️ **Important:** Each assigned seat is billed, even if the person never uses Copilot. Revoke unused seats to stop the charge.

---

## 3️⃣ Team-Based Assignment for Pilots

*Recommended approach for running a Copilot pilot*

### A) Create a GitHub Team

**👤 Role:** GitHub **organization owner** · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Organizations** → *[organization]* → **Teams** → **New team**

**Steps:**

1. Enter a team name (e.g., `copilot-pilot`).
2. Add a description (e.g., "Copilot pilot participants").
3. Set visibility to **Visible** or **Secret**.
4. Click **Create team**.

### B) Add Pilot Users to the Team

**Navigate:** Profile picture → **Organizations** → *[organization]* → **Teams** → *[copilot-pilot]*

1. Click **Add a member**.
2. Search for the user, select them, and confirm.
3. Repeat for each pilot user.

> 📌 **Enterprise Managed Users / team sync:** if the team is linked to an IdP group, add people to the group in your identity provider instead — GitHub updates the team for you.

### C) Grant Copilot Access to the Team

**Navigate:** Profile picture → **Organizations** → *[organization]* → **Settings** → **Copilot** → **Access**

1. Click **Start adding seats** (click **Allow this organization to assign seats** first if it appears).
2. Select **Purchase for selected members**.
3. On the **Users and teams** tab, search for `copilot-pilot` and select the team.
4. Click **Continue to purchase**, then **Purchase seats**.

> ✅ **Result:** members of `copilot-pilot` get Copilot, and people you add to the team later get a seat automatically.

> 💡 **Tip:** Team-based assignment makes it easy to add or remove pilot participants and to report usage by team.

---

## 4️⃣ Enable Copilot for All Members (Organization Level)

*Grant Copilot to every current and future member of the organization*

**👤 Role:** GitHub **organization owner** · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Organizations** → *[organization]* → **Settings** → **Copilot** → **Access**

**Steps:**

1. Click **Start adding seats** (click **Allow this organization to assign seats** first if it appears).
2. Select **Purchase for all members**.
3. In the "Confirm seats purchase for all members" dialog, click **Purchase seats**.

> ⚠️ **Important:** Every current member gets a seat, and new members get one automatically when they join. Monitor your seat count.

---

## 5️⃣ Mixed Plans: Business + Enterprise in the Same Enterprise

*Give different organizations different Copilot plans*

### Understanding Mixed Plans

| Plan | Price | AI credits per user per month | What's different |
|------|-------|-------------------------------|------------------|
| **Copilot Business** | $19 per seat | 1,900 | Chat, inline suggestions, agents (cloud agent, agent mode, code review), MCP, custom instructions, content exclusion, policy management |
| **Copilot Enterprise** | $39 per seat | 3,900 | Everything in Business, plus a larger AI credit allowance, access to some additional models, and Spark (public preview) |

> 💡 Under usage-based billing, Copilot Enterprise isn't cheaper for heavy users — its extra credits cost exactly what you pay for them. Choose it for its features. See [AI Credits Budgeting Scenarios](AI%20Credits%20Budgeting%20Scenarios.md).

### Assign Tiers at the Enterprise Level

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **Billing and licensing** → **Licensing** → **Manage** *(in the "Copilot" section)* → **Organizations** tab

**Steps:**

1. Find the organization.
2. Open its **Copilot** dropdown and click **Copilot: Business** or **Copilot: Enterprise**. *(Applies immediately.)*

> 💡 **Tip:** You can run some organizations on Copilot Business and others on Copilot Enterprise under the same enterprise. The Copilot Enterprise options appear only if your enterprise has a Copilot Enterprise plan.

---

## 6️⃣ Resolving Duplicate Tier Assignments

*When a user gets Copilot through more than one route*

### How Duplicates Happen

A user can be assigned Copilot more than once — for example, as a member of two organizations on different plans, or through both an organization and a direct enterprise license.

### Resolution Rules

| Scenario | Result |
|----------|--------|
| Business in Org A + Enterprise in Org B | The user gets **Copilot Enterprise** (the highest plan) and consumes **one** license |
| Business in Org A + Business in Org B | The user consumes **one** Business license |
| Organization seat + direct enterprise license | The user consumes **one** license, at the highest plan assigned |

### Identifying Duplicate Assignments

**Navigate:** Enterprise → **Billing and licensing** → **Licensing** → **Manage** *(in the "Copilot" section)* → **All members** tab

Review who holds a license and how it was assigned, and compare it with each organization's **Settings** → **Copilot** → **Access** page.

> 💡 **Tip:** If a user already has Copilot Enterprise through one organization, removing their lower-plan seat elsewhere makes the assignments easier to reason about.

---

## 7️⃣ Copilot Enterprise vs Business Assignment at the Enterprise Level

*Manage which plan each organization uses*

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **Billing and licensing** → **Licensing** → **Manage** *(in the "Copilot" section)* → **Organizations** tab

**Options:**

| Setting on an org's **Copilot** dropdown | Effect |
|------------------------------------------|--------|
| **Copilot: Business** | Seats in that org are Copilot Business |
| **Copilot: Enterprise** | Seats in that org are Copilot Enterprise |
| **Enabled** *(enterprises on Copilot Business only)* | Copilot is on for that org |

> ⚠️ **Important:** Moving an organization from Copilot Enterprise to Copilot Business removes Enterprise-only capabilities for its users right away, and its contribution to the shared AI credit pool drops at the start of the next billing cycle.

---

## 🚀 Quick Pilot Rollout Recipe

*Recommended steps for a controlled Copilot pilot:*

### Steps

1. **Turn Copilot on for the pilot organization** — Enterprise → **Billing and licensing** → **Licensing** → **Manage** *(in the "Copilot" section)* → **Allow for specific organizations** → **Organizations** tab → pilot org's **Copilot** dropdown → **Enabled**.
2. **Create a pilot team** — Organization → **Teams** → **New team** → `copilot-pilot` → **Create team**.
3. **Add pilot users to the team** (50–100 is a good size).
4. **Grant Copilot to the team** — Organization → **Settings** → **Copilot** *(sidebar, under "Code, planning, and automation")* → **Access** → **Start adding seats** → **Purchase for selected members** → select `copilot-pilot` → **Continue to purchase** → **Purchase seats**.
5. **Set policies and budgets** — model access, public-code matching, content exclusions, and a universal user-level budget (see [AI Credits Budget & Overage Planning](AI%20Credits%20Budget%20%26%20Overage%20Planning.md)).
6. **Run for 30–60 days**, then review usage on Enterprise → **Billing and licensing** → **AI usage** and your adoption metrics.

---

## 📝 Additional Notes

> 💡 **Customization:** UI labels can vary slightly depending on your enterprise agreement, plan, and trial status. The paths above reflect GitHub's documentation as of October 2026.

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


### Q: A user says Copilot isn't working even though we assigned them a seat — what should we check?
**A:** Check, in order: (1) Copilot is turned on for their organization at the enterprise level (Step 1); (2) they're signed in to the right GitHub account in their IDE and have an active SSO session if the organization uses SAML; (3) their IDE and Copilot extension are up to date — older versions can show wrong usage and billing information; (4) the feature they're using is allowed by your Copilot policies; and (5) they haven't used up their AI credit budget — they can see this under **Your Copilot** → **Usage**.

---

### Q: Can we auto-assign Copilot to all new org members?
**A:** Yes. In **Settings** → **Copilot** → **Access**, click **Start adding seats** → **Purchase for all members** → **Purchase seats**. Every current member gets a seat, and new members get one automatically when they join. Each seat is billed, so monitor your seat count.

---

### Q: A user has both Copilot Business and Enterprise assigned — what happens?
**A:** They get Copilot Enterprise (the highest plan assigned) and consume only **one** license, so there's no double charge. You can still remove the extra assignment to keep things tidy.

---

### Q: How do we revoke a seat immediately?
**A:** It depends where the seat came from. **Enterprise-level licenses** (direct assignment or an enterprise team): unassign the license or remove the user from the enterprise team — access is revoked **immediately**. **Organization seats:** in **Settings** → **Copilot** → **Access**, select the member's checkbox, click **Cancel seat**, then **Remove seats** in the confirmation dialog — the user keeps access until the **start of the next billing cycle**. Removing the user from the organization also revokes the organization seat.

---

### Q: Seats are showing as "pending" — what does that mean?
**A:** Usually the person wasn't a member of the organization when you assigned the seat, so GitHub sent them an organization invitation that they haven't accepted yet. Check the organization's pending invitations and ask them to accept.

---

### Q: We removed a user from the org but they still appear to have a Copilot seat — why?
**A:** They may still have Copilot through another organization, an enterprise team, or a direct enterprise license. Check Enterprise → **Billing and licensing** → **Licensing** → **Manage** (*Copilot*) → **All members**. Organization-level removals also take effect from the start of the next billing cycle.

---

### Q: How many seats can we assign during a pilot without over-purchasing?
**A:** There's no fixed pool of seats to over-buy: you're billed for each seat you grant, prorated for the rest of the cycle. Grant seats only to a dedicated pilot team (e.g., `copilot-pilot`), and cancel seats for people who stop using Copilot.

</details>

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Admin Controls (Models, Content Exclusion, Instructions) | `Copilot/Admin Controls (Models, Content Exclusion, Instructions).md` |
| AI Credits Budget & Overage Planning | `Copilot/AI Credits Budget & Overage Planning.md` |
| Measuring Adoption & ROI | `Copilot/Measuring Adoption & ROI.md` |
| Enterprise Trial (GHEC, EMU, DRUS) | `Setup/Enterprise Trial (GHEC, EMU, DRUS).md` |

---

## 📚 Resources

- [Granting users access to Copilot in your enterprise](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-enterprise/manage-access/grant-access)
- [Granting access to Copilot for members of your organization](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-organization/manage-access/grant-access)
- [Revoking access to Copilot for members of your organization](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-organization/manage-access/revoke-access)
- [Plans for GitHub Copilot](https://docs.github.com/en/copilot/get-started/plans)
- [Managing policies and features for Copilot in your enterprise](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-enterprise/manage-enterprise-policies)

---

*Last updated: October 2026*
