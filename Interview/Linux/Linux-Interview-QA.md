# Linux Interview Questions & Answers (10+ YOE DevOps/Cloud)

A single consolidated collection of 79 Linux interview Q&A. Answers are **brief**, in **easy English**, with **diagrams** where helpful. No duplicate questions.

## Contents (by topic)

- **Fundamentals** — Q1, Q2, Q30
- **Commands & CLI** — Q4, Q5, Q58, Q59, Q60
- **Filesystem & Storage** — Q3, Q16, Q18, Q61, Q62
- **Permissions & Ownership** — Q7, Q8, Q9, Q63, Q64
- **Processes & Signals** — Q6, Q10, Q11, Q12, Q65
- **Memory Management** — Q13, Q14, Q15, Q66, Q67
- **Boot & Init Systems** — Q32, Q33, Q34, Q35, Q36
- **systemd & Services** — Q37, Q38, Q39, Q40, Q41
- **Networking** — Q19, Q20, Q42, Q43, Q44, Q45
- **Package Management & Patching** — Q46, Q47, Q48, Q49, Q50
- **Logging** — Q51, Q52, Q53, Q54, Q55
- **Users, Groups & sudo** — Q31, Q68, Q69, Q70
- **Cron / Scheduling** — Q21, Q22, Q71, Q72
- **Shell Scripting** — Q23, Q24, Q73, Q74
- **Performance & Troubleshooting** — Q25, Q29, Q75, Q76
- **Security & Hardening** — Q28, Q77, Q78, Q79
- **Production Scenarios** — Q26, Q27 (also: Q18, Q29, Q45, Q55, Q71)

> Note: questions keep their original numbers (Q1–Q79). Cross-references between answers (e.g., "see Q27") remain valid.

---

# 1. Fundamentals

## Q1. What is Linux?

**Short answer:** Linux is an open-source, Unix-like operating system **kernel**. The kernel is the core that talks to the hardware. When you bundle the kernel with tools, libraries, and a package manager, you get a full OS called a **distribution** (distro) — e.g., Ubuntu, RHEL, CentOS, Debian, Amazon Linux, SUSE.

**Why it's used so widely:**
- Stability — runs for months/years without reboot
- High performance — lightweight, efficient
- Strong security — permissions, users, SELinux/AppArmor
- Multi-user — many people can use one system at once
- Multi-tasking — runs many processes together
- Automation — powerful shell scripting

**Where it runs:** Most servers, cloud environments, Kubernetes clusters, and CI/CD platforms run on Linux.

**Kernel vs Distribution (key point):**

```
        ┌─────────────────────────────────────────┐
        │            DISTRIBUTION (Distro)          │
        │   e.g. Ubuntu, RHEL, Debian, Amazon Linux │
        │                                           │
        │   ┌─────────────────────────────────┐     │
        │   │  User apps (nginx, python, git) │     │
        │   ├─────────────────────────────────┤     │
        │   │  Shell + GNU tools (bash, ls…)  │     │
        │   ├─────────────────────────────────┤     │
        │   │  Package manager (apt/yum/dnf)  │     │
        │   ├─────────────────────────────────┤     │
        │   │        LINUX KERNEL             │◄── the actual "Linux"
        │   ├─────────────────────────────────┤     │
        │   │        HARDWARE (CPU, RAM…)     │     │
        │   └─────────────────────────────────┘     │
        └─────────────────────────────────────────┘
```

**One-liner for interview:** "Linux is the open-source kernel; a distribution is the kernel plus the tools around it that make a usable OS."

**Kernel vs Distribution — side by side:**

| Aspect | Linux Kernel | Linux Distribution |
|--------|--------------|--------------------|
| What it is | The core program that talks to hardware | Full OS built around the kernel |
| Job | Manage CPU, memory, devices, processes | Give users a ready-to-use system |
| Usable alone? | No — it's just the engine | Yes — you install and run it |
| Examples | Linux 5.15, 6.x (one project, kernel.org) | Ubuntu, RHEL, Debian, SUSE, Amazon Linux |
| Who ships it | Linus Torvalds + kernel community | Vendors: Canonical, Red Hat, etc. |
| Contains | Scheduler, memory mgr, drivers, syscalls | Kernel + bash + apt/yum + apps + config |

**Analogy:** Kernel = a **car engine**. Distribution = the **full car** (engine + seats + steering + dashboard) that you can actually drive.

```
   Kernel  ──►  just the engine  (can't drive it)
   Distro  ──►  engine + body + wheels + controls  (ready to drive)
```

**Interview tip:** Distributions **package the kernel together with system libraries, a package manager, and utilities** to provide a usable operating system. Many distros share the *same* kernel but differ in package manager (apt vs yum), release cycle, and defaults.

---

## Q2. Difference between Linux Kernel and Linux Distribution?

> Merged into **Q1** (see the "Kernel vs Distribution — side by side" table and car analogy there). Kept as a numbered placeholder so later cross-references (e.g., Q30) stay valid.

**In one line:** the **kernel** is the core engine that manages hardware; a **distribution** is the complete usable OS = kernel + libraries + package manager + utilities. See **Q1** for the full comparison.

---

## Q30. Difference between the Linux Kernel and the Shell?

> Different from Q2 (kernel vs *distribution*). Here it's kernel vs *shell* — the boundary between who runs the hardware and who reads your typing.

**Short answer:** The **kernel** is the OS core that controls hardware (CPU, memory, disks, network) and enforces security. The **shell** (bash, zsh, fish) is a normal user program that reads your commands, finds the programs, and asks the kernel to run them.

| Component | Role | Example |
|-----------|------|---------|
| Kernel | Hardware + process/memory management | Schedules `ls`, reads disk blocks |
| Shell | Command interpreter (user space) | Parses `ls -l \| wc -l` |
| User programs | Do the actual work | `ls`, `nginx`, `python3` |

**What happens when you type `ls`:**
```
You type: ls
   │
   ▼
┌─────────────┐   1. searches $PATH, finds /bin/ls
│   SHELL     │   2. asks kernel to run it (fork + exec)
│ (bash/zsh)  │
└──────┬──────┘
       │  system call
       ▼
┌─────────────┐   3. schedules the process, reads the
│   KERNEL    │      directory from disk, enforces perms
└──────┬──────┘
       │  results
       ▼
   SHELL prints output on your terminal
```

---

# 2. Commands & CLI

## Q4. Which Linux commands do you use daily as a DevOps Engineer?

**Short answer:** Group them by what they're *for*, and tie each group to real troubleshooting or automation — don't just list them.

**Grouped by purpose:**

| Group | Commands | Where I use it |
|-------|----------|----------------|
| Navigate & inspect | `ls`, `cd`, `pwd`, `find` | Locate config/logs, `find / -name "*.conf"` in scripts |
| View files | `cat`, `less`, `head`, `tail` | `tail -f /var/log/app.log` to watch live errors |
| Search | `grep` | `grep -i error app.log`, filter `ps`/`journalctl` output |
| Manage files | `cp`, `mv`, `rm`, `tar`, `zip` | Backups, `tar -czf backup.tar.gz /data` in cron jobs |
| Permissions | `chmod`, `chown` | Fix "permission denied" on deploy/key files |
| Disk | `df`, `du` | `df -h` for full disk, `du -sh *` to find the culprit |
| Processes | `ps`, `top`, `free` | Find CPU/memory hogs, check OOM risk |
| Network | `ss` (or `netstat`), `curl`, `wget` | `ss -tulpn` for open ports, `curl` health checks |
| Services & logs | `systemctl`, `journalctl` | Start/stop/enable services, read logs of a failed unit |

**Told as real stories (this is what interviewers want):**

- **Disk full incident:** `df -h` showed `/var` at 100% → `du -sh /var/* | sort -h` pointed to huge logs → cleared old logs and added `logrotate`.
- **Service won't start:** `systemctl status nginx` said failed → `journalctl -u nginx -n 50` revealed a bad config line → fixed and reloaded.
- **App slow:** `top` showed a process pegging CPU → `ps -ef | grep <pid>` identified it → checked logs with `tail -f`.
- **Port check:** after a deploy, `ss -tulpn | grep 8080` confirmed the app was actually listening.
- **Automation:** used `find`, `grep`, `tar`, and `curl` inside bash scripts for backups, log scraping, and post-deploy health checks.

```
Troubleshoot flow (typical):
 systemctl status  ─►  journalctl -u <svc>  ─►  tail -f /var/log/...  ─►  fix  ─►  systemctl restart
 df -h / top / free ─► find the resource that's maxed out ─► du / ps to find who ─► fix
```

**Interview tip:** Rather than reciting the list, say *"Here's how I used `df`, `du`, and `journalctl` to solve a disk-full outage."* Context beats memorization.

---

## Q5. Difference between grep, find, and locate?

**Short answer:** `grep` searches **inside** files (content). `find` searches **for** files (by name/attributes, live). `locate` also finds files by name but from a **prebuilt database** (super fast, may be stale).

| Command | Searches | How | Speed | Freshness |
|---------|----------|-----|-------|-----------|
| `grep` | Text **inside** files | Reads file contents | Medium | Always live |
| `find` | **Files** by name/size/time/perms | Walks the directory tree now | Slow (real-time) | Always live |
| `locate` | **Files** by name | Looks up a prebuilt index (`mlocate.db`) | Very fast | Can be stale until `updatedb` |

```
grep   →  "which files CONTAIN the word error?"      → grep -r "error" /var/log
find   →  "where are the .log files, live right now?" → find /var -name "*.log"
locate →  "where is nginx.conf?" (instant, from index) → locate nginx.conf
```

**Interview tip:** Key line — *"`find` is real-time but slow; `locate` is instant but relies on a database that may be outdated; `grep` is different entirely — it searches inside files, not for them."*

---

## Q58. stdin/stdout/stderr, redirection, and pipes — explain.

**Short answer:** Every process has 3 default streams: **stdin (0)** input, **stdout (1)** normal output, **stderr (2)** errors. Redirection sends them to files; pipes send one command's stdout into the next's stdin.

```
        ┌─────────────┐
stdin 0 ─►  process   ─► stdout 1
        │             ─► stderr 2
        └─────────────┘
```

| Syntax | Meaning |
|--------|---------|
| `> file` | stdout → file (overwrite) |
| `>> file` | stdout → file (append) |
| `2> file` | stderr → file |
| `2>&1` | stderr → wherever stdout goes |
| `> file 2>&1` | both stdout+stderr → file |
| `< file` | file → stdin |
| `cmd1 \| cmd2` | cmd1 stdout → cmd2 stdin |
| `\| tee file` | pipe through AND save a copy to file |

---

## Q59. Text processing: sed, awk, cut — when to use which?

**Short answer:** `cut` extracts columns/fields, `sed` does stream find/replace and line edits, `awk` is a mini-language for column-based logic and aggregation.

| Tool | Best at | Example |
|------|---------|---------|
| `cut` | Simple field/column extraction | `cut -d: -f1 /etc/passwd` (usernames) |
| `sed` | Find/replace, delete/print lines | `sed 's/old/new/g' file` |
| `awk` | Columns + conditions + math | `awk '{sum+=$3} END{print sum}'` |

**Interview tip:** Show the log-analysis `awk | sort | uniq -c | sort -rn` pipeline — it's a daily DevOps move. Mention `sed -i` edits in place (great in automation, but back up first).

---

## Q60. xargs and find -exec — running commands over many results?

**Short answer:** Both apply a command to a list of items. `xargs` builds command lines from stdin; `find -exec` runs a command per match. Use them to act on many files safely.

```
# Delete .tmp files older than 7 days (two equivalent ways):
find /tmp -name '*.tmp' -mtime +7 -delete
find /tmp -name '*.tmp' -mtime +7 -exec rm {} +

# xargs from a pipe:
grep -rl "TODO" . | xargs sed -i 's/TODO/DONE/g'
```

**Interview tip:** The gotcha that matters — filenames with spaces break naive `xargs`; use `find -print0 | xargs -0`. Mention `xargs -P` for parallelism (e.g., process 1000 files across cores).

---

# 3. Filesystem & Storage

## Q3. Explain the Linux directory structure.

**Short answer:** Linux uses a single tree starting at `/` (root). Everything — files, disks, devices — lives under it. The layout follows the **FHS (Filesystem Hierarchy Standard)**.

```
/
├── bin    → essential user commands (ls, cp, cat)
├── sbin   → system/admin commands (reboot, iptables)
├── etc    → configuration files (the "control panel")
├── home   → normal users' home dirs (/home/epatesi)
├── root   → the root user's home dir
├── var    → variable data — logs, caches, spool (/var/log)
├── tmp    → temporary files (cleared on reboot)
├── usr    → user programs & libraries (/usr/bin, /usr/lib)
├── opt    → optional / third-party software
├── lib    → shared libraries for /bin & /sbin
├── boot   → kernel + bootloader files (GRUB, vmlinuz)
├── dev    → device files (disks, tty) — /dev/sda
├── proc   → virtual: live kernel/process info (/proc/cpuinfo)
├── sys    → virtual: hardware & kernel settings
├── mnt    → temporary mount point (manual mounts)
└── media  → auto-mounted removable media (USB, CD)
```

**The ones you actually touch in production:**

| Directory | Why it matters |
|-----------|----------------|
| `/etc` | All config lives here (nginx, ssh, systemd units) |
| `/var/log` | Where you go to debug — app & system logs |
| `/home` | User data and app deploy dirs |
| `/proc`, `/sys` | Live system info for troubleshooting/tuning |
| `/dev` | Disks and devices (e.g., `/dev/sda`, `/dev/nvme0n1`) |
| `/tmp` | Scratch space (don't store anything important) |
| `/opt`, `/usr/local` | Where custom/3rd-party apps get installed |

**Memory aids:**
- `/etc` = **e**verything **t**o **c**onfigure
- `/var` = **var**iable data that grows (logs!)
- `/proc` & `/sys` = not real files on disk — they're the kernel talking to you

**Interview tip:** Don't memorize every directory. Focus on the ones you use daily in production — `/etc`, `/var/log`, `/home`, `/proc`, `/dev` — and explain *why* you go there.

---

## Q16. How do you check disk usage?

**Short answer:** `df -h` shows **filesystem-level** usage (how full each disk/mount is). `du -sh *` shows **directory/file-level** usage (what's taking the space).

| Command | Level | Answers |
|---------|-------|---------|
| `df -h` | Filesystem / mount | *"Which disk is full and by how much?"* |
| `du -sh *` | Directory / file | *"What inside this folder is eating space?"* |

**`df -h` — is a disk full?**
```
df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/nvme0n1p1   50G   47G   3G   94% /          ← almost full!
/dev/nvme0n1p2  100G   20G  80G   20% /var
```
`-h` = human-readable (G/M). Watch the **Use%** column.

**`du -sh *` — what's taking the space?**
```
du -sh *              # size of each item in current dir (summarized)
du -sh /var/* | sort -h    # sort smallest→largest to find the culprit
du -h --max-depth=1 /var   # one level deep
```

**Typical "disk full" workflow:**
```
df -h                     ─►  find the mount at high Use%  (e.g., / at 94%)
cd /   &   du -sh * | sort -h   ─►  drill into the biggest dir
repeat du into subdirs    ─►  find the huge file/folder  ─►  clean up / logrotate
```

**Two flags people forget:**
- `df -i` → check **inode** usage. A disk can show space free but still fail to create files if **inodes are exhausted** (millions of tiny files).
- Deleted-but-open files: if `df` says full but `du` doesn't add up, a process may still hold a deleted file open → find with `lsof | grep deleted`, then restart that process.

**Interview tip:** `df` = the *whole disk* view, `du` = *what's inside*. Real story: `df -h` shows `/var` at 100% → `du -sh /var/* | sort -h` points to huge logs → clean up + add `logrotate`. Bonus: mention `df -i` for the inode-exhaustion trap.

---

## Q17. Difference between df and du?

> Merged into **Q16** — the `df` vs `du` comparison, gotcha, and workflow all live there. Kept as a numbered placeholder so cross-references stay valid. See **Q16**.

---

## Q18. Your disk is full. How do you troubleshoot? (Scenario)

> Scenario question — the interviewer wants a **structured method**, not just commands. Command syntax lives in Q16; here the focus is the *order and reasoning*.

**My approach (say it as steps):**

```
1. CONFIRM   → df -h            which mount is full? (/  vs  /var  vs  /data)
2. LOCATE    → du -sh * | sort -h   drill top-down into the biggest dirs
3. INSPECT   → check usual suspects: /var/log, /tmp, app data, old artifacts
4. ACT       → delete/archive junk; compress or rotate logs (logrotate)
5. HIDDEN    → df full but du low? deleted-but-open file → lsof | grep deleted → restart holder
6. INODES    → df -i   (space free but can't create files = inodes exhausted)
7. EXPAND    → if truly out: grow the volume / add disk / extend LVM
8. PREVENT   → set up logrotate + monitoring alert so it doesn't recur
```

**Step-by-step reasoning:**
1. **Confirm which filesystem** — `df -h`. Don't guess; find the actual full mount.
2. **Find the big directories** — `du -sh * | sort -h`, drilling down level by level.
3. **Check usual suspects** — `/var/log` (runaway logs), `/tmp`, old builds/artifacts, core dumps, package caches.
4. **Clean up safely** — remove or archive unnecessary files; compress or rotate logs.
5. **Deleted-but-open files** — if `df` says full but `du` doesn't match, a process holds a deleted file; find it (`lsof | grep deleted`) and restart it to release space.
6. **Inode exhaustion** — `df -i`; millions of tiny files can fill inodes even with space free.
7. **Expand storage** — if genuinely needed: extend the LVM volume / resize the cloud disk / add a mount.
8. **Prevent recurrence** — add `logrotate`, disk-usage alerts (e.g., alert at 80%), and cleanup cron jobs.

**Interview tip:** Lead with the **structured process** (confirm → locate → clean → verify → prevent), and score bonus points by mentioning the two traps most people miss: **deleted-open files** and **inode exhaustion**. End with *prevention* (logrotate + alerting) — that shows senior thinking.

---

## Q61. Inodes, and hard links vs symbolic links?

**Short answer:** An **inode** stores a file's metadata (permissions, owner, size, timestamps, and pointers to data blocks) — but **not** the name. The directory entry maps a **name → inode**. A **hard link** is another name pointing to the *same inode*; a **symlink (soft link)** is a tiny separate file that stores a *path* to another name.

```
Hard link (same inode):
   name A ─┐
           ├─► inode 1234 ─► data blocks
   name B ─┘        (link count = 2; delete one name, data stays)

Symlink (points to a path):
   linkname ─► "/path/to/target" ─► (its own inode) ─► inode 1234 ─► data
              (if target is renamed/deleted → dangling link)
```

---

## Q62. Mounting, /etc/fstab, and extending storage (LVM)?

> Q35 covered how a bad fstab blocks boot. Here: how mounting and growing storage actually work.

**Short answer:** A filesystem must be **mounted** onto a directory to be usable. `/etc/fstab` defines mounts applied at boot. **LVM** adds a flexible layer so you can grow volumes online without repartitioning.

```
Mount now:      mount /dev/nvme1n1 /data
Persist:        add to /etc/fstab, use UUID (stable) not /dev/sdX (can change!)

/etc/fstab line:
  UUID=abc-123   /data   ext4   defaults,nofail   0 2
  └── device     └mount  └fs    └options          └dump └fsck order
```

**Why UUID + `nofail`:** device names (`/dev/sdb`) can reorder across reboots/cloud attach — **UUID** is stable. `nofail` stops a missing disk from blocking boot (the Q35 trap).

**LVM (the senior part) — grow a volume with no downtime:**
```
Physical Volume (disk) → Volume Group (pool) → Logical Volume (usable) → filesystem

Extend online:
  lvextend -L +50G /dev/vg0/data          # grow the logical volume
  resize2fs /dev/vg0/data                 # grow ext4 to fill it (xfs_growfs for XFS)
  # → no unmount, no reboot; app keeps running
```

**Interview tip:** Mention **UUID over device names** and `nofail` in fstab, and that **LVM lets you extend a filesystem live** (`lvextend` + `resize2fs`/`xfs_growfs`). In cloud, pair with growing the EBS/disk first, then LVM/resize — a common "disk almost full, no downtime" fix (ties to Q18).

---

# 4. Permissions & Ownership

## Q7. Explain Linux file permissions.

**Short answer:** Every file/dir has permissions for **three groups** — Owner, Group, Others — and each group can have **read (r)**, **write (w)**, **execute (x)**.

- **r (read)** = view file / list directory
- **w (write)** = modify file / add-remove files in directory
- **x (execute)** = run file as program / enter (cd into) directory

**Reading `ls -l` output:**
```
-rwxr-xr--   1  root  devops  ...  script.sh
│└┬┘└┬┘└┬┘
│ │  │  └── Others : r--  (read only)
│ │  └───── Group  : r-x  (read + execute)
│ └──────── Owner  : rwx  (read + write + execute)
└────────── type: - = file,  d = directory,  l = link
```

So `-rwxr-xr--` means:
| Who | Permissions | Meaning |
|-----|-------------|---------|
| Owner | rwx | read, write, execute |
| Group | r-x | read, execute |
| Others | r-- | read only |

**Interview tip:** The first character is the **type** (`-` file, `d` dir, `l` link), then permissions come in **three sets of three**: owner, group, others.

---

## Q8. What does chmod 755 mean?

**Short answer:** `755` sets **Owner = full access (7)**, **Group = read+execute (5)**, **Others = read+execute (5)**. Common for scripts and directories.

**How the numbers work (add r+w+x):**
```
r = 4    w = 2    x = 1

7 = 4+2+1 = rwx   (read + write + execute)
5 = 4+0+1 = r-x   (read + execute, no write)
```

| Digit | Applies to | Value | Permission |
|-------|-----------|-------|------------|
| 7 | Owner | 4+2+1 | rwx — full access |
| 5 | Group | 4+0+1 | r-x — read + execute |
| 5 | Others | 4+0+1 | r-x — read + execute |

Result equals `-rwxr-xr-x`.

**Common modes to remember:**
| Mode | Symbolic | Typical use |
|------|----------|-------------|
| 755 | rwxr-xr-x | scripts, directories (all can run/enter, only owner edits) |
| 644 | rw-r--r-- | normal files (owner edits, others read) |
| 700 | rwx------ | private scripts/dirs (owner only) |
| 600 | rw------- | secrets, SSH private keys |
| 777 | rwxrwxrwx | everyone full — avoid (insecure!) |

```
chmod 755 deploy.sh   →  -rwxr-xr-x   (owner edits & runs, others run)
chmod 600 id_rsa      →  -rw-------   (only owner reads — required for SSH keys)
```

**Interview tip:** Say `4=read, 2=write, 1=execute`, add them per group. Flag that `chmod 777` is a security red flag — never use it in production.

---

## Q9. Difference between chmod and chown?

**Short answer:** `chmod` changes **what actions are allowed** (read/write/execute permissions). `chown` changes **who owns** the file (the user and/or group).

| | chmod | chown |
|--|-------|-------|
| Changes | Permissions (r/w/x) | Ownership (user / group) |
| Answers | *"What can be done to it?"* | *"Who owns it?"* |
| Example | `chmod 644 file` | `chown alice file` |
| Typical use | Make a script executable, lock down a key | Give a file to another user/service account |

```
chmod  →  changes the LOCKS on the door (who may read/write/execute)
chown  →  changes the OWNER of the door
```

**Examples:**
```
chmod +x deploy.sh              # make script executable
chmod 600 id_rsa                # restrict a private key
chown ubuntu file.txt           # change owner to 'ubuntu'
chown ubuntu:devops file.txt    # change owner AND group
chown -R www-data:www-data /var/www   # recursive: whole web dir
```

Related: `chgrp devops file.txt` changes only the group (a subset of what `chown` can do).

**Interview tip:** One line — *"`chmod` = permissions, `chown` = ownership."* Common real fix: after copying app files as root, run `chown -R appuser:appuser /app` so the service can access them.

---

## Q63. Special permission bits (setuid, setgid, sticky) and umask?

**Short answer:** Beyond rwx there are three special bits: **setuid** (run as file owner), **setgid** (run as group / inherit group on dirs), **sticky** (only owner can delete in a shared dir). **umask** decides default permissions for new files.

| Bit | On a file | On a directory | Symbol |
|-----|-----------|----------------|--------|
| setuid (4) | Runs as the **file's owner** | — | `s` in owner-x (`-rws...`) |
| setgid (2) | Runs as the file's group | New files **inherit the dir's group** | `s` in group-x |
| sticky (1) | — | Only owner can delete their files | `t` in others-x |

```
Classic examples:
  -rwsr-xr-x  /usr/bin/passwd     ← setuid: normal user updates /etc/shadow (owned by root)
  drwxrwxrwt  /tmp                ← sticky: anyone writes, but can't delete others' files

Set them:
  chmod u+s file   (setuid)   chmod 4755 file
  chmod g+s dir    (setgid)   chmod 2775 dir
  chmod +t dir     (sticky)   chmod 1777 dir
```

**umask (default perms):** new files start from 666 (files) / 777 (dirs) **minus** the umask. Common `umask 022` → files `644`, dirs `755`.

**Interview tip:** setuid on `passwd` is the classic example (and a security risk to audit — `find / -perm -4000`). setgid on a shared team dir keeps group ownership consistent. Sticky on `/tmp` stops users deleting each other's files.

---

## Q64. ACLs and file attributes (chattr) — beyond standard permissions?

**Short answer:** Standard perms only cover one owner + one group + others. **ACLs** let you grant permissions to **additional specific users/groups**. **chattr** sets low-level filesystem **attributes** (like immutable) that even root must clear first.

**ACLs — per-user/group grants:**
```
getfacl file                          # view ACLs
setfacl -m u:alice:rw file            # give alice read+write (on top of normal perms)
setfacl -m g:devs:rx dir              # give group 'devs' read+execute
setfacl -x u:alice file               # remove alice's entry
# a '+' at end of ls -l perms (-rw-r--r--+) means ACLs are present
```

**chattr — filesystem attributes:**
```
chattr +i /etc/resolv.conf     # IMMUTABLE: nobody (even root) can modify/delete/rename
lsattr /etc/resolv.conf        # view attributes
chattr -i file                 # remove immutable before editing
chattr +a logfile              # append-only (great for tamper-resistant logs)
```

**Why senior-relevant:** ACLs solve "team A and user B both need access but they're different groups" without inventing new groups. `chattr +i` protects critical configs from accidental change/automation drift; `chattr +a` makes append-only audit logs.

**Interview tip:** Use ACLs when the owner/group/other model is too coarse (the `+` in `ls -l` is the tell). Mention `chattr +i` for protecting a config and `+a` for tamper-evident logs — and that immutability blocks *even root* until cleared.

---

# 5. Processes & Signals

## Q6. How do you search for a running process?

**Short answer:** Use `ps -ef | grep <name>` or `pgrep <name>` to find a process by name, and `top`/`htop` to watch processes live.

**Find a specific process:**
```
ps -ef | grep nginx        # list all processes, filter for nginx
pgrep nginx                # print just the PIDs of nginx
pgrep -a nginx             # PIDs + full command line
ps aux | grep java         # BSD-style, shows %CPU and %MEM
```

Tip: `grep` may match itself. Avoid it with:
```
ps -ef | grep [n]ginx      # the [n] trick hides the grep line
pgrep nginx                # pgrep never matches itself
```

**Watch processes live (active monitoring):**
```
top        # built-in, real-time CPU/memory per process
htop       # nicer, colored, scrollable (install if missing)
```

**Useful follow-ups once you have the PID:**
```
kill <PID>        # ask process to stop (SIGTERM)
kill -9 <PID>     # force kill (SIGKILL) — last resort
pkill nginx       # kill by name
ps -p <PID> -o pid,ppid,cmd,%cpu,%mem   # details for one PID
```

```
Flow:
  pgrep / ps -ef | grep   ─►  get PID  ─►  inspect (top/ps -p)  ─►  kill/pkill if needed
```

**Interview tip:** Mention `pgrep` over `ps | grep` because it's cleaner and won't match its own grep line. Use `top`/`htop` for *live* monitoring, `ps` for a *snapshot*.

---

## Q10. How do you check running processes?

> Related to Q6 (searching for a *specific* process). This answer is the **broader view**: which tool to pick and why. New here: `pstree` and the "purpose of each tool" framing.

**Short answer:** Different tools serve different needs — a **snapshot**, **real-time monitoring**, **search by name**, or **process hierarchy**.

| Tool | Purpose | Use when |
|------|---------|----------|
| `ps -ef` | **Snapshot** — one-time full list | You want a frozen view to grep/script |
| `top` | **Real-time** monitoring (built-in) | Watch CPU/memory changing live |
| `htop` | **Real-time**, nicer UI (scroll, colors) | Same as top, more readable (if installed) |
| `pgrep` | **Search by name** → PIDs | Scripting, quick "is it running?" (see Q6) |
| `pstree` | **Hierarchy** — parent/child tree | Understand what spawned what |

**`pstree` — the new one (process family tree):**
```
pstree -p              # show tree with PIDs

systemd(1)─┬─sshd(820)───sshd(1450)───bash(1452)
           ├─nginx(910)─┬─nginx(911)
           │            └─nginx(912)
           └─dockerd(1002)───containerd(1015)
```
This shows **parent → child** relationships — e.g., which process started nginx workers, or that a shell was spawned by sshd.

```
Choose the tool:
  snapshot?      → ps -ef
  live watch?    → top / htop
  find by name?  → pgrep   (details in Q6)
  who spawned?   → pstree
```

**Interview tip:** Don't just list them — say each has a *purpose*: `ps` = snapshot, `top`/`htop` = live, `pgrep` = search, `pstree` = hierarchy. That "right tool for the job" framing is exactly what the question is testing.

---

## Q11. Difference between a process and a thread?

**Short answer:** A **process** is an independent running program with its **own memory**. A **thread** is a lighter unit of execution **inside** a process that **shares** that process's memory with other threads.

| Aspect | Process | Thread |
|--------|---------|--------|
| Memory | Own, isolated address space | Shared within the parent process |
| Weight | Heavy (more resources) | Light (cheap to create) |
| Communication | IPC (pipes, sockets) — slower | Shared memory — fast |
| Crash impact | One crash won't kill others | One bad thread can crash the whole process |
| Created by | `fork()` | thread library (e.g., pthreads) |

```
┌────────── PROCESS (own memory) ──────────┐
│   code | data | heap | files             │
│                                          │
│   ┌────────┐ ┌────────┐ ┌────────┐       │
│   │Thread 1│ │Thread 2│ │Thread 3│  ← share the same memory
│   └────────┘ └────────┘ └────────┘       │
└──────────────────────────────────────────┘

Two processes = two separate boxes (isolated).
Threads       = multiple workers inside ONE box (shared).
```

**Analogy:** A process is a **house** (its own rooms). Threads are **people living in that house** — they share the rooms (memory). Different houses (processes) don't share anything directly.

**Interview tip:** Key trade-off — threads are fast and share memory (great for concurrency) but a bug in one thread can crash the whole process; processes are isolated and safer but heavier and need IPC to talk.

---

## Q12. How do you stop a process?

**Short answer:** Send it a **signal**. `kill <PID>` asks it to stop gracefully (SIGTERM); `kill -9 <PID>` forces it (SIGKILL) when it won't respond.

**Graceful vs forceful:**
```
kill <PID>        # SIGTERM (15) — "please shut down cleanly"  ← default, preferred
kill -9 <PID>     # SIGKILL (9)  — "die now" — OS force-kills it ← last resort
```

**Common signals to know:**
| Signal | Number | Meaning | Can app catch it? |
|--------|--------|---------|-------------------|
| SIGTERM | 15 | Polite stop — lets app clean up | Yes (default `kill`) |
| SIGKILL | 9 | Force kill — cannot be ignored | No — kernel does it |
| SIGHUP | 1 | Reload config (many daemons) | Yes |
| SIGINT | 2 | Interrupt (Ctrl+C in terminal) | Yes |

**Why the order matters:**
```
Try graceful first:   kill <PID>          (app closes files, finishes requests, exits)
Only if stuck:        kill -9 <PID>       (app killed instantly, no cleanup)
```

**By name instead of PID:**
```
pkill nginx           # send SIGTERM to all matching by name
pkill -9 nginx        # force kill by name
killall nginx         # kill all processes named exactly 'nginx'
```

**Interview tip:** Stress that `kill -9` should be used **cautiously** — it gives the app no chance to clean up (flush buffers, close DB connections, release locks), which can cause data corruption or stale lock files. Always try `kill` (SIGTERM) first.

---

## Q65. Zombie vs orphan processes, process states, and nice/renice?

**Short answer:** A **zombie** is a finished process whose parent hasn't read its exit status yet (it's dead but still in the process table). An **orphan** is a live process whose parent died — it gets **re-parented to PID 1 (systemd)**, which cleans it up. **nice/renice** set CPU scheduling priority.

```
Zombie (defunct):  child exits ─► waits for parent to call wait() ─► until then shows <defunct>, state Z
                   Fix: parent must reap it; if parent is buggy, kill the PARENT → PID 1 reaps the zombie.

Orphan:            parent dies first ─► child re-parented to systemd(1) ─► reaped normally (harmless)
```

**Process states (the `S` column in `ps`/`top`):**
| State | Meaning |
|-------|---------|
| R | Running / runnable |
| S | Sleeping (waiting, interruptible) |
| D | Uninterruptible sleep (usually blocked on I/O — can't be killed!) |
| Z | Zombie (defunct) |
| T | Stopped (e.g., Ctrl+Z) |

**nice / renice (priority):** range **-20 (highest priority) to +19 (lowest)**. Higher nice = "nicer" to others = less CPU.
```
nice -n 10 ./batch.sh        # start a low-priority job
renice -n 5 -p 1234          # change priority of a running PID
ionice -c3 ./backup.sh       # low I/O priority (idle class)
```

**Senior gotchas:**
- Many zombies = a **buggy parent** not reaping children (a code bug, not a resource leak you can `kill -9` directly — you kill the parent).
- **State D** (uninterruptible) processes can't be killed even with `-9` — usually stuck on NFS/disk I/O; fix the I/O source.

**Interview tip:** Crisp line — *"zombie = dead but unreaped (parent's fault); orphan = alive but parent gone (adopted by PID 1)."* Mention `D` state can't be killed (I/O wait) and `nice`/`ionice` to keep batch jobs from starving production.

---

# 6. Memory Management

## Q13. How do you check memory usage?

**Short answer:** `free -h` for a quick summary, `top`/`htop` for per-process usage, `vmstat` for live trends, and `/proc/meminfo` for full detail.

| Command | Shows | Use when |
|---------|-------|----------|
| `free -h` | Total / used / free / available (human-readable) | Quick "how much RAM is left?" |
| `top` / `htop` | Memory **per process** | Find which process eats RAM |
| `vmstat 1` | Live memory + swap + CPU, refreshing each second | Watch trends, spot swapping |
| `cat /proc/meminfo` | Full kernel memory detail | Deep dive / scripting |

**`free -h` output explained (the key one):**
```
              total   used   free   shared  buff/cache   available
Mem:           16Gi   6Gi   1Gi     0.3Gi        9Gi         9Gi
Swap:          2Gi    0Gi   2Gi
                                                  │            │
                       cache the kernel can reclaim┘            │
                       what apps can REALLY still use ──────────┘
```

**Big interview point — `free` vs `available`:**
- **free** = totally unused RAM (looks scary-low, that's normal).
- **buff/cache** = RAM used for disk caching — reclaimable anytime.
- **available** = the number that matters — free + reclaimable cache = what new apps can actually use.

> "Low `free` is fine as long as `available` is healthy. Linux uses spare RAM for cache on purpose — unused RAM is wasted RAM."

**Swap:** disk space used as overflow when RAM is full. Heavy swap use = system is slow / under memory pressure. Watch the `si`/`so` (swap in/out) columns in `vmstat`.

```
Check flow:
  free -h  ─►  RAM low & available low?  ─►  top/htop to find the process  ─►  vmstat to see if swapping
```

**Interview tip:** Don't panic at low `free` — look at **available**. Rising swap-in/out or the OOM killer firing (`dmesg | grep -i oom`) are the real signs of memory pressure.

---

## Q14. What is Swap Memory?

> Q13 mentioned swap while checking memory; this explains the concept itself.

**Short answer:** Swap is **disk space used as an extension of RAM**. When physical RAM is full, the kernel moves less-used memory pages to swap to free up RAM. It prevents crashes under memory pressure, but disk is **much slower** than RAM.

```
   RAM full?
      │
      ▼
  Kernel moves "cold" (rarely used) pages ─────► SWAP (on disk)
      │                                            ▲
      └── keeps "hot" pages in fast RAM            │
                                          slow — disk speed, not RAM speed
```

**Key points:**
- Acts as a **safety cushion** — buys time instead of an instant out-of-memory crash.
- **Slow** — swapping heavily makes the whole system sluggish (called *thrashing*).
- **Heavy swap use = warning sign** — usually means not enough RAM, or a memory leak in an app.

**Where it lives:** a swap partition or a swap file (`/swapfile`). Check with:
```
swapon --show      # show active swap devices/files
free -h            # Swap row: total / used / free
```

**Quick facts for interviews:**
| Question | Answer |
|----------|--------|
| Is swap = more RAM? | No — it's slower disk used *like* RAM overflow |
| Should production swap heavily? | No — occasional is fine, constant swap = problem |
| Swappiness? | `vm.swappiness` (0–100) controls how eagerly kernel swaps; servers often set it low (e.g., 10) |
| Kubernetes? | K8s traditionally requires swap **disabled** (`swapoff -a`) for predictable scheduling |

**Interview tip:** Say swap prevents immediate crashes but **excessive swap usage signals insufficient RAM or a memory leak** — the fix is to add RAM or find the leaking process, not to add more swap. Tie it to K8s: that's why `kubeadm` needs swap disabled.

---

## Q15. What is OOM Killer?

**Short answer:** The **Out Of Memory (OOM) Killer** is a Linux kernel mechanism that **kills one or more processes when memory is critically exhausted** (RAM + swap both full), so the whole system doesn't freeze or crash. It's the kernel's last resort to stay alive.

```
 RAM full ─► Swap full ─► kernel cannot free memory
                              │
                              ▼
                     OOM Killer wakes up
                              │
             picks the "worst offender" and kills it
                              │
                              ▼
             memory freed ─► system stays responsive
```

**How it chooses a victim:** each process has an **oom_score** (higher = more likely killed). It roughly favors killing processes that use **lots of memory** and are **not critical**. You can influence it:
```
cat /proc/<PID>/oom_score        # see a process's score
echo -1000 > /proc/<PID>/oom_score_adj   # protect a process (less likely killed)
```

**How to confirm the OOM Killer fired (very common troubleshooting):**
```
dmesg | grep -i "out of memory"
dmesg -T | grep -i oom
journalctl -k | grep -i oom
```
You'll see lines like `Out of memory: Killed process 1234 (java)`.

**In Kubernetes:** if a container exceeds its memory **limit**, it gets killed and shows status **`OOMKilled`** — same idea, enforced per-container via cgroups.

**Interview tip:** Frame it as the kernel's **survival mechanism** — it sacrifices one process to keep the system usable. In real incidents you'll spot it via `dmesg`/`journalctl -k`, then fix the root cause (raise memory, tune limits, or fix a memory leak). Tie it to K8s `OOMKilled` for bonus points.

---

## Q66. Virtual memory, paging, and the page cache?

**Short answer:** Each process sees its own **virtual address space**; the kernel maps virtual pages to physical RAM (or disk) via the MMU. **Paging** moves pages between RAM and disk. The **page cache** uses free RAM to cache disk data for speed.

```
Process virtual memory ──(MMU maps pages)──► Physical RAM
                                   │
                          not in RAM? = PAGE FAULT
                                   │
                          load from disk (or swap)  ← slow

Free RAM is used as PAGE CACHE (cached file data) → that's the "buff/cache" in free -h (Q13)
```

**Key ideas for 10+ YOE:**
- **Virtual memory** lets processes be isolated and use more address space than physical RAM.
- **Page fault**: accessing a page not in RAM → kernel fetches it. *Minor* fault = already in memory/cache; *major* fault = must read disk (slow — watch these).
- **Page cache**: Linux caches file reads/writes in spare RAM → repeated reads are RAM-fast. This is *why* `free` shows low "free" but high "available" (Q13).
- **Dirty pages**: modified cached pages not yet written to disk; flushed by the kernel (`sync`, `vm.dirty_ratio`).

**Interview tip:** Connect to Q13 — "buff/cache is the page cache; it's reclaimable, so low free RAM is normal." Mention **major page faults** as a memory-pressure signal (`ps -o min_flt,maj_flt` / `sar -B`).

---

## Q67. Memory overcommit and swappiness — what would you tune?

**Short answer:** Linux **overcommits** — it hands out more virtual memory than physically exists, betting not all is used at once. **swappiness** controls how aggressively it swaps. Both are `sysctl` knobs you tune for the workload.

| sysctl | What it does | Typical tuning |
|--------|--------------|----------------|
| `vm.overcommit_memory` | 0=heuristic, 1=always allow, 2=strict (no overcommit) | 2 + `overcommit_ratio` for memory-critical DBs |
| `vm.swappiness` | 0–100: how eagerly to swap (higher = swap sooner) | Servers/DBs: low (1–10); default 60 |
| `vm.dirty_ratio` | % RAM of dirty pages before forced writeback | Lower for latency-sensitive I/O |

```
Set live + persist:
  sysctl vm.swappiness=10                 # now
  echo 'vm.swappiness=10' >> /etc/sysctl.d/99-tuning.conf   # persist across reboot
  sysctl -p                                # reload
```

**Why it matters:**
- **Overcommit** is why an app can `malloc` huge amounts yet the box is fine — until real use hits the ceiling and the **OOM killer** fires (Q15). DB vendors often disable overcommit for predictability.
- **Low swappiness** keeps hot app memory in RAM instead of swapping (avoids latency); **Kubernetes** disables swap entirely (ties to Q14).

**Interview tip:** Name the three knobs (`overcommit_memory`, `swappiness`, `dirty_ratio`), how to persist via `/etc/sysctl.d/`, and connect overcommit → OOM (Q15) and low-swappiness → DB/latency tuning. That's the senior tuning story.

---

# 7. Boot & Init Systems

## Q32. Walk through the Linux boot process end to end.

**Short answer:** Firmware → bootloader → kernel → init (systemd) → targets/services.

```
Power on
  │
  ▼
1. FIRMWARE (BIOS or UEFI)  → POST, finds boot device
  │                           UEFI reads EFI System Partition (ESP)
  ▼
2. BOOTLOADER (GRUB2)       → loads kernel + initramfs, passes cmdline
  │
  ▼
3. KERNEL                   → mounts initramfs (temp rootfs), loads drivers
  │                           for real disk, then mounts real root (/)
  ▼
4. initramfs → pivot_root   → hands off to /sbin/init
  │
  ▼
5. init = systemd (PID 1)   → activates default.target (e.g., multi-user)
  │                           starts services in dependency order
  ▼
6. System ready (login / services up)
```

**Key facts for 10+ YOE:**
- **initramfs** exists to load drivers needed to *find* the real root filesystem (e.g., LVM, RAID, encrypted disk). Without it the kernel couldn't mount root on complex storage.
- Kernel cmdline (from GRUB) sets `root=`, `ro`, `console=`, etc. — visible via `cat /proc/cmdline`.
- PID 1 (`systemd`) is the ancestor of every process; if it dies, the system panics.

**Interview tip:** Name the five stages crisply (firmware → GRUB → kernel → initramfs → systemd) and explain *why initramfs exists* — that's the senior signal.

---

## Q33. BIOS vs UEFI — why does it matter operationally?

**Short answer:** Both are firmware that start the boot, but UEFI is the modern replacement: it understands filesystems (the ESP), supports disks >2 TB via GPT, boots faster, and enables **Secure Boot**.

| | BIOS (legacy) | UEFI |
|--|---------------|------|
| Partitioning | MBR (max 2 TB, 4 primary) | GPT (huge disks, many partitions) |
| Boot data | MBR boot sector (512 bytes) | Files on the ESP (FAT32) |
| Secure Boot | No | Yes (signed bootloader/kernel) |
| Speed | Slower | Faster |

**Operational impact:** cloud images and modern servers are UEFI+GPT. Cloning a BIOS disk image onto a UEFI instance (or vice versa) fails to boot — the boot method must match. Secure Boot can block **unsigned kernel modules** (e.g., custom drivers), a real gotcha.

**Interview tip:** Tie it to reality — GPT for large disks, and Secure Boot blocking unsigned modules is the operational surprise.

---

## Q34. What is initramfs and when would you rebuild it?

**Short answer:** initramfs is a small temporary root filesystem loaded into RAM by the bootloader. Its job: load the **drivers/modules needed to mount the real root** (LVM, RAID, LUKS encryption, iSCSI), then pivot to the real `/`.

**Rebuild it when** you change something the *early* boot depends on:
- Added/changed storage layout (LVM, RAID, encrypted root)
- New disk driver / kernel module needed at boot
- After certain kernel updates (usually automatic)

```
Rebuild:
  Debian/Ubuntu:  update-initramfs -u
  RHEL/CentOS:    dracut -f
```

**Why it bites:** a wrong initramfs (missing storage driver) → kernel can't find root → drops to an emergency shell or "Cannot mount root fs" panic. Classic cause of a server that won't boot after a storage/kernel change.

**Interview tip:** Frame it as "the bridge that lets the kernel reach a complex root disk" — and rebuilding it after storage/kernel changes.

---

## Q35. A server hangs on boot / won't come up. How do you recover?

**Short answer:** Get to a shell (GRUB → recovery/emergency), find the failing stage, fix it, then continue. Work from earliest stage to latest.

```
1. GRUB menu → edit entry → add to kernel line:
     systemd.unit=rescue.target      (single-user-ish, minimal services)
     or  emergency.target            (barest shell, root ro)
     or  init=/bin/bash              (bypass init entirely)
2. Remount root writable:  mount -o remount,rw /
3. Diagnose:
     journalctl -xb            → what failed this boot
     systemctl --failed        → failed units
     cat /proc/cmdline         → wrong root= / bad param?
     check /etc/fstab          → bad mount blocks boot (very common!)
4. Fix (fstab typo, missing disk → add 'nofail', rebuild initramfs, etc.)
5. Reboot
```

**Most common real causes:** bad `/etc/fstab` entry (a missing/renamed disk blocks boot — use `nofail`), broken initramfs after a change, full `/` , or a failing service in the boot path.

**Interview tip:** Mention `/etc/fstab` first — a bad entry is the #1 "server won't boot" cause, and adding `nofail` prevents it. Show you know `rescue` vs `emergency` targets.

---

## Q36. SysVinit vs systemd — what changed and why?

**Short answer:** SysVinit started services **sequentially** via shell scripts in a fixed runlevel order. systemd starts them **in parallel** based on a **dependency graph**, with socket activation, and manages the full service lifecycle (restart, logging, resource limits).

| | SysVinit | systemd |
|--|----------|---------|
| Startup | Sequential scripts (`/etc/init.d`) | Parallel, dependency-based |
| Config | Shell scripts + runlevels | Declarative unit files |
| Speed | Slower | Faster (parallel + on-demand) |
| Features | Start/stop | Restart policies, cgroups, timers, journald |
| "Runlevels" | 0–6 | targets (e.g., `multi-user.target`) |

```
runlevel 3  ≈  multi-user.target   (CLI, networking, no GUI)
runlevel 5  ≈  graphical.target    (+ GUI)
runlevel 0/6 ≈ poweroff / reboot target
```

**Why the change:** parallel startup and dependency ordering made boots faster and more reliable; unit files are declarative and consistent; built-in cgroup integration lets systemd track/limit every service.

**Interview tip:** Key line — *"SysVinit = sequential shell scripts by runlevel; systemd = parallel, dependency-driven, declarative units with lifecycle management."* Map runlevels → targets to show you bridge old and new.

---

# 8. systemd & Services

> Concept of *why* systemd replaced SysVinit is in Q36; rescue/emergency targets in Q35. This topic is about **operating** systemd day to day.

## Q37. Anatomy of a systemd unit file — walk through one.

**Short answer:** A unit file declaratively describes a service: what to run, when, how to restart, and what it depends on. Lives in `/etc/systemd/system/` (admin) or `/lib/systemd/system/` (packages).

```ini
[Unit]
Description=My App
After=network-online.target        # ordering: start after network is up
Wants=network-online.target        # weak dependency

[Service]
ExecStart=/usr/bin/myapp --config /etc/myapp.yaml
Restart=on-failure                 # auto-restart policy
RestartSec=5
User=appuser                       # drop privileges
MemoryMax=512M                     # cgroup resource limit
EnvironmentFile=/etc/myapp.env

[Install]
WantedBy=multi-user.target         # where it hooks when 'enabled'
```

**Three sections:** `[Unit]` (metadata + ordering/deps), `[Service]` (how to run it), `[Install]` (what `enable` links it to).

**After editing always:** `systemctl daemon-reload` (reloads unit definitions), then `restart`.

**Interview tip:** Call out `Restart=on-failure`, running as a non-root `User=`, and `daemon-reload` after edits — those are the operational senior signals.

---

## Q38. `systemctl enable` vs `start` (and stop/disable/mask)?

**Short answer:** `start` acts **now**; `enable` sets **boot behavior**. They're independent.

| Command | Effect |
|---------|--------|
| `start` | Runs the service **now** (not persistent) |
| `enable` | Starts it **on boot** (creates symlink; doesn't start now) |
| `enable --now` | Both: enable + start immediately |
| `stop` / `disable` | Stop now / don't start at boot |
| `mask` | **Fully block** it — symlinks to /dev/null, can't even be started manually |

```
enable  ──► persists across reboot   (boot-time)
start   ──► affects current session  (right now)
   → you often want BOTH: systemctl enable --now myapp
```

**`mask` gotcha:** masking is stronger than disable — a masked unit *cannot* be started until `unmask`. Useful to guarantee something (e.g., a conflicting service) never runs.

**Interview tip:** The classic mistake is `start` without `enable` → service works until the next reboot, then it's gone. Mention `mask` for "make absolutely sure this never starts."

---

## Q39. How do you troubleshoot a failed systemd service?

**Short answer:** `status` → `journalctl -u` → check config/deps → fix → `daemon-reload` + restart.

```
systemctl status myapp        → active/failed, recent log lines, PID, exit code
systemctl --failed            → list everything that failed
journalctl -u myapp -n 100    → last 100 log lines for this unit
journalctl -u myapp -f        → follow live
journalctl -u myapp --since "10 min ago"
systemd-analyze verify myapp.service   → validate the unit file
systemctl list-dependencies myapp      → dependency issues
```

**Read the exit clue in `status`:**
- `code=exited, status=203/EXEC` → binary/path wrong or not executable
- `status=200/USER` → the `User=` doesn't exist
- `Result: timeout` → didn't signal ready in time (Type/`TimeoutStartSec`)
- repeated restart loop → `Restart=` + a crash; fix root cause, not the policy

**Interview tip:** Lead with `journalctl -u <unit>` — most people forget services log to the journal, not always to `/var/log`. Mention decoding the `status=` exit code.

---

## Q40. systemd timers vs cron — when do you pick which?

> Cron basics are in Q21/Q22. This is the *comparison and when to choose*.

**Short answer:** Both schedule jobs. **Timers** are systemd units — better logging, dependencies, `Persistent=` for missed runs, and randomized delays. **Cron** is simpler and universal.

| | cron | systemd timer |
|--|------|---------------|
| Setup | one crontab line | timer unit + service unit |
| Logging | you redirect output | automatic in journald |
| Missed runs (host off) | skipped | `Persistent=true` runs on next boot |
| Jitter/spread | manual | `RandomizedDelaySec` built-in |
| Dependencies | none | can require network/mounts first |

```
myjob.timer  ──triggers──►  myjob.service
OnCalendar=*-*-* 02:00:00     (like cron's 0 2 * * *)
Persistent=true               (catch up if missed)
```

**Pick:** cron for quick, portable, simple jobs; timers when you need **logging, catch-up, dependency ordering, or jitter** (e.g., stagger 500 nodes hitting a server).

**Interview tip:** The senior differentiators: `Persistent=true` (missed-run catch-up) and `RandomizedDelaySec` (avoid thundering-herd) — cron can't do these cleanly.

---

## Q41. What is journald, and how does it relate to /var/log and rsyslog?

**Short answer:** `journald` is systemd's logging service. It captures stdout/stderr of every unit plus kernel/syslog messages into a **structured, indexed binary journal**, queried with `journalctl`. It can forward to `rsyslog` for traditional text files in `/var/log`.

```
services / kernel ──► journald (binary, indexed)
                          │  query: journalctl -u, -p, --since, -k, -b
                          └─(optional forward)─► rsyslog ──► /var/log/*.log ──► ship offsite
```

**Why structured matters:** filter by unit (`-u`), priority (`-p err`), time (`--since`), boot (`-b`), kernel (`-k`) — no `grep` gymnastics.

**Persistence gotcha:** by default some distros keep the journal in `/run` (RAM) → **lost on reboot**. Make it persistent: create `/var/log/journal/` and set `Storage=persistent` in `journald.conf`. Cap size with `SystemMaxUse=`.

**Interview tip:** Explain the split — journald = structured/queryable, rsyslog = text files + forwarding to central logging. Flag the persistent-storage config, a common "my logs vanished after reboot" surprise. (Broader logging in the Logging topic.)

---

# 9. Networking

## Q19. What is SSH?

**Short answer:** SSH (**Secure Shell**) is a protocol for **securely logging into and administering remote systems** over an untrusted network. It **encrypts** everything between client and server, replacing old insecure tools like **Telnet** (which sent passwords in plain text).

```
  You (client)                              Remote server
  ┌──────────┐    encrypted tunnel (port 22)   ┌──────────┐
  │  ssh ────┼════════════════════════════════►│  sshd    │
  │          │◄════════════════════════════════┼─ shell   │
  └──────────┘   nobody in between can read it  └──────────┘
```

**Key facts:**
- Default port: **22** (TCP)
- Client command: `ssh user@host`
- Server side: the `sshd` daemon (managed via `systemctl status sshd`)
- Encrypts the whole session → passwords, commands, output all protected

**Basic usage:**
```
ssh ubuntu@10.0.0.5              # log in as ubuntu
ssh -i mykey.pem ubuntu@host     # log in using a private key
ssh -p 2222 user@host            # non-default port
scp file.txt user@host:/tmp/     # copy a file over SSH
```

**Why it replaced Telnet:**
| | Telnet | SSH |
|--|--------|-----|
| Encryption | None (plain text!) | Fully encrypted |
| Passwords | Sent in the clear | Protected |
| Auth options | Password only | Password **or** key-based |
| Use today | Deprecated / unsafe | Standard for remote admin |

**Interview tip:** One line — *"SSH is the encrypted, secure replacement for Telnet for remote server administration, running on port 22."* Bonus: mention **key-based auth** is preferred over passwords (Q20).

---

## Q20. Difference between Password Authentication and SSH Keys?

**Short answer:** Password auth proves who you are with a **secret you type** (can be guessed/brute-forced). SSH keys use a **key pair** — a **private key** you keep and a **public key** on the server — which is far more secure and needs no typing.

| | Password Auth | SSH Keys |
|--|---------------|----------|
| What you present | A password | A private key (math proof) |
| Stored on server | Password hash | Your **public** key only |
| Brute-force risk | High (guessable) | Practically none |
| Convenience | Type it every time | Automatic (no typing) |
| Automation (CI/CD, Ansible) | Hard/insecure | Ideal |
| Best practice | Disable in production | Preferred method |

**How key-based auth works:**
```
  Client                                   Server
  ┌───────────────┐                        ┌────────────────────────┐
  │ private key   │  1. "I want in"        │ has your PUBLIC key in  │
  │ (id_rsa)      │ ─────────────────────► │ ~/.ssh/authorized_keys  │
  │  SECRET,      │  2. server sends a      │                        │
  │  never leaves │     challenge          │                        │
  │  your machine │ ◄───────────────────── │                        │
  │               │  3. sign with private  │ 4. verify with public   │
  │               │ ─────────────────────► │    key ─► access ✔      │
  └───────────────┘                        └────────────────────────┘
   Private key NEVER travels over the network.
```

**Set up keys (quick):**
```
ssh-keygen -t ed25519                 # generate a key pair
ssh-copy-id user@host                 # push public key to server's authorized_keys
ssh user@host                         # now logs in with no password
chmod 600 ~/.ssh/id_ed25519           # private key must be owner-only (see Q8)
```

**Hardening (production):** in `/etc/ssh/sshd_config` set `PasswordAuthentication no` and `PermitRootLogin no`, then `systemctl restart sshd` (deeper hardening in Q79).

**Interview tip:** Say keys are **more secure and automation-friendly** — the private key never leaves your machine, only the public key sits on the server. In production you **disable password auth** and use keys only. Ties back to `chmod 600` on the private key (Q8).

---

## Q42. `ip`/`ss` vs the old `ifconfig`/`netstat` — what do you use and why?

**Short answer:** The `net-tools` commands (`ifconfig`, `netstat`, `route`) are **deprecated**; the modern `iproute2` suite (`ip`, `ss`) is faster, shows more (multiple IPs, namespaces), and is what ships by default now.

| Task | Old (deprecated) | Modern |
|------|------------------|--------|
| Show interfaces/IPs | `ifconfig` | `ip addr` (`ip a`) |
| Show routes | `route -n` | `ip route` (`ip r`) |
| Show sockets/ports | `netstat -tulpn` | `ss -tulpn` |
| ARP table | `arp -n` | `ip neigh` |

```
ip a                 → interfaces + IP addresses
ip r                 → routing table (default gateway)
ss -tulpn            → listening TCP/UDP ports + owning process
ss -tan state established   → active connections
```

**Why `ss` beats `netstat`:** it reads kernel socket data directly (much faster on busy hosts with thousands of connections) and gives richer filtering.

**Interview tip:** Say you've moved to `ip`/`ss`; `ifconfig` may not even be installed on modern minimal images. `ss -tulpn` is the go-to for "what's listening and who owns it."

---

## Q43. How does Linux resolve a hostname? (DNS resolution order)

**Short answer:** The resolver follows **NSS** (`/etc/nsswitch.conf` `hosts:` line) — usually `/etc/hosts` first, then DNS. On modern systems `systemd-resolved` sits in the middle.

```
app calls getaddrinfo("api.example.com")
        │
        ▼
/etc/nsswitch.conf  hosts: files dns
        │
   1. /etc/hosts        → static override? use it, stop.
        │ (no match)
   2. DNS resolver:
        systemd-resolved (127.0.0.53)  or  /etc/resolv.conf nameservers
        │
        ▼
   returns IP (respecting TTL cache)
```

**Debug tools:**
```
resolvectl query host      # systemd-resolved (shows which server + cache)
dig host / dig +short host # raw DNS query, see the actual answer/TTL
getent hosts host          # follows the FULL nsswitch path (not just DNS)
```

**Senior gotcha:** `dig` queries DNS *directly* and may work while the **app fails**, because the app honors `/etc/hosts` and NSS. Use `getent hosts` to test the real resolution path. Also watch `/etc/resolv.conf` being a symlink to systemd-resolved.

**Interview tip:** Mention `/etc/hosts` → NSS → DNS order, and that `getent hosts` (not just `dig`) reflects what the app actually sees.

---

## Q44. How do you check if a port is open / a service is reachable?

**Short answer:** Split it into three questions: is my app **listening locally**, can I **reach the remote port**, and is a **firewall** blocking it?

```
1. LISTENING locally?      ss -tulpn | grep :8080     (bound? which process?)
2. REACH remote port?      nc -zv host 8080           (TCP connect test)
                           curl -v http://host:8080/  (app-layer + response)
3. DNS ok but no connect?  ping host (ICMP), then nc  (isolate DNS vs TCP)
4. FIREWALL?               local: iptables -L / nft list ruleset / ufw status
                           cloud: security group / NACL rules
```

**Layered reasoning (the senior part):**
```
resolves? ──no──► DNS problem (Q43)
   │yes
reaches TCP? ──no──► firewall / SG / not listening / wrong port
   │yes
app responds? ──no──► app-level issue (backend, cert, 5xx)
```

**Interview tip:** `nc -zv` = pure TCP reachability, `curl -v` = application layer (adds TLS + HTTP status). If `ss` shows the app bound to `127.0.0.1` instead of `0.0.0.0`, it'll never accept remote traffic — a very common "port open but unreachable" cause.

---

## Q45. A service is unreachable across the network. How do you troubleshoot? (Scenario)

**Short answer:** Walk the path **bottom-up**: local listen → local firewall → network/route → remote firewall/SG → DNS → app. Isolate the layer, don't guess.

```
Client ──► DNS ──► route/gateway ──► [cloud SG/NACL] ──► server firewall ──► app(listening?)

Check in order:
  1. On server:  ss -tulpn        → app actually listening? on 0.0.0.0 or 127.0.0.1?
  2. On server:  firewall rules    → iptables/nftables/ufw allow the port?
  3. Cloud:      Security Group / NACL open for source + port?
  4. From client: nc -zv server port   → TCP reaches?
  5. Path:       ping / traceroute / mtr server  → where do packets die?
  6. DNS:        getent hosts server        → resolving to the right IP? (Q43)
  7. App:        curl -v + server logs      → 5xx / TLS / backend?
```

**Common real causes:** app bound to `127.0.0.1`, security group not opened, firewall drop, wrong DNS record, or MTU/asymmetric routing (rare but nasty — `mtr` reveals it).

**Interview tip:** The winning line — *"I isolate the layer: local listen → local firewall → cloud SG → routing → DNS → app. `mtr` shows where packets drop; `ss` shows if it's even listening on the right interface."* Bind-to-localhost and closed security groups are the top two culprits.

---

# 10. Package Management & Patching

## Q46. apt vs yum/dnf — high-level vs low-level tools?

**Short answer:** Two package worlds: **Debian/Ubuntu** (`.deb`, `apt` over `dpkg`) and **RHEL/CentOS/Fedora** (`.rpm`, `dnf`/`yum` over `rpm`). The high-level tool resolves **dependencies + repos**; the low-level tool installs a single file.

| | Debian/Ubuntu | RHEL/Fedora |
|--|---------------|-------------|
| Package | `.deb` | `.rpm` |
| High-level (deps + repos) | `apt` | `dnf` (yum = older) |
| Low-level (one file) | `dpkg` | `rpm` |
| Install | `apt install nginx` | `dnf install nginx` |
| Repo config | `/etc/apt/sources.list(.d)` | `/etc/yum.repos.d/*.repo` |

```
apt/dnf  ─► resolves dependencies, downloads from repos ─► calls dpkg/rpm
dpkg/rpm ─► installs ONE local file, does NOT fetch dependencies
```

**Interview tip:** The key distinction — `apt`/`dnf` handle **dependency resolution and repositories**; `dpkg`/`rpm` operate on a single local package and will fail/complain on missing deps. That's why you rarely use the low-level tool directly.

---

## Q47. What happens internally when you `apt install` a package?

**Short answer:** It reads the local package index, resolves dependencies, downloads `.deb`s, then unpacks + configures each in order.

```
apt update        → refresh local index from repo metadata (Release/Packages)
apt install nginx →
   1. resolve dependency tree (nginx needs libX, libY…)
   2. download .deb files to /var/cache/apt/archives
   3. verify GPG signatures (trusted repo keys)
   4. dpkg --unpack  → lay files down
   5. dpkg --configure → run post-install scripts, start service
```

**Senior points:**
- `apt update` ≠ `apt upgrade`. `update` refreshes the **index**; `upgrade` installs newer versions.
- Packages are **GPG-signed**; an untrusted/expired repo key makes installs fail (security by design).
- Post-install (`postinst`) scripts create users, set up systemd units, etc. — a failed script leaves a package "half-configured" (`dpkg --configure -a` to fix).

**Interview tip:** Mention GPG signature verification and the update-vs-upgrade distinction — common confusion that signals depth.

---

## Q48. How do you patch servers safely at scale? (Kernel updates & reboots)

**Short answer:** Patch in **waves** (canary → batches), automate with config management, and know which patches need a **reboot** (kernel, glibc, systemd) vs not.

```
Strategy:
  1. Stage in non-prod → validate
  2. Canary: patch 1-5% of fleet, watch metrics/health
  3. Roll in batches behind LB, drain → patch → reboot → rejoin
  4. Automate: Ansible / unattended-upgrades / AWS SSM Patch Manager
  5. Maintain rollback (snapshot / AMI / pinned versions)

Reboot needed?  needs-restart -r   (RHEL)   /   /var/run/reboot-required (Debian)
```

**Kernel updates:** a new kernel only takes effect **after reboot**. To avoid downtime, either schedule rolling reboots behind a load balancer, or use **live patching** (Ubuntu Livepatch, RHEL kpatch/ksplice) to apply critical kernel CVEs **without rebooting** — buys time until a maintenance window.

**Interview tip:** Senior signals: **canary + batched rollout behind an LB**, checking `needs-restart`/`reboot-required`, and **livepatch** for zero-downtime critical kernel fixes. Always keep a rollback (snapshot/AMI).

---

## Q49. How do you pin a version or roll back a bad package?

**Short answer:** Pin to control *which* version installs; roll back by installing the previous known-good version (and holding it).

```
Pin / hold (don't auto-upgrade):
  Ubuntu:  apt-mark hold kubelet          (or apt preferences in /etc/apt/preferences.d)
  RHEL:    dnf versionlock add kubelet     (dnf-plugin-versionlock)

Install a specific version:
  apt install nginx=1.24.0-1
  dnf install nginx-1.24.0

Roll back:
  Ubuntu:  apt install nginx=<old-version>   (downgrade), then apt-mark hold
  RHEL:    dnf downgrade nginx   OR   dnf history undo <ID>   ← nice: undoes a transaction
```

**RHEL edge:** `dnf history` logs every transaction; `dnf history undo <ID>` reverts a whole install/update set — cleaner than manual downgrades.

**Interview tip:** Mention **holding/versionlock** (this is exactly why a kubeadm playbook pins `kubelet`/`kubeadm`), specific-version syntax, and `dnf history undo` for transactional rollback.

---

## Q50. dpkg/rpm low-level: querying and fixing broken packages?

**Short answer:** Use the low-level tools to **inspect** what's installed and **repair** a broken state when the high-level tool is stuck.

```
Query (what/where):
  dpkg -l | grep nginx        rpm -qa | grep nginx      # is it installed?
  dpkg -L nginx               rpm -ql nginx             # files it owns
  dpkg -S /usr/sbin/nginx     rpm -qf /usr/sbin/nginx   # which package owns this file?

Repair:
  dpkg --configure -a         # finish half-configured packages
  apt --fix-broken install    # resolve broken dependency state
  rpm --rebuilddb             # rebuild corrupted rpm database
```

**When it matters:** a disk-full or interrupted install leaves packages half-configured; `dpkg --configure -a` / `apt --fix-broken install` recovers without reinstalling. `dpkg -S` / `rpm -qf` (which package owns a file) is gold when auditing or debugging config drift.

**Interview tip:** The reverse lookup — *"which package installed this file?"* (`dpkg -S` / `rpm -qf`) — and recovering a broken package DB show hands-on ops experience beyond `apt install`.

---

# 11. Logging

> journald vs rsyslog concept is in Q41; disk-full-from-logs and deleted-open-log-file are in Q18/Q27. This topic is about the log layout, querying, rotation, centralization, and log-driven debugging.

## Q51. What lives in /var/log — the key files?

**Short answer:** `/var/log` holds system and service logs. Know the handful you actually open during incidents.

| File / dir | Contains |
|------------|----------|
| `/var/log/syslog` (Debian) / `messages` (RHEL) | General system messages |
| `/var/log/auth.log` (Debian) / `secure` (RHEL) | Auth: logins, sudo, sshd |
| `/var/log/kern.log` | Kernel messages (also `dmesg`) |
| `/var/log/dmesg` | Boot-time kernel ring buffer |
| `/var/log/journal/` | systemd binary journal (if persistent) |
| `/var/log/<service>/` | App/service logs (nginx, mysql, etc.) |
| `/var/log/cloud-init.log` | Cloud instance first-boot provisioning |

**Interview tip:** For a security/login issue → `auth.log`/`secure`. For hardware/OOM/driver → `kern.log`/`dmesg`. For "why did my cloud VM not configure" → `cloud-init.log`. Naming the right file fast is the senior signal.

---

## Q52. journalctl in practice — the queries that matter.

> Concept of journald is in Q41; here are the *operational* queries.

**Short answer:** `journalctl` filters the structured journal by unit, time, priority, and boot — no `grep` needed.

```
journalctl -u nginx -f                 # follow one service live
journalctl -u nginx --since "1 hour ago" --until "10 min ago"
journalctl -p err -b                   # errors+ this boot (emerg..err)
journalctl -k                          # kernel messages only
journalctl -b -1                       # the PREVIOUS boot (why did it crash?)
journalctl _PID=1234                   # by field (structured metadata)
journalctl --disk-usage                # how big is the journal?
journalctl --vacuum-time=7d            # trim to last 7 days
```

**Priority levels (`-p`):** `emerg(0) alert crit err(3) warning notice info debug(7)`.

**Senior move:** `journalctl -b -1 -p err` = "show me the errors from the boot that crashed" — invaluable for post-mortem after an unexpected reboot.

**Interview tip:** Highlight `-b -1` (previous boot) and `-p err` filtering — these separate people who *use* the journal from people who only `tail` text files.

---

## Q53. What is logrotate and why is it critical?

**Short answer:** `logrotate` rotates, compresses, and deletes old logs on a schedule so they don't fill the disk. Config in `/etc/logrotate.conf` and `/etc/logrotate.d/<app>`.

```
/var/log/myapp/*.log {
    daily                # rotate daily
    rotate 14            # keep 14 old files
    compress             # gzip old logs
    missingok
    notifempty
    copytruncate         # copy then truncate in place (see gotcha below)
}
```

**The critical gotcha (ties to Q27):** if you rotate by **renaming** a file an app still has open, the app keeps writing to the old (now-renamed) inode → disk fills invisibly. Two fixes:
- `copytruncate` — copy the file then truncate the original in place (app's FD stays valid).
- `postrotate ... systemctl reload myapp` — signal the app to reopen its log file.

**Interview tip:** Connect it to Q18/Q27 — logrotate is the *prevention* for disk-full-by-logs, and `copytruncate` vs reopen-on-signal is the detail that prevents the deleted-open-file trap.

---

## Q54. How do you centralize logs across many servers?

**Short answer:** Ship logs off each host to a central store so you can search across the fleet and survive node loss. Common stacks: **rsyslog/syslog-ng forwarding**, **ELK/OpenSearch** (Elasticsearch + Logstash/Beats + Kibana), or **Loki + Promtail + Grafana**.

```
Each node:  app/journald ──► shipper (Fluent Bit / Promtail / rsyslog fwd)
                                   │
                                   ▼
                      Central store (Elasticsearch / Loki / S3)
                                   │
                                   ▼
                      Query UI (Kibana / Grafana)
```

**Why it matters (senior framing):**
- **Search across N servers** in one place instead of SSH-ing each.
- **Survives node death** — logs aren't lost when an instance is terminated (huge for autoscaling/containers).
- **Security** — an attacker who wipes local logs can't erase the shipped copy.
- **Retention/compliance** — central retention + archival (e.g., S3).

**Interview tip:** Mention the two modern stacks (ELK vs Loki — Loki indexes labels not full text, so it's cheaper), and that in cloud/K8s you ship with **Fluent Bit** to CloudWatch/OpenSearch/Loki. Tie to why ephemeral nodes *must* ship logs.

---

## Q55. Log-based troubleshooting — how do you correlate an incident? (Scenario)

**Short answer:** Establish a **timeline**, correlate across sources (app + system + auth + kernel), and pivot on a **request/trace ID**.

```
1. WHEN did it start?     journalctl --since "<time>"  ;  check monitoring/alerts
2. WHAT changed?          deploy time? config? cert expiry? (correlate to step 1)
3. WHERE?                 app logs (errors/stack) + syslog + auth.log + dmesg(OOM/HW)
4. CORRELATE:             grep the same request/trace ID across services
5. SYSTEM vs APP?         OOM in dmesg? disk full? or app 5xx/DB timeout?
6. CONFIRM root cause → fix → verify logs go quiet
```

**Techniques:** filter by time window first (narrows noise), then by severity; use a **correlation/trace ID** to follow one request across microservices; check **auth.log** if it might be access-related; check **dmesg/kern.log** for OOM or hardware.

**Interview tip:** The senior habit — *"I start from a timeline and correlate app, system, and kernel logs around that window, then pivot on a trace ID."* Mention that centralized logging (Q54) makes cross-service correlation possible; on a single box you're stuck grepping per service.

---

# 12. Users, Groups & sudo

## Q31. Difference between logging in as root and using sudo?

**Short answer:** `root` is the all-powerful superuser (**UID 0**) with full control. `sudo` lets a **normal user run specific commands as root** (or another user) — only when needed, with **logging** and **policy**. Best practice is: personal account + `sudo`, not shared root login.

| Approach | Risk | Typical use |
|----------|------|-------------|
| **root login** | No per-admin attribution; one typo can break the whole box; shared password | Emergency recovery console only — discouraged for daily SSH |
| **sudo** | Elevated only when needed; every action is logged | Day-to-day admin on Ubuntu, RHEL, most enterprises |

**Why sudo wins (the point being tested):**
```
Shared root login:                     Personal account + sudo:
  who did what?  ─► unknown              who did what?  ─► logged per user
  always full power ─► one typo = boom    power only when needed, scoped
  shared password ─► leak = full access   own account ─► revoke individually
```

- **Attribution** — logs show *which person* ran the command.
- **Least privilege** — grant only the commands a user needs.
- **Safety** — you're not sitting in a root shell where every typo is dangerous.

**Where auth events land:**
```
Ubuntu/Debian:  /var/log/auth.log
RHEL/CentOS:    /var/log/secure
Either:         journalctl _COMM=sudo   (or search for sshd)
```

**Least-privilege example** (`/etc/sudoers.d/deploy`):
```
deploy ALL=(ALL) NOPASSWD: /bin/systemctl restart myapp.service
```
This grants the `deploy` user rights to restart **one service** — not a full root shell. (Edit sudoers with `visudo`, which checks syntax before saving.)

**Strong answer to say:** *"I avoid routine root login. I use my own account with sudo so actions are attributable, I can grant narrow permissions, and mistakes are less likely to take down the entire system."*

**Interview tip:** Stress **attribution + least privilege + safety**. Bonus: mention editing sudoers with **`visudo`** (prevents lockout from syntax errors) and scoping rules per-command in `/etc/sudoers.d/` rather than giving blanket `ALL=(ALL) ALL`.

---

## Q68. Anatomy of /etc/passwd, /etc/shadow, and /etc/group?

**Short answer:** These three files define accounts. `/etc/passwd` = account info (world-readable), `/etc/shadow` = password hashes (root-only), `/etc/group` = group membership.

```
/etc/passwd:  alice:x:1000:1000:Alice:/home/alice:/bin/bash
              │     │  │    │    │      │           └ login shell
              │     │  │    │    │      └ home dir
              │     │  │    │    └ GECOS (full name)
              │     │  │    └ GID (primary group)
              │     │  └ UID
              │     └ 'x' = password is in /etc/shadow
              └ username

/etc/shadow:  alice:$6$salt$hash:19700:0:99999:7:::
              │     │            └ password aging fields (last change, min, max, warn…)
              │     └ hashed password ($6$=SHA-512;  '!' or '*' = login disabled)
              └ username

/etc/group:   devs:x:1001:alice,bob    ← group name : GID : members
```

**Senior points:**
- `x` in passwd means the real hash lives in **shadow** (readable only by root) — separation so passwd can stay world-readable.
- `!`/`*` in the shadow hash field = account **cannot password-login** (common for service accounts).
- Use `getent passwd alice` (not just `cat`) — it also resolves users from LDAP/SSSD, not only local files.

**Interview tip:** Know the passwd field order (user:x:UID:GID:info:home:shell) and that shadow holds the hash + aging. Mention `getent` for directory-backed users.

---

## Q69. useradd/usermod and primary vs secondary groups?

**Short answer:** A user has **one primary group** (owns files they create) and **zero or more secondary (supplementary) groups** (extra access). Manage with `useradd`/`usermod`.

```
useradd -m -s /bin/bash -G docker,sudo alice   # create alice, add to secondary groups
usermod -aG docker bob        # ADD bob to 'docker' (secondary) — -a is CRITICAL
id alice                      # show UID, primary GID, all groups
groups alice                  # list alice's groups
```

**The #1 gotcha:** `usermod -G docker bob` (without `-a`) **replaces** all of bob's secondary groups → you can lock people out. Always `usermod -aG` to *append*.

```
Primary group   → the GID in /etc/passwd; new files get this group
Secondary groups→ extra memberships (e.g., docker, sudo, wheel)
Group change takes effect on NEXT login (or `newgrp`) — not the current session.
```

**Interview tip:** Stress `-aG` (append) vs `-G` (replace) — a classic outage cause. And note group changes need a re-login to take effect (why "I added myself to docker but still get permission denied").

---

## Q70. System/service accounts, nologin, and UID ranges?

**Short answer:** Not all accounts are for humans. **Service accounts** run daemons with no interactive login. **UID ranges** separate system accounts from real users, and a **nologin/false shell** blocks interactive access.

| UID range | Purpose |
|-----------|---------|
| 0 | root |
| 1–999 | system / service accounts (nginx, mysql, sshd) |
| 1000+ | regular human users |

```
Create a locked-down service account:
  useradd --system --no-create-home --shell /usr/sbin/nologin appsvc

nologin shell → login attempts are refused (prints a message and exits)
No password set + nologin shell = the account can OWN/run things but nobody can log in as it
```

**Why it matters (least privilege):** daemons run under dedicated low-privilege service accounts (not root), so a compromised service is contained. This is exactly the `User=appuser` in a systemd unit (Q37) and the least-privilege principle from Q28/Q31.

**Interview tip:** Explain the UID split (system <1000 vs users ≥1000), `--system` + `nologin` for service accounts, and connect it to running daemons as non-root (Q37) for blast-radius reduction.

---

# 13. Cron / Scheduling

## Q21. What is Cron?

**Short answer:** Cron is the Linux **job scheduler**. It runs commands or scripts **automatically** at set times or intervals, handled by the `cron` daemon (`crond`) in the background.

**Common uses:** backups, log cleanup/rotation, monitoring scripts, report generation, scheduled maintenance.

**The crontab time format (5 fields):**
```
 ┌───────────── minute        (0-59)
 │ ┌───────────── hour        (0-23)
 │ │ ┌───────────── day of month (1-31)
 │ │ │ ┌───────────── month     (1-12)
 │ │ │ │ ┌───────────── day of week (0-6, Sun=0)
 │ │ │ │ │
 * * * * *   command_to_run
```

**Examples:**
```
0 2 * * *     /opt/backup.sh          # every day at 2:00 AM
*/5 * * * *   /opt/check_health.sh     # every 5 minutes
0 0 * * 0     /opt/weekly_cleanup.sh   # every Sunday at midnight
30 3 1 * *    /opt/monthly_report.sh   # 3:30 AM on the 1st of each month
```
Tip: use **crontab.guru** to read/write schedules quickly.

**Managing cron jobs:**
```
crontab -e        # edit YOUR cron jobs
crontab -l        # list your cron jobs
crontab -r        # remove all your cron jobs (careful!)
```
System-wide jobs also live in `/etc/crontab`, `/etc/cron.d/`, and `/etc/cron.{daily,weekly,monthly}/`.

**Gotchas (interview gold):**
- Cron runs with a **minimal environment** — no full `$PATH`. Use **absolute paths** (`/usr/bin/python3`, not `python3`).
- **Redirect output** or you won't see errors: `... >> /var/log/job.log 2>&1`.
- Cron **won't run missed jobs** if the machine was off — use `anacron` for that.

**Interview tip:** Define it as the **time-based scheduler** (daemon = `crond`), explain the 5-field format, and mention the classic gotchas: absolute paths + redirect output for logging. For "run at boot" or intervals inside apps, note that **systemd timers** are a modern alternative (Q40).

---

## Q22. Explain a Cron expression.

> The full 5-field diagram and examples are already in **Q21**. Kept here as a short standalone since it's often asked directly.

**Example:** `0 2 * * *` → runs **every day at 2:00 AM**.

**The five fields (left to right):**
```
0    2    *    *    *
│    │    │    │    └── Day of week   (0-6, Sun=0)
│    │    │    └────── Month          (1-12)
│    │    └─────────── Day of month   (1-31)
│    └──────────────── Hour           (0-23)
└───────────────────── Minute         (0-59)
```
`*` = "every / any value". So `0 2 * * *` = minute 0, hour 2, any day, any month, any weekday → 2:00 AM daily.

**Interview tip:** Read it **right-to-left** as "at minute X, hour Y, on these days/months/weekdays." See Q21 for more examples (`*/5` = every 5 units, ranges, lists) and the common gotchas.

---

## Q71. "The cron job runs fine manually but not via cron." Why? (Scenario)

> Q21 mentioned this gotcha briefly; here's the full debugging approach — a very common real incident.

**Short answer:** Cron runs with a **minimal environment** (stripped `$PATH`, no profile, different `HOME`, no interactive shell). Things that work in your logged-in shell silently fail under cron.

**The usual culprits:**
```
1. PATH        cron's PATH ≈ /usr/bin:/bin only.
               "python3" works for you, cron can't find it → use ABSOLUTE paths.
2. ENV VARS    your ~/.bashrc / ~/.profile is NOT sourced by cron.
               → set vars in the crontab or the script explicitly.
3. WORKING DIR cron starts in the user's HOME, not where you tested.
               → cd into the right dir or use absolute paths.
4. NO OUTPUT   errors go nowhere by default → you don't see the failure.
               → append:  >> /var/log/myjob.log 2>&1
5. USER        a root crontab vs your user crontab run as different users/perms.
```

**Debugging steps:**
```
- Redirect output first:   * * * * * /path/job.sh >> /tmp/job.log 2>&1
- Check cron actually ran:  grep CRON /var/log/syslog   (or journalctl -u cron)
- Reproduce cron's env:     env -i /bin/sh -c '/path/job.sh'   (empty env like cron)
- Use absolute paths + set PATH at top of the script.
```

**Interview tip:** The one-liner — *"cron doesn't source your shell profile and has a minimal PATH, so use absolute paths and redirect output to a log."* Reproducing with `env -i` is the senior move that proves it's an environment issue.

---

## Q72. User crontab vs system cron (/etc/cron.d, cron.daily) and anacron?

**Short answer:** **User crontabs** (`crontab -e`) run as that user and have no username field. **System cron** (`/etc/crontab`, `/etc/cron.d/*`) adds a **username field** and is managed as files (good for config management). **anacron** runs periodic jobs that were **missed** while the machine was off.

| Location | Runs as | Username field? | Managed how |
|----------|---------|-----------------|-------------|
| `crontab -e` (per user) | that user | No | Interactive / per user |
| `/etc/crontab`, `/etc/cron.d/*` | any (field 6) | **Yes** | Files (Ansible/packages) |
| `/etc/cron.{daily,weekly,monthly}/` | root | scripts | Drop-in scripts, run by cron/anacron |

```
System cron line (note the extra 'user' field):
  0 2 * * *  deploy  /opt/backup.sh
  └ schedule ────────┘ └user┘ └command

anacron:  laptops/VMs that aren't always on → runs a "daily" job on next boot
          if it was missed. Cron alone would just skip it.
          (systemd timers with Persistent=true do the same — see Q40.)
```

**Why it matters for DevOps:** put cron jobs in `/etc/cron.d/<app>` as files so they're version-controlled and deployed by Ansible — cleaner than editing per-user crontabs by hand. Use anacron (or systemd timers) for hosts that aren't 24/7.

**Interview tip:** Highlight the **username field** in system cron, that `/etc/cron.d` is the config-management-friendly place, and **anacron for missed jobs** on non-always-on hosts (tie to Q40 timers `Persistent=true`).

---

# 14. Shell Scripting

## Q23. Why is Shell Scripting important in DevOps?

**Short answer:** Shell scripting lets you **automate repetitive operational tasks**, so work becomes **faster, consistent, and less error-prone** than doing it by hand. It's the glue that ties Linux commands, tools, and pipelines together.

**What it automates in DevOps:**
| Task | Example |
|------|---------|
| Deployments | pull code, build, restart service |
| Log rotation/cleanup | compress & delete old logs on a schedule |
| Health checks | `curl` an endpoint, alert if it fails |
| Backup automation | `tar` + upload to S3 via cron |
| Monitoring | collect metrics, parse logs, trigger alerts |
| Server provisioning | install packages, configure services on boot |

**Why it matters (the point interviewers want):**
```
Manual, repeated by hand          →   Automated with a script
──────────────────────────           ──────────────────────────
slow, done differently each time  →   fast, identical every run
prone to human error              →   consistent & repeatable
not documented                    →   the script IS the documentation
one server at a time              →   loop over many servers
```

- **Reduces manual effort** — run once, reuse forever.
- **Minimizes human error** — no forgotten steps, same result every time.
- **Repeatable & auditable** — the script is a record of exactly what happens.
- **Scales** — apply the same action across many servers in a loop.

**Where it fits with other tools:** shell scripts handle glue logic and quick tasks; for larger config management you move to **Ansible**, and for infrastructure to **Terraform** — but shell scripting underpins all of them (bootstrap scripts, CI steps, entrypoints).

**Interview tip:** Emphasize **consistency + reduced human error + repeatability**, not just "it saves time." Give a concrete example: *"I wrote a backup script (`tar` + S3 upload) run by cron, plus a health-check script that curls the app and alerts on failure."* Mention shell is the base layer beneath Ansible/CI pipelines.

---

## Q24. Difference between #!/bin/bash and #!/bin/sh?

**Short answer:** The first line of a script (the **shebang**) tells the system which interpreter to run it with. `#!/bin/bash` forces the **Bash** shell (with all its extra features). `#!/bin/sh` uses the system's **default POSIX shell**, which may or may not be Bash depending on the OS.

| | `#!/bin/bash` | `#!/bin/sh` |
|--|---------------|-------------|
| Shell used | Always Bash | System default POSIX shell |
| Features | Full Bash (arrays, `[[ ]]`, `+=`, etc.) | Only basic POSIX features |
| Portability | Less (needs Bash installed) | More (works on any POSIX system) |
| On Ubuntu/Debian | Bash | `dash` (a lightweight shell, **not** Bash) |
| On many others | Bash | Often Bash (or a link to it) |

**The Ubuntu trap (interview favorite):**
```
On Ubuntu/Debian:  /bin/sh  ─►  dash   (NOT bash)

So a script with #!/bin/sh that uses Bash-only syntax
(like arrays or [[ ]]) will FAIL on Ubuntu with a syntax error.
```

**Example that breaks under `sh` but works under `bash`:**
```bash
#!/bin/bash
arr=(one two three)      # arrays are Bash-only
echo "${arr[1]}"         # → two    (fails with #!/bin/sh on dash)
```

**Which to use:**
- Using Bash features (arrays, `[[ ]]`, `==`) → use `#!/bin/bash`.
- Want maximum portability / minimal deps → write POSIX and use `#!/bin/sh`.

**Interview tip:** The key point — `#!/bin/sh` is **not guaranteed to be Bash** (on Ubuntu it's `dash`), so Bash-specific syntax can silently break. If your script uses Bash features, **explicitly use `#!/bin/bash`**.

---

## Q73. How do you make a bash script robust for production?

**Short answer:** Fail fast, handle errors, quote variables, and clean up on exit. The `set -euo pipefail` header + a `trap` cleanup are the backbone.

```bash
#!/usr/bin/env bash
set -euo pipefail        # the "unofficial strict mode"
IFS=$'\n\t'

trap 'echo "Error on line $LINENO"; cleanup' ERR
trap cleanup EXIT

cleanup() { rm -f "$tmpfile"; }
tmpfile="$(mktemp)"
```

| Flag | Effect |
|------|--------|
| `set -e` | Exit immediately if any command fails (non-zero) |
| `set -u` | Error on **undefined** variables (catches typos) |
| `set -o pipefail` | A pipeline fails if **any** stage fails (not just the last) |
| `set -x` | Debug: print each command as it runs |

**Other must-dos:**
- **Quote everything:** `"$var"`, `"$@"` — unquoted vars break on spaces/globbing (the #1 bash bug).
- **`trap ... EXIT`** for cleanup (temp files, locks) even on failure.
- **Check inputs** early; use `mktemp` for temp files (not predictable names).
- Prefer `$(...)` over backticks; use `[[ ]]` over `[ ]` in bash.

**Interview tip:** Lead with `set -euo pipefail` and explain each flag, then `trap EXIT` for cleanup and **quoting `"$@"`**. Bonus: `pipefail` matters because `cmd | grep x` would otherwise hide `cmd`'s failure. Mention **shellcheck** as a linter in CI.

---

## Q74. Exit codes, $?, and conditionals/test operators?

**Short answer:** Every command returns an **exit code**: `0` = success, non-zero = failure. `$?` holds the last one. Scripts use these for control flow and must `exit` with a meaningful code.

```bash
command
echo $?             # 0 = ok, non-zero = error

if command; then ...; fi          # branches on exit code (0 = true)
cmd1 && cmd2                       # run cmd2 only if cmd1 succeeded
cmd1 || echo "failed"              # run right side only if left failed

exit 1                             # signal failure to the caller/CI
```

**Common exit codes:** `0` ok · `1` general error · `2` misuse · `126` not executable · `127` command not found · `130` Ctrl+C · `137` = 128+9 (SIGKILL, e.g., OOM-killed, ties to Q15).

**test / conditionals (`[[ ]]`):**
```bash
[[ -f "$file" ]]        # file exists (regular file)
[[ -d "$dir" ]]         # directory exists
[[ -z "$s" ]]           # string is empty ;  -n = non-empty
[[ "$a" == "$b" ]]      # string equal ;  != not equal
(( n > 5 ))             # arithmetic comparison
[[ -x "$bin" ]]         # file is executable
```

**Interview tip:** Explain `$?` + `&&`/`||` short-circuit chaining, that scripts should `exit` non-zero on failure so **CI/pipelines detect it**, and decode `137 = 128 + SIGKILL(9)` (often OOM — connect to Q15). Prefer `[[ ]]` (bash) over `[ ]` for safer tests.

---

# 15. Performance & Troubleshooting

## Q25. A Linux server suddenly becomes slow. How would you investigate? (Scenario)

> Scenario question — interviewers want a **systematic method**, not random commands. It reuses tools from Q13 (memory), Q16 (disk); the new parts here are **disk I/O** and **network** checks, and the overall approach.

**Short answer:** Check the four core resources in order — **CPU, memory, disk space, disk I/O** — then network, logs, and recent changes. Eliminate bottlenecks one by one instead of guessing.

**The USE method (mental model):** for each resource check **U**tilization, **S**aturation, **E**rrors.

```
1. QUICK PULSE   → uptime / top       load average vs CPU count?
2. CPU           → top, htop, vmstat  a process pegging CPU? high %us vs %wa?
3. MEMORY        → free -h            low available? swapping? (see Q13/Q14)
4. DISK SPACE    → df -h              a full mount? (see Q16)
5. DISK I/O      → iostat -x 1, iotop  high %util / await = disk bottleneck
6. NETWORK       → ss, ping, traffic   latency, dropped packets, saturated NIC
7. LOGS          → journalctl, /var/log, dmesg   errors, OOM, disk errors
8. RECENT CHANGE → any deploy/config change just before it started?
```

**Reading the signals:**
| Symptom | Likely cause | Where to look |
|---------|--------------|---------------|
| High load, high `%us` (user CPU) | App burning CPU | `top` → find the process |
| High load, high `%wa` (I/O wait) | Disk is the bottleneck | `iostat -x`, `iotop` |
| Low free mem + swapping | Memory pressure | `free -h`, `vmstat` (si/so), `dmesg` (OOM) |
| Fine CPU/mem but slow responses | Network or downstream dep | `ss`, `ping`, app logs |

**Key commands:**
```
uptime                # load average (1/5/15 min) — compare to CPU count
vmstat 1              # CPU, run queue, swap in/out, io — live
iostat -x 1           # per-disk %util and await (I/O latency)
iotop                 # which process is doing the I/O (if installed)
ss -s                 # socket summary; ss -tulpn for listeners
dmesg -T | tail       # kernel errors: OOM, disk/hardware issues
```

**The golden rule:** load average > number of CPUs = system is overloaded. Then figure out *what* is overloading it (CPU-bound vs I/O-bound vs memory-bound).

**Interview tip:** Lead with **"I isolate the bottleneck systematically — CPU, memory, disk, I/O, network — and I don't jump to conclusions."** Bonus: mention the **`%wa` (I/O wait)** clue in `top` to distinguish a CPU problem from a disk problem, and always **correlate with recent deployments/changes**.

---

## Q29. (Scenario) CPU utilization is consistently above 95%. How would you troubleshoot?

> Related to Q25 (general slow-server method), but here the symptom is known: **CPU-bound**. The new angle is deciding **legitimate load vs abnormal**, then optimize/scale.

**Short answer:** Find *which* process is burning CPU, decide if the load is **expected (real traffic) or abnormal (bug/loop/runaway)**, check recent changes and logs, then either **optimize the app** or **scale resources** — matching the fix to the cause.

**Steps:**
```
1. WHO?        top / htop (press P = sort by CPU),  ps -eo pid,%cpu,cmd --sort=-%cpu
               pidstat 1   → per-process CPU over time
2. WHAT KIND?  is it %us (user code) or %sy (kernel/syscalls)?
               high %us = app logic;  high %sy = syscalls/context-switching
3. LEGIT?      does CPU match real traffic/workload?  or a single runaway process?
4. LOGS        errors, retries, tight loops, GC storms, stuck threads
5. CHANGES     any recent deploy/config change right before it started?
6. LIMITS      app config / thread pools / cgroup or container CPU limits
7. PROFILE     if unclear: perf top,  strace -p <PID>,  thread dump (jstack for Java)
8. FIX         legit load → scale (add CPU / more replicas / autoscale)
               abnormal   → fix the bug / loop / bad query, then redeploy
```

**Legitimate vs abnormal (the key decision):**
| Signal | Likely legitimate | Likely abnormal |
|--------|-------------------|-----------------|
| CPU tracks traffic up/down | yes | |
| One process pegged at 100% with no matching load | | yes (loop/bug) |
| Started right after a deploy | | yes (regression) |
| Grows over time without traffic change | | yes (leak/GC/retry storm) |
| All replicas evenly busy under real load | yes (need to scale) | |

**Interview tip:** The mark of seniority is **not just "kill the process."** Say: *"First I identify the process, then decide if the load is legitimate — because the fix differs. Real load → scale out/up. Abnormal (loop, bad deploy, runaway query) → fix the root cause; scaling would just waste money hiding a bug."* Mention `%us` vs `%sy` and profiling (`perf`, `strace`, thread dumps).

---

## Q75. Load average and CPU states (%us/%sy/%wa/%st) — how do you read them?

**Short answer:** **Load average** = average number of processes running or *waiting* (for CPU or uninterruptible I/O) over 1/5/15 min. Compare it to the **CPU core count**. **CPU states** in `top` tell you *where* the time goes.

```
uptime → load average: 8.20, 5.10, 2.30    (1min, 5min, 15min)

Rule:  load ≈ number of cores  → fully used, healthy
       load  >  cores          → overloaded (things are waiting)
       4 cores, load 8         → 2x oversubscribed
       trend: 1min >> 15min     → problem is ramping UP right now
```

**CPU states (the `%Cpu(s)` line in top):**
| State | Meaning | High value points to |
|-------|---------|-----------------------|
| `%us` | User CPU (app code) | App is CPU-bound (Q29) |
| `%sy` | System/kernel (syscalls, context switches) | Syscall-heavy / too many threads |
| `%wa` | I/O wait (CPU idle, waiting on disk) | **Disk is the bottleneck** (see Q76) |
| `%id` | Idle | Spare capacity |
| `%st` | **Steal** — hypervisor gave your vCPU to someone else | **Noisy neighbor** on a shared VM/cloud host |

**The cloud gotcha — `%st` (steal):** high steal on a VM means the host is oversubscribed and you're not getting the CPU you pay for. The fix isn't in your app — it's a bigger/dedicated instance. This is a senior differentiator most people miss.

**Interview tip:** Say "load only means something relative to core count," and use the CPU states to classify the problem: `%us`=app, `%wa`=disk, `%st`=noisy-neighbor VM. Load caused by `%wa` is **not** a CPU shortage — it's I/O.

---

## Q76. Diagnosing disk I/O bottlenecks, and the performance tool taxonomy?

**Short answer:** When `%wa` is high, use `iostat -x` to find the busy disk. Watch **%util** (how busy) and **await** (latency per I/O). Then pick the right tool for the resource.

```
iostat -x 1
Device   r/s   w/s   await   %util
nvme0n1  120   340   28.5    98.7   ← %util ~100% + high await = disk saturated

%util  → % of time the disk was busy (near 100% = maxed)
await  → avg ms per I/O (rising = latency problem)
```
Find *who* is doing the I/O: `iotop` or `pidstat -d 1`.

**Performance tool taxonomy (which tool for which resource):**
| Resource | Tools |
|----------|-------|
| CPU | `top`, `htop`, `pidstat -u`, `mpstat`, `perf top` |
| Memory | `free`, `vmstat`, `sar -r`, `smem` (see Q13/Q66) |
| Disk I/O | `iostat -x`, `iotop`, `pidstat -d` |
| Network | `ss`, `iftop`, `nload`, `sar -n DEV` |
| Everything / history | `sar` (historical), `dstat`, `glances` |
| Syscall-level | `strace`, `perf`, `bpftrace` (eBPF) |

**Interview tip:** For disk: **%util near 100% + rising await = I/O-bound**, then `iotop`/`pidstat -d` to find the culprit process. Name `sar` for **historical** data (crucial when the incident already passed) and `perf`/`bpftrace` (eBPF) for deep, low-overhead tracing — that signals real 10+ YOE depth.

---

# 16. Security & Hardening

## Q28. How do you secure Linux servers in a production environment?

> Consolidated hardening answer. Some pieces are covered in detail elsewhere — SSH key auth (Q20), root vs sudo (Q31), permissions (Q7–Q9), firewalls (Q78), SSH hardening (Q79).

**Short answer:** Reduce the attack surface and enforce least privilege at every layer — **access, patching, firewall, monitoring, and minimal footprint** — and bake it into **provisioning**, not as an afterthought.

**Hardening checklist (grouped):**

```
ACCESS
  • Disable root SSH login       PermitRootLogin no        (see Q20/Q79)
  • SSH keys only, no passwords  PasswordAuthentication no (see Q20)
  • Least privilege via sudo     no shared root; per-user sudo (Q31)
FIREWALL
  • Host firewall (ufw/firewalld/iptables) — default deny, allow only needed ports (Q78)
PATCHING
  • Regular OS security updates  (unattended-upgrades / patch cycle) (Q48)
FOOTPRINT
  • Remove unused services/packages   (systemctl disable, uninstall)
MONITORING & AUDIT
  • Watch auth & system logs     /var/log/auth.log, journalctl
  • Centralized logging + audit  (ship logs off-box; auditd) (Q54)
GOVERNANCE
  • Review users/permissions regularly
  • Automate compliance checks   (CIS benchmark, Lynis, OpenSCAP)
```

**Why each matters:**
| Control | Protects against |
|---------|------------------|
| No root SSH + key-only auth | Brute-force logins, credential theft |
| Least privilege (sudo) | Blast radius if one account is compromised |
| Host firewall (default deny) | Exposed/forgotten services |
| Patching | Known CVEs / exploits |
| Remove unused services | Fewer things to attack (smaller surface) |
| Log monitoring + auditing | Detecting intrusions, forensics |
| Central logging | Attacker can't wipe local logs to hide |
| Regular user/permission review | Stale/over-privileged accounts |
| Automated compliance (CIS) | Drift; provable, repeatable security |

**Key principle — shift security left into provisioning:**
```
Bad:   build server → run it → "harden it later" (often never happens)
Good:  hardening baked into the image / Ansible playbook / Terraform
       → every server is born secure and identical
```

**Interview tip:** Don't just list controls — group them (**access, patching, firewall, monitoring, footprint**) and stress two ideas: **least privilege** everywhere, and **security built into provisioning** (Ansible/golden AMI/CIS-hardened image) so it's consistent and auditable, not manual.

---

## Q77. SELinux vs AppArmor — what are they and how do you work with them?

**Short answer:** Both are **MAC (Mandatory Access Control)** — an extra layer *beyond* normal file permissions that confines what a process can do, even as root. **SELinux** (RHEL) uses **labels** on everything; **AppArmor** (Ubuntu/SUSE) uses **per-application path-based profiles**.

| | SELinux | AppArmor |
|--|---------|----------|
| Default on | RHEL/CentOS/Fedora | Ubuntu/Debian/SUSE |
| Model | Labels/contexts on files+processes | Path-based per-app profiles |
| Granularity | Very fine (complex) | Simpler to read/write |
| Modes | enforcing / permissive / disabled | enforce / complain |

```
SELinux:
  getenforce                 # Enforcing / Permissive / Disabled
  setenforce 0               # switch to permissive (temporary, for debugging)
  ausearch -m avc / audit.log → find what got DENIED (AVC denials)
  restorecon -Rv /path       # fix mislabeled files (common web-root gotcha)

AppArmor:
  aa-status                  # loaded profiles + mode
  aa-complain /path/to/bin   # profile logs but doesn't block (debug)
```

**Senior gotcha:** the classic *"works with SELinux off, breaks with it on"* — don't just disable it (`setenforce 0`). Check **AVC denials** (`ausearch`/`audit.log`), then fix the **label** (`restorecon`, `semanage fcontext`) or add a policy. Disabling MAC in prod is a real finding in audits.

**Interview tip:** MAC = confinement *even for root*. Say you set **permissive to diagnose**, read AVC denials, and fix labels rather than disabling. Naming `restorecon`/`ausearch` (SELinux) or `aa-complain` (AppArmor) shows hands-on depth.

---

## Q78. Linux firewalls: iptables vs nftables vs ufw/firewalld?

**Short answer:** All manage the kernel's **netfilter** packet filtering. **iptables** is the legacy tool; **nftables** is its modern replacement; **ufw** (Ubuntu) and **firewalld** (RHEL) are friendly front-ends on top.

```
        ufw / firewalld   ← easy front-ends (what you use day-to-day)
              │
        nftables (nft)    ← modern engine (replaces iptables)
              │
        netfilter (kernel) ← the actual packet filter
   (iptables = legacy front-end to the same kernel layer)
```

| Tool | Layer | Use |
|------|-------|-----|
| `iptables` | Legacy rules | Older systems, still common |
| `nftables` (`nft`) | Modern rules engine | Default on newer distros |
| `ufw` | Front-end (Debian/Ubuntu) | Simple: `ufw allow 22/tcp` |
| `firewalld` | Front-end (RHEL), zones | `firewall-cmd --add-service=ssh` |

```
ufw enable; ufw default deny incoming; ufw allow 22/tcp; ufw allow 443/tcp
firewall-cmd --permanent --add-port=443/tcp; firewall-cmd --reload
```

**Cloud reality:** on cloud VMs, **Security Groups / NSGs** usually do the perimeter filtering, and host firewalls add defense-in-depth. Default policy should be **deny inbound, allow only needed ports** (ties to Q28, Q45).

**Interview tip:** Explain the stack (front-end → nftables → netfilter), that **nftables is replacing iptables**, and use `ufw`/`firewalld` for simplicity. Mention **default-deny inbound** + cloud SGs as the outer layer.

---

## Q79. Deep SSH hardening and fail2ban?

> Q20 covered key vs password auth; this is the fuller hardening + brute-force defense.

**Short answer:** Lock down `sshd_config`, use keys only, restrict who can log in, and add **fail2ban** to auto-ban brute-force sources.

**Key `/etc/ssh/sshd_config` settings:**
```
PermitRootLogin no                 # no direct root (see Q31)
PasswordAuthentication no          # keys only (see Q20)
PubkeyAuthentication yes
AllowUsers deploy admin            # whitelist who can log in
Port 2222                          # optional: reduce noise (not real security)
MaxAuthTries 3
ClientAliveInterval 300            # drop idle sessions
X11Forwarding no
# then: sshd -t (validate)  &&  systemctl reload sshd
```

**fail2ban — auto-ban brute-force:**
```
watches /var/log/auth.log → N failed logins in a window → temp-ban the IP via firewall

/etc/fail2ban/jail.local:
  [sshd]
  enabled = true
  maxretry = 5
  bantime = 1h
  findtime = 10m

fail2ban-client status sshd      # see banned IPs
```

**Senior additions:** use a **bastion/jump host** so servers aren't directly exposed; **short-lived certificates** (SSH CA) instead of long-lived keys at scale; centralize auth logs (Q54) so bans/attempts are visible fleet-wide.

**Interview tip:** Give the top `sshd_config` lines (no root, no passwords, `AllowUsers`, `MaxAuthTries`), add **fail2ban** for brute-force, and level up with **bastion host + SSH CA (short-lived certs)** for scale — that's the 10+ YOE answer. Always `sshd -t` before reload so you don't lock yourself out.

---

# 17. Production Scenarios

> Additional scenario-style questions also live with their topics: Q18 (disk full), Q29 (high CPU), Q45 (service unreachable), Q55 (log correlation), Q71 (cron not running).

## Q26. (Scenario) The application is down, but the server is reachable. What would you check first?

> The **host is fine** (you can SSH in, it responds to ping) but the **app isn't serving**. So focus on the *application layer*, not the machine. Reuses process checks (Q6) and resource checks (Q25).

**Short answer:** Work up the stack: is the **process running** → **logs** → **resources** → **dependencies** → **ports/firewall** → **recent changes**. Restart only *after* you know the likely cause.

**Decision tree:**
```
App down, server reachable
        │
        ▼
1. Is the process running?     systemctl status app  /  pgrep -a app
   ├─ No  → why did it die? →  journalctl -u app,  dmesg (OOM?),  exit code
   └─ Yes → it's running but not serving → keep going
        │
        ▼
2. App & system logs           journalctl -u app -n 100,  tail -f /var/log/app/*
        │  (errors? stack traces? "address already in use"? DB connect fail?)
        ▼
3. Resources OK?               df -h (disk full?),  free -h (OOM?),  top (CPU?)   [see Q25]
        │
        ▼
4. Dependencies up?            DB / cache / downstream API reachable?
        │                       curl the dependency,  nc -zv db-host 5432
        ▼
5. Ports & firewall            ss -tulpn | grep <port>  (is it listening?)
        │                       firewall/security-group allowing the port?
        ▼
6. Recent change?              new deploy / config change / cert expiry just before?
        │
        ▼
7. Restart ONLY after diagnosing → systemctl restart app  (and watch logs)
```

**What each step catches:**
| Check | Command | Catches |
|-------|---------|---------|
| Process alive | `systemctl status`, `pgrep -a` | crashed/stopped service |
| Logs | `journalctl -u app`, `/var/log` | app errors, DB failures, bad config |
| Resources | `df -h`, `free -h`, `top` | disk full, OOM kill, CPU pegged |
| Dependencies | `curl`, `nc -zv host port` | DB/cache/API down |
| Listening port | `ss -tulpn` | app not bound / wrong port |
| Firewall/SG | rules review | traffic blocked before reaching app |
| Recent change | deploy/config history | bad release, expired cert |

**Key distinction (say this):** *"Because the server is reachable, I don't start with the host — I start at the application: is the process even running, and if so, why isn't it serving? Logs usually tell the story."*

**Interview tip:** Two things score points: (1) **restart last, not first** — restarting destroys evidence and may just delay a recurring crash; (2) always check **dependencies** (DB/cache/API) and **listening ports** — an app that's "running" but can't bind a port or reach its DB looks down to users.

---

## Q27. (Scenario) "No space left on device" but deleting large logs didn't free space. Why?

> The **deleted-but-open file** case (also referenced in Q16). Here the focus is the *why* and the *fix procedure*.

**Short answer:** A **running process still has the deleted file open**. On Linux, deleting a file only removes its **directory entry (name)** — the actual data blocks are freed only when the **last file descriptor** referring to them is closed. So the process keeps writing/holding the space even though the file "doesn't exist."

```
Normal delete:   name ──► inode ──► data blocks        rm removes name,
                                                        refcount → 0, blocks freed OK

Open-file delete: name ──► inode ──► data blocks
                            ▲
                     process still has FD open  (refcount > 0)
                 rm removes the name, but blocks are NOT freed
                 until the process closes it / exits  ✗  (df still shows full)
```

This is exactly why `df` (says full) and `du` (says less) **disagree** — see Q16.

**How to find and fix it:**
```
1. Find deleted-but-open files:
   lsof | grep deleted
   lsof +L1                     # files with link count 0 (deleted, still open)
   → shows PID, process name, and size of the held file

2. Release the space (choose one):
   - Restart the offending process/service:   systemctl restart <service>
   - Or truncate the FD in place (no restart): : > /proc/<PID>/fd/<FD>
     (frees space immediately by emptying the open file)
```

**Typical culprit:** an app logging to a file that got `rm`'d (or rotated incorrectly) but the app was never signaled to reopen its log — so it keeps writing to the now-nameless file.

**Interview tip:** Nail the concept — *"deleting a file removes its name, not its data; the space frees only when the last open file descriptor is closed."* Fix by restarting the process (or `truncate`/`: > /proc/PID/fd/N`), and **prevent** it with proper `logrotate` using `copytruncate` or a post-rotate reload signal (Q53).

---

*End of document — 79 questions across 17 topic sections.*
