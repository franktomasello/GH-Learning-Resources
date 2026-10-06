# 🧪 GitHub Enterprise Trial (GHEC, EMU & DRUS) Setup Runbook

> **Complete guide to starting a GitHub Enterprise Cloud trial — Standard GHEC, EMU (managed users), or Data Residency US (DRUS)**

| 🧭 **At a Glance** | |
|---|---|
| **Goal** | Start and run a GitHub Enterprise Cloud trial |
| **Use this when** | A customer wants to evaluate GitHub Enterprise |
| **People you need** | Trial requester (becomes enterprise owner); IdP admin for EMU |
| **Where you click** | GitHub and your IdP |
| **End result** | A working 30-day trial with SSO and security features ready to test |
| **New to a term?** | See the [Glossary](../Glossary.md) for plain-English definitions |

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [📋 Overview](#-overview)
- [1️⃣ Start the Trial](#1️⃣-start-the-trial)
- [2️⃣ Complete the Setup Email](#2️⃣-complete-the-setup-email)
- [3️⃣ Configure Identity Provider (EMU & DRUS Only)](#3️⃣-configure-identity-provider-emu--drus-only)
- [4️⃣ Configure SAML SSO (Standard GHEC Only)](#4️⃣-configure-saml-sso-standard-ghec-only)
- [5️⃣ Request Add-On Trials](#5️⃣-request-add-on-trials)
- [6️⃣ Trial Duration & Extensions](#6️⃣-trial-duration--extensions)
- [🚀 Tips for a Successful Trial](#-tips-for-a-successful-trial)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Start trial:** Browse to `github.com/account/enterprises/new` → Sign in → Choose **personal accounts** or **managed users** (managed users: also choose **GitHub.com** or **GHE.com**) → enter enterprise name and URL slug → follow the on-screen prompts
- **Activate (managed users only):** Open the setup-user email in a private window → set the password → turn on 2FA → save recovery codes
- **IdP (EMU/DRUS only):** Profile picture → **Enterprise** → **Identity provider** → **Single sign-on configuration** → configure SAML/OIDC + SCIM before inviting users
- **Standard GHEC SSO (optional):** Enterprise or Organization → **Settings** → **Authentication security** → **SAML single sign-on** (Standard GHEC only — not EMU/DRUS)
- **Included:** most GHEC features, plus **Secret Protection** and **Code Security** on GitHub.com trials (not GHE.com); up to 3,000 Actions minutes
- **Not included:** Copilot Business / Enterprise — ask your GitHub account team about a Copilot pilot
- **Copilot seats:** Enterprise → **Billing and licensing** → **Licensing** → Copilot **Manage** (turn on orgs / **Assign licenses**); policies under **AI controls** → **Copilot**
- **Length:** 30 days; unconverted trial enterprises are deleted 90 days after the trial ends. Ask your account team early if you need more time

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

## ✅ Prerequisites

| Requirement | Who / Role needed | ✓ |
|-------------|-------------------|---|
| A GitHub.com personal account to start the trial (no payment method needed) | Trial requester — becomes the first **enterprise owner** (personal-accounts trials) | ☐ |
| Enterprise name and URL slug decided (globally unique — changing it later is limited, see the Q&A) | Trial requester | ☐ |
| Identity model chosen: personal accounts (Standard GHEC), managed users on GitHub.com (EMU), or managed users on GHE.com (DRUS) — **can't be changed later** | Decision makers | ☐ |
| Managed users on GitHub.com: a shortcode chosen (3–8 letters or numbers, **can't be changed later**). GHE.com generates one at random | Trial requester | ☐ |
| Managed-users trials: IdP admin ready to create the GitHub app (Entra ID, Okta, or PingFederate) | IdP administrator | ☐ |
| A GitHub account team contact (for a Copilot pilot or a trial extension) | Trial requester | ☐ |

---

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **GitHub trial requester or enterprise owner** | Starts the trial and decides which identity model will be tested. | `github.com/account/enterprises/new` → sign in → choose personal accounts or managed users (and GitHub.com or GHE.com) → enter enterprise details → follow the on-screen prompts → (managed users) complete the setup-user email. Handoff: enterprise URL, trial type, and setup user invitation. |
| **Microsoft Entra, Okta, or PingFederate admin** | Completes the provider-side app setup for EMU or DRUS trials. | Entra: Microsoft Entra admin center → **Entra ID** → **Enterprise apps** → **New application** → **GitHub Enterprise Managed User** (SAML) or **GitHub Enterprise Managed User (OIDC)** → **Single sign-on** and **Provisioning**. Okta: Okta Admin Console → **Applications** → **Browse App Catalog** → **GitHub Enterprise Managed User** (github.com) or **GitHub Enterprise Managed User - GHE.com** (DRUS) → **Sign On** and **Provisioning**. PingFederate: Administrative Console → **Applications** → **SP Connections** → GitHub EMU SP connection → **Browser SSO** and **Outbound Provisioning**. GitHub-side SSO values are set at Profile picture → **Enterprise** → **Identity provider** → **Single sign-on configuration**. Handoff: SSO values, SCIM status, and pilot group. |
| **Azure subscription Owner and Microsoft Entra consent approver, if testing Azure billing** | Provides the subscription and consent needed for metered billing. | Azure portal → Subscriptions → [subscription] → Access control (IAM) → confirm Owner. Enterprise path: GitHub → Enterprises page (github.com/settings/enterprises) → [enterprise] → Billing and licensing → Payment information → Metered billing via Azure → Add Azure Subscription. Organization path: GitHub → profile picture → Organizations → [organization] → Settings → Billing and licensing → Payment information → Metered billing via Azure → Add Azure Subscription. Then Microsoft sign-in → Permissions requested → Accept → Select subscription → Connect. Handoff: connected subscription ID. |

---

## 📋 Overview

This runbook covers every step to initiate and configure a GitHub Enterprise Cloud trial:

| Trial Type | Description | Best For |
|------------|-------------|----------|
| **Standard GHEC** | Classic enterprise with personal GitHub accounts | Orgs that want SSO but let developers keep personal accounts |
| **EMU (Enterprise Managed Users)** | Enterprise controls all user accounts via IdP | Strict identity governance, regulated industries |
| **DRUS (Data Residency US)** | EMU with data stored in the US region | Data sovereignty requirements within the US |

---

## 1️⃣ Start the Trial

**👤 Role:** GitHub trial requester (becomes enterprise owner) · **📍 Portal:** GitHub

**Navigate:** Browser → `https://github.com/account/enterprises/new` (the **Set up a trial of GitHub Enterprise Cloud** button in GitHub Docs opens the same page)

**Steps:**

1. Sign in with your GitHub.com personal account (or create one). No payment method is needed.
2. Choose the enterprise type:

| Choose | What you get | This guide calls it |
|--------|-------------|---------------------|
| **Enterprise with personal accounts** | People sign in with their own GitHub.com accounts; SAML SSO is optional | Standard GHEC |
| **Enterprise with managed users** → **GitHub.com** | Your IdP creates and controls every account; includes Secret Protection and Code Security | EMU |
| **Enterprise with managed users** → **GHE.com** | Same as EMU, on your own `SUBDOMAIN.ghe.com` in the region you choose; some features aren't available | DRUS (US region) |

3. Enter the enterprise name, URL slug, and any other details the page asks for (managed users on GitHub.com: also the **shortcode**).
4. Follow the on-screen prompts to finish.

> ✅ **Result:**
> - **Personal accounts:** the enterprise is created and you are its enterprise owner. Go straight to Step 4.
> - **Managed users:** GitHub creates the enterprise and emails you an invitation for the **setup user** (`SHORTCODE_admin`). Continue with Step 2.
>
> 📌 **Slug:** Enterprise slugs are globally unique. You can change one later only in limited cases (see the Q&A), so choose carefully.

---

## 2️⃣ Complete the Setup Email

**👤 Role:** GitHub trial requester · **📍 Portal:** Email + a **private / incognito** browser window

> 📌 **Managed-users (EMU / DRUS) trials only.** Personal-accounts trials skip this step.

**Steps:**

1. Open the email from GitHub inviting you to choose a password for the **setup user** (`SHORTCODE_admin`).
2. Copy the link into a **private / incognito** window, so it doesn't mix with your personal GitHub session.
3. Set the setup user's password and save it in your company password manager.
4. Continue straight to [Step 3 → part 0](#0-activate-and-secure-the-emu-setup-user) to turn on 2FA and save recovery codes **before** you do anything else.

> ✅ **Result:** You can sign in as the setup user — the first enterprise owner and the only account in the enterprise not created by SCIM.
> 🧯 **Email missing or link no longer works?** Check spam, then contact GitHub Support or your account team. Don't start a second trial — it creates a separate enterprise.

---

## 3️⃣ Configure Identity Provider (EMU & DRUS Only)

**👤 Role:** GitHub **setup user** (`SHORTCODE_admin`, an enterprise owner) · **📍 Portal:** GitHub + your IdP (Entra / Okta / Ping)

> ⚠️ **Prerequisite:** For EMU and DRUS trials, you **must** configure SCIM provisioning and SAML/OIDC in your IdP **before** inviting any users.
> 📌 **Constraint:** EMU and DRUS use the **Identity provider** path below — **not** `Settings → Authentication security` (that path is for Standard GHEC only, see Step 4). EMU has **no** "Require SAML authentication" checkbox and no backup username/password sign-in.
> 💡 **Tip:** For the exact IdP-side app-creation and provisioning clicks, follow your IdP's dedicated EMU guide (see [Related Guides](#-related-guides)) alongside this section.

### 0) Activate and Secure the EMU Setup User

**👤 Role:** GitHub **setup user** (`SHORTCODE_admin`) · **📍 Portal:** GitHub

> 📌 **Constraint:** GitHub creates a setup user named after your enterprise shortcode plus `_admin` (e.g., `octocorp_admin`). This account owns the SCIM token and is your break-glass access. Secure it **before** configuring SSO/SCIM below.

**Steps:**

1. In the **private / incognito** window, sign in as the setup user (password set in [Step 2](#2️⃣-complete-the-setup-email)).
2. Confirm the password is saved in your company password manager.
3. Go to profile picture → **Settings** → **Password and authentication** → under **Two-factor authentication** click **Enable two-factor authentication** → choose **Set up using an app** (TOTP recommended) → scan the code and **complete the challenge**.
4. Click **Download** (or copy/print) the personal **2FA recovery codes** and store them in your vault.
5. Download the **enterprise recovery codes**: Profile picture → **Enterprise** → **Identity provider** → **Single sign-on configuration** → under **SAML single sign-on** or **OIDC single sign-on**, click **Save your recovery codes** → **Download** (or **Print** / **Copy**). Store them apart from the personal codes.

> 🔐 **Security-critical:** Every future setup-user sign-in needs a 2FA challenge **or** an enterprise recovery code; losing both locks you out. Password reset for the setup user must go through **GitHub Support**.

### A) Configure SAML Single Sign-On (EMU/DRUS)

**👤 Role:** GitHub **setup user** (`SHORTCODE_admin`) + your IdP administrator · **📍 Portal:** GitHub + your IdP admin console

**Navigate:** Profile picture → **Enterprise** → **Identity provider** → **Single sign-on configuration**

> 📌 **First, capture the three SAML artifacts from your IdP** (Entra example). In the **GitHub Enterprise Managed User** enterprise app → **Single sign-on → SAML**:
> - Under **Basic SAML Configuration → Edit**, enter **Identifier** `https://github.com/enterprises/{SLUG}` (no trailing slash), **Reply URL** `https://github.com/enterprises/{SLUG}/saml/consume`, **Sign on URL** `https://github.com/enterprises/{SLUG}/sso` (GHE.com: swap in the `{SUBDOMAIN}.ghe.com` forms) → **Save**.
> - Under **SAML Certificates**, click **Download** on **Certificate (Base64)** (this is the **Public Certificate**).
> - Under **Set up [app]**, copy the **Login URL** (= **Sign on URL**) and **Microsoft Entra Identifier** (= **Issuer**).
>
> Paste those three values into the GitHub fields in step 2 below.

**Steps:**

1. Under **SAML single sign-on**, click **Add SAML configuration**.
2. Enter your IdP's **Sign on URL**, **Issuer**, and **Public Certificate** (the three artifacts captured above), then choose the **Signature Method** and **Digest Method** (SHA-256 recommended).
3. Click **Test SAML configuration** — this must pass before you can save.
4. Click **Save SAML settings**.
5. Immediately **Download**, **Print**, or **Copy** your enterprise **SSO recovery codes** and store them securely.

> 🔐 **Security-critical:** The SSO recovery codes are your break-glass access if the IdP is unavailable. Store them in your secrets manager before leaving this page.
> 💡 **OIDC (Entra only):** Instead of SAML, you can select **Enable OIDC configuration** under **OIDC single sign-on**, click **Save**, consent as a **Global Administrator** with **Consent on behalf of your organization**, save the recovery codes, then click **Enable OIDC Authentication**. OIDC additionally passes Microsoft Entra **Conditional Access** to GitHub.

### B) Configure SCIM Provisioning

**👤 Role:** your IdP administrator (using the setup user's SCIM token) · **📍 Portal:** your IdP admin console

The **SCIM Tenant URL** you paste into the IdP is:

| Platform | Tenant URL |
|----------|-----------|
| **GitHub.com (EMU)** | `https://api.github.com/scim/v2/enterprises/{ENTERPRISE_SLUG}` |
| **GHE.com (DRUS)** | `https://api.{SUBDOMAIN}.ghe.com/scim/v2/enterprises/{SUBDOMAIN}` |

**Steps:**

1. Sign in as the **setup user**, then create the SCIM token: Profile picture → **Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)** → **Generate new token (classic)** with the **`scim:enterprise`** scope and **No expiration**. Copy it immediately (shown once).
2. In your IdP, open the **GitHub Enterprise Managed User** app → **Provisioning** tab → **+ New configuration** (older tenants: **Get started** → **Provisioning Mode = Automatic**).
3. Paste the **SCIM Tenant URL** above into **Tenant URL** and the token from step 1 into **Secret Token** / **Bearer Token** → click **Test Connection** → confirm it succeeds → click **Create** (older UI: **Save**).
4. Go to **Overview → Properties** (pencil) → enable notification emails + accidental-deletion prevention → **Apply**.
5. Review **Attribute Mapping → Users** (username, email, display name) → **Save**, then **Attribute Mapping → Groups** → **Save**.
6. Set **Scope = Sync only assigned users and groups**.
7. Under **Users and groups → Add user/group**, assign users and give at least one the **Enterprise Owner** app role so a managed admin exists.
8. Test one user with **Provision on demand** and confirm the user appears in GitHub before full rollout.
9. Go to **Overview → Start provisioning** — assigned users are created automatically in GitHub.

> ✅ **Verification:** Always confirm **Test Connection** succeeds and that your **Provision on demand** test user lands in GitHub before starting full provisioning — otherwise failures surface only after you invite users.
> 💡 **Tip:** Do not manually create user accounts in an EMU enterprise. All user lifecycle management flows through SCIM.
> 📌 **Constraint:** Use **one IdP** for both SSO and SCIM. Mixing IdPs (e.g., Okta for SSO and Entra for SCIM) is not supported.

---

## 4️⃣ Configure SAML SSO (Standard GHEC Only)

**👤 Role:** GitHub **enterprise owner** or **organization owner** · **📍 Portal:** GitHub

> 📌 **Constraint:** This step applies to **Standard GHEC only**. EMU and DRUS enterprises use the **Identity provider** path in Step 3, not `Settings → Authentication security`.

*For Standard GHEC, SAML SSO can be configured at the organization level or the enterprise level. Enterprise-level SAML is also supported and, when configured, overrides any org-level SAML settings.*

**Enterprise level (recommended) — Navigate:** **Enterprises** page (github.com/settings/enterprises) → *[enterprise]* → **Settings** (top) → **Authentication security**

1. Under **SAML single sign-on**, select **Require SAML authentication**.
2. Enter your IdP's **Sign on URL**, **Issuer** (optional), and **Public Certificate**.
3. (Optional) Click the pencil next to the signature and digest methods and choose the ones your IdP uses (SHA-256 is typical).
4. Click **Test SAML configuration** — it must pass before you can save.
5. Click **Save**.
6. Click **Download** (or **Print** / **Copy**) to save the enterprise **recovery codes** in your password manager.

**Organization level — Navigate:** Profile picture → **Organizations** → *[organization]* → **Settings** → **Authentication security** *(Security section)*

1. Under **SAML single sign-on**, select **Enable SAML authentication**.
2. Enter the **Sign on URL**, **Issuer** (optional), and **Public Certificate**, and adjust the signature and digest methods if needed.
3. Click **Test SAML configuration** — it must pass before you can save.
4. (Optional — do this after members have linked their identities) Select **Require SAML SSO authentication for all members of the *organization name* organization**. Members who haven't authenticated through your IdP are removed.
5. Click **Save**, then save the organization **recovery codes** when prompted.

> 🔐 **Security-critical:** Recovery codes are your break-glass access if the IdP is unavailable. Store them before leaving the page.
> ✅ **Result:** Members must sign in through your IdP to reach enterprise or organization resources. Enterprise-level SAML overrides any organization-level SAML.

---

## 5️⃣ Request Add-On Trials

*Enhance your trial with additional products to evaluate the full platform.*

### A) Copilot Business Trial

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

> 📌 Copilot Business and Copilot Enterprise are **not included** in the self-serve GHEC trial.

**Steps:**

1. Contact your GitHub Sales representative or Solutions Engineer and ask whether a Copilot pilot can be added to your evaluation (terms vary — confirm with your account team).
2. Once Copilot is available on the enterprise, set it up in this order:

**(a) Turn Copilot on for organizations — Navigate:** your enterprise (EMU/DRUS: profile picture → **Enterprise**; standard GHEC: **Enterprises** page → *[enterprise]*) → **Billing and licensing** → **Licensing** → **Manage** *(in the "Copilot" section)*

1. Next to **Organization access**, choose all organizations or **Allow for specific organizations**.
2. For specific organizations, click the **Organizations** tab and set each organization's **Copilot** dropdown to **Enabled**. *(Applies immediately — there is no Save button.)*

**(b) Set policies — Navigate:** your enterprise → **AI controls** → **Copilot** *(sidebar)*

1. Set policies on the **Copilot** page, and under "Features & clients" click **Configure features & clients** for feature and client policies (public-code matching, Copilot Chat, CLI, and so on). Use the **Agents** and **MCP** sidebar pages if you'll pilot agents or MCP servers.
2. For each policy, choose **Enabled**, **Disabled**, or **Let organizations decide**. *(Applies on selection — there is no Save button.)*

> ⏰ **Before October 22, 2026:** set the **Default policy for new features** on the **Copilot** page (**Enabled**, **Disabled**, or **Let organizations decide**). From that date, GA features left **Unconfigured** follow it — and it's **Enabled** by default.

**(c) Give pilot users seats** — use either route:

- **Organization:** org **Settings** → **Copilot** → **Access** → **Start adding seats** → **Purchase for selected members** → add people or teams on the **Users and teams** tab → **Continue to purchase** → **Purchase seats**.
- **Enterprise (Copilot Business):** the same **Manage** page as (a) → **All members** or **Enterprise Teams** tab → **Assign licenses** → search → **Add licenses**. *(Set the **Policies for enterprise-assigned users** policy in (b) first.)*

> 💡 **Tip:** **AI controls** is a top-of-page enterprise tab, not under **Settings**. It holds Copilot **policies**; turning Copilot on for organizations and assigning licenses both happen under **Billing and licensing → Licensing**.

### B) GitHub Advanced Security (GHAS) Trial

**👤 Role:** GitHub **organization owner** · **📍 Portal:** GitHub

> ✅ **Included on GitHub.com trials:** a GHEC trial created on GitHub.com already includes **GitHub Secret Protection** and **GitHub Code Security** — no separate request needed. Trials on **GHE.com** don't include them.

**Navigate:** Profile picture → **Organizations** → *[organization]* → **Settings** → **Advanced Security ▾** → **Configurations**

**Steps:**

1. Click **New configuration**, then use the quick setup dialog (**Review** → **Save and enable**) or choose **Custom configuration** → turn on **Secret Protection** (with push protection) and **Code Security** (default setup) → **Save configuration**.
2. On the **Repositories** tab, select the trial repositories → **Apply configuration ▾** → your configuration → **Apply**.

> 💡 Code scanning default setup uses Actions minutes — the trial includes up to 3,000 standard runner minutes.

> 💡 **Tip:** Request add-on trials early in your evaluation period so you have maximum time to test.

---

## 6️⃣ Trial Duration & Extensions

| Detail | Value |
|--------|-------|
| **Default trial duration** | 30 days |
| **Actions minutes during the trial** | Up to 3,000 standard GitHub-hosted runner minutes (the 50,000 paid-plan minutes don't apply). EMU trials need a linked Azure subscription to go beyond this |
| **Extensions** | Not self-service — ask your GitHub account team as early as possible |
| **Cancel** | Enterprise → **Settings** → **Danger zone** |
| **After expiry** | Unconverted trial enterprises are deleted **90 days** after the trial ends |

> ⚠️ **Important:** if you invite an **existing** organization into the trial enterprise, the trial features are disabled for it. Create new organizations for the evaluation.

---

## 🚀 Tips for a Successful Trial

1. **Define success criteria** before starting — what does "yes, we buy" look like?
2. **Schedule a kickoff call** with your GitHub SE/CSM within the first week
3. **Talk to your account team early** if you may need more than 30 days
4. **Invite a small pilot group** first to validate your IdP integration before broad rollout
5. **Document your configuration decisions** — they carry over if you convert to a paid plan

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
| **Trial activation link expired or fails** | The setup email was not used in time or was opened by the wrong account. | Ask the GitHub account team or trial flow owner to resend activation and complete setup with the intended enterprise owner identity. |
| **Trial feature is missing** | The add-on trial was not activated for the selected enterprise/org or the plan type does not support it. | Confirm the exact enterprise/org with GitHub Sales or your Solutions Engineer, then recheck the documented settings page after activation. |
| **Trial ends sooner than expected after billing setup** | On a **personal-accounts** trial, linking an Azure subscription ends the trial immediately and starts paid usage. (Managed-users trials must link Azure to go past 3,000 Actions minutes, and stay in trial.) | Don't link Azure to a personal-accounts trial until you're ready to pay. If it already happened, contact your GitHub account team. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>

### Q: The setup-user email never arrived, or its link no longer works. What now?
**A:** Check spam and any mail filters first. If it's still missing or the link fails, contact GitHub Support or your account team and ask them to resend the setup-user invitation.

Don't start a second trial to "fix" it. A new trial creates a separate enterprise, and the slug you wanted stays taken by the first one.

---

### Q: Which trial type should I choose — Standard GHEC, EMU, or DRUS?
**A:** Pick the model you'd run in production — you can't convert a trial between types later.

- **Standard GHEC (personal accounts):** developers already have GitHub.com accounts and you want a low-friction evaluation with optional SAML SSO.
- **EMU:** you need full identity control — every account created and managed by your IdP.
- **DRUS (GHE.com):** you have a formal data residency requirement as well as EMU.

If production intent is still undecided, Standard GHEC is the simplest to set up and evaluate.

---

### Q: Can I extend the trial, and who do I contact?
**A:** Extensions aren't self-service. Contact your GitHub Solutions Engineer (SE), Customer Success Manager (CSM), or Sales representative well before the 30 days end, and explain what you still need to evaluate. If the trial expires, the enterprise is deleted 90 days later unless you convert it.

---

### Q: The enterprise namespace I want is already taken — what can I do?
**A:** Enterprise slugs are unique across all of GitHub.

- **Taken by another customer:** pick a different slug — for example add a suffix (`contoso-corp` instead of `contoso`).
- **Taken by your own earlier trial:** contact GitHub Support to have it released.

Changing a slug later (Enterprise → **Settings** → **Danger zone** → **Change enterprise URL slug**) is self-service only if you pay by credit card or PayPal. EMU and invoiced enterprises must ask GitHub Sales, and GHE.com enterprises can't change it — so choose carefully.

---

### Q: I can't find Copilot or Advanced Security in my trial — what should I check?
**A:**

- **Secret Protection and Code Security** are already included in GitHub.com trials (not GHE.com). Find them under Organization → **Settings** → **Advanced Security**.
- **Copilot** isn't part of the self-serve trial. Arrange a pilot with your GitHub Sales representative or Solutions Engineer. Once it's active, policies are under Enterprise → **AI controls** → **Copilot**.

If something still doesn't appear, ask your GitHub contact to confirm it was applied to the right enterprise or organization.

---

### Q: What happens to my data when the trial expires?
**A:** It depends on how the trial ends:

| How it ends | What happens |
|-------------|--------------|
| **Expires** after 30 days | Organizations you transferred in go back to their previous plans. Owners and members keep access to the enterprise and the organizations created during the trial in a **downgraded state**, so you can buy GitHub Enterprise or move your work elsewhere |
| **You cancel it** (Enterprise → **Settings** → **Danger zone**) | Transferred organizations go back to their previous plans. Everyone loses access to the enterprise and to organizations created during the trial |
| **Not converted** | The trial enterprise is **deleted 90 days** after the trial ends |

To keep everything, purchase GitHub Enterprise before the 90 days are up — ideally before the trial expires.

---

### Q: Can I convert a trial directly to a paid enterprise without starting over?
**A:** Yes. Purchase GitHub Enterprise for the trial enterprise (for invoicing, go through your GitHub Sales representative). The enterprise, its configuration, repositories, and users carry over — nothing is rebuilt. That's why it pays to configure the trial as if it were production.

One exception: organizations you **transferred into** the trial are removed if the trial expires or is canceled before you buy.

---

### Q: I set up an EMU trial but realized I need Standard GHEC instead — can I switch?
**A:** No. The identity model (Standard vs EMU vs DRUS) is set at enterprise creation and cannot be changed. You must create a new trial with the correct type. If you need to preserve any repository data from the EMU trial, use GitHub Enterprise Importer (GEI) to migrate repos to the new enterprise. Plan the identity model decision carefully before starting the trial.

</details>

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Enterprise Environment Scaffolding Checklist | `Setup/Enterprise Environment Scaffolding Checklist.md` |
| Data Residency Decision Guide | `Setup/Data Residency Decision Guide (DRUS vs Standard vs GHES).md` |
| EMU Benefits and Advantages | `Identity/EMU Benefits and Advantages.md` |
| EMU + Entra ID (SAML) + Azure Billing + Copilot | `Setup/EMU + Entra ID (SAML) + Azure Billing + Copilot.md` |
| EMU + Entra ID (OIDC) + Azure Billing + Copilot | `Setup/EMU + Entra ID (OIDC) + Azure Billing + Copilot.md` |
| EMU + Okta (SAML) + Azure Billing + Copilot | `Setup/EMU + Okta (SAML) + Azure Billing + Copilot.md` |
| EMU + PingFederate (SAML) + Azure Billing + Copilot | `Setup/EMU + PingFederate (SAML) + Azure Billing + Copilot.md` |

---

## 📝 Resources

| Resource | Link |
|----------|------|
| **Start a trial** | [github.com/account/enterprises/new](https://github.com/account/enterprises/new) |
| **Setting up a trial of GitHub Enterprise Cloud** | [docs.github.com](https://docs.github.com/en/enterprise-cloud@latest/admin/overview/setting-up-a-trial-of-github-enterprise-cloud) |
| **Choosing an enterprise type** | [docs.github.com](https://docs.github.com/en/enterprise-cloud@latest/admin/concepts/enterprise-fundamentals/choose-an-enterprise-type) |
| **Getting started with Enterprise Managed Users** | [docs.github.com](https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/understanding-iam-for-enterprises/getting-started-with-enterprise-managed-users) |

---

*Last updated: October 2026*
