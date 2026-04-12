# 🔀 EMU Dual Presence Guide

> How to set up open-source contribution alongside Enterprise Managed Users governance

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the key steps:**

- **EMU account:** Provisioned automatically via IdP (Entra ID / Okta) + SCIM -- username format `username_shortcode`
- **Personal account:** User creates separately at github.com/signup with a personal email
- **Public org for OSS:** Create a non-EMU org on github.com for open-source publishing
- **Mirror internal to public:** Use GitHub Actions workflow with `MIRROR_TOKEN` secret to push approved code

---

## ✅ Prerequisites

| Requirement | Status |
|-------------|--------|
| EMU enterprise configured with IdP and SCIM provisioning | ☐ |
| Users have personal GitHub.com accounts (separate from EMU) | ☐ |
| Open-source contribution policy defined and approved | ☐ |
| Separate public organization created on github.com (if mirroring) | ☐ |

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

## ❓ Common Questions & Troubleshooting

### Q: Users are confused about which GitHub account to use for what. How do we communicate this clearly?
**A:** Publish clear onboarding documentation that states: use the EMU account (e.g., `username_shortcode`) for all enterprise/internal work, and use a separate personal GitHub.com account for open-source contributions. Include this guidance in new-hire onboarding, your internal wiki, and Slack channel topics. A simple rule: if you are accessing enterprise repos, use your managed account; if you are contributing to public open-source, use your personal account.

---

### Q: Can the enterprise track or monitor what employees do on their personal GitHub.com accounts?
**A:** No. The enterprise has no visibility into personal GitHub.com accounts. Personal accounts are completely independent of the EMU enterprise -- the enterprise cannot see repos, contributions, stars, or any activity on a personal account. These are two entirely separate identities with no technical link between them.

---

### Q: A user accidentally pushed enterprise/proprietary code to their personal GitHub account. What should we do?
**A:** Treat this as a security incident. Immediately assess what was exposed (source code, secrets, internal documentation). Revoke and rotate any credentials found in the pushed code. Use `git filter-repo` to remove the proprietary content from the personal repo's history, or delete the personal repo entirely if appropriate. Document the incident and review your organization's code publishing approval process to prevent recurrence.

---

### Q: How can we mirror approved internal code to a public repository for open-source release?
**A:** Use the Internal-to-Public Repository Mirroring approach: set up a GitHub Actions workflow in the EMU enterprise that pushes approved code from a designated release branch to a separate public organization on github.com. Apply security controls including secret scanning, push protection, and required PR reviews before code reaches the mirror branch. See the Internal-to-Public Repository Mirroring runbook for detailed setup instructions.

---

### Q: Can a user's EMU account and personal account be linked or merged?
**A:** No. EMU accounts and personal accounts are completely separate identities that cannot be linked, merged, or associated in any way. Each has its own username, profile, and access. Contributions made under one account will not appear under the other. This separation is by design to maintain the enterprise's tenant boundary and governance controls.

---

### Q: Can EMU users interact with public repositories on github.com at all?
**A:** EMU users can view public repositories on github.com, but they cannot interact with them -- they cannot create issues, open pull requests, leave comments, star, watch, or fork public repos outside the enterprise. For any public interaction, users must use a separate personal GitHub.com account.

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Guest Collaborators in EMU | `Identity/Guest Collaborators in EMU.md` |
| Internal-to-Public Repository Mirroring | `Migration/Internal-to-Public Repository Mirroring.md` |
| EMU Benefits and Advantages | `Identity/EMU Benefits and Advantages.md` |
| Responsible AI Guardrails | `Copilot/Responsible AI Guardrails.md` |

---

## 📝 Resources

| Resource | Link |
|----------|------|
| EMU restrictions | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/understanding-iam-for-enterprises/abilities-and-restrictions-of-managed-user-accounts) |
| About EMU | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/admin/concepts/identity-and-access-management/enterprise-managed-users) |
| Internal-to-Public Mirroring | See: Internal-to-Public Repository Mirroring Runbook.md in this folder |

---

*Last updated: April 2026*
