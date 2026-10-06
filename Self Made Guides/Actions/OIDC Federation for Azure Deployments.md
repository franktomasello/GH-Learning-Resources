# 🔗 GitHub Actions OIDC Federation for Azure Deployments Runbook

> **Complete guide to configuring passwordless deployments from GitHub Actions to Azure using OpenID Connect (OIDC) federation**

---

## 📑 Contents

- [⚡ Quick-Start Summary](#-quick-start-summary)
- [✅ Accuracy & Click-Path Notes](#-accuracy--click-path-notes)
- [✅ Prerequisites](#-prerequisites)
- [👥 Provider Account Action Matrix](#-provider-account-action-matrix)
- [📋 Overview](#-overview)
- [1️⃣ Create an App Registration in Entra ID](#1️⃣-create-an-app-registration-in-entra-id)
- [2️⃣ Assign Azure RBAC Roles for Deployment Targets](#2️⃣-assign-azure-rbac-roles-for-deployment-targets)
- [3️⃣ Add a Federated Credential to the App Registration](#3️⃣-add-a-federated-credential-to-the-app-registration)
- [4️⃣ Issuer and Subject: What Azure Must Match](#4️⃣-issuer-and-subject-what-azure-must-match)
- [5️⃣ Scope Access by Org, Repo, Branch, or Environment](#5️⃣-scope-access-by-org-repo-branch-or-environment)
- [6️⃣ Configure the GitHub Actions Workflow](#6️⃣-configure-the-github-actions-workflow)
- [7️⃣ GHE.com (Data Residency) Configuration](#7️⃣-ghecom-data-residency-configuration)
- [8️⃣ Benefits Summary](#8️⃣-benefits-summary)
- [9️⃣ Updating OIDC Trust When Migrating from GitHub.com to GHE.com](#9️⃣-updating-oidc-trust-when-migrating-from-githubcom-to-ghecom)
- [🧯 Known Errors & Resolutions](#-known-errors--resolutions)
- [❓ Common Questions & Troubleshooting](#-common-questions--troubleshooting)
- [🔗 Related Guides](#-related-guides)
- [📚 Resources](#-resources)

---


## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **App registration:** `Microsoft Entra admin center → Entra ID → App registrations → New registration` → name → **Accounts in this organizational directory only** → **Register**
- **RBAC role:** `Azure portal → [subscription / resource group] → Access control (IAM) → Add → Add role assignment` → role → **Members** → **Select members** → app → **Review + assign**
- **Federated credential:** `App registration → Certificates & secrets → Federated credentials → Add credential` → **GitHub Actions deploying Azure resources** (or **Other issuer**) → **Add**
- **Workflow:** `permissions: { id-token: write, contents: read }` + `azure/login` with client, tenant, and subscription IDs
- **Check your subject format:** repos created, renamed, or transferred after **July 15, 2026** use `repo:OWNER@OWNER-ID/REPO@REPO-ID:...`

---

## ✅ Accuracy & Click-Path Notes

<details>
<summary><em>Show click-path conventions</em></summary>


- Reviewed against current public GitHub and Microsoft documentation in October 2026 where public documentation is available. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
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
| Create the app registration and add federated credentials | **Application Administrator**, **Cloud Application Administrator**, or the app's **owner** (users can register apps only if the tenant allows it) | ☐ |
| Assign Azure roles to the service principal | **Owner**, **User Access Administrator**, or **Role Based Access Control Administrator** at the target scope | ☐ |
| A GitHub repository with Actions enabled | — | ☐ |
| Create Actions secrets or variables | **Repository administrator** (repo-level) or **organization owner** (org-level) | ☐ |
| Know your subject format (legacy name-based or immutable ID-based) | Repository administrator — see Section 4 | ☐ |

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **Microsoft Entra app owner, Application Administrator, Cloud Application Administrator, or Global Administrator** | Creates the app registration or updates the existing deployment identity, then adds the GitHub Actions federated credential. | Microsoft Entra admin center → Entra ID → App registrations → New registration → Name → Accounts in this organizational directory only → Register → Overview → copy Application (client) ID and Directory (tenant) ID → Certificates & secrets → Federated credentials → Add credential → GitHub Actions deploying Azure resources (or Other issuer for GHE.com or immutable subjects) → enter Organization, Repository, Entity type, value, and Name (Audience stays api://AzureADTokenExchange) → Add. Handoff: client ID, tenant ID, credential name, issuer, subject, and audience. |
| **Azure subscription, resource group, or resource Owner/RBAC administrator** | Grants the service principal only the Azure role needed by the workflow. | Azure portal → Subscriptions, Resource groups, or the target resource → [scope] → Access control (IAM) → Add → Add role assignment → select the least-privilege role → Next → Members → User, group, or service principal → Select members → [app/service principal] → Select → Review + assign. Handoff: Azure subscription ID, scope, role, and service principal name. |
| **GitHub repository or organization admin** | Stores Azure identifiers and ensures the workflow can request an OIDC token. | GitHub → [owner/repository] → Settings → Secrets and variables → Actions → Secrets → New repository secret → add AZURE_CLIENT_ID, AZURE_TENANT_ID, and AZURE_SUBSCRIPTION_ID (one at a time, clicking Add secret each time). In the workflow file, set `permissions: id-token: write` and `contents: read`, then use `azure/login` with those secrets. Handoff: successful workflow run and `az account show` output. |

---

## 📋 Overview

OIDC federation eliminates the need for long-lived Azure credentials stored as GitHub secrets. Instead, GitHub Actions requests a short-lived token from Azure at runtime.

| Benefit | Description |
|---------|-------------|
| **No secret rotation** | No client secrets or certificates to manage or rotate |
| **Reduced attack surface** | Short-lived tokens expire after the workflow run |
| **Scoped access** | Trust can be limited to a specific org, repo, branch, or environment |
| **Audit trail** | Azure logs show which GitHub workflow requested each token |

---

## 1️⃣ Create an App Registration in Entra ID

**👤 Role:** **Application Administrator** / **Cloud Application Administrator** (or a user allowed to register apps) · **📍 Portal:** Microsoft Entra admin center

**Navigate:** Entra ID → **App registrations** → **New registration**

**Steps:**

1. Enter a **Name** (for example `github-actions-deployer`).
2. Under **Supported account types**, select **Accounts in this organizational directory only**.
3. Leave **Redirect URI** blank.
4. Click **Register**.
5. On the **Overview** page, copy these values:

| Value | Where to find it |
|-------|------------------|
| **Application (client) ID** | App registration → **Overview** |
| **Directory (tenant) ID** | App registration → **Overview** |
| **Subscription ID** | Azure portal → **Subscriptions** |

> 💡 Registering the app also creates its **service principal** (enterprise application), which is what you assign Azure roles to.

---

## 2️⃣ Assign Azure RBAC Roles for Deployment Targets

**👤 Role:** **Owner**, **User Access Administrator**, or **Role Based Access Control Administrator** at the scope · **📍 Portal:** Azure portal

**Navigate:** Azure portal → **Subscriptions** (or **Resource groups** / the resource) → *[scope]* → **Access control (IAM)** → **Add** → **Add role assignment**

**Steps:**

1. On the **Role** tab, select the least-privileged role (see the table), then click **Next**.
2. On the **Members** tab, set **Assign access to** to **User, group, or service principal**.
3. Click **Select members**, search for the app registration name, select it, and click **Select**.
4. Click **Review + assign**, then **Review + assign** again.

> ⚠️ **Warning:** Avoid **Owner** or **User Access Administrator** unless the pipeline truly manages access. Prefer resource-group scope over subscription scope.

| Common deployment scenario | Suggested role |
|---------------------------|------------------|
| Deploy to App Service / Functions | **Website Contributor** |
| Deploy to AKS | **Azure Kubernetes Service Contributor Role** |
| Deploy infrastructure (Terraform/Bicep) | **Contributor** (scoped to the resource group) |
| Read-only checks | **Reader** |

---

## 3️⃣ Add a Federated Credential to the App Registration

**👤 Role:** App **owner**, **Application Administrator**, or **Cloud Application Administrator** · **📍 Portal:** Microsoft Entra admin center

**Navigate:** Entra ID → **App registrations** → *[your app]* → **Certificates & secrets** → **Federated credentials** tab → **Add credential**

**Steps:**

1. Under **Federated credential scenario**, select **GitHub Actions deploying Azure resources**.
2. Fill in:

| Field | Value |
|-------|-------|
| **Organization** | Your GitHub organization name |
| **Repository** | Your repository name |
| **Entity type** | **Environment**, **Branch**, **Pull request**, or **Tag** |
| **Based on selection** | The value — for example `production` (environment) or `main` (branch) |
| **Name** | A descriptive name, for example `gh-my-repo-production` |
| **Audience** | Leave the default `api://AzureADTokenExchange` |

3. Check the **Subject identifier** preview matches what your workflow will send (Section 4).
4. Click **Add**.

> ⚠️ **Immutable subjects:** if your repository uses the new ID-based subject (Section 4), choose **Other issuer** instead and paste the exact issuer and subject — the GitHub scenario builds a name-only subject.

---

## 4️⃣ Issuer and Subject: What Azure Must Match

Azure compares the token's **issuer**, **subject**, and **audience** to the federated credential — all three must match **exactly**.

### Issuer

| GitHub platform | Issuer URL |
|----------------|------------|
| **GitHub.com** | `https://token.actions.githubusercontent.com` |
| **GHE.com (data residency)** | `https://token.actions.SUBDOMAIN.ghe.com` |
| **Custom enterprise issuer** (if enabled with `include_enterprise_slug`) | `https://token.actions.githubusercontent.com/ENTERPRISE-SLUG` |

### Subject — two formats

| Format | When it applies | Example |
|--------|-----------------|---------|
| **Legacy (name-based)** | Repositories created before July 15, 2026 that haven't opted in | `repo:octo-org/octo-repo:ref:refs/heads/main` |
| **Immutable (ID-based)** | Repositories created, renamed, or transferred after July 15, 2026, or that opted in | `repo:octo-org@123456/octo-repo@456789:ref:refs/heads/main` |

> 💡 **Find the IDs:** `gh api repos/OWNER/REPO --jq '.owner.id, .id'`. The fastest way to see the exact subject is the Azure error message, which quotes the subject it received.

| Entity type | Subject (legacy format) |
|-------------|-------------------------|
| **Branch** | `repo:ORG/REPO:ref:refs/heads/BRANCH` |
| **Environment** | `repo:ORG/REPO:environment:ENV_NAME` |
| **Tag** | `repo:ORG/REPO:ref:refs/tags/TAG` |
| **Pull request** | `repo:ORG/REPO:pull_request` |

> ⚠️ **Environment wins:** if the job uses `environment:`, the subject is the **environment** form — a branch-based credential won't match.

### Setting the issuer manually (GHE.com or immutable subjects)

1. On **Add credential**, select **Other issuer**.
2. **Issuer:** the URL from the table above.
3. **Subject identifier:** the exact subject string.
4. **Name:** a descriptive name; leave **Audience** as `api://AzureADTokenExchange`.
5. Click **Add**.

---

## 5️⃣ Scope Access by Org, Repo, Branch, or Environment

Add one federated credential per trust you need (an app registration can hold up to 20):

| Scope | Subject example (legacy) | Use case |
|-------|--------------------------|----------|
| **Environment** | `repo:MyOrg/my-repo:environment:production` | Only jobs using the `production` environment can deploy |
| **Specific branch** | `repo:MyOrg/my-repo:ref:refs/heads/main` | Only `main` can deploy |
| **Pull requests** | `repo:MyOrg/my-repo:pull_request` | PR validation |
| **Tag** | `repo:MyOrg/my-repo:ref:refs/tags/v1.0.0` | One release tag |

> ⚠️ **No wildcards:** standard federated credentials need an exact subject match, so `refs/heads/*` won't work. Create one credential per branch or environment — or look at Entra's flexible federated identity credentials if you need pattern matching.

> 💡 **Tip:** Use **environment-based** credentials for production and add environment protection rules (required reviewers, deployment branches) in GitHub.

---

## 6️⃣ Configure the GitHub Actions Workflow

### Required Permissions

```yaml
permissions:
  id-token: write   # Required for OIDC token request
  contents: read     # Required for actions/checkout
```

### Full workflow example

```yaml
name: Deploy to Azure

on:
  push:
    branches: [main]

permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production   # Subject becomes ...:environment:production

    steps:
      - name: Checkout code
        uses: actions/checkout@v6

      - name: Log in to Azure
        uses: azure/login@v2   # Consider pinning to a full commit SHA
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: Deploy to Azure Web App
        uses: azure/webapps-deploy@v3
        with:
          app-name: my-web-app
          package: .
```

### Store the IDs in GitHub

**👤 Role:** **Repository administrator** · **📍 Portal:** GitHub

**Navigate:** Repository → **Settings** → **Secrets and variables** → **Actions**

1. On the **Secrets** tab, click **New repository secret**.
2. Enter the **Name** and **Secret**, then click **Add secret**. Repeat for each row:

| Secret name | Value |
|-------------|-------|
| `AZURE_CLIENT_ID` | Application (client) ID |
| `AZURE_TENANT_ID` | Directory (tenant) ID |
| `AZURE_SUBSCRIPTION_ID` | Subscription ID |

> 💡 **Tip:** These are identifiers, not credentials, so they can also be Actions **variables** (`vars.AZURE_CLIENT_ID`). If the workflow uses an environment, you can store them as environment secrets.

---

## 7️⃣ GHE.com (Data Residency) Configuration

| Setting | GitHub.com | GHE.com |
|---------|------------|---------|
| **OIDC issuer** | `https://token.actions.githubusercontent.com` | `https://token.actions.SUBDOMAIN.ghe.com` |
| **OIDC discovery** | `https://token.actions.githubusercontent.com/.well-known/openid-configuration` | `https://token.actions.SUBDOMAIN.ghe.com/.well-known/openid-configuration` |
| **Audience** | `api://AzureADTokenExchange` | `api://AzureADTokenExchange` |

### Workflow adjustment for GHE.com

No workflow YAML changes are needed — the runner requests the token from your GHE.com issuer automatically. In Azure, create the federated credential with **Other issuer** and the GHE.com issuer URL.

---

## 8️⃣ Benefits Summary

| Traditional Approach (Client Secret) | OIDC Federation |
|--------------------------------------|-----------------|
| Long-lived secret stored in GitHub | No stored secrets |
| Must rotate every 1-2 years | No rotation needed |
| Secret leak = persistent access | Token expires in minutes |
| Broad access (any workflow) | Scoped to branch/environment |
| Manual secret management | Zero credential maintenance |

---

## 9️⃣ Updating OIDC Trust When Migrating from GitHub.com to GHE.com

*Moving repositories from GitHub.com to GHE.com changes the issuer — and usually the subject.*

### Steps

1. **Add a new federated credential** (Other issuer) on the existing app registration:

| Field | New value |
|-------|-----------|
| **Issuer** | `https://token.actions.SUBDOMAIN.ghe.com` |
| **Subject identifier** | The subject the migrated repository sends — migrated repositories are new repositories, so expect the **immutable** form, for example `repo:NEW_ORG@OWNER-ID/REPO@REPO-ID:environment:production` |

2. **Run a deployment** from the GHE.com repository. If it fails, copy the subject from the Azure error and fix the credential.
3. **Remove the old credential** that used `https://token.actions.githubusercontent.com` once the new one works.

> 💡 **Tip:** keep both credentials during the transition.

> ⚠️ **Warning:** if the organization or repository name changed, the subject must use the new names (and IDs).

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


### Q: I am getting "AADSTS700016: Application not found" when my workflow tries to log in to Azure. What is wrong?
**A:** This error means Azure cannot find the App Registration. Verify that the `client-id` (Application ID) and `tenant-id` (Directory ID) stored in your GitHub secrets are correct and correspond to an active App Registration in the correct Entra ID tenant. Copy-paste errors and extra whitespace in secret values are common causes.

---

### Q: My workflow fails with "No matching federated identity record found." How do I fix this?
**A:** The token's subject (or issuer) doesn't exactly match any federated credential. The Azure error quotes the subject it received — compare it to your credentials. Common causes: the job uses `environment:` but the credential is branch-based (or the reverse), a typo or case difference, a renamed organization or repository, or the repository uses the **immutable** subject (`repo:ORG@ID/REPO@ID:...`) while the credential has the name-only form. Add a credential with the exact subject.

---

### Q: OIDC was working on GitHub.com but stopped after we migrated to GHE.com. What changed?
**A:** Two things changed: the **issuer** is now `https://token.actions.SUBDOMAIN.ghe.com`, and the **subject** probably changed too — migrated repositories are new repositories, so they use the immutable ID-based subject, and the organization name may differ. Add a credential with **Other issuer**, the GHE.com issuer, and the exact subject from the Azure error, then remove the old one after testing.

---

### Q: Azure login succeeds but I get "Permission denied" when deploying to a resource. What should I check?
**A:** The App Registration needs the correct Azure RBAC role assignment on the target resource (subscription, resource group, or individual resource). Verify the role assignment in Azure Portal under Access Control (IAM) for the specific resource. Common mistake: assigning the role at the subscription level when the deployment targets a different subscription, or using a role with insufficient permissions (e.g., Reader instead of Contributor).

---

### Q: My deployment takes longer than 1 hour and the OIDC token expires mid-job. How do I handle this?
**A:** The Azure access token that `azure/login` obtains is short-lived (typically about an hour). For long deployments, split the work into steps or jobs that each finish in time, or run `azure/login` again before the later steps — each run of the action requests a fresh GitHub OIDC token and exchanges it.

---

### Q: Do I need to store any Azure credentials as GitHub secrets with OIDC?
**A:** You store the Application (client) ID, Directory (tenant) ID, and Subscription ID as GitHub secrets -- but these are identifiers, not credentials. No client secrets, certificates, or passwords are needed. The OIDC exchange generates a short-lived token at runtime without any stored credential, which is the primary security advantage of this approach.

</details>

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| GitHub Enterprise Importer (GEI) & Actions Importer | `Migration/GitHub Enterprise Importer (GEI) & Actions Importer.md` |
| Data Residency Decision Guide (DRUS vs Standard vs GHES) | `Setup/Data Residency Decision Guide (DRUS vs Standard vs GHES).md` |
| GitHub App for CI/CD (No Seat Cost) | `Actions/GitHub App for CI-CD (No Seat Cost).md` |

---

## 📚 Resources

- [Configuring OpenID Connect in Azure](https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-azure)
- [OpenID Connect reference (claims, immutable subjects, GHE.com)](https://docs.github.com/en/actions/reference/security/oidc)
- [About OpenID Connect](https://docs.github.com/en/actions/concepts/security/openid-connect)
- [Microsoft: Connect GitHub and Azure (OIDC)](https://learn.microsoft.com/en-us/azure/developer/github/connect-from-azure-openid-connect)

---

*Last updated: October 2026*
