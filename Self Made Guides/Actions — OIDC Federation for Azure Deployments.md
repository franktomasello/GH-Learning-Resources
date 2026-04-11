# 🔗 OIDC Federation for GitHub Actions to Azure Deployments Runbook

> **Complete guide to configuring passwordless deployments from GitHub Actions to Azure using OpenID Connect (OIDC) federation**

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

---

## 📚 Resources

- [Configuring OpenID Connect in Azure](https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-azure)
- [About Security Hardening with OpenID Connect](https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/about-security-hardening-with-openid-connect)

---

*Last updated: April 2026*