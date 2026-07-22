# Resolution Summary

**Report:** Validation-Report-INIT-GATEFLOW-001.md  
**Reviewed on:** 2026-07-22  
**Findings reviewed:** 25 of 25

## Approved Fixes (ready to apply)

| # | Type | Location | Original Finding | Decision | Action |
|---|------|----------|-----------------|----------|--------|
| VF-001 | Structural | §4 Integration Points | gateflow-ops listed as W1 integration vs out-of-scope | Approved | Remove gateflow-ops from W1 Integration Points; mark W2 deferred; W1 = gateflow status JSON API only |
| VF-002 | Structural | §2 FR-10 | AC references §5.4 in outline | Approved | Inline metrics dimensions in FR-10 AC or cite §4 Metrics Schema in PRD |
| VF-003 | Semantic | §5 W1 exit + Dependencies | rc-2 / W1 / Phase B sequencing implicit | Approved | Add normative delivery sequence: Joint Gate 1 → rc-2 pin → W1 criteria 3–11 → W1 exit → Phase B |
| VF-004 | Semantic | §3 Evaluation Strategy | Qualitative agent outcome untestable | Approved | Replace with concrete checks: wave artifacts present, harness green, PE sign-off at wave-human-decision |
| VF-005 | Semantic | §2 FR-3/6/14; §4 Architecture | Solution prescriptions blur requirement purity | Approved | Add **Implementation constraints (H1)** subsection; reframe FRs as capabilities where feasible |
| VF-006 | Structural | §5 Open Questions | Numbering gap at 3–4 | Approved | Renumber Open Questions 1–6 or add note "3–4 resolved — see Decisions table" |
| VF-007 | Semantic | §3 Dispatch Preconditions | Extra checks vs INIT-PRAYOG-SKILLS-002 §3.3 | Approved | Cite handoff-envelope.md rules 2, 5, 8 as normative extensions; note alignment needed in paired INIT |
| VF-008 | Semantic | §3 Evaluation Strategy | workflow_scenarios.json scope overstated | Approved | Scope to navigation fixtures; add Gateflow-specific fixture requirement for rc-2 dispatch |
| VF-009 | Semantic | Document control | "W0 dispatch logic" wording | Approved | Change to **W1 PolicyEngine reading `dispatch`** |
| VF-010 | Semantic | US-4 vs FR-10 | Export surface missing in FR-10 | Approved | Add FR/AC for metrics query/export (SQL view or `/metrics` endpoint) |
| VF-011 | Semantic | §4 Workflow Navigation | Illustrative rc-2 nodes could read as Gateflow policy | Approved | Strengthen prefix: **Illustrative only — SSOT in prayog-skills rc-2** |
| VF-012 | Semantic | §2 US-2 | Structured comment schema undefined | Approved | Add Notifier comment template / event schema in FR-9 AC |
| VF-018 | Semantic | §2 (missing) | No Error Handling section | Approved | Add §2.x Error Handling table (webhook, handoff parse, AgentRunner crash, ForgeClient 5xx, Postgres, cancel) |
| VF-019 | Semantic | US-1, FR-2 | Label trigger preconditions undefined | Approved | Add authoritative wave-run precondition checklist |
| VF-020 | Semantic | FR-6 | AgentRunner failure path missing | Approved | Add AC: failed dispatch stops run, `outcome: failed`, notify PE, no workflow advance |
| VF-021 | Semantic | FR-14 / ForgeClient | GitHub API comment failure | Approved | Add negative path: retry/queue comments or `notify_pending` in RunStore |
| VF-022 | Semantic | §5 Dependencies | No Assumptions table | Approved | Add Assumptions table (ID, assumption, status, dependent FRs) |
| VF-023 | Semantic | FR-8 | decision/terminal stops omitted from AC | Approved | Extend FR-8 AC for `decision` and `terminal` node behavior |
| VF-024 | Semantic | W1 exit #11 | Runbook deliverable unowned | Approved | Add W1 deliverable: runbook "orchestrate new initiative repo" in gateflow docs |
| VF-025 | Semantic | FR-1 | Concurrent label trigger policy undefined | Approved | Add requirement for concurrent-run policy; **sub-decision:** reject if active run exists (default lean) — confirm at apply if PE prefers queue/supersede |

## Confirmed Items (add source tags)

| # | Type | Location | Finding | Action |
|---|------|----------|---------|--------|
| VF-013 | Semantic | §2 US-1, L86 | RunStore record within 30s of webhook | Keep 30s SLA `(Source: User-confirmed)` |
| VF-014 | Semantic | §1 Success Criteria, L58 | Reconstruct timeline within 5 minutes | Keep 5-minute target `(Source: User-confirmed)` |
| VF-015 | Semantic | §3 Evaluation, L211 | Gateflow CI pulls fixtures from pinned prayog-skills ref | Add to Evaluation Strategy `(Source: User-confirmed)` |

## Modified Recommendations

| # | Type | Location | Original Recommendation | User's Alternative |
|---|------|----------|------------------------|-------------------|
| VF-016 | Semantic | US-3 / Tech lead persona | Confirm or restrict tech lead audit access | **Open access** — engineering tool; audit logs (RunStore, ForgeClient) readable by any programme engineer; do not role-restrict US-3 |
| VF-017 | Semantic | §4 Programme Config keys | Define keys in harness/programme schema before spec PR | **gateflow repo config only for W1** — keys live in gateflow programme config; harness/meta schema deferred |

## Rejected Items (remove or rewrite)

*None.*

## Added as Open Questions

| # | Finding | Open Question Text |
|---|---------|-------------------|
| VF-025 (sub) | Concurrent run policy | When a label trigger arrives while a run is active on the same PR: reject, queue, or supersede? Default lean: **reject if active run exists**. |

## Skipped (no action)

*None.*

---

## Handoff

```yaml
handoff:
  contract: sdd-delivery/v2
  stage: review-findings
  outcome: pass
  artifact:
    path: prd/reports/Resolution-Validation-Report-INIT-GATEFLOW-001.md
    digest: sha256:16680c5752c6270f91bd978a5ac6e3e2155398b76ca5cd2bf08c23911a02086d
  blockers: []
  signals:
    findings_reviewed: 25
    approved: 20
    confirmed: 3
    modified: 2
    skipped: 0
    rejected: 0
  next_candidates:
    - update-documents
  human_checkpoint: false
  external_action: false
```
