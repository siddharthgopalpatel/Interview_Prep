# Linux Interview Q&A Bank — Part 4 (Categories 8-11)
# Candidate: Siddharth Patel | 12 YOE | Senior DevOps & Cloud Engineer

---

## CATEGORY 8: BOOT PROCESS & TROUBLESHOOTING (6 Questions)

---

### Q1: Walk me through the complete Linux boot process from power-on to a running application.
**Project Reference:** Project 8 (Multi-Region HA & Disaster Recovery)
**Expected Depth:** Full chain with emphasis on what can go wrong at each stage, especially in cloud (EC2) context

**Answer:**

The Linux boot process follows this sequence:

**1. BIOS/UEFI (Firmware stage):**
- POST (Power-On Self Test) — verifies hardware (RAM, CPU, peripherals)
- In EC2: This is handled by the hypervisor (Nitro). You never interact with it, but understanding it explains why "instance stuck in pending" sometimes means hardware issues
- UEFI (modern) maintains a boot order stored in NVRAM. Locates the EFI System Partition (ESP) and loads the bootloader
- BIOS (legacy) reads the MBR (first 512 bytes of disk) which contains the stage 1 bootloader

**2. GRUB2 (Bootloader stage):**
- GRUB loads from /boot/efi (UEFI) or MBR → /boot/grub2 (BIOS)
- Reads /boot/grub2/grub.cfg — lists available kernels, their parameters, and initramfs locations
- Presents menu (timeout 5s typically, 0s in cloud for fastest boot)
- Loads the selected kernel image (vmlinuz) and initramfs into memory
- Passes kernel command line parameters (root=, console=, crashkernel=, etc.)
- In EC2: GRUB config is baked into the AMI. Wrong GRUB config = instance won't boot = need to mount volume on rescue instance

**3. Kernel initialization:**
- Kernel decompresses itself, sets up memory management, initializes CPU
- Detects hardware, loads compiled-in drivers
- Mounts initramfs as temporary root filesystem (/)
- In EC2: Kernel needs NVMe drivers and ENA network drivers. Missing these = instance can't see its own disk

**4. initramfs (Initial RAM Filesystem):**
- Contains minimal userspace: udev, systemd (or init), essential drivers
- Loads necessary kernel modules (storage drivers, filesystem drivers, LVM, RAID, dm-crypt)
- Finds and mounts the real root filesystem
- Executes pivot_root or switch_root to transition to the real root
- Critical: If root is on LVM, encrypted, or NFS — initramfs MUST contain those drivers

**5. systemd (PID 1):**
- Takes over as PID 1 after switch_root
- Reads default target: `systemctl get-default` (typically multi-user.target for servers)
- Starts units in dependency order (parallelized where possible)
- Mount filesystems (from /etc/fstab), start networking, start services
- Reaches the target (equivalent of old runlevels)

**6. cloud-init (Cloud-specific, after systemd):**
- Runs as systemd services at specific boot stages
- Configures networking, hostname, SSH keys, user-data scripts
- This is where your instance becomes "yours" vs a generic AMI

**7. Application starts:**
- Either via systemd unit (enabled service) or launched by user-data/cloud-init

**In Project 8 context:** When DR failover launches instances from AMI in us-west-2, this entire chain executes. The AMI bakes steps 1-5. cloud-init handles instance-specific config (region-aware endpoints, DR-specific parameters). If any step fails, the instance never passes ALB health check and ASG replaces it.

---

### Q2: What is cloud-init, what are its execution stages, and how do you debug failures?
**Project Reference:** Project 8 (Multi-Region HA & DR) / Project 2 (3-Tier AWS Architecture)
**Expected Depth:** Stages, ordering, real debugging experience, common failure modes

**Answer:**

**What cloud-init is:**
cloud-init is the industry-standard multi-distribution method for cross-platform cloud instance initialization. It runs on first boot (and optionally subsequent boots) to configure an instance from metadata/user-data provided by the cloud platform.

**Five execution stages (in order):**

| Stage | systemd Unit | When | What It Does |
|-------|-------------|------|--------------|
| 1. Generator | cloud-init-generator | Very early boot | Determines if cloud-init should run at all (checks /etc/cloud/cloud-init.disabled) |
| 2. Local | cloud-init-local.service | Before networking | Applies networking config from datasource (DHCP/static). Must run before network is up |
| 3. Network | cloud-init.service | After network is up | Fetches metadata from IMDS (169.254.169.254), sets hostname, SSH keys, mounts |
| 4. Config | cloud-config.service | After network stage | Runs config modules: package installation, write_files, runcmd prep, NTP config |
| 5. Final | cloud-final.service | Last (near login) | Runs user-data scripts, runcmd, final modules, phone-home |

**How we use it in Project 8:**
- AMI has base OS + application binaries baked in
- cloud-init user-data configures region-specific endpoints (Aurora endpoint differs per region)
- Sets environment variables for DR awareness
- Registers instance with target group
- In Project 2: user-data bootstraps the application tier (pulls config from Parameter Store, starts the app)

**Debugging cloud-init failures:**

```bash
# 1. Check cloud-init status (did it finish? did it error?)
cloud-init status --long
# Shows: status, any errors, timestamps

# 2. Main log file — all stages, all output
cat /var/log/cloud-init.log

# 3. Output log — stdout/stderr from user-data scripts
cat /var/log/cloud-init-output.log

# 4. Check what datasource was used
cloud-init query ds

# 5. Check what user-data was received (verify it arrived)
cloud-init query userdata

# 6. Re-run specific modules for testing
cloud-init single --name runcmd --frequency always

# 7. Analyze boot — shows timing per stage
cloud-init analyze show

# 8. Check if cloud-init ran at all
systemctl status cloud-init.service cloud-config.service cloud-final.service
```

**Common failure modes I've encountered:**
1. **User-data script missing shebang** (`#!/bin/bash`) — silently fails
2. **Script exceeds 16KB limit** — need to use multipart MIME or include directive
3. **Script depends on network but runs in local stage** — use runcmd (runs in final stage)
4. **Instance metadata service (IMDS) unreachable** — IMDSv2 token issue, or iptables blocking 169.254.169.254
5. **cloud-init ran on previous boot, won't re-run** — `/var/lib/cloud/instance/sem/` semaphore files prevent re-execution. Delete `/var/lib/cloud/instances/` to force re-run
6. **YAML syntax error in cloud-config** — entire cloud-init fails silently. Always validate with `cloud-init schema --config-file`

---

### Q3: An EC2 instance boots with the wrong kernel after patching. How do you fix it?
**Project Reference:** Project 9 (Enterprise OS Patching Automation)
**Expected Depth:** GRUB configuration, grubby, kernel management on RHEL, AMI implications

**Answer:**

**How this happens in our patching context:**
After `dnf update kernel*`, a new kernel is installed but GRUB may still default to the old one, OR the new kernel boots but has issues (missing driver, regression). In Project 9, our 7-dimension validation catches this — `uname -r` vs latest installed kernel mismatch = FAIL.

**Diagnosis:**

```bash
# What kernel is currently running?
uname -r
# Example: 5.14.0-362.8.1.el9_3.x86_64

# What kernels are installed?
rpm -q kernel | sort -V
# or: dnf list installed kernel*

# What kernel is GRUB configured to boot?
grubby --default-kernel
# /boot/vmlinuz-5.14.0-362.13.1.el9_3.x86_64

# What's the default index?
grubby --default-index
# 0 (newest)
```

**Fixing — Option 1: Set correct default kernel:**

```bash
# List all kernels with their index
grubby --info=ALL

# Set specific kernel as default
grubby --set-default /boot/vmlinuz-5.14.0-362.13.1.el9_3.x86_64

# Or by index
grubby --set-default-index=0

# Verify
grubby --default-kernel

# Regenerate GRUB config (if needed)
grub2-mkconfig -o /boot/grub2/grub.cfg    # BIOS
grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg  # UEFI
```

**Fixing — Option 2: Bad kernel, need to rollback:**

```bash
# Boot into previous kernel
grubby --set-default /boot/vmlinuz-5.14.0-362.8.1.el9_3.x86_64

# Remove the bad kernel
dnf remove kernel-5.14.0-362.13.1.el9_3

# Reboot
systemctl reboot
```

**Fixing — Option 3: Instance won't boot at all (can't SSH):**

1. Stop the instance
2. Detach root EBS volume
3. Attach to a rescue instance
4. Mount it: `mount /dev/xvdf1 /mnt`
5. Fix GRUB: `chroot /mnt` → `grubby --set-default /boot/vmlinuz-<good-kernel>`
6. Or edit `/mnt/boot/grub2/grubenv` → change `saved_entry`
7. Unmount, reattach to original instance, start

**In Project 9, we prevent this proactively:**
- `allow_reboot` is a survey parameter — kernel patches that require reboot are explicit
- After reboot, validation checks `uname -r` matches the latest installed kernel
- If mismatch → automatic rollback (grubby set previous kernel + reboot)
- We keep minimum 2 kernels installed (`installonly_limit=3` in dnf.conf)

---

### Q4: What is initramfs and when do you need to regenerate it?
**Project Reference:** Project 9 (Enterprise OS Patching Automation)
**Expected Depth:** Purpose, contents, when regeneration is required, how to do it safely

**Answer:**

**What initramfs is:**
initramfs (Initial RAM Filesystem) is a compressed cpio archive loaded into memory by the bootloader alongside the kernel. It provides a minimal root filesystem containing drivers, tools, and scripts needed to mount the real root filesystem.

**Why it exists:**
The kernel itself can't contain drivers for every possible storage configuration. If your root is on LVM, RAID, NVMe, encrypted LUKS, or NFS — the kernel needs those modules BEFORE it can access root. initramfs provides them.

**What's inside:**
```bash
# Examine contents of initramfs
lsinitrd /boot/initramfs-$(uname -r).img

# Or extract it
mkdir /tmp/initramfs && cd /tmp/initramfs
/usr/lib/dracut/skipcpio /boot/initramfs-$(uname -r).img | zcat | cpio -idmv
```

Contents typically include:
- Kernel modules (NVMe, ENA, XFS/ext4, dm-crypt, LVM)
- udev rules (device detection)
- systemd (or busybox init)
- dracut scripts (RHEL's initramfs generator)
- Minimal binaries (mount, modprobe, cryptsetup)

**When you MUST regenerate:**

| Scenario | Why | Command |
|----------|-----|---------|
| Changed storage driver | New driver needed to see root disk | `dracut -f` |
| Moved root to LVM/RAID | LVM tools must be in initramfs | `dracut -f` |
| Enabled LUKS encryption | cryptsetup must be in initramfs | `dracut -f` |
| Changed filesystem (ext4→XFS) | Different fs module needed | `dracut -f` |
| Migrating to different instance type | May need different storage driver (NVMe) | `dracut -f --add-drivers nvme` |
| Dracut module config changed | /etc/dracut.conf.d/ modified | `dracut -f` |
| Kernel update installed | Automatically done by package manager | (automatic) |

**How to regenerate safely:**

```bash
# For current running kernel
dracut -f /boot/initramfs-$(uname -r).img $(uname -r)

# For a specific kernel
dracut -f /boot/initramfs-5.14.0-362.13.1.el9_3.x86_64.img 5.14.0-362.13.1.el9_3.x86_64

# With verbose output (for debugging)
dracut -fv 2>&1 | tee /tmp/dracut.log

# Add specific driver
dracut -f --add-drivers "nvme ena"

# Verify it was created
ls -la /boot/initramfs-$(uname -r).img
lsinitrd /boot/initramfs-$(uname -r).img | grep nvme
```

**In Project 9 context:**
- When we patch kernel packages, `dnf update kernel` automatically generates new initramfs via kernel post-install scriptlets
- Our integrity_check role verifies initramfs exists and is non-zero size for the running kernel
- If migrating AMIs between instance families (e.g., m5 to m6i), we ensure NVMe and ENA drivers are in initramfs before creating the AMI
- A corrupt/missing initramfs = unbootable instance = rescue instance required

**Common failure:** `dracut -f` fails because /boot is full. Always check `df /boot` before kernel operations.

---

### Q5: What are systemd targets, and how do you change the default target?
**Project Reference:** Project 9 (Enterprise OS Patching Automation)
**Expected Depth:** Target concept, dependency tree, how to switch, rescue mode, comparison to runlevels

**Answer:**

**What systemd targets are:**
Targets are systemd unit files that represent a desired system state — a synchronization point that groups other units together. They replace SysV init runlevels but are more flexible (a unit can belong to multiple targets, targets can depend on other targets).

**Key targets and their old runlevel equivalents:**

| Target | Old Runlevel | Purpose |
|--------|-------------|---------|
| poweroff.target | 0 | System halt |
| rescue.target | 1 | Single-user, root only, minimal services |
| multi-user.target | 3 | Full multi-user, no GUI (servers use this) |
| graphical.target | 5 | Full multi-user + GUI |
| reboot.target | 6 | Reboot |
| emergency.target | — | Even more minimal than rescue (only root fs mounted read-only) |

**How targets work (dependency tree):**
```
graphical.target
 └── multi-user.target
      ├── basic.target
      │    ├── sockets.target
      │    ├── timers.target
      │    ├── paths.target
      │    └── sysinit.target
      │         ├── local-fs.target (mount filesystems)
      │         └── swap.target
      ├── network-online.target
      ├── sshd.service
      ├── crond.service
      └── [your application.service]
```

**Viewing and changing default target:**

```bash
# What's the current default?
systemctl get-default
# multi-user.target (typical for servers)

# Change default target (persistent across reboots)
systemctl set-default multi-user.target

# Switch target NOW (without reboot) — careful, this stops services!
systemctl isolate rescue.target

# Boot into specific target (one-time, at GRUB):
# Edit kernel line, append: systemd.unit=rescue.target
```

**Listing and inspecting targets:**

```bash
# All available targets
systemctl list-units --type=target

# What services are part of multi-user.target?
systemctl list-dependencies multi-user.target

# What target a service belongs to
systemctl show sshd.service -p WantedBy
# WantedBy=multi-user.target
```

**In Project 9 context:**
- After kernel reboot during patching, we verify the system reached `multi-user.target`
- `systemctl is-system-running` returns: `running` (all good), `degraded` (some units failed), `maintenance` (rescue mode)
- Our validation checks `systemctl is-system-running` — if `degraded`, we identify failed units and report them
- If a reboot lands in emergency.target (bad fstab entry, fsck failed), patching is considered FAILED → rollback

**Common interview gotcha:** "What's the difference between `isolate` and `set-default`?"
- `isolate` = switch NOW, non-persistent
- `set-default` = change symlink at `/etc/systemd/system/default.target`, takes effect on next boot

---

### Q6: An instance won't boot after patching. Walk me through your full troubleshooting approach.
**Project Reference:** Project 9 (Enterprise OS Patching Automation) / Project 8 (HA/DR)
**Expected Depth:** Systematic approach covering all boot stages, cloud-specific tools, rescue techniques

**Answer:**

**Context:** In Project 9, our validation caught the failure and rolled back, but the 20% batch (those 2 servers) need recovery. Or in Project 8: DR instance launched from patched AMI won't come up.

**Systematic approach (layer by layer):**

**Step 1: Gather information without touching the instance**

```bash
# Check instance status checks (AWS Console or CLI)
aws ec2 describe-instance-status --instance-id i-xxx
# System Status Check: FAILED = hypervisor/hardware issue (AWS problem)
# Instance Status Check: FAILED = OS-level issue (our problem)

# Get system log (serial console output — shows boot messages)
aws ec2 get-console-output --instance-id i-xxx --output text
# This shows kernel panics, fsck errors, mount failures, systemd failures

# Get screenshot (sometimes more revealing)
aws ec2 get-console-screenshot --instance-id i-xxx
```

**Step 2: Analyze console output for failure stage**

| What You See | Failed Stage | Fix |
|-------------|-------------|-----|
| No output at all | Firmware/GRUB | Corrupt boot sector, wrong AMI, instance store issue |
| "Kernel panic - not syncing" | Kernel | Bad kernel, missing initramfs, wrong root= parameter |
| "ALERT! /dev/xvda1 does not exist" | initramfs | Missing NVMe driver in initramfs (common after migration) |
| "Failed to mount /data" | systemd mount | Bad fstab entry |
| "A start job is running for..." (hangs) | systemd service | Service timeout blocking boot |
| "Welcome to emergency mode" | systemd | Failed dependency (usually filesystem) |
| Boot completes but no SSH | Network/sshd | cloud-init networking failed, security group, sshd crashed |

**Step 3: Attempt least-invasive fixes**

```bash
# If SSH works but system is degraded:
systemctl --failed                    # Show failed units
journalctl -b --priority=err         # Errors since last boot
journalctl -u cloud-init             # cloud-init issues

# If network is issue:
# Check security group, NACL, route table (nothing changed during patching?)
```

**Step 4: Rescue instance approach (when SSH is impossible)**

```bash
# 1. Stop the broken instance (don't terminate!)
aws ec2 stop-instances --instance-ids i-broken

# 2. Detach root volume
aws ec2 detach-volume --volume-id vol-xxx

# 3. Launch rescue instance (same AZ, same OS)
# 4. Attach broken volume to rescue instance
aws ec2 attach-volume --volume-id vol-xxx --instance-id i-rescue --device /dev/xvdf

# 5. Mount and investigate
mount /dev/xvdf1 /mnt        # or /dev/nvme1n1p1
# Check logs:
cat /mnt/var/log/messages
cat /mnt/var/log/cloud-init.log
# Check fstab:
cat /mnt/etc/fstab           # Look for bad entries
# Check GRUB:
cat /mnt/boot/grub2/grubenv

# 6. Fix common issues:
# Bad fstab: comment out the offending line, add nofail
# Wrong kernel: chroot /mnt && grubby --set-default <good-kernel>
# Missing module: chroot /mnt && dracut -f

# 7. Unmount, reattach, start
umount /mnt
aws ec2 detach-volume --volume-id vol-xxx
aws ec2 attach-volume --volume-id vol-xxx --instance-id i-broken --device /dev/xvda
aws ec2 start-instances --instance-ids i-broken
```

**Step 5: If rescue doesn't work — rebuild**

- In Project 9: Restore from pre-patching AMI snapshot (we take one before every patch)
- In Project 8: Launch from known-good AMI in DR region

**Most common post-patching boot failures I've seen:**
1. **fstab references a device that changed** (e.g., EBS volume UUID changed) — use `nofail` option
2. **Kernel update removed a custom module** — rebuild with DKMS or rollback kernel
3. **initramfs not regenerated for new kernel** — rescue and `dracut -f`
4. **SELinux relabeling on next boot** (after package update touches policy) — looks like hang, just takes 10-20 minutes
5. **cloud-init re-runs and overwrites networking** — `/etc/cloud/cloud.cfg` misconfigured

**Prevention (built into Project 9):**
- AMI snapshot before patching
- Keep 2+ kernels installed (rollback option)
- Validate initramfs exists for new kernel before reboot
- `nofail` on all non-root fstab entries
- Test patching on staging AMI first (same AMI → same result in prod)

---
