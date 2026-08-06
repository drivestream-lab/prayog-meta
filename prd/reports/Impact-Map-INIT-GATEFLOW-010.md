---
schema_version: 1
initiative: INIT-GATEFLOW-010
map_revision: 1
source_prd: prd/INIT-GATEFLOW-010.md
source_prd_digest: sha256:457f19617113171c973abdbc15d1afaa00df2f6947ab4567b57d8440bd88b206
previous_revision: null
previous_artifact_commit: null
change_reason: Initial impact map — eng-lane pin tip executor parity (gateflow only)
material_change: true
generated_at: 2026-08-05T09:27:00Z
---

# Impact map — INIT-GATEFLOW-010 — revision 1

> This file is generated locally before PR creation and becomes the scope source
> of truth when committed. Effective approval is derived from a tech-lead
> GitHub APPROVED review on the exact current meta PR head SHA; dynamic PR state
> is not stored in this artifact's frontmatter.

## 1. Source and revision

| Field | Value |
|-------|-------|
| PRD | `prd/INIT-GATEFLOW-010.md` |
| PRD digest | `sha256:457f19617113171c973abdbc15d1afaa00df2f6947ab4567b57d8440bd88b206` |
| Map revision | `1` |
| Previous revision | none |
| Previous artifact commit | none |
| Change reason | Initial map for eng-lane tip parity (CAP-01…06 / REQ-01…20) |
| Material change | yes — first map |

**Identity collision:** `no-collision` — no `Impact-Map-INIT-GATEFLOW-010`, no
branch/PR for INIT-GATEFLOW-010; open GATEFLOW-002…009 PRs are distinct INITs.

## 2. Affected repositories

| Service | Repo | Team | Scope summary | Scope digest | Spec to create | Confidence |
|---------|------|------|---------------|--------------|----------------|------------|
| gateflow | drivestream-lab/gateflow | @drivestream-lab/prayog-pe-team | Eng-lane pin tip executor parity (gateflow only): parse/apply board-status hops; create-tickets predicates; implement-start board-resolve + In Progress; closeout Done; closure Enter-at POST /api/v1/initiatives/closure/start with Done-gate + EPIC Done programme hygiene + purge-app; verify scripts CAP-01..06 / REQ-01..20; pin consume v0.5.0-rc.2; no meta purge / PM Enter-at / ops UI | `sha256:09c89c143c14401c8812738c162c05a2f5e504cabafd1818ee72eb4e9b781532` | `INIT-GATEFLOW-010-gateflow.md` | High |

**H2 payload (canonical):**

```text
repo=drivestream-lab/gateflow
status=affected
capabilities=CAP-01,CAP-02,CAP-03,CAP-04,CAP-05,CAP-06,REQ-01,REQ-02,REQ-03,REQ-04,REQ-05,REQ-06,REQ-07,REQ-08,REQ-09,REQ-10,REQ-11,REQ-12,REQ-13,REQ-14,REQ-15,REQ-16,REQ-17,REQ-18,REQ-19,REQ-20
contracts=CTR-01,CTR-02,CTR-03
depends_on=prayog-skills
scope=Eng-lane pin tip executor parity (gateflow only): parse/apply board-status hops; create-tickets predicates; implement-start board-resolve + In Progress; closeout Done; closure Enter-at POST /api/v1/initiatives/closure/start with Done-gate + EPIC Done programme hygiene + purge-app; verify scripts CAP-01..06 / REQ-01..20; pin consume v0.5.0-rc.2; no meta purge / PM Enter-at / ops UI
```

## 3. Deferred repositories

| Service | Repo | Reason | Revisit condition |
|---------|------|--------|-------------------|
| gateflow-ops | drivestream-lab/gateflow-ops | Ops UI / Mission Control out of scope (PRD non-goals; REQ-18 deferred list) | Later INIT after eng tip parity freeze |
| prayog-skills | drivestream-lab/prayog-skills | Pin consume-only — **no** workflow redesign in this INIT (REQ-01 / A3) | Only if tip family change forces pin work (separate INIT) |

## 4. Transitively affected repositories

| Service | Repo | Depends on | Potential impact | Disposition |
|---------|------|------------|------------------|-------------|
| gateflow-ops | drivestream-lab/gateflow-ops | gateflow (upstream) | May later surface lane/closure status | monitor — deferred |
| launchpad | drivestream-lab/launchpad | — | Materialize existing skills; no new `/update-board-status` human skill | monitor — not in delivery scope |
| prayog-skills | drivestream-lab/prayog-skills | — (SSOT) | Gateflow remounts tip `v0.5.0-rc.2`; no redesign | monitor — deferred (consume-only) |

## 5. Not affected

| Service | Repo | Reason |
|---------|------|--------|
| prayog-meta | drivestream-lab/prayog-meta | Hosts PRD + impact map + reports; **not** an eng delivery repo for this INIT (primary delivery = gateflow only) |
| launchpad | drivestream-lab/launchpad | No launchpad feature delivery; harness remount already programme practice |

## 6. Cross-repository contracts

| Contract ID | Provider repo | Consumer repo | Capability | Owner | Status |
|-------------|---------------|---------------|------------|-------|--------|
| CTR-01 | prayog-skills | gateflow | Pinned `workflow.yaml` + `sdd-delivery/v2` @ `v0.5.0-rc.2` (consume; CAP-06 / REQ-01–02) | prayog-pe-team | unchanged — available |
| CTR-02 | GitHub (forge/board) | gateflow | Board create/status + Draft PRs via ForgeClient (CAP-02…05 / REQ-03…16) | prayog-pe-team | changed — gateflow must execute board-status + closure forge hops |
| CTR-03 | gateflow | GitHub (forge/board) | Outbound forge: `open_draft_pr`, `create_board_tickets`, `update_board_status`; never merge / `*-lgtm` (REQ-09, REQ-16) | prayog-pe-team | changed — add `update_board_status` parity |

## 7. Dependency and build order

```text
prayog-skills pin v0.5.0-rc.2 (already remounted / consume-only)
  → gateflow W0 (parse board-status + purpose/owner; all nodes get_node)
  → gateflow W1 (APPLY_FORGE board-status; implement-start In Progress)
  → gateflow W2 (ticket gate 400/422; create predicates; verify)
  → gateflow W3 (closeout Done; wave-complete no auto-chain; spec verify)
  → gateflow W4 (closure Enter-at + Done-gate + EPIC Done + purge + REQ-20; freeze)
  → gateflow-ops (deferred — out of 010)
```

| Repo | Depends on | Reason |
|------|------------|--------|
| gateflow | prayog-skills (pin `v0.5.0-rc.2`) | Navigation + forge policy SSOT (REQ-01) |
| gateflow | GitHub App / ForgeClient | Board + Draft PR side effects (CTR-02/03) |
| gateflow-ops | gateflow | Upstream API — deferred |

## 8. Revision diff

_Omit for revision 1._

## 9. Downstream ripple ledger

| Repo | In-flight artifact | Required action | Reason | Owner | Blocking |
|------|--------------------|-----------------|--------|-------|----------|
| gateflow | none (no INIT-010 app spec yet) | **open** after Gate 1 approval | First map; new eng scope | prayog-pe-team | no |
| gateflow-ops | none | hold | Deferred | prayog-pe-team | no |
| prayog-skills | tip remounted (programme) | continue | Monitor / consume-only | prayog-pe-team | no |
| launchpad | none | continue | Monitor only | prayog-pe-team | no |

## 10. Open questions

| ID | Lane | Question | Owner | Blocking | Required by | Default if deferred | Status |
|----|------|----------|-------|----------|-------------|---------------------|--------|
| IM-01 | PE | Exact problem+json / OpenAPI error body field names (**OQ-01**) | PE | no | OpenAPI / spec PR | Defer field names; HTTP **400**/**422** semantics remain normative in PRD | open |
| IM-02 | PM | Parallel open GATEFLOW meta PRs (#10–#23) — sequencing vs this INIT | PM | no | Gate 1 scheduling | Proceed; distinct INIT ids | open |

## 11. PR readiness handoff

| Item | Value |
|------|-------|
| Verdict | **PR READY** |
| Collision detection | no-collision |
| Collision evidence | Searched `Impact-Map-INIT-GATEFLOW-010*`, branches `*010*`, `gh pr list` for INIT-GATEFLOW-010 / GATEFLOW-010 — none; open PRs are INIT-002…009 (unrelated) |
| Human resolution | none |
| Resolution completed | n/a |
| Existing PR | none |
| Proposed branch | `chore/INIT-GATEFLOW-010-prd` |
| Proposed base | `develop` |
| Proposed title | `[INIT-GATEFLOW-010] PRD — Engineering-lane pin tip executor parity` |
| Files to commit | `prd/INIT-GATEFLOW-010.md`, `prd/INIT-GATEFLOW-010-outline.md`, `prd/reports/Impact-Map-INIT-GATEFLOW-010.md`, `prd/reports/Validation-Report-INIT-GATEFLOW-010.md`, `prd/reports/Resolution-INIT-GATEFLOW-010.md`, `prd/reports/Update-Summary-INIT-GATEFLOW-010.md` |
| Reviewer | @drivestream-lab/prayog-pe-team |
| Initial Gate 1 label | `impact-map-pending` |
| Additional invalidation label | none |
| Blocking items | none |

**No GitHub side effects have occurred.** Ask whether to publish via
`/commit-workspace` then `/open-draft-pr` (explicit authorization required).

### Proposed Draft PR body

```markdown
## Product change

Gateflow eng-lane tip parity against prayog-skills `sdd-delivery/v2` @
`v0.5.0-rc.2`: board-status hops, create-tickets predicates, implement-start
board-resolve + In Progress, closeout Done, initiative closure Enter-at
(purge-app only), verify scripts. Primary delivery = **gateflow only**.
PM Enter-at and meta purge are out of scope.

## Impact-map summary
- Revision: 1
- PRD digest: `sha256:457f19617113171c973abdbc15d1afaa00df2f6947ab4567b57d8440bd88b206`
- Affected repos: drivestream-lab/gateflow
- Deferred repos: gateflow-ops, prayog-skills (consume-only)
- Blocking questions: none (IM-01 = OQ-01 non-blocking)
- Artifact: `prd/reports/Impact-Map-INIT-GATEFLOW-010.md`

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
initiative: INIT-GATEFLOW-010
map_revision: 1
meta_pr_head_sha: {SHA after this artifact is committed}
prd_digest: sha256:457f19617113171c973abdbc15d1afaa00df2f6947ab4567b57d8440bd88b206
artifact: prd/reports/Impact-Map-INIT-GATEFLOW-010.md
```

The gate remains closed until the review, current PR head SHA, PRD digest, map
revision, and artifact path all match.

All Gate 1 labels must be provisioned before PR creation/update:

```bash
launchpad apply-gates --meta --apply
```

---

```yaml
handoff:
  contract: sdd-delivery/v2
  stage: prd-impact-map
  outcome: pass
  artifact:
    path: prd/reports/Impact-Map-INIT-GATEFLOW-010.md
    digest: sha256:99f4f22bdf8cc33935d443468bcb738492a2eddfca0c3366a80ed57773efb1d1
  blockers: []
  signals:
    map_revision: 1
    source_prd_digest: sha256:457f19617113171c973abdbc15d1afaa00df2f6947ab4567b57d8440bd88b206
    pr_ready: true
    collision_detection: no-collision
    affected_repos:
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
    title: "[INIT-GATEFLOW-010] PRD — Engineering-lane pin tip executor parity"
    body_path: prd/reports/Impact-Map-INIT-GATEFLOW-010.md
```
