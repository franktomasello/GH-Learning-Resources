# 🔑 GitHub Copilot BYOK (Bring Your Own Key) Configuration Runbook

> **Complete guide to configuring your own AI model provider API keys for GitHub Copilot at the enterprise level**

| 🧭 **At a Glance** | |
|---|---|
| **Goal** | Let Copilot use models through the customer's own AI provider account |
| **Use this when** | The customer has an existing agreement with Microsoft Foundry, OpenAI, Anthropic, AWS Bedrock, or similar |
| **People you need** | Enterprise owner; provider account owner |
| **Where you click** | GitHub (AI controls) and the AI provider's console |
| **End result** | Custom models in the Copilot model picker, billed by the provider |
| **New to a term?** | See the [Glossary](../Glossary.md) for plain-English definitions |

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [📋 Overview](#-overview)
- [🏗️ Supported Providers](#️-supported-providers)
- [1️⃣ Configure BYOK at the Enterprise Level](#1️⃣-configure-byok-at-the-enterprise-level)
- [2️⃣ Set Policies for BYOK Model Access](#2️⃣-set-policies-for-byok-model-access)
- [3️⃣ What BYOK Does](#3️⃣-what-byok-does)
- [4️⃣ What BYOK Does NOT Do](#4️⃣-what-byok-does-not-do)
- [5️⃣ Use Cases for Public Sector and Regulated Industries](#5️⃣-use-cases-for-public-sector-and-regulated-industries)
- [🚀 Quick BYOK Setup Recipe](#-quick-byok-setup-recipe)
- [📝 Additional Notes](#-additional-notes)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📚 Resources](#-resources)

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- Add a key: `Enterprise → AI controls → Copilot → Configure custom models → Add API key` → **Provider**, **Name**, **API key** → select models → **Save**
- Grant access: **Added models** tab → **Configure** next to a model → **Access** tab → **Allow for all organizations** or **Choose per organization** → **Save**
- Let org owners add their own keys: enable the **Enable custom models** policy

---

## ✅ Accuracy & Click-Path Notes

<details>
<summary><em>Show click-path conventions</em></summary>

- Reviewed against current public GitHub documentation in October 2026 where public documentation is available. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
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
| GitHub Enterprise Cloud enterprise with Copilot Business or Copilot Enterprise | GitHub **enterprise owner** | ☐ |
| An API key from a supported provider (Anthropic, Microsoft Foundry, OpenAI, xAI, AWS Bedrock, Google AI Studio, or an OpenAI-compatible provider) | Provider account owner | ☐ |
| For Microsoft Foundry: the model's **deployment URL** and model IDs | Foundry resource owner | ☐ |
| Agreement on which organizations may use each model | Platform / security owners | ☐ |

---

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **GitHub enterprise owner** | Adds the provider key and decides which organizations can use each model. | GitHub → Enterprise → AI controls → Copilot → Configure custom models → Add API key → choose Provider → enter Name and API key → fetch or enter the models → Save → Added models → Configure (next to a model) → Access → Allow for all organizations or Choose per organization → Save. Handoff: models listed under Added models with the intended access. |
| **Microsoft Foundry resource owner, if Microsoft Foundry is the provider** | Provides the deployment URL, key, and model IDs for the deployed model. | Microsoft Foundry portal → open the project → the model deployment → copy its endpoint (deployment URL), key, and model/deployment name. Deploy the model first if needed. Handoff: deployment URL, API key, model IDs, and quota owner. |
| **AI provider account owner for other providers** | Creates or rotates the provider API key and owns billing with the provider. | Provider admin console → API keys → create a new key → copy it once → restrict or tag it for GitHub Copilot where supported → record the rotation owner and billing account. Handoff: API key, any endpoint or region the form asks for, and rotation date. |

---

## 📋 Overview

BYOK lets you connect your own AI provider keys so Copilot can use models through **your** provider account.

| Aspect | Details |
|--------|---------|
| **Two kinds** | **Enterprise BYOK** ("custom models", set up by admins — **public preview**) and **local BYOK** (a user's own key, stored only in their client) |
| **Where models can be used** | Copilot Chat, Copilot CLI, and IDEs |
| **Who configures it** | Enterprise owners — and organization owners, if the **Enable custom models** policy allows it |
| **Billing** | Usage is billed by your provider under your own terms, and doesn't count against your Copilot allowance |
| **Who can use the models** | Users on the enterprise's Copilot Business or Copilot Enterprise plan, in the organizations you allow. They need a Copilot license and internet access |
| **Where users find them** | At the bottom of the model picker, under the enterprise (or organization) name |

> ⚠️ **Important:** BYOK here means AI provider API keys for Copilot. It is NOT customer-managed encryption keys (CMK) for encrypting platform data at rest — those are separate features.

> 📌 **Local BYOK:** users can add their own keys in VS Code, JetBrains, Xcode, Copilot CLI, the GitHub Copilot app, and the Copilot SDK. Those keys stay on the user's machine and aren't shared. For Copilot Business and Copilot Enterprise users, local BYOK in IDEs can be turned off by an enterprise or organization policy.

---

## 🏗️ Supported Providers

| Provider | What you enter | How models are added |
|----------|----------------|----------------------|
| **Anthropic** | API key | Click the sync icon to fetch the models tied to your key |
| **OpenAI** | API key | Click the sync icon to fetch models |
| **xAI** | API key | Click the sync icon to fetch models |
| **Microsoft Foundry** (formerly Azure AI Foundry) | API key + **Deployment URL** | Type each model ID and click the checkmark |
| **AWS Bedrock** | API key + any fields the form asks for | As prompted |
| **Google AI Studio** | API key | As prompted |
| **OpenAI-compatible providers** | API key + endpoint as prompted | As prompted |

> 📌 For Microsoft Foundry, models with different deployment URLs can't be added under the same API key — add a separate key for each deployment URL.

> 💡 Fine-tuned models are supported too, but results vary with the fine-tuning setup. Test the model before rolling it out. Give each API key only the minimum scopes it needs.

---

## 1️⃣ Configure BYOK at the Enterprise Level

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **AI controls** → **Copilot** *(sidebar)* → **Configure custom models**

**Steps:**

1. At the top of the enterprise page, click **AI controls**, then **Copilot** in the sidebar.
2. Click **Configure custom models**.
3. Above the list of API keys, click **Add API key**.
4. Under **Provider**, select your provider.
5. Under **Name**, type a name for the key. It's shown in the model picker.
6. Under **API key**, type or paste the key.
7. Add models:
   - **Anthropic, OpenAI, xAI:** click the sync icon in the **API key** field to fetch your models, then pick them from **Available models**.
   - **Microsoft Foundry:** under **Deployment URL**, type the deployment URL. Then under **Available models**, type a model ID and click the checkmark (**Add model**). Repeat for each model.
8. Click **Save**.

> ✅ **Result:** the models appear on the **Added models** tab. Members of the organizations you allow (Section 2) see them at the bottom of the model picker, under the enterprise name.

---

## 2️⃣ Set Policies for BYOK Model Access

*Control which organizations can use each custom model*

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **AI controls** → **Copilot** *(sidebar)* → **Configure custom models** → **Added models** tab

**Steps:**

1. Make sure the model is set to **Enabled** — the **Access** tab only appears for enabled models.
2. Next to the model, click **Configure**. *(If some organizations already have access, the button reads **All organizations** or **X organizations** instead — click that.)*
3. In the dialog, click the **Access** tab.
4. Choose who gets the model:
   - **Allow for all organizations** — every organization in the enterprise, or
   - **Choose per organization** — then check or uncheck each organization in the list.
5. Click **Save**.

> 💡 **Tip:** Start with a few organizations to validate the setup and watch provider costs before enabling it everywhere.

### Optional: let organization owners add their own keys

1. **Enterprise owner:** enable the **Enable custom models** policy for the enterprise.
2. **Organization owner:** Organization → **Settings** → under **Code, planning, and automation**, click **Copilot** → **Models** → **Custom models** tab.
3. Follow steps 3–8 from Section 1. These models appear in the model picker under the organization name.

---

## 3️⃣ What BYOK Does

| Capability | Description |
|------------|-------------|
| **Provider billing** | Model usage is billed by your provider under your agreement — not from your Copilot allowance |
| **Model selection** | Users can pick your custom models in Copilot Chat, Copilot CLI, and IDEs |
| **Governance** | Enterprise owners control which models each organization can use |
| **Existing contracts** | Use existing enterprise agreements and credits with your AI provider |
| **Cost tracking** | Track model spend in your provider's billing dashboard |

---

## 4️⃣ What BYOK Does NOT Do

| Limitation | Details |
|------------|---------|
| **Not air-gapped** | Enterprise BYOK runs through the Copilot API, so users still need internet access and a Copilot license. (Only *local* BYOK removes the dependency on GitHub's Copilot API.) |
| **Not a data residency guarantee by itself** | Data handling for model calls follows your provider's terms and region |
| **Provider compliance is the provider's responsibility** | GitHub doesn't certify BYOK providers for FedRAMP, ITAR, and so on |
| **Not a replacement for Copilot licensing** | Users still need a Copilot Business or Copilot Enterprise seat |
| **Not encryption key management** | It's about AI model API keys, not encrypting data at rest |
| **Not every Copilot surface** | Custom models are supported in Copilot Chat, Copilot CLI, and IDEs — check GitHub's docs before assuming other features use them |

> ⚠️ **Important:** Evaluate your AI provider's compliance certifications independently. GitHub's compliance posture doesn't extend to third-party providers you connect.

---

## 5️⃣ Use Cases for Public Sector and Regulated Industries

| Use Case | How BYOK Helps |
|----------|----------------|
| **Existing provider contracts** | Route through an AI provider you already have an enterprise agreement with |
| **Billing consolidation** | Model costs flow through one provider account for easier tracking |
| **Governance and approval** | Use only providers that passed your security review |
| **Budget control** | Set spend limits and alerts at the provider, independent of GitHub |
| **Model standardization** | Make sure teams use approved models through the same provider account |

> 💡 **Tip:** Public sector customers often pair BYOK with Microsoft Foundry or AWS Bedrock where they already have a cloud agreement with the right compliance certifications. On GHE.com (data residency), also look at the **Restrict Copilot to data residency models** and **Restrict Copilot to FedRAMP models** policies for GitHub-hosted models.

---

## 🚀 Quick BYOK Setup Recipe

*Steps for a straightforward BYOK setup:*

### Steps

1. **Get an API key** from your provider (and, for Microsoft Foundry, the deployment URL and model IDs).
2. **Open custom models:** Enterprise → **AI controls** → **Copilot** *(sidebar)* → **Configure custom models**.
3. **Add the key:** **Add API key** → **Provider** → **Name** → **API key** → fetch or enter models → **Save**.
4. **Limit access for the pilot:** **Added models** → make sure the model is **Enabled** → **Configure** → **Access** → **Choose per organization** → check the pilot orgs → **Save**.
5. **Tell the pilot users** that the models are in their Copilot model picker.
6. **Monitor cost** in your provider's billing dashboard.

---

## 📝 Additional Notes

> 💡 **Customization:** Enterprise BYOK is in public preview, so supported providers and options can change. Check GitHub's documentation for the latest list.

---

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
| **Copilot stops working for a user mid-cycle** | The user's user-level budget is used up, the shared AI credit pool is exhausted with **AI credits paid usage** disabled, or a spending limit with **Stop usage** was reached. | Check the user on **Billing and licensing** → **AI usage** and the budgets on **Budgets and alerts**; raise their budget, approve their budget request (not available for EMU enterprises), or enable AI credits paid usage. |
| **Content exclusions do not apply immediately** | Client policy cache, unsupported surface/mode, symlink/remote filesystem limitation, or indirect IDE context. | Reload the IDE policy, verify the exclusion syntax at enterprise/org/repo scope, and document surfaces where exclusions are limited. |
| **Usage metrics look empty or inconsistent** | Telemetry is disabled, data freshness delay applies, users are unlicensed, or different APIs report different scopes. | Enable the metrics policy, confirm seats and telemetry, wait for data freshness, and avoid comparing dashboards/API endpoints as if they share identical data models. |
| **Cloud agent or MCP action is denied** | Agent policy, MCP policy, repository permissions, secrets, or server allowlist does not permit the operation. | Review Enterprise AI controls > Agents/MCP, repo-level permissions, MCP server configuration, and audit logs for the denied action. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>

### Q: BYOK is not the same as CMK for data at rest — correct?
**A:** Correct. BYOK in the Copilot context means bringing your own AI model provider API keys so that inference requests route through your provider account. This is completely separate from customer-managed encryption keys (CMK/BYOK) for encrypting platform data at rest.

---

### Q: Can we use BYOK for air-gapped environments?
**A:** Not with Enterprise BYOK — it's handled server-side through the Copilot API, so users need internet access and a Copilot license. *Local* BYOK (keys configured in a user's own client) removes the dependency on GitHub's Copilot API, which GitHub notes can suit air-gapped environments.

---

### Q: The provider endpoint is returning errors after BYOK setup — what should we check?
**A:** Check that (1) the API key is valid and hasn't expired or been revoked, (2) you aren't hitting the provider's rate limits or quota, (3) the key has the scopes the model needs, and (4) for Microsoft Foundry or OpenAI-compatible providers, the deployment URL or endpoint is correct — models with different Foundry deployment URLs need separate keys.

---

### Q: Can different orgs under the same enterprise use different BYOK providers?
**A:** Yes. Access is set **per model**: **Added models** → **Configure** → **Access** → **Choose per organization**. Organization owners can also add their own keys if the **Enable custom models** policy is on.

---

### Q: Does BYOK change where inference happens?
**A:** Yes. Requests for a custom model are served by your provider instead of a GitHub-hosted model, so the provider's terms and region govern how that model handles the data. Copilot itself still runs through GitHub.

---

### Q: Do we still need Copilot licenses if we use BYOK?
**A:** Yes, for Enterprise BYOK. Custom models apply to users on the enterprise's Copilot Business or Copilot Enterprise plan. Model usage is billed by your provider and doesn't count against your Copilot allowance.

---

### Q: How do we rotate or update a BYOK API key?
**A:** Rotation is manual. Create the new key at your provider, then go to Enterprise → **AI controls** → **Copilot** → **Configure custom models** and add the new key (**Add API key**) with the same models. Check that the models work, then remove the old key and revoke it at the provider. If you replace a key, recheck each model's **Access** settings.

</details>

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Admin Controls (Models, Content Exclusion, Instructions) | `Copilot/Admin Controls (Models, Content Exclusion, Instructions).md` |
| Data Residency Decision Guide (DRUS vs Standard vs GHES) | `Setup/Data Residency Decision Guide (DRUS vs Standard vs GHES).md` |
| AI Credits Budget & Overage Planning | `Copilot/AI Credits Budget & Overage Planning.md` |

---

## 📚 Resources

- [Enable custom models for your enterprise](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-enterprise/enable-custom-models)
- [Enable custom models for your organization](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-organization/enable-custom-models)
- [About bring your own key (BYOK)](https://docs.github.com/en/copilot/concepts/models/bring-your-own-key)
- [Enterprise BYOK for GitHub Copilot changelog](https://github.blog/changelog/2025-11-20-enterprise-bring-your-own-key-byok-for-github-copilot-is-now-in-public-preview/)

---

*Last updated: October 2026*
