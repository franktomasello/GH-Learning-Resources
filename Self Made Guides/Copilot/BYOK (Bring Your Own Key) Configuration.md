# 🔑 GitHub Copilot BYOK (Bring Your Own Key) Configuration Runbook

> **Complete guide to configuring your own AI model provider API keys for GitHub Copilot at the enterprise level**

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [📋 Overview](#-overview)
- [🏗️ Supported Providers](#-supported-providers)
- [1️⃣ Configure BYOK at the Enterprise Level](#1-configure-byok-at-the-enterprise-level)
- [2️⃣ Set Policies for BYOK Model Access](#2-set-policies-for-byok-model-access)
- [3️⃣ What BYOK Does](#3-what-byok-does)
- [4️⃣ What BYOK Does NOT Do](#4-what-byok-does-not-do)
- [5️⃣ Use Cases for Public Sector and Regulated Industries](#5-use-cases-for-public-sector-and-regulated-industries)
- [🚀 Quick BYOK Setup Recipe](#-quick-byok-setup-recipe)
- [📝 Additional Notes](#-additional-notes)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📚 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- Configure BYOK: `Enterprise → Settings → Copilot → Policies → Model management → Configure BYOK`
- Add provider and enter API key, then test the connection
- Set access policy: `Enterprise → Settings → Copilot → Policies → Model management` — choose all orgs or selected orgs
- Rotate keys: same BYOK configuration page — update key and re-test

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
| GitHub Enterprise Cloud account with enterprise owner access | ☐ |
| Copilot Business or Copilot Enterprise subscription active | ☐ |
| API key from a supported AI model provider (Anthropic, OpenAI, Azure, AWS, Google, xAI) | ☐ |
| Provider endpoint URL (required for Azure AI Foundry, AWS Bedrock, OpenAI-compatible) | ☐ |

---

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **GitHub enterprise owner** | Configures the BYOK provider and access policy in GitHub Copilot. | GitHub → profile photo → Your enterprises → [enterprise] → Settings → Copilot → Policies → Model management → Configure BYOK → Add provider → select provider → enter required key and endpoint fields → Test connection → Save → set access policy to all organizations or selected organizations. Handoff: provider status shows saved and test succeeds. |
| **Azure AI Foundry resource owner, if Azure AI Foundry is the selected provider** | Provides the endpoint URL, key, deployment name, and quota confirmation for the Azure-hosted model. | Azure AI Foundry portal → Models + endpoints → [deployment] → Endpoint and keys → copy endpoint URL and key. If the model is not deployed yet: Foundry portal → Discover → Models → [model] → Deploy → Default settings or Custom settings → complete deployment → open deployment details. Handoff: endpoint URL, key, deployment name, region, and quota owner. |
| **AI provider account owner for non-Azure providers** | Creates or rotates the provider API key and confirms billing ownership outside GitHub. | Provider admin console → API keys or access keys → Create new key → copy key once → restrict or tag it for GitHub Copilot BYOK where supported → record rotation owner and billing account. Handoff: key, endpoint if required, region if required, and rotation date. |

---

## 📋 Overview

BYOK (Bring Your Own Key) allows enterprise administrators to configure their own API keys from supported AI model providers so that Copilot routes requests through the enterprise's own provider account.

| Aspect | Details |
|--------|---------|
| **What it does** | Routes Copilot model requests through your own AI provider account |
| **Who configures it** | Enterprise owners |
| **Billing** | Usage is billed by the AI model provider to your account, not through GitHub |
| **Scope** | Enterprise-wide, with policies for which orgs/users can use BYOK models |

> ⚠️ **Important:** BYOK refers to AI model provider API keys for Copilot. This is NOT the same as customer-managed encryption keys (CMK/BYOK) for encrypting platform data at rest. Those are separate features.

---

## 🏗️ Supported Providers

| Provider | Key Type | Notes |
|----------|----------|-------|
| **Anthropic** | API key | Claude models |
| **OpenAI** | API key | GPT models |
| **xAI** | API key | Grok models |
| **Microsoft Azure AI Foundry** | API key + endpoint | Azure-hosted models |
| **AWS Bedrock** | Access key + secret | AWS-hosted models |
| **Google AI Studio** | API key | Gemini models |
| **OpenAI-compatible endpoints** | API key + endpoint | Any provider with an OpenAI-compatible API |

---

## 1️⃣ Configure BYOK at the Enterprise Level

**Navigation:**

```
Enterprise → Settings → Copilot → Policies
  → Model management → Configure BYOK
```

**Steps:**

1. Click **Add provider**
2. Select the AI model provider from the list
3. Enter the required credentials:

| Provider | Required Fields |
|----------|----------------|
| **Anthropic** | API key |
| **OpenAI** | API key |
| **xAI** | API key |
| **Azure AI Foundry** | API key, Endpoint URL |
| **AWS Bedrock** | Access key ID, Secret access key, Region |
| **Google AI Studio** | API key |
| **OpenAI-compatible** | API key, Endpoint URL |

4. Test the connection to verify the key works
5. Save the configuration

> ✅ **Result:** The enterprise now has a BYOK provider configured. Models from this provider will be available based on the policies you set in the next step.

---

## 2️⃣ Set Policies for BYOK Model Access

*Control which organizations and users can use the BYOK-configured models*

**Navigation:**

```
Enterprise → Settings → Copilot → Policies
  → Model management
```

**Steps:**

1. For each configured BYOK provider, set the access policy:

| Policy | Effect |
|--------|--------|
| **All organizations** | Every org in the enterprise can use the BYOK models |
| **Selected organizations** | Only chosen orgs can use the BYOK models |
| **Disabled** | BYOK models are configured but not available to any org |

2. Save the policy

> 💡 **Tip:** Start with a selected set of organizations to validate the configuration and monitor usage before rolling out enterprise-wide.

---

## 3️⃣ What BYOK Does

| Capability | Description |
|------------|-------------|
| **Provider billing** | Model usage is billed directly to your provider account |
| **Model selection** | Users can select BYOK models in Copilot Chat model picker |
| **Governance** | Enterprise controls which providers and models are available |
| **Existing contracts** | Leverage existing enterprise agreements with AI providers |
| **Cost tracking** | Track Copilot model usage through your provider's billing dashboard |

---

## 4️⃣ What BYOK Does NOT Do

| Limitation | Details |
|------------|---------|
| **Not air-gapped** | BYOK still requires network connectivity to GitHub and the AI provider |
| **Not a data residency guarantee** | Data handling is governed by the AI provider's terms, not GitHub's |
| **Provider compliance is the provider's responsibility** | GitHub does not certify or audit BYOK providers for FedRAMP, ITAR, etc. |
| **Not a replacement for Copilot licensing** | You still need Copilot Business or Enterprise seats; BYOK adds model routing |
| **Not encryption key management** | BYOK for Copilot is about AI model API keys, not encrypting data at rest |

> ⚠️ **Important:** Evaluate your AI model provider's compliance certifications independently. GitHub's compliance posture does not extend to third-party providers configured via BYOK.

---

## 5️⃣ Use Cases for Public Sector and Regulated Industries

| Use Case | How BYOK Helps |
|----------|----------------|
| **Existing provider contracts** | Route through an AI provider you already have an enterprise agreement with |
| **Billing consolidation** | All AI model costs flow through a single provider account for easier tracking |
| **Governance and approval** | Use only providers that have passed your organization's security review |
| **Budget control** | Set spend limits and alerts at the provider level, independent of GitHub |
| **Model standardization** | Ensure all teams use approved models through the same provider account |

> 💡 **Tip:** For public sector customers, BYOK is often paired with Azure AI Foundry or AWS Bedrock where the organization already has a cloud provider agreement with appropriate compliance certifications (FedRAMP, IL4/IL5, etc.).

---

## 🚀 Quick BYOK Setup Recipe

*Steps for a straightforward BYOK configuration:*

### Steps

1. **Obtain an API key** from your AI model provider

2. **Navigate to BYOK configuration:**
   ```
   Enterprise → Settings → Copilot → Policies
     → Model management → Configure BYOK
   ```

3. **Add the provider** and enter your API key

4. **Test the connection**

5. **Set the access policy** to selected organizations for initial rollout

6. **Notify org admins** that BYOK models are now available in the model picker

7. **Monitor usage** through both GitHub's Copilot metrics and your provider's billing dashboard

---

## 📝 Additional Notes

> 💡 **Customization:** The BYOK feature is evolving. The exact list of supported providers and configuration options may expand over time. Check the GitHub documentation for the latest supported providers and configuration steps.

## 🧯 Known Errors & Resolutions

<details>
<summary><em>Show known errors table</em></summary>


> This section lists the known product errors and admin-facing symptoms that commonly occur with this workflow. Exact message text can vary by product rollout, tenant policy, and provider, so use the log or settings page named in the resolution to confirm the root cause.

| Error or symptom | Likely cause | Resolution |
|------------------|--------------|------------|
| **Page, tab, or button is missing** | Wrong account context, missing admin role, unavailable plan/add-on, or feature rollout not enabled for the selected enterprise/org/repo. | Switch to the correct account and scope, confirm the prerequisite role, verify licensing or add-on activation, then refresh the page. If the control is still absent, use the direct settings URL from the relevant GitHub Docs page and confirm the feature is available for your plan. |
| **Changes appear saved but behavior does not change** | Policy inheritance, cached UI state, propagation delay, or an overlapping enterprise/org/repo policy. | Reopen the settings page, verify the effective policy at the lowest affected scope, wait for propagation where documented, and check for a stricter policy at an enterprise or organization level. |
| **403, forbidden, or resource not accessible** | The signed-in user or token can see the page but lacks the specific permission for the action. | Use an enterprise owner, organization owner, repository admin, or token with the exact scopes/permissions listed in the runbook. For SAML-protected orgs, authorize the token or SSH key for SSO before retrying. |
| **Copilot feature, model, or policy is not visible** | Plan, license assignment, enterprise policy, org delegation, or feature rollout does not permit it. | Check enterprise AI controls, organization Copilot settings, assigned seat status, and the plan requirements for the feature. |
| **Premium requests are rejected after the included allowance** | Paid usage is disabled, no billing entity is selected, or a stop-usage budget is exhausted. | Enable Premium request paid usage where appropriate, set or delete conflicting budgets, and have users with multiple licenses choose a billing entity. |
| **Content exclusions do not apply immediately** | Client policy cache, unsupported surface/mode, symlink/remote filesystem limitation, or indirect IDE context. | Reload the IDE policy, verify the exclusion syntax at enterprise/org/repo scope, and document surfaces where exclusions are limited. |
| **Usage metrics look empty or inconsistent** | Telemetry is disabled, data freshness delay applies, users are unlicensed, or different APIs report different scopes. | Enable the metrics policy, confirm seats and telemetry, wait for data freshness, and avoid comparing dashboards/API endpoints as if they share identical data models. |
| **Coding agent or MCP action is denied** | Agent policy, MCP policy, repository permissions, secrets, or server allowlist does not permit the operation. | Review Enterprise AI controls > Agents/MCP, repo-level permissions, MCP server configuration, and audit logs for the denied action. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>


### Q: BYOK is not the same as CMK for data at rest — correct?
**A:** Correct. BYOK in the Copilot context means bringing your own AI model provider API keys so that inference requests route through your provider account. This is completely separate from customer-managed encryption keys (CMK/BYOK) for encrypting platform data at rest.

---

### Q: Can we use BYOK for air-gapped environments?
**A:** No. BYOK still requires network connectivity to both GitHub and the AI model provider endpoint. It changes where inference happens but does not remove the need for outbound network access.

---

### Q: The provider endpoint is returning errors after BYOK setup — what should we check?
**A:** Verify three things: (1) the API key is valid and has not expired or been revoked at the provider, (2) check the provider's rate limits and quota — you may be hitting usage caps, and (3) ensure the endpoint URL is correct (especially for Azure AI Foundry or OpenAI-compatible endpoints where a custom URL is required).

---

### Q: Can different orgs under the same enterprise use different BYOK providers?
**A:** Yes, BYOK can be configured at the enterprise level with policies controlling which organizations have access to which BYOK-configured providers. You can set access to "Selected organizations" per provider.

---

### Q: Does BYOK change where inference happens?
**A:** Yes. With BYOK, inference requests are routed through your chosen provider's infrastructure instead of GitHub's default model endpoints. This means data handling and compliance are governed by your provider's terms, not GitHub's.

---

### Q: Do we still need Copilot licenses if we use BYOK?
**A:** Yes. BYOK adds model routing but does not replace Copilot licensing. You still need Copilot Business or Enterprise seats for each user. BYOK model usage is billed by the AI provider separately from the GitHub Copilot subscription.

---

### Q: How do we rotate or update a BYOK API key?
**A:** Navigate to Enterprise > Settings > Copilot > Policies > Model management > Configure BYOK, select the provider, and update the API key. Test the connection after updating to verify the new key works. There is no automatic key rotation — this must be done manually.

</details>

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Admin Controls (Models, Content Exclusion, Instructions) | `Copilot/Admin Controls (Models, Content Exclusion, Instructions).md` |
| Data Residency Decision Guide (DRUS vs Standard vs GHES) | `Setup/Data Residency Decision Guide (DRUS vs Standard vs GHES).md` |
| Premium Request Budget & Overage Planning | `Copilot/Premium Request Budget & Overage Planning.md` |

---

## 📚 Resources

- [Use your own API keys for Copilot](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-enterprise/use-your-own-api-keys)
- [Enterprise BYOK for GitHub Copilot changelog](https://github.blog/changelog/2025-11-20-enterprise-bring-your-own-key-byok-for-github-copilot-is-now-in-public-preview/)

---

*Last updated: April 2026*
