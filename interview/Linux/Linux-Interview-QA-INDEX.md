# Linux Interview Q&A Bank — Master Index

## For: Senior DevOps & Cloud Engineer (12 YOE)
## Total Questions: 90+
## Tied to: All 9 Portfolio Projects

---

## How to Use This

1. **Before interview:** Review Category 11 (Real-World Scenarios) first — these are the most common
2. **Deep prep:** Go through Categories 1-10 systematically
3. **Quick refresh:** Each answer includes the project reference — tie your answer to YOUR real experience
4. **Priority order:** Categories 3, 4, 1, 5, 6 are highest probability for 12 YOE DevOps role

---

## Files & Categories

| # | Category | File | Questions | Priority |
|---|----------|------|-----------|----------|
| 1 | Package Management & Patching | `Linux-Package-Management-Patching-QA.md` | Q1-Q10 | 🔴 High |
| 2 | systemd & Service Management | `Linux-Systemd-Service-Management-QA.md` | Q11-Q20 | 🔴 High |
| 3 | Kernel — Cgroups, Namespaces, Containers | `Linux-Kernel-Networking-Interview-QA.md` (Part 1) | Q1-Q10 | 🔴 Very High |
| 4 | Networking — iptables, DNS, Troubleshooting | `Linux-Kernel-Networking-Interview-QA.md` (Part 2) | Q11-Q22 | 🔴 Very High |
| 5 | Process Management & Signals | `Linux-Part3-Category5-Process-Management.md` | Q1-Q8 | 🟠 High |
| 6 | Storage & Filesystem | `Linux-Part3-Category6-Storage-Filesystem.md` | Q9-Q16 | 🟠 High |
| 7 | Security — SELinux, Permissions, SSH, Certs | `Linux-Part3-Category7-Security-Part1.md` + `Part2.md` | Q17-Q26 | 🟠 High |
| 8 | Boot Process & Troubleshooting | `Linux-QA-Part4-Boot-Process.md` | Q1-Q6 | 🟡 Medium |
| 9 | Shell Scripting & Text Processing | `Linux-QA-Part4-Shell-Scripting-1.md` + `2.md` | Q7-Q14 | 🟡 Medium |
| 10 | Performance Troubleshooting | `Linux-QA-Part4-Performance-1.md` + `2.md` | Q15-Q22 | 🟠 High |
| 11 | Real-World Scenario Questions | `Linux-QA-Part4-Scenarios-1.md` + `2.md` | Q23-Q32 | 🔴 Very High |

---

## Quick Reference: Project → Linux Topics Triggered

| Project | Linux Areas Interviewer Will Probe |
|---|---|
| **P9: OS Patching** | dnf/yum, rpm, kernel mgmt, systemd, ss, journalctl, openssl, nc, SELinux, fstab, firewalld, LVM, md5sum |
| **P3: Kubernetes** | cgroups, namespaces, kernel modules, sysctl, iptables/IPVS, overlayFS, containerd, swap, PID 1, OOM killer |
| **P1: DevSecOps** | Docker internals, shell scripting, sed, exit codes, process signals, non-root containers, SSH |
| **P2: 3-Tier AWS** | systemd services, cloud-init, EC2 networking, disk management, log rotation, CloudWatch agent |
| **P8: HA/DR** | Boot process, DNS/resolv.conf, NTP/chrony, cloud-init, systemd ordering, certificates |
| **P6: Istio** | Network namespaces, iptables NAT, tcpdump, port inspection (ss), cgroups overhead |
| **P7: FinOps** | CPU/memory monitoring (top, vmstat), disk types, x86 vs ARM, instance boot |
| **P4: Landing Zone** | SSH/key management, network troubleshooting tools (traceroute, dig, nc) |
| **P5: Serverless** | Firewall concepts (maps to iptables), Python runtime |

---

## Interview Strategy Tips

1. **Always tie Linux answers to your projects** — "In my OS patching automation handling 500+ servers..."
2. **Show depth, not breadth** — Pick one topic and go 3 levels deep rather than surface-level multiple topics
3. **Mention the WHY** — Not just "I used ss -tlnp" but "I used ss because netstat is deprecated and ss reads directly from kernel socket tables via netlink"
4. **Show automation mindset** — "I don't just run the command, I automated this check in my Ansible role's validation dimension"
5. **Acknowledge trade-offs** — "We chose XFS over EXT4 because we needed parallel I/O for the database workload, accepting the no-shrink limitation"

---

## Top 15 Most Likely Questions (Based on 12 YOE DevOps Profile)

1. Walk me through what happens when a container starts at the kernel level
2. How does Kubernetes use cgroups and namespaces?
3. A pod is OOMKilled — troubleshoot from the node
4. How does kube-proxy route traffic using iptables?
5. Your production server is slow — walk me through troubleshooting
6. How does the Linux boot process work? (BIOS → GRUB → kernel → systemd)
7. Explain iptables tables and chains — how does a packet traverse them?
8. How do you safely patch 500+ Linux servers with zero downtime?
9. A service won't start after deployment — systematic troubleshooting
10. DNS works with `dig` but application can't connect — why?
11. Difference between SIGTERM and SIGKILL — how does K8s use them?
12. How do you resize a disk on a running Linux instance?
13. What is SELinux and how do you troubleshoot AVC denials?
14. Explain the difference between cgroups v1 and v2
15. A process is at 100% CPU — how do you investigate without killing it?
