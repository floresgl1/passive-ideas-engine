# passive-ideas-engine

Shared memory for an automated passive-income idea engine.

This repo isn't an application — it's the common ground between scheduled Claude
routines that never talk to each other directly, plus two manual commands.
Everything one part learns, it writes here; everything another needs, it reads
here.

## The parts

**Daily — generate ideas.** *(cloud trigger, `0 14 * * *`)*
Reads `profile.md`, writes a dated digest of 3–4 tagged ideas to
`ideas/YYYY-MM-DD.md`, commits it directly to `main`, and posts a short version
to Discord. It never rewrites `profile.md` and never touches an existing idea
file. Every idea it emits carries `competence: unlabeled` — it is forbidden from
guessing that value.

**Weekly — three jobs in one run.** *(cloud trigger, `0 17 * * 0`)*
- **PART A** clusters the week's ideas by concept, counts how many *distinct
  days* each cluster recurred, and opens exactly one GitHub issue if the top
  cluster clears the bar (≥3 days, or ≥2 days if it crosses the keystone).
  A quiet week is allowed to be quiet.
- **PART B** scans the connected project repos for work committed since
  `last_refresh`, judges which changes are genuine capability changes, places
  them in the right tier, checks for graduations out of Direction & Intent, and
  rewrites `profile.md`.
- **PART C** counts quiz-queue health across all of `ideas/` and posts it to a
  separate Discord channel — depth, label distribution, labeled-this-week,
  oldest unlabeled, and whether the intake loop has stalled.

**`/quiz` — measure competence.** *(manual, `.claude/commands/quiz.md`)*
The only thing permitted to write `competence` and `labeled_at`. Picks the
oldest unlabeled idea, asks one free-response probe aimed at the idea's hard
part, grades against a fixed bar, falls back to a multiple-choice probe, and
commits one verdict per commit.

**`/resume-line` — generate an outward claim.** *(manual, `.claude/commands/resume-line.md`)*
Reads `career-log.md` and produces résumé/LinkedIn phrasings under a hard claim
bar. Read-only, and firewalled to `career-log.md` as its sole source.

```
profile.md ──reads──▶ daily ──writes──▶ ideas/YYYY-MM-DD.md
    ▲                                        │
    │                                        ├──reads──▶ /quiz ──writes──▶ competence, labeled_at
    │                                        ├──reads──▶ weekly PART A ──▶ one GitHub issue
    │                                        └──reads──▶ weekly PART C ──▶ Discord queue health
    │
    └──rewrites── weekly PART B ──scans──▶ project repos
                       ╎
                       ╎ (not yet wired)
                       ▼
                  career-log.md ──reads──▶ /resume-line ──▶ résumé / LinkedIn text
```

## The tier rule

`profile.md` sorts capabilities into tiers, and the tier is about evidence, not
enthusiasm:

- **`[strong]` / `[emerging]`** — shipped. There is something real that exists
  and works. `[emerging]` is shipped but shallow; `[strong]` is shipped and
  proven.
- **`[conceptual]`** — understood but not shipped. Ideas may lean on this only
  as a stretch, never as a foundation.
- **Direction & Intent** — quarantined. Aspirations live in their own section
  and are explicitly *not* treated as capabilities.

## The firewalls

Three separate rules keep unearned claims from propagating. They exist because
each one guards a place where two different kinds of "movement" look alike:

1. **The daily engine may not guess `competence`.** It generates ideas *from*
   `profile.md`, so any competence it infers is the profile's own claim echoed
   back under a new name — and the quiz exists to test that claim, not a copy of
   itself. `unlabeled` means NOT YET MEASURED, not "no knowledge".
2. **`ideas/` is not evidence of capability** (weekly PART B, Step 0.5). A new
   idea file every day is churn, not activity. `competence: known` means an idea
   could be explained when quizzed — recall of a suggestion, not shipped work.
   Tier promotion requires committed, functional code in a real project repo.
3. **A quiz verdict may never reach an outward claim** (`/resume-line`). Only a
   tier transition licenses a résumé line. `career-log.md`'s schema has no
   `competence` slot at all, so there is nothing for the label to land in.

## profile.md is both the brain and the state file

There is no separate database, config, or cursor file. `profile.md` holds the
capability model *and* the scheduling state — `last_refresh` lives inside it,
which is how the weekly routine knows what window to scan. Editing `profile.md`
by hand changes both what the engine believes and where it looks next.

`career-log.md` is its complement: `profile.md` is a snapshot that gets
rewritten, so it cannot retain the fact that a transition *happened*.
`career-log.md` is that history, append-only.

## prompts/ is a drafting surface, not the thing that runs

The scheduled routines run from prompt payloads stored in their cloud triggers.
`prompts/` holds the repo's copy so the prompts are versioned and reviewable —
but a copy can drift from the payload, and it has: on 2026-08-04 the repo copies
were found missing PART A, PART C, the `ideas/`-is-not-evidence firewall, and
the entire competence prohibition.

The working discipline is: **edit `prompts/` first, paste across, bump the
version marker in the same change.** Each prompt carries a
`<!-- prompt-version: … -->` marker at its foot and instructs the routine to
echo it in its Discord post and commit message, so a stale payload announces
itself instead of being found by reading both copies side by side.

By contrast, `.claude/commands/*.md` **are** the files that run. There is no
second copy, so `/quiz` and `/resume-line` cannot drift — which is why the
`/resume-line` claim bar lives there rather than in a payload.

## Layout

```
profile.md              the brain + state (last_refresh lives here)
career-log.md           append-only history of capability transitions
prompts/
  generate-ideas.md     drafting copy of the daily trigger payload
  weekly-routine.md     drafting copy of the weekly trigger payload (PART A/B/C)
.claude/commands/
  quiz.md               /quiz — the only writer of competence
  resume-line.md        /resume-line — the only writer of outward claims
ideas/                  dated outputs from the daily routine
```
