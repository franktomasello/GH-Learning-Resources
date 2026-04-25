# 🔄 GitHub Repository Transfer & Organization Rename Runbook

> **Complete guide to transferring repositories between owners and renaming organizations, including what carries over, what breaks, and post-change checklists**

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [📋 Overview](#-overview)
- [1️⃣ Transfer a Repository](#1-transfer-a-repository)
- [2️⃣ What Transfers Automatically](#2-what-transfers-automatically)
- [3️⃣ Post-Transfer Checklist](#3-post-transfer-checklist)
- [4️⃣ Rename an Organization](#4-rename-an-organization)
- [5️⃣ Automatic Redirects](#5-automatic-redirects)
- [6️⃣ What to Update Manually](#6-what-to-update-manually)
- [7️⃣ Plan as a Coordinated Event](#7-plan-as-a-coordinated-event)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📚 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Transfer repo:** `Repo → Settings → General → Danger Zone → Transfer repository`
- **Rename org:** `Org → Settings → General → Change organization's name`
- **Update git remote (post-rename):** `git remote set-url origin https://github.com/NEW-ORG-NAME/repo.git`

---

## ✅ Accuracy & Click-Path Notes

<details>
<summary><em>Show click-path conventions</em></summary>


- Reviewed against current public GitHub and Microsoft documentation in April 2026 where public documentation is available. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
- When a path starts with `Enterprise`, begin at GitHub, click your profile photo, click `Your enterprises` or `Enterprise`, select the enterprise, then continue with the listed top tab or left-sidebar item.
- When a path starts with `Organization` or `Org`, begin at GitHub, click your profile photo, click `Your organizations`, select the organization, click `Settings`, then continue with the listed sidebar item.
- When a path starts with `Repository`, `Repo`, or a repository name, open the repository, click the `Settings` tab, then continue with the listed sidebar item.
- When a path starts with a vendor portal such as `Microsoft Entra admin center`, `Azure portal`, `Okta Admin Console`, `PingFederate`, `PingOne`, `OneLogin`, `AD FS Management`, `Visual Studio Admin Portal`, or `Azure DevOps`, sign in to that admin portal first, select the tenant, application, or project named in the step, then follow each listed blade, tab, button, and confirmation in order.
- If the expected button is missing, verify you are signed in with the role named in Prerequisites, the feature or license is enabled, and the object is owned by the selected enterprise, organization, or repository. Use page search only to locate the same page, not to skip required confirmation, test, save, or consent clicks.

</details>

---

## ✅ Prerequisites

| Requirement | Status |
|-------------|--------|
| Repo admin role on source repo (for transfers) | ☐ |
| Org owner role on destination org (for transfers) | ☐ |
| Org owner role (for renames) | ☐ |
| Communication plan for affected teams | ☐ |
| Inventory of integrations, CI/CD, SSO, and external references | ☐ |

---

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **GitHub organization owner or enterprise owner** | Renames or transfers only after identity and integration owners are ready to update downstream references. | For organization rename: GitHub → profile photo → Your organizations → [organization] → Settings → General → Organization name → Rename. Repository transfer: GitHub → [owner/repository] → Settings → General → Danger Zone → Transfer ownership → enter target owner/repository name → confirm transfer. Handoff: old URL, new URL, redirect status, and cutover time. |
| **Microsoft Entra application admin, if Entra SAML or SCIM references the old org URL** | Updates SAML and SCIM URLs after the org rename. | Microsoft Entra admin center → Entra ID → Enterprise apps → [GitHub app] → Single sign-on → SAML → Basic SAML Configuration → Edit → update Identifier, Reply URL, and Sign on URL → Save. Then Provisioning → Admin Credentials → update Tenant URL if the org slug changed → Test Connection → Save. Handoff: successful SAML test and SCIM test. |
| **Okta or PingFederate admin, if that IdP references the old org URL** | Updates SAML and provisioning URLs after the org rename. | Okta: Okta Admin Console → Applications → Applications → [GitHub app] → Sign On → Edit → update organization or SAML URL values → Save, then Provisioning → Integration → update API or base URL → Test API Credentials → Save. PingFederate: Administrative Console → Applications → SP Connections → [GitHub connection] → Browser SSO and Outbound Provisioning → update Entity ID, ACS, and SCIM Base URL → Save → activate. Handoff: successful SSO and provisioning test. |

---

## 📋 Overview

This runbook covers two high-impact administrative operations that require careful planning and coordination.

| Operation | Risk Level | Key Concern |
|-----------|-----------|-------------|
| **Repository transfer** | Medium | Team access, integrations, and policy inheritance |
| **Organization rename** | High | URL changes propagate across all repos, CI/CD, SSO, and integrations |

> ⚠️ **Warning:** Both operations should be planned as coordinated events with advance notice to affected teams.

---

# Part 1: Repository Transfer

---

## 1️⃣ Transfer a Repository

**Navigation:**

```
Repository → Settings → General (sidebar)
  → Danger Zone → Transfer repository
    → Enter destination owner → Type repository name to confirm
      → Confirm transfer
```

> ✅ **Result:** The repository is moved to the new owner (user or organization). A redirect from the old URL is created automatically.

> ⚠️ **Important:** You must be an admin on the repository to initiate a transfer. If transferring to an organization, you must also be an owner of the destination organization.

---

## 2️⃣ What Transfers Automatically

The following are preserved during a repository transfer:

| Item | Transfers? | Notes |
|------|-----------|-------|
| **Git history** | Yes | All commits, branches, and tags |
| **Issues** | Yes | Including labels, milestones, and assignees |
| **Pull requests** | Yes | Including review comments and status |
| **Wiki** | Yes | Full wiki content |
| **Stars** | Yes | Star count is preserved |
| **Watchers** | Yes | Notification subscriptions carry over |
| **Webhooks** | Yes | Existing webhook configurations |
| **Repository secrets** | Yes | Actions secrets transfer with the repo |
| **Deploy keys** | Yes | Existing deploy keys remain active |

> 💡 **Tip:** GitHub automatically sets up URL redirects from the old location to the new one. However, these redirects are not permanent -- they will break if a new repository is created at the old path.

---

## 3️⃣ Post-Transfer Checklist

These items do **NOT** transfer automatically and must be reconfigured:

| Item | Action Required |
|------|----------------|
| **Team access** | Grant appropriate teams in the destination org access to the repo |
| **Org-level policies and rulesets** | Verify the destination org's branch protection rules, rulesets, and policies apply correctly |
| **Integrations and GitHub Apps** | Reinstall or reconfigure any org-level GitHub Apps or third-party integrations |
| **CODEOWNERS** | Update the `CODEOWNERS` file if team names differ between source and destination orgs |
| **CI/CD references** | Update any hardcoded references to the old `owner/repo` path in pipelines |
| **Package references** | Update any package registry references that include the old owner |

> ⚠️ **Warning:** Org-level security configurations (secret scanning, code scanning settings) from the **source** org do not follow the repo. The destination org's configurations will apply instead.

---

# Part 2: Organization Rename

---

## 4️⃣ Rename an Organization

**Navigation:**

```
Organization → Settings → General (sidebar)
  → Change organization's name → Enter new name
    → Confirm rename
```

> ✅ **Result:** The organization is renamed. GitHub creates automatic redirects from the old organization URL to the new one.

> ⚠️ **Warning:** This is a high-impact change. All repository URLs under the organization change immediately (e.g., `github.com/old-name/repo` becomes `github.com/new-name/repo`).

---

## 5️⃣ Automatic Redirects

GitHub sets up redirects from old URLs to the new organization name. However:

| Aspect | Detail |
|--------|--------|
| **Redirect scope** | Web URLs, git clone URLs, API calls |
| **Duration** | Temporary -- redirects are not guaranteed to persist indefinitely |
| **Breaks if** | Someone creates a new organization with the old name |

> ⚠️ **Important:** Do not rely on redirects long-term. Update all references to use the new organization name as soon as possible.

---

## 6️⃣ What to Update Manually

The following must be updated after renaming the organization:

| Item | Action Required |
|------|----------------|
| **Git remotes** | All developers must update their local git remotes to the new URL |
| **Webhooks** | Update any webhooks that reference the old org name in their payload URLs or configurations |
| **GitHub Apps** | Reconfigure any GitHub Apps that reference the old organization name |
| **CI/CD configurations** | Update all pipeline configs (Actions workflows, Jenkins, CircleCI, etc.) that reference the old org name |
| **SCIM / SSO configuration** | Update your IdP (Okta, Azure AD, etc.) with the new organization name/URL |
| **Documentation and wikis** | Update internal docs, READMEs, and runbooks that reference the old org name |
| **Package registries** | Update references in package manifests (npm, Maven, NuGet, etc.) |
| **Dependabot and branch policies** | Verify these continue to work under the new name |

### Git Remote Update Command

Developers can update their local remotes with:

```
git remote set-url origin https://github.com/NEW-ORG-NAME/repo-name.git
```

---

## 7️⃣ Plan as a Coordinated Event

An organization rename affects every team and every repository. Treat it as a planned maintenance event.

| Phase | Actions |
|-------|---------|
| **1. Announce** | Notify all teams and stakeholders at least 1-2 weeks in advance |
| **2. Inventory** | Catalog all integrations, CI/CD pipelines, SSO configs, and external references |
| **3. Schedule** | Choose a low-activity window (weekend, end of sprint) |
| **4. Execute** | Perform the rename |
| **5. Update** | Immediately update SCIM/SSO, CI/CD, webhooks, and GitHub Apps |
| **6. Communicate** | Send post-rename instructions to developers (git remote update steps) |
| **7. Verify** | Confirm all critical pipelines, integrations, and SSO are functional |

> 💡 **Tip:** Create a shared checklist from the inventory in Phase 2 and assign owners to each item. This ensures nothing is missed during the rename.

## 🧯 Known Errors & Resolutions

<details>
<summary><em>Show known errors table</em></summary>


> This section lists the known product errors and admin-facing symptoms that commonly occur with this workflow. Exact message text can vary by product rollout, tenant policy, and provider, so use the log or settings page named in the resolution to confirm the root cause.

| Error or symptom | Likely cause | Resolution |
|------------------|--------------|------------|
| **Page, tab, or button is missing** | Wrong account context, missing admin role, unavailable plan/add-on, or feature rollout not enabled for the selected enterprise/org/repo. | Switch to the correct account and scope, confirm the prerequisite role, verify licensing or add-on activation, then refresh the page. If the control is still absent, use the direct settings URL from the relevant GitHub Docs page and confirm the feature is available for your plan. |
| **Changes appear saved but behavior does not change** | Policy inheritance, cached UI state, propagation delay, or an overlapping enterprise/org/repo policy. | Reopen the settings page, verify the effective policy at the lowest affected scope, wait for propagation where documented, and check for a stricter policy at an enterprise or organization level. |
| **403, forbidden, or resource not accessible** | The signed-in user or token can see the page but lacks the specific permission for the action. | Use an enterprise owner, organization owner, repository admin, or token with the exact scopes/permissions listed in the runbook. For SAML-protected orgs, authorize the token or SSH key for SSO before retrying. |
| **Audit log search returns no events** | The date range, action qualifier, actor, or retention window excludes the event. | Widen the query, search by known action names, and use exported/streamed logs for events older than the UI retention window. |
| **Audit log stream is configured but SIEM receives no events** | Destination credentials, network allow lists, event hub/topic configuration, or stream status is wrong. | Check stream health in GitHub, rotate destination credentials if needed, allow GitHub source IPs, and pause/resume only within documented retention limits. |
| **Ruleset blocks a push or merge unexpectedly** | A branch/tag/push ruleset or legacy branch protection rule targets the ref. | Open the repository rules view for the affected branch/tag, identify the active rule, and either comply with the rule or request a bypass from the owner. |
| **Repository transfer or org rename leaves broken references** | Profile URLs, marketplace/action namespaces, webhooks, secrets, environments, and external integrations may not redirect or transfer. | Inventory dependent systems before the change, update remote URLs and integration settings after the change, and validate webhooks, Actions, Apps, and security configurations. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>


### Q: We transferred a repository but the old URL is not redirecting. What happened?
**A:** GitHub creates temporary redirects from the old URL to the new location, but these redirects break if a new repository is created at the old path (`owner/repo-name`). Redirects are also not guaranteed to persist indefinitely. Update all references (CI/CD configs, documentation, git remotes, package manifests) to the new URL proactively rather than relying on redirects.

---

### Q: The repository transfer failed. What are the most common causes?
**A:** You must have admin access on the source repository AND be an owner of the destination organization. Other causes include: the destination org already has a repo with the same name, the repo uses features not available in the destination (e.g., different plan tier), or there are pending required reviews that block the transfer. Verify permissions on both sides before retrying.

---

### Q: We renamed our organization and now CI/CD pipelines are broken. What do we need to update?
**A:** Update all hardcoded organization names in: GitHub Actions workflow files (especially `actions/checkout` and cross-repo references), git remote URLs on developer machines (`git remote set-url origin`), webhook configurations, package registry references (npm, Maven, NuGet), and any external tools that reference the org name. Run `git remote set-url origin https://github.com/NEW-ORG-NAME/repo.git` on every developer workstation.

---

### Q: Our SCIM/SSO configuration stopped working after an organization rename. How do we fix it?
**A:** Your identity provider (Entra ID, Okta, PingFederate) stores the organization name or URL in its configuration. After renaming the org, update the SAML SSO URL and SCIM endpoint in your IdP to reflect the new organization name. Test SSO login and verify SCIM provisioning is syncing correctly. Failing to update the IdP will prevent users from authenticating or being provisioned.

---

### Q: After transferring a repo to a new org, team access and security configurations are missing. Is that expected?
**A:** Yes. Team access, org-level rulesets, security configurations (secret scanning, code scanning settings), and GitHub App installations do not transfer automatically. The destination org's existing policies will apply instead. You must manually grant team access, verify branch protection rules, reinstall GitHub Apps, and confirm security configurations in the new org.

---

### Q: Can I undo a repository transfer or organization rename?
**A:** There is no built-in "undo" for either operation. For a repository transfer, you can transfer the repo back to the original owner if you still have the necessary permissions. For an organization rename, you can rename the org again to the old name (if no one has claimed it). In both cases, any external references, CI/CD pipelines, and integrations will need to be updated again.

</details>

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Organization Design Patterns (Flat Structure, Teams, Naming) | `Setup/Organization Design Patterns (Flat Structure, Teams, Naming).md` |
| GitHub Enterprise Importer (GEI) & Actions Importer | `Migration/GitHub Enterprise Importer (GEI) & Actions Importer.md` |
| Enterprise Environment Scaffolding Checklist | `Setup/Enterprise Environment Scaffolding Checklist.md` |

---

## 📚 Resources

- [Transferring a repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/transferring-a-repository)
- [Renaming an organization](https://docs.github.com/en/organizations/managing-organization-settings/renaming-an-organization)

---

*Last updated: April 2026*
