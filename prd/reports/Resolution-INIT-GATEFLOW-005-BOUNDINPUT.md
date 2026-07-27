# Resolution Summary

**Report:** prd/reports/Validation-Report-INIT-GATEFLOW-005-BOUNDINPUT.md  
**Initiative:** INIT-GATEFLOW-005-BOUNDINPUT  
**Reviewed on:** 2026-07-27  
**resolution_revision:** 1  
**Findings reviewed:** 10 of 10  

**Gateflow evidence (prayog-fleet-cbm / `data-repos-prayog-gateflow` only):**  
- Invent-prose: `CursorAgentRunner._build_prompt` hardcodes Gateflow skill message + implement-lane handoff rules (not pin `prompts/template.md`).  
- Ambient ingest: `HandoffReader.find_latest_handoff` scans `DEFAULT_ARTIFACT_GLOBS` / `artifact_globs` by **mtime**.  
- Concurrent run: `WaveStartService.start_wave` + `TriggerRouter` → `ConflictError` / `NO_CONCURRENT_RUN`.  
- Wave-start API: `POST` via `waves_routes.start_wave` → `WaveStartService`.

---

## Decisions

| CHG | VF | Severity | Target | Chosen option | Rationale | Action to apply |
|-----|-----|----------|--------|---------------|-----------|-----------------|
| CHG-01 | VF-01 | Should Fix | FR-2 / A3 | A | Fix broken §4.4 cross-ref | Renumber heading to **§4.4 Bound-input mapping** (keep `#bound-input-mapping`) |
| CHG-02 | VF-02 | Should Fix | FR-8 | A | Clear W0/W1 exits | Split into **FR-8a** define/store (W0) and **FR-8b** ingest-only (W1); update wave table + AC |
| CHG-03 | VF-03 | Should Fix | FR-7 | A | Testable telemetry | Always require `runner` + `model_id`; profile/provider optional with explicit null/omit |
| CHG-04 | VF-04 | Should Fix | FR-10 | A | Testable prove-it rule | Prove-it = any skill with `dispatch: orchestrated` on active pin (pick one at W0) |
| CHG-05 | VF-05 | Should Fix | Security Auth | A | Drop escape hatch | Auth = programme service token (INIT-001 inherit); remove “unless separately changed” |
| CHG-06 | VF-06 | Verify | §1 Problem | A (reject as fact → rewrite) | User chose risk framing; CBM also shows invent-prose+globs exist today | Rewrite problem to risk framing (“without this substrate, automation would…”). Optional later: cite eng evidence — not required by this CHG |
| CHG-07 | VF-07 | Verify | FR-8 / 001 boundary | A | Narrow supersession | Document: BOUNDINPUT **replaces** 001 automated ingest **for packaged-skill automated runs only**; no formal 001 amendment in this INIT |
| CHG-08 | VF-08 | Gap | Error Handling | A (add-requirement) | Completes negative path; API already fails closed on several errors | Add Error Handling rows: wave-start auth reject; API/transport/unavailable → fail closed, record reason, no AgentRunner |
| CHG-09 | VF-09 | Gap | Error Handling | A (add-requirement) | Align with live gateflow concurrency | Add row: concurrent automate / active run → reject (409 Conflict / fail closed), record reason — inherit INIT-001 / existing WaveStartService behavior |
| CHG-10 | VF-10 | Gap | Product ids | Convention | `id-conventions.md`: REQ-* canonical; FR-n ≡ REQ-n | Document control: Canonical **`REQ-n`**; **`FR-n` ≡ `REQ-n`**; no CAP-*. On apply, retitle FR-1…FR-10 → **REQ-1…REQ-10** (FR-8a/8b → REQ-8a/8b) |

---

## Approved Fixes (ready to apply)

| CHG | VF | Type | Location | Original Finding | Decision | Action |
|-----|-----|------|----------|------------------|----------|--------|
| CHG-01 | VF-01 | Structural | FR-2 AC; A3 | §4.4 cited but unnumbered | A | Renumber to §4.4 Bound-input mapping |
| CHG-02 | VF-02 | Semantic | §5 W0/W1; FR-8 | FR-8 atomic vs partial wave exits | A | Split FR-8a / FR-8b; update phased rollout |
| CHG-03 | VF-03 | Semantic | FR-7 | “when available” escape | A | Always runner+model_id; optional profile/provider with null/omit |
| CHG-04 | VF-04 | Semantic | FR-10 | Vague prove-it policy | A | Rule: any `dispatch: orchestrated` on pin; pick at W0 |
| CHG-05 | VF-05 | Semantic | Security Auth | “unless separately changed” | A | Inherit programme service token; remove hatch |
| CHG-06 | VF-06 | Semantic | §1 Problem | Unsourced current-state claim | A rewrite | Risk framing wording |
| CHG-07 | VF-07 | Semantic | Bound-input / 001 | Glob SSOT boundary unclear | A | Narrow supersession for packaged-skill automated runs; no 001 amendment |
| CHG-08 | VF-08 | Semantic | Error Handling | Missing auth/API failures | A add | Auth + API unavailable rows |
| CHG-09 | VF-09 | Semantic | Error Handling | Missing concurrent run | A add | Concurrent reject row (align live WaveStartService) |
| CHG-10 | VF-10 | Structural | Document control | FR-only ids | Convention | REQ canonical + FR≡REQ; migrate table ids to REQ-* |

## Confirmed Items (add source tags)

*None as confirm-as-written. VF-06/VF-07 were verify with rewrite/document actions above.*

## Rejected Items (remove or rewrite)

| CHG | VF | Type | Location | Finding | User Direction |
|-----|-----|------|----------|---------|----------------|
| CHG-06 | VF-06 | Semantic | §1 Problem | Current-state invent-prose/glob as fact | **Reject fact framing** — rewrite to risk framing |

## Added as Open Questions

*None.*

## Skipped (no action)

*None.*

## Modified / custom recommendations

| CHG | VF | Original Recommendation | User's Alternative | Rationale |
|-----|-----|-------------------------|-------------------|-----------|
| CHG-10 | VF-10 | Add FR≡REQ line or dual-label | **Check convention** → REQ-* canonical; FR-n ≡ REQ-n; migrate ids on apply | `prayog-skills/references/id-conventions.md` |

---

## Recommended Next Steps

1. Apply approved `CHG-01`–`CHG-10` via `/update-documents` (or direct Draft edit).  
2. Re-run `/validate-requirements` incremental against the same Validation-Report path.  
3. Then `/prd-impact-map` (gateflow only).

---

```yaml
handoff:
  contract: sdd-delivery/v2
  stage: review-findings
  outcome: pass
  artifact:
    path: prd/reports/Resolution-INIT-GATEFLOW-005-BOUNDINPUT.md
    digest: sha256:8bba3fe3bfe99a3b9ef15e13c6e762de863134abae41f2fbb89dee6aa199148f
  blockers: []
  signals:
    resolution_revision: 1
    chg_count: 10
    findings_reviewed: 10
    gateflow_cbm_project: data-repos-prayog-gateflow
  next_candidates:
    - update-documents
  human_checkpoint: false
  external_action: false
```
