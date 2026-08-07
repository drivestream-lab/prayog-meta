# Resolution Summary

**Report:** `prd/reports/Validation-Report-INIT-GATEFLOW-012.md` (report_revision 1)
**Initiative:** INIT-GATEFLOW-012
**Reviewed on:** 2026-08-07
**resolution_revision:** 1
**Findings reviewed:** 13 of 13

## Decisions

| CHG | VF | Severity | Target | Chosen option | Rationale | Action to apply |
|-----|-----|----------|--------|---------------|-----------|-----------------|
| CHG-01 | VF-01 | Critical | Banner | A | Banner claims G1–G8 but only G1–G5 exist anywhere in the doc — plain stale-header bug | Change banner line 10 to read "G1–G5" |
| CHG-02 | VF-02 | Critical | REQ-23; §4 Architecture Overview | A | Code shows `WaveStartService` calls `RunRepository.find_active_run` directly for the API lane-start route; `TriggerRouter` is a separate webhook-only class | Replace `TriggerRouter.authorize_and_check` with `WaveStartService` (calling `RunRepository.find_active_run`) as the enforcement point in the Architecture Overview diagram and REQ-23's phrasing |
| CHG-03 | VF-03 | Critical | REQ-23–REQ-25; US-9; D10 | A | Confirmed via VF-09: CAP-02/CAP-03 must run in the async worker phase, after the concurrency gate, for D10's "no isolation needed" guarantee to hold mechanically | Rewrite Architecture Overview to state CAP-02 (clone/refresh), CAP-03 (branch resolve), and CAP-04 (harness check) execute inside `RunOrchestrator.process_job` (async worker phase, same place `sync_harness` already runs) — i.e., **after** `WaveStartService`'s `find_active_run` gate has already passed pre-enqueue. Update the pipeline diagram's step order accordingly. |
| CHG-04 | VF-04 | Critical | REQ-10; Problem Statement | A | "A gap" understates the real, active, more dangerous current behavior | Rewrite Problem Statement/banner to name the actual current behavior: `RunOrchestrator.process_job` falls back to `Path.cwd()` (Gateflow's own process directory) as the coding workspace when `workspace_path` is omitted. Add explicit language to REQ-10 that CAP-02 **replaces** this `Path.cwd()` fallback. |
| CHG-05 | VF-05 | Should Fix | OQ-3 | A | A resolved item sitting in a table titled "Open questions" is misleading | Move OQ-3 out of the "Open questions" table into a short "resolved from outline" note alongside G1–G5 |
| CHG-06 | VF-06 | Should Fix | REQ-30, REQ-31 | A | REQ-30 covers three separable pin/contract additions (workspace-prep, branch create-or-reuse, branch-delete shapes) mapping to W1/W2/W5 respectively | State explicitly that the `prayog-skills` contract PR lands once, covering all three shapes together, and that W1/W2 exit is gated on that (W5-scoped-in-name-only) contract PR landing early. Reconcile the "cross-cutting/no wave" framing (line 133) with the "W5 exit" framing (§5) to say the same thing. |
| CHG-07 | VF-07 | Should Fix | CAP-04, US-3 | A | INIT-GATEFLOW-004 already has overlapping "Harness posture" / "GitHub access" scorecard categories; leaving this unflagged risks two independently-built "is this repo ready" checks | Add a cross-reference (Non-Goals or References) noting the overlap with INIT-GATEFLOW-004's scorecard categories and flagging it as an open coordination point — do not resolve which one is authoritative in this pass |
| CHG-08 | VF-08 | Should Fix | REQ-10, REQ-15 | A | House convention (010/011) keeps exact on-disk-path/method-name specificity in a non-normative section; REQ text should state observable behavior only | Move the `{workspace_root}/{org}/{repo}` path formula into the existing "Code-grounded implementation notes (non-normative)" section (§4); reframe REQ-10/REQ-15 to "a deterministic, discoverable workspace exists for the repo" |
| CHG-09 | VF-09 | Verify | CAP-02, CAP-03, CAP-05 | confirm | User confirmed CAP-02/CAP-03 execute in the async worker phase, after the concurrency gate — same decision as CHG-03 | Add `(Source: User-confirmed)` tag where this is stated; implemented jointly with CHG-03's rewrite |
| CHG-10 | VF-10 | Verify | Document-wide ("Programme" concept) | reject → rename | User rejected the "Programme" vs "programme" naming collision and chose to rename the new entity to **"Tenant"** — this also reconnects with the dormant JWT `AuthContext.tenant_id` field found during validation (currently unused, per VF-10's own evidence) | Rename "Programme" → "Tenant" throughout `prd/INIT-GATEFLOW-012.md`: title/body prose, Document control fields, CAP-01 name ("Programme registry" → "Tenant registry"), all US-*/REQ-*/AC text, Appendix A table names (`programmes` → `tenants`, `programme_repos` → `tenant_repos`, `programme_users` → `tenant_users`, `programme_id` → `tenant_id`), Appendix B routes (`/api/v1/programmes` → `/api/v1/tenants`), and G1–G5/D1–D10 decision text referencing "programme." **Do not** rename the pre-existing, unrelated "programme" usage (the one prayog programme governing this whole repo, `PROGRAMME_SERVICE_TOKEN`, `ProgrammeAuthSettings`, `verify_programme_service_token`, "programme admin/operator" persona if it refers to the factory-level role) — only the new per-tenant registry entity and its direct derivatives. Flag ambiguous cases for a follow-up pass rather than guessing silently. |
| CHG-11 | VF-11 | Gap | CAP-03 (US-6, US-7) | add-requirement | CAP-03 has zero negative-path AC, unlike every other capability in the document | Add acceptance criteria + Error-table rows for: (a) fork-from-`develop` failure (repo has no `develop` branch, or `ForgeClient.ensure_branch_from_base` fails), and (b) continuation whose recorded head ref no longer exists on the remote (PR closed / branch deleted out-of-band) |
| CHG-12 | VF-12 | Gap | CAP-01 (Tenant registry, post-rename) | add-requirement | Write-only registry API leaves no way to verify what was registered after the fact | Add a minimal read/list requirement (e.g. `GET /api/v1/tenants`, `GET /api/v1/tenants/{id}`) to CAP-01 and Appendix B, deferring filtering/pagination/exact schema to OQ-01's OpenAPI pass |
| CHG-13 | VF-13 | Gap | REQ-13 | add-requirement | REQ-13 silently depends on unconfirmed Assumption A2; the dependency is visible from the Assumptions table but not from REQ-13's own row | Add "(depends on A2 — persistent disk; see §2 Assumptions)" inline to REQ-13's row in the Requirements table |

## Approved Fixes (ready to apply)

| CHG | VF | Type | Location | Original Finding | Decision | Action |
|-----|-----|------|----------|------------------|----------|--------|
| CHG-01 | VF-01 | Structural | Banner, line 10 | "G1–G8" claimed, only G1–G5 exist | Option A | Change to "G1–G5" |
| CHG-02 | VF-02 | Semantic | §4 Architecture Overview; REQ-23 | Wrong class (`TriggerRouter`) named as concurrency enforcer | Option A | Correct to `WaveStartService` / `RunRepository.find_active_run` |
| CHG-03 | VF-03 | Semantic | §4 Architecture Overview | Ordering doesn't guarantee concurrency-before-workspace-mutation | Option A | State CAP-02/03/04 run in async worker phase, after the gate; fix diagram order |
| CHG-04 | VF-04 | Semantic | Banner; Problem Statement | "Gap" framing understates real `Path.cwd()` fallback | Option A | Name the actual fallback behavior; REQ-10 states CAP-02 replaces it |
| CHG-05 | VF-05 | Structural | §2 Open questions, line 404 | Resolved item inside "Open questions" table | Option A | Move OQ-3 to a resolved-note alongside G1–G5 |
| CHG-06 | VF-06 | Structural | Line 133 vs. line 572 | REQ-30/31 wave attribution contradiction | Option A | Reconcile as one contract PR covering all three shapes; fix both tables to agree |
| CHG-07 | VF-07 | Semantic | Whole document | No cross-reference to INIT-GATEFLOW-004 overlap | Option A | Add cross-reference flagging the overlap, unresolved for now |
| CHG-08 | VF-08 | Semantic | REQ-10, REQ-15 | Exact path formula stated as normative | Option A | Move to non-normative implementation notes; reframe REQ text |
| CHG-11 | VF-11 | Semantic | US-6, US-7; Error table | CAP-03 has no negative-path coverage | add-requirement | Add AC + Error-table rows for fork failure and missing continuation ref |
| CHG-12 | VF-12 | Structural | CAP-01; Appendix B | Registry is write-only | add-requirement | Add minimal read/list requirement + route |
| CHG-13 | VF-13 | Semantic | REQ-13 row | Silent dependency on unconfirmed A2 | add-requirement | Add inline pointer to A2 |

## Confirmed Items (add source tags)

| CHG | VF | Type | Location | Finding | Action |
|-----|-----|------|----------|---------|--------|
| CHG-09 | VF-09 | Semantic | §4 Architecture Overview | Phase placement of CAP-02/CAP-03 was ambiguous | Add `(Source: User-confirmed)` — async worker phase, after the concurrency gate. Implemented as part of CHG-03. |

## Rejected Items (remove or rewrite)

| CHG | VF | Type | Location | Finding | User Direction |
|-----|-----|------|----------|---------|-----------------|
| CHG-10 | VF-10 | Structural | Document-wide | "Programme" (new entity) collides with "programme" (existing prayog programme) | Rename the new entity to **"Tenant"** throughout the document (see CHG-10 action above for exact scope/exclusions) |

## Added as Open Questions

*(None — all findings resulted in a concrete fix, confirmation, or rename direction; none were deferred to a new OQ.)*

## Skipped (no action)

*(None — all 13 findings were reviewed and resolved.)*

## Modified / custom recommendations

*(None — all decisions matched a recommended or listed option; no custom/D-option text was supplied.)*

---

## Summary

- **Findings reviewed:** 13 of 13
- **Approved (Option A):** 11 (CHG-01–08, CHG-11–13)
- **Confirmed:** 1 (CHG-09)
- **Rejected → rename direction given:** 1 (CHG-10 — "Programme" → "Tenant")
- **Skipped:** 0
- **Added as OQ:** 0

**Notable coupling:** CHG-03 and CHG-09 are the same underlying edit (Architecture Overview phase placement) — apply together. CHG-10 (Tenant rename) touches nearly every section of the document and should be applied **before** CHG-01–08/11–13 text edits where the two overlap (e.g. CHG-06's REQ-30 text, CHG-12's new route naming) to avoid re-editing renamed text twice.

---

```yaml
handoff:
  contract: sdd-delivery/v2
  stage: review-findings
  outcome: pass
  artifact:
    path: prd/reports/Resolution-INIT-GATEFLOW-012.md
    digest: sha256:2749659973e2703c926094adb6dc3842ddff00b42983b243a0008207392db332
  blockers: []
  signals:
    findings_reviewed: 13
    approved: 11
    confirmed: 1
    rejected_with_rename: 1
    skipped: 0
    added_as_oq: 0
  next_candidates:
    - validate-requirements
  human_checkpoint: false
  external_action: false
```
