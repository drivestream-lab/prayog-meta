# INIT-GATEFLOW-015 — Skill efficacy, factory effectiveness, and delivery-copilot productivity metrics (outline)

**Status:** outline · **Author:** programme PM · **Date:** 2026-08-11  
**Draft PRD:** [INIT-GATEFLOW-015](./INIT-GATEFLOW-015.md)  
**Vision:** [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md) §10 Success metrics  
**Component:** GATEFLOW · **Type:** product metrics / programme learning surface  
**Predecessor:** INIT-GATEFLOW-001/FR-10 (delivered `GET /api/v1/metrics/runs` v0 — stage duration p50/p95 by `workflow_node`, 90-day retention); the current implementation additionally breaks this down by `runner`/`model_id` — this initiative **extends** that surface with outcome-aware rates; it does not replace it.  
**Related, not blocking:** INIT-GATEFLOW-004 (operations dashboard) — CAP-03 fulfills 004's `REQ-37`/`A2` aggregate dependency, marked `[TBD in spec]` there (this INIT's "gate dwell time" = 004's "human-wait time"); will eventually render these APIs; this initiative does not build screens.

> **Outline** — problem framing and scope lock. Detailed requirements live in the
> Draft PRD.
>
> **Hard rule for this initiative:** every new metric must be derivable from data
> Gateflow already persists, or a named, minimal persistence change (REQ-0). No
> new metric may depend on functionality outside the gateflow repo — cross-repo
> composition (e.g. intent→merge lead time, which needs a meta-side timestamp)
> is explicitly deferred, not quietly absorbed into this INIT.

---

## Document control

| Field | Value |
|---|---|
| Initiative ID | INIT-GATEFLOW-015 |
| Artifact | `prd/INIT-GATEFLOW-015-outline.md` (this outline); Draft PRD at `./INIT-GATEFLOW-015.md` |
| Programme | prayog |
| Primary repo | drivestream-lab/gateflow |
| Supporting | prayog-skills (`workflow.yaml` node ids and `codify_hint.ref` values are read-only join keys for every new metric; no contract change expected); prayog-meta (vision §10 is the source of truth this INIT operationalizes into a queryable API) |
| Explicitly **not** touched this INIT | gateflow-ops screens/charts; intent→merge lead time (needs a meta-side PRD/impact-acceptance timestamp — cross-repo, deferred); token/cost metrics (`CursorAgentRunner` exposes no usage data today); a precomputed-rollup worker job (live aggregation only, same pattern as the existing `/metrics/runs`); per-developer attribution |
| Metric pillars in scope | **CAP-A** Skill/Spec Efficacy · **CAP-B** Factory Effectiveness · **CAP-C** Delivery Scorecard (gateflow-owned half only) |
| Target users | prayog-skills owners (act on CAP-A); PE / operators (act on CAP-B); programme leadership (reads CAP-C) |
| Depends on | INIT-GATEFLOW-001 (RunStore, `run_events`, `stages`, `/metrics/runs` v0 — delivered); INIT-GATEFLOW-011 (checkpoint evidence persistence — delivered); learning ingest (delivered per vision §9). This initiative **composes existing tables**; it introduces no new store. |

---

## 1. Problem statement

Gateflow already records almost everything needed to answer "is this working" —
it just doesn't compute any of it. Today there is exactly one derived signal:
`GET /api/v1/metrics/runs`, which answers **how long** a stage took (p50/p95 by
`workflow_node`/`runner`/`model_id`). It never answers **how well**, and never
answers **whether it's getting better**.

Three concrete gaps make that so, verified directly against the running code:

1. **Outcome persistence is lossy.** `RunOutcomeType` already defines
   `success | failed | stopped | blocked | findings | pending`, but the stage
   recorder only writes `success`/`failed` — every `findings`, `stopped`, or
   `blocked` outcome is persisted as `None`. No first-pass rate or findings
   rate is computable today, for any node, no matter how the question is
   asked.
2. **Stopped runs have no persisted link to their continuation.** When a wave
   STOPs at a `human-checkpoint` or an explicit-authorize gate, Gateflow
   writes a `RUN_STOPPED` event with a `stop_reason` — but resuming that wave
   later creates a **new** run. Nothing stores which new run continues which
   stopped one, so "how long did a wave sit waiting on a human" has no direct
   answer in the data as stored.
3. **The learning loop has no closing metric.** Learning items already carry
   `codify_hint.target`/`ref` pointing back at the exact skill/node they
   should fix (confirmed directly in `learning-extract`'s own output
   template: `codify_hint: { target: skill, ref: "loop-spec" }`, where
   `loop-spec` is a real `workflow_node` id). Nothing aggregates open-vs-
   codified by that target, so the vision's own "keep improving our skills"
   loop (§10) has no queryable proof it closes.

Without fixing these, three questions the programme keeps asking have no
evidence-backed answer, only anecdote:

- Are specs/skills actually getting better, or does it just feel that way?
- Is Gateflow reducing PE babysitting, or moving the babysitting later in the
  pin?
- Is the org shipping more governed work with less rework — the actual
  delivery-copilot thesis from the [system product brief](./../planning/prayog-system-product-brief.md)?

---

## 2. Proposed solution (summary)

| What we're building | Solves |
|---|---|
| **REQ-0 — fix stage-outcome persistence** | `findings`/`stopped`/`blocked` collapse to `None` today; blocks every rate metric below |
| **CAP-A — Skill/Spec Efficacy API** | No way to see if a skill/prompt revision actually raised first-pass rate |
| **CAP-B — Factory Effectiveness API** | No way to see unattended-Pass-1 rate, or exactly where PEs still get pulled in mid-chain |
| **CAP-C — Delivery Scorecard API** (gateflow half) | No way to see rework rate or factory coverage — org sees anecdote, not evidence |
| **Learning codify-rate join** | Learning loop exists and is correlated to a skill target; nobody can prove it closes |

**Unchanged on purpose:** `GET /api/v1/metrics/runs` (duration-only, global,
90-day retention) stays exactly as delivered in INIT-GATEFLOW-001. These are
three additive, purpose-built read APIs — not a replacement, not a breaking
change.

---

## 3. Locked product decisions

| ID | Decision |
|---|---|
| **D1** | Stage-outcome persistence must carry the **full** `RunOutcomeType` vocabulary (`success/failed/stopped/blocked/findings/pending`) on `stage_completed` — this is a **prerequisite**, not an optional nicety |
| **D2** | **Unattended Pass-1** = automated `external-action` hops (`authorization: automated`, e.g. `wave-pr-action`, `wave-done-action`) do **not** break the unattended streak; only a mid-chain PE-initiated skill dispatch, or a `human-checkpoint`/`authorization: explicit` STOP, ends it |
| **D3** | **Rework** = a `findings`/`blocked` re-entry into an earlier node, occurring **after** a `human-checkpoint` on that wave already recorded `pass`. Pre-checkpoint self-loops (e.g. plain `loop-spec` iterating on itself) are **excluded** from rework and counted instead in CAP-A's findings-rate |
| **D4** | Learning codify-rate joins to `workflow_node` **only** when `codify_hint.target == "skill"` (join key: `ref == workflow_node id`); `SPEC`/`HARNESS`/`ENV` classes report as flat, unjoined org-wide rates |
| **D5** | All three new endpoints share **tenant_id** scoping; the window field stays pillar-specific (`retention_days` for CAP-A/B, matching today's convention; `as_of` snapshot for CAP-C) — no single rigid response envelope forced across pillars with different semantics |
| **D6** | CAP-C ships **only** Gateflow's evidence half: initiatives closed with evidence, rework rate, factory coverage %. Intent→merge lead time stays `unavailable` pending a future meta-bridge initiative — same precedent as the existing `prd_approval: unavailable until W3 meta bridge` field |
| **D7** | Computation is **live aggregation on read**, same pattern as today's `aggregate_run_metrics` — no new worker job, no materialized rollup table, this INIT |
| **D8** | Gate dwell time (CAP-B) is derived by **inference** — join a `STOPPED` run to its continuation run via `initiative_id` + `wave_id` + chronological ordering. No new `continues_run_id` column this INIT |
| **D9** | **No cost/token metrics** this INIT — `CursorAgentRunner` exposes no usage data today; explicitly out of scope until a runner does |

---

## 4. Users and jobs-to-be-done

| User | Job to be done |
|---|---|
| **prayog-skills owner** | "I want to see whether a prompt/skill revision actually raised first-pass rate on its node, so I know whether to keep the change." |
| **PE / operator** | "I want to see exactly where I'm still being pulled in mid-chain, so I can push to close that gap in the pin instead of babysitting it forever." |
| **Programme leadership** | "I want evidence that more delivery work is flowing through the governed path, and that rework caught after human sign-off is going down — not anecdote." |

---

## 5. Scope — in

| Area | What we build |
|---|---|
| REQ-0 outcome persistence fix | Persist full `RunOutcomeType` on `stage_completed` instead of collapsing to `success`/`failed`/`None` |
| CAP-A Skill/Spec Efficacy API | `GET /api/v1/metrics/skill-efficacy` — first-pass rate, findings rate, retry avg, learning codify rate; by `workflow_node` (+ optional `model_id`/`prompt_revision`) |
| CAP-B Factory Effectiveness API | `GET /api/v1/metrics/factory-effectiveness` — unattended Pass-1 rate, stop-reason breakdown, gate dwell time, wave cycle time by lane |
| CAP-C Delivery Scorecard API (gateflow half) | `GET /api/v1/metrics/delivery-scorecard` — initiatives closed with evidence, rework rate, factory coverage % |
| Tenant scoping | All three new endpoints scoped by tenant/programme, consistent with run list/status — unlike today's global `/metrics/runs` |

---

## 6. Scope — out

| Out | Rationale |
|---|---|
| gateflow-ops screens/charts | Later — this INIT ships JSON APIs only |
| Intent→merge lead time | Needs a meta-side PRD/impact-acceptance timestamp; cross-repo, deferred |
| Token/cost metrics | Runner exposes no usage data today |
| Precomputed rollup worker job | Premature at current dogfood scale; live aggregation only (D7) |
| `continues_run_id` schema column | Inferred join (D8) is sufficient this INIT |
| Per-developer attribution | Explicit anti-goal — Gateflow attributes to runner/model/skill node, never a human identity doing the coding (matches the system brief's anti-JTBD) |
| Changing `/metrics/runs`' existing shape | Additive only; no breaking change to the delivered v0 surface |

---

## 7. Capability walkthroughs (summary)

| Capability | Done looks like |
|---|---|
| REQ-0 fix | A `findings`-outcome `stage_completed` event round-trips through Postgres with `outcome_type = findings`, not `None` |
| CAP-A | Same `workflow_node`, two `prompt_revision`s ⇒ two different, queryable, comparable first-pass rates |
| CAP-B | A wave whose only STOPs were automated `external-action` hops reports `unattended = true` end-to-end |
| CAP-C | A wave with a findings re-entry **after** an already-passed `human-checkpoint` increments `rework_count`; a wave whose findings loop happened before any checkpoint does not |

---

## 8. Delivery waves (proposed)

| Wave | Intent |
|---|---|
| **W0** | REQ-0: persist full `RunOutcomeType` on `stage_completed` |
| **W1** | CAP-A: Skill/Spec Efficacy API |
| **W2** | CAP-B: Factory Effectiveness API (incl. inferred dwell-time join) |
| **W3** | CAP-C: Delivery Scorecard API (gateflow-owned half) |

---

## 9. Success criteria (initiative exit)

1. A `findings` outcome on any orchestrated stage persists as `outcome_type = findings` (not `None`), and a first-pass rate is computable per `workflow_node` directly from that persisted value.
2. `GET /api/v1/metrics/skill-efficacy` returns first-pass rate, findings rate, and retry avg per `workflow_node`, tenant-scoped, within the existing retention window.
3. `GET /api/v1/metrics/factory-effectiveness` returns unattended Pass-1 rate and a stop-reason breakdown that correctly excludes automated `external-action` hops from breaking the unattended streak (D2).
4. `GET /api/v1/metrics/delivery-scorecard` returns a rework rate computed **only** from findings/blocked re-entries after an already-passed `human-checkpoint` (D3), plus factory coverage %.
5. None of the three new endpoints changes or regresses the existing `GET /api/v1/metrics/runs` response shape.

---

## 10. Dependencies and non-goals for partners

| Partner | Expectation |
|---|---|
| **prayog-skills** | No contract change expected — `workflow_node` ids and `codify_hint.ref` are read-only join keys. If a future pin renames a node id, historical metrics for the old id remain queryable as a distinct series (no silent merge) |
| **prayog-meta** | Vision §10 "Success metrics" is the source this INIT operationalizes into a queryable API; the PRD/impact-acceptance timestamp bridge needed for intent→merge lead time is future work, tracked separately, not absorbed here |
| **gateflow-ops** | Not built this INIT; inherits three new JSON APIs to render later |

---

## 11. Risks

| Risk | Mitigation |
|---|---|
| Historical `stage_completed` events recorded before REQ-0 ships have no `findings`/`stopped`/`blocked` outcome | First-pass/findings-rate metrics reflect only data recorded after this INIT ships; called out explicitly in the API response/docs, not silently backfilled |
| Inferred dwell-time join (D8: `initiative_id` + `wave_id` + chronology) can misattribute a continuation run if two waves for the same initiative genuinely overlap | Named accepted risk this INIT; revisit with an explicit link column if it proves unreliable in practice |
| `codify_hint.ref` values drift from `workflow_node` ids over time (a skills-repo rename) | D4's join is best-effort; unmatched refs report as unjoined rather than erroring |
| Rework definition (D3) undercounts real waste if a `human-checkpoint` reviewer rubber-stamps without genuine review | Named limitation — this metric measures **process evidence**, not reviewer diligence |

---

## 12. Open questions

| ID | Open question | Status |
|---|---|---|
| OQ-1 | Should CAP-A's findings-rate exclude the very first hop before any human ever saw the code, or count it identically to a mid-loop iteration? | Open — lean: count identically (it's skill-quality signal, not org-rework, per D3) |
| OQ-2 | Should the `stop_reason` taxonomy in CAP-B be a fixed enum, or a free-text passthrough from the pin's own node ids? | Open |
| OQ-3 | Does programme leadership want a rolling window (e.g. last 90 days) or all-time for CAP-C's factory coverage %? | Open |

`OQ-1`–`OQ-3` must be resolved in the Draft PRD. Do **not** implement from this outline alone.

---

## 13. Next steps

1. Resolve OQ-1–OQ-3 in the Draft PRD.
2. Impact map → programme sign-off → waves (from Draft PRD).
3. Do **not** build from outline alone — Draft PRD is normative.

---

## Exit gate (outline wording)

> **Exit:** Stage outcomes persist the full `RunOutcomeType` vocabulary; three
> new tenant-scoped metrics APIs (skill-efficacy, factory-effectiveness,
> delivery-scorecard) are live, computed by live aggregation on read; the
> existing `GET /api/v1/metrics/runs` is unchanged; no cross-repo dependency,
> no per-developer attribution, and no cost/token metric ships this INIT.

---

## References

- Vision: [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md) §10 Success metrics
- System identity: [planning/prayog-system-product-brief.md](../planning/prayog-system-product-brief.md)
- Predecessor delivery: [prd/INIT-GATEFLOW-001.md](./INIT-GATEFLOW-001.md) (metrics v0)
- Codebase grounding: `drivestream-lab/gateflow` — `src/business_services/metrics_emitter.py`, `src/models/run_store_types.py` (`RunOutcomeType`), `src/business_services/checkpoint_evidence_service.py`, `src/models/learning_models.py`, `src/business_services/run_orchestrator.py`; `drivestream-lab/prayog-skills` — `workflow.yaml`, `skills/development/learning-extract/references/output-template.md`
