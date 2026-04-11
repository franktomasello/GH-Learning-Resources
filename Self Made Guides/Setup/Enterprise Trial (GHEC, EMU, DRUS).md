# 🧪 GitHub Enterprise Trial Setup Runbook (GHEC, EMU, DRUS)

> **Complete guide to starting a GitHub Enterprise Cloud trial — Standard GHEC, EMU (managed users), or Data Residency US (DRUS)**

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

*For standard GHEC, SAML SSO is configured at the organization level, not the enterprise level.*

**Navigation:**

```
Organization → Settings → Authentication security
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

---

## 📝 Resources

| Resource | Link |
|----------|------|
| **Start a trial** | https://github.com/enterprise/trial |
| **Trial setup documentation** | https://docs.github.com/en/enterprise-cloud@latest/admin/setting-up-your-enterprise/setting-up-a-trial-of-github-enterprise-cloud |

---

*Last updated: April 2026*