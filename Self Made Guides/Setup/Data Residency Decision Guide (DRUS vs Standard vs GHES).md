# 🌐 GitHub Data Residency Decision (DRUS, Standard & GHES) Guide

> Decision framework for choosing between standard GitHub Enterprise Cloud, Data Residency (DRUS/GHE.com), and GitHub Enterprise Server

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Standard GHEC:** No data residency guarantee, broadest feature set, github.com — choose when no formal residency requirement exists
- **DRUS (GHE.com):** Formal US data residency, EMU required, SUBDOMAIN.ghe.com — choose when regulatory/contractual language mandates US data residency
- **GHES:** Self-hosted, full infrastructure control — choose for air-gapped, IL4/IL5, or disconnected environments
- **Migration to DRUS:** Full migration project (4-8 weeks) — new GHE.com enterprise + reconfigure IdP + GEI repo migration + update all integrations
- **Copilot inference:** DRUS does NOT guarantee US-only inference — use BYOK for provider-level region control

---

## ✅ Accuracy & Click-Path Notes

- Reviewed against current public GitHub and Microsoft documentation in April 2026 where public documentation is available. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
- When a path starts with `Enterprise`, begin at GitHub, click your profile photo, click `Your enterprises` or `Enterprise`, select the enterprise, then continue with the listed top tab or left-sidebar item.
- When a path starts with `Organization` or `Org`, begin at GitHub, click your profile photo, click `Your organizations`, select the organization, click `Settings`, then continue with the listed sidebar item.
- When a path starts with `Repository`, `Repo`, or a repository name, open the repository, click the `Settings` tab, then continue with the listed sidebar item.
- When a path starts with a vendor portal such as `Microsoft Entra admin center`, `Azure portal`, `Okta Admin Console`, `PingFederate`, `PingOne`, `OneLogin`, `AD FS Management`, `Visual Studio Admin Portal`, or `Azure DevOps`, sign in to that admin portal first, select the tenant, application, or project named in the step, then follow each listed blade, tab, button, and confirmation in order.
- If the expected button is missing, verify you are signed in with the role named in Prerequisites, the feature or license is enabled, and the object is owned by the selected enterprise, organization, or repository. Use page search only to locate the same page, not to skip required confirmation, test, save, or consent clicks.

---

## ✅ Prerequisites

| Requirement | Status |
|-------------|--------|
| Data residency requirements documented (regulatory, contractual, or policy-driven) | ☐ |
| Compliance team consulted on FedRAMP / IL level needs | ☐ |
| Identity model decision made (Standard vs EMU) | ☐ |
| Migration effort assessed if moving from existing github.com enterprise | ☐ |
| Copilot inference geography requirements clarified with stakeholders | ☐ |

---

## 📋 Overview

| Deployment | Hosting | Identity | Data Location | URL |
|-----------|---------|----------|---------------|-----|
| **Standard GHEC** | GitHub SaaS (github.com) | Personal accounts or EMU | GitHub-managed (primarily US) | github.com |
| **GHEC Data Residency (DRUS)** | GitHub SaaS (ghe.com) | EMU required | Formal US data residency | SUBDOMAIN.ghe.com |
| **GitHub Enterprise Server (GHES)** | Self-hosted | Customer-managed | Your infrastructure | Your URL |

## 1️⃣ Decision Framework

### Choose Standard GHEC When:
- No formal data residency requirement
- Broadest feature parity needed (github.com gets features first)
- Simplest path for adoption and migration
- Standard Enterprise or EMU identity model
- FedRAMP Tailored authorization is sufficient

### Choose DRUS (GHE.com) When:
- Formal US data residency is a stated requirement
- Regulatory or contractual language requires data-residency deployment
- Customer security/compliance team specifically asks for it
- Organization is willing to accept the EMU operating model
- Willing to work with potential feature differences from github.com

### Choose GHES When:
- Air-gapped or disconnected environment required
- IL4/IL5/classified workloads
- Full infrastructure control required
- Existing on-premises commitment with no cloud option

## 2️⃣ Feature Comparison

| Feature | Standard GHEC | DRUS (GHE.com) | GHES |
|---------|:------------:|:--------------:|:----:|
| GitHub Copilot | ✅ | ✅ | ❌ |
| GitHub Actions (hosted runners) | ✅ | ✅ | ❌ (self-hosted only) |
| Advanced Security (GHAS) | ✅ | ✅ | ✅ |
| Data residency guarantee | ❌ | ✅ | ✅ (your infra) |
| EMU support | ✅ | ✅ (required) | N/A |
| Public repositories | ✅ (standard) | ❌ (EMU) | ✅ |
| FedRAMP | Tailored ATO | Tailored ATO | Customer's boundary |
| Copilot inference region lock | ❌ (not guaranteed) | ❌ (not guaranteed) | N/A |

> ⚠️ **Important:** Data residency (DRUS) governs where covered GitHub platform data is stored. It does NOT by itself guarantee that Copilot inference stays in the US.

## 3️⃣ Common Misconceptions

| Misconception | Reality |
|--------------|---------|
| "Standard GHEC defaults to US data residency" | Standard GHEC is primarily US-hosted but this is NOT a formal data residency guarantee |
| "DRUS = FedRAMP Moderate" | DRUS is data residency, not an authorization level change |
| "DRUS = GCC High equivalent" | DRUS is not equivalent to Azure GCC High or IL4/IL5 |
| "DRUS makes Copilot US-only" | Copilot inference geography is separate from platform data residency |
| "We can switch from standard to DRUS with a toggle" | Moving to DRUS requires a full migration to a new GHE.com enterprise |

## 4️⃣ Migration Implications

Moving from standard GHEC to DRUS requires:
1. New enterprise provisioned on GHE.com
2. Reconfigure IdP (SAML/OIDC + SCIM) for new enterprise
3. Migrate repositories using GitHub Enterprise Importer
4. Update ALL integrations: API endpoints, OIDC issuers, package registry URLs, webhook URLs
5. Update OIDC trust: `https://token.actions.SUBDOMAIN.ghe.com` replaces `https://token.actions.githubusercontent.com`

> 💡 **Tip:** Plan this as a migration project with 4-8 weeks timeline, not a settings change.

## 5️⃣ Copilot Inference Geography

| Topic | What Public Docs Say |
|-------|---------------------|
| Platform data storage | DRUS stores covered data in the US |
| Copilot inference location | NOT guaranteed to be US-only by DRUS alone |
| BYOK | Can route inference through your chosen provider endpoint — region guarantee comes from THAT provider |
| Best approach | Treat inference geography as a provider-by-provider validation exercise |

## 6️⃣ FedRAMP Positioning

- GitHub Enterprise Cloud has a **FedRAMP Tailored ATO**
- Do NOT equate FedRAMP Moderate with IL4/IL5
- Do NOT assume DRUS changes the FedRAMP authorization scope
- For exact authorization scope, route through GitHub's compliance/account team

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

### Q: The customer assumes DRUS provides FedRAMP Moderate authorization — is that correct?
**A:** No. DRUS (Data Residency US) provides formal US data residency for covered GitHub platform data. It does not change GitHub's FedRAMP authorization level, which is currently FedRAMP Tailored. Do not equate DRUS with FedRAMP Moderate, GCC High, or IL4/IL5. If the customer requires a specific authorization level, route through GitHub's compliance team for the exact scope. DRUS addresses "where is my data stored," not "what compliance framework is the platform certified under."

---

### Q: The customer thinks data residency means Copilot inference stays in the US — is that true?
**A:** No. Data residency (DRUS) governs where covered GitHub platform data is stored at rest. Copilot inference geography is separate and is not guaranteed to be US-only by DRUS alone. If the customer needs inference region guarantees, explore Bring Your Own Key (BYOK) options where the customer routes inference through their own API endpoint with a provider that offers region guarantees. Treat inference geography as a provider-by-provider validation exercise, not something DRUS solves.

---

### Q: The customer says "can we just toggle data residency on for our existing enterprise" — what is the answer?
**A:** No. Moving from standard GHEC (github.com) to DRUS (GHE.com) requires a full migration. You must provision a new enterprise on GHE.com, reconfigure your IdP (SAML/OIDC + SCIM) for the new enterprise, migrate all repositories using GitHub Enterprise Importer, and update every integration: API endpoints, OIDC trust policies, package registry URLs, webhook URLs, and CI/CD pipeline configurations. Plan this as a 4-8 week migration project, not a settings change.

---

### Q: What are the key feature differences between github.com and GHE.com?
**A:** GHE.com (DRUS) has functional parity with github.com for most features, but there can be feature lag — new features typically ship to github.com first and arrive on GHE.com later. Key differences include: the URL structure changes (e.g., `SUBDOMAIN.ghe.com` instead of `github.com`), OIDC trust issuer URLs change, package registry URLs change, and GHE.com requires EMU (standard enterprise with personal accounts is not available on GHE.com). Check GitHub's data residency feature overview documentation for the current parity status.

---

### Q: The customer underestimates the migration effort from standard GHEC to DRUS — how do I set expectations?
**A:** Emphasize that this is a migration, not a configuration change. The effort includes: (1) new enterprise provisioning, (2) full IdP reconfiguration, (3) repository migration via GEI, (4) updating all OIDC trust policies (the token issuer URL changes), (5) updating all API integrations (different base URL), (6) updating all package registry references, (7) updating all webhook URLs, (8) updating CI/CD pipelines, and (9) user communication and re-onboarding. Most organizations need 4-8 weeks with dedicated engineering resources. Position it as equivalent to a cloud migration.

---

### Q: When should we recommend GHES instead of DRUS?
**A:** Recommend GHES when the customer requires: air-gapped or disconnected environments, IL4/IL5/classified workloads, full infrastructure control (own hardware, own network), or when they have an existing on-premises commitment with no cloud option. GHES does not support Copilot or hosted runners, so weigh those trade-offs. If the customer's only requirement is US data residency and they want full SaaS features (including Copilot), DRUS is the better choice.

---

### Q: Can we use DRUS for non-US data residency (e.g., EU)?
**A:** DRUS specifically stands for Data Residency US. GitHub has announced data residency support for additional regions (including the EU). Check the current GitHub data residency documentation for available regions and timelines. The setup process is similar — a dedicated GHE.com subdomain with EMU — but the region is selected at enterprise creation and cannot be changed afterward.

---

### Q: The customer conflates DRUS with Azure GCC High — how do I clarify?
**A:** DRUS and Azure GCC High are entirely different offerings. Azure GCC High is a US government-specific Azure cloud environment meeting IL4/IL5 requirements. DRUS is GitHub's data residency offering that stores covered platform data in the US — it runs on GitHub's own infrastructure, not in Azure GCC High. There is no GitHub equivalent of Azure GCC High. If the customer needs IL4/IL5, the only option is GHES deployed within their own FedRAMP-authorized boundary or GCC High environment.

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Enterprise Trial (GHEC, EMU, DRUS) | `Setup/Enterprise Trial (GHEC, EMU, DRUS).md` |
| Enterprise Environment Scaffolding Checklist | `Setup/Enterprise Environment Scaffolding Checklist.md` |
| Standard Enterprise to EMU Migration | `Setup/Standard Enterprise to EMU Migration.md` |
| OIDC Federation for Azure Deployments | `Actions/OIDC Federation for Azure Deployments.md` |

---

## 📝 Resources

| Resource | Link |
|----------|------|
| About data residency | [docs.github.com](https://docs.github.com/en/enterprise-cloud@latest/admin/data-residency/about-github-enterprise-cloud-with-data-residency) |
| Feature overview for GHE.com | [docs.github.com](https://docs.github.com/en/enterprise-cloud@latest/admin/data-residency/feature-overview-for-github-enterprise-cloud-with-data-residency) |
| About storage with data residency | [docs.github.com](https://docs.github.com/en/enterprise-cloud@latest/admin/data-residency/about-storage-of-your-data-with-data-residency) |
| Network details for GHE.com | [docs.github.com](https://docs.github.com/en/enterprise-cloud@latest/admin/data-residency/network-details-for-ghecom) |
| GitHub FedRAMP page | [government.github.com](https://government.github.com/fedramp/) |
| About GHES | [docs.github.com](https://docs.github.com/en/enterprise-server@latest/admin/overview/about-github-enterprise-server) |
| BYOK for Copilot | [docs.github.com](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-enterprise/use-your-own-api-keys) |

---

*Last updated: April 2026*
