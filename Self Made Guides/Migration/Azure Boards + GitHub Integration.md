# 🔗 GitHub + Azure Boards Integration Runbook

> Complete guide to connecting Azure Boards with GitHub for work item tracking while migrating code to GitHub

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Connect Boards to GitHub:** `Azure DevOps → Project Settings → Boards → GitHub connections → Connect`
- **Link work items:** Use `AB#1234` syntax in commit messages, PR titles, or branch names
- **Configure state transitions:** `Azure DevOps → Project Settings → Boards → GitHub connections → Configure`

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

## 🧯 Known Errors & Resolutions

> This section lists the known product errors and admin-facing symptoms that commonly occur with this workflow. Exact message text can vary by product rollout, tenant policy, and provider, so use the log or settings page named in the resolution to confirm the root cause.

| Error or symptom | Likely cause | Resolution |
|------------------|--------------|------------|
| **Page, tab, or button is missing** | Wrong account context, missing admin role, unavailable plan/add-on, or feature rollout not enabled for the selected enterprise/org/repo. | Switch to the correct account and scope, confirm the prerequisite role, verify licensing or add-on activation, then refresh the page. If the control is still absent, use the direct settings URL from the relevant GitHub Docs page and confirm the feature is available for your plan. |
| **Changes appear saved but behavior does not change** | Policy inheritance, cached UI state, propagation delay, or an overlapping enterprise/org/repo policy. | Reopen the settings page, verify the effective policy at the lowest affected scope, wait for propagation where documented, and check for a stricter policy at an enterprise or organization level. |
| **403, forbidden, or resource not accessible** | The signed-in user or token can see the page but lacks the specific permission for the action. | Use an enterprise owner, organization owner, repository admin, or token with the exact scopes/permissions listed in the runbook. For SAML-protected orgs, authorize the token or SSH key for SSO before retrying. |
| **AB# links do not appear in commits or pull requests** | The repository is not connected to the Azure DevOps project, the work item ID is invalid, or the syntax has spaces. | Open Project settings > GitHub connections, confirm the repo is connected once, and use `AB#123` with no space. |
| **Repository is not available to connect** | The GitHub identity is not a repository admin, SAML SSO is not authorized, or the repo is already connected elsewhere. | Use an admin account or SSO-authorized PAT, remove stale duplicate connections, and reconnect the repository from the intended Azure DevOps project. |
| **Work item state does not change when PRs merge** | State transition rules were not configured for the GitHub connection. | Open the connection configuration and map PR events such as opened, closed, and merged to the desired Azure Boards states. |
| **Duplicate or stale work item links appear** | The repository was renamed, transferred, deleted, or connected to multiple Azure DevOps organizations. | Remove stale connections, reconnect the current repository, and clean duplicate links from the work item Links tab. |

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
| Azure Boards + GitHub overview | [Microsoft Learn](https://learn.microsoft.com/en-us/azure/devops/boards/github/?view=azure-devops) |
| Connect Azure Boards to GitHub | [Microsoft Learn](https://learn.microsoft.com/en-us/azure/devops/boards/github/connect-to-github) |
| AB# linking syntax | [Microsoft Learn](https://learn.microsoft.com/en-us/azure/devops/boards/github/link-to-from-github) |
| Planning migration to GitHub | [GitHub Docs](https://docs.github.com/en/migrations/overview/planning-your-migration-to-github) |

---

*Last updated: April 2026*
