# 📖 Glossary — Plain-English Terms Used in These Guides

> **Quick definitions for the acronyms and product terms that come up in customer conversations.** Each entry says what the thing is and why it matters.

---

## 📑 Contents

- [Accounts, identity, and sign-in](#accounts-identity-and-sign-in)
- [Roles and permissions](#roles-and-permissions)
- [Enterprise structure](#enterprise-structure)
- [Billing and licensing](#billing-and-licensing)
- [GitHub Copilot](#github-copilot)
- [Security](#security)
- [GitHub Actions and automation](#github-actions-and-automation)
- [Governance and compliance](#governance-and-compliance)
- [Migration](#migration)
- [Microsoft Azure and Entra](#microsoft-azure-and-entra)

---

## Accounts, identity, and sign-in

| Term | What it means | Why it matters |
|------|---------------|----------------|
| **GHEC** (GitHub Enterprise Cloud) | GitHub's enterprise plan, hosted by GitHub on github.com | The plan almost every guide here assumes |
| **GHE.com** / **data residency** | GitHub Enterprise Cloud hosted in a specific region (for example US or EU) at `SUBDOMAIN.ghe.com` | Keeps covered data in-region. Always uses EMU. Some features differ |
| **DRUS** | Shorthand used in these guides for GitHub Enterprise Cloud with **d**ata **r**esidency in the **US** | Same as GHE.com in the US region |
| **GHES** (GitHub Enterprise Server) | The self-hosted version of GitHub that a customer runs on their own servers | Different product; most click paths here don't apply |
| **EMU** (Enterprise Managed Users) | An enterprise where the company's identity provider creates and controls every user account | Strong control and offboarding; users can't contribute to public open source with that account |
| **Standard GHEC** / **personal accounts** | An enterprise where people use their own GitHub.com accounts, optionally protected by SSO | Easier for open source; less central control |
| **Managed user** | A user account in an EMU enterprise (username ends in `_shortcode`) | Created and removed by the IdP, not by the user |
| **Shortcode** | The short string (3–8 letters or numbers) GitHub adds to every EMU username, for example `jsmith_contoso` | Identifies the enterprise. Chosen when the enterprise is created (random on GHE.com) and can't be changed later |
| **Setup user** | The special first admin account in a new EMU enterprise (`SHORTCODE_admin`) | Used only to configure SSO and SCIM; store its password and recovery codes safely |
| **IdP** (identity provider) | The system that signs people in — for example Microsoft Entra ID, Okta, PingFederate, OneLogin, or AD FS | The source of truth for who can access GitHub |
| **SSO** (single sign-on) | Signing in to GitHub through the company IdP instead of a separate password | Required for EMU; optional but common for standard GHEC |
| **SAML** | A standard protocol for SSO, supported by most IdPs | The most common way to connect an IdP to GitHub |
| **OIDC** (OpenID Connect) | A newer sign-in standard. For EMU it's supported with Entra ID and enables Conditional Access | Also used separately by GitHub Actions to log in to clouds without stored secrets |
| **SCIM** | A standard that lets the IdP create, update, and remove GitHub accounts and groups automatically | Powers automatic onboarding and offboarding |
| **CAP** (Conditional Access Policy) | Entra ID rules such as "only from managed devices" or "only from these locations" | Enforced on GitHub when EMU uses OIDC with Entra ID |
| **UPN** (User Principal Name) | A user's sign-in name in Entra ID, usually their work email | Used to match Visual Studio subscriptions and to build EMU usernames |
| **Recovery codes** | One-time codes that let an owner sign in if SSO breaks | Download them right after setting up SSO and store them securely |
| **Proof of presence** | An enterprise setting that makes people re-authenticate (optionally with MFA) at the IdP before high-impact actions | Public preview, Entra ID only |

---

## Roles and permissions

| Term | What it means |
|------|---------------|
| **Enterprise owner** | Full control of the enterprise: policies, billing, SSO, organizations |
| **Billing manager** | Manages billing, budgets, and usage reports — without other admin rights |
| **Organization owner** | Full control of one organization: members, teams, settings |
| **Security manager** | Organization role that manages security settings and alerts across all repositories |
| **Repository administrator** | Full control of a single repository's settings |
| **Write / Maintain / Triage / Read** | Standard repository roles, from "can push code" down to "can only view" |
| **Guest collaborator** | An EMU role for contractors and vendors: internal repositories are hidden except in organizations where they're members |
| **Outside / repository collaborator** | Someone given access to specific repositories without joining the organization |
| **Unaffiliated user** | Someone in the enterprise who isn't in any organization — they don't consume a GitHub Enterprise license (useful for Copilot-only users) |
| **Migrator role** | Lets a non-owner run GitHub Enterprise Importer migrations into an organization |

---

## Enterprise structure

| Term | What it means |
|------|---------------|
| **Enterprise** | The top-level account that holds organizations, policies, and billing |
| **Organization (org)** | A group of repositories, members, and teams inside the enterprise |
| **Team** | A group of organization members used to grant repository access; teams can be nested |
| **Enterprise team** | A team defined at the enterprise level that can span organizations — used for Copilot licensing, roles, and cost centers |
| **Internal repository** | Visible to all enterprise members (but not the public) |
| **Custom properties** | Structured labels on repositories (for example `data-classification`) that rulesets and policies can target |
| **Slug** | The short name in a URL, for example `github.com/enterprises/SLUG` |

---

## Billing and licensing

| Term | What it means |
|------|---------------|
| **License / seat** | One person's paid access to a product (GitHub Enterprise, Copilot Business, and so on) |
| **Usage-based (metered) billing** | Paying for what is actually used (Actions minutes, AI credits, storage) on top of licenses |
| **Cost center** | A billing group (organizations, repositories, users, or enterprise teams) so spending can be charged to a department |
| **Budget** | A spending limit with alerts at 75%, 90%, and 100%; can optionally **stop usage** at the limit |
| **Azure subscription billing** | Paying for GitHub's metered usage through a Microsoft Azure subscription (appears on the Azure invoice) |
| **MACC** | Microsoft Azure Consumption Commitment — GitHub usage billed through Azure counts toward it |
| **Visual Studio subscription with GitHub Enterprise** | A Microsoft bundle that includes a GitHub Enterprise license for each subscriber |

---

## GitHub Copilot

| Term | What it means |
|------|---------------|
| **Copilot Business / Copilot Enterprise** | The two company plans for Copilot ($19 and $39 per user per month) |
| **AI credits** | Since June 1, 2026, how Copilot usage is measured (1 credit = $0.01). Each plan includes a monthly pool per user |
| **AI controls** | The enterprise tab where Copilot, agent, and MCP policies live |
| **Copilot cloud agent** | Copilot working on its own in a GitHub-hosted environment and opening a pull request (formerly "coding agent") |
| **MCP** (Model Context Protocol) | A standard way to give Copilot extra tools and data sources (for example Sentry or Azure) |
| **BYOK** (bring your own key) | Using your own AI provider account (for example Microsoft Foundry or OpenAI) for Copilot models |
| **Content exclusion** | Rules that keep specific files or repositories out of Copilot's context |
| **Custom instructions** | Standing guidance for Copilot (repository, path, or organization level), for example coding standards |
| **Copilot Spaces** | Shared collections of repos, files, issues, and notes that ground Copilot's answers |
| **Default policy for new features** | Decides whether unconfigured Copilot features turn on automatically (applies from October 22, 2026) |

---

## Security

| Term | What it means |
|------|---------------|
| **GHAS** (GitHub Advanced Security) | GitHub's security suite. On GHEC it's now sold as two products: **Secret Protection** and **Code Security** |
| **Secret Protection** | Secret scanning (finds leaked credentials) plus **push protection** (blocks them before they're pushed) |
| **Code Security** | Code scanning with **CodeQL**, dependency review, and related features |
| **CodeQL** | GitHub's code analysis engine that finds security vulnerabilities |
| **Default setup / advanced setup** | Two ways to run CodeQL: automatic (GitHub configures it) or a workflow file you customize |
| **Copilot Autofix** | AI-suggested fixes for code scanning alerts (doesn't need a Copilot license) |
| **Security configuration** | A saved bundle of security settings applied to many repositories at once |
| **Secret risk assessment** | A free, one-off scan of an organization that counts leaked secrets |
| **Security and quality tab** | The organization or repository tab (formerly "Security") that shows alerts, coverage, and assessments |

---

## GitHub Actions and automation

| Term | What it means |
|------|---------------|
| **GitHub Actions** | GitHub's built-in CI/CD (workflows defined in `.github/workflows/`) |
| **Runner** | The machine that runs a workflow job — GitHub-hosted, **larger** (bigger GitHub-hosted), or **self-hosted** (your own) |
| **Runner group** | Controls which organizations and workflows can use a set of runners |
| **ARC** (Actions Runner Controller) | Runs self-hosted runners on Kubernetes and scales them automatically |
| **GitHub App** | An integration with its own identity and short-lived tokens — the recommended way to automate without a user seat |
| **PAT** (personal access token) | A token tied to one user. **Classic** PATs use broad scopes; **fine-grained** PATs are limited to chosen repositories and permissions |
| **GITHUB_TOKEN** | The automatic token each workflow run gets for its own repository |
| **OIDC federation (Actions)** | Workflows log in to Azure, AWS, or GCP with a short-lived token instead of a stored secret |
| **Workflow execution protections** | Rules for who can trigger which workflows and with which events |

---

## Governance and compliance

| Term | What it means |
|------|---------------|
| **Ruleset** | Rules for branches or tags (for example "require a pull request review") applied at repository, organization, or enterprise level |
| **Branch protection rule** | The older, single-repository way to protect a branch |
| **Bypass list** | Who may skip a ruleset (always, or only through a pull request) |
| **CODEOWNERS** | A file that names who must review changes to specific paths |
| **Audit log** | The record of who did what in the enterprise (180 days in the UI) |
| **Audit log streaming** | Sending audit events continuously to a SIEM or storage account for long-term retention |
| **SIEM** | A security event platform such as Splunk, Microsoft Sentinel, or Datadog |
| **IP allow list** | Restricts access to the enterprise to approved IP address ranges |

---

## Migration

| Term | What it means |
|------|---------------|
| **GEI** (GitHub Enterprise Importer) | GitHub's tool for migrating repositories (and some metadata) from Azure DevOps, Bitbucket, GitLab, or another GitHub |
| **ado2gh / bbs2gh / gl2gh / gei** | The GitHub CLI extensions GEI uses for each source |
| **Actions Importer** | Converts pipelines from other CI tools into GitHub Actions workflows |
| **Mannequin** | A placeholder user that holds migrated activity until it's linked ("reclaimed") to a real GitHub user |
| **Mirror push** | Copying a Git repository with `git push --mirror` — code and history only, no pull requests or issues |

---

## Microsoft Azure and Entra

| Term | What it means |
|------|---------------|
| **Microsoft Entra ID** | Microsoft's identity service (formerly Azure Active Directory) |
| **Enterprise application** | The IdP's record of GitHub (for example "GitHub Enterprise Managed User") used for SSO and SCIM |
| **App registration / service principal** | An identity in Entra used by software, such as a GitHub Actions deployment |
| **Federated credential** | A trust rule on an app registration that accepts GitHub's OIDC tokens for a specific repository, branch, or environment |
| **Global Administrator / Application Administrator** | Entra admin roles that can grant consent and manage enterprise applications |
| **Azure RBAC** | Azure's role-based access control (for example **Contributor** on a resource group) |

---

*Last updated: October 2026*
