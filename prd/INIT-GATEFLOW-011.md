# INIT-GATEFLOW-011 — Day-1 visibility and GitHub reconcile

**Status:** draft PRD · **Author:** programme PM · **Date:** 2026-08-06  
**Outline:** [INIT-GATEFLOW-011-outline](./INIT-GATEFLOW-011-outline.md)  
**Vision:** [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md)  
**Component:** GATEFLOW · **Type:** platform / delivery control plane  
**Predecessor:** INIT-GATEFLOW-010 (engineering-lane pin tip executor parity)

> **Draft PRD** — Locked decisions D1–D6 (outline) carried forward; G1–G7 added
> below from Draft PRD review. Pin SSOT: prayog-skills `sdd-delivery/v2` @
> **`v0.5.0-rc.2`** tip. Primary delivery = **gateflow only**; `gateflow-ops`
> screens and `prayog-meta` process are explicitly out of scope. This INIT
> adds **no mutation surface** — every capability below is a read-only
> observer of state that already exists in GitHub, the board, or Gateflow's
> own run store.
>
> **Pre-flight gate (outline §0, not build scope) — CONFIRMED COMPLETE.**
> Verified directly against local checkouts of all three repos (not just this
> workspace): `gateflow`, `gateflow-ops`, and `prayog-meta` all pin
> `prayog-skills` submodule at `ebaa912b73fcb10392e2046101993a1f8f0af4a2` —
> the current SSOT tip. Both `gateflow`'s and `gateflow-ops`'s vendored
> `workflow.yaml` show `wave-acceptance` (not `live-verify`); `gateflow`'s own
> `.harness-pin.yaml` skill list no longer includes `/verify`; and
> `docs/specification/as-built/implementation-status.md` explicitly records
> **"remount hygiene 2026-08-06."** The outline's §0 evidence table (all three
> at `6561c7c`, 3 commits behind) is now stale — the remount already
> happened. `/spec-draft` for this INIT is unblocked.

---

## Document control

| Field | Value |
|-------|-------|
| Initiative ID | INIT-GATEFLOW-011 |
| Programme | prayog |
| Primary repo | drivestream-lab/gateflow |
| Supporting repos | prayog-meta (this PRD; read-only meta-PR bridge, Phase 1 only); prayog-skills (pin consume-only) |
| Explicitly not touched | gateflow-ops (no screens built here) |
| Skills pin | `agent_skills.ref: v0.5.0-rc.2` tip — consume only, no redesign |
| Delivery contract | `sdd-delivery/v2` — checkpoint evidence sourced from pinned `delivery-contract.yaml` (`github.labels`, `review_roles`), never hardcoded per phase |
| Primary delivery | **gateflow only**; no `gateflow-ops` UI, no `prayog-meta` process change |
| Target users | Developer/engineer running the delivery loop; engineering lead / PE reviewing gates |
| Draft locks | All new routes are **GET-only** (structural enforcement of D1 — no POST/PUT/DELETE surface in this INIT); every checkpoint status-check persists a check record (checked SHA + timestamp + verdict); waves **W5–W9** added in this Draft PRD to close the outline §5/§8 coverage gap (PM judgement — see G6) |
| As-built baseline | 2026-08-06 · all three repos (`gateflow`, `gateflow-ops`, `prayog-meta`) confirmed at `prayog-skills` tip `ebaa912b73fcb10392e2046101993a1f8f0af4a2` by direct inspection of local checkouts (Source: verified this session — submodule status + vendored `workflow.yaml` + `gateflow`'s own `implementation-status.md` "remount hygiene 2026-08-06" entry). §0 gate is satisfied |
| Depends on | INIT-GATEFLOW-010 (eng lanes proven and human-gated correctly); outline §0 remount (satisfied — see As-built baseline) |

---

## 1. Executive Summary

### Problem Statement

A developer can do every real step of the delivery loop — review a spec,
code a wave, verify it, merge it — but cannot do any of it *through*
Gateflow, because Gateflow cannot yet answer two basic questions: **"what's
going on right now?"** (no initiative list, no wave map, no run/checkpoint
summary) and **"did I actually finish what I think I finished?"** (no way to
ask Gateflow to confirm a GitHub approval — a label, a review, a merge — is
real, current, and complete before relying on it). `(Source: outline §1,
User-confirmed)`

### Proposed Solution

Build one reusable, read-only **checkpoint status-check** capability
(`CAP-01`/`CAP-02`) that inspects a PR's current head against the **pinned
contract's own vocabulary** (`delivery-contract.yaml` labels — `spec-lgtm`,
`wave-accepted`, `impact-map-lgtm` — and `review_roles`), and reuse it at
every phase that needs a reconcile (spec completion, wave acceptance, wave
merge, closure merge) instead of rebuilding a checker per phase. Pair it with
a **visibility layer** (`CAP-03`–`CAP-10`) that reflects state Gateflow
already owns (runs, board tickets) or can read read-only (prayog-meta PR
state), covering every in-scope Day-1 phase named in the outline — including
Phases 2, 7, 10, 12, 13, which the outline's own §8 wave plan left unmapped
(closed here by **W5–W9**, see G6).

No screens are built here (`gateflow-ops`, later). No label, approval, or
merge is ever applied by Gateflow (`D1`, unchanged from INIT-GATEFLOW-010).

### Success Criteria

| KPI | Target | Measurement |
|-----|--------|-------------|
| **Reconcile accuracy** | Given any spec/wave/closure PR, checkpoint status-check returns a specific, accurate, current answer — never a stale one | Unit + live verify against fixture PRs with fresh vs. stale evidence |
| **Version fidelity** | Every check response includes `checked_sha` + `checked_at`; a satisfying label/approval dated before a later commit is reported `stale`, not `pass` | Unit (REQ-03) |
| **Specific misses, not booleans** | Every failing check lists each missing item by name (e.g. "wave-accepted label not present", "2 required checks still running") | Unit + verify |
| **Read-only surface** | 0 mutating routes (no POST/PUT/DELETE) shipped in this INIT; 0 `apply_labels` / review / merge / `update_board_status` calls from any new code path | Code guard + route inventory audit (REQ-28) |
| **Initiative visibility** | A developer can list all initiatives (approval state, affected repos, stage) and see an initiative's wave map (done/ready/blocked/active) without asking a colleague | Manual UAT against `CAP-03`, `CAP-05` |
| **Full-scope coverage** | Every phase named in outline §5 (1, 2, 5, 6, 7, 9, 10, 11, 12, 13) has a shipping capability and an exit wave | Wave-to-phase traceability table (§5 below) |
| **Screen-ready** | Every capability below is consumable by a future `gateflow-ops` screen without further Gateflow changes | Design review against Appendix A routes |

### Product id map

| CAP | Phase(s) | REQ | User story |
|-----|----------|-----|------------|
| CAP-01 Checkpoint status-check (core) | 5, 9, 11, 13 (reused) | REQ-01–REQ-05 | US-1 |
| CAP-02 Check persistence / history | cross-cutting | REQ-06–REQ-08 | US-2 |
| CAP-03 Initiative list & detail | 1 | REQ-09–REQ-11 | US-3 |
| CAP-04 Spec lane read-out | 2 | REQ-12–REQ-13 | US-4 |
| CAP-05 Wave map read-out | 6 | REQ-14–REQ-15 | US-5 |
| CAP-06 Wave implementation progress | 7 | REQ-16–REQ-17 | US-6 |
| CAP-07 Closeout read-out + drift safeguard | 10 | REQ-18–REQ-20 | US-7 |
| CAP-08 Merge confirm + next-wave nudge | 11 | REQ-21–REQ-22 | US-8 |
| CAP-09 Completion eligibility | 12 | REQ-23–REQ-24 | US-9 |
| CAP-10 Closure preview + reuse | 13 | REQ-25–REQ-27 | US-10 |

---

## 2. User Experience & Functionality

### User Personas

| Persona | Role | Primary need |
|---------|------|---------------|
| **Developer/engineer** | Runs the delivery loop day to day | Open one place, see what's ready/waiting, do review/coding/verification, get a clear current answer on whether they're clear to move on |
| **Engineering lead / PE** | Reviews gates. Not a distinct authenticated role — Gateflow uses a shared PAT/programme token; anyone with repo access can occupy this persona. `(Source: User-confirmed)` | A clear record of when something was checked and against what version; trust that nothing moves forward on a stale or incomplete approval |

### User Stories & Acceptance Criteria

#### US-1 — Reconcile any checkpoint on demand `(CAP-01)`

**As a** developer, **I want** to ask Gateflow whether a spec/wave/closure PR's
approval is real, current, and complete, **so that** I never proceed on a
stale or incomplete decision.

**Acceptance criteria:**

- [ ] Accepts a checkpoint reference (checkpoint node id ∈ `coding-readiness`,
      `wave-acceptance`, `wave-signoff`, `initiative-closure-signoff-app`,
      `initiative-closure-signoff-meta`, `prd-impact-acceptance`) plus enough
      identifiers to resolve the PR (initiative/wave id, or explicit
      org/repo/PR number)
- [ ] Evidence checked = pinned `delivery-contract.yaml` **labels** for that
      checkpoint's lane (e.g. `spec-lgtm` present, `spec-blocked` /
      `spec-revised` / `spec-stale` absent) + **review** state for the
      checkpoint's `review_roles` entry + required CI check-run conclusions —
      all read live from GitHub at call time, never hardcoded per phase (D4)
- [ ] Response always includes `checked_sha` (PR head SHA at call time) and
      `checked_at` (timestamp)
- [ ] If the satisfying label/review predates a later commit on the same PR,
      the response is `NOT SATISFIED` with reason `stale — new commits since
      approval` — never silently `pass` (D2)
- [ ] On any non-pass result, the response lists each missing/unsatisfied
      item by name, not a single boolean (D3)
- [ ] The endpoint never calls `apply_labels`, creates/updates a review,
      merges, or calls `update_board_status` — code guard + verify assert 0
      write calls from this path (D1)

#### US-2 — Every check leaves a record `(CAP-02)`

**As an** engineering lead, **I want** every checkpoint status-check to be
recorded, **so that** there's a durable trail of what was checked, when, and
against what code.

**Acceptance criteria:**

- [ ] Every CAP-01 call persists a check record (checkpoint id, PR reference,
      `checked_sha`, `checked_at`, verdict, missing items) into Gateflow's
      existing run/timeline store, correlated to the initiative/wave when
      resolvable
- [ ] A separate history read-out returns prior persisted records for
      audit/timeline display; it is labeled **historical** and is never
      substituted for a live check when a caller needs the current pass/fail
      decision (G5)
- [ ] The general checkpoint read-out used by Phases 5/9/11/13 composes the
      live CAP-01 check with the initiative/wave/run identifiers Gateflow
      already has, so callers don't separately resolve which PR belongs to
      which wave

#### US-3 — See every initiative at a glance `(CAP-03)`

**As a** developer, **I want** a list of initiatives with approval state,
affected repos, and current stage, **so that** I can correctly say what's
ready and what's moving, unprompted.

**Acceptance criteria:**

- [ ] List/detail returns per initiative: id, name/description, PRD approval
      state (derived via CAP-01 evidence rules against `prd-impact-acceptance`
      — `impact-map-lgtm` + Approve on exact head), affected repos, current
      stage in plain language, link to any in-flight run
- [ ] Composed only from data Gateflow already owns (runs, board tickets) plus
      one read-only read of prayog-meta's PR/label state for the PRD-approval
      line — no new source of truth invented (D4)
- [ ] If prayog-meta is unreachable, gateflow-owned fields still return; the
      meta-derived field is explicitly marked `unavailable`, not a full
      failure of the read-out

#### US-4 — See what the spec pass produced `(CAP-04)`

**As a** developer, **I want** a read-out of the automated spec pass,
**so that** I go from starting the spec lane to reviewing the right branch in
under a minute.

**Acceptance criteria:**

- [ ] Returns the Draft Spec PR link (from `spec-pr-action`), a plain-language
      list of generated artifacts, findings/open questions from
      `initiative-feasibility` / `spec-technical-review`, and the exact next
      step resolved from the pin (what to check out, what to run)
- [ ] If the spec walk has not reached `spec-pr-action` yet, the read-out
      says so plainly instead of returning a broken or missing link

#### US-5 — See the whole shape of remaining work `(CAP-05)`

**As a** developer, **I want** a wave map for an initiative, **so that** I see
done/ready-to-start/blocked/active at a glance.

**Acceptance criteria:**

- [ ] Per wave: status ∈ `done` / `ready-to-start` / `blocked` / `active`,
      derived from board ticket status + run state
- [ ] If `blocked`, the read-out names why (e.g. "predecessor wave W1 not
      Done")
- [ ] Derivation uses only existing board/run data — no new wave-state store
      invented (D4)

#### US-6 — Watch a wave unfold without leaving Gateflow `(CAP-06)`

**As a** developer, **I want** task-by-task progress on an in-progress wave,
**so that** the coding pass doesn't look like a black-box spinner.

**Acceptance criteria:**

- [ ] Returns task-by-task progress from the run's task/step timeline, not a
      single spinner
- [ ] Returns the resulting Draft PR link the moment `wave-pr-action` succeeds
- [ ] If a task fails or the run stops (`needs-input`), the read-out names
      which task and why

#### US-7 — Trust closeout because you can see it `(CAP-07)`

**As a** developer, **I want** to see what closeout added and be warned if
code changed after I accepted the wave, **so that** I never trust a silent
black box right before merge.

**Acceptance criteria:**

- [ ] Returns what closeout added (lessons captured, learning-store records
      updated) after `learning-extract` / `ground-spec` complete
- [ ] **Drift safeguard:** compares the wave PR's head SHA recorded the last
      time `wave-acceptance` was satisfied (CAP-02 persisted record) against
      the PR's head SHA at closeout time; if they differ, flags "product code
      changed after acceptance" explicitly instead of proceeding silently;
      if no baseline was ever recorded for this wave, reports "unknown — no
      baseline recorded" rather than silently skipping
- [ ] The safeguard is advisory/read-only — it does not block or fail
      closeout; `learning-extract → ground-spec → wave-done-action →
      wave-signoff` mechanics are unchanged (D1, D6)

#### US-8 — One continuous flow from merge to next wave `(CAP-08)`

**As a** developer, **I want** Gateflow to confirm a wave PR really merged and
tell me if the next wave just unblocked, **so that** finishing one wave and
starting the next feels continuous.

**Acceptance criteria:**

- [ ] Reuses CAP-01 evidence rules against `wave-signoff` to report merged
      state + merge commit SHA, or specific missing items (e.g. "not yet
      merged")
- [ ] After a confirmed merge, if `wave-complete` resolves to another wave
      whose predecessor is now Done, the read-out surfaces "wave W`n+1` is
      now unblocked" rather than requiring manual re-derivation from the wave
      map

#### US-9 — No manual tallying of wave statuses `(CAP-09)`

**As a** developer, **I want** a single "ready to close" / "waiting on wave N"
read-out, **so that** I don't manually tally wave statuses.

**Acceptance criteria:**

- [ ] Reports "ready to close" only when every `wave_ticket_ids` entry for the
      initiative is board Done (same Done semantics as
      INIT-GATEFLOW-010 REQ-13); if `wave_ticket_ids` is empty or
      unresolvable, reports "no waves found" instead
- [ ] Otherwise reports "waiting on wave N", naming the specific blocking
      wave(s)
- [ ] Pure rollup of CAP-05 wave-map data — does not re-query GitHub/board
      with different logic

#### US-10 — Approve a cleanup PR fully informed `(CAP-10)`

**As a** developer, **I want** a plain-language before/after summary of what a
cleanup pass deletes or keeps, **so that** I'm not trusting a black box.

**Acceptance criteria:**

- [ ] Before `purge-initiative-artifacts-app`/`-meta` runs, lists what's about
      to be deleted vs. kept, sourced from the purge skill's own manifest/plan
      — this INIT does not compute the delete list independently
- [ ] After the purge skill runs, lists what was actually deleted/kept, for
      before/after comparison
- [ ] Once the closure PR exists, reuses CAP-01 to confirm
      `initiative-closure-signoff-app`/`-meta` is genuinely satisfied on the
      current head before/after merge — no separate approval-checking logic
      built for closure

### Non-Goals

| Non-goal | Why |
|----------|-----|
| Phase 3 — Specification engineering review | Already works correctly, entirely human, in Cursor/GitHub. Nothing to build. |
| Phase 4 — Specification approval and merge | Already works correctly, entirely in GitHub. Gateflow must never participate in applying the approval or merging. |
| Phase 8 — Code and feature verification | Already works correctly, entirely human. Nothing to build. |
| Phase 14 — Meta closure | Depends on the product-management side Gateflow doesn't touch today — its own later initiative. |
| The actual UI screens/buttons (`gateflow-ops`) | Follow-on initiative — this INIT builds what those screens will call. |
| Any automatic label/approval/merge action | Hard boundary, unchanged from INIT-GATEFLOW-010 — a human always makes that call. |
| Automatic progression on GitHub events (webhooks) | Day-1 is "developer asks Gateflow to check" — addable later without a rebuild, not on now. |
| `prayog-meta` / product-management process changes | Out of bounds for a gateflow-owned initiative. |
| `launchpad` command-naming cleanup | Already fixed upstream in `prayog-skills` — no separate work needed once remounted. |

### Assumptions

| ID | Assumption | Status | Dependent REQs |
|----|------------|--------|----------------|
| A1 | `delivery-contract.yaml`'s `github.labels` + `review_roles` remain the SSOT for which label(s)/role satisfy each checkpoint; Gateflow must not hardcode label strings per phase | Confirmed — `delivery-contract.yaml` itself declares `github.labels` + `review_roles` as pin content (not computed at runtime), so any consumer — gating or metrics — must read them rather than hardcode label strings | REQ-01, REQ-02, REQ-09, REQ-21, REQ-27 |
| A2 | "Current version of code" = PR head SHA at call time; satisfying evidence is compared against the SHA it was recorded/applied against | Confirmed (D2) | REQ-03, REQ-19 |
| A3 | Gateflow's existing run/timeline store can host a new lightweight check-record type without a parallel store | Confirmed — reuses INIT-GATEFLOW-010 timeline pattern | REQ-06, REQ-19 |
| A4 | prayog-meta's PR/label state is reachable read-only by Gateflow for the Phase 1 meta bridge, via the same `ForgeClient.get_pull_request` GitHub REST edge Gateflow already uses against `gateflow`/`gateflow-ops` | Confirmed by code (`src/infra_services/forge_client.py`) — no new transport needed, only a different `owner`/`repo` argument | REQ-09, REQ-11 |
| A5 | Outline §0 remount completes on all three repos before `/spec-draft`; this PRD's checkpoint-id and label vocabulary (`coding-readiness`, `wave-acceptance`, `spec-lgtm`, `wave-accepted`) reflects the current tip | **Confirmed on all three repos** by direct inspection this session (submodule SHA + vendored `workflow.yaml` + `gateflow` as-built "remount hygiene 2026-08-06") | Document control, all REQs referencing pin node ids |
| A6 | `ForgeClient.get_pull_request` + `GithubPullRequestDocument` are real today but read only `title`/`body`/`state`/`labels`/`head`/`base` — **no** merged/mergeable state, no PR reviews, no check-runs | Confirmed by code (`src/infra_services/forge_client.py`, `src/models/meta_pr_models.py`) — CAP-01 requires **new** `ForgeClient` read methods (list reviews, list check-runs) and an extended PR document, not just reuse as-is | REQ-01, REQ-02, REQ-21 |
| A7 | `MetaPrIntakeService.accept()` (`src/business_services/meta_pr_intake.py`) is the closest existing precedent for a "reconcile" check — it fetches a PR, reads `head.sha`, and derives/enforces an initiative id — but it does **not** check `impact-map-lgtm` or any review-approval state today | Confirmed by code — proves the D2/D3 gap is real even in Gateflow's one existing accept-gate, not just a hypothetical | CAP-01 design basis |

### Error table (product-normative)

| Situation | Response | Side effects |
|-----------|----------|---------------|
| Checkpoint status-check for unresolvable PR/checkpoint id | **404** | 0 GitHub calls beyond resolution attempt |
| Checkpoint requested via initiative+wave, but no run/PR exists yet for that pairing | **404** with reason `no run found for this wave` | Distinct from a malformed identifier (REQ-08) |
| Checkpoint evidence dated before a later commit on the same PR | `NOT SATISFIED`, reason `stale — new commits since approval` | Recorded as a check record (REQ-06); never reported as `pass` |
| Request includes a mutating parameter (e.g. attempted apply/approve/merge flag) on any read-only route | **400** | 0 GitHub write attempted — structural guard (G1) |
| prayog-meta unreachable during initiative read-out | **200** with meta-derived field marked `unavailable` | Gateflow-owned fields still returned (REQ-11) |
| Closure preview requested before purge skill has run | **200**, "not yet run" plan preview from manifest | No delete attempted |
| GitHub API unreachable/rate-limited during a live status-check | Check fails closed: "could not verify — GitHub unreachable" | No cached/stale verdict ever presented as current (ties to D2) |

### Open questions

| ID | Open question | Status |
|----|----------------|--------|
| OQ-01 | Exact problem+json / OpenAPI response field names for the routes in Appendix A | Open — deferred to gateflow OpenAPI pass |
| OQ-02 | Whether the prayog-meta read (REQ-09/REQ-11) reuses the same GitHub App/ForgeClient credentials Gateflow uses against `gateflow`/`gateflow-ops`, or a separate read-only scope | Open — engineering routing |

---

## 3. Functional requirements

### Capabilities

| ID | Capability | Covers |
|----|------------|--------|
| CAP-01 | Checkpoint status-check (core, reusable) | REQ-01–REQ-05 |
| CAP-02 | Check persistence / history | REQ-06–REQ-08 |
| CAP-03 | Initiative list & detail | REQ-09–REQ-11 |
| CAP-04 | Spec lane read-out | REQ-12–REQ-13 |
| CAP-05 | Wave map read-out | REQ-14–REQ-15 |
| CAP-06 | Wave implementation progress read-out | REQ-16–REQ-17 |
| CAP-07 | Closeout read-out + drift safeguard | REQ-18–REQ-20 |
| CAP-08 | Merge confirm + next-wave nudge | REQ-21–REQ-22 |
| CAP-09 | Completion eligibility read-out | REQ-23–REQ-24 |
| CAP-10 | Closure preview + reuse | REQ-25–REQ-27 |
| — | Global read-only guard | REQ-28 |

### Requirements

| ID | Requirement | Outline / gap | Condition | Observable result | Evidence |
|----|-------------|----------------|-----------|--------------------|----------|
| REQ-01 | Checkpoint status-check accepts a checkpoint node id + PR-resolving identifiers; is read-only | D1, Phase 5/9/11/13 | Any call | Response returned; 0 GitHub writes | unit + verify |
| REQ-02 | Evidence = pinned `delivery-contract.yaml` label set (present required, absent blocking) + `review_roles` review state + required checks, evaluated live at current head SHA | D4, A1, G2 | Live call | Verdict matches pinned label/role vocabulary, not hardcoded strings | unit |
| REQ-03 | Response includes `checked_sha` + `checked_at`; evidence predating a later commit → `NOT SATISFIED` (`stale`) | D2, A2, G3 | New commit after approval | Never reports stale evidence as `pass` | unit + verify |
| REQ-04 | Non-pass responses list each missing item by name | D3 | Any failing check | Itemized list, not boolean | unit |
| REQ-05 | Status-check never calls `apply_labels`, review-create/update, merge, or `update_board_status` | D1 | Any call | 0 write calls (code guard) | code guard + verify |
| REQ-06 | Every CAP-01 call persists a check record (checkpoint id, PR ref, `checked_sha`, `checked_at`, verdict, missing items) to existing run/timeline store | G4 (Draft PRD decision) | Any CAP-01 call | Record retrievable via history read-out | unit |
| REQ-07 | History read-out returns prior persisted records, explicitly labeled historical | G5 | History call | Response marks records as historical, distinct from live verdict | unit |
| REQ-08 | General checkpoint read-out composes CAP-01 with initiative/wave/run identifiers Gateflow already resolves | Phase 5/9/11/13 | Phase-scoped call | Caller supplies initiative+wave, not raw org/repo/PR | unit + verify |
| REQ-09 | Initiative list/detail returns id, description, PRD approval state (via CAP-01 against `prd-impact-acceptance`), repos, stage, in-flight run link | Phase 1 | List/detail call | All fields present or explicitly `unavailable` | unit + verify |
| REQ-10 | Initiative read-out composed only from Gateflow-owned data + one read-only meta PR/label read | D4 | Any call | No new source of truth invented | inspection |
| REQ-11 | prayog-meta unreachable → gateflow-owned fields still return; meta field marked `unavailable` | Phase 1 resilience | Meta unreachable | Partial success, not full failure | unit + verify |
| REQ-12 | Spec-lane read-out returns Draft Spec PR link, generated artifacts, findings/open questions, exact next step | Phase 2 | Spec walk in progress or done | Fields resolved from pin + run state | unit + verify |
| REQ-13 | If spec walk has not reached `spec-pr-action`, read-out says so plainly | Phase 2 edge case | No PR yet | No broken link | unit |
| REQ-14 | Wave map returns per-wave status ∈ done/ready-to-start/blocked/active, with reason when blocked | Phase 6 | Any initiative with waves | Status + reason (if blocked) present | unit + verify |
| REQ-15 | Wave status derived only from existing board/run data | D4 | Any call | No new wave-state store | inspection |
| REQ-16 | In-progress wave read-out returns task-by-task progress + Draft PR link on `wave-pr-action` success | Phase 7 | Wave in progress | Per-task status, not single spinner | unit + verify |
| REQ-17 | Task failure/stop surfaces which task and why | Phase 7 edge case | Task fails / needs-input | Named task + reason | unit |
| REQ-18 | Closeout read-out lists what closeout added (lessons, learning-store records) | Phase 10 | Closeout complete | Itemized additions | unit + verify |
| REQ-19 | Drift safeguard compares wave-acceptance-time head SHA (persisted, REQ-06) vs. closeout-time head SHA; flags if different; if no persisted `wave-acceptance` check record exists for this wave, reports `unknown — no baseline recorded` instead of silently skipping or falsely reporting no drift | Phase 10, A2 | SHA differs, or no baseline recorded | Explicit "code changed after acceptance" flag, or explicit "no baseline" flag | unit + verify |
| REQ-20 | Drift safeguard is advisory only; does not block/fail closeout mechanics | D1, D6 | Any closeout | Closeout proceeds unchanged regardless of flag | unit |
| REQ-21 | Merge confirm reuses CAP-01 against `wave-signoff` for merged state + merge commit SHA, or missing items | Phase 11 | Merge confirm call | Merged/not-merged + evidence | unit + verify |
| REQ-22 | After confirmed merge, if next wave now unblocked, surface "wave W`n+1` is now unblocked" | Phase 11 | Predecessor now Done | Nudge present in response | unit + verify |
| REQ-23 | Completion eligibility reports "ready to close" iff every `wave_ticket_ids` entry is board Done; else "waiting on wave N"; if `wave_ticket_ids` is empty or unresolvable, reports a distinct "no waves found" state — never "ready to close" | Phase 12, matches GATEFLOW-010 REQ-13 semantics | Any initiative | Correct rollup state | unit + verify |
| REQ-24 | Completion eligibility is a pure rollup of CAP-05 data — no second GitHub/board query with different logic | Phase 12 | Any call | Single source of truth reused | inspection |
| REQ-25 | Closure preview (pre-purge) lists what will be deleted/kept, sourced from purge skill's own manifest | Phase 13 | Before purge runs | List matches purge skill's plan | unit + verify |
| REQ-26 | Closure preview (post-purge) lists what was actually deleted/kept | Phase 13 | After purge runs | Before/after comparison available | unit + verify |
| REQ-27 | Once closure PR exists, reuses CAP-01 to confirm `initiative-closure-signoff-app`/`-meta` before/after merge | Phase 13 | Closure PR exists | Same evidence rules as CAP-01, no new logic | unit + verify |
| REQ-28 | No capability in this INIT (CAP-01–CAP-10) calls `apply_labels`, review create/update, merge, or `update_board_status` | D1, D4, D6 | Any route in Appendix A | Route inventory audit shows 0 mutating routes | code guard + verify |

### Checkpoint evidence mapping (product-normative)

CAP-01's evidence resolution is enumerable directly from the pinned
`delivery-contract.yaml`. Three of the six `review_roles` checkpoints have
**no associated label at all** — their evidence is review/merge state only,
not a label check:

| Checkpoint (pin node id) | Required label | Blocking labels | `review_roles` entry | Evidence class |
|---|---|---|---|---|
| `prd-impact-acceptance` | `impact-map-lgtm` | `impact-map-blocked`, `impact-map-revised`, `impact-map-stale` | `engineering-gate` (`meta-pm`) | Label + review |
| `coding-readiness` | `spec-lgtm` | `spec-blocked`, `spec-revised`, `spec-stale` | `engineering-gate` (`app`) | Label + review |
| `wave-acceptance` | `wave-accepted` | *(none declared)* | `engineering-gate` (`app`) | Label + review |
| `wave-signoff` | *(none — no label declared)* | — | `engineering-gate` (`app`) | Review/merge state only |
| `initiative-closure-signoff-app` | *(none — no label declared)* | — | `engineering-gate` (`app`) | Review/merge state only |
| `initiative-closure-signoff-meta` | *(none — no label declared)* | — | `engineering-gate` (`meta-pm`) | Review/merge state only |

Source: `prayog-skills/delivery-contract.yaml` (`github.labels`, `review_roles`),
verified against tip `ebaa912b73fcb10392e2046101993a1f8f0af4a2` this session.

**Implementation notes (non-normative):** Gateflow may implement checkpoint
evidence resolution via a `CheckpointEvidenceResolver` reading
`delivery-contract.yaml` at the same tip used for `PolicyEngine`, and persist
check records via the existing `RunStore`/timeline mechanism used by
INIT-GATEFLOW-010. Module names are design detail, not product vocabulary.
See §4.

---

## 4. Technical Specifications

### Architecture Overview *(design — not product vocabulary)*

```text
Read-only API surface (GET only — see Appendix A; programme token auth)
  ├─ /checkpoints/status, /checkpoints/history         (CAP-01, CAP-02)
  ├─ /initiatives, /initiatives/{id}                   (CAP-03)
  ├─ /initiatives/{id}/spec                            (CAP-04)
  ├─ /initiatives/{id}/waves                           (CAP-05)
  ├─ /initiatives/{id}/waves/{w}/implementation        (CAP-06)
  ├─ /initiatives/{id}/waves/{w}/closeout              (CAP-07)
  ├─ /initiatives/{id}/waves/{w}/merge                 (CAP-08)
  ├─ /initiatives/{id}/completion                      (CAP-09)
  └─ /initiatives/{id}/closure                         (CAP-10)

CheckpointEvidenceResolver ← delivery-contract.yaml (github.labels, review_roles)
                            + live GitHub read (labels, reviews, check-runs, head SHA)
CheckRecordStore ← existing RunStore/timeline (new record type, read-only API surface)
BoardService (read) ← ticket status for wave map / completion rollup
RunOrchestrator / RunStore (read) ← task timeline for spec/implementation/closeout read-outs
Meta bridge (read-only GitHub read against prayog-meta repo) ← PRD/impact-map evidence for Phase 1
```

### Integration Points

| System | Use |
|--------|-----|
| GitHub via ForgeClient (read scopes only for these routes) | PR labels, reviews, check-runs, head SHA, merge state |
| PostgreSQL RunStore | Existing runs/timeline + new check-record type (CAP-02) |
| Remounted `prayog-skills/delivery-contract.yaml` | Label + `review_roles` SSOT for checkpoint evidence — never hardcoded |
| prayog-meta repo (read-only) | PRD/impact-map approval evidence for Phase 1 initiative list |
| Programme token | Auth for all new routes (existing pattern) |

### Security & Privacy

- Programme service token on all new routes (existing pattern).
- Every new route is GET-only; no write-scope GitHub credential is required
  for this INIT's code paths (structural property, not just convention).
- Approval labels (`*-lgtm`, `wave-accepted`) are read, never applied, by any
  code path introduced here.
- No secrets in check records, history entries, or verify artifacts.
- Content skills still must not treat local `gh` as automate success —
  ForgeClient only (carried from INIT-GATEFLOW-009/010).

### AI / agent evaluation

Not a new model product — no LLM in the request path for any capability
above. Quality bar = contract/evidence-resolution correctness (unit tests
against fixture PRs with fresh vs. stale label/review/check states) plus live
verify scripts proving at least one real reconcile per checkpoint type,
matching the INIT-GATEFLOW-009/010 evaluation pattern (verify scripts and
units, not rubrics).

### Code-grounded implementation notes (non-normative, verified against `gateflow` @ `0cf6c20e`)

These are **reuse/extend** points found by direct code inspection, not
speculation — they change *how much is new* per capability, not the product
requirements above:

| Existing code | What it already does | What CAP-01–CAP-10 still needs to add |
|---|---|---|
| `ForgeClient.get_pull_request` (`src/infra_services/forge_client.py`) | Fetches one PR into `GithubPullRequestDocument` (`title`, `body`, `state`, `labels`, `head.sha`, `base.sha`) | Extend the document with merged/mergeable state; add `list_reviews` and `list_check_runs` read methods — **none exist today** (A6) |
| `MetaPrIntakeService.accept()` (`src/business_services/meta_pr_intake.py`) | The one existing "reconcile"-shaped check: parse PR URL → fetch via `ForgeClient` → read `head.sha` → derive/enforce initiative id | Does **not** check `impact-map-lgtm` or review-approval state (A7) — CAP-01's `CheckpointEvidenceResolver` should **generalize this service's shape** (parse ref → fetch → evaluate against contract), not sit beside it as a parallel path |
| `ForgeClient.apply_pull_request_labels` / `_assert_projection_labels` | Already hard-forbids writing any `*-lgtm` label at the client level | Confirms G1/REQ-05/REQ-28 are enforceable the same way, at the same layer — no new guard mechanism needed |
| `BoardService.list_tickets` (`src/business_services/board_service.py`) | Reads tickets filtered by `initiative_id` / `ticket_type` / `state` (open\|closed\|all); each returned ticket also carries its board `column` | Directly backs CAP-05 (wave map) and CAP-09 (completion rollup); no new board read path needed |
| `RunRepository` / `StageRepository` / `RunEventRepository` (`src/database/postgres/repository/run_store_repository.py`); `RunModel` already carries `initiative_id`, `wave_id`, `pr_number`, `workflow_node` | Existing run/timeline persistence Gateflow already owns | CAP-02's check-record can follow the same repository pattern (new sibling table/repository, not a new store); CAP-03/CAP-05/CAP-06 read `RunModel`/`StageRepository` directly — confirms D4 with code, not just policy |
| `src/api/v1/{board,metrics,runs,waves}_routes.py` | Established `/api/v1/` route module convention | This INIT adds new sibling modules (`checkpoints_routes.py`, `initiatives_routes.py`) — no `initiatives_routes.py` exists today |

---

## 5. Risks & Roadmap

### Delivery waves

| Wave | Intent | Phase(s) unlocked | Exit REQs |
|------|--------|--------------------|-----------|
| **W0** | Checkpoint status-check against a single PR (labels, checks, reviews) — read-only, no persistence yet | (foundation) | REQ-01, REQ-02, REQ-04, REQ-05 |
| **W1** | Add "last checked, what version" persistence + general checkpoint read-out composing it with run/timeline data | 5, 9 | REQ-03, REQ-06–REQ-08 |
| **W2** | Initiative list/detail using data Gateflow already owns (runs, board tickets) | 1 (partial) | REQ-09 (partial), REQ-10 |
| **W3** | Extend initiative read-out with meta PR/approval state (read-only from prayog-meta) | 1 (complete) | REQ-09 (complete), REQ-11 |
| **W4** | Wave map read-out (done/ready/blocked/active) | 6, 11 (nudge) | REQ-14, REQ-15 |
| **W5** *(new)* | Spec lane generation read-out | 2 | REQ-12, REQ-13 |
| **W6** *(new)* | Wave implementation task-by-task progress read-out | 7 | REQ-16, REQ-17 |
| **W7** *(new)* | Closeout read-out + post-acceptance drift safeguard — depends on W1's persisted check record | 10 | REQ-18–REQ-20 |
| **W8** *(new)* | Merge confirm + next-wave nudge (reuses W1) + completion eligibility rollup (reuses W4) | 11, 12 | REQ-21–REQ-24 |
| **W9** *(new)* | Closure preview (before/after) + reuse of merge confirm for closure sign-off | 13 | REQ-25–REQ-27 |

**Why W5–W9 exist:** the outline's own §8 wave plan (W0–W4) only unlocks
Phases 1, 5, 6, 9, 11 — but §5 Scope-in commits to Phases 2, 7, 10, 12, 13 as
well. Rather than trim scope or silently fold ungoverned work into existing
waves, this Draft PRD adds explicit waves so every in-scope phase has a named
exit (G6).

**Note on W1 / Phase 11:** the outline's §8 attributed Phase 11's unlock
entirely to W1. This Draft PRD deliberately splits that unlock across **W1**
(the general checkpoint read-out), **W4** (the "start next wave" nudge), and
**W8** (merge confirm) as part of refining the outline's proposed waves — not
a scope reduction of W1.

### Technical risks

| Risk | Mitigation |
|------|------------|
| Status-check looks "stale" if GitHub state changes between check and use | Always check live at call time; never cache silently (REQ-02, REQ-03) |
| Persisted check records grow unbounded | Reuse existing RunStore/timeline retention; no new storage tier introduced (A3) |
| Developers distrust automated cleanup (Phase 13) without transparency | Explicit before/after summary is scope, not a nice-to-have (REQ-25, REQ-26) |
| Scope creep into meta closure (Phase 14) or UI screens | Explicit non-goal; separate initiatives |
| Read-only meta PR bridge (Phase 1) misread as PM-process ownership | D5 + non-goals make the boundary explicit |
| Outline §0 remount skipped/partial | **Resolved** — confirmed complete on all three repos this session (see As-built baseline); no longer an open risk, listed for traceability |
| New W5–W9 waves add scope beyond outline's original W0–W4 estimate | Each new wave reuses W0/W1/W4 machinery (CAP-01/02/05) — no new checker/store invented (G6) |
| `ForgeClient` review/check-run read methods don't exist yet (`A6`) — treating W0 as pure reuse of `get_pull_request` would underestimate it | Scope W0 to explicitly include the `ForgeClient` extension (`list_reviews`, `list_check_runs`, extended PR document); do not treat it as pure reuse |

### Phased rollout

- **MVP (W0–W4):** Reusable status-check + persistence + initiative/wave
  visibility — closes the outline's original wave plan.
- **This Draft PRD's addition (W5–W9):** Full outline §5 phase coverage
  (spec/implementation/closeout/completion/closure read-outs).
- **Later:** `gateflow-ops` screens consuming Appendix A; Phase 14 meta
  closure (separate initiative, D5); webhook-driven auto-progression (deferred
  by design, not by omission).

---

## 6. Locked decisions reference

### Carried forward from outline (D1–D6)

| ID | Decision |
|----|----------|
| D1 | Status-check is read-only; Gateflow may report, never label/approve/merge on anyone's behalf |
| D2 | Every status check records what version it checked and when |
| D3 | Status check returns specific missing items, not just pass/fail |
| D4 | Visibility is a read-only reflection of state that already exists elsewhere; no new sources of truth |
| D5 | PM/requirements side (PRD/closeout process) is out of scope — different system, likely its own initiative |
| D6 | No screens; this INIT builds what a screen calls |

### Added in this Draft PRD (G1–G7)

| ID | Decision |
|----|----------|
| G1 | All new routes are **GET-only** — structural enforcement of D1, not just a coding convention |
| G2 | Checkpoint evidence is pinned-contract-driven (`delivery-contract.yaml` `github.labels` + `review_roles`) — never hardcoded per-phase label strings |
| G3 | "Current version" = PR head SHA at check time; evidence dated before a later commit on the same PR = `NOT SATISFIED`, reason `stale` |
| G4 | Every checkpoint status-check call persists a check record to Gateflow's existing run/timeline store (checkpoint id, PR ref, `checked_sha`, `checked_at`, verdict, missing items) |
| G5 | Persisted/historical check records are for audit/timeline display only — never substituted for a live check when a caller needs a current decision |
| G6 | Waves **W5–W9** added in this Draft PRD (not in outline §8) to close the Phase 2/7/10/12/13 coverage gap the outline left open — explicit PM judgement call, not an outline-locked decision |
| G7 | API surface routes are named concretely in this Draft PRD (Appendix A) rather than deferred — explicit PM decision; exact response-schema field names remain `OQ-01` |

---

## 7. Next steps

1. ~~Confirm outline §0 remount clean on all three repos~~ — **done**, verified
   this session directly against `gateflow`, `gateflow-ops`, and
   `prayog-meta` checkouts. `/spec-draft` is unblocked.
2. Review this Draft PRD with PE / programme; confirm G1–G7, the W5–W9
   addition, and the code-grounded implementation notes (§4) — in particular
   that CAP-01 extends `ForgeClient`/`MetaPrIntakeService` rather than
   building a parallel path.
3. Impact map → meta Gate 1 → gateflow spec / implement waves per §5 above.
4. Do **not** implement Gateflow code from this PRD alone — follow SDD
   (spec → feasibility → technical review → plan → waves), same as every
   prior INIT.

---

## Appendix A — Target API surface (all GET, programme token; see `G7`)

| Phase | Route | Capability |
|-------|-------|------------|
| 5, 9, 11, 13 (reused) | `GET /api/v1/checkpoints/status` | CAP-01 |
| cross-cutting | `GET /api/v1/checkpoints/history` | CAP-02 |
| 1 | `GET /api/v1/initiatives` | CAP-03 |
| 1 | `GET /api/v1/initiatives/{initiative_id}` | CAP-03 |
| 2 | `GET /api/v1/initiatives/{initiative_id}/spec` | CAP-04 |
| 6 | `GET /api/v1/initiatives/{initiative_id}/waves` | CAP-05 |
| 7 | `GET /api/v1/initiatives/{initiative_id}/waves/{wave_id}/implementation` | CAP-06 |
| 10 | `GET /api/v1/initiatives/{initiative_id}/waves/{wave_id}/closeout` | CAP-07 |
| 11 | `GET /api/v1/initiatives/{initiative_id}/waves/{wave_id}/merge` | CAP-08 |
| 12 | `GET /api/v1/initiatives/{initiative_id}/completion` | CAP-09 |
| 13 | `GET /api/v1/initiatives/{initiative_id}/closure` | CAP-10 |

Exact response schema / problem+json field names → **OQ-01** (deferred to
gateflow OpenAPI pass).

---

## References

- Outline: [INIT-GATEFLOW-011-outline](./INIT-GATEFLOW-011-outline.md)
- Vision: [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md)
- Delivery contract: `sdd-delivery/v2` (`prayog-skills/delivery-contract.yaml`, `workflow.yaml`)
- Predecessor: [INIT-GATEFLOW-010](./INIT-GATEFLOW-010.md) (eng-lane pin tip executor parity — this INIT's visibility layer sits on top of that executor)
- Prove-out precedent: [INIT-GATEFLOW-009](./INIT-GATEFLOW-009.md) (Draft PR tip deliverable, authorize API pattern)
- Gateflow as-built: `drivestream-lab/gateflow` → `docs/specification/as-built/implementation-status.md`
