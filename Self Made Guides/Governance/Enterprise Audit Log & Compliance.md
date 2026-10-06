# 📋 GitHub Enterprise Audit Log & Compliance Runbook

> How to access, stream, search, and use audit logs for governance, compliance, and security monitoring

| 🧭 **At a Glance** | |
|---|---|
| **Goal** | Search, export, and stream the enterprise audit log |
| **Use this when** | Compliance reviews, incident response, or SIEM integration |
| **People you need** | Enterprise owner; cloud or SIEM admin |
| **Where you click** | GitHub (enterprise Settings) and your SIEM or cloud storage |
| **End result** | Searchable history, a working stream, and a compliance checklist |
| **New to a term?** | See the [Glossary](../Glossary.md) for plain-English definitions |

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [📋 Overview](#-overview)
- [1️⃣ Access the Enterprise Audit Log](#1️⃣-access-the-enterprise-audit-log)
- [2️⃣ Audit Log Streaming (SIEM Integration)](#2️⃣-audit-log-streaming-siem-integration)
- [3️⃣ Audit Log API](#3️⃣-audit-log-api)
- [4️⃣ Git Events](#4️⃣-git-events)
- [5️⃣ IP Allow Lists](#5️⃣-ip-allow-lists)
- [6️⃣ Compliance Checklist](#6️⃣-compliance-checklist)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Enterprise audit log:** `Enterprise → Settings → Audit log`
- **Stream to a SIEM:** `Enterprise → Settings → Audit log → Log streaming` → **Configure stream ▾** → provider → details → **Check endpoint** → **Save**
- **API request events + source IPs:** `Enterprise → Settings → Audit log → Settings` → **Enable API Request Events** / **Enable source IP disclosure** → **Save**
- **Export:** `Audit log` → **Export ▾** (JSON/CSV) or **Export Git Events ▾** → **Download Results**
- **IP allow list:** `Enterprise → Settings → Authentication security` → add CIDR → **Add** → **Enable IP allow list** → **Save**
- **Usage report:** `Enterprise → Billing and licensing → Usage` → **Get usage report**

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
| GitHub Enterprise Cloud | — | ☐ |
| Enterprise audit log, streaming, settings, IP allow list | GitHub **enterprise owner** | ☐ |
| Organization audit log | **Organization owner** | ☐ |
| A streaming destination (S3, Azure Blob Storage, Azure Event Hubs, Datadog, Google Cloud Storage, or Splunk) | Cloud / SIEM administrator | ☐ |
| Destination credentials (access keys or OIDC role, SAS URL, connection string, token, JSON key, HEC token) | Cloud / SIEM administrator | ☐ |
| API access | A token with the `read:audit_log` scope | ☐ |

---

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **GitHub enterprise owner** | Configures audit log streaming in GitHub. | GitHub → Enterprise → Settings → Audit log → Log streaming → Configure stream ▾ → choose the provider → enter the destination details → Check endpoint → Save. Handoff: active stream and a successful endpoint check. |
| **Azure Storage administrator (Blob Storage)** | Creates a container-level SAS URL. | Azure portal → Storage accounts → [account] → Data storage → Containers → [container] → Settings → Shared access tokens → Permissions: **Create** and **Write** only → set an expiry that fits your rotation policy → Generate SAS token and URL → copy **Blob SAS URL**. Handoff: Blob SAS URL and its expiry date. |
| **Azure Event Hubs administrator** | Provides the event hub name and connection string. | Azure portal → search **Event Hubs** → [namespace] → [event hub] → Shared Access Policies → select or create a policy → copy **Connection string-primary key**. Handoff: event hub instance name and connection string. |

---

## 📋 Overview

| Level | What it captures | Click path |
|-------|-----------------|-----------|
| **Enterprise** | Activity across all organizations (plus user events with EMU) | Enterprise → **Settings** → **Audit log** |
| **Organization** | Activity within one organization | Org → **Settings** → **Archive** → **Logs** → **Audit log** |
| **User** | One user's security activity | Profile → **Settings** → **Archives** → **Security log** |

**What's available where:**

| Data | Web UI | JSON/CSV export | REST API | Streaming |
|------|:-:|:-:|:-:|:-:|
| Web events | 180 days | 180 days | 180 days | Your retention |
| Git events | ❌ | ✅ JSON, 7 days | ✅ 7 days (`include=git`) | ✅ |
| API request events | ❌ | ❌ | ❌ | ✅ if enabled |
| SSO responses, workflow runs/jobs, self-hosted runner status | ❌ | ✅ | ✅ | ✅ |

> ⚠️ **Important:** streaming only includes activity from the moment you enable it. Turn it on **before** you need it.

---

## 1️⃣ Access the Enterprise Audit Log

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **Settings** tab → **Audit log**

> 📌 The UI shows the last **three months** by default. Add a `created:` qualifier to see older events (up to 180 days).

### Search syntax

| Filter | Example |
|--------|---------|
| By action | `action:repo.create` |
| By actor | `actor:jsmith` |
| By organization | `org:engineering` |
| By repository | `repo:engineering/auth-service` |
| By date | `created:>2026-09-01` |
| Combined | `action:repo.destroy actor:admin created:>2026-09-01` |

### Common audit log actions

| Action | What it captures |
|--------|-----------------|
| `repo.create` | Repository created |
| `repo.destroy` | Repository deleted |
| `repo.access` | Repository visibility changed |
| `org.invite_member` | User invited to an organization |
| `org.remove_member` | User removed from an organization |
| `team.add_member` | User added to a team |
| `protected_branch.create` | Branch protection rule created |
| `repository_ruleset.create` | Ruleset created |
| `business.sso_response` | Enterprise SAML SSO response |
| `copilot.cfb_seat_added` | Copilot seat assigned |
| `copilot.cfb_seat_cancelled` | Copilot seat removed |
| `ip_allow_list.enable` | IP allow list turned on |

> 💡 Search `action:copilot` for all Copilot plan changes, and `actor:Copilot` for Copilot agent activity.

### Export

- **Audit events:** filter the log if needed → **Export ▾** → choose **JSON** or **CSV**.
- **Git events:** **Export Git Events ▾** → choose a date range → **Download Results** (compressed JSON).

> 📌 Exports are limited to 100 MB compressed or 10 minutes of processing. Git event exports don't include pushes made through the web UI or APIs (for example, merging a pull request in the browser).

---

## 2️⃣ Audit Log Streaming (SIEM Integration)

**👤 Role:** GitHub **enterprise owner** (+ the destination's administrator) · **📍 Portal:** GitHub + your cloud/SIEM

**Navigate:** Enterprise → **Settings** → **Audit log** → **Log streaming**

**Steps:**

1. Prepare the destination and credentials (see the table below and the Action Matrix).
2. Under **Audit log**, click **Log streaming**.
3. Click **Configure stream ▾** and choose your provider.
4. Enter the destination details.
5. Click **Check endpoint**.
6. When the check succeeds, click **Save**.

| Provider | What GitHub asks for |
|----------|----------------------|
| **Amazon S3** | **Access keys** (region, bucket, access key ID, secret key) — or **OpenID Connect** (no long-lived secret) |
| **Azure Blob Storage** | Blob SAS URL with **Create** + **Write** permissions (container fills in automatically) |
| **Azure Event Hubs** | Event hub instance name + connection string |
| **Datadog** | Client token or API key + your Datadog **Site** |
| **Google Cloud Storage** | Bucket name + the service account's JSON key (service account needs **Storage Object Creator** on the bucket) |
| **Splunk** | HTTP Event Collector (HEC) endpoint and token |

> 📌 **Good to know:**
> - Streams include audit **and** Git events for every organization in the enterprise.
> - A paused stream keeps a 7-day buffer. A daily health check emails enterprise owners if a stream is misconfigured — fix it within six days to avoid dropped events.
> - Streaming to multiple endpoints is in public preview. Microsoft Purview is supported for Copilot agent session events only.
> - Delivery is at-least-once, so expect occasional duplicates.

### Recommended stream settings (for incident response)

On Enterprise → **Settings** → **Audit log** → **Settings** tab:

1. Under **API Requests**, select **Enable API Request Events** (streamed only), then click **Save**.
2. Under **Disclose actor IP addresses in audit logs**, select **Enable source IP disclosure**, then click **Save**.

---

## 3️⃣ Audit Log API

Use a token with the `read:audit_log` scope. Each endpoint allows 1,750 queries per hour per user and IP address.

```bash
# Enterprise audit log — events on one day, 100 per page
curl -H "Authorization: Bearer <TOKEN>" \
  "https://api.github.com/enterprises/ENTERPRISE/audit-log?phrase=created:2026-09-01&per_page=100"

# Include Git events (last 7 days) or everything
curl -H "Authorization: Bearer <TOKEN>" \
  "https://api.github.com/enterprises/ENTERPRISE/audit-log?include=all&phrase=action:git.push"

# Organization audit log
curl -H "Authorization: Bearer <TOKEN>" \
  "https://api.github.com/orgs/ORG/audit-log?phrase=action:repo.create"
```

> 💡 Results use cursor-based pagination — follow the `link` header for the next page. Timestamps are UTC epoch milliseconds.

---

## 4️⃣ Git Events

Git events (`git.clone`, `git.fetch`, `git.push`) are collected automatically on GitHub Enterprise Cloud — there's nothing to turn on.

| Where | Availability |
|-------|-------------|
| Audit log UI search | ❌ Not searchable |
| **Export Git Events** | ✅ JSON, last 7 days |
| REST API (`include=git` or `include=all`) | ✅ Last 7 days |
| Streaming | ✅ Kept as long as your destination keeps them |

> ⚠️ **Note:** Git events are high-volume and only kept for **seven days** in GitHub. Stream them if you need them for compliance.

---

## 5️⃣ IP Allow Lists

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **Settings** → **Authentication security** → **IP allow list**

**Steps:**

1. In **IP address or range in CIDR notation**, type an address or range, add a short description, and click **Add**. Repeat for every range — your admins, VPN, CI/CD, and integrations.
2. *(Optional)* Under **Check IP address**, test that an address is allowed.
3. *(Optional)* Select **Enable IP allow list configuration for installed GitHub Apps** so apps can add their own ranges.
4. *(EMU with OIDC only)* Under **IP allow list configuration**, choose **GitHub** (or your IdP's allow list).
5. Select **Enable IP allow list**.
6. Click **Save**.

> ⚠️ **Important:** The allow list blocks web, API, and Git access (including personal access tokens, SSH keys, and app tokens) from addresses that aren't listed. Add every range you and your tooling use **before** enabling it.

---

## 6️⃣ Compliance Checklist

| Control | Where to verify |
|---------|----------------|
| SSO enforced | Enterprise → **Settings** → **Authentication security** (EMU: **Identity provider**) |
| SCIM provisioning active (EMU) | Enterprise → **Identity provider** |
| Audit log streaming configured | Enterprise → **Settings** → **Audit log** → **Log streaming** |
| API request events and source IPs | Enterprise → **Settings** → **Audit log** → **Settings** |
| IP allow list enabled | Enterprise → **Settings** → **Authentication security** |
| Credential inventory reviewed | Enterprise → **Settings** → **Authentication security** → **Credentials** → **Export CSV** (SSH keys, PATs, OAuth and GitHub App tokens) |
| Proof of presence for high-impact actions *(public preview, Entra ID)* | Enterprise → **Settings** → **Authentication security** → **Proof of presence** → **Re-authentication** or **MFA** |
| Secret scanning + push protection | Org → **Settings** → **Advanced Security ▾** → **Configurations** |
| Code scanning enabled | Org → **Security and quality** tab → **Coverage** |
| Copilot content exclusions set | Enterprise → **AI controls** → **Copilot** → **Content exclusion** |
| Required pull request reviews | Org → **Settings** → **Repository** → **Rulesets** (or Enterprise → **Policies** → **Code**) |
| Actions restricted to an allowlist | Enterprise → **Policies** → **Actions** |
| Spend and usage | Enterprise → **Billing and licensing** → **Usage** → **Get usage report** |

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
| **Audit log search returns no events** | The date range, action qualifier, actor, or retention window excludes the event. | Widen the query, search by known action names, and use exported/streamed logs for events older than the UI retention window. |
| **Audit log stream is configured but SIEM receives no events** | Destination credentials, network allow lists, event hub/topic configuration, or stream status is wrong. | Check stream health in GitHub, rotate destination credentials if needed, allow GitHub source IPs, and pause/resume only within documented retention limits. |
| **Ruleset blocks a push or merge unexpectedly** | A branch/tag/push ruleset or legacy branch protection rule targets the ref. | Open the repository rules view for the affected branch/tag, identify the active rule, and either comply with the rule or request a bypass from the owner. |
| **Repository transfer or org rename leaves broken references** | Profile URLs, marketplace/action namespaces, webhooks, secrets, environments, and external integrations may not redirect or transfer. | Inventory dependent systems before the change, update remote URLs and integration settings after the change, and validate webhooks, Actions, Apps, and security configurations. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>

### Q: I configured audit log streaming but events are not appearing in my SIEM. What should I check?
**A:** Open **Log streaming**, edit the stream, and click **Check endpoint**. Common causes: an expired SAS URL or token, missing write permissions, a firewall blocking GitHub's `hooks` IP ranges (from the `meta` API), or a paused stream. Streaming never backfills — it only sends events from when it was enabled. Enterprise owners also get an email when the daily health check fails.

---

### Q: Can we search audit logs older than 180 days in the GitHub UI?
**A:** No. The UI, exports, and API cover 180 days of web events (the UI shows three months unless you add `created:`), and Git events for only 7 days. For longer retention, stream to a SIEM or storage destination — streaming starts from the day you enable it, so set it up early.

---

### Q: Git events are generating massive volume in our SIEM. Is that expected?
**A:** Yes. Git events (`git.clone`, `git.fetch`, `git.push`) fire for every clone, fetch, and push, and a stream includes them automatically. If the volume is too high, filter them at your SIEM's ingestion layer — keep them if your compliance requirements call for Git activity tracking.

---

### Q: The IP allow list locked out our admin. How do we regain access?
**A:** Connect from an allowed address (for example your VPN) and fix the list. If no administrator can reach GitHub from an allowed address, contact GitHub Support. To prevent it, add a "break-glass" range (VPN gateway or emergency access point) and use **Check IP address** before you click **Enable IP allow list**.

---

### Q: How do I find out who deleted a repository or changed its visibility?
**A:** Search the enterprise audit log for `action:repo.destroy` (deletions) or `action:repo.access` (visibility changes). Combine filters, for example `action:repo.destroy actor:username created:>2026-09-01`. Each entry shows the actor, time, and repository — and the source IP if you've enabled source IP disclosure.

---

### Q: Can we use the audit log API to build custom compliance dashboards?
**A:** Yes. Use `GET /enterprises/{enterprise}/audit-log` with the `phrase` parameter (same qualifiers as the UI search) and cursor pagination, with a `read:audit_log` token. For continuous, long-term reporting, streaming into a SIEM or data lake is more reliable than polling — the API is limited to 1,750 queries per hour.

</details>

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Enterprise Environment Scaffolding Checklist | `Setup/Enterprise Environment Scaffolding Checklist.md` |
| Branch Protection Rules & Rulesets | `Governance/Branch Protection Rules & Rulesets.md` |
| Secret Protection Enablement | `Security/Secret Protection Enablement.md` |
| Cost Centers & Department Billing | `Billing/Cost Centers & Department Billing.md` |

---

## 📝 Resources

| Resource | Link |
|----------|------|
| Accessing the audit log for your enterprise | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/admin/monitoring-activity-in-your-enterprise/reviewing-audit-logs-for-your-enterprise/accessing-the-audit-log-for-your-enterprise) |
| Audit log streaming | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/admin/monitoring-activity-in-your-enterprise/reviewing-audit-logs-for-your-enterprise/streaming-the-audit-log-for-your-enterprise) |
| Exporting audit log activity | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/admin/monitoring-activity-in-your-enterprise/reviewing-audit-logs-for-your-enterprise/exporting-audit-log-activity-for-your-enterprise) |
| Using the audit log API | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/admin/monitoring-activity-in-your-enterprise/reviewing-audit-logs-for-your-enterprise/using-the-audit-log-api-for-your-enterprise) |
| Audit log events for your enterprise | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/admin/monitoring-activity-in-your-enterprise/reviewing-audit-logs-for-your-enterprise/audit-log-events-for-your-enterprise) |
| Restricting network traffic with an IP allow list | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/admin/configuring-settings/hardening-security-for-your-enterprise/restricting-network-traffic-to-your-enterprise-with-an-ip-allow-list) |

---

*Last updated: October 2026*
