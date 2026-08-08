---
schema_version: 1
initiative: INIT-GATEFLOW-013
map_revision: 1
source_prd: prd/INIT-GATEFLOW-013.md
source_prd_digest: sha256:c3653bdc5ab7f7aa679962d034a74efe3ea11040ab9db5c79b1e207a3540dbb1
previous_revision: null
previous_artifact_commit: null
change_reason: Initial impact map — Programme-first onboarding, choosing which repos matter, and real readiness checks
material_change: true
generated_at: 2026-08-08T15:50:00Z
---

# Impact map — INIT-GATEFLOW-013 — revision 1

> This file is generated locally before PR creation and becomes the scope source
> of truth when committed. Effective approval is derived from a tech-lead
> GitHub APPROVED review on the exact current meta PR head SHA; dynamic PR state
> is not stored in this artifact's frontmatter.

## 1. Source and revision

| Field | Value |
|-------|-------|
| PRD | `prd/INIT-GATEFLOW-013.md` |
| PRD digest | `sha256:c3653bdc5ab7f7aa679962d034a74efe3ea11040ab9db5c79b1e207a3540dbb1` |
| Map revision | `1` |
| Previous revision | none |
| Previous artifact commit | none |
| Change reason | Initial map — programme-first onboarding, catalogue-driven repo selection, real (Launchpad-based) readiness check replacing INIT-GATEFLOW-012's file-presence check |
| Material change | yes — first map |

**Identity collision:** `no-collision` — no `Impact-Map-INIT-GATEFLOW-013`
exists; no branch matching `*013*`; `gh pr list --state all` for
`INIT-GATEFLOW-013` / `013` returns no results; current working-tree changes
for this initiative are untracked on `develop`, not yet on any branch/PR.

## 2. Affected repositories

| Service | Repo | Team | Scope summary | Scope digest | Spec to create | Confidence |
|---------|------|------|---------------|--------------|----------------|------------|
| gateflow | drivestream-lab/gateflow | @drivestream-lab/prayog-pe-team | Programme-first onboarding: connect a tenant to its programme's meta repo (reusing INIT-GATEFLOW-012's git-workspace client); read the shared catalogue from the synced meta checkout; choose/deselect repos from it (retiring INIT-GATEFLOW-012's hand-typed `repos[]` at tenant setup — REQ-12/13); set up chosen repos (reusing the same client); run a real Launchpad `status` readiness check per repo (replacing INIT-GATEFLOW-012's file-presence check — takes over ownership of `tenant_repos.harness_verified` for newly-selected repos only, existing repos untouched); refresh the catalogue on demand. CAP-01…07 / REQ-01…28. No `gateflow-ops` UI; no `prayog-skills` contract change. | `sha256:17921af2c90e9d375914cf7e649d8dc25ad8dba7a669cf74d3d1ffe3188d719e` | `INIT-GATEFLOW-013-gateflow.md` | High |

**H2 payload (canonical):**

```text
repo=drivestream-lab/gateflow
status=affected
capabilities=CAP-01,CAP-02,CAP-03,CAP-04,CAP-05,CAP-06,CAP-07,REQ-01,REQ-02,REQ-03,REQ-04,REQ-05,REQ-06,REQ-07,REQ-08,REQ-09,REQ-10,REQ-11,REQ-12,REQ-13,REQ-14,REQ-15,REQ-16,REQ-17,REQ-18,REQ-19,REQ-20,REQ-21,REQ-22,REQ-23,REQ-24,REQ-25,REQ-26,REQ-27,REQ-28
contracts=CTR-01,CTR-02
depends_on=
scope=Programme-first onboarding: connect to the programme meta repo, read its shared catalogue, choose repos from it (retiring hand-typed tenant setup), set up chosen repos, run a real Launchpad-based readiness check replacing the file-presence check, and stay current via refresh; deselection and one-connection-per-tenant included; no gateflow-ops UI; no prayog-skills contract change
```

## 3. Deferred repositories

| Service | Repo | Reason | Revisit condition |
|---------|------|--------|-------------------|
| gateflow-ops | drivestream-lab/gateflow-ops | Onboarding UI/dashboard explicitly out of scope this INIT (PRD D10, Non-Goals) — this INIT ships only the APIs a future UI would call | Follow-on initiative, after this one ships |

## 4. Transitively affected repositories

| Service | Repo | Depends on | Potential impact | Disposition |
|---------|------|------------|-------------------|-------------|
| prayog-meta | drivestream-lab/prayog-meta | — (contract only) | This is the **first** Gateflow consumer of `config/programme.yaml` and `config/service-catalog-<org>.yaml` — gateflow now reads (clones and parses) these files at runtime. No content or code change required in prayog-meta, but these files' shape becomes a real, load-bearing contract, not just human-maintained config | monitor — no code change, but schema stability now matters to a live consumer |
| launchpad | drivestream-lab/launchpad | — (contract only) | This is the **first** Gateflow consumer of the `status --repo`/`--meta` CLI command's output — gateflow's readiness answer now depends on this command's behavior and exit/output shape. No launchpad code change required | monitor — no code change, but CLI output shape now matters to a live consumer |
| gateflow-ops | drivestream-lab/gateflow-ops | gateflow (upstream API) | This INIT's new capabilities (catalogue, selection, readiness) are the natural surface a future onboarding UI would consume | monitor — deferred (§3) |

## 5. Not affected

| Service | Repo | Reason |
|---------|------|--------|
| prayog-skills | drivestream-lab/prayog-skills | No node/action shape change needed — every capability in this INIT is pure Gateflow-internal logic, consistent with how INIT-GATEFLOW-012 itself ultimately shipped without needing the contract change it once anticipated |

## 6. Cross-repository contracts

| Contract ID | Provider repo | Consumer repo | Capability | Owner | Status |
|-------------|---------------|---------------|------------|-------|--------|
| CTR-01 | prayog-meta | gateflow | `config/programme.yaml` + `config/service-catalog-<org>.yaml` shape — read-only source of the repo catalogue (CAP-01/CAP-02, REQ-01–07) | prayog-pe-team | **new** — first consumer; no provider-side change, but this is now a real, load-bearing read contract |
| CTR-02 | launchpad | gateflow | `status --repo`/`--meta` via `--config-dir` — read-only readiness verdict (CAP-05/CAP-06, REQ-17–23) | prayog-pe-team | **new** — first consumer; no provider-side change, but CLI output shape now matters to a live caller |

## 7. Dependency and build order

```text
INIT-GATEFLOW-012 (Tenant registry + workspace lifecycle — fully shipped, all waves merged)
  → gateflow W0 (connect to programme + read catalogue — REQ-01–07,28; reuses 012's git-workspace client)
      → gateflow W1 (choose/deselect repos — REQ-08–13,26–27; retires 012's hand-typed repos[] at registration)
          → gateflow W2 (set up chosen repos — REQ-14–16; reuses same client)
              → gateflow W3 (real readiness check + one trustworthy answer — REQ-17–23; takes over tenant_repos.harness_verified for newly-selected repos)
                  → gateflow W4 (stay current / refresh — REQ-24–25)
```

| Repo | Depends on | Reason |
|------|------------|--------|
| gateflow | INIT-GATEFLOW-012 (already merged on `develop`) | Reuses its `TenantGitWorkspaceClient` and `GithubPatProbe` mechanisms unchanged, and takes over ownership of its harness-readiness gate |
| gateflow | prayog-meta (read-only, CTR-01) | Catalogue parse (CAP-02) reads its config files directly |
| gateflow | launchpad (read-only, CTR-02) | Real readiness check (CAP-05) shells out to its `status` command |

**No cross-repo build blocking exists this INIT** — unlike INIT-GATEFLOW-012 (which had a `prayog-skills` contract PR as a hard gate for later waves), this INIT's prerequisite (INIT-GATEFLOW-012) is already fully merged, and prayog-meta/launchpad require zero changes. Gateflow implementation can start immediately once Gate 1 clears.

## 8. Revision diff

_Omit for revision 1._

## 9. Downstream ripple ledger

| Repo | In-flight artifact | Required action | Reason | Owner | Blocking |
|------|--------------------|-----------------|--------|-------|----------|
| gateflow | none (no INIT-013 app spec yet) | **open** after Gate 1 approval | First map; new eng scope, single-repo delivery, no cross-repo build gate | prayog-pe-team | no |
| gateflow-ops | none | hold | Deferred (§3) | prayog-pe-team | no |
| prayog-meta | none | continue | Monitor only — new read-only consumer, no content change required | prayog-pe-team | no |
| launchpad | none | continue | Monitor only — new read-only consumer, no code change required | prayog-pe-team | no |
| prayog-skills | none | continue | Not affected (§5) — no contract change needed | prayog-pe-team | no |

## 10. Open questions

| ID | Lane | Question | Owner | Blocking | Required by | Default if deferred | Status |
|----|------|----------|-------|----------|-------------|---------------------|--------|
| IM-01 | PE | Wave-start behavior when a selected repo was never inspected, or its last inspect failed (**OQ-1**) | PE | no | gateflow implementation plan | Lean toward block outright (fail-closed), consistent with this programme's general pattern | open |
| IM-02 | PE | Where the Launchpad binary runs from inside Gateflow's execution environment (**OQ-2**) | PE/Ops | **yes** | Before implementation begins | No default — this is a new runtime/container dependency that must be explicitly provisioned | open |
| IM-03 | PE | Catalogue refresh trigger — scheduled, on-demand, or both (**OQ-3**) | PE | no | gateflow implementation plan | Start on-demand only; add scheduling later if needed | open |
| IM-04 | PE | Sub-project (submodule) credential handling during repo setup (**OQ-4**) | PE | no | gateflow implementation plan | Assume the tenant's existing credential already covers same-org submodules until proven otherwise | open |
| IM-05 | PE | Version-skew compatibility policy between Gateflow's available Launchpad binary and a repo's pinned harness profile (**OQ-5**) | PE | no | Before implementation | No default — needs an explicit policy | open |
| IM-06 | PM/PE | What currently depends on hand-typed tenant setup (scripts, runbooks, habits) that would break once it's retired (**OQ-8**) | PM | **yes** | Before implementation locks the exact cutover | No default — must be sized, not discovered after the fact | open |
| IM-07 | PE | Whether a repo that predates this initiative can ever get a fresh real check without first connecting its tenant to the programme (**OQ-9**) | PE | no | Before implementation | Genuinely undecided per the PRD itself — flagged for explicit resolution, not a default | open |
| IM-08 | PM/PE | Direct communication to the team behind INIT-GATEFLOW-012 that this initiative takes over ownership of their harness-readiness gate (a live, currently-gating capability) and retires their hand-typed repo-add path | PM | no | Before the gateflow spec PR opens | Proceed — the PRD already frames this as a considered handover (D9), not scope creep, but it deserves direct notice since INIT-GATEFLOW-012 is closed and has no open PR to comment on | open |
| IM-09 | PE | Unresolved coordination point with `INIT-GATEFLOW-004`'s onboarding scorecard — carried from both INIT-GATEFLOW-012's and this PRD's own Non-Goals | PE | no | Whichever of 004/013 lands second | Ship independently for now; reconcile later if both exist live | open |

## 11. PR readiness handoff

| Item | Value |
|------|-------|
| Verdict | **PR READY** |
| Collision detection | no-collision |
| Collision evidence | Searched `Impact-Map-INIT-GATEFLOW-013*` (none), branches matching `*013*` (none), `gh pr list --state all` for `INIT-GATEFLOW-013` and `013` (no results); current working-tree changes for this initiative are untracked, not yet on a branch/PR |
| Human resolution | none |
| Resolution completed | n/a |
| Existing PR | none |
| Proposed branch | `chore/INIT-GATEFLOW-013-prd` |
| Proposed base | `develop` |
| Proposed title | `[INIT-GATEFLOW-013] PRD — Programme-first onboarding, choosing which repos matter, and real readiness checks` |
| Files to commit | `prd/INIT-GATEFLOW-013.md`, `prd/INIT-GATEFLOW-013-outline.md`, `prd/reports/Impact-Map-INIT-GATEFLOW-013.md`, `prd/reports/Validation-Report-INIT-GATEFLOW-013.md`, `prd/reports/Resolution-INIT-GATEFLOW-013.md` |
| Reviewer | @drivestream-lab/prayog-pe-team |
| Initial Gate 1 label | `impact-map-pending` |
| Additional invalidation label | none |
| Blocking items | IM-02 (Launchpad binary provisioning — new runtime dependency) and IM-06 (what depends on the old repo-add path) — recommend both are explicitly answered before/during PE review, not silently assumed |

**No GitHub side effects have occurred.** Ask whether to publish via
`/commit-workspace` then `/open-draft-pr` (explicit authorization required).

### Proposed Draft PR body

```markdown
## Product change

Gateflow tenants currently discover and register repos by hand-typing them
one at a time, and the only readiness check that exists is a thin
file-presence guess. This initiative gives every tenant a guided path
sourced entirely from its programme's own shared records — connect once,
see the full catalogue, choose and deselect repos, get them set up
automatically — and replaces that file-presence check with a real,
Launchpad-based readiness check. Hand-typing a repo directly is retired
outright, for new and existing tenants alike. Primary delivery = **gateflow
only**; no `prayog-skills` contract change, no `gateflow-ops` UI this INIT.

## Impact-map summary
- Revision: 1
- PRD digest: `sha256:c3653bdc5ab7f7aa679962d034a74efe3ea11040ab9db5c79b1e207a3540dbb1`
- Affected repos: drivestream-lab/gateflow
- Deferred repos: gateflow-ops
- Blocking questions: IM-02 (Launchpad binary provisioning), IM-06 (legacy repo-add path dependents)
- Artifact: `prd/reports/Impact-Map-INIT-GATEFLOW-013.md`

## Gate 1 — engineering handoff readiness
- [ ] Product/domain blocking questions are resolved in committed artifacts
- [ ] Impact-map scope and dependency order are complete
- [ ] PRD digest and map revision match this PR head
- [ ] PE/tech lead has reviewed the exact current head
- [ ] PE/tech lead has weighed in on IM-02 (where the Launchpad binary runs
      from) and IM-06 (what depends on hand-typed tenant setup today)

Requested reviewer: @drivestream-lab/prayog-pe-team
Initial label: `impact-map-pending`
```

## 12. Approval request (after Draft PR creation)

Tech lead must review this artifact on the meta PR and submit GitHub
**Approve** on the exact PR head SHA using:

```text
Impact map approved
initiative: INIT-GATEFLOW-013
map_revision: 1
meta_pr_head_sha: {SHA after this artifact is committed}
prd_digest: sha256:c3653bdc5ab7f7aa679962d034a74efe3ea11040ab9db5c79b1e207a3540dbb1
artifact: prd/reports/Impact-Map-INIT-GATEFLOW-013.md
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
    path: prd/reports/Impact-Map-INIT-GATEFLOW-013.md
    digest: sha256:f3db973c24970b54feba13dbc0c4a963318872dc868d7cbcd16ea3a055febb64
  blockers:
    - IM-02
    - IM-06
  signals:
    map_revision: 1
    source_prd_digest: sha256:c3653bdc5ab7f7aa679962d034a74efe3ea11040ab9db5c79b1e207a3540dbb1
    pr_ready: true
    collision_detection: no-collision
    affected_repos:
      - drivestream-lab/gateflow
  next_candidates:
    - prd-pr-action
  human_checkpoint: false
  external_action: true
  forge:
    action: open_draft_pr
    draft: true
    apply_labels:
      - impact-map-pending
    title: "[INIT-GATEFLOW-013] PRD — Programme-first onboarding, choosing which repos matter, and real readiness checks"
    body_path: prd/reports/Impact-Map-INIT-GATEFLOW-013.md
```
