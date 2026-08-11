# Career Log

Durable record of capability transitions that were written into `profile.md`.

`profile.md` is a snapshot — it is rewritten in place, so it cannot retain the
fact that a transition happened. This file is that history.

## Contract

**Written by:** the weekly routine's PART B, and nothing else. A record is
appended only for a tier transition PART B **actually wrote into `profile.md`
during that run**, in the **same commit as the `profile.md` edit**. That
coupling is the audit: `git show` on any record displays the tier change that
justifies it, so an unearned record is visible rather than merely prohibited.

**Read by:** `/resume-line`, which is the only thing permitted to turn a record
into outward-facing text. This file holds facts; claims are generated on demand
under the bar in `.claude/commands/resume-line.md`.

**WRITE SCOPE:** you may only ADD a new `##` record at the END of this file.
Never edit, reword, reorder, or delete an existing record. Never reformat the
file. Records are the substrate every outward claim rests on; a rewritten record
is an unfalsifiable one.

**No competence field exists, by design.** A quiz verdict in `ideas/` measures
whether an idea's mechanism could be reconstructed from memory. It is not
shipped work, and it must never reach a résumé line. The schema gives it no slot
to land in.

## Record schema

```
## YYYY-MM-DD — <artifact>
- axis: <capability axis, e.g. hardware>
- transition: [tier] → [tier]
- artifact: <repo or artifact name>
- evidence: <repo path, commit count/range, concrete signals — tests, measured
  results, deployments. Numbers here are what license numbers in a claim.>
```

Records are chronological, oldest first, newest appended at the end.

**This log starts on 2026-08-04.** Transitions that happened before it existed
are not here and must not be backfilled — a record written now would have no
`profile.md` edit beside it in any commit, which is exactly the unearned record
the same-commit rule exists to expose. The three pre-existing graduations remain
noted inline in `profile.md`'s Skills section, and their history is in git.

---

## 2026-08-11 — circuit-simulator
- axis: hardware / circuit analysis
- transition: [conceptual] → [emerging]
- artifact: circuit-simulator (Python `circuitsim` package)
- evidence: Python + NumPy. Modified Nodal Analysis solver for resistive DC
  circuits — arbitrary node count, multiple ideal DC voltage sources,
  `SingularCircuitError` detection for floating nodes and shorted/
  contradictory sources. src-layout package (`components.py` → `circuit.py`
  → `solver.py`). 1 commit (2026-08-03), 403 lines. 11 pytest tests — voltage
  dividers, unequal dividers, parallel-load current, multi-source
  superposition, two singular-circuit cases, duplicate-name and
  negative-resistance validation — all green, re-run and confirmed in a
  fresh venv this session. No CI configured. No design doc predates the
  commit. Scope: resistors and ideal DC sources only — no reactive elements,
  no dependent sources, no visualizer or UI. Keystone: the
  hardware/architecture axis had zero shipped inventory before this across
  every prior refresh (2026-08-01 through 2026-08-09); this is the first
  shipped artifact on that axis, and resolves the specific node/edge
  data-model decision `profile.md` had named as the stalled blocker.

## 2026-08-11 — stripe-reconciler (OAuth account confirmation)
- axis: applied web application security
- transition: [emerging] → [strong]
- artifact: stripe-reconciler
- evidence: Python/Flask + PostgreSQL. Commits 2026-08-09 23:49 UTC through
  2026-08-11 03:23 UTC (`cbcdef0`..`d8a179e`, 8 commits, merged via PR #10).
  Adds: OAuth callback and confirmation gate; AES-256-GCM authenticated
  encryption for Stripe refresh tokens at rest, with associated-data binding
  the ciphertext to its own row (prevents copying a ciphertext onto another
  row) and versioned-key rotation support (`token_crypto.py`); an
  atomicity-first "epoch seam" for provisional connection rows
  (`test_epoch_constraints.py`, `schema.sql`); retirement of the dead
  `/stripe/claim` endpoint; and a real concurrency bug caught by CI —
  cache fill happening outside the row lock, causing a token-refresh race —
  reproduced deliberately (inserted sleep reproduced CI's exact failures)
  and fixed with a per-connection lock, with the test suite widened to
  catch it rather than just re-passing. Backend suite: 11 test files, 298
  checks green in a fresh venv built from `requirements.txt` under the CI
  workflow environment. No new production deployment observed this run.
