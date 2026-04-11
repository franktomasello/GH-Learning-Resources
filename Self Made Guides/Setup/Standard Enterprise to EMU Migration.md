# 🔄 Standard Enterprise to EMU Migration Runbook

> Step-by-step guide to migrating from standard GitHub Enterprise Cloud (personal accounts) to Enterprise Managed Users

---

## 📋 Overview

Migrating from standard GitHub Enterprise to EMU is a **one-way migration** that moves from user-owned accounts to enterprise-owned, IdP-controlled accounts. This requires careful planning.

| Aspect | Standard Enterprise | EMU (Target) |
|--------|-------------------|--------------|
| Account ownership | User owns account | Enterprise owns account |
| Authentication | Personal login + optional SAML | IdP-managed via SAML/OIDC |
| Provisioning | Invitation-based | SCIM required |
| Public repos/gists | Supported | Not supported |
| Open-source contribution | Same account | Requires separate personal account |

## 1️⃣ Pre-Migration Inventory

Before starting, catalog everything in your current enterprise:

### Checklist
- [ ] All repositories (names, sizes, visibility, LFS usage)
- [ ] All organization memberships and team structures
- [ ] Repository permissions and CODEOWNERS files
- [ ] Branch protection rules and rulesets
- [ ] GitHub Apps and OAuth Apps installed
- [ ] Webhooks (org-level and repo-level)
- [ ] Actions secrets, variables, and environment configurations
- [ ] Personal Access Tokens (PATs) used for automation
- [ ] SSH keys used for CI/CD
- [ ] GitHub Packages registries in use
- [ ] OIDC trust relationships (AWS, Azure, GCP)

> ⚠️ **Important:** PATs from personal accounts will NOT work after migration. Plan to replace them with GitHub Apps or fine-grained PATs issued to managed accounts.

## 2️⃣ Create the New EMU Enterprise

1. Work with your GitHub account team to provision a new EMU enterprise
2. Receive the **setup user** account (`SHORTCODE_admin`)
3. In a private/incognito window:
   - Set password for the setup user
   - Enable 2FA
   - Save recovery codes securely

```
GitHub (as SHORTCODE_admin)
  → Profile picture → Your enterprises
    → Select your enterprise
      → Settings → Authentication security
```

## 3️⃣ Configure Identity Provider

### For Microsoft Entra ID (OIDC — Recommended)

```
Microsoft Entra Admin Center
  → Enterprise applications → + New application
    → Search: "GitHub Enterprise Managed User (OIDC)"
      → Create
        → Single sign-on → OIDC
          → Configure tenant ID and client credentials
```

### For Okta / PingFederate (SAML)

```
IdP Admin Console
  → Applications → Add Application
    → Search: "GitHub Enterprise Managed User"
      → Configure SAML SSO
        → Set Entity ID, ACS URL, Sign-on URL
```

## 4️⃣ Configure SCIM Provisioning

```
IdP Admin Console
  → Enterprise Application → Provisioning
    → Automatic provisioning
      → Tenant URL: from GitHub enterprise settings
      → Secret token: generate from GitHub setup user
        → Test connection → Start provisioning
```

### Map Required Attributes
- `userName` → UPN or custom attribute
- `emails` → user.mail or user.userPrincipalName
- `displayName` → user.displayName

### Assign Users and Groups
- Assign users to the Enterprise Application in your IdP
- Map IdP groups to GitHub teams
- Include **Enterprise Owner** role for at least one admin user

## 5️⃣ Migrate Repositories

### Using GitHub Enterprise Importer (GEI)

```bash
# Install GEI
gh extension install github/gh-gei

# Migrate a single repo
gh gei migrate-repo \
  --github-source-org SOURCE_ORG \
  --source-repo REPO_NAME \
  --github-target-org TARGET_ORG \
  --target-repo REPO_NAME

# Migrate an entire org
gh gei migrate-org \
  --github-source-org SOURCE_ORG \
  --github-target-org TARGET_ORG \
  --github-target-enterprise TARGET_ENTERPRISE
```

### What GEI Migrates
- Git history (all commits, branches, tags)
- Pull requests with reviews and comments
- Issues (from some sources)

### What Requires Manual Reconfiguration
- Actions secrets and variables
- Webhook configurations
- Branch protection rules and rulesets
- Environment protection rules
- OIDC trust policies
- GitHub Apps (reinstall in new enterprise)

## 6️⃣ Recreate Organization Structure

- [ ] Create organizations matching your governance model
- [ ] Set up teams with appropriate repo access
- [ ] Configure repository visibility (private/internal)
- [ ] Apply rulesets at org level
- [ ] Configure Copilot policies and access
- [ ] Set up cost centers and billing

## 7️⃣ Update Integrations

| Integration | Action Required |
|------------|----------------|
| Personal Access Tokens | Replace with GitHub Apps or fine-grained PATs from managed accounts |
| SSH keys | Generate new keys for managed user accounts |
| GitHub Apps | Reinstall in new enterprise |
| Webhooks | Recreate pointing to new repo URLs |
| OIDC trusts (AWS/Azure/GCP) | Update issuer URL if moving to GHE.com |
| CI/CD pipelines | Update repo URLs, secrets, runner configs |
| Package registries | Update registry URLs and tokens |

## 8️⃣ Cutover

1. Set a coordinated cutover date with all teams
2. Put old enterprise repos in **read-only** mode (archive repos)
3. Redirect users to new enterprise with clear instructions
4. Verify all users can authenticate via IdP
5. Run validation tests on critical workflows
6. Monitor for 2-4 weeks before decommissioning old enterprise

## ⚠️ Critical EMU Restrictions to Communicate

| Restriction | Impact | Workaround |
|------------|--------|------------|
| No public repos | Cannot host OSS in EMU | Separate public org on github.com |
| No gists | Managed users cannot create gists | Use repos for code sharing |
| No external interaction | Cannot PR/issue/star outside enterprise | Separate personal GitHub.com account |
| SCIM-only provisioning | No manual user invites | Ensure IdP admin has fast-track process |

---

## 📝 Resources

| Resource | Link |
|----------|------|
| About EMU | [docs.github.com](https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/understanding-iam-for-enterprises/about-enterprise-managed-users) |
| EMU abilities and restrictions | [docs.github.com](https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/understanding-iam-for-enterprises/abilities-and-restrictions-of-managed-user-accounts) |
| GitHub Enterprise Importer | [docs.github.com](https://docs.github.com/en/migrations/using-github-enterprise-importer) |
| Configuring SCIM for EMU | [docs.github.com](https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/provisioning-user-accounts-with-scim/configuring-scim-provisioning-for-enterprise-managed-users) |

---

*Last updated: April 2026*
