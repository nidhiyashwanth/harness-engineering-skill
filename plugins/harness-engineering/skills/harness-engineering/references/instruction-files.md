# Instruction files: route, don't accumulate

*Source: Learn Harness Engineering, Lecture 04.*

## The failure of the giant instruction file

The natural drift is a feedback loop: `mistake → add a rule → temporary fix → more bloat → repeat`,
until `AGENTS.md` is 600+ lines of "everything, all at once." That fails in five ways:

- **Context budget** — 10–20K tokens of instructions crowd out code, tool output, and reasoning
  (instructions can eat 10–15% of the window).
- **Lost in the middle** — critical rules buried in long text are less likely to be retrieved and followed.
- **Priority conflicts** — hard constraints, guidelines, and old lessons all look equally important.
- **Maintenance decay** — adding feels free; deleting feels risky; signal-to-noise steadily declines.
- **Contradictions** — rules written at different times diverge, leaving the agent to choose.

The root problem: when constraints and suggestions look identical, the agent **can't tell what
matters** — it cannot distinguish red lines from preferences.

## The principle: the entry file is a router, not an encyclopedia

Load **broad orientation first, detailed topic rules only on demand** ("reveal on demand"):
frequent guidance stays nearby, occasional guidance is tucked into topic docs, never-used guidance is
deleted. The goal is high **signal-to-noise ratio** — the share of loaded instructions that actually
matter to the current task.

## The target architecture

```
AGENTS.md  (50–200 lines)
├── Project overview          # one or two sentences
├── First-run commands        # setup, test, full verification
├── Hard constraints          # ≤15, global, non-negotiable, short, testable, loaded first
└── Routing map               # when to read each topic doc
    ├── API      → api.md       (50–150 lines · Source · Applicability · Expiry)
    ├── Database → database.md   (50–150 lines · Source · Applicability · Expiry)
    └── Testing  → testing.md    (50–150 lines · Source · Applicability · Expiry)
```

**Hard-constraints discipline:** no more than 15, loaded first, specific and actionable, and
referenced by the topic docs that depend on them. If a "constraint" can't be stated short and
testable, it's probably guidance — put it in a topic doc.

**Keep truth near the code:** type definitions, interfaces, comments, and configuration explanations
belong beside the implementation, not duplicated in the entry file.

## Instruction metadata

Give each topic doc a small header so stale rules can be found and retired:

- **Source** — where the rule came from (an incident, an ADR, a convention).
- **Applicability** — which modules / task types it governs.
- **Expiry** — when to re-check it (or a condition that makes it obsolete).

Durable behavior belongs in **tests**, not narrative. Convert historical "we once hit a bug, so
always…" notes into an executable check, or delete them.

## The splitting procedure

1. **Project overview** — one or two sentences on what the project is.
2. **First-run commands** — setup, test, full verification.
3. **≤15 global hard constraints** — only rules that truly apply everywhere.
4. **Topic links + conditions** — say *when* each document must be read.
5. **50–150-line topic docs** — focused guidance, near the relevant work.
6. **Instruction metadata** — record source, applicability, expiry.

## Refactor checklist (for an existing bloated file)

- [ ] Audit instruction SNR — mark each rule's relevance across ~five common task types.
- [ ] Split into 3–5 topic docs — keep the entry file under ~100–200 lines.
- [ ] Convert historical notes to tests — or delete them.
- [ ] Rerun the same task set — compare success and compliance before/after.
- [ ] Review regularly — remove stale, redundant, and contradictory rules.

## Evidence

Splitting a 600-line entry file down to ~80 lines (with topic docs) moved task success **45% → 72%**
and security compliance **60% → 95%** on the lecture's example.

**Takeaway:** The entry file is a router, not an encyclopedia.
