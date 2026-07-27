---
schema_version: 1
initiative: INIT-GATEFLOW-005-BOUNDINPUT
map_revision: 1
source_prd: prd/INIT-GATEFLOW-005-BOUNDINPUT.md
source_prd_digest: sha256:40fb856dd5068290c1d14f010239e6206bee2625aaf6a6e3970d769e9bb5e970
previous_revision: null
previous_artifact_commit: null
change_reason: Initial impact map — gateflow-only delivery for bound-input skill invocation (consumes INIT-PRAYOG-SKILLS-003-PROMPTS packages)
material_change: true
generated_at: 2026-07-27T14:28:27Z
---

# Impact map — INIT-GATEFLOW-005-BOUNDINPUT — revision 1

> This file is generated locally before PR creation and becomes the scope source
> of truth when committed. Effective approval is derived from a tech-lead
> GitHub APPROVED review on the exact current meta PR head SHA; dynamic PR state
> is not stored in this artifact's frontmatter.

## 1. Source and revision

| Field | Value |
|-------|-------|
| PRD | `prd/INIT-GATEFLOW-005-BOUNDINPUT.md` |
| PRD digest | `sha256:40fb856dd5068290c1d14f010239e6206bee2625aaf6a6e3970d769e9bb5e970` |
| Map revision | `1` |
| Previous revision | none |
| Previous artifact commit | none |
| Change reason | Initial map — gateflow-only BOUNDINPUT after validation pass (report_revision 2) |
| Material change | yes — first map |

**Identity collision:** `no-collision` — no open PR/branch/Impact-Map for INIT-GATEFLOW-005-BOUNDINPUT; PR #14 is unrelated (INIT-PRAYOG-SKILLS-003-PROMPTS, merged).

## 2. Affected repositories

| Service | Repo | Team | Scope summary | Scope digest | Spec to create | Confidence |
|---------|------|------|---------------|--------------|----------------|------------|
| gateflow | drivestream-lab/gateflow | @drivestream-lab/prayog-pe-team | Bound-input skill invocation: resolve pin `prompts/`; wave-start bind (`ticket`/`initiative`); validate/render `{{var}}`; thin Cursor AgentRunner (remove invent-prose); persist `prompt_id`/`prompt_revision`/`runner`/`model_id`; **REQ-8a** define+store `run.handoff_path`; **REQ-8b** ingest only from stored path; fail closed; prove-it on any `dispatch: orchestrated` skill; programme service token auth; concurrent-run reject | `sha256:2cc5e2151451c47b973fdc86dafe19d5cb2fadd651212a5a00f9d681f274039b` | `INIT-GATEFLOW-005-BOUNDINPUT-gateflow.md` | High |

## 3. Deferred repositories

| Service | Repo | Reason | Revisit condition |
|---------|------|--------|-------------------|
| — | — | none | — |

## 4. Transitively affected repositories

| Service | Repo | Depends on | Potential impact | Disposition |
|---------|------|------------|------------------|-------------|
| prayog-skills | drivestream-lab/prayog-skills | — | Prompt packages already delivered (INIT-003); Gateflow consumes pin | **monitor** — no BOUNDINPUT delivery in skills |
| prayog-meta | drivestream-lab/prayog-meta | prayog-skills (pin) | Hosts PRD/map; pin already on prompt-capable rc-2 line | **monitor** — PRD lane only |
| launchpad | drivestream-lab/launchpad | prayog-skills (harness) | Harness sync if pin bumps; no feature work | **monitor** |
| gateflow-ops | drivestream-lab/gateflow-ops | gateflow | May later display prompt revision / handoff; **out of this INIT** | **not affected** (see §5) |

## 5. Not affected

| Service | Repo | Reason |
|---------|------|--------|
| gateflow-ops | drivestream-lab/gateflow-ops | Ops console / BFF — PRD Non-Goal; impact map gateflow-only |
| prayog-skills (feature delivery) | drivestream-lab/prayog-skills | Package SSOT already shipped under INIT-003; this INIT only consumes |
| launchpad (feature delivery) | drivestream-lab/launchpad | Factory/harness CLI — not invocation runtime |
| prayog-meta (engineering runtime) | drivestream-lab/prayog-meta | Meta hosts artifacts; no Gateflow runtime code |

## 6. Cross-repository contracts

| Contract ID | Provider repo | Consumer repo | Capability | Owner | Status |
|-------------|---------------|---------------|------------|-------|--------|
| CTR-01 | drivestream-lab/prayog-skills | drivestream-lab/gateflow | Pin prompt packages (`prompts/template.md` + `schema.yaml`); fail-closed resolve for automated runs; `prompt_id` + `prompt_revision` on outcome | prayog-pe-team | **changed** — consume now (was deferred on INIT-003 map) |
| CTR-02 | drivestream-lab/gateflow | drivestream-lab/gateflow | Wave-start known-context bind + Gateflow-owned `run.handoff_path` define/store/ingest (REQ-2, REQ-8a, REQ-8b) | prayog-pe-team | **new** |

## 7. Dependency and build order

```text
prayog-skills pin (INIT-003 packages — delivered)
  → Gate 1 (this Draft + impact map)
  → gateflow W0: resolve/bind/render/thin Cursor + REQ-8a + persist ids + prove-it hop
  → gateflow W1: REQ-8b ingest-only + isolation tests
  → gateflow W2: broaden orchestrated skills / dogfood
```

| Repo | Depends on | Reason |
|------|------------|--------|
| drivestream-lab/gateflow | drivestream-lab/prayog-skills (active pin with packages) | Consume prompt SSOT; no skills code change in this INIT |
| drivestream-lab/prayog-skills | none (this INIT) | Upstream already delivered |

## 8. Revision diff

*Omitted — map_revision 1 (initial).*

## 9. Downstream ripple ledger

| Repo | In-flight artifact | Required action | Reason | Owner | Blocking |
|------|--------------------|-----------------|--------|-------|----------|
| drivestream-lab/gateflow | none | **open** after Gate 1 approval | First affected scope | prayog-pe-team | no (pending Gate 1) |
| drivestream-lab/prayog-skills | INIT-003 delivered | **continue** / monitor | No BOUNDINPUT scope | prayog-pe-team | no |

## 10. Open questions

| ID | Lane | Question | Owner | Blocking | Required by | Default if deferred | Status |
|----|------|----------|-------|----------|-------------|---------------------|--------|
| — | — | none blocking for Gate 1 | — | — | — | — | — |

Engineering `[TBD]` items in the PRD (wave-start JSON field names, RunStore columns, concrete `handoff_path` representation, W0 skill id pick) route to the **gateflow spec PR** — not Gate 1 blockers.

## 11. PR readiness handoff

| Item | Value |
|------|-------|
| Verdict | **PR READY** |
| Collision detection | no-collision |
| Collision evidence | No Impact-Map-INIT-GATEFLOW-005-BOUNDINPUT; no open PR titled BOUNDINPUT/005; local untracked Draft+reports only; PR #14 is merged 003 (unrelated) |
| Human resolution | none |
| Resolution completed | n/a |
| Existing PR | none |
| Proposed branch | `chore/INIT-GATEFLOW-005-BOUNDINPUT-prd` |
| Proposed base | `develop` |
| Proposed title | `[INIT-GATEFLOW-005-BOUNDINPUT] PRD — Bound-input skill invocation` |
| Files to commit | `prd/INIT-GATEFLOW-005-BOUNDINPUT.md`, `prd/INIT-GATEFLOW-005-BOUNDINPUT-outline.md`, `prd/reports/Validation-Report-INIT-GATEFLOW-005-BOUNDINPUT.md`, `prd/reports/Resolution-INIT-GATEFLOW-005-BOUNDINPUT.md`, `prd/reports/Impact-Map-INIT-GATEFLOW-005-BOUNDINPUT.md` |
| Reviewer | @drivestream-lab/prayog-pe-team |
| Initial Gate 1 label | `impact-map-pending` |
| Additional invalidation label | none |
| Blocking items | none |

**No GitHub side effects have occurred.** Ask whether to create the Draft PR after explicit authorization.

### Proposed Draft PR body

```markdown
## Product change

Adds **INIT-GATEFLOW-005-BOUNDINPUT**: Gateflow runtime substrate to resolve
prayog-skills pin prompt packages, bind wave-start known context, render
`{{var}}`, dispatch thin Cursor messages (no invent-prose), persist
`prompt_id`/`prompt_revision`/runner/model, and own per-run `handoff_path`
(define/store + ingest-only). Impact map is **gateflow only**.

## Impact-map summary

- Revision: **1**
- PRD digest: `sha256:40fb856dd5068290c1d14f010239e6206bee2625aaf6a6e3970d769e9bb5e970`
- Scope digest (gateflow): `sha256:2cc5e2151451c47b973fdc86dafe19d5cb2fadd651212a5a00f9d681f274039b`
- Affected repos: drivestream-lab/gateflow
- Deferred repos: none
- Blocking questions: none
- Artifact: `prd/reports/Impact-Map-INIT-GATEFLOW-005-BOUNDINPUT.md`

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
initiative: INIT-GATEFLOW-005-BOUNDINPUT
map_revision: 1
meta_pr_head_sha: {SHA after this artifact is committed}
prd_digest: sha256:40fb856dd5068290c1d14f010239e6206bee2625aaf6a6e3970d769e9bb5e970
artifact: prd/reports/Impact-Map-INIT-GATEFLOW-005-BOUNDINPUT.md
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
    path: prd/reports/Impact-Map-INIT-GATEFLOW-005-BOUNDINPUT.md
    digest: sha256:cc04688c96395cf23de5fa75079aee49e8f6c3e7163d25774452d312a172d172
  blockers: []
  signals:
    map_revision: 1
    pr_ready: true
    affected_repos: drivestream-lab/gateflow
    prd_digest: sha256:40fb856dd5068290c1d14f010239e6206bee2625aaf6a6e3970d769e9bb5e970
    collision_detection: no-collision
  next_candidates:
    - prd-pr-action
  human_checkpoint: false
  external_action: true
```
