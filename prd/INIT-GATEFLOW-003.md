# INIT-GATEFLOW-003 — Live Cursor AgentRunner

**Status:** draft PRD · **Author:** programme PM · **Date:** 2026-07-24  
**Outline:** [INIT-GATEFLOW-003-outline](./INIT-GATEFLOW-003-outline.md) (**synced**)  
**Vision:** [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md)  
**Predecessors:** [INIT-GATEFLOW-001](./INIT-GATEFLOW-001.md), [INIT-GATEFLOW-002](./INIT-GATEFLOW-002.md) (both **finished / delivered**)  
**Component:** GATEFLOW · **Type:** platform / delivery control plane

> **Draft PRD** — builds on finished INIT-GATEFLOW-001 and INIT-GATEFLOW-002.
> Discovery decisions locked in outline §11 and Discovery (2026-07-24).
> Engineering detail routes to impact map and gateflow spec PR. Dogfood
> programme work is **not** a driver of this INIT.
>
> **Dispatch rule:** any pinned skill node with `dispatch: orchestrated` is
> **triggered** by Gateflow and flows the delivery workflow as defined in the
> pin and as configured (runner/model). Live Cursor is the first live runner
> plug. **Prove-it uses two scenarios** with **exact existing skill names**:
> (1) pre–Gate 2 eng lane, (2) coding cycle including `verify` — to show
> automation works across multiple paths (intended defaults `cursor` / `auto`).

---

## Document control

| Field | Value |
|-------|-------|
| Initiative ID | INIT-GATEFLOW-003 |
| Programme | prayog |
| Primary repo | drivestream-lab/gateflow |
| Related (later) | drivestream-lab/gateflow-ops (consumer only; UI out of scope) |
| Supporting repos | prayog-meta, prayog-skills (Scenario A pin `dispatch` supporting delivery), launchpad (existing sync only — no Launchpad product work) |
| Depends on | INIT-GATEFLOW-001 and INIT-GATEFLOW-002 **finished / delivered** (control plane, API wave start, per-skill runner/model config, PR thread from run start, fail-closed stubs for non-live slots, ForgeClient deploy path) `(Source: User-confirmed)` |
| Skills pin | No **version-bump exit gate**; Scenario A skills **must** be `dispatch: orchestrated` in the pin for 003 exit (supporting skills delivery) `(Source: User-confirmed)` |
| Primary delivery repo | **Primary:** gateflow · **Supporting:** prayog-skills pin `dispatch` edits for Scenario A |
| Product ids | Legacy `FR-n` ≡ `REQ-n` for this programme’s GATEFLOW PRDs |
| Target users | Engineering (live Cursor prove-it), tech lead (audit live vs not-yet-live), programme sponsor (cycle-time evidence) |

---

## 1. Executive Summary

### Problem Statement

INIT-GATEFLOW-001 and INIT-GATEFLOW-002 are **finished** `(Source: User-confirmed)`.
Gateflow can start a wave via API, honor the pinned workflow, stop at human
checkpoints, open a PR at run start, comment through stages, record metrics, and
expose run / metrics / board APIs. What is still missing for a **credible coding
stage** is a **live Cursor AgentRunner** in the Gateflow worker that actually
codes — with **defined cycle-time metrics** — not a stand-in path that leaves
“start the wave” short of real agent execution.

### Proposed Solution

**INIT-GATEFLOW-003** makes the **Cursor AgentRunner live** in Gateflow:

1. When programme config selects Cursor for a skill node with
   `dispatch: orchestrated`, Gateflow’s worker runs a **real Cursor coding
   agent** against the workspace (harness / skills / MDC already applied as
   today).
2. **Orchestrated ⇒ triggered:** Gateflow dispatches every resolved
   `dispatch: orchestrated` skill per the pinned workflow outcomes — it does
   **not** hardcode a node allowlist. Which nodes are orchestrated is SSOT in
   prayog-skills; runner/model come from gateflow config `(Source: User-confirmed)`.
3. Reuse 001/002 behaviour for wave start, contract stops, PR comments, and
   metrics APIs — **do not** rebuild the control plane.
4. **Fail fast** on missing/invalid Cursor credentials, Cursor start/crash, or
   any other required dependency for the live path — stop the run, record
   failure, notify as today; **no silent fake success** `(Source: User-confirmed)`.
5. Resolve `runner` (and model profile) from **gateflow repo config**; selecting
   an unsupported / not-live runner (e.g. OpenCode, Claude Code) **fails fast**
   at start — same honesty posture as INIT-002 stubs `(Source: User-confirmed)`.
6. Emit and expose **defined cycle-time metrics** for live Cursor orchestrated
   stages and for the wave run that used them `(Source: User-confirmed)`.

Gateflow still **does not write application code itself** (the agent does),
**does not merge PRs**, and **does not redefine delivery process** — prayog-skills
remains SSOT for workflow and skills. Launchpad does **not** choose the agent and
has **no product work** in this INIT.

### Success Criteria

| KPI | Target | Measurement |
|-----|--------|-------------|
| **Orchestrated nodes triggered** | Every resolved `dispatch: orchestrated` skill is dispatched by Gateflow and flows per pinned workflow + programme config (runner/model) — **no** hardcoded node allowlist in Gateflow | PolicyEngine / integration audit against pin |
| **Prove-it Scenario A — pre–Gate 2 eng** | Live Cursor runs the **set** of existing skills **`spec-draft`**, **`initiative-feasibility`**, **`spec-technical-review`**, **`spec-implementation-plan`** — each **must** be `dispatch: orchestrated` in the pin; config intended `cursor` + `auto`. Not one false linear order (pin happy path vs findings path differ) `(Source: User-confirmed)` | **Live coding work** in workspace + RunStore stages for those `workflow_node` ids with `runner=cursor` `(Source: User-confirmed)` |
| **Prove-it Scenario B — coding cycle** | Live Cursor runs existing skills **`pre-implement`**, **`loop-spec`**, **`verify`**, **`ground-spec`** per pinned outcomes (orchestrated; same config posture) | **Live coding work** + RunStore stages for those ids with `runner=cursor` `(Source: User-confirmed)` |
| **Post–Gate 2: Scenario A still runnable** | After Gate 2 opens, Scenario A skills remain triggerable when orchestrated — automation not coding-cycle-only | Live coding work + RunStore on a Scenario A node after Gate 2 |
| **Cycle-time — stage** | 100% of live Cursor orchestrated stages record `started_at`, `ended_at`, `duration_ms`, `runner`, `model_profile`, `model_id`, `outcome` | RunStore completeness check |
| **Cycle-time — wave** | 100% of runs that dispatch live Cursor record wave cycle time from **API accept / enqueue** to **stop at next contract node** (or terminal `failed`) as `wave_duration_ms` (or equivalent RunStore fields) | RunStore + metrics API |
| **Cycle-time — aggregates** | Metrics API returns **p50 and p95** `duration_ms` for orchestrated stages filtered by `runner=cursor` (and by `workflow_node` where data exists) | `GET` metrics contract tests (extends INIT-002 FR-21) |
| **Fail-fast dependencies** | Missing/invalid Cursor auth, Cursor start failure, or Cursor crash → run **fails**; no pretend success; Engineering-visible signal (PR/status as today) | Negative-path tests + RunStore `failed` |
| **Config-driven runner honesty** | Unsupported / not-live runners selected in config (OpenCode, Claude Code, unknown id) **block at start** — no silent Cursor substitute | PolicyEngine / config validation tests |
| **Contract still honored** | 100% stops at `human-checkpoint`, `external-action`, `decision`, `terminal` — zero auto-merge | Run event audit |
| **No Launchpad product claim** | No new Launchpad feature required for 003 exit | Impact map / delivery scope |

Numeric sponsor SLAs (e.g. “wave under N minutes vs manual”) are **out of scope**
as exit gates; this INIT **defines and ships** the cycle-time metrics above so
later calibration has evidence `(Source: User-confirmed)`.

### Scope boundary with INIT-GATEFLOW-001 / 002

| Topic | INIT-GATEFLOW-003 rule |
|-------|------------------------|
| Control plane + API waves + PR thread + board APIs | **Reuse** — finished in 001/002; do not rebuild |
| Per-skill runner/model config | **Reuse** (INIT-002 FR-16); Cursor becomes a **real** selectable live runner |
| INIT-002 FR-17 “Cursor implemented” | **Superseded for product meaning of live** — 003 requires a **live Cursor AgentRunner in the worker** that performs **live coding work**; any prior stand-in does not satisfy 003 exit `(Source: User-confirmed)`. *Implementation Note:* Cursor SDK adapter — see §4 |
| OpenCode / Claude Code | Remain **not live**; selecting them **fails fast** (extends FR-18 honesty) |
| Launchpad | Sync consumer only; **no** product work `(Source: User-confirmed)` |
| Skills pin | No **version-bump exit gate**; Scenario A **must** be `orchestrated` in pin (supporting prayog-skills delivery) `(Source: User-confirmed)` |
| Live prove-it scenarios | **Two scenarios** — skill **sets** (A) and (B) as below; orchestrated ⇒ triggered; live coding work evidence; intended `cursor` / `auto` `(Source: User-confirmed)` |

---

## 2. User Experience & Functionality

### User Personas

| Persona | Role | Primary need |
|---------|------|--------------|
| **Engineering** | Delivery / prove-it owner | Live Cursor on Scenario A and B with real coding work — not a stand-in |
| **Tech lead** | Gate / constitution owner | Audit which runners are live vs not-live; fail-fast is honest |
| **Programme sponsor** | Learning owner | Cycle-time metrics for Cursor stages and waves to compare later |

### Target experience

```text
API / programme start (INIT-002 path)
        │
        ▼
Validate config (runner/model from gateflow repo config)
  → unsupported / not-live runner → fail fast (no dispatch)
  → if runner: cursor → require Cursor credentials → else fail fast
        │
        ▼
Resolve next node from pinned workflow
  → dispatch: orchestrated + runner: cursor
        → live Cursor AgentRunner (this INIT)
  → dispatch: manual / human-checkpoint / external-action / …
        → stop or hand off per pin (unchanged 001/002)
        │
        ▼
Follow pinned outcomes to next node; comments + cycle-time metrics
        │
        ▼
Stop at contract / human checkpoint → human verifies / merges
```

**Normative rule:** whichever skill nodes are `dispatch: orchestrated` in the
pinned workflow **get triggered** and flow delivery as defined by the pin and
as configured (runner/model). Gateflow does not hardcode the node list
`(Source: User-confirmed)`.

### Prove-it scenarios (exact existing skills)

Automation is proven with **multiple scenarios**, not a single happy path. These
two cases use **exact existing skill / `workflow_node` names** from the pinned
development lane `(Source: User-confirmed)`:

| Scenario | Purpose | Exact skills (**set**, not one forced linear order) |
|----------|---------|-----------------------------------------------------|
| **A — Pre–Gate 2 eng** | Prove live Cursor before coding readiness | **Set:** `spec-draft`, `initiative-feasibility`, `spec-technical-review`, `spec-implementation-plan`. Pin graph: happy path `initiative-feasibility` **pass →** `spec-implementation-plan`; **findings →** `spec-technical-review` (then human checkpoint). Human stops are expected |
| **B — Coding cycle** | Prove live Cursor through the coding wave | **Set / wave order per pin:** `pre-implement`, `loop-spec`, `verify`, `ground-spec` |

**Shared rules for both scenarios:**

- Skills **must** be `dispatch: orchestrated` in the pin to be auto-triggered; Scenario A **must** be orchestrated for 003 exit `(Source: User-confirmed)`
- Runner/model from gateflow config — intended defaults **`cursor`** + **`auto`** (configurable / overridable)
- Flow follows pinned outcomes and contract stops — Gateflow does not invent a parallel graph
- Prove-it evidence = **live coding work** in the workspace + RunStore `runner=cursor` — not stand-in / empty success `(Source: User-confirmed)`
- Human-checkpoint / gate stops between skills are **expected stops** (not run failures); prove-it may span multiple runs or resume-after-human
- After Gate 2 opens, Scenario A skills remain triggerable when orchestrated

### User Stories & Acceptance Criteria

#### US-1 — Engineering proves automation via Scenario A and Scenario B

**As** Engineering, **I want** live Cursor proven on the **two exact existing-skill
scenarios** (pre–Gate 2 eng and coding cycle) **so that** we know orchestrated
automation works across delivery, not only on one node.

**Acceptance criteria:**

- [ ] Gateflow triggers AgentRunner for every resolved `dispatch: orchestrated`
  skill — no hardcoded node allowlist `(Source: User-confirmed)`
- [ ] **Scenario A** proves live Cursor on the skill **set**
  `spec-draft`, `initiative-feasibility`, `spec-technical-review`,
  `spec-implementation-plan` — each **must** be `orchestrated` in the pin
  `(Source: User-confirmed)`
- [ ] **Scenario B** proves live Cursor on
  `pre-implement`, `loop-spec`, `verify`, `ground-spec` `(Source: User-confirmed)`
- [ ] Both scenarios use config-driven runner/model (intended `cursor` + `auto`);
  flow follows the pinned workflow `(Source: User-confirmed)`
- [ ] When resolved `runner` is `cursor`, worker runs a **live** Cursor agent that
  performs **live coding work** in the workspace (not stand-in); Gateflow does
  not author application code `(Source: User-confirmed)`
- [ ] Human-checkpoints between Scenario A skills are **expected stops**, not
  failures; prove-it may use multiple runs / resume-after-human
- [ ] After **Gate 2** opens, Scenario A skills remain triggerable when
  orchestrated — not coding-cycle-only `(Source: User-confirmed)`
- [ ] Harness / skills / MDC remain from programme pin / Launchpad sync as today

#### US-2 — Engineering / tech lead trusts fail-fast on dependencies and unsupported runners

**As** Engineering or tech lead, **I want** missing Cursor access, Cursor crashes, and
unsupported runners to fail fast **so that** production never pretends success.

**Acceptance criteria:**

- [ ] Runner id resolved from **gateflow repo config** (`runner.default` and
  per-node overrides) — not hardcoded in PolicyEngine `(Source: User-confirmed)`
- [ ] Selecting OpenCode, Claude Code, or any unknown / not-live runner id that the
  run would require **blocks the whole run at start** with a clear structured
  error — no silent fallback to Cursor `(Source: User-confirmed)`
- [ ] Missing or invalid Cursor credentials / auth material required by the worker
  **fails fast** before or at AgentRunner start — run `failed`; notify as today
  `(Source: User-confirmed)`
- [ ] Cursor start failure, timeout, or crash → stop run; `outcome: failed`; no
  workflow advance; visible on run PR / status as today
- [ ] No silent inventing of successful coding stage outcomes

#### US-3 — Sponsor sees defined cycle-time metrics for live Cursor

**As a** programme sponsor, **I want** defined stage and wave cycle-time metrics
for live Cursor runs **so that** we can measure and later tune delivery with
evidence.

**Acceptance criteria:**

- [ ] Every live Cursor orchestrated stage persists `started_at`, `ended_at`,
  `duration_ms`, `runner`, `model_profile`, `model_id`, `outcome`
- [ ] Every run that dispatches live Cursor persists **wave cycle time** from API
  accept/enqueue to stop at next contract node (or terminal failure)
- [ ] Metrics API exposes **p50 and p95** stage `duration_ms` for
  `runner=cursor` (and by `workflow_node` when data exists) — extends INIT-002
  FR-21 / INIT-001 FR-10
- [ ] Cycle-time fields are queryable via existing run/metrics APIs — no
  gateflow-ops UI required
- [ ] Sponsor SLA thresholds (manual vs automated minutes) are **not** required for
  003 exit; shipping the defined metrics is `(Source: User-confirmed)`

#### US-4 — Tech lead keeps process SSOT and other plugs honest

**As a** tech lead, **I want** process to stay in pinned skills/harness and other
agent brands to stay not-live **so that** Cursor is a plug, not a process fork.

**Acceptance criteria:**

- [ ] prayog-skills / harness pin remain SSOT for workflow, skills, MDC
- [ ] Launchpad product changes are **out of scope**; existing harness sync may
  remain `(Source: User-confirmed)`
- [ ] OpenCode / Claude Code remain not live until a later INIT
- [ ] Human checkpoints never auto-passed; Gateflow never auto-merges

### Wave-run preconditions (delta)

Inherits INIT-GATEFLOW-002 [wave-run preconditions](./INIT-GATEFLOW-002.md#wave-run-preconditions).
Additional / clarified for this INIT:

| # | Precondition | Source |
|---|--------------|--------|
| 10 | Resolved runner + notifier for this run are **implemented / live** where required (not stubs / not-live) | INIT-002 FR-18; FR-28 |
| 11 | Resolved runner/model config for required orchestrated nodes is **valid** | INIT-002 FR-16 |
| **12** | If resolved runner is `cursor`, **Cursor worker credentials / auth material** required for live AgentRunner are present and usable — else **fail fast** (no dispatch or immediate `failed`) | FR-29 `(Source: User-confirmed)` |

### Functional Requirements (FR) — with Acceptance Criteria

| ID | Requirement | Acceptance criteria |
|----|-------------|---------------------|
| **FR-27** | Live Cursor + two prove-it scenarios | Gateflow **triggers** every resolved `dispatch: orchestrated` skill and flows per pin + config — **no** hardcoded allowlist. When `runner: cursor`, worker runs a **live** Cursor agent that performs **live coding work** (stand-in / empty success does **not** satisfy) `(Source: User-confirmed)`. **Prove-it skill sets:** **Scenario A** — `spec-draft`, `initiative-feasibility`, `spec-technical-review`, `spec-implementation-plan` (**must** be `orchestrated` in pin) `(Source: User-confirmed)`; **Scenario B** — `pre-implement`, `loop-spec`, `verify`, `ground-spec`. Intended config `cursor` + `auto` (configurable). After Gate 2, Scenario A remains triggerable when orchestrated. Human-checkpoints between Scenario A skills are expected stops. *Implementation Note:* Cursor SDK AgentRunner adapter (FR-6 I/O) in Gateflow worker — see §4 |
| **FR-28** | Config-driven runner + fail-fast unsupported | Resolve runner from gateflow repo config (default + per-`workflow_node` overrides from FR-16); OpenCode, Claude Code, and unknown/not-live ids **fail fast at start** when required — never silent substitute with Cursor `(Source: User-confirmed)` |
| **FR-29** | Fail-fast on Cursor / dependency failures | Missing/invalid Cursor auth, inability to start Cursor, timeout, or crash → stop run; `outcome: failed`; record + notify as today; **no pretend success** `(Source: User-confirmed)` |
| **FR-30** | Defined cycle-time metrics | Persist stage cycle time (`started_at`/`ended_at`/`duration_ms`) plus `runner`, `model_profile`, `model_id`, `outcome` for every live Cursor orchestrated stage; persist wave cycle time (API accept → contract stop or terminal failure); metrics API returns p50/p95 for `runner=cursor` (and by `workflow_node` when data exists) `(Source: User-confirmed)` |
| **FR-31** | Reuse 001/002 control plane | Wave start API, PR-at-start thread, Notifier comments, contract stops, RunStore, run/metrics/board APIs, ForgeClient deploy path — **consume, do not rebuild**; no new Launchpad product features |

**Inherited (unless noted):** INIT-GATEFLOW-001 FR-1, FR-3–FR-8, FR-10–FR-12, FR-14 remain in force as applicable under 002 supersessions. INIT-GATEFLOW-002 FR-15–FR-16, FR-18–FR-26b remain in force. **FR-17 product meaning of “Cursor implemented” is satisfied only when FR-27 is met** (live Cursor agent + live coding work in worker). FR-6 remains the AgentRunner interface contract; FR-27 is the live Cursor fulfillment. Legacy `FR-n` ≡ `REQ-n`.

### Error Handling

| Failure | System response | Visibility |
|---------|-----------------|---------------|
| Unsupported / not-live runner selected (OpenCode, Claude Code, unknown) | Block at start; no dispatch | Structured API/error with runner id + config key |
| Missing/invalid Cursor credentials | Fail fast; no successful coding stage | Structured reason; run `failed` or start rejected; PR/status as today |
| Cursor cannot start | Stop; `failed`; no workflow advance | PR thread + RunStore |
| Cursor timeout / crash mid-stage | Stop; `failed`; no workflow advance | PR thread + RunStore |
| Cursor completes but contract stop reached | Stop at contract node (success path for automation); human owns next step | PR thread + RunStore |
| Human-checkpoint / gate between Scenario A (or B) skills | **Expected stop** — not a run failure; prove-it may resume after human or use multiple runs | PR thread + RunStore `stopped` / checkpoint outcome |
| Metrics persist failure | Do not invent success metrics; preserve run outcome; flag for ops | RunStore + status API |
| Inherited 002 failures (auth, preconditions, ForgeClient, stubs) | Unchanged from INIT-GATEFLOW-002 Error Handling | Unchanged |

### Non-Goals

| Non-goal | Rationale |
|----------|-----------|
| Launchpad product changes | Launchpad does not select agents; no new Launchpad delivery |
| Cloud Cursor agents / cloud agent runtime | Explicitly out this INIT `(Source: User-confirmed)` |
| Live OpenCode / Claude Code | Later INIT; fail-fast if selected |
| Live Slack / Teams notifiers | Unchanged from 002 posture |
| gateflow-ops UI | Still deferred |
| Rebuilding API start / board APIs / PR-at-start platform | Finished in 001/002 |
| Redefining SDD / skills / harness in Gateflow | SSOT remains prayog-skills |
| Dogfood programme as driver of this INIT | Explicitly deferred |
| Product-mandated Gateflow CI AgentRunner stub | Omitted — live Cursor is the product bar; unit-test doubles are eng detail `(Source: User-confirmed)` |
| Hardcoding orchestrated node allowlists or Cursor brand in PolicyEngine | Pin `dispatch` is SSOT; runner/model from config |
| Requiring Gateflow code change to add/remove an orchestrated skill | Change pin `dispatch`; Gateflow honors |
| Pin version-bump or calendar deadline as exit gate | No version-bump exit gate; Scenario A pin **dispatch** content edits are in scope |
| Sponsor SLA thresholds (e.g. beat manual N minutes) | Metrics are defined and shipped; calibration later |

### Product Principles

1. **Orchestrated ⇒ triggered** — pin `dispatch: orchestrated` nodes flow as defined; no Gateflow node allowlist  
2. **Cursor is a plug** — process SSOT stays in pinned skills / harness / MDC  
3. **Config chooses the runner** — gateflow repo config; PolicyEngine does not hardcode brand  
4. **Fail fast, stay honest** — missing deps and unsupported runners never fake success  
5. **Measure cycle time** — stage and wave durations are first-class exit evidence  
6. **Reuse the control plane** — 003 fills the live Cursor gap; does not rebuild 001/002  
7. **Launchpad ≠ agent picker** — sync only; no Launchpad product work  

---

## 3. AI System Requirements

Gateflow orchestrates coding agents; it is not an LLM product itself.

### Tool & Runner Requirements

| Slot | This INIT | Contract |
|------|-----------|----------|
| **AgentRunner** | Cursor **live (SDK in worker)**; OpenCode **not live**; Claude Code **not live** | Same I/O as INIT-001 FR-6; selection by resolved `runner` id from config |
| **Notifier** | Unchanged from 002 (GitHub comments live; Slack/Teams stubs) | Structured run events |
| **ToolProvider** | `none` (unchanged) | No Graphify/MCP |
| **ModelGateway** | Cursor-native (no proxy) | Profiles from gateflow repo config; per-node overrides via FR-16 |
| **ForgeClient** | Unchanged | Deployed GitHub writes |

### Evaluation Strategy

| Dimension | Method | Pass threshold |
|-----------|--------|----------------|
| **Orchestrated trigger** | Pin with `dispatch: orchestrated` skills; start run | AgentRunner invoked; flow follows pin outcomes |
| **Scenario A — pre–Gate 2 eng** | Live Cursor on skill **set** `spec-draft`, `initiative-feasibility`, `spec-technical-review`, `spec-implementation-plan` (must be orchestrated) | **Live coding work** + RunStore `runner=cursor` for each skill in the set |
| **Scenario B — coding cycle** | Live Cursor on `pre-implement`, `loop-spec`, `verify`, `ground-spec` | **Live coding work** + RunStore `runner=cursor` |
| **Post–Gate 2 Scenario A** | After Gate 2 open, trigger a Scenario A orchestrated skill with live Cursor | Live coding work + RunStore evidence |
| **Expected human stop** | Reach human-checkpoint mid Scenario A | Run stopped at checkpoint — **not** counted as AgentRunner failure |
| **No allowlist** | Add/remove orchestrated via pin only | Resolver honors pin; 0 hardcoded node lists |
| **Fail-fast unsupported runner** | Config with `opencode` / `claude` / unknown as required runner | Run blocked at start; 0 fake Cursor substitutes |
| **Fail-fast Cursor auth** | Worker without valid Cursor credentials | Fail fast; `failed` or start rejected; 0 pretend success stages |
| **Fail-fast Cursor crash** | Injected/simulated AgentRunner failure | Run stopped; `failed`; no contract auto-pass |
| **Stage cycle time** | Inspect RunStore after live Cursor stage | 100% stages have `duration_ms` (+ timestamps, runner, `model_profile`, `model_id`, outcome) |
| **Wave cycle time** | Inspect RunStore after run stop | Wave duration present from accept → stop/fail |
| **Aggregate cycle time** | Call metrics API filtered by `runner=cursor` | p50 and p95 returned when ≥ 1 sample exists |
| **Contract stops** | Inherit 001/002 policy tests | 0 auto human-gate transitions; 0 auto-merge |

---

## 4. Technical Specifications

### Architecture Overview

```text
                    ┌─────────────────────────────────────┐
  PE / tools ──────►│  Gateflow HTTP API (INIT-002)       │
                    │  • wave-start / runs / metrics      │
                    └──────────────┬──────────────────────┘
                                   │ enqueue
                                   ▼
                         Async worker + PolicyEngine
                                   │
                    resolve runner from gateflow config
                                   │
              ┌────────────────────┼────────────────────┐
              ▼                    ▼                    ▼
     runner=cursor          not-live / unknown     Notifier + ForgeClient
     live Cursor SDK        fail fast at start     (unchanged from 002)
     AgentRunner in worker
              │
              ▼
     Record stage + wave cycle-time metrics (FR-30)
              │
              ▼
     Stop at contract / human checkpoint
```

**Pluggability rule (unchanged):** no slot embeds workflow node ids, gate label
strings, or dispatch allowlists — pinned contract + programme config only.

**Launchpad:** may already sync harness before a skill runs; does **not** select
Cursor; **no** Launchpad product changes in this INIT.

### Integration Points

| Integration | Direction | Mechanism |
|-------------|-----------|-----------|
| **Clients** | Inbound | HTTP API + programme service token (INIT-002) |
| **GitHub** | Outbound | ForgeClient (unchanged) |
| **prayog-skills** | Read + supporting delivery | Installed programme pin; Scenario A `dispatch: orchestrated` pin edits **in scope** (no version-bump exit gate) |
| **launchpad** | Pre-dispatch (existing) | Harness sync on worker if already wired — no new product work |
| **PostgreSQL** | Read/write | RunStore + job queue + cycle-time fields |
| **Cursor SDK** | Outbound | **Live** AgentRunner on Gateflow worker (this INIT) |
| **gateflow-ops** | Future consumer | Existing APIs — UI out of scope |

### Programme Config (gateflow repo)

Stays in **gateflow repository** (not meta/harness). Relevant keys for this INIT:

| Key | Purpose | Example / default |
|-----|---------|-------------------|
| `runner.default` | Default AgentRunner | `cursor` (intended prove-it default) |
| `model.profiles` | Named profile → model | `default: cursor/auto` (intended prove-it default; **configurable**) |
| `model.overrides` | Per-`workflow_node` profile and/or runner | e.g. `verify` / `loop-spec` / `spec-draft`: `{ runner: cursor, profile: default }` |
| Cursor auth / credentials | How worker obtains Cursor access | **Fail-fast if missing** — secret shape `[TBD in gateflow spec]` `(Source: User-confirmed)` |
| `notifier.default` | Unchanged from 002 | `github_comment` |
| `metrics.retention_days` | Unchanged | `90` |

### Cycle-time metrics (normative definitions) {#cycle-time-metrics}

| Metric | Definition | Stored / exposed |
|--------|------------|------------------|
| **Stage cycle time** | `ended_at - started_at` for one orchestrated AgentRunner dispatch | `duration_ms` on stage; dimensions: `workflow_node`, `runner`, `model_profile`, `model_id`, `outcome` |
| **Wave cycle time** | Time from API accept/enqueue of the run to stop at next contract node **or** terminal `failed` | `wave_duration_ms` (or equivalent run-level fields) on RunStore; included in run detail |
| **Aggregates** | p50 / p95 of stage `duration_ms` | Metrics API; filterable by `runner` (must support `cursor`) and `workflow_node` |

### Security & Privacy

| Concern | Requirement |
|---------|-------------|
| **Cursor credentials** | Programme-supplied secret/config for worker; never committed to repo; missing → fail fast (FR-29) |
| **API auth** | Unchanged — programme service token |
| **GitHub auth (deployed)** | Unchanged — ForgeClient |
| **Human gates** | Zero automated gate-approval label writes; zero auto-merge |
| **Cloud Cursor agents** | Out of scope — do not send programme work to cloud agent runtime in this INIT |

### Repositories

| Repo | This INIT deliverable |
|------|------------------------|
| **gateflow** | Live Cursor AgentRunner in worker; fail-fast auth/unsupported runners; cycle-time stage + wave metrics wiring |
| **gateflow-ops** | Out of scope |
| **prayog-skills** | **Supporting:** Scenario A skills marked `dispatch: orchestrated` in pin for 003 exit `(Source: User-confirmed)` |
| **launchpad** | No product delivery |
| **prayog-meta** | This PRD + impact map |

---

## 5. Risks & Roadmap

### Phased Rollout

| Phase | Scope | Entry |
|-------|-------|-------|
| **W0** | Live Cursor AgentRunner skeleton in worker; config resolve + fail-fast for unsupported runners; credential presence check | Draft PRD + impact map approved |
| **W1** | Scenario B (coding cycle: `pre-implement` → `loop-spec` → `verify` → `ground-spec`) with live Cursor; fail-fast crash/auth; stage + wave cycle-time | W0 merged |
| **W2** | Scenario A skill **set** with pin `dispatch: orchestrated` (required); post–Gate 2 Scenario A still runnable; metrics p50/p95 for `runner=cursor`; live coding work evidence | W1 merged |

Exact wave split may adjust in gateflow spec PR; product scope above is normative.

### Dependencies

| Dependency | Assumption |
|------------|------------|
| INIT-GATEFLOW-001 / 002 | **Finished / delivered** `(Source: User-confirmed)` |
| Cursor SDK (local / non-cloud worker path) | Available to Gateflow worker |
| Programme Cursor credentials | Supplied to worker via secret/config — fail fast if absent |
| PostgreSQL | RunStore |
| Existing ForgeClient + Notifier | Unchanged |

### Assumptions

| ID | Assumption | Status | Dependent FRs |
|----|------------|--------|---------------|
| A1 | INIT-GATEFLOW-001 and INIT-GATEFLOW-002 are finished/delivered | Confirmed `(Source: User-confirmed)` | FR-27–FR-31 |
| A2 | Cursor agent can run in Gateflow’s programme-like worker (non-cloud) | Confirmed `(Source: User-confirmed)` | FR-27, FR-29 |
| A3 | Programme can supply Cursor auth material to the worker without repo commits | Confirmed posture; secret shape TBD in spec `(Source: User-confirmed)` | FR-29 |
| A4 | Existing metrics/run APIs can expose new cycle-time fields without gateflow-ops UI | Confirmed (extends FR-20/21) | FR-30 |
| A5 | Prove-it uses **two scenarios** as skill **sets:** (A) `spec-draft`, `initiative-feasibility`, `spec-technical-review`, `spec-implementation-plan`; (B) `pre-implement`, `loop-spec`, `verify`, `ground-spec`. Orchestrated ⇒ triggered; live coding work evidence; intended `cursor` + `auto`; after Gate 2, Scenario A remains triggerable | Confirmed `(Source: User-confirmed)` | FR-27, FR-28 |
| A6 | Scenario B already orchestrated in today’s pin; Scenario A **must** be `orchestrated` in the pin for 003 exit (supporting prayog-skills delivery) | Confirmed `(Source: User-confirmed)` | FR-27 |

### Risk Register

| Risk | Likelihood | Impact | Mitigation | Owner |
|------|------------|--------|------------|-------|
| Cursor SDK worker auth harder than expected | Medium | High | Fail-fast product rule already set; spec narrows secret injection early | PE |
| Stand-in vs live ambiguity in codebase | Medium | High | FR-27 exit = live SDK only; remove or quarantine stand-in from `runner=cursor` path | Eng |
| Cycle-time field naming drift vs 001/002 schema | Low | Medium | Extend existing RunStore event schema; document in gateflow spec | PE |
| Unsupported runner list grows ad hoc | Low | Medium | Registry of live vs not-live ids; unknown → fail fast | Eng |

### Decisions (resolved) {#decisions-resolved}

| # | Decision | Resolution |
|---|----------|------------|
| 1 | New INIT vs patch 002 | **New INIT** — INIT-GATEFLOW-003 `(Source: User-confirmed)` |
| 2 | Cloud Cursor agents | **Out** of this INIT `(Source: User-confirmed)` |
| 3 | CI AgentRunner stub as product requirement | **Omit** — exit is live Cursor `(Source: User-confirmed)` |
| 4 | Dogfood as INIT driver | **Deferred** `(Source: User-confirmed)` |
| 5 | Primary delivery | **Primary:** gateflow · **Supporting:** prayog-skills pin `dispatch` for Scenario A `(Source: User-confirmed)` |
| 6 | 001 / 002 status | Both **finished / delivered** `(Source: User-confirmed)` |
| 7 | Launchpad role | Does **not** choose agent; **no** Launchpad product work `(Source: User-confirmed)` |
| 8 | Prove-it scenarios | **Scenario A** skill **set:** `spec-draft`, `initiative-feasibility`, `spec-technical-review`, `spec-implementation-plan` (**must** be orchestrated). **Scenario B:** `pre-implement`, `loop-spec`, `verify`, `ground-spec`. Evidence = **live coding work**. Intended `cursor` + `auto`. After Gate 2, A stays runnable `(Source: User-confirmed)` |
| 9 | Dependencies / auth failures | **Fail fast** — no pretend success `(Source: User-confirmed)` |
| 10 | Runner selection | **From config**; unsupported (OpenCode, Claude, …) **fail fast** `(Source: User-confirmed)` |
| 11 | Cycle-time metrics | **Defined and required** for exit (stage + wave + p50/p95 for Cursor) `(Source: User-confirmed)` |
| 12 | Pin version-bump / deadline | No **version-bump exit gate**; Scenario A pin **dispatch** content edits in scope `(Source: User-confirmed)` |

### Open Questions

1. Exact Cursor credential / secret injection shape for the worker — Engineering confirm in gateflow spec (product rule is fail-fast if absent).
2. Exact RunStore field names for wave cycle time if not already present — Engineering confirm in spec (definitions in [§4 Cycle-time metrics](#cycle-time-metrics) are normative).
3. Whether any leftover stand-in code path is deleted vs unreachable when `runner=cursor` — eng detail in spec; product rule: live coding work + live Cursor agent satisfies FR-27.

~~4. Pin edit timing for Scenario A orchestrated~~ — **Resolved:** Scenario A **must** be orchestrated; supporting skills delivery in W2 (Decision #5/#8/#12).

---

## Appendix A — Traceability

| Outline / Discovery | PRD section |
|---------------------|-------------|
| Outline §1–2 problem / solution | §1 Executive Summary |
| Outline §3 target experience | §2 Target experience |
| Outline §4 Launchpad rules | §2 Non-Goals; FR-31; Integration Points |
| Outline §5 personas / JTBD | §2 Personas + User Stories |
| Outline §6 features | FR-27–FR-31; US-1–US-4 |
| Outline §7 non-goals | §2 Non-Goals |
| Outline §10 success | §1 Success Criteria (+ cycle-time) |
| Outline §11 + Discovery answers | §5 Decisions |
| Discovery: live Cursor + cycle-time metrics | FR-30; Success Criteria |
| Discovery: fail-fast dependencies | FR-29 |
| Discovery: config runner + unsupported fail-fast | FR-28 |
| Discovery: two prove-it scenarios (exact skills) + orchestrated ⇒ triggered | FR-27; A5–A6; Decision #8; Prove-it scenarios |
| Discovery: no pin version-bump exit gate | Decision #12; Non-Goals |
| Resolution CHG-01–CHG-11 | Prove-it set; supporting skills; live coding work; Engineering persona; FR≡REQ |

## Appendix B — Process next steps

| Step | Owner | Artifact |
|------|-------|----------|
| 1 | PM / sponsor | Review this Draft PRD |
| 2 | Engineering | `/validate-requirements` incremental |
| 3 | Engineering | `/prd-impact-map` → **gateflow** primary |
| 4 | Engineering | Gateflow spec PR (Cursor auth injection, cycle-time field names) |
| 5 | Engineering | Delivery waves W0–W2 per §5 (+ supporting skills pin for Scenario A) |

## Appendix C — One-sentence product

> With the control plane finished (001/002), Gateflow **triggers every
> `dispatch: orchestrated` skill** and flows delivery as pinned and configured —
> live Cursor is the first runner plug, proven on **Scenario A** and **Scenario B**
> exact existing skills with **live coding work**, unsupported agents fail fast,
> cycle-time metrics are recorded, and Launchpad does not choose the agent.
