# prompt-optimization

A single-plugin [Claude Code](https://code.claude.com/docs/en/overview) marketplace hosting the **prompt-optimization** plugin: a phased workflow that turns a draft prompt, system message, `CLAUDE.md`, or multi-agent orchestrator into production-grade instructions for Claude Opus 4.7.

## Install

### Claude Code (CLI)

```bash
claude plugin marketplace add ZaBrisket/prompt-optimization
claude plugin install prompt-optimization@prompt-optimization
```

Or, from within an interactive Claude Code session:

```
/plugin marketplace add ZaBrisket/prompt-optimization
/plugin install prompt-optimization@prompt-optimization
```

The first command registers this repo as a marketplace; the second installs the plugin from it (`<plugin>@<marketplace>`). Restart Claude Code if prompted so the new skill, command, and agent load.

### Cowork (Desktop)

1. In the Cowork desktop app, open the **Customize** menu and add this repo (`ZaBrisket/prompt-optimization`) as a marketplace.
2. Install **prompt-optimization** from the marketplace.

### Local development

```bash
claude plugin marketplace add /path/to/this/repo
claude plugin install prompt-optimization@prompt-optimization
```

## What you get

- **Skill — `prompt-optimization`** — auto-triggers when you provide a draft prompt to improve, ask to optimize/harden/refine a prompt, or describe a multi-agent research workflow. Runs a sequential protocol: intent extraction, complexity triage, widget-based clarification, mandatory live calibration against Anthropic documentation, landscape research, draft analysis, prompt construction, delta analysis, and a final QC pass with a clean file write.
- **Command — `/optimize-prompt <draft | file path | description>`** — an explicit entry point that hands off to the skill and runs the full protocol without skipping its interaction gates.
- **Agent — `landscape-research`** — a read-only research worker the skill dispatches in parallel during its orchestrated Phase 2 (mode 2-O) to deepen landscape research across multiple lanes, then synthesizes the results centrally.

## Orchestrated landscape research (mode 2-O)

When the Agent/Task tool is available and a draft is multi-sub-domain, multi-stakeholder, or contested, Phase 2 fans out: one `landscape-research` lane per sub-domain or query-taxonomy cluster, plus a dedicated adversarial-pass lane, each searching deeply in its own context. The orchestrator collects the findings packs and runs synthesis and the neutrality / coverage-bias gates on the aggregate corpus. Depth is bounded by ≤6 subagents, per-lane search budgets, and at most one remediation pass. When subagents are unavailable (claude.ai chat, bare API), Phase 2 falls back to the single-threaded path automatically.

## Repository layout

```
.
├── README.md                                # this file
├── .claude-plugin/marketplace.json          # marketplace definition
└── plugins/
    └── prompt-optimization/
        ├── .claude-plugin/plugin.json        # plugin manifest
        ├── skills/prompt-optimization/       # SKILL.md + references/
        ├── commands/optimize-prompt.md       # /optimize-prompt
        ├── agents/landscape-research.md       # research worker
        └── README.md
```

## Author

Mac Zabriskie
