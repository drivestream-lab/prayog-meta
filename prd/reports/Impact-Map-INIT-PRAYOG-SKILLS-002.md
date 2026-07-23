---
schema_version: 1
initiative: INIT-PRAYOG-SKILLS-002
map_revision: 1
source_prd: prd/INIT-PRAYOG-SKILLS-002.md
source_prd_digest: sha256:ca462e7cebbb4330ac29c7c0594807e67ee4771d4a4516f3b9e360ba810c1bba
previous_revision: null
previous_artifact_commit: null
change_reason: Initial impact map — workflow dispatch policy on prayog-skills rc-1
material_change: true
generated_at: 2026-07-23T05:38:00Z
---

# Impact map — INIT-PRAYOG-SKILLS-002 — revision 1

> This file is generated locally before PR creation and becomes the scope source
> of truth when committed. Effective approval is derived from a tech-lead
> GitHub APPROVED review on the exact current meta PR head SHA; dynamic PR state
> is not stored in this artifact's frontmatter.

## 1. Source and revision

| Field | Value |
|-------|-------|
| PRD | `prd/INIT-PRAYOG-SKILLS-002.md` |
| PRD digest | `sha256:ca462e7cebbb4330ac29c7c0594807e67ee4771d4a4516f3b9e360ba810c1bba` |
| Map revision | `1` |
| Previous revision | none |
| Previous artifact commit | none |
| Change reason | Initial impact map — `dispatch` field SSOT on prayog-skills rc-1 |
| Material change | yes — first canonical scope artifact for this initiative |

## 2. Affected repositories

| Service | Repo | Team | Scope summary | Scope digest | Spec to create | Confidence |
|---------|------|------|---------------|--------------|----------------|------------|
| prayog-skills | drivestream-lab/prayog-skills | @drivestream-lab/prayog-pe-team | **rc-1 delivery:** required `dispatch` on all **13** skill nodes (`9 manual`, `4 orchestrated`); enum `{manual, orchestrated}` for rc-1 v1; annotate full `workflow.yaml` graph; document enum + schema-default + consumer algorithm in `delivery-contract.yaml`; extend `test_workflow_contract.py` (presence, enum, wave-lane cardinality = 4); `handoff-envelope.md` cross-ref for optional future `executed_by` only; CHANGELOG + release notes; merge **rc-1** and release tag **after Joint Gate 1** with INIT-GATEFLOW-001 | `sha256:d6f9796142356e4d35a45e7ff808752ad86982da448e4a4da7ff6e2405afce2d` | `INIT-PRAYOG-SKILLS-002-prayog-skills.md` | High |

## 3. Deferred repositories

| Service | Repo | Reason | Revisit condition |
|---------|------|--------|-------------------|
| — | — | No repos deferred — single-repo contract delivery | — |

## 4. Transitively affected repositories

| Service | Repo | Depends on | Potential impact | Disposition |
|---------|------|------------|------------------|-------------|
| gateflow | drivestream-lab/gateflow | prayog-skills (pin) | PolicyEngine reads `dispatch`; paired INIT-GATEFLOW-001 | **monitor** — consumer; implementation in GATEFLOW INIT |
| prayog-meta | drivestream-lab/prayog-meta | prayog-skills (pin) | `.harness-pin.yaml` update to release tag post-merge | **monitor** — pin step after rc-1 tag + Joint Gate 1 |
| launchpad | drivestream-lab/launchpad | prayog-skills (harness) | Harness sync unchanged; no launchpad feature delivery | monitor — no W1 delivery |

## 5. Not affected

| Service | Repo | Reason |
|---------|------|--------|
| gateflow-ops | drivestream-lab/gateflow-ops | No BFF/UI; contract change only in prayog-skills SSOT |
| prayog-meta (PRD lane) | drivestream-lab/prayog-meta | Hosts PRD + impact map; harness pin is post-delivery monitor action only |

## 6. Cross-repository contracts

| Contract ID | Provider repo | Consumer repo | Capability | Owner | Status |
|-------------|---------------|---------------|------------|-------|--------|
| CTR-01 | prayog-skills | gateflow | `dispatch` enum on `type: skill` nodes; wave-lane annotation; schema default missing → `manual` | prayog-pe-team | new — pending Joint Gate 1 |
| CTR-02 | prayog-skills | orchestrator consumers | Normative consumer dispatch algorithm in `delivery-contract.yaml` | prayog-pe-team | new |
| CTR-03 | prayog-skills | prayog-meta | Release tag + pin guidance for harness after rc-1 merge | prayog-pe-team | new — post-tag |

## 7. Dependency and build order

```text
Joint Gate 1 (INIT-PRAYOG-SKILLS-002 + INIT-GATEFLOW-001)
  → prayog-skills rc-1 merge + release tag
  → prayog-meta harness pin (release tag)
  → gateflow PolicyEngine consumes dispatch (INIT-GATEFLOW-001 W1)
  → Phase B dogfood (paired)
```

| Repo | Depends on | Reason |
|------|------------|--------|
| prayog-skills | INIT-GATEFLOW-001 (Joint Gate 1) | Paired approval before merge/tag |
| prayog-meta | prayog-skills (tag) | Pin requires released contract |
| gateflow | prayog-skills (pin) | FR-5 reads pinned `workflow.yaml` |

## 8. Revision diff

*Omitted — revision 1 (initial map).*

## 9. Downstream ripple ledger

| Repo | In-flight artifact | Required action | Reason | Owner | Blocking |
|------|--------------------|-----------------|--------|-------|----------|
| prayog-skills | none (spec not yet opened) | **open** after map approval | Initial map rev 1 | prayog-pe-team | no |
| gateflow | none (GATEFLOW INIT) | **hold** | Consumer — wait for prayog-skills tag + Joint Gate 1 | prayog-pe-team | no |
| prayog-meta | none | **hold** | Harness pin after release tag | prayog-pe-team | no |

## 10. Open questions

| ID | Lane | Question | Owner | Blocking | Required by | Default if deferred | Status |
|----|------|----------|-------|----------|-------------|---------------------|--------|
| IM-01 | PM / PE | **Joint Gate 1** with INIT-GATEFLOW-001 before rc-1 merge, release tag, or Gateflow dispatch integration | PM + PE | yes | rc-1 delivery / Phase B | No merge until paired session | open |
| IM-02 | PM + PE | `observed` enum — defer rc-1 v1 vs include now? (PRD OQ #1) | PM + PE | no | rc-1 v1 enum | Defer — manual + orchestrated only | open |
| IM-03 | PE | Release tag name — `v0.5.0-rc.2` vs `v0.4.4-rc.2` vs other? (PRD OQ #3) | PE | no | Tag step | TBD at Joint Gate 1 | open |
| IM-04 | PM + PE | Release pin timing vs Gateflow W0 merge? (PRD OQ #4) | PM + PE | no | Pin + integration | Coordinate at Joint Gate 1 | open |
| IM-05 | PM + PE | Align INIT-GATEFLOW-001 FR-10 `dispatch_mode: observed` with rc-1 v1 enum deferral (PRD OQ #5) | PM + PE | no | Joint Gate 1 | Document cross-INIT alignment | open |

## 11. PR readiness handoff

| Item | Value |
|------|-------|
| Verdict | **PR READY** |
| Collision detection | no-collision |
| Collision evidence | No prior `Impact-Map-INIT-PRAYOG-SKILLS-002`; no open meta PR or branch for this initiative; outline exists only |
| Human resolution | none |
| Resolution completed | n/a |
| Existing PR | none |
| Proposed branch | `chore/INIT-PRAYOG-SKILLS-002-prd` |
| Proposed base | `develop` |
| Proposed title | `[INIT-PRAYOG-SKILLS-002] PRD — Workflow dispatch policy (rc-1)` |
| Files to commit | `prd/INIT-PRAYOG-SKILLS-002.md`, `prd/INIT-PRAYOG-SKILLS-002-outline.md`, `prd/reports/Impact-Map-INIT-PRAYOG-SKILLS-002.md`, validation r1/r2 + resolution reports; optional paired link fix in `INIT-GATEFLOW-001.md` |
| Reviewer | @drivestream-lab/prayog-pe-team |
| Initial Gate 1 label | `impact-map-pending` |
| Additional invalidation label | none |
| Blocking items | IM-01 (Joint Gate 1) — blocks rc-1 merge/tag and Gateflow dispatch only |

**No GitHub side effects have occurred.** Ask the user whether to create the Draft PR.

### Proposed Draft PR body

```markdown
## Product change

Adds **`dispatch`** field SSOT to prayog-skills: every `type: skill` node in `workflow.yaml` declares `manual` or `orchestrated`. Wave lane (`pre-implement` … `ground-spec`) is orchestrated; all upstream skills are manual. Delivery on **rc-1** branch; release tag after **Joint Gate 1** with INIT-GATEFLOW-001.

## Impact-map summary

- Revision: 1
- PRD digest: `sha256:ca462e7cebbb4330ac29c7c0594807e67ee4771d4a4516f3b9e360ba810c1bba`
- Scope digest (prayog-skills): `sha256:d6f9796142356e4d35a45e7ff808752ad86982da448e4a4da7ff6e2405afce2d`
- Affected repos: drivestream-lab/prayog-skills
- Monitor: gateflow (consumer), prayog-meta (harness pin post-tag)
- Validation: r2 clean pass (0 findings)
- Artifact: `prd/reports/Impact-Map-INIT-PRAYOG-SKILLS-002.md`
- Blocking: IM-01 Joint Gate 1 with INIT-GATEFLOW-001

## Gate 1 — engineering handoff readiness

- [x] Draft PRD + validation r2 clean pass
- [ ] Impact-map rev 1 approved on current PR head
- [ ] PE/tech lead review on exact head SHA
- [ ] Joint Gate 1 with INIT-GATEFLOW-001 (IM-01)

Requested reviewer: @drivestream-lab/prayog-pe-team
Initial label: `impact-map-pending`
```

## 12. Approval request (after PR creation)

Tech lead must approve on the exact PR head SHA after commit:

```text
Impact map approved
initiative: INIT-PRAYOG-SKILLS-002
map_revision: 1
meta_pr_head_sha: {SHA after commit}
prd_digest: sha256:ca462e7cebbb4330ac29c7c0594807e67ee4771d4a4516f3b9e360ba810c1bba
artifact: prd/reports/Impact-Map-INIT-PRAYOG-SKILLS-002.md
```

---

```yaml
handoff:
  contract: sdd-delivery/v2
  stage: prd-impact-map
  outcome: pass
  artifact:
    path: prd/reports/Impact-Map-INIT-PRAYOG-SKILLS-002.md
    digest: sha256:c9afe712eeae52f38cb3d0b8ce10f7491d1929d4564f1b4d449e63df9caa1b31
  blockers: []
  signals:
    pr_ready: true
    map_revision: 1
    material_change: true
    prd_digest: sha256:ca462e7cebbb4330ac29c7c0594807e67ee4771d4a4516f3b9e360ba810c1bba
    scope_digest_prayog_skills: sha256:d6f9796142356e4d35a45e7ff808752ad86982da448e4a4da7ff6e2405afce2d
    affected_repos:
      - drivestream-lab/prayog-skills
    deferred_repos: []
    monitor_repos:
      - drivestream-lab/gateflow
      - drivestream-lab/prayog-meta
    collision_detection: no-collision
    validation: Validation-Report-INIT-PRAYOG-SKILLS-002-r2.md (pass)
    paired_initiative: INIT-GATEFLOW-001
    joint_gate_1_blocker: IM-01
  next_candidates:
    - prd-pr-action
  human_checkpoint: true
  external_action: true
```
