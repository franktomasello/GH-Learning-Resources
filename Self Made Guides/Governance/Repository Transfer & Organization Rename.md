# 🔄 Repository Transfer & Organization Rename Runbook

> **Complete guide to transferring repositories between owners and renaming organizations, including what carries over, what breaks, and post-change checklists**

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Transfer repo:** `Repo → Settings → General → Danger Zone → Transfer repository`
- **Rename org:** `Org → Settings → General → Change organization's name`
- **Update git remote (post-rename):** `git remote set-url origin https://github.com/NEW-ORG-NAME/repo.git`

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

---

## ❓ Common Questions & Troubleshooting

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
