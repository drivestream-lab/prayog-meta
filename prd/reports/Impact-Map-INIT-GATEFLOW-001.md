---
schema_version: 1
initiative: INIT-GATEFLOW-001
map_revision: 1
source_prd: prd/INIT-GATEFLOW-001.md
source_prd_digest: sha256:2349b8819a7bd44de460eaf00b017f68056fa97942663895765d4fdbfb78493c
previous_revision: null
previous_artifact_commit: null
change_reason: Initial impact map — W1 scoped to gateflow repo only per PRD §4/§5
material_change: true
generated_at: 2026-07-22T17:42:00Z
---

# Impact map — INIT-GATEFLOW-001 — revision 1

> This file is generated locally before PR creation and becomes the scope source
> of truth when committed. Effective approval is derived from a tech-lead
> GitHub APPROVED review on the exact current meta PR head SHA; dynamic PR state
> is not stored in this artifact's frontmatter.

## 1. Source and revision

| Field | Value |
|-------|-------|
| PRD | `prd/INIT-GATEFLOW-001.md` |
| PRD digest | `sha256:2349b8819a7bd44de460eaf00b017f68056fa97942663895765d4fdbfb78493c` |
| Map revision | `1` |
| Previous revision | none |
| Previous artifact commit | none |
| Change reason | Initial impact map — Gateflow delivery orchestrator W1 |
| Material change | yes — first canonical scope artifact for this initiative |

## 2. Affected repositories

| Service | Repo | Team | Scope summary | Scope digest | Spec to create | Confidence |
|---------|------|------|---------------|--------------|----------------|------------|
| gateflow | drivestream-lab/gateflow | @drivestream-lab/prayog-pe-team | W1 control plane in **gateflow repo only**: GitHub App webhooks (FR-1); label trigger + wave-run preconditions (FR-2); PostgreSQL RunStore (FR-3); HandoffReader (FR-4); WorkflowEngine + PolicyEngine resolver with `dispatch: orchestrated` after rc-2 pin (FR-5); Cursor AgentRunner adapter (FR-6); findings retry budget with exhaustion → stop + comment (FR-7); contract stop nodes including terminal (FR-8); Notifier/ForgeClient run events (FR-9); metrics v0 + export surface (FR-10); ToolProvider slots wired `none` (FR-11); native status JSON API (FR-12); per-node runner/model profiles from gateflow programme config (FR-13); ForgeClient GitHub outbound (FR-14). Pluggable slots: AgentRunner, ToolProvider, ForgeClient, Notifier. Programme config lives in gateflow repo (W1). Dogfood target: gateflow repo. | `sha256:a0d4b8c85574c3ed25d9c1f27ce269cc8745e1b4344820c42e04e9361f107b78` | `INIT-GATEFLOW-001-gateflow.md` | High |

## 3. Deferred repositories

| Service | Repo | Reason | Revisit condition |
|---------|------|--------|-------------------|
| gateflow-ops | drivestream-lab/gateflow-ops | PRD §4 Repositories and §5 W1 exit: BFF/UI **out of W1 scope**; W1 uses gateflow native status API (FR-12) | W2 / H2 after gateflow control plane proven operational |

## 4. Transitively affected repositories

| Service | Repo | Depends on | Potential impact | Disposition |
|---------|------|------------|------------------|-------------|
| gateflow-ops | drivestream-lab/gateflow-ops | gateflow (upstream) | Future BFF consumes gateflow status JSON API; no W1 deliverable | monitor — deferred W2+ |
| prayog-skills | drivestream-lab/prayog-skills | — (SSOT provider) | gateflow reads pinned `workflow.yaml`, `delivery-contract.yaml`, handoff spec; **W1 PolicyEngine `dispatch` requires rc-2 pin** via paired INIT-PRAYOG-SKILLS-002 | monitor — Joint Gate 1 + rc-2; not in this map's W1 delivery scope |
| launchpad | drivestream-lab/launchpad | — | Pre-dispatch `sync-harness` on worker workspace (FR integration); harness verify on gateflow repo | monitor — integration consumer only; no launchpad code delivery in W1 |

## 5. Not affected

| Service | Repo | Reason |
|---------|------|--------|
| prayog-meta | drivestream-lab/prayog-meta | Hosts PRD, impact map, and validation reports; no runtime delivery code. W1 programme config explicitly **not** in harness/meta YAML (PRD §4 Programme Config). |
| prayog-skills | drivestream-lab/prayog-skills | Contract SSOT changes route through **INIT-PRAYOG-SKILLS-002** (paired Joint Gate 1); gateflow is read-only consumer in W1. |
| launchpad | drivestream-lab/launchpad | Invoked by gateflow worker; no new launchpad features required for W1 exit criteria. |

## 6. Cross-repository contracts

| Contract ID | Provider repo | Consumer repo | Capability | Owner | Status |
|-------------|---------------|---------------|------------|-------|--------|
| CTR-01 | prayog-skills | gateflow | Pinned `workflow.yaml`, `delivery-contract.yaml`, handoff-envelope spec; navigation + `dispatch` eligibility (rc-2) | prayog-pe-team | new — rc-2 pin pending Joint Gate 1 |
| CTR-02 | GitHub (forge) | gateflow | Inbound App webhooks: PR, issue, label events (FR-1) | prayog-pe-team | new |
| CTR-03 | gateflow | GitHub (forge) | ForgeClient outbound: comments, PR updates, run-status labels; forbids gate approval labels (FR-9, FR-14) | prayog-pe-team | new |
| CTR-04 | launchpad | gateflow | Worker workspace harness sync before AgentRunner dispatch | prayog-pe-team | unchanged integration |
| CTR-05 | gateflow | gateflow-ops | Status JSON API (`GET /runs/{id}` or equivalent) for BFF | prayog-pe-team | deferred W2+ |

## 7. Dependency and build order

```text
Joint Gate 1 (INIT-GATEFLOW-001 + INIT-PRAYOG-SKILLS-002 meta PRDs)
  → gateflow W0 skeleton (pre–rc-2: webhooks, RunStore, HandoffReader, resolver without dispatch)
  → prayog-skills rc-2 pin (`dispatch` field)
  → gateflow W1 (PolicyEngine + AgentRunner + TriggerRouter + metrics + status API)
  → W1 exit on gateflow repo
  → Phase B dogfood (gateflow repo)
  → gateflow-ops (W2+, deferred)
```

| Repo | Depends on | Reason |
|------|------------|--------|
| gateflow | prayog-skills (pin) | WorkflowEngine/PolicyEngine loads pinned contract (FR-5); dispatch requires rc-2 |
| gateflow | GitHub App | Webhook ingress and ForgeClient egress (FR-1, FR-14) |
| gateflow | PostgreSQL | RunStore SSOT all environments (FR-3, A4) |
| gateflow | launchpad | Harness sync on worker before agent dispatch |
| gateflow-ops | gateflow | Upstream status API — deferred W2+ |

## 8. Revision diff

*Initial map — no prior revision.*

## 9. Downstream ripple ledger

*Initial map — no in-flight app artifacts for this initiative.*

| Repo | In-flight artifact | Required action | Reason | Owner | Blocking |
|------|--------------------|-----------------|--------|-------|----------|
| gateflow | none | open | Initial scope — spec PR after impact-map approval | prayog-pe-team | no |
| gateflow-ops | none | hold | Deferred W2+ per PRD | prayog-pe-team | no |

## 10. Open questions

| ID | Lane | Question | Owner | Blocking | Required by | Default if deferred | Status |
|----|------|----------|-------|----------|-------------|---------------------|--------|
| IM-01 | PM / PE | **Joint Gate 1** — single gate with INIT-PRAYOG-SKILLS-002 before rc-2 pin and W1 PolicyEngine reading `dispatch` | PM + PE | yes | W1 criteria 3–11 / Phase B | W0 skeleton only without dispatch | open |
| IM-02 | PE | Pilot trigger label: `gateflow:run-w0` vs programme-specific prefix? (PRD OQ #1) | PE | no | gateflow programme config | `gateflow:run-w0` per PRD example | open |
| IM-03 | PE | rc-2 pin timing vs Gateflow W0 merge (PRD OQ #6) | PE | no | W1 dispatch loop | W0 proceeds without dispatch; W1 blocked until rc-2 | open |
| IM-04 | PE | Concurrent label on active run: reject vs queue vs supersede (PRD OQ #7) | PE | no | FR-1 implementation | **Reject** (FR-1 default) | open |

## 11. PR readiness handoff

| Item | Value |
|------|-------|
| Verdict | **PR READY** |
| Collision detection | no-collision |
| Collision evidence | Local: no `Impact-Map-INIT-GATEFLOW-001.md` prior revision; no local branch matching `INIT-GATEFLOW-001`; `prd/INIT-GATEFLOW-001.md` present untracked on `develop`. Remote: `gh pr list` on prayog-meta returned no open Gateflow/INIT-GATEFLOW PRs (merged housekeeping PRs #2–#3 unrelated). |
| Human resolution | none |
| Resolution completed | n/a |
| Existing PR | none |
| Proposed branch | `chore/INIT-GATEFLOW-001-prd` |
| Proposed base | `develop` |
| Proposed title | `[INIT-GATEFLOW-001] PRD — Gateflow delivery orchestrator (W1 gateflow only)` |
| Files to commit | `prd/INIT-GATEFLOW-001.md`, `prd/INIT-GATEFLOW-001-outline.md`, `planning/gateflow-programme-vision.md`, `prd/reports/Impact-Map-INIT-GATEFLOW-001.md`, validation/resolution reports under `prd/reports/` |
| Reviewer | @drivestream-lab/prayog-pe-team |
| Initial Gate 1 label | `impact-map-pending` |
| Additional invalidation label | none |
| Blocking items | IM-01 (Joint Gate 1) blocks rc-2-dependent W1 exit and Phase B — **does not block** this meta PR or W0 skeleton spec work |

**No GitHub side effects have occurred.** Ask the user whether to create or update the Draft PR. Continue only after explicit authorization.

### Proposed Draft PR body

```markdown
## Product change

Gateflow is the programme delivery control plane: it listens to GitHub, reads handoff envelopes and the pinned prayog-skills contract, and dispatches coding agents only when the resolved workflow node is `type: skill` with `dispatch: orchestrated` and PE has authorized a run via programme-configured label. W1 delivers the operational control plane in **gateflow repo only** — webhooks, PostgreSQL RunStore, PolicyEngine, Cursor AgentRunner, ForgeClient/Notifier, metrics v0, and native status JSON API. gateflow-ops is deferred to W2+.

## Impact-map summary

- Revision: 1
- PRD digest: `sha256:2349b8819a7bd44de460eaf00b017f68056fa97942663895765d4fdbfb78493c`
- Affected repos: drivestream-lab/gateflow
- Deferred repos: drivestream-lab/gateflow-ops (W2+)
- Monitor: prayog-skills (rc-2 / Joint Gate 1), launchpad (harness sync)
- Blocking questions: IM-01 (Joint Gate 1 with INIT-PRAYOG-SKILLS-002) for W1 dispatch exit / Phase B
- Artifact: `prd/reports/Impact-Map-INIT-GATEFLOW-001.md`

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
initiative: INIT-GATEFLOW-001
map_revision: 1
meta_pr_head_sha: {SHA after this artifact is committed}
prd_digest: sha256:2349b8819a7bd44de460eaf00b017f68056fa97942663895765d4fdbfb78493c
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
    digest: sha256:795babe697aee43dc1f3cd3d4c7f6cb153a53221fa7e170cee8581a03f3c16b1
  blockers: []
  signals:
    pr_ready: true
    map_revision: 1
    prd_digest: sha256:2349b8819a7bd44de460eaf00b017f68056fa97942663895765d4fdbfb78493c
    affected_repos:
      - drivestream-lab/gateflow
    deferred_repos:
      - drivestream-lab/gateflow-ops
    collision_detection: no-collision
    validation_gate1_ready: true
    open_questions_blocking:
      - IM-01
  next_candidates:
    - prd-pr-action
  human_checkpoint: true
  external_action: true
```
