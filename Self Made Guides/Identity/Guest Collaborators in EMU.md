# 👥 GitHub EMU Guest Collaborators Runbook

> **Complete guide to provisioning and managing guest collaborators for contractors, vendors, and partners in GitHub Enterprise Managed Users (EMU)**

| 🧭 **At a Glance** | |
|---|---|
| **Goal** | Give contractors and vendors limited access in an EMU enterprise |
| **Use this when** | Onboarding external people who shouldn't see all internal code |
| **People you need** | IdP application admin; organization owners; repository admins |
| **Where you click** | Your IdP (Entra ID or Okta) and GitHub |
| **End result** | Guests provisioned by the IdP with access only where you grant it |
| **New to a term?** | See the [Glossary](../Glossary.md) for plain-English definitions |

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [📋 Overview](#-overview)
- [🔑 What Are Guest Collaborators?](#-what-are-guest-collaborators)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [1️⃣ Enable Guest Collaborator Role in Your IdP](#1️⃣-enable-guest-collaborator-role-in-your-idp)
- [2️⃣ Provision Users via SCIM with Guest Collaborator Role](#2️⃣-provision-users-via-scim-with-guest-collaborator-role)
- [3️⃣ Grant Access to Guest Collaborators](#3️⃣-grant-access-to-guest-collaborators)
- [4️⃣ Access Limitations for Guest Collaborators](#4️⃣-access-limitations-for-guest-collaborators)
- [5️⃣ Best Practices](#5️⃣-best-practices)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Make the role available (once):** Entra — app registration **Manifest** → add the Guest Collaborator app role → **Save**. Okta — app → **Provisioning** → **Go to Profile Editor** → **Roles** → add `guest_collaborator` → **Save**
- **Assign (Entra ID):** `Entra ID → Enterprise applications → [GitHub EMU app] → Users and groups → Add user/group` → user/group → **Select a role: Guest Collaborator** → **Assign**
- **Assign (Okta):** `Applications → [GitHub EMU app] → Assignments → Assign` → user/group → **Roles: Guest Collaborator** → **Save and Go Back**
- **Org access:** `Org → People → Invite member` → managed username → role → **Send invitation**
- **Single-repo access:** `Repo → Settings → Collaborators and teams → Add people` → managed username → role → add

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

| Attribute | Guest collaborator | Regular member |
|-----------|-------------------|----------------|
| **Account type** | Managed (provisioned via IdP) | Managed (provisioned via IdP) |
| **Internal repositories** | Only in organizations where they're a **member** | **All** internal repositories in the enterprise once they're a member of any organization |
| **Private repositories** | Per the org's base permissions and team or repo grants | Same |
| **Typical access** | Membership in one org, or repository collaborator on specific repos | Membership in one or more orgs |
| **Intended for** | Contractors, vendors, partner agencies | Employees |

> 💡 **Tip:** Use guest collaborators for any external party who needs access to specific repositories but should not have broad visibility into your enterprise's internal codebase.

---

## ✅ Prerequisites

| Requirement | Who / Role needed | ✓ |
|-------------|-------------------|---|
| An **Enterprise Managed Users** enterprise (the role doesn't exist for personal-account enterprises) | — | ☐ |
| IdP SSO and SCIM provisioning working (Entra ID, Okta, PingFederate, or the SCIM REST API) | IdP administrator | ☐ |
| Permission to edit the IdP app (Entra: app registration manifest; Okta: profile editor) | **Application Administrator** / **Cloud Application Administrator** (Entra) or Okta app admin | ☐ |
| Grant access in GitHub | **Organization owner** (org membership) or **repository admin** (repo collaborator) | ☐ |

> ⚠️ **Important:** guest collaborators are **only** available with Enterprise Managed Users.

---

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **Microsoft Entra application admin, for Entra-backed EMU** | Assigns the user or group to the GitHub EMU app with the Guest Collaborator role. | Microsoft Entra admin center or Azure portal → Entra ID → Enterprise applications → GitHub Enterprise Managed User → Users and groups → Add user/group → select user or group → Select a role → Guest Collaborator → Assign. Handoff: assigned user/group and role. |
| **Okta application admin, for Okta-backed EMU** | Assigns the person or group to the GitHub EMU app with the `guest_collaborator` role attribute. | Okta Admin Console → Applications → Applications → [GitHub Enterprise Managed User app] → Assignments → Assign → Assign to People or Assign to Groups → select user/group → set Role to `guest_collaborator` → Save and Go Back. Handoff: assignment and role value. |
| **GitHub organization or repository admin** | Grants explicit access after SCIM provisions the guest collaborator account. | For organization access: GitHub → profile picture → Organizations → [organization] → People → Invite member → search the managed username → select role → Send invitation. Repository-only access: GitHub → [owner/repository] → Settings → Collaborators and teams → Add people or Add teams → select collaborator/team → choose role → Add. Handoff: org or repository access confirmation. |

---

## 1️⃣ Enable Guest Collaborator Role in Your IdP

### A) Entra ID — make the role available (once)

**👤 Role:** **Application Administrator** or **Cloud Application Administrator** · **📍 Portal:** Azure portal / Microsoft Entra admin center

1. Go to **Enterprise applications** → **All applications** → your GitHub Enterprise Managed User app → **Users and groups**.
2. If **Add user/group** already offers a **Guest Collaborator** (or older **Restricted User**) role, skip to Step B.
3. Otherwise open **App registrations** → **All applications** → your EMU app → **Manifest**.
4. Search for the `id` `1ebc4a02-e56c-43a6-92a5-02ee09b90824`:
   - If it exists, make sure its `displayName` and `description` are `Guest Collaborator`.
   - If it doesn't, add this block inside `appRoles`:

```json
{
  "allowedMemberTypes": ["User"],
  "description": "Guest Collaborator",
  "displayName": "Guest Collaborator",
  "id": "1ebc4a02-e56c-43a6-92a5-02ee09b90824",
  "isEnabled": true,
  "lang": null,
  "origin": "Application",
  "value": null
}
```

5. Click **Save**.

> ⚠️ The `id` value must be exactly this GUID, or the update fails.

### B) Entra ID — assign people as guests

**Navigate:** Entra ID → **Enterprise applications** → *[GitHub EMU app]* → **Users and groups**

1. Click **Add user/group**.
2. Under **Users and groups**, select the user or group, then click **Select**.
3. Under **Select a role**, choose **Guest Collaborator**, then click **Select**.
4. Click **Assign**.

> ✅ **Result:** the next SCIM cycle provisions the account with the guest collaborator role.

### C) Okta — make the role available (once)

**👤 Role:** Okta application administrator · **📍 Portal:** Okta Admin Console

1. Open your Enterprise Managed Users application and click **Provisioning**.
2. Click **Go to Profile Editor**.
3. At the bottom, find **Roles** and click the edit icon.
4. Add a role with **Display name** `Guest Collaborator` and **Value** `guest_collaborator`.
5. Click **Save**.

### D) Okta — assign people as guests

1. In the EMU app, click **Assignments** → **Assign** → **Assign to People** (or **Assign to Groups**).
2. Next to the user or group, click **Assign**.
3. Under **Roles**, choose **Guest Collaborator**.
4. Click **Save and Go Back**, then **Done**.

> 💡 **Other IdPs:** use the **Roles** attribute in your EMU application, or the `roles` attribute when provisioning through GitHub's SCIM REST API.

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

> ⚠️ **Important:** a newly provisioned guest collaborator can sign in but has no organization or repository access until you grant it (next section).

---

## 3️⃣ Grant Access to Guest Collaborators

*Guest collaborators must be explicitly added to organizations or repositories.*

### A) Add to a specific organization

**👤 Role:** **Organization owner** · **📍 Portal:** GitHub

1. Organization → **People** → **Invite member**.
2. Type the guest's managed username and click **Invite**.
3. Under **Role in the organization**, choose **Member**.
4. *(Optional)* Choose a license (if prompted) and teams.
5. Click **Send invitation**.

> 📌 As an organization member, a guest can access that organization's **internal** repositories — and private ones according to the org's **base permissions** and team grants. They still can't see internal repositories in other organizations. You can also add guests through IdP groups linked to GitHub teams.

### B) Add directly as a repository collaborator

**👤 Role:** **Repository administrator** · **📍 Portal:** GitHub

1. Repository → **Settings** → **Collaborators and teams**.
2. Click **Add people** and search for the guest's managed username.
3. Choose a role (**Read**, **Triage**, **Write**, **Maintain**, or **Admin**) and confirm.

> ✅ **Result:** access to that repository only — no other internal or private repositories in the organization.

| Role | Access |
|------|--------|
| **Read** | View and clone code, issues, pull requests |
| **Triage** | Read + manage issues and pull requests (no code writes) |
| **Write** | Push code, manage issues and pull requests |
| **Maintain** | Write + manage some repository settings (no destructive actions) |
| **Admin** | Full repository administration |

---

## 4️⃣ Access Limitations for Guest Collaborators

| Capability | Regular member | Guest collaborator |
|------------|---------------|-------------------|
| **Internal repositories** | All internal repos in the enterprise (once a member of any org) | Only internal repos in orgs where they're a member |
| **Private repositories** | Per base permissions and grants | Per base permissions and grants (same rules) |
| **Repository-only access** | Possible as a repository collaborator | Possible as a repository collaborator |
| **Creating or forking repositories** | Per organization and enterprise policies | Per the same policies, in orgs where they're a member |
| **Outside the enterprise** | View public repos only | View public repos only |

> 💡 For the tightest access, add a guest as a **repository collaborator** instead of an organization member.

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

---

## 🧯 Known Errors & Resolutions

<details>
<summary><em>Show known errors table</em></summary>

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

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>

### Q: A guest collaborator cannot see internal-visibility repositories in the organization. Is this a bug?
**A:** Check how they were added. As a **repository collaborator**, a guest only sees that repository. As an **organization member**, a guest can access that organization's internal repositories — but not internal repositories in **other** organizations. If they should see more, add them as a member of the right organization (or to the specific repositories).

---

### Q: SCIM provisioning is not creating the guest collaborator account. What should I check?
**A:** Make sure the role exists in the IdP app (Entra: the `1ebc4a02-…` app role in the manifest; Okta: a `guest_collaborator` value under **Roles** in the profile editor), that the user or group is assigned with **Guest Collaborator** selected, and that SCIM provisioning is running. Check the IdP's provisioning logs for errors such as username conflicts.

---

### Q: Can we convert a guest collaborator to a full enterprise member without deprovisioning them?
**A:** Yes. Change the user's role in the IdP assignment from **Guest Collaborator** to the regular user role and let SCIM sync. The same managed account becomes a regular enterprise member — including access to internal repositories across the enterprise once they're in an organization.

---

### Q: Does a guest collaborator consume a license seat in our enterprise?
**A:** Only once they have access. A guest collaborator who isn't an organization member or a repository collaborator doesn't consume a license. As soon as you add them to an organization or as a collaborator on a private or internal repository, they consume a GitHub Enterprise license like any member.

---

### Q: How do we offboard a guest collaborator when their contract ends?
**A:** Remove the user (or remove them from the assigned group) in the GitHub EMU application in your IdP. SCIM suspends the managed account and its access ends. Always offboard through the IdP — managed accounts can't be removed from the enterprise directly in GitHub.

---

### Q: Can guest collaborators create or fork repositories?
**A:** It depends on how they're added and on your policies. As a repository collaborator they can't create repositories in the organization. As an organization member, the organization's repository-creation and forking policies (and enterprise policies) apply to them like any member. To keep guests tightly scoped, add them as repository collaborators and restrict repository creation and forking by policy.

</details>

---

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

*Last updated: October 2026*
