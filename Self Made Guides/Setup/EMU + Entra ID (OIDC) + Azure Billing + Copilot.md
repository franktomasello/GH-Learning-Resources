# 🚀 GitHub EMU + Microsoft Entra ID (OIDC), Azure Billing & Copilot Setup Runbook

> **Complete end-to-end guide for configuring EMU with Microsoft Entra ID (OIDC), Azure billing, and GitHub Copilot**

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [📋 Overview](#-overview)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [1️⃣ Create/Secure the EMU Setup User](#1-createsecure-the-emu-setup-user)
- [2️⃣ Create the SCIM Token](#2-create-the-scim-token)
- [3️⃣ Enable Microsoft Entra ID OIDC SSO](#3-enable-microsoft-entra-id-oidc-sso)
- [4️⃣ Configure SCIM Provisioning](#4-configure-scim-provisioning)
- [5️⃣ Create Organization(s) Inside the Enterprise](#5-create-organizations-inside-the-enterprise)
- [6️⃣ Connect Entra ID Groups to GitHub Teams](#6-connect-entra-id-groups-to-github-teams)
- [7️⃣ Connect Azure Subscription (Billing)](#7-connect-azure-subscription-billing)
- [8️⃣ Enable Copilot & Assign Seats](#8-enable-copilot--assign-seats)
- [9️⃣ Validation Checklist](#9-validation-checklist)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)
- [📝 Source Documentation](#-source-documentation)
- [📝 Revision History](#-revision-history)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **GitHub:** Sign in as `SHORTCODE_admin` → Enterprise → Identity provider → Single sign-on → Enable OIDC configuration → Save (redirects to Entra)
- **Entra ID:** Sign in as Global Admin → Consent on behalf of organization → Accept (auto-creates the OIDC Enterprise App)
- **SCIM:** As `SHORTCODE_admin`, generate PAT with `scim:enterprise` scope → Entra App → Provisioning → Automatic → Enter Tenant URL + token → Test Connection
- **Billing:** Enterprise → Billing and licensing → Payment information → Metered billing via Azure → Add Azure Subscription → Accept → Connect
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

**Scope:**

- **Identity:** Microsoft Entra ID (OIDC) for SSO
- **Provisioning:** SCIM (Automatic User Provisioning)
- **Billing:** Azure Subscription (Metered)
- **Copilot:** Business/Enterprise (Enable & Assign via Teams or Enterprise Teams)

---

## ✅ Prerequisites

### People & Permissions

| Requirement | Who / Role needed | ✓ |
|------|---------------------|:--:|
| **GitHub enterprise access** | Access to the setup user (`@SHORTCODE_admin`) email to set the initial password. The setup user is a GitHub **enterprise owner**. | ☐ |
| **Entra OIDC consent** | A Microsoft Entra **Global Administrator** to consent to the "GitHub Enterprise Managed User (OIDC)" app during SSO setup | ☐ |
| **Entra SCIM provisioning** | Microsoft Entra **Application Administrator, Cloud Application Administrator, or Application Owner** (of the OIDC app) to configure provisioning and assignments | ☐ |
| **GitHub billing connection** | A GitHub **enterprise owner** to start the Azure metered billing connection | ☐ |
| **Azure billing** | A user with **Owner** permission on the target Azure Subscription **AND** able to provide **tenant-wide admin consent** (or coordinate with a Global Administrator) | ☐ |
| **GitHub Copilot** | A GitHub **enterprise owner** to manage Copilot access, policies, and seats | ☐ |

> ⚠️ **Important:** Being an Entra Global Admin alone is **not sufficient** for the billing step. You must also have Owner permissions on the specific Azure Subscription resource.

### Hosting Location

| Environment | Description |
|-------------|-------------|
| **GitHub.com** | Standard cloud hosting |
| **GHE.com** | Data Residency (dedicated subdomain) |

This affects your SCIM Tenant URL format in Step 4.

### Before You Begin Checklist

- [ ] Setup user invitation email received
- [ ] Access to Microsoft Entra admin center
- [ ] Azure subscription ID ready
- [ ] Secure password manager for storing recovery codes and tokens

---

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **GitHub EMU setup user (`SHORTCODE_admin`)** | Starts OIDC SSO from GitHub and creates the SCIM token. | GitHub → profile picture → Enterprise → Identity provider → Single sign-on configuration → OIDC single sign-on → Enable OIDC configuration → Save → complete Entra redirect. For SCIM: setup user → Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token (classic) → select `scim:enterprise` → Generate token. Handoff: SCIM token and Tenant URL. |
| **Microsoft Entra Global Administrator** | Consents to the GitHub Enterprise Managed User (OIDC) application. | During the GitHub redirect, sign in as Global Administrator → review Permissions requested → Consent on behalf of your organization if shown → Accept. If consent is blocked: Microsoft Entra admin center → Entra ID → Enterprise apps → Activity → Admin consent requests → My Pending → [GitHub Enterprise Managed User (OIDC)] → Review permissions and consent → Approve. Handoff: OIDC enterprise app exists and consent is granted. |
| **Microsoft Entra Application Administrator, Cloud Application Administrator, or Application Owner** | Configures SCIM provisioning and app assignments after OIDC consent. | Microsoft Entra admin center → Entra ID → Enterprise apps → GitHub Enterprise Managed User (OIDC) → Provisioning → + New configuration (older tenants: Get started, Provisioning Mode: Automatic) → Admin Credentials → Tenant URL and Secret Token → Test Connection → Create (older UI: Save) → Users and groups → Add user/group → Assign → Provisioning → Start provisioning. Handoff: successful test connection, assigned pilot group, and provisioning logs. |
| **GitHub enterprise or organization owner** | Starts the Azure metered billing connection from GitHub. | Enterprise path: GitHub → profile picture → Enterprise → Billing and licensing → Payment information → Metered billing via Azure → Add Azure Subscription. Organization path: GitHub → profile picture → Organizations → [organization] → Settings → Billing and licensing → Payment information → Metered billing via Azure → Add Azure Subscription. Then sign in to Microsoft → Permissions requested → Accept → Select a subscription → Connect. Handoff: the subscription ID is visible on Payment information. |
| **Azure subscription Owner** | Provides the Azure subscription that GitHub will bill against, or grants another signer the required Azure RBAC rights. | Azure portal → Subscriptions → [subscription] → Access control (IAM) → Role assignments → confirm the signer is listed under Owner. To grant access: Add → Add role assignment → Privileged administrator roles → Owner → Members → Select members → [user] → Select → Review + assign. Handoff: subscription ID and tenant ID. |
| **Microsoft Entra Global Administrator or consent approver** | Approves tenant-wide consent when the Microsoft consent prompt blocks the GitHub billing app. | Microsoft Entra admin center → Entra ID → Enterprise apps → Activity → Admin consent requests → My Pending → [GitHub request] → Review permissions and consent → Approve. If the Global Administrator completes the GitHub flow directly, approve the Permissions requested prompt by clicking Accept. |

---

## 1️⃣ Create/Secure the EMU Setup User

*The setup user (`@SHORTCODE_admin`) is the only local account that can bypass SSO in emergencies.*

**👤 Role:** EMU setup user (`SHORTCODE_admin`) · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Settings** → **Password and authentication**

### Steps

1. Open the "setup user invite" email in a **private/incognito browser window**.
2. Set a strong password (store in a secure vault).
3. **Immediately enable 2FA:** go to Profile picture → **Settings** → **Password and authentication**, then:
   1. Under **Two-factor authentication**, click **Enable two-factor authentication**.
   2. Choose **Set up using an app**.
   3. Scan the displayed **QR code** with your authenticator app (or click **enter this text code** to copy the setup key).
   4. Enter the **6-digit code** from the app and click **Continue**.
   5. On the **recovery codes** screen, click **Download** (and/or **Copy** / **Print**), store them in your vault, then click **I have saved my recovery codes** to finish.
4. Download the **enterprise recovery codes** (separate from your personal 2FA codes): Profile picture → **Enterprise** → **Identity provider** → **Single sign-on configuration** → under **SAML single sign-on** or **OIDC single sign-on**, click **Save your recovery codes** → **Download** (or **Print** / **Copy**). *(If the link isn't shown yet, you'll be prompted to save them during OIDC setup in Step 3.)*
5. Store credentials in a secure company vault (e.g., 1Password, LastPass, Azure Key Vault).

> 🔐 **Note on the shortcode:** The username is your enterprise **shortcode** + `_admin` (e.g., `octocorp_admin`). The shortcode is chosen (or randomly assigned) at creation and **cannot be changed later**.

> ⚠️ **Critical (January 2025 Change):** All subsequent logins for the setup user require either a successful 2FA challenge OR use of an enterprise recovery code. If you do not save your enterprise recovery codes (generated in Step 3), you will be locked out.

> 📝 **Note:** If you ever need to reset the setup user password, you must contact [GitHub Support](https://support.github.com/). The standard email-based password reset does not work for setup users.

---

## 2️⃣ Create the SCIM Token

*This token allows Microsoft Entra ID to provision users to GitHub via SCIM. It must be created while signed in as the setup user, who is an enterprise owner.*

**👤 Role:** EMU setup user (`SHORTCODE_admin`) · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)** → **Generate new token (classic)**

### Token Configuration

| Setting | Value |
|---------|-------|
| **Note** | `SCIM Token for Entra ID` (or similar) |
| **Expiration** | **No expiration** ⚠️ If this expires, provisioning stops entirely |
| **Scopes** | Select **only** `scim:enterprise` |

1. In the **Note** field, type `SCIM Token for Entra ID`.
2. Open the **Expiration** dropdown and select **No expiration** (confirm the warning).
3. In the scopes list, check the box for **`scim:enterprise`** only (leave all other scopes unchecked).
4. **Scroll to the bottom** and click **Generate token**.
5. Click the **copy** icon to copy the token immediately — it is shown only once and cannot be viewed again.

> 📝 **Store this token securely.** You will need it when configuring provisioning in Microsoft Entra ID (Step 4).

---

## 3️⃣ Enable Microsoft Entra ID OIDC SSO

*Connects identity so users can authenticate via your IdP.*

**👤 Role:** EMU setup user (`SHORTCODE_admin`), then Entra **Global Administrator** · **📍 Portal:** GitHub → Microsoft Entra

**Navigate:** Profile picture → **Enterprise** → **Identity provider** → **Single sign-on configuration**

### Steps (in GitHub)

1. Under **OIDC single sign-on**, select **Enable OIDC configuration**.
2. Click **Save** — GitHub redirects you to Microsoft Entra.

### In Microsoft Entra ID (during redirect)

3. Sign in as a user with **Global Administrator** rights.
4. Review the permissions requested for the **GitHub Enterprise Managed User (OIDC)** application.
5. Enable **Consent on behalf of your organization**.
6. Click **Accept**.

### Back in GitHub

7. **Save your enterprise recovery codes:** click **Download**, **Print**, or **Copy**. Store them securely — they are separate from your personal 2FA codes and allow emergency access if your IdP becomes unavailable.
8. Click **Enable OIDC Authentication**.

> 💡 **Why OIDC:** OIDC supports Microsoft Entra **Conditional Access**, which SAML does not pass to GitHub. After consent, a **GitHub Enterprise Managed User (OIDC)** enterprise app appears in the tenant; you configure SCIM provisioning on that same app (Step 4).

> ✅ **Verification:** After completing this step, the setup user can still access the enterprise using their local credentials, but all other users will authenticate via Entra ID.

---

## 4️⃣ Configure SCIM Provisioning

*Pushes users and groups from Microsoft Entra ID to GitHub automatically.*

### 4A) Determine Your Tenant URL

| Hosting | Tenant URL Format |
|---------|-------------------|
| **GitHub.com** | `https://api.github.com/scim/v2/enterprises/{enterprise-slug}` |
| **GHE.com** | `https://api.{subdomain}.ghe.com/scim/v2/enterprises/{subdomain}` |

> 💡 **Example:** If your enterprise URL is `https://github.com/enterprises/acme-corp`, your Tenant URL is `https://api.github.com/scim/v2/enterprises/acme-corp`

### 4B) Configure Microsoft Entra ID

**👤 Role:** Entra **Application Administrator, Cloud Application Administrator, or Application Owner** · **📍 Portal:** Microsoft Entra

**Navigate:** Microsoft Entra admin center → **Entra ID** → **Enterprise apps** → **GitHub Enterprise Managed User (OIDC)** → **Provisioning**

1. In the Microsoft Entra admin center, open **Enterprise apps**.
2. Select **GitHub Enterprise Managed User (OIDC)** (created automatically during Step 3).
3. Click the **Provisioning** tab, then click **+ New configuration** (older tenants show **Get started** and a **Provisioning Mode** dropdown — set it to **Automatic**).
4. Under **Admin Credentials**, enter:

| Field | Value |
|-------|-------|
| **Tenant URL** | Your URL from Step 4A |
| **Secret Token** | The PAT created in Step 2 |

5. Click **Test Connection** — must show success ✅.
6. Click **Create** (older UI: **Save**).
7. On the provisioning **Overview**, click **Edit provisioning** (or open **Provisioning → Properties**) and click the **pencil**. Enable **Send an email notification when a failure occurs** and enter a recipient address; enable **Prevent accidental deletion** and set a threshold. Click **Apply** (or **Save**).

### 4x) Review Attribute Mappings

**👤 Role:** Entra **Application Administrator, Cloud Application Administrator, or Application Owner** · **📍 Portal:** Microsoft Entra

**Navigate:** Microsoft Entra admin center → **Entra ID** → **Enterprise apps** → **GitHub Enterprise Managed User (OIDC)** → **Provisioning** → **Mappings**

1. In the app, open **Provisioning → Mappings** (or **Attribute Mapping**).
2. Click **Provision Microsoft Entra ID Users**, confirm the required attributes (userName, emails, name, externalId), then click **Save**.
3. Click **Provision Microsoft Entra ID Groups**, confirm the group mappings, then click **Save**.

### 4C) Configure Provisioning Settings

**👤 Role:** Entra **Application Administrator, Cloud Application Administrator, or Application Owner** · **📍 Portal:** Microsoft Entra

**Navigate:** Microsoft Entra admin center → **Entra ID** → **Enterprise apps** → **GitHub Enterprise Managed User (OIDC)** → **Provisioning** → **Settings**

| Setting | Value |
|---------|-------|
| **Scope** | Sync only assigned users and groups |
| **Provisioning Status** | **On** |

1. In the app, open **Provisioning**, then expand the **Settings** section.
2. Open the **Scope** dropdown and select **Sync only assigned users and groups**. *(Required — the Enterprise Owner app role assigned in Step 4D only provisions when Scope is set to this.)*
3. Click **Save**.
4. Return to **Provisioning → Overview** and click **Start provisioning** to begin the first sync cycle. *(In the older UI, set **Provisioning Status** to **On** and click **Save** first.)*

### 4D) Assign Users and Groups

1. Go to the **Users and groups** tab in the Entra application.
2. Click **Add user/group**.
3. Under **Users and groups**, click **None Selected**, pick your test user or pilot group, and click **Select**.
4. Click **Select a role**, choose **Enterprise Owner** for the first admin (or the member role for everyone else), and click **Select**.
5. Click **Assign**. Assign at least one user the **Enterprise Owner** role so a managed admin exists.

> 📌 **Constraint:** Role-based (app role) assignment requires the provisioning **Scope** to be **Sync only assigned users and groups** (set in Step 4C).

### 4E) Initial Sync

**👤 Role:** Entra **Application Administrator, Cloud Application Administrator, or Application Owner** · **📍 Portal:** Microsoft Entra admin center

| Method | Timing |
|--------|--------|
| **Automatic** | Wait ~40 minutes for the initial sync cycle |
| **Manual testing** | Use **Provision on demand** to test individual users immediately |

> ⚠️ **Important:** The combination of Okta and Entra ID for SSO and SCIM (in either order) is explicitly **not supported**.

---

## 5️⃣ Create Organization(s) Inside the Enterprise

*EMU enterprises cannot invite or import existing organizations — you must create them fresh.*

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Enterprise** → **Organizations** → **New organization**

1. Click **New organization**.
2. Enter the **Organization name** (e.g., `acme-engineering`, `acme-platform`).
3. Click **Create organization**.

> 💡 **Best Practice:** Plan your organization structure before creating. Consider separating by business unit, product line, or team function.

---

## 6️⃣ Connect Entra ID Groups to GitHub Teams

*This is the "Golden Path" for automated permissions management.*

**👤 Role:** GitHub **organization owner** · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Organizations** → *[organization]* → **Teams** → **New team**

1. Click **New team**.
2. Enter the **Team name** (e.g., `developers`, `platform-engineers`) and an optional **Description**.
3. Under **Identity Provider Groups**, select the synced Entra group from the dropdown.
4. Click **Create team**.

### Important Constraints

> ⚠️ **No Nested Teams:** When a team is linked to an IdP group, you cannot nest it under other teams.

> ⚠️ **Nested Groups in Entra:** Microsoft Entra ID does not support provisioning nested groups. Only direct members sync.

---

## 7️⃣ Connect Azure Subscription (Billing)

*Required for metered billing: Copilot, Actions minutes, Packages storage, GHAS, and Codespaces.*

**👤 Role:** GitHub **enterprise owner** (in GitHub) + Azure **subscription Owner** with tenant-wide admin consent (in Microsoft) · **📍 Portal:** GitHub → Microsoft

**Navigate:** Profile picture → **Enterprise** → **Billing and licensing** → **Payment information** → **Metered billing via Azure** → **Add Azure Subscription**

### Prerequisites Check

- [ ] You are a GitHub **enterprise owner** (a billing manager cannot connect a subscription)
- [ ] Tenant-wide admin consent capability in Azure
- [ ] **Owner** permission on the target Azure Subscription

### Azure Connection Flow

1. Click **Add Azure Subscription**.
2. Sign in with your Microsoft account.
3. Review the **Permissions requested** prompt.
4. Click **Accept**.
5. Under **Select a subscription**, choose the Azure Subscription.
6. Check the confirmation box.
7. Click **Connect**.

> ⚠️ **Troubleshooting:** If you see "You need admin approval" instead of the permissions prompt, work with your Azure AD Global Administrator to grant consent or configure an [admin consent workflow](https://learn.microsoft.com/en-us/azure/active-directory/manage-apps/configure-admin-consent-workflow).

### Verification

- ✅ Azure Subscription ID appears under "Payment information"
- ✅ You can now enable metered services like Copilot and GHAS

---

## 8️⃣ Enable Copilot & Assign Seats

Set up Copilot in this order: **8A** turn Copilot on for organizations, **8B** set policies, **8C** give people seats. With Azure metered billing connected (Step 7), Copilot charges bill to your Azure subscription.

> 📌 **Where things live:** organization access and licenses are under **Billing and licensing → Licensing**; policies are under **AI controls**. Selections on these pages apply immediately — there is **no Save button**.

### 8A — Turn Copilot on for organizations

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

> ⚠️ **Do this first:** until Copilot is enabled for an organization here, its owners can't assign seats in 8C (Route 1).

### 8B — Set Copilot policies

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Enterprise** → **AI controls** → **Copilot** *(sidebar)*

1. At the top of the enterprise page, click **AI controls** *(a top-of-page tab — not under **Settings**)*.
2. In the sidebar, open the page that holds the policies you want:
   - **Copilot** — administration, privacy, model, billing, and usage policies, including **Policies for enterprise-assigned users** (required before Route 2 in 8C).
   - **Copilot** → under "Features & clients", click **Configure features & clients** — feature and client policies such as Copilot on GitHub.com, Copilot Chat in the IDE, and Copilot in the CLI.
   - **Agents** — AI agent policies, such as **Copilot cloud agent** (formerly Copilot coding agent).
   - **MCP** — Model Context Protocol (MCP) policies.
3. Set each policy:
   - **Dropdown:** open it and choose an enforcement option — **Enabled**, **Disabled**, or **No policy** (lets each organization owner decide).
   - **Toggle:** click it.
   - **No visible control:** click the policy name to see its options.
4. Check that each policy shows the value you chose. *(Changes apply on selection — there is no Save button.)*

> 💡 **Suggestions matching public code:** agree on this setting with your legal team before you enable it.

### 8C — Assign Copilot seats

Use either route — or both. A person assigned through more than one route still uses **one** license (the highest tier).

**Route 1 — Organization seats** (organization owner)

**👤 Role:** GitHub **organization owner** · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Organizations** → *[organization]* → **Settings** → **Copilot** *(sidebar, under "Code, planning, and automation")* → **Access**

1. If you see **Allow this organization to assign seats**, click it.
2. Click **Start adding seats**.
3. Choose who gets Copilot:
   - **Everyone:** select **Purchase for all members**, then in the "Confirm seats purchase for all members" dialog click **Purchase seats**.
   - **Specific people or teams:** select **Purchase for selected members**. In the "Enable Copilot access for users and teams" dialog, use the **Users and teams** tab to search for and add people or teams (or **Upload CSV** to add many at once), then click **Continue to purchase** → **Purchase seats**.

> 💡 **Hands-off seats:** give seats to a team that's linked to an Entra ID group (Step 6). When Entra ID adds someone to the group, SCIM adds them to the team and they get a Copilot seat automatically.

> 💡 **Billing:** a seat is billed from the moment it's granted (prorated mid-cycle), whether or not the person uses Copilot yet.

**Route 2 — Enterprise licenses** (enterprise owner · Copilot Business · no organization membership required)

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Enterprise** → **Billing and licensing** → **Licensing** → **Manage** *(in the "Copilot" section)*

**Before you start:** set the **Policies for enterprise-assigned users** policy (8B), make sure the people already exist in the enterprise (provisioned by SCIM), and create the enterprise team first if you're licensing a team.

1. Click the **All members** tab (individual users) or the **Enterprise Teams** tab.
2. Click **Assign licenses**.
3. Search for the users or enterprise teams, then click **Add licenses**.

> ✅ **Enterprise teams are generally available** (since June 2026). License an enterprise team and people gain or lose Copilot as they join or leave it. With Enterprise Managed Users you can sync the enterprise team to an Entra ID group, so licensing is driven entirely from your IdP.

> 💡 **When to use this route:** people who need Copilot but no organization access. Enterprise members who aren't in any organization usually don't consume a GitHub Enterprise Cloud license. Direct enterprise assignment is for **Copilot Business**.

### Result: zero-touch provisioning

When this is set up end to end:

1. A user is added to an Entra ID group.
2. SCIM provisions their managed user account.
3. They're added to the GitHub team (Route 1) or enterprise team (Route 2) linked to that group.
4. They get a Copilot seat automatically through that team.
5. They sign in to their IDE and start using Copilot.

---

## 9️⃣ Validation Checklist

Run through these checks to confirm successful setup:

| Check | How to Verify | Expected Result |
|-------|---------------|------------------|
| **OIDC SSO** | Have a provisioned user attempt to sign in | Redirects to Entra ID, successfully authenticates |
| **SCIM Provisioning** | Check Enterprise → People tab | Test users appear with `_shortcode` suffix |
| **Group Sync** | Check Organization → Teams | IdP group members appear in linked team |
| **Azure Billing** | Enterprise → Billing and licensing → Payment information | Azure Subscription ID displayed |
| **Copilot Access** | User opens VS Code with GitHub Copilot extension | Copilot icon active, suggestions working |

### Troubleshooting Quick Reference

| Issue | Common Cause | Solution |
|-------|--------------|----------|
| Users not provisioning | SCIM token expired or invalid | Regenerate PAT with `scim:enterprise` scope |
| SSO redirect fails | Entra app misconfigured | Verify OIDC app settings in Entra admin center |
| "Admin approval required" for Azure | Insufficient Azure AD permissions | Request tenant-wide admin consent |
| Copilot not activating | Copilot isn't turned on for the organization, or the user has no seat | Turn the org on at Enterprise → **Billing and licensing** → **Licensing** → Copilot **Manage**, then assign a seat (Step 8C) |
| Team membership not syncing | Nested groups in Entra | Flatten group structure or add users directly |

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


### Q: What is the difference between OIDC and SAML for EMU, and which should I choose?
**A:** OIDC is recommended for Entra ID because it supports Conditional Access Policies (CAP) natively — GitHub can honor Entra session policies such as IP restrictions and device compliance. SAML does not pass CAP signals to GitHub. If your organization uses Entra Conditional Access, choose OIDC. If your IdP does not support OIDC with GitHub (e.g., Okta, PingFederate), SAML is your only option.

---

### Q: My Conditional Access Policies are not being applied to GitHub sessions — what is wrong?
**A:** Verify that you configured OIDC (not SAML) for SSO. Conditional Access only works with the OIDC flow. Also confirm the Conditional Access Policy in Entra targets the "GitHub Enterprise Managed User (OIDC)" Enterprise Application specifically. Check that the policy conditions (device compliance, location, etc.) are correctly configured and that the user is in scope of the policy assignment.

---

### Q: I get a "tenant ID mismatch" or "token audience" error during OIDC setup — how do I fix it?
**A:** This usually happens when the Global Administrator who consents during the OIDC redirect is signed into a different Entra tenant than the one hosting your Enterprise Application. Ensure the admin uses an account from the correct tenant. Clear browser cookies or use an incognito window to avoid cross-tenant session bleed. The tenant ID in the OIDC configuration must match the tenant where the Enterprise Application was created.

---

### Q: SCIM provisioning is failing with 401 or 403 errors — what should I check?
**A:** A 401 error means the SCIM PAT is invalid, expired, or was not created by the setup user. Verify the token has the `scim:enterprise` scope and was generated by the `SHORTCODE_admin` account. A 403 error can mean the token lacks the required scope or the Tenant URL is incorrect. Regenerate the token if needed and confirm the Tenant URL matches your hosting environment (github.com vs GHE.com).

---

### Q: How do I switch from SAML to OIDC on an existing EMU enterprise?
**A:** Contact GitHub Support to assist with the transition. The general process involves: (1) creating the "GitHub Enterprise Managed User (OIDC)" Enterprise Application in Entra, (2) having GitHub Support disable the current SAML configuration, (3) enabling OIDC via the enterprise settings, (4) consenting as a Global Administrator, and (5) verifying SCIM continues to work with the existing token. Plan this during a maintenance window as users will briefly lose access during the switch.

---

### Q: Users are provisioned but get a "could not authenticate" error when signing in — why?
**A:** The most common cause is that the user was provisioned via SCIM but has not been assigned to the OIDC Enterprise Application in Entra for SSO purposes. SCIM provisioning and SSO authentication use the same Enterprise Application, but the user must be in scope for both. Also verify that the user's Entra account is active and not blocked by a Conditional Access Policy.

---

### Q: The Entra provisioning logs show "quarantined" — how do I recover?
**A:** Quarantine activates when Entra detects repeated provisioning failures (typically over 40% error rate). Check the provisioning logs for specific error details — common causes are an expired SCIM token, incorrect Tenant URL, or GitHub API rate limiting. Fix the root cause, regenerate the SCIM token if needed, update the configuration, and click "Restart provisioning." Entra exits quarantine automatically after a successful cycle.

---

### Q: Can I use the same Entra Enterprise Application for both OIDC SSO and SCIM provisioning?
**A:** Yes. When you enable OIDC, GitHub automatically creates the "GitHub Enterprise Managed User (OIDC)" Enterprise Application in your Entra tenant. You configure SCIM provisioning on this same application. Do not create a separate SAML-based Enterprise Application — use only the OIDC one for both SSO and SCIM.

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

| Resource | URL |
|----------|-----|
| Enterprise Settings | `https://github.com/enterprises/{slug}/settings` |
| Microsoft Entra Admin Center | `https://entra.microsoft.com` |
| GitHub Support Portal | `https://support.github.com` |
| GitHub Status | `https://www.githubstatus.com` |

---

## 📝 Source Documentation

| # | Document | URL |
|---|----------|-----|
| 1 | Connecting an Azure subscription | [docs.github.com](https://docs.github.com/enterprise-cloud@latest/billing/managing-the-plan-for-your-github-account/connecting-an-azure-subscription) |
| 2 | Setup user 2FA requirement (Jan 2025) | [github.blog](https://github.blog/changelog/2025-01-17-setup-user-for-emu-enterprises-requires-2fa-or-use-of-a-recovery-code/) |
| 3 | Configuring SCIM provisioning for EMU | [docs.github.com](https://docs.github.com/enterprise-cloud@latest/admin/managing-iam/provisioning-user-accounts-with-scim/configuring-scim-provisioning-for-users) |
| 4 | SCIM REST API documentation | [docs.github.com](https://docs.github.com/enterprise-cloud@latest/admin/managing-iam/provisioning-user-accounts-with-scim/provisioning-users-and-groups-with-scim-using-the-rest-api) |
| 5 | Configuring OIDC for EMU | [docs.github.com](https://docs.github.com/enterprise-cloud@latest/admin/managing-iam/configuring-authentication-for-enterprise-managed-users/configuring-oidc-for-enterprise-managed-users) |
| 6 | Managing team memberships with IdP groups | [docs.github.com](https://docs.github.com/enterprise-cloud@latest/admin/managing-iam/provisioning-user-accounts-with-scim/managing-team-memberships-with-identity-provider-groups) |
| 7 | Copilot policies for enterprise | [docs.github.com](https://docs.github.com/enterprise-cloud@latest/copilot/managing-copilot/managing-copilot-for-your-enterprise/managing-policies-and-features-for-copilot-in-your-enterprise) |
| 8 | Microsoft Entra OIDC provisioning tutorial | [learn.microsoft.com](https://learn.microsoft.com/en-us/entra/identity/saas-apps/github-enterprise-managed-user-oidc-provisioning-tutorial) |
| 9 | Enterprise Teams generally available (June 2026) | [github.blog](https://github.blog/changelog/2026-06-04-enterprise-teams-is-now-generally-available/) |
| 10 | Downloading enterprise recovery codes | [docs.github.com](https://docs.github.com/enterprise-cloud@latest/admin/managing-iam/managing-recovery-codes-for-your-enterprise/downloading-your-enterprise-accounts-single-sign-on-recovery-codes) |

---

## 📝 Revision History

| Date | Version | Changes |
|------|---------|----------|
| December 2025 | 2.0 | Verified against current documentation; updated OIDC navigation path; clarified Azure permissions; added Enterprise Teams option for Copilot; fixed source references |
| July 2026 | 2.1 | Verified all click paths, roles, and SSO/SCIM/billing/Copilot steps against current GitHub, Microsoft Entra, Okta, and Ping docs; standardized formatting. |
| October 2026 | 2.2 | Re-verified against GitHub's docs source: enterprise teams are GA (June 2026); Copilot org access moved to Billing and licensing → Licensing; removed nonexistent Save clicks on Copilot pages; corrected seat-assignment flows; Copilot coding agent renamed Copilot cloud agent; added the enterprise recovery-codes step; current menu labels. |

---

*Last updated: October 2026*
