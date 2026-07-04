# Verification: completion gates and end-to-end testing

*Source: Learn Harness Engineering, Lectures 09 (completion gates) and 10 (end-to-end testing).*

This is the **Feedback** subsystem — the highest-leverage place to invest. Two ideas: the agent's
confidence is not evidence (so the harness must own the verdict), and only a full-pipeline run proves
system behavior.

---

## Done is evidence, not a feeling (Lecture 09)

### The slippery slope
"Looks correct" quietly becomes "done":
1. **Code looks fine** — syntax valid, local logic plausible.
2. **Partial tests pass** — unit tests green, often with mocks and isolated state.
3. **Runtime is skipped** — startup, config, migrations, real services stay unobserved.
4. **Agent says done** — confidence replaces evidence; defects move to the next session.

The lecture's password-reset case looks productive (schema, endpoint, email template, green unit
tests) but the email config is missing, the migration can fail halfway, and the full reset flow never
ran. **Feature complete = end-to-end verification passed, not "code is written."**

**Calibration bias:** model-reported confidence systematically exceeds actual quality (Guo et al.;
echoed by Anthropic's 2026 finding that agents over-rate their own work). The fix is **not** asking
the same agent to be more objective — it's an *independent* checker with real runtime tests.

### The three-layer completion gate
The agent can *request* a verdict; the harness *owns* it. Cost and information rise per layer, so stop
at the first failure:

```
Agent says done → L1 static → L2 runtime → L3 end-to-end → done
```
- **Layer 1 — Static / syntax:** lint, typecheck, parse, migration-parse. Necessary, never sufficient.
- **Layer 2 — Runtime behavior:** test execution, app startup, health checks, critical-path validation.
- **Layer 3 — System-level:** end-to-end tests, integration validation, user-scenario simulation with real side effects.

```markdown
## Definition of Done
- Feature complete = end-to-end verification passed
- Required levels: 1) unit tests pass  2) integration tests pass  3) E2E flow passes
- Do not proceed to level 2 if level 1 fails
- Do not proceed to level 3 if level 2 fails
```

### Vocabulary
- **Premature completion** — asserting completion while correctness specs remain unmet.
- **Termination criteria** — executable judgment conditions defined by the harness, not the worker.
- **Verification–validation gate** — verification checks specified behavior; validation checks
  system-level user requirements.
- **Runtime signals** — logs, process state, health checks, side effects, cleanup status.
- **Completion priority** — functional correctness first, performance second, style last.

**Refactoring before verification corrupts the verdict.** "While we're at it" work moves the boundary
between verified and unverified code. Block cleanup / optimization / style passes until the core
feature passes its functional checks.

### Worker–checker separation
Use distinct roles so an independent agent judges "done":
- **Planner** — expands requirements.
- **Generator** — implements feature by feature.
- **Evaluator** — click-tests and rejects weak work.

(The same split drove the 20-min/$9-broken vs. 6-hr/$200-playable game result.)

### Write repairable errors
Failure messages should tell the agent *what failed and where to look* so it can self-correct:
```
Bad:    Test failed

Better: POST /api/reset-password returned 500.
        Check that email service config exists in env vars.
        Template should be at templates/reset-email.html.
```
Catching defects *inside* the session this way saves **5–10×** the cost of post-hoc repair.

---

## Only a full pipeline run counts as real verification (Lecture 10)

**Unit tests prove isolated parts; E2E proves the parts still work after they're wired together.**
The defect lives *between* components. In the Electron export example, renderer, preload, and service
layers each pass their unit tests — and the first real click exposes wrong path format, stuck
progress, memory leakage, permission differences, and missing error propagation. Only the full path
`Click → Preload bridge → Service → File system → Exported file` observes the real system.

### Unit-test blind spots (by design — isolation is what makes them fast)
- **Interface mismatch** — renderer sends a relative path; preload expects absolute.
- **State propagation** — schema changes but cached ORM entries reflect the old shape.
- **Resource lifecycle** — file handles, DB connections, sockets leak across boundaries.
- **Environment dependency** — mocks hide config, latency, permissions, service availability.

### E2E changes how agents write code
When the agent knows the real flow will run, it stops optimizing for the fastest green check and
starts respecting integration pressure: it considers **component interactions**, follows
**architectural boundaries**, and implements **error paths**.

### Make architecture executable
Without architectural intent, a full-pipeline run only proves "the whole mess runs." Because agents
copy existing patterns, constraints must exist on day one. Encode them as checks:

```
Layered Domain Architecture:  Types → Config → Repo → Service → Runtime → UI
Rule: dependencies flow forward through fixed layers; cross-domain concerns enter through
explicit provider interfaces; any other dependency is forbidden and mechanically enforced.
```

**Promote review feedback into the harness** — a recurring review comment should become a permanent
automated check with a teaching error message:
```
ERROR: Found direct import of 'fs' in src/renderer/App.tsx:12
WHY:   Renderer process has no access to Node.js APIs for security
FIX:   Move file operations to src/preload/file-ops.ts and call via window.api.readFile()
```
Flow: review finds violation → add a check → error explains the repair → harness runs it
automatically → next violation fails immediately.

### How to apply it
- **Require E2E for cross-component work** — hierarchy: unit → integration → E2E when multiple components change.
- **Automate architecture rules** — convert "renderer must not touch the filesystem" into lint/test checks.
- **Write repair instructions** — what, why, and fix steps in every failure message.
- **Grow the harness from reviews** — every recurring defect category becomes a new permanent check.

Evidence: in the Electron case E2E caught all five system-level defects that unit tests missed; the
E2E phase cost 15s vs. the 2s unit phase — an acceptable tradeoff for agents.

**Takeaway:** Run the real path — UI to bridge to service to side effect — with architecture checks
that teach the repair.
