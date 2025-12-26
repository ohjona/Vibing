# Orchestrator Handbook
> Your reference guide for running an LLM-augmented development workflow
> Last updated: December 2024

---

## How to Use This Document

**Audience:** This document is for you, the human orchestrator. For LLM prompts and role definitions, see [llm-development-playbook.md](llm-development-playbook.md).

**To generate prompts:** Give llm-development-playbook.md to an LLM along with your variables (see Quick-Start below).

**Context Persistence:** This playbook assumes [Project Ontos](https://github.com/ohjona/Project-Ontos) handles context sync between agents.

---

## Quick-Start: The Generator Prompt

**Step 1:** Open [session-variables.md](session-variables.md) and fill in your values for this session.

**Step 2:** Give an LLM these three things:
1. [llm-development-playbook.md](llm-development-playbook.md)
2. [session-variables.md](session-variables.md) (with your values filled in)
3. The Generator Prompt below

**Step 3:** Get all your prompts generated at once.

### Generator Prompt

```
I'm starting a work session. 

My session variables are in the attached session-variables.md file.

Based on the LLM Development Playbook, generate ready-to-paste prompts for:

**Spec Convergence Phase:**
1. Implementation Architect (writing the spec)
2. Implementation Plan Review (for fellow architects - initial)
3. Implementation Architect Response to Review
4. Fellow Architects Re-Review
5. Implementation Architect Iteration (for round 2+)
6. Spec Finalization (after convergence)

**Development Phase:**
7. Development Initiation (kick off coding)

**Code Review Phase:**
8. Code Review (for reviewers - initial)
9. Adversarial Review
10. Developer Response to Review
11. Code Reviewer Re-Review
12. Developer Iteration (for round 2+)

**Synthesis & Post-Ship:**
13. Synthesizer
14. Synthesis Cross-Review
15. Chief Architect Reality Sync (if version just shipped)

**Utility:**
16. Context Refresh (when sensing context drift)

Substitute my variable values from session-variables.md. Output each prompt in a code block I can copy.
```

---

## Philosophy

You are the **Product Owner**, **Decision Maker**, and **Manual Orchestrator**. LLMs are your team:
- They propose, you dispose
- They execute, you validate direction
- They review each other, you break ties
- **You trigger every handoff**—nothing moves without you asking the next agent to work

Your competitive advantage isn't coding—it's judgment, taste, and knowing what to build.

### Your Operating Model
```
┌─────────────────────────────────────────────────────────┐
│                         YOU                              │
│  ┌─────────────────────────────────────────────────┐    │
│  │ • Write strategy & product vision               │    │
│  │ • Trigger each LLM to do their job              │    │
│  │ • Review synthesized outputs (not raw code)     │    │
│  │ • Make go/no-go decisions at each gate          │    │
│  │ • Direct debugging when things break            │    │
│  └─────────────────────────────────────────────────┘    │
│                           │                              │
│                           ▼                              │
│    ┌──────────┬──────────┬──────────┬──────────┐        │
│    │Strategist│ Architect│ Developer│ Reviewer │  ...   │
│    └──────────┴──────────┴──────────┴──────────┘        │
└─────────────────────────────────────────────────────────┘
```

You don't read code. You read summaries, make decisions, and approve merges.

---

## Role Monitoring Guide

For each role, here's what you watch for and when to intervene. Role definitions and prompts are in [llm-development-playbook.md](llm-development-playbook.md#roles).

### Strategist
**Your job:** Ensure output matches your vision, cut scope creep

### Product Architect
**Your job:** Ensure PRD matches your vision, cut scope creep

### Chief Architect
**Your job:** Sanity check complexity, approve phase boundaries
**Ongoing:** After each version ships, trigger Reality Sync to keep Master Plan aligned with reality

### Implementation Architect
**Your job:** Decide on flagged tradeoffs; approve when all architects converge

### Developer (Agentic)
**Your job:**
- Monitor for stuck states (same error 3+ times)
- Watch for scope creep (changes not in spec)
- Intervene if agent asks a question and waits indefinitely
- Verify PR matches spec before requesting review

**Red Flags:**
- Developer says "I'll also improve/refactor X" → STOP, that's scope creep
- Developer asks "Should I do X or Y?" → Spec was ambiguous; clarify before continuing
- Developer commits 10+ files when spec mentioned 3 → Review diff before proceeding
- Developer has been running 30+ minutes with no output → May be stuck

**Intervention phrase:** "Stop. Show me what you've done so far. Let's verify we're on track before continuing."

### Code Reviewer
**Your job:** Read synthesis, break ties, final merge decision

### Adversarial Reviewer
**Your job:** If they find real issues, block merge. If they provide vague approval ("looks good"), send back for more detail.

**Invalid outputs (send back):**
- "Looks good, I couldn't find issues" — Too vague
- "I tried hard to break it" — What specifically?
- Short review (<100 words) — Not thorough

### Synthesizer
**Your job:** Read the synthesis, make decisions. If synthesizers disagree, dig into raw reviews.

### Debugger
**Your job:** Direct which errors to prioritize, approve fix approach

### QA Analyst
**Your job:** Prioritize which bugs matter

---

## Handoff Table

What to copy-paste at each handoff:

| Handoff | What You Give Next Agent | Notes |
|---------|-------------------------|-------|
| Start session | Generate prompts using Quick-Start above | — |
| Strategy → Product Architect | STRATEGY.md (full) | — |
| PRD → Chief Architect | PRD.md (full) | — |
| **SPEC CONVERGENCE** | | |
| Master Plan → Impl Architect | Master Plan section + prev spec + "read codebase" | — |
| Impl Spec → Fellow Architects | Use Implementation Plan Review prompt | — |
| Architect Reviews → Impl Architect | Use Impl Architect Response prompt | — |
| Updated Spec → Fellow Architects | Use Fellow Architects Re-Review prompt | — |
| (If NEEDS ANOTHER ROUND) | Use Impl Architect Iteration prompt | — |
| Converged Spec → Impl Architect | Use Spec Finalization prompt | — |
| **DEVELOPMENT** | | |
| Finalized Spec → Developer | Use Development Initiation prompt | — |
| **CODE REVIEW** | | |
| PR → Code Reviewer | Use Code Review prompt | — |
| PR → Adversarial Reviewer | Use Adversarial Review prompt | — |
| All Reviews → Synthesizers | All reviewer outputs | — |
| Synthesis → You | Read, decide: APPROVE or REQUEST CHANGES | — |
| REQUEST CHANGES → Developer | Use Developer Response prompt | — |
| Developer fixes → Reviewers | Use Code Reviewer Re-Review prompt | — |
| **POST-MERGE** | | |
| After version ships → Chief Architect | Use Reality Sync prompt | — |

---

## Quality Gates

### Before Phase 3 (Implementation Planning)
- [ ] Strategy approved by you
- [ ] PRD reviewed and scope locked
- [ ] Master plan phase boundaries make sense

### Before Phase 4 (Development)
- [ ] Implementation spec has Assumptions section
- [ ] All architects have converged (not just stopped arguing)
- [ ] You've decided on all flagged tradeoffs
- [ ] Spec is detailed enough that Developer won't need judgment calls

### Before Merge
- [ ] Code Reviewer approved with spec citations
- [ ] Adversarial Reviewer listed specific attack vectors tried
- [ ] Synthesizer flagged any shallow reviews
- [ ] No unresolved conflicts between reviewers
- [ ] You've read synthesis and made decision

### Before Shipping Version
- [ ] Acceptance criteria validated by QA
- [ ] Critical bugs fixed
- [ ] You've actually used it yourself

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
3. Open Debugger session with: exact reproduction steps, expected vs. actual behavior, relevant logs/errors
4. Fix → Single focused review → Merge
5. Add Post-Mortem to session log: What broke, why review missed it, prevention for next time

**Prevention:** Adversarial Reviewer should try to break the feature, not just review code.

---

### E3: Security Vulnerability Found

**Symptoms:**
- Adversarial reviewer finds auth bypass, injection, data leak
- External security report received
- Suspicious behavior in production logs

**Protocol:**
1. Do NOT discuss in public PR comments or issues
2. Assess severity:
   - **Critical (exploitable now):** Drop everything. Fix in single session. Merge immediately.
   - **High (exploitable with effort):** Fix within 24 hours. Limited review.
   - **Medium/Low:** Add to next version scope with priority.
3. After fix: Document in private security log, not public session log

**Prevention:** Add security considerations to Implementation Architect spec.

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

**Prevention:** Development Initiation prompt includes: "CRITICAL: Implement ONLY what's specified."

---

### E5: Context Collapse (Model Contradicts Itself)

**Symptoms:**
- Model refers to decisions that weren't made
- Model forgets earlier decisions in same session
- Responses become generic instead of project-specific
- Model asks questions already answered

**Protocol:**
1. Do NOT continue. You'll compound the confusion.
2. Use the Context Refresh prompt (in llm-development-playbook.md)
3. If summary is wrong: Correct explicitly, then continue
4. If summary is vague: Start fresh session

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
3. If still generic: Add a new reviewer (different model) with instruction: "Other reviewers approved this quickly. I'm suspicious. Find something wrong."
4. If new reviewer also approves generically: Proceed, but note in session log that review was shallow.

**Prevention:** Require minimum review specificity; require Adversarial Reviewer to list attack vectors tried.

---

## Escalation Matrix

When agents disagree, use these resolution rules. Avoid ad-hoc decisions.

### Architect Disagreements

| Scenario | Resolution |
|----------|------------|
| 2 agree, 1 disagrees | Dissenter must provide CONCRETE alternative with rationale. If alternative is weaker, majority wins. If equal or stronger, escalate to you. |
| All 3 disagree | You decide. But first, ask each: "What's your core concern? What would change your mind?" |
| Same point debated 2+ rounds | STOP debating. Either (a) scope is too big—split it, or (b) both approaches work—you pick one. |

### Review Disagreements

| Scenario | Resolution |
|----------|------------|
| Code Reviewer approves, Adversarial blocks | **Adversarial wins by default.** To override: YOU must understand the concern and explicitly accept the risk. |
| Both approve but Synthesizers disagree on severity | Dig into raw reviews. Synthesizer disagreement means something got compressed away. |
| Reviewer flags issue, Developer disagrees | Developer must explain why it's not an issue. Reviewer has final say unless you override. |

### Convergence Stalls

| Scenario | Resolution |
|----------|------------|
| 3 rounds, no progress | Ask: "Is this MUST-RESOLVE-NOW or CAN-REVISIT-LATER?" Defer if possible. |
| 5 rounds, still debating | **Mandatory scope split.** The version is too big. |
| Architects stop arguing but don't actually agree | **Not convergence.** Require: "I agree this is better because [X]." Exhaustion ≠ agreement. |

---

## Failure Modes Catalog

Learn from these common failure patterns to prevent them.

| Failure Mode | Symptoms | Root Cause | Prevention |
|--------------|----------|------------|------------|
| **Spec Drift** | Code does Y, spec says X, both "work" | Spec not updated during development | Require spec-citation in code review |
| **Review Theater** | Everyone approves, no substantive comments | Reviewers don't read, or sycophancy | Require specific line citations; use Adversarial Reviewer |
| **Convergence Collapse** | Architects stop debating but don't agree | Exhaustion, social pressure | Require "I agree this is better because [X]" |
| **Scope Creep** | Developer adds "improvements" | Ambiguous spec, developer judgment | Strict rule: not in spec = not in PR |
| **Testing Theater** | 100% coverage, broken feature | Tests test implementation, not behavior | Require edge case tests; Adversarial tests the tests |
| **Sycophancy Cascade** | All reviewers agree, all are wrong | Same model family, shared blind spots | Use different models; require dissent before approval |
| **Master Plan Rot** | Plan says X, codebase is Y | No Reality Sync after shipping | Chief Architect review after each version |

---

## Quick Reference

### Anti-Sycophancy Triggers
- "What would YOU have done differently?"
- "Ultrathink" (Claude) / "Think step by step" (GPT) / "Take your time, analyze deeply" (Gemini)
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

### Quality Gates Checklist
- [ ] Spec has Assumptions section reviewed by architects
- [ ] Adversarial reviewer listed specific attack vectors tried
- [ ] Synthesizer flagged any shallow reviews
- [ ] No unresolved conflicts between reviewers

---

## Orchestration Efficiency

You're the manual trigger for everything. Here's how to minimize friction:

### Tab Discipline

Set up one browser tab per role category:
- **Tab 1:** Architects (Strategy, Product, Chief, Implementation)
- **Tab 2:** Developer (Antigravity, Cursor, Claude Code)
- **Tab 3:** Reviewers (Code, Adversarial, Synthesizers)
- **Tab 4:** This handbook + prompt generation

Each tab maintains context for its role. Don't mix roles in a single context.

### Tool Assignment

Fill this in as you figure out what works:

| Role | Primary Tool | Backup Tool | Notes |
|------|-------------|-------------|-------|
| Strategist | | | Best for pushback, devil's advocate |
| Product Architect | | | Thorough with acceptance criteria |
| Chief Architect | | | Broad technical knowledge |
| Implementation Architect | | | Detail-oriented, spec writing |
| Developer | Antigravity / Cursor | Claude Code | Agentic, autonomous |
| Code Reviewer | | | Different model than Developer |
| Adversarial Reviewer | | | **Different model than Code Reviewer** |
| Synthesizer | | | Good at compression |
| Debugger | | | Good at logs, hypothesis |
| QA Analyst | | | Systematic |

### Critical Rule: Model Diversity in Review
```
Developer ──────────► Model A
Code Reviewer ──────► Model B (different from A)
Adversarial ────────► Model C (different from A and B)
Synthesizer A ──────► Any model
Synthesizer B ──────► Different from Synthesizer A
```
If all reviewers use the same model family, they share blind spots.

---

## Iteration Log

Track what you learn about your process:

| Date | Observation | Change Made |
|------|-------------|-------------|
| | | |

---

## Open Questions

### Process Questions (Figure Out Over Time)
- [ ] Which model is best at finding strategy holes?
- [ ] Which model writes the cleanest specs?
- [ ] Is LLM-on-LLM review actually catching bugs, or just theater?
- [ ] Where do I still need to add human judgment?

---

*This document is for you. To generate prompts, give an LLM [llm-development-playbook.md](llm-development-playbook.md) and [session-variables.md](session-variables.md) with your values filled in.*
