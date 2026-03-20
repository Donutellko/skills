---
description: Analyze prompts and agents to inject anti-hallucination directives where appropriate
argument-hint: "[file|folder]"
allowed-tools: Read, Write, Edit, Glob, Grep, Task, TodoWrite
ultrathink: true
---

# Anti-Hallucination

Analyze prompts, agent definitions, or command files to inject anti-hallucination safeguards. Identifies areas prone to fabrication and adds evidence-based analysis directives.

## Usage

```
/anti-hallucination [target]
```

**Target options:**
- **File path** (e.g., `agents/code-reviewer.md`): Enhance single agent/prompt file
- **Folder path** (e.g., `agents/`): Enhance all `.md` files in folder recursively
- **No argument**: Defaults to `agents/` folder

## Step 1: Parse the Argument

Determine the scope based on user input: `$ARGUMENTS`

| Input | Scope | Pattern |
|-------|-------|---------|
| No argument | All agents | `agents/*.md` |
| File path | Single file | Exact path |
| Folder path | All .md files in folder | `{folder}/**/*.md` |

**Error Handling:**
- If file path doesn't exist: Report error and exit
- If folder path doesn't exist: Report error and exit
- If no .md files found in scope: Report "No files to analyze" and exit

## Step 2: Discover Files

Use Glob to find target files based on scope:

```
# For single file
{file_path}

# For folder
{folder}/**/*.md

# Default (no argument)
agents/*.md
```

Count total files to analyze.

## Step 3: Launch Anti-Hallucination Agent

Use the Task tool with `subagent_type: "anti-hallucination-agent"`:

### For single file:
```
Analyze and enhance this agent/prompt file with anti-hallucination safeguards: {file_path}

1. Read the file completely
2. Identify the agent's domain and purpose
3. Determine risk areas for hallucination:
   - Where might the agent fill gaps with assumptions?
   - Where might vague input lead to fabricated output?
   - What evidence should be required before making claims?
   - What cross-validation is needed?

4. Select appropriate anti-hallucination pattern(s):
   - Pattern 1: Evidence-Based Scoring - for assessment/review agents
   - Pattern 2: Read-Before-Reference - for code/file analysis agents
   - Pattern 3: Error Handling - for data processing agents
   - Pattern 4: Verification Before Claims - for technical documentation agents

5. Generate customized anti-hallucination policy:
   - Customize mandatory rules for the domain
   - Add domain-specific red flags and acceptable evidence
   - Include cross-validation requirements if applicable
   - Add examples of correct vs. incorrect analysis

6. Insert policy at optimal location:
   - After "When Invoked" or "Focus Areas" section
   - Before detailed workflow sections
   - As new section before scoring/assessment sections

7. Update success criteria to include anti-hallucination verification

8. Report changes made with summary of:
   - Which pattern(s) applied
   - Risk areas addressed
   - Where policy was inserted
   - Key customizations for this domain
```

### For multiple files (folder):
```
Analyze and enhance all agent/prompt files in this scope with anti-hallucination safeguards.

Files to analyze:
{list of file paths}

For each file:
1. Read the file completely
2. Identify the agent's domain and purpose
3. Determine risk areas for hallucination
4. Select appropriate anti-hallucination pattern(s)
5. Generate customized anti-hallucination policy
6. Insert policy at optimal location
7. Update success criteria

After processing all files, generate a summary report:

## Anti-Hallucination Enhancement Summary

### Files Enhanced
| File | Domain | Pattern(s) Applied | Risk Areas Addressed |
|------|--------|-------------------|---------------------|
| ... | ... | ... | ... |

### Files Skipped
[List files that already have anti-hallucination policies]

### Common Risk Areas Found
[Patterns observed across multiple agents]

### Recommendations
[Additional safeguards or cross-agent consistency improvements]
```

## Step 4: Report to User

After enhancement completes:
1. Summarize changes made:
   - Number of files enhanced
   - Patterns applied by domain
   - Risk areas addressed
2. List any files that were skipped (already had policies)
3. Highlight any files needing manual review

## CRITICAL

- **READ BEFORE MODIFYING** - Always read files completely before injecting policies
- **DOMAIN-SPECIFIC** - Policies must be customized to each agent's domain, not generic
- **PRESERVE STRUCTURE** - Insert policies at natural locations, don't disrupt agent flow
- **NO OVER-RESTRICTION** - Policies should prevent fabrication without making agents unusable
- **VERIFY COMPLETENESS** - Ensure policy addresses all identified risk areas
- Use TodoWrite to track progress

## Anti-Hallucination Pattern Reference

### Pattern 1: Evidence-Based Scoring
For: code-reviewer, auditor-security, qa-expert, prompt-expert
Focus: Conservative scoring, evidence vs. claims, cross-validation

### Pattern 2: Read-Before-Reference
For: code-reviewer, architect-backend, simplify-code
Focus: Only cite read files, quote actual code, explicit coverage

### Pattern 3: Error Handling
For: database-administrator, performance-engineer
Focus: Stop on missing data, no gap-filling, explicit errors

### Pattern 4: Verification Before Claims
For: writer-technical, readme-expert
Focus: Source attribution, explicit uncertainty, no speculation

## Usage Examples

### Example 1: Enhance single agent
```bash
/anti-hallucination agents/code-reviewer.md
# Adds anti-hallucination policy to code-reviewer
```

### Example 2: Enhance all agents
```bash
/anti-hallucination agents/
# Enhances all agents in the agents/ folder
```

### Example 3: Enhance default scope
```bash
/anti-hallucination
# Same as /anti-hallucination agents/
```

### Example 4: Enhance specific folder
```bash
/anti-hallucination prompts/
# Enhances all .md files in prompts/ folder
```
