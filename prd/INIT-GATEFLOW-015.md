# INIT-GATEFLOW-015 — Skill efficacy, factory effectiveness, and delivery-copilot productivity metrics

**Status:** Draft PRD · **Author:** programme PM · **Date:** 2026-08-11  
**Outline:** [INIT-GATEFLOW-015-outline.md](./INIT-GATEFLOW-015-outline.md)  
**Vision:** [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md) §10 Success metrics  
**Component:** GATEFLOW · **Type:** product metrics / programme learning surface  
**Predecessor:** INIT-GATEFLOW-001/FR-10 (delivered `GET /api/v1/metrics/runs` v0 — stage duration p50/p95 by `workflow_node`, 90-day retention); the current implementation additionally breaks this down by `runner`/`model_id` — this initiative **extends** that surface with outcome-aware rates; it does not replace it.  
**Related, not blocking:** INIT-GATEFLOW-016 (operations dashboard, formerly INIT-GATEFLOW-004 — retired) — CAP-03 fulfills the ops initiative's gate-dwell-time / human-wait-time aggregate dependency; will eventually render these APIs; this initiative does not build screens.

---

## Document control

| Field | Value |
|---|---|
| Initiative ID | INIT-GATEFLOW-015 |
| Programme | prayog |
| Primary repo | drivestream-lab/gateflow |
| Supporting | prayog-skills (`workflow.yaml` node ids and `codify_hint.ref` values are read-only join keys; no contract change expected); prayog-meta (vision §10 is the source of truth this INIT operationalizes into a queryable API) |
| Explicitly **not** touched this INIT | gateflow-ops screens/charts; intent→merge lead time (needs a meta-side PRD/impact-acceptance timestamp — cross-repo, deferred); token/cost metrics (`CursorAgentRunner` exposes no usage data today); a precomputed-rollup worker job (live aggregation only); per-developer attribution |
| Metric pillars in scope | Skill/Spec Efficacy (CAP-02) · Factory Effectiveness (CAP-03) · Delivery Scorecard, gateflow-owned half (CAP-04) — behind a prerequisite persistence fix (CAP-01) |
| Target users | prayog-skills owners (act on CAP-02); PE / operators (act on CAP-03); programme leadership (reads CAP-04) |
| Depends on | INIT-GATEFLOW-001 (RunStore, `run_events`, `stages`, `/metrics/runs` v0 — delivered); INIT-GATEFLOW-011 (checkpoint evidence persistence — delivered); learning ingest (delivered). This initiative **composes existing tables**; it introduces no new store. |
| Exit proof | Unit tests against fixture events for each rate/derivation rule, plus a verify script asserting the three new endpoints return tenant-scoped, correctly-derived values against a seeded fixture wave |

---

## 1. Executive Summary

### Problem Statement

Gateflow already records almost everything needed to answer "is this working," but computes almost none of it. Today there is exactly one derived signal — `GET /api/v1/metrics/runs`, stage duration p50/p95 by `workflow_node`/`runner`/`model_id` — which answers **how long**, never **how well**, and never **whether it's getting better**. Three concrete gaps, verified directly against the running code, block every richer question: stage-outcome persistence collapses `findings`/`stopped`/`blocked` to `None` even though `RunOutcomeType` already defines them; a `STOPPED` run has no persisted link to the run that later continues its wave; and learning items already carry a skill-target linkage (`codify_hint.target`/`ref`) that nothing aggregates.

### Proposed Solution

Fix the outcome-persistence gap as a prerequisite (CAP-01), then ship three additive, tenant-scoped, live-aggregated read APIs that turn existing RunStore/checkpoint/learning data into decision-grade rates: **Skill/Spec Efficacy** (first-pass rate, findings rate, retry avg, learning codify rate — per `workflow_node`), **Factory Effectiveness** (unattended Pass-1 rate, stop-reason breakdown, gate dwell time, wave cycle time by lane), and a **Delivery Scorecard** covering only Gateflow's own evidence half (rework rate, initiatives closed with evidence, factory coverage among board-tracked initiatives). `GET /api/v1/metrics/runs` is unchanged.

### Success Criteria

| KPI | Target | Measurement |
|---|---|---|
| **Outcome persistence fixed** | 100% of `stage_completed` events recorded after CAP-01 ships persist the actual `RunOutcomeType` value, never `None` for `findings`/`stopped`/`blocked` | Unit |
| **Skill efficacy queryable** | `GET /api/v1/metrics/skill-efficacy` returns first-pass rate, findings rate, retry avg per `workflow_node`, matching a hand-computed fixture exactly | Unit + verify |
| **Factory effectiveness queryable** | `GET /api/v1/metrics/factory-effectiveness` correctly excludes automated `external-action` hops from breaking the unattended streak in a fixture wave that includes one | Unit + verify |
| **Delivery scorecard queryable** | `GET /api/v1/metrics/delivery-scorecard` counts rework **only** for findings/blocked re-entry after an already-passed `human-checkpoint`, verified against a fixture with both a pre-checkpoint and a post-checkpoint findings loop | Unit + verify |
| **No regression** | `GET /api/v1/metrics/runs` response shape and values are byte-for-byte unchanged against its existing test suite | Unit (existing suite green) |

### Capability ↔ wave ↔ requirement map

| CAP | Wave | Capability | REQ |
|---|---|---|---|
| CAP-01 | W0 | Persist full stage-outcome vocabulary (prerequisite) | REQ-01–REQ-03 |
| CAP-02 | W1 | Skill/Spec Efficacy API | REQ-04–REQ-10 |
| CAP-03 | W2 | Factory Effectiveness API | REQ-11–REQ-17 |
| CAP-04 | W3 | Delivery Scorecard API (gateflow-owned half) | REQ-18–REQ-23 |

---

## 2. User Experience & Functionality

### User Personas

| Persona | Role | Primary need |
|---|---|---|
| **prayog-skills owner** | Maintains `workflow.yaml`, `SKILL.md`, prompts | Prove whether a skill/prompt revision actually raised first-pass rate on its node, with evidence, not vibes |
| **PE / operator** | Authorizes lanes, owns gates | See exactly where mid-chain intervention still clusters, so the pin — not the PE's attention — absorbs that friction next |
| **Programme leadership** | Reads the scorecard | See whether rework caught after human sign-off is trending down and evidenced initiative closure is trending up |

### User Stories & Acceptance Criteria

#### US-1 — Persist the outcome the pin already resolves `(CAP-01)`

**As a** metrics consumer, **I want** every stage's actual outcome (including `findings`, `stopped`, `blocked`) persisted instead of collapsed to `None`, **so that** any rate metric built on top of it is trustworthy from day one.

**Acceptance criteria:**

- [ ] A `stage_completed` event recorded for an outcome of `findings`, `stopped`, or `blocked` persists that exact `outcome_type` value — never `None`
- [ ] A `stage_completed` event recorded for `success`/`failed` continues to persist exactly as today (no behavior change for the existing values)
- [ ] `aggregate_run_metrics` and the three new endpoints can group by the full `RunOutcomeType` vocabulary without special-casing missing values
- [ ] Events recorded **before** this fix ships are not backfilled or reinterpreted; any new rate metric explicitly reports its effective "data available from" point rather than silently treating old `None` values as `success`

#### US-2 — See if a skill/prompt revision is actually working `(CAP-02)`

**As a** prayog-skills owner, **I want** first-pass rate, findings rate, retry avg, and learning codify rate per `workflow_node` (optionally sliced by `model_id`/`prompt_revision`), **so that** I can prove a skill change helped instead of guessing from anecdote.

**Acceptance criteria:**

- [ ] `GET /api/v1/metrics/skill-efficacy` returns, per `workflow_node`: run count, first-pass rate, findings rate, retry avg, within the caller's tenant and the existing retention window
- [ ] First-pass rate for a node = stages reaching `success` with zero prior `findings` re-entry on that same node within the run, divided by total stages for that node
- [ ] Findings rate for a node counts **every** `findings` occurrence at that node, regardless of whether it happens before or after any `human-checkpoint` on the wave (that org-level split is CAP-04's job, not this one)
- [ ] The response supports filtering the same `workflow_node` by `model_id` and/or `prompt_revision`, so two revisions of the same skill are directly comparable
- [ ] Learning codify rate is reported **per `workflow_node`** only for learning items where `codify_hint.target == "skill"` (joined on `ref == workflow_node id`); items with `target` in `{SPEC, HARNESS, ENV}` report as flat, unjoined org-wide rates
- [ ] A `codify_hint.ref` that matches no known `workflow_node` reports under an explicit "unjoined" bucket — it is never dropped silently or treated as an error
- [ ] The response is scoped to the caller's tenant; a caller cannot see another tenant's skill-efficacy data

#### US-3 — See exactly where the factory still needs a human `(CAP-03)`

**As a** PE, **I want** unattended Pass-1 rate, a stop-reason breakdown, gate dwell time, and wave cycle time by lane, **so that** I know whether I'm still babysitting the pin and precisely where.

**Acceptance criteria:**

- [ ] `GET /api/v1/metrics/factory-effectiveness` returns unattended Pass-1 rate, tenant-scoped, within the retention window
- [ ] A wave counts as **unattended** when, between its entrypoint and the first STOP whose node `type` is `human-checkpoint` or whose `authorization` is `explicit`, no run event records a mid-chain PE-initiated skill dispatch — automated `external-action` hops (`authorization: automated`, e.g. `wave-pr-action`, `wave-done-action`) do **not** break the streak
- [ ] The response includes a `stop_reason` breakdown — counts of `STOPPED` runs grouped by the pin's own `stop_reason` string, passed through as-is (no gateflow-side hardcoded taxonomy)
- [ ] For a `STOPPED` run, gate dwell time is the elapsed time between that run's `ended_at` and the `created_at` of the next run for the same `initiative_id` + `wave_id` (inferred continuation — no new schema column this INIT)
- [ ] A `STOPPED` wave with no continuation run **yet** reports as currently open/waiting — never as a zero, negative, or silently omitted dwell value
- [ ] The response includes wave cycle time (existing `wave_duration_ms`) as p50/p95, grouped by lane (spec / implement / closeout) — reusing existing persisted data, no new persistence needed
- [ ] The response is scoped to the caller's tenant, consistent with CAP-02/CAP-04

#### US-4 — See whether the org is shipping more, with less rework `(CAP-04)`

**As a** programme leader, **I want** a delivery scorecard covering rework rate, initiatives closed with evidence, and factory coverage among board-tracked initiatives, **so that** I can evidence the delivery-copilot thesis instead of asserting it.

**Acceptance criteria:**

- [ ] `GET /api/v1/metrics/delivery-scorecard` is tenant-scoped and returns an explicit `as_of` snapshot timestamp, **plus** a trailing-90-day delta alongside the all-time cumulative value for each metric (resolves OQ-3 — see §2 Open questions)
- [ ] **Rework rate** counts a wave only when a `findings`/`blocked` re-entry into an earlier node occurs **strictly after** a `human-checkpoint` on that wave already recorded `outcome_type = pass`. Pre-checkpoint self-loops are **excluded** here — they are already counted in CAP-02's findings rate
- [ ] **Initiatives closed with evidence** counts initiatives whose closure/completion readout exists (via the existing `InitiativeReadoutService`/closure data) — not a manual or chat claim of completion
- [ ] **Factory coverage** is defined precisely as: of the EPIC-ticketed initiatives visible on the connected board (via `InitiativeReadoutService`'s existing "runs + board EPIC tickets" composition), the % that have at least one Gateflow-tracked run. This is **not** a claim about total company engineering activity Gateflow cannot observe — the response/docs state this scope explicitly
- [ ] Intent→merge lead time is **not** present in this response — it stays `unavailable`, same precedent as the existing `prd_approval: unavailable until W3 meta bridge` field, pending a future meta-bridge initiative
- [ ] The response is scoped to the caller's tenant

### Non-Goals

| Non-goal | Why |
|---|---|
| gateflow-ops screens/charts | Later — this INIT ships JSON APIs only |
| Intent→merge lead time | Needs a meta-side PRD/impact-acceptance timestamp; cross-repo, deferred |
| Token/cost metrics | `CursorAgentRunner` exposes no usage data today |
| Precomputed rollup worker job | Premature at current dogfood scale; live aggregation only, same pattern as `/metrics/runs` |
| New `continues_run_id` schema column | Inferred join (initiative+wave+chronology) is sufficient this INIT |
| Per-developer attribution | Explicit anti-goal — Gateflow attributes to runner/model/skill node, never a human identity doing the coding |
| Changing `/metrics/runs`' existing shape | Additive only; no breaking change to the delivered v0 surface |
| A literal "% of all company delivery work through Gateflow" metric | Not observable — Gateflow cannot see ad-hoc Cursor usage outside itself; factory coverage is explicitly scoped to board-tracked initiatives (see US-4) |
| Backfilling historical `stage_completed` events for CAP-01 | Old events keep their existing (lossy) values; new rate metrics report their effective start point instead |

### Assumptions

| ID | Assumption | Status | Dependent REQs |
|---|---|---|---|
| A1 | `workflow_node` ids in `run_events`/`stages` and `codify_hint.ref` values in learning items share the same namespace when `codify_hint.target == "skill"` (verified directly against `learning-extract`'s own output template: `ref: "loop-spec"` matches a real pin node id) | Confirmed by inspection | REQ-08, REQ-09 |
| A2 | `InitiativeReadoutService` already composes "runs + board EPIC tickets," so it can see EPIC-ticketed initiatives that never ran a Gateflow wave — this is the only honest denominator for factory coverage | Confirmed by inspection | REQ-21 |
| A3 | A `STOPPED` run's continuation is reliably the next-created run sharing the same `initiative_id` + `wave_id`; concurrent overlapping waves for one initiative are rare enough at current scale to accept as a named risk rather than requiring a new schema column `(Source: User-confirmed)` | Assumed — named risk | REQ-14, REQ-15 |
| A4 | `stop_reason` values are stable enough within a retention window to group meaningfully even without a fixed enum, modeled on `WorkflowEngine.resolve_next`'s specific no-allowlist approach to node resolution (not a documented system-wide taxonomy policy) | Confirmed by inspection | REQ-13 |
| A5 | Existing role gate `require_role(RoleType.TENANT_ADMIN)` (used by `GET /api/v1/metrics/runs`, `GET /initiatives`) is the correct role for all three new endpoints | Confirmed by existing pattern | REQ-04, REQ-11, REQ-18 |

### Error table (product-normative)

| Situation | Result | Side effects |
|---|---|---|
| Query spans events recorded before CAP-01 ships | Pre-fix events excluded from outcome-aware rates; response/docs state the effective data-available-from point | 0 miscount |
| `codify_hint.ref` matches no known `workflow_node` | Reported in an explicit "unjoined" bucket | 0 dropped/errored rows |
| `STOPPED` wave has no continuation run yet | Reported as open/waiting, no dwell value | 0 negative/zero dwell |
| Cross-tenant read attempt on any of the three new endpoints | Refused / scoped empty | 0 cross-tenant data exposure |
| Caller queries CAP-04 expecting intent→merge lead time | Field absent / `unavailable`, not fabricated | 0 fabricated cross-repo value |
| Model/prompt filter on CAP-02 matches no stages | Empty result for that `workflow_node`, not an error | 0 error |
| Malformed/unknown filter value on CAP-02 (`model_id`/`prompt_revision` matching no known value at all) | Named-clean empty response, not an error | 0 error |
| Valid tenant with no data recorded yet on any of the three new endpoints | Well-formed empty/zero response | 0 fabricated value, 0 404 |

### Open questions

| ID | Open question | Status |
|---|---|---|
| OQ-1 | Should CAP-02's findings-rate exclude the very first hop before any human ever saw the code, or count it identically to a mid-loop iteration? | **Resolved** — count identically; it is skill-quality signal (CAP-02), not org-rework (CAP-04, which uses the post-checkpoint-only definition, D3) |
| OQ-2 | Should the `stop_reason` taxonomy in CAP-03 be a fixed enum, or a free-text passthrough from the pin's own node ids? | **Resolved** — free-text passthrough, grouped as-is; no gateflow-side hardcoded taxonomy, modeled on `WorkflowEngine.resolve_next`'s specific no-allowlist approach to node resolution |
| OQ-3 | Does programme leadership want a rolling window or all-time for CAP-04's factory coverage / rework metrics? | **Resolved** — report **both**: an all-time cumulative value and a trailing-90-day delta, alongside an explicit `as_of` snapshot timestamp |

`OQ-1`–`OQ-3` from the outline are resolved above and not carried as blocking opens.

---

## 3. Functional requirements

### Capabilities

| ID | Capability | Covers |
|---|---|---|
| CAP-01 | Persist full stage-outcome vocabulary (prerequisite) | REQ-01–REQ-03 |
| CAP-02 | Skill/Spec Efficacy API | REQ-04–REQ-10 |
| CAP-03 | Factory Effectiveness API | REQ-11–REQ-17 |
| CAP-04 | Delivery Scorecard API (gateflow-owned half) | REQ-18–REQ-23 |

### Requirements

| ID | Requirement | Outline | Condition | Observable result | Evidence |
|---|---|---|---|---|---|
| REQ-01 | Stage-outcome recording maps the **full** `RunOutcomeType` vocabulary (`success/failed/stopped/blocked/findings/pending`) onto `stage_completed`, not just `success`/`failed` | D1 | Stage completes with any outcome | Persisted `outcome_type` matches the actual value; never silently `None` for `findings`/`stopped`/`blocked` | unit |
| REQ-02 | Existing `success`/`failed` recording behavior is unchanged | D1, regression guard | Stage completes with `success`/`failed` | Identical to pre-fix behavior | unit (existing suite green) |
| REQ-03 | Metrics queries can group by the full outcome vocabulary; pre-fix events are excluded from outcome-aware rates rather than miscounted as a default value | D1, Risk | Query spans pre-fix and post-fix events | Response reports an effective data-available-from boundary | unit + inspection |
| REQ-04 | `GET /api/v1/metrics/skill-efficacy` returns run count, first-pass rate, findings rate, retry avg per `workflow_node`, tenant-scoped, within the retention window | D5, D7, A5 | Authorized call | Response computed live from `stage_completed`/`stages` scoped to caller's tenant | unit + verify |
| REQ-05 | First-pass rate for a node = stages reaching `success` with zero prior `findings` re-entry on that node within the run, over total stages for that node | D1 | Query | Rate matches a hand-computed fixture | unit |
| REQ-06 | Findings rate for a node counts every `findings` occurrence at that node regardless of position relative to any `human-checkpoint` on the wave | D3 (CAP-02 scope) | Query | Rate includes all findings occurrences, pre- and post-checkpoint alike | unit |
| REQ-07 | Response supports filtering/grouping the same `workflow_node` by `model_id` and/or `prompt_revision` | Skill-efficacy use case | Query with filter | Filtered response only includes matching stage rows | unit + verify |
| REQ-08 | Learning codify rate is reported per `workflow_node` only for `codify_hint.target == "skill"` (join key `ref == workflow_node id`); `SPEC`/`HARNESS`/`ENV` report as flat, unjoined org-wide rates | D4, A1 | Query learning items | Per-node rate only for `target=="skill"`; others flat | unit |
| REQ-09 | Unmatched `codify_hint.ref` values report under an explicit "unjoined" bucket | D4, Risk | `ref` matches no known `workflow_node` | Item appears in unjoined bucket; 200 response, not dropped or errored | unit |
| REQ-10 | CAP-02 response is scoped by tenant | D5 | Cross-tenant query attempt | Refused / scoped empty | unit + verify |
| REQ-11 | `GET /api/v1/metrics/factory-effectiveness` returns unattended Pass-1 rate, tenant-scoped, within the retention window | D5, D7, A5 | Authorized call | Response computed live | unit + verify |
| REQ-12 | A wave is unattended when no mid-chain PE-initiated skill dispatch occurs between entrypoint and the first `human-checkpoint`/`authorization: explicit` STOP; automated `external-action` hops do not break the streak | D2 | Wave trace | Streak preserved across automated hops; broken only by PE dispatch or a real gate | unit |
| REQ-13 | Response includes a `stop_reason` breakdown grouped as free text, passthrough from the pin | D2, A4, OQ-2 | Query `RUN_STOPPED` events | Breakdown matches raw `stop_reason` values grouped correctly | unit + verify |
| REQ-14 | Response includes gate dwell time per `STOPPED` run, inferred via `initiative_id` + `wave_id` + chronological ordering to the next run | D8, A3 | Stopped run followed by a later run for the same wave | Dwell time computed correctly from the two timestamps | unit |
| REQ-15 | A `STOPPED` wave with no continuation run yet reports as open/waiting, never a zero/negative/omitted dwell value | D8, Risk | Stopped wave, no continuation yet | Reported as open, no dwell value | unit |
| REQ-16 | Response includes wave cycle time (`wave_duration_ms`) as p50/p95 grouped by lane (spec/implement/closeout) | Existing data reuse | Query runs by lane | Correct percentile grouping | unit + verify |
| REQ-17 | CAP-03 response is scoped by tenant | D5 | Cross-tenant query attempt | Refused / scoped empty | unit + verify |
| REQ-18 | `GET /api/v1/metrics/delivery-scorecard` is tenant-scoped and returns an `as_of` snapshot plus an all-time cumulative value and a trailing-90-day delta per metric | D5, D6, OQ-3 | Authorized call | Response includes all three framings | unit + verify |
| REQ-19 | Rework count/rate counts only findings/blocked re-entry occurring strictly after a `human-checkpoint` on that wave already recorded `pass`; pre-checkpoint self-loops are excluded | D3 | Wave with checkpoint pass then later findings | Counted only in the post-checkpoint case | unit |
| REQ-20 | Initiatives-closed-with-evidence counts initiatives with a persisted closure/completion readout, not a manual/chat claim | CAP-04 pillar | Query initiatives | Count matches initiatives with a persisted readout | unit + verify |
| REQ-21 | Factory coverage % = EPIC-ticketed initiatives visible on the connected board (via existing "runs + board EPIC tickets" composition) that have ≥1 Gateflow-tracked run, over total EPIC-ticketed initiatives visible on that board | D6, A2 | Query initiatives + board | Percentage matches this precise, scoped definition — not total company activity | unit + verify |
| REQ-22 | Intent→merge lead time is absent/`unavailable` in this response, not fabricated from partial data | D6 | Query | Field absent or explicitly `unavailable` | inspection |
| REQ-23 | CAP-04 response is scoped by tenant | D5 | Cross-tenant query attempt | Refused / scoped empty | unit + verify |

**Implementation notes (non-normative):** The natural as-built seam is a new sibling module to `src/business_services/metrics_emitter.py` (or new methods on it) reusing `RunRepository`/`RunEventRepository`/`StageRepository`/`LearningRepository`, mounted as new routes alongside `src/api/v1/metrics_routes.py`'s existing `get_run_metrics`. Module/class names are design detail for engineering.

---

## 4. Technical Specifications

### Architecture Overview *(design — not product vocabulary)*

```text
CAP-01 — outcome persistence fix
  RunOrchestrator._run_orchestrated_stage
    → MetricsEmitter.record_stage_duration(outcome=<full RunOutcomeType>)
    → run_events.stage_completed.outcome_type persisted correctly

CAP-02 — skill/spec efficacy
  GET /api/v1/metrics/skill-efficacy
    → query stage_completed events (post CAP-01) grouped by workflow_node
    → first-pass / findings rate / retry avg
    → join learning_items where codify_hint.target == "skill" on ref == workflow_node

CAP-03 — factory effectiveness
  GET /api/v1/metrics/factory-effectiveness
    → trace wave entrypoint → first human-checkpoint/explicit STOP
    → unattended = no mid-chain PE dispatch (automated external-action hops excluded)
    → stop_reason breakdown from RUN_STOPPED events
    → dwell time = STOPPED.ended_at → next run (same initiative_id+wave_id).created_at
    → wave_duration_ms p50/p95 by lane

CAP-04 — delivery scorecard (gateflow-owned half)
  GET /api/v1/metrics/delivery-scorecard
    → rework: findings/blocked re-entry strictly after a passed human-checkpoint
    → initiatives closed with evidence: closure/completion readout exists
    → factory coverage: board EPIC-ticketed initiatives with ≥1 Gateflow run
    → intent→merge lead time: absent/unavailable (meta-bridge, future INIT)
```

### Integration Points

| System | Use |
|---|---|
| `RunRepository` / `RunEventRepository` / `StageRepository` (existing) | Source of stage/run/event data for all three new endpoints |
| `LearningRepository` (existing) | Source of `codify_hint`-linked learning items for CAP-02 |
| `InitiativeReadoutService` (existing) | Source of board EPIC-ticket + closure/completion readout data for CAP-04 |
| `prayog-skills` `workflow.yaml` | Read-only join key (`workflow_node` ids); no contract change |
| `require_role(RoleType.TENANT_ADMIN)` (existing) | Auth gate reused for all three new endpoints, consistent with `GET /api/v1/metrics/runs` |
| `GET /api/v1/metrics/runs` (existing, INIT-GATEFLOW-001) | Unchanged; new endpoints are additive siblings |
| gateflow-ops | Not built this INIT; CAP-03 specifically fulfills 004's `REQ-37`/`A2` aggregate dependency (marked `[TBD in spec]` there); inherits three new JSON APIs to render later |

### Security & Privacy

- All three new endpoints are gated the same way as the existing `/metrics/runs` — `require_role(RoleType.TENANT_ADMIN)` — and additionally scoped by `tenant_id`, which `/metrics/runs` today is not (D5).
- No new PII or credential surface — these are aggregates over data Gateflow already persists.
- No per-developer identity is ever attributed by these endpoints (explicit non-goal, matches the system brief's anti-JTBD).
- Factory coverage (CAP-04) must not be presented as total-company coverage; the response/docs state its precise, narrower scope (REQ-21).

### AI / agent evaluation

Not a new model product. Quality bar is derivation-rule correctness against fixtures:

- Unit tests with hand-constructed `stage_completed`/`RUN_STOPPED`/learning-item fixtures for every rate/rule in §3 (first-pass, findings, unattended streak with an automated hop, dwell time with and without a continuation, rework post-checkpoint-only, factory coverage denominator).
- A verify script seeding a fixture wave through the real pin shape and asserting all three endpoints return the expected, hand-computed values.

---

## 5. Risks & Roadmap

### Delivery waves (product-normative)

| Wave | Intent | Exit REQs |
|---|---|---|
| **W0** | Persist full `RunOutcomeType` on `stage_completed` | REQ-01–REQ-03 |
| **W1** | Skill/Spec Efficacy API | REQ-04–REQ-10 |
| **W2** | Factory Effectiveness API (incl. inferred dwell-time join) | REQ-11–REQ-17 |
| **W3** | Delivery Scorecard API (gateflow-owned half) | REQ-18–REQ-23 |

### Technical risks

| Risk | Mitigation |
|---|---|
| Historical `stage_completed` events recorded before CAP-01 ships have no `findings`/`stopped`/`blocked` outcome | First-pass/findings-rate metrics reflect only data recorded after W0 ships; called out explicitly in response/docs, not silently backfilled (REQ-03) |
| Inferred dwell-time join (`initiative_id` + `wave_id` + chronology) can misattribute a continuation run if two waves for the same initiative genuinely overlap | Named accepted risk this INIT (A3); revisit with an explicit link column if it proves unreliable in practice |
| `codify_hint.ref` values drift from `workflow_node` ids over time (a skills-repo rename) | Best-effort join; unmatched refs report as unjoined rather than erroring (REQ-09) |
| Rework definition (REQ-19) undercounts real waste if a `human-checkpoint` reviewer rubber-stamps without genuine review | Named limitation — this metric measures **process evidence**, not reviewer diligence |
| Factory coverage (REQ-21) misread as "% of all company delivery work" | Explicit scoping in response/docs to board-tracked EPIC initiatives only (A2) |

### Phased rollout

- **MVP (this INIT, W0–W3):** outcome persistence fixed; three new tenant-scoped, live-aggregated metrics APIs; `/metrics/runs` unchanged.
- **Later, explicit follow-ups:** gateflow-ops screens rendering these APIs (INIT-GATEFLOW-016 track); a meta-bridge initiative for intent→merge lead time; precomputed rollups if/when live aggregation stops scaling; cost/token metrics if a runner exposes usage data.

---

## 6. Locked decisions reference

### Carried forward from outline (D1–D9)

| ID | Decision |
|---|---|
| D1 | Stage-outcome persistence must carry the full `RunOutcomeType` vocabulary — prerequisite, not optional |
| D2 | Unattended Pass-1 = automated `external-action` hops don't break the streak; only mid-chain PE dispatch or a `human-checkpoint`/`explicit` STOP ends it |
| D3 | Rework = findings/blocked re-entry **after** a `human-checkpoint` already passed; pre-checkpoint self-loops go to CAP-02's findings-rate instead |
| D4 | Learning codify-rate joins to `workflow_node` only when `codify_hint.target == "skill"`; other classes report flat |
| D5 | Shared `tenant_id` scoping across all three endpoints; window field stays pillar-specific |
| D6 | CAP-04 ships only Gateflow's evidence half; intent→merge lead time stays `unavailable` |
| D7 | Live aggregation on read; no new worker job or rollup table this INIT |
| D8 | Gate dwell time derived by inference (`initiative_id` + `wave_id` + chronology); no new schema column |
| D9 | No cost/token metrics this INIT |

### Resolved this Draft PRD (outline OQ-1–OQ-3 + discovery)

| ID | Resolution |
|---|---|
| OQ-1 | Findings-rate counts pre- and post-checkpoint occurrences identically at CAP-02 (skill-quality); CAP-04's rework metric is the one that applies the post-checkpoint-only split (D3) |
| OQ-2 | `stop_reason` is a free-text passthrough from the pin, grouped as-is — no gateflow-side hardcoded taxonomy |
| OQ-3 | CAP-04 reports an `as_of` snapshot **plus** both an all-time cumulative value and a trailing-90-day delta per metric |
| Discovery — factory coverage denominator | Scoped precisely to EPIC-ticketed initiatives visible on the connected board (A2) — not a claim about total company delivery activity Gateflow cannot observe |
| Discovery — auth pattern | Reuses the existing `require_role(RoleType.TENANT_ADMIN)` gate already used by `/metrics/runs` and `/initiatives` (A5) |

---

## 7. Next steps

1. Impact map → programme sign-off → gateflow implementation waves per §5.
2. Do **not** implement Gateflow code from this Draft PRD alone — follow the full detailed-design process (spec → feasibility → technical review → plan → waves).
3. No prayog-meta vision/ADR supersession is required this INIT — this initiative operationalizes vision §10, it does not change it.
4. Run `validate-requirements` against this Draft PRD before impact mapping.

---

## Appendix A — Data model changes (illustrative — engineering owns exact schema)

| Change | Notes |
|---|---|
| `record_stage_duration` outcome mapping | Extend beyond `success`/`failed` to the full `RunOutcomeType` enum — no new column, a mapping-logic fix |
| No new tables | All three new endpoints read existing `runs`, `stages`, `run_events`, `learning_items`/`learning_extracts`, and board/initiative readout data |
| No new `continues_run_id` column this INIT | Continuation inferred via `initiative_id` + `wave_id` + chronology (D8) |

## Appendix B — Target capability surface (illustrative — engineering owns exact routes)

| Capability | Notes |
|---|---|
| Fix stage-outcome persistence | Internal fix; no new route; CAP-01 |
| `GET /api/v1/metrics/skill-efficacy` | Tenant-scoped; CAP-02 |
| `GET /api/v1/metrics/factory-effectiveness` | Tenant-scoped; CAP-03 |
| `GET /api/v1/metrics/delivery-scorecard` | Tenant-scoped; `as_of` + cumulative + 90-day delta; CAP-04 |

Exact request/response field names and error body shapes are engineering's to design in the next stage — not fixed here. The three endpoints' broad shape (fields listed in §2's acceptance criteria) is product-normative; exact JSON key naming is not.

## Appendix C — Response shape conventions (product-normative on scoping, illustrative on exact keys)

| Convention | Applies to | Notes |
|---|---|---|
| `tenant_id` scoping | CAP-02, CAP-03, CAP-04 | Shared across all three (D5); `/metrics/runs` remains global/unscoped, unchanged |
| `retention_days` window field | CAP-02, CAP-03 | Matches existing `/metrics/runs` convention |
| `as_of` snapshot + cumulative + 90-day delta | CAP-04 only | Resolves OQ-3; CAP-04 is a state snapshot, not a percentile-over-window stat |
| Unjoined/open bucket instead of drop/error | CAP-02 (unjoined learning refs), CAP-03 (dwell with no continuation yet) | Never silently omit or fabricate a value |

---

## Exit gate

> **Exit:** Stage outcomes persist the full `RunOutcomeType` vocabulary; three new tenant-scoped metrics APIs (skill-efficacy, factory-effectiveness, delivery-scorecard) are live, computed by live aggregation on read, each verified against hand-computed fixtures for every derivation rule in §3; the existing `GET /api/v1/metrics/runs` is unchanged; no cross-repo dependency, no per-developer attribution, no cost/token metric, and no overclaimed "total company coverage" framing ships this INIT.

---

## References

- Outline: [prd/INIT-GATEFLOW-015-outline.md](./INIT-GATEFLOW-015-outline.md)
- Vision: [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md) §10 Success metrics
- System identity: [planning/prayog-system-product-brief.md](../planning/prayog-system-product-brief.md)
- Predecessor delivery: [prd/INIT-GATEFLOW-001.md](./INIT-GATEFLOW-001.md) (metrics v0)
- Codebase grounding: `drivestream-lab/gateflow` — `src/business_services/metrics_emitter.py`, `src/models/run_store_types.py` (`RunOutcomeType`), `src/business_services/checkpoint_evidence_service.py`, `src/models/learning_models.py`, `src/business_services/run_orchestrator.py`, `src/api/v1/metrics_routes.py`, `src/api/v1/initiatives_routes.py`; `drivestream-lab/prayog-skills` — `workflow.yaml`, `skills/development/learning-extract/references/output-template.md`
