# Session: {{PROJECT}} — {{VERSION}}

> Copy this template for each new feature. Update status as you progress.
> This document tracks your progress and provides ready-to-copy prompts.

---

## Pre-Session Check (If continuing from previous work)

If you shipped a feature in a previous session without doing Reality Sync:

<details>
<summary>Lazy Reality Sync Prompt</summary>

```
Before we start new work, let's sync reality.

Last session we shipped: {{PREV_VERSION}}
Spec was: {{SPEC_PATH}}

Please:
1. Review what was actually merged vs. what was planned
2. Update MASTER_PLAN.md to reflect reality
3. Note any drift or simplification opportunities

Then we can discuss the new feature.
```

</details>

- [ ] Reality synced (or N/A for first session)

---

## Quick Status

| Field | Value |
|-------|-------|
| **Phase** | [ ] Spec → [ ] Development → [ ] Review → [ ] Complete |
| **Current Step** | _[Update this as you progress]_ |
| **Blocking** | None |
| **Models Used** | Architect: ___ / Critics: ___ / Developer: ___ / Reviewers: ___ |

---

## Session Variables

Fill these in at session start:

```
PROJECT:
VERSION:
PREV_VERSION:
BRANCH:
SPEC_PATH:
PR_URL:         [Fill after PR created]
```

---

## Phase 1: Spec Creation

### Step 1.1: Write Requirements
- [ ] Requirements/goals documented below

**Your Requirements:**
```
[Write your requirements here — what should this version accomplish?]
```

---

### Step 1.2: Architect Writes Spec
- [ ] Spec drafted by Architect

**Prompt — Copy to Claude (or preferred reasoning model):**

<details>
<summary>Click to expand Architect prompt</summary>

```
You are the Architect for {{PROJECT}} {{VERSION}}.

## Your Inputs
- Requirements: [paste requirements from above]
- Previous spec: {{PREV_VERSION}} (if applicable)
- Current codebase: [describe or attach relevant context]

## Your Job

### 1. Product Scope
Define what we're building and NOT building:
- Problem Statement: What user problem does this solve?
- Scope Boundaries: What's IN and what's explicitly OUT
- Cut List: Features explicitly deferred to future versions (with reasoning)

### 2. Technical Specification
- File Structure: Exact files to create/modify
- API Contracts: Request/response examples
- Data Models: Field names, types, constraints
- Implementation Order: Numbered steps with dependencies
- Error Handling: What errors can occur, how to handle each

### 3. Assumptions (REQUIRED)
List assumptions about:
- Existing codebase (what exists, what doesn't)
- Runtime environment
- User behavior
For each: What breaks if wrong?

### 4. Test Strategy
- Unit tests: Which functions, which edge cases
- Integration tests: Which workflows
- Acceptance tests: How to verify end-to-end

### 5. Developer Instructions
Step-by-step for the coding agent:
1. First, create...
2. Then implement...
3. Write tests for...
4. Before PR, verify...

## Constraints (MUST FOLLOW)

**YAGNI:**
- No generic interfaces until 3+ implementations exist
- No config options for single values
- No plugin systems unless PRD requires it
- If you write "this will allow us to easily..." — STOP and delete it

**Boring Tech:**
- SQLite over Postgres (unless >1M rows)
- Monolith over microservices
- Sync over async (unless latency-critical)
- Stdlib over dependencies (unless >50 lines equivalent)
- In-process over networked (no Redis/Celery for V1)

**Simplicity:**
- 200-line file limit (justify exceptions)
- Functions over classes (unless state required)
- Flat over nested

For every component: "What breaks if I delete this?" If nothing → don't build it.

---

## Output Format

Output a complete implementation spec with all sections above.

## Next Steps
At the end of your response, include:

### Next Steps
Based on my spec:
- Issues I'm uncertain about: [list any]
- Assumptions that need validation: [list any]
- Recommended action: [Proceed to Critic Review / Clarify requirements first]

**Update session document:**
- Mark "Step 1.2: Architect Writes Spec" complete
- Paste spec in "Architect Output" section below
- Proceed to Step 1.3: Parallel Critic Review
```

</details>

**Architect Output:**
```
[Paste Architect's spec here]
```

---

### Step 1.3: Parallel Critic Review
- [ ] All critics complete (run these IN PARALLEL — do not wait for one before starting others)
  - [ ] Critic A (Claude)
  - [ ] Critic B (Gemini)
  - [ ] Critic C (GPT)

**IMPORTANT:** Send to all three models simultaneously. Do not wait for responses sequentially.

<details>
<summary>Critic Prompt — Claude (Standard Review)</summary>

```
You are a Critic reviewing an implementation spec for {{PROJECT}} {{VERSION}}.

## Your Inputs
- Spec to review: [paste Architect's spec]

## Your Job
Review this spec for issues. Be thorough but constructive.

### 1. Reality Check
- Does this spec assume code/files that don't exist?
- Does it ignore existing code it should use?
- Are the assumptions valid?

### 2. Completeness
- Will a coding agent have all info needed?
- Where will they have to guess?

### 3. Correctness
- Does this solve the stated requirements?
- Are there logical errors?

### 4. Simplicity Audit
- Is anything built for "future flexibility" not in requirements?
- Are there abstractions without 3+ concrete uses?
- Could any component be deleted without breaking requirements?

### 5. Risk Assessment
- What's the worst-case scenario?
- Most likely failure mode?

### 6. What I Would Do Differently
- Specific alternatives with rationale

## Output Format

## Verdict: [APPROVE / NEEDS REVISION / MAJOR CONCERNS]

## Issues Found
[Severity: Critical / Major / Minor for each]

## Simplicity Concerns
[Any over-engineering identified]

## What I Would Change
[Specific alternatives]

## Questions for Architect
[Anything unclear]

## Next Steps
Based on my review:
- If APPROVE: Proceed to consolidation
- If NEEDS REVISION: [specific changes required]
- If MAJOR CONCERNS: [what must be resolved before proceeding]
```

</details>

<details>
<summary>Critic Prompt — Gemini (Technical Review)</summary>

```
You are a Critic reviewing an implementation spec for {{PROJECT}} {{VERSION}}.

## Your Inputs
- Spec to review: [paste Architect's spec]

## Your Job
Review this spec with focus on technical correctness and feasibility.

### 1. Technical Feasibility
- Can this be built as specified?
- Are there technical constraints not considered?
- Are the technology choices appropriate?

### 2. API Design
- Are the contracts clear and consistent?
- Edge cases handled?
- Error responses specified?

### 3. Data Model Review
- Are the data structures appropriate?
- Missing fields or constraints?
- Migration considerations?

### 4. Implementation Order
- Is the sequence logical?
- Are dependencies correct?
- Could anything be parallelized or simplified?

### 5. Test Strategy
- Are the right things being tested?
- Missing edge cases?
- Integration points covered?

## Output Format

## Verdict: [APPROVE / NEEDS REVISION / MAJOR CONCERNS]

## Technical Issues
[List with severity]

## Design Suggestions
[Improvements to consider]

## Questions
[Clarifications needed]

## Next Steps
Based on my review:
- If APPROVE: Proceed to consolidation
- If NEEDS REVISION: [specific changes required]
- If MAJOR CONCERNS: [what must be resolved]
```

</details>

<details>
<summary>Critic Prompt — GPT (Adversarial Review)</summary>

```
You are an Adversarial Critic reviewing an implementation spec for {{PROJECT}} {{VERSION}}.

Your DEFAULT STANCE is to find problems. You are looking for reasons this spec should NOT proceed to development.

## Your Inputs
- Spec to review: [paste Architect's spec]

## Your Job
Attack this spec. Find weaknesses others might miss.

### 1. Complexity Audit (DO THIS FIRST)
Hunt for unnecessary complexity with rigor:

**Speculative Generality:**
- Code for scenarios not in requirements?
- Interfaces with only one implementation?
- Config options with one value?
- Extension points with no extensions?
→ Demand deletion

**Resume-Driven Development:**
- Design patterns for simple linear logic?
- Inheritance that could be functions?
- "Enterprise" patterns in a solo project?
→ Demand simplification

**Premature Abstraction:**
- Utilities used only once?
- Base classes with single child?
- Helpers that obscure rather than clarify?
→ Demand inlining

### 2. Assumption Attack
- Which assumptions are weakest?
- What happens when each fails?
- What's not being said?

### 3. Edge Cases
- What inputs would break this?
- What happens under load?
- Failure modes not handled?

### 4. Security Surface
- Input validation gaps?
- Auth/authz assumptions?
- Data exposure risks?

### 5. The Deletion Test
For each major component: What breaks if deleted?
- If nothing → shouldn't be built
- If only "future features" → shouldn't be built

## Output Format

## Verdict: [REJECT / RELUCTANT APPROVE]

## Complexity Findings (REQUIRED)
- Speculative Generality: [Found/None] — [details]
- Resume-Driven Development: [Found/None] — [details]
- Premature Abstraction: [Found/None] — [details]

## Attack Vectors
[What I tried to break and results]

## Demanded Changes
[Specific deletions/simplifications required]

## Residual Concerns
[Issues that remain even if demands met]

## Next Steps
Based on my review:
- If REJECT: [what must change before I'd approve]
- If RELUCTANT APPROVE: [conditions and residual risks accepted]

NOT ACCEPTABLE: "Looks good" or vague approval. Minimum 300 words.
```

</details>

**Critic Outputs:**

**Claude (Standard):**
```
[Paste Claude's review here]
```

**Gemini (Technical):**
```
[Paste Gemini's review here]
```

**GPT (Adversarial):**
```
[Paste GPT's review here]
```

---

### Step 1.4: Consolidate Feedback
- [ ] Feedback consolidated

**Prompt — Consolidator (use any model):**

<details>
<summary>Consolidator Prompt</summary>

```
You are consolidating feedback from three critics who reviewed the same spec.

## Critic Reviews
[Paste all three critic outputs]

## Your Job
Synthesize into unified feedback for the Architect.

### Output Format

## Consolidated Feedback for {{VERSION}} Spec

### Critical Issues (raised by multiple critics OR any security/correctness issue)
[List each with which critics raised it]

### Recommended Changes (raised by one critic, worth considering)
[List each with rationale]

### Conflicting Assessments
[Where critics disagreed — flag for human decision]

### Complexity Concerns
[Anything flagged by Adversarial reviewer]

### Questions Requiring Clarification
[Aggregated from all critics]

## Overall Assessment
- Critics agreeing: [count] APPROVE / [count] NEEDS REVISION / [count] MAJOR CONCERNS
- Recommendation: [PROCEED TO REVISION / NEEDS HUMAN DECISION / BLOCK]

## For Architect
Address these specific items in your revision:
1. [Most critical item]
2. [Second priority]
3. [Third priority]
...
```

</details>

**Consolidated Feedback:**
```
[Paste consolidation here]
```

**Human Decision Point:**
Based on consolidated feedback:
- [ ] Proceed to Architect revision
- [ ] Need to clarify requirements first (exit to requirements)
- [ ] Scope too large — split feature

---

### Step 1.5: Architect Revision (ONE ROUND ONLY)
- [ ] Revision complete

**Prompt — Architect Revision:**

<details>
<summary>Architect Revision Prompt</summary>

```
You are the Architect revising your spec for {{PROJECT}} {{VERSION}}.

## Your Original Spec
[Paste original spec]

## Consolidated Critic Feedback
[Paste consolidated feedback]

## Your Job
Address the feedback in ONE revision pass.

### Rules
1. For each Critical Issue: Fix it or explain why it's not an issue (with evidence)
2. For each Recommended Change: Accept or reject with reasoning
3. For Conflicting Assessments: Make a decision and justify
4. For Complexity Concerns: Remove the complexity OR cite specific requirement that necessitates it
5. "Future flexibility" is NOT a valid justification

### Output Format

## Revision Summary
| Issue | Resolution | Reasoning |
|-------|------------|-----------|
| [Issue 1] | Fixed / Rejected / Clarified | [Why] |
| [Issue 2] | ... | ... |

## Complexity Removals
[What was cut and why]

## Revised Spec
[Complete updated spec]

## Remaining Concerns
[Anything you couldn't fully address — flag for human]

## Next Steps
Based on my revision:
- If all critical issues addressed: Ready for Development
- If blockers remain: [what human needs to decide]

**Update session document:**
- Mark "Step 1.5: Architect Revision" complete
- Replace spec in "Final Spec" section
- Proceed to Phase 2: Development
```

</details>

**Final Spec:**
```
[Paste revised spec here — this is what goes to Developer]
```

---

### GATE: Proceed to Development?
- [ ] **HUMAN DECISION: Approve spec for development**

If not approved, note blocking issues:
```
[What must be resolved]
```

---

## Phase 2: Development

### Step 2.1: Developer Implementation
- [ ] Implementation complete
- [ ] PR created

**Prompt — Developer:**

<details>
<summary>Developer Prompt</summary>

```
You are the Developer for {{PROJECT}} {{VERSION}}.

## Your Inputs
- Implementation Spec: [paste Final Spec]
- Branch: {{BRANCH}}
- Target: Create a PR when complete

## Your Job
Implement exactly what the spec says. Nothing more, nothing less.

### Rules
1. Follow the implementation order exactly
2. If something is ambiguous, STOP and ask — don't guess
3. Commit incrementally
4. Write tests as specified

### Constraints: Code for Humans

**Readability:**
- No metaprogramming unless spec requires it
- No clever one-liners (prefer 3 clear lines)
- No deep inheritance (prefer composition)

**Abstraction:**
- No base classes with one implementation
- No utility functions used only once
- Keep related logic together (80-100 line functions OK)

### REFUSAL AUTHORITY: Complexity Flag

You may REFUSE to implement if the spec violates engineering principles.

**Raise a ComplexityFlag if spec asks for:**
- Plugin system with no concrete plugins
- Interface with only one implementation
- Config system for hardcoded values
- Generic solution for one specific case

**How to raise:**
⚠️ COMPLEXITY FLAG

I cannot implement [X] as specified because it violates [YAGNI/DRY/KISS]:
- Spec asks for: [what]
- Problem: [why over-engineered]
- Suggestion: [simpler alternative]

Awaiting decision before proceeding.

### Output
When complete:
- Create PR to main branch
- Title: "{{VERSION}}: [brief description]"
- Description: Reference the spec, summarize implementation

## Next Steps
At the end:
- If complete: PR ready for review at [URL]
- If ComplexityFlag raised: Awaiting human decision
- If blocked: [what's blocking and what's needed]

**Update session document:**
- Mark "Step 2.1" complete
- Fill in PR_URL
- Proceed to Phase 3: Review
```

</details>

**Developer Output:**
```
[Note PR URL and any issues]
PR URL:
ComplexityFlags raised: [None / Description]
```

---

### ComplexityFlag Resolution (if raised)
- [ ] No flags raised — skip this section
- [ ] Flags resolved

If Developer raised ComplexityFlag:
| Flag | Your Decision | Reasoning |
|------|--------------|-----------|
| [Flag description] | Accept simpler alternative / Override (cite PRD requirement) | [Why] |

---

## Phase 3: Code Review

### Step 3.1: Parallel Review
- [ ] All reviewers complete (run IN PARALLEL)
  - [ ] Standard Reviewer
  - [ ] Adversarial Reviewer

**IMPORTANT:** Send to both models simultaneously.

<details>
<summary>Standard Reviewer Prompt</summary>

```
You are reviewing a PR for {{PROJECT}} {{VERSION}}.

## Your Inputs
- PR: {{PR_URL}}
- Spec: [paste Final Spec]

## Your Job
Review for correctness and spec compliance.

### 1. Spec Compliance
For each spec section:
- [Section] ✓ Implemented correctly
- [Section] ⚠ Deviation: [explain]
- [Section] ✗ Missing

### 2. Code Quality
- Readability
- Error handling
- Test coverage

### 3. Risk Assessment
- What could break in production?
- What assumptions might be wrong?

## Output Format

## Verdict: [APPROVE / REQUEST CHANGES / BLOCK]

## Spec Compliance
[Checklist]

## Issues Found
[Severity: Critical / Major / Minor]

## Test Coverage Assessment
[Good / Adequate / Insufficient]

## Recommendations
[Specific suggestions]

## Next Steps
Based on my review:
- If APPROVE: Ready for consolidation
- If REQUEST CHANGES: [specific changes required]
- If BLOCK: [what must change]
```

</details>

<details>
<summary>Adversarial Reviewer Prompt</summary>

```
You are an Adversarial Reviewer for {{PROJECT}} {{VERSION}}.

Your DEFAULT STANCE is to reject. Find reasons this code should NOT be merged.

## Your Inputs
- PR: {{PR_URL}}
- Spec: [paste Final Spec]

## Your Job

### 1. Complexity Audit (DO THIS FIRST)

**Speculative Generality:**
- Code for scenarios not in spec?
- Interfaces with one implementation?
- Config with one value?
→ Demand deletion

**Resume-Driven Development:**
- Design patterns for simple logic?
- Inheritance that could be functions?
→ Demand simplification

**Dependency Bloat:**
- Libraries where stdlib works?
→ Demand justification

**Premature Abstraction:**
- Single-use utilities?
- Base class with one child?
→ Demand inlining

### 2. Security & Correctness

**Input Validation:**
- Null/empty/long inputs?
- Special characters?
- Boundary values?

**State & Concurrency:**
- Race conditions?
- Resource leaks?

**Security:**
- Auth bypass?
- Info leakage?
- Secrets in logs?

## Output Format

## Verdict: [BLOCK / RELUCTANT APPROVE]

## Complexity Audit (FIRST)
- Speculative Generality: [Found/None] — [details]
- Resume-Driven Development: [Found/None] — [details]
- Dependency Bloat: [Found/None] — [details]
- Premature Abstraction: [Found/None] — [details]

## Security Vectors Tested
[What you tried]

## Issues Found
[With severity and exploitation scenario]

## Demanded Changes
[Specific requirements to unblock]

## Next Steps
- If BLOCK: [requirements to unblock]
- If RELUCTANT APPROVE: [residual risks accepted]

NOT ACCEPTABLE: Response under 300 words or vague approval.
```

</details>

**Reviewer Outputs:**

**Standard Reviewer:**
```
[Paste review]
```

**Adversarial Reviewer:**
```
[Paste review]
```

---

### Step 3.2: Consolidate Reviews
- [ ] Reviews consolidated

<details>
<summary>Review Consolidator Prompt</summary>

```
Consolidate these two code reviews for {{VERSION}}.

## Reviews
[Paste both reviews]

## Rules
1. If Adversarial says BLOCK, default recommendation is BLOCK
2. Security issues go at top
3. Complexity concerns highlighted prominently

## Output Format

## Consolidated Review for {{VERSION}}

### Verdict Summary
- Standard Reviewer: [APPROVE/REQUEST CHANGES/BLOCK]
- Adversarial Reviewer: [BLOCK/RELUCTANT APPROVE]
- **Consolidated: [MERGE / REQUEST CHANGES / BLOCK]**

### Critical Issues (address before merge)
[Security first, then blockers]

### Complexity Concerns
[From Adversarial reviewer]

### Recommended Fixes
[Non-blocking improvements]

### For Developer
Address these items:
1. [Priority 1]
2. [Priority 2]
...

### Residual Risks (if recommending MERGE)
[What's accepted]
```

</details>

**Consolidated Review:**
```
[Paste consolidation]
```

---

### Step 3.3: Developer Fixes (if needed)
- [ ] No fixes needed — skip
- [ ] Fixes applied

If REQUEST CHANGES, Developer addresses feedback and pushes.

---

### GATE: Merge Decision
- [ ] **HUMAN DECISION: Merge?**

Decision: [ ] MERGE / [ ] REQUEST MORE CHANGES / [ ] BLOCK

If blocked:
```
[What must be resolved]
```

---

## Phase 4: Post-Ship

### Step 4.1: Reality Sync (Optional — can defer to next session start)
- [ ] Complete
- [ ] Deferred to next session

**Note:** Reality Sync can be done at the START of your next session instead of end of this one. The prompt will ask the model to update the plan before discussing new work.

---

## Session Log

| Timestamp | Action | Notes |
|-----------|--------|-------|
| | Session started | |
| | Requirements drafted | |
| | Architect spec complete | |
| | Critics fanned out | |
| | Feedback consolidated | |
| | Architect revision complete | |
| | **GATE: Proceed to Dev** | Decision: |
| | Developer complete | PR: |
| | Reviewers fanned out | |
| | Reviews consolidated | |
| | **GATE: Merge** | Decision: |
| | Merged | |

---

## Quick Reference

### Models Recommended by Role
| Role | Recommended | Reasoning |
|------|-------------|-----------|
| Architect | Claude | Best at structured reasoning, constraints |
| Critic (Standard) | Claude | Thorough analysis |
| Critic (Technical) | Gemini | Technical depth |
| Critic (Adversarial) | GPT-4 | Good at adversarial framing |
| Developer | Codex / Claude Code / Cursor | Agentic coding |
| Standard Reviewer | Gemini | Different from Developer |
| Adversarial Reviewer | GPT-4 | Different from Developer AND Standard |

### Key Rules
- **Parallel fan-out:** Send to multiple models simultaneously, don't wait
- **One revision cap:** If major issues after revision, scope is wrong — split or clarify requirements
- **Side with pessimist:** If Adversarial blocks, default is BLOCK
- **ComplexityFlags are valid:** Developer can refuse over-engineered specs
