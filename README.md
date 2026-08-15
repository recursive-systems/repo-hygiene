# repo-hygiene

Your agent reads CLAUDE.md and believes every word of it.

The build command that changed in March. The convention you abandoned. The plan that
shipped last quarter and still reads like marching orders. A human skims stale docs with
suspicion. An agent executes them, confidently, every session.

In a repo that AI agents operate, the instruction layer is infrastructure, and it rots by
default: a 2026 scan of 356 repositories found stale code references in 23% of their agent
config files.

This [Agent Skill](https://agentskills.io) audits that layer and shows you what is wrong
and what it costs.

## How it works

Findings are ranked by a severity model from context engineering: **incorrect beats missing
beats noise.** One wrong line in an always-loaded file poisons every session in the repo and
outranks any amount of clutter.

The audit maps the repo's context surface first (what loads always, on trigger, on demand),
then runs two passes:

- **Mechanical checks**, deterministic: paths and commands that no longer exist, finished
  plans left readable as current, scratch files, build artifacts bloating git history, the
  same fact stated in three places.
- **Judgment checks**, which need reading: the per-line test on CLAUDE.md (would removing
  this cause the agent to make mistakes?), guidance that belongs in a skill instead of an
  always-loaded file, skills that overlap or that nothing ever triggered.

Every finding quotes its evidence with file and line, and carries one recommendation: fix,
delete, demote, promote, merge, split, archive, convert, or keep. You approve each one. The
skill deletes nothing on its own, and "keep" is a real verdict: big data files, append-only
logs, and odd-but-load-bearing rules earn their place.

## When it loads

Any request to audit, clean up, or consolidate a repo's instruction layer: pruning a
bloated CLAUDE.md, hunting stale docs, deciding what belongs in an always-loaded file
versus a skill, merging overlapping skills, archiving finished plans, clearing scratch and
committed artifacts. Also after heavy agent use, when the tree has accumulated session
leftovers nobody chose to keep.

Not for code refactoring, dependency upgrades, or performance work. This is the instruction
and documentation layer, not the code.

## Install

```bash
# into the current project, for whichever agents you have
npx skills add recursive-systems/repo-hygiene

# global instead of project-local
npx skills add recursive-systems/repo-hygiene -g

# target specific agents
npx skills add recursive-systems/repo-hygiene -a claude-code -a codex

# see what's in here without installing
npx skills add recursive-systems/repo-hygiene --list
```

`npx skills` ([vercel-labs/skills](https://github.com/vercel-labs/skills)) resolves each
agent's own config directory (`.claude/skills/`, `.agents/skills/`, `~/.codex/skills/`) and
symlinks a single canonical copy into each. Use `--copy` where symlinks aren't available.

Manual install works too: copy `skills/repo-hygiene/` into your agent's skills directory.

It's plain Markdown against the open [Agent Skills
specification](https://agentskills.io/specification), so it runs in Claude Code, Codex,
Cursor, Gemini CLI, OpenCode, Goose, Copilot, Amp, and anything else that reads `SKILL.md`.

**Pointing an agent at this repo?** Hand it the link. It should read [AGENTS.md](AGENTS.md).

## Layout

A sub-200-line `SKILL.md` carries the audit procedure. Four reference files hold the
detail, and each pointer in `SKILL.md` says *when* to load the file. The skill practices
the progressive disclosure it preaches.

```
skills/repo-hygiene/
├── SKILL.md                      # the audit procedure — loaded on activation
└── references/                   # loaded on demand
    ├── mechanical-checks.md      # extraction patterns, commands, false-positive traps
    ├── judgment-checks.md        # per-line test, demote/promote ladder, skill boundaries
    ├── report-format.md          # report template, worked example, apply-run rules
    └── principles.md             # why each rule exists, sources, vocabulary
```

## Sources

Distilled from the context-engineering writing of 2025–2026: HumanLayer's blog and
[12-factor agents](https://github.com/humanlayer/12-factor-agents) (Dex Horthy on
intentional compaction, the instruction budget, the leverage hierarchy), the [AI That
Works](https://github.com/ai-that-works/ai-that-works) sessions, Anthropic's
[context-engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
and [Agent Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)
posts, the [Claude Code best practices](https://code.claude.com/docs/en/best-practices),
the [context-rot paper](https://arxiv.org/abs/2606.09090) that supplies the 23% figure, and
Every's [compound engineering](https://every.to/guides/compound-engineering). Full
citations live in
[`references/principles.md`](skills/repo-hygiene/references/principles.md).

The skill has no evals yet. The mechanical checks are deterministic; the judgment checks
are a considered opinion, not a demonstrated improvement.

## Provenance

Written August 2026, prompted by watching an agent-operated repo accumulate exactly the
cruft this skill now hunts.

## License

MIT — see [LICENSE](LICENSE).
