# INIT-GATEFLOW-014 — One identity to call Gateflow, and retire the old shared-secret doors (outline)

**Status:** outline · **Author:** programme PM · **Date:** 2026-08-10  
**Draft PRD:** [INIT-GATEFLOW-014](./INIT-GATEFLOW-014.md)  
**Vision:** [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md)
**Component:** GATEFLOW · **Type:** trust / access to the control plane
**Predecessor:** INIT-GATEFLOW-013 (programme-first onboarding and real readiness) — **complete**; this initiative sits on top of the product surface that 012 and 013 already shipped.
**Related, not blocking:** INIT-GATEFLOW-004 (operations dashboard) — will eventually call the same APIs; this initiative does not build screens.

> **Outline** — problem framing and scope lock. Detailed requirements live in the
> Draft PRD. Locked decisions include discovery after outline (D10–D19).
>
> **Hard rule for this initiative:** landing the new identity story without
> deleting the old shared-secret story in the same delivery is a **failed**
> exit. “Deprecate later” is out of scope.

---

## Document control

| Field | Value |
|---|---|
| Initiative ID | INIT-GATEFLOW-014 |
| Artifact | `prd/INIT-GATEFLOW-014-outline.md` (this outline); Draft PRD at `./INIT-GATEFLOW-014.md` |
| Programme | prayog |
| Primary repo | drivestream-lab/gateflow |
| Supporting | prayog-meta (vision / decision records that still describe the old access story — updated so product truth matches); live-check scripts and examples inside gateflow that today teach the old secrets |
| Explicitly **not** touched this INIT | Operations dashboard screens; inventing a full login product / identity provider as a separate product line; changing the pin’s rules for when forge needs a human authorize vs automatic; encrypting stored secrets (plaintext accepted); implementing Claude Code / OpenCode **runners** (platform provision slots only this INIT) |
| In scope for credentials | **Per-programme** GitHub PAT (App later, same per-programme pattern) managed via platform_admin onboard; **platform-level** code-agent provision (Cursor now; choose at twin/initiative run) |
| Target users | **platform_admin** (seeded); **tenant_admin** (only programme role) |
| Depends on | INIT-GATEFLOW-012 and INIT-GATEFLOW-013 delivered — tenants, programme connect/choose, waves, runs, board, checkpoints, initiatives, and forge authorize already exist as callables. This initiative changes **who may call them**, **where secrets live**, and removes the old “shared password” API doors. |

---

## 1. Problem statement

INIT-GATEFLOW-012 and INIT-GATEFLOW-013 built a real multi-tenant factory front door: set up a tenant, connect to the programme, choose repos, get a trustworthy readiness answer, start waves, read runs, move board work, and authorize forge when the pin asks for a human.

They did **not** settle a single answer to: **who is allowed to do that?**

Today the factory still opens through **several parallel doors**:

1. **One global shared secret** — used for most day-to-day control-plane calls (start a wave, read runs, board, checkpoints, initiatives, metrics, authorize forge).
2. **A per-tenant shared secret** — used for onboarding calls (see the tenant, connect to the programme, choose repos).
3. **An open “create a tenant” door** — no secret required to register.
4. **A modern user-login path that is wired but not used** for these product calls — so the control plane is effectively “secret or open,” not “signed-in person.”

Separately, runtime secrets are incoherent: each tenant already holds a **PAT** in the DB, while **Cursor** is a process-global env key — not a clear “programme brings GitHub; factory offers agents” story.

Without this initiative, every later surface inherits a factory that cannot say **which person, for which programme,** called it — and cannot choose a code agent at run time without baking agent keys into every programme.

---

## 2. Proposed solution (summary)

Give Gateflow **one way in** for product calls — a signed-in user — **retire the old shared-secret API doors**, and split runtime secrets cleanly:

| What we're building | Solves |
|---|---|
| **One identity (JWT)** — platform_admin + tenant_admin | Shared-secret API doors |
| **Validate-then-create programme** — platform_admin receives that programme’s **PAT** + meta + workspace; validates; then creates | Open register; connect-after-create disorder |
| **Per-programme GitHub** — PAT day one (App later, still per programme); stored plaintext, managed via platform onboard | Process-global forge PAT as the product story |
| **Platform code-agent catalogue** — platform_admin provisions Cursor (slots for Claude/OpenCode later); **tenant_admin chooses** agent when running twin/initiative | Agent keys on every programme; env-only Cursor |
| **Fail closed + debt purge** | Dual-auth and teaching the old programme token |

**Unchanged on purpose:** GitHub PAT/App and agent keys are **runtime secrets inside Gateflow** — never the caller’s API Bearer. Webhooks stay signature-checked.

---

## 3. Locked product decisions

| ID | Decision |
|---|---|
| **D1** | Product APIs: Gateflow **user JWT only** — no dual-auth with programme service token |
| **D2** | Removing global programme-token auth is **in-band**; JWT while old token works = failed exit |
| **D3** | Open register + tenant bearer removed in this INIT |
| **D4** | GitHub credentials are **runtime-only** — never API Bearer |
| **D5** | Webhooks stay signature-based |
| **D6** | Teaching surfaces rewritten in this delivery |
| **D7** | No ops/login UI this INIT |
| **D8** | Vision/ADRs superseded to match JWT-only product edge |
| **D9** | New INIT on top of 012/013 behaviors |
| **D10** | **platform_admin** seeded by script; receives programme details; onboards programme; then onboards **tenant_admin** |
| **D11** | platform_admin may **list** programmes; does **not** onboard repos or run the twin |
| **D12** | Product edge fail closed |
| **D13** | Programme onboard is **validate-then-create** (clone meta, extract catalogue, then create) |
| **D14** | **Programme** plaintext secrets = **GitHub only** (PAT required day one; App materials later, still per programme) — never agent keys on the programme |
| **D15** | **Workspace root** set when onboarding the programme |
| **D16** | Auth debt purge + prove absence are peer capabilities |
| **D17** | Only **one** programme role: **tenant_admin** |
| **D18** | GitHub credentials are **per programme** (programme provides PAT at onboard; platform_admin stores it linked to that programme) |
| **D19** | **Code agents are platform-level**: platform_admin provisions the agent catalogue; tenant_admin **chooses** agent when running an initiative/twin — not at programme create |

---

## 4. Users and jobs-to-be-done

| User | Job to be done |
|---|---|
| **platform_admin** | “I seed the factory, provision platform code agents, receive a programme’s PAT/meta/workspace, validate-then-create the programme, attach tenant_admin, and list programmes — I don’t run twin or repo onboard.” |
| **tenant_admin** | “I log in, onboard/deboard repos, and run the developer twin — choosing which **platform** code agent to use for that run.” |
| **Person running live checks** | “Documented JWT path works; old programme API secret is refused.” |

---

## 5. Scope — in

| Area | What we build |
|---|---|
| One identity at the product edge | User JWT for platform_admin and tenant_admin |
| Validate-then-create programme | PAT + meta + workspace; catalogue from validation; no agent keys |
| Per-programme GitHub | PAT day one; App later same pattern; runtime forge/git use that programme’s credential |
| Platform agent catalogue | Provision Cursor now; slots for other runners; choose at twin/initiative start |
| Control-plane + repo lifecycle cutover | tenant_admin JWT; old doors removed |
| Fail closed + debt purge + prove absence | Same as before |

---

## 6. Scope — out

| Out | Rationale |
|---|---|
| Ops dashboard / login screens | Later |
| Full IdP product | Seed + Gateflow-issued JWTs this INIT |
| Multiple programme roles | Only tenant_admin |
| platform_admin running twin or repo onboard | Explicit split |
| Implementing Claude Code / OpenCode runners | Provision slots only |
| One shared factory GitHub PAT for all programmes | Rejected — PAT is per programme |
| Agent keys on the programme row | Rejected — agents are platform-level (D19) |
| Pin explicit vs automated forge policy | Unchanged |
| Encrypting DB secrets | Plaintext accepted |
| Dual-auth / “legacy still works” | Failed exit |

---

## 7. Capability walkthroughs (summary)

| Capability | Done looks like |
|---|---|
| One identity | JWT in; missing/invalid refused |
| Validate-then-create | Bad PAT/meta ⇒ no Programme; success stores per-programme PAT + workspace + catalogue |
| Platform agents | platform_admin provisions **DB catalogue**; start resolves **effective** runner (caller or programme per-lane default); unprovisioned/missing → reject; env Cursor not a product path |
| tenant_admin operate | Repo onboard/deboard + twin under JWT |
| Fail closed + debt purge | Old programme token / tenant bearer / open register gone (refuse when JWT edge live; delete dead doors) |

---

## 8. Delivery waves (proposed)

| Wave | Intent |
|---|---|
| **W0** | Seed platform_admin + user JWT edge |
| **W1** | Validate-then-create (per-programme PAT) + tenant_admin + platform agent provision |
| **W2** | Cut over repo lifecycle + twin (JWT + choose platform agent); fail closed |
| **W3** | Dead-door removal |
| **W4** | Prove absence + exit checklist |

---

## 9. Success criteria (initiative exit)

1. tenant_admin runs twin and repo lifecycle with user JWT only; old programme service token → refused.  
2. platform_admin validate-then-create stores **that programme’s PAT** (no agent keys on programme).  
3. platform_admin can provision platform agents in a **DB catalogue**; twin/initiative run uses **effective** runner (caller or per-lane default); missing/unprovisioned ⇒ rejected; env Cursor not a product path.  
4. Unauthenticated product calls fail closed; open register + tenant bearer gone.  
5. Teaching surfaces match JWT + new secret model; dual-auth does not ship.

---

## 10. Dependencies and non-goals for partners

| Partner | Expectation |
|---|---|
| **prayog-meta** | Vision/ADRs updated for JWT-only product edge + secret split (per-programme GitHub; platform agents) |
| **gateflow-ops** | Not built; inherits callable auth story |
| **GitHub / forge** | Outbound uses **per-programme** stored credential — not caller Bearer |
| **012 / 013 teams** | Access + secret placement change; onboarding/twin *behaviors* largely reused |

---

## 11. Risks

| Risk | Mitigation |
|---|---|
| Platform agent key blast radius (shared across programmes) | Named accepted risk; only platform_admin provisions |
| Programme PAT still wide within one programme | Same class as 012; explicit |
| Lab automation still sends programme API token | Debt purge + negative verify |
| Existing lab tenants at cutover | **Resolved (OQ-3)** — wipe; re-onboard via Draft path |
| Dual-auth sneaks back | D1/D2 + exit gate |

---

## 12. Open questions

| ID | Open question | Status |
|---|---|---|
| OQ-1 | Exact lab JWT minting mechanics | **Resolved** — seed script mints tokens **and** login API returns JWT (no UI) — see Draft PRD |
| OQ-2 | GitHub App per-programme **runtime** timing vs storage shape first | **Resolved** — storage shape only this INIT; PAT required; App fields reserved unused — see Draft PRD |
| OQ-3 | Existing 012/013 lab tenants: migrate, wipe, or re-onboard? | **Resolved** — wipe; re-onboard via new path — see Draft PRD |
| OQ-4 | Agent / runner obligation at start | **Resolved** — effective runner (+ model) always required (caller or programme per-lane default); must be provisioned in platform DB catalogue; env Cursor not a product path — see Draft PRD |

`OQ-1`–`OQ-4` are resolved in the Draft PRD. Do **not** implement from this outline alone.

---

## 13. Next steps

1. Impact map → programme sign-off → waves (from Draft PRD).  
2. Do **not** build from outline alone — Draft PRD is normative.  
3. Re-validate after outline sync if needed.

---

## Exit gate (outline wording)

> **Exit:** User JWT only on product APIs; per-programme GitHub PAT (no agent keys on programme); platform **DB** agent catalogue with effective runner (caller or per-lane default); env Cursor not a product path; old programme service token, tenant bearer, and open register gone; dual-auth or “delete later” is a **failed** exit.
