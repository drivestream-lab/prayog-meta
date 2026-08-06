# Resolution Summary

**Report:** `prd/reports/Validation-Report-INIT-GATEFLOW-009.md`  
**Initiative:** INIT-GATEFLOW-009  
**Reviewed on:** 2026-08-03  
**resolution_revision:** 1  
**Findings reviewed:** 12 of 12  

**Programme lens (applies to every CHG):** Approvals follow **sdd-delivery/v2** on GitHub PRs (labels + Q&A + Approve). Gateflow owns orchestrated forge/CI/order mechanics. Do not invent PE-waive ceremonies or IDE gates. Apply across **Draft PRD + outline** (and any validation wording that contradicted this).

## Decisions

| CHG | VF | Severity | Target | Chosen option | Rationale | Action to apply |
|-----|-----|----------|--------|---------------|-----------|-----------------|
| CHG-01 | VF-01 | Critical | Outline ↔ Draft | A | Single exit bar | Rewrite outline to match Draft locked decisions: authorize **live only** (no paper waiver); freeze = **feature readiness** (not “Horizon 1.5 closed” headline); drop waiver/A3-or-waive and Horizon-closed success language |
| CHG-02 | VF-02 | Critical | US-2 / W2 | A | Exit bar must be decidable | Make wrap-up **in-scope required**: rewrite US-2 AC + add Success Criteria KPI; keep W2 mandatory live prove-out |
| CHG-03 | VF-03 | Should Fix | US-1 | Custom | Ordering is Gateflow-owned | Rewrite US-1 AC: tip must carry committed step artifacts **per Gateflow process**; do **not** invent PRD-level commit-before-open sequencing |
| CHG-04 | VF-04 | Should Fix | §4 Architecture | A | Avoid false linear order | Split ASCII into **independent** prove-out paths (spec tip deliverable vs authorize API), or mark order non-normative |
| CHG-05 | VF-05 | Should Fix | §1 Success Criteria | A | Align with CHG-02 | Add wrap-up live-prove KPI to Success Criteria |
| CHG-06 | VF-06 | Should Fix | US-5 | Custom | Same as Gateflow vs human paths | Orchestrated: fail-closed checks are whatever **Gateflow repo/CI** enforce — PRD verifies that, does not invent branch-protection settings. Human developer path: inputs come from human PR workflow — do not duplicate as a second rule set |
| CHG-07 | VF-07 | Should Fix | US-5 | Custom | Need, not tool lock | Require **basic automated PR checks** on Gateflow PRs (incl. examples such as branch-naming / PR hygiene). Drop exclusive “lint + unit” lock; optional Implementation Note may list lint/unit as examples |
| CHG-08 | VF-08 | Should Fix | Document control | Custom | Adhere to delivery process | Remove PE-waive / ceremony / `[confirm on filing]`. State Gate 1 = standard meta PR path: `impact-map-pending` → PR Q&A → `impact-map-lgtm` + Approve on exact head (sdd-delivery/v2). Apply same to outline D5 / Gate 1 posture |
| CHG-09 | VF-09 | Verify | §1 Problem Statement | confirm | Coding lane is reuse baseline | Keep “already trusted”; add `(Source: User-confirmed)` (optional cite as-built / prior INIT) |
| CHG-10 | VF-10 | Gap | Assumptions | add-requirement | Preconditions visible | Add Assumptions table: pin tip remounted; meta fixtures for W0; authorize token available; forge = GitHub App path; Gate 1 = standard `impact-map-*` PR path — link to dependent US/waves |
| CHG-11 | VF-11 | Gap | US-1 / US-3 | add-requirement | Match real Gateflow stops | Add Error Handling / alternative paths: spec start rejected (bad preconditions); commit/forge fail mid-walk; authorize deny/wrong state/unavailable; empty tip does not count as success |
| CHG-12 | VF-12 | Gap | US-1–US-5 | add-requirement | Impact map / tickets | Assign `CAP-*` / `REQ-*` (or US→REQ map) **without inventing new behaviour** |
| CHG-13 | adhoc | New information | Outline + Draft Document control / Locked decisions | User (2026-08-03) | Explicit delivery contract | State in **both** outline and Draft PRD that this INIT **adheres to `sdd-delivery/v2`** (GitHub PR + labels + Q&A + Approve; Gate 1 = `prd-impact-acceptance` / `impact-map-*`; no alternate PE-waive ceremony). Cite contract name explicitly—not only implied via Gate 1 wording |

## Approved Fixes (ready to apply)

| CHG | VF | Type | Location | Original Finding | Decision | Action |
|-----|-----|------|----------|------------------|----------|--------|
| CHG-01 | VF-01 | Semantic | Outline §§1–2, A3/A4 | Outline waiver + Horizon-closed vs Draft locks | Option A | Align outline to Draft locked decisions |
| CHG-02 | VF-02 | Semantic | US-2 vs W2 | Wrap-up optional vs W2 mandatory | Option A | Wrap-up required in US-2 + KPI + W2 |
| CHG-03 | VF-03 | Semantic | US-1 AC | Commit/open escape hatch | Custom | Gateflow-owned order; tip has artifacts |
| CHG-04 | VF-04 | Semantic | §4 Architecture | Linear wrap-up→authorize | Option A | Independent paths / non-normative order |
| CHG-05 | VF-05 | Semantic | Success Criteria | Missing wrap-up KPI | Option A | Add wrap-up KPI |
| CHG-06 | VF-06 | Semantic | US-5 AC | Vague branch protection | Custom | Gateflow CI/protection as SSOT for orchestrated |
| CHG-07 | VF-07 | Semantic | US-5 / §3 | Lint+unit prescription | Custom | Basic automated PR checks (incl. branch naming etc.) |
| CHG-08 | VF-08 | Structural | Document control + outline | PE-waive / confirm | Custom | Standard Gate 1 meta PR path only |
| CHG-10 | VF-10 | Semantic | New Assumptions | No assumptions table | add-requirement | Add Assumptions (delivery/Gateflow preconditions) |
| CHG-11 | VF-11 | Semantic | Error Handling | Missing negative paths | add-requirement | Add Gateflow-aligned error/alt paths |
| CHG-12 | VF-12 | Structural | US-1–US-5 | No CAP/REQ ids | add-requirement | Assign ids only |
| CHG-13 | adhoc | New information | Outline + Draft | Contract not named explicitly | User addendum | Explicit `sdd-delivery/v2` adherence in both docs |

## Confirmed Items (add source tags)

| CHG | VF | Type | Location | Finding | Action |
|-----|-----|------|----------|---------|--------|
| CHG-09 | VF-09 | Semantic | §1 Problem Statement | “Coding waves already trusted” unsourced | Add `(Source: User-confirmed)` |

## Rejected Items (remove or rewrite)

*None.*

## Added as Open Questions

*None.* (VF-08 resolved to delivery-process adherence, not an OQ.)

## Skipped (no action)

*None.*

## Modified / custom recommendations

| CHG | VF | Original Recommendation | User's Alternative | Rationale |
|-----|-----|-------------------------|-------------------|-----------|
| CHG-03 | VF-03 | State commit-then-open explicitly or `[TBD]` | Ordering decided by **Gateflow** | Factory owns forge/process order; PRD requires tip deliverable |
| CHG-06 | VF-06 | Name branches / required checks or `[TBD]` | Orchestrated = Gateflow; human developer = human PR inputs | Don’t invent protection settings in PRD |
| CHG-07 | VF-07 | Need in AC; lint/unit as Implementation Note | Basic automated PR checks (branch naming etc.) | Broader than lint+unit; still solution-light |
| CHG-08 | VF-08 | Resolve PE-waive yes/no or OQ | **No PE-waive.** Adhere to existing Git PR + labels Gate 1 | Delivery process already set |
| CHG-13 | adhoc | (implied in CHG-08) | **Explicit callout** of `sdd-delivery/v2` in outline **and** Draft PRD | User: call out adherence before further apply |

## Apply scope reminder

When running `update-documents`, apply CHGs to:

1. `prd/INIT-GATEFLOW-009.md` (Draft PRD)  
2. `prd/INIT-GATEFLOW-009-outline.md` (especially CHG-01, CHG-08; remove Horizon-closed / waiver / A3-or-waive / D5 ceremony language)  

Do not re-decide semantics in `update-documents` — execute these CHGs only.

---

## Recommended Next Steps

1. **`update-documents`** with this resolution — apply all `CHG-*`  
2. Re-run **`validate-requirements`** (incremental) against `prd/reports/Validation-Report-INIT-GATEFLOW-009.md`  
3. When clean → **`prd-impact-map`** (gateflow only) → meta Draft PR (`impact-map-pending`) per delivery process  

---

```yaml
handoff:
  contract: sdd-delivery/v2
  stage: review-findings
  outcome: pass
  artifact:
    path: prd/reports/Resolution-INIT-GATEFLOW-009.md
    digest: sha256:4a3a66f338407c9030f2c0dd92338d63f89d29b7c8e02980fb6d8dfd61ace2fc
  blockers: []
  signals:
    resolution_revision: 1
    findings_reviewed: 12
    findings_total: 12
    approved: 8
    custom: 4
    confirmed: 1
    rejected: 0
    skipped: 0
    added_as_oq: 0
    chg_ids: [CHG-01, CHG-02, CHG-03, CHG-04, CHG-05, CHG-06, CHG-07, CHG-08, CHG-09, CHG-10, CHG-11, CHG-12, CHG-13]
    apply_targets:
      - prd/INIT-GATEFLOW-009.md
      - prd/INIT-GATEFLOW-009-outline.md
  next_candidates:
    - update-documents
  human_checkpoint: false
  external_action: false
  forge:
    action: commit_workspace
    draft: false
```
