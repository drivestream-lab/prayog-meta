---
schema_version: 1
initiative: INIT-GATEFLOW-014
map_revision: 1
source_prd: prd/INIT-GATEFLOW-014.md
source_prd_digest: sha256:e8c5103ea55a16823bf6a4e5c10bfc34be8e9f94efee12f6a3da69722fc3ea3e
previous_revision: null
previous_artifact_commit: null
change_reason: Initial impact map — JWT-only product edge, Programme entity + per-programme PAT, platform agent DB catalogue, refuse/delete shared-secret doors, wipe 012/013 lab tenants, meta vision/ADR supersession
material_change: true
generated_at: 2026-08-10T10:40:15Z
---

# Impact map — INIT-GATEFLOW-014 — revision 1

> This file is generated locally before PR creation and becomes the scope source
> of truth when committed. Effective approval is derived from a tech-lead
> GitHub APPROVED review on the exact current meta PR head SHA; dynamic PR state
> is not stored in this artifact's frontmatter.

## 1. Source and revision

| Field | Value |
|-------|-------|
| PRD | `prd/INIT-GATEFLOW-014.md` |
| PRD digest | `sha256:e8c5103ea55a16823bf6a4e5c10bfc34be8e9f94efee12f6a3da69722fc3ea3e` |
| Map revision | `1` |
| Previous revision | none |
| Previous artifact commit | none |
| Change reason | Initial map — one identity (Gateflow-issued user JWT) for product APIs; Programme validate-then-create with per-programme GitHub PAT; platform agent DB catalogue + lane defaults; refuse old shared-secret doors in W2 and delete in W3; wipe 012/013 lab tenants; prove absence + rewrite teaching surfaces; prayog-meta vision/ADR supersession (REQ-39). Supersedes INIT-GATEFLOW-012 G3. |
| Material change | yes — first map |

**Catalog used:** `config/service-catalog-drivestream-lab.yaml` (no `config/service-catalog.yaml` in this workspace; `AGENTS.md` points at `config/service-catalog*.yaml`). Catalog services have no `owns` / `depends_on` fields; matching used `description` + `links.upstream` (gateflow-ops → gateflow).

**Identity collision:** `no-collision` — no `Impact-Map-INIT-GATEFLOW-014` exists; no local/remote branch matching `*014*` / `INIT-GATEFLOW-014`; `gh pr list --state all` for `INIT-GATEFLOW-014` / `GATEFLOW-014` returns no results; working-tree artifacts for this initiative are untracked on `develop`, not yet on any branch/PR. Prior maps for 001–013 / PRAYOG-SKILLS-* are different initiatives (`unrelated` by id).

## 2. Affected repositories

| Service | Repo | Team | Scope summary | Scope digest | Spec to create | Confidence |
|---------|------|------|---------------|--------------|----------------|------------|
| gateflow | drivestream-lab/gateflow | @drivestream-lab/prayog-pe-team | Primary delivery: seed + login JWT edge (CAP-01); Programme validate-then-create with per-programme PAT (CAP-02); `tenant_admin` attach + list (CAP-03); platform agent **DB** catalogue + per-lane defaults, no env-Cursor product path (CAP-04); operate 012/013 callables under JWT (CAP-05); fail-closed + **refuse** programme token / tenant bearer / open register once JWT edge is live — no dual-auth window (CAP-06); dead-door **deletion** + wipe cutover (CAP-07); prove absence + rewrite gateflow teaching/verify surfaces (CAP-08 except REQ-39). CAP-01…08 / REQ-01…38, REQ-40…47. Breaking / greenfield. No `gateflow-ops` UI; no `prayog-skills` contract change. | `sha256:53bb4c4f204888154afc40385c7e717f92d39f98463fcc6e97694e465a0ac9ef` | `INIT-GATEFLOW-014-gateflow.md` | High |
| prayog-meta | drivestream-lab/prayog-meta | @drivestream-lab/prayog-pm-team, @drivestream-lab/prayog-pe-team | Supporting content delivery (REQ-39 / CAP-08): vision (and ADR language hosted or referenced from meta) supersedes programme-token / tenant-bearer product-edge teaching to JWT-only + secret split; note INIT-GATEFLOW-012 G3 and ADR-005/011 supersession. No runtime service change. Same meta PR lane as this PRD/map (W4 window). | `sha256:0b594aadf73b9800bb6a6a90a0f6ac24c73fe3c1e07498ff248039fed312c603` | none — planning/vision (+ ADR citations) in meta; not an app product-spec | High |

**H2 payload (canonical):**

```text
repo=drivestream-lab/gateflow
status=affected
capabilities=CAP-01,CAP-02,CAP-03,CAP-04,CAP-05,CAP-06,CAP-07,CAP-08,REQ-01,REQ-02,REQ-03,REQ-04,REQ-05,REQ-06,REQ-07,REQ-08,REQ-09,REQ-10,REQ-11,REQ-12,REQ-13,REQ-14,REQ-15,REQ-16,REQ-17,REQ-18,REQ-19,REQ-20,REQ-21,REQ-22,REQ-23,REQ-24,REQ-25,REQ-26,REQ-27,REQ-28,REQ-29,REQ-30,REQ-31,REQ-32,REQ-33,REQ-34,REQ-35,REQ-36,REQ-37,REQ-38,REQ-40,REQ-41,REQ-42,REQ-43,REQ-44,REQ-45,REQ-46,REQ-47
contracts=CTR-01
depends_on=
scope=JWT-only product edge (seed+login), Programme validate-then-create with per-programme PAT, tenant_admin attach, platform agent DB catalogue with lane defaults and no env-Cursor auth path, refuse then delete shared-secret doors, wipe 012/013 lab tenants, prove absence and rewrite gateflow teaching surfaces; supersedes 012 G3; REQ-39 lives in prayog-meta; no gateflow-ops UI; no prayog-skills contract change
```

```text
repo=drivestream-lab/prayog-meta
status=affected
capabilities=CAP-08,REQ-39
contracts=CTR-02
depends_on=
scope=Vision and ADR updates that supersede programme-token / tenant-bearer product-edge language to JWT-only plus secret-split truth, noting INIT-GATEFLOW-012 G3 and ADR-005/011 supersession; no app runtime change
```

## 3. Deferred repositories

| Service | Repo | Reason | Revisit condition |
|---------|------|--------|-------------------|
| gateflow-ops | drivestream-lab/gateflow-ops | Ops dashboard / login screens explicitly out of scope (PRD Non-Goals; related INIT-GATEFLOW-004). This INIT ships API-only login + JWT product edge that a future BFF/UI would call. | After this INIT ships; when 004 / ops console adopts the JWT caller story |

## 4. Transitively affected repositories

| Service | Repo | Depends on | Potential impact | Disposition |
|---------|------|------------|-------------------|-------------|
| gateflow-ops | drivestream-lab/gateflow-ops | gateflow (`links.upstream`) | Breaking product-edge auth change (CTR-01): any BFF still using programme token or tenant bearer will fail once W2 refuses those doors | monitor — deferred (§3); must not ship dual-auth to keep ops working |
| launchpad | drivestream-lab/launchpad | — (prior 013 read contract only) | No new Launchpad CLI/contract work this INIT; Programme validate-then-create still reads meta catalogue via existing gateflow paths | not affected for delivery — no code change expected |
| GitHub (external) | — | — | Per-programme PAT validation + forge/git continue to use stored programme PAT (not caller JWT) | outbound dependency of gateflow CAP-02/CAP-05 — not a catalog service |

## 5. Not affected

| Service | Repo | Reason |
|---------|------|--------|
| prayog-skills | drivestream-lab/prayog-skills | PRD explicitly expects **no** `workflow.yaml` / sdd-delivery contract change; pin explicit-vs-automated forge policy unchanged (REQ-27). Auth + entity model live entirely in gateflow. |
| launchpad | drivestream-lab/launchpad | Description owns factory CLI / harness sync / playbook — not Gateflow product auth, Programme entity, or agent catalogue. No new consumer contract this INIT beyond what 013 already established. |

## 6. Cross-repository contracts

| Contract ID | Provider repo | Consumer repo | Capability | Owner | Status |
|-------------|---------------|---------------|------------|-------|--------|
| CTR-01 | gateflow | product callers (lab scripts, future gateflow-ops, humans) | Product API Bearer contract: **Gateflow-issued user JWT only** for Appendix C families; programme service token and tenant bearer refused in W2 and deleted in W3 (CAP-05/06/07, REQ-04, REQ-29–34). Breaking. | prayog-pe-team | **changed** — replaces shared-secret product doors; supersedes 012 G3 |
| CTR-02 | prayog-meta | programme (humans / PE) | Vision/ADR product-truth language for access + secret split (CAP-08, REQ-39) | prayog-pm-team / prayog-pe-team | **changed** — teaching supersession in W4 |

## 7. Dependency and build order

```text
INIT-GATEFLOW-012 + INIT-GATEFLOW-013 (shipped — callables remain the behavior substrate)
  → gateflow W0 (seed platform_admin + JWT mint/login — CAP-01 / REQ-01–07,43)
      → gateflow W1 (Programme validate-then-create + tenant_admin + DB agent catalogue + lane defaults — CAP-02…04 / REQ-08–22,40–42,44–45,47)
          → gateflow W2 (operate under JWT + refuse old doors — CAP-05/06 / REQ-23–33)  [no dual-auth window]
              → gateflow W3 (delete dead doors + wipe cutover — CAP-07 / REQ-34–35,46)
                  → gateflow W4 (prove + rewrite gateflow teaching surfaces — REQ-36–38)
                  → prayog-meta W4 (vision/ADR supersession — REQ-39)  [parallel with gateflow W4]
```

| Repo | Depends on | Reason |
|------|------------|--------|
| gateflow | INIT-GATEFLOW-012/013 (merged) | Reuses shipped callables; changes who may call them, where secrets live, Programme entity |
| prayog-meta | gateflow W2+ product truth stable enough to document | REQ-39 should not re-teach old doors; schedule in same window as W4 teaching rewrite |
| gateflow-ops | — | Deferred; must not unblock dual-auth |

**No cross-repo eng build gate** (unlike INIT-GATEFLOW-012’s prayog-skills contract PR). Gateflow implementation can start after Gate 1; meta REQ-39 is content in this meta workspace, not a blocking app-repo PR.

## 8. Revision diff

_Omit for revision 1._

## 9. Downstream ripple ledger

| Repo | In-flight artifact | Required action | Reason | Owner | Blocking |
|------|--------------------|-----------------|--------|-------|----------|
| gateflow | none (no INIT-014 app spec yet) | **open** after Gate 1 approval | First map; primary eng delivery | prayog-pe-team | no |
| prayog-meta | this PRD/map (untracked on `develop`) | **open** (content in meta PR; REQ-39 in W4) | Supporting affected scope | prayog-pm-team / prayog-pe-team | no |
| gateflow-ops | none | **hold** | Deferred (§3) | prayog-pe-team | no |
| prayog-skills | none | **continue** | Not affected (§5) | prayog-pe-team | no |
| launchpad | none | **continue** | Not affected for delivery (§5) | prayog-pe-team | no |

## 10. Open questions

| ID | Lane | Question | Owner | Blocking | Required by | Default if deferred | Status |
|----|------|----------|-------|----------|-------------|---------------------|--------|
| IM-01 | PE | Exact JWT claim schema (issuer/audience/role/programme binding) and password/credential store for login API (**PRD OQ-5**) | PE | no | gateflow W0 design | Product requires Gateflow-issued user JWT with role + `tenant_admin` programme binding only — eng owns schema | open |
| IM-02 | PE | Whether health/internal/webhook paths remain on the JWT public allowlist exactly as today (**PRD OQ-6**) | PE | no | gateflow W0/W2 | Webhooks stay signature-based (REQ-28); Appendix C product APIs JWT-only | open |
| IM-03 | PE | Where ADR-005 / ADR-011 live for supersession notes (gateflow ADRs vs meta citations) while REQ-39 commits product truth in prayog-meta vision | PE/PM | no | W4 | Update meta vision; cite/supersede ADRs wherever they currently live | open |
| IM-04 | PM | Direct notice that this INIT **supersedes INIT-GATEFLOW-012 G3** (JWT parked / tenant bearer as product auth) and wipes lab 012/013 tenant rows | PM | no | Before gateflow W2/W3 cutover | Proceed — PRD already locks wipe + supersession; still worth an explicit team note | open |

## 11. PR readiness handoff

| Item | Value |
|------|-------|
| Verdict | **PR READY** |
| Collision detection | no-collision |
| Collision evidence | Searched `Impact-Map-INIT-GATEFLOW-014*` (none), branches matching `*014*` / `INIT-GATEFLOW-014` (none), `gh pr list --state all` for `INIT-GATEFLOW-014` and `GATEFLOW-014` (no results); initiative files untracked on `develop` |
| Human resolution | none |
| Resolution completed | n/a |
| Existing PR | none |
| Proposed branch | `chore/INIT-GATEFLOW-014-prd` |
| Proposed base | `develop` |
| Proposed title | `[INIT-GATEFLOW-014] PRD — One identity to call Gateflow, retire shared-secret doors` |
| Files to commit | `prd/INIT-GATEFLOW-014.md`, `prd/INIT-GATEFLOW-014-outline.md`, `prd/reports/Impact-Map-INIT-GATEFLOW-014.md`, `prd/reports/Validation-Report-INIT-GATEFLOW-014.md`, `prd/reports/Resolution-INIT-GATEFLOW-014.md`, `prd/reports/Update-Summary-INIT-GATEFLOW-014.md` |
| Reviewer | @drivestream-lab/prayog-pe-team |
| Initial Gate 1 label | `impact-map-pending` |
| Additional invalidation label | none |
| Blocking items | none for Gate 1 — IM-01/IM-02 are eng design (non-blocking); IM-03/IM-04 are W4 / communication |

**No GitHub side effects have occurred.** Ask whether to publish via
`/commit-workspace` then `/open-draft-pr` (explicit authorization required).

### Proposed Draft PR body

```markdown
## Product change

Gateflow’s product APIs still teach and accept shared-secret doors (global
programme service token, tenant bearer, open register) while JWT middleware
is wired but unused. This initiative makes Gateflow-issued user JWTs the only
product identity, introduces a Programme entity with per-programme GitHub PAT
and lane defaults, moves code-agent credentials into a platform DB catalogue
(no env-Cursor product path), refuses old doors as soon as the JWT edge is
live (no dual-auth window), then deletes them and wipes 012/013 lab tenants.
Primary delivery = **gateflow**; supporting = **prayog-meta** vision/ADR
truth (REQ-39). No `prayog-skills` contract change; no `gateflow-ops` UI.

## Impact-map summary
- Revision: 1
- PRD digest: `sha256:e8c5103ea55a16823bf6a4e5c10bfc34be8e9f94efee12f6a3da69722fc3ea3e`
- Affected repos: drivestream-lab/gateflow, drivestream-lab/prayog-meta
- Deferred repos: gateflow-ops
- Blocking questions: none for Gate 1
- Artifact: `prd/reports/Impact-Map-INIT-GATEFLOW-014.md`

## Gate 1 — engineering handoff readiness
- [ ] Product/domain blocking questions are resolved in committed artifacts
- [ ] Impact-map scope and dependency order are complete
- [ ] PRD digest and map revision match this PR head
- [ ] PE/tech lead has reviewed the exact current head
- [ ] PE acknowledges breaking auth cutover (CTR-01) and wipe cutover (REQ-35)

Requested reviewer: @drivestream-lab/prayog-pe-team
Initial label: `impact-map-pending`
```

## 12. Approval request (after Draft PR creation)

Tech lead must review this artifact on the meta PR and submit GitHub
**Approve** on the exact PR head SHA using:

```text
Impact map approved
initiative: INIT-GATEFLOW-014
map_revision: 1
meta_pr_head_sha: {SHA after this artifact is committed}
prd_digest: sha256:e8c5103ea55a16823bf6a4e5c10bfc34be8e9f94efee12f6a3da69722fc3ea3e
artifact: prd/reports/Impact-Map-INIT-GATEFLOW-014.md
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
    path: prd/reports/Impact-Map-INIT-GATEFLOW-014.md
    digest: sha256:2352471915f8234d474e5034e74861cb6f09a5eeeee177788f8cd7829559ceb3
  blockers: []
  signals:
    map_revision: 1
    source_prd_digest: sha256:e8c5103ea55a16823bf6a4e5c10bfc34be8e9f94efee12f6a3da69722fc3ea3e
    pr_ready: true
    collision_detection: no-collision
    affected_repos:
      - drivestream-lab/gateflow
      - drivestream-lab/prayog-meta
  next_candidates:
    - prd-pr-action
  human_checkpoint: false
  external_action: true
  forge:
    action: open_draft_pr
    draft: true
    apply_labels:
      - impact-map-pending
    title: "[INIT-GATEFLOW-014] PRD — One identity to call Gateflow, retire shared-secret doors"
    body_path: prd/reports/Impact-Map-INIT-GATEFLOW-014.md
```
