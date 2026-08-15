# Report format

One artifact. Default to writing it in the conversation; write a file only if the user asked
for one, and put it where they say (never commit it uninvited — an audit that leaves its own
scratch behind has failed its own test).

## Structure

```markdown
# Hygiene audit: <repo> @ <short-sha> (<date>)

## Summary
2–4 sentences: overall state, the one or two findings that matter most, and what was
checked (so "no finding" is meaningful, not just "didn't look").

## Findings
Ordered by severity (incorrect → missing → noise), then by tier (always-loaded first).

### F1. <one-line claim>  [incorrect | always-loaded]
- Evidence: <file:line>, the quoted line, and what proves the problem
  (the command output that fails, the data-layer value that disagrees)
- Recommendation: fix | delete | demote | promote | merge | split | archive | convert | keep
- Proposed change: the concrete edit, one or two lines
- Recoverable: yes via git history / no (untracked)

## Open questions
Items where evidence was ambiguous. One line each: what was found, what would resolve it,
who can answer. These are questions for the human, not soft findings.

## Checked and clean
The checks that ran and found nothing, one line each. This is what makes the audit
re-runnable and diffable against the next one.
```

## Rules

- **Every finding quotes its evidence.** A finding the user can't verify from the report
  alone gets cut or moved to open questions. No "consider reviewing X."
- **One recommendation per finding.** If two dispositions are defensible, pick one and note
  the alternative in a clause, not a second recommendation.
- **"Keep" is a real outcome.** Use it when something looks like cruft but earned its place
  (an append-only log, a big data file, an odd-but-load-bearing rule). Flagging everything
  is as useless as flagging nothing, and "keep" findings teach the next auditor.
- **Count the wins.** If the repo does something notably right (lean CLAUDE.md, clean skill
  boundaries), one line in the summary. It calibrates the reader on what the findings mean.
- **Never guess a verdict.** Ambiguity goes to open questions with the question stated
  crisply enough that a one-word answer resolves it.

## Worked example (abbreviated)

```markdown
### F1. CLAUDE.md documents a test command that no longer exists  [incorrect | always-loaded]
- Evidence: CLAUDE.md:12 says `npm run test:all`; package.json:8-14 defines only
  `test` and `test:watch`. `npm run test:all` exits 1 ("missing script").
- Recommendation: fix — change to `npm test`.
- Recoverable: n/a (edit)

### F4. Deployment checklist lives in CLAUDE.md but applies only to releases  [noise | always-loaded]
- Evidence: CLAUDE.md:31-58 (27 lines, ~15 instructions) describe the release process.
  Releases happen ~monthly (git log --grep=release); every session pays for the lines.
- Recommendation: demote — move to a `release` skill; leave a one-line pointer.
- Recoverable: yes (tracked)

### F7. `docs/plan-search-v2.md` reads as a live plan but shipped in March  [noise | never-loaded]
- Evidence: all 14 checkboxes unchecked, but the files it plans exist (src/search/, added
  in commits 3f1c2a9..8809d1e, March). Last edit to the plan: Feb 28.
- Recommendation: archive to docs/archive/2026-02-search-v2-plan.md; it documents intent
  the commits don't.
- Recoverable: yes (tracked)

## Open questions
- clients/sweep-01-block-ranking.md: plan or record? Untracked-by-any-index, last edited
  Aug 2, no inbound links. If the sweep is finished, this archives; if scheduled, it stays.
```

## Apply runs

When the user approves findings and asks for them to be applied:

- Apply only the approved finding IDs. Restate the list before starting.
- Group commits by kind (fixes, demotions, archives) so each is revertable.
- Same-commit co-change: an edit that moves or renames something updates every pointer to
  it, verified by re-running the stale-reference check before committing.
- End by re-running the mechanical pass and reporting the diff in findings.
