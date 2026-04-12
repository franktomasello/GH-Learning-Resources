# 🔎 Secret Risk Assessment Runbook

> **Run an enterprise-wide secret scan across ALL repos (including those without GHAS enabled) -- FREE**

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Run assessment:** `Enterprise → Settings → Code Security → Secret scanning → Run a risk assessment`
- **Enable push protection (org):** `Org → Settings → Code security → Secret scanning → Push protection → Enable`
- **Create custom patterns:** `Org → Settings → Code security → Secret scanning → Custom patterns → New pattern`

---

## ✅ Prerequisites

| Requirement | Status |
|-------------|--------|
| Enterprise owner role | ☐ |
| GitHub Enterprise Cloud account | ☐ |
| No GHAS license required (assessment is free) | ☐ |

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

## ❓ Common Questions & Troubleshooting

### Q: The assessment says "0 secrets found" but we know secrets exist in our repositories. Why?
**A:** The risk assessment detects secrets matching GitHub's supported provider patterns (AWS keys, Azure tokens, GitHub PATs, etc.). If your organization uses custom or proprietary secret formats (e.g., internal API tokens with a non-standard prefix), the assessment will not detect them. Create custom secret scanning patterns at the org or enterprise level to cover your proprietary formats, then enable ongoing secret scanning for continuous detection.

---

### Q: Can we schedule the risk assessment to run automatically on a recurring basis?
**A:** No. The enterprise-level secret risk assessment is an on-demand, point-in-time scan. For ongoing, continuous detection of secrets, enable secret scanning on your repositories. Secret scanning runs automatically on every push and periodically scans existing content, providing real-time alerts rather than periodic snapshots.

---

### Q: The results show secrets found in archived repositories. Should we be concerned?
**A:** Yes. Archived repositories are still scanned because the secrets in them may still be active and exploitable. Revoke and rotate any credentials found in archived repos immediately, just as you would for active repos. The archived status only prevents new commits -- it does not reduce the risk of exposed credentials in existing history.

---

### Q: Who has permission to run the enterprise-wide secret risk assessment?
**A:** Only enterprise owners can initiate the secret risk assessment. Organization owners, billing managers, and other roles do not have access to this feature. If you need the assessment run, contact someone with the enterprise owner role.

---

### Q: How long does the risk assessment take to complete?
**A:** The duration depends on the number of repositories and their size across your enterprise. For small enterprises (under 100 repos), results may be ready within minutes. For large enterprises with thousands of repositories, the assessment may take several hours. You will be notified when the results are ready.

---

### Q: Does running the risk assessment enable secret scanning on our repositories?
**A:** No. The risk assessment is a standalone, read-only scan that does not change any repository settings. It does not enable secret scanning, push protection, or any other feature. After reviewing the results, you must separately enable secret scanning and push protection on your repositories to get ongoing protection.

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Secret Protection Enablement | `Security/Secret Protection Enablement.md` |
| Enterprise Environment Scaffolding Checklist | `Setup/Enterprise Environment Scaffolding Checklist.md` |
| Code Scanning (CodeQL) Enablement & Troubleshooting | `Security/Code Scanning (CodeQL) Enablement & Troubleshooting.md` |

---

## 📚 Resources

- [Understanding your organization's exposure to leaked secrets](https://docs.github.com/en/enterprise-cloud@latest/code-security/securing-your-organization/understanding-your-organizations-exposure-to-leaked-secrets)

---

*Last updated: April 2026*
