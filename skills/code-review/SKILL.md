---
description: Run comprehensive code review with multi-agent coordination and validation
allowed-tools: Read, Write, Edit, Glob, Grep, Task, AskUserQuestion
ultrathink: true
---

# Code Review

Run a comprehensive code review using multiple specialized agents, then automatically validate the output.

## Usage
```
/code-review
```

### Examples
```
/code-review                    # Full code review + validation
```

---

## Anti-Hallucination Policy

**CRITICAL**: All analysis, validation, and reporting MUST be grounded in actual code that was explicitly read. Never fabricate file paths, function names, architectural patterns, or issues.

### Mandatory Rules for All Agents

1. **No File Path Fabrication**: Only reference files that were read using Read tool
2. **No Code Invention**: Only reference functions, classes, methods that exist in read files
3. **No Pattern Assumption**: Only describe architectural patterns verified in actual code
4. **Explicit Read Tracking**: Log which files were read vs. not read in each phase
5. **Gap Acknowledgment**: State "File not read" or "Implementation not verified" when uncertain
6. **Quote Actual Code**: When citing issues, quote actual lines with line numbers from read files
7. **No Issue Fabrication**: Only report issues that exist in code that was actually read and analyzed

### Read-Only Analysis Requirements

**CRITICAL for Step 1 (Explore & Architect Agents)**:
1. **No Assumptions About Project Structure**: Read actual files to determine language, framework, architecture
2. **Explicit Coverage**: Document which directories/files were explored vs. not explored
3. **Evidence-Based Patterns**: Only claim architectural patterns exist if verified in read code (e.g., "Found layered architecture: controllers/ at line 12 of main.py imports services/ modules")
4. **No Boilerplate Detection**: Do not assume standard project structure without reading it

**CRITICAL for Step 2 (Review Execution)**:
1. **Issue Must Reference Actual Code**: Every reported issue must include file path, line numbers, and quoted code
2. **No Generic Issues**: Do not report "typical problems" without verifying they exist in THIS codebase
3. **Cross-File Claims Require Reading Both**: If claiming coupling between modules, must read both modules
4. **Missing Functionality Claims**: Only report missing functionality if analyzing requirements docs or user stories (not assumptions about what "should" exist)

**CRITICAL for Step 4 (Validation)**:
1. **Verify Every Finding**: Validation agents MUST read the actual code referenced in each finding
2. **Remove Fabricated Issues**: If finding references non-existent files/functions, remove it entirely
3. **No Assumed Fixes**: Do not validate a finding as "correct" without reading the actual code it references
4. **Document What Was Verified**: State which findings were verified by reading code vs. not verified

### Red Flags for Invalid Analysis

**Vague Claims Without Evidence** (Must Be Removed in Validation):
- "This module likely has performance issues" (no profiling data, no specific code reference)
- "Error handling appears insufficient" (no specific try/catch blocks referenced with line numbers)
- "The architecture seems to follow microservices" (no actual service boundaries identified in code)
- "Tests are probably missing for edge cases" (no test files read, no specific untested code paths identified)
- "This function might have security vulnerabilities" (no specific vulnerability pattern with code quote)

**Acceptable Evidence-Based Findings** (Keep in Validation):
- "Function authenticate() at auth.py:127 accepts user input without validation (line 128 directly passes to SQL query)"
- "Module payments/processor.py imports 47 dependencies (lines 1-52), creating tight coupling with entire codebase"
- "No error handling in api/endpoints.py:89-102 - uncaught exceptions will crash service"
- "Test file tests/test_auth.py (lines 15-30) only covers happy path, missing edge case for expired tokens referenced at auth.py:145"

### Validation Cross-Check Requirements

**Before including finding in final CODE_REVIEW.md, verify**:
- [ ] Referenced file path exists and was read (not assumed to exist)
- [ ] Quoted code matches actual file content with correct line numbers
- [ ] Issue is present in current code (not based on outdated assumptions)
- [ ] Related files referenced in finding were also read (e.g., if claiming module A calls module B, both were read)
- [ ] Architectural claims are backed by actual code structure (not inferred from directory names alone)

**If you cannot verify these with read content, remove the finding entirely**

### Step-Specific Anti-Hallucination Checkpoints

**Step 1 (Review Plan Creation)**:
- [ ] Project type determined by reading actual source files (not guessed from directory names)
- [ ] Framework identified from read configuration or import statements (with file:line references)
- [ ] Architectural patterns verified in actual code structure (quote imports, class hierarchies)
- [ ] Critical files identified by reading them (not assumed from names like "main.py")
- [ ] Complexity assessment based on actual metrics (file sizes, import counts, nesting depth with evidence)

**Step 2 (Review Execution)**:
- [ ] Every issue includes file path, line numbers, and code quote
- [ ] No issues reported about unread files
- [ ] Specialist agents read files in their focus areas before reporting
- [ ] Cross-module concerns verified by reading all involved modules
- [ ] Severity assessments based on actual impact (not assumed from issue type)

**Step 3 (Report Writing)**:
- [ ] Executive summary reflects only findings from read code
- [ ] Issue counts match actual issues with file references
- [ ] No generic best practices reported without code evidence
- [ ] Recommendations address actual code problems (not theoretical improvements)

**Step 4 (Validation)**:
- [ ] Validation agents read every file referenced in CODE_REVIEW.md
- [ ] Findings with non-existent references removed entirely
- [ ] Added issues backed by reading actual code (not assumptions)
- [ ] "Findings Removed" section documents why each was invalid (quote mismatch, file doesn't exist, etc.)

---

## Step 1: Create Review Plan

Launch agents in parallel to explore the codebase and create a review plan.

**Explore Agent** (`subagent_type: "Explore"`):
```
"Explore the codebase to understand its structure and identify what needs review.

**CRITICAL - Anti-Hallucination Policy**: Only report findings from files you actually read. Do not assume project structure, language, or framework from directory names alone. Every claim must reference specific files with line numbers.

Discover:
- Project type (Python, Node, Go, etc.) and framework - READ actual source files to determine
- Directory structure and key entry points - READ entry point files to verify
- Core modules, services, and their relationships - READ import statements to map dependencies
- Configuration and infrastructure files - READ config files, do not assume standard patterns
- Test coverage areas - READ test files to assess what is tested

Return a structured summary of:
1. Project overview (language, framework, architecture pattern) - WITH FILE REFERENCES (e.g., "Python 3.11 detected in pyproject.toml:3, FastAPI framework imported in main.py:1")
2. Key directories and their purposes - WITH READ FILE EXAMPLES from each directory
3. Critical files that must be reviewed (entry points, core logic, APIs) - LIST ONLY FILES YOU READ
4. Areas of complexity or risk (large files, deep nesting, many dependencies) - WITH SPECIFIC METRICS (e.g., "auth.py has 847 lines, 12-level nesting at line 234, imports 23 modules")"
```

**Architect Reviewer** (`subagent_type: "architect-reviewer"`):
```
"Analyze the codebase architecture to identify review priorities.

**CRITICAL - Anti-Hallucination Policy**: Only describe architectural patterns actually present in read code. Do not assume "typical" patterns for the framework/language. Every architectural claim must cite specific files and code structure.

Assess:
- Architectural patterns in use (layered, microservices, monolith, etc.) - VERIFY by reading actual module structure and imports
- Dependency graph and coupling between modules - MAP by reading import statements across files
- API boundaries and integration points - IDENTIFY by reading route definitions, endpoint files
- Data flow and state management patterns - TRACE by reading actual data handling code

Return recommended review focus areas:
1. High-risk architectural concerns to examine - WITH FILE REFERENCES (e.g., "Circular dependency: auth.py:5 imports user.py, user.py:12 imports auth.py")
2. Critical paths that need thorough review - WITH ACTUAL CODE PATHS TRACED (e.g., "Request flow: main.py:45 -> router.py:89 -> handler.py:123")
3. Integration points prone to issues - WITH READ API/INTEGRATION CODE (e.g., "External API called at payments.py:67 with no timeout configured")
4. Suggested specialist agents needed (security, database, performance, etc.) - JUSTIFIED by actual code patterns found (e.g., "Need security agent: found SQL query construction at db.py:234 without parameterization")"
```

After both agents complete, synthesize their findings into a **Review Plan**:

```markdown
## Review Plan

### Codebase Profile
- Language/Framework: [from Explore]
- Architecture: [from Architect]
- Size: [file counts by type]

### Review Scope
| Priority | Area | Files | Rationale |
|----------|------|-------|-----------|
| Critical | [area] | [files] | [why] |
| High | [area] | [files] | [why] |
| Medium | [area] | [files] | [why] |

### Specialist Agents Required
- [agent]: [what they'll review]
- [agent]: [what they'll review]

### Review Focus Questions
1. [specific question to answer]
2. [specific concern to investigate]
```

Save this plan to memory (do not write to file) and pass it to Step 2.

---

## Step 2: Execute Review Based on Plan

Use the Task tool to launch the multi-agent-coordinator with the review plan:

**Multi-Agent Coordinator** (`subagent_type: "multi-agent-coordinator"`):
```
"Execute a code review following this plan:

[INSERT REVIEW PLAN FROM STEP 1]

Based on this plan:
1. Launch the identified specialist agents for their designated areas
2. Ensure critical and high-priority areas get thorough coverage
3. Coordinate agent execution intelligently
4. Synthesize results into actionable recommendations
5. Resolve any conflicting findings between agents

**CRITICAL: READ-ONLY ANALYSIS ONLY - NO CODE CHANGES**
Agents must only analyze, review, and report. Do not modify any files, make any edits, or implement any fixes. This is strictly an analysis and reporting phase.

**CRITICAL: ISSUES-ONLY REPORTING - NO POSITIVE FEEDBACK**
Do NOT include:
- Things that were done correctly
- Good practices already in use
- Positive feedback or praise
- Working features or implementations
- General commentary about code quality

ONLY report:
- Bugs and issues that need fixing
- Security vulnerabilities
- Missing functionality that should be added
- Performance problems
- Code that violates best practices
- Technical debt that needs addressing
- Specific, actionable improvements required

Be direct and concise. Skip all congratulatory or validating language.

**CRITICAL - Anti-Hallucination Policy for All Specialist Agents**:
Every reported issue MUST include:
- Exact file path (verified with Read tool)
- Line numbers where issue exists
- Quoted code showing the problem
- NO generic issues without specific code references
- NO assumed problems based on "typical" patterns for this language/framework"
```

## Step 3: Write Review Report

After the coordinator completes, write a comprehensive report file named `CODE_REVIEW.md` with this structure:

```markdown
# Code Review

**Date:** YYYY-MM-DD HH:MM

## Executive Summary

[Issue counts and severity breakdown]

## Critical Issues

[Issues that must be fixed immediately]

## High Priority Issues

[Significant issues that should be addressed soon]

## Medium Priority Issues

[Issues that should be addressed when possible]

## Low Priority Issues

[Minor improvements and suggestions]

## Actionable Recommendations

[Implementation priorities and suggested order of fixes]
```

**EXCLUDE:** positive feedback, things done correctly, general praise
**INCLUDE ONLY:** problems, bugs, vulnerabilities, missing features, required improvements

## Step 4: Proceed to Validation

Continue to the **Validation Workflow** section below with `CODE_REVIEW.md` as the target file.

---

## Validation Workflow

After the review completes, validate and improve the output.

### Validation Step 1: Read CODE_REVIEW.md

Read the file created by the review workflow.

### Validation Step 2: Launch Review Agents in Parallel

Use the Task tool to launch ALL of these agents simultaneously:

**Architect Reviewer** (`subagent_type: "architect-reviewer"`):
```
"Review this code review document for architectural soundness:

[paste document content]

Analyze:
- Are the identified issues accurate?
- Are there architectural concerns or anti-patterns missed?
- Are the recommendations appropriate?
- Are there dependency issues not mentioned?

Return specific issues and recommendations."
```

**Code Reviewer** (`subagent_type: "code-reviewer"`):
```
"Review this code review document for technical accuracy:

[paste document content]

Analyze:
- Are the identified issues technically accurate?
- Are there missing issues or concerns?
- Are edge cases considered?
- Is error handling addressed?

Return specific issues and recommendations."
```

**Explore Agent** (`subagent_type: "Explore"`):
```
"Explore the codebase to validate this code review document:

[paste relevant file paths from document]

Check:
- Do the referenced files/paths exist?
- Are the identified issues actually present in the code?
- Are there conflicts with the analysis?

Return findings about document accuracy."
```

### Validation Step 3: Synthesize and Validate Findings

After all agents complete:
1. Collect all agent feedback
2. **Validate each finding against the codebase (Apply Anti-Hallucination Policy):**
   - Check if referenced files/functions actually exist (read them with Read tool)
   - Verify the issue is real and not based on incorrect assumptions (quote actual code with line numbers)
   - Confirm the recommendation is applicable to the actual code (cross-reference with read files)
   - **Use validation checkpoints from Anti-Hallucination Policy section above**
3. **Remove invalid findings entirely** - If a finding references non-existent code, misunderstands the architecture, or is otherwise incorrect, remove it completely. Do not try to fix or reword invalid findings. Document removal reason in "Findings Removed" section.
4. Keep only validated, accurate summaries and recommendations (backed by read code)
5. Identify consensus issues (flagged by multiple agents, all verified against actual code)
6. Prioritize by impact (blockers vs improvements, based on actual code severity)

### Validation Step 4: Update CODE_REVIEW.md

Edit the document to:
- Remove any invalid findings
- Add any missed issues discovered during validation
- Clarify ambiguous sections
- Note any unresolved concerns

Add a "Review Notes" section at the end:
```markdown
## Review Notes

**Validated:** [date]

### Issues Addressed
- [issue 1]: [how it was fixed]
- [issue 2]: [how it was fixed]

### Findings Removed (Invalid)
- [removed finding]: [why it was invalid]

### Additional Issues Found
- [new issue discovered during validation]
```

### Validation Step 5: Report Summary

Report to user:
- Output file location: `CODE_REVIEW.md`
- Number of issues found by category
- Number of findings removed during validation
- Any open questions requiring user input

---

## CRITICAL

- **ALWAYS USE MULTIPLE AGENTS** - All work MUST be delegated to agents via the Task tool. Never perform review or validation directly. Every workflow uses 3+ agents in parallel.
- **DO NOT COMMIT** - Leave that to the user
- Use TodoWrite to track progress through all phases
