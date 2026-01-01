# LLM Development Playbook

A lightweight system for solo developers to build software with LLM agents.

## What This Is

Five files that help you orchestrate multiple LLMs:

| File | Audience | Purpose |
|------|----------|---------|
| `session-template.md` | Both | Copy per feature — tracks progress, provides prompts |
| `llm-development-playbook.md` | LLMs | Role definitions, prompt reference |
| `orchestrator-handbook.md` | You | Decisions, emergencies, escalation |
| `session-variables.md` | Both | Variable reference |
| `CHANGELOG-v1.2.md` | You | Implementation decisions and rationale |

## How It Works

You are the orchestrator. LLMs are your team. You make two decisions per feature.

```
You write requirements
    ↓
Architect writes spec
    ↓
Critics review IN PARALLEL (Claude + Gemini + GPT)
    ↓
Consolidate → Architect revises ONCE
    ↓
★ YOU DECIDE: Proceed to Development?
    ↓
Developer writes code
    ↓
Reviewers review IN PARALLEL (Standard + Adversarial)
    ↓
Consolidate feedback
    ↓
★ YOU DECIDE: Merge?
    ↓
Done
```

**Handoffs:** 6-8 per feature (down from 12-16 in v1.1)
**Your decisions:** 2 clear gates

## Philosophy: Parallel Over Sequential

v1.2 replaces sequential convergence loops with parallel fan-out:
- Same number of perspectives
- ~50% fewer handoffs
- Model diversity preserved (different models review in parallel)

## Quick Start

1. Copy `session-template.md` to `sessions/[version].md`
2. Fill in variables at top
3. Follow the template step by step — it provides ready-to-copy prompts
4. Make decisions at the two gates

## Key Concepts

- **Parallel Fan-Out**: Critics and reviewers work simultaneously, not sequentially
- **One Revision Cap**: If major issues persist after one revision, scope is wrong
- **Model Diversity**: Different models for Architect, Critics, Developer, Reviewers
- **Side with Pessimist**: Adversarial concerns win by default
- **ComplexityFlag**: Developer can refuse over-engineered specs

## Complexity Constraints (Enforced Throughout)

- **YAGNI**: No generic interfaces until 3+ implementations
- **Boring Tech**: SQLite > Postgres, monolith > microservices, in-process > networked
- **Cut List**: Every spec must defer non-essential features
- **Deletion Test**: If nothing breaks when removed, don't build it

## Files

```
├── session-template.md           # Copy per feature (the main workflow doc)
├── llm-development-playbook.md   # Reference for LLMs
├── orchestrator-handbook.md      # Reference for you
├── session-variables.md          # Variable definitions
├── CHANGELOG-v1.2.md             # Implementation decisions and rationale
└── README.md                     # You are here
```

## Dependencies

Works with [Project Ontos](https://github.com/ohjona/Project-Ontos) for context persistence between agents. Not required, but recommended.

---

*Built for solo developers who want to ship, not babysit.*
