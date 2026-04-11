# 🔗 Visual Studio Subscription to GitHub Enterprise Linking Runbook

> **Complete guide to linking Visual Studio subscriptions with GitHub Enterprise Cloud for entitled seat management**

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
GitHub.com → Your Enterprise → Settings → Billing
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

---

## 📚 Resources

- [Access GitHub Enterprise with your Visual Studio subscription](https://learn.microsoft.com/en-us/visualstudio/subscriptions/access-github)
- [Setting up Visual Studio subscriptions with GitHub Enterprise](https://docs.github.com/en/billing/managing-licenses-for-visual-studio-subscriptions-with-github-enterprise/setting-up-visual-studio-subscriptions-with-github-enterprise)

---

*Last updated: April 2026*
