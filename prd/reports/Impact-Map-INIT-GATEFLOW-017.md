---
schema_version: 1
initiative: INIT-GATEFLOW-017
map_revision: 1
source_prd: prd/INIT-GATEFLOW-017.md
source_prd_digest: sha256:c0fe55040928a13976133edde5cf71f0524815c17c0a8de79173ed3fa0657f67
previous_revision: null
previous_artifact_commit: null
change_reason: Initial impact map — one human, many programmes, one login; enter-then-grant; delete 014 bind; purge 016 invite
material_change: true
generated_at: 2026-08-13T12:50:44Z
---

# Impact map — INIT-GATEFLOW-017 — revision 1

> This file is generated locally before PR creation and becomes the scope source
> of truth when committed. Effective approval is derived from a tech-lead
> GitHub APPROVED review on the exact current meta PR head SHA; dynamic PR state
> is not stored in this artifact's frontmatter.

## 1. Source and revision

| Field | Value |
|-------|-------|
| PRD | `prd/INIT-GATEFLOW-017.md` |
| PRD digest | `sha256:c0fe55040928a13976133edde5cf71f0524815c17c0a8de79173ed3fa0657f67` |
| Map revision | `1` |
| Previous revision | none |
| Previous artifact commit | none |
| Change reason | Initial map — split 014 create+bind into enter-identity then grant; many programmes per login; suspend/password; purge invite; console for both roles |
| Material change | yes — first map for this initiative |

**Catalog used:** `config/service-catalog-drivestream-lab.yaml` (no `config/service-catalog.yaml` in this workspace; `AGENTS.md` points at `config/service-catalog*.yaml`). Catalog services have no `owns` / `depends_on` fields; matching used `description` + `links.upstream` (gateflow-ops → gateflow).

**Identity collision:** `no-collision` — no `Impact-Map-INIT-GATEFLOW-017` existed; `gh pr list --state all` for `INIT-GATEFLOW-017` / `GATEFLOW-017` returned none; no remote branch matching `*017*`. Local branch `chore/INIT-GATEFLOW-017-prd` tracks `origin/develop` and holds this initiative's untracked files — it is the proposed delivery branch, not a competing PR. Prior maps 001–016 / PRAYOG-SKILLS-* are different initiatives (`unrelated` by id).

## 2. Affected repositories

| Service | Repo | Team | Scope summary | Scope digest | Spec to create | Confidence |
|---------|------|------|---------------|--------------|----------------|------------|
| gateflow | drivestream-lab/gateflow | @drivestream-lab/prayog-pe-team | Identity APIs: enter/find (name, unique email, password write-only); grant/detach/many programmes; who-can-enter; suspend/unsuspend; password-set kills prior sign-in; fail-closed unknown identity/programme and wrong actor; delete 014 create+bind; sign-in + resolve granted programmes (not 1:1 JWT bind). Programme onboard/catalogue APIs stay (CAP-06 / A1). CAP-01–06 / REQ-01–30 / CTR-01–04. | `sha256:3d39ee6d5947bcd18c3e0b46de16709b3fdddeefeb0e8d5bbae8ecf98b1e5834` | `INIT-GATEFLOW-017-gateflow.md` | High |
| gateflow-ops | drivestream-lab/gateflow-ops | @drivestream-lab/prayog-pe-team | Console: `platform_admin` enters/finds identities, grants/detaches, sees who-can-enter, suspends, sets password, onboards programme and shows catalogue; `tenant_admin` signs in once, sees granted programmes, enters one, keeps 013/016 delivery **minus invite**; zero-programme empty state; no roster. Purge 016 invite UI. CAP-01–06 / REQ-01–30 / CTR-01–04 consumer. | `sha256:13cee9aae41718fd4ad8655d77738042761b0e45db7eefc898a693c42c2fb987` | `INIT-GATEFLOW-017-gateflow-ops.md` | High |

**H2 payload (canonical):**

```text
repo=drivestream-lab/gateflow
status=affected
capabilities=CAP-01,CAP-02,CAP-03,CAP-04,CAP-05,CAP-06,REQ-01,REQ-02,REQ-03,REQ-04,REQ-05,REQ-06,REQ-07,REQ-08,REQ-09,REQ-10,REQ-11,REQ-12,REQ-13,REQ-14,REQ-15,REQ-16,REQ-17,REQ-18,REQ-19,REQ-20,REQ-21,REQ-22,REQ-23,REQ-24,REQ-25,REQ-26,REQ-27,REQ-28,REQ-29,REQ-30
contracts=CTR-01,CTR-02,CTR-03,CTR-04
depends_on=
scope=Split 014 create+bind into enter-identity then grant; unique email login; name; factory list; many programme grants; detach; suspend/unsuspend; password-set kills prior sign-in; fail-closed unknown identity/programme and wrong actor; delete 014 bind; keep 014 programme onboard APIs; no dual-model JWT 1:1 bind
```

```text
repo=drivestream-lab/gateflow-ops
status=affected
capabilities=CAP-01,CAP-02,CAP-03,CAP-04,CAP-05,CAP-06,REQ-01,REQ-02,REQ-03,REQ-04,REQ-05,REQ-06,REQ-07,REQ-08,REQ-09,REQ-10,REQ-11,REQ-12,REQ-13,REQ-14,REQ-15,REQ-16,REQ-17,REQ-18,REQ-19,REQ-20,REQ-21,REQ-22,REQ-23,REQ-24,REQ-25,REQ-26,REQ-27,REQ-28,REQ-29,REQ-30
contracts=CTR-01,CTR-02,CTR-03,CTR-04
depends_on=drivestream-lab/gateflow
scope=Console for platform_admin: enter/find identities, grant/detach, who-can-enter, suspend, set password, programme onboard and show catalogue; tenant_admin: sign in once, see granted programmes, enter one, keep 013/016 delivery minus invite; zero-programme empty state; no roster; purge 016 invite UI
```

## 3. Deferred repositories

| Service | Repo | Reason | Revisit condition |
|---------|------|--------|-------------------|
| — | — | None deferred. 016 excluded the platform-admin console; this INIT puts `platform_admin` identity/grant/onboard in the console. Encrypting secrets, SSO, delete-identity, and extra roles stay non-goals, not deferred scope. | — |

## 4. Transitively affected repositories

| Service | Repo | Depends on | Potential impact | Disposition |
|---------|------|------------|------------------|-------------|
| gateflow-ops | drivestream-lab/gateflow-ops | gateflow (`links.upstream`) | Consumes CTR-01–04; identity/grant/sign-in change on gateflow | **affected** — already in §2; not monitor-only |
| launchpad | drivestream-lab/launchpad | — | No CLI/harness change; programme meta onboard stays existing gateflow paths (A1) | not affected for delivery |

## 5. Not affected

| Service | Repo | Reason |
|---------|------|--------|
| prayog-skills | drivestream-lab/prayog-skills | No `workflow.yaml` / sdd-delivery / pin change. Membership is a Gateflow product model, not a skill-contract change. |
| launchpad | drivestream-lab/launchpad | Factory CLI / harness sync / playbook — not identity, grant, or console. |
| prayog-meta | drivestream-lab/prayog-meta | Hosts this PRD, outline, and impact map; not an app runtime target. 014-style vision/ADR rewrite is not a 017 CAP (OQ-04 is a later 014 *document* update). |

## 6. Cross-repository contracts

| Contract ID | Provider repo | Consumer repo | Capability | Owner | Status |
|-------------|---------------|---------------|------------|-------|--------|
| CTR-01 | gateflow | gateflow-ops | CAP-01 / CAP-03 — enter, list/search identities; suspend; unsuspend; set password (REQ-01–05, REQ-12–14, REQ-25, REQ-29–30) | @drivestream-lab/prayog-pe-team | **new** |
| CTR-02 | gateflow | gateflow-ops | CAP-02 — grant, detach, who-can-enter, many programmes (REQ-06–11, REQ-24, REQ-28) | @drivestream-lab/prayog-pe-team | **new** (replaces 014 create+bind attach) |
| CTR-03 | gateflow | gateflow-ops | CAP-03 / CAP-04 — sign-in; suspend and password-set end the prior sign-in (REQ-12–14, REQ-18) | @drivestream-lab/prayog-pe-team | **changed** vs 014 login (email unique; not programme-bound 1:1) |
| CTR-04 | gateflow | gateflow-ops | CAP-04 / CAP-05 — resolve programmes this sign-in may enter; wrong-actor refuse; no factory list for `tenant_admin` (REQ-16–17, REQ-22–23, REQ-26) | @drivestream-lab/prayog-pe-team | **new** |

014 REQ-15 create+bind and 014 REQ-06 programme-bound JWT are product-superseded (017 REQ-21 / REQ-09 / REQ-16; OQ-04). 016 CAP-A invite (`POST .../tenants/{id}/users`) is product-superseded (017 REQ-20; OQ-02).

## 7. Dependency and build order

```text
gateflow (CTR-01–04 provider — identity/grant/sign-in; delete 014 bind)
  → gateflow-ops (CTR-01–04 consumer — console; purge invite; programme enter)
```

| Repo | Depends on | Reason |
|------|------------|--------|
| gateflow | none this INIT (014/013 APIs already live) | Owns identity, grant, session, fail-closed; onboard APIs kept |
| gateflow-ops | gateflow | Every console act is a CTR-01–04 call; invite UI cannot ship against a live 014 bind |

**Cross-repo sequencing:** do not ship ops identity/grant screens while gateflow still creates membership via 014 create+bind (kill line). Internal wave order is an engineering-plan concern after Gate 1; default provider then consumer.

## 8. Revision diff

_Omit for revision 1._

## 9. Downstream ripple ledger

| Repo | In-flight artifact | Required action | Reason | Owner | Blocking |
|------|--------------------|-----------------|--------|-------|----------|
| gateflow | none (no INIT-017 app spec yet) | **open** after Gate 1 approval | First map; primary API delivery | prayog-pe-team | no |
| gateflow-ops | INIT-GATEFLOW-016 PRD merged on `develop` (PR #41); invite is 016 CAP-A | **open** after Gate 1; **do not implement 016 invite** (REQ-20) | Console + invite purge; 016 delivery otherwise kept | prayog-pe-team | no |
| prayog-skills | none | **continue** | Not affected (§5) | prayog-pe-team | no |
| launchpad | none | **continue** | Not affected (§5) | prayog-pe-team | no |
| prayog-meta | this PRD/map (untracked on current branch) | **open** (content in meta PR only) | Hosts PRD/map; not an app spec | prayog-pm-team / prayog-pe-team | no |

## 10. Open questions

| ID | Lane | Question | Owner | Blocking | Required by | Default if deferred | Status |
|----|------|----------|-------|----------|-------------|---------------------|--------|
| IM-01 | PE | Session vs 014 programme-bound JWT: one sign-in then **enter** a granted programme (CTR-03/04) vs minting a per-programme token | PE | no | spec-draft | Product: one identity sign-in; programme is authorization, not a second login. Schema is engineering (014 OQ-5 superseded for 1:1 bind). | open |
| IM-02 | PE | INIT-GATEFLOW-016 invite (CAP-A / REQ-02) vs 017 REQ-20 — 016 PRD is merged | PE | no | Before ops W0 identity screens | 017 wins; do not ship invite; OQ-02 is the 016 document follow-on | open |
| IM-03 | PE | Wave split: gateflow API waves vs gateflow-ops console waves, given kill line (no identity screens on 1:1 bind) | PE | no | spec-implementation-plan | Sequential: gateflow enter+grant live, then ops screens | open |

PRD OQ-01 (leftover bind rows), OQ-03 (seeded `platform_admin` not grantable), OQ-02/OQ-04 (016/014 doc updates) stay on the PRD; none block Gate 1.

## 11. PR readiness handoff

| Item | Value |
|------|-------|
| Verdict | **PR READY** |
| Collision detection | no-collision |
| Collision evidence | Searched `Impact-Map-INIT-GATEFLOW-017*` (none pre-existing), `gh pr list --state all` for `INIT-GATEFLOW-017` and `GATEFLOW-017` (empty), `git ls-remote --heads origin` for `*017*` (none). Local `chore/INIT-GATEFLOW-017-prd` tracks `origin/develop` with untracked 017 files — proposed delivery branch, not a second PR. |
| Human resolution | none |
| Resolution completed | n/a |
| Existing PR | none |
| Proposed branch | `chore/INIT-GATEFLOW-017-prd` |
| Proposed base | `develop` |
| Proposed title | `[INIT-GATEFLOW-017] PRD — One human, many programmes, one login` |
| Files to commit | `prd/INIT-GATEFLOW-017.md`, `prd/INIT-GATEFLOW-017-outline.md`, `prd/reports/Impact-Map-INIT-GATEFLOW-017.md`, `prd/reports/Validation-Report-INIT-GATEFLOW-017.md`, `prd/reports/Resolution-INIT-GATEFLOW-017.md`, `prd/reports/Update-Summary-INIT-GATEFLOW-017.md` |
| Reviewer | @drivestream-lab/prayog-pe-team |
| Initial Gate 1 label | `impact-map-pending` |
| Additional invalidation label | none |
| Blocking items | none — IM-01–IM-03 are non-blocking engineering sequencing/schema items |

**No GitHub side effects have occurred.** Ask whether to publish via
`/commit-workspace` then `/open-draft-pr` (explicit authorization required).

### Proposed Draft PR body

```markdown
## Product change
One human, many programmes, one login. `platform_admin` enters an identity
(name, email, password) with no programme, then grants that existing identity
entry to one or more programmes. 014 create+bind is deleted. 016 invite is
purged. `tenant_admin` signs in once and enters each granted programme.
Affected delivery: **gateflow** (identity/grant APIs) then **gateflow-ops**
(console).

## Impact-map summary
- Revision: 1
- PRD digest: `sha256:c0fe55040928a13976133edde5cf71f0524815c17c0a8de79173ed3fa0657f67`
- Affected repos: drivestream-lab/gateflow, drivestream-lab/gateflow-ops
- Deferred repos: none
- Blocking questions: none (IM-01–IM-03 are non-blocking)
- Artifact: `prd/reports/Impact-Map-INIT-GATEFLOW-017.md`

## Gate 1 — engineering handoff readiness
- [ ] Product/domain blocking questions are resolved in committed artifacts
- [ ] Impact-map scope and dependency order are complete
- [ ] PRD digest and map revision match this PR head
- [ ] PE/tech lead has reviewed the exact current head
- [ ] PE acknowledges 014 bind deletion and 016 invite purge (kill line)

Requested reviewer: @drivestream-lab/prayog-pe-team
Initial label: `impact-map-pending`
```

## 12. Approval request (after Draft PR creation)

Tech lead must review this artifact on the meta PR and submit GitHub
**Approve** on the exact PR head SHA using:

```text
Impact map approved
initiative: INIT-GATEFLOW-017
map_revision: 1
meta_pr_head_sha: {SHA after this artifact is committed}
prd_digest: sha256:c0fe55040928a13976133edde5cf71f0524815c17c0a8de79173ed3fa0657f67
artifact: prd/reports/Impact-Map-INIT-GATEFLOW-017.md
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
    path: prd/reports/Impact-Map-INIT-GATEFLOW-017.md
    digest: sha256:c0fe55040928a13976133edde5cf71f0524815c17c0a8de79173ed3fa0657f67
  blockers: []
  signals:
    map_revision: 1
    source_prd_digest: sha256:c0fe55040928a13976133edde5cf71f0524815c17c0a8de79173ed3fa0657f67
    pr_ready: true
    collision_detection: no-collision
    affected_repos:
      - drivestream-lab/gateflow
      - drivestream-lab/gateflow-ops
  next_candidates:
    - prd-pr-action
  human_checkpoint: false
  external_action: true
  forge:
    action: open_draft_pr
    draft: true
    apply_labels:
      - impact-map-pending
    title: "[INIT-GATEFLOW-017] PRD — One human, many programmes, one login"
    body_path: prd/reports/Impact-Map-INIT-GATEFLOW-017.md
```
