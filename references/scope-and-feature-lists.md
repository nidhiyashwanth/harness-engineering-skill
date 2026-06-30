# Scope control: WIP = 1 and the feature list primitive

*Source: Learn Harness Engineering, Lectures 07 (bound scope) and 08 (feature lists).*

## The failure: too much active scope (Lecture 07)

Agents overreach because nearby work feels cheap. A broad prompt like "add authentication" invites
the agent to touch models, routes, frontend, mail, hashing, error handling, and tests at once — and
the result is **lots of code, little verified behavior**. The harness has to make scope *expensive*
until the current task is verified.

**Why it's mathematical, not just stylistic:** with context capacity `C` and `k` activated tasks,
each task gets ~`C/k` reasoning budget. Below the depth needed to finish, *every* task ends partially
implemented. The two failures reinforce each other:

```
overreach → diluted attention → under-finish → next-session drag → more overreach
```

## The fix: WIP = 1

The workflow that actually finishes:

1. **Pick exactly one task** — move one unit from queue to active; everything else stays parked.
2. **Execute only that task** — no "while I'm here" refactors, adjacent features, or speculative cleanup.
3. **Run completion evidence** — done = behavior passes an executable check, not "code was written."
4. **Commit and unlock next** — only passing work releases the next item.

`QUEUE → ONE ACTIVE ITEM → E2E VERIFY → CHECKPOINT → NEXT`

### Vocabulary
- **Overreach** — activating more work units than can be completed in one verified session.
- **Under-finish** — activated tasks outnumber tasks that pass end-to-end verification.
- **WIP limit** — a hard cap on in-flight work; for agents, default to **WIP = 1**.
- **Completion evidence** — the executable proof needed to move a task from `active` to `passing`.
- **Scope surface** — a repo-visible task graph with states `not_started / active / blocked / passing`.
- **Completion pressure** — harness rules that force finishing the current task before starting another.

### How to draw the boundary
- **Enforce one active feature** in `AGENTS.md` / `CLAUDE.md`: one feature at a time; next only after
  verification; do not refactor feature B while implementing feature A.
- **Define completion evidence** for every task — the command, assertion, or check that proves it.
- **Externalize the scope surface** in Markdown or JSON so future sessions know what's active/blocked/passing.
- **Track VCR** (Verified Completion Rate = verified / activated). **Block new activation when VCR < 1.0.**

Evidence: an unconstrained session activated 5 features at **20% E2E pass**; WIP = 1 hit **100% E2E
pass** in session 1 and **87.5%** completion by session 4.

---

## The feature list as a harness primitive (Lecture 08)

A feature list is **not a memo** — it is the shared data structure that tells the **scheduler** what
to run, the **verifier** what counts as done, and the **handoff** what state exists. Agents do not
share your definition of done; without a feature list they stop when code "looks plausible."

**Memo mode** ("did auth, cart mostly done, still need payments") forces the next session to *infer*
what passed. **Primitive mode** ("F01–F05 passing; F06 active; F07–F10 not_started") lets it resume
from the next unresolved item without re-implementing finished work.

### Every row is a triple
A row is executable by the harness only if all three fields are present:

- **Behavior** — what to build, e.g. `POST /cart/items` with `product_id` + `quantity` returns 201.
- **Verification** — the exact command / test / workflow that must pass before completion is allowed.
- **State** — one of `not_started`, `active`, `blocked`, `passing`.

### It's a state machine — and the harness owns transitions
```
not_started → active → (verify) → passing
                  ↘ blocked (dependency / environment)
```
**Pass-state gating:** `agent requests → harness verifies → state updates`. The agent may work a row
and *request* verification, but **the only path from `active` to `passing` is a successful
verification command with attached evidence.** The agent cannot mark its own work passing.

### Why a primitive beats a document
Documents can be ignored; primitives are system inputs — like a database constraint, the list
*constrains* what can happen. Four consumers read the same table:
- **Scheduler** — picks the next `not_started` feature.
- **Verifier** — runs that row's check and controls pass transitions.
- **Handoff reporter** — generates session summaries from the state table.
- **Progress tracker** — tallies state distribution and remaining pressure.

### Calibrate granularity to one session
- **Too broad** — "Implement the shopping cart" hides several behaviors; unlikely to finish cleanly.
- **Right size** — "User can add items to cart" — implementable and verifiable in one session.
- **Too narrow** — "Create the name field on the Cart model" — overhead without user-visible completion.

### Minimal structured form (see `assets/feature-list.template.json`)
```json
{
  "id": "F03",
  "behavior": "POST /cart/items returns 201",
  "verification": "curl ... | jq .status == 201",
  "state": "active",
  "evidence": "commit abc123, test output log"
}
```

Put the feature-list location, WIP limit, and pass-gate policy into the agent instructions. Evidence:
startup inference **20 min → 3 min**, **45%** higher completion, **0** duplicate builds.

**Takeaway:** Do less, but finish — one active task, executable evidence, then the next.
