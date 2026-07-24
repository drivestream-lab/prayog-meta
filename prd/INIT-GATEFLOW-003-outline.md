# INIT-GATEFLOW-003 — Live Cursor AgentRunner (outline)

**Status:** outline (synced with Draft PRD) · **Author:** programme PM · **Date:** 2026-07-24  
**Draft PRD:** [INIT-GATEFLOW-003](./INIT-GATEFLOW-003.md)  
**Vision:** [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md)  
**Predecessors:** [INIT-GATEFLOW-001](./INIT-GATEFLOW-001.md), [INIT-GATEFLOW-002](./INIT-GATEFLOW-002.md) (**finished / delivered**)  
**Component:** GATEFLOW · **Type:** platform / delivery control plane

> **Outline** — product intent locked with Draft PRD Discovery (2026-07-24).
> Engineering detail routes to impact map and gateflow spec PR. Dogfood
> programme work is deferred and does not drive this initiative.

---

## Document control

| Field | Value |
|-------|-------|
| Initiative ID | INIT-GATEFLOW-003 |
| Artifact | `prd/INIT-GATEFLOW-003-outline.md` |
| Programme | prayog |
| Primary repo | drivestream-lab/gateflow |
| Related (later) | drivestream-lab/gateflow-ops (consumer only; UI out of scope) |
| Supporting | prayog-meta, prayog-skills, launchpad (existing sync only — see §4) |
| Depends on | INIT-GATEFLOW-001 and INIT-GATEFLOW-002 **finished** (control plane, API wave start, per-skill runner/model config, PR thread from run start, fail-closed stubs for non-live slots, ForgeClient deploy path) |
| Skills pin | No pin-bump linkage required for exit; Scenario A may need pin `dispatch: orchestrated` edits |
| Target users | PE / eng (live Cursor on orchestrated skills), tech lead (audit live vs not-live), programme sponsor (cycle-time evidence) |

---

## 1. Problem statement

INIT-GATEFLOW-001 and INIT-GATEFLOW-002 are **finished**. Gateflow can start a
wave via API, honor the pinned workflow, stop at human checkpoints, open a PR
at run start, comment through stages, record metrics, and expose run / metrics /
board APIs — with ForgeClient for deployed GitHub writes.

What is still missing for a **credible coding / delivery stage**:

- The programme model treats the coding agent as a **plug-in behind a slot**.
  Process truth (SDD, harness, skills, MDC) comes from the pinned skills /
  harness — **not** from which agent brand runs.
- Cursor is the intended **first live** plug for orchestrated skills, but a
  **real Cursor SDK runner in the Gateflow worker** was left as follow-on after
  001/002.
- Automation must be proven on **exact existing skills** across **multiple
  scenarios** (pre–Gate 2 eng + coding cycle), with **defined cycle-time
  metrics** — not a single anonymous path.
- OpenCode, Claude Code, and other brands remain future plugs — this INIT does
  not make them live.

---

## 2. Proposed solution (summary)

**INIT-GATEFLOW-003** makes the **Cursor AgentRunner live** in Gateflow:

1. **`dispatch: orchestrated` ⇒ triggered** — Gateflow dispatches every
   resolved orchestrated skill and flows the pinned workflow + programme config
   (runner/model). **No** hardcoded node allowlist.
2. When config selects Cursor, the worker runs a **real Cursor coding agent**
   (harness / skills / MDC already applied as today).
3. Reuse 001/002 behaviour for wave start, contract stops, PR comments, and
   metrics — **do not** rebuild the control plane.
4. **Fail fast** if Cursor cannot authenticate, start, or finish — stop, record,
   notify; **no silent fake success**. Unsupported runners (OpenCode, Claude,
   …) also **fail fast** when selected.
5. **Defined cycle-time metrics** for live Cursor stages and waves (shipped as
   exit evidence; sponsor SLA thresholds later).
6. Keep other agent brands **not live** until a later INIT.

Gateflow still does **not** write application code itself (the agent does),
**does not** merge PRs, and **does not** redefine delivery process — prayog-skills
remains SSOT for workflow and skills.

---

## 3. Target experience (happy path)

```text
API / programme start (INIT-002 path)
        │
        ▼
Validate config (runner/model from gateflow repo config)
  → unsupported runner → fail fast
  → runner: cursor → require credentials → else fail fast
        │
        ▼
Resolve next node from pinned workflow
  → dispatch: orchestrated + runner: cursor → live Cursor AgentRunner
  → else stop / hand off per pin
        │
        ▼
Follow pinned outcomes; PR comments; cycle-time metrics
  → stop at contract / human checkpoint
        │
        ▼
Human verifies / merges
```

---

## 4. Launchpad — what it is and is not (explicit)

| Topic | Rule for this INIT |
|-------|--------------------|
| Who chooses the coding agent? | **Gateflow** programme config (runner slot) |
| Who owns process / skills / MDC / SDD? | **prayog-skills + harness pin** — agent-agnostic |
| What does Launchpad do today? | Factory / harness sync that Gateflow’s worker may **already call** |
| Does Launchpad pick Cursor for a repo? | **No** |
| Launchpad product changes in this INIT? | **No** |

---

## 5. Users and jobs-to-be-done

| User | Job to be done |
|------|----------------|
| **Eng / PE** | “Orchestrated skills run with live Cursor on Scenario A and B — not a stand-in.” |
| **Tech lead** | “Live Cursor is auditable; unsupported agents fail fast.” |
| **Programme sponsor** | “Stage and wave cycle-time metrics exist for Cursor so we can calibrate later.” |

---

## 6. Scope — in (features)

### 6.1 Live Cursor AgentRunner

- Cursor is a **live** coding-agent adapter behind the existing AgentRunner slot
- Runs in Gateflow’s **worker / programme-like** environment
- Same contract I/O spirit (workspace + skill prompt + model profile → outcome)

### 6.2 Orchestrated ⇒ triggered

- Every `dispatch: orchestrated` skill is triggered and flows per pin + config
- Runner/model from gateflow config (intended defaults **`cursor`** + **`auto`**)

### 6.3 Prove-it — two scenarios (exact existing skills)

| Scenario | Exact skills |
|----------|----------------|
| **A — Pre–Gate 2 eng** | `spec-draft` → `initiative-feasibility` → `spec-technical-review` → `spec-implementation-plan` |
| **B — Coding cycle** | `pre-implement` → `loop-spec` → `verify` → `ground-spec` |

After Gate 2 opens, Scenario A skills remain triggerable when orchestrated.

### 6.4 Fail-fast honesty

- Missing Cursor auth, start failure, crash → fail closed
- Unsupported / not-live runners selected in config → fail fast at start

### 6.5 Defined cycle-time metrics

- Stage `duration_ms` + wave cycle time for live Cursor runs
- Metrics API p50/p95 for `runner=cursor`

### 6.6 Leave other plugs untouched

- OpenCode / Claude Code remain **not live**
- No product-mandated Gateflow CI AgentRunner stub

---

## 7. Scope — out (explicit non-goals)

| Non-goal | Why |
|----------|-----|
| Launchpad product changes | Launchpad does not select agents |
| Cloud Cursor agents / cloud agent runtime | Explicitly out |
| Live OpenCode / Claude Code | Later INIT; fail-fast if selected |
| Live Slack / Teams notifiers | Unchanged from 002 |
| gateflow-ops UI | Still deferred |
| Rebuilding API / board / PR-at-start platform | Finished in 001/002 |
| Redefining SDD / skills / harness in Gateflow | SSOT remains prayog-skills |
| Dogfood programme as driver of this INIT | Explicitly deferred |
| Product-mandated Gateflow CI AgentRunner stub | Omitted |
| Pin-bump or calendar deadline linkage | None for this INIT |
| Sponsor SLA thresholds (beat manual N minutes) | Metrics defined; calibration later |

---

## 8. Relationship to finished predecessors

| Topic | After 001 + 002 | This INIT |
|-------|-----------------|-----------|
| Control plane + API waves | **Finished** | Reuses |
| Per-skill runner/model config | **Finished** | Cursor becomes a **real** selectable live runner |
| PR thread / metrics / board APIs | **Finished** | Reuses; cycle-time fields for live Cursor |
| Cursor | Slot + deferred live SDK claim | **Live Cursor AgentRunner** (FR-27) |
| Other agents | Stubs / not live | Still not live; fail-fast if selected |
| Launchpad | Sync consumer | Unchanged — no product work |

---

## 9. What a later INIT may cover

- Live OpenCode / Claude (or other AgentRunner plugs)
- Cloud agent runtimes (if the programme wants them)
- Stronger dogfood exit tied to a named programme calendar
- gateflow-ops UX on top of existing APIs
- Sponsor SLA calibration against cycle-time baselines

---

## 10. Success criteria (outline level)

| Outcome | How we know |
|---------|-------------|
| Orchestrated ⇒ triggered | Every `dispatch: orchestrated` skill flows per pin + config |
| Scenario A proven | Live Cursor on exact skills listed in §6.3 A |
| Scenario B proven | Live Cursor on exact skills listed in §6.3 B |
| Cycle-time defined | Stage + wave durations persisted; p50/p95 for `runner=cursor` |
| Failures are honest | Cursor auth/start/crash and unsupported runners → fail fast |
| Contract still honored | Human checkpoints never auto-passed; no auto-merge |
| No Launchpad product claim | No new Launchpad feature required for exit |
| Process SSOT unchanged | Skills / harness / MDC from pin — agent is a plug |

---

## 11. Open questions — resolved (2026-07-24)

| # | Question | Decision |
|---|----------|----------|
| 1 | New INIT vs patch 002? | **New INIT** — INIT-GATEFLOW-003 `(Source: User-confirmed)` |
| 2 | Cloud Cursor agents? | **Out** `(Source: User-confirmed)` |
| 3 | CI AgentRunner stub as product requirement? | **Omit** `(Source: User-confirmed)` |
| 4 | Dogfood as INIT driver? | **Deferred** `(Source: User-confirmed)` |
| 5 | Primary delivery repo? | **gateflow only** `(Source: User-confirmed)` |
| 6 | 001 / 002 status? | Both **finished / delivered** `(Source: User-confirmed)` |
| 7 | Launchpad role? | Does **not** choose agent; **no** product work `(Source: User-confirmed)` |
| 8 | Prove-it path? | **Two scenarios** with exact skills in §6.3; orchestrated ⇒ triggered; intended `cursor` + `auto` `(Source: User-confirmed)` |
| 9 | Dependencies / auth? | **Fail fast** `(Source: User-confirmed)` |
| 10 | Unsupported runners? | Config-driven; fail fast `(Source: User-confirmed)` |
| 11 | Cycle-time metrics? | **Defined and required** for exit `(Source: User-confirmed)` |
| 12 | Pin-bump / deadline linkage? | **None** `(Source: User-confirmed)` |

### Still open for PE / spec

1. Cursor auth / secret injection shape for the worker (product rule: fail-fast if absent).  
2. Exact RunStore field names for wave cycle time.  
3. Stand-in code path deleted vs unreachable when `runner=cursor`.  
4. Pin edit timing to mark Scenario A skills `orchestrated` if still `manual`.

---

## 12. Next steps (process)

| Step | Owner | Artifact |
|------|-------|----------|
| 1 | PM / sponsor | Review Draft PRD + this synced outline |
| 2 | PE | `/validate-requirements` on Draft PRD |
| 3 | PE | `/prd-impact-map` → **gateflow** primary |
| 4 | PE | Gateflow spec PR |
| 5 | PE | Delivery waves W0–W2 (B then A) |

---

## Appendix A — Feature checklist (discussion lock)

| # | Feature | In this INIT |
|---|---------|--------------|
| 1 | Live Cursor AgentRunner in Gateflow worker | Yes |
| 2 | Orchestrated ⇒ triggered (no node allowlist) | Yes |
| 3 | Prove-it Scenario A (exact pre–Gate 2 skills) | Yes |
| 4 | Prove-it Scenario B (exact coding-cycle skills incl. `verify`) | Yes |
| 5 | Fail-fast on Cursor / unsupported runners | Yes |
| 6 | Defined cycle-time metrics | Yes |
| 7 | Reuse 001/002 API / PR / metrics / board platform | Yes — consume, don’t rebuild |
| 8 | Launchpad product changes | No |
| 9 | Cloud Cursor agents | No |
| 10 | Live OpenCode / Claude | No |
| 11 | Dogfood as exit driver | No — deferred |
| 12 | Product-mandated Gateflow CI AgentRunner stub | No — omitted |

---

## Appendix B — One-sentence product

> With the control plane finished (001/002), Gateflow **triggers every
> `dispatch: orchestrated` skill** and flows delivery as pinned and configured —
> live Cursor is the first runner plug, proven on Scenario A and B existing
> skills, unsupported agents fail fast, cycle-time metrics are recorded, and
> Launchpad does not choose the agent.
