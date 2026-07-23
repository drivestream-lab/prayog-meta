# Requirements Review

**Document:** prd/INIT-PRAYOG-SKILLS-002.md  
**Validated on:** 2026-07-23  
**Sources checked:** 7 source documents  
**Checks run:** 15 (11 semantic + 4 structural)

### Semantic Checks (content accuracy)

| Check | Findings | Status |
|-------|----------|--------|
| 1. Source Accuracy | 1 | FAIL |
| 2. Inference Detection | 1 | FAIL |
| 3. Requirement Purity | 0 | PASS |
| 4. Over-Generalization | 0 | PASS |
| 5. Scope Boundary | 1 | FAIL |
| 6. Testability | 1 | FAIL |
| 7. Ambiguity | 1 | FAIL |
| 8. Assumption-Req Dependency | 0 | PASS |
| 9. Negative Path Coverage | 1 | FAIL |
| 10. Actor Capability | 0 | PASS |
| 11. Intra-Document Consistency | 2 | FAIL |

### Structural Checks (document integrity)

| Check | Findings | Status |
|-------|----------|--------|
| S1. Staleness | 0 | PASS |
| S2. Contradictions | 2 | FAIL |
| S3. Cross-References | 0 | PASS |
| S4. Completeness | 1 | FAIL |

---

## Critical (MUST FIX — factually wrong or misleading)

| # | Type | Location | Check | Finding | Source Says | Doc Claims | Recommendation |
|---|------|----------|-------|---------|-------------|------------|----------------|
| VF-001 | Semantic | §2 FR-1 (line 132); §3 Evaluation Strategy (line 185) | 1, 11 | **Skill node count is wrong.** PRD states **12** skill nodes (`12/12`; **8 manual**, 4 orchestrated). Pinned `workflow.yaml` has **13** `type: skill` nodes (**9 manual**, 4 orchestrated). §4 annotation table is complete (13 rows) but numeric claims elsewhere are stale. | `prayog-skills/workflow.yaml`: 13 skill nodes — `validate-requirements`, `review-findings`, `update-documents`, `prd-impact-map`, `spec-draft`, `initiative-feasibility`, `spec-technical-review`, `spec-implementation-plan`, `board-seed`, `pre-implement`, `loop-spec`, `verify`, `ground-spec` | FR-1: "All **12** skill nodes…"; Evaluation: "**12/12**… **8 manual**, 4 orchestrated" | Update all counts to **13/13** and **9 manual + 4 orchestrated** (FR-1 AC, Success Criteria if referenced, §3 Evaluation pass threshold). |

---

## Should Fix (reframe, relocate, or make precise)

| # | Type | Location | Check | Finding | Recommendation |
|---|------|----------|-------|---------|----------------|
| VF-002 | Semantic | §1 Proposed Solution enum table (lines 48–52) vs §2 FR-4 (line 135) | 11, S2 | **`observed` enum scope ambiguous.** §1 lists `observed` as a dispatch value; Decision #1 and FR-4 restrict rc-2 v1 to `{manual, orchestrated}` only. Readers may think rc-2 must implement three enum values. | Add footnote in §1 table: "`observed` — reserved/future; **excluded from rc-2 v1 contract tests** per Decision #1." Align §1 wording with FR-4. |
| VF-003 | Structural | §5 Decisions header (line 382) vs §5 Open Questions (lines 393–395) | 7, S2 | **Decision status label contradicts Open Questions.** Section titled "Decisions (resolved — Draft PM stance)" while Open Questions state Decisions 1–6 require Joint Gate 1 confirmation. | Rename to **"Decisions (draft PM stance — Joint Gate 1 confirmation pending)"** or move unsettled items back to Open Questions only. |
| VF-004 | Semantic | §2 US-2 AC (line 105) | 6, 7 | **US-2 overstates algorithm inputs.** AC says algorithm uses "`next.type` and `next.dispatch` **only**" but normative algorithm (§4) also requires `programme_trigger_authorized`; INIT-GATEFLOW-001 adds further preconditions. | Rephrase to: "**No skill id allowlists** in consumer source; eligibility read from pinned `workflow.yaml` `dispatch` field only." |
| VF-005 | Semantic | §2 Non-Goals vs sibling INIT-GATEFLOW-001 | 5 | **Sibling scope tension on `observed`.** INIT-GATEFLOW-001 FR-10 / metrics schema reference `dispatch_mode: observed` and `dispatch: observed` nodes. This PRD defers `observed` enum (Decision #1). Not a direct contradiction (Gateflow can record manual-skill timing), but cross-INIT alignment is unstated. | Add explicit cross-ref in Open Questions or Dependencies: "Align INIT-GATEFLOW-001 metrics `observed` handling with rc-2 v1 enum deferral at Joint Gate 1." |
| VF-006 | Semantic | §5 Assumptions A2 (line 368) | 6 | **Wave lane cardinality untested in FR-4.** Assumption A2 locks four wave skills; FR-4 tests wave lane values but does not require a test that **exactly four** nodes are `orchestrated` (guard against accidental fifth orchestrated node). | Add FR-4 AC or evaluation row: contract test asserts orchestrated count = 4 and set equals `{pre-implement, loop-spec, verify, ground-spec}`. |

---

## Verify (needs user confirmation)

| # | Type | Location | Check | Finding | Question for User |
|---|------|----------|-------|---------|-------------------|
| VF-007 | Semantic | §5 Technical dependencies (line 359) | 2, 10 | **`rc-2` branch existence not verified from meta repo.** PRD assumes active rc-2 development branch on `drivestream-lab/prayog-skills`. | Confirm rc-2 branch exists (or will be created) before implementation step 5? |
| VF-008 | Semantic | §2 FR-3 (line 134) | 2 | **Handoff cross-ref content undefined.** `handoff-envelope.md` (current pin) has no `dispatch` mention — correct for v0.4.3. FR-3 requires a cross-ref paragraph but not its normative text. | Should the handoff cross-ref document optional future `executed_by` only, or also instruct producers on manual vs orchestrated runs? |

---

## Gaps (missing coverage)

| # | Type | Location | Check | Finding | Suggested Addition |
|---|------|----------|-------|---------|-------------------|
| VF-009 | Structural | §5 Open Questions (lines 397–400) | S4 | **Open Questions lack owners.** Four Joint Gate 1 agenda items have no assigned owner (PE / PM / sponsor). | Add Owner column or inline owner per question (e.g., PE for tag naming, PM+PE for pin timing). |
| VF-010 | Semantic | §2 Error Handling (lines 141–151) | 9 | **Producer-side failure modes thin.** Error table covers consumer behavior only. Missing: invalid workflow edit merged despite review (beyond CI), or rc-2 pin upgrade mid-programme with mixed v0.4.3/v0.4.3+ consumers. | Add brief prayog-skills-side row: invalid `dispatch` → CI fail; pin upgrade → migration note §4 covers consumer default. |

---

## Clean (no issues found)

| Check | Verified |
|-------|----------|
| 3. Requirement Purity | FRs appropriately specify contract deliverables for a platform SSOT INIT; no inappropriate UI/design leakage. |
| 4. Over-Generalization | Wave-lane and schema-default claims match outline, vision, and paired INIT-GATEFLOW-001 scope. |
| 8. Assumption-Req Dependency | A1 (Joint Gate 1) explicitly gates rc-2 delivery in §5 blocked table; dependent FRs acknowledge blocker. |
| 10. Actor Capability | PE, orchestrator engineer, tech lead, sponsor roles match programme model. |
| S1. Staleness | No orphaned `[TBD]` markers contradict resolved Decisions; Open Questions appropriately flag Joint Gate 1 items. |
| S3. Cross-References | Links to INIT-GATEFLOW-001, outline, vision, FR-5, Decision #9, and §4 sections resolve correctly. |

**Stage artifacts:** `Stage4_Scenario_Matrix.md` and `Stage6_User_Flows.md` — **unavailable** (Check 9 scenario-matrix sub-check skipped).

---

## Recommended Next Steps

1. **Review findings interactively** — Use the `review-findings` skill with this report file to walk through each finding, collect decisions, and produce a resolution summary.
2. **Or resolve manually** — Fix **VF-001 (Critical)** first (13 skill nodes), then Should Fix items VF-002–VF-006.
3. **Apply fixes** — Use `update-documents` with the resolution summary or edit the PRD directly.
4. **Re-run validation** — After fixes, re-run `/validate-requirements` with this report as the prior report for incremental mode.

---

```yaml
handoff:
  contract: sdd-delivery/v2
  stage: validate-requirements
  outcome: findings
  artifact:
    path: prd/reports/Validation-Report-INIT-PRAYOG-SKILLS-002.md
    digest: sha256:8d8adf946c4333e2758b127a660fc0ab6ca343b8e71f3c139f8798687c97cc53
  blockers:
    - VF-001
  signals:
    finding_count: 10
    critical_count: 1
    should_fix_count: 5
    verify_count: 2
    gap_count: 2
    checks_run: 15
    checks_skipped: 0
    stage_artifacts_unavailable:
      - Stage4_Scenario_Matrix
      - Stage6_User_Flows
  next_candidates:
    - review-findings
  human_checkpoint: true
  external_action: false
```
