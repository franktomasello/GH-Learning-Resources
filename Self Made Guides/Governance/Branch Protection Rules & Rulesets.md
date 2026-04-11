# 🛡️ Branch Protection Rules & Rulesets Configuration Runbook

> **Complete guide to configuring branch protection at the repo, org, and enterprise level using classic rules and modern rulesets**

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

## 📝 Resources

| Resource | Link |
|----------|------|
| **About rulesets** | https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets |
| **Org-level rulesets** | https://docs.github.com/en/organizations/managing-organization-settings/creating-rulesets-for-repositories-in-your-organization |
| **About protected branches** | https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches |

---

*Last updated: April 2026*