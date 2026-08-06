# INIT-PRAYOG-SKILLS-003-PROMPTS — Skill prompt package (outline)

**Status:** outline (aligned with Draft) · **Author:** programme PM · **Date:** 2026-07-27  
**Draft PRD:** [INIT-PRAYOG-SKILLS-003-PROMPTS](./INIT-PRAYOG-SKILLS-003-PROMPTS.md)  
**Related:** [INIT-PRAYOG-SKILLS-002](./INIT-PRAYOG-SKILLS-002.md) (`dispatch` — orthogonal)  
**Downstream (later):** INIT-GATEFLOW-005-BOUNDINPUT  
**Component:** PRAYOG-SKILLS · **Type:** platform / delivery contract

> Prompt packages for **every** skill under **`skills/requirements/`** and
> **`skills/development/`** — **no exceptions** (Draft inventory **13/13** =
> 4 + 9). `dispatch` is workflow-only / configurable and does **not** select
> coverage. Packages are independent of how Gateflow runs a node. Humans may
> freeform; automated consumers fail closed.

---

## Document control

| Field | Value |
|-------|-------|
| Initiative ID | INIT-PRAYOG-SKILLS-003-PROMPTS |
| Primary repo | drivestream-lab/prayog-skills |
| Track | `features/rc-2` / `v0.5.0-rc.2` family |
| Coverage | `skills/requirements/*` ∪ `skills/development/*` — **no exceptions** |
| Contract | `sdd-delivery/v2` + CHANGELOG only (no `v2.x` label) |
| Gate 1 | Solo Gate 1 on this Draft — **not** Joint Gate with BOUNDINPUT |
| Predecessor | INIT-PRAYOG-SKILLS-002 (`dispatch` orthogonal) |

---

## 1. Problem

Skills under `skills/requirements/` and `skills/development/` need a pinned,
versioned **invocation brief** so Gateflow (and other orchestrator consumers) do
not invent prose when they automate a skill. `dispatch` (INIT-002) answers
workflow **eligibility** only — it does not define the brief and does not select
which skills get packages.

---

## 2. Solution

Per-skill package beside **every** skill in both directories:

```text
skills/<area>/<skill-id>/     # area ∈ {requirements, development}
  SKILL.md                    # procedure SSOT (unchanged role)
  prompts/
    template.md               # simple {{var}} only
    schema.yaml               # prompt_id + semver revision + variables
    fixtures/                 # ≥1 golden; normalized match
```

Mental model:

```text
/<skill> + versioned prompt + bound inputs
  → render → outcome{prompt_id, prompt_revision}
  → hand off rendered message (invoke skill)
  → orchestrator persists (out of this INIT)
```

“Hand off / invoke” ≠ workflow `dispatch` (INIT-002 eligibility).

**Consume model:** packages always present for target dirs; Gateflow (or any
orchestrator) loads the package when it automates that skill under then-current
workflow policy. Humans not required to use the package.

---

## 3. Principles

1. Procedure vs invocation  
2. Pin / rc-2 resolve for automated consumers  
3. Orchestrator-neutral contract language  
4. Schema-bound inputs  
5. Independent prompt revisions  
6. **`dispatch` ≠ prompts**  
7. **Directory coverage, no exceptions** — `requirements/` + `development/`  
8. Humans freeform; automated runs fail closed  
9. Eval before promote  

---

## 4. Scope in

| ID | Capability |
|----|------------|
| FR-1 | Per-skill `prompts/` for `area` ∈ `{requirements, development}` |
| FR-2 | `template.md` + `schema.yaml`; semver `revision`; independent of skill identity |
| FR-3 | Shared variable dictionary; schema sets required/optional |
| FR-4 | Document resolve algorithm; fail closed; outcome ids; dispatch-independent; humans freeform |
| FR-5 | Contract tests for **all** v1 targets (**13/13**) |
| FR-6 | Delivery-contract / skill-reference docs (orchestrator-neutral) |
| FR-7 | Eval-before-promote: checklist + ≥1 golden fixture per skill |
| FR-8 | CHANGELOG + rc-2 pin guidance |
| FR-9 | Coverage = both directories, **no exceptions**; `dispatch` irrelevant |

### Target inventory (Draft — normative, no exceptions)

#### `skills/requirements/` (4)

| Skill node id |
|---------------|
| `validate-requirements` |
| `review-findings` |
| `update-documents` |
| `prd-impact-map` |

#### `skills/development/` (9)

| Skill node id |
|---------------|
| `spec-draft` |
| `initiative-feasibility` |
| `spec-technical-review` |
| `spec-implementation-plan` |
| `board-seed` |
| `pre-implement` |
| `loop-spec` |
| `verify` |
| `ground-spec` |

**Coverage rules:** New skill under either directory → must get a package.
`dispatch` is not a coverage criterion.

### Shared variables (v1)

| Name | v1 default `required` |
|------|------------------------|
| `ticket` | true |
| `initiative` | false |
| `handoff_path` | true |
| `workspace` | true |
| `skill_id` | true |

Shared meanings locked; **v1 schemas MUST use these defaults** (deviation =
Decision + MAJOR).

### Template

Simple `{{var}}` only — no filters/conditionals.

---

## 5. Scope out

| Non-goal | Rationale |
|----------|-----------|
| `dispatch` selects prompt coverage | Workflow-only; configurable independently |
| Exceptions inside `requirements/` or `development/` | Explicitly forbidden |
| Humans must use packages | Freeform allowed |
| Gateflow runtime / BOUNDINPUT | Separate INIT |
| `sdd-delivery/v2.x` label | CHANGELOG only |
| Template filters/conditionals | Simple `{{var}}` |
| W0 single-skill exemplar | All packages in W1 |
| SaaS prompt registry | Git + pin first |

---

## 6–8. Users / contract / success

| Persona | Need |
|---------|------|
| Skills maintainer | Packages beside every requirements/ and development skill |
| PE | Promote via `revision` + pin/tag |
| Orchestrator engineer | Resolve from pin; fail closed; outcome ids |
| Programme eng | Trust 13/13 briefs on pin |

**Success:** **13/13** packages; fail closed on automate; coverage independent of
`dispatch`; eval gate before pin/tag promote.

---

## 9. Waves

| Wave | Intent | Exit |
|------|--------|------|
| **W1** | Contract + **all 13** packages + tests + eval docs | FR-1–FR-7 + FR-9 |
| **W2** | CHANGELOG / pin guidance | FR-8 |

No W0 exemplar.

---

## 10–12. Dependencies / risks / open Qs

| Dependency | Note |
|------------|------|
| INIT-002 | `dispatch` orthogonal / configurable |
| Both skill directories | Full inventory = coverage |
| BOUNDINPUT | Separate INIT (later) |
| rc-2 track | Active |

**Risks:** incomplete 13-skill coverage; orchestrator hardcodes prose; prompt
drift from `SKILL.md`.

**Open questions:** none.

---

## Appendix C — Decisions (locked)

| # | Decision |
|---|----------|
| 1–8 | Layout beside skill; `template.md` + `schema.yaml` + fixtures; semver revision; shared five vars; simple `{{var}}`; normalized fixtures; eval checklist+fixture; rc-2 track |
| 9 | Coverage = **all** skills under `skills/requirements/` **and** `skills/development/` — **no exceptions** |
| 10–16 | BOUNDINPUT separate; CHANGELOG-only; W1 ships all packages; outcome returns `prompt_id` + `prompt_revision`; orchestrator persists |
| 17 | `dispatch` is workflow-only / configurable; does not drive prompt coverage |
| 18 | Packages independent of how Gateflow runs the node; consume when automated; humans freeform |
