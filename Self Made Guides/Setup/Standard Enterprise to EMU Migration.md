# 🔄 Standard Enterprise to EMU Migration Runbook

> Step-by-step guide to migrating from standard GitHub Enterprise Cloud (personal accounts) to Enterprise Managed Users

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Inventory:** Catalog all repos, org memberships, teams, Apps, webhooks, Actions secrets, PATs, SSH keys, OIDC trusts before starting
- **New EMU enterprise:** Work with GitHub account team → Receive `SHORTCODE_admin` → Set password + 2FA → Configure IdP (SAML/OIDC + SCIM)
- **Migrate repos:** `gh extension install github/gh-gei` → `gh gei migrate-repo` or `gh gei migrate-org` for bulk migration
- **Reconfigure:** Recreate org structure, teams, rulesets, Actions secrets, webhooks, GitHub Apps, OIDC trust policies manually
- **Cutover:** Archive old enterprise repos → Redirect users → Validate workflows → Monitor 2-4 weeks → Decommission old enterprise

---

## ✅ Prerequisites

| Requirement | Status |
|-------------|--------|
| GitHub account team engaged to provision new EMU enterprise | ☐ |
| Pre-migration inventory completed (repos, teams, Apps, secrets, webhooks, OIDC trusts) | ☐ |
| IdP admin access (Entra ID, Okta, or PingFederate) for EMU configuration | ☐ |
| GEI CLI installed (`gh extension install github/gh-gei`) | ☐ |
| Cutover date coordinated with all teams | ☐ |
| Communication plan for users about dual-account model | ☐ |

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

## ❓ Common Questions & Troubleshooting

### Q: Can I migrate from standard enterprise to EMU in-place without creating a new enterprise?
**A:** No. There is no in-place conversion. EMU requires a fundamentally different identity model (IdP-provisioned accounts vs personal accounts). You must create a new EMU enterprise, configure identity and provisioning, migrate repositories using GitHub Enterprise Importer (GEI), and have users switch to their new managed accounts. Plan this as a full migration project with a coordinated cutover.

---

### Q: All our automation PATs stopped working after migration — what happened?
**A:** PATs from personal accounts in the old enterprise do not transfer to the new EMU enterprise. Managed user accounts are entirely new identities, so all existing PATs, SSH keys, and OAuth tokens are invalid. Before migration, inventory all PATs used for automation. After migration, generate new PATs from managed user accounts, or preferably migrate automation to GitHub Apps (which are more resilient and not tied to individual user accounts).

---

### Q: Commit attribution in the new enterprise shows different usernames — how do we preserve history?
**A:** GEI migrates git history faithfully (all commits, branches, tags), but commit author information is based on the git `user.email` in each commit. If the email in old commits does not match the new managed user's email, GitHub will not link those commits to the new account. The git history is preserved, but the GitHub UI may show commits as authored by an unrecognized user. There is no way to retroactively re-attribute commits to new managed accounts without rewriting git history, which is not recommended.

---

### Q: Our CI/CD pipelines use OIDC trust with AWS/Azure/GCP — do the trust policies need updating?
**A:** Yes. If your OIDC trust policies reference the GitHub token issuer URL, they must be updated. For standard github.com, the issuer is `https://token.actions.githubusercontent.com`. If the new EMU enterprise is on GHE.com (data residency), the issuer changes to `https://token.actions.SUBDOMAIN.ghe.com`. Even on github.com, the organization and repository names may change, which affects OIDC subject claims. Update all trust policies before cutover to avoid CI/CD failures.

---

### Q: Users are confused by having two GitHub accounts (old personal + new managed) — how do we manage this?
**A:** This is expected and should be communicated clearly before migration. The old personal GitHub account remains active and can still be used for open-source contributions. The new managed account (`username_SHORTCODE`) is for enterprise work only. Advise users to: (1) configure separate browser profiles or use incognito for each account, (2) update their local git configs to use the correct email for the managed account, (3) set up SSH config with separate keys for each account. Provide clear documentation distinguishing the two.

---

### Q: How long should we keep the old enterprise live after cutover?
**A:** Keep the old enterprise in read-only mode (archive all repos) for at least 4-8 weeks after cutover. This gives teams time to discover any missed migrations, verify that all integrations work in the new enterprise, and reference old configuration. Monitor access to the old enterprise — if traffic drops to near zero after 4 weeks, it is safe to decommission. Before decommissioning, do a final check that all repositories, secrets, and configuration have been migrated.

---

### Q: GEI migrated our repos, but Actions secrets, webhooks, and branch protections are missing — is that expected?
**A:** Yes. GEI migrates git history, pull requests, and some metadata, but it does NOT migrate Actions secrets, variables, environment configurations, webhooks, branch protection rules, rulesets, GitHub Apps, or OIDC trust policies. These must be manually reconfigured in the new enterprise. Build a checklist from the Pre-Migration Inventory (Step 1) and work through each item. Consider scripting the recreation of secrets and rulesets using the GitHub API.

---

### Q: Some of our repos use GitHub Packages — will the registry URLs change?
**A:** If both enterprises are on github.com, the package registry URLs change because the org names may differ. If the new enterprise is on GHE.com (data residency), the registry base URL changes entirely (e.g., from `ghcr.io` to `containers.SUBDOMAIN.ghe.com`). Update all Dockerfiles, CI/CD pipeline references, and developer workstation configurations to point to the new registry URLs. Plan to re-publish packages to the new registry if needed.

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| EMU Benefits and Advantages | `Identity/EMU Benefits and Advantages.md` |
| GitHub Enterprise Importer | `Migration/GitHub Enterprise Importer (GEI) & Actions Importer.md` |
| Guest Collaborators in EMU | `Identity/Guest Collaborators in EMU.md` |
| EMU Dual Presence (Enterprise + Open Source) | `Identity/EMU Dual Presence (Enterprise + Open Source).md` |

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
