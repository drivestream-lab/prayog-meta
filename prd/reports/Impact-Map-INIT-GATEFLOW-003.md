---
schema_version: 1
initiative: INIT-GATEFLOW-003
map_revision: 2
source_prd: prd/INIT-GATEFLOW-003.md
source_prd_digest: sha256:6062fa11d136e9a49dd6546377ec5d907789ef388108b477b846f6299673f9ad
previous_revision: 1
previous_artifact_commit: 64d008904c4d6e9ebf3d06d620591f8d942d4d8e
change_reason: Correct corrupted frontmatter source_prd_digest (must match PRD sha256); attestation fields aligned to map_revision 2
material_change: true
generated_at: 2026-07-24T12:53:58Z
---

# Impact map — INIT-GATEFLOW-003 — revision 2

> This file is generated locally before PR creation and becomes the scope source
> of truth when committed. Effective approval is derived from a tech-lead
> GitHub APPROVED review on the exact current meta PR head SHA; dynamic PR state
> is not stored in this artifact's frontmatter.

## 1. Source and revision

| Field | Value |
|-------|-------|
| PRD | `prd/INIT-GATEFLOW-003.md` |
| PRD digest | `sha256:6062fa11d136e9a49dd6546377ec5d907789ef388108b477b846f6299673f9ad` |
| Map revision | `2` |
| Previous revision | `1` |
| Previous artifact commit | `64d008904c4d6e9ebf3d06d620591f8d942d4d8e` |
| Change reason | Fix frontmatter `source_prd_digest` to match PRD; bump for PE attestation |
| Material change | yes — attestation digest field was wrong on rev 1 |

## 2. Affected repositories

| Service | Repo | Team | Scope summary | Scope digest | Spec to create | Confidence |
|---------|------|------|---------------|--------------|----------------|------------|
| gateflow | drivestream-lab/gateflow | @drivestream-lab/prayog-pe-team | Live Cursor AgentRunner in worker (FR-27); fail-fast unsupported/not-live runners (FR-28) and Cursor auth/start/crash (FR-29); stage+wave cycle-time with model_profile fields and p50/p95 for runner=cursor (FR-30); reuse 001/002 control plane (FR-31); Scenario B prove-it W1 and Scenario A prove-it W2 after pin support; no node allowlist; intended cursor/auto | `sha256:aaf398dc53a4606b34e9e24fa7513cd3a3b7e64ea677047faf5d2e90c64eba85` | `INIT-GATEFLOW-003-gateflow.md` | High |
| prayog-skills | drivestream-lab/prayog-skills | @drivestream-lab/prayog-pe-team | Supporting delivery: mark Scenario A skill set (spec-draft, initiative-feasibility, spec-technical-review, spec-implementation-plan) `dispatch:orchestrated` in pin for 003 exit; no version-bump exit gate (FR-27, A6) | `sha256:16d076c640e84d8a29885c55cb5398d3ecd8d1e6860f9d0166a7c133d4fae3bb` | `INIT-GATEFLOW-003-prayog-skills.md` (pin/content change; may be thin vs gateflow spec) | High |

## 3. Deferred repositories

| Service | Repo | Reason | Revisit condition |
|---------|------|--------|-------------------|
| gateflow-ops | drivestream-lab/gateflow-ops | UI/BFF explicitly out of scope; cycle-time exposed via existing run/metrics APIs (A4, Non-Goals) | Later INIT when ops UI consumes cycle-time fields |

## 4. Transitively affected repositories

| Service | Repo | Depends on | Potential impact | Disposition |
|---------|------|------------|------------------|-------------|
| gateflow-ops | drivestream-lab/gateflow-ops | gateflow (catalog `links.upstream`) | May later display stage/wave cycle-time fields from metrics/run APIs | monitor — deferred |
| launchpad | drivestream-lab/launchpad | — | Existing worker harness sync before AgentRunner; does not choose Cursor | monitor — no Launchpad product work |

## 5. Not affected

| Service | Repo | Reason |
|---------|------|--------|
| prayog-meta | drivestream-lab/prayog-meta | Hosts PRD + outline + this impact map; programme runner/model config stays in **gateflow** repo |
| launchpad | drivestream-lab/launchpad | Sync consumer only; no new Launchpad features for 003 exit (Decision #7) |

## 6. Cross-repository contracts

| Contract ID | Provider repo | Consumer repo | Capability | Owner | Status |
|-------------|---------------|---------------|------------|-------|--------|
| CTR-01 | prayog-skills | gateflow | Pinned workflow + `dispatch`; Scenario A nodes become `orchestrated` for 003 exit (FR-27, A6) | prayog-pe-team | changed — pin content (no version-bump exit gate) |
| CTR-02 | Cursor SDK (external) | gateflow | Live AgentRunner in worker; fail-fast auth/start/crash (FR-27, FR-29) | prayog-pe-team | new — live path |
| CTR-03 | gateflow | clients / tools / future gateflow-ops | Cycle-time stage+wave fields + p50/p95 for `runner=cursor` (FR-30; extends FR-20/21) | prayog-pe-team | changed — widened metrics |
| CTR-04 | launchpad | gateflow | Worker harness sync pre-dispatch | prayog-pe-team | unchanged |
| CTR-05 | gateflow | GitHub (forge) | ForgeClient + Notifier path unchanged from 002 | prayog-pe-team | unchanged |

## 7. Dependency and build order

```text
INIT-GATEFLOW-001 / 002 control plane (product: finished/delivered)
  → gateflow W0 (live Cursor AgentRunner skeleton; config resolve; fail-fast unsupported runners + credential presence)
  → gateflow W1 (Scenario B: pre-implement → loop-spec → verify → ground-spec; fail-fast crash/auth; stage + wave cycle-time)
  → prayog-skills supporting pin edit (Scenario A skills → dispatch: orchestrated) — before / with W2 prove-it
  → gateflow W2 (Scenario A skill set prove-it; post–Gate 2 Scenario A still runnable; metrics p50/p95)
  → gateflow-ops (deferred)
```

| Repo | Depends on | Reason |
|------|------------|--------|
| gateflow | prayog-skills (pin + Scenario A orchestrated for W2) | Orchestrated ⇒ triggered; Scenario A exit needs pin content |
| gateflow | launchpad | Existing harness sync on worker |
| gateflow | Cursor SDK + programme secrets | Live AgentRunner path |
| gateflow | PostgreSQL | RunStore + cycle-time fields |
| prayog-skills | none (SSOT) | Supporting pin `dispatch` content edit |
| gateflow-ops | gateflow | Upstream APIs — deferred |

## 8. Revision diff

| Repo | Prior status | Current status | Scope digest changed? | Change |
|------|--------------|----------------|-----------------------|--------|
| gateflow | affected | affected | no | unchanged |
| prayog-skills | affected | affected | no | unchanged |
| gateflow-ops | deferred | deferred | n/a | unchanged |
| launchpad | monitor / not affected | monitor / not affected | n/a | unchanged |
| prayog-meta | not affected | not affected | n/a | unchanged |

## 9. Downstream ripple ledger

| Repo | In-flight artifact | Required action | Reason | Owner | Blocking |
|------|--------------------|-----------------|--------|-------|----------|
| gateflow | none (spec not yet opened) | **open** after map approval | First eng handoff for 003 | prayog-pe-team | no |
| prayog-skills | pin `v0.5.0-rc.2` live; Scenario A still `manual` today | **open** (supporting pin PR / content change) after map approval | Required for Scenario A exit (W2) | prayog-pe-team | no |
| gateflow-ops | none | hold | Deferred | prayog-pe-team | no |
| launchpad | none | continue | Monitor only | prayog-pe-team | no |

## 10. Open questions

| ID | Lane | Question | Owner | Blocking | Required by | Default if deferred | Status |
|----|------|----------|-------|----------|-------------|---------------------|--------|
| IM-01 | PE | Exact Cursor credential / secret injection shape for the worker (PRD OQ #1) | PE | no | gateflow W0/W1 | Fail-fast if absent; shape in gateflow spec | open |
| IM-02 | PE | Exact RunStore field names for wave cycle time if not already present (PRD OQ #2) | PE | no | gateflow W1 | Normative definitions in PRD §4 remain; names in spec | open |
| IM-03 | PE | Stand-in path deleted vs unreachable when `runner=cursor` (PRD OQ #3) | PE | no | gateflow W1 exit | Product bar = live coding work + live Cursor agent | open |
| IM-04 | PE | Confirm gateflow runtime delivery of INIT-002 is complete for A1, given meta PR [#10](https://github.com/drivestream-lab/prayog-meta/pull/10) still open | PE | no | gateflow W0 start | Proceed per PRD A1 User-confirmed; reconcile meta PR separately | open |

## 11. PR readiness handoff

| Item | Value |
|------|-------|
| Verdict | **PR READY** |
| Collision detection | no-collision |
| Collision evidence | No `Impact-Map-INIT-GATEFLOW-003`; no PR for head `chore/INIT-GATEFLOW-003-prd`; open PR #10 is INIT-GATEFLOW-002 (unrelated); local branch is this initiative’s WIP |
| Human resolution | none |
| Resolution completed | n/a |
| Existing PR | none |
| Proposed branch | `chore/INIT-GATEFLOW-003-prd` |
| Proposed base | `develop` |
| Proposed title | `[INIT-GATEFLOW-003] PRD — Live Cursor AgentRunner` |
| Files to commit | `prd/INIT-GATEFLOW-003.md`, `prd/INIT-GATEFLOW-003-outline.md`, `prd/reports/Impact-Map-INIT-GATEFLOW-003.md` (Validation/Resolution reports optional hygiene — match 001/002 final PRD+outline+map pattern unless you want them retained) |
| Reviewer | @drivestream-lab/prayog-pe-team |
| Initial Gate 1 label | `impact-map-pending` |
| Additional invalidation label | none |
| Blocking items | none |

**No GitHub side effects have occurred.** Ask the user whether to create or update the Draft PR. Continue only after explicit authorization.

### Proposed Draft PR body

```markdown
## Product change

INIT-GATEFLOW-003 makes the **Cursor AgentRunner live** in Gateflow: orchestrated skills are triggered per pin (no allowlist), proven on Scenario A (pre–Gate 2 eng skill set) and Scenario B (coding cycle), with fail-fast honesty for auth/unsupported runners and defined stage/wave cycle-time metrics. **Primary delivery: gateflow.** **Supporting: prayog-skills** pin `dispatch: orchestrated` for Scenario A (no version-bump exit gate). Launchpad and gateflow-ops UI have no product work in this INIT.

## Impact-map summary

- Revision: **2**
- PRD digest: `sha256:6062fa11d136e9a49dd6546377ec5d907789ef388108b477b846f6299673f9ad`
- Scope digest (gateflow): `sha256:aaf398dc53a4606b34e9e24fa7513cd3a3b7e64ea677047faf5d2e90c64eba85`
- Scope digest (prayog-skills): `sha256:16d076c640e84d8a29885c55cb5398d3ecd8d1e6860f9d0166a7c133d4fae3bb`
- Affected repos: drivestream-lab/gateflow, drivestream-lab/prayog-skills
- Deferred repos: drivestream-lab/gateflow-ops
- Monitor: launchpad
- Blocking questions: none (IM-01–04 → PE / gateflow spec)
- Artifact: `prd/reports/Impact-Map-INIT-GATEFLOW-003.md`

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
initiative: INIT-GATEFLOW-003
map_revision: 2
meta_pr_head_sha: {SHA after this artifact is committed}
prd_digest: sha256:6062fa11d136e9a49dd6546377ec5d907789ef388108b477b846f6299673f9ad
artifact: prd/reports/Impact-Map-INIT-GATEFLOW-003.md
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
    path: prd/reports/Impact-Map-INIT-GATEFLOW-003.md
    digest: sha256:71ad4b588a2dd0a7b0d9ef3b3d75d6789a01b5aab07344ed9be7637b02c7519d
  blockers: []
  signals:
    pr_ready: true
    map_revision: 2
    prd_digest: sha256:6062fa11d136e9a49dd6546377ec5d907789ef388108b477b846f6299673f9ad
    scope_digest_gateflow: sha256:aaf398dc53a4606b34e9e24fa7513cd3a3b7e64ea677047faf5d2e90c64eba85
    scope_digest_prayog_skills: sha256:16d076c640e84d8a29885c55cb5398d3ecd8d1e6860f9d0166a7c133d4fae3bb
    collision_detection: no-collision
    affected_repos:
      - drivestream-lab/gateflow
      - drivestream-lab/prayog-skills
    deferred_repos:
      - drivestream-lab/gateflow-ops
  next_candidates:
    - prd-pr-action
  human_checkpoint: true
  external_action: true
```
