# Observability: make the agent's runtime legible

*Source: Learn Harness Engineering, Lecture 11.*

## The premise

Reliability is an **evidence problem**. The **harness — not the agent's memory** — must preserve what
the system did and *why* the work should be accepted. Without it, every retry is a guess: an agent
can edit many files, run 20 minutes, and still not explain why tests fail or which critical paths
changed.

### Four costs of missing observability
- **"Looks correct"** — code review shows what was *written*; only runtime traces show what *ran*.
- **Mystical evaluation** — without rubrics, acceptance rests on implicit assumptions and inconsistent judgment.
- **Blind retries** — the agent patches nearby code instead of the root cause because failure context is gone.
- **Handoff cliff** — missing state forces the next session to re-diagnose, wasting **30–50%** of its time.

The dark-mode comparison: a vague loop (fuzzy spec, generator guesses, evaluator rejects with "it
doesn't feel right," 3–4 repeats) takes ~45 min for barely-acceptable output; an observable loop
(sprint contract + recorded runtime + evidence-citing rubric) takes ~15 min for high quality —
**3× efficiency**.

## Observability has two layers — adding logs is not enough

- **Runtime observability — "What did the system do?"** logs, traces, process states, health checks,
  data flow, resource usage, errors, side effects.
- **Process observability — "Why should this pass?"** plans, sprint contracts, scoring rubrics,
  exclusions, acceptance criteria, evaluator decisions.

`CONTRACT + SIGNALS → REVIEW → SPECIFIC VERDICT → NEXT FIX`

**Why the agent can't fix this by printing logs:** *unknown unknowns* (it only logs what it thinks
matters), *inconsistent formats* (each session logs differently, defeating systematic analysis), and
*process gaps* (sprint contracts and rubrics are structured harness artifacts, not print statements).

## Vocabulary
- **Task trace** — the complete decision-path record from task start to completion (like request tracing for agent work).
- **Sprint contract** — a short-term agreement defining scope, verification standards, and exclusions *before* coding starts.
- **Evaluator rubric** — structured scoring that turns "is it good?" into evidence-based dimensions.
- **Runtime signals** — objective system behavior: logs, traces, health, data flow, resources, exceptions.
- **Layered observability** — runtime and process evidence designed together so each reinforces the other.

## How to build it

1. **Collect runtime signals automatically** — lifecycle, critical paths, data flow, resources,
   exceptions, and full error context (not ad-hoc prints).
2. **Negotiate sprint contracts** — define scope / verification standards / exclusions before the
   generator writes code:
   ```yaml
   # Sprint Contract: Dark Mode Support
   Scope:        [theme toggle component, global CSS variables, dark mode tests]
   Verification: [visual regression passes, main E2E flow passes, no flash of unstyled content]
   Exclusions:   [no print styles, no third-party dark mode]
   ```
3. **Use evaluator rubrics** — score concrete dimensions; cite evidence, not vibes
   ("contrast 2.1:1, WCAG AA needs 4.5:1"):

   | Dimension | A | B | C | D |
   |-----------|---|---|---|---|
   | Code correctness | All tests pass | Main flow passes | Partial pass | Build fails |
   | Architecture | Fully compliant | Minor deviations | Obvious deviations | Serious violations |
   | Test coverage | Main + edge cases | Main flow only | Only skeleton | No tests |

4. **Standardize traces** — OpenTelemetry: session trace, task span, verification sub-spans, standard
   attributes — so analysis is consistent across sessions.

## Anthropic's three-agent DAW experiment (what "observable" buys)

Planner expands product context; generator works sprint by sprint; evaluator uses Playwright and
scores hard thresholds. Making each phase observable produced an auditable cost/time breakdown:

| Phase | Duration | Cost |
|-------|----------|------|
| Planner | 4.7 min | $0.46 |
| Build round 1 | 2 hr 7 min | $71.08 |
| QA round 1 | 8.8 min | $3.24 |
| Build round 2 | 1 hr 2 min | $36.89 |
| QA round 2 | 6.8 min | $3.09 |
| Build round 3 | 10.9 min | $5.88 |
| QA round 3 | 9.6 min | $4.06 |
| **Total** | **3 hr 50 min** | **$124.70** |

Roles: **Planner** turns a short requirement into product context + high-level design; **Generator**
negotiates a sprint contract, implements, self-evaluates, hands off; **Evaluator** uses Playwright,
checks UI/API/database behavior, and returns evidence-backed feedback.

**Takeaway:** Make the harness remember the evidence — runtime signals for *what happened*, process
artifacts for *why it passes*.
