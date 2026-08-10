<!-- SOURCE OF TRUTH: this file is the DRAFTING SURFACE for the "Quiz - Probe"
     trigger. The trigger payload is what actually RUNS. Edit here first, then
     paste across, and bump the prompt-version marker at the foot in the same
     change. -->

# Quiz Probe — post one question to the quiz channel and stop

This is PHASE 1 of the asynchronous quiz. It asks; it never grades. Grading is
`quiz-collect.md`'s job, and the split is not cosmetic — the person being
quizzed answers hours later, in a different session, from a phone.

`/quiz` (`.claude/commands/quiz.md`) remains the face-to-face version and is
still the reference for probe quality and the grading bar. This file changes
the DELIVERY, not the measurement. Where the two disagree about how to write a
probe or where the bar sits, `/quiz` wins and this file is the bug.

> **BRANCH RULE:** work on `main`. `git checkout main`, `git pull` before
> reading, `git push` after writing state. The daily and weekly routines both
> read and write `main`; state committed anywhere else is invisible to Phase 2.

> **WRITE SCOPE (critical):** the only files you may write are
> `quiz-state.json` and `quiz-queue.json` (the latter only to mark expired
> entries, per Step 1). You may NOT write to `ideas/` at all — not a verdict,
> not a field, not a reformat. Phase 1 has measured nothing yet, so it has
> nothing to record.

> **NEVER TOUCH IDEA FILES:** some ideas have no `competence` / `labeled_at`
> lines at all, and many that do will stay `unlabeled` forever because they
> never recurred. Neither is a defect to repair. Do not backfill fields, do not
> add fields, do not quiz anything that is not a `quiz-queue.json` entry.

---

## Step 0 — If a probe is already outstanding, stop

Read `quiz-state.json`. If `stage` is anything other than `"idle"`:

- If `expires_at` is still in the future — **stop, post nothing.** A question is
  already waiting for an answer. Two open questions in one channel is how a
  reply gets matched to the wrong idea.
- If `expires_at` has passed — the probe was abandoned. Set `stage` to `"idle"`,
  leave the idea `unlabeled`, commit, and continue to Step 1 with a fresh idea.
  An unanswered probe is not a failed probe: never write `no knowledge` for
  silence. Silence means the person did not show up, which measures nothing
  about what they know.

If `quiz-state.json` does not exist, treat it as `{"stage": "idle"}`.

## Step 1 — Find the queue

The queue is `quiz-queue.json`, **not** a grep over `ideas/`. The weekly routine
stocks it with one entry per concept that recurred on ≥2 distinct days. Ideas
that never recurred are not queue entries and must not be quizzed — they stay
`unlabeled` permanently, which is an accurate record of never having been
measured, not a backlog.

Take the entry with `status: "pending"` and the **oldest** `queued_at`.
Oldest-first, because the ideas closest to dying unexamined are the ones this
exists to catch — and under the 21-day window they now genuinely do die.

Before selecting, sweep: any `"pending"` entry whose `expires_at` has passed
becomes `"expired"`. Do not quiz it, do not touch the idea's file, and never
write a verdict for an entry that aged out — expiry measures attendance, not
knowledge.

Read the idea from `idea_file` at `idea_heading`. If the heading is not found
byte-for-byte, **stop and say so** rather than guessing a nearby one: every idea
in a file has a byte-identical `- competence: unlabeled` line, so a near-miss
anchor silently targets the wrong idea.

If no `"pending"` entry remains, post nothing and stop. Do not invent work, and
do not fall back to grepping `ideas/` for something to ask.

## Step 2 — Write the probe

Generate the free-response probe exactly as `/quiz` Step 3 specifies. That
section is the authority; do not paraphrase it from memory. In particular:

- Identify the idea's HARD PART first, then aim at it **without naming it**.
- The question must NOT be answerable by paraphrasing the idea's own text. If
  someone who had read only the description could answer it, the probe LEAKED —
  regenerate.
- One question. No hints, no examples, no sub-questions.

The asynchronous channel makes leaking worse, not better: a leaked probe sits
in the channel indefinitely, and the answer it invites looks like knowledge
forever after.

## Step 3 — Post it, and capture the message ID

Post the idea's **name and [category] only**, then the probe. Never the body,
the summary, or any sentence from the file — the person is answering from
memory and you have the text open.

**Do NOT include the queue depth.** One idea, and silence about how many others
are waiting. A quiz that opens with "9 pending" is a quiz that stops getting
opened — the count adds nothing to the question and turns a two-minute answer
into a chore with a visible backlog attached.

Post with `?wait=true` so Discord returns the created message and you can record
its ID — Phase 2 uses it as the anchor to read replies after:

    python3 -c 'import json,sys; print(json.dumps({"content": sys.stdin.read()}))' \
      < probe.txt > probe.json
    curl -sS -X POST "$QUIZ_WEBHOOK_URL?wait=true" \
      -H 'Content-Type: application/json' --data-binary @probe.json

With `?wait=true` a successful post returns **HTTP 200 and a JSON message
object** — not the 204 the other routines check for. Parse `.id` from that body.
If you get 204, you omitted `?wait=true` and have no anchor: the probe is now
live in the channel with no way to match a reply to it. Say so plainly rather
than guessing an ID.

## Step 4 — Record state and commit

Write `quiz-state.json`:

```json
{
  "stage": "awaiting_free_response",
  "idea_file": "ideas/2026-08-04.md",
  "idea_heading": "### 1. Agent Guardrails Kit — [Leverage]",
  "probe_text": "<the exact question you asked>",
  "probe_message_id": "<id from the wait=true response>",
  "posted_at": "<UTC now, date -u +%FT%TZ>",
  "expires_at": "<posted_at + 3 days>"
}
```

`idea_heading` must be the idea's exact `###` line, byte for byte. Phase 2
anchors its edit on it, and every idea in a file has a byte-identical
`- competence: unlabeled` line beneath it — an anchor that is merely close will
silently label the wrong idea.

All timestamps UTC. Idea filenames are UTC and the weekly routine's
"labeled this week" window is computed in UTC in the cloud; a local timestamp
here produces labels that fall out of their own window.

```
git add quiz-state.json
git commit -m "Quiz probe posted: <idea name>"
git push origin main
```

Commit before you finish, always. A probe posted to Discord with no committed
state is a question nobody can grade.

If the tree is dirty or the push is rejected, **stop and say so.** Do not force,
rebase around it, or stash someone else's work — the daily routine pushes to
`main` at 14:00 UTC and may simply have got there first.

## What you must never do

- Never post two probes without an answer between them.
- Never post the idea's body, or any sentence from the file, alongside the probe.
- Never write to `ideas/` from this phase.
- Never label an idea because a probe expired unanswered.
- Never report a probe as posted without an HTTP 200 and a parsed message ID.

<!-- prompt-version: 2026-08-10.2 -->
