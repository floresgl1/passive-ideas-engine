<!-- SOURCE OF TRUTH: this file is the DRAFTING SURFACE for the "Ideas - Weekly"
     trigger (trig_01KGvdWC64aUgmam9Y4uxs4U, cron `0 17 * * 0`). The trigger
     payload is what actually RUNS. Edit here first, then paste across, and bump
     the prompt-version marker at the foot in the same change.
     Reconciled from the live payload on 2026-08-04, after the two copies had
     drifted apart unnoticed: this file had lost PART A, Step 0.5, and PART C. -->

# Weekly Sunday Routine — Pick the Winner, then Refresh the Profile

> **BRANCH RULE (critical — the shared-memory loop depends on it):**
> Read the `ideas/` files from the `main` branch, and commit/push the updated
> `profile.md` DIRECTLY to `main`. Do NOT create a new branch and do NOT open a
> PR. `git checkout main`, pull latest, do the work, commit, push to `main`.
> The daily routine writes idea files to `main`; if this routine reads or writes
> any other branch, PART A will see an empty `ideas/` folder and the loop breaks.

> **WRITE SCOPE (critical):** The only files you may write are `profile.md` and
> a GitHub issue. You must NEVER modify, rewrite, or reformat any file in
> `ideas/`. Those files carry quiz results written by another routine; editing
> them destroys measurements that cannot be recovered.

This routine does THREE jobs in one run, in this order:
  PART A — pick this week's build winner from the past week's ideas (looks backward).
  PART B — refresh profile.md from newly-shipped work (looks at what shipped).
  PART C — report quiz-queue health to the quiz channel.
Do them in order. They read different things and don't conflict.

## The idea file format you can rely on

The daily routine emits every idea in exactly this shape:

```
### <N>. <Idea Name> — [<Category>]
- competence: unlabeled
- labeled_at: —

**Idea:** ...
**Leverages:** ...
**One new thing to learn:** ...
**Why it's worth it:** ...
**First step today:** ...
**Keystone:** crosses the keystone — ...   (present only when true)
```

`competence` is one of: `unlabeled` | `known` | `needs work` | `no knowledge`.
`labeled_at` is `—` until a quiz assigns a label, then `YYYY-MM-DD`.
These two fields are machine state written by the quiz routine. You READ them.
You never write them, and you never reason about what they "should" be.

================================================================
## PART A — Pick the Winner
================================================================

Goal: surface the ONE idea worth protecting build-time for this week, but ONLY
if the week's ideas actually justify it. No manufactured momentum — a quiet
week is allowed to be quiet. This mirrors the profile's own discipline: only
claim (open an issue) when the evidence (recurrence) earns it.

### A1 — Gather the week
Read every `ideas/YYYY-MM-DD.md` file dated within the last 7 days. Each holds
~3–4 tagged ideas, so you're clustering ~20–28 ideas total.

### A2 — Cluster by concept (semantic, NOT string-match)
The daily engine rephrases the same idea differently across days. Two ideas
belong to the SAME cluster when they share BOTH:
  - the same underlying capability being leveraged, AND
  - the same rough opportunity type (API / template / teaching asset / tool).
Example — SAME cluster: "productize finance_bot's safety layer" and "sell an
unattended-job harness" (same capability, same product shape). DIFFERENT
clusters: "circuit simulator as a teaching tool" vs "sell the agent template".
Do not over-collapse (everything into one blob) or under-collapse (real
recurrences read as one-offs). Judge by concept, not wording.

Ignore `competence` and `labeled_at` entirely when clustering. They describe
what you know about an idea, not what the idea IS.

### A3 — Count recurrence and apply the threshold
For each cluster, count the number of DISTINCT DAYS it appeared on.
Open an issue for the top cluster ONLY IF:
  - it appeared on **≥3 distinct days**, OR
  - it appeared on **≥2 distinct days AND crosses the keystone** (converts a
    zero-inventory direction — e.g. hardware — into shipped capability).
Frequency is the default signal; keystone is a tiebreaker that lowers the bar.
If two clusters tie, prefer the keystone-crossing one, then the higher day count.

**Competence does NOT affect this threshold or the ranking — at all.**
Recurrence measures the OPPORTUNITY (the engine keeps rediscovering it
independently). Competence measures YOUR READINESS. They are different axes,
and mixing them would corrupt the recurrence signal this whole step rests on.
A cluster you know nothing about can absolutely win. Report competence in A4;
never rank by it.

### A4 — Act
IF a cluster clears the bar:
  Open ONE GitHub issue in this repo (exactly one — ruthless singularity is the
  point). Title: "This week's build: <short name>". Body includes: which
  capability it leverages, how many distinct days it recurred (list the dates),
  why it won, whether it crosses the keystone, and the single concrete first
  step to take.

  Add a **"What you know about this"** line reporting the `competence` values
  across the ideas in the winning cluster — e.g. "2 of 3 unlabeled, 1 needs
  work". This is context for the reader, not a judgment. If EVERY idea in the
  winning cluster is `unlabeled`, say so plainly: "This week's winner was picked
  with no readiness information — none of these ideas have been quizzed yet."

  Then add a **"Suggested starting point"** section to the issue body — a
  scaffold TAILORED to THIS specific winner (not a generic template), derived
  from its concrete first step. Include:
    - a proposed minimal folder/file layout for just the first step (a few
      files, not a whole architecture), and
    - the first 2–3 shell commands to stand it up (e.g. mkdir, git init if a
      new repo, touch the starter files).
  Frame it explicitly as a SUGGESTION, with a line like: "Starting point only —
  adjust the structure to however you'd rather architect it." Keep it minimal:
  the goal is to lower activation energy for the first step, NOT to design the
  whole project or presume the architecture. If the first step is genuinely
  non-code (e.g. "pick a target vertical"), skip the scaffold and just state
  the decision to make.

  If the winning cluster is `unlabeled` or `no knowledge`, append one line to
  the scaffold: "Note: this scaffold assumes an understanding of the idea that
  hasn't been verified yet — consider quizzing it first."
IF NO cluster clears the bar:
  Open NO issue. Instead note "No clear winner this week" and list the top 2–3
  clusters with their day counts, so the shortlist is visible without forcing a
  mediocre build.

Carry the PART A result (issue opened + title, or "no winner") into the Discord
post at the end.

================================================================
## PART B — Refresh the Profile
================================================================

You maintain `profile.md` in this repo — a tiered capability profile that the
daily idea routine depends on. Keep it current WITHOUT inflating it. Honesty
about what has and hasn't shipped is the entire point; an inflated profile makes
the daily idea engine over-reach.

## Step 0 — Read state
Open `profile.md`. Read the `## last_refresh` date at the bottom. That date is
your cutoff: you only care about what changed AFTER it.

## Step 0.5 — FIREWALL: `ideas/` is not evidence (read before scanning)

This repo is one of your scanned sources, and `ideas/` gains a new file EVERY
DAY. That churn is not activity. Nothing in `ideas/` may ever influence a tier:

- An idea appearing in `ideas/` means the daily engine SUGGESTED it. Nothing was
  built. Suggestions are not capabilities at any tier.
- `competence: known` means you could explain an idea when quizzed. That is
  recall of a suggestion — NOT [emerging], NOT [strong], NOT evidence of
  anything shipped.
- `competence: no knowledge` is a fact about one idea, not about your skills.
  It must never demote or remove a demonstrated skill.
- A GitHub issue opened by PART A is a plan, not a shipped artifact.

Tier promotion requires working, committed, functional code in a real project
repo — the rule in Step 2, unchanged.

Concretely: **if the only things that changed since `last_refresh` are files in
`ideas/`, `profile.md` itself, or issues in this repo, then NOTHING substantive
changed.** Say exactly that, update the date, and stop.

## Step 1 — Scope (cheap mechanical filter)
Across my connected repos and project docs, find only items with a
modified/commit time LATER than last_refresh. Ignore everything older — it's
already reflected. Apply the Step 0.5 firewall as you scope. If nothing changed
since last_refresh, say so, update the date to today, and stop.

## Step 2 — Judge (the smart filter — this is the crux)
For each changed item, decide: does this reflect a GENUINE capability change,
or is it trivial (formatting, typos, renames, dependency bumps, doc tweaks)?
Judge by substance, not line count. Only genuine changes proceed.

For each genuine change, place it in the CORRECT TIER — this is the rule that
protects the whole system:
- **[strong]/[emerging]** — only if it SHIPPED in a real repo (working code,
  committed, functional). [strong] = shown across repos or with real depth.
- **[conceptual]** — understood/reasoned but NOT built into a shipped project
  (e.g. coursework theory). Never let this read as shippable capability.
- **Direction & Intent** — a stated goal or "want to build X", NOT proof.
  Keep quarantined, structurally separate from Skills.

## Step 3 — Graduations (check every time)
Look for items in `Direction & Intent` that newly-shipped work now PROVES. If
an aspiration has become a shipped artifact, MOVE it out of quarantine into
Skills at the right tier. This is the most valuable update you can make —
especially any hardware/keystone line graduating from [conceptual] to [strong].
Flag each graduation explicitly.

## Step 4 — Preserve invariants
- Keep the epistemic through-line as ONE [strong] skill with its META-SKILL
  guardrail note (quality-shaper + teaching wedge, never a standalone product).
  If new work shows it in another domain, update the domain count; don't
  duplicate the entry.
- Keep the `## Keystone` section accurate. If the keystone artifact shipped,
  update it to reflect that the hinge has been crossed.
- Keep `## Trajectory` a true 2–3 sentence read of current direction.
- Do NOT delete demonstrated skills that didn't change this week.

## Step 5 — Write back state
Set `## last_refresh` to today's date (YYYY-MM-DD). Commit the updated
profile.md.

================================================================
## PART C — Quiz Queue Health
================================================================

You have already read the week's idea files. Now read ALL files in `ideas/`
(not just the last 7 days) and count, mechanically:

- **Queue depth** — total ideas with `competence: unlabeled`.
- **Label distribution** — counts of `known` / `needs work` / `no knowledge`.
- **Labeled this week** — ideas whose `labeled_at` falls within the last 7 days.
- **Oldest unlabeled** — the date of the earliest idea file still containing an
  `unlabeled` idea.

These are counts, not judgments. Do not label anything, do not infer what a
label should be, and do not edit any file in `ideas/`.

Then assess ONE thing: **is the intake loop running?** If queue depth is
non-zero AND "labeled this week" is zero, the loop has stalled — no ideas are
being quizzed while new ones keep arriving. Say so directly.

================================================================
## Output — two separate Discord posts
================================================================

### Post 1 — ideas channel
Post the combined PART A + PART B summary via webhook:
IDEAS_WEBHOOK_URL

Structure it in two short sections:

**This week's build** (from PART A):
- If an issue was opened: the title, the winning capability, and how many days
  it recurred. Lead with it if it crosses the keystone. Include the one-line
  competence context.
- If not: "No clear winner this week" + the top 2–3 clusters and day counts.

**Profile refresh** (from PART B):
- What changed, what tier each change landed in, any graduations (especially a
  hardware line moving [conceptual] → [strong], called out clearly), and any
  duplicates collapsed. If nothing substantive changed, say exactly that.

Keep the whole message readable in one Discord message. Do NOT include queue
counts here — those go to the quiz channel.

### Post 2 — quiz channel
Post the PART C report via a DIFFERENT webhook:
QUIZ_WEBHOOK_URL

This is a separate channel with a separate purpose. Keep it short — four or
five lines:
- Queue depth (unlabeled ideas waiting).
- Label distribution.
- Labeled this week.
- Oldest unlabeled idea's date.
- If the loop has stalled (queue non-zero, nothing labeled this week), lead with
  that in one plain sentence.

If queue depth is zero, post one line saying the queue is clear.

Do not post quiz questions here and do not quiz anything — that is a different
routine's job. This post is status only.

Also commit the updated profile.md as described in PART B Step 5.

================================================================
## Prompt version — echo it on every run
================================================================

Include the `prompt-version` marker from the foot of this prompt, verbatim, as
the LAST line of Post 1 (ideas channel), and append it to the `profile.md`
commit message.

The repo keeps this prompt at `prompts/weekly-routine.md`. That copy and the
running payload have already drifted apart once, silently, and the gap was only
found by reading both side by side. Echoing the marker makes any future
divergence self-announcing: if a run's marker doesn't match the marker in the
repo copy, the payload is stale. No audit required to notice.

<!-- prompt-version: 2026-08-04.1 -->
