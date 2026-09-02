# Ansible — Interview Q&A

> Extracted from the consolidated General Interview Q&A Bank.
> Source sections: 24, 38, 42

## Candidate Context
- **Name:** Siddharth Patel
- **Employer:** Ericsson (entire career — 10+ years)
- **Career Arc:** Linux Admin (2015–2018) → DevOps Engineer (2018–2025) → DevSecOps Engineer (2025–Present)

---

# SECTION 24: Ansible

---

### Q: How do you download modules in Ansible? Which modules have you used?

**Project Reference:** P9 (OS Patching — 12 Ansible roles)

**Answer:**

> "**Terminology clarification:** Ansible has 'modules' (task-level plugins like `yum`, `copy`, `service`) and 'collections' (packages of modules + roles from Ansible Galaxy).
>
> **Downloading collections:**
> ```bash
> # From Ansible Galaxy
> ansible-galaxy collection install amazon.aws
> ansible-galaxy collection install community.general
>
> # From requirements file (what we use in CI)
> ansible-galaxy install -r requirements.yml
> ```
>
> `requirements.yml`:
> ```yaml
> collections:
>   - name: amazon.aws
>     version: \">=6.0.0\"
>   - name: community.general
>     version: \">=7.0.0\"
> ```
>
> **Modules I use regularly (P9 OS Patching):**
>
> | Module | Use Case |
> |---|---|
> | `ansible.builtin.dnf` | Install/update packages (patching) |
> | `ansible.builtin.service` | Start/stop/restart services |
> | `ansible.builtin.systemd` | Systemd service management |
> | `ansible.builtin.copy` | Deploy config files |
> | `ansible.builtin.template` | Jinja2 templated configs |
> | `ansible.builtin.shell` | Custom commands (ss, md5sum checks) |
> | `ansible.builtin.stat` | Check file existence/permissions |
> | `ansible.builtin.reboot` | Controlled reboot with wait |
> | `ansible.builtin.wait_for` | Wait for port/service availability |
> | `ansible.builtin.uri` | HTTP health checks |
> | `amazon.aws.ec2_instance` | EC2 management |
> | `community.general.nmcli` | Network configuration |
>
> All modules are idempotent — run them 10 times, same result. That's why Ansible works for patching: re-run is safe."

---

---
---

# SECTION 38: Ansible Fundamentals

---

### Q: Why is Ansible used?

**Project Reference:** P9 (OS Patching — 500+ servers)

**Answer:**

> "Ansible is used for **configuration management and orchestration of existing servers** — things Terraform can't do.
>
> **Why Ansible specifically:**
>
> 1. **Agentless** — Uses SSH. No agent to install/maintain on 500+ servers. Just SSH access and Python (already on every Linux server).
>
> 2. **Idempotent** — Run it 10 times, same result. Safe to re-run. If package is already installed, it skips. This is critical for patching — re-running after a failure is safe.
>
> 3. **Declarative + Procedural** — Describe desired state (package: latest) but also support ordered steps (stop service → patch → validate → start service).
>
> 4. **Inventory-based** — Target groups of servers by role, environment, location. 'Patch all web servers in us-east-1 but not databases.'
>
> 5. **Human-readable** — YAML playbooks. Operations team can read and understand what automation does without programming knowledge.
>
> **Our use cases:**
> - OS patching (P9) — 500+ servers, 12 roles, 18-step lifecycle
> - Initial server configuration (post-AMI-launch baseline)
> - Application deployment to VMs (before we migrated to K8s)
> - Certificate rotation across fleet
>
> **Ansible vs Terraform:** Terraform creates infrastructure (VPC, EC2, RDS). Ansible configures what's ON the infrastructure (packages, services, files). They complement, not compete."

---

---
---

# SECTION 42: Configuration Management

---

### Q: Tell me a couple of Ansible handlers you have used.

**Project Reference:** P9 (OS Patching — service restart handlers)

**Answer:**

> "Handlers are tasks that run ONLY when notified — typically for service restarts after config changes.
>
> **Handlers I use:**
>
> ```yaml
> handlers:
>   - name: restart httpd
>     ansible.builtin.service:
>       name: httpd
>       state: restarted
>
>   - name: reload nginx
>     ansible.builtin.service:
>       name: nginx
>       state: reloaded
>
>   - name: restart sshd
>     ansible.builtin.service:
>       name: sshd
>       state: restarted
>
>   - name: daemon-reload
>     ansible.builtin.systemd:
>       daemon_reload: yes
> ```
>
> **Usage in tasks:**
> ```yaml
> - name: Update sshd config
>   ansible.builtin.template:
>     src: sshd_config.j2
>     dest: /etc/ssh/sshd_config
>   notify: restart sshd       # Only restarts if file actually changed
> ```
>
> **Why handlers matter:**
> - Handler runs ONCE at the end of the play, even if notified multiple times (efficient — don't restart a service 5 times)
> - Only runs if the task actually CHANGED something (idempotent — if config is already correct, no unnecessary restart)
> - In our patching (P9): after updating packages, handler restarts affected services only if the package was actually updated"

---

### Q: What is the key differentiator between Ansible and other tools like Chef and Puppet?

**Project Reference:** P9 (OS Patching — chose Ansible)

**Answer:**

> "**Ansible is agentless.** That's the #1 differentiator.
>
> | Aspect | Ansible | Chef / Puppet |
> |---|---|---|
> | **Agent** | None — uses SSH | Requires agent on every node |
> | **Architecture** | Push-based (you trigger it) | Pull-based (agent polls server periodically) |
> | **Language** | YAML (simple, readable) | Ruby DSL (Chef) / Puppet DSL (steeper learning curve) |
> | **Setup effort** | Minimal — SSH + Python on target | Install agent, configure master server, certificates |
> | **State** | Stateless (no central DB of node state) | Stateful (master tracks node state) |
> | **Scaling** | Ansible AAP/Tower for enterprise | Chef Server / Puppet Master |
>
> **Why we chose Ansible (P9):**
> - 500+ servers already had SSH access — no agent deployment required
> - Operations team could read YAML playbooks immediately — no Ruby learning curve
> - Push-based fits our patching model (controlled trigger, not continuous enforcement)
> - Ansible Automation Platform (AAP) gave us enterprise features (RBAC, scheduling, audit, API)
>
> **When Chef/Puppet might be better:**
> - Continuous enforcement (desired state every 30 minutes) — large fleet that must ALWAYS match policy
> - Already have Chef/Puppet infrastructure invested
>
> For our use case (orchestrated patching, one-time configuration), push-based Ansible is the natural fit."

---
---

