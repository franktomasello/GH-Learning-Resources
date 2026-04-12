# 🚀 GHEC EMU Enterprise Setup Runbook (OIDC)

> **Complete end-to-end guide for configuring EMU with Microsoft Entra ID (OIDC), Azure billing, and GitHub Copilot**

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **GitHub:** Sign in as `SHORT-CODE_admin` → Enterprise → Identity provider → Single sign-on → Enable OIDC configuration → Save (redirects to Entra)
- **Entra ID:** Sign in as Global Admin → Consent on behalf of organization → Accept (auto-creates the OIDC Enterprise App)
- **SCIM:** As `SHORT-CODE_admin`, generate PAT with `scim:enterprise` scope → Entra App → Provisioning → Automatic → Enter Tenant URL + token → Test Connection
- **Billing:** Enterprise Settings → Billing & Licensing → Payment information → Add Azure Subscription → Accept → Connect
- **Copilot:** Enterprise Settings → Policies → Copilot → Enable access → Org Settings → Copilot → Access → Assign seats

---

## 📋 Overview

**Scope:**

- **Identity:** Microsoft Entra ID (OIDC) for SSO
- **Provisioning:** SCIM (Automatic User Provisioning)
- **Billing:** Azure Subscription (Metered)
- **Copilot:** Business/Enterprise (Enable & Assign via Teams or Enterprise Teams)

---

## 0️⃣ Prerequisites (Critical)

### People & Permissions

| Role | Required Permissions |
|------|---------------------|
| **GitHub** | Access to the setup user (`@SHORT-CODE_admin`) email to set the initial password |
| **Microsoft Entra ID** | A **Global Administrator** to consent to the "GitHub Enterprise Managed User (OIDC)" app during SSO setup |
| **Azure Billing** | A user who can provide **tenant-wide admin consent** AND has **Owner** rights on the target Azure Subscription |

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

## 1️⃣ Create/Secure the EMU Setup User

*The setup user (`@SHORT-CODE_admin`) is the only local account that can bypass SSO in emergencies.*

### Steps

1. Open the "setup user invite" email in a **private/incognito browser window**
2. Set a strong password (store in secure vault)
3. **Immediately enable 2FA:**
   - Navigate to: Profile picture → **Settings** → **Password and authentication**
   - Configure a TOTP app (recommended) or other 2FA method
   - Download and securely store your **personal 2FA recovery codes**
4. Store credentials in a secure company vault (e.g., 1Password, LastPass, Azure Key Vault)

> ⚠️ **Critical (January 2025 Change):** All subsequent logins for the setup user require either a successful 2FA challenge OR use of an enterprise recovery code. If you do not save your enterprise recovery codes (generated in Step 3), you will be locked out.

> 📝 **Note:** If you ever need to reset the setup user password, you must contact [GitHub Support](https://support.github.com/). The standard email-based password reset does not work for setup users.

---

## 2️⃣ Create the SCIM Token

*This token allows Microsoft Entra ID to provision users to GitHub via SCIM.*

### Navigation (GitHub)

```
Profile Picture → Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token (classic)
```

### Token Configuration

| Setting | Value |
|---------|-------|
| **Note** | `SCIM Token for Entra ID` (or similar) |
| **Expiration** | **No expiration** ⚠️ If this expires, provisioning stops entirely |
| **Scopes** | Select **only** `scim:enterprise` |

1. Click **Generate token**
2. **Copy the token immediately** — you cannot view it again

> 📝 **Store this token securely.** You will need it when configuring provisioning in Microsoft Entra ID (Step 4).

---

## 3️⃣ Enable Microsoft Entra ID OIDC SSO

*Connects identity so users can authenticate via your IdP.*

### Navigation (GitHub)

```
Profile Picture → Your enterprises → Select enterprise → Identity provider tab → Single sign-on configuration
```

### Steps

1. Under "OIDC single sign-on", select **Enable OIDC configuration**
2. Click **Save** — you will be redirected to Microsoft

### In Microsoft Entra ID (during redirect)

3. Sign in as a user with **Global Administrator** rights
4. Review the permissions requested for "GitHub Enterprise Managed Users with OIDC"
5. ✅ Check **"Consent on behalf of your organization"**
6. Click **Accept**

### Back in GitHub

7. **Save your Enterprise Recovery Codes:**
   - Click **Download**, **Print**, or **Copy**
   - Store these securely — they are separate from your personal 2FA codes
   - These codes allow emergency access if your IdP becomes unavailable
8. Click **Enable OIDC Authentication**

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

1. Go to [entra.microsoft.com](https://entra.microsoft.com/) → **Enterprise applications**
2. Select **GitHub Enterprise Managed User (OIDC)** (created automatically during Step 3)
3. Click the **Provisioning** tab → **Get started**
4. Set **Provisioning Mode** to **Automatic**
5. Under **Admin Credentials**:

| Field | Value |
|-------|-------|
| **Tenant URL** | Your URL from Step 4A |
| **Secret Token** | The PAT created in Step 2 |

6. Click **Test Connection** — must show success ✅
7. Click **Save**

### 4C) Configure Provisioning Settings

| Setting | Value |
|---------|-------|
| **Scope** | Sync only assigned users and groups |
| **Provisioning Status** | **On** |

Click **Save**

### 4D) Assign Users and Groups

1. Go to **Users and groups** tab in the Entra application
2. Click **Add user/group**
3. Add a test user or pilot group
4. For users who need Enterprise Owner role, set the **Role** attribute accordingly

### 4E) Initial Sync

| Method | Timing |
|--------|--------|
| **Automatic** | Wait ~40 minutes for the initial sync cycle |
| **Manual testing** | Use **Provision on demand** to test individual users immediately |

> ⚠️ **Important:** The combination of Okta and Entra ID for SSO and SCIM (in either order) is explicitly **not supported**.

---

## 5️⃣ Create Organization(s) Inside the Enterprise

*EMU enterprises cannot invite or import existing organizations — you must create them fresh.*

### Navigation

```
Profile Picture → Your enterprises → Select enterprise → Organizations tab → New organization
```

### Steps

1. Enter organization name (e.g., `acme-engineering`, `acme-platform`)
2. Click **Create organization**

> 💡 **Best Practice:** Plan your organization structure before creating. Consider separating by business unit, product line, or team function.

---

## 6️⃣ Connect Entra ID Groups to GitHub Teams

*This is the "Golden Path" for automated permissions management.*

### Navigation

```
Profile Picture → Your organizations → Select organization → Teams tab → New team
```

### Configuration

| Field | Value |
|-------|-------|
| **Team name** | e.g., `developers`, `platform-engineers` |
| **Description** | Optional but recommended |
| **Identity Provider Groups** | Select the synced Entra group from dropdown |

Click **Create team**

### Important Constraints

> ⚠️ **No Nested Teams:** When a team is linked to an IdP group, you cannot nest it under other teams.

> ⚠️ **Nested Groups in Entra:** Microsoft Entra ID does not support provisioning nested groups. Only direct members sync.

---

## 7️⃣ Connect Azure Subscription (Billing)

*Required for metered billing: Copilot, Actions minutes, Packages storage, GHAS, and Codespaces.*

### Prerequisites Check

- [ ] Tenant-wide admin consent capability in Azure
- [ ] Owner permissions on the target Azure Subscription

### Navigation

```
Enterprise Settings → Billing & Licensing → Payment information → Metered billing via Azure → Add Azure Subscription
```

### Azure Connection Flow

1. Sign in with your Microsoft account
2. Review the **"Permissions requested"** prompt
3. Click **Accept**
4. Under **"Select a subscription"**, choose the Azure Subscription ID
5. Check the confirmation box
6. Click **Connect**

> ⚠️ **Troubleshooting:** If you see "You need admin approval" instead of the permissions prompt, work with your Azure AD Global Administrator to grant consent or configure an [admin consent workflow](https://learn.microsoft.com/en-us/azure/active-directory/manage-apps/configure-admin-consent-workflow).

### Verification

- ✅ Azure Subscription ID appears under "Payment information"
- ✅ You can now enable metered services like Copilot and GHAS

---

## 8️⃣ Enable Copilot & Assign Seats

*Two methods available: Organization Teams (traditional) or Enterprise Teams (public preview)*

### Phase A: Enable Copilot at Enterprise Level

**Navigation:**
```
Enterprise Settings → Policies → Copilot
```

**Configuration:**

| Policy | Recommendation |
|--------|----------------|
| **Access to Copilot** | Enabled (or allowed for specific organizations) |
| **Suggestions matching public code** | Consult legal counsel |
| **Copilot Chat** | Enable for full functionality |
| **Copilot in the CLI** | Enable as needed |

Click **Save**

> ⚠️ **Important:** If this policy is not enabled, you cannot assign Copilot seats at the organization level.

### Phase B: Assign Seats

#### Option 1: Via Organization Teams (Traditional Method)

**Navigation:**
```
Organization → Settings → Copilot → Access
```

1. Click **Allow this organization to assign seats**
2. Click **Start adding seats**
3. Select **Purchase for selected members**
4. Select the team created in Step 6
5. Click **Continue to purchase** → **Purchase seats**

#### Option 2: Via Enterprise Teams (Public Preview)

*Allows assigning Copilot licenses at the enterprise level, without requiring organization membership.*

**Navigation:**
```
Enterprise Settings → People → Enterprise teams → [Create/Select team]
Enterprise Settings → Billing & Licensing → Licensing → Copilot Business → Add seats → Add enterprise teams
```

> 💡 **When to use Enterprise Teams:**
> - Users who need Copilot but not full GitHub Enterprise Cloud licenses
> - Simplified management for large enterprises
> - Direct IdP group synchronization at enterprise level

### Result (Zero-Touch Provisioning)

When configured correctly:
1. User is added to Entra ID group
2. SCIM automatically provisions their GitHub account
3. User is automatically added to the linked GitHub Team
4. Copilot license is automatically assigned via Team membership
5. User can immediately use Copilot in their IDE

---

## 9️⃣ Validation Checklist

Run through these checks to confirm successful setup:

| Check | How to Verify | Expected Result |
|-------|---------------|------------------|
| **OIDC SSO** | Have a provisioned user attempt to sign in | Redirects to Entra ID, successfully authenticates |
| **SCIM Provisioning** | Check Enterprise → People tab | Test users appear with `_shortcode` suffix |
| **Group Sync** | Check Organization → Teams | IdP group members appear in linked team |
| **Azure Billing** | Enterprise → Billing & Licensing → Payment information | Azure Subscription ID displayed |
| **Copilot Access** | User opens VS Code with GitHub Copilot extension | Copilot icon active, suggestions working |

### Troubleshooting Quick Reference

| Issue | Common Cause | Solution |
|-------|--------------|----------|
| Users not provisioning | SCIM token expired or invalid | Regenerate PAT with `scim:enterprise` scope |
| SSO redirect fails | Entra app misconfigured | Verify OIDC app settings in Entra admin center |
| "Admin approval required" for Azure | Insufficient Azure AD permissions | Request tenant-wide admin consent |
| Copilot not activating | Policy not enabled at enterprise level | Enable in Enterprise → Policies → Copilot |
| Team membership not syncing | Nested groups in Entra | Flatten group structure or add users directly |

---

## ❓ Common Questions & Troubleshooting

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
| 9 | Enterprise Teams (Sept 2025) | [github.blog](https://github.blog/changelog/2025-09-04-manage-copilot-and-users-via-enterprise-teams-in-public-preview/) |
| 10 | Downloading enterprise recovery codes | [docs.github.com](https://docs.github.com/enterprise-cloud@latest/admin/managing-iam/managing-recovery-codes-for-your-enterprise/downloading-your-enterprise-accounts-single-sign-on-recovery-codes) |

---

## 📝 Revision History

| Date | Version | Changes |
|------|---------|----------|
| December 2025 | 2.0 | Verified against current documentation; updated OIDC navigation path; clarified Azure permissions; added Enterprise Teams option for Copilot; fixed source references |

---

*Last updated: December 2025*