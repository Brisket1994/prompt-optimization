# Changelog

All notable changes to this project are documented here. The format is based on
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres
to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
- Qualified the `xhigh` effort default in `references/opus-4-7-config.md`: it is the
  Claude Code default on Opus 4.7, while the Messages API default is `high`.
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
- Added the missing `#opus-47-anti-patterns` anchor to the anti-pattern cross-reference
  in `SKILL.md`.

## [0.1.0] — 2026-05-24

### Added
- Initial packaging of the `prompt-optimization` skill as a single-plugin Claude Code
  marketplace, with the `/optimize-prompt` command and the read-only `landscape-research`
  worker agent.
- Orchestrated Phase 2 landscape-research mode (2-O) with full propagation into the
  QA layer.

[0.2.0]: https://github.com/ZaBrisket/prompt-optimization/releases/tag/v0.2.0
[0.1.1]: https://github.com/ZaBrisket/prompt-optimization/releases/tag/v0.1.1
[0.1.0]: https://github.com/ZaBrisket/prompt-optimization/releases/tag/v0.1.0
