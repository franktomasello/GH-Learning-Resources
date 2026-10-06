# 🔗 Visual Studio Subscription Linking for GitHub Enterprise Runbook

> **How Visual Studio subscriptions with GitHub Enterprise get matched to GitHub accounts — assignment, invitation, verification, and manual reconciliation**

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [📋 Overview](#-overview)
- [1️⃣ Step 1 — Assign the Subscription in the Visual Studio Admin Portal](#1️⃣-step-1--assign-the-subscription-in-the-visual-studio-admin-portal)
- [2️⃣ Step 2 — Invite the Subscriber to a GitHub Organization](#2️⃣-step-2--invite-the-subscriber-to-a-github-organization)
- [3️⃣ Step 3 — Subscriber Accepts the Invitation](#3️⃣-step-3--subscriber-accepts-the-invitation)
- [4️⃣ Step 4 — Verify and Reconcile Licenses on GitHub](#4️⃣-step-4--verify-and-reconcile-licenses-on-github)
- [5️⃣ Common Issues & Troubleshooting](#5️⃣-common-issues--troubleshooting)
- [6️⃣ Important Caveats](#6️⃣-important-caveats)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📚 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Assign (VS admin):** `https://manage.visualstudio.com` → agreement → **Subscribers** → **Add** → assign a **Visual Studio … with GitHub Enterprise** subscription to the user's UPN
- **Invite (org owner):** `Org → People → Invite member` → the subscriber's **UPN email** → **Invite** (EMU: provision with SCIM instead)
- **Verify (enterprise owner):** `Enterprise → Billing and licensing → Licensing` → next to **Enterprise Cloud**, **Manage** → check the license type
- **Fix a mismatch:** same list → **⋯** next to the user → **Change to Visual Studio license** → pick the VS email → **Confirm change**

---

## ✅ Accuracy & Click-Path Notes

<details>
<summary><em>Show click-path conventions</em></summary>


- Reviewed against current public GitHub and Microsoft documentation in October 2026 where public documentation is available. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
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
| A Microsoft Enterprise Agreement with **Visual Studio Enterprise with GitHub Enterprise** or **Visual Studio Professional with GitHub Enterprise** subscriptions | Microsoft licensing / procurement | ☐ |
| Assign subscriptions to people | **Visual Studio subscriptions admin** | ☐ |
| An enterprise on GitHub with at least one organization | GitHub **enterprise owner** | ☐ |
| Invite subscribers to an organization | GitHub **organization owner** | ☐ |
| Verify and manually match licenses | GitHub **enterprise owner** | ☐ |
| Each subscriber's UPN (work email) | Visual Studio subscriptions admin | ☐ |

---

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **Visual Studio subscriptions administrator** | Assigns the combined Visual Studio + GitHub Enterprise subscription to the right corporate identity. | Visual Studio Subscriptions Admin Portal (`https://manage.visualstudio.com`) → select the agreement → Subscribers → Add → enter the subscriber's name and UPN email → choose the Visual Studio … with GitHub Enterprise subscription → Add. For existing subscribers, move them to the combined offering. Handoff: subscriber UPN list and subscription level. |
| **GitHub organization owner** | Invites each subscriber to an organization in the enterprise. | GitHub → Organizations → [organization] → People → Invite member → type the subscriber's UPN email → choose role (Member) and teams → Send invitation. For many users, script it with the REST API. Handoff: list of invitations sent. |
| **GitHub enterprise owner** | Confirms each member consumes a Visual Studio license and fixes mismatches. | GitHub → Enterprise → Billing and licensing → Licensing → next to Enterprise Cloud, Manage → review license types → for an "Enterprise" user who should be on VS: ⋯ → Change to Visual Studio license → select the VS login email → Confirm change. Handoff: license list or CSV export with all subscribers matched. |

---

## 📋 Overview

**Visual Studio subscriptions with GitHub Enterprise** is a combined Microsoft offering (under an Enterprise Agreement) that gives each subscriber Visual Studio **and** a GitHub Enterprise license.

| Offering | Includes GitHub Enterprise? |
|---|---|
| **Visual Studio Enterprise with GitHub Enterprise** | ✅ Yes |
| **Visual Studio Professional with GitHub Enterprise** | ✅ Yes |
| **Visual Studio Enterprise** or **Professional** on their own | ❌ No — GitHub Enterprise is a separate purchase |
| **GitHub Copilot** | ❌ Licensed separately |

**How matching works:** a subscriber uses the GitHub Enterprise part of the license by **joining an organization in your enterprise**. GitHub automatically matches the GitHub account to the Visual Studio subscription when a **verified email on the GitHub account matches the subscriber's UPN**. With **Enterprise Managed Users**, the UPN must match the SCIM `userName` (or the linked identity's email).

> ⚠️ **Important:** subscribers **can't self-assign** or self-link the benefit — there's no "link" button on GitHub. An admin assigns the subscription, and an organization owner invites the person.

---

## 1️⃣ Step 1 — Assign the Subscription in the Visual Studio Admin Portal

**👤 Role:** **Visual Studio subscriptions admin** · **📍 Portal:** `https://manage.visualstudio.com`

**Steps:**

1. Sign in and select the agreement.
2. Open **Subscribers** and click **Add** (individual subscriber).
3. Enter the person's name and **work email (UPN)**.
4. Choose the **Visual Studio Enterprise with GitHub Enterprise** or **Visual Studio Professional with GitHub Enterprise** subscription.
5. Click **Add**.

**Check the result:**

| Check | What to confirm |
|-------|-----------------|
| **Subscription** | The combined offering "… with GitHub Enterprise" |
| **Email** | The person's corporate UPN — not a personal address |
| **Status** | Active |

> 💡 If people were assigned plain Visual Studio before GitHub Enterprise was added to your agreement, move them to the combined offering in the admin portal. Unless notifications are turned off, each subscriber gets two confirmation emails.

---

## 2️⃣ Step 2 — Invite the Subscriber to a GitHub Organization

**👤 Role:** GitHub **organization owner** (enterprise owner creates the organization first) · **📍 Portal:** GitHub

**Navigate:** Organization → **People** → **Invite member**

**Steps:**

1. Click **Invite member**.
2. Type the subscriber's **UPN email address** (recommended — it lets GitHub match the license and stops another enterprise from claiming it).
3. Choose the role (**Member**) and any teams.
4. Click **Send invitation**.

> 📌 **Enterprise Managed Users:** you don't invite — provision the user from your IdP with SCIM. Make sure the Visual Studio UPN matches the SCIM `userName` (or the linked identity's email).

> 💡 For many subscribers, script the invitations with the REST API (GitHub publishes a sample PowerShell script in `github/platform-samples`).

---

## 3️⃣ Step 3 — Subscriber Accepts the Invitation

**👤 Role:** The **subscriber** · **📍 Portal:** Email + GitHub

1. Open the GitHub invitation email and accept it — with an existing personal account or a new one.
2. If using an existing account, add the Visual Studio (UPN) email to the account: **Settings** → **Emails** → **Add email address** → verify it.
3. After joining, the **GitHub Enterprise** tile at `https://my.visualstudio.com/benefits` updates.

> ✅ **Result:** the person is now an enterprise member using a Visual Studio license instead of a separately purchased GitHub Enterprise license.

> ⚠️ Under the terms of use, the GitHub account and the Visual Studio subscription must belong to the **same person**.

---

## 4️⃣ Step 4 — Verify and Reconcile Licenses on GitHub

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **Billing and licensing** → **Licensing** → next to **Enterprise Cloud**, **Manage**

### Verify

- Review each user's **license type**. Users matched to Visual Studio show a Visual Studio license; users showing **Enterprise** aren't matched.
- Pending invitations include subscribers who haven't joined an organization yet. You can also see pending invitations in the Visual Studio admin portal.
- *(Optional)* Download the license CSV. Rows missing **Name** or **Profile** are people who haven't accepted an invitation yet.

### Match a user manually

1. Find the user with an **Enterprise** license type who should be on Visual Studio.
2. Click **⋯** → **Change to Visual Studio license**.
3. Select the user's Visual Studio login email.
4. Click **Confirm change**.

> 📌 Automatically matched users can't be re-mapped. Audit regularly: download the assigned-users summary from the Visual Studio portal and compare it with your enterprise members' verified emails.

---

## 5️⃣ Common Issues & Troubleshooting

| Issue | Cause | Resolution |
|-------|-------|------------|
| **User shows an "Enterprise" license instead of Visual Studio** | No verified GitHub email matches the UPN | Have the user add and verify the UPN email, or match them manually (Step 4) |
| **Subscriber can't see the GitHub Enterprise benefit** | Setup isn't finished — they haven't been invited or haven't joined an organization | Org owner sends the invitation; the subscriber accepts it |
| **EMU user consumes a paid license** | UPN doesn't match the SCIM `userName` or linked identity email | Fix the IdP attribute mapping or the Visual Studio assignment |
| **Another enterprise claimed the license** | The subscriber joined another enterprise with a matching email | Invite using the UPN and reconcile; contact GitHub Support if needed |
| **Access lost after subscription change** | The Visual Studio subscription expired, was removed, or reassigned | Restore the assignment, or the user consumes a standard GitHub Enterprise license (or is removed) |

---

## 6️⃣ Important Caveats

- **Licenses are counted together.** Your total GitHub Enterprise licenses = standard licenses + Visual Studio subscription licenses that include GitHub.
- **One person, one license.** A subscriber matched to Visual Studio consumes the Visual Studio license, not an extra standard one.
- **Mixing is fine.** Some members can be on Visual Studio licenses and others on standard or usage-based GitHub Enterprise licenses — customers with a Visual Studio bundle can switch non-subscribers to usage-based billing.
- **Unaffiliated users count too.** Unaffiliated enterprise members linked to a Visual Studio subscription consume a bundled Visual Studio license.
- **GitHub Enterprise Server:** a subscriber consumes one license as long as their GHES email matches their UPN (and Cloud/Server license sync is set up for users on both).

## 🧯 Known Errors & Resolutions

<details>
<summary><em>Show known errors table</em></summary>


> This section lists the known product errors and admin-facing symptoms that commonly occur with this workflow. Exact message text can vary by product rollout, tenant policy, and provider, so use the log or settings page named in the resolution to confirm the root cause.

| Error or symptom | Likely cause | Resolution |
|------------------|--------------|------------|
| **Page, tab, or button is missing** | Wrong account context, missing admin role, unavailable plan/add-on, or feature rollout not enabled for the selected enterprise/org/repo. | Switch to the correct account and scope, confirm the prerequisite role, verify licensing or add-on activation, then refresh the page. If the control is still absent, use the direct settings URL from the relevant GitHub Docs page and confirm the feature is available for your plan. |
| **Changes appear saved but behavior does not change** | Policy inheritance, cached UI state, propagation delay, or an overlapping enterprise/org/repo policy. | Reopen the settings page, verify the effective policy at the lowest affected scope, wait for propagation where documented, and check for a stricter policy at an enterprise or organization level. |
| **403, forbidden, or resource not accessible** | The signed-in user or token can see the page but lacks the specific permission for the action. | Use an enterprise owner, organization owner, repository admin, or token with the exact scopes/permissions listed in the runbook. For SAML-protected orgs, authorize the token or SSH key for SSO before retrying. |
| **Budget does not block usage** | Budget is alert-only, scoped to the wrong account/resource/SKU, or another budget is the one applying. | Edit or recreate the budget with the correct scope and SKU. For metered products, enable `Stop usage when budget limit is reached` if you need a hard stop, then verify there are no overlapping budgets with a different behavior. |
| **Usage report or cost center shows zero usage** | Usage has not processed yet, resources are not assigned to the cost center, or the report date range is wrong. | Confirm the cost center resources, select a date range after assignment, and wait for normal billing data latency before reconciling. |
| **Azure subscription is not listed or connection fails** | The Azure user lacks subscription owner rights or tenant-wide consent is required. | Sign in with an Azure subscription owner who can grant consent, or have an Entra global administrator approve the GitHub Subscription Permission Validation app, then repeat the Add Azure Subscription flow. |
| **Admin approval required during Azure billing connection** | The tenant blocks user consent for the GitHub billing app. | Use the tenant admin consent workflow or have a Global Administrator grant consent, then return to GitHub and select the subscription again. |
| **GitHub Enterprise benefit tile isn't active for a subscriber** | The subscription isn't a "… with GitHub Enterprise" offering, or the subscriber hasn't joined an organization yet. | Check the assignment in the Visual Studio Admin Portal, then have an organization owner invite the subscriber (by UPN) and the subscriber accept. |
| **Subscription cannot be matched to the GitHub account** | No verified email on the GitHub account matches the subscriber's UPN. | Add and verify the UPN email on the GitHub account, or match the user manually with **Change to Visual Studio license**. |
| **User shows a standard Enterprise license instead of Visual Studio** | The accounts weren't matched automatically, or the wrong GitHub account joined. | Confirm the right person accepted the invitation, then use **Change to Visual Studio license** on the Licensing page. |
| **Benefit disappeared after working previously** | The Visual Studio subscription expired, was reassigned, or the bundled benefit changed. | Recheck the subscriber record in the Visual Studio Admin Portal and restore the eligible assignment or replace the seat with a paid GitHub Enterprise license. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>


### Q: A user is looking for a "Link Visual Studio subscription" button in GitHub. Where is it?
**A:** There isn't one — subscribers can't self-link. The Visual Studio admin assigns a "… with GitHub Enterprise" subscription, an organization owner invites the person (ideally by their UPN), and GitHub matches the accounts when a verified email equals the UPN. An enterprise owner can match the rest manually.

---

### Q: A user seems to be using a standard license instead of their Visual Studio license. How do we fix this?
**A:** Go to Enterprise → **Billing and licensing** → **Licensing** → **Manage** next to Enterprise Cloud. If the user's license type is **Enterprise**, click **⋯** → **Change to Visual Studio license**, choose their Visual Studio login email, and click **Confirm change**. A matched user consumes only the Visual Studio license.

---

### Q: A user's Visual Studio subscription lapsed. What happens to their GitHub Enterprise access?
**A:** They're no longer covered by a Visual Studio license. If they stay in your organizations, they consume a standard GitHub Enterprise license instead (or are billed under usage-based licensing). Restore the Visual Studio assignment, or remove them from the enterprise if they shouldn't have access.

---

### Q: The Visual Studio email doesn't match the user's GitHub email. How do we resolve this?
**A:** Have the user add the UPN email to their GitHub account (**Settings** → **Emails** → **Add email address**) and verify it — GitHub then matches automatically. Or an enterprise owner matches them manually with **Change to Visual Studio license**. With EMU, fix the SCIM `userName` mapping instead.

---

### Q: Does Visual Studio Professional include GitHub Enterprise?
**A:** Only as the combined **Visual Studio Professional with GitHub Enterprise** offering. Plain Visual Studio Professional or Enterprise subscriptions don't include GitHub Enterprise — it's the "… with GitHub Enterprise" combined offering that does.

---

### Q: The wrong GitHub account was matched to a subscriber's license. How do we fix it?
**A:** Under the terms of use, the GitHub account and the subscription must belong to the same person. Remove the wrong account from your organizations, invite the correct account using the subscriber's UPN, and — if it isn't matched automatically — use **Change to Visual Studio license** for the correct account.

</details>

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Cost Centers & Department Billing | `Billing/Cost Centers & Department Billing.md` |
| Copilot Seat Assignment & Enablement | `Copilot/Seat Assignment & Enablement.md` |
| Enterprise Trial (GHEC, EMU, DRUS) | `Setup/Enterprise Trial (GHEC, EMU, DRUS).md` |

---

## 📚 Resources

- [Visual Studio subscriptions with GitHub Enterprise (Microsoft)](https://learn.microsoft.com/en-us/visualstudio/subscriptions/access-github)
- [About Visual Studio subscriptions with GitHub Enterprise](https://docs.github.com/en/billing/concepts/enterprise-billing/visual-studio-subs)
- [Setting up Visual Studio subscriptions with GitHub Enterprise](https://docs.github.com/en/billing/how-tos/set-up-payment/set-up-vs-subscription)
- [Roles for Visual Studio subscriptions with GitHub Enterprise](https://docs.github.com/en/billing/reference/roles-for-visual-studio)

---

*Last updated: October 2026*
