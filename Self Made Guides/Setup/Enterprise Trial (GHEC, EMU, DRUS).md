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

---

## ❓ Common Questions & Troubleshooting

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
**A:** Add-on trials (Copilot Business, GHAS) must be activated by your GitHub Sales representative or Solutions Engineer — they are not self-service. After requesting, allow 1-2 business days for activation. Once activated, Copilot appears under Enterprise > Settings > AI controls > Copilot, and GHAS appears under Organization > Settings > Advanced Security. If it still does not appear, confirm with your GitHub contact that the add-on was applied to the correct enterprise or org.

---

### Q: What happens to my data when the trial expires?
**A:** When a trial expires, the enterprise enters a frozen state. You lose the ability to create new repos, push code, or manage settings, but existing data (repos, issues, PRs) is preserved for a grace period. If you convert to a paid plan within the grace period, all data is retained. If the trial is not converted, GitHub will eventually delete the enterprise and all associated data. Contact your GitHub representative to convert before expiration to avoid any data loss.

---

### Q: Can I convert a trial directly to a paid enterprise without starting over?
**A:** Yes. When you are ready to purchase, work with your GitHub Sales representative to convert the trial to a paid plan. All configuration, repositories, users, and settings carry over — you do not need to rebuild anything. This is one of the key benefits of properly configuring your trial as if it were production.

---

### Q: I set up an EMU trial but realized I need Standard GHEC instead — can I switch?
**A:** No. The identity model (Standard vs EMU vs DRUS) is set at enterprise creation and cannot be changed. You must create a new trial with the correct type. If you need to preserve any repository data from the EMU trial, use GitHub Enterprise Importer (GEI) to migrate repos to the new enterprise. Plan the identity model decision carefully before starting the trial.

---

## 📝 Resources

| Resource | Link |
|----------|------|
| **Start a trial** | [github.com/enterprise/trial](https://github.com/enterprise/trial) |
| **Trial setup documentation** | [docs.github.com](https://docs.github.com/en/enterprise-cloud@latest/admin/setting-up-your-enterprise/setting-up-a-trial-of-github-enterprise-cloud) |

---

*Last updated: April 2026*