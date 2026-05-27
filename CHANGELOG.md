# Changelog

All notable changes to this project are documented here. The format is based on
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres
to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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

[0.1.1]: https://github.com/ZaBrisket/prompt-optimization/releases/tag/v0.1.1
[0.1.0]: https://github.com/ZaBrisket/prompt-optimization/releases/tag/v0.1.0
