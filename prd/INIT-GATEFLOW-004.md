# INIT-GATEFLOW-004 — Gateflow Mission Control

**Status:** draft PRD · **Author:** programme PM · **Date:** 2026-07-27  
**Outline:** [INIT-GATEFLOW-004-outline](./INIT-GATEFLOW-004-outline.md) (**synced**)  
**Vision:** [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md) (§9 maturity · §10 H2 · lift-on-metrics)  
**Predecessors:** [INIT-GATEFLOW-001](./INIT-GATEFLOW-001.md), INIT-GATEFLOW-002 (finished/delivered; PRD artifact not in this repo), [INIT-GATEFLOW-003](./INIT-GATEFLOW-003.md) (control plane + live Cursor; **003 W2 engg spec-lane prove-it pairs with this PRD as dogfood**)  
**Component:** GATEFLOW · **Type:** operations / Mission Control

> **Draft PRD** — Discovery decisions locked in outline §11 (2026-07-27),
> including #12 (no partial onboarding) and #13 (collect numbers first; no
> timed cockpit drill). Engineering design (APIs, schemas, UI stack) routes
> to impact map and gateflow-ops / gateflow spec PRs — not here.
>
> **Dogfood role:** This PRD is the **real work** used to prove Gateflow’s
> **engg spec-lane** automation with live agents (003 W2 companion).
> Human checkpoints stay on; we collect firsthand efficacy so we can later
> **lift** controls when metrics justify it (vision §9).

---

## Document control

| Field | Value |
|-------|-------|
| Initiative ID | INIT-GATEFLOW-004 |
| Programme | prayog |
| Primary repo | drivestream-lab/gateflow-ops |
| Supporting repos | drivestream-lab/gateflow (API gaps only — e.g. wave summaries, repo onboarding records); prayog-meta (this PRD + impact map) |
| Out of this INIT | **Launchpad** greenfield / new-repo scaffolding; **prayog-skills** pin / `dispatch` for engg spec-lane (owned by **INIT-GATEFLOW-003 W2**, not 004) |
| Depends on | INIT-GATEFLOW-001–002 **finished / delivered**; INIT-GATEFLOW-003 **control plane + live Cursor available** (run/metrics/wave-start APIs; RunStore cycle-time fields FR-30/REQ-30); **003 W2 engg spec-lane prove-it may still be open** and may use this PRD as dogfood subject |
| Product ids | Canonical `REQ-n`; legacy `FR-n` ≡ `REQ-n` for this programme’s GATEFLOW PRDs (no `CAP-*` in this INIT) |
| Target users | Engineering (operate waves), tech lead (trust & lift decisions), programme sponsor (agent efficacy between human stops) |
| Identity (v0) | **Thin ops-user identity** — signed-in operators only; **no roles / RBAC** in this INIT |

---

## 1. Executive Summary

### Problem Statement

INIT-GATEFLOW-001–002 delivered the **control plane foundation**; INIT-GATEFLOW-003
made **live Cursor** and run/metrics/wave-start APIs available. We can start a
wave via API, honor the pinned workflow, stop at human gates, record cycle-time
metrics, and open a PR thread (**003 W2 engg spec-lane prove-it may still be
open**). What we still cannot do well is **operate and learn** from that
automation as a programme. Operators use APIs, scripts, and
GitHub fragments instead of one place to see what is running, where a wave
stopped, and whether the agent was useful. Repos already on the Prayog paved
road are not first-class fleet members, and without a clear picture of **agent
work vs human stops** we cannot honestly move toward “agents do most of the
work between intentional controls.”

### Proposed Solution

**INIT-GATEFLOW-004** delivers **Gateflow Mission Control (v0)** in
**gateflow-ops**:

1. **Onboard** harness-enabled repos into a fleet after a **strict readiness
   scorecard** passes.
2. **Operate** — fleet home, start waves from the UI, open in-flight or runs
   from the **last 30 days**.
3. **Run cockpit (high bar)** — pinned process map (read-only), per-step
   timeline with duration/outcome/agent info, **full log pane** (narrative +
   deep links), and one-click GitHub PR jump.
4. **Efficacy visibility** — surface unattended progress between human stops,
   retries/findings, and human-wait time for later lift decisions — **without**
   removing checkpoints in this INIT.
5. **Dogfood** — this PRD’s journey through engg **spec-lane** skills is the live
   subject of INIT-GATEFLOW-003 W2 prove-it.

We do **not** invent delivery process in the console, create new repos, or
auto-merge. Greenfield scaffolding stays with **Launchpad**.

### Success Criteria

| KPI | Target | Measurement |
|-----|--------|-------------|
| **Ops identity (thin)** | 100% of Mission Control sessions are attributable to a signed-in ops-user record; **zero** role/permission configuration required | Auth audit + user record on wave-start actions |
| **Scorecard-gated onboarding** | 0 fleet members added without scorecard **pass**; scorecard is **pass/fail only** (no partial); fail **blocks** with operator-readable reasons `(Source: Outline §11 #12)` | Onboarding integration tests + blocked-on-fail audit |
| **Wave operable from console** | Operator can **start** a wave and open run detail from UI without raw API calls as the happy path | End-to-end UI test on ≥ 1 onboarded repo |
| **Cockpit answers the job** | From cockpit alone, operator can identify (a) process position, (b) each step’s timing/outcome/agent, (c) readable log narrative with links, and (d) wave PR — **no timed drill** as exit gate; Mission Control **collects** the numbers so experience can be measured later `(Source: Outline §11 #13)` | Open ≥ 1 completed + ≥ 1 human-stopped wave; verify required cockpit elements are present and telemetry is retained for later analysis |
| **Stop clarity** | 100% of displayed wave ends are classified as **human checkpoint**, **failure**, or **complete** — never ambiguous | UI state audit on sample runs |
| **Efficacy visible** | Tech lead can compare **unattended orchestrated stage time** vs **human-wait time** for a wave from the console using plain-language labels | Manual review on ≥ 1 dogfood wave |
| **Process SSOT unchanged** | 0 delivery rules authored only inside gateflow-ops; process map is **projection** of pinned workflow | Product review + no in-console process editor |
| **Greenfield unchanged** | No Launchpad product work required for 004 exit | Impact map scope |
| **Dogfood served** | 003 W2 spec-lane prove-it can use **this PRD** as live subject with checkpoints on | Programme sign-off on prove-it pairing |
| **Lift not premature** | 0 human checkpoints removed or auto-passed as a 004 exit requirement | Programme audit |

Numeric promotion thresholds for lifting checkpoints are **out of scope** as exit
gates; Mission Control **surfaces** signals so the programme can calibrate later
`(Source: Outline §11, Vision §9)`.

### Scope boundary with INIT-GATEFLOW-001–003

| Topic | INIT-GATEFLOW-004 rule |
|-------|------------------------|
| Control plane, wave start API, RunStore, metrics APIs | **Consume** — available from 001–003 (003 W2 prove-it may still be open); do not rebuild in ops |
| Live Cursor AgentRunner | **Reuse** — console starts / observes; does not run agents |
| Delivery process & skills | **Display** pinned workflow — SSOT remains prayog-skills |
| Greenfield repo creation | **Launchpad** — out of scope |
| Auto-lift checkpoints / auto-merge | **Out** — visibility only; human merge unchanged |
| gateflow-ops prior state | Scaffold / APIs only — **first real Mission Control product** |
| gateflow-ops Mission Control UI | **This INIT (004)** — deferred in 003 `(Source: User-confirmed)` |
| Roles / RBAC / enterprise SSO | **Deferred** — thin ops-user + sign-in only |

---

## 2. User Experience & Functionality

### User Personas

| Persona | Role | Primary need |
|---------|------|--------------|
| **Engineering** | Wave operator | Onboard harnessed repos, start waves, see exactly which steps the agent finished and where to act |
| **Tech lead** | Gate / lift decision owner | See whether agents do useful work between intentional stops — without removing gates yet |
| **Programme sponsor** | Investment / narrative owner | Real Mission Control story: fleet, visible waves, evidence — not a spreadsheet of API calls |

### Target experience

```text
Sign in to Mission Control
        │
        ▼
Onboard an already-harnessed repo
  → strict readiness scorecard must pass
  → save as fleet member with sensible wave defaults
        │
        ▼
Fleet home → pick repo → start wave (or open existing)
        │
        ▼
Run cockpit
  → process map: finished / current / waiting on human
  → timeline: duration, outcome, agent/model per step
  → full log pane + deep links + Open PR on GitHub
        │
        ▼
Wave stops at human checkpoint (expected)
  → human decides; metrics retained for lift discussions
```

### User Stories & Acceptance Criteria

#### US-1 — Engineering signs in and onboards a harnessed repo

**As** Engineering, **I want** to sign in and onboard a repo that already has
Prayog harness + skills **so that** it becomes an operable fleet member with
clear readiness feedback.

**Acceptance criteria:**

- [ ] Operator **signs in** to Mission Control and is associated with a
  persisted **ops-user** record on subsequent actions
- [ ] **No roles, RBAC, or permission tiers** exist in v0 — every signed-in ops
  user has the same Mission Control capabilities `(Source: Outline §11 #11)`
- [ ] Operator can register an **existing** GitHub repo (org/name) for onboarding
- [ ] **Strict readiness scorecard** runs before fleet membership — outcome is
  **pass** or **fail** only, with operator-readable reasons per failed check
  category `(Source: User-confirmed)`
- [ ] Scorecard **fail** **blocks** fleet membership and shows **why** each
  failed category failed — no silent half-member, **no partial membership**
  `(Source: Outline §11 #12)`
- [ ] Scorecard **pass** persists the repo as an onboarded fleet member with
  **simple wave defaults** (e.g. trigger posture inherited from gateflow
  programme config for that repo) `[defaults shape: TBD in gateflow-ops spec]`
- [ ] Onboarding does **not** create the repo or install harness from scratch
  (Launchpad owns greenfield)
- [ ] Scorecard check **categories** (product-level; eng wiring in spec):
  - Harness present (`.harness-pin.yaml` or equivalent programme signal)
  - Skills / workflow pin resolvable for the repo’s programme
  - Gateflow programme config / service-catalog linkage (always required for v0 pass)
  - Repo reachable by Gateflow worker / ForgeClient (auth posture)
  - Minimum gateflow API connectivity for runs + metrics
- [ ] Operator **abandons** mid-onboarding (closes UI before pass) → **no** fleet
  member is created `(Source: User-confirmed)`
- [ ] Re-onboard of an **already onboarded** repo is **idempotent** — updates
  scorecard result / defaults; remains a single fleet member `(Source: User-confirmed)`

#### US-2 — Engineering operates waves from fleet home

**As** Engineering, **I want** a fleet home listing onboarded repos and their
wave health **so that** I can start or inspect waves without API gymnastics.

**Acceptance criteria:**

- [ ] Fleet home lists **all onboarded repos** with last-known wave health:
  `running`, `stopped_for_human`, `failed`, or `idle`
- [ ] Operator can navigate from fleet home into a repo’s wave list / detail
- [ ] Operator can **start a new wave** from the UI for an onboarded repo on the
  happy path `(Source: Outline §11 #8)`
- [ ] Operator can open any **in-flight** wave or any wave from the **last 30
  days** into the cockpit `(Source: User-confirmed)`
- [ ] Wave start from UI calls existing Gateflow wave-start path — Mission
  Control does not fork orchestration logic

#### US-3 — Engineering uses the run cockpit (Mission Control bar)

**As** Engineering, **I want** a high-bar run cockpit **so that** I can answer
*what happened* and *what I do next* without leaving Mission Control.

**Acceptance criteria:**

- [ ] **Process map** shows the **pinned delivery path** (read-only projection)
  and this wave’s position: finished nodes, current node, waiting on human
- [ ] **Step timeline** lists every recorded step with: timestamp(s), **duration**,
  **outcome**, and **agent / model** when an orchestrated stage ran
- [ ] **Full log pane** provides a **readable narrative** per step and at wave
  level — not links-only archaeology `(Source: Outline §11 #10)`
- [ ] Log pane includes **deep links** to PR comments, validation/findings
  reports, and artifacts when Gateflow / GitHub already record them
- [ ] **Open PR on GitHub** (and related issue/PR links where available) in one
  click from the cockpit
- [ ] Wave end state is **unambiguous**: human checkpoint vs failure vs complete
- [ ] Cockpit meets the **high bar** definition: map + timeline + full log pane
  — not a basic status list `(Source: Outline §11 #1)`
- [ ] Exit does **not** require a timed operator drill; completeness of the four
  answers above is the product bar `(Source: Outline §11 #13)`
- [ ] No drag-and-drop process editor, fleet globe, or workflow studio in v0

#### US-4 — Tech lead reviews efficacy signals for lift discussions

**As a** tech lead, **I want** plain-language efficacy signals on a wave **so
that** I can discuss what to automate next without removing checkpoints in v0.

**Acceptance criteria:**

- [ ] Cockpit or wave summary shows **unattended orchestrated progress** (time
  and/or stage count between human stops) vs **time at human stops**
- [ ] Retries / “needs another pass” patterns are visible where RunStore already
  records them (e.g. repeated stage outcomes, findings loops)
- [ ] Signals align with vision §9 promotion criteria categories: unattended hop
  success, rework/findings, fail-fast honesty (display only), gate dwell
- [ ] Mission Control **collects and retains** the numbers needed to evaluate
  operator experience and lift decisions over time — targets are set later from
  samples, not upfront `(Source: Outline §11 #13)`
- [ ] Console does **not** auto-promote `dispatch: orchestrated`, remove
  checkpoints, or change the skills pin
- [ ] Numeric **lift thresholds** are **not** configured in Mission Control v0
  `(Source: Outline §11 deferred #1)`

#### US-5 — Programme sponsor validates the Mission Control narrative

**As a** programme sponsor, **I want** a credible ops surface for the Gateflow
story **so that** we can demonstrate fleet operations and evidence-based
autonomy — not API scripts.

**Acceptance criteria:**

- [ ] ≥ 1 harnessed repo onboarded and visible on fleet home
- [ ] ≥ 1 wave started from Mission Control and inspectable in cockpit end-to-end
- [ ] Dogfood wave for this PRD (003 W2 companion) is observable in cockpit with
  human checkpoints clearly marked as **expected stops**
- [ ] No dependency on Launchpad greenfield work for sponsor demo path

#### US-6 — Programme dogfoods spec-lane prove-it (003 W2 companion)

**As** the programme, **we want** this PRD’s delivery journey to be the live
subject of INIT-GATEFLOW-003 W2 **so that** spec-lane automation is proven with
real agent work and Mission Control observability.

**Acceptance criteria:**

- [ ] Programme may use **this PRD** as the live dogfood subject for 003 W2
  engg **spec-lane** prove-it — pin / `dispatch` for those skills remains
  **INIT-GATEFLOW-003 W2** delivery, **out of 004** `(Source: User-confirmed)`
- [ ] Human checkpoints during dogfood remain **on** — expected stops, not failures
- [ ] Cockpit shows dogfood wave progress with full log pane + efficacy signals
- [ ] Prove-it success is **003 W2 exit evidence**; Mission Control delivery is
  **004 exit evidence** — paired but distinct

### Onboarding scorecard (product categories)

Exact checks and probes are **engineering spec**; operators see these **categories**:

| # | Category | Pass meaning (operator language) |
|---|----------|----------------------------------|
| 1 | **Harness posture** | Repo has programme harness installed and pin resolvable |
| 2 | **Workflow / skills** | Pinned `workflow.yaml` and skills contract readable for this programme |
| 3 | **Gateflow linkage** | Repo registered in programme service catalog / gateflow config as operable (always required for v0 pass) |
| 4 | **GitHub / Forge access** | Gateflow can reach repo for PR thread and status (worker + ForgeClient) |
| 5 | **API readiness** | Runs and metrics APIs return data for this repo (smoke probe) |

**Fail** on any category → not onboardable. There is **no partial onboarding
state** in v0. Mission Control must block onboarding and explain why the repo
failed the scorecard `(Source: User-confirmed)`.

### Functional Requirements (FR)

| ID | Requirement | Acceptance criteria |
|----|-------------|---------------------|
| **FR-32** / **REQ-32** | Thin ops-user identity + sign-in | Persist signed-in ops-user identity; sign-in required for Mission Control; actions attributable to user; **no** roles/RBAC/permission matrix in v0 `(Source: Outline §11 #11)`. *Implementation Note:* record fields in §4 Ops identity |
| **FR-33** / **REQ-33** | Strict onboarding scorecard | Scorecard categories above; **pass/fail only**; pass required for fleet membership; fail **blocks** with operator-readable reasons per failed category; **no partial state**; category 3 always required; no harness install from console; abandon → no member; re-onboard → idempotent `(Source: Outline §11 #9, #12)` |
| **FR-34** / **REQ-34** | Fleet home | List onboarded repos + last-known wave health; navigate to repo waves |
| **FR-35** / **REQ-35** | Start and inspect waves from UI | Start wave happy path from UI; open in-flight runs and runs from the **last 30 days**; delegate to Gateflow wave-start API `(Source: Outline §11 #8)` |
| **FR-36** / **REQ-36** | Run cockpit — Mission Control bar | Process map (pinned projection); step timeline (time, outcome, agent/model); **full log pane** with narrative + deep links; GitHub PR jump; unambiguous stop state; **no timed drill** exit gate `(Source: Outline §11 #1, #10, #13)` |
| **FR-37** / **REQ-37** | Efficacy visibility + collect numbers | Display unattended orchestrated progress vs human-wait time; retries/findings where recorded; **collect/retain** usage and efficacy numbers for later target-setting; **no** auto-lift or threshold configuration in v0 `(Source: Outline §11 #13)` |
| **FR-38** / **REQ-38** | Process display-only | Project pinned workflow for map/timeline; **zero** in-console authoring of delivery rules |
| **FR-39** / **REQ-39** | Consume Gateflow control plane | Use existing run/metrics/wave APIs; extend gateflow **only** where console needs missing programme capabilities (onboarding records, clearer wave summaries) — not a second orchestrator |
| **FR-40** / **REQ-40** | Dogfood / 003 W2 engg spec-lane companion | Programme may use this PRD as live subject for engg **spec-lane** prove-it; checkpoints remain on; **prayog-skills** pin / `dispatch` for `spec-draft`, `initiative-feasibility`, `spec-technical-review`, `spec-implementation-plan` is **out of 004** (owned by INIT-GATEFLOW-003 W2) `(Source: User-confirmed)` |

**Inherited (unless noted):** INIT-GATEFLOW-001–003 requirements remain in force for
the control plane Mission Control consumes (003 W2 prove-it may still be open).
Human checkpoints are never auto-passed; Gateflow never auto-merges. Canonical
product ids are **`REQ-n`**; legacy `FR-n` ≡ `REQ-n`.

### Error Handling

| Failure | System response | Visibility |
|---------|-----------------|------------|
| Sign-in failure / expired session | Block Mission Control actions; no anonymous fleet ops | Sign-in prompt with clear error |
| Scorecard fail | Do not add to fleet; persist last scorecard result | Category-level reasons on onboarding screen |
| Scorecard probe error (transient) | Allow retry; do not silent-pass | “Could not verify — retry” with probe id |
| Abandon mid-onboarding | Do not create fleet member | Last scorecard result may be retained for retry; no membership |
| Re-onboard already-member | **Idempotent** update of scorecard/defaults; single fleet member | Updated member record; no duplicate |
| Wave start blocked (preconditions) | Do not enqueue run; surface Gateflow structured error | Cockpit / start form shows precondition reason |
| Run detail incomplete (missing log slice) | Show available narrative + links; indicate gaps | “Incomplete log” badge — do not invent events |
| Gateflow API unavailable | Degraded fleet home; block start | Service error banner |
| Human checkpoint stop | **Expected** — not failure styling | Clear “waiting on human” state |
| Agent stage failure | Failure styling; no workflow auto-advance | Timeline + log pane show failed outcome |

### Non-Goals

| Non-goal | Rationale |
|----------|-----------|
| Greenfield repo creation / scaffolding | Launchpad owns paved-road factory `(Source: Outline §7)` |
| Editing delivery process in console | Process SSOT stays in pinned skills `(Source: Outline §11 #6)` |
| Auto-removing human checkpoints / auto-merge | Lift-on-metrics later; merge human-owned |
| Live second coding agent (OpenCode / Claude) | Later INIT after Cursor efficacy clear |
| Slack / Teams as primary ops surface | Mission Control is home; notifiers unchanged |
| Fancy multi-programme world map / cinema UI | High bar ≠ spectacle |
| Separate AI workflow engine in console | Must not fork process ownership |
| Replacing GitHub / IDE as code authoring surface | Mission Control operates waves; agents code in worker |
| Roles, RBAC, per-feature permissions | Thin ops-user only `(Source: Outline §11 #11)` |
| Enterprise SSO / SCIM / multi-tenant org admin | Later INIT |
| Numeric lift playbooks / auto-promotion | Visibility only in v0 `(Source: Vision §9)` |
| Partial / warning-state fleet membership | Pass/fail only; fail blocks with reasons `(Source: Outline §11 #12)` |
| Timed cockpit drill as exit KPI | Collect numbers first; set targets after samples `(Source: Outline §11 #13)` |
| **prayog-skills** pin / `dispatch` for engg spec-lane | Owned by **INIT-GATEFLOW-003 W2**; 004 only reads the pin for process map `(Source: User-confirmed)` |
| Rebuilding Gateflow control plane | Finished in 001–003 |

### Product Principles

1. **Operate, don’t orchestrate twice** — Mission Control consumes Gateflow APIs  
2. **High bar, not fancy chrome** — cockpit answers what happened and what’s next  
3. **Brownfield onboard only** — harnessed repos; greenfield is Launchpad  
4. **Process is read-only** — pinned skills remain SSOT  
5. **Evidence before autonomy** — show lift signals; do not lift gates in v0  
6. **Collect numbers before targets** — retain efficacy / experience data; calibrate later  
7. **Thin identity** — know who operated; defer enterprise IAM  
8. **Dogfood is real work** — this PRD proves spec-lane with checkpoints on  
9. **Fail closed on onboarding** — pass or block with reasons; never half-member  

---

## 3. AI System Requirements

Mission Control **does not run** coding agents. It **displays** agent execution
that Gateflow already dispatches (INIT-GATEFLOW-003 live Cursor).

### Tool Requirements

| Capability | Owner | Mission Control role |
|------------|-------|----------------------|
| **AgentRunner** (Cursor live) | gateflow worker | **Observe** — show runner, model, stage outcomes in timeline |
| **Delivery skills** | prayog-skills pin | **Project** pinned workflow for process map |
| **Run / metrics APIs** | gateflow | **Consume** — timeline, cycle-time, efficacy aggregates |
| **GitHub PR thread** | ForgeClient | **Link** — deep links + Open PR |

### Evaluation Strategy

| Dimension | Method | Pass threshold |
|-----------|--------|----------------|
| **Cockpit completeness** | Open a completed dogfood wave | Process map + timeline + log narrative + PR link present |
| **Human stop clarity** | Open wave stopped at checkpoint | State = waiting on human; not styled as agent failure |
| **Efficacy labels** | Review dogfood wave | Unattended vs human-wait breakdown visible in plain language |
| **Numbers collected** | Inspect retained wave / stage metrics after dogfood | Unattended duration, human-wait duration, stage outcomes, retries/findings where recorded — present for later analysis; **no** target thresholds required for exit |
| **Scorecard honesty** | Attempt onboard of non-harnessed or incomplete repo | Fail with readable reasons; 0 fleet membership; **no** partial member |
| **No process fork** | Audit console actions | 0 mutations to workflow.yaml or skills pin from ops UI |
| **003 W2 pairing** | Run spec-lane prove-it on this PRD | Live Cursor stages visible in cockpit; checkpoints on |

---

## 4. Technical Specifications

### Architecture Overview

```text
  Operator browser
        │
        ▼
  gateflow-ops (Mission Control UI + BFF)
        │  thin ops-user auth
        ├──► gateflow HTTP API (runs, metrics, wave-start) — primary
        ├──► onboarding / fleet store (ops DB or gateflow extension)
        └──► GitHub (links; optional read for enrichments)
        │
        ▼
  Gateflow control plane (001–003) — unchanged orchestration
        │
        ▼
  Worker + live Cursor AgentRunner + RunStore + ForgeClient
```

**Rule:** gateflow-ops is a **consumer and presenter**. It does not embed
PolicyEngine, workflow resolution, or AgentRunner execution.

### Integration Points

| Integration | Direction | Mechanism |
|-------------|-----------|-----------|
| **gateflow** | Inbound to ops | Run detail, metrics, wave-start, board/read APIs (001–003); optional new endpoints for fleet onboarding records / wave summaries (FR-39) |
| **PostgreSQL** | Read (via gateflow) | RunStore stages, cycle-time fields (FR-30), outcomes |
| **GitHub** | Outbound links | PR URL, comment anchors, issue links from run metadata |
| **prayog-skills** | Read | Pinned workflow for process map projection |
| **launchpad** | None (product) | No greenfield scaffolding from Mission Control |
| **prayog-meta** | Read | Service catalog / programme config signals for scorecard |
| **Programme service token** | Behind BFF | Existing 001–003 API auth patterns may apply server-side |

### Log pane — product outcomes

Engineering selects transports in spec; operator-visible **minimum content**:

| Source (priority) | Pane contribution |
|-------------------|-------------------|
| RunStore stage events | Ordered narrative: node entered, runner started/ended, outcome, duration |
| Notifier / PR comments | Quoted or summarized GitHub comment snippets with link |
| Findings / validation artifacts | Link + short label when run references report paths or URLs |
| Wave-level summary | Enqueue time, stop reason, total wave duration (`wave_duration_ms` when present) |

Gaps in sources must show **incomplete log** honesty (badge / notice) — never
fabricated narrative. Incomplete log is a **display** state for evidence gaps,
not an onboarding / fleet-membership state.

### Operator-experience numbers — product outcomes {#operator-experience-numbers}

v0 **collects** numbers; it does **not** set timed cockpit SLAs or lift
thresholds as exit gates `(Source: Outline §11 #13)`.

| Number (product meaning) | Why we collect it | Eng detail |
|--------------------------|-------------------|------------|
| Stage `duration_ms` + outcome + runner/model | Reconstruct agent work between stops | Reuse RunStore / FR-30 where present |
| Wave duration (accept → stop/fail) | Wave-level cycle time | Existing wave cycle-time fields |
| Human-wait duration at checkpoints | Gate dwell for lift discussions | Derive from checkpoint enter/leave timestamps `[TBD in spec]` |
| Unattended orchestrated stage time between human stops | Efficacy vs babysitting | Aggregate from stage metrics `[TBD in spec]` |
| Retry / findings-loop counts where recorded | Rework signal | RunStore outcomes already available |
| Ops-user attribution on onboard / wave-start | Who operated | Thin ops-user id |

Exact retention schema and query APIs are **engineering spec**; product rule is
**retain enough to set targets later**.

### Ops identity — product outcomes

| Topic | v0 decision | Eng spec deferred |
|-------|-------------|-------------------|
| User record | Thin identity record: id, display identity, created_at | Exact fields (Implementation Note for REQ-32) |
| Sign-in | Required for all Mission Control routes | Session vs token bridge; bootstrap / invite flow `[TBD]` |
| Authorization | Single capability tier for all signed-in users | None |
| Attribution | Wave-start and onboarding actions store `ops_user_id` | Audit log shape |

### Security & Privacy

| Concern | Requirement |
|---------|-------------|
| **Console access** | Unauthenticated users cannot onboard, start waves, or view fleet |
| **API credentials** | Programme service tokens stay server-side (BFF); not exposed to browser |
| **Repo scope** | Fleet members only repos explicitly onboarded; no org-wide trawl by default |
| **Human gates** | Mission Control does not auto-approve gates or merge PRs |
| **PII** | Ops-user identity minimal; no enterprise directory sync in v0 |

### Repositories

| Repo | This INIT deliverable |
|------|------------------------|
| **gateflow-ops** | Mission Control UI + BFF; fleet onboarding; cockpit; thin ops-user auth |
| **gateflow** | **Supporting only** — APIs/records if console cannot be met by existing run/metrics contracts |
| **prayog-skills** | **Not affected** — pin / `dispatch` for engg spec-lane stays on **INIT-GATEFLOW-003 W2**; 004 reads pin for process map only |
| **launchpad** | **No product delivery** |
| **prayog-meta** | This PRD, impact map, dogfood programme artifacts |

---

## 5. Risks & Roadmap

### Phased Rollout

| Phase | Scope | Entry |
|-------|-------|-------|
| **W0** | Thin ops-user + sign-in skeleton; fleet data model; scorecard probes (read-only) | Draft PRD + impact map approved |
| **W1** | Fleet home + onboarding pass/fail + start wave from UI | W0 auth + gateflow API smoke |
| **W2** | Run cockpit: process map + timeline + log pane + GitHub jump | W1 wave operable |
| **W3** | Efficacy panel + dogfood hardening with 003 W2 prove-it | W2 cockpit on real runs |
| **Exit** | Success criteria table §1 met on ≥ 1 onboarded repo + dogfood wave | Programme sign-off |

### Technical Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| **Log pane richness vs API gaps** | Cockpit feels “basic” despite high bar | FR-39 targeted gateflow extensions; incomplete-log honesty (display only) |
| **Scorecard false positives** | Unready repo enters fleet | Fail-closed; pass/fail only; operator-readable failures |
| **Process map drift from pin** | Misleading workflow display | Read pin version used at wave start; refresh on run open |
| **Thin auth too thin for prod** | Shared-console abuse | Network boundary + programme token behind BFF; document v0 limits |
| **003 W2 / 004 coupling** | Dogfood blocked if 003 pin work slips | Parallel tracks; skills pin owned by **003**; 004 exit does not require pin change |
| **Scope creep into Launchpad** | Timeline slip | Non-goals enforced in impact map |
| **Premature SLAs** | Fake precision without samples | No timed drill / lift thresholds as exit; collect numbers first |

### Assumptions

| ID | Assumption | Status | Dependent REQs |
|----|------------|--------|----------------|
| A1 | INIT-GATEFLOW-001 and INIT-GATEFLOW-002 are finished/delivered | Confirmed `(Source: User-confirmed)` | REQ-35, REQ-39 |
| A2 | INIT-GATEFLOW-003 control plane + live Cursor + run/metrics/wave-start APIs are available; **003 W2 prove-it may still be open** | Confirmed `(Source: User-confirmed)` | REQ-35, REQ-36, REQ-37, REQ-39, REQ-40 |
| A3 | RunStore stage + wave cycle-time fields (003 REQ-30) are available for cockpit timeline and efficacy numbers | Confirmed `(Source: User-confirmed)` | REQ-36, REQ-37 |
| A4 | Gateflow can reach onboarded repos via existing ForgeClient for scorecard GitHub access and Open PR | Confirmed `(Source: User-confirmed)` | REQ-33, REQ-36 |
| A5 | Engg **spec-lane** pin / `dispatch` for dogfood is owned by **INIT-GATEFLOW-003 W2** — **out of 004** delivery; 004 may still be used as dogfood subject | Confirmed `(Source: User-confirmed)` | REQ-40 |

### Dependencies

| Dependency | Role for 004 |
|------------|--------------|
| gateflow run / metrics / wave-start APIs (001–003) | Consume for fleet operate + cockpit |
| RunStore cycle-time fields (003 REQ-30) | Timeline + efficacy numbers |
| ForgeClient / GitHub reachability | Scorecard category 4 + Open PR |
| prayog-skills pin (read for process map) | Display-only projection (REQ-36, REQ-38); pin/`dispatch` edits are **003 W2**, not 004 |
| prayog-meta service catalog / programme config signals | Scorecard category 3 (always required) |

### Decisions (resolved) {#decisions-resolved}

| # | Decision | Resolution |
|---|----------|------------|
| 1 | Basic run list vs Mission Control bar? | **Mission Control** — high bar, not fancy chrome `(Source: Outline §11 #1)` |
| 2 | Greenfield create vs onboard harnessed? | **Onboard harness-enabled only**; greenfield = Launchpad `(Source: Outline §11 #2)` |
| 3 | Primary product surface? | **gateflow-ops** Mission Control `(Source: Outline §11 #3)` |
| 4 | Autonomy stance? | Agents do most work **between** intentional controls; checkpoints stay until metrics justify lift `(Source: Outline §11 #4)` |
| 5 | Role of this PRD for 003? | **Dogfood subject** for engg spec-lane prove-it `(Source: Outline §11 #5)` |
| 6 | Process editing in console? | **Out** — display only `(Source: Outline §11 #6)` |
| 7 | Vision first? | Vision maturity + lift-on-metrics updated before this outline `(Source: Outline §11 #7)` |
| 8 | Start wave from UI in v0? | **Yes** `(Source: Outline §11 #8)` |
| 9 | Onboard readiness depth? | **Strict scorecard** — must pass `(Source: Outline §11 #9)` |
| 10 | Evidence richness? | **Full log pane with links** `(Source: Outline §11 #10)` |
| 11 | Identity model? | **Thin ops-user**; no roles/RBAC `(Source: Outline §11 #11)` |
| 12 | Partial onboarding state? | **No** — pass/fail only; fail blocks with reasons `(Source: Outline §11 #12)` |
| 13 | Timed cockpit drill as success bar? | **No** — collect numbers first; set targets after samples `(Source: Outline §11 #13)` |

### Open Questions

1. Exact scorecard probe implementation per category — Engineering in gateflow-ops / gateflow spec (categories in §2 are normative).
2. Sign-in mechanism (session vs token bridge, invite/bootstrap) — Engineering in gateflow-ops spec (product: sign-in required; thin user record).
3. Exact field names / retention for human-wait and unattended aggregates — Engineering in spec (product outcomes in [§4 Operator-experience numbers](#operator-experience-numbers)).
4. Numeric lift thresholds — deferred until after firsthand Mission Control samples (vision §9).

### Relationship to later INITs

| Topic | Deferred to later INIT |
|-------|------------------------|
| Richer log transports / long retention | Post-v0 observability |
| Historical process map snapshot per wave | H2+ DB projection |
| “Propose process change” → git/skills PR | Not silent console edit |
| OpenCode / Claude live runners | After Cursor efficacy |
| Numeric lift playbooks / timed experience SLAs | After months of Mission Control data |
| Roles / RBAC / SSO | Programme scale |

---

## Appendix A — Ownership split

| Topic | Owner in this INIT |
|-------|--------------------|
| Create new repo / greenfield paved road | **Launchpad** — out of scope |
| Bring existing harnessed repo into operations | **Mission Control (004)** |
| Delivery process & skills | **prayog-skills** — console displays |
| Who runs the coding agent | **Gateflow** — console starts / observes |
| When to loosen human checkpoints | **Programme / tech lead** — metrics-informed, not auto |
| Merge to main | **Human accountable** |
| Who can use Mission Control | **Thin ops-user** — no role model in v0 |

## Appendix B — Feature checklist

| # | Feature | In this INIT |
|---|---------|--------------|
| 0 | Thin ops-user identity + sign-in (no roles) | Yes |
| 1 | Onboard harness-enabled repo to fleet | Yes |
| 2 | Strict onboarding scorecard (pass required; no partial) | Yes |
| 3 | Fleet home (repo list + wave health) | Yes |
| 4 | Start + inspect waves from console | Yes |
| 5 | Run cockpit: process map + step timeline | Yes |
| 6 | Full log pane with links + Open PR on GitHub | Yes |
| 7 | Per-step timing / outcome / agent info | Yes |
| 8 | Efficacy signals + collect numbers for later lift | Yes (visibility + retain) |
| 9 | Dogfood subject for 003 spec-lane prove-it | Yes (programme role) |
| 10 | Greenfield / Launchpad create | No |
| 11 | Edit delivery process in the console | No |
| 12 | Auto-lift checkpoints / auto-merge | No |
| 13 | Second coding agent / Slack-primary ops | No |
| 14 | Roles / RBAC / permission tiers | No |
| 15 | Timed cockpit drill as exit KPI | No |
| 16 | Partial fleet membership | No |

## Appendix C — Traceability

| Outline / Discovery | PRD section |
|---------------------|-------------|
| Outline §1–2 problem / solution | §1 Executive Summary |
| Outline §3 target experience | §2 Target experience |
| Outline §4 ownership / Launchpad | Appendix A; Non-Goals |
| Outline §5 personas / JTBD | §2 Personas + User Stories |
| Outline §6 features | REQ-32–REQ-40 (FR alias); US-1–US-6 |
| Outline §7 non-goals | §2 Non-Goals |
| Outline §10 success | §1 Success Criteria |
| Outline §11 #1 | §5 Decision #1 |
| Outline §11 #2 | §5 Decision #2 |
| Outline §11 #3 | §5 Decision #3 |
| Outline §11 #4 | §5 Decision #4 |
| Outline §11 #5 | §5 Decision #5 |
| Outline §11 #6 | §5 Decision #6 |
| Outline §11 #7 Vision first | §5 Decision #7 |
| Outline §11 #8 | §5 Decision #8 |
| Outline §11 #9 | §5 Decision #9 |
| Outline §11 #10 | §5 Decision #10 |
| Outline §11 #11 | §5 Decision #11 |
| Outline §11 #12 pass/fail only | REQ-33; Decision #12; Non-Goals |
| Outline §11 #13 collect numbers | REQ-37; Decision #13; §4 Operator-experience numbers |

## Appendix D — Process next steps

| Step | Owner | Artifact |
|------|-------|----------|
| 1 | PM / sponsor | Review this Draft PRD |
| 2 | Engineering | `/validate-requirements` |
| 3 | Engineering | `/prd-impact-map` → **gateflow-ops** primary |
| 4 | Engineering | Spec PRs (ops auth, scorecard probes, telemetry retention) |
| 5 | Programme + Engineering | Delivery waves W0–W3 + 003 W2 dogfood pairing |

## Appendix E — One-sentence product

> Gateflow Mission Control lets the programme **onboard already-harnessed
> repos**, **run and inspect delivery waves**, and **see each step’s outcome,
> timing, and GitHub evidence** on a real cockpit — so we can move toward agents
> doing most of the work between intentional human controls **only when
> metrics earn it** — without Launchpad greenfield or rewriting the delivery
> process in the UI.
