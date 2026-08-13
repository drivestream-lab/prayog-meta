# Resolution Summary

**Report:** `prd/reports/Validation-Report-INIT-GATEFLOW-017.md`  
**Initiative:** INIT-GATEFLOW-017  
**Reviewed on:** 2026-08-13  
**resolution_revision:** 2  
**previous_revision:** 1 (2026-08-13) — CHG-01–CHG-14 already applied against `prd/INIT-GATEFLOW-017.md`  
**Findings reviewed:** 1 of 1 (open findings from Validation-Report revision 2)

## Decisions

| CHG | VF | Severity | Target | Chosen option | Rationale | Action to apply |
|-----|-----|----------|--------|---------------|-----------|-----------------|
| CHG-15 | VF-15 | Gap | REQ-30 | add-requirement | REQ-30 exists; journeys/§7 are the QA checklist | J8: after password set, factory list / membership views do not show the password (REQ-30). J10: identities on a programme and programmes for an identity are shown without passwords. §7 row REQ-30: list/search/membership returns or displays a password → not shown / 0 leak. |

## Approved Fixes (ready to apply)

| CHG | VF | Type | Location | Original Finding | Decision | Action |
|-----|-----|------|----------|------------------|----------|--------|
| CHG-15 | VF-15 | Semantic | §12 J8, J10; §7 | REQ-30 missing from journeys and §7 | add-requirement | Add J8/J10 inspection edges and a §7 row as in Decisions |

## Confirmed Items (add source tags)

*None.*

## Rejected Items (remove or rewrite)

*None.*

## Added as Open Questions

*None.*

## Skipped (no action)

*None — the one open finding was decided.*

## Modified / custom recommendations

*None — listed add-requirement option used verbatim (J8 + J10 + §7).*

---

```yaml
handoff:
  contract: sdd-delivery/v2
  stage: review-findings
  outcome: pass
  artifact:
    path: prd/reports/Resolution-INIT-GATEFLOW-017.md
  blockers: []
  signals:
    previous_resolution_revision: 1
    resolution_revision: 2
    approved: 1
    skipped: 0
    chg_count: 1
    chg_ids: CHG-15
  next_candidates:
    - update-documents
  human_checkpoint: false
  external_action: false
```
