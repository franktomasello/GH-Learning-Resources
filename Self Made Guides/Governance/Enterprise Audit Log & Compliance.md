# 📋 GitHub Enterprise Audit Log & Compliance Guide

> How to access, stream, search, and use audit logs for governance, compliance, and security monitoring

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

---

## ❓ Common Questions & Troubleshooting

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
