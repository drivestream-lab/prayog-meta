# Resolution Summary

**Report:** `prd/reports/Validation-Report-INIT-GATEFLOW-015.md` (report_revision 1)
**Initiative:** INIT-GATEFLOW-015
**Reviewed on:** 2026-08-11
**resolution_revision:** 1
**Findings reviewed:** 5 of 5

## Decisions

| CHG | VF | Severity | Target | Chosen option | Rationale | Action to apply |
|---|---|---|---|---|---|---|
| CHG-01 | VF-01 | Should Fix | REQ-04, REQ-11 | A | The Predecessor line should attribute exactly what each source promised — INIT-001/FR-10 for the `workflow_node` breakdown, current code for the additional `runner`/`model_id` split — rather than implying both came from the same PRD commitment | Reword the "Predecessor" line (in both `INIT-GATEFLOW-015-outline.md` and `INIT-GATEFLOW-015.md`) to: attribute the `workflow_node` p50/p95 breakdown and 90-day retention to INIT-GATEFLOW-001/FR-10 specifically; attribute the additional `runner`/`model_id` breakdown to "the current implementation" (citing `aggregate_run_metrics`), not to INIT-001's promise |
| CHG-02 | VF-02 | Should Fix | REQ-13 | A | Avoid stating an extrapolated design principle as if it were a documented, system-wide policy; scope the justification to the actual source | Reword Assumption A4 and the OQ-2 resolution text to say the `stop_reason` free-text design is "modeled on `WorkflowEngine.resolve_next`'s specific no-allowlist approach to node resolution," not "the pin's existing 'no allowlists' philosophy" |
| CHG-03 | VF-03 | Should Fix | CAP-03 (REQ-11–REQ-17) | A | Naming the dependency relationship to INIT-GATEFLOW-004 turns two loosely related PRDs into a coherently sequenced pair and prevents "gate dwell time" / "human-wait time" from reading as two different, uncoordinated metrics | Add to the Draft PRD (Document control "Related, not blocking" line, and §4 Integration Points): "CAP-03 fulfills INIT-GATEFLOW-004's REQ-37/A2 aggregate dependency, marked `[TBD in spec]` in that PRD." Add a one-line terminology map: "gate dwell time (this INIT) = human-wait time (INIT-GATEFLOW-004)." Apply the same addition to the outline's "Related, not blocking" line |
| CHG-04 | VF-05 | Verify | REQ-14, REQ-15 | confirm | Accepted as a reasonable judgment call at current dogfood scale; not worth blocking on historical concurrency data collection for this INIT | Add `(Source: User-confirmed)` tag to Assumption A3's "rare enough at current scale" claim |
| CHG-05 | VF-04 | Gap | REQ-04, REQ-07, REQ-10, REQ-17, REQ-23 | add-requirement | Negative-path completeness — these are real, distinct failure modes not covered by the existing 6 rows | Add two rows to the §2 Error table: "Malformed/unknown filter value on CAP-02" → empty/named-clean response, not an error; "Valid tenant with no data yet on any of the three endpoints" → well-formed empty/zero response, not a 404/error |

## Approved Fixes (ready to apply)

| CHG | VF | Type | Location | Original Finding | Decision | Action |
|---|---|---|---|---|---|---|
| CHG-01 | VF-01 | Semantic | Document control / Executive Summary "Predecessor" line | Over-attributes runner/model_id breakdown to INIT-GATEFLOW-001/FR-10 | Option A | Split attribution: workflow_node → INIT-001/FR-10; runner/model_id → current implementation |
| CHG-02 | VF-02 | Semantic | §2 OQ-2 resolution; §6 Locked decisions; Assumption A4 | Over-generalizes `resolve_next`'s no-allowlist behavior into a system-wide philosophy | Option A | Scope wording to `WorkflowEngine.resolve_next` specifically |
| CHG-03 | VF-03 | Semantic | "Related, not blocking" line; §4 Integration Points | Understates relationship to INIT-GATEFLOW-004 REQ-37/A2; no terminology map | Option A | Add fulfills-dependency cross-reference + terminology map |
| CHG-05 | VF-04 | Semantic | §2 Error table | Missing 2 negative-path rows | add-requirement | Add malformed-filter and empty-tenant rows |

## Confirmed Items (add source tags)

| CHG | VF | Type | Location | Finding | Action |
|---|---|---|---|---|---|
| CHG-04 | VF-05 | Semantic | §2 Assumptions, A3 | "Rare enough at current scale" is unsourced | Add `(Source: User-confirmed)` tag |

## Rejected Items (remove or rewrite)

*None.*

## Added as Open Questions

*None.*

## Skipped (no action)

*None.*

## Modified / custom recommendations

*None — all decisions used the recommended Option A / confirm / add-requirement path.*

---

```yaml
handoff:
  contract: sdd-delivery/v2
  stage: review-findings
  outcome: pass
  artifact:
    path: prd/reports/Resolution-INIT-GATEFLOW-015.md
  blockers: []
  signals:
    findings_reviewed: 5
    approved: 4
    confirmed: 1
    rejected: 0
    skipped: 0
    added_as_oq: 0
  next_candidates:
    - update-documents
  human_checkpoint: false
  external_action: false
```
