# 🔐 IAM Homelab Project

A hands-on Identity and Access Management (IAM) homelab built from scratch on a local virtual machine. This project demonstrates practical IAM skills including directory services, user provisioning, and access control — all core competencies for IAM Analyst and Engineer roles.

---

## 🛠️ Environment

| Component | Details |
|---|---|
| Host Machine | MacBook Air (Intel Core) |
| Operating System | macOS Sequoia |
| Virtualization | VMware Fusion 13.6.3 |
| Guest OS | Ubuntu Server 26.04 LTS |

---

## ✅ Phase 1: Directory Services with OpenLDAP

### What I Built
Deployed and configured an **OpenLDAP** directory service on Ubuntu Server — a lightweight, open-source implementation of the LDAP protocol used in enterprise environments for centralized identity management.

### Steps Completed

1. **Provisioned a Ubuntu Server 26.04 LTS VM** in VMware Fusion
2. **Installed and configured OpenLDAP (slapd)** with a custom domain
3. **Configured the LDAP directory** with domain `homelab.local`
4. **Created an Organizational Unit (OU)** to structure user accounts
5. **Provisioned user accounts** using LDIF files
6. **Verified directory entries** using `ldapsearch` queries

### Commands Used

```bash
# Install OpenLDAP
sudo apt install slapd ldap-utils -y

# Reconfigure and set up domain
sudo dpkg-reconfigure slapd

# Verify LDAP is running
sudo systemctl status slapd

# Query the directory
ldapsearch -x -LLL -H ldap:/// -b "dc=homelab,dc=local"

# Add users via LDIF
ldapadd -x -D "cn=admin,dc=homelab,dc=local" -W -f adduser.ldif
```

### Sample LDIF — User Provisioning

```ldif
dn: ou=users,dc=homelab,dc=local
objectClass: organizationalUnit
ou: users

dn: cn=John Smith,ou=users,dc=homelab,dc=local
objectClass: inetOrgPerson
cn: John Smith
sn: Smith
givenName: John
mail: john.smith@homelab.local
userPassword: Password123
```

### IAM Concepts Demonstrated
- **Directory Services** — Centralized identity store using LDAP protocol
- **Organizational Units (OUs)** — Logical grouping of user accounts
- **User Provisioning** — Creating and managing user identities
- **LDIF Format** — Industry-standard format for directory operations
- **Access Control** — Admin-authenticated directory modifications

---

## ✅ Phase 2: Group Management, RBAC & Identity Lifecycle

### What I Built
Implemented **Role-Based Access Control (RBAC)** by creating groups and assigning users to them, then practiced the full **identity lifecycle** — provisioning, modifying, and deprovisioning user accounts.

### Steps Completed

1. **Created an Organizational Unit (OU)** for groups
2. **Provisioned multiple user accounts** — Kevin Durant and Stephen Curry
3. **Created an `admins` group** and assigned Kevin Durant
4. **Created a `viewers` group** and assigned Stephen Curry
5. **Modified a user attribute** — updated Kevin Durant's email address
6. **Deprovisioned a user** — deleted Stephen Curry to simulate offboarding
7. **Verified all changes** using `ldapsearch` queries

### RBAC Structure

| User | Group | Role |
|---|---|---|
| Kevin Durant | admins | Administrator access |
| Stephen Curry | viewers | Read-only access |

### Commands Used

```bash
# Add a group via LDIF
ldapadd -x -D "cn=admin,dc=homelab,dc=local,dc=org" -W -f addgroup.ldif

# Modify a user attribute
ldapmodify -x -D "cn=admin,dc=homelab,dc=local,dc=org" -W -f modifyuser.ldif

# Delete/deprovision a user
ldapdelete -x -D "cn=admin,dc=homelab,dc=local,dc=org" -W "cn=Stephen Curry,ou=users,dc=homelab,dc=local,dc=org"

# Verify groups
ldapsearch -x -LLL -H ldap:/// -b "cn=admins,dc=homelab,dc=local,dc=org"
```

### Sample LDIF — Group Creation

```ldif
dn: cn=admins,dc=homelab,dc=local,dc=org
objectClass: groupOfNames
cn: admins
member: cn=Kevin Durant,dc=homelab,dc=local,dc=org
```

### Sample LDIF — User Modification

```ldif
dn: cn=Kevin Durant,ou=users,dc=homelab,dc=local,dc=org
changetype: modify
replace: mail
mail: kdurant@homelab.local.org
```

### IAM Concepts Demonstrated
- **Role-Based Access Control (RBAC)** — Permissions assigned via groups not individuals
- **Group Management** — Creating and managing groups in a directory service
- **Identity Lifecycle Management** — Full cycle of provision, modify, and deprovision
- **User Offboarding** — Securely removing access when a user leaves
- **Attribute Management** — Updating user attributes in a live directory

---

## 🚧 Coming Soon

- [ ] **Keycloak SSO** — Single Sign-On with OAuth2 and OpenID Connect (OIDC)
- [ ] **Multi-VM Environment** — Joining a second VM to the LDAP domain
- [ ] **SAML Integration** — Federated identity management

---

## 🎯 Goals

This homelab is designed to build practical, demonstrable IAM skills including:
- Directory service administration
- Identity lifecycle management (provision, modify, deprovision)
- Single Sign-On (SSO) configuration
- Role-Based Access Control (RBAC)
- Industry-standard protocols: LDAP, OAuth2, OIDC, SAML

---

## 📚 Resources

- [OpenLDAP Documentation](https://www.openldap.org/doc/)
- [Ubuntu Server Guide](https://ubuntu.com/server/docs)
- [Keycloak Documentation](https://www.keycloak.org/documentation)

---

*This project is actively being built and updated as I progress through each phase.*
