# INIT-GATEFLOW-005-BOUNDINPUT — Bound-input skill invocation

**Status:** draft PRD · **Author:** programme PM · **Date:** 2026-07-27  
**Outline:** [INIT-GATEFLOW-005-BOUNDINPUT-outline](./INIT-GATEFLOW-005-BOUNDINPUT-outline.md)  
**Vision:** [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md)  
**Upstream (delivered):** [INIT-PRAYOG-SKILLS-003-PROMPTS](./INIT-PRAYOG-SKILLS-003-PROMPTS.md) — skill prompt packages  
**Related:** [INIT-PRAYOG-SKILLS-002](./INIT-PRAYOG-SKILLS-002.md) (`dispatch` — eligibility, orthogonal) · [INIT-GATEFLOW-001](./INIT-GATEFLOW-001.md) (control plane foundation)  
**Component:** GATEFLOW · **Type:** platform / invocation substrate

> **Draft PRD** for Gate 1 review. Engineering implementation routes to
> impact map and **gateflow** spec PR (`drivestream-lab/gateflow` only).
> **Invocation SSOT** = prayog-skills prompt package on the active pin.
> **`handoff_path`** = Gateflow-defined and stored on the run — **not** a repo
> layout concern. **Do not treat `dispatch: orchestrated` alone as invocation
> done.**

---

## Document control

| Field | Value |
|-------|-------|
| Initiative ID | INIT-GATEFLOW-005-BOUNDINPUT |
| Programme | prayog |
| Primary repo | drivestream-lab/gateflow |
| Impact map scope | **gateflow only** `(Source: User-confirmed)` |
| Supporting / monitor | prayog-skills (packages **delivered**); prayog-meta (pin bump monitor); gateflow-ops (**out of delivery**) |
| Upstream | INIT-PRAYOG-SKILLS-003-PROMPTS (**delivered** — meta PR #14) |
| Target users | Programme eng (wave-start), PE (pin consume), Gateflow / platform eng |
| Product ids | Canonical **`REQ-n`**; legacy display alias **`FR-n` ≡ `REQ-n`**; no `CAP-*` in this INIT |
| Gate 1 | Solo Gate 1 on this Draft — upstream prompt packages already delivered |

---

## 1. Executive Summary

### Problem Statement

Pinned prompt packages exist (INIT-PRAYOG-SKILLS-003-PROMPTS), but without a
Gateflow runtime substrate to bind known context, render the pin template, and
dispatch without inventing prose — and without a per-run handoff baton owned by
Gateflow instead of git-tree discovery — automated runs **would** invent
invocation briefs, **would** violate the upstream consumer contract (fail closed,
return `prompt_id` / `prompt_revision`), and **would** remain fragile under
concurrent runs.

### Proposed Solution

**Bound-input skill invocation** in Gateflow: on wave-start, create/continue a
run, **define and store `handoff_path` on the run**, resolve the skill’s prompt
package from the pin, bind known context + stored `handoff_path`, validate and
render simple `{{var}}` substitution, dispatch **only** the rendered message via
Cursor AgentRunner, persist prompt and runner telemetry, and ingest handoff
**only** from the stored `handoff_path`.

```text
wave-start
  → Gateflow defines + stores run.handoff_path
  → authorize automate skill (dispatch + trigger — unchanged)
  → resolve pin prompts/ → validate → render
  → AgentRunner(message = rendered template only)
  → persist prompt_id + prompt_revision (+ runner + model)
  → ingest handoff from run.handoff_path
```

### Success Criteria

| KPI | Target | Measurement |
|-----|--------|-------------|
| **Pin SSOT** | 100% of automated packaged-skill dispatches use rendered pin `template.md` only | Dispatch audit + anti-hardcode tests |
| **No invent-prose** | Zero Gateflow-owned invocation briefs on automated packaged-skill path | Code/review gate + REQ-5 AC |
| **Fail closed** | Missing/invalid package or required bind failure → **no** AgentRunner call | Integration tests |
| **Traceability** | 100% automated packaged-skill stages record `prompt_id` + `prompt_revision` | RunStore completeness |
| **Handoff ownership** | Every automated run has Gateflow-defined `handoff_path` stored on the run; ingest uses **only** that value | RunStore + ingest tests |
| **Prove-it** | ≥1 Cursor automated hop green on pinned package | W0/W1 exit evidence |

---

## 2. User Experience & Functionality

### User Personas

| Persona | Role | Primary need |
|---------|------|--------------|
| **Programme eng** | Starts waves | Wave-start with known context; agent gets pinned brief |
| **PE** | Pin / programme owner | Prompt revision bumps apply without Gateflow code changes |
| **Gateflow engineer** | Control-plane implementer | Clear bind → render → thin dispatch → store/ingest contract |
| **Platform** | Reliability owner | Per-run handoff baton; prompt + runner telemetry |

### User Stories & Acceptance Criteria

#### US-1 — Programme eng starts an automated skill with bound context

**As a** programme eng, **I want** to call wave-start with known context (`ticket`, `initiative`, …) **so that** Gateflow automates a skill using the pinned prompt package, not a thin stub brief.

**Acceptance criteria:**

- [ ] Wave-start accepts known context required for bind (`ticket` required; `initiative` optional) `(Source: User-confirmed)`
- [ ] On success path, AgentRunner receives **only** the rendered pin template message
- [ ] Missing required wave-start context → fail closed; no AgentRunner call
- [ ] Substrate works for **any** packaged skill Gateflow automates under then-current policy `(Source: User-confirmed)`

#### US-2 — PE benefits from pin prompt revisions without Gateflow changes

**As a** PE, **I want** Gateflow to resolve `prompt_id` + `revision` from the active pin **so that** promoting a prompt package improves automated briefs without shipping Gateflow code.

**Acceptance criteria:**

- [ ] Resolve uses pin package layout from INIT-003 (`prompts/template.md` + `schema.yaml`)
- [ ] Stage/outcome records `prompt_id` and `prompt_revision` matching the resolved package
- [ ] No Gateflow source change required to pick up a new evaluated prompt revision on pin/tag promote

#### US-3 — Platform trusts per-run handoff ownership

**As a** platform owner, **I want** Gateflow to define and store `handoff_path` on the run **so that** ingest never depends on repo document trees or mtime globs.

**Acceptance criteria:**

- [ ] At run create/continue, Gateflow defines `handoff_path` and persists it on the run record `(Source: User-confirmed)`
- [ ] Bound map injects `handoff_path` = stored run value into the prompt render
- [ ] Post-agent ingest reads **only** `run.handoff_path`
- [ ] Ambient repo globs / mtime discovery are **not** SSOT for automated ingest under this INIT
- [ ] Two concurrent runs cannot ingest each other’s handoff via shared repo layout

#### US-4 — Gateflow engineer removes invent-prose paths

**As a** Gateflow engineer, **I want** an explicit requirement to remove Gateflow-owned invocation prose **so that** the prayog-skills prompt package is the only automated brief SSOT.

**Acceptance criteria:**

- [ ] Automated packaged-skill path has zero Gateflow-authored invocation templates/strings used as the AgentRunner message
- [ ] Tests (or equivalent AC evidence) fail if a hardcoded brief is used when a pin package exists
- [ ] Missing/invalid package → fail closed (no stub fallback brief)

### Functional Requirements (REQ) — with Acceptance Criteria

Canonical ids are **`REQ-n`** (`FR-n` ≡ `REQ-n`).

| ID | Requirement | Acceptance criteria |
|----|-------------|---------------------|
| **REQ-1** | Resolve prompt package from active pin for automated skill | Load `prompts/` per INIT-003 for the skill; missing/invalid → fail closed before dispatch |
| **REQ-2** | Bind inputs via wave-start + run record | Mapping per [§4.4](#bound-input-mapping); `ticket`/`initiative` from wave-start; `skill_id`/`workspace` known on run; `handoff_path` from stored run field |
| **REQ-3** | Validate bound inputs against `schema.yaml` | Required miss → fail closed; optional omit → empty string at render (INIT-003) |
| **REQ-4** | Render simple `{{var}}` only | No filters/conditionals; every template var declared in schema |
| **REQ-5** | AgentRunner message = rendered template only | **Remove** Gateflow-owned invocation prose; message body equals render output `(Source: User-confirmed)` |
| **REQ-6** | Persist `prompt_id` + `prompt_revision` on outcome/stage | 100% of automated packaged-skill stages; values match resolved package |
| **REQ-7** | Persist runner + model on automated stage | Always record `runner` + `model_id`; `model_profile` / `model_provider` optional — omit or null if the adapter does not return them |
| **REQ-8a** | Define + store `handoff_path` on the run | Gateflow defines and persists `handoff_path` on the run record; independent of repos `(Source: User-confirmed)` — **W0** |
| **REQ-8b** | Ingest handoff only from stored `run.handoff_path` | Post-agent ingest reads **only** the stored path; no ambient glob/mtime SSOT `(Source: User-confirmed)` — **W1** |
| **REQ-9** | Fail closed before dispatch | Package/schema/bind failures never call AgentRunner |
| **REQ-10** | Prove-it on Cursor | ≥1 automated hop using pinned package for **any skill with `dispatch: orchestrated` on the active pin** (concrete skill id chosen at W0); substrate valid for any packaged skill |

### Error Handling

| Condition | Required behavior |
|-----------|-------------------|
| Prompt package missing or schema invalid | **FAIL CLOSED** — no AgentRunner; record failure reason on run |
| Required bind var missing (e.g. `ticket`) | **FAIL CLOSED** — no AgentRunner |
| Optional bind var omitted | Empty string substitution |
| `run.handoff_path` unset when required for automate | **FAIL CLOSED** — do not dispatch |
| Handoff missing/unreadable at `run.handoff_path` after agent | **FAIL CLOSED** on ingest — do not advance on ambient discovery |
| AgentRunner failure/timeout | Stop stage; record failure; do not invent alternate brief |
| Wave-start auth reject | **FAIL CLOSED** — record reason; no AgentRunner |
| Wave-start API / transport unavailable or timeout | **FAIL CLOSED** — record reason; no AgentRunner |
| Concurrent automate request while active run exists | **Reject** (HTTP 409 Conflict / fail closed); record reason — inherit INIT-001 / WaveStartService concurrency |
| Human manual skill execution | Out of scope — may freeform (INIT-003) |

### Non-Goals

| Non-goal | Rationale |
|----------|-----------|
| Authoring / revising prompt templates or schemas | INIT-PRAYOG-SKILLS-003-PROMPTS |
| Eval-before-promote of prompts | Skills / pin process |
| Changing `dispatch` enum or eligibility rules | INIT-PRAYOG-SKILLS-002 |
| Repo-relative handoff discovery as SSOT | Gateflow-owned `handoff_path` `(Source: User-confirmed)` |
| Worktree-per-run as exit criterion | Stored per-run `handoff_path` is the isolation bar |
| Second AgentRunner (OpenCode / Claude / …) | Cursor only `(Source: User-confirmed)` |
| gateflow-ops / Mission Control UI | Impact map gateflow-only |
| Template engines beyond `{{var}}` | Upstream v1 |
| Requiring humans to use packages | Humans freeform |
| Rewriting PolicyEngine / pin walker architecture | Enhance invocation path only |
| SaaS prompt registry | Git + pin SSOT |
| Formal INIT-GATEFLOW-001 PRD amendment | Narrow supersession documented here only `(Source: User-confirmed)` |

### Product Principles

1. Consume pin SSOT for automated packaged-skill briefs  
2. Fail closed on package / schema / bind failures  
3. Wave-start carries known context  
4. Thin AgentRunner — rendered message only  
5. Persist `prompt_id`, `prompt_revision`, runner, model  
6. Workflow-agnostic substrate  
7. Gateflow owns `handoff_path` (define + store); not a repo concern  
8. Cursor first — no second runner in this INIT  

---

## 3. AI System Requirements

### Tool Requirements

| Tool / surface | Role in this INIT |
|----------------|-------------------|
| Active prayog-skills pin | Source of `prompts/template.md` + `schema.yaml` |
| Wave-start API (gateflow) | Entry for known context (`ticket`, `initiative`, …) |
| RunStore (PostgreSQL) | Persist `handoff_path`, prompt ids, runner/model, stage outcomes |
| AgentRunner (Cursor) | Execute rendered message only |
| Handoff ingest | Read envelope from stored `run.handoff_path` only |

### Evaluation Strategy

| Dimension | Pass threshold |
|-----------|----------------|
| **Anti-hardcode** | Automated packaged-skill path uses pin render only; invent-prose path removed |
| **Fail closed** | Integration cases: missing package, invalid schema, missing `ticket`, missing handoff at ingest — zero AgentRunner / zero ambient advance |
| **Traceability** | Every successful automated stage row has `prompt_id` + `prompt_revision` |
| **Handoff isolation** | Two-run fixture: ingest for run A never reads run B’s baton |
| **Prove-it** | ≥1 live Cursor automated hop on pinned package (`dispatch: orchestrated` skill) |

Prompt **content** quality eval remains INIT-003 (eval-before-promote); this INIT evaluates **consumption correctness**.

---

## 4. Technical Specifications

### Architecture Overview

```text
wave-start API
      │
      ▼
RunStore: create/continue run
      │  Gateflow.define_handoff_path(run) → store run.handoff_path
      ▼
Policy / trigger (dispatch eligibility — unchanged)
      │
      ▼
PromptResolver (pin) → validate → render {{var}}
      │
      ▼
AgentRunner (Cursor) ← message = render only
      │
      ▼
Persist prompt_id, prompt_revision, runner, model
      │
      ▼
HandoffIngest(run.handoff_path only) → workflow advance
```

### 4.4 Bound-input mapping {#bound-input-mapping}

Shared dictionary from INIT-PRAYOG-SKILLS-003-PROMPTS:

| Name | Required (003 default) | Source | Notes |
|------|------------------------|--------|-------|
| `ticket` | true | **Already known** via wave-start | Required for automate bind |
| `initiative` | false | **Already known** via wave-start when provided | Omit → `""` |
| `skill_id` | true | **Already known** — resolved automated skill id | |
| `workspace` | true | **Already known** — run worker workspace root | |
| `handoff_path` | true | **Gateflow defines and stores on the run** | Independent of repos `(Source: User-confirmed)` |

#### `handoff_path` (normative product rules)

1. Gateflow **defines** `handoff_path` when the run is created or continued.  
2. Gateflow **stores** it on the run record (RunStore SSOT).  
3. Gateflow **injects** the stored value into the prompt bind map.  
4. Gateflow **ingests** the handoff envelope from that stored value only.  
5. `handoff_path` is **not** derived from repository document trees, skill folder
   conventions, or ambient globs/mtime `(Source: User-confirmed)`.  
6. Concrete representation (worker path vs internal URI vs blob key) is an
   **engineering choice** in the gateflow spec PR — ownership stays Gateflow.

**Example:**

```text
wave-start(ticket=GF-241, initiative=INIT-GATEFLOW-005-BOUNDINPUT, …)
  → run_id = run_7f3a
  → store run.handoff_path = <Gateflow-defined location for run_7f3a>
  → bind handoff_path into rendered prompt
  → agent uses that baton
  → ingest reads run.handoff_path only
```

INIT-001 programme config `handoff.artifact_globs` is **not** automated ingest
SSOT under this INIT. For **packaged-skill automated runs**, this INIT
**replaces** INIT-001 ambient `handoff.artifact_globs` / mtime ingest with
Gateflow-stored `run.handoff_path`. **No formal INIT-GATEFLOW-001 PRD amendment**
is required in this INIT `(Source: User-confirmed)`.

### Normative consumer flow (automated runs)

```text
run = create_or_continue_from(wave_start)
run.handoff_path = Gateflow.define_handoff_path(run)   # persist

skill = skill selected for automated run
pkg   = resolve_prompt_package(pin, skill)

if pkg missing OR schema invalid:
    FAIL CLOSED

inputs = {
  ticket:       wave_start.ticket,
  initiative:   wave_start.initiative or "",
  skill_id:     skill.id,
  workspace:    run.workspace,
  handoff_path: run.handoff_path,
}

validate(inputs, pkg.schema.variables)
message = render(pkg.template, inputs)

outcome.prompt_id = pkg.prompt_id
outcome.prompt_revision = pkg.revision
# record runner + model_id (profile/provider optional)

AgentRunner.run(message)
ingest_handoff(run.handoff_path)
```

### Integration Points

| Integration | Direction | Mechanism |
|-------------|-----------|-----------|
| prayog-skills pin | Read | Prompt package resolve (INIT-003) |
| Wave-start API | Inbound | Known context bind `(Source: User-confirmed)` |
| RunStore | Read/write | `handoff_path`, prompt ids, runner/model, stages |
| AgentRunner (Cursor) | Outbound | Rendered message only |
| Workflow `dispatch` | Read | Eligibility only — orthogonal to prompts (INIT-002) |
| gateflow-ops | — | **Out of this INIT** |

### Security & Privacy

| Concern | Requirement |
|---------|-------------|
| **Fail closed** | Never dispatch on invalid/missing package or required bind failure |
| **Handoff isolation** | Per-run stored `handoff_path`; no cross-run ambient discovery |
| **Audit** | Prompt revision + runner/model retained with stage record |
| **Secrets** | No prompt package content used as secret store; existing Gateflow secret handling unchanged |
| **Auth** | Wave-start / status auth use **programme service token** (inherit INIT-001); no alternate auth model in this INIT |

### Repositories

| Repo | This INIT |
|------|-----------|
| **gateflow** | **All delivery** — resolve, bind, render, thin dispatch, persist, handoff define/store/ingest |
| prayog-skills | Monitor — packages already delivered |
| prayog-meta | Monitor — pin bump only if needed |
| gateflow-ops | Not affected / out of delivery |

---

## 5. Risks & Roadmap

### Phased Rollout

| Wave | Intent | Exit (product) |
|------|--------|----------------|
| **W0** | Resolve + wave-start bind + validate/render + thin Cursor dispatch + persist prompt ids; define/store `handoff_path`; strip invent-prose; one automated hop | REQ-1–REQ-7, **REQ-8a**, REQ-9, REQ-10 (one hop on an orchestrated skill) |
| **W1** | Ingest strictly from stored `handoff_path`; harden fail-closed + isolation tests | **REQ-8b**, dual-run isolation AC (REQ-7 telemetry already in W0) |
| **W2** | Broaden automated skills as programme policy allows; verify / dogfood on substrate | Multi-skill evidence under current `dispatch` policy |

### Technical Risks

| Risk | Impact | Mitigation | Owner |
|------|--------|------------|-------|
| Wave-start missing required known fields | No safe bind | Fail closed; document required wave-start fields in spec | Gateflow |
| Pin lag (packages not on consumed pin) | Cannot automate honestly | Fail closed; pin guidance from INIT-003 | PE + Gateflow |
| Residual invent-prose code paths | Contract violation | REQ-5 + anti-hardcode tests | Gateflow |
| Agent writes handoff outside `run.handoff_path` | Ingest failure | Bind path in prompt; ingest only stored path; fail closed | Gateflow |
| Confusion with INIT-001 artifact globs | Wrong SSOT | Narrow supersession for packaged-skill automated runs (this PRD) | PM + Gateflow |
| Scope creep into worktree / repo layout | Delay | Explicit non-goals | PM |

### Assumptions (confirmed)

| ID | Assumption | Status | Trace |
|----|------------|--------|-------|
| A1 | Upstream prompt packages delivered (INIT-003) | Confirmed | REQ-1 |
| A2 | Bound context via wave-start API | Confirmed | REQ-2 |
| A3 | `ticket`, `initiative`, `skill_id`, `workspace` already known | Confirmed | §4.4 |
| A4 | Gateflow defines + stores `handoff_path`; not repo-owned | Confirmed | REQ-8a |
| A5 | Remove Gateflow invent-prose; pin template SSOT | Confirmed | REQ-5 |
| A6 | Impact map gateflow only | Confirmed | Repos |
| A7 | Prove-it = any `dispatch: orchestrated` skill on pin; substrate for any packaged skill | Confirmed | REQ-10 |
| A8 | Second runner out of scope | Confirmed | Non-goals |
| A9 | Worktree not exit criterion | Confirmed | Non-goals |

### Open items (engineering — not product re-litigation)

| # | Item | Disposition |
|---|------|-------------|
| 1 | Wave-start JSON field names for known context | Spec PR `[TBD]` |
| 2 | RunStore column/event field names | Spec PR `[TBD]` |
| 3 | Concrete `handoff_path` representation | Spec PR — ownership locked `[TBD form]` |
| 4 | Prove-it skill id | Pick at W0 per REQ-10 rule (`dispatch: orchestrated` on active pin) `[TBD id]` |

### Decisions log (Discovery → Draft)

| # | Decision | Source |
|---|----------|--------|
| 1 | INIT id INIT-GATEFLOW-005-BOUNDINPUT | User-confirmed |
| 2 | Upstream INIT-PRAYOG-SKILLS-003-PROMPTS | Delivered |
| 3 | Wave-start bind surface | User-confirmed |
| 4 | Known context: ticket, initiative, skill_id, workspace | User-confirmed |
| 5 | handoff_path Gateflow-defined + stored; not repos | User-confirmed |
| 6 | Impact map gateflow only | User-confirmed |
| 7 | Remove invent-prose; pin SSOT | User-confirmed |
| 8 | No second runner | User-confirmed |
| 9 | Workflow-agnostic substrate | User-confirmed |
| 10 | Problem statement uses risk framing (not unsourced current-state fact) | User-confirmed (Resolution CHG-06) |
| 11 | Packaged-skill automated ingest replaces 001 globs; no formal 001 amendment | User-confirmed (Resolution CHG-07) |
| 12 | Canonical product ids `REQ-n` (`FR-n` ≡ `REQ-n`) | User-confirmed / id-conventions (Resolution CHG-10) |
| 13 | REQ-8a / REQ-8b wave split | User-confirmed (Resolution CHG-02) |

---

## 6. Next steps

| Step | Owner | Artifact |
|------|-------|----------|
| 1 | PE | `/validate-requirements` incremental on this Draft |
| 2 | PE | `/prd-impact-map` → **gateflow only** |
| 3 | PE / sponsor | Gate 1 on Draft + impact map |
| 4 | PE | Gateflow spec PR implementing REQ-1–REQ-10 (incl. REQ-8a/8b) |

---

## Appendix A — Relationship to adjacent work

| This INIT | Adjacent |
|-----------|----------|
| Runtime bind / render / thin dispatch / persist / Gateflow-owned handoff | INIT-PRAYOG-SKILLS-003-PROMPTS — prompt SSOT (**delivered**) |
| Honours eligibility when automating | INIT-PRAYOG-SKILLS-002 — `dispatch` (orthogonal) |
| Enhances invocation on control plane; narrow supersession of ambient glob ingest for packaged-skill automated runs | INIT-GATEFLOW-001 — foundation (no formal amendment in this INIT) |
| Does not author prompts | prayog-skills |
| Does not deliver ops UI | gateflow-ops |
| Does not define repo document trees for handoffs | RunStore owns `handoff_path` |
