---
schema_version: 1
initiative: INIT-GATEFLOW-011
map_revision: 1
source_prd: prd/INIT-GATEFLOW-011.md
source_prd_digest: sha256:eca06cbe986d619db58ae3ca84f4ec0987c19aa19b8f1485693280c6a655e4dd
previous_revision: null
previous_artifact_commit: null
change_reason: Initial impact map — Day-1 visibility and GitHub reconcile (gateflow only)
material_change: true
generated_at: 2026-08-06T20:01:00+05:30
---

# Impact map — INIT-GATEFLOW-011 — revision 1

> This file is generated locally before PR creation and becomes the scope source
> of truth when committed. Effective approval is derived from a tech-lead
> GitHub APPROVED review on the exact current meta PR head SHA; dynamic PR state
> is not stored in this artifact's frontmatter.

## 1. Source and revision

| Field | Value |
|-------|-------|
| PRD | `prd/INIT-GATEFLOW-011.md` |
| PRD digest | `sha256:eca06cbe986d619db58ae3ca84f4ec0987c19aa19b8f1485693280c6a655e4dd` |
| Map revision | `1` |
| Previous revision | none |
| Previous artifact commit | none |
| Change reason | Initial map for Day-1 visibility + GitHub reconcile (CAP-01…10 / REQ-01…28) |
| Material change | yes — first map |

**Identity collision:** `no-collision` — no `Impact-Map-INIT-GATEFLOW-011`, no
branch/PR for INIT-GATEFLOW-011; `gh pr list --state open` returned zero PRs;
`gh pr list --search "GATEFLOW-011"` returned zero results; no local branch
matches `*011*`.

## 2. Affected repositories

| Service | Repo | Team | Scope summary | Scope digest | Spec to create | Confidence |
|---------|------|------|---------------|--------------|----------------|------------|
| gateflow | drivestream-lab/gateflow | @drivestream-lab/prayog-pe-team | Day-1 visibility + GitHub reconcile (gateflow only): reusable checkpoint status-check (CAP-01) generalizing the existing `MetaPrIntakeService`/`ForgeClient.get_pull_request` shape, extended with new `list_reviews`/`list_check_runs` read methods (A6); check persistence/history (CAP-02); initiative list/detail with read-only meta bridge (CAP-03); spec/wave-map/implementation/closeout/merge/completion/closure read-outs (CAP-04…10); all-`GET` read-only API surface (G1) under `/api/v1/checkpoints/*`, `/api/v1/initiatives/*`; pin consume `v0.5.0-rc.2` `delivery-contract.yaml` `github.labels`+`review_roles` (never hardcoded); 0 mutating routes, 0 `apply_labels`/review/merge/`update_board_status` calls (REQ-28); waves W0–W9 | `sha256:afdc7bd51bcd0c12f614feebf338bdc78766396bb84777121a2565c0ecc7966d` | `INIT-GATEFLOW-011-gateflow.md` | High |

**H2 payload (canonical):**

```text
repo=drivestream-lab/gateflow
status=affected
capabilities=CAP-01,CAP-02,CAP-03,CAP-04,CAP-05,CAP-06,CAP-07,CAP-08,CAP-09,CAP-10,REQ-01,REQ-02,REQ-03,REQ-04,REQ-05,REQ-06,REQ-07,REQ-08,REQ-09,REQ-10,REQ-11,REQ-12,REQ-13,REQ-14,REQ-15,REQ-16,REQ-17,REQ-18,REQ-19,REQ-20,REQ-21,REQ-22,REQ-23,REQ-24,REQ-25,REQ-26,REQ-27,REQ-28
contracts=CTR-01,CTR-02,CTR-03
depends_on=prayog-skills
scope=Day-1 visibility + GitHub reconcile (gateflow only): checkpoint status-check (CAP-01) generalizing MetaPrIntakeService/ForgeClient.get_pull_request with new list_reviews/list_check_runs read methods; check persistence (CAP-02); initiative/wave/spec/implementation/closeout/completion/closure read-outs (CAP-03..10); all-GET read-only API surface (G1); pin consume v0.5.0-rc.2 delivery-contract.yaml labels+review_roles; no writes, no screens, no prayog-meta process change
```

## 3. Deferred repositories

| Service | Repo | Reason | Revisit condition |
|---------|------|--------|-------------------|
| gateflow-ops | drivestream-lab/gateflow-ops | Ops UI/screens out of scope (PRD Non-Goals; D6 "no screens, this INIT builds what a screen calls") — this INIT only builds the read-only API surface a future screen would call | Follow-on initiative once `gateflow-ops` is ready to consume Appendix A routes |
| prayog-skills | drivestream-lab/prayog-skills | Pin consume-only — **no** workflow/contract redesign in this INIT (A1, A5) | Only if the tip family changes in a way that affects checkpoint ids or `github.labels`/`review_roles` shape (separate INIT) |

## 4. Transitively affected repositories

| Service | Repo | Depends on | Potential impact | Disposition |
|---------|------|------------|-------------------|-------------|
| gateflow-ops | drivestream-lab/gateflow-ops | gateflow (upstream) | Explicit design intent: "every capability is consumable by a future `gateflow-ops` screen without further Gateflow changes" (Success Criteria) — once this INIT ships, `gateflow-ops` gains a ready-to-consume API it doesn't have today | monitor — deferred, future consumer |
| prayog-skills | drivestream-lab/prayog-skills | — (SSOT) | Gateflow remounts tip `v0.5.0-rc.2` and reads `delivery-contract.yaml` `github.labels`/`review_roles` more deeply than before (CAP-01); no redesign, but confirms the contract is now load-bearing for a second, independent consumer (checkpoint evidence, not just navigation) | monitor — deferred (consume-only) |

## 5. Not affected

| Service | Repo | Reason |
|---------|------|--------|
| prayog-meta | drivestream-lab/prayog-meta | Hosts this PRD/impact map/reports; **not** an eng delivery repo for this INIT. REQ-09/REQ-11's "read-only meta bridge" is a generic GitHub REST call (`ForgeClient.get_pull_request` against a different `owner`/`repo` argument, per A4) — no code, process, or PRD change inside `prayog-meta` itself. D5/Non-Goals explicitly exclude `prayog-meta` process changes. |
| launchpad | drivestream-lab/launchpad | No launchpad feature delivery in this PRD; harness/pin remount (outline §0) is already-completed programme practice, not new work this INIT performs |

## 6. Cross-repository contracts

| Contract ID | Provider repo | Consumer repo | Capability | Owner | Status |
|-------------|---------------|---------------|------------|-------|--------|
| CTR-01 | prayog-skills | gateflow | Pinned `workflow.yaml` + `delivery-contract.yaml` (`github.labels`, `review_roles`) @ `v0.5.0-rc.2` — consumed for checkpoint evidence resolution, not just navigation (CAP-01/REQ-02, A1/G2) | prayog-pe-team | changed — new consumption depth (labels + review_roles as gating evidence, not just workflow navigation) |
| CTR-02 | GitHub (PR labels, reviews, check-runs, head SHA, merge state) | gateflow | Read-only evidence source for checkpoint status-check (CAP-01); requires **new** `ForgeClient` read methods (`list_reviews`, `list_check_runs`) that don't exist today (A6) | prayog-pe-team | new — read-only, no write scope required |
| CTR-03 | GitHub (`prayog-meta` repo PR/label data) | gateflow | Read-only meta-PR bridge for initiative PRD-approval evidence (CAP-03/REQ-09-11, A4); same GitHub REST edge as CTR-02, different target repo | prayog-pe-team | new — read-only, no `prayog-meta`-side code |

No outbound write contract exists for this INIT — G1/REQ-28 make every new route `GET`-only; `apply_pull_request_labels`, board-status writes, and merges are explicitly out of scope (unlike INIT-GATEFLOW-010's CTR-02/CTR-03, which were write contracts).

## 7. Dependency and build order

```text
prayog-skills pin v0.5.0-rc.2 (already remounted / consume-only — confirmed this session)
  → gateflow W0 (checkpoint status-check foundation: ForgeClient extension + evidence resolver, no persistence)
  → gateflow W1 (check persistence + general checkpoint read-out — unlocks Phase 5/9)
  → gateflow W2 (initiative list/detail — gateflow-owned data)
  → gateflow W3 (initiative read-out + prayog-meta read-only bridge — CTR-03)
  → gateflow W4 (wave map read-out)
  → gateflow W5 (spec lane read-out)
  → gateflow W6 (wave implementation progress read-out)
  → gateflow W7 (closeout read-out + drift safeguard — depends on W1's persisted record)
  → gateflow W8 (merge confirm + next-wave nudge; completion eligibility — depends on W1, W4)
  → gateflow W9 (closure preview + merge-confirm reuse — depends on W1)
  → gateflow-ops (deferred — future consumer of Appendix A, out of this INIT)
```

| Repo | Depends on | Reason |
|------|------------|--------|
| gateflow | prayog-skills (pin `v0.5.0-rc.2`) | Checkpoint evidence SSOT — `delivery-contract.yaml` `github.labels`/`review_roles` (CTR-01, A1/G2) |
| gateflow | GitHub REST (read-only) | PR labels/reviews/check-runs/head SHA/merge state (CTR-02); `prayog-meta` PR/label data (CTR-03) |
| gateflow-ops | gateflow | Future consumer of the Appendix A API surface — deferred, not built this INIT |

## 8. Revision diff

_Omit for revision 1._

## 9. Downstream ripple ledger

| Repo | In-flight artifact | Required action | Reason | Owner | Blocking |
|------|--------------------|-------------------|--------|-------|----------|
| gateflow | none (no INIT-011 app spec yet) | **open** after Gate 1 approval | First map; new read-only visibility/reconcile scope on top of INIT-010's eng-lane executor | prayog-pe-team | no |
| gateflow-ops | none | hold | Deferred — future consumer, not built this INIT | prayog-pe-team | no |
| prayog-skills | tip remounted (programme, confirmed this session) | continue | Monitor / consume-only; deeper consumption depth noted (CTR-01) | prayog-pe-team | no |
| launchpad | none | continue | Monitor only — no launchpad work in this INIT | prayog-pe-team | no |

## 10. Open questions

| ID | Lane | Question | Owner | Blocking | Required by | Default if deferred | Status |
|----|------|----------|-------|----------|-------------|---------------------|--------|
| IM-01 | PE | Exact problem+json / OpenAPI response field names for Appendix A routes (**OQ-01**) | PE | no | OpenAPI / spec PR | Defer field names; route shapes + 400/404/200 semantics remain normative in PRD | open |
| IM-02 | PE | Does the `prayog-meta` read-only bridge (CTR-03, REQ-09/11) reuse the same GitHub App/`ForgeClient` credentials gateflow uses against `gateflow`/`gateflow-ops`, or a separate read-only scope (**OQ-02**)? | PE | no | W3 (initiative read-out completion) | Assume same credentials, narrower read scope if the App installation requires it; revisit only if `prayog-meta` isn't in the same GitHub App installation | open |
| IM-03 | PE | `ForgeClient` has no `list_reviews`/`list_check_runs` today (A6) — confirm W0 scope explicitly includes this client extension, not just reuse of `get_pull_request` | PE | no | W0 kickoff | Scope W0 to include the extension (already reflected in PRD Risks table); no silent underestimate | open |

## 11. PR readiness handoff

| Item | Value |
|------|-------|
| Verdict | **PR READY** |
| Collision detection | no-collision |
| Collision evidence | Searched `Impact-Map-INIT-GATEFLOW-011*` (none), branches `*011*` (none), `gh pr list --search "GATEFLOW-011"` (zero results), `gh pr list --state open` (zero open PRs in this repo currently) |
| Human resolution | none |
| Resolution completed | n/a |
| Existing PR | none |
| Proposed branch | `chore/INIT-GATEFLOW-011-prd` |
| Proposed base | `develop` |
| Proposed title | `[INIT-GATEFLOW-011] PRD — Day-1 visibility and GitHub reconcile` |
| Files to commit | `prd/INIT-GATEFLOW-011.md`, `prd/INIT-GATEFLOW-011-outline.md`, `prd/reports/Impact-Map-INIT-GATEFLOW-011.md`, `prd/reports/Validation-Report-INIT-GATEFLOW-011.md`, `prd/reports/Resolution-INIT-GATEFLOW-011.md` |
| Reviewer | @drivestream-lab/prayog-pe-team |
| Initial Gate 1 label | `impact-map-pending` |
| Additional invalidation label | none |
| Blocking items | none |

**No GitHub side effects have occurred.** Ask whether to publish via
`/commit-workspace` then `/open-draft-pr` (explicit authorization required).

### Proposed Draft PR body

```markdown
## Product change

Gateflow cannot yet answer "what's going on right now?" or "did I actually
finish what I think I finished?" — this INIT builds one reusable, read-only
checkpoint status-check (CAP-01/02) that reconciles any spec/wave/closure PR
against the pinned contract's own label/review vocabulary, plus a visibility
layer (CAP-03–10) covering every in-scope Day-1 phase. No screens, no
mutations — every new route is GET-only. Primary delivery = **gateflow only**.

## Impact-map summary
- Revision: 1
- PRD digest: `sha256:eca06cbe986d619db58ae3ca84f4ec0987c19aa19b8f1485693280c6a655e4dd`
- Affected repos: drivestream-lab/gateflow
- Deferred repos: gateflow-ops, prayog-skills (consume-only)
- Blocking questions: none (IM-01/02/03 non-blocking)
- Artifact: `prd/reports/Impact-Map-INIT-GATEFLOW-011.md`

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
initiative: INIT-GATEFLOW-011
map_revision: 1
meta_pr_head_sha: {SHA after this artifact is committed}
prd_digest: sha256:eca06cbe986d619db58ae3ca84f4ec0987c19aa19b8f1485693280c6a655e4dd
artifact: prd/reports/Impact-Map-INIT-GATEFLOW-011.md
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
    path: prd/reports/Impact-Map-INIT-GATEFLOW-011.md
    digest: sha256:efdb50bd7357532e8c882229569281211795c5f91aa29de8c637c5aaa0c4aa08
  blockers: []
  signals:
    map_revision: 1
    source_prd_digest: sha256:eca06cbe986d619db58ae3ca84f4ec0987c19aa19b8f1485693280c6a655e4dd
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
    title: "[INIT-GATEFLOW-011] PRD — Day-1 visibility and GitHub reconcile"
    body_path: prd/reports/Impact-Map-INIT-GATEFLOW-011.md
```
