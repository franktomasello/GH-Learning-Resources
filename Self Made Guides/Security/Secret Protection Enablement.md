# 🔐 GitHub Secret Protection Enablement Runbook

> **Complete guide to enabling secrets protection across repositories, organizations, and at scale via Security Configurations**

| 🧭 **At a Glance** | |
|---|---|
| **Goal** | Find leaked secrets and block new ones |
| **Use this when** | Enabling Secret Protection, or after a credential leak |
| **People you need** | Repository admins; organization owners or security managers |
| **Where you click** | GitHub (repo and org settings) |
| **End result** | Secret scanning and push protection on the right repositories |
| **New to a term?** | See the [Glossary](../Glossary.md) for plain-English definitions |

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [📋 Overview](#-overview)
- [🔑 What "Secret Protection" Means in GitHub](#-what-secret-protection-means-in-github)
- [1️⃣ Enable Secret Protection for a Single Repository](#1️⃣-enable-secret-protection-for-a-single-repository)
- [2️⃣ Enable Secret Protection at the Organization Level (Guided)](#2️⃣-enable-secret-protection-at-the-organization-level-guided)
- [3️⃣ Enable at Scale Using Security Configurations](#3️⃣-enable-at-scale-using-security-configurations)
- [4️⃣ Enable Push Protection for Your User Account](#4️⃣-enable-push-protection-for-your-user-account)
- [5️⃣ Enable via REST API](#5️⃣-enable-via-rest-api)
- [🚀 Quick "Most Complete" Rollout Recipe](#-quick-most-complete-rollout-recipe)
- [📝 Additional Notes](#-additional-notes)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Single repo:** `Repo → Settings → Advanced Security` (under "Security and quality") → **Secret Protection** → **Enable** → **Enable Secret Protection**
- **Push protection (repo):** same page → **Secret Protection** section → **Push protection** → **Enable**
- **Org-wide (guided):** `Org → Security and quality tab → Assessments` → **Get started ▾** → **For all repositories** → **Enable Secret Protection**
- **At scale:** `Org → Settings → Advanced Security ▾ → Configurations` → **New configuration** → **Custom configuration** → **Save configuration** → **Repositories** tab → **Apply configuration ▾** → **Apply**
- **User-level push protection:** `Profile → Settings → Code security` → **Push protection for yourself**

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
| GitHub Team or GitHub Enterprise Cloud | — | ☐ |
| **GitHub Secret Protection** (or GitHub Advanced Security) for private and internal repositories — public repositories are free | Enterprise owner / billing | ☐ |
| Single-repository enablement (Section 1) | **Repository administrator** | ☐ |
| Organization-wide enablement and configurations (Sections 2–3) | **Organization owner** or **security manager** | ☐ |
| REST API (Section 5) | Repo admin, or org owner / security manager, with a suitable token | ☐ |
| A list of target repositories (public vs private/internal) and a pilot group | Security team | ☐ |

> 💡 Run the free **secret risk assessment** first to size the problem — see `Security/Secret Risk Assessment (Organization-Wide Scan).md`.

---

## 📋 Overview

This runbook covers every practical way to turn on secret protection in GitHub:

| Method | Scope | Best for |
|--------|-------|----------|
| **Repo-by-repo** | One repository | Testing, specific repositories |
| **Guided org enablement (Assessments)** | Public or all repositories in an org | Fast organization rollout |
| **Security configurations** | Selected or all repositories | Controlled, enforceable rollouts |
| **User-level push protection** | Individual user | Personal protection when pushing to public repositories |
| **REST API** | Automation and scripting | Bulk operations |

---

## 🔑 What "Secret Protection" Means in GitHub

| Feature | Description |
|---------|-------------|
| **Secret scanning alerts** | Finds secrets already in the repository — full Git history, plus issues, pull requests, discussions, and wikis |
| **Push protection** | Blocks supported secrets from being pushed; bypasses create alerts |
| **Validity checks** | Checks whether a detected partner token is still active |
| **Non-provider (generic) patterns** | Detects things like private keys and connection strings |
| **AI-detected secrets** | Uses AI to find unstructured secrets such as passwords |
| **Custom patterns** | Your own regular expressions for internal secret formats |
| **Delegated bypass / dismissal** | Require review before bypassing push protection or dismissing alerts |

> 💡 Enabling **Secret Protection** on a repository turns on secret scanning alerts. Push protection is a separate switch in the same section.

### Availability

| Repository type | Availability |
|-----------------|--------------|
| **Public repositories** | Secret scanning is free |
| **Private and internal repositories** | Need **GitHub Secret Protection** (or GitHub Advanced Security) |
| **User-owned repositories (EMU)** | Supported with Enterprise Managed Users |

> 📌 **UI note:** on GitHub.com, the repository settings section is labeled **Security and quality**, and the organization tab is **Security and quality**. In **organization settings**, the section is still **Security**.

---

## 1️⃣ Enable Secret Protection for a Single Repository

**👤 Role:** **Repository administrator** · **📍 Portal:** GitHub

**Navigate:** Repository → **Settings** → **Advanced Security** *(under "Security and quality")*

### A) Turn on Secret Protection (enables secret scanning alerts)

1. In the sidebar, under **Security and quality**, click **Advanced Security**.
2. To the right of **Secret Protection**, click **Enable**.
3. Review the impact, then click **Enable Secret Protection**.

> ✅ **Result:** secret scanning alerts are on, and GitHub scans the repository's history.

### B) Turn on push protection

1. On the same page, in the **Secret Protection** section, to the right of **Push protection**, click **Enable**.

> ✅ **Result:** pushes containing supported secrets are blocked unless bypassed, and each bypass creates an alert.

---

## 2️⃣ Enable Secret Protection at the Organization Level (Guided)

**👤 Role:** **Organization owner** or **security manager** · **📍 Portal:** GitHub

**Navigate:** Organization → **Security and quality** tab → under "Security", **Assessments**

**Steps:**

1. Under your organization name, click the **Security and quality** tab.
2. In the sidebar, under **Security**, click **Assessments**.
3. In the banner, open **Get started ▾** and pick one:

| Option | What happens next |
|--------|-------------------|
| **For public repositories for free** | Enables Secret Protection for public repositories only |
| **For all repositories** | Shows a cost estimate. Click **Enable Secret Protection** to turn on alerts **and** push protection everywhere — or **Configure in settings** to choose repositories |

> 💡 **Tip:** enterprise owners can also turn on **public monitoring** to catch secrets that enterprise members leak in public repositories outside your organizations.

---

## 3️⃣ Enable at Scale Using Security Configurations

*Recommended for controlled organization rollouts*

**👤 Role:** **Organization owner** or **security manager** · **📍 Portal:** GitHub

**Navigate:** Organization → **Settings** → in the **Security** section, **Advanced Security ▾** → **Configurations**

### A) Quick setup

1. Click **New configuration**.
2. In the setup dialog, review the default settings and the selected repositories, and adjust them.
3. Click **Review**, then **Save and enable**.

### B) Create a custom security configuration

1. Click **New configuration**, then **Custom configuration**.
2. Enter a **name** and **description**.
3. Turn on **Secret Protection** (paid for private and internal repositories), then set each option to enabled, disabled, or keep existing:

| Setting | Description |
|---------|-------------|
| **Validity checks** | Check whether detected partner secrets are still valid |
| **Extended metadata** | Extra details about detected secrets (needs validity checks) |
| **Generic patterns** | Non-provider patterns such as private keys and connection strings |
| **Scan for AI-detected secrets** | AI detection of unstructured secrets like passwords |
| **Push protection** | Block secrets from being pushed |
| **Bypass privileges** | Let selected roles or teams bypass; everyone else must request a review |
| **Prevent direct alert dismissals** | Dismissals need a reviewer's approval |

4. *(Optional)* Under **Policy**, set **Use as default for newly created repositories** and/or **Enforce configuration** (**Enforce** blocks repository owners from changing the features you set).
5. Click **Save configuration**.

### C) Apply your configuration to repositories

1. On the **Configurations** page, click the **Repositories** tab.
2. *(Optional)* Filter the repository table.
3. Select repositories one by one, select the whole page with the header checkbox, or select the page and click **Select all** for every matching repository.
4. Click **Apply configuration ▾** and choose your configuration.
5. Review the license-consumption summary, then click **Apply**.

> ⚠️ A default configuration only applies automatically to **new** repositories. Apply it by hand to existing and transferred-in repositories.

> 📌 **Enterprise enforcement:** configurations created at the enterprise level can be set to **Enforce for repository owners** or **Enforce for repository and organization owners**, so neither repo nor org admins can override them.

---

## 4️⃣ Enable Push Protection for Your User Account

*Protects your own pushes to public repositories, separate from org and repo settings*

**👤 Role:** Any user · **📍 Portal:** GitHub

1. Click your profile picture → **Settings**.
2. In the **Security** section of the sidebar, click **Code security**.
3. Under **User**, next to **Push protection for yourself**, enable or disable it.

> 💡 **Note:** It's on by default and stops you from pushing supported secrets to **public** repositories on GitHub.

---

## 5️⃣ Enable via REST API

*Useful for automation and bulk scripting*

### Update a repository's `security_and_analysis`

Call **Update a repository** (`PATCH /repos/{owner}/{repo}`) as a repository admin, or an organization owner or security manager.

| Field | Values |
|-------|--------|
| `secret_scanning.status` | `"enabled"` or `"disabled"` |
| `secret_scanning_push_protection.status` | `"enabled"` or `"disabled"` |
| `secret_scanning_validity_checks.status` | `"enabled"` or `"disabled"` |
| `secret_scanning_non_provider_patterns.status` | `"enabled"` or `"disabled"` |
| `secret_scanning_ai_detection.status` | `"enabled"` or `"disabled"` |
| `secret_scanning_delegated_bypass.status` | `"enabled"` or `"disabled"` |

**Example request body:**

```json
{
  "security_and_analysis": {
    "secret_scanning": { "status": "enabled" },
    "secret_scanning_push_protection": { "status": "enabled" }
  }
}
```

> 📌 The `advanced_security` field is for the bundled GitHub Advanced Security product only — it can't be used with standalone Secret Protection or Code Security. For many repositories, prefer applying a security configuration.

---

## 🚀 Quick "Most Complete" Rollout Recipe

*If your goal is to turn it on everywhere with the most control:*

### Steps

1. **Size the risk:** run the free secret risk assessment (Organization → **Security and quality** → **Assessments**).
2. **Open Configurations:** Organization → **Settings** → **Advanced Security ▾** → **Configurations**.
3. **Create a custom configuration** with:
   - Secret Protection
   - Push protection
   - Validity checks (and extended metadata)
   - Generic patterns
   - Scan for AI-detected secrets
   - Bypass privileges for your security team
   - Prevent direct alert dismissals
4. **Policy:** set it as the default for new repositories and choose **Enforce**.
5. **Click Save configuration.**
6. **Apply it:** **Repositories** tab → select all → **Apply configuration ▾** → your configuration → **Apply**.
7. **Add a resource link:** Advanced Security ▾ → **Global settings** → **Add a resource link in the CLI and the web UI when a commit is blocked**.

---

## 📝 Additional Notes

> 💡 **Licensing models:** organizations on the original bundled **GitHub Advanced Security** license see slightly different setting names and order in the configuration editor than organizations on the separate **Secret Protection** and **Code Security** products.

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

### Q: Push protection is blocking my push, but the detected string is a false positive. How do I bypass it?
**A:** If bypass privileges aren't restricted, follow the link in the block message, choose a reason (**It's used in tests**, **It's a false positive**, or **I'll fix it later**), and push again. If your organization uses **bypass privileges** (delegated bypass), submit a bypass request and wait for an approved reviewer. Every bypass creates an alert for the security team.

---

### Q: Secret scanning is enabled but it is not finding secrets I know are in the repo. Why?
**A:** Check that the secret type is in the [supported secret scanning patterns](https://docs.github.com/en/code-security/reference/secret-security/supported-secret-scanning-patterns) list. Generic secrets (private keys, connection strings) need **Generic patterns** turned on, and passwords need **Scan for AI-detected secrets**. Internal formats need a custom pattern at the repository, organization, or enterprise level.

---

### Q: Push protection is enabled, but users can still push commits containing secrets. What is wrong?
**A:** By default, contributors can bypass push protection by giving a reason. To require approval, turn on **Bypass privileges** in your security configuration (or `secret_scanning_delegated_bypass` via the API) and name the roles or teams that can bypass or approve requests. Also remember that push protection only blocks **supported** patterns.

---

### Q: A secret was already committed to git history before we enabled scanning. How do I remove it?
**A:** First, immediately revoke the exposed credential with the issuing provider. Then use `git filter-repo` or the BFG Repo Cleaner to rewrite history and remove the secret from all commits. After rewriting, force-push to GitHub. Note that anyone who cloned the repo will need to re-clone. Simply deleting the file in a new commit does NOT remove it from history.

---

### Q: I created a custom secret scanning pattern, but it is not matching secrets I expect it to find. What should I check?
**A:** In the custom pattern editor, enter a test string and click **Save and dry run** to see what the pattern matches before you publish it.

Common issues:

- Unescaped special characters.
- Anchors that are too strict.
- Missing **Before secret** / **After secret** context.

Then confirm the pattern is published at the right scope (repository, organization, or enterprise). If it should block pushes, confirm push protection is enabled for it.

---

### Q: What is the difference between secret scanning alerts and push protection?
**A:** They work together:

- **Secret scanning alerts** *detect* secrets that are already in your repositories and their history.
- **Push protection** *prevents* new leaks by blocking pushes that contain supported secret patterns.

Push protection stops new leaks; alerts catch the secrets committed before push protection was turned on.

</details>

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Secret Risk Assessment (Organization-Wide Scan) | `Security/Secret Risk Assessment (Organization-Wide Scan).md` |
| Code Scanning (CodeQL) Enablement & Troubleshooting | `Security/Code Scanning (CodeQL) Enablement & Troubleshooting.md` |
| Responsible AI Guardrails | `Copilot/Responsible AI Guardrails.md` |
| Enterprise Environment Scaffolding Checklist | `Setup/Enterprise Environment Scaffolding Checklist.md` |

---

## 📝 Resources

| Resource | Link |
|----------|------|
| Enabling secret scanning for a repository | [GitHub Docs](https://docs.github.com/en/code-security/how-tos/secure-your-secrets/detect-secret-leaks/enable-secret-scanning) |
| Enabling push protection for a repository | [GitHub Docs](https://docs.github.com/en/code-security/how-tos/secure-your-secrets/prevent-future-leaks/enable-push-protection) |
| Managing user push protection | [GitHub Docs](https://docs.github.com/en/code-security/how-tos/secure-your-secrets/prevent-future-leaks/manage-user-push-protection) |
| Protecting your organization's secrets | [GitHub Docs](https://docs.github.com/en/code-security/how-tos/secure-at-scale/configure-organization-security/configure-specific-tools/protect-your-secrets) |
| Creating a custom security configuration | [GitHub Docs](https://docs.github.com/en/code-security/how-tos/secure-at-scale/configure-organization-security/establish-complete-coverage/create-custom-configuration) |
| Global security settings | [GitHub Docs](https://docs.github.com/en/code-security/how-tos/secure-at-scale/configure-organization-security/establish-complete-coverage/configure-global-settings) |

---

*Last updated: October 2026*
