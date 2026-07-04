---
name: harness-engineering
description: >-
  Build and improve the "harness" around a coding agent so a capable model reliably finishes real
  tasks in a repository: the instructions, tools, environment, state, and feedback that live outside
  the model's weights. Use when the user wants to make a repo agent-ready or
  agent-friendly; set up or refactor an AGENTS.md / CLAUDE.md; add verification commands, a Definition
  of Done, or completion gates; track work with a feature list, PROGRESS.md, or DECISIONS.md; diagnose
  why an AI coding agent keeps failing, overreaching, stalling across sessions, or "declaring done"
  when it isn't; prepare a project for autonomous or long-running agentic coding; or design a
  planner/generator/evaluator workflow. Trigger even when the user doesn't say
  "harness": phrases like "the agent keeps breaking my build", "Claude forgets context
  between sessions", "make my repo work better with Cursor/Codex/Claude Code", "set up rules for the
  AI", or "why does the AI say it's done when it's not" all indicate harness work.
---

# Harness Engineering

## What a harness is, and why this skill exists

A **harness** is the engineering infrastructure around a model — everything *outside the model
weights* that turns raw capability into reliable execution. When a capable agent fails on a real
task, the cause is almost always a harness defect, not a model deficit. The same model that ships
broken features under a bare prompt ships working ones under a good harness. So the core move of
this skill is: **don't reach for a better model first — find the failing harness layer and repair
it.**

A harness has **five subsystems**. Diagnose and build along these axes:

| Subsystem | What it provides | Typical artifacts |
|-----------|------------------|-------------------|
| **Instructions** | Orientation, run/verify commands, hard constraints, routes to detail | `AGENTS.md` / `CLAUDE.md`, topic docs |
| **Tools** | Enough shell/file/test access to do real work (least privilege) | shell, file ops, test runners |
| **Environment** | Self-describing deps, versions, services | `Makefile`, devcontainer, Docker, lockfiles |
| **State** | Continuity so long tasks survive session boundaries | `PROGRESS.md`, `DECISIONS.md`, commits, feature list |
| **Feedback** | Executable verification looped back to the agent | tests, lint, typecheck, E2E, completion gates |

**Feedback is usually the lowest-investment, highest-return subsystem — improve it first** if you
have to pick one. A repo with even a `README` but no executable verification lets the agent declare
"done" by feeling rather than by evidence.

## How to use this skill

Two entry points. Figure out which one the user is in, then route to the relevant references —
**read a reference file only when the work calls for it** (this skill is a router, not an
encyclopedia; loading everything wastes the very context budget good harnesses protect).

### Entry point A — Diagnosing a failing agent

When the user describes a *symptom* ("it keeps breaking the build", "forgets everything next
session", "says it's done but it isn't", "touches a hundred files and finishes nothing"), don't
guess at a fix. Attribute the symptom to a subsystem, then repair that layer:

| Symptom the user reports | Most likely failing layer | Read |
|--------------------------|---------------------------|------|
| Agent guesses at scope; misreads vague requirements | Instructions (no Definition of Done) | `references/foundations.md`, `references/verification.md` |
| Agent uses wrong conventions / can't find how to run or test | Instructions / Environment (repo isn't the source of truth) | `references/repository.md`, `references/instruction-files.md` |
| `AGENTS.md` is huge and the agent ignores rules | Instructions (bloat) | `references/instruction-files.md` |
| Each new session rediscovers the project; reasoning lost | State (no handoff) | `references/continuity.md` |
| First session burns time on setup; nothing runs | Environment (no initialization phase) | `references/continuity.md` |
| Agent starts everything, finishes nothing | State (no scope limit) | `references/scope-and-feature-lists.md` |
| "Done" claimed but runtime/integration is broken | Feedback (no completion gate) | `references/verification.md` |
| Unit tests green but the real flow fails | Feedback (no E2E) | `references/verification.md` |
| Can't tell what actually ran or why a retry failed | Feedback / State (no observability) | `references/observability.md` |
| Repo gets messier every session | State (no clean-exit discipline) | `references/continuity.md` |

Always **attribute before fixing**, and keep a short failure log — after a few rounds the recurring
bottleneck becomes obvious, and that's the layer worth the most investment.

### Entry point B — Setting up or upgrading a harness from scratch

Run the build workflow below. Don't try to install all five subsystems at maximum strength at once —
add them incrementally and re-measure, because each layer's payoff is only visible once the layer
below it works.

## The build workflow

**Step 0 — Run the fresh-session test (always start here).**
Imagine a brand-new agent session with *only* the repository contents. Can it answer: (1) What is
this system? (2) How is it organized? (3) How do I run it? (4) How do I verify it? (5) Where are we
now? Each unanswered question is a blank spot the agent will fill with a guess. The build steps below
close these gaps in priority order. Details: `references/repository.md`.

**Step 1 — Make the repo the source of truth.** Anything the agent can't see (Slack, Confluence,
Jira, someone's head) effectively does not exist. Put decision-useful knowledge *near the code* it
governs. → `references/repository.md`

**Step 2 — Write a routing `AGENTS.md` (or `CLAUDE.md`), not an encyclopedia.** A 50–200 line entry
file: one-line overview, first-run commands, ≤15 global hard constraints, and links to focused
50–150 line topic docs revealed on demand. → `references/instruction-files.md`
(Template: `assets/AGENTS.md.template`)

**Step 3 — Make feedback executable.** List the exact commands to test, typecheck, lint, and fully
verify. Write a **Definition of Done** for each task that a command can check — not a subjective
feeling. → `references/verification.md`

**Step 4 — Establish the environment / initialization phase.** Before any feature work, ensure deps
install, the app starts, one example test passes, and there's a clean committed baseline. This
feature-free phase produces *reliability, not features*. → `references/continuity.md`

**Step 5 — Add state & continuity.** `PROGRESS.md` (state, checks, blockers, ordered next actions),
`DECISIONS.md` (choice, why, rejected alternative), atomic commits. Define clock-in / clock-out
routines and a clean-exit checklist. → `references/continuity.md`
(Templates: `assets/PROGRESS.md.template`, `assets/DECISIONS.md.template`)

**Step 6 — Bound scope with WIP = 1 and a feature list.** One active task to verified completion
before the next. Track work as a feature list where every row has *behavior + verification + state*,
and the harness — not the agent — owns the transition to `passing`. → `references/scope-and-feature-lists.md`
(Template: `assets/feature-list.template.json`)

**Step 7 — Gate completion and verify end-to-end.** The worker can't self-pass. Build a three-layer
gate (static → runtime → end-to-end) and require a full-pipeline run for cross-component work.
Consider a planner/generator/evaluator split so an independent checker judges "done". →
`references/verification.md`

**Step 8 — Make the runtime observable.** Capture runtime signals (what happened) and process
artifacts (why it should pass): traces, sprint contracts, evaluator rubrics. → `references/observability.md`

**Step 9 — Keep it clean and prune it.** Clean state is a required *exit* condition, not optional
housekeeping. Periodically simplify the harness: as models improve, remove constraints that no longer
earn their keep (disable one, benchmark, keep/remove/replace). → `references/continuity.md`

## Non-negotiable principles (the short list)

These recur across every layer; when in doubt, fall back to them:

1. **Fix the harness, not the model** — attribute every failure to a subsystem first.
2. **If the agent can't see it, it doesn't exist** — the repo is the system of record.
3. **The entry file routes; it does not accumulate** — protect the context budget.
4. **Persist the why, the state, and the next move** — every session must hand off cleanly.
5. **Do less, but finish** — WIP = 1; make scope expensive until the current task is verified.
6. **Done is evidence, not a feeling** — the harness owns the verdict; the worker cannot self-pass.
7. **Only a full-pipeline run proves system behavior** — unit-green is necessary, never sufficient.

## Reference files

Read these as the work requires — each is self-contained and maps to specific course lectures:

- `references/foundations.md` — capability vs. reliability, the five subsystems, the diagnostic loop, ablation. (Lectures 01–02)
- `references/repository.md` — repo as source of truth, the fresh-session test, ACID state thinking. (Lecture 03)
- `references/instruction-files.md` — splitting instructions, the routing `AGENTS.md`, instruction metadata. (Lecture 04)
- `references/continuity.md` — context handoff, the initialization phase, and clean-exit discipline. (Lectures 05, 06, 12)
- `references/scope-and-feature-lists.md` — WIP = 1 and the feature list as a harness primitive. (Lectures 07–08)
- `references/verification.md` — Definition of Done, completion gates, end-to-end testing, worker–checker separation. (Lectures 09–10)
- `references/observability.md` — runtime signals, process artifacts, sprint contracts, evaluator rubrics. (Lecture 11)

Templates to copy into a target repo live in `assets/`.
