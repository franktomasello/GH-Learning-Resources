# EMU Benefits and Advantages

## What is GitHub EMU?

**Enterprise Managed Users (EMU)** is a GitHub Enterprise Cloud model where your company's Identity Provider (IdP) provisions and controls user accounts (lifecycle, usernames/profile data, org membership, and access). Users authenticate via the IdP (SAML or OIDC, depending on IdP). ￼

---

## What is the alternative to EMU?

**Enterprise with personal accounts:** employees use standard GitHub.com accounts (often also used publicly). You can enforce SAML SSO for enterprise resources, but the account is ultimately owned by the individual, not the company. ￼

---

## Key Benefits of EMU

### 1) Centralized identity ownership + zero-touch lifecycle

- **SCIM-driven onboarding/offboarding:** IdP provisions accounts, updates profile data, and (critically) removes access when identity changes. ￼
- Helps eliminate "orphaned" access and reduces manual admin work.

### 2) Stronger authentication posture through the IdP

- Managed users authenticate only through your IdP and do not have a GitHub-stored password or 2FA methods (reduces credential sprawl and centralizes enforcement). ￼
- With OIDC + Entra ID, GitHub can validate access using Conditional Access Policy (CAP). ￼

### 3) Hard tenant boundary for IP protection

- Managed users cannot create public content or collaborate outside the enterprise with that identity. ￼
- On GitHub.com they can view public repos, but cannot interact (no issues/PRs/comments/reactions, and can't star/watch/fork outside repos). ￼

### 4) Cleaner governance and least-privilege patterns at scale

- Enterprise admins can standardize access using IdP groups (team/org membership and enterprise roles). ￼
- EMU supports repo collaborators within the enterprise (grant repo access without full org membership) and a guest collaborator role for tighter segmentation. ￼

### 5) Corporate ownership of identifiers and profile data

- Usernames are generated from IdP identifiers; profile name/email are IdP-controlled (users can't self-edit). ￼
- Developers' managed identities and content are only visible to other enterprise members (not publicly discoverable). ￼

---

## Trade-offs and "Gotchas" to Plan For

### Open-source participation requires a separate identity

- EMU accounts are tenant-scoped: devs typically need a separate personal GitHub account for public OSS work and community interaction. ￼

### Migration and attribution considerations

- Changing a user's email in the IdP can unlink contribution history tied to the previous email. Plan identity/email conventions carefully before rollout. ￼
- Usernames are normalized from IdP identifiers; collisions can occur if "unique" parts are stripped during normalization. ￼

### Setup/admin mechanics (important nuance)

- EMU uses a special setup user for initial configuration, and provisioning requires a PAT (classic) with scim:enterprise scope (and GitHub's guide specifies no expiration). ￼
- This is separate from day-to-day managed users, who authenticate via the IdP and don't manage GitHub passwords/2FA on their own accounts. ￼

### Product/feature limitations to be aware of

- **Copilot:** managed users can't sign up for Copilot Free/Pro; access must come from Copilot Business/Enterprise. ￼
- **Codespaces:** managed users can only create enterprise-owned codespaces (and on GHE.com Codespaces isn't available). ￼
- Other restrictions exist (e.g., external interactions, certain user-level features), so validate EMU fit if your dev workflows depend on broader GitHub.com social/open features. ￼

---

## When Personal Accounts Are a Better Fit

- Teams that heavily contribute to open source under a single identity
- Orgs that prioritize developer autonomy and lowest-friction external collaboration

---

## Bottom Line

**Choose EMU** when security, compliance, centralized access control, and strict tenant boundaries are top priorities.

**Choose personal accounts** when open-source collaboration and public GitHub participation are core to how your developers work. ￼

---

**Reference:** https://emu-instructions.githubapp.com
