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

## ❓ Common Questions & Troubleshooting

### Q: A user cannot find the "Link Visual Studio subscription" option in their GitHub settings. Where is it?
**A:** The option is at GitHub.com > Profile Picture > Settings > Billing and plans > Plans > "Link Visual Studio subscription." The user must have a Visual Studio Enterprise subscription (not Professional or other tiers) -- only VS Enterprise includes the GitHub Enterprise benefit. Also verify the VS subscription is active and assigned in the Visual Studio Admin Portal.

---

### Q: A user appears to be double-billed -- they have both a VS-linked seat and a purchased seat. How do we fix this?
**A:** Navigate to Enterprise > Settings > Billing > Licensing and check the user's license type. If they show both a VS entitlement and a purchased seat, remove the purchased seat. GitHub deduplicates billing, but having both entries can cause confusion. The VS entitlement will continue to provide the user's GitHub Enterprise access at no additional seat cost.

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

## 📚 Resources

- [Access GitHub Enterprise with your Visual Studio subscription](https://learn.microsoft.com/en-us/visualstudio/subscriptions/access-github)
- [Setting up Visual Studio subscriptions with GitHub Enterprise](https://docs.github.com/en/billing/managing-licenses-for-visual-studio-subscriptions-with-github-enterprise/setting-up-visual-studio-subscriptions-with-github-enterprise)

---

*Last updated: April 2026*
