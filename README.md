# FreeBSD Active Directory Domain Controller (Samba)

This project documents the deployment of a fully functional Active Directory Domain Controller using Samba on FreeBSD.

The objective was to build a Windows-compatible AD environment using open-source tooling while following proper infrastructure practices.

---

## 🖥 Environment

- Hypervisor: Proxmox VE
- OS: FreeBSD 15.0-RELEASE-p3
- Filesystem: ZFS
- Hostname: dc1.lab.local
- Domain (Realm): LAB.LOCAL
- NetBIOS Domain: LAB
- IP Address: 192.168.1.10
- DNS Backend: SAMBA_INTERNAL
- Samba Version: 4.19

---

## ⚙️ Installation & Provisioning

### 1️⃣ Install Samba

```
pkg update
pkg install samba419
```

---

### 2️⃣ Provision Active Directory Domain

```
samba-tool domain provision \
  --use-rfc2307 \
  --realm=LAB.LOCAL \
  --domain=LAB \
  --server-role=dc \
  --dns-backend=SAMBA_INTERNAL
```

This command performed the following:

- Created new AD forest
- Configured LDAP directory
- Generated Kerberos KDC configuration
- Configured internal DNS
- Created SYSVOL + NETLOGON shares
- Generated domain SID
- Established Administrator account

---

### 3️⃣ Enable and Start Samba

```
sysrc samba_server_enable="YES"
service samba_server start
```

---

### 4️⃣ Kerberos Configuration

Samba-generated Kerberos configuration was copied to the system:

```
cp /var/db/samba4/private/krb5.conf /etc/krb5.conf
```

Validation:

```
kinit administrator
klist
```

Kerberos ticket successfully issued for:

```
administrator@LAB.LOCAL
```

---

## 🧪 DNS Validation

---

## 🔄 DNS Reconfiguration (Critical Step)

After domain provisioning, the DNS resolver was changed to use the local Samba DNS service.

Initial temporary resolver:

```
nameserver 192.168.1.1
```

Updated resolver configuration:

```
nameserver 127.0.0.1
search lab.local
```

This ensures the Domain Controller resolves:

- Its own SRV records
- LDAP services
- Kerberos services
- AD-integrated DNS entries

Active Directory Domain Controllers must use themselves as DNS servers.

SRV record validation:

```
host -t SRV _ldap._tcp.lab.local
```

Result confirmed LDAP service registration via internal DNS.

---

## 🚀 Next Steps

- Join Windows 10 client to LAB domain
- Create domain users
- Test Group Policy
- Validate SYSVOL access
- Implement AD-integrated file shares

---

## 🧠 Skills Demonstrated

- FreeBSD system administration (15.0-RELEASE-p3)
- ZFS configuration
- Samba AD provisioning
- DNS troubleshooting
- Kerberos configuration
- Service management via rc.d
- Active Directory architecture
