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

**Synthesis is the orchestrator's job.** It is not delegated to a "synthesis subagent" — because then the synthesis is itself isolated from the orchestrator's context and judgment, and the synthesis output is just another subagent output that needs synthesizing. The orchestrator's context is precisely where synthesis belongs.

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

After all subagents return, run synthesis. Synthesis is the primary quality gate — do not delegate it to another subagent. Synthesis tasks:

1. **Cross-subagent consistency.** Where subagents touched the same topic, are they consistent? Surface every disagreement explicitly.
2. **Coverage check.** For every sub-domain promised in Scope, is it covered in the returned outputs? List gaps.
3. **Source diversity check.** Across the full corpus, are at least three of (primary documents / academic / practitioner / mainstream / dissenting) represented? If not, name the missing categories.
4. **Bias check.** Strip any embedded conclusions from subagent outputs and re-present as topics-with-evidence.
5. **Synthesized findings.** What does the combined work imply that no single subagent could have surfaced? This is the deliverable's value-add.

Length: [calibrated to the deliverable, typically 25–50% of the combined subagent output].

## Deliverable

Write the final deliverable to [path / filename]. The deliverable contains:

- [Section 1 of the deliverable — typically a 1-page executive summary]
- [Section 2 — synthesized findings]
- [Section 3 — supporting evidence by sub-domain]
- [Section 4 — gaps and deferred questions]
- [Section 5 — sources, with citation discipline matched across subagents]

Do not include in the deliverable: the orchestration scaffolding above, the subagent prompts, the wave structure, the synthesis self-check. Those belong in the orchestrator's chat output, never in the written file. (This mirrors the Deliverable Contract partition in the parent skill.)

## Remediation rule (if any)

If synthesis surfaces a material gap that the original decomposition did not anticipate, dispatch exactly one remediation subagent with a narrowly-scoped prompt. Cap at one remediation pass. If a second pass would help, document the unexplored thread in the deliverable's "Gaps and deferred questions" section rather than recursing further.
```

### Authoring notes

- Every subagent section must restate "not in scope" — this is what differentiates subagents and prevents overlap.
- Synthesis is sized at 25–50% of combined subagent output because synthesis is a quality gate, not a summary.
- The deliverable partitioning (write file = deliverable body; orchestrator scaffolding = chat-side) mirrors the parent skill's Deliverable Contract.
- Cite-every-claim discipline propagates from orchestrator to subagents to deliverable.
- For Meta-Rule 13 compliance: every numeric value that the orchestrator passes to a subagent must be either (a) sourced and labeled as such, or (b) instructed to be verified by the subagent independently. Never pass an orchestrator's guess as fact.

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

Five mandatory tasks: (1) cross-subagent consistency (especially A↔C on overlapping competitors); (2) coverage check (all competitors named in A appear in B/C/D); (3) source-diversity check (3-of-5 categories across the full corpus); (4) bias check (strip embedded conclusions like "X is the dominant threat"); (5) synthesized findings (what the combined work implies about Acme's defensible position).

Length: ~1500–2200 words (25–50% of combined subagent output).

## Deliverable

Write to `diligence-brief-acme.md`. Sections: executive summary, synthesized findings, supporting evidence by sub-domain, gaps / deferred questions, sources.

Do not include in the deliverable: the subagent prompts above, the synthesis self-check (those belong in the orchestrator's chat output per the Deliverable Contract partition).
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

## Sources

Accessed 2026-05-20:

- [Create custom subagents — Claude Code Docs](https://code.claude.com/docs/en/sub-agents) — subagent definition surface, frontmatter fields, context isolation, the constraint that subagents cannot spawn other subagents.
- [Model configuration — Claude Code Docs](https://code.claude.com/docs/en/model-config) — `CLAUDE_CODE_SUBAGENT_MODEL` env var precedence, 1M context routing via `[1m]` suffix, default effort `xhigh` on Opus 4.7.
- [Migrating to Claude Opus 4.7 — Claude API Docs](https://platform.claude.com/docs/en/about-claude/models/migration-guide) — "Fewer subagents spawned by default" behavioral change (informs Section 1 and Section 11).
- `SKILL.md` Meta-Rule 13 — anchor for the unverified-numeric-estimate rule and the orchestrator-dispatch instruction.
