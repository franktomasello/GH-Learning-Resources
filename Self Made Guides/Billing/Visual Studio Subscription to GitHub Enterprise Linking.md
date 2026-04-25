# 🔗 Visual Studio Subscription Linking for GitHub Enterprise Runbook

> **Complete guide to linking Visual Studio subscriptions with GitHub Enterprise Cloud for entitled seat management**

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Verify in VS Admin Portal:** `https://manage.visualstudio.com → Subscribers → Search for user`
- **Verify on GitHub side:** `Enterprise → Billing and licensing → Licensing → View license usage`
- **User self-link (GitHub):** `GitHub.com → Profile → Settings → Billing and plans → Plans → Link Visual Studio subscription`
- **User self-link (VS portal):** `https://my.visualstudio.com → Benefits → Tools → GitHub Enterprise → Activate`

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
| Visual Studio Enterprise subscription (not Professional) | ☐ |
| Subscription assigned to user's corporate email (not personal email) | ☐ |
| Subscription associated with the correct Entra ID tenant | ☐ |
| GitHub Enterprise Cloud account linked to the enterprise | ☐ |
| User has a GitHub account with a verified email matching the VS subscription | ☐ |

---

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **Visual Studio subscriptions administrator** | Confirms that the user has the eligible Visual Studio Enterprise entitlement assigned to the correct corporate identity. | Visual Studio Admin Portal (`https://manage.visualstudio.com`) → Subscribers → search user by name or email → open subscriber → verify Subscription level is Visual Studio Enterprise, Status is active, email is the corporate email, and tenant is correct. Handoff: subscriber email, subscription level, and tenant confirmation. |
| **GitHub enterprise owner or billing admin** | Verifies that GitHub sees the Visual Studio entitlement on the enterprise license side. | GitHub → profile photo → Your enterprises → [enterprise] → Billing and licensing → Licensing → View license usage → search user → confirm Visual Studio column or license type shows the Visual Studio subscription entitlement. Handoff: user row or export showing entitlement status. |
| **End user** | Links their own GitHub account to the Visual Studio subscription. | GitHub → profile photo → Settings → Billing and plans → Plans → Link Visual Studio subscription → sign in with the corporate Microsoft account → approve prompts → confirm the linked subscription. Alternative: Visual Studio Benefits Portal → GitHub Enterprise benefit → Activate → sign in to GitHub. Handoff: GitHub profile shows the linked subscription or GitHub access is restored. |

---

## 📋 Overview

Visual Studio Enterprise subscriptions include a GitHub Enterprise Cloud seat at no additional cost. This runbook walks through verifying eligibility, confirming the link on both the VS Admin Portal and GitHub Enterprise sides, and troubleshooting common issues.

| VS Subscription Tier | GitHub Enterprise Included? | Notes |
|---|---|---|
| **Visual Studio Enterprise** | Yes | GHEC seat included |
| **Visual Studio Professional** | No | Not included |
| **GitHub Copilot** | No | Not included in any VS subscription tier |

> ⚠️ **Important:** Only Visual Studio Enterprise subscriptions include GitHub Enterprise Cloud. Professional and other tiers do not.

---

## 1️⃣ Step 1 — Verify in the Visual Studio Admin Portal

*Confirm the user has an eligible subscription and it is properly configured*

### Navigate to the Admin Portal

**Navigation:**

```
https://manage.visualstudio.com → Subscribers
  → Search for user by name or email
```

### Verify the Following

| Check | What to Confirm |
|-------|-----------------|
| **Subscription level** | Must be Visual Studio Enterprise (not Professional) |
| **Assigned email** | Must be the user's work email (corporate identity) |
| **Entra tenant** | Subscription must be associated with the correct Microsoft Entra (Azure AD) tenant |
| **Status** | Subscription must be active (not expired or pending) |

> 💡 **Tip:** If the subscription is assigned to a personal email instead of a work email, the link to GitHub Enterprise will not work. The VS admin must reassign it to the correct corporate identity.

---

## 2️⃣ Step 2 — Verify on the GitHub Enterprise Side

*Confirm that the Visual Studio entitlement is visible in GitHub Enterprise billing*

### Navigate to Enterprise Licensing

**Navigation:**

```
GitHub.com → Your Enterprise → Billing and licensing
  → Licensing → View license usage
```

### What to Look For

| Column | Meaning |
|--------|---------|
| **Visual Studio** | Shows a checkmark if the seat is linked via a VS Enterprise subscription |
| **License type** | Should show "Visual Studio subscription" for entitled users |

> 💡 **Tip:** If a user does not appear in the Visual Studio column, they have either not completed the self-service link step (Step 3) or there is an email/tenant mismatch.

---

## 3️⃣ Step 3 — User Self-Service Linking

*The individual user must complete this step themselves to activate their entitlement*

### Option A: Link from GitHub Settings

**Navigation:**

```
GitHub.com → Profile Picture → Settings
  → Billing and plans → Plans
    → Link Visual Studio subscription
```

### Option B: Link from the Visual Studio Benefits Portal

**Navigation:**

```
https://my.visualstudio.com → Benefits tab
  → Tools section → GitHub Enterprise benefit
    → Activate / Link
```

> ⚠️ **Important:** The user must be signed into GitHub with the account they want linked. If they use the wrong GitHub account, the entitlement will attach to the wrong identity.

> ✅ **Result:** Once linked, the user's GitHub Enterprise seat is covered by their Visual Studio subscription and will not incur a separate purchased-seat charge.

---

## 4️⃣ Common Issues & Troubleshooting

| Issue | Cause | Resolution |
|-------|-------|------------|
| **User not appearing in VS column** | User never completed the self-service link step | Have the user follow Step 3 above |
| **Email mismatch** | VS subscription assigned to a different email than the user's GitHub verified email | Align emails — either update the VS assignment or add the correct email to the GitHub account |
| **Wrong Entra tenant** | VS subscription associated with a tenant that does not match the GitHub Enterprise SAML/EMU IdP | VS admin must reassign the subscription under the correct tenant |
| **Double-billing** | User has both a VS-linked entitlement and a separately purchased GHEC seat | Remove the purchased seat — GitHub will continue to use the VS entitlement |
| **Access lost after subscription lapse** | VS Enterprise subscription expired or was reassigned | User loses their VS-entitled GHEC seat; must be given a purchased seat or have the subscription reinstated |

---

## 5️⃣ Important Caveats

- **VS-entitled seats are tied to the individual.** If the Visual Studio subscription lapses, is reassigned, or expires, the user loses their GitHub Enterprise Cloud access.
- **You can mix VS-linked and purchased seats.** An enterprise can have some users on VS entitlements and others on directly purchased seats simultaneously.
- **GitHub deduplicates billing.** A user who has both a VS entitlement and a purchased seat will only consume one seat — GitHub will not charge twice for the same user.
- **Linking is a one-time action.** Once a user links their VS subscription to their GitHub account, it persists unless the subscription is removed or the user unlinks it.

## 🧯 Known Errors & Resolutions

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
| **Link Visual Studio subscription option is missing** | The user does not have an eligible Visual Studio Enterprise subscription with the GitHub Enterprise benefit or is signed into the wrong GitHub account. | Verify assignment in the Visual Studio Admin Portal, confirm the subscription is active, and have the user retry from the correct GitHub account. |
| **Subscription cannot be matched to the GitHub account** | The Visual Studio subscription email does not match a verified email on the GitHub account. | Either update the Visual Studio assignment email or add and verify that email on the GitHub account, then repeat the linking flow. |
| **User appears unlicensed in GitHub after linking** | The user has not joined an organization in the enterprise, the link has not synchronized, or the wrong account was linked. | Confirm the user accepted an org invitation, check enterprise license usage after synchronization, and unlink/relink if the wrong GitHub account was used. |
| **Benefit disappeared after working previously** | The Visual Studio subscription expired, was reassigned, or the bundled benefit changed. | Recheck the subscriber record in the Visual Studio Admin Portal and restore the eligible assignment or replace the seat with a paid GitHub Enterprise license. |

---

## ❓ Common Questions & Troubleshooting

### Q: A user cannot find the "Link Visual Studio subscription" option in their GitHub settings. Where is it?
**A:** The option is at GitHub.com > Profile Picture > Settings > Billing and plans > Plans > "Link Visual Studio subscription." The user must have a Visual Studio Enterprise subscription (not Professional or other tiers) -- only VS Enterprise includes the GitHub Enterprise benefit. Also verify the VS subscription is active and assigned in the Visual Studio Admin Portal.

---

### Q: A user appears to be double-billed -- they have both a VS-linked seat and a purchased seat. How do we fix this?
**A:** Navigate to Enterprise > Billing and licensing > Licensing and check the user's license type. If they show both a VS entitlement and a purchased seat, remove the purchased seat. GitHub deduplicates billing, but having both entries can cause confusion. The VS entitlement will continue to provide the user's GitHub Enterprise access at no additional seat cost.

---

### Q: A user's Visual Studio subscription lapsed. What happens to their GitHub Enterprise access?
**A:** The user automatically loses their VS-entitled GitHub Enterprise Cloud seat. They will be unable to access enterprise resources until either: (1) the VS subscription is renewed and they re-link it, or (2) the enterprise admin provisions them a separate purchased seat. There is no grace period -- access is lost when the subscription expires.

---

### Q: The VS subscription email does not match the user's GitHub account email. How do we resolve this?
**A:** The email on the Visual Studio subscription must match a verified email on the user's GitHub account. Either: (1) have the VS admin update the subscription assignment to use the email that matches the GitHub account, or (2) have the user add their VS subscription email as a verified email on their GitHub account (Settings > Emails > Add email address). Both emails must match for the link to work.

---

### Q: Can a user link a Visual Studio Professional subscription to get a free GitHub Enterprise seat?
**A:** No. Only Visual Studio Enterprise subscriptions include the GitHub Enterprise Cloud benefit. Visual Studio Professional, Test Professional, and MSDN Platforms subscriptions do not include this entitlement. Users on these plans need a separately purchased GitHub Enterprise seat.

---

### Q: A user linked their VS subscription to the wrong GitHub account. How do they fix it?
**A:** The user must unlink the VS subscription from the incorrect GitHub account, then re-link it to the correct account. Unlink by going to GitHub.com > Settings > Billing and plans and removing the VS subscription link. Then sign in with the correct GitHub account and complete the linking process from Step 3 of this guide.

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Cost Centers & Department Billing | `Billing/Cost Centers & Department Billing.md` |
| Copilot Seat Assignment & Enablement | `Copilot/Seat Assignment & Enablement.md` |
| Enterprise Trial (GHEC, EMU, DRUS) | `Setup/Enterprise Trial (GHEC, EMU, DRUS).md` |

---

## 📚 Resources

- [Access GitHub Enterprise with your Visual Studio subscription](https://learn.microsoft.com/en-us/visualstudio/subscriptions/access-github)
- [Setting up Visual Studio subscriptions with GitHub Enterprise](https://docs.github.com/en/billing/managing-licenses-for-visual-studio-subscriptions-with-github-enterprise/setting-up-visual-studio-subscriptions-with-github-enterprise)

---

*Last updated: April 2026*
