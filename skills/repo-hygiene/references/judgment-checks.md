# Judgment checks

These require reading files and understanding the repo, not grepping. Work top tier down:
always-loaded files first, then skill descriptions, then skill bodies, then docs. A bad line
in an always-loaded file is the most expensive line in the repo.

## The per-line test (always-loaded files)

For every line: **would removing this cause the agent to make mistakes?** If you can't
articulate the mistake, the line is a cut candidate. Over-specified files backfire: the
important rules get lost in the noise and the agent starts ignoring the file wholesale.

Common cut categories, each with its disposition:

| Pattern | Example | Disposition |
|---|---|---|
| Not universal | Release-day checklist in CLAUDE.md | **Demote** to a skill or on-demand doc |
| A linter's job | "Use 2-space indent, no semicolons" | **Convert**: point at the linter config |
| Should be deterministic | "Always run tests before committing" | **Convert** to a hook / CI gate |
| Inlined snippet | 30 pasted lines showing "our error pattern" | **Fix**: replace with `file:line` pointer |
| Self-evident | "Write clean code", "be careful with auth" | **Delete** |
| Fact the code states | File-by-file directory tour | **Delete**; agents can read the tree |
| Changes frequently | Current sprint priorities | **Demote** to a dated doc; prose rots |
| No stated reason | "Never use library X" (why?) | **Fix**: add the reason or open question |

What earns its place in an always-loaded file: commands the agent can't guess, non-default
conventions with reasons, environment quirks, repo etiquette, pointers to canonical sources,
and hard rules whose violation is expensive. Identity and stack stay unconditional;
situational guidance either moves to a skill or gets an explicit condition ("when writing
tests: ...").

## The demote/promote ladder

Placement, not existence, is usually the question. One move per finding:

- **Demote** when content is correct but not universally needed: always-loaded → skill body
  → reference file → plain doc the agent is pointed at. Detail is cheapest at the bottom.
- **Promote** when the same correction keeps being made in sessions, feedback logs, or code
  review with no captured fix: chat log → skill line → (only if truly universal)
  always-loaded line. Each repeated mistake is a missing instruction; capture it once, where
  it triggers.

Promotion is the compounding loop (each task's learnings make the next task easier).
Demotion is what keeps the loop from silting up the top tier.

## Duplication and contradiction (across files)

Every fact and rule gets exactly one canonical home. For each duplicate found mechanically
or by reading:

1. Decide the canonical home. Data beats prose: if a number lives in a data file, config,
   or lockfile, that is the home, always. For rules, the home is the most specific tier that
   all users of the rule load.
2. Everything else becomes a pointer or gets deleted.
3. If copies disagree, that is a severity: **incorrect** finding, and resolving which copy
   is right comes before any cleanup. Check git log to see which copy was maintained.

Also hunt paraphrased duplication (same rule, different words, drifting independently) and
**dead docs**: prose describing an architecture, workflow, or convention the code abandoned.
An agent will confidently rebuild the old world from a dead doc. Verify against the code
before flagging, then recommend delete or rewrite, never "leave with a warning banner."

## Skills layer

**Should a skill exist?** Look in feedback logs, session retros, and commit messages for the
same correction appearing more than twice. The Hashimoto rule: every time an agent makes a
mistake, engineer it so that mistake can't happen again. If the fix is prose, it belongs in
a skill; if it's mechanizable, a hook or test beats prose.

**Should two skills merge?** The tool-clarity test: describe a task to a colleague and ask
which skill applies. If a human can't answer definitively, an agent can't either. Overlap in
frontmatter descriptions is the early signal, since descriptions are what routing sees.

**Should a skill die?** Signals: nothing has triggered it (no references to it in logs or
session history), it was written speculatively before any observed failure, or the workflow
it encodes no longer exists. Auto-generated agent files that were never hand-pruned
measurably hurt performance; treat "generated and never edited since" as a flag.

**Should a skill split?** When SKILL.md exceeds ~200 lines, check whether its sections are
used together. Mutually exclusive or rarely-co-used content moves to `references/` with a
pointer that says *when* to load it. A pointer without a "when" defeats progressive
disclosure: the agent either always loads it or never does.

**Frontmatter quality.** The description is the always-loaded part of a skill and the only
thing routing sees. It must: name concrete trigger situations in the vocabulary a user would
actually use, state what the skill is *not* for, and fit the spec's 1024-char limit. A
description that just summarizes the topic ("Guidelines for X") won't trigger.

## Instruction phrasing (while you're in there)

When a finding's fix involves rewriting an instruction, prefer forms that survive:

- State the reason with the rule. "Never quote firm without a site visit (estimates from
  photos have burned us)" outlives "never quote firm."
- Prefer positive instructions with conditions over bare prohibitions.
- Prefer pointers to authoritative sources over restated content.
- Absolute over relative: dates, versions, and counts written relatively ("last week",
  "the new API") rot fastest.
