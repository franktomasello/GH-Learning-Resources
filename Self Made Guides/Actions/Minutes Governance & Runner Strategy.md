# 🏗️ GitHub Actions Minutes Governance & Runner Strategy Runbook

> **Complete guide to managing Actions minutes, controlling costs, configuring runner strategies, and governing workflow usage across your enterprise**

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [📋 Overview](#-overview)
- [1️⃣ Included Minutes](#1️⃣-included-minutes)
- [2️⃣ Overage Rates](#2️⃣-overage-rates)
- [3️⃣ Self-Hosted Runners](#3️⃣-self-hosted-runners)
- [4️⃣ Setting Actions Budgets and Alerts](#4️⃣-setting-actions-budgets-and-alerts)
- [5️⃣ Restricting Which Actions Can Run](#5️⃣-restricting-which-actions-can-run)
- [6️⃣ Runner Groups for Organization Isolation](#6️⃣-runner-groups-for-organization-isolation)
- [7️⃣ Workflow Controls](#7️⃣-workflow-controls)
- [8️⃣ Monitoring Usage](#8️⃣-monitoring-usage)
- [9️⃣ Decision Guide: Which Runner Type to Use](#9️⃣-decision-guide-which-runner-type-to-use)
- [🔟 GitHub-Hosted Runners with Azure Private Networking](#-github-hosted-runners-with-azure-private-networking)
- [1️⃣1️⃣ Self-Hosted Runner Setup](#1️⃣1️⃣-self-hosted-runner-setup)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📚 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Hard-stop budget:** `Enterprise → Billing and licensing → Budgets and alerts → New budget` → **Product-level budget** → **Actions** → scope → amount → **Stop usage when budget limit is reached** → **Create budget**
- **Restrict allowed actions:** `Enterprise → Policies → Actions` → **Allow enterprise, and select non-enterprise, actions and reusable workflows** → **Save**
- **Enterprise runner group:** `Enterprise → Policies → Actions → Runner groups` tab → **New runner group** → **Save group**
- **Org self-hosted runner:** `Org → Settings → Actions → Runners` → **New runner** → **New self-hosted runner**
- **Azure private networking:** `Enterprise → Settings → Hosted compute networking` → **New network configuration ▾** → **Azure private network**

---

## ✅ Accuracy & Click-Path Notes

<details>
<summary><em>Show click-path conventions</em></summary>


- Reviewed against current public GitHub and Microsoft documentation in October 2026. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
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
| GitHub Enterprise Cloud | — | ☐ |
| Enterprise Actions policies, enterprise runner groups, hosted compute networking | GitHub **enterprise owner** | ☐ |
| Budgets and usage reports | **Enterprise owner** or **billing manager** | ☐ |
| Organization runners and runner groups | **Organization owner** (or "Manage organization runners and runner groups" permission) | ☐ |
| Azure subscription, VNET, and subnet (for Azure private networking) | Azure subscription / network owner | ☐ |
| Infrastructure for self-hosted runners (if used) | Platform team | ☐ |

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **GitHub enterprise or organization Actions admin** | Enables the runner group to use Azure private networking. | GitHub → Enterprise → Settings → Hosted compute networking → New network configuration ▾ → Azure private network → name it → Add Azure Virtual Network → paste the network settings resource ID → Add Azure Virtual Network → save the configuration. Then Enterprise → Policies → Actions → Runner groups → New runner group → Organization access → Network configurations: pick the configuration → Create group, and add a larger runner to that group. Handoff: network configuration name and runner group name. |
| **Azure subscription or network Owner** | Creates or validates the VNET, subnet, DNS, routing, and private endpoint dependencies used by GitHub-hosted runners. | Register the `GitHub.Network` resource provider → create (or reuse) a VNET and subnet in a supported region → delegate the subnet to `GitHub.Network/networkSettings` → apply the NSG rules from GitHub's script → create the `GitHub.Network/networkSettings` resource with your enterprise `databaseId` → copy its resource ID. GitHub's docs provide a script for these steps. Handoff: network settings resource ID, subscription, resource group, VNET, and subnet. |

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

GitHub Enterprise Cloud includes a monthly quota for **private and internal** repositories on standard GitHub-hosted runners:

| Plan | Minutes per month | Artifact + Packages storage | Cache (per repository) | Custom image storage |
|------|------------------|------------------------------|------------------------|----------------------|
| **GitHub Enterprise Cloud** | 50,000 | 50 GB (shared with GitHub Packages) | 10 GB | 150 GB |

> 💡 **Tip:** The quota is shared across the enterprise and resets each month. Standard runners are **free** for public repositories, GitHub Pages, and Dependabot. The billing dashboard shows Actions usage as spend (dollars), not minutes.

### Baseline GitHub-hosted runner pricing

| Runner | Billing SKU | Per-minute rate |
|--------|-------------|-----------------|
| **Linux 1-core (x64)** | `actions_linux_slim` | $0.002 |
| **Linux 2-core (x64)** | `actions_linux` | $0.006 |
| **Linux 2-core (arm64)** | `actions_linux_arm` | $0.005 |
| **Windows 2-core (x64)** | `actions_windows` | $0.010 |
| **Windows 2-core (arm64)** | `actions_windows_arm` | $0.010 |
| **macOS 3-core or 4-core** | `actions_macos` | $0.062 |

**Storage beyond the quota:** $0.25 per GB-month (artifacts + Packages), $0.07 per GB-month (cache and custom images), accrued hourly.

> ⚠️ **Important:** Larger runners have their own rates, don't use the included minutes, and are always billed — even for public repositories.

---

## 2️⃣ Overage Rates

| Runner type | Billing behavior |
|-------------|------------------|
| **Standard GitHub-hosted runners** | Use included minutes first, then bill per minute by SKU |
| **Larger runners** | Always billed at the larger-runner rate |
| **Self-hosted runners** | Free — no included minutes used, no per-minute charge |
| **Copilot code review and cloud agent** | Run on Actions and use minutes on private repositories, in addition to AI credits |

> 💡 **Tip:** Without a payment method, usage stops when the quota runs out, and larger runners are blocked entirely.

---

## 3️⃣ Self-Hosted Runners

| Aspect | Detail |
|--------|--------|
| **Cost** | No Actions minutes or per-minute charges (you pay for your own infrastructure) |
| **Infrastructure** | You manage the machine (VM, physical, container, or Kubernetes with ARC) |
| **Best for** | High-volume jobs, special hardware, private network access |

> ⚠️ **Security:** use self-hosted runners only with private repositories — forks of public repositories can run untrusted code on them.

> 🚨 **Minimum runner version:** since **September 29, 2026** (GHE.com: July 31, 2026), self-hosted runners older than **2.329.0** can't register, and runners below the job-execution minimum stop running jobs. Keep runners on auto-update or upgrade them regularly.

---

## 4️⃣ Setting Actions Budgets and Alerts

**👤 Role:** **Enterprise owner** or **billing manager** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **Billing and licensing** → **Budgets and alerts**

**Steps:**

1. Click **New budget**.
2. Under **Budget Type**, choose **Product-level budget** and select **Actions** (or **SKU-level budget** for one runner SKU).
3. Under **Budget scope**, choose **Enterprise**, an **Organization**, a **Repository**, or a **Cost center**.
4. Under **Budget**, enter the amount.
5. To block spending at the limit, select **Stop usage when budget limit is reached**.
6. Under **Alerts**, select **Receive budget threshold alerts** (75%, 90%, 100%) and choose the **Alert Recipients**.
7. Click **Create budget**.

| Option | Effect |
|--------|--------|
| **$0 budget + stop usage** | No paid usage once the included minutes and storage are gone |
| **Custom budget + stop usage** | Jobs run until the cap is reached |
| **Budget without stop usage** | Alerts only — usage continues past the budget |

> ⚠️ **Warning:** a hard-stop budget blocks workflow runs once it's reached. Tell teams before you apply it.

> 💡 Also on this page: **Receive alerts when my included usage reaches 90% and 100%** for the included-minutes quota.

---

## 5️⃣ Restricting Which Actions Can Run

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **Policies** tab → **Actions**

**Steps:**

1. Under **Policies**, choose which organizations can use Actions (all, specific, or none).
2. Choose an actions policy:

| Policy | Effect |
|--------|--------|
| **Allow all actions and reusable workflows** | Anything can run |
| **Allow enterprise actions and reusable workflows** | Only actions in your enterprise's repositories (this blocks `actions/checkout` too) |
| **Allow enterprise, and select non-enterprise, actions and reusable workflows** | Enterprise actions **plus** your allowlist (recommended) |

3. If you chose the third option, select any of **Allow actions created by GitHub**, **Allow Marketplace actions by verified creators**, and **Allow or block specified actions and reusable workflows** (for example `azure/login@*, octo-org/*, !octo-org/risky-action@*`).
4. *(Optional)* Select **Require actions to be pinned to a full-length commit SHA**.
5. Click **Save**.

> 💡 **Tip:** Local actions (`uses: ./...`) are never restricted. Under **Runners** on the same page you can also disable repository-level self-hosted runners.

### Workflow execution protections (GA September 2026)

Control **who** can trigger workflows and **which events** can start them — at enterprise, organization, or repository level, optionally scoped to specific workflow files.

1. Enterprise → **Policies** tab → **Actions** → **Policies** (organizations and repositories: **Settings** → **Actions** → **Policies**).
2. Create a policy: name it, choose an enforcement status (**Evaluate** lets you watch the impact in policy insights first), and target organizations, repositories, or workflow files.
3. Add **actor** rules (users, roles, teams, apps) and **event** rules (allowed triggers).

> ⚠️ **November 2, 2026:** public repositories without an event policy get a default rule that **blocks `pull_request_target`** (currently in evaluate mode). Check policy insights and explicitly allow the trigger where a workflow truly needs it.

---

## 6️⃣ Runner Groups for Organization Isolation

**👤 Role:** GitHub **enterprise owner** · **📍 Portal:** GitHub

**Navigate:** Enterprise → **Policies** tab → **Actions** → **Runner groups** tab

**Steps:**

1. Click **New runner group**.
2. Under **Group name**, type a name.
3. Open the **Organization access** dropdown and choose all organizations or **Selected organizations** (then pick them).
4. Set **Workflow access** — all workflows, or specific workflows (use full refs such as `refs/heads/main`).
5. Click **Save group**.

> ✅ **Result:** only the selected organizations (and workflows) can run jobs on runners in that group.

> 💡 **Organization level:** Org → **Settings** → **Actions** → **Runner groups** → **New runner group**. Runners registered without a group go to the **Default** group.

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

> 🚨 **Node 20 removed (September 23, 2026):** JavaScript actions that still declare `node20` no longer run. Update to current majors (for example `actions/checkout@v7`, `azure/login@v3`) and check third-party actions in your allowlist.

---

## 8️⃣ Monitoring Usage

| Method | How to access | Detail |
|--------|---------------|--------|
| **Usage page** | Enterprise → **Billing and licensing** → **Usage** → **Metered usage** | Filter by product, organization, repository, or SKU |
| **Usage report (CSV)** | Same page → **Get usage report** | Emailed CSV by organization, repository, and SKU |
| **Billing usage API** | `GET /enterprises/{enterprise}/settings/billing/usage` (and `/usage/summary`) | Programmatic usage data |

> 💡 **Tip:** Review usage monthly to catch spikes before they become large overage charges.

---

## 9️⃣ Decision Guide: Which Runner Type to Use

| Scenario | Recommended Runner | Reason |
|----------|--------------------|--------|
| **Standard CI/CD, low-to-medium volume** | GitHub-hosted | Zero maintenance, pre-configured environments |
| **Large builds, need more CPU/RAM** | Larger runners (GitHub-hosted) | Configurable specs up to 64 cores |
| **High volume, cost-sensitive** | Self-hosted | No per-minute charges |
| **Private network access required** | Self-hosted, or larger runners with Azure private networking | Must reach internal resources |
| **Dynamic scaling with Kubernetes** | Actions Runner Controller (ARC) | Auto-scales runner pods in your cluster |
| **Compliance / data residency** | Self-hosted | Full control over where code is built |

---

## 🔟 GitHub-Hosted Runners with Azure Private Networking

**👤 Role:** GitHub **enterprise owner** + Azure network owner · **📍 Portal:** Azure, then GitHub

> 📌 Azure private networking works with **larger runners** (2–64 vCPU Ubuntu and Windows), not standard runners. Initial setup must be done at the **enterprise** level.

**Steps (Azure):** register `GitHub.Network`, create the VNET and subnet, delegate the subnet to `GitHub.Network/networkSettings`, apply the NSG rules, and create the network settings resource for your enterprise (GitHub's docs provide a script). Copy the resource ID.

**Steps (GitHub):**

1. Enterprise → **Settings** → **Hosted compute networking**.
2. Click **New network configuration ▾** → **Azure private network**.
3. Name the configuration, click **Add Azure Virtual Network**, paste the network settings resource ID, and click **Add Azure Virtual Network**.
4. Create an enterprise runner group (Section 6). Under **Network configurations**, select this configuration, then click **Create group**.
5. Add a **larger runner** to that runner group.

> ✅ **Result:** larger runners in the group get a network interface in your subnet and can reach private resources (databases, internal APIs) without exposing them publicly.

> 💡 You can add a **failover network** to a configuration (**Edit configuration** → **Add failover network**).

---

## 1️⃣1️⃣ Self-Hosted Runner Setup

**👤 Role:** **Organization owner** · **📍 Portal:** GitHub + the runner machine

**Navigate:** Organization → **Settings** → **Actions** → **Runners**

**Steps:**

1. Click **New runner**, then **New self-hosted runner**.
2. Select the operating system image and architecture.
3. On the runner machine, run the shown commands in order: download and extract the runner, run `config` with the URL and the time-limited token shown on the page.
4. Run the runner (or install it as a service — on Windows, `config` offers this; on Linux/macOS, install the service afterward).
5. Back on **Runners**, confirm the runner is listed as **Idle**, and the terminal shows `Connected to GitHub` / `Listening for Jobs`.

> 💡 **Tip:** run runners as a service so they restart after reboots, use ephemeral runners or ARC where possible, and never attach self-hosted runners to public repositories.

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
**A:** Use **Get usage report** to find the repositories and SKUs using the most. Move high-volume work to self-hosted runners (free), check macOS, Windows, and larger-runner use (more expensive), add `timeout-minutes` and concurrency limits, and remember that Copilot code review and cloud agent use Actions minutes too. Turn on the 90%/100% included-usage alerts.

---

### Q: My self-hosted runner is online but jobs are not being picked up. What should I check?
**A:** Check that (1) the runner is listed as **Idle** at Org → **Settings** → **Actions** → **Runners**, (2) every label in `runs-on` matches a label on the runner, and (3) the runner's group allows the organization, repository, and workflow (check **Workflow access** too). Then check the runner application logs.

---

### Q: VNET injection for GitHub-hosted runners is not working. What are the common issues?
**A:** Confirm you're using **larger runners** (standard runners can't use it), the subnet is delegated to `GitHub.Network/networkSettings`, the subnet has enough free IPs for your peak concurrency, the region is supported, the NSG rules allow GitHub's required traffic, and the runner group has the network configuration selected. Larger runners in a VNET must use dynamic IPs, not static IPs.

---

### Q: Workflows are running on the wrong runner type (e.g., GitHub-hosted instead of self-hosted). How do I fix this?
**A:** Check the `runs-on` label in your workflow YAML. For self-hosted runners, use labels like `self-hosted` plus any custom labels you assigned. For GitHub-hosted runners, use labels like `ubuntu-latest`. If you have runner groups, verify the group assignment and that the correct org has access. Using runner groups to isolate runner types by organization or purpose helps prevent routing mistakes.

---

### Q: Our Actions budget was hit and workflows are queuing but not running. What do we do?
**A:** Raise the budget or clear **Stop usage when budget limit is reached** at Enterprise → **Billing and licensing** → **Budgets and alerts**. Check for a narrower budget too (organization, repository, or cost center). Longer term, move heavy workloads to self-hosted runners and set `timeout-minutes` everywhere.

---

### Q: How can we prevent developers from using untrusted third-party Actions from the Marketplace?
**A:** At Enterprise → **Policies** → **Actions**, choose **Allow enterprise, and select non-enterprise, actions and reusable workflows**, select **Allow actions created by GitHub**, and list approved third-party actions (for example `azure/login@*`). Consider **Require actions to be pinned to a full-length commit SHA**, then click **Save**.

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

- [GitHub Actions billing](https://docs.github.com/en/billing/concepts/product-billing/github-actions)
- [Setting up budgets](https://docs.github.com/en/billing/how-tos/set-up-budgets)
- [Enforcing policies for GitHub Actions in your enterprise](https://docs.github.com/en/enterprise-cloud@latest/admin/enforcing-policies/enforcing-policies-for-your-enterprise/enforcing-policies-for-github-actions-in-your-enterprise)
- [Adding self-hosted runners](https://docs.github.com/en/actions/how-tos/manage-runners/self-hosted-runners/add-runners)
- [Configuring private networking for GitHub-hosted runners in your enterprise](https://docs.github.com/en/enterprise-cloud@latest/admin/configuring-settings/configuring-private-networking-for-hosted-compute-products/configuring-private-networking-for-github-hosted-runners-in-your-enterprise)

---

*Last updated: October 2026*
