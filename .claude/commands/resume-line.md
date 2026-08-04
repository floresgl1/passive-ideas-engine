---
description: Generate résumé/LinkedIn phrasings from a career-log.md record, under a hard claim bar.
---

# Résumé Line — turn a logged event into a claim you can defend

`career-log.md` records **facts**. This command produces **claims**. The split
exists because a fact can be written automatically and a claim cannot: a model
phrasing a résumé line drifts generous exactly the way a grader drifts generous,
and this is the one output in the system a stranger acts on.

The bar below is the whole command. Everything else is plumbing that keeps you
from routing around it.

> **READ-ONLY (critical):** this command writes NOTHING. Not `career-log.md`,
> not `profile.md`, not a file in `ideas/`, not a scratch file. Its entire
> output is text in this conversation. If a line is good, you paste it
> somewhere yourself. An unreviewed claim must not be able to outlive the
> session where you could still catch it.

> **SOURCE FIREWALL:** `career-log.md` is the ONLY permitted source of material.
> Not `profile.md` — it contains `[conceptual]` items and a `Direction & Intent`
> section, which are precisely the things that must never reach outward-facing
> text. Not `ideas/`, not git history, not anything you remember from earlier in
> this conversation. If it isn't in a log record, it cannot appear in a line.

---

## Step 1 — Read the log

Read `career-log.md`. If it has no records, say so in one line and stop. Do not
invent an event, and do not offer to reconstruct one from the repos — an event
exists because PART B wrote it, or it does not exist.

## Step 2 — Select ONE record

- No argument → the most recent record.
- An artifact name or a date → that record.
- A number → that many records, newest first, handled **one at a time**.

**One record, one claim.** Never merge two records into a single line. Fusing
"shipped a circuit solver" and "shipped a trading system" into "built ML and
hardware systems" is the highest-yield inflation available to you, because
neither half is false and the whole is a bigger claim than either record makes.

## Step 3 — Enumerate the material BEFORE phrasing anything

Write out, from the record alone:

- **Nouns available** — the artifact, plus every concrete noun in `evidence`.
- **Verbs unlocked** — by tier, per Step 4's cap. Just the permitted list.
- **Numbers available** — every figure in `evidence`, verbatim.

A word that is not on this list cannot appear in the line. Doing this first is
what makes the bar enforceable instead of aspirational; if you phrase first and
check afterward, you will check the sentence you already like.

## Step 4 — The bar

Verbatim, and it does not bend:

> A line may claim only what the record's `evidence` field literally supports.

All four checks must pass:

1. **Every noun traces to the record.** "Platform", "framework", "pipeline",
   "system", "suite" are permitted only if `evidence` names one. A solver is a
   solver.
2. **Scope verbs are capped by tier.**
   - → `[emerging]` unlocks: built, shipped, wrote, implemented, validated.
   - → `[strong]` additionally unlocks: designed. `[strong]` means *shown
     across repos or with real depth* — the record must show that, not merely
     carry the tag.
   - **architected** needs `[strong]` AND an `evidence` token naming a design
     artifact that preceded the build: a design doc, decision record, or
     written contract. Without one, "designed" and "architected" are
     interchangeable and the stronger word wins by default — which is drift
     with extra steps. The token makes the choice checkable.
   - **Never unlocked by any tier alone:** led, owned, at scale, production,
     production-grade, enterprise, in production. These describe circumstances
     — users, traffic, a deployment — and a commit range cannot supply them.
     They require an `evidence` token that names the circumstance.
3. **No adjective the evidence cannot falsify.** robust, scalable, performant,
   efficient, seamless, comprehensive, sophisticated, cutting-edge. Banned
   unless a number in `evidence` backs that specific claim. `14 tests green`
   licenses "test-covered"; it does not license "robust", and it does not
   license "comprehensive" — that is a judgment about coverage the number
   doesn't make.
4. **The outward stranger test.** Someone reads the line, then opens the repo.
   They must not feel misled. This is not "is each word individually
   defensible" — it is "does the impression the sentence creates match what is
   actually there." A line can pass checks 1–3 word by word and fail here.

**Tiebreak: when torn between two phrasings, take the weaker one.** The stronger
phrasing's upside is a marginally better bullet. Its downside is walking a claim
back in a room with someone who read the repo.

**A thin record produces a short line.** Length is not a goal. If the honest
line is nine words, ship nine words. Padding a sparse record is the most common
way inflation enters, because a short bullet *feels* weak — that feeling is not
evidence.

**If the record supports no defensible claim, say so and stop.** An event can be
real and still not be résumé material. That is a normal outcome.

## Step 5 — Produce the lines

Two registers, same record, same material list, same bar:

- **Résumé bullet** — past tense, verb-first, one line, carries the number if
  there is one.
- **LinkedIn** — first person, may say what it unlocked or why it mattered.

LinkedIn's looser register is where the bar gets quietly dropped. It is the same
bar. A casual voice is not a lower standard of truth.

## Step 6 — Show the trace, and show what you refused

After each line, print:

- **Licensed by** — the `evidence` token behind each load-bearing word. A word
  with nothing to point at is the drift, made visible.
- **Refused** — the stronger phrasing you considered, and the exact token that
  was missing. If you refused nothing, you did not test the line; generate the
  inflated version, name why it fails, and discard it.

---

## Worked example — one record, both ways

```
## 2026-08-11 — circuit-solver
- axis: hardware
- transition: [conceptual] → [emerging]
- artifact: circuit-solver
- evidence: floresgl1/circuit-solver, 31 commits (a3f1c2..9de4b7), nodal
  analysis solver over a graph circuit model, 14 tests green, hand-calc vs
  PSpice cross-check agreeing within 1%
```

**BAD:**

> Architected a production-grade circuit simulation platform with robust
> numerical solvers and comprehensive test coverage.

- "Architected" — check 2. Tier is `[emerging]`; only `[strong]` unlocks it.
- "production-grade" — check 2. Never tier-unlocked; needs users or a
  deployment, and `evidence` has neither.
- "platform" — check 1. The record says *solver*. No such noun exists.
- "solvers" — check 1. One solver. The plural invents inventory.
- "robust", "comprehensive" — check 3. Unfalsifiable. `14 tests green` is a
  number; "comprehensive" is a verdict about coverage that the number does not
  deliver.
- And check 4 on its own: someone who opens a 31-commit repo after reading
  "platform" has been misled, which is the entire failure in one word.

**GOOD:**

> Built a nodal-analysis circuit solver over a graph-based circuit model
> (31 commits, 14 tests); cross-checked solver output against PSpice to within
> 1%.

Every noun — solver, circuit model, commits, tests, PSpice — is in `evidence`.
"Built" is unlocked by `→ [emerging]`. Both numbers are verbatim.

**The subtle one, which matters more than the obvious puffery.** An earlier
draft of the GOOD line read *"14 hand-calculated PSpice cross-check cases."*
That fuses two separate `evidence` tokens — `14 tests green` and `hand-calc vs
PSpice cross-check` — into one stronger claim the record never makes. The
record does not say the 14 tests are the cross-check.

**Fusion is the subtlest inflation and the one you will write by accident.**
Each token is true; the sentence joining them is not in the record. Check every
conjunction: if two evidence tokens end up in one clause, confirm the record
actually relates them.

---

## What you must never do

- Never write to any file. This command is read-only, and that is structural,
  not stylistic.
- Never source material from anywhere but `career-log.md`.
- Never generate a line for an event that is not in the log.
- Never merge two records into one claim.
- **Never treat a `competence` label as evidence.** `competence: known` in
  `ideas/` means the person could reconstruct an idea's mechanism from memory.
  It does not mean anything was built. There is no path from a quiz verdict to
  a résumé line — the tier transition in `career-log.md` is the only thing that
  ever licenses one.
- Never lower the bar because the target is "casual", a draft, or "just for
  LinkedIn."
- Never restore a phrasing the bar rejected because the user liked it better.
  Say what token would license it; that is a real answer, and it points at work
  worth doing.
