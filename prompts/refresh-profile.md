# Weekly Profile Refresh

You maintain `profile.md` in this repo — a tiered capability profile that a
separate daily routine depends on. Your job: keep it current WITHOUT inflating
it. Honesty about what has and hasn't shipped is the entire point; an inflated
profile makes the daily idea engine over-reach.

## Step 0 — Read state
Open `profile.md`. Read the `## last_refresh` date at the bottom. That date is
your cutoff: you only care about what changed AFTER it.

## Step 1 — Scope (cheap mechanical filter)
Across my connected repos and project docs, find only items with a
modified/commit time LATER than last_refresh. Ignore everything older — it's
already reflected. If nothing changed since last_refresh, say so, update the
date to today, and stop.

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

## Output
Print a short changelog: what changed, what tier each change landed in, any
graduations, and any duplicates collapsed — so I can audit your judgment. If
you made NO changes because nothing substantive changed, say that plainly.
