# RDPI Bootstrap — Decision Log

Tracks key design decisions, alternatives considered, and rationale. Updated as the spec evolves.

---

## D-001: 4 Skills Instead of 8 QRSPI Stages

**Decision:** Collapse QRSPI's 8 stages into 4 top-level skills (Research, Design, Plan, Implement).

**Alternatives considered:**
- 8 separate skills (one per QRSPI stage) — maximum context isolation
- 6 skills (merge Questions+Research, merge Worktree+Implement) — middle ground

**Rationale:**
- Context isolation where it matters most: between R→D→P→I. Within a phase, sub-agents provide additional isolation.
- UX simplicity: 4 commands vs 8.
- Orchestrator overhead within a phase is manageable — interview context is high-signal user input, not LLM noise.

**Tradeoff:** Research phase accumulates interview + blind research results in one context. Accepted because the blind sub-agent runs in its own context, and the interview is user-driven (high quality tokens).

**Source:** Devil's advocate review round 1 (C-1), confirmed by discussion.

---

## D-002: Spec Artifact (Our Extension)

**Decision:** Research phase produces a `spec.md` artifact that serves as the contract for subsequent phases.

**Alternatives considered:**
- No Spec — Design doc serves as the requirements + architecture contract (Dex's approach)
- Spec merged into Design

**Rationale:** Having an explicit requirements document before Design allows the user to verify mutual understanding at the requirements level, separate from architecture. It also enables complexity-based phase skipping — if Design is skipped, the Spec still anchors the Plan.

**Tradeoff:** Extra artifact that Dex doesn't describe. Marked as optional for user review.

**Source:** Discussion about C-2 from devil's advocate review round 1.

---

## D-003: Single Design + Critic, Not 3 Perspectives

**Decision:** One design document + one critic review (opt-in for Full depth), instead of 3 perspective agents (conservative, pragmatist, harsh critic).

**Alternatives considered:**
- 3 perspective agents + synthesis (original v2 spec)
- Single design, no critic at all

**Rationale:**
- 3 agents violate instruction budget (~40 per skill) and context minimization.
- Dex describes Design as an interactive document, not a waterfall of sub-agents.
- One critic gives 80% of the value at 33% of the cost.

**Tradeoff:** Less diverse perspectives. Mitigated by the interactive nature of Design (human provides the second perspective).

**Source:** Devil's advocate review round 1 (C-3), confirmed by discussion. Later simplified further in round 2 (D-3, D-11) — critic made opt-in, not default.

---

## D-004: QRSPI Naming (Not CRISPY)

**Decision:** Use "QRSPI" as the methodology name.

**Alternatives considered:**
- "CRISPY" — how the devil's advocate interpreted the automatic transcript

**Rationale:** The actual presentation slides read "QRSPI". The automatic transcript phonetically transcribed it as "CRISPY". Confirmed by user who has access to the slides.

**Source:** User correction after devil's advocate review round 2 (D-1).

---

## D-005: Remove Review Agents from Research and Design

**Decision:** No sub-agent reviews in Research or Design phases. Plan phase keeps a spec-consistency checker.

**Alternatives considered:**
- Review agents at every phase (original v2-v3 spec)
- No reviews anywhere

**Rationale:**
- Research produces facts — nothing to "review" with opinions.
- Design is interactive (human shapes it) — human IS the reviewer.
- Plan is not read by human — spec-consistency check is the quality gate.
- Removing review agents cuts ~5-7 sub-agent calls from default flow, respects instruction budget.

**Source:** Discussion about D-2, D-11 from devil's advocate review round 2. Aligned with Dex's simplification message.

---

## D-006: Worktree and PR Folded into Implement

**Decision:** QRSPI's separate Worktree and PR stages are sub-steps of our Implement phase.

**Alternatives considered:**
- Separate `/rdpi-worktree` and `/rdpi-pr` skills

**Rationale:** UX simplicity. Worktree setup is 3 git commands. PR creation is one sub-agent call. Neither justifies a separate skill with its own context clearing. Acknowledged as a deviation from QRSPI.

**Source:** Discussion about D-7, D-8 from devil's advocate review round 2.

---

## D-007: Structure/Outline Can Include Types and Signatures

**Decision:** Structure starts high-level but allows drilling into types, function signatures, and file paths when the user asks.

**Alternatives considered:**
- "No code, no file paths" restriction (original v3 spec)

**Rationale:** Dex explicitly compares Structure to C header files — "the signatures and the new types." Prohibiting code contradicts this. The user should be able to ask for more detail.

**Source:** Devil's advocate review round 2 (D-4), direct quote from transcript.

---

## D-008: Instruction Budget — 40 Per Skill, 150-200 Total

**Decision:** Each phase SKILL.md aims for under 40 instructions. The 150-200 number is the total across all instruction sources in a context window.

**Alternatives considered:**
- 150-200 per skill, 500 lines limit (original v3 spec)

**Rationale:** Dex says "before it was 85 instructions, now they're all less than 40." The 150-200 is the total model capacity across system prompt + tools + MCP + CLAUDE.md + skill combined. Each individual skill should be a fraction of that.

**Source:** Devil's advocate review round 2 (D-6), direct quote from transcript.

---

## D-009: Interactive Design (Not Waterfall)

**Decision:** Design phase is an interactive brain-dump where human and agent shape the document together, not a pipeline of sub-agents.

**Alternatives considered:**
- Design agent → Critic → Spec Consistency → Structure (waterfall, v3 spec)

**Rationale:** Dex describes Design as "taking cloud code plan mode and the ask user question tool and just brain dumping it all to a single document that you can interact with." It's moldable and flexible, not a sequence of automated steps.

**Source:** Devil's advocate review round 2 (D-3), direct quote from transcript.

---

## D-010: Model Selection Per Agent Role

**Decision:** Each agent role gets a default model + thinking level assignment. Opus for design/orchestration, Sonnet for coding/planning, Haiku for monitoring.

**Alternatives considered:**
- One model for everything
- Let each agent pick its own model

**Rationale:** Right tool for the right job. Design decisions need Opus + extended thinking. Code from a plan is mechanical — Sonnet is sufficient. Log reading is simple — Haiku saves cost. Bootstrap asks about model limits and budget.

**Source:** Discussion about model selection, user confirmed the table.

---

## D-011: Draft PR for Code Review

**Decision:** Implementation creates a Draft PR. User reviews code there, promotes to Ready when satisfied.

**Alternatives considered:**
- Regular PR (no draft stage)
- Local code review only

**Rationale:** Draft PR gives the user a familiar code review interface without publishing to the team prematurely. Dex says "read the code" — PR interface is the natural place for that.

**Source:** Discussion about D-8 and Principle 12, our extension.

---

## D-012: Inter-Slice Review — Mandatory First, Configurable After

**Decision:** After the first vertical slice, user review is mandatory. For subsequent slices, it's configurable (user can opt for auto-proceed).

**Alternatives considered:**
- Mandatory after every slice
- Never mandatory (user pulls when ready)

**Rationale:** First slice establishes direction — catching errors early is high-leverage. Subsequent slices in the same pattern are lower-risk. Dex says checkpoints are for "sensitive or hard or complex" work, not mandatory everywhere.

**Source:** Devil's advocate review round 2 (D-12), discussion.
