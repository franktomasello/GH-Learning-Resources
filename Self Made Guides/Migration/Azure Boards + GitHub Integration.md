# 🔗 GitHub + Azure Boards Integration Runbook

> Complete guide to connecting Azure Boards with GitHub for work item tracking while migrating code to GitHub

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [📋 Overview](#-overview)
- [1️⃣ Connect Azure Boards to GitHub](#1️⃣-connect-azure-boards-to-github)
- [2️⃣ Link Work Items to GitHub Activity](#2️⃣-link-work-items-to-github-activity)
- [3️⃣ Common Patterns for Migration](#3️⃣-common-patterns-for-migration)
- [4️⃣ Best Practices](#4️⃣-best-practices)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Connect:** `Azure DevOps → [project] → Project settings → GitHub connections` → **Connect your GitHub account** (first time) or **New connection** → pick the org → select repos → **Save** → on GitHub, **Approve, Install, & Authorize**
- **Link work items:** put `AB#1234` in a **commit message**, **pull request description**, or **issue description**
- **Move work items automatically:** `Fixes AB#1234` (or a state name, like `Closed AB#1234`) — applied when the PR merges into the **default branch**
- **Branch from a work item:** work item → **⋯** → **New GitHub branch** → name, repo, base → **Create**

---

## ✅ Accuracy & Click-Path Notes

<details>
<summary><em>Show click-path conventions</em></summary>


- Reviewed against current public GitHub and Microsoft documentation in October 2026 where public documentation is available. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
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
| Azure DevOps Services project with Azure Boards (Azure DevOps Server only works with GitHub Enterprise Server) | — | ☐ |
| Create the GitHub connection | Member of **Project Collection Administrators** (or the project's creator) **and** **admin** of each GitHub repository | ☐ |
| Link work items | **Contributor** access to both the Boards project and the GitHub repository | ☐ |
| Install the **Azure Boards** GitHub App on the organization | GitHub **organization owner** (or repo admin for selected repos) | ☐ |
| SAML-protected org with a PAT connection | The PAT must be authorized for SSO | ☐ |

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **Azure DevOps Project Administrator** | Creates the Azure Boards connection to GitHub from the project. | Azure DevOps → [organization] → [project] → Project settings → GitHub connections → Connect your GitHub account (first time) or New connection → GitHub → select the organization → keep the repositories to connect → Save → on GitHub, Approve, Install, & Authorize. Handoff: connected repository list in GitHub connections. |
| **GitHub organization owner** | Authorizes or installs the Azure Boards GitHub App for the selected repositories. | GitHub authorization prompt (Approve, Install, & Authorize), or later GitHub → Organizations → [org] → Settings → under Third-party Access, GitHub Apps → Azure Boards → Configure → Repository access → select repositories → Save. Handoff: Azure Boards app installed for the expected repos. |
| **Team/project lead** | Confirms developers use the work-item linking syntax after the connection exists. | Azure Boards → Boards → Work Items → [work item] → the Development section should show linked commits, branches, or pull requests after developers include `AB#<id>` in a commit message or pull request description. Handoff: test PR or commit linked to a work item. |

---

## 📋 Overview

Keep work items in Azure Boards while hosting code in GitHub. Commits, pull requests, branches, and issues link back to work items through the **Azure Boards app for GitHub**.

| What you get | How |
|--------------|-----|
| Links from GitHub to Boards | `AB#ID` in commit messages, PR descriptions, or issue descriptions |
| Automatic state changes | Keywords (`fix`, `fixes`, `fixed`) or state names before `AB#ID`, on merge to the default branch |
| Branches from work items | **New GitHub branch** on the work item |
| PR status checks and Copilot integration | Require the GitHub App connection (not a PAT) |

---

## 1️⃣ Connect Azure Boards to GitHub

**👤 Role:** **Project Collection Administrator** + GitHub **repository admin** · **📍 Portal:** Azure DevOps, then GitHub

**Navigate:** `https://dev.azure.com/{organization}/{project}` → **Project settings** → **GitHub connections**

**Steps:**

1. Click **Connect your GitHub account** (first connection) or **New connection** (later ones) and choose **GitHub**.
2. Sign in to GitHub with an account that **administers** the repositories.
3. Select the GitHub organization. Only organizations you own or administer appear.
4. In **Add GitHub Repositories**, keep the repositories you want (all repos you administer are preselected) and clear the rest.
5. Click **Save**.
6. On GitHub, click **Approve, Install, & Authorize** and confirm your credentials.

> ✅ **Result:** the connection lists the selected repositories (up to 2,000 per connection).

### Authentication options

| Method | Notes |
|--------|-------|
| **GitHub account (installs the Azure Boards app)** | Recommended. Required for PR status checks and the GitHub Copilot integration |
| **Personal access token** | Fine-grained (Metadata read, Contents/Webhooks/Pull requests read-write, Issues optional, org Members read) or classic (`repo`, `read:user`, `user:email`, `admin:repo_hook`). Authorize it for SSO on SAML orgs. No PR status checks or Copilot integration |
| **OAuth app** | Only for GitHub Enterprise Server |

> 💡 **Tip:** connect each GitHub repository to projects in **one** Azure DevOps organization only — otherwise `AB#` mentions can link unexpectedly.

---

## 2️⃣ Link Work Items to GitHub Activity

### Using `AB#` syntax

| Where | Works? | Example |
|-------|--------|---------|
| **Commit message** | ✅ | `git commit -m "Fix login bug AB#1234"` |
| **Pull request description** | ✅ | `Adds the auth flow. AB#1234` |
| **GitHub issue description** | ✅ | `Tracked in AB#1234` |
| Pull request **title** or **comments** | ❌ | No link is created |

> ✅ **Result:** the work item's **Development** section shows the linked commit, pull request, or issue. Links from a PR description also appear in the PR's Development section on GitHub.

### Branches

Create the branch from the work item so it's linked automatically:

1. On the board, open the work item's actions (**⋯**) → **New GitHub branch**.
2. Enter the branch name, pick the repository and base branch.
3. Click **Create**.

You can also add a link to an existing branch, commit, or PR from the work item (**Add link** → GitHub branch / commit / pull request).

### Automatic state transitions

Put a keyword or state name right before the `AB#` reference in a commit message or PR description:

| Text | Result (when merged into the default branch) |
|------|---------------------------------------------|
| `AB#123` | Link only |
| `Fixes AB#123` / `Fixed AB#123` | Moves 123 to the first **Resolved**-category state (or **Completed** if none) |
| `Closed AB#123` | Moves 123 to the **Closed** state, if it exists |
| `Fixes AB#123, AB#124` | Transitions only 123 — repeat the keyword for each item (`Fixes AB#123, Fixes AB#124`) |

> ⚠️ Transitions only happen when the pull request merges into the **default branch**. There's no separate "map PR events to states" screen.

---

## 3️⃣ Common Patterns for Migration

### Pattern 1: Boards + GitHub (Bridge Model)
- Keep all planning/tracking in Azure Boards
- Move source control and CI/CD to GitHub
- Use `AB#` links to maintain traceability
- Best for: teams migrating incrementally

### Pattern 2: Full GitHub Migration
- Move planning to GitHub Issues and GitHub Projects
- GitHub Enterprise Importer migrates **repositories** (Git history, pull requests, PR work-item links, branch policies) — it does **not** migrate Azure Boards work items; plan a separate approach (scripts or third-party tools) for those
- Best for: teams ready for full platform consolidation

### Pattern 3: Hybrid Long-Term
- Some teams stay on Boards, others move to GitHub Issues
- Cross-link via `AB#` syntax
- Best for: large enterprises with mixed team preferences

---

## 4️⃣ Best Practices

- Connect with a **GitHub account** (Azure Boards app), not a PAT, for production
- Make `AB#` in commit messages and PR descriptions a team convention from day one — add it to your PR template
- Use `Fixes AB#ID` in PR descriptions so work items move when PRs merge to the default branch
- Create branches from work items (**New GitHub branch**) to link them automatically
- Connect each repository to **one** Azure DevOps organization
- Build an Azure DevOps dashboard that shows GitHub-linked work items

## 🧯 Known Errors & Resolutions

<details>
<summary><em>Show known errors table</em></summary>


> This section lists the known product errors and admin-facing symptoms that commonly occur with this workflow. Exact message text can vary by product rollout, tenant policy, and provider, so use the log or settings page named in the resolution to confirm the root cause.

| Error or symptom | Likely cause | Resolution |
|------------------|--------------|------------|
| **Page, tab, or button is missing** | Wrong account context, missing admin role, unavailable plan/add-on, or feature rollout not enabled for the selected enterprise/org/repo. | Switch to the correct account and scope, confirm the prerequisite role, verify licensing or add-on activation, then refresh the page. If the control is still absent, use the direct settings URL from the relevant GitHub Docs page and confirm the feature is available for your plan. |
| **Changes appear saved but behavior does not change** | Policy inheritance, cached UI state, propagation delay, or an overlapping enterprise/org/repo policy. | Reopen the settings page, verify the effective policy at the lowest affected scope, wait for propagation where documented, and check for a stricter policy at an enterprise or organization level. |
| **403, forbidden, or resource not accessible** | The signed-in user or token can see the page but lacks the specific permission for the action. | Use an enterprise owner, organization owner, repository admin, or token with the exact scopes/permissions listed in the runbook. For SAML-protected orgs, authorize the token or SSH key for SSO before retrying. |
| **AB# links do not appear in commits or pull requests** | The repository isn't connected, the ID is invalid, there's a space (`AB# 123`), or `AB#` is in the PR title or a comment. | Check Project settings > GitHub connections, and put `AB#123` in the commit message or PR description. |
| **Repository is not available to connect** | The GitHub identity is not a repository admin, SAML SSO is not authorized, or the repo is already connected elsewhere. | Use an admin account or SSO-authorized PAT, remove stale duplicate connections, and reconnect the repository from the intended Azure DevOps project. |
| **Work item state does not change when PRs merge** | No keyword before `AB#`, the PR merged into a non-default branch, or the target state doesn't exist in the process. | Use `Fixes AB#123` (or a valid state name) in the PR description and merge into the default branch. |
| **Duplicate or stale work item links appear** | The repository was renamed, transferred, deleted, or connected to multiple Azure DevOps organizations. | Remove stale connections, reconnect the current repository, and clean duplicate links from the work item Links tab. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>


### Q: I am using AB# syntax in commits and PRs but the links are not appearing in Azure Boards. What is wrong?
**A:** Check that the repository is listed under **Project settings** → **GitHub connections**, that `AB#` is in the **commit message** or **PR description** (not the PR title or a comment), and that the ID is valid with no space (`AB#1234`, not `AB# 1234`). If the repo is connected to projects in more than one Azure DevOps organization, links can go to the wrong place.

---

### Q: Work item state transitions are not triggering automatically when PRs are merged. How do I fix this?
**A:** Transitions come from the text, not a settings screen. Put `Fixes AB#1234` (or a valid state name, such as `Closed AB#1234`) in the PR description or commit message, and merge into the **default** branch. Repeat the keyword for each work item you want to move.

---

### Q: Can I connect multiple GitHub repositories to a single Azure Board?
**A:** Yes — up to 2,000 repositories per connection, and a project can have several connections. Add repositories from **Project settings** → **GitHub connections** → the connection's **⋯** menu, or change the Azure Boards app's repository access on GitHub.

---

### Q: Should I connect with a GitHub account (app) or a personal access token?
**A:** Use the GitHub account connection, which installs the Azure Boards app — it's recommended and required for PR status checks and the GitHub Copilot integration. PAT connections depend on one person's token (which can expire or be revoked) and lack those features. OAuth apps are only for GitHub Enterprise Server.

---

### Q: We are seeing duplicate or stale links between Azure Boards work items and GitHub. How do we clean this up?
**A:** Stale links can occur if repositories are renamed, transferred, or deleted after the connection was established. Review the GitHub connection in Azure DevOps project settings and remove any disconnected or renamed repositories, then re-add them with their current names. For duplicate links on individual work items, manually remove the incorrect links from the work item's "Links" tab in Azure Boards.

---

### Q: Can I migrate our Azure Boards work items to GitHub Issues eventually?
**A:** Not with GitHub Enterprise Importer — GEI migrates repositories (code, pull requests, PR work-item links, branch policies), not Azure Boards work items. Many teams keep Boards connected long term. If you do move planning to GitHub Issues and Projects, use a script against both APIs or a third-party tool, then retire the Boards connection.

</details>

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
| Connect Azure Boards to GitHub | [Microsoft Learn](https://learn.microsoft.com/en-us/azure/devops/boards/github/connect-to-github?view=azure-devops) |
| Link GitHub commits, PRs, branches, and issues | [Microsoft Learn](https://learn.microsoft.com/en-us/azure/devops/boards/github/link-to-from-github?view=azure-devops) |
| What GEI migrates from Azure DevOps | [GitHub Docs](https://docs.github.com/en/migrations/ado/understand-migrations-from-azure-devops-to-github) |

---

*Last updated: October 2026*
