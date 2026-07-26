# Interview Q&A Generation Framework

## Purpose

This file defines the COMPLETE structure for generating interview Q&A banks for any project. Point to any project `.md` file + this framework → generate a comprehensive Q&A covering all interview dimensions.

**Usage:** "Read [project-file.md] and this framework file, then generate a full Interview Q&A bank."

---

## Q&A Generation Categories (7 Dimensions)

When generating Q&A for any project, cover ALL 7 categories below:

---

## Category 1: Project Story (STAR Format)

Generate questions that test the candidate's ability to tell their project story concisely.

### Questions to Generate:

1. "Walk me through this project in 2 minutes."
2. "What was the business problem you were solving?"
3. "Why did you choose this approach over alternatives?"
4. "What was the most challenging part?"
5. "What would you do differently if you started over?"
6. "What was the measurable impact/result?"
7. "How long did it take to implement? Who was involved?"
8. "What trade-offs did you make and why?"

### Answer Format:

```
Situation: [Business context, pain point, scale]
Task: [Your responsibility, what was expected]
Action: [What YOU specifically designed/built/decided]
Result: [Measurable outcome — numbers, time saved, incidents prevented]
```

---

## Category 2: Technical Deep-Dive (How Does It Work Under the Hood)

Generate questions that go ONE LEVEL DEEPER than the project description. Interviewer wants to verify you actually built it, not just read about it.

### Question Patterns to Generate:

**Architecture:**
- "Draw the architecture on a whiteboard. Explain data flow."
- "Why this component? What alternatives did you consider?"
- "What happens when [component X] fails?"
- "How does [component X] communicate with [component Y]?"
- "What are the single points of failure?"

**Implementation Details:**
- "How does [specific technology] work internally?"
- "What configuration was critical and why?"
- "What's the exact sequence of events when [action] happens?"
- "How do you handle [edge case]?"
- "What are the default values for [X] and why did you change them?"

**Scale & Performance:**
- "How does this scale? What's the bottleneck?"
- "What's the throughput/latency you achieved?"
- "What happens at 10x current load?"
- "How do you monitor performance of this system?"

**Security:**
- "How is authentication/authorization handled?"
- "Where are secrets stored? How are they rotated?"
- "What's the blast radius if [component] is compromised?"
- "What compliance requirements does this meet?"

---

## Category 3: Troubleshooting & Incident Scenarios

Generate realistic "something broke" scenarios based on the project. Tests real-world debugging ability.

### Question Patterns to Generate:

- "It's 3 AM, [component] is down. Walk me through your troubleshooting."
- "[Specific error/symptom]. What's your first step?"
- "Users report [symptom]. How do you diagnose?"
- "This worked yesterday but broke today. What changed?"
- "Monitoring shows [metric] spiking. What do you do?"
- "[Component A] can't reach [Component B]. How do you debug?"
- "Deployment succeeded but application is returning errors. Next steps?"
- "Performance degraded by 50% after latest release. How do you investigate?"

### Answer Format:

```
1. Verify the symptom (don't assume — confirm what's actually broken)
2. Check monitoring/logs (what signals are available?)
3. Identify recent changes (deployments, config changes, traffic patterns)
4. Narrow the blast radius (which component, which layer?)
5. Root cause analysis (use specific tools: kubectl, aws cli, logs)
6. Fix (immediate mitigation vs permanent fix)
7. Post-incident (what prevented this from happening again?)
```

---

## Category 4: System Design (Architecture-Level Thinking)

Generate "design from scratch" questions related to the project's domain. Tests ability to think at system level, not just implementation level.

### Question Patterns to Generate:

- "If you had to design [project domain] from scratch for a company 10x your size, how would you?"
- "A new company asks you to build [similar system]. What's your architecture?"
- "How would you evolve this to handle [new requirement]?"
- "Your current design handles X. What changes for 100X?"
- "Compare your approach with [alternative approach]. When would you choose the other?"

### Answer Structure (for system design answers):

```
1. Clarify requirements (ask questions first!)
   - Scale: how many users/requests/data?
   - SLA: uptime requirement?
   - Budget: any constraints?
   - Team: who maintains this?

2. High-level architecture (draw boxes and arrows)
   - Components and their responsibilities
   - Data flow direction
   - External dependencies

3. Deep-dive on critical components
   - Why this technology choice?
   - How does it scale?
   - What happens on failure?

4. Trade-offs discussed
   - What you optimized for
   - What you sacrificed
   - When you'd choose differently

5. Operational considerations
   - Monitoring & alerting
   - Deployment strategy
   - Cost estimation
   - Security posture
```

---

## Category 5: Comparison & Decision-Making

Generate questions that test WHY you chose specific tools/approaches. Shows experience and judgment.

### Question Patterns to Generate:

**Tool Choices:**
- "Why [tool X] over [tool Y]?" (e.g., Jenkins vs GitLab CI, Ansible vs Terraform)
- "When would [tool X] be the WRONG choice?"
- "If you were starting today, would you still choose [tool X]?"
- "Company is using [different tool]. How would you migrate?"

**Architecture Decisions:**
- "Why [pattern X] over [pattern Y]?" (e.g., active-passive vs active-active)
- "What are the downsides of your approach?"
- "At what point does your current approach break?"
- "How do you evaluate new tools for your stack?"

**Process Decisions:**
- "Why this workflow over [alternative]?"
- "How did you get team buy-in for this approach?"
- "What did you try that DIDN'T work before landing on this?"

### Answer Format:

```
Context: [What were the constraints/requirements]
Options Considered: [List 2-3 alternatives]
Decision: [What you chose]
Reasoning: [Why — tied to specific requirements]
Trade-offs: [What you gave up]
Validation: [How you know it was the right choice — metrics]
```

---

## Category 6: Behavioral & Leadership

Generate behavioral questions relevant to the project's context. At 12 YOE, every technical project has a human/leadership dimension.

### Question Patterns to Generate:

**Influence & Leadership:**
- "How did you convince the team/management to invest in this?"
- "Was there resistance? How did you handle it?"
- "Who were the stakeholders and how did you manage expectations?"
- "How did you prioritize this against competing priorities?"

**Collaboration:**
- "How did you work with other teams on this?"
- "What was the hardest cross-team coordination challenge?"
- "How did you handle disagreements on technical decisions?"
- "How did you ensure knowledge transfer after building this?"

**Failure & Learning:**
- "What went wrong during implementation?"
- "Tell me about a production incident related to this system."
- "What would you do differently?"
- "What did this project teach you?"

**Mentoring:**
- "How did you involve junior engineers in this project?"
- "How do you explain [complex concept from project] to a junior?"
- "How did you upskill the team on [new technology in project]?"

### Answer Format (STAR):

```
Situation: [Context — when, where, what was happening]
Task: [Your specific responsibility]
Action: [What YOU did — not the team, YOU]
Result: [Outcome — quantified if possible]
Learning: [What you took away for the future]
```

---

## Category 7: Future & Improvement

Generate questions about evolution, next steps, and industry awareness. Shows you think beyond just building — you think about maintaining and evolving.

### Question Patterns to Generate:

- "What's on your roadmap for improving this?"
- "What's the technical debt in this system?"
- "If budget doubled, what would you add?"
- "How would AI/ML improve this system?"
- "What industry trends affect this project's future?"
- "What's the 3-year vision for this system?"
- "What would make you redesign this from scratch?"

---

## Additional Cross-Cutting Questions (Apply to ANY Project)

These apply regardless of project domain:

### Monitoring & Observability
- "How do you know this system is healthy?"
- "What alerts do you have? What's the on-call experience?"
- "How do you distinguish noise from real issues?"
- "What dashboards exist? Who looks at them?"

### Cost & Efficiency
- "What does this cost to run monthly?"
- "How did you optimize cost?"
- "What's the cost-per-transaction/request?"
- "How do you prevent cost creep?"

### Security & Compliance
- "How is this audited?"
- "What's the attack surface?"
- "How do you handle secrets rotation?"
- "What compliance standards does this meet?"

### Deployment & Operations
- "How do you deploy changes to this system?"
- "What's the rollback strategy?"
- "How do you handle zero-downtime upgrades?"
- "What's the runbook for [common failure scenario]?"

---

## Output Format for Generated Q&A Bank

When generating the actual Q&A bank, use this structure:

```markdown
# Interview Q&A Bank: [Project Name]

## Section 1: Project Story (5-8 questions)
### Q1: [Question]
**Answer:** [Concise, structured answer]
**Follow-up they might ask:** [Likely follow-up]

## Section 2: Technical Deep-Dive (10-15 questions)
### Q1: [Question]
**Answer:** [Detailed technical answer]
**Key terms to mention:** [Important keywords/concepts]

## Section 3: Troubleshooting Scenarios (5-8 questions)
### Q1: [Scenario]
**Answer:** [Step-by-step debugging approach]
**Tools/commands you'd use:** [Specific tools]

## Section 4: System Design (3-5 questions)
### Q1: [Design question]
**Answer:** [Structured design answer with trade-offs]

## Section 5: Comparison & Decisions (5-8 questions)
### Q1: [Why did you choose X?]
**Answer:** [Context → Options → Decision → Reasoning]

## Section 6: Behavioral (5-8 questions)
### Q1: [Behavioral question]
**Answer (STAR):** [Situation → Task → Action → Result]

## Section 7: Future & Improvements (3-5 questions)
### Q1: [What would you improve?]
**Answer:** [Honest assessment with concrete plan]
```

---

## How to Use This Framework

### Step 1: Point to a project file
"Read `/path/to/Project-XX.md`"

### Step 2: Reference this framework
"Use the Interview-QA-Framework.md to generate a complete Q&A bank"

### Step 3: Specify any focus
Optional: "Focus more on [troubleshooting / system design / behavioral]"

### Step 4: Output
I'll generate a complete Q&A bank covering all 7 dimensions, tailored to that specific project.

---

## Quantity Guide

For a thorough preparation per project:

| Category | Questions | Total Time to Review |
|---|---|---|
| Project Story | 5-8 | 15 min |
| Technical Deep-Dive | 10-15 | 30 min |
| Troubleshooting | 5-8 | 20 min |
| System Design | 3-5 | 30 min |
| Comparison & Decisions | 5-8 | 15 min |
| Behavioral | 5-8 | 20 min |
| Future & Improvements | 3-5 | 10 min |
| **TOTAL per project** | **~40-55 questions** | **~2.5 hours** |

For 10 projects: ~400-550 questions total. But many overlap across projects, so effective unique questions: ~250-300.

---

## Priority Order (If Limited Time)

If you can only prep some categories:

1. **Project Story** — You WILL be asked this. Non-negotiable.
2. **Technical Deep-Dive** — Most interview time spent here.
3. **Troubleshooting** — Separates senior from mid-level.
4. **Behavioral** — Can't fake this. Prepare real stories.
5. **Comparison & Decisions** — Shows judgment and experience.
6. **System Design** — Only if company has a dedicated design round.
7. **Future & Improvements** — Nice-to-have, shows growth mindset.
