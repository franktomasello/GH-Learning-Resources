# 🔍 GitHub Code Scanning (CodeQL) Enablement & Troubleshooting Runbook

> **Complete guide to enabling CodeQL code scanning across repositories and organizations, choosing query suites, enabling Copilot Autofix, and troubleshooting common issues**

| 🧭 **At a Glance** | |
|---|---|
| **Goal** | Turn on CodeQL code scanning and fix common problems |
| **Use this when** | Enabling Code Security, or scans show no results |
| **People you need** | Repository admins; organization owners or security managers |
| **Where you click** | GitHub (repo and org settings) |
| **End result** | Code scanning running across repositories, with Autofix and a coverage view |
| **New to a term?** | See the [Glossary](../Glossary.md) for plain-English definitions |

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [📋 Overview](#-overview)
- [1️⃣ Enable Default Setup for a Single Repository (Recommended)](#1️⃣-enable-default-setup-for-a-single-repository-recommended)
- [2️⃣ Enable via Workflow File (Advanced Setup)](#2️⃣-enable-via-workflow-file-advanced-setup)
- [3️⃣ Enable Code Scanning Org-Wide](#3️⃣-enable-code-scanning-org-wide)
- [4️⃣ Default vs Extended Query Suites](#4️⃣-default-vs-extended-query-suites)
- [5️⃣ Switch to Extended Query Suite](#5️⃣-switch-to-extended-query-suite)
- [6️⃣ Enable Copilot Autofix for Code Scanning](#6️⃣-enable-copilot-autofix-for-code-scanning)
- [7️⃣ Troubleshooting: Zero Results Despite Many Repos Enabled](#7️⃣-troubleshooting-zero-results-despite-many-repos-enabled)
- [8️⃣ Supported Languages](#8️⃣-supported-languages)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📚 Resources](#-resources)

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Single repo (default setup):** `Repo → Settings → Advanced Security` (under "Security and quality") → **CodeQL analysis** → **Set up ▾** → **Default** → **Enable CodeQL**
- **Org-wide:** `Org → Settings → Advanced Security ▾ → Configurations` → **New configuration** → **Custom configuration** → Code Security + **Default setup** → **Save configuration** → **Repositories** tab → select repos → **Apply configuration ▾** → **Apply**
- **Recommend the Extended suite:** `Org → Settings → Advanced Security ▾ → Global settings` → **Recommend the extended query suite for repositories enabling default setup**
- **Copilot Autofix:** `Org → Settings → Advanced Security ▾ → Global settings` → **Copilot Autofix**
- **Check coverage:** `Org → Security and quality tab → Coverage`

---

## ✅ Accuracy & Click-Path Notes

<details>
<summary><em>Show click-path conventions</em></summary>

- Reviewed against current public GitHub documentation in October 2026. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
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
| **GitHub Code Security** (or GitHub Advanced Security) for private and internal repositories — public repositories don't need it | Enterprise owner / billing | ☐ |
| **GitHub Actions** enabled for the repositories | Organization owner / repo admin | ☐ |
| Single-repository setup (Sections 1, 2, 5) | **Repository administrator** | ☐ |
| Organization-wide setup and global settings (Sections 3, 5, 6) | **Organization owner** or **security manager** | ☐ |
| At least one CodeQL-supported language in the repository (Section 8) | — | ☐ |
| Copilot Autofix | Nothing extra — it does **not** need a Copilot license | ☐ |

---

## 📋 Overview

This runbook covers how to enable and configure CodeQL code scanning, choose a query suite, turn on Copilot Autofix, and troubleshoot missing results.

| Method | Scope | Best for |
|--------|-------|----------|
| **Default setup** | One repository | Fastest start — GitHub picks languages and settings |
| **Advanced setup (workflow file)** | One repository | Custom build steps, extra queries, matrix builds |
| **Security configuration** | Many or all repositories in an organization | Organization rollouts |
| **Extended query suite** | Per repository, or recommended org-wide | Broader coverage |
| **Copilot Autofix** | Organization-wide (global setting) | AI-suggested fixes for alerts |

> 📌 **UI note:** on GitHub.com the repository settings section that holds **Advanced Security** is labeled **Security and quality**, and the organization/repository **Security** tab is now **Security and quality**. In **organization** settings the section is still labeled **Security**.

---

## 1️⃣ Enable Default Setup for a Single Repository (Recommended)

**👤 Role:** **Repository administrator** · **📍 Portal:** GitHub

**Navigate:** Repository → **Settings** → **Advanced Security** *(under "Security and quality")*

**Steps:**

1. In the sidebar, under **Security and quality**, click **Advanced Security**.
2. Under **Code Security**, next to **CodeQL analysis**, click **Set up ▾**, then **Default**.
3. *(Optional)* In the **CodeQL default configuration** dialog, click **Edit** to change **Languages** or the **Query suites** selection.
4. Click **Enable CodeQL**.

> ✅ **Result:** GitHub runs an initial analysis, then scans on pushes to the default branch, on pull requests, and on a weekly schedule.

> 💡 **Tip:** If your code uses private package registries, grant code scanning access to them for better results. If a repository has had no pushes or pull requests for 6 months, its weekly scan is paused (see Section 7).

---

## 2️⃣ Enable via Workflow File (Advanced Setup)

**👤 Role:** **Repository administrator** · **📍 Portal:** GitHub

Use advanced setup when you need custom build steps, extra queries, or matrix builds.

**Steps:**

1. Repository → **Settings** → **Advanced Security**.
2. In the **CodeQL analysis** row, click **Set up ▾**, then **Advanced**. *(Already on default setup? Click **⋯** → **Switch to advanced** → **Disable CodeQL** first.)*
3. Review the generated `.github/workflows/codeql.yml` and edit languages, build steps, and queries as needed.
4. Click **Commit changes...**, enter a commit message, and choose a branch.
5. Click **Commit new file** (default branch) or **Propose new file** (new branch + pull request).

> 💡 **Tip:** Advanced setup uses Actions minutes like any workflow. For compiled languages, add explicit build steps if autobuild can't build the project.

---

## 3️⃣ Enable Code Scanning Org-Wide

**👤 Role:** **Organization owner** or **security manager** · **📍 Portal:** GitHub

**Navigate:** Organization → **Settings** → in the **Security** section, **Advanced Security ▾** → **Configurations**

### Step A — Create a configuration

1. Click **New configuration**, then **Custom configuration**.
2. Enter a **name** and **description** (for example `codeql-default`).
3. Turn on **Code Security**, and set **Default setup** to **Enabled** — or **Enabled with advanced setup allowed**, so repositories that already run their own CodeQL workflow keep it.
4. *(Optional)* Set the other features you want (secret scanning, dependency scanning, and so on).
5. *(Optional)* Under **Policy**, set **Use as default for newly created repositories** and/or **Enforce configuration**.
6. Click **Save configuration**.

### Step B — Apply it to repositories

1. On the **Configurations** page, click the **Repositories** tab.
2. Filter the table if needed, then select repositories — or select the page checkbox and click **Select all**.
3. Click **Apply configuration ▾** and choose your configuration.
4. Review the license-consumption summary, then click **Apply**.

> ✅ **Result:** each repository shows a status such as `attached`, `attaching`, or `failed`.

> ⚠️ **Important:** a default configuration is only applied automatically to **new** repositories. Repositories transferred into the organization need a configuration applied by hand.

> 📌 **Enterprise enforcement (September 2026):** enterprise-level security configurations have an **Enforcement** dropdown — **Don't enforce**, **Enforce for repository owners**, or **Enforce for repository and organization owners** — so organization admins can't override enterprise settings.

---

## 4️⃣ Default vs Extended Query Suites

CodeQL ships with two built-in query suites. Choosing the right one balances precision against coverage.

| Aspect | Default Suite | Extended Suite |
|--------|--------------|----------------|
| **Focus** | High-confidence CWE/OWASP findings | Broader vulnerability coverage |
| **False positive rate** | Low -- curated for precision | Higher -- trades precision for breadth |
| **Best for** | Teams new to code scanning, production gating | Security teams wanting maximum coverage |
| **Query count** | Smaller, high-signal set | Larger set including lower-severity queries |
| **Recommended use** | PR blocking, developer workflow | Security audits, triage-capable teams |

> 💡 **Tip:** Start with the Default suite. Once your team is comfortable triaging alerts, consider switching to Extended for broader coverage.

---

## 5️⃣ Switch to Extended Query Suite

**Per repository** — **👤 Role:** **Repository administrator**

1. Repository → **Settings** → **Advanced Security**.
2. In the **CodeQL analysis** row, click **⋯** → **View CodeQL configuration**.
3. Click **Edit**.
4. In the **Query suite** row under **Scan settings**, select **Extended**.
5. Click **Save changes**. A new analysis runs with the new suite.

**Organization-wide recommendation** — **👤 Role:** **Organization owner** or **security manager**

1. Organization → **Settings** → **Advanced Security ▾** → **Global settings**.
2. Under code scanning, select **Recommend the extended query suite for repositories enabling default setup**.

> 📌 The org setting **recommends** Extended to repositories as they enable default setup. It doesn't switch repositories that already run default setup — change those per repository.

> ⚠️ **Warning:** Extended will likely increase the number of alerts. Make sure your team has a triage process first.

---

## 6️⃣ Enable Copilot Autofix for Code Scanning

**👤 Role:** **Organization owner** or **security manager** · **📍 Portal:** GitHub

Copilot Autofix uses AI to suggest fixes for code scanning alerts — in pull requests and for existing alerts on the default branch.

**Navigate:** Organization → **Settings** → **Advanced Security ▾** → **Global settings**

**Steps:**

1. Under code scanning, select **Copilot Autofix**. It applies to repositories using CodeQL default or advanced setup.

> ✅ **Result:** when CodeQL finds an alert, Copilot Autofix can propose a fix the developer reviews, edits, and commits.

> 📌 Copilot Autofix **doesn't require a GitHub Copilot subscription**. It needs code scanning with CodeQL (Code Security for private repositories).

> 💡 **Also on Global settings:** **AI Scan** (AI-powered detections for eligible repositories — it no longer requires CodeQL default setup, and its effective status per repository appears in the **Coverage** view) and **Keep scheduled scans running every 30 days for inactive repositories**.

---

## 7️⃣ Troubleshooting: Zero Results Despite Many Repos Enabled

If code scanning looks enabled across your organization but you see few or no results, check these common causes:

| # | Cause | How to check | Fix |
|---|-------|-------------|-----|
| 1 | **Scanning never configured** | Buying Code Security or GHAS doesn't start scans. Check the **Coverage** view. | Apply a security configuration with default setup, or add a workflow |
| 2 | **GitHub Actions disabled** | Default setup needs Actions. Check Org/Repo → **Settings** → **Actions**. | Allow Actions for the repositories |
| 3 | **Language not supported** | Repos with only unsupported languages produce no CodeQL results (Section 8). | Use a third-party SARIF tool for those languages |
| 4 | **Analysis failing** | Check the repo's **Actions** tab and the code scanning **tool status** page. If every language fails, default setup stays enabled but doesn't scan. | Fix the build or switch to advanced setup |
| 5 | **Inactive repository** | Weekly scans pause after 6 months with no pushes or pull requests. | Push a change, or turn on **Keep scheduled scans running every 30 days for inactive repositories** |
| 6 | **Results hidden by filters** | Filters (branch, severity, tool) on the alerts page can hide results. | Clear all filters |
| 7 | **Configuration not attached** | The configuration status shows `failed`, or the repo was transferred in after setup. | Re-apply the configuration on the **Repositories** tab |

> 💡 **Tip:** The fastest check is Organization → **Security and quality** tab → **Coverage**. It shows which repositories have each feature enabled.

---

## 8️⃣ Supported Languages

| Language | Notes |
|----------|-------|
| **C / C++** | Compiled — default setup can use build mode `none`; advanced setup may need build steps |
| **C#** | Compiled — build mode `none` in default setup |
| **Go** | |
| **Java / Kotlin** | Analyzed together (`java-kotlin`) |
| **JavaScript / TypeScript** | Analyzed together (`javascript-typescript`) |
| **Python** | |
| **Ruby** | |
| **Rust** | Build mode `none` in default setup |
| **Swift** | Compiled — check runner support before using self-hosted runners |
| **GitHub Actions workflows** | Scans workflow files for security issues |

> ⚠️ **Important:** A repository whose code is only in other languages (for example PHP or Perl) produces no CodeQL results.

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
| **Security setting is missing or disabled** | The repository is not eligible, the required Code Security/Secret Protection/GHAS license is not enabled, or the user lacks admin/security manager permissions. | Verify repository ownership and plan eligibility, enable the required license/add-on at the correct scope, and retry as an org owner, security manager, or repository admin. |
| **Default setup enables but produces no CodeQL alerts** | No supported language was detected, the first analysis failed, or no relevant vulnerable code is present. | Check the code scanning tool status page, confirm supported languages, review the first workflow run, and switch to advanced setup if the build requires custom steps. |
| **Compiled language CodeQL analysis fails** | The autobuild cannot compile the project or dependencies are missing. | Use advanced setup with explicit build commands, install dependencies on the runner, and verify the build succeeds before CodeQL analysis. |
| **Secret scanning misses an expected secret** | The secret format is unsupported, below confidence thresholds, or requires a custom pattern. | Check supported patterns, add a custom pattern for proprietary formats, and test the pattern before applying at scale. |
| **Push protection blocks a legitimate commit** | A supported secret pattern was detected in the pushed diff. | Remove or rotate the secret where appropriate. Use the documented bypass process only for verified false positives or approved test credentials. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>

### Q: Code scanning is enabled across our org but we see zero alerts. What should we check?
**A:** Start with Organization → **Security and quality** → **Coverage** to see which repositories actually have code scanning. Then work through Section 7: scanning never configured, Actions disabled, unsupported languages, failing analyses, inactive repositories, hidden filters, or a configuration that didn't attach.

---

### Q: My CodeQL workflow is failing on a compiled language like C++ or Java. What is going wrong?
**A:** Default setup analyzes C/C++, C#, Java, and Rust with build mode `none`, so it doesn't need to compile them. In advanced setup with `autobuild` or `manual` build mode, CodeQL needs a successful build — add explicit build steps (for example `mvn compile` for Java or `cmake` and `make` for C++) and make sure dependencies install on the runner.

---

### Q: We are getting too many false positives from code scanning. How do we reduce noise?
**A:** If you're on the Extended suite, switch back to Default, which is tuned for high-confidence results. Dismiss individual alerts as **False positive** or **Won't fix** with a reason. For recurring noise in advanced setup, exclude queries or paths with a CodeQL configuration file (`query-filters`, `paths-ignore`).

---

### Q: Copilot Autofix is enabled but it is not generating fix suggestions for our findings. Why?
**A:** Check that **Copilot Autofix** is selected at Organization → **Settings** → **Advanced Security ▾** → **Global settings**, and that the repository uses CodeQL (default or advanced setup) with Code Security enabled for private repositories. Not every alert type and language gets a suggestion. For existing default-branch alerts, open the alert and ask Autofix to generate a fix. A Copilot subscription is **not** required.

---

### Q: Code scanning results are not showing up on pull requests, only on the default branch. How do I fix this?
**A:** In advanced setup, make sure the workflow has an `on: pull_request` trigger for the target branches — default setup includes it automatically. Also check that the analysis on the PR finished, and that the alerts are in lines the pull request changed (only those are annotated on the PR).

---

### Q: How do I exclude test files or generated code from code scanning results?
**A:** In advanced setup, create a CodeQL configuration file (for example `.github/codeql/codeql-config.yml`) with `paths-ignore` patterns such as `**/test/**` or `**/generated/**`, and reference it from the workflow's `init` step. (`paths-ignore` on the workflow trigger only skips runs; it doesn't exclude files from analysis.) For default setup, you can apply a custom configuration file at scale through the `github-codeql-config-file` repository property.

---

### Q: Can I run CodeQL on languages like Rust or PHP?
**A:** Rust is supported now (see Section 8). PHP isn't. For unsupported languages, run a third-party scanner that outputs SARIF and upload the results to code scanning — they appear alongside CodeQL alerts on the **Security and quality** tab.

</details>

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Secret Protection Enablement | `Security/Secret Protection Enablement.md` |
| Responsible AI Guardrails | `Copilot/Responsible AI Guardrails.md` |
| Branch Protection Rules & Rulesets | `Governance/Branch Protection Rules & Rulesets.md` |
| Copilot Cloud Agent & MCP Configuration | `Copilot/Cloud Agent & MCP Configuration.md` |

---

## 📚 Resources

- [Configuring default setup for code scanning](https://docs.github.com/en/code-security/how-tos/find-and-fix-code-vulnerabilities/configure-code-scanning/configure-code-scanning)
- [Configuring advanced setup for code scanning](https://docs.github.com/en/code-security/how-tos/find-and-fix-code-vulnerabilities/configure-code-scanning/configuring-advanced-setup-for-code-scanning)
- [Creating a custom security configuration](https://docs.github.com/en/code-security/how-tos/secure-at-scale/configure-organization-security/establish-complete-coverage/create-custom-configuration)
- [Applying a custom security configuration](https://docs.github.com/en/code-security/how-tos/secure-at-scale/configure-organization-security/establish-complete-coverage/apply-custom-configuration)
- [Configuring global security settings for your organization](https://docs.github.com/en/code-security/how-tos/secure-at-scale/configure-organization-security/establish-complete-coverage/configure-global-settings)
- [Editing your default setup configuration](https://docs.github.com/en/code-security/how-tos/find-and-fix-code-vulnerabilities/manage-your-configuration/edit-default-setup)

---

*Last updated: October 2026*
