# 🤖 GitHub App for CI/CD (No Seat Cost) Runbook

> **Complete guide to using GitHub Apps for CI/CD authentication instead of machine users, saving $21/user/month per seat**

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

---

## 📚 Resources

- [About Creating GitHub Apps](https://docs.github.com/en/apps/creating-github-apps/about-creating-github-apps/about-creating-github-apps)
- [actions/create-github-app-token](https://github.com/actions/create-github-app-token)
- [Automatic Token Authentication (GITHUB_TOKEN)](https://docs.github.com/en/actions/security-for-github-actions/security-guides/automatic-token-authentication)

---

*Last updated: April 2026*