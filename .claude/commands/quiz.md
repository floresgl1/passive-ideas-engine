---
description: Quiz yourself on an unlabeled idea and write the earned competence verdict back to its file.
---

# Idea Quiz — turn `unlabeled` into an earned verdict

This is the ONLY routine permitted to write `competence` or `labeled_at`. The
daily engine is forbidden from guessing them; the weekly routine only reads
them. A verdict exists because it was measured here, or it does not exist.

That is the whole reason this command is careful about what it may touch.

> **BRANCH RULE:** work on `main`. `git checkout main`, `git pull` before
> reading, `git push` after each verdict. The daily and weekly routines both
> read and write `main`; a label committed anywhere else never reaches them.

> **WRITE SCOPE (critical):** the only thing you may change in `ideas/` is the
> `- competence:` and `- labeled_at:` lines of the single idea being quizzed
> right now. Never rewrite an idea's title, category, or body. Never reformat a
> file. Never touch a second idea in the same edit. These files are the
> measurement record.

> **LEGACY FILES:** some older ideas have no `competence` / `labeled_at` lines
> at all. They are tracked debt with a pending decision. They are NOT in the
> queue, and you must NOT backfill, add fields to, or quiz them. Skip silently.

---

## Step 1 — Find the queue

```
grep -l "competence: unlabeled" ideas/*.md
```

Ideas whose `competence` is anything other than `unlabeled` are already
measured and are out of scope for this command — re-quizzing labeled ideas is
the decay routine's job, not this one.

Take the **oldest** matching file (filenames sort chronologically), read it, and
pick the **first** idea in it still marked `unlabeled`. Oldest-first, because the
ideas closest to dying unexamined are the ones this exists to catch.

If the queue is empty, say so in one line and stop. Do not invent work.

## Step 2 — Pick one idea, and say almost nothing about it

Announce only the idea's **name** and **[category]**, plus how many remain in the
queue. Example:

> Quizzing: **Port-Parity Test Harness** [Leverage] — 6 more in the queue.

**Do not paste, summarize, or paraphrase the idea's body into the conversation
before the verdict is written.** You have the full text open; the person being
quizzed is working from memory. Leaking it costs you the measurement.

## Step 3 — The free-response probe

Ask ONE open question, then wait for a real answer.

### Generating the probe

**Before writing the question, identify the idea's HARD PART** — the thing that
would sink the build in week two. That is check 2's target. Aim the probe there,
not at the idea's overall description.

**RULE: the question must NOT be answerable by paraphrasing the idea's own
text.** Self-test before asking: could someone who had read only the idea's
description answer this by restating it? If yes, the probe LEAKED — regenerate.
Ask *how the hard part works*, never *what this is*.

Aim at the hard part without naming it. A question that announces where the
difficulty lies has answered check 2 on their behalf.

- Do not name the core mechanism, the hard part, or any specific technique from
  the file. If your question contains a term the answer should have supplied,
  rewrite the question.
- Do not offer hints, examples, or a starting point. Silence is data.
- One question. Do not stack sub-questions that decompose the problem for them —
  the decomposition is part of what's being measured.

### Worked example — "Agent Guardrails Kit" [Leverage]

The idea as written in the file: generalize finance_bot's bounded-agent
architecture — step caps, retry budgets, enumerated error categories,
constrained output vocabulary, a written v1 contract — into a framework-agnostic
Python package other builders drop into their own agent loops.

**BAD** (leaks — this is a comprehension question):

> "What would they be getting when they buy this kit?"

Why it leaks: the idea's own first sentence IS the answer. "Step caps, retry
budgets, enumerated error categories" can be read straight off the page. This
measures whether someone can restate a digest — the one thing textual fluency
guarantees and knowledge does not.

**GOOD** (mechanism — forces reconstruction of the hard part):

> "Someone installs this into their own agent loop tomorrow. Walk me through
> what actually happens at runtime the first time their agent blows its budget."

Why it can't be answered from the digest: the file never describes runtime
behaviour anywhere. Answering requires reconstructing where the counter lives,
how a package that doesn't own the loop gets control at all, and what it does
when the cap trips — raise, halt flag, return partial. And it surfaces the hard
part without naming it: anyone who actually knows this idea will volunteer "the
tricky bit is you don't own their loop," which is precisely what check 2 is
listening for.

Both questions are about the SAME idea, and differ only in comprehension versus
mechanism. That contrast is the entire lesson. A good/bad pair drawn from two
different ideas teaches nothing about the distinction.

## Step 4 — Grade it

The bar, verbatim, and it does not bend:

> `known` = unprompted, you produced the core mechanism and named what's hard
> about it, specifically enough that someone with general engineering skill and
> no knowledge of this domain could begin the first step from your answer alone.

The stranger in that sentence brings general engineering competence and **zero
domain knowledge**. They supply the ability to type; the answer must supply
everything else. "I'd use nodal analysis" fails, because it is only a passing
grade if you imagine a stranger who already knows the domain.

`known` requires ALL THREE:

1. **Mechanism, not category.** The answer contains something that could not
   have been produced by a person who read only the idea's title. "Build the
   conductance matrix from node connections, solve Gx=b" passes. "Nodal
   analysis", "static analysis", "an API for it" are addresses, not mechanisms.
2. **The hard part is named.** Every idea has one thing that would sink the
   build in week two. Knowing an idea means knowing where that is. A person who
   read the digest knows the idea is *good*; a person who knows the idea knows
   what's *hard* about it.
3. **No hand-wave at the load-bearing step.** If the answer routes through "and
   then you just…" exactly where the difficulty lives, it fails — no matter how
   good the surrounding prose is.

Two rules that keep you honest:

- **Grade against the mechanism, not against the file's wording.** An answer
  phrased nothing like the digest is still correct — reconstructing a mechanism
  live is stronger evidence than recalling it, not weaker. Conversely, a fluent
  restatement of the idea's pitch ("a tool that helps teams find divergence")
  is a near word-match and passes nothing. Textual similarity to the file is
  ANTI-correlated with what you are measuring.
- **When torn between `known` and the MC probe, take the probe.** You will feel
  pressure to award partial credit for a plausible-sounding answer. Do not.
  `known` eventually becomes an outward-facing claim about this person's
  skills; generosity here is how an unearned claim escapes.

**Passes all three → `known`.** Skip to Step 6.

**Anything less — blank, vague, partial, adjacent — → Step 5.** The trigger for
the MC probe is *failing the bar*, not saying nothing. A partial answer is not a
blank and must not be rounded up.

## Step 5 — The MC probe (only if the bar was not met)

Same idea. Ask which option describes how it actually works.

- Five options: one correct mechanism, three wrong ones, and a final
  **"none of these / I don't recognize it."**
- Wrong options must be **plausible within the domain and wrong in mechanism** —
  not silly, not off-topic. If a distractor is obviously absurd, recognition
  becomes free and the probe measures nothing.
- All five must match in length and specificity. A longer, more detailed option
  is a tell, and you will write one by accident unless you check.

Then:

- **Recognizes the correct mechanism → `needs work`.** Couldn't produce it,
  can recognize it. That is exactly what `needs work` means.
- **Wrong option, or "none of these" → `no knowledge`.**

"None of these" is always wrong — the correct answer is present. It exists so
an honest "I don't recognize this" doesn't have to be disguised as a guess.
Treat choosing it as information, not failure.

## Step 6 — Write the verdict back

Compute today's date **in UTC**, not local time:

```
date -u +%F
```
```powershell
[datetime]::UtcNow.ToString('yyyy-MM-dd')
```

Idea filenames are UTC and the weekly routine's "labeled this week" window is
computed in UTC in the cloud. A local date here produces `labeled_at` values
that precede their own file's date and evening labels that silently fall out of
the weekly window. Unintuitive, consistent, non-negotiable.

Edit the two lines to:

```
- competence: known | needs work | no knowledge
- labeled_at: YYYY-MM-DD
```

**Anchor the edit on the idea's exact `###` heading and change only the two
lines directly beneath it.** Every idea in the file has a byte-identical
`- competence: unlabeled` line — a global or first-match replace will silently
label the wrong idea. Include the heading in the match. Never `replace_all`.

Then commit and push immediately — one verdict, one commit:

```
git add ideas/<file>.md
git commit -m "Quiz: <idea name> → <verdict>"
git push origin main
```

Per-verdict commits, not per-session. An interactive quiz gets abandoned
mid-session — that's the normal case, not the failure case — and nothing
measured should be lost to it, or left sitting uncommitted where the next daily
run's pull can clobber it.

If the tree is dirty or the push is rejected, **stop and say so**. Do not force,
rebase around it, or stash someone else's work.

## Step 7 — Now teach it

The verdict is recorded; the answer is no longer worth protecting. Show the
mechanism the idea actually rested on and what its hard part was, in two or
three sentences. For `needs work` and `no knowledge` this is the entire payoff —
the point is to stop ideas dying unexamined, and an idea you just failed and
then learned is no longer dying.

Then offer, in one short line: another one, or stop.

## Session length

One idea per invocation by default. If the invocation carries a number, do that
many in sequence, re-running Steps 2–7 each time.

Keep the ritual small. A quiz that opens with "you have 7 pending, let's do all
7" is a quiz that stops getting opened. The bottleneck in this system is
willingness to show up, not ideas graded per sitting.

## What you must never do

- Never write a verdict that wasn't measured in this session.
- Never label an idea the person didn't answer.
- Never adjust a label after the fact because the answer "was probably fine."
- Never edit an idea's text, title, category, or any file outside the one being
  quizzed.
- Never backfill fields into legacy fieldless ideas.
- Never post to Discord from this command. `QUIZ_WEBHOOK_URL` belongs to the
  weekly routine's queue-health report; this session is face to face.
