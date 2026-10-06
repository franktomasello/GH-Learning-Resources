# 🚚 GitHub Migration with GEI & Actions Importer Runbook

> **Complete guide to migrating repositories, CI/CD pipelines, and history to GitHub using GitHub Enterprise Importer (GEI), Actions Importer, and git mirror push**

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [📋 Overview](#-overview)
- [1️⃣ GEI Setup](#1️⃣-gei-setup)
- [2️⃣ Migrating from Azure DevOps](#2️⃣-migrating-from-azure-devops)
- [3️⃣ Migrating from GitLab](#3️⃣-migrating-from-gitlab)
- [4️⃣ Migrating from Bitbucket Server / Data Center](#4️⃣-migrating-from-bitbucket-server--data-center)
- [5️⃣ GitHub-to-GitHub Migration (e.g., GitHub.com to GHE.com)](#5️⃣-github-to-github-migration-eg-githubcom-to-ghecom)
- [6️⃣ What GEI Migrates vs. What Needs Manual Reconfiguration](#6️⃣-what-gei-migrates-vs-what-needs-manual-reconfiguration)
- [7️⃣ Actions Importer: Convert CI/CD Pipelines to GitHub Actions](#7️⃣-actions-importer-convert-cicd-pipelines-to-github-actions)
- [8️⃣ Manual Mirror Push (Simple Git-Only Migration)](#8️⃣-manual-mirror-push-simple-git-only-migration)
- [9️⃣ Post-Migration Checklist](#9️⃣-post-migration-checklist)
- [🔟 Phased Migration Approach](#-phased-migration-approach)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📚 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Install the right extension:** `gh extension install github/gh-ado2gh` (Azure DevOps) · `github/gh-bbs2gh` (Bitbucket Server/DC) · `github/gh-gl2gh` (GitLab) · `github/gh-gei` (GitHub → GitHub)
- **Generate a script (ADO):** `gh ado2gh generate-script --ado-org SOURCE --github-org DESTINATION --output migrate.ps1`
- **Migrate one repo (GitHub → GitHub):** `gh gei migrate-repo --github-source-org SOURCE --source-repo REPO --github-target-org DEST --target-repo REPO`
- **GHE.com target:** add `--target-api-url https://api.SUBDOMAIN.ghe.com`
- **Actions Importer:** `gh extension install github/gh-actions-importer` → `gh actions-importer configure` → `audit` / `dry-run` / `migrate`
- **Reclaim mannequins:** `gh gei generate-mannequin-csv …` → edit → `gh gei reclaim-mannequin --github-target-org DEST --csv FILE.csv`

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

## ✅ Prerequisites

| Requirement | Who / Role needed | ✓ |
|-------------|-------------------|---|
| GitHub CLI 2.4.0+ and the extension for your source | Migration operator | ☐ |
| Destination organization on GitHub Enterprise Cloud (GitHub.com or GHE.com) | GitHub **enterprise owner** | ☐ |
| **Organization owner** of the destination, or the **migrator role** granted on it | Org owner grants with `grant-migrator-role` | ☐ |
| Destination **classic** PAT — org owner: `repo`, `workflow`, `admin:org`; migrator: `repo`, `workflow`, `read:org` | Migration operator | ☐ |
| Source access — ADO PAT with **Work Items (Read)**, **Code (Read)**, **Identity (Read)** (full access recommended for `inventory-report`); GitLab/Bitbucket/GitHub tokens per the docs | Source admin | ☐ |
| "Repository migrations" added to the **bypass list** of destination rulesets (or rulesets that could block history) | Org/enterprise owner | ☐ |
| Docker (for Actions Importer) | Migration operator | ☐ |

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **Azure DevOps organization or project administrator, for ADO migrations** | Creates the source PAT and confirms the migration operator can read source repositories, pull requests, and work items. | Azure DevOps → User settings → Personal access tokens → New Token → select organization → set expiration → scopes Work Items (Read), Code (Read), and Identity (Read) — or Full access if you'll run inventory-report → Create → copy token. Then Azure DevOps → [organization] → Project settings → Permissions or Repositories → validate access. Handoff: ADO org, project, source repo list, PAT owner, and expiration. |
| **GitHub target organization owner** | Creates the destination org and token used by GEI or Actions Importer. | GitHub → profile picture → Organizations → [target org] → Settings → Member privileges and repository defaults → validate policies. Token: GitHub → profile picture → Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token (classic) → scopes repo, workflow, and admin:org (or read:org for a migrator) → Generate token → authorize it for SSO if required. Grant non-owners the migrator role with `gh <extension> grant-migrator-role`. Handoff: target org, PAT scopes, and migration operator. |
| **Microsoft Entra, Okta, or PingFederate admin, if SAML/SCIM is enforced during migration** | Ensures the migration operator and pilot users are assigned before cutover. | Entra: Enterprise apps → [GitHub app] → Users and groups → Add user/group → Assign. Okta: Applications → [GitHub app] → Assignments → Assign. PingFederate/PingOne: application access or source LDAP group → add migration operator and pilot users. Handoff: SSO login validation and assigned migration group. |

---

## 📋 Overview

| Tool | What it migrates | Best for |
|------|-----------------|----------|
| **GitHub Enterprise Importer (GEI)** | Repositories plus source-specific metadata (pull requests, and for some sources issues, releases, etc.) | Migrations to GitHub Enterprise Cloud from Azure DevOps Cloud, Bitbucket Server/Data Center, GitLab, GitHub.com, or GHES |
| **GitHub Actions Importer** | CI/CD pipeline definitions → Actions workflows | Azure DevOps, Jenkins, GitLab CI, CircleCI, Travis CI, Bamboo, Bitbucket Pipelines |
| **`git push --mirror`** | Git history, branches, tags | Simple moves where PRs and issues aren't needed |

| Source | GEI extension | Install |
|--------|---------------|---------|
| Azure DevOps **Cloud** (not Azure DevOps Server) | `ado2gh` | `gh extension install github/gh-ado2gh` |
| Bitbucket Server / Data Center | `bbs2gh` | `gh extension install github/gh-bbs2gh` |
| GitLab (GitLab.com or self-managed) | `gl2gh` | `gh extension install github/gh-gl2gh` |
| GitHub.com, GHES, or another GHE.com enterprise | `gei` | `gh extension install github/gh-gei` |

> 💡 Run `gh extension upgrade github/gh-<name>` before each migration wave — the extensions are updated often.

---

## 1️⃣ GEI Setup

### Install and verify

```bash
gh --version                                 # 2.4.0 or newer
gh extension install github/gh-ado2gh        # pick the extension for your source
gh extension upgrade github/gh-ado2gh
gh ado2gh --help
```

### Authentication

Set tokens as environment variables (or pass them as flags):

```bash
export GH_PAT="TOKEN"            # destination (classic PAT)
export ADO_PAT="TOKEN"           # Azure DevOps source
export GITLAB_PAT="TOKEN"        # GitLab source
export BBS_USERNAME="USER"       # Bitbucket Server source
export BBS_PASSWORD="PASSWORD"
export GH_SOURCE_PAT="TOKEN"     # GitHub source (GEI)
```

### Grant the migrator role (if the operator isn't an org owner)

```bash
gh ado2gh grant-migrator-role --github-org DESTINATION --actor USERNAME --actor-type USER
```

> 💡 **GHE.com:** add `--target-api-url https://api.SUBDOMAIN.ghe.com` to commands that talk to the destination.

---

## 2️⃣ Migrating from Azure DevOps

> 📌 GEI supports **Azure DevOps Cloud** only. For Azure DevOps Server, migrate to Azure DevOps Cloud first.

### Generate a migration script

```bash
gh ado2gh generate-script \
  --ado-org SOURCE \
  --github-org DESTINATION \
  --output migrate.ps1
```

Useful flags: `--all` (also rewires pipelines, creates teams, configures Boards integration), `--download-migration-logs`, `--target-api-url` (GHE.com). Review the script, then run it.

### Migrate a single repository

```bash
gh ado2gh migrate-repo \
  --ado-org SOURCE \
  --ado-team-project TEAM-PROJECT \
  --ado-repo CURRENT-NAME \
  --github-org DESTINATION \
  --github-repo NEW-NAME
```

> ✅ **Result:** Git history, pull requests (with user history, attachments, and **work item links**), and repository branch policies are migrated. Azure Boards work items and Azure Pipelines are **not** migrated — keep using them through the integrations, or convert pipelines with Actions Importer.

---

## 3️⃣ Migrating from GitLab

```bash
gh gl2gh migrate-repo \
  --gitlab-server-url https://gitlab.example.com \
  --gitlab-group SOURCE_GROUP \
  --gitlab-project SOURCE_PROJECT \
  --github-org DESTINATION \
  --github-repo NEW_REPO_NAME \
  --use-github-storage
```

> 💡 GL2GH exports the project archive, uploads it (GitHub-owned storage or your own AWS S3 / Azure Blob Storage), then imports it. Use `gh gl2gh generate-script` for many projects. GitLab.com export archives are limited to 40 GB.

---

## 4️⃣ Migrating from Bitbucket Server / Data Center

```bash
gh bbs2gh migrate-repo \
  --bbs-server-url https://bitbucket.example.com \
  --bbs-project PROJ \
  --bbs-repo CURRENT-NAME \
  --github-org DESTINATION \
  --github-repo NEW-NAME \
  --use-github-storage
```

> ⚠️ **Prerequisites:** the migration machine must reach Bitbucket. The archive is generated on the Bitbucket server — provide `--ssh-user`/`--ssh-private-key` (or `--smb-user` on Windows) so BBS2GH can download it, and choose storage (`--use-github-storage`, `--aws-bucket-name`, or Azure). Use `gh bbs2gh generate-script` for many repositories.

---

## 5️⃣ GitHub-to-GitHub Migration (e.g., GitHub.com to GHE.com)

### Migrate an entire organization (GitHub.com source)

```bash
gh gei migrate-org \
  --github-source-org SOURCE \
  --github-target-org NEW-ORG \
  --github-target-enterprise DESTINATION-ENTERPRISE
```

> 📌 Org migration brings teams, repositories, team repository access, member privileges, org webhooks (re-enable afterward), and the default branch name. **Team membership isn't migrated**, and all repositories arrive **private**.

### Migrate a single repository

```bash
gh gei migrate-repo \
  --github-source-org SOURCE \
  --source-repo CURRENT-NAME \
  --github-target-org DESTINATION \
  --target-repo NEW-NAME \
  --target-api-url https://api.SUBDOMAIN.ghe.com
```

Useful flags: `--target-repo-visibility` (defaults to private), `--skip-releases` (if releases exceed 10 GB), `--queue-only` with `gh gei wait-for-migration --migration-id ID`.

---

## 6️⃣ What GEI Migrates vs. What Needs Manual Reconfiguration

| Item | Azure DevOps | GitLab | Bitbucket Server | GitHub → GitHub |
|------|:-:|:-:|:-:|:-:|
| **Git source (history, branches, tags)** | ✅ | ✅ (+ wiki) | ✅ | ✅ |
| **Pull requests / merge requests** | ✅ | ✅ | ✅ | ✅ |
| **Issues** | ❌ (Boards stays in ADO) | ✅ | ❌ | ✅ |
| **Milestones / releases** | ❌ | ✅ | ❌ | ✅ |
| **Branch protections / policies** | ✅ Repo branch policies (not user-scoped or cross-repo) | ❌ | ❌ | ✅ (some exceptions) |
| **Webhooks** | ❌ | ❌ | ❌ | ✅ (re-enable after) |
| **Actions workflows / pipelines** | ❌ Use Actions Importer | ❌ Use Actions Importer | ❌ | ✅ workflow files |
| **Secrets, variables, environments, runners** | ❌ | ❌ | ❌ | ❌ |
| **Rulesets, packages, Projects, Dependabot/code scanning alerts** | ❌ | ❌ | ❌ | ❌ |
| **Teams and user access** | ❌ | ❌ | ❌ | Teams + team repo access (org migrations only) |
| **Git LFS objects** | ❌ Push afterward | ❌ Push afterward | ❌ Push afterward | ❌ Push afterward |

> ⚠️ **Warning:** secrets, environments, runners, rulesets, app installations, and OIDC trusts are never migrated. Plan them as manual follow-up.

> 📌 **Size limits:** 40 GiB of Git source per repository (public preview), 400 MiB per file during migration (100 MiB afterward), 2 GiB per commit or push.

---

## 7️⃣ Actions Importer: Convert CI/CD Pipelines to GitHub Actions

### Install and configure

```bash
gh extension install github/gh-actions-importer
gh actions-importer update          # pulls the latest container image (needs Docker)
gh actions-importer configure       # choose your CI provider and enter tokens/URLs
```

### Step A — Audit

```bash
gh actions-importer audit azure-devops --output-dir tmp/audit
```

> ✅ **Result:** a summary of every pipeline and how much converts automatically. `forecast` estimates runner usage.

### Step B — Dry-run one pipeline

```bash
gh actions-importer dry-run azure-devops pipeline \
  --pipeline-id 42 \
  --output-dir tmp/dry-run
```

> 💡 **Tip:** the dry run writes converted workflow YAML locally without changing anything. Review it carefully.

### Step C — Migrate (opens a pull request)

```bash
gh actions-importer migrate azure-devops pipeline \
  --pipeline-id 42 \
  --target-url https://github.com/octo-org/octo-repo \
  --output-dir tmp/migrate
```

> ✅ **Result:** a pull request with the converted workflow opens in the target repository. Classic release pipelines use `… azure-devops release --pipeline-id`. Other providers use their own subcommand (`jenkins`, `gitlab`, `circle-ci`, `travis-ci`, `bamboo`, `bitbucket`).

---

## 8️⃣ Manual Mirror Push (Simple Git-Only Migration)

*Use when you only need Git history, branches, and tags — no PRs, issues, or metadata.*

### Steps

```bash
# 1. Clone a bare mirror of the source repo
git clone --mirror https://source-platform.example.com/org/repo.git

# 2. Enter the mirrored repo directory
cd repo.git

# 3. Create an empty target repo on GitHub
gh repo create MyGitHubOrg/repo --private

# 4. Point the remote at GitHub
git remote set-url origin https://github.com/MyGitHubOrg/repo.git

# 5. Push the full mirror
git push --mirror
```

> ⚠️ **Warning:** `git push --mirror` overwrites the target. Use it only on an empty repository. It doesn't migrate PRs, issues, CI/CD, or LFS objects (push those with `git lfs push --all`).

---

## 9️⃣ Post-Migration Checklist

| Task | Details |
|------|---------|
| **Check migration logs** | `gh <extension> download-logs` (or the `migration-log` issue in the repo) |
| **Recreate secrets, variables, environments** | Repository, organization, and environment level |
| **Re-enable webhooks / reconfigure integrations** | Migrated webhooks are disabled; others must be recreated |
| **Rulesets and branch protection** | Recreate what wasn't migrated |
| **Team access and membership** | Add members to teams; fix team references in `CODEOWNERS` |
| **Update OIDC trusts** | New issuer for GHE.com and new subjects (see `Actions/OIDC Federation for Azure Deployments.md`) |
| **Reclaim mannequins** | CLI or browser (below) |
| **Push Git LFS objects** | `git lfs push --all` to the new remote |
| **Verify CI/CD** | Run test builds |
| **Update external references** | Wikis, docs, Jira/Boards, deployment scripts |
| **Notify teams** | New URLs and any workflow changes |

### Reclaim mannequins

**CLI (recommended):**

```bash
gh gei generate-mannequin-csv --github-target-org DESTINATION --output mannequins.csv
# edit mannequins.csv: add the target username for each mannequin
gh gei reclaim-mannequin --github-target-org DESTINATION --csv mannequins.csv
```

Use `gh ado2gh`, `gh bbs2gh`, or `gh gl2gh` with the same subcommands for those sources. For one user: `gh gei reclaim-mannequin --github-target-org DESTINATION --mannequin-user OLD --target-user NEW`.

**Browser:** Organization → **Settings** → **Import/Export** (under "Access") → **Reattribute** next to the mannequin → pick the member → **Invite**.

> 📌 Members must already be in the organization. They get an attribution invitation and the mannequin is reclaimed when they accept (track it under **Attribution Invitations**).

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
**A:** Start it with `--queue-only` and track it with `wait-for-migration --migration-id ID`, so your terminal session doesn't have to stay open. Check the repository against the limits (40 GiB Git source, 400 MiB per file, 2 GiB per commit) with `git-sizer`, and move large files to Git LFS. Download the migration log to see where it failed.

---

### Q: After migration, I see "mannequin" placeholder accounts instead of real users in PRs and issues. How do I fix this?
**A:** Mannequins are placeholders for source users. Reclaim them in bulk with `generate-mannequin-csv` → edit → `reclaim-mannequin --csv`, one at a time with `--mannequin-user` and `--target-user`, or in the browser at Organization → **Settings** → **Import/Export** → **Reattribute**. The target user must already be an organization member and accept the attribution invitation.

---

### Q: Some pull requests are missing after migration. What could have gone wrong?
**A:** Download the migration log (`download-logs`) — it lists items that couldn't be migrated and why. Common causes: missing source token scopes, data the source doesn't export, or PRs whose head branch was deleted. To retry, delete the destination repository and migrate it again — GEI won't migrate into an existing repository.

---

### Q: Actions Importer is producing invalid YAML output for some pipelines. How should I handle this?
**A:** Expect some manual work. Run `audit` to see which constructs convert, use `dry-run` to generate YAML locally, and fix what's left (custom tasks, plugins, complex templates). Custom transformers can automate repeated fixes. Treat the output as a starting point and test it before merging the PR from `migrate`.

---

### Q: Are secrets (Actions secrets, environment variables) migrated by GEI?
**A:** No. Actions secrets, variables, environments, Dependabot and Codespaces secrets, and webhook secrets are never migrated. Inventory them before the migration and recreate them in the destination (repository, organization, and environment level).

---

### Q: Can I do a dry-run or test migration before the real cutover?
**A:** Yes. Run a trial migration into a test organization (the source isn't changed), validate it, then delete the test repositories. For pipelines, use `gh actions-importer dry-run`. Follow the phased approach in Section 🔟 (Pilot → Org-by-Org → Cutover → Decommission).

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

- [GitHub Enterprise Importer documentation](https://docs.github.com/en/migrations/using-github-enterprise-importer)
- [Migrations from Azure DevOps to GitHub](https://docs.github.com/en/migrations/ado/understand-migrations-from-azure-devops-to-github)
- [Migrations from GitLab](https://docs.github.com/en/migrations/using-github-enterprise-importer/migrate-from-gitlab/understand-migrations)
- [About migrations between GitHub products](https://docs.github.com/en/migrations/using-github-enterprise-importer/migrating-between-github-products/about-migrations-between-github-products)
- [Reclaiming mannequins](https://docs.github.com/en/migrations/using-github-enterprise-importer/completing-your-migration-with-github-enterprise-importer/reclaiming-mannequins-for-github-enterprise-importer)
- [Migrating from Azure DevOps with GitHub Actions Importer](https://docs.github.com/en/actions/tutorials/migrate-to-github-actions/automated-migrations/azure-devops-migration)

---

*Last updated: October 2026*
