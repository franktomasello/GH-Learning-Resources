# 🏗️ GitHub Organization Design Patterns Guide

> Best practices for structuring enterprises, organizations, teams, and repositories — especially when migrating from GitLab or Azure DevOps

---

## 📋 Overview

GitHub uses a flatter hierarchy than GitLab or Azure DevOps. Understanding the mapping is critical for a clean migration.

| Level | GitHub | GitLab Equivalent | Azure DevOps Equivalent |
|-------|--------|-------------------|------------------------|
| Top-level | **Enterprise** | Instance/Group | Organization |
| Business unit | **Organization** | Top-level Group | Project |
| Access group | **Team** (nested) | Subgroup | Team |
| Code unit | **Repository** | Project | Repository |

> 💡 **Key difference:** GitHub repos live directly under an org — there is no nested folder structure. Use naming, topics, teams, and custom properties for organization.

## 1️⃣ Enterprise & Organization Structure

### Recommended Pattern

```
Enterprise (single admin boundary)
  ├── Org: engineering          (core development teams)
  ├── Org: data-science         (analytics, ML teams)
  ├── Org: platform             (shared workflows, templates, actions)
  ├── Org: security             (security tooling, policies)
  └── Org: sandbox              (experimentation, training)
```

### When to Create Separate Orgs

| Reason | Separate Orgs? |
|--------|---------------|
| Different departments/business units | Yes — one org per major unit |
| Different cost centers for billing | Yes — cost centers are org-level |
| Different security/compliance requirements | Yes — org-level policies differ |
| Different IdPs (non-EMU only) | Yes — per-org SAML in standard enterprise |
| Sub-teams within a department | No — use nested teams instead |
| Project groupings | No — use topics and naming conventions |

### When to Use One Enterprise vs Multiple

| Scenario | Recommendation |
|----------|---------------|
| Single IdP, shared billing | One enterprise |
| Multiple IdPs that can federate into one Entra tenant | One enterprise (EMU) |
| Multiple IdPs that CANNOT federate | Multiple enterprises OR standard enterprise with per-org SAML |
| Hard compliance boundaries (FedRAMP vs non-FedRAMP) | Multiple enterprises |
| Completely independent billing and governance | Multiple enterprises |

## 2️⃣ Repository Naming Conventions

### Recommended Pattern: `team-project-component`

| Example | Team | Project | Component |
|---------|------|---------|-----------|
| `platform-auth-service` | platform | auth | service |
| `data-etl-pipeline` | data | etl | pipeline |
| `mobile-ios-app` | mobile | ios | app |
| `infra-terraform-modules` | infra | terraform | modules |

> 💡 **Tip:** Prefix with team or domain name so repos sort together alphabetically.

## 3️⃣ Topics for Discoverability

Tag repos with topics for filtering and search:

```
Repository → Settings → scroll to Topics
  → Add topics (e.g., "python", "api", "production", "team-platform")
```

### Suggested Topic Taxonomy

| Category | Example Topics |
|----------|---------------|
| Language | `python`, `java`, `typescript`, `go` |
| Type | `api`, `library`, `service`, `cli`, `docs` |
| Team | `team-platform`, `team-data`, `team-mobile` |
| Environment | `production`, `staging`, `internal` |
| Status | `active`, `deprecated`, `archived` |

## 4️⃣ Nested Teams for Access Control

```
Organization → Teams → New team
  → Set parent team (for nesting)
```

### Example Team Hierarchy

```
engineering (parent team — broad read access)
  ├── backend (write access to backend repos)
  │   ├── backend-auth (maintain access to auth service)
  │   └── backend-payments (maintain access to payments)
  ├── frontend (write access to frontend repos)
  └── devops (admin access to infra repos)
```

- Parent team permissions cascade to child teams
- Child teams can have additional permissions
- Sync teams from IdP groups (Entra ID, Okta) for automatic membership

### IdP Group Sync

```
Organization → Settings → Teams → select team
  → IdP group sync → Connect IdP group
```

## 5️⃣ Custom Properties (Enterprise Feature)

Add structured metadata to repos beyond topics:

```
Enterprise Settings → Custom properties
  → New property (e.g., "department", "data-classification", "owner-team")
```

- Custom properties are searchable and filterable
- Can be used in rulesets to target repos by property value
- Better than topics for formal governance metadata

## 6️⃣ Internal Repositories (Enterprise Feature)

```
Repository → Settings → Danger Zone → Change visibility
  → Internal
```

| Visibility | Who Can See |
|-----------|-------------|
| **Private** | Only explicitly granted users/teams |
| **Internal** | All members of the enterprise (across all orgs) |
| **Public** | Everyone on the internet |

> 💡 **Tip:** Use **Internal** for shared libraries, standards, and tools that should be visible enterprise-wide without explicit access grants.

## 7️⃣ Enterprise Teams (Cross-Org Collaboration)

```
Enterprise → People → Teams → Create team
  → Add members from any org in the enterprise
```

- Enterprise teams span multiple organizations
- Grant access to repos across orgs without moving repos
- Sync with IdP groups for automatic membership
- Only enterprise owners or designated team maintainers can manage

---

## 📝 Resources

| Resource | Link |
|----------|------|
| About organizations | [docs.github.com](https://docs.github.com/en/organizations/collaborating-with-groups-in-organizations/about-organizations) |
| About teams | [docs.github.com](https://docs.github.com/en/organizations/organizing-members-into-teams/about-teams) |
| Custom properties | [docs.github.com](https://docs.github.com/en/organizations/managing-organization-settings/managing-custom-properties-for-repositories-in-your-organization) |
| Repository visibility | [docs.github.com](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/managing-repository-settings/setting-repository-visibility) |
| Enterprise teams | [docs.github.com](https://docs.github.com/en/enterprise-cloud@latest/admin/managing-accounts-and-repositories/managing-groups-of-people-with-roles/managing-teams-in-your-enterprise) |

---

*Last updated: April 2026*
