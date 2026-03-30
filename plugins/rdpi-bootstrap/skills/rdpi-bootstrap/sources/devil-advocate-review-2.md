# Devil's Advocate Review — Round 2

**Date:** 2026-03-30
**Reviewed:** SPEC.md v3 vs DEX_HORTHY_VIDEO_2_original_transcript.txt
**Agent:** devils-advocate sub-agent

## Findings and Decisions

### D-1: CRISPY vs QRSPI Naming | HIGH
**Finding:** Transcript phonetically sounds like "CRISPY". Agent claimed QRSPI is our invention.
**Decision:** REJECTED. User confirmed slides read "QRSPI". Automatic transcript was misleading.
→ See DECISIONS.md D-004

### D-2: Plan Phase Over-Engineered | HIGH
**Finding:** Elaborate artifact structure (per-slice-per-role plans, reviews) contradicts Dex's "plan is for the agent."
**Decision:** ACCEPTED. Simplified to: one plan.md + spec-consistency.md + summary.md. Removed per-role plans and review agents.

### D-3: Design Phase — Waterfall vs Interactive | HIGH
**Finding:** Dex describes Design as interactive brain-dump, not a pipeline of sub-agents.
**Decision:** ACCEPTED. Redesigned Design as interactive document. Removed Critic and Spec Consistency sub-agents from Design. Kept Structure/Outline sub-agent (separate context).
→ See DECISIONS.md D-009

### D-4: Structure Can Include Types/Signatures | HIGH
**Finding:** Dex compares Structure to C header files. Spec said "no code, no file paths."
**Decision:** ACCEPTED. Removed the restriction.
→ See DECISIONS.md D-007

### D-5: Design and Structure in Different Context Windows | MEDIUM
**Finding:** Dex separates them. Spec combines in one phase.
**Decision:** ACKNOWLEDGED as trade-off. Structure sub-agent runs in its own context within the Design phase, providing the separation.

### D-6: Per-Stage Budget Is ~40, Not 150-200 | MEDIUM
**Finding:** 150-200 is total across all sources. Each stage should be under 40.
**Decision:** ACCEPTED. Updated Principle 6.
→ See DECISIONS.md D-008

### D-7: Worktree as Separate Stage | MEDIUM
**Finding:** QRSPI has it as a distinct stage.
**Decision:** KEPT inside Implement, acknowledged deviation.
→ See DECISIONS.md D-006

### D-8: PR as Separate Stage | MEDIUM
**Finding:** QRSPI has it as a distinct stage.
**Decision:** KEPT inside Implement, acknowledged deviation. Added Draft PR workflow.
→ See DECISIONS.md D-006, D-011

### D-9: Artifacts Replace Compaction | MEDIUM
**Finding:** Dex says artifacts make auto-compaction unnecessary.
**Decision:** ACCEPTED. Added as Principle 2 (Artifacts Replace Compaction).

### D-10: Spec Writing May Reintroduce Bias | MEDIUM
**Finding:** Main agent combines task description + blind research when writing Spec.
**Decision:** ACKNOWLEDGED. Labeled Spec as our extension. Bias is limited because the Spec synthesizes (not researches) — the research was already done blindly.

### D-11: Sub-Agent Orchestration Bloats Instructions | MEDIUM
**Finding:** Critic, Spec Consistency, multi-agent reviews add to instruction count.
**Decision:** ACCEPTED. Removed from default flow. Critic opt-in for Full depth. Spec Consistency only in Plan phase.
→ See DECISIONS.md D-005

### D-12: Mandatory Review After Every Slice | LOW
**Finding:** Dex says checkpoints are optional, not mandatory.
**Decision:** ACCEPTED. Mandatory for first slice, configurable after.
→ See DECISIONS.md D-012

### D-13: Team Pre-Alignment Not Operationalized | LOW
**Finding:** Design docs should be shared with code owners.
**Decision:** DEFERRED to backlog (B-008).

### D-14: MCP Tool Bloat | LOW
**Finding:** Loading all discovered agents bloats context.
**Decision:** ACCEPTED. Added Principle 7 (Minimal Tool Surface Per Phase).

### D-15: Spec Fills in Unspecified Details | LOW
**Finding:** Much of the spec is extrapolation beyond what Dex discussed.
**Decision:** ACCEPTED. Added "Our Extensions Beyond QRSPI" section to clearly mark what's ours vs Dex's.

### D-16: Complexity-Based Phase Skipping | LOW
**Finding:** Not from Dex. Our addition.
**Decision:** ACKNOWLEDGED, marked as our extension.

### D-17: Non-Biased Research Nuance | LOW
**Finding:** Observation about question-encoded bias is ours, not Dex's.
**Decision:** ACKNOWLEDGED in Principle 7.
