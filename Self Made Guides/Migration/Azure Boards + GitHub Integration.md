# 🔗 Azure Boards + GitHub Integration Runbook

> Complete guide to connecting Azure Boards with GitHub for work item tracking while migrating code to GitHub

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

## 📝 Resources

| Resource | Link |
|----------|------|
| Azure Boards + GitHub overview | [Microsoft Learn](https://learn.microsoft.com/en-us/azure/devops/boards/github/overview) |
| Connect Azure Boards to GitHub | [Microsoft Learn](https://learn.microsoft.com/en-us/azure/devops/boards/github/connect-to-github) |
| AB# linking syntax | [Microsoft Learn](https://learn.microsoft.com/en-us/azure/devops/boards/github/link-to-from-github) |
| Planning migration to GitHub | [GitHub Docs](https://docs.github.com/en/migrations/overview/planning-your-migration-to-github) |

---

*Last updated: April 2026*
