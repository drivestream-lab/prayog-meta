# INIT-PRAYOG-SKILLS-002 — Workflow dispatch policy

**Status:** draft PRD · **Author:** programme PM · **Date:** 2026-07-23  
**Outline:** [INIT-PRAYOG-SKILLS-002-outline](./INIT-PRAYOG-SKILLS-002-outline.md)  
**Vision:** [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md)  
**Paired initiative:** [INIT-GATEFLOW-001](./INIT-GATEFLOW-001.md) (PolicyEngine consumes `dispatch`)  
**Component:** PRAYOG-SKILLS · **Type:** platform / delivery contract

> **Draft PRD** for Gate 1 review. Engineering implementation routes to
> `drivestream-lab/prayog-skills` **rc-1** branch and spec PR after Joint Gate 1.
> This INIT defines **automation eligibility SSOT** — orchestrators read the pin;
> they must not hardcode skill node lists.

---

## Document control

| Field | Value |
|-------|-------|
| Initiative ID | INIT-PRAYOG-SKILLS-002 |
| Programme | prayog |
| Primary repo | drivestream-lab/prayog-skills |
| Target branch | **rc-1** (feature); release tag TBD after Joint Gate 1 |
| Target users | PE, orchestrator consumers (Gateflow), skills maintainers |
| Gate 1 coupling | **Joint Gate 1** with INIT-GATEFLOW-001 — required before rc-1 merge, meta harness pin, or integration test |
| Contract id | `sdd-delivery/v2` (rc-1 additive semantics) |

---

## 1. Executive Summary

### Problem Statement

`sdd-delivery/v2` defines **navigation** via `workflow.yaml` node `type`
(`skill`, `human-checkpoint`, `external-action`, …). That tells consumers where
to **stop**, but not which `skill` nodes an **orchestrator may dispatch** vs
which remain **PE/agent manual**.

[INIT-GATEFLOW-001](./INIT-GATEFLOW-001.md) must not hardcode node id lists
(e.g. “wave lane = pre-implement … ground-spec”). Automation eligibility belongs
in **prayog-skills SSOT**, not Gateflow source or duplicate programme config.

### Proposed Solution

Add a required **`dispatch`** field on every **`type: skill`** node in
`workflow.yaml`:

| Value | Meaning |
|-------|---------|
| `manual` | PE or agent runs the skill; orchestrator **must not** dispatch |
| `orchestrated` | Orchestrator **may** dispatch when programme trigger + handoff authorize |
| `observed` | Orchestrator **must not** dispatch; **must** record timing from handoffs (metrics-only) — **reserved/future; excluded from rc-1 v1 contract tests** per Decision #1 (see Open Questions) |

Non-`skill` nodes (`human-checkpoint`, `external-action`, `decision`, `terminal`)
do **not** carry `dispatch` — behavior remains **`type`-only**.

Document semantics in `delivery-contract.yaml`, validate in
`tests/test_workflow_contract.py`, and ship on **prayog-skills rc-1**. Programme
repos pin the release tag after **Joint Gate 1** for Gateflow dogfood.

### Success Criteria (rc-1 delivery)

| KPI | Target | Measurement |
|-----|--------|-------------|
| **Contract completeness** | 100% of `type: skill` nodes have valid `dispatch` enum | `test_workflow_contract.py` CI |
| **Wave lane policy** | `pre-implement`, `loop-spec`, `verify`, `ground-spec` = `orchestrated` | Workflow diff + contract test |
| **Upstream policy** | All other skills = `manual` | Workflow diff + contract test |
| **Consumer neutrality** | Gateflow PolicyEngine implements dispatch check **without node id constants** | Gateflow spec + integration tests |
| **Backward compat** | v0.4.3 pins without `dispatch` → schema default `manual` only | Consumer fixture tests |
| **Pin readiness** | Meta harness pin record documents release tag ref after Joint Gate 1 | `.harness-pin.yaml` + CHANGELOG |

---

## 2. User Experience & Functionality

### User Personas

| Persona | Role | Primary need |
|---------|------|--------------|
| **PE** | Skills maintainer; contract owner | Change automation policy in one place (workflow graph) |
| **Orchestrator engineer** | Gateflow / future consumers | Read pin; never maintain parallel allowlists |
| **Tech lead** | Gate reviewer | Confidence that wave automation cannot bypass manual upstream skills |
| **Programme sponsor** | Learning owner | Clear boundary between manual SDD and orchestrated wave execution |

### User Stories & Acceptance Criteria

#### US-1 — PE declares which skills orchestrators may dispatch

**As a** PE, **I want** automation eligibility declared on each workflow skill node **so that** Gateflow and future orchestrators read policy from the contract pin, not hardcoded lists.

**Acceptance criteria:**

- [ ] Every `type: skill` node in `workflow.yaml` includes `dispatch: manual | orchestrated`
- [ ] Wave lane skills (`pre-implement`, `loop-spec`, `verify`, `ground-spec`) are `orchestrated`
- [ ] All other skill nodes are `manual`
- [ ] No partial graph — rc-1 ships with full annotation in one change set

#### US-2 — Orchestrator consumer reads dispatch without node allowlists

**As an** orchestrator engineer, **I want** a normative dispatch algorithm documented in the delivery contract **so that** PolicyEngine matches prayog-skills SSOT.

**Acceptance criteria:**

- [ ] `delivery-contract.yaml` documents `dispatch` enum semantics and consumer algorithm (aligned with [INIT-GATEFLOW-001 FR-5](./INIT-GATEFLOW-001.md))
- [ ] **No skill id allowlists** in consumer source; eligibility read from pinned `workflow.yaml` `dispatch` field only
- [ ] Missing `dispatch` on pre-release pins (v0.4.3) → schema default `manual` (documented; no orchestrator fallback allowlist)

#### US-3 — Skills maintainer changes policy safely

**As a** skills maintainer, **I want** contract tests to fail when `dispatch` is missing or invalid **so that** policy regressions cannot merge silently.

**Acceptance criteria:**

- [ ] `tests/test_workflow_contract.py` asserts presence and enum validity on all skill nodes
- [ ] CI fails on invalid enum value or missing field
- [ ] Workflow scenario fixtures remain green (navigation unchanged)

#### US-4 — Programme pin moves to release tag after Joint Gate 1

**As a** PE, **I want** release notes and CHANGELOG to document pin guidance **so that** meta and Gateflow upgrade together after Gate 1.

**Acceptance criteria:**

- [ ] rc-1 CHANGELOG describes `dispatch` field, schema default, and wave-lane policy
- [ ] Migration note covers v0.4.3 → release pin for consumers
- [ ] Meta harness pin updated **only after** Joint Gate 1 approval

### Functional Requirements (FR) — with Acceptance Criteria

| ID | Requirement | Acceptance criteria |
|----|-------------|---------------------|
| **FR-1** | Required `dispatch` on every `type: skill` node | All **13** skill nodes in current graph annotated; non-skill nodes omit field; YAML schema/doc states field is required for skill nodes on rc-1+ |
| **FR-2** | Full `workflow.yaml` graph annotated per rc-1 default policy | Single PR annotates entire graph; no follow-up “fill remaining nodes” allowed for rc-1 exit |
| **FR-3** | Document `dispatch` in delivery contract + navigation refs | `delivery-contract.yaml` adds enum definition, schema-default rule, consumer algorithm; `references/handoff-envelope.md` cross-ref paragraph documents **optional future `executed_by` only** — not producer instructions for manual vs orchestrated runs `(Source: User-confirmed)`; no new required handoff fields in v1 |
| **FR-4** | Extend `test_workflow_contract.py` | New tests: every skill node has `dispatch`; value ∈ `{manual, orchestrated}` for rc-1 v1; wave lane values match policy table; **exactly four** nodes are `orchestrated` with set `{pre-implement, loop-spec, verify, ground-spec}` |
| **FR-5** | CHANGELOG + release notes | Pin guidance for meta/gateflow; breaking-change callout: consumers must not use node allowlists |
| **FR-6** | Orchestrator-neutral naming | Enum value `orchestrated` (not `gateflow` or product-specific strings); documentation refers to “orchestrator consumers” |
| **FR-7** | Backward compatibility for v0.4.3 pins | Documented schema default: absent `dispatch` → treat as `manual`; consumers must not infer orchestration from node id lists |
| **FR-8** | Wave-lane boundary explicit | Post-`board-seed` wave skills only may be `orchestrated`; upstream meta/spec skills remain `manual` |

### Error Handling (consumer-facing — normative for Gateflow)

These are **consumer obligations** documented in FR-3; Gateflow implements in INIT-GATEFLOW-001 FR-5.

| Condition | Required consumer behavior |
|-----------|---------------------------|
| `next.type != skill` | STOP (human-checkpoint, external-action, decision, terminal) |
| `next.dispatch == manual` | STOP; may record metrics from handoff timestamps |
| `next.dispatch == orchestrated` but trigger not authorized | STOP |
| `dispatch` field absent (v0.4.3 pin) | Treat as `manual`; Gateflow W0/W1 pre-release-pin blocks with comment per INIT-GATEFLOW-001 Decision #9 |
| Invalid / unknown `dispatch` value | STOP; log contract violation; do not dispatch |

### Error Handling (prayog-skills — producer-side)

| Condition | Required producer behavior |
|-----------|---------------------------|
| Invalid or missing `dispatch` on skill node | `test_workflow_contract.py` CI **fail**; merge blocked |
| Orchestrated count ≠ 4 or wrong skill set | Contract test **fail** (FR-4) |
| Pin upgrade v0.4.3 → release tag mid-programme | Consumers apply schema default `manual` until pin upgrade — see [Migration §4](#migration-v043--release-pin) |

### Non-Goals (rc-1 delivery)

| Non-goal | Rationale |
|----------|-----------|
| Gateflow implementation | INIT-GATEFLOW-001 |
| Change skill procedures / SKILL.md bodies | Unless handoff template note only |
| Auto-merge or gate label semantics | Unchanged; `external-action` / human gates |
| Graphify / ToolProvider | Gateflow + separate skills initiative |
| Rename or restructure workflow graph | Additive field only |
| Orchestrator node allowlists in consumer repos | Explicit anti-pattern; forbidden by paired INIT |
| `observed` enum in rc-1 v1 | Deferred — manual + handoff metrics sufficient for H1 (Open Questions) |

### Product Principles

1. **SSOT in prayog-skills** — dispatch policy lives in pinned `workflow.yaml`, not orchestrator code
2. **Navigation unchanged** — `dispatch` adds eligibility; it does not alter `outcomes` routing
3. **Explicit automation boundary** — only declared `orchestrated` skills may be machine-dispatched
4. **Orchestrator-neutral contract** — any future consumer uses the same enum and algorithm
5. **Fail closed** — missing or manual → no orchestrator dispatch
6. **Joint Gate 1 pairing** — rc-1 merge and Gateflow `dispatch` consumption advance together

---

## 3. AI System Requirements

This INIT is a **delivery contract** change, not an LLM product. Evaluation focuses
on contract correctness and consumer compliance.

### Evaluation Strategy

| Dimension | Method | Pass threshold (rc-1) |
|-----------|--------|------------------------|
| **Graph annotation** | Workflow diff review + FR-4 tests | **13/13** skill nodes annotated; **9 manual**, 4 orchestrated |
| **Wave lane cardinality** | FR-4 contract test | Exactly 4 orchestrated nodes; set equals wave lane table |
| **Navigation regression** | Existing `test_scenario_routes` + handoff skill checks | 100% green; zero routing changes |
| **Consumer algorithm alignment** | Cross-review with INIT-GATEFLOW-001 FR-5 fixtures | Identical stop/dispatch rules for shared scenarios |
| **Schema default** | Consumer test: v0.4.3 workflow fixture without `dispatch` | All skills treated as `manual`; zero orchestrated dispatches |
| **Anti-allowlist** | Gateflow static analysis / code review | Zero hardcoded wave skill id lists in PolicyEngine |

---

## 4. Technical Specifications

### Architecture Overview

```text
workflow.yaml (skill nodes + dispatch)
        │
        ▼
delivery-contract.yaml ──► documents enum + consumer algorithm
        │
        ▼
test_workflow_contract.py ──► CI gate (presence, enum, wave lane)
        │
        ▼
Pinned by meta / gateflow ──► PolicyEngine reads dispatch at runtime
```

**Additive change only:** node ids, `type`, `outcomes`, and `profile` are
unchanged. rc-1 adds one field per skill node.

### Workflow node annotation (rc-1 default policy)

Full graph — **normative for rc-1**:

| Node id | `type` | `dispatch` (rc-1) | Rationale |
|---------|--------|-------------------|-----------|
| `validate-requirements` | skill | `manual` | Meta upstream — PE/agent invoked |
| `review-findings` | skill | `manual` | Meta upstream |
| `update-documents` | skill | `manual` | Meta upstream |
| `prd-impact-map` | skill | `manual` | Meta upstream |
| `spec-draft` | skill | `manual` | Spec upstream |
| `initiative-feasibility` | skill | `manual` | Spec upstream |
| `spec-technical-review` | skill | `manual` | Spec upstream |
| `spec-implementation-plan` | skill | `manual` | Spec upstream |
| `board-seed` | skill | `manual` | Last manual skill before wave lane |
| `pre-implement` | skill | `orchestrated` | Wave lane entry |
| `loop-spec` | skill | `orchestrated` | Wave execution |
| `verify` | skill | `orchestrated` | Wave execution |
| `ground-spec` | skill | `orchestrated` | Wave execution |

Non-skill nodes (`requirements-human-decision`, `gate-1`, `prd-pr-action`, …)
**omit** `dispatch`.

**Example (rc-1 YAML fragment):**

```yaml
pre-implement:
  type: skill
  profile: development
  dispatch: orchestrated
  outcomes:
    pass: loop-spec
    # ...
```

### Consumer dispatch algorithm (normative)

For any orchestrator consumer (Gateflow PolicyEngine, future):

```text
next = resolve(handoff.stage, handoff.outcome)

if next.type != skill:
    STOP   # human-checkpoint, external-action, decision, terminal

dispatch_value = next.dispatch ?? manual   # schema default when field absent

if dispatch_value != orchestrated:
    STOP or observe-only (manual; observed when adopted)

if not programme_trigger_authorized:
    STOP

dispatch skill(next)
```

**Extensions from INIT-GATEFLOW-001** (orchestrator-specific; not duplicated here):
pre-release pin block, concurrent run reject, handoff blockers, contract match,
`human_checkpoint` honor — see [INIT-GATEFLOW-001 §3 Dispatch Preconditions](./INIT-GATEFLOW-001.md).

### Contract and version strategy

| Item | rc-1 lean |
|------|-----------|
| Contract id | Remain `sdd-delivery/v2` with **rc-1 additive** field semantics |
| Doc bump | rc-1 CHANGELOG required; explicit `sdd-delivery/v2.1` label **TBD at Joint Gate 1** |
| Backward compat | v0.4.3 pins lack `dispatch` → consumers apply schema default `manual` only |
| Handoff envelope | No required new fields in v1; optional future `executed_by: manual \| orchestrated` noted in handoff spec cross-ref |

### Migration: v0.4.3 → release pin {#migration-v043--release-pin}

| Actor | Action |
|-------|--------|
| **prayog-skills** | Merge **rc-1**; tag release after Joint Gate 1 |
| **meta harness** | Update `.harness-pin.yaml` to release tag **after** Joint Gate 1 |
| **Gateflow** | PolicyEngine reads `dispatch`; pre-release-pin loud block (INIT-GATEFLOW-001 Decision #9) |
| **Consumers on v0.4.3** | Continue safely — all skills implicit `manual`; no orchestration until pin upgrade |

### Integration Points

| Integration | Direction | Mechanism |
|-------------|-----------|-----------|
| **INIT-GATEFLOW-001** | Consumer | PolicyEngine + FR-5 reads pinned `workflow.yaml` |
| **meta harness** | Pin consumer | `.harness-pin.yaml` → prayog-skills release tag |
| **launchpad** | Indirect | Harness sync before agent dispatch (unchanged) |
| **handoff-envelope.md** | Documentation | Cross-reference `dispatch`; no new required fields v1 |

### Security & Privacy

| Concern | Requirement |
|---------|-------------|
| **Policy tampering** | `workflow.yaml` changes require PE review + contract tests in prayog-skills CI |
| **Bypass via consumer allowlists** | Forbidden — Gateflow FR-5 explicitly disallows hardcoded skill lists |
| **Silent automation expansion** | Adding `orchestrated` to upstream skills requires explicit workflow diff + Gate 1 awareness |

---

## 5. Risks & Roadmap

### Phased Rollout

| Phase | Scope | Entry criteria |
|-------|-------|----------------|
| **Phase A — Contract (rc-1)** | Annotate workflow; extend tests; document algorithm | This PRD + Joint Gate 1 |
| **Phase B — Pin + integrate** | Meta harness pin; Gateflow PolicyEngine consumes `dispatch` | Joint Gate 1 passed; release tagged |
| **Phase C — Dogfood** | Label-triggered wave runs | INIT-GATEFLOW-001 W1 exit + release pin active |

#### Normative delivery sequence (paired with INIT-GATEFLOW-001)

```text
Draft PRDs (both INITs) + validate-requirements (parallel ok)
  → Joint Gate 1 (single session — blocking)
  → prayog-skills rc-1 merge + release tag
  → meta harness pin
  → Gateflow W1 PolicyEngine + dogfood Phase B
```

**May proceed before Joint Gate 1:** outline/Draft PRD review, `/validate-requirements`,
impact-map scoping on each INIT.

**Blocked until Joint Gate 1:**

| Blocked action | Rationale |
|----------------|-----------|
| Merge `dispatch` to prayog-skills **rc-1** / release tag | Consumer semantics must match Gateflow FR-5 |
| Meta harness pin to release tag | Pin without aligned Gateflow policy is untestable |
| Gateflow W1 PolicyEngine reading `dispatch` | Must match rc-1 annotation |
| Dogfood Phase B (label-triggered wave runs) | End-to-end contract must be Gate 1–approved |

### Dependencies

#### Joint Gate 1 (blocking — with INIT-GATEFLOW-001)

Single session confirms:

- `dispatch` enum (`manual` + `orchestrated`; `observed` defer decision)
- Full wave-lane annotation table (§4)
- Normative consumer algorithm (§4)
- Align INIT-GATEFLOW-001 metrics `observed` / `dispatch_mode: observed` handling with rc-1 v1 enum deferral
- rc-1 pin timing relative to Gateflow W0 merge
- v0.4.3 schema-default bounds (missing → `manual` only — no orchestrator node allowlists)

#### Technical dependencies

| Dependency | Assumption |
|------------|------------|
| INIT-GATEFLOW-001 Draft PRD | Gateflow FR-5 consumes `dispatch`; no hardcoded skill lists |
| **rc-1 branch** | Active development on prayog-skills `(Source: User-confirmed)` |
| `test_workflow_contract.py` | Existing navigation tests remain green |
| Meta harness | Pin moves to release tag after Joint Gate 1 |
| **INIT-GATEFLOW-001 metrics** | `dispatch_mode: observed` in FR-10 must align with rc-1 v1 enum deferral at Joint Gate 1 |

#### Assumptions

| ID | Assumption | Status | Dependent FRs |
|----|------------|--------|---------------|
| A1 | Joint Gate 1 passes with INIT-GATEFLOW-001 before release tag | Pending | All rc-1 delivery |
| A2 | Wave lane remains four skills (`pre-implement` … `ground-spec`) for H1 | Confirmed | FR-2, FR-8 |
| A3 | Gateflow is first orchestrator consumer | Confirmed | FR-3, FR-6 |
| A4 | `observed` enum not required for H1 dogfood | Proposed | Open Questions #1 |

### Risk Register

| Risk | Likelihood | Impact | Mitigation | Owner |
|------|------------|--------|------------|-------|
| rc-1 merge delays Gateflow W1 dispatch | Medium | Medium | Schema default `manual`; Gateflow loud pre-release-pin block; no consumer allowlists | PM + PE |
| Wrong `dispatch` on node | Low | High | PE workflow diff review; FR-4 contract tests; Joint Gate 1 sign-off | PE |
| Enum proliferation | Medium | Low | Start with `manual` + `orchestrated`; defer `observed` | PM |
| Consumer allowlist drift | Medium | High | Paired INIT non-goals; Gateflow FR-5 AC; code review | PE |
| Pin mismatch (meta vs gateflow) | Low | High | Joint Gate 1 coordinates pin timing | PM |

### Decisions (draft PM stance — Joint Gate 1 confirmation pending) {#decisions-resolved}

| # | Decision | Resolution |
|---|----------|------------|
| 1 | rc-1 v1 enum scope | **`manual` + `orchestrated` only** — defer `observed` to post-dogfood unless Joint Gate 1 overrides |
| 2 | Wave lane annotation | **`pre-implement`, `loop-spec`, `verify`, `ground-spec` = `orchestrated`**; all other skills = `manual` |
| 3 | Schema default (v0.4.3) | **Missing `dispatch` → `manual`** — no consumer node allowlist fallback |
| 4 | Naming | **`orchestrated`** — orchestrator-neutral (FR-6) |
| 5 | Graph change scope | **Full annotation in one rc-1 PR** — no partial graph (FR-2) |
| 6 | Handoff envelope v1 | **No new required fields** — optional future `executed_by` noted in cross-ref only `(Source: User-confirmed)` |

### Open Questions (Joint Gate 1 agenda)

*Decisions 1–6 are Draft PM stance — confirm with PE at Joint Gate 1.*

| # | Question | Owner |
|---|----------|-------|
| 1 | **`observed` enum:** defer rc-1 v1 (recommended) vs include now for metrics-only skills? | PM + PE |
| 2 | **Contract doc bump:** rc-1 CHANGELOG only vs explicit `sdd-delivery/v2.1` identifier? | PE |
| 3 | **Release tag name:** `v0.5.0-rc.2` vs `v0.4.4-rc.2` vs other? | PE |
| 4 | **Release pin timing:** same sprint as Gateflow W0 merge vs pin-first before Gateflow integration? | PM + PE |
| 5 | **INIT-GATEFLOW-001 metrics alignment:** reconcile FR-10 `dispatch_mode: observed` with rc-1 v1 enum deferral? | PM + PE |

---

## Appendix A — Traceability

| Outline section | PRD section |
|-----------------|-------------|
| §2 Proposed solution | §1 Executive Summary; §4 Workflow annotation |
| §3.1 Capabilities FR-1–FR-6 | §2 Functional Requirements |
| §3.2 Illustrative annotation | §4 Workflow node annotation table |
| §3.3 Consumer algorithm | §4 Consumer dispatch algorithm |
| §4 Non-goals | §2 Non-Goals |
| §5 Contract strategy | §4 Contract and version strategy |
| §6 Joint Gate 1 | §5 Dependencies; §5 Phased Rollout |
| §7 Success criteria | §1 Success Criteria |
| §8 Risks | §5 Risk Register |
| §9 Open questions | §5 Open Questions |
| Appendix A checklist | §4 (full node table); §4 Migration; §2 FR-3 handoff cross-ref |

## Appendix B — Relationship to INIT-GATEFLOW-001

| INIT-PRAYOG-SKILLS-002 | INIT-GATEFLOW-001 |
|------------------------|-------------------|
| Defines `dispatch` on workflow nodes | Reads `dispatch`; FR-5 |
| Process / automation SSOT | Execution control plane |
| rc-1 release on prayog-skills | Pins release tag for dogfood |
| Contract tests | PolicyEngine implementation |
| **Joint Gate 1 (blocking)** | **Joint Gate 1 (blocking)** |

## Appendix C — Process next steps

| Step | Owner | Artifact |
|------|-------|----------|
| 1 | PM / PE | Review this Draft PRD |
| 2 | PE | `/validate-requirements` on Draft |
| 3 | PE / sponsor | **Joint Gate 1** — both Draft PRDs + enum + wave lane |
| 4 | PE | `/prd-impact-map` on each INIT |
| 5 | PE | Implement on prayog-skills **rc-1** **(after step 3)** |
| 6 | PE | Merge rc-1 → release tag; update meta harness pin |
| 7 | PE | Gateflow spec PR consumes `dispatch` (paired with steps 5–6) |
