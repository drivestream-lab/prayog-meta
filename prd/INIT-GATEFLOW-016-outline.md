# INIT-GATEFLOW-016 — Gateflow Mission Control, greenfield API-grounded scope (outline)

**Status:** outline · **Author:** programme PM · **Date:** 2026-08-12  
**Draft PRD:** [INIT-GATEFLOW-016.md](./INIT-GATEFLOW-016.md) (drafted 2026-08-12; OQ-1–OQ-3 resolved there)  
**Vision:** [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md) (§9 maturity · §10 H2 · lift-on-metrics)  
**Component:** GATEFLOW-OPS · **Type:** operations / Mission Control  
**Predecessor:** INIT-GATEFLOW-004 (Gateflow Mission Control) — **retired 2026-08-12, deleted outright, superseded by this initiative**. 004 was drafted 2026-07-27, before INIT-GATEFLOW-013 (programme-first onboarding + real readiness) and INIT-GATEFLOW-015 (skill efficacy / factory effectiveness / delivery scorecard) shipped, and undersold what gateflow now exposes. Rather than patch a stale scope, this initiative re-derives the feature set bottom-up from gateflow's real, current route surface.  
**Related, not blocking:** none beyond the predecessor note above.

> **Outline — problem framing and scope lock.** Detailed requirements live in the
> Draft PRD. Every feature named below is grounded against a route that exists
> in `gateflow` today (verified via CBM `get_architecture`/`search_graph` against
> project `data-repos-prayog-gateflow`, cross-checked directly against
> `gateflow/src/api/v1/*.py` on disk this session) — nothing here depends on a
> capability that doesn't exist yet.
>
> **Hard rule for this initiative:** zero new gateflow backend work. If a
> feature needs an endpoint that doesn't exist, it is out of scope this
> initiative, named explicitly in §6, not quietly implied.

---

## Document control

| Field | Value |
|---|---|
| Initiative ID | INIT-GATEFLOW-016 |
| Artifact | `prd/INIT-GATEFLOW-016-outline.md` (this outline); Draft PRD at [`./INIT-GATEFLOW-016.md`](./INIT-GATEFLOW-016.md) |
| Programme | prayog |
| Primary repo | drivestream-lab/gateflow-ops |
| Supporting | drivestream-lab/gateflow (read-only API consumer this INIT — no gateflow contract or code change) |
| Explicitly **not** touched this INIT | Any gateflow backend change (the 4 named gaps in §6 — fleet-summary endpoint, workflow-pin readiness probe, process-map endpoint, log-pane narrative/deep-link richness — are deferred fast-follow); platform-admin console (`programme_admin_routes.py` / `agent_catalogue_router` surface); Launchpad greenfield scaffolding; `prayog-skills` pin/`dispatch` edits |
| Target users | Engineering / PE (onboard repos, operate waves, authorize forge actions); tech lead (efficacy signals, lift decisions); programme leadership (delivery scorecard, initiative tracking) |
| Depends on | INIT-GATEFLOW-001–003 (control plane + live Cursor — delivered); INIT-GATEFLOW-010/011 (initiative closure + readout composition — delivered); INIT-GATEFLOW-013 (programme-first onboarding + real readiness — delivered); INIT-GATEFLOW-014 (Runs API under JWT — delivered); INIT-GATEFLOW-015 (skill efficacy / factory effectiveness / delivery scorecard — delivered). This initiative builds a UI/BFF only; it introduces no new gateflow capability. |

---

## 1. Problem statement

`gateflow-ops` is still an empty chassis. Direct inspection of the repo
(`gateflow-ops/app/**`) confirms exactly two things are real: a sign-in flow,
and one "hello world" exemplar page (`SystemStatusPage`) that pings a
throwaway `/api/dev-echo` route. Its own code comment calls it "the living
exemplar — copy its shape for real features," not a feature. There is no
fleet home, no onboarding flow, no wave-start UI, no run cockpit, and no
efficacy panel.

Meanwhile `gateflow` has quietly accumulated a rich, tenant-scoped API
surface across five separate initiatives that nobody has built a UI for:

- **INIT-GATEFLOW-013** shipped programme-catalogue browsing, repo
  admission with a live PAT/Forge probe, and a harness/Launchpad readiness
  check — the exact machinery an onboarding scorecard needs.
- **INIT-GATEFLOW-010/011** shipped a full initiative/wave "readout"
  composition layer (`initiatives_routes.py`) — spec, implementation,
  closeout, merge, completion, and closure previews, each composed live from
  runs + board tickets. This has never been surfaced in any UI.
- **INIT-GATEFLOW-014** shipped the JWT-gated runs API.
- **INIT-GATEFLOW-015** shipped three efficacy/scorecard read APIs
  (skill-efficacy, factory-effectiveness, delivery-scorecard) purpose-built
  to remove the exact blocker the prior Mission Control attempt (004) was
  stuck on.

The prior attempt to scope this work, **INIT-GATEFLOW-004**, was drafted
2026-07-27 — before 013 and 015 shipped — and its scope assumptions are now
stale (it treated the efficacy dependency as `[TBD in spec]`, which 015 has
since resolved; it never anticipated the initiative/readout composition
layer at all). It was fully validated and impact-mapped but never reached
spec/build stage, so retiring it costs nothing in shipped code. This
initiative replaces it outright with a feature set re-derived bottom-up from
gateflow's actual, current route table.

---

## 2. Proposed solution (summary)

| What we're building | Solves |
|---|---|
| **CAP-A — Identity & access** | Sign-in, tenant visibility, and inviting a teammate — the console's front door |
| **CAP-B — Fleet onboarding** | No UI exists over 013's already-shipped catalogue/select/readiness endpoints |
| **CAP-C — Wave operations** | No UI exists to start a wave, list/inspect runs, or authorize a pending forge action — operators still use raw API calls |
| **CAP-D — Checkpoint evidence** | No UI exists over the live checkpoint status/history API |
| **CAP-E — Board & tickets** | No UI exists to see or link board tickets from the console |
| **CAP-F — Initiative & delivery tracking** | The richest unused surface in gateflow — full initiative/wave readout composition has zero UI today |
| **CAP-G — Metrics & efficacy panel** | 015 shipped exactly what 004 was blocked waiting on; still has zero UI |

**Unchanged on purpose:** no gateflow route, schema, or contract changes.
Every capability above is a pure `gateflow-ops` UI/BFF build against what's
live today.

---

## 3. Locked product decisions

| ID | Decision |
|---|---|
| **D1** | INIT-GATEFLOW-004 is retired outright (deleted, not superseded-in-place). This initiative's scope carries forward zero assumptions from it — every feature below is re-derived from gateflow's live route table |
| **D2** | **No gateflow backend work this initiative.** Build only against endpoints that exist today. The 4 named gaps in §6 (fleet-summary endpoint, workflow-pin readiness probe, process-map endpoint, log-pane richness) are deferred fast-follow, not blocking |
| **D3** | **Thin ops-user identity** — single capability tier, no roles/RBAC in v0. Matches gateflow's own route gating: every consumed endpoint this initiative touches is `require_role(TENANT_ADMIN)` |
| **D4** | Routes gated `PLATFORM_ADMIN` (`programme_admin_routes.py`, `agent_catalogue_router`) are **out of scope**. Role mismatch with the thin tenant-ops-user identity in D3; a platform-admin console is a distinct future persona/initiative |
| **D5** | Process stays **display-only** — `gateflow-ops` never authors delivery process; `prayog-skills`' pin remains SSOT. No in-console process editor |
| **D6** | **Initiative & delivery tracking (CAP-F) is in scope as a first-class feature area.** It was never part of 004's scope, is fully live and tenant-scoped today, and composes runs + board tickets into the richest read surface found in this initiative's research |
| **D7** | Onboarding (CAP-B) is built directly on INIT-GATEFLOW-013's existing catalogue/select/readiness-refresh endpoints. The console composes those 3 calls into a single pass/fail presentation client-side; no new composite gateflow endpoint is requested this initiative (consistent with D2) |

---

## 4. Users and jobs-to-be-done

| User | Job to be done |
|---|---|
| **Engineering / PE** | "I want to onboard a harnessed repo, start a wave, and authorize a pending forge action — without curl." |
| **Tech lead** | "I want to see skill efficacy and factory-effectiveness signals so I know what to automate next." |
| **Programme leadership** | "I want to see the delivery scorecard and track initiatives end-to-end (spec → implement → closeout → merge → closure) without stitching together the board and GitHub myself." |

---

## 5. Scope — in

| Area | What we build | Backing gateflow API(s) |
|---|---|---|
| **CAP-A** Identity & access | Sign in; view tenant list/detail; invite a teammate to the tenant | `POST /api/auth/login`; `GET /api/v1/tenants`, `GET /api/v1/tenants/{tenant_id}`; `POST /api/v1/tenants/{tenant_id}/users` |
| **CAP-B** Fleet onboarding | Connect/sync programme; browse catalogue candidates; admit a repo (PAT probe); check readiness (harness + Launchpad verdict); remove a repo | `PUT/GET .../programme/connect`, `.../connection`; `GET .../programme/catalogue`; `POST .../catalogue/refresh`; `POST .../repos/select`; `POST .../repos/readiness/refresh`; `POST .../repos/deselect` |
| **CAP-C** Wave operations | Start implement/spec/closeout wave; list runs (filterable); open run detail/timeline; authorize a pending forge action | `POST /api/v1/waves/{implement,spec,closeout}/start`; `GET /api/v1/runs`; `GET /api/v1/runs/{run_id}`; `POST /api/v1/runs/{run_id}/forge/authorize` |
| **CAP-D** Checkpoint evidence | View live checkpoint status (raw PR ref or composed initiative+wave ref); view checkpoint history | `GET /api/v1/checkpoints/status`; `GET /api/v1/checkpoints/history` |
| **CAP-E** Board & tickets | List board tickets; create EPIC/Feature ticket; update ticket status; link a PR to a ticket | `GET/POST /api/v1/board/tickets`; `PATCH .../tickets/{id}/status`; `POST .../tickets/{id}/links` |
| **CAP-F** Initiative & delivery tracking | List/detail initiatives; wave map; spec/implementation/closeout/merge/completion readouts; closure preview; start closure | `GET /api/v1/initiatives[/{id}]`; `GET .../waves`; `GET .../spec`; `GET .../waves/{wave_id}/{implementation,closeout,merge}`; `GET .../completion`; `GET .../closure`; `POST /api/v1/initiatives/closure/start` |
| **CAP-G** Metrics & efficacy panel | Stage-duration heatmap; skill/spec efficacy; factory effectiveness; delivery scorecard | `GET /api/v1/metrics/{runs,skill-efficacy,factory-effectiveness,delivery-scorecard}` |

---

## 6. Scope — out

| Out | Rationale |
|---|---|
| Fleet/wave-health summary endpoint | Named gateflow backend gap — no aggregate "latest run per onboarded repo" exists; deferred fast-follow (D2) |
| Workflow/skills-pin readiness probe | Named gateflow backend gap — onboarding scorecard category "workflow/skills pin resolvable" has no backing probe today; deferred fast-follow (D2) |
| Process-map (pinned workflow DAG) projection endpoint | Named gateflow backend gap — `WorkflowEngine` holds the node graph internally but nothing exposes it over HTTP; deferred fast-follow (D2) |
| Log-pane narrative + PR-comment/findings deep links | Named gateflow backend gap — run timeline has structured outcomes/durations but no readable narrative or comment-level links; deferred fast-follow (D2) |
| Platform-admin console (programme create/wipe, tenant-admin attach, lane-defaults, effective-runner, agent-catalogue) | Role mismatch (`PLATFORM_ADMIN`, not `TENANT_ADMIN`) — distinct future persona/initiative (D4) |
| Composite server-side onboarding-scorecard endpoint | Client-side composition of 3 existing calls is sufficient this initiative (D7); revisit only if drift/inconsistency proves it unreliable |
| Editing delivery process in console | Process stays display-only; `prayog-skills` pin remains SSOT (D5) |
| Launchpad greenfield / new-repo scaffolding | Different job; out of every Mission Control attempt, including this one |
| Auto-lift checkpoints / auto-merge | Visibility only — CAP-D/CAP-G surface signals; humans still own gates and merges |

---

## 7. Capability walkthroughs (summary)

| Capability | Done looks like |
|---|---|
| CAP-B | An operator picks a catalogue candidate, admits it, and sees a single pass/fail readiness verdict composed from the 3 underlying 013 calls — never a partial/ambiguous state |
| CAP-C | An operator starts an implement wave from the console, watches it appear in the run list, opens its timeline, and authorizes a pending forge action in one click when the run stops at an `external-action` gate |
| CAP-F | An operator opens an initiative and sees its full lifecycle — spec readout, per-wave implementation/closeout/merge readouts, and a completion/closure preview — composed entirely from existing gateflow data, no manual board-and-GitHub stitching |
| CAP-G | A tech lead compares skill-efficacy first-pass rate across two prompt revisions on the same `workflow_node`, and a programme lead reads the delivery scorecard's rework rate and factory coverage trailing-90-day delta |

---

## 8. Delivery waves (proposed)

| Wave | Intent |
|---|---|
| **W0** | CAP-A (identity chassis — largely already built) + CAP-B (fleet onboarding) |
| **W1** | CAP-C (fleet home, wave start/inspect, forge authorization) |
| **W2** | CAP-F (initiative & delivery tracking) — highest product differentiation, zero backend gap |
| **W3** | CAP-G (metrics & efficacy panel) — zero backend gap |
| **W4** | CAP-D (checkpoint evidence) + CAP-E (board & ticket linking) |

---

## 9. Success criteria (initiative exit)

1. An operator can sign in, onboard a harnessed repo through to a pass/fail readiness verdict, and see it on a fleet home — without a raw API call.
2. An operator can start a wave, open its run timeline, and authorize a pending forge action from the console.
3. An operator can open any initiative and see its full composed readout (spec → implementation → closeout → merge → completion/closure) sourced entirely from existing gateflow data.
4. A tech lead and a programme lead can each read their respective efficacy/scorecard views (CAP-G) end-to-end, tenant-scoped.
5. Zero gateflow route, schema, or contract changes ship as part of this initiative.

---

## 10. Dependencies and non-goals for partners

| Partner | Expectation |
|---|---|
| **gateflow** | Read-only consumer this initiative — no contract change. The 4 named gaps in §6 are logged as a future gateflow-side fast-follow, not committed here |
| **prayog-skills** | Not touched — pin/`dispatch` ownership unchanged |
| **launchpad** | Not touched — no greenfield scaffolding |
| **prayog-meta** | Hosts this outline, the eventual Draft PRD, and impact map; not an eng delivery target |

---

## 11. Risks

| Risk | Mitigation |
|---|---|
| Cockpit (CAP-C run detail) will feel thin on narrative for v0 — no readable log narrative or PR-comment deep links exist yet | Named, accepted risk this initiative; timeline/outcome/duration data is still fully real and useful; richer narrative is explicit fast-follow scope (§6) |
| Client-side onboarding-scorecard composition (D7) could drift from a true pass/fail-only presentation if not centralized carefully | Compose in one shared BFF module, not per-screen; revisit with a server-side composite endpoint if drift appears |
| CAP-F readouts depend on board/GitHub state accuracy that gateflow does not own | Named limitation, inherited from `InitiativeReadoutService`'s own scope — display what gateflow composes, no invented data |
| Building a large feature surface (7 capability areas) risks scope creep relative to a v0 | Wave sequencing (§8) ships CAP-A/B first (thin, already-shipped backend) before the larger CAP-F/G areas |

---

## 12. Open questions

| ID | Open question | Status |
|---|---|---|
| OQ-1 | Does the 003 W2 engg spec-lane dogfood role that lived in the now-deleted INIT-GATEFLOW-004 carry forward to this initiative, or is it dropped entirely? | Open — needs explicit programme decision, not assumed |
| OQ-2 | Should CAP-B's client-side scorecard composition (D7) eventually become a server-side gateflow endpoint? | Open — deferred until real usage shows drift (see Risks) |
| OQ-3 | Is the proposed W0–W4 wave grouping (§8) final, or should CAP-F/CAP-G be reordered relative to CAP-C given they have zero backend gap? | Open — confirm before Draft PRD |

`OQ-1`–`OQ-3` must be resolved in the Draft PRD. Do **not** build from this outline alone.

---

## 13. Next steps

1. Resolve OQ-1–OQ-3 in the Draft PRD.
2. Run `/validate-requirements` against the Draft PRD.
3. Impact map → programme sign-off → waves (from Draft PRD).
4. Do **not** build `gateflow-ops` from this outline alone — Draft PRD is normative.

---

## Exit gate (outline wording)

> **Exit:** `gateflow-ops` delivers a real Mission Control — onboarding,
> fleet operations, wave lifecycle with forge authorization, full initiative
> delivery tracking, and the efficacy/scorecard panel — entirely against
> gateflow's existing, live API surface. Zero gateflow route, schema, or
> contract changes ship this initiative; the 4 named backend gaps and the
> platform-admin console are explicit non-goals, tracked as future
> initiatives, not silently absorbed here.

---

## References

- Vision: [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md) §9 maturity · §10 H2 lift-on-metrics
- System identity: [planning/prayog-system-product-brief.md](../planning/prayog-system-product-brief.md)
- Predecessor (retired): INIT-GATEFLOW-004 — deleted 2026-08-12, superseded by this initiative
- Delivered dependencies: [prd/INIT-GATEFLOW-013.md](./INIT-GATEFLOW-013.md) (onboarding + readiness), [prd/INIT-GATEFLOW-014.md](./INIT-GATEFLOW-014.md) (Runs API under JWT), [prd/INIT-GATEFLOW-015.md](./INIT-GATEFLOW-015.md) (skill efficacy / factory effectiveness / delivery scorecard)
- Codebase grounding: `drivestream-lab/gateflow` — `src/api/v1/{metrics_routes,runs_routes,waves_routes,forge_routes,checkpoints_routes,board_routes,tenant_routes,catalogue_connection_routes,initiatives_routes}.py`; `drivestream-lab/gateflow-ops` — `app/(auth)/login/page.tsx`, `app/(dashboard)/page.tsx`, `lib/{auth,bff,upstream-fetch}.ts` (confirmed chassis-only, no product screens)
