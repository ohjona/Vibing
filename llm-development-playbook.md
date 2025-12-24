# LLM Development Playbook
> A solo developer's operating system for LLM-augmented product development
> Last updated: December 2024

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
│              ┌────────────────────────┐                  │
│              │   ONTOS CONTEXT MAP    │ ◄── Ground Truth │
│              │  (All agents read this)│                  │
│              └────────────────────────┘                  │
│                           │                              │
│    ┌──────────┬──────────┬──────────┬──────────┐        │
│    │Strategist│ Architect│ Developer│ Reviewer │  ...   │
│    └──────────┴──────────┴──────────┴──────────┘        │
└─────────────────────────────────────────────────────────┘
```

You don't read code. You read summaries, make decisions, and approve merges.

---

## Context Backbone: Project Ontos

All agents operate from a shared context graph via [Project Ontos](https://github.com/ohjona/Project-Ontos).

### Why This Matters
- **No context drift**: Every agent reads from `Ontos_Context_Map.md`, not their own interpretation
- **Decisions survive migrations**: Strategy and product layers persist even when atoms (code) get rewritten
- **Session continuity**: Archive sessions to preserve decision rationale for future agents

### Ontos Layer Mapping to This Playbook

| Ontos Layer | Playbook Artifact | Survives Rewrite? |
|-------------|-------------------|-------------------|
| `kernel` | Mission, values | ✅ Always |
| `strategy` | STRATEGY.md | ✅ Always |
| `product` | PRD.md, MASTER_PLAN.md | ✅ Always |
| `atom` | VERSION_X_SPEC.md, Code | ❌ Disposable |

### Tagging Playbook Docs for Ontos

Add YAML frontmatter to each artifact so Ontos can track dependencies:

```yaml
# STRATEGY.md
---
id: strategy
type: strategy
depends_on: [mission, problem_statement]
status: approved
---
```

```yaml
# PRD.md
---
id: prd_v1
type: product
depends_on: [strategy]
status: approved
---
```

```yaml
# VERSION_1_SPEC.md
---
id: v1_implementation_spec
type: atom
depends_on: [master_plan]
status: in_progress
---
```

### Agent Activation Pattern
Every agent interaction should start with:
```
"Activate Ontos" → Agent reads context map → Agent confirms what it loaded
```

### Session Archival
After significant decisions:
```
"Archive Ontos" → Decisions logged → Context preserved for next session
```

Run `python3 ontos_init.py` or "Maintain Ontos" to rebuild the context map after creating/updating artifacts.

---

## Anti-Sycophancy Prompt Patterns

LLMs default to agreeing. Your prompts must force independent thinking.

### Key Phrases That Force Divergence

| Pattern | Why It Works |
|---------|-------------|
| "What would you have done differently?" | Can't rubber-stamp—must propose alternatives |
| "Keep your integrity" | Permission to disagree, even if other models approved |
| "Be open-minded while keeping your integrity" | Balance: consider others, but don't just conform |
| "Let's check... what else could have been added" | Forces additive thinking, not just validation |
| "ultrathink" | Triggers extended reasoning (Claude), prevents fast pattern-matching |

### Phrase Substitutions

| Instead of... | Use... |
|---------------|--------|
| "Review this code" | "What would YOU have done differently?" |
| "Check for bugs" | "Ultrathink. Try to break this." |
| "Is this good?" | "What's missing that should be here?" |
| "Any issues?" | "What concerns would a senior engineer raise?" |
| "Approve or reject" | "RELUCTANT APPROVE or BLOCK—no easy approvals" |

### The "Ultrathink" Pattern

When you want deep analysis, not surface validation:
```
Ultrathink and review [X].

1. How well does this match our plan? [forces spec comparison]
2. What's missing? [forces critical evaluation]  
3. What would YOU have done differently? [forces independent position]
```

This works because:
- "Ultrathink" signals you want depth, not speed
- Numbered questions prevent "looks good!" one-liners
- "What would YOU do" requires taking a position, not just validating

### Synthesis Cross-Review Prompt

When multiple models have produced reviews/syntheses:
```
[Models A, B, and C], all acting as fellow architects, have reviewed 
the implementation plan and left reviews in [location].

Please ultrathink and review them. Carefully consider what we can:
- Learn
- Adopt
- Refine
- Ignore

Be open-minded while keeping your integrity.
```

### Why "Fellow Architects" Not "Reviewers"
Framing them as peers with equal authority prevents deference. "Reviewer" implies hierarchy; "fellow architect" implies each has valid independent judgment.

### Model-Specific Notes
_Track which models respond best to which anti-sycophancy patterns:_

```
Claude: 
Gemini: 
GPT: 
```

---

## Risk Acknowledgment: The No-Code-Review Model

You've chosen to never review code yourself. This is a valid choice for solo development of simple apps, but be honest about the tradeoffs:

### Where This Works
- Single-feature applications
- Low-stakes projects (not handling money, health, security)
- Apps where "ship and fix" is acceptable
- When you can validate by *using* the product, not reading the code

### Where This Breaks
- **LLM sycophancy**: Model B reviewing Model A's code often assumes A knew what it was doing. They catch syntax, miss logic.
- **Synthesizer compression**: If 3 reviewers each flag a subtle issue differently, your summarizer might compress it to "minor concerns."
- **No ground truth**: Bad architecture that works today becomes expensive tomorrow. You won't see it coming.
- **Security**: LLMs miss vulnerabilities that humans catch. For anything touching auth, payments, or user data—consider paying a human to review.

### Your Active Mitigations
1. **Force independent judgment**: Prompts like "what would YOU have done differently" break rubber-stamping
2. **Multi-synthesizer pattern**: Multiple models synthesize → then review each other's syntheses
3. **Project Ontos**: Persistent context graph that all tools read from—ground truth survives tool switches
4. **Adversarial Reviewer role**: One LLM prompted to reject by default
5. **Spec-citation requirement**: Reviewers must map code to spec lines

---

## The Team (LLM Roles)

### 🎯 Strategist
**What they do:** Pressure-test your strategy, find gaps, suggest pivots
**Input:** Your rough problem statement, market observations
**Output:** Refined strategy doc, risk flags, alternative approaches
**Your job:** Accept/reject/refine their pushback

### 📋 Product Architect  
**What they do:** Translate strategy into structured PRD, define scope, write acceptance criteria
**Input:** Approved strategy, your feature priorities
**Output:** PRD with user stories, success metrics, scope boundaries
**Your job:** Ensure it matches your vision, cut scope creep

### 🏗️ Chief Architect
**What they do:** Break PRD into technical phases, choose patterns, define system boundaries
**Input:** Approved PRD
**Output:** Master implementation plan, tech stack decisions, phase breakdown
**Your job:** Sanity check complexity, approve phase boundaries
**Ongoing:** After each version ships, Chief Architect reviews codebase and updates Master Plan if reality drifted from the plan. This keeps the Master Plan useful, not fictional.

### 👷 Implementation Architect
**What they do:** Create detailed specs for a single phase/version, then iterate with fellow architects until convergence
**Input:** Master plan section + previous version spec + current codebase state + session history
**Output:** Detailed implementation plan that builds on what ACTUALLY exists
**Iteration:** Reads fellow architect reviews, updates spec, flags tradeoffs for your decision
**Your job:** Decide on flagged tradeoffs; approve when converged
**Key responsibility:** Flag any deviations between Master Plan and reality; recommend how to handle

### 🔨 Developer (Agentic)
**What they do:** Actually write code, create PRs
**Input:** Implementation plan
**Output:** Working code, PR ready for review
**Your job:** Nothing—this runs autonomously

### 🔍 Code Reviewer
**What they do:** Review PRs for bugs, security, adherence to plan
**Input:** PR diff + original implementation plan
**Output:** Approval or change requests with specific line citations
**Your job:** Read synthesis, break ties, final merge decision

### 😈 Adversarial Reviewer (CRITICAL ROLE)
**What they do:** Actively try to find problems. Prompted to reject by default.
**Input:** PR diff + implementation plan + other reviewers' approvals
**Output:** List of concerns, edge cases, security issues, or "I tried to break this and couldn't"
**Your job:** If adversarial reviewer finds real issues, block merge
**Why this exists:** Normal reviewers are biased toward approval. This one is biased toward rejection. Balance.

### 📝 Synthesizer (Multi-Model Pattern)
**What they do:** Compress multiple review outputs into decision-ready summary for you
**Input:** All reviewer outputs (Code Reviewer + Adversarial Reviewer)
**Output:** One-page summary: what's approved, what's contested, what needs your decision
**Your job:** Read this instead of raw reviews. Make decisions.

**Critical pattern:** Don't trust a single synthesizer.
```
Reviews ──► Synthesizer A (Claude) ──► Summary A
        └─► Synthesizer B (Gemini) ──► Summary B
                                           │
                                           ▼
                              You compare A vs B
                              Disagreement = dig deeper
```

If Synthesizer A says "ready to merge" and Synthesizer B says "concerns about auth," that's a signal—not a tiebreaker for you to pick one.

### 🐛 Debugger
**What they do:** Diagnose and fix issues when things break
**Input:** Error logs, reproduction steps, relevant code context
**Output:** Diagnosis + fix PR
**Your job:** Direct which errors to prioritize, approve fix approach

### 🧪 QA Analyst
**What they do:** Write test cases, validate against acceptance criteria
**Input:** PRD acceptance criteria + working build
**Output:** Test results, bug reports
**Your job:** Prioritize which bugs matter

---

## The Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                        YOUR INPUT REQUIRED                       │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 0: Strategy                                               │
│  ─────────────────                                               │
│  You write: Problem statement, market thesis, rough vision       │
│  Strategist: Challenges assumptions, refines, structures         │
│  You: Accept/reject, finalize strategy doc                       │
│                                                                  │
│  Artifact: STRATEGY.md                                           │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 1: Product Definition                                     │
│  ──────────────────────────                                      │
│  You: Define core features, priorities, constraints              │
│  Product Architect: Writes full PRD with acceptance criteria     │
│  You: Review, adjust scope, approve                              │
│                                                                  │
│  Artifact: PRD.md                                                │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 2: Technical Planning                                     │
│  ─────────────────────────                                       │
│  Chief Architect: Creates master plan, breaks into versions      │
│  You: Review phase boundaries, approve                           │
│                                                                  │
│  Artifact: MASTER_PLAN.md                                        │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
         ┌──────────────────────────────────────────┐
         │         FOR EACH VERSION/PHASE          │
         └──────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 3: Implementation Planning + Convergence                  │
│  ──────────────────────────────────────────────                  │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ Implementation Architect: Writes detailed spec          │    │
│  └─────────────────────────────────────────────────────────┘    │
│                          │                                       │
│                          ▼                                       │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ Fellow Architects: Review spec, leave feedback          │    │
│  └─────────────────────────────────────────────────────────┘    │
│                          │                                       │
│                          ▼                                       │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ Implementation Architect: Responds, updates spec        │    │
│  └─────────────────────────────────────────────────────────┘    │
│                          │                                       │
│                          ▼                                       │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ You: Review flagged tradeoffs, make decisions           │    │
│  └─────────────────────────────────────────────────────────┘    │
│                          │                                       │
│                          ▼                                       │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ Fellow Architects: Re-review updated spec           │◄──┐   │
│  │ Output: CONVERGED or NEEDS ANOTHER ROUND            │   │   │
│  └─────────────────────────────────────────────────────────┘   │   │
│                          │                                     │   │
│               ┌──────────┴──────────┐                          │   │
│               ▼                     ▼                          │   │
│          All CONVERGED      NEEDS ANOTHER ROUND ───────────────┘   │
│               │                                                    │
│               ▼                                                    │
│  Artifact: VERSION_X_SPEC.md (approved)                            │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 4: Development (Autonomous)                               │
│  ─────────────────────────────────                               │
│  Developer: Implements code, creates PR                          │
│  You: Nothing—let it run                                         │
│                                                                  │
│  Artifact: Pull Request                                          │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 5: Review + Iteration                                     │
│  ────────────────────────────                                    │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Code Reviewer: Reviews PR, comments on GitHub          │    │
│  │  Adversarial Reviewer: Tries to break it, comments      │    │
│  │  Synthesizers: Compress reviews for your decision       │    │
│  └─────────────────────────────────────────────────────────┘    │
│                          │                                       │
│                          ▼                                       │
│                 You: Review synthesis                            │
│                    │                                             │
│         ┌─────────┴─────────┐                                   │
│         ▼                   ▼                                    │
│      APPROVE             REQUEST CHANGES                         │
│         │                   │                                    │
│         │                   ▼                                    │
│         │    ┌──────────────────────────────────┐               │
│         │    │ Developer: Reads PR comments     │               │
│         │    │ Implements fixes, pushes         │◄──┐           │
│         │    └──────────────────────────────────┘   │           │
│         │                   │                       │           │
│         │                   ▼                       │           │
│         │           Reviewers re-review ────────────┘           │
│         │           (until approved)                            │
│         │                                                        │
│         └──────────────► MERGE                                  │
│                                                                  │
│  Gate: PR approved and merged                                    │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 6: Validation                                             │
│  ─────────────────                                               │
│  QA Analyst: Tests against acceptance criteria                   │
│  Debugger: Fixes critical issues                                 │
│  You: Accept version as complete or define rework                │
│                                                                  │
│  Gate: Version shipped                                           │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
                    [Next version or DONE]
```

---

## Tool Assignment (Your Current Setup)

Fill this in as you figure out what works. Format: `Role: Tool (notes)`

| Role | Primary Tool | Backup Tool | Notes |
|------|-------------|-------------|-------|
| Strategist | | | Best for pushback, devil's advocate |
| Product Architect | | | Needs to be thorough with acceptance criteria |
| Chief Architect | | | Needs broad technical knowledge |
| Implementation Architect | | | Detail-oriented, spec writing |
| Developer | Antigravity / Cursor | Claude Code / Codex | Agentic, autonomous |
| Code Reviewer | | | Different model than Developer |
| Adversarial Reviewer | | | **Must be different model than Code Reviewer** |
| Synthesizer | | | Good at compression without losing conflicts |
| Debugger | | | Good at reading logs, suggesting fixes |
| QA Analyst | | | Can reference PRD, systematic |

### Critical Rule: Model Diversity in Review
```
Developer ──────────► Model A
Code Reviewer ──────► Model B (different from A)
Adversarial ────────► Model C (different from A and B)
Synthesizer A ──────► Model A, B, or C
Synthesizer B ──────► Different from Synthesizer A
```
If all reviewers use the same model family, they share blind spots.
If both synthesizers are the same model, you lose the disagreement signal.

### Model Strengths (Your Observations)

_Add notes here as you learn which models excel at what:_

```
Claude: 
Gemini: 
GPT/Codex: 
Other:
```

---

## Artifact Templates

### STRATEGY.md
```markdown
# [Project Name] Strategy

## Problem Statement
What pain exists? For whom? Why now?

## Market Thesis  
Why will this work? What's the wedge?

## Vision
What does success look like in 1 year? 3 years?

## Anti-Goals
What are we explicitly NOT doing?

## Key Risks
What could kill this?
```

### PRD.md
```markdown
# [Project Name] PRD

## Overview
One paragraph: what is this?

## Success Metrics
How do we know it worked?

## User Stories
- As a [user], I want [goal] so that [reason]

## Scope
### In Scope (v1)
### Out of Scope (later)
### Never

## Acceptance Criteria
Specific, testable conditions for "done"
```

### MASTER_PLAN.md
```markdown
# [Project Name] Master Plan

## Tech Stack
- Frontend:
- Backend:
- Database:
- Hosting:

## Architecture Overview
[High-level diagram or description]

## Version Breakdown

### v0.1 - Foundation
Goal:
Includes:
Estimated complexity:

### v0.2 - Core Feature
...
```

### VERSION_X_SPEC.md
```markdown
# [Project Name] v[X] Implementation Spec

## Goals for This Version
What's shipping?

## Technical Approach
How will we build it?

## File Structure
```
/src
  /components
  ...
```

## API Contracts
Endpoints, request/response shapes

## Data Models
Schema definitions

## Implementation Order
1. First build...
2. Then...

## Testing Requirements
What must be tested before PR?
```

---

## Quality Gates

### Before Phase 3 (Implementation Planning)
- [ ] Strategy approved by you
- [ ] PRD reviewed and scope locked
- [ ] Master plan phase boundaries make sense

### Before Phase 4 (Development)
- [ ] Implementation spec reviewed
- [ ] No ambiguities that will cause agentic tool to hallucinate
- [ ] Spec is detailed enough that Developer won't need to make judgment calls

### Before Merge
- [ ] Code Reviewer approved with spec citations
- [ ] Adversarial Reviewer couldn't break it (or issues are addressed)
- [ ] Synthesizer summary shows no unresolved conflicts
- [ ] You've read synthesis and made decision

### Before Shipping Version
- [ ] Acceptance criteria validated by QA
- [ ] Critical bugs fixed
- [ ] You've actually used it yourself

---

## Orchestration Efficiency

You're the manual trigger for everything. Here's how to minimize friction:

### Session Management (with Ontos)

**Starting a session:**
```
1. Open your tool (Claude Code, Cursor, Gemini CLI, etc.)
2. Say: "Activate Ontos"
3. Agent reads Context Map, loads relevant history
4. You're caught up without re-explaining
```

**Ending a session:**
```
1. Say: "Archive Ontos"
2. Agent saves decisions and context changes
3. Next session picks up where you left off
```

**After creating/updating artifacts:**
```
1. Say: "Maintain Ontos"
2. Context map rebuilds with new dependencies
```

### Continuous Orchestration (Your Workflow)

You work throughout the day, triggering agents as needed. Each tab = one role. No context switching within a tab.

**Setup:**
- Tab 1: Chief Architect / Implementation Architect
- Tab 2: Developer (Antigravity / Cursor / Claude Code)
- Tab 3: Code Reviewers
- Tab 4: Synthesis / Cross-review

**Each tab starts with:** "Activate Ontos" — so it has the full context graph.

**Tab discipline:** If Tab 1 is your architect, it stays architect. Don't ask it to debug. Open a new tab for that.

### What to Copy-Paste at Each Handoff

| Handoff | What You Give Next Agent | Ontos Command |
|---------|-------------------------|---------------|
| Start session | — | "Activate Ontos" |
| Strategy → Product Architect | STRATEGY.md (full) | — |
| PRD → Chief Architect | PRD.md (full) | — |
| **SPEC CONVERGENCE** | | |
| Master Plan → Impl Architect | Master Plan section + prev spec path + "read codebase" | — |
| Impl Spec → Fellow Architects | Use Implementation Plan Review prompt | — |
| Architect Reviews → Impl Architect | Use Impl Architect Response prompt | — |
| Updated Spec → Fellow Architects | Use Fellow Architects Re-Review prompt | — |
| (If NEEDS ANOTHER ROUND) | Use Impl Architect Iteration prompt | — |
| (Repeat until all CONVERGED) | — | — |
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
| (Repeat until APPROVE) | — | — |
| **POST-MERGE** | | |
| After merge | — | "Archive Ontos" |
| After version ships → Chief Architect | Use Reality Sync prompt | — |
| After Reality Sync | Update MASTER_PLAN.md | "Maintain Ontos" |

### Done Criteria (When to Stop Iterating)

| Phase | It's Done When... |
|-------|-------------------|
| Strategy | You believe it, and Strategist couldn't poke fatal holes |
| PRD | Acceptance criteria are specific enough to test |
| Master Plan | Phase boundaries feel shippable, not arbitrary |
| Impl Spec (initial) | Implementation Architect has written complete spec |
| Spec Convergence | All architects agree; no blocking concerns; tradeoffs decided |
| Code | PR exists and tests pass |
| Review Iteration | Developer addressed all feedback; reviewers have no new concerns |
| Review | Synthesis says "ready" with no unresolved conflicts |
| Version | You used it and it works |

### After Version Ships (Reality Sync)

Before starting the next version:

1. **Archive the session** — "Archive Ontos" to capture decisions
2. **Run Reality Sync** — Chief Architect compares codebase to Master Plan
3. **Update Master Plan** — Apply Chief Architect's recommended changes
4. **Maintain Ontos** — Rebuild context map with updated artifacts

This keeps your Master Plan useful. Without this step, the plan becomes fiction and the Implementation Architect has to do all the reality-checking.

### When to Escalate (Your Judgment Required)

These situations need you, not more LLM passes:
- Reviewers disagree and neither is obviously wrong
- Adversarial reviewer found something, but you're not sure if it matters
- Agentic developer is stuck in a loop
- The spec was ambiguous and the developer guessed wrong
- Something works but feels wrong (trust this instinct)

---

## Spec Convergence Protocol

Implementation plan review takes the most time—more than coding itself. This is correct. Bad specs create bad code. Invest here.

### The Pattern

```
Implementation Architect writes spec
            │
            ▼
    ┌───────────────────┐
    │  Fellow Architects│ ◄── Multiple architects review (initial)
    │  Review (Initial) │     Each writes feedback (MD files)
    └───────────────────┘
            │
            ▼
    ┌───────────────────┐
    │  Implementation   │     Reads all reviews
    │  Architect        │     Updates spec
    │  Responds         │     Flags tradeoffs for your decision
    └───────────────────┘
            │
            ▼
    ┌───────────────────┐
    │  You Review       │     Decide on flagged tradeoffs
    │  Tradeoffs        │     
    └───────────────────┘
            │
            ▼
    ┌───────────────────┐
    │  Fellow Architects│     Check if concerns addressed
    │  Re-Review        │◄─┐ Output: CONVERGED or NEEDS ANOTHER ROUND
    └───────────────────┘  │
            │              │
            ▼              │
      All CONVERGED? ──No──┤
            │              │
            │    ┌─────────┴─────────┐
            │    │ Implementation    │
            │    │ Architect         │
            │    │ Iteration         │
            │    └───────────────────┘
           Yes
            │
            ▼
      Spec is ready for Developer
```

### What "Convergence" Means

You're not building false consensus. You're iterating until:
- All architects agree THIS approach is better than alternatives
- Ambiguities are resolved (coding agent won't have to guess)
- Disagreements are either resolved or explicitly deferred

**Typical:** 3 rounds
**Complex topics:** 5 rounds
**If >5 rounds:** The scope is too big. Split the version.

### How to Run Convergence

**Step 1 — Fellow Architects Review (Initial):**
```
[Architect A], [Architect B], and [Architect C] are fellow architects.

Here is the implementation plan for {{VERSION}}: {{SPEC_PATH}}

Each of you: review independently. What would you do differently?
Don't coordinate. I want your individual perspectives.

Save your reviews to: {{REVIEW_PATH}}
```

**Step 2 — Implementation Architect Responds:**
```
Use the "Implementation Architect Response to Review Prompt"

Implementation Architect reads all reviews, updates spec, flags tradeoffs.
```

**Step 3 — You Review Tradeoffs:**
- Read the updated spec's revision history
- Decide on any flagged tradeoffs
- Tell Implementation Architect your decisions

**Step 4 — Fellow Architects Re-Review:**
```
Use the "Fellow Architects Re-Review Prompt"

Each architect checks if their concerns were addressed.
They output: CONVERGED or NEEDS ANOTHER ROUND
```

**Step 5 — Check Convergence:**
- If all architects say CONVERGED → Spec is ready for Developer
- If any say NEEDS ANOTHER ROUND → Implementation Architect uses Iteration Prompt, then back to Step 4

**Typical:** 3 rounds (Steps 2-4 repeated twice after initial review)
**Complex:** 5 rounds
**If >5:** Scope is too big. Split the version.

### Exit Criteria

Spec is ready when ALL of these are true:
- [ ] No architect has blocking concerns
- [ ] Every "what would you do differently" has been addressed or explicitly deferred
- [ ] A coding agent reading this spec would not need to make judgment calls
- [ ] Acceptance criteria are specific enough to test

### When to Defer (Not Resolve)

Some disagreements don't need resolution before coding:
- Style preferences (tabs vs spaces level stuff)
- Optimizations that can be done later
- Features explicitly out of scope for this version

Log deferrals in the spec: "DEFERRED: [issue] - will address in v{{NEXT_VERSION}}"

---

## Handoff Kit

The goal: **Input variables once → Get ready-to-paste prompt.**

Your friction isn't the prompts themselves—it's finding them, copy-pasting, then manually updating URLs, PR numbers, version numbers, etc. This kit solves that.

### Variable Reference

These variables appear across prompts. Collect them once per session/task:

| Variable | Description | Example |
|----------|-------------|---------|
| `{{VERSION}}` | Current version number | `v2.8.1` |
| `{{PREV_VERSION}}` | Previous version number | `v2.8.0` |
| `{{PR_URL}}` | Pull request URL | `https://github.com/user/repo/pull/24` |
| `{{PR_NUMBER}}` | Just the PR number | `24` |
| `{{BRANCH}}` | Current branch name | `feat/v2.8-session-logging` |
| `{{SPEC_PATH}}` | Path to current implementation spec | `.ontos-internal/strategy/v2.8/VERSION_2.8.1_SPEC.md` |
| `{{PREV_SPEC_PATH}}` | Path to previous version's spec | `.ontos-internal/strategy/v2.8/VERSION_2.8.0_SPEC.md` |
| `{{REVIEW_PATH}}` | Path to review outputs | `.ontos-internal/strategy/proposals/v2.9/` |
| `{{TOOL_NAME}}` | Which tool wrote the code | `Antigravity (Claude Opus 4.5)` |
| `{{PROJECT}}` | Project name | `Project Ontos` |
| `{{ROUND_NUMBER}}` | Current review iteration | `2` |

### Quick-Start: The Generator Prompt

Paste this to any LLM at the start of a work session:

```
I'm starting a work session. Generate my handoff prompts for today.

Project: {{PROJECT}}
Current Version: {{VERSION}}
Previous Version: {{PREV_VERSION}}
Branch: {{BRANCH}}
PR URL (if exists): {{PR_URL}}
Current Spec Location: {{SPEC_PATH}}
Previous Spec Location: {{PREV_SPEC_PATH}}
Review Location: {{REVIEW_PATH}}
Coding Tool: {{TOOL_NAME}}

Based on my LLM Development Playbook, generate ready-to-paste prompts for:

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
13. Synthesis Cross-Review
14. Chief Architect Reality Sync (if version just shipped)

Use my exact variable values above. Output each prompt in a code block I can copy.
```

This front-loads the variable collection. You do it once, get all prompts ready.

### Alternative: Manual Prompt Assembly

If you prefer to grab prompts individually, use the templates below. Replace `{{VARIABLES}}` with your actual values.

---

## Prompts Library

### Strategist Prompt
```
You are a strategy consultant reviewing my product thesis. Your job is to:
1. Find holes in my logic
2. Challenge assumptions I haven't validated
3. Identify risks I'm not seeing
4. Suggest alternatives if my approach is weak

Be direct. I don't need encouragement—I need to know if this will fail.

Here's my strategy:
[paste STRATEGY.md]
```

### Product Architect Prompt
```
You are a product manager writing a detailed PRD. Given the approved strategy below, create a comprehensive PRD including:
- Clear scope boundaries (in/out/never)
- User stories with acceptance criteria
- Success metrics that are measurable
- Edge cases and error states

Be specific enough that an engineer could build from this without asking clarifying questions.

Strategy:
[paste STRATEGY.md]
```

### Chief Architect Prompt
```
You are a senior technical architect. Given this PRD, create a master implementation plan:
- Recommend tech stack with justification
- Break into shippable versions (aim for smallest useful increments)
- Identify technical risks and mitigation
- Define system boundaries and interfaces

Keep it simple. I'm a solo developer—don't over-engineer.

PRD:
[paste PRD.md]
```

### Chief Architect Reality Sync Prompt (After Each Major Version)
```
We just shipped {{VERSION}}. Before planning the next version, sync the Master Plan with reality.

Review:
1. The current codebase (what actually exists)
2. Session logs from {{VERSION}} development (decisions made)
3. The current MASTER_PLAN.md (what we said we'd build)

Identify:
## Deviations
[Where does the codebase differ from what MASTER_PLAN.md says?]

## Decisions That Should Be Documented
[Choices made during implementation that future work depends on]

## Master Plan Updates Needed
[Specific edits to MASTER_PLAN.md to reflect reality]

## Upcoming Version Implications
[How does the current state affect plans for next versions?]

Output the specific text changes needed for MASTER_PLAN.md.
Don't just note "update section X"—show the actual new text.
```

### Implementation Architect Prompt
```
You are a technical lead writing an implementation spec for {{VERSION}}.

Context:
- Master Plan (intended direction): [paste relevant section of MASTER_PLAN.md]
- Previous version spec: {{PREV_SPEC_PATH}} (what was planned)
- Recent session logs: Check .ontos-internal/session_logs/ for decisions made
- Current codebase: Review the repo to see what actually exists

Your job:
1. First, understand the CURRENT STATE—what was actually built, not just planned
2. Note any deviations between previous spec and actual implementation
3. Write a spec for {{VERSION}} that builds on reality, not just the plan

Include in your spec:
- Exact file structure (accounting for what already exists)
- API contracts with request/response examples
- Data models with field types
- Step-by-step implementation order
- What to test before PR
- Any constraints inherited from previous implementation decisions

Be detailed enough that an AI coding agent can implement without ambiguity.
If a developer would need to make a judgment call, you haven't been specific enough.

IMPORTANT: If the current codebase differs from the Master Plan, note the deviation 
and decide whether to:
a) Align this version with the Master Plan (refactor)
b) Update the Master Plan to reflect reality (acknowledge drift)
c) Work with reality as-is (pragmatic)

Flag these decisions clearly—I need to approve them.
```

### Implementation Plan Review Prompt (Fellow Architects)
```
Implementation Architect has written an implementation plan for {{VERSION}}.

The plan is at: {{SPEC_PATH}}

You are a fellow architect reviewing this plan. Ultrathink and evaluate:

1. **Reality Check**: Does this plan account for the CURRENT codebase, or does it 
   assume things that don't exist / ignore things that do?

2. **Completeness**: Will a coding agent have all info needed, or will it have to guess?

3. **Correctness**: Does this actually solve what the PRD requires?

4. **Continuity**: Does this build sensibly on {{PREV_VERSION}}, or does it ignore 
   decisions/constraints from previous implementations?

5. **Simplicity**: Is this overengineered? What could be simpler?

6. **What would you have done differently?**

Be specific. If you'd change the API design, show your alternative.
If you'd use a different pattern, explain why.

If the plan flags deviations from the Master Plan, evaluate whether the proposed 
approach (refactor / update plan / work with reality) is the right call.

Don't just validate—take a position. I need your independent technical judgment.

Output:
## Assessment
[Overall: Ready / Needs Revision / Major Rework]

## Reality Alignment
[Does this match what actually exists in the codebase?]

## Ambiguities That Will Cause Problems
[Places where the coding agent will have to guess]

## What I Would Do Differently
[Your alternative approach, with rationale]

## Specific Feedback
[Line-by-line or section-by-section comments]
```

### Implementation Architect Response to Review Prompt
```
Fellow architects have reviewed your implementation plan for {{VERSION}}.

Their reviews are in: {{REVIEW_PATH}}

Read all reviews carefully. For each piece of feedback:

1. **If you agree**: Update the spec to address it
2. **If you disagree**: Write a brief response explaining why your approach is better
3. **If it's a tradeoff**: Note both options and flag for my decision

After addressing all feedback:
- Save the updated spec to {{SPEC_PATH}}
- Create a changelog at the top of the spec summarizing what changed:
  ```
  ## Revision History
  - v2: Addressed architect reviews (YYYY-MM-DD)
    - Changed X per [Architect A] feedback
    - Kept Y despite [Architect B] suggestion because [reason]
    - FLAG FOR DECISION: [tradeoff that needs my input]
  ```

Do NOT:
- Ignore feedback without addressing it
- Make the spec vague to avoid conflict
- Remove detail to "simplify" unless the detail was actually wrong

The goal is a spec so clear that a coding agent won't need to make judgment calls.
```

### Fellow Architects Re-Review Prompt
```
Implementation Architect has addressed your feedback on the implementation plan for {{VERSION}}.

Updated spec: {{SPEC_PATH}}
(Check the Revision History at the top to see what changed)

Review the updates. For each piece of your original feedback:

1. **Addressed adequately**: Note it's resolved
2. **Not addressed / disagree with response**: Explain why, be specific
3. **New concerns**: If the changes introduced new issues, flag them

Output:
## Resolved
[List which of your concerns are now addressed]

## Still Outstanding
[List concerns that weren't adequately addressed, with specifics]

## New Concerns
[Any issues introduced by the changes]

## Verdict
CONVERGED (ready for development) / NEEDS ANOTHER ROUND (with clear asks)

Be open-minded while keeping your integrity. If the Implementation Architect 
made a reasonable case for their approach, acknowledge it even if you'd have 
done it differently.
```

### Implementation Architect Iteration Prompt (Multiple Review Rounds)
```
This is round {{ROUND_NUMBER}} of spec review for {{VERSION}}.

Previous feedback has been addressed. Fellow architects have left new comments.

Read the NEW feedback only. Address it following the same rules:
- Agree → update spec
- Disagree → explain why
- Tradeoff → flag for my decision

Update the revision history. Focus on the delta.

We're aiming for convergence—all architects agree this spec is ready.
If you're still getting fundamental pushback, the scope may be too big. Flag that.
```

### Spec Finalization Prompt (After Convergence)
```
All fellow architects have agreed the implementation plan for {{VERSION}} is ready.

The spec at {{SPEC_PATH}} has gone through multiple rounds of review and may have 
accumulated revision history, resolved discussions, and deferred items.

Clean it up for the Developer:

1. **Remove cruft**: Delete revision history, resolved debates, "FLAG FOR DECISION" markers
2. **Consolidate**: If decisions were made in back-and-forth, state the final decision only
3. **Clarify order**: Make the step-by-step implementation order crystal clear
4. **Add agentic instructions**: Include specific guidance for the coding agent:
   - Which files to create first
   - Which tests to write
   - When to commit (after each component? all at once?)
   - What to include in the PR description

Output a clean, final version of the spec with a new section:

## Developer Instructions
[Step-by-step instructions for the coding agent]
1. First, create...
2. Then implement...
3. Write tests for...
4. Before creating PR, verify...

The Developer reading this should have ZERO questions. If they'd need to guess, 
you haven't been specific enough.

Save to: {{SPEC_PATH}} (overwrite the reviewed version)
```

### Development Initiation Prompt
```
You are the Developer for {{VERSION}}.

Your instructions:
- Implementation Spec: {{SPEC_PATH}}
- Branch: {{BRANCH}}
- Target: Create a PR when complete

Read the Implementation Spec carefully, especially the "Developer Instructions" section.

Execute the implementation:
1. Follow the step-by-step order exactly
2. Do not skip steps or reorder without reason
3. If something is ambiguous, STOP and ask—don't guess
4. Commit incrementally as specified in the instructions
5. Write tests as specified before creating PR

When complete:
- Create a PR to the main branch
- Title: "{{VERSION}}: [brief description]"
- Description: Summarize what was implemented, reference the spec
- Request review

Do NOT:
- Add features not in the spec (scope creep)
- Skip tests to save time
- Make architectural decisions—those were already made
- Merge your own PR

If you encounter blockers or the spec is unclear, stop and report. 
Better to ask than to guess wrong.
```

### Code Reviewer Prompt
```
{{TOOL_NAME}} has implemented {{VERSION}} per the implementation spec.

Ultrathink and review this PR: {{PR_URL}}

Evaluate:
1. How well is it implemented based on our plan? Cite specific spec sections.
2. What's missing that should have been included?
3. What would YOU have done differently? (Don't just validate—take a position.)

For each significant code block, map it to the spec section it implements.

Format:
## Spec Compliance
[Section X.X] ✓ Implemented correctly
[Section X.X] ⚠ Deviation: [explain]
[Section X.X] ✗ Missing

## What I Would Have Done Differently
[Your independent technical judgment—not just validation]

## Issues Found
[Severity: critical/medium/low]

## Recommendation
APPROVE / REQUEST CHANGES / BLOCK

Implementation Spec: {{SPEC_PATH}}
```

### Developer Response to Review Prompt
```
Fellow architects have reviewed your PR: {{PR_URL}}

Their feedback is in the PR comments. Read all comments carefully.

For each piece of feedback:
1. If you agree: implement the fix
2. If you disagree: explain why in a reply comment, then wait for my decision
3. If you're unsure: ask for clarification in a reply comment

After addressing all feedback:
- Push your changes to the same branch
- Summarize what you changed in a comment

Do NOT:
- Ignore feedback without addressing it
- Make changes beyond what was requested
- Close or merge the PR yourself

Implementation Spec (for reference): {{SPEC_PATH}}
```

### Developer Iteration Prompt (Multiple Review Rounds)
```
This is round {{ROUND_NUMBER}} of review for {{PR_URL}}.

Previous round feedback has been addressed. New comments have been added.

Read the NEW comments only (since your last push). Address them following the same rules:
- Agree → implement
- Disagree → reply with reasoning
- Unsure → ask for clarification

Focus on the delta. Don't re-review old resolved threads.

Push changes when done and summarize what changed.
```

### Code Reviewer Re-Review Prompt
```
Developer has addressed your feedback on PR: {{PR_URL}}

New commits have been pushed. Review the changes.

For each piece of your original feedback:

1. **Addressed adequately**: Resolve the comment thread
2. **Not addressed / disagree with fix**: Reply explaining why, be specific
3. **New concerns**: If the changes introduced new issues, comment on them

Focus on the DELTA—new commits since your last review. Don't re-review unchanged code.

Output:
## Resolved
[Which of your concerns are now addressed]

## Still Outstanding  
[Concerns that weren't adequately fixed]

## New Concerns
[Any issues introduced by the changes]

## Verdict
APPROVE / REQUEST CHANGES (with clear asks) / BLOCK (with reasons)

Implementation Spec (for reference): {{SPEC_PATH}}
```

### Adversarial Reviewer Prompt
```
You are a security-minded engineer whose job is to FIND PROBLEMS in {{VERSION}}.

Your default stance is to reject. You are looking for reasons this code should NOT be merged.

Specifically hunt for:
1. Edge cases the happy path ignores
2. What happens with malformed input?
3. What happens under load?
4. What happens if external services fail?
5. Security: injection, auth bypass, data leaks
6. Race conditions or state management bugs
7. Things that will break in production but work in dev

You have access to:
- Implementation spec: {{SPEC_PATH}}
- PR: {{PR_URL}}
- Other reviewers' approvals (be skeptical of these)

Your output format:
## Issues I Found
[list each with severity and exploitation scenario]

## Edge Cases Not Handled
[list]

## Questions the Other Reviewers Should Have Asked
[list]

## My Verdict
BLOCK (with reasons) / RELUCTANT APPROVE (I tried hard and couldn't break it)

Do NOT approve just because another reviewer did. Find your own problems.
```

### Synthesizer Prompt
```
You are preparing a decision summary for a non-technical product owner who will approve or reject this merge.

You have multiple code reviews. Your job is to synthesize them into a clear decision document.

CRITICAL RULES:
1. If reviewers DISAGREE on anything, you must flag it prominently—do not smooth over conflicts
2. If any reviewer found a security issue, it goes at the top
3. Compress, but don't lose information that would change the decision
4. The product owner will not read the raw reviews—if you miss something, they'll never see it

Output format:
## TL;DR
[One sentence: Is this ready to merge? Yes/No/Conditional]

## Reviewer Consensus
[What all reviewers agreed on]

## Conflicts Between Reviewers
[Any disagreements—this section is REQUIRED even if empty]

## Critical Issues (if any)
[Anything that should block merge]

## Open Questions for Your Decision
[Anything that needs the product owner's judgment call]

Reviews to synthesize:
[paste all reviews]
```

### Cross-Synthesis Review Prompt (Anti-Sycophancy Check)
```
Claude, Gemini, and GPT, acting as fellow architects, have each produced 
synthesis summaries of the code reviews for {{VERSION}}.

Their syntheses are in: {{REVIEW_PATH}}

Ultrathink and review their syntheses. Consider:
- What did the other synthesizers miss or compress away?
- Where do they disagree, and who's more right?
- What would change the product owner's decision that got lost?

Be open-minded while keeping your integrity. Don't defer to consensus if you 
think the consensus is wrong.

Output:
## Key Disagreements Between Syntheses
[What they saw differently]

## Information Lost in Compression
[Things in raw reviews that all syntheses dropped but matter]

## My Independent Assessment
[Your own synthesis, informed by but not deferring to the others]
```

---

## Session Management (Ontos Integration)

Your decisions need to survive sessions. Integrate Ontos commands into your workflow.

### Start of Session
```
"Activate Ontos"
```
Agent confirms what context it loaded. Verify it has the right layers for the task.

### After Significant Decisions
- Approved a major strategy change
- Merged a version
- Changed architectural direction

```
"Archive Ontos"
```
Decision rationale gets logged. Future agents (and future you) can understand why.

### After Code Migrations
When you rewrite atoms (e.g., Streamlit → Next.js):
```
"Maintain Ontos"
```
Rebuilds context map. Strategy and product layers stay intact; atom references update.

### Session Log Location
Archive logs live in: `.ontos-internal/session_logs/`

Review these periodically. They're your project's institutional memory.

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

### Questions for LLM Colleagues: Metrics Integration

The goal is to track outcomes, not just process. We need to measure whether this workflow actually produces good results, not just whether it runs smoothly.

**Ask your LLM colleagues:**

```
I want to track metrics for my development workflow. Options:

A) Extend Ontos session log schema
   - Add fields to the YAML frontmatter (spec_rounds, review_rounds, bugs_found, etc.)
   - Modify ontos_end_session.py to prompt for these
   - Metrics live alongside session narrative

C) Add a Retrospective section to existing log template
   - New markdown section in the log template
   - Filled in manually or via prompt at session end
   - No schema changes needed

Questions:
1. Given Ontos's existing schema (id, type, status, event_type, concepts, impacts), 
   which approach fits better architecturally?

2. What specific metrics would be most valuable to track?
   - Spec revision rounds (before code)
   - Code review rounds (after code)
   - Bugs found in review vs. post-merge
   - Time from spec start to merge
   - Others?

3. How should these metrics roll up? Per-version summaries? Trends over time?

4. Should this integrate with decision_history.md or stay separate?

Be specific about implementation. I need a concrete recommendation, not options.
```

---

*This is a living document. Update it as you learn.*
