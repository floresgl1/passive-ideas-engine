# Career Log

Durable record of capability transitions that were written into `profile.md`.

`profile.md` is a snapshot — it is rewritten in place, so it cannot retain the
fact that a transition happened. This file is that history.

## Contract

**Written by:** the weekly routine's PART B, and nothing else. A record is
appended only for a tier transition PART B **actually wrote into `profile.md`
during that run**, in the **same commit as the `profile.md` edit**. That
coupling is the audit: `git show` on any record displays the tier change that
justifies it, so an unearned record is visible rather than merely prohibited.

**Read by:** `/resume-line`, which is the only thing permitted to turn a record
into outward-facing text. This file holds facts; claims are generated on demand
under the bar in `.claude/commands/resume-line.md`.

**WRITE SCOPE:** you may only ADD a new `##` record at the END of this file.
Never edit, reword, reorder, or delete an existing record. Never reformat the
file. Records are the substrate every outward claim rests on; a rewritten record
is an unfalsifiable one.

**No competence field exists, by design.** A quiz verdict in `ideas/` measures
whether an idea's mechanism could be reconstructed from memory. It is not
shipped work, and it must never reach a résumé line. The schema gives it no slot
to land in.

## Record schema

```
## YYYY-MM-DD — <artifact>
- axis: <capability axis, e.g. hardware>
- transition: [tier] → [tier]
- artifact: <repo or artifact name>
- evidence: <repo path, commit count/range, concrete signals — tests, measured
  results, deployments. Numbers here are what license numbers in a claim.>
```

Records are chronological, oldest first, newest appended at the end.

---

*No records yet. The first will be appended by PART B on the next weekly run
that writes a tier transition.*
