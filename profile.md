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
**Tech:** Python (MediaPipe Pose Tasks, OpenCV, matplotlib, pytest) as source of truth; Flutter/Dart port (ML Kit, ffmpeg, camera, share_plus), iOS build now verified in CI via a dedicated macOS compile-check workflow (unsigned build, pinned Flutter SDK). ~8.4k LOC, 66 commits, PR workflow, custom Claude Code sub-agents checked into the repo.
**Transferable capability:** Turn a noisy real-world signal into a defensible measurement pipeline — physically-reasoned landmark choice, scale-invariant normalization, correct math (circular smoothing, pixel-space angles). Maintains two parallel language implementations *and* holds a known divergence open on purpose until thresholds are validated. Knows the difference between a measurement and a verdict, and ships the product that claims only the former.

### finance_bot
Fully automated algorithmic paper-trading system over a 12-ticker watchlist: data collection, feature engineering, XGBoost BUY/SELL/HOLD classifier, Alpaca execution, self-logging/evaluation/alerting.
**Tech:** Python (~8.2k LOC, 37 modules, 50 commits) — XGBoost, scikit-learn, pandas, ta, SHAP, FinBERT, yfinance, Finnhub, Alpaca, Groq (Llama 3.3 70B), Tavily, tenacity. Three scheduled GitHub Actions workflows, pytest, Discord alerting, design/audit docs.
**Transferable capability:** Build and operate an unattended production system with money-adjacent stakes — the whole loop, not just the model: pre-flight validation between refresh and live run, loss-limit/halt-flag circuit breakers, staleness detection, transition-only degradation alarms. Understands ML-on-time-series failure modes (walk-forward retraining, regime-aware dynamic labeling, model+encoder bundled to prevent drift). Builds LLM agents with real discipline: bounded steps, finite retry budget, enumerated error categories, constrained verdict vocabulary, deliberately information-only v1.

### stripe-reconciler
Turns a Stripe payout into balanced double-entry journal entries — books each balance transaction, splits revenue from fees, routes unknowns to Suspense, exports a journal that sums to zero.
**Tech:** Flask + PostgreSQL, React 19 + Vite, Stripe Apps auth, bcrypt, flask-limiter, Resend, gunicorn on Render. ~3.6k LOC, 54 commits, six backend test suites gated by CI.
**Transferable capability:** Build a multi-tenant SaaS product end to end and get the unglamorous parts right. Security is reasoned, not cargo-culted: CSRF origin checks with a documented webhook exemption, timing-attack equalization, digest-only single-use reset tokens, origin-derived cookie flags, tenant-isolation tests as a first-class suite. Disciplined domain modeling: integer cents throughout, explicit Suspense account over silent defaults. CI fails closed when no suites are found. Extended this week to scope payouts/journals per Stripe *connection* rather than per user — authorization epochs, connection-scoped uniqueness — so one bookkeeper can manage multiple client accounts without cross-account leakage (six suites, 117 checks, green).

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

### [emerging] — appears once or shallowly, shipped
- Model interpretability and comparison — SHAP, compare_models.py (finance_bot).
- Embeddings and vector retrieval / RAG — CLIP, ChromaDB (visual-search-engine).
- NLP for domain signal — FinBERT sentiment, held dormant as a veto (finance_bot).
- Relational data modeling — reasoned nullability, idempotent uniqueness constraints, inline migration notes (stripe-reconciler).
- Systems programming — processes, IPC, signals, threads, mutexes in C/C++ (CSE4600-HW1).
- Applied web application security — CSRF enforcement with reasoned exemptions, timing equalization, digest-only tokens, single-use resets, webhook signature verification, tenant isolation under test (stripe-reconciler). One repo, unusually deliberate.
- React 19 + Vite SPA against an authenticated API (stripe-reconciler).
- Flutter/Dart mobile — camera capture, on-device inference, local persistence, share-sheet-only export (golf).
- Containerization and deployment — Docker/compose, gunicorn on Render (visual-search-engine, stripe-reconciler, finance_bot).
- AI-assisted development workflow at the tooling level — repo-scoped sub-agents with enforced scopes, guardrail files (golf).

### [conceptual] — understood, reasoned through, NOT yet shipped
*(All from coursework. Genuine understanding, no repo. Do not treat as build-capacity.)*
- **Computer-architecture datapath & tradeoff reasoning** — single-cycle MIPS datapath/control; Iron Law as a working lens; quantitative tradeoff analysis (rejected an ALU multiplier for ~20% slowdown once critical path and cost/perf were computed) (CSE 4010).
- **Real-time scheduling analysis** — EDF, LDF, Rate Monotonic, cyclic executive, polling servers; precedence-graph reasoning; hyperperiod/schedulability analysis (Embedded Systems / MSP432).
- **Hardware–software boundary reasoning** — driver placement, polling vs. interrupt tradeoffs (CPU/latency/power), interrupt signal direction (Embedded Systems).
- **Networking across the full stack** — forwarding vs. routing (data/control plane), transport reliability, IP addressing/subnetting, link-layer mechanics; hands-on Wireshark capture (HTTP caching, DNS, TCP handshakes) (CSE 4100).
- **Circuit analysis** — nodal/mesh/phasor/s-domain; circuit-as-graph modeling; hand-calc vs. PSpice cross-verification with a ~1% mismatch debug threshold (CSE 4030).

---

## Direction & Intent (quarantined — expressed, NOT demonstrated)

### Hardware / computer architecture — target career track
- **Goal:** entry-level hardware/firmware role (IBM named as target). Coursework deliberately reframed around industry requirements.
- **Wants to move into (not yet done):** pipelining w/ hazard detection & forwarding, memory hierarchy/caches, UART/SPI/I2C, signal integrity, power analysis, firmware/embedded, Linux kernel basics.
- **Wants hands-on:** FPGA boards (Basys 3, DE10-Lite), industry tooling (Vivado, ModelSim, Quartus), a documented portfolio (testbenches, waveforms, timing diagrams), team projects.
- **Language intent:** C/C++ for firmware/embedded; also Python/JS proficiency.
- **Embedded next step queued:** timers & PWM (MSP432).

### Stated build intentions (unstarted or in-flight)
- **Visual JS circuit simulator** — self-scoped 5-phase architecture (data model → solver → visualizer → interactivity → dependent sources); deliberately chose the hard version (real analysis algorithms, not faked output); **stalled at the opening design question.** See Keystone.
- **finance_bot Agent v2 ESCALATE** — LLM agent promoting HOLD→SELL on held positions using news/sentiment the model can't see; scoped to held-positions-only to bound failure/cost.
- Verify fractional-share + OTO stop-loss compatibility in Alpaca paper before any live deployment (load-bearing first step for a $1,000 experiment).
- Complete the SELL payoff/breakeven derivation; make an explicit go/no-go accepting single-regime risk.
- Longer-horizon: evolve the hand-rolled ReAct agent toward a framework (CrewAI mentioned).
- Network diagnostics exploration (ping/traceroute/pathping, measured-vs-theoretical throughput); more Wireshark (TCP handshake analysis).

---

## Keystone

The **hardware/architecture axis is all DIRECTION and ZERO SHIPPED INVENTORY.** Every hardware capability is [conceptual]; every hardware goal is quarantined. The single highest-leverage move is to graduate one hardware line from [conceptual] to [strong] by shipping an artifact.

The **self-scoped-but-stalled visual JS circuit simulator is the shovel-ready way to do it.** It sits exactly at the two-axis intersection — proven software-delivery capability applied to the hardware/analog direction — and would unlock a whole category of otherwise-unreachable ideas (hardware-shippable and hardware-teaching). For the passive-income goal, building this keystone is worth more than selecting any single idea, because several of the best ideas depend on it existing.

Still zero shipped inventory as of this refresh. The idea engine proposed the circuit simulator six times (2026-08-01 through 2026-08-06) with no reported progress, then tried two different untested hardware conceptual lines (computer-architecture Iron Law reasoning on 08-08, polling-vs-interrupt reasoning on 08-09) — also unbuilt. None of this is evidence of anything; per this file's own firewall, a proposal is not a capability. The keystone remains open.

---

## Trajectory

Older work (Oct–Nov 2025) is tutorial-shaped: notebook-first ML/CV that stands up a modern stack and stops at the demo. From March 2026 onward the center of gravity moves from *building a model* to *operating a system*: finance_bot, stripe-reconciler, and golf are long-running multi-month repos with CI, tests, design docs, deliberate failure handling. The sharpest trend is epistemic rather than technical — organizing work around knowing what hasn't been validated and refusing to act as though it has. A distinct, newer axis (hardware/architecture/embedded) is now clearly stated as a career direction but carries no shipped inventory yet. Direction: production ML / applied-AI systems engineering, with an emerging, not-yet-shipped pull toward hardware.

---

## last_refresh
2026-08-09
