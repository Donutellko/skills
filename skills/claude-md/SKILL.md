---
description: Create, update, or analyze CLAUDE.md configuration files
argument-hint: "[project|user|main|global|<path>] [create|update|analyze]"
allowed-tools: Read, Write, Edit, Glob, Grep, Task, TodoWrite
---

# CLAUDE.md Management

Create, update, or analyze CLAUDE.md configuration files. Supports both user global (~/.claude/CLAUDE.md) and project-specific files.

## Usage

```
/claude-md [scope] [operation]
```

**Scope options:**
- `project` (default): Project CLAUDE.md in current working directory
- `user`, `main`, `global`: User's global ~/.claude/CLAUDE.md
- `<path>`: Explicit path to a CLAUDE.md file

**Operation options:**
- `analyze` (default): Review and identify issues
- `update`: Modify with improvements
- `create`: Generate new file from scratch

## Anti-Hallucination Policy

**CRITICAL**: This skill delegates analysis to the claude-md-expert agent. All analysis, updates, and creation operations MUST be grounded in actual file content and explicit user requirements.

### Mandatory Rules

1. **No Issue Fabrication**: Only report inconsistencies, unclear directives, or missing scenarios that are EVIDENT from actual file content
2. **No Content Invention**: When creating/updating files, only include directives explicitly requested by user or derived from read project files
3. **Read-Before-Analyze**: Agent MUST read the target CLAUDE.md file completely before making any claims about its content
4. **Quote-Based Issues**: Every issue reported must reference specific line numbers and quote actual problematic text
5. **No Assumption-Based Changes**: Updates must address actual problems in current file, not theoretical improvements
6. **Conservative Assessment**: When directives are ambiguous, flag them as unclear rather than assuming intent

### Red Flags for Over-Analysis (Agent Should Avoid)

**Fabricated Issues** (Not Acceptable):
- Claiming "conflicting directives" without quoting the actual conflict
- Reporting "vague terms" without showing context where they cause ambiguity
- Asserting "missing scenarios" without evidence the scenarios are relevant to project
- Inventing "inconsistencies" based on assumed standards not in the file

**Acceptable Issue Reporting**:
- "Lines 42-45 conflict: Line 42 says 'use tabs' but line 45 says 'use 4-space indentation'"
- "Line 67 'optimize for performance' is unclear - no metrics or thresholds defined"
- "Missing error handling guidance evident from lines 30-35 which mention 'quality standards' but don't specify"

### Create Operation Requirements

When creating new CLAUDE.md files, the agent must:
1. **Ask User for Content**: Do not invent personal preferences, coding standards, or project context
2. **Read Project Files**: For project CLAUDE.md, read actual project files (package.json, pyproject.toml, README.md) to derive real context
3. **No Generic Templates**: Avoid filling sections with placeholder text like "follow best practices" or "use proper naming"
4. **Explicit Gaps**: If user hasn't provided required information, ask for it rather than inventing defaults

### Update Operation Requirements

When updating CLAUDE.md files, the agent must:
1. **Read Current File First**: Never assume what's in the file based on typical patterns
2. **User-Directed Changes Only**: Only make changes explicitly requested or clearly fixing identified issues
3. **Preserve Intent**: Do not rewrite sections that are already clear and correct
4. **Quote Before/After**: Show actual current text and proposed new text for all changes

### Analysis Discipline

**Before reporting issues, the agent must verify**:
- [ ] Issue is backed by specific line numbers and quoted text from actual file
- [ ] Issue represents an actual problem (ambiguity, conflict, incompleteness) not a stylistic preference
- [ ] Severity assessment (Critical/Major/Minor) is justified by impact on Claude's ability to follow the configuration
- [ ] Suggested fix is specific and actionable, not generic advice

**If the agent cannot verify these with quoted content from the read file, do not report the issue**

### Reporting Requirements

When the skill reports results to the user:
1. **Quote Actual Issues**: Show line numbers and actual text from the agent's analysis
2. **No Summary Inflation**: Report actual issue counts from agent output, not rounded estimates
3. **Specific Examples**: Show top issues with their actual problematic text, not paraphrased summaries
4. **No Invented Assessment**: Overall assessment must reflect actual issues found, not assumed quality problems

## Step 1: Parse Arguments

Determine scope and operation from: `$ARGUMENTS`

### Scope Resolution

| Input | Resolves To |
|-------|-------------|
| (empty) | `{cwd}/CLAUDE.md` |
| `project` | `{cwd}/CLAUDE.md` |
| `user` | `~/.claude/CLAUDE.md` |
| `main` | `~/.claude/CLAUDE.md` |
| `global` | `~/.claude/CLAUDE.md` |
| `/path/to/file` | Exact path |
| `~/path/to/file` | Expanded path |

### Operation Detection

If second argument is provided, use it. Otherwise:
- If file exists -> `analyze`
- If file doesn't exist -> `create`

If user says "update", "fix", "improve", "change" -> `update`
If user says "create", "new", "generate" -> `create`
If user says "analyze", "review", "check", "audit" -> `analyze`

## Step 2: Execute Operation

### For Analyze

Launch the claude-md-expert agent:

```
Use the Task tool with subagent_type: "claude-md-expert"

Analyze the CLAUDE.md configuration file at: {resolved_path}

This is a {"user global" | "project"} configuration file.

**ANTI-HALLUCINATION REQUIREMENTS**:
- Read the entire file first before making any claims
- Quote specific line numbers for every issue identified
- Only report issues evident from actual file content
- Do not invent problems based on assumed standards
- Severity must be justified by actual impact on usability

Perform a thorough analysis focusing on:

1. **Internal Consistency**
   - Does the file follow its own rules?
   - Are there conflicting directives?
   - Quote the conflicting sections with line numbers

2. **Clarity**
   - Are directives specific and actionable?
   - Are conditional scopes explicit?
   - Are vague terms defined?
   - Quote unclear directives with line numbers

3. **Completeness**
   - Are common scenarios covered?
   - Are enforcement mechanisms mentioned?
   - Only flag missing scenarios if they're clearly relevant to the project context

4. **Appropriateness**
   - Is content in the right file (user vs project)?
   - Is reference documentation clearly marked?
   - Quote misplaced content with line numbers

IMPORTANT: Treat this as a CONFIGURATION FILE, not an agent definition.
Do NOT flag missing "When Invoked", "Focus Areas", or "Success Criteria" sections.

Save analysis to {cwd}/PLAN.md with:
- Issue counts by severity (based on actual issues found)
- Specific issues with quoted current text (with line numbers) and proposed fixes
- Implementation checklist
```

### For Update

Launch the claude-md-expert agent:

```
Use the Task tool with subagent_type: "claude-md-expert"

Update the CLAUDE.md configuration file at: {resolved_path}

This is a {"user global" | "project"} configuration file.

User requested changes: {any additional context from user}

**ANTI-HALLUCINATION REQUIREMENTS**:
- Read the entire file first - do not assume content
- Only change what the user explicitly requested or what is clearly broken
- Quote current text before making edits (with line numbers)
- Do not invent problems to fix
- Do not rewrite clear sections unnecessarily

Steps:
1. Read the entire file to understand current structure
2. Identify what needs to be changed (user request or evident problem only)
3. Make targeted edits that:
   - Maintain consistency with existing style
   - Don't introduce conflicts
   - Keep appropriate scope (user vs project)
   - Address actual problems, not theoretical improvements
4. Summarize what was changed with before/after quotes and justification

If updating user global file, note any changes that should also be synced to project files (or vice versa).
```

### For Create

Launch the claude-md-expert agent:

```
Use the Task tool with subagent_type: "claude-md-expert"

Create a new CLAUDE.md configuration file at: {resolved_path}

This will be a {"user global" | "project"} configuration file.

**ANTI-HALLUCINATION REQUIREMENTS**:
- Do NOT invent personal preferences, coding standards, or directives
- Ask user for input on tone, style, and preferences before creating content
- For project files, read actual project files (package.json, pyproject.toml, README.md, etc.) to derive context
- Do not use generic placeholder text like "follow best practices"
- If critical information is missing, ask user rather than guessing

{"User global" guidelines:
- Ask user for personal preferences (tone, verbosity) - do not invent defaults
- Ask user for coding standards they want enforced
- Include language-specific standards only if relevant to user's work
- Include output formatting requirements specified by user
- Do NOT include project-specific context}

{"Project" guidelines:
- Read actual project files to determine project type, language, structure
- Include project coding standards (may sync with user global if user confirms)
- Include language-specific standards relevant to this project (derived from actual project files)
- Include output formatting requirements (ask user or sync from user global)
- Include project context maintenance directive
- Add Project Context section with REAL data:
  - Project overview (from README.md or ask user)
  - Directory structure (from actual directory listing)
  - Setup instructions (from existing docs or ask user)
  - Relevant reference documentation (from actual files, not invented)}

Before creating, check if the file already exists. If it does, ask user to confirm overwrite or switch to update mode.

After creating, display the key sections to the user for review and ask for confirmation before writing.
```

## Step 3: Report Results

After the agent completes:

### For Analyze
1. Read the generated PLAN.md
2. **VERIFY** the analysis followed anti-hallucination policy:
   - Check that issues quote actual line numbers and text
   - Verify issues represent real problems, not stylistic preferences
   - Confirm severity is justified by actual impact
3. Summarize (using actual data from PLAN.md):
   - Issue counts (exact numbers: Critical/Major/Minor)
   - Top 3-5 issues with their quoted line numbers
   - Overall assessment based on actual findings
4. Ask if user wants to implement fixes

### For Update
1. **VERIFY** the update followed anti-hallucination policy:
   - Confirm agent read the file before making changes
   - Check that changes address user request or actual problems
   - Verify agent quoted before/after text
2. Summarize changes made (with actual before/after quotes)
3. Note any sync needed between user global and project files
4. Show diff or key changed sections

### For Create
1. **VERIFY** the creation followed anti-hallucination policy:
   - Confirm agent asked user for preferences/standards (not invented)
   - For project files, verify agent read actual project files
   - Check content is specific, not generic placeholders
2. Confirm file was created
3. Show structure overview with key sections
4. Note any follow-up needed (e.g., "update project context with /claude-md project update")

## CRITICAL

- **CONFIGURATION FILE** - Never treat CLAUDE.md as an agent definition
- **SCOPE AWARENESS** - User global is personal; project is shareable
- **NO AGENT SECTIONS** - Don't add "When Invoked", "Focus Areas", etc.
- **SYNC AWARENESS** - Note when changes should propagate between files
- **SAVE TO ROOT** - Analysis goes to {cwd}/PLAN.md, not .claude/
- **ANTI-HALLUCINATION** - All delegated agent operations must follow the Anti-Hallucination Policy above - verify agent output is evidence-based before reporting to user

## Quick Reference

### User Global (~/.claude/CLAUDE.md)
Contains:
- Personal tone/personality preferences
- Universal coding standards
- Language-specific standards
- Output formatting rules

### Project ({cwd}/CLAUDE.md)
Contains:
- Project coding standards
- Language-specific rules for this project
- Output formatting rules
- Context maintenance directive
- Project reference documentation (auto-generated)

### Key Differences
| Aspect | User Global | Project |
|--------|-------------|---------|
| Personality/tone | Yes | No |
| Coding standards | Universal | Project-specific |
| Project context | No | Yes |
| Team shareable | No | Yes |
