# passive-ideas-engine

Shared memory for an automated passive-income idea engine.

This repo isn't an application — it's the common ground between two scheduled
Claude routines that never talk to each other directly. Everything one routine
learns, it writes here; everything the other needs, it reads here.

## The loop

**Weekly — refresh the profile.**
The weekly routine reads `profile.md`, looks at its `last_refresh` timestamp,
and scans my repos and project docs for anything that changed since then. It
judges which of those changes represent a *genuine* capability change (something
actually built, shipped, or learned) rather than noise, then rewrites
`profile.md` to reflect the new state — including updating `last_refresh` to the
time of that run.

**Daily — generate ideas.**
The daily routine reads `profile.md` together with `prompts/generate-ideas.md`
and writes a dated idea file into `ideas/`. It doesn't rewrite the profile; it
only consumes it. Each day's output is a separate file, so the history of what
was suggested (and against which version of my skills) stays intact.

```
profile.md  ──reads──▶  daily routine  ──writes──▶  ideas/YYYY-MM-DD.md
    ▲
    └──rewrites──  weekly routine  ──scans──▶  repos / project docs
```

## The tier rule

`profile.md` sorts capabilities into tiers, and the tier is about evidence, not
enthusiasm:

- **`[strong]` / `[emerging]`** — shipped. There is something real that exists
  and works. `[emerging]` is shipped but shallow; `[strong]` is shipped and
  proven.
- **`[conceptual]`** — understood but not shipped. I can reason about it, but
  nothing has been built. Ideas may lean on this only as a stretch, never as a
  foundation.
- **Direction & Intent** — quarantined. Aspirations, interests, and "things I'd
  like to get into" live in their own section and are explicitly *not* treated
  as capabilities. The idea generator must not mistake intent for skill.

## profile.md is both the brain and the state file

There is no separate database, config, or cursor file. `profile.md` holds the
capability model *and* the scheduling state — `last_refresh` lives inside it,
which is how the weekly routine knows what window to scan. Editing `profile.md`
by hand changes both what the engine believes about me and where it will start
looking next time.

## Layout

```
profile.md              the brain + state (last_refresh lives here)
prompts/
  generate-ideas.md     run daily
  refresh-profile.md    run weekly
ideas/                  dated outputs from the daily routine
```
