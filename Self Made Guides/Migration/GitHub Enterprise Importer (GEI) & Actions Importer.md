# 🚚 Migration to GitHub (GEI & Actions Importer) Runbook

> **Complete guide to migrating repositories, CI/CD pipelines, and history to GitHub using GitHub Enterprise Importer (GEI), Actions Importer, and git mirror push**

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

---

## 📚 Resources

- [GitHub Enterprise Importer Documentation](https://docs.github.com/en/migrations/using-github-enterprise-importer)
- [GitHub Actions Importer Documentation](https://docs.github.com/en/actions/migrating-to-github-actions/using-github-actions-importer)

---

*Last updated: April 2026*