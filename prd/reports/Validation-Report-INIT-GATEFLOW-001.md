# Requirements Review

**Document:** prd/INIT-GATEFLOW-001.md  
**Validated on:** 2026-07-22  
**Sources checked:** 8 source documents  
**Checks run:** 15 (11 semantic + 4 structural)

### Source index

| ID | Document | Role |
|----|----------|------|
| SRC-1 | prd/INIT-GATEFLOW-001-outline.md | Parent outline / scope SSOT |
| SRC-2 | planning/gateflow-programme-vision.md | Programme vision |
| SRC-3 | prd/INIT-PRAYOG-SKILLS-002-outline.md | Paired `dispatch` contract |
| SRC-4 | prayog-skills/workflow.yaml (pin v0.4.3) | Workflow navigation SSOT |
| SRC-5 | prayog-skills/delivery-contract.yaml | Contract + github.labels |
| SRC-6 | prayog-skills/references/handoff-envelope.md | Handoff + navigation rules |
| SRC-7 | prayog-skills/tests/fixtures/workflow_scenarios.json | Resolver test fixtures |
| SRC-8 | .harness-pin.yaml | Current skills pin (v0.4.3) |

**Stage artifacts:** Stage4 Scenario Matrix — unavailable. Stage6 User Flows — unavailable.

### Semantic Checks (content accuracy)

| Check | Findings | Status |
|-------|----------|--------|
| 1. Source Accuracy | 1 | FAIL |
| 2. Inference Detection | 2 | FAIL |
| 3. Requirement Purity | 1 | FAIL |
| 4. Over-Generalization | 0 | PASS |
| 5. Scope Boundary | 1 | FAIL |
| 6. Testability | 3 | FAIL |
| 7. Ambiguity | 2 | FAIL |
| 8. Assumption-Req Dependency | 2 | FAIL |
| 9. Negative Path Coverage | 1 | FAIL |
| 10. Actor Capability | 1 | FAIL |
| 11. Intra-Document Consistency | 4 | FAIL |

### Structural Checks (document integrity)

| Check | Findings | Status |
|-------|----------|--------|
| S1. Staleness | 0 | PASS |
| S2. Contradictions | 2 | FAIL |
| S3. Cross-References | 2 | FAIL |
| S4. Completeness | 3 | FAIL |

**Findings total:** 25 (0 Critical, 12 Should Fix, 5 Verify, 8 Gaps)

---

## Critical (MUST FIX — factually wrong or misleading)

*None.*

---

## Should Fix (reframe, relocate, or make precise)

| # | Type | Location | Check | Finding | Recommendation |
|---|------|----------|-------|---------|----------------|
| VF-001 | Structural | §4 Integration Points, L275 | 11, S2 | Lists **gateflow-ops** as active integration (`BFF → gateflow status JSON API`) while FR-12, §4 Repositories, and W1 exit criteria state gateflow-ops is **out of W1 scope**. | Remove or mark gateflow-ops integration as **W2 deferred**; W1 integration is gateflow status JSON API only. |
| VF-002 | Structural | §2 FR-10, L143 | S3 | Acceptance criteria reference **"§5.4 in outline"** — external cross-doc pointer; PRD §4 Metrics Schema is partial. | Inline full metrics dimensions in FR-10 AC or cite **§4 Metrics Schema** within this PRD. |
| VF-003 | Semantic | §5 W1 exit + §5 Dependencies, L343–386 | 11, S2 | **W1 exit criterion 3** requires rc-2 pin + end-to-end dispatch, but Joint Gate 1 blocks rc-2 until PRD cleared; Phase B requires W1 exit. Sequencing is implicit, not normative. | Add explicit **delivery sequence**: Joint Gate 1 → rc-2 pin → complete W1 criteria 3–11 → W1 exit → Phase B. |
| VF-004 | Semantic | §3 Evaluation Strategy, L214 | 6 | **"Same quality bar as manual SDD waves (qualitative Gate 1+ review)"** is not objectively testable. | Replace with concrete acceptance checks (e.g. wave artifacts present, harness green, PE sign-off checklist at `wave-human-decision`). |
| VF-005 | Semantic | §2 FR-3, FR-6, FR-14; §4 Architecture | 3 | Multiple FRs prescribe **PostgreSQL, Cursor SDK, Docker, GitHub App** — solution prescriptions. Acceptable for platform INIT but blurs requirement vs implementation. | Label as **Implementation constraints (H1)** or move to Technical Specifications with FRs reframed as capabilities ("durable run store", "coding agent adapter"). |
| VF-006 | Structural | §5 Open Questions, L418–425 | S3, S4 | Open Questions numbered **1, 2, 5, 6, 7, 8** — gaps at 3–4 because they moved to Decisions table without renumbering. | Renumber Open Questions 1–6 or add note: "3–4 resolved — see Decisions table." |
| VF-007 | Semantic | §3 Dispatch Preconditions, L199–200 | 5, 11 | Algorithm adds `handoff.human_checkpoint` and `handoff.contract != installed_contract` checks **not present** in INIT-PRAYOG-SKILLS-002 §3.3 consumer algorithm. | Align paired INIT or cite **handoff-envelope.md rules 2, 5, 8** explicitly as normative extensions in both PRDs. |
| VF-008 | Semantic | §3 Evaluation Strategy, L211 | 1, 6 | Claims **100% match with all scenarios in `workflow_scenarios.json`** — file has 10 scenarios and **does not cover** dispatch policy, loop-spec self-`findings`, or AgentRunner paths. | Scope test claim to navigation-only fixtures today; add Gateflow-specific fixture requirement for rc-2 dispatch scenarios. |
| VF-009 | Semantic | Document control, L23 | S2 | Says blocked before **"W0 `dispatch` logic"** but W0 wave table explicitly ships **resolver without dispatch**; dispatch is W1. | Change to **"W1 PolicyEngine reading `dispatch`"**. |
| VF-010 | Semantic | §2 US-4 vs FR-10, L118 vs L143 | 11 | US-4 requires **export query (SQL or API)** for p50/p95 aggregates; FR-10 does not specify export surface. | Add FR or AC for metrics query/export (SQL view or `/metrics` endpoint). |
| VF-011 | Semantic | §4 Workflow Navigation, L261–263 | 4 | Illustrative rc-2 node list (`pre-implement` … `ground-spec`) in Gateflow PRD — correct SSOT location is prayog-skills, but could be read as Gateflow policy. | Prefix with **"Illustrative only — SSOT in prayog-skills rc-2"** (stronger than current one line). |
| VF-012 | Semantic | §2 US-2, L96 | 6 | **"Structured comment"** on stage start/end — no schema defined (fields, format, idempotency on redelivery). | Add comment template spec or reference Notifier event schema in FR-9 AC. |

---

## Verify (needs user confirmation)

| # | Type | Location | Check | Finding | Question for User |
|---|------|----------|-------|---------|-------------------|
| VF-013 | Semantic | §2 US-1, L86 | 2, 6 | **RunStore record within 30s of webhook delivery** — not stated in outline or vision. | Confirm 30s SLA or mark `[TBD]` / remove numeric target. |
| VF-014 | Semantic | §1 Success Criteria, L58 | 2, 6 | **Reconstruct timeline within 5 minutes** — unsourced numeric target. | Confirm 5-minute target or soften to qualitative drill-down test. |
| VF-015 | Semantic | §3 Evaluation, L211 | 1 | **`workflow_scenarios.json`** lives in prayog-skills repo; Gateflow resolver tests may need **local copy or pin-time fixture import**. | Confirm Gateflow CI pulls fixtures from pinned prayog-skills ref. |
| VF-016 | Semantic | §2 Tech lead persona, US-3 | 10 | Tech lead audit assumes access to **ForgeClient audit log** and RunStore — not confirmed as programme capability. | Confirm tech leads get read access or restrict audit to PE/sponsor. |
| VF-017 | Semantic | §4 Programme Config keys, L280–287 | 2 | Config key names (`trigger.label`, `retry.findings_budget`, …) are **proposed** — no source in meta harness YAML yet. | Confirm keys will be defined in harness/programme config schema before spec PR. |

---

## Gaps (missing coverage)

| # | Type | Location | Check | Finding | Suggested Addition |
|---|------|----------|-------|---------|-------------------|
| VF-018 | Semantic | §2 (missing section) | 9, S4 | **No Error Handling section** — failure modes scattered across FRs only. | Add §2.x Error Handling table: webhook failure, handoff parse error, AgentRunner timeout/crash, ForgeClient 5xx, Postgres unavailable, mid-run cancellation. |
| VF-019 | Semantic | §2 US-1, FR-2 | 9 | **Label trigger preconditions undefined** — "handoff preconditions satisfied" not specified (e.g. minimum `handoff.stage`, board-seed complete, no blockers). | Define authoritative precondition checklist for wave run authorization. |
| VF-020 | Semantic | §2 FR-6 / AgentRunner | 9 | **AgentRunner failure path missing** — no requirement for timeout, non-zero exit, or partial artifact on agent crash. | Add FR/AC: failed dispatch stops run, records `outcome: failed`, notifies PE, does not advance workflow. |
| VF-021 | Semantic | §4 ForgeClient / §2 FR-14 | 9 | **GitHub API failure handling** — no retry/backoff or run-state behavior when ForgeClient cannot comment. | Add negative path: queue/retry comments or mark run `notify_pending` in RunStore. |
| VF-022 | Semantic | §5 Dependencies | 8, S4 | **No formal Assumptions table** with confirmation status — rc-2, Cursor container, GitHub App install are implicit. | Add Assumptions table (ID, assumption, status, dependent FRs). |
| VF-023 | Semantic | §2 FR-8 | 9 | **`type: decision` and `type: terminal` nodes** listed in dispatch algorithm but FR-8 AC only mentions human-checkpoint and external-action explicitly. | Extend FR-8 AC to cover `decision` and `terminal` stop behavior. |
| VF-024 | Semantic | §5 W1 exit criterion 11 | 9 | **Future-wave ready** requires documented steps but no FR owns documentation deliverable. | Add FR or W1 deliverable: "Runbook: orchestrate new initiative repo" in gateflow docs. |
| VF-025 | Semantic | §2 FR-1 | 9 | **Concurrent label triggers** on same PR — idempotency for events covered but not concurrent run policy. | Define: reject second run if active run exists, or queue, or supersede — PE decision. |

---

## Clean (no issues found)

| Check | Verified |
|-------|----------|
| **4. Over-Generalization** | No `(Source: SRC-N)` citations to mis-scope; illustrative rc-2 policy correctly attributed to prayog-skills. |
| **S1. Staleness** | `[TBD]` markers are intentional open decisions; resolved items 3–4 correctly recorded in Decisions table (renumbering issue tracked separately). |
| **Partial — Non-goals** | Non-goals align with outline and vision; hardcoding prohibitions consistent with pluggability rule. |
| **Partial — Joint Gate 1** | Blocking dependency with INIT-PRAYOG-SKILLS-002 consistently stated across PRD §5 and outline. |
| **Partial — Retry exhaustion** | US-5, FR-7, and Decisions table consistently specify stop + GitHub comment only. |
| **Partial — W1 scope** | gateflow-only W1 delivery consistent across Phase A/B, impact map note, and Decisions #4 (gateflow-ops integration row excepted — VF-001). |

---

## Recommended Next Steps

1. **Review findings interactively** — use `review-findings` with this report to walk through VF-001–VF-025 and collect decisions.
2. **Or resolve manually** — priority order:
   - **Should Fix:** VF-001, VF-003, VF-009 (scope/sequencing clarity)
   - **Gaps:** VF-018, VF-019, VF-020 (error paths + trigger preconditions)
   - **Verify:** VF-013, VF-014 (numeric SLAs)
   - Remaining Should Fix / Gaps
3. **Apply fixes** — use `update-documents` after review-findings resolution, or edit Draft PRD directly.
4. **Re-run validation** — incremental mode with this report as prior after fixes.
5. **Joint Gate 1** — proceed only after Critical = 0 and blocking Should Fix / Gaps addressed (VF-003 sequencing, VF-019 preconditions recommended).

---

## Handoff

```yaml
handoff:
  contract: sdd-delivery/v2
  stage: validate-requirements
  outcome: findings
  artifact:
    path: prd/reports/Validation-Report-INIT-GATEFLOW-001.md
    digest: sha256:d9027ae7f17d22f8fcf877b906fafd3514636180e8a76ae078bc2b8f5c2e62b1
  blockers:
    - VF-001
    - VF-003
    - VF-018
    - VF-019
    - VF-020
  signals:
    findings_total: 25
    critical: 0
    should_fix: 12
    verify: 5
    gaps: 8
  next_candidates:
    - review-findings
    - update-documents
  human_checkpoint: false
  external_action: false
```
