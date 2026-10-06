# 🤖 GitHub App CI/CD Authentication (No Seat Cost) Runbook

> **Complete guide to using GitHub Apps for CI/CD authentication instead of machine users, saving $21/user/month per seat**

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [📋 Overview](#-overview)
- [1️⃣ Why GitHub Apps Over Machine Users](#1️⃣-why-github-apps-over-machine-users)
- [2️⃣ Create a GitHub App](#2️⃣-create-a-github-app)
- [3️⃣ Configure Permissions and Create the Private Key](#3️⃣-configure-permissions-and-create-the-private-key)
- [4️⃣ Install the App on Your Organization](#4️⃣-install-the-app-on-your-organization)
- [5️⃣ Generate an Installation Access Token in CI/CD](#5️⃣-generate-an-installation-access-token-in-cicd)
- [6️⃣ Alternative: GITHUB_TOKEN for Actions-Only Workflows](#6️⃣-alternative-github_token-for-actions-only-workflows)
- [7️⃣ Machine User Account (Last Resort)](#7️⃣-machine-user-account-last-resort)
- [8️⃣ EMU (Enterprise Managed Users) Considerations](#8️⃣-emu-enterprise-managed-users-considerations)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📚 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Create the app:** `Org → Settings → Developer settings → GitHub Apps → New GitHub App` → name, homepage, untick webhook **Active**, set **Permissions**, **Only on this account** → **Create GitHub App**
- **Private key:** app settings → **Private keys** → **Generate a private key**
- **Install:** app settings → **Install App** → **Install** (next to the org) → **Only select repositories** → **Install**
- **Store credentials:** Client ID as an Actions **variable** (`APP_CLIENT_ID`), private key as a **secret** (`APP_PRIVATE_KEY`)
- **In workflows:** `actions/create-github-app-token@v3` with `client-id` and `private-key`

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
| Register and install a GitHub App owned by the organization | **Organization owner** (or an app manager for that org) | ☐ |
| Store the app's client ID and private key for workflows | **Organization owner** (org-level) or **repository administrator** (repo-level) | ☐ |
| GitHub Actions enabled on the target repositories | Organization owner / repo admin | ☐ |
| The permissions your CI/CD needs (Contents, Pull requests, Checks, and so on) | Pipeline owner | ☐ |

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **GitHub organization owner** | Creates and installs the GitHub App before retiring a machine user or IdP-backed service account. | GitHub → profile picture → Organizations → [organization] → Settings → Developer settings → GitHub Apps → New GitHub App → name, homepage URL, untick webhook Active, set the minimum Permissions, Only on this account → Create GitHub App → Private keys → Generate a private key → Install App → Install → Only select repositories → Install. Handoff: client ID (stored as a variable), private key (stored as a secret), and the repository list. |
| **Microsoft Entra or Okta admin, only if replacing an IdP-backed machine user** | Removes or disables the old service account after the GitHub App workflow is validated. | Entra: Microsoft Entra admin center → Entra ID → Users → [service account] → Applications or Assigned roles → remove GitHub app assignment, then Block sign-in if approved. Okta: Okta Admin Console → Directory → People → [service account] → Applications → remove GitHub assignment, then More Actions → Deactivate if approved. Handoff: decommission ticket and validation that no workflows still use the old account. |
| **Azure deployment owner, only if the GitHub App workflow also deploys to Azure** | Keeps Azure deployment identity separate from GitHub App authentication. | Use the Azure OIDC flow in the Azure deployment guide: Microsoft Entra admin center → App registrations → [app] → Certificates & secrets → Federated credentials → Add credential, then Azure portal → [target scope] → Access control (IAM) → Add role assignment. Handoff: Azure client ID, tenant ID, subscription ID, and role assignment scope. |

---

## 📋 Overview

| Authentication Method | Seat Cost | Security | Permissions | Best For |
|----------------------|-----------|----------|-------------|----------|
| **GitHub App** | None ($0) | Best -- scoped, short-lived tokens | Granular per-permission | CI/CD, automation, bots |
| **GITHUB_TOKEN** | None ($0) | Good -- auto-scoped to workflow | Limited to Actions context | Actions-only workflows |
| **Machine user (PAT)** | $21/month (consumes a seat) | Weakest -- long-lived, broad access | Coarse-grained scopes | Last resort only |

> 💡 **Tip:** GitHub Apps are the recommended approach for any automation that needs to authenticate as a non-human identity. They do not consume a license seat and provide the best security model.

---

## 1️⃣ Why GitHub Apps Over Machine Users

| Factor | GitHub App | Machine User (PAT) |
|--------|------------|---------------------|
| **License cost** | Free (no seat) | $21/user/month |
| **Token lifetime** | Short-lived (1 hour) | Long-lived (up to forever with classic PATs) |
| **Permission model** | Granular (per-API endpoint) | Coarse scopes (e.g., full `repo` access) |
| **Audit trail** | Actions attributed to the app | Actions attributed to a generic user |
| **Rate limits** | Higher (scales with installations) | Standard user rate limits |
| **Rotation** | Automatic (tokens regenerated per run) | Manual rotation required |

---

## 2️⃣ Create a GitHub App

**👤 Role:** **Organization owner** · **📍 Portal:** GitHub

**Navigate:** Organization → **Settings** → **Developer settings** *(sidebar)* → **GitHub Apps** → **New GitHub App**

**Steps:**

1. Fill in the form:

| Field | Value |
|-------|-------|
| **GitHub App name** | A clear name, 34 characters max (for example `my-org-ci-bot`) |
| **Homepage URL** | Your organization or repository URL |
| **Webhook → Active** | **Untick** — CI/CD token generation doesn't need webhooks |
| **Permissions** | For each permission, choose **Read-only**, **Read & write**, or **No access** (see Section 3) |
| **Where can this GitHub App be installed?** | **Only on this account** |

2. Click **Create GitHub App**.

> 💡 **Enterprise-owned apps:** an enterprise owner can also register the app at the enterprise level (Enterprise → **Settings** → **GitHub Apps**) and install it on organizations in the enterprise. With EMU, the install option reads **This enterprise**.

---

## 3️⃣ Configure Permissions and Create the Private Key

### Permissions

Set them while creating the app, or later at app settings → **Permissions & events** (installations must approve added permissions).

| Permission | Access level | Use case |
|------------|-------------|----------|
| **Contents** | Read & write | Clone, push commits, create releases |
| **Pull requests** | Read & write | Create or update PRs, post comments |
| **Checks** | Read & write | Report CI check runs |
| **Issues** | Read & write | Create or update issues |
| **Actions** | Read-only | Read workflow run status |
| **Packages** | Read & write | Publish or consume GitHub Packages |
| **Metadata** | Read-only | Always granted automatically |

> ⚠️ **Warning:** Grant only what your workflows need. Every extra permission widens what a leaked token can do.

### Generate and store the credentials

1. On the app's settings page, copy the **Client ID** (it's different from the App ID).
2. Under **Private keys**, click **Generate a private key**. A `.pem` file downloads.
3. Store them for Actions — Organization (or Repository) → **Settings** → **Secrets and variables** → **Actions**:
   - **Variables** tab → **New organization variable** → name `APP_CLIENT_ID`, value = client ID → choose **Repository access** → **Add variable**.
   - **Secrets** tab → **New organization secret** → name `APP_PRIVATE_KEY`, value = the **entire** `.pem` contents (including the `BEGIN`/`END` lines) → choose **Repository access** → **Add secret**.
4. Delete the downloaded `.pem` file from your machine.

---

## 4️⃣ Install the App on Your Organization

**👤 Role:** **Organization owner** · **📍 Portal:** GitHub

**Navigate:** Organization → **Settings** → **Developer settings** → **GitHub Apps** → **Edit** next to your app → **Install App**

**Steps:**

1. Click **Install** next to your organization.
2. Choose the scope:

| Option | Effect |
|--------|--------|
| **All repositories** | Tokens can reach any repository in the organization |
| **Only select repositories** | Tokens are limited to the repositories you pick in **Select repositories** |

3. Click **Install**.

> 💡 **Tip:** Start with **Only select repositories** and add repositories as needed.

---

## 5️⃣ Generate an Installation Access Token in CI/CD

### Using the `actions/create-github-app-token` action

```yaml
name: CI Pipeline

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Generate app token
        id: app-token
        uses: actions/create-github-app-token@v3
        with:
          client-id: ${{ vars.APP_CLIENT_ID }}
          private-key: ${{ secrets.APP_PRIVATE_KEY }}

      - name: Checkout with app token
        uses: actions/checkout@v7
        with:
          token: ${{ steps.app-token.outputs.token }}

      - name: Use the token with the GitHub CLI
        env:
          GH_TOKEN: ${{ steps.app-token.outputs.token }}
        run: gh pr list
```

### Scoping the token to specific repositories

```yaml
      - name: Generate app token (scoped)
        id: app-token
        uses: actions/create-github-app-token@v3
        with:
          client-id: ${{ vars.APP_CLIENT_ID }}
          private-key: ${{ secrets.APP_PRIVATE_KEY }}
          owner: ${{ github.repository_owner }}
          repositories: "repo-a,repo-b"
```

> ✅ **Result:** a short-lived installation access token (valid for one hour) for the later steps. No license seat is consumed.

> 📌 **Token format (October 2026):** installation tokens still start with `ghs_` but are now **stateless and about 520 characters long** (previously 40). Treat them as opaque strings — check any length validation, fixed-size secret fields, proxies that truncate `Authorization` headers, and log-redaction patterns.

---

## 6️⃣ Alternative: GITHUB_TOKEN for Actions-Only Workflows

*If your workflow only needs to operate within the current repository during an Actions run, `GITHUB_TOKEN` requires zero setup.*

### Usage

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      checks: write

    steps:
      - name: Checkout
        uses: actions/checkout@v7

      - name: Run tests and report
        run: ./run-tests.sh
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### When GITHUB_TOKEN Is Sufficient

| Scenario | GITHUB_TOKEN Works? |
|----------|-------------------|
| Clone the current repo | ✅ Yes |
| Post status checks on the current repo | ✅ Yes |
| Create releases on the current repo | ✅ Yes |
| Clone or push to a different repo | ❌ No -- use GitHub App |
| Trigger workflows in other repos | ❌ No -- use GitHub App |
| Access organization-level APIs | ❌ No -- use GitHub App |

> 💡 **Tip:** `GITHUB_TOKEN` is automatically provided to every Actions workflow. No secrets configuration is needed.

---

## 7️⃣ Machine User Account (Last Resort)

*Only use a machine user when GitHub Apps and GITHUB_TOKEN cannot meet the requirement.*

### When a Machine User May Be Needed

| Scenario | Reason |
|----------|--------|
| Third-party tool requires a PAT and cannot use App tokens | Tool limitation |
| Self-hosted runner needs persistent git credentials | Non-Actions CI system |
| Legacy system integration | Cannot be updated to support App auth |

### Setup

1. Create a dedicated GitHub user account (e.g., `my-org-bot`)
2. Add the user to the organization
3. Generate a personal access token (fine-grained preferred)
4. Store the PAT as a secret in GitHub Actions

> ⚠️ **Warning:** Machine users consume a license seat ($21/user/month on GHEC). The PAT is long-lived and must be manually rotated. Always prefer GitHub Apps.

---

## 8️⃣ EMU (Enterprise Managed Users) Considerations

| Factor | GitHub App | Machine User on EMU |
|--------|------------|---------------------|
| **Seat cost** | None | $21/month |
| **Provisioning** | Install via org settings | Must be SCIM-provisioned through IdP |
| **Token type** | Installation tokens (1 hour) | Personal access tokens — enterprise PAT policies apply |
| **IdP requirement** | None | Must exist in Entra ID / Okta and be assigned to the GitHub app in IdP |
| **Deprovisioning** | Uninstall from org | Must be deprovisioned via SCIM |

> 💡 **Tip:** On EMU, machine users cannot be created directly in GitHub. They must be provisioned through your identity provider via SCIM. GitHub Apps bypass this complexity entirely and are strongly preferred.

### EMU Machine User Provisioning (If Required)

**Steps:**

1. Create a service account in your IdP (Entra ID or Okta)
2. Assign it to the GitHub EMU SCIM application
3. Wait for SCIM provisioning to create the GitHub user
4. Sign in as the account and create a fine-grained personal access token with only the access it needs
5. Store the PAT as a GitHub Actions secret

> ⚠️ **Warning:** If the IdP service account is disabled or deprovisioned, the GitHub machine user and all its PATs are immediately invalidated. Plan for IdP service account lifecycle management.

## 🧯 Known Errors & Resolutions

<details>
<summary><em>Show known errors table</em></summary>


> This section lists the known product errors and admin-facing symptoms that commonly occur with this workflow. Exact message text can vary by product rollout, tenant policy, and provider, so use the log or settings page named in the resolution to confirm the root cause.

| Error or symptom | Likely cause | Resolution |
|------------------|--------------|------------|
| **Page, tab, or button is missing** | Wrong account context, missing admin role, unavailable plan/add-on, or feature rollout not enabled for the selected enterprise/org/repo. | Switch to the correct account and scope, confirm the prerequisite role, verify licensing or add-on activation, then refresh the page. If the control is still absent, use the direct settings URL from the relevant GitHub Docs page and confirm the feature is available for your plan. |
| **Changes appear saved but behavior does not change** | Policy inheritance, cached UI state, propagation delay, or an overlapping enterprise/org/repo policy. | Reopen the settings page, verify the effective policy at the lowest affected scope, wait for propagation where documented, and check for a stricter policy at an enterprise or organization level. |
| **403, forbidden, or resource not accessible** | The signed-in user or token can see the page but lacks the specific permission for the action. | Use an enterprise owner, organization owner, repository admin, or token with the exact scopes/permissions listed in the runbook. For SAML-protected orgs, authorize the token or SSH key for SSO before retrying. |
| **Workflow is queued, blocked, or canceled for billing** | Included minutes are exhausted, no payment method is available, hard-stop budget is reached, or larger runners require paid billing. | Check Billing and licensing > Usage and Budgets and alerts, add or verify payment, adjust budgets, or move appropriate workloads to self-hosted runners. |
| **Runner label is not found or job never starts** | The workflow references a label that no online runner has, or the runner group is not available to the repository. | Confirm the exact labels in Actions > Runners, put the runner in an accessible group, and update `runs-on` to match. |
| **GitHub App token returns 403 or 404** | The app is not installed on the repository or lacks the specific repository permission. | Install the app on the target repo, grant the narrow required permissions, regenerate the installation token, and retry. |
| **OIDC token is unavailable** | The workflow lacks `permissions: id-token: write` or is running from an event where the job cannot request a token. | Add the id-token permission at workflow or job scope and test with the OIDC debugger before creating cloud trust conditions. |
| **Azure federated credential rejects the token** | Issuer, audience, subject, branch, environment, or GHE.com token issuer does not match the credential. | Compare the live token claims to the federated credential and update the Azure issuer/subject/audience exactly, including GHE.com issuer differences. |

</details>

---

## ❓ Common Questions & Troubleshooting

<details>
<summary><em>Show Q&A</em></summary>


### Q: My workflow fails with "Resource not accessible by integration" when using a GitHub App token. What is wrong?
**A:** The app either lacks the permission the API call needs, or isn't installed on that repository. Check (1) app settings → **Permissions & events** (for example **Contents: Read & write** to push), and that the organization approved any newly added permissions; and (2) the installation's repository list (Org → **Settings** → under "Third-party Access", **GitHub Apps** → **Configure** next to the app). Also check whether you narrowed the token with `repositories`.

---

### Q: Token generation is failing in my workflow with the `actions/create-github-app-token` action. What should I check?
**A:** Check that (1) `APP_CLIENT_ID` holds the app's **Client ID** (not the numeric App ID) and is available to the repository, (2) `APP_PRIVATE_KEY` holds the full `.pem` contents including the `BEGIN` and `END` lines, (3) the key hasn't been deleted in the app settings, and (4) the app is installed on the repository owner you're targeting (set `owner` when generating a token for a different organization).

---

### Q: When should I use a GitHub App vs a Personal Access Token (PAT)?
**A:** Use a GitHub App for organization-wide CI/CD automation -- it provides scoped, short-lived tokens with no seat cost and better audit trails. Use a PAT only for quick personal scripts or when a third-party tool specifically requires a PAT and cannot accept App tokens. GitHub Apps are strongly recommended for any production automation.

---

### Q: I created a GitHub App but it does not appear in my organization. Where is it?
**A:** If you registered it under your personal account (profile **Settings** → **Developer settings** → **GitHub Apps**), it's owned by you, not the organization. Register org apps at Organization → **Settings** → **Developer settings** → **GitHub Apps** → **New GitHub App**, or transfer the app's ownership to the organization from its advanced settings. Then install it from **Install App**.

---

### Q: How are GitHub App installation tokens scoped? Can a token access any repo in the org?
**A:** Installation tokens are scoped to the repositories where the App is installed. If the App is installed on "All repositories," the token can access any repo in the org. If installed on "Only select repositories," the token is limited to those specific repos. You can further narrow the scope at runtime by passing the `repositories` parameter to the `actions/create-github-app-token` action.

---

### Q: How long do GitHub App installation tokens last, and do I need to handle rotation?
**A:** Installation tokens expire after one hour, and the action creates a fresh one on each run — no token rotation needed. Do rotate the app's **private key** periodically: generate a new key, update `APP_PRIVATE_KEY`, then delete the old key in the app settings.

</details>

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Minutes Governance & Runner Strategy | `Actions/Minutes Governance & Runner Strategy.md` |
| OIDC Federation for Azure Deployments | `Actions/OIDC Federation for Azure Deployments.md` |
| Copilot Seat Assignment & Enablement | `Copilot/Seat Assignment & Enablement.md` |

---

## 📚 Resources

- [Registering a GitHub App](https://docs.github.com/en/apps/creating-github-apps/registering-a-github-app/registering-a-github-app)
- [Making authenticated API requests with a GitHub App in a GitHub Actions workflow](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/making-authenticated-api-requests-with-a-github-app-in-a-github-actions-workflow)
- [Installing your own GitHub App](https://docs.github.com/en/apps/using-github-apps/installing-your-own-github-app)
- [actions/create-github-app-token](https://github.com/actions/create-github-app-token)
- [Automatic token authentication (GITHUB_TOKEN)](https://docs.github.com/en/actions/concepts/security/github_token)

---

*Last updated: October 2026*
