# 🚀 GitHub Standard GHEC + Okta (SAML), Azure Billing & Copilot Setup Runbook

> **Complete end-to-end runbook for configuring Standard (non-EMU) GHEC with Okta (SAML), SCIM org provisioning, Azure billing, and GitHub Copilot**

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Okta:** Applications → Browse App Catalog → "GitHub Enterprise Cloud - Organization" → Add Integration → Set org name → Sign On tab → Capture SSO URL, Issuer, X.509 cert
- **GitHub:** Org Settings → Authentication security → SAML single sign-on → Paste Okta values → Test → Save → Enforce SAML SSO
- **SCIM:** Sign in as setup user → Visit `/orgs/YOUR_ORG/sso` → Okta App → Provisioning → Enable API Integration → Authenticate with GitHub
- **Billing:** Org/Enterprise → Billing and licensing → Payment information → Add Azure Subscription → Accept → Connect
- **Copilot:** Enterprise AI controls → Copilot → Enable → Org Settings → Copilot → Access → Assign seats

---

## ✅ Accuracy & Click-Path Notes

- Reviewed against current public GitHub and Microsoft documentation in April 2026 where public documentation is available. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
- When a path starts with `Enterprise`, begin at GitHub, click your profile photo, click `Your enterprises` or `Enterprise`, select the enterprise, then continue with the listed top tab or left-sidebar item.
- When a path starts with `Organization` or `Org`, begin at GitHub, click your profile photo, click `Your organizations`, select the organization, click `Settings`, then continue with the listed sidebar item.
- When a path starts with `Repository`, `Repo`, or a repository name, open the repository, click the `Settings` tab, then continue with the listed sidebar item.
- When a path starts with a vendor portal such as `Microsoft Entra admin center`, `Azure portal`, `Okta Admin Console`, `PingFederate`, `PingOne`, `OneLogin`, `AD FS Management`, `Visual Studio Admin Portal`, or `Azure DevOps`, sign in to that admin portal first, select the tenant, application, or project named in the step, then follow each listed blade, tab, button, and confirmation in order.
- If the expected button is missing, verify you are signed in with the role named in Prerequisites, the feature or license is enabled, and the object is owned by the selected enterprise, organization, or repository. Use page search only to locate the same page, not to skip required confirmation, test, save, or consent clicks.

---

## 📋 Overview

This guide walks through setting up **Standard (non-EMU) GitHub Enterprise Cloud (GHEC)**, including:

- **Okta (SAML SSO)** for authentication
- **Enforce SAML SSO** for the organization (and enterprise, if applicable)
- **SCIM provisioning** (Okta → GitHub org membership lifecycle)
- **Azure subscription** attachment for metered billing
- **GitHub Copilot** enablement + policy controls
- **Critical post-enablement** items (PAT/SSH SSO authorization)

### Standard (non-EMU) means:

- Users keep their regular GitHub.com accounts
- SAML SSO is enforced at the org (and optionally at the enterprise, if you have one)
- SCIM manages organization membership via Okta assignments (not "managed user accounts"—that is EMU)

## ✅ Prerequisites

| Requirement | Owner / Role | Notes |
| --- | --- | --- |
| GitHub Enterprise Cloud (Standard / non-EMU) org | Org Owner | You must be able to access Org Settings → Authentication security |
| (If applicable) GitHub Enterprise account | Enterprise Owner | Needed only if you will configure Enterprise SAML and/or Enterprise Copilot policies |
| Okta Admin access | Okta Admin | Must be able to install and configure the Okta catalog app |
| Pilot users/groups in Okta | IAM team | Use a small pilot cohort first to reduce risk |
| Dedicated GitHub "SCIM setup user" | Org Owner | GitHub recommends a dedicated user to authorize SCIM OAuth in Okta |
| Azure subscription + ability to consent | Azure admin | Needed to connect metered billing via Azure. If subscription is in a different tenant, you may need to specify a different tenant ID during connection |
| Copilot plan decision | Enterprise/Org owner | Copilot Business vs Copilot Enterprise |

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **Okta application admin** | Creates the GitHub Enterprise Cloud - Organization app, configures SAML, and enables SCIM OAuth provisioning. | Okta Admin Console → Applications → Applications → Browse App Catalog → search GitHub Enterprise Cloud - Organization → Add Integration → enter organization name and app label → Done → Assignments → Assign setup/pilot user → Sign On → View Setup Instructions. After GitHub SAML is enabled, go to Provisioning → Configure API Integration → Enable API integration → Authenticate with GitHub Enterprise Cloud - Organization → Grant → Authorize OktaOAN → Save → To App → Edit → enable Create Users, Update User Attributes, and Deactivate Users → Save. Handoff: setup instructions, OAuth authorization, and provisioning status. |
| **GitHub organization owner or dedicated SCIM setup user** | Enables org SAML and authorizes Okta SCIM on behalf of the organization. | GitHub → profile photo → Your organizations → [org] → Settings → Authentication security → SAML single sign-on → Enable SAML authentication → paste Okta Sign on URL, Issuer, and Public Certificate → Test SAML configuration → Save → visit `https://github.com/orgs/ORG/sso` → complete SSO → return to Okta OAuth flow → Grant → Authorize OktaOAN. Handoff: SAML test success, recovery codes, and Okta authorization. |
| **Okta group owner** | Maintains the users or groups assigned to the GitHub organization app. | Okta Admin Console → Directory → Groups → [group] → People → Assign people → select users → Save, then Applications → Applications → [GitHub org app] → Assignments → Assign → Assign to Groups → select group → Done. Handoff: pilot group and users assigned. |
| **GitHub enterprise or organization owner** | Starts the Azure metered billing connection from GitHub. | Enterprise path: GitHub → profile photo → Your enterprises → [enterprise] → Billing and licensing → Payment information → Metered billing via Azure → Add Azure Subscription. Organization path: GitHub → profile photo → Your organizations → [organization] → Settings → Billing and licensing → Payment information → Metered billing via Azure → Add Azure Subscription. Then sign in to Microsoft → Permissions requested → Accept → Select a subscription → Connect. Handoff: the subscription ID is visible on Payment information. |
| **Azure subscription Owner** | Provides the Azure subscription that GitHub will bill against, or grants another signer the required Azure RBAC rights. | Azure portal → Subscriptions → [subscription] → Access control (IAM) → Role assignments → confirm the signer is listed under Owner. To grant access: Add → Add role assignment → Privileged administrator roles → Owner → Members → Select members → [user] → Select → Review + assign. Handoff: subscription ID and tenant ID. |
| **Microsoft Entra Global Administrator or consent approver** | Approves tenant-wide consent when the Microsoft consent prompt blocks the GitHub billing app. | Microsoft Entra admin center → Entra ID → Enterprise apps → Activity → Admin consent requests → My Pending → [GitHub request] → Review permissions and consent → Approve. If the Global Administrator completes the GitHub flow directly, approve the Permissions requested prompt by clicking Accept. |

---

## 1️⃣ Create & Secure the "SCIM Setup User" (Standard non-EMU)

> ⚠️ **Important:** This user is required because Okta's GitHub SCIM provisioning uses a third-party OAuth authorization that acts on behalf of a specific GitHub user. If that user loses access or leaves the org, SCIM can stop working—so you want a stable, dedicated identity.

### Process

1. Create a dedicated GitHub.com account (example: `gh-okta-scim@yourdomain.com`)
2. **Secure the account:**
    - Enable 2FA
    - Ensure recovery methods are stored per your approved process (vault / break-glass)
3. **Add the setup user to your GitHub organization as an Owner:**
    - GitHub → your Organization → Settings → People → Invite member
    - After accepted → set role to Owner
4. **Assign this same identity in Okta** to the GitHub app (so it can SSO and authorize provisioning)

### Important Notes

> 📌 **Note:** This setup user will consume a GitHub license. Treat it as a system account: minimal use outside of IAM configuration.

## 2️⃣ Create the Okta Application (GitHub Enterprise Cloud – Organization)

### Navigation (Okta)

```
Okta Admin Console
  → Applications
    → Applications
      → Browse App Catalog
        → Search: "GitHub Enterprise Cloud - Organization"
          → Add Integration

```

### Configuration Steps

1. **Select the Okta catalog app:** `GitHub Enterprise Cloud - Organization`
    - ⚠️ Do NOT use "GitHub Enterprise Managed User" (that is for EMU)
2. **Fill in required fields:**
    - Organization name (GitHub org slug)
    - Application label (unique name in Okta)
3. **Assign the app to:**
    - The SCIM setup user
    - Your pilot user/group

### Download Required Items (Okta IdP values)

In the Okta app:

1. Go to **Sign On**
2. Under **SIGN ON METHODS**, click **View Setup Instructions**
3. Capture:
    - **Sign on URL**
    - **Issuer**
    - **X.509 certificate** (public cert)

## 3️⃣ Enable SAML SSO in GitHub

You will configure **Organization SAML** (always for the target org).

If your org is under an Enterprise account, you may also configure **Enterprise SAML**.

> ⚠️ **Warning:** Enabling SAML impacts how members authenticate. Ensure you have recovery codes stored for break-glass access.

### 3A — (Conditionally Required) Configure Enterprise SAML

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
2. Enable / configure SAML per GitHub's enterprise SAML documentation (Okta values differ from org app)

### 3B — Enable & Test Organization SAML (Required)

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
2. Populate fields using Okta values from Step 2:
    - **Sign on URL** (Okta)
    - **Issuer** (Okta) — _Note: GitHub indicates Issuer is required for some features like team synchronization_
    - **Public Certificate** (Okta X.509)
3. Click **Test SAML configuration** (or equivalent prompt) and complete the auth flow
4. Click **Save**
5. **Immediately download and secure SSO recovery codes:**
    - Organization Settings → Authentication security → Single sign-on recovery codes

> 🔐 **Critical:** Before enabling or immediately after enabling SAML, download and securely store your organization SSO recovery codes. These are essential for break-glass scenarios if your IdP becomes unavailable.

## 4️⃣ Enforce SAML SSO for the Organization (Required)

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

## 5️⃣ Configure SCIM Provisioning (Okta → GitHub Organization)

> 💡 **Tip:** In Standard non-EMU, the Okta integration uses a third-party OAuth flow for SCIM. The OAuth app acts on behalf of the GitHub user who authorized it (hence Step 1).

### 5A — Create an active SAML session for the setup user (Required)

1. Sign into GitHub as the SCIM setup user
2. Create an active org SAML session by visiting:

```
https://github.com/orgs/ORGANIZATION-NAME/sso

```
3. Complete the SSO authentication prompt successfully

### 5B — Enable Okta API Integration (Required)

**Navigation (Okta)**

```
Okta Admin Console
  → Applications
    → Applications
      → [Your "GitHub Enterprise Cloud - Organization" app]
        → Provisioning

```

**Steps**

1. Click **Configure API Integration**
2. Check **Enable API integration**
3. Click **Authenticate with GitHub Enterprise Cloud - Organization**
4. Complete the GitHub OAuth authorization as the SCIM setup user

> 📌 **Note:** Okta may show an **Import Groups** option. GitHub's docs state this is not supported; selecting/deselecting does not change behavior.

### 5C — Configure Provisioning Settings + Mappings (Required)

1. In the Okta app → **Provisioning**
2. Enable the desired provisioning actions (Create/Update/Deactivate users) as appropriate for your org membership lifecycle
3. Keep mappings aligned to GitHub's Okta SCIM guidance; avoid custom attributes until the base flow is stable

### 5D — Assign Users/Groups (Required)

**Navigation Path:**

```
Okta Admin Console
  → Applications
    → Applications
      → [Your "GitHub Enterprise Cloud - Organization" app]
        → Assignments
          → Assign
            → Assign to People / Assign to Groups
              → (Select users/groups)
                → Assign
                  → Done

```

**Steps**

1. In Okta → the GitHub app → **Assignments**
2. Assign:
    - Pilot group(s) first
3. **Validate lifecycle:**
    - Add assignment → user becomes org member
    - Remove assignment → user is removed from org (per your provisioning settings)

## 6️⃣ Attach Azure Subscription for Metered Billing

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
        → Billing and licensing
          → Payment information
            → Metered billing via Azure
              → Add Azure Subscription

```

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

### 7A — (Conditionally Required) Enable Copilot at the Enterprise level via Payment Verification

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

### 7B — Configure Copilot Enterprise Policies

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

### 7C — Configure Copilot Organization Policies (Required)

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

### 7D — Assign Copilot licenses (Required)

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

## 8️⃣ Critical Post-Enablement: SSO Authorization for Credentials (Required)

When SAML is enabled/enforced, users often must authorize credentials (depending on token type and whether they have a linked external identity).

### 8A — Authorize SSH Keys for SSO

**Required for SSH usage in SSO orgs**

**Navigation**

```
GitHub (top-right profile picture)
  → Settings
    → SSH and GPG keys
      → (next to the key) Configure SSO
        → Authorize (for the org)

```

### 8B — Authorize Personal Access Tokens

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
- Okta admin with privileges to create/configure app integrations: ☐
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
- [ ] Okta provisioning "Authenticate with GitHub…" completed
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

- ✅ Standard GHEC organization fully configured with Okta SAML authentication
- ✅ SAML SSO enforced for the organization
- ✅ SCIM provisioning active for automated org membership lifecycle management
- ✅ Azure subscription connected for metered billing (if applicable)
- ✅ GitHub Copilot enabled and configured
- ✅ Users have authorized SSH keys and PATs for SSO access
- ✅ Pilot users provisioned and able to access GitHub via SSO

---
## 🧯 Known Errors & Resolutions

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

---

## ❓ Common Questions & Troubleshooting

### Q: I installed the wrong Okta catalog app — how do I tell EMU vs standard org?
**A:** The correct app for standard (non-EMU) GHEC is **"GitHub Enterprise Cloud - Organization"**. The EMU app is named **"GitHub Enterprise Managed User"**. If you installed the EMU app, SCIM and SAML will target the enterprise endpoint instead of the org endpoint, and provisioning will fail. Delete the incorrect app in Okta and install "GitHub Enterprise Cloud - Organization" from the catalog.

---

### Q: The OAuth authorization for SCIM provisioning keeps failing — what is going wrong?
**A:** Okta's standard org SCIM integration uses a third-party OAuth flow (not a PAT). The setup user must: (1) be an org owner, (2) have an active SAML session for the org (visit `https://github.com/orgs/YOUR_ORG/sso` first), and (3) complete the OAuth authorization as that user in the Okta Provisioning tab. If the authorization popup fails silently, try in an incognito window, ensure pop-ups are not blocked, and confirm the setup user can access the org.

---

### Q: SAML enforcement removed bots and service accounts from the org — how do I avoid this?
**A:** Before enforcing SAML, review the list of members who have not authenticated via the IdP. Bots and service accounts without external identities will be removed. To prevent this: (1) assign IdP identities to service accounts and have them complete SSO before enforcement, or (2) switch automation to use GitHub Apps, which are not affected by SAML enforcement. After enforcement, removed accounts can rejoin within three months with their previous access restored.

---

### Q: We forgot to save our SSO recovery codes before enforcement — what now?
**A:** If SAML is already enforced and working, you can still access the recovery codes. Navigate to Organization Settings > Authentication security > Single sign-on recovery codes. Download and store them immediately. If your IdP goes down and you do not have recovery codes, you will need to contact GitHub Support for assistance, which can take time. Always store recovery codes in a secure vault accessible to at least two org owners.

---

### Q: Users cannot push after enforcement — they get "Resource protected by organization SAML enforcement" errors. What is the fix?
**A:** After SAML enforcement, PAT classic tokens must be individually SSO-authorized for the org. Each user must go to Settings > Developer settings > Personal access tokens, click "Configure SSO" next to the token, and authorize it for the org. SSH keys also need SSO authorization under Settings > SSH and GPG keys > Configure SSO. Fine-grained PATs are authorized during creation and do not require this extra step.

---

### Q: Okta provisioning is working, but some users appear as "outside collaborator" instead of "member" — why?
**A:** This can happen if users were manually invited as outside collaborators before SCIM provisioning took over. SCIM does not automatically upgrade outside collaborators to members. Remove the user's outside collaborator role manually, then let Okta re-provision them as a member. Also verify the Okta app role mapping is set to "Member" (not "Outside Collaborator") for the affected users.

---

### Q: We need to test SAML before enforcing it for everyone — how?
**A:** After enabling SAML (but before enforcing), SAML is optional — users can still access the org without SSO. Use the "Test SAML configuration" button in GitHub org settings to validate the flow. Assign a pilot group in Okta and have them authenticate via `https://github.com/orgs/YOUR_ORG/sso`. Once the pilot confirms everything works, proceed to enforcement. This test phase lets you catch configuration issues without locking anyone out.

---

### Q: Can we use the same Okta app for multiple GitHub organizations?
**A:** No. Each GitHub organization requires its own Okta catalog app instance because the SAML Entity ID and SCIM endpoint are org-specific. Install a separate "GitHub Enterprise Cloud - Organization" app in Okta for each org you need to configure. Each app will have its own SAML settings, provisioning configuration, and user assignments.

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
- [Configuring SAML SSO and SCIM using Okta - GitHub Docs](https://docs.github.com/en/organizations/managing-saml-single-sign-on-for-your-organization/configuring-saml-single-sign-on-and-scim-using-okta)
- [About SCIM for organizations - GitHub Docs](https://docs.github.com/en/organizations/managing-saml-single-sign-on-for-your-organization/about-scim-for-organizations)

---

*Last updated: April 2026*
