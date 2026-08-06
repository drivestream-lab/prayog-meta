# INIT-GATEFLOW-002 — API-triggered waves & platform readiness

**Status:** draft PRD · **Author:** programme PM · **Date:** 2026-07-24  
**Outline:** [INIT-GATEFLOW-002-outline](./INIT-GATEFLOW-002-outline.md)  
**Vision:** [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md)  
**Predecessor:** [INIT-GATEFLOW-001](./INIT-GATEFLOW-001.md) (Horizon 1 control plane — **delivered** in gateflow repo)  
**Component:** GATEFLOW · **Type:** platform / delivery control plane

> **Draft PRD** — builds on the **delivered** INIT-GATEFLOW-001 control plane in
> the gateflow repo `(Source: User-confirmed)`. Discovery decisions locked in
> outline §11 and programme Resolutions (2026-07-24). Engineering detail routes
> to impact map and gateflow spec PR. Dogfood programme work is **not** a
> driver of this INIT.

---

## Document control

| Field | Value |
|-------|-------|
| Initiative ID | INIT-GATEFLOW-002 |
| Programme | prayog |
| Primary repo | drivestream-lab/gateflow |
| Related (later) | drivestream-lab/gateflow-ops (consumes APIs; UI out of scope) |
| Supporting repos | prayog-meta, prayog-skills, launchpad |
| Depends on | INIT-GATEFLOW-001 control plane **delivered** in gateflow repo (run engine, contract stops, Cursor path, ForgeClient, GitHub comments baseline, pin **`v0.5.0-rc.2`**) `(Source: User-confirmed)` |
| Skills pin | prayog-skills **`v0.5.0-rc.2`** (`dispatch`) — unchanged consumer |
| Primary delivery repo | **gateflow only** |
| Target users | PE (wave start, inspect runs), tech lead (audit), programme sponsor (metrics), future ops / next INIT (board APIs) |

---

## 1. Executive Summary

### Problem Statement

INIT-GATEFLOW-001 is **delivered** in the gateflow repo `(Source: User-confirmed)`,
but operators still lack a first-class **API to start a wave**, **per-skill
runner and model choice**, a rich enough **run/metrics API** for a future ops
console, and **board operation APIs** that work when Gateflow is deployed without
laptop GitHub tooling.

### Proposed Solution

**INIT-GATEFLOW-002** makes Gateflow **API-first**: authorize wave runs via
programme-authenticated API only (not board column changes; not label trigger),
run orchestrated skills with **per-`workflow_node` runner and model** from
gateflow repo config, **open or update a PR at run start** so progress comments
have a durable thread through stages (same naming on success and failure
paths), deepen **run and metrics APIs**, and ship **wider board operation APIs**
(status, link PR, create/list tickets as dumb forge primitives) that the wave
engine does **not** call. Cursor and GitHub comments stay live; OpenCode, Claude
Code, Slack, and Teams remain honest stubs. Deployed GitHub writes use
**ForgeClient** only (**FR-26a**); laptop board-seed may still use local `gh`
under programme policy (**FR-26b**).

Gateflow still **does not write application code**, **does not merge PRs**,
**does not replace the board-seed skill**, and **does not move board tickets**
as part of the wave workflow.

### Success Criteria

| KPI | Target | Measurement |
|-----|--------|-------------|
| **API wave start** | Authenticated wave-start API starts a run when preconditions pass; clear structured error when they fail | Integration tests + RunStore |
| **Contract stop compliance** | 100% stops at `human-checkpoint`, `external-action`, `decision`, `terminal` — zero auto-transitions | Run event audit |
| **Per-skill runner/model** | ≥ 2 orchestrated nodes can resolve different configured `model_profile` values on Cursor path; fields persisted | Config fixtures + RunStore |
| **PR thread for every run** | 100% of started runs open/update a PR at **run start** (same naming conventions); stage and terminal comments posted on that PR | ForgeClient audit + Notifier |
| **Stub honesty** | Selecting stub runner or stub notifier in config that would be required for the run **blocks at start** — no silent Cursor/GitHub substitute | PolicyEngine / config validation tests |
| **API readiness** | Documented run list/detail, metrics aggregates, and board APIs (status, link PR, create/list tickets) callable with programme service token | API contract tests + docs |
| **Workflow/board separation** | Completing a wave never calls board APIs (including no auto-link of PR to ticket) | Integration audit |
| **Deployed forge path** | Production Gateflow runtime has **zero** dependency on `gh` CLI for PRs, comments, or board APIs | Spec-defined production-path verification |

Numeric cycle-time baselines remain `[TBD]` — dogfood calibration deferred.

### Scope boundary with INIT-GATEFLOW-001

| Topic | INIT-GATEFLOW-002 rule |
|-------|------------------------|
| Wave start | **API only** — product path `(Source: User-confirmed)` |
| INIT-001 FR-2 (label trigger) | **Removed / superseded** for programmes on 002 — label is not a supported start mechanism `(Source: User-confirmed)` |
| INIT-001 PRD | Left **unchanged** as completed delivery reference; this Scope boundary is supersession SSOT `(Source: User-confirmed)` |
| Other INIT-001 FRs | FR-1, FR-3–FR-8, FR-10–FR-12, FR-14 remain in force unless noted; FR-13 superseded by FR-16; FR-9 live notifier extended by FR-23 |

---

## 2. User Experience & Functionality

### User Personas

| Persona | Role | Primary need |
|---------|------|--------------|
| **PE** | Wave execution owner | Start wave N via API; trust contract stops; see PR + comments |
| **PE (board setup)** | Board seeder | Laptop `gh` OK locally; deployed apply via Gateflow/ForgeClient |
| **Tech lead** | Gate / constitution owner | Audit runner × model per skill; no auto gate writes |
| **Programme sponsor** | Learning owner | Stage metrics by skill / runner / model |
| **Future ops / next INIT** | Board lifecycle + UI | Stable board and run APIs without rewriting the engine |

### Target experience

```text
Board already seeded (board-seed skill — manual)
        │
        ▼
API: start wave N  (ticket id OR initiative + wave id)
        │
        ▼
Validate config (block if stub runner/notifier required)
  → open/update PR at run start (same naming success|failure paths)
  → run orchestrated skills (per-node runner + model)
  → Notifier comments on that PR through stages
  → stop at contract stop; record metrics
        │
        ▼
Human verifies / merges
  → (optional) caller uses board APIs for status / link PR
        │
        ▼
API: start next wave
```

Status API / RunStore remains available as a supplementary channel; mid-run
GitHub visibility is the run PR thread `(Source: User-confirmed)`.

### User Stories & Acceptance Criteria

#### US-1 — PE starts a wave via API

**As a** PE, **I want** to start wave N with an authenticated API call **so that**
Gateflow runs orchestrated skills without board column changes or label trigger.

**Acceptance criteria:**

- [ ] Wave start is an authenticated Gateflow API (programme service token)
- [ ] Request may identify the wave by **issue/ticket id** **or** **initiative id +
  wave id** (e.g. `W0`); if **both** are provided they **must agree** or the API
  rejects `(Source: User-confirmed)`
- [ ] API is the **only** supported start path for this INIT; INIT-001 label
  trigger is **removed** for 002 programmes `(Source: User-confirmed)`
- [ ] Gateflow does not start a run because a board column changed
- [ ] When [wave-run preconditions](#wave-run-preconditions) fail, API returns a
  structured error with reason; no dispatch; no silent success
- [ ] When preconditions pass, a RunStore run is created and work is enqueued

#### US-2 — PE observes progress on a PR for every run

**As a** PE, **I want** every wave run to open or update a PR at **run start**
with structured comments through stages **so that** I have a durable review
thread without waiting for terminal outcome.

**Acceptance criteria:**

- [ ] At **run start**, ForgeClient opens or updates a PR (branch + PR) using the
  **same naming conventions** for runs that later succeed or fail
  `(Source: User-confirmed)`
- [ ] Notifier posts structured events on that PR (stage started, stage completed,
  stop, failed) via ForgeClient — not `gh`
- [ ] Mid-run GitHub visibility is that PR thread; status API is supplementary
  `(Source: User-confirmed)`
- [ ] Human merges; Gateflow never auto-merges
- [ ] Wave workflow does **not** auto-link the PR to the wave ticket; linking is
  via board API only `(Source: User-confirmed)`

#### US-3 — PE chooses runner and model per orchestrated skill

**As a** PE, **I want** programme config to set runner and model per orchestrated
`workflow_node` **so that** e.g. `loop-spec` can differ from `verify`.

**Acceptance criteria:**

- [ ] `model.overrides` (and runner override fields) in **gateflow repo config**
  resolve per `workflow_node` for `dispatch: orchestrated` only
  `(Source: User-confirmed)`
- [ ] Unset nodes use programme `runner.default` + `model.profiles.default`
- [ ] Resolved `runner`, `model_profile`, `model_id`, `model_provider` persisted on
  every orchestrated stage
- [ ] Does not change which nodes are dispatch-eligible (contract `dispatch` SSOT)

#### US-4 — Tech lead trusts stubs and contract stops

**As a** tech lead, **I want** stub agents/notifiers to fail closed and human
gates never auto-passed **so that** production behavior is auditable.

**Acceptance criteria:**

- [ ] Cursor AgentRunner implemented; OpenCode and Claude Code registered as stubs
- [ ] GitHub comment Notifier implemented; Slack and Teams registered as stubs
- [ ] If config selects a stub runner or stub notifier that the run would require,
  Gateflow **blocks the whole run at start** with a clear error — no mid-run
  surprise and no silent fallback `(Source: User-confirmed)`
- [ ] `notifier.default` (and any per-run override) must resolve to an
  **implemented** backend; unimplemented default blocks at start
- [ ] Zero auto-transitions on human-checkpoint / external-action / decision /
  terminal; zero gate-approval label writes; zero auto-merge

#### US-5 — Sponsor and ops consume APIs

**As a** programme sponsor or future ops builder, **I want** list/filter runs,
full timelines, metrics by node/runner/model, and board operation APIs **so that**
the next INIT and gateflow-ops can build UX without rewriting Gateflow.

**Acceptance criteria:**

- [ ] Run list/filter and run detail (stage/event timeline) APIs documented and
  authenticated
- [ ] Metrics aggregates queryable by `workflow_node`, `runner`, `model_id`
- [ ] Board APIs support: update ticket/column status; link PR to ticket;
  **create and list tickets** as **dumb forge primitives** (caller supplies
  fields; no WorkManifest/governance parsing in Gateflow)
  `(Source: User-confirmed)`
- [ ] Wave workflow never invokes board APIs on start/finish
- [ ] No gateflow-ops UI required for this INIT exit

#### US-6 — PE seeds board with dual transport awareness

**As a** PE, **I want** board seeding to keep using the board-seed skill for
*what* to create, while apply transport is **conditional** **so that** laptop
work can use `gh` and deployed Gateflow uses ForgeClient.

**Acceptance criteria:**

- [ ] This INIT does **not** replace or delete the board-seed skill
- [ ] Gateflow board/forge APIs in deployment use ForgeClient only (FR-26a; no
  `gh` in production path)
- [ ] Programme / skills policy may allow laptop `gh` for board-seed apply
  (FR-26b) `(Source: User-confirmed)`
- [ ] Skills follow-on may wire board-seed apply to call Gateflow board APIs when
  off-laptop — not required to ship inside this INIT’s skills repo delivery

### Wave-run preconditions {#wave-run-preconditions}

Authoritative checklist before PolicyEngine may dispatch after **API** trigger
(extends INIT-GATEFLOW-001 preconditions; trigger source is API):

| # | Precondition | Source |
|---|--------------|--------|
| 1 | Request authenticated with programme service token | FR-15 |
| 2 | Wave identity resolved (ticket id **or** initiative + wave id; if both, must agree) | FR-15 |
| 3 | Latest handoff envelope readable per programme config | INIT-001 FR-4 |
| 4 | `handoff.contract` matches installed contract | handoff-envelope.md |
| 5 | Resolved next node is `type: skill` with `dispatch: orchestrated` | pin `v0.5.0-rc.2` |
| 6 | `handoff.human_checkpoint` not blocking | handoff rules |
| 7 | No active run for same wave identity / PR/issue scope | concurrent reject |
| 8 | Handoff has no unresolved blockers | handoff envelope |
| 9 | `handoff.stage` in wave lane (post-`board-seed`) | workflow pin |
| 10 | Resolved runner + notifier for this run are **implemented** (not stubs) | FR-18 |
| 11 | Resolved runner/model config for required orchestrated nodes is **valid** (no unknown override / missing profile / unresolvable model) | FR-16 |

Failure of any precondition: structured API error; no dispatch.

### Functional Requirements (FR) — with Acceptance Criteria

| ID | Requirement | Acceptance criteria |
|----|-------------|---------------------|
| **FR-15** | API wave trigger | Authenticated wave-start endpoint; accepts **either** ticket id **or** initiative_id + wave_id; if both provided they **must agree** else reject; returns `run_id` on accept; rejects with structured reason on precondition failure; does not infer intent from board columns; label trigger not supported |
| **FR-16** | Per-orchestrated-node runner + model | Gateflow repo config resolves runner + model profile per `workflow_node` for `dispatch: orchestrated`; default fallback; persist four model/runner fields on stages; no hardcoded node allowlists |
| **FR-17** | Multi AgentRunner adapters | Cursor **implemented**; OpenCode and Claude Code **stubs** registered by id (by W0/W1); selecting stub for a required runner blocks run at start (FR-18) |
| **FR-18** | Fail-closed stubs | Before enqueue/dispatch, validate that every runner and notifier the run will need is implemented; `notifier.default` (and overrides) must resolve to an **implemented** backend; if any required slot is stub → **block entire run**; never silent fallback; unused stub in registry is OK if not selected |
| **FR-19** | PR thread from run start | At **run start**, create/update branch + PR via ForgeClient using **same naming conventions** for runs that later succeed or fail; post Notifier comments on that PR through stages and terminal outcomes; no auto-merge; no workflow auto-link to ticket; status API supplementary for mid-run visibility `(Source: User-confirmed)` |
| **FR-20** | Ops-ready run APIs | `GET` list/filter runs; `GET` run detail with full stage/event timeline; programme service token auth; stable JSON contract documented for gateflow-ops |
| **FR-21** | Ops-ready metrics APIs | Aggregates by `workflow_node`, `runner`, `model_id` (p50/p95 duration and outcome rates where data exists); extends INIT-001 `GET /metrics/runs` |
| **FR-22** | Multi-stage metrics | Emit events for API trigger, each orchestrated stage, contract stops, findings loops, and observable handoff/gate signals per vision §11; queryable in RunStore |
| **FR-23** | Notifier slot with stubs | GitHub PR comments **live** via ForgeClient; Slack and Teams **stubs**; event schema unchanged from INIT-001 FR-9; stub selection fail-closed per FR-18 |
| **FR-24** | Wider board operation APIs | Authenticated APIs to: update ticket/column status; link PR to ticket; **create ticket**; **list tickets** (filters `[TBD in spec]`). **Dumb forge primitives** — caller supplies fields; no WorkManifest/governance parsing `(Source: User-confirmed)`. Creates are **idempotent** for **EPIC** and **Feature** using `initiative_id` + type (`EPIC`\|`Feature`); optional client `Idempotency-Key` for multi-step create; other key shapes `[TBD in spec]`. **Partial-failure** responses when multi-step create fails mid-way. Via ForgeClient in deployment. **Wave workflow must not call these** on start/finish |
| **FR-25** | Deployed forge path (no `gh`) | Production Gateflow runtime performs all forge/board writes through ForgeClient (App installation token preferred). Production path must not depend on `gh`. *Implementation Note:* how deploy/CI proves this is specified in the gateflow spec — not a product AC wording. |
| **FR-26a** | Deployed board/forge transport | Gateflow-deployed board and forge apply uses ForgeClient only (testable runtime rule) |
| **FR-26b** | Laptop board-seed transport policy | Programme/skills policy: laptop board-seed apply may use `gh`; does not replace board-seed skill judgment; not enforced inside Gateflow runtime |

**Inherited (unless noted):** INIT-GATEFLOW-001 FR-1, FR-3–FR-8, FR-10–FR-12, FR-14 remain in force. **FR-2 (label trigger) is superseded / removed** for INIT-GATEFLOW-002 programmes. FR-13 is **superseded** by FR-16. FR-9 remains the live GitHub Notifier; FR-23 extends the slot with stubs.

### Error Handling

| Failure | System response | PE visibility |
|---------|-----------------|---------------|
| Missing/invalid programme token | 401; no run | API error body |
| Unresolvable wave identity | 400; no run | Structured reason |
| Both identities provided but disagree | 400; no run | Structured reason |
| Precondition failure | 409/422; no run | Structured reason list |
| Stub runner/notifier required | 422; no run | Names stub id + config key |
| Active concurrent run | Reject; no second run | API error + optional existing `run_id` |
| AgentRunner timeout/crash | Stop run; `failed`; comments on run PR | PR thread + RunStore |
| ForgeClient comment failure mid-run | Mark `notify_pending` if delivery cannot complete; run continues per policy | RunStore flag; status API reconstructs |
| Terminal / start PR open or update failure | Preserve run outcome in RunStore; escalate `notify_pending`; do not invent success visibility | Status API must reconstruct run; PE alert via ops path `[TBD in spec]` |
| Invalid per-node runner/model config (unknown override node, missing profile, unresolvable model) | Block at start; no dispatch (FR-16 / FR-18) | Structured API error with config key / node id |
| Board API bad payload | 400; no partial silent success | API error |
| Board API forge failure | Error with guidance; no silent success | API error |
| Board multi-step create partial failure | Return which resources created vs failed; support idempotent retry | Structured partial-failure body |
| Metrics / run list invalid filter | 400 | Structured reason |
| Metrics / run read failure (store unavailable) | 503 | API error |
| Unused stub present in registry | Allowed | N/A — only selected stubs block |
| `gh` invoked in production path | Forbidden — must not occur | Deploy/spec verification |

### Non-Goals

| Non-goal | Rationale |
|----------|-----------|
| Board column as wave **trigger** | Deferred; API authorize only |
| Label trigger (INIT-001 FR-2) | **Removed** for 002 programmes |
| Wave engine auto board status / auto-link PR | Board APIs are separate; caller owns lifecycle |
| Replace board-seed skill | Skill owns *what* to seed |
| WorkManifest/governance parsing in Gateflow board APIs | Dumb primitives only |
| gateflow-ops UI / BFF pages | API readiness only |
| Working OpenCode / Claude Code / Slack / Teams | Stubs only |
| Dogfood programme as INIT driver | Explicitly deferred |
| Graphify / ToolProvider beyond `none` | Parked |
| LiteLLM / model gateway | Later horizon |
| Auto-merge; gate-approval label writes | Contract invariants |
| Redefine delivery process in Gateflow | prayog-skills SSOT |
| Meta/harness shared config keys for runner/model | Stay in gateflow repo config |

### Product Principles

1. **API authorize** — explicit programme call starts waves; not board or label inference  
2. **Workflow SSOT** — `dispatch` and node `type` from pinned prayog-skills  
3. **Slots with honest stubs** — fail closed; no silent substitutes  
4. **Forge in production = ForgeClient** — laptop `gh` OK only off the deploy path (FR-26b)  
5. **Board APIs ≠ wave engine** — ready for next INIT; unused by run lifecycle  
6. **PR thread from run start** — durable comments through stages  
7. **Measure to improve** — stage metrics with runner × model dimensions  

---

## 3. AI System Requirements

Gateflow orchestrates coding agents; it is not an LLM product itself.

### Tool & Runner Requirements

| Slot | This INIT | Contract |
|------|-----------|----------|
| **AgentRunner** | Cursor **live**; OpenCode **stub**; Claude Code **stub** | Same I/O as INIT-001; selection by resolved `runner` id |
| **Notifier** | GitHub comments **live**; Slack **stub**; Teams **stub** | Structured run events; GitHub via ForgeClient |
| **ToolProvider** | `none` (unchanged) | No Graphify/MCP |
| **ModelGateway** | Cursor-native (no proxy) | Profiles from gateflow repo config; per-node overrides via FR-16 |
| **ForgeClient** | Required for all deployed GitHub writes | PRs, comments, board APIs; no `gh` in production |

### Evaluation Strategy

| Dimension | Method | Pass threshold |
|-----------|--------|----------------|
| **API trigger** | Contract tests: valid/invalid identity, auth, preconditions, disagreeing dual identity | 100% expected status codes |
| **Per-node model resolution** | Fixtures with ≥ 2 orchestrated overrides | Correct runner/profile on RunStore stages |
| **Stub fail-closed** | Config with `runner: opencode` or notifier Slack as default | Run blocked at start; 0 dispatches |
| **PR from run start** | Start run; assert PR exists before first orchestrated stage completes | PR + stage comments present |
| **Board API isolation** | Complete wave run; assert zero board API side effects from workflow | 0 board mutations from run worker |
| **No `gh` in deploy path** | Spec-defined production-path verification | Fail if production forge path depends on `gh` |
| **Contract stops** | Inherit INIT-001 policy tests | 0 auto human-gate transitions |

---

## 4. Technical Specifications

### Architecture Overview

```text
                    ┌─────────────────────────────────────┐
  PE / tools ──────►│  Gateflow HTTP API                  │
                    │  • wave-start (FR-15)               │
                    │  • GET runs / metrics (FR-20/21)     │
                    │  • Board ops (FR-24) — separate      │
                    └──────────────┬──────────────────────┘
                                   │ enqueue (wave start only)
                                   ▼
                         Async worker + PolicyEngine
                                   │
              ┌────────────────────┼────────────────────┐
              ▼                    ▼                    ▼
        AgentRunner          ForgeClient           Notifier
        cursor live          (no gh in deploy)     GitHub live
        opencode stub        PRs + board APIs      Slack/Teams stub
        claude stub
```

**Pluggability rule (unchanged):** no slot embeds workflow node ids, gate label
strings, or dispatch allowlists — pinned contract + programme config only.

**Board APIs** share ForgeClient but are **not** invoked by the wave worker on
run lifecycle events.

### Integration Points

| Integration | Direction | Mechanism |
|-------------|-----------|-----------|
| **Clients** | Inbound | HTTP API + programme service token |
| **GitHub** | Outbound | ForgeClient (App token); PRs, comments, board ops |
| **prayog-skills** | Read | Pin `v0.5.0-rc.2` workflow + contract + handoff |
| **launchpad** | Pre-dispatch | Harness sync on worker (inherit INIT-001) |
| **PostgreSQL** | Read/write | RunStore + job queue |
| **Cursor SDK** | Outbound | Live AgentRunner |
| **gateflow-ops** | Future consumer | Run/metrics/board APIs — UI out of scope |
| **Laptop `gh`** | Off deploy path only | Board-seed local apply (FR-26b); never production Gateflow |

### Programme Config (gateflow repo)

Stays in **gateflow repository** (not meta/harness) `(Source: User-confirmed)`.

| Key | Purpose | Example / default |
|-----|---------|-------------------|
| `runner.default` | Default AgentRunner | `cursor` |
| `model.profiles` | Named profile → model | `default: cursor/auto` |
| `model.overrides` | Per-`workflow_node` profile and/or runner | e.g. `loop-spec: { profile: heavy }` |
| `notifier.default` | Default notifier — must be implemented | `github_comment` |
| `pr.branch_prefix` / naming | Shared success+failure PR naming | Programme convention `[TBD in spec]` |
| `handoff.*` | Inherit INIT-001 | unchanged |
| `retry.findings_budget` | Inherit INIT-001 | `3` |
| `metrics.retention_days` | Inherit INIT-001 | `90` |

### Security & Privacy

| Concern | Requirement |
|---------|-------------|
| **API auth** | Programme service token; network boundary; no per-user RBAC in this INIT |
| **GitHub auth (deployed)** | App installation token via ForgeClient; PAT dev-only if needed |
| **No `gh` in production** | Deploy path must not shell to `gh` |
| **Board API abuse** | Same token scope; audit log of board mutations separate from wave run events |
| **Human gates** | Zero automated gate-approval label writes; zero auto-merge |
| **Secrets** | App key, Postgres, deploy keys, tokens in secret store — not in repo |

### Repositories

| Repo | This INIT deliverable |
|------|------------------------|
| **gateflow** | API trigger; per-node runner/model; adapter stubs; PR from run start; run/metrics APIs; board APIs; ForgeClient-only deploy path |
| **gateflow-ops** | Out of scope (consumer later) |
| **prayog-skills** | No required delivery in this INIT; optional later wiring of board-seed → Gateflow board APIs |
| **prayog-meta** | This PRD + impact map |

---

## 5. Risks & Roadmap

### Phased Rollout

| Phase | Scope | Entry |
|-------|-------|-------|
| **W0** | API trigger skeleton + run list/detail; register AgentRunner/Notifier stubs; config validation for stubs | Draft PRD + impact map approved |
| **W1** | Per-node runner/model; PR from run start + GitHub notifier; metrics APIs; Cursor-only happy path | W0 merged |
| **W2** | Wider board APIs (status, link, create, list) via ForgeClient; deploy `gh`-free verification | W1 merged |

Exact wave split may adjust in gateflow spec PR; product scope above is normative.

### Dependencies

| Dependency | Assumption |
|------------|------------|
| INIT-GATEFLOW-001 | **Delivered** in gateflow repo `(Source: User-confirmed)` |
| prayog-skills `v0.5.0-rc.2` | `dispatch` field present |
| GitHub App | Installation on programme repos for ForgeClient |
| PostgreSQL | RunStore |
| Cursor SDK | Live AgentRunner |

### Assumptions

| ID | Assumption | Status | Dependent FRs |
|----|------------|--------|---------------|
| A1 | INIT-GATEFLOW-001 control plane is delivered in the gateflow repo | Confirmed `(Source: User-confirmed)` | FR-15–FR-26a |
| A2 | GitHub App installation token available for ForgeClient in deployment | Confirmed (programme) | FR-19, FR-24, FR-25, FR-26a |
| A3 | App permissions suffice for PR create/comment and board create/list/status/link (including Projects where used) | Pending PE confirm in spec | FR-24 |
| A4 | Programme service token authenticates wave-start and ops/board APIs | Confirmed (inherits INIT-001 model) | FR-15, FR-20, FR-21, FR-24 |
| A5 | Board-seed skill remains SSOT for *what* to seed; Gateflow board APIs are dumb apply primitives | Confirmed `(Source: User-confirmed)` | FR-24, FR-26b |

### Risk Register

| Risk | Likelihood | Impact | Mitigation | Owner |
|------|------------|--------|------------|-------|
| Board API scope (create/list) larger than expected | Medium | Medium | Spec narrows filters/fields; keep skill judgment out of Gateflow | PE |
| Dual-transport confusion (`gh` vs ForgeClient) | Medium | High | Document FR-26a/b; production path forbids `gh` | PE + PM |
| Stub fail-closed too strict for partial configs | Low | Medium | Validate only **required** slots for the run | Eng |
| Always-open PR noise on failures | Medium | Low | Same naming; clear failed outcome in title/body via config templates | PE |

### Decisions (resolved) {#decisions-resolved}

| # | Decision | Resolution |
|---|----------|------------|
| 1 | Wave start path | **API only**; INIT-001 label trigger **removed** for 002 `(Source: User-confirmed)` |
| 2 | Wave identity | **Either** ticket id or initiative + wave id; if both, must agree `(Source: User-confirmed)` |
| 3 | Failure PR naming | **Same conventions as success** `(Source: User-confirmed)` |
| 4 | Board API MVP | **Wider:** status, link PR, create, list as **dumb primitives** `(Source: User-confirmed)` |
| 5 | Config location | **gateflow repo config** `(Source: User-confirmed)` |
| 6 | Board-seed transport | **FR-26a** ForgeClient deploy; **FR-26b** laptop `gh` policy `(Source: User-confirmed)` |
| 7 | Stub selection | **Block whole run at start** if required slot is stub `(Source: User-confirmed)` |
| 8 | Auto-link PR from workflow | **No** — board API only `(Source: User-confirmed)` |
| 9 | PR comment timeline | Open/update PR at **run start**; comments through stages `(Source: User-confirmed)` |
| 10 | INIT-001 readiness | **Delivered** in gateflow repo; 002 proceeds on top `(Source: User-confirmed)` |

### Open Questions

1. Exact HTTP paths and request/response schemas for FR-15 / FR-20 / FR-21 / FR-24 — gateflow spec PR.
2. PR title/body templates and `pr.branch_prefix` defaults — PE confirm in spec.
3. Board API list filters (by initiative label, project, state) — PE confirm in spec.
4. ~~Label trigger legacy vs deprecated~~ — **Resolved:** removed for 002 (Decision #1).
5. Assumption A3 App/Projects permission matrix — PE confirm in spec.
6. PE alert channel when run-start / terminal PR open or update fails (beyond status API + `notify_pending`) — PE confirm in spec.

---

## Appendix A — Traceability

| Outline section | PRD section |
|-----------------|-------------|
| §1–2 problem / solution | §1 Executive Summary |
| §3 target experience | §2 Target experience |
| §4 laptop vs deployed | FR-25, FR-26a/b; US-6 |
| §5 personas / JTBD | §2 Personas + User Stories |
| §6.1–6.9 features | FR-15–FR-26b; US-1–US-6 |
| §7 non-goals | §2 Non-Goals |
| §10 success | §1 Success Criteria |
| §11 resolved OQs | §5 Decisions |
| Resolution VF-* | §5 Decisions; Assumptions; Error Handling |

## Appendix B — Process next steps

| Step | Owner | Artifact |
|------|-------|----------|
| 1 | PM / sponsor | Review this Draft PRD (post-resolution updates) |
| 2 | PE | `/validate-requirements` incremental vs prior Validation-Report |
| 3 | PE | `/prd-impact-map` → **gateflow** primary |
| 4 | PE | Gateflow spec PR (API schemas, PR naming, board API fields) |
| 5 | PE | Delivery waves W0–W2 per §5 |

## Appendix C — One-sentence product

> After the board is seeded, an API call starts a wave; Gateflow opens a PR at
> run start, runs orchestrated skills with per-skill runner and model settings,
> comments through stages, records metrics, and exposes run, metrics, and board
> APIs — using ForgeClient when deployed — while humans merge and the next
> initiative owns board lifecycle UX.
