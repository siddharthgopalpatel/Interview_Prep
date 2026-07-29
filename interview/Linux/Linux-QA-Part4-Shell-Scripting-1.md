## CATEGORY 9: SHELL SCRIPTING & TEXT PROCESSING (8 Questions)

---

### Q7: Explain sed in-place editing, regex substitution, and address ranges. Give real examples from your work.
**Project Reference:** Project 1 (DevSecOps CI/CD Pipeline — sed in Jenkinsfile)
**Expected Depth:** Practical usage in CI/CD, in-place vs stdout, BRE vs ERE, backup files, multi-command

**Answer:**

**How we use sed in Project 1 (Jenkinsfile):**
```groovy
// Update image tag in GitOps repo for ArgoCD
sh "sed -i 's|newTag:.*|newTag: ${IMAGE_TAG}|' overlays/prod/kustomization.yml"
```

This is the core of our GitOps promotion — Jenkins updates the Kustomize overlay, commits, and ArgoCD syncs.

**In-place editing (`-i`):**
```bash
# In-place (modifies file directly) — GNU sed
sed -i 's/old/new/g' file.txt

# In-place with backup (creates file.txt.bak)
sed -i.bak 's/old/new/g' file.txt

# macOS BSD sed requires an argument to -i (even empty string)
sed -i '' 's/old/new/g' file.txt

# CRITICAL: In CI/CD, always use GNU sed or account for this difference
```

**Regex substitution patterns:**
```bash
# Basic: s/pattern/replacement/flags
sed 's/http:/https:/g' urls.txt          # g = all occurrences on line
sed 's/^#ServerName/ServerName/' httpd.conf  # Uncomment a line
sed 's/\(v[0-9]*\)\.\([0-9]*\)/\1.\2.0/' versions.txt  # Capture groups (BRE)
sed -E 's/(v[0-9]+)\.([0-9]+)/\1.\2.0/' versions.txt   # Extended regex (ERE, cleaner)

# Different delimiters (useful for paths)
sed 's|/usr/local/bin|/opt/bin|g' script.sh
sed 's#https://old.com#https://new.com#g' config.yml
```

**Address ranges (specify WHICH lines to act on):**
```bash
# Line numbers
sed '5s/foo/bar/' file           # Only line 5
sed '10,20s/foo/bar/g' file     # Lines 10-20
sed '1,/^END/d' file            # Delete from line 1 to first line matching ^END

# Pattern addresses
sed '/^#/d' config              # Delete all comment lines
sed '/START/,/END/s/old/new/g' file  # Substitute only between START and END blocks
sed '/server_name/a\    proxy_pass http://backend;' nginx.conf  # Append after match

# Negation
sed '/^$/!s/$/;/' file          # Add semicolon to all NON-empty lines
```

**Multi-command and advanced:**
```bash
# Multiple commands
sed -e 's/foo/bar/g' -e 's/baz/qux/g' file
sed '/pattern/{s/old/new/; s/this/that/}' file

# Delete and insert
sed '/deprecated_config/d' app.conf              # Delete matching lines
sed '/\[section\]/a new_key = value' app.conf    # Append after match
sed '/\[section\]/i # Added by automation' app.conf  # Insert before match

# Print specific lines (like head/tail but flexible)
sed -n '5,10p' file             # Print lines 5-10 only
sed -n '/ERROR/p' log.txt       # Like grep but with sed
```

**In our CI/CD context, why sed over other tools:**
- It's POSIX, available everywhere (Alpine containers, minimal images)
- Single-line replacements in YAML/config files are sed's sweet spot
- For complex YAML manipulation, we'd use `yq` instead
- Always test with `sed 's/.../.../p'` (print) before adding `-i`

---

### Q8: Explain awk — field extraction, pattern matching, and multi-line processing. When do you use awk over sed/grep?
**Project Reference:** Project 9 (Enterprise OS Patching — parsing command output in validation scripts)
**Expected Depth:** Column extraction, BEGIN/END blocks, field separators, real admin use cases

**Answer:**

**When to use awk vs sed vs grep:**
- `grep`: Find lines matching a pattern (filter)
- `sed`: Transform text (substitute, delete, insert)
- `awk`: Process structured/columnar data (extract, compute, format)

**Basic field extraction (bread and butter for sysadmins):**
```bash
# Default separator is whitespace
df -h | awk '{print $5, $6}'           # Usage% and mountpoint
ps aux | awk '{print $2, $11}'          # PID and command
free -m | awk '/^Mem:/{print $3}'       # Used memory in MB

# Custom field separator
awk -F: '{print $1, $7}' /etc/passwd    # Username and shell
awk -F',' '{print $2}' data.csv         # Second column of CSV
```

**Pattern matching (condition {action}):**
```bash
# Print lines where condition is true
awk '$3 > 90 {print $1, "is at", $3"%"}' disk_usage.txt
awk '/ERROR/ {print}' application.log
awk '$NF == "FAILED" {print $0}' validation_report.txt

# In Project 9 — parse dnf output for pending updates:
dnf check-update | awk 'NF==3 {print $1}'  # Package names only (3-field lines)
```

**BEGIN/END blocks:**
```bash
# BEGIN runs before processing, END runs after
awk 'BEGIN {print "Server","CPU%","Mem%"} 
     $3>80 {print $1,$3,$4} 
     END {print "---Report Complete---"}' metrics.txt

# Count occurrences
awk '/ERROR/{count++} END{print "Total errors:", count}' app.log

# Sum a column (e.g., disk usage)
df | awk '/^\/dev/{sum+=$3} END{print "Total used:", sum/1024/1024, "GB"}'
```

**Multi-line processing:**
```bash
# Process records separated by blank lines (RS = Record Separator)
awk 'BEGIN{RS=""; FS="\n"} {print $1, $3}' multi_record.txt

# Join continuation lines
awk '/\\$/{sub(/\\$/,""); hold=hold $0; next} {print hold $0; hold=""}' config.txt
```

**Real examples from Project 9 validation:**
```bash
# Parse systemctl output for failed services
systemctl --failed --no-legend | awk '{print $2}'

# Extract memory usage percentage
free | awk '/^Mem:/{printf "%.1f%%\n", $3/$2*100}'

# Parse ss/netstat for listening ports
ss -tlnp | awk 'NR>1 {split($4,a,":"); print a[length(a)], $NF}'

# Check disk usage and flag warnings
df -h | awk 'NR>1 && +$5 > 90 {print "CRITICAL:", $6, "at", $5}'
```

**Associative arrays (awk's killer feature):**
```bash
# Count log entries per hour
awk -F'[: ]' '{hours[$4]++} END{for(h in hours) print h":00", hours[h]}' access.log

# Sum bytes per IP from access log
awk '{bytes[$1]+=$10} END{for(ip in bytes) print ip, bytes[ip]}' access.log | sort -k2 -rn
```

---

### Q9: Explain grep — regex types, recursive search, context lines. How do you use it in troubleshooting?
**Project Reference:** Project 9 (OS Patching — log analysis in validation)
**Expected Depth:** BRE vs ERE vs PCRE, practical troubleshooting patterns, performance considerations

**Answer:**

**Regex types in grep:**
```bash
# BRE (Basic Regular Expression) — default
grep 'error\|warning' log.txt          # Escape alternation
grep 'v[0-9]\{1,3\}' versions.txt     # Escape quantifiers

# ERE (Extended) — -E or egrep — cleaner syntax
grep -E 'error|warning' log.txt        # No escape needed
grep -E 'v[0-9]{1,3}' versions.txt

# PCRE (Perl-compatible) — -P — most powerful
grep -P '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}' log.txt  # IP addresses
grep -P '(?<=password=)\S+' config.txt  # Lookbehind (extract value after password=)
```

**Context lines (critical for log troubleshooting):**
```bash
grep -B 5 'OOMKilled' /var/log/messages       # 5 lines Before
grep -A 10 'Exception' app.log                 # 10 lines After
grep -C 3 'connection refused' syslog          # 3 lines Context (both)

# In Project 9: Check for errors after patching
journalctl --since "1 hour ago" | grep -C 2 -iE 'error|failed|fatal'
```

**Recursive search (searching codebases/configs):**
```bash
grep -r 'password' /etc/                  # Recursive through directory
grep -rn 'TODO' --include='*.py' .        # With line numbers, filter file type
grep -rl 'deprecated_function' src/       # Only filenames (for scripting)
grep -rI 'pattern' .                      # Skip binary files

# Exclude directories (important in repos)
grep -r --exclude-dir={.git,node_modules,vendor} 'API_KEY' .
```

**Practical troubleshooting patterns:**
```bash
# Find all unique error types in last hour
journalctl --since "1 hour ago" -p err --no-pager | grep -oP '(?<=: ).*' | sort | uniq -c | sort -rn

# Check if a service is listening
ss -tlnp | grep ':8080'

# Find which config file contains a setting
grep -rl 'max_connections' /etc/

# Extract IPs from logs
grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}' access.log | sort | uniq -c | sort -rn | head

# Check for failed SSH attempts
grep -c 'Failed password' /var/log/secure    # Count
grep 'Failed password' /var/log/secure | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn  # By IP

# In Project 9 — check for critical errors post-patch:
grep -iE 'segfault|oom|panic|fatal' /var/log/messages
```

**Performance considerations:**
```bash
# Fixed string (faster, no regex engine)
grep -F 'exact string' hugefile.log       # fgrep equivalent

# Stop after first match
grep -m 1 'pattern' file

# Count only (don't output matches)
grep -c 'ERROR' log.txt

# For huge files, combine with head/tail first
tail -100000 app.log | grep 'timeout'
```

**Inversion and advanced:**
```bash
# Lines NOT matching (exclusion)
grep -v '^#' config.conf | grep -v '^$'   # Remove comments and blank lines

# Multiple patterns from file
grep -f patterns.txt logfile.txt

# Quiet mode (for scripting — exit code only)
if grep -q 'healthy' /tmp/healthcheck; then echo "OK"; fi
```

---

### Q10: Explain the find command — searching by time, size, type, and the difference between -exec and xargs.
**Project Reference:** Project 9 (OS Patching — cleanup old logs, find large files)
**Expected Depth:** Time predicates, size, combining conditions, exec vs xargs performance, real cleanup scripts

**Answer:**

**Basic syntax:** `find [path] [conditions] [actions]`

**By time (critical for operations):**
```bash
# Modified time (-mtime: days, -mmin: minutes)
find /var/log -mtime +30 -name "*.log"      # Files modified >30 days ago
find /tmp -mmin -60                           # Files modified in last 60 minutes
find / -mtime 0                               # Files modified today (last 24h)

# Access time and change time
find /data -atime +90 -type f                 # Not accessed in 90 days (cleanup candidate)
find /etc -cmin -5                            # Config changed in last 5 minutes (troubleshooting)

# Newer than reference file
find /app -newer /app/last_deploy_marker      # Files changed since last deploy
```

**By size:**
```bash
find / -size +100M -type f                    # Files larger than 100MB
find /var/log -size +1G -exec ls -lh {} \;   # Log files over 1GB (with details)
find / -size 0 -type f                        # Empty files
find /data -size +500M -size -2G              # Between 500MB and 2GB
```

**By type:**
```bash
find / -type f                # Regular files
find / -type d                # Directories
find / -type l                # Symbolic links
find / -type s                # Sockets
find /dev -type b             # Block devices
```

**Combining conditions:**
```bash
# AND (default)
find /var/log -name "*.log" -mtime +7 -size +10M

# OR
find / \( -name "*.tmp" -o -name "*.bak" \) -mtime +30

# NOT
find /etc -type f ! -name "*.conf"

# Complex: find large, old log files but exclude rotated
find /var/log -name "*.log" -mtime +30 -size +50M ! -name "*rotated*"
```

**-exec vs xargs:**

```bash
# -exec: runs command ONCE PER FILE (simple, slower for many files)
find /tmp -name "*.tmp" -exec rm {} \;
# Spawns: rm file1.tmp; rm file2.tmp; rm file3.tmp (3 processes)

# -exec with +: batches files (like xargs, but built-in)
find /tmp -name "*.tmp" -exec rm {} +
# Spawns: rm file1.tmp file2.tmp file3.tmp (1 process, faster!)

# xargs: reads stdin, batches into command arguments
find /tmp -name "*.tmp" | xargs rm
# Same as -exec {} + but separate command

# xargs with -0 (handle filenames with spaces/special chars)
find /tmp -name "*.tmp" -print0 | xargs -0 rm

# xargs with parallelism
find . -name "*.gz" -print0 | xargs -0 -P 4 gunzip    # 4 parallel processes
```

**When to use which:**
| Use Case | Best Tool | Why |
|----------|-----------|-----|
| Simple delete/chmod | `-exec {} +` | Built-in, handles special chars |
| Need parallelism | `xargs -P` | Built-in parallel execution |
| Complex per-file logic | `-exec sh -c '...' _ {} \;` | Need shell features |
| Files with weird names | `-print0 \| xargs -0` | Null-delimited = safe |
| Interactive confirmation | `xargs -p` | Prompts before executing |

**Real-world examples from Project 9:**
```bash
# Find old kernel packages filling /boot
find /boot -name "vmlinuz-*" -mtime +90 | sort

# Disk space emergency — find largest files
find / -xdev -type f -size +100M -exec ls -lhS {} + 2>/dev/null | head -20

# Clean old log files (safe: only .log.gz older than 60 days)
find /var/log -name "*.log.gz" -mtime +60 -delete

# Find files changed during patching window (forensics)
find / -xdev -mmin -30 -type f 2>/dev/null | grep -v '/proc\|/sys'

# Find world-writable files (security audit)
find / -xdev -type f -perm -0002 -exec ls -l {} +
```

---
