# Linux Interview Q&A — Category 7: Security (Part 2 — sudoers, PKI, SSH Tunnels, umask, ACLs)

---

### Q22: Explain the sudoers file — syntax, NOPASSWD, and security implications. How do you manage it safely?

**Project Reference:** Project 9 (OS Patching — Ansible needs sudo), Project 2 (3-Tier AWS — EC2 instance access)
**Expected Depth:** Proper sudoers management, security best practices, understanding of privilege escalation risks

**Answer:**

**sudoers file location and editing:**
```bash
# NEVER edit directly — use visudo (validates syntax before saving)
visudo
# OR edit a drop-in file:
visudo -f /etc/sudoers.d/90-ansible

# Why visudo? A syntax error in sudoers = LOCKED OUT of sudo = locked out of the system
```

**Syntax format:**
```
who   where=(as_whom)   what
user  host=(runas)      commands

# Examples:
root        ALL=(ALL)       ALL
%wheel      ALL=(ALL)       ALL                    # Group wheel gets full sudo
siddharth   ALL=(ALL)       /usr/bin/systemctl, /usr/bin/journalctl  # Limited commands
ansible     ALL=(ALL)       NOPASSWD: ALL          # No password prompt (automation)
deploy      ALL=(root)      NOPASSWD: /usr/bin/systemctl restart myapp
```

**Field breakdown:**
- `user/group` — who gets the privilege (`%` prefix = group)
- `ALL` (first) — from which hosts (useful in NIS/LDAP environments)
- `(ALL)` — can run as which users (ALL = any user including root)
- `NOPASSWD:` — don't prompt for password
- `commands` — what they can run (full path required!)

**NOPASSWD security implications:**
```bash
# DANGEROUS — gives full root with no authentication:
ansible ALL=(ALL) NOPASSWD: ALL

# BETTER — limit to specific commands needed:
ansible ALL=(ALL) NOPASSWD: /usr/bin/yum, /usr/bin/dnf, /usr/bin/systemctl, /sbin/reboot

# BEST — use command aliases for organization:
Cmnd_Alias PATCHING = /usr/bin/yum, /usr/bin/dnf, /usr/bin/systemctl, /sbin/reboot
Cmnd_Alias MONITORING = /usr/bin/journalctl, /usr/bin/ss, /usr/bin/df
ansible ALL=(ALL) NOPASSWD: PATCHING, MONITORING
```

**Security best practices:**
1. **Use drop-in files** (`/etc/sudoers.d/`) — easier to manage, less risk of corrupting main file
2. **File naming convention:** `##-description` (## = priority number, no dots in filename!)
3. **Restrict commands:** Never give `NOPASSWD: ALL` unless absolutely necessary
4. **Deny dangerous commands:**
```bash
# Allow everything EXCEPT shell escapes:
deploy ALL=(ALL) NOPASSWD: ALL, !/bin/bash, !/bin/sh, !/usr/bin/su, !/usr/bin/vi
# But this is bypassable! (vi → :!/bin/bash). Better to whitelist.
```
5. **Log sudo usage:** All sudo commands are logged to `/var/log/secure` (RHEL) or `/var/log/auth.log` (Debian)
6. **Use `requiretty`** for service accounts (prevents background exploitation)

**In Project 9 (Ansible automation):**
```bash
# /etc/sudoers.d/90-ansible (deployed via Ansible itself during server provisioning)
Defaults:ansible !requiretty
ansible ALL=(ALL) NOPASSWD: ALL
```
We accept `NOPASSWD: ALL` for the ansible user because:
- It's a service account (no interactive login)
- SSH key-only authentication (no password exists)
- Key is stored in Ansible Vault / AAP credential store
- All commands are logged and auditable
- Alternative (limited commands) breaks when adding new automation

**Common gotchas:**
- Filename with `.` in `/etc/sudoers.d/` → silently ignored (e.g., `ansible.conf` won't load)
- `#includedir` in sudoers has a `#` — it's NOT a comment (this trips people up)
- Order matters: last matching rule wins

---

### Q23: Explain /etc/pki and certificate trust management on RHEL. How does a system know which CAs to trust?

**Project Reference:** Project 9 (OS Patching — CA bundle validation, /etc/pki integrity)
**Expected Depth:** RHEL-specific PKI structure, trust store management, practical troubleshooting

**Answer:**

**RHEL PKI directory structure:**
```
/etc/pki/
├── ca-trust/                    # System trust store management
│   ├── source/                  # Where YOU add custom CA certs
│   │   ├── anchors/            # Trusted CAs (add .pem/.crt here)
│   │   └── blacklist/          # Explicitly distrusted CAs
│   ├── extracted/              # Auto-generated from source/ (DON'T EDIT)
│   │   ├── pem/               # PEM bundles
│   │   ├── openssl/           # OpenSSL hash format
│   │   └── java/              # Java keystore (cacerts)
│   └── ca-bundle.crt → ../extracted/pem/tls-ca-bundle.pem
├── tls/
│   ├── certs/                  # System certificates
│   │   └── ca-bundle.crt      # The combined CA bundle (symlink)
│   ├── private/                # Private keys (700 permissions)
│   └── openssl.cnf            # OpenSSL configuration
├── rpm-gpg/                    # RPM signing keys
└── nssdb/                      # NSS database (Firefox, Chrome)
```

**How the trust chain works:**
```
Application (curl, python-requests, openssl)
  ↓ reads
/etc/pki/tls/certs/ca-bundle.crt (symlink)
  ↓ points to
/etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem (auto-generated)
  ↓ built from
/etc/pki/ca-trust/source/anchors/ (your custom CAs)
  + ca-certificates package (Mozilla's root store)
```

**Adding a custom CA (corporate/internal CA):**
```bash
# Step 1: Place cert in the right directory
cp corporate-ca.pem /etc/pki/ca-trust/source/anchors/

# Step 2: Update the consolidated trust store
update-ca-trust

# Step 3: Verify
openssl verify -CAfile /etc/pki/tls/certs/ca-bundle.crt internal-server.pem
# internal-server.pem: OK

# Also verify curl trusts it:
curl https://internal-service.corp.local  # Should work without -k
```

**Removing/Blacklisting a CA:**
```bash
# Explicitly distrust a CA (e.g., compromised CA):
cp compromised-ca.pem /etc/pki/ca-trust/source/blacklist/
update-ca-trust
# Now any cert signed by this CA will be REJECTED system-wide
```

**Why this matters for patching (Project 9):**
```yaml
- name: Verify CA bundle integrity post-patch
  ansible.builtin.stat:
    path: /etc/pki/tls/certs/ca-bundle.crt
    checksum_algorithm: sha256
  register: ca_bundle_stat

- name: Ensure custom corporate CAs still present after update
  ansible.builtin.shell: |
    grep -c "Corporate Internal CA" /etc/pki/tls/certs/ca-bundle.crt
  register: corp_ca_check
  failed_when: corp_ca_check.stdout | int < 1
  # Package update (ca-certificates) can remove custom additions if not in anchors/

- name: Regenerate trust store if ca-certificates was updated
  ansible.builtin.command: update-ca-trust
  when: "'ca-certificates' in updated_packages"
```

**Common production issues:**
1. `ca-certificates` package updated → custom CAs disappear (if placed in wrong directory)
2. Java apps use different trust store (`/etc/pki/java/cacerts`) — must update separately or use `update-ca-trust` which handles all formats
3. Python uses its own certifi package bundle — may not read system store (depends on how requests library is installed)
4. Container images have their OWN CA bundle — need to mount or bake in custom CAs

---

### Q24: How does SSH port forwarding work? Explain local, remote, and dynamic forwarding with use cases.

**Project Reference:** Project 2 (3-Tier AWS — accessing services in private subnets)
**Expected Depth:** All three types with practical scenarios, security implications

**Answer:**

**Local Port Forwarding (-L) — "Bring remote service to my machine"**

```
Use case: Access RDS database in private subnet from my laptop

ssh -L 5432:rds-instance.internal:5432 ec2-user@bastion

Laptop:5432 ──SSH tunnel──▶ Bastion ──▶ RDS:5432 (private subnet)

# Now connect locally:
psql -h localhost -p 5432 -U admin mydb
```

Data flow: `localhost:5432 → SSH encrypted → bastion → plaintext → RDS:5432`

**Real example in Project 2 (accessing Aurora in data subnet):**
```bash
# Access Aurora DB through bastion without exposing DB to internet:
ssh -L 3306:aurora-cluster.us-east-1.rds.amazonaws.com:3306 \
    -i project2.pem ec2-user@bastion.example.com -N

# In another terminal:
mysql -h 127.0.0.1 -P 3306 -u admin -p

# -N = no remote command (just forwarding)
```

**Remote Port Forwarding (-R) — "Expose my local service to remote network"**

```
Use case: Let remote server access a service on my laptop (webhook development)

ssh -R 8080:localhost:3000 user@remote-server

Remote-server:8080 ──SSH tunnel──▶ My laptop:3000

# Anyone accessing remote-server:8080 reaches my local dev server on :3000
```

Less common in DevOps, but useful for:
- Exposing local dev environment for testing
- Allowing CI server to reach a local service during testing

**Dynamic Port Forwarding (-D) — "SOCKS proxy through SSH"**

```
Use case: Browse internal network resources as if I'm inside the VPC

ssh -D 9090 ec2-user@bastion

# Configure browser/app to use SOCKS5 proxy: localhost:9090
# ALL traffic through the proxy goes via bastion
# Can access any internal resource the bastion can reach
```

```bash
# Use with curl:
curl --socks5 localhost:9090 http://internal-service.private:8080

# Use with any app via proxychains:
proxychains psql -h internal-db -U admin
```

**Comparison:**

| Type | Flag | Direction | Use case |
|------|------|-----------|----------|
| Local | `-L` | Remote → Local | Access remote DB/service locally |
| Remote | `-R` | Local → Remote | Expose local service to remote |
| Dynamic | `-D` | All traffic through tunnel | SOCKS proxy, browse internal network |

**SSH config for persistent tunnels (Project 2):**
```
# ~/.ssh/config
Host tunnel-rds
    HostName bastion.example.com
    User ec2-user
    IdentityFile ~/.ssh/project2.pem
    LocalForward 3306 aurora-cluster.internal:3306
    LocalForward 6379 redis.internal:6379
    ServerAliveInterval 60
    ServerAliveCountMax 3

# Usage: ssh tunnel-rds -N &
# Now localhost:3306 → Aurora, localhost:6379 → Redis
```

**Security implications:**
- SSH tunnels bypass network firewalls/security groups (traffic looks like SSH)
- Audit: `AllowTcpForwarding no` in sshd_config disables forwarding
- `GatewayPorts no` (default) prevents remote forwarding from binding to all interfaces
- Use `PermitOpen` to restrict which destinations can be forwarded to

---

### Q25: What is umask? How does it affect file creation? How do you set it properly for services?

**Project Reference:** Project 9 (OS Patching — file permission validation), Project 2 (3-Tier AWS — application file security)
**Expected Depth:** Umask calculation, per-service configuration, security implications

**Answer:**

**What umask does:**
Umask is a MASK that removes permissions from newly created files/directories. It's subtracted from the maximum default permissions.

```
Default permissions (before umask):
  Files:       666 (rw-rw-rw-)    ← No execute by default for files
  Directories: 777 (rwxrwxrwx)

With umask 022:
  Files:       666 - 022 = 644 (rw-r--r--)
  Directories: 777 - 022 = 755 (rwxr-xr-x)

With umask 077:
  Files:       666 - 077 = 600 (rw-------)
  Directories: 777 - 077 = 700 (rwx------)
```

**Common umask values:**

| umask | Files become | Dirs become | Use case |
|-------|-------------|-------------|----------|
| 022 | 644 (rw-r--r--) | 755 (rwxr-xr-x) | Default for most systems |
| 002 | 664 (rw-rw-r--) | 775 (rwxrwxr-x) | Shared group directories |
| 077 | 600 (rw-------) | 700 (rwx------) | Sensitive services (SSH keys, certs) |
| 027 | 640 (rw-r-----) | 750 (rwxr-x---) | Application files (owner + group read) |

**Where umask is set:**
```bash
# System-wide default:
/etc/profile           # Login shells
/etc/bashrc            # Non-login shells
/etc/login.defs        # UMASK setting (pam_umask reads this)

# Per-user:
~/.bashrc or ~/.profile

# Per-service (systemd):
[Service]
UMask=0027            # Files created by this service get 640/750
```

**Setting umask for services (Project 9 context):**
```ini
# /etc/systemd/system/myapp.service
[Service]
UMask=0027
# Application creates log files as 640 (owner rw, group r, others nothing)
# Application creates directories as 750
```

**Why this matters in production:**
```bash
# Scenario: Application creates /var/log/myapp/app.log
# With umask 022: -rw-r--r-- (world readable — might contain sensitive data!)
# With umask 027: -rw-r----- (only owner and group can read)

# Scenario: SSH daemon creates host keys
# UMask=0077 in sshd.service ensures: -rw------- (only root)
```

**Checking current umask:**
```bash
umask        # Shows current mask (e.g., 0022)
umask -S     # Symbolic: u=rwx,g=rx,o=rx (what IS allowed)
```

**Important nuance — umask is NOT subtraction for each bit:**
```
# It's a bitwise AND with the complement:
# actual = default AND (NOT umask)
# 666 AND (NOT 022) = 666 AND 755 = 644 ✓

# This means umask 033 on files:
# 666 AND (NOT 033) = 666 AND 744 = 644 (NOT 633!)
# The 'x' bit in umask doesn't matter for files (they don't get x by default)
```

**In Project 9 (post-patch validation):**
```yaml
- name: Verify critical services have secure umask
  ansible.builtin.shell: |
    grep -r "UMask" /etc/systemd/system/{{ item }}.service 2>/dev/null || \
    grep -r "UMask" /usr/lib/systemd/system/{{ item }}.service 2>/dev/null
  loop: "{{ critical_services }}"
  register: umask_check
  # Alert if UMask is not set (defaults to 022 — may be too permissive)
```

---

### Q26: When do you need File ACLs (getfacl/setfacl) beyond basic Unix permissions? Give practical examples.

**Project Reference:** Project 9 (OS Patching — complex permission requirements), Project 2 (3-Tier AWS — shared directories)
**Expected Depth:** Real use cases, syntax, interaction with standard permissions, gotchas

**Answer:**

**When standard permissions aren't enough:**

Standard Unix: ONE owner, ONE group, others. That's it.
Problem: What if multiple users/groups need DIFFERENT access to the same file?

**Scenario 1: Multiple groups need access (Project 2 — shared logs)**
```bash
# Standard approach (only one group owner):
chown app:developers /var/log/app/
# Now only 'developers' group has access

# But monitoring team also needs read access!
# Option A: Add monitoring users to developers group (security risk — oversharing)
# Option B: ACLs (precise control):

setfacl -m g:monitoring:rx /var/log/app/
setfacl -m g:monitoring:r /var/log/app/*.log

# Result:
getfacl /var/log/app/
# user::rwx
# group::rwx        (developers — original group)
# group:monitoring:r-x  (monitoring — ACL addition)
# other::---
# mask::rwx
```

**Scenario 2: Specific user exception**
```bash
# Web directory owned by www-data, but deploy user needs write access:
setfacl -m u:deploy:rwx /var/www/html/
setfacl -Rm u:deploy:rwx /var/www/html/  # -R = recursive
```

**Scenario 3: Default ACLs (new files inherit permissions)**
```bash
# Shared project directory — all new files should be group-writable:
setfacl -d -m g:devteam:rwx /shared/project/
#        ^^ default ACL — applies to NEWLY created files

# Verify:
getfacl /shared/project/
# default:user::rwx
# default:group::rwx
# default:group:devteam:rwx
# default:other::r-x
# default:mask::rwx
```

**Scenario 4: Ansible service account needs specific access (Project 9)**
```bash
# Ansible needs to read app configs but shouldn't own them:
setfacl -m u:ansible:r /etc/myapp/config.yml
setfacl -m u:ansible:rx /etc/myapp/
# Ansible can read config for validation without being root for that operation
```

**ACL commands reference:**
```bash
# Set ACL:
setfacl -m u:user:rwx /path          # Modify user ACL
setfacl -m g:group:rx /path          # Modify group ACL
setfacl -m o::r /path                # Modify others

# Remove specific ACL:
setfacl -x u:user /path              # Remove user's ACL entry

# Remove ALL ACLs:
setfacl -b /path                     # Strip all ACLs (back to standard perms)

# Recursive:
setfacl -Rm g:monitoring:rx /var/log/app/

# Default ACL (for directories — inheritance):
setfacl -d -m g:devteam:rwx /shared/

# Backup ACLs:
getfacl -R /important/dir > acl_backup.txt
# Restore:
setfacl --restore=acl_backup.txt
```

**The mask — effective permissions:**
```bash
getfacl /file
# user::rw-
# user:bob:rwx     #effective:rw-    ← Bob has rwx but mask limits to rw-
# group::r--
# mask::rw-        ← Maximum allowed for named users/groups
# other::---

# The mask acts as an upper limit on ACL entries
# chmod g=rx changes the MASK (not the group entry!) when ACLs exist
```

**Interaction with standard permissions:**
- `ls -l` shows `+` to indicate ACLs exist: `-rw-rwxr--+`
- `chmod` on a file with ACLs changes the mask (can inadvertently restrict ACL entries)
- `cp` preserves ACLs (with -a). `mv` preserves. `tar` needs `--acls` flag.
- Backup tools must explicitly support ACLs or they're lost

**When NOT to use ACLs:**
- If standard permissions (user/group/other) suffice — keep it simple
- If you need ACLs on many files — consider restructuring with group-based access
- If underlying filesystem doesn't support them (some NFS configs, FAT32)
- If tools in the pipeline don't preserve them (many backup/copy tools strip ACLs)

**In Project 9 (backup and validation):**
```yaml
- name: Backup ACLs before patching
  ansible.builtin.shell: |
    getfacl -R /etc/myapp > /var/backup/acls-pre-patch.txt
  
- name: Restore ACLs if patching overwrote configs
  ansible.builtin.shell: |
    setfacl --restore=/var/backup/acls-pre-patch.txt
  when: acls_changed | bool
```

**Filesystem support:**
- ext4: ✅ Full ACL support (default)
- XFS: ✅ Full ACL support (default)
- tmpfs: ✅ Supported
- NFS: Depends on server config (NFSv4 ACLs differ from POSIX ACLs)
- OverlayFS: ✅ Supported (Docker containers can use ACLs within their layer)
