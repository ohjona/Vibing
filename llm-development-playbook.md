# LLM Development Playbook: Multi-Agent Review Workflow

**Version:** 2.0
**Updated:** 2026-01-15
**Based on:** Ontos v3.0 Development Process (Phases 0-5, battle-tested)

---

## Table of Contents

1. [Overview](#1-overview)
2. [Core Principles](#2-core-principles)
3. [Model Assignments](#3-model-assignments)
4. [Role Definitions](#4-role-definitions)
5. [Workflow Phases](#5-workflow-phases)
6. [Phase A: Spec Development](#6-phase-a-spec-development)
7. [Phase B: Spec Review](#7-phase-b-spec-review)
8. [Phase C: Implementation](#8-phase-c-implementation)
9. [Phase D: Code Review](#9-phase-d-code-review)
10. [Prompt Engineering](#10-prompt-engineering)
11. [Output Standards](#11-output-standards)
12. [File Naming Conventions](#12-file-naming-conventions)
13. [Phase Variants](#13-phase-variants)
14. [Pre-Phase Documentation](#14-pre-phase-documentation)
15. [Automation Architecture](#15-automation-architecture)
16. [Troubleshooting](#16-troubleshooting)
17. [Quick Reference](#17-quick-reference)

---

## 1. Overview

### 1.1 What This Is

A **multi-agent LLM workflow** for software development that uses intentional model diversity and role differentiation to catch issues that single-model workflows miss.

### 1.2 The Problem

When one LLM instance does everything (spec, review, implement, test):

| Problem | Symptom | Impact |
|---------|---------|--------|
| **Blind spots** | Can't see own assumptions | Bugs in "obvious" parts |
| **Rubber-stamping** | Approves own work | Issues pass review |
| **Role fatigue** | Perspective narrows over time | Quality degrades |
| **Shared biases** | Same model, same biases | Systematic errors |

### 1.3 The Solution

**Two types of diversity working together:**

| Diversity Type | What It Means | What It Catches |
|----------------|---------------|-----------------|
| **Role Diversity** | Different review focuses (peer, adversarial, alignment) | Different categories of issues |
| **Model Diversity** | Different AI models (Claude, Gemini, Codex) | Different blind spots, biases |

**Key insight:** Role diversity alone isn't enough. The same model playing different roles still shares underlying biases. Model diversity ensures genuinely independent perspectives.

### 1.4 The Workflow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        COMPLETE DEVELOPMENT CYCLE                        │
└─────────────────────────────────────────────────────────────────────────┘

PHASE A              PHASE B                PHASE C              PHASE D
Spec Development     Spec Review            Implementation       Code Review
     │                   │                       │                    │
     ▼                   ▼                       ▼                    ▼
┌─────────┐        ┌───────────┐          ┌───────────┐        ┌───────────┐
│  Chief  │        │  Review   │          │   Chief   │        │   Chief   │
│Architect│───────▶│   Board   │          │ Architect │        │ Architect │
│ (Claude)│        │(3 models) │          │  writes   │        │ PR Review │
└─────────┘        └─────┬─────┘          │  prompt   │        └─────┬─────┘
     │                   │                └─────┬─────┘              │
     ▼                   ▼                      │                    ▼
┌─────────┐        ┌───────────┐                │              ┌───────────┐
│  Spec   │        │Consolida- │                │              │  Review   │
│  v1.0   │        │   tion    │                │              │   Board   │
└─────────┘        └─────┬─────┘                │              │(3 models) │
                         │                      │              └─────┬─────┘
                         ▼                      │                    │
                   ┌───────────┐                │                    ▼
                   │   Chief   │                │              ┌───────────┐
                   │ Architect │                │              │Consolida- │
                   │ Response  │                │              │   tion    │
                   └─────┬─────┘                │              └─────┬─────┘
                         │                      │                    │
                         ▼                      ▼                    ▼
                   ┌───────────┐          ┌───────────┐        ┌───────────┐
                   │   Spec    │          │Antigravity│        │Antigravity│
                   │   v1.1    │          │implements │        │  Fixes    │
                   └───────────┘          └─────┬─────┘        └─────┬─────┘
                                                │                    │
                                                ▼                    ▼
                                          ┌───────────┐        ┌───────────┐
                                          │    PR     │        │  Codex    │
                                          │  Created  │        │Verification│
                                          └───────────┘        └─────┬─────┘
                                                                     │
                                                                     ▼
                                                               ┌───────────┐
                                                               │   Chief   │
                                                               │ Architect │
                                                               │  Approval │
                                                               └─────┬─────┘
                                                                     │
                                                                     ▼
                                                               ┌───────────┐
                                                               │   MERGE   │
                                                               └───────────┘
```

---

## 2. Core Principles

### 2.1 Model Diversity is Non-Negotiable

**Don't use the same model for all roles.** Even with different prompts, the same model has the same blind spots.

| ❌ Same Model | ✅ Different Models |
|--------------|---------------------|
| Claude reviews Claude's spec | Gemini reviews Claude's spec |
| Codex verifies Codex's code | Different perspective on same code |
| Shared biases pass through | Independent verification |

**Minimum diversity:** At least 2 different model families in every review board.

### 2.2 One Revision Cap

**Prevent infinite convergence loops.** Each review cycle allows ONE round of revisions.

| Round | Purpose | If Issues Remain |
|-------|---------|------------------|
| Round 1 | Full review — find all issues | Consolidate, fix |
| Round 2 | Verification — confirm fixes | Escalate or accept |
| Round 3+ | **Avoid** — indicates deeper problem | Re-scope, re-spec |

**Why this matters:** Iteration without convergence wastes tokens and time. If two rounds don't resolve issues, the problem is usually scope or requirements, not execution.

### 2.3 Role Differentiation

**Different roles catch different issues.** Don't use the same prompt for all reviewers.

| Role | Focus | Question |
|------|-------|----------|
| **Peer** | Quality, completeness, UX | "Is this good?" |
| **Alignment** | Compliance, constraints | "Does this follow the rules?" |
| **Adversarial** | Breaking, edge cases | "How does this fail?" |

### 2.4 Document-Driven Review

**Anchor reviews to approved documents, not author framing.**

| ❌ Author-Selected Focus | ✅ Document-Driven |
|-------------------------|-------------------|
| "Review these 7 areas I identified" | "Verify alignment with Architecture v1.4" |
| "Focus on technical correctness" | "Check against Roadmap Section 6" |
| Author steers away from weak areas | Documents define complete scope |

**Why:** Authors know where they're uncertain (and steer away). They don't know where they're overconfident (where the bugs are).

### 2.5 Explicit Permission to Disagree

**Reviewers mirror the prompt's energy.**

| Prompt Energy | Reviewer Response |
|---------------|-------------------|
| "Please review and let me know" | Polite approval |
| "Find the problems in this" | Actual critique |
| "What's the architect not seeing?" | Hidden issues surfaced |

**Default to critique-inviting language.** See [Section 10: Prompt Engineering](#10-prompt-engineering).

### 2.6 Git as Source of Truth

**PR comments are the official record.** Markdown files are backups.

| Output | Location | Purpose |
|--------|----------|---------|
| Code reviews | PR comments | Official record, visible to all |
| Spec reviews | `.ontos-internal/` | Reference, backup |
| Approvals | PR comments | Audit trail |

---

## 3. Model Assignments

### 3.1 Default Assignments

| Role | Default Model | Why |
|------|---------------|-----|
| **Chief Architect** | Claude Opus 4.5 | Strong reasoning, synthesis, planning |
| **Peer Reviewer** | Gemini 2.5 Pro | Research, UX perspective, documentation |
| **Alignment Reviewer** | Claude Opus 4.5 | Constraint checking, consistency |
| **Adversarial Reviewer** | Codex (OpenAI) | Edge cases, breaking things, security |
| **Developer (Antigravity)** | Codex (OpenAI) | Implementation, code generation |

### 3.2 Assignment Flexibility

**Override defaults based on context:**

| Situation | Override |
|-----------|----------|
| Security-critical phase | Codex as Alignment (security focus) |
| UX-heavy phase | Gemini as Peer + Alignment |
| Complex architecture | Claude for all review roles (accept reduced diversity) |
| Research-heavy spec | Gemini as Chief Architect |

### 3.3 Minimum Diversity Requirements

| Review Board | Minimum |
|--------------|---------|
| Spec Review (3 reviewers) | At least 2 different model families |
| Code Review (3 reviewers) | At least 2 different model families |
| Verification | Different model than original reviewer |

### 3.4 Model Strengths Reference

| Model | Strengths | Best For |
|-------|-----------|----------|
| **Claude** | Reasoning, planning, long context, nuance | Architecture, complex decisions, synthesis |
| **Gemini** | Research, current info, multimodal, UX | Documentation, user-facing review, research |
| **Codex** | Code generation, debugging, edge cases | Implementation, adversarial review, testing |

---

## 4. Role Definitions

### 4.1 Chief Architect

**The decision-maker and synthesizer.**

| Responsibility | Actions |
|----------------|---------|
| Write specs | Create implementation specifications |
| Make decisions | Resolve open questions, choose approaches |
| Respond to reviews | Accept/modify/reject review findings |
| Write implementation prompts | Translate specs into developer instructions |
| Final approval | Authorize merges |

**Model:** Claude Opus 4.5 (default)

**Key outputs:**
- Spec v1.0, v1.1
- CA Response documents
- Implementation prompts
- Final approval

### 4.2 Peer Reviewer

**The quality and completeness checker.**

| Focus | Questions |
|-------|-----------|
| Completeness | Is anything missing? |
| Quality | Is this well-designed? |
| Clarity | Can this be implemented without questions? |
| UX | Will users understand this? |

**Model:** Gemini 2.5 Pro (default)

**Key outputs:**
- Spec reviews (quality focus)
- Code reviews (quality focus)
- Research findings

### 4.3 Alignment Reviewer

**The compliance and consistency checker.**

| Focus | Questions |
|-------|-----------|
| Architecture | Does this match the architecture? |
| Roadmap | Does this implement what was planned? |
| Constraints | Are layer constraints respected? |
| Consistency | Is this consistent with prior decisions? |
| Backward compatibility | Does this break existing behavior? |

**Model:** Claude Opus 4.5 (default)

**Key outputs:**
- Spec reviews (alignment focus)
- Code reviews (compliance focus)
- Deviation reports

### 4.4 Adversarial Reviewer

**The breaker and edge case finder.**

| Focus | Questions |
|-------|-----------|
| Failure modes | How does this break? |
| Edge cases | What inputs cause problems? |
| Security | What can be exploited? |
| Assumptions | What assumptions might be wrong? |
| Regressions | Could this break existing functionality? |

**Model:** Codex (OpenAI) (default)

**Key outputs:**
- Spec reviews (attack focus)
- Code reviews (breaking focus)
- Verification reviews

### 4.5 Developer (Antigravity)

**The implementer.**

| Responsibility | Actions |
|----------------|---------|
| Follow prompts | Execute implementation instructions literally |
| Write code | Implement features as specified |
| Write tests | Add tests for new functionality |
| Fix issues | Address review findings |
| Report blockers | Flag unclear instructions |

**Model:** Codex (OpenAI) (default)

**Key outputs:**
- Working code
- Tests
- Fix summaries
- PR descriptions

### 4.6 Consolidator

**The synthesizer of reviews.**

| Responsibility | Actions |
|----------------|---------|
| Read all reviews | Understand each reviewer's findings |
| Identify consensus | What do reviewers agree on? |
| Surface disagreements | Where do reviewers differ? |
| Prioritize issues | What's blocking vs. nice-to-have? |
| Create action items | Clear instructions for next step |

**Model:** Any (often the human orchestrator, or Claude)

**Key outputs:**
- Consolidation documents
- Action item lists

---

## 5. Workflow Phases

### 5.1 Phase Overview

| Phase | Purpose | Input | Output |
|-------|---------|-------|--------|
| **A: Spec Development** | Define what to build | Requirements, roadmap | Spec v1.0 |
| **B: Spec Review** | Validate the spec | Spec v1.0 | Spec v1.1 (approved) |
| **C: Implementation** | Build it | Spec v1.1 | PR with code |
| **D: Code Review** | Validate the code | PR | Merged code |

### 5.2 Phase Dependencies

```
A ──────▶ B ──────▶ C ──────▶ D ──────▶ Merge
          │                   │
          │                   │
          ▼                   ▼
     If major issues     If major issues
     return to A         return to C
```

### 5.3 Phase Timing (Typical)

| Phase | Duration | Parallel Work? |
|-------|----------|----------------|
| A: Spec Development | 1-4 hours | No |
| B: Spec Review | 1-2 hours | Yes (3 reviewers parallel) |
| C: Implementation | 2-8 hours | No |
| D: Code Review | 1-2 hours | Yes (3 reviewers parallel) |

---

## 6. Phase A: Spec Development

### 6.1 Purpose

Chief Architect creates a detailed implementation specification.

### 6.2 Inputs

| Input | Purpose |
|-------|---------|
| Roadmap | What to build |
| Architecture | Constraints and patterns |
| Previous phase outputs | Context |

### 6.3 Process

```
1. Review roadmap requirements
2. Check architecture constraints
3. Research open questions (if any)
4. Write spec v1.0
5. Self-review against checklist
6. Submit for Phase B
```

### 6.4 Spec Template

```markdown
# [Project] [Phase] Implementation Spec

**Version:** 1.0
**Author:** Chief Architect (Claude Opus 4.5)
**Date:** [Date]
**Status:** Draft — Pending Review

---

## 1. Overview

[What this phase delivers — 2-3 sentences]

**Target Version:** [Version number]
**Theme:** [Phase theme]
**Risk Level:** [LOW / MEDIUM / HIGH]

---

## 2. Scope

### 2.1 In Scope

- [ ] [Deliverable 1]
- [ ] [Deliverable 2]
- [ ] [Deliverable 3]

### 2.2 Out of Scope

| Item | Why Excluded |
|------|--------------|
| [Item] | [Reason] |

---

## 3. Dependencies

### 3.1 Prerequisites

- [x] [Completed prerequisite]
- [ ] [Required before implementation]

### 3.2 Blocked By

| Blocker | Status | Mitigation |
|---------|--------|------------|
| [Blocker or "None"] | | |

---

## 4. Technical Design

### 4.1 [Component/Feature 1]

**Purpose:** [What it does]

**Files:**
- `[file]` — [CREATE/MODIFY/DELETE] — [Description]

**Implementation:**
[Technical details, approach, code snippets if helpful]

### 4.2 [Component/Feature 2]

[Same structure]

---

## 5. Open Questions

Questions requiring research or decision:

| # | Question | Options | Recommendation | Status |
|---|----------|---------|----------------|--------|
| Q1 | [Question] | A: [Option], B: [Option] | [Recommendation] | Open |

---

## 6. Test Strategy

### 6.1 Unit Tests

| Component | Test Coverage |
|-----------|---------------|
| [Component] | [What to test] |

### 6.2 Integration Tests

| Scenario | Test |
|----------|------|
| [Scenario] | [Test description] |

### 6.3 Manual Testing

| Test | Expected Result |
|------|-----------------|
| [Test] | [Result] |

---

## 7. Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| [Risk] | Low/Med/High | Low/Med/High | [Mitigation] |

---

## 8. Acceptance Criteria

- [ ] [Criterion 1]
- [ ] [Criterion 2]
- [ ] All tests pass
- [ ] Documentation updated

---

*Spec Version: 1.0*
*Ready for Review Board*
```

### 6.5 Quality Checklist

Before submitting for review:

- [ ] All scope items are specific and testable
- [ ] Technical design has file-level detail
- [ ] Open questions are clearly stated
- [ ] Risks are identified with mitigations
- [ ] Acceptance criteria are measurable
- [ ] References approved documents (roadmap, architecture)

---

## 7. Phase B: Spec Review

### 7.1 Purpose

Review Board validates the spec before implementation begins.

### 7.2 Structure

```
B.1: Review Board (Parallel)
     ├── Peer Reviewer (Gemini)
     ├── Alignment Reviewer (Claude)
     └── Adversarial Reviewer (Codex)
            │
            ▼
B.2: Consolidation
            │
            ▼
B.3: Chief Architect Response
            │
            ▼
B.4: Verification (if needed)
            │
            ▼
      Spec v1.1 (Approved)
```

### 7.3 B.1: Review Board

**Three reviewers work in parallel.** Each has a different focus.

#### Peer Reviewer Prompt (Gemini)

```markdown
## Your Role

You are the **Peer Reviewer**. Review for quality, completeness, and implementability.

**Your job:** Find problems. Improve the spec. Don't validate.

---

## Focus Areas

1. **Completeness** — Is anything missing?
2. **Clarity** — Can this be implemented without questions?
3. **Quality** — Is the approach sound?
4. **Open Questions** — Research and provide recommendations

---

## Input Documents

**Under Review:**
- [Spec v1.0 path]

**Reference (must align with):**
- [Roadmap path]
- [Architecture path]

---

## Review Instructions

### Part 1: Completeness Check

| Expected Section | Present? | Adequate? |
|------------------|----------|-----------|
| Scope | ✅/❌ | ✅/❌ |
| Technical design | ✅/❌ | ✅/❌ |
| Test strategy | ✅/❌ | ✅/❌ |
| Risks | ✅/❌ | ✅/❌ |

### Part 2: Quality Assessment

| Aspect | Rating | Notes |
|--------|--------|-------|
| Technical approach | Good/Adequate/Poor | |
| Clarity | Good/Adequate/Poor | |
| Testability | Good/Adequate/Poor | |

### Part 3: Open Questions Research

For each open question in the spec:

| Question | Research Conducted | Recommendation | Confidence |
|----------|-------------------|----------------|------------|
| Q1 | [What you researched] | [Your recommendation] | High/Med/Low |

### Part 4: Issues Found

**Major (Should Fix):**
| # | Issue | Section | Suggestion |
|---|-------|---------|------------|
| P-M1 | [Issue] | [Section] | [Fix] |

**Minor (Consider):**
| # | Issue | Section | Suggestion |
|---|-------|---------|------------|
| P-m1 | [Issue] | [Section] | [Fix] |

### Part 5: Verdict

**Recommendation:** [Approve / Approve with changes / Request revision]
**Blocking issues:** [Count]

---

## Critical Instructions

1. You are not here to approve. You are here to find problems.
2. Research open questions thoroughly before recommending.
3. Be specific. "Needs work" is useless.
4. Check the spec against reference documents.
5. If something seems off but you can't articulate why, flag it anyway.

---

## Output File

Save as: `[Phase]_Spec_Review_Gemini.md`
```

#### Alignment Reviewer Prompt (Claude)

```markdown
## Your Role

You are the **Alignment Reviewer**. Verify the spec aligns with approved documents and constraints.

**Your job:** Check compliance. Flag deviations. Ensure consistency.

---

## Focus Areas

1. **Roadmap Alignment** — Does this implement what was planned?
2. **Architecture Compliance** — Are constraints respected?
3. **Consistency** — Is this consistent with prior decisions?
4. **Constraints** — Are all constraints documented and followed?

---

## Input Documents

**Under Review:**
- [Spec v1.0 path]

**Reference (authoritative):**
- [Roadmap path] — What should be built
- [Architecture path] — How it should be built

---

## Review Instructions

### Part 1: Roadmap Alignment

| Roadmap Requirement | Spec Section | Addressed? | Correctly? |
|---------------------|--------------|------------|------------|
| [Requirement 1] | [Section] | ✅/❌ | ✅/❌ |
| [Requirement 2] | [Section] | ✅/❌ | ✅/❌ |

**Deviations from Roadmap:**
| Deviation | Spec Says | Roadmap Says | Severity |
|-----------|-----------|--------------|----------|
| [Deviation or "None"] | | | |

### Part 2: Architecture Compliance

| Constraint | Spec Compliant? | Evidence |
|------------|-----------------|----------|
| [Constraint 1] | ✅/❌ | [Section ref] |
| [Constraint 2] | ✅/❌ | [Section ref] |

**Architecture Violations:**
| Violation | Spec Section | Correct Approach |
|-----------|--------------|------------------|
| [Violation or "None"] | | |

### Part 3: Consistency Check

| Prior Decision | Spec Consistent? | Notes |
|----------------|------------------|-------|
| [Decision 1] | ✅/❌ | |

### Part 4: Issues Found

**Critical (Alignment):**
| # | Issue | Type | Why Critical |
|---|-------|------|--------------|
| A-C1 | [Issue] | Roadmap/Arch/Consistency | [Reason] |

**Major (Alignment):**
| # | Issue | Type | Recommendation |
|---|-------|------|----------------|
| A-M1 | [Issue] | [Type] | [Fix] |

### Part 5: Verdict

**Alignment Status:** [Fully Aligned / Minor Deviations / Major Deviations]
**Recommendation:** [Approve / Request changes]
**Blocking issues:** [Count]

---

## Critical Instructions

1. Deviations from approved documents are issues, not style choices.
2. Check EVERY roadmap requirement against the spec.
3. Verify architecture constraints are explicitly followed.
4. Flag unauthorized scope additions or removals.
5. Be objective — flag deviations without judgment; let CA justify.

---

## Output File

Save as: `[Phase]_Spec_Review_Claude.md`
```

#### Adversarial Reviewer Prompt (Codex)

```markdown
## Your Role

You are the **Adversarial Reviewer**. Look for ways the spec could fail, edge cases, and hidden assumptions.

**Your mindset:** Skeptical. Assume there are bugs — find them.

---

## Focus Areas

1. **Assumptions** — What assumptions might be wrong?
2. **Edge Cases** — What inputs cause problems?
3. **Failure Modes** — What happens when things go wrong?
4. **Security** — What can be exploited?
5. **Blind Spots** — What is the architect not seeing?

---

## Input Documents

**Under Review:**
- [Spec v1.0 path]

---

## Review Instructions

### Part 1: Assumption Attack

| Assumption in Spec | Why It Might Be Wrong | Impact If Wrong |
|--------------------|----------------------|-----------------|
| [Assumption 1] | [Weakness] | [Impact] |
| [Assumption 2] | [Weakness] | [Impact] |

### Part 2: Edge Case Analysis

| Scenario | Handled? | If Not, What Happens? |
|----------|----------|----------------------|
| [Edge case 1] | ✅/❌ | [Consequence] |
| [Edge case 2] | ✅/❌ | [Consequence] |

### Part 3: Failure Mode Analysis

| Failure | How It Happens | Detection | Recovery |
|---------|----------------|-----------|----------|
| [Failure 1] | [Cause] | [How noticed] | [How to fix] |
| [Failure 2] | [Cause] | [How noticed] | [How to fix] |

### Part 4: Security Review

| Attack Vector | Applicable? | Mitigation in Spec? |
|---------------|-------------|---------------------|
| [Vector 1] | Yes/No | ✅/❌ |

### Part 5: Blind Spot Identification

**What's the architect not seeing?**
[Identify areas where the spec is overconfident or suspiciously simple]

**What seems too easy?**
[Flag areas that might be harder than the spec suggests]

### Part 6: Issues Found

**Critical (Adversarial):**
| # | Issue | Attack Vector | Impact |
|---|-------|---------------|--------|
| X-C1 | [Issue] | [How found] | [Impact] |

**High (Adversarial):**
| # | Issue | Attack Vector | Impact |
|---|-------|---------------|--------|
| X-H1 | [Issue] | [How found] | [Impact] |

### Part 7: Verdict

**Spec Robustness:** [Robust / Adequate / Fragile]
**Recommendation:** [Approve / Request changes]
**Top 3 Concerns:**
1. [Concern]
2. [Concern]
3. [Concern]

---

## Critical Instructions

1. Your job is to break things (on paper). Find the failures.
2. Don't just list edge cases — assess impact.
3. Challenge "obvious" parts — that's where bugs hide.
4. Ask uncomfortable questions.
5. If something feels wrong but you can't articulate why, flag it.

---

## Output File

Save as: `[Phase]_Spec_Review_Codex.md`
```

### 7.4 B.2: Consolidation

**Purpose:** Synthesize all reviews into a decision-ready document.

```markdown
## Your Role

You are the **Review Consolidator**. Synthesize all three spec reviews into a decision-ready document for the Chief Architect.

---

## Input Documents

- `[Phase]_Spec_Review_Gemini.md` (Peer)
- `[Phase]_Spec_Review_Claude.md` (Alignment)
- `[Phase]_Spec_Review_Codex.md` (Adversarial)

---

## Consolidation Template

### 1. Verdict Summary

| Reviewer | Role | Verdict | Blocking Issues |
|----------|------|---------|-----------------|
| Gemini | Peer | [Verdict] | [Count] |
| Claude | Alignment | [Verdict] | [Count] |
| Codex | Adversarial | [Verdict] | [Count] |

**Consensus:** [X]/3 Approve
**Overall:** [Ready for implementation / Needs revision]

### 2. Blocking Issues

| # | Issue | Flagged By | Category | Impact |
|---|-------|------------|----------|--------|
| B1 | [Issue] | [Reviewer(s)] | [Category] | [Impact] |

### 3. Required Actions for Chief Architect

| Priority | Action | Addresses Issue |
|----------|--------|-----------------|
| 1 | [Specific action] | B1 |
| 2 | [Specific action] | B2 |

### 4. Open Question Recommendations

| Question | Gemini | Claude | Codex | Consolidation Recommendation |
|----------|--------|--------|-------|------------------------------|
| Q1 | [Rec] | [Rec] | [Rec] | [Final recommendation] |

### 5. Agreement Analysis

**Strong Agreement (All 3):**
| Topic | Consensus |
|-------|-----------|
| [Topic] | [Agreement] |

**Disagreement:**
| Topic | Views | Recommendation |
|-------|-------|----------------|
| [Topic] | Gemini: X, Claude: Y, Codex: Z | [Recommendation] |

### 6. Decision Summary

**Spec Status:** [Ready for CA Response / Major Revision Needed]

**Chief Architect must:**
1. [Action]
2. [Action]
3. Update spec to v1.1

---

## Output File

Save as: `[Phase]_Spec_Review_Consolidation.md`
```

### 7.5 B.3: Chief Architect Response

**Purpose:** Respond to findings, make decisions, update spec.

```markdown
## Your Role

You are the **Chief Architect**. Respond to the consolidated review findings, make decisions, and update the spec.

---

## Input Documents

- `[Phase]_Spec_Review_Consolidation.md`
- Original reviews (for reference)
- `[Phase]-Implementation-Spec.md` (v1.0)

---

## Response Template

### 1. Blocking Issues Response

For each blocking issue:

#### B1: [Issue Title]

**Issue:** [Description]
**Flagged by:** [Reviewer(s)]

**Response:** [Accept / Modify / Reject]

**Action:** [What will be done]

**Spec Change:** Section [X] — [Change description]

**Reasoning:** [Why this response]

### 2. Open Questions Decisions

#### Q1: [Question]

**Recommendations:**
- Gemini: [Recommendation]
- Claude: [Recommendation]
- Codex: [Recommendation]
- Consolidation: [Recommendation]

**Decision:** [Your decision]

**Reasoning:** [Why]

**Spec Change:** Section [X] — [Change]

### 3. Changelog for v1.1

| Section | Change | Reason |
|---------|--------|--------|
| [Section] | [Change] | [From issue B1 / Q1 / etc.] |

### 4. Verification Review Decision

**Required?** [Yes / No]

**If Yes, Scope:** [What needs re-review]

**If No, Reasoning:** [Why changes are minor enough]

---

## Outputs

1. This response document: `[Phase]_Chief_Architect_Response.md`
2. Updated spec: `[Phase]-Implementation-Spec.md` (v1.1)
```

### 7.6 B.4: Verification Review (If Needed)

**When to use:** Only if CA made significant changes (rejected recommendations, changed scope substantially).

**Format:** Brief (~1 page per reviewer). Focus on:
- Are CA's changes adequate?
- Any new issues introduced?
- Ready for implementation?

---

## 8. Phase C: Implementation

### 8.1 Purpose

Chief Architect writes implementation prompt, Developer (Antigravity) implements.

### 8.2 Structure

```
C.1: Chief Architect writes implementation prompt
            │
            ▼
C.2: Antigravity implements
            │
            ▼
C.3: PR created
```

### 8.3 C.1: Chief Architect Writes Implementation Prompt

**Key insight:** Chief Architect has codebase context that Antigravity doesn't. CA must translate the spec into step-by-step instructions.

```markdown
## Your Role

You are the **Chief Architect**. Write a detailed implementation prompt for Antigravity (Developer) based on the approved spec v1.1.

**Your job:** Translate the spec into step-by-step instructions that Antigravity can follow without ambiguity.

---

## Why You Write This

You have context that Antigravity doesn't:
- You can see the actual codebase structure
- You know where files are and how they're organized
- You can verify current state before writing instructions
- You can identify exact line numbers and code patterns

Antigravity will execute your instructions literally. Vague instructions lead to wrong implementations.

---

## Pre-Writing Checklist

Before writing the implementation prompt, verify current state:

```bash
# 1. Verify you're on main and up to date
git checkout main
git pull origin main

# 2. Verify tests pass
pytest tests/ -v

# 3. Explore relevant files
# [Check each file mentioned in spec]

# 4. Note current line numbers, function signatures, etc.
```

---

## Implementation Prompt Structure

Your output should include:

### Overview
- Phase context
- What's being built
- Risk level

### Pre-Implementation Checklist
- Branch creation
- State verification
- Required reading

### Architecture Constraints
- Layer constraints (what can import what)
- Patterns to follow
- Verification commands

### Task Sequence

For EACH task:

```markdown
### Task X.Y: [Title]

**Issue:** [From spec]
**Priority:** [Must/Should/Could]

**Current Code:**
\`\`\`python
# [file:lines]
[Exact current code from codebase]
\`\`\`

**Fixed/New Code:**
\`\`\`python
# [file:lines]
[Exact new code]
\`\`\`

**Files:**
- `[file]` — [CREATE/MODIFY/DELETE]

**Step-by-Step:**
1. Open `[file]`
2. Find `[function/class]` (line [X])
3. [Specific change]
4. Save

**Test:**
\`\`\`bash
pytest tests/[test].py -v
\`\`\`

**Verification:**
\`\`\`bash
[Manual verification command]
\`\`\`

**Commit:**
\`\`\`
[type]([scope]): [description]
\`\`\`
```

### Regression Testing Protocol
- Commands to run after each change
- What to do if tests fail

### Final Verification
- Full test suite
- Manual smoke tests
- Architecture constraint verification

### PR Preparation
- Branch name
- PR title
- PR description template
- Expected commit history

---

## Quality Checklist

Before giving to Antigravity:

- [ ] Current state documented (you verified the codebase)
- [ ] Every task has exact code (not "modify the function")
- [ ] File paths are real (you checked)
- [ ] Line numbers are current (you looked)
- [ ] Tests are specified for each change
- [ ] Commits are atomic
- [ ] No ambiguity

---

## Output File

Save as: `[Phase]_Implementation_Prompt_Antigravity.md`
```

### 8.4 C.2: Antigravity Implements

**Input:** Implementation prompt from CA

**Process:**
1. Read prompt completely
2. Complete pre-implementation checklist
3. Execute tasks in order
4. Run tests after each task
5. Commit at checkpoints
6. Run final verification
7. Create PR

### 8.5 C.3: Create PR

**PR Elements:**
- Clear title
- Description matching template
- All tests passing
- Ready for Phase D

---

## 9. Phase D: Code Review

### 9.1 Purpose

Validate the implementation before merge.

### 9.2 Structure

```
D.1: Chief Architect PR Review
            │
            ▼
D.2: Review Board (Parallel)
     ├── Peer Reviewer (Gemini)
     ├── Alignment Reviewer (Claude)
     └── Adversarial Reviewer (Codex)
            │
            ▼
D.3: Consolidation
            │
            ▼
D.4: Antigravity Fixes (if needed)
            │
            ▼
D.5: Codex Verification
            │
            ▼
D.6: Chief Architect Final Approval
            │
            ▼
        MERGE
```

### 9.3 D.1: Chief Architect PR Review

**First-pass review before Review Board.**

```markdown
## Your Role

You are the **Chief Architect**. Perform first-pass PR review.

**Focus:**
1. Code matches spec v1.1
2. Tests pass
3. No scope creep
4. Clean commits

---

## Quick Checks

\`\`\`bash
# Fetch PR
git fetch origin pull/[N]/head:pr-[N]
git checkout pr-[N]

# Tests pass?
pytest tests/ -v

# Commands work?
[smoke test commands]
\`\`\`

---

## Review Template

### Summary

| Check | Status |
|-------|--------|
| Tests pass | ✅/❌ |
| Matches spec | ✅/❌ |
| No scope creep | ✅/❌ |
| Clean commits | ✅/❌ |

**Verdict:** [Ready for Review Board / Needs fixes first]

### Spec Compliance

| Spec Section | Implemented? | Correctly? |
|--------------|--------------|------------|
| [Section] | ✅/❌ | ✅/❌ |

### Issues Found

| # | Issue | Severity |
|---|-------|----------|
| CA-1 | [Issue or "None"] | [Severity] |

---

## Output

Post as PR comment + save to `[Phase]_PR_Review_Chief_Architect.md`
```

### 9.4 D.2: Review Board

**Three parallel code reviews.** Similar to spec review but focused on code.

#### Peer Reviewer (Gemini) — Code Quality

```markdown
## Your Role

You are the **Peer Reviewer**. Review for code quality, test coverage, and documentation.

---

## Focus Areas

1. Code quality — Clean, readable, maintainable
2. Test coverage — Adequate tests for each change
3. Documentation — Matches code
4. Commit quality — Atomic, well-messaged

---

## Review Template

### Fix-by-Fix Review

For each fix/feature:

| Aspect | Rating |
|--------|--------|
| Code clarity | Good/Adequate/Poor |
| Test coverage | Good/Adequate/Poor |
| Error handling | Good/Adequate/Poor |

### Issues Found

| # | Issue | File | Line(s) | Severity |
|---|-------|------|---------|----------|
| P-1 | [Issue] | [file] | [lines] | Must Fix/Should Fix/Minor |

### Verdict

**Recommendation:** [Approve / Request Changes]

---

## Output

Post as PR comment + save to `[Phase]_Code_Review_Gemini.md`
```

#### Alignment Reviewer (Claude) — Compliance

```markdown
## Your Role

You are the **Alignment Reviewer**. Verify code matches spec and maintains constraints.

---

## Focus Areas

1. Spec compliance — Does code match spec v1.1?
2. Architecture — Are layer constraints respected?
3. Backward compatibility — Any breaking changes?
4. Consistency — Matches existing patterns?

---

## Review Template

### Spec Compliance

| Spec Requirement | Implemented? | Correctly? |
|------------------|--------------|------------|
| [Requirement] | ✅/❌ | ✅/❌ |

### Architecture Check

\`\`\`bash
# Verify constraints
grep -rn "from [package].io" [package]/core/
# Should be empty
\`\`\`

| Constraint | Status |
|------------|--------|
| [Constraint] | ✅/❌ |

### Backward Compatibility

| Interface | Changed? | Breaking? |
|-----------|----------|-----------|
| CLI | No/Yes | No/Yes |
| Exit codes | No/Yes | No/Yes |
| Output format | No/Yes | No/Yes |

### Issues Found

| # | Issue | Type | Severity |
|---|-------|------|----------|
| A-1 | [Issue] | Spec/Arch/Compat | [Severity] |

### Verdict

**Recommendation:** [Approve / Request Changes]

---

## Output

Post as PR comment + save to `[Phase]_Code_Review_Claude.md`
```

#### Adversarial Reviewer (Codex) — Breaking

```markdown
## Your Role

You are the **Adversarial Reviewer**. Hunt for bugs, edge cases, and regressions.

---

## Focus Areas

1. Regressions — Did changes break existing functionality?
2. Edge cases — Are unusual inputs handled?
3. Error handling — What happens when things fail?
4. Test adequacy — Do tests actually catch issues?

---

## Review Template

### Regression Hunt

\`\`\`bash
# Run full test suite
pytest tests/ -v

# Try edge cases
[edge case commands]
\`\`\`

| Area | Regression? | Evidence |
|------|-------------|----------|
| [Area] | None/Found | [Evidence] |

### Edge Case Analysis

| Edge Case | Handled? | Test Exists? |
|-----------|----------|--------------|
| [Case] | ✅/❌ | ✅/❌ |

### Test Adequacy

| Fix | Test Exists? | Actually Tests the Fix? |
|-----|--------------|-------------------------|
| [Fix] | ✅/❌ | ✅/❌ |

### Issues Found

| # | Issue | Attack Vector | Impact |
|---|-------|---------------|--------|
| X-1 | [Issue] | [How found] | [Impact] |

### Verdict

**Recommendation:** [Approve / Request Changes]
**Top Concerns:**
1. [Concern]
2. [Concern]

---

## Output

Post as PR comment + save to `[Phase]_Code_Review_Codex.md`
```

### 9.5 D.3: Consolidation

Same structure as spec review consolidation, but for code review findings.

**Key sections:**
- Verdict summary (4 reviewers now — CA + 3 board)
- Blocking issues
- Required actions for Antigravity
- Agreement analysis

### 9.6 D.4: Antigravity Fixes

**If blocking issues exist:**

```markdown
## Your Role

You are **Antigravity**. Fix the blocking issues identified in consolidation.

---

## Input

Consolidation Section 3: "Required Actions for Antigravity"

---

## Process

1. Address issues in priority order
2. One commit per logical fix
3. Run tests after each fix
4. Post fix summary to PR

---

## Fix Summary Template

### Summary

| Issue | Status |
|-------|--------|
| B1 | ✅ Fixed |
| B2 | ✅ Fixed |

### Fixes Applied

#### B1: [Issue]

**Issue:** [Description]
**Fix:** [What was done]
**Files:** [Changed files]
**Commit:** `fix([scope]): [description]`

### Verification

\`\`\`bash
pytest tests/ -v
# All pass
\`\`\`

**Ready for D.5: Codex Verification**
```

### 9.7 D.5: Codex Verification

**Codex verifies their flagged issues are actually fixed.**

```markdown
## Your Role

You are **Codex (Adversarial)**. Verify that the issues you flagged have been fixed.

---

## Input

- Your original review
- Antigravity's fix summary
- Updated code

---

## Verification Template

### Issue Verification

#### X-1: [Issue I Flagged]

**Original Issue:** [What I found]
**Antigravity's Fix:** [What they did]

**Verification:**
- [ ] Code change is correct
- [ ] Issue is actually fixed
- [ ] No new issues introduced

**Verdict:** ✅ Fixed / ❌ Not Fixed

### Regression Check

\`\`\`bash
pytest tests/ -v
\`\`\`

| Check | Status |
|-------|--------|
| All tests pass | ✅/❌ |
| No new regressions | ✅/❌ |

### Verdict

**Recommendation:** [Approve / Request further fixes]

---

## Output

Post as PR comment + save to `[Phase]_Code_Verification_Codex.md`
```

### 9.8 D.6: Chief Architect Final Approval

```markdown
## Your Role

You are the **Chief Architect**. Make final merge decision.

---

## Prerequisites

- [ ] Codex verification passed
- [ ] All blocking issues resolved
- [ ] All tests pass

---

## Approval Template

# ✅ APPROVED FOR MERGE

**All criteria met. PR #[N] is authorized for merge.**

### Summary

| Criterion | Status |
|-----------|--------|
| Codex verification | ✅ |
| All issues resolved | ✅ |
| Tests pass | ✅ |

### Merge Instructions

**Method:** Squash and merge

**Commit message:**
\`\`\`
[type]: [Phase] — [Theme] (#[PR])

[Summary of changes]

Reviewed-by: Gemini (Peer), Claude (Alignment), Codex (Adversarial)
Verified-by: Codex
Approved-by: Chief Architect
\`\`\`

### Post-Merge

- [ ] Tag release (if applicable)
- [ ] Update changelog
- [ ] Close issues

---

## Output

Post as PR comment + save to `[Phase]_Final_Approval_Chief_Architect.md`
```

### 9.9 PR Comment Order

| Order | Comment | From |
|-------|---------|------|
| 1 | Chief Architect PR Review | D.1 |
| 2 | Peer Code Review | D.2 (Gemini) |
| 3 | Alignment Code Review | D.2 (Claude) |
| 4 | Adversarial Code Review | D.2 (Codex) |
| 5 | Consolidation | D.3 |
| 6 | Fix Summary | D.4 (Antigravity) — if needed |
| 7 | Adversarial Verification | D.5 (Codex) — if D.4 happened |
| 8 | Final Approval | D.6 (Chief Architect) |

---

## 10. Prompt Engineering

### 10.1 The Core Problem

Most review prompts fail by being too gentle. Reviewers mirror prompt energy.

| Prompt Energy | Result |
|---------------|--------|
| "Please review" | Polite approval |
| "Any feedback?" | Minimal feedback |
| "Find the problems" | Actual problems found |

### 10.2 Anti-Patterns to Avoid

#### Leading Questions

| ❌ Leading | ✅ Open |
|-----------|---------|
| "Does this work?" | "How does this break?" |
| "Is this complete?" | "What's missing?" |
| "Any issues?" | "What are the top 3 issues?" |

#### Author-Selected Focus

| ❌ Author-Selected | ✅ Document-Driven |
|-------------------|-------------------|
| "Review these 7 areas" | "Verify against Architecture v1.4" |
| "Focus on technical correctness" | "Check all roadmap requirements" |

#### Implicit Approval Expectation

| ❌ Approval-Seeking | ✅ Critique-Inviting |
|--------------------|---------------------|
| "Please review and let me know" | "Your job is to find problems" |
| "Confirm the approach" | "Challenge the approach" |

#### Comfort-Seeking Language

| ❌ Soft | ✅ Direct |
|--------|----------|
| "Feel free to raise concerns" | "What concerns you most?" |
| "If you notice anything" | "What did you notice that's off?" |
| "Any suggestions" | "What would you change?" |

### 10.3 Patterns That Work

#### Explicit Permission to Disagree

```markdown
**Your job is not to approve.** You are here to find problems.

**The architect has blind spots.** They can't see their own assumptions. You can.

**Be constructive, but don't be gentle.** Catch problems before implementation.
```

#### Structured Critique Frameworks

```markdown
### Assumption Attack

| Assumption | Why It Might Be Wrong | Impact If Wrong |
|------------|----------------------|-----------------|

### Failure Mode Analysis

| Failure | How It Happens | Would We Notice? |
|---------|----------------|------------------|
```

#### Uncomfortable Questions

```markdown
### Uncomfortable Questions

Questions that challenge the architect's decisions:

1. [Question challenging core assumption]
2. [Question suggesting alternative]
3. [Question exposing reasoning gap]

**If something seems odd but you can't articulate why, flag it anyway.**
```

#### Blind Spot Identification

```markdown
### What's the Architect Not Seeing?

They probably got the hard parts right (they thought about those).
They probably got the "obvious" parts wrong (they didn't think about those).

**Where is the spec overconfident?**
**What seems too simple?**
**What's conspicuously absent?**
```

### 10.4 Language Substitution Guide

| ❌ Weak | ✅ Strong |
|--------|----------|
| "Please review" | "Find the problems in" |
| "Any feedback" | "What are the top 3 issues" |
| "Feel free to" | "You must" |
| "It would be helpful" | "I need you to" |
| "Does this work" | "How does this break" |
| "Is this complete" | "What's missing" |
| "Looks good?" | "What's the weakest part" |

### 10.5 Pre-Send Checklist

Before sending a review prompt:

- [ ] No leading questions
- [ ] No author-selected focus
- [ ] Explicit critique invitation
- [ ] Permission to disagree stated
- [ ] "What am I not seeing?" included
- [ ] "What would you do differently?" included
- [ ] Reference documents cited
- [ ] Structured frameworks provided
- [ ] Severity stratification required

---

## 11. Output Standards

### 11.1 PR Comments

**Use `<details>` tags for long sections:**

```markdown
## Summary

[Always visible — key findings]

---

<details>
<summary><strong>Detailed Analysis (click to expand)</strong></summary>

[Detailed content]

</details>
```

### 11.2 Review Structure

Every review should have:

1. **Summary** — Always visible, key verdict
2. **Issues Found** — By severity
3. **Detailed Analysis** — In collapsible sections
4. **Verdict** — Clear recommendation
5. **Signature** — Role, model, date

### 11.3 Issue Severity Levels

| Level | Meaning | Action |
|-------|---------|--------|
| **Critical/Blocking** | Cannot proceed | Must fix before continue |
| **Major** | Should fix | Fix before merge |
| **Minor** | Consider | Fix or defer |

### 11.4 Signature Block

```markdown
---

**Review signed by:**
- **Role:** [Role]
- **Model:** [Model]
- **Date:** [Date]
- **Review Type:** [Spec Review / Code Review / etc.]
```

---

## 12. File Naming Conventions

### 12.1 Directory Structure

```
.ontos-internal/
└── strategy/
    └── v[VERSION]/
        └── Phase[N]/
            ├── Phase[N]-Implementation-Spec.md (v1.0, v1.1)
            ├── Phase[N]_Spec_Review_Gemini.md
            ├── Phase[N]_Spec_Review_Claude.md
            ├── Phase[N]_Spec_Review_Codex.md
            ├── Phase[N]_Spec_Review_Consolidation.md
            ├── Phase[N]_Chief_Architect_Response.md
            ├── Phase[N]_Implementation_Prompt_Antigravity.md
            ├── Phase[N]_PR_Review_Chief_Architect.md
            ├── Phase[N]_Code_Review_Gemini.md
            ├── Phase[N]_Code_Review_Claude.md
            ├── Phase[N]_Code_Review_Codex.md
            ├── Phase[N]_Code_Review_Consolidation.md
            ├── Phase[N]_Fix_Summary_Antigravity.md
            ├── Phase[N]_Code_Verification_Codex.md
            └── Phase[N]_Final_Approval_Chief_Architect.md
```

### 12.2 File Naming Pattern

```
Phase[N]_[Step]_[Role/Model].md
```

**Examples:**
- `Phase3_Spec_Review_Gemini.md`
- `Phase4_Code_Review_Codex.md`
- `Phase5_Chief_Architect_Response.md`

---

## 13. Phase Variants

### 13.1 Feature Phase (Default)

Standard workflow for new features.

**Risk:** Medium-High
**Focus:** New functionality, architecture compliance
**Breaking changes:** May be acceptable

### 13.2 Patch/Polish Phase

For bug fixes and polish after a release.

**Risk:** Low-Medium
**Focus:** No regressions, backward compatibility
**Breaking changes:** NOT acceptable

**Key differences:**

| Aspect | Feature Phase | Patch Phase |
|--------|---------------|-------------|
| Breaking changes | Allowed | NOT allowed |
| Scope | New features | Bug fixes only |
| Alignment focus | Architecture | Backward compatibility |
| Adversarial focus | Edge cases | Regressions |
| Test focus | New functionality | No regressions |

**Alignment Reviewer additions for patch phases:**
- Check CLI compatibility
- Check output format stability
- Check exit code consistency
- Verify no API changes

**Adversarial Reviewer additions for patch phases:**
- Check each fix fully addresses issue
- Check each fix doesn't break something else
- Verify test catches the regression

### 13.3 Hotfix Phase

Emergency fixes for production issues.

**Risk:** High (speed vs. thoroughness)
**Focus:** Fix the issue, minimize blast radius
**Process:** Abbreviated — CA + Adversarial only

---

## 14. Pre-Phase Documentation

### 14.1 Strategic Analysis

Before starting development, generate a **Strategic Analysis** document covering:

- Problem statement
- Market fit
- Value propositions
- Strategy & roadmap
- Conceptual design (space-time ontology, etc.)
- Mechanism ontology

**Purpose:** Aligns all contributors on the "why" before the "what."

**See:** `ontos-analysis-generation-instructions.md` for full template.

### 14.2 Technical Architecture Map

Generate a **Technical Architecture Map** covering:

- Directory structure
- Code metrics
- Package architecture
- CLI reference
- Core modules
- Configuration
- Dependencies

**Purpose:** LLM-readable reference for code analysis. Share with reviewers for context.

**See:** `ontos-analysis-generation-instructions.md` for full template.

---

## 15. Automation Architecture

### 15.1 Current State: Manual Orchestration

The human acts as "message bus" between LLM instances:
1. Copy prompt to Model A
2. Copy output to file
3. Copy prompt to Model B
4. Copy output to file
5. Repeat

**Overhead:** ~50% of effort is orchestration, not content.

### 15.2 Future: Two-Layer Automation

```
┌─────────────────────────────────────────┐
│         STRATEGY LAYER                  │
│         (Web Claude)                    │
│                                         │
│  - Generate prompts from templates      │
│  - Make decisions at breakpoints        │
│  - Adjust based on context              │
└──────────────────┬──────────────────────┘
                   │
                   │ Prompts + Config
                   ▼
┌─────────────────────────────────────────┐
│         EXECUTION LAYER                 │
│         (Local Conductor Script)        │
│                                         │
│  - Read workflow config                 │
│  - Invoke CLIs (Claude Code, Gemini,    │
│    Codex) in headless mode              │
│  - Collect outputs                      │
│  - Pause at breakpoints for human       │
└─────────────────────────────────────────┘
```

### 15.3 Conductor Responsibilities

| Responsibility | Implementation |
|----------------|----------------|
| Workflow sequencing | Read YAML/JSON config |
| CLI invocation | Subprocess calls to claude, gemini, codex |
| Output collection | Capture stdout, save to files |
| Breakpoints | Pause, notify human, wait for signal |
| Error handling | Retry, escalate, log |

### 15.4 Human Touchpoints

Even with automation, humans decide at:

1. **After consolidation** — Accept recommendations or override?
2. **After implementation** — PR looks good?
3. **After code review consolidation** — Merge or more fixes?
4. **Final approval** — Actually merge?

---

## 16. Troubleshooting

### 16.1 Reviews Are Too Gentle

**Symptom:** Reviews approve everything, find no issues.

**Causes:**
- Prompt is approval-seeking
- Same model reviewing own work
- No explicit critique invitation

**Fixes:**
- Use critique-inviting language
- Ensure model diversity
- Add "Your job is to find problems"
- Add "What's the architect not seeing?"

### 16.2 Reviews Are Unconstructive

**Symptom:** Reviews list many issues but no solutions.

**Causes:**
- Prompt focuses only on finding problems
- No solution requirement

**Fixes:**
- Add "For each issue, suggest a fix"
- Require severity rating
- Add structured tables with "Recommendation" column

### 16.3 Too Many Review Rounds

**Symptom:** 3+ rounds of review without convergence.

**Causes:**
- Issues aren't being fully addressed
- Scope is creeping
- Reviewers keep finding new things

**Fixes:**
- One revision cap — escalate after round 2
- Fix scope at spec approval
- Distinguish blocking vs. non-blocking issues

### 16.4 Implementation Doesn't Match Spec

**Symptom:** Code diverges from approved spec.

**Causes:**
- Implementation prompt too vague
- Developer made independent decisions
- Spec was ambiguous

**Fixes:**
- CA writes implementation prompt with exact code
- Include line numbers, file paths
- Add "Don't deviate from spec" instruction
- Add "Flag unclear instructions" instruction

### 16.5 Model Gives Inconsistent Output

**Symptom:** Same prompt gives different quality results.

**Causes:**
- Prompt is ambiguous
- Missing context
- Temperature too high

**Fixes:**
- Make prompt more specific
- Include all necessary context
- Use structured templates
- Reduce temperature (if configurable)

### 16.6 Consolidation Misses Issues

**Symptom:** Issues from individual reviews not in consolidation.

**Causes:**
- Consolidator rushed
- No checklist

**Fixes:**
- Add checklist: "Verify all Critical/Major issues from each review appear"
- Use structured consolidation template
- Cross-reference issue IDs

---

## 17. Quick Reference

### 17.1 Phase Checklist

#### Phase A: Spec Development
- [ ] Review roadmap requirements
- [ ] Check architecture constraints
- [ ] Write spec v1.0
- [ ] Self-review against template
- [ ] Submit for Phase B

#### Phase B: Spec Review
- [ ] B.1: Run 3 parallel reviews (Gemini, Claude, Codex)
- [ ] B.2: Consolidate findings
- [ ] B.3: CA responds, updates spec to v1.1
- [ ] B.4: Verification review (if needed)

#### Phase C: Implementation
- [ ] C.1: CA writes implementation prompt
- [ ] C.2: Antigravity implements
- [ ] C.3: PR created

#### Phase D: Code Review
- [ ] D.1: CA first-pass review
- [ ] D.2: Run 3 parallel reviews
- [ ] D.3: Consolidate findings
- [ ] D.4: Antigravity fixes (if needed)
- [ ] D.5: Codex verification
- [ ] D.6: CA final approval
- [ ] Merge

### 17.2 Model Quick Reference

| Model | Use For |
|-------|---------|
| Claude | Chief Architect, Alignment Reviewer |
| Gemini | Peer Reviewer, Research |
| Codex | Adversarial Reviewer, Developer |

### 17.3 File Quick Reference

| Document | Naming |
|----------|--------|
| Spec | `Phase[N]-Implementation-Spec.md` |
| Spec Review | `Phase[N]_Spec_Review_[Model].md` |
| Consolidation | `Phase[N]_[Spec/Code]_Review_Consolidation.md` |
| CA Response | `Phase[N]_Chief_Architect_Response.md` |
| Implementation Prompt | `Phase[N]_Implementation_Prompt_Antigravity.md` |
| Code Review | `Phase[N]_Code_Review_[Model].md` |
| Verification | `Phase[N]_Code_Verification_Codex.md` |
| Approval | `Phase[N]_Final_Approval_Chief_Architect.md` |

### 17.4 Prompt Energy Quick Reference

| Weak → | Strong |
|--------|--------|
| "Please review" | "Find the problems" |
| "Any issues?" | "What are the top 3 issues?" |
| "Does this work?" | "How does this break?" |
| "Is this complete?" | "What's missing?" |
| "Looks good?" | "What's the weakest part?" |

---

## Appendix A: Related Documents

| Document | Purpose |
|----------|---------|
| `prompt-engineering-guide-rigorous-critique.md` | Deep dive on critique prompts |
| `ontos-analysis-generation-instructions.md` | Strategic + Technical analysis templates |
| `phase[N]-complete-prompt-suite.md` | Phase-specific complete prompts |

---

## Appendix B: Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-01-13 | Initial version |
| 2.0 | 2026-01-15 | Added: Model diversity principle, C.1 CA writes implementation prompt, Phase variants, Pre-phase documentation, Automation architecture, Expanded prompt engineering, Battle-tested refinements |

---

*Playbook Version: 2.0*
*Battle-tested through Ontos v3.0 Phases 0-5*
