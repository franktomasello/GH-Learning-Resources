# 🔎 Secret Risk Assessment Runbook

> **Run an enterprise-wide secret scan across ALL repos (including those without GHAS enabled) -- FREE**

---

## 📋 Overview

The Secret Risk Assessment is a **free**, enterprise-wide scan that detects leaked secrets across **all** repositories -- including those that do not have GHAS or secret scanning enabled.

| Aspect | Detail |
|--------|--------|
| **Cost** | Free -- no GHAS license required |
| **Scope** | All repositories in the enterprise (public, private, internal) |
| **Type** | Point-in-time scan |
| **Purpose** | Discover exposure, prioritize remediation, build business case for GHAS |

> 💡 **Tip:** This is the best first step for organizations evaluating their secret exposure before committing to a full GHAS rollout.

---

## 1️⃣ How to Run the Assessment

**Navigation:**

```
Enterprise → Settings → Code Security (sidebar)
  → Secret scanning → Run a risk assessment
```

- Scans across **all orgs and repos** in the enterprise
- Does **NOT** require GHAS licenses on every repo

> ✅ **Result:** GitHub scans all repositories in the enterprise and generates a report of detected secrets.

> 💡 **Tip:** The assessment may take some time depending on the number of repositories. You will be notified when results are ready.

---

## 2️⃣ What the Assessment Provides

| Data Point | Description |
|------------|-------------|
| **Total secrets detected** | Count of all secrets found across the enterprise |
| **Affected repositories** | Number and list of repos containing leaked secrets |
| **Secret type breakdown** | Categories of secrets found (AWS keys, GitHub tokens, Azure credentials, etc.) |
| **Prioritization guidance** | Severity-based ranking to guide remediation efforts |

> 💡 **Tip:** Use the secret type breakdown to identify which credential providers are most at risk and prioritize rotating those credentials first.

---

## 3️⃣ What to Do with the Results

| Step | Action | Detail |
|------|--------|--------|
| 1 | **Revoke and rotate** | Immediately revoke any active credentials found in the report |
| 2 | **Build business case** | Use the report data to justify enabling GHAS/secret scanning across all repos |
| 3 | **Enable push protection** | Block future secret commits at the push level |

> ⚠️ **Warning:** Treat every detected secret as potentially compromised. Even if a secret appears in a private repo, it should be rotated immediately.

---

## 4️⃣ Enable Push Protection After Assessment

Push protection blocks secrets from being committed in the first place.

### Org Level (Recommended)

**Navigation:**

```
Organization → Settings → Code security (sidebar)
  → Secret scanning → Push protection → Enable
```

### Repo Level

**Navigation:**

```
Repository → Settings → Code security and analysis (sidebar)
  → Secret scanning → Push protection → Enable
```

> ✅ **Result:** Pushes containing supported secret patterns are blocked before they reach the repository.

---

## 5️⃣ Configure Custom Patterns

For organization-specific secret formats (internal tokens, proprietary API keys), create custom patterns.

**Navigation:**

```
Organization → Settings → Code security (sidebar)
  → Secret scanning → Custom patterns → New pattern
```

**Steps:**

1. Define a **regex pattern** for the secret format
2. Test against sample data before enabling
3. Save and enable the pattern

> 💡 **Tip:** Test thoroughly with real examples to minimize false positives before enabling across the organization.

---

## 6️⃣ Handling Detected Secrets

For each secret found in the assessment or through ongoing scanning:

| Step | Action |
|------|--------|
| 1 | **Revoke the credential immediately** with the issuing provider |
| 2 | **Close the alert as "Revoked"** in the GitHub Security tab |
| 3 | **Check access logs** at the credential provider for unauthorized usage |

> ⚠️ **Important:** The risk assessment is **point-in-time**. Ongoing detection requires GHAS secret scanning enabled on repositories.

---

## 📚 Resources

- [Understanding your organization's exposure to leaked secrets](https://docs.github.com/en/enterprise-cloud@latest/code-security/securing-your-organization/understanding-your-organizations-exposure-to-leaked-secrets)

---

*Last updated: April 2026*
