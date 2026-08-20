<!-- design-version: 2026-08-20.1 -->
# Tutor-Conversion Design

**Status:** Recorded design, not a task list. **Correction (.3): repo-discovery
does NOT gate #0.** #0's cluster side draws only from `ideas/` and
`quiz-queue.json` — neither touches project repos, so there's no allowlist to be
pinned to. Discovery starts mattering only at the rung record's `ship_target`
field ("what shipped artifact discharges this rung"), which is #7. The real gate
on #0 is **evidence that concept identity is stable enough to be worth a primary
key** — see "The split-rate precondition" below. The async loop has now closed
two full cycles (both `no knowledge`, 2026-08-17); split-rate observation begins
**2026-08-23** at the first re-clustering.

**Revision note (2026-08-15.2):** Folds in a repo-checked critique. Changes from
.1: added **#0 concept identity** as the true bottom of the stack; **split #5**
into a freshness leak (spacing fixes) and an answer-key leak (spacing does not);
gave **#1** a named home via the `career-log.md` precedent; replaced **#2**'s
storage question with an **arbitration** question (storage subordinate); fixed
the **`unlabeled`** vocabulary error and the **#4** routing that depended on it;
added a **third fork option** (event-couple a request record, not the material);
added a **remediation firewall invariant**. The .1 shape (trigger-based
separation, #5-as-acceptance-on-#2, #7 deferred) stands.

**Revision note (2026-08-15.3):** Corrected the gate — discovery does not gate
#0 (only #7's `ship_target`). Settled the **merge/split policy** (the load-bearing
weakness of the ID scheme) and, in doing so, surfaced the **split-rate
precondition**: whether #0 is a foundation or an ornament is an *empirical*
question no existing data can answer. Both new sections at the end. Everything
above stands; the "build order" is now explicitly gated on evidence, not a
keyboard or discovery.

**Revision note (2026-08-16.1):** No design conclusion changes. This revision is
entirely about what is and is not blocking, and it corrects one wrong assumption
that was quietly costing weeks.

- **Prerequisite 1 (reconcile the weekly payload) is CLOSED and verified.**
- **Prerequisite 2 is REWRITTEN.** .3 framed "let the loop close a few cycles" as
  something that would happen given time. It would not have. The loop had never
  closed because **nothing was scheduled to run it** — the `Quiz - Probe` and
  `Quiz - Collect` triggers specified in `docs/quiz-bot-setup.md` §4 had never
  been created. The prompts existed; the runners did not. Both have now been
  created, but **the loop has still not closed once**, so nothing downstream of
  that evidence has moved.
- **Stale sequencing claim corrected.** The summary table's footnote still said
  "nothing here is built until repo-discovery lands," which .3 had already
  overturned in its own header. Fixed, because left standing it points the next
  reader at #1 — the wrong blocker.

**Revision note (2026-08-20.1):** No structural changes. This revision settles
the four open design decisions and updates prerequisite 2 to reflect the loop
having closed.

- **Prerequisite 2 is DEMONSTRATED.** The quiz loop closed two full cycles on
  2026-08-17: *Bounded Risk-Escalation Agent Template* (probe → free-response
  miss → MC → `no knowledge`) and *Dependent-Source SPICE CLI* (same path →
  `no knowledge`). Both `quiz-queue.json` entries are now `"labeled"`;
  `quiz-state.json` is `"idle"`. The runners work. The next queue stock comes
  from the 2026-08-23 weekly re-clustering.
- **Decisions 1, 2, 3, 5 are SETTLED.** See "Settled decisions" section below.
  All four decisions in the waiting list that were open in .1 are now closed;
  only the split-rate evidence gates further work.

---

## The threshold

The engine was built to **generate**: open-ended, outward — "here are new
things to build." A tutor **closes loops**: targeted, inward — "here's the
thing you don't know yet; now do you know it?"

Those are different machines. Several assumptions built for the generator break
under the tutor. This doc records what has to change.

The whole conversion in one line:

> Concept identity (#0) gives history (#1) a key; history enables loop-closing
> (#2); the loop isn't trustworthy until it defeats **both** the freshness leak
> (#5a, spacing) **and** the answer-key leak (#5b, re-framing) that the repo
> already predicted.

---

## The remediation routine is separate from idea-generation

Confirmed decision: remediation is its own routine, **not** folded into
`generate-ideas.md`. Different triggers, different input widths:

- `generate-ideas.md` — **time-triggered** (cron, daily). Input: whole profile
  + repo state. Generative, open-ended.
- Remediation — **event-triggered** (a `/quiz` verdict came back weak). Input:
  narrow — one concept, one tier, one failure. Targeted, closed.

This is the same cut the repo already made between `quiz-probe.md` and
`quiz-collect.md`: split by *when it runs*, not by *what it's about*.

### The fork: what triggers remediation — three options, not two

1. **Poll** — scheduled routine reads `quiz-state.json`, finds weak entries,
   generates material. Must track "already remediated this?" or it re-delivers.
2. **Event-couple the material** — emit the primer in the same flow that writes
   the weak verdict. Keeps same-commit auditability, but **puts generative work
   (#4) on the critical path of a measurement you already have.** `quiz-collect.md`
   Step 4 requires verdict + state + queue to land in one commit precisely so a
   measurement can't be lost or double-graded; wedging a slow, failure-prone
   generation step into that transaction risks the measurement to produce the
   primer. Rejected for that reason.
3. **Event-couple a *request record* (preferred)** — write a cheap, deterministic
   remediation-request record in the *same commit* as the verdict; a *separate*
   routine turns requests into material later. Keeps the same-commit-coupling
   argument in full, keeps generation off the measurement's critical path, and
   dissolves the "already remediated?" bookkeeping because **the request record
   *is* that state.** This is the `career-log.md` + `profile.md` coupling applied
   to the derived *request*, not the derived *material*.

Landed: **option 3.**

---

## The foundation (load-bearing — build first)

### #0 — Concept identity (the true bottom of the stack)
History has to attach to *something durable*. **The system has no such object.**
What it has is a queue entry pointing at one `idea_file` + byte-exact
`idea_heading`, chosen as the cluster's most recent framing (`weekly-routine.md`,
A7). Clusters are re-derived semantically by an LLM every Sunday and carry no ID;
next week the same concept resurfaces as a different heading in a different dated
file. Consequences:

- **"Weak for the third time despite two primers" has no key.** You can't count
  failures against a thing that gets re-identified weekly.
- **A7 skips clusters already `status: "labeled"`**, so a re-quiz path can't
  re-enter through the normal queue without contradicting that rule.

#1 is a table that needs a primary key, and the key doesn't exist. Concept
identity is therefore *upstream* of history and is the real first build. (It is
also the same missing object #7 needs — see Deferred.)

### #1 — Learning-state history
Once #0 gives concepts a durable key, attach **history** to it: last-failed
timestamp, failure count, material already given, whether it helped. Without
history, remediation can't distinguish "first time weak" from "weak despite two
primers."

**Named home — it needs a fourth file, and the repo already has the precedent.**
None of the three existing files can hold this:
- `ideas/*.md` — both quiz paths may write *only* the two lines under one `###`
  heading (`quiz.md:17-22`, `quiz-collect.md:20-25`). History can't go here
  without breaching that scope.
- `quiz-queue.json` — justified as writable precisely because it "is a DERIVED
  index that points AT ideas, and holds no measurement of its own"
  (`weekly-routine.md:22-25`). History *is* measurement; putting it here voids
  that justification.
- `profile.md` — a snapshot rewritten weekly, firewalled from quiz verdicts.

The README's own reasoning for `career-log.md` transfers verbatim: *profile.md is
a snapshot that gets rewritten, so it cannot retain the fact that a transition
happened.* An idea file's two lines are that same snapshot. So:

> `career-log.md` : `profile.md` :: **`<history file>`** : `ideas/*.md`

#1 is therefore an **append-only measurement-history file**, shaped like
`career-log.md`, keyed by #0's concept identity.

### #2 — Loop-closing / re-test path
A weak concept, once remediated, needs a route back to `/quiz` to prove the
material worked — otherwise material is delivered into a void.

**The real open question is arbitration, not storage.** `quiz-state.json`
enforces exactly one outstanding probe (`quiz-probe.md` Step 0); selection is
strictly oldest-pending; A7's recurrence gate exists because "Ideas generate at
~4/day and no human answers 28 probes a week." **The throughput ceiling is
attendance, not software.** Every re-test spends the single open slot that would
otherwise measure something never measured. So the question isn't "new file or a
field" — it's: **when a re-test and a never-measured concept are both due, which
gets the slot, and does remediation starve intake or vice versa?** That policy
decision determines whether the tutor conversion improves the system or just
consumes it. Storage falls out of the arbitration answer (and intersects #0's
identity record); arbitration does not fall out of storage.

### #5a (acceptance on #2) — The freshness leak
A tutor that *just showed a primer on X* then re-tests X: you pass because it's
fresh, not because you understand. **Spacing fixes this** — needs #1's history
and #3's cooldown.

### #5b (acceptance on #2) — The answer-key leak (spacing does NOT fix this)
`quiz-collect.md:158` predicted this in advance: a labeled entry leaves the queue
permanently, *"If a decay routine is ever built to re-quiz labeled ideas… the
channel will by then hold a written answer key, and that routine must not draw
its probes from ideas whose mechanism was taught here."* **The tutor conversion
is that routine.** Both grading paths teach the mechanism *after* the verdict
(`quiz.md` Step 7, `quiz-collect.md` Step 5), so by re-test time the Discord
channel already holds a persisted answer key — and remediation material adds a
second. **An answer key doesn't decay; cooldown does nothing here.** A re-test
that clears #5b must either aim at a *different hard part* than the one taught,
or accept that it can only ever measure **recognition, not production.** This is
a distinct, second constraint on #2 — the one the repo already predicted and the
one #3 cannot discharge.

---

## The layers (make it good — build after the foundation)

### #3 — Cooldown / spacing
Don't re-surface or re-test too soon. Discharges #5a only. **Resist full SRS** —
a crude cooldown field prevents the worst behavior.

### #4 — Material type contract
Two forms to start — "explain" vs "drill" — chosen by history. **Routing fix:**
tier `unlabeled` must **not** route to exposition. Per README firewall #1,
*`unlabeled` means NOT YET MEASURED, not "no knowledge"* — it's off the ladder,
not rung zero. A never-seen concept needs a **probe**, not a primer; routing it
to explanatory material hands the answer to something never measured. Only
genuinely-measured-weak concepts (`no knowledge` / `needs work`) route to
material.

### #6 — Channel type-marker
Cheaper than .1 stated: the channels are **already split** (`QUIZ_WEBHOOK_URL`
vs. the ideas channel, per `quiz.md`'s closing rule). A type marker only earns
its keep *if* remediation shares a channel with ideas — otherwise **the channel
is the marker.** Decide channel placement first; the marker may be free.

### Firewall invariant (new — remediation adds an edge the three firewalls miss)
Remediation creates: verdict → material → re-test → "I demonstrably learned
this." State explicitly: **a post-remediation `known` is exactly as
non-load-bearing as a cold one.** Remediation history gets *no* path into
`profile.md` tiers or `career-log.md`. Firewall #2 (`ideas/` is not evidence) and
#3 (a verdict may never reach an outward claim) are **unchanged** by the tutor.

---

## Deferred

### #7 — RTL-ladder tiers in the tutor vocabulary
The RTL ladder (sequential logic → FSMs → datapath component → control+datapath →
simple CPU core) needs its rungs as gradeable concepts. **Blocked by more than
discovery:** every quizzable thing is anchored to `idea_file` + `idea_heading`,
and probe generation reads the idea's body to find its hard part. RTL rungs have
no idea file — so #7 needs a **concept record decoupled from `ideas/`**, which is
**the same missing object as #0.** #7 and #0 may be one item; if so, discovery is
gating *less* than .1 assumed. Build rung 1 as a plain repo first — an honest,
recorded gap.

---

## Sequencing summary

| Item | Role | When |
|------|------|------|
| #0 Concept identity | Foundation (bottom of stack) | **First** — may be one item with #7 |
| #1 History (append-only file, career-log-shaped) | Foundation | After #0 |
| #2 Loop-closing (arbitration is the question) | Foundation | After #1 |
| #5a Freshness leak | Acceptance on #2 (needs #3) | With #2 |
| #5b Answer-key leak (re-framing, not spacing) | Acceptance on #2 | With #2 |
| #3 Cooldown | Refinement (discharges #5a only) | With #2 |
| #4 Material types (`unlabeled` → probe, not primer) | Layer | After foundation |
| #6 Channel marker (may be free) | Trivial | Anytime |
| Firewall invariant | Invariant | With #2 |
| #7 RTL tiers | Deferred (needs #0's object) | After #0 + discovery |

**This is 5+ of the items sitting at the foundation — not a small first build.**
Captured so it's ready when the build unblocks — not a signal to start.

**Corrected in .1:** this footnote previously read "nothing here is built until
repo-discovery lands." That contradicted .3's own header correction and is wrong.
Repo-discovery gates only #7's `ship_target`. The live gate is the split-rate
evidence described below, which cannot begin accumulating until the quiz loop
closes at least once.

---

## Merge/split policy (#0's load-bearing weakness — now settled)

Sunday's weekly routine re-clusters ideas **semantically, via LLM**, and mints
concept IDs that are history's primary key. So when clustering changes, the key
changes underneath the history rows that point at it. Merge and split are **not
symmetric**; the asymmetry is about *recoverability*, not distortion.

**Merge (C1 + C2 → C7): lossy but honest → recoverable.**
You still have both input records; you know exactly what fed C7. The danger is
*careless combination* (e.g. adding failure counts and overstating weakness).
Fixable with a non-fusing rule.

> **Merge rule:** preserve provenance — do NOT fuse the two histories into one
> combined count. Record that both original concepts now co-point at C7, keeping
> each provenance chain intact. Combination is reversible because nothing is
> destroyed.

**Split (C1 → C1a + C1b): lossy AND unrecoverable → must not inherit.**
The LLM has asserted that past "C1 failed" records were *never really about one
thing*. But a pre-split record cannot assign itself to C1a or C1b: **at the time
it was recorded, the distinction the LLM just made did not exist.** Any
assignment is a fabrication — an unvalidated claim about what the failure
measured — which is exactly what measurement-over-verdict forbids. No checkout
state or branch recovers it; the information was never captured because the
category didn't exist yet.

> **Split rule:** pre-split history is NOT inherited by either child. It stays
> sealed on the retired parent as a **tombstone** ("these failures happened
> against a concept since judged to be two things; they cannot be honestly
> attributed to either"). Both children start with **empty** history.

**The teeth (accepted, eyes open):** every split resets both children's
failure-counts to zero. Repeat observations are already rare by design (A7 gate +
one-outstanding-probe lock + 21-day expiry throttle to ~one measurement per
multi-day cycle). A weekly re-clustering that can wipe an accumulated count means
#1's failure-count field — the entire reason #1 exists — could stay empty
**indefinitely**, not for lack of failures but because the identity it hangs on
keeps dissolving before a count can build.

---

## The split-rate precondition (the real gate on #0)

The merge/split reasoning did not unblock #0 — it revealed the actual
precondition, sharper than "wait for a keyboard" or "wait for discovery":

**Whether #0/#1 is a foundation or an ornament is an empirical question that
depends on how often real clustering splits vs. holds concepts stable across
weeks — and there is zero data on this, because the loop has never closed a full
cycle, let alone run enough weeks to observe re-clustering behavior.**

- If concepts are **mostly stable** week-to-week → history accumulates → #0 is a
  foundation, build it.
- If splits are **frequent** → history never fills → the tutor should collapse to
  the **stateless "explain what you just failed"** version: no #0, no #1, no
  history file at all.

This is a **fork you must pick from evidence, not a fact you can passively wait
for** — because under the current throughput design the stabilizing evidence may
never arrive on its own. Picking it requires watching several weeks of actual
Sunday re-clustering and measuring the split rate.

### What actually moves #0 forward (both cheap, both in your control)

**1. Reconcile the drifted weekly payload. — CLOSED 2026-08-16.**

Concept-ID minting will live in A7, inside the weekly payload, so designing
minting against `weekly-routine.md` while that file was not what executes would
have been designing against fiction. Both payloads were pasted across and then
**read back and verified identical** to their repo copies:

| Trigger | Was | Now |
|---|---|---|
| Ideas - Weekly | `2026-08-04.2` | `2026-08-10.1` |
| Ideas - Daily | no marker at all | `2026-08-14.1` |

The weekly payload now actually contains A7. Worth recording plainly: **the live
weekly routine had never once stocked the queue on its own.** Both existing
`quiz-queue.json` entries exist because a session noticed the staleness and
hand-overrode its own payload — four separate runs did this rather than fix it,
because the tracking issue (#13, now closed) misdiagnosed the blocker as "session
tooling can't write the trigger config." Reading the live payload was always
possible; the write is blocked by an ownership rule (`created_via: http_api`).
A wrong blocker note cost twelve days. The design lesson generalizes: **record
why something is blocked precisely, or the note becomes the blocker.**

**2. Let the async loop close a few real cycles. — CLOSED 2026-08-17.**

.3 treated this as a waiting problem. It was a wiring problem. The loop had never
closed because **no scheduled runner existed**: `prompts/quiz-probe.md` and
`prompts/quiz-collect.md` were drafting surfaces for the `Quiz - Probe` and
`Quiz - Collect` triggers described in `docs/quiz-bot-setup.md` §4, and neither
trigger had ever been created. `quiz-state.json` sat `idle` while entries
accumulated — not a throughput ceiling, an absent process.

Both triggers were created 2026-08-16, with the Discord bot and its environment
variables. Two full cycles closed on 2026-08-17:

| Entry | Probe posted | Free-response | MC probe | Verdict |
|---|---|---|---|---|
| Bounded Risk-Escalation Agent Template | 08-16 23:31 | missed bar | posted 08-16 23:57 | `no knowledge` (08-17 00:06) |
| Dependent-Source SPICE CLI | 08-17 02:05 | missed bar | posted 08-17 14:37 | `no knowledge` (08-17 17:49) |

Both `quiz-queue.json` entries are now `"labeled"`. `quiz-state.json` is back to
`"idle"`. The runners are verified working. The next queue stock comes from the
2026-08-23 weekly re-clustering (A7).

**The failure mode to continue watching is silent.** If the bot's MESSAGE
CONTENT INTENT is off, every reply reads as an empty string; a blank is not a
pass, so every answer routes to the MC probe. That yields verdicts that look
legitimate and measure nothing — which would poison the very split-rate data
this section exists to collect. The tell: you answer in real words and get a
multiple-choice question back. Both cycles above routed to MC; that is
consistent with either "the free-response genuinely missed the bar" or "the
intent was off and the content was blank." The next cycle should be watched
for this.

### Settled decisions (all four closed as of 2026-08-20.1)

**1. Minting authority + write-scope amendment. — SETTLED.**

The weekly routine mints concept records; it may **never** write history. Only
grading paths (`quiz-collect.md`) write history. This follows the `quiz-queue.json`
precedent: the queue was justified as writable by the weekly routine precisely
because *"it is a DERIVED index that points AT ideas, and holds no measurement of
its own"* (`weekly-routine.md:22-25`). A concept-identity record is the same kind
of object — a derived index that says "these ideas are the same concept," not a
measurement of what anyone knows.

Write-scope amendments when #0 is built:

- **`weekly-routine.md`** adds: the concept-record file (e.g. `concepts.json`).
- **`quiz-collect.md`** adds: append-only writes to the history file, scoped to
  the single concept named in `quiz-state.json`.
- **`quiz-probe.md`** adds: nothing. It reads concept records for
  selection/arbitration (#2) but never writes them.

The load-bearing line: **the weekly routine may never touch the history file, even
to initialize a record.** An empty history is "never measured," which is true, not
a gap to backfill. This mirrors the existing rule that the weekly routine never
touches `ideas/` files.

**2. Epoch decision. — SETTLED.**

History is forward-only. The tutor's epoch = the date the first concept ID is
minted (which will be the first Sunday after #0 is built). Everything before is
permanently outside the loop.

The eight pre-epoch verdicts (Backtest Validity Toolkit, Secure Multi-Tenant SaaS
Starter Kit, Swing-Check API, Unattended-Pipeline Watchdog, Circuit Sandbox, plus
Bounded Risk-Escalation Agent Template and Dependent-Source SPICE CLI from the
queue era) were all measured before concept IDs exist. Even the two queue-era
verdicts were identified by `idea_file` + `idea_heading`, not by a durable concept
key — attributing them to a future concept ID would be fabricating a relationship
that was never captured.

On launch day, every concept's history starts at zero. That is honest — the first
measurement under the new system is the first honest one.

**3. Poisoned-by-default. — SETTLED.**

Every verdict to date has a teach-post in the Discord channel (`quiz-collect.md`
Step 5, `/quiz` Step 7). Those are permanent, unretractable answer keys (#5b). The
concepts they cover can never be honestly re-tested by drawing from the channel
history, because the mechanism was taught in the same place the reply will be read.

Accepted consequences:

- At tutor launch, the set of concepts eligible for re-testing is **legitimately
  zero**.
- The first re-testable concepts will be ones quizzed *after* the tutor has a #5b
  strategy (re-framing to test a different hard part, or accepting
  recognition-only measurement).
- **#2's acceptance criteria must be written to expect an empty re-test queue at
  launch.** A test that asserts "the re-test path works when it has items" will
  fail on day one and look broken when it is actually correct.

This is exactly the scenario `quiz-collect.md:158` predicted. Making it an
explicit acceptance criterion closes the loop on that prediction.

**4. Merge/split policy. — SETTLED** (in .3, see above).

**5. Tolerance for a stale writer. — SETTLED.**

The paste-across deployment model means there is always a window where one trigger
runs the new payload and another runs the old. Two rules make the history file safe
against a writer that does not know it exists:

> **File format:** JSONL — one JSON object per line, one record per append. Never
> a structured JSON object that requires reading to update. An append cannot
> corrupt what is already there, and a writer that does not know about the file
> simply does not append — no damage.

> **Reader contract:** every reader treats "no record for this concept" as "never
> measured." This is the correct interpretation (absence of evidence, not evidence
> of absence), so it is not a special case — it is the default. No reader should
> ever distinguish "the file exists but has no entry" from "the file does not
> exist."

Worst case during rollout: a grading event happens while `quiz-collect` is still
running the old payload. The verdict lands in `ideas/` (that path is unchanged)
but no history entry is appended. That is an **undercount**, not a miscount — the
concept looks like it has fewer measurements than it does, which biases toward
re-testing it sooner than necessary. That is the safe direction. And the rollout
window is short (minutes), so the probability of a grading event during it is low
— the quiz loop fires three times a week.

**Stopping point:** #0 is designed, its hardest question (merge/split) is
settled, and all five waiting-period decisions are now closed. It is not buildable
yet — not for lack of decisions, but for lack of evidence that concept identity
is stable enough to deserve a primary key.

**As of .1 the loop was unblocked but untested. As of .2 (2026-08-20) the loop
has closed two full cycles** — both on 2026-08-17, both `no knowledge`. The
runners are verified working. Both queue entries are consumed. The queue will be
restocked on **2026-08-23** (next Sunday's re-clustering), and that is the first
opportunity to observe whether the weekly routine holds concepts stable or splits
them.

Distinguishing "concepts hold" from "concepts split" needs concepts recurring
across multiple weeks, not one. The earliest an honest foundation-vs-ornament
call could be made is several Sundays out.

**Nothing in "The foundation" section should be started before that call.** There
is no remaining design work this doc can license today — only observation.
