# INIT-GATEFLOW-009 — Both-lane delivery factory prove-out

**Status:** outline · **Author:** programme PM · **Date:** 2026-08-03  
**Vision:** [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md)  
**Component:** GATEFLOW · **Type:** programme prove-out / confidence (not a greenfield product)

> **Outline only.** Written for programme readers in plain language. Engineering
> detail belongs in the impact map and gateflow prove-out work—not in a rebuild
> of capabilities we already have. Expand via the Draft PRD; **Gate 1** follows
> **`sdd-delivery/v2`** on the meta Draft PR (labels + Q&A + Approve)—not a
> separate ceremony.

---

## Document control

| Field | Value |
|-------|-------|
| Initiative ID | INIT-GATEFLOW-009 |
| Artifact | `prd/INIT-GATEFLOW-009-outline.md` (this outline) |
| Programme | prayog |
| Primary repo | drivestream-lab/gateflow |
| Supporting repos | prayog-meta (this PRD), prayog-skills (pin consume only), gateflow-ops (**out of scope**) |
| Skills pin | Stay on **`v0.5.0-rc.2` family** — remount/consume tip only; **do not** redesign the pin or cut a new RC for this INIT |
| Delivery contract | **`sdd-delivery/v2`** — adhere; do not invent parallel approval paths |
| Vision slice | Feature-readiness prove-out → unlock ops portal (Horizon 2) as a **later** initiative |
| Target readers | PE, tech lead, programme sponsor |
| Gate 1 posture | **`sdd-delivery/v2`:** meta Draft PR path — `impact-map-pending` → PR Q&A → `impact-map-lgtm` + Approve on exact head (`prd-impact-acceptance`). No PE-waive / alternate ceremony. |

---

## 1. Problem statement

Gateflow already runs a trustworthy **coding** wave: start the work, let allowed
steps run, open a draft pull request, stop for humans, then wrap up and store
what we learned.

It can also **start** a **spec** wave (from an approved programme PR and a
checked-out meta folder). Wrap-up is designed to work on **any** wave PR—not
only coding.

What the programme still lacks is **honest confidence**:

1. We have not yet **proven live** that a spec wave walks through to a real
   Draft Spec PR whose tip contains committed step artifacts, and stops where a
   person should decide.
2. We have not yet **proven live** that wrap-up works on a **spec** PR (only
   coding was proven; spec wrap-up was formally skipped).
3. We have not yet **proven live** that “ask a human before we touch something
   sensitive” (for example board seeding) works outside the lab via
   **stop → authorize API** (live required; no paper waiver as exit).
4. **Feature readiness** is not written down, so the ops portal would start
   without a shared proven vs deferred story.
5. Basic change safety on Gateflow PRs is still a **placeholder**, so we risk
   quietly breaking what we just proved.

Until those gaps close, investing in an ops console would decorate an unfinished
factory.

---

## 2. Proposed solution (summary)

**This initiative does not build a new Gateflow.**

It proves the factory we already have, writes **features proven vs deferred**,
and turns on basic PR checks—so the programme can start the ops portal next
without moving the goalposts. Delivery and approvals follow **`sdd-delivery/v2`**.

In plain terms we will:

1. Prove a **spec** wave under Gateflow (Draft Spec PR tip with committed
   artifacts + first honest stop).
2. Prove **wrap-up** on that Draft Spec PR (same wrap-up path coding already uses)—**required**.
3. Prove **authorize API** for sensitive forge **live only** (no paper waiver as exit).
4. Publish a **feature readiness** record with a deferral list (ops portal as next).
5. Replace placeholder CI with **real basic automated PR checks**.
6. Confirm we are running the **skills tip we intend** (rc.2 family)—no pin
   redesign.

Coding-lane automation, shared wrap-up, authorize APIs, run history, learning
store, and the current agent path stay as they are. We **reuse**; we do not
re-buy.

---

## 3. Positioning (why this INIT, not another build)

| Temptation | Why we refuse it here |
|------------|------------------------|
| Rebuild start / walker / draft-PR for coding | Already proven live |
| Invent a “spec-only wrap-up” product | One wrap-up path already serves both lanes |
| Bring a second coding agent live | Cursor path is enough for this prove-out |
| Add Slack / Teams alerts | GitHub comments are enough for this closeout |
| Start the ops portal now | Portal is the **next** initiative after freeze |
| Change the skills pin / new RC tag | Process stays on the rc.2 family by programme choice |
| Invent approval paths outside `sdd-delivery/v2` | Gate 1/2 stay on GitHub PR + labels + Approve |
| Programme-wide “Gateflow runs Gateflow” dogfood | Still deferred until feature readiness freeze |

Industry peers and long-term differentiation stay in
[vision §11–12](../planning/gateflow-programme-vision.md). This outline only
positions the **prove-out job**.

---

## 4. Users and jobs-to-be-done

| User | Job to be done |
|------|----------------|
| **PE** | “I want one trustworthy playbook for **spec** and **coding**: start → watch progress → decide at the stop → wrap up—without babysitting every step.” |
| **Tech lead** | “I want evidence humans still own sensitive steps, and that we did not silently rebuild the factory under a new label.” |
| **Programme sponsor** | “I want features actually proven—with deferred items named—before we fund an ops portal.” |

---

## 5. What this initiative must achieve

These seven achievements are the spine of success. The initiative is done only
when all are true (**A3 live only**—no waiver exit).

| ID | Achievement | Plain-language meaning | Known gap today |
|----|-------------|------------------------|-----------------|
| **A1** | Spec tip deliverable is live-proven | A recorded live run reaches a Draft Spec PR with committed step artifacts and stops at the first honest human/manual stop | Spec can start; tip deliverable not live-proven |
| **A2** | Spec wrap-up is live-proven | Same wrap-up on that Draft Spec PR through to wave sign-off; old “spec wrap-up skipped” note lifted | Wrap-up API exists; spec never proven |
| **A3** | Authorize API is live-proven | Stop → authorize API → forge side effect recorded live; **no paper waiver as exit** | Authorize path exists in the lab; live still missing |
| **A4** | Feature readiness freeze is published | One written freeze: features proven vs deferred; ops portal named as next | No feature readiness freeze record |
| **A5** | Basic change safety is on | Gateflow PRs run real basic automated checks (not a placeholder) | CI still placeholder |
| **A6** | Pin tip we intend is the tip we run | Remount/consume check for rc.2 family tip; no laptop overlay of process | Hygiene before prove-outs |
| **A7** | Programme records match reality | As-built / notes updated so they match prove-outs and the freeze | Scattered rows; some “deferred” notes may be stale |

**Hygiene under A7 (not a separate product):** if older notes still say coding
publish evidence is “deferred” while prove-outs already passed, correct the
record at freeze—do not invent a new capability wave.

---

## 6. Scope

### 6.1 In scope

| Theme | What we do |
|-------|------------|
| Spec confidence | Live prove-out of Draft Spec PR tip deliverable (A1) |
| Wrap-up parity | Live prove-out of wrap-up on a Draft Spec PR; lift the old skip note (A2) |
| Governed forge | Live prove-out of authorize API (A3)—no waiver exit |
| Programme freeze | Feature readiness write-up + deferral list (A4, A7) |
| Safety | Real basic automated PR checks on Gateflow PRs (A5) |
| Tip hygiene | Confirm intended skills tip before prove-outs (A6) |
| Delivery adherence | Follow **`sdd-delivery/v2`** for meta/app PR gates |

Most delivery is **witnessed runs**, **verify/playbook updates**, and
**records**—not new customer-facing product surfaces.

### 6.2 Out of scope

| Item | Why out |
|------|---------|
| Ops portal / dashboard UI | Next chapter after feature readiness freeze |
| Second coding agent going live | Not required for this prove-out |
| Slack / Teams alerts | Current progress comments are enough |
| Exhaustive “try every skill” bake-offs | One honest prove-out per gap, not a matrix |
| Auto-merge or “resume after authorize without a new start” | Humans keep merge / re-start ownership |
| Changing skills pin or cutting a new skills RC | Programme builds under rc.2 family |
| PE-waive / alternate Gate 1 ceremony | Approvals stay on Git PR + labels per `sdd-delivery/v2` |
| Programme-wide “Gateflow runs Gateflow” dogfood | Deferred until feature readiness freeze |
| Rebuilding coding start, walker, wrap-up, authorize, learning, or metrics | Already built—**reuse, don’t re-buy** |

### 6.3 Journeys we prove (story form)

**Spec tip deliverable**  
PE starts a spec wave → Gateflow runs the steps the process allows without a
human → Gateflow commits step outputs **per Gateflow process** → a Draft Spec
PR tip has real content → the run **stops** where a person must decide (or the
next step is still manual).

**Spec wrap-up**  
PE starts wrap-up on that Draft Spec PR → learning is captured when present →
the run **stops** at wave sign-off for the human. **Required** for this INIT.

**Authorize API**  
Sensitive forge step **stops** → PE calls authorize API → side effect appears
(e.g. board tickets). Independent of wrap-up order.

**Coding lane**  
Already proven. This initiative only **protects** it (checks + freeze).

---

## 7. Delivery waves (product outcomes)

| Wave | Business outcome | Achievements |
|------|------------------|--------------|
| **W0** | Tip confirmed; prove-out plan and fixtures ready for spec | A6 (+ setup for A1) |
| **W1** | Spec tip deliverable live prove-out **passed** | A1 |
| **W2** | Spec wrap-up live prove-out **passed**; old skip note lifted | A2 |
| **W3** | Authorize API live; **feature readiness** freeze written; basic automated PR checks on | A3, A4, A5, A7 |

No wave is “rebuild the orchestrator.”

---

## 8. Success criteria (outline)

| ID | We call the initiative done when… |
|----|-----------------------------------|
| S1 | A1 recorded with date and owner (Draft Spec PR tip + first honest stop) |
| S2 | A2 recorded; “spec wrap-up skipped” note lifted |
| S3 | A3 live proof on file (no waiver exit) |
| S4 | A4 feature readiness freeze published with deferral list naming ops portal as next |
| S5 | A5 basic automated PR checks gate Gateflow pull requests |
| S6 | A6 and A7 complete so tip and records match the prove-outs |

Numeric cycle-time targets are **not** exit gates for this prove-out.

---

## 9. Dependencies

| Dependency | Status | Need from it |
|------------|--------|--------------|
| Skills pin (rc.2 family, Spec Pass-1 tip) | Delivered | Consume only—no redesign |
| Coding-lane + coding wrap-up already proven | Delivered | Reuse as baseline |
| Spec start + meta intake already built | Delivered | Prove hops, don’t rebuild start |
| Shared wrap-up path | Delivered | Prove on a spec PR (**required**) |
| Human-gated authorize path | Built in lab | Prove live (no waiver exit) |
| Meta PR + meta folder for prove-out | Programme process | PE supplies fixtures before W1 |
| `sdd-delivery/v2` Gate 1/2 | Programme process | Meta/app PR labels + Approve |
| Ops portal | Not started | Starts **after** this INIT |

---

## 10. Risks and mitigations

| Risk | Mitigation |
|------|------------|
| Team treats this as a rebuild | Non-goals + “reuse baseline” section in Draft PRD |
| Spec prove-out blocked on missing meta PR/folders | W0 checklist before W1 |
| Paper waiver used as authorize exit | A3 / locked decisions: **live only**; no waiver exit |
| Records disagree with reality | A7 truth scrub at freeze |
| Scope creeps into ops UI | Hard non-goal; separate initiative after freeze |
| Parallel approval invents “ceremony waive” | Explicit **`sdd-delivery/v2`** adherence |

---

## 11. Decisions (proposed)

| # | Topic | Lean |
|---|-------|------|
| D1 | Initiative ID | `INIT-GATEFLOW-009` (title: both-lane factory prove-out / feature readiness) |
| D2 | Spec prove depth | Stop at **first honest human/manual stop** after Draft Spec PR tip has artifacts |
| D3 | Sensitive forge under Gateflow | **Stop → authorize API** (not IDE typing); **live proof required**; no paper waiver as exit |
| D4 | CI bar | **Basic automated PR checks** on PRs (e.g. branch naming / hygiene; no heavy live environment in CI) |
| D5 | Meta Gate 1 | Adhere to **`sdd-delivery/v2`** (meta PR + `impact-map-*` labels + Approve). No ceremony waiver. |
| D6 | Skills pin / RC | **Do not change**; remount/consume current rc.2 family tip only |
| D7 | Programme dogfood | Remains **deferred** until feature readiness freeze |
| D8 | Spec prove-out focus | **Draft Spec PR tip** with committed step artifacts; human reviewer confirms |
| D9 | Freeze messaging | **Feature** readiness list—not “Horizon 1.5 closed” as headline |
| D10 | Planning/vision update | **Same package** as feature readiness freeze |
| D11 | Delivery contract | Explicit adherence to **`sdd-delivery/v2`** |

---

## 12. Open questions

| # | Question | Status |
|---|----------|--------|
| Q1–Q3 | Authorize API live vs waiver; impact map; freeze packaging | **Resolved** in Draft PRD — see [INIT-GATEFLOW-009](./INIT-GATEFLOW-009.md) locked decisions |

---

## 13. What comes after (not this outline)

**Ops portal (Horizon 2)** — visibility over runs and waves—only after feature
readiness freeze (A4).

---

## 14. Outline → Draft PR boundary

| In this outline | In Draft PR | In gateflow delivery |
|-----------------|-------------|----------------------|
| Problem, achievements A1–A7, non-rebuild stance | Acceptance criteria; user stories + CAP/REQ ids | Prove-out scripts, dogfood runs, CI, as-built freeze |
| Waves W0–W3 as business outcomes | Wave exit checklists | Evidence links (dates, owners, PRs) |
| Decisions D1–D11 | Locked decisions (**no** authorize / Gate 1 waiver paths) | Impact map: gateflow-only; meta PR Gate 1 per `sdd-delivery/v2` |

---

## References

- Vision (horizons, dogfood deferred): [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md)
- Delivery contract: `sdd-delivery/v2` (`prayog-skills/delivery-contract.yaml`, `workflow.yaml`)
- Prior control-plane PRD: [INIT-GATEFLOW-001](./INIT-GATEFLOW-001.md)
- Skills pin / dispatch era: [INIT-PRAYOG-SKILLS-002](./INIT-PRAYOG-SKILLS-002.md)
- Gateflow as-built / prove-out evidence: `drivestream-lab/gateflow` → `docs/specification/as-built/implementation-status.md`
