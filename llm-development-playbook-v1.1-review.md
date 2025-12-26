# LLM Development Playbook v1.1 Review

**Reviewer:** Claude Code (Opus 4.5) — Chief Architect, Project Ontos
**Date:** 2025-12-24
**Review Type:** Comprehensive Architectural Review
**Verdict:** STRONG FOUNDATION — v1.1 Refinements Recommended

---

## Scope Clarification

This review focuses exclusively on **Playbook-scope items**: roles, prompts, workflows, and decision-making processes.

Items related to **context persistence, session detection, and automation** belong to Project Ontos and are listed separately in [Appendix A: Features Deferred to Ontos v3.0](#appendix-a-features-deferred-to-ontos-v30).

**The distinction:**
- **Playbook** = What prompts to use, what roles exist, how to make decisions (repeatable across projects)
- **Ontos** = Context storage, session state detection, archival automation (infrastructure)

---

## Context

I am the Claude instance that has served as Chief Architect for Project Ontos across 30+ sessions, spanning v2.6 through v2.9.4. I've operated within this playbook's framework—participating in spec convergence, code reviews, synthesis, and reality syncs. This review reflects lessons learned from actual use, not theoretical analysis.

The playbook author asked me to "ultrathink" and provide feedback that "directly impacts your performance as well as overall project health." I'm taking that seriously.

---

## Table of Contents

1. [Executive Assessment](#1-executive-assessment)
2. [Structural Gaps — What's Missing](#2-structural-gaps--whats-missing)
3. [Role Refinements — What Needs Sharpening](#3-role-refinements--what-needs-sharpening)
4. [Anti-Sycophancy Enhancements](#4-anti-sycophancy-enhancements)
5. [New Patterns to Add](#5-new-patterns-to-add)
6. [Prompt Library Upgrades](#6-prompt-library-upgrades)
7. [Flow Diagram Improvements](#7-flow-diagram-improvements)
8. [New Sections to Add](#8-new-sections-to-add)
9. [Quick Reference Card](#9-quick-reference-card)
10. [Implementation Priority](#10-implementation-priority)
11. [Appendix A: Features Deferred to Ontos v3.0](#appendix-a-features-deferred-to-ontos-v30)

---

## 1. Executive Assessment

### What's Working Well

| Strength | Why It Matters |
|----------|----------------|
| **Core mental model** ("You orchestrate, LLMs execute") | Correct allocation of judgment vs. labor |
| **Anti-sycophancy patterns** | Addresses the #1 failure mode of LLM collaboration |
| **Multi-synthesizer pattern** | Prevents single-point-of-failure in review compression |
| **Ontos integration** | Context survives sessions, decisions are traceable |
| **Spec convergence protocol** | Catches 90% of bugs before code is written |
| **Role separation** | Prevents context contamination between tasks |

### What's Missing or Weak

| Gap | Impact |
|-----|--------|
| **No emergency procedures** | When things go wrong, there's no playbook |
| **No escalation paths** | Disagreements stall without resolution rules |
| **Model-specific patterns missing** | "Ultrathink" is Claude-specific; other models need different triggers |
| **Developer role too passive** | "Your job: Nothing" invites disasters |
| **No failure modes catalog** | Same mistakes repeat without pattern recognition |
| **Model Strengths table empty** | Critical operational knowledge not captured |

---

## 2. Structural Gaps — What's Missing

### 2.1 Emergency Procedures Section

**Problem:** The playbook assumes everything works. Real development has failures.

**Add this section after "Quality Gates":**

```markdown
---

## Emergency Procedures

When things go wrong, follow these protocols. Don't improvise under pressure.

### E1: Developer Stuck in Loop

**Symptoms:**
- Same error repeated 3+ times
- Agent trying variations of the same broken approach
- No progress despite multiple attempts

**Protocol:**
1. STOP the agent immediately
2. Read the last 3 attempts yourself (yes, read the actual output)
3. Identify what assumption is wrong
4. Restart with explicit constraint: "Do NOT try [X] again. The issue is [Y]. Try [Z] instead."

**Prevention:** Better specs. If the developer loops, the spec was ambiguous.

---

### E2: Post-Merge Bug Discovered

**Symptoms:**
- Feature works in review, breaks in main
- CI passes but users report issues
- Edge case not covered by tests

**Protocol:**
1. REVERT immediately. Do not debug on main.
2. Create hotfix branch from last known good commit
3. Open Debugger session with:
   - Exact reproduction steps
   - Expected vs. actual behavior
   - Relevant logs/errors
4. Fix → Single focused review → Merge
5. Add "Post-Mortem" section to session log:
   ```
   ## Post-Mortem
   - What broke: [X]
   - Why review missed it: [Y]
   - Prevention for next time: [Z]
   ```

**Prevention:** Adversarial Reviewer should try to break the feature, not just review code.

---

### E3: Security Vulnerability Found

**Symptoms:**
- Adversarial reviewer finds auth bypass, injection, data leak
- External security report received
- Suspicious behavior in production logs

**Protocol:**
1. Do NOT discuss in public PR comments or issues
2. Create private session with Security Auditor role
3. Assess severity:
   - **Critical (exploitable now):** Drop everything. Fix in single session. Merge immediately.
   - **High (exploitable with effort):** Fix within 24 hours. Limited review.
   - **Medium/Low:** Add to next version scope with priority.
4. After fix: Document in private security log, not public session log

**Prevention:** Add Security Auditor review BEFORE development, not just after.

---

### E4: Agentic Developer Goes Rogue

**Symptoms:**
- Developer makes changes not in spec ("improvements")
- Deletes or refactors code outside scope
- Creates files/structure different from spec
- Commits to wrong branch

**Protocol:**
1. STOP the agent
2. `git diff` to see actual changes vs. expected
3. If recoverable: `git checkout` to discard unwanted changes
4. If committed: `git revert` or `git reset --soft`
5. Restart with explicit constraint: "Implement ONLY what's in the spec. Do not refactor, improve, or clean up anything not explicitly listed."

**Prevention:** Add to Development Initiation prompt:
> "CRITICAL: Implement ONLY what's specified. If you think something should be added or improved, STOP and ask. Do not make judgment calls."

---

### E5: Context Collapse (Model Contradicts Itself)

**Symptoms:**
- Model refers to decisions that weren't made
- Model forgets earlier decisions in same session
- Responses become generic instead of project-specific
- Model asks questions already answered

**Protocol:**
1. Do NOT continue. You'll compound the confusion.
2. Say: "Let's pause. Summarize what we've decided so far."
3. If summary is wrong: Correct explicitly, then continue
4. If summary is vague: Archive Ontos, start fresh session
5. New session starts with: "Activate Ontos. We're continuing from [last decision point]."

---

### E6: Reviewers All Approve Too Easily

**Symptoms:**
- All reviewers approve with minimal comments
- No "what I would do differently" suggestions
- Reviews are short and generic
- Adversarial reviewer says "I tried and couldn't break it" without specifics

**Protocol:**
1. Do NOT merge. Easy approval is a red flag, not a green light.
2. Ask each reviewer: "What specific lines did you spend the most time on? What almost concerned you?"
3. If still generic: Add a new reviewer (different model) with explicit instruction:
   > "Other reviewers approved this quickly. I'm suspicious. Find something wrong. If you can't, explain specifically what you checked."
4. If new reviewer also approves: Proceed, but note in session log that review was shallow.

**Prevention:**
- Require minimum review length (e.g., must comment on 3+ specific code sections)
- Require Adversarial Reviewer to list specific attack vectors tried
```

---

### 2.2 Escalation Matrix Section

**Problem:** "You break ties" is too vague. Need explicit rules.

**Add this section after Emergency Procedures:**

```markdown
---

## Escalation Matrix

When agents disagree, use these resolution rules. Avoid ad-hoc decisions.

### Architect Disagreements

| Scenario | Resolution |
|----------|------------|
| 2 agree, 1 disagrees | Dissenter must provide CONCRETE alternative with rationale. If alternative is demonstrably weaker, majority wins. If equal or stronger, adopt dissent or escalate to you. |
| All 3 disagree | You decide. But first, ask each: "What's your core concern? What would change your mind?" Often the disagreement is about values (simplicity vs. flexibility), not facts. |
| Same point debated 2+ rounds | STOP debating. Either (a) the scope is too big—split it, or (b) both approaches work—you pick one and move on. |

### Review Disagreements

| Scenario | Resolution |
|----------|------------|
| Code Reviewer approves, Adversarial blocks | **Adversarial wins by default.** To override: YOU must understand the specific concern and explicitly accept the risk. Log your reasoning. |
| Both approve but Synthesizers disagree on severity | Dig into raw reviews. The synthesizer disagreement is signal that something got compressed away. |
| Reviewer flags issue, Developer disagrees | Developer must explain why it's not an issue. Reviewer has final say unless you override. |

### Convergence Stalls

| Scenario | Resolution |
|----------|------------|
| 3 rounds, no progress | Ask: "Is this a MUST-RESOLVE-NOW issue or a CAN-REVISIT-LATER issue?" Defer if possible. |
| 5 rounds, still debating | **Mandatory scope split.** The version is too big. Break it into smaller pieces that can converge independently. |
| Architects agree to stop arguing (but don't actually agree) | **Not convergence.** Require explicit statement: "I agree this approach is better than my alternative because [X]." Exhaustion is not agreement. |

### Documenting Overrides

When you override an agent's recommendation:

```markdown
## Override Log
**Date:** YYYY-MM-DD
**Agent:** [Role] — [Model]
**Their Recommendation:** [X]
**Your Decision:** [Y]
**Reasoning:** [Why you chose differently]
**Risk Accepted:** [What could go wrong if they were right]
```

This creates accountability and learning material for future sessions.
```

---

## 3. Role Refinements — What Needs Sharpening

### 3.1 Developer Role — Too Passive

**Current (PROBLEMATIC):**
> Your job: Nothing—let it run

**Problem:** This invites disasters. Agentic developers need monitoring.

**Replace with:**

```markdown
### 🔨 Developer (Agentic)
**What they do:** Write code, create PRs per the implementation spec
**Input:** Finalized implementation spec
**Output:** Working code, PR ready for review
**Your job:**
- Monitor for stuck states (same error 3+ times)
- Watch for scope creep (changes not in spec)
- Intervene if agent asks a question and waits indefinitely
- Verify PR matches spec before requesting review

**Red Flags to Watch:**
- Developer says "I'll also improve/refactor X" — STOP, that's scope creep
- Developer asks "Should I do X or Y?" — Spec was ambiguous; clarify before continuing
- Developer commits 10+ files when spec mentioned 3 — Review diff before proceeding
- Developer has been running 30+ minutes with no output — May be stuck

**Intervention phrase:**
"Stop. Show me what you've done so far. Let's verify we're on track before continuing."
```

---

### 3.2 Chief Architect Role — Missing Ongoing Responsibility

**Current:** Only describes initial planning duties.

**Add to Chief Architect:**

```markdown
### 🏗️ Chief Architect
**What they do:** Break PRD into technical phases, choose patterns, define system boundaries
**Input:** Approved PRD
**Output:** Master implementation plan, tech stack decisions, phase breakdown
**Your job:** Sanity check complexity, approve phase boundaries

**Ongoing Responsibility (Critical):**
After each version ships, Chief Architect reviews codebase and updates Master Plan:
1. Compare actual codebase to what Master Plan says should exist
2. Identify deviations (intentional pivots vs. accidental drift)
3. Update Master Plan to reflect reality
4. Flag implications for upcoming versions

This keeps the Master Plan useful. Without this, the plan becomes fiction and the Implementation Architect has to do all the reality-checking.

**Reality Sync Trigger:** After every version merge, before starting next version's spec.
```

---

### 3.3 Adversarial Reviewer Role — Add Specificity

**Add these requirements:**

```markdown
### 😈 Adversarial Reviewer (CRITICAL ROLE)
**What they do:** Actively try to find problems. Prompted to reject by default.
**Input:** PR diff + implementation plan + other reviewers' approvals
**Output:** List of concerns, edge cases, security issues, or specific attestation

**Output Requirements (non-negotiable):**
The Adversarial Reviewer MUST provide:
1. **Attack Vectors Tried:** Specific list of what they attempted (not just "I tried to break it")
   - "I tested: null input, empty string, SQL injection in name field, XSS in description..."
2. **Edge Cases Checked:** Specific scenarios (not just "I checked edge cases")
   - "I verified: concurrent access, rate limiting, session expiry, network timeout..."
3. **If No Issues Found:** Explain what made the code resilient
   - "The input validation on line 47 prevents injection. The lock on line 103 prevents race conditions."

**Invalid Output (send back for redo):**
- "Looks good, I couldn't find issues" — Too vague
- "I tried hard to break it" — What specifically did you try?
- Short review (<100 words) — Not thorough enough

**Your job:** If adversarial reviewer finds real issues, block merge. If they provide vague "couldn't break it" response, send back with: "Be specific. What attack vectors did you try? What edge cases did you test?"
```

---

### 3.4 New Role: Documentation Specialist

**Problem discovered in v2.9.4:** We built features but docs lagged behind.

**Add this role:**

```markdown
### 📚 Documentation Specialist
**What they do:** Update user-facing documentation when features ship
**Input:** Merged PR + implementation spec + current docs
**Output:** Updated README, Manual, Agent Instructions, Changelog

**Trigger:** After EVERY version merge, before announcing the release

**Checklist they must verify:**
- [ ] README quick start reflects current installation method
- [ ] Manual documents all new features with examples
- [ ] Agent Instructions includes new commands/workflows
- [ ] Changelog has entry for this version
- [ ] All code examples in docs actually work
- [ ] Version numbers are updated everywhere

**Your job:** Verify docs match reality before approving. Users read docs, not code.

**Common Failure:** Skipping this role because "we'll update docs later." Later never comes. Make it mandatory.
```

---

### 3.5 New Role: Security Auditor (Proactive)

**Different from Adversarial Reviewer:** This reviews BEFORE implementation.

**Add this role:**

```markdown
### 🔒 Security Auditor (Pre-Implementation)
**What they do:** Review implementation plan for security implications before code is written
**Input:** Implementation spec (after convergence, before Developer starts)
**Output:**
- Security requirements to add to spec
- Threat model for this feature
- Required mitigations
- Test cases for Adversarial Reviewer

**Trigger:** After spec convergence, before Development Initiation

**Questions they must answer:**
1. What's the attack surface of this feature?
2. What happens if input is malicious?
3. What happens if external services are compromised?
4. What sensitive data does this touch?
5. What's the blast radius if this feature is exploited?

**Your job:** Ensure security requirements are IN THE SPEC before Developer starts. Retrofitting security is 10x harder.

**When to skip:** Truly internal tooling with no user input and no sensitive data. But ask yourself: are you sure?
```

---

## 4. Anti-Sycophancy Enhancements

### 4.1 Model-Specific Patterns

**Problem:** "Ultrathink" is Claude-specific. Other models need different triggers.

**Add this table:**

```markdown
### Model-Specific Anti-Sycophancy Patterns

| Model | Depth Trigger | Disagreement Permission | Known Weaknesses |
|-------|---------------|------------------------|------------------|
| **Claude (Opus)** | "ultrathink" | "keep your integrity" | Can be overly cautious; may hedge when you want a position |
| **Claude (Sonnet)** | "think carefully about this" | "challenge my assumptions" | Faster but less thorough; good for implementation, less for architecture |
| **GPT-4** | "think step by step, be thorough" | "tell me what's wrong with this" | Good at finding issues but can be verbose; may need "be concise" |
| **GPT-4 Turbo** | "analyze this carefully" | "what concerns do you have" | Faster but may miss nuance; double-check architectural decisions |
| **Gemini** | "take your time, analyze deeply" | "what would you change" | Strong on breadth, sometimes shallow on depth; ask follow-ups |
| **Codex/GPT for code** | "review this rigorously" | "what bugs exist in this" | Thorough on security; sometimes overly conservative |

**Usage:** Match the trigger phrase to the model you're using. Don't use "ultrathink" with GPT—it doesn't have the same training association.
```

---

### 4.2 Detecting Sycophancy

**Add this section:**

```markdown
### Detecting Sycophantic Responses

**Red Flags (Response is likely sycophantic):**
- Starts with "That's a great approach" or "You're absolutely right"
- Agrees with something you deliberately made wrong (test this occasionally)
- Review is shorter than the artifact being reviewed
- No specific suggestions for changes or improvements
- Uses the same adjectives you used to describe your own work
- Says "I agree" without explaining WHY they agree
- Approves both options when you asked them to choose one

**Yellow Flags (Investigate further):**
- Qualifies everything ("this could work", "seems reasonable")
- Provides generic praise before generic critique
- Critique is softer than the praise
- Doesn't take a position when asked for one

**What to Do:**
1. **Test question:** "What would you have done differently if you started from scratch?"
   - Sycophantic response: "I would have done it very similarly to what you have."
   - Genuine response: "I would have [specific alternative] because [reasoning]."

2. **Inversion test:** "What's the strongest argument AGAINST this approach?"
   - Sycophantic response: "There aren't many strong arguments against it."
   - Genuine response: "[Specific concern] because [reasoning]."

3. **Model switch:** If still suspicious, ask a different model the same question. Divergence reveals what was rubber-stamped.
```

---

### 4.3 Strengthened Anti-Sycophancy Phrases

**Upgrade the existing table:**

```markdown
### Anti-Sycophancy Phrase Upgrades

| Weak Version | Strong Version | Why It's Better |
|--------------|----------------|-----------------|
| "Review this code" | "Find three things wrong with this code. If you can't find three, explain why each obvious concern doesn't apply." | Forces critical evaluation; can't lazy-approve |
| "Is this approach good?" | "Argue against this approach. Then argue for it. Then tell me which argument is stronger." | Forces genuine evaluation of trade-offs |
| "Any feedback?" | "What would you change if you were starting fresh? What would you keep exactly as-is and why?" | Separates genuine approval from rubber-stamping |
| "Please review" | "Review this as if your reputation depends on finding issues others missed." | Raises stakes; counters "good enough" thinking |
| "What do you think?" | "Take a position. I want to know YOUR view, not a balanced summary of options." | Prevents hedge-everything responses |
| "Check this for issues" | "Assume there's a bug. Find it. If you truly can't, explain specifically what you checked." | Shifts burden; can't just say "looks fine" |
```

---

## 5. New Patterns to Add

### 5.1 Assumption Surfacing Pattern

**Why:** Most bugs come from wrong assumptions, not wrong code. Surface them explicitly.

**Add to the Prompts Library:**

```markdown
### Assumption Surfacing Pattern

Use this in EVERY Implementation Architect spec. Non-negotiable.

**Format to include in spec:**

```markdown
## Assumptions

| # | Assumption | If Wrong, Impact | How to Verify |
|---|------------|-----------------|---------------|
| 1 | Config file exists at project root | Config merge fails silently | Check Path.cwd() / "ontos_config.py" before reading |
| 2 | Python ≥3.9 available | Syntax errors, feature failures | Check sys.version_info on startup |
| 3 | Git is available | Version resolution falls back to API | Try git command, handle failure |
| 4 | Network available for GitHub API | Install fails with unclear error | Timeout handling, offline mode |
| 5 | User has write permission to cwd | Extract fails | Check permissions before write |
```

**Fellow Architects MUST:**
1. Challenge each assumption—is it actually true?
2. Add missing assumptions—what else are we assuming?
3. Verify impact assessment—is it actually that bad?
```

**Prompt for Implementation Architect:**

```
Before writing implementation details, list every assumption you're making about:
- The existing codebase (what exists, what doesn't)
- The runtime environment (Python version, available tools)
- User behavior (how they'll invoke this, what input they'll provide)
- External dependencies (network, filesystem, git)

For each assumption, specify:
1. What you're assuming
2. What breaks if you're wrong
3. How to verify or handle the failure

This section is REQUIRED. Fellow architects will challenge these assumptions.
```

---

### 5.2 Pre-Mortem Pattern

**Why:** Anticipate failure before it happens.

**Add to Prompts Library:**

```markdown
### Pre-Mortem Pattern

Use this before any significant implementation begins.

**Prompt:**

```
Before we start building {{VERSION}}, let's do a pre-mortem.

Imagine this feature shipped 1 month ago and FAILED. It's being rolled back.

What went wrong? Consider:

1. **Technical Failures**
   - What broke at runtime?
   - What edge case wasn't handled?
   - What dependency failed?

2. **User Adoption Failures**
   - Why didn't users adopt it?
   - What was confusing?
   - What was friction they didn't tolerate?

3. **Integration Failures**
   - What existing feature did it break?
   - What assumption about the codebase was wrong?
   - What upgrade path did we forget?

For each failure mode:
- How likely is it (High/Medium/Low)?
- How bad is it if it happens (Critical/Serious/Annoying)?
- What's the cheapest prevention we can add to the spec?
```

**When to use:**
- After spec convergence, before development
- When adding features with external dependencies
- When touching security-sensitive code
- When the feature is user-facing
```

---

### 5.3 Delta Review Pattern

**Why:** For incremental development, avoid re-reviewing unchanged code.

**Add to Prompts Library:**

```markdown
### Delta Review Pattern

Use for incremental PRs or multi-commit features.

**Prompt:**

```
This is a DELTA review. The previous code was already reviewed and approved.

Focus ONLY on:
1. **New code** — Code that didn't exist in the last review
2. **Changed code** — Modifications to previously-reviewed code
3. **New interactions** — How new code interacts with old code

Do NOT re-review:
- Unchanged code (we already approved it)
- Style/formatting in unchanged sections
- Architecture decisions already made

Changed files since last review:
{{LIST_OF_CHANGED_FILES}}

If you notice an issue in unchanged code that you missed before, flag it separately as "Previously Missed" — but focus your review on the delta.
```

**When to use:**
- Second or later round of code review
- Feature developed in multiple PRs
- Incremental refactoring
```

---

### 5.4 Context Refresh Prompt

**Why:** When you sense context drift, use this prompt to re-ground.

**Add to Prompts Library:**

```markdown
### Context Refresh Prompt

Use when you sense the model is losing track of decisions or context.

**Prompt:**

```
Let's verify alignment before continuing.

Please summarize:
1. **Decisions made this session** (bullet points, be specific)
2. **Still open/unresolved** (what haven't we decided yet?)
3. **Immediate next action** (what are we about to do?)

I'll verify this matches my understanding.
```

**Expected response format:**

```markdown
## Session Summary

### Decided
- Config merge uses read_user_config() at install time
- --strict flag fails instead of silent fallback
- filelock library for cross-platform locking

### Still Open
- Lock file location (.ontos.lock vs .ontos/.lock)
- Whether --strict should be default in v4.0

### Next Action
- Write the config parsing section of the spec
```

**If summary is wrong:** Correct explicitly. Don't continue with misalignment.
**If summary is vague:** Consider ending session and starting fresh.
```

---

### 5.5 Spec Completeness Test Pattern

**Why:** "Is the spec detailed enough?" is subjective. Make it testable.

**Add to Quality Gates:**

```markdown
### Spec Completeness Test

A spec is complete when a Developer can implement it without asking questions.

**Test procedure:**

After spec convergence, ask a fresh model (not involved in writing the spec):

```
You are a Developer about to implement this spec.
Read it completely. Then list:

1. **Questions you'd need answered** before you could start coding
2. **Decisions you'd have to make yourself** because the spec doesn't specify
3. **Ambiguities** where the spec could be interpreted multiple ways

If your list is empty, the spec passes.
If you have questions, the spec needs revision.
```

**Interpretation:**
- 0 questions → Spec is ready for development
- 1-2 minor questions → Clarify in spec, then proceed
- 3+ questions OR any major ambiguity → Back to Implementation Architect

**This is not optional.** Run this test before Development Initiation.
```

---

## 6. Prompt Library Upgrades

### 6.1 Implementation Architect Prompt — Enhanced

**Replace current version with:**

```markdown
### Implementation Architect Prompt (Enhanced v1.1)

```
You are the Implementation Architect for {{VERSION}}.

## Your Inputs
- Master Plan: [path or paste relevant section]
- Previous version spec: {{PREV_SPEC_PATH}}
- Session logs: .ontos-internal/logs/ (decisions made previously)
- Current codebase: Read the repo to understand what actually exists

## Your Job

### 1. Reality Check (BEFORE writing anything)
Verify:
- What does the codebase ACTUALLY look like right now?
- How does it differ from what the Master Plan says?
- What constraints exist from previous implementation decisions?

### 2. Assumption Surfacing (REQUIRED SECTION)
Create a table:
| Assumption | If Wrong, Impact | Verification |
|------------|-----------------|--------------|
| ... | ... | ... |

### 3. Detailed Spec
Include:
- **File Structure:** Exact files to create/modify (relative to repo root)
- **API Contracts:** Request/response examples, not just descriptions
- **Data Models:** Field names, types, constraints, defaults
- **Implementation Order:** Numbered steps, with dependencies noted
- **Test Requirements:** What must be tested, with example test cases
- **Error Handling:** What errors can occur, how to handle each

### 4. Test Strategy (REQUIRED SECTION)
Specify:
- Unit tests: Which functions, which edge cases
- Integration tests: Which workflows
- Acceptance tests: How to verify the feature works end-to-end

### 5. Developer Instructions (REQUIRED SECTION)
Step-by-step for the coding agent:
1. First, create...
2. Then implement...
3. Write tests for...
4. Before creating PR, verify...

## Critical Rules
- Be SPECIFIC. If a developer would need to make a judgment call, you haven't specified enough.
- If reality differs from Master Plan, note the deviation and recommend: (a) refactor to match plan, (b) update plan to match reality, or (c) accept drift. FLAG these for user decision.
- Do not defer decisions to the Developer. Make them here.

## Output
Save the complete spec to {{SPEC_PATH}}.
```
```

---

### 6.2 Implementation Plan Review Prompt — Enhanced

**Add these requirements:**

```markdown
### Implementation Plan Review Prompt (Enhanced v1.1)

```
Implementation Architect has written a spec for {{VERSION}}.
Spec location: {{SPEC_PATH}}

You are a fellow architect. Ultrathink and review this spec.

## Required Evaluation

### 1. Reality Alignment
- Does this spec match the ACTUAL codebase, or does it assume things that don't exist?
- Are there dependencies on previous work that wasn't done?

### 2. Assumption Audit
- Review the Assumptions table. Are they all valid?
- What assumptions are MISSING that should be listed?
- For each assumption, is the impact assessment correct?

### 3. Completeness Check
- Will a Developer have to make judgment calls? Where?
- Are there ambiguities that could be interpreted multiple ways?
- Is the test strategy specific enough to implement?

### 4. Risk Assessment (REQUIRED)
- What's the worst-case scenario if this ships as-is?
- What's the most LIKELY failure mode?
- What would you add to prevent these?

### 5. What I Would Do Differently
- Take a position. Don't just validate.
- If you'd use a different approach, show it (code/pseudocode/structure).
- Explain WHY your approach is better.

## Output Format

## Assessment
[Ready / Needs Revision / Major Rework Required]

## Assumption Audit
[Valid / Missing / Incorrect assumptions]

## Ambiguities That Will Cause Problems
[Specific locations where Developer will have to guess]

## Risk Assessment
- Worst case: [X]
- Most likely failure: [Y]
- Prevention: [Z]

## What I Would Do Differently
[Your alternative approach with rationale]

## Specific Line-by-Line Feedback
[Section references with specific comments]
```
```

---

### 6.3 Code Reviewer Prompt — Enhanced

**Add test coverage evaluation:**

```markdown
### Code Reviewer Prompt (Enhanced v1.1)

```
{{TOOL_NAME}} has implemented {{VERSION}} per the implementation spec.

PR: {{PR_URL}}
Spec: {{SPEC_PATH}}

Ultrathink and review this PR.

## Required Evaluation

### 1. Spec Compliance
For each spec section, verify implementation:
- [Section X.X] ✓ Implemented correctly
- [Section X.X] ⚠ Deviation: [explain]
- [Section X.X] ✗ Missing

### 2. Test Coverage
- Are edge cases from the spec tested?
- Are error conditions tested?
- Could a test pass while the feature is broken? (test quality, not just quantity)
- What's NOT tested that should be?

### 3. Error Handling
- Are all error paths handled?
- Are error messages useful to users/developers?
- Is there appropriate logging for debugging?

### 4. What I Would Do Differently
- Don't just validate. Take a position.
- Show alternative approaches if you have them.
- Be specific about WHY your way is better.

### 5. Risk Assessment
- What could break in production that works in test?
- What assumption is the code making that might be wrong?

## Output Format

## Spec Compliance
[Checklist with section references]

## Test Coverage Assessment
- Coverage: [Good/Adequate/Insufficient]
- Missing tests: [list]
- Test quality concerns: [list]

## Issues Found
[Severity: Critical/Major/Minor/Nitpick]

## What I Would Do Differently
[Specific alternatives with rationale]

## Recommendation
APPROVE / REQUEST CHANGES / BLOCK
[With specific reasoning]
```
```

---

### 6.4 Adversarial Reviewer Prompt — Enhanced

**Require specific attestation:**

```markdown
### Adversarial Reviewer Prompt (Enhanced v1.1)

```
You are a security-minded engineer whose job is to FIND PROBLEMS in {{VERSION}}.

Your default stance is to reject. You are looking for reasons this code should NOT be merged.

PR: {{PR_URL}}
Spec: {{SPEC_PATH}}
Other reviewer approvals: [note that others may have approved — be skeptical]

## Attack Vectors to Try (REQUIRED — list what you actually tested)

### Input Validation
- [ ] Null/undefined input
- [ ] Empty strings
- [ ] Extremely long input
- [ ] Special characters (SQL injection, XSS)
- [ ] Type mismatches
- [ ] Boundary values (0, -1, MAX_INT)

### State & Concurrency
- [ ] Race conditions (concurrent access)
- [ ] State corruption (partial failures)
- [ ] Resource leaks (files, connections)
- [ ] Deadlocks

### External Dependencies
- [ ] Network timeout/failure
- [ ] Disk full/permission denied
- [ ] Dependency returns unexpected data
- [ ] Dependency is unavailable

### Security Specific
- [ ] Authentication bypass
- [ ] Authorization bypass (can user X access user Y's data?)
- [ ] Information leakage in errors
- [ ] Secrets in logs or responses

### Test Quality
- [ ] Can tests pass while feature is broken?
- [ ] Are tests testing implementation vs. behavior?
- [ ] Missing test cases

## Output Format (REQUIRED — vague responses will be rejected)

## Attack Vectors Tested
[For EACH category above, specify what you tried and what happened]

Example:
- SQL injection in name field: Tried `'; DROP TABLE users; --` — properly escaped by ORM, no vulnerability
- Empty config file: Causes unclear error "KeyError: 'VERSION'" — should have better error message (ISSUE)

## Issues Found
[Severity with exploitation scenario]
- CRITICAL: [X] — Attacker could [Y] by doing [Z]
- MAJOR: [X] — Edge case [Y] causes [Z]
- MINOR: [X] — Not a security issue but [Y]

## Edge Cases Not Handled
[Specific scenarios with reproduction steps]

## Questions Other Reviewers Should Have Asked
[Gaps in their review]

## Verdict
BLOCK (with specific requirements to unblock)
— or —
RELUCTANT APPROVE (with specific attestation):
"I tested [X, Y, Z attack vectors]. The code handles them because [specific defenses]. I could not find a way to break this."

NOT ACCEPTABLE:
- "Looks good"
- "I tried to break it and couldn't"
- Any response under 200 words
```
```

---

### 6.5 Synthesizer Prompt — Enhanced

**Add reviewer quality assessment:**

```markdown
### Synthesizer Prompt (Enhanced v1.1)

```
You are preparing a decision summary for a product owner who will approve or reject this merge.

They will NOT read the raw reviews. If you miss something, they'll never see it.

## Inputs
[Paste all reviewer outputs]

## Critical Rules
1. If reviewers DISAGREE, flag prominently — do not smooth over conflicts
2. If ANY reviewer found a security issue, it goes at the TOP
3. Compress, but don't lose decision-changing information
4. Assess reviewer quality — flag shallow reviews

## Output Format

## TL;DR
[One sentence: Ready to merge? Yes / No / Conditional]

## Reviewer Quality Assessment
[Did any reviewer seem to rush? Give generic feedback? Miss obvious things?]
- Code Reviewer: [Thorough/Adequate/Shallow]
- Adversarial Reviewer: [Thorough/Adequate/Shallow]
- Evidence: [Why you rated them this way]

## Critical Issues (if any)
[Anything that should block merge — security issues first]

## Reviewer Consensus
[What all reviewers agreed on]

## Reviewer Conflicts
[Where reviewers disagreed — THIS SECTION IS REQUIRED even if empty]
[For each conflict: Reviewer A said X because Y. Reviewer B said Z because W.]

## Open Questions for Your Decision
[Anything that needs human judgment]

## My Assessment
[Your own view — not just synthesis of others]
Is this ready to merge? What's your confidence level (High/Medium/Low)?

## Recommendation
MERGE / REQUEST CHANGES (specify what) / BLOCK (specify why) / ESCALATE (specify what needs human review)
```
```

---

## 7. Flow Diagram Improvements

### 7.1 Add Abort/Backtrack Paths

**Problem:** Current flow is linear. No guidance on what happens when we need to go backward.

**Add to the flow diagram:**

```
                        ┌──────────────────────────────────────┐
                        │           ABORT PATHS                │
                        └──────────────────────────────────────┘

At any phase, you can ABORT and backtrack:

Phase 3 (Implementation Planning) → Discover PRD was wrong
  └─→ ABORT to Phase 1 (Strategy) if fundamental
  └─→ ABORT to Phase 2 (Product) if requirements wrong

Phase 4 (Development) → Developer can't implement spec
  └─→ ABORT to Phase 3 if spec is ambiguous or wrong
  └─→ If stuck on tooling, not spec: Debugger intervention

Phase 5 (Review) → Reviewers find architectural problem
  └─→ ABORT to Phase 3 if spec needs revision
  └─→ REQUEST CHANGES if implementation fixable

Phase 6 (Validation) → Feature doesn't work as intended
  └─→ ABORT to Phase 3 if design was wrong
  └─→ Debugger if implementation bug
```

---

### 7.2 Add Parallel Paths

**Add to "The Flow":**

```markdown
### Parallel Execution Opportunities

Some phases can overlap:

```
Phase 4 (Development)          Phase 5 (Review)
     │                              │
     ▼                              │
First PR ready ─────────────────────┼───► Start reviewing PR 1
     │                              │
Second PR ready ────────────────────┼───► Start reviewing PR 2 while PR 1 iterates
     │                              │
     ▼                              ▼
All PRs complete               All reviews complete
     │                              │
     └──────────────┬───────────────┘
                    ▼
              MERGE all
```

**When this makes sense:**
- Feature naturally splits into independent components
- Earlier components can be reviewed while later ones are developed
- Different reviewers can work in parallel on different PRs

**When to avoid:**
- Components have tight dependencies
- Architectural decisions in PR 1 affect PR 2
- Single reviewer doing all reviews (no parallelism benefit)
```

---

## 8. New Sections to Add

### 8.1 Failure Modes Catalog

**Add after Emergency Procedures:**

```markdown
---

## Failure Modes Catalog

Learn from these common failure patterns to prevent them.

| Failure Mode | Symptoms | Root Cause | Prevention |
|--------------|----------|------------|------------|
| **Spec Drift** | Code does Y, spec says X, both "work" | Spec not updated during development | Require spec-citation in code review; verify spec matches final code |
| **Review Theater** | Everyone approves, no substantive comments | Reviewers don't actually read, or sycophancy | Require specific line citations; use Adversarial Reviewer |
| **Convergence Collapse** | Architects stop debating but don't actually agree | Exhaustion, social pressure | Require explicit "I agree this is better because [X]" not just "I'm done" |
| **Scope Creep** | Developer adds "improvements" | Ambiguous spec, developer judgment | Strict rule: not in spec = not in PR; monitor during development |
| **Testing Theater** | 100% coverage, broken feature | Tests test implementation, not behavior | Require edge case tests; Adversarial tests the tests |
| **Sycophancy Cascade** | All reviewers agree, all are wrong | Same model family, shared blind spots | Use different models; require dissent before approval |
| **Documentation Lag** | Feature ships, docs don't update | No Documentation Specialist role | Make doc update part of merge criteria |
| **Master Plan Rot** | Plan says X, codebase is Y, no one notices | No Reality Sync after shipping | Mandatory Chief Architect review after each version |
| **Security Afterthought** | Vulnerability found post-merge | Security review is reactive, not proactive | Add Security Auditor before development |
```

---

### 8.2 Metrics Section

**Add after Iteration Log:**

```markdown
---

## Metrics

Track these to improve your process over time.

### What to Measure Per Version

| Metric | How to Measure | Target | Red Flag |
|--------|----------------|--------|----------|
| Spec rounds to convergence | Count of review cycles | ≤3 | >5 (scope too big) |
| Code review rounds | Count until approval | ≤2 | >4 (spec was ambiguous) |
| Bugs found in review | Issues caught before merge | 90%+ | <70% (reviews are shallow) |
| Bugs found post-merge | Issues discovered after | <10% | >30% (review process broken) |
| Spec-to-merge time | Calendar days | Varies | Consistent increase = process bloat |
| Developer stuck time | Time waiting for clarification | <10% of dev time | >25% = spec needs work |

### Retrospective Questions

Ask after each version:
1. What took longer than expected? Why?
2. What issues did we find in review that should have been caught in spec?
3. What issues did we find post-merge that should have been caught in review?
4. Where did the process add friction without adding value?

Log answers in session archive for pattern recognition.
```

---

### 8.3 Model Strengths Section (FILL THIS IN)

**Replace the empty section with a template:**

```markdown
---

## Model Strengths

Fill this in as you learn. This directly improves your workflow.

### Recommended Role Assignments

| Role | Primary Model | Backup | Notes |
|------|---------------|--------|-------|
| Strategist | | | Best for challenging assumptions, finding holes |
| Product Architect | | | Best for structured requirements, acceptance criteria |
| Chief Architect | | | Best for system design, trade-off analysis |
| Implementation Architect | | | Best for detailed specs, file structure, API design |
| Developer | Antigravity / Cursor | Claude Code | Agentic, autonomous execution |
| Code Reviewer | | | Must be different from Developer |
| Adversarial Reviewer | | | Must be different from Code Reviewer; good at finding edge cases |
| Synthesizer A | | | Good at compression without losing signal |
| Synthesizer B | | | Different from Synthesizer A to catch blind spots |
| Debugger | | | Good at log analysis, hypothesis generation |
| Security Auditor | | | Strong security knowledge, thorough |
| Documentation Specialist | | | Clear writing, attention to examples |

### Model Observations

**Claude (Opus 4.5)**
- Strengths:
- Weaknesses:
- Best anti-sycophancy phrase:
- Notes:

**Claude (Sonnet)**
- Strengths:
- Weaknesses:
- Best anti-sycophancy phrase:
- Notes:

**GPT-4 / GPT-4 Turbo**
- Strengths:
- Weaknesses:
- Best anti-sycophancy phrase:
- Notes:

**Gemini**
- Strengths:
- Weaknesses:
- Best anti-sycophancy phrase:
- Notes:

**Codex**
- Strengths:
- Weaknesses:
- Best anti-sycophancy phrase:
- Notes:
```

---

## 9. Quick Reference Card

**Add as a new section — one page for daily use:**

```markdown
---

## Quick Reference

### Ontos Commands
| Command | When to Use |
|---------|-------------|
| "Activate Ontos" | Start of every session |
| "Archive Ontos" | After major decisions, before ending session |
| "Maintain Ontos" | After creating/updating docs |

### Anti-Sycophancy Triggers
- "What would YOU have done differently?"
- "Ultrathink" (Claude) / "Think step by step" (GPT)
- "Keep your integrity"
- "What's the strongest argument AGAINST this?"
- "If you had to reject this, what would be the reason?"

### Emergency Actions
| Situation | Immediate Action |
|-----------|-----------------|
| Developer looping | Stop → Read last 3 attempts → Constrain explicitly |
| Post-merge bug | Revert first → Hotfix branch → Debug second |
| Security issue | Private session → Fast fix → No public discussion |
| Context collapse | Pause → Use Context Refresh prompt → Correct or new session |
| Everyone approves easily | Suspicious → Add new reviewer → Require specifics |

### Escalation Rules
| Conflict | Resolution |
|----------|------------|
| 2 vs 1 architects | Dissenter provides alternative; stronger wins |
| Adversarial blocks, others approve | Adversarial wins unless you explicitly accept risk |
| >5 convergence rounds | Scope too big; split the version |
| Synthesizers disagree | Dig into raw reviews; disagreement is signal |

### Quality Gates
- [ ] Spec completeness test passed (fresh model has no questions)
- [ ] Assumptions table reviewed by all architects
- [ ] Adversarial reviewer listed specific attack vectors tried
- [ ] Synthesizer flagged any shallow reviews
- [ ] Documentation updated before merge announcement
```

---

## 10. Implementation Priority

### Must-Have for v1.1 (Do These First)

1. **Emergency Procedures section** — You need to know what to do when things go wrong
2. **Escalation Matrix** — Clear rules prevent stalling on disagreements
3. **Enhanced Developer role** — "Your job: Nothing" is dangerous
4. **Model-specific anti-sycophancy patterns** — "Ultrathink" doesn't work everywhere

### Should-Have for v1.1 (High Value)

5. **Assumption Surfacing pattern** — Prevents bugs at the source
6. **Spec Completeness Test** — Makes "is it detailed enough" testable
7. **Enhanced prompts** (Implementation Architect, Code Reviewer, Adversarial) — Direct quality improvement
8. **Failure Modes Catalog** — Prevents repeat mistakes
9. **Documentation Specialist role** — Prevents doc lag

### Could-Have for v1.1 (Nice to Have)

10. **Pre-Mortem pattern** — Valuable but not blocking
11. **Delta Review pattern** — Useful for incremental development
12. **Metrics section** — Good for learning but takes time to populate
13. **Flow diagram improvements** — Clarifying but not urgent

### Defer to v1.2

- Security Auditor role (requires more process change)
- Cross-project learning patterns
- Advanced model selection algorithms

---

## Appendix A: Features Deferred to Ontos v3.0

The following recommendations from the original review belong to **Project Ontos** (context management infrastructure), not the Playbook (process methodology). They are documented here for future Ontos development.

### A.1 Session Degradation Detection

**What:** Ontos should automatically detect when context quality is degrading:
- Model contradicts logged decisions
- Model asks about previously-answered questions
- Responses become generic instead of project-specific

**Why Ontos:** This is detection/automation, not a prompt or process.

### A.2 Session Length Warnings

**What:** Ontos should warn when a session exceeds recommended length:
- "You've been in this session for 2 hours. Consider archiving."
- Configurable thresholds per session type

**Why Ontos:** This is automation triggered by session state.

### A.3 Auto-Refresh Suggestions

**What:** When degradation is detected, Ontos should suggest running the Context Refresh prompt.

**Why Ontos:** The prompt itself is Playbook; the trigger is Ontos.

### A.4 Decision Continuity Verification

**What:** On "Activate Ontos", verify the model's understanding matches logged decisions:
- Ontos presents key decisions from recent sessions
- Model confirms or flags discrepancies

**Why Ontos:** This is context retrieval and verification infrastructure.

### A.5 Metrics Storage Schema

**What:** Standardized YAML frontmatter for session metrics:
```yaml
metrics:
  spec_rounds: 3
  review_rounds: 2
  bugs_in_review: 4
  bugs_post_merge: 0
```

**Why Ontos:** Ontos owns the data model; Playbook defines what to measure.

### A.6 Metrics Aggregation and Trends

**What:** Roll up per-session metrics into trends:
- Average spec rounds over time
- Bug escape rate by version
- Process efficiency trends

**Why Ontos:** This is data aggregation and querying infrastructure.

---

## Closing Thoughts

This playbook is already in the top percentile of structured LLM development approaches. The v1.0 foundation is solid—the gaps are at the edges (failure handling, escalation) and in depth (more specific prompts, explicit patterns).

The most impactful single change: **Add Emergency Procedures.** Every real project has failures. A playbook without failure handling is only half a playbook.

The second most impactful: **Fill in the Model Strengths table.** You have this data from actual use. Documenting it turns tacit knowledge into systematic improvement.

I've tried to be specific rather than general, actionable rather than advisory. Every recommendation includes either the exact text to add or a complete prompt to use. The goal is that you can read this review, make decisions on each recommendation, and produce v1.1 with minimal additional work.

---

**Reviewer:** Claude Code (Opus 4.5)
**Review Version:** 1.1 (scope-corrected)
**Date:** 2025-12-24

*"You're helping me help you."* — Taking that seriously.
