# 🤖 GitHub Copilot Admin Controls Runbook (Model Restriction, Content Exclusion, Custom Instructions)

> **Complete guide to managing Copilot AI models, content exclusions, custom instructions, and policy controls across enterprise and organization levels**

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- Restrict models: `Enterprise → AI controls → Copilot → Models` — toggle models on/off
- Content exclusions: `Enterprise → AI controls → Copilot → Content exclusion` — add glob patterns or repos
- Org custom instructions: `Org Settings → Copilot → Custom instructions` — enter up to 6,000 characters
- Repo instructions: create `.github/copilot-instructions.md` in repo root
- Public code filter: `Enterprise → AI controls → Copilot → Policies → Suggestions matching public code → Block`

---

## ✅ Prerequisites

| Requirement | Status |
|-------------|--------|
| GitHub Enterprise Cloud account with enterprise owner access | ☐ |
| Copilot Business or Copilot Enterprise subscription active | ☐ |
| Organization admin access (for org-level controls) | ☐ |
| Repository admin or write access (for repo-level instructions) | ☐ |

---

## 📋 Overview

This runbook covers the key admin controls for GitHub Copilot that let you govern what models are available, what code Copilot can reference, and what instructions guide its behavior:

| Control | Scope | Purpose |
|---------|-------|---------|
| **Model restriction** | Enterprise / Org | Control which AI models users can access |
| **Content exclusion** | Enterprise / Org | Prevent Copilot from referencing sensitive files or repos |
| **Custom instructions** | Org / Repo | Guide Copilot's behavior with coding standards and context |
| **Public code filter** | Enterprise / Org | Block suggestions matching public code |
| **Extensions policy** | Enterprise | Control which Copilot Extensions are allowed |

---

## 🏗️ Policy Hierarchy

Understanding where each control lives and how they cascade:

| Level | Role | Example |
|-------|------|---------|
| **Enterprise** | Guardrails | Disable models org admins cannot re-enable, enforce content exclusions |
| **Organization** | Standards | Set coding standards, restrict models further, add org-level exclusions |
| **Repository** | Project truth | Repo-specific instructions that reflect the actual codebase |
| **Content exclusion** | Sensitive paths | Keep secrets, configs, and proprietary logic out of Copilot context |

> 💡 **Tip:** Enterprise settings act as a ceiling. Organizations can restrict further but cannot enable something the enterprise has disabled.

---

## 1️⃣ Restrict AI Models at the Enterprise Level

*Control which premium and base models are available across all organizations*

**Navigation:**

```
Enterprise → AI controls → Copilot → Models
```

**Steps:**

1. Review the list of available models (base and premium)
2. Toggle models to **Enabled** or **Disabled**
3. Disabled models will not appear as options for users in any organization under this enterprise

> ⚠️ **Important:** Disabling a model at the enterprise level cannot be overridden by organization admins. Users will not see disabled models in their model picker.

---

## 2️⃣ Restrict AI Models at the Organization Level

*Further restrict models within a specific organization*

**Navigation:**

```
Profile Picture → Organizations → [Your Organization] → Settings
  → Copilot → Models
```

**Steps:**

1. Review the models that the enterprise has left enabled
2. Toggle individual models to **Enabled** or **Disabled** for this organization
3. Save changes

> 💡 **Tip:** Org admins can only disable models the enterprise has enabled. They cannot re-enable models the enterprise has disabled.

---

## 3️⃣ Set Up Content Exclusions at the Enterprise Level

*Prevent Copilot from using content from specific files or repositories across all organizations*

**Navigation:**

```
Enterprise → AI controls → Copilot → Content exclusion
```

**Steps:**

1. Click **Add exclusion**
2. Choose the exclusion type:

| Exclusion Type | Format | Example |
|----------------|--------|---------|
| **File path patterns** | Glob patterns | `**/.env`, `**/secrets/**`, `config/credentials.*` |
| **Entire repositories** | `org/repo` | `my-org/internal-secrets` |

3. Enter the repository or path pattern
4. Save the exclusion

> ⚠️ **Important:** Content exclusions apply to Copilot code completions and Copilot Chat. Excluded content will not be used as context for suggestions.

---

## 4️⃣ Set Up Content Exclusions at the Organization Level

*Add organization-specific exclusions on top of enterprise exclusions*

**Navigation:**

```
Profile Picture → Organizations → [Your Organization] → Settings
  → Copilot → Content exclusions
```

**Steps:**

1. Click **Add exclusion**
2. Specify the repository (within the organization)
3. Enter file path patterns to exclude:

| Pattern | What It Excludes |
|---------|------------------|
| `**/.env` | All `.env` files in any directory |
| `**/secrets/**` | Everything under any `secrets` directory |
| `config/production.*` | Production config files |
| `**/*.pem` | All PEM certificate files |

4. Save the exclusion

> 💡 **Tip:** Organization-level exclusions are additive to enterprise-level exclusions. You do not need to duplicate enterprise exclusions at the org level.

---

## 5️⃣ Create Organization-Level Custom Instructions

*Set coding standards and context that apply to all Copilot interactions within the organization*

**Navigation:**

```
Profile Picture → Organizations → [Your Organization] → Settings
  → Copilot → Custom instructions
```

**Steps:**

1. Enable **Custom instructions**
2. Enter instructions in the text field (up to 6,000 characters)
3. Save changes

**Example instructions:**

- "Always use TypeScript strict mode"
- "Follow the Google Java Style Guide"
- "Include JSDoc comments on all public functions"
- "Use snake_case for Python variables and function names"

> 💡 **Tip:** Organization instructions are automatically included as context in Copilot Chat conversations for all members of the organization.

---

## 6️⃣ Create Repository-Level Custom Instructions

*Provide project-specific context that lives alongside the code*

### A) General Repository Instructions

Create a file at the root of the repository:

```
.github/copilot-instructions.md
```

This file contains instructions that apply to all Copilot interactions within the repository.

### B) Task-Specific Instruction Files

Create instruction files in the following directory:

```
.github/instructions/*.instructions.md
```

Each file can target specific tasks or file types using front matter:

```markdown
---
applyWhen: "**/*.test.ts"
---

When writing tests, use Vitest as the test framework.
Always include edge case tests.
Use descriptive test names following the pattern: "should [expected behavior] when [condition]".
```

> 💡 **Tip:** Repository-level instructions are version-controlled and travel with the code. This makes them the best place for project-specific standards that all contributors (and Copilot) should follow.

### Instruction Precedence

| Priority | Source | Scope |
|----------|--------|-------|
| 1 (highest) | `.github/instructions/*.instructions.md` | Task/file-specific |
| 2 | `.github/copilot-instructions.md` | Repository-wide |
| 3 | Organization custom instructions | All repos in the org |

---

## 7️⃣ Enable or Disable the Public Code Filter

*Control whether Copilot suggests code that matches publicly available code*

**Navigation:**

```
Enterprise → AI controls → Copilot → Policies
  → Suggestions matching public code → Block / Allow
```

Or at the organization level:

```
Profile Picture → Organizations → [Your Organization] → Settings
  → Copilot → Policies → Suggestions matching public code
```

**Options:**

| Setting | Effect |
|---------|--------|
| **Block** | Copilot filters out suggestions that match public code (reduces IP risk) |
| **Allow** | Copilot does not filter suggestions based on public code matches |

> 💡 **Tip:** Enabling the public code filter is recommended for enterprises concerned about intellectual property and licensing compliance.

---

## 8️⃣ Manage Copilot Extensions Policy at the Enterprise Level

*Control which Copilot Extensions can be used across the enterprise*

**Navigation:**

```
Enterprise → AI controls → Copilot → Policies
  → Copilot Extensions
```

**Options:**

| Setting | Effect |
|---------|--------|
| **No policy** | Organization admins decide their own Extensions policy |
| **Enabled** | Extensions are available to all organizations |
| **Disabled** | Extensions are blocked across all organizations |

> ⚠️ **Important:** When set to **Disabled** at the enterprise level, organization admins cannot enable Extensions for their org.

---

## 📝 Additional Notes

> 💡 **Customization:** The exact wording and layout of settings may vary slightly depending on your GitHub Enterprise Cloud version and whether you are using Copilot Business or Copilot Enterprise. The navigation paths above reflect the current UI at time of writing.

---

## ❓ Common Questions & Troubleshooting

### Q: Can we restrict models at the org level if the enterprise allows them?
**A:** Yes, an organization can be more restrictive than the enterprise but not less restrictive. If the enterprise enables a model, the org admin can disable it for their org. However, if the enterprise disables a model, the org admin cannot re-enable it.

---

### Q: My content exclusion patterns are not working — what's wrong?
**A:** Check three things: (1) verify your glob syntax is correct (e.g., `**/.env` not `*.env`), (2) ensure patterns are relative to the repository root, and (3) wait up to 30 minutes for exclusions to propagate. Exclusions do not take effect instantly after saving.

---

### Q: Can we see which files are currently excluded?
**A:** There is no dashboard or report that lists excluded files. The best way to test is to ask Copilot Chat about content in an excluded file — if the exclusion is working, Copilot will not reference that file's content.

---

### Q: Custom instructions are not being followed — what should I check?
**A:** Verify the file path is exactly `.github/copilot-instructions.md` at the repository root. Also confirm that the custom instructions feature is enabled at the organization level under Settings > Copilot > Custom instructions. If using task-specific instructions, check that the `applyWhen` front matter pattern matches the files you are working with.

---

### Q: Can we enforce custom instructions across all repos in the org?
**A:** Use organization-level custom instructions for shared standards that apply everywhere. Repository-level instructions in `.github/copilot-instructions.md` then add project-specific context on top. Org instructions are automatically included in all Copilot Chat conversations for org members.

---

### Q: We disabled a model at the enterprise level but users still see it — why?
**A:** Model restriction changes can take a few minutes to propagate. Ask users to restart their IDE or refresh their Copilot Chat session. If the model still appears, verify the change was saved correctly in Enterprise > AI controls > Copilot > Models.

---

### Q: Can content exclusions block Copilot code completions and Chat separately?
**A:** No, content exclusions apply to both Copilot code completions and Copilot Chat simultaneously. You cannot exclude a file from Chat but allow it for completions or vice versa.

---

### Q: What happens if org-level and enterprise-level exclusions overlap?
**A:** Organization-level exclusions are additive to enterprise-level exclusions. You do not need to duplicate enterprise exclusions at the org level. The most restrictive combination of all applicable exclusions applies.

---

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Seat Assignment & Enablement | `Copilot/Seat Assignment & Enablement.md` |
| BYOK (Bring Your Own Key) Configuration | `Copilot/BYOK (Bring Your Own Key) Configuration.md` |
| Context Management (Spaces, Indexing, Instructions, Exclusions) | `Copilot/Context Management (Spaces, Indexing, Instructions, Exclusions).md` |
| Responsible AI Guardrails | `Copilot/Responsible AI Guardrails.md` |
| Coding Agent & MCP Configuration | `Copilot/Coding Agent & MCP Configuration.md` |

---

## 📚 Resources

- [Configure access to AI models](https://docs.github.com/en/copilot/how-tos/use-ai-models/configure-access-to-ai-models)
- [Exclude content from Copilot](https://docs.github.com/en/copilot/how-tos/configure-content-exclusion/exclude-content-from-copilot)
- [Add organization custom instructions](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-organization-instructions)
- [Add custom instructions for GitHub Copilot](https://docs.github.com/en/copilot/customizing-copilot/adding-custom-instructions-for-github-copilot)

---

*Last updated: April 2026*
