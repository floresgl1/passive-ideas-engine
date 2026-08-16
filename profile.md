# Capability Profile

> **How to read this file (tiers matter):**
> - **Skills — [strong] / [emerging]** = DEMONSTRATED and SHIPPED in a real repo.
> - **Skills — [conceptual]** = UNDERSTOOD but NOT YET SHIPPED (reasoned through, never built into a project). Do not treat as shippable capability.
> - **Direction & Intent** = QUARANTINED. Goals and wants, NOT proof. Never source shippable ideas from here as if demonstrated.
> The separation is deliberate: it keeps downstream idea-generation from over-reaching.

---

## Projects (demonstrated, shipped)

### golf-swing-analyzer
Analyzes a golf swing from a single phone video — per-frame pose estimation, phase segmentation, tempo/body-angle measurement, four biomechanical fault flags, corrective drills, closed diagnose → prescribe → verify loop across sessions.
**Tech:** Python (MediaPipe Pose Tasks, OpenCV, matplotlib, pytest) as source of truth; Flutter/Dart port (ML Kit, ffmpeg, camera, share_plus), iOS build now verified in CI via a dedicated macOS compile-check workflow (unsigned build, pinned Flutter SDK) plus a post-generate iOS config script that closed a gap the compile check couldn't see (missing camera-permission string; a weak `grep -q` guard that passed with 1 of 3 build configs unpatched). This week the config script's Podfile patch was gated on whether the host could actually have generated a Podfile in the first place (only a host with a working Xcode does) — fixing a bug where the documented local workflow (`flutter create` → `configure_ios.py`) exited 1 unconditionally on Linux/CI hosts, which never produce one. ~8.4k LOC, 66 commits, PR workflow, custom Claude Code sub-agents checked into the repo.
**Transferable capability:** Turn a noisy real-world signal into a defensible measurement pipeline — physically-reasoned landmark choice, scale-invariant normalization, correct math (circular smoothing, pixel-space angles). Maintains two parallel language implementations *and* holds a known divergence open on purpose until thresholds are validated. Knows the difference between a measurement and a verdict, and ships the product that claims only the former.

### finance_bot
Fully automated algorithmic paper-trading system over a 12-ticker watchlist: data collection, feature engineering, XGBoost BUY/SELL/HOLD classifier, Alpaca execution, self-logging/evaluation/alerting.
**Tech:** Python (~8.2k LOC, 37 modules, 52 commits) — XGBoost, scikit-learn, pandas, ta, SHAP, FinBERT, yfinance, Finnhub, Alpaca, Groq (Llama 3.3 70B), Tavily, tenacity. Three scheduled GitHub Actions workflows, pytest, Discord alerting, design/audit docs.
**Transferable capability:** Build and operate an unattended production system with money-adjacent stakes — the whole loop, not just the model: pre-flight validation between refresh and live run, loss-limit/halt-flag circuit breakers, staleness detection, transition-only degradation alarms. Understands ML-on-time-series failure modes (walk-forward retraining, regime-aware dynamic labeling, model+encoder bundled to prevent drift). Builds LLM agents with real discipline: bounded steps, finite retry budget, enumerated error categories, constrained verdict vocabulary, deliberately information-only v1. This week added a read-only health-snapshot script (`pipeline_status.py`) and a repo-scoped Claude Code sub-agent (`daily-run-triage`, enforced Bash/Read/Grep/Glob-only scope, "report only" hard rule) that diagnoses silent failures — runs that exit 0 without trading. That diagnostic surfaced a real 5-day silent bug (the daily run guard being stamped by a market-closed no-op, skipping the actual 15:00 UTC trading run every day since 2026-08-07), which was then fixed and documented in a new `docs/PIPELINE.md` invariant section.

### stripe-reconciler
Turns a Stripe payout into balanced double-entry journal entries — books each balance transaction, splits revenue from fees, routes unknowns to Suspense, exports a journal that sums to zero.
**Tech:** Flask + PostgreSQL, React 19 + Vite, Stripe Apps auth, bcrypt, flask-limiter, Resend, gunicorn on Render. ~3.9k LOC, 77 commits, 11 backend test suites (298 checks) gated by CI.
**Transferable capability:** Build a multi-tenant SaaS product end to end and get the unglamorous parts right. Security is reasoned, not cargo-culted: CSRF origin checks with a documented webhook exemption, timing-attack equalization, digest-only single-use reset tokens, origin-derived cookie flags, tenant-isolation tests as a first-class suite. Disciplined domain modeling: integer cents throughout, explicit Suspense account over silent defaults. CI fails closed when no suites are found. Extended to scope payouts/journals per Stripe *connection* rather than per user — authorization epochs, connection-scoped uniqueness — so one bookkeeper can manage multiple client accounts without cross-account leakage. This week completed the OAuth account-confirmation flow on top of that: refresh tokens encrypted at rest (AES-256-GCM, associated data binds ciphertext to its row so it can't be copied onto another), an atomicity-first "epoch seam" for provisional connection rows, and a real cache/row-lock race caught by CI, reproduced deliberately, and fixed rather than papered over.
**Since that refresh (this run):** the OAuth grant is now actually *spent*, not just stored — earlier reads went out on the platform secret key in the Connect shape, which cannot work for a non-Connect Marketplace app and would have surfaced in production as every client looking like it had revoked access. All Stripe reads now route through the per-connection access token via one `stripe_auth()` chokepoint, with three previously-conflated failures now distinguished (`NeedsReauthorization` vs. a transient token-endpoint error vs. a permission gap on our own manifest). Manifest permissions were corrected against Stripe's real, verified grant list — `bank_account_read` doesn't exist and was dropped, with the confirmation screen's strength downgraded from "strong" to "weak" rather than silently keeping a check that could never pass — and the account-read call was fixed from the Connect-shaped `Account.retrieve(id)` to the app-token-scoped `GET /v1/account`. Verified end-to-end against live Stripe, not just against CI's stub. (Marketing/landing-page copywriting also shipped in this window but isn't a tracked engineering capability, so it's omitted here.)

### ledger-core
Takes any list of transactions and returns a double-entry journal that sums to zero, or a report of exactly what doesn't balance and why — stripe-reconciler's ledger discipline decoupled from Stripe into a standalone, processor-agnostic library. Now dispatches transactions end to end (`entries_for`): unclassifiable or failed transactions route to Suspense with a named reason on a review list, and a specific real double-booking risk is caught — Stripe reports an automatic payout both as a balance transaction and as a `Payout` object, and booking both would silently double the cash movement; the second is refused and flagged `REVIEW_ALREADY_BOOKED` rather than dropped or double-counted.
**Tech:** Python, zero runtime dependencies. Published to PyPI as **`ledger-tieout`** (the import package stays `ledger_core`; the name `ledger-core`/`ledger_core` was already taken by an unrelated project, and PyPI normalizes away separators, so no punctuation variant was free) via trusted OIDC publishing on a `v*` tag — no long-lived token stored anywhere. `pyproject.toml` packaging, GitHub Actions CI now spanning Python 3.10–3.13 plus `ruff`/`mypy --strict`, fail-closed on zero-collected tests, a Stripe input adapter with no network/SDK dependency. 680 tests green (unit suites, a property-based zero-sum test, and a new currency-formatting suite), re-run and confirmed in a fresh venv this session.
**Transferable capability:** The same two disciplines proven in stripe-reconciler — integer cents throughout and an explicit Suspense account instead of a silent default — generalized to accept any transaction shape, plus a currency guard and, new this week, real per-currency minor-unit formatting: the prior flat two-decimal formatter rendered 150 JPY as "1.50 JPY", a hundredfold display error caught and fixed with a table-driven exponent lookup plus a dedicated money-formatting test suite. Design-doc-first: `docs/DESIGN.md` predates any implementing code. **Honest scope note resolved:** the README's `pip install` line is no longer aspirational — 0.1.0 and 0.2.0 are both live on PyPI, installable, with a documented, secretless release pipeline.

### visual-search-engine
Natural-language search over an image collection: CLIP embeddings for images and text, vector DB, REST API.
**Tech:** PyTorch, HuggingFace transformers (CLIP ViT-B/32), ChromaDB, FastAPI/Pydantic, OpenCV, Docker + docker-compose. ~980 LOC plus three executed notebooks, committed embeddings and PCA visualizations.
**Transferable capability:** Carry a modern retrieval/RAG stack from exploration to a deployable service — embedding generation, vector storage, similarity retrieval, HTTP API, container. Notebooks executed with outputs preserved (reproducible, not aspirational).

### traffic-demographic-analysis *(prototype)*
Vehicle detection/classification on Caltrans traffic footage, intended to correlate with Census ACS + LODES employment data for the Inland Empire logistics corridor.
**Tech:** PyTorch, OpenSeeD/detectron2, YOLOv8, geopandas/shapely/pyproj, pysal/esda, statsmodels. 432-line model wrapper, 235 lines of tests, GPU/CPU setup scripts, methodology doc.
**Transferable capability:** Scope a genuine research question (a correlation between a CV-derived measurement and an independent public dataset, not a detection demo) and stand up its infrastructure. **Honest scope note:** analysis never run; notebooks unexecuted, results still "to be completed." Demonstrates setup and research framing, NOT delivered findings.

### CSE4600-HW1 *(coursework)*
OS coursework in C/C++: fork topologies, signals/sigaction, exec, wait across grandchildren, pipes, pthreads, a deliberate race condition with two mutex fixes.
**Tech:** C/C++, POSIX, pthreads, mutexes. ~1.3k LOC across 26 programs.
**Transferable capability:** Working understanding of the systems layer beneath the Python — process lifecycle, IPC, concurrency primitives. Race-condition-then-fix progression shows the failure was reproduced before it was patched.

### circuit-simulator
A DC circuit simulator built on Modified Nodal Analysis (MNA): resistors, ideal DC voltage and current sources, and all four controlled/dependent sources (VCVS/VCCS/CCCS/CCVS), solving for every node voltage and reporting per-component current and power, with `SingularCircuitError`/`IllConditionedCircuitError` detection for floating nodes, shorted/contradictory sources, and numerically-unsafe (near-singular) circuits. Circuits can also be read from a SPICE-style netlist and solved from a CLI.
**Tech:** Python, NumPy, pytest. src-layout package (`components.py` → `circuit.py` → `solver.py`/`netlist.py` → `__main__.py`, one-way dependency arrow). 7 commits on `main` (2026-08-03 data model/solver; 2026-08-11 current sources; 2026-08-14 merged the `current-sources` branch — PR #2, 1,266 added / 49 deleted lines — adding the four controlled sources, the netlist reader, the CLI, and ill-conditioning rejection). 68 tests, all green (32 solver, 18 dependent-source — including an op-amp closed-loop-gain convergence check and a gyrator, both physically-meaningful compositions rather than bare unit assertions, plus a power-conservation check across all four source types at once — and 18 netlist). The ill-conditioning check is reasoned, not a naive threshold: it estimates relative solve error on the *equilibrated* matrix via Ruiz equilibration, so pure diagonal scaling (a 1Ω resistor beside a 1PΩ one) isn't mistaken for genuine precision loss. Still no CI configured; no design doc predates the commits.
**Transferable capability:** Real numerical circuit analysis, now deep rather than shallow — four distinct controlled-source stamp shapes that each break MNA matrix symmetry differently (VCCS adds off-diagonal conductance into `G`; VCVS puts its gain into `C` with no `B` counterpart; CCCS reaches into `B` to read another element's branch current; CCVS is the only one that writes into `D`), a netlist grammar handling a real engineering-notation edge case (SPICE's `m`-vs-`meg` trap) with line-numbered errors, and a numerically-reasoned safety net in place of a bare `try/except`. This closes the "no dependent sources" gap the Keystone note had flagged as the hardware axis's next-highest-leverage move. **Honest scope note:** this is phases 1, 2, and 5 of the 5-phase circuit-simulator direction named in Direction & Intent (data model → solver → visualizer → interactivity → dependent sources) — phases 3–4 (visualizer, interactivity) are still unshipped, and this remains a Python CLI/library, not the originally-stated JS/visual front end.

### Skipped (empty scaffolds — excluded from all reasoning)
- **coral-reef-monitor** — empty requirements.txt, 0-byte notebook, empty `__init__.py`, one commit, no implementation.
- **CSE4600-Assignment1** — README/LICENSE/.gitignore only, one commit, no code.

---

## Skills

### [strong] — demonstrated across repos or with real depth
- **Epistemic discipline about one's own system** — measurement vs. verdict; hold a port divergence open rather than mirror an unvalidated change; ship tentative language when reference values aren't validated; mark infrastructure dormant pending evidence; self-audit silent failures by severity. **Confirmed across 5+ independent domains** (golf thresholds, ALU-multiplier rejection, real-time self-corrections, networking self-corrections, trading confidence intervals). This is the through-line of the whole profile and the least common skill in the set.
  > **META-SKILL GUARDRAIL:** This RAISES THE QUALITY BAR on any idea and can be a TEACHING / CONTENT wedge. It must NEVER become a standalone product ("sell my good judgment" = vapor). Downstream idea-generation should use it to shape quality and as a teaching angle, never as a product itself.
- **Applied statistical reasoning under uncertainty** *(GRADUATED to [strong])* — CI lower bound vs. a dynamically-computed breakeven, hit-rate-needs-payoff-ratio, sample-size intuition, leading-vs-lagging indicator separation, volume gates as necessary-but-not-sufficient. Shipped: it gates finance_bot's live behavior.
- **Supervised ML on time series with leakage awareness** — walk-forward retraining, chronological splits, regime-aware dynamic labeling, held-out reporting (finance_bot; reinforced by golf's validation harness).
- **Feature engineering from raw signals** — technical indicators, market context, timezone-safe heterogeneous merges (finance_bot).
- **Computer vision on real footage** — pose estimation, keypoint biomechanics, scale normalization, detection/segmentation (golf, visual-search-engine; note: traffic set up this capability but its analysis never ran).
- **LLM agent engineering** *(PROMOTED toward [strong])* — bounded loops, step caps, retry budgets, enumerated error categories, constrained output vocabularies, a written v1 contract; underpinned by architectural reasoning (structural SELL-blindspot diagnosis, sentiment as veto-vs-feature-vs-raw, agent-as-architectural-response). One repo, but the depth clearly meets the bar.
- **Python service and pipeline architecture** — separation of concerns as an explicit rule so one failure can't corrupt another (finance_bot, stripe-reconciler, golf).
- **REST API design** — Flask and FastAPI, Pydantic schemas, session auth, route-level rate limiting (stripe-reconciler, visual-search-engine).
- **Testing discipline** — pytest and Dart suites, characterization tests, golden fixtures, robustness tests; recognizes that a threshold test which can't detect a threshold change is vacuous (golf, finance_bot, stripe-reconciler).
- **CI/CD and scheduled automation** — GitHub Actions, concurrency groups, cron, caching, PR-gated fail-closed workflows (finance_bot, stripe-reconciler, golf).
- **Production observability and failure handling** — pre-flight validation, staleness detection, halt flags, loss limits, transition-only alerting, severity-classified silent-failure audits (finance_bot).
- **Design documentation as a first-class artifact** — roadmaps as single source of truth, decision records, contracts written before the implementing commit (golf, finance_bot, stripe-reconciler).
- **Learns by building instrumentation** *(GRADUATED out of Intent)* — builds tooling/harnesses/simulators to serve his own understanding; proven by Claude Code sub-agents and harness-before-feature patterns in the repos, not merely stated.
- **Applied web application security** *(PROMOTED to [strong])* — CSRF enforcement with reasoned exemptions, timing equalization, digest-only tokens, single-use resets, webhook signature verification, tenant isolation under test, plus an OAuth account-confirmation flow: AES-256-GCM at-rest encryption for refresh tokens with associated-data row binding and key rotation, an atomicity-first "epoch seam" for provisional rows, and a real concurrency race caught by CI, reproduced deliberately, and fixed with correct reasoning about lock scope (stripe-reconciler). This week the grant went from stored-but-unused to actually spent end to end against live Stripe, surfacing and fixing a real Connect-vs-Marketplace credential-scope bug plus two manifest permission defects. Still one repo, but now deep and verified against a live integration rather than only against CI's stub.
- **Relational / double-entry data modeling** *(PROMOTED to [strong])* — integer cents throughout, explicit Suspense account over silent defaults, idempotent uniqueness constraints, inline migration rationale notes (stripe-reconciler); demonstrated a second time, decoupled from any one processor, as `ledger-core` — a standalone, zero-dependency, design-doc-first library, now published to PyPI (`ledger-tieout`) with 680 tests green, fail-closed CI across four Python versions, and a real currency-formatting bug (JPY rendered at 100x) caught and fixed this week. Shown across two repos with real depth, not a one-off.
- **AI-assisted development workflow at the tooling level** *(PROMOTED to [strong])* — repo-scoped Claude Code sub-agents with an enforced tool scope and explicit hard rules, shipped as checked-in tooling rather than ad hoc prompting; proven independently in golf and finance_bot's `daily-run-triage` sub-agent, which is scoped to read-only tools (Bash/Read/Grep/Glob, no Edit/Write), forbidden from fixing anything it finds, and already used to diagnose a real 5-day silent production bug. Shown across two unrelated repos, not a one-off pattern.
- **Circuit analysis** *(PROMOTED to [strong])* — Modified Nodal Analysis solver covering resistors, independent DC voltage/current sources, and all four controlled/dependent sources (VCVS/VCCS/CCCS/CCVS), a SPICE-style netlist reader, a CLI, and a numerically-reasoned ill-conditioning safety net (Ruiz-equilibrated condition-number estimate, not a naive threshold) (circuit-simulator). 68 tests, including physically-meaningful compositions — an op-amp's closed-loop gain converging correctly, a gyrator — rather than only unit-level assertions. Scoped narrowly still: DC only, no reactive elements or time-domain/phasor/s-domain analysis; broader coursework understanding beyond what's shipped here remains `[conceptual]`.

### [emerging] — appears once or shallowly, shipped
- Model interpretability and comparison — SHAP, compare_models.py (finance_bot).
- Embeddings and vector retrieval / RAG — CLIP, ChromaDB (visual-search-engine).
- NLP for domain signal — FinBERT sentiment, held dormant as a veto (finance_bot).
- Systems programming — processes, IPC, signals, threads, mutexes in C/C++ (CSE4600-HW1).
- React 19 + Vite SPA against an authenticated API (stripe-reconciler).
- Flutter/Dart mobile — camera capture, on-device inference, local persistence, share-sheet-only export (golf).
- Containerization and deployment — Docker/compose, gunicorn on Render (visual-search-engine, stripe-reconciler, finance_bot).

### [conceptual] — understood, reasoned through, NOT yet shipped
*(All from coursework. Genuine understanding, no repo. Do not treat as build-capacity.)*
- **Computer-architecture datapath & tradeoff reasoning** — single-cycle MIPS datapath/control; Iron Law as a working lens; quantitative tradeoff analysis (rejected an ALU multiplier for ~20% slowdown once critical path and cost/perf were computed) (CSE 4010).
- **Real-time scheduling analysis** — EDF, LDF, Rate Monotonic, cyclic executive, polling servers; precedence-graph reasoning; hyperperiod/schedulability analysis (Embedded Systems / MSP432).
- **Hardware–software boundary reasoning** — driver placement, polling vs. interrupt tradeoffs (CPU/latency/power), interrupt signal direction (Embedded Systems).
- **Networking across the full stack** — forwarding vs. routing (data/control plane), transport reliability, IP addressing/subnetting, link-layer mechanics; hands-on Wireshark capture (HTTP caching, DNS, TCP handshakes) (CSE 4100).

---

## Direction & Intent (quarantined — expressed, NOT demonstrated)

### Hardware / computer architecture — target career track
- **Goal:** entry-level hardware/firmware role (IBM named as target). Coursework deliberately reframed around industry requirements.
- **Wants to move into (not yet done):** pipelining w/ hazard detection & forwarding, memory hierarchy/caches, UART/SPI/I2C, signal integrity, power analysis, firmware/embedded, Linux kernel basics.
- **Wants hands-on:** FPGA boards (Basys 3, DE10-Lite), industry tooling (Vivado, ModelSim, Quartus), a documented portfolio (testbenches, waveforms, timing diagrams), team projects.
- **Language intent:** C/C++ for firmware/embedded; also Python/JS proficiency.
- **Embedded next step queued:** timers & PWM (MSP432).

### Stated build intentions (unstarted or in-flight)
- **Visual JS circuit simulator** — self-scoped 5-phase architecture (data model → solver → visualizer → interactivity → dependent sources); deliberately chose the hard version (real analysis algorithms, not faked output). Phases 1, 2, and 5 (data model, MNA solver, dependent sources) have shipped as `circuit-simulator` — see Projects and Keystone — but in Python, not the originally-stated JS/visual form. **Still open:** the visualizer and interactivity (phases 3–4), and, if the "visual" framing is kept, a JS port or front end.
- **finance_bot Agent v2 ESCALATE** — LLM agent promoting HOLD→SELL on held positions using news/sentiment the model can't see; scoped to held-positions-only to bound failure/cost.
- Verify fractional-share + OTO stop-loss compatibility in Alpaca paper before any live deployment (load-bearing first step for a $1,000 experiment).
- Complete the SELL payoff/breakeven derivation; make an explicit go/no-go accepting single-regime risk.
- Longer-horizon: evolve the hand-rolled ReAct agent toward a framework (CrewAI mentioned).
- Network diagnostics exploration (ping/traceroute/pathping, measured-vs-theoretical throughput); more Wireshark (TCP handshake analysis).

---

## Keystone

**The hinge was crossed on 2026-08-11** (circuit-simulator's MNA solver, phases 1–2 of the self-scoped 5-phase plan) and **advanced further this week**: phase 5 (dependent sources) shipped 2026-08-14 — all four controlled-source types, a SPICE-style netlist reader, a CLI, and ill-conditioning rejection, merged to `main` in a single 1,266-line PR. Circuit analysis has moved [emerging] → [strong] accordingly (see Skills).

What remains unshipped is now narrower and more specific: **phases 3–4 only** — a visualizer and click-to-edit interactivity — plus the original visual/JS framing if that's kept (the shipped work is a Python CLI/library, not a browser front end). Every other piece of the self-scoped plan (data model, solver, dependent sources) is done and tested.

The idea engine's other three untested hardware `[conceptual]` lines (real-time scheduling, computer-architecture Iron Law, polling-vs-interrupt) remain unbuilt; this crossing doesn't resolve those, it resolves circuit analysis specifically.

---

## Trajectory

Older work (Oct–Nov 2025) is tutorial-shaped: notebook-first ML/CV that stands up a modern stack and stops at the demo. From March 2026 onward the center of gravity moves from *building a model* to *operating a system*: finance_bot, stripe-reconciler, and golf are long-running multi-month repos with CI, tests, design docs, deliberate failure handling. The sharpest trend is epistemic rather than technical — organizing work around knowing what hasn't been validated and refusing to act as though it has. The hardware/architecture axis, previously the slowest-moving, closed real ground this week: circuit-simulator's dependent-source branch merged (all four controlled-source types, a netlist reader, a CLI, numerically-reasoned conditioning checks), promoting circuit analysis to `[strong]` and leaving only the visualizer/interactivity phases open on that axis — a genuine acceleration, not just added breadth. `ledger-core` also crossed from "internal library" to "installable package" this week (published to PyPI as `ledger-tieout`; a real user could `pip install` it today), though that specific milestone has no tier to move into and so has no `career-log.md` entry — a gap in the record-keeping schema worth the reader's attention, not a gap in the work itself. Direction: production ML / applied-AI systems engineering, with the hardware pull now advancing at a pace comparable to the software-discipline axis rather than visibly lagging it.

---

## last_refresh
2026-08-16
