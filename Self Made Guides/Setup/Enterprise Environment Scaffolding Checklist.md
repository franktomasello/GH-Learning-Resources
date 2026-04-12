# 🏗️ GitHub Enterprise Environment Scaffolding Checklist

> **Comprehensive checklist for scaffolding a new GitHub Enterprise Cloud environment from scratch — identity, governance, security, billing, and Copilot**

---

## 📋 Overview

This runbook walks through every major decision and configuration step when standing up a new GitHub Enterprise Cloud environment. Follow the sections in order — each builds on the previous.

| Phase | What You Configure |
|-------|-------------------|
| **Identity Model** | Standard vs EMU vs EMU with data residency |
| **Enterprise & Org Structure** | Enterprises, organizations, teams |
| **Identity & Provisioning** | SAML SSO, OIDC, SCIM |
| **Baseline Governance** | Repo policies, rulesets, Actions, PATs, Apps |
| **Security Controls** | Secret scanning, code scanning, security configurations |
| **Billing** | Azure subscription, cost centers, budgets |
| **Copilot** | Access, models, content exclusions |
| **Validation** | End-to-end smoke tests |

---

## 1️⃣ Choose an Identity Model

| Feature | Standard Enterprise | EMU | EMU with Data Residency |
|---------|-------------------|-----|------------------------|
| **User accounts** | Users create their own github.com accounts | Accounts provisioned and managed by IdP | Same as EMU |
| **SSO** | SAML SSO (optional per org, required at enterprise) | SAML or OIDC (required) | SAML or OIDC (required) |
| **SCIM provisioning** | Available at org level with supported IdPs (Entra ID, Okta) — not available at enterprise level | Required — IdP provisions and deprovisions users | Required |
| **Public repos** | Supported | Not supported | Not supported |
| **External collaboration** | Users can contribute to any public repo | Restricted — EMU users cannot interact outside the enterprise | Restricted |
| **Data residency** | No (US-hosted) | No (US-hosted) | Yes — choose region at setup |

> ⚠️ **Important:** The identity model cannot be changed after the enterprise is created. Choose carefully.

> 💡 **Tip:** If your organization requires public repositories or your developers contribute to open-source, Standard Enterprise is the right choice. If you need full lifecycle control over user accounts, choose EMU.

---

## 2️⃣ Design Enterprise & Organization Structure

### Recommended Structure

```
Enterprise (one per company)
├── Org: platform-engineering      (shared platform, IaC, tooling)
├── Org: business-unit-a           (department or product line)
├── Org: business-unit-b           (department or product line)
├── Org: open-source               (public repos — Standard Enterprise only)
└── Org: sandbox                   (experimentation, training)
```

| Decision | Recommendation |
|----------|---------------|
| **Number of enterprises** | One per company (multi-enterprise adds billing and policy complexity) |
| **Org boundaries** | Align to departments, business units, or product lines |
| **Shared platform org** | Create a dedicated org for IaC, reusable Actions, templates, and internal packages |
| **Sandbox org** | Optional — useful for training and experimentation without affecting production |

---

## 3️⃣ Configure Identity & Provisioning

### For Standard Enterprise (SAML SSO)

**Navigation:**

```
Enterprise → Settings → Authentication security
  → SAML single sign-on → Enable SAML authentication
    → Enter IdP Sign-on URL, Issuer, Public certificate
      → Test SAML configuration → Save
```

> 💡 **Tip:** Enable SAML at the enterprise level to enforce SSO across all organizations. Org-level SAML is also available but enterprise-level is recommended for consistency.

### For EMU (SAML/OIDC + SCIM)

**Navigation:**

```
Enterprise → Settings → Authentication security
  → Configure SAML or OIDC (depending on IdP)
    → Enable SCIM provisioning
      → Generate SCIM token → Configure in your IdP
```

| Step | Action |
|------|--------|
| **1. Configure SSO** | Set up SAML or OIDC in your IdP (Entra ID, Okta, PingFederate) |
| **2. Enable SCIM** | Configure SCIM provisioning in your IdP using the enterprise SCIM endpoint |
| **3. Provision users** | Assign users and groups in your IdP — they will be auto-created in GitHub |
| **4. Map groups to teams** | IdP groups map to GitHub teams for repository access |

### Guest Collaborators (EMU)

**Navigation:**

```
Enterprise → Settings → Policies → Guest collaborators
  → Enable guest collaborators → Configure invitation policies
```

> 💡 **Tip:** Guest collaborators allow external users (contractors, partners) to access specific repositories in an EMU enterprise without being provisioned through the IdP.

---

## 4️⃣ Apply Baseline Governance

### A) Default Repository Visibility

**Navigation:**

```
Enterprise → Settings → Policies → Repository creation
  → Set default visibility (Private recommended)
  → Restrict which visibility levels members can choose
```

| Setting | Recommended Value |
|---------|------------------|
| **Default visibility** | Private |
| **Allow public repos** | Only if Standard Enterprise and intentional |
| **Allow internal repos** | Yes (for cross-org sharing within the enterprise) |

---

### B) Repository Rulesets

**Navigation:**

```
Enterprise → Settings → Policies → Repository rulesets
  → New ruleset
```

| Ruleset Type | Recommended Rules |
|-------------|-------------------|
| **Branch ruleset (main)** | Require PR reviews, require status checks, block force pushes, require signed commits |
| **Tag ruleset** | Restrict who can create release tags |

> 💡 **Tip:** Enterprise-level rulesets cascade to all organizations and repositories. Use them for non-negotiable guardrails (e.g., no force pushes to `main`).

---

### C) Actions Policies

**Navigation:**

```
Enterprise → Settings → Policies → Actions
  → Configure allowed actions and workflows
```

| Setting | Recommended Value |
|---------|------------------|
| **Allow actions** | Allow select actions → GitHub-authored + verified marketplace + specific trusted actions |
| **Fork pull request workflows** | Require approval for first-time contributors |
| **Default workflow permissions** | Read repository contents (not write) |

---

### D) Personal Access Token (PAT) Policies

**Navigation:**

```
Enterprise → Settings → Policies → Personal access tokens
  → Configure PAT policies
```

| Setting | Recommended Value |
|---------|------------------|
| **Fine-grained PATs** | Allow — with approval required for organization access |
| **Classic PATs** | Restrict or block (fine-grained preferred) |
| **Max token lifetime** | Set a maximum (e.g., 90 days) |

---

### E) GitHub App Governance

**Navigation:**

```
Enterprise → Settings → Policies → GitHub Apps
  → Configure app installation policies
```

| Setting | Recommended Value |
|---------|------------------|
| **Who can install apps** | Organization owners only |
| **App approval** | Require enterprise owner approval for new app installations |

---

## 5️⃣ Set Up Security Controls

### A) Enable Security Configurations at Scale

**Navigation:**

```
Organization → Security (sidebar) → Assessments
  → Get started → For all repositories (or Configure in settings)
```

> 💡 **Tip:** Use the Organization-level Security Configurations to apply consistent security settings across all repos. See the [GitHub Secret Protection Enablement Runbook](GitHub%20Secret%20Protection%20Enablement%20Runbook.md) for detailed steps.

---

### B) Secret Scanning + Push Protection

**Navigation:**

```
Organization → Settings → Advanced Security → Configurations
  → Create or edit configuration
    → Secret Protection → Enable
    → Push Protection → Enable
```

| Feature | Description |
|---------|-------------|
| **Secret scanning alerts** | Detect secrets already present in repositories |
| **Push protection** | Block secrets from being pushed going forward |
| **Validity checks** | Verify if detected secrets are still active |

---

### C) Code Scanning with CodeQL

**Navigation:**

```
Organization → Settings → Advanced Security → Configurations
  → Create or edit configuration
    → Code Security → Enable
    → Code scanning → CodeQL → Enable default setup
```

| Setting | Recommended Value |
|---------|------------------|
| **Default setup** | Enable for all supported languages |
| **Schedule** | Weekly + on pull requests |
| **Alert severity threshold** | High and Critical block merges (via rulesets) |

---

## 6️⃣ Configure Billing

### A) Connect Azure Subscription or EA Billing

**Navigation:**

```
Enterprise → Settings → Billing → Payment information
  → Connect Azure subscription (or configure EA billing)
```

> ⚠️ **Important:** An Azure subscription or Enterprise Agreement must be connected before any paid features (Secret Protection, Code Security, Copilot) can be enabled at scale.

---

### B) Create Cost Centers

**Navigation:**

```
Enterprise → Settings → Billing → Cost centers
  → Create cost center → Name → Assign organizations or users
```

| Cost Center Example | Assigned To |
|--------------------|-------------|
| **Platform Engineering** | platform-engineering org |
| **Business Unit A** | business-unit-a org |
| **Copilot Pilot** | Specific user group |

---

### C) Set Budgets and Spending Limits

**Navigation:**

```
Enterprise → Settings → Billing → Budgets
  → Create budget → Set amount → Assign to cost center
    → Configure alerts (50%, 75%, 100% thresholds)
```

> 💡 **Tip:** Set spending alerts well below your actual budget so you have time to react before hitting limits.

---

## 7️⃣ Enable Copilot

### A) Configure Copilot Access

**Navigation:**

```
Enterprise → Settings → AI controls → Copilot → Access
  → Enable for: All organizations / Selected organizations
    → Assign seats: All members / Selected members
```

---

### B) Set Model Policies

**Navigation:**

```
Enterprise → Settings → AI controls → Copilot → Models
  → Enable or disable specific models
```

---

### C) Configure Content Exclusions

**Navigation:**

```
Enterprise → Settings → AI controls → Copilot → Content exclusions
  → Add exclusion rules (by repository or file path patterns)
```

> 💡 **Tip:** Use content exclusions to prevent Copilot from accessing sensitive files (e.g., `**/*.env`, `**/secrets/**`).

---

### D) Custom Instructions

**Navigation:**

```
Enterprise → Settings → AI controls → Copilot → Custom instructions
  → Add coding guidelines, style rules, or organizational standards
```

---

## 8️⃣ Validation Checklist

*Run through this checklist to confirm everything is working end-to-end*

| # | Validation Step | How to Test | Expected Result |
|---|----------------|-------------|-----------------|
| 1 | **SSO works** | Log in with an IdP-managed account | User is authenticated via SAML/OIDC |
| 2 | **SCIM provisions users** (EMU) | Assign a test user in the IdP | User appears in the enterprise within minutes |
| 3 | **SCIM deprovisions users** (EMU) | Unassign the test user in the IdP | User is suspended in GitHub |
| 4 | **Policies cascade** | Check an org-level repo for enterprise ruleset | Enterprise rulesets appear and are enforced |
| 5 | **Default repo visibility** | Create a new repo in any org | Default visibility matches enterprise policy |
| 6 | **Actions policies enforced** | Try to use a disallowed action in a workflow | Workflow fails with a policy error |
| 7 | **Secret scanning active** | Push a test secret to a test repo | Alert is generated (or push is blocked) |
| 8 | **Code scanning active** | Open a PR with a known vulnerability pattern | CodeQL flags the issue |
| 9 | **Copilot available** | Open VS Code with Copilot extension, sign in | Copilot provides suggestions |
| 10 | **Billing connected** | Check Enterprise → Billing | Azure subscription or EA is active, usage is tracked |
| 11 | **Cost centers reporting** | Check Enterprise → Billing → Cost centers | Usage is allocated to the correct cost centers |

> ✅ **Result:** If all validation steps pass, your GitHub Enterprise Cloud environment is scaffolded and ready for onboarding teams.

---

## ❓ Common Questions & Troubleshooting

### Q: Should we use one enterprise or multiple enterprises?
**A:** Use one enterprise per company in almost all cases. Multiple enterprises add significant complexity: separate billing, separate audit logs, separate policy governance, and no shared visibility. Only consider multiple enterprises when you have hard compliance boundaries (e.g., FedRAMP vs non-FedRAMP workloads), completely independent IdPs that cannot federate, or legally distinct entities with no shared governance. If in doubt, start with one enterprise and use organizations for separation.

---

### Q: How many organizations should we create inside our enterprise?
**A:** Keep the number low. Create separate organizations only when you need distinct admin models, compliance boundaries, cost centers for billing, or different IdPs (standard enterprise with per-org SAML). Do not create one org per team or one org per project — use teams and repositories within a single org instead. A typical enterprise has 2-5 organizations (e.g., engineering, platform, security, sandbox).

---

### Q: When should we use internal vs private repository visibility?
**A:** Use **internal** for repositories that should be discoverable and readable by everyone in the enterprise across all organizations — this enables InnerSource and knowledge sharing. Use **private** for repositories with restricted access where only explicitly granted users or teams should see the code. A good default is to make most repos internal and reserve private for sensitive or regulated code.

---

### Q: We have shared platform repos (Actions, templates, IaC) — where should they live?
**A:** Create a dedicated "platform" or "shared" organization for cross-cutting resources like reusable Actions, workflow templates, Terraform modules, and internal packages. Set these repos to **internal** visibility so all enterprise members can use them. This avoids duplication across orgs and establishes a single source of truth for shared tooling. Grant write access only to the platform team.

---

### Q: Enterprise policies I set are not cascading to organizations as expected — what is happening?
**A:** Enterprise policies have three modes: **Enforced** (applies to all orgs, cannot be overridden), **Allowed** (org owners can enable/disable within the enterprise's allowed range), and **No policy** (fully delegated to org owners). If you set a policy at the enterprise level but org owners can still override it, you likely chose "Allow" or "No policy" instead of "Enforce." Navigate to Enterprise > Settings > Policies and check the enforcement level for each policy. Rulesets set at the enterprise level always cascade and cannot be overridden.

---

### Q: We already set up our environment but realize we chose the wrong identity model — can we switch?
**A:** No. The identity model (Standard vs EMU vs EMU with Data Residency) is set at enterprise creation and cannot be changed. Switching requires creating a new enterprise with the correct identity model, reconfiguring identity and provisioning, and migrating all repositories using GitHub Enterprise Importer (GEI). Treat this as a 4-8 week migration project. This is why the identity model decision in Step 1 is the most important choice in this guide.

---

### Q: How should we handle cost allocation across multiple business units?
**A:** Use GitHub's Cost Centers feature (Enterprise > Billing > Cost centers). Create a cost center for each business unit or department and assign organizations or specific user groups to each. This allows you to track and allocate metered usage (Copilot, Actions, Packages, GHAS) per cost center. Set spending budgets with alerts at 50%, 75%, and 100% thresholds to prevent surprise overages.

---

### Q: Our security team wants to enable Advanced Security (GHAS) for all repos — should we do it at once?
**A:** Enable incrementally. Start by applying the GitHub recommended security configuration to a pilot set of repositories or one organization. Review the initial alerts (secret scanning, code scanning) and establish a triage process before rolling out broadly. Enabling GHAS across hundreds of repos at once can generate a flood of alerts that overwhelm teams. Use org-level Security Configurations to apply settings consistently, and ramp up over 2-4 weeks.

---

## 📝 Resources

- [About GitHub Enterprise Cloud](https://docs.github.com/en/enterprise-cloud@latest/admin/overview/about-github-enterprise-cloud)
- [Setting up your enterprise](https://docs.github.com/en/enterprise-cloud@latest/admin/setting-up-your-enterprise)
- [Enterprise Managed Users](https://docs.github.com/en/enterprise-cloud@latest/admin/concepts/identity-and-access-management/enterprise-managed-users)

---

*Last updated: April 2026*
