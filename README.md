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
ldapsearch -x -LLL -H ldap:/// -b "dc=homelab,dc=local,dc=org"

# Add users via LDIF
ldapadd -x -D "cn=admin,dc=homelab,dc=local,dc=org" -W -f adduser.ldif
```

### Sample LDIF — User Provisioning

```ldif
dn: ou=users,dc=homelab,dc=local,dc=org
objectClass: organizationalUnit
ou: users

dn: cn=Kevin Durant,ou=users,dc=homelab,dc=local,dc=org
objectClass: inetOrgPerson
cn: Kevin Durant
sn: Durant
givenName: Kevin
mail: kevin.durant@homelab.local.org
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

1. **Created a second user (Stephen Curry)** by writing a new LDIF file `adduser2.ldif` and running `ldapadd`
2. **Created an OU for groups** by writing `addgroup.ldif` with `objectClass: organizationalUnit`
3. **Created the `admins` group** and assigned Kevin Durant as a member using `groupOfNames` objectClass
4. **Added the admins group** to the directory using `ldapadd`
5. **Verified the admins group** using `ldapsearch` with the full DN path
6. **Created the `viewers` group** by writing `addgroup2.ldif` and assigned Stephen Curry
7. **Troubleshot an attribute error** — fixed `members` typo to `member` in the LDIF file
8. **Added the viewers group** to the directory using `ldapadd`
9. **Verified both groups** using targeted `ldapsearch` queries
10. **Modified Kevin Durant's email** by writing `modifyuser.ldif` with `changetype: modify` and running `ldapmodify`
11. **Verified the email change** using `ldapsearch` to confirm the updated attribute
12. **Deprovisioned Stephen Curry** using `ldapdelete` to simulate user offboarding
13. **Verified the deletion** by running `ldapsearch` on the users OU to confirm removal

### RBAC Structure

| User | Group | Role |
|---|---|---|
| Kevin Durant | admins | Administrator access |
| Stephen Curry | viewers | Read-only access |

### Commands Used

```bash
# Add second user
ldapadd -x -D "cn=admin,dc=homelab,dc=local,dc=org" -W -f adduser2.ldif

# Create OU for groups and add admins group
ldapadd -x -D "cn=admin,dc=homelab,dc=local,dc=org" -W -f addgroup.ldif

# Verify admins group
ldapsearch -x -LLL -H ldap:/// -b "cn=admins,dc=homelab,dc=local,dc=org"

# Add viewers group
ldapadd -x -D "cn=admin,dc=homelab,dc=local,dc=org" -W -f addgroup2.ldif

# Verify viewers group
ldapsearch -x -LLL -H ldap:/// -b "cn=viewers,dc=homelab,dc=local,dc=org"

# Verify all users
ldapsearch -x -LLL -H ldap:/// -b "ou=users,dc=homelab,dc=local,dc=org"

# Modify a user attribute
ldapmodify -x -D "cn=admin,dc=homelab,dc=local,dc=org" -W -f modifyuser.ldif

# Delete/deprovision a user
ldapdelete -x -D "cn=admin,dc=homelab,dc=local,dc=org" -W "cn=Stephen Curry,ou=users,dc=homelab,dc=local,dc=org"

# Verify deletion
ldapsearch -x -LLL -H ldap:/// -b "ou=users,dc=homelab,dc=local,dc=org"
```

### Sample LDIF — Second User (adduser2.ldif)

```ldif
dn: cn=Stephen Curry,ou=users,dc=homelab,dc=local,dc=org
objectClass: inetOrgPerson
cn: Stephen Curry
sn: Curry
givenName: Stephen
mail: stephen.curry@homelab.local.org
userPassword: Password456
```

### Sample LDIF — Group OU and Admins Group (addgroup.ldif)

```ldif
dn: ou=groups,dc=homelab,dc=local,dc=org
objectClass: organizationalUnit
ou: groups

dn: cn=admins,dc=homelab,dc=local,dc=org
objectClass: groupOfNames
cn: admins
member: cn=Kevin Durant,dc=homelab,dc=local,dc=org
```

### Sample LDIF — Viewers Group (addgroup2.ldif)

```ldif
dn: cn=viewers,dc=homelab,dc=local,dc=org
objectClass: groupOfNames
cn: viewers
member: cn=Stephen Curry,ou=users,dc=homelab,dc=local,dc=org
```

### Sample LDIF — User Modification (modifyuser.ldif)

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
- **Troubleshooting** — Identifying and fixing LDIF syntax errors in a live environment

---

## ✅ Phase 3: Keycloak SSO & Identity Federation

### What I Built
Deployed and configured **Keycloak 26.2.4** as an Identity Provider (IdP), implementing Single Sign-On (SSO) with OpenID Connect (OIDC), and federated it with OpenLDAP so users from the directory service can authenticate through Keycloak.

### Architecture

```
MacBook Air (Host)
└── VMware Fusion
    └── Ubuntu Server 26.04 LTS
        ├── OpenLDAP (port 389) ← Directory Service
        └── Keycloak 26.2.4 (port 8080) ← Identity Provider
            └── User Federation → OpenLDAP
```

---

### Part 1: Installing Keycloak on Ubuntu Server (VM Terminal)

**Step 1 — Install Java 21**
Keycloak requires Java to run. Installed OpenJDK 21 and verified the installation:
```bash
sudo apt install openjdk-21-jdk -y
java -version
# Output: openjdk version "21.0.11-ea" 2026-04-21
```

**Step 2 — Download Keycloak**
Downloaded Keycloak 26.2.4 directly from GitHub releases:
```bash
wget https://github.com/keycloak/keycloak/releases/download/26.2.4/keycloak-26.2.4.tar.gz
```

**Step 3 — Extract and Move to /opt**
```bash
tar -xzf keycloak-26.2.4.tar.gz
sudo mv keycloak-26.2.4 /opt/keycloak
```

Verified the installation:
```bash
ls /opt/keycloak
# Output: LICENSE.txt README.md bin conf lib providers themes version.txt
```

**Step 4 — Create a Dedicated Service Account**
Created a dedicated system user to run Keycloak securely — best practice to never run services as root:
```bash
sudo useradd -r -d /opt/keycloak -s /sbin/nologin keycloak
sudo chown -R keycloak:keycloak /opt/keycloak
sudo chmod -R 750 /opt/keycloak
```

**Step 5 — Bootstrap the Admin User**
Created the initial admin user using the bootstrap command:
```bash
sudo -u keycloak /opt/keycloak/bin/kc.sh bootstrap-admin user --username admin --password <password>
```

**Step 6 — Start Keycloak in the Background**
Used `&` at the end to run Keycloak in the background, which frees the terminal to run more commands while Keycloak is running:
```bash
sudo -u keycloak KEYCLOAK_ADMIN=admin KEYCLOAK_ADMIN_PASSWORD=<password> /opt/keycloak/bin/kc.sh start-dev &
```

Waited for the confirmation message:
```
Keycloak 26.2.4 on JVM started. Listening on: http://0.0.0.0:8080
```

---

### Part 2: Accessing Keycloak via Lynx Text Browser (VM Terminal)

Keycloak only allows admin console access from localhost for security. Since the VM has no graphical browser, installed **Lynx** — a text-based browser — to access Keycloak locally from inside the VM:

```bash
sudo apt install lynx -y
lynx http://localhost:8080
```

Inside Lynx:
- Accepted the cookie prompt by pressing **A** (Always)
- Saw the "Create a temporary administrative user" form
- Used **Tab** to navigate between fields
- Entered username: `admin` and password
- Pressed **Enter** on the Create user button
- Saw confirmation: `KC-SERVICES0077: Created temporary admin user`
- Pressed **Q** to quit Lynx

---

### Part 3: Keycloak Admin Console (Mac Browser)

Opened Safari/Chrome on the MacBook and navigated to:
```
http://172.16.103.129:8080
```

**Step 7 — Create a Permanent Admin Account**

The yellow banner warned that the current account was temporary. Created a permanent admin:
- Navigated to **Users → Add user**
- Set Username: `keycloakadmin`, filled in first and last name
- Clicked **Create**
- Clicked **Credentials tab → Set password**, turned Temporary **Off**, clicked **Save**
- Clicked **Role mapping tab → Assign role**
- Changed filter to **Filter by realm roles**
- Selected **admin** role and clicked **Assign**
- Clicked top right dropdown → **Sign out**
- Logged back in as `keycloakadmin`
- Yellow warning banner was gone confirming permanent admin access

**Step 8 — Create the Homelab Realm**

A Realm in Keycloak is an isolated tenant — best practice is never to use the master realm for actual users:
- Clicked **Manage realms → Create realm**
- Set Realm name: `homelab`
- Clicked **Create**

**Step 9 — Create a User in the Homelab Realm**

- Clicked **Users → Add user**
- Set Username: `homelabuser`, Email: `homelabuser@homelab.local`, filled in name
- Clicked **Create**
- Clicked **Credentials tab → Set password**, turned Temporary **Off**, clicked **Save**

**Step 10 — Create a Realm Role and Assign It**

- Clicked **Realm roles → Create role**
- Set Role name: `homelab-user`, clicked **Save**
- Navigated back to **Users → homelabuser → Role mapping tab**
- Clicked **Assign role**, selected `homelab-user`, clicked **Assign**

**Step 11 — Create an OpenID Connect Client**

A Client represents an application that uses Keycloak for authentication:
- Clicked **Clients → Create client**
- Set Client type: `OpenID Connect`, Client ID: `homelab-app`
- Clicked **Next**, turned **Client authentication On**
- Clicked **Next**, set Valid redirect URIs: `http://localhost:8080/*`
- Clicked **Save**

**Step 12 — Test SSO Login**

Navigated to the Keycloak account portal in the Mac browser:
```
http://172.16.103.129:8080/realms/homelab/account
```
- Logged in as `homelabuser` with the set password
- Successfully accessed the account portal confirming SSO was working

---

### Part 4: Connecting Keycloak to OpenLDAP (User Federation)

**Step 13 — Configure LDAP User Federation**

- Clicked **User federation → Add provider → LDAP**
- Set the following configuration:

| Setting | Value |
|---|---|
| Vendor | Other |
| Connection URL | ldap://localhost:389 |
| Bind DN | cn=admin,dc=homelab,dc=local,dc=org |
| Users DN | ou=users,dc=homelab,dc=local,dc=org |
| Username LDAP attribute | cn |
| RDN LDAP attribute | cn |
| UUID LDAP attribute | cn |
| User object classes | inetOrgPerson |
| Edit mode | READ_ONLY |

- Clicked **Test connection** → Success
- Clicked **Test authentication** → Success
- Clicked **Save**

**Step 14 — Troubleshoot and Sync LDAP Users**

Initial sync failed because UUID, Username, and RDN attributes were set to `uid` instead of `cn`. Fixed all three attributes to `cn` and saved.

- Clicked **Action → Sync all users**
- Navigated to **Users → View all users**
- Confirmed **Kevin Durant** appeared from OpenLDAP alongside `homelabuser`

---

### Keycloak Configuration Summary

| Setting | Value |
|---|---|
| Realm | homelab |
| Client | homelab-app |
| Client Type | OpenID Connect (OIDC) |
| User Federation | LDAP (OpenLDAP) |
| LDAP Connection | ldap://localhost:389 |
| Users DN | ou=users,dc=homelab,dc=local,dc=org |
| Admin Console URL | http://172.16.103.129:8080 |
| SSO Portal URL | http://172.16.103.129:8080/realms/homelab/account |

### IAM Concepts Demonstrated
- **Identity Provider (IdP)** — Keycloak as a centralized authentication authority
- **Single Sign-On (SSO)** — One login for multiple applications
- **OpenID Connect (OIDC)** — Industry standard authentication protocol
- **Realms** — Isolated IAM tenants for different organizations or environments
- **Clients** — Applications registered to use Keycloak for authentication
- **User Federation** — Connecting external directory services to an IdP
- **Identity Synchronization** — Syncing users from LDAP into Keycloak
- **Least Privilege** — Running Keycloak as a dedicated service account, not root
- **Troubleshooting** — Resolving LDAP attribute mismatches during federation setup

---

## ✅ Phase 4: Multi-VM Environment & Centralized Authentication

### What I Built
Provisioned a second Ubuntu Server VM (`iam-client-01`) and joined it to the OpenLDAP domain, enabling centralized authentication across multiple machines — simulating a real enterprise environment where employees use one set of credentials to log into any company machine.

### Architecture

```
MacBook Air (Host)
└── VMware Fusion
    ├── homelab-server (172.16.103.129)
    │   ├── OpenLDAP (port 389) ← Central Directory
    │   └── Keycloak (port 8080) ← Identity Provider
    └── iam-client-01
        └── LDAP Client → authenticates against homelab-server
```

### Steps Completed

**Step 1 — Provisioned Second VM**
- Created a new Ubuntu Server 26.04 LTS VM in VMware Fusion
- Named the server `iam-client-01`
- Installed OpenSSH Server during setup
- Updated all packages after installation

**Step 2 — Configured DNS**
Added Google DNS to `/etc/resolv.conf` to enable internet access:
```
nameserver 8.8.8.8
nameserver 8.8.4.4
```
Verified connectivity with `ping -c 4 google.com` — 0% packet loss

**Step 3 — Installed LDAP Client Packages**
```bash
sudo apt install libnss-ldap libpam-ldap ldap-utils nscd -y
```

During configuration selected **passwd**, **group**, and **shadow** as the name services to enable LDAP lookups for.

**Step 4 — Configured nslcd to Point to LDAP Server**

Edited `/etc/nslcd.conf` to point to the OpenLDAP server:
```bash
sudo nano /etc/nslcd.conf
```

Key settings:
```
uri ldap://172.16.103.129
base dc=homelab,dc=local,dc=org
```

Restarted the service:
```bash
sudo systemctl restart nslcd
```

**Step 5 — Added Unix Attributes to Kevin Durant (on homelab-server)**

Linux requires `uidNumber`, `gidNumber`, `homeDirectory` and `loginShell` to recognize LDAP users. Added the `posixAccount` objectClass and required attributes to Kevin Durant's LDAP entry:

```bash
sudo nano updatekevin.ldif
```

```ldif
dn: cn=Kevin Durant,ou=users,dc=homelab,dc=local,dc=org
changetype: modify
add: objectClass
objectClass: posixAccount
-
add: uid
uid: kevin.durant
-
add: uidNumber
uidNumber: 10001
-
add: gidNumber
gidNumber: 10001
-
add: homeDirectory
homeDirectory: /home/kevin.durant
-
add: loginShell
loginShell: /bin/bash
```

```bash
ldapmodify -x -D "cn=admin,dc=homelab,dc=local,dc=org" -W -f updatekevin.ldif
```

**Step 6 — Verified LDAP User Visible on Client VM**

On `iam-client-01`:
```bash
getent passwd | grep "kevin"
# Output: kevin.durant:*:10001:10001:Kevin Durant:/home/kevin.durant:/bin/bash
```

**Step 7 — Enabled LDAP Authentication via PAM**

Created home directory for Kevin Durant:
```bash
sudo mkdir -p /home/kevin.durant
sudo chown 10001:10001 /home/kevin.durant
```

Ran PAM configuration:
```bash
sudo pam-auth-update
```

Enabled the following PAM profiles:
- ✅ Unix authentication
- ✅ LDAP Authentication
- ✅ Register user sessions in the systemd control group hierarchy
- ✅ Create home directory on login
- ✅ Inheritable Capabilities Management

**Step 8 — Successfully Logged in as Kevin Durant**

```bash
su - kevin.durant
# Output: kevin.durant@iam-client-01:~$

whoami
# Output: kevin.durant
```

Kevin Durant successfully authenticated on `iam-client-01` using credentials stored on `homelab-server` OpenLDAP — centralized authentication working across two VMs!

### IAM Concepts Demonstrated
- **Centralized Authentication** — One set of credentials works across multiple machines
- **LDAP Client Configuration** — Joining a Linux machine to an LDAP domain
- **PAM (Pluggable Authentication Modules)** — Configuring Linux authentication stack
- **NSS (Name Service Switch)** — Routing identity lookups to LDAP
- **posixAccount** — Unix/Linux user attributes required for system login
- **Multi-machine IAM** — Simulating enterprise environment with multiple servers
- **Troubleshooting** — Resolving objectClass violations and missing attributes

---

## 🚧 Coming Soon

- [ ] **SAML Integration** — Federated identity management

---

## 🎯 Goals

This homelab is designed to build practical, demonstrable IAM skills including:
- Directory service administration
- Identity lifecycle management (provision, modify, deprovision)
- Single Sign-On (SSO) configuration
- Role-Based Access Control (RBAC)
- Multi-machine centralized authentication
- Industry-standard protocols: LDAP, OAuth2, OIDC, SAML

---

## 📚 Resources

- [OpenLDAP Documentation](https://www.openldap.org/doc/)
- [Ubuntu Server Guide](https://ubuntu.com/server/docs)
- [Keycloak Documentation](https://www.keycloak.org/documentation)
- [Keycloak Downloads](https://github.com/keycloak/keycloak/releases)
- [Keycloak Getting Started Guide](https://www.keycloak.org/getting-started)

---

*This project is actively being built and updated as I progress through each phase.*
