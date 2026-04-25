# 🔐 GitHub Secret Protection Enablement Runbook

> **Complete guide to enabling secrets protection across repositories, organizations, and at scale via Security Configurations**

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [📋 Overview](#-overview)
- [🔑 What "Secrets Protection" Means in GitHub](#-what-secrets-protection-means-in-github)
- [1️⃣ Enable Secret Protection for a Single Repository](#1-enable-secret-protection-for-a-single-repository)
- [2️⃣ Enable Secret Protection at the Organization Level](#2-enable-secret-protection-at-the-organization-level)
- [3️⃣ Enable at Scale Using Security Configurations](#3-enable-at-scale-using-security-configurations)
- [4️⃣ Enable Push Protection for Your User Account](#4-enable-push-protection-for-your-user-account)
- [5️⃣ Enable via REST API](#5-enable-via-rest-api)
- [🚀 Quick "Most Complete" Rollout Recipe](#-quick-most-complete-rollout-recipe)
- [📝 Additional Notes](#-additional-notes)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Single repo:** `Repo → Settings → Security → Advanced Security → Secret Protection → Enable`
- **Org-wide:** `Org → Security → Assessments → Get started → For all repositories`
- **At scale (Security Configs):** `Org → Settings → Advanced Security → Configurations → Apply to All repositories`
- **Push protection (repo):** `Repo → Settings → Security → Code security → Secret Protection → Push Protection → Enable`
- **User-level push protection:** `Profile → Settings → Code security → Push protection for yourself → Toggle on`

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
| GitHub Enterprise Cloud or GitHub Team plan | ☐ |
| GitHub Secret Protection product enabled (for private/internal repos) | ☐ |
| Org owner or repo admin role | ☐ |
| Understanding of which repos to target (public vs. private/internal) | ☐ |

---

## 📋 Overview

This runbook covers every practical way to "turn on secrets protection" in GitHub:

| Method | Scope | Best For |
|--------|-------|----------|
| **Repo-by-repo** | Single repository | Testing, specific repo needs |
| **Org-wide** | All/public/selected repos | Organization rollouts |
| **Security Configurations** | At-scale enablement | Enterprise-wide deployments |
| **User-level push protection** | Individual user | Personal protection on public repos |
| **REST API** | Automation/scripting | Bulk operations |

---

## 🔑 What "Secrets Protection" Means in GitHub

GitHub's "secrets protection" capabilities include:

| Feature | Description |
|---------|-------------|
| **Secret scanning alerts** | Detect secrets already in the repo |
| **Push protection** | Block secrets from being pushed going forward |

> 💡 **Note:** In GitHub's UI/docs, secret scanning alerts for users are enabled when you enable "Secret Protection" for a repository. Push protection requires Secret Protection first.

### Availability

| Repo Type | Availability |
|-----------|--------------|
| **Public repos** | Secret scanning available |
| **Org-owned repos** | Requires GitHub Team with GitHub Secret Protection enabled  |

---

## 1️⃣ Enable Secret Protection for a Single Repository

### A) Turn on Secret Protection (enables secret scanning alerts)

**Navigation:**

```
Repository → Settings → Security (sidebar) → Advanced Security
  → Secret Protection → Enable
    → Review impact → Enable Secret Protection
```

> ✅ **Result:** Secret scanning alerts for users are enabled when you enable Secret Protection.

---

### B) Turn on Push Protection (repo-level)

> ⚠️ **Prerequisite:** Secret Protection must already be enabled.

**Navigation:**

```
Repository → Settings → Security (sidebar) → Code security
  → Secret Protection → Push Protection → Enable
```

> ✅ **Result:** Pushes containing supported secrets are blocked (unless bypassed), and bypasses generate alerts.

---

## 2️⃣ Enable Secret Protection at the Organization Level

*Fastest org-wide entry points via the guided enablement flow*

### Navigate to Assessments

**Navigation:**

```
Organization → Security (sidebar) → Assessments
```

### Enablement Options

In the banner, open **Get started** dropdown and choose one:

| Option | Effect |
|--------|--------|
| **For public repositories for free** | Enables only public repos |
| **For all repositories** | Shows estimate, enables secret scanning alerts + push protection across your org |
| **Configure in settings** | Customize which repos get enabled (takes you to Security Configurations) |

---

## 3️⃣ Enable at Scale Using Security Configurations

*Recommended for real organization rollouts*

> 💡 **Tip:** Security configurations are GitHub's at-scale mechanism to apply enablement settings across many repositories.

---

### A) Apply the GitHub-recommended Configuration to ALL Repositories

**Navigation:**

```
Profile Picture → Organizations → [Your Organization] → Settings
  → Advanced Security (sidebar) → Configurations
```

**Steps:**

1. In the **GitHub recommended** row, click **Apply to** dropdown
2. Select one of:
   - **All repositories**
   - **All repositories without configurations**

> ✅ **Result:** The recommended configuration includes GitHub Secret Protection features (may incur costs for private/internal repos depending on licensing).

---

### B) Create a Custom Security Configuration

**Navigation:**

```
Profile Picture → Organizations → [Your Organization] → Settings
  → Advanced Security (sidebar) → Configurations
    → New configuration
```

**Configuration Steps:**

1. Enter **Name** and **Description**
2. Under **Secret Protection** (optional, paid for private repos), enable it
3. Configure the following options:

| Setting | Description |
|---------|-------------|
| **Validity checks** | Verify if detected secrets are still valid |
| **Non-provider patterns** | Scan for patterns not from known providers |
| **Scan for generic passwords** | Detect common password patterns (AI detection)  |
| **Push protection** | Block secrets from being pushed |
| **Bypass privileges** | Allow selected members to bypass; others require review/approval |
| **Prevent direct alert dismissals** | Require justification for dismissing alerts |

4. Click **Save configuration**

---

### C) Apply Your Custom Security Configuration to Repositories

**Navigation:**

```
Profile Picture → Organizations → [Your Organization] → Settings
  → Advanced Security (sidebar) → Configurations
```

**Steps:**

1. In **Apply configurations**, optionally filter repositories
2. Select repositories in the table
3. Apply the configuration

> 💡 **Tip:** The Configurations table supports multiple selection methods for applying to repos.

---

## 4️⃣ Enable Push Protection for Your User Account

*Protect your own pushes to any public repo (separate from org/repo settings)*

**Navigation:**

```
Profile Picture → Settings
  → Code security (sidebar, under "Security")
    → Push protection for yourself → Toggle on/off
```

> 💡 **Note:** This is enabled by default and can be disabled.

---

## 5️⃣ Enable via REST API

*Useful for automation and bulk scripting*

### Update a Repository with security_and_analysis

Use the **Update a repository** endpoint (`PATCH`) and set fields under `security_and_analysis`:

| Field | Values |
|-------|--------|
| `secret_scanning.status` | `"enabled"` or `"disabled"` |
| `secret_scanning_push_protection.status` | `"enabled"` or `"disabled"` |

**Example Request Body:**

```json
{
  "security_and_analysis": {
    "secret_scanning": { "status": "enabled" },
    "secret_scanning_push_protection": { "status": "enabled" }
  }
}
```

---

## 🚀 Quick "Most Complete" Rollout Recipe

*If your goal is to turn it on everywhere with the most control:*

### Steps

1. **Navigate to Configurations:**
   ```
   Org Settings → Advanced Security → Configurations
   ```

2. **Create a custom configuration** enabling:
   - Secret Protection
   - Push protection
   - Validity checks
   - Non-provider patterns
   - Generic passwords
   - Bypass privileges
   - Prevent direct dismissals

3. **Apply to repositories:**
   - Select **All repositories** (or all without configs)
   - Enable **Enforce** if you want to prevent repo owners from changing enforced settings

---

## 📝 Additional Notes

> 💡 **Customization:** If you're targeting GHEC vs GHES, and whether you're using GitHub Secret Protection / Code Security (new products) vs legacy GHAS licensing, the runbook's wording can be tailored to match exactly what your admins will see. Some settings paths may vary slightly (e.g., "Code security" vs "Advanced Security" at the repository level depending on the specific task).

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
**A:** When push protection blocks a push, you can bypass it by selecting a reason (e.g., "used in tests" or "false positive") in the GitHub UI or CLI prompt. If your organization requires bypass approval, the push will be held until an authorized reviewer approves it. Note that all bypasses generate an alert for security teams to review.

---

### Q: Secret scanning is enabled but it is not finding secrets I know are in the repo. Why?
**A:** Secret scanning only detects patterns from its supported providers list. Verify that the secret type you expect to find is in the [supported secret patterns list](https://docs.github.com/en/code-security/secret-scanning/introduction/supported-secret-scanning-patterns). Custom or proprietary secret formats require you to create a custom pattern (org-level or enterprise-level) to be detected.

---

### Q: Push protection is enabled, but users can still push commits containing secrets. What is wrong?
**A:** Check whether your organization allows push protection bypass without requiring approval. If bypass is configured to "Always allow," users can self-approve and push the secret through. To enforce stricter controls, configure bypass privileges to require review/approval from a designated security team before the push is allowed.

---

### Q: A secret was already committed to git history before we enabled scanning. How do I remove it?
**A:** First, immediately revoke the exposed credential with the issuing provider. Then use `git filter-repo` or the BFG Repo Cleaner to rewrite history and remove the secret from all commits. After rewriting, force-push to GitHub. Note that anyone who cloned the repo will need to re-clone. Simply deleting the file in a new commit does NOT remove it from history.

---

### Q: I created a custom secret scanning pattern, but it is not matching secrets I expect it to find. What should I check?
**A:** Verify the regex syntax is correct by testing it against sample data using the "Test" feature in the custom pattern editor before enabling. Common issues include unescaped special characters, overly strict anchoring, and missing character classes. Also confirm the pattern is enabled and applied to the correct scope (org or enterprise).

---

### Q: What is the difference between secret scanning alerts and push protection?
**A:** Secret scanning alerts are a detection mechanism that scans existing repository content and history, alerting you to secrets that are already present. Push protection is a prevention mechanism that blocks secrets from being committed in the first place by rejecting pushes that contain supported secret patterns. Both features complement each other: push protection stops new leaks, while secret scanning alerts catch secrets that were committed before push protection was enabled.

</details>

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Secret Risk Assessment (Enterprise-Wide Scan) | `Security/Secret Risk Assessment (Enterprise-Wide Scan).md` |
| Code Scanning (CodeQL) Enablement & Troubleshooting | `Security/Code Scanning (CodeQL) Enablement & Troubleshooting.md` |
| Responsible AI Guardrails | `Copilot/Responsible AI Guardrails.md` |
| Enterprise Environment Scaffolding Checklist | `Setup/Enterprise Environment Scaffolding Checklist.md` |

---

## 📝 Resources

| Resource | Link |
|----------|------|
| About secret scanning | [GitHub Docs](https://docs.github.com/en/code-security/secret-scanning/introduction/about-secret-scanning) |
| Configuring secret scanning | [GitHub Docs](https://docs.github.com/en/code-security/secret-scanning/enabling-secret-scanning-features/enabling-secret-scanning-for-your-repository) |
| Push protection for repositories | [GitHub Docs](https://docs.github.com/en/code-security/secret-scanning/enabling-secret-scanning-features/enabling-push-protection-for-your-repository) |
| Security configurations | [GitHub Docs](https://docs.github.com/en/code-security/securing-your-organization/enabling-security-features-in-your-organization/configuring-global-security-settings-for-your-organization) |

---

*Last updated: April 2026*
