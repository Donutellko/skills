# Devil's Advocate Review — Round 1

**Date:** 2026-03-30
**Reviewed:** SPEC.md v2 vs DEX_HORTHY_VIDEO.md + DMIRTY_BEREZNITSKI_VIDEO.md
**Agent:** devils-advocate sub-agent

## Findings and Decisions

### C-1: 4 Skills Instead of 8 — Context Pollution Risk | HIGH
**Finding:** Combining Q+R+Spec in one phase accumulates context.
**Decision:** ACCEPTED as trade-off. Documented in SPEC.md "Why 4 Skills" section. Sub-agents provide isolation within phases.
→ See DECISIONS.md D-001

### C-2: Spec Artifact — Redundant with Design? | HIGH
**Finding:** QRSPI has no "Spec" stage. Design doc IS the sync point.
**Decision:** KEPT Spec as our extension, but made it optional for user review. Design is now interactive (not a review artifact). Structure is mandatory review.
→ See DECISIONS.md D-002

### C-3: 3 Perspective Agents — Overkill | HIGH
**Finding:** 3 designs + synthesis violates instruction budget and context minimization.
**Decision:** SIMPLIFIED to single design + one critic (opt-in). Later further simplified — critic removed from default flow.
→ See DECISIONS.md D-003

### C-4: "No Magic Prompts" Missing | HIGH
**Finding:** Not in spec.
**Decision:** ADDED as Principle 10 (merged with Pipeline as Control).

### C-5: "Read the Code" Missing | HIGH
**Finding:** Human should read code, not just artifacts.
**Decision:** ADDED as Principle 12. Code review in Draft PR.

### C-6: Plan — Review Agents but User Doesn't Read | MEDIUM
**Finding:** Contradiction: plan not for humans, but has review agents.
**Decision:** SIMPLIFIED. Plan review is internal feedback loop. User sees only summary + high-level issues from spec-consistency check.

### C-7: No "Restart Phase" Guidance | MEDIUM
**Finding:** Bad trajectory advice missing.
**Decision:** ADDED as Principle 11 (Bad Trajectory Recovery).

### C-8: QRSPI vs CRISPY Naming Confusion | MEDIUM
**Finding:** Naming inconsistency.
**Decision:** Added Terminology section. Later confirmed QRSPI from slides.
→ See DECISIONS.md D-004

### C-9: No Complexity Scaling | MEDIUM
**Finding:** Always runs full 4 phases regardless of task size.
**Decision:** ADDED Complexity-Based Phase Skipping section. Research recommends next step based on complexity.

### C-10: Instruction Budget Not Enforced | MEDIUM
**Finding:** Stated but not measured.
**Decision:** ADDED concrete limit: SKILL.md < 500 lines. Later corrected to ~40 instructions per skill.
→ See DECISIONS.md D-008

### C-11: Blind Research — Not Truly Blind | LOW
**Finding:** Questions encode task assumptions.
**Decision:** ACKNOWLEDGED in Principle 7 as a known limitation. Research agent also reports unexpected findings.

### C-12: Code Review Less Important Than Earlier Reviews | LOW
**Finding:** Spec treats all review points equally.
**Decision:** Noted. Code review at implementation time should focus on correctness/style, not architecture. Not yet added to spec.

### C-13: "Pipeline as Control" Not Explicit | LOW
**Finding:** Implicit but not stated.
**Decision:** ADDED as part of Principle 10.

### C-14: No Realistic Expectations | LOW
**Finding:** 2-3x boost expectation missing.
**Decision:** ADDED to bootstrap output (step 8).
