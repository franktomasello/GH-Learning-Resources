# 🚚 GitHub Migration with GEI & Actions Importer Runbook

> **Complete guide to migrating repositories, CI/CD pipelines, and history to GitHub using GitHub Enterprise Importer (GEI), Actions Importer, and git mirror push**

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [📋 Overview](#-overview)
- [1️⃣ GEI Setup](#1-gei-setup)
- [2️⃣ Migrating from Azure DevOps](#2-migrating-from-azure-devops)
- [3️⃣ Migrating from GitLab](#3-migrating-from-gitlab)
- [4️⃣ Migrating from Bitbucket Server](#4-migrating-from-bitbucket-server)
- [5️⃣ GitHub-to-GitHub Migration (e.g., Standard to DRUS/GHE.com)](#5-github-to-github-migration-eg-standard-to-drusghecom)
- [6️⃣ What GEI Migrates vs. What Needs Manual Reconfiguration](#6-what-gei-migrates-vs-what-needs-manual-reconfiguration)
- [7️⃣ Actions Importer: Convert CI/CD Pipelines to GitHub Actions](#7-actions-importer-convert-cicd-pipelines-to-github-actions)
- [8️⃣ Manual Mirror Push (Simple Git-Only Migration)](#8-manual-mirror-push-simple-git-only-migration)
- [9️⃣ Post-Migration Checklist](#9-post-migration-checklist)
- [🔟 Phased Migration Approach](#-phased-migration-approach)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📚 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Install GEI CLI:** `gh extension install github/gh-gei`
- **Generate migration script (ADO):** `gh gei generate-script --ado-source-org "Org" --github-target-org "Org"`
- **Migrate single repo:** `gh gei migrate-repo --source-repo "repo" --github-target-org "Org"`
- **Install Actions Importer:** `gh extension install github/gh-actions-importer`
- **Reclaim mannequins:** `gh gei reclaim-mannequin --github-target-org "Org" --csv`

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
| `gh` CLI installed with GEI extension (`gh extension install github/gh-gei`) | ☐ |
| Source platform PAT with appropriate scopes (ADO, GitLab, Bitbucket, or GitHub) | ☐ |
| Target GitHub PAT with `repo`, `admin:org`, `workflow`, `read:packages` scopes | ☐ |
| Target organization created on GitHub | ☐ |
| Network connectivity from migration machine to both source and target platforms | ☐ |

---

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **Azure DevOps organization or project administrator, for ADO migrations** | Creates the source PAT and confirms the migration operator can read source repositories, pull requests, and work items. | Azure DevOps → User settings → Personal access tokens → New Token → select organization → set expiration → scopes such as Code Read, Project and Team Read, and Work Items Read as required by the migration → Create → copy token. Then Azure DevOps → [organization] → Project settings → Permissions or Repositories → validate access. Handoff: ADO org, project, source repo list, PAT owner, and expiration. |
| **GitHub target organization owner** | Creates the destination org and token used by GEI or Actions Importer. | GitHub → profile photo → Your organizations → [target org] → Settings → Member privileges and repository defaults → validate policies. Token: GitHub → profile photo → Settings → Developer settings → Personal access tokens → Tokens (classic) or Fine-grained tokens → Generate new token → grant required repo, admin:org, workflow, and package read permissions for the migration path → Generate token. Handoff: target org, PAT scopes, and migration operator. |
| **Microsoft Entra, Okta, or PingFederate admin, if SAML/SCIM is enforced during migration** | Ensures the migration operator and pilot users are assigned before cutover. | Entra: Enterprise apps → [GitHub app] → Users and groups → Add user/group → Assign. Okta: Applications → [GitHub app] → Assignments → Assign. PingFederate/PingOne: application access or source LDAP group → add migration operator and pilot users. Handoff: SSO login validation and assigned migration group. |

---

## 📋 Overview

| Tool | What It Migrates | Best For |
|------|-----------------|----------|
| **GitHub Enterprise Importer (GEI)** | Repos, PRs, issues, metadata | Full-fidelity org/repo migrations from ADO, GitLab, Bitbucket Server, or GitHub |
| **GitHub Actions Importer** | CI/CD pipeline definitions | Converting Jenkins, Azure Pipelines, GitLab CI, CircleCI, Travis CI, Bamboo, Bitbucket Pipelines to GitHub Actions |
| **`git push --mirror`** | Git history, branches, tags | Simple repo moves where metadata (PRs/issues) is not needed |

---

## 1️⃣ GEI Setup

### Install the GEI CLI Extension

```bash
gh extension install github/gh-gei
```

### Verify Installation

```bash
gh gei --version
```

### Authentication

Set personal access tokens for source and target:

```bash
export GH_PAT=ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
export ADO_PAT=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx   # if migrating from ADO
export GITLAB_API_PRIVATE_TOKEN=glpat-xxxxxxxxxxxxxxxxxx  # if migrating from GitLab
export BBS_USERNAME=admin                                  # if migrating from Bitbucket Server
export BBS_PASSWORD=xxxxxxxx                               # if migrating from Bitbucket Server
```

> 💡 **Tip:** The target GitHub PAT needs `repo`, `admin:org`, `workflow`, and `read:packages` scopes. Source PATs vary by platform.

---

## 2️⃣ Migrating from Azure DevOps

### Generate Migration Script

```bash
gh gei generate-script \
  --ado-source-org "MyADOOrg" \
  --github-target-org "MyGitHubOrg" \
  --ado-pat "$ADO_PAT" \
  --github-target-pat "$GH_PAT" \
  --output migrate.sh
```

### Migrate a Single Repo

```bash
gh gei migrate-repo \
  --ado-source-org "MyADOOrg" \
  --ado-team-project "MyProject" \
  --source-repo "my-repo" \
  --github-target-org "MyGitHubOrg" \
  --target-repo "my-repo" \
  --ado-pat "$ADO_PAT" \
  --github-target-pat "$GH_PAT"
```

> ✅ **Result:** Repo, pull requests, and work item links are migrated to GitHub.

---

## 3️⃣ Migrating from GitLab

### Migrate a Single Repo from GitLab

```bash
gh gei migrate-repo \
  --gitlab-source-org "MyGitLabGroup" \
  --source-repo "my-repo" \
  --github-target-org "MyGitHubOrg" \
  --target-repo "my-repo" \
  --github-target-pat "$GH_PAT"
```

> 💡 **Tip:** Set `GITLAB_API_PRIVATE_TOKEN` as an environment variable. The token needs `read_api` scope at minimum.

---

## 4️⃣ Migrating from Bitbucket Server

### Migrate a Single Repo from Bitbucket Server

```bash
gh gei migrate-repo \
  --bbs-server-url "https://bitbucket.example.com" \
  --bbs-project "PROJ" \
  --source-repo "my-repo" \
  --github-target-org "MyGitHubOrg" \
  --target-repo "my-repo" \
  --bbs-username "$BBS_USERNAME" \
  --bbs-password "$BBS_PASSWORD" \
  --github-target-pat "$GH_PAT"
```

> ⚠️ **Prerequisite:** Bitbucket Server must be accessible from the machine running the migration. For Bitbucket Data Center, use the same `--bbs-server-url` flag.

---

## 5️⃣ GitHub-to-GitHub Migration (e.g., Standard to DRUS/GHE.com)

### Migrate an Entire Organization

```bash
gh gei migrate-org \
  --github-source-org "OldOrg" \
  --github-target-org "NewOrg" \
  --github-source-pat "$SOURCE_GH_PAT" \
  --github-target-pat "$TARGET_GH_PAT"
```

### Migrate a Single Repo Between GitHub Instances

```bash
gh gei migrate-repo \
  --github-source-org "OldOrg" \
  --source-repo "my-repo" \
  --github-target-org "NewOrg" \
  --target-repo "my-repo" \
  --github-source-pat "$SOURCE_GH_PAT" \
  --github-target-pat "$TARGET_GH_PAT" \
  --target-api-url "https://api.SUBDOMAIN.ghe.com"
```

> 💡 **Tip:** When migrating to GHE.com (DRUS), use `--target-api-url` to point to the correct API endpoint.

---

## 6️⃣ What GEI Migrates vs. What Needs Manual Reconfiguration

| Item | ADO | GitLab | Bitbucket Server | GitHub-to-GitHub |
|------|-----|--------|-------------------|------------------|
| **Git source (history, branches, tags)** | ✅ | ✅ | ✅ | ✅ |
| **Pull requests / Merge requests** | ✅ | ✅ | ✅ | ✅ |
| **Issues** | ✅ (from work items) | ✅ | ❌ | ✅ |
| **Comments on PRs** | ✅ | ✅ | ✅ | ✅ |
| **Releases** | N/A | ✅ | N/A | ✅ |
| **Branch protections** | ❌ Manual | ❌ Manual | ❌ Manual | ❌ Manual |
| **Secrets** | ❌ Manual | ❌ Manual | ❌ Manual | ❌ Manual |
| **Webhooks** | ❌ Manual | ❌ Manual | ❌ Manual | ❌ Manual |
| **Actions / Pipelines** | ❌ Use Actions Importer | ❌ Use Actions Importer | ❌ Manual | ✅ (workflows migrate) |
| **CODEOWNERS** | ❌ Manual | ❌ Manual | ❌ Manual | ✅ |
| **GitHub Apps / Integrations** | ❌ Manual | ❌ Manual | ❌ Manual | ❌ Manual |
| **Team permissions** | ❌ Manual | ❌ Manual | ❌ Manual | ❌ Manual |

> ⚠️ **Warning:** Secrets, webhooks, branch protections, and OIDC trust configurations are NEVER migrated automatically. Always plan for manual reconfiguration.

---

## 7️⃣ Actions Importer: Convert CI/CD Pipelines to GitHub Actions

### Install Actions Importer

```bash
gh extension install github/gh-actions-importer
```

### Step A: Audit Existing Pipelines

```bash
gh actions-importer audit \
  --source-url "https://dev.azure.com/MyOrg" \
  --target-url "https://github.com/MyGitHubOrg" \
  --output-dir audit-results
```

> ✅ **Result:** Produces a summary of all pipelines and conversion readiness.

### Step B: Dry-Run a Single Pipeline

```bash
gh actions-importer dry-run \
  --source-url "https://dev.azure.com/MyOrg" \
  --pipeline-id 42 \
  --output-dir dry-run-results
```

> 💡 **Tip:** The dry-run generates the converted workflow YAML without committing anything. Review it before proceeding.

### Step C: Migrate (Commit the Converted Workflow)

```bash
gh actions-importer migrate \
  --source-url "https://dev.azure.com/MyOrg" \
  --pipeline-id 42 \
  --target-url "https://github.com/MyGitHubOrg/my-repo" \
  --output-dir migrate-results
```

> ✅ **Result:** A pull request is opened in the target repo with the converted GitHub Actions workflow.

---

## 8️⃣ Manual Mirror Push (Simple Git-Only Migration)

*Use when you only need Git history, branches, and tags -- no PRs, issues, or metadata.*

### Steps

```bash
# 1. Clone a bare mirror of the source repo
git clone --mirror https://source-platform.example.com/org/repo.git

# 2. Enter the mirrored repo directory
cd repo.git

# 3. Create the target repo on GitHub (if it doesn't already exist)
gh repo create MyGitHubOrg/repo --private --confirm

# 4. Set the remote to GitHub
git remote set-url origin https://github.com/MyGitHubOrg/repo.git

# 5. Push the full mirror
git push --mirror
```

> ⚠️ **Warning:** `git push --mirror` will overwrite everything in the target repo. Only use on an empty target. This does NOT migrate PRs, issues, CI/CD, or any platform metadata.

---

## 9️⃣ Post-Migration Checklist

| Task | Details |
|------|---------|
| **Recreate secrets** | Add all repository and organization secrets in GitHub Settings |
| **Reconfigure webhooks** | Set up webhooks for Slack, Jira, deployment triggers, etc. |
| **Set branch protections** | Configure required reviews, status checks, and merge rules |
| **Update OIDC trusts** | If using OIDC federation for deployments, update the issuer URL to match the new GitHub instance |
| **Reclaim mannequins** | Run `gh gei reclaim-mannequin` to map placeholder users to real GitHub accounts |
| **Verify CI/CD** | Trigger test builds to confirm Actions workflows execute correctly |
| **Update external references** | Update links in wikis, documentation, Jira integrations, and deployment scripts |
| **Verify LFS objects** | Confirm Git LFS files transferred correctly if applicable |
| **Notify teams** | Communicate new repo URLs and any workflow changes |

### Reclaim Mannequins

```bash
gh gei reclaim-mannequin \
  --github-target-org "MyGitHubOrg" \
  --mannequin-user "old-username" \
  --target-user "new-github-username" \
  --github-target-pat "$GH_PAT"
```

> 💡 **Tip:** Export all mannequins first with `gh gei reclaim-mannequin --csv` to get a full mapping file.

---

## 🔟 Phased Migration Approach

*Recommended for large enterprise migrations*

### Phase 1: Pilot (1-2 weeks)

| Activity | Details |
|----------|---------|
| Select 2-5 representative repos | Mix of simple and complex repos |
| Run full migration cycle | GEI + Actions Importer + manual reconfig |
| Validate with repo owners | Confirm history, PRs, and CI/CD work correctly |
| Document gaps and issues | Create a runbook of manual steps specific to your environment |

### Phase 2: Org-by-Org Rollout (2-6 weeks)

| Activity | Details |
|----------|---------|
| Migrate one org/team at a time | Allows focused support and troubleshooting |
| Run audit with Actions Importer | Identify pipelines that need manual conversion |
| Reclaim mannequins per org | Map users as each org is migrated |
| Set up branch protections and secrets | Apply your organization's security standards |

### Phase 3: Cutover

| Activity | Details |
|----------|---------|
| Set source repos to read-only | Prevent new commits to the old platform |
| Final incremental migration | Catch any commits made since Phase 2 |
| Update DNS/bookmarks | Redirect old repo URLs if possible |
| Verify all CI/CD pipelines | Confirm deployments target the correct repos |

### Phase 4: Decommission

| Activity | Details |
|----------|---------|
| Archive source repos | Keep read-only for reference (2-4 weeks recommended) |
| Revoke source PATs | Remove migration credentials |
| Decommission old platform | After confirmation period |

## 🧯 Known Errors & Resolutions

<details>
<summary><em>Show known errors table</em></summary>


> This section lists the known product errors and admin-facing symptoms that commonly occur with this workflow. Exact message text can vary by product rollout, tenant policy, and provider, so use the log or settings page named in the resolution to confirm the root cause.

| Error or symptom | Likely cause | Resolution |
|------------------|--------------|------------|
| **Page, tab, or button is missing** | Wrong account context, missing admin role, unavailable plan/add-on, or feature rollout not enabled for the selected enterprise/org/repo. | Switch to the correct account and scope, confirm the prerequisite role, verify licensing or add-on activation, then refresh the page. If the control is still absent, use the direct settings URL from the relevant GitHub Docs page and confirm the feature is available for your plan. |
| **Changes appear saved but behavior does not change** | Policy inheritance, cached UI state, propagation delay, or an overlapping enterprise/org/repo policy. | Reopen the settings page, verify the effective policy at the lowest affected scope, wait for propagation where documented, and check for a stricter policy at an enterprise or organization level. |
| **403, forbidden, or resource not accessible** | The signed-in user or token can see the page but lacks the specific permission for the action. | Use an enterprise owner, organization owner, repository admin, or token with the exact scopes/permissions listed in the runbook. For SAML-protected orgs, authorize the token or SSH key for SSO before retrying. |
| **401, 403, missing permissions, or SAML enforcement error** | The source or target token lacks required scopes/roles or has not been authorized for a SAML-protected organization. | Upgrade the GEI or Actions Importer CLI extension, create tokens with the required scopes, authorize them for SSO, and verify source/target org ownership or migrator roles. |
| **404 Not Found during migration** | The source URL, org slug, repo name, API endpoint, or target org is wrong or inaccessible. | Copy the exact source and target slugs from the UI, verify the API URL for GHE.com or GHES, and run a small pilot migration before the production batch. |
| **Archive generation failed, timeout, or large repository failure** | Repository size, LFS objects, unreachable source storage, or transient source export failure. | Retry once with the latest CLI, inspect the verbose migration log, reduce repository size where possible, and migrate a comparable repository to isolate source-specific data issues. |
| **Mannequin or placeholder users remain after migration** | Source identities were not mapped to GitHub users during migration. | Export mannequins, build an owner-approved mapping file, reclaim mannequins to the correct target accounts, and document any intentionally unmapped identities. |
| **Actions Importer output is invalid or incomplete** | The source pipeline uses tasks, service connections, templates, or approvals that do not have a direct Actions equivalent. | Use audit and dry-run output, manually rewrite unsupported tasks, recreate secrets/environments/service connections, and test generated workflows in a branch before cutover. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>


### Q: GEI migration fails with a timeout error on a large repository. How do I resolve this?
**A:** Large repositories can exceed the default timeout. Try using `--queue-only` to queue the migration and let it run asynchronously, then check status with `gh gei wait-for-migration`. For very large repos, also try setting `--target-repo-visibility` explicitly. If the issue persists, consider splitting the migration (e.g., migrate git history first with `git push --mirror`, then use GEI for metadata).

---

### Q: After migration, I see "mannequin" placeholder accounts instead of real users in PRs and issues. How do I fix this?
**A:** GEI creates mannequin (placeholder) accounts for users from the source platform who do not have a corresponding GitHub account. Reclaim these mannequins by navigating to Enterprise Settings > People > Mannequins, or use the CLI: `gh gei reclaim-mannequin --github-target-org "MyOrg" --mannequin-user "old-user" --target-user "github-user"`. Export all mannequins first with `--csv` to plan the mapping.

---

### Q: Some pull requests are missing after migration. What could have gone wrong?
**A:** The most common cause is API rate limiting on the source platform. GEI fetches PRs via the source API, and rate limits can cause incomplete transfers. Rerun the migration for the specific repository -- GEI is idempotent and will pick up missed items. Also check that the source PAT has sufficient scopes to read all PR data.

---

### Q: Actions Importer is producing invalid YAML output for some pipelines. How should I handle this?
**A:** Actions Importer handles straightforward pipeline constructs well but may struggle with complex conditional logic, custom plugins, or advanced template expressions. Always use `gh actions-importer dry-run` first to generate the YAML without committing it, then manually review and fix any syntax issues. Treat the output as a starting point that may require hand-editing for complex pipelines.

---

### Q: Are secrets (Actions secrets, environment variables) migrated by GEI?
**A:** No. Secrets are never migrated automatically by any migration tool. You must manually recreate all repository secrets, organization secrets, and environment secrets on the target GitHub instance. Inventory your secrets before migration and have the values ready for re-entry. This is by design for security reasons.

---

### Q: Can I do a dry-run or test migration before the real cutover?
**A:** Yes. Run GEI against a test target organization first to validate the migration without affecting production. For Actions Importer, use `gh actions-importer dry-run` to generate converted workflow YAML without committing it. The phased approach in Section 10 of this guide (Pilot > Org-by-Org Rollout > Cutover > Decommission) is recommended for production migrations.

</details>

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Standard Enterprise to EMU Migration | `Setup/Standard Enterprise to EMU Migration.md` |
| Organization Design Patterns (Flat Structure, Teams, Naming) | `Setup/Organization Design Patterns (Flat Structure, Teams, Naming).md` |
| Azure Boards + GitHub Integration | `Migration/Azure Boards + GitHub Integration.md` |
| SVN to GitHub | `Migration/SVN to GitHub.md` |
| OIDC Federation for Azure Deployments | `Actions/OIDC Federation for Azure Deployments.md` |

---

## 📚 Resources

- [GitHub Enterprise Importer Documentation](https://docs.github.com/en/migrations/using-github-enterprise-importer)
- [GitHub Actions Importer Documentation](https://docs.github.com/en/actions/tutorials/migrate-to-github-actions/automated-migrations/azure-devops-migration)

---

*Last updated: April 2026*
