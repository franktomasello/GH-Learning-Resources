# 🚀 GitHub Standard GHEC + AD FS (SAML), Azure Billing & Copilot Setup Runbook

> **Complete end-to-end runbook for configuring Standard (non-EMU) GHEC with Active Directory Federation Services (AD FS) SAML, user provisioning, Azure billing, and GitHub Copilot**

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [📋 Overview](#-overview)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [1️⃣ Create SCIM Setup User (Note: SCIM Not Natively Supported by AD FS)](#1-create-scim-setup-user-note-scim-not-natively-supported-by-ad-fs)
- [2️⃣ Add Relying Party Trust in AD FS](#2-add-relying-party-trust-in-ad-fs)
- [3️⃣ Configure Claim Rules](#3-configure-claim-rules)
- [4️⃣ Enable & Test SAML SSO in GitHub](#4-enable--test-saml-sso-in-github)
- [5️⃣ Enforce SAML SSO for the Organization (Required)](#5-enforce-saml-sso-for-the-organization-required)
- [6️⃣ User Provisioning (Manual or Scripted — No Native SCIM)](#6-user-provisioning-manual-or-scripted--no-native-scim)
- [7️⃣ Assign Users (AD Group → Mapped via Claims)](#7-assign-users-ad-group--mapped-via-claims)
- [8️⃣ Attach Azure Subscription for Metered Billing](#8-attach-azure-subscription-for-metered-billing)
- [9️⃣ Enable GitHub Copilot (Enterprise + Organization)](#9-enable-github-copilot-enterprise--organization)
- [🔟 Critical Post-Enablement: SSO Authorization for Credentials (Required)](#-critical-post-enablement-sso-authorization-for-credentials-required)
- [✅ Pre-Flight / Validation Checklist](#-pre-flight--validation-checklist)
- [🎯 Success Criteria](#-success-criteria)
- [💡 Recommendation: Consider Migrating to Microsoft Entra ID](#-recommendation-consider-migrating-to-microsoft-entra-id)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **AD FS:** Trust Relationships → Relying Party Trusts → Add → Manual → Entity ID `https://github.com/orgs/YOUR_ORG` → ACS URL → Configure claim rules (email → NameID)
- **GitHub:** Org Settings → Authentication security → SAML single sign-on → Paste AD FS SSO URL, Federation Service ID, token-signing cert → Test → Save → Enforce
- **Provisioning:** Manual (invite users) or API scripts (AD group sync) — AD FS does NOT support native SCIM
- **Billing:** Org/Enterprise → Billing and licensing → Payment information → Add Azure Subscription → Accept → Connect
- **Copilot:** Enterprise AI controls → Copilot → Enable → Org Settings → Copilot → Access → Assign seats

---

## ✅ Accuracy & Click-Path Notes

<details>
<summary><em>Show click-path conventions</em></summary>


- Reviewed against current public GitHub and Microsoft documentation in April 2026 where public documentation is available. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
- When a path starts with `Enterprise`, begin at GitHub, click your profile photo, click `Your enterprises` or `Enterprise`, select the enterprise, then continue with the listed top tab or left-sidebar item.
- When a path starts with `Organization` or `Org`, begin at GitHub, click your profile photo, click `Your organizations`, select the organization, click `Settings`, then continue with the listed sidebar item.
- When a path starts with `Repository`, `Repo`, or a repository name, open the repository, click the `Settings` tab, then continue with the listed sidebar item.
- When a path starts with a vendor portal such as `Microsoft Entra admin center`, `Azure portal`, `Okta Admin Console`, `PingFederate`, `PingOne`, `OneLogin`, `AD FS Management`, `Visual Studio Admin Portal`, or `Azure DevOps`, sign in to that admin portal first, select the tenant, application, or project named in the step, then follow each listed blade, tab, button, and confirmation in order.
- If the expected button is missing, verify you are signed in with the role named in Prerequisites, the feature or license is enabled, and the object is owned by the selected enterprise, organization, or repository. Use page search only to locate the same page, not to skip required confirmation, test, save, or consent clicks.

</details>

---

## 📋 Overview

This guide walks through setting up **Standard (non-EMU) GitHub Enterprise Cloud (GHEC)**, including:

- **AD FS (SAML SSO)** for authentication
- **Enforce SAML SSO** for the organization (and enterprise, if applicable)
- **User provisioning** (manual or scripted — AD FS does NOT natively support SCIM)
- **Azure subscription** attachment for metered billing
- **GitHub Copilot** enablement + policy controls
- **Critical post-enablement** items (PAT/SSH SSO authorization)

### Standard (non-EMU) means:

- Users keep their regular GitHub.com accounts
- SAML SSO is enforced at the org (and optionally at the enterprise, if you have one)
- Organization membership is managed manually, via API scripts, or through a SCIM bridge/proxy (not "managed user accounts"—that is EMU)

### Important Note on AD FS

- AD FS is an **on-premises only** identity provider
- AD FS does **NOT** natively support SCIM provisioning for GitHub
- For organizations seeking full SCIM lifecycle management and cloud-native capabilities, consider migrating to **Microsoft Entra ID** (see Section 12)

## ✅ Prerequisites

| Requirement | Owner / Role | Notes |
| --- | --- | --- |
| GitHub Enterprise Cloud (Standard / non-EMU) org | Org Owner | You must be able to access Org Settings → Authentication security |
| (If applicable) GitHub Enterprise account | Enterprise Owner | Needed only if you will configure Enterprise SAML and/or Enterprise Copilot policies |
| AD FS server (Windows Server with AD FS role) | AD/Infrastructure admin | AD FS must be installed, configured, and operational |
| Publicly accessible federation endpoint (or WAP) | Network/Infrastructure admin | AD FS must be reachable from GitHub's servers for SAML. Use Web Application Proxy (WAP) if AD FS is not directly internet-facing |
| AD FS admin access | AD FS Admin | Must be able to add Relying Party Trusts and configure claim rules |
| Active Directory users/groups | IAM team | Use a small pilot cohort first to reduce risk |
| Dedicated GitHub "SCIM setup user" (optional) | Org Owner | Only needed if you plan to use API-based provisioning scripts or a SCIM bridge |
| Azure subscription + ability to consent | Azure admin | Needed to connect metered billing via Azure. If subscription is in a different tenant, you may need to specify a different tenant ID during connection |
| Copilot plan decision | Enterprise/Org owner | Copilot Business vs Copilot Enterprise |

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **AD FS administrator** | Creates the relying party trust and claim rules for GitHub SAML. | AD FS Management → Relying Party Trusts → Add Relying Party Trust → Claims aware → enter or import GitHub metadata/URLs → configure identifiers and endpoints → Finish. Then Relying Party Trusts → [GitHub trust] → Edit Claim Issuance Policy → Add Rule → send LDAP attributes or transform claims for NameID/email → OK. Handoff: Sign-on URL, Issuer, certificate, and claim rule export. |
| **GitHub organization owner** | Enables org SAML using the AD FS values and handles manual or scripted user provisioning. | GitHub → profile photo → Your organizations → [org] → Settings → Authentication security → SAML single sign-on → Enable SAML authentication → paste AD FS Sign on URL, Issuer, and Public Certificate → Test SAML configuration → Save → download recovery codes. Provisioning is manual/scripted because AD FS does not provide native GitHub SCIM. Handoff: SAML test success, recovery codes, and user provisioning roster. |
| **Active Directory group owner** | Maintains the AD group used by AD FS claim rules or manual provisioning. | Active Directory Users and Computers → [domain] → Users or target OU → [group] → Members → Add → enter users → Check Names → OK. Handoff: AD group name, pilot users, and owner for ongoing changes. |
| **GitHub enterprise or organization owner** | Starts the Azure metered billing connection from GitHub. | Enterprise path: GitHub → profile photo → Your enterprises → [enterprise] → Billing and licensing → Payment information → Metered billing via Azure → Add Azure Subscription. Organization path: GitHub → profile photo → Your organizations → [organization] → Settings → Billing and licensing → Payment information → Metered billing via Azure → Add Azure Subscription. Then sign in to Microsoft → Permissions requested → Accept → Select a subscription → Connect. Handoff: the subscription ID is visible on Payment information. |
| **Azure subscription Owner** | Provides the Azure subscription that GitHub will bill against, or grants another signer the required Azure RBAC rights. | Azure portal → Subscriptions → [subscription] → Access control (IAM) → Role assignments → confirm the signer is listed under Owner. To grant access: Add → Add role assignment → Privileged administrator roles → Owner → Members → Select members → [user] → Select → Review + assign. Handoff: subscription ID and tenant ID. |
| **Microsoft Entra Global Administrator or consent approver** | Approves tenant-wide consent when the Microsoft consent prompt blocks the GitHub billing app. | Microsoft Entra admin center → Entra ID → Enterprise apps → Activity → Admin consent requests → My Pending → [GitHub request] → Review permissions and consent → Approve. If the Global Administrator completes the GitHub flow directly, approve the Permissions requested prompt by clicking Accept. |

---

## 1️⃣ Create SCIM Setup User (Note: SCIM Not Natively Supported by AD FS)

> ⚠️ **Important:** AD FS does not natively support SCIM provisioning. Unlike cloud IdPs (Okta, OneLogin, Entra ID), AD FS cannot automatically provision or deprovision GitHub organization members.

### Options for User Provisioning

| Approach | Complexity | Notes |
| --- | --- | --- |
| **Manual management** | Low | Add/remove org members manually in GitHub |
| **API scripts** | Medium | Use GitHub's REST/GraphQL API to sync AD group membership to GitHub org membership on a schedule |
| **SCIM bridge/proxy** | High | Deploy a middleware layer that translates AD group changes into SCIM calls to GitHub |

### If Using API Scripts or SCIM Bridge

1. Create a dedicated GitHub.com account (example: `gh-adfs-scim@yourdomain.com`)
2. **Secure the account:**
    - Enable 2FA
    - Ensure recovery methods are stored per your approved process (vault / break-glass)
3. **Add the setup user to your GitHub organization as an Owner:**
    - GitHub → your Organization → Settings → People → Invite member
    - After accepted → set role to Owner
4. **Generate a PAT (classic)** with `admin:org` scope for API-based provisioning
5. Authorize the PAT for SSO (see Section 10B)

### Important Notes

> 📌 **Note:** This setup user will consume a GitHub license (if used). Treat it as a system account: minimal use outside of IAM configuration. If using manual management only, this step can be skipped.

## 2️⃣ Add Relying Party Trust in AD FS

### Navigation (AD FS Management Console)

```
AD FS Management Console (on the AD FS server)
  → Trust Relationships
    → Relying Party Trusts
      → Add Relying Party Trust...

```

### Configuration Steps

1. **Start the Add Relying Party Trust Wizard**
2. Select **Enter data about the relying party manually** (or use metadata URL import if available)
3. **Display name:** `GitHub - YOUR_ORG` (or a descriptive name)
4. **Profile:** Select **AD FS profile**
5. **Configure URL:** Enable **SAML 2.0 WebSSO protocol**
    - Relying party SAML 2.0 SSO service URL (Assertion Consumer Service):

    ```
    https://github.com/orgs/YOUR_ORG/saml/consume
    ```

6. **Configure Identifiers:** Add the relying party trust identifier (Entity ID):

    ```
    https://github.com/orgs/YOUR_ORG
    ```

7. **Access Control:** Select the appropriate access control policy (e.g., Permit Everyone, or restrict to specific AD groups)
8. **Review and Finish:** Confirm settings and close the wizard
9. The **Edit Claim Issuance Policy** dialog may open automatically — configure claim rules in the next step

Replace `YOUR_ORG` with your actual GitHub organization slug.

### GitHub Org SAML Values (for reference)

| Field | Value |
| --- | --- |
| Entity ID / Identifier | `https://github.com/orgs/YOUR_ORG` |
| ACS (Assertion Consumer Service) URL | `https://github.com/orgs/YOUR_ORG/saml/consume` |
| Sign-on URL | `https://github.com/orgs/YOUR_ORG/sso` |

## 3️⃣ Configure Claim Rules

### Navigation (AD FS Management Console)

```
AD FS Management Console
  → Trust Relationships
    → Relying Party Trusts
      → [Right-click "GitHub - YOUR_ORG"]
        → Edit Claim Issuance Policy...

```

### Required Claim Rules

**Rule 1: Send LDAP Attributes as Claims**

1. Click **Add Rule...**
2. Claim rule template: **Send LDAP Attributes as Claims**
3. Claim rule name: `Send Email as NameID` (or descriptive name)
4. Attribute store: **Active Directory**
5. Map the following LDAP attributes:

| LDAP Attribute | Outgoing Claim Type |
| --- | --- |
| E-Mail-Addresses | E-Mail Address |

**Rule 2: Transform Email to Name ID**

1. Click **Add Rule...**
2. Claim rule template: **Transform an Incoming Claim**
3. Claim rule name: `Transform Email to NameID`
4. Configure:
    - Incoming claim type: **E-Mail Address**
    - Outgoing claim type: **Name ID**
    - Outgoing name ID format: **Email**
    - Select: **Pass through all claim values**

### Optional Additional Claims

You may also map additional attributes as needed:

| LDAP Attribute | Outgoing Claim Type | Notes |
| --- | --- | --- |
| Display-Name | `displayName` | User's full name |
| User-Principal-Name | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn` | If using UPN instead of email for NameID |

### Gather AD FS IdP Values for GitHub

From the AD FS server, collect:

1. **Federation Service Identifier** (Entity ID) — found in AD FS → Service → Edit Federation Service Properties
2. **SSO Service URL** — typically: `https://your-adfs-server/adfs/ls/`
3. **Token-signing certificate** — export as Base64-encoded `.cer`:

```
AD FS Management Console
  → Service
    → Certificates
      → Token-signing certificate
        → [Right-click] → View Certificate
          → Details → Copy to File...
            → Base-64 encoded X.509 (.CER)

```

## 4️⃣ Enable & Test SAML SSO in GitHub

You will configure **Organization SAML** (always for the target org).

If your org is under an Enterprise account, you may also configure **Enterprise SAML**.

> ⚠️ **Warning:** Enabling SAML impacts how members authenticate. Ensure you have recovery codes stored for break-glass access.

### 4A — (Conditionally Required) Configure Enterprise SAML

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
2. Enable / configure SAML per GitHub's enterprise SAML documentation (AD FS values differ from org app)

### 4B — Enable & Test Organization SAML (Required)

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
2. Populate fields using AD FS values from Step 3:
    - **Sign on URL** — AD FS SSO Service URL (e.g., `https://your-adfs-server/adfs/ls/`)
    - **Issuer** — AD FS Federation Service Identifier — _Note: GitHub indicates Issuer is required for some features like team synchronization_
    - **Public Certificate** — AD FS token-signing certificate (Base64 .cer contents)
3. Click **Test SAML configuration** (or equivalent prompt) and complete the auth flow
4. Click **Save**
5. **Immediately download and secure SSO recovery codes:**
    - Organization Settings → Authentication security → Single sign-on recovery codes

> 🔐 **Critical:** Before enabling or immediately after enabling SAML, download and securely store your organization SSO recovery codes. These are essential for break-glass scenarios if your IdP becomes unavailable.

## 5️⃣ Enforce SAML SSO for the Organization (Required)

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

## 6️⃣ User Provisioning (Manual or Scripted — No Native SCIM)

> ⚠️ **Important:** AD FS does not provide native SCIM support. You must manage GitHub organization membership through one of the following approaches.

### Option A: Manual Provisioning

1. Invite users to the GitHub organization manually:

```
GitHub (top-right profile picture)
  → Organizations
    → (next to your org) Settings
      → People
        → Invite member

```

2. Users accept the invitation and authenticate via SAML (AD FS) on first access
3. To remove users: Organization Settings → People → remove member

### Option B: API-Based Provisioning (Scripted)

1. Use the GitHub REST API to manage organization membership programmatically
2. Build a sync script that:
    - Reads AD group membership (via LDAP or PowerShell)
    - Compares against GitHub org membership (via GitHub API)
    - Invites new members / removes departed members
3. Schedule the script to run on a regular interval (e.g., daily via Task Scheduler or cron)
4. Use the PAT generated in Step 1 for API authentication

### Option C: SCIM Bridge / Proxy

1. Deploy a middleware application that:
    - Listens for AD group membership changes
    - Translates them into SCIM API calls to GitHub
2. This is the most complex option but provides the closest experience to native SCIM
3. Consider third-party tools or custom solutions for this approach

### Important Notes

> 📌 **Note:** Regardless of provisioning method, users must still authenticate via SAML (AD FS) to access the org. For new hires: invite to GitHub org, then user authenticates via AD FS on first access. For departures: remove from GitHub org (manually or via script) and disable AD account.

## 7️⃣ Assign Users (AD Group → Mapped via Claims)

### Controlling Access via AD Groups

Since AD FS uses Active Directory as its identity store, you can control which users can access GitHub by:

1. **Create an AD security group** (e.g., `GitHub-Org-Members`)
2. **Add pilot users** to the AD group
3. **Configure AD FS access control:**

```
AD FS Management Console
  → Trust Relationships
    → Relying Party Trusts
      → [Right-click "GitHub - YOUR_ORG"]
        → Edit Access Control Policy...

```

4. Select **Permit specific group** and choose your AD security group
    - Alternatively, add an Issuance Authorization Rule to restrict access based on group membership
5. **Validate access:**
    - Users in the AD group can authenticate via SAML to the GitHub org
    - Users not in the AD group are denied at the AD FS level

### Lifecycle Management

- **Add user to AD group** → user can authenticate to GitHub via SAML → invite to org (manual or scripted)
- **Remove user from AD group** → user can no longer authenticate via SAML → remove from org (manual or scripted)

## 8️⃣ Attach Azure Subscription for Metered Billing

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

## 9️⃣ Enable GitHub Copilot (Enterprise + Organization)

### 9A — (Conditionally Required) Enable Copilot at the Enterprise level via Payment Verification

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

### 9B — Configure Copilot Enterprise Policies

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

### 9C — Configure Copilot Organization Policies (Required)

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

### 9D — Assign Copilot licenses (Required)

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

## 🔟 Critical Post-Enablement: SSO Authorization for Credentials (Required)

When SAML is enabled/enforced, users often must authorize credentials (depending on token type and whether they have a linked external identity).

### 10A — Authorize SSH Keys for SSO

**Required for SSH usage in SSO orgs**

**Navigation**

```
GitHub (top-right profile picture)
  → Settings
    → SSH and GPG keys
      → (next to the key) Configure SSO
        → Authorize (for the org)

```

### 10B — Authorize Personal Access Tokens

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
- AD FS server hostname / federation endpoint: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
- AD FS endpoint publicly accessible (directly or via WAP): ☐
- AD FS admin with privileges to add Relying Party Trusts: ☐
- Provisioning approach chosen (manual / API script / SCIM bridge): \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
- Azure Subscription ID (for billing): \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
- Azure admin who can grant tenant-wide consent: ☐

### SAML

- [ ] Relying Party Trust created in AD FS
- [ ] Claim rules configured (NameID = email)
- [ ] Org SAML enabled and tested
- [ ] Recovery codes downloaded and stored
- [ ] Enforcement applied successfully
- [ ] Pilot users can access org resources via SSO

### User Provisioning

- [ ] Provisioning approach implemented (manual / API script / SCIM bridge)
- [ ] Pilot users invited and authenticated via SAML
- [ ] User removal process tested and confirmed

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

- ✅ Standard GHEC organization fully configured with AD FS SAML authentication
- ✅ SAML SSO enforced for the organization
- ✅ User provisioning process established (manual, scripted, or SCIM bridge)
- ✅ Azure subscription connected for metered billing (if applicable)
- ✅ GitHub Copilot enabled and configured
- ✅ Users have authorized SSH keys and PATs for SSO access
- ✅ Pilot users provisioned and able to access GitHub via SSO

## 💡 Recommendation: Consider Migrating to Microsoft Entra ID

If your organization is currently using AD FS and is evaluating long-term identity strategy, consider migrating to **Microsoft Entra ID** (formerly Azure AD) for the following benefits:

| Capability | AD FS | Entra ID |
| --- | --- | --- |
| SAML SSO | ✅ | ✅ |
| Native SCIM provisioning | ❌ | ✅ |
| Automated user lifecycle management | Manual/scripted | Native |
| Conditional Access policies | Limited | Full |
| Cloud-native (no on-prem infrastructure) | ❌ | ✅ |
| GitHub EMU support | ❌ | ✅ |
| Ongoing maintenance burden | High (on-prem servers) | Low (SaaS) |

Microsoft provides migration tooling and documentation for moving from AD FS to Entra ID. This migration would unlock native SCIM provisioning, automated lifecycle management, and a path to GitHub Enterprise Managed Users (EMU) if desired in the future.

---
## 🧯 Known Errors & Resolutions

<details>
<summary><em>Show known errors table</em></summary>


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

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>


### Q: GitHub cannot reach my AD FS server — SAML test fails with a connection error. What do I need?
**A:** AD FS must be publicly accessible from the internet for GitHub's SAML flow to work. If your AD FS server is behind a firewall, deploy a Web Application Proxy (WAP) server in your DMZ that proxies SAML requests to the internal AD FS server. The WAP publishes the AD FS federation endpoint (typically `https://your-adfs-server/adfs/ls/`) externally. Without WAP or public access, GitHub cannot validate SAML assertions and SSO will fail.

---

### Q: My claim rules are configured, but GitHub shows "NameID not found in SAML assertion" — what is wrong?
**A:** This error means the SAML assertion does not contain a properly formatted NameID claim. Verify you have both claim rules configured: (1) an LDAP Attributes rule that sends the email as an E-Mail Address claim, and (2) a Transform rule that converts E-Mail Address to Name ID with the "Email" outgoing format. The two rules must be in the correct order — the LDAP rule first, then the transform rule. Use a SAML tracer browser extension to inspect the actual assertion.

---

### Q: AD FS does not support SCIM — how do I manage user provisioning?
**A:** AD FS has no native SCIM support. Your options are: (1) **Manual provisioning** — invite users to the org individually via GitHub settings, (2) **API scripts** — build a PowerShell or Python script that reads AD group membership and uses the GitHub REST API to invite/remove org members on a schedule, or (3) **SCIM bridge** — deploy middleware that translates AD changes into SCIM API calls. For most organizations, option 2 (API scripts) provides the best balance of automation and simplicity.

---

### Q: The AD FS certificate needs renewal — how do I update it in GitHub without breaking SSO?
**A:** When the AD FS token-signing certificate is near expiration, generate the new certificate in AD FS Management Console (Service > Certificates). Export it in Base64-encoded X.509 (.CER) format. Update the certificate in GitHub first (Organization Settings > Authentication security > SAML > Public certificate), save, then activate the new certificate as primary in AD FS. Updating GitHub before AD FS ensures there is no window where the certificates are mismatched.

---

### Q: GitHub rejects the SAML response with a "signature algorithm mismatch" error — what is the issue?
**A:** GitHub requires SHA-256 for both the signature and digest algorithms. AD FS may default to SHA-1 on older configurations. In AD FS Management Console, open the Relying Party Trust properties for GitHub, go to the Advanced tab, and change the Secure Hash Algorithm to SHA-256. If the Advanced tab is not available, use PowerShell: `Set-AdfsRelyingPartyTrust -TargetName "GitHub - YOUR_ORG" -SignatureAlgorithm "http://www.w3.org/2001/04/xmldsig-more#rsa-sha256"`.

---

### Q: Users authenticate successfully via AD FS but end up in an SSO redirect loop — what causes this?
**A:** An SSO redirect loop usually means the SAML configuration values do not match between AD FS and GitHub. Verify: (1) the Entity ID (Relying Party Trust Identifier) matches `https://github.com/orgs/YOUR_ORG` exactly, (2) the ACS URL matches `https://github.com/orgs/YOUR_ORG/saml/consume`, (3) the Federation Service Identifier (Issuer) in GitHub matches the Entity ID in AD FS. Also check for clock skew between the AD FS server and GitHub — time differences greater than 5 minutes can cause assertion validation failures.

---

### Q: We are considering migrating from AD FS to Entra ID — what are the benefits for GitHub?
**A:** Migrating to Entra ID unlocks native SCIM provisioning (eliminating the need for manual provisioning or scripts), Conditional Access Policies, cloud-native management with no on-premises servers, and a path to GitHub Enterprise Managed Users (EMU) if desired. Microsoft provides migration tooling for AD FS to Entra ID. For GitHub specifically, you would switch from the AD FS Relying Party Trust to the Entra ID Enterprise Application for SAML and gain automatic user lifecycle management through SCIM.

---

### Q: Can I use the AD FS proxy URL as the Sign-on URL in GitHub, or must it be the direct AD FS URL?
**A:** Use the externally accessible URL — which is typically the Web Application Proxy (WAP) URL or a load-balanced federation endpoint. This is the URL GitHub will redirect users to for authentication. If you use `https://adfs.yourdomain.com/adfs/ls/`, that URL must be resolvable and reachable from the public internet. Internal-only URLs will cause authentication to fail for users outside your corporate network.

</details>

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
- [AD FS Operations - Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/identity/ad-fs/ad-fs-operations)

---

*Last updated: April 2026*
