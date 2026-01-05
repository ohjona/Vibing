# LLM Development Playbook

A lightweight system for solo developers to build software with LLM agents.

## What This Is

| File | Audience | Purpose |
|------|----------|---------|
| `llm-development-playbook.md` | LLMs | Prompts, roles, constraints |
| `orchestrator-handbook.md` | You | Workflow, emergencies, escalation |
| `README.md` | Both | You are here |

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

**Handoffs:** 6-8 per feature
**Your decisions:** 2 gates

## Quick Start

1. Write your requirements
2. Copy the Architect prompt from the playbook, fill in your details
3. Send to critics in parallel, consolidate, revise once
4. Gate 1: Proceed to development?
5. Developer implements, reviewers review in parallel
6. Gate 2: Merge?

## Key Concepts

- **Parallel Fan-Out**: Critics and reviewers work simultaneously, not sequentially
- **One Revision Cap**: If major issues persist after one revision, scope is wrong
- **Model Diversity**: Different models for Architect, Critics, Developer, Reviewers
- **Side with Pessimist**: Adversarial concerns win by default
- **ComplexityFlag**: Developer can refuse over-engineered specs

## Complexity Constraints

- **YAGNI**: No generic interfaces until 3+ implementations
- **Boring Tech**: SQLite > Postgres, monolith > microservices
- **Cut List**: Every spec must defer non-essential features
- **Deletion Test**: If nothing breaks when removed, don't build it

---

*Built for solo developers who want to ship, not babysit.*
