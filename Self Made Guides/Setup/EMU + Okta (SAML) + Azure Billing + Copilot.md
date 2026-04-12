# 🚀 GitHub Enterprise Managed Users (EMU) Setup Guide (Okta SAML)

> **Complete end-to-end runbook for configuring EMU with Okta (SAML), Azure billing, and GitHub Copilot**

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Okta:** Applications → Browse App Catalog → "GitHub Enterprise Managed User" → Add Integration → Sign On tab → Set Enterprise Name to your slug
- **GitHub:** Sign in as `SHORTCODE_admin` → Enterprise → Identity provider → Add SAML configuration → Paste Okta Sign-on URL, Issuer, X.509 cert → Save
- **SCIM:** As `SHORTCODE_admin`, generate PAT with `scim:enterprise` scope → Okta App → Provisioning → Enable API Integration → Paste token → Save
- **Billing:** Enterprise → Billing & Licensing → Payment information → Add Azure Subscription → Accept permissions → Connect
- **Copilot:** Enterprise Settings → AI controls → Copilot → Enable for orgs → Org Settings → Copilot → Access → Assign seats

---

## 📋 Overview

This guide walks through setting up a new GitHub Enterprise Cloud (GHEC) with Enterprise Managed Users (EMU), including:
- ✅ EMU with Okta (SAML SSO)
- ✅ SCIM provisioning for user lifecycle management
- ✅ Azure subscription attachment for billing
- ✅ GitHub Copilot enablement
- ✅ Organization structure guidance

**Hosting Options:** EMU can be hosted on GitHub.com or (for data residency) on a customer subdomain of GHE.com. Setup is similar, but SAML/SCIM URLs and Okta app selection differ.

---

## 0️⃣ Prerequisites Checklist

### Required Items

Before beginning, ensure you have:

| Requirement | Status |
|-------------|--------|
| EMU Enterprise Created — A new enterprise with EMU enabled | ☐ |
| Setup User — SHORTCODE_admin created by GitHub, with password set and 2FA enabled | ☐ |
| Okta Admin Access — Ability to create App Integrations and configure SAML SSO + SCIM | ☐ |
| Azure Subscription — Subscription ID and someone who can grant tenant-wide admin consent | ☐ |

> 💡 **Note:** SCIM provisioning is required for EMU to manage user lifecycle and account creation.

---

## 1️⃣ Create & Configure the EMU Setup User

### Process

1. GitHub emails an invite to set the password for SHORTCODE_admin
2. In a private/incognito window:
   - Set password
   - Enable 2FA
   - Save recovery codes securely

### Important Notes

> ⚠️ **Setup User Purpose:** This account is primarily for SCIM provisioning via token and recovery scenarios. Day-to-day enterprise administration should be done with provisioned managed user accounts.

> 🚨 **Email Conflict:** If the provided email address is already associated as a primary email with an existing GitHub account, the activation link will not work. Modify the existing account's primary email first.

---

## 2️⃣ Create the Okta Application

### Navigation

```
Okta Admin Console
  → Applications
    → Applications
      → Browse App Catalog   (or "Browse App Integration Catalog")
        → Search: "GitHub Enterprise Managed User"
          → Select the correct integration
            → Add Integration
```

### Select the Correct Okta Integration

- **For GitHub.com hosted EMU:** install **GitHub Enterprise Managed User**
- **For GHE.com hosted EMU (Data Residency):** install **GitHub Enterprise Managed User - GHE.com**

### SAML Configuration

**Set Enterprise Name:**

```
Okta Admin Console
  → Applications
    → Applications
      → GitHub Enterprise Managed User
        → Sign On (tab)
          → Next to "Enterprise Name"
            → Type: [your enterprise slug, e.g., "octocorp"]
            → Save
```

> 💡 **Note:** Enter your enterprise **slug** (e.g., "octocorp" if your enterprise URL is `github.com/enterprises/octocorp` or `octocorp.ghe.com`).

### Download Required Items (Okta IdP values)

From the Okta application, gather the three SAML values:

1. **Sign on URL** (IdP Sign-On URL)
2. **Issuer**
3. **Signing certificate** (X.509)

**Navigation Path:**

```
Okta Admin Console
  → Applications
    → Applications
      → GitHub Enterprise Managed User
        → Sign On (tab)
          → Under "SAML 2.0"
            → More details (click)
              → Copy: Sign on URL, Issuer, Signing certificate
```

### SAML Values (GitHub SP values)

These values are auto-configured in the Okta app and are provided here for reference.

#### For GitHub.com Hosted EMU

| Field | Value |
|-------|-------|
| Identifier (Entity ID) | `https://github.com/enterprises/YOUR_ENTERPRISE` |
| Reply URL (ACS URL) | `https://github.com/enterprises/YOUR_ENTERPRISE/saml/consume` |
| Sign-on URL | `https://github.com/enterprises/YOUR_ENTERPRISE/sso` |

#### For GHE.com Hosted EMU (Data Residency)

| Field | Value |
|-------|-------|
| Identifier (Entity ID) | `https://SUBDOMAIN.ghe.com/enterprises/SUBDOMAIN` |
| Reply URL (ACS URL) | `https://SUBDOMAIN.ghe.com/enterprises/SUBDOMAIN/saml/consume` |
| Sign-on URL | `https://SUBDOMAIN.ghe.com/enterprises/SUBDOMAIN/sso` |

> ⚠️ **Critical:** Ensure your Identifier format matches GitHub exactly and does not include a trailing slash.

### Assign Users (for SSO testing)

Assign at least one user (or group) to the Okta application so you can validate SSO.

**Navigation Path:**

```
Okta Admin Console
  → Applications
    → Applications
      → GitHub Enterprise Managed User
        → Assignments (tab)
          → Assign
            → Assign to People (or Assign to Groups)
              → (Select user/group)
                → Assign
                  → Done
```

---

## 3️⃣ Enable SAML SSO in GitHub

> ⚠️ **Warning:** Enabling SAML impacts how members authenticate. Enterprise Managed Users does not provide a backup username/password sign-in URL; if SAML fails, use enterprise SSO recovery codes or contact GitHub Enterprise Support.

### Navigation Path

```
GitHub (as SHORTCODE_admin)
  → Profile picture (top-right)
    → Enterprise
      → Identity provider (top navigation tab)
        → Single sign-on configuration
          → Under "SAML single sign-on"
            → Add SAML configuration
```

### Configuration Steps

1. Under **"SAML single sign-on"**, click **Add SAML configuration**
2. Enter the following values from Okta:
   - **Sign on URL:** Paste the Sign on URL from Okta
   - **Issuer:** Paste the Issuer from Okta
   - **Public Certificate:** Paste the contents of the Okta Signing certificate (X.509)
   - **Signature Method:** Select from dropdown (typically RSA-SHA256)
   - **Digest Method:** Select from dropdown (typically SHA-256)
3. Click **Test SAML configuration** to validate the setup
4. Click **Save SAML settings**
5. **Immediately download and securely store your enterprise SSO recovery codes**

> 🔐 **Critical:** Recovery codes are essential for break-glass scenarios if your IdP becomes unavailable.

---

## 4️⃣ Configure SCIM Provisioning

SCIM handles user creation and deactivation in EMU. This requires creating a token in GitHub and configuring provisioning in Okta.

### 4A — Create the SCIM Token in GitHub

The token must be created as the setup user with specific requirements:

**Token Requirements:**
- ✓ Scope: `scim:enterprise`
- ✓ Expiration: No expiration
- ✓ Created by: SHORTCODE_admin

**Creation Steps:**

```
GitHub (as SHORTCODE_admin)
  → Profile picture (top-right)
    → Settings
      → Developer settings
        → Personal access tokens
          → Tokens (classic)
            → Generate new token (classic)
```

**Configuration:**
- **Note:** "Okta SCIM Provisioning" (or similar descriptive name)
- **Expiration:** No expiration
- **Scope:** Select `scim:enterprise` only
- Click **Generate token**

> 🔑 **Important:** Copy the token immediately after generation. You won't be able to see it again.

### 4B — Configure Provisioning in Okta

**Navigation Path:**

```
Okta Admin Console
  → Applications
    → Applications
      → GitHub Enterprise Managed User
        → Provisioning (tab)
          → Integration (in settings menu)
            → Edit
              → Configure API Integration
```

**Configuration:**

1. Check **Enable API integration**
2. Under **API Token**, paste the PAT created in step 4A
3. **(For GHE.com only)** Under **Base URL**, enter:
   ```
   https://api.{SUBDOMAIN}.ghe.com/scim/v2/enterprises/{SUBDOMAIN}
   ```
   > 💡 **Note:** For GitHub.com, the Base URL field is **not required** and should be left blank.
4. Click **Test API Credentials**
5. Click **Save**

**Enable Provisioning Actions:**

After saving the API integration, enable user provisioning:

```
Same Provisioning tab
  → To App (in settings menu)
    → Edit
      → Enable (check):
        -  Create Users
        -  Update User Attributes
        -  Deactivate Users
      → Save
```

> 📌 **Note:** Okta's "Import Groups" setting is **not supported** by GitHub for EMU and checking/unchecking it has **no impact** on behavior.

### 4C — Configure Attribute Mappings

Okta generally provides correct default mappings for the GitHub EMU integration. If you need to review/edit mappings:

**Navigation Path (view/edit mappings):**

```
Okta Admin Console
  → Applications
    → Applications
      → GitHub Enterprise Managed User
        → Provisioning (tab)
          → To App (in settings menu)
            → Edit
              → (Scroll) Attribute Mappings
                → Mappings / Go to Profile Editor (label varies)
```

- Review default user attribute mappings (typically sufficient for most deployments).

### 4D — Assign Users/Groups for Provisioning

1. Navigate to **Assignments** in the Okta app
2. Assign users and/or groups to the application
3. Okta will SCIM-provision these members into the EMU enterprise

**Navigation Path:**

```
Okta Admin Console
  → Applications
    → Applications
      → GitHub Enterprise Managed User
        → Assignments (tab)
          → Assign
            → Assign to People / Assign to Groups
              → (Select users/groups)
                → Assign
                  → Done
```

**Provisioning Notes:**

> 📌 **Important Constraints:**
> - To avoid exceeding GitHub's rate limit, **do not assign more than 1,000 users per hour** to the SCIM integration.

---

## 5️⃣ Attach Azure Subscription for Billing

Connect your Azure subscription so GitHub usage (Copilot, Actions, Codespaces, etc.) is billed through Azure.

### Prerequisites

- ✓ **Billing manager or owner** of the GitHub enterprise account
- ✓ Azure Subscription ID
- ✓ Azure user who can provide tenant-wide admin consent

### Configuration Steps

**Navigation Path:**

```
GitHub
  → Profile picture (top-right)
    → Enterprise
      → Billing & Licensing (top navigation tab)
        → Payment information
          → Scroll to bottom, under "Metered billing via Azure"
            → Add Azure Subscription
```

**Process:**

1. Click **Add Azure Subscription**
2. Sign in to your Microsoft account (if prompted)
3. Review the **"Permissions requested"** prompt
4. Click **Accept**
5. Under **"Select a subscription"**, choose your Azure Subscription ID
6. Check the confirmation box: "By clicking 'Connect', you are confirming..."
7. Click **Connect**

> 💡 **Admin Consent:** If you don't see a "Permissions requested" prompt and instead see a message about needing admin approval, you may need to configure an admin consent workflow in Azure or work with your Azure AD global administrator.

---

## 6️⃣ Enable GitHub Copilot

### 6A — Enable at Enterprise Level

With Azure billing connected via metered billing, Copilot usage will be billed through your Azure subscription.

**Navigation Path:**

```
GitHub
  → Profile picture (top-right)
    → Enterprise
      → Settings (top navigation)
        → AI controls (top navigation)
          → Copilot (sidebar)
```

**Access Management Configuration:**

Under **Access management**, choose:
- **Disabled** — No organizations can use Copilot
- **All organizations** — Enable for all organizations in the enterprise
- **Specific organizations** — Select which organizations can use Copilot

Select the Copilot tier (**Business** or **Enterprise**) for each enabled organization.

### 6B — Configure Copilot Policies

1. Navigate to the **Policies** tab under AI controls → Copilot
2. Configure policies for:
   - Suggestions matching public code (Allowed/Blocked)
   - Copilot in GitHub.com
   - Copilot Chat in the IDE
   - Copilot in the CLI
   - Other feature policies

**Policy Options:**

For each policy, select:
- **Enabled/Allowed** — Feature is on for all organizations
- **Disabled/Blocked** — Feature is off for all organizations
- **No policy** — Delegate decision to organization owners

### 6C — Assign Copilot Seats

After enabling at the enterprise level, organization owners assign seats:

```
GitHub
  → Profile picture (top-right)
    → Your organizations
      → Select organization
        → Settings
          → Copilot
            → Access
              → Add members or enable for all
```

Organization owners can assign Copilot seats to individual members or teams.

---

## 7️⃣ Organization Structure Guidance

Organizations are boundaries for ownership, settings, and repository visibility patterns. EMU provides centralized identity and lifecycle management across all organizations.

### Recommended Patterns

**📐 Design Principles**

"Few orgs" bias: Keep organization count low unless you have genuine separation needs:
- Compliance boundaries
- Distinct admin models
- Legal entity separation

**🏢 Common Structures**

**Org-per-business-unit:**
- Use when autonomy differs
- Separate admins and policies
- Different billing/cost centers

**Org-per-environment:**
- Only if required (e.g., regulated prod code vs everything else)
- Otherwise, teams and repositories are usually sufficient

### Inside an Organization

**Teams:**
- Use Teams for repository access control
- Use Okta groups for consistent membership (synced via SCIM)
- Start with a small set of "platform" teams:
  - Developers
  - Maintainers
  - Security Champions
  - CI Admins
- Grow organically from there

**Repository Visibility:**
- EMU provides an **Internal** visibility type
- Perfect for InnerSourcing within your enterprise
- Visible to all enterprise members, but not public

---

## 📝 Quick Reference Worksheet

### GitHub.com Hosted EMU

```
SAML Entity ID:     https://github.com/enterprises/{ENTERPRISE_SLUG}
SAML ACS URL:       https://github.com/enterprises/{ENTERPRISE_SLUG}/saml/consume
SAML SSO URL:       https://github.com/enterprises/{ENTERPRISE_SLUG}/sso
SCIM Tenant URL:    https://api.github.com/scim/v2/enterprises/{ENTERPRISE_SLUG}
```

### GHE.com (Data Residency) Hosted EMU

```
SAML Entity ID:     https://{SUBDOMAIN}.ghe.com/enterprises/{SUBDOMAIN}
SAML ACS URL:       https://{SUBDOMAIN}.ghe.com/enterprises/{SUBDOMAIN}/saml/consume
SAML SSO URL:       https://{SUBDOMAIN}.ghe.com/enterprises/{SUBDOMAIN}/sso
SCIM Tenant URL:    https://api.{SUBDOMAIN}.ghe.com/scim/v2/enterprises/{SUBDOMAIN}
```

---

## ✅ Pre-Flight Checklist

### Before Starting

- Enterprise slug/subdomain: __________________
- Setup user credentials stored securely
- Setup user 2FA enabled with recovery codes saved
- Okta admin with privileges to create/configure app integrations
- Azure Subscription ID (for billing): __________________
- Azure billing manager/owner or admin who can grant tenant-wide consent

### PAT Token Checklist

- Created as setup user (SHORTCODE_admin)
- Scope: `scim:enterprise`
- Expiration: No expiration
- Token stored securely

---

## 👥 Example Initial Group Model

| Group Name | Enterprise Role | Purpose |
|------------|-----------------|---------|
| GitHub-Enterprise-Owners | Enterprise Owner | Full admin access |
| GitHub-Developers | Member | Standard developer access |
| GitHub-Security | Member | Security champions |
| GitHub-CI-Admins | Member | CI/CD pipeline admins |

---

## 🎯 Success Criteria

After completing this guide, you should have:
- ✅ EMU enterprise fully configured with Okta authentication
- ✅ SCIM provisioning active for automated user lifecycle management
- ✅ Azure subscription connected for metered billing
- ✅ GitHub Copilot enabled and configured
- ✅ Initial organization structure established
- ✅ First users provisioned and able to access GitHub

---

## ❓ Common Questions & Troubleshooting

### Q: I installed the wrong Okta app — how do I tell if I have the EMU app vs the standard org app?
**A:** The correct catalog app for EMU is named **"GitHub Enterprise Managed User"** (for github.com) or **"GitHub Enterprise Managed User - GHE.com"** (for data residency). The standard org app is named **"GitHub Enterprise Cloud - Organization"**. If you accidentally installed the standard org app, SCIM provisioning and SAML will target org-level endpoints instead of enterprise-level. Delete the incorrect app and install the EMU-specific one from the Okta catalog.

---

### Q: SCIM provisioning errors show "conflicting user" or users are not appearing in the enterprise — what is happening?
**A:** This typically occurs when the provisioned username or email already exists on GitHub. For EMU, the username is generated as `IDP_USERNAME_SHORTCODE` (e.g., `jsmith_contoso`). If there is a collision, SCIM will fail for that user. Check the Okta provisioning logs (Applications > GitHub EMU > Provisioning > Logs) for specific error messages. Resolve conflicts by adjusting the `userName` mapping in Okta or by having the conflicting account change its username.

---

### Q: Okta Push Groups are assigned but members are not syncing to GitHub teams — why?
**A:** Push Groups in Okta push group membership to GitHub, but the groups must be mapped to GitHub teams within an organization. Verify that: (1) the group is assigned under Assignments in the Okta app, (2) the Push Groups tab shows the group is actively pushing, (3) the group name matches or is mapped to a GitHub team. Also note that Okta does not support nested groups — only direct members of the pushed group will sync.

---

### Q: SAML assertion errors appear when testing the configuration — what should I check?
**A:** Common SAML assertion errors include: signature validation failure (wrong certificate pasted into GitHub), NameID format mismatch (GitHub expects `unspecified` or `emailAddress`), and audience restriction failure (Entity ID mismatch). In Okta, go to the Sign On tab and click "View SAML setup instructions" or "More details" to verify the Sign on URL, Issuer, and certificate. Ensure the Enterprise Name field in Okta exactly matches your enterprise slug.

---

### Q: Provisioned usernames are missing the shortcode suffix — what went wrong?
**A:** If usernames appear without the `_SHORTCODE` suffix, you may have installed the standard organization app instead of the EMU app. The EMU app automatically appends the enterprise shortcode to provisioned usernames. Verify you are using the "GitHub Enterprise Managed User" catalog app in Okta, and that the Enterprise Name field is populated with your correct enterprise slug.

---

### Q: Users are caught in an SSO login loop and cannot access GitHub — how do I break the cycle?
**A:** An SSO loop typically indicates a mismatch between the SAML configuration in Okta and GitHub. Verify: (1) the Entity ID in Okta matches `https://github.com/enterprises/YOUR_ENTERPRISE` exactly (no trailing slash), (2) the ACS URL is correct, (3) the Issuer value in GitHub matches what Okta sends. Clear browser cookies and try in an incognito window. If the loop persists, use enterprise recovery codes to access the enterprise and reconfigure SAML.

---

### Q: How do I test SCIM provisioning for a single user before rolling out to everyone?
**A:** In Okta, use the "Provision on demand" feature: go to the Assignments tab, assign a single test user, then go to Provisioning > Integration > and click "Force Sync" or use the "Provision user" button on the individual assignment. Check the Okta provisioning logs and verify the user appears in the GitHub enterprise People tab within a few minutes.

---

### Q: We switched Okta tenants — do we need to reconfigure everything?
**A:** Yes. A new Okta tenant means new app integrations, new SAML certificates, and new SCIM connections. You will need to: (1) install the EMU catalog app in the new Okta tenant, (2) update the SAML certificate and Sign on URL in GitHub enterprise settings, (3) generate a new SCIM token and configure provisioning in the new tenant, and (4) reassign all users and groups. Plan this as a maintenance window.

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Enterprise Environment Scaffolding Checklist | `Setup/Enterprise Environment Scaffolding Checklist.md` |
| Guest Collaborators in EMU | `Identity/Guest Collaborators in EMU.md` |
| EMU Dual Presence (Enterprise + Open Source) | `Identity/EMU Dual Presence (Enterprise + Open Source).md` |
| EMU Benefits and Advantages | `Identity/EMU Benefits and Advantages.md` |
| Copilot Seat Assignment & Enablement | `Copilot/Seat Assignment & Enablement.md` |
| Cost Centers & Department Billing | `Billing/Cost Centers & Department Billing.md` |

---

## 📝 Resources

| # | Document | URL |
|---|----------|-----|
| 1 | Configuring SAML SSO for EMU | [docs.github.com](https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/configuring-authentication-for-enterprise-managed-users/configuring-saml-single-sign-on-for-enterprise-managed-users) |
| 2 | Configuring SCIM provisioning for EMU | [docs.github.com](https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/provisioning-user-accounts-with-scim/configuring-scim-provisioning-for-users) |
| 3 | Connecting an Azure subscription | [docs.github.com](https://docs.github.com/en/enterprise-cloud@latest/billing/managing-the-plan-for-your-github-account/connecting-an-azure-subscription) |
| 4 | Copilot policies for enterprise | [docs.github.com](https://docs.github.com/en/enterprise-cloud@latest/copilot/managing-copilot/managing-copilot-for-your-enterprise/managing-policies-and-features-for-copilot-in-your-enterprise) |
| 5 | Okta GitHub EMU integration | [saml-doc.okta.com](https://saml-doc.okta.com/SAML_Docs/How-to-Configure-SAML-2.0-for-GitHub-Enterprise-Managed-User.html) |

---

*Last updated: January 2026*
