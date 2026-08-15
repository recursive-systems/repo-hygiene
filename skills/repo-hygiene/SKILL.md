---
name: repo-hygiene
description: "Use when auditing, cleaning, tidying, or consolidating a repository that AI agents operate in — pruning CLAUDE.md or AGENTS.md, finding stale instructions or dead docs, deciding whether guidance belongs in an always-loaded file or a skill, consolidating overlapping skills, archiving finished plans and TODO files, removing scratch files or committed build artifacts, or reviewing a repo's instruction layer after heavy agent use. Trigger on 'clean up this repo', 'audit the docs', 'is my CLAUDE.md too big', 'consolidate these skills', 'what's stale in here', or any request to reduce context bloat. Audit-first: produces a ranked findings report with evidence; deletes nothing without approval. Not for code refactoring or dependency upgrades."
license: MIT
metadata:
  author: recursive-systems
  version: "1.0"
---

# Repo Hygiene

A human reads a stale README skeptically. An agent executes it with full confidence: wrong
build command, dead convention, missing guardrail, followed to the letter. In a repo that
agents operate, the instruction layer is load-bearing infrastructure, and it rots by default.
Roughly a quarter of repos with agent config files already reference paths, commands, or
libraries that no longer exist (arXiv 2606.09090).

The failure is not messiness. It is that every session pays for the mess. Always-loaded files
spend a finite attention budget (frontier models follow ~150–200 instructions with
reasonable consistency, and the harness spends ~50 before your repo says a word), and a wrong
line in CLAUDE.md poisons every conversation that starts in the repo.

This skill audits a repo and produces a ranked findings report. The human decides what to
keep. Nothing is deleted, moved, or rewritten without approval.

## Severity model

Rank every finding by the context-defect hierarchy (worst first):

1. **Incorrect** — instructions or docs that are wrong at HEAD: stale paths, dead commands,
   facts contradicted by the code or data layer. An agent will follow these confidently.
   A stale cache is worse than an empty one.
2. **Missing** — recurring mistakes with no captured fix, guidance that exists only in one
   person's head or in old chat logs.
3. **Noise** — correct but costly: duplicated guidance, self-evident rules, finished plans
   left where they read as current direction, oversized always-loaded files.

Fix incorrect before missing before noise. A report that leads with formatting nits while a
dead build command sits in CLAUDE.md has the priorities backwards.

## Order of operations

1. **Map the context surface** (§1) — what loads always, on trigger, on demand
2. **Mechanical checks** (§2) — cheap, deterministic, run every time
3. **Judgment checks** (§3) — the per-line test, duplication, skill boundaries
4. **Report** (§4) — ranked findings with evidence; the human decides
5. **Apply** (§5) — only approved items, then re-run §2

## 1. Map the context surface

Cost is a function of when a file loads, so inventory by tier before judging anything:

| Tier | What | Cost |
|---|---|---|
| Always loaded | CLAUDE.md, AGENTS.md, .cursorrules, copilot-instructions.md, skill frontmatter descriptions | Every session pays. Highest scrutiny. |
| Trigger loaded | Skill bodies (SKILL.md) | Paid when the description matches. |
| On demand | Skill references/, docs the agent is pointed at | Paid only when followed. Cheapest place for detail. |
| Never loaded | Everything else in the tree | Costs only when it collides with search, grep, or git history. |

The leverage hierarchy runs the same direction: a bad line in an always-loaded file is the
most expensive line in the repo. Audit tier by tier, top down. Moving content down a tier
(demotion into a skill or reference) is often the fix; deleting is not the only tool.

Also record what the repo treats as canonical: data files, lockfiles, config that generators
or linters read. Prose gets judged against these in §3.

## 2. Mechanical checks

Deterministic, no judgment required, so run all of them every audit:

1. **Stale references.** Every file path, script, command, function, and package named in an
   always-loaded file or skill must exist and run at HEAD. Extract them and check each one.
   This is the single most common defect and always severity: incorrect.
2. **Size against budget.** Line counts for every always-loaded file. Under ~60 lines is
   healthy for a root CLAUDE.md; ~300 is a hard ceiling; skill bodies belong under ~500
   lines. Over budget is not itself the finding — it directs where §3 digs.
3. **Committed artifacts.** Build outputs, renders, large binaries, generated files in git.
   Every agent grep and git log pays for them. Check history size, not just the worktree.
4. **Finished work left live.** TODO.md, PLAN.md, SPEC.md, scratch notes, research docs:
   anything whose checkboxes are all checked or whose subject shipped. Done plans are
   archives, not instructions.
5. **Scratch in the tree.** Debug logs, one-off scripts, `test2.py`, `.bak` files, agent
   session leftovers. Check `git status` for long-lived untracked files too.
6. **Foreign objects in config directories.** Non-skill directories inside the skills folder,
   workspaces inside `.claude/`, anything an agent will mistake for loadable instructions.
7. **Duplicate strings.** The same command, rule, or fact stated verbatim in more than one
   file. Verbatim duplication is mechanical to find; paraphrased duplication is §3's job.

**Load [`references/mechanical-checks.md`](references/mechanical-checks.md)** when actually
running this pass — it holds the extraction patterns, the commands, and the false-positive
traps for each check.

## 3. Judgment checks

These need reading, not grepping. Work top tier down.

**On every always-loaded line:** *would removing this cause the agent to make mistakes?* If
not, cut it. Bloated always-loaded files cause agents to ignore the instructions that matter.
Specific flags:

- **Not universal.** Applies only to some tasks → demote to a skill or an on-demand doc.
  Always-loaded files are for what every session needs.
- **A linter's job.** Style rules a formatter or linter already enforces → point at the
  config, don't restate it. Never send an LLM to do a linter's job.
- **Should be deterministic.** "Always do X before Y" instructions that a hook, test, or CI
  gate could enforce → convert. Advisory prose is for what can't be mechanized.
- **Inlined snippets.** Pasted code goes stale silently; `file:line` pointers don't.
- **Self-evident.** "Write clean code," standard conventions, anything the agent does
  correctly without being told.

**Across files:** every fact and rule needs exactly one canonical home; everything else
points at it. Flag paraphrased duplication (it drifts), prose contradicting the data layer
or the code (severity: incorrect, the data layer wins), and docs describing abandoned
architecture.

**On the skills layer:**

- **Should exist:** recurring corrections in feedback logs or session history with no
  captured fix. Each repeated mistake is a missing skill or missing skill line.
- **Should merge or die:** if a human can't say definitively which of two skills applies to
  a task, an agent can't either. Also flag skills added speculatively that nothing ever
  triggered.
- **Should split:** an unwieldy SKILL.md whose sections are rarely used together belongs in
  references loaded on demand.
- **Weak frontmatter:** descriptions are always-loaded; each must say when to trigger in the
  user's vocabulary, and every reference pointer must say *when* to load the file.

**Load [`references/judgment-checks.md`](references/judgment-checks.md)** when running this
pass — it expands each test with worked examples and decision tables, including the
demote/promote ladder between tiers.

## 4. Report

One artifact, not a pile of notes. For each finding:

- **Severity** (incorrect / missing / noise) and tier of the file involved
- **Evidence** — quote the line, cite `file:line`, and for staleness show what makes it
  stale (the command that fails, the path that 404s, the data-layer value that disagrees)
- **Recommendation** — one of: *fix, delete, demote* (move down a tier), *promote* (capture
  a missing fix as an instruction or skill), *merge, split, archive, convert* (to
  hook/lint/CI), or *keep*
- **Recoverability** — note when a deletion is recoverable from git history and when it
  isn't

Order findings by severity, then by tier. "Keep" is a real outcome; an audit that flags
everything is as useless as one that flags nothing. If evidence is ambiguous (a file that
might be a live plan, a rule whose reason you can't see), put it in an open-questions section
instead of guessing a verdict.

**Load [`references/report-format.md`](references/report-format.md)** for the report
template and a worked example.

## 5. Apply

Only after the human approves specific findings:

- Batch related changes; keep the co-change rule: a commit that changes build, test, or
  layout also updates the config files describing them, in the same commit.
- Prefer demotion over deletion when content is correct but misplaced.
- Archive rather than delete plans and research with historical value (a dated `archive/`
  or the repo's own convention); delete true scratch.
- Re-run §2 afterwards. Hygiene changes create their own stale references.

## Gotchas

- **Data is not cruft.** Canonical data files, append-only logs, and manifests can be large
  and messy-looking; they are the repo's memory. Never flag the data layer for deletion
  because of size.
- **Big is not dirty.** The finding is never "this repo has many files." It is a specific
  line that is wrong, duplicated, misplaced, or dead.
- **Verify staleness before flagging it.** A doc nothing links to may be loaded by a script;
  a "finished" plan may be paused. Check inbound references and recent git activity before
  calling something dead. When still unsure, it goes in open questions, not findings.
- **Don't flag the maintainer's taste.** Voice, formatting preferences, and idiosyncratic
  structure are theirs. Hygiene findings are about correctness, cost, and placement.
- **The audit must not add cruft.** Output is the conversation or one report file the user
  asked for. No committed scratch, no leftover analysis files.
- **Deletion needs approval, every time.** Even in an "apply" run, anything not explicitly
  approved stays. When running unattended, stop at the report.

## Why these rules

The principles behind the checks (attention budget, context rot, progressive disclosure,
compaction, the severity hierarchy) with sources and URLs:
**load [`references/principles.md`](references/principles.md)** when the user asks *why* a
finding matters, when writing a report summary that should teach, or when a
recommendation is challenged.
