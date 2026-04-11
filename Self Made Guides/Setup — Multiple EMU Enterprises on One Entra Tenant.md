# 🔗 Multiple EMU Enterprises on One Entra Tenant Runbook

> How to connect multiple GitHub Enterprise Managed User instances to a single Microsoft Entra ID tenant

---

## 📋 Overview

Multiple EMU enterprises CAN connect to a single Entra ID tenant. Each EMU enterprise requires its own separate Enterprise Application registration.

## 1️⃣ Create Separate Enterprise Applications

For each EMU enterprise, create a dedicated Entra Enterprise Application:

```
Microsoft Entra Admin Center
  → Enterprise applications → + New application
    → Browse gallery → Search: "GitHub Enterprise Managed User"
      → Create (name it: "GitHub EMU - Enterprise A")
```

Repeat for each enterprise:
- "GitHub EMU - Enterprise A"
- "GitHub EMU - Enterprise B"
- etc.

## 2️⃣ Configure SAML/OIDC Per Application

Each Enterprise Application has its own SAML/OIDC configuration pointing to its respective EMU enterprise:

### For github.com EMU

| Field | Enterprise A | Enterprise B |
|-------|-------------|-------------|
| Entity ID | `https://github.com/enterprises/ENTERPRISE_A` | `https://github.com/enterprises/ENTERPRISE_B` |
| Reply URL | `https://github.com/enterprises/ENTERPRISE_A/saml/consume` | `https://github.com/enterprises/ENTERPRISE_B/saml/consume` |
| Sign-on URL | `https://github.com/enterprises/ENTERPRISE_A/sso` | `https://github.com/enterprises/ENTERPRISE_B/sso` |

### For GHE.com EMU (Data Residency)

| Field | Enterprise A | Enterprise B |
|-------|-------------|-------------|
| Entity ID | `https://SUBDOMAIN_A.ghe.com/enterprises/SUBDOMAIN_A` | `https://SUBDOMAIN_B.ghe.com/enterprises/SUBDOMAIN_B` |

## 3️⃣ Configure SCIM Provisioning Separately

Each Enterprise Application needs its own SCIM configuration:

```
Entra ID → Enterprise Applications → [GitHub EMU App for Enterprise A]
  → Provisioning → Automatic
    → Tenant URL: (from Enterprise A's GitHub settings)
    → Secret token: (from Enterprise A's setup user)
      → Test connection → Save
```

Repeat with Enterprise B's credentials for the second app.

## 4️⃣ Scope User Populations Using Entra Groups

```
Entra ID → Enterprise Applications → [GitHub EMU App]
  → Users and groups → Add user/group
    → Select the Entra group for this enterprise
```

| Entra Group | Assigned To | Result |
|-------------|------------|--------|
| `GitHub-Enterprise-A-Users` | Enterprise A App | Users provisioned as `user_SHORTCODE_A` |
| `GitHub-Enterprise-B-Users` | Enterprise B App | Users provisioned as `user_SHORTCODE_B` |

> 💡 **Tip:** A single Entra user CAN be assigned to multiple Enterprise Applications, creating separate managed identities in each EMU enterprise.

## 5️⃣ Key Considerations

| Topic | Detail |
|-------|--------|
| **Username uniqueness** | Each enterprise has a different shortcode suffix (e.g., `jsmith_contoso` vs `jsmith_fabrikam`) |
| **Email domains** | Separate email domains are NOT required — same domain works across enterprises |
| **SCIM attribute mappings** | Must produce unique usernames per enterprise |
| **Testing** | Test authentication and provisioning for each enterprise independently |
| **Audit** | Each enterprise has its own audit log |
| **Billing** | Each enterprise has separate billing |

## ⚠️ Important Notes

- Each EMU enterprise has fully separate governance, billing, and audit
- There is NO cross-enterprise visibility without additional tooling
- Guest collaborators can enable cross-enterprise access where needed
- SCIM provisioning cycles are independent — changes to one don't affect the other

---

## 📝 Resources

| Resource | Link |
|----------|------|
| Configuring SCIM for EMU | https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/provisioning-user-accounts-with-scim/configuring-scim-provisioning-for-enterprise-managed-users |
| About EMU | https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/understanding-iam-for-enterprises/about-enterprise-managed-users |
| Configuring SAML for EMU | https://docs.github.com/en/enterprise-cloud@latest/admin/managing-iam/configuring-authentication-for-enterprise-managed-users/configuring-saml-single-sign-on-for-enterprise-managed-users |

---

*Last updated: April 2026*
