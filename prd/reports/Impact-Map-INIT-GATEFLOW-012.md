---
schema_version: 1
initiative: INIT-GATEFLOW-012
map_revision: 1
source_prd: prd/INIT-GATEFLOW-012.md
source_prd_digest: sha256:542a3680ac0a05917758c90a23c38681a20d47e0428bc30a41d539fd2f7bfb5b
previous_revision: null
previous_artifact_commit: null
change_reason: Initial impact map — Tenant registry and workspace/branch lifecycle (contract change, two repos)
material_change: true
generated_at: 2026-08-07T17:35:00Z
---

# Impact map — INIT-GATEFLOW-012 — revision 1

> This file is generated locally before PR creation and becomes the scope source
> of truth when committed. Effective approval is derived from a tech-lead
> GitHub APPROVED review on the exact current meta PR head SHA; dynamic PR state
> is not stored in this artifact's frontmatter.

## 1. Source and revision

| Field | Value |
|-------|-------|
| PRD | `prd/INIT-GATEFLOW-012.md` |
| PRD digest | `sha256:542a3680ac0a05917758c90a23c38681a20d47e0428bc30a41d539fd2f7bfb5b` |
| Map revision | `1` |
| Previous revision | none |
| Previous artifact commit | none |
| Change reason | Initial map for Tenant registry + workspace/branch lifecycle (CAP-01…06 / REQ-01…32) — **genuine `prayog-skills` contract change**, not consume-only |
| Material change | yes — first map |

**Identity collision:** `no-collision` — no `Impact-Map-INIT-GATEFLOW-012` exists yet;
no branch matching `*012*`; `gh pr list` (all states, searched for
`INIT-GATEFLOW-012` and `012`) returns no results; working tree files for this
initiative (`prd/INIT-GATEFLOW-012.md`, `-outline.md`, and the two `reports/`
files) are untracked on `develop`, not yet on any branch/PR.

## 2. Affected repositories

| Service | Repo | Team | Scope summary | Scope digest | Spec to create | Confidence |
|---------|------|------|---------------|--------------|----------------|------------|
| gateflow | drivestream-lab/gateflow | @drivestream-lab/prayog-pe-team | Tenant registry (Postgres-backed, plaintext PAT — G1) + workspace/branch lifecycle: repo clone/refresh replacing today's `Path.cwd()` fallback in `RunOrchestrator.process_job`; branch create-or-reuse via `ForgeClient.ensure_branch_from_base`/`branch_slug_from_head_ref`; harness-readiness gate extending `LaunchpadClient.sync_harness`; repo-scoped `NO_CONCURRENT_RUN` broadening in `WaveStartService`/`RunRepository.find_active_run`; new (dormant) `ForgeClient.delete_branch` method; new tenant read/list API (REQ-32). CAP-01…06 / REQ-01…27, REQ-32. No `gateflow-ops` UI; no PAT encryption/secrets-manager this INIT. | `sha256:85e75d8b61e0002b4c60aecd257aa0f9fdc99428a275083f0861f461ae04c678` | `INIT-GATEFLOW-012-gateflow.md` | High |
| prayog-skills | drivestream-lab/prayog-skills | @drivestream-lab/prayog-pe-team | **Genuine contract change** (not consume-only, unlike every prior INIT this programme has shipped): new Tenant-aware workspace-prep node shape, a branch create-or-reuse node shape, and a branch-delete forge action type declared with `authorization: explicit` (never `automated`); zero live outcome edges wired to the branch-delete action in the default `workflow.yaml` graph (structural dormancy, G5). REQ-28…31. | `sha256:5afe942947ed86667c0de224760f461fb85c2415da735e7a6909791ed094f857` | `INIT-GATEFLOW-012-prayog-skills.md` | Medium — exact node/action YAML shape is engineering's to design; the PRD fixes intent (`authorization: explicit`, zero live edges) and REQ-30/31, not syntax |

**H2 payload (canonical):**

```text
repo=drivestream-lab/gateflow
status=affected
capabilities=CAP-01,CAP-02,CAP-03,CAP-04,CAP-05,CAP-06,REQ-01,REQ-02,REQ-03,REQ-04,REQ-05,REQ-06,REQ-07,REQ-08,REQ-09,REQ-10,REQ-11,REQ-12,REQ-13,REQ-14,REQ-15,REQ-16,REQ-17,REQ-18,REQ-19,REQ-20,REQ-21,REQ-22,REQ-23,REQ-24,REQ-25,REQ-26,REQ-27,REQ-32
contracts=CTR-01,CTR-02,CTR-03
depends_on=prayog-skills
scope=Tenant registry (Postgres-backed, plaintext PAT) and workspace/branch lifecycle: repo clone/refresh replacing Path.cwd() fallback, branch create-or-reuse via ForgeClient, harness-readiness gate extending LaunchpadClient, repo-scoped NO_CONCURRENT_RUN broadening, dormant ForgeClient.delete_branch method, tenant read/list API; no gateflow-ops UI; no PAT encryption/secrets-manager this INIT
```

```text
repo=drivestream-lab/prayog-skills
status=affected
capabilities=REQ-28,REQ-29,REQ-30,REQ-31
contracts=CTR-01
depends_on=
scope=Contract change (not consume-only): new Tenant-aware workspace-prep node shape, branch create-or-reuse node shape, and branch-delete forge action type with authorization explicit; zero live outcome edges wired to branch-delete
```

## 3. Deferred repositories

| Service | Repo | Reason | Revisit condition |
|---------|------|--------|-------------------|
| gateflow-ops | drivestream-lab/gateflow-ops | Onboarding UI ("Mission Control") explicitly out of scope this INIT (PRD Non-Goals) — CAP-01's API is built so a later UI can consume it | Follow-on initiative, after this one ships the APIs |

## 4. Transitively affected repositories

| Service | Repo | Depends on | Potential impact | Disposition |
|---------|------|------------|-------------------|-------------|
| launchpad | drivestream-lab/launchpad | — (contract only) | CAP-04's harness-readiness check consumes launchpad's harness artifact scheme (`.harness-pin.yaml` / `.harness/` presence) as a read-only contract; no launchpad code changes this INIT | monitor — not in delivery scope |
| gateflow-ops | drivestream-lab/gateflow-ops | gateflow (upstream API) | CAP-01's new Tenant registry API (register/attach/read/list) is the natural surface a future onboarding UI would consume | monitor — deferred (§3) |

## 5. Not affected

| Service | Repo | Reason |
|---------|------|--------|
| prayog-meta | drivestream-lab/prayog-meta | Hosts the PRD, outline, and this impact map; not an eng delivery repo for this INIT (primary delivery = gateflow + prayog-skills) |

## 6. Cross-repository contracts

| Contract ID | Provider repo | Consumer repo | Capability | Owner | Status |
|-------------|---------------|---------------|------------|-------|--------|
| CTR-01 | prayog-skills | gateflow | Pinned `workflow.yaml` / `delivery-contract.yaml` — new Tenant-aware workspace-prep node shape, branch create-or-reuse node shape, branch-delete forge action type (CAP-01…03/06, REQ-28…31) | prayog-pe-team | **changed** — this is the contract change itself; gateflow must remount and parse the new shape with 0 BROKEN nodes |
| CTR-02 | GitHub (git + REST API) | gateflow | Outbound: `ForgeClient.ensure_branch_from_base` (existing), new `ForgeClient.delete_branch` method (CAP-06/REQ-26–27); new local git clone/fetch transport authenticated with the tenant PAT (CAP-02/REQ-10–15) — first local git execution inside Gateflow | prayog-pe-team | **changed** — new outbound git-write surface (`delete_branch`) and new local-git transport, both new this INIT |
| CTR-03 | launchpad | gateflow | Harness artifact scheme (`.harness-pin.yaml` / `.harness/` presence) — read-only contract consumed by CAP-04's harness-readiness check | prayog-pe-team | unchanged format — **newly consumed** by gateflow this INIT (extends `LaunchpadClient.sync_harness` from a path-exists-only stub to a real check) |

## 7. Dependency and build order

```text
prayog-skills contract PR (CTR-01 — new node/action shapes; REQ-28,30,31)
  → gateflow W0 (Tenant registry: data model + API + credential + user attach + read/list — REQ-01–09,32)
      → gateflow W1 (repo clone/refresh — REQ-10–15; consumes W0's tenant/repo list + credential)
          → gateflow W2 (branch create-or-reuse — REQ-16–19; consumes W1's workspace)
              → gateflow W3 (harness-readiness check — REQ-20–22; consumes CTR-03)
                  → gateflow W4 (repo-level sequencing — REQ-23–25)
                      → gateflow W5 (branch purge capability, dormant — REQ-26,27,29; depends on the prayog-skills contract PR's branch-delete shape, REQ-28)
```

| Repo | Depends on | Reason |
|------|------------|--------|
| gateflow | prayog-skills (new contract shapes, CTR-01) | W1/W2 need the workspace-prep/branch node shapes to exist in the pin before those waves can exit; W5 needs the branch-delete action declared (REQ-28) before REQ-26/27/29 can be proven end-to-end (see PRD §3 note on REQ-30/31 wave attribution) |
| gateflow | GitHub (git + REST) | Local git clone/fetch (CTR-02) and `ForgeClient.delete_branch` (CTR-02) |
| gateflow | launchpad (contract only, CTR-03) | Harness artifact scheme read by CAP-04 — no launchpad code dependency, format contract only |
| prayog-skills | — | SSOT; no upstream repo dependency for this contract change |

## 8. Revision diff

_Omit for revision 1._

## 9. Downstream ripple ledger

| Repo | In-flight artifact | Required action | Reason | Owner | Blocking |
|------|--------------------|-----------------|--------|-------|----------|
| gateflow | none (no INIT-012 app spec yet) | **open** after Gate 1 approval | First map; new eng scope, two-repo delivery | prayog-pe-team | no |
| prayog-skills | none (no INIT-012 contract spec yet) | **open** after Gate 1 approval | First map; genuine contract change, not consume-only — routed like a real spec pass | prayog-pe-team | no |
| gateflow-ops | none | hold | Deferred (§3) | prayog-pe-team | no |
| launchpad | none | continue | Monitor only — contract consumer, no code change | prayog-pe-team | no |

## 10. Open questions

| ID | Lane | Question | Owner | Blocking | Required by | Default if deferred | Status |
|----|------|----------|-------|----------|-------------|---------------------|--------|
| IM-01 | PE | Exact problem+json / OpenAPI error body field names for the new Tenant/workspace/branch routes (**OQ-01**) | PE | no | OpenAPI / spec PR | Defer field names; HTTP 400/401/422 semantics remain normative in PRD | open |
| IM-02 | PE | Exact eager-verification GitHub call for US-3 — repo metadata read vs. scoped permissions probe (**OQ-7**) | PE | no | gateflow implementation plan | Engineering routing decision, not blocking Gate 1 | open |
| IM-03 | PE | Whether the deterministic clone path needs per-wave subpaths to avoid a future worktree-isolation collision (**OQ-8**) | PE | no | Whoever revisits D10 later | Not blocking this INIT; flagged for a future initiative | open |
| IM-04 | PM/PE | Confirm PE/tech lead has read and explicitly owns G1 (plaintext-PAT-in-Postgres risk acceptance) before Gate 1 approval — this is a security posture decision, not a routine implementation detail | PM | **yes** | Gate 1 sign-off | No default — must be an explicit, recorded acknowledgment, not silent approval | open |
| IM-05 | PE | Confirm PE/tech lead is aware this INIT reopens the `delete_branch` boundary that INIT-GATEFLOW-008 and INIT-GATEFLOW-010 deliberately closed, and introduces the first local-git-credential surface inside Gateflow (PRD §7, item 4) | PM | no | Before the `prayog-skills` contract PR opens | Proceed — PRD already frames both as considered decisions (D5/G5, G2), not scope creep | open |
| IM-06 | PE | Reconcile CAP-04 (harness-readiness check)/US-3 (eager PAT verify) with `INIT-GATEFLOW-004`'s onboarding "scorecard" categories ("Harness posture," "GitHub/Forge access") — same PRD Non-Goal, carried here for cross-initiative visibility at Gate 1 | PE | no | Whichever of 004/012 lands second | Ship independently for now; reconcile later if both exist live | open |

## 11. PR readiness handoff

| Item | Value |
|------|-------|
| Verdict | **PR READY** |
| Collision detection | no-collision |
| Collision evidence | Searched `Impact-Map-INIT-GATEFLOW-012*` (none), branches matching `*012*` (none), `gh pr list --state all` for `INIT-GATEFLOW-012` and `012` (no results); current working tree changes for this initiative are untracked, not yet on a branch/PR |
| Human resolution | none |
| Resolution completed | n/a |
| Existing PR | none |
| Proposed branch | `chore/INIT-GATEFLOW-012-prd` |
| Proposed base | `develop` |
| Proposed title | `[INIT-GATEFLOW-012] PRD — Tenant registry and workspace/branch lifecycle` |
| Files to commit | `prd/INIT-GATEFLOW-012.md`, `prd/INIT-GATEFLOW-012-outline.md`, `prd/reports/Impact-Map-INIT-GATEFLOW-012.md`, `prd/reports/Validation-Report-INIT-GATEFLOW-012.md`, `prd/reports/Resolution-INIT-GATEFLOW-012.md` |
| Reviewer | @drivestream-lab/prayog-pe-team |
| Initial Gate 1 label | `impact-map-pending` |
| Additional invalidation label | none |
| Blocking items | IM-04 (G1 plaintext-PAT risk acknowledgment) — recommend PE/tech lead explicitly confirms this in the meta PR review, not a silent approval |

**No GitHub side effects have occurred.** Ask whether to publish via
`/commit-workspace` then `/open-draft-pr` (explicit authorization required).

### Proposed Draft PR body

```markdown
## Product change

Gateflow gains a Tenant registry (PAT + repo list + workspace root + board
default, registered via direct API call — no UI) and the workspace/branch
lifecycle to go with it: repo clone/refresh, branch create-or-reuse, a
harness-readiness gate, repo-scoped exclusivity, and a dormant (built but not
wired live) branch-purge capability. This is a genuine **`prayog-skills`
contract change** — new node/action shapes, not a consume-only pin bump.
Primary delivery = **gateflow + prayog-skills**. No `gateflow-ops` UI this
INIT.

## Impact-map summary
- Revision: 1
- PRD digest: `sha256:542a3680ac0a05917758c90a23c38681a20d47e0428bc30a41d539fd2f7bfb5b`
- Affected repos: drivestream-lab/gateflow, drivestream-lab/prayog-skills
- Deferred repos: gateflow-ops
- Blocking questions: IM-04 (G1 plaintext-PAT risk acknowledgment)
- Artifact: `prd/reports/Impact-Map-INIT-GATEFLOW-012.md`

## Gate 1 — engineering handoff readiness
- [ ] Product/domain blocking questions are resolved in committed artifacts
- [ ] Impact-map scope and dependency order are complete
- [ ] PRD digest and map revision match this PR head
- [ ] PE/tech lead has reviewed the exact current head
- [ ] PE/tech lead explicitly acknowledges G1 (plaintext PAT in Postgres, no
      encryption this INIT) — IM-04

Requested reviewer: @drivestream-lab/prayog-pe-team
Initial label: `impact-map-pending`
```

## 12. Approval request (after Draft PR creation)

Tech lead must review this artifact on the meta PR and submit GitHub
**Approve** on the exact PR head SHA using:

```text
Impact map approved
initiative: INIT-GATEFLOW-012
map_revision: 1
meta_pr_head_sha: {SHA after this artifact is committed}
prd_digest: sha256:542a3680ac0a05917758c90a23c38681a20d47e0428bc30a41d539fd2f7bfb5b
artifact: prd/reports/Impact-Map-INIT-GATEFLOW-012.md
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

---

```yaml
handoff:
  contract: sdd-delivery/v2
  stage: prd-impact-map
  outcome: pass
  artifact:
    path: prd/reports/Impact-Map-INIT-GATEFLOW-012.md
    digest: sha256:4ac08178bad3bfc643f69e84a358d7f7f7cd2b2571d178d24d65b8b38a38a954
  blockers:
    - IM-04
  signals:
    map_revision: 1
    source_prd_digest: sha256:542a3680ac0a05917758c90a23c38681a20d47e0428bc30a41d539fd2f7bfb5b
    pr_ready: true
    collision_detection: no-collision
    affected_repos:
      - drivestream-lab/gateflow
      - drivestream-lab/prayog-skills
  next_candidates:
    - prd-pr-action
  human_checkpoint: false
  external_action: true
  forge:
    action: open_draft_pr
    draft: true
    apply_labels:
      - impact-map-pending
    title: "[INIT-GATEFLOW-012] PRD — Tenant registry and workspace/branch lifecycle"
    body_path: prd/reports/Impact-Map-INIT-GATEFLOW-012.md
```
