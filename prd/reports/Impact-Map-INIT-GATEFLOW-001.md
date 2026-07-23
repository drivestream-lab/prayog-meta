---
schema_version: 1
initiative: INIT-GATEFLOW-001
map_revision: 3
source_prd: prd/INIT-GATEFLOW-001.md
source_prd_digest: sha256:9fa343f11f9497cd278c18ba4b87391b15cab566f285e88a7f4cda9bf700802d
previous_revision: 2
previous_artifact_commit: 4428c523aa0fbd065895a97841528c61d1af1ace
change_reason: PRD pin to v0.5.0-rc.2 (INIT-PRAYOG-SKILLS-002 delivered); Gateflow PolicyEngine unblocked; intermediate validation/resolution reports removed — final PRD + this map only
material_change: true
generated_at: 2026-07-23T10:50:00Z
---

# Impact map — INIT-GATEFLOW-001 — revision 3

> This file is generated locally before PR creation and becomes the scope source
> of truth when committed. Effective approval is derived from a tech-lead
> GitHub APPROVED review on the exact current meta PR head SHA; dynamic PR state
> is not stored in this artifact's frontmatter.

## 1. Source and revision

| Field | Value |
|-------|-------|
| PRD | `prd/INIT-GATEFLOW-001.md` |
| PRD digest | `sha256:9fa343f11f9497cd278c18ba4b87391b15cab566f285e88a7f4cda9bf700802d` |
| Map revision | `3` |
| Previous revision | `2` |
| Previous artifact commit | `4428c523aa0fbd065895a97841528c61d1af1ace` (PR #4 merge on develop; rev 2 map tip) |
| Change reason | Skills pin **`v0.5.0-rc.2`** delivered; Gateflow W1 PolicyEngine unblocked; report hygiene (final PRD + map only) |
| Material change | yes — pin availability / dependency gate cleared; acceptance path uses delivered `dispatch` pin |

## 2. Affected repositories

| Service | Repo | Team | Scope summary | Scope digest | Spec to create | Confidence |
|---------|------|------|---------------|--------------|----------------|------------|
| gateflow | drivestream-lab/gateflow | @drivestream-lab/prayog-pe-team | W1 control plane (gateflow repo only): API + async worker with Postgres job queue (Decision #6); GitHub App webhooks (FR-1); programme-wide trigger gateflow:run-wave (Decision #3); wave-run preconditions incl. concurrent reject (Decision #2); PostgreSQL RunStore (FR-3); HandoffReader PR head ref + default_branch fallback for issue triggers (Decision #5, FR-4); WorkflowEngine + PolicyEngine with dispatch:orchestrated on pin v0.5.0-rc.2 (INIT-PRAYOG-SKILLS-002 delivered); legacy pin/orchestration-unavailable block + comment (Decision #9); Cursor AgentRunner (FR-6); retry budget exhaustion stop + comment (FR-7); contract stops incl. terminal (FR-8); ForgeClient PR/issue comments only H1 (Decision #1, FR-9); metrics v0 90-day retention GET /metrics/runs JSON (Decision #7, FR-10); ToolProvider none (FR-11); status JSON API with programme service token auth (Decision #4, FR-12); single default model profile H1 (Decision #8, FR-13); ForgeClient outbound (FR-14); programme config in gateflow repo; dogfood gateflow repo only | `sha256:f81fd7c11c9b438524032898b31b028b376bcf766cb5ef675f0ecb81f326e9a0` | `INIT-GATEFLOW-001-gateflow.md` | High |

## 3. Deferred repositories

| Service | Repo | Reason | Revisit condition |
|---------|------|--------|-------------------|
| gateflow-ops | drivestream-lab/gateflow-ops | W1 uses gateflow native status API only; BFF/UI out of W1 exit (Decision #12, Non-Goals) | W2 / H2 after W1 exit proven |

## 4. Transitively affected repositories

| Service | Repo | Depends on | Potential impact | Disposition |
|---------|------|------------|------------------|-------------|
| gateflow-ops | drivestream-lab/gateflow-ops | gateflow (upstream) | Future BFF consumes status JSON API | monitor — deferred W2+ |
| prayog-skills | drivestream-lab/prayog-skills | — (SSOT) | Pin **`v0.5.0-rc.2`** already delivers `dispatch` for Gateflow | monitor — delivered; no W1 delivery in gateflow INIT |
| launchpad | drivestream-lab/launchpad | — | Worker harness sync before AgentRunner | monitor — no launchpad feature delivery in W1 |

## 5. Not affected

| Service | Repo | Reason |
|---------|------|--------|
| prayog-meta | drivestream-lab/prayog-meta | Hosts PRD + impact map; W1 programme config lives in gateflow repo (not harness YAML keys for triggers) |
| prayog-skills | drivestream-lab/prayog-skills | Contract already delivered via INIT-PRAYOG-SKILLS-002 / pin `v0.5.0-rc.2`; gateflow is read-only consumer |
| launchpad | drivestream-lab/launchpad | Integration consumer only; no new launchpad features for W1 exit |

## 6. Cross-repository contracts

| Contract ID | Provider repo | Consumer repo | Capability | Owner | Status |
|-------------|---------------|---------------|------------|-------|--------|
| CTR-01 | prayog-skills | gateflow | Pinned workflow + delivery contract + handoff spec; `dispatch` eligibility (**`v0.5.0-rc.2`**) | prayog-pe-team | available — delivered |
| CTR-02 | GitHub (forge) | gateflow | Inbound webhooks: PR, issue, label (FR-1) | prayog-pe-team | new |
| CTR-03 | gateflow | GitHub (forge) | ForgeClient: PR/issue comments, run-status labels; no gate-label writes (FR-9, FR-14) | prayog-pe-team | new |
| CTR-04 | launchpad | gateflow | Worker harness sync pre-dispatch | prayog-pe-team | unchanged |
| CTR-05 | gateflow | gateflow-ops | Status JSON API for BFF | prayog-pe-team | deferred W2+ |

## 7. Dependency and build order

```text
prayog-skills pin v0.5.0-rc.2 (delivered — INIT-PRAYOG-SKILLS-002)
  → gateflow W0 skeleton (API, worker stub, webhooks, RunStore, HandoffReader, resolver)
  → gateflow W1 (PolicyEngine + AgentRunner + async jobs + metrics + status API) — unblocked
  → W1 exit on gateflow repo
  → Phase B dogfood
  → gateflow-ops (W2+, deferred)
```

| Repo | Depends on | Reason |
|------|------------|--------|
| gateflow | prayog-skills (pin **`v0.5.0-rc.2`**) | Contract navigation + dispatch (FR-5); pin delivered |
| gateflow | GitHub App | Webhook ingress + ForgeClient (FR-1, FR-14) |
| gateflow | PostgreSQL | RunStore + job queue (FR-3, Decision #6) |
| gateflow | launchpad | Harness sync on worker |
| gateflow-ops | gateflow | Upstream API — deferred W2+ |

## 8. Revision diff

| Repo | Prior status (rev 2) | Current status | Scope digest changed? | Change |
|------|----------------------|----------------|-----------------------|--------|
| gateflow | affected | affected | yes | **widened** — pin **`v0.5.0-rc.2`** / PolicyEngine path unblocked (was pre-rc-2 gated) |
| gateflow-ops | deferred | deferred | no | unchanged |
| prayog-skills | monitor | monitor | no | status note: pin **delivered** |
| launchpad | monitor | monitor | no | unchanged |
| prayog-meta | not affected | not affected | no | unchanged |

**Prior PRD digest (rev 2):** `sha256:253fe3ea30134659b50d5fd38693a11e444c8f4642c4fef0c863b27eccf811c2`  
**Current PRD digest:** `sha256:9fa343f11f9497cd278c18ba4b87391b15cab566f285e88a7f4cda9bf700802d`

## 9. Downstream ripple ledger

| Repo | In-flight artifact | Required action | Reason | Owner | Blocking |
|------|--------------------|-----------------|--------|-------|----------|
| gateflow | none (spec not yet opened) | **re-draft** then open after map approval | Scope digest changed rev 2→3 (pin delivery / unblock) | prayog-pe-team | no — no merged spec yet |
| gateflow-ops | none | hold | Still deferred W2+ | prayog-pe-team | no |
| prayog-skills | pin `v0.5.0-rc.2` delivered | continue | Monitor only for this INIT | prayog-pe-team | no |
| launchpad | none | continue | Monitor only | prayog-pe-team | no |

## 10. Open questions

| ID | Lane | Question | Owner | Blocking | Required by | Default if deferred | Status |
|----|------|----------|-------|----------|-------------|---------------------|--------|
| IM-05 | PE | ForgeClient auth: App installation token only vs PAT allowed in dev? (PRD OQ #1) | PE | no | W0 ForgeClient | App token preferred; PAT dev-only per FR-14 | open |
| IM-01 | PM / PE | Skills pin before W1 dispatch | PM + PE | no | — | — | **resolved** — pin **`v0.5.0-rc.2`** delivered |
| IM-02 | PE | Pilot trigger label naming | PE | no | — | — | **resolved** — `gateflow:run-wave` (Decision #3) |
| IM-03 | PE | Pin timing vs Gateflow W0 | PE | no | — | — | **resolved** — pin delivered; Gateflow unblocked |
| IM-04 | PE | Concurrent trigger policy | PE | no | — | — | **resolved** — reject W1 (Decision #2) |

## 11. PR readiness handoff

| Item | Value |
|------|-------|
| Verdict | **PR READY** |
| Collision detection | no-collision |
| Collision evidence | PR #4 MERGED (`https://github.com/drivestream-lab/prayog-meta/pull/4`); no open meta PRs; leftover local/remote branches `chore/INIT-GATEFLOW-001-prd*` / `revert/…` are stale post-merge (not competing open PRs) |
| Human resolution | none |
| Resolution completed | n/a |
| Existing PR | none (open) |
| Proposed branch | `chore/INIT-GATEFLOW-001-prd` |
| Proposed base | `develop` |
| Proposed title | `[INIT-GATEFLOW-001] PRD — Gateflow delivery orchestrator (W1 gateflow only)` |
| Files to commit | `prd/INIT-GATEFLOW-001.md`, `prd/INIT-GATEFLOW-001-outline.md`, `planning/gateflow-programme-vision.md`, `.harness-pin.yaml`, `prd/reports/Impact-Map-INIT-GATEFLOW-001.md` (intermediate Validation/Resolution/Update-Summary reports **deleted**) |
| Reviewer | @drivestream-lab/prayog-pe-team |
| Initial Gate 1 label | `impact-map-pending` |
| Additional invalidation label | `impact-map-revised` (map rev 3 vs develop rev 2) |
| Blocking items | none |

**No GitHub side effects have occurred.** Ask the user whether to create or update the Draft PR. Continue only after explicit authorization.

### Proposed Draft PR body

```markdown
## Product change

Gateflow delivery control plane — W1 scoped to **gateflow repo only**. Orchestrates wave skills when `dispatch: orchestrated` on pin **`v0.5.0-rc.2`** (INIT-PRAYOG-SKILLS-002 **delivered**; PolicyEngine unblocked). PE authorizes via `gateflow:run-wave`, stops on contract nodes, records runs in PostgreSQL. Rev 3 impact map follows pin delivery and report hygiene (final PRD + map only).

## Impact-map summary

- Revision: **3** (was 2 on develop)
- PRD digest: `sha256:9fa343f11f9497cd278c18ba4b87391b15cab566f285e88a7f4cda9bf700802d`
- Scope digest (gateflow): `sha256:f81fd7c11c9b438524032898b31b028b376bcf766cb5ef675f0ecb81f326e9a0`
- Affected repos: drivestream-lab/gateflow
- Deferred repos: drivestream-lab/gateflow-ops (W2+)
- Monitor: prayog-skills (pin delivered), launchpad
- Blocking questions: none (IM-05 ForgeClient auth non-blocking)
- Artifact: `prd/reports/Impact-Map-INIT-GATEFLOW-001.md`

## Gate 1 — engineering handoff readiness

- [x] Product/domain decisions committed (Decisions 1–13; skills pin delivered)
- [ ] Impact-map rev 3 approved on current PR head
- [ ] PE/tech lead review on exact head SHA

Requested reviewer: @drivestream-lab/prayog-pe-team
Labels: `impact-map-revised` + `impact-map-pending` until re-approved
```

## 12. Approval request (after Draft PR creation)

Tech lead must review this artifact on the meta PR and submit GitHub
**Approve** on the exact PR head SHA using:

```text
Impact map approved
initiative: INIT-GATEFLOW-001
map_revision: 3
meta_pr_head_sha: {SHA after this artifact is committed}
prd_digest: sha256:9fa343f11f9497cd278c18ba4b87391b15cab566f285e88a7f4cda9bf700802d
artifact: prd/reports/Impact-Map-INIT-GATEFLOW-001.md
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
    path: prd/reports/Impact-Map-INIT-GATEFLOW-001.md
    digest: sha256:a0731f6b80516bc0ecb472d963fbfb902464c56a052e372fa04c0baa4ad455a2
  blockers: []
  signals:
    pr_ready: true
    map_revision: 3
    prd_digest: sha256:9fa343f11f9497cd278c18ba4b87391b15cab566f285e88a7f4cda9bf700802d
    scope_digest_gateflow: sha256:f81fd7c11c9b438524032898b31b028b376bcf766cb5ef675f0ecb81f326e9a0
    collision_detection: no-collision
    intermediate_reports_deleted: true
  next_candidates:
    - prd-pr-action
  human_checkpoint: true
  external_action: true
```
