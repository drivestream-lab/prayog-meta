# INIT-PRAYOG-SKILLS-003-PROMPTS — Skill prompt package (requirements + development)

**Status:** draft PRD · **Author:** programme PM · **Date:** 2026-07-27  
**Outline:** [INIT-PRAYOG-SKILLS-003-PROMPTS-outline](./INIT-PRAYOG-SKILLS-003-PROMPTS-outline.md)  
**Vision:** [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md)  
**Predecessor:** [INIT-PRAYOG-SKILLS-002](./INIT-PRAYOG-SKILLS-002.md) (`dispatch` on pin `v0.5.0-rc.2` — workflow eligibility, separate from prompts)  
**Downstream:** INIT-GATEFLOW-005-BOUNDINPUT (bind / render / dispatch / outcome fields — separate INIT)  
**Component:** PRAYOG-SKILLS · **Type:** platform / delivery contract

> **Draft PRD** for Gate 1 review. Engineering implementation routes to
> `drivestream-lab/prayog-skills` on **`features/rc-2`** under the **current
> rc-2 tag line** (`v0.5.0-rc.2` family). This INIT defines the **versioned
> invocation brief** for **every** skill under **`skills/requirements/`** and
> **`skills/development/`** — **no exceptions**.
> **`dispatch` is workflow eligibility only** — configurable (manual today,
> orchestrated later); it does **not** select which skills get prompt packages.
> Prompt packages are independent of how Gateflow runs a node. Humans may
> freeform; Gateflow (or any orchestrator) consumes the package when it
> automates that skill under then-current workflow policy.

---

## Document control

| Field | Value |
|-------|-------|
| Initiative ID | INIT-PRAYOG-SKILLS-003-PROMPTS |
| Programme | prayog |
| Primary repo | drivestream-lab/prayog-skills |
| Target branch | **`features/rc-2`** |
| Target tag / pin line | **Current rc-2 tag** (`v0.5.0-rc.2` family); promote via pin/tag bump on this line `(Source: User-confirmed)` |
| Target users | Skills maintainers, PE (pin promote), Gateflow / orchestrator consumers |
| Gate 1 coupling | Gate 1 on this Draft before merge of prompt-package contract to `features/rc-2`; **not** Joint Gate with BOUNDINPUT (downstream INIT not yet drafted) |
| Contract id | `sdd-delivery/v2` (rc-2 additive prompt-package semantics) |
| Predecessor | INIT-PRAYOG-SKILLS-002 — `dispatch` on pin **`v0.5.0-rc.2`** (orthogonal to prompts) |

---

## 1. Executive Summary

### Problem Statement

Skills under `skills/requirements/` and `skills/development/` need a pinned,
versioned **invocation brief** so Gateflow (and other orchestrator consumers) do
not invent prose when they automate a skill. Without a prompt package, quality
cannot improve by prompt revision + pin/tag alone. `dispatch`
(INIT-PRAYOG-SKILLS-002) only answers workflow **eligibility** and is
**configurable over time**; it does **not** define the brief and does **not**
select which skills get packages `(Source: User-confirmed)`.

### Proposed Solution

Add a **prompt package** beside **every** skill under:

- `skills/requirements/`
- `skills/development/`

**No exceptions** `(Source: User-confirmed)`.

```text
SKILL.md                 →  procedure SSOT (unchanged role)
prompts/template.md      →  invocation brief (simple {{var}} substitution only)
prompts/schema.yaml      →  variable schema + prompt_id + semver revision
pin / rc-2 tag           →  what consumers resolve when programme promotes
```

Mental model:

```text
/<skill>  +  versioned prompt  +  bound inputs (per schema)
         →  render → outcome{prompt_id, prompt_revision} → dispatch
         →  orchestrator persists outcome (out of this INIT)
```

**Prompt packages are independent of how Gateflow runs the node.** Gateflow
honours `workflow.yaml` / `dispatch` (manual today, may be orchestrated later).
When it automates a skill, it loads that skill’s package from the pin
`(Source: User-confirmed)`.

**Humans** executing skills may supply their own comments and are **not**
required to use the package `(Source: User-confirmed)`.

### Success Criteria (rc-2 delivery)

| KPI | Target | Measurement |
|-----|--------|-------------|
| **Coverage** | 100% of skills under `skills/requirements/` **and** `skills/development/` have a valid `prompts/` package | Contract tests vs directory inventory (**13/13** at Draft time) |
| **Resolvability** | Consumer loads `prompt_id` + `revision` + template from pin without hardcoded invocation prose; outcome returns both ids | Delivery-contract algorithm + consumer fixture (BOUNDINPUT later) |
| **Independence** | Prompt `revision` can ship/promote without skill identity change; coverage independent of `dispatch` | Package schema + CHANGELOG + contract tests |
| **Neutrality** | Docs refer to “orchestrator consumers,” not a single product as SSOT | Doc review |
| **Eval gate** | Checklist + ≥1 golden fixture per target skill before pin/tag promote | Eval doc + fixture files in package |
| **Tests** | Contract suite fails if any target skill lacks valid package | CI on `features/rc-2` |
| **Track** | All delivery on `features/rc-2` / current rc-2 tag line | Branch + tag evidence |

---

## 2. User Experience & Functionality

### User Personas

| Persona | Role | Primary need |
|---------|------|--------------|
| **Skills maintainer** | Authors skill packages | Keep invocation quality next to every requirements/ and development skill |
| **PE** | Contract / pin owner | Promote evaluated prompt revisions via pin/tag without orchestrator code changes |
| **Orchestrator engineer** | Gateflow / future consumers | Resolve prompt + schema from pin when automating a skill; fail closed if missing |
| **Programme eng** | Delivery consumer | Trust pinned briefs exist for all SDD skills in those two directories |

### User Stories & Acceptance Criteria

#### US-1 — Maintainer authors a versioned invocation brief beside each target skill

**As a** skills maintainer, **I want** a `prompts/` package beside every skill under `skills/requirements/` and `skills/development/` **so that** briefs live with the skill with no coverage exceptions.

**Acceptance criteria:**

- [ ] Layout is `skills/<area>/<skill-id>/prompts/{template.md,schema.yaml,fixtures/}` for every skill where `<area>` ∈ `{requirements, development}`
- [ ] `SKILL.md` remains procedure SSOT; template does not replace procedure
- [ ] **No exceptions** within those two directories `(Source: User-confirmed)`

#### US-2 — PE promotes prompt quality via revision + pin/tag

**As a** PE, **I want** to bump `revision` (semver) and promote via the current rc-2 pin/tag **so that** consumers pick up improved briefs without skill rename or orchestrator code changes.

**Acceptance criteria:**

- [ ] `schema.yaml` carries `prompt_id` and `revision` (semver); revision is independent of skill identity
- [ ] Eval-before-promote checklist + golden fixture pass before pin/tag promote
- [ ] CHANGELOG documents prompt revision bumps on the rc-2 track

#### US-3 — Orchestrator resolves brief from pin and fails closed

**As an** orchestrator engineer, **I want** a normative resolution algorithm **so that** when Gateflow (or any orchestrator) automates a skill, it loads the pinned brief and never invents prose.

**Acceptance criteria:**

- [ ] Docs document resolve → validate → render → return `prompt_id`/`prompt_revision` on outcome
- [ ] Missing/invalid package → **FAIL CLOSED** for automated runs (no ad-hoc fallback prose)
- [ ] Consumer surfaces `prompt_id` + `prompt_revision` on the run **outcome**; orchestrator owns persistence
- [ ] Resolution is **not** gated on today’s `dispatch` value — packages exist regardless; consumption happens when the orchestrator runs that skill under then-current workflow policy `(Source: User-confirmed)`
- [ ] Humans may freeform; not required to use the package `(Source: User-confirmed)`

#### US-4 — Contract tests enforce directory coverage

**As a** skills maintainer, **I want** CI to fail when any skill under `skills/requirements/` or `skills/development/` lacks a valid prompt package **so that** coverage regressions cannot merge.

**Acceptance criteria:**

- [ ] Contract tests assert `template.md` + `schema.yaml` + `fixtures/` for every skill in both directories
- [ ] Schema validates: `prompt_id`, semver `revision`, variables against shared dictionary rules
- [ ] Every `{{var}}` in template is declared in schema
- [ ] ≥1 golden fixture per target skill renders (normalized match)
- [ ] Coverage tests do **not** use `dispatch` as a selection criterion

### Functional Requirements (FR) — with Acceptance Criteria

| ID | Requirement | Acceptance criteria |
|----|-------------|---------------------|
| **FR-1** | Per-skill `prompts/` beside the skill | Path `skills/<area>/<skill-id>/prompts/` for `area` ∈ `{requirements, development}` |
| **FR-2** | Versioned template + schema | `template.md` + `schema.yaml`; semver `revision`; independent of skill identity |
| **FR-3** | Shared variable dictionary + per-skill schema | §4 dictionary; schema sets required/optional |
| **FR-4** | Document consumer resolution algorithm | Fail closed for automated runs; outcome returns ids; independent of current `dispatch`; humans freeform |
| **FR-5** | Contract tests for all v1 targets | Every skill under both directories; **13/13** at Draft inventory |
| **FR-6** | Delivery-contract / skill-reference docs | Orchestrator-neutral naming |
| **FR-7** | Eval-before-promote | Checklist + ≥1 golden fixture per target skill |
| **FR-8** | CHANGELOG + rc-2 release notes | Pin guidance on `features/rc-2` / `v0.5.0-rc.2` family |
| **FR-9** | v1 coverage = `skills/requirements/*` ∪ `skills/development/*` | **No exceptions**; `dispatch` irrelevant to coverage `(Source: User-confirmed)` |

### Package layout (normative)

```text
skills/<area>/<skill-id>/     # area ∈ {requirements, development}
  SKILL.md
  prompts/
    template.md
    schema.yaml
    fixtures/
      happy_path.inputs.yaml
      happy_path.expected.md
```

**Example `schema.yaml`:**

```yaml
prompt_id: validate-requirements
revision: 1.0.0
variables:
  ticket:
    required: false
    type: string
  initiative:
    required: false
    type: string
  handoff_path:
    required: false
    type: string
  workspace:
    required: true
    type: string
  skill_id:
    required: true
    type: string
```

### Semver revision rules (prompt package)

| Bump | When |
|------|------|
| **MAJOR** | Required variable added/removed/renamed; breaking binding change |
| **MINOR** | New optional variable; additive non-breaking guidance |
| **PATCH** | Wording / formatting only |

### Shared variable dictionary (v1)

| Name | Type | Shared meaning | Recommended default in target schemas |
|------|------|----------------|---------------------------------------|
| `ticket` | string | Tracker / Forge ticket id | required |
| `initiative` | string | Programme INIT id | optional |
| `handoff_path` | string | Path to latest handoff artifact | required |
| `workspace` | string | Checkout / workspace root | required |
| `skill_id` | string | Resolved skill node id | required |

**Rules:** Shared meanings locked; each `schema.yaml` sets `required` (normative); recommended defaults are guidance only `(Source: User-confirmed)`. Missing optional → empty string at render.

### Error Handling (consumer-facing — automated runs)

| Condition | Required behavior |
|-----------|-------------------|
| Automated run of a packaged skill | Load prompt package from pin |
| Package missing / schema invalid | **FAIL CLOSED** |
| Required bound inputs fail validation | **FAIL CLOSED** |
| Optional bound input omitted | Empty string substitution |
| Package valid | Render; return `prompt_id` + `prompt_revision` on outcome |
| Human executing skill manually | Not required to use package; may freeform `(Source: User-confirmed)` |

### Error Handling (producer-side)

| Condition | Required behavior |
|-----------|-------------------|
| Any `skills/requirements/*` or `skills/development/*` skill missing `prompts/` | Contract test **fail** |
| Invalid schema / non-semver / undeclared `{{var}}` / missing fixture | Contract test **fail** |

### Non-Goals (rc-2 delivery)

| Non-goal | Rationale |
|----------|-----------|
| Using `dispatch` to select prompt coverage | Workflow-only; configurable independently `(Source: User-confirmed)` |
| Exceptions inside `requirements/` or `development/` | Explicitly forbidden `(Source: User-confirmed)` |
| Requiring humans to use packages | Freeform allowed |
| Gateflow runtime / BOUNDINPUT | Separate INIT |
| `sdd-delivery/v2.x` label | CHANGELOG only |
| Template filters/conditionals | Simple `{{var}}` |
| W0 single-skill exemplar | Ship all target packages in W1 |
| SaaS prompt registry | Git + pin first |

### Product Principles

1. Procedure vs invocation  
2. Pin / rc-2 resolve for automated consumers  
3. Orchestrator-neutral contract language  
4. Schema-bound inputs  
5. Independent prompt revisions  
6. **`dispatch` ≠ prompts** — workflow configurable; packages always present for target dirs `(Source: User-confirmed)`  
7. **Directory coverage, no exceptions** — `requirements/` + `development/` `(Source: User-confirmed)`  
8. Humans freeform; automated runs fail closed  
9. Eval before promote  

---

## 3. AI System Requirements

Versioned briefs for all skills under `skills/requirements/` and
`skills/development/`. Runtime rendering belongs to consumers (BOUNDINPUT).

### Evaluation Strategy

| Dimension | Pass threshold (rc-2) |
|-----------|------------------------|
| **Coverage** | **13/13** packages valid (4 requirements + 9 development) at Draft inventory |
| **Schema / template / fixtures** | 100% valid; normalized fixture match |
| **Eval-before-promote** | CHANGELOG lists `prompt_id@revision` + fixtures green + checklist |
| **Anti-hardcode** | Zero hardcoded invocation prose for automated runs of packaged skills |

---

## 4. Technical Specifications

### Architecture Overview

```text
skills/{requirements,development}/<skill-id>/prompts/   # coverage set
        │
        ▼
delivery-contract / refs      # resolve algorithm
        │
        ▼
contract tests (directory inventory)
        │
        ▼
pin / v0.5.0-rc.2 family
        │
        ▼
INIT-GATEFLOW-005-BOUNDINPUT  # when orchestrator automates skill
```

### Target skills (v1 coverage — normative)

**Every skill under both directories — no exceptions** `(Source: User-confirmed)`:

#### `skills/requirements/` (4)

| Skill node id | Required |
|---------------|----------|
| `validate-requirements` | **Yes** |
| `review-findings` | **Yes** |
| `update-documents` | **Yes** |
| `prd-impact-map` | **Yes** |

#### `skills/development/` (9)

| Skill node id | Required |
|---------------|----------|
| `spec-draft` | **Yes** |
| `initiative-feasibility` | **Yes** |
| `spec-technical-review` | **Yes** |
| `spec-implementation-plan` | **Yes** |
| `board-seed` | **Yes** |
| `pre-implement` | **Yes** |
| `loop-spec` | **Yes** |
| `verify` | **Yes** |
| `ground-spec` | **Yes** |

**Coverage rules:**

- New skill added under either directory → **must** get a prompt package.
- **`dispatch` is not a coverage criterion** `(Source: User-confirmed)`.
- Package existence is independent of whether Gateflow currently runs the node as manual or orchestrated.

### Consumer resolution algorithm (normative — automated runs)

```text
skill = packaged skill selected for automated run
        # under skills/requirements/ or skills/development/
pkg   = resolve_prompt_package(pin, skill)

if pkg missing OR schema invalid:
    FAIL CLOSED

validate bound_inputs against pkg.schema.variables
message = render(pkg.template, bound_inputs)   # simple {{var}} only
outcome.prompt_id = pkg.prompt_id
outcome.prompt_revision = pkg.revision
dispatch(message)
# orchestrator persists — out of this INIT
```

**Humans:** may ignore the package `(Source: User-confirmed)`.

### Contract and version strategy

| Item | rc-2 lean |
|------|-----------|
| Branch / tag | `features/rc-2` / `v0.5.0-rc.2` family `(Source: User-confirmed)` |
| Contract id | `sdd-delivery/v2` + CHANGELOG only |
| `dispatch` | Orthogonal and configurable; unchanged by this INIT `(Source: User-confirmed)` |
| Outcome | Must return `prompt_id` + `prompt_revision`; orchestrator persists `(Source: User-confirmed)` |

### Integration Points

| Integration | Mechanism |
|-------------|-----------|
| INIT-PRAYOG-SKILLS-002 | `dispatch` eligibility — orthogonal |
| INIT-GATEFLOW-005-BOUNDINPUT | Consume package when automating; return outcome fields |
| meta harness | Pin to rc-2 tag |
| delivery-contract docs | Algorithm + layout |

---

## 5. Risks & Roadmap

### Phased Rollout

| Wave | Scope | Exit |
|------|-------|------|
| **W1** | Contract + **all 13** packages + tests + eval docs | FR-1–FR-7 + FR-9; **13/13** |
| **W2** | CHANGELOG / pin guidance | FR-8 |

No W0 exemplar — all packages in W1 `(Source: User-confirmed)`.

### Dependencies

| Dependency | Assumption |
|------------|------------|
| INIT-002 | `dispatch` orthogonal / configurable |
| `skills/requirements/` + `skills/development/` | Full inventory = coverage `(Source: User-confirmed)` |
| BOUNDINPUT | Separate INIT (later) |
| rc-2 track | Active |

#### Assumptions

| ID | Assumption | Status | Dependent FRs |
|----|------------|--------|---------------|
| A1 | Delivery on `features/rc-2` / `v0.5.0-rc.2` family | Confirmed | FR-8 |
| A2 | Split layout `template.md` + `schema.yaml` | Confirmed | FR-1, FR-2 |
| A3 | Semver `revision` in schema | Confirmed | FR-2, FR-4 |
| A4 | Eval = checklist + golden fixtures | Confirmed | FR-7 |
| A5 | Coverage = all skills in `requirements/` + `development/` (no exceptions) | Confirmed | FR-5, FR-9 |
| A6 | Shared dictionary five names | Confirmed | FR-3 |
| A7 | Simple `{{var}}`; normalized fixtures | Confirmed | FR-2, FR-5 |
| A8 | All target packages ship in W1 (no W0 exemplar) | Confirmed | FR-5, FR-9 |
| A9 | Outcome returns `prompt_id` + `prompt_revision`; orchestrator persists | Confirmed | FR-4 |
| A10 | `dispatch` does not select coverage; packages independent of run mode; humans freeform | Confirmed | FR-4, FR-9 |

### Risk Register

| Risk | Mitigation | Owner |
|------|------------|-------|
| Incomplete 13-skill coverage | Directory contract tests | PE |
| Orchestrator hardcodes prose | Fail closed + BOUNDINPUT AC | PE + Gateflow |
| Prompt drift from SKILL.md | Eval-before-promote | PE |
| Layout churn | Freeze at Gate 1; ship all 13 together | PM |

### Decisions (locked) {#decisions-resolved}

| # | Decision | Resolution |
|---|----------|------------|
| 1–8 | Prior locks | Initiative id, layout, file shape, semver, shared vars, eval, release track, etc. |
| 9 | v1 coverage | **All skills under `skills/requirements/` and `skills/development/` — no exceptions** `(Source: User-confirmed)` |
| 10–16 | Prior locks | Downstream separate; CHANGELOG-only; simple `{{var}}`; normalized fixtures; W1 all packages; outcome ids |
| 17 | Dispatch vs prompts | **`dispatch` is workflow-only / configurable**; does not drive prompt coverage `(Source: User-confirmed)` |
| 18 | Consume model | Packages independent of how Gateflow runs the node; consume when automated; humans freeform `(Source: User-confirmed)` |

---

## Appendix A — Traceability

Outline §1–§12 map to Draft §§1–5 and Decisions (see outline).

## Appendix B — Adjacent work

| This INIT | Adjacent |
|-----------|----------|
| Prompt packages for requirements + development | INIT-002 `dispatch` (**orthogonal**, configurable) |
| Consume on automate | INIT-GATEFLOW-005-BOUNDINPUT (later) |

## Appendix C — Process next steps

| Step | Owner | Artifact |
|------|-------|----------|
| 1 | PE | Incremental `/validate-requirements` |
| 2 | PE / sponsor | Gate 1 |
| 3 | PE | `/prd-impact-map` → prayog-skills |
| 4 | PE | W1 — all 13 packages on `features/rc-2` |
| 5 | PE | W2 CHANGELOG / pin guidance |
| 6 | PM | BOUNDINPUT outline later |

## Appendix D — Checklist

- [x] Per-FR AC  
- [x] Full v1 target list (4 + 9 = 13)  
- [x] Shared dictionary  
- [x] Layout + revision scheme  
- [x] Eval bar  
- [x] Resolution algorithm (dispatch-independent)  
- [x] Risks + decisions  
- [x] User stories  
