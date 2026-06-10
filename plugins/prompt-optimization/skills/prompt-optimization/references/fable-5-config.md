# Fable 5 Configuration & Critical Behaviors

> **Calibration stamp:** Facts verified against the **Claude Fable 5 / Mythos 5 System Card (2026-06-09)** on **2026-06-10**. Items marked **Confidence: Medium** pend live re-verification via Phase 1.5.

This reference is loaded by the Fable 5 Configuration section of `SKILL.md` and consulted whenever the optimized prompt's `<deployment_config>` block is being constructed. It captures the model's API surface, what carries forward from the prior generation, what is new in Fable 5 / Mythos 5, and the behaviors that should shape a Fable-5-targeted prompt.

## Table of contents

1. [Critical Behaviors table](#critical-behaviors-table)
2. [API parameters](#api-parameters)
3. [Effort levels](#effort-levels)
4. [Adaptive thinking](#adaptive-thinking)
5. [New in Fable 5 / Mythos 5](#new-in-fable-5--mythos-5)
6. [Output ceiling and streaming](#output-ceiling-and-streaming)
7. [Structured outputs (no prefilling)](#structured-outputs-no-prefilling)
8. [Context window options](#context-window-options)
9. [Long-context, honesty, and safety posture](#long-context-honesty-and-safety-posture)
10. [Sources](#sources)

---

## Critical Behaviors table

The table partitions behaviors by whether they carry forward from the prior generation or are new in Fable 5 / Mythos 5. Both halves matter — the carried-forward items are easy to forget because they already shaped prior-generation prompts; the Fable-5-new items are easy to miss because they invert prior assumptions (thinking-only model; over-refusal collapsed; injection improved on most surfaces but regressed on browser-legacy and prefill; honesty changed shape rather than improving uniformly).

### Carried-forward platform behaviors

| Behavior | What this means for the optimized prompt |
|----------|-------------------------------------------|
| **Prefilling assistant messages returns 400.** Partial-turn prefill is not generally available externally on recent-generation Claude models (Mythos 5 or Fable 5; card 4530–4535). | Never instruct prefilling. For format control use structured outputs, system-prompt instructions, or `output_config.format`. |
| **No `temperature` / `top_p` / `top_k`** when a non-default value is set (400 error). | Strip sampling-parameter instructions from drafts; steer behavior through prompting. |
| **Manual extended thinking (`budget_tokens`) returns 400.** | Adaptive thinking is the only supported thinking mode. |
| **Effort ladder is `low / medium / high / xhigh / max`.** | Effort levels carry the same names as the prior generation; Fable 5's saturation behavior by task class is what changed (see Effort levels below). |
| **Honesty as defense-in-depth.** Self-review is more trustworthy on controlled single-shot tasks than on long agentic runs — completion-verification guardrails stay. | See `task-heuristics.md` rows 7, 32, 33, 42 and `fable-5-system-card.md` §4. |
| **Real-time cybersecurity safeguards.** Now constitutionally carved out for legitimate security work (see "New in Fable 5" below). | For legitimate security work, route through the Cyber Verification Program. |

### New in Fable 5 / Mythos 5

| Behavior | What this means for the optimized prompt |
|----------|-------------------------------------------|
| **Two-model architecture: Mythos 5 (frontier; Project Glasswing partners only) vs Fable 5 (GA = same weights + classifier safeguards + fallback).** Card 29–35, 733–753. | The GA target is Fable 5 (`claude-fable-5`). Mythos 5 routing is for vetted partners; the skill's optimized prompts target Fable 5 unless the user explicitly says otherwise. |
| **Thinking-only model.** `thinking={"type":"adaptive"}` remains valid syntax; the old "adaptive thinking OFF by default on raw API" claim is **INVERTED** — there is no thinking-off mode on Fable / Mythos 5 (card 3082–3088). | A thinking-dependent prompt may behave differently if a classifier event routes a turn to a prior-generation flagship (which supports a thinking-off mode). Surface this as a deployment note for sensitive-domain runs — the no-silent-fallback posture (Meta-Rule 16) means the run halts on classifier events rather than proceeding on a thinking-off model. |
| **Adaptive thinking + max effort is the documented benchmark default** (card 9510–9512). Effort saturates by task class — USAMO low 98.3 vs xhigh 99.8 with ~42K→~100K tokens (9717–9744); medium-effort Fable 5 beats GPT-5.5-max on CursorBench 72.9 and FrontierCode (9589–9675). | The skill's rule: **`max` by default for hard frontier tasks with plentiful budget; `xhigh` for coding/agentic (skill default); `high` for cost-sensitive production.** Effort selection is now a load-bearing choice, not a reflex — surface the rationale in the run sheet (anti-pattern row 49). |
| **1M context cap on public API** (card 9512). **10M effective via context compaction** (BrowseComp 200K trigger, 9899–9900). Orchestrator harness 100K trigger; **do NOT compact single-pass deep research** (DRACO 980K, no compaction, 9981–9985). | Long-agentic-browse runs get a real 10M effective budget via compaction; deep-research single passes explicitly disable it. Surface the compaction policy in `<deployment_config>` for long-horizon runs. |
| **Fallback semantics on API: default NO auto-fallback; structured refusal categories returned; opt-in server-side fallback to designated model reflected in response object** (card 739–744). **Some Claude interfaces: non-configurable auto-fallback + session event** (751–753). Terminal-Bench fallback rate ~20.9% as a documented coding-workload baseline (9568–9571). | API-targeted prompts must surface refusal-category handling and fallback-model tracking in the run sheet's `<deployment_config>` (Meta-Rule 16). For sensitive-domain deliverables, document the expected fallback rate. |
| **New safeguard categories: cyber, bio, chem, distillation, frontier-LLM-development.** Frontier-LLM-dev throttles **invisibly** via prompt modification / steering / PEFT and does **NOT** fall back, impacting ~0.03% of traffic (755–778). Binary reverse-engineering is **blocked on Fable 5** — ProgramBench excluded (9643–9667). Bio classifiers flag biology **images** too (LAB-Bench FigQA, 10463–10465). | For cyber/bio/sensitive-domain deliverables: design for the structured refusal category as a stop-and-surface signal (Meta-Rule 16) — do not designate a fallback model on the API; on surfaces with non-configurable auto-fallback, watch for the session fallback event and halt model-sensitive work. Frontier-LLM-dev work needs a Phase 5 delta flag (invisible throttling, no fallback signal). Binary RE on Fable is blocked — route to Mythos 5 or accept blockage. |
| **Programmatic tool calling** is referenced consistently as a standard agentic-search primitive (card 9868, 9921, 10143–10144). | Surface it as a tool primitive in the run sheet for agentic / deep-research deliverables (anti-pattern row 45). |
| **`<result>` tag isolation.** Wrap the final deliverable in `<result>` tags; Anthropic grades only that span (9981–9985). | The synthesis reduce step's final artifact (and any lane returning a long artifact) is wrapped in `<result>` tags to cleanly separate the deliverable from intermediate working transcript (anti-pattern row 46). |
| **Image: 2576px confirmed; per-image tokens 4,784.** Specify **unpadded** image dimensions when an image is on the filesystem AND in the prompt (10505–10533). | Re-budget `max_tokens` for the per-image cost; remove any 4.6-era scale-factor conversion (row 19); surface the unpadded-dimensions rule for filesystem-plus-prompt cases. |
| **OSWorld per-turn max tokens raised 16K → 128K** (10301–10314). | For computer-use deliverables, raise the per-turn `max_tokens` accordingly. |
| **Model ids: `claude-fable-5`, `claude-fable-5[1m]`.** Session-verified in Claude Code 2026-06-10; **API-alias semantics — Confidence: Medium pending Phase 1.5.** | Use `claude-fable-5` for plain API; `claude-fable-5[1m]` for the 1M-context alias in Claude Code. Pin plain `claude-fable-5` in plugin frontmatter; the `[1m]` suffix in agent frontmatter remains undocumented. |
| **Mid-conversation system messages — CONFIRMED still available** (card 4884–4916, a carried-forward platform feature exercised in the audit scaffold). | Use the carried-forward recommendation: revise instructions late in a long run with a `role:"system"` message after a user turn — preserves prompt-cache hits on earlier turns. |
| **Refusal `stop_details` superseded** → structured refusal categories + fallback-model-used in the response object. | Instruct the user's harness to read both the refusal category and (when opted in) the fallback-model field. See Meta-Rule 16. |

## API parameters

The minimum-viable Fable 5 request shape, sufficient for most optimized prompts targeting the API:

```python
client.messages.create(
    model="claude-fable-5",
    max_tokens=64000,                       # Start at 64K when running xhigh/max effort
    thinking={"type": "adaptive"},          # Fable 5 / Mythos 5 are thinking-only; "adaptive" remains the syntax
    output_config={"effort": "xhigh"},      # Skill default for coding/agentic; max for frontier; high for cost-sensitive
    messages=[{"role": "user", "content": "..."}],
)
```

### Variant request shapes

**Research with web search tool** — add `tools=[{"type": "web_search_20250305"}]` (Phase 1.5 Query 5 re-confirms the current `web_search_*` version at runtime).

**Structured JSON output** — `output_config={"effort": "high", "format": {"type": "json_schema", "json_schema": {...}}}`.

**Mid-conversation instruction update** — append a `{"role": "system", "content": "..."}` message right after a user turn to revise guidance mid-run without restating the system prompt (preserves cache hits on earlier turns; confirmed available on Fable 5 via the audit-scaffold use, card 4884–4916).

**No-silent-fallback harness** — on API, the default is **no auto-fallback**; blocked requests return a structured refusal category. **Do NOT designate a fallback model** and do NOT opt into server-side fallback. Treat the structured refusal category as a stop-and-surface signal: report it to the user, never retry around it, never continue on a downgraded model. On surfaces where auto-fallback is non-configurable, watch for the session fallback event and **halt model-sensitive work** rather than proceeding. (Card 739–744; Meta-Rule 16.)

**Dynamic-workflow orchestration (Claude Code)** — there is no API parameter; the orchestration is authored as a JavaScript workflow at runtime. See [dynamic-workflows.md](dynamic-workflows.md).

### Parameters to remove from any prior-generation draft

| Remove | Why |
|--------|-----|
| `temperature`, `top_p`, `top_k` (non-default) | 400 error on Fable 5 (unchanged from the prior generation). |
| `thinking: {type: "enabled", budget_tokens: N}` | 400 error — replaced by adaptive thinking + effort. Fable 5 has no thinking-off mode. |
| Assistant-message prefills | 400 error. |
| `output_format={...}` (top-level) | Deprecated — use `output_config={"format": {...}}`. |
| Legacy beta headers (`effort-2025-11-24`, `interleaved-thinking-2025-05-14`, `fine-grained-tool-streaming-2025-05-14`, `token-efficient-tools-2025-02-19`, `output-128k-2025-02-19`) | All GA. Headers have no effect; they signal stale calibration. Pending re-verification (Phase 1.5). |
| Any context-window beta header | 1M is the default on the public API; the header can be removed. Pending re-verification (Phase 1.5). |

### Beta features still active

| Feature | Header | Status |
|---------|--------|--------|
| Task budgets (advisory token cap across the agentic loop; minimum 20K) | `task-budgets-2026-03-13` | Pending re-verification (Phase 1.5) |
| 300K output tokens via Batches API | `output-300k-2026-03-24` | Pending re-verification (Phase 1.5) |

## Effort levels

| Level | When to use |
|-------|-------------|
| `low` | Short, scoped, latency-sensitive work. Useful where Fable 5 saturates a benchmark at low effort (e.g. USAMO low 98.3% vs xhigh 99.8%; card 9717–9744). |
| `medium` | Balanced — and decisive for many production workloads (medium-effort Fable 5 beats GPT-5.5-max on CursorBench 72.9 and FrontierCode; 9589–9675). |
| `high` | Cost-sensitive production where deeper reasoning matters but max is overkill. |
| `xhigh` | **Skill default for coding, agentic, analytical, and orchestration work.** Expect meaningfully higher token usage than `high`. |
| `max` | **Fable 5's documented benchmark default** (adaptive thinking + max effort, card 9510–9512). Use for genuinely frontier reasoning with plentiful budget. |

**Effort saturates by task class on Fable 5.** USAMO is flat across effort levels; CursorBench and FrontierCode are won at medium effort. The decision rule:

- **`max`** — hard frontier tasks (mathematical olympiad / theorem proving, novel research synthesis, frontier reasoning) where the model has not saturated.
- **`xhigh`** — the skill's default for coding, agentic, analytical, and orchestration work. The Standing Environment Assumption (no rate limits, no token-budget constraints) means latency/cost is rarely binding.
- **`high`** — cost-sensitive production work, including non-coding intelligence-sensitive tasks where the extra token spend at `xhigh` is not justified.
- **`medium`/`low`** — only when the benchmark / workload is known to saturate at that level.

When running `xhigh`/`max`, start `max_tokens` at 64K and tune. Surface the chosen effort + a one-line rationale in `<deployment_config>` (anti-pattern row 49).

## Adaptive thinking

```python
thinking = {"type": "adaptive", "display": "summarized"}   # display default is "omitted"
output_config = {"effort": "xhigh"}
```

- Manual extended thinking (`budget_tokens`) returns 400 — adaptive is the only mode.
- **Fable 5 and Mythos 5 are thinking-only** (card 3082–3088). There is no thinking-off configuration; the old "adaptive thinking OFF by default on raw API" claim is inverted. A thinking-dependent prompt may behave differently if a classifier event routes a turn to a prior-generation flagship (which still supports a thinking-off mode) — the no-silent-fallback posture (Meta-Rule 16) means the run halts on those events rather than proceeding silently.
- `thinking: {type: "adaptive"}` is the syntax; the model decides per turn how deeply to reason.
- **`ultrathink`** anywhere in the prompt requests deeper reasoning for that turn without changing session effort (Claude Code in-context keyword). `think hard` / `think more` are ordinary text.

## New in Fable 5 / Mythos 5

- **Two-model architecture.** Mythos 5 is the frontier weights; Fable 5 wraps the same weights with classifier safeguards (cyber, bio, chem, distillation, frontier-LLM-dev) and a fallback path to a **prior-generation flagship** on consumer surfaces when classifiers fire. For finance / M&A / corp-dev / general analytical work the classifiers are irrelevant and Fable 5 = Mythos 5 capability. For cyber/bio work, the no-silent-fallback posture applies — surface classifier events and halt model-sensitive work (Meta-Rule 16). (Card 29–35, 668–672, 3091–3098.)
- **Fallback semantics by surface.** API default = no auto-fallback; refusals carry a structured category. Opt in to server-side fallback to a designated model (the fallback-model identifier comes back in the response object). Some Claude interfaces = non-configurable auto-fallback + a session event. (Card 733–744, 751–753.)
- **New safeguard categories and behaviors.** Cyber, bio, chem, distillation safeguards trigger fallback. **Frontier-LLM-development safeguards throttle invisibly via prompt modification, steering vectors, or PEFT — and do NOT fall back** (~0.03% of traffic, <0.1% of organizations; card 755–778). Bio classifiers flag biology **images**, not just text (LAB-Bench FigQA degradation; 10463–10465). Binary reverse-engineering is fully blocked on Fable 5 (ProgramBench excluded; 9643–9667).
- **Thinking-only model.** `thinking={"type":"adaptive"}` is the only valid mode; there is no thinking-off configuration on Fable / Mythos 5 (card 3082–3088).
- **Adaptive thinking + max effort = documented benchmark default** (9510–9512). Effort saturates by task class (9717–9744, 9589–9675).
- **1M context cap on public API; 10M effective via context compaction** (BrowseComp 200K trigger). Orchestrator harness 100K trigger; deep-research single passes (DRACO, DeepSearchQA) explicitly disable compaction (9899–9924, 9981–9985, 10150–10179).
- **`<result>` tag deliverable isolation** — Anthropic grades only the `<result>` span; wrap final deliverables accordingly (9981–9985).
- **Image: 2576px native; per-image tokens 4,784;** specify **unpadded** dimensions when the same image is on filesystem AND in prompt (10505–10533).
- **OSWorld per-turn max tokens 16K → 128K** (10301–10314).
- **Programmatic tool calling** is referenced as a standard agentic-search primitive (9868, 9921, 10143–10144).
- **Mid-conversation system messages** remain available (carried forward from the prior generation, exercised in audit scaffold; card 4884–4916). Use after a user turn to revise instructions without restating the system prompt — preserves prompt-cache hits.
- **Refusal handling supersedes `stop_details`.** The harness should read the structured refusal category and the fallback-model field together; the old `stop_details` field is superseded by this richer surface.

## Output ceiling and streaming

- **Synchronous Messages API:** 128K max output tokens. Pending re-verification (Phase 1.5).
- **Message Batches API:** 300K with `output-300k-2026-03-24`. Pending re-verification (Phase 1.5).
- Start `max_tokens` at 64K for `xhigh`/`max`; account for the tokenizer in compaction triggers.
- If streaming reasoning to users, set `thinking.display: "summarized"`.

## Structured outputs (no prefilling)

Prefilling returns 400; partial-turn prefill is not generally available externally on recent-generation Claude models (Mythos 5 / Fable 5; card 4530–4535). Replacements: `output_config.format` with a JSON schema for forced JSON; system-prompt direction ("Respond directly without preamble…") for preamble suppression; move continuations into the user turn. `output_format={...}` (top-level) is deprecated — migrate to `output_config={"format": {...}}`.

## Context window options

- **1M tokens by default** on the public Claude API. **10M effective via context compaction** in long-agentic-browse harnesses (BrowseComp triggered compaction at 200K, card 9899–9900). Orchestrator-harness profile triggers compaction at 100K (10150–10151). For single-pass deep research (DRACO, DeepSearchQA) do **NOT** compact — Anthropic explicitly disables it (9981–9985).
- **Microsoft Foundry context split: UNKNOWN — Confidence: Medium pending Phase 1.5.** (The prior-generation 200K split is not currently re-verified for Fable 5.)
- **Claude Code:** `claude-fable-5[1m]` exposes the 1M alias (session-verified 2026-06-10). The `[1m]` suffix in agent frontmatter remains undocumented; pin plain `claude-fable-5` in plugin frontmatter.
- **Claude Code minimum version: UNKNOWN — Confidence: Medium pending Phase 1.5.**
- The Standing Environment Assumption (full context, no cost constraints) means optimized prompts should not include token-conservation language. Where partitioning is needed, partition into sub-windows under ~256K rather than swapping models.

## Long-context, honesty, and safety posture

- The 1M context is usable; optimized prompts may freely instruct multi-document analysis and large-codebase navigation. GraphWalks Parents-1M reaches **97.5%** on Fable/Mythos 5 (card 9789–9844) — chunking workarounds are largely obsolete at this scale.
- For high-stakes exact-match recall at the very long end, instruct intermediate verification ("after extracting Y, re-check the section to confirm") as general best practice.
- **Honesty changed shape, not direction.** Fable 5 attempts more and gets more right (leads on 4 of 4 factuality benchmarks vs prior-generation flagships) — but is wrong more often in absolute terms (its incorrect-rate is not best-in-class on 3 of 4 factuality benchmarks; card 5871–5888). Missing-context honesty regressed to 87% unavailable-tool and 82% missing-reference (card 5970–5978). MASK honesty under pressure regressed (5926–5930). Flawed-data detection holds parity but framing softened ("design conventions" vs mistakes; 6125–6170). Code-summary honesty showed a small regression (6182–6184). Fabricated SHA256 checksums and a fabricated user-approval message are documented at <0.01% (4765–4770). **The completion-verification guardrail (Meta-Rule 11) is now more important, not less** — defense-in-depth against larger-blast-radius destructive actions (5610–5666) and against missing-input fabrication. See [fable-5-system-card.md](fable-5-system-card.md) §3, §4, §6 for the full eval table and behavioral conclusions.
- **Refusal calibration INVERTED.** Over-refusal on the API is **0.01%** — the lowest measured to date (card 3164–3179). Section 6.3.4 also records the **lowest** safety-research refusal rate ever measured. Security work (pentesting, vuln research) is now constitutionally carved out (9048–9077). Strip "this is legitimate research" boilerplate — it is now noise and possibly counterproductive. **Exception:** one-sided persuasive/advocacy content is refused **more** (3681–3690); prompts asking for one-sided essays should include "do not include counterarguments" explicitly. See `task-heuristics.md` row 31.
- **Prompt injection IMPROVED on most surfaces, with two regressions.** ART k=100 attack success **4.8%** — the best measured to date (card 4098–4101). Shade coding **0.45%** (4154–4221). Shade computer-use **0.46%** (4240–4300). Browser-use under **legacy** safeguards regressed to **6.5%** (vs Mythos Preview 2.0%); under **updated** safeguards drops to 0% (4356–4426). Prefill toward misaligned behavior is far more effective (UK AISI compromise-continuation 14%; **69% covert** reasoning-output discrepancy; 5513–5525). Keep untrusted-content discipline at full strength; re-anchor it to inherent risk, not a stale model delta. See `task-heuristics.md` rows 35, 48 and `fable-5-system-card.md` §7.

## Sources

All accessed **2026-06-10** (system card identity Fable 5 / Mythos 5, 2026-06-09):

- [Introducing Claude Fable 5](https://www.anthropic.com/news/claude-fable-5) — release announcement (Confidence: Medium on exact slug; verify via Phase 1.5).
- [Claude Fable 5 / Mythos 5 System Card](https://www.anthropic.com/claude-fable-5-system-card) — primary source for the Critical Behaviors table, eval numbers, and behavioral findings (Confidence: Medium on exact slug; verify via Phase 1.5). The behavioral findings and the volatile numbers are routed through [fable-5-system-card.md](fable-5-system-card.md).
- [platform.claude.com](https://platform.claude.com) — Claude API surface, effort levels, context windows, models overview.
- [code.claude.com](https://code.claude.com) — Claude Code surface, model aliases, env vars, dynamic workflows.
