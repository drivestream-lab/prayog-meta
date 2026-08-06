# Gateflow programme vision

**Programme:** prayog · **Org:** drivestream-lab · **Status:** draft  
**Last updated:** 2026-08-01  
**Related initiatives:** [INIT-GATEFLOW-001](../prd/INIT-GATEFLOW-001.md) ·
[INIT-PRAYOG-SKILLS-002](../prd/INIT-PRAYOG-SKILLS-002.md) · gateflow repo specs
`docs/specification/product/INIT-GATEFLOW-00{1–8}-*.md`

> **2026-08-01 refresh** — Bridges the Jul-22 vision with as-built Gateflow
> specs (001–008): API-first wave starts, pin `authorization` on
> `external-action`, forge publish/mutate, Pass-1/Pass-2 closeout, and a
> **Horizon 1.5** factory slice between MVP engine and ops/multi-runner (H2).
> North star unchanged: contract consumer, not process author; humans keep
> high-value gates; no auto-merge.

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
`skill` nodes on agents we already use, applies pin-declared forge side effects
via ForgeClient, stops where the contract assigns humans or **explicit**
external actions, and measures outcomes (including structured learning ingest).

---

## 2. One-sentence product

> Gateflow accepts PE-authorized lane starts, reads durable handoffs, resolves
> pinned `workflow.yaml`, dispatches coding agents for eligible `skill` nodes,
> and applies pin-declared forge actions — stopping at human checkpoints and
> every `external-action` the pin marks `authorization: explicit`.

---

## 3. What we are (and are not)

| We are | We are not |
|--------|------------|
| Workflow engine for `sdd-delivery/v2` | A replacement for prayog-skills or workflow.yaml |
| Dispatcher for Cursor / OpenCode / Claude Code (over time) | A new coding agent or IDE |
| Run store, metrics, learning ingest, and ops visibility | Auto-merge or worker-driven board column moves |
| Pin-driven forge publish/mutate (ForgeClient) | Skill/`gh` as production forge success path |
| Pluggable tool enabler (codegraph, scanners, …) | A Graphify fork or mandatory tool stack |
| Forge API client (GitHub App; cloud-safe) | `gh` CLI or PE laptop for production GitHub ops |
| Programme factory citizen (launchpad, meta, catalog) | A merge of platform repos into meta |

**Principle:** Process lives in **workflow + skills**. Execution lives in
**agents**. Orchestration, forge side effects (when the pin allows), and
observability live in **Gateflow**.

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
**handoff envelope** from durable artifacts (run-scoped baton path for
packaged automated skills), and resolves the next node — the same rules PE and
agents follow today (see prayog-skills `references/handoff-envelope.md` and
`docs/for-gateflow.md`).

### Node types and Gateflow behavior

| `workflow.yaml` | Gateflow behavior |
|-----------------|-------------------|
| `type: skill` + `dispatch: orchestrated` | May dispatch agent when PE-authorized start + bind/handoff allow |
| `type: skill` + `dispatch: manual` | **Stop** dispatch; observe metrics from handoff |
| `type: skill` + `dispatch: observed` `[TBD]` | **Stop** dispatch; metrics-only observation |
| `type: human-checkpoint` | **Stop** — never auto-transition |
| `type: external-action` + `authorization: explicit` | **Stop** — mutate only after programme authorize (or dual-executor human forge skill) |
| `type: external-action` + `authorization: automated` | **Apply** pin `forge` via ForgeClient when `requires` complete; no interactive STOP; continue on `pass` |
| `type: decision` | **Stop** — human or policy owns the transition |
| `type: terminal` | End run |

**Pin fields Gateflow honors (never hardcodes node id allowlists):**

| Field | Owns | Gateflow use |
|-------|------|--------------|
| `type` | prayog-skills | Stop vs walk semantics |
| `dispatch` on skills | prayog-skills ([INIT-PRAYOG-SKILLS-002](../prd/INIT-PRAYOG-SKILLS-002.md)) | AgentRunner eligibility |
| `authorization` on external-actions | prayog-skills (`explicit` \| `automated`; required) | ForgeClient apply vs STOP+authorize |
| `forge` / forge-side-effects | prayog-skills | Workspace publish + mutate vocabulary |

Contract principles in `delivery-contract.yaml` remain invariants:

- `human-gates-never-auto-transition`
- `external-writes-require-authorization` — satisfied by pin `authorization`
  (programme authorize for `explicit`; pin-declared automation for `automated`)
- `navigation-is-read-only`
- `artifacts-not-chat-are-durable-state`

Human-owned stops are **pin node ids** (e.g. `live-verify`, `wave-signoff`,
`prd-impact-acceptance`, `coding-readiness`, merge/`*-lgtm` paths) — Gateflow
honors resolved ids; it does not maintain a parallel gate list and must not
hardcode retired names (`gate-1`, `gate-2`, `wave-human-decision`, …) as live
transitions. When the workflow changes, prayog-skills owns the edit; Gateflow
picks it up from the programme pin.

### Wave lifecycle (Pass-1 / Pass-2)

Implement and spec waves share a two-pass shape (pin owns edges; Gateflow walks):

```text
Pass-1 (lane start API)
  implement: pre-implement → loop-spec → wave-pr-action (automated) → live-verify STOP
  spec:      spec-draft → … → spec-pr-action (automated when pin says so) → human ADR / gates
  park:      wave-awaiting-closeout is status/UI — does not auto-dispatch closeout

Pass-2 (closeout start API — new run, same wave PR)
  learning-extract → ground-spec → wave-signoff STOP
  Learning-Extract YAML ingested to PostgreSQL (programme learning SSOT; skill ≠ HTTP)
```

Content skills **never** mutate forge. Workspace publish (`forge.commit_workspace`)
and Draft PR open (`open_draft_pr`) are ForgeClient-owned when the pin enables them.
Board seed / PRD / merge external-actions stay **`authorization: explicit`** on the
day-one pin — humans (or programme authorize) keep those seats.

### Dispatch scope (operational entry, not a parallel flow)

Gateflow dispatches **`type: skill` nodes with `dispatch: orchestrated`** after a
PE-authorized **lane start** (implement / spec / closeout). All `dispatch: manual`
skills remain outside automated AgentRunner dispatch until the pin promotes them
(spec-lane orchestration is a pin dependency, not a Gateflow invention).

Illustrative implement Pass-1 (prayog-skills owns values):

```text
… board-seed / board-tickets (external-action, authorization: explicit)
     → pre-implement → loop-spec  (skill, dispatch: orchestrated; publish required)
     → wave-pr-action (external-action, authorization: automated — ForgeClient opens Draft PR)
     → live-verify (human-checkpoint — STOP)
     → [PE closeout start] learning-extract → ground-spec → wave-signoff (STOP)
```

- Retry within budget on `findings` transitions the workflow already defines
  (**programme config**, default: 3 — not hardcoded in orchestrator source)
- Milestone Notifier comments on the wave PR (sparse hop chatter; full timeline in RunStore)
- Never auto-pass any `human-checkpoint` node
- Never auto-merge; never write `*-lgtm` or invent PE gate labels

### Trigger model (current)

**Primary:** authenticated **lane start APIs** with programme service token —

- `POST /api/v1/waves/implement/start`
- `POST /api/v1/waves/spec/start` (meta PR accept + dual workspace bind)
- `POST /api/v1/waves/closeout/start` (fixed Enter-at `learning-extract`)

PE supplies wave identity, workspace targeting, runner/model, and (for implement)
board/ticket fields. Gateflow does **not** infer run intent from board columns.

**Secondary / legacy:** GitHub App webhooks remain for ingress and future signals;
**label-as-wave-start** (INIT-001 `gateflow:run-wave`) is **not** the current
programme start path — restored only if programme config re-enables it.

Board moves, PR review, merges, and other `authorization: explicit` forge actions
remain human or authorize-API steps per pin — not Gateflow-invented gates.

---

## 6. Architecture — extensible control plane

Gateflow is built as **slots**, not a monolith. H1 shipped Cursor + GitHub
Notifier defaults; H1.5 adds pin-driven Forge publish/mutate without rewriting
the walker core. **Pluggability rule:** slot implementations must not embed
workflow node ids, gate label strings, dispatch allowlists, or forge
authorization policy — those come from the pinned prayog-skills contract (+
wave-start / env for runner selection and secrets).

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

Telemetry always records `runner + model_id` per orchestrated stage (§10).

### GitHub integration — ForgeClient (cloud-safe)

Gateflow runs in **Docker / cloud** for programme rollout. Production paths
**must not depend on the `gh` CLI** or a PE laptop session.

| Direction | Mechanism |
|-----------|-----------|
| **Inbound** | Lane start / closeout / authorize **APIs** (programme token); GitHub App **webhooks** for events (label start optional/legacy) |
| **Outbound** | **ForgeClient** — GitHub REST/GraphQL via App installation token (preferred) or scoped PAT (pilot only) |

**ForgeClient** (forge adapter slot) owns all **GitHub platform** writes —
including pin-driven workspace publish (`commit_workspace` to run head) and
mutate actions (`open_draft_pr`, `create_board_tickets`, …). Content AgentRunner
hops must not succeed via `git commit` / `gh`. **Notifier** is a separate fan-out
slot for run progress — default posts sparse milestone comments via ForgeClient;
future impls may push to **Slack** / **Teams** without changing WorkflowEngine.

| Notifier backend | Horizon | Transport |
|------------------|---------|-----------|
| GitHub comment (via ForgeClient) | H1 / H1.5 | PR/issue thread (milestone events) |
| Slack | H2+ | Webhook / bot API |
| Microsoft Teams | H2+ | Webhook / bot API |

Notifier receives structured run events from MetricsEmitter — not ad hoc strings
from agents. Stage hop chatter stays primarily on the RunStore timeline.

| Outbound action (allowed) | API | Notes |
|---------------------------|-----|-------|
| Milestone run progress comments | Issues/PR comments API | Sparse; timeline is SSOT for hops |
| **Publish** workspace paths to run head | Git data API (blobs/tree/commit/ref) | Pin `forge.commit_workspace`; before handoff ingest |
| Open Draft **wave/spec PR** when pin `authorization: automated` | Pulls API | After coding hops; not PR-at-start; not auto-merge |
| Open/update PR after **explicit** authorize | Pulls API | Same ForgeClient path; programme authorize API |
| Board ticket seed (EPIC/Feature) after **explicit** authorize | Issues/Projects API | Dumb primitives; worker must not board-mutate on hop complete |
| Apply **projection** labels from pin/Launchpad vocabulary | Labels API | Never invent `*-lgtm` or Gateflow-only PE gates |
| Observe contract labels (from pinned `delivery-contract.yaml`) | Webhook + API read | Metrics only — not hardcoded label strings |

| Outbound action (forbidden) | Why |
|-----------------------------|-----|
| Auto-merge PRs | Human / `explicit` merge accountability; H4 opt-in at earliest |
| Set gate approval labels (`*-lgtm`, contract approval labels) | Human gate — observe only |
| Auto-move programme / project board columns from the wave worker | PE-owned; board APIs are callable primitives, not walker side effects |
| Env/API flag that overrides pin `authorization: explicit` → automated | Pin remains policy SSOT |

**Git in workspace** (clone, fetch; agent local edits) uses **deploy keys** or
fine-grained tokens on the worker — separate from ForgeClient platform API auth.
Durable tip for the wave is ForgeClient publish to the **run head**, not skill `git push` as success.

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

Full peer comparison and moat analysis: [§11 Market landscape](./gateflow-programme-vision.md#11-market-landscape-and-differentiation).

We **implement** Prayog-native execution over pinned `workflow.yaml` — we do not
fork Spec Kit or embed LangGraph as the product core.

---

## 7. Agent and model strategy

Gateflow selects **runner + model profile** per skill dispatch. Resolved model
identity is **always recorded** on orchestrated stage events for efficacy analysis.

| Horizon | Runners | Models | Config |
|---------|---------|--------|--------|
| **1 — MVP** | Cursor (live SDK adapter) | Cursor-supported (`auto`, named models, …) | **Wave-start dispatch plan** (`runner` / `model_id` + optional per-node map); fail-closed stubs for non-live runners |
| **1.5 — Factory** | Same | Same | Unchanged resolution; telemetry on every orchestrated hop |
| **2 — Scale** | + OpenCode, Claude Code (live) | Per-skill profiles per runner | Same rules; stubs become live adapters |
| **3 — Gateway** | Same runners | LiteLLM (or equivalent) proxy for non-Cursor runners | Central gateway; telemetry keeps normalised `model_id` |

**Model efficacy (programme learning):** compare duration, first-pass `pass` rate,
and findings density grouped by `workflow_node × model_id × runner` — inputs for
programme model tuning, not workflow changes.

OpenCode and gateway abstraction extend the same **AgentRunner + ModelGateway**
slots without forking orchestration logic.

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

## 9. Roadmap horizons

Horizons are **direction**, not committed release dates. Status reflects
gateflow as-built + product specs as of **2026-08-01**.

### Horizon 1 — Execution engine (MVP) — **delivered (core)**

Specs: INIT-GATEFLOW-001…003 (+ BOUNDINPUT substrate 005 W0/W1).

- GitHub App webhooks + **ForgeClient** (REST/GraphQL outbound; **no `gh` CLI**)
- Run store (PostgreSQL), handoff parser, workflow / PolicyEngine
- Live Cursor SDK **AgentRunner**; adapter registry with OpenCode/Claude **stubs** (fail-closed)
- ToolProvider slots conceptually reserved; providers remain `none`
- Retry budget; programme-token status + metrics APIs (gateflow-ops BFF still deferred)
- Container/cloud-ready API + worker path
- Metrics v0: stage events with **runner + model_id**; wave `duration_ms`
- Packaged skill bind: pin `prompts/` + `schema.yaml`, thin `{{var}}`, run-scoped `handoff_path`

### Horizon 1.5 — Delivery factory (pin forge + lanes) — **in flight**

Specs: INIT-GATEFLOW-006, 008 (006A), 007. Extends H1 without jumping to H2 ops UI.

- Pin `forge:` workspace **publish** and **mutate** via ForgeClient (ADR-009)
- Pin `authorization: explicit | automated` on every `external-action` (008)
- Implement/spec **lane start** APIs + meta PR dual-workspace intake (ADR-010)
- Pass-1 Draft PR after coding (`wave-pr-action` / `spec-pr-action` automated)
- Pass-2 **closeout** start + Learning-Extract → Postgres ingest (007)
- Board APIs as dumb ForgeClient primitives (worker isolation retained)
- WorkManifest `prayog/v1` validation before explicit board create
- **Exit (feature):** Pass-1 walker + automated forge + closeout/learning paths
  unit- and contract-stable; live programme exercise deferred until PE declares
  the factory slice stable

### Horizon 2 — Observability and multi-runner

- **gateflow-ops** UI (runs, waves, drill-down) — consumes status/metrics JSON
- Live **OpenCode + Claude Code** AgentRunner adapters (stubs already registered)
- **Optional ToolProvider** enablement (Graphify / MCP / scanners per stage)
- **Optional Notifier** backends (Slack, Teams)
- Spec-lane full orchestration when pin promotes `dispatch: orchestrated` on spec skills
- Programme board **read** / richer visibility (still no worker auto-move)

### Horizon 3 — Intelligence and gateway

- LiteLLM (or equivalent) model gateway for non-Cursor runners
- Shared codegraph / MCP provider behind `ToolProvider`
- Cost attribution, quality scorecards, manual vs automated comparison
- Mission Control–style learning browse (beyond DB ingest)
- Optional: Temporal-grade durability if fleet scale demands it

### Horizon 4 — Policy and fleet (explicit non-goals until proven)

- Policy-gated auto-merge (only if programme explicitly opts in)
- PM-lane automation
- Cross-initiative queue optimization
- Authorize→resume walker shortcuts that blur Pass-1 / Pass-2 Enter-at contracts

---

## 10. Success metrics

Metrics serve **operational control** (wave runs) and **contract profiling**
(every `workflow.yaml` skill and gate interval) so the programme can enhance
prayog-skills with evidence. All telemetry lands in **PostgreSQL** (RunStore).

### Skill and gate profiling (programme learning)

Record **start and end** per workflow node where observable:

| Category | Examples | Primary signals |
|----------|----------|-----------------|
| Upstream / spec skills | `spec-draft`, `initiative-feasibility`, … | Handoff stage transitions; artifact digests |
| Gate intervals | Unlock labels from pinned `delivery-contract.yaml` | Contract `github.labels` + PR timeline; not hardcoded strings |
| Orchestrated skills | Any `skill` with `dispatch: orchestrated` | Gateflow dispatch lifecycle + prompt_id/revision |
| Automated forge | `wave-pr-action`, `spec-pr-action` (`authorization: automated`) | ForgeClient apply + continue |
| Explicit forge | board / PRD / merge external-actions | STOP + authorize (or human forge skill) |
| Human checkpoints | `live-verify`, `wave-signoff`, … | Stop events from workflow resolution |
| Findings loops | Transitions on `findings` outcome | Outcome graph + programme retry budget |
| Learning items | Learning-Extract `L-*` taxonomy | Postgres ingest after closeout (not git SSOT) |

Aggregates per skill node (p50/p95 duration, first-pass `pass` rate, findings
rate) compare **manual vs automated** waves and **across programmes** — inputs
for prayog-skills tuning, not Gateflow process changes.

**Model efficacy:** same aggregates **split by `runner`, `model_profile`, and
`model_id`** per `workflow_node` so programmes can tune cost/quality via
**wave-start dispatch plan** (and later programme config / gateway) when using
Cursor SDK, OpenCode, or future runners.

### Run metrics (Gateflow core)

- **Unattended Pass-1 completion rate** — lane start → evidence ready at next
  human-checkpoint (`live-verify`) without mid-chain PE skill invocation
  (automated forge hops count as unattended when pin allows)
- **Closeout completion rate** — closeout start → `wave-signoff` stop with
  learning ingest success
- **Wave run duration** — accept/enqueue to stop (Pass-1 and Pass-2 separately)
- **Retry count** — `findings` cycles on workflow-defined loop paths per wave
- **Human intervention count** — should cluster at `human-checkpoint` and
  `authorization: explicit` external-actions (not at automated wave/spec PR)

### Quality metrics (programme hypothesis)

- Verify / live-verify first-pass rate
- Bugbot / review findings density per wave
- Merge without rework rate
- Learning codify rate (`L-*` open → codified into skills/harness)

### Benchmark framing

Compare against teams using Cursor-only or autonomous agents without SDD gates.
Gateflow wins on **governed automation + visibility**, not raw “fastest PR.”

---

## 11. Market landscape and differentiation

Research across 2025–2026 engineering-agent products shows convergent architecture:
**process SSOT in repo artifacts → control plane outside agents → human/external
stops → durable run state → pluggable runners**. Gateflow sits in Layer 3 of
that stack; it is not a coding agent and not a workflow SSOT replacement.

```text
Layer 4 — Programme / factory     prayog-meta · launchpad · catalog
Layer 3 — Delivery control plane  Gateflow (+ peers below)
Layer 2 — Agent runtime           Cursor · OpenCode · OpenHands · Copilot SDK
Layer 1 — Model inference         Claude · GPT · Gemini · …
```

### Parallel products (selected)

| Product | Layer | Workflow / process SSOT | Human gates | Run persistence | Auto-merge default |
|---------|-------|-------------------------|-------------|-----------------|-------------------|
| **Gateflow** (Prayog) | Control plane | **prayog-skills `workflow.yaml`** + handoff envelope | `human-checkpoint` always; `external-action` per pin `authorization` | RunStore + metrics + learning | **No** (non-goal until H4 opt-in) |
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
| Human gates sacred at platform level | `human-gates-never-auto-transition` |
| Durable handoffs, not chat | Handoff envelope + RunStore |
| Deterministic orchestration routing | PolicyEngine; no LLM in resolver |
| Pluggable agent runners | AgentRunner slot (Cursor → …) |
| Workspace / worktree isolation | WorkspacePrep slot |
| Programme metrics vs raw speed | Unattended completion, retry count, wave duration |

### Where Gateflow differentiates

1. **Contract consumer, not process author** — Navigation follows pinned
   `workflow.yaml` (`type`, `dispatch`, `authorization`, `forge`). Spec Kit,
   SpecOrch, and Conductor each **define** their own workflow. Prayog already
   invested in `sdd-delivery/v2`; Gateflow orchestrates that graph without SSOT
   drift — including pin-declared automated forge, not Gateflow-local policy.

2. **Programme factory citizen** — Harness sync (launchpad), service catalog,
   pinned skills release, meta governance (PRDs where Gate 1 applies; ADRs +
   repo specs for pin-consume catch-ups). Peers are repo-local or single-product.

3. **Governed automation hypothesis** — Optimise for unattended Pass-1 / Pass-2
   completion **within contract stops** and programme learning (metrics +
   Learning-Extract), not ticket-to-merged-PR autonomy (Optio) or multi-day
   autonomous missions (Factory/Devin).

4. **Explicit non-goals as product choice** — No auto-merge, no worker auto board
   column moves, no inventing PE gate labels, no env override of pin
   `authorization`. Aligns with “Copilot not Autopilot”; diverges from
   full-autonomy orchestrators by design.

5. **Slots without preset stack** — ToolProvider, ModelGateway, Notifier remain
   extension points (ToolProvider still `none`; multi-runner stubs registered).
   Avoids mandatory codegraph or LangGraph-as-core (contrast Open SWE).

### Landscape-informed horizon notes

| Signal from market | Gateflow response |
|--------------------|-------------------|
| Optio / OpenSymphony reconciliation loops | Idempotent webhooks + fail-closed authorize/apply |
| CI / review feedback resume | H2: workflow-valid resume on external signals (not ad hoc chat) |
| Ops UI expectation (Optio, OpenSymphony TUI, Factory Mission Control) | H1/H1.5 JSON APIs; H2 gateflow-ops UI |
| Multi-runner day one (Optio) | H1 Cursor live + stub registry; H2 live OpenCode/Claude |
| Auto-merge pressure (Optio, Cursor Approval Agents) | H4 opt-in only; never default |
| Draft-PR-as-progress UX | H1.5: PR after coding via automated `wave-pr-action`, not empty PR-at-start |

**Verdict:** The market validates Gateflow’s layer and patterns. Differentiation
is **honoring existing Prayog workflow SSOT** plus **programme-level governed
automation** — not competing with Spec Kit or agent IDEs on process definition.

---

## 12. Initiative sequence

| Order | Artifact | Purpose | Status (2026-08-01) |
|-------|----------|---------|---------------------|
| 1 | This vision doc | Programme north star (`planning/`) | **Refreshed** (H1.5 + pin auth) |
| 2 | [INIT-GATEFLOW-001](../prd/INIT-GATEFLOW-001.md) + skills pin | Control plane MVP + `dispatch` | **Delivered** |
| 3 | INIT-GATEFLOW-002 / 003 (+ meta PRDs as filed) | API starts, board primitives, live Cursor | **Delivered** (H1) |
| 4 | INIT-PRAYOG-SKILLS-003-PROMPTS + INIT-GATEFLOW-005 | Packaged prompts + bound-input substrate | **Delivered** (W0/W1); W2 multi-skill broaden later |
| 5 | INIT-GATEFLOW-006 / 008 | Forge publish/mutate + `authorization` dual mode | **In flight / code complete** (stabilize with 007) |
| 6 | INIT-GATEFLOW-007 | Closeout start + learning DB ingest | **In flight** — part of H1.5 feature slice |
| 7 | Retrospective meta PRDs / Gate 1 hygiene | Fold 006–008 product decisions into prayog-meta | **Open** (PE schedule) |
| 8 | Horizon 2+ INITs | gateflow-ops UI, live multi-runner, ToolProvider, gateway | **Not started** |

Repo product specs (SSOT for gateflow delivery waves):
`drivestream-lab/gateflow` → `docs/specification/product/` +
`docs/specification/as-built/implementation-status.md`.

Programme-level dogfood / “Gateflow runs Gateflow” is **out of this vision
sequence** — plan separately once H1.5 features are declared stable.

---

## 13. Open decisions

| # | Decision | Lean |
|---|----------|------|
| 1 | Metrics + learning SSOT | PostgreSQL in Gateflow RunStore (events + `L-*` ingest); export / UI later (H2–H3) |
| 2 | LiteLLM placement | Separate gateway service when Horizon 3 starts |
| 3 | Programme dogfood timing | **Deferred** — schedule only after H1.5 features are stable; not a horizon exit gate |
| 4 | prayog-skills pin | Remount must keep `.harness-pin.yaml` ref ≡ consumed submodule SHA/tag; track beyond `v0.5.0-rc.2` as pin evolves |
| 5 | ForgeClient auth in dev | Explicit `GITHUB_AUTH_MODE` (`pat` \| `app`); App preferred for deploy |
| 6 | `dispatch: observed` enum | Still deferred — not a Gateflow delivery blocker |
| 7 | Joint Gate 1 / pin timing | **Resolved** for `dispatch` era; **re-open** for retrospective PRDs on 006–008 |
| 8 | Label wave-start | **Parked** — API lane starts are primary; re-enable only via programme config |
| 9 | Authorize→resume after explicit forge | **Out of H1.5** — Pass-2 is new closeout Enter-at; revisit in H2+ if needed |
| 10 | Spec-lane `dispatch: orchestrated` | **Pin dependency** (prayog-skills); Gateflow fail-closed until promoted |

---

## References

### Programme

- Workflow SSOT: prayog-skills [`workflow.yaml`](../prayog-skills/workflow.yaml)
- Delivery contract: prayog-skills [`delivery-contract.yaml`](../prayog-skills/delivery-contract.yaml) (`sdd-delivery/v2`)
- Gateflow consumer brief: prayog-skills `docs/for-gateflow.md`
- Forge side effects: prayog-skills `references/forge-side-effects.md`
- Handoff navigation: prayog-skills `references/handoff-envelope.md`
- Workflow dispatch policy: [INIT-PRAYOG-SKILLS-002](../prd/INIT-PRAYOG-SKILLS-002.md)
- Gateflow product specs / as-built: `drivestream-lab/gateflow` `docs/specification/`
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
