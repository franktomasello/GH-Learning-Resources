# 🛡️ Responsible AI Guardrails for GitHub Copilot Runbook

> Platform controls and organizational policies to govern responsible AI use with GitHub Copilot

---

## 📋 Overview

Responsible AI governance requires BOTH platform controls (what GitHub provides) AND organizational controls (what the enterprise enforces through policy and process).

| Layer | Who Manages | Examples |
|-------|------------|---------|
| **Platform controls** | Enterprise/org admins | Content exclusion, model restriction, public code filter |
| **Code governance** | Repo admins/teams | Branch protection, required reviews, status checks |
| **Organizational policy** | Leadership/security | Usage guidelines, training, acceptable use |
| **Developer practices** | Individual developers | Prompt hygiene, code review diligence |

---

## 1️⃣ Platform Controls (Admin-Configurable)

### Content Exclusions

Prevent Copilot from reading sensitive files:

```
Enterprise → AI controls → Copilot → Content exclusion
  → Add file path patterns or entire repositories
```

Or at org level:

```
Organization → Settings → Copilot → Content exclusions
```

Example patterns:
- `**/.env` — exclude all environment files
- `**/secrets/**` — exclude secrets directories
- `**/generated/**` — exclude generated code
- Entire repositories containing proprietary algorithms

### Model Restrictions

```
Enterprise → AI controls → Copilot → Models
  → Enable only approved models
  → Disable premium/reasoning models if not needed
```

### Public Code Filter

```
Organization → Settings → Copilot → Policies
  → Suggestions matching public code → Block
```

> ✅ **Result:** Copilot will not suggest code that closely matches publicly available code, reducing IP risk.

### Copilot Extensions Policy

```
Enterprise → AI controls → Copilot → Policies
  → Copilot Extensions → Control which third-party extensions are allowed
```

### No Training Commitment

For Business and Enterprise plans: GitHub does NOT use your code or prompts to train AI models. Prompts are NOT retained after serving responses.

---

## 2️⃣ Code Governance Controls

### Required PR Reviews

All AI-generated code goes through the same review process as human-written code:

```
Organization → Settings → Rules → Rulesets → New ruleset
  → Require a pull request before merging
    → Required approvals: 1 (or more)
```

> 💡 **Tip:** Copilot coding agent ALWAYS opens PRs — it never commits directly. This is by design.

### Required Status Checks

Run security scanning on every PR (AI-generated or not):

```
Organization → Settings → Rules → Rulesets
  → Require status checks → Add: "CodeQL", "tests", "lint"
```

### CODEOWNERS for Critical Paths

Create `CODEOWNERS` file to require specific team approvals on sensitive paths:

```
# .github/CODEOWNERS
/src/auth/          @security-team
/config/            @platform-team
/.github/workflows/ @devops-team
```

---

## 3️⃣ Custom Instructions for Standards

### Organization-Level

```
Organization → Settings → Copilot → Custom instructions
```

Example instructions:
- "Never use `eval()` or `exec()` in any language"
- "Always sanitize user input before database queries"
- "Follow OWASP Top 10 secure coding practices"
- "Use parameterized queries, never string concatenation for SQL"

### Repository-Level

Create `.github/copilot-instructions.md` with project-specific rules.

---

## 4️⃣ Mitigating "Vibe Coding"

"Vibe coding" = letting AI generate code rapidly with minimal human review. Risks: technical debt, security vulnerabilities, reduced understanding.

### Governance Controls

| Control | Navigation | Effect |
|---------|-----------|--------|
| Required PR reviews | Org → Settings → Rules → Rulesets | AI code must be reviewed |
| GHAS code scanning | Org → Settings → Code security → Enable | Catches vulnerabilities regardless of author |
| Secret scanning + push protection | Org → Settings → Code security → Secret scanning | Blocks leaked credentials |
| Branch protection | Org → Settings → Rules → Rulesets | Prevents direct pushes to main |
| Content exclusions | Org → Settings → Copilot → Content exclusions | Keeps sensitive files out of AI context |

### Additional Best Practices

- Monitor Copilot usage metrics for teams with unusually high acceptance rates — may indicate insufficient review
- Train developers: Copilot is an accelerator, not a replacement for understanding
- Run periodic code quality reviews on AI-heavy PRs

---

## 5️⃣ Monitoring & Audit

### Usage Metrics

```
Enterprise → AI controls → Insights
  → Active users, acceptance rates, model usage
```

### Audit Log

```
Enterprise → Settings → Audit log
  → Filter for Copilot-related events
```

### Security Scanning Results

```
Organization → Security → Overview
  → Code scanning, secret scanning, Dependabot alerts
```

---

## 📝 Resources

| Resource | Link |
|----------|------|
| Copilot Trust Center | [GitHub Resources](https://resources.github.com/copilot-trust-center/) |
| Content exclusions | [GitHub Docs](https://docs.github.com/en/copilot/how-tos/configure-content-exclusion/exclude-content-from-copilot) |
| Custom instructions | [GitHub Docs](https://docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions-for-github-copilot) |
| Managing Copilot policies | [GitHub Docs](https://docs.github.com/en/copilot/managing-copilot/managing-copilot-for-your-enterprise/managing-policies-and-features-for-copilot-in-your-enterprise) |
| About rulesets | [GitHub Docs](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets) |

---

*Last updated: April 2026*
