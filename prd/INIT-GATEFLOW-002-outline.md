# INIT-GATEFLOW-002 — Gateflow API-triggered waves & platform readiness (outline)

**Status:** outline (Draft PRD available) · **Author:** programme PM · **Date:** 2026-07-24  
**Vision:** [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md)  
**Predecessor:** [INIT-GATEFLOW-001](./INIT-GATEFLOW-001.md) (Horizon 1 control plane)  
**Draft PRD:** [INIT-GATEFLOW-002](./INIT-GATEFLOW-002.md)  
**Component:** GATEFLOW · **Type:** platform / delivery control plane

> **Outline** — product intent for review. **Draft PRD** expands acceptance
> criteria: [INIT-GATEFLOW-002.md](./INIT-GATEFLOW-002.md). Engineering detail
> routes to impact map and gateflow spec PR. Dogfood programme work is deferred
> and does not drive this initiative.

---

## Document control

| Field | Value |
|-------|-------|
| Initiative ID | INIT-GATEFLOW-002 |
| Artifact | `prd/INIT-GATEFLOW-002-outline.md` |
| Programme | prayog |
| Primary repo | drivestream-lab/gateflow |
| Related (later) | drivestream-lab/gateflow-ops (consumes APIs; UI not in this INIT) |
| Supporting | prayog-meta, prayog-skills, launchpad |
| Depends on | INIT-GATEFLOW-001 capabilities (run engine, contract stops, Cursor path, GitHub comments baseline) |
| Target users | PE (start waves, inspect runs), tech lead (audit), programme sponsor (metrics), future ops UI / next INIT (board APIs) |

---

## 1. Problem statement

INIT-GATEFLOW-001 establishes the delivery control plane: honor the pinned
workflow, run eligible automated skills, stop where humans own the next step,
and record what happened.

What is still missing for day-to-day programme use:

- Starting a wave is not yet a clear **product API** call — operators need an
  explicit, repeatable way to say “run this wave now”
- Coding agents and models are not yet **chosen per skill** (for example,
  `loop-spec` vs `verify`)
- Progress and history are hard to consume without a richer **run and metrics
  API** surface that a future ops console can sit on
- Board updates (In Progress / Ready / Blocked) should not be buried inside the
  wave engine — but the **APIs to perform board operations** must exist so the
  next initiative can wire that experience safely
- Creating the board (EPIC + wave tickets) today often relies on **developer
  laptop tools**. That is fine locally, but a **deployed** Gateflow cannot depend
  on those laptop-only tools — GitHub changes from Gateflow must use Gateflow’s
  own programme GitHub connection
- We need room to plug in more agents (OpenCode, Claude Code) and more
  notification channels (Slack, Teams) **without rewriting** the core — even if
  only GitHub comments and Cursor are live in this INIT

---

## 2. Proposed solution (summary)

**INIT-GATEFLOW-002** makes Gateflow **API-first for wave execution and platform
readiness**:

1. **Start a wave through an API** (authorized programme call) — not from board
   column changes
2. **Run the automated portion** of that wave under the same contract rules as
   INIT-001 (only skills marked for orchestration; stop at human / external
   checkpoints)
3. **Choose runner and model per automated skill** via programme configuration
4. **Open or update a pull request at run start** (same naming for success or
   failure) so progress comments have a durable thread through stages
5. **Notify via GitHub PR comments** (live); leave Slack and Teams as stubs for
   later
6. **Record metrics across stages** of the run so the programme can learn
7. **Expose run, metrics, and board-operation APIs** so gateflow-ops and the
   next INIT can build product flows without changing the wave engine
8. **Keep GitHub writes cloud-safe when Gateflow is deployed** — laptop board
   seeding may still use local developer tools; anything Gateflow does in
   production talks to GitHub through Gateflow’s own connection (not laptop CLI)

Gateflow still does **not** write application code itself, does **not** merge
PRs, and does **not** move board tickets as part of the wave workflow. It also
does **not** replace the board-seeding *skill* — that skill still decides *what*
to put on the board; Gateflow supplies a deployable *way* to apply GitHub
changes when the programme is not on a laptop.

---

## 3. Target experience (happy path)

```text
Board already seeded — wave tickets / wave information exist
        │
        ▼
PE (or tool) calls Gateflow API: “start wave N”
        │
        ▼
Validate config (block if stub runner/notifier required)
  → open/update PR at run start (same naming success|failure paths)
  → run orchestrated skills (per-node runner + model)
  → Notifier comments on that PR through stages
  → stop at contract stop; record metrics
        │
        ▼
Human verifies, clears blockers if needed,
opens or completes review, merges to develop
        │
        ▼
PE calls API again for the next wave — flow continues
```

**Board status changes** (for example In Progress → Ready / Blocked) are
**out of the wave path**. Separate **board APIs** exist so a later initiative
(or ops UI) can update tickets without mixing board logic into workflow
execution.

---

## 4. Board seeding vs Gateflow (laptop vs deployed)

This is a common point of confusion — and a hard requirement for production.

### What board seeding is

After the spec is merged, the programme runs a **board-seeding step** (the
`board-seed` skill). That step creates the initiative tree on the engineering
board — typically an EPIC and wave tickets such as W0, W1, and so on — using the
implementation plan. It is **manual** by design: a person authorizes seeding.
It does **not** write product code and it is **not** the same thing as starting
a wave run.

### Two places GitHub changes can happen

| Situation | What is fine |
|-----------|----------------|
| **Developer or PE on their laptop** runs board seeding | Using familiar local GitHub tooling is acceptable |
| **Gateflow running as a deployed service** (server / container / cloud) | Must **not** depend on laptop GitHub tooling. All GitHub create/update actions from Gateflow use **Gateflow’s programme GitHub connection** |

Same idea for opening PRs and posting run comments in this INIT: when Gateflow
does those actions in deployment, it uses that programme connection — not a
developer’s personal CLI session.

### What we are *not* saying

- We are **not** deleting or replacing the board-seeding skill.
- We are **not** asking the wave engine to seed the board automatically when a
  wave starts.
- We are **not** moving “what should be on the board?” judgment into Gateflow.

The skill (and the person running it) still own **what** to seed. Gateflow owns
a **deployable path** to perform GitHub board and forge actions when the
programme needs that path in production. Board **operation APIs** in this INIT
are the product surface for that path; a later skills update can teach board
seeding to call those APIs when apply must happen off-laptop.

### Simple rule

> **On a laptop, local GitHub tools are fine for board seeding.  
> In a Gateflow deployment, GitHub changes go through Gateflow’s own connection.**

---

## 5. Users and jobs-to-be-done

| User | Job to be done |
|------|----------------|
| **PE** | “I want to start wave N with one API call and trust Gateflow to run the automated skills, open a PR at run start, comment through stages, and tell me what happened.” |
| **PE (board setup)** | “On my laptop I can still seed the board with familiar tools; when Gateflow is deployed, board and PR actions must work without my laptop CLI.” |
| **Tech lead** | “I want an audit trail of which agent and model ran each skill, and proof we still stopped at human checkpoints.” |
| **Programme sponsor** | “I want stage-level metrics so we can improve skills and model choices with evidence.” |
| **Future ops / next INIT** | “I want stable APIs for runs, metrics, and board operations so we can build ticket lifecycle UX without reopening the engine.” |

---

## 6. Scope — in (features)

### 6.1 Start wave via API

- Authorized API to trigger a specific wave run
- Same spirit as INIT-001 / vision: **explicit human (or system) authorization**
  — Gateflow does **not** start work because a board column changed
- If preconditions fail, the API explains why; no silent start
- INIT-001 label trigger is **removed / superseded** for 002 programmes —
  **API is the only** supported start path (outline §11 Decision 1)

### 6.2 Automated wave execution (contract-aligned)

- Run only skills the pinned workflow marks as orchestrated
- Stop wherever the delivery contract assigns humans or external actions
- Does not auto-merge and does not set gate-approval labels

### 6.3 Runner and model per automated skill

- Programme configuration can set, for each orchestrated skill (for example
  `loop-spec`):
  - which **coding agent runner** to use
  - which **model / model profile** to attach
- Skills without an override use the programme default
- Applies only to orchestrated skills — does not change which skills are
  eligible to run

### 6.4 Coding agent adapters

| Adapter | This INIT |
|---------|-----------|
| **Cursor** | Implemented — live path |
| **OpenCode** | Stub — selectable later; clear “not implemented” if chosen now |
| **Claude Code** | Stub — same as OpenCode |

No silent fallback to Cursor when a stub runner is selected.

### 6.5 Pull request thread from run start

- At **run start**, Gateflow creates/updates a branch and **opens or updates a
  PR** using the **same naming conventions** for runs that later succeed or fail
- Purpose: durable place for progress and outcome comments through stages;
  evidence for human review
- **Human** still verifies and merges (Gateflow does not auto-merge)
- In deployment, opening the PR and posting comments use Gateflow’s programme
  GitHub connection (see §4) — not laptop-only tooling

### 6.6 Notifications

| Channel | This INIT |
|---------|-----------|
| **GitHub PR comments** | Implemented — live |
| **Slack** | Stub — integrate later |
| **Microsoft Teams** | Stub — integrate later |

Comments use structured run events (started, stage complete, stopped, failed) —
not ad hoc agent chat dumps.

### 6.7 Metrics across stages

Capture timing and outcomes across the run lifecycle, consistent with vision
programme learning intent, including for example:

- Wave run start (API trigger) through completion
- Each automated skill stage
- Stops at human checkpoints
- Findings / retry loops where they occur
- Observable handoff or gate signals where already available

Metrics are stored so they can be queried via API. Board-column observation as a
*trigger* remains deferred; events from the run remain available for later board
clients.

### 6.8 Ops-ready run and metrics APIs

Expand Gateflow’s HTTP surface so a future **gateflow-ops** console (or scripts)
can:

- List and filter runs
- Inspect a run’s full stage / event timeline
- Read metrics aggregates (by skill, runner, model)
- Rely on stable shapes and the same programme authentication model

This INIT delivers **API readiness**, not the ops UI itself.

### 6.9 Board operation APIs (ready; not used by the wave workflow)

Provide explicit APIs for board-related actions, for example:

- Update a ticket / card status or column (In Progress, Ready, Blocked, …)
- Link or attach a PR to a ticket
- Supporting actions needed so **off-laptop** board work can eventually apply
  through Gateflow (create or adjust tickets / project membership as the Draft
  PRD narrows) — without putting that logic inside the wave run

**Important boundaries:**

- The wave workflow **does not call** these APIs as part of start/finish
- These APIs are the deployable alternative to laptop-only GitHub tooling when
  the programme applies board changes from Gateflow (see §4)
- This INIT does **not** replace the board-seeding skill; a later skills update
  may call these APIs when seeding must run without a laptop

They exist so the **next INIT** (and ops tools) can own board lifecycle without
mixing it into orchestration.

---

## 7. Scope — out (explicit non-goals)

| Non-goal | Why |
|----------|-----|
| Board column change as the wave **trigger** | Deferred; API trigger keeps authorization explicit (001 / vision) |
| Label trigger (INIT-001 FR-2) | **Removed** for 002 programmes — API only |
| Wave engine auto-updating board tickets | Board lifecycle belongs to separate APIs + next INIT |
| Replacing the board-seeding skill with Gateflow | Skill still decides *what* to seed; Gateflow supplies deployable *apply* path |
| Requiring laptop GitHub tools inside a Gateflow deployment | Production must use Gateflow’s programme GitHub connection (§4) |
| gateflow-ops UI / pages | Next product surface; this INIT makes APIs ready |
| Working OpenCode / Claude Code dispatch | Stubs only |
| Working Slack / Teams delivery | Stubs only |
| Dogfood programme as a driver of this INIT | Explicitly deferred |
| Graphify / extra tool providers | Parked (vision) |
| Model gateway (e.g. LiteLLM) | Later horizon |
| Auto-merge PRs | Human accountability |
| Auto-set gate approval labels | Contract invariant |
| Redefining delivery process in Gateflow | SSOT remains prayog-skills workflow |

---

## 8. Relationship to INIT-GATEFLOW-001 and vision

| Topic | INIT-001 / vision | This INIT |
|-------|-------------------|-----------|
| Control plane basics | Webhooks, run store, contract stops, Cursor, GitHub comments | Builds on that foundation |
| How a wave starts | Label authorize (H1) | **API authorize only**; INIT-001 label **removed** for 002; board column trigger later |
| Runner / model | Single default profile | **Per orchestrated skill** |
| Multi-agent / multi-notify | Interfaces reserved | **Stubs** for OpenCode, Claude Code, Slack, Teams |
| Ops visibility | Thin status / metrics JSON | **Richer APIs** for future ops UI |
| Board | Read later; avoid mixing into engine | **Board APIs only**; no workflow-driven board moves |
| Board seeding | Manual skill; often laptop tooling | Skill stays; **deployed** apply uses Gateflow’s GitHub connection |
| PR | Open/update allowed as forge ops | **Open/update PR at run start**; same naming success\|failure; comments through stages |

---

## 9. What the next INIT is expected to cover

Not in this outline’s delivery, but this INIT must leave the door open:

- Product flow that uses **board APIs** after / around wave runs (e.g. reflect
  In Progress / Ready / Blocked from run outcomes)
- Optional board-based **trigger** (if the programme later chooses it)
- **gateflow-ops** UI consuming run, metrics, and board APIs
- Turning Slack / Teams / OpenCode / Claude stubs into live integrations
- Teaching the board-seeding skill (or ops tooling) to **call Gateflow board
  APIs** when seeding must apply without a laptop

---

## 10. Success criteria (outline level)

| Outcome | How we know |
|---------|-------------|
| API start works | Authorized call starts a wave when preconditions pass; clear error when they do not |
| Contract still honored | Automated skills only where allowed; human checkpoints never auto-passed |
| Per-skill control | At least two orchestrated skills can use different configured model profiles (Cursor path) |
| PR always present | Every started run opens/updates a PR at **run start**; stage + terminal comments on that PR |
| Stubs are honest | Choosing OpenCode, Claude Code, Slack, or Teams fails clearly — no silent substitute |
| API-ready | Run list/detail, metrics, and board-operation endpoints exist and are documented for the next INIT |
| Separation held | Completing a wave does not itself move board tickets |
| Deployed GitHub path | Gateflow production path for PRs, comments, and board APIs does not require laptop GitHub tooling |

Numeric cycle-time targets `[TBD — Draft PRD]`.

---

## 11. Open questions — resolved (2026-07-24)

| # | Question | Decision |
|---|----------|----------|
| 1 | Label vs API start | **API only**; INIT-001 label trigger **removed / superseded** for 002 programmes |
| 2 | Wave identity on API | **Both accepted:** issue/ticket id **and** initiative id + wave id (e.g. `W0`). |
| 3 | Failure PR naming | **Same conventions as success** (not a separate failure naming scheme). |
| 4 | Board API MVP | **Wider:** status/column update + link PR to ticket **plus** create/list tickets (seed-apply helpers for off-laptop seeding). |
| 5 | Per-skill runner/model config | **Stay in gateflow repo config** (same as INIT-001). |
| 6 | Board-seed apply transport | **Conditional dual path:** laptop / local developer context may use **local GitHub tooling (`gh`)**; Gateflow **deployed** path uses **ForgeClient** (programme GitHub connection). Board-seed skill stays; underlying apply switches by condition. Skills follow-on may wire the skill to call Gateflow board APIs when off-laptop. |
| 7 | Stub runner / notifier selected | **Block the whole run at configuration/start** — do not start and fail mid-run when a stub would be required. `(Source: User-confirmed 2026-07-24)` |
| 8 | Auto-link PR to wave ticket from workflow | **No auto-link from the wave workflow.** Linking uses the separate board API (caller invokes). Wave engine does not call board APIs on start/finish. `(Source: User-confirmed 2026-07-24)` |
| 9 | PR comment timeline | Open/update PR at **run start**; comments through stages (aligns Draft PRD Decision #9) |

### Discovery notes (for Draft PRD)

- **Why now:** API-first waves + ops/board API readiness after INIT-001; dogfood not a driver.
- **Stack constraint:** Same as INIT-001 — gateflow repo, programme token, ForgeClient, Postgres, gateflow programme config.
- **Success:** Outline §10 criteria remain the exit bar unless Draft PRD adds numeric targets later.


---

## 12. Next steps (process)

| Step | Owner | Artifact |
|------|-------|----------|
| 1 | PM / sponsor | Review this outline |
| 2 | PM + PE | Open questions §11 **resolved** (2026-07-24) — ready for Draft PRD |
| 3 | PE | Draft PRD — **done** → [INIT-GATEFLOW-002.md](./INIT-GATEFLOW-002.md) |
| 4 | PE | `/validate-requirements` on Draft |
| 5 | PE | Impact map (gateflow primary; board API consumers noted) |
| 6 | PE | Spec / delivery waves |

---

## Appendix A — Feature checklist (discussion lock)

| # | Feature | In this INIT |
|---|---------|--------------|
| 1 | API wave trigger | Yes |
| 2 | Contract-aligned automated wave execution | Yes |
| 3 | Per-orchestrated-skill runner + model | Yes |
| 4 | Cursor agent — live | Yes |
| 5 | OpenCode / Claude Code — stubs | Yes |
| 6 | PR at run start (success and failure naming) | Yes |
| 7 | GitHub PR comments — live | Yes |
| 8 | Slack / Teams notifier — stubs | Yes |
| 9 | Multi-stage metrics | Yes |
| 10 | Ops-ready run & metrics APIs | Yes |
| 11 | Board operation APIs (not called by workflow) | Yes |
| 12 | Deployed GitHub path (no laptop tooling required) | Yes — requirement |
| — | Board-driven trigger | No — later |
| — | Workflow auto board updates | No — next INIT may use board APIs |
| — | Replace board-seeding skill | No — skill stays; APIs enable off-laptop apply |
| — | gateflow-ops UI | No — later |
| — | Dogfood as initiative driver | No — deferred |

---

## Appendix B — One-sentence product

> After the board is seeded, an API call starts a wave; Gateflow opens a PR at
> run start, runs the automated skills with per-skill agent and model settings,
> comments through stages, records metrics, and exposes run, metrics, and board
> APIs — using Gateflow’s own GitHub connection when deployed — while humans
> merge and the next initiative owns board lifecycle UX.
