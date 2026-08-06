# Resolution Summary

**Report:** `prd/reports/Validation-Report-INIT-GATEFLOW-010.md`  
**Initiative:** INIT-GATEFLOW-010  
**Reviewed on:** 2026-08-05  
**resolution_revision:** 1  
**Findings reviewed:** 12 of 12  

## Decisions

| CHG | VF | Severity | Target | Chosen option | Rationale | Action to apply |
|-----|-----|----------|--------|---------------|-----------|-----------------|
| CHG-01 | VF-01 | Should Fix | REQ-02, REQ-03 | A | Keep REQs outcome-based | Normative REQ text = observable outcomes; move `ForgeActionType` / `BoardService` / `NodeForgePolicy` to Implementation Note or §4 |
| CHG-02 | VF-02 | Should Fix | §1 Problem Statement | A | Match real gap | Reword: implement-start does not yet **board-resolve** tickets / fail closed on missing board issues (non-empty string alone is insufficient) |
| CHG-03 | VF-03 | Should Fix | REQ-08, REQ-13, error table | A | Testable HTTP/Done | Use **400** for missing/malformed input; **422** for resolve / Done-gate failures; Done = pin `status: done` via `update_board_status` / BoardService column+state contract |
| CHG-04 | VF-04 | Should Fix | REQ-02, W0 | A | Avoid brittle count | Replace `39/39` with “all nodes in remounted `workflow.yaml` parse” (optional tip note: currently 39 on tip SHA …) |
| CHG-05 | VF-05 | Should Fix | US-6 | A | Traceability | Add **REQ-19**: after `wave-signoff` / `wave-complete`, no auto-start next wave or closure; cite from US-6 and exit wave |
| CHG-06 | VF-06 | Should Fix | Proposed Solution / §4 | A | Layer purity | Soften Exec Solution to outcomes (“Gateflow applies pin board-status hops…”); keep module names in §4 as design |
| CHG-07 | VF-07 | Verify | §1; REQ-01–18 | confirm | Gaps still accurate | Add `(Source: User-confirmed)` and short **As-built baseline** (date 2026-08-05 + gateflow SHA when recorded) |
| CHG-08 | VF-08 | Verify | REQ-14, US-5 | confirm | Pin vs programme | Keep REQ-14; add callout: “Gateflow programme board hygiene — not a pin external-action node.” |
| CHG-09 | VF-09 | Gap | REQ-14, REQ-15 | add-requirement | Partial-fail safety | Add recovery AC/REQ: if EPIC Done then purge-app or closure PR fails → record failure; do not claim closure complete; PE may re-enter / compensate |
| CHG-10 | VF-10 | Gap | REQ-04, REQ-08 | add-requirement | Retry semantics | Idempotent In Progress if already `in_progress`; **422** if ticket already Done |
| CHG-11 | VF-11 | Gap | Document structure | add-requirement | Explicit deps | Add Assumptions: A1 Done vocabulary = pin `status: done` / BoardService; A2 PE retains `epic_ticket_id` + `wave_ticket_ids[]`; A3 pin tip frozen for INIT delivery |
| CHG-12 | VF-12 | Gap | CAP-* / OQ-* | add-requirement | Id hygiene | Assign CAP-01 Spec, CAP-02 Tickets, CAP-03 Implement, CAP-04 Closeout, CAP-05 Closure, CAP-06 Pin fidelity; map REQs; add **OQ-01** problem+json / OpenAPI error field names |

## Approved Fixes (ready to apply)

| CHG | VF | Type | Location | Original Finding | Decision | Action |
|-----|-----|------|----------|------------------|----------|--------|
| CHG-01 | VF-01 | Semantic | §3 | Class names in REQ-02/03 | Option A | Outcomes normative; names → Implementation Note / §4 |
| CHG-02 | VF-02 | Semantic | §1 | Overstated missing-ticket hard-fail | Option A | Reword to board-resolve gap |
| CHG-03 | VF-03 | Semantic | Error table; REQ-08/13 | Ambiguous 4xx / Done | Option A | 400 / 422 + Done = pin `status: done` |
| CHG-04 | VF-04 | Semantic | REQ-02 / W0 | Hard-coded 39/39 | Option A | All remounted nodes parse |
| CHG-05 | VF-05 | Semantic | US-6 | No REQ for wave-complete | Option A | Add REQ-19 + citations |
| CHG-06 | VF-06 | Semantic | Exec / §4 | SOLUTION language in Exec | Option A | Soften Exec; §4 remains design |
| CHG-09 | VF-09 | Semantic | Error / negative paths | No partial-fail after EPIC Done | add-requirement | Recovery AC / REQ language |
| CHG-10 | VF-10 | Semantic | US-3 / errors | No already In Progress/Done AC | add-requirement | Idempotent in_progress; 422 if Done |
| CHG-11 | VF-11 | Semantic | Structure | No Assumptions table | add-requirement | Add A1–A3 |
| CHG-12 | VF-12 | Structural | Document-wide | No CAP-* / OQ-* | add-requirement | CAP-01…06 + OQ-01; map REQs |

## Confirmed Items (add source tags)

| CHG | VF | Type | Location | Finding | Action |
|-----|-----|------|----------|---------|--------|
| CHG-07 | VF-07 | Semantic | §1; REQs | Gap claims unsourced | Add `(Source: User-confirmed)` + As-built baseline (2026-08-05 + gateflow tip SHA) |
| CHG-08 | VF-08 | Semantic | REQ-14 | EPIC Done not a pin node | Keep REQ-14; add programme-hygiene callout (not pin external-action) |

## Rejected Items (remove or rewrite)

_None._

## Added as Open Questions

| CHG | VF | Finding | OQ id | Open Question Text |
|-----|-----|---------|-------|---------------------|
| CHG-12 | VF-12 | Deferred OpenAPI / problem+json field names | OQ-01 | Exact problem+json / OpenAPI error body field names for Gateflow lane APIs (deferred detail — freeze may leave TBD until OpenAPI pass). |

## Skipped (no action)

_None._

## Modified / custom recommendations

_None — all options applied as recommended (VF-03/10 HTTP map and Done-ticket policy as recorded in Actions)._

---

```yaml
handoff:
  contract: sdd-delivery/v2
  stage: review-findings
  outcome: pass
  artifact:
    path: prd/reports/Resolution-INIT-GATEFLOW-010.md
    digest: sha256:80249336f8d18568d75941e0b0794d8df4db41ffaad52c703310d7cbb86d555c
  blockers: []
  signals:
    resolution_revision: 1
    findings_reviewed: 12
    chg_count: 12
    skipped_count: 0
  next_candidates:
    - update-documents
  human_checkpoint: false
  external_action: false
  forge: {}
```
