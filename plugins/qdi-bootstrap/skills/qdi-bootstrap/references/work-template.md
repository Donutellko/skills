---
name: qdi-work
description: >
  Quick fixes and small changes in one conversation. Same artifact discipline as full QDI, lighter process.
  Trigger: small bugfix, minor change, quick follow-up to an existing task. Use when full QDI is overkill.
---

# QDI Work

You are running a lightweight single-conversation workflow for small changes and bugfixes. Same artifact discipline as full QDI (numbering, rdpi/ folder, caveats), but no phase breaks — go with the flow.

**Tech stack:** {{TECH_STACK}}
**Build command:** {{BUILD_CMD}}
**Test command:** {{TEST_CMD}}

## Context Loading

First, understand the context:

1. Check if the user referenced an existing task ID or name
2. If yes — glob `./rdpi/*{arg}*/` to find the task folder, read key artifacts:
   - Latest `*-spec.md` for requirements context
   - Latest `*-implementation-log.md` for what's been done
   - Latest `*-refinement.md` for pending change requests (if any)
3. If no existing task — create a new folder: `{{ARTIFACT_FOLDER_FORMAT}}`

## Numbering

Scan the task folder for the highest existing NN prefix. New artifacts use NN+1.

**Rule:** A file's number must never be lower than any existing number anywhere in the task folder.

## Decide: Continue or New

If working within an existing task folder, determine whether this is a **direct continuation** of the latest implementation or a **separate piece of work**:

- **Direct continuation** (same area, related fix) → append to the latest `NN-implementation-log.md`
- **Separate work** (different concern, new fix) → create `NN-work/NN-work-log.md`

If unclear, ask using `AskUserQuestion`:
- Append to existing log `(recommended if it's a follow-up)`
- Create new work log `(recommended if it's a different concern)`

## Flow

There is no rigid step sequence. Adapt to the conversation. But the general shape is:

### 1. Understand

Ask 2-4 focused questions using `AskUserQuestion`. **Always suggest defaults and annotated options.**

If the user already explained everything in the invocation message, skip straight to research. Don't ask questions you can answer from the existing artifacts or codebase.

### 2. Research

Spawn a quick research sub-agent (model: {{MODEL_WORK_RESEARCH}}):

```
Agent prompt:
"You are a codebase researcher investigating a specific issue.

Context:
<paste what you know — task description, relevant artifacts, user's answers>

Research:
1. Find the relevant code paths
2. Identify the root cause or affected area
3. Note existing patterns to follow
4. Check for related tests

Report facts with file paths. Keep it concise — this is a small task.

Save to: <path>/NN-work/research-notes.md (or inline if appending to existing log)"
```

If the task is trivial (typo fix, config change, obvious one-liner), skip the sub-agent entirely.

### 3. Propose

Present **2-3 options** inline (not a separate file). For each:
- One-line approach description
- Key trade-off or risk
- Recommendation tag: `(recommended)`, `(fast)`, `(safe)`, `(risky)`

Use `AskUserQuestion` for the user to pick. If there's only one obvious approach, present it as the recommendation and ask for confirmation.

### 4. Implement

For small changes — implement directly.
For larger changes — spawn an implementation sub-agent (model: {{MODEL_WORK_IMPLEMENT}}):

```
Agent prompt:
"Implement the following change:
<paste the selected approach + relevant context>

Tech stack: {{TECH_STACK}}
Build: {{BUILD_CMD}}
Test: {{TEST_CMD}}

After implementing:
1. Run the build command
2. Run relevant tests
3. Report what was changed and test results"
```

Verify the result (build + tests). Commit with a descriptive message.

### 5. Log

Write to the work log (either new `NN-work/NN-work-log.md` or appending to existing implementation log):

```markdown
## Work: [Short Description]

**Date:** [date]
**Type:** bugfix / small change / follow-up
**Related to:** [existing task reference, if any]

### What was asked
[Brief summary of the request]

### What was decided
[Selected approach and why]

### What was done
- Files changed: [list]
- Tests: passing / added / N/A

### Caveats
[Anything unexpected or worth noting — skip if none]
```

### 6. PR (if appropriate)

{{#if PR_WORKFLOW != "none"}}
Ask using `AskUserQuestion`:

"Want me to create a PR for this?"
- Yes, Draft PR `(recommended for non-trivial changes)`
- No, I'll handle it `(fine for tiny fixes)`

If yes — push branch, create Draft PR with summary from the work log.
{{/if}}

### 7. Document (if non-trivial)

If caveats were discovered, briefly propose where to document them using `AskUserQuestion`:
- Project CLAUDE.md `(recommended for broad patterns)`
- Task folder only `(default for minor notes)`
- Skip

Only ask this if something genuinely surprising or reusable was found. Don't ask for routine fixes.

## Recovery

If the change turns out to be bigger than expected, suggest switching to the full pipeline:

"This looks bigger than a quick fix. Want to switch to the full QDI pipeline?"
- `/qdi-questions {{TASK_ID}}` `(recommended — start fresh with proper scoping)`
- Continue here anyway `(risky — context may degrade)`

## Tone

Keep it conversational and efficient. No ceremony for ceremony's sake. The user chose `qdi-work` because they want to move fast — respect that while maintaining artifact discipline.
