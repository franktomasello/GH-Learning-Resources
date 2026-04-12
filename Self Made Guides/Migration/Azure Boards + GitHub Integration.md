# 🔗 Azure Boards + GitHub Integration Runbook

> Complete guide to connecting Azure Boards with GitHub for work item tracking while migrating code to GitHub

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Connect Boards to GitHub:** `Azure DevOps → Project Settings → Boards → GitHub connections → Connect`
- **Link work items:** Use `AB#1234` syntax in commit messages, PR titles, or branch names
- **Configure state transitions:** `Azure DevOps → Project Settings → Boards → GitHub connections → Configure`

---

## ✅ Prerequisites

| Requirement | Status |
|-------------|--------|
| Azure DevOps project with Azure Boards enabled | ☐ |
| GitHub organization with target repositories | ☐ |
| Permissions to configure GitHub connections in Azure DevOps project settings | ☐ |
| GitHub App connection method configured (recommended over OAuth) | ☐ |

---

## 📋 Overview

Keep work items in Azure Boards while hosting code in GitHub. Link commits, branches, and PRs back to Boards work items using `AB#` syntax.

---

## 1️⃣ Connect Azure Boards to GitHub

### From Azure DevOps Side

```
Azure DevOps → Project Settings → Boards → GitHub connections
  → Connect your GitHub account or organization
    → Authorize Azure Boards
      → Select repositories to connect
```

### Authentication Options

| Method | Best For |
|--------|----------|
| OAuth (personal) | Individual developers, testing |
| GitHub App (recommended) | Organization-wide, production use |

> 💡 **Tip:** The GitHub App method is recommended for enterprise — it provides org-level access without depending on individual user tokens.

---

## 2️⃣ Link Work Items to GitHub Activity

### Using AB# Syntax

Reference Azure Boards work items from GitHub commits, branches, and PRs:

| GitHub Action | Syntax | Example |
|--------------|--------|---------|
| Commit message | `AB#<work-item-id>` | `git commit -m "Fix login bug AB#1234"` |
| PR title/body | `AB#<work-item-id>` | PR title: "Add auth flow AB#1234" |
| Branch name | Include work item ID | `feature/AB#1234-login-fix` |

> ✅ **Result:** The work item in Azure Boards automatically shows the linked GitHub commit, PR, or branch with status.

### State Transitions

Configure Azure Boards to automatically update work item state based on GitHub PR status:
- PR opened → Work item moves to "In Progress"
- PR merged → Work item moves to "Done"

Configure at: Azure DevOps → Project Settings → Boards → GitHub connections → select connection → Configure

---

## 3️⃣ Common Patterns for Migration

### Pattern 1: Boards + GitHub (Bridge Model)
- Keep all planning/tracking in Azure Boards
- Move source control and CI/CD to GitHub
- Use `AB#` links to maintain traceability
- Best for: teams migrating incrementally

### Pattern 2: Full GitHub Migration
- Migrate work items to GitHub Issues or GitHub Projects
- Use GitHub Enterprise Importer for ADO work item migration where supported
- Best for: teams ready for full platform consolidation

### Pattern 3: Hybrid Long-Term
- Some teams stay on Boards, others move to GitHub Issues
- Cross-link via `AB#` syntax
- Best for: large enterprises with mixed team preferences

---

## 4️⃣ Best Practices

- Use the **GitHub App** connection method (not OAuth) for production
- Establish `AB#` linking as a team convention from day one
- Configure **state transitions** so work items auto-update when PRs are merged
- Set up a **shared dashboard** in Azure DevOps that shows GitHub-linked work items

---

## ❓ Common Questions & Troubleshooting

### Q: I am using AB# syntax in commits and PRs but the links are not appearing in Azure Boards. What is wrong?
**A:** Verify that the GitHub connection is properly configured in Azure DevOps project settings (Project Settings > Boards > GitHub connections). Confirm that the specific repository is included in the connection -- not all repos may be connected by default. Also ensure the `AB#` prefix is followed by a valid work item ID with no spaces (e.g., `AB#1234`, not `AB# 1234`).

---

### Q: Work item state transitions are not triggering automatically when PRs are merged. How do I fix this?
**A:** State transitions must be explicitly configured in the Azure Boards GitHub connection settings. Navigate to Azure DevOps > Project Settings > Boards > GitHub connections, select your connection, and click Configure. Map the PR events (opened, merged, closed) to the desired work item state transitions. Without this configuration, `AB#` links will appear but state changes will not be automatic.

---

### Q: Can I connect multiple GitHub repositories to a single Azure Board?
**A:** Yes. Multiple GitHub repositories can be connected to the same Azure DevOps project and Board. Each repository's commits, PRs, and branches can link to the same work items using `AB#` syntax. Add additional repos through Project Settings > Boards > GitHub connections > Add repositories.

---

### Q: Should I use the OAuth connection or the GitHub App connection method?
**A:** The GitHub App connection is recommended for organization-wide production use. OAuth connections are tied to an individual user's token and will break if that user leaves the organization or revokes their token. The GitHub App method provides org-level access that is independent of any individual user.

---

### Q: We are seeing duplicate or stale links between Azure Boards work items and GitHub. How do we clean this up?
**A:** Stale links can occur if repositories are renamed, transferred, or deleted after the connection was established. Review the GitHub connection in Azure DevOps project settings and remove any disconnected or renamed repositories, then re-add them with their current names. For duplicate links on individual work items, manually remove the incorrect links from the work item's "Links" tab in Azure Boards.

---

### Q: Can I migrate our Azure Boards work items to GitHub Issues eventually?
**A:** Yes, GitHub Enterprise Importer (GEI) supports migrating ADO work items to GitHub Issues. However, many teams prefer the bridge model -- keeping planning in Azure Boards while moving code to GitHub -- during a transitional period. When ready for full migration, use GEI to migrate work items and then retire the Azure Boards connection.

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| GitHub Enterprise Importer (GEI) & Actions Importer | `Migration/GitHub Enterprise Importer (GEI) & Actions Importer.md` |
| Organization Design Patterns (Flat Structure, Teams, Naming) | `Setup/Organization Design Patterns (Flat Structure, Teams, Naming).md` |
| Standard Enterprise to EMU Migration | `Setup/Standard Enterprise to EMU Migration.md` |

---

## 📝 Resources

| Resource | Link |
|----------|------|
| Azure Boards + GitHub overview | [Microsoft Learn](https://learn.microsoft.com/en-us/azure/devops/boards/github/overview) |
| Connect Azure Boards to GitHub | [Microsoft Learn](https://learn.microsoft.com/en-us/azure/devops/boards/github/connect-to-github) |
| AB# linking syntax | [Microsoft Learn](https://learn.microsoft.com/en-us/azure/devops/boards/github/link-to-from-github) |
| Planning migration to GitHub | [GitHub Docs](https://docs.github.com/en/migrations/overview/planning-your-migration-to-github) |

---

*Last updated: April 2026*
