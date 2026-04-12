# 🚀 GitHub Enterprise Managed Users (EMU) Setup Guide (SAML)

> **Complete end-to-end runbook for configuring EMU with Microsoft Entra ID (SAML), Azure billing, and GitHub Copilot**

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Entra ID:** Create Enterprise App "GitHub Enterprise Managed User" from gallery → Configure SAML with Entity ID `https://github.com/enterprises/YOUR_ENTERPRISE`
- **GitHub:** Sign in as `SHORTCODE_admin` → Enterprise Settings → Authentication security → Paste Sign-on URL, Issuer, Base64 cert → Save
- **SCIM:** As `SHORTCODE_admin`, generate PAT with `scim:enterprise` scope → Enter in Entra App → Provisioning → Automatic → Test Connection
- **Billing:** Enterprise → Billing & licensing → Payment information → Add Azure Subscription → Accept permissions → Connect
- **Copilot:** Enterprise Settings → AI controls → Copilot → Enable for orgs → Org Settings → Copilot → Access → Assign seats

---

## 📋 Overview

This guide walks through setting up a new GitHub Enterprise Cloud (GHEC) with Enterprise Managed Users (EMU), including:

- ✅ EMU with Microsoft Entra ID (SAML SSO)
- ✅ SCIM provisioning for user lifecycle management
- ✅ Azure subscription attachment for billing
- ✅ GitHub Copilot enablement
- ✅ Organization structure guidance

**Hosting Options:** EMU can be hosted on GitHub.com or (for data residency) on a customer subdomain of GHE.com. Setup is similar, but SAML/SCIM URLs differ.

---

## 0️⃣ Prerequisites Checklist

### Required Items

Before beginning, ensure you have:

| Requirement | Status |
|-------------|--------|
| **EMU Enterprise Created** — A new enterprise with EMU enabled | ☐ |
| **Setup User** — `SHORTCODE_admin` created by GitHub, with password set and 2FA enabled | ☐ |
| **Entra Admin Access** — Ability to create Enterprise Apps and configure SAML SSO + SCIM | ☐ |
| **Azure Subscription** — Subscription ID and someone who can grant tenant-wide admin consent | ☐ |

> 💡 **Note:** SCIM provisioning is **required** for EMU to manage user lifecycle and account creation.

---

## 1️⃣ Create & Configure the EMU Setup User

### Process

1. GitHub emails an invite to set the password for `SHORTCODE_admin`
2. In a **private/incognito window**:
   - Set password
   - Enable 2FA
   - Save recovery codes securely

### Important Notes

> ⚠️ **Setup User Purpose:** This account is primarily for SCIM provisioning via token and recovery scenarios. Day-to-day enterprise administration should be done with provisioned managed user accounts.

> 🚨 **Email Conflict:** If the provided email address is already associated as a primary email with an existing GitHub account, the activation link will not work. Modify the existing account's primary email first.

---

## 2️⃣ Create the Entra ID Enterprise Application

### Navigation

```
Microsoft Entra Admin Center
  → Entra ID
    → Enterprise applications
      → + New application
        → Browse Azure AD Gallery
          → Search: "GitHub Enterprise Managed User"
            → Create/Add
```

### SAML Configuration

1. Navigate to **Single sign-on** → Select **SAML**
2. Edit **Basic SAML Configuration**

### SAML Values

#### For GitHub.com Hosted EMU

| Field | Value |
|-------|-------|
| **Identifier (Entity ID)** | `https://github.com/enterprises/YOUR_ENTERPRISE` |
| **Reply URL (ACS URL)** | `https://github.com/enterprises/YOUR_ENTERPRISE/saml/consume` |
| **Sign-on URL** | `https://github.com/enterprises/YOUR_ENTERPRISE/sso` |

#### For GHE.com Hosted EMU (Data Residency)

| Field | Value |
|-------|-------|
| **Identifier (Entity ID)** | `https://SUBDOMAIN.ghe.com/enterprises/SUBDOMAIN` |
| **Reply URL (ACS URL)** | `https://SUBDOMAIN.ghe.com/enterprises/SUBDOMAIN/saml/consume` |
| **Sign-on URL** | `https://SUBDOMAIN.ghe.com/enterprises/SUBDOMAIN/sso` |

> **⚠️ Critical:** The Identifier format differs from the application's suggested format. Ensure the Identifier **does not** contain a trailing slash.

### Download Required Items

From the Entra Enterprise App:

1. **SAML certificate (Base64)** — Download this file
2. **Login URL** — Copy this value
3. **Microsoft Entra Identifier** — Copy this value

### Assign Users

Navigate to **Users and Groups** and assign one or more users with the **Enterprise Owner** role.

---

## 3️⃣ Enable SAML SSO in GitHub

### Navigation Path

```
GitHub (as SHORTCODE_admin)
  → Profile picture (top-right)
    → Your enterprises
      → Select your enterprise
        → Settings
          → Authentication security
```

### Configuration Steps

1. Locate the **SAML single sign-on** section
2. Check **Require SAML authentication**
3. Enter the following values from Entra:
   - **Sign on URL:** Paste the Login URL from Entra
   - **Issuer:** Paste the Microsoft Entra Identifier
   - **Public certificate:** Paste the contents of the Base64 certificate
4. Click **Save**

> **🔐 Critical:** Before enabling SAML, download and securely store your **enterprise SSO recovery codes**. These are essential for break-glass scenarios if your IdP becomes unavailable.

---

## 4️⃣ Configure SCIM Provisioning

SCIM handles user creation and deactivation in EMU. This requires creating a token in GitHub and configuring provisioning in Entra.

### 4A — Create the SCIM Token in GitHub

The token must be created as the setup user with specific requirements:

**Token Requirements:**
- ✓ Scope: `scim:enterprise`
- ✓ Expiration: No expiration
- ✓ Created by: `SHORTCODE_admin`

#### Creation Steps

```
GitHub (as SHORTCODE_admin)
  → Profile picture
    → Settings
      → Developer settings
        → Personal access tokens
          → Tokens (classic)
            → Generate new token (classic)
```

**Configuration:**
- **Note:** "Entra SCIM Provisioning" (or similar descriptive name)
- **Expiration:** No expiration
- **Scope:** Select `scim:enterprise` only

**🔑 Important:** Copy the token immediately after generation. You won't be able to see it again.

### 4B — Configure Provisioning in Entra

#### Navigation Path

```
Entra Admin Center
  → Entra ID
    → Enterprise applications
      → GitHub Enterprise Managed User
        → Provisioning
```

#### Configuration

1. Set **Provisioning Mode** = **Automatic**
2. Under **Admin Credentials**, enter:
   - **Tenant URL:** (see table below)
   - **Secret Token:** The PAT created in step 4A
3. Click **Test Connection**
4. Click **Save**

#### Tenant URL Values

| Environment | Tenant URL |
|-------------|------------|
| **GitHub.com** | `https://api.github.com/scim/v2/enterprises/YOUR_ENTERPRISE` |
| **GHE.com** | `https://api.SUBDOMAIN.ghe.com/scim/v2/enterprises/SUBDOMAIN` |

### 4C — Configure Attribute Mappings

1. Under **Mappings**, select **Synchronize Microsoft Entra users to GitHub Enterprise Managed User**
2. Review default attribute mappings (typically work for most scenarios)
3. Ensure **Provisioning Status** is set to **On** under Settings

### 4D — Assign Users/Groups for Provisioning

1. Navigate to **Users and groups** in the Enterprise App
2. Assign users and/or groups to the application
3. Entra will SCIM-provision these members into the EMU enterprise

#### Provisioning Notes

> **📌 Important Constraints:**
> - Entra does **not** provision nested groups
> - Provisioning cycle runs approximately every **40 minutes**
> - Use **"Provision on demand"** for immediate provisioning
> - Do not assign more than **1,000 users per hour** to avoid rate limits

> **⚠️ Unsupported Configuration:** The combination of Okta and Entra ID for SSO and SCIM (in either order) is explicitly **not supported** by GitHub.

---

## 5️⃣ Attach Azure Subscription for Billing

Connect your Azure subscription so GitHub usage (Copilot, Actions, Codespaces, etc.) is billed through Azure.

### Prerequisites

- ✓ Owner of the GitHub enterprise account
- ✓ Azure Subscription ID
- ✓ Azure user who can provide tenant-wide admin consent

### Configuration Steps

#### Navigation Path

```
GitHub
  → Your enterprise (https://github.com/enterprises/YOUR_ENTERPRISE)
    → Billing & licensing
      → Payment information
        → Metered billing via Azure
          → Add Azure Subscription
```

#### Process

1. Click **Add Azure Subscription**
2. Sign in to your Microsoft account
3. Review the **"Permissions requested"** prompt
4. Click **Accept**
5. Select your Azure Subscription ID
6. Check the confirmation box
7. Click **Connect**

> **💡 Admin Consent:** If you don't see a "Permissions requested" prompt and instead see a message about needing admin approval, you may need to configure an admin consent workflow in Azure or work with your Azure AD global administrator.

---

## 6️⃣ Enable GitHub Copilot

### 6A — Enable at Enterprise Level

With Azure billing connected via metered billing, Copilot usage will be billed through your Azure subscription.

#### Navigation Path

```
GitHub Enterprise Settings
  → AI controls (top navigation)
    → Copilot (sidebar)
```

#### Access Management Configuration

Under **Access management**, choose:

- **Disabled** — No organizations can use Copilot
- **All organizations** — Enable for all organizations in the enterprise
- **Specific organizations** — Select which organizations can use Copilot

Select the **Copilot tier** (Business or Enterprise) for each enabled organization.

### 6B — Configure Copilot Policies

1. Navigate to the **Policies** tab under **AI controls → Copilot**
2. Configure policies for:
   - Suggestions matching public code (Allowed/Blocked)
   - Copilot in GitHub.com
   - Copilot Chat in the IDE
   - Copilot in the CLI
   - Other feature policies

#### Policy Options

For each policy, select:
- **Enabled/Allowed** — Feature is on for all organizations
- **Disabled/Blocked** — Feature is off for all organizations
- **No policy** — Delegate decision to organization owners

### 6C — Assign Copilot Seats

After enabling at the enterprise level:

```
Organization Settings
  → Copilot
    → Access
      → Add members or enable for all
```

Organization owners can assign Copilot seats to individual members or teams.

---

## 7️⃣ Organization Structure Guidance

Organizations are boundaries for ownership, settings, and repository visibility patterns. EMU provides centralized identity and lifecycle management across all organizations.

### Recommended Patterns

#### 📐 Design Principles

**"Few orgs" bias:** Keep organization count low unless you have genuine separation needs:
- Compliance boundaries
- Distinct admin models
- Legal entity separation

#### 🏢 Common Structures

**Org-per-business-unit:**
- Use when autonomy differs
- Separate admins and policies
- Different billing/cost centers

**Org-per-environment:**
- Only if required (e.g., regulated prod code vs everything else)
- Otherwise, teams and repositories are usually sufficient

### Inside an Organization

**Teams:**
- Use **Teams** for repository access control
- Use **Entra groups** for consistent membership (synced via SCIM)
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

```yaml
SAML Entity ID:     https://github.com/enterprises/{ENTERPRISE_SLUG}
SAML ACS URL:       https://github.com/enterprises/{ENTERPRISE_SLUG}/saml/consume
SAML SSO URL:       https://github.com/enterprises/{ENTERPRISE_SLUG}/sso
SCIM Tenant URL:    https://api.github.com/scim/v2/enterprises/{ENTERPRISE_SLUG}
```

### GHE.com (Data Residency) Hosted EMU

```yaml
SAML Entity ID:     https://{SUBDOMAIN}.ghe.com/enterprises/{SUBDOMAIN}
SAML ACS URL:       https://{SUBDOMAIN}.ghe.com/enterprises/{SUBDOMAIN}/saml/consume
SAML SSO URL:       https://{SUBDOMAIN}.ghe.com/enterprises/{SUBDOMAIN}/sso
SCIM Tenant URL:    https://api.{SUBDOMAIN}.ghe.com/scim/v2/enterprises/{SUBDOMAIN}
```

---

## ✅ Pre-Flight Checklist

### Before Starting

- [ ] Enterprise slug/subdomain: `__________________`
- [ ] Setup user credentials stored securely
- [ ] Setup user 2FA enabled with recovery codes saved
- [ ] Entra admin with Application Administrator or higher role
- [ ] Azure Subscription ID (for billing): `__________________`
- [ ] Azure admin who can grant tenant-wide consent

### PAT Token Checklist

- [ ] Created as setup user (`SHORTCODE_admin`)
- [ ] Scope: `scim:enterprise`
- [ ] Expiration: No expiration
- [ ] Token stored securely

---

## 👥 Example Initial Group Model

| Group Name | Enterprise Role | Purpose |
|-----------|----------------|---------|
| **GitHub-Enterprise-Owners** | Enterprise Owner | Full admin access |
| **GitHub-Developers** | Member | Standard developer access |
| **GitHub-Security** | Member | Security champions |
| **GitHub-CI-Admins** | Member | CI/CD pipeline admins |

---

## 🎯 Success Criteria

After completing this guide, you should have:

- ✅ EMU enterprise fully configured with Entra ID authentication
- ✅ SCIM provisioning active for automated user lifecycle management
- ✅ Azure subscription connected for metered billing
- ✅ GitHub Copilot enabled and configured
- ✅ Initial organization structure established
- ✅ First users provisioned and able to access GitHub

---

## ❓ Common Questions & Troubleshooting

### Q: The setup user activation link does not work — what is wrong?
**A:** The most common cause is an email conflict. If the email address provided for the setup user is already associated as a primary email on another GitHub account, the activation link will silently fail. Find the existing GitHub account using that email, change its primary email to something else, and then retry the activation link. If the link has expired, contact GitHub Support to resend it.

---

### Q: I get a "user already exists" error when SCIM tries to provision a user — how do I fix this?
**A:** This occurs when the user's email or username already exists on GitHub (either as a personal account or in another enterprise). The conflicting account must change its primary email or username before SCIM can provision the managed user. Check the Entra ID provisioning logs for the specific attribute conflict and resolve it on the GitHub side.

---

### Q: SCIM provisioning in Entra ID shows a "quarantined" status — what does that mean?
**A:** Quarantine means Entra ID detected a high error rate during provisioning cycles. Common causes include an expired or invalid SCIM PAT, an incorrect Tenant URL, or rate limiting from assigning too many users at once. Check the provisioning logs in Entra for specific errors. Regenerate the SCIM token if needed, verify the Tenant URL, and restart provisioning. Entra will automatically exit quarantine after a successful cycle.

---

### Q: Should the SCIM attribute mapping use UPN or mail for the username?
**A:** For EMU, the `userName` SCIM attribute determines the GitHub username (with the shortcode suffix appended). Most organizations map `userPrincipalName` to `userName`. However, if your UPNs contain characters GitHub does not allow (such as `#` or `%`), or if UPNs differ significantly from email addresses, consider using `mail` instead. Test with a small pilot group before rolling out broadly.

---

### Q: My SAML Entity ID is correct, but authentication still fails — what should I check?
**A:** A common culprit is a trailing slash in the Entity ID. The Entity ID must be exactly `https://github.com/enterprises/YOUR_ENTERPRISE` with no trailing slash. Also verify the ACS URL and Sign-on URL match exactly. Check that the Base64 certificate you pasted does not contain extra whitespace or line breaks. Finally, confirm you are using the correct Entra Enterprise Application (the EMU gallery app, not the standard org app).

---

### Q: How do I rotate the SCIM token without breaking provisioning?
**A:** Sign in as the setup user (`SHORTCODE_admin`), generate a new PAT with the `scim:enterprise` scope, then immediately update the Secret Token field in the Entra ID provisioning configuration. Click Test Connection to confirm the new token works, then save. The old token is invalidated when you generate a new one, so update Entra promptly to minimize downtime.

---

### Q: The test user can sign in via SSO but lands on an empty enterprise with no orgs — why?
**A:** SSO authentication and SCIM provisioning are separate steps. The user must be both assigned in the Entra Enterprise Application (for SCIM) and assigned to an Entra group that is linked to a GitHub team within an organization. Verify the user appears in the enterprise People tab, and that they have been added to at least one organization via team membership.

---

### Q: The setup user is locked out of 2FA — how do I regain access?
**A:** Use the personal 2FA recovery codes that were saved when 2FA was first enabled on the setup user. If those are lost, use the enterprise SSO recovery codes to access the enterprise and contact GitHub Support to reset the setup user's 2FA. This is why securely storing both sets of recovery codes during initial setup is critical.

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
| 3 | Managing team memberships with IdP groups | [docs.github.com](https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/provisioning-user-accounts-with-scim/managing-team-memberships-with-identity-provider-groups) |
| 4 | Connecting an Azure subscription | [docs.github.com](https://docs.github.com/en/enterprise-cloud@latest/billing/managing-the-plan-for-your-github-account/connecting-an-azure-subscription) |
| 5 | Copilot policies for enterprise | [docs.github.com](https://docs.github.com/en/enterprise-cloud@latest/copilot/managing-copilot/managing-copilot-for-your-enterprise/managing-policies-and-features-for-copilot-in-your-enterprise) |
| 6 | Microsoft Entra SAML provisioning tutorial | [learn.microsoft.com](https://learn.microsoft.com/en-us/entra/identity/saas-apps/github-enterprise-managed-user-provisioning-tutorial) |

---

*Last updated: December 2025*