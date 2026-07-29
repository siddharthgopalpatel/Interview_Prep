# Linux Interview Q&A — Category 7: Security (SELinux, Permissions, SSH, Certificates)

---

### Q17: Deep dive into Linux file permissions — rwx, setuid, setgid, sticky bit. When do special permissions matter?

**Project Reference:** Project 9 (OS Patching — file integrity checks), Project 2 (3-Tier AWS — EC2 file permissions)
**Expected Depth:** Beyond basic rwx — understanding special bits, numeric representation, security implications

**Answer:**

**Standard permissions (rwx):**
```
-rwxr-xr-- 1 root devops 4096 Jul 28 10:00 deploy.sh
│├──┤├──┤├──┤
│ │    │    └── Others: read only (4)
│ │    └────── Group (devops): read + execute (5)
│ └─────────── Owner (root): read + write + execute (7)
└──────────────File type (- = file, d = dir, l = link)

Numeric: 754
```

**For directories, permissions mean different things:**
- `r` = can list contents (ls)
- `w` = can create/delete files IN this directory
- `x` = can traverse (cd into, access files if you know the name)

**Special permissions (the 4th octet):**

**1. SetUID (4000) — Run as file OWNER:**
```bash
ls -la /usr/bin/passwd
-rwsr-xr-x 1 root root 68208 /usr/bin/passwd
   ^── 's' in owner execute position = SetUID

# Why: passwd needs to write to /etc/shadow (root-owned)
# Any user running passwd temporarily executes AS ROOT
# Security risk: SetUID root binaries are prime attack targets
```

**2. SetGID (2000) — Run as file GROUP / Inherit directory group:**
```bash
# On files: execute as the file's group
ls -la /usr/bin/write
-rwxr-sr-x 1 root tty 20112 /usr/bin/write
       ^── 's' in group execute position

# On directories (MORE COMMON): new files inherit the directory's group
chmod g+s /shared/project/
# Now all files created here get group 'devops' regardless of who creates them
# Critical for shared project directories
```

**3. Sticky bit (1000) — Only owner can delete:**
```bash
ls -ld /tmp
drwxrwxrwt 15 root root 4096 /tmp
          ^── 't' in others execute position

# Anyone can create files in /tmp (rwx for others)
# But ONLY the file owner (or root) can delete their own files
# Without sticky bit: anyone could delete anyone's temp files
```

**Security implications in production:**

```bash
# Find all SetUID binaries on the system (audit!)
find / -perm -4000 -type f 2>/dev/null
# Every SetUID binary is a potential privilege escalation vector

# Find world-writable files (security risk)
find / -perm -002 -type f 2>/dev/null

# Find files with no owner (orphaned — possibly from deleted accounts)
find / -nouser -o -nogroup 2>/dev/null
```

**In Project 9 (integrity checks):**
```yaml
- name: Verify critical file permissions haven't changed post-patch
  ansible.builtin.stat:
    path: "{{ item.path }}"
  register: file_perms
  loop:
    - { path: "/etc/shadow", mode: "0000" }
    - { path: "/etc/ssh/sshd_config", mode: "0600" }
    - { path: "/usr/bin/sudo", mode: "4111" }
  failed_when: file_perms.stat.mode != item.mode
```

**Numeric calculation:**
```
Special: SetUID=4, SetGID=2, Sticky=1
Regular: r=4, w=2, x=1

chmod 4755 = SetUID + rwx(owner) + rx(group) + rx(others)
chmod 2775 = SetGID + rwx(owner) + rwx(group) + rx(others)
chmod 1777 = Sticky + rwx(all)
```

---

### Q18: Explain SELinux — enforcing vs permissive. How do you troubleshoot AVC denials in production?

**Project Reference:** Project 9 (OS Patching — manages SELinux state, validates post-patch)
**Expected Depth:** Practical SELinux troubleshooting, not just "I disable it"

**Answer:**

**SELinux (Security-Enhanced Linux) — Mandatory Access Control:**
Even if DAC (file permissions) allows access, SELinux can DENY it based on security policies. Defense-in-depth — if an attacker gains `root`, SELinux still restricts what they can do.

**Modes:**

| Mode | Behavior | Use case |
|------|----------|----------|
| **Enforcing** | Denies AND logs violations | Production (always) |
| **Permissive** | Logs but ALLOWS violations | Troubleshooting, policy development |
| **Disabled** | No SELinux at all | ❌ Never in production (requires reboot to re-enable + full relabel) |

```bash
# Check current mode
getenforce       # Returns: Enforcing/Permissive/Disabled
sestatus         # Full status including policy type

# Temporarily switch (no reboot, reverts on reboot):
setenforce 0     # Permissive (for troubleshooting)
setenforce 1     # Back to Enforcing

# Permanently set:
vi /etc/selinux/config
SELINUX=enforcing
```

**SELinux concepts (quick):**
- **Context/Label:** Every file, process, port has a label: `user:role:type:level`
- **Type Enforcement:** Rules define which process types can access which file types
- **Example:** `httpd_t` (Apache process) can read `httpd_sys_content_t` (web files) but NOT `user_home_t` (home directories)

**Troubleshooting AVC denials (the real skill):**

```bash
# Step 1: Find the denial
ausearch -m avc -ts recent
# OR
grep "denied" /var/log/audit/audit.log | tail -20
# OR (most useful)
sealert -a /var/log/audit/audit.log

# Example denial:
# type=AVC msg=audit(1690000000.123:456): avc: denied { read } for
#   pid=1234 comm="nginx" name="ssl.key" dev="sda1" ino=5678
#   scontext=system_u:system_r:httpd_t:s0
#   tcontext=unconfined_u:object_r:user_home_t:s0 tclass=file

# Translation: nginx (httpd_t) tried to read a file labeled user_home_t — DENIED
```

**Step 2: Diagnose and fix:**

```bash
# Option A: Fix the file label (MOST COMMON — file is mislabeled)
# Check current label:
ls -Z /etc/nginx/ssl/ssl.key
# unconfined_u:object_r:user_home_t:s0  ← WRONG! Should be cert_t or httpd_config_t

# Fix:
restorecon -Rv /etc/nginx/ssl/    # Restores default labels from policy
# OR set specific context:
semanage fcontext -a -t httpd_sys_content_t "/etc/nginx/ssl(/.*)?"
restorecon -Rv /etc/nginx/ssl/

# Option B: Allow non-standard port
# Nginx listening on 8443 but SELinux only allows httpd_t on standard ports:
semanage port -l | grep http_port_t    # See allowed ports
semanage port -a -t http_port_t -p tcp 8443  # Add port

# Option C: Set a boolean (pre-defined policy toggles)
# Nginx needs to connect to backend:
setsebool -P httpd_can_network_connect on
# List available booleans:
getsebool -a | grep httpd

# Option D: Create custom policy (LAST RESORT)
# Generate policy from denial:
ausearch -m avc -ts recent | audit2allow -M my_nginx_fix
semodule -i my_nginx_fix.pp
```

**In Project 9 (OS Patching):**
```yaml
- name: Verify SELinux is enforcing post-patch
  ansible.builtin.command: getenforce
  register: selinux_state
  failed_when: selinux_state.stdout != "Enforcing"

- name: Check for AVC denials after patching
  ansible.builtin.shell: |
    ausearch -m avc --start recent -i 2>/dev/null | grep -c "denied" || echo 0
  register: avc_count
  # Alert if new denials appear post-patch (patching broke SELinux labels)

- name: Restore file contexts if packages updated config files
  ansible.builtin.command: restorecon -Rv /etc
  when: packages_updated | bool
```

**Key insight:** "I disable SELinux" is a red flag in interviews. The correct answer is: keep it enforcing, learn to troubleshoot denials (90% are file context issues fixed with `restorecon`), use booleans for common needs, and only write custom policy as a last resort.

---

### Q19: Explain SSH key-based authentication — how it works under the hood. What's agent forwarding and ProxyJump?

**Project Reference:** Project 2 (3-Tier AWS — SSH access to private EC2), Project 9 (OS Patching — SSH config backup)
**Expected Depth:** Cryptographic handshake understanding, practical multi-hop scenarios

**Answer:**

**SSH key authentication flow:**

```
1. Client connects to server on port 22
2. Server sends its HOST key (client verifies against ~/.ssh/known_hosts)
3. Key exchange (Diffie-Hellman) establishes encrypted channel
4. Client says "I want to authenticate as user X with public key Y"
5. Server checks if public key Y exists in ~X/.ssh/authorized_keys
6. Server generates random challenge → encrypts with public key → sends to client
7. Client decrypts challenge with PRIVATE key → sends back hash
8. Server verifies hash → authentication successful
```

**Key point:** Private key NEVER leaves the client machine. Server only has the public key.

**Key types (recommendation order):**
```bash
ssh-keygen -t ed25519 -C "siddharth@work"  # Best: small, fast, secure
ssh-keygen -t rsa -b 4096                   # Compatible: larger but universally supported
# Never: DSA (deprecated), ECDSA (questionable curves)
```

**SSH Agent — avoid typing passphrase repeatedly:**
```bash
eval $(ssh-agent)           # Start agent
ssh-add ~/.ssh/id_ed25519   # Add key (prompts for passphrase once)
# Now all SSH connections use the agent — no passphrase prompt
```

**Agent Forwarding (-A) — use your local keys on remote servers:**
```
Scenario: Laptop → Bastion → Private server (needs to pull from Git)

Without agent forwarding:
  - Must copy private key to bastion (SECURITY RISK!)
  - Or create separate key for bastion

With agent forwarding:
  Laptop (has key) → SSH to Bastion (-A flag) → Bastion uses YOUR key via agent → Git pull works
```

```bash
ssh -A user@bastion
# Now on bastion, your local key is available via agent socket
git clone git@github.com:company/repo.git  # Works using YOUR key
```

**Security risk of -A:** Any root user on the bastion can hijack your agent socket. Use only on trusted hosts. Better alternative: ProxyJump.

**ProxyJump (-J) — the modern solution:**
```bash
# Single command: connect through bastion to private server
ssh -J bastion.example.com private-server.internal

# In ~/.ssh/config (permanent):
Host private-*
    ProxyJump bastion.example.com
    User ec2-user
    IdentityFile ~/.ssh/id_ed25519

Host bastion.example.com
    User ec2-user
    IdentityFile ~/.ssh/id_ed25519
```

**Why ProxyJump is better than Agent Forwarding:**
- TCP connection is proxied — your key NEVER touches the bastion
- No agent socket on bastion to hijack
- Bastion only sees encrypted tunnel traffic
- Works with `scp` and `rsync` transparently

**In Project 2 (3-Tier AWS architecture):**
```
Internet → Bastion (public subnet) → App servers (private subnet)

~/.ssh/config:
Host bastion
    HostName bastion.example.com
    User ec2-user
    IdentityFile ~/.ssh/project2.pem

Host app-*
    ProxyJump bastion
    User ec2-user
    IdentityFile ~/.ssh/project2.pem
```

**In Project 9 (SSH config backup):**
```yaml
- name: Backup SSH configuration before patching
  ansible.builtin.copy:
    src: /etc/ssh/sshd_config
    dest: /var/backup/sshd_config.pre-patch
    remote_src: yes

- name: Verify SSH access after patching
  ansible.builtin.wait_for:
    port: 22
    host: "{{ inventory_hostname }}"
    timeout: 60
  delegate_to: localhost
```

---

### Q20: How do you validate certificates on Linux using openssl? Check cert, key, chain validity.

**Project Reference:** Project 9 (OS Patching — certificate validation post-patch)
**Expected Depth:** Practical openssl commands for production cert troubleshooting

**Answer:**

**The certificate chain:**
```
Root CA (trusted, in OS trust store)
  └── Intermediate CA (signed by Root)
       └── Server Certificate (signed by Intermediate)
            └── Private Key (on server, never shared)
```

**Essential openssl commands for production:**

**1. View certificate details:**
```bash
# From a file:
openssl x509 -in cert.pem -noout -text
openssl x509 -in cert.pem -noout -dates    # Just expiry dates
openssl x509 -in cert.pem -noout -subject -issuer  # Who issued it

# From a live server:
openssl s_client -connect example.com:443 -servername example.com < /dev/null 2>/dev/null | \
  openssl x509 -noout -dates
```

**2. Check certificate expiry (critical for automation):**
```bash
# Days until expiry:
openssl x509 -in cert.pem -noout -enddate
# notAfter=Oct 15 12:00:00 2026 GMT

# Check if cert expires within 30 days:
openssl x509 -in cert.pem -checkend 2592000  # 30 days in seconds
# Exit code 0 = still valid, 1 = expires within 30 days
```

**3. Verify certificate matches private key:**
```bash
# The modulus must match:
openssl x509 -in cert.pem -noout -modulus | md5sum
openssl rsa -in key.pem -noout -modulus | md5sum
# If md5sums match → cert and key are a pair

# Simpler (compare directly):
diff <(openssl x509 -in cert.pem -noout -modulus) \
     <(openssl rsa -in key.pem -noout -modulus)
# No output = they match
```

**4. Verify the full chain:**
```bash
# Verify cert against CA bundle:
openssl verify -CAfile ca-bundle.crt cert.pem
# cert.pem: OK

# Check chain order (common misconfiguration):
openssl s_client -connect example.com:443 -servername example.com < /dev/null 2>/dev/null
# Look at certificate chain:
# 0 s:CN=example.com          ← Server cert
# 1 s:CN=Let's Encrypt R3     ← Intermediate
# 2 s:CN=ISRG Root X1         ← Root (optional, sometimes omitted)
```

**5. Test SSL connection end-to-end:**
```bash
# Full connection test with protocol/cipher info:
openssl s_client -connect example.com:443 -servername example.com

# Test specific TLS version:
openssl s_client -connect example.com:443 -tls1_2
openssl s_client -connect example.com:443 -tls1_3

# Check which ciphers are supported:
nmap --script ssl-enum-ciphers -p 443 example.com
```

**In Project 9 (automated certificate validation post-patch):**
```yaml
- name: Validate certificate is not expired
  ansible.builtin.shell: |
    openssl x509 -in {{ cert_path }} -checkend 604800  # 7 days
  register: cert_expiry
  failed_when: cert_expiry.rc != 0
  # Fails if cert expires within 7 days

- name: Validate cert-key pair match
  ansible.builtin.shell: |
    CERT_MOD=$(openssl x509 -in {{ cert_path }} -noout -modulus | md5sum | awk '{print $1}')
    KEY_MOD=$(openssl rsa -in {{ key_path }} -noout -modulus | md5sum | awk '{print $1}')
    [ "$CERT_MOD" = "$KEY_MOD" ]
  register: pair_check
  failed_when: pair_check.rc != 0

- name: Validate full certificate chain
  ansible.builtin.shell: |
    openssl verify -CAfile {{ ca_bundle_path }} {{ cert_path }}
  register: chain_check
  failed_when: "'OK' not in chain_check.stdout"

- name: Check certificate file permissions
  ansible.builtin.stat:
    path: "{{ key_path }}"
  register: key_stat
  failed_when: key_stat.stat.mode != "0600"
  # Private key must be readable only by owner
```

**Common production issues:**
- Certificate expired → service fails to start or clients reject connection
- Key/cert mismatch → service fails to start ("key values mismatch")
- Incomplete chain → works in browsers (they fetch intermediate) but fails for API clients/curl
- Wrong permissions on key → service can't read key file
- Package update overwrites CA bundle → custom certs no longer trusted

---

### Q21: How do you find and fix "permission denied" errors systematically on Linux?

**Project Reference:** Project 9 (OS Patching — post-patch validation), Project 3 (EKS — container permission issues)
**Expected Depth:** Systematic debugging methodology, not just "chmod 777"

**Answer:**

**Systematic approach (check in this order):**

**Layer 1: Standard Unix permissions (DAC)**
```bash
# Check file permissions and ownership:
ls -la /path/to/file
# -rw-r----- 1 root ssl-cert 1234 cert.key

# Who am I running as?
id
# uid=1000(app) gid=1000(app) groups=1000(app),33(www-data)

# Does user/group have access to EVERY directory in the path?
namei -l /path/to/file
# dr-xr-xr-x root root  /
# drwxr-xr-x root root  path
# drwxr-x--- root ssl-cert  to    ← HERE! 'app' not in ssl-cert group
# -rw-r----- root ssl-cert  file
```

**Layer 2: SELinux (MAC)**
```bash
# Check if SELinux is blocking:
ausearch -m avc -ts recent | grep denied
# OR
grep "denied" /var/log/audit/audit.log | tail

# Quick test: temporarily set permissive
setenforce 0
# Try operation again — if it works, SELinux was blocking
setenforce 1
# Fix: restorecon, setsebool, or semanage fcontext
```

**Layer 3: ACLs (extended permissions)**
```bash
# Check for ACLs (+ sign in ls output):
ls -la /path/to/file
# -rw-r-----+ 1 root root 1234 file  ← '+' means ACL exists

getfacl /path/to/file
# user::rw-
# user:app:---     ← EXPLICIT DENY for 'app' user!
# group::r--
# mask::r--
# other::---
```

**Layer 4: Mount options**
```bash
# Check if filesystem is mounted with restrictive options:
mount | grep /path
# /dev/sda2 on /tmp type ext4 (rw,nosuid,noexec)
#                                        ^^^^^^^ can't execute files!

# Also check for read-only:
mount | grep " ro,"
```

**Layer 5: Capabilities (fine-grained root powers)**
```bash
# Process needs to bind to port 80 but isn't root:
# Error: permission denied (binding to port < 1024)
# Fix: grant capability instead of running as root:
setcap cap_net_bind_service=+ep /usr/bin/myapp

# Check capabilities:
getcap /usr/bin/myapp
```

**Layer 6: Namespace/Container isolation**
```bash
# In containers (Project 3):
# Process runs as UID 1000 inside container
# But file on mounted volume is owned by root
# Fix: securityContext in K8s:
spec:
  securityContext:
    fsGroup: 1000  # Changes group ownership of mounted volumes
    runAsUser: 1000
```

**Decision tree:**
```
Permission Denied
  ├── ls -la → wrong owner/permissions? → chmod/chown
  ├── namei -l → parent directory blocks? → fix parent perms
  ├── getenforce → Enforcing? → check ausearch for AVC
  ├── getfacl → explicit deny? → setfacl to fix
  ├── mount → noexec/nosuid/ro? → remount or move file
  ├── getcap → missing capability? → setcap
  └── container? → check securityContext, user mapping, volume ownership
```

**In Project 9 (post-patch):**
Patching can reset permissions (package update overwrites config file with default perms). Our integrity check validates critical file permissions haven't changed:
```yaml
- name: Verify application can read its config after patching
  ansible.builtin.command: |
    sudo -u {{ app_user }} test -r {{ app_config_path }}
  register: read_check
  failed_when: read_check.rc != 0
```
