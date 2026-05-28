# Orchestrated Multi-Agent Research — Complete Operational Guide

> **Calibration stamp:** Product facts in this reference (subagent constraints, environment-variable behavior, dispatch model) verified against authoritative Anthropic documentation on **2026-05-20**. Re-verify on every skill run via Phase 1.5.

This reference is the full operational manual for the `orchestrated-research` task type. It covers decomposition, subagent prompt construction, dependency management, output scope calibration, synthesis design, the remediation pattern, environment configuration, the orchestrator `CLAUDE.md` template (Section 10), and the failure mode catalog. Phase 4 of `SKILL.md` cites **Section 10** by number when producing the orchestrator deliverable — that section's heading and content are load-bearing; do not move them.

## Table of contents

1. [Overview — when orchestrated research applies](#1-overview--when-orchestrated-research-applies)
2. [Task decomposition](#2-task-decomposition)
3. [Subagent prompt construction](#3-subagent-prompt-construction)
4. [Context isolation](#4-context-isolation)
5. [Dependency management](#5-dependency-management)
6. [Output scope calibration](#6-output-scope-calibration)
7. [Synthesis design — the primary quality gate](#7-synthesis-design--the-primary-quality-gate)
8. [The remediation pattern](#8-the-remediation-pattern)
9. [Environment configuration](#9-environment-configuration)
10. [The orchestrator CLAUDE.md template](#10-the-orchestrator-claudemd-template)
11. [Failure mode catalog](#11-failure-mode-catalog)

---

## 1. Overview — when orchestrated research applies

The orchestrated-research pattern applies when:

- The task is large enough that a single linear pass would either run out of context, miss perspectives, or produce a thin synthesis from a single vantage.
- The work decomposes cleanly into independent (or partially independent) investigations whose results combine into a single deliverable.
- The deployment target is the Claude Code CLI (subagent dispatch is a CLI-shaped capability — see `platform-baseline.md`).

It does not apply when:

- The task is small enough that the orchestrator could do it alone in a single pass without context pressure.
- The work is genuinely sequential — each step depends entirely on the previous output, with no parallel branches.
- The deployment target is claude.ai chat, the API without your own dispatch harness, or Cowork (which has different multi-agent ergonomics).

When the Phase 0 Step 0C sniff (`inference-heuristics.md`) surfaces orchestrated-research signals, Phase 1's Widget Call 1 leads with it as the task-type option — the user confirms before Phase 2.

### What makes Opus 4.7 different here

Two behavioral facts shape every other section in this reference:

1. **Opus 4.7 spawns fewer subagents by default than 4.6.** The orchestrator's instinct is to do work itself rather than delegate. The orchestrator `CLAUDE.md` must therefore *explicitly* instruct subagent dispatch — not just authorize it.
2. **Subagents cannot spawn other subagents.** This is a hard runtime constraint. The dispatch graph is one level deep (orchestrator → subagents → synthesis), not arbitrarily nested.

## 2. Task decomposition

Decomposition is the work of identifying which subagent each gets to do. Good decompositions:

- **Maximize parallel independence.** Each subagent's work should be runnable without waiting on another subagent's output.
- **Minimize scope overlap.** Two subagents researching the same sub-domain return overlapping results; synthesis adds little. Carve scopes by sub-domain, source-type, or perspective — not by surface convenience.
- **Differentiate scope explicitly.** Two subagents with identical prompts but different "you are agent #2" framing will produce overlapping outputs. Each subagent's prompt must spell out what *it specifically* should cover that the others do not.
- **Respect synthesis bandwidth.** A 20-subagent fan-out that returns 20 long reports overwhelms synthesis. Cap subagent counts where synthesis can actually integrate the outputs — generally ≤ 6 for analytical work, more is possible for narrow-scope extraction tasks.

**Decomposition axes — pick the one that fits the task.**

| Axis | When it fits | Example |
|------|--------------|---------|
| **Sub-domain** | Topic has natural divisions (sectors, geographies, frameworks) | One subagent per industry vertical for a competitive landscape |
| **Source type** | The work needs source-diversity coverage | One subagent for primary sources, one for practitioner press, one for academic literature, one for dissenting press |
| **Perspective** | The work is multi-stakeholder | One subagent for incumbent / regulator / disruptor / customer vantages |
| **Time horizon** | The work has temporal layers | One subagent for historical baseline, one for current state, one for near-term forecast |
| **Sequential dependency** | Some work depends on earlier results | Wave 1: market sizing + competitive set in parallel → Wave 2: positioning informed by both |

Hybrid decompositions are common — a wave-1 / wave-2 sequence where each wave is itself parallel.

## 3. Subagent prompt construction

Each subagent gets its own prompt embedded in the orchestrator's `Agent` / `Task` dispatch call. The subagent's prompt is the only thing it sees besides its tool surface and the working directory.

**Mandatory elements of every subagent prompt.**

- **Identity and scope.** "You are the [X] subagent. Your scope is [precisely defined]. You are NOT responsible for [explicitly excluded areas]." The explicit exclusion matters: subagents with overlapping scopes produce overlapping results.
- **Output shape.** "Return a [length] [format] structured as: [headings or fields]. Do not include [excluded sections]." Without explicit output shape, subagents return wildly different shapes that synthesis cannot integrate.
- **Source / tool discipline.** "Cite every claim. For sources, use [tool]. For [specific data], use [tool]." Use direct tool instruction when the task obviously requires a specific tool — adaptive thinking decides the rest.
- **Independence requirement.** "Do not embed unverified numerical estimates from outside sources. Where you cite a number, verify it from a primary source within your scope or flag it as 'reported, not verified.'" This implements `SKILL.md` Meta-Rule 13 — never let unverified numbers propagate between agents.
- **Synthesis-readiness.** "Your output feeds a synthesis pass that combines your work with [N] other subagents. Make your output skim-able: lead with the key finding, then evidence."

**Avoid:**

- Reusing one boilerplate subagent prompt with `{topic}` swapped out. The shared scaffolding propagates the same blindspots across all subagents.
- Letting a subagent "decide what to cover" — the orchestrator's job is to decide that, the subagent's job is to execute.
- Embedding the orchestrator's hypothesis about the answer. Subagents are research agents, not validators of preconceived conclusions.

## 4. Context isolation

Each subagent runs in its own context window with a custom system prompt, specific tool access, and independent permissions. The context savings are real: a subagent that reads 200K tokens of source material returns its summary, and the orchestrator's context absorbs only the summary, not the source.

Constraints that follow from context isolation:

- The orchestrator cannot stream "live" context to a subagent mid-run. Everything the subagent needs goes into its initial prompt.
- Constants the orchestrator wants every subagent to know (a shared definition, a numeric baseline, a glossary) must be repeated in each subagent's prompt or pulled from a shared file each subagent reads at start.
- Subagent outputs return only the summary the subagent emits — not their working notes, not their tool-call traces. Design subagent prompts to emit the synthesis-ready summary, not exploratory thinking.

## 5. Dependency management

Three dependency patterns cover almost everything:

### Fully parallel

All subagents are independent. The orchestrator dispatches them in a single batch and waits for all results before synthesis. This is the default and the most productive shape — use it when the decomposition permits.

```
Orchestrator dispatch:
  ↓
  ├─ Subagent A (independent)
  ├─ Subagent B (independent)
  ├─ Subagent C (independent)
  └─ Subagent D (independent)
  ↓
  Synthesis
```

### Wave-based

Wave 2 subagents need results from Wave 1. The orchestrator dispatches Wave 1 in parallel, waits, then dispatches Wave 2 with Wave 1's results in their prompts.

```
Wave 1:                Wave 2:
  ├─ A (market size)   ├─ E (positioning informed by A + B + C)
  ├─ B (competitive)   └─ F (entry path informed by A + B + C + D)
  ├─ C (regulation)
  └─ D (incumbents)
  ↓
  (orchestrator collects Wave 1 results)
  ↓
  Wave 2 dispatch with Wave 1 results embedded in prompts
  ↓
  Synthesis
```

### Hybrid (shared constants + parallel)

A pre-dispatch step establishes shared constants (definitions, baselines, glossaries) the orchestrator embeds in every subsequent subagent prompt. The rest of the work runs fully parallel.

```
Shared-constants step (orchestrator-only or first subagent):
  ↓ produces definitions / baselines / glossaries
  ↓
  ├─ Subagent A (gets shared constants in prompt)
  ├─ Subagent B (gets shared constants in prompt)
  └─ Subagent C (gets shared constants in prompt)
  ↓
  Synthesis
```

**The shared-constants pattern is the antidote to Meta-Rule 13's "no unverified numerical estimates in subagent prompts."** Instead of the orchestrator inventing a number and feeding it to subagents (who treat it as fact), the shared constants are derived from primary sources first and then propagated explicitly. If a constant cannot be verified, the orchestrator instructs each subagent to verify independently — never both.

## 6. Output scope calibration

Each subagent needs to know how long its output should be and what shape it should take. Calibration rules:

- **Match the synthesis bandwidth.** If synthesis is a single chat-context pass that integrates 6 subagents, each subagent's output should fit in a fraction of that context. Long outputs (multi-page reports) from 6 subagents will not synthesize coherently; short outputs (one paragraph) miss the value of parallel research.
- **Specify the structure, not just the length.** "1,500 words" is weak. "A 1,500-word brief structured as: (1) key finding, (2) evidence, (3) gaps you couldn't close, (4) sources" is operable.
- **Demand citations.** Every factual claim a subagent emits must carry a citation the synthesis pass can verify. Without this, synthesis cannot tell which subagent claims are well-sourced and which are inferences the subagent shouldn't be making.
- **Allow refusal-shaped output.** "If you cannot close [topic] in your scope, return what you have and explicitly list what remains uncovered." Subagents that bluff their way to a complete-looking output undermine synthesis quality.

## 7. Synthesis design — the primary quality gate

**Synthesis is the most important section of the orchestrator.** Subagent outputs are inputs to synthesis; synthesis is what reaches the user. A multi-agent run with brilliant subagents and weak synthesis is worse than a single-agent run that did less but synthesized well.

**Synthesis is not "summarize the subagent results."** It is integration: spotting where subagents agree (and why), where they disagree (and what that means), where they all overlooked the same thing, and what the *combined* set of findings implies that no single subagent could have surfaced.

**Mandatory synthesis tasks.**

1. **Cross-subagent consistency check.** For every claim where multiple subagents touched the topic, are they consistent? If not, surface the disagreement explicitly — do not silently choose one.
2. **Coverage check.** For every sub-domain the orchestrator promised to cover, is it actually covered in the subagent outputs? If gaps remain, surface them as "deferred" rather than papering over.
3. **Source diversity check.** Synthesis verifies the corpus across subagents meets the source-type diversity bar (Phase 2's 3-of-5 categories applied across the full subagent corpus, not just per subagent).
4. **Bias / position check.** Has any subagent embedded a conclusion in its output? Synthesis strips conclusions and re-presents as topics-and-evidence, not positions.
5. **Synthesized findings.** Beyond the subagent outputs themselves, what does the combined work imply? The synthesized findings are the deliverable's value-add.
6. **Source-validation revisit pass on load-bearing sources.** For every URL cited as primary support for an executive-summary key finding, the synthesis agent re-fetches the source and verifies the subagent's attribution. Verdicts: `verified` / `partially verified — see note` / `not verified` / `unreachable`. A `not verified` load-bearing source either strikes the key finding or forces re-derivation from a different sourced support; `partially verified` revises the key finding to match the source. The source-validation log is working machinery — it stays in the chat run sheet, never in the written deliverable. Full protocol in [synthesis-deliverable.md](synthesis-deliverable.md) § Source-validation revisit protocol.
7. **Devil's-advocate dispatch and integration.** Always-on. After the synthesis agent drafts key findings and runs the source-validation pass, it dispatches a `devils-advocate` worker (or inlines an equivalent brief when the bundled agent is unavailable) in either **adversarial mode** (default for analytical / decision-support deliverables) or **confirmatory mode** (for purely descriptive inventory / fact-extract deliverables). The agent returns a structured findings pack; the synthesis agent integrates the verdicts into the executive summary under the verdict-ladder discipline that prevents verdict-mush (`unresolved` requires a sourced, credible counterweight — not the bare existence of a contrary view). Full protocol in [synthesis-deliverable.md](synthesis-deliverable.md) § Devil's-advocate dispatch brief.

**Synthesis is the orchestrator's job.** It is not delegated to a "synthesis subagent" — because then the synthesis is itself isolated from the orchestrator's context and judgment, and the synthesis output is just another subagent output that needs synthesizing. The orchestrator's context is precisely where synthesis belongs. The devil's-advocate dispatch in task 7 is the one sanctioned exception: it is a **dispatch** that returns evidence and tentative verdicts, not a delegation of synthesis. The synthesis agent still owns what lands in the executive summary, applies the verdict-ladder discipline, and writes the deliverable.

## 8. The remediation pattern

Sometimes Wave 1 subagent outputs reveal gaps the orchestrator didn't anticipate. The remediation pattern handles this without breaking the wave structure:

1. **Detect.** During synthesis, the orchestrator notices a gap (a sub-domain that surfaced as material but wasn't in the original decomposition).
2. **Scope.** The orchestrator defines a targeted subagent prompt to close exactly that gap — narrow scope, short output, no overlap with already-dispatched work.
3. **Dispatch.** A remediation-pass subagent fires (single, scoped) and returns.
4. **Re-synthesize.** Synthesis incorporates the remediation output and the deliverable is built.

**Cap remediation at one extension.** A second remediation pass is a signal that the original decomposition was wrong — at that point, restart Phase 2 of the skill rather than chaining remediations.

## 9. Environment configuration

Per `SKILL.md` § Standing Environment Assumptions: orchestrator and every subagent run `claude-opus-4-7` with adaptive thinking, no cost-driven routing. The orchestration-specific environment layered on top:

- **`CLAUDE_CODE_SUBAGENT_MODEL=claude-opus-4-7[1m]`** — every subagent runs on the 1M-context Opus 4.7 alias. The env var takes top precedence over per-invocation model and per-subagent frontmatter, so it is the right surface for "every subagent on Opus 4.7."
- **`ANTHROPIC_DEFAULT_OPUS_MODEL=claude-opus-4-7[1m]`** — for pinned third-party deployments (Bedrock, Vertex, Foundry), this ensures the `opus` alias resolves to 4.7 with 1M context.
- **Effort:** session default is `xhigh` on Opus 4.7 (Claude Code default). Either accept the default or set explicitly via `--effort xhigh` or `CLAUDE_CODE_EFFORT_LEVEL=xhigh`.
- **Adaptive thinking:** on by default on Claude Code; on the raw API, must be set explicitly (`thinking: {type: "adaptive"}`).
- **Permission mode:** for an orchestrated-research run that's read-only research, `auto` or `default` is appropriate; for a run that writes deliverables, ensure permission mode authorizes the writes.

These environment requirements live in the chat run sheet's `<deployment_config>` block, alongside the prompt — never inside the orchestrator `CLAUDE.md` body itself.

## 10. The orchestrator CLAUDE.md template

**This section is the load-bearing one — `SKILL.md` Phase 4 cites `orchestrated-research.md Section 10` by number when producing the orchestrator deliverable. Keep this heading and section number stable.**

The template below is the structure the orchestrator `CLAUDE.md` takes. The `prompt-template.md` orchestrator template references this section rather than restating it — they are two views of the same artifact.

### Structure

```markdown
# [Orchestrator name — e.g., "Diligence brief on [target]"]

## Mission

[A single sentence stating the deliverable. The orchestrator produces this, period. No restatement, no preamble.]

## Scope

[What is in scope and out of scope. Bullet list. Lead with in-scope, then out-of-scope. The out-of-scope list is as important as the in-scope list — it suppresses Phase 3-style "gap flags" on excluded sub-domains.]

## Subagent dispatch

Spawn the following subagents in parallel using the Agent / Task tool. Each subagent has its own scope, output shape, and source discipline. Do not collapse parallel work into a single sequential thread — Opus 4.7 under-delegates by default, so this instruction is explicit and load-bearing.

### Subagent A — [name]

**Scope:** [precisely defined]
**Not in scope:** [exclusions to prevent overlap with other subagents]
**Output:** [length + structure — e.g., "1500-word brief with headings: Key finding / Evidence / Gaps / Sources"]
**Source discipline:** [tools and source-type requirements]
**Verification:** [if numeric — must verify against primary sources; never adopt unverified estimates passed in]

### Subagent B — [name]

[Same shape as Subagent A.]

### Subagent C — [name]

[Same shape.]

### Subagent D — [name]

[Same shape.]

[Continue for additional subagents up to the practical synthesis bandwidth — usually ≤ 6 for analytical work.]

## Wave structure (if applicable)

[If the dispatch is wave-based, name Wave 1 and Wave 2 subagents and describe what Wave 2 receives from Wave 1. If fully parallel, omit this section.]

## Shared constants (if applicable)

[If the hybrid pattern applies, list the constants every subagent gets in its prompt and their primary source. If no shared constants, omit this section.]

## Synthesis

After all subagents return, run synthesis. Synthesis is the primary quality gate — do not delegate it to another subagent (the Wave 2 devil's-advocate dispatch below is a dispatch, not a delegation of synthesis itself). Synthesis tasks:

1. **Cross-subagent consistency.** Where subagents touched the same topic, are they consistent? Surface every disagreement explicitly.
2. **Coverage check.** For every sub-domain promised in Scope, is it covered in the returned outputs? List gaps.
3. **Source diversity check.** Across the full corpus, are at least three of (primary documents / academic / practitioner / mainstream / dissenting) represented? If not, name the missing categories.
4. **Bias check.** Strip any embedded conclusions from subagent outputs and re-present as topics-with-evidence.
5. **Synthesized findings.** What does the combined work imply that no single subagent could have surfaced? This is the deliverable's value-add.
6. **Source-validation revisit pass on load-bearing sources.** For every URL cited as primary support for an executive-summary key finding, re-fetch the source and verify the subagent's attribution. Record `verified` / `partially verified — see note` / `not verified` / `unreachable`. A `not verified` source either strikes the key finding or forces re-derivation; `partially verified` revises the finding to match the source. The source-validation log stays in the chat run sheet, never in the written deliverable. See `synthesis-deliverable.md` § Source-validation revisit protocol.
7. **Devil's-advocate integration.** After the Wave 2 dispatch below returns, apply the verdict-ladder discipline and integrate verdicts into the executive summary. `unresolved` only fires when adversarial evidence meets the credibility bar.

Length: [calibrated to the deliverable, typically 25–50% of the combined subagent output].

## Wave 2 — Devil's advocate (mandatory)

After synthesis tasks 1–6 complete and key findings are drafted, dispatch the `devils-advocate` worker (or inline an equivalent brief when the bundled agent is unavailable) in one of two modes:

- **Adversarial mode** — default. Use when the deliverable advances an implicit thesis (analytical, decision-support, evaluative, comparative, recommendation-shaped). The agent searches for the strongest sourced evidence each key finding's opposite is true and returns per-finding `confirmed` / `refuted by adversarial pass` / `unresolved — both views credible`.
- **Confirmatory mode** — use when the deliverable is purely descriptive (inventory, fact-extract, directory, roster). The agent audits included entries (`included correctly` / `included in error` / `boundary case`) and surfaces missed candidates.

Mode-sniff rule: if the deliverable advances a thesis the audience could plausibly disagree with → adversarial; if purely descriptive → confirmatory. Default to adversarial when on the boundary. Record the mode and one-line rationale in the dispatch brief and in the executive summary.

Dispatch brief carries: key-findings list with source-validated URLs; per-finding tentative conclusions; overall conclusion (adversarial mode) OR included entries with inclusion criteria (confirmatory mode); the verdict-ladder discipline verbatim; the topic-not-position discipline verbatim; the search budget.

The devil's-advocate findings pack is working machinery — it stays in the chat run sheet alongside the source-validation log. Only the integrated verdicts land in the deliverable's executive-summary block. See `synthesis-deliverable.md` § Devil's-advocate dispatch brief.

## Deliverable

Write the final deliverable to [path / filename]. The deliverable **leads with the executive summary** and then continues with the supporting body. Required structure:

- **Executive summary (first section, no exceptions).** Per the template in `synthesis-deliverable.md` § Executive summary template: framing paragraph, key findings (each with primary source URL + validation verdict + confidence), per-finding conclusions, overall conclusion, devil's-advocate pass with verdicts.
- Synthesized findings — the full integrative findings the executive summary distilled from.
- Supporting evidence by sub-domain — the per-subagent material organized around the key findings, with citations.
- Gaps and deferred questions — including any source-validation `revisit deferred` items, any `audit deferred` items from confirmatory mode, and any sub-domain coverage gaps surfaced in synthesis task 2.
- Sources — every URL cited anywhere in the deliverable, with consistent citation discipline across subagents.

**Every claim in the deliverable carries a URL.** Synthesis-agent integrative reasoning across multiple cited findings is explicit ("Combining findings 1 and 3, …") rather than implicit. Claims without a URL are either explicitly-framed integrative reasoning or deleted.

Do not include in the deliverable: the orchestration scaffolding above, the subagent prompts, the wave structure, the synthesis self-check, the source-validation log, the devil's-advocate findings pack. Those belong in the orchestrator's chat output, never in the written file. (This mirrors the Deliverable Contract partition in the parent skill. The Phase 6B pre-write strip check covers the new working-artifact types.)

## Remediation rule (if any)

If synthesis surfaces a material gap that the original decomposition did not anticipate, dispatch exactly one remediation subagent with a narrowly-scoped prompt. Cap at one remediation pass. If a second pass would help, document the unexplored thread in the deliverable's "Gaps and deferred questions" section rather than recursing further. The devil's-advocate Wave 2 dispatch is separate from this remediation rule — it always runs once and is not subject to the remediation cap.
```

### Authoring notes

- Every subagent section must restate "not in scope" — this is what differentiates subagents and prevents overlap.
- Synthesis is sized at 25–50% of combined subagent output because synthesis is a quality gate, not a summary.
- The deliverable partitioning (write file = deliverable body; orchestrator scaffolding + source-validation log + devil's-advocate findings pack = chat-side) mirrors the parent skill's Deliverable Contract.
- Cite-every-claim discipline propagates from orchestrator to subagents to deliverable — every claim in the deliverable body carries a URL.
- The executive summary is the deliverable's first section, no exceptions. Its template is in `synthesis-deliverable.md` and its structure (key findings → primary URLs + validation verdicts → per-finding conclusions → overall conclusion → devil's-advocate verdicts) is load-bearing for downstream reader sanity-checking.
- For Meta-Rule 13 compliance: every numeric value that the orchestrator passes to a subagent must be either (a) sourced and labeled as such, or (b) instructed to be verified by the subagent independently. Never pass an orchestrator's guess as fact. The synthesis agent's source-validation pass (task 6) is the downstream guardrail that catches unverified numbers that nevertheless made it into the deliverable's load-bearing claims.
- Devil's-advocate Wave 2 is **always-on**, but the mode adapts to the deliverable shape — adversarial for analytical deliverables, confirmatory for purely descriptive deliverables. Default to adversarial on the boundary.

### Section 10 — Worked example

Populated orchestrator for a competitive-landscape diligence brief — one of the most common orchestrated-research applications. Kept compact to demonstrate the template-to-instance mapping without overwhelming.

```markdown
# Diligence brief on Acme SaaS — competitive landscape

## Mission
Produce a 5–7 page diligence brief on Acme SaaS's competitive position in mid-market workforce-management software, suitable for a sponsor's investment committee.

## Scope
- **In:** direct competitors ($20M–$200M ARR, North America); customer-segment overlap; recent funding / M&A (18 months); pricing models
- **Out:** adjacent-market HCM suites; geographies outside North America

## Subagent dispatch (parallel via Agent / Task tool)

**Subagent A — Direct competitor inventory.** Scope: 8–12 mid-market workforce-management competitors. Not in scope: HCM suites; customer overlap. Output: 1500-word brief — top 3 threats + per-competitor profile + sources. Source discipline: Pitchbook, Crunchbase, company sites; cite every claim.

**Subagent B — Customer-segment overlap and switching costs.** Scope: map Acme's ICP against competitor ICPs; quantify switching costs. Not in scope: competitor profiles; pricing. Output: 1200-word brief — overlap matrix + switching-cost categories + sources. Source discipline: G2 / Capterra, customer interviews if available.

**Subagent C — Recent funding / M&A (18 mo).** Scope: funding rounds, acquisitions, partnerships involving Subagent A's competitor list. Not in scope: pre-2024-12 history. Output: 800-word brief — timeline + strategic implications + sources. Verification: every transaction confirmed against 2 sources.

**Subagent D — Pricing-model differences.** Scope: pricing-model survey across Subagent A's competitor list. Not in scope: discount strategies. Output: 900-word brief — pricing taxonomy + per-competitor placement + sources. Source discipline: public pricing pages, sales-call notes if available.

## Synthesis (orchestrator-only — do not delegate)

Seven mandatory tasks: (1) cross-subagent consistency (especially A↔C on overlapping competitors); (2) coverage check (all competitors named in A appear in B/C/D); (3) source-diversity check (3-of-5 categories across the full corpus); (4) bias check (strip embedded conclusions like "X is the dominant threat"); (5) synthesized findings (what the combined work implies about Acme's defensible position); (6) source-validation revisit on every URL cited as primary support for an executive-summary key finding; (7) integrate Wave 2 devil's-advocate verdicts under the verdict-ladder discipline.

Length: ~1500–2200 words (25–50% of combined subagent output).

## Wave 2 — Devil's advocate (mandatory)

Dispatch `devils-advocate` in **adversarial mode** (this deliverable advances a recommendation on Acme's competitive position — analytical, not descriptive). Brief carries the key-findings list with source-validated URLs and the per-finding tentative conclusions. Verdict-ladder discipline verbatim: `unresolved` requires sourced, credible counterweight.

## Deliverable

Write to `diligence-brief-acme.md`. Leads with the **executive summary** per `synthesis-deliverable.md` § Executive summary template (framing → key findings + primary URL + validation verdict + confidence → per-finding conclusions → overall conclusion → devil's-advocate pass). Then: synthesized findings; supporting evidence by sub-domain; gaps / deferred questions; sources.

Every claim in the body carries a URL.

Do not include in the deliverable: the subagent prompts above, the synthesis self-check, the source-validation log, the devil's-advocate findings pack (those belong in the orchestrator's chat output per the Deliverable Contract partition).
```

## 11. Failure mode catalog

The patterns below are the ways orchestrated research most commonly degrades. Each row pairs the failure with its diagnostic signal and its fix.

| Failure | Signal | Fix | Prevented by |
|---------|--------|-----|--------------|
| **Orchestrator does the work itself** | Single long output; no `Agent` / `Task` dispatch in the trace; subagents named but never spawned | Strengthen dispatch language. Spell out *when, why, and how many* subagents. Opus 4.7 under-delegates by default. | §3 (subagent prompt construction) + §10 (template dispatch instruction) |
| **Duplicate subagent scope** | Two subagents return overlapping findings on the same sub-domain | Tighten "not in scope" exclusions in each subagent prompt. Carve by sub-domain, source-type, or perspective — not by surface convenience. | §2 (task decomposition) |
| **Boilerplate-cloned subagent prompts** | Each subagent's output has the same blindspots | Diversify subagent prompts beyond the topic — vary scope, source discipline, output shape. | §3 (subagent prompt construction) |
| **Unverified numeric propagation** | A number in synthesis traces back to the orchestrator's guess via subagent prompts | Use the shared-constants pattern with sourced constants, OR instruct each subagent to verify independently. Never both. | §5 (shared-constants pattern) |
| **Synthesis as summary** | Synthesis just lists subagent outputs side by side; no cross-subagent integration | Restructure synthesis around the five mandatory tasks (consistency, coverage, source diversity, bias, synthesized findings). | §7 (synthesis — five mandatory tasks) |
| **Synthesis delegated to a subagent** | A "synthesis subagent" appears in the dispatch graph | Pull synthesis back into the orchestrator. Subagent context isolation defeats the purpose of synthesis. | §7 ("synthesis is the orchestrator's job") |
| **Deliverable contaminated with scaffolding** | The written file contains subagent prompts, wave structure, synthesis self-check | Apply the Deliverable Contract partition — orchestrator scaffolding belongs in chat, never in the file. | §10 (deliverable partition) |
| **Embedded conclusions** | Synthesis or deliverable states positions ("X is more important than Y") rather than topics-with-evidence | Phase 2 neutrality discipline. Reframe as "Address X and Y; present both sides where they disagree." | §7 (bias check) + Phase 2 neutrality (`landscape-research.md`) |
| **Coverage gap from over-pruning** | A sub-domain the user named at Phase 0 doesn't appear in any subagent output | Re-decompose. Verify that the subagent scope partition covers every promised sub-domain. | §2 (decomposition) |
| **Cascading remediation** | Remediation pass spawns another remediation pass | Cap at one remediation. The second is a signal the original decomposition was wrong — restart Phase 2 instead. | §8 (remediation cap) |
| **Subagent runs out of context mid-task** | Subagent returns truncated output | Use the 1M-context Opus alias (`claude-opus-4-7[1m]`) via `CLAUDE_CODE_SUBAGENT_MODEL`. If 1M is insufficient, the subagent scope is wrong — narrow it. | §9 (environment configuration) |
| **Subagent fabricates citations** | Citations in subagent output don't resolve | Subagent prompt must demand cite-every-claim with verifiable sources. Synthesis's source-diversity check should catch this. | §3 (source discipline) + §7 (source-diversity check) |
| **Synthesis without source attribution** | Deliverable body contains claims that float sourceless; reader cannot trace evidence | Synthesis-as-owner contract: every claim cites the URL that supports it. Integrative reasoning is explicit ("Combining findings 1 and 3, …"). | §7 (task 6) + [synthesis-deliverable.md](synthesis-deliverable.md) § Synthesis-as-owner contract |
| **Missing executive summary** | Deliverable opens with synthesized findings or jumps directly to evidence — no key-findings + URL + per-finding-conclusion + overall-conclusion block at the top | Executive summary is the deliverable's first section, no exceptions. Use the template in [synthesis-deliverable.md](synthesis-deliverable.md). | §10 (Deliverable section) + [synthesis-deliverable.md](synthesis-deliverable.md) § Executive summary template |
| **Source-validation pass skipped** | Executive-summary key findings carry the subagent's original confidence and attribution but no `verified` / `partially verified` / `not verified` / `unreachable` verdict | Synthesis task 6 is mandatory. Every load-bearing URL is re-fetched and the subagent's attribution is checked against the source. | §7 (task 6) + [synthesis-deliverable.md](synthesis-deliverable.md) § Source-validation revisit protocol |
| **Load-bearing source unverified but key finding kept at full confidence** | A `not verified` or `unreachable` verdict landed in the log but the key finding still rides into the overall conclusion as if it were verified | Strike the finding, re-derive from a different sourced support, or downgrade its confidence and surface the downgrade in the executive summary. Never silently retain. | §7 (task 6) + [synthesis-deliverable.md](synthesis-deliverable.md) § Source-validation revisit protocol |
| **Devil's-advocate dispatched but findings not integrated** | The Wave 2 dispatch ran but the executive summary's devil's-advocate block is empty, generic, or omitted — the verdicts never reached the deliverable | The synthesis agent integrates the verdicts under the verdict-ladder discipline before the executive summary is finalized. Empty block is itself a fail. | §7 (task 7) + [synthesis-deliverable.md](synthesis-deliverable.md) § Devil's-advocate dispatch brief |
| **Verdict-mush** | Every key finding flipped to `unresolved — both views credible`; the overall conclusion hedges into uselessness | Apply the verdict-ladder discipline: `unresolved` requires sourced, credible counterweight that survives a Tier-2+ citation in the source-trust hierarchy. Default to `confirmed` when nothing meets the bar. | §7 (task 7) + [synthesis-deliverable.md](synthesis-deliverable.md) § Verdict-ladder discipline |
| **Mode mismatch — adversarial on a descriptive deliverable** | Devil's advocate ran in adversarial mode on a pure inventory / fact-extract deliverable and fabricated thesis-style disagreements where none honestly exist | Apply the mode sniff: descriptive deliverables (inventory, fact-extract, directory, roster) run in confirmatory mode. Adversarial mode is for thesis-advancing deliverables. | §10 (Wave 2 section) + [synthesis-deliverable.md](synthesis-deliverable.md) § Mode sniff |
| **Conclusion not traceable to a key finding** | The overall conclusion contains a claim that does not derive from any per-finding conclusion in the executive summary | The overall conclusion is constrained to claims that derive from the per-finding conclusions. If a claim has no upstream key finding, either delete it from the conclusion or surface a new key finding that supports it. | §10 (Deliverable section) + [synthesis-deliverable.md](synthesis-deliverable.md) § Executive summary template |

## Sources

Accessed 2026-05-20:

- [Create custom subagents — Claude Code Docs](https://code.claude.com/docs/en/sub-agents) — subagent definition surface, frontmatter fields, context isolation, the constraint that subagents cannot spawn other subagents.
- [Model configuration — Claude Code Docs](https://code.claude.com/docs/en/model-config) — `CLAUDE_CODE_SUBAGENT_MODEL` env var precedence, 1M context routing via `[1m]` suffix, default effort `xhigh` on Opus 4.7.
- [Migrating to Claude Opus 4.7 — Claude API Docs](https://platform.claude.com/docs/en/about-claude/models/migration-guide) — "Fewer subagents spawned by default" behavioral change (informs Section 1 and Section 11).
- `SKILL.md` Meta-Rule 13 — anchor for the unverified-numeric-estimate rule and the orchestrator-dispatch instruction.
