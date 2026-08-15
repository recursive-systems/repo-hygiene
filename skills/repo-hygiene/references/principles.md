# Principles and sources

Why the checks are what they are. Load this when a recommendation is challenged, when the
user asks why a finding matters, or when a report summary should teach rather than just list.

## The core mechanics

**Attention budget / instruction budget.** Frontier models follow roughly 150–200
instructions with reasonable consistency; adherence degrades from there, and smaller models
degrade much faster. The harness's own system prompt spends ~50 of those before the repo
says anything. Bigger context windows do not raise this budget. Every always-loaded line is
a withdrawal from the same account, which is why the per-line test ("would removing this
cause mistakes?") is the central judgment check.
Sources: HumanLayer, "Writing a good CLAUDE.md" (humanlayer.dev/blog/writing-a-good-claude-md);
"Long-Context Isn't the Answer" (humanlayer.dev/blog/long-context-isnt-the-answer);
Anthropic, "Effective context engineering for AI agents"
(anthropic.com/engineering/effective-context-engineering-for-ai-agents).

**Context rot, both senses.** Anthropic's sense: as tokens in the window grow, recall of any
given token drops, so noise has a real cost even when every line is true. The arXiv sense
(2606.09090, "Context Rot in AI-Assisted Software Development"): agent config files go stale
mechanically; a scan of 356 repos found stale code-element references in 23% of
CLAUDE.md/AGENTS.md/.cursorrules files. The paper's remedy, documentation-consistency
checking applied to config files, is the mechanical pass of this skill.

**The severity hierarchy.** Incorrect beats missing beats noise, from HumanLayer's ACE-FCA
("Advanced Context Engineering for Coding Agents",
humanlayer.dev/blog/advanced-context-engineering): wrong context actively misleads, missing
context merely under-informs, noise taxes attention. Agents amplify the first: "a stale
cache is worse than an empty one" — a human reads a stale README skeptically, an agent
executes it (dev.to/wolfejam, "Your AGENTS.md Is Already Stale").

**The leverage hierarchy.** CLAUDE.md > prompts > research > plans > code. A bad line of
code is one bad line; a bad line of a plan yields hundreds of bad lines of code; a bad line
of CLAUDE.md poisons every session in the repo. This is why the audit works top tier down
and why an always-loaded finding outranks a same-severity finding in a doc.
Source: ACE-FCA and the AI That Works session on it
(github.com/ai-that-works/ai-that-works, episode 2025-08-05).

**Progressive disclosure / just-in-time context.** The repo's instruction layer should work
like a manual: table of contents (frontmatter descriptions), chapters (skill bodies),
appendix (references), each loaded only when needed, with lightweight identifiers (paths,
pointers) in the upper tiers instead of inlined content. This is what makes demotion the
default fix: correct content in an expensive tier isn't deleted, it's moved down.
Sources: Anthropic, "Equipping agents for the real world with Agent Skills"
(anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills);
Anthropic context-engineering post above.

**Deterministic beats advisory.** Prose instructions are advisory and cost budget every
session; hooks, linters, tests, and CI are enforced and cost nothing at prompt time. "Never
send an LLM to do a linter's job." Repeated always-do-X prose is a conversion candidate, not
a formatting fix.
Sources: HumanLayer CLAUDE.md post; Claude Code best practices
(code.claude.com/docs/en/best-practices), which also supplies the per-line test and the
"over-specified CLAUDE.md" failure pattern.

**Reactive configuration.** Config added speculatively (skills nobody triggered,
auto-generated agent files never hand-pruned) tends to hurt: an ETH Zurich result found
LLM-auto-generated agent files reduced performance while costing 20% more. Add instructions
against observed failures; discard configurations that earn nothing. The Hashimoto rule is
the positive form: every observed agent mistake gets engineered away, once.
Source: HumanLayer, "Skill Issue: Harness Engineering for Coding Agents"
(humanlayer.dev/blog/skill-issue-harness-engineering-for-coding-agents).

**Compaction and artifact lifecycle.** Healthy agent workflows produce markdown artifacts
on purpose: research docs, plans, compaction summaries (ACE-FCA's "frequent intentional
compaction"; HumanLayer's `thoughts/` directory). Hygiene's job is not to ban these but to
manage their lifecycle: quarantined from always-loaded context, retrieved on demand, and
archived once done, with older material kept at decreasing resolution ("decaying-resolution
memory", AI That Works 2025-07-15). A finished plan left readable-as-current is how last
quarter's intent becomes this quarter's bug.

**Compounding.** "Each unit of engineering work should make subsequent units easier, not
harder" (Every, "Compound Engineering", every.to/guides/compound-engineering). The
instruction layer is supposed to accumulate captured learnings — that is the promote half of
the ladder. Accumulation without periodic consolidation is how it becomes rot, which is why
this audit is a recurring practice, not a one-time cleanup.

## Vocabulary worth using in reports

- **context engineering** — curating the smallest high-signal token set for a task
- **context rot** — recall degradation with window growth; also staleness in config files
- **attention budget / instruction budget** — the finite adherence capacity above
- **progressive disclosure** — TOC → chapter → appendix loading
- **just-in-time context** — pointers in context, content at runtime
- **context firewall** — subagent isolation of noisy exploration
- **intentional compaction** — distilling state into a verified artifact + fresh window
- **the dumb zone** — degraded high-utilization regime; healthy target ~40–60%
- **canonical home** — the one place a fact lives; everything else points
- **demote / promote** — moving content down or up the loading tiers
- **co-change rule** — code and the config describing it change in the same commit

## Full source list

- humanlayer.dev/blog/advanced-context-engineering (canonical:
  github.com/humanlayer/advanced-context-engineering-for-coding-agents)
- humanlayer.dev/blog/writing-a-good-claude-md
- humanlayer.dev/blog/stop-claude-from-ignoring-your-claude-md
- humanlayer.dev/blog/long-context-isnt-the-answer
- humanlayer.dev/blog/skill-issue-harness-engineering-for-coding-agents
- github.com/humanlayer/12-factor-agents (esp. #3 own your context window,
  #9 compact errors, #10 small focused agents)
- github.com/ai-that-works/ai-that-works (episodes: 2025-08-05 context engineering for
  coding agents, 2025-07-15 decaying-resolution memory, 2026-03-10 skills deep dive)
- anthropic.com/engineering/effective-context-engineering-for-ai-agents
- anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills
- code.claude.com/docs/en/best-practices
- arxiv.org/abs/2606.09090 (context rot in AI config files, the 23% figure)
- dev.to/wolfejam/your-agentsmd-is-already-stale-and-your-agent-trusts-it-completely-2nfh
- every.to/guides/compound-engineering
