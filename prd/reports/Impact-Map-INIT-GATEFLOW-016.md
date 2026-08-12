---
schema_version: 1
initiative: INIT-GATEFLOW-016
map_revision: 1
source_prd: prd/INIT-GATEFLOW-016.md
source_prd_digest: sha256:2ee19c297b4f948c9f3fbb29d5b45e1e5db9e32fce4a21780915872b3640947a
previous_revision: null
previous_artifact_commit: null
change_reason: Initial impact map — Gateflow Mission Control, greenfield API-grounded scope
material_change: true
generated_at: 2026-08-12T10:42:00Z
---

# Impact map — INIT-GATEFLOW-016 — revision 1

> This file is generated locally before PR creation and becomes the scope source
> of truth when committed. Effective approval is derived from a tech-lead
> GitHub APPROVED review on the exact current meta PR head SHA; dynamic PR state
> is not stored in this artifact's frontmatter.

## 1. Source and revision

| Field | Value |
|-------|-------|
| PRD | `prd/INIT-GATEFLOW-016.md` |
| PRD digest | `sha256:2ee19c297b4f948c9f3fbb29d5b45e1e5db9e32fce4a21780915872b3640947a` |
| Map revision | `1` |
| Previous revision | none |
| Previous artifact commit | none |
| Change reason | Initial impact map |
| Material change | yes — first map for this initiative |

## 2. Affected repositories

| Service | Repo | Team | Scope summary | Scope digest | Spec to create | Confidence |
|---------|------|------|---------------|--------------|----------------|------------|
| gateflow-ops | drivestream-lab/gateflow-ops | @drivestream-lab/prayog-pe-team | Gateflow Mission Control UI/BFF across 7 capability areas (CAP-A–G, REQ-01–REQ-31): identity & access, fleet onboarding, wave operations, checkpoint evidence, board & tickets, initiative & delivery tracking, metrics & efficacy panel — entirely a UI/BFF build against gateflow's existing, live API surface | `sha256:4daa0360b2c895fca619c93bc2bf765c6cca1a0c05d12e0cae9331bead47df08` | `INIT-GATEFLOW-016-gateflow-ops.md` | High |

## 3. Deferred repositories

| Service | Repo | Reason | Revisit condition |
|---------|------|--------|-------------------|
| — | — | None deferred for this initiative's exit — the 4 named gateflow backend gaps (fleet-summary endpoint, workflow-pin readiness probe, process-map endpoint, log-pane richness) and the platform-admin console are explicit non-goals of this PRD, not scope deferred from this map; they are candidate future initiatives, not tracked here | — |

## 4. Transitively affected repositories

| Service | Repo | Depends on | Potential impact | Disposition |
|---------|------|------------|------------------|-------------|
| gateflow | drivestream-lab/gateflow | — (upstream) | `gateflow-ops` consumes 9 route groups from `gateflow` (tenant, catalogue-connection, waves, runs, forge, checkpoints, board, initiatives, metrics) across every capability area. The PRD's own Assumption A4 names concurrent gateflow route/contract changes as a risk. No code or contract change is requested of `gateflow` this initiative (D2) | monitor — API surface must stay stable for the duration of `gateflow-ops` delivery; not a build target |

## 5. Not affected

| Service | Repo | Reason |
|---------|------|--------|
| prayog-skills | drivestream-lab/prayog-skills | Pin/`dispatch` ownership unchanged; this initiative never edits delivery process (D5) |
| launchpad | drivestream-lab/launchpad | No greenfield scaffolding; explicit non-goal |
| prayog-meta | drivestream-lab/prayog-meta | Hosts this PRD, outline, and impact map; not an engineering delivery target |

## 6. Cross-repository contracts

| Contract ID | Provider repo | Consumer repo | Capability | Owner | Status |
|-------------|---------------|---------------|------------|-------|--------|
| CTR-01 | gateflow | gateflow-ops | CAP-A Identity & access (`tenant_routes`) | @drivestream-lab/prayog-pe-team | unchanged |
| CTR-02 | gateflow | gateflow-ops | CAP-B Fleet onboarding (`catalogue_connection_routes`) | @drivestream-lab/prayog-pe-team | unchanged |
| CTR-03 | gateflow | gateflow-ops | CAP-C Wave operations (`waves_routes`, `runs_routes`, `forge_routes`) | @drivestream-lab/prayog-pe-team | unchanged |
| CTR-04 | gateflow | gateflow-ops | CAP-D Checkpoint evidence (`checkpoints_routes`) | @drivestream-lab/prayog-pe-team | unchanged |
| CTR-05 | gateflow | gateflow-ops | CAP-E Board & tickets (`board_routes`) | @drivestream-lab/prayog-pe-team | unchanged |
| CTR-06 | gateflow | gateflow-ops | CAP-F Initiative & delivery tracking (`initiatives_routes`) | @drivestream-lab/prayog-pe-team | unchanged |
| CTR-07 | gateflow | gateflow-ops | CAP-G Metrics & efficacy panel (`metrics_routes`) | @drivestream-lab/prayog-pe-team | unchanged |

All 7 contracts are pre-existing, already-live gateflow routes (delivered across INIT-GATEFLOW-001–003, 010/011, 013, 014, 015). None are new or changed by this initiative — only their consumer (`gateflow-ops`) is new.

## 7. Dependency and build order

```text
gateflow (already live, unchanged) → gateflow-ops (this initiative, W0–W4)
```

| Repo | Depends on | Reason |
|------|------------|--------|
| gateflow-ops | gateflow | Every capability area (CTR-01–CTR-07) reads/writes through gateflow's existing API; no reverse dependency |
| gateflow | none | Not modified this initiative; pre-existing, independently delivered |

No cross-repo sequencing risk — the upstream (`gateflow`) is already fully delivered; `gateflow-ops`'s own internal wave order (W0–W4, PRD §5) is the only sequencing that matters here.

## 8. Revision diff

Omit for revision 1.

## 9. Downstream ripple ledger

N/A — revision 1, no prior map to ripple from.

## 10. Open questions

| ID | Lane | Question | Owner | Blocking | Required by | Default if deferred | Status |
|----|------|----------|-------|----------|-------------|---------------------|--------|
| IM-01 | PE | Confirm `gateflow`'s consumed API surface (all 7 contract groups, CTR-01–07) is treated as frozen/versioned for the duration of `gateflow-ops` delivery, given Assumption A4's named concurrent-change risk | PE | no | Before W0 starts | Proceed — no gateflow change is planned; this is a monitoring commitment, not a blocker | open |
| IM-02 | PM/PE | Whether the 7-capability-area scope (W0–W4) runs as one sequential team effort or can parallelize CAP-F/CAP-G (zero backend gap) alongside CAP-B/C, given they have no shared backend dependency | PE | no | `spec-implementation-plan` | Sequential per PRD §5 wave order (already locked, OQ-3) — parallelization is an execution optimization, not a scope question | open |

## 11. PR readiness handoff

| Item | Value |
|------|-------|
| Verdict | **PR READY** |
| Collision detection | no-collision |
| Collision evidence | Searched local branches (`git branch -a`, no `*INIT-GATEFLOW-016*` match), `gh pr list --state all --search "INIT-GATEFLOW-016"` (no results), `prd/reports/Impact-Map-INIT-GATEFLOW-016*` (none pre-existing) |
| Human resolution | none |
| Resolution completed | n/a |
| Existing PR | none |
| Proposed branch | `chore/INIT-GATEFLOW-016-prd` |
| Proposed base | `develop` |
| Proposed title | `[INIT-GATEFLOW-016] PRD — Gateflow Mission Control, greenfield API-grounded scope` |
| Files to commit | `prd/INIT-GATEFLOW-016.md`, `prd/INIT-GATEFLOW-016-outline.md`, `prd/reports/Impact-Map-INIT-GATEFLOW-016.md`, `prd/reports/Validation-Report-INIT-GATEFLOW-016.md`, `prd/reports/Resolution-INIT-GATEFLOW-016.md`, plus the retirement/cross-reference cleanup this initiative required: deletion of `prd/INIT-GATEFLOW-004.md`, `prd/INIT-GATEFLOW-004-outline.md`, `prd/reports/{Validation-Report,Resolution,Impact-Map}-INIT-GATEFLOW-004.md`; edits to `planning/gateflow-programme-vision.md`, `prd/INIT-GATEFLOW-012.md`, `prd/INIT-GATEFLOW-013.md`, `prd/INIT-GATEFLOW-013-outline.md`, `prd/reports/Impact-Map-INIT-GATEFLOW-013.md`, `prd/INIT-GATEFLOW-014.md`, `prd/INIT-GATEFLOW-014-outline.md`, `prd/INIT-GATEFLOW-015.md`, `prd/INIT-GATEFLOW-015-outline.md` |
| Reviewer | @drivestream-lab/prayog-pe-team |
| Initial Gate 1 label | `impact-map-pending` |
| Additional invalidation label | none |
| Blocking items | none — both open questions (IM-01, IM-02) are non-blocking monitoring/execution items, not scope gates |

**No GitHub side effects have occurred.** Ask whether to publish via
`/commit-workspace` then `/open-draft-pr` (explicit authorization required).

### Proposed Draft PR body

```markdown
## Product change
Gateflow Mission Control (v0) for `gateflow-ops`, built bottom-up from gateflow's
real, live API surface across 7 capability areas — identity, fleet onboarding,
wave operations, checkpoint evidence, board/tickets, initiative delivery
tracking, and metrics/efficacy. Zero new gateflow backend work. Supersedes the
retired INIT-GATEFLOW-004.

## Impact-map summary
- Revision: 1
- PRD digest: `sha256:2ee19c297b4f948c9f3fbb29d5b45e1e5db9e32fce4a21780915872b3640947a`
- Affected repos: gateflow-ops
- Transitively affected (monitor only): gateflow
- Deferred repos: none
- Blocking questions: none (IM-01, IM-02 are non-blocking)
- Artifact: `prd/reports/Impact-Map-INIT-GATEFLOW-016.md`

## Gate 1 — engineering handoff readiness
- [ ] Product/domain blocking questions are resolved in committed artifacts
- [ ] Impact-map scope and dependency order are complete
- [ ] PRD digest and map revision match this PR head
- [ ] PE/tech lead has reviewed the exact current head

Requested reviewer: @drivestream-lab/prayog-pe-team
Initial label: `impact-map-pending`
```

## 12. Approval request (after Draft PR creation)

Tech lead must review this artifact on the meta PR and submit GitHub
**Approve** on the exact PR head SHA using:

```text
Impact map approved
initiative: INIT-GATEFLOW-016
map_revision: 1
meta_pr_head_sha: {SHA after this artifact is committed}
prd_digest: sha256:2ee19c297b4f948c9f3fbb29d5b45e1e5db9e32fce4a21780915872b3640947a
artifact: prd/reports/Impact-Map-INIT-GATEFLOW-016.md
```

The gate remains closed until the review, current PR head SHA, PRD digest, map
revision, and artifact path all match.

All Gate 1 labels must be provisioned before PR creation/update:

```bash
launchpad apply-gates --meta --apply
```

Apply labels through the GitHub REST issues-label endpoint and request team
review through the REST `requested_reviewers` endpoint. Do not use `gh pr edit`
for these operations because its GraphQL path may fail on deprecated Projects
Classic fields.

---

```yaml
handoff:
  contract: sdd-delivery/v2
  stage: prd-impact-map
  outcome: pass
  artifact:
    path: prd/reports/Impact-Map-INIT-GATEFLOW-016.md
    digest: sha256:2ee19c297b4f948c9f3fbb29d5b45e1e5db9e32fce4a21780915872b3640947a
  blockers: []
  signals:
    map_revision: 1
    source_prd_digest: sha256:2ee19c297b4f948c9f3fbb29d5b45e1e5db9e32fce4a21780915872b3640947a
    pr_ready: true
    collision_detection: no-collision
    affected_repos:
      - drivestream-lab/gateflow-ops
    transitively_affected_repos:
      - drivestream-lab/gateflow
  next_candidates:
    - prd-pr-action
  human_checkpoint: false
  external_action: true
  forge:
    action: open_draft_pr
    draft: true
    apply_labels:
      - impact-map-pending
    title: "[INIT-GATEFLOW-016] PRD — Gateflow Mission Control, greenfield API-grounded scope"
    body_path: prd/reports/Impact-Map-INIT-GATEFLOW-016.md
```
