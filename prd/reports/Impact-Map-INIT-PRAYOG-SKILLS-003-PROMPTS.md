---
schema_version: 1
initiative: INIT-PRAYOG-SKILLS-003-PROMPTS
map_revision: 2
source_prd: prd/INIT-PRAYOG-SKILLS-003-PROMPTS.md
source_prd_digest: sha256:78e4b372e2a9d4c3aac92f0162d5ed791963908c826c06d1db153999f75c63a3
previous_revision: 1
previous_artifact_commit: 6b4e5111dda24ffe54e764d0290bfb7a03bafe62
change_reason: Fix PRD digest attestation (frontmatter/handoff had artifact digest); invalidate premature LGTM (placeholder meta_pr_head_sha + wrong digest); PRD clarifications (mental model invoke≠dispatch; normative v1 var defaults; W1 quality-variance risk); refresh Update-Summary to 13/13
material_change: true
generated_at: 2026-07-27T13:07:16Z
---

# Impact map — INIT-PRAYOG-SKILLS-003-PROMPTS — revision 2

> This file is generated locally before PR creation and becomes the scope source
> of truth when committed. Effective approval is derived from a tech-lead
> GitHub APPROVED review on the exact current meta PR head SHA; dynamic PR state
> is not stored in this artifact's frontmatter.

## 1. Source and revision

| Field | Value |
|-------|-------|
| PRD | `prd/INIT-PRAYOG-SKILLS-003-PROMPTS.md` |
| PRD digest | `sha256:08109488057ea898981c4425c6b6999a7a86156fba2b5c4a7c57c3c58915922c` |
| Map revision | `2` |
| Previous revision | `1` |
| Previous artifact commit | `6b4e5111dda24ffe54e764d0290bfb7a03bafe62` |
| Change reason | Digest attestation fix + PRD clarifications; prior LGTM stale |
| Material change | yes — digests, approval contract, and PRD wording changed |

## 2. Affected repositories

| Service | Repo | Team | Scope summary | Scope digest | Spec to create | Confidence |
|---------|------|------|---------------|--------------|----------------|------------|
| prayog-skills | drivestream-lab/prayog-skills | @drivestream-lab/prayog-pe-team | **rc-2 (`features/rc-2`):** per-skill `prompts/` for every skill under `skills/requirements/` and `skills/development/` (**13/13**); `template.md` + `schema.yaml` (`prompt_id`, semver `revision`) + `fixtures/`; shared variable dictionary with **normative v1 `required` defaults**; delivery-contract resolve (fail closed; **hand off rendered message / invoke skill** — not workflow `dispatch`; outcome returns `prompt_id` + `prompt_revision`); contract tests enforce directory coverage **independent of `dispatch`**; eval-before-promote; CHANGELOG + pin guidance on `v0.5.0-rc.2` family; humans freeform; W1 ships all 13 (no exemplar) with fixture/eval mitigation | `sha256:a76e172e081c363328842f51d1ed925a9e45b019a872e7465140643df2e6aa59` | `INIT-PRAYOG-SKILLS-003-PROMPTS-prayog-skills.md` | High |

## 3. Deferred repositories

| Service | Repo | Reason | Revisit condition |
|---------|------|--------|-------------------|
| gateflow | drivestream-lab/gateflow | Runtime bind/render/invoke/outcome consume is **INIT-GATEFLOW-005-BOUNDINPUT** (not drafted); this INIT is package SSOT only | BOUNDINPUT Draft + Gate 1; after packages exist on pin |

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
| CTR-02 | prayog-skills | orchestrator consumers | Normative resolve → validate → render → hand off / invoke → outcome ids in delivery-contract / refs | prayog-pe-team | new |
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

## 8. Revision diff

**Prior PRD digest (rev 1):** `sha256:d02b51c18897dae90eb8083b6ee209a537aa717c15539819d595ff0d3fbc0f53`  
**Current PRD digest:** `sha256:08109488057ea898981c4425c6b6999a7a86156fba2b5c4a7c57c3c58915922c`

**Prior scope digest (prayog-skills):** `sha256:b050fd0ecc4b5147b08c811e157ed2bd0a710b4407bff049a5af46bb58b606d6`  
**Current scope digest (prayog-skills):** `sha256:a76e172e081c363328842f51d1ed925a9e45b019a872e7465140643df2e6aa59`

| Repo | Prior status | Current status | Scope digest changed? | Change |
|------|--------------|----------------|-----------------------|--------|
| prayog-skills | affected | affected | yes | widened (clarifications) — normative var defaults, invoke≠dispatch wording, W1 risk |
| gateflow | deferred | deferred | n/a | unchanged disposition |
| prayog-meta | monitor | monitor | n/a | unchanged |
| launchpad | monitor | monitor | n/a | unchanged |

**Attestation note:** Rev 1 frontmatter / handoff / approval template incorrectly used the **impact-map artifact** digest (`sha256:33871850…`) as `prd_digest`. §1 table had the correct PRD file digest. Rev 2 uses one PRD digest everywhere.

## 9. Downstream ripple ledger

| Repo | In-flight artifact | Required action | Reason | Owner | Blocking |
|------|--------------------|-----------------|--------|-------|----------|
| prayog-skills | none | **hold** until map rev 2 re-approved | Rev 1 LGTM invalid (placeholder SHA + wrong digest); material PRD/map change | prayog-pe-team | yes — Gate 1 |
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
| Collision evidence | Same as rev 1; PR [#14](https://github.com/drivestream-lab/prayog-meta/pull/14) exists |
| Human resolution | none |
| Resolution completed | n/a |
| Existing PR | https://github.com/drivestream-lab/prayog-meta/pull/14 |
| Proposed branch | `chore/INIT-PRAYOG-SKILLS-003-PROMPTS-prd` |
| Proposed base | `develop` |
| Proposed title | `[INIT-PRAYOG-SKILLS-003-PROMPTS] PRD — Skill prompt packages (requirements + development)` |
| Files to commit | Draft PRD + outline + reports + this impact map rev 2 |
| Reviewer | @drivestream-lab/prayog-pe-team |
| Initial Gate 1 label | `impact-map-pending` |
| Additional invalidation label | `impact-map-revised` |
| Blocking items | Re-approval required — prior APPROVED review by `0xbeefdead` used placeholder `meta_pr_head_sha` and wrong `prd_digest` |

**Also in this PR (housekeeping, not 003 product scope):** collapse INIT-PRAYOG-SKILLS-002 `-r2`/`-r3` validation reports into the unsuffixed file.

### Proposed Draft PR body

```markdown
## Product change

Adds **versioned skill prompt packages** to prayog-skills: every skill under `skills/requirements/` and `skills/development/` (**13/13**) gets `prompts/{template.md,schema.yaml,fixtures/}` with semver `revision`. Coverage is independent of `dispatch`. Automated consumers fail closed and return `prompt_id` + `prompt_revision`; humans may freeform. Delivery on **`features/rc-2`** / `v0.5.0-rc.2` family. Runtime bind/render is **INIT-GATEFLOW-005-BOUNDINPUT** (deferred).

## Impact-map summary

- Revision: 2
- PRD digest: `sha256:08109488057ea898981c4425c6b6999a7a86156fba2b5c4a7c57c3c58915922c`
- Scope digest (prayog-skills): `sha256:a76e172e081c363328842f51d1ed925a9e45b019a872e7465140643df2e6aa59`
- Affected repos: drivestream-lab/prayog-skills
- Deferred: gateflow (BOUNDINPUT later)
- Monitor: prayog-meta (pin), launchpad (harness sync)
- Validation: clean pass (0 findings)
- Artifact: `prd/reports/Impact-Map-INIT-PRAYOG-SKILLS-003-PROMPTS.md`
- Blocking questions: none (re-approval required for rev 2)

## Housekeeping (same PR)

- Collapsed INIT-PRAYOG-SKILLS-002 `-r2`/`-r3` validation report copies into the unsuffixed report (process only; unrelated to 003 scope).

## Gate 1 — engineering handoff readiness

- [x] Draft PRD + validation clean pass
- [ ] Impact-map **rev 2** approved on current PR head (rev 1 LGTM stale)
- [ ] PE/tech lead review cites exact head SHA + PRD digest above
- [ ] Not a Joint Gate with BOUNDINPUT (downstream INIT)

Requested reviewer: @drivestream-lab/prayog-pe-team
Labels: `impact-map-pending` + `impact-map-revised`
```

## 12. Approval request (after this revision is on the PR head)

Tech lead must **Approve** on the **exact current PR head SHA** (copy from GitHub — do not paste a placeholder) using:

```text
Impact map approved
initiative: INIT-PRAYOG-SKILLS-003-PROMPTS
map_revision: 2
meta_pr_head_sha: <exact PR head SHA at approval time>
prd_digest: sha256:78e4b372e2a9d4c3aac92f0162d5ed791963908c826c06d1db153999f75c63a3
artifact: prd/reports/Impact-Map-INIT-PRAYOG-SKILLS-003-PROMPTS.md
```

**Stale approval:** Review by `0xbeefdead` on head `6b4e5111…` used `meta_pr_head_sha: {SHA after this artifact is committed}` and `prd_digest: sha256:78e4b372e2a9d4c3aac92f0162d5ed791963908c826c06d1db153999f75c63a3 (artifact digest). That review does **not** open Gate 1 for rev 2.

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
    digest: sha256:78e4b372e2a9d4c3aac92f0162d5ed791963908c826c06d1db153999f75c63a3
  blockers:
    - GATE1-REAPPROVAL
  signals:
    pr_ready: true
    map_revision: 2
    material_change: true
    prior_lgtm_stale: true
    prd_digest: sha256:78e4b372e2a9d4c3aac92f0162d5ed791963908c826c06d1db153999f75c63a3
    scope_digest_prayog_skills: sha256:a76e172e081c363328842f51d1ed925a9e45b019a872e7465140643df2e6aa59
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
    housekeeping_note: INIT-002-rN-collapse
  next_candidates:
    - prd-pr-action
  human_checkpoint: true
  external_action: true
```
