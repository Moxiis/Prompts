# GitHub Spec Kit — Practical Cheatsheet

A reference for working with [Spec Kit](https://github.com/github/spec-kit), GitHub's toolkit for **Spec-Driven Development (SDD)**.

## Table of Contents

1. [What Spec Kit Is](#1-what-spec-kit-is)
2. [Command Quick Reference](#2-command-quick-reference)
3. [Project Structure & Core Artifacts](#3-project-structure--core-artifacts)
4. [Constitution vs. Agent Context File](#4-constitution-vs-agent-context-file-copilot-instructionsmd--claudemd--agentsmd)
5. [Mapping Your Old "Project Plan" to Spec Kit](#5-mapping-your-old-project-plan-to-spec-kit)
6. [Typical End-to-End Workflow](#6-typical-end-to-end-workflow)
7. [Greenfield Workflow](#7-greenfield-workflow-new-project)
8. [Brownfield Workflow](#8-brownfield-workflow-existing-codebase)
9. [Monorepo & Multiple Constitutions](#9-monorepo--multiple-constitutions)
10. [Presets & Extensions](#10-presets--extensions)
11. [Spec Persistence Models](#11-spec-persistence-models)
12. [Complex Features & Context-Window Management](#12-complex-features--context-window-management)
13. [Quality Gates & Critique Loop](#13-quality-gates--critique-loop)
14. [Extra Tips & Things People Miss](#14-extra-tips--things-people-miss)

---

## 1. What Spec Kit Is

**Spec Kit = a CLI (`specify`) + Markdown templates + slash commands** that turn AI coding agents into something closer to a structured pair-programmer instead of a "type a prompt, get a code dump" tool.

Core idea: **specifications become the source of truth, code becomes the generated output.** Instead of code being king and specs being throwaway scaffolding, you write a spec, plan, and task list — and the agent generates and regenerates code from those.

**Core philosophy:**

- Intent-driven: define the *what/why* before the *how*
- Specs are refined over multiple steps, not generated in one shot
- Heavy reliance on the agent to interpret structured Markdown, not free-form chat

**Development phases it targets:**

| Phase | Focus |
|---|---|
| **Greenfield** ("0-to-1") | New project from scratch |
| **Creative exploration** | Parallel implementations of the same spec (different stacks/UX) |
| **Brownfield** ("iterative enhancement") | Adding features to / modernizing an existing codebase |

**Why it exists:** AI agents are literal-minded pair programmers, not mind readers. A vague one-shot prompt produces code that compiles but misses intent. Spec Kit forces the ambiguity to surface *before* code is written, where it's cheap to fix.

---

## 2. Command Quick Reference

Run `specify init` once per project to install these as slash commands in your agent (`.claude/commands/`, `.github/prompts/`, etc.). Codex CLI in skills mode uses `$speckit-*` instead of `/speckit.*`.

### Core commands

| Command | Purpose | Produces |
|---|---|---|
| `/speckit.constitution` | Create/update project governing principles | `.specify/memory/constitution.md` |
| `/speckit.specify` | Define **what** to build and **why** | New branch/dir + `specs/NNN-name/spec.md` |
| `/speckit.plan` | Define **how** — tech stack & architecture | `plan.md`, `research.md`, `data-model.md`, `contracts/`, `quickstart.md` |
| `/speckit.tasks` | Break the plan into ordered, executable tasks | `tasks.md` |
| `/speckit.taskstoissues` | Push tasks into GitHub Issues for tracking | GitHub issues |
| `/speckit.implement` | Execute tasks in dependency order | Source code |

### Optional (quality-gate) commands

| Command | Purpose | Produces |
|---|---|---|
| `/speckit.clarify` | Ask up to 5 targeted questions to resolve ambiguity *before* planning | `## Clarifications` section in `spec.md` |
| `/speckit.checklist` | Generate "unit tests for requirements" — checks spec quality, not implementation | `specs/.../checklists/*.md` |
| `/speckit.analyze` | Cross-check `spec.md` / `plan.md` / `tasks.md` for gaps & contradictions | Findings report |

### CLI (not slash commands — run in your terminal)

```bash
specify init <name>                       # new project
specify init . --integration claude       # existing dir, pick agent
specify init --here --force --integration copilot   # re-scaffold in place
specify check                             # detect installed agent tools
specify self upgrade   /  self check      # upgrade or check the specify CLI itself
specify extension search / add <name>     # browse & install extensions
specify preset search / add <name>        # browse & install presets
specify integration list                  # list supported AI agent integrations
```

> 30+ agent integrations are supported (Copilot, Claude Code, Gemini CLI, Cursor, Codex, Windsurf, Kiro, and more) — same workflow, different command-file format per agent.

---

## 3. Project Structure & Core Artifacts

### 3.1 Directory tree (annotated)

```text
my-project/
├── .specify/
│   ├── memory/
│   │   └── constitution.md        # project-wide governing principles (this never lives in specs/)
│   ├── templates/
│   │   ├── constitution-template.md
│   │   ├── spec-template.md
│   │   ├── plan-template.md
│   │   ├── tasks-template.md
│   │   ├── overrides/             # your project-local template tweaks — HIGHEST priority
│   │   └── commands/              # source for the installed /speckit.* prompts
│   ├── presets/templates/         # installed preset overrides (if any)
│   ├── extensions/templates/      # installed extension templates (if any)
│   ├── scripts/
│   │   ├── bash/                  # create-new-feature.sh, setup-plan.sh, check-prerequisites.sh...
│   │   └── powershell/            # same scripts for Windows
│   └── init-options.json
├── specs/
│   └── 001-photo-albums/          # one directory per feature, numbered
│       ├── spec.md                # WHAT & WHY
│       ├── plan.md                # HOW (stack, architecture)
│       ├── research.md            # technical research notes
│       ├── data-model.md          # entities & relationships
│       ├── contracts/             # API / event contracts (OpenAPI, GraphQL, WS events…)
│       ├── quickstart.md          # key scenarios to manually validate
│       ├── checklists/
│       │   └── requirements.md    # output of /speckit.checklist
│       └── tasks.md               # dependency-ordered task breakdown
└── .claude/ (or .github/prompts/, .cursor/, …)   # agent-specific command files
```

### 3.2 Constitution — `​.specify/memory/constitution.md`

**What it is:** the one file that doesn't belong to any single feature. Project-wide rules that `/speckit.plan`, `/speckit.tasks`, and `/speckit.analyze` check every feature against. Written once, amended rarely.

**Typical structure:**

- **Project Purpose** — one paragraph, what problem this solves
- **Core Principles** — numbered (I, II, III…), each a short rule + rationale
- **Success Criteria** — what "MVP done" looks like, what's explicitly out of scope
- **Governance** — amendment process, versioning policy (semantic versioning), compliance review cadence

**Principle categories worth borrowing** (from the original SDD methodology's example "nine articles" — not mandatory, just good defaults):

- **Library-first** — features start as standalone, reusable modules, not code glued into the app
- **CLI-first / observability** — every capability should be inspectable via text in/out
- **Test-first (non-negotiable)** — tests written and failing *before* implementation
- **Simplicity gate** — cap project/module count; new complexity must be justified in writing
- **Anti-abstraction** — use the framework directly; no wrapper "just in case"
- **Integration-first testing** — real databases/services in tests over mocks where practical

# AGENT: Please add the principles from my constitution.md file that I provided in the attachment. I think they are important, so I want to include them in this cheat sheet as a reference for future users. If they fit the "Principle categories worth borrowing" feel free to add them there as well

**Concrete example:**

```markdown
# Project Constitution: PhotoVault
**Version**: 1.1.0 | **Ratified**: 2026-01-10 | **Last Amended**: 2026-03-02

## Project Purpose
Organize personal photos into date-based albums with drag-and-drop
reordering. No nested albums, no cloud upload.

## Core Principles

### I. Simplicity First
Start with the minimum viable feature set. Added complexity must be
justified, in writing, against a simpler alternative.

### II. Local-First Data
All metadata lives in a local SQLite database. No image is ever
uploaded to a remote service.

### III. Test-First (NON-NEGOTIABLE)
No implementation code before tests. Tests must fail (red) before
implementation begins (green).

## Success Criteria
- MVP: create/reorder albums, tile photo previews, no login
- Explicitly out of scope (v1): sharing, cloud sync, facial recognition

## Governance
### Amendment Process
1. Propose via PR to this file with rationale + impact assessment
2. Requires maintainer sign-off
3. Constitution supersedes all other docs on conflict

### Versioning
MAJOR = principle removed/redefined · MINOR = principle added · PATCH = wording fix
```

### 3.3 Specification — `spec.md`

**What it is:** the **what/why** for one feature. Deliberately tech-agnostic — no stack, no APIs, no code structure. If the prompt doesn't specify something, the template forces an explicit `[NEEDS CLARIFICATION: ...]` marker instead of letting the agent guess.

**Typical structure:**

- Header: feature branch name, created date, status
- **User Scenarios & Testing** — user stories with priority (P1, P2…), `Given/When/Then` acceptance scenarios, edge cases
- **Requirements** — numbered functional requirements (`FR-001`, `FR-002`…), each a testable "System MUST…" statement
- **Success Criteria** — measurable, tech-agnostic (`SC-001`…) — these define "done," not the implementation

**Concrete example:**

```markdown
# Feature Specification: Photo Album Organizer
**Feature Branch**: `001-photo-albums` | **Created**: 2026-04-02 | **Status**: Draft

## User Scenarios & Testing
### User Story 1 - Create an Album (Priority: P1)
As a user, I want to create an album grouped by date so I can
organize photos without thinking about folders.

**Acceptance Scenarios**:
1. **Given** the main page is open, **When** I click "New Album" and
   pick a date, **Then** an empty album tile appears in date order.

### Edge Cases
- Two albums with the same date → both shown, ordered by creation time
- Album with 1000+ photos → [NEEDS CLARIFICATION: max photos per album?]

## Requirements
### Functional Requirements
- **FR-001**: System MUST allow users to create an album with a date
- **FR-002**: System MUST prevent nested albums

## Success Criteria
- **SC-001**: A new album can be created in under 3 seconds
- **SC-002**: Reordering persists after a page reload, 100% of the time
```

### 3.4 Plan — `plan.md`

**What it is:** the **how** — generated *from* the spec plus the constitution. This is where the tech stack, architecture, and constitutional compliance checks live.

**Typical structure:**

- **Technical Context** — language, framework, storage, testing tool
- **Constitutional Compliance / "Phase -1 Gates"** — explicit checkboxes against each constitution principle (e.g. "Using ≤3 projects? ✅", "Framework used directly? ✅"); failures must be justified in a "Complexity Tracking" section, not silently skipped
- Companion files it generates: `research.md` (tech investigation), `data-model.md` (entities), `contracts/` (API/event contracts), `quickstart.md` (validation scenarios)

**Concrete example (abridged):**

```markdown
# Implementation Plan: Photo Album Organizer
**Input**: specs/001-photo-albums/spec.md

## Technical Context
Language: JavaScript (vanilla) | Stack: Vite | Storage: local SQLite | Testing: Vitest

## Constitutional Compliance
- Simplicity (Principle I): ✅ single project, no added framework
- Local-First (Principle II): ✅ no network calls for images
- Test-First (Principle III): ✅ contract tests written before implementation

## Phase 0 → research.md   Phase 1 → data-model.md, contracts/, quickstart.md
```

### 3.5 Tasks — `tasks.md`

**What it is:** the dependency-ordered, executable breakdown of the plan — what `/speckit.implement` actually walks through.

- Tasks organized **by user story/phase** (Setup → Foundational → US1 → US2 → … → Polish)
- Task IDs (`T001`, `T002`…), `[P]` marks tasks safe to run in parallel, `[US#]` links a task back to its user story
- Each user-story phase ends with a **checkpoint** confirming that slice works independently

```markdown
## Phase 2: User Story 1 - Create Album (P1) 🎯 MVP
- [ ] T003 [P] [US1] Contract test: POST /albums
- [ ] T004 [US1] Implement album creation service
- [ ] T005 [US1] Wire up "New Album" UI button
**Checkpoint**: Album creation works end-to-end, independently testable
```

---

## 4. Constitution vs. Agent Context File (`copilot-instructions.md` / `CLAUDE.md` / `AGENTS.md`)

This is the single most-confused distinction in the Spec Kit community (it has its own recurring GitHub Discussion thread). Here's the practical answer:

| | **Constitution** (`.specify/memory/constitution.md`) | **Agent context file** (`copilot-instructions.md`, `CLAUDE.md`, `AGENTS.md`...) |
|---|---|---|
| Scope | Whole project, all features | Whatever the agent reads automatically, every session |
| Lifespan | Long-lived, amended rarely, versioned | Auto-rewritten by `update-agent-context` script after every `/speckit.plan` |
| Content | **Durable principles**: simplicity rules, testing discipline, architectural non-negotiables, amendment process | **Current operating state**: active feature pointer, language/framework/DB *currently* in use, recent-changes log, agent-specific working rules |
| Who checks it | `/speckit.plan`, `/speckit.tasks`, `/speckit.analyze` explicitly read and validate against it | The agent's normal context-loading mechanism (every chat turn, automatically) |
| Changes how often | Rarely — it's a constitution, not a changelog | Every feature — it's a status board |

**What should go in the agent context file that should NOT be duplicated from constitution/spec/plan:**

- "Feature in progress" pointer: which `spec.md`/`plan.md` you're currently working from
- The *currently detected* language/framework/database (auto-extracted from the latest `plan.md`)
- A short recent-changes log for orientation after a context reset
- Agent-specific operating rules: required lint/test commands before finishing, shell/IDE quirks, subagent/persona definitions, PR/commit etiquette
- Imports/links to other docs (`README.md`, `AGENTS.md`, `codingstandards.md`) if you run multiple agents and want one shared baseline

**What should NOT go there** (it belongs elsewhere and duplicating it causes drift):

- Durable architecture rules → constitution.md
- Feature requirements / acceptance criteria → spec.md
- Stack rationale for a specific feature → plan.md (this file gets overwritten anyway)

**Practical recommendation:** pick one source of truth. Either let the constitution stay pure principles and have `copilot-instructions.md` be a one-line pointer (`See .specify/memory/constitution.md`), or accept that the auto-generated context file is allowed to *summarize* the constitution's existence without re-stating its content.

---

## 5. Mapping Your Old "Project Plan" to Spec Kit

*Assumption: by "project plan" you mean the common single-document format — vision/goals, requirements, tech stack, timeline/milestones, task list, all in one file, written once up front.* The mapping below should hold even if your format differs slightly — adapt it to your actual document if needed.

| Your old "project plan" section | Spec Kit home | Why the split helps |
|---|---|---|
| Vision, non-negotiable rules ("always/never do X") | `constitution.md` — once, project-wide | Survives every feature; machine-checked by `/plan` and `/analyze` instead of just sitting in a doc nobody re-reads |
| Requirements, user stories, acceptance criteria | `spec.md` — per feature | Stays tech-agnostic, so a stack change doesn't force a rewrite |
| Tech stack, architecture, data model | `plan.md` + `research.md` + `data-model.md` — per feature | Generated from spec + constitution; regenerable if requirements change |
| Timeline / milestones / work breakdown | `tasks.md` — per feature | Auto-derived, dependency-ordered, marked for safe parallelism |
| "Definition of done" checklist | `checklists/*.md` + `/speckit.analyze` | Checked against the artifacts automatically, not just eyeballed |

**The structural shift that matters most:** one monolithic document written once → many small, *regenerable* artifacts written per feature, anchored by a single constant file (the constitution) that holds only what shouldn't change feature-to-feature. This is what makes "requirements changed" a normal workflow step instead of a rewrite of your whole plan.

# AGENT: As Mapping Your Old "Project Plan" to Spec Kit I refer to the project plan docs.txt file I worked on before focusing on Speckit. I don’t want it included as a separate file or topic in the cheat sheet. Instead, incorporate the information from that file into the cheat sheet and map it to the Speckit workflow. You can remove the fifth point, “Mapping your old project plan to Speckit,” and integrate the content from project plan.txt into the cheat sheet, as this was previously addressed

---

## 6. Typical End-to-End Workflow

**Full production path** (recommended for anything beyond a throwaway prototype):

```mermaid
flowchart LR
    A["/speckit.constitution"] --> B["/speckit.specify"]
    B --> C["/speckit.clarify"]
    C --> D["/speckit.plan"]
    D --> E["/speckit.checklist"]
    E --> F["/speckit.tasks"]
    F --> G["/speckit.analyze"]
    G --> H["/speckit.implement"]
```

**Lean path** (quick experiments / spikes — explicitly skips the quality gates):

```text
/speckit.specify → /speckit.plan → /speckit.tasks → /speckit.implement
```

Notes:

- `/speckit.constitution` is normally run **once** per project; you don't repeat it for every feature.
- `/speckit.clarify` and `/speckit.checklist` operate on `spec.md` *before* planning — cheapest place to catch gaps.
- `/speckit.analyze` runs *after* `/speckit.tasks`, *before* `/speckit.implement` — last checkpoint while the plan/tasks are still cheap to adjust. You can re-run it after implementation too, but don't skip the pre-implementation pass.
- If you intentionally want the lean path (a spike), say so explicitly — otherwise the agent may stall waiting for clarification.

---

## 7. Greenfield Workflow (new project)

1. **Install once:** `uv tool install specify-cli --from git+https://github.com/github/spec-kit.git`
2. **Bootstrap:** `specify init photo-app --integration claude` (or `copilot`, `gemini`, etc.)
3. `cd photo-app`, open your agent
4. `/speckit.constitution Create principles focused on code quality, testing standards, UX consistency, and performance requirements.`
5. `/speckit.specify Build a photo album organizer. Albums grouped by date... [focus on WHAT/WHY, not stack]`
6. `/speckit.clarify` — resolve ambiguities before planning
7. *(optional)* `/speckit.checklist` — validate the spec itself
8. `/speckit.plan Use Vite, vanilla HTML/CSS/JS, local SQLite, no image uploads.`
9. `/speckit.tasks`
10. `/speckit.analyze` — catch gaps before implementation starts
11. `/speckit.implement`
12. Manually test; paste runtime errors (browser console, etc.) back to the agent
13. For the **next feature**, repeat from step 5 — skip the constitution step unless principles need to change

# AGENT: After point 12, note that we can modify the current feature using the standard agent chat we used to create it. If something isn’t working, you can try to fix it with the agent in the current chat window. If the context window is overloaded, start a new chat with the agent and continue working on the feature there

---

## 8. Brownfield Workflow (existing codebase)

The key difference: there's usually no pre-existing constitution or specs, and you don't want the agent rewriting working code wholesale.

1. **Scaffold in place, without disturbing Git:**

   ```bash
   specify init --here --integration copilot
   # or, if you don't want Spec Kit touching git at all:
   specify init --here --no-git
   ```

2. **Generate the constitution from the codebase**, not from a blank template:

   ```text
   /speckit.constitution As this is a pre-existing brownfield project, analyze the
   codebase exhaustively (do not skim) and derive principles for code quality,
   testing standards, and performance. Include governance for how these guide
   future decisions.
   ```

3. *(Recommended for large codebases)* Ask the agent to write a **compressed view of the codebase** first, and reference it from every spec afterward:

   ```text
   Analyze this codebase. Document architecture, key modules, data flows,
   integration points, and existing conventions. Save as docs/codebase-research.md.
   ```

4. `/speckit.specify` — brownfield specs are often just a short natural-language outcome description (the architecture context already lives in the constitution / research doc), not a full PRD.
5. `/speckit.clarify` if anything is underspecified.
6. `/speckit.plan` — be explicit that the *existing* stack should be reused (otherwise the agent may introduce a new framework instead of matching what's there).
7. `/speckit.tasks`
8. `/speckit.analyze` — **don't skip this on brownfield.** Real-world walkthroughs that skipped it hit avoidable issues (CORS misconfiguration, debugging detours) that a pre-implementation check would have caught.
9. `/speckit.implement` — for terminal/CLI agents, you may need to enable an auto-approve/"YOLO" mode to let the agent start local dev servers; without it, implementation can stall mid-run.

**Multi-repo brownfield:** install Spec Kit in one dedicated repo (e.g. `design-notes/`) and open your other repos as sibling workspace folders for code context. Spec Kit can read across them for context and write specs locally, but it won't create branches in the other repos automatically — it asks for confirmation first.

---

## 9. Monorepo & Multiple Constitutions

A Spec Kit "project" is **directory-scoped**: it's whichever directory contains a `.specify/`. One monorepo can contain several independent Spec Kit projects, each with **its own constitution and feature numbering** — e.g. one for `web`, one for `api`, one for a `preprocessing` pipeline.

```text
my-monorepo/
├── .git/                       # one shared Git repo at the root
├── apps/
│   ├── web/
│   │   └── .specify/           # Spec Kit project "web"
│   │       └── memory/constitution.md
│   └── api/
│       └── .specify/           # Spec Kit project "api"
│           └── memory/constitution.md
└── packages/
    └── preprocessing/
        └── .specify/           # Spec Kit project "preprocessing"
```

```bash
specify init apps/web --integration claude
specify init apps/api --integration claude
cd apps/web   # commands resolve to the NEAREST .specify/, not the repo root
```

**Targeting a project without `cd`** (CI / scripted runs):

```bash
export SPECIFY_INIT_DIR=apps/web      # selects the PROJECT
export SPECIFY_FEATURE_DIRECTORY=specs/003-search   # selects the FEATURE within it
```

A bad path errors loudly instead of silently falling back to the repo root — a typo never writes specs into the wrong project.

**Key facts:**

- Each member project's `constitution.md` is fully independent — Spec Kit has **no built-in inheritance**. If you want shared rules across `web`/`api`/`preprocessing`, you maintain that duplication or wiring yourself.
- Git branches are still shared at the repo root (one Git repo), even though spec directories live per-project — manage branch namespacing accordingly, or use separate Git repos per member project if you need full isolation.

---

## 10. Presets & Extensions

Two complementary customization systems, plus project-local overrides:

| | **Extensions** | **Presets** |
|---|---|---|
| Adds | New commands / new capabilities | Nothing new — reshapes existing output |
| Use when | You need functionality beyond core SDD (Jira sync, code review, project health checks) | You want a different *format/terminology/style* (compliance traceability, Agile/Kanban framing, a stricter test-first gate) |
| Install | `specify extension add <name>` | `specify preset add <name>` |
| Example | **Jira Integration** (syncs tasks to Jira), **Red Team** (adversarial spec review), **Worktrees** (parallel-agent git isolation) | A compliance preset adding regulatory traceability fields to `spec.md`; the community "pirate-speak" preset that renames specs → "Voyage Manifests" |

**Resolution priority** (highest wins, templates resolved at runtime):

| Priority | Layer | Location |
|---|---|---|
| 1 (highest) | Project-local overrides | `.specify/templates/overrides/` |
| 2 | Presets | `.specify/presets/templates/` |
| 3 | Extensions | `.specify/extensions/templates/` |
| 4 (lowest) | Core | `.specify/templates/` |

Multiple presets/extensions can stack with explicit priority ordering. Removing one automatically restores the next-highest-priority version underneath it.

```bash
specify extension search        # browse available extensions
specify extension add red-team  # install one
specify preset search
specify preset add <name>
```

**When to use which (rule of thumb):** new command/workflow → extension. Reformat what specs/plans/tasks look like → preset. Enforce a regulatory or org standard → usually a preset (template change) or an extension (added gate command) depending on whether it's a format change or a new check.

---

## 11. Spec Persistence Models

# Agent. Provide information about what spec persistence models are, why they are important, and how to use them. Include basic details for readers unfamiliar with the topic

Spec Kit deliberately doesn't force one answer to "what happens to `spec.md`/`plan.md`/`tasks.md` after requirements change?" Pick one as a **team convention** — it's not a CLI flag.

| Model | Mutation rule | Best fit | Watch out for |
|---|---|---|---|
| **Flow-Back** | Edit any artifact first (spec, plan, tasks, or code), then reconcile the rest | Small teams, fast iteration, implementation discoveries reshaping the plan | Silent drift — a code-level fix that never gets reflected back into `spec.md` |
| **Flow-Forward** | Never mutate a completed feature directory — create a **new** numbered feature directory instead | Auditability, historical clarity, well-scoped features rarely revisited | Duplicated/fragmented context across many directories — needs naming/linking discipline |
| **Living Spec** | `spec.md` is the contract; edit it first, then regenerate `plan.md`/`tasks.md` from it | Spec is genuinely the stable, authoritative contract | Losing implementation rationale if derived files are regenerated without preserving key decisions |

**How to choose** — answer two questions and write the answer into your constitution or onboarding notes:

1. Should a completed feature directory be a historical record (→ Flow-Forward) or an editable work area (→ Flow-Back)?
2. Is `spec.md` the single source of truth (→ Living Spec), or are `plan.md`/`tasks.md` allowed to become co-equal sources (→ Flow-Back)?

**Practical workflow per model** (when requirements change mid-project):

- **Flow-Forward:** `/speckit.specify` for a new feature dir → `/speckit.plan` → `/speckit.tasks` → `/speckit.implement`. Keep the old directory; cross-link if it's superseded.
- **Living Spec:** edit `spec.md` (via `/speckit.clarify` or directly) → rerun `/speckit.plan` → rerun `/speckit.tasks` → `/speckit.analyze` → `/speckit.implement`. Carry forward any still-relevant rationale before discarding old derived files.
- **Flow-Back:** make the change wherever the insight landed → decide what else it invalidates → update the disagreeing artifacts → run `/speckit.analyze` → resume implementation only once the artifact set is trustworthy again.

You can even mix models across a monorepo's member projects, as long as each project's contributors know which convention applies to that project.

---

## 12. Complex Features & Context-Window Management

# AGENT: Give the information about what it is and why it is important to keep the context window fresh

**Root cause of "the agent lost the plot mid-implementation":** context window exhaustion. A single long `/speckit.implement` run that tries to hold an entire large feature in context degrades as the window fills — usually right around context compaction.

**Four options, in order of how much overhead they add:**

| Option | How | Best for |
|---|---|---|
| 1. Limit tasks per run | `/speckit.implement only execute tasks T001-T010, then stop and report progress` (or scope by phase) | Any agent — simplest, no special support needed |
| 2. Sub-agent delegation | `/speckit.implement delegate each parallel [P] task to a sub-agent` | Agents that support sub-agents (Copilot CLI, Copilot VS Code extension) — maximizes parallelism |
| 3. Combine both | `/speckit.implement execute only the Core phase, delegate [P] tasks to sub-agents` | Large features on sub-agent-capable agents |
| 4. Decompose into smaller specs ("spec of specs") | Break the feature into independent sub-features, each with its own `spec.md`/`plan.md`/`tasks.md` | Only when even a single phase overwhelms context — highest overhead |

Because completed tasks are marked `[X]` in `tasks.md`, re-invoking `/speckit.implement` always **picks up where it left off** — this is what makes option 1 safe to use liberally.

**When to start a new chat session vs. continue in the same one — practical rule of thumb:**

- **New session per phase/command** when: you're switching from planning-type work to execution-type work (e.g., a fresh session for `/speckit.plan`, a fresh one for `/speckit.implement`), or a single phase is large enough that you'd otherwise hit option 1 above repeatedly within one session.
- **Stay in one session** for: the spec → clarify → checklist loop on a single feature — that context is small and benefits from continuity (the agent remembers *why* it asked what it asked).
- **Always start fresh for review/critique** (see §13) — a reviewer with no investment in the work it's checking catches more.

A real community pattern worth copying ([AIDE extension](https://github.com/mnriem/spec-kit-extensions/tree/main/aide)) explicitly tells the user: *"Start a fresh chat session for each step. This keeps the AI focused on the specific task at hand."* — applied to vision → roadmap → progress → queue → item → execute stages of a larger build.

---

## 13. Quality Gates & Critique Loop

# agent: Explain the purpose of this point and why it matters, addressing readers who are unfamiliar with the original material and are new to the topic

**Does Spec Kit already have a "catch what I'm missing" mechanism? Yes — three layers, increasing in scope:**

| Gate | Checks | When |
|---|---|---|
| `/speckit.clarify` | Ambiguity in the spec itself — functional scope, data model, UX flow, edge cases, terminology | Before `/speckit.plan` |
| `/speckit.checklist` | Quality of the *requirements writing* — completeness, clarity, measurability ("unit tests for English," not for code) | After `/speckit.plan`, before/alongside `/speckit.tasks` |
| `/speckit.analyze` | Cross-artifact consistency — do spec, plan, and tasks actually agree with each other | After `/speckit.tasks`, before `/speckit.implement` (re-runnable after, too) |

These are all **self-review by the same agent/session**, though — useful but not adversarial. For a genuinely independent check, two extra patterns are worth adding:

**1. Fresh-session / different-agent critique (DIY, no extra tooling needed):**
Open a new chat (or switch models/agents) and feed it only `spec.md` + `plan.md` + the constitution — no implementation history — with a prompt like: *"Review this spec and plan as if you'd never seen them. What's underspecified, contradictory, or over-engineered relative to the constitution?"* A reviewer with zero investment in the existing answer is structurally more likely to flag real gaps than the same session that wrote it.

**2. Community extensions built specifically for this:**

| Extension | What it does |
|---|---|
| **Red Team** | Adversarial review before `/speckit.plan` — parallel "lens" agents surface risks (prompt injection, integrity gaps, cross-spec drift) that `/clarify`/`/analyze` don't structurally catch |
| **Spec Critique** | Dual-lens review of spec + plan from product-strategy *and* engineering-risk perspectives |
| **Verify** / **Verify Tasks** | Post-implementation check that code actually matches the spec; flags tasks marked `[X]` with no real implementation behind them |
| **Staff Review** | Staff-engineer-level code review: security, performance, test coverage vs. the spec |
| **Multi-Model Review** | Routes spec authoring / implementation / review across *different* models, so the reviewer is never the same model that wrote the artifact |
| **Architecture Guard** | Continuous drift detection between constitution/specs/plans and actual code, producing refactor tasks |

**Recommended order for anything non-trivial:** `clarify` → `checklist` → `plan` → `analyze` → (optional adversarial pass: fresh session or one of the extensions above) → `implement` → `analyze` again post-implementation.

---

## 14. Extra Tips & Things People Miss

- **Git branch creation is becoming opt-in.** Historically, `/speckit.specify` auto-created a numbered feature branch. As of recent Spec Kit versions, that behavior is being extracted into an opt-in **git extension** (`specify extension add git`). If your project stops auto-branching after an upgrade, check whether the git extension is installed/enabled rather than assuming something broke.
- **Pin your install to a release tag**, not `main`: `uv tool install specify-cli --from git+https://github.com/github/spec-kit.git@vX.Y.Z`. `main` may include unreleased changes.
- **Back up before upgrading:** `specify init --here --force` will overwrite customized `.specify/memory/constitution.md` and anything under `.specify/templates/` / `.specify/scripts/`. Copy them out first, upgrade, copy back.
- **`specify check` vs `specify self check`:** the former shows which AI agent tools are detected on your machine; the latter (read-only) tells you if your installed `specify` CLI itself is current.
- **No-Git projects still work:** set `SPECIFY_FEATURE_DIRECTORY="specs/001-my-feature"` manually before planning commands if you're not using Git at all.
- **Skills mode:** for agents that support it, `specify init --integration codex --integration-options="--skills"` installs agent *skills* instead of slash-command prompt files — same workflow, different invocation surface.
- **Parallel feature work / parallel agents:** the **Worktrees** extension spins up isolated `git worktree` checkouts per feature automatically, so multiple agents (or you switching between features) don't fight over the same working directory.
- **The `[NEEDS CLARIFICATION: ...]` marker is doing real work** — it's the mechanism that stops the agent from quietly guessing (e.g., assuming email/password auth when you never said so). Treat its presence in a generated spec as a to-do list, not noise to delete.
- **Large constitutions can bloat every planning call.** If your constitution is getting long (common in monorepos or enterprise setups), consider splitting shared/core rules from module-specific rules and only loading what's relevant — this is an active discussion in the project, not yet a built-in feature.
- **Treat `/speckit.specify` and `/speckit.plan` prompts as investments, not formalities**, especially on brownfield work — thin specs are the most common cause of avoidable rework (missing auth assumptions, missing CORS/origin context, etc. surfacing mid-implementation instead of at planning time).

# Agent: Add new headline or subheadline about GitHub integration. For example, note that the plan or Spec‑Kit creates a separate GitHub repository, so include details about GitHub and Git

# agent: Add a headline or subheadline such as “Useful links” so the links are helpful for readers unfamiliar with the topic
