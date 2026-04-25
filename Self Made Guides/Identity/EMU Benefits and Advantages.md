# 🔐 GitHub EMU Benefits & Decision Guide

> **A comprehensive guide to understanding Enterprise Managed Users and when to choose this identity model**

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the key decision points:**

- **Choose EMU when:** Security-first, compliance requirements, centralized access control, strict tenant boundaries
- **Choose personal accounts when:** Open-source collaboration is core, developer autonomy prioritized
- **Key EMU benefit:** IdP-controlled lifecycle (SCIM onboard/offboard), no GitHub-stored passwords, hard tenant boundary
- **Key trade-off:** EMU accounts cannot interact with public repos -- dual-account approach needed for OSS

---

## ✅ Accuracy & Click-Path Notes

- Reviewed against current public GitHub and Microsoft documentation in April 2026 where public documentation is available. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
- When a path starts with `Enterprise`, begin at GitHub, click your profile photo, click `Your enterprises` or `Enterprise`, select the enterprise, then continue with the listed top tab or left-sidebar item.
- When a path starts with `Organization` or `Org`, begin at GitHub, click your profile photo, click `Your organizations`, select the organization, click `Settings`, then continue with the listed sidebar item.
- When a path starts with `Repository`, `Repo`, or a repository name, open the repository, click the `Settings` tab, then continue with the listed sidebar item.
- When a path starts with a vendor portal such as `Microsoft Entra admin center`, `Azure portal`, `Okta Admin Console`, `PingFederate`, `PingOne`, `OneLogin`, `AD FS Management`, `Visual Studio Admin Portal`, or `Azure DevOps`, sign in to that admin portal first, select the tenant, application, or project named in the step, then follow each listed blade, tab, button, and confirmation in order.
- If the expected button is missing, verify you are signed in with the role named in Prerequisites, the feature or license is enabled, and the object is owned by the selected enterprise, organization, or repository. Use page search only to locate the same page, not to skip required confirmation, test, save, or consent clicks.

---

## ✅ Prerequisites

| Requirement | Status |
|-------------|--------|
| Identity provider (Entra ID, Okta, or PingFederate) configured | ☐ |
| SAML or OIDC authentication configured in the IdP | ☐ |
| SCIM provisioning application set up in the IdP | ☐ |
| Enterprise setup user account and PAT with `scim:enterprise` scope | ☐ |

---

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **GitHub enterprise owner or setup user** | Owns the GitHub-side EMU enterprise, setup user, recovery codes, and final SSO/SCIM enablement. | GitHub → profile photo → Your enterprises → [enterprise] → Identity provider → Single sign-on configuration → configure OIDC or SAML, then Identity provider → SCIM or provisioning setup where shown. Handoff: enterprise shortcode, setup user status, recovery codes stored, SSO test, and provisioning test. |
| **Microsoft Entra, Okta, or PingFederate admin** | Creates and owns the IdP application, group assignments, SAML/OIDC claims, and SCIM lifecycle rules. | Entra: Microsoft Entra admin center → Entra ID → Enterprise apps → New application → GitHub Enterprise Managed User. Okta: Okta Admin Console → Applications → Browse App Catalog → GitHub Enterprise Managed User. PingFederate: Administrative Console → Applications → SP Connections → Use a template for this connection → GitHub EMU Connector. Handoff: SSO URLs or consent status, SCIM token destination configured, and pilot group assigned. |
| **Azure subscription Owner, if metered billing is part of rollout** | Connects or approves the Azure subscription used for GitHub metered billing. | Azure portal → Subscriptions → [subscription] → Access control (IAM) → Role assignments → confirm Owner, then support the GitHub billing connection through GitHub → profile photo → Your enterprises → [enterprise] → Billing and licensing → Payment information → Metered billing via Azure → Add Azure Subscription. Handoff: connected subscription ID. |

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

## 🧯 Known Errors & Resolutions

> This section lists the known product errors and admin-facing symptoms that commonly occur with this workflow. Exact message text can vary by product rollout, tenant policy, and provider, so use the log or settings page named in the resolution to confirm the root cause.

| Error or symptom | Likely cause | Resolution |
|------------------|--------------|------------|
| **Page, tab, or button is missing** | Wrong account context, missing admin role, unavailable plan/add-on, or feature rollout not enabled for the selected enterprise/org/repo. | Switch to the correct account and scope, confirm the prerequisite role, verify licensing or add-on activation, then refresh the page. If the control is still absent, use the direct settings URL from the relevant GitHub Docs page and confirm the feature is available for your plan. |
| **Changes appear saved but behavior does not change** | Policy inheritance, cached UI state, propagation delay, or an overlapping enterprise/org/repo policy. | Reopen the settings page, verify the effective policy at the lowest affected scope, wait for propagation where documented, and check for a stricter policy at an enterprise or organization level. |
| **403, forbidden, or resource not accessible** | The signed-in user or token can see the page but lacks the specific permission for the action. | Use an enterprise owner, organization owner, repository admin, or token with the exact scopes/permissions listed in the runbook. For SAML-protected orgs, authorize the token or SSH key for SSO before retrying. |
| **Managed user cannot sign in** | The user is not assigned to the IdP application, SCIM has not provisioned the account, or SAML/OIDC configuration is wrong. | Check IdP assignment, provisioning logs, userName/NameID mappings, and the GitHub authentication test before enforcing broadly. |
| **Managed user cannot interact with public repositories** | EMU accounts are restricted from contributing outside enterprise-owned resources. | Use the dual-presence model: managed account for enterprise work and a personal GitHub.com account for open source participation. |
| **Guest collaborator cannot see internal repositories** | Guest collaborators do not receive broad internal repository access by default. | Add the guest collaborator to the specific organization/team/repository that should grant access, or use a regular enterprise member role if broad internal access is intended. |
| **Duplicate or wrong managed username appears** | Shortcode, userName mapping, or multiple enterprise assignments created distinct managed accounts. | Verify the IdP app and SCIM mapping for the intended enterprise, then deprovision incorrect assignments through the IdP. |

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

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Standard Enterprise to EMU Migration | `Setup/Standard Enterprise to EMU Migration.md` |
| Guest Collaborators in EMU | `Identity/Guest Collaborators in EMU.md` |
| EMU Dual Presence (Enterprise + Open Source) | `Identity/EMU Dual Presence (Enterprise + Open Source).md` |
| Data Residency Decision Guide (DRUS vs Standard vs GHES) | `Setup/Data Residency Decision Guide (DRUS vs Standard vs GHES).md` |

---

## 📝 Resources

- [EMU Instructions Guide](https://emu-instructions.githubapp.com)

---

*Last updated: April 2026*
