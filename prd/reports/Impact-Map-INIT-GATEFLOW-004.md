---
schema_version: 1
initiative: INIT-GATEFLOW-004
map_revision: 2
source_prd: prd/INIT-GATEFLOW-004.md
source_prd_digest: sha256:4be733db2a69739a2fded2165e68697ca7396525e161e0bd4c53c19242e722ee
previous_revision: 1
previous_artifact_commit: a5c59f8346846fb66a04a88621fb16baddf26916
change_reason: Move prayog-skills from affected → not affected; engg spec-lane pin/dispatch owned by INIT-GATEFLOW-003 W2
material_change: true
generated_at: 2026-07-27T10:25:00Z
---

# Impact map — INIT-GATEFLOW-004 — revision 2

> This file is generated locally before PR creation and becomes the scope source
> of truth when committed. Effective approval is derived from a tech-lead
> GitHub APPROVED review on the exact current meta PR head SHA; dynamic PR state
> is not stored in this artifact's frontmatter.

## 1. Source and revision

| Field | Value |
|-------|-------|
| PRD | `prd/INIT-GATEFLOW-004.md` |
| PRD digest | `sha256:4be733db2a69739a2fded2165e68697ca7396525e161e0bd4c53c19242e722ee` |
| Map revision | `2` |
| Previous revision | `1` |
| Previous artifact commit | `a5c59f8346846fb66a04a88621fb16baddf26916` |
| Change reason | `prayog-skills` out of 004 delivery — pin/`dispatch` for engg spec-lane stays on INIT-GATEFLOW-003 W2 |
| Material change | yes — affected set narrowed; REQ-40 no longer implies skills delivery |

## 2. Affected repositories

| Service | Repo | Team | Scope summary | Scope digest | Spec to create | Confidence |
|---------|------|------|---------------|--------------|----------------|------------|
| gateflow-ops | drivestream-lab/gateflow-ops | @drivestream-lab/prayog-pe-team | Mission Control v0: thin ops-user identity+sign-in (REQ-32); strict scorecard onboarding (REQ-33); fleet home (REQ-34); start/inspect waves last 30 days (REQ-35); run cockpit map+timeline+full log pane+GitHub jump (REQ-36); efficacy visibility+collect numbers (REQ-37); process display-only (REQ-38); BFF consumer of gateflow; **reads** pin for process map only | `sha256:081ffe18350b475e0d3e9d8b3b4129e08f01f1f3ff11f0ca56bd0ff11653c220` | `INIT-GATEFLOW-004-gateflow-ops.md` | High |
| gateflow | drivestream-lab/gateflow | @drivestream-lab/prayog-pe-team | Supporting only: extend run/metrics/wave APIs where Mission Control needs missing programme capabilities — fleet onboarding records, clearer wave summaries (REQ-39); not a second orchestrator | `sha256:a3fd7feeabf1333d010d38639e4c33657a365b7940862444f12da6b26a56f987` | `INIT-GATEFLOW-004-gateflow.md` (thin / API delta) | High |

## 3. Deferred repositories

| Service | Repo | Reason | Revisit condition |
|---------|------|--------|-------------------|
| — | — | None deferred for 004 product exit | — |

## 4. Transitively affected repositories

| Service | Repo | Depends on | Potential impact | Disposition |
|---------|------|------------|------------------|-------------|
| launchpad | drivestream-lab/launchpad | — | Greenfield factory unchanged; no Mission Control scaffolding | monitor — not affected (product) |
| gateflow-ops | drivestream-lab/gateflow-ops | gateflow (`links.upstream`) | Consumes run/metrics/wave APIs + optional REQ-39 extensions | affected (primary) |
| prayog-skills | drivestream-lab/prayog-skills | — | Ops may read pin for process map; pin/`dispatch` edits are 003 W2 | monitor — not affected (004 delivery) |

## 5. Not affected

| Service | Repo | Reason |
|---------|------|--------|
| prayog-skills | drivestream-lab/prayog-skills | Pin / `dispatch` for engg **spec-lane** stays on **INIT-GATEFLOW-003 W2**, not 004; 004 only **reads** the pin for process map (REQ-36, REQ-38) |
| prayog-meta | drivestream-lab/prayog-meta | Hosts PRD, outline, validation/resolution reports, this impact map; not an eng delivery target for Mission Control runtime |
| launchpad | drivestream-lab/launchpad | No greenfield / harness-install product work in 004 (Non-Goals; Launchpad ownership unchanged) |

## 6. Cross-repository contracts

| Contract ID | Provider repo | Consumer repo | Capability | Owner | Status |
|-------------|---------------|---------------|------------|-------|--------|
| CTR-01 | gateflow | gateflow-ops | Existing run / metrics / wave-start / board APIs for operate + cockpit (REQ-35–37, REQ-39 consume) | prayog-pe-team | unchanged — primary consume |
| CTR-02 | gateflow | gateflow-ops | Fleet onboarding records + clearer wave summaries if existing APIs insufficient (REQ-39) | prayog-pe-team | new — supporting delta |
| CTR-03 | prayog-skills | gateflow-ops | Pinned workflow projection for process map (REQ-36, REQ-38) — **read only**; no 004 pin edit | prayog-pe-team | unchanged — read pin |
| CTR-04 | gateflow | GitHub (ForgeClient) | Scorecard GitHub access + Open PR links (REQ-33, REQ-36); reused | prayog-pe-team | unchanged |

*Former CTR-04 (engg spec-lane pin `dispatch` change) removed from 004 — owned by INIT-GATEFLOW-003 W2.*

## 7. Dependency and build order

```text
INIT-GATEFLOW-001–002 finished; INIT-GATEFLOW-003 control plane + live Cursor available
  (003 W2 engg spec-lane pin + prove-it may still be open — may dogfood this PRD; not 004 delivery)

  → gateflow supporting API delta (REQ-39) — only where ops cannot meet cockpit/onboarding from existing APIs
  → gateflow-ops Mission Control W0–W3 (REQ-32–38) — primary product
```

| Repo | Depends on | Reason |
|------|------------|--------|
| gateflow-ops | gateflow | Catalog upstream; BFF consumes APIs (CTR-01/02) |
| gateflow-ops | prayog-skills (read) | Process map projection from pin (CTR-03) — no pin edit in 004 |
| gateflow | prayog-skills (runtime) | Orchestration honors existing pin; pin content changes are 003 |
| launchpad | — | Not in delivery chain |
| prayog-skills | — | Not in 004 delivery chain |

## 8. Revision diff

| Repo | Prior status (rev 1) | Current status (rev 2) | Scope digest changed? | Change |
|------|----------------------|------------------------|-----------------------|--------|
| gateflow-ops | affected | affected | yes | narrowed — pin read only; no skills delivery coupling |
| gateflow | affected | affected | yes | narrowed — no 004 ownership of engg spec-lane pin |
| prayog-skills | affected | **not affected** | n/a | **removed** — pin/`dispatch` stays on INIT-GATEFLOW-003 W2 |
| launchpad | monitor / not affected | monitor / not affected | n/a | unchanged |
| prayog-meta | not affected | not affected | n/a | unchanged |

## 9. Downstream ripple ledger

| Repo | In-flight artifact | Required action | Reason | Owner | Blocking |
|------|--------------------|-----------------|--------|-------|----------|
| gateflow-ops | none (spec not yet opened) | **open** after map approval | First Mission Control product delivery | prayog-pe-team | no |
| gateflow | none for 004 API delta | **open** after map approval (thin supporting) | REQ-39 only if gaps proven | prayog-pe-team | no |
| prayog-skills | pin live; engg spec-lane may still need `orchestrated` for 003 W2 | **hold** / **close** for 004 — do not open 004 skills spec | Removed from 004 affected set | prayog-pe-team | no |
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
| Collision detection | **no-collision** |
| Collision evidence | Prior Draft PR [#12](https://github.com/drivestream-lab/prayog-meta/pull/12) **closed** (same initiative; reopen or new Draft after this revision). No competing open 004 PR. |
| Human resolution | none |
| Resolution completed | n/a |
| Existing PR | [#12](https://github.com/drivestream-lab/prayog-meta/pull/12) closed — recreate Draft when authorized |
| Proposed branch | `chore/INIT-GATEFLOW-004-prd` |
| Proposed base | `develop` |
| Proposed title | `[INIT-GATEFLOW-004] PRD — Gateflow Mission Control` |
| Files to commit | `prd/INIT-GATEFLOW-004.md`, `prd/INIT-GATEFLOW-004-outline.md`, `prd/reports/Impact-Map-INIT-GATEFLOW-004.md` (+ prior vision / 001 / 002 status files if not yet on head) |
| Reviewer | @drivestream-lab/prayog-pe-team |
| Initial Gate 1 label | `impact-map-pending` (plus `impact-map-revised` if reopening after prior review) |
| Additional invalidation label | none until prior approval existed |
| Blocking items | none (IM-01–05 → PE/PM; non-blocking) |

**No GitHub side effects have occurred for this revision.** Ask the user whether to commit and recreate the Draft PR.

### Proposed Draft PR body

```markdown
## Product change

INIT-GATEFLOW-004 delivers **Gateflow Mission Control (v0)** in **gateflow-ops**: thin ops-user identity, strict scorecard onboarding of harnessed repos, fleet home, start/inspect waves from the UI, high-bar run cockpit (process map + timeline + full log pane + GitHub jump), and efficacy visibility while collecting numbers for later lift decisions. **Primary: gateflow-ops.** **Supporting: gateflow** (API gaps only, REQ-39). **prayog-skills** is **not affected** — engg spec-lane pin/`dispatch` stays on INIT-GATEFLOW-003 W2; 004 only reads the pin for process map. Launchpad greenfield stays out of scope. Checkpoints stay on; no auto-merge.

## Impact-map summary

- Revision: **2**
- PRD digest: `sha256:4be733db2a69739a2fded2165e68697ca7396525e161e0bd4c53c19242e722ee`
- Scope digest (gateflow-ops): `sha256:081ffe18350b475e0d3e9d8b3b4129e08f01f1f3ff11f0ca56bd0ff11653c220`
- Scope digest (gateflow): `sha256:a3fd7feeabf1333d010d38639e4c33657a365b7940862444f12da6b26a56f987`
- Affected repos: drivestream-lab/gateflow-ops, drivestream-lab/gateflow
- Deferred repos: none
- Not affected: prayog-skills (003 W2 owns pin), launchpad (product), prayog-meta (host)
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
map_revision: 2
meta_pr_head_sha: {SHA after this artifact is committed}
prd_digest: sha256:4be733db2a69739a2fded2165e68697ca7396525e161e0bd4c53c19242e722ee
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
    digest: sha256:9b8590c0006987fc3e8dffa94c176ca366ecaa564654eab0da7c17f1318ac629
  blockers: []
  signals:
    map_revision: 2
    previous_revision: 1
    prd_digest: sha256:4be733db2a69739a2fded2165e68697ca7396525e161e0bd4c53c19242e722ee
    affected_repos:
      - drivestream-lab/gateflow-ops
      - drivestream-lab/gateflow
    deferred_repos: []
    collision_detection: no-collision
    pr_ready: true
    existing_pr: "12-closed"
    proposed_branch: chore/INIT-GATEFLOW-004-prd
    proposed_base: develop
  next_candidates:
    - prd-pr-action
  human_checkpoint: true
  external_action: true
```
