# 🧠 GitHub Copilot Context Management Guide

> How to give GitHub Copilot the right context for higher-quality, grounded output across your enterprise

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [📋 Overview](#-overview)
- [1️⃣ Repository Custom Instructions](#1-repository-custom-instructions)
- [2️⃣ Organization Custom Instructions](#2-organization-custom-instructions)
- [3️⃣ Content Exclusion](#3-content-exclusion)
- [4️⃣ Repository Indexing](#4-repository-indexing)
- [5️⃣ Copilot Spaces](#5-copilot-spaces)
- [6️⃣ Putting It All Together](#6-putting-it-all-together)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- Repo instructions: create `.github/copilot-instructions.md` in repo root
- Path-specific instructions: create `.github/instructions/*.instructions.md`
- Org instructions: `Org Settings → Copilot → Custom instructions → Add instructions`
- Content exclusions: `Enterprise → AI controls → Copilot → Content exclusion` or `Org Settings → Copilot → Content exclusions`
- Check indexing: `Repository → Settings → Copilot → Indexing`
- Create Spaces: `GitHub.com → Copilot → Spaces → New Space`

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
| Copilot Business or Copilot Enterprise subscription active | ☐ |
| Organization admin access (for org-level instructions and exclusions) | ☐ |
| Repository write access (for repo-level instructions) | ☐ |
| Enterprise owner access (for enterprise-level content exclusions) | ☐ |

---

## 📋 Overview

Copilot output quality depends on context. There are four context pillars:

| Pillar | Purpose | Scope |
|--------|---------|-------|
| **Custom Instructions** | Tell Copilot HOW to behave | Org-wide or repo-specific |
| **Content Exclusion** | Tell Copilot what NOT to read | Enterprise, org, or repo |
| **Repository Indexing** | Let Copilot understand the full codebase | Per-repository |
| **Copilot Spaces** | Bundle repos, files, docs into shared context | Cross-repo, team-level |

> 💡 **The biggest unlock is not a different model — it is giving Copilot the right standing instructions, exclusions, and task context.**

---

## 1️⃣ Repository Custom Instructions

Create `.github/copilot-instructions.md` in any repository:

```markdown
## Project Overview
This is a Node.js API using Express and TypeScript.

## Build & Test
- Install: `npm install`
- Test: `npm test`
- Lint: `npm run lint`

## Coding Standards
- Use async/await, not callbacks
- Follow existing error handling patterns in /src/middleware
- All new endpoints require integration tests
- Use Zod for input validation
```

### Path-Specific Instructions

Create `.github/instructions/*.instructions.md` for folder-specific rules:

```
.github/instructions/api.instructions.md     → rules for /src/api/
.github/instructions/tests.instructions.md   → rules for /tests/
```

---

## 2️⃣ Organization Custom Instructions

```
Profile picture → Organizations → your org → Settings
  → Copilot → Custom instructions
    → Add instructions
```

Use for:
- Preferred language/style across all repos
- Internal naming conventions
- Standard review expectations
- Company-wide architectural guidance
- Security requirements (e.g., "never use eval()", "always sanitize user input")

---

## 3️⃣ Content Exclusion

Prevent Copilot from reading sensitive files:

### Enterprise Level

```
Enterprise → AI controls → Copilot → Content exclusion
  → Add exclusion rules
```

### Organization Level

```
Organization → Settings → Copilot → Content exclusions
  → Add file path patterns or entire repositories
```

### Example Exclusion Patterns

| Pattern | Effect |
|---------|--------|
| `**/.env` | Exclude all .env files |
| `**/secrets/**` | Exclude all secrets directories |
| `**/generated/**` | Exclude generated code |
| `config/credentials.yml` | Exclude specific file |
| Entire repository | Exclude all files in a repo |

> ⚠️ **Content exclusion inherits:** Enterprise → Organization → Repository. More restrictive rules win.

---

## 4️⃣ Repository Indexing

Repository indexing lets Copilot understand your full codebase for better answers in Copilot Chat.

### How It Works
- GitHub automatically indexes repositories for Copilot
- Indexed repos provide better `@workspace` answers and more relevant code suggestions
- Indexing happens in the background after repo creation or significant changes

### Check Indexing Status

```
Repository → Settings → Copilot → Indexing
  → View indexing status
```

### What Indexing Does NOT Mean
- It does NOT send your entire codebase to the model with every prompt
- It creates an index that helps Copilot find relevant code when you ask
- Content exclusions still apply — excluded files are not indexed

---

## 5️⃣ Copilot Spaces

Spaces bundle multiple repos, files, docs, PRs, issues, and notes into a shared context set.

### When to Use Spaces

| Scenario | Use Spaces? |
|----------|------------|
| Working across multiple related repos | Yes |
| Onboarding new team members | Yes — bundle key repos + docs |
| Feature development with specs | Yes — include spec, repo, design docs |
| Quick question about one file | No — use `#file` in chat |
| Single-repo work | No — `@workspace` is sufficient |

### Creating a Space

```
GitHub.com → Copilot → Spaces → New Space
  → Add repositories, files, documents
    → Add notes or context
      → Share with team
```

### Best Practices for Spaces
- Include the **spec/issue** alongside the **code repos**
- Add **API documentation** or **design notes** as context
- Keep Spaces focused — one per feature or project, not one giant "everything" Space
- Update Spaces as the project evolves

---

## 6️⃣ Putting It All Together

### Recommended Setup Order

1. **Enterprise:** Set content exclusions for sensitive paths
2. **Organization:** Add custom instructions for shared standards
3. **Repository:** Add `.github/copilot-instructions.md` for project-specific context
4. **Team:** Create Copilot Spaces for active projects
5. **Developer:** Use `#file`, `@workspace`, and Spaces references in chat

### Context Hierarchy

```
Enterprise (content exclusion — what Copilot CANNOT see)
  → Organization (custom instructions — shared standards)
    → Repository (instructions + indexing — project truth)
      → Space (bundled context — task-specific)
        → Chat (ad-hoc references — #file, @workspace)
```

## 🧯 Known Errors & Resolutions

<details>
<summary><em>Show known errors table</em></summary>


> This section lists the known product errors and admin-facing symptoms that commonly occur with this workflow. Exact message text can vary by product rollout, tenant policy, and provider, so use the log or settings page named in the resolution to confirm the root cause.

| Error or symptom | Likely cause | Resolution |
|------------------|--------------|------------|
| **Page, tab, or button is missing** | Wrong account context, missing admin role, unavailable plan/add-on, or feature rollout not enabled for the selected enterprise/org/repo. | Switch to the correct account and scope, confirm the prerequisite role, verify licensing or add-on activation, then refresh the page. If the control is still absent, use the direct settings URL from the relevant GitHub Docs page and confirm the feature is available for your plan. |
| **Changes appear saved but behavior does not change** | Policy inheritance, cached UI state, propagation delay, or an overlapping enterprise/org/repo policy. | Reopen the settings page, verify the effective policy at the lowest affected scope, wait for propagation where documented, and check for a stricter policy at an enterprise or organization level. |
| **403, forbidden, or resource not accessible** | The signed-in user or token can see the page but lacks the specific permission for the action. | Use an enterprise owner, organization owner, repository admin, or token with the exact scopes/permissions listed in the runbook. For SAML-protected orgs, authorize the token or SSH key for SSO before retrying. |
| **Copilot feature, model, or policy is not visible** | Plan, license assignment, enterprise policy, org delegation, or feature rollout does not permit it. | Check enterprise AI controls, organization Copilot settings, assigned seat status, and the plan requirements for the feature. |
| **Premium requests are rejected after the included allowance** | Paid usage is disabled, no billing entity is selected, or a stop-usage budget is exhausted. | Enable Premium request paid usage where appropriate, set or delete conflicting budgets, and have users with multiple licenses choose a billing entity. |
| **Content exclusions do not apply immediately** | Client policy cache, unsupported surface/mode, symlink/remote filesystem limitation, or indirect IDE context. | Reload the IDE policy, verify the exclusion syntax at enterprise/org/repo scope, and document surfaces where exclusions are limited. |
| **Usage metrics look empty or inconsistent** | Telemetry is disabled, data freshness delay applies, users are unlicensed, or different APIs report different scopes. | Enable the metrics policy, confirm seats and telemetry, wait for data freshness, and avoid comparing dashboards/API endpoints as if they share identical data models. |
| **Coding agent or MCP action is denied** | Agent policy, MCP policy, repository permissions, secrets, or server allowlist does not permit the operation. | Review Enterprise AI controls > Agents/MCP, repo-level permissions, MCP server configuration, and audit logs for the denied action. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>


### Q: How long does repository indexing take?
**A:** It depends on repository size. Small repos may index in minutes, while large repos with extensive history can take hours. Indexing happens in the background after repo creation or significant changes. Check the status at Repository > Settings > Copilot > Indexing.

---

### Q: @workspace is not returning relevant results — what should I check?
**A:** Ensure indexing is complete for the repository (check status in repo settings). Verify that the relevant files are not excluded via content exclusion rules. If the repo was recently created or had major changes, wait for re-indexing to finish before expecting accurate results.

---

### Q: When should I use Spaces vs @workspace?
**A:** Use `@workspace` for single-repo local context when you are working within one repository. Use Spaces when you need cross-repo context — for example, working across multiple related repos, onboarding with bundled docs and code, or feature development that spans several projects.

---

### Q: Content exclusion is not working for a specific file — what's wrong?
**A:** Exclusions can take up to 30 minutes to propagate after saving. Verify your glob pattern syntax is correct and that patterns are relative to the repository root. Test by asking Copilot about content in the excluded file — if exclusion is working, Copilot will not reference it.

---

### Q: Can we share Copilot Spaces across the team?
**A:** Yes, Spaces support sharing with team members. When creating or editing a Space, you can share it with specific users or teams. This is useful for onboarding, cross-functional projects, and shared context sets.

---

### Q: My custom instructions in `.github/copilot-instructions.md` don't seem to be taking effect — why?
**A:** Verify the file is at exactly `.github/copilot-instructions.md` in the repository root (not in a subdirectory). Confirm the custom instructions feature is enabled at the organization level. Also check that your instructions are clear and specific — vague instructions may not produce noticeable changes in Copilot behavior.

---

### Q: Can I use Spaces to give Copilot context about internal documentation or design specs?
**A:** Yes. Spaces can bundle repositories, files, documents, PRs, issues, and free-form notes. Adding API documentation, design specs, or architecture docs to a Space gives Copilot richer context for more grounded answers.

</details>

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Admin Controls (Models, Content Exclusion, Instructions) | `Copilot/Admin Controls (Models, Content Exclusion, Instructions).md` |
| Coding Agent & MCP Configuration | `Copilot/Coding Agent & MCP Configuration.md` |
| Responsible AI Guardrails | `Copilot/Responsible AI Guardrails.md` |

---

## 📝 Resources

| Resource | Link |
|----------|------|
| Repository custom instructions | [GitHub Docs](https://docs.github.com/en/copilot/customizing-copilot/adding-custom-instructions-for-github-copilot) |
| Organization custom instructions | [GitHub Docs](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-organization-instructions) |
| Content exclusion | [GitHub Docs](https://docs.github.com/en/copilot/how-tos/configure-content-exclusion/exclude-content-from-copilot) |
| Repository indexing | [GitHub Docs](https://docs.github.com/en/copilot/concepts/context/repository-indexing) |
| Copilot Spaces | [GitHub Docs](https://docs.github.com/en/copilot/concepts/context/spaces) |
| Using Copilot Spaces | [GitHub Docs](https://docs.github.com/en/copilot/how-tos/provide-context/use-copilot-spaces) |

---

*Last updated: April 2026*
