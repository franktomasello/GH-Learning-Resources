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
