# 🚀 Copilot Standalone Guidance

> **How to assign Copilot Business licenses without consuming GitHub Enterprise (GHE) licenses**

---

## 📋 Objective

Assign Copilot Business licenses to ~1,000 users **without consuming GitHub Enterprise (GHE) licenses**.

---

## 🔧 Phase 1: The Setup (Critical Configuration)

### Step 1: Navigate to Enterprise Settings

1. Click your profile photo (top-right) → **Your enterprises**
2. Select the target enterprise

### Step 2: Create the Enterprise Team (The "Zero-Cost" Step)

> ⚠️ **CRITICAL**: This step determines if you pay for 1,000 extra GHE licenses or not. Follow exactly.

1. Go to **People** → **Enterprise teams**
2. Click **Create enterprise team**
3. **Name:** Use a functional name (e.g., `copilot-standalone-users`, `contractor-devs`)
4. **Organizations:** **LEAVE THIS BLANK**

#### 🛑 WARNING: Organization Access
- **DO NOT** select any organization
- **Why?** If you grant this team access to an organization (even "Read" access), every user in it becomes a "standard enterprise member" and immediately consumes a full GitHub Enterprise license (approx. $21/mo or $231/yr depending on contract)
- **Correct State:** The team exists at the *enterprise* level only, with no org affiliation

---

## 👥 Phase 2: Add Users to the Team

Choose the method that matches your infrastructure.

### Option A: IdP Group Sync (Best for Long-Term/EMU)

*Requires Enterprise Managed Users (EMU) or Team Sync configured.*

1. In your IdP (Okta, Azure AD, etc.), create a group (e.g., `github-copilot-users`)
2. In GitHub **Enterprise teams**, select your team (`copilot-standalone-users`)
3. Go to **Settings** → **Identity Provider Groups**
4. Connect the IdP group
   - **Constraint:** The GitHub team must have **no manually assigned users** before you link the group
   - **Result:** Membership is now 100% automated by your IdP

### Option B: Scripted REST API (Best for Bulk Initial Setup)

*Use this if you have a CSV of usernames and don't use IdP sync.*

**Correct Endpoint:**
```
PUT /enterprises/{enterprise}/teams/{enterprise-team}/memberships/{username}
```

**Bulk Add Option (Recommended for 1,000 users):**
```
POST /enterprises/{enterprise}/teams/{enterprise-team}/memberships/add
```
*(accepts an array of usernames)*

**Important Note:** Enterprise team slugs receive an `ent:` prefix. Use the full slug format in API calls.

**Logic:**
1. Read list of usernames
2. Loop through list: Call endpoint for each user (or use bulk add)
3. Handle 404s (user not found) or 422s (already member)

### Option C: Manual UI (Not Recommended)

- Valid only for <50 users
- Do not attempt for 1,000 users

---

## 📜 Phase 3: Assign the License

1. In the Enterprise account, go to **Billing & licensing**
2. Select **Licensing** → **Copilot**
3. Click **Manage** (next to Copilot Business)
4. Switch to the **Enterprise teams** tab (not the default "Organizations" tab)
5. Click **Assign licenses**
6. Select your team (`copilot-standalone-users`) and confirm

### ✅ Outcome
- **All 1,000 users now have a Copilot Business license**
- **Cost:** 1,000 × Copilot Business rate
- **GHE Seat Consumption:** 0 (Verified)

---

## ✓ Phase 4: Verification & Limits (Triple-Checked)

| Check | Verified Fact |
|:------|:--------------|
| **Team Capacity** | **5,000 Members.** The limit was increased from 500 to 5,000 per enterprise team in December 2025. |
| **License Consumption** | **Copilot Only.** Users in an enterprise team *without* org access do NOT consume a GHE/Visual Studio license. |
| **Billing Deduplication** | **Unique User.** If a user is already licensed for Copilot in an Org, adding them here generally does not double-bill; the system bills per unique user based on GitHub's combined billing model. |
| **Scope** | **Copilot Business Only.** This method does *not* grant Copilot Enterprise features (which require repo context/org access). |
| **Feature Status** | **Public Preview.** Enterprise Teams remain in Public Preview, though widely stable for this use case. |

---

## ⚠️ Common Pitfall to Avoid

### The "Upgrade" Trap

If you later decide to upgrade these users to **Copilot Enterprise** (to use Chat with your codebase), you will need to grant them access to repositories.

- **The Moment you do this:** You must add them to an Organization
- **The Cost:** They will immediately consume a **GitHub Enterprise license** + the **Copilot Enterprise upgrade** cost
- **Conclusion:** Keep them on **Copilot Business** if the goal is "standalone" usage (IDE completion/chat without repo context)

---

## 📚 Sources

1. [Creating enterprise teams](https://docs.github.com/en/enterprise-cloud@latest/enterprise-onboarding/setting-up-organizations-and-teams/creating-teams)
2. [REST API endpoints for enterprise team memberships](https://docs.github.com/en/enterprise-cloud@latest/rest/enterprise-teams/enterprise-team-members)
3. [Granting users access to GitHub Copilot in your enterprise](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/administer-copilot/manage-for-enterprise/manage-access/grant-access)
4. [Combined GitHub Enterprise cloud and server use](https://docs.github.com/en/enterprise-cloud@latest/billing/concepts/enterprise-billing/combined-enterprise-use)
5. [Enterprise teams product limits increased by over 10x](https://github.blog/changelog/2025-12-08-enterprise-teams-product-limits-increased-by-over-10x/)

---

*Last Updated: December 2025*
