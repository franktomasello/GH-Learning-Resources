# 🔐 EMU Benefits and Advantages

> **A comprehensive guide to understanding Enterprise Managed Users and when to choose this identity model**

---

## 📋 Overview

### What is GitHub EMU?

**Enterprise Managed Users (EMU)** is a GitHub Enterprise Cloud model where your company's Identity Provider (IdP) provisions and controls user accounts (lifecycle, usernames/profile data, org membership, and access). Users authenticate via the IdP (SAML or OIDC, depending on IdP).

### What is the Alternative to EMU?

**Enterprise with personal accounts:** employees use standard GitHub.com accounts (often also used publicly). You can enforce SAML SSO for enterprise resources, but the account is ultimately owned by the individual, not the company.

---

## ✅ Key Benefits of EMU

### 1) Centralized Identity Ownership + Zero-Touch Lifecycle

- **SCIM-driven onboarding/offboarding:** IdP provisions accounts, updates profile data, and (critically) removes access when identity changes
- Helps eliminate "orphaned" access and reduces manual admin work

### 2) Stronger Authentication Posture Through the IdP

- Managed users authenticate only through your IdP and do not have a GitHub-stored password or 2FA methods (reduces credential sprawl and centralizes enforcement)
- With OIDC + Entra ID, GitHub can validate access using Conditional Access Policy (CAP)

### 3) Hard Tenant Boundary for IP Protection

- Managed users cannot create public content or collaborate outside the enterprise with that identity
- On GitHub.com they can view public repos, but cannot interact (no issues/PRs/comments/reactions, and can't star/watch/fork outside repos)

### 4) Cleaner Governance and Least-Privilege Patterns at Scale

- Enterprise admins can standardize access using IdP groups (team/org membership and enterprise roles)
- EMU supports repo collaborators within the enterprise (grant repo access without full org membership) and a guest collaborator role for tighter segmentation

### 5) Corporate Ownership of Identifiers and Profile Data

- Usernames are generated from IdP identifiers; profile name/email are IdP-controlled (users can't self-edit)
- Developers' managed identities and content are only visible to other enterprise members (not publicly discoverable)

---

## ⚠️ Trade-offs and Gotchas to Plan For

### Open-Source Participation Requires a Separate Identity

- EMU accounts are tenant-scoped: devs typically need a separate personal GitHub account for public OSS work and community interaction

### Migration and Attribution Considerations

- Changing a user's email in the IdP can unlink contribution history tied to the previous email. Plan identity/email conventions carefully before rollout
- Usernames are normalized from IdP identifiers; collisions can occur if "unique" parts are stripped during normalization

### Setup/Admin Mechanics

- EMU uses a special setup user for initial configuration, and provisioning requires a PAT (classic) with `scim:enterprise` scope (and GitHub's guide specifies no expiration)
- This is separate from day-to-day managed users, who authenticate via the IdP and don't manage GitHub passwords/2FA on their own accounts

### Product/Feature Limitations

| Feature | Limitation |
|---------|------------|
| **Copilot** | Managed users can't sign up for Copilot Free/Pro; access must come from Copilot Business/Enterprise |
| **Codespaces** | Managed users can only create enterprise-owned codespaces (and on GHE.com Codespaces isn't available) |
| **Other** | Restrictions exist for external interactions and certain user-level features |

> 💡 **Note:** Validate EMU fit if your dev workflows depend on broader GitHub.com social/open features.

---

## 🤔 When Personal Accounts Are a Better Fit

- Teams that heavily contribute to open source under a single identity
- Orgs that prioritize developer autonomy and lowest-friction external collaboration

---

## 🎯 Bottom Line

| Choose EMU When... | Choose Personal Accounts When... |
|-------------------|----------------------------------|
| Security is top priority | Open-source collaboration is core |
| Compliance requirements exist | Developer autonomy is prioritized |
| Centralized access control needed | Public GitHub participation matters |
| Strict tenant boundaries required | Lowest-friction external collaboration |

---

## 📚 Reference

- [EMU Instructions Guide](https://emu-instructions.githubapp.com)

---

*Last updated: December 2025*
