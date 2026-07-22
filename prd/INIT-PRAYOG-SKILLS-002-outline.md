# INIT-PRAYOG-SKILLS-002 — Workflow dispatch policy (outline)

**Status:** outline · **Author:** programme PM · **Date:** 2026-07-22  
**Related:** [INIT-GATEFLOW-001-outline](./INIT-GATEFLOW-001-outline.md) · [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md)  
**Component:** PRAYOG-SKILLS · **Type:** platform / delivery contract

> **Outline only.** Sections marked `[TBD]` expand in Draft PR after PE review.
> Engineering detail routes to prayog-skills rc-2 branch and spec PR on
> `drivestream-lab/prayog-skills`. **Paired with INIT-GATEFLOW-001; Joint Gate 1
> required before rc-2 merge, pin, or integration test.**

---

## Document control

| Field | Value |
|-------|-------|
| Initiative ID | INIT-PRAYOG-SKILLS-002 |
| Artifact | `prd/INIT-PRAYOG-SKILLS-002-outline.md` (outline); Draft PRD `[TBD]` |
| Programme | prayog |
| Primary repo | drivestream-lab/prayog-skills |
| Target branch | **rc-2** (feature); release tag TBD after Gate 1 |
| Target users | PE, orchestrator consumers (Gateflow), skills maintainers |
| Paired initiative | [INIT-GATEFLOW-001](./INIT-GATEFLOW-001-outline.md) — consumes `dispatch` |
| **Gate 1 coupling** | **Joint Gate 1** with INIT-GATEFLOW-001 — required before rc-2 merge, pin, or integration test |

---

## 1. Problem statement

`sdd-delivery/v2` defines **navigation** via `workflow.yaml` node `type`
(`skill`, `human-checkpoint`, `external-action`, …). That tells consumers where
to **stop**, but not which `skill` nodes an **orchestrator may dispatch** vs
which remain **PE/agent manual**.

[INIT-GATEFLOW-001](./INIT-GATEFLOW-001-outline.md) must not hardcode node id
lists (e.g. “wave lane = pre-implement … ground-spec”). Automation eligibility
belongs in **prayog-skills SSOT**, not Gateflow code or duplicate programme
config.

---

## 2. Proposed solution (summary)

Add a **`dispatch`** field on every **`type: skill`** node in `workflow.yaml`:

| Value | Meaning |
|-------|---------|
| `manual` | PE or agent runs the skill; orchestrator **must not** dispatch |
| `orchestrated` | Orchestrator **may** dispatch when programme trigger + handoff authorize |
| `observed` | Orchestrator **must not** dispatch; **must** record timing from handoffs (metrics-only) `[TBD — include in v1 or defer]` |

Non-`skill` nodes (`human-checkpoint`, `external-action`, `decision`, `terminal`)
do **not** carry `dispatch` — behavior remains **`type`-only**.

Document semantics in `delivery-contract.yaml` and validate in
`tests/test_workflow_contract.py`. Ship on **prayog-skills rc-2**; programme
pins rc-2 when stable for Gateflow dogfood.

---

## 3. Scope — in (rc-2)

### 3.1 Capabilities

| ID | Capability | Notes |
|----|------------|-------|
| FR-1 | Every `type: skill` node has required `dispatch` enum | Contract test |
| FR-2 | Annotate full `workflow.yaml` graph per policy below | No partial graph |
| FR-3 | Document `dispatch` in delivery contract + handoff/navigation refs | Consumer-facing |
| FR-4 | Extend `test_workflow_contract.py` for enum + presence | CI gate |
| FR-5 | CHANGELOG + rc-2 release notes | Pin guidance for meta/gateflow |
| FR-6 | Orchestrator-neutral naming (`orchestrated`, not `gateflow`) | Any consumer |

### 3.2 Illustrative annotation (rc-2 default policy)

Meta / spec upstream — **`manual`**:

```text
validate-requirements, review-findings, update-documents, prd-impact-map
spec-draft, initiative-feasibility, spec-technical-review, spec-implementation-plan
board-seed
```

Wave execution lane — **`orchestrated`**:

```text
pre-implement, loop-spec, verify, ground-spec
```

Policy changes in **prayog-skills only** — Gateflow reads pin; no node lists in
orchestrator.

### 3.3 Consumer dispatch algorithm (normative)

For any orchestrator (Gateflow, future):

```text
next = resolve(handoff.stage, handoff.outcome)

if next.type != skill:
    STOP   # human-checkpoint, external-action, decision, terminal

if next.dispatch != orchestrated:
    STOP or observe-only (manual / observed)

if not programme_trigger_authorized:
    STOP

dispatch skill(next)
```

---

## 4. Scope — out (explicit non-goals)

| Non-goal | Rationale |
|----------|-----------|
| Gateflow implementation | INIT-GATEFLOW-001 |
| Change skill procedures / SKILL.md bodies | Unless handoff template note only |
| Auto-merge or gate label semantics | Unchanged; `external-action` / human gates |
| Graphify / ToolProvider | Gateflow + separate skills initiative |
| Rename or restructure workflow graph | Additive field only |

---

## 5. Contract and version strategy

| Item | Lean |
|------|------|
| Contract id | Stay `sdd-delivery/v2` with **rc-2 additive** semantics `[TBD: v2.1 doc bump]` |
| Backward compat | v0.4.3 pins lack `dispatch`; consumers apply **schema default** missing field → `manual` — no node allowlist fallback |
| Handoff envelope | No required new fields in v1; optional future `executed_by: manual \| orchestrated` |

---

## 6. Dependencies and pairing with INIT-GATEFLOW-001

### 6.1 Joint Gate 1 (blocking — both INITs together)

**INIT-PRAYOG-SKILLS-002 and INIT-GATEFLOW-001 must pass a single Joint Gate 1**
before any integration work or dogfood that depends on `dispatch`. Neither INIT
proceeds alone past outline → Draft PRD without the other present at Gate 1.

| Blocked until Joint Gate 1 | Rationale |
|----------------------------|-----------|
| Merge `dispatch` to prayog-skills **rc-2** / tag | Consumer semantics must match Gateflow FR-5 |
| Meta harness pin to rc-2 | Pin without aligned Gateflow policy is untestable |
| Gateflow spec PR / W0 that reads `dispatch` | PolicyEngine must match rc-2 annotation |
| Dogfood Phase B (label-triggered wave runs) | End-to-end contract must be Gate 1–approved |

**May proceed in parallel before Joint Gate 1:** outline review, Draft PRD
expansion, `/validate-requirements` on each Draft, impact-map scoping.

**Joint Gate 1 confirms (single session):** `dispatch` enum (`manual` /
`orchestrated` / `observed` decision), full wave-lane annotation, normative
consumer algorithm (§3.3), rc-2 pin timing, v0.4.3 schema-default bounds
(missing → `manual` only — no orchestrator node allowlists).

### 6.2 Technical pairing

| Dependency | Assumption |
|------------|------------|
| INIT-GATEFLOW-001 outline | Gateflow FR-5 consumes `dispatch`; no hardcoded skill lists |
| rc-2 branch | Active development on prayog-skills |
| Programme pin | `.harness-pin.yaml` moves to rc-2 tag **after Joint Gate 1** |
| Gateflow dogfood | Requires rc-2 pin; until then consumers use schema default (missing `dispatch` → `manual`) only |

```text
INIT-PRAYOG-SKILLS-002 (rc-2)     INIT-GATEFLOW-001
  workflow.yaml + dispatch    →     PolicyEngine reads dispatch
  contract docs + tests             RunStore + AgentRunner
         └──── Joint Gate 1 (both INITs) → pin rc-2 → dogfood ────┘
```

---

## 7. Success criteria (rc-2)

| Criterion | Target |
|-----------|--------|
| Contract tests green | All skills have valid `dispatch` |
| Wave lane | `pre-implement` … `ground-spec` = `orchestrated` |
| Upstream | All other skills = `manual` (or `observed` if adopted) |
| Gateflow spec | Can implement dispatch check without node id constants |
| Pin record | Meta harness pin documents rc-2 ref |

---

## 8. Risks `[TBD — expand in Draft]`

| Risk | Mitigation |
|------|------------|
| rc-2 delays Gateflow | Consumers use schema default (missing `dispatch` → `manual`); **no** programme-config node allowlist in Gateflow |
| Wrong `dispatch` on node | PE review of workflow diff; contract tests |
| Enum proliferation | Start with `manual` + `orchestrated`; add `observed` only if needed |

---

## 9. Open questions (Joint Gate 1)

Resolved at **Joint Gate 1 with INIT-GATEFLOW-001** (single session — do not
implement rc-2 or Gateflow `dispatch` consumption until cleared):

1. Include `observed` in v1 or defer (manual covers metrics-only for now)?
2. Contract doc bump: rc-2 changelog only vs explicit `sdd-delivery/v2.1`?
3. Single rc-2 tag name (`v0.5.0-rc.2` vs `v0.4.4-rc.2`)?
4. rc-2 pin timing relative to Gateflow W0 merge (same sprint vs pin-first)?

---

## 10. Next steps (process)

| Step | Owner | Artifact |
|------|-------|----------|
| 1 | PM / PE | Review outlines → Draft PRDs (parallel with Gateflow) |
| 2 | PE | **`/validate-requirements` on each Draft** (parallel ok) |
| 3 | PE / sponsor | **Joint Gate 1** — both Draft PRDs + enum + wave lane **(blocking)** |
| 4 | PE | Implement on prayog-skills **rc-2** branch **(after step 3)** |
| 5 | PE | Merge rc-2 → tag; update meta harness pin |
| 6 | PE | Gateflow spec PR consumes `dispatch` (paired with step 4–5) |

---

## Appendix A — Outline → full PRD checklist

- [ ] Per-FR acceptance criteria
- [ ] Full node annotation table (every skill id + dispatch value)
- [ ] Migration note for v0.4.3 → rc-2 pins
- [ ] Handoff spec cross-reference paragraph

---

## Appendix B — Relationship to INIT-GATEFLOW-001

| INIT-PRAYOG-SKILLS-002 | INIT-GATEFLOW-001 |
|------------------------|-------------------|
| Defines `dispatch` on workflow nodes | Reads `dispatch`; FR-5 |
| Process / automation SSOT | Execution control plane |
| rc-2 release | Pins rc-2 for dogfood |
| Contract tests | PolicyEngine implementation |
| **Joint Gate 1 (blocking)** | **Joint Gate 1 (blocking)** |
