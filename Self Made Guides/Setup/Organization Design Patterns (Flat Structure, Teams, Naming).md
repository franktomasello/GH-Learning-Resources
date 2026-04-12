# 🏗️ GitHub Organization Design Patterns Guide

> Best practices for structuring enterprises, organizations, teams, and repositories — especially when migrating from GitLab or Azure DevOps

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Org structure:** Keep 3-7 orgs max — align to business units or compliance boundaries, not teams or projects
- **Repo naming:** Use `team-project-component` pattern (e.g., `platform-auth-service`) for alphabetical grouping
- **Teams:** Organization → Teams → New team → Set parent for nesting → Sync with IdP groups for automated membership
- **Custom properties:** Enterprise Settings → Custom properties → Define metadata fields (department, data-classification, owner-team)
- **Internal repos:** Set visibility to Internal for cross-org discoverability within the enterprise

---

## ✅ Prerequisites

| Requirement | Status |
|-------------|--------|
| GitHub Enterprise Cloud account created | ☐ |
| Enterprise owner or org owner access | ☐ |
| Organization naming and boundary decisions made | ☐ |
| Repository naming convention agreed upon by teams | ☐ |
| IdP groups defined for team sync (if using SCIM) | ☐ |

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

## ❓ Common Questions & Troubleshooting

### Q: We migrated from GitLab and created one org per GitLab subgroup — now we have 50+ orgs. Is that a problem?
**A:** Yes. GitHub has a flatter hierarchy than GitLab, and creating one org per subgroup leads to excessive admin overhead, fragmented policies, and poor cross-org discoverability. Consolidate to 3-7 organizations aligned to major business units or compliance boundaries. Use teams (including nested teams) for the granularity that GitLab subgroups provided. Use topics, naming conventions, and custom properties to organize repos within each org.

---

### Q: We established a naming convention but nobody is following it — how do we enforce it?
**A:** GitHub does not natively enforce repo naming conventions at the platform level. Options include: (1) use a GitHub App or Action that runs on `repository.created` webhook events and renames or flags non-compliant repos, (2) restrict repository creation to org owners or a platform team who enforce the convention manually, (3) use repository templates with pre-set names that follow the pattern. Custom properties can supplement naming by providing structured metadata even if the repo name is imperfect.

---

### Q: Topics are not discoverable — our developers cannot find repos by topic. What is the best approach?
**A:** Topics are searchable via `https://github.com/orgs/YOUR_ORG/repositories?q=topic:TOPIC_NAME` but this URL is not always obvious. Improve discoverability by: (1) documenting your topic taxonomy in a central README or wiki, (2) using the organization-level Repositories tab which supports topic filtering, (3) pinning key repos to the organization profile page, and (4) using custom properties for governance metadata (topics for informal categorization, custom properties for formal metadata).

---

### Q: When should we use enterprise teams vs organization teams?
**A:** Use **organization teams** for repository access control within a single org — this is the most common use case. Use **enterprise teams** when you need to grant access to repos across multiple organizations or assign Copilot licenses at the enterprise level without requiring org membership. Enterprise teams span the entire enterprise and are managed by enterprise owners. If all your repos are in one org, organization teams are sufficient.

---

### Q: We set custom properties at the enterprise level, but they are not showing up on our repos — what is wrong?
**A:** Custom properties defined at the enterprise level are available on all repos across all orgs, but property values must be set on each repository individually (or in bulk via the API). Simply defining a custom property does not auto-populate values. Navigate to individual repos > Settings > Custom properties to set values, or use the REST API to set values in bulk. Also verify the property was created at the correct level (enterprise vs org).

---

### Q: How do we handle a shared "monorepo" that multiple teams need access to across orgs?
**A:** Place the monorepo in the organization where it is most naturally managed (typically the platform or engineering org). Set visibility to **internal** so all enterprise members can read it. Use team-based permissions for write/maintain/admin access. If specific teams from other orgs need write access, either add those users to a team in the repo's org, or use enterprise teams to grant cross-org access. Avoid duplicating the repo across multiple orgs.

---

### Q: We want to restrict who can create repos in our organization — is that possible?
**A:** Yes. Go to Organization > Settings > Member privileges > Repository creation. You can restrict repo creation to org owners only, or allow all members to create repos with specific visibility constraints (e.g., members can create private repos but not public or internal). For tighter control, restrict creation to owners and use a request workflow (e.g., a GitHub Issue template or a Slack integration) for teams to request new repos.

---

### Q: How should we structure teams when we have both permanent staff and contractors?
**A:** Create separate teams for contractors (e.g., `contractor-team-x`) with limited permissions (read or write, never admin). In EMU environments, contractors can be provisioned via SCIM like regular users, or invited as guest collaborators with access to specific repos. Use nested teams to group contractors under a parent team for easy auditing. Set up a process to review and remove contractor access when engagements end — IdP group sync makes this automatic when contractors are removed from the IdP group.

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Enterprise Environment Scaffolding Checklist | `Setup/Enterprise Environment Scaffolding Checklist.md` |
| GitHub Enterprise Importer | `Migration/GitHub Enterprise Importer (GEI) & Actions Importer.md` |

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
