# QDI Bootstrap — Specification

**Date**: 2026-04-10
**Status**: Draft v1
**Authors**: Donat + Claude
**Based on**: RDPI Bootstrap (Dex Horthy "Context Engineering" / QRSPI, Dmitry Bereznitsky "Process over Prompts")
**Relation to RDPI**: QDI is a streamlined 3-phase adaptation. Removes the dedicated Plan phase; embeds planning into Design output.

---

## Terminology

| Term | Meaning |
|------|---------|
| **QRSPI** | The full methodology from Dex Horthy: Questions → Research → Design → Structure/Outline → Plan → Implement. |
| **QDI** | Our 3 top-level skills: Questions, Design, Implement. Each skill internally implements multiple QRSPI stages. |
| **Phase** | One of the 3 QDI skills. Each phase runs in a fresh Claude session. |
| **Artifact** | A structured markdown file produced by a phase, serving as the contract for the next phase. Durable state — context window is ephemeral. |
| **Vertical slice** | An end-to-end piece of functionality (not a horizontal layer). |
| **Work** | A lightweight single-conversation mode (`qdi-work`) for small changes that don't need the full pipeline. |

### Why 3 Phases, Not 4?

RDPI has 4 phases (Research, Design, Plan, Implement). QDI collapses to 3:

1. **Plan is embedded in Design.** The Design phase produces both an outline (user-facing) and an implementation plan (agent-facing). A separate Plan phase adds a context switch without proportional value — the Design agent already has the full picture.
2. **Fewer context switches.** Each phase boundary means clearing context and re-reading artifacts. 3 switches instead of 4 reduces overhead.
3. **Questions replaces Research.** The name reflects the primary activity — interviewing the user. Codebase research still happens via sub-agent, but it serves the interview, not the other way around.

### Why Add qdi-work?

Not every task needs the full pipeline. Bug fixes, small changes, and quick iterations within an existing task need the same artifact discipline (numbering, folder structure, caveats) but not the ceremony (phase breaks, mandatory reviews, separate Design).

`qdi-work` is a single-conversation minified QDI: interview → research → propose → implement → document. It writes to the same `rdpi/` folder with the same numbering scheme, so artifacts are interleaved naturally with full-pipeline work.

---

## Differences from RDPI

| Aspect | RDPI | QDI |
|--------|------|-----|
| Phases | 4 (Research, Design, Plan, Implement) | 3 (Questions, Design, Implement) + Work |
| Planning | Dedicated Plan phase with spec-consistency sub-agent | Plan embedded in Design output |
| Research approach | Blind (sub-agent doesn't know the task) | Task-aware (sub-agent knows the task context) |
| Interview structure | Single round | Two rounds (general → codebase scan → follow-up) |
| Design output | design.md + structure-outline.md | design-outline.md + implementation-plan.md |
| Artifact numbering | Per-phase folders (01-research, 02-design, etc.) | Globally monotonic NN, both folders and files |
| Refinement loop | Not formalized | Explicit: refinement → Design or Implement, numbers increment |
| Documentation step | Not formalized | Explicit: propose CLAUDE.md/memory/folder after implementation |
| Small tasks | Skip phases via complexity recommendation | Dedicated `qdi-work` skill |
| Code in Design | Not explicitly prohibited | Explicitly prohibited (structured text + pseudocode only) |

### Why Task-Aware Research (Not Blind)?

RDPI uses blind research to prevent confirmation bias: the sub-agent gets only questions, not the task. QDI makes a different trade-off:

- **In Questions phase**: the codebase scan sub-agent is task-aware because its purpose is to scope the affected areas and find relevant patterns — not to form opinions about solutions. Bias risk is low because it reports facts, and there's a follow-up interview round to challenge assumptions.
- **In Design phase**: research sub-agents are task-aware because they need to evaluate specific technical constraints to inform solution proposals. Blind research at this stage would miss context-dependent trade-offs.

The bias mitigation shifts from structural isolation (blind sub-agent) to explicit constraints in the sub-agent prompt ("report facts only, no recommendations").

---

## Core Principles

### 1. Clean Context Between Phases

Each QDI phase runs in a fresh Claude session. Artifacts on disk are the only state transfer. This prevents the "dumb zone" problem (context degradation at ~40% window fill).

**Exception:** `qdi-work` runs in a single conversation by design — it's for small tasks where context degradation isn't a concern.

### 2. Artifacts Replace Compaction

Every phase produces structured markdown artifacts on disk. The context window is ephemeral and disposable. You can always resume from where you left off by reading the artifacts.

### 3. Globally Monotonic Numbering

All artifacts within a task folder share a single incrementing counter. Rules:

1. Both folders and files carry the NN prefix: `NN-phase/NN-artifact.md`
2. Design produces two files sharing one number (outline + plan)
3. Numbers NEVER decrease — scan for the highest NN and increment
4. Adding a file to an older folder: the file's number must be ≥ the highest existing number anywhere in the task folder
5. `qdi-work` follows the same numbering — its artifacts interleave with full-pipeline artifacts

**Rationale:** Global numbering creates a clear timeline across iterations. When reviewing a task folder, the numbers tell you the order things happened regardless of which skill produced them.

### 4. Always Suggest Defaults and Options

Every `AskUserQuestion` must include:
- A recommended default marked `(recommended)`
- Other options annotated: `(fast)`, `(risky)`, `(safe)`, `(innovative)`, `(compromise)`
- Enough context for the user to decide without reading external docs

This applies to all QDI skills including `qdi-work`.

### 5. No Code in Design

The Design phase produces structured text, schemas, diagrams, and pseudocode (when beneficial). No Python, TypeScript, or other implementation code. This forces the Design to stay at the right abstraction level and prevents premature implementation decisions.

The implementation plan directs the implementing agent to specific files and describes what to change — but in prose and schemas, not code.

### 6. Two-Round Interview

The Questions phase uses two interview rounds separated by a codebase scan:

1. **Round 1:** General understanding — expectations, requirements, constraints
2. **Codebase scan sub-agent:** Task-aware exploration of affected areas
3. **Round 2:** Follow-up questions driven by scan findings

This surfaces relevant constraints early (before Design) without asking the user to pre-research the codebase themselves. Round 2 questions are more targeted because they're grounded in actual code findings.

### 7. No Implementation Questions in Questions Phase

Questions phase focuses on **what** and **why** — never **how**. Implementation approach decisions belong in Design where research context is available. This separation prevents premature commitment to a solution before the problem is fully understood.

### 8. Refinement Loop

After implementation, the user can trigger another iteration:
- **Via Design:** Creates a refinement note, then full Design → Implement cycle with incrementing numbers
- **Via direct Implement:** For minor adjustments that don't need redesign
- **Via qdi-work:** For quick follow-up fixes

The numbering continues across iterations, creating an audit trail.

### 9. Document Discoveries

After implementation, the skill proposes where to document caveats and surprising findings:
- Project CLAUDE.md — broad conventions
- Repository CLAUDE.md — repo-specific patterns
- Memory — user preferences and workflow insights
- Task folder — minor, task-specific notes

This turns implementation experience into reusable knowledge instead of letting it evaporate with the context window.

### 10. Right Model for the Right Job

| Role | Model | Thinking | Rationale |
|------|-------|----------|-----------|
| Questions: First interview | Opus | standard | Interview needs depth |
| Questions: Codebase scan | Sonnet | standard | Factual code exploration |
| Questions: Spec writing | Opus | extended | Synthesis of interview + research |
| Design: Research sub-agents | Sonnet | standard | Factual exploration |
| Design: Solution proposals | Opus | extended | Architecture decisions |
| Design: Outline & plan | Opus | extended | Key artifact, shapes all implementation |
| Implement: Orchestrator | Opus | standard | Coordination |
| Implement: Coders | Sonnet | standard | Mechanical code from plan |
| Implement: PR creation | Sonnet | standard | Straightforward |
| Work: Main agent | Opus | standard | Needs judgment for flexible flow |
| Work: Research sub-agent | Sonnet | standard | Quick codebase scan |
| Work: Implement sub-agent | Sonnet | standard | Code from decisions |

### 11. Inherited Principles (from RDPI)

These carry over unchanged:
- **"Go back and forth with me"** — never assume, confirm through dialogue
- **Instruction budget** — under 40 instructions per skill
- **Vertical slicing** — end-to-end slices, not horizontal layers
- **Bad trajectory recovery** — if corrected 2-3 times, suggest restart
- **Human reads code** — code review happens in the PR
- **Never invoke the next phase** — print command as plain text, user runs it

---

## QDI Phases

### Phase 1: Questions (`/qdi-questions`)

**Goal:** Understand the task through two-round interview, produce a Spec.

**Steps:**
1. First interview — general idea, expectations, requirements (no implementation questions)
2. Codebase scan — task-aware sub-agent explores affected areas
3. Follow-up interview — questions driven by scan findings
4. Write spec — requirements + codebase context summary + complexity assessment

**Output:** `NN-questions/NN-spec.md`, `NN-questions/codebase-context.md`

### Phase 2: Design (`/qdi-design`)

**Goal:** Research, propose solutions, produce outline + implementation plan.

**Steps:**
1. Research — task-aware sub-agents (1 standard, 2 parallel for complex tasks)
2. Propose 2-4 solutions with pros/cons/effort/recommendation tags
3. User picks approach; spec updated if gaps found
4. Prepare high-level outline (user reviews — mandatory checkpoint)
5. Write implementation plan (directions, schemas, pseudocode — no code)

**Output:** `NN-design/NN-design-outline.md`, `NN-design/NN-implementation-plan.md`, `NN-design/research-context.md`

### Phase 3: Implement (`/qdi-implement`)

**Goal:** Execute the plan, document discoveries, handle refinements.

**Steps:**
1. Branch setup
2. Execute vertical slices (sub-agents for large slices)
3. Draft PR
4. Document discoveries (propose CLAUDE.md/memory/folder targets)
5. Refinement check (done / minor tweaks / needs redesign)

**Output:** `NN-implementation/NN-implementation-log.md`, `NN-implementation/pr-summary.md`, optionally `NN-refinement/NN-refinement.md`

### Lightweight: Work (`/qdi-work`)

**Goal:** Small changes and bugfixes in one conversation, same artifact discipline.

**Steps (flexible, not rigid):**
1. Context loading — read existing rdpi/ artifacts if relevant
2. Light interview — 2-4 questions max
3. Quick research — single sub-agent
4. Propose 2-3 options inline
5. Implement (directly or via sub-agent)
6. Log to work-log or append to existing implementation-log
7. PR if appropriate
8. Document caveats if non-trivial

**Output:** `NN-work/NN-work-log.md` (new task) or appends to existing `NN-implementation-log.md` (continuation)

---

## Artifact Directory Structure

```
./rdpi/{YYYY-MM-DD}-{task-id}-{short-name}/
├── 01-questions/
│   ├── 01-spec.md                    — Requirements (USER REVIEW: optional)
│   └── codebase-context.md           — Sub-agent findings
├── 02-design/
│   ├── 02-design-outline.md          — Approved outline (USER REVIEW: mandatory)
│   ├── 02-implementation-plan.md     — Plan for agent (USER REVIEW: optional)
│   └── research-context.md           — Research findings
├── 03-implementation/
│   ├── 03-implementation-log.md      — Progress + caveats + corrections
│   └── pr-summary.md                 — PR link
├── 04-refinement/
│   └── 04-refinement.md              — Change requests
├── 05-design/                        — Iteration 2
│   ├── 05-design-outline.md
│   └── 05-implementation-plan.md
├── 06-implementation/
│   └── 06-implementation-log.md
├── 07-work/                          — Quick fix via qdi-work
│   └── 07-work-log.md
└── ...                               — Numbers keep increasing
```

---

## What qdi-bootstrap Produces

### Output

```
.claude/skills/
├── qdi-questions/SKILL.md     — Questions phase
├── qdi-design/SKILL.md        — Design phase
├── qdi-implement/SKILL.md     — Implement phase
├── qdi-work/SKILL.md          — Lightweight single-conversation mode
└── README-QDI.md              — Usage guide

./rdpi/
└── QDI_SKILLS_SPEC.md         — Project preferences with provenance
```

---

## Scope

### In Scope
- Feature implementation workflow (Questions → Design → Implement)
- Bug fixing via full pipeline or lightweight `qdi-work`
- Refinement loop with incrementing artifact numbers
- Post-implementation documentation proposals
- Generating project-specific QDI skills
- Runtime discovery and assignment of available agents

### Out of Scope
- Refactoring-only workflows
- Migration workflows
- Incident investigation
- Running the QDI phases itself (that's what the generated skills do)

---

## Dependencies

### Required
- `AskUserQuestion` tool — for interview and interactive design
- `Agent` tool — for sub-agents (codebase scan, research, implementation)
- File system tools (Read, Write, Glob, Grep) — for codebase analysis and skill generation

### Optional (discovered at runtime)
- context7 MCP — for library/framework documentation lookup
- Deployment/CI tools
- Browser testing capabilities
