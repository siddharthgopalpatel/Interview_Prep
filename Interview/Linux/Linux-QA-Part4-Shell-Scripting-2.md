## CATEGORY 9: SHELL SCRIPTING & TEXT PROCESSING — Continued (Questions 11-14)

---

### Q11: Explain Bash exit codes and error handling — set -e, trap, and pipefail. How do you write robust scripts?
**Project Reference:** Project 9 (Enterprise OS Patching — bash scripts within Ansible roles) / Project 1 (CI/CD health check scripts)
**Expected Depth:** Exit code conventions, defensive scripting patterns, trap for cleanup, real pipeline safety

**Answer:**

**Exit codes convention:**
```bash
# 0 = success, non-zero = failure
# Standard meanings:
# 0   — Success
# 1   — General error
# 2   — Misuse of shell command
# 126 — Command found but not executable
# 127 — Command not found
# 128+N — Killed by signal N (e.g., 137 = 128+9 = SIGKILL, 143 = SIGTERM)
# 130 — Ctrl+C (SIGINT)

# Check last command's exit code
echo $?

# In scripting — use exit codes meaningfully
validate_health() {
    curl -sf http://localhost:8080/health || return 1
}
validate_health
if [ $? -ne 0 ]; then
    echo "Health check FAILED"
    exit 1
fi
```

**set -e (errexit) — exit on any error:**
```bash
#!/bin/bash
set -e    # Script exits immediately if ANY command returns non-zero

apt-get update
apt-get install nginx    # If this fails, script STOPS here
systemctl start nginx    # This never runs if above failed

# GOTCHA: set -e ignores failures in:
# - Commands in if/while conditions
# - Commands before && or ||
# - Commands in pipelines (unless pipefail is set)

# Example of the trap:
set -e
false | true    # This does NOT exit! (pipeline exit = last command = true = 0)
```

**set -o pipefail — pipeline fails if ANY command in pipe fails:**
```bash
#!/bin/bash
set -eo pipefail

# Without pipefail: exit code = last command only
curl http://api.example.com | jq '.status'
# If curl fails but jq succeeds (on empty input), pipeline "succeeds" — WRONG

# With pipefail: exit code = rightmost failed command
curl http://api.example.com | jq '.status'
# Now if curl fails (exit 7), entire pipeline fails — CORRECT
```

**The defensive scripting header (my standard for all production scripts):**
```bash
#!/bin/bash
set -euo pipefail
IFS=$'\n\t'

# set -e:         Exit on error
# set -u:         Treat unset variables as error (catches typos!)
# set -o pipefail: Pipeline fails if any command fails
# IFS:            Safer word splitting (only newline and tab, not space)
```

**trap — cleanup and error handling:**
```bash
#!/bin/bash
set -euo pipefail

# Cleanup function
cleanup() {
    local exit_code=$?
    echo "Cleaning up temporary files..."
    rm -f "$TMPFILE"
    # Re-register with ALB if we deregistered
    if [ "$DRAINED" = "true" ]; then
        aws elbv2 register-targets --target-group-arn "$TG_ARN" --targets Id="$INSTANCE_ID"
    fi
    exit $exit_code
}

# Register trap — runs on EXIT (normal or error), INT (Ctrl+C), TERM (kill)
trap cleanup EXIT INT TERM

TMPFILE=$(mktemp)
DRAINED="false"

# Now if ANYTHING fails, cleanup() runs automatically
aws elbv2 deregister-targets --target-group-arn "$TG_ARN" --targets Id="$INSTANCE_ID"
DRAINED="true"
# ... do patching work ...
```

**Error handling patterns in Project 9 scripts:**
```bash
# Pattern 1: Retry with backoff
retry() {
    local max_attempts=$1; shift
    local delay=$1; shift
    local attempt=1
    
    while [ $attempt -le $max_attempts ]; do
        if "$@"; then
            return 0
        fi
        echo "Attempt $attempt/$max_attempts failed. Retrying in ${delay}s..."
        sleep $delay
        attempt=$((attempt + 1))
        delay=$((delay * 2))  # Exponential backoff
    done
    return 1
}

retry 3 5 curl -sf http://localhost:8080/health

# Pattern 2: Collect errors but don't stop
errors=0
check_service httpd || ((errors++))
check_service redis || ((errors++))
check_disk_space    || ((errors++))
[ $errors -eq 0 ] || { echo "$errors checks failed"; exit 1; }
```

---

### Q12: How do pipes work at the OS level? Explain the pipe syscall and buffering.
**Project Reference:** Project 9 (OS Patching — complex command pipelines in validation)
**Expected Depth:** Kernel-level understanding, pipe2 syscall, buffer sizes, implications for scripting

**Answer:**

**At the kernel level:**
```c
// The pipe() syscall creates a unidirectional data channel
int pipefd[2];
pipe(pipefd);
// pipefd[0] = read end (file descriptor)
// pipefd[1] = write end (file descriptor)
```

**What the shell does with `cmd1 | cmd2`:**
1. Shell calls `pipe()` — gets two file descriptors (read_fd, write_fd)
2. Shell `fork()`s twice — creates two child processes
3. Child 1 (cmd1): closes read_fd, redirects stdout (fd 1) to write_fd → `dup2(write_fd, STDOUT_FILENO)`
4. Child 2 (cmd2): closes write_fd, redirects stdin (fd 0) to read_fd → `dup2(read_fd, STDIN_FILENO)`
5. Both children `exec()` their respective commands
6. Data flows: cmd1 writes to pipe → kernel buffer → cmd2 reads from pipe
7. Shell waits for both to finish

**Key characteristics:**
- **Pipe buffer size:** 64KB on modern Linux (since 2.6.11). Check with `ulimit -a` or `cat /proc/sys/fs/pipe-max-size`
- **Blocking behavior:** If buffer is full, writer blocks. If buffer is empty, reader blocks.
- **Unidirectional:** Data flows one way only. For bidirectional, need two pipes (or use socketpair)
- **No seeking:** Can't lseek() on a pipe. It's a stream.

**Buffering implications for real work:**

```bash
# Problem: Why does `cmd | head -1` sometimes hang or show different behavior?

# stdout buffering modes:
# - Line-buffered: when connected to terminal (flushes on \n)
# - Block-buffered (4KB): when connected to pipe or file (DEFAULT in pipe!)
# - Unbuffered: stderr is always unbuffered

# This means:
tail -f /var/log/app.log | grep 'ERROR'
# grep may not output immediately — it's block-buffered because stdout goes to terminal
# BUT its stdin comes from pipe, and grep may buffer internally

# Fix: force line buffering
tail -f /var/log/app.log | grep --line-buffered 'ERROR'
# Or use stdbuf:
tail -f /var/log/app.log | stdbuf -oL grep 'ERROR'
```

**Named pipes (FIFOs) — persistent pipes:**
```bash
# Create a named pipe (exists as a file)
mkfifo /tmp/mypipe

# Producer (in one terminal)
echo "data" > /tmp/mypipe    # Blocks until consumer reads

# Consumer (in another terminal)
cat /tmp/mypipe              # Reads and unblocks producer

# Use case: inter-process communication without intermediate files
```

**Why this matters in DevOps scripting:**
```bash
# Gotcha 1: Exit codes in pipes (already covered with pipefail)
set -o pipefail
cat /var/log/app.log | grep 'pattern' | wc -l

# Gotcha 2: Variables set in pipe subshell are LOST
count=0
cat file | while read line; do count=$((count+1)); done
echo $count   # Still 0! The while loop ran in a subshell

# Fix: use process substitution or here-string
while read line; do count=$((count+1)); done < <(cat file)
echo $count   # Correct!

# Gotcha 3: SIGPIPE
head -1 /dev/urandom | xxd    # head exits after 1 line
# xxd continues writing → gets SIGPIPE (exit 141 = 128+13)
# This is normal, not an error — but pipefail might catch it
```

---

### Q13: Explain shell variable types — local, export, environment, and positional parameters.
**Project Reference:** Project 9 (OS Patching — Ansible shell modules, bash functions)
**Expected Depth:** Scope rules, inheritance, subshell behavior, practical implications

**Answer:**

**Variable types and scope:**

```bash
# 1. Shell variables (local to current shell — NOT inherited by children)
MY_VAR="hello"
bash -c 'echo $MY_VAR'    # Empty! Child process doesn't see it

# 2. Environment variables (inherited by child processes)
export MY_VAR="hello"
bash -c 'echo $MY_VAR'    # "hello" — child inherits it

# Or in one step:
export DB_HOST="aurora-prod.cluster-xyz.us-east-1.rds.amazonaws.com"

# 3. Inline environment (set for ONE command only)
HTTP_PROXY=http://proxy:3128 curl http://api.internal
# HTTP_PROXY is NOT set in current shell after this
```

**Local variables in functions:**
```bash
# Without 'local' — variable leaks to calling scope
my_func() {
    result="leaked"    # This modifies the parent's namespace!
}
my_func
echo $result    # "leaked" — BAD in complex scripts

# With 'local' — scoped to function
my_func() {
    local result="contained"
    echo $result
}
my_func
echo $result    # Empty — GOOD, no side effects
```

**Positional parameters:**
```bash
#!/bin/bash
# $0 = script name
# $1, $2, ... = arguments
# $# = number of arguments
# $@ = all arguments as separate words (preserves quoting)
# $* = all arguments as single word
# $? = exit code of last command
# $$ = current process PID
# $! = PID of last background process

# Example: Project 9 wrapper script
deploy_patch() {
    local environment="$1"    # dev, staging, prod
    local patch_type="$2"     # security, all
    local hosts="$3"          # hostname pattern
    
    echo "Patching $environment ($patch_type) on $hosts"
    ansible-playbook -i "inventory/${environment}" patch.yml \
        -e "patch_type=${patch_type}" \
        -l "$hosts"
}

# shift — move positional params left
while [ $# -gt 0 ]; do
    case "$1" in
        --env)  ENV="$2"; shift 2 ;;
        --type) TYPE="$2"; shift 2 ;;
        --dry-run) DRY_RUN=true; shift ;;
        *) echo "Unknown: $1"; exit 1 ;;
    esac
done
```

**Subshell behavior (critical to understand):**
```bash
# Parentheses create a subshell — changes don't propagate back
VAR="original"
(VAR="changed"; echo "inside: $VAR")   # "changed"
echo "outside: $VAR"                    # "original"

# Pipes create subshells for each segment!
count=0
echo -e "a\nb\nc" | while read line; do
    count=$((count + 1))
done
echo $count    # 0! (the while ran in a subshell)

# Fix with process substitution:
count=0
while read line; do
    count=$((count + 1))
done < <(echo -e "a\nb\nc")
echo $count    # 3 ✓
```

**Environment in DevOps context:**
```bash
# Ansible shell module — inherits environment from the connection
# If you need specific env vars:
- name: Run with custom environment
  shell: /opt/app/healthcheck.sh
  environment:
    PATH: "/opt/app/bin:{{ ansible_env.PATH }}"
    APP_ENV: production

# In Jenkinsfile — environment block sets for all steps
environment {
    AWS_REGION = 'us-east-1'
    KUBECONFIG = credentials('kubeconfig-prod')
}

# In systemd unit — Environment directive
[Service]
Environment="DB_HOST=aurora.cluster.rds.amazonaws.com"
EnvironmentFile=/etc/app/config.env
```

---

### Q14: Explain here documents, process substitution, and command substitution. When do you use each?
**Project Reference:** Project 9 (OS Patching — generating reports, config files)
**Expected Depth:** Syntax, quoting behavior, real-world patterns, differences

**Answer:**

**Here documents (heredoc) — multi-line string input:**
```bash
# Basic heredoc (variables ARE expanded)
cat <<EOF
Server: $(hostname)
Date: $(date)
Environment: $ENVIRONMENT
Status: Patching complete
EOF

# Quoted delimiter (variables NOT expanded — literal)
cat <<'EOF'
This $variable is literal
No $(command) substitution happens
EOF

# Indented heredoc (<<- strips leading tabs)
if true; then
    cat <<-EOF
	This line has a tab prefix that gets stripped
	Useful for readable indented scripts
	EOF
fi

# Heredoc to a command (common for creating config files)
sudo tee /etc/nginx/conf.d/app.conf > /dev/null <<EOF
server {
    listen 80;
    server_name ${APP_DOMAIN};
    location / {
        proxy_pass http://localhost:${APP_PORT};
    }
}
EOF
```

**Real Project 9 usage — generating ServiceNow update payload:**
```bash
generate_cr_update() {
    cat <<EOF
{
    "state": "implement",
    "work_notes": "Patching batch ${BATCH_NUM}/${TOTAL_BATCHES} complete.\nServers: ${SERVERS}\nResult: ${RESULT}",
    "assigned_to": "${ASSIGNED_TO}"
}
EOF
}

curl -X PATCH "https://instance.service-now.com/api/now/table/change_request/${CR_ID}" \
    -H "Content-Type: application/json" \
    -d "$(generate_cr_update)"
```

**Process substitution — treat command output as a file:**
```bash
# Syntax: <(command) creates a temporary file descriptor
# Useful when a command expects a FILE argument, not stdin

# Compare two remote configs without temp files
diff <(ssh server1 cat /etc/nginx/nginx.conf) <(ssh server2 cat /etc/nginx/nginx.conf)

# Compare before/after (Project 9 — pre vs post-patch package list)
diff <(cat /tmp/packages_before.txt) <(rpm -qa | sort) > /tmp/package_changes.txt

# Feed multiple "files" to a command
paste <(cut -d: -f1 /etc/passwd) <(cut -d: -f7 /etc/passwd)

# Use with while read (avoids subshell problem of pipes!)
while read -r line; do
    process "$line"
done < <(find /var/log -name "*.log" -mtime +30)
# Variables modified inside the loop persist!
```

**Command substitution — capture command output as string:**
```bash
# Modern syntax (preferred, nestable)
KERNEL_VERSION=$(uname -r)
INSTANCE_ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

# Backtick syntax (legacy, harder to nest)
KERNEL_VERSION=`uname -r`

# Nesting (only possible with $() syntax)
FILE_COUNT=$(find /var/log -name "$(date +%Y%m%d)*.log" | wc -l)

# In Project 1 Jenkinsfile:
IMAGE_TAG=$(git rev-parse --short HEAD)
```

**Comparison table:**

| Feature | Heredoc (`<<EOF`) | Process Sub (`<(cmd)`) | Command Sub (`$(cmd)`) |
|---------|-------------------|----------------------|----------------------|
| Returns | Multi-line stdin | File descriptor (path) | String (stdout captured) |
| Use case | Feed multi-line input | Where filename expected | Capture output in variable |
| Example | `mysql <<EOF` | `diff <(cmd1) <(cmd2)` | `VER=$(uname -r)` |
| Quoting | `<<'EOF'` = no expansion | N/A | Always evaluated |

**Here string (<<<) — single-line input shortcut:**
```bash
# Instead of: echo "string" | command
# Use: command <<< "string"

grep -c 'error' <<< "$LOG_OUTPUT"
read -r first rest <<< "hello world foo"
# first="hello", rest="world foo"

# Useful in Ansible shell modules:
bc <<< "scale=2; $USED / $TOTAL * 100"
```

---
