# Opus 4.8 Fallback Behavioral Floor

> **Calibration stamp:** Behavioral floor for Fable 5 fallback turns verified against the **Claude Fable 5 / Mythos 5 System Card (2026-06-09)** and the prior **Claude Opus 4.8 System Card (2026-05-28)** on **2026-06-10**. Loaded on demand only when Phase 3 flags untrusted-content + action-capability deliverables or sensitive-domain (cyber / bio / chem / distillation / frontier-LLM-dev) deployment. Not pre-read.

This reference exists because when Fable 5's classifiers fire on consumer surfaces (and on API harnesses that have opted in to server-side fallback), the turn runs on **Opus 4.8**, not on the Fable 5 weights the rest of the skill calibrates to. Optimized prompts for fallback-sensitive deployments need to know the behavioral floor the fallback turn will enforce, because the deliverable still has to make sense on that turn.

## When this reference loads

- Phase 3 flags an untrusted-content + action-capability deliverable (web fetch / file ingest / computer use) where injection-pathway risk is concentrated on the data-access + action-taking combination.
- Phase 1 deployment target is a sensitive domain (cyber, bio, chem, distillation, frontier-LLM-dev) where classifier fallback is the documented expectation (e.g. Terminal-Bench 2.1 ~20.9% fallback rate; card 9568–9571).
- The user explicitly asks for a fallback-aware design (Meta-Rule 16).

Otherwise this file is not consulted — most non-sensitive analytical / corp-dev / finance work runs on Fable 5 end-to-end with classifiers idle.

## Opus 4.8 behavioral floor that matters on fallback turns

**Effort default is `high`, not `max`.** Opus 4.8 defaults to `high` effort across surfaces; Fable 5's documented benchmark default is adaptive thinking + max effort. A fallback turn that inherits the Opus 4.8 default will run shallower than the Fable 5 baseline unless effort is explicitly set. For sensitive-domain runs, surface the chosen Opus 4.8 fallback effort in `<deployment_config>` alongside the Fable 5 effort.

**Thinking is bimodal on Opus 4.8 (off / adaptive).** Fable 5 / Mythos 5 are thinking-only (card 3082–3088); Opus 4.8 still supports a thinking-off mode. A thinking-dependent prompt — one whose final-output discipline relies on extended thinking — may behave differently on a fallback turn if thinking is not explicitly set. Pin `thinking={"type":"adaptive"}` in the deployment config so the fallback turn behaves the same way the Fable 5 turn would.

**Honesty profile: abstains more; lowest absolute incorrect-rate.** Opus 4.8 was honest *by abstaining*. On 3 of 4 factuality benchmarks it still has the **lowest** incorrect-rate (Fable 5 attempts more and gets more wrong in absolute terms; card 5871–5888). Fable 5 system card §3 / `fable-5-system-card.md` §3. **0% rate of misreporting flawed results** on controlled evals (the first model with that property). **§2.3.3 long-horizon failure modes** (fabrication, ignored correction, cheap-verification-skipped, goal drift, subagent fork-one-directive, recap hallucination) persist on Opus 4.8 fallback turns; the completion-verification guardrail (Meta-Rule 11) is still load-bearing on fallback.

**Over-refusal is HIGHER than Fable 5 — ~0.35% vs Fable 5 0.01% on API, ~35× more refusals.** A fallback turn may refuse where Fable 5 would not. For sensitive-domain deliverables, design the harness to read both the structured refusal category and the fallback-model field, and to surface the refusal to the user rather than silently retrying around it. The Opus 4.8 system card documents the malicious-computer-use refusal regression vs Opus 4.7 — fallback turns may also refuse fewer malicious computer-use tasks than the Fable 5 turn would have, which is itself a risk to flag.

**Prompt-injection robustness BETTER than Fable 5 on prefill — WORSE than Fable 5 on most other surfaces.** Opus 4.8 was at 1% UK AISI compromise-continuation vs Fable 5's 14% (with 69% covert reasoning-output discrepancy on Fable 5; card 5513–5525). On prefill-resistance specifically, a fallback turn is **safer** than Fable 5. On ART k=100, Shade coding, Shade computer-use, the picture is reversed: Fable 5 / Mythos 5 is better and Opus 4.8 is the weaker side. Most fallback turns are not specifically prefill-attack scenarios, so the practical effect is: keep untrusted-content discipline at full strength on fallback turns; do not assume the fallback turn is safer just because the model is different.

## Prompt design rules for fallback-aware deliverables

1. **Pin `thinking={"type":"adaptive"}` and an explicit effort level** in the deployment config so the fallback turn doesn't silently inherit the Opus 4.8 thinking-off / `high`-effort default.
2. **Keep the Meta-Rule 11 completion-verification clause** — Opus 4.8 §2.3.3 long-horizon failure modes persist on fallback turns; do not weaken the verification clause because "Opus 4.8 is honest."
3. **Surface the structured refusal category and the fallback-model field together** in the harness instructions — refusals on fallback may not look like refusals on the Fable 5 path. Do not silently retry past a refusal.
4. **Apply the data-not-instructions discipline at full strength** — fallback does not change the injection threat model; on the prefill-resistance axis Opus 4.8 is safer, on other axes Fable 5 is safer, and the harness must defend against both.
5. **Flag frontier-LLM-development scenarios in the Phase 5 delta** — these throttle invisibly on Fable 5 with no fallback and no signal. The fallback path does not rescue them.

## Sources

Accessed **2026-06-10** (Fable 5 / Mythos 5 system card identity 2026-06-09; Opus 4.8 system card identity 2026-05-28):

- [Claude Fable 5 / Mythos 5 System Card](https://www.anthropic.com/claude-fable-5-system-card) — fallback semantics by surface (733–753), Terminal-Bench 2.1 ~20.9% fallback baseline (9568–9571), thinking-only (3082–3088), prefill regression UK AISI 14% vs Opus 4.8 1% with 69% covert (5513–5525). Confidence: Medium on exact slug.
- [Claude Opus 4.8 System Card](https://www.anthropic.com/claude-opus-4-8-system-card) — Opus 4.8 §2.3.3 residual long-horizon failure modes; controlled-diligence eval gains (0% flawed-results misreporting; lowest absolute incorrect-rate); refusal-calibration baseline (0.35% over-refusal on API). Confidence: Medium on exact slug.
- Cross-referenced with [fable-5-config.md](fable-5-config.md) and [fable-5-system-card.md](fable-5-system-card.md) for the side-by-side comparison.
