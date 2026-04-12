# 🛡️ Branch Protection Rules & Rulesets Configuration Runbook

> **Complete guide to configuring branch protection at the repo, org, and enterprise level using classic rules and modern rulesets**

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Classic rule (repo):** `Repo → Settings → Branches → Add branch protection rule`
- **Repo-level ruleset:** `Repo → Settings → Rules → Rulesets → New ruleset → New branch ruleset`
- **Org-level ruleset:** `Org → Settings → Rules → Rulesets → New ruleset → New branch ruleset`
- **Enterprise-level ruleset:** `Enterprise → Settings → Policies → Repository rulesets → New ruleset`
- **CODEOWNERS:** Create `.github/CODEOWNERS` file and enable "Require review from Code Owners" in protection rule

---

## ✅ Prerequisites

| Requirement | Status |
|-------------|--------|
| Repo admin (for repo-level rules), org owner (for org-level), or enterprise owner (for enterprise-level) | ☐ |
| GitHub Enterprise Cloud (for org-level and enterprise-level rulesets) | ☐ |
| Branch naming conventions established | ☐ |
| CI/CD status check names identified (if requiring status checks) | ☐ |

---

## 📋 Overview

This runbook covers every method for enforcing branch protection in GitHub:

| Method | Scope | Best For |
|--------|-------|----------|
| **Classic branch protection** | Single repository | Legacy setups, simple rules |
| **Repo-level rulesets** | Single repository | Modern, flexible protection with bypass lists |
| **Org-level rulesets** | All repos in an organization | Consistent policy across teams |
| **Enterprise-level rulesets** | All orgs in the enterprise | Enterprise-wide governance |
| **CODEOWNERS** | Path-specific within a repo | Requiring domain experts to approve specific files |

> 💡 **Tip:** GitHub recommends **rulesets** over classic branch protection rules. Rulesets support bypass lists, layering, and org/enterprise-level enforcement.

---

## 1️⃣ Classic Branch Protection (Repo-Level)

*Legacy approach — still supported but rulesets are preferred.*

### Configure a Classic Branch Protection Rule

**Navigation:**

```
Repository → Settings → Branches (sidebar, under "Code and automation")
  → Add branch protection rule
```

**Steps:**

1. Enter a **Branch name pattern** (e.g., `main`, `release/*`)
2. Check **Require a pull request before merging**
3. Set **Required number of approvals** (e.g., `1` or `2`)
4. (Optional) Check additional protections:

| Setting | Description |
|---------|-------------|
| **Dismiss stale PR approvals** | Re-require approval when new commits are pushed |
| **Require review from code owners** | CODEOWNERS must approve changes to their paths |
| **Require status checks to pass** | CI must pass before merging |
| **Require branches to be up to date** | Branch must be current with the base branch |
| **Require signed commits** | All commits must be GPG signed |
| **Include administrators** | Apply rules to admins too |

5. Click **Create** (or **Save changes**)

> ✅ **Result:** The branch matching your pattern is protected with the configured rules.

---

## 2️⃣ Repo-Level Rulesets (Modern, Recommended)

*Rulesets provide more flexibility than classic rules, including bypass lists and layering.*

### Create a Repo-Level Ruleset

**Navigation:**

```
Repository → Settings → Rules (sidebar) → Rulesets
  → New ruleset → New branch ruleset
```

**Steps:**

1. Enter a **Ruleset name** (e.g., `Main branch protection`)
2. Set **Enforcement status** to **Active**
3. Under **Target branches**, click **Add target** → select **Default branch** (or add specific patterns)
4. Under **Branch rules**, enable:

| Rule | Configuration |
|------|--------------|
| **Restrict deletions** | Prevent branch deletion |
| **Require a pull request before merging** | Enable, then set required approvals (e.g., `1` or `2`) |
| **Require status checks to pass** | Add required checks by name |
| **Block force pushes** | Prevent force pushes to protected branches |

5. (Optional) Configure **Bypass list** — see Section 8 below
6. Click **Create**

> ✅ **Result:** The ruleset is active and enforcing rules on your target branches.

---

## 3️⃣ Org-Level Rulesets (Enforce Across All Repos)

*Apply consistent branch protection across every repository in the organization.*

### Create an Org-Level Ruleset

**Navigation:**

```
Organization → Settings → Rules (sidebar) → Rulesets
  → New ruleset → New branch ruleset
```

**Steps:**

1. Enter a **Ruleset name** (e.g., `Org-wide main protection`)
2. Set **Enforcement status** to **Active**
3. Under **Target repositories**, choose:

| Option | Effect |
|--------|--------|
| **All repositories** | Applies to every repo in the org |
| **Dynamic list by name** | Target repos matching a naming pattern |
| **Select repositories** | Manually pick specific repos |

4. Under **Target branches**, add **Default branch** (or specific patterns)
5. Configure branch rules (require PR, approvals, status checks, etc.)
6. (Optional) Configure **Bypass list** for admin roles or automation
7. Click **Create**

> ✅ **Result:** All targeted repositories enforce the same branch protection rules.

---

## 4️⃣ Enterprise-Level Rulesets

*Enforce policies across all organizations in the enterprise.*

### Create an Enterprise-Level Ruleset

**Navigation:**

```
Enterprise → Settings → Policies (sidebar) → Repository rulesets
  → New ruleset → New branch ruleset
```

**Steps:**

1. Enter a **Ruleset name** (e.g., `Enterprise default branch policy`)
2. Set **Enforcement status** to **Active**
3. Under **Target organizations**, choose all or specific orgs
4. Under **Target repositories**, choose all or specific patterns
5. Under **Target branches**, add **Default branch**
6. Configure branch rules (require PR, approvals, status checks, etc.)
7. Click **Create**

> ⚠️ **Important:** Enterprise-level rulesets cannot be overridden by org or repo admins. Use this for non-negotiable policies only.

---

## 5️⃣ Finding Which Rule Is Enforcing Approval Requirements

*When a PR requires approvals and you need to find the source of that requirement:*

**Check in this order:**

| Priority | Where to Check | Navigation |
|----------|---------------|------------|
| 1 | **Repo-level rulesets** | `Repo → Settings → Rules → Rulesets` |
| 2 | **Classic branch protection** | `Repo → Settings → Branches` |
| 3 | **Org-level rulesets** | `Org → Settings → Rules → Rulesets` |
| 4 | **Enterprise-level rulesets** | `Enterprise → Settings → Policies → Repository rulesets` |

> 💡 **Tip:** On a pull request, click the merge check details to see which specific rule or ruleset is requiring the approval.

---

## 6️⃣ CODEOWNERS for Path-Specific Approvals

*Require specific teams or individuals to approve changes to files they own.*

### Set Up CODEOWNERS

**Steps:**

1. Create a `CODEOWNERS` file in one of these locations:
   - `.github/CODEOWNERS`
   - `docs/CODEOWNERS`
   - Root of the repository

2. Define ownership rules:

```
# Global owners (fallback)
*       @org/platform-team

# Frontend paths
/src/frontend/   @org/frontend-team

# Infrastructure
/terraform/      @org/infra-team

# Sensitive configs
/.github/workflows/   @org/devops-leads
```

3. Enable **Require review from code owners** in your branch protection rule or ruleset

> ⚠️ **Prerequisite:** The branch must have either a classic branch protection rule or a ruleset with "Require a pull request before merging" enabled for CODEOWNERS to take effect.

---

## 7️⃣ Recommended Baseline Configuration

*A common starting point for most organizations:*

| Setting | Recommended Value |
|---------|-------------------|
| **Required approvals** | 1-2 (depending on team size) |
| **Dismiss stale approvals** | Enabled |
| **Require CODEOWNERS review** | Enabled for critical paths |
| **Require status checks** | Enabled (CI must pass) |
| **Block force pushes** | Enabled |
| **Restrict deletions** | Enabled |
| **Method** | Org-level ruleset (for consistency) |

> 💡 **Tip:** Start with 1 required approval for most repos and 2 for critical/production repos. Use CODEOWNERS for path-specific expertise requirements.

---

## 8️⃣ Bypass Lists for Admins & Automation (Rulesets Feature)

*Rulesets allow specific roles or apps to bypass rules — classic branch protection does not support this.*

### Configure a Bypass List

**Navigation (within any ruleset):**

```
Ruleset → Bypass list → Add bypass
```

**Bypass actor options:**

| Actor Type | Example Use Case |
|------------|-----------------|
| **Repository admin** | Emergency hotfixes |
| **Organization admin** | Cross-repo maintenance |
| **Specific team** | Release engineering team |
| **GitHub App** | CI/CD bots (e.g., Dependabot, deploy bot) |

**Bypass modes:**

| Mode | Description |
|------|-------------|
| **Always** | Actor can bypass without review |
| **Pull requests only** | Actor must still use a PR but can bypass approval requirements |

> ⚠️ **Important:** Keep bypass lists minimal. Audit bypass usage regularly to ensure it is not being overused.

---

## ❓ Common Questions & Troubleshooting

### Q: I created a branch protection rule or ruleset, but it does not seem to be enforcing. What should I check?
**A:** First, verify the branch name pattern targets the correct branch (e.g., `main` vs `master`, or that a wildcard like `release/*` matches your branch names). For rulesets, confirm the enforcement status is set to "Active" -- rulesets in "Evaluate" or "Disabled" mode do not enforce. Also check whether the rule is at the repo, org, or enterprise level, as a higher-level rule may be taking precedence.

---

### Q: Our admin can bypass branch protection and push directly to the protected branch. How do we prevent this?
**A:** Classic branch protection rules allow repository admins to bypass protections by default (unless "Include administrators" is checked). For stricter enforcement, use rulesets instead. Rulesets support explicit bypass lists, so you can exclude admins from the bypass list entirely, or require them to go through the same PR process as everyone else.

---

### Q: Should we use rulesets or classic branch protection rules? What is the difference?
**A:** Rulesets are the newer, recommended approach. They offer several advantages over classic rules: explicit bypass lists (control exactly who can bypass), layering (multiple rulesets can apply to the same branch), org-level and enterprise-level enforcement, and the ability to target repos dynamically by name pattern. Classic branch protection is still supported but lacks these capabilities. Use rulesets for new configurations and consider migrating existing classic rules.

---

### Q: How do I require CODEOWNERS approval for changes to specific file paths?
**A:** First, create a `CODEOWNERS` file in `.github/CODEOWNERS`, `docs/CODEOWNERS`, or the repository root, defining ownership rules for paths. Then enable "Require review from Code Owners" in your branch protection rule or ruleset. When a PR modifies files matched by a CODEOWNERS entry, the designated owner(s) must approve before the PR can merge.

---

### Q: CI status checks are passing but not blocking the merge when they fail. What is wrong?
**A:** Verify that the status check name in the branch protection rule or ruleset matches the exact name of the check as it appears in the PR (this is case-sensitive). A common issue is a mismatch between the check name configured in the rule and the actual job or workflow name. Check the PR's "Checks" tab to see the exact name GitHub is receiving, then update the protection rule to match.

---

### Q: Can multiple rulesets apply to the same branch at the same time?
**A:** Yes, rulesets are designed to layer. When multiple rulesets target the same branch, all rules from all matching rulesets are combined. The most restrictive setting wins -- for example, if one ruleset requires 1 approval and another requires 2, the branch will require 2 approvals. This layering works across repo-level, org-level, and enterprise-level rulesets.

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Responsible AI Guardrails | `Copilot/Responsible AI Guardrails.md` |
| Enterprise Environment Scaffolding Checklist | `Setup/Enterprise Environment Scaffolding Checklist.md` |
| Code Scanning (CodeQL) Enablement & Troubleshooting | `Security/Code Scanning (CodeQL) Enablement & Troubleshooting.md` |
| Copilot Admin Controls (Models, Content Exclusion, Instructions) | `Copilot/Admin Controls (Models, Content Exclusion, Instructions).md` |

---

## 📝 Resources

| Resource | Link |
|----------|------|
| **About rulesets** | [GitHub Docs](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets) |
| **Org-level rulesets** | [GitHub Docs](https://docs.github.com/en/organizations/managing-organization-settings/creating-rulesets-for-repositories-in-your-organization) |
| **About protected branches** | [GitHub Docs](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches) |

---

*Last updated: April 2026*