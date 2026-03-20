---
description: Run vulture to detect dead Python code and generate analysis report
argument-hint: "[path]"
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Task, TodoWrite
ultrathink: true
---

# I See Dead People

Detect and analyze dead (unused) Python code using vulture.

## Usage
```
/i-see-dead-people [path]
```

- `[path]` - Optional path to analyze (defaults to `src/` or `.` if no src directory)

## Workflow

## Anti-Hallucination Policy

**CRITICAL**: Verify all dead code claims against actual codebase content. Never recommend deletion based solely on agent analysis without direct verification.

### Mandatory Rules

1. **No File Path Fabrication**: Only include findings in report where Read tool confirmed file exists at specified path
2. **No Line Number Invention**: Verify line numbers with Read tool before including in report - incorrect line numbers undermine report credibility
3. **No Usage Speculation**: Only claim code is "unused" after Grep tool confirms no references exist in codebase
4. **Verify Agent Claims**: Agent conclusions must be validated with Read/Grep before inclusion in report - agents can hallucinate references or miss usage patterns
5. **No Confidence Inflation**: Assign confidence levels based ONLY on verified evidence (grep results, read content), not agent assertions
6. **Evidence-Based False Positives**: Only move findings to "False Positives" section after Read/Grep confirms the item is actually used

### Read-Before-Report Requirements

**CRITICAL**: Before adding ANY finding to VULTURE_REPORT.md, verify:
1. **File Exists**: Use Read tool to confirm file at reported path exists
2. **Item Exists at Line**: Use Read tool with line offset to verify item exists at reported line number
3. **Usage Verified**: Use Grep tool to search for references across codebase
4. **Agent Claims Validated**: Cross-reference agent analysis with actual grep/read results

**If verification fails for any finding, DO NOT include it in report**

### Validation Workflow Discipline

**Lines 304-310 describe validation steps - make them mandatory**:
- [ ] Read ALL files referenced in report to verify they exist (not just agent claims)
- [ ] Grep for each "dead" item name to verify it's truly unused (not just agent analysis)
- [ ] Verify line numbers match actual code with Read tool + offset (not just vulture output)
- [ ] Check for dynamic usage patterns with explicit grep searches (not just agent speculation)
- [ ] Remove findings where verification fails - better to under-report than fabricate dead code

### Red Flags for Unverified Claims

**Agent Analysis Without Codebase Verification** (Must Validate):
- "No references found" - verify with `grep -r "item_name" path/`
- "Item is unused" - verify file exists and grep confirms no usage
- "False positive due to dynamic usage" - verify with grep for getattr/\_\_all\_\_/dynamic patterns
- "High confidence dead code" - verify confidence with actual read/grep evidence
- "Line X contains unused Y" - verify with Read tool that line X actually contains Y

**Acceptable Evidence** (Report These):
- File verified with Read tool + grep shows 0 references outside definition = HIGH CONFIDENCE
- File verified with Read tool + grep shows usage in tests only + item not in \_\_all\_\_ = MEDIUM CONFIDENCE
- File verified with Read tool + grep shows \_\_all\_\_ export or dynamic getattr = FALSE POSITIVE
- Vulture finding + Read confirms line number + Grep shows no callers = HIGH CONFIDENCE

### Confidence Level Verification

**Before assigning confidence level, verify with tools**:
- **High Confidence**: Read confirmed file exists + Grep shows 0 references except definition + no \_\_all\_\_ entry
- **Medium Confidence**: Read confirmed file exists + Grep shows limited usage (test only) OR single agent flagged with strong grep evidence
- **Low Confidence**: Read confirmed file exists + Grep shows possible references OR dynamic usage patterns detected

**If Read/Grep verification contradicts agent analysis, trust the tools over agent claims**

### Report Generation Discipline

**Before writing VULTURE_REPORT.md, verify**:
- [ ] Every file path was verified with Read tool (not assumed from vulture output)
- [ ] Every line number was verified with Read tool (not copied blindly from vulture)
- [ ] Every "unused" claim was verified with Grep tool (not accepted from agent analysis)
- [ ] Every confidence level is justified by grep results (not agent assertions)
- [ ] Every false positive has verified evidence of usage (grep results showing references)
- [ ] Cleanup commands reference only verified findings (not unvalidated recommendations)

**If you cannot verify a finding with Read/Grep tools, remove it from the report**

### Example: Correct vs. Incorrect Analysis

**Incorrect** (Agent Hallucination):
```
Agent: "function dead_function() at line 42 is unused - no references found"
Report: Add to "High Confidence Dead Code" section
```
This accepts agent claim without verification - agent may have missed dynamic usage or had incorrect line number.

**Correct** (Tool-Verified):
```
Agent: "function dead_function() at line 42 is unused - no references found"
1. Read path/file.py:42 → Confirm "def dead_function()" exists at line 42
2. Grep "dead_function" in codebase → 1 result (only the definition)
3. Grep "getattr.*dead_function" → 0 results (no dynamic usage)
4. Grep "__all__" in file → dead_function not in __all__
Report: Add to "High Confidence Dead Code" - verified with grep showing only definition
```
This validates agent analysis with actual tool verification before reporting.

**Correct** (False Positive Detected):
```
Agent: "function callback_handler() at line 100 is unused"
1. Read path/file.py:100 → Confirm "def callback_handler()" exists
2. Grep "callback_handler" → 1 result (only definition)
3. Grep "register.*callback" → Found registration in config.py
4. Read config.py → Confirms string-based callback registration
Report: Add to "False Positives" section - used via dynamic registration in config.py (line 45)
```
This detects false positive through systematic grep verification.

### Validation Step Requirements

**Lines 252-297 describe validation agents - enforce verification**:

During Validation Step 3 (Lines 299-314), MANDATORY verification:
1. **Read every file** referenced in report - confirm it exists (not just agent validation)
2. **Grep every item name** - confirm usage pattern matches report claims (not just agent feedback)
3. **Check line numbers** with Read tool - confirm items exist at reported lines (not just agent validation)
4. **Verify cleanup safety** - grep for references that would break if item deleted (not just agent opinion)

**If validation reveals incorrect findings, remove them completely from report - do not keep unverified claims**

### Self-Verification Checklist

Before finalizing VULTURE_REPORT.md, verify:
- [ ] Every finding in "Confirmed Dead Code" section was verified with Read (file exists) + Grep (no references)
- [ ] Every confidence level is justified by grep result counts (not agent assertions)
- [ ] Every false positive has grep evidence of actual usage (not speculation)
- [ ] Every file path in report was tested with Read tool (not assumed valid)
- [ ] Every line number was confirmed with Read tool (not copied from vulture/agent output)
- [ ] Cleanup commands reference only grep-verified findings (not unvalidated items)
- [ ] Validation notes document what was verified with tools vs. removed due to failed verification

**If any checklist item fails, do not deliver the report - fix verification gaps first**

---

### Step 1: Parse the Argument

Determine the scope based on user input: `$ARGUMENTS`

- **No argument**: Default to `src/` if it exists, otherwise `.`
- **Path provided**: Use the specified path

**Error Handling:**
- If specified path doesn't exist: Report error and exit
- If path is not a valid directory or file: Report error and exit

### Step 2: Determine Delegation Strategy

Choose analysis approach based on scope:

```
IF analyzing single small directory (< 20 Python files):
    → Proceed with direct vulture execution
ELSE IF large codebase (20+ Python files):
    → Consider batch execution or focused analysis
    → May need to run vulture multiple times on subdirectories
```

To check file count:
```bash
find {path} -name "*.py" | wc -l
```

### Step 3: Run Vulture

Execute vulture to detect dead code:

```bash
uv run vulture {path} --min-confidence 80
```

If vulture is not installed:
```bash
uv add vulture --dev
```

**Error Handling:**
- **Vulture crashes**: Log the error, check if path is valid, try with smaller scope
- **Empty output**: Report "No dead code detected at 80% confidence threshold"
- **Invalid paths**: Verify path exists and contains Python files before running
- **Permission errors**: Report the error and suggest checking file/directory permissions

Capture the full output for analysis.

### Step 4: Parse Results

Vulture output format:
```
path/to/file.py:42: unused function 'my_function' (90% confidence)
path/to/file.py:15: unused import 'os' (90% confidence)
```

Categorize findings by type:
- **Unused imports**
- **Unused functions/methods**
- **Unused classes**
- **Unused variables**
- **Unused attributes**

### Step 5: Launch Analysis Agents (Max 3 Concurrent)

Use the Task tool to analyze findings. **Maximum 3 concurrent agents.**

Launch all three agents simultaneously (they will run in parallel):

**Code Reviewer** (`subagent_type: "code-reviewer"`):
```
Analyze these vulture dead code findings:

[paste vulture output]

Requirements:
- Verify if each finding is truly unused or a false positive
- Check for dynamic usage (getattr, __all__, etc.)
- Identify public API that should be preserved
- Recommend: DELETE, KEEP (with reason), or INVESTIGATE

Focus on unused functions, methods, and classes.
```

**Dev Python** (`subagent_type: "dev-python"`):
```
Analyze these vulture dead code findings for Python-specific patterns:

[paste vulture output]

Requirements:
- Identify dunder methods called implicitly by Python
- Check for abstract methods/base class implementations
- Find callback functions registered elsewhere
- Detect pytest fixtures that appear unused
- Verify CLI entry points defined in pyproject.toml

Categorize findings as false positives or true dead code.
```

**Explore Agent** (`subagent_type: "Explore"`):
```
Explore the codebase to verify these potentially dead code items:

[paste list of functions/classes from vulture]

Requirements:
- Search for all references (imports, calls, inheritance)
- Check config files, tests, and documentation
- Look for dynamic usage patterns (string-based imports, getattr)

Report items as: truly unused, possibly used, or definitely used.
```

### Step 6: Synthesize Findings

After all agents complete:
1. Collect all agent analyses
2. **MANDATORY: Verify agent claims with tools before assigning confidence:**
   - For each finding, use Read tool to confirm file exists and item is at reported line
   - For each finding, use Grep tool to count actual references in codebase
   - Cross-reference agent conclusions with grep results - discard findings where agent claims conflict with grep evidence
3. **Tool-Verified Confidence Assignment** - Base confidence ONLY on grep result counts (not agent assertions):
   - **High Confidence**: Grep shows 0-1 references (only definition) + Read confirmed file/line + no \_\_all\_\_ entry
   - **Medium Confidence**: Grep shows 2-5 references (limited usage) + Read confirmed file/line
   - **Low Confidence**: Grep shows 6+ references OR dynamic usage patterns detected + Read confirmed file/line
4. Separate tool-verified dead code from tool-verified false positives (grep shows active usage)
5. Prioritize by impact (large unused classes > single unused variables)
6. **Remove findings that fail verification** - If Read shows wrong line number or Grep contradicts agent claim, delete the finding

**CRITICAL**: Do not assign confidence based on "flagged by N agents" - agents can all be wrong. Only grep result counts determine confidence.

### Step 7: Generate Report

Write `VULTURE_REPORT.md` to repository root:

```markdown
# Dead Code Analysis Report

**Generated:** [date]
**Path Analyzed:** [path]
**Tool:** vulture (min-confidence 80%)

## Summary

| Category | Count | Action Needed |
|----------|-------|---------------|
| Unused imports | X | DELETE |
| Unused functions | X | DELETE |
| Unused classes | X | DELETE |
| Unused variables | X | DELETE |
| False positives | X | IGNORE |

**Total dead code:** X items
**Estimated lines to remove:** ~X

## Confirmed Dead Code

### High Confidence (Delete These)

#### Unused Imports
| File | Line | Import | Confidence |
|------|------|--------|------------|
| path/file.py | 42 | `unused_module` | 90% |

#### Unused Functions/Methods
| File | Line | Name | Confidence | Notes |
|------|------|------|------------|-------|
| path/file.py | 100 | `dead_function()` | 95% | No callers found |

#### Unused Classes
| File | Line | Name | Confidence | Notes |
|------|------|------|------------|-------|
| path/file.py | 200 | `DeadClass` | 90% | No inheritance or instantiation |

#### Unused Variables
| File | Line | Name | Confidence |
|------|------|------|------------|
| path/file.py | 50 | `unused_var` | 85% |

### False Positives (Keep These)

| File | Line | Item | Reason to Keep |
|------|------|------|----------------|
| path/file.py | 10 | `__all__` entry | Public API |
| path/file.py | 20 | `pytest fixture` | Used by pytest |

## Suggested Cleanup Commands

**NOTE**: These commands are suggestions for the user to run manually. Do NOT execute them automatically.

```bash
# To remove unused imports automatically:
uv run autoflake --remove-all-unused-imports --in-place --recursive {path}

# To verify changes after manual cleanup:
uv run ruff check {path}
uv run mypy {path}
uv run pytest
```

## Recommendations

1. [Priority cleanup recommendations]
2. [Files with most dead code]
3. [Patterns to avoid in future]

## Agent Analysis Details

### Code Reviewer Findings
[Summary of code reviewer analysis with specific findings, recommendations, and confidence levels]

### Python Expert Findings
[Summary of dev-python analysis with Python-specific patterns identified, false positives detected, and language-specific considerations]

### Codebase Exploration Findings
[Summary of explore agent analysis with reference search results, usage patterns found, and dynamic usage detection]
```

### Step 8: Proceed to Validation

Continue to the **Validation Workflow** section below with `VULTURE_REPORT.md` as the target file.

---

## Validation Workflow

After generating the report, validate and improve the output.

### Validation Step 1: Read VULTURE_REPORT.md

Read the file created by the analysis workflow.

### Validation Step 2: Launch Validation Agents in Parallel

Use the Task tool to launch ALL of these agents simultaneously:

**Code Reviewer** (`subagent_type: "code-reviewer"`):
```
Review this dead code analysis report for technical accuracy:

[paste report content]

Requirements:
- Verify the identified dead code is actually unused
- Check if suggested deletions would break anything
- Validate the confidence levels are appropriate
- Identify any false positives that should be kept

Return specific issues and corrections needed.
```

**Dev Python** (`subagent_type: "dev-python"`):
```
Review this dead code analysis report for Python-specific accuracy:

[paste report content]

Requirements:
- Verify Python-specific patterns are correctly identified
- Check for missed dunder methods or special cases
- Validate pytest fixture and CLI entry point analysis
- Confirm abstract method and callback detection

Return specific issues and recommendations.
```

**Explore Agent** (`subagent_type: "Explore"`):
```
Explore the codebase to validate this dead code analysis report:

[paste file paths and item names from report]

Requirements:
- Verify all referenced files exist
- Search for each "dead" item to confirm it's unused
- Check for dynamic imports or string-based references
- Validate line numbers are accurate

Return findings about report accuracy.
```

### Validation Step 3: Synthesize and Validate Findings

After all validation agents complete:

1. Collect all agent feedback
2. **MANDATORY: Validate EVERY finding against the codebase with tools (not agent claims):**
   - **Read** referenced files to verify existence - `Read /path/to/file.py` must succeed
   - **Read with offset** to verify line numbers - `Read /path/to/file.py offset=40 limit=5` must show item at reported line
   - **Grep** for item names to verify usage pattern - `grep "item_name" -r path/` must return results matching claim
   - **Grep** for dynamic patterns if false positive claimed - `grep -E "(getattr|__all__|register)" file.py` must show evidence
   - Confirm confidence levels match grep result counts (0 refs = high, few refs = medium, many refs = low/false positive)
   - Check that cleanup commands reference only grep-verified unused items
3. **MANDATORY: Remove invalid findings entirely** - If Read fails, line number is wrong, or Grep contradicts "unused" claim, DELETE the finding
4. **Trust tools over agents** - If Read/Grep verification contradicts agent analysis, remove the finding from report
5. Keep only tool-validated, accurate recommendations (verified with Read + Grep evidence)
6. Identify consensus issues (flagged by multiple validation agents) AND verified with tools
7. Adjust confidence levels based on grep result counts (not agent opinions)

### Validation Step 4: Update VULTURE_REPORT.md

Edit the report to:
- Remove invalid findings
- Correct inaccurate confidence levels
- Fix incorrect line numbers or file paths
- Update false positive categorization
- Add any dead code missed during initial analysis
- Clarify cleanup command recommendations

Add a "Validation Notes" section at the end:
```markdown
## Validation Notes

**Validated:** [date]

### Changes Made
- [finding 1]: [removed/corrected/confidence adjusted]
- [finding 2]: [removed/corrected/confidence adjusted]

### Findings Removed (Invalid)
- [removed finding]: [why it was invalid - file doesn't exist, item is used, etc.]

### Additional Dead Code Found
- [any new dead code discovered during validation]

### Confidence Adjustments
- [item]: [old confidence] -> [new confidence] because [reason]
```

### Validation Step 5: Report Summary

Report to the user:
- Where the report was saved (VULTURE_REPORT.md in repository root)
- Total dead code items found by confidence level
- Number of findings removed/corrected during validation
- Estimated lines of code that can be removed
- Recommended next steps for cleanup

---

## Success Criteria

Dead code analysis meets quality standards when:
- [ ] Vulture executed successfully on specified path
- [ ] Findings categorized by type (imports, functions, classes, variables, attributes)
- [ ] Agent analysis completed with all three agents
- [ ] VULTURE_REPORT.md generated in repository root
- [ ] Validation workflow completed with all findings verified
- [ ] Invalid findings removed from final report
- [ ] Confidence levels assigned (High/Medium/Low)
- [ ] Cleanup commands provided as suggestions (not executed)
- [ ] User receives summary with actionable next steps

### Anti-Hallucination Verification

- [ ] Every file path in report verified with Read tool (not assumed from vulture/agent output)
- [ ] Every line number verified with Read tool to confirm item exists at that line (not blindly copied)
- [ ] Every "unused" claim verified with Grep tool showing no references beyond definition (not accepted from agent analysis alone)
- [ ] Every confidence level justified by grep result evidence (not agent assertions)
- [ ] Every false positive backed by grep evidence of actual usage (not speculation)
- [ ] All findings contradicted by Read/Grep verification removed from report (no unverified claims)
- [ ] Validation notes document verification method for each finding (tool evidence vs. agent claim)

---

## Integration Points

### Collaboration Patterns

**Delegate to code-reviewer when:**
- Validating dead code findings for technical accuracy
- Verifying deletion recommendations won't break code
- Checking if code is truly unused vs false positive
- Confirming public API preservation

**Delegate to dev-python when:**
- Identifying Python-specific patterns (dunder methods, fixtures, etc.)
- Analyzing abstract methods and callbacks
- Detecting CLI entry points and dynamic imports
- Understanding language-specific false positives

**Delegate to Explore agent when:**
- Searching for usage patterns across codebase
- Verifying items are truly unused
- Finding dynamic references and string-based imports
- Validating file paths and line numbers

### When to Invoke Other Agents

Always invoke all three analysis agents (code-reviewer, dev-python, Explore) for comprehensive coverage. Each provides unique perspective on dead code detection.

Always invoke all three validation agents after report generation to ensure accuracy.

---

## Usage Examples

### Example 1: Analyze src directory
```bash
/i-see-dead-people src/
# Analyzes all Python files in src/ for dead code
```

### Example 2: Analyze entire codebase
```bash
/i-see-dead-people .
# Analyzes all Python files in current directory recursively
```

### Example 3: Analyze specific package
```bash
/i-see-dead-people src/services/
# Analyzes only the services package
```

---

## CRITICAL

- **ALWAYS USE AGENTS** - All analysis MUST be delegated to agents via the Task tool
- **MAXIMUM 3 CONCURRENT AGENTS** - Never spawn more than 3 at once to avoid resource exhaustion
- **SAVE REPORT TO ROOT** - Always save VULTURE_REPORT.md in the repository root, NOT in `.claude/`
- **DO NOT DELETE CODE** - Only generate the report; let user decide what to remove
- **DO NOT COMMIT** - Leave that to the user
- **USE TODOWRITE** - Track progress through all phases using TodoWrite tool
- **ALWAYS VALIDATE** - Every analysis MUST be followed by the validation workflow
- **VERIFY BEFORE RECOMMENDING DELETION** - False positives are common with dynamic Python code
