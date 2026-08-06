# INIT-GATEFLOW-009 — Both-lane delivery factory prove-out

**Status:** draft PRD · **Author:** programme PM · **Date:** 2026-08-03  
**Outline:** [INIT-GATEFLOW-009-outline](./INIT-GATEFLOW-009-outline.md)  
**Vision:** [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md)  
**Component:** GATEFLOW · **Type:** prove-out / confidence (reuse existing factory — do not rebuild)

> **Draft PRD.** This initiative does **not** invent a new Gateflow. It proves
> the factory we already have for the **spec** lane the same way we already
> trust the **coding** lane—especially that a **Draft Spec PR** carries real
> committed artifacts—and proves the **API authorize** path for sensitive
> forge steps. This INIT **adheres to `sdd-delivery/v2`** (GitHub PR + labels +
> Q&A + Approve for Gate 1/2). Engineering detail routes to impact map and
> gateflow prove-out work. Skills pin stays on the **`v0.5.0-rc.2` family**
> (consume only).

---

## Document control

| Field | Value |
|-------|-------|
| Initiative ID | INIT-GATEFLOW-009 |
| Programme | prayog |
| Primary repo | drivestream-lab/gateflow |
| Supporting repos | prayog-meta (this PRD); prayog-skills (pin consume only); gateflow-ops (**out of scope**) |
| Skills pin | **`v0.5.0-rc.2` family** — remount/consume tip only; no new RC / no pin redesign |
| Delivery contract | **`sdd-delivery/v2`** — adhere; do not invent parallel approval paths |
| Impact map scope | **gateflow only** `(Source: User-confirmed)` |
| Gate 1 posture | **`sdd-delivery/v2`:** meta Draft PR — `impact-map-pending` → PR Q&A → `impact-map-lgtm` + Approve on exact head (`prd-impact-acceptance`). No PE-waive / alternate ceremony. |
| Target users | PE (starts waves / calls authorize API), human reviewer (confirms PR deliverables), programme sponsor (features ready before ops portal) |

---

## 1. Executive Summary

### Problem Statement

Coding waves under Gateflow are already trusted `(Source: User-confirmed)`:
work runs, a draft PR appears, humans review, wrap-up stores learning. Spec
waves can be **started**, but the programme has not yet **proven** that
Gateflow leaves a **Draft Spec PR whose tip actually contains the files each
automated step produced**—the same deliverable a human developer gets when they
commit and update the PR by hand. Sensitive steps that must wait for a human OK
are gated by an **API authorize** call in Gateflow (there is no person typing
inside the agent). That path is built in the lab but not yet proven live.
Without those proofs, we should not start an ops portal on top of unfinished
trust.

### Proposed Solution

Prove and record—not rebuild—that:

1. A **spec** wave under Gateflow produces a **Draft Spec PR** with artifacts
   committed on the tip through the allowed automated steps.
2. A **human reviewer** confirms those deliverables on the PR, and **wrap-up
   is live-proven** on that PR as part of initiative success (after tip
   confirmation).
3. When the process requires a human OK before a sensitive forge action,
   Gateflow **stops** and only continues after an **authorize API** call (not
   after someone typing in Cursor).
4. Programme records list which **features** are proven ready, and which are
   deferred (including the ops portal as next)—without relying on internal
   horizon nicknames as the success story.
5. Gateflow pull requests run **real basic automated checks**, not a
   placeholder.

### Success Criteria

| KPI | Target | Measurement |
|-----|--------|-------------|
| **Draft Spec PR deliverable** | ≥ 1 live prove-out opens a Draft Spec PR whose tip includes committed artifacts from the automated steps run | Live verify record + PR tip inspection by human reviewer |
| **Honest stop** | That run stops at the first place a human must decide or the next step is still manual—zero silent skip of human ownership | Run timeline + reviewer confirmation |
| **Spec wrap-up** | ≥ 1 live prove-out of wrap-up on that Draft Spec PR through wave sign-off; prior “spec wrap-up skipped” note lifted | Live verify + wave sign-off record |
| **Authorize API path** | ≥ 1 live prove-out: run stops for a sensitive forge step → authorize API succeeds → forge side effect appears (e.g. board tickets) | Live verify record `(Source: User-confirmed — live required; no paper waiver as exit)` |
| **Feature readiness record** | Written list of **features proven** vs **features deferred** (ops portal named as next); as-built updated to match | Meta/gateflow freeze package |
| **Basic automated PR checks** | Gateflow PRs fail when basic automated checks fail (e.g. branch naming / PR hygiene; lint/unit as examples) | CI / check workflow replacing placeholder |

Numeric cycle-time targets are **not** exit gates for this prove-out.

### Product id map

| CAP | REQ | User story |
|-----|-----|------------|
| CAP-01 Spec Draft Spec PR tip deliverable | REQ-01 | US-1 |
| CAP-02 Spec wrap-up prove-out | REQ-02 | US-2 |
| CAP-03 Authorize API path | REQ-03 | US-3 |
| CAP-04 Feature readiness freeze | REQ-04 | US-4 |
| CAP-05 Basic automated PR checks | REQ-05 | US-5 |

---

## 2. User Experience & Functionality

### User Personas

| Persona | Role | Primary need |
|---------|------|--------------|
| **PE** | Starts waves; calls Gateflow APIs (start, authorize, wrap-up) | Trust that API-driven runs leave real PR tips and honor stops |
| **Human reviewer** | Reviews Draft Spec / wave PRs | Sees committed artifacts on the PR tip—same as after a careful human developer session |
| **Programme sponsor** | Funds next investment | Clear **feature** readiness before ops portal spend |

### How the same skills show up in two worlds

| Situation | Who drives | How work lands on GitHub |
|-----------|------------|---------------------------|
| **Human developer** in Cursor / Claude | Person in the IDE | Creates files → uses commit / update-PR skills (or Gateflow-equivalent forge) when they choose |
| **Gateflow orchestrator** | APIs + worker | Creates files in the workspace → **Gateflow must commit** those files to the run head and open/update the Draft PR so the tip matches what the step produced |

This initiative proves the **second** row for the **spec** lane, and proves the
**authorize API** for sensitive steps. It does not redesign how humans work in
the IDE.

### User Stories & Acceptance Criteria

#### US-1 / REQ-01 — Spec wave leaves a real Draft Spec PR `(CAP-01)`

**As a** PE, **I want** to start a spec wave through Gateflow **so that** a
Draft Spec PR appears with the work of each automated step **committed on the
PR tip**—not only sitting in a local folder.

**Acceptance criteria:**

- [ ] Spec wave start is accepted when programme PR + meta folder preconditions are met
- [ ] After each automated content step, Gateflow commits produced artifacts to the run head **per Gateflow process** so the Draft Spec PR tip contains those artifacts (ordering is Gateflow-owned—not prescribed in this PRD)
- [ ] Draft Spec PR exists and is findable from the run record
- [ ] When a later automated step (e.g. initiative feasibility) creates **new** files, Gateflow commits those onto the **same** PR tip (human developer equivalent: commit + update PR)
- [ ] Run stops at the first honest human/manual stop; does not invent merge or gate approvals
- [ ] Human reviewer can confirm on the PR: expected artifacts are present in the tip `(Source: User-confirmed)`

#### US-2 / REQ-02 — Human reviewer confirms the deliverable; wrap-up is proven `(CAP-01, CAP-02)`

**As a** human reviewer, **I want** the Draft Spec PR tip to contain the
committed outputs of the automated walk **so that** I review the same kind of
PR a careful human developer would open—not an empty shell—and the programme
**proves wrap-up** on that PR.

**Acceptance criteria:**

- [ ] Prove-out evidence includes PR URL and a short reviewer confirmation that tip contents match the steps run
- [ ] Empty Draft Spec PR (no committed step outputs) does **not** count as success
- [ ] After tip confirmation, wrap-up is **live-proven** on that PR through wave sign-off and lifts any prior “spec wrap-up skipped” programme note (**required** for initiative success)

#### US-3 / REQ-03 — Sensitive forge waits for an API authorize (not IDE typing) `(CAP-03)`

**As a** PE, **I want** Gateflow to **stop** when the process requires a human
OK before a sensitive action (for example creating board tickets), and only
continue after I call the **authorize API**, **so that** the orchestrator
cannot invent board/merge actions while nobody is in Cursor.

**Acceptance criteria:**

- [ ] On a sensitive step marked as needing human OK first, the run **stops** and does not perform the forge side effect yet
- [ ] Calling the authorize API with approval performs the side effect (e.g. board tickets appear)
- [ ] Calling authorize without approval (or wrong state) does not perform the side effect
- [ ] Success for this initiative **requires** a recorded **live** run of stop → authorize API → side effect `(Source: User-confirmed)` — a paper-only exception is **not** an exit path

#### US-4 / REQ-04 — Feature readiness before ops portal `(CAP-04)`

**As a** programme sponsor, **I want** a written list of which **features** are
proven ready and which are deferred **so that** we fund an ops portal against
clear capability—not against an internal horizon nickname.

**Acceptance criteria:**

- [ ] Freeze package lists features such as: coding-lane prove-out (already baseline), spec Draft Spec PR deliverable, authorize API path, wrap-up on spec PR, basic automated PR checks
- [ ] Deferred features named explicitly (e.g. ops portal UI, second coding agents, Slack/Teams, programme-wide self-dogfood)
- [ ] Language prefers **feature names** over “Horizon 1.5 closed” as the headline `(Source: User-confirmed)`
- [ ] Vision / planning note updated in the **same** freeze package as the feature readiness record `(Source: User-confirmed — lean same package)`

#### US-5 / REQ-05 — Basic safety on every Gateflow change `(CAP-05)`

**As a** tech lead, **I want** Gateflow pull requests to run real basic
automated checks **so that** prove-out gains are not silently broken.

**Acceptance criteria:**

- [ ] Placeholder CI replaced with **basic automated PR checks** (examples: branch naming / PR hygiene; lint/unit may appear as Implementation Note examples)
- [ ] For **orchestrated** Gateflow PRs, fail-closed behaviour matches **Gateflow repo/CI** (this PRD does not invent branch-protection settings)
- [ ] Human-developer PRs follow human PR workflow inputs/process—not a second invented rule set in this INIT

### Assumptions

| ID | Assumption | Status | Depends |
|----|------------|--------|---------|
| A-01 | Intended skills tip (`v0.5.0-rc.2` family) is remounted/consumed for prove-outs | Confirmed programme stance | W0, REQ-01 |
| A-02 | Meta PR + meta folder fixtures are available before W1 | Programme process | W0, REQ-01 |
| A-03 | PE can call authorize API with programme service token | Existing lab path | REQ-03 |
| A-04 | Production forge uses GitHub App / ForgeClient (not `gh` as cloud success path) | Existing | REQ-01, REQ-03 |
| A-05 | Gate 1 follows **`sdd-delivery/v2`** meta PR path (`impact-map-*` + Approve) | Confirmed `(Source: User-confirmed)` | Document control, impact map |

### Error Handling / Alternative paths

| Scenario | Expected behaviour | REQ |
|----------|--------------------|-----|
| Spec start rejected (missing programme PR / meta folder) | Start fails closed; no empty Draft Spec PR counted as success | REQ-01 |
| Commit / forge failure mid-walk | Run records failure; tip without committed step outputs does **not** count as success | REQ-01 |
| Empty Draft Spec PR opened | Does **not** satisfy REQ-01 / Success Criteria | REQ-01 |
| Authorize deny / wrong state | No forge side effect | REQ-03 |
| Authorize API unavailable / timeout | Side effect not performed; prove-out recorded as fail until live path works (no paper waiver exit) | REQ-03 |

### Non-Goals

| We are NOT doing | Why |
|------------------|-----|
| Rebuilding coding start, walker, wrap-up, authorize, learning, or metrics | Already built—**reuse** |
| Ops portal / dashboard UI | Next initiative after feature readiness is recorded |
| Second coding agent (OpenCode / Claude) live | Not required for this prove-out |
| Slack / Teams alerts | GitHub comments enough for now |
| Exhaustive “try every skill” bake-offs | One honest Draft Spec PR deliverable path, not a matrix |
| Auto-merge or “authorize then silently resume the old run into the next skill” | Humans keep merge / re-start ownership; wrap-up stays a **new** start when used |
| Changing skills pin or cutting a new skills RC | Programme stays on rc.2 family |
| Treating IDE “please confirm” prompts as the Gateflow human gate | Gateflow’s gate is **stop + authorize API** |
| Inventing approval paths outside **`sdd-delivery/v2`** (PE-waive ceremony, parallel Gate 1) | Approvals stay on GitHub PR + labels + Approve |

---

## 3. AI System Requirements

### Tool / runtime expectations (reuse)

| Capability | Expectation for this INIT |
|------------|---------------------------|
| Coding agent | Existing Cursor path — no new agent |
| Process pin | Existing skills tip (rc.2 family) — consume only |
| Spec start + meta folder bind | Existing — prove the walk and commits |
| Commit to run head + open Draft Spec PR | Existing forge behaviour — prove artifacts land on tip **per Gateflow process** |
| Authorize API | Existing — **live** prove-out required |
| Wrap-up start | Existing shared path — **required** live prove-out after reviewer confirms PR tip |
| Delivery gates | **`sdd-delivery/v2`** meta/app PR labels + Approve |

### Evaluation strategy (how we know quality)

| Check | How we evaluate |
|-------|-----------------|
| Draft Spec PR tip completeness | Human reviewer confirms files from automated steps are on the tip |
| Stop honesty | Timeline shows stop at human/manual boundary; no merge/gate invent |
| Spec wrap-up | Live prove-out through wave sign-off on that PR |
| Authorize API | Live scripted prove-out: stop → API approve → side effect visible |
| No rebuild drift | Change set is prove-out scripts, basic automated PR checks, and records—not parallel product APIs |

---

## 4. Technical Specifications

> Keep light: this INIT is prove-out. Detail belongs in impact map / gateflow
> verify work. Order below is **non-normative**—paths A and B are independent.

### Architecture overview (reuse)

```text
Path A — Spec tip deliverable + wrap-up (CAP-01, CAP-02)
PE APIs (spec start / wrap-up)
    → Gateflow worker walks pin-allowed steps
    → Content step writes files locally
    → Gateflow commits to run head per Gateflow process
    → Open/update Draft Spec PR when process allows
    → Later content step → commit again onto same tip
    → Stop for human/manual boundary
    → Human reviewer confirms tip
    → Wrap-up start on that PR → wave sign-off (required)

Path B — Authorize API (CAP-03) — independent of Path A order
    → Sensitive forge step: STOP
    → Authorize API approve → forge side effect (e.g. board tickets)
```

### Integration points

| Integration | Role in this INIT |
|-------------|-------------------|
| GitHub (Draft Spec PR, commits, comments, meta PR Gate 1) | Deliverable + **`sdd-delivery/v2`** approval surface |
| Gateflow authorize API | Human OK for sensitive forge—**API**, not IDE typing |
| Meta PR + meta folder | Preconditions for spec start |
| Run timeline / metrics APIs | Evidence for prove-outs |
| CI / basic automated PR checks | Protect proven behaviour (Gateflow-owned for orchestrated PRs) |

### Security & privacy

- No auto-merge; Forge never writes `*-lgtm` approval labels.
- Approvals use PE-set `*-lgtm` + GitHub Approve per **`sdd-delivery/v2`**.
- Authorize API remains authenticated with the programme service token.
- Production forge stays on the existing GitHub App / ForgeClient path (no `gh` as the cloud success path).

---

## 5. Risks & Roadmap

### Phased rollout (product waves)

| Wave | Outcome (feature language) |
|------|----------------------------|
| **W0** | Intended skills tip confirmed; spec prove-out plan + fixtures ready (meta PR, folders, reviewer checklist) |
| **W1** | **Draft Spec PR deliverable** live-proven (commits on tip through automated steps + honest stop); human reviewer confirms |
| **W2** | Wrap-up on that Draft Spec PR **live-proven** (required); any prior “spec wrap-up skipped” note lifted |
| **W3** | **Authorize API** live-proven (stop → API → side effect); **feature readiness** record + planning note in same package; basic automated PR checks on |

### Risks

| Risk | Mitigation |
|------|------------|
| Team rebuilds APIs instead of proving tip deliverables | Non-goals + reuse baseline; review rejects parallel product surfaces |
| Draft Spec PR opens empty | Success criteria require committed artifacts; reviewer confirmation required |
| People confuse IDE confirm with Gateflow authorize | US-3 / REQ-03: API trigger only |
| Meta fixtures missing | W0 checklist before W1 |
| Horizon nickname creeps into sponsor messaging | Feature readiness list is the freeze headline |
| Parallel “ceremony waive” approval invented | Explicit **`sdd-delivery/v2`** adherence |

### Locked decisions `(Source: User-confirmed)`

| Topic | Decision |
|-------|----------|
| Spec wave prove-out | **Required** live; focus on Draft Spec PR + committed artifacts on tip |
| Human reviewer | Confirms PR deliverables as part of success |
| Spec wrap-up | **Required** live prove-out on that Draft Spec PR |
| Gateflow human OK for sensitive forge | **Authorize API** (not typing in agent); **live proof required**; no paper waiver as exit |
| Freeze messaging | Talk about **features** proven/deferred—not “Horizon 1.5 closed” as the headline |
| Planning/vision update | Same freeze package as the feature readiness record |
| Impact map | gateflow only |
| Skills pin | rc.2 family consume only |
| Delivery contract | **`sdd-delivery/v2`** — Gate 1 = meta PR `impact-map-*` + Approve; no PE-waive ceremony |

---

## 6. Baseline we reuse (do not re-buy)

| Feature already in place | Role in this INIT |
|--------------------------|-------------------|
| Coding-lane wave start → draft PR → human stop | Baseline trust — protect, don’t rebuild |
| Spec wave start + meta intake | Start the prove-out — don’t invent a second start |
| Commit-to-branch + automated Draft PR open | Mechanism that must leave artifacts on the tip (Gateflow-owned order) |
| Shared wrap-up start + learning store | **Required** live prove-out after reviewer confirms PR |
| Authorize API + board helpers | Live-prove stop → API → side effect |
| Run timeline / metrics | Evidence |
| Cursor agent path | Unchanged |
| `sdd-delivery/v2` Gate 1/2 | Meta/app PR approval path — adhere |

---

## References

- Outline: [INIT-GATEFLOW-009-outline](./INIT-GATEFLOW-009-outline.md)
- Vision: [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md)
- Delivery contract: `sdd-delivery/v2` (`prayog-skills/delivery-contract.yaml`, `workflow.yaml`)
- Prior control plane: [INIT-GATEFLOW-001](./INIT-GATEFLOW-001.md)
- Skills pin era: [INIT-PRAYOG-SKILLS-002](./INIT-PRAYOG-SKILLS-002.md)
- Gateflow as-built: `drivestream-lab/gateflow` → `docs/specification/as-built/implementation-status.md`
