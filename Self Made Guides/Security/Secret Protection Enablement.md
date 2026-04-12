# 🔐 GitHub Secret Protection Enablement Runbook

> **Complete guide to enabling secrets protection across repositories, organizations, and at scale via Security Configurations**

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

---

## ❓ Common Questions & Troubleshooting

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

## 📝 Resources

| Resource | Link |
|----------|------|
| About secret scanning | [GitHub Docs](https://docs.github.com/en/code-security/secret-scanning/introduction/about-secret-scanning) |
| Configuring secret scanning | [GitHub Docs](https://docs.github.com/en/code-security/secret-scanning/enabling-secret-scanning-features/enabling-secret-scanning-for-your-repository) |
| Push protection for repositories | [GitHub Docs](https://docs.github.com/en/code-security/secret-scanning/enabling-secret-scanning-features/enabling-push-protection-for-your-repository) |
| Security configurations | [GitHub Docs](https://docs.github.com/en/code-security/securing-your-organization/enabling-security-features-in-your-organization/configuring-global-security-settings-for-your-organization) |

---

*Last updated: February 2026*