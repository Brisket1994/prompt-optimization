# Task-Type Heuristics & Opus 4.7 Anti-Patterns

> **Calibration stamp:** Anti-patterns and deprecation claims verified against authoritative Anthropic documentation on **2026-05-20**. Re-verify via Phase 1.5 on every run.

This reference is consulted by Phase 3 (Draft Analysis) and by Phase 4 (Produce Optimized Prompt) to apply task-type-specific enhancements and to sweep for Opus 4.7 anti-patterns. It is loaded twice per run — once for the task-type guidance that matches the Phase 1 selection, and once for the named anchor `§Opus 4.7 Anti-Patterns` which Phase 3 cites directly.

## Table of contents

1. [How to use this reference](#how-to-use-this-reference)
2. [Task-type guidance](#task-type-guidance)
   1. [Code generation](#code-generation)
   2. [Data analysis](#data-analysis)
   3. [Research / search](#research--search)
   4. [Writing / content](#writing--content)
   5. [Agent / workflow](#agent--workflow)
   6. [Orchestrated multi-agent research](#orchestrated-multi-agent-research)
   7. [Creative](#creative)
   8. [Multi-modal](#multi-modal)
   9. [API / integration](#api--integration)
   10. [Multi-model](#multi-model)
   11. [Knowledge work (Cowork-primary)](#knowledge-work-cowork-primary)
3. [Opus 4.7 Anti-Patterns](#opus-47-anti-patterns)
4. [Sources](#sources)

---

## How to use this reference

Phase 3 cross-references this file against the user's draft. For each task-type section relevant to the Phase 1 selection, apply:

- **Enhancements** — concrete patterns that Opus 4.7 specifically rewards.
- **Anti-patterns** — things to strip out if the draft contains them.
- **Missing-enhancement checks** — gaps Phase 3 should flag for Phase 4 to address.

Then run the [Opus 4.7 Anti-Patterns](#opus-47-anti-patterns) sweep against the draft regardless of task type — those anti-patterns are model-wide, not task-specific.

## Universal rule — Completion verification for any file-writing prompt

Any optimized prompt that writes to disk — regardless of primary task type (code generation, agent/workflow, Cowork knowledge work, API integration producing artifacts, multi-modal saving extracted data, orchestrated-research deliverables) — must include a completion-verification clause. Opus 4.7 occasionally misreports completion (`SKILL.md` Meta-Rule 11; `## Opus 4.7 Anti-Patterns` row 7 below). Pattern: "After any file write, re-read the file (`ls`, `cat`, file-tool re-read) to verify the change landed before reporting done. If verification fails, retry once; if the second attempt also fails, surface the failure rather than reporting success."

Per-section task-type guidance below assumes this rule and does not restate it.

## Task-type guidance

### Code generation

**Opus 4.7-specific enhancements.**

- Default to `xhigh` effort. Code and agentic tasks are exactly the workload Opus 4.7's `xhigh` ladder rung was added for. Use `high` only when latency clearly matters more than depth; use `max` when you need the deepest reasoning and you've benchmarked it against `xhigh`.
- Adaptive thinking should be explicit in the chat run sheet's `<deployment_config>` block — Opus 4.7 doesn't think by default on the raw API.
- For agentic file modification, instruct Edit/Write/NotebookEdit use directly (`SKILL.md` Meta-Rule 14). Don't pre-specify a manual tool inventory beyond what the task obviously requires.
- Opus 4.7 spawns **fewer** subagents by default than 4.6. For multi-file refactors or large investigations, explicitly authorize subagent dispatch ("delegate cross-file searches to subagents"); the model will not infer this from a draft that simply says "be thorough."
- Opus 4.7 uses **fewer** tool calls by default. If the work needs heavy tool use (file traversal, grep-then-edit), instruct it: "Use Edit / Grep / Read freely — investigating before writing is preferred."
- Include over-engineering guardrails by default. `SKILL.md` Meta-Rule 11 spells these out: don't add error handling that can't fire, don't introduce premature abstractions, don't write speculative comments. These guardrails benefit Opus 4.7 specifically because its more literal instruction-following will apply your standards exactly.

**Anti-patterns to strip.**

- "Think step by step" / "think carefully" prompting where adaptive thinking is enabled. Those phrases are no-ops at best; at worst they fight the adaptive surface. Raise effort instead.
- `temperature` / `top_p` / `top_k` tuning in the draft. 400 error on Opus 4.7.
- `budget_tokens` or `thinking: {type: "enabled", ...}` syntax. Replaced by adaptive + effort.
- Manual prompt-caching directives that pre-judge what should be cached.

**Missing-enhancement checks.**

- Does the prompt specify the language/framework/runtime? Opus 4.7 is more literal; ambiguity isn't filled by inference.
- Are success criteria testable? "Produce a working X" needs the "working" criterion defined.
- Is the file-writing scope explicit? "Edit the project" without scoping invites broad changes.

### Data analysis

**Opus 4.7-specific enhancements.**

- For analytical depth, use `high` or `xhigh` effort. `medium` will scope too tightly for ambiguous analyses.
- Specify input data shape and output structure separately — Opus 4.7 won't generalize from "make a chart" to "save as PNG with these axes" without being told.
- For controversial or contested analyses (e.g., causal claims, contested benchmarks), the optimized prompt instructs the executor to surface both sides; never pre-bake the conclusion. This is Phase 2's neutrality discipline carried forward into the prompt.
- When the analysis needs current data, instruct web search use directly (Meta-Rule 14); for offline data, instruct file tools directly.
- For long-document summarization, lean on the 1M context — partition into sub-windows under 256K rather than swapping models for cost reasons (the standing environment forbids cost-driven swaps anyway).

**Anti-patterns to strip.**

- Embedded conclusions ("show that X causes Y") — replace with "evaluate whether X causes Y, presenting both sides."
- Pre-fixed output verbosity ("write exactly 500 words"). Opus 4.7 calibrates length to complexity; rigid length targets fight that.
- "Don't hallucinate" instructions. Use sourcing requirements ("cite the data source for every claim") instead.

**Missing-enhancement checks.**

- Is the analysis temporal scope defined? (Last 12 months, all-time, snapshot date)
- Are coverage axes (geography, source type, stakeholder voice) flagged for the executor to address, per Phase 2's coverage-bias discipline?
- Are edge cases enumerated so the executor knows which to address?

### Research / search

**Opus 4.7-specific enhancements.**

- Instruct web search use directly when the task obviously needs current data. Adaptive thinking will choose when to invoke; you don't need to write a manual tool allowlist.
- For multi-source research, surface the source-type diversity requirement (Phase 2.2's 3-of-5 categories) in the optimized prompt: "Cover primary documents, expert commentary, practitioner press, mainstream press, and dissenting press where each is available."
- For research that benefits from independent perspectives, consider whether the task is actually orchestrated-research — fan out → synthesize is what subagents are for.
- The 1M context is GA — research that needs to ingest hundreds of pages of source material can do so in one window.

**Anti-patterns to strip.**

- "Search comprehensively" without dimensions. Opus 4.7 won't fill the scope; specify the query taxonomy and source diversity expected.
- Pre-baked positions ("research the evidence that X is true"). Reframe as "research the evidence on X, presenting both sides."
- Manual citation format demands that fight the model's tool outputs.

**Missing-enhancement checks.**

- Is the query taxonomy (consensus / dissent / practitioner / academic / recent / failure cases / adjacent / non-English) specified?
- Are the deliverable's structure and length defined?
- Does the prompt instruct the executor on how to handle conflicting sources?

### Writing / content

**Opus 4.7-specific enhancements.**

- Opus 4.7's more direct tone may produce a different voice than 4.6 did. If the project depends on a specific voice, write 1–2 positive examples of the desired voice into the prompt — positive examples outperform negative instructions.
- Response length calibrates to task complexity. Instead of "write 800 words," specify "produce a piece covering X / Y / Z at the depth a sophisticated reader on this topic would expect."
- For brand-voice or compliance-sensitive content, instruct explicitly: "Avoid claims about [list]. Verify against [source] for [list]."
- For prose, instruct continuation style if it matters: "On follow-ups, expand only sections the user named — do not rewrite untouched material."

**Anti-patterns to strip.**

- Excessive emphasis (`CRITICAL: ALWAYS DO X!!!`). Strip aggressive emphasis markers — they don't outperform neutral imperatives on Opus 4.7 (Meta-Rule 10).
- Long lists of negative instructions. Translate to positive examples.
- Prefill-style directives ("begin your response with…"). Prefilling returns 400; replace with system-prompt direction.

**Missing-enhancement checks.**

- Is the audience defined?
- Are tone, register, and any style guide named?
- Are length and format expectations articulated as task-complexity bounds rather than fixed counts?

### Agent / workflow

**Opus 4.7-specific enhancements.**

- Instruct careful action discipline: hard-to-reverse operations (force-pushes, destructive deletes, schema changes) should require explicit confirmation rather than proceeding on the model's judgment.
- For agentic loops, lean on built-in progress updates — don't scaffold "summarize every 3 tool calls"; Opus 4.7 already does this, and the scaffolding fights its native cadence.
- For agentic file modification, instruct Edit/Write/NotebookEdit use; let adaptive thinking pick the rest.
- If the agent has a permission surface (Claude Code, Cowork), state the permission boundary in the prompt body so the executor understands the action scope.

**Anti-patterns to strip.**

- "Try to" / "if possible" hedging — Opus 4.7 reads these literally and may treat the requirement as optional.
- Excessive `IMPORTANT` / `MUST` markers around instructions that don't actually warrant the emphasis. Sparse emphasis carries more signal.
- Open-ended "do whatever you think is best" framings on consequential actions.

**Missing-enhancement checks.**

- Is the success criterion testable from outside?
- Is the action scope bounded? (Files, directories, branches, services)
- Are reversibility guardrails in place for destructive actions?

### Orchestrated multi-agent research

This deserves a full operational guide — see `references/orchestrated-research.md`. Key heuristics:

**Opus 4.7-specific enhancements.**

- The orchestrator MUST be instructed explicitly to dispatch subagents (`SKILL.md` Meta-Rule 13). 4.7 under-delegates by default. "Use subagents" is not enough — the prompt should say *when, why, and how many*.
- Subagent prompts must never embed unverified numerical estimates. Use the shared-constants pattern or instruct each subagent to verify independently. (Meta-Rule 13.)
- Synthesis is the primary quality gate — design it as the heaviest section of the orchestrator.
- The deliverable is a `CLAUDE.md`. Structure per `orchestrated-research.md` Section 10. The orchestrator template in `prompt-template.md` cross-references that section rather than restating it.
- Environment: `CLAUDE_CODE_SUBAGENT_MODEL=claude-opus-4-7[1m]` per the Standing Environment Assumption — no Sonnet/Haiku subagents.

**Anti-patterns to strip.**

- "The orchestrator decides whether to dispatch" — too soft. The model will choose not to dispatch by default.
- Reused identical subagent prompts that don't differentiate scope (each subagent gets the same instructions → results overlap, synthesis adds nothing).
- Synthesis as a thin "summarize the results" step — it's the primary quality gate, not a postscript.

**Missing-enhancement checks.**

- Are subagent scopes mutually exclusive (or, if not, is overlap deliberate and reconciled in synthesis)?
- Does each subagent prompt specify output scope (length, format, citation discipline)?
- Does the orchestrator specify how to handle subagent failures (remediation pass, partial-result synthesis)?

### Creative

**Opus 4.7-specific enhancements.**

- For creative work the more direct 4.7 tone may need explicit dial-up of warmth, playfulness, or stylistic variance. Positive examples of the target voice outperform "be more creative" instructions.
- Instruct constraint shapes (form, meter, character count, structural constraints) explicitly. Opus 4.7's literalism makes it good at constrained creative work when the constraints are spelled out.
- For iterative drafts, instruct "revise only X" or "preserve Y" explicitly — Opus 4.7 will respect tight scopes.

**Anti-patterns to strip.**

- "Be creative" / "think outside the box" — empty injunctions that don't tell the model what success looks like.
- Excessive scaffolding around creative output (e.g., elaborate frameworks for short-form content).

**Missing-enhancement checks.**

- Is the genre / form / audience defined?
- Are the constraints (length, structure, tone) listed?

### Multi-modal

**Opus 4.7-specific enhancements.**

- Opus 4.7 supports high-resolution images (up to 2576px long edge / ~3.75MP). For screenshot understanding, computer-use loops, and document analysis this is a step change vs. 4.6.
- Image tokens can be ~3× higher than on 4.6 at full resolution. Re-budget `max_tokens` accordingly, or downsample before sending if fidelity isn't needed.
- Pointing and bounding-box coordinates returned by the model are 1:1 with image pixels on Opus 4.7 — remove any scale-factor conversion left over from 4.6.
- For mixed image-text deliverables, specify which dimensions of the image should be addressed and how text and image should relate in the output.

**Anti-patterns to strip.**

- Bounding-box conversion logic in instructions assuming 4.6's scaling.
- Manual image-region attention directions ("look at the top-left") that the model can navigate itself given a high-res image.

**Missing-enhancement checks.**

- Is the image resolution / source defined?
- Are vision-specific guardrails in place (PII redaction, sensitive-content handling)?

### API / integration

**Opus 4.7-specific enhancements.**

- The `<deployment_config>` block belongs in the chat run sheet, never in the prompt body or the written file. It covers: model ID, thinking mode, effort, max output tokens, context window, structured outputs format, batch vs. messages endpoint, any beta headers (e.g., `task-budgets-2026-03-13` if used).
- For structured outputs, prefer `output_config.format` with a JSON schema. The fallback for environments without schema support is "Output ONLY valid JSON with no preamble" (Meta-Rule 6).
- Instruct the user's harness on completion-handling: `refusal` stop reason, `model_context_window_exceeded` stop reason, and the standard `end_turn` / `max_tokens` / `tool_use` cases.
- For long-running agentic loops on API, consider task budgets (beta `task-budgets-2026-03-13`) as an advisory cap — `task_budget` is visible to the model and steers pacing; `max_tokens` is a hard ceiling the model doesn't see.

**Anti-patterns to strip.**

- Inline `temperature` / `top_p` / `top_k` (400 on 4.7).
- `output_format` top-level — migrate to `output_config.format`.
- `betas=["interleaved-thinking-2025-05-14"]`, `betas=["effort-2025-11-24"]`, `betas=["fine-grained-tool-streaming-2025-05-14"]` — all GA on 4.7.

**Missing-enhancement checks.**

- Is the model ID specified? (`claude-opus-4-7` — exact)
- Is the request shape (Messages API, Batches, with thinking, with output_config) explicit?
- Are streaming-vs-blocking expectations stated?

### Multi-model

This task type covers prompts whose **subject matter** is multi-model orchestration — for example, optimizing a prompt that will dispatch work across Opus, Sonnet, and Haiku based on subtask shape. **Importantly:** this does not authorize the skill to recommend non-Opus routing for the optimized prompt's own execution. Per `SKILL.md` § Standing Environment Assumptions: Opus 4.7 everywhere for the skill's own runtime. What the optimized prompt's subject-matter system does is the user's design choice; what the skill's optimized prompts run on is fixed.

**Opus 4.7-specific enhancements.**

- The optimized prompt may describe model-routing decisions the user's harness should make ("route classification subtasks to Haiku, route synthesis to Opus") — that's subject-matter, not skill-runtime.
- When the harness includes a multi-model dispatch, instruct the user's prompts to specify expected model behaviors per route (effort levels, output shapes) so each model receives appropriately tuned input.
- Cross-model migration scaffolding (e.g., "this prompt was written for Opus 4.6 originally; adapt for Sonnet 4.6 fallback") should be explicit if relevant.

**Anti-patterns to strip.**

- Implicit assumption that a single prompt works identically across models. Opus 4.7's `xhigh` effort level isn't available on 4.6; output shape calibration differs.
- Cost-driven routing recommendations in the skill's optimized prompt itself — Standing Environment forbids them.

**Missing-enhancement checks.**

- Is the model-routing rule explicit?
- Are per-route effort levels and output shapes specified?

### Knowledge work (Cowork-primary)

**Opus 4.7-specific enhancements.**

- Cowork is the right target for tasks that work across local files, folders, and desktop applications. Instruct the executor to use file tools directly when the task obviously requires them.
- Cowork's persistent agent memory is the right surface for cross-session continuity. If the prompt is meant to accumulate context over time, instruct memory writes explicitly.
- Skills are first-class on Cowork — the optimized prompt may reference user-installed skills by name when the task obviously calls for them.
- For document-editing workflows (Word, slides, sheets), instruct the deliverable shape explicitly — Cowork can produce files in many formats.

**Anti-patterns to strip.**

- API-shaped configuration assumptions (effort levels via `output_config`) — Cowork manages thinking via its own surface.
- Chat-shaped expectations (Cowork is task-shaped; output lands as files / actions, not as chat messages).

**Missing-enhancement checks.**

- Is the file-write scope bounded?
- Are application targets (Word, Excel, PowerPoint, OS apps) named where relevant?
- Is the permission scope (read-only, edit, delete) explicit?

## Opus 4.7 Anti-Patterns

Phase 3 cites this section by exact heading — keep the heading verbatim. The table covers model-wide anti-patterns that apply across task types. Apply every row that matches the draft; flag in Phase 5 delta which rows triggered fixes.

| # | Anti-pattern | Why it's wrong on Opus 4.7 | Fix | Severity |
|---|--------------|----------------------------|-----|----------|
| 1 | **Aggressive emphasis markers** (`CRITICAL!!!`, `ALWAYS`, `NEVER` in dense clusters, ALL-CAPS commands) | Opus 4.7 reads instructions literally and doesn't reward emphasis intensity. Dense markers obscure the signal of the instructions that *actually* matter. (Meta-Rule 10.) | Strip emphasis to where it carries real weight. Use neutral imperatives. Reserve `IMPORTANT`/`CRITICAL` for the one or two genuinely consequential instructions. | Medium |
| 2 | **Deprecated parameter — `budget_tokens`** (manual extended thinking) | Returns 400 on Opus 4.7. Manual extended thinking is gone. | Replace with `thinking: {type: "adaptive"}` plus `output_config.effort`. Document in the chat run sheet's `<deployment_config>`, never the prompt body. | Blocking |
| 3 | **Deprecated parameter — `output_format`** (top-level structured outputs) | Deprecated; replaced by `output_config.format`. Still functional but flagged for removal. | Migrate to `output_config={"format": {...}}`. Update any inline JSON-schema directives accordingly. | Blocking |
| 4 | **Deprecated parameter — prefilled assistant messages** | Returns 400 on Opus 4.7 (carried over from 4.6). | Use `output_config.format` with a JSON schema, or system-prompt directives. For preamble suppression: "Respond directly without preamble. Do not start with 'Here is…' or 'Based on…'." | Blocking |
| 5 | **Deprecated parameter — `temperature` / `top_p` / `top_k`** (non-default values) | Non-default values return 400 on Opus 4.7. | Omit these parameters entirely. Steer behavior through prompting. | Blocking |
| 6 | **Deprecated beta headers** — `effort-2025-11-24`, `fine-grained-tool-streaming-2025-05-14`, `interleaved-thinking-2025-05-14`, `token-efficient-tools-2025-02-19`, `output-128k-2025-02-19` | All GA on Opus 4.7. Headers have no effect; they signal stale calibration. | Remove from the chat run sheet's `<deployment_config>`. | Medium |
| 7 | **Guardrail gap — file-write completion misreport** | Opus 4.7 occasionally claims a file was written when it was not. (`SKILL.md` Meta-Rule 11.) | For any prompt that writes to disk, include a completion-verification clause: "After any file write, re-read the file to verify the change landed before reporting done." | High |
| 8 | **Guardrail gap — destructive actions on the model's judgment** | Opus 4.7's more direct, autonomous behavior makes it more likely to take irreversible actions on its own read of the situation. | Specify reversibility guardrails for destructive actions (force-push, `rm -rf`, schema changes). Pattern: "For destructive actions, surface the action and ask for explicit confirmation before proceeding." | High |
| 9 | **Guardrail gap — code over-engineering** | Opus 4.7's literal instruction-following tends to apply every requirement strictly, including loose "make it robust" framings, producing speculative error handling and premature abstraction. | Instruct: "Add error handling only for failures that can actually occur at runtime. Trust internal code and framework guarantees. Validate at system boundaries only." | High |
| 10 | **Orchestrator under-delegation** (multi-agent prompts that name subagents but don't dispatch them) | Opus 4.7 spawns fewer subagents by default than 4.6. Vague dispatch language ("use subagents when helpful") leaves the work on the orchestrator. (Meta-Rule 13.) | Instruct the orchestrator explicitly: "Dispatch parallel subagents using the Agent / Task tool. Give each subagent its own scope and output shape. Do not collapse parallel work into a single sequential thread." | High |
| 11 | **Tool-use defaults — manual tool allowlists in prompt body** | Opus 4.7's adaptive thinking decides which tools to invoke. Pre-specifying a manual allowlist in the prompt body fights this and tends to over-restrict. (Meta-Rule 14.) | Instruct tool use only when the task obviously requires a specific tool (web search for current data; Edit/Write for file modification). Otherwise let adaptive thinking decide. Never embed pre-specified tool inventories. | Medium |
| 12 | **Tool-use defaults — fewer-tool-calls floor** | Opus 4.7 uses fewer tool calls by default. Tasks that need heavy tool use (deep file traversal, multi-page web research) will under-tool unless instructed. | For tool-heavy tasks: "Use [tool] freely — investigating before committing is preferred. Do not skip tool calls because the answer seems inferable." Combined with `high`/`xhigh` effort, this restores tool richness. | Medium |
| 13 | **Style — "think step by step" prompting on adaptive thinking** | No-op at best, friction at worst. Adaptive thinking already manages reasoning depth. | Remove the directive. Raise effort if reasoning seems shallow. | Low |
| 14 | **Style — open-ended hedging language** (`try to`, `if possible`, `maybe`) | Opus 4.7 reads literally; hedges turn requirements into options. | Restate as imperatives: "Do X. If Y is not possible, surface the blocker and stop." | Medium |
| 15 | **Style — negative-only instructions** (long lists of "don't do X") | Opus 4.7 responds better to positive exemplars of the target behavior than to enumerated prohibitions. | For each "don't do X", provide a positive example of the desired alternative. | Low |
| 16 | **Verbosity floor / ceiling assumptions** (`write 500 words`) | Opus 4.7 calibrates length to task complexity. Rigid length targets fight the calibration. | Replace with complexity-shaped guidance: "Produce a piece covering X / Y / Z at the depth a sophisticated reader would expect." | Medium |
| 17 | **Token-conservation language in prompt body** | The Standing Environment Assumption is full available context. Token-budget directives in the prompt body are stale and contradict the deployment. | Remove all "be concise to save tokens" or "minimize output for cost" instructions from the body. If conciseness is a UX goal, frame it as a UX requirement, not a cost requirement. | High |
| 18 | **Model-routing recommendations in the prompt body** | Standing Environment forbids Sonnet/Haiku routing for the optimized prompt's execution. If the body recommends model routing for its own execution, it contradicts the deployment. | Strip routing recommendations. Move any harness-level model-routing logic to the chat run sheet's `<deployment_config>`, not the body. | High |
| 19 | **High-resolution image scaling assumptions** (4.6-era bounding-box conversion) | On Opus 4.7, pointing/bounding-box coordinates are 1:1 with image pixels. Scale-factor conversion logic carried over from 4.6 returns wrong coordinates. | Remove scale-factor conversion code/instructions. | High |
| 20 | **Cyber-content draft assumptions** | Opus 4.7 has new real-time cybersecurity safeguards. Drafts that assume the old refusal calibration may now refuse. | For legitimate security work, instruct the executor to surface refusals rather than retry around them, and route the user toward the Cyber Verification Program for production-grade access. | Medium |

**Severity legend.** `Blocking` = causes 400 errors / API failures. `High` = causes wrong outputs or silent quality loss. `Medium` = degrades quality. `Low` = style friction. Rows marked Blocking or High are always *material* gaps in Phase 6A QC; Medium/Low are minor unless they aggregate.

## Sources

Accessed 2026-05-20:

- [Migrating to Claude Opus 4.7 — Claude API Docs](https://docs.anthropic.com/en/docs/about-claude/models/migrating-to-claude-4) — every deprecation in rows 2–6 and 19; behavioral changes in rows 7–18 and 20.
- [Building with extended thinking — Claude API Docs](https://docs.anthropic.com/en/docs/build-with-claude/extended-thinking) — adaptive-thinking syntax and the manual-thinking 400 error.
- [Model configuration — Claude Code Docs](https://docs.claude.com/en/docs/claude-code/model-config) — `xhigh` ladder rung; `xhigh` as Opus 4.7 default effort.
- [Create custom subagents — Claude Code Docs](https://docs.claude.com/en/docs/claude-code/sub-agents) — subagent definition surface, `CLAUDE_CODE_SUBAGENT_MODEL` env var, parallel-dispatch shape (informs row 10).
- [Introducing Claude Opus 4.7](https://www.anthropic.com/news/claude-opus-4-7) — high-resolution image support (row 19), real-time cybersecurity safeguards (row 20).
- `SKILL.md` Meta-Rules 6, 7, 10, 11, 13, 14 — anchors for the style and orchestrator anti-pattern rows.
