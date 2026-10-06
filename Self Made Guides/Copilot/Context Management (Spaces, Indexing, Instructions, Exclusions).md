# 🧠 GitHub Copilot Context Management Guide

> How to give GitHub Copilot the right context for higher-quality, grounded output across your enterprise

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [📋 Overview](#-overview)
- [1️⃣ Repository Custom Instructions](#1️⃣-repository-custom-instructions)
- [2️⃣ Organization Custom Instructions](#2️⃣-organization-custom-instructions)
- [3️⃣ Content Exclusion](#3️⃣-content-exclusion)
- [4️⃣ Repository Indexing](#4️⃣-repository-indexing)
- [5️⃣ Copilot Spaces](#5️⃣-copilot-spaces)
- [6️⃣ Putting It All Together](#6️⃣-putting-it-all-together)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📝 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- Repo instructions: commit `.github/copilot-instructions.md`
- Path-specific instructions: commit `.github/instructions/NAME.instructions.md` with an `applyTo` front matter block
- Org instructions: `Org Settings → Copilot → Custom instructions` → type instructions → **Save changes**
- Content exclusions: `Enterprise → AI controls → Copilot → Content exclusion` · `Org Settings → Copilot → Content exclusion` · `Repo Settings → Copilot → Content exclusion`
- Indexing: automatic — nothing to configure
- Spaces: `github.com/copilot/spaces` → **Create space** → name + owner → **Create Space** → **Add sources**

---

## ✅ Accuracy & Click-Path Notes

<details>
<summary><em>Show click-path conventions</em></summary>


- Reviewed against current public GitHub documentation in October 2026. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
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
| Copilot Business or Copilot Enterprise | GitHub **enterprise owner** or **organization owner** | ☐ |
| Repository custom instructions | **Write** access to the repository (to commit the files) | ☐ |
| Organization custom instructions and organization content exclusions | GitHub **organization owner** | ☐ |
| Repository content exclusions | **Repository administrator** | ☐ |
| Enterprise content exclusions | GitHub **enterprise owner** | ☐ |
| Copilot Spaces | Any user with a Copilot license. Organization-owned spaces need membership in that organization | ☐ |

---

## 📋 Overview

Copilot output quality depends on context. There are four context pillars:

| Pillar | Purpose | Scope | Who sets it up |
|--------|---------|-------|----------------|
| **Custom instructions** | Tell Copilot *how* to behave | Organization, repository, or path | Org owners, repo contributors |
| **Content exclusion** | Tell Copilot what *not* to read | Enterprise, organization, or repository | Owners and repo admins |
| **Repository indexing** | Let Copilot search the whole codebase by meaning | Per repository (automatic) | Nobody — it's automatic |
| **Copilot Spaces** | Bundle repos, files, issues, PRs, notes, and uploads into shared context | Personal or organization | Any Copilot user |

> 💡 **The biggest unlock is not a different model — it is giving Copilot the right standing instructions, exclusions, and task context.**

---

## 1️⃣ Repository Custom Instructions

**👤 Role:** **Write** access to the repository · **📍 Where:** the repository's files

### Repository-wide instructions

1. In the repository, create the file `.github/copilot-instructions.md`.
2. Add your guidance in plain Markdown, for example:

```markdown
## Project overview
This is a Node.js API using Express and TypeScript.

## Build & test
- Install: `npm ci`
- Test: `npm test`
- Lint: `npm run lint`

## Coding standards
- Use async/await, not callbacks
- Follow the error handling patterns in /src/middleware
- All new endpoints need integration tests
- Use Zod for input validation
```

3. Commit the file to the default branch.

> 💡 **Tip:** You can ask Copilot cloud agent to write this file for you: go to `github.com/copilot/agents`, pick the repository, and ask it to "onboard" the repository by creating `.github/copilot-instructions.md`.

### Path-specific instructions

1. Create the `.github/instructions` directory (subfolders are allowed).
2. Create a file whose name ends in `.instructions.md`, for example `api.instructions.md`.
3. Start the file with front matter that says which files it applies to:

```markdown
---
applyTo: "src/api/**/*.ts,src/api/**/*.tsx"
---
- Validate every request body with Zod
- Return errors in the shared ErrorResponse shape
```

4. Commit the file.

> 📌 On GitHub.com, path-specific instructions are used by **Copilot cloud agent** and **Copilot code review**. Add `excludeAgent: "code-review"` or `excludeAgent: "cloud-agent"` to the front matter to keep one of them from reading the file.

### Agent instructions

`AGENTS.md` files can live anywhere in the repository — the nearest one to the file being worked on wins. A single `CLAUDE.md` or `GEMINI.md` in the repository root also works.

---

## 2️⃣ Organization Custom Instructions

**👤 Role:** GitHub **organization owner** · **📍 Portal:** GitHub

**Navigate:** Organization → **Settings** → **Copilot** *(sidebar)* → **Custom instructions**

**Steps:**

1. In the left sidebar, click **Copilot**, then **Custom instructions**.
2. Under **Preferences and instructions**, type your instructions in the text box.
3. Click **Save changes**.
4. Test it: go to `github.com/copilot` and start a conversation.

Good uses:
- Preferred languages and style across all repositories
- Internal naming conventions
- Standard review expectations
- Company-wide architectural guidance
- Security requirements (for example, "never use eval()", "always sanitize user input")

> ⚠️ **Where it applies:** organization instructions are only used by Copilot Chat on GitHub.com, Copilot code review on GitHub.com, and Copilot cloud agent on GitHub.com — **not** by Copilot in IDEs. Put anything IDE users need in the repository instructions.

---

## 3️⃣ Content Exclusion

**👤 Role:** GitHub **enterprise owner** · **organization owner** · **repository administrator** · **📍 Portal:** GitHub

| Scope | Click path |
|-------|-----------|
| Enterprise | Enterprise → **AI controls** → **Copilot** *(sidebar)* → **Content exclusion** |
| Organization | Organization → **Settings** → **Copilot** → **Content exclusion** |
| Repository | Repository → **Settings** → **Copilot** → **Content exclusion** |

**Steps (any scope):**

1. Open the **Content exclusion** page for the scope.
2. In the text box, enter the paths to exclude — one YAML entry per line (examples below).
3. Save your changes.

**Example (organization scope):**

```yaml
"*":
  - "**/.env"
  - "**/secrets/**"
payments-service:
  - "/config/credentials.yml"
  - "**/generated/**"
```

| Pattern | Effect |
|---------|--------|
| `"*": ["**/.env"]` | Every `.env` file in every repository |
| `"*": ["**/secrets/**"]` | Everything under any `secrets` folder |
| `payments-service: ["**/generated/**"]` | Generated code in one repository |
| `payments-service: ["/config/credentials.yml"]` | One specific file |

> 📌 At the **enterprise** level, identify a repository by its full clone URL (for example `https://github.com/octo-org/payments-service.git:`). At the **repository** level, list only paths.

> ⏱️ **Timing:** changes can take up to **30 minutes** to reach IDEs that already loaded the settings. To apply them sooner, restart JetBrains IDEs or Visual Studio, or run **Developer: Reload Window** in VS Code.

> ⚠️ **Limits:** content exclusion isn't supported everywhere (for example, Edit and Agent modes in IDEs). See `Copilot/Admin Controls (Models, Content Exclusion, Instructions).md` for details.

---

## 4️⃣ Repository Indexing

Repository indexing builds a **semantic code search** index so Copilot can find code by meaning, not just exact text.

### How it works

- **Automatic.** A repository is indexed the first time someone chats with Copilot in that repository's context. There's nothing to turn on and no settings page.
- **Fast.** First-time indexing takes up to about **60 seconds** for a large repository. After that, the index usually updates within seconds of a new conversation.
- **Unlimited.** There's no limit on how many repositories can be indexed.
- **Used by:** Copilot Chat on GitHub and in VS Code, and Copilot cloud agent.
- **Not used for training.** Copilot doesn't use your indexed repository to train models.
- **Respects exclusions.** Excluded content is filtered out before it reaches Copilot Chat.

> 📌 **Non-GitHub repositories:** VS Code can also index workspaces from outside GitHub (GitLab, local repos) — this uploads the data to GitHub. It's off by default; an owner must set the **Semantic indexing for non-GitHub repositories** policy to **Enabled**. GitHub.com only, not GHE.com.

---

## 5️⃣ Copilot Spaces

Spaces bundle repositories, files, pull requests, issues, notes, images, and uploads into one context that Copilot uses to answer questions. GitHub-based sources stay in sync as they change.

### When to use Spaces

| Scenario | Use Spaces? |
|----------|------------|
| Working across multiple related repos | ✅ Yes |
| Onboarding new team members | ✅ Yes — bundle key repos, docs, and checklists |
| Feature work with specs | ✅ Yes — include the spec, repo, and design notes |
| Style guides or review checklists | ✅ Yes |
| Quick question about one file | ❌ No — use `#file` in IDE chat |
| Single-repo question in the IDE | ❌ No — use `@workspace` or `#codebase` |

### Create a space

**👤 Role:** Any user with a Copilot license · **📍 Portal:** GitHub

1. Go to `https://github.com/copilot/spaces` and click **Create space**.
2. Enter a name.
3. Choose the owner — **you** or an **organization** you belong to. *(Pick the organization for team spaces.)*
4. Click **Create Space**.
5. *(Optional)* Under the name, add a description.
6. Add **Instructions** — free text describing what Copilot should focus on in this space.
7. Click **Add sources** and choose one or more:
   - **Add files and repositories** — files, folders, or whole repositories
   - **Link files, pull requests, and issues** — paste GitHub URLs
   - **Upload a file** — images, text files, documents, spreadsheets
   - **Add text content** — notes, transcripts, and so on

> 💡 **Repository vs file:** an attached **repository** is searched for relevant pieces per question. An attached **file** is loaded in full for every question — use files for the few documents Copilot must always consider. Spaces read the `main` branch.

### Share a space

1. In the top-right corner of the space, click the **share** icon.
2. Search for users or teams and choose a role for each.
3. **Organization-owned spaces:** next to the organization name, choose a base role for everyone else — **Viewer**, **Editor**, **Admin**, or **No access**.
4. **Personal spaces:** to share publicly, under **General access** select **Anyone with link**.
5. *(Optional)* Click **Copy link**.

| Role | Can do |
|------|--------|
| **Viewer** | Ask questions; see attachments and instructions |
| **Editor** | Viewer + edit sources, name, description, and instructions |
| **Admin** | Editor + change sharing and delete the space |

> 📌 Viewers only see sources they already have access to. Questions in a space use **AI credits** from your enterprise's pool, like any Copilot Chat request. Team members find shared spaces on the **Organizations** tab at `github.com/copilot/spaces`, and can use spaces in the IDE through the GitHub MCP server.

---

## 6️⃣ Putting It All Together

### Recommended setup order

1. **Enterprise:** set content exclusions for sensitive paths.
2. **Organization:** add custom instructions for shared standards (used on GitHub.com).
3. **Repository:** commit `.github/copilot-instructions.md` and any path-specific files.
4. **Team:** create organization-owned Copilot Spaces for active projects.
5. **Developer:** use `#file`, `#codebase`, `@workspace`, and spaces when chatting.

### Context hierarchy

```
Enterprise   (content exclusion — what Copilot CANNOT see)
  → Organization (custom instructions — shared standards)
    → Repository (instructions + automatic indexing — project truth)
      → Space (bundled context — task-specific)
        → Chat (ad-hoc references — #file, #codebase, @workspace)
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
| **Copilot stops working for a user mid-cycle** | The user's user-level budget is used up, the shared AI credit pool is exhausted with **AI credits paid usage** disabled, or a spending limit with **Stop usage** was reached. | Check the user on **Billing and licensing** → **AI usage** and the budgets on **Budgets and alerts**; raise their budget, approve their budget request (not available for EMU enterprises), or enable AI credits paid usage. |
| **Content exclusions do not apply immediately** | Client policy cache, unsupported surface/mode, symlink/remote filesystem limitation, or indirect IDE context. | Reload the IDE policy, verify the exclusion syntax at enterprise/org/repo scope, and document surfaces where exclusions are limited. |
| **Usage metrics look empty or inconsistent** | Telemetry is disabled, data freshness delay applies, users are unlicensed, or different APIs report different scopes. | Enable the metrics policy, confirm seats and telemetry, wait for data freshness, and avoid comparing dashboards/API endpoints as if they share identical data models. |
| **Cloud agent or MCP action is denied** | Agent policy, MCP policy, repository permissions, secrets, or server allowlist does not permit the operation. | Review Enterprise AI controls > Agents/MCP, repo-level permissions, MCP server configuration, and audit logs for the denied action. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>


### Q: How long does repository indexing take?
**A:** First-time indexing takes up to about 60 seconds, even for a large repository. After that, the index usually updates within seconds of starting a new conversation. It's automatic — there's no indexing settings page to check.

---

### Q: @workspace or #codebase isn't returning relevant results — what should I check?
**A:** Check that the files you expect aren't covered by a content exclusion, and that you're asking in the right repository or workspace. Ask a more specific question, or name the files with `#file`. For questions that span several repositories, use a space.

---

### Q: When should I use Spaces vs @workspace?
**A:** Use `@workspace` or `#codebase` for questions about the one repository you have open. Use a space when the context spans several repositories or includes non-code material — specs, issues, notes, or uploaded documents — or when you want to share that context with a team.

---

### Q: Content exclusion is not working for a specific file — what's wrong?
**A:** Changes can take up to 30 minutes to reach IDEs that already loaded the settings — restart the IDE (or **Developer: Reload Window** in VS Code) to apply them sooner. Then check the pattern syntax and repository identifier for the scope, and confirm the surface supports exclusions (Edit and Agent modes in IDEs don't).

---

### Q: Can we share Copilot Spaces across the team?
**A:** Yes. Create the space under your **organization**, then click the share icon to add users or teams with **Viewer**, **Editor**, or **Admin** roles, or set a base role for all organization members.

---

### Q: My custom instructions in `.github/copilot-instructions.md` don't seem to be taking effect — why?
**A:** Check that the file is exactly `.github/copilot-instructions.md` and is committed to the branch Copilot is using. In Copilot Chat, open the response's references to see whether the file was used. For Copilot code review, the user's personal **Use custom instructions when reviewing pull requests** option must be on (it is by default). Finally, make the instructions short, specific, and non-conflicting.

---

### Q: Can I use Spaces to give Copilot context about internal documentation or design specs?
**A:** Yes. Add the documents as files from a repository, upload them, or paste them as text content. Attach small, critical documents as **files** so Copilot considers them for every question.

</details>

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Admin Controls (Models, Content Exclusion, Instructions) | `Copilot/Admin Controls (Models, Content Exclusion, Instructions).md` |
| Cloud Agent & MCP Configuration | `Copilot/Cloud Agent & MCP Configuration.md` |
| Responsible AI Guardrails | `Copilot/Responsible AI Guardrails.md` |

---

## 📝 Resources

| Resource | Link |
|----------|------|
| Repository custom instructions | [GitHub Docs](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/add-custom-instructions/add-repository-instructions) |
| Organization custom instructions | [GitHub Docs](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/add-custom-instructions/add-organization-instructions) |
| Content exclusion | [GitHub Docs](https://docs.github.com/en/copilot/how-tos/configure-content-exclusion/exclude-content-from-copilot) |
| Repository indexing | [GitHub Docs](https://docs.github.com/en/copilot/concepts/context/repository-indexing) |
| About Copilot Spaces | [GitHub Docs](https://docs.github.com/en/copilot/concepts/context/spaces) |
| Create Copilot Spaces | [GitHub Docs](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/copilot-spaces/create-copilot-spaces) |
| Share Copilot Spaces | [GitHub Docs](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/copilot-spaces/collaborate-with-others) |

---

*Last updated: October 2026*
