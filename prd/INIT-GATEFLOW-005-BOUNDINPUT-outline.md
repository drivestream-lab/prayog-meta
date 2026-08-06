# INIT-GATEFLOW-005-BOUNDINPUT — Bound-input skill invocation (outline)

**Status:** outline (synced with Draft) · **Author:** programme PM · **Date:** 2026-07-27  
**Draft PRD:** [INIT-GATEFLOW-005-BOUNDINPUT](./INIT-GATEFLOW-005-BOUNDINPUT.md)  
**Upstream (delivered):** [INIT-PRAYOG-SKILLS-003-PROMPTS](./INIT-PRAYOG-SKILLS-003-PROMPTS.md) — skill prompt packages  
**Related:** [INIT-PRAYOG-SKILLS-002](./INIT-PRAYOG-SKILLS-002.md) (`dispatch` — eligibility, orthogonal to prompts) · [INIT-GATEFLOW-001](./INIT-GATEFLOW-001.md) (control plane foundation)  
**Vision:** [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md)  
**Component:** GATEFLOW · **Type:** platform / invocation substrate

> **Outline** — Draft PRD expanded. Remaining `[TBD]` items are engineering
> (API/RunStore field names, `handoff_path` representation). Engineering detail
> routes to impact map and gateflow spec PR.
> **Invocation SSOT is the prayog-skills prompt package** — Gateflow must not
> invent invocation prose for automated packaged skills.
> **`handoff_path` is Gateflow-owned** (define + store on the run) — **not** a
> repo layout / document-tree concern.
> **Do not treat `dispatch: orchestrated` alone as invocation done.**

---

## Document control

| Field | Value |
|-------|-------|
| Initiative ID | INIT-GATEFLOW-005-BOUNDINPUT |
| Artifact | `prd/INIT-GATEFLOW-005-BOUNDINPUT-outline.md` (outline); Draft PRD [INIT-GATEFLOW-005-BOUNDINPUT](./INIT-GATEFLOW-005-BOUNDINPUT.md) |
| Programme | prayog |
| Primary repo | drivestream-lab/gateflow |
| Supporting / monitor | prayog-skills (pin packages — **delivered**); prayog-meta (pin bump monitor); gateflow-ops (**out of delivery**) |
| Upstream | INIT-PRAYOG-SKILLS-003-PROMPTS (**delivered** — PR #14) |
| Target users | Programme eng (wave-start), PE (pin consume), platform / Gateflow eng |
| Impact map (lean) | **gateflow only** `(Source: User-confirmed)` |
| Product ids | Canonical **`REQ-n`**; **`FR-n` ≡ `REQ-n`**; no `CAP-*` |

---

## 1. Problem statement

INIT-PRAYOG-SKILLS-003-PROMPTS delivered pinned, versioned **prompt packages**
(`prompts/{template.md,schema.yaml,fixtures/}`) for every skill under
`skills/requirements/` and `skills/development/`. The normative consumer
algorithm is: resolve → validate → render (`{{var}}`) → return
`prompt_id` / `prompt_revision` on outcome → hand off rendered message.

**Without a Gateflow runtime substrate** that performs that algorithm when it
automates a skill, automation **would**:

- Invent invocation prose instead of rendering the pin template
- Fail to bind known context via **wave-start API** (`ticket`, `initiative`, …)
- Lack fail-closed package / schema / required-var handling
- Miss persisted `prompt_id` / `prompt_revision` (and runner + model)
- Discover handoffs via ambient repo globs / mtime instead of a Gateflow-owned
  per-run `handoff_path`

…and **would** violate the upstream consumer contract for automated packaged
skills.

---

## 2. Proposed solution (summary)

**Bound-input skill invocation** in Gateflow:

```text
wave-start API
  → create/load run; Gateflow defines handoff_path and stores it on the run
  → authorize automate skill  (workflow `dispatch` + programme trigger — unchanged)
  → resolve prompt package from pin
  → bind: known context + stored handoff_path (+ skill_id, workspace)
  → validate → render template (simple {{var}} only)
  → AgentRunner(message = rendered prompt only)
  → persist prompt_id + prompt_revision (+ runner + model)
  → ingest handoff from run.handoff_path (Gateflow SSOT)
```

**Invocation SSOT** = prayog-skills prompt package on the active pin. Gateflow
**removes** Gateflow-owned invocation prose for automated packaged skills
`(Source: User-confirmed)`.

**Handoff SSOT for automated runs** = Gateflow-defined `handoff_path` stored on
the run — **independent of repository document layout**
`(Source: User-confirmed)`.

`dispatch` remains **eligibility only** (INIT-002) — orthogonal to prompts
(INIT-003). Humans may freeform; this INIT applies to **automated** runs.

---

## 3. Product principles

1. **Consume pin SSOT** — Automated packaged-skill message = rendered
   `prompts/template.md` only; no Gateflow-authored invocation brief  
2. **Fail closed** — Missing package / invalid schema / required bind failure →
   do not call AgentRunner  
3. **Wave-start bind** — v1 bound context enters through the **wave-start API**  
4. **Thin AgentRunner** — Adapter receives rendered message; no lane-/skill-specific
   Gateflow prose in the runner  
5. **Outcome observability** — Persist `prompt_id`, `prompt_revision`, runner, model
   on run/stage outcome  
6. **Workflow-agnostic substrate** — Same pipeline for any skill Gateflow automates;
   works for any packaged skill under upstream coverage  
7. **Gateflow owns handoff_path** — Define + store on the run; ingest from that
   stored value; **not** a repo / `reports/` layout concern; worktree isolation
   is **out of exit** for this INIT  
8. **Cursor first** — Second runner out of scope  

---

## 4. Scope — in

### 4.1 Capabilities

| ID | Capability | Notes |
|----|------------|-------|
| REQ-1 | Resolve prompt package from active pin for automated skill | Per INIT-003 layout + algorithm |
| REQ-2 | Bind inputs for render via **wave-start** + run record | See §4.4 mapping `(Source: User-confirmed)` |
| REQ-3 | Validate required/optional vars per `schema.yaml` | Fail closed on required miss; optional → empty string |
| REQ-4 | Render simple `{{var}}` only | No filters/conditionals in v1 |
| REQ-5 | AgentRunner message = rendered template only | **Remove** Gateflow-owned invocation prose `(Source: User-confirmed)` |
| REQ-6 | Persist `prompt_id` + `prompt_revision` on outcome/stage | Upstream consumer contract; orchestrator owns persistence |
| REQ-7 | Persist runner + model on automated stage | Always `runner` + `model_id`; profile/provider optional null/omit |
| REQ-8a | **Define + store `handoff_path` on the run** | Gateflow SSOT; **not** repo layout — **W0** `(Source: User-confirmed)` |
| REQ-8b | Ingest only from stored `run.handoff_path` | No ambient glob/mtime SSOT — **W1** `(Source: User-confirmed)` |
| REQ-9 | Fail closed before dispatch | Package missing/invalid or bind failure → no agent call |
| REQ-10 | Prove-it on Cursor | ≥1 hop for any `dispatch: orchestrated` skill on pin (pick id at W0) |

### 4.2 Normative flow (automated runs)

```text
run = create_or_continue_from(wave_start)
run.handoff_path = Gateflow.define_handoff_path(run)   # store on run
# not derived from repo document trees

skill  = skill selected for automated run
pkg    = resolve_prompt_package(pin, skill)

if pkg missing OR schema invalid:
    FAIL CLOSED

inputs = {
  ticket:       wave_start.ticket,          # known
  initiative:   wave_start.initiative,      # known (optional → "")
  skill_id:     skill.id,                   # known / resolved
  workspace:    run.workspace,              # known
  handoff_path: run.handoff_path,           # Gateflow-defined + stored
}

validate(inputs, pkg.schema.variables)
message = render(pkg.template, inputs)

outcome.prompt_id = pkg.prompt_id
outcome.prompt_revision = pkg.revision
# also record runner + model

AgentRunner.run(message)   # no Gateflow invocation prose
ingest_handoff(run.handoff_path)   # only this stored path
```

### 4.3 Coverage / prove-it

- Substrate must work for **any** packaged skill (upstream: all
  `skills/requirements/` + `skills/development/`).
- Prove-it uses **whatever Gateflow already automates** under current workflow
  policy; not a separate “lane-first” product split `(Source: User-confirmed)`.

### 4.4 Bound-input mapping (locked) {#bound-input-mapping}

Shared dictionary from INIT-003. Gateflow fills the render map as follows:

| Name | Required (003 default) | Source | Notes |
|------|------------------------|--------|-------|
| `ticket` | true | **Already known** via wave-start | Client / wave identity |
| `initiative` | false | **Already known** via wave-start when provided | Omit → empty string at render |
| `skill_id` | true | **Already known** — resolved automated skill id | Not invented in prompt prose |
| `workspace` | true | **Already known** — run worker workspace root | |
| `handoff_path` | true | **Gateflow defines and stores on the run** | Delivery baton for this run; **independent of repos** `(Source: User-confirmed)` |

#### What `handoff_path` means (product)

- The **durable stage baton location** for this automated run — where the handoff
  envelope for the run is read/written for orchestration.
- Gateflow **allocates** it at run create (or continue), **persists** it on the
  run record, **injects** it into the prompt bind map, and **ingests only** from
  that stored value after the agent finishes.
- It is **not** chosen from git repo conventions (`prd/reports/**`,
  `docs/specification/reports/**`, skill-local folders, or mtime globs).
- Parallel runs each have their own stored `handoff_path` → no cross-run
  handoff poisoning via shared document trees.

**Example:**

```text
wave-start(ticket=GF-241, initiative=INIT-GATEFLOW-005-BOUNDINPUT, …)
  → run_id = run_7f3a
  → Gateflow stores run.handoff_path = <Gateflow-defined location for run_7f3a>
  → bind into prompt: handoff_path = run.handoff_path
  → agent uses that path as the baton
  → Gateflow ingest: read run.handoff_path only
```

Exact storage representation (URI / worker path / blob key) is an engineering
detail in the gateflow spec PR — **ownership and SSOT stay in Gateflow**.

Ambient `handoff.artifact_globs` (INIT-001 programme config) are **not** SSOT
for automated ingest under this INIT. For **packaged-skill automated runs**,
this INIT **replaces** INIT-001 ambient glob/mtime ingest; **no formal 001
amendment** in this INIT `(Source: User-confirmed)`.

---

## 5. Scope — out (explicit non-goals)

| Non-goal | Rationale |
|----------|-----------|
| Authoring / revising prompt templates or schemas | INIT-PRAYOG-SKILLS-003-PROMPTS |
| Eval-before-promote of prompts | Skills / pin process |
| Changing `dispatch` enum or eligibility rules | INIT-PRAYOG-SKILLS-002 |
| Repo-relative handoff discovery (`reports/**`, mtime globs) as SSOT | Gateflow-owned `handoff_path` `(Source: User-confirmed)` |
| Worktree-per-run isolation as exit criterion | Gateflow-stored handoff path is the isolation bar |
| Second AgentRunner (OpenCode / Claude / …) | Cursor path only for this INIT |
| gateflow-ops / Mission Control UI | Out of delivery; impact map gateflow-only |
| Template engines beyond simple `{{var}}` | Upstream v1 constraint |
| Requiring humans to use packages | Humans freeform (INIT-003) |
| Rewriting PolicyEngine / pin walker architecture | Enhance invocation path; not replace control plane |
| SaaS prompt registry | Git + pin SSOT |

---

## 6. Users and jobs-to-be-done

| User | Job to be done |
|------|----------------|
| **Programme eng** | Call wave-start with known context so the automated skill receives the pinned brief |
| **PE** | After skills pin/tag promote, Gateflow picks up new `prompt_revision` without Gateflow code changes |
| **Gateflow engineer** | Implement resolve → bind → validate → render → thin dispatch → persist prompt ids → define/store/ingest `handoff_path` |
| **Platform** | Every automated stage attributable to prompt revision + runner + model; handoff baton owned per run in Gateflow |

---

## 7. Success criteria

| Criterion | Target |
|-----------|--------|
| Pin SSOT | Automated packaged-skill runs use prayog-skills rendered template only |
| No invent-prose | Zero Gateflow-owned invocation briefs on automated packaged-skill path |
| Fail closed | Missing/invalid package or required bind failure → no AgentRunner call |
| Traceability | 100% automated packaged-skill stages record `prompt_id` + `prompt_revision` |
| Observability | Runner + model recorded on those stages |
| Handoff ownership | Every automated run has Gateflow-defined `handoff_path` stored on the run; ingest uses only that value |
| No repo discovery SSOT | Automated ingest does not rely on ambient repo globs / mtime |
| Generality | Substrate not specialized to a single workflow lane |
| Prove-it | ≥1 Cursor automated hop green on pinned package |

---

## 8. Delivery shape (waves — debate)

| Wave | Intent |
|------|--------|
| W0 | Resolve + wave-start bind + validate/render + thin Cursor dispatch + persist prompt ids; **REQ-8a** define/store `handoff_path`; one orchestrated-skill hop; strip invent-prose |
| W1 | **REQ-8b** ingest strictly from stored `handoff_path`; harden fail-closed + isolation |
| W2 | Broaden automated skills as programme policy allows; verify / dogfood on substrate |

Exact FR acceptance criteria and wave exits → Draft PRD.

---

## 9. Dependencies and assumptions

| Dependency | Assumption |
|------------|------------|
| INIT-PRAYOG-SKILLS-003-PROMPTS | **Delivered** — packages + consumer algorithm on rc-2 / pin family |
| Active pin | Contains valid `prompts/` for skills Gateflow automates |
| INIT-PRAYOG-SKILLS-002 | `dispatch` eligibility unchanged; orthogonal to prompts |
| INIT-GATEFLOW-001 | Control plane / AgentRunner / RunStore foundation available to enhance |
| Wave-start API | Carries known context (`ticket`, `initiative`, …) in **gateflow** |
| RunStore | Can store `handoff_path` (and prompt outcome fields) per run/stage |
| Cursor AgentRunner | First (only) runner for this INIT exit |

```text
INIT-PRAYOG-SKILLS-003-PROMPTS (delivered)
  prompts/template.md + schema.yaml + fixtures
           │
           ▼ pin
INIT-GATEFLOW-005-BOUNDINPUT (this INIT)
  wave-start + known context
  → Gateflow defines/stores handoff_path on run
  → resolve → validate → render → thin AgentRunner
  → persist prompt ids → ingest run.handoff_path
```

---

## 10. Risks `[TBD — expand in Draft]`

| Risk | Mitigation sketch |
|------|-------------------|
| Wave-start missing required known fields (e.g. `ticket`) | Fail closed; document wave-start required set in Draft |
| Pin lag (packages not on consumed pin) | Fail closed; pin guidance from INIT-003 |
| Residual hardcoded prose paths | Explicit REQ-5 remove; AC + tests for anti-hardcode |
| Agent writes handoff somewhere other than `run.handoff_path` | Prompt binds the path; ingest only stored path; fail closed if missing |
| Confusion with INIT-001 artifact globs | Draft states globs are not automated ingest SSOT under this INIT |
| Over-scope into worktree / repo layout design | Non-goal; Gateflow-owned path + store is exit bar |

---

## 11. Open questions

Discovery locks are in Appendix C. Remaining for Draft / PE / eng:

1. Wave-start JSON field names for already-known context (`ticket`, `initiative`, …) `[TBD — API shape]`  
2. RunStore field names for `handoff_path`, `prompt_id`, `prompt_revision` `[TBD — schema]`  
3. Concrete representation of Gateflow-defined `handoff_path` (internal URI vs worker path vs blob key) — **ownership locked; form open** `[TBD — spec PR]`  
4. Prove-it skill id under current automation policy `[TBD — pick at Draft/W0]`  

---

## 12. Next steps (process)

| Step | Owner | Artifact |
|------|-------|----------|
| 1 | PM / PE | Review Draft PRD (outline synced) |
| 2 | PE | `/validate-requirements` on Draft |
| 3 | PE | `/prd-impact-map` → **gateflow only** |
| 4 | PE / sponsor | Gate 1 |
| 5 | PE | Gateflow spec PR after Gate 1 |

---

## Appendix A — Outline → full PRD checklist

- [ ] Per-FR acceptance criteria (testable)
- [ ] Wave-start known-context field table (API-level)
- [ ] RunStore fields: `handoff_path`, prompt ids, runner, model
- [ ] Anti-hardcode AC (no Gateflow invocation prose)
- [ ] Fail-closed matrix (package / schema / bind / missing handoff at ingest)
- [ ] Explicit non-SSOT: ambient repo globs for automated ingest
- [ ] User stories with acceptance criteria
- [ ] Risk register with owners
- [ ] W0/W1/W2 exit bars

---

## Appendix B — Relationship to adjacent work

| This INIT | Adjacent |
|-----------|----------|
| Runtime bind / render / thin dispatch / persist / Gateflow-owned handoff | INIT-PRAYOG-SKILLS-003-PROMPTS — prompt package SSOT (**delivered**) |
| Honours eligibility when automating | INIT-PRAYOG-SKILLS-002 — `dispatch` (orthogonal) |
| Enhances invocation on existing control plane | INIT-GATEFLOW-001 — foundation; artifact globs not automated ingest SSOT here |
| Does **not** author prompts | prayog-skills |
| Does **not** deliver ops UI | gateflow-ops out of impact delivery |
| Does **not** define repo document trees for handoffs | Gateflow run record owns `handoff_path` |

---

## Appendix C — Discovery decisions (locked 2026-07-27)

| # | Decision |
|---|----------|
| 1 | Initiative id: **INIT-GATEFLOW-005-BOUNDINPUT** |
| 2 | Upstream prompt SSOT: **INIT-PRAYOG-SKILLS-003-PROMPTS** (delivered) |
| 3 | Bound context enters via **wave-start API** |
| 4 | `ticket`, `initiative`, `skill_id`, `workspace` are **already known** context |
| 5 | **`handoff_path`**: Gateflow **defines and stores** on the run; **nothing to do with repos** |
| 6 | Isolation exit: Gateflow-stored per-run `handoff_path` (not worktree; not repo globs) |
| 7 | Impact map: **gateflow only** |
| 8 | Prove-it: whatever Gateflow already automates; substrate for **any** packaged skill |
| 9 | **Remove** Gateflow-owned invocation prose — SSOT is prayog-skills prompt package |
| 10 | Second runner: **out of scope** |
| 11 | Workflow-agnostic substrate (no lane-ordered product split) |
| 12 | Humans freeform unchanged; automated runs fail closed per INIT-003 |
| 13 | Problem statement uses **risk framing** (Resolution CHG-06) |
| 14 | Packaged-skill automated ingest **replaces** 001 globs; no formal 001 amendment (CHG-07) |
| 15 | Canonical **`REQ-n`** (`FR-n` ≡ `REQ-n`); REQ-8a / REQ-8b wave split (CHG-02, CHG-10) |
