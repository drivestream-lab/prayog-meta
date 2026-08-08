# Resolution Summary

**Report:** `prd/reports/Validation-Report-INIT-GATEFLOW-013.md` (report_revision 1)
**Initiative:** INIT-GATEFLOW-013
**Reviewed on:** 2026-08-08
**resolution_revision:** 1
**Findings reviewed:** 7 of 7

## Decisions

| CHG | VF | Severity | Target | Chosen option | Rationale | Action to apply |
|-----|-----|----------|--------|---------------|-----------|-----------------|
| CHG-01 | VF-01 | Critical | REQ-11 | A | D3 is about Launchpad being inspect-only, an unrelated capability; A3 is the actual assumption behind reusing the credential probe | Change REQ-11's Outline column from `D3` to `A3` |
| CHG-02 | VF-02 | Should Fix | Appendix A | A | "Loosen or remove" is ambiguous and could be misread as still accepting a value; must match REQ-12's explicit reject-if-provided rule | Reword Appendix A's first row so it cannot be read as "make optional and silently accept" — state the field is removed, or any non-empty value is rejected |
| CHG-03 | VF-03 | Should Fix | §5 Technical risks | A | The risk row asserts an outcome as settled fact when `OQ-9` explicitly leaves it open | Reword the risk row to name the ambiguity itself (undecided whether a fresh check is possible for legacy repos) rather than asserting one unresolved outcome |
| CHG-04 | VF-04 | Should Fix | Non-Goals / Dependencies | A | INIT-GATEFLOW-012's flagged coordination note about INIT-GATEFLOW-004 is now stale since CAP-05/CAP-06 here supersede the capability that note referred to | Add a row noting CAP-05/CAP-06 are the natural successor to INIT-GATEFLOW-012's superseded harness check for any future INIT-GATEFLOW-004 reconciliation, still unresolved |
| CHG-05 | VF-05 | Gap | CAP-03 (US-3) | add-requirement | Deselection is a real, implied-possible operation (D4: selection "can be changed") with no defined behavior anywhere | Add an AC/REQ covering what happens to a deselected repo's local setup, stored readiness answer, and any in-flight wave |
| CHG-06 | VF-06 | Gap | REQ-17, REQ-20 | add-requirement | Both requirements depend on Assumption A2 but don't say so inline, hiding the dependency from a §3-only read | Add "(depends on A2 / `OQ-2`)" inline to REQ-17 and REQ-20 |
| CHG-07 | VF-07 | Gap | CAP-01 | add-requirement | Tenant↔programme cardinality is a real, observable product decision currently stated only as a non-normative Appendix aside | Add a REQ under CAP-01 stating the cardinality explicitly (one programme connection per tenant) |

## Approved Fixes (ready to apply)

| CHG | VF | Type | Location | Original Finding | Decision | Action |
|-----|-----|------|----------|------------------|----------|--------|
| CHG-01 | VF-01 | Structural | §3, REQ-11 row | Wrong cross-reference (`D3` instead of `A3`) | Option A | Change `D3` → `A3` |
| CHG-02 | VF-02 | Structural | Appendix A, row 1 | Ambiguous "loosen or remove" contradicts REQ-12 | Option A | Reword to match REQ-12's reject-if-provided rule |
| CHG-03 | VF-03 | Semantic | §5 Technical risks, row 4 | Asserts an OQ-9-open outcome as fact | Option A | Reword to name the ambiguity, not a settled outcome |
| CHG-04 | VF-04 | Semantic | Whole document | No mention of INIT-GATEFLOW-004; stale coordination note from 012 | Option A | Add a Non-Goals/Dependencies row on the CAP-05/CAP-06 ↔ 004 reconciliation |
| CHG-05 | VF-05 | Semantic | US-3; Error table | Deselection behavior unaddressed | add-requirement | Add AC + REQ for deselection behavior |
| CHG-06 | VF-06 | Semantic | REQ-17, REQ-20 | A2 dependency not inline-cited | add-requirement | Add inline "(depends on A2 / `OQ-2`)" to both rows |
| CHG-07 | VF-07 | Structural | Appendix A, row 2; CAP-01 | Cardinality only in a non-normative aside | add-requirement | Add a normative REQ under CAP-01 for one-programme-per-tenant |

## Confirmed Items (add source tags)

*(None — no Verify findings this round.)*

## Rejected Items (remove or rewrite)

*(None.)*

## Added as Open Questions

*(None — all findings resulted in a concrete fix or new requirement, not a deferral to OQ.)*

## Skipped (no action)

*(None — all 7 findings were reviewed and approved.)*

## Modified / custom recommendations

*(None — every decision matched the recommended option; no custom text was supplied.)*

---

## Summary

- **Findings reviewed:** 7 of 7
- **Approved (Option A / add-requirement):** 7
- **Confirmed:** 0
- **Rejected:** 0
- **Skipped:** 0
- **Added as OQ:** 0

All seven approved fixes are additive or corrective wording changes — none require renumbering existing REQs (CHG-05 and CHG-07 add new REQs; exact numbering, e.g. appending after REQ-25, is left to `update-documents`).

---

```yaml
handoff:
  contract: sdd-delivery/v2
  stage: review-findings
  outcome: pass
  artifact:
    path: prd/reports/Resolution-INIT-GATEFLOW-013.md
    digest: sha256:48e6c5b35510fa54a95ccf3830815c80c889d5cd1b8c03c7339dd50c452ff414
  blockers: []
  signals:
    findings_reviewed: 7
    approved: 7
    confirmed: 0
    rejected: 0
    skipped: 0
    added_as_oq: 0
  next_candidates:
    - validate-requirements
  human_checkpoint: false
  external_action: false
```
