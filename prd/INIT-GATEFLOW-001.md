# INIT-GATEFLOW-001 — Gateflow delivery orchestrator

**Status:** draft PRD · **Author:** programme PM · **Date:** 2026-07-22  
**Outline:** [INIT-GATEFLOW-001-outline](./INIT-GATEFLOW-001-outline.md)  
**Vision:** [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md)  
**Paired initiative:** [INIT-PRAYOG-SKILLS-002](./INIT-PRAYOG-SKILLS-002.md) (`dispatch` on rc-2)  
**Component:** GATEFLOW · **Type:** platform / delivery control plane

> **Draft PRD** for Gate 1 review. Competitive landscape and full differentiation:
> [vision §12](../planning/gateflow-programme-vision.md#12-market-landscape-and-differentiation).
> Engineering implementation detail routes to impact map and gateflow spec PR.

---

## Document control

| Field | Value |
|-------|-------|
| Initiative ID | INIT-GATEFLOW-001 |
| Programme | prayog |
| Primary repos | drivestream-lab/gateflow (W1 delivery); drivestream-lab/gateflow-ops (W2+) |
| Supporting repos | prayog-meta, prayog-skills, launchpad |
| Gate 1 coupling | **Joint Gate 1** with INIT-PRAYOG-SKILLS-002 — required before rc-2 pin, **W1 PolicyEngine reading `dispatch`**, or dogfood Phase B |
| Primary delivery repo (W1) | **gateflow only** — impact map and dogfood scoped to gateflow through W1 |
| Target users | PE (wave execution), tech lead (gates), programme sponsor (metrics) |

---

## 1. Executive Summary

### Problem Statement

After upstream workflow steps complete (through `board-seed` on the pinned
`workflow.yaml`), PE **manually triggers** each wave `skill` node. Run state
lives in chat, agent tools are used ad hoc, and the programme has **no durable
audit trail or metrics** comparing manual vs orchestrated wave execution.

### Proposed Solution

**Gateflow** is a delivery control plane that listens to GitHub, reads handoff
envelopes and the pinned prayog-skills contract, resolves the next workflow
node, and **dispatches coding agents** only when the resolved node is
`type: skill` with `dispatch: orchestrated` and PE has authorized a run via
programme-configured trigger. It **stops** on all contract-assigned human and
external-action nodes, records run history in PostgreSQL, and exposes progress
via pluggable **AgentRunner**, **ToolProvider**, **ForgeClient**, and
**Notifier** slots.

Gateflow **does not write code** and **does not define the delivery process**.

### Success Criteria (MVP / H1)

| KPI | Target | Measurement |
|-----|--------|-------------|
| **Unattended wave completion** | ≥ 1 dogfood run completes from PE label trigger to next `human-checkpoint` without mid-chain PE skill invocation | RunStore + GitHub comment trail |
| **Contract stop compliance** | 100% of stops at `human-checkpoint`, `external-action`, `decision`, or `terminal` nodes — zero auto-transitions (see FR-8) | Run event log audit |
| **Dispatch policy compliance** | 0 dispatches when resolved skill has `dispatch != orchestrated` | PolicyEngine integration tests + run audit |
| **Run reconstructability** | PE can reconstruct full run timeline from PostgreSQL + GitHub comments within 5 minutes `(Source: User-confirmed)` | Manual drill-down test per dogfood run |
| **Orchestrated stage telemetry** | 100% of orchestrated dispatches record `started_at`, `ended_at`, `outcome`, `runner`, `model_id` | RunStore row completeness check |
| **Harness compliance** | `launchpad status --repo gateflow` green after each delivery wave | launchpad verify |
| **W1 operational readiness** | Gateflow control plane deployed, webhook→dispatch→stop loop proven on **gateflow repo**; ready to orchestrate future initiative waves without core rewrites | W1 exit checklist (§5) |

Numeric cycle-time baselines (`p50` duration vs manual waves) — **calibrate after Phase B dogfood** `[TBD]`.

---

## 2. User Experience & Functionality

### User Personas

| Persona | Role | Primary need |
|---------|------|--------------|
| **PE** | Programme engineer; wave execution owner | Authorize runs; trust contract stops; minimal babysitting |
| **Tech lead** | Gate reviewer; constitution enforcer | Same workflow gates as today; audit trail of what ran |
| **Programme sponsor** | Investment / learning owner | Evidence that automation improves cycle time and quality |

### User Stories & Acceptance Criteria

#### US-1 — PE authorizes a wave run

**As a** PE, **I want** to apply a programme-configured label to authorize a wave run **so that** Gateflow dispatches eligible skills until the next contract stop.

**Acceptance criteria:**

- [ ] Trigger label name is read from **gateflow programme config** (W1 — not harness/meta YAML); not hardcoded in Gateflow source
- [ ] Applying the label when [wave-run preconditions](#wave-run-preconditions) are satisfied creates a RunStore run record within 30s of webhook delivery `(Source: User-confirmed)`
- [ ] Applying the label when preconditions fail posts a ForgeClient comment explaining the block reason; no dispatch occurs
- [ ] Gateflow does not infer run intent from programme board state alone

#### Wave-run preconditions {#wave-run-preconditions}

Authoritative checklist before PolicyEngine may dispatch after label trigger:

| # | Precondition | Source |
|---|--------------|--------|
| 1 | Programme trigger label matches gateflow config `trigger.label` | Programme config |
| 2 | Latest handoff envelope readable from **triggering PR head ref**, or **default branch** when issue trigger has no linked PR (per FR-4) | FR-4 |
| 3 | `handoff.contract` matches Gateflow installed contract | handoff-envelope.md rule 2 |
| 4 | Resolved next node is `type: skill` with `dispatch: orchestrated` (rc-2 pin) | FR-5 |
| 5 | `handoff.human_checkpoint` is not blocking dispatch | handoff-envelope.md rules 5, 8 |
| 6 | No active run exists for the same PR/issue (concurrent trigger → **reject**) | FR-1 |
| 7 | Handoff has no unresolved `blockers` preventing progress | handoff envelope |
| 8 | `handoff.stage` is in wave lane per pinned `workflow.yaml` (post-`board-seed`) | FR-5 |

Failure of any precondition: ForgeClient comment with reason; no dispatch.

#### US-2 — PE observes run progress without chat archaeology

**As a** PE, **I want** run progress posted to the PR/issue thread **so that** I can see stage transitions without opening Cursor or chat.

**Acceptance criteria:**

- [ ] Each orchestrated stage posts a Notifier comment on run start and run end using the **run event schema** (see FR-9): `run_id`, `workflow_node`, `event` (`stage_started` \| `stage_completed` \| `run_stopped`), `outcome`, `duration_ms`, `timestamp`
- [ ] Stop at `human-checkpoint` posts explicit stop reason with workflow node id
- [ ] Comments are rendered via ForgeClient (GitHub API); production path does not invoke `gh` CLI
- [ ] **H1 uses PR/issue comments only** — no GitHub commit status checks for run progress (deferred W2+)

#### US-3 — Any programme engineer verifies contract gates were honored

**As a** programme engineer (PE, tech lead, or sponsor), **I want** an audit trail showing Gateflow never auto-set gate labels or auto-merged **so that** constitution invariants hold. Audit data is **not role-restricted among programme engineers** — any programme engineer with the programme service token may inspect runs `(Source: User-confirmed)`.

**Acceptance criteria:**

- [ ] RunStore and status API readable by any programme engineer holding the **programme service token** (no per-user RBAC on audit views in W1)

- [ ] RunStore records every stop event with `workflow_node`, `node_type`, and timestamp
- [ ] ForgeClient audit log shows zero writes to gate approval labels defined in pinned `delivery-contract.yaml`
- [ ] Zero auto-merge API calls in ForgeClient audit for MVP scope

#### US-4 — Sponsor compares manual vs automated waves

**As a** programme sponsor, **I want** per-skill duration and outcome metrics **so that** we can tune prayog-skills and model profiles with evidence.

**Acceptance criteria:**

- [ ] PostgreSQL events queryable by `workflow_node`, `dispatch_mode`, `runner`, `model_id`
- [ ] At least one gate-interval metric uses label names loaded from pinned `delivery-contract.yaml`, not hardcoded strings
- [ ] Export via **`GET /metrics/runs` JSON aggregate** (and documented SQL view) returns p50/p95 duration per orchestrated skill node; retention per programme config (`metrics.retention_days`, default 90)

#### US-5 — PE recovers from findings loops within budget

**As a** PE, **I want** Gateflow to retry `findings` transitions up to a programme-configured budget **so that** verify loops do not require manual re-trigger each pass.

**Acceptance criteria:**

- [ ] Retry budget default = 3; overridable in gateflow programme config (W1) without code change
- [ ] Each `findings` retry increments counter on run record; counter visible in status API
- [ ] Budget exhaustion **stops the run**, posts a structured ForgeClient comment on the PR/issue (reason, retry count, last workflow node), and does **not** open a GitHub issue or auto-route to another checkpoint

### Functional Requirements (FR) — with Acceptance Criteria

| ID | Requirement | Acceptance criteria |
|----|-------------|---------------------|
| **FR-1** | GitHub App receives webhooks (PR, issue, label) | Webhook endpoint validates App signature; rejects invalid payloads with 401; persists event id for idempotency; duplicate delivery does not create duplicate runs; **if active run exists on same PR/issue, reject new run** (no queue/supersede in W1) |
| **FR-2** | Explicit wave trigger via programme-configured label | Label name from **gateflow programme config** only; label event routes to TriggerRouter after [wave-run preconditions](#wave-run-preconditions); no board-only inference |
| **FR-3** | Durable run store (H1: PostgreSQL) | Runs, stages, and events persisted in PostgreSQL; no SQLite fallback; schema supports `run_id`, `workflow_node`, `outcome`, timestamps |
| **FR-4** | Handoff envelope read from repo artifacts | Resolve git ref from **triggering PR head SHA** when label is on a PR; when label is on an **issue without linked PR**, fall back to repo **default branch** per `handoff.ref_fallback` (default: `default_branch`); scan configured artifact globs for latest durable `handoff:` YAML block (see Programme Config); parse failure blocks run and notifies PE; no chat/session fallback |
| **FR-5** | Workflow resolver per `sdd-delivery/v2` | Loads pinned `workflow.yaml` + `delivery-contract.yaml`; resolves next node from `handoff.stage` + `handoff.outcome`; dispatches only when `type: skill` AND `dispatch: orchestrated`; missing `dispatch` on v0.4.3 pin → treat as `manual`; **before rc-2 pin: block dispatch with ForgeClient comment** (never silent no-op); **no hardcoded node allowlists in source** |
| **FR-6** | Coding agent adapter (H1: Cursor AgentRunner) | Adapter implements `AgentRunner` interface; receives workspace, skill prompt, `model_profile`; returns `RunResult` with `runner`, `model_id`, `outcome`; **on failure/timeout: stop run, record `outcome: failed`, notify PE, do not advance workflow**; interface documented for H2 runners |
| **FR-7** | Retry budget on `findings` loops | Programme config key; default 3; PolicyEngine enforces before re-dispatch; counter persisted on run; **on exhaustion: stop run + GitHub comment only** (no issue creation) |
| **FR-8** | Stop on human/external/decision/terminal nodes | Never auto-transition `human-checkpoint`; never execute `external-action`; **stop at `decision` and `terminal` nodes without dispatch**; honor `handoff.human_checkpoint: true` |
| **FR-9** | Notifier → GitHub comments via ForgeClient | Structured run events fan out to Notifier; H1 impl posts PR/issue comments only (no commit status checks) using run event schema (`run_id`, `workflow_node`, `event`, `outcome`, `duration_ms`, `timestamp`); event schema stable for H2 Slack/Teams backends |
| **FR-10** | Metrics v0 + skill stage profiling | Emit events per [§4 Metrics Schema](#metrics-schema-runstore-events): orchestrated skills, observed skills (where parseable), gate intervals, findings loops; **`GET /metrics/runs` JSON aggregate** returns p50/p95 duration by `workflow_node`; SQL view documented for PE; **90-day retention** (configurable); no scheduled export in W1 |
| **FR-11** | ToolProvider slots (all `none` in MVP) | `StageToolResolver` resolves slot from programme config or workflow metadata — not hardcoded node→slot map; H1 returns empty context |
| **FR-12** | gateflow read-only status API | JSON endpoint on **gateflow** returns run id, current stage, outcome, timestamps; **authenticated via programme service token** (shared among programme engineers; network boundary in W1); sufficient for W1 without gateflow-ops BFF |
| **FR-13** | Runner + model profile per dispatch | **H1: single `default` profile** for all orchestrated wave skills; per-`workflow_node` overrides deferred until Phase C baseline; resolved from gateflow programme config; all four fields (`runner`, `model_profile`, `model_id`, `model_provider`) persisted on orchestrated stages where available |
| **FR-14** | ForgeClient for outbound GitHub | App installation token (preferred); scoped PAT allowed in dev only; supports comments, PR create/update, programme run-status labels; forbids gate approval labels and auto-merge; **on comment API failure: retry with backoff and/or mark run `notify_pending` in RunStore** |

### Error Handling

| Failure | System response | PE visibility |
|---------|-----------------|---------------|
| Invalid webhook signature | Reject 401; no run created | N/A |
| Duplicate webhook delivery | Idempotent skip; no duplicate run | N/A |
| Concurrent label on active run | Reject new run; ForgeClient comment | Comment on PR/issue |
| Handoff parse failure | Block run; no dispatch | ForgeClient comment with parse error |
| Contract mismatch (`handoff.contract`) | Block run | ForgeClient comment |
| Precondition failure | Block run | ForgeClient comment with failed precondition |
| rc-2 / dispatch unavailable (W0 pre-pin) | Block run; no dispatch | ForgeClient comment (rc-2 pin required for orchestration) |
| AgentRunner timeout/crash | Stop run; `outcome: failed`; no workflow advance | Notifier comment + RunStore |
| Retry budget exhausted | Stop run | ForgeClient comment (reason, count, node) |
| ForgeClient API failure (5xx/rate limit) | Retry/queue comment; set `notify_pending` if exhausted | RunStore flag; comment when recovered |
| PostgreSQL unavailable | Fail webhook processing; return 503 | GitHub retry; alert via ops `[TBD]` |
| PE cancellation (future) | Stop run at stage boundary | Notifier comment — **formal cancel API deferred W2**; W1 relies on retry-budget exhaustion and AgentRunner failure stops |

### Non-Goals (MVP)

| Non-goal | Rationale |
|----------|-----------|
| Auto-merge PRs | `external-action` nodes require explicit authorization |
| Auto-update programme board | PE-owned; outside workflow dispatch |
| Dispatch when `dispatch != orchestrated` | Per pinned `workflow.yaml` |
| Build a coding agent | Cursor / others via AgentRunner adapters |
| Graphify / MCP in MVP | ToolProvider slots wired; provider = `none` |
| Fork or redefine delivery process | SSOT is prayog-skills |
| Hardcode orchestrated skill lists, gate labels, node→model/tool maps | Contract + programme config only |
| OpenCode / Claude Code / LiteLLM in MVP | H2+; interface in H1 spec |
| `gh` CLI in production | ForgeClient + GitHub API only |
| Slack / Teams notifications | Notifier slot reserved; H1 GitHub only |
| GitHub commit status checks for run progress | H1 uses structured PR/issue comments only (US-2); status checks deferred W2+ |
| Run queue / supersede on concurrent trigger | W1 rejects concurrent runs (FR-1); queue/supersede deferred W2+ if dogfood requires |
| Auto-set gate approval labels | Observe for metrics; humans set gates |
| Full ops dashboard / gateflow-ops BFF | H2; W1 uses gateflow native status API only |

### Product Principles

1. **Workflow SSOT** — `workflow.yaml` + `delivery-contract.yaml` govern navigation and dispatch eligibility
2. **Artifacts not chat** — handoff envelopes are durable state
3. **Contract invariants** — never auto-transition human gates; never execute external actions without authorization
4. **Explicit triggers** — gateflow programme config (W1) label authorizes; resolver checks `dispatch: orchestrated`
5. **Pluggable, not preset** — slots swap implementations; config drives triggers, retries, models, tools
6. **Cloud-native forge** — GitHub API only in production
7. **Measure to improve** — profile every workflow skill for prayog-skills learning

---

## 3. AI System Requirements

Gateflow orchestrates AI coding agents; it is not itself an LLM product. This
section defines agent dispatch, model configuration, and evaluation.

### Tool & Runner Requirements

| Slot | H1 implementation | Contract |
|------|-------------------|----------|
| **AgentRunner** | Cursor SDK adapter | Input: workspace, skill prompt, `model_profile`, optional `tool_context` → Output: `RunResult{runner, model_id, model_provider, outcome, artifacts}` |
| **ToolProvider** | `none` (no-op) | `StageToolResolver` returns empty context; interface ready for Graphify/MCP H2 |
| **ModelGateway** | Cursor-native (no proxy) | Gateflow programme config (W1) resolves `model_profile` → runner-specific model id; H3 LiteLLM slot reserved |

### Dispatch Preconditions (normative)

Full authoritative checklist: [wave-run preconditions](#wave-run-preconditions).

```text
next = resolve(handoff.stage, handoff.outcome)

if pre_rc2_or_dispatch_unavailable:           STOP + notify (never silent no-op)
if active_run_on_same_pr_issue:              STOP + notify (reject; W1)
if handoff.blockers unresolved:               STOP + notify
if handoff.stage not in wave_lane:            STOP + notify
if next.type != skill:                        STOP
if next.dispatch != orchestrated:              STOP or observe-only
if not programme_trigger_authorized:           STOP
if handoff.human_checkpoint == true:            STOP
if handoff.contract != installed_contract:      STOP + notify

dispatch via AgentRunner(next)
```

Source: [INIT-PRAYOG-SKILLS-002 §3.3](./INIT-PRAYOG-SKILLS-002-outline.md) plus normative extensions from
[handoff-envelope.md](../prayog-skills/references/handoff-envelope.md) rules 2, 5, and 8
(contract match, never auto-transition human checkpoint, `next_candidates` never bypasses
`human_checkpoint`). Requires rc-2 pin for `dispatch` field. Paired INIT alignment tracked in Joint Gate 1.

### Evaluation Strategy

| Dimension | Method | Pass threshold (MVP) |
|-----------|--------|----------------------|
| **Resolver correctness** | Contract tests against pinned `workflow.yaml`; fixtures imported from pinned prayog-skills ref `(Source: User-confirmed)` | 100% match with handoff spec navigation for all scenarios in prayog-skills `workflow_scenarios.json` (navigation only); **plus Gateflow-specific fixtures for rc-2 `dispatch` policy** |
| **Policy compliance** | Integration tests: no dispatch on `manual`, `human-checkpoint`, missing trigger | 0 violations in CI |
| **Run completeness** | Post-dogfood audit of RunStore rows | 100% orchestrated stages have start/end/outcome/model fields |
| **Agent outcome** | PE checklist at `wave-human-decision` | Wave artifacts present on branch; `launchpad status --repo gateflow` green; PE sign-off recorded |
| **Model efficacy** | Compare `workflow_node × model_id` first-pass rate after ≥ 3 runs | Baseline report for sponsor; no numeric target until Phase C |

---

## 4. Technical Specifications

### Architecture Overview

```text
GitHub webhooks ──► API / TriggerRouter ──► enqueue run job ──► Postgres job table
        │                    │ ack fast (202)
        │                    │
        │              Async worker ◄── claim jobs
        │                    │
        │                    ▼
        │              PolicyEngine ◄── workflow.yaml (pin)
        │                    │         HandoffReader ◄── repo artifacts
        │                    ▼
        │              RunStore (PostgreSQL) ◄── MetricsEmitter
        │                    │
        │    ┌───────────────┼───────────────┬───────────────┐
        │    ▼               ▼               ▼               ▼
        │  AgentRunner  ToolProvider    ForgeClient      Notifier
        │  (Cursor H1)  (none H1)       (GitHub API)     (→ ForgeClient H1)
```

**Deployment note (W1):** the API process validates webhooks and enqueues work
then returns quickly; the **async worker** owns PolicyEngine dispatch,
AgentRunner lifecycle, and ForgeClient/Notifier side effects (Decision #6).

**Pluggability rule:** no slot implementation embeds workflow node ids, gate
label strings, or dispatch allowlists — those come from pinned contract +
programme config.

**Control-plane components (gateflow repo):**

| Component | Responsibility |
|-----------|----------------|
| `WorkflowEngine` | Load pin; resolve transitions |
| `PolicyEngine` | Enforce dispatch algorithm, retry budget, contract invariants |
| `HandoffReader` | Parse handoff envelope from repo |
| `TriggerRouter` | Map programme-config labels → run authorization |
| `RunStore` | PostgreSQL persistence |
| `MetricsEmitter` | Stage/gate/run events |
| `StageToolResolver` | Resolve tool slot from config/metadata |

### Implementation constraints (H1)

Platform INIT — H1 ships with these technology choices; later horizons may swap
implementations via pluggable slots without changing PolicyEngine contract:

| Constraint | H1 choice | FR |
|------------|-----------|-----|
| Run store | PostgreSQL (all envs) | FR-3 |
| Coding agent | Cursor SDK via AgentRunner adapter | FR-6 |
| GitHub platform API | GitHub App + ForgeClient (no `gh` CLI) | FR-14 |
| Deployment | Docker: **API + async worker** (webhook ack fast; worker owns AgentRunner + job queue via Postgres job table) | W1 exit #1 |
| Programme config location | **gateflow repo config** (W1) | FR-2, FR-7, FR-13 |

Capabilities (durable run history, agent dispatch, GitHub integration) are
requirement-level; the rows above are **implementation constraints**, not
process SSOT.

### Workflow Navigation (contract SSOT)

| Check | Source | Gateflow action |
|-------|--------|-----------------|
| Stop at gate / merge | `type: human-checkpoint` / `external-action` | Stop |
| Orchestrator eligible? | `skill` + `dispatch: orchestrated` | May dispatch if triggered |
| Manual skill | `skill` + `dispatch: manual` | Stop; observe metrics from handoff |
| Metrics-only skill | `skill` + `dispatch: observed` `[TBD enum]` | Stop; observe metrics |

**Illustrative only — SSOT in prayog-skills rc-2** (Gateflow must not hardcode):
wave lane skills `pre-implement`, `loop-spec`, `verify`, `ground-spec` →
`dispatch: orchestrated`; all other skills → `manual`.

### Integration Points

| Integration | Direction | Mechanism |
|-------------|-----------|-----------|
| **GitHub** | Inbound | App webhooks (labels, PR, issue) |
| **GitHub** | Outbound | ForgeClient REST/GraphQL; App installation token |
| **prayog-skills** | Read | Pinned `workflow.yaml`, `delivery-contract.yaml`, handoff spec |
| **launchpad** | Pre-dispatch | `sync-harness` on worker workspace before AgentRunner |
| **PostgreSQL** | Read/write | RunStore SSOT |
| **Cursor SDK** | Outbound | AgentRunner adapter on container worker |
| **gateflow-ops** | Read (W2+) | BFF → gateflow status JSON API — **deferred W2**; W1 uses gateflow native API |
| **Repo git** | Worker | Deploy keys / fine-grained token for agent push (separate from ForgeClient) |

### Programme Config (gateflow repo — W1)

W1 programme config lives in the **gateflow repository** (not prayog-meta harness).
Harness/meta schema for shared keys is deferred to W2+ `(Source: User-confirmed)`.

| Key | Purpose | Example / W1 default |
|-----|---------|----------------------|
| `trigger.label` | Wave run authorization label (programme-wide for W1 dogfood) | `gateflow:run-wave` |
| `handoff.ref` | Git ref for HandoffReader (PR triggers) | `pr_head` |
| `handoff.ref_fallback` | Ref when issue trigger has no linked PR | `default_branch` |
| `handoff.artifact_globs` | Paths to scan for latest `handoff:` block | `["docs/specification/reports/**", "prd/reports/**"]` |
| `retry.findings_budget` | Max `findings` loop passes | `3` |
| `metrics.retention_days` | RunStore event retention | `90` |
| `runner.default` | Default AgentRunner adapter | `cursor` |
| `model.profiles` | Named profile → runner model mapping | `default: cursor/auto` |
| `model.overrides` | Optional per-`workflow_node` profile | `{}` (H1 — defer tuning until Phase C) |
| `tools.slots` | Optional node → tool slot mapping | `{}` (H1 empty) |

### Metrics Schema (RunStore events) {#metrics-schema-runstore-events}

Each event row queryable by:
`programme_slug`, `org`, `repo`, `initiative_id`, `workflow_node`, `wave`,
`outcome`, `dispatch_mode` (`orchestrated` | `manual` | `observed`), optional
`run_id`, and for orchestrated stages: `runner`, `model_profile`, `model_id`,
`model_provider`.

Gate interval labels resolved from pinned `delivery-contract.yaml`
(`github.labels`), not Gateflow constants.

### Security & Privacy

| Concern | Requirement |
|---------|-------------|
| **GitHub auth** | App installation token in production; PAT dev-only, scoped |
| **Webhook integrity** | Validate App webhook signatures; reject replayed events via idempotency key |
| **Secrets** | App private key, Postgres URL, deploy keys in secret store — not in repo |
| **Agent workspace** | Ephemeral worktree per run; no cross-run artifact leakage |
| **Audit** | ForgeClient write log retained with run record |
| **Status API auth (W1)** | Programme service token (shared among programme engineers); API behind programme network boundary; no per-user RBAC |
| **Human gates** | Zero automated writes to gate approval labels |

### Repositories

| Repo | W1 deliverable | Post-W1 |
|------|----------------|---------|
| **gateflow** | Full control plane: API, webhooks, PolicyEngine, RunStore, Cursor AgentRunner, ForgeClient, Notifier, status JSON API, ToolProvider slots (`none`) | Dogfood + future initiative orchestration |
| **gateflow-ops** | **Out of W1 scope** — no BFF or UI required for W1 exit | BFF + minimal UI in W2 / H2 |

**W1 impact map scope:** `drivestream-lab/gateflow` only. gateflow-ops follows once
the control plane is proven operational.

---

## 5. Risks & Roadmap

### Phased Rollout

| Phase | Scope | Entry criteria |
|-------|-------|----------------|
| **Phase A — Build (manual SDD)** | Gateflow W0/W1 on **gateflow repo only** | This PRD + impact map approved |
| **Phase B — Dogfood automation** | Label-triggered wave runs on **gateflow repo** | W1 exit criteria met; **Joint Gate 1** passed; rc-2 pin active |
| **Phase C — Learning** | Manual vs automated metrics comparison | ≥ 1 completed Phase B run |

#### Normative delivery sequence (W1 → Phase B)

```text
Joint Gate 1 (with INIT-PRAYOG-SKILLS-002)
  → rc-2 pin on prayog-skills (dispatch field)
  → complete W1 exit criteria 3–11 (incl. e2e dispatch on gateflow repo)
  → W1 exit declared
  → Phase B dogfood (label-triggered waves)
```

W0 may proceed before Joint Gate 1 (skeleton without dispatch). W1 criteria
3–11 that depend on `dispatch` require rc-2 pin after Joint Gate 1.

#### Engineering waves (PRD level)

| Wave | Theme | Scope | Outcomes |
|------|-------|-------|----------|
| **W0** | Control plane skeleton | gateflow | GitHub App webhooks; PostgreSQL RunStore; HandoffReader; WorkflowEngine resolver (no dispatch); ForgeClient comments; programme config loader |
| **W1** | **Operational control plane** | gateflow | PolicyEngine + Cursor AgentRunner; TriggerRouter; retry budget (exhaustion → stop + comment); Notifier; metrics v0; **gateflow status JSON API**; container deploy; pluggable slots wired |
| **W2** | Dogfood hardening + ops visibility | gateflow; gateflow-ops stub | Idempotency; failure paths; Phase B pilot; gateflow-ops BFF optional |

Detail: gateflow spec PR.

#### W1 exit criteria — control plane ready for future waves

W1 is **not complete** until Gateflow is **up, running, and reusable** for
orchestrating delivery on programme repos beyond the bootstrap build. PE must be
able to trigger the next wave on gateflow (Phase B) and later initiatives
without rewriting the orchestration core.

| # | Criterion | Verification |
|---|-----------|--------------|
| 1 | **Deployed runtime** | Gateflow **API + async worker** run in Docker (or programme dev environment); health endpoint returns 200; webhook returns quickly while worker processes jobs |
| 2 | **Webhook path live** | GitHub App delivers label events to Gateflow; signature validation passes |
| 3 | **End-to-end dispatch loop** | Label trigger → handoff read → resolver → AgentRunner dispatch → stop at next contract node — proven on gateflow repo (requires rc-2 pin for `dispatch`) |
| 4 | **RunStore durable** | All stages and events persisted in PostgreSQL; run reconstructable from DB alone |
| 5 | **Human-visible progress** | ForgeClient comments on PR/issue for start, stage complete, stop, and retry-budget exhaustion |
| 6 | **Status API** | `GET /runs/{id}` (or equivalent) returns current stage, history, outcomes — no gateflow-ops dependency |
| 7 | **Contract compliance** | Zero auto-transitions on human gates; zero gate-label writes; dispatch only when `dispatch: orchestrated` |
| 8 | **Pluggable slots** | AgentRunner, ToolProvider, ForgeClient, Notifier interfaces implemented; swapping adapter does not require PolicyEngine changes |
| 9 | **Programme config** | Trigger label, retry budget, model profiles loaded from config — no hardcoded node lists in source |
| 10 | **Harness green** | `launchpad status --repo gateflow` passes after W1 merge |
| 11 | **Future-wave ready** | Runbook in gateflow docs: **"Orchestrate a new initiative repo"** (App install + gateflow programme config + skills pin); PE can authorize runs without WorkflowEngine code changes |

**W1 documentation deliverable:** runbook above satisfies criterion 11.

**Explicitly not required for W1 exit:** gateflow-ops UI/BFF, multi-runner adapters, ToolProvider beyond `none`, Slack/Teams Notifier.

### Product Horizons

| Horizon | Theme | Init |
|---------|-------|------|
| **H1 — MVP** | Cursor, ForgeClient, run engine, metrics v0, slots wired (`none`) | **This INIT** |
| **H2 — Scale** | Multi-runner, ops UI, optional tool providers | INIT-GATEFLOW-002 `[TBD]` |
| **H3 — Gateway** | LiteLLM proxy, cost/quality dashboards | INIT-GATEFLOW-003 `[TBD]` |
| **H4 — Policy** | Opt-in merge/fleet automation | Future |

### Dependencies

#### Joint Gate 1 (blocking — with INIT-PRAYOG-SKILLS-002)

Both INITs must pass a **single Joint Gate 1** before rc-2 pin, **W1 PolicyEngine
reading `dispatch`**, or dogfood Phase B.

| Blocked until Joint Gate 1 | May proceed before |
|----------------------------|-------------------|
| PolicyEngine reading `dispatch` | W0 skeleton without dispatch |
| Meta harness pin to rc-2 | Draft PRD, validate-requirements, impact map |
| Dogfood Phase B | Phase A manual SDD build |

#### Technical dependencies

| Dependency | Assumption |
|------------|------------|
| prayog-skills rc-2 | `dispatch` field via INIT-PRAYOG-SKILLS-002 |
| prayog-skills @ v0.4.3 | Missing `dispatch` → schema default `manual` |
| launchpad | Harness sync before agent dispatch |
| GitHub App | Installed on drivestream-lab programme repos |
| Cursor SDK | Container worker or cloud VM |
| PostgreSQL | All environments; no SQLite |
| Docker | Production deployment target |

#### Assumptions

| ID | Assumption | Status | Dependent FRs |
|----|------------|--------|---------------|
| A1 | prayog-skills rc-2 with `dispatch` merges after Joint Gate 1 | Pending | FR-5, W1 #3 |
| A2 | GitHub App installed on drivestream-lab programme repos | Confirmed | FR-1, FR-14 |
| A3 | Cursor SDK runs in container worker (not PE laptop-only) | Pending | FR-6, W1 #1 |
| A4 | PostgreSQL available in all Gateflow environments | Confirmed | FR-3 |
| A5 | Gateflow programme config file in gateflow repo for W1 | Approved | FR-2, FR-7, FR-13 |
| A6 | Any programme engineer may read RunStore/status API with programme service token (open audit; no RBAC) | Confirmed | US-3, FR-12 |
| A7 | RunStore metrics events retained 90 days in W1 | Approved | FR-10 |
| A8 | Single `default` model profile for all orchestrated wave skills in H1 | Approved | FR-13 |

### Risk Register

| Risk | Likelihood | Impact | Mitigation | Owner |
|------|------------|--------|------------|-------|
| Agent run cost / duration | Medium | Medium | Retry budget; metrics; model profiles in config | PE |
| Webhook delivery gaps | Medium | High | Idempotent handlers; event dedup in RunStore | Eng |
| Handoff parse failures | Low | High | Block run; notify PE; no silent continue | Eng |
| rc-2 / dispatch delay | Medium | Medium | Schema default `manual`; no allowlist fallback | PM + PE |
| Over-automation pressure | Medium | High | Non-goals; contract invariants in PolicyEngine tests | Sponsor |
| Cursor SDK container friction | Medium | Medium | Phase A manual SDD de-risks; pilot on gateflow repo first | PE |

### Decisions (resolved) {#decisions-resolved}

| # | Decision | Resolution |
|---|----------|------------|
| 1 | Run progress surface (H1) | **PR/issue comments only** via ForgeClient — no commit status checks until W2+ |
| 2 | Concurrent wave trigger (W1) | **Reject** new run while one is active on same PR/issue; queue/supersede deferred W2+ |
| 3 | Pilot trigger label (W1) | **Programme-wide** `gateflow:run-wave`; per-initiative labels later |
| 4 | Status API access (W1) | **Shared programme service token** + network boundary; no RBAC among programme engineers |
| 5 | Handoff source path | **PR head ref** + configured artifact globs; latest durable `handoff:` block wins |
| 6 | W1 deployment shape | **API + async worker**; Postgres job table; webhook ack fast |
| 7 | Metrics retention / export (H1) | **90-day retention**; `GET /metrics/runs` JSON aggregate; SQL view for PE |
| 8 | Model profiles (H1 dogfood) | **Single `default` profile** for all orchestrated nodes; per-node overrides after Phase C |
| 9 | Pre–rc-2 label trigger | **Block + ForgeClient comment** when orchestration unavailable — never silent no-op |
| 10 | PE mid-run cancellation | **Formal cancel API deferred W2**; W1 uses failure/retry-budget stops |
| 11 | Retry budget exhaustion | **Stop run + ForgeClient GitHub comment only** — no issue creation, no auto-route |
| 12 | W1 dogfood / impact map scope | **gateflow repo only** through W1; gateflow-ops deferred to W2+ |

### Open Questions (Joint Gate 1 agenda)

*Decisions 1–12 resolved — see [Decisions](#decisions-resolved) table.*

1. ForgeClient auth: App token only vs PAT in dev?
2. rc-2 pin timing vs Gateflow W0 merge?

---

## Appendix A — Traceability

| Outline section | PRD section |
|-----------------|-------------|
| FR-1 … FR-14 | §2 Functional Requirements |
| §5.2 navigation | §4 Workflow Navigation |
| §5.4 metrics | §4 Metrics Schema; §3 Evaluation |
| §5.6 pluggable runtime | §4 Architecture; §3 Tool Requirements |
| §9 dogfood | §5 Phased Rollout |
| §11 Joint Gate 1 | §5 Dependencies |
| §3 positioning | Vision §12 (reference only) |

## Appendix B — Process next steps

| Step | Owner | Artifact |
|------|-------|----------|
| 1 | PM / sponsor | Review this Draft PRD |
| 2 | PE | `/validate-requirements` on Draft |
| 3 | PE / sponsor | **Joint Gate 1** with INIT-PRAYOG-SKILLS-002 Draft |
| 4 | PE | `/prd-impact-map` → **gateflow only** (W1); gateflow-ops W2+ |
| 5 | PE | Gateflow spec PR + rc-2 (paired) |
| 6 | PE | Dogfood Phase B |
