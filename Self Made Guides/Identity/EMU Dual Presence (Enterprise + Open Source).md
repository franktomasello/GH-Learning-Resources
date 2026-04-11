# 🔀 EMU Dual Presence Guide

> How to set up open-source contribution alongside Enterprise Managed Users governance

---

## 📋 Overview

EMU users CANNOT contribute to public repos outside the enterprise. Open-source participation requires a **dual-account approach**.

| Account | Purpose | Governance |
|---------|---------|-----------|
| EMU account (`jsmith_contoso`) | All internal/enterprise work | IdP-controlled, enterprise policies |
| Personal GitHub.com account (`jsmith`) | Open-source, public repos, community | User-owned, independent |

> ⚠️ **These are two completely separate identities.** The enterprise has no control over the personal account, and the personal account has no access to the EMU enterprise.

---

## 1️⃣ Set Up the EMU Account

This is handled by your IdP (Entra ID, Okta, PingFederate) via SCIM provisioning:
- User is added to the appropriate IdP group
- SCIM creates the managed account automatically
- Username format: `USERNAME_SHORTCODE`

### EMU Account Restrictions
- Cannot create public repos or gists
- Cannot interact with repos outside the enterprise (no PRs, issues, stars, forks)
- Cannot contribute to open-source projects
- Fully governed by enterprise policies

---

## 2️⃣ Set Up the Personal Account

Users create a separate, free GitHub.com account independently:
1. Go to github.com/signup
2. Use a personal email address (NOT the work/state email)
3. Choose a personal username (e.g., `jsmith`, not `jsmith_contoso`)

### Personal Account Capabilities
- Full access to public repos and open-source projects
- Can create gists, star repos, fork projects
- Not controlled by the enterprise
- User owns this account

> 💡 **Tip:** If a user already registered their work email with a personal GitHub.com account, they should change that account's primary email to a personal address to avoid confusion with the EMU account.

---

## 3️⃣ Establish the Governance Bridge

### Open-Source Contribution Policy
Define clear rules for what code can flow from internal to public:
- What types of code can be open-sourced?
- Who approves the release?
- What review process is required?
- Are there IP or licensing constraints?

### CLA/DCO (Contributor License Agreements)
For inbound contributions to your public repos:
- **CLA:** Contributors sign a legal agreement before contributing
- **DCO (Developer Certificate of Origin):** Contributors certify they have the right to submit code
- Configure via GitHub Apps (e.g., CLA Assistant) on the public org

### Automated Mirroring (Optional)
Mirror approved internal code to public repos:
- Use GitHub Actions in the EMU enterprise to push to a public org
- Add secret scanning and review gates before any push to the public mirror
- See the **Internal-to-Public Repository Mirroring Runbook** for details

---

## 4️⃣ Organizational Structure

### Recommended Setup

```
EMU Enterprise (SUBDOMAIN.ghe.com or github.com)
  ├── Org: internal-development (private/internal repos)
  ├── Org: shared-platform (internal tools, libraries)
  └── Org: security (security policies, scanning)

Separate on github.com (NOT in the EMU enterprise):
  └── Org: your-agency-oss (public open-source repos)
```

### Access Patterns

| Scenario | Which Account? |
|----------|---------------|
| Internal code review, PRs, issues | EMU account |
| CI/CD pipeline automation | GitHub App (no seat cost) |
| Contributing to open-source project | Personal account |
| Maintaining agency's public repos | Personal account (in the public org) |
| Reading public documentation | Either account |

---

## 5️⃣ Communication to Users

Publish clear onboarding guidance:

> "State employees access enterprise GitHub **only** via IdP-provisioned managed accounts. For open-source contributions, use a separate personal GitHub.com account with a personal email address. Do not use personal GitHub.com accounts for state work."

---

## 📝 Resources

| Resource | Link |
|----------|------|
| EMU restrictions | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/understanding-iam-for-enterprises/abilities-and-restrictions-of-managed-user-accounts) |
| About EMU | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/admin/concepts/identity-and-access-management/enterprise-managed-users) |
| Internal-to-Public Mirroring | See: Internal-to-Public Repository Mirroring Runbook.md in this folder |

---

*Last updated: April 2026*
