# Foundations: capability vs. reliability, and the diagnostic loop

*Source: Learn Harness Engineering, Lectures 01–02.*

## The core distinction

**Model capability** and **execution reliability** are different things. As of late 2025 the
strongest coding agents reach only ~50–60% on SWE-bench Verified — and real work is messier than a
curated benchmark (vague specs, implicit conventions, missing tests, state spread across sessions).
A higher benchmark number does not buy reliability on *your* repo.

The decisive evidence is the same-model comparison: an identical Opus 4.5 prompt produced a broken
game in a 20-minute / $9 bare run, and a fully playable one in a 6-hour / $200 run with a
planner + generator + evaluator harness. **The model did not change — the harness did.** Treat this
as the governing intuition for all harness work.

## What counts as "harness"

> If it is not model weights, it is harness.

A prompt file is *not* a harness. It is text the agent reads; it cannot run tests, preserve state,
control the environment, or close a feedback loop by itself. A harness is the five subsystems working
together so the agent can **act, observe, recover, and improve**:

1. **Instructions** — overview, stack, run commands, hard constraints, links to deeper docs.
2. **Tools** — enough shell, file, and test access to do real work, under least privilege.
3. **Environment** — self-describing dependencies, versions, services (Docker / devcontainers).
4. **State** — progress files and commits so long tasks survive session boundaries.
5. **Feedback** — explicit verification commands: tests, typecheck, lint, full checks.

Missing any subsystem makes the agent awkward to use. **Feedback typically has the lowest investment
and highest return — improve it first.**

## Where agents actually get stuck (these are harness failures first)

- **Vague requirements** — "add search" forces guesses about scope, pagination, highlighting.
- **Implicit conventions** — rules in Slack or someone's head are invisible to the agent.
- **Incomplete setup** — the agent burns context on dependency and version conflicts.
- **No verification** — without executable checks, the agent declares done when it *feels* done.
- **State loss** — long tasks fail when each session rediscovers the project from scratch.

## Vocabulary for debugging (use these labels instead of "try a better model")

- **Capability gap** — benchmark performance vs. real-world task performance.
- **Harness-induced failure** — the model *could* do the task, but the environment is structurally defective.
- **Verification gap** — distance between the agent's confidence and actual correctness.
- **Definition of Done** — completion criteria verified by command (tests, lint, typecheck, build).

## The diagnostic loop (apply on every failure)

```
Execute  →  Observe failure  →  Attribute to a layer  →  Fix that harness layer  →  Rerun
```

1. **Attribute** the failure to exactly one subsystem: task spec, context, environment, verification, or state.
2. **Write a Definition of Done** so completion is executable, not subjective.
3. **Add / fix `AGENTS.md`** with stack, conventions, and verification commands at the repo root.
4. **Keep a failure log** — after a few rounds the recurring bottleneck is obvious; invest there.

## Ablation: how to find the bottleneck empirically

Hold the model and task fixed, **remove one subsystem at a time**, and measure the drop in success.
The subsystem whose removal hurts most is where your harness is currently carrying the load — and
often where the next improvement pays off. Ablation supports diagnosis; root-causing a persistent
bottleneck still needs the failure log.

## Verification command block (make this explicit early)

```
Tests:             pytest tests/ -x
Type check:        mypy src/ --strict
Lint:              ruff check src/
Full verification: make check
```

## Evidence to anchor expectations

- A TypeScript + React team improved the *same* GPT-4o from **20% → 60% → 80% → 80–100%** by adding,
  in order: `AGENTS.md` (stack/conventions/decisions), verification (`yarn test && yarn lint && yarn
  build`), then progress-state templates. README-only was the 20% floor.
- The FastAPI team cut context spent exploring (was ~40%) and turned flaky runs into three-for-three
  successes by adding `AGENTS.md`, verification commands, and ADRs — **~60% better context
  efficiency, same model.**
- OpenAI's experiment: three engineers, ~1M lines, 1,500 PRs (~3.5 PRs/person/day) — they wrote no
  code themselves and strengthened the harness whenever something failed.

**Takeaway:** Do not swap the model first. Every failure is a signal that one harness layer needs repair.
