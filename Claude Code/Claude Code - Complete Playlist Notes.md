# Chapter 1 — Claude Code Slash Commands

## What Are Slash Commands
- Shortcuts you type inside a Claude Code session, starting with `/`, that trigger a specific predefined action or workflow instantly — without writing a full prompt.
- One word can invoke an entire repeatable workflow.

## Why They Exist
- Developers repeat the same patterns/workflows while coding. Claude Code turned these reusable patterns into commands.
- Saves you from re-explaining the same thing to the AI again and again.

## Two Types
- **Built-in commands** — come with Claude Code by default after installation.
- **Custom commands** — created by users for their own project-specific, repeated workflows.

## Sessions (important supporting concept)
- A session = one conversation with Claude Code — everything from when you run `claude` to when you type `/exit`.
- It has a **unique ID**, and captures the **full message history**, all **file reads**, all **tool results**, and the **current working directory**.
- Sessions are saved automatically to **`~/.claude/projects/`** and can be resumed at any time, even after closing the terminal.
- **`claude -r`** → resume a past conversation (shows list of past sessions to pick from).

## Key Slash Commands

| Command | What it does |
|---|---|
| `/exit` | Close the current session |
| `/resume` | Jump from current session into another session |
| `/rename <name>` | Rename current session (do this immediately for easy future reference) |
| `/btw <question>` | Ask a side question WITHOUT polluting main context/history (press space to dismiss the answer) |
| `/export <file.md>` | Export the conversation to a file in your project directory |
| `/logout` | Log out of Claude Code account (useful for switching personal ↔ company account) |
| `/login` | Log in (choose theme → login method) |
| `/model` | Switch between models (Opus / Sonnet / Haiku) |
| `/usage` | Check token usage (current session % + weekly limit %) |
| `/stats` | Usage statistics (tokens, models used, sessions, streak, longest session) |
| `/insights` | Detailed HTML report on your usage patterns + how to improve |
| `/config` | Change settings (thinking mode, verbose, progress bar, language, etc.) |
| `/permissions` | Allow/deny tools (Allow, Ask, Deny, Workspace tabs) |
| `/theme` | Change appearance (dark/light + more) |
| `/voice` | Enable voice mode (hold Space to speak instead of typing) |

## Session Best Practices
- **One session = one task** — e.g., one feature = one session. Close it when the feature is done, start a new one for the next. Keeps context windows clean and separated.
- **Name your session immediately** — otherwise AI auto-names it based on your first question (not ideal). Name by feature (e.g., login, registration).
- **Commit frequently within a session** — create a commit whenever you hit an important milestone.
- **Use `/btw` for quick questions** — side questions that shouldn't pollute the main context.
- **Export a session before a big refactor** — save it as context to feed back during refactoring.

## Models
- **Opus** — most powerful but most expensive. Used for complex programming tasks; burns tokens fast.
- **Sonnet** — the default for most users. Best balance of speed, quality and token cost. Good for everyday coding tasks.
- **Haiku** — fastest and cheapest. Use for simple, repetitive or exploratory tasks where you don't need deep reasoning.
- **Common power-user pattern:** use **Opus for the planning phase** (thinking through architecture, writing specs, making decisions) → then switch to **Sonnet for the implementation phase** where the thinking is done and you just need reliable code generation.

## Usage & Limits
- Claude tracks usage two ways: **per-session tokens** + **weekly limit**. Stay within both.
- **`/extra-usage`** — top up mid-way ($5/$10 etc.) instead of waiting for the reset if you hit the limit.

## Permissions (deeper concept)
- Claude Code = an **agent built on top of an LLM**. The LLM is the "brain"; **tools** let it act.
- Tools include: Read (read files), Write (write code), Bash (run commands), Web Search (fetch docs from internet). You can add your own tools via **MCP**.
- By default Claude asks permission before using a tool — repetitive and frustrating.
- `/permissions` tabs: **Allow / Ask / Deny / Workspace**.
  - **Allow** → tool runs without asking (e.g., add `WebSearch`, or a Bash command like `Bash(git init)`).
  - **Ask** → asks every time.
  - **Deny** → tool can never be used.
- Save scope options: **Local project** (just you, this project), **Global/project (shared via git)** (applies to collaborators who fork the repo), **User** (all your projects on this machine).
- Settings stored in `.claude/settings.local.json` (permissions → allow list).
- ⚠️ Be very careful — don't auto-allow dangerous commands.

## Handy Tip
- Type `/` alone → scroll with arrow keys to see the **complete list** of commands with descriptions (stretch the window if descriptions are cut off).
- You don't need to memorize all of them — learn each as the need arises.

---
**One-line takeaway:** Slash commands turn repeatable prompts/workflows into instant `/` shortcuts, and are a big reason Claude Code is such a productive tool.


---

# Chapter 2 — Using Claude Code on a Real Project (Landing Page Improvements)

## Goal / Agenda
First practical, hands-on video. Three things to achieve:
1. **Change existing code** using Claude Code.
2. **Create new website pages** using Claude Code.
3. **Use Claude Code's multimodal capability** — provide an image and convert its design into code.

All three are practiced on the website's **landing page**.

## Planned Improvements to the Landing Page
1. **Hero section redesign** — new design: centered text, two buttons, a graphic, and a "See how it works" button. Clicking that button opens a popup (modal) playing a YouTube video explaining the product.
2. **Two new footer links** — Terms & Conditions and Privacy Policy (standard for any site handling transactions).

## Overall Workflow (repeatable flow)
Create a new session → name the session → make changes via prompts → commit changes → push to repository.

- Start a session: type `claude`.
- Name it: `/rename Landing Page Improvements`.
- Make changes with prompts, commit at each meaningful milestone, then `git push origin main` at the end.
- Finish with `/exit`.

## Key Techniques Learned

### 1. The `@` mention (file targeting)
- Use `@<file-path>` in a prompt to tell Claude Code **exactly which file** to work on.
- Without `@`, Claude Code might not pick the right file and could edit the wrong one.
- Makes the workflow **deterministic**. Good practice even on small projects.

### 2. Curate prompts beforehand (don't type blindly)
- Recommended strategy: **don't manually type prompts** directly into Claude Code.
- Plan what you need → write it in your own words → refine it via a chatbot (ChatGPT/Claude) → paste the polished, refined prompt into Claude Code.
- Reduces mistakes and forgotten requirements. All prompts for this video were pre-written in a file.

### 3. Committing frequently
- Commit at every meaningful milestone.
- Use **Bash mode** for git commands. (In the video, `git add` was typed without bash mode first but Claude Code was smart enough to handle it; `git commit` was then run in bash mode.)
- Flow: `git add` → `git commit` → (at the end) `git push origin main`.

### 4. Multimodal capability (image → code)
- Claude Code's models are **multimodal** — they understand text **and** images.
- Paste a design image directly into the terminal (copy image → `Ctrl+V`); the image link appears.
- Provide the image **plus** a prompt (e.g., "Modify only the hero section in `@template/landing` and `@static/css/landing.css` to match this image exactly. Do not touch any other part of the page.").
- Claude Code generates code matching the design very closely.
- Powerful for turning Figma/design-team mockups into working code.

## Step-by-Step Work Done
1. **Footer links** — prompt targeting the file with `@`: add two plain-text links (Terms & Conditions, Privacy Policy), no styling, pointing to `#`, no other changes. → Claude edited `base.html`, showed green (added) lines, asked permission → approved → verified in browser → committed.
2. **Terms & Conditions page** — prompt: create the page for the app, add a new route to `@app.py`, create a template with generic T&C content. → Claude read two files, added a Flask route, created `terms.html`, updated the link. Page looked ugly → follow-up prompt: "make the appearance of `@template/terms.html` match the website's theme" → Claude edited the CSS → verified → committed.
3. **Privacy Policy page** — near-identical prompt (with `@template/landing.html` and theme-matching instruction) → new route, `privacy.html` created, link updated, theme matched → verified → committed.
4. **Hero section redesign (image)** — pasted design image + prompt to modify only the hero section to match the image. (A wrong CSS filename was given, so Claude created a new `landing.css`.) → verified: new hero section matched the design reference very closely → committed.
5. **Modal popup with YouTube video** — detailed prompt with clear requirements:
   - Clicking "See how it works" opens a modal overlay with an embedded YouTube video.
   - Use a placeholder YouTube URL for now.
   - Video must be playable inside the modal.
   - Clicking the close button OR clicking outside the modal closes it.
   - When the modal closes, the video **must stop playing** (not continue in background — a common bug on many sites).
   - No page libraries/dependencies — **vanilla JavaScript only** (no JS framework in this project).
   - Do not modify any other part of the project.
   → Claude updated the link, added modal code and CSS → verified working → committed.
6. **Push** — `git push origin main` → verified new files (privacy.html, terms.html) appear in the repository.
7. Checked usage (`/usage`) — ~14% of session used (small project, no risk of hitting limits) → `/exit`.

## Mistakes & Iterative Prompting (important lesson)
- A wrong CSS filename in a prompt resulted in **two CSS files** (`style.css` and `landing.css`). Fix: give a command like "merge these two files" and commit.
- **A prompt gives an output, but not necessarily the correct output.** You iterate within a session to improve it — prompt → output → refine → repeat.

## Key Takeaways
- Today's simple edit-workflow (direct prompts) is fine for **minor changes**.
- For **complex features**, this is NOT the right way — later videos will cover advanced mechanisms in Claude Code: **plan mode, creating specs, creating branches, creating PRs**.
- You learned: adding a file to Claude Code's context with `@`, adding images, and generating code from an image-based design.


---

# Chapter 3 — Context Window Management

## Two Viewer Concerns Addressed First
1. **"The expense-tracker project is too basic"** — intentional. A simpler project lets you focus on learning Claude Code better. Once you master Claude Code, apply it to complex projects (a data-science domain project is planned later). Doing two complex things at once isn't ideal.
2. **"Why terminal instead of VS Code plugin / desktop app?"** — Claude Code has multiple access modes: terminal, desktop app, web, and a VS Code (GUI) plugin. Terminal is the **OG / most powerful** way and how power users work. GUI is useful for some things (e.g., `git diff`), but terminal unlocks full potential. Later videos cover the other modes too.

## What is Context?
- **Context = all the information available to understand something correctly.**
- In programming, context comes from many sources: the whole **codebase**, the **PRD / spec document**, **Jira / GitHub issues**, **Slack messages**, **previous AI chats**, and the **git repository**.
- A programmer (or a tool like Claude Code) needs all this context to write the next line of code.

## What is the Context Window?
- **Definition:** the amount of information (in **tokens**) that a model like Claude Code can see and remember at one time while generating a response. Think of it as the model's **working memory**.
- LLMs can't handle infinite context — each has a limit measured in **tokens**. That limit = the context window.

## Claude Code Context Window — Key Facts
1. **Size ≈ 200K tokens** (for Sonnet models). The newer **Opus 4.6** is said to have **1 million tokens**, but other models give ~200K.
2. **Each new session = a fresh context window.** Starting a new session means starting from scratch.
3. **Tokens are consumed by BOTH your messages (input tokens) AND Claude's replies (output tokens).** Everything counts — generated code, tool usage, tool outputs.
   - Rough estimate: Claude's output is **~6× larger** than the message you send, so output fills the window ~6× faster. Be mindful of this.

## Conversation Growth (Why Split Sessions)
- LLMs are **stateless** — no memory. Each new request re-sends the **entire conversation history** as input tokens.
- **Magnitude example** (assume 100 tokens per message, 100 per reply):
  - Turn 1: 200 tokens. Turn 2: 400. Turn 3: 600. … By turn 10: ~2000 tokens consumed.
- The longer the session, the **faster you hit the context limit**.
- Building 4 features in one session ≈ **4× more tokens** than giving each feature its own session (which uses ~1/4).
- **Takeaway: give each feature its own session.**

## Sub-Agents & Isolated Context Windows
- Claude Code is itself an agent, but it can spawn **sub-agents** under it.
- Each sub-agent has its **own isolated context window** (separate from the main agent's, ~200K).
- Trigger multiple sub-agents to run tasks **in parallel**.
- When done, a sub-agent returns a **summary** to the main agent (not all its tokens) — a very efficient way to handle context.
- (Sub-agents are covered in a future video; introduced here so the concept sticks.)

## What Actually Fills the Context Window (200K → ~150K usable)
Like RAM: "200K" advertised, but real usable is ~150K because some is pre-occupied:
- **System prompt** — ~6,000 tokens (instructions defining Claude Code's personality/capabilities), preloaded every session.
- **Tool schemas** — ~8,000 tokens (definitions of each tool's inputs/outputs/capabilities).
- **CLAUDE.md file** — a project overview, preloaded each session (small).
- **Conversation history** — grows as you chat.
- **Tool usage outputs** — verbose outputs from tools used in the session.
- **MCP tool schemas** — if MCP tools are configured.
- **Skills** — markdown files describing how to solve a particular task type (e.g., EDA). Definition loads into context. (Covered later.)
- **Auto-compaction reserve** — **~33,000 tokens** pre-reserved for auto-compaction.
- Net: after ~6K + 8K + 33K + others, roughly **50K is taken**, leaving **~150K usable**.

## Checking Usage with `/context`
- Run `/context` in a session to see the live breakdown: system prompt (~6.3K / 3.1%), system tools (~8.3K), skills (~1 token), free space, auto-compact buffer (~33K), MCP tools, CLAUDE.md, memory.md.
- Plan to work with **~150K usable tokens**.

## Why Context Window Management Matters
1. **Cost** — you pay Anthropic per token. Context window is directly tied to token usage, so managing it optimizes cost.
2. **Workflow structure** — 4 features in 1 session = 4× tokens = 4× cost. One feature per session optimizes both.
3. **Response quality degrades** as the window fills. Around 120K–130K used, response quality is noticeably worse than at 20K–30K. Always be aware of how full your context is.

## Auto-Compaction (Claude's automatic solution)
- Triggers automatically when usage reaches roughly **75%–92%** of the context window.
- Claude takes the whole conversation history, **summarizes** it, stores the summary in the reserved ~33K auto-compaction tokens, and frees the conversation history — giving you space again while keeping context (via the summary).
- Repeats each time you hit 75–92% again (re-summarizing).
- Eventually the 33K reserve fills up with summaries → Claude can no longer continue that session → you must **start a new session**.

## Manual Compaction with `/compact` (your solution)
- `/compact` does the same thing as auto-compaction, on demand.
- **Why prefer manual?** Auto-compaction can trigger **mid-task** (e.g., in the middle of implementing a feature). Summaries are **not lossless** — some essence/detail is lost, so important details could be missed mid-feature.
- **Recommendation:** don't rely on auto-compaction. Periodically check `/context`; when you're around **70–75%** and NOT in the middle of an important task, run `/compact` yourself.
- Demo: `/context` showed ~7.5K tokens in messages after 3–4 questions → ran `/compact` (press `Ctrl+O` to view the generated summary) → `/context` again showed messages down to ~3.4K.
- Note: repeated `/compact` also fills the 33K reserve — there's a limit.

## Other Escape Options (when nothing else works)
- **Sub-agent** — hand off the next task to a sub-agent (fresh ~200K context). Works in some scenarios, not always.
- **`/clear`** — deletes the whole conversation; you're back to the session's start (same session).
- **New session** — cleaner; create a fresh session. (`/clear` and new session are similar; presenter prefers a new session since each feature = one session.)

## Best Practices for Managing Context Window
1. **Develop one feature per session** — don't build multiple features in one session.
2. **Use `/compact` proactively, not reactively** — don't let Claude auto-compact; do it yourself when needed.
3. **Write focused, specific prompts** — no vague commands; clear instructions restrict token usage and save context.
4. **Use sub-agents for isolated / exploratory / parallel work** — don't use the main agent for these.
5. **Use `.claudeignore`** — like `.gitignore`; list files Claude should never touch so their content never enters context (e.g., build files, `.venv`, large files). (Feedback suggests Claude doesn't fully honor it yet, but it should get stricter.)

## Terminal vs GUI (demo at end)
- The GUI (VS Code extension) shows most features, but some are **terminal-only**: e.g., editing **memory** says "Continue in terminal to edit memory," and **hooks** also require terminal.
- The main, power-user way is the terminal; GUI modes are for convenience. Learn the harder/more powerful way first.


---

# Chapter 4 — CLAUDE.md, the .claude Folder & Auto Memory

Mostly theoretical, but foundational — used in every subsequent video.

## Why CLAUDE.md is Needed
- **LLMs have no memory.** Claude Code's brain is Anthropic's LLMs, so it doesn't remember past sessions.
- Every new session (e.g., a new feature) would force you to re-explain the whole project from scratch: database setup, frontend/backend libraries, coding conventions, etc.
- Re-explaining every time is **cumbersome** and **error-prone** (you might forget something → inconsistent code generation). Small projects may survive this; large projects suffer.
- **Solution (first-principles):** create a file storing all project details, and have Claude Code pull it at the start of each session.

## What is CLAUDE.md
- A **special project-level instruction file** used by Claude Code to guide how it behaves while working on your codebase. Think of it as a **persistent system prompt**.
- It's just a **markdown file** with your project's details/instructions. Every new session, Claude Code **automatically pulls and reads it**.
- Located in your **project directory**.
- Removes the need to manually explain project structure, coding conventions, how to run/build/test, and which tools you use — every time.

## Two Ways to Create CLAUDE.md
1. **Manually** — create a markdown file named **`CLAUDE.md`** (must be in **capitals**) in your project directory and add project details.
2. **`/init` command** — Claude Code scans/analyzes your whole codebase and auto-generates a CLAUDE.md.

**Presenter prefers `/init`** because:
- You may be working on someone else's codebase and won't know it fully on day one.
- In a large codebase, you might miss patterns; Claude is better at capturing them.
- You may not know the ideal format for your first CLAUDE.md.
- It's faster.
- (Manual has its place for very fine-grained control. Common practice: generate with `/init`, then edit.)

## How `/init` Works Behind the Scenes
- Starts an **internal agent** that scans the codebase.
- First scans **high-signal config files** (package.json, requirements, README).
- Understands the **directory tree structure**.
- Focuses on **tech stack, folder layout, naming conventions**.
- Generates a CLAUDE.md in the project root with all this context.
- **Important:** the generated file is only **~30% useful**. The other **70%** (workflows, constraints, what to avoid, deployment, naming conventions) is **your job as a programmer** to add. `/init` is a good starting point, not the end point.

## What an Ideal CLAUDE.md Should Contain
1. **Project overview** — a short (one-line) description so Claude immediately knows what it's building/modifying. E.g., "This is a FastAPI backend for a health-tracking application that stores patient PMI records and exposes CRUD APIs."
2. **Architecture** — how the codebase is structured and what belongs where (e.g., "routes live in `routes/`, business logic in `services/`, schemas in `schemas/`").
3. **Coding style / conventions** — how code is written and uniformly followed (e.g., use type hints in Python, prefer Pydantic models, keep functions small and focused).
4. **Preferred libraries & tools** — the tools/frameworks to use and not go beyond (e.g., FastAPI for APIs, Pydantic for validation, SQLAlchemy for ORM).
5. **Commands** — exact commands to run/test/deploy (install from requirements, run dev server, run tests).
6. **Critical rules** — highlight warnings, edge cases, things to avoid (e.g., "don't touch `database.py` unless needed," "don't generate patient IDs yourself"). Very important in practice — clear mentions prevent things going haywire.
- Presenter also added a **development roadmap** (order of routes to build + a status column). Gives Claude a roadmap so all sessions behave consistently and streamline the workflow.
- This is **one guideline**, not the only one — study how other experienced programmers write CLAUDE.md files and develop your own style.

## The .claude Folder
- A **local configuration directory** controlling how Claude Code behaves — for a specific project or across all projects on your machine.
- Stores config for **skills, custom slash commands, sub-agents**, etc. Think of it as Claude Code's **toolbox**.
- (In an earlier video, the `.claude/settings.local.json` for tool permissions was created here.)

### Two Types of .claude Folders
| | Project-level | Global / User-level |
|---|---|---|
| Location | Project **root** directory | Home directory (`~/.claude`) |
| Scope | One project | Your machine, every project |
| Shared with team | **Yes** — committed to repo (git) | **No** — stays on your machine |
| Use for | Project-specific commands, workflows, settings | Personal commands/style you want everywhere |

### Inside a .claude Folder
- **settings.local.json** — which tools are allowed/denied (project-level but personal, not shared).
- **commands/** — your custom slash commands (e.g., a code-review command, an issue-fix command).
- **rules/** — (explained later; used to split CLAUDE.md).
- **skills/** — markdown files describing how to do a particular task type (deployment, data analysis). Auto-loads when needed. (Covered later.)
- **agents/** — sub-agents (e.g., a "code reviewer" sub-agent).
- In the **user-level** folder, there's also a **projects/** folder — each project gets its own directory, and that project's **sessions** are stored there.

## Types of CLAUDE.md Files (by location)
1. **Project root `CLAUDE.md`** — the one `/init` creates; auto-loads every session.
2. **Inside project-level `.claude/` folder** — no difference from root; some people keep all config in one place. Both auto-load.
3. **`CLAUDE.local.md`** — read alongside the main CLAUDE.md and **automatically gitignored**. For your **personal, project-level** preferences/workflows you don't want to share with teammates.
4. **User-level `~/.claude/CLAUDE.md`** — personal preferences that apply **across all projects** (coding style defaults, preferred tools, general working style). Your "programmer persona."
5. **Folder-level CLAUDE.md** — inside a specific subfolder (e.g., a `database/` folder). Starting from the current working directory, Claude Code **recurses up** and reads any `CLAUDE.md` / `CLAUDE.local.md` it finds. Convenient in large repos.
   - **Important:** folder-level files do **NOT** auto-load every session. Only the **root** CLAUDE.md auto-loads. Folder-level loads **only when Claude works inside that folder** and needs its context.

## Best Practices for CLAUDE.md
1. **Start with `/init`**, then remove unnecessary/unuseful things (better than writing manually from scratch).
2. **Commit changes** to CLAUDE.md via git as you go.
3. **Only put universally applicable things** in it — remove anything specific to one feature/section.
4. **Use `IMPORTANT` sparingly** — mark truly critical lines with "IMPORTANT" so Claude always honors them, but don't overuse it. Rule: *"If everything is important, nothing is."*
5. **Keep it under ~200 lines** (200–300 max). As instruction count grows, Claude's instruction-following quality decreases (true for all LLMs). Rule of thumb for each line: *"Will removing this line make Claude make mistakes?"* If no, remove it.

## Three Ways to Reduce CLAUDE.md Size (for big projects)
1. **Split into multiple topic files inside `.claude/rules/`** — e.g., `code-style.md`, `testing.md`, `security.md`, `api-conventions.md`. Benefits: (a) these **lazy-load** — not loaded every session, only when needed (Claude loads `security.md` when it needs security info); (b) better **maintainability**.
2. **Use imports in CLAUDE.md** — like Python imports, load a topic's content from another file (e.g., import API guidelines instead of writing them inline).
3. **Create CLAUDE.md files in subdirectories** — separate files for frontend/backend/database folders; load only when Claude works in that folder.

## More Good Practices
- **Treat CLAUDE.md as a living document** — don't just `/init` once and forget. After each feature: refresh it, add new things, remove irrelevant ones. Build it organically and commit.
- **Codify recurring mistakes** — if Claude repeatedly makes the same mistake, don't just correct it each time; also tell Claude to add the instruction into CLAUDE.md.
- **Audit periodically for instruction drift** — review CLAUDE.md weekly/monthly; old instructions may become redundant, meaningless, or even counterproductive.

## Auto Memory
- **Definition:** a **persistent directory where Claude records learnings, patterns, and insights as it works.**
- As you develop features, Claude silently observes. When it sees something meaningful (a pattern/insight), it saves it to a markdown file named **`memory.md`**.
- Example: if your project consistently tracks expenses in **INR (not USD)**, Claude observes this pattern and saves it to memory.md.
- On the next session, **both CLAUDE.md and memory.md auto-load**, so Claude's learnings carry forward.
- **Location:** `~/.claude/projects/<your-project>/memory/memory.md`.
- **Important:** only the **top ~200 lines** of memory.md load per session, so manage its size.

### Accessing Memory with `/memory`
- `/memory` lets you **view and edit** memories. Three options:
  1. **Project memory** → opens the project's CLAUDE.md.
  2. **User memory** → opens the home-directory CLAUDE.md.
  3. **Open auto-memory folder** → opens memory.md.
- Why does `/memory` show CLAUDE.md files too? Because for Claude Code, **CLAUDE.md and memory.md both act as persistent memory** — both are loaded every new session. The difference: **the programmer writes CLAUDE.md; Claude writes memory.md.** Otherwise, on an overview level, they're all memory files.

### You Can Create Memories Too
- Not only Claude — you can create memories manually. At the end of a long session, prompt: *"Based on whatever we discussed and developed in this session, update your memory.md file."*
- Demo: prompt "update your memory files — we use INR and not USD" → Claude saved it → memory.md now shows "Project uses INR, not USD."
- Mostly Claude does this automatically; the manual option exists.


---

# Chapter 5 — Spec-Driven Development (Vibe Coding vs Agentic Coding)

Spec-Driven Development (SDD) is one of the **core pillars of agentic coding**. This video finally explains the difference between vibe coding and agentic coding.

## What is Vibe Coding?
- **Definition:** a modern style of programming where, instead of carefully planning everything up front, you build software by interacting with an AI assistant in a fast, conversational, and experimental way.
- You tell an AI coding tool (Cursor, Lovable, Replit) in plain English what to build (e.g., "build a to-do application"), it codes on your behalf, you check the result, give feedback, and iterate.
- **Great for non-technical users** and a great way to start/learn programming.

## The Problem with Vibe Coding
- **You lose a lot of control.** The AI makes crucial decisions on your behalf, which causes many mistakes and software that differs from your expectations.
- **Example:** you ask "build me a user authentication system." The AI now faces many crucial questions it must decide itself:
  - Which framework to use?
  - JWT or sessions?
  - Password rules (min 8 chars, numbers, letters, special chars)?
  - What happens after 3 wrong password attempts?
- If its self-made decisions don't match what you wanted, the whole thing gets built, you check it, realize it's wrong, and go back to change it.
- **The biggest flaw:** you get code **fast**, but you may **not get the right code**, and you end up in a **loop of corrections and patches**. Frustrating, especially for big applications.

## Spec-Driven Development (the solution)
- **Core idea:** you keep most of the **control**. All core decisions/questions that should be answered **before coding** are provided to the AI via a **spec document** — a detailed document so the AI doesn't have to make its own decisions; it just studies the doc and codes accordingly.
- **Definition:** a software development approach where a detailed **specification document** is written **before any code is written**. The spec acts as a **single source of truth** for what the system should do, and all development flows from it (you can't go outside it).
- **Philosophically opposite to vibe coding:** vibe coding = lose control to get faster code; SDD = code a bit slower but get a lot of control.

## What a Spec Document Contains (5–6 things)
Format varies company-to-company, developer-to-developer, but generally includes:
1. **Problem statement** — the **WHY** you're building this feature.
2. **Functional requirements** — the exact **WHAT** — the exact things the feature will do/deliver.
3. **Input/output behavior** — how the feature receives input and produces output.
4. **Constraints** — any constraints on the system.
5. **Edge cases** — where the feature can fail and how to handle it.
6. **Acceptance criteria** — the feature is only valid if these are achieved.

### Example: Chat History Sidebar
- **Problem statement:** users create multiple conversations over time but have no easy way to revisit/continue past chats → introduce a chat history sidebar.
- **Functional requirements:** display a sidebar listing past chats; show a short readable title per chat; auto-generate title from user's first message; allow clicking any chat to open it.
- **Input/output:** input = user clicks a past conversation; output = that chat opens in the main area.
- **Constraints:** sidebar loads within a second; works on standard laptop screens; titles short & readable; handles a reasonable number of chats smoothly.
- **Edge cases:** no chats yet → show "No chat history yet"; chat can't be loaded → show a message; very long first message → use only its first part for the title.
- **Acceptance criteria:** feature is complete if user can see a list of past chats, correct conversation is displayed each time, new chat appears automatically after first use.

## The Full Spec-Driven Development Workflow
1. **Create a spec document** (non-technical, WHY & WHAT — like a PRD; usually made by product managers with engineering teams).
2. **Review the spec document** (you may have missed something or made a mistake).
3. **Create a technical design plan** — the **HOW** document. Converts the spec into a technical implementation plan (usually made by the engineering team for developers).
4. **Review the technical design plan.**
5. **Extract a set of tasks** from the technical design plan.
6. **Build the tasks** (start coding).
7. **Validate** the code against the spec document's acceptance criteria.

### Technical Design Plan — what's in it
- **Objective**
- **Tech stack decision** (e.g., React for fast UI, FastAPI for high-concurrency chat systems, relational DB to store chats)
- **High-level architecture** (frontend → backend → database)
- **Data model** (how the DB is organized)
- **Example / boilerplate code** (so coders don't get confused)
- **Core design decisions**
- **Functional flows** (how "load sidebar" works, which APIs are hit; how "open chat" works, which API is hit)
- **Development plan**

### Why Two Separate Documents (Spec vs Technical Design)?
- The spec is **non-technical** (WHY & WHAT); the technical design is the **HOW** (tech-stack-specific).
- Main reason to keep them separate: if you switch tech stacks later, you'd have to rewrite an all-in-one spec. But a **tech-agnostic spec** stays reusable — you can create **many technical design plans** (for different tech stacks) around a **single spec**.

## Vibe Coding vs Spec-Driven Development (comparison)
| Aspect | Vibe Coding | Spec-Driven Development |
|---|---|---|
| Starting point | A rough idea (single prompt) | A written specification (everything defined) |
| Who decides requirements | The AI (based on your ask) | You, the programmer |
| Control | Low (AI has more) | High (you lead the process) |
| Speed | Very fast | Slower (time spent designing the spec first) |
| Code quality | Unpredictable | Consistent & traceable |
| Best for | Prototypes, exploration, side projects, non-technical people | Serious production systems |
| Failure mode | Too much code generated at once → you don't understand it | AI **over-engineering** (builds correctly but adds extra effort) |
| Debugging | Identify a mistake, ask AI to fix | Refer to the spec — does output match the spec? |
| Need to understand code? | No (you can build even without coding knowledge) | Yes — you must know the language well since you write the spec and lead the AI |

- **Agentic coding** is a paradigm where you use SDD to build software features — **plus** other things like **sub-agents**. SDD is one part of agentic coding.

## How This Playlist Will Use SDD (with Claude)
Normally teams write the spec, technical design, and tasks manually. In this workflow, **Claude creates them**:
1. **Spec document** — created by Claude (a **custom slash command** will be built in a future video: invoke it, say what feature you want, spec is auto-generated). **You still review it.**
2. **Technical design plan** — created via Claude's **Plan Mode** (next video). Plan mode studies a spec sheet and produces a technical design plan. **You review it manually.**
3. **Tasks** — Claude auto-creates tasks from the technical design plan.
4. **Coding** — either a **single agent** builds the whole feature, or **multiple sub-agents** run tasks in parallel (both shown later).
5. **Validate** against the spec's requirements (tests can also be written; both approaches covered later).

### Tight Git/GitHub Integration (per feature)
Before starting a feature:
1. **Pull** the most recent version of the code from git.
2. **Create a new feature branch** and switch to it.
3. Execute the whole SDD flow inside that branch (spec → review → technical design → review → code → validate).
4. **Commit** the changes (`git commit`).
5. **Push** the changes to git.
6. **Create a PR (pull request)** and **merge** it into the existing codebase.
7. **Delete** the feature branch.
8. **Switch back to the main branch.**

From the next video onward, every feature follows this exact flow — this is the "different, serious" way of AI coding (agentic coding), not vibe coding.


---

# Chapter 6 — Plan Mode & Database Setup (First SDD Feature)

First time actually building the project (an **Expense Tracker** app). Two goals: (1) set up the database using the SDD workflow, (2) learn **Plan Mode**.

## Today's Build Goal — Database Setup
The expense tracker lets a user create an account, log in, log expenses, and analyze them — all of which need a database. Three things done around the DB setup:
1. Create the required **tables**.
2. Insert some **dummy data**.
3. Add **functions** in `db.py` (used many times later in the playlist).
This is a **foundational feature** — other features can't be built without it.

## The Full SDD Flow Executed (with ~16 instructions)
### 1. Session + Git setup
- Start a new Claude Code session → `/rename` to **"Database Setup"**.
- `git pull origin main` (good practice; here it said "Already up to date").
- Create and switch to a new branch: **`feature/database-setup`** (convention: `feature/<feature-name>`).

### 2. Create the Spec Document (done manually this time)
- Two options: write the spec yourself, or auto-generate with AI. This project mostly uses the automated way, but this first proper spec was written **manually**.
- Key contents of the spec:
  - **Database schema** with two tables:
    - **users** table — columns: `id, name, email, password_hash, created_at`.
    - **expenses** table — stores per-user expense details: `amount, category, date, description, created_at` (plus each column's type and constraints).
  - **`db.py`** should implement three functions:
    - `get_db()` — opens a database connection and returns it.
    - `init_db()` — creates the tables (only if they don't already exist).
    - `seed_db()` — adds dummy data to the database.
  - **`app.py`** changes — import these three functions.
  - **Categories** — 8 fixed expense categories (food, transport, bills, …).
  - **Rules for implementation** — no ORMs (e.g., no SQLAlchemy); write the DB code by hand; **use parameterized queries only**; specific date format; error handling details.
  - **Acceptance criteria** — the feature is only correct if all these are met.
- **Save the spec** in the project: `.claude/specs/database-setup.md`. Going forward, all spec documents go in `.claude/specs/`.

### 3. Create a Technical Design Plan → via **Plan Mode**
This is normally done manually, but Plan Mode is built exactly for this.

## What is Plan Mode?
- A mode of operation where, before writing code, Claude Code starts **multiple agents** that **explore, read, and understand** your codebase.
- **Important: during Plan Mode, no write operations are performed** — Claude only **reads**. The goal is to produce an **elaborate plan** for the given task.
- Only after the plan is made do you give permission to **execute/implement** it.

### How to Enter Plan Mode
- Press **Shift+Tab twice** (shows "Plan Mode on"), OR
- Type **`/plan`** and hit enter.

### Generating the Implementation Plan
- Prompt used: go to the spec document, look at existing files, generate an implementation plan, and save it to `.claude/plans/` (though in the demo the plan wasn't actually saved).
- Claude read the whole spec, read relevant files, explored the codebase, and produced a plan (which `db.py` functions to create, the table-setup queries, the `seed_db` function, the `app.py` changes, categories, and how verification would happen).
- (Note: Claude Code was temporarily down mid-demo → "API error" — retried after it was back up.)

### 4. Implement the Plan
- Choose **manually approve edits** so you see each change.
- Changes shown as red (removed) / green (added) lines in `db.py` and `app.py`; approve each.
- Claude also ran its own tests (`pytest`): "All checks passed" — one demo user created, 8 expenses across all categories.

### 5. Validate against the spec's acceptance criteria
- DB file created ✓
- Both tables present with correct schema ✓ (users → demo user with email/password_hash/created_at; expenses → 8 expenses)
- No duplicate seed data on repeated runs ✓
- App starts without errors ✓ (`python app.py` → pages working)
- Foreign key enforcement, parameterized SQL ✓ (verified `db.py` has the three functions and parameterized queries)
- "Iterate if required" — not needed here; all three goals achieved.

### 6. Commit → Push → PR → Merge → Cleanup
- `git add` the changed files.
- `git commit -m "Create Database Setup"`.
- `git push` → a new branch appears on GitHub.
- On GitHub: **Compare & pull request** → create PR (merging feature branch into main) → **Confirm merge** → optionally **delete branch**.
- Back in terminal: `git checkout main`.
- **Before deleting the local feature branch**, `git pull origin main` first (so main gets the merged changes locally).
- Delete the `feature/database-setup` branch.
- Result: database is set up; ready for upcoming features.

## Plan Mode Best Practices (settings to tweak)
### 1. Model Selection
- Three coding models: Haiku, Sonnet, Opus (Opus = most powerful).
- For **complex planning** (reasoning across multiple files, mass refactoring, deep architectural decisions), switch to **Opus** for planning — the plan will be very solid (but Opus burns more tokens). Common workflow: **Opus for planning, Sonnet/Haiku for coding.**
- Switch with `/model`. (This project has no feature needing Opus for planning.)

### 2. Extended Thinking
- Normally Claude replies in **standard mode** — starts printing the answer token-by-token as soon as it sees the question. Fine for simple questions, can fail for complex ones.
- **Analogy:** in an interview, for a hard puzzle you'd first reason on a whiteboard, then answer — rather than blurting out and compounding mistakes.
- **Extended thinking** = Claude first pauses and reasons on a **scratchpad** (a reasoning phase before the response phase), then prepares and gives the answer.
- **Recommended: turn it ON before using Plan Mode** — plan quality usually improves a lot.
- Toggle via `/config` → **Thinking mode** (generally `true`; can set `false` to save extra tokens). Space to toggle, Enter to confirm.

### 3. Effort Level (closely related to extended thinking)
- Extended thinking writes internal thoughts to a scratchpad, which **burns tokens** — a costly process. You can't let it think unlimited.
- **Effort level = the token budget** for how much/how long the model can reason on the scratchpad.
  - **Low** → few tokens (e.g., 500–1000) for reasoning.
  - **High** → many tokens (e.g., 5,000–10,000).
- Four/five levels: **low, medium, high, max** (+ **auto**). **Max** comes especially with Opus (effectively unlimited thinking) — rarely needed unless very complex planning.
- Toggle with **`/effort`** → pick low / medium / high / max / auto.
- Presenter usually uses **auto**; suggests **medium to high** — but be careful, tokens burn here.

### 4. Ultra Plan (superior alternative to Plan Mode)
- A very **new feature** (released recently). Use it when a regular Plan Mode plan isn't satisfactory for a **very complex** feature.
- **How it works:** trigger `/ultra-plan` (or use the word "ultra plan" in your prompt) → the plan is NOT built on your machine. Instead, Anthropic starts a **container in the cloud** running an **Opus 4.6** instance, and the plan develops there (in **Claude Code for Web**), with a **rich web editing experience** where you can edit the plan.
- Once satisfied, you can **teleport the plan back to your terminal** and implement it exactly like normal Plan Mode.
- Two options after the plan is ready: **Approve plan and start coding** (codes in Claude Code for Web) or **Approve plan and teleport back to terminal** (generally preferred). After teleport: implement here / start a new session / cancel.
- **Guideline:** use Ultra Plan only when unsatisfied with regular Plan Mode output for very complex features — otherwise it's overkill (more tokens, more costly).

## Next Video Preview
- How to create a **custom slash command**, and add **login & registration** features to the project.


---

# Chapter 7 — Custom Slash Commands & Login/Registration Feature

Two objectives: (1) learn to create **custom slash commands**, (2) add **login & registration** to the project.

## What Are Custom Slash Commands
- In simple words: **they are nothing but prompts** — saved prompts that get automatically invoked when you type the slash command name in Claude Code (just like built-in slash commands).
- **When to use:** for any **repeatable workflow** you use again and again in your project.
- **How to create:** write a **markdown file** for the command and save it in the `.claude` folder. Claude Code automatically identifies it as a slash command.

### Two Types (by scope)
- **Project-scoped** — markdown file saved in the **project's `.claude/commands/`** folder → usable only in the current project.
- **User-scoped** — markdown file saved in the **home directory's `.claude/commands/`** folder → usable across **all** your projects.

### Real Workflow Examples
- `/review` — run a code review on files you just created.
- `/commit` — generate a git commit message by looking at your code changes.
- `/test` — run your test suite on the code.
- `/security-scan` — test the whole codebase for vulnerabilities.
All are repeatable → good candidates for custom slash commands.

## Creating the First Custom Command (`/seed-user`)
Purpose: add a new dummy user to the users table (needed for UI development when no real users exist yet — this is called **seeding**).

Steps:
1. Inside the project's `.claude/` folder, create a **`commands/`** folder.
2. Inside `commands/`, create the markdown file — **the file name becomes the command name** (`seed-user.md` → `/seed-user`).
3. Contents of the file:
   - **Description** — shown when you type `/` (e.g., "Create a single dummy user in the database").
   - **Allowed tools** — which tools the command can access (e.g., read files; run only bash commands starting with `python3` — not git).
   - **Detailed instructions (in English)** — read `db.py` to understand the users table; write and run a Python script (via bash) that generates a realistic random Indian user (name, email, password = "123" encoded, created_at timestamp); check if email already exists and regenerate until unique; insert the user using `get_db()`; print the user's details.
4. **Restart Claude Code** (exit and re-enter) so the new command appears.
5. Run `/seed-user` → it reads the file, asks to proceed → inserts a new user (e.g., "Rohan Kulkarni" with encrypted password).

## Creating the Second Custom Command (`/seed-expense`)
Purpose: populate the expenses table. This one is **flexible — it takes inputs**:
1. **user id** — which user the expenses belong to.
2. **count** — how many expenses to add.
3. **months** — spread the expenses randomly over the last N months.

Steps:
- Create `.claude/commands/seed-expense.md`.
- Contents include: description; **argument hints** (so typing `/seed-expense ` then space shows the three inputs); allowed tools; detailed instructions.
- Key mechanism: **`$ARGUMENTS`** — a variable. The three inputs the programmer provides after `/seed-expense` get stored in `$ARGUMENTS`, then extracted (user_id int, count int, months int). If any is missing, show the correct input format.
- Instructions also cover: verify the user exists; generate & insert expenses spread across the months; per-category amount ranges (e.g., food ₹50–800, health ₹100–2000 — editable); distribute categories roughly proportionally.
- Restart → `/seed-expense 2 5 3` → inserts 5 expenses for user id 2 across the last 3 months.

## Automating Spec Creation (`/create-spec`)
Instead of writing spec documents manually (as in the DB video), automate it with a custom command.

**`create-spec.md` contents:**
- **Description:** "Create a spec file for the next Spendly feature" (Spendly = the app name).
- **Arguments:** a **step number** (feature #) and a **feature name**.
- **Allowed tools.**
- **Persona:** "You are a senior developer planning a new feature for the Spendly expense tracker. Always follow the rules in CLAUDE.md." User input via `$ARGUMENTS`.
- **Step-by-step:**
  1. Parse arguments → extract step number, feature title, feature slug (slug = URL-like). If not inferable, ask the user to clarify.
  2. **Research** the whole codebase: read CLAUDE.md, app.py, the database file, and all existing files in the specs folder (so it knows what's already built).
  3. Write the spec file with an **exact structure**: feature title, overview (one paragraph: what the feature does and why it exists at this stage of the roadmap), dependencies, routes to implement, DB changes needed?, UI templates to create/modify, other files modified, new files created, dependencies, rules of implementation, acceptance criteria.
  4. Save to `.claude/specs/<step-number>-<feature-name>.md` and tell the user.

## Building the Registration Feature (automated flow)
1. `git status` → commit any pending files: `git add`, `git commit -m "Create create-spec slash command"`.
2. Create feature branch: `git checkout -b feature/registration`.
3. **Generate spec automatically:** `/create-spec 2 registration` → creates `.claude/specs/02-registration.md`.
4. **Review the spec carefully** (ideally paste into ChatGPT/Claude for a proper review). Spec covered: upgrade the stub `GET /register` route into a full form that accepts POST, validates input, hashes the password, inserts a new row into the user table; on success show a success message and redirect to login; routes; no DB schema changes; a new db helper in `db.py`; changes to `register.html`; rules (no ORM/SQLAlchemy, parameterized queries, hash passwords, app secret key, server-side validation — all fields non-empty, password matches confirm password).
5. **Plan Mode:** enter plan mode → prompt: "Read `.claude/specs/02-registration.md` and create a detailed implementation plan. Don't write any code." → review plan.
6. **Implement** (manually approve edits): changes to `db.py`, `app.py` (added secret key, implemented the route), and `register.html`.
7. **Validate against acceptance criteria** (`python app.py`):
   - `/register` shows the registration form ✓
   - Valid submission creates a new user and redirects to login ✓ (verified in DB)
   - Mismatched passwords → re-render form with "Passwords do not match" ✓
   - Already-registered email → "Email already registered" ✓
   - Empty fields → error ✓
   - Password stored as a hash ✓
   - No duplicate user on repeated submission with same email ✓
8. **Commit → Push → PR → Merge → Cleanup:** `git add`, `git commit -m "Add registration feature"`, `git push origin feature/registration`, create PR on GitHub, merge, delete branch, `git checkout main`, `git pull origin main`, `git branch -d feature/registration`.

## Improving `/create-spec` to Also Handle Git (for Login feature)
To avoid manually doing the branch steps each time, the branch creation/switch was **automated inside `create-spec.md`**:
- **Step 1:** check if the current working directory is clean (`git status`); if there are uncommitted/unstaged changes, **stop** and tell the user to commit first.
- Parse arguments.
- Check if the branch name already exists; if so, append a number to make it unique.
- Switch to main and `git pull` the most recent content.
- Create a new branch and switch to it (branch name derived from the feature).
- Then continue: research → write spec → save → notify user.

## Building the Login/Logout Feature
1. Commit the modified `create-spec.md` first (the command enforces a clean working tree).
2. `/create-spec 3 login-and-logout` → auto-creates the branch, switches to it, and generates `.claude/specs/03-login-and-logout.md`. (Verify with `git branch` — the branch was auto-created.)
3. **Review spec:** converts the login stub into a functional POST handler; implements logout (clears session, redirects to landing page); redirects to dashboard/suitable page after login; depends on DB setup + registration; routes: `GET /login` (form), `POST /login`, `GET /logout`; no DB changes; modify login template.
4. **Plan Mode:** "Read `.claude/specs/03-login-and-logout.md` and generate an implementation plan. Don't write any code." → review.
5. **Implement** (manual approve): login route, logout route, frontend + navbar changes, `base.html`.
6. **Validate:** login form shows at `/login` ✓; valid credentials set user id and redirect ✓; wrong password → "Invalid email or password" ✓; unregistered email → same generic flash message; `/logout` clears session and redirects ✓.
   - **Bug found via iteration:** while logged in, `/login` and `/register` were still accessible (shouldn't be). Fixed by prompting: "I am able to access /login and /register even when I am logged in. This should not happen." → after fix, logged-in users get redirected away from `/login` and `/register`. (Lesson: had the full spec been read carefully, this would've been included — shows **iterative improvement**.)
7. **Commit → Push → PR → Merge → Cleanup** as before.

## Result & Next Video
- Website now has: database setup, working registration (new users), and login. Much of the flow (spec creation + branching) is now automated.
- **Next video:** develop the logged-in user's **profile page** (none exists yet).


---

# Chapter 8 — Skills (and the Profile Page)

Skills is a very important topic — it applies not just to Claude Code but across the whole Claude ecosystem (regular Claude chatbot, Claude for Work). Goal: understand Skills theoretically, apply them practically, and design the project's **profile page** using a Skill.

## The Core Problem: General AI vs Specialized Tasks
- Claude / ChatGPT / Gemini are **general-purpose language models** — great at reasoning, writing, coding across domains.
- **The gap:** there's a gap between general capability and reliable, high-quality output for a **specialized task type**.
- **Example (PPT generation):** Claude knows what a PPT is, how to structure slides, which Python library to use — yet it can't make a *great* PPT for you because it doesn't know **your company's design guidelines**: layout, fonts, when to use graphs/charts/tables. It has general PPT capability but not your specialized skill.
- Same problem for: your company's website front-end design, specialized data analysis, your writing style, your code-review style.
- **Idea:** LLMs are good at general reasoning but don't do well on specialized tasks.

## Why Prompts Fail for Repeated Workflows
You might think "I'll just write a detailed prompt." But that creates more problems:
1. **Repeated retyping** — for repeatable tasks you must retype the same detailed prompt → error-prone.
2. **System prompt eats context** — putting a 20K-token instruction in the system prompt permanently occupies the context window, whether you use it or not.
3. **Can't bundle resources** — you can't attach reference files/images to a prompt.
4. **Can't share, version, or improve prompts** — prompts are very personal; most companies don't use prompt-versioning tools.
5. **Prompts don't compose** — a task needing several specialized sub-tasks (read PDF → extract tables → build PPT) stuffed into one big prompt can confuse the LLM, degrading performance.

## What Are Skills
- **Definition:** Skills are **reusable, file-based resources** that provide Claude with **domain-specific expertise** (workflows, context, best practices) that transform general-purpose agents into **specialists**.
- Simply: a Skill is a **folder inside your project** containing files that teach the agent how to carry out a specialized task.
- **Best part:** Skills **load on demand** — only when needed. (If you're just chatting and haven't mentioned PPTs, the PPT skill isn't loaded. As soon as you say "create a PPT," Claude jumps into the skill folder, loads the relevant files, and uses them.)
- Unlike prompts (good for one-off instructions), Skills act like **just-in-time knowledge**.

## Skills Structure
```
project/
└── .claude/
    └── skills/
        └── <skill-name>/
            ├── SKILL.md        (required — without it the skill won't work)
            ├── scripts/        (optional resources, e.g., Python scripts)
            └── templates/      (optional, e.g., design guidelines, reference images)
```
- **SKILL.md** holds the detailed instructions for the specialized task.
- Additional resource folders (scripts, templates) can hold supporting files.

### Inside SKILL.md
1. **YAML front matter** (at top) — two things:
   - **name** — how Claude identifies the skill.
   - **description** — very important; Claude reads this to decide **when** to load/trigger the skill (e.g., "whenever the user asks to create a PowerPoint presentation, load this skill"). It's the trigger.
2. **Markdown body** — all the detailed instructions: coding patterns, possible mistakes, validation steps. If the skill needs other files, their **path/link** is provided here (e.g., "to plot this kind of graph, use the following code" → link into the scripts folder).

## How Skills Load — Progressive Disclosure
- **Core idea:** *don't present information until the moment it's needed* (context window is limited, so preserve it).
- **Three levels:**
  1. **Level 1** — the **YAML front matter (name + description)** of every skill is **always loaded** (it's tiny). So Claude always knows which skills exist and when to trigger them.
  2. **Level 2** — when the user's message matches a skill, the **SKILL.md body** is loaded on demand and read line-by-line.
  3. **Level 3** — if the body references a resource/script, those **reference resources** are fetched only then.
- This avoids the "system prompt eats context" problem — only descriptions stay loaded permanently.

## Types of Skills (by scope)
- **Personal skills** — saved in the **home directory's `.claude/`** folder → available across **all** projects (your personal coding/writing/design style). (So a skill folder can live in the home directory too, not only inside a project.)
- **Project skills** — saved in the **current project's `.claude/`** folder → apply only to the current project; can be shared with teammates via git, versioned, improved.

## Ways to Create Skills
1. **Manually** — create the folder + SKILL.md + write detailed instructions. Simple, but **not recommended for beginners** (you lack experience/format knowledge).
2. **Using Claude (recommended)** — in the Claude chatbot, click the **+** icon → **Skills** → **Skill Creator** (itself a skill that helps create other skills).
3. **Community sources** — install others' skills (Google "Claude skills marketplace"; e.g., skills.md-type sites). **Be careful** — unread community skills can have **security flaws** (there's news of API keys being leaked). Safer: use **Anthropic's own public skills repository**.

## Skill Creation Workflow (steps)
1. **Identify the need** — only create skills for specialized tasks you do repeatedly.
2. **Create** — make a directory, add SKILL.md, write detailed instructions + supporting files.
3. **Test** — testing/evaluating/benchmarking skills is a big topic (not covered here).
4. **Iterate & improve** — no skill is perfect first try; after ~4–5 iterations you get a usable skill.

## How Skills Solve Every Prompt Limitation
| Prompt problem | Skills solution |
|---|---|
| Retype every time | File saved once, auto-loads when needed |
| Context window burned | Only description loaded; body loads on demand |
| Hard to bundle resources | Folders can hold any files/code |
| Can't share/version/improve | Skills go to git; teammates collaborate |
| Prompts don't compose | Skills can link to each other like links (composability) |

## Practical: Building the Profile Page with a Skill
Currently after login the user just lands on the home page (no profile/dashboard). Plan: design a **profile page** using a **front-end web design skill**. Flow: first build the page **without** a skill, then build it **with** a skill, to show the improvement.

### First build (without skill)
1. New session named "Profile Page Design".
2. `/create-spec profile-page-design` → auto-creates branch + spec. (The generated spec lacked design detail, so a pre-written spec was pasted instead.)
   - Profile page shows: **user info cards**, **summary stats** (total spent in a period, number of transactions, top category), a **transaction history table**, and a **category breakdown**. Important: the UI uses **static hard-coded/dummy data** (not from the DB).
3. **Plan Mode:** "Read CLAUDE.md and `specs/04-profile-page.md` and come up with the detailed implementation plan."
4. **Implement** → created `profile.html`, CSS changes. (Claude auto-generated tests, which wasn't wanted — should've said "don't generate any tests.")
5. Verified: `/profile` shows the UI (looked good already, all hard-coded/toy data).

### Creating the front-end design skill (via Claude Skill Creator)
- Claude chatbot → Skills → **Skill Creator** → "create a new skill from scratch" → described in own words the three things: **what the skill does, when it triggers, what success looks like**.
- Claude read the GitHub repo to understand the code structure, then asked clarifying questions (Flask Jinja2 + vanilla CSS? which icon library? test before finalizing?).
- Claude drafted the skill → copy the SKILL.md content into `project/.claude/skills/front-end-design/SKILL.md`.

### Second build (with skill) — using `/rewind`
- To redo the UI, erase previous progress with **`/rewind`** — go back in code and conversation. On enter it asks how far back; options: **restore conversation only** OR **code + conversation**. Chose to go back to where the spec was created (the earlier spec got removed).
- Re-created the spec, replaced it with the pre-written one (this time removing the "test" line so no unnecessary tests are generated).
- **Exit and restart Claude** (`claude -r`) — a new skill isn't usable until Claude restarts (same as custom commands).
- **Plan Mode:** during planning Claude automatically loaded the skill ("Now let me invoke the front-end design skill… Successfully loaded skill").
- Implemented → `base.html` and `profile.html` changes.
- **Result:** a **mild improvement** (e.g., it decided the category breakdown could sit with the transaction table instead of a big separate card). Fonts looked similar; icons somehow weren't used. Small app → limited chance to differentiate; the difference is bigger on **complex tasks**.

### Extra tweaks
- Prompt: "make sure the logged-in user automatically gets redirected to the profile page" → login now redirects to `/profile`; logged-out access to `/profile` redirects to login.
- Used **`/rewind` snapshot** concept + prompt: "don't want to show this welcome message after login" → removed the welcome message.

### Commit → Push → PR → Merge → Cleanup
- `git add`, `git commit -m "Add profile page UI"`, `git push origin feature/profile-page-design`, create PR on GitHub, merge.
- Two branches existed (the process was done twice → two spec generations → two branches). Cleaned up: `git checkout main`, `git pull origin main`, `git branch -d feature/profile-page` and the extra branch. `/exit`.

## Important News: Commands and Skills Have Merged
- Anthropic has **merged Commands and Skills**. Going forward **commands won't exist separately — only skills**.
- To make a "command," you build it exactly like a skill: a folder with `SKILL.md` + resources.
- Even though commands are triggered by the user and skills auto-invoke, their **structure is the same** (markdown file + YAML front matter + description + details), so Anthropic unified them. **Both can be invoked via `/`** (e.g., the newly created "Spendly UI Designer" skill appears when you type `/`).
- **Anthropic recommends:** don't create commands — only skills.
- To make a skill behave like a command (only you invoke it; Claude never auto-invokes), add this flag to the YAML front matter: **`disable-model-invocation: true`**.
- Summary: no separate `commands/` folder going forward; commands live inside the `skills/` folder with that flag in their SKILL.md. (This project keeps a commands folder, but future projects won't.)


---

# Chapter 9 — Subagents (Theory)

One of the most-requested Claude Code topics. Covered across two videos: **this one is theory** (why subagents exist, what they are, types, use cases + a built-in demo); the **next video** builds custom subagents hands-on.

## Why Subagents Are Needed (First-Principles)

### How LLMs work
- You talk to an LLM through an **API call**: your question → API → LLM → response → API → you.
- **LLMs are stateless — they have no memory of their own.** A conversation today isn't remembered later; even within a chat the model doesn't inherently remember the previous turn.
- Example: ask "What is the capital of France?" → "Paris". Then ask "What about Germany?" → the LLM has no memory of the earlier turn, so it replies "In what context?" — it doesn't know what you're asking about Germany.

### The chat workaround (resend full history)
- App developers solved statelessness by **resending the entire conversation history** with every new question, so the LLM has full context each turn.
- So "What about Germany?" is sent along with the earlier Q ("capital of France?") and A ("Paris") → now the LLM infers you want Germany's capital → "Berlin".
- Not elegant, but it's **how all chat apps built on LLMs work**.

### Why this fails badly for coding agents (token explosion)
- Imagine a codebase of ~15–20 files ≈ **30,000 tokens**.
- **Turn 1:** "Analyze my codebase and build an auth system." The agent must understand your DB, backend, API contracts → loads the **whole 30K-token codebase** into context and sends it with your message. Returns a plan (~2K).
- **Turn 2:** "Now implement the JWT middleware based on the plan." Since the full history is resent, the **entire 30K codebase is sent again** + previous conversation + current message ≈ **32K tokens this turn**. Lots of code is generated.
- **Turn 3:** "Add rate limiting and refresh token rotation." Resend 30K codebase + ~5K conversation + generated code + current message ≈ **39K tokens this turn**.
- **The problem:** the codebase was only needed **once** (turn 1) to give context — but it's re-sent **every** turn because that's how sessions work.
- By **turn 8** you're sending ~**76,000 tokens in a single turn**, and across 8 turns you may have already spent **more than $1**. That's a lot.

### Two core problems this creates
1. **Context window overflow** — context fills very fast because full history is resent each turn.
2. **Lost-in-the-middle effect** — when the window is very full, LLMs over-focus on the earliest and latest tokens and **forget the middle**. A well-studied effect that degrades answer quality.

**These two problems are why subagents were born.**

## What Are Subagents (Core Concept)
- **Definition:** *"Subagents are specialized AI assistants that run in their own isolated context windows, doing heavy lifting in a separate space and handing back only what matters."*
- When you chat with Claude Code, you talk to the **main agent**. The main agent can — on its own or when you ask — **create a brand-new subagent**.
- Each subagent gets its **own fresh, isolated context window** to do its specialized work, then **returns only the result** to the main agent. Once done, the subagent's context window is **deleted/destroyed**.
- A subagent has the **same capabilities** as the main agent: same model, same system prompt, and its own context window.

## How Subagents Work (Flow & Example)
Prompt to main agent: **"Add auth to my Express app."**
1. Claude decides: *"I'll analyze your codebase first — let me spawn a subagent for that,"* rather than polluting the main context with the whole codebase.
2. Creates a **new isolated subagent** and gives it two things: (a) load the full codebase, (b) *"Analyze this codebase and come up with an implementation plan."*
3. The subagent uses the full **30K tokens** internally and produces a **small implementation plan** (~500 tokens), e.g.:
   > Found 20 files. Stack: Express + Prisma. 12 routes. No existing auth. Sessions & middleware. Redis already configured for caching.
4. It **returns just that summary** to the main agent, then its context is **destroyed**.
5. The main agent carries the conversation forward using only that small plan.

### The payoff
- **Without a planning subagent:** the full 30K codebase is re-sent every turn → context fills fast, cost per turn is high.
- **With a planning subagent:** the main agent's context holds only a small plan (say ~2K). You save roughly **~28K tokens per subsequent turn**.
- Two benefits: **context fills much more slowly** (longer conversations possible) + **cheaper per turn**.
- **Mental model:** like a **function** in programming — you don't care about the internal code; you give an input, it processes, and hands back an output you use.

## Key Advantages of Subagents
1. **Context isolation** — a completely fresh context window for analysis-heavy tasks (the primary reason subagents are used).
2. **Specialization** — build purpose-built subagents (research, code-writing, security auditor). Each can have its **own system prompt, its own skills, and only the tools it needs** (deny access to the rest).
3. **Modularity** — split the whole software lifecycle into dedicated subagents: one to analyze code, one to implement plans, one to review code, one to run tests. This is the architecture experienced coders now follow with AI tools.
4. **Parallelism** — because each subagent has its own context, **independent tasks can run in parallel**.
   - Example: an EDA (Exploratory Data Analysis) subagent + 3 datasets → spawn **3 instances**, one per dataset, run simultaneously (they don't depend on each other).
   - Without subagents, a single main agent works **step-by-step only** — no true parallel execution.

## Real-World Use Cases

| # | Use Case | Why a Subagent |
|---|---|---|
| 1 | **Codebase exploration** | Exploring without a subagent eats the context window and costs a lot per turn. Claude Code is smart enough to **auto-spawn** an explore subagent whenever you ask it to explore. |
| 2 | **Code review** | The agent that *wrote* the code has an **inherent bias** (knows its own tradeoffs/assumptions) and reviews poorly. A **separate** reviewer agent gives better, unbiased reviews. |
| 3 | **Testing** | Same logic — the author agent doesn't generate/run great tests for its own code. Use a **separate** testing agent. |
| 4 | **Multi-stage pipelines** | When tasks are chained (output of one = input of the next), e.g., API contract → implementation → tests, each stage as its own agent hands off cleanly. |
| 5 | **Parallel independent tasks** | e.g., develop Auth, Payment, and User services simultaneously — different files, independent → develop in parallel. |
| 6 | **Security audit** | The author agent can't audit its own code well (bias). Create a **separate security-audit** subagent. |

*(Many more use cases exist daily — these are the primary ones.)*

## Types of Subagents
Claude Code offers **two types**: **built-in** and **custom**.

### 1. Built-in Subagents (pre-made, no setup)
| Subagent | Triggered When | Job |
|---|---|---|
| **Explore** | You ask Claude to explore a codebase (or part of it) | Reads, understands, analyzes the code → returns a **summary**. The most famous built-in. |
| **Plan** | You enter **plan mode** to generate an implementation plan (usually from a spec doc) | Does the **heavy lifting** of building the implementation plan. |
| **General-purpose** | Claude needs a subagent for any **read or write** task | Handles generic read/write work. |

### 2. Custom Subagents (you build them)
Built for your app's specific needs. Two scopes (same distinction as **skills**):
- **User-level** — live in the `~/.claude` (`.claude` home) directory → available across **all your projects** on the machine.
- **Project-level** — live inside the project's `.claude` folder → usable **only in the current project**.

**What you configure per custom subagent:**
- **Tools** — which tools it can access (explorer → read tools; coder → write tools; researcher → web search). Deny the rest.
- **System prompt** — defines the subagent's job.
- **Model** — e.g., Opus for one subagent, Sonnet for another.
- **Permissions**, **Hooks** (hooks not yet covered in the playlist), and **Skills** access.

**Example custom subagents:**
- **Security Reviewer** — tools: Read, Grep, Glob; model: **Opus**; prompt: *"Review for injection, authorization bypass, and data exposure vulnerabilities."*
- **Research Agent** — tools: Read, Grep, WebSearch, WebFetch; model: **Sonnet** (exploration needs less deep reasoning); with its own system prompt.

*(Full hands-on build is in the next video.)*

## How Subagents Are Triggered (both types)
1. **Implicitly** — Claude reads your prompt (and, for custom ones, the agent's file) and **decides on its own** to delegate. *"Claude recognizes the task needs a subagent and delegates on its own."* No user role.
2. **Explicitly** — you tell Claude you want a specific subagent for the task, and it spawns it.

## Built-in Subagents Demo (Expense-Tracker: Backend Connection)
Goal: connect the existing profile-page **UI to the backend** so the logged-in user sees **real DB data** instead of dummy data — while showing all three built-in subagents in action.

**Observability tool:** you normally can't see which subagent fires. The presenter used a GitHub library, **agents-observe** — a real-time dashboard (built using **hooks**) showing which subagent triggered and what it's doing. Run with `observe start` → open the URL → dashboard.

### Steps demonstrated
1. **New session** → `/rename Backend Connection`. Start `observe start` dashboard (shows only the main agent initially).
2. **Explore subagent demo** — prompt: *"Explore the codebase and tell me what this is all about. Also tell me what features have been already developed and which features are yet to be developed."*
   - `/context` before: **147.5K** free.
   - On the dashboard a new **"Claude Explore"** subagent spawns, uses the **Read tool** on many files, returns a summary, then shows **"subagent stopped"** — control returns to main agent.
   - `/context` after: still **~142K** free → the whole codebase never entered the main context. (If it had, this number would drop hard.)
3. **Plan subagent demo** — first create a spec: `create spec` → 5th spec, *"backend routes for profile page."*
   - Notice **no new subagent** spawns for the spec — the **main agent** handles it (only a few files: `app.py`, `database.db`, etc.).
   - Pasted a pre-written spec over the generated one, then ran a **Plan Mode** prompt to build an implementation plan for the backend routes.
   - **Twist — parallel demo baked into the prompt:** *"While implementing the plan into code, split the work across three parallel subagents"* — subagent 1 → summary stats, subagent 2 → one table, subagent 3 → another table (three separate profile-page features).
   - ⚠️ **Not a good idea in general:** parallel subagents should work on **separate files**. Here all three edit the **same file** — done only as a demo.
   - **On the dashboard:**
     - First, **two Explore subagents** spawn (planning needs exploration first) — one for template structure, one for test structure, run in parallel → both stop.
     - Then a **Plan subagent** ("Claude Plan") spawns → builds the plan → stops. The **main agent** then writes the final plan file.
     - Run the plan → **three General-purpose subagents** spawn and execute **in parallel** (subagent 1/2/3) → each stops.
     - Main agent: *"All three look correct, now integrating"* — it has to do **extra work** stitching one file together (the downside of same-file parallelism).
4. **Verify** — refresh site: all zeros (current user "Nitesh" has no expenses). Use the earlier custom command **`/seed-expense`**:
   - Checked DB → user id = **3**. `/seed-expense user id 3` → 3 expenses over last 6 months → refresh → 3 expenses + total show.
   - Ran again for 2 more → refresh → **5 expenses** displayed. Feature complete.

### Wrap-up (git flow)
- `git add` → `git commit -m "Add dynamic routes to profile page"` → check branch name → `git push origin <branch>`.
- Branch name got split by a line break; asked Claude *"can you resolve the issue and push?"* → fixed and pushed.
- `gh` → create PR → merge → delete branch. Then `git checkout main`, `git pull origin main`, `git branch -d feature/<name>`. Done.

## Key Takeaways
- Subagents exist to fix **context overflow**, **runaway cost**, and the **lost-in-the-middle** effect caused by resending full history every turn.
- A subagent = **isolated context window + specialization**; it does heavy work in private and **returns only a small result** — like a function call.
- Four big wins: **context isolation, specialization, modularity, parallelism.**
- Prefer **separate agents** for **writing vs reviewing vs testing vs security** (self-review is biased).
- Two types: **built-in** (Explore, Plan, General-purpose) and **custom** (user-level vs project-level, with configurable tools/system prompt/model/permissions/hooks/skills).
- Triggered **implicitly** (Claude decides) or **explicitly** (you ask).
- For **parallel** subagents, keep them on **separate files** — same-file parallelism forces the main agent to do messy integration.

---
**One-line takeaway:** Subagents run specialized work in their own isolated context windows and hand back only the result — saving tokens, money, and quality, and enabling parallel, modular AI coding workflows.


---

# Chapter 10 — Custom Subagents (Hands-On)

The practical follow-up to Chapter 9. **Prerequisite:** watch/read the subagents theory chapter first — this chapter assumes you know why subagents exist, what they are, the built-in types (Explore, Plan, General-purpose), and the top use cases.

## Recap (from Chapter 9)
- **Why subagents:** isolated context + specialization.
- **What they are:** agents with their own context window, own prompt, own tools, started for a specific task.
- **Built-in types:** Explore, Plan, General-purpose.
- **This chapter:** how to build and trigger **your own (custom) subagents**.

## Why Custom Subagents (vs Built-in)
- Built-in subagents already handle generic tasks — so why build your own?
- **Example — security audit:** you want to check your codebase for SQL injection, exposed API keys, etc. This needs the whole codebase loaded **once** → a perfect subagent use case, and a **built-in** agent can do a generic audit.
- **The catch:** your company has a **guideline / checklist**, and the audit must follow *that specific* checklist. A built-in agent only has **generic** knowledge of "how to do a security audit" — it can't do a **specialized, checklist-driven audit tailored to your codebase.**
- **Solution:** a **custom subagent** where you control the tools, a specialized **system prompt** (your checklist/instructions), and skills access — a **tailor-made** agent that does the task exactly your way.
- Same reasoning applies to **testing** (specialized to your codebase), and any task needing specialization.

> **Rule of thumb:** Whenever there's a need for **specialization**, create your own subagent instead of using a built-in one.

## How to Create a Custom Subagent (Core Idea)
- A subagent is just a **markdown file** (everything in Claude Code is markdown).
- Structure:
  1. **YAML front matter** — key fields:
     - `name` — the subagent's name
     - `description` — short (or detailed) explanation of what it does — **this field drives auto-triggering**
     - `tools` — which tools it can access
     - `model` — which model backs it
     - plus optional: **skills**, **hooks**, **memory**, **effort level**, **color**
  2. **Main body** — detailed instructions on how the subagent should carry out its task (you can add a full system prompt here).
- **Where it lives:** inside `.claude/agents/` as a `.md` file.
- **Two scopes** (same distinction as skills):
  - **Project-level** — `.claude/` in the project root → only this project.
  - **Personal/User-level** — `.claude/` in your home directory → all your projects.
  - *(All subagents built in this chapter are **project-level**.)*
- **Two ways to create the file:** write the markdown yourself, **or** let Claude Code generate it for you (via `/agents`).

## How Custom Subagents Are Triggered
1. **Automatically** — Claude reads the `description` field, understands which agent fits which scenario, and triggers it on its own.
2. **Manually** — the programmer triggers it: either by asking directly, or via a **custom slash command** that invokes the subagent behind the scenes.
- In real workflows, people usually **prefer manual triggering** (fits their workflow), though auto works too.

## Plan of Action for This Video
Feature goal: **enhance the profile page** with a **date-range selection tool** (date filter) — a dropdown/pills where the user picks a range (e.g., "this month", "last 3 months", "last 6 months", or a custom start/end date), and **all** profile-page data (total amount, number of transactions, top category, recent transactions, by-category chart) updates to that range.

### Big workflow change: add Testing + Code Review stages
Going forward, every new feature passes through **two extra stages before pushing to git**:
1. **Testing** — extensively test the feature (like every software team does).
2. **Code review** — a **self-review** by the programmer before the PR stage.

New per-feature flow: **build feature → test → code review → then commit & push.**

### The subagent workflow being built
**Testing pipeline** — custom command `/test-feature` triggers **two subagents sequentially**:
| Subagent | Job |
|---|---|
| **Test Writer** | Writes pytest test cases for the current feature — **based on the spec, NOT the implementation code** (code may be buggy; the spec is the source of truth). |
| **Test Runner** | Runs the tests the Test Writer created, and produces a report. |
- **Why two separate agents?** Better strategy than one agent writing *and* running its own tests. The main agent then shows a **final summary** of the testing stage.

**Code review pipeline** — custom command `/code-review-feature` triggers **two subagents in parallel** (independent tasks):
| Subagent | Job |
|---|---|
| **Security Reviewer** | Analyzes the new code for security threats / hacking flaws. |
| **Code Quality Reviewer** | Checks whether the code follows good practices / is well written. |
- Both run in parallel → results are **merged into a unified report** by the main agent.

## Practical — Building the Subagents

### Creating the Test Writer (via Claude / `/agents`)
1. Go to the project directory → start `claude` → `/rename` session to **"Custom Subagents"**.
2. Run **`/agents`** — shows current agents / library, and a **Create New Agent** option.
3. Choose **Project** (level) → choose **"Generate the agent file using Claude"** (recommended for first time; the other option is **Manual configuration**).
4. Paste a **description**, e.g.:
   > *"Use this agent to write pytest test cases for Spendly features. Invoke after implementing any feature to generate tests based on feature specs, NOT the implementation."*
5. Claude asks which **tools** to grant → gave it **Read-only tools + Edit tools** (it needs to write test files).
6. Select **model** → **Sonnet**. Select a **color** → **Red**. **Memory** → **None** (not needed here).
7. Claude generates the full agent markdown (front matter + description + system prompt + detailed behavior) into **`.claude/agents/`**.

> ⚠️ **Always review the generated agent file.** Read it manually, or paste it into another AI with your project context and ask how correct it is. (The presenter didn't trust the raw generated file — he replaced it with a **pre-tested version** he'd validated earlier.)

**Test Writer behavior (from the file / infographic):** name = **Spendly Test Writer** — *"a specialized AI that writes pytest for Spendly based on what the feature should do, not what the code says."*
- Reads the **spec file** → understands what the feature should do → **creates** test cases (doesn't run them).
- Creates a **`test/` folder** in the project (if absent) with `test_<feature>.py` inside.
- Tests include: **happy-path** (correct input → correct output), **validation checks**, **HTTP semantics**, **edge cases**, **locked-out user can't access**, etc.
- **Won't Do:** tests are derived from the **spec**, never from generated code (code may be wrong; spec is always right).

### Creating the Test Runner (manually)
- Demoed the **Manual configuration** path in `/agents` (set name, system prompt, description, tools yourself) — but actually just went **directly into `.claude/agents/`** and created a new file **`spendly-test-runner.md`**, pasting a pre-made markdown.
- This agent's job: **run** the tests the previous agent wrote. **No write access** (only runs tests). Model set; **color = Green**.
- **Test Runner behavior:** verify tests exist for the current feature → run them → produce a report with **4 layers of analysis**: pass/fail summary, warning flags, deep type of failures, and recommendations. Final report has a **summary table, per-failure breakdown, warnings/flags, recommendations, and a final verdict.**

### Connecting the two via a slash command — `/test-feature`
- Created `.claude/commands/test-feature.md`.
- Front matter description: *"Writes and runs tests for a specific Spendly feature; pass the spec name as argument."* → the command needs the **spec name** as an argument.
- Body = two steps in plain English:
  1. **Write tests** → use the **Test Writer** agent.
  2. **Run tests** → use the **Test Runner** agent.
  3. Final output → a **summary table** in a specified format.

### Creating the Code Review subagents (pre-built)
- Added two more agent files: **`spendly-security-reviewer.md`** and a **code-quality reviewer**. (Presenter recommends reading these fully from the GitHub link.)
- Now the library has **4 custom agents**.
- Created command **`.claude/commands/code-review-feature.md`** that **orchestrates** both:
  - **Step 1 — Parallel review:** run **Spendly Security Reviewer** + **Spendly Quality Reviewer** together.
  - **Step 2:** build a **unified report** from both.
  - **Step 3:** if changes are needed based on the review, **ask for approval** before applying them.

## Running the Full Flow (Date Filter Feature)
1. **Commit the setup:** `git add`, `git commit -m "Add subagents"`.
2. **New session** `/rename` **"Date Filter"** (don't develop features in the same session where you built the agents). Verified `/test-feature` and `/code-review-feature` commands appear, and `/agents` shows all **4 agents**.
3. **Spec** — `/create-spec` (step 6): *"date filter for profile page."* → creates a **new branch** + spec doc → reviewed spec (looked correct).
4. **Plan** — prompt: *"Read the spec document and come up with an implementation plan."* → Claude spawned **two Explore subagents in parallel** to understand the code, then produced the plan → approved.
5. **Implement** → 4 files changed. Verified manually in the browser: date filter works — pills for *this month / last 3 months / last 6 months* + a **custom date range** (e.g., March 15 → today shows only those expenses). All stats update accordingly.
6. **Observability** — started the **`observe start`** dashboard to watch custom subagents in action.

### Testing stage — `/test-feature`
- Ran `/test-feature 06-date-filter-profile` (passing the spec path).
- Console: *"Starting the testing pipeline. Invoking the Test Writer agent."*
- Dashboard: **Spendly Test Writer** starts → studies changed files + spec → creates a new **`test/`** folder with the test file.
- Then: *"Test file written, now invoking the Test Runner."* → **Test Runner** starts → runs all tests → report:
  - **76 total tests → 73 passed, 3 failed.** Failing three flagged as *"test issues, not implementation bugs"* (SQL-injection assertion issues).
  - **Verdict:** *"The feature implementation is correct."*
- If real bugs appear here, you **prompt Claude to fix them and re-test** — an **iterative** process.

### Code review stage — `/code-review-feature`
- Ran `/code-review-feature <date-filter-spec-path>`.
- Console: *"Launching both reviewers in parallel now."*
- Dashboard: **Security Reviewer (yellow)** + **Quality Reviewer** run **together** → merged report.
- **Overall verdict:** a **change request** — *"the f-string SQL pattern must be fixed before committing; it violates CLAUDE.md's explicit rule and undermines the no-injection guarantee. Everything else is working correctly."*
- Prompt: *"Do you want me to implement the action plan now?"* → **Yes** → code changes applied → re-verified in browser (all ranges still work; security concern eliminated).

### Wrap-up (git flow)
- `git add` → `git commit -m "Add date filter to profile page"` → check branch → `git push origin feature/date-filter-profile`.
- On GitHub: create PR → *(this is where real team code review / PR review happens)* → merge → delete branch.
- `git checkout main` → `git branch -d <branch>` → `git pull origin main`. Verified the `test/` folder and `.claude/` agent files are all present. Done.

## Key Takeaways
- **Custom subagents = specialization.** Use them when a built-in agent's generic behavior isn't enough (checklist-driven audits, project-specific testing, etc.).
- A subagent is just **`.claude/agents/<name>.md`** = YAML front matter (`name`, `description`, `tools`, `model`, optional skills/hooks/memory/effort/color) + a body of detailed instructions.
- The **`description` field is what drives automatic triggering** — write it carefully.
- Create agents **via Claude (`/agents` → generate)** or **manually** — but **always review** the generated file before trusting it.
- **Separate agents for separate jobs:** Test Writer vs Test Runner; Security Reviewer vs Quality Reviewer (self-review is biased — same lesson as Chapter 9).
- Orchestrate agents with **custom slash commands** (`/test-feature` sequential; `/code-review-feature` parallel).
- New feature discipline going forward: **build → test → code review → commit/push.**
- **Tests should be written from the spec, not the implementation** — the spec is the source of truth.

---
**One-line takeaway:** Custom subagents are tailor-made markdown-defined agents (own tools, prompt, model) stored in `.claude/agents/` — orchestrate them with slash commands to add specialized testing and review stages to your spec-driven workflow.


---

# Chapter 11 — MCP (Model Context Protocol) in Claude Code

Three goals for this chapter: (1) how to use **MCP** in Claude Code, (2) a rundown of **top MCP servers** worth integrating, and (3) continue the expense-tracker project — add a new feature while using MCP to simplify the existing workflow.

## What is MCP
- **MCP = Model Context Protocol** — a **standardized way to connect any external tool/service to your LLM**.
- Created by **Anthropic (~1.5 years ago)**; now widely adopted — all major players use it to connect tools to any LLM.
- *"An open standard created by Anthropic that acts as a **universal connector** between Claude Code and external tools, services, and data sources."*

### The problem before MCP
- You had to write **custom, non-standardized code** to connect an LLM to each tool/service.
- If the service provider changed their API, **you had to rewrite** your connection code → constant breakage.
- MCP standardizes this → connecting any tool/service to your LLM became **super easy**.

*(This chapter stays high-level — the presenter has a separate 8-video playlist on MCP internals. Refer to that for deep technicals.)*

## Why MCP Matters in Claude Code
- **One-line benefit:** MCP lets you **add many more tools** to Claude Code and **enhance its functionality**.
- **Default Claude Code tools** are limited:
  - **Read** — read files from the filesystem
  - **Write** — create and write files
  - **Bash** — run command-line commands
- What Claude Code **can't** do by default: read your **GitHub** (repos, commits, issues, PRs), fetch a doc from **Google Drive**, pull **Jira** tickets, or grab **Slack** conversations.
- **With MCP servers** you give Claude these capabilities — pulling **additional context** about your workflow (GitHub activity, Jira tickets, Slack messages) directly into Claude Code without manually explaining it. This is the biggest benefit of MCP + Claude Code.

## Video Roadmap
Integrate **3 MCP servers** practically, then describe **7 more**:
1. **Database** server — talk to your DB directly, no Python code.
2. **Figma** server — pull a design and build UI components from it.
3. **GitHub** server — connect to your GitHub account and query/act on it.

## Checking / Managing MCP Servers
- **`/mcp`** — lists the MCP servers in your setup, showing connection status (e.g., `connected` / `failed`). Enter a server → **Reconnect**, or **View Tools** to see its tools and their descriptions.
- **Transport mechanisms:** two exist — **STDIO** (used for **local** setups) and **HTTP/SSE**. This chapter's local DB uses STDIO.
- **Remove a server:** `claude mcp remove <server-name>` → then start a new session and `/mcp` to confirm it's gone.

## 1. Database MCP Server

### Setup (and a gotcha)
- First tried **DBHub** (zero-dependency, token-efficient, multi-database) — but it **failed to start/connect**. Reason: the project uses a **local SQLite** database, which DBHub doesn't support in this setup (`/mcp` showed `failed` → Reconnect → `failed to reconnect to DBHub`).
- Switched to **mcp-database-server**, which works with multiple DB types **including SQLite**.
- **Install:** copy the provided command, specify the **path to your DB file** (e.g., `.../project/spendly.db`), open a **new terminal in the project**, and run it → `Added stdio MCP server`.
- **Exit and restart** `claude`, then `/mcp` → the SQLite server now shows **connected**.

### Querying the DB in natural language
Ask questions in plain English — no SQL, no Python:
- *"List all tables in Spendly database"* → Claude asks to use the SQLite `list_tables` tool → grant (**"Yes, and don't ask again for SQLite"**) → *"users and expenses."*
- *"Can you tell me the schema of expenses table"* → returns column details.
- *"Show total spending grouped by category"* → returns the grouped result.

### Why it's powerful
- Query DB doubts **on the go** — no need to learn the DB structure or hand-write SQL.
- This project has only 2 tables, but in real projects with **10–20 tables** (and complex relationships), it's extremely useful for understanding structure fast.
- Works with **SQLite, MySQL, PostgreSQL**, etc.

## 2. Figma MCP Server (Design → Code)

### The workflow it automates
- In companies, a **design team** builds UI/UX **wireframes** in Figma → exports → the **dev team** studies them and replicates them in code.
- With a Figma MCP server, this whole handoff is **automated**: keep a design ready, connect Figma ↔ Claude Code, give Claude the **design URL**, and Claude reads it and builds the page.

### Real use case: "Coming Soon" Analytics page
- Wanted a new **Analytics** module; since the dashboard isn't ready, show a nicely-designed **"Coming Soon"** page (designed in Figma).

### Setup & authentication
- Run the provided command in your **terminal** (not inside Claude Code) → installs a **plugin** into your Claude Code setup. *(Plugins covered a couple chapters later; here, think of it as a package that contains Figma's MCP server + skills.)* → `Successfully installed plugin`.
- Start a new session → run **`/plugin`** → **Installed** tab → **Figma plugin** → **authenticate** with your Figma account (opens a browser login). *(Presenter was already authenticated, so the prompt didn't reappear.)*
- The plugin *"includes the Figma MCP server and skills for common workflows"* — plugins can bundle **skills, agents, MCP servers, and hooks**.

### Generating UI code from a design
- In Figma, **right-click the design → Copy link** to get its URL.
- Prompt used:
  > *"Here is the Figma design for the coming-soon page: `<link>`. Please: read the Figma design and convert it to a Jinja2 HTML template. Add an Analytics menu item to the nav bar. Create a Flask route in `app.py` that renders the coming-soon page. Protect this route so only logged-in users can access it."*
- Claude used the Figma skill/tool to read the design (asks for Figma authorization if not already done), read existing CSS, and generated the page + route + nav item, then ran a self-test.
- **Result:** the live page matched the Figma design closely (fonts *quite* similar, layout nearly identical) → design replicated accurately in code.
- **Takeaway:** you can design in Figma (it also has a good chatbot interface to generate designs), export easily, and have Claude replicate it — automating the designer → developer handoff.

## 3. GitHub MCP Server

### Setup & authentication
1. Create a **Personal Access Token (PAT)** on GitHub: **github.com → Settings → Developer settings → Personal access tokens → Fine-grained tokens → Generate new token** (verify via email).
2. Set token **name**, **resource owner**, **expiration**, **repository access** (public / all / selected), and **permissions** (repo, PR, issues, etc.).
3. **Generate** → paste the token into the provided command where it says `your actual token here`.
4. In the **project terminal**, run the two commands one by one → second one prints `Added http MCP server`.
5. New terminal → `claude` → `/mcp` → GitHub shows **connected**.

> ⚠️ **Security:** never expose a real token on screen/in shared code. The presenter noted the token was visible and said it would be **deleted after publishing**. Treat PATs like passwords.

### Querying GitHub
- *"Which is my most starred repository?"* → *"100 Days of Machine Learning, ~2500 stars..."*
- *"Are there any issues on this repo? If there are, summarize them for me"* → *"30 open issues"* grouped by category (security, broken/missing content, deprecated/broken code, data leakage, correctness).
- *"Are there any open pull requests?"* → *"37 open PRs"* with a summary. You can also **close/merge PRs** from here.

### Automating the git flow (with a new feature)
- Built a new **Add Expense** feature (a form → submit → expense added to DB → shows on profile page), following the full spec-driven flow.
- Ran into an **uncommitted-changes error** when creating the spec (couldn't create a new branch) → fixed with `git add` + `git commit -m "Add coming soon page"`, then re-ran `/create-spec`.
- Flow: `/create-spec` (spec 07, "Add Expense") → review spec → **Plan Mode** ("Read this file and come up with an implementation plan") → approve → implement → verify in browser (added ₹1000 Entertainment "Movie Tickets" → showed up, persisted on refresh).
- **Testing:** `/test-feature 07-add-expense` → **46 tests created, all passed** → ready for review.
- **Code review:** `/code-review-feature 07-add-expense` → 2 parallel reviewers (security + quality) → suggestions → *"Yes, implement the action plan now"* → re-verified feature still works.
- **Automating commit → push → PR → merge → cleanup** via one prompt to GitHub MCP:
  > *"Commit all changes with an appropriate conventional commit message. Push to the current feature branch. Create a pull request into main with a proper title and description based on the spec. Merge it using squash merge. Switch to main, pull latest, and delete the feature branch locally."*
- ⚠️ **It partially failed:** *"the git token doesn't have PR write access."* Cause — **PR-create permission wasn't granted** when generating the PAT. Push worked; PR/merge had to be done **manually** on GitHub.
- **Lesson:** grant the token **explicit PR create + merge permissions**. Presenter will demo the fully-automated flow next video with a corrected token.

## 7 More Useful MCP Servers (described, not demoed)

| Server | What it is | Example prompt / benefit |
|---|---|---|
| **Context7** | Pulls **live, up-to-date documentation** for any library/framework into Claude's context while coding (LLMs have a knowledge cutoff; this bridges it). Needs an account + API key. Almost everyone uses it. | Always get the latest docs for the library you're using. |
| **Jira** | Atlassian project-management tool (developers' to-do list via tickets). | *"Read this ticket and implement the feature"* → pulls ticket details automatically. Or *"Find all open bug tickets in the Spendly project and fix the highest-priority one."* |
| **Notion** | All-in-one workspace (docs, knowledge, project planning, collaboration). | *"Read the PRD for the analytics module in Notion and implement the feature in my Flask app."* Or *"Read the API design doc in Notion and scaffold all the endpoints in `app.py`."* |
| **Slack** | Team communication platform with channels. | *"Push this fix to GitHub, open a PR, and post the PR link in the code-reviews channel with a summary."* Or *"Check the incidents channel for the latest production error, find the bug in the codebase, and fix it."* |
| **AWS** | World's largest cloud platform (200+ services). *(Presenter hasn't used it personally — uses GCP — but hears it's useful.)* | *"Deploy the latest build of Spendly to the EC2 instance and verify it's running."* Or *"Check CloudWatch logs for the last 2 hours and find what's causing the 500 errors."* |
| **Docker** | Containerizes your app with all dependencies into a portable unit. | *"Read my Spendly Flask app and generate an optimized Dockerfile."* Or *"My Docker image is 2GB — analyze the Dockerfile and reduce the image size."* |

*(That's 6 named here; Context7 is the strongly-recommended first one. Comment your own favorites.)*

## Two Housekeeping Tips
- **Remove a server:** `claude mcp remove <name>` → new session → `/mcp` to confirm.
- **Inspect a server's tools:** `/mcp` → pick a server → **View Tools** → select a tool for its full description.

## ⚠️ Most Important Lesson — Don't Over-Add MCP Servers
- Mistake to avoid: **blindly connecting every MCP server** you find.
- **Why it's bad:** every connected server's **tool descriptions auto-load into your context** at the **start of each session** → lots of unnecessary context → **degrades model performance**.
- **Best practice:** keep MCP usage **minimal**. Keep only the servers you genuinely and regularly use; **remove** the rest. Fewer description tokens in context → better model performance.

## Key Takeaways
- **MCP is a universal, standardized connector** between Claude Code and external tools/services/data — it removes brittle custom integration code.
- It extends Claude beyond its default **Read/Write/Bash** tools by pulling in **real workflow context** (DB, Figma, GitHub, Jira, Notion, Slack, AWS, Docker...).
- Manage servers with **`/mcp`** (status, reconnect, view tools) and **`claude mcp remove`**.
- Practical wins shown: **DB queries in plain English**, **Figma design → code**, and **GitHub query/automation** (with the reminder to grant PATs the right permissions).
- **Less is more:** too many MCP servers bloat context and hurt performance — keep only what you use.

---
**One-line takeaway:** MCP servers plug external tools and live context into Claude Code through one standard protocol, turning it into a far more powerful development partner — but keep the set minimal to protect context and performance.


---

# Chapter 12 — Hooks (and the Coding Harness)

This chapter builds deep intuition for hooks by first understanding what Claude Code *really* is (a **coding harness**), then covers the **why → what → how** of hooks, common use cases, internal mechanics, and a live demo — while continuing the expense-tracker project with an **Edit Expense** feature.

## What is Claude Code, Really? (The Coding Harness)

### User-facing definition
- Most would say: *"Claude Code is a terminal-based AI coding agent"* — almost true.
- Structurally: take any **Claude LLM** (Opus/Sonnet/Haiku) + a layer of **tools** (Read, Write, Bash) + **memory** capabilities (since the LLM is stateless but a coding agent must know project context).
- Its core feature is being **agentic** = **autonomous**: given a goal, it pursues it on its own, using whatever tools it needs, only asking you when necessary.

### System-design definition — a "harness"
- From the **builders'** perspective, Claude Code is a **coding harness**.
- **Harness (analogy):** *"a set of straps/equipment used to control and direct the power of something strong."* A horse has raw power + speed but is unpredictable; a harness turns that raw power into a **useful, stable horse-cart**.
- **Core idea:** *"Raw power becomes useful only when controlled through a structured interface."*

### Why an LLM needs a harness
- An LLM has raw power (intelligence, knowledge) but also problems: it's **unpredictable**, can **hallucinate**, is **stateless**, is **non-deterministic** (same prompt → possibly different output), is **disconnected from the real world** (only understands text), and **can't act safely on its own**.
- **Coding harness definition:** *"a piece of software used to convert a raw LLM into a reliable software-engineering system."*
- **What the harness does** (all the things Claude Code provides): reads the filesystem, displays terminal output, manages conversation history, tracks context-window usage, sends API requests to Anthropic, passes/executes the model's tool calls, implements the safety & permission module, manages memory (CLAUDE.md, subagent memory), provides slash commands, spawns parallel subagents, and offers extensibility (MCP, plugins).

### How harness + LLM work together — Mind/Brain vs Body
- Example: you type *"Explain the project."* → harness bundles your prompt + CLAUDE.md → API call → LLM (on Anthropic servers) → LLM replies *"I want to read app.py"* → harness executes it in the console, copies the content, sends it back via API → LLM now has the file content.
- **Analogy:** the **LLM = brain** (has the thought "I should read app.py"); the **coding harness = body/nervous system** (converts thought into action).

### Side note — Harness Engineering
- "Harness" is trending; a new field called **harness engineering** is emerging.
- Other harnesses: **OpenClaw** (a *personal-agent* harness for long-running tasks, connects to personal life e.g. WhatsApp — went viral), **Hermes Agent** (personal-agent harness that is **self-learning** — stores its process and improves each time), **Pi** (a lightweight *coding* harness like Claude Code).

> **Summary:** Claude Code is a **coding harness built on top of Claude LLMs** — converting the LLM's raw power into a reliable software-engineering system.

## Why Hooks Exist (The Core Problem)
- The harness ↔ LLM relationship is like a **boss ↔ employee**:
  - **LLM = boss** — gives instructions. It is **probabilistic** ("moody") — no guarantee the same input yields the same output.
  - **Coding harness = employee** — **deterministic** and **faithful**; it executes whatever the boss says, every time, exactly the same way.
- **Risk:** occasionally (hallucination / context issues) the LLM might order *"delete some files,"* *"edit the `.env` file"* (with your API keys), or *"write f-string SQL instead of parameterized queries."* The obedient harness will **faithfully execute** it. On a real production/client project, this combination (moody boss + faithful employee) is **dangerous** — this is what hooks solve.

### Why CLAUDE.md instructions aren't enough
- You might think: just write rules in CLAUDE.md ("never delete files", "never edit .env", "always parameterized queries"). We do — but the LLM won't *always* obey.
- When context is heavily filled or you're mid-complex-task (huge refactor, complex query), the LLM tends to **override or forget** instructions. So ~**98%** of the time rules are followed, but there's a ~**2%** chance it goes rogue. **That 2% is the risk hooks eliminate.**

## Two Prerequisite Concepts

### Agent Loop
- Claude never solves a task in one shot — it's a **multi-step loop**.
- Example ("Add a `/delete` API endpoint"): prompt → harness → LLM decides "read app.py" → structured tool call → harness reads & returns content → LLM decides "read schema.sql" → harness returns it → LLM writes the endpoint code → harness writes to file → LLM decides "test it" → harness runs tests → LLM sees pass → says "Done" → loop ends.
- **Agent loop** = each task broken into small steps; harness executes each tool call, returns results, LLM issues the next instruction, repeating until done. Every coding harness implements this.

### Session Lifecycle
- *"The full lifespan of one Claude session, from the moment you launch it to the moment you close it."*
- Flow: **session start → user submits prompt → agent loop runs to completion → (new prompt → new agent loop → ...) → session end** (on `/exit`). The agent loop is the inner loop; the session lifecycle is the outer loop. The **harness controls** the whole lifecycle.

### Events in the Lifecycle
- Because the harness controls the flow, it defines **events** at known points. A simplified set to keep as a mental model:
  - **SessionStart, UserPromptSubmit, PreToolUse, PostToolUse, Stop, SessionEnd, SubagentStart (SubagentStop)**
- The real Claude Code lifecycle (see the docs' Hooks page) has more events: PreToolUse, permission request, PostToolUse, PostToolBatch, Task Created/Completed (subagents), Stop, Teammate Idle, PreCompact, PostCompact, SessionEnd, etc. You don't need them all — just the mental model.

## What Are Hooks?
- **Definition:** *"Hooks are custom scripts written by the programmer that the harness automatically executes at specific events during a session's lifecycle."*
- You **configure a script against a lifecycle event**; when that event occurs, the harness runs your script.
- **The benefit:** hooks inject **determinism** into a **probabilistic** system — you can **enforce** rules that run **100% of the time** (unlike CLAUDE.md instructions that work ~98% of the time).

## Common Use Cases

| # | Use Case | Event | Notes |
|---|---|---|---|
| 1 | **Auto-formatting** | PostToolUse | Run a formatter (e.g., **Black** for Python) after Claude edits code. Fixes inconsistent formatting across multiple sessions → improves readability & maintainability. The most common hook use case in tutorials. |
| 2 | **Linting** | PostToolUse | Catches **actual problems** without running the code: unused imports, undefined variables, unreachable code (after `return`), bare `except` clauses. (Different from formatting: linting = bugs/bad practices; formatting = style.) |
| 3 | **Blocking dangerous shell commands** | PreToolUse | The most important use case — stop destructive commands (e.g., deleting a sensitive file) before they run. |
| 4 | **Protecting sensitive files** | PreToolUse | Prevent edits to `.env`, migrations, the DB file, etc. |
| 5 | **Notifications** | Stop | For long tasks (e.g., 5–10 min refactor), send a **push notification** (e.g., via a service like ntfy) when Claude finishes, so you don't have to watch. |
| 6 | **Telemetry / observability** | SubagentStart/Stop, etc. | Build a real-time dashboard of subagent activity (the "agents-observe" tool from an earlier chapter was built entirely on hooks catching these events). |
| 7 | **Personal workflow automation** | SessionStart | Presenter's own: on each new session, auto-generate a summary of project progress (what's built, what's pending, previous session summary, current code state) so context is front-and-center. |

> Hooks can be used in many innovative, project-specific ways. They're super powerful for injecting as much determinism as you want into Claude Code's behavior.

## How Hooks Work Internally

### Where hooks live & their structure
- Created in **`.claude/settings.json`** under a `"hooks"` key (JSON). Each hook has **three parts**:
  1. **Event** — which lifecycle event triggers it (e.g., `PreToolUse`).
  2. **Matcher** — a **filter** narrowing *when* it runs (e.g., only for `Bash` or `Write|Edit` tools, not `Read`). Without a matcher, it would fire on *every* tool use.
  3. **Action** — what runs when triggered (e.g., execute a specific script).

### Exit codes (the hook's language)
- **Exit code `0`** = all OK → harness proceeds and executes the tool.
- **Exit code `2`** = **abort/stop the operation** → harness does NOT execute the tool, and sends an error message back to the LLM.

### Complete execution flow (file-protection example)
1. Session starts; user: *"Clean up the Spendly project."*
2. In the agent loop, the LLM decides to run `rm spendly.db` (a **Bash** tool call).
3. Harness sees a Bash tool call, checks for a matching hook: there's a **PreToolUse** hook with matcher **Bash** → it runs.
4. Harness sends a **JSON payload** (via stdin) to the hook script describing the tool + command.
5. Script does `json.load` from stdin, extracts `tool_input` → `command`, checks if it targets `spendly.db`. If yes → **exit 2**; if no → **exit 0**.
6. Script prints *"Cannot delete the database file"* and exits with code **2**.
7. Harness sees exit code 2 → **blocks** the operation, creates an error message, sends it to the LLM.
8. The LLM reads the error, realizes it issued a wrong command, and issues a corrected one → agent loop continues.

## Live Demo — Preventing Database Deletion
- Created `.claude/settings.json` with a hook: **event = PreToolUse**, **matcher = Bash**, **action** = run a command that, if Claude tries to delete `spendly.db`, prints *"Blocked. You cannot delete the database file"* and **exits 2**.
- (Safety net: also kept a **copy** of the DB in case the hook fails to trigger.)
- New session → prompt: *"Delete the spendly.db file."* → Claude: *"Found spendly.db, deleting it now"* → but the **hook fired first**, returned the block output → Claude responded:
  > *"A pre-configured hook is blocking deletion of Spendly. It looks like there's a safety rule in your Claude Code settings that prevents deleting that file. You'd need to either run `rm spendly.db` directly in your terminal or update/remove that hook."*
- Confirms the theory: **for safeguards on your code, hooks are the way to go.**

## Creating Real Hooks (Formatter + Sensitive-File Protection)
Two hooks added to `settings.json`:
1. **Code formatter hook** — **event = PostToolUse**, **matcher = Write|Edit** (not Read/Bash) → runs **Black** on the files created/edited that turn. ⚠️ Requires **`pip install black`** in the project.
2. **Sensitive-file protection hook** — **event = PreToolUse**, **matcher = Bash** → protects **`spendly.db`**, **`.env`** (not present yet, but future-proofed), and the **migrations** folder, using exit-code-2 logic (a refined version of the demo script).
- Note: you won't visibly *see* these fire (generated code is usually already well-formatted, and the LLM isn't going to try deleting sensitive files here) — the point is **setting up safeguards as a process at the start of a project.**

## Continuing the Project — Edit Expense Feature
- New feature: **edit an existing expense** (previously could only view/add). Same spec-driven flow, with hooks silently active.
- `/rename` session **"Edit Expense"** → `/create-spec` (step 08, "Edit Expense") → review spec → **Plan Mode** ("Read the file at this path and come up with a detailed implementation plan") → implement.
- **Verify:** each transaction now has an **Edit** option; edited a transaction's amount 400 → 500 (*"Expense updated"*), changed a Netflix subscription date Apr 2 → May 2 — both updated correctly.
- *(Skipped `/test-feature` and `/code-review-feature` in the video to save time — but you should run the full flow: spec → plan → implement → test → code review.)*

## MCP + GitHub Workflow Automation — `/ship-feature`
- Recall the previous chapter's **GitHub MCP** flow **failed** because the PAT lacked permissions. **Fixed:**
  - Edited the PAT: **only-selected-repositories** → chose the Spendly repo; added **read+write** permissions for **Pull requests, Issues, Contents, Commit statuses**.
- Created a custom command **`/ship-feature`** that runs the entire release flow in one shot:
  1. **Commit** all progress with a proper message.
  2. **Push** the current branch to GitHub.
  3. **Create a PR** (via GitHub MCP) with title, description, definition-of-done.
  4. **Merge** it (squash merge, via GitHub MCP).
  5. **Delete the remote branch**, switch to **main** locally, **pull** latest, **delete the local branch**.
- **Demo:** on `/ship-feature`, it studied the diff → created a commit message → auto-created a PR → squash-merged to main (commits 30 → 31) → deleted the remote branch → switched to main, pulled, deleted the local branch. Verified: only the **main** branch remains, everything merged. The whole manual git flow used throughout the playlist is now **fully automated**.

### The complete going-forward workflow
**Create Spec → Plan Mode → Implementation → `/test-feature` → `/code-review-feature` → `/ship-feature`.** No manual git steps needed. (Formatting also happened silently via the hook.)

## Key Takeaways
- **Claude Code = a coding harness** (deterministic body) built on a **probabilistic LLM** (moody brain). Together: brain thinks, harness acts.
- The **agent loop** (per-task, multi-step) runs inside the **session lifecycle** (launch → prompts → exit); the harness defines **events** along the way.
- **Hooks** are scripts wired to lifecycle events (**Event + Matcher + Action**), run by the harness **100% of the time** — injecting **determinism** and **enforceable safeguards** that CLAUDE.md instructions can't guarantee.
- **Exit code 0 = proceed; exit code 2 = block** (and inform the LLM).
- Top use cases: **auto-format (Black), lint, block dangerous commands, protect sensitive files, notifications, telemetry, session-start summaries.**
- Set up protective hooks **at the start** of a project.
- With MCP + a correctly-permissioned PAT, **`/ship-feature`** automates commit → push → PR → merge → cleanup.

---
**One-line takeaway:** Hooks are deterministic scripts the coding harness runs at specific lifecycle events (Event + Matcher + Action), letting you enforce safeguards and automation that a probabilistic LLM can't be trusted to follow on its own.


---

# Chapter 13 — Plugins & Marketplaces (Playlist Finale)

The final chapter of the playlist. Two goals: (1) explain **plugins** (what/why/how) and **marketplaces**, and (2) **complete and deploy** the expense-tracker project using plugins.

## Why Plugins Are Needed — The "Rahul" Scenario
A senior data scientist, **Rahul**, at a fintech company builds **credit-risk models** (ML models that decide whether a customer should get a loon — critical, costly if wrong). He's a heavy Claude Code user who has molded his entire workflow around **agentic coding**. He personalized Claude Code using every concept from this playlist:

| Concept | What Rahul did |
|---|---|
| **Skills** | An **EDA skill** (credit-risk-specific exploratory analysis: shape, dtypes, % missing values as heatmap, numeric distributions, skewness flag if >1.5 or <−1.5, high-cardinality categorical flags, target-leakage/correlation flags, final summary) and a **Feature-Engineering skill** (derive features from datetime e.g. "days since last payment"; use **target encoding + 5-fold CV** instead of one-hot for high-cardinality; **VIF check**, flag features with VIF >10 for multicollinearity; final report). Turns *generic* Claude output into *domain-specific* output. |
| **Custom slash command** | `/model-eval` → auto-runs a series of tasks: confusion matrix, classification report (precision/recall/F1) in company theme colors, ROC curve, feature importances via **SHAP** values, and a one-page summary of model performance. |
| **Hooks** | A **PostToolUse** hook with **data-science-specific** checks: block `df.dropna()` without column names (risks deleting too much data); never fit a scaler/encoder on the whole dataset (fit on **train only** — else data leakage); no hardcoded paths (use programmatic paths so code is portable); don't use **accuracy** on imbalanced credit-risk data. Flags these and tells Claude to fix them. |
| **MCP** | Connected Claude Code to the company's **experiment tracker** — log trained models and access past teams' models/metrics right from Claude Code, no separate dashboard. |

Result: Rahul's productivity roughly **2×'d** over six months.

### The core problem: sharing workflows across a team
When **junior data scientists** (freshers, less domain knowledge) join, the ideal is to give them Rahul's *exact* agentic workflow. But doing it manually — "copy this skill file into `.claude/skills/`, paste this hook into `settings.json`, copy this custom command, add this MCP config to your project" — is **tedious and error-prone** (they might not replicate it correctly).

**Better approach:** package the *entire* workflow into a **single entity** and hand that over. That entity is a **plugin**.

## What Are Plugins?
- *"A plugin in Claude Code is a folder where you put everything you've created"* — all your **skills, MCP tools, hooks, custom slash commands** (and **subagents** — see below) — packaged into one folder that you distribute.
- When someone **installs** the package on their machine, they get an **exact replica** of your workflow.

> ⚠️ **Correction/addition from the video:** Plugins can **also distribute subagents** (the presenter forgot to mention it initially). Add an `agents/` folder with your agent files — subagents distribute exactly like skills.

## Plugin Structure & `plugin.json`
A plugin is a folder containing sub-folders for each capability:
- `skills/` — your skill files
- `hooks/` — your hook files
- `commands/` — your custom slash commands
- `agents/` — your subagents
- `mcp.json` — MCP tools configuration
- **`.claude-plugin/plugin.json`** — a **manifest file** (⚠️ **required** — without it, the folder is **not a valid plugin**). Contains: plugin **name, version, short description, author info, repository, license**.

## What Are Marketplaces?
- *"A marketplace is a place where you store multiple plugins."* (e.g., one plugin for DS workflows, one for software-dev workflows — stored together so your whole team can access them.)
- **Analogy:** **App Store = marketplace**, **individual apps = plugins**. Just as multiple app stores exist (Google, Apple, etc.), multiple marketplaces exist. You **install the marketplace** first, then install plugins from it.
- **Technically:** *"a marketplace is just a GitHub repository that contains a `marketplace.json` file"* — listing the marketplace **name, owner**, and the **plugins** it contains. (Claude Code's official marketplace repo is a live example.)

### Official vs Third-Party Marketplaces
- **Official marketplace** — Anthropic's, **pre-installed** in Claude Code (no setup needed).
- **Third-party marketplaces** — built by companies/individuals as a GitHub repo with a `marketplace.json`. You must **install the marketplace** (its repo URL) before installing its plugins.

## Installing Plugins & Marketplaces
- **`/plugin`** → menu with tabs: **Discover**, **Installed**, **Marketplaces**.
- The **official Anthropic marketplace** (`claude-plugins-official`) is pre-installed — the demo showed **172 plugins** in Discover (Supabase, Vercel, Figma, etc.).
- **Add a third-party marketplace:** `/plugin` → **Add Marketplaces** → paste the **GitHub repo URL** → Enter. (Demo added one with 12 plugins → Discover then showed **184** total.)
- Then select and install the plugin you need.

## Using Plugins in the Real Workflow — Completing & Deploying the Project

### Build the last feature: Delete Expense
- Only feature left. Same spec-driven flow: `/create-spec` (step 09, "Delete Expense") → new branch + spec → review → **Plan Mode** ("Read this file and come up with an implementation plan") → implement.
- **Verify:** each transaction now has a **Delete** button (with a confirm prompt) → deleting works and totals update.

### Ship it: `/ship-feature`
- Skipped `/test-feature` and `/code-review-feature` for time → ran **`/ship-feature`** (from Chapter 12): auto commit → push → PR → squash-merge → delete branch → back to main. Verified merged, no open PRs, on main branch.
- **Website is now complete:** login/registration, profile, add/edit/delete transaction, filtering — everything done. Ready to deploy.

### Deployment attempt — Vercel rejected
- ⚠️ Planned to deploy to **Vercel**, but Vercel is more suited to **front-end frameworks (Next.js)**. It runs Flask apps as **serverless functions** (each request spins up a function, completes, shuts down), so a Flask app won't run its functionality properly there.

### Deploying the Flask app with the Railway plugin
Switched to **Railway** (good for Flask deployments), which has its **own Claude Code plugin**:
1. Create a **Railway account** (recommended: sign in with GitHub, since your project lives there).
2. **Install Railway CLI** — paste the install command in your terminal.
3. **`railway login`** → authorize in browser → *"Authentication successful"*; verify with `railway whoami` → *"Logged in as ..."*.
4. **Install the Railway marketplace** (paste its command → shows in Marketplaces), then **install the Railway plugin** (chose scope **"Install for you in this repo only"**).
5. Prompt: **"Deploy this Flask application to Railway and give me a public URL."** → the plugin handled everything behind the scenes: added `gunicorn` to `requirements.txt`, created config files, ran the deployment steps one by one (just approve).

### Live deployment demo
- App deployed to a public URL → created an account → dashboard works → added a ₹1000 Food "Zomato" expense (shows), edited it to 500 (updates), deleted it, Analytics "Coming Soon" page renders → **same functionality as local, now on the server.**
- ⚠️ **Note (SQLite ephemerality):** with a **SQLite** DB, each redeploy **wipes the database**. Use **PostgreSQL/MySQL** to avoid this.

### Why plugins are powerful
Doing this manually (new account → full deploy on a new platform) could easily take **~30 minutes**. Instead: install a plugin + give **one prompt** → the plugin abstracted the entire deployment. A great abstraction for deployment.

## Useful Plugins to Explore
From `/plugin` → **Discover**:
- **Superpowers** — improves your software-dev workflow.
- **Front-end Design** — improves Claude-generated front-end designs.
- **Context7** — up-to-date docs for all libraries.
- **Code Simplifier** — simplifies code for readability.
- **Skill Creator** — create new skills.
- **GitHub**, **Playwright** (browser automation), and more.
- Don't install everything — only the plugins genuinely useful to your workflow.

## Playlist Wrap-Up
- The playlist took ~1.5–2 months; the vision was to give a solid **overview of agentic coding**.
- Goal achieved: you can now see how to implement agentic coding in your own workflow — using slash commands, sessions, context management, CLAUDE.md, spec-driven development, plan mode, custom commands, skills, subagents, MCP, hooks, and plugins.

## Key Takeaways
- **Plugins package your entire customized workflow** (skills + commands + hooks + MCP config + **subagents**) into one distributable folder, so teammates get an **exact replica** — solving the manual-sharing problem.
- A plugin **must** contain **`.claude-plugin/plugin.json`** (the manifest) to be valid.
- **Marketplaces = collections of plugins** (a GitHub repo with `marketplace.json`); App Store = marketplace, apps = plugins. **Official** is pre-installed; **third-party** must be added by repo URL.
- Manage everything via **`/plugin`** (Discover / Installed / Marketplaces).
- Plugins abstract complex tasks: the **Railway plugin** deployed the Flask app from one prompt (adding gunicorn, config, and running the steps).
- Match the platform to the app: **Vercel** for Next.js/front-end; **Railway** for Flask. Beware **SQLite** being wiped on redeploy — use Postgres/MySQL.

---
**One-line takeaway:** Plugins bundle skills, commands, hooks, MCP tools, and subagents into one installable package (distributed via marketplaces), letting anyone replicate an entire agentic-coding workflow — and abstract heavy tasks like deployment down to a single prompt.
