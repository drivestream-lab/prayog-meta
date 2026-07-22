# INIT-GATEFLOW-001 — Gateflow delivery orchestrator (outline)

**Status:** outline · **Author:** programme PM · **Date:** 2026-07-22  
**Vision:** [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md)  
**Component:** GATEFLOW · **Type:** platform / delivery control plane

> **Outline only.** Sections marked `[TBD]` expand in Draft PR after PE/product
> review. Engineering detail routes to impact map and gateflow spec PR.
> **Market landscape and full differentiation:** [vision §12](../planning/gateflow-programme-vision.md#12-market-landscape-and-differentiation).  
> **Workflow `dispatch` policy (rc-2):** [INIT-PRAYOG-SKILLS-002-outline](./INIT-PRAYOG-SKILLS-002-outline.md).

---

## Document control

| Field | Value |
|-------|-------|
| Initiative ID | INIT-GATEFLOW-001 |
| Artifact | `prd/INIT-GATEFLOW-001-outline.md` (outline); Draft PRD [INIT-GATEFLOW-001](./INIT-GATEFLOW-001.md) |
| Programme | prayog |
| Primary repos | drivestream-lab/gateflow, drivestream-lab/gateflow-ops |
| Supporting repos | prayog-meta, prayog-skills, launchpad |
| Paired initiative | [INIT-PRAYOG-SKILLS-002-outline](./INIT-PRAYOG-SKILLS-002-outline.md) — `dispatch` on rc-2 |
| **Gate 1 coupling** | **Joint Gate 1** with INIT-PRAYOG-SKILLS-002 — required before rc-2 pin, W0 `dispatch` logic, or dogfood Phase B |
| Target users | PE (wave execution), tech lead (gates), programme sponsor (metrics) |

---

## 1. Problem statement

Today, after upstream workflow steps complete (through `board-seed` per pinned
`workflow.yaml`), **PE manually triggers** each wave `skill` node. Quality is
strong when upstream spec debate was thorough, but:

- Execution is **slow and inconsistent** across initiatives
- Run state lives in **chat**, not durable artifacts
- There is **no programme metric** for automation vs manual waves
- Parallel agent tools (Cursor, OpenCode, etc.) are used **ad hoc**, not under
  delivery policy

We need a **control plane** that resolves the pinned workflow and dispatches
eligible `skill` nodes reliably — **stopping** wherever `workflow.yaml` and
`sdd-delivery/v2` assign humans or external actions.

---

## 2. Proposed solution (summary)

**Gateflow** is a delivery orchestrator that:

1. Listens to **GitHub** (webhooks, labels, PR events)
2. Reads **handoff envelopes** and pinned **`workflow.yaml`** + **`delivery-contract.yaml`** from prayog-skills
3. **Resolves** the next workflow node from handoff stage + outcome (same rules as handoff spec)
4. **Dispatches** coding agents via **AgentRunner** when resolved node is `type: skill`, **`dispatch: orchestrated`**, and PE trigger authorizes (see [INIT-PRAYOG-SKILLS-002](./INIT-PRAYOG-SKILLS-002-outline.md))
5. **Injects tool context** via **ToolProvider** slots when enabled (none in H1; Graphify/MCP H2)
6. **Stops** on `human-checkpoint`, `external-action`, `decision`, and `terminal` nodes per contract
7. **Notifies** via **Notifier** slot (H1: GitHub comments through **ForgeClient**; Slack/Teams reserved for H2+)
8. Records **run history and metrics** in PostgreSQL for programme visibility

Gateflow **does not write code** and **does not define the delivery process**.
It enables agents with harness, skills, and (optionally, later) pluggable tools.

---

## 3. Positioning and differentiation (outline)

Industry research (Jul 2026) validates the **delivery control plane** pattern:
orchestration outside agents, human gates sacred, durable run state, pluggable
runners. Full landscape analysis lives in [vision §12](../planning/gateflow-programme-vision.md#12-market-landscape-and-differentiation). This section states **why Gateflow for Prayog** at outline stage — Draft PR expands acceptance criteria only, not competitive essays.

### Closest parallels

| Peer | Similar to Gateflow | Differs from Gateflow |
|------|---------------------|------------------------|
| **OpenSymphony** | Orchestrator outside harness; repo workflow SSOT; workspace isolation | Prose `WORKFLOW.md`; Linear trigger; no programme factory |
| **Spec Kit workflows** | YAML orchestration; gate pause/resume; run persistence | **Owns** Spec Kit workflow — does not consume prayog-skills graph |
| **Microsoft Conductor** | Deterministic YAML routing; `human_gate`; zero-token orchestration | Generic multi-agent CLI — not SDD programme or handoff envelope |
| **SpecOrch** | Spec-first control plane; gate-first; evidence loop | **Owns** seven-plane process — SSOT drift vs existing Prayog contract |
| **Optio** | Run store; worktree isolation; multi-runner; CI feedback resume | Ticket-to-**auto-merge**; no `sdd-delivery/v2` node types |
| **Open SWE / Factory** | Long-running runs; HITL at plan/validator gates | LangGraph / mission orchestrator **is** the process owner |
| **Cursor Cloud Agents** | First H1 `AgentRunner`; Automations API | Event→agent triggers — no external workflow resolver |

### Gateflow moat (outline — detail in vision §12)

1. **Contract consumer** — Resolves pinned `workflow.yaml` + handoff envelope;
   does not maintain a parallel gate list or fork Spec Kit / Conductor graphs.
2. **Programme factory** — launchpad harness sync, service catalog, pinned
   prayog-skills release; peers are repo-local tools.
3. **Governed automation** — Measure unattended completion **within contract
   stops**; not optimising for autonomous merge (Optio) or multi-day missions
   (Factory/Devin) in H1.
4. **Explicit pilot non-goals** — No auto-merge, no auto board, no dispatch unless
   resolved skill has `dispatch: orchestrated`; aligns with GitHub Copilot guardrails.
5. **Pluggable slots, empty defaults** — ToolProvider / ModelGateway = `none`
   in MVP; no LangGraph-as-core.

### Outline vs Draft PR boundary

| In this outline | In Draft PR `[TBD]` | In gateflow spec PR |
|-----------------|---------------------|---------------------|
| Positioning summary (§3) | FR acceptance criteria per capability | Reconciliation loop, adapter interfaces |
| Vision §12 reference | Numeric success targets (§10) | OSS pattern implementation detail |
| Moat bullets | User stories with testable outcomes | RunStore schema, webhook idempotency |

---

## 4. Users and jobs-to-be-done

| User | Job to be done |
|------|----------------|
| **PE** | “When I authorize a wave run, I want eligible workflow skills to dispatch until the next contract stop point — without babysitting each skill.” |
| **Tech lead** | “I want the same workflow gates and constitution enforcement, with an audit trail of what ran.” |
| **Programme sponsor** | “I want to see if automated waves beat manual cycle time and rework.” |

---

## 5. Scope — in (MVP / Horizon 1)

### 5.1 Capabilities

| ID | Capability | Notes |
|----|------------|-------|
| FR-1 | GitHub App receives webhooks (PR, issue, label) | Primary event source |
| FR-2 | Explicit wave trigger via label (e.g. `gateflow:run-w0`) | **Programme config**, not a code constant; no board-only inference |
| FR-3 | Run store persists wave run and stage history | **PostgreSQL only** (local dev: Postgres container or shared dev instance) |
| FR-4 | Handoff envelope read from repo artifacts | Not chat memory |
| FR-5 | Workflow resolver per `sdd-delivery/v2` — dispatch only when `type: skill` **and** `dispatch: orchestrated`; follow outcome graph | See §5.2; requires rc-2 pin |
| FR-6 | Dispatch Cursor local agent with skill prompt | First `AgentRunner`; see FR-13 for model |
| FR-7 | Retry budget on workflow `findings` loops (default: 3) | **Programme config**; on exhaustion: **stop + GitHub comment only** |
| FR-8 | Stop on all `human-checkpoint` and `external-action` nodes; never auto-transition per contract | `human-gates-never-auto-transition` |
| FR-9 | Run progress via **ForgeClient** (GitHub API comments on PR/issue) | Phase 0 UI; **not `gh` CLI** — see §5.6 |
| FR-10 | Metrics v0: stage duration, retries, unattended completion; **skill stage profiling** (see §5.4) | PostgreSQL event log |
| FR-11 | Stage-scoped **tool provider slots** (all `none` in MVP) | Extensibility; Graphify/MCP H2 — §5.6 |
| FR-12 | gateflow read-only status JSON API | W1 on gateflow; gateflow-ops BFF deferred W2+ |
| FR-13 | **Runner + model profile** per dispatch — programme config, resolved model recorded on every orchestrated stage | Cursor SDK H1; OpenCode H2; see §5.4 |
| FR-14 | **ForgeClient** — outbound GitHub via App installation token (PAT pilot ok); PR create/update, labels, statuses as needed | Cloud/Docker; no `gh` — §5.6 |

### 5.2 Workflow navigation (contract SSOT)

Gateflow implements prayog-skills handoff navigation rules:

1. Read latest handoff + pinned `workflow.yaml` + `delivery-contract.yaml`
2. Verify handoff contract matches installed contract
3. Resolve transition from `handoff.stage` + `handoff.outcome`
4. **Stop** on `human-checkpoint`, `external-action`, `decision`, or `terminal` (node `type`)
5. **Stop** (or observe-only for metrics) when resolved `skill` has `dispatch: manual` or `observed`
6. **Dispatch** only when resolved `skill` has `dispatch: orchestrated` **and** PE label trigger authorizes
7. Never bypass `handoff.human_checkpoint: true` or contract principles

**Orchestration policy SSOT** — which skills are automatable is defined by
**`dispatch` on each skill node** in prayog-skills `workflow.yaml`
([INIT-PRAYOG-SKILLS-002](./INIT-PRAYOG-SKILLS-002-outline.md), rc-2). Gateflow
**must not** hardcode node id lists or parallel programme-config skill allowlists.
On pins without `dispatch` (v0.4.3), **schema default only:** missing field →
`treat as manual` — never a config list of orchestrated node ids.

| Check | Source | Gateflow action |
|-------|--------|-----------------|
| Stop at gate / merge | `type: human-checkpoint` / `external-action` | Stop |
| Skill runnable by orchestrator? | `skill` + `dispatch: orchestrated` | May dispatch if triggered |
| Skill manual | `skill` + `dispatch: manual` | Stop; observe metrics from handoff |
| Skill metrics-only | `skill` + `dispatch: observed` `[TBD]` | Stop; observe metrics |

Illustrative rc-2 annotation (defined in prayog-skills, not Gateflow):

| `dispatch` | Example nodes |
|------------|---------------|
| `manual` | `validate-requirements` … `board-seed`, `spec-draft`, … |
| `orchestrated` | `pre-implement`, `loop-spec`, `verify`, `ground-spec` |

Wave subgraph (navigation from workflow outcomes — unchanged):

```text
board-seed → pre-implement → loop-spec ⇄ verify → ground-spec → wave-human-decision (STOP: type)
```

### 5.4 Skill and gate profiling (metrics intent)

Gateflow metrics serve two purposes: **operate wave runs** and **profile the
delivery contract** so prayog-skills can be improved over time. Timing is
recorded per **`workflow.yaml` node id** (skills and gate intervals), not ad
hoc chat sessions.

| Telemetry class | What is measured | H1 source |
|-----------------|------------------|-----------|
| **Orchestrated skill** | `started_at` → `ended_at` per dispatched `skill` node | Gateflow run engine (AgentRunner lifecycle) |
| **Observed skill** | Start/end when PE or agent completes a skill outside dispatch | Handoff envelope + artifact digest timestamps in repo |
| **Gate interval** | Elapsed between workflow stop points | GitHub signals + handoffs; **approval label names from pinned `delivery-contract.yaml`**, not Gateflow constants — e.g. spec PR ready → gate-2 unlock label |
| **Wave run** | Label trigger → next `human-checkpoint` | RunStore run record |
| **Findings loop** | Retry count and duration per `findings` transition | PolicyEngine + stage events |

Example intervals the programme cares about (illustrative — workflow nodes and
**contract-defined labels** from the pin, not Gateflow-invented gates):

```text
spec-draft … spec-implementation-plan  (skills with dispatch: manual or observed)
spec PR open ──────────────► [gate-2 unlock label from delivery-contract.yaml]
spec-merge ──► board-seed ──► … next skills per dispatch on pinned workflow.yaml
```

Each event row in PostgreSQL should be queryable by:
`programme_slug`, `org`, `repo`, `initiative_id`, `workflow_node`, `wave`,
`outcome`, `dispatch_mode` (`orchestrated` | `manual` | `observed`), optional
`run_id`.

**Runner and model dimensions (orchestrated stages)** — every Gateflow dispatch
records what actually ran, for **model efficacy** analysis per skill:

| Field | Example | Purpose |
|-------|---------|---------|
| `runner` | `cursor`, `opencode` (H2) | Which AgentRunner adapter |
| `model_profile` | programme config key, e.g. `default`, `loop-spec-heavy` | What Gateflow resolved before dispatch |
| `model_id` | `cursor/auto`, `cursor/glm-4.7`, `claude-sonnet-4` | Actual model the runner reported |
| `model_provider` | `cursor`, `anthropic`, `openai` | Normalised provider for cross-runner queries |

Observed/manual stages may omit model fields unless the handoff or runner API
surfaces them; orchestrated stages **must** populate all four where available.

**Configuration (programme / meta config, not workflow.yaml):** Gateflow resolves
`runner + model_profile` per `workflow_node` at dispatch time via **pluggable
programme config** (harness / meta YAML) — never hardcoded node→model maps in
source. H1: programme default + optional per-node overrides; Cursor SDK passes
selected model to the agent. H2: same pattern for OpenCode and other runners via
ModelGateway slot.

**Product intent:** aggregate outcome and duration **by workflow_node × model_id**
(e.g. `loop-spec` first-pass rate: auto vs glm) to tune programme model profiles
and inform prayog-skills — without changing workflow SSOT.

Aggregated skill profiles (p50/p95 duration, first-pass rate, findings density
per node) feed programme learning and **future skill enhancements** in
prayog-skills — Gateflow measures; skills team acts.

H1 delivers orchestrated wave timing plus passive observation where handoffs and
GitHub labels already exist; full upstream coverage expands as webhook and
handoff parsing mature (H2).

### 5.6 Pluggable runtime — runners, tools, forge (cloud)

Gateflow scales on **four independent slots**. Orchestration core stays fixed;
adapters swap per programme and horizon. **Pluggability rule:** no slot
implementation may embed workflow node ids, gate label names, or dispatch
allowlists — those come from the pinned contract + programme config.

```text
Control plane → AgentRunner (who codes)
              → ToolProvider (what tools/context)
              → ForgeClient (GitHub platform API)
              → Notifier (human-visible alerts — fan-out)
```

**AgentRunner (H1 Cursor → H2 OpenCode, Claude Code)**

Same dispatch contract for every adapter: prepared workspace, skill prompt,
`model_profile`, optional `tool_context` → `RunResult` with `runner`,
`model_id`, outcome. H1 ships Cursor SDK adapter; spec PR documents interface
for additional runners without control-plane forks.

**ToolProvider (H1 none → H2 Graphify / MCP / scanners)**

`StageToolResolver` resolves tool slot for the current `workflow_node` from
**programme config or future workflow metadata** — not a hardcoded node→slot
table in Gateflow source. Slot names (e.g. `prep_tools`, `implement_tools`) are
adapter contract identifiers; H1 all providers = `none`. Enabled providers
materialise context in workspace before dispatch. Skills remain procedure SSOT in
prayog-skills; skill **enhancements** to use tools are a separate initiative —
Gateflow only injects the hook.

**ForgeClient (H1 required — cloud-safe GitHub)**

Production runs in **Docker/cloud** without `gh` CLI or PE laptop sessions.

| Direction | Mechanism |
|-----------|-----------|
| Inbound | GitHub App webhooks (FR-1) |
| Outbound | GitHub REST/GraphQL via **App installation token** (preferred) or scoped PAT (pilot) |

ForgeClient handles **GitHub platform operations only** (comments, PRs, labels,
statuses). It is not a general notification bus.

| ForgeClient may (operations) | ForgeClient must not (human gates) |
|------------------------------|-------------------------------------|
| PR/issue comments | Auto-merge |
| Open/update wave PR (agent branch) | Set gate approval labels (contract `github.labels`; observe only) |
| Run status labels (programme-defined) | Auto-move programme/project board |
| Optional commit statuses | Execute `external-action` nodes |

Agent **git push** uses deploy keys / fine-grained tokens on the worker —
separate from ForgeClient platform API auth.

**Notifier (H1 GitHub only — slot reserved for chat)**

Structured run events fan out to one or more Notifier backends configured per
programme. **H1:** single impl — GitHub PR/issue comments via ForgeClient (FR-9).
**Not in this outline:** Slack, Microsoft Teams, or other chat integrations —
architecture leaves the **Notifier slot** and event schema in place for H2+
without MVP scope creep.

| Notifier backend | Outline scope |
|------------------|---------------|
| GitHub comment (→ ForgeClient) | **H1 — FR-9** |
| Slack | Slot only; future INIT |
| Microsoft Teams | Slot only; future INIT |
| Email / PagerDuty / … | Slot only; future |

Notifier consumes the same run/stage events as MetricsEmitter; adding Slack does
not change AgentRunner, ToolProvider, or workflow resolution.

### 5.7 Repositories

| Repo | W1 deliverable |
|------|----------------|
| **gateflow** | Full control plane — API, webhooks, run engine, Cursor AgentRunner, ForgeClient, Notifier (GitHub H1), ToolProvider slots (none), status JSON API |
| **gateflow-ops** | **Out of W1 scope** — BFF/UI in W2+ |

---

## 6. Scope — out (explicit non-goals)

| Non-goal | Rationale |
|----------|-----------|
| Auto-merge PRs | `external-action` nodes require explicit authorization per contract |
| Auto-update programme board | Outside workflow dispatch; PE-owned unless future opt-in policy |
| Dispatch when `dispatch != orchestrated` or non-`skill` `type` | Per pinned `workflow.yaml` — not informal upstream/wave boundaries |
| Build a coding agent | Use Cursor / others via adapters |
| Plug Graphify or codegraph in MVP | Slots only; provider = `none` |
| Redefine or fork delivery process | Navigation SSOT is prayog-skills `workflow.yaml`; automation SSOT is `dispatch` field |
| Hardcode orchestrated skill node lists | Use `dispatch: orchestrated` from workflow pin (INIT-PRAYOG-SKILLS-002) |
| Hardcode node→tool-slot or node→model maps in source | Programme config or workflow metadata; adapter interfaces only |
| Hardcode GitHub gate label strings | Load from pinned `delivery-contract.yaml` (`github.labels`) |
| Change prayog-skills procedures in this INIT | Paired INIT-PRAYOG-SKILLS-002 on rc-2 |
| OpenCode / Claude Code / LiteLLM in MVP | Horizon 2–3; **AgentRunner interface** in H1 spec |
| **`gh` CLI for GitHub operations** | ForgeClient + GitHub API only in cloud/Docker |
| **Slack / Teams / chat notifications** | Notifier slot reserved; H1 is GitHub comments only (FR-9) |
| Auto-set gate approval labels | Observe contract labels for metrics; humans set gates |
| Full ops dashboard | Horizon 2 |

---

## 7. Product principles

1. **Workflow SSOT** — `workflow.yaml` + `delivery-contract.yaml` govern navigation (`type`) and automation eligibility (`dispatch` on skills); Gateflow does not maintain parallel lists
2. **Artifacts not chat** — handoff envelopes and reports are durable state
3. **Contract invariants** — never auto-transition `human-checkpoint`; never execute `external-action` without authorization
4. **Explicit triggers** — label authorizes a run; resolver checks `dispatch: orchestrated` on next skill
5. **Pluggable, not preset** — **AgentRunner**, **ToolProvider**, **ForgeClient**, **Notifier** slots; triggers, retry budgets, model profiles, and tool slots in **programme config** — never hardcoded node lists or label names in source
6. **Cloud-native forge** — outbound GitHub via API (App token); never depend on `gh` CLI in production
7. **Measure to improve** — every run emits metrics for programme learning; **profile each workflow skill** (duration, outcome, retries, **model_id**) to inform prayog-skills and model tuning

---

## 8. Roadmap horizons (product)

| Horizon | Theme | Init linkage |
|---------|-------|--------------|
| **H1 — MVP** | Cursor AgentRunner, ForgeClient, run engine, metrics v0, tool slots wired (`none`) | **This INIT** |
| **H2 — Scale** | Multi-runner, model profiles, ops UI, optional tool providers | INIT-GATEFLOW-002 `[TBD]` |
| **H3 — Gateway** | LiteLLM proxy, shared tool services, cost/quality dashboards | INIT-GATEFLOW-003 `[TBD]` |
| **H4 — Policy** | Opt-in automation policies (merge, fleet queue) | Future; not committed |

Detail: [planning/gateflow-programme-vision.md §9](../planning/gateflow-programme-vision.md).

---

## 9. Dogfood plan

| Phase | How | When |
|-------|-----|------|
| **A** | Deliver Gateflow MVP using **manual SDD** (this INIT → spec → waves by PE) | Now |
| **B** | Run **next** gateflow waves via Gateflow label trigger | After H1 engine stable **and Joint Gate 1 + rc-2 pin** (Phase A may use manual SDD without `dispatch`) |
| **C** | Compare metrics: manual vs automated waves | After Phase B |

---

## 10. Success criteria (MVP)

| Metric | Target `[TBD — calibrate in pilot]` |
|--------|-------------------------------------|
| Unattended completion | One W0 run: label → next `human-checkpoint` after wave `skill` nodes without mid-chain PE skill invocation |
| Human interventions | Only at workflow stop types (`human-checkpoint`, `external-action`, `decision`) per contract |
| Retry budget | ≤ 3 workflow `findings` cycles on loop paths unless PE overrides |
| Run visibility | PE can reconstruct run from GitHub comments + status API |
| Skill stage records | Every orchestrated wave `skill` node emits start/end + outcome + **runner + model_id** in PostgreSQL |
| Gate interval (pilot) | At least one observed interval logged using **contract-sourced** gate labels per dogfood initiative `[TBD]` |
| Harness compliance | `launchpad status --repo gateflow` green post-wave |

---

## 11. Dependencies and assumptions

### 11.1 Joint Gate 1 (blocking — both INITs together)

**INIT-GATEFLOW-001 and INIT-PRAYOG-SKILLS-002 must pass a single Joint Gate 1**
before integration or dogfood that depends on `dispatch`. Gateflow must not ship
production PolicyEngine logic that reads `dispatch` until rc-2 semantics are
Gate 1–approved alongside this INIT.

| Blocked until Joint Gate 1 | Rationale |
|----------------------------|-----------|
| Gateflow W0 PolicyEngine reading `dispatch` | Must match prayog-skills rc-2 SSOT |
| Meta harness pin to rc-2 | Untestable without aligned consumer |
| rc-2 merge / tag on prayog-skills | Paired INIT owns field definition |
| Dogfood Phase B (label-triggered runs) | End-to-end contract review in one session |

**May proceed in parallel before Joint Gate 1:** outline → Draft PRD,
`/validate-requirements`, impact-map scoping, Phase A manual SDD for Gateflow
build (no `dispatch` consumption).

**Joint Gate 1 confirms:** FR-5 consumer algorithm, `dispatch` enum and wave-lane
values, v0.4.3 schema-default bounds (missing → `manual` only), rc-2 pin timing,
pilot trigger label (programme config).

### 11.2 Technical dependencies

| Dependency | Assumption |
|------------|------------|
| prayog-skills rc-2 | [INIT-PRAYOG-SKILLS-002](./INIT-PRAYOG-SKILLS-002-outline.md) adds `dispatch`; pin **after Joint Gate 1** |
| prayog-skills @ v0.4.3 | Current pin; `dispatch` absent — consumers treat skills as `manual` until rc-2 |
| launchpad | Harness sync before agent dispatch |
| gateflow / gateflow-ops repos | Live on develop, bootstrap complete |
| GitHub App | Installation on drivestream-lab programme repos; **installation token for ForgeClient API** |
| Cursor SDK | Container worker or cloud VM for pilot (not PE laptop-only) |
| PostgreSQL | RunStore SSOT for all environments; no SQLite fallback |
| Repo git access | Deploy keys or fine-grained token on worker for agent push |
| Container runtime | Docker (orchestrator TBD in spec); **no `gh` CLI** in image |

---

## 12. Risks `[TBD — expand in Draft]`

| Risk | Mitigation sketch |
|------|-------------------|
| Agent run cost / duration | Retry budget; metrics; model profiles later |
| Webhook reliability | Idempotent handlers; run store dedup |
| Handoff parse failures | Block run; notify PE; no silent continue |
| rc-2 / dispatch delay | Schema default on missing `dispatch` → `manual`; no node allowlist fallback in Gateflow |
| Over-automation pressure | Non-goals in PRD; `type` + `dispatch` + contract principles |

---

## 13. Open questions (Joint Gate 1)

Items **4, 8** and all `dispatch`-related choices are resolved at **Joint Gate 1
with INIT-PRAYOG-SKILLS-002** — do not implement W0 `dispatch` consumption or
rc-2 pin until that session completes.

1. Confirm pilot label name: `gateflow:run-w0` vs programme-specific prefix?
2. GitHub comments only vs commit status checks for Phase 0 evidence?
3. When retry budget exhausts: **stop + GitHub comment only** (no issue, no auto-route) — **decided**
4. W1 impact map / dogfood: **gateflow repo only**; gateflow-ops W2+ — **decided**
5. Metrics retention period and export format for sponsor review?
6. Per-skill model profile defaults for pilot (e.g. all `cursor/auto` vs `loop-spec` → glm)?
7. ForgeClient auth: GitHub App installation token only vs PAT allowed in dev?
8. rc-2 pin timing vs Gateflow W0 merge — **Joint Gate 1 agenda** with [INIT-PRAYOG-SKILLS-002](./INIT-PRAYOG-SKILLS-002-outline.md)

---

## 14. Next steps (process)

| Step | Owner | Artifact |
|------|-------|----------|
| 1 | PM / sponsor | Review outlines → expand to Draft PRDs (parallel with skills INIT) |
| 2 | PE | `/validate-requirements` on each Draft (parallel ok) |
| 3 | PE / sponsor | **Joint Gate 1** — both Draft PRDs + `dispatch` contract **(blocking)** |
| 4 | PE | `/prd-impact-map` → **gateflow only** (W1); gateflow-ops W2+ **(after step 3)** |
| 5 | PE | Gateflow spec PR + rc-2 implementation (paired) |
| 6 | PE | Dogfood Phase B after W0 + rc-2 pin |

---

## Appendix A — Outline → full PRD checklist

Sections to flesh out before Draft PR:

- [ ] §5.6 ForgeClient + AgentRunner interface acceptance criteria
- [ ] §5.4 skill profiling acceptance criteria (orchestrated vs observed stages)
- [ ] §5 FR acceptance criteria (testable, per capability)
- [ ] §10 numeric targets after baseline manual wave timing
- [ ] §12 risk register with owners
- [ ] User stories (PE / tech lead / sponsor) — positioning fixed in §3; do not re-litigate landscape in PRD body
- [ ] §5.8 waves at PRD level (high-level W0/W1 themes only — detail in spec)

---

## Appendix B — Relationship to vision doc

| Vision (planning/) | This INIT |
|--------------------|-----------|
| Programme north star, horizons, architecture slots | Gate 1 scope boundary for H1 |
| §12 market landscape and differentiation | §3 outline summary; Draft PR references vision, does not duplicate |
| Contract-driven navigation + `dispatch` | FR-5, FR-8, §5.2; [INIT-PRAYOG-SKILLS-002](./INIT-PRAYOG-SKILLS-002-outline.md) |
| **Joint Gate 1 with skills INIT** | §11.1 — blocks rc-2 pin and dogfood Phase B |
| Pluggable runtime (AgentRunner, ToolProvider, ForgeClient, Notifier) | §5.6, FR-9, FR-11, FR-13, FR-14 |
| Tooling parked | FR-11 slots; Graphify/MCP H2 |
| Chat notifications parked | Notifier slot; Slack/Teams H2+ — not this outline |
| Cloud GitHub (API not `gh`) | FR-9, FR-14, §5.6 |
| OSS influences + landscape-informed horizon notes | Engineering appendix in spec PR, not PRD body |
