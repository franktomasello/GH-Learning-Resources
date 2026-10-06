# 🚀 GitHub Standard GHEC + PingFederate (SAML), Azure Billing & Copilot Setup Runbook

> **Complete end-to-end runbook for configuring Standard (non-EMU) GHEC with PingFederate/PingOne (SAML), SCIM org provisioning, Azure billing, and GitHub Copilot**

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [📋 Overview](#-overview)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [1️⃣ Create & Secure the "SCIM Setup User" (Standard non-EMU)](#1-create--secure-the-scim-setup-user-standard-non-emu)
- [2️⃣ Create the PingFederate SP Connection (or PingOne Application)](#2-create-the-pingfederate-sp-connection-or-pingone-application)
- [3️⃣ Enable SAML SSO in GitHub](#3-enable-saml-sso-in-github)
- [4️⃣ Enforce SAML SSO for the Organization (Required)](#4-enforce-saml-sso-for-the-organization-required)
- [5️⃣ Configure SCIM Provisioning (PingFederate → GitHub Organization)](#5-configure-scim-provisioning-pingfederate--github-organization)
- [6️⃣ Attach Azure Subscription for Metered Billing](#6-attach-azure-subscription-for-metered-billing)
- [7️⃣ Enable GitHub Copilot (Enterprise + Organization)](#7-enable-github-copilot-enterprise--organization)
- [8️⃣ Critical Post-Enablement: SSO Authorization for Credentials (Required)](#8-critical-post-enablement-sso-authorization-for-credentials-required)
- [✅ Pre-Flight / Validation Checklist](#-pre-flight--validation-checklist)
- [🎯 Success Criteria](#-success-criteria)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **PingFederate:** **Applications** → **Integration** → **SP Connections** → **Create Connection** → **Browser SSO Profiles** (SAML 2.0) → Entity ID `https://github.com/orgs/YOUR_ORG` → ACS URL with POST binding · *PingOne:* **Applications** → **Applications** → **+** → **SAML Application**
- **GitHub SAML:** Org Settings → Authentication security → SAML single sign-on → Paste PingFederate SSO URL, Issuer, X.509 cert → Test → Save → Enforce
- **SCIM (custom, unsupported for Ping):** As setup user, generate PAT (classic) with `admin:org` scope → SSO-authorize for org → point a SCIM client at `https://api.github.com/scim/v2/organizations/YOUR_ORG` with the PAT as bearer token. See the 🚨 note in Step 5 — PingFederate is **not** an officially supported org-SCIM IdP.
- **Billing:** Org/Enterprise → Billing and licensing → Payment information → Add Azure Subscription → Accept → Connect
- **Copilot:** Enterprise → **Billing and licensing** → **Licensing** → Copilot **Manage** → turn on the org → **AI controls** → **Copilot** (enterprise policies) → Org **Settings** → **Copilot** → **Policies** / **Models** → **Access** → **Start adding seats** (or the enterprise **Manage** page → **Assign licenses**)

---

## ✅ Accuracy & Click-Path Notes

<details>
<summary><em>Show click-path conventions</em></summary>


- Reviewed against current public GitHub and Microsoft documentation in October 2026 where public documentation is available. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
- When a path starts with `Enterprise`, begin at GitHub, click your profile picture, click `Enterprise` (managed/EMU accounts) — or open the `Enterprises` page at github.com/settings/enterprises (standard accounts) —, select the enterprise, then continue with the listed top tab or left-sidebar item.
- When a path starts with `Organization` or `Org`, begin at GitHub, click your profile picture, click `Organizations`, select the organization, click `Settings`, then continue with the listed sidebar item.
- When a path starts with `Repository`, `Repo`, or a repository name, open the repository, click the `Settings` tab, then continue with the listed sidebar item.
- When a path starts with a vendor portal such as `Microsoft Entra admin center`, `Azure portal`, `Okta Admin Console`, `PingFederate`, `PingOne`, `OneLogin`, `AD FS Management`, `Visual Studio Admin Portal`, or `Azure DevOps`, sign in to that admin portal first, select the tenant, application, or project named in the step, then follow each listed blade, tab, button, and confirmation in order.
- If the expected button is missing, verify you are signed in with the role named in Prerequisites, the feature or license is enabled, and the object is owned by the selected enterprise, organization, or repository. Use page search only to locate the same page, not to skip required confirmation, test, save, or consent clicks.

</details>

---

## 📋 Overview

This guide walks through setting up **Standard (non-EMU) GitHub Enterprise Cloud (GHEC)**, including:

- **PingFederate / PingOne (SAML SSO)** for authentication
- **Enforce SAML SSO** for the organization (and enterprise, if applicable)
- **SCIM provisioning** (custom SCIM client → GitHub org membership lifecycle). ⚠️ GitHub's officially supported org-SCIM identity providers are **Microsoft Entra ID, Okta, and OneLogin** only; **PingFederate is not a supported org-SCIM IdP**. Ping-driven provisioning here means calling the org SCIM REST API as a custom client — see Step 5.
- **Azure subscription** attachment for metered billing
- **GitHub Copilot** enablement + policy controls
- **Critical post-enablement** items (PAT/SSH SSO authorization)

### Standard (non-EMU) means:

- Users keep their regular GitHub.com accounts
- SAML SSO is enforced at the org (and optionally at the enterprise, if you have one)
- SCIM manages organization membership via PingFederate assignments (not "managed user accounts"—that is EMU)

## ✅ Prerequisites

| Requirement | Owner / Role | Notes |
| --- | --- | --- |
| GitHub Enterprise Cloud (Standard / non-EMU) org | Org Owner | You must be able to access Org Settings → Authentication security |
| (If applicable) GitHub Enterprise account | Enterprise Owner | Needed only if you will configure Enterprise SAML and/or Enterprise Copilot policies |
| PingFederate or PingOne Admin access | Ping Admin | Must be able to create SP Connections (PingFederate) or Applications (PingOne) |
| Pilot users/groups in PingFederate/PingOne | IAM team | Use a small pilot cohort first to reduce risk |
| Dedicated GitHub "SCIM setup user" | Org Owner | GitHub recommends a dedicated user to own the SCIM token for provisioning |
| Azure subscription + ability to consent | Azure admin | Needed to connect metered billing via Azure. If subscription is in a different tenant, you may need to specify a different tenant ID during connection |
| Copilot plan decision | Enterprise/Org owner | Copilot Business vs Copilot Enterprise |

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **PingFederate or PingOne administrator** | Creates the SAML application/SP connection and configures provisioning to the GitHub organization. | PingFederate: Administrative Console → Applications → SP Connections → Create Connection or Use a template → Browser SSO → configure Entity ID, ACS/Reply URL, Sign-on URL, signing certificate, assertion attributes, and activation → Save. Then SP Connections → [connection] → Connection Type → Outbound Provisioning → Configure Provisioning → Target → SCIM Base URL and Bearer token → Attribute Mapping → Activation & Summary → Active → Save. PingOne: Admin Console → Connections → Applications → Add Application → SAML Application → configure ACS URL, Entity ID, SSO URL, certificate, and attribute mappings → Save, then Provisioning or Access → assign users/groups. Handoff: SAML values, active provisioning channel, and assigned pilot group. |
| **GitHub organization owner or dedicated SCIM setup user** | Enables GitHub org SAML and provides the PAT the SCIM client uses. | GitHub → profile picture → Organizations → [org] → Settings → Authentication security → SAML single sign-on → Enable SAML authentication → paste Ping SSO URL, Issuer, and Public Certificate → Test SAML configuration → Save → visit `https://github.com/orgs/ORG/sso` → complete SSO → setup user → Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token with `admin:org` → Configure SSO → Authorize for the org. Note: standard GHEC has **no** org-level "SCIM provisioning → Generate token" or tenant-URL page (that is an EMU/enterprise feature); the org SCIM base URL is the fixed value `https://api.github.com/scim/v2/organizations/ORG`. Handoff: SAML test success, SCIM base URL, SSO-authorized PAT, and recovery codes. |
| **Identity directory owner** | Maintains the source group or LDAP filter used for GitHub membership. | PingFederate administrative console → Applications → Integration → SP Connections → [GitHub connection] → Outbound Provisioning → Source and Source Location → configure the LDAP source/filter, or PingOne Admin Console → Directory → Groups → [group] → add users → assign group to application. Handoff: group/filter name and pilot members. |
| **GitHub enterprise or organization owner** | Starts the Azure metered billing connection from GitHub. | Enterprise path: GitHub → Enterprises page (github.com/settings/enterprises) → [enterprise] → Billing and licensing → Payment information → Metered billing via Azure → Add Azure Subscription. Organization path: GitHub → profile picture → Organizations → [organization] → Settings → Billing and licensing → Payment information → Metered billing via Azure → Add Azure Subscription. Then sign in to Microsoft → Permissions requested → Accept → Select a subscription → Connect. Handoff: the subscription ID is visible on Payment information. |
| **Azure subscription Owner** | Provides the Azure subscription that GitHub will bill against, or grants another signer the required Azure RBAC rights. | Azure portal → Subscriptions → [subscription] → Access control (IAM) → Role assignments → confirm the signer is listed under Owner. To grant access: Add → Add role assignment → Privileged administrator roles → Owner → Members → Select members → [user] → Select → Review + assign. Handoff: subscription ID and tenant ID. |
| **Microsoft Entra Global Administrator or consent approver** | Approves tenant-wide consent when the Microsoft consent prompt blocks the GitHub billing app. | Microsoft Entra admin center → Entra ID → Enterprise apps → Activity → Admin consent requests → My Pending → [GitHub request] → Review permissions and consent → Approve. If the Global Administrator completes the GitHub flow directly, approve the Permissions requested prompt by clicking Accept. |

---

## 1️⃣ Create & Secure the "SCIM Setup User" (Standard non-EMU)

**👤 Role:** GitHub org owner · **📍 Portal:** GitHub

> ⚠️ **Important:** This user is required because org SCIM acts on behalf of a specific GitHub user via a PAT (Personal Access Token). If that user loses access or leaves the org, SCIM can stop working—so you want a stable, dedicated identity.

### Process

1. Create a dedicated GitHub.com account (example: `gh-ping-scim@yourdomain.com`)
2. **Secure the account:**
    - Enable 2FA
    - Ensure recovery methods are stored per your approved process (vault / break-glass)
3. **Add the setup user to your GitHub organization as an Owner:**
    - GitHub → your Organization → Settings → People → Invite member
    - After accepted → set role to Owner
4. **Create a PAT (classic) with `admin:org` scope** for this user — you will need it for the SCIM client configuration in Step 5. Set expiration per policy, **copy the token immediately (it is shown only once)**, store it in your vault, then **SSO-authorize it for the org** (see Step 8B) so SCIM/API calls do not return 403.

### Important Notes

> 📌 **Note:** This setup user will consume a GitHub license. Treat it as a system account: minimal use outside of IAM configuration.

## 2️⃣ Create the PingFederate SP Connection (or PingOne Application)

**👤 Role:** PingFederate/PingOne administrator · **📍 Portal:** PingFederate / PingOne

### Option A: PingFederate (On-Prem / Standalone)

**Navigate:** PingFederate administrative console → **Applications** → **Integration** → **SP Connections** → **Create Connection** *(PingFederate 9.x: **Identity Provider** → **SP Connections** → **Create New**)*

**Configuration Steps**

1. **Connection Template:** select **Do not use a template for this connection**, then click **Next**.
2. **Connection Type:** select **Browser SSO Profiles** with protocol **SAML 2.0**, then click **Next**.
3. **Connection Name:** `GitHub Enterprise Cloud - YOUR_ORG` (or a descriptive label)
4. **Partner's Entity ID (SP Entity ID):**
    ```
    https://github.com/orgs/YOUR_ORG
    ```
5. **Assertion Consumer Service (ACS) URL:**
    ```
    https://github.com/orgs/YOUR_ORG/saml/consume
    ```
    - Binding: **POST**
6. **Browser SSO → SAML Profiles:** enable **SP-Initiated SSO**
7. **Assertion Lifetime:** configure per your organization's session policy
8. **Attribute Contract / Attribute Mapping:**
    - Map the `NameID` (Subject) to a unique, stable user identifier (e.g., email address or UPN)
    - Ensure the attribute source is configured correctly for your user directory (LDAP, Active Directory, etc.)
9. **Signing Configuration:**
    - Select or create a signing certificate
    - Sign the assertion or the whole response with **RSA SHA256** — GitHub requires one of them to be signed
10. On **Activation & Summary**, set **Connection Status** to **Active**, then click **Save**.

### Option B: PingOne (Cloud)

**Navigate:** PingOne admin console → **Applications** → **Applications** → **+**

**Configuration Steps**

1. Enter an application name (for example, `GitHub Enterprise Cloud - YOUR_ORG`), select **SAML Application**, and click **Configure**.
    - ⚠️ Do NOT use a GitHub EMU template or connection (that is for EMU enterprises)
2. Enter the SAML values manually (or **Import from URL** `https://github.com/orgs/YOUR_ORG/saml/metadata`):
    - **ACS URL:** `https://github.com/orgs/YOUR_ORG/saml/consume`
    - **Entity ID:** `https://github.com/orgs/YOUR_ORG`
    - **Sign-on URL:** `https://github.com/orgs/YOUR_ORG/sso`
3. **Attribute Mappings:**
    - Map `saml_subject` (NameID) to a unique user identifier (e.g., email)
4. Click **Save**, then toggle the application to **Enabled**.

### Gather Required IdP Values (Both PingFederate and PingOne)

From your PingFederate SP Connection or PingOne Application configuration, capture:

1. **IdP SSO Service URL** (Single Sign-On Service URL)
2. **IdP Entity ID** (Issuer)
3. **X.509 Signing Certificate** (public certificate, Base64-encoded)

> 💡 **Tip:** **PingFederate:** These values are available under SP Connection, Protocol Settings, Export Metadata, or from the server's federation metadata endpoint (e.g., `https://your-pingfed-server:9031/pf/federation_metadata.ping?PartnerSpId=https://github.com/orgs/YOUR_ORG`)

> 💡 **Tip:** **PingOne:** These values are available under the Application, Configuration tab, or by downloading the IdP metadata XML.

## 3️⃣ Enable SAML SSO in GitHub

**👤 Role:** GitHub org owner (Step 3B) / enterprise owner (Step 3A) · **📍 Portal:** GitHub

You will configure **Organization SAML** (always for the target org).

If your org is under an Enterprise account, you may also configure **Enterprise SAML**.

> ⚠️ **Warning:** Enabling SAML impacts how members authenticate. Ensure you have recovery codes stored for break-glass access.

### 3A — (Conditionally Required) Configure Enterprise SAML

**When this step is required:**

- If you are a GitHub Enterprise (enterprise account) customer and intend to require SAML at the enterprise level, complete this before org enforcement. **Enterprise SAML completely replaces org-level SAML configuration and enforces SAML SSO for every organization in the enterprise.**

**Navigate:** **Enterprises** page (github.com/settings/enterprises) → *[select enterprise]* → **Settings** → **Authentication security** → **SAML single sign-on**

**Configuration Steps**

1. In Enterprise **Settings** → **Authentication security** → **SAML single sign-on**
2. Enable / configure SAML per GitHub's enterprise SAML documentation (PingFederate values differ from the org app)

### 3B — Enable & Test Organization SAML (Required)

**Navigate:** Profile picture → **Organizations** → *[your org]* → **Settings** → **Authentication security** → **SAML single sign-on**

**Configuration Steps (Organization SAML)**

1. Under **SAML single sign-on**, select **Enable SAML authentication**
2. Populate fields using PingFederate/PingOne values from Step 2:
    - **Sign on URL** (IdP SSO Service URL from PingFederate/PingOne)
    - **Issuer** (IdP Entity ID from PingFederate/PingOne) — _Note: GitHub indicates Issuer is required for some features like team synchronization_
    - **Public Certificate** (X.509 Signing Certificate from PingFederate/PingOne)
3. Click **Test SAML configuration** and complete the auth flow (it must pass before you can save)
4. Click **Save**
5. **Immediately download and secure SSO recovery codes** from **Settings** → **Authentication security** → under **SAML single sign-on**, click **Save your recovery codes** → **Download**

> 🔐 **Critical:** Before enabling or immediately after enabling SAML, download and securely store your organization SSO recovery codes. These are essential for break-glass scenarios if your IdP becomes unavailable.

## 4️⃣ Enforce SAML SSO for the Organization (Required)

**👤 Role:** GitHub org owner · **📍 Portal:** GitHub

> ⚠️ **Important:** Enforcement removes org members who have not authenticated through the IdP, and can also remove bots/service accounts that don't have external identities. **If a user rejoins the organization within three months, the user's access privileges and settings will be restored.**

**Navigate:** Profile picture → **Organizations** → *[your org]* → **Settings** → **Authentication security** → **SAML single sign-on**

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

**👤 Role:** GitHub org owner / SCIM setup user · **📍 Portal:** GitHub + PingFederate / PingOne

> 🚨 **Blocker / support constraint:** GitHub's **officially supported** identity providers for **organization-level SCIM** are **Microsoft Entra ID, Okta, and OneLogin** only. **PingOne is documented for SAML SSO only (not SCIM), and PingFederate is not listed for org SCIM at all.** Supported org SCIM is authorized through the partner IdP's GitHub-published OAuth app (authorized by an org owner) — not by handing a bearer PAT to a generic IdP. You *can* drive the org SCIM REST API from a custom client (base URL `https://api.github.com/scim/v2/organizations/ORG`, classic PAT scoped `admin:org`), but with PingFederate/PingOne this is an **unsupported/undocumented custom-integration path — proceed at your own risk**. If you need a supported turnkey integration, use Entra, Okta, or OneLogin. The steps below assume you are building the Ping → REST API path as a custom SCIM client.

> 💡 **Tip:** In Standard non-EMU, SCIM manages organization membership lifecycle. A SCIM client calls GitHub's org SCIM REST API using a classic PAT owned by the dedicated setup user.

### 5A — Generate SCIM PAT from the Setup User (Required)

1. Sign into GitHub as the SCIM setup user
2. Create an active org SAML session by visiting `https://github.com/orgs/ORGANIZATION-NAME/sso`
3. Complete the SSO authentication prompt successfully
4. Generate a **Personal Access Token (classic)** with the `admin:org` scope.
    - **Navigate:** Profile picture → **Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)** → **Generate new token** → check **`admin:org`** → **Generate token**
5. **Copy the token immediately (shown once)** and store it securely — you will need it for the SCIM client configuration.
6. **SSO-authorize the token for the org:** next to the new token click **Configure SSO** → **Authorize** for your organization. SCIM/API calls will return **403** until the PAT is SSO-authorized (see Step 8B).

### 5B — Configure Outbound Provisioning in PingFederate (Required)

The org SCIM base URL is a **fixed value** — there is no per-org "generate token" or tenant-URL page to read it from:

```
SCIM Base URL (GitHub.com):  https://api.github.com/scim/v2/organizations/YOUR_ORG
Authentication:              Bearer token = the classic PAT (admin:org) from Step 5A (SSO-authorized)
```

*(GHE.com data residency applies to EMU enterprises only — not to Standard non-EMU organizations.)*

**Option A: PingFederate (On-Prem)**

1. Navigate to your GitHub SP Connection in PingFederate
2. Go to **Connection Type** and enable **Outbound Provisioning**
3. Configure the SCIM outbound provisioning channel with the **SCIM Base URL** above and **Authentication:** Bearer Token = the Step 5A PAT
4. Configure attribute mappings per GitHub's SCIM schema requirements
5. Click **Save**, then set the provisioning channel to **Active**

**Option B: PingOne (Cloud)**

1. Navigate to your GitHub application in PingOne
2. Go to the **Provisioning** tab
3. Enable provisioning and configure the **SCIM Base URL** above with the Step 5A PAT as the **Authentication Token**
4. Configure attribute mappings
5. Click **Save** and enable provisioning

> 📌 **Note:** Standard-GHEC organizations do **not** expose an enterprise/EMU-style "SCIM provisioning → Generate token" page or a per-org tenant URL under **Authentication security**. The base URL is the fixed value shown above and the credential is the SSO-authorized classic PAT (`admin:org`).

### 5C — Assign Users/Groups (Required)

**PingFederate — Navigate:** PingFederate administrative console → **Applications** → **Integration** → **SP Connections** → *[your GitHub SP Connection]* → **Outbound Provisioning** → **Configure Provisioning** → **Manage Channels** → *[channel]* → **Source Location** (configure the user/group filter)

**PingOne — Navigate (either path):**
- PingOne admin console → **Applications** → **Applications** → *[GitHub application]* → **Access** tab → add the group(s) that may sign in → **Save**

**Steps**

1. Assign:
    - Pilot group(s) first
2. **Validate lifecycle:**
    - Add assignment → user becomes org member
    - Remove assignment → user is removed from org (per your provisioning settings)

## 6️⃣ Attach Azure Subscription for Metered Billing

**👤 Role:** GitHub enterprise owner (or org owner for org-level billing) + Azure **subscription Owner** with tenant-wide admin consent · **📍 Portal:** GitHub + Azure

**Required if you are billing via Azure**

### Prerequisites

- ✓ You are an Owner of the GitHub org or enterprise account you are connecting
- ✓ You know the Azure subscription ID
- ✓ You are logged into Azure with a user who can provide tenant-wide admin consent (or you have an admin-consent workflow)
- ✓ **If the Azure subscription is in a different tenant than your default, you may need to specify a different tenant ID during connection**

### Configuration Steps (GitHub)

**Navigate:** Profile picture → **Organizations** (`https://github.com/settings/organizations`) or **Enterprises** (`https://github.com/settings/enterprises`) → *[target org/enterprise]* → **Billing and licensing** → **Payment information** → **Metered billing via Azure** → **Add Azure Subscription**

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

If your organization belongs to an enterprise account (the usual GHEC setup), set up Copilot in this order: **7A** turn Copilot on for the organization, **7B** set enterprise policies, **7C** set organization policies, then give people seats with **7D** (organization) and/or **7E** (enterprise).

> 📌 **Where things live:** organization access and licenses are under the enterprise's **Billing and licensing → Licensing**; enterprise policies are under **AI controls**; organization policies and seats are under the organization's **Settings → Copilot**. Selections on these pages apply immediately — there is **no Save button**.

### 7A — Turn Copilot on for organizations

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** **Enterprises** page (github.com/settings/enterprises) → *[your enterprise]* → **Billing and licensing** → **Licensing** → **Manage** *(in the "Copilot" section)*

1. At the top of the enterprise page, click **Billing and licensing**.
2. In the "Billing and licensing" sidebar, click **Licensing**.
3. In the "Copilot" section, click **Manage**.
4. Next to **Organization access**, open the dropdown and choose whether to enable Copilot for **all organizations** or to **Allow for specific organizations**.
5. If you chose **Allow for specific organizations**:
   1. Click the **Organizations** tab.
   2. Find the organization.
   3. To the right of its name, open the **Copilot** dropdown and click **Enabled** (Copilot Business plan) — or **Copilot: Enterprise** / **Copilot: Business** if your enterprise has a Copilot Enterprise plan.
6. Confirm the organization now shows Copilot as enabled. *(The selection applies immediately — there is no Save button.)*

> ⚠️ **Do this first:** until Copilot is enabled for an organization here, its owners can't assign seats in 7D.

### 7B — Set enterprise Copilot policies

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** **Enterprises** page (github.com/settings/enterprises) → *[your enterprise]* → **AI controls** → **Copilot** *(sidebar)*

1. At the top of the enterprise page, click **AI controls** *(a top-of-page tab — not under **Settings**)*.
2. In the sidebar, open the page that holds the policies you want:
   - **Copilot** — administration, privacy, model, billing, and usage policies, including **Policies for enterprise-assigned users** (required before 7E).
   - **Copilot** → under "Features & clients", click **Configure features & clients** — feature and client policies such as Copilot on GitHub.com, Copilot Chat in the IDE, and Copilot in the CLI.
   - **Agents** — AI agent policies, such as **Copilot cloud agent** (formerly Copilot coding agent).
   - **MCP** — Model Context Protocol (MCP) policies.
3. Set each policy:
   - **Dropdown:** open it and choose an enforcement option — **Enabled**, **Disabled**, or **No policy** (lets each organization owner decide in 7C).
   - **Toggle:** click it.
   - **No visible control:** click the policy name to see its options.
4. Check that each policy shows the value you chose. *(Changes apply on selection — there is no Save button.)*

> 💡 **Suggestions matching public code:** agree on this setting with your legal team before you enable it.

### 7C — Set organization Copilot policies

**👤 Role:** GitHub **organization owner** · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Organizations** → *[organization]* → **Settings** → **Copilot** *(sidebar, under "Code, planning, and automation")*

1. Click **Policies** to set feature and privacy policies, or **Models** to choose which models beyond the basic set are available (some can add cost).
2. For each policy, open its dropdown and choose an enforcement option. *(Changes apply on selection.)*

> 📌 **Enterprise wins:** a policy the enterprise set in 7B can't be changed here — only policies left at **No policy** are editable by the organization.

### 7D — Assign seats in the organization

**👤 Role:** GitHub **organization owner** · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Organizations** → *[organization]* → **Settings** → **Copilot** *(sidebar, under "Code, planning, and automation")* → **Access**

1. If you see **Allow this organization to assign seats**, click it.
2. Click **Start adding seats**.
3. Choose who gets Copilot:
   - **Everyone:** select **Purchase for all members**, then in the "Confirm seats purchase for all members" dialog click **Purchase seats**.
   - **Specific people or teams:** select **Purchase for selected members**. In the "Enable Copilot access for users and teams" dialog, use the **Users and teams** tab to search for and add people or teams (or **Upload CSV** to add many at once), then click **Continue to purchase** → **Purchase seats**.

> 💡 **Seats by team:** give seats to a GitHub team and manage membership there. (Team synchronization with IdP groups is only available for Entra ID and Okta.)

> 💡 **Billing:** a seat is billed from the moment it's granted (prorated mid-cycle), whether or not the person uses Copilot yet.

### 7E — Assign Copilot Business licenses at the enterprise level

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** **Enterprises** page (github.com/settings/enterprises) → *[your enterprise]* → **Billing and licensing** → **Licensing** → **Manage** *(in the "Copilot" section)*

**Before you start:** set the **Policies for enterprise-assigned users** policy (7B), make sure the people are already members of the enterprise (organization members, or users you've invited to the enterprise), and create the enterprise team first if you're licensing a team.

1. Click the **All members** tab (individual users) or the **Enterprise Teams** tab.
2. Click **Assign licenses**.
3. Search for the users or enterprise teams, then click **Add licenses**.

> ✅ **Enterprise teams are generally available** (since June 2026). License an enterprise team and people gain or lose Copilot as they join or leave it.

> 💡 **When to use this route:** people who need Copilot but no organization access. Enterprise members who aren't in any organization usually don't consume a GitHub Enterprise Cloud license. Direct enterprise assignment is for **Copilot Business**.

> 📌 **One license per person:** someone assigned through both 7D and 7E uses **one** license (the highest tier).

## 8️⃣ Critical Post-Enablement: SSO Authorization for Credentials (Required)

**👤 Role:** Each affected user (self-service) · **📍 Portal:** GitHub

When SAML is enabled/enforced, users often must authorize credentials (depending on token type and whether they have a linked external identity).

### 8A — Authorize SSH Keys for SSO

**👤 Role:** each **organization member** (for their own SSH keys and tokens) · **📍 Portal:** GitHub

**Required for SSH usage in SSO orgs**

**Navigate:** Profile picture → **Settings** → **SSH and GPG keys** → next to the key click **Configure SSO** → **Authorize** (for the org)

### 8B — Authorize Personal Access Tokens

**👤 Role:** each **organization member** (for their own SSH keys and tokens) · **📍 Portal:** GitHub

**Required for PAT classic in SSO orgs**

**Navigate:** Profile picture → **Settings** → **Developer settings** → **Personal access tokens** → next to the token click **Configure SSO** → **Authorize** (for the org)

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
- [ ] PAT with `admin:org` scope created, stored securely, and SSO-authorized for the org
- [ ] SCIM client configured with fixed base URL `https://api.github.com/scim/v2/organizations/YOUR_ORG` and the PAT as bearer token (custom/unsupported path for Ping — see Step 5 note)
- [ ] Assign/unassign test behaves as expected

### Azure Billing

- [ ] Azure subscription successfully connected under Payment information
- [ ] "Metered billing via Azure" shows the correct subscription ID

### Copilot

- [ ] Enterprise Copilot Business licenses assigned via Billing and licensing → Licensing (if enterprise-managed)
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


### Q: What is the difference between an SP Connection (PingFederate) and an Application (PingOne) for this setup?
**A:** An SP Connection in PingFederate (self-managed) is the equivalent of an Application in PingOne (cloud). Both define the SAML trust relationship between Ping and GitHub. In PingFederate, you configure SP Connections under **Applications** → **Integration** → **SP Connections** (9.x: **Identity Provider** → **SP Connections**). In PingOne, you configure them under **Applications** → **Applications**. The SAML values (Entity ID, ACS URL, Sign-on URL) are the same — the admin UI and navigation differ.

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
- [PingIdentity GitHub Integration Documentation](https://docs.pingidentity.com/integrations/github/)

---

*Last updated: October 2026*
