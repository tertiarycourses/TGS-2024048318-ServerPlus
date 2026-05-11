# Lab 11 — Directory Services with OpenLDAP

Maps to CompTIA Server+ objective **2.3** — Directory connectivity, and objective **3.3** — User accounts, groups, scope-based delegation. Production servers don't manage users locally; they bind to a directory.

Run all commands on https://killercoda.com/playgrounds/scenario/ubuntu

**Required software (free, apt):** `slapd`, `ldap-utils`, `ldapscripts`

```bash
DEBIAN_FRONTEND=noninteractive apt update && \
  apt install -y slapd ldap-utils ldapscripts
```

When prompted (or via `dpkg-reconfigure -p low slapd`), set:
- Admin password: `LabAdmin1!`
- DNS domain name: `lab.local`
- Organization: `Lab`

```bash
echo "slapd slapd/internal/adminpw password LabAdmin1!" | debconf-set-selections
echo "slapd slapd/password1 password LabAdmin1!" | debconf-set-selections
echo "slapd slapd/password2 password LabAdmin1!" | debconf-set-selections
echo "slapd slapd/domain string lab.local" | debconf-set-selections
echo "slapd shared/organization string Lab" | debconf-set-selections
dpkg-reconfigure -f noninteractive slapd
```

---

## Step 1 — Verify the directory is running

```bash
ldapsearch -x -H ldap://127.0.0.1 -b "dc=lab,dc=local" -s base "(objectclass=*)" namingContexts
```

You should see `dc=lab,dc=local`.

## Step 2 — Create the OU structure (organisational units = scope)

```bash
cat > /tmp/ou.ldif <<'EOF'
dn: ou=people,dc=lab,dc=local
objectClass: organizationalUnit
ou: people

dn: ou=groups,dc=lab,dc=local
objectClass: organizationalUnit
ou: groups
EOF
ldapadd -x -D "cn=admin,dc=lab,dc=local" -w LabAdmin1! -f /tmp/ou.ldif
```

## Step 3 — Add user accounts

```bash
cat > /tmp/users.ldif <<'EOF'
dn: uid=alice,ou=people,dc=lab,dc=local
objectClass: inetOrgPerson
cn: Alice Admin
sn: Admin
uid: alice
userPassword: {SSHA}placeholder
mail: alice@lab.local

dn: uid=bob,ou=people,dc=lab,dc=local
objectClass: inetOrgPerson
cn: Bob Operator
sn: Operator
uid: bob
userPassword: {SSHA}placeholder
mail: bob@lab.local
EOF
ldapadd -x -D "cn=admin,dc=lab,dc=local" -w LabAdmin1! -f /tmp/users.ldif

ldappasswd -x -D "cn=admin,dc=lab,dc=local" -w LabAdmin1! \
  -s 'AlicePass!' "uid=alice,ou=people,dc=lab,dc=local"
ldappasswd -x -D "cn=admin,dc=lab,dc=local" -w LabAdmin1! \
  -s 'BobPass!'   "uid=bob,ou=people,dc=lab,dc=local"
```

## Step 4 — Create groups and assign membership

```bash
cat > /tmp/groups.ldif <<'EOF'
dn: cn=admins,ou=groups,dc=lab,dc=local
objectClass: groupOfNames
cn: admins
member: uid=alice,ou=people,dc=lab,dc=local

dn: cn=operators,ou=groups,dc=lab,dc=local
objectClass: groupOfNames
cn: operators
member: uid=bob,ou=people,dc=lab,dc=local
EOF
ldapadd -x -D "cn=admin,dc=lab,dc=local" -w LabAdmin1! -f /tmp/groups.ldif
```

## Step 5 — Bind as a user and search

```bash
ldapwhoami -x -D "uid=alice,ou=people,dc=lab,dc=local" -w 'AlicePass!'
ldapsearch -x -H ldap://127.0.0.1 -D "uid=alice,ou=people,dc=lab,dc=local" -w 'AlicePass!' \
   -b "ou=people,dc=lab,dc=local" "(objectclass=inetOrgPerson)" uid mail
```

## Step 6 — Delegated administration (scope-based access)

Add an ACL that lets the **admins** group manage entries under `ou=people` but lets everyone else only read public attributes:

```bash
cat > /tmp/acl.ldif <<'EOF'
dn: olcDatabase={1}mdb,cn=config
changetype: modify
add: olcAccess
olcAccess: to dn.subtree="ou=people,dc=lab,dc=local"
  by group.exact="cn=admins,ou=groups,dc=lab,dc=local" write
  by self write
  by * read
EOF
ldapmodify -Y EXTERNAL -H ldapi:/// -f /tmp/acl.ldif
```

This is **delegation by group membership** — a Server+ 3.3 sub-objective.

## Step 7 — How a Linux host would actually use this directory

A client server installs `sssd` (or `nslcd` + `libnss-ldap` + `libpam-ldap`), points `/etc/sssd/sssd.conf` at `ldap://ldap.lab.local`, and from then on `id alice`, `getent passwd alice`, `ssh alice@host` all resolve through LDAP. The same directory backs file-share ACLs, web SSO, and database auth.

## Step 8 — Note on Active Directory

Active Directory uses LDAP under the hood plus Kerberos for auth. Server+ may reference AD in the same breath as "directory services"; the **concepts** (OU = scope, group = role, ACL = delegation) are identical to what you just built.

---

## Free online tools
- **OpenLDAP admin guide** — https://www.openldap.org/doc/admin26/
- **SSSD docs** — https://sssd.io/docs.html
- **Apache Directory Studio (LDAP browser)** — https://directory.apache.org/studio/

## What you learned
- LDAP tree structure: base DN → OUs → entries.
- Adding users, groups, and password changes via `ldapadd` / `ldappasswd`.
- Group-based delegation through `olcAccess` ACLs (scope-based admin).
- How Linux servers consume the directory (sssd) and how this maps to Active Directory.
