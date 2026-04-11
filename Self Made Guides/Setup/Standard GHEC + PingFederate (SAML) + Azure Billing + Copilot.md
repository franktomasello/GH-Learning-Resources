# 🚀 Guide to Setup GHEC (Standard / Non-EMU) + PingFederate (SAML) + Attach Azure Subscription + Turn on Copilot

> **Complete end-to-end runbook for configuring Standard (non-EMU) GHEC with PingFederate/PingOne (SAML), SCIM org provisioning, Azure billing, and GitHub Copilot**

## 📋 Overview

This guide walks through setting up **Standard (non-EMU) GitHub Enterprise Cloud (GHEC)**, including:

- **PingFederate / PingOne (SAML SSO)** for authentication
- **Enforce SAML SSO** for the organization (and enterprise, if applicable)
- **SCIM provisioning** (PingFederate → GitHub org membership lifecycle)
- **Azure subscription** attachment for metered billing
- **GitHub Copilot** enablement + policy controls
- **Critical post-enablement** items (PAT/SSH SSO authorization)

### Standard (non-EMU) means:

- Users keep their regular GitHub.com accounts
- SAML SSO is enforced at the org (and optionally at the enterprise, if you have one)
- SCIM manages organization membership via PingFederate assignments (not "managed user accounts"—that is EMU)

## 0️⃣ Prerequisites Checklist

| Requirement | Owner / Role | Notes |
| --- | --- | --- |
| GitHub Enterprise Cloud (Standard / non-EMU) org | Org Owner | You must be able to access Org Settings → Authentication security |
| (If applicable) GitHub Enterprise account | Enterprise Owner | Needed only if you will configure Enterprise SAML and/or Enterprise Copilot policies |
| PingFederate or PingOne Admin access | Ping Admin | Must be able to create SP Connections (PingFederate) or Applications (PingOne) |
| Pilot users/groups in PingFederate/PingOne | IAM team | Use a small pilot cohort first to reduce risk |
| Dedicated GitHub "SCIM setup user" | Org Owner | GitHub recommends a dedicated user to own the SCIM token for provisioning |
| Azure subscription + ability to consent | Azure admin | Needed to connect metered billing via Azure. If subscription is in a different tenant, you may need to specify a different tenant ID during connection |
| Copilot plan decision | Enterprise/Org owner | Copilot Business vs Copilot Enterprise |

## 1️⃣ Create & Secure the "SCIM Setup User" (Standard non-EMU)

⚠️ **This user is required because SCIM provisioning acts on behalf of a specific GitHub user via a PAT (Personal Access Token).** If that user loses access or leaves the org, SCIM can stop working—so you want a stable, dedicated identity.

### Process

1. Create a dedicated GitHub.com account (example: `gh-ping-scim@yourdomain.com`)
2. **Secure the account:**
    - Enable 2FA
    - Ensure recovery methods are stored per your approved process (vault / break-glass)
3. **Add the setup user to your GitHub organization as an Owner:**
    - GitHub → your Organization → Settings → People → Invite member
    - After accepted → set role to Owner
4. **Create a PAT (classic) with `admin:org` scope** for this user — you will need it for SCIM configuration in PingFederate

### Important Notes

- 📌 This setup user will consume a GitHub license
- 📌 Treat it as a system account: minimal use outside of IAM configuration

## 2️⃣ Create the PingFederate SP Connection (or PingOne Application)

### Option A: PingFederate (On-Prem / Standalone)

**Navigation (PingFederate Admin Console)**

```
PingFederate Admin Console
  → Identity Provider
    → SP Connections
      → Create New
```

**Configuration Steps**

1. **Connection Type:** Browser SSO Profiles → SAML 2.0
2. **Connection Name:** GitHub Enterprise Cloud - YOUR_ORG (or a descriptive label)
3. **Partner's Entity ID (SP Entity ID):**
    ```
    https://github.com/orgs/YOUR_ORG
    ```
4. **Assertion Consumer Service (ACS) URL:**
    ```
    https://github.com/orgs/YOUR_ORG/saml/consume
    ```
    - Binding: POST
5. **Browser SSO → SAML Profiles:** Enable SP-Initiated SSO
6. **Assertion Lifetime:** Configure per your organization's session policy
7. **Attribute Contract / Attribute Mapping:**
    - Map the `NameID` (Subject) to a unique, stable user identifier (e.g., email address or UPN)
    - Ensure the attribute source is configured correctly for your user directory (LDAP, Active Directory, etc.)
8. **Signing Configuration:**
    - Select or create a signing certificate
    - Sign the SAML Assertion (required by GitHub)
9. **Activate the connection**

### Option B: PingOne (Cloud)

**Navigation (PingOne Admin Console)**

```
PingOne Admin Console
  → Connections
    → Applications
      → + Application
        → Search: "GitHub"
          → Select the GitHub SAML application template
```

**Configuration Steps**

1. **Select the GitHub application template** from the PingOne catalog
    - ⚠️ Do NOT use a GitHub EMU template (that is for EMU)
2. **Fill in required SAML configuration fields:**
    - **ACS URL:** `https://github.com/orgs/YOUR_ORG/saml/consume`
    - **Entity ID:** `https://github.com/orgs/YOUR_ORG`
    - **Sign-on URL:** `https://github.com/orgs/YOUR_ORG/sso`
3. **Attribute Mappings:**
    - Map `saml_subject` (NameID) to a unique user identifier (e.g., email)
4. **Enable the application**

### Gather Required IdP Values (Both PingFederate and PingOne)

From your PingFederate SP Connection or PingOne Application configuration, capture:

1. **IdP SSO Service URL** (Single Sign-On Service URL)
2. **IdP Entity ID** (Issuer)
3. **X.509 Signing Certificate** (public certificate, Base64-encoded)

💡 **PingFederate:** These values are available under SP Connection → Protocol Settings → Export Metadata, or from the server's federation metadata endpoint (e.g., `https://your-pingfed-server:9031/pf/federation_metadata.ping?PartnerSpId=https://github.com/orgs/YOUR_ORG`)

💡 **PingOne:** These values are available under the Application → Configuration tab, or by downloading the IdP metadata XML.

## 3️⃣ Enable SAML SSO in GitHub

You will configure **Organization SAML** (always for the target org).

If your org is under an Enterprise account, you may also configure **Enterprise SAML**.

⚠️ **Warning:** Enabling SAML impacts how members authenticate. Ensure you have recovery codes stored for break-glass access.

### 3A — (Conditionally Required) Configure Enterprise SAML

**When this step is required:**

- If you are a GitHub Enterprise (enterprise account) customer and intend to require SAML at the enterprise level, complete this before org enforcement. **Enterprise SAML completely replaces org-level SAML configuration and enforces SAML SSO for every organization in the enterprise.**

**Navigation Path (GitHub UI)**

```
GitHub (top-right profile picture)
  → Enterprises
    → [Select enterprise]
      → Settings
        → Authentication security
          → SAML single sign-on
```

**Configuration Steps**

1. In Enterprise settings → Authentication security → SAML single sign-on
2. Enable / configure SAML per GitHub's enterprise SAML documentation (PingFederate values differ from org app)

### 3B — Enable & Test Organization SAML (Required)

**Navigation Path (GitHub UI)**

```
GitHub (top-right profile picture)
  → Organizations
    → (next to your org) Settings
      → Authentication security
        → SAML single sign-on
```

**Configuration Steps (Organization SAML)**

1. Under **SAML single sign-on**, select **Enable SAML authentication**
2. Populate fields using PingFederate/PingOne values from Step 2:
    - **Sign on URL** (IdP SSO Service URL from PingFederate/PingOne)
    - **Issuer** (IdP Entity ID from PingFederate/PingOne) — _Note: GitHub indicates Issuer is required for some features like team synchronization_
    - **Public Certificate** (X.509 Signing Certificate from PingFederate/PingOne)
3. Click **Test SAML configuration** (or equivalent prompt) and complete the auth flow
4. Click **Save**
5. **Immediately download and secure SSO recovery codes:**
    - Organization Settings → Authentication security → Single sign-on recovery codes

🔐 **Critical:** Before enabling or immediately after enabling SAML, download and securely store your organization SSO recovery codes. These are essential for break-glass scenarios if your IdP becomes unavailable.

## 4️⃣ Enforce SAML SSO for the Organization (Required)

⚠️ **Enforcement removes org members who have not authenticated through the IdP,** and can also remove bots/service accounts that don't have external identities. **If a user rejoins the organization within three months, the user's access privileges and settings will be restored.**

### Navigation Path (GitHub UI)

```
GitHub (top-right profile picture)
  → Organizations
    → (next to your org) Settings
      → Authentication security
        → SAML single sign-on
```

### Enforcement Steps

1. **Ensure you have:**
    - Enabled and tested SAML SSO
    - Authenticated via IdP at least once
2. Under **SAML single sign-on**, select:
    - **Require SAML SSO authentication for all members**
3. If GitHub displays members who have not authenticated, review the list
4. Confirm the warning and click:
    - **Remove members and require SAML single sign-on**
5. **Verify recovery codes are stored securely**

## 5️⃣ Configure SCIM Provisioning (PingFederate → GitHub Organization)

💡 **In Standard non-EMU, SCIM provisioning manages organization membership lifecycle.** PingFederate/PingOne communicates with GitHub's SCIM API using a PAT from the dedicated setup user.

### 5A — Generate SCIM PAT from the Setup User (Required)

1. Sign into GitHub as the SCIM setup user
2. Create an active org SAML session by visiting:

```
https://github.com/orgs/ORGANIZATION-NAME/sso
```
3. Complete the SSO authentication prompt successfully
4. Generate a **Personal Access Token (classic)** with the `admin:org` scope:

```
GitHub (top-right profile picture)
  → Settings
    → Developer settings
      → Personal access tokens
        → Tokens (classic)
          → Generate new token
            → Scope: admin:org
```

5. Copy and securely store the token — you will need it for PingFederate provisioning configuration

### 5B — Configure Outbound Provisioning in PingFederate (Required)

**Option A: PingFederate (On-Prem)**

1. Navigate to your GitHub SP Connection in PingFederate
2. Go to **Connection Type** and enable **Outbound Provisioning**
3. Configure the SCIM outbound provisioning channel:
    - **SCIM Base URL / Tenant URL:** Obtain from GitHub org settings:
      ```
      GitHub → Organization → Settings → Authentication security → SCIM
      ```
    - **Authentication:** Bearer Token — use the PAT generated in Step 5A
4. Configure attribute mappings per GitHub's SCIM schema requirements
5. Activate the provisioning channel

**Option B: PingOne (Cloud)**

1. Navigate to your GitHub application in PingOne
2. Go to the **Provisioning** tab
3. Enable provisioning and configure:
    - **SCIM Base URL / Tenant URL:** Obtain from GitHub org settings
    - **Authentication Token:** Use the PAT generated in Step 5A
4. Configure attribute mappings
5. Save and enable provisioning

📌 **Note:** The SCIM tenant URL is found in your GitHub Organization → Settings → Authentication security, under the SCIM section (visible after SAML is enabled and enforced).

### 5C — Assign Users/Groups (Required)

**PingFederate:**

```
PingFederate Admin Console
  → Identity Provider
    → SP Connections
      → [Your GitHub SP Connection]
        → Outbound Provisioning
          → Channel → Source (configure user/group filter)
```

**PingOne:**

```
PingOne Admin Console
  → Directory
    → Groups
      → [Assign group to the GitHub application]

— or —

PingOne Admin Console
  → Connections
    → Applications
      → [GitHub Application]
        → Access
          → Assign users/groups
```

**Steps**

1. Assign:
    - Pilot group(s) first
2. **Validate lifecycle:**
    - Add assignment → user becomes org member
    - Remove assignment → user is removed from org (per your provisioning settings)

## 6️⃣ Attach Azure Subscription for Metered Billing

**Required if you are billing via Azure**

### Prerequisites

- ✓ You are an Owner of the GitHub org or enterprise account you are connecting
- ✓ You know the Azure subscription ID
- ✓ You are logged into Azure with a user who can provide tenant-wide admin consent (or you have an admin-consent workflow)
- ✓ **If the Azure subscription is in a different tenant than your default, you may need to specify a different tenant ID during connection**

### Configuration Steps (GitHub)

**Navigation Path:**

```
GitHub
  → Your org or enterprise settings entry point
    → Org list: https://github.com/settings/organizations
    → Enterprise list: https://github.com/settings/enterprises
      → Open the target org/enterprise
        → Billing & Licensing
          → Payment information
            → Metered billing via Azure
              → Add Azure Subscription
```

**Process:**

1. Navigate to your org or enterprise settings entry point:
    - Org list: `https://github.com/settings/organizations`
    - Enterprise list: `https://github.com/settings/enterprises`
2. Open the target org/enterprise
3. Click **Billing & Licensing:**
    - For orgs: in the left sidebar under settings (label can vary slightly by UI)
    - For enterprises: a top-level tab
4. Click **Payment information**
5. Scroll to the bottom. To the right of **Metered billing via Azure**, click:
    - **Add Azure Subscription**
6. Sign in to Microsoft when prompted
7. On **Permissions requested**, click **Accept** (or follow admin approval flow if required)
8. Under **Select a subscription**, pick the Azure Subscription ID
9. Click **Connect**

💡 **Admin Consent:** If you don't see a "Permissions requested" prompt and instead see a message about needing admin approval, you may need to configure an admin consent workflow in Azure or work with your Azure AD global administrator.

## 7️⃣ Enable GitHub Copilot (Enterprise + Organization)

### 7A — (Conditionally Required) Enable Copilot at the Enterprise level via Payment Verification

**When this step is required:**

- If you manage Copilot through an enterprise account, GitHub's setup flow enables Copilot via enterprise payment verification.

**Navigation Path (GitHub UI)**

```
GitHub (top-right profile picture)
  → Enterprise (or Enterprises → select enterprise)
    → Settings
      → Getting Started
        → Verify your payment method
```

Complete **Verify your payment method** to enable Copilot in the enterprise.

### 7B — Configure Copilot Enterprise Policies

**Required if you manage policies at enterprise level**

**Navigation Path (GitHub UI)**

```
GitHub (top-right profile picture)
  → Enterprise (or Enterprises → select enterprise)
    → AI controls
      → Copilot
```

From here you can set enterprise-wide Copilot policy enforcement.

**Policy Options:**

- For each policy, select:
    - **Enabled/Allowed** — Feature is on for all organizations
    - **Disabled/Blocked** — Feature is off for all organizations
    - **No policy** — Delegate decision to organization owners

### 7C — Configure Copilot Organization Policies (Required)

Even with enterprise governance, you should confirm org-level visibility and configuration.

**Navigation Path (GitHub UI)**

```
GitHub (top-right profile picture)
  → Organizations
    → (next to your org) Settings
      → (sidebar) Code, planning, and automation
        → Copilot
          → Policies / Models
```

Set:

- Feature availability policies
- Model availability policies (if applicable)

### 7D — Assign Copilot licenses (Required)

**Copilot Business:** You can assign licenses to individual users who don't consume a GitHub Enterprise license.

**Copilot Enterprise:** You typically enable for entire organizations, and all members consume a GitHub Enterprise license.

**Navigation Path:**

```
Organization Settings
  → Copilot
    → Access
      → Add members or enable for all
```

Organization owners can assign Copilot seats to individual members or teams.

## 8️⃣ Critical Post-Enablement: SSO Authorization for Credentials (Required)

When SAML is enabled/enforced, users often must authorize credentials (depending on token type and whether they have a linked external identity).

### 8A — Authorize SSH Keys for SSO

**Required for SSH usage in SSO orgs**

**Navigation**

```
GitHub (top-right profile picture)
  → Settings
    → SSH and GPG keys
      → (next to the key) Configure SSO
        → Authorize (for the org)
```

### 8B — Authorize Personal Access Tokens

**Required for PAT classic in SSO orgs**

**Navigation**

```
GitHub (top-right profile picture)
  → Settings
    → Developer settings
      → Personal access tokens
        → (next to the token) Configure SSO
          → Authorize (for the org)
```

**Token nuance:**

- 📌 GitHub states **PAT classic** requires post-creation SSO authorization
- 📌 GitHub states **fine-grained PATs** are authorized during creation, before org access is granted

## ✅ Pre-Flight / Validation Checklist

### Before Starting

- Organization slug: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
- SCIM setup user credentials stored securely: ☐
- SCIM setup user 2FA enabled with recovery codes saved: ☐
- PingFederate/PingOne admin with privileges to create SP Connections or Applications: ☐
- Azure Subscription ID (for billing): \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
- Azure admin who can grant tenant-wide consent: ☐

### SAML

- [ ] Org SAML enabled and tested
- [ ] Recovery codes downloaded and stored
- [ ] Enforcement applied successfully
- [ ] Pilot users can access org resources via SSO

### SCIM

- [ ] Setup user is an org owner
- [ ] Setup user has an active org SAML session (`/orgs/ORG/sso`)
- [ ] PAT with `admin:org` scope created and stored securely
- [ ] PingFederate/PingOne outbound provisioning configured with SCIM tenant URL and PAT
- [ ] Assign/unassign test behaves as expected

### Azure Billing

- [ ] Azure subscription successfully connected under Payment information
- [ ] "Metered billing via Azure" shows the correct subscription ID

### Copilot

- [ ] Enterprise payment verification completed (if enterprise-managed)
- [ ] Enterprise Copilot policies set (if applicable)
- [ ] Org Copilot policies set
- [ ] Licenses assigned to pilot cohort
- [ ] Pilot users can use Copilot in IDE / GitHub.com as expected

## 🎯 Success Criteria

After completing this guide, you should have:

- ✅ Standard GHEC organization fully configured with PingFederate/PingOne SAML authentication
- ✅ SAML SSO enforced for the organization
- ✅ SCIM provisioning active for automated org membership lifecycle management
- ✅ Azure subscription connected for metered billing (if applicable)
- ✅ GitHub Copilot enabled and configured
- ✅ Users have authorized SSH keys and PATs for SSO access
- ✅ Pilot users provisioned and able to access GitHub via SSO

* * *

_Last updated: April 2026_

Sources
- [About identity and access management with SAML single sign-on - GitHub Docs](https://docs.github.com/en/organizations/managing-saml-single-sign-on-for-your-organization/about-identity-and-access-management-with-saml-single-sign-on)
- [PingIdentity GitHub Integration Documentation](https://docs.pingidentity.com/integrations/github/)
