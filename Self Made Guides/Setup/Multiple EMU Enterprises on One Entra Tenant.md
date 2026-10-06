# 🔗 Multiple GitHub EMU Enterprises on One Entra Tenant Runbook

> How to connect multiple GitHub Enterprise Managed User instances to a single Microsoft Entra ID tenant

| 🧭 **At a Glance** | |
|---|---|
| **Goal** | Connect more than one EMU enterprise to a single Entra ID tenant |
| **Use this when** | The customer needs separate enterprises (for example regulated vs non-regulated) |
| **People you need** | Entra Application Administrator; setup user for each enterprise |
| **Where you click** | Microsoft Entra admin center and GitHub |
| **End result** | Each enterprise with its own SAML app, SCIM, and governance |
| **New to a term?** | See the [Glossary](../Glossary.md) for plain-English definitions |

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [📋 Overview](#-overview)
- [1️⃣ Create Separate Enterprise Applications](#1️⃣-create-separate-enterprise-applications)
- [2️⃣ Configure SAML/OIDC Per Application](#2️⃣-configure-samloidc-per-application)
- [3️⃣ Configure SCIM Provisioning Separately](#3️⃣-configure-scim-provisioning-separately)
- [4️⃣ Scope User Populations Using Entra Groups](#4️⃣-scope-user-populations-using-entra-groups)
- [5️⃣ Key Considerations](#5️⃣-key-considerations)
- [⚠️ Important Notes](#️-important-notes)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Entra ID:** Create one Enterprise Application per EMU enterprise from gallery → Name distinctly (e.g., "GitHub EMU - Enterprise A")
- **SAML/OIDC:** Configure each app with its own Entity ID, Reply URL, Sign-on URL pointing to the respective enterprise
- **SCIM:** Each app gets its own Provisioning config with separate Tenant URL + Secret Token from each enterprise's setup user
- **User scoping:** Create separate Entra groups per enterprise → Assign each group to only its corresponding Enterprise Application

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
| Two or more EMU enterprises provisioned by GitHub | GitHub **enterprise owner** (setup user for each enterprise) | ☐ |
| Setup user credentials for each enterprise (`SHORTCODE_admin`) | GitHub **enterprise owner** / setup user per enterprise | ☐ |
| Permission to create and configure enterprise applications in Entra | Entra **Application Administrator, Cloud Application Administrator, or Application Owner** | ☐ |
| Global Administrator available (OIDC integrations only) | Entra **Global Administrator** for tenant admin consent | ☐ |
| Separate Entra security groups created for each enterprise's user population | Entra **Application Administrator** (or group owner) | ☐ |
| Enterprise shortcodes decided (short, descriptive, distinguishable) | GitHub **enterprise owner** | ☐ |

---

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **GitHub setup user for each EMU enterprise** | Configures each enterprise separately and captures a separate SCIM token per enterprise. | For each enterprise: GitHub → profile picture → Enterprise → Identity provider → Single sign-on configuration → choose SAML unless this is the single permitted Entra OIDC integration → configure and test SSO. Then setup user → Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token → `scim:enterprise`. Handoff: enterprise slug, SAML values or OIDC status, Tenant URL, and SCIM token. |
| **Microsoft Entra Application Administrator, Cloud Application Administrator, or Application Owner** | Creates a separate enterprise application and provisioning configuration per EMU enterprise. | Microsoft Entra admin center → Entra ID → Enterprise apps → New application → GitHub Enterprise Managed User → Create → rename app with the enterprise slug → Single sign-on → SAML → configure enterprise-specific values → Provisioning → + New configuration (older tenants: Get started, Provisioning Mode = Automatic) → Tenant URL and Secret Token for that same enterprise → Test Connection → Create → Start provisioning → Users and groups → assign enterprise-specific groups. Handoff: one app per enterprise and no mixed SCIM tokens. |
| **Microsoft Entra Global Administrator** | Approves OIDC only where the tenant is intentionally using the single supported Entra OIDC EMU integration. | Microsoft Entra admin center → Entra ID → Enterprise apps → Activity → Admin consent requests → My Pending → approve the selected GitHub Enterprise Managed User (OIDC) request, or complete the consent prompt during GitHub OIDC setup. Handoff: documented decision identifying which single EMU enterprise uses OIDC and which enterprises use SAML. |

---

## 📋 Overview

Multiple EMU enterprises **can** share one Entra ID tenant. The rule is simple: **one enterprise application per enterprise**, each with its own sign-in settings, its own SCIM token, and its own assigned groups.

```text
Entra ID tenant
 ├─ App "GitHub EMU - Enterprise A" ── SSO + SCIM ──▶ Enterprise A  (users: jsmith_codea)
 └─ App "GitHub EMU - Enterprise B" ── SSO + SCIM ──▶ Enterprise B  (users: jsmith_codeb)
```

Work through Steps 1–4 once **per enterprise**. Don't move to the next enterprise until the current one passes its SSO test and a **Provision on demand** test.

---

## 1️⃣ Create Separate Enterprise Applications

**👤 Role:** Entra **Application Administrator, Cloud Application Administrator, or Application Owner** · **📍 Portal:** Microsoft Entra admin center

**Navigate:** **Microsoft Entra admin center** → **Entra ID** → **Enterprise apps** → **New application**

For each EMU enterprise, create a dedicated Entra enterprise application:

1. Click **New application**. The **Browse Microsoft Entra Gallery** page opens.
2. **For a SAML enterprise:** in the search box, type **GitHub Enterprise Managed User** and select it. (Entra uses this same app for GitHub.com and GHE.com enterprises.)
3. In the pane that opens, change **Name** to something that identifies the enterprise, for example **GitHub EMU - Enterprise A**, then click **Create**. Wait until the app's **Overview** page opens.
4. **For the one OIDC enterprise (optional):** don't add anything from the gallery. The **GitHub Enterprise Managed User (OIDC)** app appears in your tenant automatically when you **Enable OIDC configuration** in GitHub and a **Global Administrator** consents (see Step 2). Afterward, open it under **Enterprise apps** to set up provisioning.
5. Repeat steps 1–3 for each other enterprise (**GitHub EMU - Enterprise B**, and so on).

> 📌 **Constraint:** Every EMU enterprise needs its **own** enterprise application.
> - **SAML:** you can add as many GitHub EMU SAML apps to one tenant as you need.
> - **OIDC:** each Entra tenant supports **only one** OIDC integration with EMU. Every other enterprise on that tenant must use SAML.
> - Mixing one OIDC enterprise with SAML enterprises on the same tenant is supported.

---

## 2️⃣ Configure SAML/OIDC Per Application

**👤 Role:** GitHub setup user (`SHORTCODE_admin`, an enterprise owner) + Entra Application Administrator · **📍 Portal:** GitHub + Microsoft Entra admin center

**Navigate (GitHub side):** Profile picture → **Enterprise** → **Identity provider** → **Single sign-on configuration**

Each enterprise application has its own SAML or OIDC configuration pointing to its respective EMU enterprise. For SAML, configure the Entra side FIRST to produce the values GitHub needs, then complete the GitHub side signed in as that enterprise's setup user:

1. On the Entra side, open the matching enterprise application → **Single sign-on** → **SAML**.
2. In the **Basic SAML Configuration** tile, click **Edit** (the pencil icon).
3. Under **Identifier (Entity ID)**, delete the pre-filled default value, then click **Add identifier** and enter the enterprise's Entity ID from the table below (ensure NO trailing slash).
4. Under **Reply URL (Assertion Consumer Service URL)**, enter the Reply URL from the table below.
5. Under **Sign on URL**, enter the Sign-on URL from the table below.
6. Click **Save**, then click **X** to close the panel.
7. In the **Attributes & Claims** section, click **Edit** and confirm the **Unique User Identifier (Name ID)** claim is set to a stable value (for example `user.userprincipalname` or `user.mail`). The gallery app pre-sets the required claims — confirm them, then close the panel.
8. In the **SAML Certificates** section, under **Certificate (Base64)**, click **Download** and save the certificate file — its contents are the **Public Certificate** you paste into GitHub.
9. In the **Set up [app name]** section, click the copy icon next to **Login URL** (this is GitHub's **Sign on URL**) and next to **Microsoft Entra Identifier** (this is GitHub's **Issuer**).
10. **GitHub side (SAML):** Under **SAML single sign-on**, click **Add SAML configuration**, enter **Sign on URL**, **Issuer**, and **Public Certificate** (the three values you collected above), choose **Signature Method** and **Digest Method** (SHA-256 recommended), click **Test SAML configuration**, then **Save SAML settings**, and immediately **Download / Print / Copy** the enterprise **SSO recovery codes**.
11. **GitHub side (OIDC)** (for the single permitted Entra OIDC integration): under **OIDC single sign-on**, select **Enable OIDC configuration**, click **Save**, complete **Global Administrator** consent in Microsoft Entra (**Consent on behalf of your organization** → **Accept**), save the recovery codes, then click **Enable OIDC Authentication**.

> ⚠️ **Gotcha:** EMU has NO "Require SAML authentication" checkbox and no backup username/password sign-in — that is the standard-GHEC flow. The Entra gallery app also pre-fills a different Identifier format; overwrite it with the exact Entity ID below and ensure there is NO trailing slash.

### For github.com EMU

| Field | Enterprise A | Enterprise B |
|-------|-------------|-------------|
| Entity ID | `https://github.com/enterprises/ENTERPRISE_A` | `https://github.com/enterprises/ENTERPRISE_B` |
| Reply URL | `https://github.com/enterprises/ENTERPRISE_A/saml/consume` | `https://github.com/enterprises/ENTERPRISE_B/saml/consume` |
| Sign-on URL | `https://github.com/enterprises/ENTERPRISE_A/sso` | `https://github.com/enterprises/ENTERPRISE_B/sso` |

### For GHE.com EMU (Data Residency)

| Field | Enterprise A | Enterprise B |
|-------|-------------|-------------|
| Entity ID | `https://SUBDOMAIN_A.ghe.com/enterprises/SUBDOMAIN_A` | `https://SUBDOMAIN_B.ghe.com/enterprises/SUBDOMAIN_B` |
| Reply URL (ACS) | `https://SUBDOMAIN_A.ghe.com/enterprises/SUBDOMAIN_A/saml/consume` | `https://SUBDOMAIN_B.ghe.com/enterprises/SUBDOMAIN_B/saml/consume` |
| Sign-on URL | `https://SUBDOMAIN_A.ghe.com/enterprises/SUBDOMAIN_A/sso` | `https://SUBDOMAIN_B.ghe.com/enterprises/SUBDOMAIN_B/sso` |

---

## 3️⃣ Configure SCIM Provisioning Separately

**👤 Role:** Entra **Application Administrator, Cloud Application Administrator, or Application Owner** · **📍 Portal:** Microsoft Entra admin center

**Navigate:** **Microsoft Entra admin center** → **Entra ID** → **Enterprise apps** → *[GitHub EMU app for this enterprise]* → **Provisioning**

Each enterprise application needs its own SCIM configuration, using a token generated by that enterprise's setup user:

1. Open the enterprise application for this enterprise, then open the **Provisioning** tab.
2. Click **+ New configuration** (older tenants show **Get started** and a **Provisioning Mode = Automatic** dropdown).
3. Enter the **Tenant URL** for this enterprise: `https://api.github.com/scim/v2/enterprises/{ENTERPRISE_SLUG}` (github.com) or `https://api.{SUBDOMAIN}.ghe.com/scim/v2/enterprises/{SUBDOMAIN}` (GHE.com).
4. Enter the **Secret Token** — the personal access token (classic) with **`scim:enterprise`** scope created by THIS enterprise's setup user.
5. Click **Test Connection**, then **Create** (older UI: **Save**).
6. On the app's **Overview** page, open **Properties** (click the **Edit** pencil), enable **Send an email notification when a failure occurs** and enable **Prevent accidental deletion**, then click **Apply**.
7. Under **Mappings**, open **Provision Microsoft Entra ID Users**, review the attribute mappings, and click **Save** if you change anything. Then open **Provision Microsoft Entra ID Groups**, review, and click **Save** if changed.
8. Before starting full provisioning, click **Provision on demand**, search for one assigned test user, click **Provision**, and confirm the result shows success and the user is created in the correct enterprise. Fix any mapping errors before continuing.
9. Click **Start provisioning** from the Overview page.
10. Repeat with each other enterprise's Tenant URL and Secret Token in its own dedicated app.

> 🔐 **Security-critical:** Never reuse one SCIM token across enterprise applications. Each token works for one enterprise only, and must be created by **that** enterprise's setup user:
> 1. Profile picture → **Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)** → **Generate new token (classic)**.
> 2. **Note:** a descriptive name, for example `EMU-<slug>-SCIM`.
> 3. **Expiration:** **No expiration**.
> 4. **Scopes:** check **only** **`scim:enterprise`**.
> 5. Click **Generate token**, then copy it right away — it's shown only once.

---

## 4️⃣ Scope User Populations Using Entra Groups

**👤 Role:** Entra **Application Administrator, Cloud Application Administrator, or Application Owner** · **📍 Portal:** Microsoft Entra admin center

**Navigate:** **Microsoft Entra admin center** → **Entra ID** → **Enterprise apps** → *[GitHub EMU app]* → **Users and groups**

1. Open the app → **Provisioning** → expand **Settings** → open the **Scope** dropdown → select **Sync only assigned users and groups** → click **Save**.
2. Open the **Users and groups** tab → **Add user/group**.
3. Select the Entra group scoped to THIS enterprise, then assign it.
4. Under **Users and groups**, click **Add user/group** again to create the enterprise owner assignment.
5. Under **Users**, click **None Selected**, pick the user who will be the admin, and click **Select**.
6. Under **Select a role**, click **None Selected**, choose **Enterprise Owner**, and click **Select**.
7. Click **Assign**. (This must be a user, not a group, for the Enterprise Owner role — so a managed enterprise owner exists in that enterprise.)

> 📌 **Constraint:** Entra provisions only direct group members (nested groups are not provisioned). Role-based assignment of the **Enterprise Owner** role requires **Scope = Sync only assigned users and groups**.

| Entra Group | Assigned To | Result |
|-------------|------------|--------|
| `GitHub-Enterprise-A-Users` | Enterprise A App | Users provisioned as `user_SHORTCODE_A` |
| `GitHub-Enterprise-B-Users` | Enterprise B App | Users provisioned as `user_SHORTCODE_B` |

> 💡 **Tip:** A single Entra user CAN be assigned to multiple Enterprise Applications, creating separate managed identities in each EMU enterprise.

---

## 5️⃣ Key Considerations

| Topic | Detail |
|-------|--------|
| **Username uniqueness** | Each enterprise has a different shortcode suffix (e.g., `jsmith_contoso` vs `jsmith_fabrikam`) |
| **Email domains** | Separate email domains are NOT required — same domain works across enterprises |
| **SCIM attribute mappings** | Must produce unique usernames per enterprise |
| **Testing** | Test authentication and provisioning for each enterprise independently |
| **Audit** | Each enterprise has its own audit log |
| **Billing** | Each enterprise has separate billing |

---

## ⚠️ Important Notes

- Each EMU enterprise has fully separate governance, billing, and audit
- There is NO cross-enterprise visibility without additional tooling
- Managed users **can't** be invited into another enterprise — a person who needs access to both enterprises needs an account provisioned in each (guest collaborators only limit access *within* one enterprise)
- SCIM provisioning cycles are independent — changes to one don't affect the other

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

### Q: SCIM is provisioning users to the wrong enterprise — how did this happen?
**A:** Check which enterprise application the user is assigned to in Entra — a user assigned to "GitHub EMU - Enterprise A" is provisioned to Enterprise A. Then, in each app's **Provisioning** settings, confirm the **Tenant URL** and **Secret Token** belong to the matching enterprise.

Each enterprise needs its own application and its own SCIM token. Never reuse a token across applications.

---

### Q: A user is assigned to both Enterprise Applications — will they get two managed accounts?
**A:** Yes — one managed account in each enterprise (for example `jsmith_codea` and `jsmith_codeb`). That's fine if the person really needs both, but each account uses a license in its enterprise.

To avoid accidents, use one Entra group per enterprise, and only put someone in both groups when they need cross-enterprise access.

---

### Q: The enterprise shortcodes are confusing our users — how do we choose good shortcodes?
**A:** The shortcode is added to every managed username (for example `jsmith_contoso`). It must be 3–8 letters or numbers, and it **can't be changed** after the enterprise is created.

Pick codes that are short, meaningful, and clearly different. For example, for a North America enterprise and an EMEA enterprise, use `ctsona` and `ctsoemea` (giving `jsmith_ctsona` and `jsmith_ctsoemea`). Avoid codes that look alike.

On GHE.com, the shortcode is generated at random, so you can't choose it.

---

### Q: We need audit logs across both enterprises — is there a way to get cross-enterprise visibility?
**A:** No. Each EMU enterprise has its own separate audit log with no native cross-enterprise aggregation. To get a unified view, stream audit logs from both enterprises to a common SIEM (Splunk, Sentinel, etc.) using the audit log streaming feature (Enterprise > Settings > Audit log > Log streaming). Configure each enterprise to stream to the same destination. This is the only way to achieve cross-enterprise audit visibility.

---

### Q: Can we use the same Entra groups for both Enterprise Applications, or do they need to be separate?
**A:** You should use separate Entra groups for each Enterprise Application to maintain clear boundaries. While you technically could assign the same group to both apps, this would provision every user in that group to both enterprises (creating dual accounts and consuming licenses in both). Use group naming that clearly indicates the target enterprise, e.g., `GitHub-Enterprise-A-Users` and `GitHub-Enterprise-B-Users`.

---

### Q: One enterprise uses SAML and the other uses OIDC — is that supported on the same Entra tenant?
**A:** Yes. Each enterprise application is independent, so one enterprise can use OIDC and the others SAML.

- **SAML app:** "GitHub Enterprise Managed User"
- **OIDC app:** "GitHub Enterprise Managed User (OIDC)"

Only **one** enterprise per Entra tenant can use OIDC. Configure each enterprise with its own setup guide (see [Related Guides](#-related-guides)).

---

### Q: We are hitting Entra provisioning rate limits — how do we manage provisioning across two enterprises?
**A:** Entra runs provisioning cycles separately for each enterprise application (roughly every 40 minutes per app).

- **Initial load:** stagger it. Start Enterprise A, wait for its first cycle to finish, then start Enterprise B.
- **Ongoing:** don't assign more than **1,000 users per hour** to each app, or add more than 1,000 users to a group per hour — GitHub's SCIM rate limit.
- **Monitoring:** review each app's provisioning logs separately.

</details>

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| EMU + Entra ID (SAML) Setup Guide | `Setup/EMU + Entra ID (SAML) + Azure Billing + Copilot.md` |
| EMU + Entra ID (OIDC) Setup Guide | `Setup/EMU + Entra ID (OIDC) + Azure Billing + Copilot.md` |
| Guest Collaborators in EMU | `Identity/Guest Collaborators in EMU.md` |

---

## 📝 Resources

| Resource | Link |
|----------|------|
| Configuring SCIM for EMU | [docs.github.com](https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/provisioning-user-accounts-with-scim/configuring-scim-provisioning-for-users) |
| About EMU | [docs.github.com](https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/understanding-iam-for-enterprises/about-enterprise-managed-users) |
| Configuring SAML for EMU | [docs.github.com](https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/configuring-authentication-for-enterprise-managed-users/configuring-saml-single-sign-on-for-enterprise-managed-users) |

---

*Last updated: October 2026*
