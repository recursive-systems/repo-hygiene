# Mechanical checks

Deterministic passes. Run all of them, every audit, before any judgment work. Findings here
are cheap to verify and hard to argue with, which is why they lead the report.

Adapt commands to the repo's stack. Everything below assumes a POSIX shell and git.

## 1. Stale references

The highest-yield check. Roughly 23% of repos with agent config files reference code
elements that no longer exist (arXiv 2606.09090).

**Extract, then verify.** From every always-loaded file and every SKILL.md:

- **Paths**: anything that looks like `path/to/file.ext`, backtick-quoted paths, markdown
  link targets. Verify with `test -e` from the repo root.
- **Commands**: fenced code blocks and backtick-quoted invocations. Verify the binary exists
  (`command -v`), the script exists, and flags are plausible. Actually run the ones that are
  safe to run (`--help`, `--list`, `--selftest` style). Never run anything with side effects
  during an audit.
- **Symbols**: named functions, classes, make targets, npm scripts. Grep the codebase;
  check `package.json` / `Makefile` / `pyproject.toml` for named tasks.
- **Packages**: named dependencies. Check the lockfile or manifest.
- **URLs in instructions**: fetch only if cheap; otherwise flag ones whose repo/org names no
  longer match reality.

**False-positive traps:**

- Paths created at runtime (output dirs, per-client folders). Check whether a generator
  creates them before flagging.
- Illustrative placeholders (`{lastname}_{street}.png`, `<your-key>`). Template syntax is
  not a stale path.
- Commands meant for a different machine or CI environment. Flag as "unverifiable here,"
  not "broken."

## 2. Size against budget

```sh
wc -l CLAUDE.md AGENTS.md .cursorrules 2>/dev/null
find . -name SKILL.md -not -path "*/node_modules/*" | xargs wc -l | sort -rn
```

Budgets, from HumanLayer's CLAUDE.md guidance and the Agent Skills spec:

| File | Healthy | Ceiling |
|---|---|---|
| Root CLAUDE.md / AGENTS.md | ≤ 60 lines | ~300 lines |
| SKILL.md body | ≤ 200 lines | ~500 lines / ~5k tokens |
| Skill `description` frontmatter | one dense paragraph | 1024 chars (spec limit) |

Count instructions too, not just lines: a 50-line file of dense imperative bullets can blow
the ~150–200 instruction budget on its own. A table of facts is cheaper than a list of
directives.

Over budget is a pointer, not a verdict. The verdict comes from the per-line test in the
judgment pass.

## 3. Committed artifacts

```sh
# largest blobs in history, not just the worktree
git rev-list --objects --all | git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' \
  | awk '/^blob/ {print $3, $4}' | sort -rn | head -20

# generated files tracked at HEAD
git ls-files | grep -Ei '\.(png|jpg|pdf|zip|mp4|mov|sqlite|parquet)$|^(dist|build|out|coverage)/'
```

Distinguish three cases:

- **Generated and regenerable** (build output, coverage, rendered PDFs a script produces):
  recommend untrack + gitignore.
- **Source binaries** (design assets, fixtures): fine if small; if large, recommend a blob
  or manifest pattern (track hashes in git, bytes elsewhere).
- **Already-removed blobs bloating history**: only worth a history rewrite if clone/grep
  pain is real. Note the cost; don't recommend rewrites casually.

## 4. Finished work left live

Find plan-shaped files, then check whether they are done:

```sh
git ls-files | grep -Ei '(^|/)(todo|plan|spec|notes?|scratch|research|thoughts)[^/]*\.md$'
grep -rl -- '- \[ \]' --include='*.md' .   # files with unchecked boxes
```

A plan is "done" when its checkboxes are all checked, its subject shipped (check git log for
the feature), or its last edit predates the work it describes by a long margin. Done plans
recommend *archive*; ambiguous ones go to open questions. A plan is only "live" if something
still points at it or its work is visibly in flight.

## 5. Scratch in the tree

```sh
git status --porcelain              # long-lived untracked files
git ls-files | grep -Ei '(^|/)(tmp|temp|old|backup|copy|v2|final|test[0-9])[^/]*|\.(bak|orig|log)$'
```

Also look for agent-session leftovers: half-written analysis markdown, `output.json` at the
root, screenshots, duplicated files with ` 2` in the name. Untracked files that have sat for
weeks are either scratch to delete or work to commit; the report should force that decision.

## 6. Foreign objects in config directories

Config directories are namespaces agents trust. Anything inside them that is not what the
directory promises will eventually be loaded as if it were.

- Skills dirs (`.claude/skills/`, `.agents/skills/`): every child should be a directory
  containing a valid SKILL.md. Flag workspaces, snapshots, eval scratch, and `-old` copies
  living there.
- `.claude/commands/`, hooks dirs: same test, entries must be the artifact type the
  directory implies.
- Validate skill frontmatter parses (a tool like `npx skills add ./ --list`, or a YAML
  parse). A skill with broken frontmatter silently vanishes, which is a severity: incorrect
  finding (the repo believes it has a guardrail it doesn't have).

## 7. Duplicate strings

```sh
# candidate: repeated fenced commands across markdown
grep -rhn --include='*.md' -E '^\s*(npm|npx|python3?|make|cargo|go|git) ' . | sort | uniq -cd | sort -rn | head
```

Verbatim duplication of commands, prices, versions, or rules across files. Each hit needs a
canonical-home decision in the judgment pass: one file keeps the fact, the others point.
Prose paraphrases of the same rule won't show up here; that hunt is judgment work.
