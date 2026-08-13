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
**Tech:** Python (MediaPipe Pose Tasks, OpenCV, matplotlib, pytest) as source of truth; Flutter/Dart port (ML Kit, ffmpeg, camera, share_plus), iOS build now verified in CI via a dedicated macOS compile-check workflow (unsigned build, pinned Flutter SDK) plus a post-generate iOS config script that closed a gap the compile check couldn't see (missing camera-permission string; a weak `grep -q` guard that passed with 1 of 3 build configs unpatched). ~8.4k LOC, 66 commits, PR workflow, custom Claude Code sub-agents checked into the repo.
**Transferable capability:** Turn a noisy real-world signal into a defensible measurement pipeline — physically-reasoned landmark choice, scale-invariant normalization, correct math (circular smoothing, pixel-space angles). Maintains two parallel language implementations *and* holds a known divergence open on purpose until thresholds are validated. Knows the difference between a measurement and a verdict, and ships the product that claims only the former.

### finance_bot
Fully automated algorithmic paper-trading system over a 12-ticker watchlist: data collection, feature engineering, XGBoost BUY/SELL/HOLD classifier, Alpaca execution, self-logging/evaluation/alerting.
**Tech:** Python (~8.2k LOC, 37 modules, 52 commits) — XGBoost, scikit-learn, pandas, ta, SHAP, FinBERT, yfinance, Finnhub, Alpaca, Groq (Llama 3.3 70B), Tavily, tenacity. Three scheduled GitHub Actions workflows, pytest, Discord alerting, design/audit docs.
**Transferable capability:** Build and operate an unattended production system with money-adjacent stakes — the whole loop, not just the model: pre-flight validation between refresh and live run, loss-limit/halt-flag circuit breakers, staleness detection, transition-only degradation alarms. Understands ML-on-time-series failure modes (walk-forward retraining, regime-aware dynamic labeling, model+encoder bundled to prevent drift). Builds LLM agents with real discipline: bounded steps, finite retry budget, enumerated error categories, constrained verdict vocabulary, deliberately information-only v1. This week added a read-only health-snapshot script (`pipeline_status.py`) and a repo-scoped Claude Code sub-agent (`daily-run-triage`, enforced Bash/Read/Grep/Glob-only scope, "report only" hard rule) that diagnoses silent failures — runs that exit 0 without trading. That diagnostic surfaced a real 5-day silent bug (the daily run guard being stamped by a market-closed no-op, skipping the actual 15:00 UTC trading run every day since 2026-08-07), which was then fixed and documented in a new `docs/PIPELINE.md` invariant section.

### stripe-reconciler
Turns a Stripe payout into balanced double-entry journal entries — books each balance transaction, splits revenue from fees, routes unknowns to Suspense, exports a journal that sums to zero.
**Tech:** Flask + PostgreSQL, React 19 + Vite, Stripe Apps auth, bcrypt, flask-limiter, Resend, gunicorn on Render. ~3.9k LOC, 77 commits, 11 backend test suites (298 checks) gated by CI.
**Transferable capability:** Build a multi-tenant SaaS product end to end and get the unglamorous parts right. Security is reasoned, not cargo-culted: CSRF origin checks with a documented webhook exemption, timing-attack equalization, digest-only single-use reset tokens, origin-derived cookie flags, tenant-isolation tests as a first-class suite. Disciplined domain modeling: integer cents throughout, explicit Suspense account over silent defaults. CI fails closed when no suites are found. Extended to scope payouts/journals per Stripe *connection* rather than per user — authorization epochs, connection-scoped uniqueness — so one bookkeeper can manage multiple client accounts without cross-account leakage. This week completed the OAuth account-confirmation flow on top of that: refresh tokens encrypted at rest (AES-256-GCM, associated data binds ciphertext to its row so it can't be copied onto another), an atomicity-first "epoch seam" for provisional connection rows, and a real cache/row-lock race caught by CI, reproduced deliberately, and fixed rather than papered over.

### ledger-core
Takes any list of transactions and returns a double-entry journal that sums to zero, or a report of exactly what doesn't balance and why — stripe-reconciler's ledger discipline decoupled from Stripe into a standalone, processor-agnostic library.
**Tech:** Python, zero runtime dependencies. ~1.4k LOC across 8 commits (2026-08-11 to 2026-08-12). `pyproject.toml` packaging, GitHub Actions CI (Python 3.10/3.11/3.12, fail-closed on zero-collected tests), a Stripe input adapter with no network/SDK dependency.
**Transferable capability:** The same two disciplines already proven in stripe-reconciler — integer cents throughout (no float ever touches money) and an explicit Suspense account instead of a silent default — generalized to accept any transaction shape, plus a new currency guard (mixed-currency journals never silently net together). Design-doc-first: `docs/DESIGN.md` was written and committed before any implementing code. 577 tests green (unit suites plus a property-based zero-sum test), re-run and confirmed in a fresh venv this session. **Honest scope note:** not yet published to PyPI — the README's `pip install ledger-core` is aspirational; no production deployment or external user observed.

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
A small DC circuit simulator built on Modified Nodal Analysis (MNA): resistors, ideal DC voltage sources, and (as of this week) ideal DC current sources, solving for every node voltage and reporting per-component current and power, with explicit `SingularCircuitError` detection for floating nodes and shorted/contradictory sources.
**Tech:** Python, NumPy, pytest. src-layout package (`components.py` → `circuit.py` → `solver.py`, one-way dependency arrow). 4 commits on `main` (2026-08-03, then 2026-08-11 added current sources and per-component current/power reporting), 26 tests, all green (re-run and confirmed in a fresh venv this session). No CI configured yet; no design doc predates the commits.
**Transferable capability:** Real numerical circuit analysis (not a faked/table-driven output) shipped as working, tested code — the first shipped artifact on the hardware/architecture axis. Scoped narrowly on purpose: resistive DC networks only, no reactive elements yet. **Honest scope note:** this is the data-model-and-solver phase of the 5-phase circuit-simulator direction named in `profile.md`'s own Direction & Intent (data model → solver → visualizer → interactivity → dependent sources) — the visualizer/interactivity/dependent-sources phases are still unshipped on `main`, and this implementation is Python, not the originally-stated JS/visual version. **Not yet credited:** the `current-sources` branch (`origin/current-sources`) carries three further commits — four controlled/dependent sources (VCVS/VCCS/CCVS/CCCS), a SPICE-style netlist reader plus CLI, and ill-conditioned-circuit rejection — that are not merged to `main` as of this refresh. If merged, that would resolve the "no dependent sources" gap named in the Keystone note below; deliberately not counted until it lands on the mainline.

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
- **Applied web application security** *(PROMOTED to [strong])* — CSRF enforcement with reasoned exemptions, timing equalization, digest-only tokens, single-use resets, webhook signature verification, tenant isolation under test, plus (new) an OAuth account-confirmation flow: AES-256-GCM at-rest encryption for refresh tokens with associated-data row binding and key rotation, an atomicity-first "epoch seam" for provisional rows, and a real concurrency race caught by CI, reproduced deliberately, and fixed with correct reasoning about lock scope (stripe-reconciler). Still one repo, but now deep rather than shallow — reasoned, tested (298 checks), and has already caught a real bug in itself.
- **Relational / double-entry data modeling** *(PROMOTED to [strong])* — integer cents throughout, explicit Suspense account over silent defaults, idempotent uniqueness constraints, inline migration rationale notes (stripe-reconciler); now demonstrated a second time, decoupled from any one processor, as `ledger-core` — a standalone, zero-dependency, design-doc-first library with a new currency guard and a property-based zero-sum test, 577 checks green, fail-closed CI across three Python versions. Shown across two repos with real depth, not a one-off.
- **AI-assisted development workflow at the tooling level** *(PROMOTED to [strong])* — repo-scoped Claude Code sub-agents with an enforced tool scope and explicit hard rules, shipped as checked-in tooling rather than ad hoc prompting; proven independently in golf and (new) finance_bot's `daily-run-triage` sub-agent, which is scoped to read-only tools (Bash/Read/Grep/Glob, no Edit/Write), forbidden from fixing anything it finds, and already used to diagnose a real 5-day silent production bug. Shown across two unrelated repos, not a one-off pattern.

### [emerging] — appears once or shallowly, shipped
- Model interpretability and comparison — SHAP, compare_models.py (finance_bot).
- Embeddings and vector retrieval / RAG — CLIP, ChromaDB (visual-search-engine).
- NLP for domain signal — FinBERT sentiment, held dormant as a veto (finance_bot).
- Systems programming — processes, IPC, signals, threads, mutexes in C/C++ (CSE4600-HW1).
- React 19 + Vite SPA against an authenticated API (stripe-reconciler).
- Flutter/Dart mobile — camera capture, on-device inference, local persistence, share-sheet-only export (golf).
- Containerization and deployment — Docker/compose, gunicorn on Render (visual-search-engine, stripe-reconciler, finance_bot).
- **Circuit analysis** *(GRADUATED from [conceptual])* — Modified Nodal Analysis solver for resistive DC circuits: arbitrary node count, multiple ideal voltage sources, singular-circuit detection (circuit-simulator). Scoped narrowly — resistors and DC sources only, no reactive elements or dependent sources yet; broader coursework understanding (mesh/phasor/s-domain, PSpice cross-verification) remains conceptual beyond what's actually shipped here.

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
- **Visual JS circuit simulator** — self-scoped 5-phase architecture (data model → solver → visualizer → interactivity → dependent sources); deliberately chose the hard version (real analysis algorithms, not faked output). Phases 1–2 (data model, MNA solver) shipped as `circuit-simulator` — see Projects and Keystone — but in Python, not the originally-stated JS/visual form. **Still open:** visualizer, interactivity, dependent sources, and, if the "visual" framing is kept, a JS port or front end.
- **finance_bot Agent v2 ESCALATE** — LLM agent promoting HOLD→SELL on held positions using news/sentiment the model can't see; scoped to held-positions-only to bound failure/cost.
- Verify fractional-share + OTO stop-loss compatibility in Alpaca paper before any live deployment (load-bearing first step for a $1,000 experiment).
- Complete the SELL payoff/breakeven derivation; make an explicit go/no-go accepting single-regime risk.
- Longer-horizon: evolve the hand-rolled ReAct agent toward a framework (CrewAI mentioned).
- Network diagnostics exploration (ping/traceroute/pathping, measured-vs-theoretical throughput); more Wireshark (TCP handshake analysis).

---

## Keystone

**The hinge has been crossed.** The hardware/architecture axis had zero shipped inventory as of every prior refresh (2026-08-01 through 2026-08-09); this refresh found `circuit-simulator` — a working, tested Modified Nodal Analysis solver (see Projects) — already shipped as of 2026-08-03, and never previously reflected here because the repo was outside this routine's scope until now. Circuit analysis has moved [conceptual] → [emerging] accordingly.

This is a **partial** crossing, not a finished keystone. What shipped is phases 1–2 of the self-scoped 5-phase visual JS circuit simulator (data model, MNA solver) — real analysis, not faked output, exactly the "hard version" the original plan called for — but in Python, with no visualizer, no interactivity, no dependent sources, and no JS/visual front end. The specific blocker named across ten prior digests (the node/edge data-model decision) is resolved; what remains is the rest of the build, not a decision.

The still-open work — visualizer, interactivity, dependent sources, and the visual/JS framing if that's kept — remains the next highest-leverage move on this axis. The idea engine's other three untested hardware conceptual lines (real-time scheduling, computer-architecture Iron Law, polling-vs-interrupt) are still [conceptual] and still unbuilt; this crossing doesn't resolve those, it resolves circuit analysis specifically.

---

## Trajectory

Older work (Oct–Nov 2025) is tutorial-shaped: notebook-first ML/CV that stands up a modern stack and stops at the demo. From March 2026 onward the center of gravity moves from *building a model* to *operating a system*: finance_bot, stripe-reconciler, and golf are long-running multi-month repos with CI, tests, design docs, deliberate failure handling. The sharpest trend is epistemic rather than technical — organizing work around knowing what hasn't been validated and refusing to act as though it has. This week reinforced that pattern twice more: `ledger-core` shows the double-entry discipline generalizing cleanly out of its original repo (design-doc-first, zero deps, property-tested), and finance_bot's `daily-run-triage` sub-agent shows the same enforced-scope tooling habit from golf now repeated deliberately rather than being a one-off. A distinct, newer axis (hardware/architecture/embedded) remains narrow and one repo deep — circuit-simulator gained breadth (current sources, per-component power) but the higher-leverage gap (dependent sources, a visualizer) is sitting on an unmerged branch, not yet shipped. Direction: production ML / applied-AI systems engineering, with a hardware pull that has started shipping but is advancing slower than the software-discipline axis.

---

## last_refresh
2026-08-13
