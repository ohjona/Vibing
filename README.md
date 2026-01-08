# LLM Development Playbook

A lightweight system for solo developers to build software with LLM agents.

## Files

| File | Purpose |
|------|---------|
| `session-variables.md` | Fill in your values once per session |
| `llm-development-playbook.md` | All prompts (source of truth) |
| `orchestrator-handbook.md` | Generator Prompt + emergencies |

## How It Works

1. Fill in `session-variables.md`
2. Copy the **Generator Prompt** from `orchestrator-handbook.md` to your agentic CLI
3. LLM reads the playbook and outputs all prompts with your variables filled in
4. Copy each prompt to the appropriate model as needed
5. Make decisions at two gates: **Proceed to Development?** and **Merge?**

## Workflow

```
Generator Prompt → LLM outputs all prompts
    ↓
Architect → Spec
    ↓
3 Critics IN PARALLEL → Consolidate → Architect revises ONCE
    ↓
★ GATE 1: Proceed?
    ↓
Developer → PR
    ↓
2 Reviewers IN PARALLEL → Consolidate
    ↓
Architect Review Response → Developer Amendment → Final Review
    ↓
★ GATE 2: Merge?
```

## Key Concepts

- **Parallel Fan-Out**: Critics and reviewers work simultaneously
- **One Revision Cap**: If issues persist, scope is wrong
- **Verbatim Prompts**: LLM outputs prompts exactly as written, no improvisation
- **Side with Pessimist**: Adversarial concerns win by default
- **ComplexityFlag**: Developer can refuse over-engineered specs

---

*Built for solo developers who want to ship, not babysit.*
