# 🤖 GitHub App CI/CD Authentication (No Seat Cost) Runbook

> **Complete guide to using GitHub Apps for CI/CD authentication instead of machine users, saving $21/user/month per seat**

---

## ⚡ Quick-Start Summary

> **For experienced admins who just need the click paths:**

- **Create GitHub App:** `Org → Settings → Developer settings → GitHub Apps → New GitHub App`
- **Set permissions:** `Org → Settings → Developer settings → GitHub Apps → [App] → Permissions & events`
- **Generate private key:** `GitHub Apps → [App] → General → Private keys → Generate a private key`
- **Install App:** `GitHub Apps → [App] → Install App → Install → Select repositories`
- **In workflow:** Use `actions/create-github-app-token@v1` with `app-id` and `private-key` secrets

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
| Org owner or admin role to create and install GitHub Apps | ☐ |
| GitHub Actions enabled on target repositories | ☐ |
| Permissions identified for your CI/CD use case (Contents, PRs, Checks, etc.) | ☐ |

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

**Navigation:**

```
Organization → Settings → Developer settings (sidebar) → GitHub Apps → New GitHub App
```

**Steps:**

1. Fill in the required fields:

| Field | Value |
|-------|-------|
| **GitHub App name** | Descriptive name (e.g., `my-org-ci-bot`) |
| **Homepage URL** | Your org URL or repo URL |
| **Webhook** | Uncheck **Active** (not needed for CI/CD token generation) |

2. Click **Create GitHub App**

> 💡 **Tip:** Disabling the webhook is recommended for CI/CD-only apps. Webhooks are only needed if the app must respond to GitHub events in real time.

---

## 3️⃣ Configure Permissions

**Navigation:**

```
Organization → Settings → Developer settings → GitHub Apps → [Your App]
  → Permissions & events
```

**Set the minimum permissions your CI/CD workflows require:**

| Permission | Access Level | Use Case |
|------------|-------------|----------|
| **Contents** | Read & write | Clone repos, push commits, create releases |
| **Pull requests** | Read & write | Create/update PRs, post comments |
| **Checks** | Write | Report CI status checks |
| **Issues** | Read & write | Create/update issues from automation |
| **Metadata** | Read-only | Always required (auto-selected) |
| **Actions** | Read-only | Read workflow run status |
| **Packages** | Read & write | Publish/consume GitHub Packages |

> ⚠️ **Warning:** Only grant permissions that your workflows actually need. Each additional permission expands the access surface of generated tokens.

### Generate and Store the Private Key

**Navigation:**

```
Organization → Settings → Developer settings → GitHub Apps → [Your App]
  → General → Private keys → Generate a private key
```

A `.pem` file will download. Store it as a GitHub Actions secret:

**Navigation:**

```
Repository (or Organization) → Settings → Secrets and variables → Actions
  → New repository secret (or New organization secret)
```

| Secret Name | Value |
|-------------|-------|
| `APP_PRIVATE_KEY` | Contents of the `.pem` file |
| `APP_ID` | The App ID from the GitHub App's General page |

---

## 4️⃣ Install the App to Your Organization

**Navigation:**

```
Organization → Settings → Developer settings → GitHub Apps → [Your App]
  → Install App → Install
```

**Choose installation scope:**

| Option | Effect |
|--------|--------|
| **All repositories** | App can generate tokens for any repo in the org |
| **Only select repositories** | App is limited to the repos you choose |

> 💡 **Tip:** Start with **Only select repositories** and add repos as needed. This follows the principle of least privilege.

---

## 5️⃣ Generate Installation Access Token in CI/CD

### Using the `actions/create-github-app-token` Action

```yaml
name: CI Pipeline

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Generate App Token
        id: app-token
        uses: actions/create-github-app-token@v1
        with:
          app-id: ${{ secrets.APP_ID }}
          private-key: ${{ secrets.APP_PRIVATE_KEY }}

      - name: Checkout with App Token
        uses: actions/checkout@v4
        with:
          token: ${{ steps.app-token.outputs.token }}

      - name: Create a Pull Request
        uses: peter-evans/create-pull-request@v6
        with:
          token: ${{ steps.app-token.outputs.token }}
          title: "Automated update"
          branch: automated-update
```

### Scoping the Token to Specific Repos

```yaml
      - name: Generate App Token (scoped)
        id: app-token
        uses: actions/create-github-app-token@v1
        with:
          app-id: ${{ secrets.APP_ID }}
          private-key: ${{ secrets.APP_PRIVATE_KEY }}
          repositories: "repo-a,repo-b"
```

> ✅ **Result:** A short-lived installation access token (valid ~1 hour) is generated and can be used in subsequent steps. No seat is consumed.

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
        uses: actions/checkout@v4

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
| **PAT support** | N/A (uses installation tokens) | Only fine-grained PATs (classic PATs blocked) |
| **IdP requirement** | None | Must exist in Entra ID / Okta and be assigned to the GitHub app in IdP |
| **Deprovisioning** | Uninstall from org | Must be deprovisioned via SCIM |

> 💡 **Tip:** On EMU, machine users cannot be created directly in GitHub. They must be provisioned through your identity provider via SCIM. GitHub Apps bypass this complexity entirely and are strongly preferred.

### EMU Machine User Provisioning (If Required)

**Steps:**

1. Create a service account in your IdP (Entra ID or Okta)
2. Assign it to the GitHub EMU SCIM application
3. Wait for SCIM provisioning to create the GitHub user
4. Generate a fine-grained PAT from the provisioned account
5. Store the PAT as a GitHub Actions secret

> ⚠️ **Warning:** If the IdP service account is disabled or deprovisioned, the GitHub machine user and all its PATs are immediately invalidated. Plan for IdP service account lifecycle management.

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

### Q: My workflow fails with "Resource not accessible by integration" when using a GitHub App token. What is wrong?
**A:** This error means the App does not have the required permission for the API endpoint being called, OR the App is not installed on the target repository. Check two things: (1) the App's permissions in its settings include the necessary access (e.g., Contents: Read & Write for pushing code), and (2) the App is installed on the specific repo you are trying to access (check Org Settings > Developer settings > GitHub Apps > [Your App] > Install App).

---

### Q: Token generation is failing in my workflow with the `actions/create-github-app-token` action. What should I check?
**A:** Verify that: (1) the `APP_ID` secret contains the correct numeric App ID from the App's General page, (2) the `APP_PRIVATE_KEY` secret contains the full contents of the `.pem` file including the `-----BEGIN RSA PRIVATE KEY-----` header and footer, and (3) the private key has not been regenerated since you stored it (regenerating invalidates previous keys). Also ensure there are no extra newlines or whitespace in the secret values.

---

### Q: When should I use a GitHub App vs a Personal Access Token (PAT)?
**A:** Use a GitHub App for organization-wide CI/CD automation -- it provides scoped, short-lived tokens with no seat cost and better audit trails. Use a PAT only for quick personal scripts or when a third-party tool specifically requires a PAT and cannot accept App tokens. GitHub Apps are strongly recommended for any production automation.

---

### Q: I created a GitHub App but it does not appear in my organization. Where is it?
**A:** If the App was created under your personal account (Settings > Developer settings > GitHub Apps), it will not appear in the organization. To create an org-level App, navigate to Organization > Settings > Developer settings > GitHub Apps > New GitHub App. You can transfer an existing personal App to an org, but it is simpler to create it at the org level from the start.

---

### Q: How are GitHub App installation tokens scoped? Can a token access any repo in the org?
**A:** Installation tokens are scoped to the repositories where the App is installed. If the App is installed on "All repositories," the token can access any repo in the org. If installed on "Only select repositories," the token is limited to those specific repos. You can further narrow the scope at runtime by passing the `repositories` parameter to the `actions/create-github-app-token` action.

---

### Q: How long do GitHub App installation tokens last, and do I need to handle rotation?
**A:** Installation tokens are valid for approximately 1 hour and are automatically generated fresh on each workflow run by the `actions/create-github-app-token` action. You do not need to manually rotate them. This short lifetime is a key security advantage over PATs, which can be long-lived and require manual rotation.

## 🔗 Related Guides

| Guide | Location |
|-------|----------|
| Minutes Governance & Runner Strategy | `Actions/Minutes Governance & Runner Strategy.md` |
| OIDC Federation for Azure Deployments | `Actions/OIDC Federation for Azure Deployments.md` |
| Copilot Seat Assignment & Enablement | `Copilot/Seat Assignment & Enablement.md` |

---

## 📚 Resources

- [About Creating GitHub Apps](https://docs.github.com/en/apps/creating-github-apps/about-creating-github-apps/about-creating-github-apps)
- [actions/create-github-app-token](https://github.com/actions/create-github-app-token)
- [Automatic Token Authentication (GITHUB_TOKEN)](https://docs.github.com/en/actions/security-for-github-actions/security-guides/automatic-token-authentication)

---

*Last updated: April 2026*
