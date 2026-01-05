# Orchestrator Handbook v1.2

> Your reference for emergencies, escalation, and the two decision gates

---

## Quick-Start

1. Write your requirements
2. Copy prompts from [llm-development-playbook.md](llm-development-playbook.md) as needed
3. Run critics in parallel, consolidate, revise once
4. **Gate 1:** Proceed to development?
5. Developer implements, reviewers review in parallel
6. **Gate 2:** Merge?

That's it.

---

## The Two Gates

Your only mandatory decision points:

```
┌──────────────────────────────────────────┐
│  1. PROCEED TO DEVELOPMENT?              │
│     After Architect revision             │
│     You've seen all critic feedback      │
│     You decide: build or clarify scope   │
├──────────────────────────────────────────┤
│  2. MERGE?                               │
│     After code review consolidation      │
│     You've seen complexity concerns      │
│     You decide: ship or address issues   │
└──────────────────────────────────────────┘
```

Everything else is execution.

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

## One Revision Cap

If major issues persist after one revision:
- **Requirements unclear** → Fix requirements, don't iterate
- **Scope too large** → Split the feature
- **Fundamental disagreement** → You decide

More rounds don't fix these. They defer the problem.

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
2. Use Context Refresh prompt
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
- **Parallel fan-out:** Don't wait for responses sequentially
- **One revision cap:** If issues persist, scope is wrong
- **Side with pessimist:** Adversarial wins ties
- **ComplexityFlags are valid:** Developer can refuse

### Anti-Complexity Triggers
- "What breaks if we delete this?"
- "Show me 3 concrete uses for this abstraction"
- "What PRD requirement necessitates this?"

---

*For prompts and constraints, see [llm-development-playbook.md](llm-development-playbook.md).*
