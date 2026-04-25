# 🔎 GitHub Secret Risk Assessment (Enterprise-Wide Scan) Runbook

> **Run an enterprise-wide secret scan across ALL repos (including those without GHAS enabled) -- FREE**

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Run assessment:** `Enterprise → Settings → Code Security → Secret scanning → Run a risk assessment`
- **Enable push protection (org):** `Org → Settings → Code security → Secret scanning → Push protection → Enable`
- **Create custom patterns:** `Org → Settings → Code security → Secret scanning → Custom patterns → New pattern`

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
| Enterprise owner role | ☐ |
| GitHub Enterprise Cloud account | ☐ |
| No GHAS license required (assessment is free) | ☐ |

---

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **GitHub security manager or enterprise owner** | Runs the GitHub-side inventory and confirms which leaked secrets require provider action. | Enterprise path: GitHub → profile photo → Your enterprises → [enterprise] → Settings → Code Security → Secret scanning → Run a risk assessment. Organization path: GitHub → profile photo → Your organizations → [organization] → Settings → Code security → Secret scanning → Run a risk assessment. Then open the assessment/export, filter by provider and repository, open each alert, and record secret type, owner, and remediation status. Handoff: provider-specific remediation list. |
| **Azure or Microsoft Entra resource owner, if Azure credentials are confirmed exposed** | Revokes, rotates, or removes the exposed Azure credential and updates dependent workloads. | For app secrets: Microsoft Entra admin center → Entra ID → App registrations → [application] → Certificates & secrets → delete compromised secret or certificate → New client secret if still needed → update workload secret store. For Azure role exposure: Azure portal → Subscriptions or Resource groups → [scope] → Access control (IAM) → Role assignments → remove unneeded principal. Handoff: rotated credential ID, disabled credential ID, and validation that workloads use the replacement. |
| **Okta or PingFederate admin, if IdP tokens or provisioning credentials are confirmed exposed** | Rotates the IdP-side API token or SCIM credential and updates the GitHub integration. | Okta: Applications → [GitHub app] → Provisioning → Integration → Edit → replace API token → Test API Credentials → Save. PingFederate: Applications → SP Connections → [GitHub connection] → Outbound Provisioning → Target → replace Access Token → Save → activate/test channel. Handoff: new credential stored in vault and old credential disabled. |

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
