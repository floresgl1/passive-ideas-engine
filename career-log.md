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

## 2026-08-13 — ledger-core
- axis: relational / double-entry data modeling
- transition: [emerging] → [strong]
- artifact: ledger-core (new repo, `ledger_core` Python package)
- evidence: Python, zero runtime dependencies. Commits 2026-08-11 20:29 PT
  through 2026-08-12 20:13 PT (`fdd7835`..`7bc1815`, 8 commits). Design
  contract (`docs/DESIGN.md`) written and committed before any implementing
  code. Carries stripe-reconciler's two disciplines — integer cents
  throughout (no float ever touches money) and an explicit Suspense account
  instead of a silent default — out of a Stripe-specific shell into a
  processor-agnostic library: any list of `Transaction` objects in, a
  `Journal` that sums to zero or a `BalanceReport` naming exactly what
  doesn't balance and why, out. Adds a currency guard (mixed-currency
  journals never silently net together) and a `Suspense` routing path with
  the reason attached, neither of which stripe-reconciler generalized. 577
  tests green (`tests/test_entries.py`, `tests/test_balance.py`,
  `tests/test_journal.py`, `tests/test_stripe_adapter.py`, plus
  `tests/test_zero_sum_property.py` — a property-based test), re-run and
  confirmed in a fresh venv this session. CI (`.github/workflows/test.yml`)
  fails closed on zero-collected tests across Python 3.10/3.11/3.12, the
  same fail-closed pattern already proven in finance_bot, stripe-reconciler,
  and golf. A `stripe.py` adapter maps raw Stripe balance-transaction
  mappings to the processor-agnostic `Transaction` type with no network or
  SDK dependency. This is the same relational-modeling discipline already
  shown in stripe-reconciler, now demonstrated independently in a second
  repo, decoupled from any one processor. No production deployment; not yet
  published to PyPI (the README's `pip install ledger-core` is aspirational
  — no publish step or PyPI project observed).

## 2026-08-13 — finance_bot (daily-run-triage sub-agent)
- axis: AI-assisted development workflow / meta-tooling
- transition: [emerging] → [strong]
- artifact: finance_bot (`.claude/agents/daily-run-triage.md`,
  `pipeline_status.py`)
- evidence: Python. Commit `470c3f6` (2026-08-12) adds a repo-scoped Claude
  Code sub-agent (`daily-run-triage.md`, 118 lines) with an enforced
  read-only tool scope (Bash, Read, Grep, Glob only — no Edit/Write) and
  explicit hard rules: report only, never fix a task, never clear the guard
  or halt flag, never place or cancel an order, never print secrets. Fixed
  verdict output format (HEALTHY/DEGRADED/FAILED/NOT-A-TRADING-DAY). Backed
  by `pipeline_status.py` (287 lines), a read-only health-snapshot script
  correlating PythonAnywhere task exit codes, the run guard, the halt flag,
  `signal_log.csv`, and news-agent output — exit code 2 on missing
  credentials/network failure, 0 otherwise regardless of what it finds. This
  is the same enforced-scope, guardrail-file pattern already shipped in
  golf's `.claude/agents/`, now demonstrated independently in a second,
  unrelated repo — the standard this profile uses elsewhere to promote a
  one-repo pattern to [strong]. The tooling was put to immediate use: the
  companion commit `09cf92a` (2026-08-12) used it to diagnose and fix a real
  5-day silent production bug (the daily run guard being stamped by a
  market-closed no-op, which caused the actual 15:00 UTC trading run to skip
  every trading day since 2026-08-07), documented in a new invariant section
  in `docs/PIPELINE.md`. No dedicated test suite for the sub-agent
  definition itself (a prompt file, not executable code); no separate design
  doc predates this commit beyond the sub-agent file's own inline spec.

## 2026-08-16 — circuit-simulator (dependent sources)
- axis: hardware / circuit analysis
- transition: [emerging] → [strong]
- artifact: circuit-simulator (Python `circuitsim` package)
- evidence: Python + NumPy. PR #2 (`current-sources` branch → `main`),
  merge commit `ef963c27` on 2026-08-14, 1,266 additions / 49 deletions
  across 3 commits (`e8e12cb` four controlled sources, `ee7967f`
  SPICE-style netlist reader + CLI, `0ec9f7c` ill-conditioning rejection).
  Adds VCVS/VCCS/CCCS/CCVS (SPICE E/G/F/H) with four distinct MNA stamp
  shapes — VCCS adds off-diagonal conductance into `G`, VCVS's gain goes
  into `C` with no `B` counterpart, CCCS reaches into `B` to read another
  element's branch current, CCVS is the only one that writes into `D`; a
  netlist parser (`netlist.py`, new, 140 lines) that handles SPICE's
  `m`-vs-`meg` trap and errors with line number and text on malformed
  input; a CLI (`__main__.py`, new, 70 lines: `circuitsim netlist.cir`);
  and `IllConditionedCircuitError`, which estimates relative solve error on
  the *Ruiz-equilibrated* matrix rather than thresholding raw condition
  number, so pure diagonal scaling (mixing a 1Ω resistor with a 1PΩ one)
  isn't mistaken for genuine precision loss. Test suite grew from 26 to 68
  (32 solver, 18 dependent-source — including an op-amp closed-loop-gain
  convergence check and a gyrator, both physically-meaningful compositions,
  plus a power-conservation check exercising all four source types at
  once — and 18 netlist), all green, re-run and confirmed in a fresh venv
  this session. Still no CI configured; no design doc predates the commits.
  Closes the specific "no dependent sources" gap `profile.md`'s Keystone
  note had named as the hardware axis's next-highest-leverage move; the
  visualizer and interactivity phases (3–4 of the self-scoped 5-phase plan)
  remain unshipped.
