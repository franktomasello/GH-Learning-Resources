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

## ❓ Common Questions & Troubleshooting

### Q: EMU users cannot contribute to open-source projects. Is there a workaround?
**A:** This is by design -- EMU accounts are scoped to the enterprise tenant and cannot interact with repos outside it. The recommended approach is the dual-account model: employees use their EMU account for enterprise work and a separate personal GitHub.com account for open-source contributions. See the EMU Dual Presence guide for detailed setup instructions.

---

### Q: EMU users cannot create gists. How should they share code snippets?
**A:** Gist creation is not supported for managed user accounts. As an alternative, users can create repositories within the enterprise for code sharing. For quick snippet sharing, consider using internal tools like Slack snippets, a shared internal repo, or a wiki page. If sharing externally, use the personal GitHub.com account.

---

### Q: A user was deprovisioned from the enterprise, but their commits still appear in repository history. Is this expected?
**A:** Yes, this is expected Git behavior. Commits are immutable records in Git history and remain associated with the managed account even after the account is suspended or removed. The commit author metadata (name and email) persists in the git log. The deprovisioned user's account will appear as a suspended/inactive account, but their contributions to the codebase are preserved.

---

### Q: Can we switch from EMU back to a standard enterprise with personal accounts?
**A:** This is not a simple toggle. Switching from EMU to a standard enterprise requires a full migration to a new enterprise -- you would need to create a new standard enterprise, migrate all repositories and data using GitHub Enterprise Importer, re-provision users with personal accounts, and reconfigure all policies. This is a significant undertaking and should be carefully planned. Contact your GitHub account team for guidance.

---

### Q: EMU users cannot sign up for Copilot Free or Pro. How do they get Copilot access?
**A:** Managed users cannot self-enroll in Copilot Free or Pro plans. Copilot access must be provisioned through a Copilot Business or Copilot Enterprise license assigned at the enterprise or organization level. Enterprise owners or org admins enable Copilot and assign seats to managed users.

---

### Q: We are seeing username collisions during SCIM provisioning. What causes this?
**A:** EMU usernames are generated by normalizing IdP identifiers and appending an enterprise shortcode (e.g., `jsmith_contoso`). Collisions occur when two different IdP identities normalize to the same username prefix (e.g., `j.smith` and `jsmith` both becoming `jsmith`). Plan your IdP username conventions carefully before EMU rollout, and check for potential collisions in advance by reviewing the normalization rules.

## 📝 Resources

- [EMU Instructions Guide](https://emu-instructions.githubapp.com)

---

*Last updated: December 2025*
