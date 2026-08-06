---
schema_version: 1
initiative: INIT-GATEFLOW-002
map_revision: 1
source_prd: prd/INIT-GATEFLOW-002.md
source_prd_digest: sha256:2f339bae00df71e21b45e51c7551f1fb06490dd1805b96e08daca741c140332c
previous_revision: null
previous_artifact_commit: null
change_reason: Initial impact map after Draft PRD validation pass; intermediate Validation/Resolution/Update-Summary reports removed — final PRD + outline + this map only
material_change: true
generated_at: 2026-07-24T06:37:28Z
---

# Impact map — INIT-GATEFLOW-002 — revision 1

> This file is generated locally before PR creation and becomes the scope source
> of truth when committed. Effective approval is derived from a tech-lead
> GitHub APPROVED review on the exact current meta PR head SHA; dynamic PR state
> is not stored in this artifact's frontmatter.

## 1. Source and revision

| Field | Value |
|-------|-------|
| PRD | `prd/INIT-GATEFLOW-002.md` |
| PRD digest | `sha256:2f339bae00df71e21b45e51c7551f1fb06490dd1805b96e08daca741c140332c` |
| Map revision | `1` |
| Previous revision | none |
| Previous artifact commit | none |
| Change reason | Initial map; report hygiene (PRD + outline + map only) |
| Material change | yes — first map for this initiative |

## 2. Affected repositories

| Service | Repo | Team | Scope summary | Scope digest | Spec to create | Confidence |
|---------|------|------|---------------|--------------|----------------|------------|
| gateflow | drivestream-lab/gateflow | @drivestream-lab/prayog-pe-team | API-first wave start (FR-15); per-node runner/model (FR-16); AgentRunner/Notifier stubs fail-closed (FR-17/18/23); PR thread at run start (FR-19); ops run/metrics APIs (FR-20/21/22); wider board APIs as dumb forge primitives (FR-24); ForgeClient-only deploy path (FR-25/26a); laptop `gh` policy FR-26b is non-runtime; pin `v0.5.0-rc.2` consumer; primary delivery **gateflow only**; gateflow-ops UI out of scope | `sha256:3662f15e366994defaf08fa43b0a5a568eb152f6a74bcff73a0b0453dba0c901` | `INIT-GATEFLOW-002-gateflow.md` | High |

## 3. Deferred repositories

| Service | Repo | Reason | Revisit condition |
|---------|------|--------|-------------------|
| gateflow-ops | drivestream-lab/gateflow-ops | PRD delivers API readiness only; BFF/UI explicitly out of scope (US-5, Non-Goals, Repositories) | Next INIT / after board + run APIs proven on gateflow |

## 4. Transitively affected repositories

| Service | Repo | Depends on | Potential impact | Disposition |
|---------|------|------------|------------------|-------------|
| gateflow-ops | drivestream-lab/gateflow-ops | gateflow (catalog `links.upstream`) | Future BFF/UI consumes run, metrics, and board APIs | monitor — deferred |
| prayog-skills | drivestream-lab/prayog-skills | — (SSOT) | Pin `v0.5.0-rc.2` unchanged consumer; optional later board-seed → Gateflow board APIs (FR-26b / US-6) | monitor — no required delivery in this INIT |
| launchpad | drivestream-lab/launchpad | — | Worker harness sync before dispatch (inherit INIT-001) | monitor — no launchpad feature delivery |

## 5. Not affected

| Service | Repo | Reason |
|---------|------|--------|
| prayog-meta | drivestream-lab/prayog-meta | Hosts PRD + outline + this impact map; programme config for runner/model stays in **gateflow** repo |
| prayog-skills | drivestream-lab/prayog-skills | Contract pin already delivered; gateflow is read-only consumer for this INIT |
| launchpad | drivestream-lab/launchpad | Integration consumer only; no new launchpad features for 002 exit |

## 6. Cross-repository contracts

| Contract ID | Provider repo | Consumer repo | Capability | Owner | Status |
|-------------|---------------|---------------|------------|-------|--------|
| CTR-01 | prayog-skills | gateflow | Pinned workflow + delivery contract + handoff; `dispatch` on **`v0.5.0-rc.2`** | prayog-pe-team | unchanged — available |
| CTR-02 | gateflow | clients / tools | Authenticated wave-start API + run/metrics/board HTTP APIs (FR-15, FR-20, FR-21, FR-24) | prayog-pe-team | new |
| CTR-03 | gateflow | GitHub (forge) | ForgeClient: PR open/update at run start, stage comments, board ops; no `gh` in production (FR-19, FR-24, FR-25, FR-26a) | prayog-pe-team | changed — widened vs INIT-001 comments baseline |
| CTR-04 | gateflow | gateflow-ops | Stable run/metrics/board JSON contracts for future BFF | prayog-pe-team | new — consumer deferred |
| CTR-05 | launchpad | gateflow | Worker harness sync pre-dispatch | prayog-pe-team | unchanged |

## 7. Dependency and build order

```text
prayog-skills pin v0.5.0-rc.2 (delivered — INIT-PRAYOG-SKILLS-002)
  → gateflow W0 (API trigger skeleton + run list/detail; stub registration; config validation)
  → gateflow W1 (per-node runner/model; PR at run start + GitHub notifier; metrics APIs; Cursor happy path)
  → gateflow W2 (wider board APIs via ForgeClient; deploy gh-free verification)
  → gateflow-ops (deferred — next INIT / after API readiness)
```

| Repo | Depends on | Reason |
|------|------------|--------|
| gateflow | prayog-skills (`v0.5.0-rc.2`) | Contract navigation + dispatch eligibility |
| gateflow | GitHub App | ForgeClient PRs, comments, board APIs |
| gateflow | PostgreSQL | RunStore + job queue |
| gateflow | launchpad | Harness sync on worker |
| gateflow-ops | gateflow | Upstream APIs — deferred |

## 8. Revision diff

Omit — revision 1 (no prior map).

## 9. Downstream ripple ledger

| Repo | In-flight artifact | Required action | Reason | Owner | Blocking |
|------|--------------------|-----------------|--------|-------|----------|
| gateflow | none (spec not yet opened) | **open** after map approval | First map / first engg handoff for 002 | prayog-pe-team | no |
| gateflow-ops | none | hold | Deferred | prayog-pe-team | no |
| prayog-skills | pin `v0.5.0-rc.2` delivered | continue | Monitor only | prayog-pe-team | no |
| launchpad | none | continue | Monitor only | prayog-pe-team | no |

## 10. Open questions

| ID | Lane | Question | Owner | Blocking | Required by | Default if deferred | Status |
|----|------|----------|-------|----------|-------------|---------------------|--------|
| IM-01 | PE | Exact HTTP paths and schemas for FR-15 / FR-20 / FR-21 / FR-24 (PRD OQ #1) | PE | no | gateflow spec PR | Spec owns paths; product FRs remain normative | open |
| IM-02 | PE | PR title/body templates and `pr.branch_prefix` (PRD OQ #2) | PE | no | W1 PR-at-start | Programme convention in gateflow config | open |
| IM-03 | PE | Board API list filters (PRD OQ #3) | PE | no | W2 board APIs | Narrow filters in spec; dumb primitives only | open |
| IM-04 | PE | Assumption A3 App/Projects permission matrix (PRD OQ #5) | PE | no | W2 board create/list | Confirm in spec before W2 exit | open |
| IM-05 | PE | PE alert channel on PR open/update failure (PRD OQ #6) | PE | no | W1 error path | status API + `notify_pending` until decided | open |

## 11. PR readiness handoff

| Item | Value |
|------|-------|
| Verdict | **PR READY** |
| Collision detection | no-collision |
| Collision evidence | No `Impact-Map-INIT-GATEFLOW-002`; no open meta PR for 002; local branch `chore/INIT-GATEFLOW-002-outline` is this initiative’s WIP; INIT-GATEFLOW-001 PRs #4/#9 merged / #7 closed — different initiative |
| Human resolution | none |
| Resolution completed | n/a |
| Existing PR | none |
| Proposed branch | `chore/INIT-GATEFLOW-002-prd` |
| Proposed base | `develop` |
| Proposed title | `[INIT-GATEFLOW-002] PRD — API-triggered waves & platform readiness` |
| Files to commit | `prd/INIT-GATEFLOW-002.md`, `prd/INIT-GATEFLOW-002-outline.md`, `prd/reports/Impact-Map-INIT-GATEFLOW-002.md` (intermediate Validation/Resolution/Update-Summary reports **deleted**) |
| Reviewer | @drivestream-lab/prayog-pe-team |
| Initial Gate 1 label | `impact-map-pending` |
| Additional invalidation label | none |
| Blocking items | none |

**No GitHub side effects have occurred.** Ask the user whether to create or update the Draft PR. Continue only after explicit authorization.

### Proposed Draft PR body

```markdown
## Product change

INIT-GATEFLOW-002 makes Gateflow **API-first**: authenticated wave-start API (label trigger removed for 002), per-skill runner/model, PR thread from **run start**, richer run/metrics APIs, and wider board operation APIs (dumb forge primitives) with ForgeClient-only production path. Primary delivery is **gateflow only**; gateflow-ops UI deferred. Skills pin remains **`v0.5.0-rc.2`**.

## Impact-map summary

- Revision: **1**
- PRD digest: `sha256:2f339bae00df71e21b45e51c7551f1fb06490dd1805b96e08daca741c140332c`
- Scope digest (gateflow): `sha256:3662f15e366994defaf08fa43b0a5a568eb152f6a74bcff73a0b0453dba0c901`
- Affected repos: drivestream-lab/gateflow
- Deferred repos: drivestream-lab/gateflow-ops
- Monitor: prayog-skills, launchpad
- Blocking questions: none (IM-01–05 → gateflow spec)
- Artifact: `prd/reports/Impact-Map-INIT-GATEFLOW-002.md`

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
initiative: INIT-GATEFLOW-002
map_revision: 1
meta_pr_head_sha: {SHA after this artifact is committed}
prd_digest: sha256:2f339bae00df71e21b45e51c7551f1fb06490dd1805b96e08daca741c140332c
artifact: prd/reports/Impact-Map-INIT-GATEFLOW-002.md
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
    path: prd/reports/Impact-Map-INIT-GATEFLOW-002.md
    digest: sha256:20a32beb198a931eac4a6d5198ffd701e22db716e281427abc63a2dbebaa4480
  blockers: []
  signals:
    pr_ready: true
    map_revision: 1
    prd_digest: sha256:2f339bae00df71e21b45e51c7551f1fb06490dd1805b96e08daca741c140332c
    scope_digest_gateflow: sha256:3662f15e366994defaf08fa43b0a5a568eb152f6a74bcff73a0b0453dba0c901
    collision_detection: no-collision
    intermediate_reports_deleted: true
    files_to_commit: prd+outline+impact-map-only
  next_candidates:
    - prd-pr-action
  human_checkpoint: true
  external_action: true
```
