# AGENTS.md

Instructions for an AI agent pointed at this repository.

## What this repo is

One [Agent Skill](https://agentskills.io): `repo-hygiene`. It audits repositories that AI
agents operate in — the CLAUDE.md/AGENTS.md layer, skills, docs, and tree — for stale
instructions, duplication, misplaced guidance, and leftover scratch, and produces a ranked
findings report the human decides on.

The skill lives at [`skills/repo-hygiene/`](skills/repo-hygiene/). Nothing else in this repo
is required to use it.

## If you were given this link and asked to use the skill

Read [`skills/repo-hygiene/SKILL.md`](skills/repo-hygiene/SKILL.md) and follow it. That file
is the whole procedure. It tells you which files in
[`skills/repo-hygiene/references/`](skills/repo-hygiene/references/) to load and when — load
them on demand, not up front.

You do not need to install anything to do this.

## If you were asked to install it

```bash
npx skills add recursive-systems/repo-hygiene        # project-local
npx skills add recursive-systems/repo-hygiene -g     # global
```

`npx skills` ([vercel-labs/skills](https://github.com/vercel-labs/skills)) detects which
agents are present, resolves each one's skills directory (`.claude/skills/`,
`.agents/skills/`, `~/.codex/skills/`, …), and symlinks a single canonical copy into each.
Add `--copy` where symlinks aren't available, or `-a claude-code -a codex` to target
specific agents.

Manual install: copy `skills/repo-hygiene/` into the agent's skills directory.

Verify with `npx skills add recursive-systems/repo-hygiene --list` — it should print the
`repo-hygiene` skill. If it prints "No skills found," the frontmatter is malformed; see
[Editing this skill](#editing-this-skill).

## Layout

```
skills/repo-hygiene/
├── SKILL.md          # the audit procedure — loaded when the skill activates
└── references/       # loaded individually, on demand
    ├── mechanical-checks.md   # deterministic checks: commands, patterns, traps
    ├── judgment-checks.md     # per-line test, demote/promote, skill boundaries
    ├── report-format.md       # report template, worked example, apply-run rules
    └── principles.md          # sources and the reasoning behind each rule
```

## Editing this skill

Follow the [Agent Skills specification](https://agentskills.io/specification) and the
[authoring best practices](https://agentskills.io/skill-creation/best-practices).

Constraints that will silently break the skill if violated:

- **`name` must stay `repo-hygiene`** and must match the directory name exactly, or no
  client will load it.
- **`description` must parse as YAML.** It is deliberately double-quoted — it contains
  commas, dashes, and apostrophes, and an unquoted scalar containing `: ` is a parse error
  that makes the skill vanish with no warning beyond a "skipped" line. Keep the quotes.
- **`description` max 1024 characters.** It runs long already; check after editing.
- **Keep `SKILL.md` under 500 lines / ~5,000 tokens.** It loads in full on activation. New
  detail belongs in `references/`.
- **Every reference pointer must say *when* to load the file**, not just that it exists.
  That is what makes progressive disclosure work — and this skill audits other repos for
  exactly this mistake, so don't ship it here.
- **Keep reference links one level deep** from `SKILL.md`.

After any frontmatter change, re-verify with:

```bash
npx skills add ./ --list
```

Silence or "No skills found" means it's broken. Run this before every commit that touches
frontmatter — a regex or eyeball pass will not catch a YAML error.

## Scope

This skill covers the instruction and documentation layer of a repo. It is explicitly
**not** for code refactoring, dependency upgrades, or performance work, and its audit is
read-only: it recommends, the human decides, and nothing is deleted without approval.
