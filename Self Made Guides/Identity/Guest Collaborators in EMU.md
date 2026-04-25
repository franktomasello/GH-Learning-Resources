# 👥 GitHub EMU Guest Collaborators Runbook

> **Complete guide to provisioning and managing guest collaborators for contractors, vendors, and partners in GitHub Enterprise Managed Users (EMU)**

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Assign role (Entra ID):** `Azure Portal → Entra ID → Enterprise Applications → GitHub EMU App → Users and groups → Add → Select "Guest Collaborator" role`
- **Assign role (Okta):** `Okta Admin → Applications → GitHub EMU App → Assignments → Set Role to guest_collaborator`
- **Add to org:** `Org → People → Invite member → Search managed username`
- **Add to repo:** `Repo → Settings → Collaborators and teams → Add people → Search managed username`

---

## ✅ Accuracy & Click-Path Notes

- Reviewed against current public GitHub and Microsoft documentation in April 2026 where public documentation is available. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
- When a path starts with `Enterprise`, begin at GitHub, click your profile photo, click `Your enterprises` or `Enterprise`, select the enterprise, then continue with the listed top tab or left-sidebar item.
- When a path starts with `Organization` or `Org`, begin at GitHub, click your profile photo, click `Your organizations`, select the organization, click `Settings`, then continue with the listed sidebar item.
- When a path starts with `Repository`, `Repo`, or a repository name, open the repository, click the `Settings` tab, then continue with the listed sidebar item.
- When a path starts with a vendor portal such as `Microsoft Entra admin center`, `Azure portal`, `Okta Admin Console`, `PingFederate`, `PingOne`, `OneLogin`, `AD FS Management`, `Visual Studio Admin Portal`, or `Azure DevOps`, sign in to that admin portal first, select the tenant, application, or project named in the step, then follow each listed blade, tab, button, and confirmation in order.
- If the expected button is missing, verify you are signed in with the role named in Prerequisites, the feature or license is enabled, and the object is owned by the selected enterprise, organization, or repository. Use page search only to locate the same page, not to skip required confirmation, test, save, or consent clicks.

---

## 📋 Overview

This runbook covers how to set up guest collaborator access in an EMU enterprise:

| Topic | Description |
|-------|-------------|
| **What are guest collaborators** | Managed accounts with limited access for external users |
| **IdP configuration** | Enable the Guest Collaborator role in your identity provider |
| **SCIM provisioning** | Provision users with the `guest_collaborator` role |
| **Access grants** | Add guests to specific orgs or repos |
| **Limitations** | What guests cannot do compared to regular members |

---

## 🔑 What Are Guest Collaborators?

Guest collaborators are **managed user accounts** provisioned through your IdP that have restricted access within your EMU enterprise:

| Attribute | Guest Collaborator | Regular Member |
|-----------|-------------------|----------------|
| **Account type** | Managed (provisioned via IdP) | Managed (provisioned via IdP) |
| **Internal repo access** | No broad access — must be explicitly granted | Can access all internal-visibility repos |
| **Org membership** | Must be explicitly added to specific orgs | Can be added to any org |
| **Repo access** | Only repos explicitly granted | All internal repos + explicitly granted private repos |
| **Intended for** | Contractors, vendors, partner agencies | Full-time employees, internal staff |

> 💡 **Tip:** Use guest collaborators for any external party who needs access to specific repositories but should not have broad visibility into your enterprise's internal codebase.

---

## ✅ Prerequisites

Before configuring guest collaborators, ensure you have:

| Requirement | Detail |
|-------------|--------|
| **EMU enterprise** | Guest collaborators are only available in Enterprise Managed Users |
| **IdP configured** | Entra ID or Okta with SAML/OIDC and SCIM already working |
| **Enterprise owner access** | You must be an enterprise owner to enable the feature |
| **SCIM provisioning active** | User provisioning via SCIM must be operational |

> ⚠️ **Important:** Guest collaborators are **not available** in standard GHEC enterprises — only in EMU enterprises.

---

## 1️⃣ Enable Guest Collaborator Role in Your IdP

### A) For Entra ID

**Navigation:**

```
Azure Portal → Entra ID → Enterprise Applications
  → GitHub Enterprise Managed User (your app)
    → Users and groups → Add user/group
```

**Steps:**

1. Open the **Azure Portal** and navigate to **Entra ID**
2. Go to **Enterprise Applications**
3. Select your **GitHub Enterprise Managed User** application
4. Click **Users and groups**
5. Click **Add user/group**
6. Select the user or group you want to provision as a guest collaborator
7. Under **Select a role**, choose **Guest Collaborator**
8. Click **Assign**

> ✅ **Result:** The user is assigned the Guest Collaborator role and will be provisioned with restricted access on next SCIM sync.

### B) For Okta

**Steps:**

1. Open the **Okta Admin Console**
2. Navigate to **Applications** → select your GitHub EMU application
3. Click **Assignments** → **Assign** → **Assign to People** (or Groups)
4. Select the user or group
5. In the assignment settings, set the **Role** attribute to `guest_collaborator`
6. Click **Save and Go Back**

> ✅ **Result:** The user is queued for provisioning with the guest collaborator role.

---

## 2️⃣ Provision Users via SCIM with Guest Collaborator Role

*Users assigned the Guest Collaborator role in your IdP are automatically provisioned via SCIM.*

**What happens during provisioning:**

| Step | Action |
|------|--------|
| 1 | IdP sends SCIM request to GitHub with `guest_collaborator` role |
| 2 | GitHub creates a managed user account with guest collaborator permissions |
| 3 | The user receives a managed account (e.g., `username_shortcode`) |
| 4 | The user can sign in but has **no access** until explicitly granted |

> ⚠️ **Important:** A newly provisioned guest collaborator has **no repository or organization access** by default. You must explicitly grant access (see next section).

---

## 3️⃣ Grant Access to Guest Collaborators

*Guest collaborators must be explicitly added to organizations or repositories.*

### A) Add to a Specific Organization

**Navigation:**

```
Organization → People → Invite member
  → Search for the guest collaborator's managed username
    → Select role (Member) → Send invitation
```

> 💡 **Note:** Even as an org member, a guest collaborator will **not** automatically see internal-visibility repositories. They only see repos they are explicitly granted access to.

### B) Add Directly as a Repository Collaborator

**Navigation:**

```
Repository → Settings → Collaborators and teams (sidebar)
  → Add people → Search for the guest collaborator's managed username
    → Select permission level (Read, Write, etc.) → Add
```

| Permission Level | Access |
|-----------------|--------|
| **Read** | View code, issues, pull requests |
| **Write** | Push code, manage issues and PRs |
| **Maintain** | Manage repo settings (no destructive actions) |
| **Admin** | Full repo administration |

---

## 4️⃣ Access Limitations for Guest Collaborators

> ⚠️ **Important:** Guest collaborators have significant access restrictions compared to regular members.

| Capability | Regular Member | Guest Collaborator |
|------------|---------------|-------------------|
| **View internal repos** | Yes (all internal repos in the enterprise) | No (must be explicitly granted) |
| **Browse org repos** | Yes (all repos visible to members) | Only explicitly granted repos |
| **Create repos** | Yes (per org policy) | No |
| **Join teams** | Yes | Only if explicitly added |
| **Enterprise-wide visibility** | Yes | No |
| **Fork internal repos** | Yes (per policy) | No |

---

## 5️⃣ Best Practices

| Practice | Rationale |
|----------|-----------|
| **Use for vendors and contractors** | Limit exposure of internal code to external parties |
| **Use for partner agencies** | Give partners access to shared repos without enterprise-wide visibility |
| **Grant minimum required access** | Add guests to specific repos, not entire orgs when possible |
| **Review guest access quarterly** | Ensure offboarded contractors have been deprovisioned in the IdP |
| **Deprovision via IdP** | Remove guest collaborator assignments in your IdP — SCIM handles account removal |
| **Use groups in your IdP** | Assign the Guest Collaborator role to IdP groups for easier lifecycle management |

> 💡 **Tip:** When a contractor engagement ends, remove them from the GitHub EMU app assignment in your IdP. SCIM deprovisioning will automatically suspend their managed account.

## 🧯 Known Errors & Resolutions

> This section lists the known product errors and admin-facing symptoms that commonly occur with this workflow. Exact message text can vary by product rollout, tenant policy, and provider, so use the log or settings page named in the resolution to confirm the root cause.

| Error or symptom | Likely cause | Resolution |
|------------------|--------------|------------|
| **Page, tab, or button is missing** | Wrong account context, missing admin role, unavailable plan/add-on, or feature rollout not enabled for the selected enterprise/org/repo. | Switch to the correct account and scope, confirm the prerequisite role, verify licensing or add-on activation, then refresh the page. If the control is still absent, use the direct settings URL from the relevant GitHub Docs page and confirm the feature is available for your plan. |
| **Changes appear saved but behavior does not change** | Policy inheritance, cached UI state, propagation delay, or an overlapping enterprise/org/repo policy. | Reopen the settings page, verify the effective policy at the lowest affected scope, wait for propagation where documented, and check for a stricter policy at an enterprise or organization level. |
| **403, forbidden, or resource not accessible** | The signed-in user or token can see the page but lacks the specific permission for the action. | Use an enterprise owner, organization owner, repository admin, or token with the exact scopes/permissions listed in the runbook. For SAML-protected orgs, authorize the token or SSH key for SSO before retrying. |
| **Managed user cannot sign in** | The user is not assigned to the IdP application, SCIM has not provisioned the account, or SAML/OIDC configuration is wrong. | Check IdP assignment, provisioning logs, userName/NameID mappings, and the GitHub authentication test before enforcing broadly. |
| **Managed user cannot interact with public repositories** | EMU accounts are restricted from contributing outside enterprise-owned resources. | Use the dual-presence model: managed account for enterprise work and a personal GitHub.com account for open source participation. |
| **Guest collaborator cannot see internal repositories** | Guest collaborators do not receive broad internal repository access by default. | Add the guest collaborator to the specific organization/team/repository that should grant access, or use a regular enterprise member role if broad internal access is intended. |
| **Duplicate or wrong managed username appears** | Shortcode, userName mapping, or multiple enterprise assignments created distinct managed accounts. | Verify the IdP app and SCIM mapping for the intended enterprise, then deprovision incorrect assignments through the IdP. |

---

## ❓ Common Questions & Troubleshooting

### Q: A guest collaborator cannot see internal-visibility repositories in the organization. Is this a bug?
**A:** No, this is by design. Guest collaborators do not have automatic access to internal-visibility repositories, even if they are members of an organization. They can only access repositories that have been explicitly granted to them. Add the guest collaborator directly to the specific repos they need via Repository > Settings > Collaborators and teams.

---

### Q: SCIM provisioning is not creating the guest collaborator account. What should I check?
**A:** Verify that the `guest_collaborator` role is correctly mapped in your IdP's Enterprise Application configuration. In Entra ID, check that the user or group is assigned with the "Guest Collaborator" role selected. In Okta, confirm the Role attribute is set to `guest_collaborator` in the assignment settings. Also verify that SCIM provisioning is active and syncing successfully -- check the IdP's provisioning logs for errors.

---

### Q: Can we convert a guest collaborator to a full enterprise member without deprovisioning them?
**A:** Yes, update the user's role assignment in your identity provider from `guest_collaborator` to the regular member role, then trigger a SCIM sync. The user's managed account will be updated to full member status, granting them access to internal-visibility repos and other member capabilities. You do not need to deprovision and re-create the account.

---

### Q: Does a guest collaborator consume a license seat in our enterprise?
**A:** Yes, guest collaborators do consume enterprise license seats just like regular members. Each provisioned guest collaborator counts as one seat toward your enterprise license total. Factor this into your capacity planning when onboarding contractors and vendors.

---

### Q: How do we offboard a guest collaborator when their contract ends?
**A:** Remove the user from the GitHub EMU application assignment in your IdP (Entra ID or Okta). SCIM deprovisioning will automatically suspend the managed account and remove all access. Do not try to remove the user directly in GitHub -- always deprovision through the IdP to ensure the account lifecycle is properly managed.

---

### Q: Can guest collaborators create repositories or fork existing repos?
**A:** No. Guest collaborators cannot create repositories or fork repos within the enterprise. They are limited to read/write access on repositories that have been explicitly granted to them. If a contractor needs to create a repo, an enterprise member or admin must create it and then grant the guest collaborator access.

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| EMU + Entra ID (OIDC) + Azure Billing + Copilot | `Setup/EMU + Entra ID (OIDC) + Azure Billing + Copilot.md` |
| EMU + Entra ID (SAML) + Azure Billing + Copilot | `Setup/EMU + Entra ID (SAML) + Azure Billing + Copilot.md` |
| EMU Benefits and Advantages | `Identity/EMU Benefits and Advantages.md` |
| EMU Dual Presence (Enterprise + Open Source) | `Identity/EMU Dual Presence (Enterprise + Open Source).md` |
| Enterprise Environment Scaffolding Checklist | `Setup/Enterprise Environment Scaffolding Checklist.md` |

---

## 📝 Resources

| Resource | Link |
|----------|------|
| **Enabling guest collaborators** | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/admin/managing-accounts-and-repositories/managing-users-in-your-enterprise/enabling-guest-collaborators) |
| **About Enterprise Managed Users** | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/admin/concepts/identity-and-access-management/enterprise-managed-users) |

---

*Last updated: April 2026*
