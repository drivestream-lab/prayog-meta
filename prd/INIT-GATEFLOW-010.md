# INIT-GATEFLOW-010 — Engineering-lane pin tip executor parity

**Status:** draft PRD · **Author:** programme PM · **Date:** 2026-08-05  
**Outline:** [INIT-GATEFLOW-010-outline](./INIT-GATEFLOW-010-outline.md)  
**Vision:** [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md)  
**Component:** GATEFLOW · **Type:** platform / delivery control plane  
**Predecessor:** INIT-GATEFLOW-009 feature readiness (this INIT defines the target eng surface)

> **Draft PRD** — Locked decisions G1–G11 from outline review. Pin SSOT:
> prayog-skills `sdd-delivery/v2` @ **`v0.5.0-rc.2`**. Primary delivery =
> **gateflow only**. Engineering detail routes to impact map and gateflow
> spec PR. PM lane and meta purge are **out of scope**.
>
> **As-built baseline (Source: User-confirmed):** gap claims below reflect
> gateflow tip **`2791ab2e841e277cb53e0cb16faaa375da66c1f1`** as of
> **2026-08-05** (audit / outline locks). Re-confirm if tip moves before delivery freeze.

---

## Document control

| Field | Value |
|-------|-------|
| Initiative ID | INIT-GATEFLOW-010 |
| Programme | prayog |
| Primary repo | drivestream-lab/gateflow |
| Supporting repos | prayog-meta (this PRD), prayog-skills (pin consume-only — **no** redesign) |
| Skills pin | `agent_skills.ref: v0.5.0-rc.2` (harness SHA **must equal** submodule tip) |
| Primary delivery | **gateflow only** through freeze |
| Target users | PE (lane execution), tech lead (gates / freeze) |
| Draft locks | EPIC → Done at closure **enter** (after wave Done-gate, before purge-app); Enter-at path **`POST /api/v1/initiatives/closure/start`** |
| As-built baseline | 2026-08-05 · gateflow `2791ab2e841e277cb53e0cb16faaa375da66c1f1` (Source: User-confirmed) |

---

## 1. Executive Summary

### Problem Statement

Gateflow remounts prayog-skills tip `v0.5.0-rc.2` but does not fully **execute**
the engineering graph: `update_board_status` nodes fail to parse (walker BLOCK),
create-tickets predicates are incomplete, implement-start does not yet
**board-resolve** tickets / fail closed on missing board issues (a non-empty
`ticket_id` string alone is insufficient), and eng initiative purge has no
Enter-at — so PE cannot run tip-faithful spec → tickets → implement → closeout →
eng closure via Gateflow alone. *(Source: User-confirmed; as-built baseline above.)*

### Proposed Solution

Make Gateflow the executable control plane for the **engineering lifecycle**
against pinned `workflow.yaml`:

```text
spec → tickets → implement (per wave) → closeout (per wave) → eng purge (initiative)
```

Two ticket APIs stay separate: **create tickets** (forge explicit) then
**implement-start** (ticket required). Gateflow applies pin board-status hops
(`in_progress` / `done`) with ticket from run context. Add
**`POST /api/v1/initiatives/closure/start`** for app purge only. Prove every
lane with Gateflow **verify scripts**.

Gateflow **does not** write product code, define the delivery process, merge
PRs, auto-apply `*-lgtm`, or orchestrate PM/meta purge.

### Success Criteria

| KPI | Target | Measurement |
|-----|--------|-------------|
| **Pin parse completeness** | 0 BROKEN tip nodes (`get_node` all ids in remounted `workflow.yaml`) | Unit against remounted pin |
| **Board status hops** | In Progress on implement enter; Done on closeout before `wave-signoff` | Verify scripts + board state |
| **Ticket gate** | Implement-start with missing/malformed ticket → **400**; unresolvable / mismatch / already Done → **422**; **0** enqueue | Unit + verify negative cases |
| **Create predicates** | Create fails closed unless `spec-pr-merged`, `implementation-plan-current`, `workmanifest-contract-pass` | Unit + verify |
| **Eng closure** | Closure-start after all `wave_ticket_ids` Done → EPIC Done → purge-app → Draft PR → stop `initiative-closure-signoff-app` | Closure verify script |
| **Lane verify suite** | Spec, tickets, implement, closeout, closure verify paths exit 0 under programme knobs | `tests/verify/*` Live-Verify records |
| **Contract hygiene** | 0 Forge merges; 0 auto `*-lgtm` | Code guard + verify audit |
| **Freeze** | Feature-readiness lists proven eng lanes vs deferred | Freeze doc in gateflow reports |

---

## 2. User Experience & Functionality

### User Personas

| Persona | Role | Primary need |
|---------|------|--------------|
| **PE** | Wave / initiative execution | Lane APIs that honor the pin; board columns match reality; verify scripts prove dogfood |
| **Tech lead** | Gate / freeze owner | Stop `purpose` visible; no invented merges/labels; clear proven vs deferred |
| **Programme** | Investment owner | Eng factory tip-parity without pulling PM/meta into this INIT |

### User Stories & Acceptance Criteria

#### US-1 — Spec lane non-regression

**As a** PE, **I want** `POST /api/v1/waves/spec/start` to keep walking
orchestrated spec Pass-1 to an honest human/manual stop **so that** tip remount
does not regress Spec delivery.

**Acceptance criteria:**

- [ ] Dual-bind `workspace` + `meta_workspace`; empty meta → fail closed, 0 enqueue (**400**)
- [ ] Automated `spec-pr-action` opens/updates Draft Spec PR without interactive authorize when requires complete
- [ ] Walker stops at `technical-review-approval` or first manual skill / human-checkpoint per pin — no merge, no `*-lgtm`
- [ ] Stop timeline includes pin `purpose` when next node is human-checkpoint
- [ ] Spec verify script exits 0 under programme knobs (G11)

#### US-2 — Create tickets after human Spec merge

**As a** PE, **I want** a create-tickets forge path separate from implement-start
**so that** the board tree exists before coding, and create fails if Spec is not
ready.

**Acceptance criteria:**

- [ ] Create uses pin `board-tickets-action` (`authorization: explicit`,
      `forge.action: create_board_tickets`, requires `initiative` + `plan_path`)
- [ ] Before apply succeeds, Gateflow hard-fails unless all three node predicates
      pass: `spec-pr-merged`, `implementation-plan-current`,
      `workmanifest-contract-pass` (pinned `workmanifest_contract.py`)
- [ ] Success returns `epic_ticket_id` and `wave_ticket_ids[]`
- [ ] Create does **not** enqueue implement/coding; no same-run resume into
      `pre-implement`
- [ ] Human `/create-board-tickets` remains a valid twin; same pin action
- [ ] Verify covers happy path + at least one predicate failure (0 tickets created)

#### US-3 — Implement-start requires a wave ticket

**As a** PE, **I want** implement-start to reject missing or mismatched tickets
**so that** coding always binds a real board wave card.

**Acceptance criteria:**

- [ ] `POST /api/v1/waves/implement/start` requires non-empty `ticket_id` (else **400**, 0 enqueue)
- [ ] Gateflow **board-resolves** ticket on the board (`org`/`repo`); missing → **422**, 0 enqueue
- [ ] Ticket must agree with request `initiative_id` + `wave_id` (existing dual-identity rules + board check); else **422**, 0 enqueue
- [ ] If ticket already board **Done** (`status: done`) → **422**, 0 enqueue
- [ ] If ticket already In Progress → idempotent: re-apply / accept In Progress (no error solely for already `in_progress`)
- [ ] On accept, **before** AgentRunner `pre-implement`: APPLY_FORGE pin
      `wave-in-progress-action` (`update_board_status`, `status: in_progress`)
      for that ticket
- [ ] Then walk `pre-implement` → `loop-spec` → automated `wave-pr-action` →
      STOP `live-verify` → `wave-awaiting-closeout`
- [ ] Implement verify proves In Progress side effect + Pass-1 stop

#### US-4 — Closeout reaches wave-signoff via Done

**As a** PE, **I want** closeout to move the wave ticket to Done then stop at
`wave-signoff` **so that** board state matches the pin before I merge.

**Acceptance criteria:**

- [ ] `POST /api/v1/waves/closeout/start` Enter-at fixed `learning-extract`
- [ ] After `ground-spec.pass`, APPLY_FORGE `wave-done-action`
      (`update_board_status`, `status: done`, ticket from run)
- [ ] Terminal stop at `wave-signoff` with `purpose: wave-signoff` on timeline
- [ ] No Forge merge
- [ ] Closeout verify exits 0 only if Done hop executed (not skipped)

#### US-5 — Eng initiative closure Enter-at

**As a** PE, **I want** `POST /api/v1/initiatives/closure/start` to purge app
working papers and open a closure Draft PR **so that** eng initiative closeout
is orchestrated tip-faithfully without meta purge.

**Acceptance criteria:**

- [ ] Route is exactly **`POST /api/v1/initiatives/closure/start`** (programme token)
- [ ] Required body fields: `initiative_id`, `org`, `repo`, `workspace_path` (app),
      `epic_ticket_id`, `wave_ticket_ids` (non-empty array), `runner`, `model_id`
- [ ] Missing/malformed required fields → **400**, 0 enqueue
- [ ] **Done-gate:** every `wave_ticket_ids` entry must be board **Done** (pin
      `status: done` / BoardService column+state contract); else **422**, 0 enqueue,
      no purge, EPIC not mutated
- [ ] **EPIC → Done:** after Done-gate passes, **before** dispatching
      `purge-initiative-artifacts-app`, apply board status Done to `epic_ticket_id`
      — **Gateflow programme board hygiene; not a pin `workflow.yaml` external-action node**
- [ ] Walker: `purge-initiative-artifacts-app` (`commit_workspace: required`) →
      automated `initiative-closure-pr-action-app` → STOP
      `initiative-closure-signoff-app`
- [ ] Does **not** dispatch `purge-initiative-artifacts-meta` or meta closure PR
- [ ] If EPIC → Done succeeds but purge-app or closure Draft PR fails: record
      failure; do **not** claim closure complete; PE may re-enter / compensate
      (**REQ-20**)
- [ ] Closure verify script exits 0 under programme knobs

#### US-6 — Programme chooses next wave vs closure

**As a** PE, **I want** Gateflow to stop at `wave-complete` decision semantics
without auto-chaining **so that** I start the next wave or closure deliberately
(**REQ-19**).

**Acceptance criteria:**

- [ ] After `wave-signoff`, Gateflow does not auto-start next wave or closure (**REQ-19**)
- [ ] Next wave = new implement-start with that wave’s `ticket_id`
- [ ] Initiative done = closure-start with epic + all wave ids Done

#### US-7 — Tip field fidelity and labels

**As a** tech lead, **I want** pin forge/checkpoint fields consumed and labels
kept sacred **so that** Gateflow does not fork `workflow.yaml`.

**Acceptance criteria:**

- [ ] Pin board-status hops parse and apply (`status: in_progress` \| `done`; ticket from run)
- [ ] Stop events expose pin `purpose` and `owner` when present
- [ ] Automated `apply_labels` only from pin lists; **never** labels ending in
      `-lgtm`
- [ ] No Forge merge action for `spec-merge` / signoffs

### Non-Goals

| Non-goal | Why |
|----------|-----|
| PM / requirements Enter-at | Parked |
| Meta purge orch (`purge-initiative-artifacts-meta`) | Later INIT |
| Same-run resume after create-tickets authorize | Create then implement-start |
| Implement-start creates board tree / picks first wave | Two-API model |
| Forge merge / `delete_branch` | Pin forbids merge |
| Auto `*-lgtm` | Human gates |
| Ops UI, second agent, Slack/Teams | Vision H2+ |
| Initiative C2 (probes, T13, parallel_safe) | Separate design |
| prayog-skills pin redesign | Consume tip only |
| Discover waves via `list_tickets(initiative_id)` alone | Label asymmetry (§8 outline) |

---

## 3. Functional requirements

### Capabilities

| ID | Capability | Covers |
|----|------------|--------|
| CAP-01 | Spec lane tip fidelity | REQ-01, REQ-10, REQ-17 (spec verify) |
| CAP-02 | Create board tickets | REQ-06, REQ-07, REQ-11 |
| CAP-03 | Implement-start + ticket gate | REQ-04, REQ-08 |
| CAP-04 | Closeout + wave-complete | REQ-05, REQ-19 |
| CAP-05 | Eng initiative closure | REQ-12, REQ-13, REQ-14, REQ-15, REQ-20 |
| CAP-06 | Pin / forge field fidelity | REQ-02, REQ-03, REQ-09, REQ-16, REQ-18 |

### Requirements

| ID | Requirement | Outline / gap | Condition | Observable result | Evidence |
|----|-------------|---------------|-----------|-------------------|----------|
| REQ-01 | Consume pin family `v0.5.0-rc.2`; harness `agent_skills.ref` equals submodule tip SHA/tag | Pin hygiene | Before W1+ verify | Pin load; `spec-draft` orchestrated; board-status nodes parse | unit + inspection |
| REQ-02 | Gateflow parses and applies pin board-status hops: `forge.status` (`in_progress`\|`done`) and ticket from run/handoff forge merge | G1 | `get_node` / APPLY_FORGE | All nodes in remounted `workflow.yaml` parse (0 BROKEN); no ValueError on board-status nodes | unit |
| REQ-03 | APPLY_FORGE board-status hop updates the board ticket to the pin status (column+state per BoardService contract) | G1 | Automated board-status hop | Board shows In Progress or Done for ticket | unit + verify |
| REQ-04 | Implement-start applies In Progress for run `ticket_id` before `pre-implement` dispatch; idempotent if already `in_progress` | G8-A1 | Accepted implement-start | Board shows In Progress before coding hop | verify |
| REQ-05 | Closeout applies Done after `ground-spec.pass` then stops at `wave-signoff` | G1 | Closeout walk | Timeline includes Done hop; terminal purpose `wave-signoff` | verify |
| REQ-06 | Create-tickets hard-fails unless `spec-pr-merged`, `implementation-plan-current`, `workmanifest-contract-pass` | G2 | Create authorize/apply | **422** (or authorize fail); 0 tickets on failure | unit + verify |
| REQ-07 | Create success returns `epic_ticket_id` + `wave_ticket_ids[]` | G9-A1 input | Create success | Response fields present | unit + verify |
| REQ-08 | Implement-start requires board-resolved `ticket_id` agreeing with initiative/wave; missing/malformed → **400**; unresolvable/mismatch/already Done → **422**; 0 enqueue | G5 | Bad/missing ticket | No job | unit + verify |
| REQ-09 | No Forge merge; `spec-merge` / signoffs are human | G6 | Any eng walk | No merge API/action | code guard + verify |
| REQ-10 | Resolve/stop events include pin `purpose` (+ `owner` when set) | G7 | Stop at human-checkpoint | Timeline/API fields set | unit |
| REQ-11 | Create does not resume into implement on same run | G8 | After create | Coding only via new implement-start | unit + verify |
| REQ-12 | `POST /api/v1/initiatives/closure/start` with required binds | G3–G4 | Valid token + body | 202 + `run_id` or **400**/**422** 0 enqueue | unit + verify |
| REQ-13 | Closure Done-gate: all `wave_ticket_ids` board Done (pin `status: done`) or reject **422** | G9-A1 | Closure start | Fail closed if any wave not Done; EPIC not mutated | unit + verify |
| REQ-14 | After Done-gate, set `epic_ticket_id` → Done **before** purge-app dispatch — programme board hygiene (**not** a pin external-action node) | Draft lock | Closure enter | EPIC Done on board before purge hop | verify |
| REQ-15 | Closure walk: purge-app → automated closure Draft PR → STOP `initiative-closure-signoff-app`; never purge-meta | G3 | Closure run | Timeline stops at signoff-app | verify |
| REQ-16 | Never auto-apply `*-lgtm`; only pin `apply_labels` | G10 | Any open_draft_pr | Label audit | unit + verify |
| REQ-17 | Verify suite covers spec, tickets, implement (w/ In Progress), closeout (w/ Done), closure | G11 | Programme exit | Live-Verify / verify exit 0 | live verify |
| REQ-18 | Feature-readiness freeze: proven vs deferred (PM Enter-at, meta purge, ops UI, C2, authorize→resume) | Exit | W4 | Freeze doc published | inspection |
| REQ-19 | After `wave-signoff` / `wave-complete`, Gateflow does not auto-start next wave or closure; PE starts next implement-start or closure-start deliberately | G8 / US-6 | After wave-signoff | No auto-chain | unit + verify |
| REQ-20 | If EPIC → Done succeeds but purge-app or closure Draft PR fails: record failure; do not claim closure complete; PE may re-enter / compensate | VF-09 | Partial failure after EPIC Done | Failure recorded; no success claim | unit + verify |

**Implementation notes (non-normative):** Gateflow may implement REQ-02/REQ-03 via
`ForgeActionType.update_board_status`, `NodeForgePolicy` status retention, and
`BoardService.update_ticket_status` — module names are design detail, not
product vocabulary. See §4.

### Assumptions

| ID | Assumption | Status | Dependent REQs |
|----|------------|--------|----------------|
| A1 | Board **Done** / **In Progress** vocabulary = pin `status: done` \| `in_progress` as applied by board-status hops / BoardService column+state contract | Confirmed | REQ-03, REQ-04, REQ-05, REQ-08, REQ-13, REQ-14 |
| A2 | PE retains `epic_ticket_id` + `wave_ticket_ids[]` from create-tickets for implement-start and closure-start | Confirmed | REQ-07, REQ-08, REQ-12, REQ-13 |
| A3 | Pin tip family `v0.5.0-rc.2` is frozen for this INIT delivery (harness ref == submodule tip) | Confirmed | REQ-01, REQ-02, REQ-18 |

### Error table (product-normative)

| Situation | HTTP / run | Side effects |
|-----------|------------|--------------|
| Implement-start missing/malformed `ticket_id` | **400** | 0 enqueue |
| Implement-start unresolvable ticket or ticket ≠ initiative/wave | **422** | 0 enqueue |
| Implement-start ticket already board Done | **422** | 0 enqueue |
| Implement-start ticket already In Progress | 202 / accept | Idempotent In Progress; proceed |
| Create predicates fail | **422** on authorize/apply | 0 board creates |
| Closure missing/malformed required fields or empty `wave_ticket_ids` | **400** | 0 enqueue |
| Closure any wave not Done | **422** | 0 purge; EPIC not mutated |
| Spec start empty meta workspace | **400** | 0 enqueue |
| Pin `update_board_status` missing ticket at apply | fail closed hop | No silent skip |
| EPIC → Done ok; purge-app or closure Draft PR fails | run failure recorded | EPIC may already be Done; **do not** claim closure complete (**REQ-20**) |

Exact problem+json / OpenAPI error body field names → **OQ-01** (deferred to
gateflow OpenAPI / spec PR).

### Open questions

| ID | Open question | Status |
|----|---------------|--------|
| OQ-01 | Exact problem+json / OpenAPI error body field names for Gateflow lane APIs | Open — freeze may leave TBD until OpenAPI pass |

---

## 4. Technical Specifications

### Architecture Overview *(design — not product vocabulary)*

```text
Programme token APIs
  ├─ POST /waves/spec/start
  ├─ Forge authorize → create_board_tickets   (explicit)
  ├─ POST /waves/implement/start  → In Progress → Pass-1 skills → wave PR → live-verify
  ├─ POST /waves/closeout/start   → learning → ground → Done → wave-signoff
  └─ POST /initiatives/closure/start
         → Done-gate(waves) → EPIC Done (programme hygiene) → purge-app → closure PR → signoff-app STOP

PolicyEngine ← workflow.yaml (dispatch, authorization, outcomes)
ForgeActionService ← open_draft_pr | create_board_tickets | update_board_status
BoardService.update_ticket_status
RunOrchestrator + AgentRunner + handoff baton
Verify scripts (tests/verify) = regression SSOT for lane claims
```

### Integration Points

| System | Use |
|--------|-----|
| Remounted `prayog-skills/workflow.yaml` | Navigation + forge policy |
| GitHub via ForgeClient | Draft PRs, board create/status, labels |
| PostgreSQL RunStore | Runs, timeline, learning ingest (existing) |
| Programme token | Lane start + forge authorize authn |

### Security & Privacy

- Programme service token on lane/forge/board routes (existing pattern).
- No secrets in handoff/verify artifacts.
- Content skills still must not treat local `gh` as automate success — ForgeClient only.
- Approval labels (`*-lgtm`) never applied by Gateflow.

### Board label asymmetry (normative constraint)

EPIC label: `gateflow/initiative:INIT-*`.  
Wave Feature label: `gateflow/initiative:INIT-*:W*`.  

Closure **must not** rely on `list_tickets(initiative_id)` alone to find waves.
Caller **must** pass `wave_ticket_ids[]` from create-tickets output.

### AI / agent evaluation

Not a new model product. Agent quality bar = pin skill outcomes + check/unit in
`loop-spec`; human live-verify scripts remain human-executed. Gateflow success =
contract/orchestration correctness measured by **verify scripts** and unit tests
(REQ-17), not LLM rubrics.

---

## 5. Risks & Roadmap

### Delivery waves (product-normative)

| Wave | Intent | Exit REQs |
|------|--------|-----------|
| **W0** | Parse board-status hops + status + ticket; purpose/owner; all remounted nodes `get_node`; unit green | REQ-01, REQ-02, REQ-10 |
| **W1** | Wire board status APPLY_FORGE; implement-start In Progress before pre-implement (idempotent) | REQ-03, REQ-04, REQ-11 |
| **W2** | Ticket gate (400/422); create predicates; verify tickets + implement | REQ-06–REQ-08, REQ-17 (partial) |
| **W3** | Closeout Done → wave-signoff verify; spec Pass-1 verify; wave-complete no auto-chain | REQ-05, REQ-09, REQ-16, REQ-17, REQ-19 |
| **W4** | Closure Enter-at + Done-gate + EPIC Done + purge walk + partial-fail (**REQ-20**); closure verify; freeze | REQ-12–REQ-15, REQ-17–REQ-18, REQ-20 |

### Technical risks

| Risk | Mitigation |
|------|------------|
| Tip retag mid-INIT | Freeze pin family; harness == submodule (A3) |
| Missing wave ids on closure | Require array; fail closed |
| Verify flakiness | Knobs + Live-Verify artifacts |
| Scope creep to meta/PM | Non-goals table; freeze deferred list |
| EPIC Done then purge/PR fail | REQ-20 recovery; no false closure complete |

### Phased rollout

- **MVP (this INIT):** Eng lane tip parity + verify suite + freeze.
- **Later:** Meta purge Enter-at; PM Enter-at; optional authorize→resume; ops UI.

---

## 6. Locked decisions reference (G1–G11)

| ID | Decision |
|----|----------|
| G1 | Full `update_board_status` parity; ticket from run |
| G2 | All three board-tickets node predicates hard-fail |
| G3 | Eng closure only through `initiative-closure-signoff-app` |
| G4 | `POST /api/v1/initiatives/closure/start` |
| G5 | Implement-start hard ticket + board resolve |
| G6 | No Forge merge; human Spec merge before create |
| G7 | Persist `purpose` / `owner`; no review_roles enforcement |
| G8 | No create→implement same-run resume; In Progress on implement enter |
| G9 | Closure requires epic + wave ids; all waves Done; **EPIC Done at enter** (before purge-app; programme hygiene, not a pin node) |
| G10 | No auto `*-lgtm` |
| G11 | Verify scripts prove all eng lanes |

---

## 7. Next steps

1. Impact map for gateflow (and any doc-only meta hygiene).
2. Meta Gate 1 → gateflow product INIT / waves W0–W4.
3. Do not implement from meta chat alone — follow SDD in gateflow.

---

## Appendix A — Target API surface

| Phase | API | Creates run? |
|-------|-----|--------------|
| Spec | `POST /api/v1/waves/spec/start` | Yes |
| Tickets | Forge authorize / `create_board_tickets` | Board side effect |
| Implement | `POST /api/v1/waves/implement/start` | Yes |
| Closeout | `POST /api/v1/waves/closeout/start` | Yes |
| Eng closure | `POST /api/v1/initiatives/closure/start` | Yes |
