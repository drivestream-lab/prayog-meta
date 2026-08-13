# INIT-GATEFLOW-016 — Gateflow Mission Control, greenfield API-grounded scope

**Status:** Draft PRD · **Author:** programme PM · **Date:** 2026-08-12  
**Outline:** [INIT-GATEFLOW-016-outline.md](./INIT-GATEFLOW-016-outline.md)  
**Vision:** [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md) §9 maturity · §10 H2 lift-on-metrics  
**Component:** GATEFLOW-OPS · **Type:** operations / Mission Control  
**Predecessor (retired):** INIT-GATEFLOW-004 (Gateflow Mission Control) — deleted 2026-08-12, superseded outright by this initiative. 004 predated INIT-GATEFLOW-013 and INIT-GATEFLOW-015, both now delivered and reused here instead of re-specified.  
**Delivered dependencies:** INIT-GATEFLOW-001–003 (control plane + live Cursor); INIT-GATEFLOW-010/011 (initiative closure + readout composition — confirmed via direct inspection of `initiatives_routes.py` and its readout services this session; both PRDs' own header still reads `Status: draft PRD`); INIT-GATEFLOW-013 (programme-first onboarding + real readiness); INIT-GATEFLOW-014 (Runs API under JWT); INIT-GATEFLOW-015 (skill efficacy / factory effectiveness / delivery scorecard).  
**Related, not blocking:** none.

---

## Document control

| Field | Value |
|---|---|
| Initiative ID | INIT-GATEFLOW-016 |
| Programme | prayog |
| Primary repo | drivestream-lab/gateflow-ops |
| Supporting | drivestream-lab/gateflow (read-only API consumer this INIT — no contract or code change expected) |
| Explicitly **not** touched this INIT | Any gateflow backend change (fleet-summary endpoint, workflow-pin readiness probe, process-map endpoint, log-pane narrative/deep-link richness — all named non-goals, later fast-follow); platform-admin console (`programme_admin_routes.py` / `agent_catalogue_router`); Launchpad greenfield scaffolding; `prayog-skills` pin/`dispatch` edits; server-side composite onboarding-scorecard endpoint (locked client-side this INIT, OQ-2) |
| Target users | Engineering / PE (onboard repos, operate waves, authorize forge actions); tech lead (efficacy signals, lift decisions); programme leadership (delivery scorecard, initiative tracking) |
| Depends on | INIT-GATEFLOW-001–003, 010/011, 013, 014, 015 — all delivered. This initiative builds a UI/BFF only; it introduces no new gateflow capability |
| Exit proof | End-to-end UI test per capability area against ≥ 1 onboarded repo and ≥ 1 real wave; zero gateflow route/schema/contract changes in the diff |

---

## 1. Executive Summary

### Problem Statement

`gateflow-ops` is an empty chassis — sign-in plus one throwaway exemplar page — while `gateflow` has quietly accumulated a rich, tenant-scoped API surface across five delivered initiatives (onboarding readiness, wave lifecycle, checkpoint evidence, board/ticket linking, full initiative/wave delivery readouts, and three efficacy APIs) that no UI has ever surfaced. The prior attempt to scope this, INIT-GATEFLOW-004, was drafted before most of that surface existed and was retired without ever reaching spec/build stage.

### Proposed Solution

Build `gateflow-ops` bottom-up from gateflow's real, live route table across seven capability areas — identity & access, fleet onboarding, wave operations, checkpoint evidence, board & tickets, initiative & delivery tracking, and metrics & efficacy — with **zero new gateflow backend work**. Every requirement below cites the exact existing gateflow route it consumes.

### Success Criteria

| KPI | Target | Measurement |
|---|---|---|
| **Onboarding without raw API calls** | An operator takes a catalogue candidate to a single pass/fail readiness verdict entirely from the UI | End-to-end UI test, ≥ 1 real repo |
| **Wave operable from console** | An operator starts a wave, opens its run timeline, and authorizes a pending forge action without a curl call | End-to-end UI test, ≥ 1 real wave that reaches an `external-action` stop |
| **Initiative tracking composed, not hand-stitched** | An operator opens one initiative and sees spec → implementation → closeout → merge → completion/closure readouts sourced entirely from gateflow, no manual board/GitHub cross-referencing | UI test against ≥ 1 initiative with a board EPIC + ≥ 1 run |
| **Efficacy/scorecard readable end-to-end** | Tech lead and programme-lead views (CAP-G) both render tenant-scoped data with no placeholder/mock values | UI test against ≥ 1 tenant with real run history |
| **Zero gateflow diff** | 0 gateflow route, schema, or contract changes ship as part of this initiative | Diff review at impact-map and PR time |
| **No platform-admin leakage** | 0 UI surfaces or BFF calls hit `PLATFORM_ADMIN`-gated routes | Route-gate audit against `require_role` on every BFF call |

### Capability ↔ wave ↔ requirement map

| CAP | Wave | Capability | REQ |
|---|---|---|---|
| CAP-A | W0 | Identity & access | REQ-01–REQ-02 |
| CAP-B | W0 | Fleet onboarding | REQ-03–REQ-08 |
| CAP-C | W1 | Wave operations | REQ-09–REQ-12 |
| CAP-F | W2 | Initiative & delivery tracking | REQ-13–REQ-21 |
| CAP-G | W3 | Metrics & efficacy panel | REQ-22–REQ-25 |
| CAP-D | W4 | Checkpoint evidence | REQ-26–REQ-27 |
| CAP-E | W4 | Board & tickets | REQ-28–REQ-31 |

---

## 2. User Experience & Functionality

### User Personas

| Persona | Role | Primary need |
|---|---|---|
| **Engineering / PE** | Wave operator | Onboard harnessed repos, start/inspect waves, authorize pending forge actions — without API gymnastics |
| **Tech lead** | Gate / lift decision owner | Read skill-efficacy and factory-effectiveness signals to decide what to automate next |
| **Programme leadership** | Delivery narrative owner | Read the delivery scorecard and track initiatives end-to-end without stitching board + GitHub manually |

### User Stories & Acceptance Criteria

#### US-1 — Identity & access `(CAP-A)`

**As an** operator, **I want** to see my tenant's identity and invite a teammate, **so that** more than one person can operate the console.

- [ ] Signed-in operator can view their tenant's detail (`GET /api/v1/tenants/{tenant_id}`) — already gated by `require_tenant_resolved` (JWT tenant must match path tenant)
- [ ] `GET /api/v1/tenants` (list) is **not** surfaced in this console — CAP-A only needs the caller's single resolved tenant (`resolved.tenant_id` already defaults to the caller's own tenant/user), not a cross-tenant list; the route is reachable by `TENANT_ADMIN` too, so this is a scope choice, not a role-gate restriction (D3)
- [ ] Operator can invite a teammate to the tenant (`POST /api/v1/tenants/{tenant_id}/users`)
- [ ] No roles/RBAC tier is introduced client-side — every signed-in operator has identical capability (D3)

#### US-2 — Onboard a harnessed repo `(CAP-B)`

**As** Engineering, **I want** to onboard a repo already on the Prayog paved road, **so that** it becomes an operable fleet member with a clear pass/fail verdict.

- [ ] Operator browses catalogue candidates (`GET .../programme/catalogue`) and can trigger a re-sync (`POST .../catalogue/refresh`)
- [ ] Operator connects/re-syncs the tenant's programme meta checkout (`PUT .../programme/connect`, `GET .../programme/connection`) before catalogue browsing is possible
- [ ] Operator admits a repo (`POST .../programme/repos/select`); the console surfaces the returned per-repo outcome (`ok/already_selected/setup_failed/status_failed/probe_failed/out_of_catalogue`) in plain language, never a raw enum
- [ ] Operator triggers a readiness check (`POST .../programme/repos/readiness/refresh`) and the console composes `harness_verified` + `verdict_type` (`ready/not_ready/tool_unavailable`) into a **single pass/fail presentation** — never a partial/ambiguous state (D7, OQ-2 locked: client-side composition, no gateflow ask this initiative)
- [ ] Operator can remove a repo from the fleet (`POST .../programme/repos/deselect`)
- [ ] A `setup_failed`/`probe_failed`/`status_failed`/`out_of_catalogue` outcome blocks fleet membership with an operator-readable reason — no silent partial member

#### US-3 — Operate waves `(CAP-C)`

**As** Engineering, **I want** to start and inspect waves from the console, **so that** I never need a raw API call for day-to-day operation.

- [ ] Operator starts an implement/spec/closeout wave from the UI (`POST /api/v1/waves/{implement,spec,closeout}/start`)
- [ ] Operator lists runs filterable by initiative/wave/org/repo/status (`GET /api/v1/runs`)
- [ ] Operator opens a run's detail and timeline — stages, events, outcomes, durations, PR number (`GET /api/v1/runs/{run_id}`)
- [ ] Operator authorizes a pending forge action when a run is `STOPPED` at an `external-action` node (`POST /api/v1/runs/{run_id}/forge/authorize`) — the console never Cursor-dispatches forge skills itself, it only calls this explicit-authorization endpoint
- [ ] A run's stop state renders unambiguously as human-checkpoint / failure / complete — never styled as an error when it is an expected stop

#### US-4 — Track initiative delivery end-to-end `(CAP-F)`

**As** programme leadership or Engineering, **I want** one composed view of an initiative's full lifecycle, **so that** I never manually cross-reference the board and GitHub.

- [ ] Operator lists initiatives composed from runs + board EPIC tickets (`GET /api/v1/initiatives`) and opens detail (`GET /api/v1/initiatives/{id}`)
- [ ] Operator views the per-wave status map — `done`/`ready-to-start`/`blocked`/`active` (`GET .../{id}/waves`)
- [ ] Operator views the spec-lane readout (`GET .../{id}/spec`), and per-wave implementation (`GET .../waves/{wave_id}/implementation`), closeout (`.../closeout`), and merge (`.../merge`) readouts
- [ ] Operator views completion eligibility (`GET .../{id}/completion`) and a closure preview (`GET .../{id}/closure`)
- [ ] Operator can start initiative closure (`POST /api/v1/initiatives/closure/start`)
- [ ] Every readout that composes board+GitHub state honestly reflects gaps (e.g. no Draft PR yet) rather than inventing data — inherited directly from `InitiativeReadoutService`'s own scope

#### US-5 — Metrics & efficacy panel `(CAP-G)`

**As a** tech lead or programme lead, **I want** the skill-efficacy, factory-effectiveness, and delivery-scorecard views, **so that** I can make lift/automation decisions on evidence.

- [ ] Tech lead reads skill/spec efficacy — first-pass rate, findings rate, retry avg, learning codify rate, filterable by `model_id`/`prompt_revision` (`GET /api/v1/metrics/skill-efficacy`)
- [ ] PE/tech lead reads factory effectiveness — unattended Pass-1 rate, stop-reason breakdown, gate dwell time, wave cycle time by lane (`GET /api/v1/metrics/factory-effectiveness`)
- [ ] Programme lead reads the delivery scorecard — rework rate, initiatives closed with evidence, factory coverage, each as `as_of` + all-time + trailing-90-day delta (`GET /api/v1/metrics/delivery-scorecard`)
- [ ] Legacy stage-duration heatmap by node/runner/model remains available (`GET /api/v1/metrics/runs`)
- [ ] All four views render tenant-scoped data only; a cross-tenant read attempt is refused/scoped empty, never another tenant's data

#### US-6 — Checkpoint evidence `(CAP-D)`

**As** Engineering, **I want** to see live checkpoint status and history, **so that** I can confirm a PR's gate state without leaving the console.

- [ ] Operator queries live checkpoint status by raw PR ref or composed initiative+wave ref (`GET /api/v1/checkpoints/status`)
- [ ] Operator views checkpoint history for a PR (`GET /api/v1/checkpoints/history`)
- [ ] A composed-ref query with no matching run returns a named "no run found for this wave" state, never a fabricated status

#### US-7 — Board & ticket linking `(CAP-E)`

**As** Engineering, **I want** to see and manage board tickets from the console, **so that** I don't context-switch to the tracker for routine linking.

- [ ] Operator lists board tickets for a repo (`GET /api/v1/board/tickets`, required `org`/`repo` filters)
- [ ] Operator creates an EPIC/Feature ticket (`POST /api/v1/board/tickets`), idempotent on `initiative_id`+`type`
- [ ] Operator updates ticket status (`PATCH /api/v1/board/tickets/{ticket_id}/status`)
- [ ] Operator links a PR to a ticket (`POST /api/v1/board/tickets/{ticket_id}/links`)

### Non-Goals

| Non-goal | Why |
|---|---|
| Fleet/wave-health summary endpoint | Named gateflow backend gap — deferred fast-follow, not this initiative |
| Workflow/skills-pin readiness probe | Named gateflow backend gap — deferred fast-follow |
| Process-map (pinned workflow DAG) projection | Named gateflow backend gap — deferred fast-follow |
| Log-pane narrative + PR-comment/findings deep links | Named gateflow backend gap — deferred fast-follow |
| Platform-admin console (programme create/wipe, tenant-admin attach, lane-defaults, effective-runner, agent-catalogue) | Role mismatch (`PLATFORM_ADMIN`, not `TENANT_ADMIN`) — distinct future persona/initiative (D4) |
| Server-side composite onboarding-scorecard endpoint | Locked as client-side composition this initiative (D7, OQ-2 resolved) |
| 003 W2 engg spec-lane dogfood role | Dropped — OQ-1 resolved; this initiative is not a dogfood vehicle |
| Editing delivery process in console | Process stays display-only; `prayog-skills` pin remains SSOT (D5) |
| Launchpad greenfield / new-repo scaffolding | Different job — out of every Mission Control attempt |
| Auto-lift checkpoints / auto-merge | Visibility only — humans still own gates and merges |
| Roles / RBAC / permission tiers | Thin ops-user only (D3) |

### Assumptions

| ID | Assumption | Status | Dependent REQs |
|---|---|---|---|
| A1 | Every gateflow route this initiative consumes remains `require_role(RoleType.TENANT_ADMIN)`-gated and stable for the duration of delivery | Confirmed by direct source inspection this session | All REQs |
| A2 | CAP-F readouts (`InitiativeReadoutService` and siblings) already compose "runs + board EPIC/Feature tickets" and correctly scope to what Gateflow can observe — this console displays that composition as-is, it does not reinterpret or backfill gaps | Confirmed by inspection (`initiatives_routes.py`) | REQ-13–REQ-21 |
| A3 | CAP-B's three onboarding calls (`catalogue`, `repos/select`, `repos/readiness/refresh`) are stable enough in shape and semantics that client-side composition into one pass/fail verdict will not drift silently | Assumed — named risk; revisit with a server-side composite endpoint only if drift appears (OQ-2 resolution) | REQ-07 |
| A4 | No gateflow route, schema, or contract change occurs concurrently with this initiative's delivery window | Assumed — named risk | All REQs |

### Error table (product-normative)

| Situation | Result | Side effects |
|---|---|---|
| Onboarding probe returns `setup_failed`/`probe_failed`/`status_failed`/`out_of_catalogue` | Fleet membership blocked, operator-readable reason shown | 0 partial/half-member state |
| Cross-tenant read attempt on any consumed endpoint | Refused / scoped empty (inherited from gateflow's own tenant scoping) | 0 cross-tenant data exposure |
| Wave-start called with an invalid/blocked precondition (e.g. wave already active for that initiative/wave) | Gateflow's structured error surfaced as-is, run not enqueued | 0 duplicate/zombie runs |
| Run detail requested for an unknown `run_id` | Named not-found state, never a fabricated result | 0 fabricated run data |
| Forge-authorize called on a run not `STOPPED` at an `external-action` node | Structured error surfaced as-is from gateflow, not swallowed or retried silently | 0 silent no-op |
| Checkpoint status queried with a composed ref and no matching run | Named "no run found for this wave" state | 0 fabricated status |
| Checkpoint status requested for a raw PR ref with no checkpoint history | Named not-found state, never a fabricated result | 0 fabricated checkpoint data |
| CAP-F readout composed from incomplete board/GitHub data | Honest gap displayed (e.g. "no Draft PR yet") | 0 fabricated data |
| Valid tenant with no data yet on any list/readout view | Well-formed empty state | 0 fabricated value, 0 404 treated as error |
| Attempted call to a `PLATFORM_ADMIN`-gated route from this console | Never attempted — no UI surface exists for it (D4) | 0 403s by design, not by accident |

### Open questions

All three outline opens are resolved and not carried forward as blocking:

| ID | Resolution |
|---|---|
| OQ-1 | Dropped — the 003 W2 engg spec-lane dogfood role does not carry forward to this initiative |
| OQ-2 | Locked — CAP-B's onboarding-scorecard composition is client-side only this initiative; no gateflow ask |
| OQ-3 | Confirmed — wave order stays as proposed: W0 (CAP-A/B) → W1 (CAP-C) → W2 (CAP-F) → W3 (CAP-G) → W4 (CAP-D/E) |

---

## 3. Functional requirements

### Capabilities

| ID | Capability | Covers |
|---|---|---|
| CAP-A | Identity & access | REQ-01–REQ-02 |
| CAP-B | Fleet onboarding | REQ-03–REQ-08 |
| CAP-C | Wave operations | REQ-09–REQ-12 |
| CAP-F | Initiative & delivery tracking | REQ-13–REQ-21 |
| CAP-G | Metrics & efficacy panel | REQ-22–REQ-25 |
| CAP-D | Checkpoint evidence | REQ-26–REQ-27 |
| CAP-E | Board & tickets | REQ-28–REQ-31 |

### Requirements

| ID | Requirement | Condition | Observable result | Evidence |
|---|---|---|---|---|
| REQ-01 | View tenant detail | Authorized operator opens tenant view | `GET /api/v1/tenants/{tenant_id}` renders; JWT tenant must match path tenant | UI test |
| REQ-02 | Invite a teammate to the tenant | Operator submits invite | `POST /api/v1/tenants/{tenant_id}/users` succeeds, 200 | UI test |
| REQ-03 | Browse + refresh catalogue candidates | Operator opens onboarding flow | `GET .../programme/catalogue` and `POST .../catalogue/refresh` render candidate list | UI test |
| REQ-04 | Connect/re-sync programme meta | Operator connects programme before catalogue browsing | `PUT .../programme/connect`, `GET .../programme/connection` succeed | UI test |
| REQ-05 | Admit a repo to the fleet | Operator selects a candidate | `POST .../programme/repos/select` returns per-repo outcome, rendered in plain language | UI test |
| REQ-06 | Check readiness | Operator triggers readiness refresh | `POST .../programme/repos/readiness/refresh` returns `harness_verified` + `verdict_type` | UI test |
| REQ-07 | Compose single pass/fail onboarding verdict client-side | REQ-05 + REQ-06 responses available | Console renders one pass/fail state, never partial (D7/OQ-2) | UI test + unit test on composition logic |
| REQ-08 | Remove a repo from the fleet | Operator deselects | `POST .../programme/repos/deselect` succeeds | UI test |
| REQ-09 | Start a wave from the UI | Operator starts implement/spec/closeout wave | `POST /api/v1/waves/{lane}/start` enqueues run | UI test, all 3 lanes |
| REQ-10 | List runs with filters | Operator filters fleet activity | `GET /api/v1/runs` returns filtered, tenant-scoped list | UI test |
| REQ-11 | Open run detail + timeline | Operator opens a run | `GET /api/v1/runs/{run_id}` renders stages/events/outcomes/durations/PR number; stop state renders unambiguously as human-checkpoint/failure/complete, never styled as an error when expected | UI test |
| REQ-12 | Authorize a pending forge action | Run `STOPPED` at `external-action` node | `POST /api/v1/runs/{run_id}/forge/authorize` executes; console never self-dispatches | UI test against a real external-action stop |
| REQ-13 | List + detail initiatives | Operator opens initiative list | `GET /api/v1/initiatives[/{id}]` renders composed runs+board data | UI test |
| REQ-14 | Wave map | Operator opens initiative's wave map | `GET .../{id}/waves` renders per-wave status | UI test |
| REQ-15 | Spec-lane readout | Operator opens spec readout | `GET .../{id}/spec` renders | UI test |
| REQ-16 | Implementation readout | Operator opens wave implementation readout | `GET .../waves/{wave_id}/implementation` renders | UI test |
| REQ-17 | Closeout readout | Operator opens wave closeout readout | `GET .../waves/{wave_id}/closeout` renders | UI test |
| REQ-18 | Merge readout | Operator opens wave merge readout | `GET .../waves/{wave_id}/merge` renders | UI test |
| REQ-19 | Completion readout | Operator checks close-readiness | `GET .../{id}/completion` renders | UI test |
| REQ-20 | Closure preview | Operator previews closure | `GET .../{id}/closure` renders pre/post purge lists | UI test |
| REQ-21 | Start initiative closure | Operator triggers closure | `POST /api/v1/initiatives/closure/start` enqueues, 202 | UI test |
| REQ-22 | Legacy stage-duration heatmap | Operator opens metrics view | `GET /api/v1/metrics/runs` renders by node/runner/model | UI test |
| REQ-23 | Skill/spec efficacy view | Tech lead opens efficacy panel | `GET /api/v1/metrics/skill-efficacy` renders, filterable by `model_id`/`prompt_revision` | UI test |
| REQ-24 | Factory effectiveness view | Tech lead/PE opens panel | `GET /api/v1/metrics/factory-effectiveness` renders | UI test |
| REQ-25 | Delivery scorecard view | Programme lead opens scorecard | `GET /api/v1/metrics/delivery-scorecard` renders `as_of`+cumulative+90-day delta | UI test |
| REQ-26 | Live checkpoint status | Operator queries a PR or composed ref | `GET /api/v1/checkpoints/status` renders; composed ref with no run shows named "no run" state | UI test |
| REQ-27 | Checkpoint history | Operator opens history for a PR | `GET /api/v1/checkpoints/history` renders | UI test |
| REQ-28 | List board tickets | Operator opens board view | `GET /api/v1/board/tickets` renders, required `org`/`repo` filters | UI test |
| REQ-29 | Create EPIC/Feature ticket | Operator creates a ticket | `POST /api/v1/board/tickets` succeeds, idempotent on `initiative_id`+`type` | UI test |
| REQ-30 | Update ticket status | Operator changes ticket column/state | `PATCH /api/v1/board/tickets/{ticket_id}/status` succeeds | UI test |
| REQ-31 | Link PR to ticket | Operator links a PR | `POST /api/v1/board/tickets/{ticket_id}/links` succeeds | UI test |

**Implementation notes (non-normative):** Exact component names, BFF route shapes, and client-side state management are engineering's to design in the Draft PRD's implementation/spec stage — not fixed here. The capability boundaries and the exact gateflow route each maps to are product-normative.

---

## 4. Technical Specifications

### Architecture Overview *(design — not product vocabulary)*

```text
Operator browser
      |
      v
gateflow-ops (Mission Control UI + BFF)
      | thin ops-user auth (session -> JWT bridge, already built)
      +--> gateflow HTTP API (CAP-A..G routes, all listed in S3) -- primary, only integration
      +--> GitHub (outbound links only -- Open PR, never a direct write)
      |
      v
Gateflow control plane (001-003, 010/011, 013, 014, 015) -- unchanged orchestration
      |
      v
Worker + live Cursor AgentRunner + RunStore + ForgeClient + InitiativeReadoutService
```

**Rule:** `gateflow-ops` is a consumer and presenter only. It never embeds `PolicyEngine`, `WorkflowEngine`, or `AgentRunner` execution, and it never calls a `PLATFORM_ADMIN`-gated route.

### Integration Points

| Gateflow route group | Ops capability | Auth gate |
|---|---|---|
| `tenant_routes` (`GET/POST /api/v1/tenants/{tenant_id}...`) | CAP-A | `require_tenant_resolved` |
| `catalogue_connection_routes` (`/api/v1/tenants/{tenant_id}/programme/...`) | CAP-B | `require_tenant_resolved` |
| `waves_routes`, `runs_routes`, `forge_routes` | CAP-C | `require_role(TENANT_ADMIN)` |
| `checkpoints_routes` | CAP-D | `require_role(TENANT_ADMIN)` |
| `board_routes` | CAP-E | `require_role(TENANT_ADMIN)` |
| `initiatives_routes` | CAP-F | `require_role(TENANT_ADMIN)` |
| `metrics_routes` | CAP-G | `require_role(TENANT_ADMIN)` |
| `programme_admin_routes`, `agent_catalogue_router` | **Not integrated** | `require_role(PLATFORM_ADMIN)` — out of scope (D4) |

### Security & Privacy

- Every BFF call forwards the operator's session-derived JWT server-side only — never exposed to the browser (existing chassis pattern, unchanged).
- No new PII surface — tenant/user records already exist in gateflow; this initiative reads/writes them through existing endpoints only.
- Forge actions execute only on explicit operator click against `POST /api/v1/runs/{run_id}/forge/authorize` — the console never auto-authorizes or Cursor-dispatches forge skills.
- No `PLATFORM_ADMIN`-gated route is ever called from this console (D4) — enforced by simply not building a UI surface for it, not by a runtime check the console could get wrong.
- Cross-tenant data exposure is prevented entirely by gateflow's existing tenant scoping; this console adds no additional scoping logic of its own to get wrong.

### AI / agent evaluation

Not applicable — Mission Control does not run or evaluate coding agents. It displays agent execution that gateflow's `AgentRunner` already dispatches. Quality bar is UI/BFF correctness against real gateflow responses:

- Manual QA against ≥ 1 real onboarded repo, ≥ 1 real wave reaching an `external-action` stop, and ≥ 1 real initiative with board EPIC + run history.
- No new evaluation harness needed — nothing here generates model output.

---

## 5. Risks & Roadmap

### Delivery waves (product-normative)

| Wave | Intent | Exit REQs |
|---|---|---|
| **W0** | Identity & access + fleet onboarding | REQ-01–REQ-08 |
| **W1** | Wave operations (start/inspect/authorize) | REQ-09–REQ-12 |
| **W2** | Initiative & delivery tracking | REQ-13–REQ-21 |
| **W3** | Metrics & efficacy panel | REQ-22–REQ-25 |
| **W4** | Checkpoint evidence + board/ticket linking | REQ-26–REQ-31 |

### Technical risks

| Risk | Mitigation |
|---|---|
| Run cockpit (CAP-C) will feel thin on narrative for v0 — no readable log narrative or PR-comment deep links exist in gateflow yet | Named, accepted risk; timeline/outcome/duration data is still fully real and useful; richer narrative is explicit fast-follow scope, not silently promised here |
| Client-side onboarding-scorecard composition (REQ-07) could drift from a true pass/fail-only presentation if not centralized | Compose in one shared BFF module, not per-screen; revisit with a server-side composite endpoint only if drift appears in practice (A3) |
| CAP-F readouts depend on board/GitHub state accuracy that gateflow itself does not own | Named limitation, inherited from `InitiativeReadoutService`'s own scope — display what gateflow composes, never invent data (A2) |
| Large feature surface (7 capability areas) risks scope creep relative to a v0 | Wave sequencing ships CAP-A/B first (thin, already-shipped backend) before the larger CAP-F/G areas |
| Concurrent gateflow route/contract changes during this initiative's delivery window | Named risk (A4); impact-map should flag if gateflow ships a breaking change mid-delivery |

### Phased rollout

- **MVP (W0–W1):** identity, onboarding, and the operate loop (start/inspect/authorize waves) — the minimum for an operator to stop using raw API calls.
- **v1 (W2–W3):** initiative/delivery tracking and the efficacy/scorecard panel — the highest-differentiation, zero-backend-gap surfaces.
- **v1.1 (W4):** checkpoint evidence + board/ticket linking — rounds out the console.
- **Later, explicit follow-ups:** the 4 named gateflow backend gaps (fleet-summary endpoint, workflow-pin readiness probe, process-map endpoint, log-pane richness); a platform-admin console; a server-side composite onboarding-scorecard endpoint, only if client-side composition proves unreliable.

---

## 6. Locked decisions reference

### Carried forward from outline (D1–D7)

| ID | Decision |
|---|---|
| D1 | INIT-GATEFLOW-004 retired outright (deleted); zero assumptions carried forward — every feature re-derived from gateflow's live route table |
| D2 | No gateflow backend work this initiative; the 4 named gaps are deferred fast-follow |
| D3 | Thin ops-user identity — single capability tier, no roles/RBAC in v0 |
| D4 | `PLATFORM_ADMIN`-gated routes are out of scope — distinct future persona/initiative |
| D5 | Process stays display-only; `prayog-skills` pin remains SSOT |
| D6 | Initiative & delivery tracking (CAP-F) is in scope as a first-class feature area |
| D7 | Onboarding scorecard composition is client-side, built directly on INIT-GATEFLOW-013's existing endpoints; no new composite gateflow endpoint requested this initiative |

### Resolved this Draft PRD (outline OQ-1–OQ-3)

| ID | Resolution |
|---|---|
| OQ-1 | Dropped — the 003 W2 engg spec-lane dogfood role does not carry forward to this initiative |
| OQ-2 | Locked — client-side composition only, no future-ask flagged; revisit only if drift appears (A3) |
| OQ-3 | Confirmed — wave order stays W0 (CAP-A/B) → W1 (CAP-C) → W2 (CAP-F) → W3 (CAP-G) → W4 (CAP-D/E) |

---

## 7. Next steps

1. Run `/validate-requirements` against this Draft PRD.
2. Impact map → programme sign-off → gateflow-ops implementation waves per §5.
3. Do **not** implement `gateflow-ops` code from this Draft PRD alone — follow the full detailed-design process (spec → feasibility → technical review → plan → waves).
4. No prayog-meta vision/ADR supersession is required — this initiative operationalizes the existing vision, it does not change it.

---

## Appendix A — Data model changes

| Change | Notes |
|---|---|
| No new ops-side data model this initiative | Every capability reads/writes through gateflow's existing persistence (`tenant_repos`, `runs`, `stages`, `run_events`, board tickets, checkpoint evidence). `gateflow-ops`'s existing thin session/auth model is unchanged |
| No new gateflow tables or columns | Confirmed non-goal (D2) |

## Appendix B — Target capability surface (illustrative — engineering owns exact BFF routes)

| Capability | Gateflow routes consumed |
|---|---|
| CAP-A | `GET/POST /api/v1/tenants/{tenant_id}...` |
| CAP-B | `PUT/GET/POST /api/v1/tenants/{tenant_id}/programme/...` |
| CAP-C | `POST /api/v1/waves/{lane}/start`, `GET /api/v1/runs[...]`, `POST /api/v1/runs/{run_id}/forge/authorize` |
| CAP-D | `GET /api/v1/checkpoints/{status,history}` |
| CAP-E | `GET/POST/PATCH /api/v1/board/tickets...` |
| CAP-F | `GET/POST /api/v1/initiatives...` |
| CAP-G | `GET /api/v1/metrics/{runs,skill-efficacy,factory-effectiveness,delivery-scorecard}` |

Exact BFF route shapes, component structure, and client state management are engineering's to design in the next stage — not fixed here. The capability-to-gateflow-route mapping is product-normative; the BFF's own internal shape is not.

## Appendix C — Traceability

| Outline section | PRD section |
|---|---|
| §1 Problem statement | §1 Executive Summary |
| §2 Proposed solution | §1 Executive Summary |
| §3 Locked decisions D1–D7 | §6 Locked decisions reference |
| §4 Users and JTBD | §2 User Personas |
| §5 Scope — in | §2 User Stories; §3 Requirements |
| §6 Scope — out | §2 Non-Goals |
| §7 Capability walkthroughs | §2 User Stories acceptance criteria |
| §8 Delivery waves | §5 Delivery waves |
| §9 Success criteria | §1 Success Criteria |
| §10 Dependencies and non-goals for partners | Document control 'Depends on' + §6 Locked decisions |
| §11 Risks | §5 Technical risks |
| §12 Open questions OQ-1–OQ-3 | §2 Open questions (resolved); §6 Resolved this Draft PRD |
| §13 Next steps | §7 Next steps |

---

## Exit gate

> **Exit:** `gateflow-ops` delivers onboarding, fleet operations, wave lifecycle with forge authorization, full initiative delivery tracking, checkpoint evidence, board/ticket linking, and the efficacy/scorecard panel — entirely against gateflow's existing, live API surface, verified end-to-end against ≥ 1 real onboarded repo, wave, and initiative. Zero gateflow route, schema, or contract changes ship this initiative; the 4 named backend gaps and the platform-admin console remain explicit non-goals, tracked as future initiatives.

---

## References

- Outline: [prd/INIT-GATEFLOW-016-outline.md](./INIT-GATEFLOW-016-outline.md)
- Vision: [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md) §9 maturity · §10 H2 lift-on-metrics
- System identity: [planning/prayog-system-product-brief.md](../planning/prayog-system-product-brief.md)
- Delivered dependencies: [prd/INIT-GATEFLOW-013.md](./INIT-GATEFLOW-013.md), [prd/INIT-GATEFLOW-014.md](./INIT-GATEFLOW-014.md), [prd/INIT-GATEFLOW-015.md](./INIT-GATEFLOW-015.md)
- Predecessor (retired): INIT-GATEFLOW-004 — deleted 2026-08-12, superseded outright
- Codebase grounding: `drivestream-lab/gateflow` — `src/api/v1/{metrics_routes,runs_routes,waves_routes,forge_routes,checkpoints_routes,board_routes,tenant_routes,catalogue_connection_routes,initiatives_routes}.py`; `drivestream-lab/gateflow-ops` — `app/(auth)/login/page.tsx`, `app/(dashboard)/page.tsx`, `lib/{auth,bff,upstream-fetch}.ts` (confirmed chassis-only, no product screens)
