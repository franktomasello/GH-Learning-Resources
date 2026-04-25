# 🔗 GitHub Actions OIDC Federation for Azure Deployments Runbook

> **Complete guide to configuring passwordless deployments from GitHub Actions to Azure using OpenID Connect (OIDC) federation**

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Create App Registration:** `Azure Portal → Entra ID → App registrations → + New registration`
- **Assign RBAC role:** `Azure Portal → Subscriptions → [Sub] → Access control (IAM) → + Add role assignment`
- **Add federated credential:** `Azure Portal → Entra ID → App registrations → [App] → Certificates & secrets → Federated credentials → + Add credential`
- **Workflow permissions:** Add `permissions: { id-token: write, contents: read }` to workflow YAML
- **Store IDs as secrets:** `Repo → Settings → Secrets and variables → Actions → New repository secret` (AZURE_CLIENT_ID, AZURE_TENANT_ID, AZURE_SUBSCRIPTION_ID)

---

## ✅ Accuracy & Click-Path Notes

- Reviewed against current public GitHub and Microsoft documentation in April 2026 where public documentation is available. Product UI labels can vary by role, license, feature rollout, and whether the account is on GitHub.com or GHE.com.
- When a path starts with `Enterprise`, begin at GitHub, click your profile photo, click `Your enterprises` or `Enterprise`, select the enterprise, then continue with the listed top tab or left-sidebar item.
- When a path starts with `Organization` or `Org`, begin at GitHub, click your profile photo, click `Your organizations`, select the organization, click `Settings`, then continue with the listed sidebar item.
- When a path starts with `Repository`, `Repo`, or a repository name, open the repository, click the `Settings` tab, then continue with the listed sidebar item.
- When a path starts with a vendor portal such as `Microsoft Entra admin center`, `Azure portal`, `Okta Admin Console`, `PingFederate`, `PingOne`, `OneLogin`, `AD FS Management`, `Visual Studio Admin Portal`, or `Azure DevOps`, sign in to that admin portal first, select the tenant, application, or project named in the step, then follow each listed blade, tab, button, and confirmation in order.
- If the expected button is missing, verify you are signed in with the role named in Prerequisites, the feature or license is enabled, and the object is owned by the selected enterprise, organization, or repository. Use page search only to locate the same page, not to skip required confirmation, test, save, or consent clicks.

---

## ✅ Prerequisites

| Requirement | Status |
|-------------|--------|
| Azure subscription with Entra ID (Azure AD) access | ☐ |
| Permissions to create App Registrations in Entra ID | ☐ |
| Permissions to assign RBAC roles on Azure resources | ☐ |
| GitHub repo with Actions enabled | ☐ |
| Admin access to create GitHub Actions secrets | ☐ |

---

## 👥 Provider Account Action Matrix

Use this table to assign provider-side work before following the numbered steps. If one person holds multiple roles, complete each portal row in order and capture the handoff artifact before moving to the next step.

| Account / role | What they must do | Full click path and handoff |
|---|---|---|
| **Microsoft Entra app owner, Application Administrator, Cloud Application Administrator, or Global Administrator** | Creates the app registration or updates the existing deployment identity, then adds the GitHub Actions federated credential. | Microsoft Entra admin center → Entra ID → App registrations → New registration → Register → Overview → copy Application (client) ID and Directory (tenant) ID → Certificates & secrets → Federated credentials → Add credential → GitHub Actions deploying Azure resources → enter Organization, Repository, Entity type, subject details, Name, and Audience → Add. Handoff: client ID, tenant ID, credential name, issuer, subject, and audience. |
| **Azure subscription, resource group, or resource Owner/RBAC administrator** | Grants the service principal only the Azure role needed by the workflow. | Azure portal → Subscriptions, Resource groups, or the target resource → [scope] → Access control (IAM) → Add → Add role assignment → select the least-privilege role → Next → Members → User, group, or service principal → Select members → [app/service principal] → Select → Review + assign. Handoff: Azure subscription ID, scope, role, and service principal name. |
| **GitHub repository or organization admin** | Stores Azure identifiers and ensures the workflow can request an OIDC token. | GitHub → [owner/repository] → Settings → Secrets and variables → Actions → Secrets → New repository secret → add AZURE_CLIENT_ID, AZURE_TENANT_ID, and AZURE_SUBSCRIPTION_ID. In the workflow file, set `permissions: id-token: write` and `contents: read`, then use `azure/login` with those secrets. Handoff: successful workflow run and `az account show` output. |

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

**Navigation:**

```
Azure Portal → Microsoft Entra ID → App registrations → + New registration
```

**Steps:**

1. Enter a **Name** (e.g., `github-actions-deployer`)
2. Under **Supported account types**, select **Accounts in this organizational directory only**
3. Leave **Redirect URI** blank
4. Click **Register**

> 💡 **Tip:** Record these values immediately after creation -- you will need them for the GitHub Actions workflow:

| Value | Where to Find |
|-------|---------------|
| **Application (client) ID** | App registration → Overview |
| **Directory (tenant) ID** | App registration → Overview |
| **Subscription ID** | Azure Portal → Subscriptions |

---

## 2️⃣ Assign Azure RBAC Roles for Deployment Targets

**Navigation:**

```
Azure Portal → Subscriptions → [Your Subscription] → Access control (IAM)
  → + Add → Add role assignment
```

**Steps:**

1. Select the appropriate role (e.g., **Contributor** for general deployments, or a custom role for least privilege)
2. Under **Assign access to**, select **User, group, or service principal**
3. Search for the App Registration name created in Step 1
4. Click **Review + assign**

> ⚠️ **Warning:** Avoid granting **Owner** or **User Access Administrator** roles unless absolutely required. Use the least privileged role that supports your deployment needs.

| Common Deployment Scenario | Recommended Role |
|---------------------------|------------------|
| Deploy to App Service / Functions | **Website Contributor** |
| Deploy to AKS | **Azure Kubernetes Service Contributor** |
| Deploy infrastructure (Terraform/Bicep) | **Contributor** (scoped to resource group) |
| Read-only access | **Reader** |

---

## 3️⃣ Add Federated Credential to the App Registration

**Navigation:**

```
Azure Portal → Microsoft Entra ID → App registrations → [Your App]
  → Certificates & secrets → Federated credentials → + Add credential
```

**Steps:**

1. Under **Federated credential scenario**, select **GitHub Actions deploying Azure resources**
2. Fill in the fields:

| Field | Value |
|-------|-------|
| **Organization** | Your GitHub org name |
| **Repository** | Your repo name |
| **Entity type** | Branch, Environment, Tag, or Pull Request |
| **GitHub entity name** | e.g., `main` for branch, `production` for environment |
| **Name** | Descriptive name (e.g., `github-actions-main-branch`) |

3. Click **Add**

---

## 4️⃣ Federated Credential Issuer (GitHub.com vs GHE.com)

The issuer URL tells Azure which GitHub OIDC provider to trust.

| GitHub Platform | Issuer URL |
|----------------|------------|
| **GitHub.com** | `https://token.actions.githubusercontent.com` |
| **GHE.com (DRUS)** | `https://token.actions.SUBDOMAIN.ghe.com` |

> ⚠️ **Warning:** If you are on GHE.com, you MUST use the GHE.com-specific issuer URL. The default GitHub.com issuer will not work.

### Setting the Issuer Manually (for GHE.com)

**Navigation:**

```
Azure Portal → Microsoft Entra ID → App registrations → [Your App]
  → Certificates & secrets → Federated credentials → + Add credential
    → Select "Other issuer"
```

**Steps:**

1. Set **Issuer** to `https://token.actions.SUBDOMAIN.ghe.com`
2. Set **Subject identifier** using the format:

| Entity Type | Subject Identifier Format |
|-------------|--------------------------|
| **Branch** | `repo:ORG/REPO:ref:refs/heads/BRANCH` |
| **Environment** | `repo:ORG/REPO:environment:ENV_NAME` |
| **Tag** | `repo:ORG/REPO:ref:refs/tags/TAG` |
| **Pull Request** | `repo:ORG/REPO:pull_request` |

---

## 5️⃣ Scope Access by Org, Repo, Branch, or Environment

You can create multiple federated credentials on the same App Registration to control which workflows can authenticate:

| Scope Level | Subject Identifier Example | Use Case |
|-------------|---------------------------|----------|
| **Specific branch** | `repo:MyOrg/my-repo:ref:refs/heads/main` | Only `main` branch can deploy |
| **Environment** | `repo:MyOrg/my-repo:environment:production` | Only the `production` environment can deploy |
| **Any branch** | `repo:MyOrg/my-repo:ref:refs/heads/*` | Any branch in the repo (use with caution) |
| **Pull requests** | `repo:MyOrg/my-repo:pull_request` | PR validation deployments |

> 💡 **Tip:** Use **environment-based** scoping for production deployments. Combine with GitHub environment protection rules (required reviewers, wait timers) for defense in depth.

---

## 6️⃣ Configure the GitHub Actions Workflow

### Required Permissions

```yaml
permissions:
  id-token: write   # Required for OIDC token request
  contents: read     # Required for actions/checkout
```

### Full Workflow Example

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
    environment: production   # Must match the federated credential entity

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Log in to Azure
        uses: azure/login@v2
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

### Store IDs as GitHub Secrets

**Navigation:**

```
Repository → Settings → Secrets and variables → Actions → New repository secret
```

| Secret Name | Value |
|-------------|-------|
| `AZURE_CLIENT_ID` | Application (client) ID from the App Registration |
| `AZURE_TENANT_ID` | Directory (tenant) ID from the App Registration |
| `AZURE_SUBSCRIPTION_ID` | Your Azure Subscription ID |

> 💡 **Tip:** These are not sensitive secrets (they are IDs, not credentials), but storing them as secrets keeps workflows clean and portable across environments.

---

## 7️⃣ GHE.com (DRUS) Configuration

When using GitHub Enterprise Cloud with data residency (GHE.com), the OIDC issuer URL is different.

### Key Difference

| Setting | GitHub.com | GHE.com (DRUS) |
|---------|------------|-----------------|
| **OIDC Issuer** | `https://token.actions.githubusercontent.com` | `https://token.actions.SUBDOMAIN.ghe.com` |
| **Audience** | `api://AzureADTokenExchange` | `api://AzureADTokenExchange` |

### Workflow Adjustment for GHE.com

No changes are needed in the workflow YAML itself. The `azure/login` action automatically uses the correct issuer based on the GitHub instance. The federated credential in Azure must use the GHE.com issuer URL.

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

*If you are moving repositories from GitHub.com to GHE.com, existing OIDC trusts must be updated.*

### Steps

1. **Create a new federated credential** on the existing App Registration with the GHE.com issuer:

| Field | New Value |
|-------|-----------|
| **Issuer** | `https://token.actions.SUBDOMAIN.ghe.com` |
| **Subject identifier** | `repo:NEW_ORG/REPO:environment:production` (update org name if changed) |

2. **Test the new trust** by running a deployment from the GHE.com repo
3. **Remove the old federated credential** that used `https://token.actions.githubusercontent.com`

> 💡 **Tip:** Keep both federated credentials active during the migration transition period. Remove the old one only after confirming deployments work from the new instance.

> ⚠️ **Warning:** If the org name changed during migration, the subject identifier must reflect the new org name. The old subject will not match.

## 🧯 Known Errors & Resolutions

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

---

## ❓ Common Questions & Troubleshooting

### Q: I am getting "AADSTS700016: Application not found" when my workflow tries to log in to Azure. What is wrong?
**A:** This error means Azure cannot find the App Registration. Verify that the `client-id` (Application ID) and `tenant-id` (Directory ID) stored in your GitHub secrets are correct and correspond to an active App Registration in the correct Entra ID tenant. Copy-paste errors and extra whitespace in secret values are common causes.

---

### Q: My workflow fails with "No matching federated identity record found." How do I fix this?
**A:** The subject claim in the OIDC token does not match any federated credential on the App Registration. Verify that the organization name, repository name, branch name (or environment name), and entity type in the federated credential exactly match your workflow context. For example, if your credential is scoped to `repo:MyOrg/my-repo:environment:production`, the workflow job must specify `environment: production`. Check for typos and case sensitivity.

---

### Q: OIDC was working on GitHub.com but stopped after we migrated to GHE.com. What changed?
**A:** The OIDC issuer URL is different on GHE.com. Update the federated credential on the Azure App Registration to use `https://token.actions.SUBDOMAIN.ghe.com` instead of `https://token.actions.githubusercontent.com`. You may need to create a new federated credential with the GHE.com issuer and remove the old one after testing.

---

### Q: Azure login succeeds but I get "Permission denied" when deploying to a resource. What should I check?
**A:** The App Registration needs the correct Azure RBAC role assignment on the target resource (subscription, resource group, or individual resource). Verify the role assignment in Azure Portal under Access Control (IAM) for the specific resource. Common mistake: assigning the role at the subscription level when the deployment targets a different subscription, or using a role with insufficient permissions (e.g., Reader instead of Contributor).

---

### Q: My deployment takes longer than 1 hour and the OIDC token expires mid-job. How do I handle this?
**A:** OIDC tokens from the `azure/login` action are valid for approximately 1 hour. For long-running deployments, break the job into smaller steps that each complete within the token lifetime, or re-authenticate mid-job by calling `azure/login` again in a later step. You can also restructure your workflow to use multiple jobs with separate login steps.

---

### Q: Do I need to store any Azure credentials as GitHub secrets with OIDC?
**A:** You store the Application (client) ID, Directory (tenant) ID, and Subscription ID as GitHub secrets -- but these are identifiers, not credentials. No client secrets, certificates, or passwords are needed. The OIDC exchange generates a short-lived token at runtime without any stored credential, which is the primary security advantage of this approach.

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| GitHub Enterprise Importer (GEI) & Actions Importer | `Migration/GitHub Enterprise Importer (GEI) & Actions Importer.md` |
| Data Residency Decision Guide (DRUS vs Standard vs GHES) | `Setup/Data Residency Decision Guide (DRUS vs Standard vs GHES).md` |
| GitHub App for CI/CD (No Seat Cost) | `Actions/GitHub App for CI-CD (No Seat Cost).md` |

---

## 📚 Resources

- [Configuring OpenID Connect in Azure](https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-azure)
- [About Security Hardening with OpenID Connect](https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/about-security-hardening-with-openid-connect)

---

*Last updated: April 2026*
