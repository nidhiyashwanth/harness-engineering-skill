# The repository as the single source of truth

*Source: Learn Harness Engineering, Lecture 03.*

## The premise

An agent sees only three inputs: **system prompts / task descriptions**, **repository file
contents**, and **tool execution output**. Slack history, Confluence pages, Jira tickets, and
hallway decisions are *invisible* unless written into the repo. So: **knowledge not in the repository
does not exist for the agent.** The larger this visibility gap, the higher the failure rate.

The repo must therefore become the **system of record** — authoritative for decisions, constraints,
execution state, and verification standards.

## The fresh-session test (run this first, every time)

Open a hypothetical new session with *only* repository contents. If it cannot answer these five
questions, the map has blank spots the agent will fill with guesses:

| Question | Where it should be answered |
|----------|------------------------------|
| What is this system? | `AGENTS.md` / `README` |
| How is it organized? | `ARCHITECTURE.md` (module-local) |
| How do I run it? | `Makefile` / scripts |
| How do I verify it? | test / lint / check commands |
| Where are we now? | `PROGRESS.md` / feature list |

Use this as the acceptance test for "is the repo agent-ready?"

## Properties of good repo knowledge

- **Near the code** — API constraints belong next to API code, not in a distant global archive.
  Type definitions, interfaces, comments, and config explanations stay beside the implementation; the
  agent sees them while working, so don't duplicate them in `AGENTS.md`.
- **Entry file routes** — `AGENTS.md` answers *what / how to run / how to verify* and points onward.
- **Minimal but complete** — keep only decision-useful knowledge, yet answer every fresh-session question.
- **Updated with code** — tie doc updates to implementation changes. **Stale docs are worse than
  missing docs** because they confidently send the agent the wrong way (knowledge decay).

## Map layout

A 50-line module-local architecture file beats a 500-page external document the agent can't see or
trust:

```
project/
├── AGENTS.md        # overview, commands, hard constraints, routes
├── src/
│   ├── api/
│   │   └── ARCHITECTURE.md     # responsibilities, interfaces, dependencies
│   └── db/
│       └── CONSTRAINTS.md      # explicit MUST / MUST NOT
├── PROGRESS.md      # done / active / blocked
└── Makefile         # setup, test, lint, check
```

## Manage agent state with ACID thinking

The database analogy gives a concrete standard for cross-session reliability:

- **Atomicity** — one logical operation = one commit; half-done work rolls back or stays uncommitted.
- **Consistency** — tests and lint define a consistent state; inconsistent states don't get committed.
- **Isolation** — concurrent agents avoid racing through shared branches or progress files.
- **Durability** — knowledge that must survive sessions lives in git-tracked files, not chat history.

## Transformation pattern (from the 30-microservice case)

A platform that kept architecture in Confluence/Slack/heads hit **70% human-intervention** once
agents arrived. The fix:

1. Root `AGENTS.md` — overview, stack versions, hard constraints.
2. Service-local `ARCHITECTURE.md` — responsibilities, interfaces, dependencies.
3. Centralized `CONSTRAINTS.md` — explicit MUST / MUST NOT.
4. Per-service `PROGRESS.md` — so a fresh session knows current state.

**Discovery cost** is the context budget burned to find one key fact; hidden knowledge leaves less
budget for the actual work. Lowering discovery cost is the whole point of drawing the map in the repo.
