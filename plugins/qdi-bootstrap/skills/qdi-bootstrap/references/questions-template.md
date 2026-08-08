---
name: qdi-questions
description: >
  Start your task. Describe the task or paste a ticket ID. Two rounds of interview produce: spec.md.
  Trigger: user describes a feature/bug or pastes a ticket ID.
---

# QDI Questions

You are running the Questions phase — the first step in the QDI workflow. Your goal is to deeply understand the task through two rounds of interview and produce a Spec that anchors all subsequent work.

**Tech stack:** {{TECH_STACK}}
**Task tracker:** {{TASK_TRACKER}}

## Inputs

The user provides a task description (feature or bug) and optionally a task ID from {{TASK_TRACKER}}.

Create the artifact directory: `{{ARTIFACT_FOLDER_FORMAT}}/01-questions/`

## Numbering

Scan the task folder for the highest existing NN prefix. For the first run this is 01. Both the folder and files inside use this number.

## Step 1: First Interview — General Idea

Go back and forth with the user using `AskUserQuestion` to understand the task at a high level. **Always suggest default answers or annotated options.**

Focus on **expectations and requirements only** — no implementation questions.

For **features**, clarify:
- What problem does this solve? Who benefits?
- User stories and acceptance criteria
- Constraints, edge cases, out-of-scope items
- Priority and desired outcome

For **bugs**, clarify:
- Reproduction steps (exact sequence)
- Expected vs actual behavior
- When it started (if known), severity and impact

Challenge assumptions. If something seems underspecified, ask. Do NOT ask about implementation approach — that belongs to the Design phase.

## Step 2: Codebase Scan — Sub-agent

Spawn a **task-aware** sub-agent to build a picture of scope and context. This sub-agent knows the task description.

```
Agent prompt:
"You are a codebase researcher exploring the system for a specific task.

Task context:
<paste summary of what you learned in Step 1>

Explore the codebase to understand:
1. Which areas of the codebase are relevant to this task?
2. What existing patterns, conventions, and abstractions exist in the affected areas?
3. What are the dependencies and data flows?
4. Are there similar features or past changes that set a precedent?
5. What is the scope of change likely to be?

Report facts with file paths and code references. Note anything unexpected.

Save findings to: <path>/NN-questions/codebase-context.md"
```

Use model: {{MODEL_QUESTIONS_SCAN}}

When the sub-agent finishes, read `codebase-context.md` to verify it contains useful findings. If thin or missed key areas, ask targeted follow-ups.

## Step 3: Follow-up Interview

Read the sub-agent's findings. Ask the user follow-up questions based on what was discovered:

- Confirm scope based on affected areas found
- Clarify edge cases revealed by existing patterns
- Resolve ambiguities surfaced by the codebase scan
- Ask about non-functional requirements if the scan revealed relevant constraints

Keep it focused — only ask what the codebase scan made relevant. Still no implementation questions.

## Step 4: Write the Spec

With the user's requirements (Steps 1 & 3) and codebase context (Step 2), write the Spec.

Write to `NN-questions/NN-spec.md`:

```markdown
# Spec: [Task Title]

## Task
[One-paragraph summary]

## Type
Feature / Bug fix

## Requirements
### Functional
- [Requirement 1]
- [Requirement 2]

### Non-functional
- [Performance, security, accessibility constraints]

## Acceptance Criteria
- [ ] [Criterion 1 — testable and specific]
- [ ] [Criterion 2]

## Constraints
- [Technical, timeline, or scope constraints]

## Out of Scope
- [What this task explicitly does NOT include]

## Codebase Context Summary
[Key findings about affected areas, patterns, and scope]

## Complexity Assessment
[Simple / Medium / Complex — with reasoning]
```

Tell the user: "I've written the Spec. You can review it at `NN-questions/NN-spec.md`, or we can proceed to Design."

## Next Step

Always recommend Design as the default next step:

```
Next step (clear your context before running):
→ /qdi-design {{TASK_ID}}  (recommended)
→ /qdi-implement {{TASK_ID}} (fast — skip Design, only for clearly simple tasks)
```

Print commands as plain text. Do NOT use the Skill tool to invoke the next phase. The user runs it in a fresh session.

## Recovery

If the conversation has gone sideways (user corrected you 2-3+ times in a row), suggest: "It seems like we're not aligned. Want to restart this phase with a fresh context?"
