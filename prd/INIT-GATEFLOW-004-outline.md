# INIT-GATEFLOW-004 — Gateflow Mission Control (outline)

**Status:** outline (**Discovery locked** 2026-07-27) · **Author:** programme PM · **Date:** 2026-07-27  
**Draft PRD:** [INIT-GATEFLOW-004](./INIT-GATEFLOW-004.md) (draft)  
**Vision:** [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md) (§9 maturity · §10 H2 · lift-on-metrics)  
**Predecessors:** INIT-GATEFLOW-001 / 002 / 003 (control plane + live Cursor; **003 W2 spec-lane prove-it pairs with this PRD as dogfood**)  
**Component:** GATEFLOW · **Type:** operations / Mission Control

> **Outline — product lens.** Written for programme outcomes and operator needs.
> Engineering design (APIs, schemas, UI stack) belongs in impact map and
> gateflow-ops / gateflow spec PRs — not here.
>
> **Dogfood role:** This initiative’s PRD is the **real work** used to prove
> Gateflow’s **engg spec-lane** automation with live agents. Human
> checkpoints stay on; we collect firsthand efficacy so we can later **lift**
> controls when metrics justify it (vision §9).

---

## Document control

| Field | Value |
|-------|-------|
| Initiative ID | INIT-GATEFLOW-004 |
| Artifact | `prd/INIT-GATEFLOW-004-outline.md` |
| Programme | prayog |
| Primary delivery | **gateflow-ops** (Mission Control experience) |
| Supporting | **gateflow** only where the console needs missing programme capabilities (e.g. clearer wave summaries, repo onboarding records); **prayog-skills** pin change so spec-lane skills can be auto-run for 003 W2 prove-it |
| Out of this INIT | **Launchpad** greenfield / new-repo scaffolding |
| Depends on | 001–003 control plane + live Cursor; APIs for runs / metrics / wave start already exist |
| Target users | Engineering (operate waves), tech lead (trust & lift decisions), programme sponsor (are agents doing useful work between human stops?) |
| Identity (v0) | **Thin ops-user identity** — signed-in operators only; **no roles / RBAC** in this INIT |

---

## 1. Problem statement

We can already **run** delivery automation (start a wave, stop at human gates,
record how long each step took, open a PR). What we still cannot do well is
**operate and learn** from that automation as a programme:

- There is no **Mission Control** — people use APIs, scripts, and GitHub
  fragments instead of one place to see “what’s running, where did it stop, and
  was the agent useful?”
- Repos that are **already on the Prayog paved road** (harness + skills) are not
  first-class “members” of a Gateflow fleet; onboarding is ad hoc.
- Without a clear picture of **agent work vs human stops**, we cannot honestly
  move toward “agents do most of the work between intentional controls” — we
  would be guessing when to loosen human checkpoints.
- A basic status page would **defeat the purpose**: we need a high bar for
  understanding a wave, not a pretty empty shell.

Greenfield “create a new repo from templates” is a **different job** — that
stays with Launchpad. This INIT is for **brownfield / already-harnessed** repos
we are ready to operate.

---

## 2. Proposed solution (summary)

**INIT-GATEFLOW-004** delivers **Gateflow Mission Control (v0)** in gateflow-ops:

1. **Onboard** a repo that is already Prayog harness-enabled into Mission Control
   (register it, confirm it looks ready, set simple defaults for how waves run).
2. **Operate** — see the fleet of onboarded repos; start or inspect a delivery
   wave for one of them.
3. **Run cockpit (high bar)** — for a wave, show:
   - where we are on the **delivery process map** (the pinned process, read-only);
   - a **timeline of each step** with time taken, outcome, and which agent/model
     ran (when applicable);
   - a **full log pane** (readable narrative) **plus** deep links to comments,
     reports, and artifacts;
   - one-click **open the wave’s PR (and related GitHub links)**.
4. **Efficacy for lift decisions** — surface the signals vision cares about
   (unattended progress between human stops, retries/findings, time spent waiting
   on humans) so tech leads can decide later what to automate more — **without**
   removing checkpoints in this INIT.
5. **Dogfood** — use **this** PRD’s journey through the **engg spec-lane** skills as the
   live proof that Gateflow can run that lane with agents (003 W2 companion).

We do **not** invent a new delivery process in the console. Process stays in
pinned skills. We do **not** create new repos. We do **not** auto-merge.

---

## 3. Target experience (happy path)

```text
Sign in to Mission Control
        │
        ▼
Onboard an already-harnessed repo
  → strict readiness scorecard must pass
  → save as a fleet member with sensible defaults
        │
        ▼
From that repo: start a wave from the UI  (or open an existing one)
        │
        ▼
Run cockpit
  → process map: where we are / what finished / what’s next stop
  → timeline: each step’s duration, result, agent info
  → full log pane + links + “Open PR on GitHub”
        │
        ▼
Wave stops at a human checkpoint (expected)
  → human decides; metrics retained for later “lift” discussions
```

---

## 4. Ownership split (explicit)

| Topic | Owner in this INIT |
|-------|--------------------|
| Create new repo / scaffold / greenfield paved road | **Launchpad** — **out of scope** |
| Bring an **existing** harnessed repo into Gateflow operations | **Mission Control (004)** |
| Delivery process & skills (what steps exist) | **prayog-skills** (pinned) — console **displays**, does not rewrite |
| Who runs the coding agent | **Gateflow** (already); console **starts / observes** |
| When to remove or loosen a human checkpoint | **Programme / tech lead decision** using metrics — **not** auto in 004 |
| Merge to main | **Human accountable** — unchanged |
| Who can use Mission Control | **Thin ops-user record** (schema + sign-in) — **no role model** in v0; all signed-in ops users share the same capabilities |

---

## 5. Users and jobs-to-be-done

| User | Job to be done |
|------|----------------|
| **Engineering** | “I can onboard our harnessed repo, run a wave, and see exactly which steps the agent finished and where I must act.” |
| **Tech lead** | “I can see whether agents are doing useful work between our intentional stops — so we know what to automate next, without removing gates yet.” |
| **Programme sponsor** | “We have a real Mission Control story: fleet of paved-road repos, visible waves, and evidence — not a spreadsheet of API calls.” |

---

## 6. Scope — in (features)

### 6.0 Ops identity (thin — deliberate v0 bar)

- Introduce a **users schema** so Mission Control operators are real records, not
  anonymous shared tokens alone  
- **Sign in** to Mission Control as an ops user (happy path)  
- **No roles, no RBAC, no permission tiers** in this INIT — every signed-in ops
  user can do everything Mission Control offers (onboard, start waves, open
  cockpit, view efficacy)  
- Purposefully **thin**: enough to know *who* operated a wave and to gate console
  access; not an enterprise IAM product  
- Programme service token / network boundary patterns from 001–003 may still
  apply **behind** the console for API calls — product outcome is operator
  identity in the UI, not a new authorization matrix  

### 6.1 Fleet onboarding (harness-enabled repos only)

- Add a repo that already has Prayog harness / skills posture  
- **Strict readiness scorecard** before it joins the fleet — clear **pass / fail**
  reasons an operator understands; fail means “not onboardable yet,” with the
  reason made explicit — there is **no partial fleet-member state** in v0  
- Store it as an onboarded fleet member with simple wave defaults only when the
  scorecard passes  
- **Not** create the repo; **not** install harness from scratch (Launchpad)

### 6.2 Fleet home

- List onboarded repos with last-known wave health (running / stopped for human /
  failed / idle)  
- Enter a repo’s waves from here  

### 6.3 Start and inspect waves

- **Start a wave from the UI** for an onboarded repo (happy path — not API-only)  
- Open any in-flight or recent wave into the cockpit  

### 6.4 Run cockpit (Mission Control bar — must not be “basic”)

| Capability | Operator value |
|------------|----------------|
| **Process map** | See the pinned delivery path and where this wave sits (finished / current / waiting on human) |
| **Step timeline** | Every step: when it ran, how long, outcome, agent/model when used |
| **Full log pane + links** | Per-step (and wave-level) log/evidence pane: readable run narrative **and** deep links to PR comments, reports, and artifacts — not links-only archaeology |
| **GitHub jump** | Open the wave PR (and related issue/PR links) in one click |
| **Stop clarity** | Human checkpoint vs failure vs done — never ambiguous |

“Not fancy” means: no theatrical fleet globe or drag-and-drop process editor.  
“Not basic” means: an operator can answer *what happened* and *what I do next*
from the cockpit (map + timeline + **full log pane**) without leaving Mission
Control.

### 6.5 Efficacy visibility (for later lift-on-metrics)

- Show, in plain language, progress **between** human stops vs time spent
  **at** human stops  
- Show retries / “needs another pass” patterns where we already record them  
- Support the vision story: checkpoints stay; we **learn** before we lift  
- Collect the numbers Mission Control needs so operator experience and lift
  decisions can be evaluated over time, rather than setting a timed cockpit
  drill upfront  

Exact numeric promotion thresholds stay **out** of this INIT (vision: TBD after
firsthand samples).

### 6.6 Dogfood companion (programme, not a separate product)

- This outline / PRD is the **subject** of engg **spec-lane** live prove-it for
  INIT-GATEFLOW-003 W2  
- Exact skills: `spec-draft`, `initiative-feasibility`, `spec-technical-review`,
  `spec-implementation-plan`  
- Human checkpoints during that journey remain **expected**  
- Supporting skills-pin change so those engg spec-lane skills may be auto-dispatched
  is **in programme scope** for the prove-it, not a Launchpad feature  

---

## 7. Scope — out (explicit non-goals)

| Non-goal | Why |
|----------|-----|
| Greenfield repo creation / scaffolding | Launchpad owns the paved-road factory |
| Replacing or editing the delivery process inside the console | Process SSOT stays in pinned skills; console is a window, not a second rulebook |
| Auto-removing human checkpoints or auto-merge | Lift-on-metrics later; merge stays human-owned |
| Live second coding agent (OpenCode / Claude) | Later INIT after Cursor efficacy is clear |
| Slack / Teams as primary ops surface | Optional later; Mission Control is the home |
| Fancy multi-programme world map / cinema UI | High bar ≠ spectacle |
| Building a separate AI workflow engine in the console | Future graph tools must not fork process ownership |
| Replacing GitHub / IDE as the only place code is written | Mission Control operates waves; agents still code in the worker |
| Roles, RBAC, or per-feature permission tiers | Thin ops-user schema only; authorization model deferred |
| Enterprise SSO / SCIM / multi-tenant org admin | Later INIT when programme scale demands it |

---

## 8. Relationship to predecessors

| Topic | After 001–003 | This INIT |
|-------|---------------|-----------|
| Run waves, stop at humans, metrics, PRs | **Available** | **Operate and explain** them in Mission Control |
| Live Cursor on coding-cycle lane | **Proven** | Reuse; do not rebuild agents |
| Spec-lane live prove-it | **Still open (003 W2)** | This PRD is the **dogfood subject** |
| gateflow-ops | Scaffold only | **First real Mission Control product** |
| Launchpad | Factory | Unchanged — greenfield stays there |
| Autonomy destination | Stated in vision | Console **shows evidence**; does not lift gates by itself |

---

## 9. What a later INIT may cover

- Richer log transports / long-retention log products beyond the cockpit pane  
- Saving a historical snapshot of the process map with each wave  
- “Propose a process change” that opens work against skills (git), not a silent
  edit in Mission Control  
- Optional experiment lab for alternate flows (explicitly **not** programme SSOT)  
- Second live coding agent; Slack/Teams alerts  
- Numeric **lift** playbooks once we have months of Mission Control data  
- Policy-gated auto-merge only if the programme opts in after long green history  

---

## 10. Success criteria (outline level)

| Outcome | How we know |
|---------|-------------|
| Ops identity (thin) | Operator **signs in** as an ops user; actions attributable to a user record; **no role matrix** to configure |
| Harnessed repo onboarded | Operator can add a paved-road repo only after **strict scorecard pass**, then see it on the fleet home |
| Wave operable from console | **Start** and open a wave from the UI without raw API gymnastics as the happy path |
| Cockpit answers the job | Process place + step timeline + **full log pane with links** + Open PR — without “basic list only”; **no timed drill**; numbers collected for later targets |
| Stops are clear | Human wait vs failure vs complete is obvious |
| Efficacy visible | Lead can discuss unattended progress vs human wait using the console; Mission Control retains the underlying numbers |
| Process ownership unchanged | No delivery rules authored only inside ops |
| Greenfield unchanged | No Launchpad product work required for 004 exit |
| Dogfood served | Spec-lane prove-it for 003 W2 can use this initiative’s PRD as the live subject; checkpoints remain on |
| Lift not premature | No checkpoint removed as a 004 exit requirement |
| Onboarding honesty | Scorecard is **pass/fail only**; fail blocks with explicit reasons — **no partial fleet member** |

---

## 11. Decisions — locked in Discovery (2026-07-27)

| # | Question | Decision |
|---|----------|----------|
| 1 | Basic run list vs Mission Control bar? | **Mission Control** — high bar, not fancy chrome `(Source: User-confirmed)` |
| 2 | Greenfield create vs onboard harnessed? | **Onboard harness-enabled only**; greenfield = Launchpad `(Source: User-confirmed)` |
| 3 | Primary product surface? | **gateflow-ops** Mission Control `(Source: User-confirmed)` |
| 4 | Autonomy stance? | Agents should eventually do most work **between** intentional controls; **checkpoints stay** until metrics justify lifting `(Source: User-confirmed)` |
| 5 | Role of this PRD for 003? | **Dogfood subject** for spec-lane prove-it `(Source: User-confirmed)` |
| 6 | Process editing / AI workflow studio in 004? | **Out** — display process; future edit path must not fork SSOT `(Source: User-confirmed)` |
| 7 | Vision first? | Vision maturity + lift-on-metrics updated before this outline `(Source: User-confirmed)` |
| 8 | Start wave from UI in v0? | **Yes** — start from Mission Control is in scope `(Source: User-confirmed)` |
| 9 | Onboard readiness depth? | **Strict scorecard** — must pass before fleet membership `(Source: User-confirmed)` |
| 10 | Evidence richness in v0? | **Full log pane with links** (narrative + deep links) `(Source: User-confirmed)` |
| 11 | Identity model in v0? | **Thin users schema** for ops users; **no roles / RBAC** — deliberately minimal `(Source: User-confirmed)` |
| 12 | Can onboarding have a partial state? | **No** — scorecard result is pass/fail only; fail blocks onboarding with explicit reasons `(Source: User-confirmed)` |
| 13 | Timed cockpit drill as success bar? | **No** — collect the numbers first; use Mission Control to measure the experience over time before setting targets `(Source: User-confirmed)` |

### Still open / deferred

1. **Numeric lift thresholds** — deferred until after firsthand Mission Control samples (vision).  
2. **Exact scorecard checks** (which harness / pin / auth signals count) — product categories in Draft PRD; eng details in spec.  
3. **Log pane sources** (what streams into the narrative) — Draft PRD outcomes; eng wiring in spec.  
4. **Sign-in mechanism** (session vs token bridge, invite/bootstrap flow) — Draft PRD outcome; eng spec — **not** a roles discussion.  
5. **How Mission Control records operator-experience numbers** (what usage / efficacy telemetry to retain for later target-setting) — product outcome in Draft PRD; exact telemetry design in spec.  

---

## 12. Next steps (process)

| Step | Owner | Artifact |
|------|-------|----------|
| 1 | PM / sponsor | Outline locked (§11 decisions 1–13) |
| 2 | PM | Draft PRD from this outline (**done** — [INIT-GATEFLOW-004](./INIT-GATEFLOW-004.md)) |
| 3 | Programme | 003 W2 companion: allow spec-lane auto-run for dogfood prove-it |
| 4 | Engineering | Validate PRD → impact map (**gateflow-ops** primary) |
| 5 | Engineering | Spec + delivery waves for Mission Control v0 |

---

## Appendix A — Feature checklist (discussion lock)

| # | Feature | In this INIT |
|---|---------|--------------|
| 0 | Thin ops-user schema + sign-in (no roles) | Yes |
| 1 | Onboard harness-enabled repo to fleet | Yes |
| 2 | Strict onboarding scorecard (pass required; no partial) | Yes |
| 3 | Fleet home (repo list + wave health) | Yes |
| 4 | Start + inspect waves from console | Yes |
| 5 | Run cockpit: process map + step timeline | Yes |
| 6 | Full log pane with links + Open PR on GitHub | Yes |
| 7 | Per-step timing / outcome / agent info | Yes |
| 8 | Efficacy signals + collect numbers for later lift | Yes (visibility + retain) |
| 9 | Dogfood subject for 003 spec-lane prove-it | Yes (programme role) |
| 10 | Greenfield / Launchpad create | No |
| 11 | Edit delivery process in the console | No |
| 12 | Auto-lift checkpoints / auto-merge | No |
| 13 | Second coding agent / Slack-primary ops | No |
| 14 | Roles / RBAC / permission tiers | No |
| 15 | Timed cockpit drill as exit KPI | No |
| 16 | Partial fleet membership | No |

---

## Appendix B — One-sentence product

> Gateflow Mission Control lets the programme **onboard already-harnessed
> repos**, **run and inspect delivery waves**, and **see each step’s outcome,
> timing, and GitHub evidence** on a real cockpit — so we can move toward agents
> doing most of the work between intentional human controls **only when
> metrics earn it** — without Launchpad greenfield or rewriting the delivery
> process in the UI.
