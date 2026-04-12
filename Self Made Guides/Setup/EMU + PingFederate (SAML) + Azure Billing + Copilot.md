# 🚀 GitHub Enterprise Managed Users (EMU) Setup Guide (PingFederate SAML)

> **Complete end-to-end runbook for configuring EMU with PingFederate (SAML), Azure billing, and GitHub Copilot**

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **PingFederate:** Identity Provider → SP Connections → Create New → SAML 2.0 → Set Entity ID to `https://github.com/enterprises/YOUR_ENTERPRISE` and ACS URL with POST binding
- **GitHub:** Sign in as `SHORTCODE_admin` → Enterprise → Identity provider → Add SAML configuration → Paste PingFederate SSO URL, Entity ID, X.509 cert → Save
- **SCIM:** As `SHORTCODE_admin`, generate PAT with `scim:enterprise` scope → PingFederate SP Connection → Outbound Provisioning → Enter SCIM Base URL + Bearer Token → Test
- **Billing:** Enterprise → Billing & Licensing → Payment information → Add Azure Subscription → Accept permissions → Connect
- **Copilot:** Enterprise Settings → AI controls → Copilot → Enable for orgs → Org Settings → Copilot → Access → Assign seats

---

## 📋 Overview

This guide walks through setting up a new GitHub Enterprise Cloud (GHEC) with Enterprise Managed Users (EMU), including:
- ✅ EMU with PingFederate / PingOne (SAML SSO)
- ✅ SCIM provisioning for user lifecycle management
- ✅ Azure subscription attachment for billing
- ✅ GitHub Copilot enablement
- ✅ Organization structure guidance

**Hosting Options:** EMU can be hosted on GitHub.com or (for data residency) on a customer subdomain of GHE.com. Setup is similar, but SAML/SCIM URLs differ.

---

## 0️⃣ Prerequisites Checklist

### Required Items

Before beginning, ensure you have:

| Requirement | Status |
|-------------|--------|
| EMU Enterprise Created — A new enterprise with EMU enabled | ☐ |
| Setup User — SHORTCODE_admin created by GitHub, with password set and 2FA enabled | ☐ |
| PingFederate / PingOne Admin Access — Ability to create SP Connections and configure SAML SSO + SCIM | ☐ |
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

## 2️⃣ Create the PingFederate SP Connection

### Option A — PingFederate (Self-Managed)

#### Navigation

```
PingFederate Admin Console
  → Identity Provider
    → SP Connections
      → Create New
        → Connection Type: Browser SSO Profiles — SAML 2.0
          → Connection Options: Browser SSO
            → Continue through wizard
```

#### Configure the SP Connection

1. **Partner's Entity ID:** Enter the GitHub SAML Entity ID (see SAML Values table below)
2. **Connection Name:** "GitHub Enterprise Managed User" (or similar descriptive name)
3. **Base URL:** Enter the GitHub SAML Sign-on URL (see SAML Values table below)

#### Browser SSO — SAML Configuration

```
PingFederate Admin Console
  → Identity Provider
    → SP Connections
      → GitHub Enterprise Managed User
        → Browser SSO
          → Configure Browser SSO
            → SAML Profiles: Select "SP-Initiated SSO" (and optionally "IdP-Initiated SSO")
              → Assertion Lifetime: Configure as needed (default is typically fine)
                → Assertion Creation
                  → Configure attribute contract and mapping
```

**Assertion Attribute Contract:**

Map the following attributes in the assertion:

| SAML Attribute | Source Value |
|----------------|-------------|
| `SAML_SUBJECT` (NameID) | User's unique identifier (e.g., email or username) |
| `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name` | User's display name |
| `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress` | User's email address |

> 💡 **Note:** Set the **NameID Format** to `urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified` or `urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress` based on your organization's requirements.

#### Configure Protocol Settings

```
PingFederate Admin Console
  → SP Connection
    → Browser SSO
      → Protocol Settings
        → Assertion Consumer Service URL
          → Binding: POST
          → Endpoint URL: (ACS URL from SAML Values table below)
```

### Option B — PingOne (Cloud)

#### Navigation

```
PingOne Admin Console
  → Connections
    → Applications
      → + (Add Application)
        → Application Name: "GitHub Enterprise Managed User"
          → Application Type: SAML Application
            → Configure
```

#### Search the Application Catalog

```
PingOne Admin Console
  → Connections
    → Applications
      → + (Add Application)
        → Search Catalog: "GitHub Enterprise Managed User"
          → Select matching integration
            → Continue
              → Configure SAML settings
```

> 💡 **Note:** If the catalog integration is not available, configure a custom SAML application using the SAML values provided below.

### SAML Configuration Values

Enter your enterprise **slug** (e.g., "octocorp" if your enterprise URL is `github.com/enterprises/octocorp` or `octocorp.ghe.com`).

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

### Download Required Items (PingFederate IdP Values)

From your PingFederate or PingOne configuration, gather the three SAML values needed for GitHub:

1. **IdP SSO Service URL** (Single Sign-On Service URL)
2. **Entity ID** (IdP Issuer / Entity ID)
3. **Signing Certificate** (X.509 certificate)

#### PingFederate (Self-Managed) Navigation

```
PingFederate Admin Console
  → Server Configuration
    → Server Settings
      → My Base URL — Note this (used to construct the SSO Service URL)
  → Security
    → Signing & Decryption Keys & Certificates
      → Export the signing certificate (X.509 / PEM format)
```

> 💡 **Tip:** The IdP SSO Service URL typically follows the pattern: `https://<PING_HOST>/idp/SSO.saml2`

> 💡 **Tip:** The IdP Entity ID can be found under **Server Configuration → Server Settings → Federation Info → Entity ID**.

#### PingOne (Cloud) Navigation

```
PingOne Admin Console
  → Connections
    → Applications
      → GitHub Enterprise Managed User
        → Configuration (tab)
          → Expand "Connection Details"
            → Copy: Single Sign-On Service URL, Issuer ID
          → Download Signing Certificate
```

### Assign Users (for SSO Testing)

Assign at least one user (or group) to the SP Connection so you can validate SSO.

#### PingFederate (Self-Managed)

User assignment in PingFederate is handled via your authentication policy and user datastore configuration. Ensure the users who need access are in the directory (LDAP/AD) connected to the SP Connection's authentication source.

#### PingOne (Cloud)

```
PingOne Admin Console
  → Connections
    → Applications
      → GitHub Enterprise Managed User
        → Access (tab)
          → + Add Group
            → (Select group)
              → Save
```

> 💡 **Note:** In PingOne, application access is controlled by group membership. Assign the appropriate group(s) containing users who need GitHub access.

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
2. Enter the following values from PingFederate / PingOne:
   - **Sign on URL:** Paste the IdP SSO Service URL from PingFederate
   - **Issuer:** Paste the Entity ID (IdP Issuer) from PingFederate
   - **Public Certificate:** Paste the contents of the PingFederate Signing Certificate (X.509)
   - **Signature Method:** Select from dropdown (typically RSA-SHA256)
   - **Digest Method:** Select from dropdown (typically SHA-256)
3. Click **Test SAML configuration** to validate the setup
4. Click **Save SAML settings**
5. **Immediately download and securely store your enterprise SSO recovery codes**

> 🔐 **Critical:** Recovery codes are essential for break-glass scenarios if your IdP becomes unavailable.

---

## 4️⃣ Configure SCIM Provisioning

SCIM handles user creation and deactivation in EMU. This requires creating a token in GitHub and configuring Outbound Provisioning in PingFederate.

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
- **Note:** "PingFederate SCIM Provisioning" (or similar descriptive name)
- **Expiration:** No expiration
- **Scope:** Select `scim:enterprise` only
- Click **Generate token**

> 🔑 **Important:** Copy the token immediately after generation. You won't be able to see it again.

### 4B — Configure SCIM in PingFederate (Self-Managed)

PingFederate uses **Outbound Provisioning** to push user data to GitHub via SCIM.

#### Navigation Path

```
PingFederate Admin Console
  → Identity Provider
    → SP Connections
      → GitHub Enterprise Managed User
        → Outbound Provisioning
          → Configure
```

#### Configuration

1. **Provisioning Target:** Select or configure a SCIM provisioning target
2. Under **SCIM Connection Settings**, enter:
   - **SCIM Base URL (Tenant URL):**
     - For GitHub.com: `https://api.github.com/scim/v2/enterprises/{ENTERPRISE_SLUG}`
     - For GHE.com: `https://api.{SUBDOMAIN}.ghe.com/scim/v2/enterprises/{SUBDOMAIN}`
   - **Authentication:** Bearer Token
   - **Secret Token:** Paste the PAT created in step 4A
3. Click **Test Connection** to verify
4. Click **Save**

#### Enable Provisioning Actions

```
PingFederate Admin Console
  → Identity Provider
    → SP Connections
      → GitHub Enterprise Managed User
        → Outbound Provisioning
          → Target Settings
            → Enable (check):
              - Create Users
              - Update Users
              - Deactivate/Delete Users
            → Save
```

### 4B (Alt) — Configure SCIM in PingOne (Cloud)

```
PingOne Admin Console
  → Connections
    → Applications
      → GitHub Enterprise Managed User
        → Provisioning (tab)
          → Enable Provisioning (toggle on)
            → Configure:
              → SCIM Base URL:
                → For GitHub.com: https://api.github.com/scim/v2/enterprises/{ENTERPRISE_SLUG}
                → For GHE.com: https://api.{SUBDOMAIN}.ghe.com/scim/v2/enterprises/{SUBDOMAIN}
              → Authentication Method: Bearer Token
              → Bearer Token: (paste PAT from step 4A)
            → Test Connection
            → Save
```

**Enable Provisioning Actions in PingOne:**

```
PingOne Admin Console
  → Connections
    → Applications
      → GitHub Enterprise Managed User
        → Provisioning (tab)
          → Rules
            → Enable:
              - Create Users
              - Update Users
              - Deactivate Users
            → Save
```

### 4C — Configure Attribute Mappings

Map SCIM attributes from your PingFederate/PingOne user store to GitHub's SCIM schema.

#### Required SCIM Attribute Mappings

| GitHub SCIM Attribute | PingFederate Source Attribute | Description |
|-----------------------|------------------------------|-------------|
| `userName` | User's unique identifier (e.g., `sAMAccountName` or `uid`) | Must be unique across the enterprise |
| `name.givenName` | `givenName` / `firstName` | User's first name |
| `name.familyName` | `sn` / `lastName` | User's last name |
| `emails[type eq "work"].value` | `mail` / `email` | User's email address |
| `displayName` | `displayName` / `cn` | User's full display name |
| `externalId` | Unique directory ID (e.g., `objectGUID` or `entryUUID`) | Persistent unique identifier from the IdP |

#### PingFederate (Self-Managed) Attribute Mapping Navigation

```
PingFederate Admin Console
  → Identity Provider
    → SP Connections
      → GitHub Enterprise Managed User
        → Outbound Provisioning
          → Attribute Mapping
            → Map source attributes to SCIM target attributes
              → Save
```

#### PingOne (Cloud) Attribute Mapping Navigation

```
PingOne Admin Console
  → Connections
    → Applications
      → GitHub Enterprise Managed User
        → Provisioning (tab)
          → Attribute Mappings
            → Edit
              → Map PingOne user attributes to GitHub SCIM attributes
                → Save
```

> 💡 **Note:** Review default mappings carefully. PingOne may provide pre-configured mappings for the GitHub EMU integration; PingFederate self-managed deployments typically require manual mapping.

### 4D — Assign Users/Groups for Provisioning

1. Ensure users and/or groups are assigned to the GitHub SP Connection
2. PingFederate / PingOne will SCIM-provision these members into the EMU enterprise

#### PingFederate (Self-Managed)

Outbound Provisioning in PingFederate provisions users based on the connected user datastore and any configured provisioning filters. Ensure the correct LDAP/AD groups or user base DN is selected.

```
PingFederate Admin Console
  → Identity Provider
    → SP Connections
      → GitHub Enterprise Managed User
        → Outbound Provisioning
          → Source Settings
            → Configure user source (datastore, base DN, filter)
              → Save
```

#### PingOne (Cloud)

```
PingOne Admin Console
  → Connections
    → Applications
      → GitHub Enterprise Managed User
        → Access (tab)
          → + Add Group
            → (Select groups for provisioning)
              → Save
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
- Use PingFederate / PingOne groups for consistent membership (synced via SCIM)
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
- PingFederate / PingOne admin with privileges to create/configure SP Connections
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
- ✅ EMU enterprise fully configured with PingFederate / PingOne authentication
- ✅ SCIM provisioning active for automated user lifecycle management
- ✅ Azure subscription connected for metered billing
- ✅ GitHub Copilot enabled and configured
- ✅ Initial organization structure established
- ✅ First users provisioned and able to access GitHub

---

## ❓ Common Questions & Troubleshooting

### Q: I am using PingFederate (self-managed) — what is the most common SP Connection misconfiguration?
**A:** The most frequent issue is an incorrect Partner Entity ID or ACS URL in the SP Connection. The Entity ID must be exactly `https://github.com/enterprises/YOUR_ENTERPRISE` (no trailing slash), and the ACS URL must be `https://github.com/enterprises/YOUR_ENTERPRISE/saml/consume` with binding set to POST. Also verify the Connection Name is descriptive but the actual SAML configuration values are what matter — the connection name is only for your reference.

---

### Q: My attribute contract mapping seems correct, but SAML authentication still fails — what should I check?
**A:** Verify the NameID format and the attribute fulfillment in the Assertion Creation step. GitHub expects `NameID` to be a unique, stable identifier (email or username). Common mistakes include mapping `NameID` to `displayName` instead of a unique identifier, or using an incorrect NameID format. Set the NameID format to `urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified` or `emailAddress`. Also ensure the email and name claims use the correct schema URIs as documented.

---

### Q: SCIM outbound provisioning in PingFederate is failing — how do I troubleshoot?
**A:** Check the PingFederate server logs (under `<PF_INSTALL>/log/`) for SCIM-related errors. Common causes include: (1) the SCIM Base URL is wrong — verify it matches `https://api.github.com/scim/v2/enterprises/YOUR_ENTERPRISE`, (2) the Bearer Token (PAT) is invalid or expired, (3) attribute mappings do not match GitHub's SCIM schema (e.g., `userName`, `emails`, `displayName` are required). Test the connection using the PingFederate Admin Console before enabling provisioning.

---

### Q: My PingFederate signing certificate is expiring — how do I renew it without downtime?
**A:** Generate a new signing certificate in PingFederate (Security > Signing & Decryption Keys & Certificates). Export the new certificate in X.509/PEM format. Before activating it in PingFederate, update the certificate in GitHub enterprise settings (Enterprise > Settings > Authentication security > SAML > Public certificate). Once GitHub has the new cert, activate it as the primary signing certificate in PingFederate. This avoids a window where the certs are mismatched.

---

### Q: What is the difference between PingFederate and PingOne, and which should I use?
**A:** PingFederate is a self-managed, on-premises (or customer-hosted) federation server. PingOne is Ping Identity's cloud-hosted identity platform. Both support SAML and SCIM with GitHub EMU. Use PingFederate if your organization already runs it on-premises and manages its own infrastructure. Use PingOne if you prefer a SaaS-based IdP. The configuration steps differ — PingFederate uses SP Connections and Outbound Provisioning, while PingOne uses Applications with built-in provisioning tabs.

---

### Q: I configured everything in PingOne but the GitHub catalog integration was not available — what do I do?
**A:** If the "GitHub Enterprise Managed User" catalog integration is not available in PingOne, create a custom SAML application instead. Manually enter the Entity ID, ACS URL, and Sign-on URL from the SAML Values table in this guide. For SCIM, configure a custom outbound provisioning channel using GitHub's SCIM Base URL and a Bearer Token. The functionality is identical — the catalog integration simply pre-fills these values.

---

### Q: Token signing in PingFederate uses SHA-1 by default — does GitHub require SHA-256?
**A:** GitHub supports both SHA-1 and SHA-256 for SAML signature and digest methods, but SHA-256 is strongly recommended and is the modern default. In PingFederate, verify the signing algorithm under the SP Connection's Protocol Settings > Signature Policy. Select RSA-SHA256 for the Signature Algorithm and SHA-256 for the Digest Algorithm. When configuring SAML in GitHub, select the matching Signature Method and Digest Method from the dropdowns.

---

### Q: Users are provisioned via SCIM but cannot sign in — what is likely the issue?
**A:** This usually means SCIM provisioning succeeded but the user is not assigned to the SP Connection for SSO. In PingFederate, user assignment for SSO is controlled by your authentication policy and the user datastore connected to the SP Connection. Verify the user exists in the LDAP/AD directory connected to PingFederate and that no access control policy is blocking their authentication. In PingOne, verify the user's group is assigned to the application under the Access tab.

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

## 📚 Resources

- [Configuring SAML SSO for Enterprise Managed Users](https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/configuring-authentication-for-enterprise-managed-users/configuring-saml-single-sign-on-for-enterprise-managed-users)
- [Configuring SCIM Provisioning for Enterprise Managed Users](https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/provisioning-user-accounts-with-scim/configuring-scim-provisioning-for-enterprise-managed-users)
- [PingIdentity GitHub EMU Integration Guide](https://docs.pingidentity.com/integrations/github/github-emu-integration-guide/)

---

*Last updated: April 2026*
