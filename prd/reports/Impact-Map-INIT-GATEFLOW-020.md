---
schema_version: 1
initiative: INIT-GATEFLOW-020
map_revision: 1
source_prd: prd/INIT-GATEFLOW-020.md
source_prd_digest: sha256:502465c74813ef0da661640322f31662178ee21cbf1c7a9f32e550f8324e2318
previous_revision: null
previous_artifact_commit: null
change_reason: Initial impact map — honest wave-start OpenCode; live hop through IaC LiteLLM; supersede INIT-GATEFLOW-003 for OpenCode only
material_change: true
generated_at: 2026-08-17T13:29:55Z
---

# Impact map — INIT-GATEFLOW-020 — revision 1

> This file is generated locally before PR creation and becomes the scope source
> of truth when committed. Effective approval is derived from a tech-lead
> GitHub APPROVED review on the exact current meta PR head SHA; dynamic PR state
> is not stored in this artifact's frontmatter.

## 1. Source and revision

| Field | Value |
|-------|-------|
| PRD | `prd/INIT-GATEFLOW-020.md` |
| PRD digest | `sha256:502465c74813ef0da661640322f31662178ee21cbf1c7a9f32e550f8324e2318` |
| Map revision | `1` |
| Previous revision | none |
| Previous artifact commit | none |
| Change reason | Initial map — mandatory runner+model start; remembered LiteLLM-derived OpenCode set; live OpenCode hop; fail-closed; new-start recovery; bind existing ops surface (no new screen) |
| Material change | yes — first map for this initiative |

**Catalog used:** `config/service-catalog-drivestream-lab.yaml` (no `config/service-catalog.yaml` in this workspace; `AGENTS.md` points at `config/service-catalog*.yaml`). Catalog services have no `owns` / `depends_on` fields; matching used `description` + `links.upstream` (gateflow-ops → gateflow). LiteLLM is IaC, not a catalog service.

**Identity collision:** `no-collision` — no `Impact-Map-INIT-GATEFLOW-020` existed; `gh pr list --state all` / `gh search prs` for `INIT-GATEFLOW-020` / `GATEFLOW-020` returned none; no local or remote branch matching `*020*`. Working tree is `develop` tracking `origin/develop` with untracked 020 PRD/report files — proposed delivery branch `chore/INIT-GATEFLOW-020-prd`, not a competing PR. Prior maps 001–017 / PRAYOG-SKILLS-* are different initiatives (`unrelated` by id). INIT-GATEFLOW-003 is the OpenCode-fail-closed predecessor, not the same initiative.

## 2. Affected repositories

| Service | Repo | Team | Scope summary | Scope digest | Spec to create | Confidence |
|---------|------|------|---------------|--------------|----------------|------------|
| gateflow | drivestream-lab/gateflow | @drivestream-lab/prayog-pe-team | Start honesty on every channel: runner **and** model after resolution (CAP-01); remembered LiteLLM-derived OpenCode set; refuse foreign/empty/missing/unauthorized; live OpenCode hop through IaC process config; stub gone; not-live Claude/Pi/unknown stay 003 fail-closed; no silent Cursor substitute; failed wave unchanged; new-start recovery (CAP-03, `OQ-04`); stage `runner`+`model_id`; no OpenCode vendor key in 014 catalogue. CAP-01–05 / REQ-01–23 / CTR-01–04. | `sha256:168535ae77cf2d5657df88e0f6aed598bf5f15cfefdade3c29118adc9e834444` | `INIT-GATEFLOW-020-gateflow.md` | High |
| gateflow-ops | drivestream-lab/gateflow-ops | @drivestream-lab/prayog-pe-team | Bind **existing** runners-and-models + start surfaces to Gateflow’s remembered OpenCode set and mandatory pair (A-02 user-confirmed; no new screen). Same honesty as API/default. Do not call LiteLLM. Do not show gateway credential. New-start after failure; do not rewrite a failed wave’s runner. CAP-01–03 / REQ-01–06, REQ-08–10, REQ-16, REQ-21, REQ-23 / CTR-01–02 consumer. | `sha256:cc2c5024b822dbbbc1135e1145e74746e2f49bbca4ac345876fa36d37db5dedd` | `INIT-GATEFLOW-020-gateflow-ops.md` | High |

**H2 payload (canonical):**

```text
repo=drivestream-lab/gateflow
status=affected
capabilities=CAP-01,CAP-02,CAP-03,CAP-04,CAP-05,REQ-01,REQ-02,REQ-03,REQ-04,REQ-05,REQ-06,REQ-07,REQ-08,REQ-09,REQ-10,REQ-11,REQ-12,REQ-13,REQ-14,REQ-15,REQ-16,REQ-17,REQ-18,REQ-19,REQ-20,REQ-21,REQ-22,REQ-23
contracts=CTR-01,CTR-02,CTR-03,CTR-04
depends_on=
scope=Mandatory runner+model on every start channel; remembered LiteLLM-derived OpenCode set; live OpenCode hop through IaC LiteLLM process config; fail-closed empty/missing/unauthorized/not-live; no stub; no silent Cursor substitute; new-start recovery without rewriting the failed wave; stage records runner+model_id; supersedes INIT-GATEFLOW-003 for OpenCode only; no LiteLLM operate; no OpenCode vendor key in 014 catalogue
```

```text
repo=drivestream-lab/gateflow-ops
status=affected
capabilities=CAP-01,CAP-02,CAP-03,REQ-01,REQ-02,REQ-03,REQ-04,REQ-05,REQ-06,REQ-08,REQ-09,REQ-10,REQ-16,REQ-21,REQ-23
contracts=CTR-01,CTR-02
depends_on=drivestream-lab/gateflow
scope=Bind existing runners-and-models and start surfaces to Gateflow remembered OpenCode set and mandatory pair; same honesty as API/default; no new ops screen; ops does not call LiteLLM; do not show gateway credential; new-start after failure; no in-place harness rewrite
```

## 3. Deferred repositories

| Service | Repo | Reason | Revisit condition |
|---------|------|--------|-------------------|
| — | — | None deferred. Live packaged-skill prove-it on OpenCode and Cursor together is a **later INIT** (`OQ-05`), not deferred work in these repos. LiteLLM deploy/UI is IaC non-goal, not a catalog deferral. | — |

## 4. Transitively affected repositories

| Service | Repo | Depends on | Potential impact | Disposition |
|---------|------|------------|------------------|-------------|
| gateflow-ops | drivestream-lab/gateflow-ops | gateflow (`links.upstream`) | Consumes CTR-01/02; start/picker must show Gateflow’s OpenCode set and refuse without a pair | **affected** — already in §2; not monitor-only |
| launchpad | drivestream-lab/launchpad | — | No CLI/harness change; LiteLLM is IaC process config, not Launchpad onboard | not affected for delivery |
| LiteLLM (IaC, external) | — | — | CTR-03 list + CTR-04 inference; this INIT consumes already-deployed gateway; does not operate LiteLLM | outbound dependency of gateflow — **not a catalog service** |

## 5. Not affected

| Service | Repo | Reason |
|---------|------|--------|
| prayog-skills | drivestream-lab/prayog-skills | Non-goal: prompt-package / pin / ForgeClient edits. REQ-12 reuses the existing packaged-skill + baton contract. Prove-it on both runners is `OQ-05` / later INIT. |
| launchpad | drivestream-lab/launchpad | Factory CLI / harness sync / playbook — not start honesty, OpenCode hop, or LiteLLM. |
| prayog-meta | drivestream-lab/prayog-meta | Hosts this PRD, outline, and impact map; not an app runtime target. 003 PRD is not edited in this pass (supersede is documented on 020). |

## 6. Cross-repository contracts

| Contract ID | Provider repo | Consumer repo | Capability | Owner | Status |
|-------------|---------------|---------------|------------|-------|--------|
| CTR-01 | gateflow | gateflow-ops and other start callers | CAP-01 / CAP-02 — accept or refuse lane start; resolved runner+model; OpenCode model must be in remembered set (REQ-01–REQ-03, REQ-05, REQ-15, REQ-23) | @drivestream-lab/prayog-pe-team | **changed** — pair mandatory; OpenCode honesty on every channel (supersedes 003 fail-closed **for OpenCode only**) |
| CTR-02 | gateflow | gateflow-ops | CAP-02 — list runners and per-runner models; OpenCode set = last successful LiteLLM list; ops does not call LiteLLM (REQ-04, REQ-06, REQ-20, REQ-21) | @drivestream-lab/prayog-pe-team | **changed** — OpenCode set is LiteLLM-derived, possibly stale |
| CTR-03 | LiteLLM (IaC) | gateflow | CAP-02 — list configured models; identity is what LiteLLM serves (REQ-05, REQ-20, REQ-23) | prayog-pe-team (consume) / IaC (provide) | **new** consume — no Gateflow OpenCode enum |
| CTR-04 | LiteLLM (IaC) | gateflow OpenCode hop | CAP-04 — inference for the start model; same identity as remembered set (REQ-11, REQ-16) | prayog-pe-team (consume) / IaC (provide) | **new** consume — hop through configured gateway |

INIT-GATEFLOW-003 OpenCode/not-live fail-fast is product-superseded **for OpenCode only**. Claude / Pi / unknown stay 003 fail-closed (REQ-15). Cursor explicit path unchanged (REQ-14). 014 env-Cursor ban does not forbid IaC LiteLLM process configuration (A-05 user-confirmed); no OpenCode vendor key in the 014 catalogue (REQ-22).

## 7. Dependency and build order

```text
IaC LiteLLM (already deployed — not this INIT)
  → gateflow (CTR-01–04: start honesty, remembered set, live OpenCode hop, fail-closed)
  → gateflow-ops (CTR-01–02 consumer — bind existing picker/start; no new screen)
```

| Repo | Depends on | Reason |
|------|------------|--------|
| gateflow | IaC LiteLLM (process config, not a catalog repo) | List + inference; fail closed if missing/empty/unauthorized |
| gateflow-ops | gateflow | Picker and start are CTR-01/02; ops must not call LiteLLM |

**Cross-repo sequencing:** do not ship an ops start that names OpenCode while gateflow still fail-closes OpenCode as 003-not-live, and do not add a new ops screen. Internal wave order is an engineering-plan concern after Gate 1; default provider then consumer. CAP-03 recovery (`OQ-04`) must not block CAP-01/02/04/05 delivery if as-built already allows a new lane start.

## 8. Revision diff

_Omit for revision 1._

## 9. Downstream ripple ledger

| Repo | In-flight artifact | Required action | Reason | Owner | Blocking |
|------|--------------------|-----------------|--------|-------|----------|
| gateflow | none (no INIT-020 app spec yet); 003 OpenCode is fail-closed as-built | **open** after Gate 1 approval | First map; primary API/worker delivery; supersedes 003 for OpenCode only | prayog-pe-team | no |
| gateflow-ops | none for 020; existing per-runner picker (A-02) | **open** after Gate 1 | Bind existing surface; no new screen | prayog-pe-team | no |
| prayog-skills | none | **continue** | Not affected (§5) | prayog-pe-team | no |
| launchpad | none | **continue** | Not affected (§5) | prayog-pe-team | no |
| prayog-meta | this PRD/map (untracked on `develop`) | **open** (content in meta PR only) | Hosts PRD/map; not an app spec | prayog-pm-team / prayog-pe-team | no |

## 10. Open questions

| ID | Lane | Question | Owner | Blocking | Required by | Default if deferred | Status |
|----|------|----------|-------|----------|-------------|---------------------|--------|
| IM-01 | PE | After a failed wave, is a second lane start on the same initiative already allowed, or is that a hidden gap? (PRD `OQ-04`) | PE | yes | spec-draft (CAP-03 / REQ-09) | If not allowed, recovery cannot ship without that start rule or a follow-up INIT; do not invent in-place rewrite | open |
| IM-02 | PE | Remember-interval duration (PRD `OQ-02`) | PE | no | feasibility | Engineering | open |
| IM-03 | PE | Wave split: gateflow start/hop waves vs gateflow-ops bind waves | PE | no | spec-implementation-plan | Sequential: gateflow honesty+live hop, then ops bind existing surface | open |
| IM-04 | PE | Confirm IaC LiteLLM process configuration is already present for the lab (PRD A-05) | PE | no | gateflow W0 | Fail closed if missing (REQ-05); this INIT does not deploy LiteLLM | open |

PRD `OQ-01` (credential rotation vs remembered set) and `OQ-06` (new start = new wave id on same initiative) stay on the PRD; neither blocks Gate 1. `OQ-05` (prove-it INIT) is out of this map. Closed `OQ-03` (ops already renders per-runner sets) is A-02; impact-map still binds the existing surface — no new screen.

## 11. PR readiness handoff

| Item | Value |
|------|-------|
| Verdict | **PR READY** |
| Collision detection | no-collision |
| Collision evidence | Searched `Impact-Map-INIT-GATEFLOW-020*` (none pre-existing), `gh pr list --state all` and `gh search prs` for `INIT-GATEFLOW-020` / `GATEFLOW-020` (empty), `git branch -a` / `git ls-remote --heads origin` for `*020*` (none). Local `develop` holds untracked 020 files — proposed branch `chore/INIT-GATEFLOW-020-prd`, not a second PR. |
| Human resolution | none |
| Resolution completed | n/a |
| Existing PR | none |
| Proposed branch | `chore/INIT-GATEFLOW-020-prd` |
| Proposed base | `develop` |
| Proposed title | `[INIT-GATEFLOW-020] PRD — Honest wave-start OpenCode` |
| Files to commit | `prd/INIT-GATEFLOW-020.md`, `prd/INIT-GATEFLOW-020-outline.md`, `prd/reports/Impact-Map-INIT-GATEFLOW-020.md`, `prd/reports/Validation-Report-INIT-GATEFLOW-020.md`, `prd/reports/Resolution-INIT-GATEFLOW-020.md`, `prd/reports/Update-Summary-INIT-GATEFLOW-020.md` |
| Reviewer | @drivestream-lab/prayog-pe-team |
| Initial Gate 1 label | `impact-map-pending` |
| Additional invalidation label | none |
| Blocking items | none for Gate 1 — IM-01 / PRD `OQ-04` blocks CAP-03 spec-draft, not map publication |

**No GitHub side effects have occurred.** Ask whether to publish via
`/commit-workspace` then `/open-draft-pr` (explicit authorization required).

### Proposed Draft PR body

```markdown
## Product change
Honest wave-start OpenCode. Every lane start must resolve a runner and a
model. OpenCode is accepted only when the model is in Gateflow’s remembered
LiteLLM-derived set; the hop runs live through already-deployed IaC LiteLLM
(no stub, no silent Cursor). Failed waves are not rewritten; the operator
starts again with another pair. Ops binds the existing picker — no new
screen. This INIT supersedes INIT-GATEFLOW-003 for OpenCode only.
Affected delivery: **gateflow** (start honesty + live hop) then
**gateflow-ops** (bind existing surface).

## Impact-map summary
- Revision: 1
- PRD digest: `sha256:502465c74813ef0da661640322f31662178ee21cbf1c7a9f32e550f8324e2318`
- Affected repos: drivestream-lab/gateflow, drivestream-lab/gateflow-ops
- Deferred repos: none
- Blocking questions: none for Gate 1 (IM-01 / OQ-04 blocks CAP-03 spec-draft)
- Artifact: `prd/reports/Impact-Map-INIT-GATEFLOW-020.md`

## Gate 1 — engineering handoff readiness
- [ ] Product/domain blocking questions are resolved in committed artifacts
- [ ] Impact-map scope and dependency order are complete
- [ ] PRD digest and map revision match this PR head
- [ ] PE/tech lead has reviewed the exact current head
- [ ] PE acknowledges 003 OpenCode fail-closed is superseded; Claude/Pi/unknown stay fail-closed
- [ ] PE acknowledges no new ops screen; LiteLLM remains IaC (not this INIT)

Requested reviewer: @drivestream-lab/prayog-pe-team
Initial label: `impact-map-pending`
```

## 12. Approval request (after Draft PR creation)

Tech lead must review this artifact on the meta PR and submit GitHub
**Approve** on the exact PR head SHA using:

```text
Impact map approved
initiative: INIT-GATEFLOW-020
map_revision: 1
meta_pr_head_sha: {SHA after this artifact is committed}
prd_digest: sha256:502465c74813ef0da661640322f31662178ee21cbf1c7a9f32e550f8324e2318
artifact: prd/reports/Impact-Map-INIT-GATEFLOW-020.md
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
    path: prd/reports/Impact-Map-INIT-GATEFLOW-020.md
    digest: sha256:502465c74813ef0da661640322f31662178ee21cbf1c7a9f32e550f8324e2318
  blockers: []
  signals:
    map_revision: 1
    source_prd_digest: sha256:502465c74813ef0da661640322f31662178ee21cbf1c7a9f32e550f8324e2318
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
    title: "[INIT-GATEFLOW-020] PRD — Honest wave-start OpenCode"
    body_path: prd/reports/Impact-Map-INIT-GATEFLOW-020.md
```
