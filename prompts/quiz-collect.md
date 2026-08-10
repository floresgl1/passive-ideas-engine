<!-- SOURCE OF TRUTH: this file is the DRAFTING SURFACE for the "Quiz - Collect"
     trigger. The trigger payload is what actually RUNS. Edit here first, then
     paste across, and bump the prompt-version marker at the foot in the same
     change. -->

# Quiz Collect — read the reply, grade it, write the verdict

This is PHASE 2 of the asynchronous quiz. It runs often, does nothing most of
the time, and grades when an answer has arrived.

`/quiz` (`.claude/commands/quiz.md`) is the authority on the grading bar. This
file changes WHEN grading happens, never HOW. Where the two disagree about the
bar, `/quiz` wins and this file is the bug.

> **BRANCH RULE:** work on `main`. `git checkout main`, `git pull` before
> reading, `git push` after each verdict.

> **WRITE SCOPE (critical):** you may write `quiz-state.json`, and — in
> `ideas/` — only the `- competence:` and `- labeled_at:` lines of the single
> idea named in `quiz-state.json`. Never rewrite a title, category, or body.
> Never reformat a file. Never touch a second idea. These files are the
> measurement record.

> **THE INTEGRITY RULE:** a verdict may only be written when this run has read
> an actual reply message from the quizzed person, matched to the probe recorded
> in `quiz-state.json`. No reply, no verdict — no exceptions, no inference, no
> "they probably knew this." Asynchrony removes the person from the room; the
> state file and the message ID are the only things left proving a measurement
> happened. Treat them as such.

---

## Step 0 — Is anything outstanding?

Read `quiz-state.json`. If `stage` is `"idle"` or the file is missing, **stop
silently.** Post nothing, commit nothing. This trigger fires often and a quiet
run is the normal outcome.

If `expires_at` has passed with no reply: set `stage` to `"idle"`, leave the
idea `unlabeled`, commit, and stop. Do not post an admonishment to the channel.
Silence is not a wrong answer, and nagging is how the ritual stops getting
opened.

## Step 1 — Read replies since the probe

    curl -sS -H "Authorization: Bot $DISCORD_BOT_TOKEN" \
      "https://discord.com/api/v10/channels/$QUIZ_CHANNEL_ID/messages?after=$PROBE_MESSAGE_ID&limit=100"

Use `probe_message_id` from state as `after` (or `mc_message_id` when `stage` is
`"awaiting_mc"` — grade the reply to the question actually being asked).

Then filter, in this order, and drop anything that fails:

1. **Author is the quizzed person** — `author.id == QUIZ_USER_ID`. Bot posts,
   webhook posts, and anyone else in the channel are not answers. Your own
   probe will appear in this list; discarding it is the point of the filter.
2. **Posted after the probe** — the `after` parameter handles this, but check
   `timestamp` anyway rather than trusting pagination.

If nothing survives the filter, **stop silently.** Commit nothing. The person
has not answered yet, which is the expected state most of the time.

If several replies survive, treat them as one answer in chronological order —
people send a thought in three messages. Do not grade only the first and do not
grade only the last.

## Step 2 — Grade against the bar

Grade exactly as `/quiz` Step 4 specifies. Read that section; do not reconstruct
it from memory. The bar verbatim:

> `known` = unprompted, you produced the core mechanism and named what's hard
> about it, specifically enough that someone with general engineering skill and
> no knowledge of this domain could begin the first step from your answer alone.

All three required: **mechanism not category**, **the hard part is named**, **no
hand-wave at the load-bearing step.** Grade against the mechanism, not against
the file's wording — textual similarity to the digest is ANTI-correlated with
what you are measuring.

Two async-specific pressures, both of which push toward false `known`:

- A reply typed on a phone is **terser** than one typed in a session. Terseness
  is not vagueness — judge what the words claim, not how many there are. But
  brevity also does not earn benefit of the doubt at the load-bearing step.
- Nobody is watching this run. **When torn between `known` and the MC probe,
  take the probe.** `known` eventually becomes an outward-facing claim on a
  résumé; generosity in an unattended job is exactly how an unearned claim
  escapes.

**Passes all three → `known`.** Go to Step 4.
**Anything less — blank, vague, partial, adjacent → Step 3.**

## Step 3 — The MC probe (only if the bar was not met)

If `stage` is already `"awaiting_mc"`, the person has now answered it: score per
`/quiz` Step 5 — correct mechanism → `needs work`; wrong option or "none of
these" → `no knowledge` — and go to Step 4.

Otherwise build the MC probe per `/quiz` Step 5: five options, one correct
mechanism, three plausible-but-wrong-in-mechanism, and a final "none of these /
I don't recognize it." All five matched in length and specificity — a longer,
more detailed option is a tell, and you will write one by accident unless you
check.

Post it the same way Phase 1 posts (`?wait=true`, HTTP 200, parse `.id`), then
update state and commit:

```json
{ "stage": "awaiting_mc", "mc_message_id": "<new id>", "expires_at": "<+3 days>" }
```

Keep `idea_file`, `idea_heading`, and `probe_text` unchanged. Then stop — the
answer arrives on a later run.

## Step 4 — Write the verdict back

Compute the date in **UTC**: `date -u +%F`.

Anchor on `idea_heading` from state and change only the two lines directly
beneath it:

```
- competence: known | needs work | no knowledge
- labeled_at: YYYY-MM-DD
```

Every idea in the file has a byte-identical `- competence: unlabeled` line. A
global or first-match replace will silently label the wrong idea, and a wrong
label is worse than no label — it is a measurement that never happened, wearing
the costume of one. Include the heading in the match. **Never `replace_all`.**

Then, in one commit:

```
git add ideas/<file>.md quiz-state.json
git commit -m "Quiz: <idea name> → <verdict>"
git push origin main
```

Set `stage` back to `"idle"` in the same commit. Verdict and state must land
together — a verdict committed with a stale state file gets re-graded on the
next run and overwritten by the second grading of the same answer.

If the tree is dirty or the push is rejected, **stop and say so.** Do not force,
rebase around it, or stash someone else's work.

## Step 5 — Teach it

Post to the channel: the verdict, then the mechanism the idea actually rested on
and what its hard part was, in two or three sentences. For `needs work` and
`no knowledge` this is the entire payoff — the point is to stop ideas dying
unexamined, and an idea you just failed and then learned is no longer dying.

This is safe to post because a labeled idea leaves the queue permanently —
`/quiz` Step 1 only ever selects `unlabeled` ideas, so nothing you teach here
can be asked again. **If a decay routine is ever built to re-quiz labeled
ideas, this stops being true**: the channel will by then hold a written answer
key, and that routine must not draw its probes from ideas whose mechanism was
taught here. Note it now, while the reason is still visible.

Do not offer "another one, or stop" — that is the face-to-face command's line.
Phase 1 posts the next probe on its own schedule.

## What you must never do

- Never write a verdict without a matched reply message read in this run.
- Never label an idea because a probe expired unanswered.
- Never grade a message from any author other than `QUIZ_USER_ID`.
- Never re-grade an answer already scored — check `stage` first.
- Never edit an idea's text, title, category, or any idea other than the one in
  `quiz-state.json`.
- Never backfill fields into legacy fieldless ideas.

<!-- prompt-version: 2026-08-10.1 -->
