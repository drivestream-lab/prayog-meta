---
schema_version: 1
initiative: INIT-GATEFLOW-001
map_revision: 2
source_prd: prd/INIT-GATEFLOW-001.md
source_prd_digest: sha256:253fe3ea30134659b50d5fd38693a11e444c8f4642c4fef0c863b27eccf811c2
previous_revision: 1
previous_artifact_commit: 41914b298949693413ff22458d7996ff19e143d5
change_reason: PRD polish — PE product decisions (Decisions 1–12), validation r5 clean pass; scope widened for API+worker, auth, handoff ref fallback
material_change: true
generated_at: 2026-07-23T05:17:00Z
---

# Impact map — INIT-GATEFLOW-001 — revision 2

> This file is generated locally before PR creation and becomes the scope source
> of truth when committed. Effective approval is derived from a tech-lead
> GitHub APPROVED review on the exact current meta PR head SHA; dynamic PR state
> is not stored in this artifact's frontmatter.

## 1. Source and revision

| Field | Value |
|-------|-------|
| PRD | `prd/INIT-GATEFLOW-001.md` |
| PRD digest | `sha256:253fe3ea30134659b50d5fd38693a11e444c8f4642c4fef0c863b27eccf811c2` |
| Map revision | `2` |
| Previous revision | `1` |
| Previous artifact commit | `41914b298949693413ff22458d7996ff19e143d5` |
| Change reason | PE product-review decisions + validation r5 pass; normative detail for deployment, auth, handoff paths |
| Material change | yes — gateflow scope_digest widened (same repo; acceptance-meaning changes) |

## 2. Affected repositories

| Service | Repo | Team | Scope summary | Scope digest | Spec to create | Confidence |
|---------|------|------|---------------|--------------|----------------|------------|
| gateflow | drivestream-lab/gateflow | @drivestream-lab/prayog-pe-team | W1 control plane (**gateflow repo only**): **API + async worker** with Postgres job queue (Decision #6); GitHub App webhooks (FR-1); programme-wide trigger `gateflow:run-wave` (Decision #3); wave-run preconditions incl. concurrent reject (Decision #2); PostgreSQL RunStore (FR-3); **HandoffReader** — PR head ref + `default_branch` fallback for issue triggers (Decision #5, FR-4); WorkflowEngine + PolicyEngine with `dispatch: orchestrated` after rc-2; pre-rc-2 block + comment (Decision #9); Cursor AgentRunner (FR-6); retry budget exhaustion → stop + comment (FR-7); contract stops incl. terminal (FR-8); **ForgeClient PR/issue comments only** H1 (Decision #1, FR-9); metrics v0 — **90-day retention**, `GET /metrics/runs` JSON (Decision #7, FR-10); ToolProvider `none` (FR-11); **status JSON API** with programme service token auth (Decision #4, FR-12); **single default model profile** H1 (Decision #8, FR-13); ForgeClient outbound (FR-14). Programme config in gateflow repo. Dogfood: gateflow repo. | `sha256:0c434aacb71615d7db8114029509919e3a0b72cf27c559c1716187fd9c194433` | `INIT-GATEFLOW-001-gateflow.md` | High |

## 3. Deferred repositories

| Service | Repo | Reason | Revisit condition |
|---------|------|--------|-------------------|
| gateflow-ops | drivestream-lab/gateflow-ops | PRD §4/§5: BFF/UI **out of W1 scope**; W1 uses gateflow native status API (FR-12) | W2 / H2 after control plane proven |

## 4. Transitively affected repositories

| Service | Repo | Depends on | Potential impact | Disposition |
|---------|------|------------|------------------|-------------|
| gateflow-ops | drivestream-lab/gateflow-ops | gateflow (upstream) | Future BFF consumes status JSON API | monitor — deferred W2+ |
| prayog-skills | drivestream-lab/prayog-skills | — (SSOT) | rc-2 `dispatch` pin required for W1 PolicyEngine dispatch | monitor — Joint Gate 1 via INIT-PRAYOG-SKILLS-002 |
| launchpad | drivestream-lab/launchpad | — | Worker harness sync before AgentRunner | monitor — no launchpad delivery in W1 |

## 5. Not affected

| Service | Repo | Reason |
|---------|------|--------|
| prayog-meta | drivestream-lab/prayog-meta | Hosts PRD, impact map, validation reports; W1 programme config not in harness/meta YAML |
| prayog-skills | drivestream-lab/prayog-skills | Contract changes via paired INIT-PRAYOG-SKILLS-002; gateflow read-only consumer |
| launchpad | drivestream-lab/launchpad | Integration consumer only; no new launchpad features for W1 exit |

## 6. Cross-repository contracts

| Contract ID | Provider repo | Consumer repo | Capability | Owner | Status |
|-------------|---------------|---------------|------------|-------|--------|
| CTR-01 | prayog-skills | gateflow | Pinned workflow + delivery contract + handoff spec; `dispatch` eligibility (rc-2) | prayog-pe-team | new — pending Joint Gate 1 |
| CTR-02 | GitHub (forge) | gateflow | Inbound webhooks: PR, issue, label (FR-1) | prayog-pe-team | new |
| CTR-03 | gateflow | GitHub (forge) | ForgeClient: PR/issue comments, run-status labels; no gate-label writes (FR-9, FR-14) | prayog-pe-team | new |
| CTR-04 | launchpad | gateflow | Worker harness sync pre-dispatch | prayog-pe-team | unchanged |
| CTR-05 | gateflow | gateflow-ops | Status JSON API for BFF | prayog-pe-team | deferred W2+ |

## 7. Dependency and build order

```text
Joint Gate 1 (INIT-GATEFLOW-001 + INIT-PRAYOG-SKILLS-002)
  → gateflow W0 skeleton (pre–rc-2: API, worker stub, webhooks, RunStore, HandoffReader, resolver without dispatch)
  → prayog-skills rc-2 pin (`dispatch` field)
  → gateflow W1 (PolicyEngine + AgentRunner + async job processing + metrics + status API)
  → W1 exit on gateflow repo
  → Phase B dogfood
  → gateflow-ops (W2+, deferred)
```

| Repo | Depends on | Reason |
|------|------------|--------|
| gateflow | prayog-skills (pin) | Contract navigation + dispatch (FR-5); rc-2 required |
| gateflow | GitHub App | Webhook ingress + ForgeClient (FR-1, FR-14) |
| gateflow | PostgreSQL | RunStore + job queue (FR-3, Decision #6) |
| gateflow | launchpad | Harness sync on worker |
| gateflow-ops | gateflow | Upstream API — deferred W2+ |

## 8. Revision diff

| Repo | Prior status | Current status | Scope digest changed? | Change |
|------|--------------|----------------|-----------------------|--------|
| gateflow | affected | affected | yes | **widened** — API+async worker, programme token auth, handoff ref fallback, 90d metrics, comments-only H1, `gateflow:run-wave` |
| gateflow-ops | deferred | deferred | no | unchanged |
| prayog-skills | monitor | monitor | no | unchanged |
| launchpad | monitor | monitor | no | unchanged |
| prayog-meta | not affected | not affected | no | unchanged |

**Prior PRD digest (rev 1):** `sha256:2349b8819a7bd44de460eaf00b017f68056fa97942663895765d4fdbfb78493c`  
**Current PRD digest:** `sha256:253fe3ea30134659b50d5fd38693a11e444c8f4642c4fef0c863b27eccf811c2`

## 9. Downstream ripple ledger

| Repo | In-flight artifact | Required action | Reason | Owner | Blocking |
|------|--------------------|-----------------|--------|-------|----------|
| gateflow | none (spec not yet opened) | **re-draft** | Scope digest widened rev 1→2; PE codebase map exists against prior PRD | prayog-pe-team | no — no merged spec yet |
| gateflow-ops | none | hold | Still deferred W2+ | prayog-pe-team | no |

## 10. Open questions

| ID | Lane | Question | Owner | Blocking | Required by | Default if deferred | Status |
|----|------|----------|-------|----------|-------------|---------------------|--------|
| IM-01 | PM / PE | **Joint Gate 1** with INIT-PRAYOG-SKILLS-002 before rc-2 pin and W1 dispatch | PM + PE | yes | W1 #3–11 / Phase B | W0 skeleton without dispatch | open |
| IM-03 | PE | rc-2 pin timing vs Gateflow W0 merge (PRD OQ #2) | PE | no | W1 dispatch loop | W0 without dispatch; W1 blocked until rc-2 | open |
| IM-02 | PE | Pilot trigger label naming | PE | no | — | — | **resolved** — `gateflow:run-wave` (Decision #3) |
| IM-04 | PE | Concurrent trigger policy | PE | no | — | — | **resolved** — reject W1 (Decision #2) |

## 11. PR readiness handoff

| Item | Value |
|------|-------|
| Verdict | **PR READY** |
| Collision detection | no-collision |
| Collision evidence | Existing PR #4 (`chore/INIT-GATEFLOW-001-prd`) is this initiative; rev 2 updates same PR artifact set |
| Human resolution | none |
| Resolution completed | n/a |
| Existing PR | https://github.com/drivestream-lab/prayog-meta/pull/4 |
| Proposed branch | `chore/INIT-GATEFLOW-001-prd` (existing) |
| Proposed base | `develop` |
| Proposed title | `[INIT-GATEFLOW-001] PRD — Gateflow delivery orchestrator (W1 gateflow only)` |
| Files to commit | `prd/INIT-GATEFLOW-001.md`, `prd/reports/Impact-Map-INIT-GATEFLOW-001.md` (rev 2), validation r4–r5, resolution r4, reports |
| Reviewer | @drivestream-lab/prayog-pe-team |
| Initial Gate 1 label | `impact-map-pending` (retain) |
| Additional invalidation label | **`impact-map-revised`** — map revision 2 on open PR; gate closed until re-approval |
| Blocking items | IM-01 (Joint Gate 1) — blocks rc-2 W1 dispatch / Phase B only |

**No GitHub side effects have occurred.** Ask the user whether to commit and push to update PR #4.

### Proposed Draft PR body (update)

```markdown
## Product change

Gateflow delivery control plane — W1 scoped to **gateflow repo only**. Orchestrates wave skills when `dispatch: orchestrated`, PE authorizes via `gateflow:run-wave`, stops on contract nodes, records runs in PostgreSQL. Rev 2 PRD incorporates PE product decisions: API+async worker, programme service token auth, handoff PR-head + default-branch fallback, 90-day metrics, comments-only H1 progress.

## Impact-map summary

- Revision: **2** (was 1)
- PRD digest: `sha256:253fe3ea30134659b50d5fd38693a11e444c8f4642c4fef0c863b27eccf811c2`
- Scope digest (gateflow): `sha256:0c434aacb71615d7db8114029509919e3a0b72cf27c559c1716187fd9c194433`
- Affected repos: drivestream-lab/gateflow
- Deferred: drivestream-lab/gateflow-ops (W2+)
- Validation: r5 clean pass (0 findings)
- Artifact: `prd/reports/Impact-Map-INIT-GATEFLOW-001.md`

## Gate 1 — engineering handoff readiness

- [x] Product/domain decisions committed (Decisions 1–12)
- [x] Validation r5 clean pass
- [ ] Impact-map rev 2 approved on current PR head
- [ ] PE/tech lead review on exact head SHA

Requested reviewer: @drivestream-lab/prayog-pe-team
Label: `impact-map-revised` → `impact-map-pending` until re-approved
```

## 12. Approval request (after PR update)

Tech lead must **re-approve** on the exact PR head SHA after rev 2 commit:

```text
Impact map approved
initiative: INIT-GATEFLOW-001
map_revision: 2
meta_pr_head_sha: {SHA after rev 2 commit}
prd_digest: sha256:253fe3ea30134659b50d5fd38693a11e444c8f4642c4fef0c863b27eccf811c2
artifact: prd/reports/Impact-Map-INIT-GATEFLOW-001.md
```

---

```yaml
handoff:
  contract: sdd-delivery/v2
  stage: prd-impact-map
  outcome: pass
  artifact:
    path: prd/reports/Impact-Map-INIT-GATEFLOW-001.md
    digest: sha256:a74aaeadafe06f1e59990d1796f10747b445367f4973bda398571e87863aa3e8
  blockers: []
  signals:
    pr_ready: true
    map_revision: 2
    material_change: true
    prd_digest: sha256:253fe3ea30134659b50d5fd38693a11e444c8f4642c4fef0c863b27eccf811c2
    scope_digest_gateflow: sha256:0c434aacb71615d7db8114029509919e3a0b72cf27c559c1716187fd9c194433
    prior_map_revision: 1
    affected_repos:
      - drivestream-lab/gateflow
    deferred_repos:
      - drivestream-lab/gateflow-ops
    existing_pr: https://github.com/drivestream-lab/prayog-meta/pull/4
    validation: Validation-Report-INIT-GATEFLOW-001-r5.md (pass)
    invalidation_label: impact-map-revised
  next_candidates:
    - prd-pr-action
  human_checkpoint: true
  external_action: true
```
