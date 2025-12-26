# Session Variables

Fill in these values for your current work session. Give this file to an LLM along with the Generator Prompt.

---

## Current Session

```
PROJECT:        
VERSION:        
PREV_VERSION:   
BRANCH:         
PR_URL:         
SPEC_PATH:      
PREV_SPEC_PATH: 
REVIEW_PATH:    
TOOL_NAME:      
ROUND_NUMBER:   
```

---

## Variable Descriptions

| Variable | Description | Example |
|----------|-------------|---------|
| `PROJECT` | Project name | `Project Ontos` |
| `VERSION` | Current version being built | `v2.9.4` |
| `PREV_VERSION` | Previous version | `v2.9.3` |
| `BRANCH` | Git branch name | `feature/config-merge` |
| `PR_URL` | Pull request URL (leave blank if no PR yet) | `https://github.com/user/repo/pull/42` |
| `SPEC_PATH` | Path to current implementation spec | `docs/VERSION_2_9_4_SPEC.md` |
| `PREV_SPEC_PATH` | Path to previous version's spec | `docs/VERSION_2_9_3_SPEC.md` |
| `REVIEW_PATH` | Directory where reviews are stored | `reviews/v2.9.4/` |
| `TOOL_NAME` | Coding tool being used for development | `Antigravity (Claude Opus 4.5)` |
| `ROUND_NUMBER` | Current review/iteration round (for multi-round prompts) | `2` |

---

## Example (Filled In)

```
PROJECT:        Project Ontos
VERSION:        v2.9.4
PREV_VERSION:   v2.9.3
BRANCH:         feature/config-merge
PR_URL:         https://github.com/ohjona/Project-Ontos/pull/47
SPEC_PATH:      docs/VERSION_2_9_4_SPEC.md
PREV_SPEC_PATH: docs/VERSION_2_9_3_SPEC.md
REVIEW_PATH:    reviews/v2.9.4/
TOOL_NAME:      Antigravity (Claude Opus 4.5)
ROUND_NUMBER:   1
```

---

*Use with [orchestrator-handbook.md](orchestrator-handbook.md) (Generator Prompt) and [llm-development-playbook.md](llm-development-playbook.md) (Prompts Library).*
