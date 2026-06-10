# Changelog

All notable changes to this project are documented here. The format is based on
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres
to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.1.0] — 2026-06-10

Remove all prior-flagship references from functional text and history; delete the prior-flagship fallback-floor reference file; adopt the **no-silent-fallback posture** across the skill. Meta-Rule 16 and anti-pattern row 36 are rewritten to prevent-and-surface (treat the structured refusal category as a stop-and-surface signal; do not designate a fallback model on the API; on surfaces with non-configurable auto-fallback, watch for the session fallback event and halt model-sensitive work). Anti-pattern row 20 cyber guidance now surfaces-and-halts rather than expecting silent fallback. No phase, anchor, section-number, or Deliverable-Contract changes.

### Changed
- Meta-Rule 16 rewritten end-to-end: no opt-in to server-side fallback; do not designate a fallback model; treat the structured refusal category as a stop-and-surface signal; on non-configurable auto-fallback surfaces, watch for the session fallback event, surface it immediately, and halt model-sensitive work rather than continuing on a downgraded model.
- `task-heuristics.md` anti-pattern row 36 rewritten as "Accepts silent fallback" — deliverable opts into server-side fallback, designates a fallback model, or proceeds after a classifier event without surfacing it. The fix is the prevent-and-surface posture above.
- `task-heuristics.md` row 20 cyber guidance rewritten: sustained cyber agentic work reliably triggers the classifiers (~27-turn average to trigger) — the optimized prompt surfaces the event and halts, rather than continuing on a downgraded model; binary reverse-engineering is blocked outright.
- All comparison tables and per-task guidance across the references (`fable-5-system-card.md`, `task-heuristics.md`, `fable-5-config.md`, `best-practices.md`, `inference-heuristics.md`, `platform-baseline.md`, `prompt-template.md`, `dynamic-workflows.md`, `calibration-protocol.md`) absolutized — eval and benchmark numbers are stated as Fable 5 absolutes with card line references; where a generational contrast is load-bearing, the prior flagship is referenced neutrally.
- Plugin and root metadata (`plugin.json`, `README.md`, plugin `README.md`) adopt the no-silent-fallback framing in their descriptions. `plugin.json` keyword `fallback-aware` replaced by `no-silent-fallback`.

### Removed
- The prior-flagship fallback-floor reference file is deleted. The no-silent-fallback posture (Meta-Rule 16) supersedes it: optimized prompts do not load a fallback-behavior floor because they do not silently accept fallback.

### Migration
No phase or anchor changes. Existing optimized prompts produced under 2.0.0 remain valid; re-running the skill surfaces the row 36 rewrite, the row 20 cyber-halt rewrite, and the Meta-Rule 16 posture change. API harnesses that had opted into server-side fallback should remove the designated fallback model and update refusal handling to surface the structured refusal category (no retry around it; no continuation on a downgraded model).

## [2.0.0] — 2026-06-10

Migrate the skill from the prior-generation flagship to **Claude Fable 5** (with Mythos 5 as the frontier sibling), recalibrated against the Claude Fable 5 / Mythos 5 System Card (2026-06-09). The phase machine, Deliverable Contract, Widget protocol, Standing Environment Assumption structure, and load-bearing anchors (`dynamic-workflows.md` Section 10; `SKILL.md` Phase 6B strip check) are unchanged; this is a model-migration with substantive re-grounding of behavioral guidance, not a workflow change.

### Changed
- Recalibrated from the prior flagship to Claude Fable 5: model id `claude-fable-5` / 1M alias `claude-fable-5[1m]` (session-verified in Claude Code 2026-06-10; API-alias semantics Confidence: Medium pending Phase 1.5). Effort doctrine: documented benchmark default is adaptive thinking + max effort, but effort saturates by task class — skill sets `xhigh` for coding/agentic/analytical/orchestration work, `max` for genuinely frontier reasoning, `high` for cost-sensitive production. Thinking-only model (no thinking-off mode on Fable / Mythos 5).
- `SKILL.md` — description and H1 retargeted; Standing Environment Assumptions updated (model, subagent routing, effort doctrine); References Guide updated; Phase 1 Widget Call 3b effort and context options updated; Phase 1.5 query templates updated (Query 4 broadened — no brand qualifier, searches for the current flagship); Phase 3 anti-pattern cross-reference updated; Meta-Rules 7, 10, 11, 14, 15 refined for Fable 5 specifics; **Meta-Rules 16 and 17 added** (fallback-handling deployment guidance — rewritten to the no-silent-fallback posture in 2.1.0; multi-agent harness selection — three card-documented patterns, non-blocking preference).
- `calibration-protocol.md` — Queries 1, 3 retargeted; Query 4 broadened with a two-tier-architecture detection clause; Query 5 triggers add (d) multi-agent / dynamic-workflow drafts and (e) browser-use / computer-use surfaces; worked example updated.
- `task-heuristics.md` — anti-pattern table heading renamed `## Fable 5 Anti-Patterns`; rows 1–6, 9, 10, 14–18 retargeted to Fable 5 language; **rows 7, 8, 13, 19, 20, 30–35 re-grounded** with Fable 5 / Mythos 5 system-card citations (long-horizon failure taxonomy, larger-blast-radius framing, CoT controllability + non-faithful CoT, image 4,784-token + unpadded-dimensions rule, security carve-out + binary RE blockage on Fable, refusal inversion + advocacy carve-out, simulation framing, survival-pressure framing, weak constraint carryover); **rows 36–49 added** (fallback-unaware deployment; simulation framing; survival-pressure / shutdown-threat framing; detect-but-execute; over-engineering bias; context anxiety / premature stopping; fabricated checksum / approval / verification; multi-agent turf-war / shared-resource collision; blocking-orchestrator-as-default; missing programmatic-tool-calling primitive; missing `<result>` isolation; hedge / virtue-signal boilerplate is suppressible; trusting self-reports without evidence; effort-by-reflex / saturation-aware effort selection). Per-task-type guidance (code, data, research, writing, agent, orchestrated, creative, multi-modal, API, multi-model, knowledge-work) re-grounded to Fable 5.
- `dynamic-workflows.md`, `synthesis-deliverable.md`, `landscape-research.md`, `prompt-template.md`, `qa-checklist.md`, `platform-baseline.md`, `inference-heuristics.md`, `best-practices.md`, plugin `README.md`, root `README.md`, `marketplace.json`, `plugin.json` — model mentions, calibration stamps (2026-05-28 → 2026-06-10), and behavioral cross-references updated for Fable 5; stale-parameter, refusal, injection, and graded-framing pitfalls re-anchored.

### Added
- `references/fable-5-config.md` — new (renamed the prior-generation reference file to `fable-5-config.md`, fully rewritten). Carries the Fable 5 Critical Behaviors table partitioning prior-generation carryforward from Fable 5 / Mythos 5 new behaviors (two-model architecture; fallback semantics by surface; thinking-only; effort saturation; 1M public-API context with 10M effective via compaction for long-agentic-browse and no-compaction for deep-research single pass; new safeguard categories with invisible-throttling exception for frontier-LLM-dev; programmatic tool calling; `<result>` isolation; 4,784-tokens-per-image at 2576px with unpadded-dimensions rule; OSWorld 128K per-turn; model ids `claude-fable-5` and `claude-fable-5[1m]`). API parameters block updated; deprecated parameters retained and beta headers carried forward as "pending re-verification (Phase 1.5)".
- `references/fable-5-system-card.md` — new (renamed the prior-generation reference file to `fable-5-system-card.md`, fully rewritten). Carries the system-card behavioral findings: §2 Mythos-vs-Fable architecture and fallback semantics; §3 controlled-diligence eval evolution (mixed picture; honesty changed shape, not direction); §4 six-pattern long-horizon failure taxonomy + new long-run pathologies (context anxiety, fatigue, simulation framing, survival-pressure, turf wars, parallel force-push, workaround evasion, detect-but-execute, over-engineering); §5 elevated eval/grader awareness with non-inverting nuance reproduced; §6 refusal-calibration inversion + advocacy carve-out + security carve-out; §7 mixed prompt-injection picture (improved on most surfaces, regressed on browser-legacy and prefill); §8 CoT controllability up + monitorability down; §9 overriding-goal-at-top + weak constraint carryover; §10 capabilities and new surfaces (SWE-bench Pro 80%/80.3% for Fable/Mythos 5, FrontierSWE #1, GraphWalks Parents-1M 97.5%, Toolathlon 19 avg turns with Pass^3 58.3%, Real-World Finance v2 Elo 1374, GDPval-AA highest measured to date with fewer tokens than the prior flagship, three multi-agent harness patterns with non-blocking dominance).
- Added a prior-flagship fallback-floor reference (removed in 2.1.0). Originally documented the prior-flagship behavioral floor for Fable 5 fallback turns: effort default `high` (not max); bimodal thinking (off / adaptive); honesty by abstention + §2.3.3 long-horizon failure modes; over-refusal materially higher than Fable 5; prefill resistance better than Fable 5 but other injection surfaces worse. Loaded on demand only when Phase 3 flagged untrusted-content + action-capability deliverables or sensitive-domain (cyber / bio / chem / distillation / frontier-LLM-dev) deployment.

### Renamed
- Renamed the prior-generation reference files to `references/fable-5-config.md` and `references/fable-5-system-card.md` (content fully rewritten).

### Migration
The Deliverable Contract, phase structure, widget protocol, Standing Environment Assumption *structure* (the model id changed; the shape did not), and load-bearing anchors are unchanged. Existing optimized prompts produced under 1.1.0 remain valid; re-running the skill on a 1.1.0-era draft surfaces the rows 36–49 additions where they apply and the row 7 / 8 / 30 / 31 / 32 / 33 / 34 / 35 re-grounding where prior-generation assumptions need updating. API harnesses pinned to the prior-generation flagship should update to `claude-fable-5` and adopt the no-silent-fallback refusal posture introduced in 2.1.0 (Meta-Rule 16).

## [1.1.0] — 2026-05-28

Fold the now-available prior-flagship **System Card** into the skill. The v1.0.0 retarget was built from the release announcement and live docs and cited the system card only thinly; this release grounds the skill's behavioral guidance in the card's prompt-actionable findings. No phase, anchor, section-number, or Deliverable-Contract changes; no breaking changes.

### Added
- New calibration-stamped reference (later renamed to `fable-5-system-card.md` in 2.0.0) holding the system card's behavioral findings and the single home for the honesty nuance and the volatile numbers (controlled-diligence-eval gains, RL-episode rates, regression deltas, capability benchmarks), all flagged "re-verify via Phase 1.5 Query 4." Covers: the §2.3.3 long-horizon failure modes that persist alongside the controlled-single-shot honesty gains (fabrication, ignored correction, cheap-verification-skipped, goal drift, subagent fork-one-directive, recap hallucination); elevated evaluation/grader awareness (with the non-inverting rule — inhibiting it increases misalignment); refusal-calibration shifts; agentic-safety regressions on browser/computer use; the low CoT-controllability ceiling; the overriding-goal-at-top trap; and a directional, hedged capability/regression map. Loaded on demand by Phase 1.5 and Phase 3; not pre-read.
- `SKILL.md` Meta-Rule 15 — treat ingested tool/file/web/screenshot content as data, not instructions; gate ingest-and-act prompts behind confirmation.
- Prior-generation anti-pattern rows 30–35 (append-only) in `task-heuristics.md`: eval/grader telegraphing (narrowly scoped); assumes-the-model-refuses-less; long-session goal drift / premature "done"; self-authored CLAUDE.md/memory as a hard guardrail; overriding-goal-at-top; ingested-content-as-instructions.
- Plugin README: a one-line note that live calibration re-verifies the system-card findings.

### Changed
- `task-heuristics.md` rows 7, 8, 13 tightened: row 7 replaces the bare "more honest" framing with the controlled-single-shot vs long-horizon nuance and adds verify-before-assert for factual claims; row 8 adds the documented computer-use refusal regression; row 13 folds in CoT-shaping (low CoT controllability). The universal completion-verification rule and Sources block updated.
- `SKILL.md` Meta-Rules 10/11/14 refined (graded-framing companion to 10; controlled-vs-long-run honesty nuance + system-card pointer in 11; verify-before-assert caveat in 14); References Guide and Phase 3 cross-reference the new reference.
- The prior-generation config reference "New in [prior flagship]" table gained a §2.3.3 pointer row; honesty/refusal/safety posture bullets refined to point at the system-card reference (no number bloat); Sources expanded.
- `dynamic-workflows.md` §6 (`<result>`-tag deliverable isolation), §9 (subagent fork-one-directive; long-run context compaction), §11 (two new failure rows); Section 10 untouched.
- `synthesis-deliverable.md` authoring rules: check the overall conclusion against the orchestrator `## Mission` (not an intermediate milestone) and preserve lane uncertainty verbatim.
- `qa-checklist.md`, `calibration-protocol.md` (Query 4 now names the system card / new reference as canonical), `best-practices.md` (two pitfalls), and `inference-heuristics.md` (graded-framing sniff + new candidate failure modes) updated for the system-card findings.
- `plugin.json` description and the root README skill bullet note the system-card grounding.

### Fixed
- `plugins/prompt-optimization/README.md` — corrected stale prior-generation references and "Wave 2" → "verify-and-converge stage" (carry-overs missed in the v1.0.0 retarget); aligned the 2-O description and the landscape-research row to the dynamic-workflow framing the rest of the repo uses.

### Migration
No migration required. The Deliverable Contract, phase structure, widget protocol, Standing Environment Assumptions, and load-bearing anchors (the prior-generation anti-pattern section heading, `dynamic-workflows.md` Section 10) are unchanged. Existing optimized prompts from 1.0.0 remain valid; re-running the skill surfaces the new anti-pattern rows where they apply.

## [1.0.0] — 2026-05-28

Retarget to a newer prior-generation flagship and pivot the orchestrated-research deliverable from manual parallel-subagent dispatch to Claude Code **dynamic workflows**. This is a deliverable-shape change; the methodology, phases, and Deliverable Contract are otherwise intact.

### Changed
- Recalibrated the skill to the then-current prior-generation flagship; **effort default became `high`** on all surfaces (had been `xhigh` on Claude Code in the previous generation) — `xhigh` is now set explicitly for coding/agentic work; calibration stamps and Phase 1.5 query templates updated; sources re-verified 2026-05-28.
- The `orchestrated-research` deliverable is now a CLAUDE.md that orchestrates a Claude Code **dynamic workflow** (a JavaScript orchestration script Claude authors and a runtime executes), replacing the parallel-subagent-dispatch CLAUDE.md. The workflow script fans out deterministically (`parallel` / `pipeline`), so the prior-generation "orchestrator under-delegation" premise is obsolete.
- The always-on devil's-advocate dispatch is now the workflow's **verify-and-converge stage** (independent adversarial agents review findings, vote, converge); the verdict-ladder discipline is unchanged. The bundled `landscape-research` and `devils-advocate` agents are now dispatched as workflow `agentType`s (subagent fallback retained).
- Converted the skill's own internal Phase 2-O landscape research to dispatch a dynamic workflow (with parallel-subagent / single-threaded 2-S fallback).
- `SKILL.md` Meta-Rule 13, Phase 2/4/6B, and the References Guide updated for dynamic workflows; `task-heuristics.md` anti-pattern table reframed (rows for workflow-inline work, cap-ignoring, and write-capable-workflow guardrails added) and the heading renamed for the new generation.
- Added new platform capabilities where relevant: mid-conversation system messages, fast mode, the 1,024-token prompt-cache minimum, documented refusal `stop_details`, improved tool triggering and compaction handling, and the honesty gains (the completion-verification guardrail is now framed as defense-in-depth). Surfaced the prompt-injection-robustness nuance: guardrails matter more for write-capable workflows (agents run `acceptEdits`).
- `platform-baseline.md` gained a Dynamic-workflows capability row; `qa-checklist.md` gained workflow-deliverable gates; both READMEs, `marketplace.json`, and `plugin.json` updated.

### Renamed
- Renamed the prior-generation config reference to track the new generation's model id.
- `references/orchestrated-research.md` → `references/dynamic-workflows.md` (now the dynamic-workflow operational guide; Section 10 is the deliverable template).

## [0.2.1] — 2026-05-28

UX cleanup. The plugin previously shipped both a `prompt-optimization` skill and an `/optimize-prompt` slash command that wrapped the skill — for first-time users this surfaced as two near-identical entries in the skills menu, with the most plugin-name-shaped invocation actually being the wrong one (the command took args; the skill is the methodology). Consolidating to one entry point.

### Removed
- `commands/optimize-prompt.md` and the `commands/` directory. The slash command is gone.

### Changed
- The skill `prompt-optimization` is now the single invocation surface. Auto-triggers on prompt-improvement intent (paste a draft and say "optimize this", "harden this CLAUDE.md", "refine this system message"); invoke explicitly with `/prompt-optimization` when the auto-trigger doesn't fire or to start cold.
- File-driven workflows: type `/prompt-optimization` and paste the path when the skill's Phase 0 asks for the draft — one extra exchange vs. the prior `/optimize-prompt <path>` inline-arg shortcut, in exchange for a clean single-entry menu.
- Plugin description and READMEs updated to reflect the single-invocation surface.

### Migration
No migration required for the underlying methodology — the protocol, Standing Environment Assumptions, Deliverable Contract, references, and worker agents are unchanged from 0.2.0. Anyone who had built muscle memory around `/optimize-prompt` switches to `/prompt-optimization` (or just pastes the draft and lets the auto-trigger fire).

## [0.2.0] — 2026-05-28

Synthesis-as-owner: every orchestrated-research deliverable the skill produces now leads with an executive summary, runs a load-bearing source-validation revisit pass, and ships an always-on devil's-advocate / confirmatory subagent under a verdict-ladder discipline. Non-orchestrated tracks and the Express track are untouched.

### Added
- `references/synthesis-deliverable.md` — new reference. Carries the synthesis-as-owner contract, the executive-summary template (key findings → primary URLs + source-validation verdicts → confidence → per-finding conclusions → overall conclusion → devil's-advocate verdicts), the source-validation revisit protocol on load-bearing sources, the always-on devil's-advocate dispatch brief (adversarial / confirmatory modes), the verdict-ladder discipline (`unresolved` requires Tier-2+ sourced counterweight), and a populated worked example.
- `agents/devils-advocate.md` — new bundled read-only worker. Adversarial mode (search for the strongest sourced evidence each key finding's opposite is true) for thesis-advancing deliverables; confirmatory mode (audit included entries + surface missed candidates) for purely descriptive inventory / fact-extract deliverables. Mode chosen by the synthesis agent at dispatch via a published sniff; honors the topic-not-position discipline.
- Executive-summary structure leading every orchestrated deliverable.
- Source-validation log working artifact (chat run sheet only).
- Devil's-advocate findings pack working artifact (chat run sheet only).
- Plugin description in `plugin.json` updated to surface the new deliverable contract and the bundled `devils-advocate` agent.

### Changed
- `orchestrated-research.md` §7 (Synthesis design) — expanded from 5 mandatory tasks to 7 (added (6) load-bearing source-validation revisit pass and (7) devil's-advocate integration). The "synthesis is the orchestrator's job, not delegated" rule now carries an explicit Wave-2 dispatch exception.
- `orchestrated-research.md` §10 (Orchestrator `CLAUDE.md` template) — added `## Wave 2 — Devil's advocate (mandatory)` between Synthesis and Deliverable; updated `## Synthesis` to the seven-task surface; updated `## Deliverable` to lead with the executive summary per `synthesis-deliverable.md`; added the every-claim-cites-a-URL rule; updated the Acme SaaS worked example.
- `orchestrated-research.md` §11 (Failure mode catalog) — added rows for synthesis-without-source-attribution, missing executive summary, source-validation skipped, unverified load-bearing source kept at full confidence, devil's-advocate dispatched but not integrated, verdict-mush, adversarial-mode-on-descriptive-deliverable mode mismatch, and conclusion not traceable to a key finding.
- `SKILL.md` — Meta-Rule 13 extended from two invariants to four (added the synthesis-as-owner / executive-summary invariant and the source-validation + devil's-advocate invariant); Phase 4 template-selection now loads `synthesis-deliverable.md` alongside Section 10 for orchestrated-research; Phase 6B pre-write strip check extended to cover the new working-artifact types; References Guide updated.
- `qa-checklist.md` — added a new sub-block under Phase 4 covering the orchestrated-research deliverable gates (executive summary first, per-finding URLs + verdicts + confidence, per-finding conclusions, traceable overall conclusion, source-validation log discipline, devil's-advocate mode + integration, verdict-ladder enforcement, partition compliance). Added thresholds for executive-summary key-finding count, load-bearing URL floor, devil's-advocate dispatch count, and credibility floor to the at-a-glance table. Phase 6B pre-write strip checklist extended.
- `task-heuristics.md` — Orchestrated multi-agent research section extended with the new enhancements, anti-patterns to strip, and missing-enhancement checks; six new model-wide anti-pattern rows added (21–26): synthesis without source attribution, missing executive summary, devil's advocate as cosmetic pass, source-validation skipped, verdict-mush, mode mismatch.
- `prompt-template.md` — Orchestrator template section now lists Wave 2 + executive-summary-leading deliverable in the template structure, and points readers at `synthesis-deliverable.md` alongside `orchestrated-research.md` Section 10.
- `landscape-research.md` — added a cross-reference note clarifying that orchestrated-research **deliverable** synthesis is separately governed by `synthesis-deliverable.md` (so a reader does not conflate Phase 2-O internal landscape synthesis with deliverable synthesis).

## [0.1.1] — 2026-05-26

### Changed
- Refreshed documentation citations to the canonical Anthropic doc hosts
  (`platform.claude.com`, `code.claude.com`); the legacy `docs.anthropic.com` /
  `docs.claude.com` URLs still redirect but are no longer the canonical home.
- Qualified the `xhigh` effort default in the prior-generation config reference: it is the
  Claude Code default at that generation, while the Messages API default is `high`.
- Tightened the skill and `/optimize-prompt` command descriptions with an explicit
  boundary clause to reduce over-triggering.

### Added
- `LICENSE` (MIT).
- `CHANGELOG.md`.
- A note clarifying that the web-search tool version in the API example is
  calibration-stamped and re-confirmed by Phase 1.5 at runtime.
- Calibration stamps on the five reference files that lacked them (one product-fact
  stamp on `orchestrated-research.md`; methodology notes on `inference-heuristics.md`,
  `prompt-template.md`, `qa-checklist.md`, and `best-practices.md`).

### Fixed
- Normalized all cross-reference links from Obsidian-style `[[name]]` (which does not
  render on GitHub) to relative Markdown paths `[name.md](name.md)`.
- Added the missing prior-generation anti-patterns anchor to the anti-pattern cross-reference
  in `SKILL.md`.

## [0.1.0] — 2026-05-24

### Added
- Initial packaging of the `prompt-optimization` skill as a single-plugin Claude Code
  marketplace, with the `/optimize-prompt` command and the read-only `landscape-research`
  worker agent.
- Orchestrated Phase 2 landscape-research mode (2-O) with full propagation into the
  QA layer.

[2.1.0]: https://github.com/ZaBrisket/prompt-optimization/releases/tag/v2.1.0
[2.0.0]: https://github.com/ZaBrisket/prompt-optimization/releases/tag/v2.0.0
[1.1.0]: https://github.com/ZaBrisket/prompt-optimization/releases/tag/v1.1.0
[1.0.0]: https://github.com/ZaBrisket/prompt-optimization/releases/tag/v1.0.0
[0.2.1]: https://github.com/ZaBrisket/prompt-optimization/releases/tag/v0.2.1
[0.2.0]: https://github.com/ZaBrisket/prompt-optimization/releases/tag/v0.2.0
[0.1.1]: https://github.com/ZaBrisket/prompt-optimization/releases/tag/v0.1.1
[0.1.0]: https://github.com/ZaBrisket/prompt-optimization/releases/tag/v0.1.0
