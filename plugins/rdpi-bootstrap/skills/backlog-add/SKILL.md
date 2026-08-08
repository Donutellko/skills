---
name: backlog-add
description: >
  Add an item to the rdpi-bootstrap BACKLOG.md. Invoke when the user says "add to backlog",
  "backlog this", "note this for later", "track this improvement", or describes a bug/idea to record.
---

# Backlog Add

Add a new tracked item to the rdpi-bootstrap `BACKLOG.md`.

## Steps

### Step 1: Locate BACKLOG.md

Find the BACKLOG.md file. It lives next to this skill:
- Glob for `**/plugins/rdpi-bootstrap/skills/rdpi-bootstrap/BACKLOG.md` from the project root
- If not found, ask the user: "Where is BACKLOG.md?"

Read the file.

### Step 2: Parse next ID

Scan all `### B-\d+:` headers. Find the highest number. Next ID = highest + 1.
Format: `B-NNN` (zero-pad to 3 digits, e.g. `B-030`).

### Step 3: Gather item details

If the user already provided a description in their message, use it — skip asking.

Otherwise ask (one `AskUserQuestion`):
- "What should I add? Give a title and optional description."

Then ask which section to place it in (skip if obvious from context):
- **Future Improvements** (default — new ideas, enhancements)
- **Unspecified Details** — missing spec detail that needs definition
- **Unresolved from Spec** — open question blocking a decision

### Step 4: Format the entry

```
### B-NNN: <Title>

<Description paragraph(s). Concrete: what the problem is, proposed fix or approach, where in the skill to apply it.>
```

If the user gave a short one-liner, keep it as-is. Don't pad.

### Step 5: Insert into file

Append the new entry at the END of the chosen section, before the next `---` separator or end of file.

Do NOT change existing entries. Do NOT reorder anything.

### Step 6: Confirm

Output exactly: `Added B-NNN to BACKLOG.md.`

Nothing else.
