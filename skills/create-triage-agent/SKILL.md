---
description: Bootstrap a new Incident Triage sub-agent (Triage Titan) — repo, code, Terraform PR, orchestrator update
argument-hint: "[rally-ticket-id]"
allowed-tools: Read, Glob, Bash, AskUserQuestion, Agent, mcp__cai-mcp__getRallyItem
---

# Create Triage Agent

Bootstrap a new child agent for the Incident Triage system (Triage Titans).

This skill is an **orchestrator** — it gathers requirements and then delegates each phase to specialized sub-agents. Do not implement code or Terraform yourself.

## Sub-agent delegation map

| Phase | Agent | Task |
|-------|-------|------|
| Plan | `expert-agentcore` | Read template agent + infra, write `{agent_name}_PLAN.md` |
| 1-5 | `dev-strands` | Create repo, implement Python code, run quality checks, commit |
| Review | `code-reviewer` | Review code quality, security, Strands patterns |
| Fix (loop) | `dev-strands` | Fix confirmed issues, re-commit; repeat until user satisfied |
| 6 | `specialist-terraform` | Add infrastructure, validate, commit, create PR |
| 7 | `prompt-expert` | Add routing guidance to orchestrator system prompt |

---

## Step 1: Locate Repositories

Resolve four path variables before anything else:
- `{code_root}` — directory containing all cloned agent repos (e.g. `~/COX/code`)
- `{template_agent}` — absolute path to the best available template (prefer splunk → newrelic → cloudwatch)
- `{infra_root}` — absolute path to `incident-triage-infrastructure`
- `{orchestrator_root}` — absolute path to `incident-triage-orchestrator-agent`

**Discovery order:**
1. Check CLAUDE.md for path hints
2. Run `ls` in workspace — if `code/` subdirectory has `incident-triage-*` repos, that's it
3. If ambiguous, use `AskUserQuestion`
4. Detect template: `ls {code_root} | grep "incident-triage-.*-agent"` → pick first of splunk → newrelic → cloudwatch

**Do not proceed until all four paths are resolved.**

---

## Step 2: Gather Requirements

### If Rally ticket ID provided as `$ARGUMENTS`

```
mcp__cai-mcp__getRallyItem(objectId=ticket_id)
```

Extract: agent purpose, data source, acceptance criteria, owner.

### AskUserQuestion (always, even with a Rally ticket)

Ask in one batch:

1. **Agent name** — short lowercase, no hyphens (e.g. `github`, `datadog`, `pagerduty`)
2. **Data source + client pattern**:
   - HTTP/REST → `httpx` (like Splunk)
   - AWS SDK → `boto3` (like CloudWatch)
   - GraphQL → `httpx` (like NewRelic)
   - Other (describe)
3. **Authentication**: API key / AWS IAM role / OAuth / other
4. **Initial tools** — 2-3 Strands `@tool` function names (e.g. `search_issues`, `get_pull_request`)

If needed, second batch:
5. Custom Python dependencies?
6. Cox CI Governance CI ID? (use `CI_TBD` if not yet assigned — blocks Terraform apply)

**Never proceed without: agent_name, data_source, auth_method, initial_tools.**

---

## Step 3: Launch Planner (`expert-agentcore`)

```
Agent(
  subagent_type="expert-agentcore",
  prompt="""
You are planning a new Incident Triage child agent. Read the template agent and
infrastructure code, then produce a concrete implementation plan.

## New Agent Spec
- Agent name: {agent_name}
- Python namespace: ktlo_{agent_name}
- GitHub repo: incident-triage-{agent_name}-agent (GHE host: ghe.coxautoinc.com, org: SRE)
- AgentCore runtime name: incident_triage_{agent_name}_nonprod (underscore, max 48 chars)
- Data source: {data_source}
- Client pattern: {client_pattern} (httpx/boto3/graphql)
- Auth method: {auth_method}
- Tools to implement: {tool_1}, {tool_2}, {tool_3}
- Custom deps: {custom_deps}
- Cox CI ID: {ci_id}
- Rally ticket: {ticket_id}

## Paths
- Template agent to copy from: {template_agent}
- Infrastructure repo: {infra_root}
- Workspace root: {workspace_root}
- Plan output file: {workspace_root}/{agent_name}_PLAN.md

## Task
1. Read the template agent source:
   - {template_agent}/src/ktlo_*/agent.py
   - {template_agent}/src/ktlo_*/config.py
   - {template_agent}/src/ktlo_*/tools/**
   - {template_agent}/src/ktlo_*/configuration/*.md
   - {template_agent}/pyproject.toml
   - {template_agent}/Dockerfile

2. Read the infrastructure terraform:
   - {infra_root}/agentcore/terraform/main.tf
   - {infra_root}/agentcore/terraform/modules/runtime/main.tf
   - {infra_root}/agentcore/terraform/modules/identity/main.tf
   - {infra_root}/agentcore/terraform/modules/inference_profiles/main.tf
   - {infra_root}/agentcore/terraform/vars/awscaisreainp/us-east-1.tfvars

3. Write {agent_name}_PLAN.md to {workspace_root}/{agent_name}_PLAN.md with:
   - Architecture decisions (client pattern, auth, tool design)
   - Exact file list to create (all paths relative to repo root)
   - Namespace substitutions from template (e.g. ktlo_splunk → ktlo_{agent_name})
   - Config fields needed in config.py
   - IAM permissions needed in identity/main.tf (scoped ARNs, not *)
   - Terraform changes needed: exact resources to add in each .tf file, with
     references to actual line numbers from what you read
   - System prompt key sections for {agent_name}_agent.md
   - Agent card description for {agent_name}_agent_description.md
"""
)
```

Wait for completion. Confirm `{agent_name}_PLAN.md` was written.

---

## Step 4: Launch Code Implementation (`dev-strands`)

```
Agent(
  subagent_type="dev-strands",
  prompt="""
Implement a new Incident Triage child agent — a FastAPI + Strands A2A server following
the established pattern in the Triage Titans system.

## Agent Spec
- Agent name: {agent_name}
- Python namespace: ktlo_{agent_name}
- Data source: {data_source}
- Client pattern: {client_pattern}
- Auth method: {auth_method}
- Tools: {tool_1}, {tool_2}, {tool_3}
- Custom deps: {custom_deps}
- Port: 9000 (all child agents)

## Paths
- New repo location: {code_root}/incident-triage-{agent_name}-agent
- Template agent: {template_agent}
- Plan file: {workspace_root}/{agent_name}_PLAN.md

## GitHub
- Host: ghe.coxautoinc.com
- Org: SRE
- Repo: incident-triage-{agent_name}-agent

## Task: Phases 1-5

**Phase 1 — Repository setup:**
1. Check if GH repo exists: `gh repo view SRE/incident-triage-{agent_name}-agent --hostname ghe.coxautoinc.com`
2. If not: create it privately, clone with SSH into {code_root}/
3. Read the plan file at {workspace_root}/{agent_name}_PLAN.md
4. Read the template agent source files (agent.py, config.py, pyproject.toml,
   Dockerfile, Makefile, .gitignore, .env.sample, .github/workflows/)
5. Create all files in the new repo using the plan's file list:
   - pyproject.toml (copy + namespace substitution + add custom deps)
   - Makefile, Dockerfile, .gitignore, .env.sample, README.md
   - .github/workflows/deploy-nonprod.yml, deploy-prod.yml
   - src/ktlo_{agent_name}/__init__.py, agent.py, config.py, model_factory.py, prompts.py
   - src/ktlo_{agent_name}/services/{agent_name}_client.py
   - src/ktlo_{agent_name}/tools/{agent_name}/{agent_name}_tools.py
   - src/ktlo_{agent_name}/tools/tool_limiter.py
   - src/ktlo_{agent_name}/skills/ (copy all 6 files from template, rename namespace)
   - src/ktlo_{agent_name}/configuration/{agent_name}_agent.md (system prompt)
   - src/ktlo_{agent_name}/configuration/{agent_name}_agent_description.md
   - src/ktlo_{agent_name}/configuration/skills/{use_case}/SKILL.md
   - tests/__init__.py, tests/conftest.py, tests/test_agent.py,
     tests/test_{agent_name}_client.py, tests/test_skills.py
   - tests_integ/__init__.py, tests_integ/test_{agent_name}_integration.py

**Skills namespace fix** (mandatory after copying):
```bash
grep -r "ktlo_TEMPLATE" src/ktlo_{agent_name}/skills/  # find old namespace
find src/ktlo_{agent_name}/skills/ -name "*.py" -exec sed -i '' "s/ktlo_TEMPLATE/ktlo_{agent_name}/g" {{}} \\;
grep -r "ktlo_" src/ktlo_{agent_name}/skills/          # verify only ktlo_{agent_name} remains
```

**Phase 2 — Data source client:**
- Implement {client_pattern} pattern in {agent_name}_client.py
- For boto3: wrap sync calls with `asyncio.to_thread()`
- For httpx: use `async with httpx.AsyncClient()`
- {auth_method} credentials: check ARN env var first, fallback to direct env var

**Phase 3 — Tools:**
- Implement {tool_1}, {tool_2}, {tool_3} as `@tool`-decorated async functions
- Tools return `str` (JSON), not dict
- Complex params (lists/dicts) passed as JSON strings, parsed inside tool
- Apply LimitToolCounts in agent.py: max 3 per query tool, 2 per discovery tool

**Phase 4 — Agent entry point + system prompt:**
- agent.py: copy template, substitute namespace, update tool imports
- Keep triage_core imports verbatim: `from triage_core import (...)`
- System prompt must include: Role, Required Inputs, Workflow, Tool Reference,
  Mandatory Output Format, Constraints
- NEVER reference orchestrator, PRR, OBS 4, or other agents in system prompt

**Phase 5 — Quality checks:**
```bash
cd {code_root}/incident-triage-{agent_name}-agent
make sync
make format
make lint
make mypy
make tests
make coverage   # must pass ≥ 30%
```
Fix all issues before committing.

**Phase 5.5 — Commit and push:**
```bash
git add .
git commit -m "feat: initial {AgentName} agent scaffold (phases 1-5)

- FastAPI + Strands A2A server
- {data_source} client
- Tools: {tool_1}, {tool_2}, {tool_3}
- Skills infrastructure
- System prompt and agent card
- CI/CD workflows
- Unit tests (≥30% coverage)

Rally: {ticket_id}

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
git push -u origin main
```
"""
)
```

---

## Step 4.5: Launch Code Reviewer (`code-reviewer`)

```
Agent(
  subagent_type="code-reviewer",
  prompt="""
Review a newly implemented Incident Triage child agent for code quality, security,
and correctness. This is a Python FastAPI + Strands A2A agent.

## Agent Spec
- Agent name: {agent_name}
- Python namespace: ktlo_{agent_name}
- Repo path: {code_root}/incident-triage-{agent_name}-agent

## Focus Areas (in priority order)
1. Security — credentials in code, unscoped IAM, missing input validation, secrets exposure
2. Correctness — async/await patterns, asyncio.to_thread usage, tool return types (must be str)
3. Strands patterns — @tool decorator, LimitToolCounts, JSON return format, error handling
4. Code quality — namespace consistency (no leftover ktlo_template references), type hints
5. Tests — mocked boto3/httpx, coverage ≥ 30%

## Files to review (read all of these)
- src/ktlo_{agent_name}/agent.py
- src/ktlo_{agent_name}/config.py
- src/ktlo_{agent_name}/services/{agent_name}_client.py
- src/ktlo_{agent_name}/tools/{agent_name}/{agent_name}_tools.py
- src/ktlo_{agent_name}/configuration/{agent_name}_agent.md
- tests/test_agent.py
- tests/test_{agent_name}_client.py
- pyproject.toml

Also run:
```bash
cd {code_root}/incident-triage-{agent_name}-agent
make lint
make mypy
```

## Output Format
Structured report with:
- Summary: files reviewed, issues by severity (Critical/Major/Minor)
- Issues grouped by severity with: file:line, quoted code, impact, fix recommendation
- One-line verdict for each severity: "X Critical issues — must fix before deploy"

Be specific and evidence-based. Every issue must cite actual file:line.
"""
)
```

---

## Step 4.6: Present Review to User and Confirm Fix Scope

After `code-reviewer` returns its report, present it to the user and ask:

```
AskUserQuestion(
  questions=[
    "Code review found the following issues: [paste summary from reviewer].
     Which severity levels should we fix before proceeding?
     Options: (A) Critical only  (B) Critical + Major  (C) All (Critical + Major + Minor)  (D) Skip — proceed as-is"
  ]
)
```

- If user picks **(D)**: proceed directly to Step 5
- If user picks (A), (B), or (C): run Step 4.7

---

## Step 4.7: Fix Issues (`dev-strands`, re-run if fixes requested)

```
Agent(
  subagent_type="dev-strands",
  prompt="""
Fix specific code review issues in an Incident Triage child agent.

## Repo
{code_root}/incident-triage-{agent_name}-agent

## Issues to Fix
{paste_exact_issues_from_reviewer_at_requested_severity}

Fix each issue:
1. Read the file at the cited location
2. Apply the minimal fix (do not refactor unrelated code)
3. Run quality checks after all fixes:
```bash
make format
make lint
make mypy
make tests
make coverage
```
4. Fix any new issues introduced by the changes
5. Commit fixes:
```bash
git add .
git commit -m "fix: address code review findings

{list_of_fixed_issues}

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
git push
```
"""
)
```

After fixes are pushed, ask the user:

```
AskUserQuestion(
  questions=[
    "Fixes applied and pushed. Should we run another review pass to verify,
     or proceed to Terraform infrastructure? (A) Re-review  (B) Proceed to Terraform"
  ]
)
```

- If **(A)**: repeat Step 4.5 → 4.6 → 4.7 as needed (continue until user is satisfied or no issues remain)
- If **(B)**: proceed to Step 5

---

## Step 5: Launch Terraform Agent (`specialist-terraform`)

```
Agent(
  subagent_type="specialist-terraform",
  prompt="""
Add infrastructure for a new Incident Triage child agent to the existing
incident-triage-infrastructure Terraform codebase.

## New Agent Spec
- Agent name: {agent_name}
- AgentCore runtime name: incident_triage_{agent_name}_nonprod (underscores)
- ECR repo: incident-triage-{agent_name}-agent (hyphens)
- Cox CI ID: {ci_id} (use CI_TBD if not yet assigned — Terraform apply will fail until real value)
- Data source: {data_source}
- Auth method: {auth_method}
- AWS account (nonprod): 588875461441
- Region: us-east-1

## Paths
- Infrastructure repo: {infra_root}
- Plan file with exact line references: {workspace_root}/{agent_name}_PLAN.md

## GitHub
- Host: ghe.coxautoinc.com
- Org: SRE
- Feature branch: feature/{ticket_id}-{agent_name}-agent-infra
- Base: main

## Task: Phase 6

**Before touching any file, read:**
- {infra_root}/agentcore/terraform/main.tf
- {infra_root}/agentcore/terraform/modules/runtime/main.tf
- {infra_root}/agentcore/terraform/modules/identity/main.tf
- {infra_root}/agentcore/terraform/modules/inference_profiles/main.tf
- {infra_root}/agentcore/terraform/vars/awscaisreainp/us-east-1.tfvars
- {infra_root}/agentcore/terraform/vars/awscaisreai/us-east-1.tfvars
Read the plan at {workspace_root}/{agent_name}_PLAN.md for exact line references.

**Changes to make:**

6.1 main.tf:
- Add `aws_ecr_repository` for `incident-triage-{agent_name}-agent`
- Add `{agent_name} = "{ci_id}"` to `ci_ids` variable default
- Add `{agent_name}_image` and `{agent_name}_inference_profile_arn` input variables
- Pass new vars to `module "runtime"` and `module "inference_profiles"`

6.2 modules/identity/main.tf:
- Add IAM statement for {agent_name} data source access
- For AWS services: scope to specific ARN patterns, never `resources = ["*"]`
- For external APIs: `secretsmanager:GetSecretValue` on `incident-triage/{agent_name}/*`

6.3 modules/inference_profiles/main.tf:
- Add `{agent_name}` to `agent_models` variable (same model as splunk/newrelic)
- Add `{agent_name}_inference_profile_arn` output
- Update atomically with ci_ids in 6.1

6.4 modules/runtime/main.tf:
- Local: `{agent_name}_endpoint_name = "stable_${{var.env_name}}"`
- `aws_bedrockagentcore_agent_runtime.{agent_name}` (runtime_name uses underscores)
- `aws_bedrockagentcore_agent_runtime_endpoint.{agent_name}`
- `aws_ssm_parameter.{agent_name}_agent_arn`: `/incident-triage/${{var.env_name}}/agents/{agent_name}/arn`
- Update `aws_ssm_parameter.all_child_agents`: add new entry
- CloudWatch log group data sources (mirror existing pattern)
- Orchestrator env vars for the new agent ARN/endpoint
- Output for `{agent_name}_agent_arn`

6.5 tfvars:
- `vars/awscaisreainp/us-east-1.tfvars`: add `{agent_name}_initial_image = "588875461441.dkr.ecr.us-east-1.amazonaws.com/incident-triage-{agent_name}-agent:latest"`
- `vars/awscaisreai/us-east-1.tfvars`: add prod equivalent

**Validate:**
```bash
cd {infra_root}
make check
```

**Commit and push:**
```bash
git checkout -b feature/{ticket_id}-{agent_name}-agent-infra
git add agentcore/terraform/
git commit -m "feat: add {AgentName} agent infrastructure

- ECR repository
- Bedrock AgentCore Runtime + endpoint
- Application Inference Profile (CI ID: {ci_id})
- IAM permissions for {data_source}
- SSM parameter registration (updates all_child_agents)
- Orchestrator env vars

Rally: {ticket_id}

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
git push -u origin feature/{ticket_id}-{agent_name}-agent-infra
```

Then create PR with `gh pr create --hostname ghe.coxautoinc.com`.
"""
)
```

---

## Step 6: Launch Orchestrator Updater (`prompt-expert`)

```
Agent(
  subagent_type="prompt-expert",
  prompt="""
Add routing guidance for a new child agent to the Incident Triage orchestrator system prompt.

## New Agent Details
- Agent name: {agent_name}
- Data source: {data_source}
- Tools it exposes: {tool_1}, {tool_2}, {tool_3}
- Required inputs: {required_inputs}
- What it returns: {return_description}

## File to Edit
{orchestrator_root}/src/configuration/orchestrator_agent.md

## Task
1. Read the orchestrator system prompt
2. Find the section that describes when to invoke each child agent (Splunk, NewRelic, etc.)
3. Add an equivalent section for {agent_name}:
   - When to invoke (what types of questions / services / signals)
   - What inputs to pass (required + optional)
   - What it returns
4. Match the style and tone of the existing agent sections exactly
5. Save the updated file

Constraints:
- Do NOT change anything else in the prompt
- Do NOT reference implementation details (ARNs, SSM paths, etc.) — routing guidance only
- Follow the prompt economy principle: no JSON schema repetition for edge cases
"""
)
```

---

## Step 7: Summary

Report to the user:

```
## Create Triage Agent — Done

Agent: incident-triage-{agent_name}-agent
Rally: {ticket_id}

Completed by sub-agents:
- [x] Plan: {workspace_root}/{agent_name}_PLAN.md  (expert-agentcore)
- [x] Code: ghe.coxautoinc.com/SRE/incident-triage-{agent_name}-agent  (dev-strands)
- [x] Infra PR: incident-triage-infrastructure  (specialist-terraform)
- [x] Orchestrator prompt updated  (prompt-expert)

Prerequisites before deploying:
1. Get Cox CI ID assigned if CI_TBD was used — blocks Terraform apply
2. Configure GitHub repo variables: ECR_REPO, AGENT_RUNTIME_NAME
3. Add GHA Environment secrets for the `nonprod` environment in the new repo:
   - UV_INDEX_CAI_JFROG_USERNAME
   - UV_INDEX_CAI_JFROG_PASSWORD
   (Without these the GHA build will fail — Docker build can't pull from Artifactory)
4. Grant the new repo AWS access so GHA can assume AWS roles (required for ECR push and deployment).
   Follow the instructions at:
   https://ghe.coxautoinc.com/ETS-CloudAutomation/github-actions-cookbook/blob/main/docs/granting-repo-access-to-an-aws-account.md

Deployment order (chicken-and-egg: ECR must exist before Runtime, image must exist before Runtime):

Step 1 — Apply infra (creates ECR, WILL FAIL on AgentCore Runtime — image not yet in ECR):
  Merge infra PR → GHA deploy-nonprod runs → expect failure on aws_bedrockagentcore_agent_runtime

Step 2 — Build and push image (will fail to register Runtime — it doesn't exist yet, but image IS pushed):
  Trigger GHA build-agent-image workflow on incident-triage-{agent_name}-agent → image lands in ECR

Step 3 — Re-apply infra (ECR has image now, Runtime creation succeeds):
  Re-run deploy-nonprod workflow (or push a no-op commit to main) → Runtime created successfully

Step 4 — Deploy agent to Runtime:
  Trigger GHA deploy-agent workflow → agent registered and running

Step 5 — Verify:
  Wait 5 min after deploy, test with a NEW session (session affinity — old sessions hit old containers)
```
