# 🏗️ GitHub Enterprise Environment Scaffolding Checklist

> **Comprehensive checklist for scaffolding a new GitHub Enterprise Cloud environment from scratch — identity, governance, security, billing, and Copilot**

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [📋 Overview](#-overview)
- [1️⃣ Choose an Identity Model](#1️⃣-choose-an-identity-model)
- [2️⃣ Design Enterprise & Organization Structure](#2️⃣-design-enterprise--organization-structure)
- [3️⃣ Configure Identity & Provisioning](#3️⃣-configure-identity--provisioning)
- [4️⃣ Apply Baseline Governance](#4️⃣-apply-baseline-governance)
- [5️⃣ Set Up Security Controls](#5️⃣-set-up-security-controls)
- [6️⃣ Configure Billing](#6️⃣-configure-billing)
- [7️⃣ Enable Copilot](#7️⃣-enable-copilot)
- [8️⃣ Validation Checklist](#8️⃣-validation-checklist)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Identity (EMU / DRUS):** Profile picture → **Enterprise** → **Identity provider** → **Single sign-on configuration** → **Add SAML configuration** (or **Enable OIDC configuration** for Entra); create the SCIM token as a **personal access token (classic)** scoped to `scim:enterprise`
- **Identity (Standard):** Enterprise → **Settings** → **Authentication security** → **SAML single sign-on**
- **Governance:** Enterprise → **Policies** → **Repository** policies (visibility, creation), **Code** (rulesets), **Actions** policies, **Personal access tokens** policies, **GitHub Apps** policies
- **Security:** Org → **Settings** → **Advanced Security ▾** → **Configurations** → **New configuration** (quick setup or **Custom configuration**: Secret Protection + push protection + Code Security default setup) → apply on the **Repositories** tab
- **Billing:** Enterprise → **Billing and licensing** → **Payment information** → **Metered billing via Azure** → **Add Azure Subscription** → create cost centers → set budgets with alerts
- **Copilot:** Enterprise → **Billing and licensing** → **Licensing** → Copilot **Manage** (turn on orgs, assign licenses) → **AI controls** → **Copilot** (policies, **Configure models**, **Content exclusion**) → Org **Settings** → **Copilot** → **Custom instructions**

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
| GitHub Enterprise Cloud account created (Standard, EMU, or DRUS) | GitHub **enterprise owner** | ☐ |
| Identity model decided (Standard vs EMU vs EMU with Data Residency) | Enterprise architect + GitHub **enterprise owner** | ☐ |
| IdP admin access (Entra ID, Okta, or PingFederate) | Entra **Application Administrator, Cloud Application Administrator, or Application Owner** (or the equivalent Okta/Ping admin) | ☐ |
| Azure subscription for metered billing | Azure **subscription Owner** + tenant-wide admin consent (Entra **Global Administrator** if consent is required) | ☐ |
| Organization structure planned (names, boundaries, team model) | GitHub **enterprise owner** | ☐ |
| Security policy requirements documented (branch rules, secret scanning, code scanning) | Security lead + GitHub **enterprise owner** | ☐ |

---

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **GitHub enterprise owner** | Creates the enterprise/org baseline and opens the provider-specific setup tracks. | GitHub → profile picture → Enterprise → Organizations, Policies, Billing and licensing, Identity provider, and Copilot settings → configure baseline controls in the order shown in this checklist. Handoff: enterprise URL, org list, policy decisions, and assigned owners. |
| **Microsoft Entra, Okta, or PingFederate admin** | Creates the IdP application, group model, SAML/OIDC settings, and provisioning connection required by the chosen identity model. | Entra: Entra ID → Enterprise apps → New application → GitHub Enterprise Managed User or GitHub Enterprise Cloud - Organization → Single sign-on → Provisioning → Users and groups. Okta: Applications → Browse App Catalog → GitHub app → Sign On → Provisioning → Assignments. PingFederate: Applications → SP Connections → GitHub connector/SP connection → Browser SSO → Outbound Provisioning. Handoff: SSO test, SCIM test, group assignments, and owner list. |
| **Azure subscription Owner and Entra consent approver** | Completes Azure billing readiness when metered services will be enabled. | Azure portal → Subscriptions → [subscription] → Access control (IAM) → confirm Owner, then Enterprise path: GitHub → Enterprises page (github.com/settings/enterprises) → [enterprise] → Billing and licensing → Payment information → Metered billing via Azure → Add Azure Subscription. Organization path: GitHub → profile picture → Organizations → [organization] → Settings → Billing and licensing → Payment information → Metered billing via Azure → Add Azure Subscription. If consent is blocked: Microsoft Entra admin center → Entra ID → Enterprise apps → Activity → Admin consent requests → My Pending → Approve. Handoff: subscription ID and consent status. |

---

## 📋 Overview

This runbook walks through every major decision and configuration step when standing up a new GitHub Enterprise Cloud environment. Follow the sections in order — each builds on the previous.

| Phase | What You Configure |
|-------|-------------------|
| **Identity Model** | Standard vs EMU vs EMU with data residency |
| **Enterprise & Org Structure** | Enterprises, organizations, teams |
| **Identity & Provisioning** | SAML SSO, OIDC, SCIM |
| **Baseline Governance** | Repo policies, rulesets, Actions, PATs, Apps |
| **Security Controls** | Secret scanning, code scanning, security configurations |
| **Billing** | Azure subscription, cost centers, budgets |
| **Copilot** | Access, models, content exclusions |
| **Validation** | End-to-end smoke tests |

---

## 1️⃣ Choose an Identity Model

| Feature | Standard Enterprise | EMU | EMU with Data Residency |
|---------|-------------------|-----|------------------------|
| **User accounts** | Users create their own github.com accounts | Accounts provisioned and managed by IdP | Same as EMU |
| **SSO** | SAML SSO — optional; configurable per org, or enforced enterprise-wide (enterprise-level overrides org-level) | SAML or OIDC (required) | SAML or OIDC (required) |
| **SCIM provisioning** | Available at org level with supported IdPs (Entra ID, Okta) — not available at enterprise level | Required — IdP provisions and deprovisions users | Required |
| **Public repos** | Supported | Not supported | Not supported |
| **External collaboration** | Users can contribute to any public repo | Restricted — EMU users cannot interact outside the enterprise | Restricted |
| **Data residency** | No (US-hosted) | No (US-hosted) | Yes — choose region at setup |

> 📌 **OIDC single sign-on for EMU is supported only with Microsoft Entra ID** (it also enables Conditional Access). Okta, PingFederate, and other IdPs use SAML.

> ⚠️ **Important:** The identity model cannot be changed after the enterprise is created. Choose carefully.

> 💡 **Tip:** If your organization requires public repositories or your developers contribute to open-source, Standard Enterprise is the right choice. If you need full lifecycle control over user accounts, choose EMU.

---

## 2️⃣ Design Enterprise & Organization Structure

### Recommended Structure

```
Enterprise (one per company)
├── Org: platform-engineering      (shared platform, IaC, tooling)
├── Org: business-unit-a           (department or product line)
├── Org: business-unit-b           (department or product line)
├── Org: open-source               (public repos — Standard Enterprise only)
└── Org: sandbox                   (experimentation, training)
```

| Decision | Recommendation |
|----------|---------------|
| **Number of enterprises** | One per company (multi-enterprise adds billing and policy complexity) |
| **Org boundaries** | Align to departments, business units, or product lines |
| **Shared platform org** | Create a dedicated org for IaC, reusable Actions, templates, and internal packages |
| **Sandbox org** | Optional — useful for training and experimentation without affecting production |

---

## 3️⃣ Configure Identity & Provisioning

### For Standard Enterprise (SAML SSO)

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **Settings** → **Authentication security** → **SAML single sign-on**

1. Check **Enable SAML authentication**.
2. Enter the IdP **Sign on URL**, **Issuer**, and **Public certificate**; choose the **Signature Method** and **Digest Method** (SHA-256 recommended).
3. Click **Test SAML configuration** and complete the IdP round-trip — the test must pass before you can save.
4. Click **Save**.
5. When ready to enforce, check **Require SAML authentication** and **Save** again, then **Download**, **Print**, or **Copy** the SSO recovery codes and store them securely.

> 💡 **Tip:** Enable SAML at the enterprise level to enforce SSO across all organizations. Org-level SAML is also available but enterprise-level SAML overrides org-level and is recommended for consistency.

> 📌 **Constraint:** This **Authentication security → Require SAML authentication** flow is correct for **Standard Enterprise only**. EMU uses the **Identity provider** path below and has no "Require SAML authentication" checkbox.

### For EMU (SAML/OIDC + SCIM)

**👤 Role:** GitHub **enterprise owner** (the setup user) · **📍 Portal:** GitHub + your IdP

> 🔐 **Sign in as the setup user first.** All EMU identity setup is done while signed in as the setup user (enterprise **shortcode** + `_admin`, e.g. `octocorp_admin`) in a private/incognito window. Enable 2FA immediately and save the personal 2FA recovery codes — every setup-user sign-in requires a 2FA challenge or an enterprise recovery code.

**Navigate (SSO):** Profile picture → **Enterprise** → **Identity provider** → **Single sign-on configuration**

1. Under **SAML single sign-on**, click **Add SAML configuration** (or, for Microsoft Entra, select **Enable OIDC configuration** under **OIDC single sign-on**).
2. For SAML, enter the **Sign on URL**, **Issuer**, and **Public Certificate**, and choose the **Signature Method** and **Digest Method** (SHA-256 recommended).
3. Click **Test SAML configuration** — it must pass before you can save.
4. Click **Save SAML settings** (OIDC: click **Save**, complete the Entra Global Administrator consent, then **Enable OIDC Authentication**).
5. Immediately **Download**, **Print**, or **Copy** the enterprise **SSO recovery codes** and store them securely.

> 📌 **OIDC single sign-on for EMU is supported only with Microsoft Entra ID** (it also enables Conditional Access). Okta, PingFederate, and other IdPs use SAML.

> 🔐 **SCIM token:** create the SCIM token as a **personal access token (classic)** with only the `scim:enterprise` scope (**No expiration** recommended) while signed in as the setup user, under **Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)** → **Generate new token (classic)**. Copy it immediately (shown once) and paste it into the IdP provisioning connection. There is no "Enable SCIM provisioning" toggle on the GitHub side.

| Step | Action |
|------|--------|
| **1. Configure SSO** | Set up SAML (any supported IdP) or OIDC (Entra only) via **Identity provider → Single sign-on configuration** as above |
| **2. Configure SCIM** | Paste the `scim:enterprise` classic PAT into your IdP's provisioning connection, using the enterprise SCIM tenant URL (`https://api.github.com/scim/v2/enterprises/{ENTERPRISE_SLUG}`) |
| **3. Provision users** | Assign users and groups in your IdP — they will be auto-created in GitHub |
| **4. Map groups to teams** | IdP groups map to GitHub teams for repository access |

### Guest Collaborators (EMU)

**👤 Role:** GitHub **enterprise owner** + IdP admin · **📍 Portal:** GitHub + your IdP

**Navigate:** **Enterprises** page (github.com/settings/enterprises) → *[enterprise]* → **Policies**

> 💡 **Tip:** Guest collaborators let external users (contractors, partners) work on a limited set of repositories in an EMU enterprise. They **are still provisioned through the IdP** — the identity provider assigns the **guest collaborator** role to a managed user (an Entra app-manifest role or an Okta profile-editor role) and the user is created via SCIM. There is no GitHub-side invitation flow. The enterprise-side control lives under the top-level **Policies** tab, which governs whether org and repo admins may add collaborators.

---

## 4️⃣ Apply Baseline Governance

### A) Default Repository Visibility

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** **Enterprises** page (github.com/settings/enterprises) → *[enterprise]* → **Policies** → **Member privileges**

1. Under **Repository creation**, set the default visibility (**Private** recommended) and restrict which visibility levels members may choose.
2. Click **Save**.

| Setting | Recommended Value |
|---------|------------------|
| **Default visibility** | Private |
| **Allow public repos** | Only if Standard Enterprise and intentional |
| **Allow internal repos** | Yes (for cross-org sharing within the enterprise) |

---

### B) Repository Rulesets

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **Policies** tab → **Code**

1. Click **New ruleset**, then **New branch ruleset** (or **New tag ruleset**).
2. Enter a **Ruleset name** and change **Enforcement status** from **Disabled** to **Active** (or **Evaluate** to test first).
3. Choose the target **organizations**, **repositories**, and **branches** (for example **Include default branch**).
4. Select the rules (see the table below).
5. Click **Create**.

| Ruleset Type | Recommended Rules |
|-------------|-------------------|
| **Branch ruleset (main)** | Require PR reviews, require status checks, block force pushes, require signed commits |
| **Tag ruleset** | Restrict who can create release tags |

> 💡 **Tip:** Enterprise-level rulesets cascade to all organizations and repositories. Use them for non-negotiable guardrails (e.g., no force pushes to `main`).

---

### C) Actions Policies

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** **Enterprises** page (github.com/settings/enterprises) → *[enterprise]* → **Policies** → **Actions**

1. Configure the allowed actions and workflows (see the table below).
2. Click **Save**.

| Setting | Recommended Value |
|---------|------------------|
| **Allow actions** | Allow select actions → GitHub-authored + verified marketplace + specific trusted actions |
| **Fork pull request workflows** | Require approval for first-time contributors |
| **Default workflow permissions** | Read repository contents (not write) |

---

### D) Personal Access Token (PAT) Policies

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** **Enterprises** page (github.com/settings/enterprises) → *[enterprise]* → **Policies** → **Personal access tokens**

1. Configure the fine-grained and classic PAT policies (see the table below).
2. Click **Save** for each policy you change.

| Setting | Recommended Value |
|---------|------------------|
| **Fine-grained PATs** | Allow — with approval required for organization access |
| **Classic PATs** | Restrict or block (fine-grained preferred) |
| **Max token lifetime** | Set a maximum (e.g., 90 days) |

---

### E) GitHub App Governance

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** **Enterprises** page (github.com/settings/enterprises) → *[enterprise]* → **Policies** → **GitHub Apps**

1. Configure the app installation and approval policies (see the table below).
2. Click **Save**.

| Setting | Recommended Value |
|---------|------------------|
| **Who can install apps** | Organization owners only |
| **App approval** | Require enterprise owner approval for new app installations |

---

## 5️⃣ Set Up Security Controls

### A) Enable Security Configurations at Scale

**👤 Role:** GitHub **organization owner** (or security manager) · **📍 Portal:** GitHub

**Navigate:** Organization → **Settings** → **Advanced Security** → **Configurations**

1. Click **New configuration**. Use the quick setup dialog (**Review** → **Save and enable**), or choose **Custom configuration**, set the features, and click **Save configuration**.
2. On the **Repositories** tab, select a pilot set (or **Select all**), click **Apply configuration ▾**, choose the configuration, and click **Apply**.
3. *(Optional)* In the configuration's **Policy** section, make it the default for new repositories and choose **Enforce**.

> 💡 **Tip:** Use the Organization-level Security Configurations to apply consistent security settings across all repos. See the [GitHub Secret Protection Enablement Runbook](../Security/Secret%20Protection%20Enablement.md) for detailed steps.

---

### B) Secret Scanning + Push Protection

**👤 Role:** GitHub **organization owner** (or security manager) · **📍 Portal:** GitHub

**Navigate:** Organization → **Settings** → **Advanced Security** → **Configurations**

1. Click **New configuration** → **Custom configuration** (or edit an existing one).
2. Turn on **Secret Protection** — this enables secret scanning alerts.
3. Set **Push protection** to **Enabled** (and optionally **Validity checks**).
4. Click **Save configuration**, then apply it on the **Repositories** tab.

| Feature | Description |
|---------|-------------|
| **Secret scanning alerts** | Detect secrets already present in repositories |
| **Push protection** | Block secrets from being pushed going forward |
| **Validity checks** | Verify if detected secrets are still active |

---

### C) Code Scanning with CodeQL

**👤 Role:** GitHub **organization owner** (or security manager) · **📍 Portal:** GitHub

**Navigate:** Organization → **Settings** → **Advanced Security** → **Configurations**

1. Click **New configuration** → **Custom configuration** (or edit an existing one).
2. Turn on **Code Security**.
3. Set **Default setup** to **Enabled** (or **Enabled with advanced setup allowed** if some repos run their own CodeQL workflow).
4. Click **Save configuration**, then apply it on the **Repositories** tab.

| Setting | Recommended Value |
|---------|------------------|
| **Default setup** | Enable for all supported languages |
| **Schedule** | Weekly + on pull requests |
| **Alert severity threshold** | High and Critical block merges (via rulesets) |

---

## 6️⃣ Configure Billing

### A) Connect Azure Subscription or EA Billing

**👤 Role:** GitHub **enterprise owner** + Azure **subscription Owner** + tenant-wide admin consent · **📍 Portal:** GitHub + Microsoft

**Navigate:** **Enterprises** page (github.com/settings/enterprises) → *[enterprise]* → **Billing and licensing** → **Payment information** → scroll to **Metered billing via Azure**

1. Click **Add Azure Subscription**.
2. Sign in to Microsoft, review **Permissions requested**, and click **Accept**.
3. On **Select a subscription**, choose the subscription, check the confirmation box, and click **Connect**.

> 📌 **Constraint:** A **billing manager** cannot connect a subscription — connecting requires a GitHub **enterprise owner**. (Enterprise Agreement customers configure EA billing with their GitHub account team instead.)

> ⚠️ **Important:** An Azure subscription or Enterprise Agreement must be connected before any paid features (Secret Protection, Code Security, Copilot) can be enabled at scale.

---

### B) Create Cost Centers

**👤 Role:** GitHub **enterprise owner** (or billing manager) · **📍 Portal:** GitHub

**Navigate:** **Enterprises** page (github.com/settings/enterprises) → *[enterprise]* → **Billing and licensing** → **Cost centers**

1. Click **New cost center** and enter a **Name**.
2. Assign the organizations, repositories, or users that should carry the spend.
3. Click **Create**.

| Cost Center Example | Assigned To |
|--------------------|-------------|
| **Platform Engineering** | platform-engineering org |
| **Business Unit A** | business-unit-a org |
| **Copilot Pilot** | Specific user group |

---

### C) Set Budgets and Hard Stops

**👤 Role:** GitHub **enterprise owner** (or billing manager) · **📍 Portal:** GitHub

**Navigate:** **Enterprises** page (github.com/settings/enterprises) → *[enterprise]* → **Billing and licensing** → **Budgets and alerts**

1. Click **New budget** and set the amount.
2. Assign the budget to a cost center, organization, or the whole enterprise.
3. Enable budget alerts — GitHub automatically emails account owners and billing managers as spending approaches and reaches the budget limit; set the budget amount and, if desired, a hard limit that blocks further usage.
4. Click **Create**.

> 💡 **Tip:** Set spending alerts well below your actual budget so you have time to react before hitting limits.

---

## 7️⃣ Enable Copilot

### A) Turn Copilot On and Assign Licenses

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** your enterprise (EMU accounts: profile picture → **Enterprise**; standard accounts: **Enterprises** page → *[enterprise]*) → **Billing and licensing** → **Licensing** → **Manage** *(in the "Copilot" section)*

1. Next to **Organization access**, choose all organizations or **Allow for specific organizations** — for specific ones, click the **Organizations** tab and set each organization's **Copilot** dropdown to **Enabled**. *(Applies immediately — there is no Save button.)*
2. To license people directly (Copilot Business): click the **All members** or **Enterprise Teams** tab → **Assign licenses** → search → **Add licenses**. Set the **Policies for enterprise-assigned users** policy (step B) first.
3. Or let organization owners assign seats: org **Settings** → **Copilot** → **Access** → **Start adding seats**.

> 💡 Enterprise-level Copilot Business management is generally available, and **enterprise teams are GA** (since June 2026) — license a team and people gain or lose Copilot as they join or leave it.

---

### B) Set Copilot Policies and Models

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** your enterprise → **AI controls** → **Copilot** *(sidebar)*

1. On the **Copilot** page, set administration, privacy, model, billing, and usage policies — for each dropdown choose **Enabled**, **Disabled**, or **Let organizations decide**.
2. Click **Configure models**, then set each model to **Enabled**, **Disabled**, or **Delegate** (lets organizations decide).
3. Under "Features & clients", click **Configure features & clients** to set feature and client policies. Use the **Agents** and **MCP** sidebar pages for agent and MCP policies.

> ⏰ **Before October 22, 2026:** set the **Default policy for new features** on the **Copilot** page (**Enabled**, **Disabled**, or **Let organizations decide**). From that date, GA features left **Unconfigured** follow it — and it's **Enabled** by default.

> 📌 Policy and model selections apply immediately — there is no Save button.

---

### C) Configure Content Exclusions

**👤 Role:** GitHub **enterprise owner** (whole enterprise) or **organization owner** (one organization) · **📍 Portal:** GitHub

**Navigate (enterprise):** your enterprise → **AI controls** → **Copilot** → **Content exclusion**
**Navigate (organization):** org **Settings** → **Copilot** → **Content exclusion**

1. Open **Content exclusion**.
2. Enter the repositories and paths to exclude, one pattern per line — for example, `"*":` followed by `- "**/.env"` excludes every `.env` file everywhere.
3. Save your changes.

> 💡 **Tip:** Use content exclusions to keep Copilot away from sensitive files (e.g., `**/*.env`, `**/secrets/**`).

---

### D) Custom Instructions

**👤 Role:** GitHub **organization owner** · **📍 Portal:** GitHub

**Navigate:** Profile picture → **Organizations** → *[organization]* → **Settings** → **Copilot** → **Custom instructions**

1. Under **Preferences and instructions**, write your coding guidelines, style rules, or organizational standards in plain language.
2. Click **Save changes**.

> 📌 Custom instructions are set per **organization** — there's no enterprise-wide setting. They apply to Copilot Chat, Copilot code review, and Copilot cloud agent on GitHub.com.
---

## 8️⃣ Validation Checklist

*Run through this checklist to confirm everything is working end-to-end*

| # | Validation Step | How to Test | Expected Result |
|---|----------------|-------------|-----------------|
| 1 | **SSO works** | Log in with an IdP-managed account | User is authenticated via SAML/OIDC |
| 2 | **SCIM provisions users** (EMU) | Assign a test user in the IdP | User appears in the enterprise within minutes |
| 3 | **SCIM deprovisions users** (EMU) | Unassign the test user in the IdP | User is suspended in GitHub |
| 4 | **Policies cascade** | Check an org-level repo for enterprise ruleset | Enterprise rulesets appear and are enforced |
| 5 | **Default repo visibility** | Create a new repo in any org | Default visibility matches enterprise policy |
| 6 | **Actions policies enforced** | Try to use a disallowed action in a workflow | Workflow fails with a policy error |
| 7 | **Secret scanning active** | Push a test secret to a test repo | Alert is generated (or push is blocked) |
| 8 | **Code scanning active** | Open a PR with a known vulnerability pattern | CodeQL flags the issue |
| 9 | **Copilot available** | Open VS Code with Copilot extension, sign in | Copilot provides suggestions |
| 10 | **Billing connected** | Check Enterprise → Billing | Azure subscription or EA is active, usage is tracked |
| 11 | **Cost centers reporting** | Check Enterprise → Billing and licensing → Cost centers | Usage is allocated to the correct cost centers |

> ✅ **Result:** If all validation steps pass, your GitHub Enterprise Cloud environment is scaffolded and ready for onboarding teams.

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


### Q: Should we use one enterprise or multiple enterprises?
**A:** Use one enterprise per company in almost all cases. Multiple enterprises add significant complexity: separate billing, separate audit logs, separate policy governance, and no shared visibility. Only consider multiple enterprises when you have hard compliance boundaries (e.g., FedRAMP vs non-FedRAMP workloads), completely independent IdPs that cannot federate, or legally distinct entities with no shared governance. If in doubt, start with one enterprise and use organizations for separation.

---

### Q: How many organizations should we create inside our enterprise?
**A:** Keep the number low. Create separate organizations only when you need distinct admin models, compliance boundaries, cost centers for billing, or different IdPs (standard enterprise with per-org SAML). Do not create one org per team or one org per project — use teams and repositories within a single org instead. A typical enterprise has 2-5 organizations (e.g., engineering, platform, security, sandbox).

---

### Q: When should we use internal vs private repository visibility?
**A:** Use **internal** for repositories that should be discoverable and readable by everyone in the enterprise across all organizations — this enables InnerSource and knowledge sharing. Use **private** for repositories with restricted access where only explicitly granted users or teams should see the code. A good default is to make most repos internal and reserve private for sensitive or regulated code.

---

### Q: We have shared platform repos (Actions, templates, IaC) — where should they live?
**A:** Create a dedicated "platform" or "shared" organization for cross-cutting resources like reusable Actions, workflow templates, Terraform modules, and internal packages. Set these repos to **internal** visibility so all enterprise members can use them. This avoids duplication across orgs and establishes a single source of truth for shared tooling. Grant write access only to the platform team.

---

### Q: Enterprise policies I set are not cascading to organizations as expected — what is happening?
**A:** For most enterprise policies you either pick a specific setting — which is then enforced on every organization, so org owners can't change it — or leave the policy at **Let organizations decide**, which lets each organization owner decide. Some policies offer options that only limit what org owners can choose (for example, which repository visibilities members can create). If org owners can still change something, the enterprise policy is probably at **Let organizations decide**. Go to your enterprise → **Policies** (a top-of-page tab, not under **Settings**), open the relevant policy page, and check its value. Enterprise rulesets are layered on top of organization and repository rules — org-level rules can't relax them — but anyone on a ruleset's bypass list can still bypass it.

---

### Q: We already set up our environment but realize we chose the wrong identity model — can we switch?
**A:** No. The identity model (Standard vs EMU vs EMU with Data Residency) is set at enterprise creation and cannot be changed. Switching requires creating a new enterprise with the correct identity model, reconfiguring identity and provisioning, and migrating all repositories using GitHub Enterprise Importer (GEI). Treat this as a 4-8 week migration project. This is why the identity model decision in Step 1 is the most important choice in this guide.

---

### Q: How should we handle cost allocation across multiple business units?
**A:** Use GitHub's Cost Centers feature (Enterprise → **Billing and licensing** → **Cost centers**). Create a cost center for each business unit or department and assign the organizations, repositories, or users that should carry that spend. User-scoped cost centers are especially useful for Copilot seats and metered AI usage — as of 2026-06-01 GitHub Copilot moved to usage-based billing, so Copilot consumption is now measured in **GitHub AI Credits** rather than the former "premium requests." Repository-scoped cost centers are useful for repository-driven metered usage such as Actions. Enable budget alerts — GitHub automatically emails account owners and billing managers as spending approaches and reaches the budget limit; set the budget amount and, if desired, a hard limit that blocks further usage.

---

### Q: Our security team wants to enable Advanced Security (GHAS) for all repos — should we do it at once?
**A:** Enable incrementally. Start by applying a security configuration to a pilot set of repositories or one organization. Review the initial alerts (secret scanning, code scanning) and establish a triage process before rolling out broadly. Enabling GHAS across hundreds of repos at once can generate a flood of alerts that overwhelm teams. Use org-level Security Configurations to apply settings consistently, and ramp up over 2-4 weeks.

</details>

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Organization Design Patterns | `Setup/Organization Design Patterns (Flat Structure, Teams, Naming).md` |
| Branch Protection Rules & Rulesets | `Governance/Branch Protection Rules & Rulesets.md` |
| Secret Protection Enablement | `Security/Secret Protection Enablement.md` |
| Copilot Admin Controls | `Copilot/Admin Controls (Models, Content Exclusion, Instructions).md` |
| Cost Centers & Department Billing | `Billing/Cost Centers & Department Billing.md` |

---

## 📝 Resources

- [About GitHub Enterprise Cloud](https://docs.github.com/en/enterprise-cloud@latest/admin/overview/about-github-enterprise-cloud)
- [Getting started with GitHub Enterprise Cloud](https://docs.github.com/en/get-started/onboarding/getting-started-with-github-enterprise-cloud)
- [Enterprise Managed Users](https://docs.github.com/en/enterprise-cloud@latest/admin/concepts/identity-and-access-management/enterprise-managed-users)

---

*Last updated: October 2026*
