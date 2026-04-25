# 📋 GitHub Enterprise Audit Log & Compliance Runbook

> How to access, stream, search, and use audit logs for governance, compliance, and security monitoring

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [📋 Overview](#-overview)
- [1️⃣ Access the Enterprise Audit Log](#1-access-the-enterprise-audit-log)
- [2️⃣ Audit Log Streaming (SIEM Integration)](#2-audit-log-streaming-siem-integration)
- [3️⃣ Audit Log API](#3-audit-log-api)
- [4️⃣ Git Events Logging](#4-git-events-logging)
- [5️⃣ IP Allow Lists](#5-ip-allow-lists)
- [6️⃣ Compliance Checklist](#6-compliance-checklist)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Enterprise audit log:** `Enterprise → Settings → Audit log`
- **Set up log streaming:** `Enterprise → Settings → Audit log → Log streaming → Set up a stream`
- **Enable git events:** `Enterprise → Settings → Audit log → Git events → Enable`
- **IP allow list:** `Enterprise → Settings → Authentication security → IP allow list → Add IP range`
- **Download usage report:** `Enterprise → Billing and licensing → Usage report → Download CSV`

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
| Enterprise owner role | ☐ |
| GitHub Enterprise Cloud account | ☐ |
| SIEM endpoint configured (for log streaming) | ☐ |
| SIEM credentials ready (API token, SAS URL, HEC token, etc.) | ☐ |

---

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **GitHub enterprise owner** | Configures audit log streaming in GitHub. | GitHub → profile photo → Your enterprises → [enterprise] → Settings → Audit log → Log streaming → Set up a stream → Select provider → enter destination details → Test endpoint if shown → Save. Handoff: active stream destination and test status. |
| **Azure Storage or Event Hubs administrator** | Creates the Azure destination and provides the exact credential GitHub requires. | For Blob Storage: Azure portal → Storage accounts → [account] → Shared access signature → configure allowed services, resource types, permissions, start, and expiry → Generate SAS and connection string → copy SAS URL. For Event Hubs: Azure portal → Event Hubs → [namespace] → Event Hubs → [hub] → Shared access policies → + Add → choose Send permission → Create → copy connection string or SAS token. Handoff: destination name, region, SAS URL or Event Hubs token, and expiry owner. |

---

## 📋 Overview

GitHub Enterprise Cloud provides audit logging at multiple levels:

| Level | What It Captures | Navigation |
|-------|-----------------|-----------|
| **Enterprise** | All activity across all orgs | Enterprise → Settings → Audit log |
| **Organization** | Activity within one org | Org → Settings → Audit log |
| **User** | Individual user's security activity | Profile → Settings → Security log |

---

## 1️⃣ Access the Enterprise Audit Log

```
Enterprise → Settings → Audit log
```

### Search Syntax

| Filter | Example |
|--------|---------|
| By action | `action:repo.create` |
| By actor | `actor:jsmith` |
| By org | `org:engineering` |
| By repo | `repo:engineering/auth-service` |
| By date | `created:>2026-03-01` |
| Combined | `action:repo.destroy actor:admin created:>2026-03-01` |

### Common Audit Log Actions

| Action | What It Captures |
|--------|-----------------|
| `repo.create` | Repository created |
| `repo.destroy` | Repository deleted |
| `repo.access` | Repository visibility changed |
| `org.invite_member` | User invited to org |
| `org.remove_member` | User removed from org |
| `team.add_member` | User added to team |
| `protected_branch.create` | Branch protection created |
| `business.sso_response` | SAML SSO authentication |
| `copilot.seat_assigned` | Copilot seat assigned |
| `copilot.seat_removed` | Copilot seat removed |

---

## 2️⃣ Audit Log Streaming (SIEM Integration)

Stream audit events in real-time to your SIEM:

```
Enterprise → Settings → Audit log → Log streaming
  → Set up a stream
    → Select provider
      → Configure endpoint
```

### Supported Streaming Destinations

| Provider | Configuration Required |
|----------|----------------------|
| Amazon S3 | Bucket name, access key, secret key, region |
| Azure Blob Storage | SAS URL |
| Azure Event Hubs | Instance name, SAS token |
| Datadog | API URL, API token |
| Google Cloud Storage | Bucket name, JSON key |
| Splunk | HEC URL, HEC token |

> 💡 **Tip:** Set up streaming early — it captures events going forward, not retroactively.

---

## 3️⃣ Audit Log API

Access audit data programmatically:

```bash
# Enterprise audit log (REST)
gh api /enterprises/{enterprise}/audit-log \
  --jq '.[] | {action: .action, actor: .actor, created_at: .@timestamp}'

# Organization audit log (REST)
gh api /orgs/{org}/audit-log?phrase=action:repo.create
```

### GraphQL API (for enterprise)

```graphql
query {
  enterprise(slug: "my-enterprise") {
    auditLog(first: 10) {
      nodes {
        ... on AuditEntry {
          action
          actorLogin
          createdAt
        }
      }
    }
  }
}
```

---

## 4️⃣ Git Events Logging

For git-level events (pushes, clones):

```
Enterprise → Settings → Audit log → Git events
  → Enable git events logging
```

> ⚠️ **Note:** Git events generate high volume. Enable only if required for compliance. Events include: `git.clone`, `git.fetch`, `git.push`.

---

## 5️⃣ IP Allow Lists

Restrict access to your enterprise by IP:

```
Enterprise → Settings → Authentication security → IP allow list
  → Add IP range (CIDR notation)
    → Enable IP allow list
```

> ⚠️ **Important:** Test thoroughly before enabling. Locked-out admins cannot disable the allow list without GitHub Support.

---

## 6️⃣ Compliance Checklist

| Control | Where to Verify |
|---------|----------------|
| SSO enforced | Enterprise → Settings → Authentication security |
| SCIM provisioning active (EMU) | Enterprise → Settings → Identity provider |
| Audit log streaming configured | Enterprise → Settings → Audit log → Log streaming |
| IP allow list enabled | Enterprise → Settings → Authentication security → IP allow list |
| Secret scanning + push protection | Org → Settings → Code security |
| Code scanning enabled | Org → Settings → Code security → Code scanning |
| Copilot content exclusions set | Enterprise → AI controls → Copilot → Content exclusion |
| Required PR reviews enforced | Org → Settings → Rules → Rulesets |
| Actions restricted to allowlist | Enterprise → Settings → Policies → Actions |

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
**A:** Verify the endpoint URL is correct and accessible, confirm the authentication credentials (API token, SAS URL, etc.) are valid and not expired, and ensure streaming is enabled (not paused) in the enterprise settings. Also note that audit log streaming only captures events going forward from the moment it is enabled -- it does not backfill historical events. Check your SIEM's ingestion logs for connection errors.

---

### Q: Can we search audit logs older than 6 months in the GitHub UI?
**A:** The enterprise audit log UI retains events for the most recent 6 months. For longer retention, configure audit log streaming to a SIEM or storage destination (S3, Splunk, Azure Blob, etc.) before the 6-month window elapses. Once events age out of the UI, they can only be queried from your SIEM or storage backend.

---

### Q: We enabled git events logging and it is generating massive volume. Is that expected?
**A:** Yes, git events (`git.clone`, `git.fetch`, `git.push`) are very high-volume because they fire for every developer interaction with repositories. Enable git events logging only if your compliance or security requirements specifically mandate tracking these operations. If the volume is overwhelming your SIEM, consider filtering git events at the SIEM ingestion layer rather than disabling them entirely.

---

### Q: The IP allow list locked out our admin. How do we regain access?
**A:** If all admins are locked out due to an IP allow list misconfiguration, contact GitHub Support for assistance. To prevent this in the future, always include a "break-glass" IP range (such as a VPN gateway or known emergency access point) in the allow list before enabling it. Test the allow list thoroughly with a subset of users before enforcing it enterprise-wide.

---

### Q: How do I find out who deleted a repository or changed its visibility?
**A:** Search the enterprise audit log using `action:repo.destroy` for deletions or `action:repo.access` for visibility changes. You can combine filters such as `action:repo.destroy actor:username created:>2026-01-01` to narrow results. The audit log entry will show the actor, timestamp, and the affected repository.

---

### Q: Can we use the audit log API to build custom compliance dashboards?
**A:** Yes, both the REST API and GraphQL API support querying audit log events programmatically. Use the REST endpoint at `/enterprises/{enterprise}/audit-log` with query parameters to filter by action, actor, date, and organization. Pipe results into your dashboard tooling or data warehouse for custom compliance reporting.

</details>

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
| About the audit log | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/admin/monitoring-activity-in-your-enterprise/reviewing-audit-logs-for-your-enterprise/about-the-audit-log-for-your-enterprise) |
| Audit log streaming | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/admin/monitoring-activity-in-your-enterprise/reviewing-audit-logs-for-your-enterprise/streaming-the-audit-log-for-your-enterprise) |
| Audit log API | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/rest/enterprise-admin/audit-log) |
| IP allow lists | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/admin/enforcing-policies/enforcing-policies-for-your-enterprise/enforcing-policies-for-security-settings-in-your-enterprise#managing-allowed-ip-addresses-for-organizations-in-your-enterprise) |
| GitHub security features | [GitHub Docs](https://docs.github.com/en/enterprise-cloud@latest/code-security/getting-started/github-security-features) |

---

*Last updated: April 2026*
