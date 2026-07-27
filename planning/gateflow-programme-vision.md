# Gateflow programme vision

**Programme:** prayog · **Org:** drivestream-lab · **Status:** draft  
**Last updated:** 2026-07-27  
**Related initiatives:** [INIT-GATEFLOW-001](../prd/INIT-GATEFLOW-001.md) · [INIT-GATEFLOW-003](../prd/INIT-GATEFLOW-003.md) · [INIT-GATEFLOW-004-outline](../prd/INIT-GATEFLOW-004-outline.md) · [INIT-PRAYOG-SKILLS-002](../prd/INIT-PRAYOG-SKILLS-002.md)

---

## 1. Why we exist

Prayog has proven that **spec-driven delivery works** when upstream debate is
thorough and downstream skills execute against frozen intent. Skills, harness,
constitutions, and handoff envelopes compress quality into the pipeline.

What remains manual is **skill dispatch between workflow stop points**: PE
triggers each skill, context lives in sessions, and there is no durable run
history or programme-level metrics.

**Gateflow** closes that gap. It is the **delivery control plane** — not a
coding agent — that resolves pinned `workflow.yaml`, dispatches eligible
`skill` nodes on agents we already use, stops where the contract assigns humans
or external actions, and measures outcomes.

**Autonomy destination:** agents do **most of the work between intentional
controls**. Human checkpoints stay **on by default** until RunStore metrics
justify lifting a stop or promoting a node to `dispatch: orchestrated` —
throttle after evidence, not before firsthand efficacy (§9, §12).

---

## 2. One-sentence product

> Gateflow watches GitHub, reads durable handoffs, resolves pinned
> `workflow.yaml`, and dispatches coding agents for eligible `skill` nodes —
> stopping wherever the delivery contract assigns humans or external actions —
> maximizing agent work between those intentional controls as metrics prove it
> safe.

---

## 3. What we are (and are not)

| We are | We are not |
|--------|------------|
| Workflow engine for `sdd-delivery/v2` | A replacement for prayog-skills or workflow.yaml |
| Dispatcher for Cursor / OpenCode / Claude Code (over time) | A new coding agent or IDE |
| Run store, metrics, and ops visibility (Mission Control) | Auto-merge or auto board updates **by default** |
| Pluggable tool enabler (codegraph, scanners, …) | A Graphify fork or mandatory tool stack |
| Forge API client (GitHub App; cloud-safe) | `gh` CLI or PE laptop for production GitHub ops |
| Programme factory citizen (launchpad, meta, catalog) | A merge of platform repos into meta |
| Evidence-gated autonomy (lift controls when green) | Blind copy of “N% agent PRs” without programme gates |

**Principle:** Process lives in **workflow + skills**. Execution lives in
**agents**. Orchestration and observability live in **Gateflow**. Intentional
human controls stay until **metrics** earn the right to widen automation.

---

## 4. Programme map

```text
                    PRAYOG PROGRAMME
                         prayog-meta
              (PM lane · vision · PRDs · factory config)
                              │
         ┌────────────────────┼────────────────────┐
         │                    │                    │
    prayog-skills         launchpad            config/
    workflow SSOT         harness factory      service catalog
         │                    │                    │
         └────────────────────┼────────────────────┘
                              │
              ┌───────────────┴───────────────┐
              │                               │
         gateflow                      gateflow-ops
    orchestrator API                  ops console (BFF)
    webhooks · runs · dispatch        runs · status · drill-down
```

| Repo | Role |
|------|------|
| **prayog-meta** | Vision, INIT PRDs, impact maps, programme config |
| **prayog-skills** | Skills, `workflow.yaml`, delivery contract SSOT |
| **launchpad** | Harness sync, verify, playbook |
| **gateflow** | Runtime control plane (Python / FastAPI) |
| **gateflow-ops** | Operations visibility (Next.js BFF) |

Platform repos stay separate; meta **governs** via config and registry only.

---

## 5. Delivery model — contract-driven navigation

Gateflow does **not** define the delivery process. Navigation is SSOT in
prayog-skills [`workflow.yaml`](../prayog-skills/workflow.yaml) under contract
`sdd-delivery/v2`. Gateflow loads the pinned workflow, reads the latest
**handoff envelope** from durable artifacts, and resolves the next node — the
same rules PE and agents follow today (see prayog-skills
`references/handoff-envelope.md`).

### Node types and Gateflow behavior

| `workflow.yaml` | Gateflow behavior |
|-----------------|-------------------|
| `type: skill` + `dispatch: orchestrated` | May dispatch agent when PE trigger + handoff authorize |
| `type: skill` + `dispatch: manual` | **Stop** dispatch; observe metrics from handoff |
| `type: skill` + `dispatch: observed` `[TBD]` | **Stop** dispatch; metrics-only observation |
| `type: human-checkpoint` | **Stop** — never auto-transition |
| `type: external-action` | **Stop** — never execute without explicit authorization |
| `type: decision` | **Stop** — human or policy owns the transition |
| `type: terminal` | End run |

The **`dispatch`** field on skill nodes is SSOT in prayog-skills (rc-2,
[INIT-PRAYOG-SKILLS-002-outline](../prd/INIT-PRAYOG-SKILLS-002-outline.md)).
Gateflow **must not** hardcode automatable node id lists. **`type`** governs
stops; **`dispatch`** governs orchestration eligibility.

Contract principles in `delivery-contract.yaml` are invariants, not
Gateflow policy:

- `human-gates-never-auto-transition`
- `external-writes-require-authorization`
- `navigation-is-read-only`
- `artifacts-not-chat-are-durable-state`

Human-owned nodes (`gate-1`, `gate-2`, `wave-human-decision`,
`requirements-human-decision`, `spec-merge`, `prd-merge`, …) are **workflow
node ids** with the types above — Gateflow honors them; it does not maintain a
parallel gate list. When the workflow changes, prayog-skills owns the edit;
Gateflow picks it up from the programme pin.

### MVP dispatch scope (operational entry, not a parallel flow)

Horizon 1 dispatches **`type: skill` nodes with `dispatch: orchestrated`**
once PE signals wave execution via programme trigger config (rc-2 pin). All
`dispatch: manual` skills and non-`skill` nodes remain outside automated
dispatch.

Illustrative rc-2 policy (prayog-skills owns values):

```text
… board-seed (skill, dispatch: manual)
     → pre-implement → loop-spec ⇄ verify → ground-spec  (dispatch: orchestrated)
     → wave-human-decision (human-checkpoint — STOP)
```

- Retry within budget on `findings` transitions the workflow already defines
  (**programme config**, default: 3 — not hardcoded in orchestrator source)
- GitHub comment trail as Phase 0 UI
- Never auto-pass any `human-checkpoint` node

### Trigger model (pilot)

PE authorizes a wave run with an explicit label (e.g. `gateflow:run-w0` — **programme
config**, not a code constant) when handoff preconditions are satisfied. Gateflow
does not infer intent from board state alone. Board moves, PR review, and merges
remain human or `external-action` steps per workflow — not Gateflow-invented gates.

---

## 6. Architecture — extensible control plane

Gateflow is built as **slots**, not a monolith. MVP ships with defaults; future
capabilities plug in without rewriting the core. **Pluggability rule:** slot
implementations must not embed workflow node ids, gate label strings, or dispatch
allowlists — those come from pinned prayog-skills contract + programme config.

```text
┌─────────────────────────────────────────────────────────────┐
│ CONTROL PLANE (Gateflow)                                     │
│  WorkflowEngine · RunStore · HandoffReader · PolicyEngine    │
│  TriggerRouter · MetricsEmitter · StageToolResolver          │
└──────────────┬──────────────────────────────────────────────┘
               │ pluggable slots (MVP: single impl or none)
┌──────────────▼──────────────────────────────────────────────┐
│ AgentRunner      cursor → opencode → claude-code (roadmap)  │
│ ModelGateway     cursor-native → litellm proxy (roadmap)      │
│ ToolProvider     none → graphify → mcp … (roadmap, / stage) │
│ ForgeClient      github-api (App token) — NOT gh CLI        │
│ Notifier         github-comment (H1) → slack / teams (H2+) │
│ WorkspacePrep    worktree + launchpad sync-harness            │
└─────────────────────────────────────────────────────────────┘
```

Three scale dimensions are **independent slots** — runner, tools, and forge —
plus **Notifier** for human-visible alerts. H2 OpenCode / Graphify / Slack do
not rewrite orchestration core.

### AgentRunner scale path

All coding agents implement the same dispatch contract: `workflow_node`, skill
prompt, prepared workspace, `model_profile`, optional `tool_context` →
`RunResult` with `runner`, `model_id`, outcome. Control plane code does not
branch on vendor beyond the adapter.

| Adapter | Horizon | Runs in |
|---------|---------|---------|
| Cursor SDK | H1 | Local dev or container worker |
| OpenCode | H2 | Container / cloud agent pod |
| Claude Code | H2 | Container / CLI agent worker |

Telemetry always records `runner + model_id` per orchestrated stage (§12).

### GitHub integration — ForgeClient (cloud-safe)

Gateflow runs in **Docker / cloud** for programme rollout. Production paths
**must not depend on the `gh` CLI** or a PE laptop session.

| Direction | Mechanism |
|-----------|-----------|
| **Inbound** | GitHub App **webhooks** (labels, PR, issue) → TriggerRouter |
| **Outbound** | **ForgeClient** — GitHub REST/GraphQL via App installation token (preferred) or scoped PAT (pilot only) |

**ForgeClient** (forge adapter slot) owns all **GitHub platform** writes.
**Notifier** is a separate fan-out slot for run progress — default H1 impl posts
via ForgeClient to PR/issue comments; future impls may push to **Slack**, **Teams**,
or other chat systems without changing WorkflowEngine or AgentRunner.

| Notifier backend | Horizon | Transport |
|------------------|---------|-----------|
| GitHub comment (via ForgeClient) | H1 | PR/issue thread |
| Slack | H2+ | Webhook / bot API |
| Microsoft Teams | H2+ | Webhook / bot API |

Notifier receives structured run events (run started, stage complete, stopped at
checkpoint, failed) from MetricsEmitter — not ad hoc strings from agents.

| Outbound action (allowed) | API | Notes |
|---------------------------|-----|-------|
| Run progress comments on PR/issue | Issues/PR comments API | Phase 0 UI (FR-9) |
| Open/update **wave PR** (agent pushed branch) | Pulls API | Operational; not auto-merge |
| Apply run status labels (ack, failed, …) | Labels API | Programme-defined; not gate labels |
| Commit status checks (optional) | Statuses API | Phase 0 evidence `[TBD]` |
| Observe contract labels (from pinned `delivery-contract.yaml`) | Webhook + API read | Metrics only — not hardcoded label strings |

| Outbound action (forbidden in MVP) | Why |
|------------------------------------|-----|
| Auto-merge PRs | `external-action` / human accountability |
| Set gate approval labels (contract `github.labels`) | Human gate — observe only |
| Auto-move programme / project board columns | PE-owned; H4 opt-in at earliest |

**Git in workspace** (clone, fetch, push from agent) uses **deploy keys** or
fine-grained tokens on the worker — separate from ForgeClient platform API auth.

### Stage-scoped tool slots (design now, plug later)

Tools attach to **workflow stages**, not ad hoc. **Node → tool slot** mapping is
**programme config or future workflow metadata** — not a hardcoded table in
Gateflow source. Slot names are adapter contract identifiers; H1 all providers =
`none`.

Illustrative slot names (examples only):

| Example stage | Example slot id | MVP |
|---------------|-----------------|-----|
| Any skill node | `upstream_tools`, `spec_tools`, … | none (parked) |
| Wave skill nodes | `prep_tools`, `implement_tools`, `verify_tools`, … | none |

Skills remain SSOT for **procedure**. Gateflow **injects tool context** at
dispatch when a programme enables a provider (`StageToolResolver` →
`ToolProvider`). Skill **enhancements** to consume Graphify, scanners, or MCP
tools are a **separate, parked** prayog-skills initiative — Gateflow supplies
the hook; skills team updates SKILL.md when ready.

```text
dispatch skill(node)
  → StageToolResolver (slot from config / workflow metadata)
  → ToolProvider (e.g. graphify): materialise graph in workspace
  → AgentRunner: skill runs with tool_context attached
```

### Deployment target

H1 targets **containerised** Gateflow API + worker (Docker; cloud VM or
orchestrator TBD in spec). Secrets: GitHub App key, Postgres URL, deploy keys
for repo git, runner credentials (Cursor SDK / later OpenCode). No requirement
for `gh`, IDE, or PE local machine in the execution path.

### Architectural influences (OSS patterns, not forks)

| Pattern | Reference | Gateflow use |
|---------|-----------|--------------|
| Run persistence + resume + event log | GitHub Spec Kit workflows | RunStore, audit trail |
| Deterministic routing (no LLM in orchestration) | Microsoft Conductor | PolicyEngine / resolver |
| Runner + middleware | Open SWE | AgentRunner adapters |
| Plugin registry | Osmia | ToolProvider, Notifier slots |
| Worktree isolation | orchestrator-sh | WorkspacePrep |
| Forge API auth | GitHub App installations | ForgeClient; no `gh` in cloud |

Full peer comparison and moat analysis: [§13 Market landscape](./gateflow-programme-vision.md#13-market-landscape-and-differentiation).

We **implement** Prayog-native execution over pinned `workflow.yaml` — we do not
fork Spec Kit or embed LangGraph as the product core.

---

## 7. Agent and model strategy

Gateflow selects **runner + model profile** per skill dispatch. Resolved model
identity is **always recorded** on orchestrated stage events for efficacy analysis.

| Horizon | Runners | Models | Config |
|---------|---------|--------|--------|
| **1 — MVP** | Cursor (SDK adapter) | Cursor-supported (`auto`, named models e.g. GLM, …) | Programme default + optional **per-`workflow_node` overrides** in meta/harness config |
| **2 — Scale** | + OpenCode, Claude Code | Per-skill profiles per runner | Same resolution rules via ModelGateway slot |
| **3 — Gateway** | Same runners | LiteLLM (or equivalent) proxy for non-Cursor runners | Central gateway; telemetry keeps normalised `model_id` |

**Model efficacy (programme learning):** compare duration, first-pass `pass` rate,
and findings density grouped by `workflow_node × model_id × runner` — inputs for
programme model tuning, not workflow changes.

Cursor constraints stay explicit in programme config; OpenCode and gateway
abstraction extend the same **AgentRunner + ModelGateway** slots without forking
orchestration logic.

---

## 8. Tooling and grounding (parked)

We are **not** plugging Graphify (or any codegraph) in MVP. Architecture must
support providers at stage slots as Gateflow matures.

| Hook | Purpose | Status |
|------|---------|--------|
| engg-reviews / meta PR | PRD ↔ codebase map (upstream quality) | Experimental in prayog-skills; parallel to SDD |
| Gateflow prep / implement | Fresh graph in agent workspace | Parked — `ToolProvider: none` in MVP |
| Skill enhancements (loop-spec, ground-spec) | Structural evidence in reports | Parked — separate skills initiative |

Provider interface aligns with prayog-skills `codegraph-provider` contract when
implemented — skills and Gateflow share semantics, not duplicate logic.

---

## 9. Programme maturity and lift-on-metrics

World-class teams (golden path / paved road → harness → agentic factory) do
**not** maximize unattended agents on day one. Knowledge bakes into the path;
automation widens when the path is trustworthy.

### Maturity stages

```text
A — Greenfield paved road     Launchpad scaffolds + harness install
B — Harnessed manual          Skills + humans; agents as copilots
C — Ripe for automation       Specs, tests, constitutions durable; dispatch gaps hurt
D — Fleet Mission Control     Gateflow operates onboarded repos; lift on metrics
```

| Stage | Owner | Gateflow role |
|-------|-------|---------------|
| **A Greenfield** | **Launchpad** | None — do not create repos from Gateflow |
| **B Harnessed manual** | Eng + skills pin | Optional observe; PE still drives skills |
| **C Ripe** | Programme | Orchestrate eligible nodes; collect efficacy |
| **D Fleet Mission Control** | gateflow + gateflow-ops | Onboard harness-enabled repos; cockpit; promote nodes when green |

Brownfield **onboarding** (already harness-enabled) is a Gateflow/ops concern.
Greenfield **creation** stays Launchpad.

### Autonomy policy (locked 2026-07-27)

| Stance | Rule |
|--------|------|
| **Destination** | Agents do **most of the work between intentional controls** |
| **Default today** | Human checkpoints and conservative `dispatch: manual` where efficacy is unproven |
| **Lift rule** | Widen `orchestrated` surface or reduce mid-chain PE only when **RunStore metrics** justify it |
| **Never silent** | No quiet removal of gates; promotion is an explicit pin / programme decision |
| **Merge** | Human-accountable merge remains default; policy-gated auto-merge is H4+ only if opted in |

### Promotion criteria (programme learning — thresholds TBD in ops)

A node class or lane may move toward more automation when, over an agreed window:

1. **Unattended hop success** — orchestrated stages complete without mid-chain PE skill invoke  
2. **Rework / findings rate** — below programme threshold  
3. **Fail-fast honesty** — no silent fake success on live runners  
4. **Gate dwell** — time on `human-checkpoint` is mostly judgment, not distrust of the prior hop  

Mission Control exists to make these signals visible so PE can **decide** lifts —
not to replace judgment with a dashboard.

---

## 10. Roadmap horizons

Horizons are **direction**, not committed release dates.

### Horizon 1 — Execution engine (MVP) — largely achieved (001–003)

- GitHub App webhooks + **ForgeClient** (REST/GraphQL outbound; **no `gh` CLI**)
- Run store (PostgreSQL), handoff parser, workflow resolver, API wave-start
- **Live** Cursor SDK **AgentRunner** (local worker); stubs fail-closed for other runners
- ToolProvider slots unused (`none`); Notifier = GitHub comments
- Metrics: stage + wave cycle-time fields; runner / model dims on orchestrated stages
- **Implement-lane** live prove-it done; **spec-lane** prove-it = 003 W2 + dogfood PRD
- gateflow-ops: APIs exist; **UI Mission Control = H2 / INIT-004**

### Horizon 2 — Mission Control, efficacy, multi-runner

- **gateflow-ops Mission Control** — onboard harness-enabled repos; run cockpit
  (pin graph projection + node timeline + metrics + GitHub PR links)
- Expand **orchestrated** surface where metrics are green (spec-lane, then others)
- **OpenCode + Claude Code** AgentRunner adapters when Cursor efficacy is proven
- Optional Notifier backends (Slack, Teams); optional ToolProvider enablement
- Programme board **read** (visibility only; no auto-move)
- Workflow **projection** in DB for UI/history (not a second process SSOT)

### Horizon 3 — Intelligence and gateway

- LiteLLM (or equivalent) model gateway for non-Cursor runners
- Shared codegraph / MCP provider behind `ToolProvider`
- Cost attribution, quality scorecards, manual vs automated comparison
- Optional: Temporal-grade durability if fleet scale demands it
- Optional lab: graph experiment runtime (e.g. LangGraph) **explicitly non-SSOT**

### Horizon 4 — Policy and fleet (explicit non-goals until proven)

- Policy-gated auto-merge (only if programme explicitly opts in **after** long green metrics)
- PM-lane automation
- Cross-initiative queue optimization
- Graph studio that opens **prayog-skills PRs** (edit path = git, not silent DB mutation)

---

## 11. Dogfood strategy

```text
Phase A — Build Gateflow (manual / early SDD)           [largely done — 001–003]
  Control plane + API waves + live Cursor implement-lane

Phase B — Gateflow runs programme delivery (automated SDD)
  Spec-lane prove-it on a real next PRD (004 dogfood subject)
  Pin CTR-01: spec-lane skills dispatch: orchestrated
  Human checkpoints ON — collect firsthand efficacy metrics

Phase C — Fleet Mission Control + lift-on-metrics
  Onboard harness-enabled repos; operate from ops cockpit
  Promote nodes / reduce mid-chain PE only when metrics green

Phase D — Broader programme / client-style rollout
  Automation baseline + scorecards across repos
```

---

## 12. Success metrics

Metrics serve **operational control**, **contract profiling**, and **autonomy
promotion** (lift-on-metrics). All telemetry lands in **PostgreSQL** (RunStore).

### North-star (autonomy destination)

| Metric | Meaning |
|--------|---------|
| **Unattended hop rate** | Orchestrated stages completed without mid-chain PE skill invocation |
| **Agent-authored stage share** | Stages with live runner (e.g. `runner=cursor`) vs manual gaps |
| **Rework / findings rate** | Follow-up findings loops or fixups after agent stages |
| **Gate dwell time** | Time at `human-checkpoint` / `external-action` (judgment vs distrust) |
| **Contract-only stops** | Waves that stop only at typed contract nodes — not ad hoc PE rescue |

Cursor-class “~30% agent PRs” is an **aspirational industry signal**, not a
near-term Gateflow KPI. We adopt the **direction** (agents do the work between
controls) and require **firsthand green metrics** before lifting brakes.

### Skill and gate profiling (programme learning)

Record **start and end** per workflow node where observable:

| Category | Examples | Primary signals |
|----------|----------|-----------------|
| Upstream skills | `spec-draft`, `initiative-feasibility`, `board-seed` | Handoff envelope stage transitions; artifact digests |
| Gate intervals | Gate unlock labels from pinned `delivery-contract.yaml` | Contract `github.labels` + PR timeline; not hardcoded strings |
| Orchestrated skills | Any `skill` with `dispatch: orchestrated` on the run path | Gateflow dispatch lifecycle |
| Human checkpoints | Any node with `type: human-checkpoint` | Stop events from workflow resolution |
| Findings loops | Transitions on `findings` outcome per workflow graph | Outcome graph + programme retry budget |

Aggregates per skill node (p50/p95 duration, first-pass `pass` rate, findings
rate) compare **manual vs automated** waves and **across programmes** — inputs
for prayog-skills tuning **and** for lift decisions (§9).

**Model efficacy:** same aggregates **split by `runner`, `model_profile`, and
`model_id`** per `workflow_node` so programmes can tune cost/quality via
**programme config** when using Cursor SDK, OpenCode, or future gateway runners.

### Run metrics (Gateflow core)

- **Unattended completion rate** — wave path to next `human-checkpoint` without
  mid-chain PE skill invocation
- **Wave run duration** — trigger / API accept to stop at contract or terminal
- **Retry count** — `findings` cycles on workflow-defined loop paths per wave
- **Human intervention count** — should cluster at `human-checkpoint` and
  `external-action` nodes per contract

### Quality metrics (programme hypothesis)

- Verify first-pass rate
- Bugbot / review findings density per wave
- Merge without rework rate

### Benchmark framing

Compare against teams using Cursor-only or autonomous agents without SDD gates.
Gateflow wins on **governed automation + visibility + evidence-gated lift**, not
raw “fastest PR” or ungoverned agent-merge share.

---

## 13. Market landscape and differentiation

Research across 2025–2026 engineering-agent products shows convergent architecture:
**process SSOT in repo artifacts → control plane outside agents → human/external
stops → durable run state → pluggable runners**. Gateflow sits in Layer 3 of
that stack; it is not a coding agent and not a workflow SSOT replacement.

Industry maturity (golden path → harness → agentic factory; Cursor ~30–35%
internal agent-merged PRs as a **throttle** signal) informs our **destination**.
Enterprise governance (accountable merge, risk-tiered review) informs our
**brakes**. Gateflow builds both: maximize agent work between intentional
controls; lift controls only when metrics are green (§9).

```text
Layer 4 — Programme / factory     prayog-meta · launchpad · catalog
Layer 3 — Delivery control plane  Gateflow (+ peers below)
Layer 2 — Agent runtime           Cursor · OpenCode · OpenHands · Copilot SDK
Layer 1 — Model inference         Claude · GPT · Gemini · …
```

### Parallel products (selected)

| Product | Layer | Workflow / process SSOT | Human gates | Run persistence | Auto-merge default |
|---------|-------|-------------------------|-------------|-----------------|-------------------|
| **Gateflow** (Prayog) | Control plane | **prayog-skills `workflow.yaml`** + handoff envelope | `human-checkpoint` / `external-action` stop | RunStore + metrics v0 | **No** (MVP non-goal) |
| [OpenSymphony](https://opensymphony.dev/) | Control plane | Repo `WORKFLOW.md` + skills | Review in loop; orchestrator ≠ merge | Orchestrator scheduling SSOT | No |
| [GitHub Spec Kit](https://github.github.com/spec-kit/) workflows | SDD + engine | **Own** Spec Kit workflow YAML | `gate` step; `workflow resume` | `.specify/workflows/runs/` | No |
| [Microsoft Conductor](https://github.com/microsoft/conductor) | Multi-agent router | **Own** workflow YAML | `human_gate` step | CLI run state | N/A |
| [SpecOrch](https://github.com/fakechris/spec-orch) | SDD control plane | **Own** seven-plane model + gates | Gate-first; commit status | Missions / sessions | Configurable |
| [Optio](https://github.com/jonwiggins/optio) | K8s orchestrator | Task pipeline (not SDD contract) | Review loop; optional human | Reconciliation control plane | **Yes** (when CI + approval pass) |
| [Open SWE](https://github.com/langchain-ai/open-swe) | Agent product | LangGraph state machine | Plan approval HITL | LangGraph Platform | No (opens PR) |
| [Factory Missions](https://factory.ai/news/missions-architecture) | Software factory | Orchestrator-owned plan | Validators + user halt | Mission Control | Enterprise policy |
| [Cursor Cloud Agents](https://cursor.com/docs/cloud-agent) | Agent runtime | Automations (event → agent) | Approval Agents (optional) | Agent + run API | Optional via Approval Agents |
| [GitHub Copilot coding agent](https://docs.github.com/copilot/concepts/agents/cloud-agent) | Platform agent | GitHub PR workflow | **Cannot self-approve/merge** | Session + PR | No |

Sources are indicative (Jul 2026); links in References below.

### Industry pattern consensus

| Pattern | Gateflow |
|---------|----------|
| Control plane ≠ coding agent | Core positioning (§3) |
| Workflow/policy in versioned artifacts | Consumes prayog-skills SSOT — does not own graph |
| Human gates sacred at platform level | `human-gates-never-auto-transition` until lift criteria met |
| Durable handoffs, not chat | Handoff envelope + RunStore |
| Deterministic orchestration routing | PolicyEngine; no LLM in resolver |
| Pluggable agent runners | AgentRunner slot (Cursor → …) |
| Workspace / worktree isolation | WorkspacePrep slot |
| Programme metrics vs raw speed | Unattended hops, rework, gate dwell, wave duration |
| Evidence before autonomy | Lift-on-metrics (§9) |

### Where Gateflow differentiates

1. **Contract consumer, not process author** — Navigation follows pinned
   `workflow.yaml` node types (`skill`, `human-checkpoint`, `external-action`).
   Spec Kit, SpecOrch, and Conductor each **define** their own workflow. Prayog
   already invested in `sdd-delivery/v2`; Gateflow orchestrates that graph without
   SSOT drift.

2. **Programme factory citizen** — Harness sync (launchpad), service catalog,
   pinned skills release, meta governance. Peers are repo-local or single-product;
   none integrate at programme level.

3. **Governed automation hypothesis** — Optimise for unattended wave completion
   **within contract stops** and programme learning (manual vs automated baseline),
   not ticket-to-merged-PR autonomy (Optio) or ungoverned agent-merge share alone.

4. **Explicit non-goals as product choice** — No auto-merge, no auto board moves
   by default. Aligns with GitHub Copilot guardrails (“Copilot not Autopilot”);
   diverges from full-autonomy orchestrators by design until metrics earn H4.

5. **Slots without preset stack** — ToolProvider, ModelGateway, Notifier are
   extension points defaulting to `none` in MVP. Avoids mandatory codegraph or
   LangGraph-as-core (contrast Open SWE). Graph DBs / experiment runtimes stay
   **projections or labs**, not a second process SSOT.

### Landscape-informed horizon notes

| Signal from market | Gateflow response |
|--------------------|-------------------|
| Optio / OpenSymphony reconciliation loops | H1: idempotent webhooks + periodic resync |
| CI / review feedback resume | H2: workflow-valid resume on external signals (not ad hoc chat) |
| Ops UI / Factory Mission Control | H2: gateflow-ops Mission Control (INIT-004) |
| Multi-runner day one (Optio) | H1 Cursor live; H2 runners after Cursor efficacy |
| Cursor ~30–35% agent PRs | Destination signal; Gateflow KPI = unattended hops + lift-on-metrics |
| Auto-merge pressure (Optio, Cursor Approval Agents) | H4 opt-in only; never default until long green history |

**Verdict:** The market validates Gateflow’s layer and patterns. Differentiation
is **honoring existing Prayog workflow SSOT** plus **evidence-gated autonomy** —
not competing with Spec Kit or agent IDEs on process definition, and not copying
agent-merge share without brakes.

---

## 14. Initiative sequence

| Order | Artifact | Purpose |
|-------|----------|---------|
| 1 | This vision doc | Programme north star (planning/) — maturity + lift-on-metrics |
| 2 | Skills pin | INIT-PRAYOG-SKILLS-002 → **`v0.5.0-rc.2`** (`dispatch`) — delivered |
| 3 | Control plane | INIT-GATEFLOW-001 — delivered |
| 4 | API / platform readiness | INIT-GATEFLOW-002 — delivered (eng) |
| 5 | Live Cursor + lanes | INIT-GATEFLOW-003 — implement-lane done; **W2 spec-lane** with 004 dogfood |
| 6 | Mission Control | **INIT-GATEFLOW-004** — ops cockpit + onboard harnessed repos + efficacy visibility |
| 7 | Follow-on | Second runner, Slack, ToolProvider, gateway — as metrics and horizons allow |

---

## 15. Open decisions

| # | Decision | Lean |
|---|----------|------|
| 1 | Metrics SSOT | PostgreSQL events in Gateflow RunStore (all envs); export later |
| 2 | LiteLLM placement | Separate gateway service when Horizon 3 starts |
| 3 | Autonomy destination | **Resolved (2026-07-27)** — agents do most work between intentional controls; checkpoints default on; **lift-on-metrics** |
| 4 | prayog-skills pin | **`v0.5.0-rc.2`** with `dispatch` (**INIT-PRAYOG-SKILLS-002 delivered**) |
| 5 | ForgeClient auth in dev | App installation token only vs PAT for local pilot |
| 6 | `dispatch: observed` enum | Deferred from rc-2 v1 — intentional; not a Gateflow delivery blocker |
| 7 | Joint Gate 1 / pin timing | **Resolved** — pin **`v0.5.0-rc.2`** delivered; Gateflow unblocked |
| 8 | Numeric lift thresholds | TBD after firsthand 003 W2 / 004 efficacy samples in Mission Control |
| 9 | Greenfield vs onboard | **Resolved** — Launchpad creates; Gateflow onboards harness-enabled repos |
| 10 | First Mission Control INIT | INIT-GATEFLOW-004 outline next (after this vision bump) |

---

## References

### Programme

- Workflow SSOT: prayog-skills [`workflow.yaml`](../prayog-skills/workflow.yaml)
- Delivery contract: prayog-skills [`delivery-contract.yaml`](../prayog-skills/delivery-contract.yaml) (`sdd-delivery/v2`)
- Handoff navigation: prayog-skills `references/handoff-envelope.md`
- Workflow dispatch policy: [INIT-PRAYOG-SKILLS-002](../prd/INIT-PRAYOG-SKILLS-002.md) (pin **`v0.5.0-rc.2`**)
- Service catalog: `config/service-catalog-drivestream-lab.yaml`
- Engg-reviews / codegraph plan: prayog-skills `docs/engg-reviews-implementation-plan.md`

### Market landscape (Jul 2026)

- [GitHub Spec Kit](https://github.github.com/spec-kit/) — SDD toolkit + workflow engine with gate steps and run resume
- [Microsoft Conductor](https://github.com/microsoft/conductor) — deterministic YAML multi-agent orchestration
- [OpenSymphony](https://opensymphony.dev/) — Symphony-model orchestrator; repo `WORKFLOW.md` SSOT
- [SpecOrch](https://github.com/fakechris/spec-orch) — spec-first delivery control plane with gate layer
- [Optio](https://github.com/jonwiggins/optio) — self-hosted K8s agent orchestrator; ticket-to-merge loop
- [Open SWE](https://github.com/langchain-ai/open-swe) — LangGraph-based async coding agent
- [Factory Missions](https://factory.ai/news/missions-architecture) — long-horizon multi-agent software factory
- [Cursor Cloud Agents](https://cursor.com/docs/cloud-agent) — background agents + Automations API
- [Keviq Core](https://github.com/Keviqai/keviq-core) — agent control-plane infrastructure (approval gates, lineage)
- Industry framing: [How AI Agents Are Reshaping Software Delivery in 2026](https://hackernoon.com/how-ai-agents-are-reshaping-software-delivery-in-2026) (HackerNoon)
- Autonomy signal: Cursor cloud agents ~30–35% internal agent-merged PRs (2026) — destination throttle; Gateflow lifts on metrics (§9, §12)
- Platform maturity: golden path / paved road (Spotify, Netflix, Google) — greenfield factory vs brownfield onboard
