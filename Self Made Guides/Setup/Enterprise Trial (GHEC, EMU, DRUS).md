# 🧪 GitHub Enterprise Trial (GHEC, EMU & DRUS) Setup Runbook

> **Complete guide to starting a GitHub Enterprise Cloud trial — Standard GHEC, EMU (managed users), or Data Residency US (DRUS)**

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [📋 Overview](#-overview)
- [1️⃣ Start the Trial](#1-start-the-trial)
- [2️⃣ Complete the Setup Email](#2-complete-the-setup-email)
- [3️⃣ Configure Identity Provider (EMU & DRUS Only)](#3-configure-identity-provider-emu--drus-only)
- [4️⃣ Configure SAML SSO (Standard GHEC Only)](#4-configure-saml-sso-standard-ghec-only)
- [5️⃣ Request Add-On Trials](#5-request-add-on-trials)
- [6️⃣ Trial Duration & Extensions](#6-trial-duration--extensions)
- [🚀 Tips for a Successful Trial](#-tips-for-a-successful-trial)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Start trial:** Browse to `github.com/enterprise/trial` → Sign in → Choose trial type (Standard GHEC / EMU / DRUS) → Enter enterprise name → Create enterprise
- **Activate:** Open setup email within 7 days → Click activation link → Complete on-screen prompts
- **IdP (EMU/DRUS only):** Enterprise → Settings → Authentication security → Configure SAML/OIDC + SCIM before inviting users
- **Add-ons:** Contact your GitHub SE/CSM to request Copilot Business (50 seats) or GHAS trial add-ons
- **Extend:** Request extension before day 25 → Contact GitHub SE/CSM/Sales

---

## ✅ Accuracy & Click-Path Notes

<details>
<summary><em>Show click-path conventions</em></summary>


- Reviewed against current public GitHub and Microsoft documentation in April 2026 where public documentation is available. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
- When a path starts with `Enterprise`, begin at GitHub, click your profile photo, click `Your enterprises` or `Enterprise`, select the enterprise, then continue with the listed top tab or left-sidebar item.
- When a path starts with `Organization` or `Org`, begin at GitHub, click your profile photo, click `Your organizations`, select the organization, click `Settings`, then continue with the listed sidebar item.
- When a path starts with `Repository`, `Repo`, or a repository name, open the repository, click the `Settings` tab, then continue with the listed sidebar item.
- When a path starts with a vendor portal such as `Microsoft Entra admin center`, `Azure portal`, `Okta Admin Console`, `PingFederate`, `PingOne`, `OneLogin`, `AD FS Management`, `Visual Studio Admin Portal`, or `Azure DevOps`, sign in to that admin portal first, select the tenant, application, or project named in the step, then follow each listed blade, tab, button, and confirmation in order.
- If the expected button is missing, verify you are signed in with the role named in Prerequisites, the feature or license is enabled, and the object is owned by the selected enterprise, organization, or repository. Use page search only to locate the same page, not to skip required confirmation, test, save, or consent clicks.

</details>

---

## ✅ Prerequisites

| Requirement | Status |
|-------------|--------|
| Existing GitHub.com account (to initiate the trial) | ☐ |
| Enterprise name (slug) decided — globally unique, cannot be changed | ☐ |
| Identity model chosen: Standard GHEC, EMU, or DRUS | ☐ |
| IdP admin access ready (EMU/DRUS: Entra ID, Okta, or PingFederate) | ☐ |
| GitHub SE/CSM contact identified (for add-on trials and extensions) | ☐ |

---

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **GitHub trial requester or enterprise owner** | Starts the trial and decides which identity model will be tested. | GitHub Enterprise trial page or GitHub sales-provided setup link → enter organization and enterprise details → select Standard GHEC, EMU, DRUS/GHE.com as applicable → complete setup email → GitHub → profile photo → Your enterprises → [trial enterprise]. Handoff: enterprise URL, trial type, and setup user invitation. |
| **Microsoft Entra, Okta, or PingFederate admin** | Completes the provider-side app setup for EMU or DRUS trials. | Entra: Microsoft Entra admin center → Entra ID → Enterprise apps → New application → GitHub Enterprise Managed User or GitHub Enterprise Cloud - Organization → Single sign-on and Provisioning. Okta: Okta Admin Console → Applications → Browse App Catalog → GitHub Enterprise Managed User or GitHub Enterprise Cloud - Organization → Sign On and Provisioning. PingFederate: Administrative Console → Applications → SP Connections → GitHub EMU Connector or GitHub SAML SP connection → Browser SSO and Outbound Provisioning. Handoff: SSO values, SCIM status, and pilot group. |
| **Azure subscription Owner and Microsoft Entra consent approver, if testing Azure billing** | Provides the subscription and consent needed for metered billing. | Azure portal → Subscriptions → [subscription] → Access control (IAM) → confirm Owner. Enterprise path: GitHub → profile photo → Your enterprises → [enterprise] → Billing and licensing → Payment information → Metered billing via Azure → Add Azure Subscription. Organization path: GitHub → profile photo → Your organizations → [organization] → Settings → Billing and licensing → Payment information → Metered billing via Azure → Add Azure Subscription. Then Microsoft sign-in → Permissions requested → Accept → Select subscription → Connect. Handoff: connected subscription ID. |

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

### Navigate to the Trial Page

**Navigation:**

```
Browser → https://github.com/enterprise/trial
```

**Steps:**

1. Click **Start a free trial**
2. Sign in with an existing GitHub account (or create one)
3. Choose your trial type:

| Option | What You Get |
|--------|-------------|
| **GitHub Enterprise Cloud** | Standard GHEC enterprise with personal accounts |
| **Enterprise Managed Users** | EMU enterprise — all accounts provisioned via IdP |
| **Enterprise Managed Users with Data Residency** | DRUS — EMU with US data residency |

4. Enter your **enterprise name** (this becomes your enterprise slug)
5. Click **Create enterprise**

> ✅ **Result:** A setup email is sent to the email address associated with your GitHub account.

---

## 2️⃣ Complete the Setup Email

> ⚠️ **Important:** The setup link in the email expires after **7 days**. If it expires, you must restart the trial process.

**Steps:**

1. Open the setup email from GitHub
2. Click the activation link
3. Follow the on-screen prompts to finalize enterprise creation

> ✅ **Result:** Your enterprise is created and you are the enterprise owner.

---

## 3️⃣ Configure Identity Provider (EMU & DRUS Only)

> ⚠️ **Prerequisite:** For EMU and DRUS trials, you **must** configure SCIM provisioning and SAML/OIDC in your IdP **before** inviting any users.

### A) Configure SAML/OIDC Single Sign-On

**Navigation:**

```
Enterprise → Settings → Authentication security
  → SAML single sign-on → Configure
```

**Steps:**

1. Enter your IdP's **Sign on URL**, **Issuer**, and **Public certificate**
2. Test the SAML configuration
3. Enable SAML SSO

### B) Configure SCIM Provisioning

**Steps:**

1. In your IdP (Entra ID, Okta, etc.), configure the GitHub EMU SCIM application
2. Map required user attributes (username, email, display name)
3. Assign users and groups in the IdP
4. Start provisioning — users are created automatically in GitHub

> 💡 **Tip:** Do not manually create user accounts in an EMU enterprise. All user lifecycle management flows through SCIM.

---

## 4️⃣ Configure SAML SSO (Standard GHEC Only)

*For standard GHEC, SAML SSO can be configured at the organization level or the enterprise level. Enterprise-level SAML is also supported and, when configured, overrides any org-level SAML settings.*

**Navigation (org-level):**

```
Organization → Settings → Authentication security
  → SAML single sign-on → Enable SAML authentication
```

**Navigation (enterprise-level — recommended):**

```
Enterprise → Settings → Authentication security
  → SAML single sign-on → Enable SAML authentication
```

**Steps:**

1. Enter your IdP's **Sign on URL**, **Issuer**, and **Public certificate**
2. Test the SAML configuration
3. Click **Enable SAML authentication**
4. (Optional) Check **Require SAML SSO authentication** to enforce for all members

> ✅ **Result:** Organization members must authenticate via your IdP to access org resources.

---

## 5️⃣ Request Add-On Trials

*Enhance your trial with additional products to evaluate the full platform.*

### A) Copilot Business Trial

**Steps:**

1. Contact your GitHub Sales representative or Solutions Engineer
2. Request a Copilot Business trial (up to **50 seats**)
3. Once activated, assign seats:

**Navigation:**

```
Enterprise → Settings → Copilot → Access management
  → Enable for specific organizations → Assign seats
```

### B) GitHub Advanced Security (GHAS) Trial

**Steps:**

1. Contact your GitHub Sales representative or Solutions Engineer
2. Request a GHAS trial add-on
3. Once activated, enable security features:

**Navigation:**

```
Organization → Settings → Advanced Security → Configurations
  → Apply GitHub recommended configuration
```

> 💡 **Tip:** Request add-on trials early in your evaluation period so you have maximum time to test.

---

## 6️⃣ Trial Duration & Extensions

| Detail | Value |
|--------|-------|
| **Default trial duration** | 30 days |
| **Extension available** | Up to 60-90 days (upon request) |
| **Extension request deadline** | Before day 25 of the trial |
| **How to request** | Contact your GitHub SE, CSM, or Sales representative |

> ⚠️ **Important:** Request your extension **before day 25**. Extensions requested after the trial expires may not be honored.

---

## 🚀 Tips for a Successful Trial

1. **Define success criteria** before starting — what does "yes, we buy" look like?
2. **Schedule a kickoff call** with your GitHub SE/CSM within the first week
3. **Request trial extensions early** — do not wait until the last day
4. **Invite a small pilot group** first to validate your IdP integration before broad rollout
5. **Document your configuration decisions** — they carry over if you convert to a paid plan

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
| **Trial ends sooner than expected after billing setup** | Some billing changes can transition the account from trial behavior to paid usage. | Review the billing warning before connecting payment and confirm with GitHub account team if you need the trial to remain active. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>


### Q: My setup link expired — it has been more than 7 days. How do I get a new one?
**A:** The activation link in the setup email expires after 7 days. If it expires, you must restart the trial process by going to `https://github.com/enterprise/trial` and creating a new trial. If the enterprise name (slug) you wanted is now taken by the expired trial, contact GitHub Support or your GitHub Sales representative to have the expired trial removed so the namespace is freed up.

---

### Q: Which trial type should I choose — Standard GHEC, EMU, or DRUS?
**A:** Choose **Standard GHEC** if your developers already have GitHub.com accounts and you want a low-friction evaluation with optional SAML SSO. Choose **EMU** if your organization requires full identity governance (all accounts provisioned and controlled by your IdP). Choose **DRUS** if you have a formal US data residency requirement in addition to EMU. If unsure, start with Standard GHEC — it is the simplest to set up and evaluate. You cannot convert between trial types, so choose based on your production intent.

---

### Q: Can I extend the trial, and who do I contact?
**A:** Yes, trials can be extended from 30 days to 60-90 days. Contact your GitHub Solutions Engineer (SE), Customer Success Manager (CSM), or Sales representative to request an extension. Submit the request before day 25 of your trial — requests made after the trial expires may not be honored. Include a brief explanation of what you still need to evaluate.

---

### Q: The enterprise namespace I want is already taken — what can I do?
**A:** Enterprise slugs are globally unique across all of GitHub. If the name you want is taken by another customer, you must choose a different slug. If it was taken by a previous trial you created that expired, contact GitHub Support to have it released. Common workarounds include appending a suffix (e.g., `contoso-corp` instead of `contoso`). The enterprise slug is permanent and cannot be changed after creation.

---

### Q: I requested a Copilot or GHAS add-on trial, but it is not showing up — what should I check?
**A:** Add-on trials (Copilot Business, GHAS) must be activated by your GitHub Sales representative or Solutions Engineer — they are not self-service. After requesting, allow 1-2 business days for activation. Once activated, Copilot appears under Enterprise > AI controls > Copilot, and GHAS appears under Organization > Settings > Advanced Security. If it still does not appear, confirm with your GitHub contact that the add-on was applied to the correct enterprise or org.

---

### Q: What happens to my data when the trial expires?
**A:** When a trial expires, the enterprise enters a frozen state. You lose the ability to create new repos, push code, or manage settings, but existing data (repos, issues, PRs) is preserved for a grace period. If you convert to a paid plan within the grace period, all data is retained. If the trial is not converted, GitHub will eventually delete the enterprise and all associated data. Contact your GitHub representative to convert before expiration to avoid any data loss.

---

### Q: Can I convert a trial directly to a paid enterprise without starting over?
**A:** Yes. When you are ready to purchase, work with your GitHub Sales representative to convert the trial to a paid plan. All configuration, repositories, users, and settings carry over — you do not need to rebuild anything. This is one of the key benefits of properly configuring your trial as if it were production.

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

---

## 📝 Resources

| Resource | Link |
|----------|------|
| **Start a trial** | [github.com/enterprise/trial](https://github.com/enterprise/trial) |
| **Trial setup documentation** | [docs.github.com](https://docs.github.com/en/get-started/signing-up-for-github/setting-up-a-trial-of-github-enterprise-cloud) |

---

*Last updated: April 2026*
