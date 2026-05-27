# Platform Baseline — Capabilities, Tiers, and Assignment Guidance

> **Calibration stamp:** Platform capability matrix verified against authoritative Anthropic documentation on **2026-05-20**. Re-verify on every skill run via Phase 1.5 Query 3.

This reference is consulted whenever Phase 1 or Phase 1.5 needs to confirm what the chosen deployment surface can actually do. The capability matrix below is the single source of truth for which platform supports what; the assignment guidance section explains why some task types are wedded to specific platforms (most notably, why orchestrated-research can only run on the Claude Code CLI).

## Standing Environment Assumptions (canonical home: SKILL.md)

For the full Standing Environment Assumptions (Max 20x, full 1M context, `claude-opus-4-7` everywhere including subagents, adaptive thinking, no cost-driven routing), see `SKILL.md` § Standing Environment Assumptions. This reference's assignment guidance respects those assumptions: recommendations to route subagents to Haiku/Sonnet for cost reasons are **not** permissible. Where this file mentions Haiku or Sonnet at all, it is to document what the platform supports — not to recommend routing the skill's optimized prompts to them.

## Rows that drive SKILL.md runtime behavior

Three rows govern SKILL.md runtime decisions and are consulted at specific phase boundaries: **Interactive widgets** drives the Widget Interaction Protocol fallback in Phase 1 / 2.5 (when ✗ on the deployment target, plain-text numbered options replace widgets); **Local file write** drives Phase 6B file-write vs claude.ai inline-block path; **Spawn subagents** drives orchestrated-research routing (feasible only where this row is ✓ with first-class parallel dispatch — i.e., the Claude Code CLI). When the deployment target has ✗ or ⚠ on any of these, surface the constraint in Phase 1's contradiction-handling flow.

## Capability matrix

Rows are capabilities relevant to optimized-prompt construction; columns are the four deployment targets the skill recognizes. Cells use the following shorthand: ✓ = first-class supported; ✓ (via X) = supported through a named mechanism; ⚠ = supported with caveats listed below; ✗ = not supported.

| Capability | Claude API | Claude Code CLI | Cowork (Desktop) | claude.ai (chat) |
|------------|------------|-----------------|------------------|------------------|
| Spawn subagents (general-purpose, Explore, Plan, custom) | ✓ (via SDK + Agent tool — build your own dispatch) | ✓ (built-in `Agent`/`Task` tool; `.claude/agents/`, `~/.claude/agents/`) | ⚠ (UI-driven; not parallel-dispatch shaped) | ✗ |
| Local file write (Edit/Write/NotebookEdit) | ✓ (via SDK in your harness) | ✓ (built-in) | ✓ (native — Cowork's primary purpose) | ✗ |
| Read local files | ✓ (via SDK) | ✓ | ✓ | ✗ |
| Web search | ✓ (server tool) | ✓ | ✓ | ✓ |
| Web fetch / retrieval | ✓ (server tool) | ✓ | ✓ | ✓ (chat may surface as citation) |
| MCP server connections | ✓ (connector tool / SDK) | ✓ (`mcpServers` config) | ✓ | ⚠ (some servers; not arbitrary) |
| Interactive widgets (`ask_user_input_v0`) | ✗ | ✗ | ✓ | ✓ |
| Headless `-p` / non-interactive mode | n/a | ✓ (`claude -p`) | ✗ | ✗ |
| Computer use / browser control | ✓ (server tool, beta scope) | ✓ (via MCP / tool) | ✓ (native, Pro/Max plans) | ✗ |
| 1M-token context (Opus 4.7) | ✓ (GA, standard pricing) | ✓ (Max/Team/Enterprise auto; alias `opus[1m]` / `claude-opus-4-7[1m]`) | ⚠ (subscription-tier dependent) | n/a |
| Adaptive thinking | ✓ (must opt in: `thinking:{type:"adaptive"}`) | ✓ (default `xhigh` effort on Opus 4.7) | ✓ | ✓ |
| Custom system prompts | ✓ | ✓ (via `CLAUDE.md`, subagent frontmatter) | ⚠ (skills & persistent memory; less raw control) | ✗ (in product UI) |
| Plugin / Skill packaging | ✓ (Files API + skill bundle) | ✓ (`.skill` archives, `~/.claude/skills/`) | ✓ | ⚠ (limited surface) |
| `output_config.format` / structured outputs | ✓ | ✓ (via SDK underneath) | ✓ | ✗ (no direct exposure) |
| Streaming reasoning summaries | ✓ (set `display:"summarized"`) | ✓ (Ctrl+O to expand) | ✓ | ⚠ (UI-dependent) |
| Persistent agent memory across sessions | ✓ (build your own, Files API) | ✓ (`memory:` frontmatter, `~/.claude/agent-memory/`) | ✓ (native — Cowork's design) | ⚠ (limited) |
| Hooks (lifecycle, pre/post) | n/a (your harness handles) | ✓ (`hooks:` config, `settings.json`) | ⚠ | ✗ |
| Permission modes (auto, plan, dontAsk, …) | n/a (your harness handles) | ✓ | ⚠ | ✗ |

### Cells flagged ⚠ — caveats

- **Cowork — subagent dispatch:** Cowork supports specialized worker agents through its UI, but it is not shaped around the CLI's parallel-dispatch pattern (`Agent` tool fan-out, results synthesis in the orchestrator). For orchestrated-research workflows that the skill produces, the CLI is the right target.
- **Cowork — 1M context:** Subscription-tier dependent. On Max 20x (the standing assumption) it is included.
- **claude.ai — MCP:** Some MCP servers are integrated into the chat product; arbitrary MCP attachment is not a chat capability the way it is on the CLI.
- **claude.ai — streaming summaries:** What renders depends on the chat UI rather than a configurable knob.
- **claude.ai — persistent memory:** Limited to product features (Projects, conversation memory), not arbitrary file-backed memory.
- **CLI / Cowork — permission modes:** Cowork has its own permission surface; the named modes in the table reflect the CLI's vocabulary.

## Subscription tiers

| Tier | Available surfaces | 1M Opus context | Notes |
|------|--------------------|------------------|-------|
| **Pro** | claude.ai, Claude Code (limited), Cowork | Requires usage credits | Entry tier; subject to rate limits the skill's optimized prompts should not assume away. |
| **Max 5x** | claude.ai, Claude Code, Cowork | Included | |
| **Max 20x** | claude.ai, Claude Code, Cowork | Included | **Standing Environment Assumption for this skill.** No rate limits, no token-budget constraints, full context. |
| **Team Standard** | All paid surfaces + org features | Requires usage credits | |
| **Team Premium** | All paid surfaces + org features | Included | Per-seat 1M Opus access. |
| **Enterprise** | All surfaces + SSO, audit, managed settings, custom retention | Included | Admins can pin model versions, restrict `availableModels`, deploy managed subagents. |
| **API pay-as-you-go** | Direct Claude API (`api.anthropic.com`) | Full access | No subscription overlay; per-token billing. |

The skill's `<deployment_config>` block never asks the optimized prompt to conserve tokens. If a user explicitly overrides the standing assumption in Phase 1 (e.g., to a Pro plan), Phase 1.5 must record the override and adjust the chat run sheet's deployment context — but the override does not propagate into the prompt body itself, since the body is a portable instruction and the user runs it under whatever tier they have.

## Platform assignment guidance

Phase 1 confirms the deployment target via Widget Call 1. When the user's selection makes the chosen task type infeasible, surface the conflict and issue a single follow-up widget (per `SKILL.md` Phase 1's "Handling User Selections" section). The cases below are the most common.

### Orchestrated research → Claude Code CLI (required)

Why the CLI is the only valid target:

- The skill's orchestrated-research deliverable is a `CLAUDE.md` that dispatches subagents via the `Agent`/`Task` tool. The CLI is the surface that ships built-in `Agent`/`Task` dispatch with parallel execution, per-subagent context isolation, and per-subagent permission scoping.
- Parallel subagent execution depends on the CLI's `Agent` tool and the `CLAUDE_CODE_SUBAGENT_MODEL` env var (which routes every subagent to Opus 4.7 per the standing assumption). Neither claude.ai chat nor Cowork's UI provides this dispatch model.
- The CLI exposes the file-based subagent definition surface (`.claude/agents/`, `~/.claude/agents/`, `--agents` JSON) that the orchestrator's `CLAUDE.md` references.
- The CLI also exposes the `CLAUDE_CODE_EFFORT_LEVEL`, `MAX_THINKING_TOKENS`, and related env vars that the orchestrated-research deliverable presumes.

If a user selects orchestrated-research as task type but a non-CLI target as deployment, the follow-up widget reframes: either keep orchestrated-research and route to CLI, or change task type to something the chosen surface supports (knowledge work / Cowork-primary; analysis / research running synchronously on chat or API).

### claude.ai (chat) — no subagents, no local file write

claude.ai is the right target for:

- Single-turn or multi-turn analysis the user reads in chat.
- Knowledge-work tasks where the deliverable is the message itself or a downloadable artifact attached to a message.
- Tasks that use claude.ai's first-party tools (web search, web fetch, code execution, computer use where enabled).

It is the wrong target for: orchestrated research, agentic file-writing pipelines, anything depending on `ask_user_input_v0` widgets in a strict-API context, and anything depending on persistent local memory across sessions beyond what the product's Projects feature offers.

### Cowork (Desktop) — knowledge work, native file access

Cowork is purpose-built for working across local files, folders, and desktop applications. Default target for:

- Knowledge-work prompts that read/write local documents.
- Spreadsheet, slide, document workflows.
- Tasks that benefit from persistent agent memory across sessions.

Not the right target for: orchestrated multi-agent research (CLI), pure API integrations the user runs in their own backend (API), or chat-shaped workflows (claude.ai).

### Claude API — your own harness

The right target when:

- The user is building a product or service around Claude Opus 4.7 directly.
- Custom dispatch, custom widgets, custom permissioning are needed.
- Cost or rate behavior must be managed by the harness, not delegated to a Claude UI.

The optimized prompt for an API target is portable — same body whether invoked through Python, TypeScript, Go, or CLI tooling. The `<deployment_config>` block goes in the chat run sheet, not the body, so the user can fold it into whatever request shape their harness builds.

## Phase 1.5 integration

Two of Phase 1.5's four mandatory queries (Query 1 and Query 3) are platform-scoped. The Step 1.5C delta must surface **two or more platform-unique features** for the confirmed deployment target. Treat this matrix as a starting point — the matrix can drift as Anthropic ships new surfaces, and Phase 1.5's Query 5 is the mechanism that catches the drift.

## Sources

Accessed 2026-05-20:

- [Models overview — Claude API Docs](https://platform.claude.com/docs/en/about-claude/models) — model availability across surfaces and pricing.
- [Model configuration — Claude Code Docs](https://code.claude.com/docs/en/model-config) — Claude Code aliases, 1M-context routing, env vars including `CLAUDE_CODE_SUBAGENT_MODEL`.
- [Create custom subagents — Claude Code Docs](https://code.claude.com/docs/en/sub-agents) — built-in subagents (Explore / Plan / general-purpose), subagent scopes (`.claude/agents/`, `~/.claude/agents/`, plugins, managed), `--agents` CLI flag.
- [Claude Platform](https://www.anthropic.com/api) — API product surface.
- [Claude Code by Anthropic](https://www.anthropic.com/claude-code) — CLI product surface.
- [Claude Cowork](https://www.anthropic.com/product/claude-cowork) — Cowork product surface and tier eligibility.
- [Migration guide — Claude API Docs](https://platform.claude.com/docs/en/about-claude/models/migration-guide) — Opus 4.7 capabilities carrying forward from 4.6 (Files API, MCP connector, batch, vision, web tools, memory tool).
