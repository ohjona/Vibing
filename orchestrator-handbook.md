# Orchestrator Handbook v1.2

> Generator Prompt + Workflow + Emergencies

---

## Generator Prompt

Copy this to your agentic CLI at the start of each session:

```
Read llm-development-playbook.md and session-variables.md.

Generate ready-to-paste prompts using the EXACT templates from the playbook. Do NOT modify, expand, or "improve" the prompts — use them VERBATIM. Fill in {{VARIABLES}} with values from session-variables.md.

Generate these prompts:
1. Architect
2. Critic — Standard (Claude)
3. Critic — Technical (Gemini)  
4. Critic — Adversarial (GPT)
5. Spec Consolidator
6. Architect Revision
7. Developer
8. Standard Reviewer
9. Adversarial Reviewer
10. Review Consolidator
11. Architect Review Response
12. Developer Amendment
13. Architect Final Review (uses Standard Reviewer prompt)
14. Reality Sync (if version just shipped)

Output each prompt in a code block I can copy.
```

**What this does:** You input variables once → LLM outputs all prompts with your values filled in → you copy each to the appropriate model as needed.

**Why verbatim matters:** The prompts are carefully designed (adversarial framing, output formats, 300-word minimums, complexity audit FIRST). If the LLM "improves" them, constraints get softened.

---

## Workflow

```
You fill session-variables.md
    ↓
Generator Prompt → LLM outputs all prompts
    ↓
Copy Architect prompt → Get spec
    ↓
Copy 3 Critic prompts → Send IN PARALLEL (Claude, Gemini, GPT)
    ↓
Copy Consolidator prompt → Synthesize feedback
    ↓
Copy Architect Revision prompt → One revision
    ↓
★ GATE 1: Proceed to Development?
    ↓
Copy Developer prompt → Get PR
    ↓
Copy 2 Reviewer prompts → Send IN PARALLEL (Standard, Adversarial)
    ↓
Copy Review Consolidator prompt → Synthesize
    ↓
Copy Architect Review Response prompt → Evaluate feedback, update spec if needed
    ↓
Copy Developer Amendment prompt → Implement changes (if any)
    ↓
Copy Standard Reviewer prompt → Architect Final Review
    ↓
★ GATE 2: Merge?
    ↓
Copy Reality Sync prompt → Update Master Plan
```

**Handoffs:** 8-10 per feature
**Your decisions:** 2 gates
**Parallel steps:** Critics (3 models), Reviewers (2 models)
**No loops:** Final Review surfaces issues at Gate 2, doesn't restart cycle

---

## The Two Gates

```
┌──────────────────────────────────────────┐
│  GATE 1: PROCEED TO DEVELOPMENT?         │
│  After Architect revision                │
│  You've seen all critic feedback         │
│  You decide: build or clarify scope      │
├──────────────────────────────────────────┤
│  GATE 2: MERGE?                          │
│  After review consolidation              │
│  You've seen complexity concerns         │
│  You decide: ship or address issues      │
└──────────────────────────────────────────┘
```

---

## One Revision Cap

If major issues persist after one revision:
- **Requirements unclear** → Fix requirements, don't iterate
- **Scope too large** → Split the feature
- **Fundamental disagreement** → You decide

More rounds don't fix these. They defer the problem.

**Final Review Rule:** If Architect Final Review finds issues, they surface at Gate 2. You decide to ship, defer, or descope. We do NOT loop back to Developer Amendment.

---

## Role Monitoring

### Architect
**Watch for:** Scope creep, YAGNI violations
**Red flag:** "This will allow us to easily..."
**Action:** Send back — "Design for THIS version only"

### Developer
**Watch for:** Stuck loops, scope creep, ComplexityFlags
**Red flags:**
- "I'll also improve/refactor X" → Scope creep
- "Should I do X or Y?" → Spec was ambiguous
- 10+ file changes when spec mentioned 3 → Review before proceeding

**Handling ComplexityFlags:**
1. **Accept:** Use simpler alternative
2. **Override:** Cite specific PRD requirement
3. **Escalate:** Ask Architect to clarify

### Reviewers
**Watch for:** Rubber-stamp approvals
**Rule:** Side with Adversarial on complexity concerns

**Invalid Adversarial outputs (send back):**
- "Looks good" — Too vague
- Short review (<300 words) — Not thorough
- No complexity audit section

---

## Emergency Procedures

### E1: Developer Stuck in Loop
1. STOP immediately
2. Read the last 3 attempts yourself
3. Restart with: "Do NOT try [X] again. The issue is [Y]. Try [Z] instead."

### E2: Post-Merge Bug
1. REVERT immediately
2. Hotfix branch from last good commit
3. Fix → Single review → Merge

### E3: Security Vulnerability
1. Do NOT discuss in public
2. Assess severity
3. Document in private security log after fix

### E4: Developer Goes Rogue
1. STOP
2. `git diff` to see actual vs. expected
3. `git checkout` or `git revert`
4. Restart with constraint

### E5: Context Collapse
1. Do NOT continue
2. Use Context Refresh prompt from playbook
3. Correct or start fresh

### E6: Reviewers Approve Too Easily
1. Do NOT merge
2. Add new reviewer: "find something wrong"

### E7: ComplexityFlag Raised
1. READ the flag
2. Accept / Override / Escalate
3. Do NOT ignore

---

## Escalation Matrix

| Conflict | Resolution |
|----------|------------|
| Critics disagree | You decide on conflicts |
| Adversarial blocks, Standard approves | **Adversarial wins** |
| ComplexityFlag raised | Accept / Override / Escalate |
| Major issues after revision | Split scope or clarify requirements |

**Side with the pessimist.** Complexity is sticky; bugs are transient.

---

## Quality Checklist

### Before Development (Gate 1)
- [ ] Spec has Assumptions section
- [ ] Cut List exists
- [ ] Adversarial complexity concerns addressed
- [ ] No "future flexibility" speculation

### Before Merge (Gate 2)
- [ ] Adversarial review 300+ words
- [ ] Complexity audit completed
- [ ] No unresolved conflicts

---

## Quick Reference

### Key Rules
- **Parallel fan-out:** Send critics/reviewers simultaneously, don't wait
- **One revision cap:** If issues persist, scope is wrong
- **Side with pessimist:** Adversarial wins ties
- **ComplexityFlags are valid:** Developer can refuse

### Anti-Complexity Triggers
- "What breaks if we delete this?"
- "Show me 3 concrete uses for this abstraction"
- "What PRD requirement necessitates this?"
