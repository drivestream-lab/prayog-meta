---
schema_version: 1
initiative: INIT-PRAYOG-SKILLS-003-PROMPTS
map_revision: 1
source_prd: prd/INIT-PRAYOG-SKILLS-003-PROMPTS.md
source_prd_digest: sha256:338718502cc182bc9968a2e4966e4886db61c0b5d6b93d711e22142cb788c3a1
previous_revision: null
previous_artifact_commit: null
change_reason: Initial impact map after validation pass (0 findings); coverage = skills/requirements/* ∪ skills/development/* (13/13)
material_change: true
generated_at: 2026-07-27T12:54:12Z
---

# Impact map — INIT-PRAYOG-SKILLS-003-PROMPTS — revision 1

> This file is generated locally before PR creation and becomes the scope source
> of truth when committed. Effective approval is derived from a tech-lead
> GitHub APPROVED review on the exact current meta PR head SHA; dynamic PR state
> is not stored in this artifact's frontmatter.

## 1. Source and revision

| Field | Value |
|-------|-------|
| PRD | `prd/INIT-PRAYOG-SKILLS-003-PROMPTS.md` |
| PRD digest | `sha256:d02b51c18897dae90eb8083b6ee209a537aa717c15539819d595ff0d3fbc0f53` |
| Map revision | `1` |
| Previous revision | none |
| Previous artifact commit | none |
| Change reason | Initial map — prompt packages for all requirements + development skills on rc-2 |
| Material change | yes — first revision |

## 2. Affected repositories

| Service | Repo | Team | Scope summary | Scope digest | Spec to create | Confidence |
|---------|------|------|---------------|--------------|----------------|------------|
| prayog-skills | drivestream-lab/prayog-skills | @drivestream-lab/prayog-pe-team | **rc-2 (`features/rc-2`):** per-skill `prompts/` for every skill under `skills/requirements/` and `skills/development/` (**13/13**); `template.md` + `schema.yaml` (`prompt_id`, semver `revision`) + `fixtures/`; shared variable dictionary; delivery-contract resolve algorithm (fail closed on automate; outcome returns `prompt_id` + `prompt_revision`); contract tests enforce directory coverage **independent of `dispatch`**; eval-before-promote; CHANGELOG + pin guidance on `v0.5.0-rc.2` family; humans freeform | `sha256:b050fd0ecc4b5147b08c811e157ed2bd0a710b4407bff049a5af46bb58b606d6` | `INIT-PRAYOG-SKILLS-003-PROMPTS-prayog-skills.md` | High |

## 3. Deferred repositories

| Service | Repo | Reason | Revisit condition |
|---------|------|--------|-------------------|
| gateflow | drivestream-lab/gateflow | Runtime bind/render/dispatch/outcome consume is **INIT-GATEFLOW-005-BOUNDINPUT** (not drafted); this INIT is package SSOT only | BOUNDINPUT Draft + Gate 1; after packages exist on pin |

## 4. Transitively affected repositories

| Service | Repo | Depends on | Potential impact | Disposition |
|---------|------|------------|------------------|-------------|
| gateflow | drivestream-lab/gateflow | prayog-skills (pin) | Will resolve prompt packages when BOUNDINPUT automates skills | **deferred** (see §3) — monitor until BOUNDINPUT |
| prayog-meta | drivestream-lab/prayog-meta | prayog-skills (pin) | `.harness-pin.yaml` bump after rc-2 tag promote carrying packages | **monitor** — post-tag pin only |
| launchpad | drivestream-lab/launchpad | prayog-skills (harness) | Harness sync after pin; no launchpad feature work | **monitor** — no W1 delivery |

## 5. Not affected

| Service | Repo | Reason |
|---------|------|--------|
| gateflow-ops | drivestream-lab/gateflow-ops | Ops console / BFF — no prompt-package or delivery-contract change |
| prayog-meta (PRD lane) | drivestream-lab/prayog-meta | Hosts PRD + impact map; engineering delivery is prayog-skills; pin is monitor-only |
| launchpad (feature delivery) | drivestream-lab/launchpad | Catalog: harness/CLI factory — not the prompt package SSOT |

## 6. Cross-repository contracts

| Contract ID | Provider repo | Consumer repo | Capability | Owner | Status |
|-------------|---------------|---------------|------------|-------|--------|
| CTR-01 | prayog-skills | gateflow (BOUNDINPUT later) | Per-skill `prompts/` layout + schema + fixtures; fail-closed resolve for automated runs | prayog-pe-team | new — consume deferred |
| CTR-02 | prayog-skills | orchestrator consumers | Normative resolve → validate → render → outcome ids in delivery-contract / refs | prayog-pe-team | new |
| CTR-03 | prayog-skills | prayog-meta | Pin/tag guidance on `v0.5.0-rc.2` family after W1 packages + W2 CHANGELOG | prayog-pe-team | new — post-tag |

## 7. Dependency and build order

```text
Gate 1 (this Draft / impact map)
  → prayog-skills W1: all 13 packages + contract tests on features/rc-2
  → prayog-skills W2: CHANGELOG / pin guidance
  → release tag on v0.5.0-rc.2 family
  → prayog-meta harness pin (monitor)
  → INIT-GATEFLOW-005-BOUNDINPUT (later) consumes packages
```

| Repo | Depends on | Reason |
|------|------------|--------|
| prayog-skills | none (this INIT) | Package SSOT; `dispatch` orthogonal (INIT-002) |
| prayog-meta | prayog-skills (tag) | Pin after promote |
| gateflow | prayog-skills (pin) + BOUNDINPUT INIT | Runtime consume out of scope here |

## 9. Downstream ripple ledger

No prior map revision — initial ledger.

| Repo | In-flight artifact | Required action | Reason | Owner | Blocking |
|------|--------------------|-----------------|--------|-------|----------|
| prayog-skills | none | **open** after Gate 1 / map approval | First delivery of prompt packages | prayog-pe-team | no |
| gateflow | none | **hold** | Deferred to BOUNDINPUT | prayog-pe-team | no |
| prayog-meta | none | **hold** | Pin after tag | prayog-pe-team | no |

## 10. Open questions

| ID | Lane | Question | Owner | Blocking | Required by | Default if deferred | Status |
|----|------|----------|-------|----------|-------------|---------------------|--------|
| IM-01 | PE | Exact release tag name on the `v0.5.0-rc.2` family after W1+W2 (reuse vs bump patch on same line) | PE | no | Tag / pin step | Stay on current rc-2 family; bump only if CHANGELOG requires | open |

## 11. PR readiness handoff

| Item | Value |
|------|-------|
| Verdict | **PR READY** |
| Collision detection | no-collision |
| Collision evidence | No `Impact-Map-INIT-PRAYOG-SKILLS-003*`; no branch `chore/INIT-PRAYOG-SKILLS-003*`; no open/merged meta PR for 003-PROMPTS. INIT-PRAYOG-SKILLS-002 (merged PR #5) is **unrelated** (dispatch SSOT vs prompt packages). Open PRs #10/#11/#13 are GATEFLOW INITs. |
| Human resolution | none |
| Resolution completed | n/a |
| Existing PR | none |
| Proposed branch | `chore/INIT-PRAYOG-SKILLS-003-PROMPTS-prd` |
| Proposed base | `develop` |
| Proposed title | `[INIT-PRAYOG-SKILLS-003-PROMPTS] PRD — Skill prompt packages (requirements + development)` |
| Files to commit | `prd/INIT-PRAYOG-SKILLS-003-PROMPTS.md`, outline, `prd/reports/Validation-Report-…`, `Resolution-Validation-Report-…`, `Update-Summary-…`, this impact map |
| Reviewer | @drivestream-lab/prayog-pe-team |
| Initial Gate 1 label | `impact-map-pending` |
| Additional invalidation label | none |
| Blocking items | none (IM-01 non-blocking) |

**No GitHub side effects have occurred.** Ask the user whether to create or update the Draft PR.

### Proposed Draft PR body

```markdown
## Product change

Adds **versioned skill prompt packages** to prayog-skills: every skill under `skills/requirements/` and `skills/development/` (**13/13**) gets `prompts/{template.md,schema.yaml,fixtures/}` with semver `revision`. Coverage is independent of `dispatch`. Automated consumers fail closed and return `prompt_id` + `prompt_revision`; humans may freeform. Delivery on **`features/rc-2`** / `v0.5.0-rc.2` family. Runtime bind/render is **INIT-GATEFLOW-005-BOUNDINPUT** (deferred).

## Impact-map summary

- Revision: 1
- PRD digest: `sha256:d02b51c18897dae90eb8083b6ee209a537aa717c15539819d595ff0d3fbc0f53`
- Scope digest (prayog-skills): `sha256:b050fd0ecc4b5147b08c811e157ed2bd0a710b4407bff049a5af46bb58b606d6`
- Affected repos: drivestream-lab/prayog-skills
- Deferred: gateflow (BOUNDINPUT later)
- Monitor: prayog-meta (pin), launchpad (harness sync)
- Validation: clean pass (0 findings)
- Artifact: `prd/reports/Impact-Map-INIT-PRAYOG-SKILLS-003-PROMPTS.md`
- Blocking questions: none

## Gate 1 — engineering handoff readiness

- [x] Draft PRD + validation clean pass
- [ ] Impact-map rev 1 approved on current PR head
- [ ] PE/tech lead review on exact head SHA
- [ ] Not a Joint Gate with BOUNDINPUT (downstream INIT)

Requested reviewer: @drivestream-lab/prayog-pe-team
Initial label: `impact-map-pending`
```

## 12. Approval request (after Draft PR creation)

Tech lead must review this artifact on the meta PR and submit GitHub
**Approve** on the exact PR head SHA using:

```text
Impact map approved
initiative: INIT-PRAYOG-SKILLS-003-PROMPTS
map_revision: 1
meta_pr_head_sha: {SHA after this artifact is committed}
prd_digest: sha256:338718502cc182bc9968a2e4966e4886db61c0b5d6b93d711e22142cb788c3a1
artifact: prd/reports/Impact-Map-INIT-PRAYOG-SKILLS-003-PROMPTS.md
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
    path: prd/reports/Impact-Map-INIT-PRAYOG-SKILLS-003-PROMPTS.md
    digest: sha256:338718502cc182bc9968a2e4966e4886db61c0b5d6b93d711e22142cb788c3a1
  blockers: []
  signals:
    pr_ready: true
    map_revision: 1
    material_change: true
    prd_digest: sha256:338718502cc182bc9968a2e4966e4886db61c0b5d6b93d711e22142cb788c3a1
    scope_digest_prayog_skills: sha256:b050fd0ecc4b5147b08c811e157ed2bd0a710b4407bff049a5af46bb58b606d6
    affected_repos:
      - drivestream-lab/prayog-skills
    deferred_repos:
      - drivestream-lab/gateflow
    monitor_repos:
      - drivestream-lab/prayog-meta
      - drivestream-lab/launchpad
    collision_detection: no-collision
    validation: Validation-Report-INIT-PRAYOG-SKILLS-003-PROMPTS.md (pass)
    paired_initiative: none
    joint_gate: false
  next_candidates:
    - prd-pr-action
  human_checkpoint: true
  external_action: true
```
