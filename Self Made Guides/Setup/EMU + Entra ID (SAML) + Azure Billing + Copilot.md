# 🚀 GitHub EMU + Microsoft Entra ID (SAML), Azure Billing & Copilot Setup Runbook

> **Complete end-to-end runbook for configuring EMU with Microsoft Entra ID (SAML), Azure billing, and GitHub Copilot**

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [📋 Overview](#-overview)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [1️⃣ Create & Configure the EMU Setup User](#1-create--configure-the-emu-setup-user)
- [2️⃣ Create the Entra ID Enterprise Application](#2-create-the-entra-id-enterprise-application)
- [3️⃣ Enable SAML SSO in GitHub](#3-enable-saml-sso-in-github)
- [4️⃣ Configure SCIM Provisioning](#4-configure-scim-provisioning)
- [5️⃣ Attach Azure Subscription for Billing](#5-attach-azure-subscription-for-billing)
- [6️⃣ Enable GitHub Copilot](#6-enable-github-copilot)
- [7️⃣ Organization Structure Guidance](#7-organization-structure-guidance)
- [📝 Quick Reference Worksheet](#-quick-reference-worksheet)
- [✅ Pre-Flight Checklist](#-pre-flight-checklist)
- [👥 Example Initial Group Model](#-example-initial-group-model)
- [🎯 Success Criteria](#-success-criteria)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Entra ID:** Create Enterprise App **GitHub Enterprise Managed User** from gallery → Configure SAML with Entity ID `https://github.com/enterprises/YOUR_ENTERPRISE`
- **GitHub:** Sign in as `SHORTCODE_admin` → **Your enterprises** → *[enterprise]* → **Identity provider** → **Single sign-on configuration** → **Add SAML configuration** → paste **Sign on URL**, **Issuer**, **Public Certificate** → **Test SAML configuration** → **Save SAML settings** → save recovery codes
- **SCIM:** As `SHORTCODE_admin`, generate a personal access token (classic) with `scim:enterprise` scope → Enter in Entra App → **Provisioning** → **Automatic** → **Test Connection**
- **Billing:** Enterprise → **Billing & Licensing** → **Payment information** → **Metered billing via Azure** → **Add Azure Subscription** → **Accept** → **Connect**
- **Copilot:** Enterprise → **AI controls** → **Copilot** → enable for orgs → assign **Copilot Business** licenses at the enterprise or org level

---

## ✅ Accuracy & Click-Path Notes

<details>
<summary><em>Show click-path conventions</em></summary>


- Reviewed against current public GitHub and Microsoft documentation in July 2026 where public documentation is available. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
- When a path starts with `Enterprise`, begin at GitHub, click your profile photo, click `Your enterprises` or `Enterprise`, select the enterprise, then continue with the listed top tab or left-sidebar item.
- When a path starts with `Organization` or `Org`, begin at GitHub, click your profile photo, click `Your organizations`, select the organization, click `Settings`, then continue with the listed sidebar item.
- When a path starts with `Repository`, `Repo`, or a repository name, open the repository, click the `Settings` tab, then continue with the listed sidebar item.
- When a path starts with a vendor portal such as `Microsoft Entra admin center`, `Azure portal`, `Okta Admin Console`, `PingFederate`, `PingOne`, `OneLogin`, `AD FS Management`, `Visual Studio Admin Portal`, or `Azure DevOps`, sign in to that admin portal first, select the tenant, application, or project named in the step, then follow each listed blade, tab, button, and confirmation in order.
- If the expected button is missing, verify you are signed in with the role named in Prerequisites, the feature or license is enabled, and the object is owned by the selected enterprise, organization, or repository. Use page search only to locate the same page, not to skip required confirmation, test, save, or consent clicks.

</details>

---

## 📋 Overview

This guide walks through setting up a new GitHub Enterprise Cloud (GHEC) with Enterprise Managed Users (EMU), including:

- ✅ EMU with Microsoft Entra ID (SAML SSO)
- ✅ SCIM provisioning for user lifecycle management
- ✅ Azure subscription attachment for billing
- ✅ GitHub Copilot enablement
- ✅ Organization structure guidance

**Hosting Options:** EMU can be hosted on GitHub.com or (for data residency) on a customer subdomain of GHE.com. Setup is similar, but SAML/SCIM URLs differ.

---

## ✅ Prerequisites

### Required Items

Before beginning, ensure you have:

| Requirement | Who / Role needed | ✓ |
|-------------|-------------------|---|
| **EMU Enterprise Created** — A new enterprise with EMU enabled | Requested via GitHub Sales / your GitHub account team | ☐ |
| **Setup User** — `SHORTCODE_admin` created by GitHub, with password set and 2FA enabled | GitHub EMU **setup user** (an enterprise owner) | ☐ |
| **Entra app + SSO/SCIM config** — Create the Enterprise App and configure SAML SSO + SCIM | Entra **Application Administrator**, **Cloud Application Administrator**, or **Application Owner** (of the app) | ☐ |
| **Enterprise SSO enablement in GitHub** — Add SAML configuration, test, and save | GitHub **enterprise owner** (the setup user) | ☐ |
| **Azure billing connection** — Connect the Azure subscription for metered billing | GitHub **enterprise owner** + Azure **subscription Owner** who can provide tenant-wide admin consent | ☐ |
| **Copilot enablement** — Enable access and set policies | GitHub **enterprise owner** | ☐ |

> 💡 **Note:** SCIM provisioning is **required** for EMU to manage user lifecycle and account creation.

---

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **Microsoft Entra Application Administrator, Cloud Application Administrator, or Application Owner** | Creates the EMU gallery app, configures SAML, and starts provisioning. | Microsoft Entra admin center → Entra ID → Enterprise apps → New application → search GitHub Enterprise Managed User → Create → Single sign-on → SAML → Basic SAML Configuration → Edit → enter Identifier, Reply URL, and Sign on URL for the enterprise → Save → SAML Certificates → download Certificate (Base64) → copy Login URL and Microsoft Entra Identifier. Then Provisioning → + New configuration (older tenants: Get started, Provisioning Mode = Automatic) → Tenant URL and Secret Token → Test Connection → Create (older UI: Save) → Users and groups → Add user/group → assign the Enterprise Owner app role → Start provisioning. Handoff: Login URL, Issuer, certificate, SCIM test success, and pilot group. |
| **GitHub EMU setup user (`SHORTCODE_admin`)** | Pastes Entra SAML values into GitHub and generates the SCIM token. | GitHub → profile photo → Your enterprises → [enterprise] → Identity provider → Single sign-on configuration → Add SAML configuration → Sign on URL, Issuer, Public Certificate → Test SAML configuration → Save SAML settings → download recovery codes. For SCIM token: setup user → Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token (classic) → `scim:enterprise` → Generate token. Handoff: saved SAML config, recovery codes, SCIM token, and Tenant URL. |
| **Microsoft Entra group owner** | Controls who is provisioned and which enterprise role they receive. | Microsoft Entra admin center → Entra ID → Groups → [pilot or production group] → Members → Add members → select users → Add, then Enterprise apps → GitHub Enterprise Managed User → Users and groups → Add user/group → select group and app role → Assign. Handoff: assigned groups and role mapping. |
| **GitHub enterprise owner** (billing manager is not sufficient to connect a subscription) | Starts the Azure metered billing connection from GitHub. | Enterprise path: GitHub → profile photo → Your enterprises → [enterprise] → Billing & Licensing → Payment information → Metered billing via Azure → Add Azure Subscription. Organization path: GitHub → profile photo → Your organizations → [organization] → Settings → Billing & Licensing → Payment information → Metered billing via Azure → Add Azure Subscription. Then sign in to Microsoft → Permissions requested → Accept → Select a subscription → Connect. Handoff: the subscription ID is visible on Payment information. |
| **Azure subscription Owner** | Provides the Azure subscription that GitHub will bill against, or grants another signer the required Azure RBAC rights. | Azure portal → Subscriptions → [subscription] → Access control (IAM) → Role assignments → confirm the signer is listed under Owner. To grant access: Add → Add role assignment → Privileged administrator roles → Owner → Members → Select members → [user] → Select → Review + assign. Handoff: subscription ID and tenant ID. |
| **Microsoft Entra Global Administrator or consent approver** | Approves tenant-wide consent when the Microsoft consent prompt blocks the GitHub billing app. | Microsoft Entra admin center → Entra ID → Enterprise apps → Activity → Admin consent requests → My Pending → [GitHub request] → Review permissions and consent → Approve. If the Global Administrator completes the GitHub flow directly, approve the Permissions requested prompt by clicking Accept. |

---

## 1️⃣ Create & Configure the EMU Setup User

**👤 Role:** GitHub EMU **setup user** · **📍 Portal:** GitHub

The setup user's username is your enterprise **shortcode** + `_admin` (e.g., `octocorp_admin`). The shortcode is chosen at creation (or randomly assigned) and **cannot be changed later**.

### Process

1. GitHub emails an invite to set the password for `SHORTCODE_admin`.
2. In a **private/incognito window**, **Set password**.
3. Enable two-factor authentication. **Navigate:** Profile photo → **Settings** → **Password and authentication** → **Two-factor authentication**.
   1. Click **Enable two-factor authentication**.
   2. Choose a method (**Set up using an app** / TOTP recommended) and scan the QR / enter the key in your authenticator.
   3. Enter the 6-digit code to **complete the challenge**.
   4. On the recovery-codes screen, click **Download** (or **Copy** / **Print**) to save the personal 2FA recovery codes, then click **I have saved my recovery codes** / **Done**.

### Important Notes

> ⚠️ **Setup User Purpose:** This account is the SCIM token owner and a break-glass account. Day-to-day enterprise administration should be done with provisioned managed enterprise-owner accounts.

> 🔐 **Every sign-in requires 2FA (Jan 2025 change):** Each setup-user sign-in requires a successful 2FA challenge **or** an enterprise recovery code. Losing both the personal 2FA recovery codes and the enterprise recovery codes locks you out permanently. Store both sets securely.

> 📌 **Password resets go through GitHub Support:** The standard email password-reset flow does **not** work for the setup user. To reset its password, open a ticket with **GitHub Support**.

> 🚨 **Email Conflict:** If the provided email address is already associated as a primary email with an existing GitHub account, the activation link will not work. Modify the existing account's primary email first.

---

## 2️⃣ Create the Entra ID Enterprise Application

**👤 Role:** Entra **Application Administrator, Cloud Application Administrator, or Application Owner** · **📍 Portal:** Microsoft Entra admin center

**Navigate:** Microsoft Entra admin center → **Entra ID** → **Enterprise apps** → **New application** → search **GitHub Enterprise Managed User** → **Create**

### Navigation

```
Microsoft Entra Admin Center
  → Entra ID
    → Enterprise applications
      → + New application
        → Browse Azure AD Gallery
          → Search: "GitHub Enterprise Managed User"
            → Create/Add
```

### SAML Configuration

1. Navigate to **Single sign-on** → Select **SAML**.
2. Click the **Edit** (pencil) on **Basic SAML Configuration**.
3. Under **Identifier (Entity ID)**, **delete the pre-filled default value**, then click **Add identifier** and enter the value from the table below (ensure **no trailing slash**).

### SAML Values

#### For GitHub.com Hosted EMU

| Field | Value |
|-------|-------|
| **Identifier (Entity ID)** | `https://github.com/enterprises/YOUR_ENTERPRISE` |
| **Reply URL (ACS URL)** | `https://github.com/enterprises/YOUR_ENTERPRISE/saml/consume` |
| **Sign-on URL** | `https://github.com/enterprises/YOUR_ENTERPRISE/sso` |

#### For GHE.com Hosted EMU (Data Residency)

| Field | Value |
|-------|-------|
| **Identifier (Entity ID)** | `https://SUBDOMAIN.ghe.com/enterprises/SUBDOMAIN` |
| **Reply URL (ACS URL)** | `https://SUBDOMAIN.ghe.com/enterprises/SUBDOMAIN/saml/consume` |
| **Sign-on URL** | `https://SUBDOMAIN.ghe.com/enterprises/SUBDOMAIN/sso` |

> **⚠️ Critical:** The Identifier format differs from the application's suggested format. Ensure the Identifier **does not** contain a trailing slash.

4. Enter **Reply URL (ACS URL)** and **Sign on URL** from the table above.
5. Click **Save** at the top of the **Basic SAML Configuration** panel, then click **X** to close the panel.

### Verify Attributes & Claims

**Navigate:** the EMU app → **Single sign-on** → section **Attributes & Claims** → **Edit**

1. Confirm the **Unique User Identifier (Name ID)** claim maps to a stable value (e.g., `user.userprincipalname` or `user.mail`).
2. Confirm the gallery-app's pre-set required claims are present.
3. Click **Save** / close the panel.

### Download Required Items

From the Entra Enterprise App **Single sign-on** page:

1. In the **SAML Certificates** section, under **Certificate (Base64)**, click **Download**.
2. In the **Set up [app name]** section, click **Copy** next to **Login URL**.
3. In the same section, click **Copy** next to **Microsoft Entra Identifier**.

### Assign Users

**Navigate:** the EMU app → **Users and groups** → **Add user/group**

1. Click **Add user/group**.
2. Under **Users**, click **None Selected**, pick the user(s)/group(s), click **Select**.
3. Under **Select a role**, click **None Selected**, choose **Enterprise Owner** (for the first admin) or the member role, click **Select**.
4. Click **Assign**.

The SCIM `roles` attribute maps the **Enterprise Owner** app role to the GitHub **enterprise owner** role (via app role assignment / `AppRoleAssignmentComplex`). Assign at least one user the **Enterprise Owner** role so a managed admin is provisioned.

> 📌 **Role-based assignment requires scoping:** Assigning the **Enterprise Owner** app role only takes effect when the provisioning **Scope** is set to **Sync only assigned users and groups** (configured in Step 4). Assign at least one user the **Enterprise Owner** role so a managed admin exists in the enterprise.

---

## 3️⃣ Enable SAML SSO in GitHub

**👤 Role:** GitHub **enterprise owner** (the setup user) · **📍 Portal:** GitHub

**Navigate:** Profile photo → **Your enterprises** → *[your enterprise]* → **Identity provider** → **Single sign-on configuration**

### Configuration Steps

1. At the top of the enterprise page, click **Identity provider**.
2. Click **Single sign-on configuration**.
3. Under **SAML single sign-on**, click **Add SAML configuration**.
4. Enter the following values from Entra:
   - **Sign on URL:** Paste the **Login URL** from Entra.
   - **Issuer:** Paste the **Microsoft Entra Identifier**.
   - **Public Certificate:** Paste the contents of the Base64 certificate.
5. Choose the **Signature Method** and **Digest Method** (SHA-256 recommended).
6. Click **Test SAML configuration**. This must pass before you can save.
7. Click **Save SAML settings**.
8. Immediately **Download**, **Print**, or **Copy** your enterprise **SSO recovery codes** and store them securely.

> 🔐 **Critical — save your recovery codes:** The enterprise **SSO recovery codes** are essential for break-glass access if your IdP becomes unavailable. Save them before you rely on SSO.

> 📌 **EMU has no "Require SAML authentication" checkbox** and no backup username/password sign-in. Enabling SSO is done entirely through **Identity provider → Single sign-on configuration → Add SAML configuration**. (The `Settings → Authentication security` path applies only to standard GitHub Enterprise Cloud, not EMU.)

---

## 4️⃣ Configure SCIM Provisioning

SCIM handles user creation and deactivation in EMU. This requires creating a token in GitHub and configuring provisioning in Entra.

### 4A — Create the SCIM Token in GitHub

**👤 Role:** GitHub **enterprise owner** (the setup user) · **📍 Portal:** GitHub

The token is a **personal access token (classic)** and must be created **while signed in as the setup user** (the setup user is an enterprise owner):

**Token Requirements:**
- ✓ Type: **personal access token (classic)**
- ✓ Scope: `scim:enterprise` (only)
- ✓ Expiration: No expiration
- ✓ Created by: `SHORTCODE_admin`

#### Creation Steps

```
GitHub (as SHORTCODE_admin)
  → Profile picture
    → Settings
      → Developer settings
        → Personal access tokens
          → Tokens (classic)
            → Generate new token (classic)
```

**Configuration:**
- **Note:** "Entra SCIM Provisioning" (or similar descriptive name)
- **Expiration:** No expiration
- **Scope:** Select `scim:enterprise` only

After setting **Note**, **Expiration = No expiration**, and checking only **`scim:enterprise`**, scroll to the bottom and click **Generate token**. Then immediately **Copy** the token (it is shown only once).

**🔑 Important:** Copy the token immediately after generation. You won't be able to see it again.

### 4B — Configure Provisioning in Entra

**👤 Role:** Entra **Application Administrator, Cloud Application Administrator, or Application Owner** · **📍 Portal:** Microsoft Entra admin center

**Navigate:** Microsoft Entra admin center → **Entra ID** → **Enterprise apps** → **GitHub Enterprise Managed User** → **Provisioning**

#### Configuration

1. Click **+ New configuration**. (Older tenants show **Get started** and a **Provisioning Mode = Automatic** dropdown — set it to **Automatic**.)
2. Under **Admin Credentials**, enter:
   - **Tenant URL:** (see table below)
   - **Secret Token:** The personal access token (classic) created in step 4A.
3. Click **Test Connection**.
4. Click **Create** (older UI: **Save**).
5. From the app's **Overview**, open **Properties** and click the **Edit** (pencil). Enable **Send an email notification when a failure occurs** (add a notification email) and **Prevent accidental deletions** (set a threshold), then click **Save**. Next, review **Attribute Mapping** (Users and Groups) in step 4C.

#### Tenant URL Values

| Environment | Tenant URL |
|-------------|------------|
| **GitHub.com** | `https://api.github.com/scim/v2/enterprises/YOUR_ENTERPRISE` |
| **GHE.com** | `https://api.SUBDOMAIN.ghe.com/scim/v2/enterprises/SUBDOMAIN` |

### 4C — Configure Attribute Mappings

1. Under **Mappings**, select **Provision Microsoft Entra ID Users** (Synchronize Microsoft Entra users to GitHub Enterprise Managed User).
2. Review the mapping and click **Save**.
3. Back under **Mappings**, open **Provision Microsoft Entra ID Groups**, review it, and click **Save**.
4. Ensure **Provisioning Status** is set to **On** under Settings.

### 4D — Assign Users/Groups for Provisioning

1. Under **Provisioning → Settings**, set **Scope** to **Sync only assigned users and groups** (required for the **Enterprise Owner** app role to take effect).
2. Navigate to **Users and groups** in the Enterprise App.
3. Assign users and/or groups to the application (assign the **Enterprise Owner** app role to at least one user).
4. Test one user first: open **Provisioning → Provision on demand**, search for and select one assigned test user, click **Provision**, and confirm all steps report success before continuing.
5. From the app's **Overview** page, click **Start provisioning**.
6. Entra will SCIM-provision these members into the EMU enterprise.

#### Provisioning Notes

> **📌 Important Constraints:**
> - Entra does **not** provision nested groups
> - Provisioning cycle runs approximately every **40 minutes**
> - Use **"Provision on demand"** for immediate provisioning
> - Do not assign more than **1,000 users per hour** to avoid rate limits

> **⚠️ Unsupported Configuration:** The combination of Okta and Entra ID for SSO and SCIM (in either order) is explicitly **not supported** by GitHub.

---

## 5️⃣ Attach Azure Subscription for Billing

**👤 Role:** GitHub **enterprise owner** + Azure **subscription Owner** · **📍 Portal:** GitHub → Microsoft

Connect your Azure subscription so GitHub usage (Copilot, Actions, Codespaces, etc.) is billed through Azure.

### Prerequisites

- ✓ You must be an **owner of the GitHub enterprise** (enterprise owner). A billing manager is **not** sufficient to connect a subscription.
- ✓ Signed in to Azure as a user with **Owner** permission on the target subscription.
- ✓ Able to provide **tenant-wide admin consent** (or coordinate with a Microsoft Entra **Global Administrator** / admin consent workflow).

### Configuration Steps

**Navigate:** Profile photo → **Your enterprises** → *[your enterprise]* → **Billing & Licensing** → **Payment information** → **Metered billing via Azure** → **Add Azure Subscription**

> 💡 **Org-level alternative:** Organizations can also connect at **Your organizations → *[organization]* → Settings → Billing & Licensing → Payment information → Metered billing via Azure**, but enterprise-level is the norm for EMU.

#### Process

1. Scroll to **Metered billing via Azure** and click **Add Azure Subscription**.
2. Sign in to your Microsoft account.
3. Review the **Permissions requested** prompt.
4. Click **Accept**.
5. Click **Select a subscription** and choose your Azure subscription.
6. Check the confirmation box.
7. Click **Connect**.
8. Confirm the connected **subscription ID** now appears on the **Payment information** page.

> **💡 Admin Consent:** If you don't see a "Permissions requested" prompt and instead see a message about needing admin approval, configure an admin consent workflow in Azure or work with your Microsoft Entra **Global Administrator**.

---

## 6️⃣ Enable GitHub Copilot

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

### 6A — Enable at Enterprise Level

With Azure billing connected via metered billing, Copilot usage will be billed through your Azure subscription.

**Navigate:** Profile photo → **Your enterprises** → *[your enterprise]* → **AI controls** → **Copilot**

> 📌 **AI controls** is a **top-of-page tab** on the enterprise account, **not** under **Settings**. Click it directly from the enterprise page, then select **Copilot** in the sidebar.

#### Access Management Configuration

Under **Access**, choose:

- **Disabled** — No organizations can use Copilot
- **All organizations** — Enable for all organizations in the enterprise
- **Specific organizations** — Select which organizations can use Copilot

For **Specific organizations**, select the organizations to enable. Then click **Save** to commit the access setting.

The Copilot plans referenced are **Copilot Business** and **Copilot Enterprise**.

### 6B — Configure Copilot Policies

1. Navigate to the **Policies** tab under **AI controls → Copilot**
2. Configure policies for:
   - Suggestions matching public code (Allowed/Blocked)
   - Copilot in GitHub.com
   - Copilot Chat in the IDE
   - Copilot in the CLI
   - Other feature policies

#### Policy Options

For each policy, select:
- **Enabled/Allowed** — Feature is on for all organizations
- **Disabled/Blocked** — Feature is off for all organizations
- **No policy** — Delegate decision to organization owners

After setting each policy (Enabled/Allowed, Disabled/Blocked, or No policy), click **Save** to apply the policy configuration.

### 6C — Assign Copilot Seats

You can assign seats at either the enterprise or the organization level:

- **Enterprise level (GA since Oct 2025):** Managing **Copilot Business** at the enterprise level is **generally available**. Enterprise owners can assign Copilot Business licenses directly at the enterprise account — to individual users **and/or** to enterprise teams — without granting org access, via the dedicated Copilot Business licensing page (**Enterprise → Billing & Licensing / Licensing**).
- **Organization level (traditional):** **Navigate:** Profile photo → **Your organizations** → *[organization]* → **Settings** → **Copilot** → **Access** → add members/teams or enable for all. Organization owners assign seats to individual members or teams.

> 📌 **Enterprise teams (the membership construct) remain in public preview**, even though enterprise-level Copilot Business license management is GA.

> ✅ **License de-duplication:** A user assigned a seat through multiple sources consumes **one** license (the highest tier).

---

## 7️⃣ Organization Structure Guidance

Organizations are boundaries for ownership, settings, and repository visibility patterns. EMU provides centralized identity and lifecycle management across all organizations.

### Recommended Patterns

#### 📐 Design Principles

**"Few orgs" bias:** Keep organization count low unless you have genuine separation needs:
- Compliance boundaries
- Distinct admin models
- Legal entity separation

#### 🏢 Common Structures

**Org-per-business-unit:**
- Use when autonomy differs
- Separate admins and policies
- Different billing/cost centers

**Org-per-environment:**
- Only if required (e.g., regulated prod code vs everything else)
- Otherwise, teams and repositories are usually sufficient

### Inside an Organization

**Teams:**
- Use **Teams** for repository access control
- Use **Entra groups** for consistent membership (synced via SCIM)
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

```yaml
SAML Entity ID:     https://github.com/enterprises/{ENTERPRISE_SLUG}
SAML ACS URL:       https://github.com/enterprises/{ENTERPRISE_SLUG}/saml/consume
SAML SSO URL:       https://github.com/enterprises/{ENTERPRISE_SLUG}/sso
SCIM Tenant URL:    https://api.github.com/scim/v2/enterprises/{ENTERPRISE_SLUG}
```

### GHE.com (Data Residency) Hosted EMU

```yaml
SAML Entity ID:     https://{SUBDOMAIN}.ghe.com/enterprises/{SUBDOMAIN}
SAML ACS URL:       https://{SUBDOMAIN}.ghe.com/enterprises/{SUBDOMAIN}/saml/consume
SAML SSO URL:       https://{SUBDOMAIN}.ghe.com/enterprises/{SUBDOMAIN}/sso
SCIM Tenant URL:    https://api.{SUBDOMAIN}.ghe.com/scim/v2/enterprises/{SUBDOMAIN}
```

---

## ✅ Pre-Flight Checklist

### Before Starting

- [ ] Enterprise slug/subdomain: `__________________`
- [ ] Setup user credentials stored securely
- [ ] Setup user 2FA enabled, with personal 2FA recovery codes AND enterprise recovery codes saved
- [ ] Entra admin with **Application Administrator**, **Cloud Application Administrator**, or **Application Owner** role
- [ ] GitHub **enterprise owner** for the Azure billing connection
- [ ] Azure Subscription ID (for billing): `__________________`
- [ ] Azure **subscription Owner** who can grant tenant-wide consent

### PAT Token Checklist

- [ ] Created as setup user (`SHORTCODE_admin`)
- [ ] Scope: `scim:enterprise`
- [ ] Expiration: No expiration
- [ ] Token stored securely

---

## 👥 Example Initial Group Model

| Group Name | Enterprise Role | Purpose |
|-----------|----------------|---------|
| **GitHub-Enterprise-Owners** | Enterprise Owner | Full admin access |
| **GitHub-Developers** | Member | Standard developer access |
| **GitHub-Security** | Member | Security champions |
| **GitHub-CI-Admins** | Member | CI/CD pipeline admins |

---

## 🎯 Success Criteria

After completing this guide, you should have:

- ✅ EMU enterprise fully configured with Entra ID authentication
- ✅ SCIM provisioning active for automated user lifecycle management
- ✅ Azure subscription connected for metered billing
- ✅ GitHub Copilot enabled and configured
- ✅ Initial organization structure established
- ✅ First users provisioned and able to access GitHub

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


### Q: The setup user activation link does not work — what is wrong?
**A:** The most common cause is an email conflict. If the email address provided for the setup user is already associated as a primary email on another GitHub account, the activation link will silently fail. Find the existing GitHub account using that email, change its primary email to something else, and then retry the activation link. If the link has expired, contact GitHub Support to resend it.

---

### Q: I get a "user already exists" error when SCIM tries to provision a user — how do I fix this?
**A:** This occurs when the user's email or username already exists on GitHub (either as a personal account or in another enterprise). The conflicting account must change its primary email or username before SCIM can provision the managed user. Check the Entra ID provisioning logs for the specific attribute conflict and resolve it on the GitHub side.

---

### Q: SCIM provisioning in Entra ID shows a "quarantined" status — what does that mean?
**A:** Quarantine means Entra ID detected a high error rate during provisioning cycles. Common causes include an expired or invalid SCIM PAT, an incorrect Tenant URL, or rate limiting from assigning too many users at once. Check the provisioning logs in Entra for specific errors. Regenerate the SCIM token if needed, verify the Tenant URL, and restart provisioning. Entra will automatically exit quarantine after a successful cycle.

---

### Q: Should the SCIM attribute mapping use UPN or mail for the username?
**A:** For EMU, the `userName` SCIM attribute determines the GitHub username (with the shortcode suffix appended). Most organizations map `userPrincipalName` to `userName`. However, if your UPNs contain characters GitHub does not allow (such as `#` or `%`), or if UPNs differ significantly from email addresses, consider using `mail` instead. Test with a small pilot group before rolling out broadly.

---

### Q: My SAML Entity ID is correct, but authentication still fails — what should I check?
**A:** A common culprit is a trailing slash in the Entity ID. The Entity ID must be exactly `https://github.com/enterprises/YOUR_ENTERPRISE` with no trailing slash. Also verify the ACS URL and Sign-on URL match exactly. Check that the Base64 certificate you pasted does not contain extra whitespace or line breaks. Finally, confirm you are using the correct Entra Enterprise Application (the EMU gallery app, not the standard org app).

---

### Q: How do I rotate the SCIM token without breaking provisioning?
**A:** Sign in as the setup user (`SHORTCODE_admin`), generate a new PAT with the `scim:enterprise` scope, then immediately update the Secret Token field in the Entra ID provisioning configuration. Click Test Connection to confirm the new token works, then save. The old token is invalidated when you generate a new one, so update Entra promptly to minimize downtime.

---

### Q: The test user can sign in via SSO but lands on an empty enterprise with no orgs — why?
**A:** SSO authentication and SCIM provisioning are separate steps. The user must be both assigned in the Entra Enterprise Application (for SCIM) and assigned to an Entra group that is linked to a GitHub team within an organization. Verify the user appears in the enterprise People tab, and that they have been added to at least one organization via team membership.

---

### Q: The setup user is locked out of 2FA — how do I regain access?
**A:** Use the personal 2FA recovery codes that were saved when 2FA was first enabled on the setup user. If those are lost, use the enterprise SSO recovery codes to access the enterprise and contact GitHub Support to reset the setup user's 2FA. This is why securely storing both sets of recovery codes during initial setup is critical.

</details>

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
| 3 | Managing team memberships with IdP groups | [docs.github.com](https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/provisioning-user-accounts-with-scim/managing-team-memberships-with-identity-provider-groups) |
| 4 | Connecting an Azure subscription | [docs.github.com](https://docs.github.com/en/enterprise-cloud@latest/billing/managing-the-plan-for-your-github-account/connecting-an-azure-subscription) |
| 5 | Copilot policies for enterprise | [docs.github.com](https://docs.github.com/en/enterprise-cloud@latest/copilot/managing-copilot/managing-copilot-for-your-enterprise/managing-policies-and-features-for-copilot-in-your-enterprise) |
| 6 | Microsoft Entra SAML provisioning tutorial | [learn.microsoft.com](https://learn.microsoft.com/en-us/entra/identity/saas-apps/github-enterprise-managed-user-provisioning-tutorial) |

---

*Last updated: July 2026*
