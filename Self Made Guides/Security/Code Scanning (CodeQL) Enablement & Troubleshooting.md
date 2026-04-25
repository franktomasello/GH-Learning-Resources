# 🔍 GitHub Code Scanning (CodeQL) Enablement & Troubleshooting Runbook

> **Complete guide to enabling CodeQL code scanning across repositories and organizations, choosing query suites, enabling Copilot Autofix, and troubleshooting common issues**

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Single repo (default setup):** `Repo → Settings → Code security → Code scanning → Set up → Default`
- **Org-wide enablement:** `Org → Settings → Code security → Code scanning → Enable for all repositories`
- **Switch to Extended suite:** `Org → Settings → Code security → Global settings → CodeQL → Query suite → Extended`
- **Enable Copilot Autofix:** `Org → Settings → Code security → Code scanning → Copilot Autofix → Enable`

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
| GitHub Advanced Security (GHAS) or GitHub Code Security license | ☐ |
| Org owner or repo admin role | ☐ |
| Repository contains a CodeQL-supported language (C/C++, C#, Go, Java, Kotlin, JS/TS, Python, Ruby, Swift) | ☐ |
| For Copilot Autofix: Copilot license at the org level | ☐ |

---

## 📋 Overview

This runbook covers how to enable and configure CodeQL-based code scanning, choose the right query suite, enable AI-powered autofix, and troubleshoot when results are missing.

| Method | Scope | Best For |
|--------|-------|----------|
| **Default setup (repo)** | Single repository | Quick enablement, recommended starting point |
| **Workflow file** | Single repository | Custom configuration, advanced control |
| **Org-wide enablement** | All repos in org | Organization rollouts |
| **Extended query suite** | Global settings | Broader vulnerability coverage |
| **Copilot Autofix** | Org-wide | AI-suggested fixes for findings |

---

## 1️⃣ Enable Default Setup for a Single Repository (Recommended)

**Navigation:**

```
Repository → Settings → Code security (sidebar)
  → Code scanning → Set up → Default
```

> ✅ **Result:** CodeQL analysis is configured automatically based on detected languages. Scans run on push to default branch and on pull requests.

> 💡 **Tip:** Default setup is the fastest path to results. GitHub detects the languages in your repo and configures everything automatically -- no workflow file needed.

---

## 2️⃣ Enable via Workflow File (Advanced)

For repositories that need custom configuration, add a CodeQL workflow file manually.

**Steps:**

1. Create the file `.github/workflows/codeql.yml` in your repository
2. Use the CodeQL starter workflow template from GitHub
3. Customize languages, query suites, and triggers as needed

> 💡 **Tip:** Use the workflow approach when you need to customize build steps, add additional queries, or configure matrix builds across multiple languages.

---

## 3️⃣ Enable Code Scanning Org-Wide

**Navigation:**

```
Organization → Settings → Code security (sidebar)
  → Code scanning → Enable for all repositories
```

> ✅ **Result:** Default setup is applied to all eligible repositories in the organization.

> ⚠️ **Important:** Enabling at the org level applies default setup. Repositories that already have a custom CodeQL workflow will not be overwritten.

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

**Navigation:**

```
Organization → Settings → Code security (sidebar)
  → Global settings → CodeQL analysis → Query suite
    → Select "Extended"
```

> ✅ **Result:** All repositories using default setup in the organization will use the Extended query suite on their next scan.

> ⚠️ **Warning:** Switching to Extended will likely increase the number of alerts. Ensure your team has a triage process in place before making this change.

---

## 6️⃣ Enable Copilot Autofix for Code Scanning

Copilot Autofix uses AI to suggest fixes for CodeQL findings directly in pull requests.

**Navigation:**

```
Organization → Settings → Code security (sidebar)
  → Code scanning → Copilot Autofix → Enable
```

> ✅ **Result:** When CodeQL finds a vulnerability in a pull request, Copilot Autofix will suggest a code fix that the developer can review and apply.

> 💡 **Tip:** Copilot Autofix significantly reduces remediation time. Developers can accept, modify, or dismiss the suggested fix directly in the PR.

---

## 7️⃣ Troubleshooting: Zero Results Despite Many Repos Enabled

If you have enabled code scanning across your organization but see no (or very few) results, check these six common causes:

| # | Cause | How to Check | Fix |
|---|-------|-------------|-----|
| 1 | **Scanning never configured** | Enabling GHAS/Code Security does NOT start scans. Check if repos have default setup or a workflow file. | Enable default setup or add a CodeQL workflow to each repo |
| 2 | **Language not supported** | CodeQL only supports specific languages (see table below). Repos with only unsupported languages will produce no results. | Verify repo languages against the supported list |
| 3 | **No triggering events since setup** | Default setup runs on push to default branch and PRs. If no code has been pushed since enablement, no scan has run. | Push a commit or manually trigger the workflow |
| 4 | **Workflow failing silently** | CodeQL workflow runs may be failing without visible alerts. | Check the **Actions** tab in each repo for failed runs |
| 5 | **Results hidden by filters** | The Security tab may have filters applied (branch, severity, tool) that hide existing results. | Reset all filters on the code scanning alerts page |
| 6 | **Org-level default setup not propagated** | Org-wide enablement may not have reached all repos, especially those with existing configurations. | Check individual repo settings to confirm scanning is active |

> 💡 **Tip:** The fastest way to verify scanning status is to check the **Security** tab at the organization level and look at the "Coverage" view to see which repos have code scanning enabled and running.

---

## 8️⃣ Supported Languages

CodeQL supports the following languages for analysis:

| Language | Notes |
|----------|-------|
| **C / C++** | Requires build steps in advanced setup |
| **C#** | .NET framework and .NET Core |
| **Go** | Full support |
| **Java** | Includes Gradle and Maven projects |
| **Kotlin** | Full support |
| **JavaScript** | Includes JSX |
| **TypeScript** | Includes TSX |
| **Python** | Full support |
| **Ruby** | Full support |
| **Swift** | Full support |

> ⚠️ **Important:** If a repository contains only languages not in this list (e.g., Rust, PHP, Perl), CodeQL will not produce any results for that repository.

## 🧯 Known Errors & Resolutions

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

---

## ❓ Common Questions & Troubleshooting

### Q: Code scanning is enabled across our org but we see zero alerts. What should we check?
**A:** Walk through the six common causes in Section 7 of this guide: (1) scanning may not be configured despite GHAS being enabled, (2) repos may only contain unsupported languages, (3) no code has been pushed since enablement to trigger a scan, (4) CodeQL workflows may be failing silently, (5) alert filters in the Security tab may be hiding results, and (6) org-level default setup may not have propagated to all repos. Check the org-level Security tab "Coverage" view first.

---

### Q: My CodeQL workflow is failing on a compiled language like C++ or Java. What is going wrong?
**A:** CodeQL requires a successful build to analyze compiled languages. If the default setup cannot build your code, switch to the advanced (workflow file) setup and add the necessary build steps (e.g., `mvn compile` for Java or `cmake && make` for C++). The workflow must produce compiled artifacts for CodeQL to analyze.

---

### Q: We are getting too many false positives from code scanning. How do we reduce noise?
**A:** If you are using the Extended query suite, switch back to the Default suite, which is curated for high-confidence findings with a low false-positive rate. You can also dismiss individual alerts as "false positive" or "won't fix" to clean up the alert list. For recurring false positives, consider adding inline suppression comments or configuring query filters in your CodeQL configuration.

---

### Q: Copilot Autofix is enabled but it is not generating fix suggestions for our findings. Why?
**A:** Verify that your organization has a GitHub Advanced Security (GHAS) license and that Copilot Autofix is enabled at the org level (Settings > Code security > Copilot Autofix). Also check that the language of the finding is supported by Copilot Autofix -- not all CodeQL-supported languages have Autofix coverage yet. Autofix suggestions only appear on pull request findings, not on default-branch scan results.

---

### Q: Code scanning results are not showing up on pull requests, only on the default branch. How do I fix this?
**A:** Verify that your CodeQL workflow includes `pull_request` as a trigger event. The default setup includes this automatically, but if you are using a custom workflow file, you must explicitly add `on: pull_request` targeting the appropriate branches. Without this trigger, scans only run on push to the default branch and results will not appear inline on PRs.

---

### Q: How do I exclude test files or generated code from code scanning results?
**A:** In a custom workflow file, use `paths-ignore` in the workflow trigger to skip scanning on pushes that only change test files. For more granular control, create a CodeQL configuration file (`.github/codeql/codeql-config.yml`) and define `paths-ignore` patterns to exclude directories like `**/test/**` or `**/generated/**` from analysis.

---

### Q: Can I run CodeQL on languages not in the supported list, like Rust or PHP?
**A:** CodeQL does not support those languages natively. For unsupported languages, you can integrate third-party SARIF-compatible scanning tools (e.g., Semgrep, Snyk) that upload results to the GitHub code scanning API. These results will appear alongside any CodeQL findings in the Security tab.

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Secret Protection Enablement | `Security/Secret Protection Enablement.md` |
| Responsible AI Guardrails | `Copilot/Responsible AI Guardrails.md` |
| Branch Protection Rules & Rulesets | `Governance/Branch Protection Rules & Rulesets.md` |
| Copilot Coding Agent & MCP Configuration | `Copilot/Coding Agent & MCP Configuration.md` |

---

## 📚 Resources

- [About code scanning with CodeQL](https://docs.github.com/en/code-security/code-scanning/introduction-to-code-scanning/about-code-scanning-with-codeql)
- [Built-in CodeQL query suites](https://docs.github.com/en/code-security/code-scanning/managing-your-code-scanning-configuration/built-in-codeql-query-suites)
- [Troubleshooting code scanning](https://docs.github.com/en/code-security/code-scanning/troubleshooting-code-scanning)

---

*Last updated: April 2026*
