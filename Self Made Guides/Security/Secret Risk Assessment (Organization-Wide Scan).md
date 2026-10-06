# 🔎 GitHub Secret Risk Assessment (Organization-Wide Scan) Runbook

> **Run a free, point-in-time scan of every repository in an organization — including repositories without Secret Protection — and turn the results into a remediation plan**

| 🧭 **At a Glance** | |
|---|---|
| **Goal** | Measure an organization's leaked-secret exposure for free |
| **Use this when** | Before buying Secret Protection, or to size a cleanup |
| **People you need** | Organization owner or security manager |
| **Where you click** | GitHub (organization Security and quality tab) |
| **End result** | A risk report and CSV, plus a plan to remediate and prevent leaks |
| **New to a term?** | See the [Glossary](../Glossary.md) for plain-English definitions |

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [📋 Overview](#-overview)
- [1️⃣ How to Run the Assessment](#1️⃣-how-to-run-the-assessment)
- [2️⃣ What the Assessment Provides](#2️⃣-what-the-assessment-provides)
- [3️⃣ What to Do with the Results](#3️⃣-what-to-do-with-the-results)
- [4️⃣ Enable Secret Protection and Push Protection](#4️⃣-enable-secret-protection-and-push-protection)
- [5️⃣ Configure Custom Patterns](#5️⃣-configure-custom-patterns)
- [6️⃣ Handling Detected Secrets](#6️⃣-handling-detected-secrets)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📚 Resources](#-resources)

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Run the assessment:** `Org → Security and quality tab → Assessments` (under "Security") → **Scan your organization**
- **Rerun (every 90 days):** same page → **Rerun scan**
- **Download the CSV:** same page → **⋯** → **Download CSV**
- **Enable Secret Protection afterward:** same page → **Get started ▾** → **For all repositories** → **Enable Secret Protection**
- **Custom patterns:** `Org → Settings → Advanced Security ▾ → Global settings` → **Custom patterns** → **New pattern**

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
| An organization on **GitHub Team** or **GitHub Enterprise Cloud** | — | ☐ |
| Run, rerun, and view the assessment | **Organization owner** or **security manager** | ☐ |
| No Secret Protection or GHAS license needed — the assessment is free | — | ☐ |
| Email notification when the report is ready (optional) | Organization owners who opted in to email notifications | ☐ |
| For an enterprise: someone with owner or security manager access in **each** organization | Enterprise owner coordinates | ☐ |

---

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **GitHub organization owner or security manager** | Runs the assessment in each organization and turns the results into a remediation list. | GitHub → [organization] → Security and quality tab → Assessments → Scan your organization → wait for the report → ⋯ → Download CSV. The report is aggregate (secret types and counts), so enable Secret Protection to get individual alerts with file locations. Handoff: per-organization CSV and the list of secret types to remediate. |
| **Azure or Microsoft Entra resource owner, if Azure credentials are confirmed exposed** | Revokes, rotates, or removes the exposed Azure credential and updates dependent workloads. | For app secrets: Microsoft Entra admin center → Entra ID → App registrations → [application] → Certificates & secrets → delete compromised secret or certificate → New client secret if still needed → update workload secret store. For Azure role exposure: Azure portal → Subscriptions or Resource groups → [scope] → Access control (IAM) → Role assignments → remove unneeded principal. Handoff: rotated credential ID, disabled credential ID, and validation that workloads use the replacement. |
| **Okta or PingFederate admin, if IdP tokens or provisioning credentials are confirmed exposed** | Rotates the IdP-side API token or SCIM credential and updates the GitHub integration. | Okta: Applications → [GitHub app] → Provisioning → Integration → Edit → replace API token → Test API Credentials → Save. PingFederate: Applications → SP Connections → [GitHub connection] → Outbound Provisioning → Target → replace Access Token → Save → activate/test channel. Handoff: new credential stored in vault and old credential disabled. |

---

## 📋 Overview

The secret risk assessment is a **free**, on-demand scan of an organization's repositories for hard-coded credentials such as API keys, tokens, and passwords — including repositories that don't have Secret Protection turned on.

| Aspect | Detail |
|--------|--------|
| **Cost** | Free on GitHub Team and GitHub Enterprise Cloud |
| **Scope** | One **organization** at a time — all its repositories (public, private, internal, archived) |
| **Type** | Point-in-time; can be rerun once every **90 days** |
| **Output** | An aggregate dashboard plus a CSV — secret types and counts, not individual alerts |
| **Purpose** | Size your exposure, prioritize remediation, and build the case for Secret Protection |

> 💡 **Tip:** For an enterprise, run it in each organization. The first run also starts a **code security risk assessment** if you haven't run one before.

---

## 1️⃣ How to Run the Assessment

**👤 Role:** **Organization owner** or **security manager** · **📍 Portal:** GitHub

**Navigate:** Organization → **Security and quality** tab → **Assessments** *(under "Security")*

**Steps:**

1. Go to the organization's main page.
2. Under the organization name, click the **Security and quality** tab.
3. In the sidebar, under **Security**, click **Assessments**.
4. Click **Scan your organization**.

> ✅ **Result:** GitHub scans the organization's repositories and builds the report. Organization owners who opted in get an email when it's ready.

**Rerun later:** on the same page, at the top right of the existing report, click **Rerun scan** (once every 90 days).

---

## 2️⃣ What the Assessment Provides

**View it:** Organization → **Security and quality** → **Assessments** → **Secret Protection** tab *(the **Code Security** tab shows the code assessment)*.

| Data point | Description |
|------------|-------------|
| **Total secrets** | All secret leaks found across the organization |
| **Public leaks** | Distinct secrets found in **public** repositories |
| **Preventable leaks** | Leaks that push protection could have prevented |
| **Secret categories** | **Provider patterns** (AWS, Azure, GitHub tokens…) vs **generic patterns** (private keys, passwords…) |
| **Repositories with leaks** | How many repositories contain leaks |
| **Secret type table** | Each secret type with distinct repositories and secrets found, sorted by count |

**Download the CSV:** at the top right of the report, click **⋯** → **Download CSV**.

| CSV column | Meaning |
|------------|---------|
| `Organization Name` | Where the secret was found |
| `Name` / `Slug` | Secret type (token name and normalized token) |
| `Push Protected` | Would push protection block it? |
| `Generic Pattern` | Is it a generic pattern? |
| `Secret Count` | Active and inactive secrets of that type |
| `Repository Count` | Distinct repositories with that type, including archived |

> 💡 **Tip:** Private-repository leaks = **Total secrets** − **Public leaks**.

---

## 3️⃣ What to Do with the Results

| Priority | What | Why |
|----------|------|-----|
| 1 | **Provider-pattern secrets in public repositories** | Anyone on the internet can see them, and you know which service to revoke |
| 2 | **Generic-pattern secrets in public repositories** | Need investigation to find the owning system |
| 3 | **Secrets in private and internal repositories** | Lower immediate risk, but exposed if access widens or a repo goes public |

Then look for patterns:

- **Many repositories with leaks** → organization-wide training and push protection.
- **The same secret type again and again** → a team or workflow that needs better secret handling (environment variables, a vault).
- **Common categories** → CI/CD processes to fix.

> ⚠️ **Warning:** Treat every detected secret as potentially compromised. Rotate it even if it's in a private repository.

---

## 4️⃣ Enable Secret Protection and Push Protection

The assessment only gives counts. To find **where** each secret is and to block new ones, turn on Secret Protection.

**👤 Role:** **Organization owner** or **security manager** · **📍 Portal:** GitHub

**Fastest path (from the assessment page):**

1. Organization → **Security and quality** → **Assessments**.
2. In the banner, click **Get started ▾**.
3. Choose **For public repositories for free**, or **For all repositories** → review the estimate → **Enable Secret Protection** (turns on alerts **and** push protection). Or click **Configure in settings** to choose repositories.

**Controlled path:** create and apply a security configuration — see `Security/Secret Protection Enablement.md`, Section 3.

**Single repository:** Repository → **Settings** → **Advanced Security** *(under "Security and quality")* → **Secret Protection** → **Enable** → **Enable Secret Protection**, then **Push protection** → **Enable**.

> ✅ **Result:** secret scanning creates an alert for each secret with its location, and push protection blocks new supported secrets.

---

## 5️⃣ Configure Custom Patterns

For internal token formats the built-in patterns don't cover.

**👤 Role:** **Organization owner** or **security manager** · **📍 Portal:** GitHub

**Navigate:** Organization → **Settings** → in the **Security** section, **Advanced Security ▾** → **Global settings** → **Custom patterns**

**Steps:**

1. Under **Custom patterns**, click **New pattern**.
2. Enter a **Pattern name** and the **Secret format** (a regular expression).
3. *(Optional)* Click **More options** to set **Before secret**, **After secret**, and extra match requirements.
4. Enter a sample **test string** and check that it matches.
5. Click **Save and dry run**, choose **All repositories in the organization** or up to 10 **Selected repositories**, then click **Run**.
6. Review the sample results (up to 1,000) for false positives. Edit and click **Save and dry run** again as needed.
7. Click **Publish pattern**.
8. *(Optional)* To block this pattern on push, click **Enable** for push protection.

> 📌 Custom patterns only scan repositories where secret scanning is enabled. The risk assessment itself uses GitHub's built-in patterns, not your custom ones.

---

## 6️⃣ Handling Detected Secrets

Once Secret Protection is on, handle each alert:

| Step | Action |
|------|--------|
| 1 | **Revoke or rotate the credential** with the issuing provider |
| 2 | **Check the provider's access logs** for unauthorized use |
| 3 | **Update dependent systems** to use the new credential |
| 4 | **Close the alert** on the **Security and quality** tab with the right reason — for example **Revoked** |

> ⚠️ **Important:** The risk assessment is **point-in-time**. Continuous detection needs Secret Protection enabled on the repositories.

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
| **Rerun scan is unavailable** | A report can only be generated once every 90 days. | Wait until 90 days after the last scan, or enable Secret Protection for continuous scanning. |
| **Assessments page or Scan your organization button is missing** | You aren't an organization owner or security manager, or you're looking at an enterprise rather than an organization. | Open the organization's **Security and quality** tab with an owner or security manager account. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>

### Q: The assessment says "0 secrets found" but we know secrets exist in our repositories. Why?
**A:** The assessment uses GitHub's built-in provider and generic patterns. Internal formats (for example, tokens with a custom prefix) aren't detected — define custom patterns and enable Secret Protection to scan for them. Also confirm you ran it in the right organization.

---

### Q: Can we schedule the risk assessment to run automatically on a recurring basis?
**A:** No. It's on demand, and a new report can be generated only once every 90 days (**Rerun scan**). For continuous detection, enable Secret Protection — it scans new pushes and alerts in near real time.

---

### Q: The results show secrets found in archived repositories. Should we be concerned?
**A:** Yes. Archiving stops new commits but doesn't make an exposed credential safe. Rotate any credentials found in archived repositories just as you would for active ones.

---

### Q: Who has permission to run the secret risk assessment?
**A:** **Organization owners** and **security managers** of the organization. It's run per organization — for an enterprise, someone with one of those roles runs it in each organization.

---

### Q: How long does the risk assessment take to complete?
**A:** It depends on the number and size of repositories. Organization owners who opted in to email notifications get an email when the report is ready; otherwise, check the **Assessments** page.

---

### Q: Does running the risk assessment enable secret scanning on our repositories?
**A:** No. It's a read-only scan and changes no settings. After reviewing the results, enable Secret Protection and push protection separately (Section 4).

---

### Q: Does the assessment show which file each secret is in?
**A:** No. The report and CSV are aggregate — secret types, counts, and how many repositories are affected. To see each secret's location, enable Secret Protection so secret scanning creates individual alerts.

</details>

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Secret Protection Enablement | `Security/Secret Protection Enablement.md` |
| Enterprise Environment Scaffolding Checklist | `Setup/Enterprise Environment Scaffolding Checklist.md` |
| Code Scanning (CodeQL) Enablement & Troubleshooting | `Security/Code Scanning (CodeQL) Enablement & Troubleshooting.md` |

---

## 📚 Resources

- [Running the secret risk assessment for your organization](https://docs.github.com/en/enterprise-cloud@latest/code-security/how-tos/secure-at-scale/configure-organization-security/configure-specific-tools/assess-your-secret-risk)
- [Interpreting secret risk assessment results](https://docs.github.com/en/code-security/tutorials/secure-your-organization/interpret-secret-risk-assessment)
- [Exporting the risk report as CSV](https://docs.github.com/en/code-security/how-tos/view-and-interpret-data/analyze-organization-data/export-risk-report-csv)
- [Protecting your organization's secrets](https://docs.github.com/en/code-security/how-tos/secure-at-scale/configure-organization-security/configure-specific-tools/protect-your-secrets)
- [Defining custom patterns for secret scanning](https://docs.github.com/en/code-security/how-tos/secure-your-secrets/customize-leak-detection/define-custom-patterns)

---

*Last updated: October 2026*
