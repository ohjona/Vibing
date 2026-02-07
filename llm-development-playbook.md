# LLM Development Playbook: Multi-Agent Review Workflow

**Version:** 3.2
**Updated:** 2026-02-06
**Based on:** Battle-tested patterns across 8+ release cycles

---

## Table of Contents

**Tier 1: Principles & Workflow**

1. [Overview](#1-overview)
2. [Core Principles](#2-core-principles)
3. [Model Assignments](#3-model-assignments)
4. [Role Definitions](#4-role-definitions)
5. [Workflow Phases](#5-workflow-phases)
6. [Project Setup](#6-project-setup)
7. [Prompt Engineering](#7-prompt-engineering)

**Tier 2: Phase Guides**

8. [Phase A: Spec Development](#8-phase-a-spec-development)
9. [Phase B: Spec Review](#9-phase-b-spec-review)
10. [Phase C: Implementation](#10-phase-c-implementation)
11. [Phase D: Code Review](#11-phase-d-code-review)

**Tier 3: Operational Protocols**

12. [Pre-Phase A: Triage & Validation](#12-pre-phase-a-triage--validation)
13. [Pre-Phase A: Proposal Review](#13-pre-phase-a-proposal-review)
14. [Spec Deviation Protocol](#14-spec-deviation-protocol)
15. [Phase Variants](#15-phase-variants)
16. [Output Standards](#16-output-standards)
17. [Process Evaluation](#17-process-evaluation)
18. [Troubleshooting](#18-troubleshooting)
19. [Quick Reference](#19-quick-reference)

[Appendix A: Related Documents](#appendix-a-related-documents)
[Appendix B: Version History](#appendix-b-version-history)

---

# Tier 1: Principles & Workflow

---

## 1. Overview

### 1.1 What This Is

An adaptive multi-agent workflow for LLM-augmented software development. It uses intentional model diversity and role differentiation to catch issues that single-model workflows miss.

This playbook is project-agnostic. It defines the workflow, roles, and principles. You supply the project-specific variables (Section 6) and reference documents.

### 1.2 The Problem and Solution

| Problem | Symptom | This Playbook's Answer |
|---------|---------|------------------------|
| **Blind spots** | Can't see own assumptions | Different models review each other's work |
| **Rubber-stamping** | Approves own work | No model reviews its own output |
| **Role fatigue** | Perspective narrows over time | Distinct reviewer roles with different mandates |
| **Shared biases** | Same model, same biases | Minimum 2 model families per review board |

**Key insight:** Role diversity alone isn't enough. The same model playing different roles still shares underlying biases. Model diversity ensures genuinely independent perspectives.

### 1.3 The Workflow

```
PHASE A: Spec Development          PHASE B: Spec Review
  CA writes spec v1.0                B.1: Review Board (3 models, parallel)
         │                           B.2: Consolidation
         ▼                           B.3: CA Response → Spec v1.1
  Submit for review ──────────▶      B.4: Verification (if needed)
                                            │
                    ┌───────────────────────┘
                    ▼
PHASE C: Implementation             PHASE D: Code Review
  C.1: CA writes impl prompt         D.1: CA PR Review
  C.2: Developer implements          D.2: Review Board (3 models, parallel)
  C.3: PR created ──────────────▶    D.3: Consolidation
                                     D.4: Developer fixes (if needed)
                                     D.5: Adversarial verification
                                     D.6: CA approval → MERGE
```

---

## 2. Core Principles

### 2.1 Model Diversity is Non-Negotiable

Don't use the same model for all roles. Even with different prompts, the same model has the same blind spots.

| Homogeneous (avoid) | Heterogeneous (use) |
|----------------------|---------------------|
| Model X reviews Model X's spec | Model Y reviews Model X's spec |
| Model X verifies Model X's code | Different perspective on same code |
| Shared biases pass through | Independent verification |

**Minimum diversity:** At least 2 different model families in every review board.

### 2.2 One-Revision Cap

Prevent infinite convergence loops. Each review cycle allows ONE round of revisions.

| Round | Purpose | If Issues Remain |
|-------|---------|------------------|
| Round 1 | Full review: find all issues | Consolidate, fix |
| Round 2 | Verification: confirm fixes | Escalate or accept |
| Round 3+ | **Avoid** | Re-scope or re-spec |

If two rounds don't resolve issues, the problem is usually scope or requirements, not execution.

#### 2.2.1 Decision Matrix After Round 2

When verification surfaces remaining or new issues:

| Issue Type | Critical | Major | Minor |
|------------|----------|-------|-------|
| **Factual error by reviewer** | Override with evidence | Override with evidence | Override with evidence |
| **Legitimate bug/gap found** | Fix (back to D.4) | Fix (back to D.4) | CA decides |
| **Style/preference disagreement** | Note and proceed | Note and proceed | Note and proceed |
| **Scope expansion request** | Reject | Reject | Reject |
| **Missing tests for existing code** | Fix | CA decides | Defer |
| **Missing tests for new code** | Fix | Fix | CA decides |

**"CA decides" tiebreaker:** Make a judgment call based on complexity. Quick gut check: does this feel like a 10-minute fix or a rabbit hole? If quick, do it. If deferring creates technical debt that'll cost more later, do it. If uncertain, ask the developer for a quick effort estimate — but don't turn the tiebreaker into a multi-step process. The point is to decide and move on.

**Escalation:** If the matrix doesn't resolve it, the problem is scope. Return to Phase A. Do NOT start Round 3.

### 2.3 Role Differentiation

Different roles catch different issues. Don't use the same prompt for all reviewers.

| Role | Focus | Core Question |
|------|-------|---------------|
| **Peer** | Quality, completeness, UX | "Is this good?" |
| **Alignment** | Compliance, constraints | "Does this follow the rules?" |
| **Adversarial** | Breaking, edge cases | "How does this fail?" |

### 2.4 Document-Driven Review

Anchor reviews to approved documents, not author framing.

| Author-Selected Focus (avoid) | Document-Driven (use) |
|-------------------------------|----------------------|
| "Review these 7 areas I identified" | "Verify alignment with Architecture doc, Roadmap section 6" |
| "Focus on technical correctness" | "Review against all approved reference documents" |
| Author steers away from weak areas | Documents define complete scope |

Authors know where they're uncertain (and steer away). They don't know where they're overconfident (where the bugs are).

### 2.5 Explicit Permission to Disagree

Reviewers mirror the prompt's energy.

| Prompt Energy | Reviewer Response |
|---------------|-------------------|
| "Please review and let me know" | Polite approval |
| "Find the problems in this" | Actual critique |
| "What's the architect not seeing?" | Hidden issues surfaced |

Default to critique-inviting language. See Section 7: Prompt Engineering.

### 2.6 Git as Working Record

Markdown files are working artifacts during development. PR comments are the publication venue for review outputs, approvals, and decisions.

| Output | Location | Purpose |
|--------|----------|---------|
| Review prompts | `.md` files delivered to orchestrator | Working artifacts |
| Review outputs | PR comments (primary), `.md` backups | Official record |
| Decisions | PR comments | Audit trail |
| Specs | Project internal directory | Reference, version-controlled |

### 2.7 Fresh Prompt Generation

Every prompt is generated fresh for the current step. Never reference prompts from previous conversation turns or prior sessions.

| Don't | Do |
|-------|-----|
| "Use the Phase B prompt I generated earlier" | Generate a new Phase B prompt with current context |
| "Same as last time but change the PR number" | Full prompt with correct PR number embedded |
| "Refer to the implementation prompt above" | Complete, self-contained prompt |

**Why:** Context windows compact or reset between sessions. PR numbers, branch names, and file states change. Each model instance starts fresh; give it everything it needs.

**Self-containment test:** If you can't copy-paste a prompt into a fresh model session and have it work, the prompt is incomplete.

---

## 3. Model Assignments

### 3.1 Default Assignments

| Role | Default Model | Why |
|------|---------------|-----|
| **Chief Architect** | Claude | Strong reasoning, synthesis, planning |
| **Peer Reviewer** | Gemini | Research, UX perspective, documentation |
| **Alignment Reviewer** | Claude | Constraint checking, consistency |
| **Adversarial Reviewer** | Codex | Edge cases, breaking things, security |
| **Developer** | Codex | Implementation, code generation |

These are defaults. Override based on your project's needs and model availability.

### 3.2 Assignment Flexibility

| Situation | Override |
|-----------|----------|
| Security-critical phase | Codex as Alignment (security focus) |
| UX-heavy phase | Gemini as Peer + Alignment |
| Complex architecture | Claude for all review roles (accept reduced diversity) |
| Research-heavy spec | Gemini as Chief Architect |
| Context-heavy phase (large diff, many reference docs) | Largest-window model for highest-context role (usually Adversarial or Consolidation). See Section 7.8.7. |

### 3.3 Minimum Diversity Requirements

| Review Board | Minimum |
|--------------|---------|
| Spec Review (3 reviewers) | At least 2 different model families |
| Code Review (3 reviewers) | At least 2 different model families |
| Verification | Different model than original reviewer |

---

## 4. Role Definitions

### 4.1 Chief Architect

The decision-maker and synthesizer. Typically a human or human-directed model.

| Responsibility | Actions |
|----------------|---------|
| Write specs | Create implementation specifications |
| Make decisions | Resolve open questions, choose approaches |
| Respond to reviews | Accept, modify, or reject review findings |
| Write implementation prompts | Translate specs into developer instructions |
| Final approval | Authorize merges |

**Key outputs:** Spec v1.0, v1.1, CA Response documents, implementation prompts, final approval.

### 4.2 Peer Reviewer

The quality and completeness checker.

| Focus | Core Questions |
|-------|----------------|
| Completeness | Is anything missing? |
| Quality | Is this well-designed? |
| Clarity | Can this be implemented without questions? |
| UX | Will users understand this? |

**Key outputs:** Spec reviews (quality focus), code reviews (quality focus).

### 4.3 Alignment Reviewer

The compliance and consistency checker.

| Focus | Core Questions |
|-------|----------------|
| Architecture | Does this match the architecture? |
| Roadmap | Does this implement what was planned? |
| Constraints | Are layer constraints respected? |
| Consistency | Is this consistent with prior decisions? |
| Backward compatibility | Does this break existing behavior? |

**Key outputs:** Spec reviews (alignment focus), code reviews (compliance focus), deviation reports.

### 4.4 Adversarial Reviewer

The breaker and edge case finder.

| Focus | Core Questions |
|-------|----------------|
| Failure modes | How does this break? |
| Edge cases | What inputs cause problems? |
| Security | What can be exploited? |
| Assumptions | What assumptions might be wrong? |
| Regressions | Could this break existing functionality? |

**Key outputs:** Spec reviews (attack focus), code reviews (breaking focus), verification reviews.

### 4.5 Developer

The implementer.

| Responsibility | Actions |
|----------------|---------|
| Follow prompts | Execute implementation instructions literally |
| Write code | Implement features as specified |
| Write tests | Add tests for new functionality |
| Fix issues | Address review findings |
| Report blockers | Flag unclear instructions or spec gaps |

**Key outputs:** Working code, tests, fix summaries, PR descriptions.

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
          ▼                   ▼
     If major issues     If major issues
     return to A         return to C
```

### 5.3 Pre-Phase A Decision Tree

Before entering Phase A, determine whether pre-work is needed:

```
                    ┌─────────────────────┐
                    │ What do you have?   │
                    └──────────┬──────────┘
                               │
              ┌────────────────┼────────────────┐
              ▼                ▼                 ▼
      Pile of findings   Approach to      Clear scope
      (audit, bugs,      validate          already defined
       discovery)        (design direction)
              │                │                 │
              ▼                ▼                 ▼
     Pre-Phase A:      Pre-Phase A:        Go directly
     Triage &          Proposal            to Phase A
     Validation        Review
     (Section 12)      (Section 13)
```

---

## 6. Project Setup

### 6.1 Project Variables

Define once per project, update rarely. These values are referenced throughout all prompts.

| Variable | Description | Example |
|----------|-------------|---------|
| `PROJECT_NAME` | Project identifier | — |
| `REPO_URL` | Repository location | `https://github.com/org/repo` |
| `INTERNAL_DOCS_DIR` | Where specs and review outputs are stored | `.project-internal/` |
| `TEST_COMMAND` | How to run the test suite | `pytest tests/ -v` |
| `SMOKE_TEST_COMMANDS` | Quick verification commands | Project-specific CLI checks |
| `ARCHITECTURE_CONSTRAINTS` | Layer rules, import restrictions, etc. | "Core must not import from UI" |
| `REFERENCE_DOCS` | Paths to roadmap, architecture, strategy docs | Project-specific |
| `BRANCH_CONVENTION` | Branch naming pattern | `feature/v{version}-{description}` |
| `MODEL_ASSIGNMENTS` | Which model plays which role | See Section 3.1 |
| `DEVELOPER_TOOL` | CLI tool the developer agent uses | Codex CLI, Claude Code, Cursor, etc. |

### 6.2 Release Decisions

Decide per release cycle before entering the workflow:

| Decision | Options | How to Decide |
|----------|---------|---------------|
| Phase variant | Feature / Patch / Hotfix | Scope and risk level (see Section 15) |
| Pre-Phase A needed? | Triage / Proposal Review / Neither | Decision tree in 5.3 |
| Track splitting? | Monolithic / Track-based | See criteria in Section 15.4 |
| Number of tracks | 2-4 | One per conceptually independent work stream |

### 6.3 Phase Transition Confirmations

**Hard gate.** Confirm these before generating any prompt suite. If any are unknown, ask first.

| Confirmation | Why | Example |
|--------------|-----|---------|
| Branch name (exact) | Propagates across all prompts in suite | `feature/v2.1.0-auth-refactor` |
| PR number (if exists) | Same | `#60` |
| Spec version (current approved) | Reviewers need to know what they're checking against | `v1.1` |
| Base branch | Usually `main` but not always | `main` |
| Files in scope | Focuses reviewers, especially for track-based work | List of changed files |

A single wrong PR number propagates across 8+ prompt files. 30 seconds of confirmation prevents 30 minutes of corrections.

**If references change after prompt generation:**
1. Identify all affected files in the suite
2. Update systematically (search-and-replace)
3. Re-deliver corrected files
4. Confirm corrections with orchestrator

---

## 7. Prompt Engineering

This section defines how to write prompts that elicit thorough, honest feedback from LLM reviewers. These principles apply to every reviewer prompt generated by this workflow, not just the Chief Architect's prompts. Every reviewer role must receive prompts built on these foundations.

### 7.1 The Core Problem

Most review prompts fail in one of two predictable ways:

| Failure Mode | What Happens | Result |
|--------------|--------------|--------|
| **Too Gentle** | Reviewer feels pressure to validate | Superficial approval, missed issues |
| **Too Harsh** | Reviewer becomes adversarial without structure | Unconstructive criticism, noise |

**The sweet spot:** Reviewers feel *expected* to find problems, *safe* to voice concerns, and *guided* toward actionable feedback.

**The fundamental law:** Reviewers mirror the prompt's energy. Ask for approval, get approval. Ask for problems, get problems.

### 7.2 Anti-Patterns

These patterns undermine review quality. Avoid them in every prompt.

**Leading questions** presume an answer and ask for confirmation:

| Leading (avoid) | Open (use) |
|-----------------|------------|
| "Will the entry points work as specified?" | "What could go wrong with the entry point configuration?" |
| "Is the migration strategy comprehensive?" | "What scenarios might the migration strategy miss?" |
| "Are there any issues with the approach?" | "What are the weakest parts of this approach?" |

**Author-selected focus** steers reviewers away from blind spots:

| Author-Selected (avoid) | Document-Driven (use) |
|--------------------------|----------------------|
| "Please review these 7 areas I've identified" | "Verify alignment with the Architecture doc and Roadmap" |
| "Focus on technical correctness and migration risk" | "Review against approved reference documents; flag any deviations" |
| "Key sections: 1, 2, 3, 5, 7" | "Review the complete spec; note which sections you found strongest and weakest" |

**Implicit approval expectation** frames review as validation:

| Approval-Oriented (avoid) | Critique-Oriented (use) |
|---------------------------|------------------------|
| "Please review and provide feedback" | "Your job is to find problems. If you find none, look harder." |
| "Let me know if you have any concerns" | "What are your top 3 concerns, even if you can't fully articulate them?" |
| "Please confirm the approach is sound" | "Where would you do something differently? Why?" |

**Comfort-seeking language** makes criticism feel optional:

| Comfort-Seeking (avoid) | Critique-Inviting (use) |
|--------------------------|------------------------|
| "Feel free to raise any concerns" | "What concerns you most about this?" |
| "If you notice anything" | "What did you notice that seems off?" |
| "Any suggestions would be helpful" | "What would you change and why?" |
| "I'd appreciate your thoughts" | "Where am I wrong?" |

### 7.3 Patterns That Work

These patterns consistently produce better review quality.

**Explicit permission to disagree.** State directly that disagreement is expected:

> **Your job is not to approve.** You are here to find problems.
>
> **The architect has blind spots.** They wrote this spec. They can't see their own assumptions. You can.
>
> **Be constructive, but don't be gentle.** The goal is to catch problems before implementation, not to validate the architect's work.

**Structured critique frameworks.** Give reviewers specific tables to fill, not open-ended requests:

> **What assumptions does this spec make that might be wrong?**
>
> | Assumption | Why It Might Be Wrong | Impact If Wrong |
> |------------|----------------------|-----------------|
>
> **What failure modes aren't addressed?**
>
> | Failure | How It Happens | Would We Notice? |
> |---------|----------------|------------------|

**Uncomfortable questions.** Explicitly invite the hard questions:

> Questions that challenge the architect's decisions:
> 1. [Question that pokes at a core assumption]
> 2. [Question that suggests an alternative approach]
> 3. [Question that exposes a gap in reasoning]
>
> **If something seems odd but you can't articulate why, flag it anyway.**

**Blind spot identification.** Ask for what the author missed, not what they got right:

> The spec author spent hours on this. They're probably right about the hard parts (they thought carefully about those). They're probably wrong about the "obvious" parts (they didn't think about those).
>
> **Where is the spec overconfident?**
> **What seems too simple?**
> **What's conspicuously absent?**

**Alternative approaches.** Force consideration of different paths:

> | Spec's Approach | Alternative Approach | Why Alternative Might Be Better |
> |-----------------|---------------------|--------------------------------|
>
> **You don't have to be right.** The point is to surface options the architect may not have considered.

**Severity stratification.** Force prioritization so real issues don't get buried:

> **Critical (Blocks Implementation):**
> [Only issues that would cause implementation to fail]
>
> **Major (Should Fix):**
> [Issues that would cause problems but aren't blocking]
>
> **Minor (Consider):**
> [Nice-to-haves and polish]
>
> **Do not list more than 3 Critical issues.** If you have more, you're miscalibrating severity.

### 7.4 Language Calibration

Quick reference for converting weak language to strong language in any prompt:

| Weak | Strong |
|------|--------|
| "Please review" | "Find the problems in" |
| "Any feedback" | "What are the top 3 issues" |
| "If you have concerns" | "What concerns you most" |
| "Feel free to" | "You must" |
| "It would be helpful" | "I need you to" |
| "Does this work" | "How does this break" |
| "Is this complete" | "What's missing" |
| "Looks good?" | "What's the weakest part" |
| "Confirm the approach" | "Challenge the approach" |
| "Any suggestions" | "What would you do differently" |
| "I think this is correct" | "Where is this wrong" |
| "Please validate" | "Please critique" |

### 7.5 Pre-Generation Checklist

Before sending any review prompt, verify:

- [ ] **No leading questions.** Questions don't presume answers.
- [ ] **No author-selected focus.** Review scope driven by reference docs, not author preference.
- [ ] **No approval expectation.** Prompt explicitly invites critique.
- [ ] **Role differentiation.** This reviewer has a distinct focus from the others.
- [ ] **Permission to disagree.** Explicit statement that criticism is expected.
- [ ] **Blind spot prompt.** Asks "what am I not seeing?"
- [ ] **Alternative prompt.** Asks "what would you do differently?"
- [ ] **Uncomfortable questions.** Invites challenging queries.
- [ ] **Reference documents cited.** Reviewers can verify alignment independently.
- [ ] **Structured critique frameworks.** Tables and frameworks, not open-ended asks.
- [ ] **Severity stratification.** Forces prioritization of issues.
- [ ] **Specific, not vague.** Actionable language throughout.
- [ ] **References confirmed.** Branch name, PR number, spec version are correct (Section 6.3).
- [ ] **Context budget checked.** Assembled prompt fits within 30-60% of effective window. If over, trimmed using priority order (Section 7.8.6). High-risk content positioned early.

### 7.6 Prompt Delivery Standards

**Hard rule: One prompt = one file.** Every prompt is delivered as a separate `.md` file. Not inline in chat. Not in code blocks. Not multiple prompts concatenated into one file. The orchestrator copies each file to the appropriate model — this only works if each prompt is its own self-contained document.

**Delivery method:**
1. Generate each prompt as a standalone `.md` file.
2. Name files using convention: `[Version]_[Phase]_[Step]_[Role].md`
3. Present all files for easy access.

**Why this matters operationally:** The orchestrator is copying prompts to 3-4 different model CLIs. Separate files make this mechanical: open file, copy, paste, done. Prompts embedded in chat or grouped into a single document create extraction overhead and increase the chance of copying the wrong content to the wrong model.

**Examples:**
- `v2.1_PhaseB_B1_Peer_Review.md`
- `v2.1_PhaseB_B1_Adversarial_Review.md`
- `v2.1_PhaseD_D2_Alignment_Review.md`

**Self-containment check.** Each file must include:
- [ ] Role assignment and mindset
- [ ] All reference document paths
- [ ] Complete review framework (tables, attack vectors, checklists)
- [ ] Output file naming instructions
- [ ] Critical instructions section

### 7.7 Prompt Suite Delivery

At each phase transition, **ask the orchestrator** how they want prompts delivered for this phase.

| Option | When Best | What's Delivered |
|--------|-----------|-----------------|
| **Full phase suite** | Straightforward project, approach is settled, ready to execute | All prompts for the phase, generated together with shared variables |
| **Step by step** | Complex project, orchestrator wants to tailor each prompt, or earlier steps may change what later steps need | One prompt at a time, orchestrator requests the next when ready |

**The CA should ask, not assume.** Different phases of the same project may warrant different approaches. Phase B on a simple patch might be full suite. Phase D on a complex feature with a large diff might be step by step so the orchestrator can adjust context and focus per reviewer.

**What "full phase suite" means:** All prompts within a phase that share the same variables (branch name, PR number, spec version). This is NOT "generate every prompt for the entire workflow." It's the prompts for the current phase only.

**Full suite contents by phase:**

| Phase | Suite Contains |
|-------|---------------|
| B (Spec Review) | B.1 Peer, B.1 Alignment, B.1 Adversarial, B.2 Consolidation |
| C (Implementation) | C.1 Implementation Prompt |
| D (Code Review) | D.1 CA PR Review, D.2 Peer, D.2 Alignment, D.2 Adversarial, D.3 Consolidation, D.4 Fix Summary template, D.5 Verification, D.6 Final Approval |

**Suite generation rules:**
1. All prompts reference the same branch name and PR number.
2. All prompts reference the same spec version.
3. Generate in workflow order.
4. Deliver as separate files (per 7.6).
5. If using tracks, generate one suite per track.
6. If any prompt in the suite exceeds context budget, flag it to the orchestrator with a recommended trim strategy before delivery.

### 7.8 Context Window Management

Every prompt competes for space in a finite context window. More context gives reviewers better grounding, but past a threshold, attention degrades — models skim instead of reading and produce shallower analysis. The goal is to maximize *useful* context, not total context.

#### 7.8.1 The Attention Curve Problem

| Context Load | Model Behavior | Review Quality |
|--------------|----------------|----------------|
| **Under 30%** of effective window | Full attention to all content | High, but may lack grounding if too little context |
| **30-60%** of effective window | Strong attention, good synthesis | **Sweet spot** — enough grounding, full engagement |
| **60-80%** of effective window | Begins prioritizing start and end, skimming middle | Adequate, but middle sections get less scrutiny |
| **80%+** of effective window | Significant attention loss, generic responses | Degrades noticeably — detail drops, frameworks get partially filled |

**Effective window is not the advertised window.** A model with a 200K token context window doesn't perform equally well at 190K as it does at 40K. Treat the effective window as roughly 40-60% of the advertised maximum for review-quality work.

#### 7.8.2 Context Budget by Phase

**Phase B: Spec Review**

| Content | Include? | Size Guidance |
|---------|----------|---------------|
| Review prompt (role, frameworks, instructions) | Always — full | ~1,500-2,500 tokens |
| Spec under review | Always — full | Varies (the whole point) |
| Architecture doc | Full if short (<3K tokens). Summary + relevant sections if long. | Extract constraints and layer rules only |
| Roadmap | Relevant section only (the section this spec implements) | Don't include the full multi-version roadmap |
| Strategy doc | Only if the spec involves strategic decisions | Usually a few paragraphs of relevant decisions |
| Prior specs (same release) | Don't include unless the spec explicitly references them | Cross-reference by name, don't embed |

**Phase C: Implementation Prompt**

| Content | Include? | Size Guidance |
|---------|----------|---------------|
| Implementation prompt (instructions, checklist) | Always — full | ~2,000-4,000 tokens |
| Approved spec v1.1 | Always — full | The developer needs every detail |
| Architecture constraints | Relevant constraints only (the ones this spec touches) | A focused list, not the full architecture doc |
| Existing code context | Only files being modified, and only the relevant sections | Don't dump entire files if only one function changes |

**Phase D: Code Review** — Context pressure is highest here. Diffs can be thousands of lines.

| Content | Include? | Size Guidance |
|---------|----------|---------------|
| Review prompt (role, frameworks, instructions) | Always — full | ~1,500-2,500 tokens |
| Diff / changed files | Always — but see diff management below | Varies significantly |
| Spec v1.1 | Relevant sections only (the sections this code implements) | Summarize overview, include technical design in full |
| Architecture constraints | Relevant constraints only | Focused list |
| Previous review outputs | Never for B.1/D.2 reviewers (prevents anchoring). Only for consolidation and verification. | Consolidation needs all three reviews |

#### 7.8.3 Diff Management for Code Review

Large diffs are the most common context window problem. Strategies in order of preference:

**1. Track splitting (preventive).** Split into tracks during Phase C (Section 15.4). Prevents the problem entirely.

**2. File grouping.** When the diff is too large for one prompt:

| Approach | When to Use | How |
|----------|-------------|-----|
| **Logical grouping** | Files cluster by subsystem | Group related files. "Review these 5 files that implement the export feature." |
| **Priority ordering** | Some files are higher risk than others | Put highest-risk files first in the prompt. Models attend best to early content. |
| **Staged review** | Diff exceeds effective window even with grouping | Split into 2-3 review passes with clear scope per pass. Consolidate findings across passes. |

**3. Context trimming.** When including full files, trim aggressively:

| Include | Exclude |
|---------|---------|
| Changed functions/methods — full | Unchanged imports (unless import changes are in scope) |
| Surrounding context (5-10 lines above/below changes) | Unchanged functions in the same file |
| New files — full | Test files that are simple and mechanical (e.g., `assert result == expected`) |
| Test files with non-obvious logic | Boilerplate, comments, docstrings in unchanged code |

#### 7.8.4 Reference Document Strategies

| Strategy | When to Use | Example |
|----------|-------------|---------|
| **Full document** | Document is short (<3K tokens) and most of it is relevant | A focused architecture constraints file |
| **Relevant section** | Document is long but one section is directly relevant | "Roadmap Section 6: CLI Completion" |
| **Extracted constraints** | You need specific rules from a larger document | "Architecture constraints relevant to this spec: (1) core/ must not import from io/, (2) all CLI commands return exit codes 0-3, (3) ..." |
| **Summary + pointer** | Document provides background context but isn't being verified line-by-line | "The Strategy doc (linked) established that backward compatibility is non-negotiable for v2.x. Key decisions: ..." |
| **Omit with citation** | Document exists for the record but isn't needed for this review | "Approved per Strategy v1.2 (not included — see [path] if needed)" |

#### 7.8.5 What to Never Cut

- **The review prompt itself** (role, frameworks, critique instructions). Cutting these undermines quality more than any context can compensate for.
- **The artifact under review** (spec, diff, code). If it doesn't fit, split the review — don't summarize what's being reviewed.
- **Severity stratification framework.** Without it, reviewers produce undifferentiated issue lists.
- **The "your job is to find problems" framing.** Removing this to save tokens is a false economy.

#### 7.8.6 What to Cut First

| Priority | Cut | Impact |
|----------|-----|--------|
| 1 (cut first) | Strategy/background documents not being directly verified | Low — these provide color, not substance |
| 2 | Full reference docs → extracted relevant sections | Low — reviewers rarely read the whole doc anyway |
| 3 | Unchanged code context around diffs | Low-Medium — reduces grounding but keeps focus on changes |
| 4 | Prior phase outputs (previous reviews, earlier spec versions) | Medium — loses historical context but prevents anchoring bias |
| 5 | Non-primary reference docs (keep architecture, cut strategy) | Medium — reduces alignment checking surface |
| 6 (cut last) | Sections of the artifact under review | High — avoid this; split the review instead |

#### 7.8.7 Model-Specific Considerations

| Consideration | Guidance |
|---------------|----------|
| **Larger effective window** | Assign to roles that need the most context (Alignment Reviewer checking multiple reference docs, Consolidation synthesizing three reviews) |
| **Smaller effective window** | Assign to roles with focused scope (Adversarial Reviewer attacking specific attack vectors, Peer Reviewer on a single spec section) |
| **Context-heavy phase** | If Phase D has a massive diff, consider assigning the largest-window model to the Adversarial Reviewer (who needs to see the most code to find edge cases) |

This interacts with Section 3 (Model Assignments). When context pressure is high, window capacity becomes a factor in model selection.

#### 7.8.8 Pre-Assembly Checklist

- [ ] **Prompt template size:** ~1,500-2,500 tokens (known in advance)
- [ ] **Artifact size:** How large is the spec or diff? If >50% of effective window, consider splitting.
- [ ] **Reference docs needed:** List them. For each, decide: full / relevant section / extracted constraints / omit with citation.
- [ ] **Total estimate vs. effective window:** Is the assembled prompt in the 30-60% sweet spot? If not, trim using the priority order (7.8.6).
- [ ] **High-risk content positioned early:** The most important content for this reviewer should appear in the first third of the prompt. Models attend to it most reliably.

### 7.9 Mid-Phase Intervention

The playbook defines decision gates between phases. This section covers what to do when things go wrong *during* a phase — when the orchestrator needs to intervene before a step completes.

**The default rule:** Let the step complete, then assess. Most problems are better handled at the next gate than by interrupting mid-stream. Intervene only when continuing would waste significant effort or propagate a problem that gets harder to fix later.

#### 7.9.1 Detection Triggers

These are signals that the current step has gone wrong and may need intervention before proceeding to the next step.

| Trigger | What You're Seeing | Severity |
|---------|--------------------|----------|
| **Sycophantic review output** | "Looks good", "No major issues", approval without evidence of testing. Reviewer filled in severity tables but everything is Minor. No uncomfortable questions. No assumption attacks. | High — output is useless |
| **Off-role review** | Peer Reviewer doing adversarial attacks. Adversarial Reviewer checking documentation quality. Alignment Reviewer suggesting UX improvements. | Medium — reduces role diversity value |
| **Truncated or incomplete output** | Review cuts off mid-section. Tables are partially filled. "Due to length constraints..." appears. Sections from the structured framework are missing. | High — incomplete analysis |
| **Spec gap discovered during implementation** | Developer reports: "The spec doesn't cover X", "This function signature doesn't exist", "The spec says modify but the file doesn't exist" | High — developer is blocked or improvising |
| **Unconstructive review** | 20+ issues, no severity stratification, style preferences mixed with real bugs, no clear blocking vs. non-blocking distinction | Medium — consolidation will struggle |
| **Model refusal or misinterpretation** | Model refuses to act as adversarial reviewer ("I don't want to be negative"), interprets the prompt as a request to write code rather than review it, or answers a different question than asked | High — output is wrong-shaped |
| **Anchoring in consolidation** | Consolidation output heavily mirrors the first review it processed. Disagreeements smoothed over. Unique findings from the second or third reviewer are downgraded or dropped. | Medium — undermines multi-perspective value |

#### 7.9.2 Intervention Actions

| Trigger | Don't | Do | If That Fails |
|---------|-------|-----|---------------|
| **Sycophantic output** | Proceed to consolidation (useless signal) | Re-prompt: "Your previous review found no significant issues. This is unlikely. Review again focusing on [2-3 specific areas]. Your job is to find problems." | Switch models. If two models both produce sycophantic reviews, check your prompt against Section 7.5 — your critique frameworks may be too weak. |
| **Off-role review** | Reject entirely (may contain useful findings) | Accept output, then supplement with focused follow-up: "Your previous review focused on [X]. This review needs [assigned role focus]. Specifically: [2-3 role-specific questions from Section 4]." | Strengthen role assignment: put role definition and "your focus is X, not Y" at the very top of the prompt. |
| **Truncated output** | Proceed with partial output (missing sections = missing coverage) | Re-prompt with reduced context — trim using Section 7.8.6 priority order. | Split review into two focused passes (e.g., spec sections 1-4, then 5-8). Consolidate findings from both. Prevention: follow Section 7.8.8 pre-assembly checklist. |
| **Unconstructive review** | Reject outright (even noisy reviews contain signal) | Request a severity pass: "You identified [N] issues. Categorize each as Critical/Major/Minor. Maximum 3 Critical. If you have more, you're miscalibrating severity." | Extract issues into a severity table yourself. If you can't determine severity from descriptions, the review is genuinely low-value. |
| **Model refusal** | Argue with the model | Reframe: "attack this spec" → "identify risks and edge cases to help improve it before implementation." Simplify prompt, core instruction in first sentence. | Switch models. Some models are better at adversarial roles than others. |
| **Anchoring in consolidation** | Proceed to CA Response (biased consolidation = skewed decisions) | Re-prompt: "For each issue found by ANY reviewer, list separately with attribution. Create Agreement Analysis comparing what each reviewer said about each topic." | Prevention: list all three reviews with equal framing. Don't label "primary"/"secondary." Require explicit "Reviewer A said X, Reviewer B said Y" format. |

**Spec gap during implementation** gets its own treatment because it has a decision threshold:

| Action | Details |
|--------|---------|
| **Don't** | Let the developer improvise. Undocumented deviations become Section 14 problems in Phase D. |
| **Do: Pause and report** | Developer stops and reports: what's missing, what they think the right approach is, whether it affects other parts of the spec. |
| **Micro-decision** | Gap resolvable in one sentence ("Use `raise ValueError` for invalid inputs"): CA decides, documents in brief spec addendum, developer continues. |
| **Return to spec** | Gap requires redesigning a component or changing the approach: stop implementation, update spec. May need targeted B.4-style verification. |

#### 7.9.3 When NOT to Intervene

Not every imperfect output needs intervention. The workflow has multiple checkpoints designed to catch problems downstream.

| Situation | Why Intervention Is Unnecessary |
|-----------|--------------------------------|
| One reviewer missed an issue the others caught | That's why you have three reviewers. Consolidation will surface it. |
| Review is good but not great | Diminishing returns. A 7/10 review with all framework sections filled is good enough. |
| Minor role overlap between reviewers | Some overlap is natural and even useful for the Agreement Analysis. Only intervene if a role's primary focus is completely absent. |
| Developer asks a clarifying question during implementation | This is normal and expected. Answer it and let them continue. Only intervene if it reveals a spec gap (see above). |
| Reviewer disagrees with the spec's approach | This is the point of review. Let it flow to consolidation and CA Response. Don't intervene to "correct" a reviewer's perspective. |

#### 7.9.4 Intervention Cost Awareness

Every intervention costs time and context window budget. Before intervening, ask:

1. **Will the next gate catch this?** If consolidation or CA Response would naturally handle the problem, don't intervene.
2. **Is the output salvageable?** Partial output with good findings is better than starting from zero. Extract what's useful before deciding to re-run.
3. **Am I intervening because of quality or preference?** If the review is substantive but formatted differently than you expected, that's not an intervention trigger.

---

# Tier 2: Phase Guides

---

## 8. Phase A: Spec Development

### 8.1 Purpose

Chief Architect creates a detailed implementation specification that the Review Board will validate and the Developer will execute. A weak spec cascades into every subsequent phase — invest the time here.

### 8.2 Process

```
A.1: Gather inputs (roadmap, architecture, prior decisions)
A.2: Scope the work
A.3: Research open questions
A.4: Write spec v1.0
A.5: Self-review against structural requirements
A.6: Submit for Phase B
```

#### A.1: Gather Inputs

Before writing anything, assemble the reference documents that constrain this spec.

| Input | What You're Looking For | Where It Lives |
|-------|------------------------|----------------|
| Roadmap | What this release must deliver. Specific deliverables, not themes. | `REFERENCE_DOCS` (Section 6.1) |
| Architecture | Constraints the spec must respect (layer rules, import restrictions, patterns). | `REFERENCE_DOCS` |
| Strategy decisions | Why decisions were made. Prevents re-litigating settled questions. | `REFERENCE_DOCS` |
| Prior specs (same release) | What's already been built. Avoid conflicts and duplication. | `INTERNAL_DOCS_DIR` |
| Open issues/findings | If coming from Pre-Phase A, the approved scope from triage or proposal review. | Pre-A outputs |

**Don't start writing until you've read these.** Specs written without checking the architecture doc are the #1 source of alignment issues in Phase B.

#### A.2: Scope the Work

Scoping is the hardest part of spec development and the most common source of Phase B failures. One release cycle (Phase A through merge) should take days, not weeks.

| Scoping Signal | Too Broad | Right-Sized | Too Narrow |
|----------------|-----------|-------------|------------|
| File count | >15 files across multiple subsystems | 5-15 files in related areas | 1-3 files, isolated change |
| Description length | Needs a paragraph | 2-3 sentences | One sentence |
| Open questions | >5 unresolved | 0-3, with recommendations | 0 (suspiciously none) |
| Dependencies | Multiple external blockers | 0-1 blockers with mitigations | None (may indicate missing awareness) |
| Estimated review complexity | Reviewers would need to split attention | Reviewers can hold full context | Reviewers finish in 10 minutes (overhead not justified) |

**"Suspiciously none" on open questions:** If a non-trivial spec has zero open questions, the architect probably hasn't thought hard enough about edge cases. Phase B will find them instead.

#### A.3: Research Open Questions

Not every spec needs research. But when open questions exist, resolve them before writing the technical design — not after.

| Research Method | When to Use | Output |
|----------------|-------------|--------|
| **Codebase investigation** | "How does X currently work?" | Factual answer with file:line references |
| **Prototype/spike** | "Will approach X work?" | Working proof-of-concept or definitive "no" with reasoning |
| **Model consultation** | "What are the tradeoffs between approaches A, B, C?" | Decision with rationale documented in Open Questions section |
| **Reference doc check** | "Has this been decided already?" | Citation of prior decision or confirmation that it's genuinely open |

**When to stop researching:** When you can write a specific technical design with clear implementation steps. Every open question needs a recommendation — "I don't know" is not a recommendation. "I think Option B because [reasoning], but the Review Board should weigh in" is.

#### A.4: Write Spec v1.0

Write against the structural requirements (Section 8.3). For each section, use the quality gate column to self-check as you go.

**Writing order that works well:**
1. **Overview and Scope first.** Forces clarity on what this spec is and isn't.
2. **Technical Design next.** The hardest section. Write it while your research is fresh.
3. **Dependencies and Migration/Compatibility.** These emerge from the technical design.
4. **Test Strategy.** Easier to write once you know the implementation approach.
5. **Risk Assessment last.** You can only assess risk after you understand the full design.
6. **Open Questions throughout.** Add these as you encounter them. Don't save them for the end.

**The "developer test" while writing:** After each technical design section, ask: "Could a developer implement this without asking me a question?" If no, the section needs more detail. Specific file paths, specific function signatures, specific behavior on edge cases. Vague specs produce implementations that deviate from intent.

#### A.5: Self-Review

Before submitting to Phase B, review your own spec against the structural requirements table (8.3) and the quality gate (8.6). This is not optional.

**Self-review checklist:**

- [ ] Read each row of the structural requirements table (8.3). Is that section present and does it meet the quality gate?
- [ ] Read the scope section. Is there anything ambiguous about what's in and out?
- [ ] Read the technical design section pretending you've never seen the codebase. Could you implement it?
- [ ] Check every file path and function name referenced in the spec. Are they accurate right now, or stale?
- [ ] Check open questions. Does each one have a recommendation? Are any marked "TBD" with no analysis?
- [ ] Check against the architecture doc. Does any proposed change violate a constraint?
- [ ] Read the risk assessment. Is it honest, or did you default to "LOW" because the change feels simple?

**The risk assessment honesty check:** If your risk level is LOW, ask yourself: "Would I bet my weekend that implementation and review go smoothly with no surprises?" If not, it's not LOW.

### 8.3 Structural Requirements for Spec

Every spec must contain these sections. The depth varies by scope, but the structure is mandatory.

| Section | Must Include | Quality Gate |
|---------|--------------|--------------|
| **Overview** | What this delivers, target version, theme, risk level | Reviewer can summarize in one sentence |
| **Scope** | In-scope items (checkboxes), out-of-scope with rationale | No ambiguity about boundaries |
| **Dependencies** | Prerequisites, blockers with mitigations | Nothing assumed |
| **Technical Design** | Per component: purpose, files (create/modify/delete), implementation approach | Developer can implement without questions |
| **Open Questions** | Questions, options, recommendations, status | All resolved before Phase B ends |
| **Test Strategy** | Unit tests, integration tests, manual testing steps | Reviewers can verify coverage |
| **Migration/Compatibility** | Breaking changes, deprecation paths, rollback plan | Alignment reviewer can verify compliance |
| **Risk Assessment** | Risk level, mitigations, monitoring | Adversarial reviewer can attack |

### 8.4 Common Phase A Failures

Patterns that reliably produce Phase B problems:

| Failure | What Happens in Phase B | Prevention |
|---------|------------------------|------------|
| **Skipped architecture check** | Alignment reviewer flags constraint violations that require redesign | A.1: Read the architecture doc before writing |
| **Vague technical design** | All three reviewers flag ambiguity. "What does 'refactor the handler' mean?" | A.4: Apply the developer test to every section |
| **Dishonest risk assessment** | Adversarial reviewer overrides risk level with evidence (see Section 9.7 example) | A.5: Apply the risk assessment honesty check |
| **Missing scope boundaries** | Reviewers disagree about what's in scope, producing conflicting feedback | A.2: Out-of-scope section with explicit rationale |
| **Open questions without recommendations** | Review board spends time solving problems the architect should have researched | A.3: Every open question gets a recommendation |
| **Stale file references** | Reviewer catches that `core/utils.py` was renamed two versions ago | A.5: Verify every file path against current codebase |
| **Scope too broad** | Reviews are superficial because reviewers can't hold full context | A.2: Apply scoping signals table |

### 8.5 Illustrative Example (Excerpt)

A well-structured Technical Design section:

```markdown
### 4.1 Export Command Refactor

**Purpose:** Consolidate three export formats into unified command with format flag.

**Files:**
- `commands/export.py` — MODIFY — Add format dispatcher, deprecate old functions
- `core/export_data.py` — CREATE — Format-agnostic export logic
- `core/types.py` — MODIFY — Add ExportFormat enum

**Implementation:**
1. Create ExportFormat enum with values: JSON, MARKDOWN, CLAUDE
2. Extract shared logic from existing _export_json(), _export_markdown() into export_data.py
3. Add --format flag to CLI with default MARKDOWN
4. Deprecate standalone export-json command (warn, don't remove)

**Constraints:**
- Must preserve existing JSON schema (backward compatibility)
- Must not import from UI layer (architecture constraint)
```

### 8.6 Quality Gate

Spec is ready for Phase B when:
- [ ] All structural sections present (checked against 8.3)
- [ ] No TBD or placeholder content
- [ ] Open questions have recommendations (even if not resolved)
- [ ] A developer unfamiliar with the project could implement from this spec
- [ ] Self-review checklist (A.5) completed
- [ ] File paths and function names verified against current codebase

---

## 9. Phase B: Spec Review

### 9.1 Purpose

Validate the spec before implementation begins. Catch issues when they're cheap to fix.

### 9.2 Structure

```
B.1: Review Board (Parallel)
     ├── Peer Reviewer
     ├── Alignment Reviewer
     └── Adversarial Reviewer
           │
           ▼
B.2: Consolidation
           │
           ▼
B.3: CA Response
           │
           ▼
B.4: Verification (if needed)
           │
           ▼
     Approved Spec v1.1 → Phase C
```

### 9.3 B.1: Review Board

Three reviewers work in parallel. Each receives a prompt built on Section 7 principles with role-specific focus.

#### Peer Reviewer Structural Requirements

| Section | Must Include |
|---------|--------------|
| Completeness Check | Missing sections, gaps in coverage |
| Quality Assessment | Design quality, clarity, implementability |
| UX Review | User-facing implications, documentation accuracy |
| Issues Found | By severity (Critical/Major/Minor) with specific locations |
| Positive Observations | What's done well (calibrates credibility) |
| Verdict | Approve / Request Changes |

**Critical instruction:** "Your job is to find problems. If you find none, look harder."

#### Alignment Reviewer Structural Requirements

| Section | Must Include |
|---------|--------------|
| Architecture Compliance | Layer violations, import rules, patterns |
| Roadmap Alignment | Does spec implement what was planned? |
| Constraint Verification | Each constraint checked with evidence |
| Backward Compatibility | Breaking changes identified, deprecation paths verified |
| Consistency Check | Conflicts with prior decisions |
| Deviation Report | Any spec divergence from approved docs |
| Verdict | Approve / Request Changes |

**Critical instruction:** "Deviations from approved documents are issues, not style choices."

#### Adversarial Reviewer Structural Requirements

| Section | Must Include |
|---------|--------------|
| Assumption Attack | Table: Assumption / Why It Might Be Wrong / Impact If Wrong |
| Failure Mode Analysis | Table: Failure / How It Happens / Would We Notice? |
| Edge Case Inventory | Specific inputs/scenarios that could break this |
| Security Surface | Attack vectors relevant to this spec |
| Blind Spot Identification | What's the architect not seeing? What seems too simple? |
| Risk Assessment Override | Does reviewer agree with CA's risk level? If not, why? |
| Verdict | Approve / Request Changes |

**Critical instruction:** "Your default stance is skeptical. The CA rated this [RISK LEVEL]. Prove them wrong."

### 9.4 B.2: Consolidation

Synthesize all three reviews into a decision-ready document.

| Section | Must Include |
|---------|--------------|
| Verdict Summary | Table: Reviewer / Role / Verdict / Blocking Issues Count |
| Blocking Issues | Combined list with issue ID, description, flagged by, category, action required |
| Should-Fix Issues | Non-blocking but recommended |
| Minor Issues | Consider but not required |
| Agreement Analysis | Where reviewers agree (strong signal) and disagree (needs CA judgment) |
| Required Actions | Prioritized list for CA with estimated effort |
| Decision Summary | Overall status: Ready / Needs Revision |

**Critical distinction in disagreements:**

| Disagreement Type | Blocking? | Resolution |
|-------------------|-----------|------------|
| Factual error in spec | Yes | CA must fix |
| Reviewer misunderstood spec | Yes | CA must clarify |
| Priority disagreement | No | Note dissent, CA decides |
| Scope disagreement | No | CA decides boundaries |

### 9.5 B.3: CA Response

Address all blocking issues. For each:
1. Accept (update spec) or Reject (with reasoning)
2. If rejected, provide evidence for override
3. Update spec to v1.1

Non-blocking issues: Accept, defer, or reject with brief rationale. No extended justification needed.

### 9.6 B.4: Verification & Split Verdict Resolution

**When to use:** CA made significant changes in B.3 (rejected recommendations, changed scope, resolved open questions differently than recommended).

**Who participates:** Only reviewers who flagged blocking issues in B.1.

**Verdicts allowed:**
- **Accept** — Issue resolved, ready for implementation
- **Accept with Note** — Resolved but reviewer wants concern on record
- **Challenge** — Issue NOT resolved, requires further CA attention

**If split verdict (some Accept, some Challenge):**

1. Consolidator distinguishes factual vs. priority disagreements
2. CA verifies contested facts independently
3. CA makes ruling — no fence-sitting
4. Documents dissenting position for the record
5. Updates spec if warranted

**After B.4:** Spec is approved. No further review rounds. If fundamental disagreements persist, the problem is scope — return to Pre-A or Phase A.

### 9.7 Illustrative Example: Good Adversarial Review (Excerpt)

From a real review where the Adversarial Reviewer overrode the CA's risk assessment:

```markdown
## Risk Assessment Override

- CA says: LOW
- Reviewer says: **HIGH**
- Reasoning: The change introduces a deterministic runtime failure 
  in the very commands it intends to fix.

## Attack 5: Regression in Working Commands

**Observed runtime failure (post-fix):**
All four "fixed" commands fail at runtime with:
`ModuleNotFoundError: No module named 'project.core'; 'project' is not a package`

| Attack | Applicable? | Risk Level | Notes |
|--------|-------------|------------|-------|
| Shared imports affected | **Yes** | **Critical** | Path insertion shadows the package |
| sys.path pollution | **Yes** | **High** | Insert at index 0 causes shadowing |

## Blind Spot Identification

**What's the CA not seeing?**
1. `scripts/project.py` shadows the package when SCRIPTS_DIR is inserted at index 0
2. Runtime execution fails even though `--help` works, so the testing plan misses the failure
3. Root cause affects more than 4 scripts; limiting scope is arbitrary

**Recommendation:** Request Changes
```

---

## 10. Phase C: Implementation

### 10.1 Purpose

Translate the approved spec into working code.

### 10.2 Structure

```
C.1: CA Writes Implementation Prompt
           │
           ▼
C.2: Developer Implements
           │
           ▼
C.3: PR Created → Phase D
```

### 10.3 C.1: Implementation Prompt

The CA writes a prompt that gives the Developer everything needed to implement the spec. Ambiguity here becomes bugs or spec deviations in Phase D.

#### Structural Requirements

| Section | Must Include |
|---------|--------------|
| Context | What this implements, target version, branch name |
| Spec Reference | Path to approved spec v1.1 |
| Implementation Checklist | Ordered tasks with checkboxes |
| File-by-File Instructions | For each file: action (create/modify), what to do, constraints |
| Test Requirements | What tests to write, what to verify |
| Commit Strategy | When to commit, message format |
| Smoke Test Commands | Commands to run before PR |
| PR Template | Title format, description structure |

**Critical instruction:** "Implement exactly what is specified. If something is unclear or seems wrong, stop and ask. Do not deviate from the spec without explicit approval."

#### Writing Effective File-by-File Instructions

This section is where most implementation prompt quality is won or lost. Each file entry should answer: what file, what action, what to do, and what constraints apply.

**Good file instruction:**
> `commands/export.py` — MODIFY
> - Add `format` parameter to `export()` with type `ExportFormat`, default `MARKDOWN`
> - Add format dispatcher: call `_export_json()` for JSON, `_export_markdown()` for MARKDOWN, `_export_claude()` for CLAUDE
> - Add deprecation warning to standalone `export-json` command: `warnings.warn("Use 'export --format json' instead", DeprecationWarning)`
> - Do NOT remove `export-json` command yet (backward compatibility)
> - Constraint: Must not import from `ui/` (architecture rule)

**Bad file instruction:**
> `commands/export.py` — MODIFY — Refactor to support multiple formats

#### Task Ordering

Order tasks so the developer can verify as they go:

| Order Principle | Rationale |
|-----------------|-----------|
| Dependencies first | Create new modules before modifying files that import them |
| Tests after each logical unit | Developer catches regressions incrementally, not all at the end |
| Risky changes early | If something is going to fail, find out before building on top of it |
| Mechanical changes last | Renames, formatting, docstring updates don't need early verification |

#### Context Window Considerations

Apply Section 7.8 principles. Key specifics for implementation prompts:

- Include the full spec for small-to-medium specs. For track-based implementation, include only relevant track sections.
- Include existing code only for files being modified, and only the relevant sections.
- Don't include Phase B review outputs. The spec already incorporates accepted changes.

### 10.4 Common Implementation Prompt Failures

| Failure | What Happens in Phase D | Prevention |
|---------|------------------------|------------|
| **Vague file instructions** | Developer interprets ambiguity differently than CA intended. Code review flags "spec deviation" that's really a prompt problem. | Apply the "could implement without interpretation" test to every file entry |
| **Wrong task order** | Developer creates a file that imports a module not yet created. Build fails mid-implementation. | Order dependencies first. Verify import chains before finalizing order. |
| **Missing constraints** | Developer uses an approach that violates architecture rules because the prompt didn't mention them. | Copy relevant constraints from spec into file-by-file instructions. Don't assume the developer will check the spec for constraints. |
| **Over-specified implementation** | Prompt dictates exact code rather than behavior. Developer follows literally, producing code that works but is poorly structured. | Specify what the code should do and what constraints it must respect. Let the developer choose how. |
| **Missing test guidance** | Developer writes tests that verify the happy path but miss the edge cases Phase B reviewers specifically flagged. | Reference specific scenarios from Phase B adversarial review findings in test requirements. |
| **No smoke test commands** | Developer creates PR without manual verification. Phase D catches issues that a quick CLI test would have found. | Always include concrete commands the developer can run. |

### 10.5 C.2: Developer Implements

Developer follows the prompt literally:
1. Read prompt completely
2. Complete pre-implementation checklist (tests pass, branch created)
3. Execute tasks in order
4. Run tests after each logical unit
5. Commit at checkpoints
6. Run final verification (all tests, smoke tests)
7. Create PR

**Spec gaps discovered during implementation:** Do not improvise. Flag the gap and request CA decision. See Section 7.9.2 for intervention actions and Section 14 (Spec Deviation Protocol) for post-review handling.

### 10.6 C.3: PR Created

PR includes:
- Clear title following project convention
- Description matching template (what, why, how to test)
- All tests passing
- Ready for Phase D

---

## 11. Phase D: Code Review

### 11.1 Purpose

Validate the implementation before merge. Catch bugs, regressions, and spec deviations.

### 11.2 Structure

```
D.1: CA PR Review
           │
           ▼
D.2: Review Board (Parallel)
     ├── Peer Reviewer
     ├── Alignment Reviewer
     └── Adversarial Reviewer
           │
           ▼
D.3: Consolidation
           │
           ▼
D.4: Developer Fixes (if needed)
           │
           ▼
D.5: Adversarial Verification
           │
           ▼
D.6: CA Final Approval → Merge
```

### 11.3 D.1: CA PR Review

First-pass review before the Review Board.

| Check | Question |
|-------|----------|
| Tests pass | Do all tests pass? |
| Spec compliance | Does code match spec v1.1? |
| Scope | Any scope creep? |
| Commits | Clean, atomic commits? |

**Output:** Brief verdict. "Ready for Review Board" or "Needs fixes first" with specific issues.

### 11.4 D.2: Review Board

Same three roles as spec review, but focused on code.

#### Peer Reviewer (Code) Structural Requirements

| Section | Must Include |
|---------|--------------|
| Spec Compliance | Does code match spec? Table by spec section. |
| Test Coverage | Are new features tested? Gaps identified. |
| Code Quality | Readability, maintainability, patterns |
| Documentation | Accurate? Complete? |
| Issues Found | By severity with file:line references |
| Verdict | Approve / Request Changes |

#### Alignment Reviewer (Code) Structural Requirements

| Section | Must Include |
|---------|--------------|
| Architecture Compliance | Layer violations, import rules |
| Backward Compatibility | CLI changes, output format changes, exit codes |
| API Stability | Public interface changes |
| Constraint Verification | Each constraint checked against actual code |
| Deviation Report | Code differs from spec v1.1 |
| Verdict | Approve / Request Changes |

#### Adversarial Reviewer (Code) Structural Requirements

| Section | Must Include |
|---------|--------------|
| Attack Vectors Tested | For each category: what you tried, what happened |
| Input Validation | Null, empty, long, special chars, boundaries |
| Error Handling | What happens when things fail? |
| State & Concurrency | Race conditions, resource leaks |
| Security | Injection, auth bypass, info leakage |
| Regression Check | Do existing features still work? |
| Issues Found | By severity with reproduction steps |
| Verdict | Approve / Block (with specific unblock requirements) |

**Critical instruction:** "NOT ACCEPTABLE: 'Looks good', 'I tried to break it and couldn't' (without specifics), any response under 200 words, approving without evidence of testing."

### 11.5 D.3: Consolidation

Same structure as B.2, but for code review findings.

| Section | Must Include |
|---------|--------------|
| Verdict Summary | Table: Reviewer / Role / Verdict / Blocking Issues |
| Blocking Issues | Must fix before merge |
| Should-Fix Issues | Recommended but not blocking |
| Minor Issues | Consider |
| Agreement Analysis | Consensus and disagreements |
| Required Actions for Developer | Prioritized fix list |
| Test Verification Requirements | How to verify each fix |

### 11.6 D.4: Developer Fixes

If blocking issues exist:
1. Address all blocking issues
2. One commit per logical fix
3. Post fix summary to PR

### 11.7 D.5: Adversarial Verification

**Why the Adversarial Reviewer verifies:** They found the hard bugs. They verify the fixes work.

| Check | Question |
|-------|----------|
| Issue addressed | Does the fix actually resolve the issue? |
| Regression | Did the fix break something else? |
| Tests | Do tests catch the original issue? |

**Output:** Approve or Request Further Fixes (with specific issues).

### 11.8 D.6: Final Approval & Retrospective

**Prerequisites:**
- [ ] Adversarial verification passed
- [ ] All blocking issues resolved
- [ ] All tests pass

**Approval includes:**
- Merge authorization
- Commit message with attribution
- Post-merge checklist (tag, changelog, etc.)

**Retrospective (add to approval):**

| Field | Content |
|-------|---------|
| Issues caught by review that would have shipped | List the saves |
| Review role that found most consequential issue | Which perspective was most valuable? |
| Anything review process missed (found later) | Feedback for process improvement |

### 11.9 Illustrative Example: Good Consolidation (Excerpt)

From a real consolidation that synthesized split verdicts:

```markdown
## Verdict Summary

| Reviewer | Role | Verdict | Blocking Issues |
|----------|------|---------|-----------------|
| CA | First-pass | Ready for Review Board | 0 |
| Gemini | Peer | Approve | 0 |
| Claude | Alignment | Approve | 0 |
| Codex | Adversarial | Request revision | 2 |

**Consensus:** 3/4 Approve
**Overall:** Needs Fixes

## Blocking Issues

| # | Issue | Flagged By | Category | Action Required |
|---|-------|------------|----------|-----------------|
| B1 | Export regression outside git repo | Codex | Regression | Restore behavior: bare export should work outside repo |
| B2 | Filtered export misclassifies migration status | Codex | Logic Bug | Compute classification on full graph before applying filters |

## Agreement Analysis

**Strong Agreement (3+ reviewers):**
- Spec compliance ✓
- Architecture compliance ✓
- Test coverage adequate ✓

**Disagreement:**
| Topic | Views | Recommendation |
|-------|-------|----------------|
| Robustness | Claude/Gemini: "Adequate" • Codex: "Gaps in error handling" | Accept Codex's concerns as should-fix |

## Required Actions for Developer

| Priority | Action | Addresses | Effort |
|----------|--------|-----------|--------|
| 1 | Fix export to work outside project context | B1 | Small |
| 2 | Compute classification on full graph before filtering | B2 | Medium |
```

### 11.10 PR Comment Order

All code review outputs posted as PR comments, in order:

| Order | Comment | From |
|-------|---------|------|
| 1 | CA PR Review | D.1 |
| 2 | Peer Code Review | D.2 |
| 3 | Alignment Code Review | D.2 |
| 4 | Adversarial Code Review | D.2 |
| 5 | Consolidation | D.3 |
| 6 | Fix Summary | D.4 (if needed) |
| 7 | Adversarial Verification | D.5 (if D.4 happened) |
| 8 | Final Approval | D.6 |

---

# Tier 3: Operational Protocols

---

## 12. Pre-Phase A: Triage & Validation

### 12.1 When to Use

After an audit, bug report, or discovery phase surfaces multiple items that need categorization before spec development begins.

**Trigger:** You have a pile of findings and need to decide what's in scope for this release.

### 12.2 Structure

```
Pre-A.1: CA Triage
           │
           ▼
Pre-A.2: Review Board Validation (Parallel)
     ├── Peer Reviewer
     ├── Alignment Reviewer
     └── Adversarial Reviewer
           │
           ▼
Pre-A.3: Consolidation
           │
           ▼
Pre-A.4: CA Response (if blocking challenges)
           │
           ▼
     Approved Scope → Phase A
```

### 12.3 Pre-A.1: CA Triage

CA categorizes each finding:

| Category | Meaning | Action |
|----------|---------|--------|
| **In Scope** | Will be addressed in this release | Proceed to Phase A |
| **Deferred** | Valid but not now | Add to backlog with rationale |
| **Rejected** | Not a real issue or already handled | Document reasoning |

### 12.4 Pre-A.2: Review Board Validation

Reviewers challenge triage decisions:

| Reviewer | Focus |
|----------|-------|
| Peer | Are deferred items actually deferrable? Are rejected items truly handled? |
| Alignment | Do scope decisions match roadmap priorities? |
| Adversarial | What's the risk of deferring each deferred item? |

### 12.5 Pre-A.3: Consolidation

**Critical distinction:**

| Challenge Type | Blocking? | Resolution |
|----------------|-----------|------------|
| CA misunderstood the finding | **Yes** | CA must re-evaluate with correct understanding |
| Reviewer disagrees with priority | No | Note dissent, CA decision stands |
| Factual error in triage rationale | **Yes** | CA must correct facts and re-evaluate |
| Reviewer wants broader scope | No | CA decides scope boundaries |

Only blocking challenges require CA response.

### 12.6 Pre-A.4: CA Response

Address ONLY blocking challenges:
1. Acknowledge the misunderstanding or factual error
2. Re-evaluate with correct information
3. Update triage decision if warranted
4. Document final decision with reasoning

---

## 13. Pre-Phase A: Proposal Review

### 13.1 When to Use

Developing new approaches, evaluating design directions, or scoping multi-feature releases before committing to specs.

**Trigger:** You have an approach to validate, not a pile of findings to categorize.

### 13.2 How It Differs from Spec Review

| Aspect | Proposal Review | Spec Review (Phase B) |
|--------|----------------|----------------------|
| Formality | Lighter structure | Full structured templates |
| Focus | Approach, philosophy, feasibility | Implementation correctness |
| Iteration | May iterate multiple times | One revision cap |
| Output | Refined proposal → feeds Phase A | Approved spec v1.1 → feeds Phase C |

### 13.3 Workflow

```
Proposal Development
        │
        ▼
Review Board reviews proposal
        │
        ▼
Consolidation
        │
        ▼
CA responds to findings
        │
        ▼
[Optional: Review Board validates response]
        │
        ▼
Finalized proposal → Phase A
```

**Key principle:** Proposal reviews are exploratory. The one-revision cap does NOT apply. However, proposals shouldn't iterate indefinitely.

### 13.4 Iteration Limits

| Round | Status | Action |
|-------|--------|--------|
| 1-2 | Normal | Continue iterating. Proposals often need refinement. |
| 3 | **Decision point** | The orchestrator must choose one of the options below. Do not start Round 4 without an explicit decision. |
| 4+ | Exceptional | Only if Round 3 decision was "one final round." If still not converging, escalate. |

**At Round 3, pick one:**

| Option | When to Use | What Happens |
|--------|-------------|--------------|
| **Split the proposal** | Scope is too broad. Reviewers are raising issues in different areas that don't interact. | Break into 2-3 focused proposals. Each one gets its own review cycle starting from Round 1. |
| **Timebox one final round** | Close to convergence but 1-2 open issues remain. | Constrain Round 4 to the specific unresolved issues only. No new scope. If it doesn't converge, escalate. |
| **Escalate to strategic decision** | Fundamental disagreement about approach, not refinement. | The problem isn't the proposal's quality — it's whether this is the right direction. Step back, revisit the strategic rationale, and decide whether to proceed, pivot, or abandon. |

---

## 14. Spec Deviation Protocol

### 14.1 When Triggered

Code review discovers implementation diverges from approved spec v1.1.

### 14.2 Two Options

No middle ground. Pick one:

| Option | When to Use | Actions |
|--------|-------------|---------|
| **A: Approve Deviation** | Deviation fixes a real problem the spec missed | Document rationale, update spec to v1.2, continue review against v1.2 |
| **B: Require Revert** | Deviation is unnecessary or introduces risk | Flag in code review, Developer reverts to match spec, continue review against v1.1 |

### 14.3 Decision Criteria

| Question | Leans Toward |
|----------|-------------|
| Does the deviation fix a problem the spec didn't anticipate? | Option A |
| Is the deviation necessary or could the spec's approach work? | Necessary → A, Not necessary → B |
| What's the risk of reverting? | High risk → A, Low risk → B |
| Was the spec thoroughly reviewed? | Yes → B (spec was probably right), No → A |
| Does the deviation change public API or user-facing behavior? | Yes → extra scrutiny, likely B unless critical bug |

**Who decides:** Chief Architect. Decision is final and must be documented in PR comments.

---

## 15. Phase Variants

### 15.1 Feature Phase (Default)

Standard workflow for new features.

| Aspect | Value |
|--------|-------|
| Risk | Medium-High |
| Focus | New functionality, architecture compliance |
| Breaking changes | May be acceptable |

### 15.2 Patch/Polish Phase

For bug fixes and polish after a release.

| Aspect | Value |
|--------|-------|
| Risk | Low-Medium |
| Focus | No regressions, backward compatibility |
| Breaking changes | **NOT acceptable** |

**Alignment Reviewer additions:**
- Check CLI compatibility
- Check output format stability
- Check exit code consistency
- Verify no API changes

**Adversarial Reviewer additions:**
- Check each fix fully addresses issue
- Check each fix doesn't break something else
- Verify test catches the regression

### 15.3 Hotfix Phase

Emergency fixes for production issues.

| Aspect | Value |
|--------|-------|
| Risk | High (speed vs. thoroughness) |
| Focus | Fix the issue, minimize blast radius |
| Process | Abbreviated: CA + Adversarial only |

### 15.4 Track-Based Implementation

For releases with multiple conceptually distinct work streams.

**Triggers:**
- >10 files changed across distinct subsystems
- 2+ features with minimal file overlap
- Context window too small for monolithic review (see Section 7.8 for assessment criteria)
- Independent verification is valuable

**Structure:**

```
Phase A → Phase B → Approved Spec v1.1
                          │
              ┌───────────┴───────────┐
              ▼                       ▼
         Track A                 Track B
     C.1a → C.2a → C.3a     C.1b → C.2b → C.3b
     D.1a → D.6a → Merge    D.1b → D.6b → Merge
```

**Rules:**
1. Single spec covers all tracks (approved together in Phase B)
2. Each track gets its own C and D cycle
3. **Default: Sequential execution.** Track A merges before Track B begins implementation. Track B's implementation prompt references Track A's merged state.
4. **Exception: Parallel execution** is acceptable when ALL of the following are true:
   - Zero file overlap between tracks (not "minimal" — zero)
   - No shared dependencies being modified (e.g., both tracks add to the same config file)
   - Each track can be tested independently without the other's changes
   - Orchestrator can manage two active C/D cycles simultaneously without confusion

**Why sequential is the default:** Merge conflicts and interaction bugs are common even with "minimal" overlap. Sequential also means Track B's prompt references the actual merged codebase, not a hypothetical future state.

**Benefits:**
- Focused context windows
- Independent verification
- Cleaner git history
- Failure isolation

**When NOT to use:**
- Changes are tightly coupled (files overlap significantly)
- Total scope is small enough for one context window
- Sequential dependency (Track B can't start until Track A's code exists)

---

## 16. Output Standards

### 16.1 PR Comments

Use collapsible sections for long content:

```markdown
## Summary

[Always visible — key findings and verdict]

---

<details>
<summary><strong>Detailed Analysis (click to expand)</strong></summary>

[Detailed content]

</details>
```

### 16.2 Review Structure

Every review output should have:
1. **Summary** — Always visible, key verdict
2. **Issues Found** — By severity
3. **Detailed Analysis** — In collapsible sections
4. **Verdict** — Clear recommendation

### 16.3 Issue Severity Levels

| Level | Meaning | Action |
|-------|---------|--------|
| **Critical/Blocking** | Cannot proceed | Must fix before continue |
| **Major** | Should fix | Fix before merge |
| **Minor** | Consider | Fix or defer |

### 16.4 File Naming Conventions

**Directory structure:**
```
[INTERNAL_DOCS_DIR]/
└── [version]/
    └── [phase]/
        ├── [Phase]-Implementation-Spec.md
        ├── [Version]_[Phase]_[Step]_[Role].md
        └── ...
```

**File naming pattern:**
```
[Version]_[Phase]_[Step]_[Role].md
```

**Examples:**
- `v2.1_PhaseB_B1_Peer_Review.md`
- `v2.1_PhaseD_D3_Consolidation.md`
- `v2.1_PhaseD_D6_Final_Approval.md`

---

## 17. Process Evaluation

### 17.1 Per-Release Retrospective

Added to D.6 Final Approval. Three fields:

| Field | Purpose |
|-------|---------|
| Issues caught by review that would have shipped | Quantify value of review process |
| Review role that found most consequential issue | Identify which perspective is most valuable |
| Anything review process missed (found later) | Feedback for improvement |

### 17.2 Periodic Review Quality Assessment

Every 3-5 releases, ask:

1. Which reviewer role surfaced the most costly issue?
2. Did any role produce consistently low-value feedback?
3. Were adversarial findings practical or theoretical?
4. Did the alignment reviewer catch deviations others missed?
5. What did the review process miss that was found post-merge?

### 17.3 When to Recalibrate

**Change model assignments when:**
- One model consistently misses issues another catches
- A model's strengths have shifted (new version, new capabilities)
- Project focus has changed (more security-critical, more UX-focused)

**Adjust role focus when:**
- One role's findings are consistently low-value
- Critical issues are being missed by all roles
- Review rounds are taking too long for the value delivered

---

## 18. Troubleshooting

These are post-hoc diagnoses — patterns you notice after a phase or release. For real-time intervention during a phase, see Section 7.9.

### 18.1 Reviews Are Too Gentle

**Symptoms:** "Looks good", "No major issues", approvals without evidence of testing.

**Causes:**
- Prompts use approval-seeking language
- Missing structured critique frameworks
- No explicit permission to disagree

**Fixes:**
- Apply Section 7.2 anti-patterns
- Add assumption attack tables and blind spot prompts
- Include "Your job is to find problems" in every review prompt

### 18.2 Reviews Are Unconstructive

**Symptoms:** Long lists of nitpicks, no severity stratification, style preferences treated as bugs.

**Causes:**
- Missing severity framework
- No distinction between blocking and non-blocking
- Reviewer not given clear focus

**Fixes:**
- Require severity stratification in output
- Add "Do not list more than 3 Critical issues"
- Clarify role-specific focus (peer vs. adversarial vs. alignment)

### 18.3 Too Many Review Rounds

**Symptoms:** Round 3, 4, 5 happening regularly.

**Causes:**
- Scope too large for one spec
- Requirements unclear
- One-revision cap not enforced

**Fixes:**
- Apply decision matrix (Section 2.2.1) rigorously
- Split scope into tracks (Section 15.4)
- Return to Phase A if Round 2 doesn't converge

### 18.4 Implementation Doesn't Match Spec

**Symptoms:** Code review finds significant deviations.

**Causes:**
- Spec was ambiguous
- Developer improvised instead of flagging gaps
- Spec gaps discovered during implementation but not communicated

**Fixes:**
- Apply Spec Deviation Protocol (Section 14)
- Strengthen implementation prompt: "If unclear, stop and ask"
- Improve spec quality gates (Section 8.6)

### 18.5 Consolidation Misses Issues

**Symptoms:** Issues flagged by reviewers not appearing in consolidation.

**Causes:**
- Consolidator summarizing instead of synthesizing
- Disagreements being smoothed over
- No structured template

**Fixes:**
- Require table format for all issues
- Mandate Agreement Analysis section
- Distinguish factual vs. priority disagreements explicitly

---

## 19. Quick Reference

### 19.1 Phase Checklist

#### Pre-Phase A: Triage (if applicable)
- [ ] Pre-A.1: CA triages findings (In Scope / Deferred / Rejected)
- [ ] Pre-A.2: Review Board validates triage
- [ ] Pre-A.3: Consolidate (distinguish misunderstandings from disagreements)
- [ ] Pre-A.4: CA responds to blocking challenges only

#### Pre-Phase A: Proposal Review (if applicable)
- [ ] Develop proposal
- [ ] Review Board reviews proposal
- [ ] Consolidate findings
- [ ] CA responds
- [ ] Finalize → proceed to Phase A

#### Phase A: Spec Development
- [ ] A.1: Gather inputs (roadmap, architecture, strategy, prior specs)
- [ ] A.2: Scope the work (apply scoping signals table)
- [ ] A.3: Research open questions (if any)
- [ ] A.4: Write spec v1.0 (against structural requirements 8.3)
- [ ] A.5: Self-review (checklist in 8.2, quality gate 8.6)
- [ ] A.6: Submit for Phase B

#### Phase B: Spec Review
- [ ] B.1: Review Board (Peer, Alignment, Adversarial) — parallel
- [ ] B.2: Consolidation
- [ ] B.3: CA Response → Spec v1.1
- [ ] B.4: Verification (if significant changes)
- [ ] Approved spec → Phase C

#### Phase C: Implementation
- [ ] C.1: CA writes implementation prompt
- [ ] C.2: Developer implements
- [ ] C.3: PR created → Phase D

#### Phase D: Code Review
- [ ] D.1: CA PR Review
- [ ] D.2: Review Board (Peer, Alignment, Adversarial) — parallel
- [ ] D.3: Consolidation
- [ ] D.4: Developer fixes (if needed)
- [ ] D.5: Adversarial verification
- [ ] D.6: CA final approval + retrospective
- [ ] Merge

#### Prompt Generation (every phase transition)
- [ ] Confirm branch name, PR number, spec version
- [ ] Ask orchestrator: full phase suite or step by step?
- [ ] Generate each prompt as its own .md file (hard rule)
- [ ] Each prompt is self-contained
- [ ] Context budget checked (30-60% of effective window)
- [ ] Deliver as separate files per 7.6

### 19.2 Quick Reference Cards

For prompt energy language substitutions, see Section 7.4.
For severity levels, see Section 16.3.
For the Round 2 decision matrix, see Section 2.2.1.

---

## Appendix A: Related Documents

Each project should maintain:

| Document | Purpose | Updated |
|----------|---------|---------|
| Roadmap | What to build, in what order | Per planning cycle |
| Architecture | Constraints, patterns, layer rules | When architecture changes |
| Strategy | Why decisions were made | When strategy changes |

These are the "approved documents" referenced throughout this playbook.

---

## Appendix B: Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-01-01 | Initial playbook |
| 2.0 | 2026-01-15 | Added templates, phase variants, troubleshooting |
| 3.0 | 2026-02-01 | Three-tier architecture. Project-agnostic rewrite. Added: Pre-Phase A workflows, spec deviation protocol, track-based implementation, process evaluation, one-revision cap decision matrix, verification protocols. Absorbed critique guide into Section 7. Removed: inline templates, automation architecture, signature blocks. |
| 3.1 | 2026-02-06 | Added: Context Window Management (7.8), Mid-Phase Intervention (7.9). Expanded: Phase A and Phase C with process guidance and common failures. Added: Proposal Review iteration limits (13.4), parallel track criteria (15.4). Merged Section 16 into 6.3. Renumbered 17-20 → 16-19. |
| 3.2 | 2026-02-06 | Compaction pass. Consolidated 7.9 intervention tables, eliminated Quick Reference duplicates, tightened prose throughout. Strengthened 7.6 file-per-prompt as hard rule with operational rationale. Rewrote 7.7 to ask orchestrator for delivery preference (full suite vs. step by step) instead of defaulting to full suite. Net: 2,044 → ~1,920 lines. |

---

*End of LLM Development Playbook v3.2*
