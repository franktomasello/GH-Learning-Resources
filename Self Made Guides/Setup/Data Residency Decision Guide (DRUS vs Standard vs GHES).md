# 🌐 GitHub Data Residency Decision (DRUS, Standard & GHES) Guide

> Decision framework for choosing between standard GitHub Enterprise Cloud, Data Residency (DRUS/GHE.com), and GitHub Enterprise Server

> 📌 **Naming note:** The official product name is **GitHub Enterprise Cloud with data residency**; GitHub's own recent shorthand is **GHEC-DR**. This guide uses **DRUS** ("Data Residency US") as informal shorthand for the US-region deployment to keep the decision framework concise — expect to see **GitHub Enterprise Cloud with data residency** / **GHEC-DR** in official docs.

| 🧭 **At a Glance** | |
|---|---|
| **Goal** | Choose between standard GHEC, GHE.com data residency, and GHES |
| **Use this when** | A customer has data-location, sovereignty, or regulatory requirements |
| **People you need** | Decision makers; security and compliance; enterprise owner |
| **Where you click** | None — this is a decision guide |
| **End result** | A clear deployment recommendation and its trade-offs |
| **New to a term?** | See the [Glossary](../Glossary.md) for plain-English definitions |

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [📋 Overview](#-overview)
- [1️⃣ Decision Framework](#1️⃣-decision-framework)
- [2️⃣ Feature Comparison](#2️⃣-feature-comparison)
- [3️⃣ Common Misconceptions](#3️⃣-common-misconceptions)
- [4️⃣ Migration Implications](#4️⃣-migration-implications)
- [5️⃣ Copilot Inference Geography](#5️⃣-copilot-inference-geography)
- [6️⃣ FedRAMP Positioning](#6️⃣-fedramp-positioning)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Standard GHEC:** No data residency guarantee, broadest feature set, github.com — choose when no formal residency requirement exists
- **DRUS (GHE.com):** Formal US data residency, EMU required, SUBDOMAIN.ghe.com — choose when regulatory/contractual language mandates US data residency
- **GHES:** Self-hosted, full infrastructure control — choose for air-gapped, IL4/IL5, or disconnected environments
- **Migration to DRUS:** Full migration project (4-8 weeks) — new GHE.com enterprise + reconfigure IdP + GEI repo migration + update all integrations
- **Copilot inference:** DRUS does NOT automatically pin inference to the US. On GHE.com, an admin can turn on the **Restrict Copilot to data residency models** policy (US and EU regions, GA Apr 2026, off by default) to keep inference, prompts, responses, logs, and telemetry in-region; BYOK remains an alternative for provider-level control

---

## ✅ Accuracy & Click-Path Notes

<details>
<summary><em>Show click-path conventions</em></summary>

- Reviewed against current public GitHub and Microsoft documentation in October 2026 where public documentation is available (including the April 13, 2026 general availability of GitHub Copilot data residency for US and EU). Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
- When a path starts with `Enterprise`, begin at GitHub, click your profile picture, click `Enterprise` (managed/EMU accounts) — or open the `Enterprises` page at github.com/settings/enterprises (standard accounts) —, select the enterprise, then continue with the listed top tab or left-sidebar item.
- When a path starts with `Organization` or `Org`, begin at GitHub, click your profile picture, click `Organizations`, select the organization, click `Settings`, then continue with the listed sidebar item.
- When a path starts with `Repository`, `Repo`, or a repository name, open the repository, click the `Settings` tab, then continue with the listed sidebar item.
- When a path starts with a vendor portal such as `Microsoft Entra admin center`, `Azure portal`, `Okta Admin Console`, `PingFederate`, `PingOne`, `OneLogin`, `AD FS Management`, `Visual Studio Admin Portal`, or `Azure DevOps`, sign in to that admin portal first, select the tenant, application, or project named in the step, then follow each listed blade, tab, button, and confirmation in order.
- If the expected button is missing, verify you are signed in with the role named in Prerequisites, the feature or license is enabled, and the object is owned by the selected enterprise, organization, or repository. Use page search only to locate the same page, not to skip required confirmation, test, save, or consent clicks.

</details>

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

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **GitHub enterprise owner or procurement owner** | Chooses GitHub.com, GHE.com with data residency, or GHES before identity and billing are configured. | GitHub sales/procurement flow or enterprise setup link → select deployment model → confirm enterprise slug or GHE.com subdomain → complete enterprise creation → GitHub → Enterprises page (github.com/settings/enterprises) → [enterprise]. Handoff: selected hosting model, enterprise URL, and subdomain if applicable. |
| **Microsoft Entra, Okta, or PingFederate admin** | Uses the app and URLs that match the selected hosting model. | For GitHub.com, use the standard GitHub Enterprise Managed User or GitHub Enterprise Cloud app. For GHE.com, use the GHE.com-specific Okta app or GitHub EMU Connector metadata and SCIM URL format. Entra/Okta/Ping portal → GitHub app → Single sign-on and Provisioning → enter GitHub.com or GHE.com URLs exactly. Handoff: SSO values and SCIM Tenant URL matching the chosen environment. |
| **Azure subscription Owner, if Azure billing is used** | Confirms the subscription can be connected regardless of the selected GitHub hosting model. | Azure portal → Subscriptions → [subscription] → Access control (IAM) → Role assignments → confirm Owner, then Enterprise path: GitHub → Enterprises page (github.com/settings/enterprises) → [enterprise] → Billing and licensing → Payment information → Metered billing via Azure → Add Azure Subscription. Organization path: GitHub → profile picture → Organizations → [organization] → Settings → Billing and licensing → Payment information → Metered billing via Azure → Add Azure Subscription. Then sign in to Microsoft → Permissions requested → Accept → Select subscription → Connect. Handoff: connected subscription ID. |

---

## 📋 Overview

| Deployment | Hosting | Identity | Data Location | URL |
|-----------|---------|----------|---------------|-----|
| **Standard GHEC** | GitHub SaaS (github.com) | Personal accounts or EMU | GitHub-managed (primarily US) | github.com |
| **GHEC Data Residency (DRUS)** | GitHub SaaS (ghe.com) | EMU required | Formal US data residency | SUBDOMAIN.ghe.com |
| **GitHub Enterprise Server (GHES)** | Self-hosted | Customer-managed | Your infrastructure | Your URL |

---

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

---

## 2️⃣ Feature Comparison

| Feature | Standard GHEC | DRUS (GHE.com) | GHES |
|---------|:------------:|:--------------:|:----:|
| GitHub Copilot | ✅ | ✅ | ❌ |
| GitHub Actions (hosted runners) | ✅ | ✅ | ❌ (self-hosted only) |
| Advanced Security | ✅ (Secret Protection + Code Security) | ✅ (Secret Protection + Code Security) | ✅ (GHAS bundle) |
| Data residency guarantee | ❌ | ✅ | ✅ (your infra) |
| EMU support | ✅ | ✅ (required) | N/A |
| Public repositories | ✅ (standard) | ❌ (EMU) | ✅ |
| FedRAMP | Tailored ATO | Tailored ATO | Customer's boundary |
| Copilot inference region lock | ❌ (BYOK only) | ⚙️ Opt-in policy: **Restrict Copilot to data residency models** (US/EU) | N/A |

> ⚠️ **Important:** Data residency (DRUS) governs where covered GitHub platform data is stored. It does NOT by itself pin Copilot inference to the US. Region-locked inference is available on GHE.com as a **separate, admin-enabled** policy — **Restrict Copilot to data residency models** (US and EU regions), GA since April 13, 2026, off by default — rather than something DRUS turns on automatically. See [5️⃣ Copilot Inference Geography](#5️⃣-copilot-inference-geography).

> 📌 **Advanced Security naming:** On GHEC / GHE.com, Advanced Security was repackaged in 2025 into two standalone products — **GitHub Secret Protection** (secret scanning + push protection) and **GitHub Code Security** (code scanning / CodeQL). "GitHub Advanced Security (GHAS)" as one SKU is now legacy for GHEC and remains the bundle name only on **GHES**.

---

## 3️⃣ Common Misconceptions

| Misconception | Reality |
|--------------|---------|
| "Standard GHEC defaults to US data residency" | Standard GHEC is primarily US-hosted but this is NOT a formal data residency guarantee |
| "DRUS = FedRAMP Moderate" | DRUS is data residency, not an authorization level change |
| "DRUS = GCC High equivalent" | DRUS is not equivalent to Azure GCC High or IL4/IL5 |
| "DRUS makes Copilot US-only" | Copilot inference geography is not automatic with DRUS, but it IS controllable on GHE.com via the **Restrict Copilot to data residency models** policy (US/EU, GA Apr 2026) — an opt-in admin setting, off by default |
| "We can switch from standard to DRUS with a toggle" | Moving to DRUS requires a full migration to a new GHE.com enterprise |

---

## 4️⃣ Migration Implications

**👤 Roles:** **Enterprise owner** (new GHE.com enterprise), **IdP administrator** (SSO and SCIM), migration operator (GEI), integration owners · **📍 Portals:** GHE.com, your IdP, GitHub CLI

Moving from standard GHEC to DRUS requires:
1. New enterprise provisioned on GHE.com
2. Reconfigure IdP (SAML/OIDC + SCIM) for new enterprise
3. Migrate repositories using GitHub Enterprise Importer
4. Update ALL integrations: API endpoints, OIDC issuers, package registry URLs, webhook URLs
5. Update OIDC trust: `https://token.actions.SUBDOMAIN.ghe.com` replaces `https://token.actions.githubusercontent.com` — and expect new **subject** values, because migrated repositories are new repositories and use the immutable `repo:OWNER@ID/REPO@ID:…` format (since July 15, 2026)

> 💡 **Tip:** Plan this as a migration project with 4-8 weeks timeline, not a settings change.

---

## 5️⃣ Copilot Inference Geography

| Topic | What Public Docs Say |
|-------|---------------------|
| Platform data storage | DRUS stores covered data in the US |
| Copilot inference location | NOT pinned to the US by DRUS alone — DRUS covers platform data storage, not inference routing |
| Copilot data residency (US & EU) | **Generally available (April 13, 2026)** for GHE.com enterprises. When the **Restrict Copilot to data residency models** policy is on, Copilot routes requests to model endpoints in your enterprise's region, only region-certified models appear, and prompts, responses, logs, and telemetry stay in-region. **Off by default.** Clients from 2025 or later are required. A separate **Restrict Copilot to FedRAMP models** policy limits users to FedRAMP Moderate–certified models. |
| BYOK | Alternative lever: route inference through your chosen provider endpoint — region guarantee comes from THAT provider |
| Best approach | On GHE.com, prefer the native **Restrict Copilot to data residency models** policy for US/EU region locking; use BYOK where you need provider-level control or a region the policy doesn't cover yet. DRUS alone doesn't enable region-locked inference — it's a separate opt-in. |

> 💡 **Enable it:** on the GHE.com enterprise, an enterprise owner sets **Restrict Copilot to data residency models** (and, for US government needs, **Restrict Copilot to FedRAMP models**) in the enterprise's Copilot policies under **AI controls**. The region is your enterprise's GHE.com region — there's no separate geography picker. Model choice is limited to region-certified models, which can lag new GitHub.com releases.

---

## 6️⃣ FedRAMP Positioning

- GitHub Enterprise Cloud has a **FedRAMP Tailored / LI-SaaS (Low) authorization** today (authorized since 2018)
- **In progress:** GitHub announced (Oct 15, 2024) it is **pursuing FedRAMP Moderate** for GitHub Enterprise Cloud. GitHub Enterprise Cloud with data residency (**GHEC-DR**) is on a FedRAMP Moderate authorization path, and as of April 2026 the underlying Copilot model hosts / infrastructure for US government customers are **FedRAMP Moderate authorized**.
- Do NOT equate FedRAMP Moderate with IL4/IL5 — **IL4/IL5 remain out of scope** for FedRAMP Moderate
- Do NOT assume DRUS by itself changes the platform's FedRAMP authorization scope
- For the exact, current authorization scope, route through GitHub's compliance/account team

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
| **SAML test fails with NameID, recipient, audience, or signature errors** | Required SAML attributes, ACS URL, Entity ID, certificate, clock sync, or signing algorithm do not match GitHub requirements. | Compare every SAML value with the GitHub settings page, send a stable email/NameID, use SHA-256 signing, refresh the certificate, and retest before enforcing. |
| **SCIM test connection fails** | Tenant URL, bearer token, SCIM endpoint, token owner, or IdP provisioning mode is incorrect. | Regenerate the SCIM token from the correct GitHub setup/admin account, paste the exact tenant URL, and confirm the IdP provisioning test succeeds before assigning users. |
| **Provisioned users are missing from GitHub** | Users or groups are not assigned to the IdP app, attribute mappings fail, or provisioning cycles have not completed. | Review IdP provisioning logs, fix mapping errors, assign a small pilot group, and wait for the next incremental provisioning cycle. |
| **Azure billing connection fails** | The Azure signer cannot grant tenant consent or does not own the subscription. | Use a subscription owner with tenant consent rights or run the Entra admin consent workflow, then repeat the GitHub Add Azure Subscription flow. |
| **Copilot controls or seats are not visible** | Copilot is not enabled for the enterprise/org, the signed-in user lacks owner/admin permissions, or the plan/add-on is not active. | Verify Copilot plan activation, enable access at the enterprise/org level, and assign seats from the documented access page. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>

### Q: The customer assumes DRUS provides FedRAMP Moderate authorization — is that correct?
**A:** No. DRUS gives formal US data residency for covered GitHub platform data. It doesn't change GitHub's FedRAMP authorization level.

- **Today:** **FedRAMP Tailored / LI-SaaS (Low)**.
- **In progress:** GitHub announced in October 2024 that it's **pursuing FedRAMP Moderate** for GitHub Enterprise Cloud, and GHEC-DR is on that path. Don't assume Moderate is in place — confirm the current scope with GitHub's compliance team.
- **Not equivalent:** don't equate DRUS with GCC High or IL4/IL5.

In short, DRUS answers "where is my data stored?", not "which compliance framework is the platform certified under?"

---

### Q: The customer thinks data residency means Copilot inference stays in the US — is that true?
**A:** Not automatically. Data residency (DRUS) controls where covered GitHub platform data is stored at rest. It doesn't, by itself, keep Copilot inference in the US.

The fix is now built in — **GitHub Copilot data residency** (US and EU) became generally available on April 13, 2026:

- **In-region inference:** on GHE.com, an enterprise owner turns on **Restrict Copilot to data residency models** (off by default). It keeps inference processing and associated data in your enterprise's region.
- **US government:** **Restrict Copilot to FedRAMP models** limits users to FedRAMP Moderate certified models.
- **Cost:** requests under either policy use **10% more AI credits**.
- **Alternative:** bring your own key (BYOK) routes inference through your own provider endpoint with region guarantees.

Key message for customers: in-region inference is a separate opt-in policy, not something DRUS turns on by itself.

---

### Q: The customer says "can we just toggle data residency on for our existing enterprise" — what is the answer?
**A:** No. Moving from standard GHEC (github.com) to DRUS (GHE.com) requires a full migration. You must provision a new enterprise on GHE.com, reconfigure your IdP (SAML/OIDC + SCIM) for the new enterprise, migrate all repositories using GitHub Enterprise Importer, and update every integration: API endpoints, OIDC trust policies, package registry URLs, webhook URLs, and CI/CD pipeline configurations. Plan this as a 4-8 week migration project, not a settings change.

---

### Q: What are the key feature differences between github.com and GHE.com?
**A:** GHE.com (DRUS) matches github.com for most features, but new features often reach github.com first. Key differences:

- URLs change (for example `SUBDOMAIN.ghe.com` instead of `github.com`).
- The Actions OIDC token issuer URL changes.
- Package registry URLs change.
- GHE.com requires EMU — enterprises with personal accounts aren't available.

Check GitHub's data residency feature overview for the current differences.

---

### Q: The customer underestimates the migration effort from standard GHEC to DRUS — how do I set expectations?
**A:** Make clear this is a migration, not a settings change. The work includes:

1. Provisioning a new enterprise.
2. Reconfiguring the IdP completely.
3. Migrating repositories with GEI.
4. Updating all OIDC trust policies (the token issuer URL changes).
5. Updating API integrations (different base URL).
6. Updating package registry references.
7. Updating webhook URLs.
8. Updating CI/CD pipelines.
9. Communicating with and re-onboarding users.

Most organizations need 4–8 weeks with dedicated engineering time. Position it like a cloud migration.

---

### Q: When should we recommend GHES instead of DRUS?
**A:** Recommend GHES when the customer requires: air-gapped or disconnected environments, IL4/IL5/classified workloads, full infrastructure control (own hardware, own network), or when they have an existing on-premises commitment with no cloud option. GHES does not support Copilot or hosted runners, so weigh those trade-offs. If the customer's only requirement is US data residency and they want full SaaS features (including Copilot), DRUS is the better choice.

---

### Q: Can we use DRUS for non-US data residency (e.g., EU)?
**A:** Yes. "DRUS" is just this guide's shorthand for the US region. **GitHub Enterprise Cloud with data residency** (GHEC-DR) is generally available in several regions:

- **EU** (Azure EU regions plus EFTA countries such as Norway and Switzerland)
- **Australia**
- **US**
- **Japan**

More are planned — check GitHub's data residency docs for the latest list. Setup is the same in every region (a dedicated GHE.com subdomain with EMU), but the region is chosen when the enterprise is created and **can't be changed later**.

---

### Q: The customer conflates DRUS with Azure GCC High — how do I clarify?
**A:** They're entirely different offerings:

- **Azure GCC High** is a US government Azure cloud that meets IL4/IL5 requirements.
- **DRUS** is GitHub's data residency offering. It stores covered data in the US on GitHub's own infrastructure — not in GCC High.

GitHub's cloud authorization is moving: GitHub Enterprise Cloud is **pursuing FedRAMP Moderate** (announced October 2024), and GHEC-DR is on that path. But **IL4/IL5 are out of scope for FedRAMP Moderate**.

If the customer truly needs IL4/IL5 today, the practical option is **GHES** inside their own FedRAMP-authorized boundary or GCC High environment. Confirm the current authorization scope with GitHub's compliance or account team before ruling cloud options in or out.

</details>

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
| Copilot data residency (US/EU) GA announcement | [github.blog](https://github.blog/changelog/2026-04-13-copilot-data-residency-in-us-eu-and-fedramp-compliance-now-available/) |
| GitHub pursuing FedRAMP Moderate (Oct 2024) | [github.com/newsroom](https://github.com/newsroom/press-releases/github-to-pursue-fedramp-moderate) |

---

*Last updated: October 2026*
