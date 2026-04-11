# 💺 GitHub Copilot Seat Assignment & Enablement Runbook

> **Complete guide to assigning Copilot seats across enterprise, organization, and team levels including pilot rollouts and mixed-plan scenarios**

---

## 📋 Overview

This runbook covers every way to assign and manage GitHub Copilot seats:

| Method | Scope | Best For |
|--------|-------|----------|
| **Enterprise-level org selection** | Entire enterprise | Choosing which orgs get Copilot |
| **Org-level selected members** | Specific users | Controlled rollouts, budget management |
| **Team-based assignment** | GitHub Team | Pilot programs, department rollouts |
| **Org-level all members** | Entire organization | Full org enablement |
| **Enterprise tier assignment** | Enterprise | Assigning Business vs Enterprise plans to orgs |

---

## 1️⃣ Enable Copilot for Specific Organizations (Enterprise Level)

*Choose which organizations under your enterprise have access to Copilot*

**Navigation:**

```
Enterprise → AI controls → Copilot → Access
  → "Enabled for selected organizations"
```

**Steps:**

1. Select **Enabled for selected organizations**
2. Use the search box to find organizations
3. Check the organizations you want to enable
4. Save changes

> 💡 **Tip:** Start with a single organization for your pilot, then expand as adoption grows.

---

## 2️⃣ Assign Seats to Specific Users (Organization Level)

*Grant Copilot access to individual members within an organization*

**Navigation:**

```
Profile Picture → Organizations → [Your Organization] → Settings
  → Copilot → Access → "Enabled for selected members"
```

**Steps:**

1. Select **Enabled for selected members**
2. Click **Add members**
3. Search for and select the users to grant access
4. Confirm the seat assignments

> ⚠️ **Important:** Each assigned seat counts toward your Copilot license total. Users who are assigned but never activate Copilot still consume a seat.

---

## 3️⃣ Team-Based Assignment for Pilots

*Recommended approach for running a Copilot pilot program*

### A) Create a GitHub Team

**Navigation:**

```
Profile Picture → Organizations → [Your Organization]
  → Teams → New team
```

**Steps:**

1. Enter a team name (e.g., `copilot-pilot`)
2. Add a description (e.g., "Copilot pilot participants - Q2 2026")
3. Set visibility to **Visible** or **Secret** depending on preference
4. Click **Create team**

### B) Add Pilot Users to the Team

**Navigation:**

```
Profile Picture → Organizations → [Your Organization]
  → Teams → [copilot-pilot] → Members → Add member
```

### C) Grant Copilot Access to the Team

**Navigation:**

```
Profile Picture → Organizations → [Your Organization] → Settings
  → Copilot → Access → "Enabled for selected members"
    → Add teams → Search for [copilot-pilot] → Select → Save
```

> ✅ **Result:** Only members of the `copilot-pilot` team will have Copilot access. New members added to the team automatically receive a seat.

> 💡 **Tip:** Team-based assignment makes it easy to add or remove pilot participants without managing individual seats. It also simplifies reporting because you can track usage by team.

---

## 4️⃣ Enable Copilot for All Members (Organization Level)

*Grant Copilot access to every member of the organization*

**Navigation:**

```
Profile Picture → Organizations → [Your Organization] → Settings
  → Copilot → Access → "All members"
```

**Steps:**

1. Select **All members**
2. Confirm the enablement

> ⚠️ **Important:** This will assign a seat to every current member and automatically assign seats to new members as they join the organization. Monitor your seat count against your license total.

---

## 5️⃣ Mixed Plans: Business + Enterprise in the Same Enterprise

*Assign different Copilot tiers to different organizations*

### Understanding Mixed Plans

| Tier | Features | Typical Use |
|------|----------|-------------|
| **Copilot Business** | Code completions, Chat, CLI | Standard developer productivity |
| **Copilot Enterprise** | Business features + knowledge bases, Bing search, PR summaries | Teams needing advanced features |

### Assign Tiers at the Enterprise Level

**Navigation:**

```
Enterprise → AI controls → Copilot → Access
```

**Steps:**

1. For each enabled organization, select the Copilot tier:
   - **Copilot Business**
   - **Copilot Enterprise**
2. Save changes

> 💡 **Tip:** You can run some organizations on Copilot Business and others on Copilot Enterprise under the same enterprise. This lets you control costs while giving advanced features to teams that need them.

---

## 6️⃣ Resolving Duplicate Tier Assignments

*When a user has both Business and Enterprise assignments*

### How Duplicates Happen

A user can end up with multiple Copilot assignments if they belong to more than one organization under the enterprise, and those organizations are assigned different Copilot tiers.

### Resolution Rules

| Scenario | Result |
|----------|--------|
| User has Business in Org A + Enterprise in Org B | User gets **Enterprise** (highest tier wins) |
| User has Business in Org A + Business in Org B | User consumes **one** Business seat |

### Identifying Duplicate Assignments

**Navigation:**

```
Enterprise → AI controls → Copilot → Access
```

Review the seat assignment summary. Users with multiple assignments will be flagged.

> 💡 **Tip:** To avoid unnecessary costs, audit seat assignments periodically. Remove users from the lower-tier organization's Copilot access if they already have a higher tier through another org.

---

## 7️⃣ Copilot Enterprise vs Business Assignment at the Enterprise Level

*Set the default tier and manage organization-level plan assignments*

**Navigation:**

```
Enterprise → AI controls → Copilot → Access
```

**Options:**

| Setting | Effect |
|---------|--------|
| **Copilot Business** for an org | All seats in that org are Business tier |
| **Copilot Enterprise** for an org | All seats in that org are Enterprise tier |

> ⚠️ **Important:** Changing an organization's tier from Enterprise to Business will immediately remove access to Enterprise-only features (knowledge bases, Bing search in Chat, PR summaries) for all users in that org.

---

## 🚀 Quick Pilot Rollout Recipe

*Recommended steps for a controlled Copilot pilot:*

### Steps

1. **Enable Copilot for the pilot organization:**
   ```
   Enterprise → AI controls → Copilot → Access
     → Enabled for selected organizations → [Pilot Org]
   ```

2. **Create a pilot team:**
   ```
   Org → Teams → New team → "copilot-pilot"
   ```

3. **Add pilot users to the team** (50-100 recommended)

4. **Grant Copilot to the team:**
   ```
   Org Settings → Copilot → Access
     → Enabled for selected members → Add teams → copilot-pilot
   ```

5. **Set policies** (model access, public code filter, content exclusions)

6. **Run for 30-60 days**, then review metrics

---

## 📝 Additional Notes

> 💡 **Customization:** The exact UI labels and options may vary slightly depending on your enterprise agreement and whether you are on Copilot Business, Copilot Enterprise, or a trial. The navigation paths above reflect the current UI at time of writing.

---

## 📚 Resources

- [Managing access to GitHub Copilot in your organization](https://docs.github.com/en/copilot/managing-copilot/managing-github-copilot-in-your-organization/managing-access-to-github-copilot-in-your-organization)
- [Managing policies and features for Copilot in your enterprise](https://docs.github.com/en/copilot/managing-copilot/managing-copilot-for-your-enterprise/managing-policies-and-features-for-copilot-in-your-enterprise)

---

*Last updated: April 2026*
