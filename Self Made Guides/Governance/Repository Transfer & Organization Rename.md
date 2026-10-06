# 🔄 GitHub Repository Transfer & Organization Rename Runbook

> **Complete guide to transferring repositories between owners and renaming organizations, including what carries over, what breaks, and post-change checklists**

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [📋 Overview](#-overview)
- [1️⃣ Transfer a Repository](#1️⃣-transfer-a-repository)
- [2️⃣ What Transfers Automatically](#2️⃣-what-transfers-automatically)
- [3️⃣ Post-Transfer Checklist](#3️⃣-post-transfer-checklist)
- [4️⃣ Rename an Organization](#4️⃣-rename-an-organization)
- [5️⃣ What Redirects — and What Doesn't](#5️⃣-what-redirects--and-what-doesnt)
- [6️⃣ What to Update Manually](#6️⃣-what-to-update-manually)
- [7️⃣ Plan as a Coordinated Event](#7️⃣-plan-as-a-coordinated-event)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📚 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Transfer repo:** `Repo → Settings` → **Danger Zone** → **Transfer** → choose the new owner → type the repo name → **I understand, transfer this repository**
- **Rename org:** `Org → Settings` → **Danger zone** → **Rename organization** → **I understand, let's rename my organization** → new name → **Change organization's name**
- **Update git remotes afterward:** `git remote set-url origin https://github.com/NEW-OWNER/REPO.git`

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
| Transfer a repository | **Admin** on the repository **and** permission to create repositories in the destination organization | ☐ |
| Rename the repository during transfer | **Owner** of the destination organization | ☐ |
| Rename an organization | **Organization owner** | ☐ |
| Update SAML / SCIM after an org rename | IdP administrator | ☐ |
| Communication plan for affected teams | Program owner | ☐ |
| Inventory of integrations, CI/CD, SSO, packages, and external references | Platform team | ☐ |

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **GitHub organization owner or enterprise owner** | Renames or transfers only after identity and integration owners are ready to update downstream references. | For organization rename: GitHub → profile picture → Organizations → [organization] → Settings → Danger zone → Rename organization → I understand, let's rename my organization → type the new name → Change organization's name. Repository transfer: GitHub → [owner/repository] → Settings → Danger Zone → Transfer → choose the new owner → type the repository name → I understand, transfer this repository. Handoff: old URL, new URL, redirect status, and cutover time. |
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

**👤 Role:** Repository **admin** with permission to create repositories in the destination organization · **📍 Portal:** GitHub

**Navigate:** Repository → **Settings** → **Danger Zone** *(bottom of the page)*

**Before you start — transfer limits:**

- The destination can't already have a repository with the same name (or a fork in the same network).
- Repositories can't move into or out of an **Enterprise Managed Users** enterprise.
- **Internal** repositories can only move to another organization in the **same** enterprise.
- Single forks of a private or internal upstream network can't be transferred.

**Steps:**

1. At the bottom of the **Settings** page, in **Danger Zone**, click **Transfer**.
2. Under **New owner**, choose **Select one of my organizations** (then pick it from the dropdown) or **Specify an organization or username** (then type it).
3. *(Optional — destination org owners only)* Enter a new **Repository name**.
4. Read the warnings about features the new owner's plan might not include.
5. Type the repository name to confirm, then click **I understand, transfer this repository**.

> ✅ **Result:** the repository moves to the new owner, and links, `git clone`, `git fetch`, and `git push` to the old location redirect automatically. Transfers to a **personal account** wait for that person to accept by email (within one day).

> ⚠️ **Name retirement:** if the repository had a GitHub Marketplace action, or more than 100 clones or Actions uses in the week before the transfer, GitHub permanently retires the old `OWNER/REPOSITORY` name.

---

## 2️⃣ What Transfers Automatically

| Item | Transfers? | Notes |
|------|-----------|-------|
| **Git history** | ✅ | Commits, branches, tags, and contribution attribution |
| **Issues and pull requests** | ✅ | Some assignees and issue types may be cleared depending on the new owner |
| **Wiki, stars, watchers** | ✅ | |
| **Webhooks, services, secrets, deploy keys** | ✅ | Stay associated with the repository |
| **Git LFS objects** | ✅ | Moved in the background |
| **Forks** | ✅ | Stay connected to the network |
| **Packages** | ⚠️ Depends | May transfer or lose their link, depending on the registry |
| **GitHub Pages site** | ⚠️ Not redirected | The site URL changes; update custom domains first |

> 💡 **Tip:** redirects are permanently deleted if someone creates a new repository or fork at the old location. Update references instead of relying on them.

---

## 3️⃣ Post-Transfer Checklist

| Item | Action required |
|------|----------------|
| **Team access** | Grant the destination organization's teams access — the org's **default repository permission** applies automatically |
| **Collaborators** | The original owner is added as a collaborator — remove if not needed |
| **Security configuration** | Apply a security configuration by hand — default configurations only attach to **new** repositories |
| **Rulesets and policies** | Check which destination org and enterprise rulesets now target the repository |
| **GitHub Apps and integrations** | Install or reconfigure org-level apps in the destination org |
| **CODEOWNERS** | Update team names if they differ in the destination org |
| **CI/CD references** | Update hard-coded `owner/repo` paths (Actions `uses:`, checkout, external pipelines) |
| **Local clones** | `git remote set-url origin NEW_URL` |

---

# Part 2: Organization Rename

---

## 4️⃣ Rename an Organization

**👤 Role:** **Organization owner** · **📍 Portal:** GitHub

**Navigate:** Organization → **Settings** → **Danger zone** *(bottom of the page)*

**Steps:**

1. Near the bottom of the settings page, under **Danger zone**, click **Rename organization**.
2. Read the warnings, then click **I understand, let's rename my organization**.
3. Type the new name, then click **Change organization's name**.

> ✅ **Result:** the organization is renamed, and repository links redirect to the new name within a few minutes.

> ⚠️ **Warning:** the old name becomes available for anyone to claim. If someone takes it, your redirects stop working.

---

## 5️⃣ What Redirects — and What Doesn't

| Works after the rename | Breaks after the rename |
|------------------------|-------------------------|
| Web links to **repositories** | The organization **profile page** (`github.com/old-name`) returns 404 |
| `git push` / `git pull` to old remote URLs (until someone claims the old name) | **API requests** using the old organization name return 404 |
| Commit attribution | `@old-name/team` mentions don't redirect |
| Packages and container images move to the new namespace | **SAML SSO** and **SCIM** stop working until the IdP app is updated |

> ⚠️ **Name retirement:** if the organization had public repositories with a Marketplace action, or with more than 100 clones or Actions uses in the week before the rename, GitHub permanently retires those `OLD-OWNER/REPOSITORY` combinations.

---

## 6️⃣ What to Update Manually

The following must be updated after renaming the organization:

| Item | Action Required |
|------|----------------|
| **Git remotes** | All developers must update their local git remotes to the new URL |
| **API scripts and tools** | Replace the old organization name — API calls with it return 404 |
| **Webhooks** | Update receivers that check the organization name in payloads |
| **GitHub Apps** | Reconfigure any GitHub Apps that reference the old organization name |
| **CI/CD configurations** | Update all pipeline configs (Actions workflows, Jenkins, CircleCI, etc.) that reference the old org name |
| **SAML SSO / SCIM** | Update the organization name in the GitHub app on your IdP (Entra ID, Okta, PingFederate) — or members can't sign in and provisioning stops |
| **Documentation and wikis** | Update internal docs, READMEs, and runbooks that reference the old org name |
| **Package registries** | Update references in package manifests (npm, Maven, NuGet, etc.) |
| **Actions `uses:` references** | Update `old-name/repo@ref` references to other repositories in the organization |
| **Links to the org profile** | Update links on other sites — the old profile URL returns 404 |

### Git remote update command

Developers update each local clone with:

```bash
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
**A:** Redirects are permanently deleted if someone creates a new repository or fork at the old `owner/repo-name` location. GitHub Pages sites aren't redirected at all. Update CI/CD configs, documentation, git remotes, and package manifests to the new URL rather than relying on redirects.

---

### Q: The repository transfer failed. What are the most common causes?
**A:** Check that you're an admin on the repository and can create repositories in the destination organization (an org or enterprise policy may block it). Other causes: the destination already has a repository or fork with that name, you're crossing the EMU boundary, you're moving an internal repository to a different enterprise, or the repository is a fork of a private or internal network.

---

### Q: We renamed our organization and now CI/CD pipelines are broken. What do we need to update?
**A:** API calls with the old organization name return 404, so update every hard-coded name: Actions workflows (`uses:` and cross-repo checkouts), external CI tools, scripts that call the API, package registry references (npm, Maven, NuGet), and webhook receivers. Developers should also run `git remote set-url origin https://github.com/NEW-ORG-NAME/repo.git` in each clone.

---

### Q: Our SCIM/SSO configuration stopped working after an organization rename. How do we fix it?
**A:** For organization-level SAML and SCIM, update the organization name in the GitHub app on your IdP (Entra ID, Okta, PingFederate) — the SAML URLs and the SCIM base URL include it. Then test SSO sign-in and SCIM provisioning. Until you do, members can't authenticate and users can't be provisioned or deprovisioned. (Enterprise-level SAML and EMU use the enterprise URL, which an org rename doesn't change.)

---

### Q: After transferring a repo to a new org, team access and security configurations are missing. Is that expected?
**A:** Yes. Team access, organization rulesets, and app installations belong to the organization, not the repository. In the destination, grant team access, check which rulesets target the repository, install the needed GitHub Apps, and apply a security configuration by hand — default configurations only attach to newly created repositories.

---

### Q: Can I undo a repository transfer or organization rename?
**A:** There's no undo button. You can transfer a repository back if you still have the right permissions, or rename the organization back if nobody has claimed the old name — unless GitHub permanently retired the old name. Either way, external references and integrations need updating again.

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

*Last updated: October 2026*
