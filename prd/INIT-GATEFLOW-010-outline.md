# INIT-GATEFLOW-010 — Engineering-lane pin tip executor parity (outline)

**Status:** outline · **Author:** programme PM · **Date:** 2026-08-05  
**Vision:** [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md)  
**Component:** GATEFLOW · **Type:** platform / delivery control plane  
**Predecessor freeze:** INIT-GATEFLOW-009 feature readiness (evolved — this INIT defines the target eng surface)

> **Outline only.** Locked product decisions from interactive pin-gap review
> (G1–G11). Draft PRD expands acceptance criteria and error tables.
> Engineering detail routes to impact map and gateflow spec PR.
>
> **SSOT:** remounted prayog-skills `workflow.yaml` + `delivery-contract.yaml`
> (`sdd-delivery/v2`, pin family **`v0.5.0-rc.2`**). Gateflow and prayog-meta
> both consume this pin — Gateflow must **execute** it for the engineering
> lifecycle, not only remount YAML.

---

## Document control

| Field | Value |
|-------|-------|
| Initiative ID | INIT-GATEFLOW-010 |
| Artifact | `prd/INIT-GATEFLOW-010-outline.md` (this outline); Draft PRD [INIT-GATEFLOW-010.md](./INIT-GATEFLOW-010.md) |
| Programme | prayog |
| Primary repo | drivestream-lab/gateflow |
| Supporting repos | prayog-meta (PRD), prayog-skills (pin consume-only — **no** pin redesign) |
| Skills pin | `agent_skills.ref: v0.5.0-rc.2` (harness SHA must equal submodule tip) |
| Target users | PE (lane execution), tech lead (gates / freeze) |
| As-built baseline | 2026-08-05 · gateflow `2791ab2e841e277cb53e0cb16faaa375da66c1f1` (Source: User-confirmed) |
| Product ids | CAP-01…06, REQ-01…20, OQ-01 — see Draft PRD |

---

## 1. Problem statement

Gateflow remounts prayog-skills tip **`v0.5.0-rc.2`** but does not fully
**execute** the engineering graph *(Source: User-confirmed; as-built baseline)*:

1. **Board-status hops** (`wave-in-progress-action`, `wave-done-action`) do not
   parse — walker **BLOCKs** on board→coding and Pass-2→signoff edges.
2. **Create tickets** and **implement-start** are not clearly separated as
   product APIs; implement-start must **board-resolve** the wave ticket and fail
   closed when the board issue is missing (non-empty `ticket_id` string alone is
   insufficient).
3. **`board-tickets-action`** node-level predicates (`spec-pr-merged`,
   `implementation-plan-current`, `workmanifest-contract-pass`) are not enforced
   as hard gates before create succeeds.
4. **Initiative eng purge** (`purge-initiative-artifacts-app` → automated closure
   Draft PR → `initiative-closure-signoff-app`) has **no Enter-at** API.
5. Checkpoint **`purpose` / `owner`** are dropped on resolve — weaker stop audit.
6. Regression must be proven via **Gateflow verify scripts** across all eng lanes.

PM / requirements Enter-at and meta purge (`purge-initiative-artifacts-meta`)
are **out of scope** for this INIT.

---

## 2. Proposed solution (summary)

Perfect Gateflow’s **engineering-lane** control surface against the pinned
workflow:

```text
spec (initiative)
  → tickets (initiative, once)
  → implement (per wave)
  → closeout (per wave)
  → eng purge / closure (initiative, once)
```

| Lane | Programme API | Pin adherence |
|------|---------------|---------------|
| Spec | `POST /api/v1/waves/spec/start` | Existing; non-regression |
| Tickets | Forge `create_board_tickets` (explicit authorize) | Predicates G2; **≠** implement-start |
| Implement | `POST /api/v1/waves/implement/start` | Ticket required; In Progress then Pass-1 |
| Closeout | `POST /api/v1/waves/closeout/start` | Through `wave-done-action` → `wave-signoff` |
| Eng closure | **`POST /api/v1/initiatives/closure/start`** (new) | purge-app → closure PR → stop signoff-app |

Gateflow **does not** invent merge Forge actions, auto `*-lgtm` labels, or
PM-lane orchestration. `workflow.yaml` remains navigation SSOT.

---

## 3. Locked product decisions (G1–G11)

| ID | Decision |
|----|----------|
| **G1** | Full pin parity for `update_board_status` (`status: in_progress` \| `done`); bind `ticket` from run context |
| **G2** | Enforce all three `board-tickets-action` node `requires` hard-fail before create succeeds |
| **G3** | Eng closure only — purge-app through STOP `initiative-closure-signoff-app`; **no** purge-meta |
| **G4** | New Enter-at `POST /api/v1/initiatives/closure/start` |
| **G5** | Implement-start: hard `ticket_id` + board resolve; must agree with initiative/wave |
| **G6** | No Forge merge; human merges Spec PR (labels); create-tickets only after merge |
| **G7** | Parse/persist human-checkpoint `purpose` (+ `owner` when present); no `review_roles` enforcement |
| **G8** | No same-run resume after create-tickets; coding via new implement-start. **A1:** apply In Progress as first automated forge step of implement-start before `pre-implement` |
| **G9** | Closure fail-closed unless all listed wave tickets Done; require `epic_ticket_id` + `wave_ticket_ids[]` (**A1**); set **EPIC → Done** on successful closure enter (programme hygiene, not a pin node); partial-fail **REQ-20** |
| **G10** | Pin `apply_labels` only; never auto-apply `*-lgtm` |
| **G11** | Regression = implement and prove **all** eng lanes via Gateflow **verify scripts** |

**Also locked:** create-tickets API ≠ implement-start; create response
(`epic_ticket_id`, `wave_ticket_ids`) supplies board binds — no separate GitHub
Project “board id” required for status hops.

---

## 4. Users and jobs-to-be-done

| User | Job to be done |
|------|----------------|
| **PE** | “After I merge the Spec PR, I create board tickets via API, then start each wave with a ticket id; Gateflow moves the card, codes, opens the Draft PR, and later closes out and purges app working papers — all proven by verify scripts.” |
| **Tech lead** | “Stops show pin `purpose`; board columns and tickets match the pin; freeze lists proven vs deferred.” |
| **Programme** | “Meta purge and PM Enter-at stay later; eng factory is tip-faithful now.” |

---

## 5. Scope — in

### 5.1 Pin-field consumption (engineering graph)

- Parse and APPLY_FORGE `update_board_status` with pin `forge.status` and
  `forge.requires: [ticket]`.
- Model `ticket` on handoff forge merge / effective policy.
- Retain `purpose` / `owner` on resolved nodes and stop timeline events.
- Enforce `board-tickets-action` **node-level** `requires` (not only
  `forge.requires`).
- `get_node` succeeds for **all** tip nodes (0 BROKEN).

### 5.2 Lane APIs

See §7. Spec / implement / closeout preserved and extended; closure Enter-at new.

### 5.3 Verify-script regression (G11)

Every eng lane has (or gains) a Gateflow `tests/verify/` path that dogfoods the
lane end-to-end under programme knobs. Initiative exit requires those scripts
green for the capabilities claimed in the freeze.

### 5.4 Capabilities map (gap → product ids)

Per `id-conventions.md`: `CAP-*` covers `REQ-*`. Full tables live in the Draft PRD.

| Gap | CAP | Primary REQs | Observable result |
|-----|-----|--------------|-------------------|
| G1 | CAP-06 | REQ-02, REQ-03 | Walker applies In Progress / Done; all remounted nodes parse |
| G2 | CAP-02 | REQ-06, REQ-07 | Create fails closed if spec not merged / plan not current / WM fail |
| G3–G4 | CAP-05 | REQ-12, REQ-15 | New closure start; purge-app → Draft PR → stop signoff-app |
| G5 | CAP-03 | REQ-08 | Implement-start board-resolves ticket; **400**/**422** fail closed |
| G6 | CAP-06 | REQ-09 | No merge Forge; human spec merge before create |
| G7 | CAP-01 / CAP-06 | REQ-10 | Stop events expose pin purpose |
| G8 | CAP-02 / CAP-03 | REQ-04, REQ-11 | Create then implement-start; In Progress on implement enter |
| G9 | CAP-05 | REQ-13, REQ-14, REQ-20 | Epic + wave ids; all waves Done; EPIC Done (programme hygiene); partial-fail recovery |
| G10 | CAP-06 | REQ-16 | No auto `*-lgtm` |
| G11 | CAP-01…05 | REQ-17 | Verify suite covers all eng lanes |
| US-6 | CAP-04 | REQ-19 | No auto-chain after `wave-complete` |

---

## 6. Scope — out

| Out | Rationale |
|-----|-----------|
| PM / requirements Enter-at (`validate-requirements` … Gate 1) | Parked — eng lane only |
| `purge-initiative-artifacts-meta` and meta closure PR/signoff orch | PM loop; later INIT |
| Same-run authorize→resume into implement | Create then implement-start (G8) |
| Implement-start auto-creates board tree / auto-picks first wave | Contradicts two-API model |
| Forge merge / `delete_branch` | Pin has no merge action |
| Auto `*-lgtm` / inventing approval labels | Human gates |
| Ops portal, second agent, Slack/Teams | Vision H2+ |
| Initiative C2 (evidence probes, T13, parallel_safe) | Separate cross-repo design |
| prayog-skills workflow / dispatch redesign | Consume tip only |
| Discovering wave tickets via `list_tickets(initiative_id)` alone | Label asymmetry — see §8 |

---

## 7. Engineering lane contracts (normative)

### 7.1 Spec lane (initiative, once)

| | |
|--|--|
| **Enter-at** | `POST /api/v1/waves/spec/start` |
| **Pin path** | `spec-draft` → automated `spec-pr-action` → feas → TDD → STOP `technical-review-approval` → human plan / `coding-readiness` / human **`spec-merge`** |
| **Binds** | `workspace` + `meta_workspace` (fail closed if meta empty on spec start) |
| **010 duty** | Non-regression; verify script Pass-1 |
| **Human** | Spec merge + labels (G6) before tickets |

### 7.2 Tickets lane (initiative, once)

| | |
|--|--|
| **API** | Forge authorize / apply `create_board_tickets` (`authorization: explicit`) |
| **Human twin** | `/create-board-tickets` |
| **Pin** | `board-tickets-action` → (graph) `wave-in-progress-action` |
| **`forge.requires`** | `initiative`, `plan_path` |
| **Node `requires` (hard)** | `spec-pr-merged`, `implementation-plan-current`, `workmanifest-contract-pass` |
| **Returns** | `epic_ticket_id`, `wave_ticket_ids[]` |
| **010 duty** | Enforce predicates; **does not** start coding; no same-run resume into implement |
| **Verify** | Create path + predicate failure cases |

### 7.3 Implement lane (per wave)

| | |
|--|--|
| **Enter-at** | `POST /api/v1/waves/implement/start` |
| **Precondition** | `ticket_id` required; board resolve; agree with `initiative_id` + `wave_id`; missing/malformed → **400**; unresolvable/mismatch/already Done → **422**; 0 enqueue. Already In Progress → idempotent |
| **First forge (G8-A1)** | Apply pin `wave-in-progress-action`: board-status / `in_progress` for run ticket |
| **Then** | `pre-implement` → `loop-spec` → automated `wave-pr-action` → STOP `live-verify` → `wave-awaiting-closeout` |
| **Verify** | Pass-1 including In Progress side effect |

### 7.4 Closeout lane (per wave)

| | |
|--|--|
| **Enter-at** | `POST /api/v1/waves/closeout/start` (fixed `learning-extract`) |
| **Pin path** | `learning-extract` → `ground-spec` → `wave-done-action` (`done`) → STOP `wave-signoff` |
| **010 duty** | Done hop must execute (G1); learning ingest retained; no Forge merge |
| **Verify** | Reach `wave-signoff` **via** Done hop |

### 7.5 Eng purge / closure lane (initiative, once)

| | |
|--|--|
| **Enter-at** | `POST /api/v1/initiatives/closure/start` |
| **Required binds** | `initiative_id`, `org`, `repo`, `workspace_path` (app), `epic_ticket_id`, `wave_ticket_ids[]`, runner/model |
| **Precondition** | Every id in `wave_ticket_ids` is **Done** (pin `status: done`); else **422** fail closed. Then set **EPIC → Done** (Gateflow programme board hygiene — **not** a pin external-action node). Handoff alone is insufficient. |
| **Pin walk** | After human initiative-closure intent: `purge-initiative-artifacts-app` → automated `initiative-closure-pr-action-app` → STOP `initiative-closure-signoff-app` |
| **Partial fail (REQ-20)** | If EPIC → Done ok but purge-app / closure PR fails: record failure; do not claim closure complete |
| **Not in 010** | Continue to `purge-initiative-artifacts-meta` |
| **Verify** | New/extended verify script for eng closure |

### 7.6 After wave-signoff (programme choice)

`wave-complete` is a pin **decision** (**REQ-19** / CAP-04). Gateflow does not
auto-pick. Programme calls either next **implement-start** (another wave ticket)
or **closure-start** (when all waves Done and PE closes initiative).

---

## 8. Code constraint — board label asymmetry

`create_board_tickets` today labels:

| Ticket | `initiative_id` passed to board create | Label |
|--------|----------------------------------------|-------|
| EPIC | `INIT-FOO` | `gateflow/initiative:INIT-FOO` |
| Wave Feature | `INIT-FOO:W1` | `gateflow/initiative:INIT-FOO:W1` |

Therefore `list_tickets(initiative_id=INIT-FOO)` returns the **EPIC**, not wave
Features. Closure Done-gate **must** use explicit **`wave_ticket_ids[]`** from
create output (G9-A1). Do not claim discovery-by-initiative-label alone in 010.

---

## 9. Delivery waves (product-normative)

| Wave | Intent | Exit REQs |
|------|--------|-----------|
| **W0** | Tip contract: board-status parse + `status` + `ticket`; all remounted nodes `get_node`; purpose/owner on resolve; unit green | REQ-01, REQ-02, REQ-10 |
| **W1** | Wire APPLY_FORGE board-status for In Progress + Done; implement-start In Progress before `pre-implement` | REQ-03, REQ-04, REQ-11 |
| **W2** | Ticket-required implement-start (board resolve; 400/422); board-tickets predicates hard-fail; verify tickets + implement | REQ-06–REQ-08, REQ-17 (partial) |
| **W3** | Closeout verify through Done → `wave-signoff`; spec Pass-1; no auto-chain (**REQ-19**) | REQ-05, REQ-09, REQ-16, REQ-17, REQ-19 |
| **W4** | Closure Enter-at + Done-gate + EPIC Done + purge-app + **REQ-20** partial-fail; closure verify; freeze | REQ-12–REQ-15, REQ-17–REQ-18, REQ-20 |

Plan may refine TASK boundaries; product exits above are normative.

---

## 10. Success criteria (programme exit)

1. **0 BROKEN** tip nodes — `WorkflowEngine.get_node` for all pin nodes.
2. Spec verify Pass-1 green (non-regression).
3. Create-tickets: predicates enforced; returns epic + wave ticket ids.
4. Implement-start: board-resolves ticket (**400**/**422**); applies In Progress (idempotent); Pass-1 verify green.
5. Closeout verify reaches `wave-signoff` via Done hop; no auto-chain after wave-complete (**REQ-19**).
6. Closure-start: rejects unless all `wave_ticket_ids` Done; sets EPIC Done (programme hygiene); purge-app → automated Draft PR → stop `initiative-closure-signoff-app`; partial-fail per **REQ-20**.
7. No Forge merge; no auto `*-lgtm`.
8. Feature-readiness freeze lists proven eng lanes vs deferred (PM Enter-at, meta purge, ops UI, C2, authorize→resume).

---

## 11. Dependencies and non-goals for partners

| Partner | Expectation |
|---------|-------------|
| **prayog-skills** | Remount tip only; no workflow redesign in this INIT |
| **prayog-meta** | This outline → Draft PRD → Gate 1 as usual |
| **Launchpad** | Materialize existing skills; no new human `/update-board-status` |
| **Humans** | Spec merge, live-verify scripts, wave-signoff merge, closure signoff merge |

---

## 12. Risks

| Risk | Mitigation |
|------|------------|
| Tip retag mid-INIT | Freeze pin family; harness ref == submodule SHA |
| Wave list omitted on closure | Require `wave_ticket_ids`; fail closed |
| Label asymmetry misunderstood | §8 normative; A1 binds only |
| Verify flakiness | Programme knobs + recorded Live-Verify artifacts |
| Scope creep to meta purge / PM | Explicit out-of-scope §6 |

---

## 13. Draft PRD locks (resolved)

| Item | Lock |
|------|------|
| EPIC → Done timing | At closure **enter** — after wave Done-gate, **before** purge-app; programme board hygiene (**not** a pin external-action node) |
| Closure Enter-at path | `POST /api/v1/initiatives/closure/start` |
| problem+json field names | **OQ-01** — deferred to gateflow OpenAPI / spec PR |
| HTTP fail-closed | **400** missing/malformed; **422** resolve / Done-gate / already Done |

Draft PRD: [INIT-GATEFLOW-010.md](./INIT-GATEFLOW-010.md).

---

## 14. Next steps

1. Review Draft PRD with PE / programme.
2. Impact map → meta Gate 1 → gateflow spec / implement waves per §9.
3. Do **not** implement Gateflow code from meta alone — follow SDD.

---

## Appendix A — Pin eng path (reference)

```text
spec-draft → spec-pr-action → feas → TDD → technical-review-approval
  → plan → coding-readiness → spec-merge
  → board-tickets-action → wave-in-progress-action
  → pre-implement → loop-spec → wave-pr-action → live-verify → wave-awaiting-closeout
  → [Enter-at] learning-extract → ground-spec → wave-done-action → wave-signoff
  → wave-complete (decision)
  → initiative-closure → purge-app → initiative-closure-pr-action-app
  → initiative-closure-signoff-app
  → [OUT OF SCOPE 010] purge-meta → …
```

## Appendix B — API surface (target)

| Phase | API | Creates run? |
|-------|-----|--------------|
| Spec | `POST /waves/spec/start` | Yes |
| Tickets | Forge authorize / `create_board_tickets` | Board side effect |
| Implement | `POST /waves/implement/start` | Yes |
| Closeout | `POST /waves/closeout/start` | Yes |
| Eng closure | `POST /initiatives/closure/start` | Yes |
