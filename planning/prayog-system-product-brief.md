# Prayog system product brief

**Programme:** prayog · **Org:** drivestream-lab · **Status:** draft  
**Last updated:** 2026-08-09  
**Audience:** programme PM / PE — product identity across repos  
**Related:** [gateflow-programme-vision.md](./gateflow-programme-vision.md) (runtime
slice only) · `config/programme.yaml` · `config/service-catalog-drivestream-lab.yaml`

> **Greenfield brief.** This document names the **system product**, not a single
> repo. Gateflow is one plane. Do not treat the runtime vision as the whole
> product story.

---

## 1. One-sentence product

> Prayog is a **governed AI-assisted software delivery factory**: it turns
> product intent into multi-step, human-gated engineering outcomes using coding
> agents as workers — with durable process, forge-safe side effects, and
> evidence — not as the process owner.

---

## 2. Problem

Coding agents are cheap. Delivery discipline is not.

Without a factory, every team invents Cursor rituals; state lives in chat;
humans manually chain steps; PRs/boards/merges happen ad hoc; there is little
run history or learning. The bottleneck is no longer “can an agent write code?”
It is:

> **How does a software company run a reliable multi-step delivery process
> around agents — with stops, forge authority, retries, and accountability?**

That pain is **internal today** and **category-wide** for any eng org adopting
agents at scale.

---

## 3. Primary user and job-to-be-done

| Role | Job |
|------|-----|
| **Programme / eng leader** | Scale agent-assisted delivery without losing SDD quality or human accountability |
| **PE / operator** | Authorize lanes and own gates — not manually dispatch every skill hop |
| **Engineer** | Implement inside a known contract (skills, harness, constitutions) |
| **PM** | Drive intent and acceptance in meta — not invent runtime mechanics |

**Primary JTBD (system):**  
When we need to deliver software with AI coding agents, help us execute our
delivery method end-to-end so agents do the work, humans keep the high-value
gates, and every wave leaves durable evidence.

**Anti-JTBD:**  
Replace engineers, auto-merge by default, or become “a better Cursor.”

---

## 4. Product planes (multi-repo fitment)

Prayog is assembled from planes. Each plane has a home repo. Roadmap items must
land in the right plane.

```text
Intent & governance     prayog-meta
Process & procedure     prayog-skills
Factory install         launchpad · *-foundation · *-rules
Runtime control plane   gateflow
Ops surface             gateflow-ops
Grounding (optional)    prayog-repo-fleet
```

| Plane | Owns | Does not own |
|-------|------|--------------|
| **meta** | Vision, PRDs, programme config, service catalog | Skill procedures, agent dispatch |
| **skills** | `workflow.yaml`, delivery contract, skill packs, handoff semantics | Installing harness, running workers |
| **launchpad + foundations/rules** | Onboard, harness sync, scaffolds, coding constitutions | Wave orchestration policy |
| **gateflow** | Lane start, pin walk, agent dispatch, ForgeClient, run/metrics/learning store | Process SSOT, ops UX, catalogue authorship |
| **gateflow-ops** | Operate/observe runs and delivery status | Orchestration authority |
| **repo-fleet** | Cross-repo code graphs / MCP evidence | Delivery navigation |

**Routing rule for “should we build X?”**

1. Process / navigation / skill procedure → **skills**  
2. Onboard / install / harness fidelity → **launchpad** (+ meta catalog)  
3. Execute / dispatch / forge / runs → **gateflow**  
4. See / operate → **gateflow-ops**  
5. Cross-repo structural evidence → **fleet**  
6. Spans planes → name it as **Prayog system** work, not a Gateflow feature

---

## 5. Market fitment (which layer we are)

| Archetype | Role | Prayog stance |
|-----------|------|---------------|
| Spec Kit–class | Structured AI development methodology | **Core** — lives in **skills** (+ meta intent) |
| Symphony–class | Coding-agent dispatch over work items | **Runtime slice** — **gateflow** |
| Conductor–class | Generic durable business workflows | **Out of scope** |
| Optio-style multi-agent router | Free-form agent decomposition | **Out of scope** as identity (pin walks a fixed graph) |
| Factory kit | Stamp methodology onto repos | **Core** — **launchpad** + foundations/rules |
| Mission Control | Ops visibility | **gateflow-ops** (early) |

**Closest honest label:**  
SDD delivery factory = methodology + install + governed runtime + ops.

**Not:** glorified coding agent · generic enterprise orchestrator · autonomous
ticket-to-merge autopilot.

---

## 6. Who we solve for now vs later

| Lens | Verdict |
|------|---------|
| **Our software company** | Primary customer. Strong fit. Dogfood is the proving ground. |
| **Wider eng orgs** | Same *category* pain. Not yet a packaged product — five repos + Prayog dialect are an assembly, not a SKU. |

**Decision (draft):**  
Ship **Purpose A** first — internal delivery OS. Treat **Purpose B**
(productize the control-plane + portable contract pack) as expansion thesis,
not current identity.

| Purpose | Meaning | When |
|---------|---------|------|
| **A — Internal delivery OS** | Run *our* method with agents, safely and repeatedly | Now |
| **B — Productized factory** | Other orgs bring/adapt a contract pack + install + runtime | After A is stable and portable |
| **C — Autonomous eng manager** | Tickets in → merge out with minimal humans | Explicit non-goal unless programme opts in much later |

---

## 7. Non-goals

- Replacing Cursor / OpenCode / Claude Code (they are **runner plugs**)
- Owning process inside the runtime (pin/skills remain SSOT)
- Generic non-software workflow orchestration
- Auto-merge or worker-driven board column moves as default
- Inventing PE gate labels or bypassing `human-checkpoint` / `authorization: explicit`
- Selling Gateflow alone as “the product”

---

## 8. Build vs buy vs package

| Capability | Stance | Notes |
|------------|--------|-------|
| Coding agents / models | **Buy / plug** | Cursor today; more runners later |
| Delivery methodology (SDD contract + skills) | **Build** | Moat and differentiation |
| Factory install (harness, scaffolds) | **Build** | Makes methodology repeatable |
| Runtime orchestrator | **Build** | Thin consumer of pin; not a second process author |
| Forge transport | **Build adapter** on GitHub APIs | No `gh` CLI in production path |
| Ops console | **Build** (thin BFF) | Consumes runtime APIs |
| Code graphs / MCP | **Build/integrate** | Amplifier; not identity |
| Durable generic workflow engine (Temporal/Conductor) | **Defer / buy later** | Only if fleet scale outgrows Postgres jobs |
| Horizontal SKU packaging | **Package later** | Portable contract + install story required first |

---

## 9. Success signals (system, not single repo)

**Internal OS is working when:**

- A PE-authorized lane runs multi-step hops without mid-chain manual skill invoke
- Humans still own checkpoints and explicit forge seats
- Waves leave durable runs, metrics, and learning — not only chat
- New repos get the same factory via launchpad (not bespoke rituals)
- Process changes ship in **skills**; runtime picks them up from the pin

**Productization is plausible when:**

- A non-Prayog programme can adopt a contract pack + install + runtime without
  forking our entire org dialect
- Ops UI and multi-runner are good enough for an operator outside the founding team

---

## 10. Working vocabulary

| Say | Instead of |
|-----|------------|
| Prayog (the delivery factory) | “Gateflow the product” |
| Planes / repos | One monolith narrative |
| Contract consumer (runtime) | Process author in the orchestrator |
| Governed automation | Autopilot / autonomous eng org |
| Runner plug | “Our coding agent” |

---

## 11. Open decisions

| # | Decision | Lean |
|---|----------|------|
| 1 | System product name in external language | **Prayog** (factory); Gateflow stays runtime name |
| 2 | Internal OS vs package timeline | Stabilize A (both-lane factory + tenant/onboard) before B |
| 3 | How much Prayog-specific dialect is allowed in “portable pack” | TBD when B starts — skills contract shapes are the export surface |
| 4 | gateflow-ops depth before multi-programme dogfood | Minimum viable Mission Control after runtime APIs are stable |
| 5 | Relationship of this brief to gateflow vision | This brief = system SSOT for identity; gateflow vision = runtime deep-dive |

---

## References

- Programme config: [`config/programme.yaml`](../config/programme.yaml)
- Service catalog: [`config/service-catalog-drivestream-lab.yaml`](../config/service-catalog-drivestream-lab.yaml)
- Runtime vision (component): [gateflow-programme-vision.md](./gateflow-programme-vision.md)
- Skills consumer brief: `prayog-skills` → `docs/for-gateflow.md`
- Launchpad docs: `launchpad` → `docs/README.md`
