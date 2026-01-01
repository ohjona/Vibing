# v1.2 Changelog

> Implementation decisions and rationale for the v1.2 restructure
> Date: January 2025

---

## Summary

v1.2 restructures the LLM Development Playbook from **sequential convergence** to **parallel fan-out**, reducing orchestration overhead by ~50%.

| Metric | v1.1 | v1.2 |
|--------|------|------|
| Handoffs per feature | 12-16 | 6-8 |
| Architect roles | 4 | 2 |
| Review iterations | 3-5 rounds | 1 revision cap |
| Decision gates | Many | 2 clear gates |

---

## Key Changes

### Parallel Fan-Out Pattern
- **Before:** Critics review sequentially, wait for convergence (3-5 rounds)
- **After:** Critics review IN PARALLEL (Claude + Gemini + GPT), consolidate, one revision

### Role Consolidation
| v1.1 | v1.2 |
|------|------|
| Strategist | Folded into Architect |
| Product Architect | Folded into Architect |
| Implementation Architect | Renamed to Architect |
| Chief Architect | Kept (major decisions only) |
| Synthesizer | Replaced by Consolidator |

### One Revision Cap
If major issues persist after one revision:
- Requirements were unclear → Fix requirements, don't iterate
- Scope is too large → Split the feature
- Fundamental disagreement → Human decides

### Two Decision Gates
1. **Proceed to Development?** — After spec revision
2. **Merge?** — After code review consolidation

---

## New Files

### session-template.md
Stateful document to copy per feature. Provides:
- Progress tracking with checkboxes
- Ready-to-copy prompts with variable placeholders
- Clear phase progression
- Session log

---

## Mechanisms Preserved

All quality mechanisms from v1.1 survive:

| Mechanism | Location |
|-----------|----------|
| YAGNI constraints | Architect Prompt, Adversarial prompts |
| Boring Tech defaults | Architect Prompt |
| Cut List requirement | Architect Prompt |
| ComplexityFlag authority | Developer Prompt |
| Complexity audit FIRST | Adversarial Critic + Reviewer |
| Side with pessimist | Consolidator Rules |
| 300+ word minimum | Adversarial prompts |
| Anti-sycophancy | Playbook reference section |
| Emergency procedures | All 7 in Handbook |

---

## Consolidator Rules

Added to ensure "side with pessimist" is enforced:

1. If Adversarial Critic says REJECT → NEEDS HUMAN DECISION
2. If 2+ critics say MAJOR CONCERNS → BLOCK
3. Security/correctness from ANY critic → Critical Issues
4. Complexity concerns from Adversarial → dedicated section

---

*Generated during v1.2 implementation.*
