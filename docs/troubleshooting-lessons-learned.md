# Troubleshooting & Lessons Learned

This section documents issues encountered during deployment and how they were resolved.

---

## 1️⃣ Kerberos Default Realm Error

### Problem

Running:

```
kinit administrator
```

Returned:

```
Configuration file does not specify default realm
```

### Root Cause

FreeBSD was using `/etc/krb5.conf`, but Samba generated its own Kerberos configuration file during domain provisioning:

```
/var/db/samba4/private/krb5.conf
```

The system was not automatically using the Samba-generated configuration.

### Resolution

Copied Samba's Kerberos configuration into the system location:

```
cp /var/db/samba4/private/krb5.conf /etc/krb5.conf
```

Validated with:

```
kinit administrator
klist
```

Kerberos ticket successfully issued.

---

## 2️⃣ DNS Configuration Required Repointing to Localhost

### Problem

After provisioning, DNS resolution needed to be handled by the Samba internal DNS service.

Initially:

```
nameserver 192.168.1.1
```

### Root Cause

Active Directory Domain Controllers must use themselves as DNS servers.  
External DNS cannot resolve AD SRV records.

### Resolution

Updated `/etc/resolv.conf`:

```
nameserver 127.0.0.1
search lab.local
```

Validated using:

```
host -t SRV _ldap._tcp.lab.local
```

Result confirmed proper SRV registration.

---

## 3️⃣ Service Management

Ensured Samba starts at boot:

```
sysrc samba_server_enable="YES"
service samba_server start
```

Validated status:

```
service samba_server status
```

---

## Key Takeaways

- Active Directory is DNS-dependent.
- Kerberos must use correct realm configuration.
- Samba provisioning generates critical configuration files.
- Domain Controllers should resolve themselves locally.
- SRV record validation is essential when troubleshooting AD.

This deployment reinforced the importance of:

- Identity infrastructure architecture
- Proper DNS design
- Understanding Kerberos flow
- Verifying services rather than assuming they work
