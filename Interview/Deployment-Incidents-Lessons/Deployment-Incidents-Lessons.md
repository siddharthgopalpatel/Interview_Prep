# Deployment Incidents Lessons — Interview Q&A

> Extracted from the consolidated General Interview Q&A Bank.
> Source sections: 9

## Candidate Context
- **Name:** Siddharth Patel
- **Employer:** Ericsson (entire career — 10+ years)
- **Career Arc:** Linux Admin (2015–2018) → DevOps Engineer (2018–2025) → DevSecOps Engineer (2025–Present)

---

# SECTION 9: Deployment Incidents & Lessons

*Note: Terraform-specific Q&A has been moved to [Terraform-Interview-QA.md](terraform/Terraform-Interview-QA.md)*

---

### Q: Can you recall from your experience where you were deploying something and it caused an outage or didn't go as planned?

**Project Reference:** P9 (OS Patching), P8 (DR drills)

**Answer:**

> "Yes. Early in the OS patching project, before we had full automation:
>
> **What happened:** We were patching a batch of RHEL servers manually. A kernel update required a reboot. After reboot, the application service started — but a shared library (.so file) had been updated by the patch, and the application binary was linked against the old version. Service came up, passed basic systemctl checks, but started throwing segfaults under load.
>
> **Impact:** 15-minute degraded performance on the contact center platform. Calls were routing but with audio quality issues.
>
> **Root cause:** We checked 'is the service running?' but not 'is the service actually healthy under traffic?' No deep validation.
>
> **What I built after this:**
> - 7-dimension automated validation in P9 (services, ports, connectivity, disk, certs, integrity checks, log errors)
> - **Integrity check dimension** — md5sum of 8 critical application files compared against baseline. If a library changes unexpectedly, automation catches it before traffic returns.
> - **Traffic drain before patching** — ALB removes the server from rotation BEFORE we patch. Traffic only returns AFTER all 7 validations pass.
>
> That incident is why our patching system has zero incidents for 18 months since automation went live."

---
---

