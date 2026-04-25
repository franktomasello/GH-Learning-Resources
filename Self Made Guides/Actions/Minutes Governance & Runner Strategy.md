# 🏗️ GitHub Actions Minutes Governance & Runner Strategy Runbook

> **Complete guide to managing Actions minutes, controlling costs, configuring runner strategies, and governing workflow usage across your enterprise**

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [📋 Overview](#-overview)
- [1️⃣ Included Minutes](#1-included-minutes)
- [2️⃣ Overage Rates](#2-overage-rates)
- [3️⃣ Self-Hosted Runners](#3-self-hosted-runners)
- [4️⃣ Setting Actions Budgets and Alerts](#4-setting-actions-budgets-and-alerts)
- [5️⃣ Restricting Which Actions Can Run](#5-restricting-which-actions-can-run)
- [6️⃣ Runner Groups for Organization Isolation](#6-runner-groups-for-organization-isolation)
- [7️⃣ Workflow Controls](#7-workflow-controls)
- [8️⃣ Monitoring Usage](#8-monitoring-usage)
- [9️⃣ Decision Guide: Which Runner Type to Use](#9-decision-guide-which-runner-type-to-use)
- [🔟 GitHub-Hosted Runners with Azure VNET Injection](#-github-hosted-runners-with-azure-vnet-injection)
- [1️⃣1️⃣ Self-Hosted Runner Setup](#11-self-hosted-runner-setup)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📚 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Set hard-stop budget:** `Enterprise → Billing and licensing → Budgets and alerts → New budget → Product: Actions → Stop usage when budget limit is reached`
- **Restrict allowed Actions:** `Enterprise → Settings → Policies → Actions → Allow select actions → Configure allowlist`
- **Create runner group:** `Enterprise → Settings → Actions → Runner groups → New runner group`
- **Add self-hosted runner:** `Org → Settings → Actions → Runners → New self-hosted runner`
- **Monitor usage:** `Enterprise → Billing and licensing → Usage report → Download CSV`

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
| Enterprise owner role (for enterprise-level policies and billing budgets) | ☐ |
| GitHub Enterprise Cloud account | ☐ |
| Azure subscription (if using VNET injection for GitHub-hosted runners) | ☐ |
| Infrastructure for self-hosted runners (if applicable) | ☐ |

---

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **GitHub enterprise or organization Actions admin** | Enables the runner group to use Azure private networking. | Enterprise path: GitHub → profile photo → Your enterprises → [enterprise] → Settings → Actions → Runner groups → [runner group] → Azure private networking → Configure → select the approved Azure networking configuration → Save. Organization path: GitHub → profile photo → Your organizations → [organization] → Settings → Actions → Runner groups → [runner group] → Azure private networking → Configure → select the approved Azure networking configuration → Save. Handoff: runner group name and confirmation that Azure private networking is enabled. |
| **Azure subscription or network Owner** | Creates or validates the VNET, subnet, DNS, routing, and private endpoint dependencies used by GitHub-hosted runners. | Azure portal → Virtual networks → [VNET] → Subnets → + Subnet or [existing subnet] → validate address range and delegation requirements → Network security groups → [NSG] → Inbound security rules and Outbound security rules → validate required traffic → Private DNS zones or Private endpoints as needed. Handoff: subscription ID, resource group, VNET name, subnet name, region, and any required DNS/private endpoint details. |

---

## 📋 Overview

This runbook covers included minutes, overage pricing, spending controls, runner strategy decisions, and governance mechanisms for GitHub Actions at the enterprise level.

| Topic | Key Question |
|-------|-------------|
| **Included minutes** | How many minutes come with GHEC? |
| **Overage rates** | What does it cost when you exceed the included pool? |
| **Budgets and alerts** | How do I cap or control overage spend? |
| **Runner strategy** | When should I use self-hosted vs GitHub-hosted runners? |
| **Governance** | How do I restrict which Actions can run? |

---

## 1️⃣ Included Minutes

GitHub Enterprise Cloud includes a shared pool of Actions minutes per month:

| Plan | Included Minutes (Linux) | Storage |
|------|-------------------------|---------|
| **GHEC** | 50,000 minutes/month | 50 GB |

> 💡 **Tip:** The 50,000 minutes are a shared enterprise pool across all organizations. The billing dashboard may show Actions usage as spend rather than raw minutes.

### Baseline GitHub-Hosted Runner Pricing

| Runner OS | Billing SKU | Per-Minute Rate |
|-----------|-------------|-----------------|
| **Linux 1-core** | `actions_linux_slim` | $0.002/min |
| **Linux 2-core** | `actions_linux` | $0.006/min |
| **Windows 2-core** | `actions_windows` | $0.010/min |
| **macOS 3-core or 4-core** | `actions_macos` | $0.062/min |

> ⚠️ **Important:** Larger runners have separate rates, are not covered by included minutes, and are charged even for public repositories.

---

## 2️⃣ Overage Rates

When included minutes are exhausted, per-minute charges apply:

| Runner Type | Billing Behavior |
|-------------|------------------|
| **Standard GitHub-hosted runners** | Use included minutes first, then bill by runner SKU after the included amount is exhausted |
| **Larger runners** | Always bill at the larger-runner SKU rate; included minutes do not apply |
| **Self-hosted runners** | Do not consume included minutes and do not incur GitHub per-minute runner charges |

> 💡 **Tip:** Self-hosted runners do **NOT** consume included minutes and do **NOT** incur overage charges. They are the primary cost-control lever for high-volume workloads.

---

## 3️⃣ Self-Hosted Runners

Self-hosted runners run on your own infrastructure and do not consume any included minutes or incur per-minute charges.

| Aspect | Detail |
|--------|--------|
| **Cost** | Zero Actions minutes consumed |
| **Infrastructure** | You manage the machine (VM, physical, container) |
| **Best for** | High-volume jobs, specialized hardware, private network access |

---

## 4️⃣ Setting Actions Budgets and Alerts

Control overage costs by creating a budget for GitHub Actions at the enterprise level.

**Navigation:**

```
Enterprise → Billing and licensing
  → Budgets and alerts → New budget
    → Product: Actions
```

| Option | Effect |
|--------|--------|
| **$0 budget with stop usage enabled** | No paid overage allowed after included minutes/storage are exhausted |
| **Custom budget with stop usage enabled** | Jobs can run until the budget cap is reached |
| **Budget without stop usage** | Alerts only; usage can continue beyond the budget |

> ⚠️ **Warning:** A hard-stop budget can block workflow execution once the budget is exhausted. Communicate this to teams before applying.

---

## 5️⃣ Restricting Which Actions Can Run

Control which Actions are allowed across the enterprise to prevent supply chain risks.

**Navigation:**

```
Enterprise → Settings → Policies (sidebar)
  → Actions → Allow specific actions → Configure allowlist
```

| Policy | Effect |
|--------|--------|
| **Allow all actions** | Any action from GitHub Marketplace or custom repos can run |
| **Allow local actions only** | Only actions defined in the repository can run |
| **Allow select actions** | Specify an allowlist of permitted actions (recommended) |

> 💡 **Tip:** Use the allowlist approach to permit only verified, trusted actions. This prevents developers from pulling in unvetted third-party actions.

---

## 6️⃣ Runner Groups for Organization Isolation

Runner groups let you assign runners to specific organizations, controlling which orgs can use which runner infrastructure.

**Navigation:**

```
Enterprise → Settings → Actions (sidebar)
  → Runner groups → New runner group
    → Name the group → Assign to specific organizations
```

> ✅ **Result:** Only the selected organizations can schedule jobs on runners in that group.

> 💡 **Tip:** Use runner groups to isolate production-grade runners from development teams, or to give specific orgs access to specialized hardware (GPU, ARM, etc.).

---

## 7️⃣ Workflow Controls

Use these mechanisms to govern workflow behavior and resource consumption:

| Control | Purpose | Example |
|---------|---------|---------|
| **`timeout-minutes`** | Limit how long a job can run | `timeout-minutes: 30` |
| **Concurrency groups** | Prevent duplicate runs | `concurrency: { group: deploy-prod, cancel-in-progress: true }` |
| **Environment protection rules** | Require approvals before deployment | Configure required reviewers on the environment |

> 💡 **Tip:** Set `timeout-minutes` on every workflow to prevent runaway jobs from consuming your entire minutes pool. The default timeout is 6 hours.

---

## 8️⃣ Monitoring Usage

| Method | How to Access | Detail |
|--------|---------------|--------|
| **Billing reports** | Enterprise → Billing and licensing | Download CSV of Actions usage by org and repo |
| **Actions usage API** | REST API | Programmatic access to billing and usage data |

> 💡 **Tip:** Set up a monthly review of billing reports to catch unexpected usage spikes early, before they turn into large overage charges.

---

## 9️⃣ Decision Guide: Which Runner Type to Use

| Scenario | Recommended Runner | Reason |
|----------|--------------------|--------|
| **Standard CI/CD, low-to-medium volume** | GitHub-hosted | Zero maintenance, pre-configured environments |
| **Large builds, need more CPU/RAM** | Larger runners (GitHub-hosted) | Configurable specs up to 64 cores |
| **High volume, cost-sensitive** | Self-hosted | No per-minute charges |
| **Private network access required** | Self-hosted or Azure VNET | Must reach internal resources |
| **Dynamic scaling with Kubernetes** | Actions Runner Controller (ARC) | Auto-scales runner pods in your cluster |
| **Compliance / data residency** | Self-hosted | Full control over where code is built |

---

## 🔟 GitHub-Hosted Runners with Azure VNET Injection

For GitHub-hosted runners that need access to private network resources, configure Azure private networking.

**Navigation:**

```
Enterprise → Settings → Actions (sidebar)
  → Runner groups → Select a runner group
    → Azure private networking → Configure
```

> ✅ **Result:** GitHub-hosted runners in this group can reach resources inside your Azure VNET (databases, internal APIs, etc.) without exposing them to the public internet.

> ⚠️ **Important:** Azure VNET injection requires an Azure subscription and network configuration. Work with your network team to set up the VNET and subnet.

---

## 1️⃣1️⃣ Self-Hosted Runner Setup

**Navigation:**

```
Organization → Settings → Actions (sidebar)
  → Runners → New self-hosted runner
    → Select OS and architecture → Follow setup instructions
```

**Steps:**

1. Choose the operating system (Linux, macOS, Windows)
2. Choose the architecture (x64, ARM, ARM64)
3. Download and extract the runner application
4. Configure the runner with the provided token
5. Start the runner as a service

> 💡 **Tip:** For production use, always run self-hosted runners as a service so they restart automatically after reboots. Never run self-hosted runners on public repositories due to security risks.

## 🧯 Known Errors & Resolutions

<details>
<summary><em>Show known errors table</em></summary>


> This section lists the known product errors and admin-facing symptoms that commonly occur with this workflow. Exact message text can vary by product rollout, tenant policy, and provider, so use the log or settings page named in the resolution to confirm the root cause.

| Error or symptom | Likely cause | Resolution |
|------------------|--------------|------------|
| **Page, tab, or button is missing** | Wrong account context, missing admin role, unavailable plan/add-on, or feature rollout not enabled for the selected enterprise/org/repo. | Switch to the correct account and scope, confirm the prerequisite role, verify licensing or add-on activation, then refresh the page. If the control is still absent, use the direct settings URL from the relevant GitHub Docs page and confirm the feature is available for your plan. |
| **Changes appear saved but behavior does not change** | Policy inheritance, cached UI state, propagation delay, or an overlapping enterprise/org/repo policy. | Reopen the settings page, verify the effective policy at the lowest affected scope, wait for propagation where documented, and check for a stricter policy at an enterprise or organization level. |
| **403, forbidden, or resource not accessible** | The signed-in user or token can see the page but lacks the specific permission for the action. | Use an enterprise owner, organization owner, repository admin, or token with the exact scopes/permissions listed in the runbook. For SAML-protected orgs, authorize the token or SSH key for SSO before retrying. |
| **Workflow is queued, blocked, or canceled for billing** | Included minutes are exhausted, no payment method is available, a hard-stop budget is reached, or larger runners require paid billing. | Check Billing and licensing > Usage and Budgets and alerts, add or verify payment, adjust budgets, or move appropriate workloads to self-hosted runners. |
| **Runner label is not found or job never starts** | The workflow references a label that no online runner has, or the runner group is not available to the repository. | Confirm the exact labels in Actions > Runners, put the runner in an accessible group, and update `runs-on` to match. |
| **GitHub App token returns 403 or 404** | The app is not installed on the repository or lacks the specific repository permission. | Install the app on the target repo, grant the narrow required permissions, regenerate the installation token, and retry. |
| **OIDC token is unavailable** | The workflow lacks `permissions: id-token: write` or is running from an event where the job cannot request a token. | Add the id-token permission at workflow or job scope and test with the OIDC debugger before creating cloud trust conditions. |
| **Azure federated credential rejects the token** | Issuer, audience, subject, branch, environment, or GHE.com token issuer does not match the credential. | Compare the live token claims to the federated credential and update the Azure issuer/subject/audience exactly, including GHE.com issuer differences. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>


### Q: Our included minutes are exhausted mid-month. How do we avoid this recurring issue?
**A:** Move high-volume or long-running workloads to self-hosted runners, which consume zero included minutes. Also review macOS, Windows, larger-runner, and GPU usage because those SKUs cost more than baseline Linux runners. Set Actions budgets and alerts to control overage costs, and use billing reports to identify which repos or workflows are consuming the most spend.

---

### Q: My self-hosted runner is online but jobs are not being picked up. What should I check?
**A:** Verify three things: (1) the runner shows as "Online" in Org Settings > Actions > Runners, (2) the `runs-on` label in your workflow matches a label assigned to the runner exactly, and (3) the runner group the runner belongs to is configured to allow the organization that owns the workflow. Also check the runner application logs for errors.

---

### Q: VNET injection for GitHub-hosted runners is not working. What are the common issues?
**A:** Ensure the Azure subnet is delegated to `GitHub.Network/networkSettings` (this is a specific Azure delegation requirement). Verify the subnet has enough available IP addresses for the number of concurrent runners you expect. Also confirm the Azure subscription, VNET, and subnet are in a supported region, and that the runner group is correctly configured with the Azure private networking settings.

---

### Q: Workflows are running on the wrong runner type (e.g., GitHub-hosted instead of self-hosted). How do I fix this?
**A:** Check the `runs-on` label in your workflow YAML. For self-hosted runners, use labels like `self-hosted` plus any custom labels you assigned. For GitHub-hosted runners, use labels like `ubuntu-latest`. If you have runner groups, verify the group assignment and that the correct org has access. Using runner groups to isolate runner types by organization or purpose helps prevent routing mistakes.

---

### Q: Our Actions budget was hit and workflows are queuing but not running. What do we do?
**A:** When a hard-stop budget is reached, GitHub-hosted runner jobs can queue or stop executing. To resolve immediately, increase or disable the stop-usage budget under Enterprise > Billing and licensing > Budgets and alerts. For a longer-term fix, move high-consumption workloads to self-hosted runners and set `timeout-minutes` on all workflows to prevent runaway jobs from consuming your budget.

---

### Q: How can we prevent developers from using untrusted third-party Actions from the Marketplace?
**A:** Configure an Actions allowlist at the enterprise level (Enterprise > Settings > Policies > Actions > Allow select actions). Specify the exact actions and versions that are permitted (e.g., `actions/checkout@v4`, `azure/login@v2`). This prevents developers from pulling in unvetted third-party actions that could introduce supply chain risks.

</details>

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| GitHub App for CI/CD (No Seat Cost) | `Actions/GitHub App for CI-CD (No Seat Cost).md` |
| OIDC Federation for Azure Deployments | `Actions/OIDC Federation for Azure Deployments.md` |
| Cost Centers & Department Billing | `Billing/Cost Centers & Department Billing.md` |
| Enterprise Environment Scaffolding Checklist | `Setup/Enterprise Environment Scaffolding Checklist.md` |

---

## 📚 Resources

- [About billing for GitHub Actions](https://docs.github.com/en/billing/managing-billing-for-github-actions/about-billing-for-github-actions)
- [About self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners)
- [About Azure private networking for GitHub-hosted runners](https://docs.github.com/en/organizations/managing-organization-settings/about-azure-private-networking-for-github-hosted-runners-in-your-organization)

---

*Last updated: April 2026*
