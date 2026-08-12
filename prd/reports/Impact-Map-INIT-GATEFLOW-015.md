---
schema_version: 1
initiative: INIT-GATEFLOW-015
map_revision: 1
source_prd: prd/INIT-GATEFLOW-015.md
source_prd_digest: sha256:8d8b5c83c0d1ac08e56a49b3ef8636a938b5bf02475e53de4cd7108fd10e3666
previous_revision: null
previous_artifact_commit: null
change_reason: Initial impact map — skill efficacy, factory effectiveness, and delivery-copilot productivity metrics (gateflow only)
material_change: true
generated_at: 2026-08-12T07:30:00+05:30
---

# Impact map — INIT-GATEFLOW-015 — revision 1

> This file is generated locally before PR creation and becomes the scope source
> of truth when committed. Effective approval is derived from a tech-lead
> GitHub APPROVED review on the exact current meta PR head SHA; dynamic PR state
> is not stored in this artifact's frontmatter.

## 1. Source and revision

| Field | Value |
|-------|-------|
| PRD | `prd/INIT-GATEFLOW-015.md` |
| PRD digest | `sha256:8d8b5c83c0d1ac08e56a49b3ef8636a938b5bf02475e53de4cd7108fd10e3666` |
| Map revision | `1` |
| Previous revision | none |
| Previous artifact commit | none |
| Change reason | Initial map for skill efficacy, factory effectiveness, and delivery-copilot productivity metrics (CAP-01…04 / REQ-01…23) |
| Material change | yes — first map |

**Identity collision:** `no-collision` — no `Impact-Map-INIT-GATEFLOW-015` file
existed before this run; no local branch matches `*015*` or `*GATEFLOW-015*`;
`gh pr list --search "INIT-GATEFLOW-015" --state all` returned zero results;
`gh pr list --search "gateflow" --state all --limit 30` returned 24 historical
PRs, none referencing INIT-GATEFLOW-015 (highest existing PRD PR is
INIT-GATEFLOW-014); no `prd/INIT-GATEFLOW-015*` file existed anywhere in git
history (`git log --all --oneline | grep 015` returned zero rows).

## 2. Affected repositories

| Service | Repo | Team | Scope summary | Scope digest | Spec to create | Confidence |
|---------|------|------|---------------|--------------|----------------|------------|
| gateflow | drivestream-lab/gateflow | @drivestream-lab/prayog-pe-team | Skill efficacy, factory effectiveness, and delivery-copilot productivity metrics (gateflow only): CAP-01 prerequisite fix — persist the full `RunOutcomeType` vocabulary (`success/failed/stopped/blocked/findings/pending`) on `stage_completed`, replacing today's lossy `success`/`failed`/`None` mapping in `MetricsEmitter.record_stage_duration`; CAP-02 `GET /api/v1/metrics/skill-efficacy` — first-pass rate, findings rate, retry avg, learning codify rate by `workflow_node` (+ optional `model_id`/`prompt_revision`), joining `LearningRepository` items where `codify_hint.target == "skill"`; CAP-03 `GET /api/v1/metrics/factory-effectiveness` — unattended Pass-1 rate (automated `external-action` hops do not break the streak), `stop_reason` breakdown, inferred gate dwell time (`initiative_id`+`wave_id`+chronology join, no new schema column), wave cycle time by lane; CAP-04 `GET /api/v1/metrics/delivery-scorecard` — rework rate (post-`human-checkpoint`-pass findings/blocked re-entry only), initiatives closed with evidence via `InitiativeReadoutService`, factory coverage among board-tracked EPIC initiatives (explicitly not total-company coverage); all three new endpoints tenant-scoped via `require_role(RoleType.TENANT_ADMIN)`, live-aggregated on read (no new worker job, no new tables); `GET /api/v1/metrics/runs` (INIT-GATEFLOW-001/FR-10) unchanged; no per-developer attribution, no cost/token metrics, no cross-repo dependency beyond a read-only `workflow.yaml`/`codify_hint.ref` join key; waves W0–W3 | `sha256:67918ab8c946a976d58f03b4e3b5d8fe6e3ab0b0d3475028c334e4bbe52d4e72` | `INIT-GATEFLOW-015-gateflow.md` | High |

**H2 payload (canonical):**

```text
repo=drivestream-lab/gateflow
status=affected
capabilities=CAP-01,CAP-02,CAP-03,CAP-04,REQ-01,REQ-02,REQ-03,REQ-04,REQ-05,REQ-06,REQ-07,REQ-08,REQ-09,REQ-10,REQ-11,REQ-12,REQ-13,REQ-14,REQ-15,REQ-16,REQ-17,REQ-18,REQ-19,REQ-20,REQ-21,REQ-22,REQ-23
contracts=CTR-01
depends_on=prayog-skills
scope=Skill efficacy, factory effectiveness, and delivery scorecard metrics (gateflow only): outcome-persistence prerequisite fix (CAP-01); three new tenant-scoped read APIs (CAP-02..04) composing existing RunStore/checkpoint/learning data; GET /metrics/runs unchanged; no new tables; read-only workflow.yaml/codify_hint join keys only
```

## 3. Deferred repositories

| Service | Repo | Reason | Revisit condition |
|---------|------|--------|-------------------|
| gateflow-ops | drivestream-lab/gateflow-ops | Screens/charts explicitly out of scope this INIT (PRD Non-Goals: "gateflow-ops screens/charts — Later — this INIT ships JSON APIs only"). CAP-03 specifically fulfills 004's `REQ-37`/`A2` aggregate dependency (marked `[TBD in spec]` there), but this INIT only builds the API a future screen would call | Follow-on initiative once `gateflow-ops` is ready to render the three new endpoints (INIT-GATEFLOW-004 track) |

## 4. Transitively affected repositories

| Service | Repo | Depends on | Potential impact | Disposition |
|---------|------|------------|-------------------|-------------|
| gateflow-ops | drivestream-lab/gateflow-ops | gateflow (upstream) | Once this INIT ships, `gateflow-ops` gains three ready-to-consume JSON APIs (`skill-efficacy`, `factory-effectiveness`, `delivery-scorecard`) it does not have today, and a named dependency it can point at for its own REQ-37/A2 | monitor — deferred, future consumer |
| prayog-skills | drivestream-lab/prayog-skills | — (SSOT gateflow depends on) | `workflow_node` ids and `codify_hint.ref` values become load-bearing for a **third** consumption pattern (metrics joins), in addition to workflow navigation and checkpoint evidence already established by earlier INITs. No contract redesign — read-only, and the PRD (A1, D4) already names the join-drift risk explicitly | monitor — deferred (consume-only) |

## 5. Not affected

| Service | Repo | Reason |
|---------|------|--------|
| prayog-meta | drivestream-lab/prayog-meta | Hosts this PRD/impact map/reports; **not** an eng delivery repo for this INIT. Vision §10 is the source this INIT operationalizes — no vision/ADR edit is required (PRD §7 Next steps explicitly: "No prayog-meta vision/ADR supersession is required this INIT"). |
| launchpad | drivestream-lab/launchpad | No harness/onboarding/scaffold change in this PRD — factory install is untouched |

## 6. Cross-repository contracts

| Contract ID | Provider repo | Consumer repo | Capability | Owner | Status |
|-------------|---------------|---------------|------------|-------|--------|
| CTR-01 | prayog-skills | gateflow | `workflow.yaml` node ids + `codify_hint.ref` values as read-only join keys for CAP-02's learning-codify-rate join (REQ-08/REQ-09) and CAP-01/CAP-03's `workflow_node`/`stop_reason` grouping (REQ-01, REQ-13) | prayog-pe-team | unchanged — deeper read consumption of an existing pin surface, no shape change requested |

No outbound write contract exists for this INIT — all three new routes are
`GET`-only (D7: live aggregation on read); no `apply_labels`, board-status
writes, or forge mutations are introduced.

## 7. Dependency and build order

```text
prayog-skills pin (existing — consume-only, no change requested)
  → gateflow W0 (CAP-01: persist full RunOutcomeType on stage_completed — prerequisite)
      → gateflow W1 (CAP-02: Skill/Spec Efficacy API)
          → gateflow W2 (CAP-03: Factory Effectiveness API, incl. inferred dwell-time join)
              → gateflow W3 (CAP-04: Delivery Scorecard API, gateflow-owned half)
  → gateflow-ops (deferred — future consumer of the three new endpoints, out of this INIT)
```

| Repo | Depends on | Reason |
|------|------------|--------|
| gateflow | prayog-skills (pin, consume-only) | `workflow_node` ids + `codify_hint.ref` join keys (CTR-01) |
| gateflow-ops | gateflow | Future consumer of the three new JSON APIs — deferred, not built this INIT |

## 8. Revision diff

_Omit for revision 1._

## 9. Downstream ripple ledger

| Repo | In-flight artifact | Required action | Reason | Owner | Blocking |
|------|--------------------|-----------------|--------|-------|----------|
| gateflow | none (no INIT-015 app spec yet) | **open** after Gate 1 approval | First map; new metrics/scorecard scope on top of INIT-GATEFLOW-001's delivered metrics v0 | prayog-pe-team | no |
| gateflow-ops | none | hold | Deferred — future consumer, not built this INIT | prayog-pe-team | no |
| prayog-skills | tip already remounted (programme, no change requested) | continue | Monitor / consume-only; deeper consumption depth noted (CTR-01) | prayog-pe-team | no |
| launchpad | none | continue | Monitor only — no launchpad work in this INIT | prayog-pe-team | no |

## 10. Open questions

| ID | Lane | Question | Owner | Blocking | Required by | Default if deferred | Status |
|----|------|----------|-------|----------|-------------|---------------------|--------|
| IM-01 | PE | Exact JSON field names / OpenAPI response shapes for the three new endpoints (Appendix B/C in the PRD are explicitly illustrative) | PE | no | Spec PR | Defer field names; route shapes + tenant-scoping + `as_of`/`retention_days` semantics remain normative in the PRD | open |
| IM-02 | PE | Should `prayog-skills` formally acknowledge that `workflow_node` id stability is now load-bearing for a third consumer (metrics joins), beyond navigation and checkpoint evidence already noted in earlier maps? | PE | no | Not required this INIT | No action required this INIT; the PRD already names the join-drift risk and treats it as best-effort (unmatched refs report as "unjoined," never erroring) | open |
| IM-03 | PE | Does the new cross-reference from INIT-GATEFLOW-015 to INIT-GATEFLOW-004's `REQ-37`/`A2` require a reciprocal update to INIT-GATEFLOW-004's own PRD, or is the one-way reference from 015 sufficient? | PE | no | Before/at gateflow-ops consumption (follow-on INIT) | One-way reference from 015 is sufficient for this INIT's exit; revisit only when a gateflow-ops-consuming INIT is scoped | open |

## 11. PR readiness handoff

| Item | Value |
|------|-------|
| Verdict | **PR READY** |
| Collision detection | no-collision |
| Collision evidence | Searched `Impact-Map-INIT-GATEFLOW-015*` (none existed before this run), branches `*015*` (none), `gh pr list --search "INIT-GATEFLOW-015" --state all` (zero results), `gh pr list --search "gateflow" --state all --limit 30` (24 historical PRs, none for 015; highest is INIT-GATEFLOW-014), `git log --all --oneline \| grep 015` (zero rows) |
| Human resolution | none |
| Resolution completed | n/a |
| Existing PR | none |
| Proposed branch | `chore/INIT-GATEFLOW-015-prd` |
| Proposed base | `develop` |
| Proposed title | `[INIT-GATEFLOW-015] PRD — Skill efficacy, factory effectiveness, and delivery-copilot productivity metrics` |
| Files to commit | `prd/INIT-GATEFLOW-015.md`, `prd/INIT-GATEFLOW-015-outline.md`, `prd/reports/Impact-Map-INIT-GATEFLOW-015.md`, `prd/reports/Validation-Report-INIT-GATEFLOW-015.md`, `prd/reports/Resolution-INIT-GATEFLOW-015.md` |
| Reviewer | @drivestream-lab/prayog-pe-team |
| Initial Gate 1 label | `impact-map-pending` |
| Additional invalidation label | none |
| Blocking items | none |

**No GitHub side effects have occurred.** Ask whether to publish via
`/commit-workspace` then `/open-draft-pr` (explicit authorization required).

### Proposed Draft PR body

```markdown
## Product change

Gateflow records almost everything needed to answer "is this working," but
computes almost none of it — today's only derived signal (`GET /metrics/runs`)
answers how long a stage took, never how well or whether it's improving. This
INIT fixes a lossy outcome-persistence gap (CAP-01, prerequisite), then ships
three additive, tenant-scoped, live-aggregated read APIs turning existing
RunStore/checkpoint/learning data into decision-grade rates: Skill/Spec
Efficacy (CAP-02), Factory Effectiveness (CAP-03 — fulfills INIT-GATEFLOW-004's
REQ-37/A2 dependency), and a Delivery Scorecard covering Gateflow's own
evidence half (CAP-04). `GET /metrics/runs` is unchanged. Primary delivery =
**gateflow only**.

## Impact-map summary
- Revision: 1
- PRD digest: `sha256:8d8b5c83c0d1ac08e56a49b3ef8636a938b5bf02475e53de4cd7108fd10e3666`
- Affected repos: drivestream-lab/gateflow
- Deferred repos: gateflow-ops
- Blocking questions: none (IM-01/02/03 non-blocking)
- Artifact: `prd/reports/Impact-Map-INIT-GATEFLOW-015.md`

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
initiative: INIT-GATEFLOW-015
map_revision: 1
meta_pr_head_sha: {SHA after this artifact is committed}
prd_digest: sha256:8d8b5c83c0d1ac08e56a49b3ef8636a938b5bf02475e53de4cd7108fd10e3666
artifact: prd/reports/Impact-Map-INIT-GATEFLOW-015.md
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
    path: prd/reports/Impact-Map-INIT-GATEFLOW-015.md
    digest: sha256:584f2f7c29cceb71048107fea2074e882374e3b91a71c127fc0cdc7e68a4ff27
  blockers: []
  signals:
    map_revision: 1
    source_prd_digest: sha256:8d8b5c83c0d1ac08e56a49b3ef8636a938b5bf02475e53de4cd7108fd10e3666
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
    title: "[INIT-GATEFLOW-015] PRD — Skill efficacy, factory effectiveness, and delivery-copilot productivity metrics"
    body_path: prd/reports/Impact-Map-INIT-GATEFLOW-015.md
```
