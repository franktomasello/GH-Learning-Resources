# 🌐 Data Residency Decision Guide

> Decision framework for choosing between standard GitHub Enterprise Cloud, Data Residency (DRUS/GHE.com), and GitHub Enterprise Server

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
