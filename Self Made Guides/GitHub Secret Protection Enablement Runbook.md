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

> 💡 **Note:** In GitHub's UI/docs, secret scanning alerts for users are enabled when you enable "Secret Protection" for a repository. Push protection requires Secret Protection first.[1]

### Availability

| Repo Type | Availability |
|-----------|--------------|
| **Public repos** | Secret scanning available |
| **Org-owned repos** | Requires GitHub Team with GitHub Secret Protection enabled [1][2] |

---

## 1️⃣ Enable Secret Protection for a Single Repository

### A) Turn on Secret Protection (enables secret scanning alerts)

**Navigation:**

```
Repository → Settings → Security (sidebar) → Advanced Security
  → Secret Protection → Enable
    → Review impact → Enable Secret Protection
```

> ✅ **Result:** Secret scanning alerts for users are enabled when you enable Secret Protection.[1]

---

### B) Turn on Push Protection (repo-level)

> ⚠️ **Prerequisite:** Secret Protection must already be enabled.

**Navigation:**

```
Repository → Settings → Security (sidebar) → Code security
  → Secret Protection → Push Protection → Enable
```

> ✅ **Result:** Pushes containing supported secrets are blocked (unless bypassed), and bypasses generate alerts.[3][4]

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

> 💡 **Tip:** Security configurations are GitHub's at-scale mechanism to apply enablement settings across many repositories.[5]

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
| **Scan for generic passwords** | Detect common password patterns (AI detection) [6] |
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

> 💡 **Note:** This is enabled by default and can be disabled.[7][8]

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

> 💡 **Customization:** If you're targeting GHEC vs GHES, and whether you're using GitHub Secret Protection / Code Security (new products) vs legacy GHAS licensing, the runbook's wording can be tailored to match exactly what your admins will see. Some settings paths may vary slightly (e.g., "Code security" vs "Advanced Security" at the repository level depending on the specific task).[2][4][1]

---

*Last updated: February 2026*