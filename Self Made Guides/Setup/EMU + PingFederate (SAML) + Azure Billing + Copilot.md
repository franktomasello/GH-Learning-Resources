# 🚀 GitHub EMU + PingFederate (SAML), Azure Billing & Copilot Setup Runbook

> **Complete end-to-end runbook for configuring EMU with PingFederate (SAML), Azure billing, and GitHub Copilot**

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [📋 Overview](#-overview)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [1️⃣ Create & Configure the EMU Setup User](#1️⃣-create--configure-the-emu-setup-user)
- [2️⃣ Create the PingFederate SP Connection](#2️⃣-create-the-pingfederate-sp-connection)
- [3️⃣ Enable SAML SSO in GitHub](#3️⃣-enable-saml-sso-in-github)
- [4️⃣ Configure SCIM Provisioning](#4️⃣-configure-scim-provisioning)
- [5️⃣ Attach Azure Subscription for Billing](#5️⃣-attach-azure-subscription-for-billing)
- [6️⃣ Enable GitHub Copilot](#6️⃣-enable-github-copilot)
- [7️⃣ Organization Structure Guidance](#7️⃣-organization-structure-guidance)
- [📝 Quick Reference Worksheet](#-quick-reference-worksheet)
- [✅ Pre-Flight Checklist](#-pre-flight-checklist)
- [👥 Example Initial Group Model](#-example-initial-group-model)
- [🎯 Success Criteria](#-success-criteria)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📚 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **PingFederate:** **Applications** → **Integration** → **SP Connections** → **Create Connection** → **Use a template for this connection** → **GitHub EMU Connector** → import GitHub's metadata (`https://github.com/enterprises/YOUR_ENTERPRISE/saml/metadata`) → check **Browser SSO Profiles** + **Outbound Provisioning** → confirm Entity ID and ACS URL (POST) → **Save** · *PingOne:* **Applications** → **Applications** → **+** → **SAML Application**
- **GitHub:** Sign in as `SHORTCODE_admin` → profile picture → **Enterprise** → **Identity provider** → **Single sign-on configuration** → **Add SAML configuration** → paste the Ping **Sign on URL**, **Issuer**, and **Public Certificate** → **Test SAML configuration** → **Save SAML settings**
- **SCIM:** As `SHORTCODE_admin`, generate a classic PAT with `scim:enterprise` and no expiration → PingFederate: GitHub SP connection → **Outbound Provisioning** → **Configure Provisioning** → **Target**: **Base URL** + **Access Token** → create an **Active** channel · PingOne: **Integrations** → **Provisioning** → **New Connection** → **GitHub EMU** → **Test Connection**
- **Billing (enterprise owner):** Enterprise → Billing and licensing → Payment information → Metered billing via Azure → Add Azure Subscription → Accept permissions → Connect
- **Copilot:** Enterprise → **Billing and licensing** → **Licensing** → Copilot **Manage** → turn on organizations → **AI controls** → **Copilot** (policies) → seats: Org **Settings** → **Copilot** → **Access** → **Start adding seats**, or the enterprise **Manage** page → **Assign licenses**

---

## ✅ Accuracy & Click-Path Notes

<details>
<summary><em>Show click-path conventions</em></summary>


- Reviewed against current public GitHub, Microsoft Entra, and Ping Identity documentation in October 2026 where public documentation is available. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
- When a path starts with `Enterprise`, begin at GitHub, click your profile picture, click `Enterprise` (managed/EMU accounts) — or open the `Enterprises` page at github.com/settings/enterprises (standard accounts) —, select the enterprise, then continue with the listed top tab or left-sidebar item.
- When a path starts with `Organization` or `Org`, begin at GitHub, click your profile picture, click `Organizations`, select the organization, click `Settings`, then continue with the listed sidebar item.
- When a path starts with `Repository`, `Repo`, or a repository name, open the repository, click the `Settings` tab, then continue with the listed sidebar item.
- When a path starts with a vendor portal such as `Microsoft Entra admin center`, `Azure portal`, `Okta Admin Console`, `PingFederate`, `PingOne`, `OneLogin`, `AD FS Management`, `Visual Studio Admin Portal`, or `Azure DevOps`, sign in to that admin portal first, select the tenant, application, or project named in the step, then follow each listed blade, tab, button, and confirmation in order.
- If the expected button is missing, verify you are signed in with the role named in Prerequisites, the feature or license is enabled, and the object is owned by the selected enterprise, organization, or repository. Use page search only to locate the same page, not to skip required confirmation, test, save, or consent clicks.

</details>

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

## ✅ Prerequisites

### Required Items

Before beginning, ensure you have:

| Requirement | Who / Role needed | ✓ |
|-------------|-------------------|---|
| EMU enterprise created — a new enterprise with EMU enabled | GitHub sales / account team | ☐ |
| Setup user — `SHORTCODE_admin` created by GitHub, with password set and 2FA enabled | GitHub **enterprise owner** (setup user) | ☐ |
| PingFederate / PingOne admin access — ability to create SP Connections and configure SAML SSO + SCIM | **PingFederate / PingOne administrator** | ☐ |
| Azure subscription — Subscription ID and someone who can grant tenant-wide admin consent | Azure **subscription Owner** + tenant-wide admin consent | ☐ |
| Ability to connect Azure billing from GitHub | GitHub **enterprise owner** (a billing manager alone is not sufficient) | ☐ |

> 💡 **Note:** SCIM provisioning is required for EMU to manage user lifecycle and account creation.

---

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **PingFederate administrator** | Creates the GitHub EMU SP connection and configures PingFederate outbound SCIM provisioning. | PingFederate administrative console → Applications → Integration → SP Connections → Create Connection → Use a template for this connection → select GitHub EMU Connector → import GitHub EMU metadata → Connection Type → Browser SSO Profiles + Outbound Provisioning → Next → Configure Browser SSO → Configure Assertion Creation → map LDAP adapter and attributes → Save. For SCIM: SP Connections → [GitHub connection] → Connection Type → Outbound Provisioning → Configure Provisioning → Target → Base URL and Access Token → Manage Channel → Create → Source → select data store → Attribute Mapping → Activation & Summary → Active → Done → Save. Handoff: exported metadata, issuer/entity ID, active channel, and SCIM test evidence. |
| **GitHub EMU setup user (`SHORTCODE_admin`)** | Enables SAML in GitHub and provides the SCIM token to the PingFederate admin. | GitHub → profile picture → Enterprise → Identity provider → Single sign-on configuration → Add SAML configuration → paste Single sign-on URL, Issuer, and verification certificate from PingFederate → Test SAML configuration → Save. For SCIM token: setup user → Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token (classic) → `scim:enterprise` → Generate token. Handoff: Tenant URL, SCIM token, SAML test success, and recovery codes. |
| **LDAP or identity directory owner** | Controls the users and groups PingFederate can provision. | PingFederate administrative console → System → Data & Credential Stores → Data Stores → [LDAP data store] → verify connection, then Applications → Integration → SP Connections → [GitHub connection] → Outbound Provisioning → Source and Source Location → configure LDAP search base and filters → Save. Handoff: source filter, pilot users, and role attribute mapping. |
| **GitHub enterprise owner** (billing manager alone is not sufficient to connect a subscription) | Starts the Azure metered billing connection from GitHub. | Enterprise path: GitHub → profile picture → Enterprise → Billing and licensing → Payment information → Metered billing via Azure → Add Azure Subscription. Organization path (org owner): GitHub → profile picture → Organizations → [organization] → Settings → Billing and licensing → Payment information → Metered billing via Azure → Add Azure Subscription. Then sign in to Microsoft → Permissions requested → Accept → Select a subscription → Connect. Handoff: the subscription ID is visible on Payment information. |
| **Azure subscription Owner** | Provides the Azure subscription that GitHub will bill against, or grants another signer the required Azure RBAC rights. | Azure portal → Subscriptions → [subscription] → Access control (IAM) → Role assignments → confirm the signer is listed under Owner. To grant access: Add → Add role assignment → Privileged administrator roles → Owner → Members → Select members → [user] → Select → Review + assign. Handoff: subscription ID and tenant ID. |
| **Microsoft Entra Global Administrator or consent approver** | Approves tenant-wide consent when the Microsoft consent prompt blocks the GitHub billing app. | Microsoft Entra admin center → Entra ID → Enterprise apps → Activity → Admin consent requests → My Pending → [GitHub request] → Review permissions and consent → Approve. If the Global Administrator completes the GitHub flow directly, approve the Permissions requested prompt by clicking Accept. |

---

## 1️⃣ Create & Configure the EMU Setup User

**👤 Role:** GitHub EMU **setup user** (`SHORTCODE_admin`) · **📍 Portal:** GitHub

### Process

1. GitHub emails an invite to set the password for SHORTCODE_admin
2. In a private/incognito window:
   - Set password (store it in your password vault)
   - **Enable 2FA:**
     1. Click your profile picture → **Settings** → **Password and authentication**.
     2. Under **Two-factor authentication**, click **Enable two-factor authentication**.
     3. Choose a method (**Set up using an app** / TOTP recommended), scan the QR code in your authenticator app, then enter the 6-digit code and click **Continue** / **Verify** to complete the challenge.
     4. On the recovery-codes screen click **Download** (or **Copy** / **Print**) to save your **personal 2FA recovery codes**, then click **I have saved my recovery codes**.
   - **Download the enterprise recovery codes** (separate from your personal 2FA codes): profile picture → **Enterprise** → **Identity provider** → **Single sign-on configuration** → under **SAML single sign-on**, click **Save your recovery codes** → **Download** (or **Print** / **Copy**). Store both sets securely. *(If the link isn't shown yet, you'll be prompted to save them when you save your SAML configuration in Step 3.)*

### Important Notes

> ⚠️ **Setup User Purpose:** This account is primarily for SCIM provisioning via token and recovery scenarios. Day-to-day enterprise administration should be done with provisioned managed user accounts.

> 🔐 **Sign-in requirement (Jan 2025):** Every future sign-in as `SHORTCODE_admin` requires a successful 2FA challenge OR an **enterprise SSO recovery code**. Losing both the personal 2FA recovery codes and the enterprise recovery codes locks you out. The setup user's password cannot be reset by the normal email flow — reset must go through **GitHub Support**.

> 🚨 **Email Conflict:** If the provided email address is already associated as a primary email with an existing GitHub account, the activation link will not work. Modify the existing account's primary email first.

---

## 2️⃣ Create the PingFederate SP Connection

Choose the option that matches your Ping deployment, and use the GitHub values in **SAML Configuration Values** below.

> 💡 **Tip:** Create the SCIM token first (Step 4A — it only needs the setup user). Then you can finish SSO and provisioning in one pass, the way Ping's own GitHub EMU procedure does.

### Option A — PingFederate (Self-Managed)

**👤 Role:** **PingFederate administrator** · **📍 Portal:** PingFederate administrative console

> 📌 **Prerequisite:** install Ping's **GitHub EMU Provisioner** (PingFederate 9.0 or later). It adds the **GitHub EMU Connector** connection template used below, and one SP connection then handles both SSO and provisioning.

**Navigate:** **Applications** → **Integration** → **SP Connections** → **Create Connection**

1. On the **Connection Template** tab, select **Use a template for this connection**, choose **GitHub EMU Connector** from the **Connection Template** list (not "GitHub Connector"), and import GitHub's SP metadata — from `https://github.com/enterprises/YOUR_ENTERPRISE/saml/metadata` (GHE.com: `https://SUBDOMAIN.ghe.com/enterprises/SUBDOMAIN/saml/metadata`) or a saved copy of that file. Click **Next**.
2. On the **Connection Type** tab, make sure both **Browser SSO Profiles** and **Outbound Provisioning** are checked. Click **Next**.
3. On **Connection Options**, keep **Browser SSO** selected and click **Next**.
4. On **General Info**, confirm **Partner's Entity ID (Connection ID)** is `https://github.com/enterprises/YOUR_ENTERPRISE` (no trailing slash), enter a **Connection Name** such as `GitHub Enterprise Managed User`, and click **Next**.
5. On **Browser SSO**, click **Configure Browser SSO**, then work through its tabs, clicking **Next** on each:
   - **SAML Profiles** — select **SP-Initiated SSO** (optionally also **IdP-Initiated SSO**).
   - **Assertion Creation** → **Configure Assertion Creation** — set **SAML_SUBJECT** to a stable, persistent identifier that matches the SCIM `userName` you'll send in Step 4B, and map your authentication source (adapter or policy) to fulfill it.
     *(GitHub requires only a persistent NameID — the SAML_SUBJECT. Extra name or email claims aren't needed for EMU, because SCIM supplies profile data.)*
   - **Protocol Settings** → **Configure Protocol Settings** — confirm the **Assertion Consumer Service URL** `https://github.com/enterprises/YOUR_ENTERPRISE/saml/consume` with **POST** binding. Under **Signature Policy**, sign the assertion or the whole response (GitHub accepts either).
   - Click **Done** to return to the connection.
6. On **Credentials**, click **Configure Credentials** → **Digital Signature Settings**, select your signing certificate, choose **RSA SHA256**, then click **Next** → **Done**.
7. On **Outbound Provisioning**: if you already have the SCIM token, configure it now using Step 4B; otherwise continue and finish it in Step 4B.
8. On **Activation & Summary**, set **Connection Status** to **Active** and click **Save**.

### Option B — PingOne (Cloud)

**👤 Role:** **PingOne administrator** · **📍 Portal:** PingOne admin console

**Navigate:** **Applications** → **Applications** → **+**

1. Enter an application name (for example, `GitHub Enterprise Managed User`) and an optional description.
2. Select **SAML Application**, then click **Configure**.
3. Give PingOne GitHub's SP details with one of these options: **Import from URL** (`https://github.com/enterprises/YOUR_ENTERPRISE/saml/metadata`), **Import Metadata** (upload the file), or enter the **ACS URLs** (`https://github.com/enterprises/YOUR_ENTERPRISE/saml/consume`) and **Entity ID** (`https://github.com/enterprises/YOUR_ENTERPRISE`) manually.
4. Click **Save**.
5. Turn on the application's enable toggle.

> 💡 **Note:** In PingOne, the GitHub EMU *provisioning* connection is separate from this SAML application — you'll create it in Step 4B (Alt).

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

### Get the IdP Values for GitHub

You need three values for Step 3: the IdP **Sign on URL**, the **Issuer** (IdP entity ID), and the **signing certificate**.

#### PingFederate (Self-Managed)

**Navigate:** **System** → **Protocol Metadata** → **Metadata Export** *(PingFederate 10.1 or later)*

1. Export PingFederate's IdP metadata — or, on the **SP Connections** list, use **Export Metadata** for the GitHub connection.
2. In the metadata, note the **entityID** (your **Issuer**) and the **SingleSignOnService** location — typically `https://<PING_HOST>/idp/SSO.saml2` (your **Sign on URL**).
3. Export the signing certificate: **Security** → **Signing & Decryption Keys & Certificates** → find the certificate → **Select Action** → **Export** → **Certificate Only** → **Next** → **Export**. You'll paste its PEM contents into GitHub as the **Public Certificate**.

#### PingOne (Cloud)

**Navigate:** **Applications** → **Applications** → *[GitHub Enterprise Managed User]* → **Configuration** tab

1. In the connection details, copy the **Issuer ID** and the single sign-on service URL.
2. Download the signing certificate — or download the IdP metadata, which contains all three values.

### Assign Users (for SSO Testing)

#### PingFederate (Self-Managed)

Who can sign in is decided by your authentication policy and the directory behind it. Make sure your pilot users exist in the directory (LDAP/AD) your adapter or policy authenticates against, and that no access policy blocks them.

#### PingOne (Cloud)

**Navigate:** **Applications** → **Applications** → *[GitHub Enterprise Managed User]* → **Access** tab

1. Add the group(s) whose members should be able to sign in.
2. Click **Save**.

> 💡 **Note:** In PingOne, application access is controlled by group membership.

---

## 3️⃣ Enable SAML SSO in GitHub

> ⚠️ **Warning:** Enabling SAML impacts how members authenticate. Enterprise Managed Users does not provide a backup username/password sign-in URL, and there is **no "Require SAML authentication" checkbox**; if SAML fails, use enterprise SSO recovery codes or contact GitHub Enterprise Support.

**👤 Role:** EMU setup user (`SHORTCODE_admin`, an enterprise owner) · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Enterprise** → **Identity provider** → **Single sign-on configuration**

### Configuration Steps

1. Under **SAML single sign-on**, click **Add SAML configuration**.
2. Enter the following values from PingFederate / PingOne:
   - **Sign on URL:** Paste the IdP SSO Service URL from PingFederate.
   - **Issuer:** Paste the Entity ID (IdP Issuer) from PingFederate.
   - **Public Certificate:** Paste the contents of the PingFederate signing certificate (X.509).
   - **Signature Method:** Choose from the dropdown (SHA-256 recommended).
   - **Digest Method:** Choose from the dropdown (SHA-256 recommended).
3. Click **Test SAML configuration** to validate the setup (it must pass before you can save).
4. Click **Save SAML settings**.
5. Immediately **Download**, **Print**, or **Copy** your enterprise **SSO recovery codes** and store them securely.

> 🔐 **Critical:** Recovery codes are essential for break-glass scenarios if your IdP becomes unavailable. EMU has no backup username/password sign-in — these codes are your only fallback.

---

## 4️⃣ Configure SCIM Provisioning

SCIM handles user creation and deactivation in EMU. This requires creating a token in GitHub and configuring Outbound Provisioning in PingFederate.

### 4A — Create the SCIM Token in GitHub

The token must be created as the setup user with specific requirements.

**👤 Role:** EMU setup user (`SHORTCODE_admin`, an enterprise owner) · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)** → **Generate new token (classic)**

**Token Requirements:**
- ✓ Type: **personal access token (classic)**
- ✓ Scope: `scim:enterprise` (only)
- ✓ Expiration: **No expiration** (GitHub recommends none; if it expires, provisioning stops)
- ✓ Created by: `SHORTCODE_admin`

**Creation Steps:**

1. In **Note**, enter a descriptive name (e.g., "PingFederate SCIM Provisioning").
2. Set **Expiration** to **No expiration**.
3. Under scopes, select `scim:enterprise` only.
4. Click **Generate token**.

> 🔑 **Important:** Copy the token immediately after generation. You won't be able to see it again.

### 4B — Configure SCIM in PingFederate (Self-Managed)

**👤 Role:** **PingFederate administrator** · **📍 Portal:** PingFederate administrative console

**Navigate:** **Applications** → **Integration** → **SP Connections** → *[GitHub EMU connection]* → **Outbound Provisioning** → **Configure Provisioning**

1. On the **Target** tab, enter:
   - **Base URL** — GitHub.com: `https://api.github.com/scim/v2/enterprises/{ENTERPRISE_SLUG}` · GHE.com: `https://api.{SUBDOMAIN}.ghe.com/scim/v2/enterprises/{SUBDOMAIN}`
   - **Access Token** — the SCIM token from Step 4A
2. Confirm **User Create**, **User Update**, and **User Disable / Delete** are selected, and set **Remove User Action** to **Disable**. Click **Next**.
3. On **Manage Channels**, click **Create**, then work through the channel tabs, clicking **Next** on each:
   - **Channel Info** — enter a channel name.
   - **Source** — select the LDAP data store that holds your users.
   - **Source Settings** — review the defaults.
   - **Source Location** — enter the base DN and a group DN or filter that scopes who is provisioned.
   - **Attribute Mapping** — map the attributes in 4C, including **Roles**.
   - **Activation & Summary** — set **Channel Status** to **Active**.
4. Click **Done**, then click **Save** on the connection.

> 📌 **Order matters:** GitHub doesn't accept provisioning until SAML SSO is configured (Step 3). Set the channel to **Active** only after SSO works.

> 💡 **Sync interval:** PingFederate checks for changes every 60 seconds by default — adjust it at **System** → **Server** → **Protocol Settings** → **Outbound Provisioning**.

### 4B (Alt) — Configure SCIM in PingOne (Cloud)

**👤 Role:** **PingOne administrator** · **📍 Portal:** PingOne admin console

**Navigate:** **Integrations** → **Provisioning**

1. Click **+**, then **New Connection**.
2. On the **Identity Store** line, click **Select**.
3. On the **GitHub EMU** tile, click **Select**, then click **Next**.
4. Enter a name and description for the connection, then click **Next**.
5. Enter the **Base URL** (same values as 4B) and the **Access Token** (the SCIM token from 4A), then click **Test Connection**.
6. Set your preferences: **Group Membership Handling** (**Merge** or **Overwrite**), and turn on user creation, updates, disabling, and deprovisioning — set the removal action to **Disable** (recommended).
7. Click **Save**.
8. Turn on the toggle at the top of the connection's details panel to enable it.
9. Create a provisioning rule that uses this connection and scopes which users and groups are provisioned (see Ping's "Creating a provisioning rule"), then confirm the attribute mappings in 4C.

### 4C — Configure Attribute Mappings

**👤 Role:** **PingFederate administrator** or **PingOne administrator** · **📍 Portal:** PingFederate administrative console / PingOne admin console

Map these GitHub SCIM attributes from your directory — in PingFederate on the channel's **Attribute Mapping** tab, in PingOne on the provisioning rule's attribute mappings (PingOne requires **Username**, **Email**, and **External ID**).

| GitHub SCIM Attribute | Typical source attribute | Description |
|-----------------------|--------------------------|-------------|
| `userName` | User's unique identifier (e.g., `sAMAccountName` or `uid`) | Must be unique across the enterprise and match the SAML_SUBJECT |
| `name.givenName` | `givenName` / `firstName` | User's first name |
| `name.familyName` | `sn` / `lastName` | User's last name |
| `emails[type eq "work"].value` | `mail` / `email` | User's email address |
| `displayName` | `displayName` / `cn` | User's full display name |
| `externalId` | Unique directory ID (e.g., `objectGUID` or `entryUUID`) | Persistent unique identifier from the IdP |
| `roles` | An attribute (PingFederate: the **Roles** mapping) whose value is `enterprise_owner`, `billing_manager`, `user`, or `guest_collaborator` | Sets the enterprise role — give at least one person `enterprise_owner` |

> ⚠️ **Identity linking:** the SAML_SUBJECT in your SSO assertion (Step 2) must match the SCIM `userName`, or people can't sign in to the account SCIM created for them.

### 4D — Assign Users/Groups for Provisioning

- **PingFederate:** the channel's **Source Location** (base DN plus group or filter) decides who is provisioned — add people to that group or adjust the filter.
- **PingOne:** the provisioning rule's group or filter scope decides who is provisioned.

**Provisioning Notes:**

> 📌 **Important Constraints:**
> - To avoid exceeding GitHub's rate limit, **do not assign more than 1,000 users per hour** to the SCIM integration.

---

## 5️⃣ Attach Azure Subscription for Billing

Connect your Azure subscription so GitHub usage (Copilot, Actions, Codespaces, etc.) is billed through Azure.

**👤 Role:** GitHub **enterprise owner** (on the Azure side, a subscription **Owner** who can grant tenant-wide admin consent) · **📍 Portal:** GitHub → Microsoft

**Navigate:** Profile picture → **Enterprise** → **Billing and licensing** → **Payment information** → scroll to **Metered billing via Azure** → **Add Azure Subscription**

### Prerequisites

- ✓ GitHub **enterprise owner** (an "owner of the enterprise"; a billing manager alone **cannot** connect a subscription)
- ✓ Azure **subscription Owner** on the target subscription
- ✓ Azure Subscription ID
- ✓ A user who can provide **tenant-wide admin consent** (or a Microsoft Entra **Global Administrator** / admin consent workflow)

### Configuration Steps

1. Click **Add Azure Subscription**.
2. Sign in to your Microsoft account (if prompted).
3. Review the **Permissions requested** prompt.
4. Click **Accept**.
5. Under **Select a subscription**, choose your Azure subscription.
6. Check the confirmation box: "By clicking 'Connect', you are confirming…".
7. Click **Connect**.

> ✅ **Result:** The connected subscription ID appears on the **Payment information** page.

> 💡 **Admin Consent:** If you don't see a **Permissions requested** prompt and instead see a message about needing admin approval, configure an admin consent workflow in Azure or work with your Microsoft Entra **Global Administrator**.

---

## 6️⃣ Enable GitHub Copilot

Set up Copilot in this order: **6A** turn Copilot on for organizations, **6B** set policies, **6C** give people seats. With Azure metered billing connected (Step 5), Copilot charges bill to your Azure subscription.

> 📌 **Where things live:** organization access and licenses are under **Billing and licensing → Licensing**; policies are under **AI controls**. Selections on these pages apply immediately — there is **no Save button**.

### 6A — Turn Copilot on for organizations

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Enterprise** → **Billing and licensing** → **Licensing** → **Manage** *(in the "Copilot" section)*

1. At the top of the enterprise page, click **Billing and licensing**.
2. In the "Billing and licensing" sidebar, click **Licensing**.
3. In the "Copilot" section, click **Manage**.
4. Next to **Organization access**, open the dropdown and choose whether to enable Copilot for **all organizations** or to **Allow for specific organizations**.
5. If you chose **Allow for specific organizations**:
   1. Click the **Organizations** tab.
   2. Find the organization.
   3. To the right of its name, open the **Copilot** dropdown and click **Enabled** (Copilot Business plan) — or **Copilot: Enterprise** / **Copilot: Business** if your enterprise has a Copilot Enterprise plan.
6. Confirm the organization now shows Copilot as enabled. *(The selection applies immediately — there is no Save button.)*

> ⚠️ **Do this first:** until Copilot is enabled for an organization here, its owners can't assign seats in 6C (Route 1).

### 6B — Set Copilot policies

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Enterprise** → **AI controls** → **Copilot** *(sidebar)*

1. At the top of the enterprise page, click **AI controls** *(a top-of-page tab — not under **Settings**)*.
2. In the sidebar, open the page that holds the policies you want:
   - **Copilot** — administration, privacy, model, billing, and usage policies, including **Policies for enterprise-assigned users** (required before Route 2 in 6C).
   - **Copilot** → under "Features & clients", click **Configure features & clients** — feature and client policies such as Copilot on GitHub.com, Copilot Chat in the IDE, and Copilot in the CLI.
   - **Agents** — AI agent policies, such as **Copilot cloud agent** (formerly Copilot coding agent).
   - **MCP** — Model Context Protocol (MCP) policies.
3. Set each policy:
   - **Dropdown:** open it and choose an enforcement option — **Enabled**, **Disabled**, or **Let organizations decide**.
   - **Toggle:** click it.
   - **No visible control:** click the policy name to see its options.
4. Check that each policy shows the value you chose. *(Changes apply on selection — there is no Save button.)*

> ⏰ **Before October 22, 2026:** decide the **Default policy for new features** on this same **Copilot** page (**Enabled**, **Disabled**, or **Let organizations decide**). From that date, GA features you've left **Unconfigured** follow it — and it's **Enabled** by default. Explicit choices are never overridden.

> 💡 **Suggestions matching public code:** agree on this setting with your legal team before you enable it.

### 6C — Assign Copilot seats

Use either route — or both. A person assigned through more than one route still uses **one** license (the highest tier).

**Route 1 — Organization seats** (organization owner)

**👤 Role:** GitHub **organization owner** · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Organizations** → *[organization]* → **Settings** → **Copilot** *(sidebar, under "Code, planning, and automation")* → **Access**

1. If you see **Allow this organization to assign seats**, click it.
2. Click **Start adding seats**.
3. Choose who gets Copilot:
   - **Everyone:** select **Purchase for all members**, then in the "Confirm seats purchase for all members" dialog click **Purchase seats**.
   - **Specific people or teams:** select **Purchase for selected members**. In the "Enable Copilot access for users and teams" dialog, use the **Users and teams** tab to search for and add people or teams (or **Upload CSV** to add many at once), then click **Continue to purchase** → **Purchase seats**.

> 💡 **Hands-off seats:** give seats to a team that's linked to an PingFederate group (see Step 7). When PingFederate adds someone to the group, SCIM adds them to the team and they get a Copilot seat automatically.

> 💡 **Billing:** a seat is billed from the moment it's granted (prorated mid-cycle), whether or not the person uses Copilot yet.

**Route 2 — Enterprise licenses** (enterprise owner · Copilot Business · no organization membership required)

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Enterprise** → **Billing and licensing** → **Licensing** → **Manage** *(in the "Copilot" section)*

**Before you start:** set the **Policies for enterprise-assigned users** policy (6B), make sure the people already exist in the enterprise (provisioned by SCIM), and create the enterprise team first if you're licensing a team.

1. Click the **All members** tab (individual users) or the **Enterprise Teams** tab.
2. Click **Assign licenses**.
3. Search for the users or enterprise teams, then click **Add licenses**.

> ✅ **Enterprise teams are generally available** (since June 2026). License an enterprise team and people gain or lose Copilot as they join or leave it. With Enterprise Managed Users you can sync the enterprise team to an PingFederate group, so licensing is driven entirely from your IdP.

> 💡 **When to use this route:** people who need Copilot but no organization access. Enterprise members who aren't in any organization usually don't consume a GitHub Enterprise Cloud license. Direct enterprise assignment is for **Copilot Business**.

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
- GitHub **enterprise owner** available to connect Azure billing
- Azure Subscription ID (for billing): __________________
- Azure **subscription Owner** + a user who can grant tenant-wide admin consent

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
**A:** Generate a new signing certificate in PingFederate (**Security → Signing & Decryption Keys & Certificates**). Export the new certificate in X.509/PEM format. Before activating it in PingFederate, update the certificate in GitHub using the EMU identity path: Profile picture → **Enterprise** → **Identity provider** → **Single sign-on configuration** → edit the **SAML single sign-on** configuration → paste the new **Public Certificate** → **Test SAML configuration** → **Save SAML settings**. Once GitHub has the new cert, activate it as the primary signing certificate in PingFederate. This avoids a window where the certs are mismatched.

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

## 📚 Resources

- [Configuring SAML SSO for Enterprise Managed Users](https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/configuring-authentication-for-enterprise-managed-users/configuring-saml-single-sign-on-for-enterprise-managed-users)
- [Configuring SCIM Provisioning for Enterprise Managed Users](https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/provisioning-user-accounts-with-scim/configuring-scim-provisioning-for-users)
- [PingIdentity GitHub EMU Integration Guide](https://docs.pingidentity.com/integrations/github/github_emu_provisioner/pf_github_emu_connector.html)

---

*Last updated: October 2026*
