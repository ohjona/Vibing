# LLM Development Playbook: Multi-Agent Review Workflow

**Version:** 3.6.0
**Updated:** 2026-04-04
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
   - [15.5 Agent Team Workflow](#155-agent-team-workflow)
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
  C.0: CA investigation (if needed)  D.1: CA PR Review
  C.1: CA writes impl prompt         D.2: Review Board (3 models, parallel)
  C.2: Developer implements          D.3: Consolidation
  C.3: PR created ──────────────▶    D.4: Developer fixes (if needed)
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
| Review verdicts | `review-verdict.md` in project internal directory (see §16.5) | Named artifact for resumability and decision traceability |
| Decisions | PR comments | Audit trail |
| Specs | Project internal directory | Reference, version-controlled |
| Session logs | Project internal directory | Chronological record for validation runs, environment debugging, complex multi-attempt tasks. Records every decision, dead end, and learning. |

### 2.7 Fresh Prompt Generation

Every prompt is generated fresh for the current step. Never reference prompts from previous conversation turns or prior sessions.

| Don't | Do |
|-------|-----|
| "Use the Phase B prompt I generated earlier" | Generate a new Phase B prompt with current context |
| "Same as last time but change the PR number" | Full prompt with correct PR number embedded |
| "Refer to the implementation prompt above" | Complete, self-contained prompt |

**Self-containment test:** If you can't copy-paste a prompt into a fresh model session and have it work, the prompt is incomplete. Context windows reset between sessions; give each model everything it needs.

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
| Context-heavy phase (large diff, many reference docs) | Largest-window model for highest-context role (usually Adversarial or Consolidation). See Section 7.8.6. |

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

Define once per project, update rarely. These values are referenced throughout all prompts: `PROJECT_NAME`, `REPO_URL`, `INTERNAL_DOCS_DIR`, `TEST_COMMAND`, `SMOKE_TEST_COMMANDS`, `ARCHITECTURE_CONSTRAINTS`, `REFERENCE_DOCS`, `BRANCH_CONVENTION`, `MODEL_ASSIGNMENTS`, `DEVELOPER_TOOL`.

See `session-variables.md` for the complete template with descriptions, release variables, and filled examples.

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
| Test count (before) | Flat or declining count on a feature PR is a signal | `897 tests passing` |

A single wrong PR number propagates across 8+ prompt files. 30 seconds of confirmation prevents 30 minutes of corrections.

**Agent team workflow adjustments:** In Tier 1/2 workflows, the PR number doesn't exist at prompt generation time — the dev team creates it. Use `[PR_NUMBER]` placeholder in review team prompts, filled after dev team completion. Branch names are set by the orchestrator in the dev team meta-prompt, not by the dev team.

**If references change after prompt generation:**
1. Identify all affected files in the suite
2. Update systematically (search-and-replace)
3. Re-deliver corrected files
4. Confirm corrections with orchestrator

### 6.4 Multi-PR Sequence Management

When a proposal or release spawns multiple PRs (e.g., staged rollout, multi-track work across versions), maintain a **sequence ledger** — a living document updated after each PR merge.

**When to use:** Any proposal generating 2+ PRs/releases, or any multi-stage implementation where later stages depend on earlier ones.

**The ledger contains one table:**

| Item | Proposal Says | Codebase Reality (post-PR N) | PR That Changed It | Impact on Downstream |
|------|--------------|------------------------------|-------------------|---------------------|
| Cache key scheme | `provider:model:hash` | `provider:model:content_hash:config_hash` | PR #2 | PR #4 prompt must use actual scheme |
| Classification enum | 3 values | 5 values (added `image_blank`, `parse_error`) | PR #3 | PR #5 review must check all 5 |

**Rules:**
1. The orchestrator updates the ledger after each merge, capturing: what changed vs. the proposal, what the next PR needs to know, and any blocking decisions still pending.
2. The ledger is front-loaded into every subsequent PR's CA prompt and review prompt. It IS the living approved document — not the original proposal.
3. Stage 2+ in a sequence must run a C.0 readiness check (§10.3) against the post-merge codebase before skipping Phase A/B. The proposal can serve as the spec for multiple stages, but only if validated against accumulated drift.

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

| Anti-Pattern | Avoid | Use Instead |
|--------------|-------|-------------|
| **Leading questions** — presume an answer | "Are there any issues with the approach?" | "What are the weakest parts of this approach?" |
| **Leading questions** | "Is the migration strategy comprehensive?" | "What scenarios might the migration strategy miss?" |
| **Author-selected focus** — steers away from blind spots | "Please review these 7 areas I've identified" | "Verify alignment with the Architecture doc and Roadmap" |
| **Author-selected focus** | "Focus on technical correctness" | "Review against approved reference documents; flag any deviations" |
| **Implicit approval** — frames review as validation | "Please review and provide feedback" | "Your job is to find problems. If you find none, look harder." |
| **Implicit approval** | "Please confirm the approach is sound" | "Where would you do something differently? Why?" |
| **Comfort-seeking** — makes criticism optional | "Feel free to raise any concerns" | "What concerns you most about this?" |
| **Comfort-seeking** | "Any suggestions would be helpful" | "What would you change and why?" |

### 7.3 Patterns That Work

Every review prompt should include these patterns. The pattern names are the scaffolding; agents generate the specific prompt language from the principle.

| Pattern | Mechanism | Key Prompt Phrases |
|---------|-----------|-------------------|
| **Permission to disagree** | State directly that disagreement is expected | "Your job is not to approve. You are here to find problems." / "The architect has blind spots." |
| **Structured critique frameworks** | Give reviewers tables to fill, not open-ended asks | Assumption / Why Wrong / Impact table; Failure / How / Would We Notice? table |
| **Uncomfortable questions** | Explicitly invite hard questions that challenge core decisions | "If something seems odd but you can't articulate why, flag it anyway." |
| **Blind spot identification** | Ask for what the author missed, not what they got right | "Where is the spec overconfident? What seems too simple? What's conspicuously absent?" |
| **Alternative approaches** | Force consideration of different paths | Spec's Approach / Alternative / Why Better table. "You don't have to be right." |
| **Severity stratification** | Force prioritization so real issues don't get buried | Critical (blocks) / Major (should fix) / Minor (consider). "Max 3 Critical." |

### 7.4 Language Calibration

| Weak | Strong |
|------|--------|
| "Please review" | "Find the problems in" |
| "Any feedback" | "What are the top 3 issues" |
| "Feel free to" | "You must" |
| "Does this work" | "How does this break" |
| "Is this complete" | "What's missing" |
| "Looks good?" | "What's the weakest part" |
| "Confirm the approach" | "Challenge the approach" |
| "Please validate" | "Please critique" |

### 7.5 Pre-Generation Checklist

Before sending any review prompt, verify:

- [ ] **Anti-patterns avoided.** No leading questions, author-selected focus, or approval-seeking language (Section 7.2).
- [ ] **Critique patterns included.** Permission to disagree, structured frameworks, blind spot prompt, severity stratification (Section 7.3).
- [ ] **Role differentiation.** This reviewer has a distinct focus from the others.
- [ ] **Reference documents cited.** Reviewers can verify alignment independently.
- [ ] **References confirmed.** Branch name, PR number, spec version are correct (Section 6.3).
- [ ] **Context budget checked.** Assembled prompt fits within 30-60% of effective window (Section 7.8).

### 7.6 Prompt Delivery Standards

**Hard rule: One prompt = one file.** Every prompt is delivered as a separate `.md` file. Not inline in chat. Not in code blocks. Not multiple prompts concatenated into one file. The orchestrator copies each file to the appropriate model — this only works if each prompt is its own self-contained document.

**Delivery method:**
1. Generate each prompt as a standalone `.md` file.
2. Name files using convention: `[Version]_[Phase]_[Step]_[Role].md`
3. Present all files for easy access.

**Why:** The orchestrator copies each file to a different model CLI. Separate files make this mechanical and prevent copying the wrong content to the wrong model.

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
| **Full phase suite** | Straightforward project, approach is settled | All prompts for the phase, generated together with shared variables |
| **Step by step** | Complex project, or earlier steps may change what later steps need | One prompt at a time, orchestrator requests the next when ready |

**Full suite contents by phase:**

| Phase | Suite Contains |
|-------|---------------|
| B (Spec Review) | B.1 Peer, B.1 Alignment, B.1 Adversarial, B.2 Consolidation |
| C (Implementation) | C.1 Implementation Prompt |
| D (Code Review) | D.1 CA PR Review, D.2 Peer, D.2 Alignment, D.2 Adversarial, D.3 Consolidation, D.4 Fix Summary template, D.5 Verification, D.6 Final Approval |

**Suite generation rules:** All prompts reference the same branch/PR/spec version. Generate in workflow order. Deliver as separate files (per 7.6). If using tracks, one suite per track. If any prompt exceeds context budget, flag to orchestrator before delivery.

**Note:** Section 7.7 applies to manual orchestration (Tier 3). Agent team workflows (Tier 1/2) use a single meta-prompt per team — internal decomposition is the team lead's responsibility.

### 7.8 Context Window Management

Every prompt competes for space in a finite context window. Past a threshold, attention degrades — models skim instead of reading. The goal is to maximize *useful* context, not total context.

#### 7.8.1 The Attention Curve

| Context Load | Review Quality |
|--------------|----------------|
| **Under 30%** of effective window | High, but may lack grounding |
| **30-60%** of effective window | **Sweet spot** — enough grounding, full engagement |
| **60-80%** of effective window | Middle sections get less scrutiny |
| **80%+** of effective window | Degrades noticeably — frameworks get partially filled |

**Effective window ≈ 40-60% of advertised maximum** for review-quality work.

#### 7.8.2 Context Budget by Phase

| Content | Phase B (Spec) | Phase C (Impl) | Phase D (Code) |
|---------|---------------|----------------|----------------|
| Prompt (role, frameworks) | Always — full (~1.5-2.5K tokens) | Always — full (~2-4K tokens) | Always — full (~1.5-2.5K tokens) |
| Artifact under review | Spec — full | Spec v1.1 — full | Diff — full (see diff mgmt below) |
| Architecture doc | Full if <3K tokens, else extract constraints | Relevant constraints only | Relevant constraints only |
| Roadmap | Relevant section only | — | — |
| Strategy doc | Only if strategic decisions involved | — | — |
| Prior specs/reviews | Cross-reference, don't embed | Don't include Phase B outputs | Never for D.2 reviewers (prevents anchoring). Only for consolidation/verification. |
| Existing code | — | Only modified files, relevant sections | — |

#### 7.8.3 Diff Management for Code Review

Strategies in order of preference: (1) **Track splitting** during Phase C (Section 15.4) prevents the problem entirely. (2) **File grouping** by subsystem or risk level — highest-risk files first, models attend best to early content. If diff exceeds window even with grouping, split into 2-3 review passes. (3) **Context trimming** — include changed functions full with 5-10 lines surrounding context; exclude unchanged functions, mechanical tests, boilerplate.

#### 7.8.4 Reference Document Strategies

| Strategy | When to Use |
|----------|-------------|
| **Full document** | Short (<3K tokens) and mostly relevant |
| **Relevant section** | Long doc, one section directly relevant |
| **Extracted constraints** | Need specific rules from a larger document |
| **Summary + pointer** | Background context, not being verified line-by-line |
| **Omit with citation** | Exists for the record but not needed for this review |

#### 7.8.5 Context Trimming Priority

When assembled prompt exceeds the 30-60% sweet spot, trim in this order:

| Priority | Action |
|----------|--------|
| 1 (cut first) | Strategy/background documents not being directly verified |
| 2 | Full reference docs → extracted relevant sections |
| 3 | Unchanged code context around diffs |
| 4 | Prior phase outputs (loses history but prevents anchoring) |
| 5 | Non-primary reference docs (keep architecture, cut strategy) |
| 6 (cut last) | Sections of the artifact under review — split the review instead |
| **Never cut** | The review prompt itself, severity framework, "your job is to find problems" framing |

#### 7.8.6 Model-Specific Considerations

When context pressure is high, window capacity becomes a factor in model selection (interacts with Section 3). Assign larger-window models to high-context roles (Alignment checking multiple docs, Consolidation synthesizing three reviews). Assign smaller-window models to focused roles (Adversarial attacking specific vectors).

#### 7.8.7 Pre-Assembly Checklist

- [ ] **Artifact size:** If >50% of effective window, consider splitting.
- [ ] **Reference docs:** For each, decide: full / relevant section / extracted constraints / omit.
- [ ] **Total estimate:** In the 30-60% sweet spot? If not, trim using priority order (7.8.5).
- [ ] **High-risk content positioned early** in the first third of the prompt.

### 7.9 Mid-Phase Intervention

**Default rule:** Let the step complete, then assess. Intervene only when continuing would waste significant effort or propagate a problem that gets harder to fix later.

#### 7.9.1 Detection Triggers

| Trigger | Severity | Signal |
|---------|----------|--------|
| **Sycophantic review** | High | Approval without evidence, all issues Minor, no assumption attacks |
| **Off-role review** | Medium | Reviewer doing a different role's job (adversarial checking docs, peer doing security) |
| **Truncated output** | High | Review cuts off, tables partially filled, "due to length constraints..." |
| **Spec gap during implementation** | High | Developer blocked: "spec doesn't cover X" or "this file doesn't exist" |
| **Unconstructive review** | Medium | 20+ issues, no severity stratification, style mixed with real bugs |
| **Model refusal** | High | Refuses adversarial role, interprets review prompt as code-writing task |
| **Anchoring in consolidation** | Medium | Output mirrors first review processed, disagreements smoothed over |

#### 7.9.2 Intervention Actions

| Trigger | Action | If That Fails |
|---------|--------|---------------|
| **Sycophantic** | Re-prompt focusing on 2-3 specific areas. "Your job is to find problems." | Switch models. If two models are sycophantic, your critique frameworks are too weak (Section 7.5). |
| **Off-role** | Accept output, supplement with focused follow-up on the assigned role's mandate (Section 4). | Put role definition at the very top of the prompt: "your focus is X, not Y." |
| **Truncated** | Re-prompt with reduced context (trim per 7.8.5). | Split into two focused review passes. |
| **Unconstructive** | Request severity pass: "Categorize each as Critical/Major/Minor. Max 3 Critical." | Extract issues into a severity table yourself. |
| **Model refusal** | Reframe: "identify risks and edge cases to help improve before implementation." | Switch models. |
| **Anchoring** | Re-prompt: require explicit "Reviewer A said X, Reviewer B said Y" format per issue. | List all three reviews with equal framing, no "primary"/"secondary" labels. |

**Spec gap during implementation:**

| Severity | Action |
|----------|--------|
| Resolvable in one sentence | CA decides, documents in brief addendum, developer continues |
| Requires redesign | Stop implementation, update spec, may need B.4-style verification |
| **Never** | Let the developer improvise — undocumented deviations become Section 14 problems |

#### 7.9.3 Before Intervening, Ask

1. **Will the next gate catch this?** If consolidation or CA Response would naturally handle it, don't intervene.
2. **Is the output salvageable?** Partial output with good findings beats starting from zero.
3. **Am I intervening because of quality or preference?** A 7/10 review with all framework sections filled is good enough. One reviewer missing an issue the others caught is the system working, not failing. A reviewer disagreeing with the spec is the point of review — don't "correct" their perspective.

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

Write against the structural requirements (Section 8.3). Use the quality gate column to self-check as you go.

**The "developer test":** After each technical design section, ask: "Could a developer implement this without asking me a question?" If no, add detail: specific file paths, function signatures, edge case behavior.

#### A.5: Self-Review

Before submitting to Phase B, review against structural requirements (8.3) and quality gate (8.6):

- [ ] Every section in 8.3 present and meets quality gate?
- [ ] Diagrams pass quality criteria (8.3.1)? Every component in diagrams matches prose, error paths shown?
- [ ] Technical design passes the developer test?
- [ ] Every file path and function name verified against current codebase?
- [ ] Every open question has a recommendation?
- [ ] Architecture constraints not violated?
- [ ] Risk assessment honest? ("Would I bet my weekend nothing goes wrong?" If not, it's not LOW.)

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
| **Exclusion List** | What this spec does NOT do, files NOT to touch, approaches NOT to take | Scope creep prevention; reviewers can check compliance |
| **Diagrams** | At least two: (1) Architecture/Component diagram, (2) State Machine or Sequence diagram. See 8.3.1. | Every named component appears in both diagrams and prose; error paths shown |

#### 8.3.1 Diagram Requirements

**Hard gate.** No spec exits Phase A without at least two diagrams:

1. **Architecture/Component Diagram.** Shows system boundaries, data stores, external dependencies, and component relationships. Must name every component and show data flow direction.
2. **State Machine or Sequence Diagram.** Shows the core flow's lifecycle: states, transitions, triggers, and error/retry paths. If the feature has no meaningful state transitions, a sequence diagram showing the request/response flow between components is acceptable.

Additional diagram types (test matrix, data flow, dependency graph) are encouraged but not required for gate passage.

**Quality criteria — a diagram is insufficient if any of these fail:**

- Every named component in the diagram appears in the prose spec, and vice versa. Mismatches between diagram and prose are a blocking finding in Phase B.
- Error and failure paths are shown, not just the happy path. A state machine with no error transitions or a sequence diagram with no failure responses is incomplete.
- External dependencies and trust boundaries are visually distinct from internal components (different box style, border, or label prefix).
- Diagrams are text-based (ASCII, Mermaid, or PlantUML) so they live in version control and can be reviewed in diffs. No image-only diagrams.
- Diagrams are embedded in the spec document, not referenced as separate files. Reviewers must see diagrams and prose together.

### 8.4 Common Phase A Failures

| Failure | Prevention |
|---------|------------|
| Vague technical design | Apply the developer test to every section (A.4) |
| Dishonest risk assessment | Apply the honesty check (A.5) |
| Skipped architecture check | Read the architecture doc before writing (A.1) |
| Missing scope boundaries | Out-of-scope section with explicit rationale (A.2) |
| Open questions without recommendations | Every open question gets a recommendation (A.3) |
| Stale file references | Verify every file path against current codebase (A.5) |
| Missing or happy-path-only diagrams | Diagram gate (8.3.1): two required diagrams with error paths shown |

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

Spec is ready for Phase B when A.5 self-review checklist is complete and:
- [ ] No TBD or placeholder content
- [ ] A developer unfamiliar with the project could implement from this spec
- [ ] At least two diagrams present and passing quality criteria (8.3.1)

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

**The structural requirements below are fixed scaffolding** (severity tables, verdict format, output sections). **The attack vectors and investigation areas are always custom** — the orchestrator identifies the 2-3 highest-risk surfaces for each spec and builds the adversarial reviewer's attack tables around those. Reviewers should also be given the source data (field reports, research findings) when available, and invited to form their own independent conclusions before evaluating the spec's approach.

#### Peer Reviewer Structural Requirements

| Section | Must Include |
|---------|--------------|
| Completeness Check | Missing sections, gaps in coverage |
| Diagram-Prose Cross-Reference | Every component in diagrams appears in prose and vice versa. Mismatches are blocking (§8.3.1). |
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
| Diagram-Architecture Cross-Reference | Diagrams match architecture doc. Components, boundaries, and data flows in diagrams consistent with prose. Mismatches are blocking (§8.3.1). |
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
| Diagram Completeness Attack | Error paths described in prose but missing from state/sequence diagram? Components in prose absent from architecture diagram? Mismatches are blocking (§8.3.1). |
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
| Verdict Summary | Table: Reviewer / Role / Lens / Verdict / Blocking Issues Count. Lens defaults to "Technical" in spec and code reviews; populated from §13.4 in proposal reviews. |
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
C.0: CA Investigation (when applicable)
           │
           ▼
C.1: CA Writes Implementation Prompt
           │
           ▼
C.2: Developer Implements
           │
           ▼
C.3: PR Created → Phase D
```

### 10.3 C.0: CA Investigation

Before the CA writes the implementation prompt, the orchestrator sends a structured investigation prompt that forces the CA to verify assumptions against the actual codebase.

**When to use:**

| Trigger | Why |
|---------|-----|
| Agent team workflows (Tier 1/2) | Orchestrator lacks codebase access; can't verify assumptions directly |
| Integration PRs | Prior merges may have changed interfaces, data contracts, file structure |
| Multi-PR sequences (§6.4) | Proposal assumptions drift with each merged PR |
| First PR after proposal approval | Proposal was written against a point-in-time snapshot of the codebase |

**Skip when:** Trivial patch with 1-3 files, orchestrator has direct codebase access and has already verified, or the CA just completed a previous PR in the same session with fresh context.

**The investigation prompt contains 5-9 specific questions** the CA must answer from the codebase before writing anything. These are not generic — they target the known risk areas for this specific task.

**Question patterns that produce high-quality investigation:**

| Pattern | Example |
|---------|---------|
| Map coupling points | "List every file that imports from `core/cache.py` with the specific functions used" |
| Trace data flow | "Trace the cache key from creation through lookup for three specific scenarios: [A], [B], [C]" |
| Build gap table | "For each field in the ontology spec, identify the current source in the codebase or mark as 'not yet implemented'" |
| Verify interfaces | "What is the actual function signature of `assess_review_state()`? What does it return?" |
| Check assumptions | "The proposal says `ProviderInput` has a `system_prompt` field. Confirm or correct." |

**Output:** Investigation findings document. Where findings contradict the proposal or spec, the findings override. This document becomes raw material for the implementation prompt (C.1).

### 10.4 C.1: Implementation Prompt

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
| Exclusion List | What NOT to do: files not to touch, approaches not to take, scope boundaries |

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

### 10.5 Common Implementation Prompt Failures

| Failure | Prevention |
|---------|------------|
| Vague file instructions | Apply "could implement without interpretation" test to every file entry |
| Missing constraints | Copy relevant constraints from spec into file-by-file instructions — don't assume developer will check the spec |
| Missing test guidance | Reference specific edge cases from Phase B adversarial findings in test requirements |
| Over-specified implementation | Specify behavior and constraints, let developer choose how |

### 10.6 C.2: Developer Implements

Developer follows the prompt literally:
1. Read prompt completely
2. Complete pre-implementation checklist (tests pass, branch created)
3. Execute tasks in order
4. Run tests after each logical unit
5. Commit at checkpoints
6. Run final verification (all tests, smoke tests)
7. Create PR

**Spec gaps discovered during implementation:** Do not improvise. Flag the gap and request CA decision. See Section 7.9.2 for intervention actions and Section 14 (Spec Deviation Protocol) for post-review handling.

### 10.7 C.3: PR Created

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

Same three roles as spec review, but focused on code. Same principle: structural requirements are fixed scaffolding, but attack vectors are custom per PR. The orchestrator identifies the highest-risk surfaces (e.g., caching interactions, integration seams, user-facing output quality) and tailors the adversarial reviewer's focus accordingly.

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
| Diagram Verification | Implementation matches spec diagrams: components exist, data flows match, error paths from state/sequence diagrams are implemented |
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

Same structure as B.2 (Section 9.4) with two additions: **Required Actions for Developer** (prioritized fix list) and **Test Verification Requirements** (how to verify each fix).

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

### 13.4 Reviewer Configuration

Proposal Review now formalizes its reviewer composition. Unlike Phase B (which runs a full 3-reviewer board focused on technical execution), Proposal Review uses a 2-reviewer configuration with explicit lens assignments.

**Recommended configuration:**

| Reviewer | Posture | Lens | Focus |
|----------|---------|------|-------|
| Reviewer 1 | Adversarial | Product | Is this the right thing to build? |
| Reviewer 2 | Alignment | Technical | Is the proposed approach sound at a high level? |

**Why Adversarial + Product:** This is the pairing most likely to surface "you're solving the wrong problem" — the finding least likely to emerge from technical review alone. Product-lens review in Phase B is too late; the Phase A effort has already been spent.

**Product-lens prompt preamble:**

```
YOUR LENS: Product

You are reviewing whether this is the right thing to build. Focus on:
problem-solution fit, scope calibration, simpler alternatives, operational
simplicity, and whether the problem statement reflects the user's actual need.

You are NOT reviewing architecture, code quality, or test coverage — that
happens in Phase B. However, flag any cross-cutting concerns where a product
decision has obvious technical consequences.
```

**Consolidation verdict table for Proposal Review:**

| Reviewer | Posture | Lens | Verdict | Blocking Issues |
|----------|---------|------|---------|-----------------|

**Agreement Analysis note:** Product-lens vs. Technical-lens disagreements are the highest-signal finding type. When the two reviewers reach opposite verdicts, surface both views explicitly — do not smooth them into a compromise.

**What the Product lens does NOT gate:** Phase B reviewer configuration is unchanged. The 3-reviewer board with 2+ model families remains mandatory. The Product lens applies only in Proposal Review (§13).

### 13.5 Iteration Limits

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

#### Evidence-Based Patch Mode

For patches driven by observational data (field reports, profiling results, user feedback, error logs) rather than a requirements document.

**Entry criteria:** Data has been collected from a deployed system, the CA can go straight to implementation without a formal spec cycle, and the data itself is available for independent analysis.

**Workflow differences:**
- **Phase A/B collapsed.** The field data serves as the input; the CA writes the implementation prompt directly (with C.0 investigation if applicable).
- **Review anchors against the implementation prompt** as the de facto spec, plus the raw field data.
- **Reviewers get the source data** and are explicitly invited to form their own conclusions before evaluating the CA's approach: "Read the field data independently. Does the implementation address the right problems? Are there issues the CA's analysis missed?"
- The risk of skipping spec review is compensated by giving reviewers more latitude to challenge the approach, not just the implementation.

### 15.3 Hotfix Phase

Emergency fixes for production issues.

| Aspect | Value |
|--------|-------|
| Risk | High (speed vs. thoroughness) |
| Focus | Fix the issue, minimize blast radius |
| Process | Abbreviated: CA + Adversarial only |

#### Validation / Observation Runs

Not a development phase, but a gate protocol for tasks that run deployed code and report results without changing it. Used for: field testing after deployment, performance benchmarking, corpus processing for data collection, pre-release acceptance testing.

**Key rules:**
- **Do not fix pipeline/logic bugs during the run.** Only infrastructure/environment fixes (permissions, paths, tooling) are allowed. If a pipeline bug is found, abort, fix in a separate PR, re-run.
- **Every intervention must be documented** in a session log (§2.6) with the exact change and rationale.
- **The output is a structured report** that feeds back into the next planning cycle — typically as input to an Evidence-Based Patch (§15.2) or a new Phase A spec.
- **Preflight gates:** Define abort-early conditions before starting (e.g., "if test suite fails, abort before full corpus run"). Saves time when the full run would fail.

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

**When NOT to use:** Changes are tightly coupled (significant file overlap), total scope fits one context window, or Track B depends on Track A's code existing.

### 15.5 Agent Team Workflow

For releases executable by two agent teams (one per model family) with reduced manual orchestration. Replaces the human orchestrator as "message bus" between roles.

**Prerequisites:** Claude Code Agent Teams enabled (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`), Codex with equivalent multi-agent capability, an orchestrator session (typically Claude web) for prompt generation.

#### 15.5.1 Orchestration Tiers

Select tier based on **risk characteristics**, not version number alone. Escalate when a patch touches higher-tier characteristics.

| Tier | Default Mapping | Risk Characteristics | Orchestration Mode |
|------|-----------------|---------------------|--------------------|
| **Tier 1: Agent Team Review** | Patch (x.y.Z) | Few files, no API changes, no new subsystems, well-tested paths | Two agent teams, minimal human involvement |
| **Tier 2: Guided Multi-Model** | Minor (x.Y.z) | New features, API surface changes, multiple subsystems, within established architecture | Agent teams with hands-on orchestrator guidance |
| **Tier 3: Full Orchestration** | Major (X.y.z) | Architectural changes, new subsystems, breaking changes | Full playbook, manual orchestration, maximum model diversity |

**Escalation triggers** (override default mapping):

| If a Patch Touches... | Escalate To |
|------------------------|-------------|
| Public API surface | Tier 2 |
| Core graph/data model logic | Tier 2 |
| Multiple subsystems | Tier 2 |
| Security-sensitive paths | Tier 2 or 3 |
| Architectural patterns (new patterns, layer violations) | Tier 3 |

#### 15.5.2 Tier 1: Team Compositions

Two agent teams, two model families. The orchestrator generates two meta prompts (15.5.4, 15.5.5), then makes two handoffs.

**Development Team (Codex):** CA agent as team lead writes the spec, decides team decomposition, spawns Developer agent(s), reviews output, and gives final approval. Developer agents are spawned dynamically based on the task.

**Review Team (Claude Code):** Lead/Consolidator spawns three fixed reviewers (Peer, Alignment, Adversarial per Section 4), synthesizes findings into standard consolidation format (Section 11.9).

**Critical asymmetry:**

| Aspect | Development Team | Review Team |
|--------|-----------------|-------------|
| Composition | **Dynamic.** CA decides how many developers and what each owns. | **Fixed.** Always three reviewers with predefined mandates. |
| Lead's role | Decision-maker: decomposes work, spawns agents, approves | Synthesizer: consolidates findings, surfaces disagreements |
| Why | "How to build" benefits from task-specific judgment | "What to check" must not be filtered through the reviewing model's assumptions |

#### 15.5.3 Tier 1: Workflow

```
Orchestrator Session (Claude web + playbook)
        │
        ├──► Development Team Prompt ──► Codex Agent Team
        │                                   CA Agent writes spec
        │                                   CA Agent spawns Developer(s)
        │                                   Developer(s) implement
        │                                   CA Agent reviews implementation
        │                                   Output: PR / diff
        │
        ├──► Review Team Prompt ──────► Claude Code Agent Team
        │    (includes PR/diff                Lead spawns 3 reviewers
        │     from Dev Team)                  Reviewers review in parallel
        │                                     Reviewers debate via messaging
        │                                     Lead consolidates
        │                                     Output: Consolidated review
        │
        └──► Consolidated review ──────► Codex CA Agent (cross-check)
                                            Evaluates review findings
                                            Accepts, pushes back, or requests fixes
                                            Output: Final approval or fix requests
```

**Human checkpoints (Tier 1):** After prompt generation (verify scope), after consolidated review (scan for single-model blind spots), after final approval (sanity check before merge).

**Handoff count:** 3, down from 6+ in manual orchestration.

#### 15.5.4 Meta Prompt: Development Team

A **constitution**, not a script. Defines the CA agent's authority and constraints; the CA makes tactical decisions about team composition.

**Platform note:** The `## Agent Team Activation` section uses generic language. When generating prompts, the orchestrator should adapt to the target platform's vocabulary (e.g., Claude Code uses "spawn teammates," Codex may use different terminology). The directive must be unmistakable: models default to solo execution unless explicitly told to spawn separate agents.

```markdown
# Development Team: [TASK_SUMMARY]

## Agent Team Activation

This task REQUIRES an agent team. You are the team lead. You MUST spawn separate agents for the Developer role(s) described below. Do NOT attempt to fulfill all roles yourself in a single session. Each agent runs in its own independent context window.

Workflow: You (CA) write the spec and decomposition plan first, THEN spawn Developer agent(s) as separate teammates, THEN verify their output.

## Your Role: Chief Architect (Team Lead)

You write the implementation spec, decompose the work, delegate to Developer agent(s), and verify the output.

## Task Context

[PATCH_SCOPE: what changed and why]
[BRANCH_NAME]
[BASE_BRANCH]
[RELEVANT_FILES: files expected to be touched]
[TEST_COMMAND]
[SMOKE_TEST_COMMANDS]
[COMMIT_FORMAT: e.g., "fix(export): description"]
[PR_TITLE_FORMAT: e.g., "fix: description (#PR)"]

## Reference Documents

[SPEC or ISSUE or BUG_REPORT]
[ARCHITECTURE_DOC: relevant sections or constraints]
[PRIOR_DECISIONS: any relevant context from previous releases]

## Phase 1: Spec

Write a brief implementation spec: what changes and why, technical approach, files affected, constraints from reference documents, test strategy.

## Phase 2: Team Decomposition

Before spawning Developer agents, declare your plan: how many developers, what each owns (specific files/directories), why this decomposition (or why one developer suffices).

Rules:
- Each Developer agent owns distinct files. No overlapping edits.
- Maximum [TIER_1_MAX: 2-3] Developer agents for Tier 1 tasks.
- If one developer suffices, use one. Don't over-decompose.

## Phase 3: Developer Handoff

For each Developer agent, provide: relevant spec section, exact files they own, constraints (backward compatibility, no API changes, etc.), and these instructions:
- "If the spec is unclear or you discover a gap, stop and message me. Do not improvise."
- Create branch `[BRANCH_NAME]` from `[BASE_BRANCH]` (if not already created).
- Commit after each logical unit of work. Message format: `[COMMIT_FORMAT]`.
- Run `[TEST_COMMAND]` after each commit.

## Phase 4: Verify and Summarize

When all developers complete:
- Verify implementation matches spec
- Check for integration issues between developers' work
- Run full test suite: `[TEST_COMMAND]`
- Run smoke tests: `[SMOKE_TEST_COMMANDS]`
- Create PR to `[BASE_BRANCH]` with title format `[PR_TITLE_FORMAT]` and description covering: what changed, why, how to test
- Prepare a summary of what was built for the Review Team
```

#### 15.5.5 Meta Prompt: Review Team

**Prescriptive** about team composition. The lead synthesizes but does not decide what gets reviewed.

```markdown
# Review Team: [TASK_SUMMARY]

## Agent Team Activation

This task REQUIRES an agent team. You are the team lead. You MUST spawn exactly three separate reviewer agents as described below. Do NOT attempt to fulfill all reviewer roles yourself in a single session. Each reviewer runs in its own independent context window with its own mandate.

Workflow: You (Lead) spawn three reviewer teammates, THEN wait for all three to complete, THEN consolidate their findings.

## Your Role: Review Lead (Team Lead)

You coordinate the review of a patch produced by a separate development team. Your job is to synthesize findings, NOT to decide what gets reviewed.

## Task Context

[PATCH_SCOPE: what changed and why]
[PR_DIFF or CODE_OUTPUT from Development Team]
[DEV_TEAM_SUMMARY: what the CA agent says was built]

## Reference Documents

[SPEC: the implementation spec from the Dev Team CA]
[ARCHITECTURE_DOC: relevant sections]
[PRIOR_DECISIONS: relevant context]

## Team Structure (Fixed — Do Not Modify)

Spawn exactly three reviewers:

**Reviewer 1: Peer Reviewer** — Completeness, quality, clarity. Core question: "Is anything missing, overcomplicated, or unmaintainable?"

**Reviewer 2: Alignment Reviewer** — Architecture compliance, constraints, backward compatibility. Core question: "Does this match approved documents? Does it break existing behavior?"

**Reviewer 3: Adversarial Reviewer** — Failure modes, edge cases, regressions, security. Core question: "How does this break? What assumptions are wrong?"

## Review Rules

- Each reviewer works independently in their own context.
- Reviewers challenge each other's findings via messaging. Disagreement is signal.
- The architect has blind spots. Your job is to find problems. If you find none, look harder.
- Anchor reviews to reference documents, not the dev team's framing.

## Consolidation

After all three reviewers report, synthesize into: Verdict Table, Blocking Issues, Agreement Analysis (with explicit disagreement preservation), and Required Actions. Use the consolidation format from the playbook's Section 11.9.

Critical rules:
- Do NOT drop single-reviewer findings. They're often the most valuable.
- Do NOT smooth over disagreements. Report both views.
- DO flag if findings suggest Tier 2 escalation.
```

#### 15.5.6 Tier 2 and Tier 3 Adjustments

**Tier 2 (Guided Multi-Model):** Same structure as Tier 1 with: dev team cap raised to 5 developers, orchestrator actively reviews spec before dev starts and findings before CA cross-check, consider adding one Gemini reviewer for cross-model diversity, CA cross-check becomes a thorough evaluation with explicit response document, may combine with Section 15.4 for multi-track work.

**Tier 2 orchestrator checkpoint checklist** (when reviewing CA's spec before dev starts):
- [ ] Does the spec match the proposal / approved scope?
- [ ] Did the CA discover anything surprising in the codebase?
- [ ] Is the decomposition reasonable (right number of developers, clean file boundaries)?
- [ ] Any scope creep signals (files or subsystems not in the original scope)?
- [ ] For multi-PR sequences: does the spec incorporate the sequence ledger findings (§6.4)?

**Proposal review via agent teams:** When running a proposal review (§13) through agent teams, key differences from spec review: the review team focuses on approach and feasibility rather than implementation correctness, consolidation produces open questions for the architect rather than blocking issues for the developer, and the one-revision cap does NOT apply.

**Tier 3 (Full Orchestration):** Revert to standard playbook (Phases A-D). Agent teams may assist within individual phases, but the orchestrator manually manages phase transitions with full model diversity per Section 3.

#### 15.5.7 When NOT to Use Agent Team Workflow

| Situation | Why | Use Instead |
|-----------|-----|-------------|
| First release on a new codebase | No established patterns for CA to decompose against | Tier 3 |
| Scope unclear or still evolving | Agent teams need a defined task; ambiguity causes drift | Pre-Phase A first |
| Security-critical changes | Single-model review board is insufficient | Tier 2 minimum, Tier 3 preferred |
| Post-mortem/incident response | Needs human judgment at every step | Tier 3 with abbreviated phases (Hotfix, 15.3) |
| Agent team tooling is unstable | Experimental features may behave unpredictably | Manual orchestration |

#### 15.5.8 Monitoring Agent Teams

Same failure modes as manual orchestration (Section 7.9), plus agent-team-specific issues:

| Failure Mode | Signal | Intervention |
|--------------|--------|--------------|
| Solo execution (no team spawned) | All roles in one session, no teammates visible | Re-prompt: "Create an agent team" as literal first instruction. Verify platform supports it. |
| Developer improvising | CA approves spec gap without updating spec | Pause, apply spec gap protocol (§7.9.2) |
| Review lead smoothing disagreements | No disagreement section, or "minor differences in emphasis" | Re-prompt with consolidation rules, or manually extract reviewer outputs |
| Single-model blind spot | All reviewers approve an area that feels risky | Known limitation. CA cross-check is the safety net. Escalate to Tier 2 if uncomfortable. |
| Teammate drift | Reviewer doing another role's job | Re-spawn the drifting reviewer with clearer mandate |
| Token budget exceeded | Stalled or truncated output | Reduce context per reviewer (§7.8.5) |

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
[Version]_[Phase]_[Step]_[Role]_[TaskID].md
```

The `[TaskID]` is a PR number, spec name, or short descriptor. Required for multi-PR projects to avoid ambiguous filenames; optional for single-PR releases.

**Examples:**
- `v2.1_PhaseB_B1_Peer_Review_PR2.md`
- `v2.1_PhaseD_D3_Consolidation_CacheFix.md`
- `v2.1_PhaseD_D6_Final_Approval_PR8.md`

### 16.5 Review Verdict Artifact

After B.2 consolidation (Spec Review) and D.3 consolidation (Code Review), save the output as a named file. This is the primary artifact for resumability and decision traceability.

**File location:**
```
[INTERNAL_DOCS_DIR]/reviews/<feature-slug>-<phase>-verdict.md
```

The `<phase>` suffix (B or D) distinguishes spec review from code review verdicts for the same feature. Examples:
- `.project-internal/reviews/auth-refactor-B-verdict.md`
- `.project-internal/reviews/auth-refactor-D-verdict.md`

**Context header (prepend to existing consolidation output):**

```markdown
# Review Verdict: [Feature Name]

**Phase:** B (Spec Review) | D (Implementation Review)
**Date:** YYYY-MM-DD
**Spec/PR:** [link or filename]
**Reviewers:** [model, posture, lens] for each reviewer
**Overall Status:** Approve | Needs Fixes | Needs Re-Scoping

---
```

The body is the standard consolidation output (verdict table, blocking issues, agreement analysis, required actions). No new format required — only the context header is added.

**What this enables:**

| Use Case | How |
|----------|-----|
| Resumability | Start a fresh session, point it at `review-verdict.md` + C.1 prompt — it knows what was found and decided without re-explaining |
| Decision traceability | When something breaks later, trace back through verdict files to find where the assumption was approved |
| Cold-start for implementation | A fresh session with C.1 + B-verdict.md should be able to execute without clarifying questions |

**No new tooling required.** The orchestrator (or the consolidation session) saves the output as a file. The gate criteria and review process are unchanged — this is a save step, not a new workflow step.

**Validation signal:** If a fresh session given only `review-verdict.md` + C.1 asks clarifying questions about prior decisions, strengthen C.1's self-contained-ness rather than adding more artifacts.

---

## 17. Process Evaluation

### 17.1 Per-Release Retrospective

Added to D.6 Final Approval. Three fields:

| Field | Purpose |
|-------|---------|
| Issues caught by review that would have shipped | Quantify value of review process |
| Review role that found most consequential issue | Identify which perspective is most valuable |
| Anything review process missed (found later) | Feedback for improvement |

### 17.2 When to Recalibrate

Every 3-5 releases, assess: Were adversarial findings practical or theoretical? Did alignment catch deviations others missed? What did review miss that was found post-merge?

**Change model assignments when** one model consistently misses issues another catches, a model's strengths have shifted, or project focus has changed. **Adjust role focus when** one role's findings are consistently low-value or review rounds take too long for the value delivered.

---

## 18. Troubleshooting

Post-hoc diagnoses — patterns you notice after a phase or release. For real-time intervention, see Section 7.9.

| Symptom | Root Cause | Fix |
|---------|-----------|-----|
| Reviews too gentle ("Looks good", no evidence of testing) | Approval-seeking language, missing critique frameworks | Apply anti-patterns (§7.2), add structured critique (§7.3) |
| Reviews unconstructive (nitpick lists, no severity) | Missing severity framework, unclear role focus | Require severity stratification, clarify role mandates (§4) |
| Too many review rounds (3+) | Scope too broad, requirements unclear | Apply decision matrix (§2.2.1), split tracks (§15.4), return to Phase A |
| Implementation doesn't match spec | Ambiguous spec, developer improvised | Spec Deviation Protocol (§14), strengthen "stop and ask" instruction |
| Consolidation misses issues | Summarizing not synthesizing, disagreements smoothed | Require table format, mandate Agreement Analysis, distinguish factual vs. priority disagreements |

---

## 19. Quick Reference

### 19.1 Phase Checklist

**Pre-Phase A** (if applicable): Triage (§12) or Proposal Review (§13) → approved scope → Phase A.

**Phase A:** A.1 Gather inputs → A.2 Scope → A.3 Research → A.4 Write spec v1.0 → A.5 Self-review → A.6 Submit for Phase B.

**Phase B:** B.1 Review Board (parallel) → B.2 Consolidation → B.3 CA Response → Spec v1.1 → B.4 Verification (if needed) → Phase C.

**Phase C:** C.0 CA investigation (if applicable, §10.3) → C.1 CA writes implementation prompt → C.2 Developer implements → C.3 PR created → Phase D.

**Phase D:** D.1 CA PR Review → D.2 Review Board (parallel) → D.3 Consolidation → D.4 Developer fixes → D.5 Adversarial verification → D.6 Final approval + retrospective → Merge.

**Prompt generation** (every phase transition): Confirm variables (§6.3), ask delivery preference (§7.7), generate as separate files (§7.6), check context budget (§7.8).

**Agent team workflow** (Tier 1/2): Assess risk → confirm tier (§15.5.1) → generate meta prompts (§15.5.4, §15.5.5) → hand off → checkpoints → merge.

### 19.2 Key Reference Sections

| Topic | Section |
|-------|---------|
| Prompt energy language | §7.4 |
| Severity levels | §16.3 |
| Round 2 decision matrix | §2.2.1 |
| Agent team tier selection | §15.5.1 |
| Meta prompt templates | §15.5.4, §15.5.5 |
| Context trimming priority | §7.8.5 |

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
| 3.3 | 2026-02-09 | Added: Agent Team Workflow (15.5) with three-tier orchestration system. Defines Development Team (Codex, dynamic composition) and Review Team (Claude Code, fixed composition) meta prompts with explicit agent team activation directives. Tier system based on risk characteristics with escalation triggers. |
| 3.3.1 | 2026-02-23 | Fixed: Development Team meta prompt (15.5.4) now includes branch creation, commit strategy, test commands, and PR creation requirements. Added missing Task Context variables. |
| 3.4.0 | 2026-03-22 | Compaction pass: 2,157 → 1,818 lines. Consolidated anti-pattern tables, pattern examples, context window tables, troubleshooting section, quick reference checklists. Added: C.0 CA Investigation step (10.3) with investigation question patterns. Multi-PR Sequence Ledger (6.4). Evidence-Based Patch mode (15.2). Validation/Observation Run protocol (15.3). Exclusion List as mandatory section in specs (8.3) and implementation prompts (10.4). Custom attack vectors principle (9.3, 11.4). Tier 2 checkpoint checklist and proposal review integration (15.5.6). Session logs artifact type (2.6). Test count tracking (6.3). Agent team workflow adjustments: PR number placeholders, branch naming (6.3). File naming with task identifier (16.4). |
| 3.5.0 | 2026-03-28 | Added mandatory diagramming gate at Phase A exit (8.3.1). Specs must include at least two text-based diagrams (architecture/component + state machine/sequence) with quality criteria. Diagrams are reviewable artifacts: all three reviewer roles cross-reference diagrams against prose in Phase B (9.3) and Phase D (11.4). Diagram-prose mismatches are blocking by default. |
| 3.6.0 | 2026-04-04 | Added Product-lens reviewer as mandatory role in Proposal Review (§13.4). Formalizes §13 reviewer configuration: 2-reviewer board with Adversarial+Product and Alignment+Technical pairings. Product lens targets "wrong thing to build" findings before Phase A effort is spent. Added Review Verdict Artifact (§16.5): `review-verdict.md` saved after B.2 and D.3 consolidation for resumability and decision traceability. Updated §2.6 output table and §9.4 B.2 verdict table to include Lens column (defaults to Technical in spec/code reviews, populated from §13.4 in proposal reviews). |

---

*End of LLM Development Playbook v3.6.0*
