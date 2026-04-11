# 🔄 Repository Transfer & Organization Rename Runbook

> **Complete guide to transferring repositories between owners and renaming organizations, including what carries over, what breaks, and post-change checklists**

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

## 📚 Resources

- [Transferring a repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/transferring-a-repository)
- [Renaming an organization](https://docs.github.com/en/organizations/managing-organization-settings/renaming-an-organization)

---

*Last updated: April 2026*
