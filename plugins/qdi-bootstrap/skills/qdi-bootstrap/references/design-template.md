---
name: qdi-design
description: >
  Propose solutions, pick an approach, produce outline + implementation plan. Review the outline before proceeding.
---

# QDI Design

You are running the Design phase — researching the problem space, proposing solutions, and producing an implementation plan. **No implementation code in this phase** — only structured text, schemas, and pseudocode when beneficial.

**Tech stack:** {{TECH_STACK}}
**Diagram format:** {{DIAGRAM_FORMAT}}

## Resolve Artifact Folder

The argument is a Task ID or short name. Resolve it:

1. Glob `./rdpi/*{arg}*/` to find matching folders
2. **One match** → use it
3. **Multiple matches** → list and ask the user to pick
4. **No match** → error: "No artifact folder found. Run `/qdi-questions` first."

## Numbering

Scan the entire task folder for the highest existing NN prefix across all folders and files. The next artifacts use NN+1. Design produces two files sharing the same number: `NN-design-outline.md` and `NN-implementation-plan.md`.

**Rule:** A file's number must never be lower than any existing number anywhere in the task folder.

## Inputs

Read artifacts from the Questions phase:
- Latest `*-questions/*-spec.md` — requirements
- Latest `*-questions/codebase-context.md` — scope context (if exists)

If a previous refinement exists (e.g., `*-refinement/*-refinement.md`), also read it — it contains change requests driving this iteration.

If a previous design iteration exists, read its outline and plan for context on what was already tried.

Create: `{folder}/NN-design/`

## Step 1: Research — Sub-agents

Launch **task-aware** research sub-agents to build the context needed for proposing solutions. These sub-agents know the task and spec.

**Standard (1 sub-agent):**

```
Agent prompt:
"You are a technical researcher preparing context for a design decision.

Task spec:
<paste spec content>

Research:
1. How does the current system handle related functionality?
2. What patterns and abstractions should a solution follow?
3. What are the constraints (performance, API contracts, data schemas)?
4. Check documentation (use context7 MCP if available) for relevant library/framework guidance
5. Are there prior approaches in the codebase that set precedent?
6. What test patterns exist for similar functionality?

Report findings with file paths and references. No recommendations — facts only.

Save to: <path>/NN-design/research-context.md"
```

**Complex features (2 parallel sub-agents):**

When the spec indicates high complexity, launch two sub-agents with different focus areas:

- **Sub-agent A:** Focus on the existing architecture — how to extend what's already there
- **Sub-agent B:** Focus on ideal patterns — what a clean-slate approach would look like

Compare their findings before proposing solutions. This surfaces trade-offs between pragmatism and ideal design.

Use model: {{MODEL_DESIGN_RESEARCH}}

## Step 2: Propose Solutions

Based on research findings and spec, propose **2-4 solutions** to the user. For each solution:

- **Name** — short descriptive label
- **Approach** — how it works (structured text, diagrams, schemas — **no implementation code**)
- **Benefits** — what's good about this approach
- **Downsides** — trade-offs, risks, limitations
- **Effort estimate** — relative: small / medium / large
- **Recommendation tag** — `(recommended)`, `(safe)`, `(fast)`, `(risky)`, `(innovative)`

Use `AskUserQuestion` with the solutions as options. Let the user pick or propose modifications.

If the user's feedback reveals **gaps in the spec**, update the spec file directly (read, modify, write back). Tell the user: "I've updated the spec to reflect [what changed]."

Confirm the selected approach before proceeding.

## Step 3: Prepare Outline

Create a high-level outline that is **easily readable by a person**. This is the key synchronization point — the user must understand and approve this.

The outline contains:
- Selected approach summary (1-2 paragraphs)
- Vertical slices — end-to-end increments, not horizontal layers
- For each slice: what it delivers, what it touches, how to verify
- Delivery order — start with what the user can see and give feedback on
- Diagrams in {{DIAGRAM_FORMAT}} where helpful
- **No implementation code** — structured text and schemas only
- Pseudocode only when it makes the explanation clearer than prose

Present the outline to the user using `AskUserQuestion`:

"Here's the outline. Please review — this is the most important checkpoint before implementation."

Options:
- Approve as-is `(recommended if you're satisfied)`
- Request changes `(describe what to adjust)`
- Restart with a different approach

Iterate until approved.

## Step 4: Save Artifacts

Once the outline is approved:

1. Save `{folder}/NN-design/NN-design-outline.md` — the approved outline
2. Write `{folder}/NN-design/NN-implementation-plan.md` — a structured plan for the implementing agent

The **implementation plan** includes:
- Vertical slice breakdown with delivery order
- For each slice: affected files/modules, direction for changes, verification steps
- Commit message suggestions
- Checkpoints where user review is needed
- Known risks and edge cases
- Schemas and data flow descriptions where relevant
- **No implementation code** — directions, schemas, pseudocode only

## Next Step

```
Next step (clear your context before running):
→ /qdi-implement {{TASK_ID}}
```

Print as plain text. Do NOT invoke the next phase via the Skill tool. The user runs it in a fresh session.

## Recovery

If the conversation has gone sideways (user corrected you 2-3+ times), suggest restarting this phase. Research context on disk is preserved — a fresh context will read it and continue.
