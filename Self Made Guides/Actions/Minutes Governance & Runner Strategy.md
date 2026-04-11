# 🏗️ GitHub Actions Minutes Governance & Runner Strategy Runbook

> **Complete guide to managing Actions minutes, controlling costs, configuring runner strategies, and governing workflow usage across your enterprise**

---

## 📋 Overview

This runbook covers included minutes, overage pricing, spending controls, runner strategy decisions, and governance mechanisms for GitHub Actions at the enterprise level.

| Topic | Key Question |
|-------|-------------|
| **Included minutes** | How many minutes come with GHEC? |
| **Overage rates** | What does it cost when you exceed the included pool? |
| **Spending limits** | How do I cap or control overage spend? |
| **Runner strategy** | When should I use self-hosted vs GitHub-hosted runners? |
| **Governance** | How do I restrict which Actions can run? |

---

## 1️⃣ Included Minutes

GitHub Enterprise Cloud includes a shared pool of Actions minutes per month:

| Plan | Included Minutes (Linux) | Storage |
|------|-------------------------|---------|
| **GHEC** | 50,000 minutes/month | 50 GB |

> 💡 **Tip:** The 50,000 minutes are a **shared enterprise pool** across all organizations. They are Linux-equivalent minutes -- other OS types consume minutes at a multiplier.

### OS Multipliers

| Runner OS | Multiplier | Effective Minutes from 50K Pool |
|-----------|-----------|-------------------------------|
| **Linux** | 1x | 50,000 minutes |
| **Windows** | 2x | 25,000 minutes |
| **macOS** | 10x | 5,000 minutes |

> ⚠️ **Important:** A 10-minute macOS job consumes 100 minutes from your included pool. Plan macOS usage carefully.

---

## 2️⃣ Overage Rates

When included minutes are exhausted, per-minute charges apply:

| Runner OS | Per-Minute Rate |
|-----------|----------------|
| **Linux** | $0.008/min |
| **Windows** | $0.016/min |
| **macOS** | $0.08/min |

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

## 4️⃣ Setting Spending Limits

Control overage costs by setting a spending limit at the enterprise level.

**Navigation:**

```
Enterprise → Settings → Billing (sidebar)
  → Spending limits → Actions → Set limit
```

| Option | Effect |
|--------|--------|
| **$0 limit** | No overage allowed -- jobs fail when included minutes are exhausted |
| **Custom amount** | Jobs run until the spending cap is reached |
| **Unlimited** | No cap on overage spending |

> ⚠️ **Warning:** Setting the limit to $0 means workflows will fail once included minutes run out. Communicate this to teams before applying.

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
| **Billing reports** | Enterprise → Settings → Billing | Download CSV of Actions usage by org and repo |
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

---

## 📚 Resources

- [About billing for GitHub Actions](https://docs.github.com/en/billing/managing-billing-for-github-actions/about-billing-for-github-actions)
- [About self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners)
- [About Azure private networking for GitHub-hosted runners](https://docs.github.com/en/actions/using-github-hosted-runners/connecting-to-a-private-network/about-azure-private-networking-for-github-hosted-runners)

---

*Last updated: April 2026*
