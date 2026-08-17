# Linux Interview Q&A — Category 6: Storage & Filesystem

---

### Q9: Explain the Linux filesystem hierarchy. What's in /var, /tmp, /opt, /etc, /proc, /sys?

**Project Reference:** Project 9 (OS Patching Automation — disk space validation, mount checks)
**Expected Depth:** Practical knowledge of what lives where and WHY, not just FHS memorization

**Answer:**

The Filesystem Hierarchy Standard (FHS) defines where things go. At 12 YOE, you need to know this for troubleshooting (which partition is full? where are logs?):

| Directory | Purpose | Key contents | Why it matters operationally |
|-----------|---------|-------------|------------------------------|
| `/etc` | System configuration (static, not binary) | `fstab`, `ssh/`, `systemd/`, `yum.repos.d/`, `pki/` | Config drift, backup before patching |
| `/var` | Variable data (changes during runtime) | `log/`, `lib/` (databases, packages), `spool/`, `cache/` | #1 cause of disk full — logs, package cache |
| `/tmp` | Temporary files (cleared on reboot) | Session files, build artifacts | Sticky bit set (users can't delete each other's files) |
| `/opt` | Optional/third-party software | Custom apps installed outside package manager | Self-contained — easy to backup/remove |
| `/proc` | Virtual filesystem — kernel/process info | Per-PID directories, `/proc/sys/` (tunable parameters) | Not on disk. Reading = querying kernel. Free. |
| `/sys` | Virtual filesystem — device/driver info | Block devices, network interfaces, kernel subsystems | Used by udev, container runtimes, `sysctl` tuning |
| `/usr` | User programs (shareable, read-only) | `bin/`, `lib/`, `share/`, `local/` | Should be read-only in production |
| `/home` | User home directories | User data, dotfiles | Usually separate partition for quotas |
| `/boot` | Kernel, initramfs, bootloader | `vmlinuz-*`, `initramfs-*`, `grub2/` | Small partition — old kernels fill it up |
| `/run` | Runtime volatile data (tmpfs) | PID files, socket files, systemd runtime | Cleared every boot, tmpfs (in RAM) |

**Practical implications for Project 9:**
```yaml
# Disk space validation in post-patch checks
- name: Check all critical mounts have space
  ansible.builtin.shell: |
    df -h {{ item }} | awk 'NR==2 {print $5}' | tr -d '%'
  loop:
    - /          # Root — OS files
    - /var       # Logs + package cache (grows during patching!)
    - /tmp       # Temp files during package install
    - /boot      # Old kernels accumulate here
  register: disk_usage
  failed_when: disk_usage.stdout | int > 90
```

**Common issues:**
- `/var/log` fills up → services crash, syslog stops, can't SSH in
- `/boot` fills up → can't install new kernel during patching
- `/tmp` fills up → applications fail (can't create temp files)
- `/` at 100% → system-wide failure, potential data corruption

---

### Q10: What is LVM? Explain PV, VG, LV. How do you extend a filesystem on a running system?

**Project Reference:** Project 9 (OS Patching — disk management), Project 2 (3-Tier AWS — EBS volumes)
**Expected Depth:** Full LVM stack understanding, practical extension procedure, advantages over raw partitions

**Answer:**

**LVM (Logical Volume Manager) — abstraction layer between physical disks and filesystems:**

```
Physical Disks/Partitions          LVM Abstraction         Filesystems
┌─────────┐  ┌─────────┐    ┌───────────────────────┐    ┌──────────┐
│ /dev/sda1│  │/dev/sdb │    │   Volume Group (VG)   │    │ /var     │
│  (PV)   │──│  (PV)   │───▶│      "vg_data"        │───▶│ (ext4)   │
└─────────┘  └─────────┘    │                       │    └──────────┘
                             │  ┌─────┐ ┌─────┐     │    ┌──────────┐
                             │  │ LV1 │ │ LV2 │     │───▶│ /home    │
                             │  └─────┘ └─────┘     │    │ (xfs)    │
                             └───────────────────────┘    └──────────┘
```

**Components:**
- **PV (Physical Volume):** Raw disk or partition marked for LVM use (`pvcreate /dev/sdb`)
- **VG (Volume Group):** Pool of storage from one or more PVs (`vgcreate vg_data /dev/sdb`)
- **LV (Logical Volume):** Carved from VG, gets a filesystem (`lvcreate -L 50G -n lv_var vg_data`)

**Why LVM over raw partitions:**
1. Resize volumes without repartitioning or rebooting
2. Span multiple physical disks
3. Snapshots (for backups before patching)
4. Thin provisioning (over-allocate, use on demand)
5. Move data between physical disks live (pvmove)

**Extending a filesystem on a running system (online resize):**

Scenario: `/var` is 90% full, need to add 20GB.

```bash
# Step 1: Add space to VG (if needed — e.g., new EBS volume attached)
pvcreate /dev/xvdf                    # Initialize new disk as PV
vgextend vg_data /dev/xvdf            # Add PV to volume group

# Step 2: Extend the logical volume
lvextend -L +20G /dev/vg_data/lv_var  # Add 20GB
# OR
lvextend -l +100%FREE /dev/vg_data/lv_var  # Use all remaining space

# Step 3: Resize the filesystem (ONLINE — no unmount needed)
# For ext4:
resize2fs /dev/vg_data/lv_var         # Auto-detects new size

# For XFS:
xfs_growfs /var                        # XFS uses mount point, not device
```

**Key points:**
- ext4 and XFS both support online grow (no downtime)
- XFS cannot shrink (only grow) — important design decision
- ext4 can shrink but requires unmounting first (downtime)
- `lvextend -r` combines step 2+3 (extends LV AND resizes filesystem)

**In Project 9 (pre-patch checks):**
```yaml
- name: Ensure /var has enough space for patching
  ansible.builtin.shell: |
    df /var | awk 'NR==2 {print $4}'  # Available KB
  register: var_free
  failed_when: var_free.stdout | int < 2097152  # Less than 2GB free
```

---

### Q11: How do you resize an EBS volume on a running EC2 instance? Walk through the full process.

**Project Reference:** Project 2 (3-Tier AWS Architecture — EBS volumes on EC2)
**Expected Depth:** AWS + Linux integration, full procedure, gotchas

**Answer:**

**Full procedure (zero-downtime — online resize):**

**Step 1: Resize EBS volume in AWS (API/Console/Terraform)**
```bash
# AWS CLI
aws ec2 modify-volume --volume-id vol-0abc123 --size 100
# Changes from current size to 100GB

# Terraform (Project 2):
resource "aws_ebs_volume" "app_data" {
  size = 100  # Changed from 50 to 100
}
# terraform apply — modifies in place
```

**Step 2: Wait for volume modification to complete**
```bash
aws ec2 describe-volumes-modifications --volume-id vol-0abc123
# State: "optimizing" or "completed"
# You can proceed at "optimizing" — partition resize is safe
```

**Step 3: On the EC2 instance — grow the partition (if partitioned)**
```bash
# Check current state
lsblk
# NAME    MAJ:MIN  SIZE  TYPE  MOUNTPOINT
# xvdf    202:80   100G  disk
# └─xvdf1 202:81   50G   part  /data     ← partition still 50G!

# Grow partition to fill disk (ONLINE, no unmount)
growpart /dev/xvdf 1
# CHANGED: partition=1 start=2048 old: size=104855552 end=104857600 new: size=209713119 end=209715167

# Verify
lsblk
# └─xvdf1 202:81   100G  part  /data    ← now 100G
```

**Step 4: Resize the filesystem (ONLINE)**
```bash
# Detect filesystem type
file -s /dev/xvdf1
# OR
df -T /data

# For ext4:
resize2fs /dev/xvdf1

# For XFS:
xfs_growfs /data    # XFS uses mount point

# Verify
df -h /data
# /dev/xvdf1  100G  45G  55G  45%  /data
```

**Common gotchas:**
1. **No partition table (raw device):** Skip `growpart`, go straight to `resize2fs`/`xfs_growfs`
2. **NVMe instance types:** Device names are `/dev/nvme1n1p1` not `/dev/xvdf1`
3. **Volume in "optimizing" state:** You CAN proceed with growpart — it's safe
4. **LVM on top of EBS:** Need to also `pvresize /dev/xvdf1` then `lvextend` then `resize2fs`
5. **Root volume resize:** Same process works on root volume (no reboot needed on modern kernels)

**Automation in Ansible (Project 9 style):**
```yaml
- name: Grow partition after EBS resize
  ansible.builtin.command: growpart /dev/xvdf 1
  register: growpart_result
  changed_when: "'CHANGED' in growpart_result.stdout"
  failed_when: growpart_result.rc != 0 and 'NOCHANGE' not in growpart_result.stdout

- name: Resize XFS filesystem
  ansible.builtin.command: xfs_growfs /data
  when: filesystem_type == 'xfs'

- name: Resize ext4 filesystem
  ansible.builtin.command: resize2fs /dev/xvdf1
  when: filesystem_type == 'ext4'
```

---

### Q12: Explain /etc/fstab — format, UUID vs device name, mount options. What happens if an entry is wrong?

**Project Reference:** Project 9 (OS Patching — mount validation post-reboot)
**Expected Depth:** fstab syntax, failure modes, recovery procedures

**Answer:**

**fstab format (6 fields):**
```
<device>       <mountpoint>  <fstype>  <options>        <dump>  <pass>
UUID=abc-123   /data         xfs       defaults,noatime  0       2
/dev/vg/lv_var /var          ext4      defaults          0       2
tmpfs          /tmp          tmpfs     size=2G,noexec    0       0
```

**Field breakdown:**
1. **Device:** What to mount (UUID, LABEL, /dev/path, NFS share)
2. **Mount point:** Where in the filesystem tree
3. **Filesystem type:** ext4, xfs, nfs, tmpfs, swap
4. **Options:** Mount options (comma-separated)
5. **Dump:** Backup flag (0 = skip, 1 = backup — rarely used)
6. **Pass:** fsck order (0 = skip, 1 = root, 2 = other filesystems)

**UUID vs device name — why UUID wins:**
```bash
# Device names can CHANGE:
# - Adding a new disk shifts /dev/sdb → /dev/sdc
# - NVMe devices enumerate differently on reboot
# - AWS can attach volumes in different order

# UUID is PERMANENT:
blkid /dev/xvdf1
# /dev/xvdf1: UUID="a1b2c3d4-..." TYPE="xfs"

# In fstab — ALWAYS use UUID in production:
UUID=a1b2c3d4-e5f6-7890-abcd-ef1234567890 /data xfs defaults,nofail 0 2
```

**Critical mount options:**

| Option | Meaning | When to use |
|--------|---------|-------------|
| `defaults` | rw, suid, dev, exec, auto, nouser, async | Base option set |
| `noatime` | Don't update access time on reads | Performance (reduces writes) |
| `noexec` | Can't execute binaries from this mount | Security for /tmp, /var/tmp |
| `nosuid` | Ignore setuid bits | Security for user-writable mounts |
| `nofail` | Don't fail boot if mount fails | NON-ROOT EBS volumes! Critical for AWS |
| `_netdev` | Wait for network before mounting | NFS, iSCSI, EFS |
| `ro` | Read-only | Immutable data |

**What happens if an fstab entry is wrong:**

| Scenario | Consequence | Recovery |
|----------|-------------|----------|
| UUID typo (device doesn't exist) | Boot hangs waiting for device (up to 90s) | Without `nofail` → drops to emergency shell |
| Wrong fstype | Mount fails | Emergency shell, fix fstab |
| Bad mount options | Mount fails or mounts degraded | Emergency shell |
| Missing `nofail` + EBS detached | **SYSTEM WON'T BOOT** | Detach root volume → attach to rescue instance → fix fstab |

**The `nofail` lesson (learned the hard way in AWS):**
```bash
# WRONG — server won't boot if EBS volume is detached:
UUID=abc-123 /data xfs defaults 0 2

# RIGHT — server boots even if volume is missing:
UUID=abc-123 /data xfs defaults,nofail 0 2
```

**In Project 9 (post-reboot validation):**
```yaml
- name: Verify all fstab entries are mounted
  ansible.builtin.shell: |
    findmnt --fstab --verify
  register: fstab_check
  failed_when: fstab_check.rc != 0

- name: Check all expected mounts are present
  ansible.builtin.shell: |
    mountpoint -q {{ item }}
  loop: "{{ expected_mounts }}"
```

---

### Q13: df says disk is full but du disagrees — total usage doesn't add up. What's happening and how do you fix it?

**Project Reference:** Project 9 (OS Patching — disk space validation)
**Expected Depth:** Understanding of file handles, deleted-but-open files, practical resolution

**Answer:**

**Root cause: Deleted files still held open by running processes.**

When a file is deleted (`rm`) but a process still has it open (file descriptor exists), the kernel:
- Removes the directory entry (file disappears from `ls`)
- Keeps the data blocks allocated (process can still read/write)
- `du` doesn't see it (no directory entry to traverse)
- `df` still counts it (blocks still allocated on the filesystem)

**The gap:**
```
df reports:  95% used (filesystem level — counts actual allocated blocks)
du reports:  60% used (traverses directory tree — can't see deleted files)
Difference:  35% = deleted files still held open
```

**How to find the culprit:**
```bash
# Method 1: lsof (best approach)
lsof +L1  # Files with link count < 1 (deleted but open)
# COMMAND   PID  USER  FD  TYPE  SIZE     NLINK  NAME
# nginx     1234 root  5w  REG   50.3GB   0      /var/log/nginx/access.log (deleted)

# Method 2: /proc filesystem
find /proc/*/fd -ls 2>/dev/null | grep '(deleted)'

# Method 3: Check total of deleted-but-open
lsof +L1 | awk '{sum += $7} END {print sum/1024/1024/1024 " GB"}'
```

**How to fix:**

**Option 1: Restart the process (releases file descriptor)**
```bash
systemctl restart nginx  # Closes all FDs, reclaims space immediately
```

**Option 2: Truncate without restart (zero-downtime):**
```bash
# Find the FD number from lsof output (e.g., fd=5)
# Truncate via /proc:
: > /proc/1234/fd/5  # Truncate to zero bytes — process keeps writing from offset 0

# OR (same thing):
echo -n > /proc/1234/fd/5
```

**Option 3: Signal the process to reopen log files:**
```bash
kill -USR1 $(cat /var/run/nginx.pid)  # Nginx reopens log files
# logrotate does this automatically with copytruncate or postrotate scripts
```

**Prevention:**
```
# logrotate configuration
/var/log/nginx/access.log {
    daily
    rotate 7
    compress
    missingok
    notifempty
    postrotate
        /bin/kill -USR1 $(cat /var/run/nginx.pid 2>/dev/null) 2>/dev/null || true
    endscript
}
```

**In Project 9 (disk validation):**
If post-patch disk check shows high usage but `du` disagrees, the patching automation would flag it and recommend service restart.

**Other causes of df/du mismatch:**
- Mounted filesystem hides files underneath the mount point
- Reserved blocks (ext4 reserves 5% for root by default — `tune2fs -m 1` to reduce)
- Filesystem corruption (run `fsck`)

---

### Q14: What is inode exhaustion? How do you detect and fix it?

**Project Reference:** Project 9 (OS Patching — pre-patch validation)
**Expected Depth:** Understanding of inodes vs blocks, real-world causes, detection and remediation

**Answer:**

**What are inodes:**
- Every file/directory has an inode — a data structure containing metadata (permissions, owner, size, timestamps, pointers to data blocks)
- Inode does NOT contain the filename (that's in the directory entry)
- Inode count is fixed at filesystem creation (`mkfs.ext4 -N <count>` or determined by bytes-per-inode ratio)
- XFS dynamically allocates inodes (rarely exhausted)

**Inode exhaustion = can't create new files even though disk has free space:**
```bash
$ touch newfile
touch: cannot touch 'newfile': No space left on device

$ df -h /
Filesystem  Size  Used  Avail  Use%  Mounted on
/dev/sda1   50G   30G   20G    60%   /         ← Plenty of space!

$ df -i /
Filesystem  Inodes    IUsed    IFree   IUse%  Mounted on
/dev/sda1   3276800   3276800  0       100%   /         ← Zero inodes!
```

**Common causes:**
1. **Millions of small files:** Mail queues, session files, PHP temp files, container layers
2. **Runaway process creating temp files:** `find /tmp -type f | wc -l` → millions
3. **Package cache:** Old yum/apt cache with thousands of metadata files
4. **Monitoring agents:** Accumulating metric files

**Detection:**
```bash
# Check inode usage
df -i

# Find directories with most files
find / -xdev -printf '%h\n' | sort | uniq -c | sort -rn | head -20
# Shows which directories contain the most files

# Quick check per directory
for dir in /tmp /var/spool /var/cache /var/lib; do
  echo "$dir: $(find $dir -type f 2>/dev/null | wc -l) files"
done
```

**Fixing inode exhaustion:**
```bash
# 1. Find and delete the small files
find /var/spool/postfix/maildrop -type f -delete  # Common culprit
find /tmp -type f -mtime +7 -delete               # Old temp files

# 2. Clean package cache
yum clean all     # RHEL
apt-get clean     # Debian

# 3. Remove old log files
find /var/log -name "*.gz" -mtime +30 -delete

# 4. If using containers — prune unused layers
docker system prune -a
```

**Prevention:**
- Separate partitions for `/tmp`, `/var/spool` (contain the blast radius)
- Set up monitoring alerts on inode usage (`df -i` in Prometheus node_exporter)
- Logrotate with `maxage` to auto-delete old logs
- tmpwatch/systemd-tmpfiles to clean temp directories
- For ext4, increase inode ratio at mkfs time: `mkfs.ext4 -i 4096 /dev/sda1` (one inode per 4KB — more inodes)

**In Project 9:**
```yaml
- name: Pre-patch inode check
  ansible.builtin.shell: |
    df -i / | awk 'NR==2 {gsub(/%/,""); print $5}'
  register: inode_pct
  failed_when: inode_pct.stdout | int > 90
```

---

### Q15: XFS vs EXT4 — when would you choose one over the other?

**Project Reference:** Project 2 (3-Tier AWS — EBS filesystem choice), Project 9 (OS Patching — filesystem operations)
**Expected Depth:** Practical decision criteria, not theoretical filesystem internals

**Answer:**

| Aspect | EXT4 | XFS |
|--------|------|-----|
| Max file size | 16 TB | 8 EB (exabytes) |
| Max volume size | 1 EB | 8 EB |
| Default on | Debian/Ubuntu | RHEL 7+, Amazon Linux 2 |
| Shrink volume | ✅ Yes (offline only) | ❌ No (grow only) |
| Grow volume | ✅ Online | ✅ Online |
| Performance (large files) | Good | Better (parallel allocation) |
| Performance (small files) | Better | Good |
| Fragmentation | Can defrag (e4defrag) | Can defrag (xfs_fsr) |
| Repair time | Faster (smaller journals) | Slower on very large volumes |
| Metadata journaling | Journal + ordered data | Metadata-only journal (faster) |
| Inode allocation | Fixed at creation | Dynamic (rarely exhausted) |
| Delete large files | Slow (block-by-block free) | Fast (extent-based free) |
| Crash consistency | Very good | Excellent (log-structured) |
| AWS default (AL2) | No | ✅ Yes |

**When to choose XFS:**
- RHEL/Amazon Linux environments (it's the default — stay consistent)
- Large files (databases, media, logs >1GB)
- High-throughput workloads (parallel I/O, streaming writes)
- Never need to shrink the volume
- Production servers where performance matters

**When to choose EXT4:**
- Need to shrink filesystem (disaster recovery, migration)
- Lots of small files (mail servers, session stores)
- Debian/Ubuntu environments (default, better tooling support)
- Boot partition (/boot — simpler, universal GRUB support)
- Development environments (more forgiving, familiar tools)

**In Project 2 (AWS 3-Tier):**
We use XFS because:
1. Amazon Linux 2 default — consistent with AMI baseline
2. EBS volumes only grow (shrink requires migration anyway)
3. Application writes large log files and database dumps
4. `xfs_growfs` works perfectly with EBS resize workflow

**Gotcha for LVM users:**
```bash
# EXT4 — can shrink LV (requires unmount):
umount /data
e2fsck -f /dev/vg/lv_data
resize2fs /dev/vg/lv_data 20G
lvreduce -L 20G /dev/vg/lv_data

# XFS — CANNOT shrink. Only option is:
# 1. Create new smaller LV
# 2. Copy data
# 3. Remount on new LV
# 4. Delete old LV
```

---

### Q16: How does the overlay filesystem work? How does Docker use it for image layers?

**Project Reference:** Project 1 (DevSecOps Pipeline — multi-stage Docker builds), Project 3 (EKS — container storage)
**Expected Depth:** OverlayFS mechanics, Docker layer model, COW behavior, practical implications

**Answer:**

**OverlayFS concept:**
Overlay filesystem merges multiple directory trees into a single unified view. Think of it as transparent layers stacked on top of each other:

```
┌─────────────────────────────────┐
│  Container Layer (upperdir)      │  ← Writable (container's changes)
│  - modified_config.py           │
│  - new_file.txt                 │
├─────────────────────────────────┤
│  Image Layer 3 (lowerdir)       │  ← Read-only
│  - app/main.py                  │
│  - requirements.txt             │
├─────────────────────────────────┤
│  Image Layer 2 (lowerdir)       │  ← Read-only
│  - /usr/bin/python3             │
│  - /usr/lib/python3/...         │
├─────────────────────────────────┤
│  Image Layer 1 - base (lowerdir)│  ← Read-only
│  - /bin/bash, /usr/lib/...      │
│  - (debian:slim base)           │
└─────────────────────────────────┘

Merged view (what the container sees):
/bin/bash, /usr/bin/python3, app/main.py, modified_config.py, new_file.txt
```

**How OverlayFS works (kernel level):**

Three key directories:
- **lowerdir:** Read-only base layers (can be multiple, stacked)
- **upperdir:** Writable layer (container-specific changes)
- **merged:** Unified view (what the process/container sees)
- **workdir:** Internal use by kernel (atomic operations)

```bash
# Manual overlay mount (same thing Docker does):
mount -t overlay overlay \
  -o lowerdir=/layer1:/layer2:/layer3,upperdir=/container_rw,workdir=/work \
  /merged_view
```

**Copy-on-Write (COW) behavior:**

| Operation | What happens |
|-----------|-------------|
| Read a file | Served from highest layer that contains it |
| Create new file | Written to upperdir only |
| Modify existing file | **Entire file copied up** to upperdir, then modified there |
| Delete a file | "Whiteout" file created in upperdir (hides lower layer file) |
| Delete a directory | "Opaque" directory created in upperdir |

**Docker's use of overlay2:**
```bash
# See Docker's storage driver:
docker info | grep "Storage Driver"
# Storage Driver: overlay2

# Inspect layers of an image:
docker inspect myimage | jq '.[0].GraphDriver.Data'
# {
#   "LowerDir": "/var/lib/docker/overlay2/abc.../diff:/var/lib/docker/overlay2/def.../diff",
#   "UpperDir": "/var/lib/docker/overlay2/xyz.../diff",
#   "MergedDir": "/var/lib/docker/overlay2/xyz.../merged",
#   "WorkDir": "/var/lib/docker/overlay2/xyz.../work"
# }
```

**Why this design matters (Project 1 — multi-stage builds):**
```dockerfile
FROM python:3.9-slim AS builder   # Layer 1: 150MB (shared across all images using this base)
RUN pip install -r requirements.txt  # Layer 2: 50MB (shared if requirements unchanged)
COPY . /app                        # Layer 3: 5MB (changes every build)
```

- Base image layers are shared across ALL containers using that image
- 100 containers from same image = 1 copy of image layers + 100 thin writable layers
- Only changed files occupy additional space
- This is why `docker pull` only downloads layers you don't already have

**Performance implications:**
- First write to an existing file = copy-up latency (can be large for big files like databases)
- Solution for databases: use volumes (bypass overlay entirely)
- Many small files modification = many copy-ups = I/O amplification

**In Project 3 (EKS with PV/PVC):**
Persistent data uses PV/PVC (direct block device), not overlay. Overlay is only for the container filesystem (application code, libraries). This separation is critical — overlay is ephemeral (gone when pod dies), PVCs persist.
