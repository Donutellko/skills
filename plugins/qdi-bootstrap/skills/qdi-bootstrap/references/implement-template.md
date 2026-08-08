---
name: qdi-implement
description: >
  Execute the implementation plan. Runs code with sub-agents, creates Draft PR, documents caveats.
  Review first slice + final PR.
---

# QDI Implement

You are running the Implement phase — executing the plan with code. You are the orchestrator: coordinate sub-agents, track progress, log caveats, and ensure each vertical slice is complete and verified.

**Tech stack:** {{TECH_STACK}}
**Build command:** {{BUILD_CMD}}
**Test command:** {{TEST_CMD}}

## Resolve Artifact Folder

The argument is a Task ID or short name. Resolve it:

1. Glob `./rdpi/*{arg}*/` to find matching folders
2. **One match** → use it
3. **Multiple matches** → list and ask the user to pick
4. **No match** → error: "No artifact folder found. Run `/qdi-questions` first."

## Numbering

Scan the entire task folder for the highest existing NN prefix across all folders and files. New artifacts use NN+1.

**Rule:** A file's number must never be lower than any existing number anywhere in the task folder.

## Inputs

Read artifacts from previous phases (find the latest of each):
- Latest `*-design/*-implementation-plan.md` — the execution plan
- Latest `*-questions/*-spec.md` — requirements
- Latest `*-design/*-design-outline.md` — architecture decisions (if exists)
- Latest `*-refinement/*-refinement.md` — change requests from prior iteration (if exists)

If the implementation plan is absent (simple task, Design was skipped), work from the Spec directly — treat acceptance criteria as your checklist.

Create: `{folder}/NN-implementation/`

## Step 1: Branch Setup

Before writing any code, check git state:

```
git branch --show-current
git status --short
```

- If **not on main**: warn the user.
- If **uncommitted changes**: warn, ask what to do using `AskUserQuestion`:
  - Commit them first `(recommended)`
  - Stash `(safe)`
  - Discard `(risky)`

Then sync and branch:

```
git fetch origin main
git checkout -b {task-id}/{short-description}
```

Never implement on main or a stale branch.

## Step 2: Execute the Plan

Work through the vertical slices in order.

For each slice:
1. **Implement** — write the code. For large slices, spawn a sub-agent (model: {{MODEL_CODERS}}) with slice instructions. For small slices, implement directly.
2. **Verify** — run verification steps (build, tests)
3. **Commit** — use the commit message from the plan
4. **Log** — append to `NN-implementation/NN-implementation-log.md`:
   ```markdown
   ## Slice N: [Name]
   - Status: complete / partial / blocked
   - Files changed: [list]
   - Tests: passing / failing / N/A
   - Caveats discovered: [anything unexpected or worth documenting]
   - User corrections: [any direction changes from the user]
   ```

**Always log discovered caveats and user corrections** — these feed the documentation and refinement steps.

### First slice — mandatory user review

After the first slice, pause and ask:

"First slice is done. Please test and confirm the direction. Check: [specific thing to verify]"

### Subsequent slices

{{#if MULTI_PHASE_DEFAULT == "multi"}}
Pause after each slice for user review.
{{else}}
Continue unless the plan marks a checkpoint or you hit an issue.
{{/if}}

If a slice fails verification: try to fix it once. If that doesn't work, log and ask the user.

## Step 3: Draft Pull Request

{{#if PR_WORKFLOW != "none"}}
After all slices are complete and verified:

1. Push the branch
2. Create a **Draft PR** with:
   - Title: task ID + short description
   - Body: summary from the design outline, list of changes, test results
   - Link to the artifact folder

"Draft PR is up at [link]. Please review the code."

Save the PR link to `NN-implementation/pr-summary.md`.
{{else}}
No PR workflow configured. Suggest reviewing with `git diff main`.
{{/if}}

{{#if DEPLOY_ENABLED}}
## Step 4: Deploy & Verify (opt-in)

Ask if the user wants to deploy for verification using `AskUserQuestion`:
- No `(recommended — review PR first)`
- Yes, deploy to staging

Always opt-in, never autonomous.
{{/if}}

## Step 5: Document Discoveries

Review the implementation log for caveats, surprising patterns, and corrections. Using `AskUserQuestion`, propose where to document each discovery:

- **Project CLAUDE.md** — for conventions/patterns affecting future work across the project `(recommended for broad caveats)`
- **Repository CLAUDE.md** — for repo-specific conventions
- **Memory** — for user preferences or workflow insights learned
- **Task folder only** — keep in implementation log `(default for minor caveats)`
- **Skip** — not worth documenting

Suggest specific text for each entry. The user confirms or edits.

## Step 6: Refinement Check

Ask using `AskUserQuestion`:

"Implementation is complete. Any refinements needed?"

Options:
- **No — we're done** `(recommended if satisfied)`
- **Yes — minor tweaks** `(describe what to change, I'll handle it now)`
- **Yes — needs another design iteration** `(will create refinement notes for /qdi-design)`

If refinements are needed:
1. Determine the next NN by scanning existing artifacts
2. Write `{folder}/NN-refinement/NN-refinement.md` with:
   - What needs to change and why
   - Reference to specific slices or decisions affected
   - User's exact feedback
3. Suggest next step:

```
Next steps (clear your context before running):
→ /qdi-design {{TASK_ID}}  (recommended — rethink the approach)
→ /qdi-implement {{TASK_ID}} (fast — implement refinements directly)
```

## Completion Summary

```
Implementation complete:
- [N] slices delivered
- [N] files created, [N] modified
- All tests passing
- {{#if PR_WORKFLOW != "none"}}Draft PR: [link]{{else}}Changes on branch: [branch-name]{{/if}}
- Caveats documented: [where]
```

## Recovery

If implementation is going sideways:
- Wrong approach → suggest `/qdi-design` to redesign
- Degraded conversation → suggest restarting. `implementation-log.md` preserves progress; completed slices are already committed.

Print any phase commands as plain text. Do NOT use the Skill tool to invoke phases.
