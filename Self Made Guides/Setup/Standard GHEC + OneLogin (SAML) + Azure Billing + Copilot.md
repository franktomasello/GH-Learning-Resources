# 🚀 GitHub Standard GHEC + OneLogin (SAML), Azure Billing & Copilot Setup Runbook

> **Complete end-to-end runbook for configuring Standard (non-EMU) GHEC with OneLogin (SAML), SCIM org provisioning, Azure billing, and GitHub Copilot**

| 🧭 **At a Glance** | |
|---|---|
| **Goal** | Set up standard GHEC with OneLogin single sign-on end to end |
| **Use this when** | A standard (personal-account) enterprise uses OneLogin |
| **People you need** | Enterprise or organization owner; OneLogin admin; Azure subscription owner |
| **Where you click** | GitHub and the OneLogin admin portal |
| **End result** | SAML SSO, SCIM provisioning, Azure billing, and Copilot ready to use |
| **New to a term?** | See the [Glossary](../Glossary.md) for plain-English definitions |

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [📋 Overview](#-overview)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [1️⃣ Create & Secure the "SCIM Setup User" (Standard non-EMU)](#1️⃣-create--secure-the-scim-setup-user-standard-non-emu)
- [2️⃣ Create the OneLogin Application (GitHub Organization Connector)](#2️⃣-create-the-onelogin-application-github-organization-connector)
- [3️⃣ Configure SAML (GitHub ↔ OneLogin Values)](#3️⃣-configure-saml-github--onelogin-values)
- [4️⃣ Enable & Test SAML SSO in GitHub](#4️⃣-enable--test-saml-sso-in-github)
- [5️⃣ Enforce SAML SSO for the Organization (Required)](#5️⃣-enforce-saml-sso-for-the-organization-required)
- [6️⃣ Configure SCIM Provisioning (OneLogin → GitHub Organization)](#6️⃣-configure-scim-provisioning-onelogin--github-organization)
- [7️⃣ Assign Users & Groups](#7️⃣-assign-users--groups)
- [8️⃣ Attach Azure Subscription for Metered Billing](#8️⃣-attach-azure-subscription-for-metered-billing)
- [9️⃣ Enable GitHub Copilot (Enterprise + Organization)](#9️⃣-enable-github-copilot-enterprise--organization)
- [🔟 Critical Post-Enablement: SSO Authorization for Credentials (Required)](#-critical-post-enablement-sso-authorization-for-credentials-required)
- [✅ Pre-Flight / Validation Checklist](#-pre-flight--validation-checklist)
- [🎯 Success Criteria](#-success-criteria)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **OneLogin:** **Applications** → **Add App** → add the GitHub organization connector from the OneLogin catalog → **Configuration** tab → enter org slug → **SSO** tab → capture **SAML 2.0 Endpoint**, **Issuer URL**, **X.509 Certificate**
- **GitHub:** **Organization** → **Settings** → **Authentication security** → **SAML single sign-on** → paste OneLogin values → **Test SAML configuration** → **Save** → **Require SAML SSO authentication for all members**
- **SCIM (optional):** Sign in as the setup user with an active SAML session → launch the OneLogin GitHub-org provisioning flow → in GitHub click **Grant** next to your org, then **Authorize** the SCIM OAuth app → in OneLogin enable **Create / Update / Deactivate users** (there is no GitHub-generated SCIM token to paste)
- **Billing:** **Enterprise** (or **Organization**) → **Billing and licensing** → **Payment information** → **Metered billing via Azure** → **Add Azure Subscription** → **Accept** → check consent box → **Connect**
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

- **OneLogin (SAML SSO)** for authentication
- **Enforce SAML SSO** for the organization (and enterprise, if applicable)
- **SCIM provisioning** (OneLogin → GitHub org membership lifecycle)
- **Azure subscription** attachment for metered billing
- **GitHub Copilot** enablement + policy controls
- **Critical post-enablement** items (PAT/SSH SSO authorization)

### Standard (non-EMU) means:

- Users keep their regular GitHub.com accounts
- SAML SSO is enforced at the org (and optionally at the enterprise, if you have one)
- SCIM manages organization membership via OneLogin assignments (not "managed user accounts"—that is EMU)

---

## ✅ Prerequisites

| Requirement | Owner / Role | Notes |
| --- | --- | --- |
| GitHub Enterprise Cloud (Standard / non-EMU) org | Org Owner | You must be able to access Org Settings → Authentication security |
| (If applicable) GitHub Enterprise account | Enterprise Owner | Needed only if you will configure Enterprise SAML and/or Enterprise Copilot policies |
| OneLogin Admin access | OneLogin Admin | Must be able to install and configure apps in the OneLogin admin portal |
| Pilot users/groups in OneLogin | IAM team | Use a small pilot cohort first to reduce risk |
| Dedicated GitHub "SCIM setup user" | Org Owner | A stable org owner who **authorizes** the OneLogin SCIM OAuth app; org SCIM then acts on behalf of this user. GitHub does **not** issue a SCIM token here (org SCIM is OAuth-based, not token-based). Optional — SCIM is optional for Standard GHEC |
| Azure subscription + ability to consent | Azure admin | Needed to connect metered billing via Azure. If subscription is in a different tenant, you may need to specify a different tenant ID during connection |
| Copilot plan decision | Enterprise/Org owner | Copilot Business vs Copilot Enterprise |

---

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **OneLogin administrator** | Creates the GitHub SAML app and configures provisioning if OneLogin SCIM is used. | OneLogin Admin Portal → Applications → Applications → Add App → search GitHub or SAML Custom Connector → Add → Configuration → enter GitHub org SAML URLs → SSO → copy SAML 2.0 Endpoint, Issuer URL, and X.509 Certificate → Provisioning → launch the GitHub OAuth authorization and enable Create/Update/Deactivate users if used → Save. Handoff: SAML values, provisioning status, and assigned pilot users/groups. |
| **GitHub organization owner or dedicated SCIM setup user** | Enables GitHub org SAML and authorizes the OneLogin SCIM OAuth app for provisioning when used. | GitHub → profile picture → **Organizations** → *[org]* → **Settings** → **Authentication security** → **SAML single sign-on** → **Enable SAML authentication** → paste OneLogin **Sign on URL**, **Issuer**, and **Public Certificate** → **Test SAML configuration** → **Save**. For SCIM (optional): with an active SAML session, launch the OneLogin provisioning flow and, when GitHub shows the org, click **Grant** then **Authorize** the SCIM OAuth app. Handoff: SAML test success, SCIM authorization confirmed, and recovery codes. |
| **OneLogin group or role owner** | Maintains app assignments and role mappings. | OneLogin Admin Portal → Users → Roles or Groups → [role/group] → Users → add users, then Applications → [GitHub app] → Users → confirm assignment. Handoff: assigned role/group and pilot users. |
| **GitHub enterprise or organization owner** | Starts the Azure metered billing connection from GitHub. | Enterprise path: GitHub → Enterprises page (github.com/settings/enterprises) → [enterprise] → Billing and licensing → Payment information → Metered billing via Azure → Add Azure Subscription. Organization path: GitHub → profile picture → Organizations → [organization] → Settings → Billing and licensing → Payment information → Metered billing via Azure → Add Azure Subscription. Then sign in to Microsoft → Permissions requested → Accept → Select a subscription → Connect. Handoff: the subscription ID is visible on Payment information. |
| **Azure subscription Owner** | Provides the Azure subscription that GitHub will bill against, or grants another signer the required Azure RBAC rights. | Azure portal → Subscriptions → [subscription] → Access control (IAM) → Role assignments → confirm the signer is listed under Owner. To grant access: Add → Add role assignment → Privileged administrator roles → Owner → Members → Select members → [user] → Select → Review + assign. Handoff: subscription ID and tenant ID. |
| **Microsoft Entra Global Administrator or consent approver** | Approves tenant-wide consent when the Microsoft consent prompt blocks the GitHub billing app. | Microsoft Entra admin center → Entra ID → Enterprise apps → Activity → Admin consent requests → My Pending → [GitHub request] → Review permissions and consent → Approve. If the Global Administrator completes the GitHub flow directly, approve the Permissions requested prompt by clicking Accept. |

---

## 1️⃣ Create & Secure the "SCIM Setup User" (Standard non-EMU)

> **👤 Role:** Org Owner · **📍 Portal:** GitHub + OneLogin

> ⚠️ **Important:** If you use org SCIM, provisioning acts on behalf of the GitHub user who **authorizes** OneLogin's SCIM OAuth app (org SCIM is OAuth-based — GitHub does **not** generate an API token here). If that user loses access or leaves the org, SCIM can stop working, so use a stable, dedicated identity as the authorizer. (SCIM is optional for Standard GHEC.)

### Process

1. Create a dedicated GitHub.com account (example: `gh-onelogin-scim@yourdomain.com`)
2. **Secure the account:**
    - Enable **2FA**
    - Ensure recovery methods are stored per your approved process (vault / break-glass)
3. **Add the setup user to your GitHub organization as an Owner:** GitHub → **Organizations** → *[org]* → **Settings** → **People** → **Invite member**; after the invite is accepted, set the role to **Owner**
4. **Assign this same identity in OneLogin** to the GitHub app (so it can SSO and be used to authorize the SCIM OAuth app)

### Important Notes

> 📌 **Note:** This setup user will consume a GitHub license. Treat it as a system account: minimal use outside of IAM configuration.

---

## 2️⃣ Create the OneLogin Application (GitHub Organization Connector)

> **👤 Role:** OneLogin Admin · **📍 Portal:** OneLogin

**Navigate:** OneLogin Admin Portal → **Applications** → **Add App** → search the GitHub organization connector

> 💡 **Tip:** Verify the current OneLogin connector name and its SCIM authentication method against OneLogin's own GitHub connector documentation — catalog names change over time. GitHub presents the OAuth app as **GitHub Enterprise Cloud - Organization** during authorization, but the OneLogin catalog entry may be labeled differently.

### Configuration Steps

1. **Select the GitHub organization connector** from the OneLogin catalog.
    - ⚠️ Do NOT use "GitHub Enterprise Managed User" (that is for EMU)
2. **Fill in required fields:**
    - In the app, open the **Configuration** tab and enter your GitHub organization name (org slug)
    - Set the application label / display name (a unique name in OneLogin)
3. **Assign the app to:**
    - The SCIM setup user
    - Your pilot user/group

### Gather Required Items (OneLogin IdP values)

In the OneLogin app:

1. Go to the **SSO** tab
2. Capture:
    - **SAML 2.0 Endpoint (HTTP)** — this is the Sign on URL
    - **Issuer URL**
    - **X.509 Certificate** (click View Details to download the public cert)

---

## 3️⃣ Configure SAML (GitHub ↔ OneLogin Values)

> **👤 Role:** OneLogin Admin · **📍 Portal:** OneLogin (using GitHub SP values)

### GitHub Org SAML Values (for reference when configuring OneLogin)

| Field | Value |
| --- | --- |
| Entity ID / Audience | `https://github.com/orgs/YOUR_ORG` |
| ACS (Assertion Consumer Service) URL | `https://github.com/orgs/YOUR_ORG/saml/consume` |
| Sign-on URL | `https://github.com/orgs/YOUR_ORG/sso` |

Replace `YOUR_ORG` with your actual GitHub organization slug.

### OneLogin App Configuration

1. In the OneLogin app → **Configuration** tab:
    - Ensure the organization name matches your GitHub org slug
2. In the OneLogin app → **SSO** tab:
    - Confirm SAML Signature Algorithm is set to **SHA-256**
3. In the OneLogin app → **Parameters** tab:
    - Verify that NameID maps to the user's email address

---

## 4️⃣ Enable & Test SAML SSO in GitHub

**👤 Role:** GitHub **organization owner** · **📍 Portal:** GitHub

You will configure **Organization SAML** (always for the target org).

If your org is under an Enterprise account, you may also configure **Enterprise SAML**.

> ⚠️ **Warning:** Enabling SAML impacts how members authenticate. Ensure you have recovery codes stored for break-glass access.

### 4A — (Conditionally Required) Configure Enterprise SAML

> **👤 Role:** Enterprise Owner · **📍 Portal:** GitHub

**When this step is required:**

- If you are a GitHub Enterprise (enterprise account) customer and intend to require SAML at the enterprise level, complete this before org enforcement. **Enterprise SAML completely replaces (overrides) all org-level SAML configuration and enforces SAML SSO for every organization in the enterprise.**

**Navigate:** **Enterprises** page (github.com/settings/enterprises) → *[your enterprise]* → **Settings** → **Authentication security** → **SAML single sign-on**

**Configuration Steps**

1. Under **SAML single sign-on**, enable and configure SAML. Enterprise-level SP values use the **enterprise slug** (they differ from the org values in Step 3):
    - **Entity ID / Audience:** `https://github.com/enterprises/YOUR_ENTERPRISE`
    - **ACS URL:** `https://github.com/enterprises/YOUR_ENTERPRISE/saml/consume`
    - **Sign-on URL:** `https://github.com/enterprises/YOUR_ENTERPRISE/sso`
2. Paste the OneLogin **Sign on URL**, **Issuer**, and **Public Certificate** (from a OneLogin app configured with the enterprise SP values above).
3. Click **Test SAML configuration** and complete the auth flow.
4. Click **Save**.

> 📌 **Constraint:** Enabling enterprise SAML overrides all org-level SAML config for organizations in the enterprise.

### 4B — Enable & Test Organization SAML (Required)

> **👤 Role:** Org Owner · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Organizations** → *[your org]* → **Settings** → **Authentication security** → **SAML single sign-on**

**Configuration Steps (Organization SAML)**

1. Under **SAML single sign-on**, select **Enable SAML authentication**
2. Populate fields using OneLogin values from Step 2:
    - **Sign on URL** — OneLogin SAML 2.0 Endpoint (HTTP)
    - **Issuer** — OneLogin Issuer URL — _Note: GitHub indicates Issuer is required for some features like team synchronization_
    - **Public Certificate** — OneLogin X.509 Certificate
3. Click **Test SAML configuration** (or equivalent prompt) and complete the auth flow
4. Click **Save**
5. **Immediately download and secure SSO recovery codes:** **Organization** → **Settings** → **Authentication security** → under **SAML single sign-on**, click **Save your recovery codes** → **Download**

> 🔐 **Critical:** Before enabling or immediately after enabling SAML, download and securely store your organization SSO recovery codes. These are essential for break-glass scenarios if your IdP becomes unavailable.

---

## 5️⃣ Enforce SAML SSO for the Organization (Required)

> ⚠️ **Important:** Enforcement removes org members who have not authenticated through the IdP, and can also remove bots/service accounts that don't have external identities. **If a user rejoins the organization within three months, the user's access privileges and settings will be restored.**

> **👤 Role:** Org Owner · **📍 Portal:** GitHub

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

---

## 6️⃣ Configure SCIM Provisioning (OneLogin → GitHub Organization)

> 💡 **Tip:** SCIM is **optional** for Standard GHEC and is scoped to the organization. Org SCIM works through a **third-party OAuth app** that the IdP owns and that a specific GitHub user **authorizes** — GitHub does **not** generate a SCIM API token or bearer token for you to paste. Provisioning then acts on behalf of the authorizing user.

> 📌 **Constraint:** GitHub publishes dedicated SCIM setup tutorials only for **Okta** and **Entra ID**. OneLogin is a supported IdP, but for the connector-specific steps follow OneLogin's own GitHub connector documentation.

### 6A — Establish an Active SAML Session (GitHub)

> **👤 Role:** SCIM setup user (Org Owner) · **📍 Portal:** GitHub

1. Sign in to GitHub as the SCIM setup user.
2. Establish an active SAML SSO session for the org by visiting `https://github.com/orgs/YOUR_ORG/sso` and authenticating via OneLogin. (Replace `YOUR_ORG` with your org slug.)

### 6B — Authorize the SCIM OAuth App (GitHub)

> **👤 Role:** SCIM setup user (Org Owner) · **📍 Portal:** GitHub (launched from OneLogin)

1. From the OneLogin GitHub-organization app's provisioning setup, launch the GitHub authorization flow.
2. When GitHub shows your organization, click **Grant** to the right of your organization's name.
3. Click **Authorize** to authorize the SCIM OAuth app on behalf of the setup user.

> 🔐 **Security:** The OAuth authorization is bound to the setup user's active SAML session and identity. Keep this a stable, dedicated org owner — if that account is deactivated, provisioning breaks.

### 6C — Enable Provisioning Actions in OneLogin (Required)

> **👤 Role:** OneLogin Admin · **📍 Portal:** OneLogin

**Navigate:** OneLogin Admin Portal → **Applications** → *[your GitHub organization app]* → **Provisioning**

1. Enable **Create users**.
2. Enable **Update users**.
3. Enable **Deactivate users**.
4. Click **Save**.

> 💡 **Tip:** OneLogin's exact provisioning UI (labels and whether authorization is launched from this tab) can vary by connector version — verify against OneLogin's current GitHub connector docs.

### 6D — Configure Provisioning Mappings (Required)

1. In the OneLogin app, open **Provisioning** or **Parameters**.
2. Keep mappings aligned to GitHub's SCIM guidance; avoid custom attributes until the base flow is stable.
3. Ensure NameID / email mapping is consistent between SAML and SCIM.

---

## 7️⃣ Assign Users & Groups

> **👤 Role:** OneLogin Admin · **📍 Portal:** OneLogin

**Navigate:** OneLogin Admin Portal → **Applications** → *[your GitHub organization app]* → **Users** → **Add users or groups**

**Steps**

1. In OneLogin → the GitHub app → **Users** (or **Access** tab)
2. Assign:
    - Pilot group(s) first
3. **Validate lifecycle:**
    - Add assignment → user becomes org member (via SCIM)
    - Remove assignment → user is removed from org (per your provisioning settings)

---

## 8️⃣ Attach Azure Subscription for Metered Billing

**Required if you are billing via Azure**

> **👤 Role:** GitHub Enterprise Owner (or Org Owner for org-level) + Azure Subscription Owner · **📍 Portal:** GitHub → Azure

### Prerequisites

- ✓ You are an **owner** of the GitHub enterprise (for enterprise-level) or the GitHub org (for org-level) you are connecting. A **billing manager** cannot connect a subscription at the enterprise level
- ✓ You know the Azure subscription ID
- ✓ You are logged into Azure with a user who can provide tenant-wide admin consent (or you have an admin-consent workflow)
- ✓ **If the Azure subscription is in a different tenant than your default, you may need to specify a different tenant ID during connection**

### Configuration Steps (GitHub)

**Navigate:** **Enterprises** page (github.com/settings/enterprises) → *[enterprise]* → **Billing and licensing** → **Payment information** → **Metered billing via Azure** → **Add Azure Subscription** _(org-level: **Organizations** → *[org]* → **Settings** → **Billing and licensing** → **Payment information**)_

**Copy-paste entry points:**

```
Org list:        https://github.com/settings/organizations
Enterprise list: https://github.com/settings/enterprises
```

**Process:**

1. Open the target org/enterprise from the entry point above.
2. Click **Billing and licensing** (a top-level tab for enterprises; in the left sidebar under **Settings** for orgs).
3. Click **Payment information**.
4. Scroll to the bottom. To the right of **Metered billing via Azure**, click **Add Azure Subscription**.
5. Sign in to Microsoft when prompted.
6. On **Permissions requested**, click **Accept** (or follow the admin approval flow if required).
7. Under **Select a subscription**, pick the Azure Subscription ID.
8. Check the confirmation/consent box acknowledging the billing relationship (**Connect** stays disabled until it is checked).
9. Click **Connect**.

> 💡 **Tip:** If you don't see a "Permissions requested" prompt and instead see a message about needing admin approval, you may need to configure an admin consent workflow in Azure or work with your Azure AD global administrator.

---

## 9️⃣ Enable GitHub Copilot (Enterprise + Organization)

If your organization belongs to an enterprise account (the usual GHEC setup), set up Copilot in this order: **9A** turn Copilot on for the organization, **9B** set enterprise policies, **9C** set organization policies, then give people seats with **9D** (organization) and/or **9E** (enterprise).

> 📌 **Where things live:** organization access and licenses are under the enterprise's **Billing and licensing → Licensing**; enterprise policies are under **AI controls**; organization policies and seats are under the organization's **Settings → Copilot**. Selections on these pages apply immediately — there is **no Save button**.

### 9A — Turn Copilot on for organizations

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

> ⚠️ **Do this first:** until Copilot is enabled for an organization here, its owners can't assign seats in 9D.

### 9B — Set enterprise Copilot policies

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** **Enterprises** page (github.com/settings/enterprises) → *[your enterprise]* → **AI controls** → **Copilot** *(sidebar)*

1. At the top of the enterprise page, click **AI controls** *(a top-of-page tab — not under **Settings**)*.
2. In the sidebar, open the page that holds the policies you want:
   - **Copilot** — administration, privacy, model, billing, and usage policies, including **Policies for enterprise-assigned users** (required before 9E).
   - **Copilot** → under "Features & clients", click **Configure features & clients** — feature and client policies such as Copilot on GitHub.com, Copilot Chat in the IDE, and Copilot in the CLI.
   - **Agents** — AI agent policies, such as **Copilot cloud agent** (formerly Copilot coding agent).
   - **MCP** — Model Context Protocol (MCP) policies.
3. Set each policy:
   - **Dropdown:** open it and choose an enforcement option — **Enabled**, **Disabled**, or **Let organizations decide** (each organization owner decides in 9C).
   - **Toggle:** click it.
   - **No visible control:** click the policy name to see its options.
4. Check that each policy shows the value you chose. *(Changes apply on selection — there is no Save button.)*

> ⏰ **Before October 22, 2026:** decide the **Default policy for new features** on this same **Copilot** page (**Enabled**, **Disabled**, or **Let organizations decide**). From that date, GA features you've left **Unconfigured** follow it — and it's **Enabled** by default. Explicit choices are never overridden.

> 💡 **Suggestions matching public code:** agree on this setting with your legal team before you enable it.

### 9C — Set organization Copilot policies

**👤 Role:** GitHub **organization owner** · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Organizations** → *[organization]* → **Settings** → **Copilot** *(sidebar, under "Code, planning, and automation")*

1. Click **Policies** to set feature and privacy policies, or **Models** to choose which models beyond the basic set are available (some can add cost).
2. For each policy, open its dropdown and choose an enforcement option. *(Changes apply on selection.)*

> 📌 **Enterprise wins:** a policy the enterprise set in 9B can't be changed here — only policies left at **Let organizations decide** are editable by the organization.

### 9D — Assign seats in the organization

**👤 Role:** GitHub **organization owner** · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Organizations** → *[organization]* → **Settings** → **Copilot** *(sidebar, under "Code, planning, and automation")* → **Access**

1. If you see **Allow this organization to assign seats**, click it.
2. Click **Start adding seats**.
3. Choose who gets Copilot:
   - **Everyone:** select **Purchase for all members**, then in the "Confirm seats purchase for all members" dialog click **Purchase seats**.
   - **Specific people or teams:** select **Purchase for selected members**. In the "Enable Copilot access for users and teams" dialog, use the **Users and teams** tab to search for and add people or teams (or **Upload CSV** to add many at once), then click **Continue to purchase** → **Purchase seats**.

> 💡 **Seats by team:** give seats to a GitHub team and manage membership there. (Team synchronization with IdP groups is only available for Entra ID and Okta.)

> 💡 **Billing:** a seat is billed from the moment it's granted (prorated mid-cycle), whether or not the person uses Copilot yet.

### 9E — Assign Copilot Business licenses at the enterprise level

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** **Enterprises** page (github.com/settings/enterprises) → *[your enterprise]* → **Billing and licensing** → **Licensing** → **Manage** *(in the "Copilot" section)*

**Before you start:** set the **Policies for enterprise-assigned users** policy (9B), make sure the people are already members of the enterprise (organization members, or users you've invited to the enterprise), and create the enterprise team first if you're licensing a team.

1. Click the **All members** tab (individual users) or the **Enterprise Teams** tab.
2. Click **Assign licenses**.
3. Search for the users or enterprise teams, then click **Add licenses**.

> ✅ **Enterprise teams are generally available** (since June 2026). License an enterprise team and people gain or lose Copilot as they join or leave it.

> 💡 **When to use this route:** people who need Copilot but no organization access. Enterprise members who aren't in any organization usually don't consume a GitHub Enterprise Cloud license. Direct enterprise assignment is for **Copilot Business**.

> 📌 **One license per person:** someone assigned through both 9D and 9E uses **one** license (the highest tier).

---

## 🔟 Critical Post-Enablement: SSO Authorization for Credentials (Required)

When SAML is enabled/enforced, users often must authorize credentials (depending on token type and whether they have a linked external identity).

### 10A — Authorize SSH Keys for SSO

**Required for SSH usage in SSO orgs**

> **👤 Role:** Any org member · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Settings** → **SSH and GPG keys** → next to the key click **Configure SSO** → **Authorize** (for the org)

### 10B — Authorize Personal Access Tokens

**Required for PAT classic in SSO orgs**

> **👤 Role:** Any org member · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Settings** → **Developer settings** → **Personal access tokens** → next to the token click **Configure SSO** → **Authorize** (for the org)

**Token nuance:**

> 📌 **Note:** GitHub states **PAT classic** requires post-creation SSO authorization. **Fine-grained PATs** are authorized during creation, before org access is granted.

---

## ✅ Pre-Flight / Validation Checklist

### Before Starting

- Organization slug: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
- SCIM setup user credentials stored securely: ☐
- SCIM setup user 2FA enabled with recovery codes saved: ☐
- OneLogin admin with privileges to create/configure app integrations: ☐
- Azure Subscription ID (for billing): \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
- Azure admin who can grant tenant-wide consent: ☐

### SAML

- [ ] Org SAML enabled and tested
- [ ] Recovery codes downloaded and stored
- [ ] Enforcement applied successfully
- [ ] Pilot users can access org resources via SSO

### SCIM (optional)

- [ ] Setup user is an org owner
- [ ] Setup user has an active SAML session and has authorized the OneLogin SCIM OAuth app (**Grant** → **Authorize**)
- [ ] OneLogin provisioning enabled with **Create / Update / Deactivate users**
- [ ] Assign/unassign test behaves as expected

### Azure Billing

- [ ] Azure subscription successfully connected under Payment information
- [ ] "Metered billing via Azure" shows the correct subscription ID

### Copilot

- [ ] Enterprise Copilot Business licenses assigned via **Billing and licensing → Licensing** (if enterprise-managed)
- [ ] Enterprise Copilot policies set under **AI controls → Copilot** (if applicable)
- [ ] Org Copilot policies set
- [ ] Licenses assigned to pilot cohort
- [ ] Pilot users can use Copilot in IDE / GitHub.com as expected

---

## 🎯 Success Criteria

After completing this guide, you should have:

- ✅ Standard GHEC organization fully configured with OneLogin SAML authentication
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
| **SCIM test connection fails** | For org SCIM, the SCIM OAuth app is not authorized, the setup user's SAML session lapsed, or the setup user lost org ownership. | Re-establish the setup user's SAML session, re-run the OAuth authorization (**Grant** → **Authorize**) so provisioning acts on behalf of a valid org owner, then confirm the IdP provisioning test succeeds before assigning users. |
| **Provisioned users are missing from GitHub** | Users or groups are not assigned to the IdP app, attribute mappings fail, or provisioning cycles have not completed. | Review IdP provisioning logs, fix mapping errors, assign a small pilot group, and wait for the next incremental provisioning cycle. |
| **Azure billing connection fails** | The Azure signer cannot grant tenant consent or does not own the subscription. | Use a subscription owner with tenant consent rights or run the Entra admin consent workflow, then repeat the GitHub Add Azure Subscription flow. |
| **Copilot controls or seats are not visible** | Copilot is not enabled for the enterprise/org, the signed-in user lacks owner/admin permissions, or the plan/add-on is not active. | Verify Copilot plan activation, enable access at the enterprise/org level, and assign seats from the documented access page. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>

### Q: How do I connect OneLogin SCIM provisioning to a GitHub organization?
**A:** GitHub organization SCIM doesn't use a token you generate in GitHub. It uses a third-party OAuth app that a specific GitHub user authorizes:

1. Sign in to GitHub as the SCIM setup user and start a SAML session at `https://github.com/orgs/YOUR_ORG/sso`.
2. Launch the provisioning authorization from the OneLogin GitHub organization app.
3. When GitHub shows your organization, click **Grant** next to it, then **Authorize**. Provisioning now acts on behalf of that user.
4. In OneLogin, enable **Create users**, **Update users**, and **Deactivate users**, click **Save**, and test with one pilot user.

GitHub publishes SCIM tutorials only for Okta and Entra ID. For OneLogin, follow OneLogin's own GitHub connector documentation.

---

### Q: OneLogin's SAML certificate is expiring — how do I renew it?
**A:** GitHub won't save a certificate until **Test SAML configuration** passes, and the test only passes once OneLogin signs with the new certificate. So swap them in this order, in one short maintenance window:

1. In OneLogin, create the new certificate (**Security** → **Certificates**) and download it.
2. Open the GitHub app → **SSO** tab → change the **X.509 Certificate** to the new one → **Save**. From here until step 3 is done, sign-ins fail with a `digest mismatch` error.
3. Right away, in GitHub: Organization → **Settings** → **Authentication security** → paste the new **Public Certificate** → **Test SAML configuration** → **Save**.
4. Remove the old certificate from OneLogin when you no longer need it.

> 💡 GitHub doesn't enforce the certificate's expiry date, so an expired certificate won't break sign-in on GitHub's side. Schedule the swap for a quiet time.

---

### Q: SAML assertion attribute mapping errors are preventing sign-in — what should I verify?
**A:** In OneLogin, open the GitHub app and check:

1. **Parameters** tab: the NameID maps to a stable, unique value such as the user's email.
2. **SSO** tab: **SAML Signature Algorithm** is **SHA-256** (SHA-1 can fail validation).
3. The **Issuer URL** and **SAML 2.0 Endpoint** match what you entered in GitHub.

Use a SAML tracer or your browser's developer tools to inspect the assertion.

---

### Q: Which OneLogin connector version should I use — there seem to be multiple?
**A:** Use the **GitHub organization connector** from the OneLogin catalog — not one labeled "GitHub Enterprise Managed User" (that's for EMU).

- During authorization, GitHub shows the OAuth app as **GitHub Enterprise Cloud - Organization**; the OneLogin catalog name may differ, so confirm it against OneLogin's docs.
- If you see several versions, choose the most recently updated one. Older connectors may target retired endpoints.
- If provisioning behaves oddly, check OneLogin's release notes for your connector version.

---

### Q: Users are authenticated via SAML but not getting added to the org — SCIM does not seem to be working. What is wrong?
**A:** Check, in order:

1. Provisioning is enabled (GitHub app → **Provisioning** tab) with **Create users**, **Update users**, and **Deactivate users** all checked.
2. The SCIM OAuth app is still authorized: the setup user has an active SAML session and completed **Grant** → **Authorize** (org SCIM runs on that grant, not a static token).
3. The OneLogin **Events** log shows no provisioning errors.
4. The user is assigned to the GitHub app in OneLogin — signing in with SAML alone doesn't trigger SCIM provisioning.

---

### Q: After SAML enforcement, some users lost access — how do I restore them?
**A:** Users who had not authenticated via the IdP before enforcement are removed from the org. If they rejoin within three months, their previous access privileges and settings are restored automatically. Direct them to `https://github.com/orgs/YOUR_ORG/sso` to authenticate. For bots or service accounts, either assign IdP identities to them or migrate to GitHub Apps. Review the enforcement removal list in Organization > People > filter by "Removed."

---

### Q: Can OneLogin handle group-based provisioning to GitHub, or only individual user assignment?
**A:** OneLogin supports both individual user and group-based assignment. You can assign groups under the GitHub app > Users/Access tab. However, OneLogin's SCIM provisioning for GitHub only manages org membership — it does not natively sync groups to GitHub teams. For team-based access control, you will need to manage GitHub team memberships separately or use the GitHub Teams API.

---

### Q: SCIM provisioning worked initially but has stopped syncing new users — what happened?
**A:** Usually the OAuth authorization broke — for example, the setup user's SAML session expired, they lost org ownership or were deactivated, or the SCIM OAuth app was revoked. (Org SCIM runs on that user's OAuth grant, not a static token.)

1. Check the OneLogin **Events** log for HTTP 401 or 403 errors from GitHub's SCIM endpoint.
2. Sign in as the setup user and refresh the SAML session at `https://github.com/orgs/YOUR_ORG/sso`.
3. Re-authorize the SCIM OAuth app (**Grant** → **Authorize**).

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
- [OneLogin GitHub Integration - KB Article](https://onelogin.service-now.com/support?id=kb_article&sys_id=kb0010344)

---

*Last updated: October 2026*
