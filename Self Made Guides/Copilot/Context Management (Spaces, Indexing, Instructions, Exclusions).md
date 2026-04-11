# 🧠 Copilot Context Management Guide

> How to give GitHub Copilot the right context for higher-quality, grounded output across your enterprise

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
