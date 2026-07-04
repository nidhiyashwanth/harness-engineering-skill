# Continuity: handoff, initialization, and clean exit

*Source: Learn Harness Engineering, Lectures 05 (handoff), 06 (initialization), 12 (clean exit).*

This file covers the **State** subsystem across the session lifecycle: how to start a session
(initialization), how to survive context limits mid-task (handoff), and how to end a session
(clean exit). A new session should **inherit** the work, not rediscover it.

---

## Part 1 — Keeping context alive across sessions (Lecture 05)

### The failure sequence
1. **Finite window** — code, tool output, decisions, and conversation outgrow context capacity.
2. **Compaction / reset** — summarizing or starting clean both lose something.
3. **Lost "why"** — the code survives, but rejected alternatives and reasoning vanish.
4. **Rebuild cost** — time spent rereading, reconstructing decisions, rerunning checks.
5. **Drift** — each session inherits a slightly different understanding; deviations compound.

**Context anxiety:** near the limit, quality goes first — rushed finishes, skipped verification,
oversimplified solutions. Persisting state externally is what relieves the pressure.

### The handoff stack
- **`PROGRESS.md` (living state)** — current commit + checks; completed / in-progress; known issues
  and blockers; ordered next actions.
- **`DECISIONS.md` (reasoning)** — decision + date; why chosen; rejected alternative; remaining constraints.
- **Git commits (checkpoints)** — atomic units with clear "what and why" messages; easy rollback/diff.

> Persist the **why**, the **state**, and the **next move**. Verification results go in the repo, not
> in chat history — chat history does not survive a reset.

### When to hand off
- **Short task** — stay in one session if it fits comfortably in the window.
- **>60% of window used** — start preparing the handoff *before* context pressure changes behavior.
- **Model-specific** — choose compaction vs. reset based on the target model, not one universal rule.

### Evidence
Rebuild time **15 → 3 min** (~78% lower); feature completion **58% → 100%**; hidden defects **43% → 8%**.

---

## Part 2 — The initialization phase (Lecture 06)

**Prepare the runway before asking the agent to fly.** Initialization is a *feature-free* lifecycle
phase whose output is **reliable infrastructure, not business code**. Skipping it (the "mixed
session": build while preparing) means discovering environment gaps mid-feature, accumulating
unverified code, and re-discovering everything next session.

### The five-part initialization contract
1. **Runnable environment** — dependencies install; app starts cleanly; commands documented.
2. **Verified test framework** — runner configured; one example test passes; tests repeat locally.
3. **Readiness document** — start/test commands; project structure; current state.
4. **Ordered task breakdown** — at least three tasks; acceptance criteria; a success sequence.
5. **Clean git checkpoint** — everything committed; working tree clean; known-good baseline.

### Four readiness gates
**Can start** (env runs without friction) · **Can test** (≥1 test passes locally) · **Can see
progress** (tasks + criteria visible) · **Can pick up next steps** (a fresh agent knows where to begin).

### Acceptance checklist
- [ ] `make setup` succeeds from scratch.
- [ ] `make test` has one passing test.
- [ ] A fresh session knows how to run and test.
- [ ] Task breakdown contains at least three tasks.
- [ ] All initialization work is committed.

Prefer **templates over scratch**: a template encodes proven structure and turns standard
infrastructure into reusable knowledge. Evidence: **31%** higher completion, investment recovered in
**3–4 sessions**, **<3 min** session-two rebuild vs. ~60% more rebuild time for the mixed approach.

---

## Part 3 — Clock-in / clock-out routines

**Clock in (start):** pull latest repo state → read `PROGRESS.md` and `DECISIONS.md` → skim recent
commits/diffs → run build/tests/lint → confirm the one next action and continue.

**Clock out (end):** finish the current atomic unit → update state/checks/blockers/next actions →
record only *new* durable decisions → commit with a clear message → leave the repo clean and resumable.

**Handoff checklist:** `PROGRESS.md` reflects state/checks/blockers/next actions · `DECISIONS.md` has
every new durable choice + reason · verification results recorded (not in chat) · work committed as an
atomic checkpoint · the next session can identify one clear first action.

---

## Part 4 — Clean exit (Lecture 12)

**Entropy growth is the default**, and agents *copy existing patterns* — including messy ones. If a
session exits dirty, the next session spends its first ~30 minutes reconstructing what happened.
So: **the session is not done until the next session can start without cleaning up after it.**

### Clean-state exit check
`Feature work complete → Build passes? → Tests pass? → Record progress → Remove artifacts → Startup works?`
`ANY NO → FIX BEFORE EXIT → RERUN THE CHECK`

```markdown
## Session Exit Checklist
- [ ] Build passes (e.g. npm run build)
- [ ] All tests pass, including pre-existing ones (e.g. npm test)
- [ ] Feature list / PROGRESS.md updated
- [ ] No debug code, temp files, or stale TODOs remain
- [ ] Standard startup path works (e.g. npm run dev)
```

### Two cleanup modes
- **Immediate** — run the exit checklist at every session end. **Idempotent**: it can run repeatedly
  and safely reach the same end state.
- **Periodic** — a weekly cleanup session for structural drift, guided by a **quality document** (a
  living map of module health: verification, understandability, test stability, boundary compliance,
  conventions). Prioritize the lowest-scoring modules.

### Prune the harness itself
As models improve, some constraints become overhead. Periodically **disable one harness component,
run benchmark tasks** (completion quality, drift, retry count, defect escape rate), then **keep,
remove, or replace**: remove if results don't degrade; restore or lighten if they do. Useful harness
combinations don't disappear — they shift toward the next capability boundary.

### Evidence (twelve weeks)
Clean-state vs. no-cleanup: build pass **97% vs 68%**, test pass **95% vs 61%**, startup **9 min vs
60+ min**, stale artifacts **11 vs 103**. ~5 minutes of exit discipline prevents dozens of hours of
later chaos; good progress records cut diagnosis time **60–80%**.
