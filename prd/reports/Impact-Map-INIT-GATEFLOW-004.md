---
schema_version: 1
initiative: INIT-GATEFLOW-004
map_revision: 1
source_prd: prd/INIT-GATEFLOW-004.md
source_prd_digest: sha256:90fff56a5c2699390177d573e31df1d34f8980aa6553d4cce17128877af2882f
previous_revision: null
previous_artifact_commit: null
change_reason: initial map — Mission Control v0 (gateflow-ops primary; gateflow + prayog-skills supporting)
material_change: true
generated_at: 2026-07-27T10:00:00Z
---

# Impact map — INIT-GATEFLOW-004 — revision 1

> This file is generated locally before PR creation and becomes the scope source
> of truth when committed. Effective approval is derived from a tech-lead
> GitHub APPROVED review on the exact current meta PR head SHA; dynamic PR state
> is not stored in this artifact's frontmatter.

## 1. Source and revision

| Field | Value |
|-------|-------|
| PRD | `prd/INIT-GATEFLOW-004.md` |
| PRD digest | `sha256:90fff56a5c2699390177d573e31df1d34f8980aa6553d4cce17128877af2882f` |
| Map revision | `1` |
| Previous revision | none |
| Previous artifact commit | none |
| Change reason | Initial map for Gateflow Mission Control (v0) |
| Material change | yes — first revision |

## 2. Affected repositories

| Service | Repo | Team | Scope summary | Scope digest | Spec to create | Confidence |
|---------|------|------|---------------|--------------|----------------|------------|
| gateflow-ops | drivestream-lab/gateflow-ops | @drivestream-lab/prayog-pe-team | Mission Control v0: thin ops-user identity+sign-in (REQ-32); strict scorecard onboarding (REQ-33); fleet home (REQ-34); start/inspect waves last 30 days (REQ-35); run cockpit map+timeline+full log pane+GitHub jump (REQ-36); efficacy visibility+collect numbers (REQ-37); process display-only (REQ-38); BFF consumer of gateflow | `sha256:682d055f5a0ee565202376bbcfb8c6e43d88464181df78ab1d1f6abbd6c26e87` | `INIT-GATEFLOW-004-gateflow-ops.md` | High |
| gateflow | drivestream-lab/gateflow | @drivestream-lab/prayog-pe-team | Supporting only: extend run/metrics/wave APIs where Mission Control needs missing programme capabilities — fleet onboarding records, clearer wave summaries (REQ-39); not a second orchestrator | `sha256:1264d9ab2feb688091974445d7a9f7326f5de1311a658aa5a46bf4717d23c769` | `INIT-GATEFLOW-004-gateflow.md` (thin / API delta) | High |
| prayog-skills | drivestream-lab/prayog-skills | @drivestream-lab/prayog-pe-team | Supporting: pin `dispatch: orchestrated` for engg **spec-lane** skills (`spec-draft`, `initiative-feasibility`, `spec-technical-review`, `spec-implementation-plan`) for 003 W2 dogfood with this PRD (REQ-40); no version-bump exit gate | `sha256:20e2e8c26fdeff1bc21331fcef95df244cd5ddbabed7d005216c8c80bf54ea92` | `INIT-GATEFLOW-004-prayog-skills.md` (pin/content; may be thin) | High |

## 3. Deferred repositories

| Service | Repo | Reason | Revisit condition |
|---------|------|--------|-------------------|
| — | — | None deferred for 004 product exit | — |

## 4. Transitively affected repositories

| Service | Repo | Depends on | Potential impact | Disposition |
|---------|------|------------|------------------|-------------|
| launchpad | drivestream-lab/launchpad | — | Greenfield factory unchanged; no Mission Control scaffolding | monitor — not affected (product) |
| gateflow-ops | drivestream-lab/gateflow-ops | gateflow (`links.upstream`) | Consumes run/metrics/wave APIs + optional REQ-39 extensions | affected (primary) |

## 5. Not affected

| Service | Repo | Reason |
|---------|------|--------|
| prayog-meta | drivestream-lab/prayog-meta | Hosts PRD, outline, validation/resolution reports, this impact map; not an eng delivery target for Mission Control runtime |
| launchpad | drivestream-lab/launchpad | No greenfield / harness-install product work in 004 (Non-Goals; Launchpad ownership unchanged) |

## 6. Cross-repository contracts

| Contract ID | Provider repo | Consumer repo | Capability | Owner | Status |
|-------------|---------------|---------------|------------|-------|--------|
| CTR-01 | gateflow | gateflow-ops | Existing run / metrics / wave-start / board APIs for operate + cockpit (REQ-35–37, REQ-39 consume) | prayog-pe-team | unchanged — primary consume |
| CTR-02 | gateflow | gateflow-ops | Fleet onboarding records + clearer wave summaries if existing APIs insufficient (REQ-39) | prayog-pe-team | new — supporting delta |
| CTR-03 | prayog-skills | gateflow-ops | Pinned workflow projection for process map (REQ-36, REQ-38) | prayog-pe-team | unchanged — read pin |
| CTR-04 | prayog-skills | gateflow / programme | Engg spec-lane skills `dispatch: orchestrated` for 003 W2 dogfood (REQ-40) | prayog-pe-team | changed — pin content (paired with 003 W2) |
| CTR-05 | gateflow | GitHub (ForgeClient) | Scorecard GitHub access + Open PR links (REQ-33, REQ-36); reused | prayog-pe-team | unchanged |

## 7. Dependency and build order

```text
INIT-GATEFLOW-001–002 finished; INIT-GATEFLOW-003 control plane + live Cursor available
  (003 W2 engg spec-lane prove-it may still be open — pairs with this PRD)

  → gateflow supporting API delta (REQ-39) — only where ops cannot meet cockpit/onboarding from existing APIs
  → gateflow-ops Mission Control W0–W3 (REQ-32–38) — primary product
  → prayog-skills engg spec-lane pin (REQ-40) — supporting; before / with dogfood prove-it (parallel to ops W3)
```

| Repo | Depends on | Reason |
|------|------------|--------|
| gateflow-ops | gateflow | Catalog upstream; BFF consumes APIs (CTR-01/02) |
| gateflow-ops | prayog-skills | Process map projection from pin (CTR-03) |
| gateflow | prayog-skills | Orchestration honors pin; dogfood uses engg spec-lane orchestrated nodes |
| prayog-skills | none (SSOT) | Supporting pin `dispatch` content edit |
| launchpad | — | Not in delivery chain |

## 8. Revision diff

*Omitted — map revision 1.*

## 9. Downstream ripple ledger

| Repo | In-flight artifact | Required action | Reason | Owner | Blocking |
|------|--------------------|-----------------|--------|-------|----------|
| gateflow-ops | none (spec not yet opened) | **open** after map approval | First Mission Control product delivery | prayog-pe-team | no |
| gateflow | none for 004 API delta | **open** after map approval (thin supporting) | REQ-39 only if gaps proven | prayog-pe-team | no |
| prayog-skills | pin live; engg spec-lane may still need `orchestrated` for dogfood | **open** (supporting pin) after map approval | REQ-40 / 003 W2 companion | prayog-pe-team | no |
| launchpad | none | continue | Monitor only | prayog-pe-team | no |

## 10. Open questions

| ID | Lane | Question | Owner | Blocking | Required by | Default if deferred | Status |
|----|------|----------|-------|----------|-------------|---------------------|--------|
| IM-01 | PE | Exact scorecard probe implementation per category (PRD OQ #1) | PE | no | gateflow-ops W0/W1 | Categories in PRD §2 normative; probes in ops/gateflow spec | open |
| IM-02 | PE | Sign-in mechanism — session vs token bridge, invite/bootstrap (PRD OQ #2) | PE | no | gateflow-ops W0 | Product: sign-in required; thin identity; shape in ops spec | open |
| IM-03 | PE | Field names / retention for human-wait and unattended aggregates (PRD OQ #3) | PE | no | gateflow-ops W2/W3 | Product outcomes in PRD §4 Operator-experience numbers | open |
| IM-04 | PM | Where to land CHG-09 sibling edit on `prd/INIT-GATEFLOW-003.md` (forward pointer to 004) — on open PR [#11](https://github.com/drivestream-lab/prayog-meta/pull/11) vs this 004 PR | PM | no | meta PR create | Prefer PR #11 or small follow-up; do not block 004 map | open |
| IM-05 | PE | Confirm which REQ-39 endpoints are missing vs already served by 001–003 APIs before opening gateflow supporting work | PE | no | gateflow supporting start | Start ops against existing APIs; open gateflow delta only when gaps proven | open |

## 11. PR readiness handoff

| Item | Value |
|------|-------|
| Verdict | **PR READY** |
| Collision detection | **no-collision** (workspace note below) |
| Collision evidence | No `Impact-Map-INIT-GATEFLOW-004`; no open/closed PR titled INIT-GATEFLOW-004; open PR [#11](https://github.com/drivestream-lab/prayog-meta/pull/11) is INIT-GATEFLOW-003 (**unrelated**); open PR [#10](https://github.com/drivestream-lab/prayog-meta/pull/10) is INIT-GATEFLOW-002 (**unrelated**). Local checkout is currently `chore/INIT-GATEFLOW-003-prd` with 004 WIP files — **do not** push 004 onto PR #11 |
| Human resolution | none |
| Resolution completed | n/a |
| Existing PR | none for 004 |
| Proposed branch | `chore/INIT-GATEFLOW-004-prd` |
| Proposed base | `develop` |
| Proposed title | `[INIT-GATEFLOW-004] PRD — Gateflow Mission Control` |
| Files to commit | `prd/INIT-GATEFLOW-004.md`, `prd/INIT-GATEFLOW-004-outline.md`, `prd/reports/Impact-Map-INIT-GATEFLOW-004.md`, `prd/reports/Validation-Report-INIT-GATEFLOW-004.md`, `prd/reports/Resolution-INIT-GATEFLOW-004.md`; optionally `planning/gateflow-programme-vision.md` if included in this initiative’s vision bump; **exclude** `prd/INIT-GATEFLOW-003.md` from 004 PR until IM-04 resolved |
| Reviewer | @drivestream-lab/prayog-pe-team |
| Initial Gate 1 label | `impact-map-pending` |
| Additional invalidation label | none |
| Blocking items | none (IM-01–05 → PE/PM; non-blocking) |

**No GitHub side effects have occurred.** Ask the user whether to create or update the Draft PR. Continue only after explicit authorization.

### Proposed Draft PR body

```markdown
## Product change

INIT-GATEFLOW-004 delivers **Gateflow Mission Control (v0)** in **gateflow-ops**: thin ops-user identity, strict scorecard onboarding of harnessed repos, fleet home, start/inspect waves from the UI, high-bar run cockpit (process map + timeline + full log pane + GitHub jump), and efficacy visibility while collecting numbers for later lift decisions. **Primary: gateflow-ops.** **Supporting: gateflow** (API gaps only, REQ-39) and **prayog-skills** (engg spec-lane pin for 003 W2 dogfood, REQ-40). Launchpad greenfield stays out of scope. Checkpoints stay on; no auto-merge.

## Impact-map summary

- Revision: **1**
- PRD digest: `sha256:90fff56a5c2699390177d573e31df1d34f8980aa6553d4cce17128877af2882f`
- Scope digest (gateflow-ops): `sha256:682d055f5a0ee565202376bbcfb8c6e43d88464181df78ab1d1f6abbd6c26e87`
- Scope digest (gateflow): `sha256:1264d9ab2feb688091974445d7a9f7326f5de1311a658aa5a46bf4717d23c769`
- Scope digest (prayog-skills): `sha256:20e2e8c26fdeff1bc21331fcef95df244cd5ddbabed7d005216c8c80bf54ea92`
- Affected repos: drivestream-lab/gateflow-ops, drivestream-lab/gateflow, drivestream-lab/prayog-skills
- Deferred repos: none
- Not affected: launchpad (product), prayog-meta (host)
- Blocking questions: none (IM-01–05 non-blocking)
- Artifact: `prd/reports/Impact-Map-INIT-GATEFLOW-004.md`

## Gate 1 — engineering handoff readiness

- [ ] Product/domain blocking questions resolved in committed artifacts
- [ ] Impact-map scope and dependency order complete
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
initiative: INIT-GATEFLOW-004
map_revision: 1
meta_pr_head_sha: {SHA after this artifact is committed}
prd_digest: sha256:90fff56a5c2699390177d573e31df1d34f8980aa6553d4cce17128877af2882f
artifact: prd/reports/Impact-Map-INIT-GATEFLOW-004.md
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
    path: prd/reports/Impact-Map-INIT-GATEFLOW-004.md
    digest: sha256:956b09facd54f78b8904a6a7c1faf35b0d522077ad7d140237f372e24c174bed
  blockers: []
  signals:
    map_revision: 1
    previous_revision: null
    prd_digest: sha256:90fff56a5c2699390177d573e31df1d34f8980aa6553d4cce17128877af2882f
    affected_repos:
      - drivestream-lab/gateflow-ops
      - drivestream-lab/gateflow
      - drivestream-lab/prayog-skills
    deferred_repos: []
    collision_detection: no-collision
    pr_ready: true
    existing_pr: none
    proposed_branch: chore/INIT-GATEFLOW-004-prd
    proposed_base: develop
  next_candidates:
    - prd-pr-action
  human_checkpoint: true
  external_action: true
```
