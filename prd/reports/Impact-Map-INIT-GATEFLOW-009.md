---
schema_version: 1
initiative: INIT-GATEFLOW-009
map_revision: 1
source_prd: prd/INIT-GATEFLOW-009.md
source_prd_digest: sha256:76ab22b3c197b9d0cb6b08e6cea379c4e07b3a471b8a88203fddca6014c1c012
previous_revision: null
previous_artifact_commit: null
change_reason: Initial impact map — gateflow-only prove-out (CAP-01–05 / REQ-01–05); sdd-delivery/v2 adherence; skills pin consume-only
material_change: true
generated_at: 2026-08-03T09:00:00Z
---

# Impact map — INIT-GATEFLOW-009 — revision 1

> This file is generated locally before PR creation and becomes the scope source
> of truth when committed. Effective approval is derived from a tech-lead
> GitHub APPROVED review on the exact current meta PR head SHA; dynamic PR state
> is not stored in this artifact's frontmatter.

## 1. Source and revision

| Field | Value |
|-------|-------|
| PRD | `prd/INIT-GATEFLOW-009.md` |
| PRD digest | `sha256:76ab22b3c197b9d0cb6b08e6cea379c4e07b3a471b8a88203fddca6014c1c012` |
| Map revision | `1` |
| Previous revision | none |
| Previous artifact commit | none |
| Change reason | Initial map — both-lane factory prove-out; **gateflow only** `(Source: User-confirmed)` |
| Material change | yes — first map |

## 2. Affected repositories

| Service | Repo | Team | Scope summary | Scope digest | Spec to create | Confidence |
|---------|------|------|---------------|--------------|----------------|------------|
| gateflow | drivestream-lab/gateflow | @drivestream-lab/prayog-pe-team | Prove-out only (no rebuild): live Draft Spec PR tip deliverable with Gateflow-owned commits (REQ-01/CAP-01); required spec wrap-up prove-out (REQ-02/CAP-02); live authorize API stop→approve→side-effect (REQ-03/CAP-03); feature readiness / as-built freeze records (REQ-04/CAP-04); basic automated PR checks replacing placeholder CI (REQ-05/CAP-05); adhere to `sdd-delivery/v2`; skills pin `v0.5.0-rc.2` family **consume only** | `sha256:d0a2b62632113db0fa64cb7d7e63dc393fa5a8217092dda242a99b3392978e9b` | `INIT-GATEFLOW-009-gateflow.md` | High |

## 3. Deferred repositories

| Service | Repo | Reason | Revisit condition |
|---------|------|--------|-------------------|
| gateflow-ops | drivestream-lab/gateflow-ops | Ops portal / BFF UI is next initiative after feature readiness freeze (PRD Non-Goals; CAP-04 deferral) | After INIT-GATEFLOW-009 freeze names ops portal as next |

## 4. Transitively affected repositories

| Service | Repo | Depends on | Potential impact | Disposition |
|---------|------|------------|------------------|-------------|
| gateflow-ops | drivestream-lab/gateflow-ops | gateflow (upstream console) | Consumes gateflow APIs later; no work this INIT | deferred — see §3 |
| prayog-skills | drivestream-lab/prayog-skills | — (SSOT) | Pin tip consumed read-only; no skills RC / pin redesign | monitor — consume only |
| launchpad | drivestream-lab/launchpad | — | Harness remount hygiene only if tip confirmation needs it; no feature delivery | monitor |

## 5. Not affected

| Service | Repo | Reason |
|---------|------|--------|
| prayog-meta | drivestream-lab/prayog-meta | Hosts this PRD + impact map + validation/resolution reports; not an engineering delivery target. Optional vision/planning note at freeze is programme meta hygiene under CAP-04 packaging — **not** a separate affected eng repo `(Source: User-confirmed — impact map gateflow only)` |
| prayog-skills | drivestream-lab/prayog-skills | Contract SSOT already on `v0.5.0-rc.2` family; this INIT does not change skills packages |
| launchpad | drivestream-lab/launchpad | No launchpad product work for this prove-out exit |

## 6. Cross-repository contracts

| Contract ID | Provider repo | Consumer repo | Capability | Owner | Status |
|-------------|---------------|---------------|------------|-------|--------|
| CTR-01 | prayog-skills | gateflow | Pinned `sdd-delivery/v2` workflow + delivery contract + handoff; remount/consume `v0.5.0-rc.2` family tip | prayog-pe-team | unchanged — consume only |
| CTR-02 | gateflow | GitHub (forge) | Draft Spec PR commits/open/update; ForgeClient path; never write `*-lgtm` | prayog-pe-team | unchanged — prove live |
| CTR-03 | gateflow | GitHub / board | Authorize API → sensitive forge side effect (e.g. board tickets) | prayog-pe-team | unchanged — prove live (REQ-03) |

## 7. Dependency and build order

```text
prayog-skills pin v0.5.0-rc.2 family (already delivered — consume)
  → gateflow W0 (tip confirm + meta fixtures / reviewer checklist)
  → gateflow W1 (Draft Spec PR tip deliverable live — REQ-01)
  → gateflow W2 (spec wrap-up live — REQ-02)
  → gateflow W3 (authorize API live — REQ-03; feature readiness + basic PR checks — REQ-04/05)
  → gateflow-ops (deferred — next INIT after freeze)
```

| Repo | Depends on | Reason |
|------|------------|--------|
| gateflow | prayog-skills pin (`v0.5.0-rc.2` family) | Process steps + `sdd-delivery/v2` (CTR-01) |
| gateflow | GitHub App / ForgeClient | Tip commits + authorize forge (CTR-02, CTR-03) |
| gateflow-ops | gateflow | Deferred until feature readiness freeze |

## 8. Revision diff

Omit — revision 1 (initial map).

## 9. Downstream ripple ledger

| Repo | In-flight artifact | Required action | Reason | Owner | Blocking |
|------|--------------------|-----------------|--------|-------|----------|
| gateflow | none | **open** after Gate 1 approval | First map; no prior in-flight spec | prayog-pe-team | no |
| gateflow-ops | none | hold | Deferred | prayog-pe-team | no |
| prayog-skills | pin tip in use | continue | Monitor / consume only | prayog-pe-team | no |
| launchpad | none | continue | Monitor only | prayog-pe-team | no |
| prayog-meta | this PRD/map (local) | continue | Host artifacts; Gate 1 on meta Draft PR | prayog-pm-team | no |

## 10. Open questions

| ID | Lane | Question | Owner | Blocking | Required by | Default if deferred | Status |
|----|------|----------|-------|----------|-------------|---------------------|--------|
| — | — | None blocking | — | — | — | — | — |

Locked decisions (authorize live-only, feature readiness freeze, wrap-up required, `sdd-delivery/v2`, gateflow-only map) are in the PRD — no IM blockers.

## 11. PR readiness handoff

| Item | Value |
|------|-------|
| Verdict | **PR READY** |
| Collision detection | no-collision |
| Collision evidence | No `Impact-Map-INIT-GATEFLOW-009*`; no local/remote branch matching `*009*`; `gh pr list` for INIT-GATEFLOW-009 empty. Prior `INIT-GATEFLOW-001`…`005` / skills INITs are **unrelated** product initiatives. |
| Human resolution | none |
| Resolution completed | n/a |
| Existing PR | none |
| Proposed branch | `chore/INIT-GATEFLOW-009-prd` |
| Proposed base | `develop` |
| Proposed title | `[INIT-GATEFLOW-009] PRD — Both-lane delivery factory prove-out (gateflow only)` |
| Files to commit | `prd/INIT-GATEFLOW-009.md`, `prd/INIT-GATEFLOW-009-outline.md`, `prd/reports/Impact-Map-INIT-GATEFLOW-009.md`, `prd/reports/Validation-Report-INIT-GATEFLOW-009.md`, `prd/reports/Resolution-INIT-GATEFLOW-009.md`, `prd/reports/Update-Summary-INIT-GATEFLOW-009.md`, `prd/reports/PR-Body-INIT-GATEFLOW-009.md` |
| Reviewer | @drivestream-lab/prayog-pe-team |
| Initial Gate 1 label | `impact-map-pending` |
| Additional invalidation label | none |
| Blocking items | none |

**No GitHub side effects have occurred.** Ask the user whether to create the Draft PR via `/open-draft-pr` (or authorize `prd-pr-action`). Continue only after explicit authorization.

### Proposed Draft PR body

See also `prd/reports/PR-Body-INIT-GATEFLOW-009.md` (`handoff.forge.body_path`).

```markdown
## Product change

INIT-GATEFLOW-009 proves the existing Gateflow factory for the **spec** lane
(Draft Spec PR tip with committed artifacts + required wrap-up) and the
**authorize API** path live—without rebuild. Freeze messaging is **feature
readiness** (not horizon nicknames). Delivery/approvals adhere to
**`sdd-delivery/v2`**. Skills pin stays on the **`v0.5.0-rc.2` family**
(consume only). Engineering delivery scope: **gateflow only**.

## Impact-map summary

- Revision: **1**
- PRD digest: `sha256:76ab22b3c197b9d0cb6b08e6cea379c4e07b3a471b8a88203fddca6014c1c012`
- Scope digest (gateflow): `sha256:d0a2b62632113db0fa64cb7d7e63dc393fa5a8217092dda242a99b3392978e9b`
- Affected repos: drivestream-lab/gateflow
- Deferred repos: drivestream-lab/gateflow-ops (ops portal after freeze)
- Monitor: prayog-skills (consume), launchpad
- Blocking questions: none
- Artifact: `prd/reports/Impact-Map-INIT-GATEFLOW-009.md`

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
initiative: INIT-GATEFLOW-009
map_revision: 1
meta_pr_head_sha: {SHA after this artifact is committed}
prd_digest: sha256:76ab22b3c197b9d0cb6b08e6cea379c4e07b3a471b8a88203fddca6014c1c012
artifact: prd/reports/Impact-Map-INIT-GATEFLOW-009.md
```

The gate remains closed until the review, current PR head SHA, PRD digest, map
revision, and artifact path all match.

All Gate 1 labels must be provisioned before PR creation/update:

```bash
launchpad apply-gates --meta --apply
```

PE updates labels as follows:

| Decision | Remove | Add |
|----------|--------|-----|
| Pending/new revision | `impact-map-lgtm`, `impact-map-blocked` | `impact-map-pending` |
| Request changes/hold | `impact-map-pending`, `impact-map-lgtm` | `impact-map-blocked` |
| Approve current head | `impact-map-pending`, `impact-map-blocked`, `impact-map-revised`, `impact-map-stale` | `impact-map-lgtm` |

---

```yaml
handoff:
  contract: sdd-delivery/v2
  stage: prd-impact-map
  outcome: pass
  artifact:
    path: prd/reports/Impact-Map-INIT-GATEFLOW-009.md
    digest: sha256:3159e62832ddc61a0dbf486da91fb301d573e33c8186722c35528bac2ba95b66
  blockers: []
  signals:
    pr_ready: true
    map_revision: 1
    prd_digest: sha256:76ab22b3c197b9d0cb6b08e6cea379c4e07b3a471b8a88203fddca6014c1c012
    scope_digest_gateflow: sha256:d0a2b62632113db0fa64cb7d7e63dc393fa5a8217092dda242a99b3392978e9b
    collision_detection: no-collision
    affected_repos: [drivestream-lab/gateflow]
    deferred_repos: [drivestream-lab/gateflow-ops]
  next_candidates:
    - prd-pr-action
  human_checkpoint: false
  external_action: true
  forge:
    action: open_draft_pr
    draft: true
    apply_labels:
      - impact-map-pending
    remove_labels: []
    title: "[INIT-GATEFLOW-009] PRD — Both-lane delivery factory prove-out (gateflow only)"
    body_path: prd/reports/PR-Body-INIT-GATEFLOW-009.md
    head_ref: chore/INIT-GATEFLOW-009-prd
    base_ref: develop
```
