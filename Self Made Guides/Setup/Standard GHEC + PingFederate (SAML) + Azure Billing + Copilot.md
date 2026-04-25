# 🚀 GitHub Standard GHEC + PingFederate (SAML), Azure Billing & Copilot Setup Runbook

> **Complete end-to-end runbook for configuring Standard (non-EMU) GHEC with PingFederate/PingOne (SAML), SCIM org provisioning, Azure billing, and GitHub Copilot**

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **PingFederate:** Identity Provider → SP Connections → Create New → SAML 2.0 → Entity ID `https://github.com/orgs/YOUR_ORG` → ACS URL with POST binding
- **GitHub:** Org Settings → Authentication security → SAML single sign-on → Paste PingFederate SSO URL, Entity ID, X.509 cert → Test → Save → Enforce
- **SCIM:** As setup user, generate PAT with `admin:org` scope → SSO-authorize for org → PingFederate → Outbound Provisioning → Enter SCIM URL + Bearer Token
- **Billing:** Org/Enterprise → Billing and licensing → Payment information → Add Azure Subscription → Accept → Connect
- **Copilot:** Enterprise AI controls → Copilot → Enable → Org Settings → Copilot → Access → Assign seats

---

## ✅ Accuracy & Click-Path Notes

- Reviewed against current public GitHub and Microsoft documentation in April 2026 where public documentation is available. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
- When a path starts with `Enterprise`, begin at GitHub, click your profile photo, click `Your enterprises` or `Enterprise`, select the enterprise, then continue with the listed top tab or left-sidebar item.
- When a path starts with `Organization` or `Org`, begin at GitHub, click your profile photo, click `Your organizations`, select the organization, click `Settings`, then continue with the listed sidebar item.
- When a path starts with `Repository`, `Repo`, or a repository name, open the repository, click the `Settings` tab, then continue with the listed sidebar item.
- When a path starts with a vendor portal such as `Microsoft Entra admin center`, `Azure portal`, `Okta Admin Console`, `PingFederate`, `PingOne`, `OneLogin`, `AD FS Management`, `Visual Studio Admin Portal`, or `Azure DevOps`, sign in to that admin portal first, select the tenant, application, or project named in the step, then follow each listed blade, tab, button, and confirmation in order.
- If the expected button is missing, verify you are signed in with the role named in Prerequisites, the feature or license is enabled, and the object is owned by the selected enterprise, organization, or repository. Use page search only to locate the same page, not to skip required confirmation, test, save, or consent clicks.

---

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

> ⚠️ **Important:** This user is required because SCIM provisioning acts on behalf of a specific GitHub user via a PAT (Personal Access Token). If that user loses access or leaves the org, SCIM can stop working—so you want a stable, dedicated identity.

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

> 📌 **Note:** This setup user will consume a GitHub license. Treat it as a system account: minimal use outside of IAM configuration.

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

> 💡 **Tip:** **PingFederate:** These values are available under SP Connection, Protocol Settings, Export Metadata, or from the server's federation metadata endpoint (e.g., `https://your-pingfed-server:9031/pf/federation_metadata.ping?PartnerSpId=https://github.com/orgs/YOUR_ORG`)

> 💡 **Tip:** **PingOne:** These values are available under the Application, Configuration tab, or by downloading the IdP metadata XML.

## 3️⃣ Enable SAML SSO in GitHub

You will configure **Organization SAML** (always for the target org).

If your org is under an Enterprise account, you may also configure **Enterprise SAML**.

> ⚠️ **Warning:** Enabling SAML impacts how members authenticate. Ensure you have recovery codes stored for break-glass access.

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

> 🔐 **Critical:** Before enabling or immediately after enabling SAML, download and securely store your organization SSO recovery codes. These are essential for break-glass scenarios if your IdP becomes unavailable.

## 4️⃣ Enforce SAML SSO for the Organization (Required)

> ⚠️ **Important:** Enforcement removes org members who have not authenticated through the IdP, and can also remove bots/service accounts that don't have external identities. **If a user rejoins the organization within three months, the user's access privileges and settings will be restored.**

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

> 💡 **Tip:** In Standard non-EMU, SCIM provisioning manages organization membership lifecycle. PingFederate/PingOne communicates with GitHub's SCIM API using a PAT from the dedicated setup user.

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

> 📌 **Note:** The SCIM tenant URL is found in your GitHub Organization → Settings → Authentication security, under the SCIM section (visible after SAML is enabled and enforced).

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
        → Billing and licensing
          → Payment information
            → Metered billing via Azure
              → Add Azure Subscription
```

**Process:**

1. Navigate to your org or enterprise settings entry point:
    - Org list: `https://github.com/settings/organizations`
    - Enterprise list: `https://github.com/settings/enterprises`
2. Open the target org/enterprise
3. Click **Billing and licensing:**
    - For orgs: in the left sidebar under settings (label can vary slightly by UI)
    - For enterprises: a top-level tab
4. Click **Payment information**
5. Scroll to the bottom. To the right of **Metered billing via Azure**, click:
    - **Add Azure Subscription**
6. Sign in to Microsoft when prompted
7. On **Permissions requested**, click **Accept** (or follow admin approval flow if required)
8. Under **Select a subscription**, pick the Azure Subscription ID
9. Click **Connect**

> 💡 **Tip:** If you don't see a "Permissions requested" prompt and instead see a message about needing admin approval, you may need to configure an admin consent workflow in Azure or work with your Azure AD global administrator.

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

> 📌 **Note:** GitHub states **PAT classic** requires post-creation SSO authorization. **Fine-grained PATs** are authorized during creation, before org access is granted.

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

---
## 🧯 Known Errors & Resolutions

> This section lists the known product errors and admin-facing symptoms that commonly occur with this workflow. Exact message text can vary by product rollout, tenant policy, and provider, so use the log or settings page named in the resolution to confirm the root cause.

| Error or symptom | Likely cause | Resolution |
|------------------|--------------|------------|
| **Page, tab, or button is missing** | Wrong account context, missing admin role, unavailable plan/add-on, or feature rollout not enabled for the selected enterprise/org/repo. | Switch to the correct account and scope, confirm the prerequisite role, verify licensing or add-on activation, then refresh the page. If the control is still absent, use the direct settings URL from the relevant GitHub Docs page and confirm the feature is available for your plan. |
| **Changes appear saved but behavior does not change** | Policy inheritance, cached UI state, propagation delay, or an overlapping enterprise/org/repo policy. | Reopen the settings page, verify the effective policy at the lowest affected scope, wait for propagation where documented, and check for a stricter policy at an enterprise or organization level. |
| **403, forbidden, or resource not accessible** | The signed-in user or token can see the page but lacks the specific permission for the action. | Use an enterprise owner, organization owner, repository admin, or token with the exact scopes/permissions listed in the runbook. For SAML-protected orgs, authorize the token or SSH key for SSO before retrying. |
| **SAML test fails with NameID, recipient, audience, or signature errors** | Required SAML attributes, ACS URL, Entity ID, certificate, clock sync, or signing algorithm do not match GitHub requirements. | Compare every SAML value with the GitHub settings page, send a stable email/NameID, use SHA-256 signing, refresh the certificate, and retest before enforcing. |
| **SCIM test connection fails** | Tenant URL, bearer token, SCIM endpoint, token owner, or IdP provisioning mode is incorrect. | Regenerate the SCIM token from the correct GitHub setup/admin account, paste the exact tenant URL, and confirm the IdP provisioning test succeeds before assigning users. |
| **Provisioned users are missing from GitHub** | Users or groups are not assigned to the IdP app, attribute mappings fail, or provisioning cycles have not completed. | Review IdP provisioning logs, fix mapping errors, assign a small pilot group, and wait for the next incremental provisioning cycle. |
| **Azure billing connection fails** | The Azure signer cannot grant tenant consent or does not own the subscription. | Use a subscription owner with tenant consent rights or run the Entra admin consent workflow, then repeat the GitHub Add Azure Subscription flow. |
| **Copilot controls or seats are not visible** | Copilot is not enabled for the enterprise/org, the signed-in user lacks owner/admin permissions, or the plan/add-on is not active. | Verify Copilot plan activation, enable access at the enterprise/org level, and assign seats from the documented access page. |

---

## ❓ Common Questions & Troubleshooting

### Q: What is the difference between an SP Connection (PingFederate) and an Application (PingOne) for this setup?
**A:** An SP Connection in PingFederate (self-managed) is the equivalent of an Application in PingOne (cloud). Both define the SAML trust relationship between Ping and GitHub. In PingFederate, you configure SP Connections under Identity Provider > SP Connections. In PingOne, you configure Applications under Connections > Applications. The SAML values (Entity ID, ACS URL, Sign-on URL) are the same — the admin UI and navigation differ.

---

### Q: The SAML certificate exported from PingFederate is in DER format — GitHub rejects it. What do I do?
**A:** GitHub requires the certificate in Base64-encoded PEM format (the text that starts with `-----BEGIN CERTIFICATE-----`). If you exported in DER/binary format, convert it using OpenSSL: `openssl x509 -inform DER -in cert.der -outform PEM -out cert.pem`. Alternatively, re-export from PingFederate and select "X.509 Certificate (PEM)" as the export format. Paste the full PEM text (including the BEGIN/END lines) into GitHub.

---

### Q: Outbound provisioning from PingFederate is failing with connection errors — what should I check?
**A:** Verify: (1) the SCIM Base URL is correct (`https://api.github.com/scim/v2/organizations/YOUR_ORG`), (2) the Bearer Token (PAT) is valid and has the `admin:org` scope, (3) the PAT has been SSO-authorized for the org, and (4) PingFederate can reach `api.github.com` on port 443 (check firewall/proxy rules). Test the connection using the PingFederate Admin Console. If using PingOne, verify the same settings under the Provisioning tab.

---

### Q: SAML enforcement removed my service accounts — how do I handle automation with PingFederate?
**A:** Service accounts without IdP identities are removed during enforcement. For PingFederate environments: (1) create IdP identities in your LDAP/AD directory for service accounts and assign them to the SP Connection, or (2) migrate automation to GitHub Apps, which are not affected by SAML enforcement. If service accounts were removed, they can rejoin within three months with their access restored.

---

### Q: The signing certificate on PingFederate is about to expire — how do I rotate without downtime?
**A:** Generate a new signing certificate in PingFederate (Security > Signing & Decryption Keys & Certificates). Before activating it as the primary cert in PingFederate, paste the new certificate into GitHub (Organization Settings > Authentication security > SAML > Public certificate > update). Save in GitHub first, then activate the new cert in PingFederate. This ensures GitHub trusts the new cert before PingFederate starts using it.

---

### Q: PingFederate's token-based authentication for SCIM uses a PAT — does the PAT need SSO authorization?
**A:** Yes. In standard non-EMU GHEC with SAML enforced, any PAT (classic) used for API access to the org must be SSO-authorized. After generating the PAT as the setup user, go to Settings > Developer settings > Personal access tokens, click "Configure SSO" next to the token, and authorize it for the org. Without this step, SCIM API calls from PingFederate will return 403 errors.

---

### Q: Users can authenticate via SAML but are not being provisioned into the org — what is missing?
**A:** SAML authentication and SCIM provisioning are separate configurations. A user can authenticate via SAML but still not be an org member if SCIM provisioning is not set up or if the user is not in scope for the outbound provisioning channel. Verify that outbound provisioning is enabled on the SP Connection (PingFederate) or the Provisioning tab (PingOne), and that the user is in the correct LDAP/AD group or PingOne group assigned to the application.

---

### Q: How do I test the SAML configuration before rolling out to all users?
**A:** Enable SAML in GitHub org settings without enforcing it. Use the "Test SAML configuration" button to validate the flow with your PingFederate credentials. Assign a pilot group in PingFederate (via access control policy on the SP Connection) and have them authenticate at `https://github.com/orgs/YOUR_ORG/sso`. Once validated, proceed with enforcement.

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Enterprise Environment Scaffolding Checklist | `Setup/Enterprise Environment Scaffolding Checklist.md` |
| Standard Enterprise to EMU Migration | `Setup/Standard Enterprise to EMU Migration.md` |
| Branch Protection Rules & Rulesets | `Governance/Branch Protection Rules & Rulesets.md` |
| Copilot Seat Assignment & Enablement | `Copilot/Seat Assignment & Enablement.md` |
| Cost Centers & Department Billing | `Billing/Cost Centers & Department Billing.md` |

---

## 📝 Resources

- [About identity and access management with SAML single sign-on - GitHub Docs](https://docs.github.com/en/organizations/managing-saml-single-sign-on-for-your-organization/about-identity-and-access-management-with-saml-single-sign-on)
- [PingIdentity GitHub Integration Documentation](https://docs.pingidentity.com/integrations/github/)

---

*Last updated: April 2026*
