# 🚀 GitHub EMU + Okta (SAML), Azure Billing & Copilot Setup Runbook

> **Complete end-to-end runbook for configuring EMU with Okta (SAML), Azure billing, and GitHub Copilot**

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [📋 Overview](#-overview)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [1️⃣ Create & Configure the EMU Setup User](#1️⃣-create--configure-the-emu-setup-user)
- [2️⃣ Create the Okta Application](#2️⃣-create-the-okta-application)
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
- [📝 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Okta:** Applications → Browse App Catalog → "GitHub Enterprise Managed User" → Add Integration → Sign On tab → Set Enterprise Name to your slug
- **GitHub:** Sign in as `SHORTCODE_admin` → Enterprise → Identity provider → Add SAML configuration → Paste Okta Sign-on URL, Issuer, X.509 cert → Save
- **SCIM:** As `SHORTCODE_admin`, generate PAT with `scim:enterprise` scope → Okta App → Provisioning → Enable API Integration → Paste token → Save
- **Billing:** Enterprise → Billing and licensing → Payment information → Add Azure Subscription → Accept permissions → Connect
- **Copilot:** Enterprise → **Billing and licensing** → **Licensing** → Copilot **Manage** → turn on organizations → **AI controls** → **Copilot** (policies) → seats: Org **Settings** → **Copilot** → **Access** → **Start adding seats**, or the enterprise **Manage** page → **Assign licenses**

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

This guide walks through setting up a new GitHub Enterprise Cloud (GHEC) with Enterprise Managed Users (EMU), including:
- ✅ EMU with Okta (SAML SSO)
- ✅ SCIM provisioning for user lifecycle management
- ✅ Azure subscription attachment for billing
- ✅ GitHub Copilot enablement
- ✅ Organization structure guidance

**Hosting Options:** EMU can be hosted on GitHub.com or (for data residency) on a customer subdomain of GHE.com. Setup is similar, but SAML/SCIM URLs and Okta app selection differ.

---

## ✅ Prerequisites

### Roles & who does what

Before beginning, ensure you have:

| Requirement | Who / Role needed | ✓ |
|-------------|-------------------|---|
| EMU enterprise created — a new enterprise with EMU enabled | GitHub **enterprise owner** | ☐ |
| Setup user — `SHORTCODE_admin`, created by GitHub, password set and 2FA enabled | GitHub EMU setup user (`SHORTCODE_admin`) | ☐ |
| Okta app + SAML SSO + SCIM | Okta **application admin** (can create App Integrations and configure SAML + SCIM) | ☐ |
| Connect Azure subscription for billing | GitHub **enterprise owner** + Azure **subscription Owner** (with tenant-wide admin consent) | ☐ |
| Enable Copilot | GitHub **enterprise owner** | ☐ |

> 💡 **Note:** SCIM provisioning is required for EMU to manage user lifecycle and account creation. GitHub fully supports using **one partner IdP** (here, Okta) for both SAML and SCIM. Mixing identity systems isn't expressly supported, and **Okta + Entra ID** (in either order) is explicitly not supported — GitHub's SCIM API returns an error.

---

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **Okta application admin** | Creates the GitHub EMU app, captures SAML values, configures SCIM, and assigns users or groups. | Okta Admin Console → Applications → Applications → Browse App Catalog → search GitHub Enterprise Managed User or GitHub Enterprise Managed User - GHE.com → Add Integration → Assignments → Assign → assign your setup/pilot admin → Sign On → Enterprise Name → enter enterprise slug → SAML 2.0 → More details → capture Sign on URL, Issuer, and Signing certificate. Then Provisioning → Integration → Edit → Configure API Integration → API Token → paste setup-user PAT → Test API Credentials → Save → To App → Edit → enable Create Users, Update User Attributes, and Deactivate Users → Save → Assignments or Push Groups. Handoff: SAML values, test API success, assigned pilot group. |
| **GitHub EMU setup user (`SHORTCODE_admin`)** | Pastes Okta SAML values into GitHub and generates the SCIM token for Okta. | GitHub → profile picture → Enterprise → Identity provider → Single sign-on configuration → Add SAML configuration → Sign on URL, Issuer, Public Certificate → Test SAML configuration → Save. For SCIM token: setup user → Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token (classic) → `scim:enterprise` → Generate token. Handoff: SCIM token, Tenant URL, and recovery codes. |
| **Okta group owner** | Controls who is provisioned and what role they receive. | Okta Admin Console → Directory → Groups → [group] → People → Assign people → select users → Save, then Applications → Applications → [GitHub EMU app] → Assignments → Assign → Assign to Groups → select group → set role attributes if required → Done. Handoff: assigned group and role attribute values. |
| **GitHub enterprise or organization owner** | Starts the Azure metered billing connection from GitHub. | Enterprise path: GitHub → profile picture → Enterprise → Billing and licensing → Payment information → Metered billing via Azure → Add Azure Subscription. Organization path: GitHub → profile picture → Organizations → [organization] → Settings → Billing and licensing → Payment information → Metered billing via Azure → Add Azure Subscription. Then sign in to Microsoft → Permissions requested → Accept → Select a subscription → Connect. Handoff: the subscription ID is visible on Payment information. |
| **Azure subscription Owner** | Provides the Azure subscription that GitHub will bill against, or grants another signer the required Azure RBAC rights. | Azure portal → Subscriptions → [subscription] → Access control (IAM) → Role assignments → confirm the signer is listed under Owner. To grant access: Add → Add role assignment → Privileged administrator roles → Owner → Members → Select members → [user] → Select → Review + assign. Handoff: subscription ID and tenant ID. |
| **Microsoft Entra Global Administrator or consent approver** | Approves tenant-wide consent when the Microsoft consent prompt blocks the GitHub billing app. | Microsoft Entra admin center → Entra ID → Enterprise apps → Activity → Admin consent requests → My Pending → [GitHub request] → Review permissions and consent → Approve. If the Global Administrator completes the GitHub flow directly, approve the Permissions requested prompt by clicking Accept. |

---

## 1️⃣ Create & Configure the EMU Setup User

**👤 Role:** GitHub EMU setup user (`SHORTCODE_admin`) · **📍 Portal:** GitHub

The setup user's username is your enterprise **shortcode** + `_admin` (for example, `octocorp_admin`). The shortcode is chosen at creation (or randomly assigned) and **cannot be changed later**.

### Process

1. GitHub emails an invite to set the password for `SHORTCODE_admin`.
2. In a private/incognito window, set the password, then enable 2FA immediately:

**Navigate:** Profile picture → **Settings** → **Password and authentication**

1. Under **Two-factor authentication**, click **Enable two-factor authentication**.
2. Choose a method — **Set up using an app** (TOTP authenticator app recommended).
3. Scan the QR code (or enter the setup key) in your authenticator app, then enter the 6-digit code to **complete the challenge**.
4. On the **recovery codes** screen, click **Download** (or **Copy**/**Print**) to save your **personal 2FA recovery codes**.
5. Click **I have saved my recovery codes** / **Continue** to finish. Store the codes in your vault.
6. Download the **enterprise recovery codes** (separate from your personal 2FA codes): Profile picture → **Enterprise** → **Identity provider** → **Single sign-on configuration** → under **SAML single sign-on**, click **Save your recovery codes** → **Download** (or **Print** / **Copy**) → store them in your vault. *(If the link isn't shown yet, you'll be prompted to save them when you save your SAML configuration in Step 3.)*

### Important Notes

> ⚠️ **Setup User Purpose:** This account is primarily the SCIM token owner and a break-glass account. Day-to-day enterprise administration should be done with provisioned managed **enterprise-owner** accounts.

> 🔐 **2FA required on every sign-in (Jan 2025 change):** Every setup-user sign-in requires a successful 2FA challenge **or** an enterprise recovery code. Losing both sets of codes locks you out. A setup-user password reset must go through **GitHub Support** — the standard email reset does not work.

> 🚨 **Email Conflict:** If the provided email address is already associated as a primary email with an existing GitHub account, the activation link will not work. Modify the existing account's primary email first.

---

## 2️⃣ Create the Okta Application

**👤 Role:** Okta application admin · **📍 Portal:** Okta

**Navigate:** Okta Admin Console → **Applications** → **Applications** → **Browse App Catalog** (or **Browse App Integration Catalog**) → search **GitHub Enterprise Managed User** → select the correct integration → **Add Integration**

### Select the Correct Okta Integration

- **For GitHub.com hosted EMU:** install **GitHub Enterprise Managed User**
- **For GHE.com hosted EMU (Data Residency):** install **GitHub Enterprise Managed User - GHE.com**

### SAML Configuration

**Set Enterprise Name:**

**Navigate:** **Okta Admin Console** → **Applications** → **Applications** → **GitHub Enterprise Managed User** → **Sign On** tab

1. Click **Edit** (top-right of the **Settings** section).
2. In **Enterprise Name**, type your enterprise slug (e.g., `octocorp`).
3. Click **Save**.

> 📌 **Note:** On the **Sign On** tab the SAML settings are read-only until you click **Edit**; the **Enterprise Name** field only becomes editable after that.

> 💡 **Note:** Enter your enterprise **slug** (e.g., "octocorp" if your enterprise URL is `github.com/enterprises/octocorp` or `octocorp.ghe.com`).

### Download Required Items (Okta IdP values)

From the Okta application, gather the three SAML values:

1. **Sign on URL** (IdP Sign-On URL)
2. **Issuer**
3. **Signing certificate** (X.509)

**Navigate:** **Okta Admin Console** → **Applications** → **Applications** → **GitHub Enterprise Managed User** → **Sign On** tab

1. Under **SAML 2.0**, click **More details**.
2. Copy the **Sign on URL** and **Issuer**.
3. Click **Download certificate** to save the X.509 signing certificate.

> 💡 **Tip:** The X.509 signing certificate is exposed as a **Download certificate** button/link (a `.cert`/`.pem` file), not plain copyable text like the URLs — click **Download certificate** (or open its contents and copy the full certificate text) for pasting into GitHub's **Public Certificate** field in Step 3. On the **Sign On** tab you can also click **View SAML setup instructions** to see all three values together.

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

**Navigate:** **Okta Admin Console** → **Applications** → **Applications** → **GitHub Enterprise Managed User** → **Assignments** tab

1. Click **Assign** → **Assign to People** (or **Assign to Groups**).
2. Click **Assign** next to each user or group, fill in any requested attributes (such as the role), and click **Save and Go Back**.
3. Click **Done**.

---

## 3️⃣ Enable SAML SSO in GitHub

**👤 Role:** GitHub EMU setup user (`SHORTCODE_admin`) · **📍 Portal:** GitHub

> ⚠️ **Warning:** Enabling SAML impacts how members authenticate. Enterprise Managed Users does not provide a backup username/password sign-in URL, and there is **no "Require SAML authentication" checkbox** — if SAML fails, use enterprise SSO recovery codes or contact GitHub Enterprise Support.

**Navigate:** Profile picture → **Enterprise** → **Identity provider** (top tab) → **Single sign-on configuration** → under **SAML single sign-on**, **Add SAML configuration**

### Configuration Steps

1. Under **SAML single sign-on**, click **Add SAML configuration**.
2. Enter the following values from Okta:
   - **Sign on URL** — Paste the Sign on URL from Okta.
   - **Issuer** — Paste the Issuer from Okta.
   - **Public Certificate** — Paste the contents of the Okta signing certificate (X.509).
   - **Signature Method** — Choose from the dropdown (SHA-256 recommended).
   - **Digest Method** — Choose from the dropdown (SHA-256 recommended).
3. Click **Test SAML configuration** to validate the setup (it must pass before you can save).
4. Click **Save SAML settings**.
5. Immediately **Download**, **Print**, or **Copy** your enterprise **SSO recovery codes** and store them securely, then click **I have saved my recovery codes** (or **Continue**/**Done**) to dismiss the recovery-codes screen and finish enabling SAML.

> 🔐 **Critical:** Recovery codes are essential for break-glass scenarios if your IdP becomes unavailable.

---

## 4️⃣ Configure SCIM Provisioning

SCIM handles user creation and deactivation in EMU. This requires creating a token in GitHub and configuring provisioning in Okta.

### 4A — Create the SCIM Token in GitHub

**👤 Role:** GitHub EMU setup user (`SHORTCODE_admin`) · **📍 Portal:** GitHub

The token must be created **while signed in as the setup user** (the setup user is an enterprise owner), with specific requirements:

**Token Requirements:**
- ✓ Type: **personal access token (classic)**
- ✓ Scope: `scim:enterprise` (only)
- ✓ Expiration: **No expiration** (recommended — if it expires, provisioning stops)
- ✓ Created by: `SHORTCODE_admin`

**Navigate:** Profile picture → **Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)** → **Generate new token (classic)**

**Configuration:**
1. **Note** — Enter "Okta SCIM Provisioning" (or a similar descriptive name).
2. **Expiration** — Select **No expiration**.
3. **Scope** — Select `scim:enterprise` only.
4. Click **Generate token**.

> 🔑 **Important:** Copy the token immediately after generation. You won't be able to see it again.

### 4B — Configure Provisioning in Okta

**👤 Role:** Okta **application admin** · **📍 Portal:** Okta Admin Console

**Navigate:** **Okta Admin Console** → **Applications** → **Applications** → **GitHub Enterprise Managed User** → **Provisioning** tab → **Integration** → **Configure API Integration** *(or **Edit**, if it's already configured)*

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

**Navigate:** same **Provisioning** tab → **To App** → **Edit**

1. Check **Create Users**, **Update User Attributes**, and **Deactivate Users**.
2. Click **Save**.

> 📌 **Note:** Okta's "Import Groups" setting is **not supported** by GitHub for EMU and checking/unchecking it has **no impact** on behavior.

### 4C — Configure Attribute Mappings

**👤 Role:** Okta **application admin** · **📍 Portal:** Okta Admin Console

Okta generally provides correct default mappings for the GitHub EMU integration. If you need to review/edit mappings:

**Navigate:** **Okta Admin Console** → **Applications** → **Applications** → **GitHub Enterprise Managed User** → **Provisioning** tab → **To App** → scroll to the **Attribute Mappings** section *(edit a mapping with its pencil icon, or use **Go to Profile Editor**)*

- Review default user attribute mappings (typically sufficient for most deployments).

### 4D — Assign Users/Groups for Provisioning

1. Navigate to **Assignments** in the Okta app
2. Assign users and/or groups to the application
3. Okta will SCIM-provision these members into the EMU enterprise

**Navigate:** **Okta Admin Console** → **Applications** → **Applications** → **GitHub Enterprise Managed User** → **Assignments** tab

1. Click **Assign** → **Assign to People** (or **Assign to Groups**).
2. Click **Assign** next to each user or group, fill in any requested attributes (such as the role), and click **Save and Go Back**.
3. Click **Done**.

**Provisioning Notes:**

> 📌 **Important Constraints:**
> - To avoid exceeding GitHub's rate limit, **do not assign more than 1,000 users per hour** to the SCIM integration.

---

## 5️⃣ Attach Azure Subscription for Billing

Connect your Azure subscription so GitHub usage (Copilot, Actions, Codespaces, etc.) is billed through Azure.

**👤 Role:** GitHub **enterprise owner** + Azure **subscription Owner** · **📍 Portal:** GitHub → Microsoft

### Prerequisites

- ✓ **Enterprise owner** of the GitHub enterprise account (a billing manager is **not** sufficient to connect a subscription)
- ✓ Azure Subscription ID
- ✓ Azure **subscription Owner** who can provide tenant-wide admin consent (or coordinate with a Microsoft Entra **Global Administrator** / admin consent workflow)

### Configuration Steps

**Navigate:** Profile picture → **Enterprise** → **Billing and licensing** → **Payment information** → scroll to **Metered billing via Azure** → **Add Azure Subscription**

**Process:**

1. Click **Add Azure Subscription**.
2. Sign in to your Microsoft account (if prompted).
3. Review the **Permissions requested** prompt.
4. Click **Accept**.
5. Under **Select a subscription**, choose your Azure subscription.
6. Check the confirmation box (for example, "By clicking 'Connect', you are confirming...").
7. Click **Connect**.

> ✅ **Result:** The connected subscription ID is visible on the **Payment information** page.

> 💡 **Admin Consent:** If you don't see a **Permissions requested** prompt and instead see a message about needing admin approval, configure an admin consent workflow in Azure or work with a Microsoft Entra **Global Administrator**.

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
   - **Dropdown:** open it and choose an enforcement option — **Enabled**, **Disabled**, or **No policy** (lets each organization owner decide).
   - **Toggle:** click it.
   - **No visible control:** click the policy name to see its options.
4. Check that each policy shows the value you chose. *(Changes apply on selection — there is no Save button.)*

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

> 💡 **Hands-off seats:** give seats to a team that's linked to an Okta group (see Step 7). When Okta adds someone to the group, SCIM adds them to the team and they get a Copilot seat automatically.

> 💡 **Billing:** a seat is billed from the moment it's granted (prorated mid-cycle), whether or not the person uses Copilot yet.

**Route 2 — Enterprise licenses** (enterprise owner · Copilot Business · no organization membership required)

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Enterprise** → **Billing and licensing** → **Licensing** → **Manage** *(in the "Copilot" section)*

**Before you start:** set the **Policies for enterprise-assigned users** policy (6B), make sure the people already exist in the enterprise (provisioned by SCIM), and create the enterprise team first if you're licensing a team.

1. Click the **All members** tab (individual users) or the **Enterprise Teams** tab.
2. Click **Assign licenses**.
3. Search for the users or enterprise teams, then click **Add licenses**.

> ✅ **Enterprise teams are generally available** (since June 2026). License an enterprise team and people gain or lose Copilot as they join or leave it. With Enterprise Managed Users you can sync the enterprise team to an Okta group, so licensing is driven entirely from your IdP.

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
- GitHub **enterprise owner** available to connect the subscription
- Azure **subscription Owner** or admin who can grant tenant-wide consent

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
| 3 | Connecting an Azure subscription | [docs.github.com](https://docs.github.com/en/enterprise-cloud@latest/billing/managing-the-plan-for-your-github-account/connecting-an-azure-subscription) |
| 4 | Copilot policies for enterprise | [docs.github.com](https://docs.github.com/en/enterprise-cloud@latest/copilot/managing-copilot/managing-copilot-for-your-enterprise/managing-policies-and-features-for-copilot-in-your-enterprise) |
| 5 | Okta GitHub EMU integration | [saml-doc.okta.com](https://saml-doc.okta.com/SAML_Docs/How-to-Configure-SAML-2.0-for-GitHub-Enterprise-Managed-User.html) |

---

*Last updated: October 2026*
