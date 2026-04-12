# 🚀 Guide to Setup GHEC (Standard / Non-EMU) + OneLogin (SAML) + Attach Azure Subscription + Turn on Copilot

> **Complete end-to-end runbook for configuring Standard (non-EMU) GHEC with OneLogin (SAML), SCIM org provisioning, Azure billing, and GitHub Copilot**

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **OneLogin:** Applications → Add App → "GitHub Enterprise Cloud - Organization" → Configuration tab → Enter org slug → SSO tab → Capture SAML 2.0 Endpoint, Issuer URL, X.509 cert
- **GitHub:** Org Settings → Authentication security → SAML single sign-on → Paste OneLogin values → Test → Save → Enforce SAML SSO
- **SCIM:** Sign in as setup user → Org Settings → Authentication security → Generate SCIM token → OneLogin App → Provisioning → Enter SCIM URL + token
- **Billing:** Org/Enterprise → Billing & Licensing → Payment information → Add Azure Subscription → Accept → Connect
- **Copilot:** Enterprise AI controls → Copilot → Enable → Org Settings → Copilot → Access → Assign seats

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

## 0️⃣ Prerequisites Checklist

| Requirement | Owner / Role | Notes |
| --- | --- | --- |
| GitHub Enterprise Cloud (Standard / non-EMU) org | Org Owner | You must be able to access Org Settings → Authentication security |
| (If applicable) GitHub Enterprise account | Enterprise Owner | Needed only if you will configure Enterprise SAML and/or Enterprise Copilot policies |
| OneLogin Admin access | OneLogin Admin | Must be able to install and configure apps in the OneLogin admin portal |
| Pilot users/groups in OneLogin | IAM team | Use a small pilot cohort first to reduce risk |
| Dedicated GitHub "SCIM setup user" | Org Owner | GitHub recommends a dedicated user to generate the SCIM API token used by OneLogin |
| Azure subscription + ability to consent | Azure admin | Needed to connect metered billing via Azure. If subscription is in a different tenant, you may need to specify a different tenant ID during connection |
| Copilot plan decision | Enterprise/Org owner | Copilot Business vs Copilot Enterprise |

## 1️⃣ Create & Secure the "SCIM Setup User" (Standard non-EMU)

> ⚠️ **Important:** This user is required because OneLogin's GitHub SCIM provisioning uses an API token generated from a GitHub user account. If that user loses access or leaves the org, SCIM can stop working—so you want a stable, dedicated identity.

### Process

1. Create a dedicated GitHub.com account (example: `gh-onelogin-scim@yourdomain.com`)
2. **Secure the account:**
    - Enable 2FA
    - Ensure recovery methods are stored per your approved process (vault / break-glass)
3. **Add the setup user to your GitHub organization as an Owner:**
    - GitHub → your Organization → Settings → People → Invite member
    - After accepted → set role to Owner
4. **Assign this same identity in OneLogin** to the GitHub app (so it can SSO and be used to generate SCIM credentials)

### Important Notes

> 📌 **Note:** This setup user will consume a GitHub license. Treat it as a system account: minimal use outside of IAM configuration.

## 2️⃣ Create the OneLogin Application (GitHub Enterprise Cloud – Organization)

### Navigation (OneLogin)

```
OneLogin Admin Portal
  → Applications
    → Add App
      → Search: "GitHub Enterprise Cloud - Organization"
        (or "GitHub Organization")
          → Select and Add

```

### Configuration Steps

1. **Select the OneLogin catalog app:** `GitHub Enterprise Cloud - Organization`
    - ⚠️ Do NOT use "GitHub Enterprise Managed User" (that is for EMU)
2. **Fill in required fields:**
    - In the app → **Configuration** tab → enter your GitHub organization name (org slug)
    - Application label / display name (unique name in OneLogin)
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

## 3️⃣ Configure SAML (GitHub ↔ OneLogin Values)

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

## 4️⃣ Enable & Test SAML SSO in GitHub

You will configure **Organization SAML** (always for the target org).

If your org is under an Enterprise account, you may also configure **Enterprise SAML**.

> ⚠️ **Warning:** Enabling SAML impacts how members authenticate. Ensure you have recovery codes stored for break-glass access.

### 4A — (Conditionally Required) Configure Enterprise SAML

**When this step is required:**

- If you are a GitHub Enterprise (enterprise account) customer and intend to require SAML at the enterprise level, complete this before org enforcement. **Enterprise SAML completely replaces org-level SAML configuration and enforces SAML SSO for every organization in the enterprise.**

**Navigation Path (GitHub UI)**

```
GitHub (top-right profile picture)
  → Enterprises
    → [Select enterprise]
      → Settings
        → Authentication security
          → SAML single sign-on

```

**Configuration Steps**

1. In Enterprise settings → Authentication security → SAML single sign-on
2. Enable / configure SAML per GitHub's enterprise SAML documentation (OneLogin values differ from org app)

### 4B — Enable & Test Organization SAML (Required)

**Navigation Path (GitHub UI)**

```
GitHub (top-right profile picture)
  → Organizations
    → (next to your org) Settings
      → Authentication security
        → SAML single sign-on

```

**Configuration Steps (Organization SAML)**

1. Under **SAML single sign-on**, select **Enable SAML authentication**
2. Populate fields using OneLogin values from Step 2:
    - **Sign on URL** — OneLogin SAML 2.0 Endpoint (HTTP)
    - **Issuer** — OneLogin Issuer URL — _Note: GitHub indicates Issuer is required for some features like team synchronization_
    - **Public Certificate** — OneLogin X.509 Certificate
3. Click **Test SAML configuration** (or equivalent prompt) and complete the auth flow
4. Click **Save**
5. **Immediately download and secure SSO recovery codes:**
    - Organization Settings → Authentication security → Single sign-on recovery codes

> 🔐 **Critical:** Before enabling or immediately after enabling SAML, download and securely store your organization SSO recovery codes. These are essential for break-glass scenarios if your IdP becomes unavailable.

## 5️⃣ Enforce SAML SSO for the Organization (Required)

> ⚠️ **Important:** Enforcement removes org members who have not authenticated through the IdP, and can also remove bots/service accounts that don't have external identities. **If a user rejoins the organization within three months, the user's access privileges and settings will be restored.**

### Navigation Path (GitHub UI)

```
GitHub (top-right profile picture)
  → Organizations
    → (next to your org) Settings
      → Authentication security
        → SAML single sign-on

```

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

## 6️⃣ Configure SCIM Provisioning (OneLogin → GitHub Organization)

> 💡 **Tip:** OneLogin supports SCIM provisioning for GitHub organizations via an API token. You generate a SCIM token from GitHub org settings and enter it into OneLogin's provisioning configuration.

### 6A — Generate SCIM Credentials in GitHub (Required)

1. Sign into GitHub as the SCIM setup user
2. Navigate to your organization's SAML settings:

```
GitHub (top-right profile picture)
  → Organizations
    → (next to your org) Settings
      → Authentication security
        → SAML single sign-on

```

3. Under SCIM provisioning, generate or locate the SCIM API token
4. Copy the SCIM base URL and bearer token — you will need these for OneLogin

### 6B — Enable Provisioning in OneLogin (Required)

**Navigation (OneLogin)**

```
OneLogin Admin Portal
  → Applications
    → [Your "GitHub Enterprise Cloud - Organization" app]
      → Provisioning

```

**Steps**

1. Click **Enable provisioning**
2. Enter the SCIM credentials from GitHub:
    - SCIM Base URL
    - SCIM Bearer Token (API token from Step 6A)
3. Configure provisioning actions:
    - Create users
    - Update users
    - Deactivate users
4. Click **Save**

### 6C — Configure Provisioning Mappings (Required)

1. In the OneLogin app → **Provisioning** or **Parameters**
2. Keep mappings aligned to GitHub's SCIM guidance; avoid custom attributes until the base flow is stable
3. Ensure NameID / email mapping is consistent between SAML and SCIM

## 7️⃣ Assign Users & Groups

**Navigation Path (OneLogin)**

```
OneLogin Admin Portal
  → Applications
    → [Your "GitHub Enterprise Cloud - Organization" app]
      → Users
        → Add users or groups

```

**Steps**

1. In OneLogin → the GitHub app → **Users** (or **Access** tab)
2. Assign:
    - Pilot group(s) first
3. **Validate lifecycle:**
    - Add assignment → user becomes org member (via SCIM)
    - Remove assignment → user is removed from org (per your provisioning settings)

## 8️⃣ Attach Azure Subscription for Metered Billing

**Required if you are billing via Azure**

### Prerequisites

- ✓ You are an Owner of the GitHub org or enterprise account you are connecting
- ✓ You know the Azure subscription ID
- ✓ You are logged into Azure with a user who can provide tenant-wide admin consent (or you have an admin-consent workflow)
- ✓ **If the Azure subscription is in a different tenant than your default, you may need to specify a different tenant ID during connection**

### Configuration Steps (GitHub)

**Navigation Path:**

```
GitHub
  → Your org or enterprise settings entry point
    → Org list: https://github.com/settings/organizations
    → Enterprise list: https://github.com/settings/enterprises
      → Open the target org/enterprise
        → Billing & Licensing
          → Payment information
            → Metered billing via Azure
              → Add Azure Subscription

```

**Process:**

1. Navigate to your org or enterprise settings entry point:
    - Org list: `https://github.com/settings/organizations`
    - Enterprise list: `https://github.com/settings/enterprises`
2. Open the target org/enterprise
3. Click **Billing & Licensing:**
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

## 9️⃣ Enable GitHub Copilot (Enterprise + Organization)

### 9A — (Conditionally Required) Enable Copilot at the Enterprise level via Payment Verification

**When this step is required:**

- If you manage Copilot through an enterprise account, GitHub's setup flow enables Copilot via enterprise payment verification.

**Navigation Path (GitHub UI)**

```
GitHub (top-right profile picture)
  → Enterprise (or Enterprises → select enterprise)
    → Settings
      → Getting Started
        → Verify your payment method

```

Complete **Verify your payment method** to enable Copilot in the enterprise.

### 9B — Configure Copilot Enterprise Policies

**Required if you manage policies at enterprise level**

**Navigation Path (GitHub UI)**

```
GitHub (top-right profile picture)
  → Enterprise (or Enterprises → select enterprise)
    → AI controls
      → Copilot

```

From here you can set enterprise-wide Copilot policy enforcement.

**Policy Options:**

- For each policy, select:
    - **Enabled/Allowed** — Feature is on for all organizations
    - **Disabled/Blocked** — Feature is off for all organizations
    - **No policy** — Delegate decision to organization owners

### 9C — Configure Copilot Organization Policies (Required)

Even with enterprise governance, you should confirm org-level visibility and configuration.

**Navigation Path (GitHub UI)**

```
GitHub (top-right profile picture)
  → Organizations
    → (next to your org) Settings
      → (sidebar) Code, planning, and automation
        → Copilot
          → Policies / Models

```

Set:

- Feature availability policies
- Model availability policies (if applicable)

### 9D — Assign Copilot licenses (Required)

**Copilot Business:** You can assign licenses to individual users who don't consume a GitHub Enterprise license.

**Copilot Enterprise:** You typically enable for entire organizations, and all members consume a GitHub Enterprise license.

**Navigation Path:**

```
Organization Settings
  → Copilot
    → Access
      → Add members or enable for all

```

Organization owners can assign Copilot seats to individual members or teams.

## 🔟 Critical Post-Enablement: SSO Authorization for Credentials (Required)

When SAML is enabled/enforced, users often must authorize credentials (depending on token type and whether they have a linked external identity).

### 10A — Authorize SSH Keys for SSO

**Required for SSH usage in SSO orgs**

**Navigation**

```
GitHub (top-right profile picture)
  → Settings
    → SSH and GPG keys
      → (next to the key) Configure SSO
        → Authorize (for the org)

```

### 10B — Authorize Personal Access Tokens

**Required for PAT classic in SSO orgs**

**Navigation**

```
GitHub (top-right profile picture)
  → Settings
    → Developer settings
      → Personal access tokens
        → (next to the token) Configure SSO
          → Authorize (for the org)

```

**Token nuance:**

> 📌 **Note:** GitHub states **PAT classic** requires post-creation SSO authorization. **Fine-grained PATs** are authorized during creation, before org access is granted.

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

### SCIM

- [ ] Setup user is an org owner
- [ ] SCIM API token generated from GitHub org settings
- [ ] OneLogin provisioning configured with SCIM credentials
- [ ] Assign/unassign test behaves as expected

### Azure Billing

- [ ] Azure subscription successfully connected under Payment information
- [ ] "Metered billing via Azure" shows the correct subscription ID

### Copilot

- [ ] Enterprise payment verification completed (if enterprise-managed)
- [ ] Enterprise Copilot policies set (if applicable)
- [ ] Org Copilot policies set
- [ ] Licenses assigned to pilot cohort
- [ ] Pilot users can use Copilot in IDE / GitHub.com as expected

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

_Last updated: April 2026_

## ❓ Common Questions & Troubleshooting

### Q: How do I set up the SCIM API token in OneLogin for GitHub provisioning?
**A:** Sign in to GitHub as the SCIM setup user, navigate to Organization Settings > Authentication security, and under SCIM provisioning, generate a SCIM token. Copy this token and the SCIM base URL. In OneLogin, go to the GitHub app > Provisioning tab, enable provisioning, and enter the SCIM Base URL and Bearer Token. Click Save and test by assigning a pilot user. The token is displayed only once — regenerate it if lost.

---

### Q: OneLogin's SAML certificate is expiring — how do I renew it?
**A:** In OneLogin, go to the GitHub app > SSO tab > Manage Certificates. Generate a new certificate. Before activating the new cert in OneLogin, download it and paste it into GitHub (Organization Settings > Authentication security > SAML > Public certificate). Save in GitHub first. Then set the new certificate as active in OneLogin and remove the old one. This prevents a mismatch window where GitHub does not trust the new cert.

---

### Q: SAML assertion attribute mapping errors are preventing sign-in — what should I verify?
**A:** In OneLogin, go to the GitHub app > Parameters tab. Verify that the NameID field maps to the user's email address (this is what GitHub uses for identity linking). Check the SSO tab to confirm the SAML Signature Algorithm is set to SHA-256 (not SHA-1, which can cause validation failures). Also verify the Issuer URL and SAML 2.0 Endpoint match what you entered in GitHub. Use the OneLogin SAML Toolkit or browser developer tools to inspect the actual assertion being sent.

---

### Q: Which OneLogin connector version should I use — there seem to be multiple?
**A:** Use the **"GitHub Enterprise Cloud - Organization"** connector from the OneLogin catalog. Older connectors may be labeled differently or target deprecated endpoints. If you see multiple versions, choose the one most recently updated in the catalog. Avoid using connectors labeled for "GitHub Enterprise Managed User" (that is for EMU, not standard orgs). If provisioning behaves unexpectedly, check the OneLogin release notes for your connector version.

---

### Q: Users are authenticated via SAML but not getting added to the org — SCIM does not seem to be working. What is wrong?
**A:** Verify provisioning is enabled in OneLogin (GitHub app > Provisioning tab) and that Create Users, Update Users, and Deactivate Users are all checked. Confirm the SCIM credentials (base URL and bearer token) are correct and the token has not expired. Check the OneLogin Events log for provisioning errors. Also ensure the user is assigned to the GitHub app in OneLogin — SAML authentication alone does not trigger SCIM provisioning.

---

### Q: After SAML enforcement, some users lost access — how do I restore them?
**A:** Users who had not authenticated via the IdP before enforcement are removed from the org. If they rejoin within three months, their previous access privileges and settings are restored automatically. Direct them to `https://github.com/orgs/YOUR_ORG/sso` to authenticate. For bots or service accounts, either assign IdP identities to them or migrate to GitHub Apps. Review the enforcement removal list in Organization > People > filter by "Removed."

---

### Q: Can OneLogin handle group-based provisioning to GitHub, or only individual user assignment?
**A:** OneLogin supports both individual user and group-based assignment. You can assign groups under the GitHub app > Users/Access tab. However, OneLogin's SCIM provisioning for GitHub only manages org membership — it does not natively sync groups to GitHub teams. For team-based access control, you will need to manage GitHub team memberships separately or use the GitHub Teams API.

---

### Q: SCIM provisioning worked initially but has stopped syncing new users — what happened?
**A:** The most common cause is an expired or invalidated SCIM bearer token. Tokens can be invalidated if the setup user's org membership changes or if the token is regenerated. Check the OneLogin Events log for HTTP 401 or 403 errors from the GitHub SCIM endpoint. If the token is the issue, regenerate it in GitHub org settings and update it in OneLogin's Provisioning configuration.

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
