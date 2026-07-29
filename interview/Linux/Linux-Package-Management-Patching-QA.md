# Interview Q&A Bank: Linux Package Management & Patching

## Context: Project 9 (OS Patching Automation) — 500+ RHEL servers/month, 18-step zero-touch lifecycle, Ansible AAP

---

### Q1: How does dnf/yum work internally? Walk me through what happens when you run `dnf install httpd`.

**Project Reference:** Project 9 (Enterprise OS Patching Automation)
**Expected Depth:** Interviewer wants to hear about transaction system, dependency resolution algorithm (libsolv), GPG verification, and how this knowledge shaped the patching automation design.

**Answer:**

When you run `dnf install httpd`, here's the internal flow I deal with at scale:

1. **Metadata download/cache check:** dnf checks `/var/cache/dnf/` for repo metadata (repomd.xml). If expired (based on `metadata_expire` in repo config), it downloads fresh metadata from all enabled repos. This is why we run `dnf makecache` as a pre-step in our patching pipeline — we don't want 500 servers hitting the repo simultaneously during the actual patch window.

2. **Dependency resolution (libsolv):** dnf uses libsolv (SAT solver — boolean satisfiability problem). It reads the primary.xml metadata, builds a dependency tree for httpd — Requires, Provides, Conflicts, Obsoletes. It finds the best solution that satisfies ALL constraints. If httpd requires `apr >= 1.6`, libsolv finds the newest apr that satisfies this. If there's a conflict, it reports it before touching anything.

3. **Transaction building:** Once dependencies are resolved, dnf builds a transaction — the ordered list of RPM operations (install, upgrade, erase). This is atomic in intent — if step 5 of 8 fails, it rolls back steps 1-4 using the RPM transaction mechanism.

4. **GPG verification:** Before executing, each package's RPM header signature is verified against imported GPG keys in `/etc/pki/rpm-gpg/`. In our patching system, we enforce `gpgcheck=1` on all repos and pre-import only approved keys. A package with an unknown signature fails the transaction — critical for supply chain security.

5. **RPM execution:** The actual install — runs `%pre` scriptlets, copies files, runs `%post` scriptlets, updates the RPM database (`/var/lib/rpm/`).

6. **Transaction history:** Everything is logged in `/var/lib/dnf/history/` — who installed what, when, what changed. This is how `dnf history undo` works.

**Why this matters in Project 9:** Our automation leverages this by:
- Pre-caching metadata to avoid network timeouts during patch window
- Using `--assumeyes` only after pre-flight dry-run validates no conflicts
- Checking GPG key consistency across fleet (one server with missing key = failed batch)
- Using transaction history for automated rollback decisions

---

### Q2: What's the difference between `dnf update`, `dnf upgrade`, and `dnf distro-sync`? When do you use each?

**Project Reference:** Project 9 (Enterprise OS Patching Automation)
**Expected Depth:** Not just "they're aliases" — when distro-sync matters, how obsoletes are handled, and which one the patching system uses.

**Answer:**

In modern dnf, `update` and `upgrade` are functionally identical — both upgrade packages to the latest available version. The distinction is historical (yum had a subtle difference where `update` preserved obsoletes behavior differently). In dnf, `update` is just an alias for `upgrade`.

**The real distinction that matters in production is `distro-sync`:**

- **`dnf upgrade`:** Moves packages UP to the latest version. Never downgrades. If your server has `openssl-1.1.1k-7` and the repo has `1.1.1k-6` (because a bad patch was yanked and reverted), `upgrade` does NOTHING — it won't downgrade.

- **`dnf distro-sync`:** Synchronizes to whatever version the repo currently has — including DOWNGRADES. If the repo has a lower version, it'll downgrade your package to match.

**When I use each in Project 9:**

- **Regular monthly patching:** `dnf upgrade` — safe, predictable, only moves forward.
- **Emergency rollback scenario:** `dnf distro-sync` when we need to force a fleet back to a specific repo snapshot (after a bad CVE patch is retracted by Red Hat).
- **Repo pinning with Satellite/Pulp:** When using content views with date-pinned repos, `distro-sync` ensures servers match the repo state exactly — no drift, no "this server was patched 3 days later so it got a different version."

```yaml
# In our Ansible role (step 8):
- name: Apply OS patches
  dnf:
    name: "*"
    state: latest          # This maps to 'dnf upgrade'
    exclude: "{{ exclude_packages | join(',') }}"
```

We use `state: latest` (equivalent to upgrade) because our repos are controlled via Satellite content views. The repo itself is the gate — `distro-sync` isn't needed because we control what versions are available.

---

### Q3: How do you exclude packages from patching? What are the different methods and when do you use each?

**Project Reference:** Project 9 (Enterprise OS Patching Automation)
**Expected Depth:** Multiple exclusion methods, their precedence, and the real-world reasoning for each.

**Answer:**

There are 4 levels of package exclusion, and we use different ones for different reasons:

**1. dnf.conf global exclude (persistent, all operations):**
```ini
# /etc/dnf/dnf.conf
[main]
excludepkgs=kernel*,postgresql*
```
Use case: Packages that should NEVER be touched by any patching — managed by another team entirely.

**2. Repo-level exclude (per repository):**
```ini
# /etc/yum.repos.d/appstream.repo
[appstream]
exclude=docker-ce*,containerd*
```
Use case: Block specific packages from specific repos (e.g., don't pull Docker from OS repos when you have Docker's official repo).

**3. Command-line `--exclude` (per invocation):**
```bash
dnf upgrade --exclude=postgresql* --exclude=oracle*
```
Use case: Ad-hoc patching with temporary exclusions.

**4. dnf versionlock plugin (pin specific versions):**
```bash
dnf versionlock add postgresql-14.2-1.el8
```
Use case: Lock a package to an exact version permanently — it won't upgrade OR downgrade.

**In Project 9, we use method 3 via Ansible — config-driven:**
```yaml
# vars/main.yml — per server group
exclude_packages:
  - postgresql*     # DB team manages this
  - oracle*         # DBA managed
  - custom-app-*   # Application team's packages
  - docker-ce*     # Container runtime, separate lifecycle

# In the patching task:
- name: Apply OS patches
  dnf:
    name: "*"
    state: latest
    exclude: "{{ exclude_packages | join(',') }}"
```

**Why command-line over dnf.conf?** Because exclusions vary per server group. Database servers exclude different packages than web servers. Ansible variables give us per-host, per-group, or per-run flexibility. If we put it in dnf.conf, we'd need to manage that file across 500 servers and it's harder to override for emergency patches where you DO want to patch a normally-excluded package.

**Precedence (matters when troubleshooting):** dnf.conf excludes + repo excludes + command-line excludes + versionlock all stack. They're additive (union). A package excluded by ANY method is excluded.

---

### Q4: A bad patch went out. How do you rollback using `dnf history undo`? What are the limitations?

**Project Reference:** Project 9 (Enterprise OS Patching Automation)
**Expected Depth:** Practical rollback mechanics, when history undo works vs doesn't, and what Project 9 does instead.

**Answer:**

**The mechanism:**
```bash
# View transaction history
dnf history list
# ID | Command               | Date       | Altered
# 45 | upgrade               | 2024-01-15 | 47 packages

# See what transaction 45 did
dnf history info 45
# Shows: upgraded openssl from 1.1.1k-6 to 1.1.1k-7, etc.

# Undo it (reverse the transaction)
dnf history undo 45
# This DOWNGRADES those 47 packages back to their pre-transaction versions
```

**How it works internally:** dnf stores before/after versions for every transaction. `undo` creates a NEW transaction that reverses the changes — downgrades upgraded packages, removes newly installed packages, re-installs removed packages.

**Limitations (why we don't rely on it in Project 9):**

1. **Dependency hell:** If package A was upgraded as a dependency of package B, undoing B's install might leave A at a version that nothing else needs but nothing conflicts with — orphaned state.

2. **Scriptlet side effects:** RPM `%post` scripts may have made system changes (created users, modified configs). `undo` reverses the FILE changes but not the scriptlet side effects.

3. **Kernel rollback:** You can't downgrade a running kernel. You need to reboot into the old kernel (which is still installed if `installonly_limit` hasn't removed it).

4. **Shared library hell:** If 15 packages depend on a shared library that was upgraded, undoing one package might break the others that now expect the new library version.

5. **Doesn't work across reboots in all cases:** If the system rebooted (new kernel loaded), some state is irreversible via dnf alone.

**What Project 9 does instead:**
```yaml
# Create AMI snapshot BEFORE patching (step 8)
- name: Create pre-patch snapshot
  amazon.aws.ec2_ami:
    instance_id: "{{ instance_id.stdout }}"
    name: "pre-patch-{{ inventory_hostname }}-{{ ansible_date_time.date }}"
    wait: yes
```

Our rollback strategy: AMI snapshot → patch → validate (7 dimensions) → if FAIL, restore from AMI. This is a full system-level revert — guaranteed clean state regardless of dependency complexity. Takes <5 minutes vs potentially hours of debugging a partial `dnf history undo`.

We keep `dnf history` as a forensic/audit tool (what changed?) but not as the rollback mechanism.

---

### Q5: What RPM database queries do you use regularly? Explain `rpm -qa`, `-qi`, `-qf`, `-V` and real use cases.

**Project Reference:** Project 9 (Enterprise OS Patching Automation)
**Expected Depth:** Not just flags — how these are used in automation, compliance, and troubleshooting.

**Answer:**

RPM queries are my daily driver for investigating package state. The RPM database (`/var/lib/rpm/`) is the source of truth for what's installed.

**`rpm -qa` — Query All installed packages:**
```bash
rpm -qa                        # List everything (use for package inventory)
rpm -qa --last | head -20      # Recently installed/updated (post-incident investigation)
rpm -qa kernel                 # All installed kernel versions
rpm -qa --qf '%{NAME}-%{VERSION}-%{RELEASE}.%{ARCH}\n' | sort  # Clean format
```
**In Project 9:** We capture `rpm -qa --last` as part of pre/post baseline (step 4). The diff tells us exactly what changed during patching — attached as evidence to ServiceNow CR.

**`rpm -qi <package>` — Query Info (metadata):**
```bash
rpm -qi openssl
# Shows: version, release, install date, vendor, signature, description
```
**Use case:** Verifying vendor (is this from Red Hat or a third party?), checking install date for compliance ("was this patched within the SLA window?").

**`rpm -qf <file>` — Query which package owns a File:**
```bash
rpm -qf /etc/httpd/conf/httpd.conf
# Returns: httpd-2.4.51-7.el8.x86_64

rpm -qf /usr/bin/python3
# Returns: python3-3.8.13-1.el8.x86_64
```
**Use case:** "This config file got modified — which package owns it? Did a package update overwrite our customization?" Critical in our integrity check (step 11) when checksums don't match.

**`rpm -V <package>` — Verify package integrity:**
```bash
rpm -V httpd
# S.5....T.  c /etc/httpd/conf/httpd.conf
# Missing:   /etc/httpd/conf.d/ssl.conf
```
Output flags: `S`=size, `5`=MD5, `T`=mtime, `M`=mode, `U`=owner, `G`=group, `c`=config file

**This is gold for security:** After a suspected compromise, `rpm -Va` verifies ALL packages — shows every file that's been modified from its installed state. If `/usr/bin/ssh` shows a checksum mismatch and you didn't patch it — that binary has been tampered with.

**In Project 9 automation:**
```yaml
# Post-patch integrity check
- name: Verify critical packages unchanged
  shell: "rpm -V {{ item }}"
  loop:
    - httpd
    - openssh-server
    - openssl
  register: rpm_verify
  failed_when: false  # Capture output, evaluate separately

# Config files (c flag) being different is expected
# Binary files being different is a RED FLAG
```

**`rpm -ql <package>` — List all files in a package:**
```bash
rpm -ql httpd | grep conf
# /etc/httpd/conf/httpd.conf
# /etc/httpd/conf/magic
```
**Use case:** "What files does this package install? What will I lose if I remove it?"

---

### Q6: How do you check if a reboot is needed after patching? Explain `needs-restarting`.

**Project Reference:** Project 9 (Enterprise OS Patching Automation)
**Expected Depth:** The tool itself, the kernel comparison trick, and how Project 9 makes reboot decisions automatically.

**Answer:**

`needs-restarting` is from the `yum-utils` package (RHEL 7) or `dnf-utils` (RHEL 8+). It checks for two things:

**1. Does the system need a reboot? (`needs-restarting -r`)**
```bash
needs-restarting -r
# Exit code 0: No reboot needed
# Exit code 1: Reboot needed (kernel, glibc, systemd, or dbus updated)
echo $?
```

It checks if core packages were updated that can't be hot-reloaded: kernel, glibc, systemd, dbus, linux-firmware. These require a full reboot to take effect.

**2. Which services need restart? (`needs-restarting -s`)**
```bash
needs-restarting -s
# httpd.service
# sshd.service
# postfix.service
```

This shows services whose binaries/libraries were updated but the running process still uses the OLD version (loaded in memory). The fix is to restart those services.

**The kernel comparison trick (what we actually use in Project 9):**
```bash
# Running kernel
uname -r
# 4.18.0-477.21.1.el8_8.x86_64

# Latest installed kernel
rpm -q kernel --last | head -1 | awk '{print $1}' | sed 's/kernel-//'
# 4.18.0-477.27.1.el8_8.x86_64

# If they differ → reboot needed
```

**In Project 9, conditional reboot logic:**
```yaml
- name: Check if kernel was updated
  shell: "rpm -q kernel --last | head -1 | awk '{print $1}' | sed 's/kernel-//'"
  register: latest_kernel

- name: Reboot if kernel updated
  reboot:
    reboot_timeout: 600
    post_reboot_delay: 30
  when: latest_kernel.stdout != ansible_kernel
```

**Why conditional reboot matters:** Not every patching run includes a kernel update. Unnecessary reboots = unnecessary risk + downtime. In a fleet of 500 servers, avoiding 400 unnecessary reboots per cycle saves hours of maintenance window time and reduces the probability of a server not coming back up cleanly.

**Post-reboot verification:** After reboot, we verify `uname -r` matches the latest installed kernel — catches the case where GRUB defaults weren't updated properly.

---

### Q7: How does kernel package management work? Multiple kernels, grubby, default kernel selection.

**Project Reference:** Project 9 (Enterprise OS Patching Automation)
**Expected Depth:** installonly_limit, how RHEL keeps multiple kernels, grubby mechanics, and what can go wrong.

**Answer:**

Kernel packages are special in RPM — they're INSTALLED (not upgraded). This means multiple kernel versions coexist on disk simultaneously.

**`installonly_limit` (dnf.conf):**
```ini
[main]
installonly_limit=3   # Keep last 3 kernels installed
```
When you install a 4th kernel, the oldest is automatically removed. Default is 3 in RHEL. We set it to 3 in production — enough for rollback but doesn't waste /boot space.

**What's on disk with 3 kernels:**
```
/boot/vmlinuz-4.18.0-477.21.1.el8_8.x86_64
/boot/vmlinuz-4.18.0-477.27.1.el8_8.x86_64
/boot/vmlinuz-4.18.0-513.5.1.el8_9.x86_64   (newest)
/boot/initramfs-4.18.0-477.21.1.el8_8.x86_64.img
/boot/initramfs-4.18.0-477.27.1.el8_8.x86_64.img
/boot/initramfs-4.18.0-513.5.1.el8_9.x86_64.img
```

**`grubby` — managing boot entries:**
```bash
# Show default kernel
grubby --default-kernel
# /boot/vmlinuz-4.18.0-513.5.1.el8_9.x86_64

# List all available kernels
grubby --info=ALL

# Set a specific kernel as default (rollback scenario)
grubby --set-default=/boot/vmlinuz-4.18.0-477.27.1.el8_8.x86_64

# Add kernel parameter
grubby --update-kernel=ALL --args="audit=1"

# Remove kernel parameter
grubby --update-kernel=ALL --remove-args="quiet"
```

**RHEL 8+ uses BLS (Boot Loader Specification):**
Kernel entries are in `/boot/loader/entries/*.conf` — one file per kernel. `grubby` manipulates these files. GRUB reads them at boot time.

**What can go wrong (and has gone wrong in production):**

1. **`/boot` full:** If `/boot` is a small partition (500MB) and kernels accumulate, new kernel install fails. Fix: `dnf remove` old kernels, or increase `installonly_limit` cautiously.

2. **grub2-mkconfig not run:** On some RHEL 7 systems, you need to regenerate grub config. RHEL 8+ with BLS doesn't need this.

3. **Wrong default after patch:** Kernel installs and becomes default, but server has specific boot parameters for the old kernel that weren't propagated. Server boots with new kernel but wrong parameters (e.g., missing `crashkernel=auto`).

**In Project 9:**
```yaml
# Post-reboot verification
- name: Verify running correct kernel
  assert:
    that:
      - ansible_kernel == latest_kernel.stdout
    fail_msg: "CRITICAL: Running {{ ansible_kernel }} but expected {{ latest_kernel.stdout }}"
```

If the assertion fails — server booted into an old kernel (GRUB misconfigured). This triggers rollback investigation.

---

### Q8: How do you manage repositories? Explain yum.repos.d, priorities, and GPG key management.

**Project Reference:** Project 9 (Enterprise OS Patching Automation)
**Expected Depth:** Repo file anatomy, priority plugin, how GPG keys work in practice at fleet scale, and Satellite/Pulp patterns.

**Answer:**

**Repo file anatomy (`/etc/yum.repos.d/*.repo`):**
```ini
[rhel-8-baseos]
name=Red Hat Enterprise Linux 8 - BaseOS
baseurl=https://satellite.company.com/pulp/repos/prod/rhel8/baseos/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
sslverify=1
sslcacert=/etc/rhsm/ca/katello-server-ca.pem
metadata_expire=86400    # 24 hours
```

Key directives:
- `baseurl` vs `mirrorlist`: baseurl is a single URL; mirrorlist returns multiple mirrors (faster, resilient)
- `enabled=0/1`: can have a repo file present but disabled (enable per-command with `--enablerepo`)
- `sslverify` + `sslcacert`: verify the REPO SERVER's identity (not just package GPG)
- `metadata_expire`: how long before re-downloading repo metadata
- `cost`: lower cost = preferred source when same package available from multiple repos

**Priority plugin (`dnf-plugin-priorities`):**
```ini
[internal-repo]
priority=10    # Lower = higher priority (1 is highest)

[epel]
priority=99    # Use only if package not in internal repo
```

Without priorities, dnf takes the NEWEST version from ANY enabled repo. With priorities, it prefers packages from higher-priority repos even if a lower-priority repo has a newer version. Critical for: keeping internal/approved packages from being overridden by EPEL or third-party repos.

**GPG key management at scale:**
```bash
# Import a key
rpm --import https://satellite.company.com/RPM-GPG-KEY-redhat-release

# List imported keys
rpm -qa gpg-pubkey*

# Verify a key's fingerprint
rpm -qi gpg-pubkey-fd431d51-4ae0493b
```

**In Project 9 (fleet-scale repo management):**
```yaml
# Ansible role: repo-management
- name: Deploy repo files from template
  template:
    src: "{{ item }}.repo.j2"
    dest: "/etc/yum.repos.d/{{ item }}.repo"
    mode: '0644'
  loop:
    - rhel8-baseos
    - rhel8-appstream
    - company-internal

- name: Import GPG keys
  rpm_key:
    key: "{{ item }}"
    state: present
  loop:
    - /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
    - /etc/pki/rpm-gpg/RPM-GPG-KEY-company

- name: Remove unauthorized repos
  file:
    path: "/etc/yum.repos.d/{{ item }}"
    state: absent
  loop: "{{ unauthorized_repos }}"
```

**Satellite/Pulp pattern:** In enterprise environments, servers don't hit Red Hat CDN directly. They hit Satellite (or Pulp), which provides content views — date-pinned snapshots of repos. This means: every server gets the SAME package versions regardless of when it's patched. No "server A got patched Monday, server B got patched Friday with different packages" drift.

---

### Q9: Explain package cache management — `dnf clean`, `makecache`, and when/why you'd use each.

**Project Reference:** Project 9 (Enterprise OS Patching Automation)
**Expected Depth:** What's actually cached, when cache causes problems, and how the patching automation handles cache strategically.

**Answer:**

**What's cached (`/var/cache/dnf/`):**
- `repomd.xml` — repo metadata index
- `primary.xml.gz` — package list with dependencies
- `filelists.xml.gz` — file lists per package
- `*.rpm` — downloaded packages (if `keepcache=1`)
- `metalink/mirrorlist` responses

**`dnf clean` variants:**
```bash
dnf clean metadata    # Remove repo metadata (forces re-download on next operation)
dnf clean packages    # Remove cached .rpm files
dnf clean all         # Remove everything (metadata + packages + other cache)
dnf clean dbcache     # Remove generated SQLite database cache
```

**`dnf makecache`:**
```bash
dnf makecache         # Download and cache metadata for all enabled repos
dnf makecache --timer # Background refresh (used by dnf-makecache.timer)
```

**When cache causes problems:**

1. **Stale metadata:** Repo has new packages but server uses old cached metadata → "No package available" or gets old version. Fix: `dnf clean metadata` or wait for `metadata_expire`.

2. **Corrupt cache:** After disk issues or interrupted downloads — dnf throws XML parsing errors. Fix: `dnf clean all`.

3. **Disk space:** On servers with `keepcache=1`, cached RPMs accumulate. On a busy server, this can be GB of wasted space. Fix: `dnf clean packages` or set `keepcache=0`.

4. **Repository change:** After pointing to a new Satellite content view or changing repo URLs, old metadata doesn't match new repo. Fix: `dnf clean all && dnf makecache`.

**In Project 9 (strategic cache handling):**
```yaml
# Pre-patching: ensure fresh metadata (step in pre-flight)
- name: Clean stale metadata
  command: dnf clean metadata
  changed_when: false

- name: Rebuild cache from current repos
  command: dnf makecache
  register: cache_result
  retries: 3
  delay: 10
  until: cache_result.rc == 0
```

**Why retries?** In a fleet of 500 servers, if all hit the Satellite server simultaneously for metadata, some will get timeouts. The retry with delay spreads the load. We also stagger patching batches (serial: 20%) which naturally reduces cache-download thundering herd.

**`keepcache` setting:**
```ini
# /etc/dnf/dnf.conf
keepcache=0   # Default: don't keep downloaded RPMs after install
keepcache=1   # Keep RPMs (useful for: air-gapped environments, or rollback via reinstall)
```

We use `keepcache=0` in our fleet — saves disk space. For rollback, we rely on AMI snapshots, not cached RPMs.

---

### Q10: How does RPM handle config files during updates? Explain `.rpmnew` and `.rpmsave` behavior.

**Project Reference:** Project 9 (Enterprise OS Patching Automation)
**Expected Depth:** The `%config` vs `%config(noreplace)` distinction, the decision matrix, and how Project 9 detects/handles these cases.

**Answer:**

This is one of the most subtle and production-impactful behaviors in RPM. It determines whether YOUR custom configs survive a package update.

**RPM's config file handling depends on the spec file directive:**

**`%config(noreplace)` — the common case (e.g., httpd.conf):**
| Your file modified? | Package file changed? | Result |
|---|---|---|
| No | No | Nothing happens |
| No | Yes | Your file replaced with new version |
| Yes | No | Your file kept |
| **Yes** | **Yes** | **Your file KEPT, new version saved as `.rpmnew`** |

**`%config` (without noreplace) — aggressive replacement:**
| Your file modified? | Package file changed? | Result |
|---|---|---|
| No | No | Nothing happens |
| No | Yes | Your file replaced with new version |
| Yes | No | Your file kept |
| **Yes** | **Yes** | **Your file REPLACED, old saved as `.rpmsave`** |

**The critical difference:** With `noreplace`, your customization survives (RPM puts its new version aside as `.rpmnew`). Without `noreplace`, RPM OVERWRITES your customization (saving your old version as `.rpmsave`).

**How RPM detects "modified":** It compares the file's MD5 against what was recorded at install time (stored in RPM database). If they differ → file has been modified.

**Real-world impact I've seen:**
- OpenSSH updates: `/etc/ssh/sshd_config` is `%config(noreplace)`. Your custom config survives. New directives appear in `/etc/ssh/sshd_config.rpmnew`. You need to MERGE them manually.
- Some packages use `%config` — after update, your custom tuning is gone. Server starts with default config. This is how patching "breaks" applications.

**In Project 9 (integrity check, step 11):**
```yaml
# Detect if patching created .rpmnew or .rpmsave files
- name: Find .rpmnew and .rpmsave files
  find:
    paths:
      - /etc
      - /opt
    patterns:
      - "*.rpmnew"
      - "*.rpmsave"
    recurse: yes
  register: config_drift_files

- name: Alert if config drift detected
  debug:
    msg: "WARNING: {{ config_drift_files.files | map(attribute='path') | list }}"
  when: config_drift_files.matched > 0

# Critical configs verified via checksum (pre-computed baselines)
checksum_files:
  - { path: "/etc/httpd/conf/httpd.conf", expected: "a4f8b3c2d1e5..." }
  - { path: "/etc/ssh/sshd_config", expected: "b7c9d2e4f6a1..." }
```

**Best practice:** After patching, always check for `.rpmnew` files. They contain NEW configuration directives that may be important (security settings, new features). Don't just ignore them — schedule a review to merge relevant changes into your customized config.

**Prevention strategy:** We maintain checksums of 8 critical config files. Post-patch, if any checksum differs from our baseline, validation FAILS and triggers investigation. This catches both `.rpmsave` (config overwritten) AND subtle cases where RPM's merge behavior corrupted a config.

---
