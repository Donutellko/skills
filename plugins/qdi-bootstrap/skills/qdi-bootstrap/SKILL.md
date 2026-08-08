---
name: qdi-bootstrap
description: >
  One-time setup: generates project-tailored QDI phase skills (Questions, Design, Implement, Work).
  Re-run with a description of desired changes to refine the generated skills.
  Trigger: user says "set up qdi", "bootstrap qdi", "configure QDI workflow", or "qdi-bootstrap".
---

# QDI Bootstrap

You are setting up the QDI development workflow for this project. QDI splits AI-assisted development into 3 phases — Questions, Design, Implement — each running in a fresh context with structured artifact handoff. This prevents context degradation and ensures quality.

Based on Dex Horthy's "Context Engineering" (QRSPI) and Dmitry Bereznitsky's "Process over Prompts," adapted into a streamlined 3-phase workflow.

## First Run vs Re-run

Check if `./rdpi/QDI_SKILLS_SPEC.md` exists.

- **Exists** → Re-run flow (jump to "Re-run" section below)
- **Doesn't exist** → First run (continue here)

## First Run — 9 Steps

### Step 0: Show introduction

**Before any tool calls or questions**, output the following introduction verbatim:

---

**Your QDI workflow:**

| Phase | Command | When |
|-------|---------|------|
| Questions | `/qdi-questions <task description or task ID>` | Start here |
| Design | `/qdi-design <task ID>` | After Questions |
| Implement | `/qdi-implement <task ID>` | After Design |
| Work | `/qdi-work <description or task ID>` | Small fixes, bugfixes |

> Clear context between each phase — open a fresh session before running the next command.
> **Work** is a single-conversation shortcut for small tasks that don't need the full pipeline.

QDI adapts QRSPI (Dex Horthy's methodology) into 3 phases: **Q**uestions → **D**esign → **I**mplement. Each phase runs one or more stages internally, with sub-agents for isolation. **Work** is a lightweight single-conversation mode for small changes.

| Phase | Sub-phase | Who | Output | User action |
|-------|-----------|-----|--------|-------------|
| **Questions** | First interview | Main ↔ user | Interview notes | Answer questions |
| | Codebase scan | Sub-agent | Scope context | — |
| | Follow-up interview | Main ↔ user | Refined understanding | Answer follow-ups |
| | Spec writing | Main agent | `NN-spec.md` | Review (optional) |
| **Design** | Research | Sub-agents | Context & findings | — |
| | Solution proposals | Main ↔ user | Multiple options | **Pick approach** |
| | Outline | Main ↔ user | `NN-design-outline.md` | **Review (mandatory)** |
| | Implementation plan | Main agent | `NN-implementation-plan.md` | Review (optional) |
| **Implement** | Branch setup | Orchestrator | Git branch | — |
| | Slice execution | Sub-agents | Code | **Test first slice** |
| | Draft PR | Sub-agent | PR link | **Code review** |
| | Documentation | Main ↔ user | CLAUDE.md / memory | Confirm |
| | Refinement | Main ↔ user | `NN-refinement.md` | Choose next step |
| **Work** | All-in-one | Main + sub-agents | `NN-work-log.md` | Go with the flow |

*After implementation, a refinement loop returns to Design or Implement with incrementing artifact numbers.*

---

Then say: "Now I'll ask you a few questions and scan the codebase to tailor these skills to your project."

### Step 1: Discover the environment

Before asking the user anything, gather context silently:

1. **Scan the codebase** — identify tech stack (`pyproject.toml`, `package.json`, `go.mod`, `Cargo.toml`, etc.), build tools, test framework, CI/CD config, existing CLAUDE.md files
2. **Discover available agents** — glob for `plugins/*/agents/*.md` and note names/descriptions
3. **Discover available skills** — check system-reminder for available skills, glob for `plugins/*/skills/*/SKILL.md` and `.claude/skills/*/SKILL.md`
4. **Check for existing project-specific skills** — look for `implement-feature`, `implement-story`, `implement-spec` or similar. Extract useful context.

### Step 2: Interview the user

Use `AskUserQuestion` for each question. **Always suggest a default answer or multiple annotated options.** Go back and forth — confirm understanding before moving on. Present auto-detected values as confirmations rather than open questions.

**3 mandatory questions:**

1. **Task types** — "What will you use QDI for?"
   - Feature implementation (recommended)
   - Bug fixing
   - Both features and bugs

2. **Model constraints** — "What's the most powerful model allowed?"
   - Opus (recommended — full power)
   - Sonnet (good balance of speed and quality)
   - Haiku (fast and cheap, quality tradeoff)
   - Follow up: "Is extended thinking allowed?" if Opus or Sonnet selected

3. **Confirmation of detected stack** — present what you found in Step 1; ask to confirm or correct: tech stack, build/test commands, conventions.

**Optional questions** — ask only if not auto-detectable:

4. Artifact folder format — default: `./rdpi/{YYYY-MM-DD}-{task-id}-{short-name}`
5. PR workflow — default: Draft PR
6. Task tracker — Jira / GitHub Issues / Linear / none
7. Diagram format — Mermaid (default) / PlantUML
8. Deploy capability — default: no autonomous deploy
9. Browser testing — default: no
10. Documentation targets — "Where should discovered caveats be documented?" Default: project CLAUDE.md + task folder.

If the user says "just use defaults", accept that and move on.

### Step 3: Auto-detect remaining values

Silently auto-detect from the codebase:
- Branch naming convention (from `git log`) — default: `feature/{ticket-id}-{short-name}`
- Commit message style (from `git log`)
- Ticket format (from CLAUDE.md or tracker config)
- Quality gates (from Makefile, CLAUDE.md, CI config)

### Step 4: Confirm ALL parameters (mandatory)

**Do NOT silently substitute defaults.**

If any setting deviates from the defaults shown in Step 0, output a concise note per deviation before the table. Skip if nothing deviates.

Present ALL parameters with provenance:

```
Here's what I've configured:

  Tech stack:      [value]                          (from [file])
  Build command:   [value]                          (from [file])
  Test command:    [value]                          (from [file])
  Task tracker:    [value]                          (user interview / default)
  PR workflow:     [value]                          (from [file] / user interview)
  Diagrams:        [value]                          (default / user choice)
  Folder format:   ./rdpi/{YYYY-MM-DD}-...          (default / user choice)
  Deploy:          [value]                          (default / from [file])
  Browser testing: [value]                          (default)
  Models:          [summary]                        (spec default / user override)
  Task types:      [value]                          (user interview)
  Documentation:   [targets]                        (default / user choice)
  Branch format:   [value]                          (auto-detected: git log)
  Commit style:    [value]                          (auto-detected: git log)
  Ticket format:   [value]                          (auto-detected)
  Quality gate:    [value]                          (auto-detected)

  Anything to change? (Enter to confirm all)
```

Wait for explicit confirmation.

### Step 5: Save the spec

Write to `./rdpi/QDI_SKILLS_SPEC.md`. Read `references/spec-template.md` and fill in all confirmed values with source provenance.

### Step 6: Propose agent assignments

Default model assignments:

| Role | Default Model | Thinking |
|------|--------------|----------|
| Questions: First interview | Opus | standard |
| Questions: Codebase scan | Sonnet | standard |
| Questions: Spec writing | Opus | extended |
| Design: Research sub-agents | Sonnet | standard |
| Design: Solution proposals | Opus | extended |
| Design: Outline & plan | Opus | extended |
| Implement: Orchestrator | Opus | standard |
| Implement: Coders | Sonnet | standard |
| Implement: PR creation | Sonnet | standard |
| Work: Main agent | Opus | standard |
| Work: Research sub-agent | Sonnet | standard |
| Work: Implement sub-agent | Sonnet | standard |

Present the mapping. If budget constraints:
- Opus limited → Sonnet + extended thinking
- Sonnet limited → Haiku (warn about quality)

### Step 7: Generate the phase skills

**Default location:** `.claude/skills/`

Read each template from `references/` and customize:

1. `references/questions-template.md` → `.claude/skills/qdi-questions/SKILL.md`
2. `references/design-template.md` → `.claude/skills/qdi-design/SKILL.md`
3. `references/implement-template.md` → `.claude/skills/qdi-implement/SKILL.md`
4. `references/work-template.md` → `.claude/skills/qdi-work/SKILL.md`
5. `references/readme-template.md` → `.claude/skills/README-QDI.md`

Replace placeholders with project-specific values:
- `{{TECH_STACK}}`, `{{BUILD_CMD}}`, `{{TEST_CMD}}`, `{{TASK_TRACKER}}`
- `{{DIAGRAM_FORMAT}}`, `{{ARTIFACT_FOLDER_FORMAT}}`
- `{{MODEL_*}}` for each role
- `{{AVAILABLE_AGENTS}}`, `{{PR_WORKFLOW}}`, `{{DEPLOY_ENABLED}}`
- `{{DOCUMENTATION_TARGETS}}`

Remove sections that don't apply.

### Step 8: Ask about global registration

"Want to use these skills in other projects too?"

- **No (default)** → Done. `.claude/skills/` is auto-discovered.
- **Yes** → Create marketplace structure, register in `~/.claude/settings.json` under `extraKnownMarketplaces` (absolute paths only).

### Step 9: Present the result

```
Created QDI workflow:
  ./rdpi/QDI_SKILLS_SPEC.md                — Your project preferences (with provenance)
  .claude/skills/qdi-questions/SKILL.md    — Questions phase
  .claude/skills/qdi-design/SKILL.md       — Design phase
  .claude/skills/qdi-implement/SKILL.md    — Implement phase
  .claude/skills/qdi-work/SKILL.md         — Lightweight single-conversation mode
  .claude/skills/README-QDI.md             — Usage guide

For medium-to-large tasks, use the full pipeline: Questions → Design → Implement.
For small fixes and bugfixes, use `/qdi-work` — same artifacts, lighter process.

If anything doesn't fit, run `/qdi-bootstrap <description>` to refine it.
```

Then repeat the commands table and add: "Easy to remember — **Q**DI. Big tasks start with **Q**: `/qdi-questions`. Quick fixes: `/qdi-work`."

---

## Re-run

When `./rdpi/QDI_SKILLS_SPEC.md` already exists:

1. Read the existing spec and summarize what's configured
2. Re-discover agents and skills
3. Ask what the user wants to update — don't re-ask everything
4. Update only the changed parts
5. Regenerate only the affected phase skills
6. Show a diff summary

---

## Numbering Scheme

QDI uses **globally monotonic numbering** across all artifacts within a task folder:

1. Each phase invocation produces artifacts with a number prefix (NN-)
2. **Both folders and files** carry the number: `NN-phase/NN-artifact.md`
3. Design produces two files sharing one number: `NN-design-outline.md` + `NN-implementation-plan.md`
4. Numbers **NEVER decrease** — scan existing folders/files for the highest NN and increment
5. When adding a file to an older folder, the file's number must be ≥ the highest existing number anywhere in the task folder
6. Standard first-run: `01-questions` → `02-design` → `03-implementation` → `04-refinement`
7. Iteration 2 via Design: `05-design` → `06-implementation` → `07-refinement`
8. Iteration 2 via direct Implement: `05-implementation` → `06-refinement`

The skill determines the next number by scanning `./rdpi/{task-folder}/` for existing numbered items.

---

## Key Principles (pass these into generated skills)

1. **Clean context between phases** — each phase runs in a fresh session. Artifacts on disk are the only state transfer.
2. **Artifacts replace compaction** — everything important is in files, not the context window.
3. **"Go back and forth with me"** — never assume, always confirm through dialogue.
4. **Always suggest defaults and options** — every `AskUserQuestion` must include a recommended default or annotated options: `(recommended)`, `(fast)`, `(risky)`, `(safe)`.
5. **Under 40 instructions per skill** — delegate to sub-agents or references if approaching the limit.
6. **No code in Design** — structured text, schemas, pseudocode when beneficial. No Python or implementation code.
7. **Vertical slicing** — end-to-end slices, not horizontal layers.
8. **Bad trajectory recovery** — if corrected 2-3 times in a row, suggest restarting the phase.
9. **Human reads code** — code review happens in the PR, not just via artifacts.
10. **Right model for the right job** — Opus for decisions, Sonnet for execution, Haiku for monitoring.
11. **Never invoke the next phase** — print the next command as plain text and stop.
12. **Document discoveries** — after implementation, propose where to document caveats: CLAUDE.md, memory, or task folder.
13. **Monotonic numbering** — artifact numbers only increase, never reuse or go below existing numbers.

---

## Reference Files

Templates are in `references/` within this skill's directory:

- `references/spec-template.md` — QDI_SKILLS_SPEC.md format
- `references/questions-template.md` — Questions phase skill template
- `references/design-template.md` — Design phase skill template
- `references/implement-template.md` — Implement phase skill template
- `references/work-template.md` — Work (lightweight) skill template
- `references/readme-template.md` — Usage guide template

Read each template when generating the corresponding skill. Do not load all templates at once.
