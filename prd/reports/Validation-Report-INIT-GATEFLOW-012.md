# Requirements Review

**Document:** `prd/INIT-GATEFLOW-012.md`
**Initiative:** INIT-GATEFLOW-012
**Validated on:** 2026-08-07
**report_revision:** 2
**previous_revision:** 1
**Sources checked:** 25 source documents/files (unchanged since revision 1 — all code sources re-read to verify each correction)
**Checks run:** 15 (11 semantic + 4 structural)
**Mode:** Incremental (prior report: this same canonical path, revision 1, 2026-08-07)
**Changes detected:** Document-wide — 1 terminology rename ("Programme" → "Tenant," per Resolution CHG-10) plus 12 targeted edits (Resolution CHG-01–08, CHG-11–13), applied via `update-documents` against `Resolution-INIT-GATEFLOW-012.md`. Given the scale of the rename, every major section changed; sources (code, sibling docs) themselves did not change.
**Checks re-run:** All 15 (1–11, S1–S4) — widespread document changes triggered RE-RUN on every check per the conservative rule; none were provably scoped enough to carry forward or skip.
**Checks carried forward:** None
**Prior findings resolved:** 13 (all of VF-01–VF-13)
**Prior findings carried forward:** 0
**New findings:** 0

### Semantic Checks (content accuracy)

| Check | Findings | Status | Mode |
|-------|----------|--------|------|
| 1. Source Accuracy | 0 | PASS | Re-run |
| 2. Inference Detection | 0 | PASS | Re-run |
| 3. Requirement Purity | 0 | PASS | Re-run |
| 4. Over-Generalization | 0 | PASS | Re-run |
| 5. Scope Boundary | 0 | PASS | Re-run |
| 6. Testability | 0 | PASS | Re-run |
| 7. Ambiguity | 0 | PASS | Re-run |
| 8. Assumption-Req Dependency | 0 | PASS | Re-run |
| 9. Negative Path Coverage | 0 | PASS | Re-run |
| 10. Actor Capability | 0 | PASS | Re-run |
| 11. Intra-Document Consistency | 0 | PASS | Re-run |

### Structural Checks (document integrity)

| Check | Findings | Status | Mode |
|-------|----------|--------|------|
| S1. Staleness | 0 | PASS | Always re-run |
| S2. Contradictions | 0 | PASS | Always re-run |
| S3. Cross-References | 0 | PASS | Always re-run |
| S4. Completeness | 0 | PASS | Always re-run |

---

## Resolved (fixed since prior report)

| VF | Prior Location | Prior Check | Prior Finding | Resolution |
|----|----------------|-------------|----------------|------------|
| VF-01 | Banner blockquote, line 10 | 11, S1, S2 | Banner claimed "G1–G8"; only G1–G5 exist | Banner now reads "G1–G5," matching §3/§6 exactly |
| VF-02 | §4 Architecture Overview; REQ-23 | 1 | `TriggerRouter.authorize_and_check` wrongly named as concurrency enforcer | Architecture Overview and REQ-23/US-9 now correctly name `WaveStartService` → `RunRepository.find_active_run`; a new implementation-notes row explicitly documents that `TriggerRouter` is a separate webhook/legacy-ingress class, not part of this route |
| VF-03 | §4 Architecture Overview; US-9; D10 | 1, 9, 11 | Architecture didn't distinguish sync vs. async execution phase; concurrency guarantee unclear | Architecture Overview now explicitly separates the synchronous `WaveStartService` (pre-enqueue, gate) phase from the asynchronous `RunOrchestrator.process_job` (post-enqueue, CAP-02/03/04) phase; US-9 gained an explicit AC stating the gate always runs first |
| VF-04 | Banner; Problem Statement | 1, 2 | "Gap" framing understated the real `Path.cwd()` fallback | Banner, Problem Statement, and REQ-10 now explicitly name the `Path.cwd()` fallback in `RunOrchestrator.process_job` and state CAP-02 replaces it |
| VF-05 | §2 Open questions, line 404 | 11, S1 | Resolved OQ-3 sat inside "Open questions" table | OQ-3 removed from the table; a "Resolved from outline" note now consolidates OQ-1–OQ-6 resolution mapping |
| VF-06 | Line 133 vs. line 572 | 11, S2 | REQ-30/31 wave attribution contradicted itself (cross-cutting vs. W5-only) | Added a reconciling note in §3 (Capabilities) and §5 (Delivery waves) stating the contract PR lands once, covering all three shapes, with W1/W2 implicitly depending on it |
| VF-07 | Whole document | 5 | No cross-reference to `INIT-GATEFLOW-004`'s overlapping scorecard checks | Added a Non-Goals row, a Technical risk row, and a References entry flagging the unreconciled overlap |
| VF-08 | REQ-10, REQ-15, Appendix A/B | 3 | Exact clone-path formula stated as normative REQ text | REQ-10/REQ-15 now state observable behavior only, with the exact `{workspace_root}/{org}/{repo}` scheme moved to the non-normative implementation-notes table (§4) |
| VF-09 | §4 Architecture Overview | 1 | Ambiguous execution-phase placement for CAP-02/CAP-03 | Confirmed and documented `(Source: User-confirmed)` in US-9 — same edit as VF-03 |
| VF-10 | Document control, throughout | 2, 7 (S2 terminology) | "Programme" (new entity) collided with "programme" (existing prayog programme) | New entity renamed to **"Tenant"** throughout, with documented exclusions (existing `Programme \| prayog` doc-control field, `programme PM` author role, literal code identifiers `PROGRAMME_SERVICE_TOKEN`/`ProgrammeAuthSettings`/`verify_programme_service_token`, and the real `gateflow-programme-vision.md` references) |
| VF-11 | US-6, US-7 (CAP-03); Error table | 9 | CAP-03 had no negative-path coverage | Added AC to US-6 (fork-from-`develop` failure) and US-7 (missing continuation ref), plus two new Error-table rows |
| VF-12 | CAP-01; Appendix B | 9, S4 | Tenant registry was write-only | Added REQ-32 (`GET /api/v1/tenants`, `GET /api/v1/tenants/{tenant_id}`), reflected in CAP-01 coverage, Architecture Overview, Integration Points, and Appendix B |
| VF-13 | REQ-13 row | 8 | REQ-13 silently depended on unconfirmed A2 | REQ-13's own row now reads "(depends on A2 — persistent disk; see §2 Assumptions)" |

---

## Critical (MUST FIX — factually wrong or misleading)

*No Critical findings.*

## Should Fix (reframe, relocate, or make precise)

*No Should Fix findings.*

## Verify (needs user confirmation)

*No Verify findings.*

## Gaps (missing coverage)

*No Gap findings.*

## Clean (no issues found)

- **Check 1 (Source Accuracy):** Re-verified every code citation touched by this revision's edits — `RunOrchestrator.process_job`'s `Path.cwd()` fallback and post-enqueue `sync_harness` call, `WaveStartService`'s pre-enqueue `find_active_run` call, and `TriggerRouter`'s separate webhook-only role — all confirmed accurate against the live `gateflow` checkout.
- **Check 2 (Inference Detection):** No fabricated claims; the one genuinely unconfirmed assumption (A2) remains correctly labeled, and REQ-13 now visibly acknowledges the dependency.
- **Check 3 (Requirement Purity):** REQ-10/REQ-15 now state observable behavior only; the exact on-disk path formula lives in the non-normative implementation-notes table where it belongs.
- **Check 4 (Over-Generalization):** No scope-broadening claims found post-edit.
- **Check 5 (Scope Boundary):** `INIT-GATEFLOW-004` overlap is now explicitly flagged (Non-Goals + Risks + References) rather than silently unaddressed.
- **Check 6 (Testability):** All new AC/Error-table additions (branch-fork failure, missing continuation ref, read/list routes) use concrete, checkable conditions (HTTP codes, named failure reasons).
- **Check 7 (Ambiguity):** No new weak modals or escape hatches introduced by the edits.
- **Check 8 (Assumption-Req Dependency):** REQ-13 now inline-acknowledges A2; no other requirement silently depends on an unconfirmed assumption.
- **Check 9 (Negative Path Coverage):** CAP-03 (US-6/US-7) now has negative-path AC and Error-table rows, matching every other capability in the document.
- **Check 10 (Actor Capability):** "Tenant admin/operator" persona (renamed from "Programme admin/operator") still matches the capabilities described; no actor-capability mismatch introduced by the rename.
- **Check 11 (Intra-Document Consistency):** Verified the rename is complete and consistent — re-scanned every remaining "programme"/"Programme" occurrence (2 found: the pre-existing `Programme \| prayog` doc-control field and the `programme_token.py`/`verify_programme_service_token`/`ProgrammeAuthSettings` code-identifier row) and confirmed both are correctly-excluded, pre-existing concepts, not missed renames.
- **Check S1 (Staleness):** No stale placeholders; the "G1–G5" banner now matches §3/§6 exactly.
- **Check S2 (Contradictions):** REQ-30/REQ-31's wave attribution is now reconciled in both places it's stated (§3 Capabilities note, §5 Delivery-waves note).
- **Check S3 (Cross-References):** All `D*`, `G*`, `A*`, `REQ-*` (including the new REQ-32), `CAP-*`, and `OQ-*` references resolve correctly; the new References entries to `INIT-GATEFLOW-004.md`, `Validation-Report-INIT-GATEFLOW-012.md`, and `Resolution-INIT-GATEFLOW-012.md` all point to files that exist at the stated relative paths.
- **Check S4 (Completeness):** REQ-32 is fully threaded through (top map, Capabilities table, Requirements table, Wave W0 exit, Architecture Overview, Appendix B) — no orphaned reference.

---

## Recommended Next Steps

1. **This document is now clean** — 0 open findings across all 15 checks. No further `review-findings` pass is required for this revision.
2. Proceed per §7 Next steps of the PRD itself: review with PE/programme, then impact map → meta Gate 1 → gateflow implementation waves + `prayog-skills` contract-change spec PR.
3. **Re-run validation** if the document is edited again — pass this same canonical path (`prd/reports/Validation-Report-INIT-GATEFLOW-012.md`) as the prior report for incremental mode.

---

## Appendix — Source index

(Unchanged from revision 1 — see `SRC-1`–`SRC-25` in that revision's history via git, or the References section of `prd/INIT-GATEFLOW-012.md`.)

---

```yaml
handoff:
  contract: sdd-delivery/v2
  stage: validate-requirements
  outcome: pass
  artifact:
    path: prd/reports/Validation-Report-INIT-GATEFLOW-012.md
    digest: sha256:325d5a62437a8aad8318314674b856367094dca9bebfcc87c1f8ac30df29f291
  blockers: []
  signals:
    critical_count: 0
    should_fix_count: 0
    verify_count: 0
    gap_count: 0
    resolved_count: 13
  next_candidates:
    - prd-impact-map
  human_checkpoint: false
  external_action: false
```
