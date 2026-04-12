# 🔑 GitHub Copilot BYOK (Bring Your Own Key) Configuration Runbook

> **Complete guide to configuring your own AI model provider API keys for GitHub Copilot at the enterprise level**

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

---

## ❓ Common Questions & Troubleshooting

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

---

## 📚 Resources

- [Use your own API keys for Copilot](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-enterprise/use-your-own-api-keys)
- [Enterprise BYOK for GitHub Copilot changelog](https://github.blog/changelog/2025-11-20-enterprise-bring-your-own-key-byok-for-github-copilot-is-now-in-public-preview/)

---

*Last updated: April 2026*
