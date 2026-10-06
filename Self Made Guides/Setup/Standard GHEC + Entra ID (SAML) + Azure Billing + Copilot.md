# 🚀 GitHub Standard GHEC + Microsoft Entra ID (SAML), Azure Billing & Copilot Setup Runbook

> **Complete end-to-end runbook for configuring Standard (non-EMU) GHEC with Microsoft Entra ID (SAML), SCIM org provisioning, Azure billing, and GitHub Copilot**

| 🧭 **At a Glance** | |
|---|---|
| **Goal** | Set up standard GHEC with Entra ID single sign-on end to end |
| **Use this when** | A standard (personal-account) enterprise uses Entra ID |
| **People you need** | Enterprise or organization owner; Entra Application Administrator; Azure subscription owner |
| **Where you click** | GitHub and the Microsoft Entra admin center |
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
- [2️⃣ Create the Entra ID Enterprise Application (GitHub Enterprise Cloud - Organization)](#2️⃣-create-the-entra-id-enterprise-application-github-enterprise-cloud---organization)
- [3️⃣ Configure SAML SSO in Entra ID](#3️⃣-configure-saml-sso-in-entra-id)
- [4️⃣ Enable & Test SAML SSO in GitHub Org Settings](#4️⃣-enable--test-saml-sso-in-github-org-settings)
- [5️⃣ Enforce SAML SSO for the Organization (Required)](#5️⃣-enforce-saml-sso-for-the-organization-required)
- [6️⃣ Configure SCIM Provisioning (Entra ID → GitHub Organization)](#6️⃣-configure-scim-provisioning-entra-id--github-organization)
- [7️⃣ Attach Azure Subscription for Metered Billing](#7️⃣-attach-azure-subscription-for-metered-billing)
- [8️⃣ Enable GitHub Copilot (Enterprise + Organization)](#8️⃣-enable-github-copilot-enterprise--organization)
- [9️⃣ Critical Post-Enablement: SSO Authorization for Credentials (Required)](#9️⃣-critical-post-enablement-sso-authorization-for-credentials-required)
- [🔟 Recovery Codes & Break-Glass Access](#-recovery-codes--break-glass-access)
- [✅ Pre-Flight / Validation Checklist](#-pre-flight--validation-checklist)
- [🎯 Success Criteria](#-success-criteria)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Entra ID:** Enterprise Applications → New → "GitHub Enterprise Cloud - Organization" → Single sign-on → SAML → Set Entity ID to `https://github.com/orgs/YOUR_ORG`
- **GitHub:** Org Settings → Authentication security → SAML single sign-on → Paste Entra Login URL, Azure AD Identifier, Base64 cert → Test → Save → Enforce
- **SCIM:** Entra → GitHub app → Provisioning → **+ New configuration** → enter Tenant URL `https://api.github.com/scim/v2/organizations/YOUR_ORG` → **Test Connection** → sign in to GitHub as setup user (active SAML session) → select org → **Authorize** the OAuth app → **Create**
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

- **Microsoft Entra ID (formerly Azure AD) SAML SSO** for authentication
- **Enforce SAML SSO** for the organization (and enterprise, if applicable)
- **SCIM provisioning** (Entra ID → GitHub org membership lifecycle)
- **Azure subscription** attachment for metered billing
- **GitHub Copilot** enablement + policy controls
- **Critical post-enablement** items (PAT/SSH SSO authorization)

### Standard (non-EMU) means:

- Users keep their regular GitHub.com accounts
- SAML SSO is enforced at the org (and optionally at the enterprise, if you have one)
- SCIM manages organization membership via Entra ID assignments (not "managed user accounts"—that is EMU)

---

## ✅ Prerequisites

| Requirement | Who / Role needed | ✓ |
| --- | --- | --- |
| GitHub Enterprise Cloud (Standard / non-EMU) org | GitHub **organization owner** (access to Org Settings → Authentication security) | ☐ |
| (If applicable) GitHub Enterprise account | GitHub **enterprise owner** — needed only for Enterprise Copilot policies. ⚠️ Do **not** enforce SAML at the enterprise level if you rely on org SCIM (Step 6); it disables org SCIM | ☐ |
| Microsoft Entra admin access | Entra **Application Administrator, Cloud Application Administrator, or Application Owner** (of the app) — create and configure Enterprise Applications | ☐ |
| Pilot users/groups in Entra ID | IAM team — use a small pilot cohort first to reduce risk | ☐ |
| Dedicated GitHub "SCIM setup user" | GitHub **organization owner** — the user who authorizes the SCIM OAuth app and holds an active SAML session | ☐ |
| Azure subscription + ability to consent | Azure **subscription Owner** + tenant-wide admin consent (or admin-consent workflow). If the subscription is in a different tenant, you may need to specify a different tenant ID during connection | ☐ |
| Copilot plan decision | GitHub **enterprise/organization owner** — Copilot Business vs Copilot Enterprise | ☐ |

---

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **Microsoft Entra Application Administrator, Cloud Application Administrator, or Application Owner** | Creates the standard GitHub organization enterprise app, configures SAML, and starts org SCIM provisioning. | Microsoft Entra admin center → Entra ID → Enterprise apps → New application → search GitHub Enterprise Cloud - Organization → Create → Single sign-on → SAML → Basic SAML Configuration → Edit → set Identifier, Reply URL, and Sign on URL for the GitHub org → Save → SAML Certificates → download Certificate (Base64) → copy Login URL and Azure AD Identifier. Then Provisioning → **+ New configuration** (older tenants: **Get started** → Provisioning Mode = Automatic) → enter Tenant URL `https://api.github.com/scim/v2/organizations/ORG` → **Test Connection** (opens a GitHub sign-in + **Authorize** dialog — select the org and Authorize the OAuth app) → **Create** (older UI: Save) → Users and groups → Add user/group → Assign → Start provisioning. Handoff: SAML values, SCIM test/authorize success, and assigned pilot group. |
| **GitHub organization owner or dedicated SCIM setup user** | Enables org SAML and authorizes the SCIM OAuth app while holding an active SAML session for provisioning setup. | GitHub → profile picture → Organizations → [org] → Settings → Authentication security → SAML single sign-on → Enable SAML authentication → paste Sign on URL, Issuer, and Public Certificate → Test SAML configuration → Save → download recovery codes. For SCIM: keep an active org SAML session (visit `https://github.com/orgs/ORG/sso` as the setup user), then when Entra's **Test Connection** opens the GitHub **authorization dialog**, select the target organization and click **Authorize**. Standard-org SCIM uses a GitHub-authorized third-party OAuth app — there is **no** GitHub-generated SCIM token to copy. Handoff: SAML test success, recovery codes, OAuth authorization confirmed, and Tenant URL. |
| **Microsoft Entra group owner** | Maintains the users and groups that should become GitHub org members. | Microsoft Entra admin center → Entra ID → Groups → [group] → Members → Add members → select users → Add, then Enterprise apps → [GitHub app] → Users and groups → Add user/group → select group → Assign. Handoff: assigned group and pilot users. |
| **GitHub enterprise or organization owner** | Starts the Azure metered billing connection from GitHub. | Enterprise path: GitHub → Enterprises page (github.com/settings/enterprises) → [enterprise] → Billing and licensing → Payment information → Metered billing via Azure → Add Azure Subscription. Organization path: GitHub → profile picture → Organizations → [organization] → Settings → Billing and licensing → Payment information → Metered billing via Azure → Add Azure Subscription. Then sign in to Microsoft → Permissions requested → Accept → Select a subscription → Connect. Handoff: the subscription ID is visible on Payment information. |
| **Azure subscription Owner** | Provides the Azure subscription that GitHub will bill against, or grants another signer the required Azure RBAC rights. | Azure portal → Subscriptions → [subscription] → Access control (IAM) → Role assignments → confirm the signer is listed under Owner. To grant access: Add → Add role assignment → Privileged administrator roles → Owner → Members → Select members → [user] → Select → Review + assign. Handoff: subscription ID and tenant ID. |
| **Microsoft Entra Global Administrator or consent approver** | Approves tenant-wide consent when the Microsoft consent prompt blocks the GitHub billing app. | Microsoft Entra admin center → Entra ID → Enterprise apps → Activity → Admin consent requests → My Pending → [GitHub request] → Review permissions and consent → Approve. If the Global Administrator completes the GitHub flow directly, approve the Permissions requested prompt by clicking Accept. |

---

## 1️⃣ Create & Secure the "SCIM Setup User" (Standard non-EMU)

**👤 Role:** GitHub organization owner · **📍 Portal:** GitHub

> ⚠️ **Important:** Standard (non-EMU) org SCIM uses a **third-party-owned OAuth app** that a specific GitHub organization owner authorizes while holding an active SAML session. There is **no** GitHub-generated SCIM token. Because the authorization is tied to the user who granted it, use a stable, dedicated identity — if that user loses access or leaves the org, provisioning can stop.

### Process

1. Create a dedicated GitHub.com account (example: `gh-entra-scim@yourdomain.com`)
2. **Secure the account:**
    - Enable 2FA
    - Ensure recovery methods are stored per your approved process (vault / break-glass)
3. **Add the setup user to your GitHub organization as an Owner:**
    - **Navigate:** Profile picture → **Organizations** → *[your org]* → **Settings** → **People** → **Invite member**
    - After the invite is accepted, set the role to **Owner**
4. **Confirm the user can authorize SCIM later:**
    - The setup user does not generate a token now. In Step 6 they sign in during Entra's **Test Connection** and click **Authorize** to connect the OAuth app to the org (an active org SAML session is required — see Step 6A).

### Important Notes

> 📌 **Note:** This setup user will consume a GitHub license. Treat it as a system account: minimal use outside of IAM configuration. The SCIM OAuth authorization is tied to this user — if the user is removed from the org or loses their SAML session, provisioning can break.

---

## 2️⃣ Create the Entra ID Enterprise Application (GitHub Enterprise Cloud - Organization)

**👤 Role:** Entra Application Administrator, Cloud Application Administrator, or Application Owner · **📍 Portal:** Microsoft Entra admin center

**Navigate:** [Microsoft Entra admin center](https://entra.microsoft.com) → **Identity** → **Applications** → **Enterprise applications** → **+ New application** → **Browse Microsoft Entra Gallery** → search **GitHub Enterprise Cloud - Organization** → **Create**

### Configuration Steps

1. **Select the gallery application:** `GitHub Enterprise Cloud - Organization`
    - ⚠️ Do NOT use "GitHub Enterprise Managed User" (that is for EMU)
2. **Fill in required fields:**
    - Name (give it a descriptive label, e.g., "GitHub Enterprise Cloud - YourOrgName")
3. **Assign the app to:**
    - The SCIM setup user
    - Your pilot user/group
    - Navigate to Users and groups → + Add user/group → select users/groups → Assign

### Note on App Registration

- The gallery application automatically creates the necessary app registration
- No manual app registration is required when using the gallery app

---

## 3️⃣ Configure SAML SSO in Entra ID

**👤 Role:** Entra Application Administrator, Cloud Application Administrator, or Application Owner · **📍 Portal:** Microsoft Entra admin center

**Navigate:** [Microsoft Entra admin center](https://entra.microsoft.com) → **Identity** → **Applications** → **Enterprise applications** → *[your "GitHub Enterprise Cloud - Organization" app]* → **Single sign-on** → **SAML**

### Configuration Steps

1. Click **SAML** under the Single sign-on method
2. Under **Basic SAML Configuration**, click **Edit** and populate:
    - **Identifier (Entity ID):** `https://github.com/orgs/YOUR_ORG`
    - **Reply URL (Assertion Consumer Service URL):** `https://github.com/orgs/YOUR_ORG/saml/consume`
    - **Sign on URL:** `https://github.com/orgs/YOUR_ORG/sso`
    - Replace `YOUR_ORG` with your actual GitHub organization slug
3. Click **Save**
4. Under **Attributes & Claims**, verify the following claim mappings:
    - **Unique User Identifier (Name ID):** `user.mail` — the Microsoft tutorial expects Name ID mapped to **user.mail**; use it as the default and keep it consistent with the SCIM `userName` to avoid duplicate identities.
    - Ensure the NameID format is set to **Email address** or **Persistent** as appropriate
5. Under **SAML Certificates**, download:
    - **Certificate (Base64)** — you will need this for GitHub
6. Under **Set up [Your App Name]**, capture:
    - **Login URL** — this is the SAML Sign-on URL for GitHub
    - **Azure AD Identifier** — this is the Issuer / Entity ID for GitHub
    - **Logout URL** — optional, for single logout

> 💡 **Tip:** You can also download the **Federation Metadata XML** file which contains all these values in a single file.

---

## 4️⃣ Enable & Test SAML SSO in GitHub Org Settings

**👤 Role:** GitHub **organization owner** · **📍 Portal:** GitHub

You will configure **Organization SAML** (always for the target org).

If your org is under an Enterprise account, you may also configure **Enterprise SAML**.

> ⚠️ **Warning:** Enabling SAML impacts how members authenticate. Ensure you have recovery codes stored for break-glass access.

### 4A — (Conditionally Required) Configure Enterprise SAML

**👤 Role:** GitHub enterprise owner · **📍 Portal:** GitHub

**When this step is required:**

- If you are a GitHub Enterprise (enterprise account) customer and intend to require SAML at the enterprise level, complete this before org enforcement. **Enterprise SAML completely replaces org-level SAML configuration and enforces SAML SSO for every organization in the enterprise.**

> 🚨 **Blocker:** If you configure or enforce SAML at the **enterprise** level:
> - **Organization-level SCIM (Step 6) isn't available**, and any existing org SCIM stops working.
> - Enterprises with personal accounts have **no** enterprise SCIM (that requires Enterprise Managed Users).
>
> So use **org-level** SAML + SCIM for a standalone organization, or move to **Enterprise Managed Users** for enterprise-level provisioning. Don't enforce enterprise SAML if you rely on org SCIM.

**Navigate:** **Enterprises** page (github.com/settings/enterprises) → *[your enterprise]* → **Settings** → **Authentication security** → **SAML single sign-on**

**Configuration Steps**

1. Under **Authentication security** → **SAML single sign-on**, enable / configure SAML per GitHub's enterprise SAML documentation (Entra ID values differ from the org app)
2. Note that enterprise SAML applies to every organization owned by the enterprise

### 4B — Enable & Test Organization SAML (Required)

**👤 Role:** GitHub organization owner · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Organizations** → *[your org]* → **Settings** → **Authentication security** → **SAML single sign-on**

**Configuration Steps (Organization SAML)**

1. Under **SAML single sign-on**, select **Enable SAML authentication**
2. Populate fields using Entra ID values from Step 3:
    - **Sign on URL** → paste the **Login URL** from Entra ID
    - **Issuer** → paste the **Azure AD Identifier** from Entra ID — _Note: GitHub indicates Issuer is required for some features like team synchronization_
    - **Public Certificate** → paste the contents of the **Certificate (Base64)** downloaded from Entra ID
    - Click the **Edit** (pencil) icon and set **Signature Method** = **RSA-SHA256** and **Digest Method** = **SHA256** (do not leave the SHA1 defaults)
3. Click **Test SAML configuration** (or equivalent prompt) and complete the auth flow
4. Click **Save**
5. **Immediately download and secure SSO recovery codes:**
    - **Navigate:** Org **Settings** → **Authentication security** → under **SAML single sign-on**, click **Save your recovery codes** → **Download**

> 🔐 **Critical:** Before enabling or immediately after enabling SAML, download and securely store your organization SSO recovery codes. These are essential for break-glass scenarios if your IdP becomes unavailable.

---

## 5️⃣ Enforce SAML SSO for the Organization (Required)

**👤 Role:** GitHub organization owner · **📍 Portal:** GitHub

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

---

## 6️⃣ Configure SCIM Provisioning (Entra ID → GitHub Organization)

> 📌 **Constraint:** Standard (non-EMU) organization SCIM doesn't use a GitHub-generated token.
> - It uses a **third-party-owned OAuth app** that a GitHub organization owner authorizes.
> - There's no "Generate token" or "Enable SCIM" control on the organization's **Authentication security** page.
> - Instead, Entra's **Test Connection** opens a GitHub sign-in and **authorization dialog**. Select the organization and click **Authorize** — that authorization takes the place of the **Secret Token**.
> - This kind of SCIM can't be used with enterprise-level SAML or with an organization that has managed users (see the Step 4A blocker).

### 6A — Prepare an Active SAML Session for the Setup User (Required)

**👤 Role:** GitHub organization owner (SCIM setup user) · **📍 Portal:** GitHub

1. Sign into GitHub as the SCIM setup user (an org owner)
2. Create an active org SAML session by visiting:

```
https://github.com/orgs/ORGANIZATION-NAME/sso

```
3. Complete the SSO authentication prompt successfully — an active SAML session is required so the authorization dialog in Step 6B can complete
4. Keep this browser session available; you will use it to **Authorize** the OAuth app during Entra's Test Connection

> 📌 **Note:** There is no SCIM token to copy. The OAuth authorization you grant in Step 6B is what connects Entra to the organization. If it later stops working, re-authorize after refreshing the setup user's SAML session.

### 6B — Configure Entra ID Provisioning & Authorize the OAuth App (Required)

**👤 Role:** Entra Application Administrator, Cloud Application Administrator, or Application Owner · **📍 Portal:** Microsoft Entra admin center

**Navigate:** [Microsoft Entra admin center](https://entra.microsoft.com) → **Identity** → **Applications** → **Enterprise applications** → *[your "GitHub Enterprise Cloud - Organization" app]* → **Provisioning**

**Steps**

1. Click **+ New configuration** (older tenants show **Get started** and a **Provisioning Mode = Automatic** dropdown — set it to **Automatic**)
2. Under **Admin Credentials**, enter the **Tenant URL:** `https://api.github.com/scim/v2/organizations/YOUR_ORG` (replace `YOUR_ORG` with your GitHub organization slug). Leave **Secret Token** blank — it is satisfied by the OAuth authorization in the next steps.
3. Click **Test Connection**. A GitHub sign-in window opens — sign in as the SCIM setup user (you must have an active org SAML session from Step 6A).
4. In the **authorization dialog**, select your target **organization** and click **Authorize**.
5. Return to Entra and click **Create** (older UI: **Save**) to save the configuration.

### 6C — Configure Provisioning Mappings (Required)

1. In the Provisioning section → **Mappings**
2. Review and configure **Provision Azure Active Directory Users**:
    - Ensure attribute mappings align with GitHub's SCIM schema
    - Key mappings: `userPrincipalName` → `userName`, `mail` → `emails[type eq "work"].value`, `displayName` → `displayName`
3. Enable the desired provisioning actions (Create / Update / Delete) as appropriate for your org membership lifecycle
4. Click **Save**

> 📌 **Note:** In tenants where UPN and mail differ, standardize on ONE attribute for BOTH the SAML NameID (Step 3) and the SCIM userName (`user.mail` is recommended, matching the Microsoft tutorial) so identity linking stays consistent.

### 6D — Start Provisioning & Assign Users/Groups (Required)

**Navigate:** [Microsoft Entra admin center](https://entra.microsoft.com) → **Identity** → **Applications** → **Enterprise applications** → *[your "GitHub Enterprise Cloud - Organization" app]* → **Users and groups** → **+ Add user/group** → *(select users/groups)* → **Assign**

**Steps**

1. In the Enterprise Application → **Users and groups**
2. Assign:
    - Pilot group(s) first
3. Return to **Provisioning** → click **Start provisioning**
4. **Validate lifecycle:**
    - Add assignment → user becomes org member
    - Remove assignment → user is removed from org (per your provisioning settings)
5. Monitor provisioning logs under **Provisioning** → **Provisioning logs** to confirm successful operations

> 💡 **Tip:** Entra ID runs an initial provisioning cycle when you first start provisioning. Subsequent incremental cycles run approximately every 40 minutes.

---

## 7️⃣ Attach Azure Subscription for Metered Billing

**👤 Role:** GitHub enterprise owner (or organization owner for org-level billing) + Azure subscription Owner · **📍 Portal:** GitHub → Microsoft

**Required if you are billing via Azure**

### Prerequisites

- ✓ You are an Owner of the GitHub org or enterprise account you are connecting
- ✓ You know the Azure subscription ID
- ✓ You are logged into Azure with a user who can provide tenant-wide admin consent (or you have an admin-consent workflow)
- ✓ **If the Azure subscription is in a different tenant than your default, you may need to specify a different tenant ID during connection**

### Configuration Steps (GitHub)

**Navigate:** GitHub → *your org or enterprise settings entry point* (org list: `https://github.com/settings/organizations` · enterprise list: `https://github.com/settings/enterprises`) → open the target org/enterprise → **Billing and licensing** → **Payment information** → **Metered billing via Azure** → **Add Azure Subscription**

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

> 💡 **Tip:** If you don't see a "Permissions requested" prompt and instead see a message about needing admin approval, you may need to configure an admin consent workflow in Azure or work with your Entra ID global administrator.

---

## 8️⃣ Enable GitHub Copilot (Enterprise + Organization)

If your organization belongs to an enterprise account (the usual GHEC setup), set up Copilot in this order: **8A** turn Copilot on for the organization, **8B** set enterprise policies, **8C** set organization policies, then give people seats with **8D** (organization) and/or **8E** (enterprise).

> 📌 **Where things live:** organization access and licenses are under the enterprise's **Billing and licensing → Licensing**; enterprise policies are under **AI controls**; organization policies and seats are under the organization's **Settings → Copilot**. Selections on these pages apply immediately — there is **no Save button**.

### 8A — Turn Copilot on for organizations

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

> ⚠️ **Do this first:** until Copilot is enabled for an organization here, its owners can't assign seats in 8D.

### 8B — Set enterprise Copilot policies

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** **Enterprises** page (github.com/settings/enterprises) → *[your enterprise]* → **AI controls** → **Copilot** *(sidebar)*

1. At the top of the enterprise page, click **AI controls** *(a top-of-page tab — not under **Settings**)*.
2. In the sidebar, open the page that holds the policies you want:
   - **Copilot** — administration, privacy, model, billing, and usage policies, including **Policies for enterprise-assigned users** (required before 8E).
   - **Copilot** → under "Features & clients", click **Configure features & clients** — feature and client policies such as Copilot on GitHub.com, Copilot Chat in the IDE, and Copilot in the CLI.
   - **Agents** — AI agent policies, such as **Copilot cloud agent** (formerly Copilot coding agent).
   - **MCP** — Model Context Protocol (MCP) policies.
3. Set each policy:
   - **Dropdown:** open it and choose an enforcement option — **Enabled**, **Disabled**, or **Let organizations decide** (each organization owner decides in 8C).
   - **Toggle:** click it.
   - **No visible control:** click the policy name to see its options.
4. Check that each policy shows the value you chose. *(Changes apply on selection — there is no Save button.)*

> ⏰ **Before October 22, 2026:** decide the **Default policy for new features** on this same **Copilot** page (**Enabled**, **Disabled**, or **Let organizations decide**). From that date, GA features you've left **Unconfigured** follow it — and it's **Enabled** by default. Explicit choices are never overridden.

> 💡 **Suggestions matching public code:** agree on this setting with your legal team before you enable it.

### 8C — Set organization Copilot policies

**👤 Role:** GitHub **organization owner** · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Organizations** → *[organization]* → **Settings** → **Copilot** *(sidebar, under "Code, planning, and automation")*

1. Click **Policies** to set feature and privacy policies, or **Models** to choose which models beyond the basic set are available (some can add cost).
2. For each policy, open its dropdown and choose an enforcement option. *(Changes apply on selection.)*

> 📌 **Enterprise wins:** a policy the enterprise set in 8B can't be changed here — only policies left at **Let organizations decide** are editable by the organization.

### 8D — Assign seats in the organization

**👤 Role:** GitHub **organization owner** · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Organizations** → *[organization]* → **Settings** → **Copilot** *(sidebar, under "Code, planning, and automation")* → **Access**

1. If you see **Allow this organization to assign seats**, click it.
2. Click **Start adding seats**.
3. Choose who gets Copilot:
   - **Everyone:** select **Purchase for all members**, then in the "Confirm seats purchase for all members" dialog click **Purchase seats**.
   - **Specific people or teams:** select **Purchase for selected members**. In the "Enable Copilot access for users and teams" dialog, use the **Users and teams** tab to search for and add people or teams (or **Upload CSV** to add many at once), then click **Continue to purchase** → **Purchase seats**.

> 💡 **Hands-off seats:** Entra ID supports team synchronization. Once team synchronization is enabled for the organization, connect a GitHub team to an Entra ID group (Organization → **Teams** → *[team]* → **Settings** → under **Identity Provider Groups** open **Select Groups**, pick the group, and click **Save changes**), then give that team seats — when Entra ID adds someone to the group, they get a Copilot seat automatically.

> 💡 **Billing:** a seat is billed from the moment it's granted (prorated mid-cycle), whether or not the person uses Copilot yet.

### 8E — Assign Copilot Business licenses at the enterprise level

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** **Enterprises** page (github.com/settings/enterprises) → *[your enterprise]* → **Billing and licensing** → **Licensing** → **Manage** *(in the "Copilot" section)*

**Before you start:** set the **Policies for enterprise-assigned users** policy (8B), make sure the people are already members of the enterprise (organization members, or users you've invited to the enterprise), and create the enterprise team first if you're licensing a team.

1. Click the **All members** tab (individual users) or the **Enterprise Teams** tab.
2. Click **Assign licenses**.
3. Search for the users or enterprise teams, then click **Add licenses**.

> ✅ **Enterprise teams are generally available** (since June 2026). License an enterprise team and people gain or lose Copilot as they join or leave it.

> 💡 **When to use this route:** people who need Copilot but no organization access. Enterprise members who aren't in any organization usually don't consume a GitHub Enterprise Cloud license. Direct enterprise assignment is for **Copilot Business**.

> 📌 **One license per person:** someone assigned through both 8D and 8E uses **one** license (the highest tier).

---

## 9️⃣ Critical Post-Enablement: SSO Authorization for Credentials (Required)

When SAML is enabled/enforced, users often must authorize credentials (depending on token type and whether they have a linked external identity).

### 9A — Authorize SSH Keys for SSO

**👤 Role:** Individual member · **📍 Portal:** GitHub

**Required for SSH usage in SSO orgs**

**Navigate:** Profile picture → **Settings** → **SSH and GPG keys** → *(next to the key)* **Configure SSO** → **Authorize** (for the org)

### 9B — Authorize Personal Access Tokens

**👤 Role:** Individual member · **📍 Portal:** GitHub

**Required for PAT classic in SSO orgs**

**Navigate:** Profile picture → **Settings** → **Developer settings** → **Personal access tokens** → *(next to the token)* **Configure SSO** → **Authorize** (for the org)

**Token nuance:**

> 📌 **Note:** GitHub states **PAT classic** requires post-creation SSO authorization. **Fine-grained PATs** are authorized during creation, before org access is granted.

---

## 🔟 Recovery Codes & Break-Glass Access

> 🔐 **Critical:** Recovery codes are your safety net if Entra ID becomes unavailable.

### Where to Find Recovery Codes

**Navigate:** Profile picture → **Organizations** → *[your org]* → **Settings** → **Authentication security** → under **SAML single sign-on**, click **Save your recovery codes** → **Download**

### Best Practices

- Download recovery codes immediately after enabling SAML
- Store in a secure vault (e.g., Azure Key Vault, hardware security module, or approved password manager)
- Ensure at least two org owners have access to recovery codes
- Test recovery code access periodically
- Regenerate codes if any are used or if personnel with access leave the organization

---

## ✅ Pre-Flight / Validation Checklist

### Before Starting

- Organization slug: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
- SCIM setup user credentials stored securely: ☐
- SCIM setup user 2FA enabled with recovery codes saved: ☐
- Entra ID admin with privileges to create/configure Enterprise Applications: ☐
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
- [ ] OAuth app authorized for the org during Entra "Test Connection" (no GitHub-generated token needed)
- [ ] Entra ID "Test Connection" succeeded and configuration saved with **Create**
- [ ] Provisioning started and initial cycle completed
- [ ] Assign/unassign test behaves as expected
- [ ] Provisioning logs show no errors

### Azure Billing

- [ ] Azure subscription successfully connected under Payment information
- [ ] "Metered billing via Azure" shows the correct subscription ID

### Copilot

- [ ] Enterprise Copilot access set under Billing and licensing → Licensing → Manage (if enterprise-managed)
- [ ] Enterprise Copilot policies set under AI controls → Copilot (if applicable)
- [ ] Org Copilot policies set
- [ ] Licenses assigned to pilot cohort (org- or enterprise-level)
- [ ] Pilot users can use Copilot in IDE / GitHub.com as expected

---

## 🎯 Success Criteria

After completing this guide, you should have:

- ✅ Standard GHEC organization fully configured with Microsoft Entra ID SAML authentication
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
| **SCIM test connection fails** | Tenant URL is wrong, the OAuth app was never authorized, or the setup user lacks an active org SAML session / owner role. | Paste the exact Tenant URL, then during **Test Connection** sign in as the setup user (with an active org SAML session) and complete the GitHub **Authorize** dialog for the target org. Standard-org SCIM uses a GitHub-authorized OAuth app — there is no token to regenerate. |
| **Provisioned users are missing from GitHub** | Users or groups are not assigned to the IdP app, attribute mappings fail, or provisioning cycles have not completed. | Review IdP provisioning logs, fix mapping errors, assign a small pilot group, and wait for the next incremental provisioning cycle. |
| **Azure billing connection fails** | The Azure signer cannot grant tenant consent or does not own the subscription. | Use a subscription owner with tenant consent rights or run the Entra admin consent workflow, then repeat the GitHub Add Azure Subscription flow. |
| **Copilot controls or seats are not visible** | Copilot is not enabled for the enterprise/org, the signed-in user lacks owner/admin permissions, or the plan/add-on is not active. | Verify Copilot plan activation, enable access at the enterprise/org level, and assign seats from the documented access page. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>

### Q: Users say they cannot access org resources after SAML was enabled — but they have GitHub accounts. What is wrong?
**A:** In standard (non-EMU) GHEC, users must link their personal GitHub account to their IdP identity by completing the SAML SSO flow at least once. Simply having a GitHub account is not enough. Direct them to `https://github.com/orgs/YOUR_ORG/sso` to authenticate. If SAML is enforced and they have not completed SSO, they will be removed from the org and must re-authenticate to rejoin (access is restored if they rejoin within three months).

---

### Q: A developer's `git push` fails with a 403 after SAML enforcement — what do they need to do?
**A:** After SAML enforcement, Personal Access Tokens (classic) must be individually authorized for the SSO-enabled org. Navigate to Settings > Developer settings > Personal access tokens, find the token, click "Configure SSO," and authorize it for the org. Fine-grained PATs are authorized during creation and do not need this extra step. SSH keys also require SSO authorization under Settings > SSH and GPG keys > Configure SSO.

---

### Q: My SSH keys are not working after SAML enforcement — how do I fix this?
**A:** SSH keys must be explicitly authorized for each SSO-enabled organization. Go to GitHub Settings > SSH and GPG keys, find your key, click "Configure SSO," and click "Authorize" next to the relevant org. This must be done for every SSH key that needs access to the org's repositories. This step is separate from adding the key to your GitHub account.

---

### Q: SAML enforcement unexpectedly removed members from the org — including bots and service accounts. How do I prevent this?
**A:** Enforcement removes every org member who hasn't authenticated through the IdP — including bots and service accounts without IdP identities. Before enforcing, review the list of unauthenticated members GitHub shows you. For service accounts, either:

1. Create IdP identities for them and have them complete SSO, or
2. Replace bot accounts with GitHub Apps, which SAML enforcement doesn't affect.

---

### Q: The SCIM OAuth authorization keeps dropping and provisioning stops — what do I do?
**A:** Org SCIM in standard GHEC runs on a **third-party-owned OAuth app** that a GitHub org owner authorized. There's no GitHub-generated SCIM token to copy or regenerate, and the authorization is tied to the person who granted it. Provisioning can break if that person's org membership changes or their SAML session lapses — so keep them an active org owner.

To reconnect:

1. Sign in as that user and visit `https://github.com/orgs/YOUR_ORG/sso` to refresh the SAML session.
2. In Entra, open the app → **Provisioning** → **Test Connection**.
3. Complete the GitHub **Authorize** dialog for the organization again.

---

### Q: There is an email mismatch between Entra ID and GitHub — users are getting duplicate identities. How do I resolve this?
**A:** In standard GHEC, the first successful SSO links the NameID Entra sends to that person's GitHub account — their GitHub email doesn't need to match. Problems start when the NameID **changes** (for example, you switch the claim from UPN to mail): GitHub sees a different identity and the sign-in fails or conflicts.

1. Pick one stable attribute (UPN **or** mail) for **Unique User Identifier (Name ID)** in the Entra app's **Attributes & Claims**, and don't change it.
2. For an affected member: Organization → **People** → the member → **SAML identity linked** → **Revoke** → **Revoke external identity**.
3. Ask them to sign in through SSO again to link the current identity.

---

### Q: We have an enterprise account — should we configure SAML at the enterprise level or the org level?
**A:**

- **Enterprise-level SAML:** consistent enforcement across all organizations. It overrides org-level SAML.
- **Org-level SAML:** different IdPs per organization, or a gradual rollout.

You can't combine them for one organization — enterprise SAML replaces org SAML.

**Important trade-off:** once SAML is enforced at the enterprise level, org-level SCIM (Step 6) isn't available and existing org SCIM stops working; enterprises with personal accounts have no SCIM at all. If you depend on org SCIM, keep SAML at the org level — or move to Enterprise Managed Users.

---

### Q: Entra ID provisioning logs show errors but users seem to be in the org — should I be concerned?
**A:** Yes. Common "silent" errors include failed attribute updates and deprovisioning failures. These can lead to stale memberships or incorrect role assignments over time. Review the provisioning logs in Entra (Enterprise Application > Provisioning > Provisioning logs) and filter for failures. Address mapping errors and ensure the SCIM endpoint is responding correctly. A healthy provisioning integration should show zero errors in steady state.

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

- [About SCIM for organizations](https://docs.github.com/en/organizations/managing-saml-single-sign-on-for-your-organization/about-scim-for-organizations)
- [Microsoft Entra ID: GitHub Enterprise Cloud tutorial](https://learn.microsoft.com/en-us/entra/identity/saas-apps/github-tutorial)
- [About identity and access management with SAML single sign-on](https://docs.github.com/en/organizations/managing-saml-single-sign-on-for-your-organization/about-identity-and-access-management-with-saml-single-sign-on)

---

*Last updated: October 2026*
