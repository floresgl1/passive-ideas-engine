---
description: Manage the current build — start a new one, mark it done, or abandon it.
---

# Build Manager

This command manages `current-build.json` — the record of what you're actively
building. The Mon/Thu idea routine reads it to steer generation around your
current focus, and the weekly routine reports build age in the Sunday Discord
post.

> **BRANCH RULE:** work on `main`. Commit and push `current-build.json` directly.

## Parse the argument

The invocation carries a subcommand and optional argument:

- `/build` (no argument) — show current state
- `/build start` — start the picked winner, or ask for a name if none
- `/build start <idea name>` — start building a specific idea
- `/build done` — mark the current build as shipped
- `/build abandon` — mark as abandoned, with a reason

---

## No argument — show state

Read `current-build.json` and report in one block:

- If `active: true, status: "building"`: what's being built, when it started,
  how many weeks it's been.
- If `active: true, status: "picked"`: what the weekly routine picked, and that
  `/build start` will kick it off.
- If `active: false`: nothing active. Mention the most recent history entry if
  one exists.

Then list the three subcommands.

---

## `start [idea name]`

### Step 1 — Guard

Read `current-build.json`.

- If `active: true` AND `status: "building"` — **stop.** Show what's being
  built and its age. The user must `/build done` or `/build abandon` first.
  One build at a time.
- If `active: true` AND `status: "picked"` — a winner was picked but not
  started. If no `<idea name>` argument, or if it matches the picked name,
  proceed with the picked winner. If the argument names something *different*,
  move the picked winner to history as `"outcome": "deferred"` and proceed
  with the new name.
- If `active: false` and no `<idea name>` argument — ask for a name and stop.

### Step 2 — Find the idea

Search `ideas/*.md` for an idea whose name matches the argument (case-insensitive
substring match). Read its full entry — you need the **Idea**, **Leverages**,
**One new thing to learn**, and **Why it's worth it** fields.

If multiple ideas match, show the matches and ask which one.

If no match is found and the name doesn't come from an idea file (the user is
building something original), that's fine — proceed with just the name, and
skip the idea-sourced sections of the design doc in Step 5.

### Step 3 — Update current-build.json

```json
{
  "active": true,
  "name": "<idea name>",
  "picked_at": "<today YYYY-MM-DD>",
  "cluster_days": <from deferred history entry if one matches, else 0>,
  "status": "building",
  "repo": "<owner/repo — filled in Step 4>",
  "history": [<preserve existing>]
}
```

If `picked_at` was already set from a weekly pick, keep the original date.
Commit and push `current-build.json` before creating the repo — the build
tracking works even if repo creation fails.

### Step 4 — Create the repo

Derive a repo name: lowercase the idea name, replace spaces with hyphens, strip
anything that isn't alphanumeric or a hyphen. Show the proposed name and
**confirm with the user** before creating.

Create a public GitHub repo under the user's account (look up the owner from
this repo's remote, or use the authenticated user). Clone it locally.

If repo creation fails (no GitHub access, name taken, etc.), say so plainly and
continue — the user can create the repo manually. Update the `repo` field in
`current-build.json` once the repo exists.

### Step 5 — Seed the design doc

Write `docs/DESIGN.md` in the new repo:

```markdown
# <Idea Name> — Design

> Seeded from passive-ideas-engine on YYYY-MM-DD.

## What this is

<the idea's one-sentence description from the idea file>

## What it leverages

<from the Leverages field>

## The one new thing to learn

<from the One new thing to learn field>

## Why it's worth building

<from the Why it's worth it field>

## Architecture

<!-- What are the major pieces? How do they connect?
     What's the data flow from input to output?
     Fill this in before writing code. -->

## MVP scope

<!-- What's the smallest version that proves the idea works?
     3-5 concrete deliverables — what can you demo at the end? -->

## What "done" looks like

<!-- How will you know this is finished?
     What test or demo would prove it works?
     What would a user actually do with it? -->
```

If the idea didn't come from a file (no match in Step 2), write the same
template but leave the first four sections as `<!-- fill in -->` prompts.

Commit the design doc and push to the new repo's `main` branch.

### Step 6 — Report

Tell the user:
- `current-build.json` updated
- Repo created (link it)
- Design doc seeded
- One line: "Fill in Architecture, MVP scope, and What done looks like before
  you start coding — that's where you'll learn the most."

---

## `done`

### Step 1 — Guard

Read `current-build.json`. If `active: false`, say there's nothing to close
and stop. If `status: "picked"` (never started building), suggest `/build
abandon` instead — you can't ship what you didn't start.

### Step 2 — Move to history

Append to the `history` array:

```json
{
  "name": "<name>",
  "picked_at": "<picked_at>",
  "cluster_days": "<cluster_days>",
  "repo": "<repo>",
  "outcome": "shipped",
  "completed_at": "<today YYYY-MM-DD>"
}
```

Reset the active fields:

```json
{
  "active": false,
  "name": "",
  "picked_at": "",
  "cluster_days": 0,
  "status": "none",
  "repo": "",
  "history": [<existing entries>]
}
```

Commit and push.

### Step 3 — Remind

Tell the user: "The next weekly refresh will scan the repo and update
profile.md with what you shipped."

---

## `abandon`

### Step 1 — Guard

Same as `done` Step 1, but allow abandoning a `"picked"` build too (you're
declining the winner).

### Step 2 — Ask why

Ask for a one-line reason. Wait for the answer.

### Step 3 — Move to history

Same as `done` Step 2, but:

```json
{
  "outcome": "abandoned",
  "completed_at": "<today YYYY-MM-DD>",
  "reason": "<one-line reason>"
}
```

Commit and push.

---

## What you must never do

- Never have two active builds at once.
- Never silently overwrite an active build — always require `done` or `abandon`.
- Never modify any file in this repo other than `current-build.json`.
- Never create a repo without confirming the name with the user.
- Never skip the design doc — the empty sections are where thinking happens.
