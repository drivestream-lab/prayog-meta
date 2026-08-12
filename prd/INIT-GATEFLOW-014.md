# INIT-GATEFLOW-014 — One identity to call Gateflow, and retire the old shared-secret doors

**Status:** draft PRD · **Author:** programme PM · **Date:** 2026-08-10  
**Outline:** [INIT-GATEFLOW-014-outline](./INIT-GATEFLOW-014-outline.md)  
**Vision:** [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md)  
**Component:** GATEFLOW · **Type:** trust / access to the control plane  
**Predecessor:** INIT-GATEFLOW-013 (programme-first onboarding and real readiness) — **complete**; this initiative sits on top of the product surface that 012 and 013 already shipped.  
**Related, not blocking:** INIT-GATEFLOW-016 (operations dashboard, formerly INIT-GATEFLOW-004 — retired) — will eventually call the same APIs; this initiative does not build screens.

> **Draft PRD** — Locked decisions D1–D19 (outline) carried forward unchanged.
> Discovery locks from this Draft: OQ-1 (seed + login JWT minting), OQ-2
> (GitHub App storage shape only), OQ-3 (wipe lab tenants), OQ-4 (effective
> runner required from DB catalogue — caller or programme per-lane default;
> unprovisioned → reject; env Cursor not a product path). Exit proof = verify /
> live-check scripts (not CI path greps as the primary gate). Primary delivery
> = **gateflow**; supporting = **prayog-meta** (vision/ADRs that still teach
> the old access story — updated so product truth matches). No
> `prayog-skills` contract change expected.
>
> **Greenfield / breaking:** this INIT does not preserve dual-auth or env-Cursor
> as product paths. It **supersedes INIT-GATEFLOW-012 G3** (JWT parked; tenant
> bearer as product auth). Eng follow-on: supersede ADR-005 / ADR-011.
>
> **Hard exit rule:** landing the new identity story without deleting the old
> shared-secret story in the same delivery is a **failed** exit. Dual-auth or
> “deprecate later” does not ship. No dual-auth window between waves: old doors
> are refused as soon as the JWT product edge is live (**W2**); **W3** deletes
> dead code. `(Source: User-confirmed)`
>
> **As-built baseline (Source: verified against gateflow code this session).**
> (1) `AuthMiddleware` already validates Bearer JWTs and sets
> `request.state.auth` (`src/common/auth/middleware.py`) but **every** real
> product prefix (`/api/v1/waves`, `/initiatives`, `/runs`, `/metrics`,
> `/board`, `/checkpoints`, `/tenants`, …) is listed in `public_paths` in
> `src/app.py`, so JWT gates nothing today — confirming D1’s “wired but not
> used” premise. (2) Day-to-day control-plane calls use
> `verify_programme_service_token` against one global
> `PROGRAMME_SERVICE_TOKEN` (`src/api/v1/programme_token.py`) — the global
> shared-secret door. (3) Onboarding calls use
> `verify_tenant_bearer_token` (`src/api/v1/tenant_token.py`) on
> `tenant_routes` / `programme_routes` — the per-tenant shared-secret door.
> (4) `POST` tenant register has **no** auth dependency
> (`src/api/v1/tenant_routes.py` `register_tenant`) — the open register door.
> (5) Cursor is process-global `CURSOR_API_KEY` via `CursorAgentSettings`
> (`src/configs/cursor_agent_settings.py`) — not a platform DB catalogue.
> (6) As-built today: plaintext GitHub PAT lives on the **tenant** row
> (`TenantSchema.pat`, INIT-GATEFLOW-012). **Product intent (this INIT):** a
> new **Programme** entity holds that programme’s PAT (+ workspace + lane
> defaults) at validate-then-create; Tenant becomes a child attach target for
> `tenant_admin`. `(Source: User-confirmed)` Module/route names below are
> evidence, not product vocabulary.

---

## Document control

| Field | Value |
|-------|-------|
| Initiative ID | INIT-GATEFLOW-014 |
| Programme | prayog |
| Primary repo | drivestream-lab/gateflow |
| Supporting | drivestream-lab/prayog-meta (vision / decision records that still describe the old access story); live-check scripts and examples inside gateflow that today teach the old secrets |
| Explicitly **not** touched this INIT | Operations dashboard screens; inventing a full login product / identity provider as a separate product line; changing the pin’s rules for when forge needs a human authorize vs automatic; encrypting stored secrets (plaintext accepted); implementing Claude Code / OpenCode **runners** (platform provision slots only this INIT) |
| Entity model | **Programme** (new) holds per-programme GitHub PAT, workspace root, and per-lane runner/model defaults; **Tenant** is a child / attach target for `tenant_admin`. `(Source: User-confirmed)` |
| In scope for credentials | **Per-programme** GitHub PAT (App later, same per-programme pattern) via platform_admin validate-then-create; **platform-level** code-agent provision in a **separate DB catalogue table** (Cursor now; choose or default at twin/initiative run) |
| Target users | **platform_admin** (seeded); **tenant_admin** (only programme role; multiple users allowed) |
| Depends on | INIT-GATEFLOW-012 and INIT-GATEFLOW-013 delivered — tenants, programme connect/choose, waves, runs, board, checkpoints, initiatives, and forge authorize already exist as callables. This initiative changes **who may call them**, **where secrets live**, introduces Programme, and removes the old “shared password” API doors. **Supersedes INIT-GATEFLOW-012 G3.** |
| Lab cutover | **Wipe** existing 012/013 lab tenants / shared-secret rows; re-onboard via Programme validate-then-create + attach Tenant/`tenant_admin` (resolves OQ-3) |
| Exit proof | Verify / live-check scripts that assert JWT happy paths **and** refusal of old doors (resolves discovery #1) |

---

## 1. Executive Summary

### Problem Statement

INIT-GATEFLOW-012 and INIT-GATEFLOW-013 shipped a real multi-tenant factory surface, but Gateflow still opens through several parallel doors: one global programme service token for control-plane calls, a per-tenant bearer for onboarding, an unauthenticated tenant register, and a JWT middleware that is wired yet bypassed for every product route. Runtime secrets are equally incoherent — GitHub PAT lives on the tenant row while Cursor is a process-global env key — so later surfaces cannot say **which person, for which programme,** called the API, or choose a code agent without baking agent keys into every programme.

### Proposed Solution

Make **Gateflow-issued user JWTs** the only product-edge credential (`platform_admin` + `tenant_admin`), refuse and then delete the programme-token / tenant-bearer / open-register doors in-band (no dual-auth window), replace open register with **validate-then-create Programme** onboard (per-programme GitHub PAT + meta + workspace; App fields reserved unused; Tenant as child), and move code-agent keys to a **platform DB catalogue** that start calls resolve against (caller choice or programme per-lane default).

### Success Criteria

| KPI | Target | Measurement |
|-----|--------|-------------|
| **JWT-only product edge** | Every product API in Appendix C accepts a valid `platform_admin` or `tenant_admin` user JWT for its allowed actions; missing/invalid JWT → refused | Unit + verify |
| **Old doors gone** | Global programme service token, tenant bearer, and open register are refused when JWT edge is live and absent at exit; dual-auth does not ship | Unit + verify (negative paths) + inspection |
| **Validate-then-create Programme** | Bad PAT/meta/workspace ⇒ no Programme row (and no child Tenant); success stores that programme’s PAT (plaintext) + workspace root + repo catalogue from validation — never agent keys | Unit + verify |
| **Platform DB agents + effective runner** | `platform_admin` provisions Cursor into a DB catalogue; start resolves effective runner (+ model) from caller or programme lane default; unprovisioned/missing → rejected; env Cursor does not authorize | Unit + verify |
| **Teaching surfaces match** | Live-check scripts and docs teach JWT + Programme PAT + DB catalogue only; they no longer teach `PROGRAMME_SERVICE_TOKEN` / tenant bearer / env Cursor as the product path | Inspection + verify |

### Capability ↔ wave ↔ requirement map

| CAP | Wave | Capability | REQ |
|-----|------|------------|-----|
| CAP-01 | W0 | Seed `platform_admin` + mint/login JWT edge | REQ-01–REQ-07, REQ-43 |
| CAP-02 | W1 | Validate-then-create Programme (per-programme PAT) | REQ-08–REQ-14 |
| CAP-03 | W1 | Attach `tenant_admin` + list programmes | REQ-15–REQ-18, REQ-44, REQ-47 |
| CAP-04 | W1 | Platform agent DB catalogue + programme lane defaults | REQ-19–REQ-22, REQ-40–REQ-42, REQ-45 |
| CAP-05 | W2 | `tenant_admin` operates under JWT (repos + twin/initiative + effective agent) | REQ-23–REQ-28 |
| CAP-06 | W2 | Fail closed + **refuse** old shared-secret doors | REQ-29–REQ-33 |
| CAP-07 | W3 | Dead-door **deletion** + wipe cutover | REQ-34–REQ-35, REQ-46 |
| CAP-08 | W4 | Prove absence + teaching-surface rewrite | REQ-36–REQ-39 |

---

## 2. User Experience & Functionality

### User Personas

| Persona | Role | Primary need |
|---------|------|--------------|
| **platform_admin** | Seeds the factory; owns platform agent keys and programme intake | Validate-then-create Programmes with that programme’s GitHub PAT, attach `tenant_admin`(s), provision platform agents in the DB catalogue, set/list programmes — without running twin or repo onboard |
| **tenant_admin** | Only programme-scoped product role | Log in, onboard/deboard repos, and run twin / initiative work — supplying runner/model or relying on that programme’s per-lane defaults against the platform DB catalogue |
| **Person running live checks** | Lab / PE verifying the cutover | Documented JWT path works; old programme API secret and tenant bearer are refused |

### User Stories & Acceptance Criteria

#### US-1 — Seed the factory identity and obtain a JWT `(CAP-01)`

**As a** platform operator, **I want** a seeded `platform_admin` and a way to obtain Gateflow-issued user JWTs (seed mint + login API, no UI), **so that** every product call can name a person and role instead of a shared password.

**Acceptance criteria:**

- [ ] A seed script creates at least one `platform_admin` user and can mint a usable Gateflow-issued JWT for that user
- [ ] Re-running seed for the same `platform_admin` is idempotent (same admin; no duplicate conflicting identity)
- [ ] A login API (no UI) accepts credentials for an existing user and returns a Gateflow-issued user JWT
- [ ] Login with invalid credentials is refused; no JWT is issued (REQ-03 / REQ-43)
- [ ] Product routes listed in Appendix C reject missing, malformed, expired, or wrong-issuer JWTs with a named unauthorized outcome
- [ ] JWT claims identify the user and role (`platform_admin` or `tenant_admin`); `tenant_admin` tokens are bound to their Programme
- [ ] GitHub PATs and agent keys are never accepted as the caller’s API Bearer

#### US-2 — Validate-then-create a programme `(CAP-02)`

**As a** platform_admin, **I want** to receive a programme’s GitHub PAT, meta repo, and workspace root, validate them, and only then create the **Programme**, **so that** bad credentials never leave a half-created programme and every programme owns its own GitHub credential. `(Source: User-confirmed)`

**Acceptance criteria:**

- [ ] Onboard input includes: programme identity, meta location, workspace root, and **required** GitHub PAT (day one)
- [ ] Validation clones/reads meta and extracts the repo catalogue **before** durable create; failure ⇒ no Programme row and no child Tenant left behind
- [ ] Success stores that programme’s PAT in plaintext on the **Programme**, sets workspace root, and persists repo catalogue from validation
- [ ] Agent keys are rejected if supplied on the programme onboard path — agents are platform-level only (DB catalogue)
- [ ] GitHub App materials may be stored in reserved fields for a later INIT; they are **not** used at runtime this INIT (storage shape only)
- [ ] Open, unauthenticated tenant register is not the product path for creating programmes (removed; not offered as a fallback)

#### US-3 — Attach tenant_admin and list programmes `(CAP-03)`

**As a** platform_admin, **I want** to attach a `tenant_admin` to a programme and list programmes I have onboarded, **so that** day-to-day operation has exactly one programme role type and I can see what exists without running work myself.

**Acceptance criteria:**

- [ ] After Programme create, platform_admin can create/attach a `tenant_admin` (Tenant child) for that programme
- [ ] Attaching to an unknown / non-existent programme is rejected with a named reason; 0 attach
- [ ] Multiple `tenant_admin` users may be attached to one programme; attaching the same identity again is idempotent
- [ ] Only one programme role **type** exists: `tenant_admin` — no additional programme roles this INIT
- [ ] platform_admin can list programmes
- [ ] platform_admin cannot onboard/deboard repos or start twin/initiative runs (those require `tenant_admin`)

#### US-4 — Provision platform code agents `(CAP-04)`

**As a** platform_admin, **I want** to provision Cursor (and reserve slots for other runners) in a **platform DB catalogue**, **so that** programmes never hold agent keys and starts resolve credentials only from that catalogue.

**Acceptance criteria:**

- [ ] platform_admin can provision Cursor into a durable **platform agent catalogue table** (key stored as a platform runtime secret, plaintext accepted)
- [ ] Provisioning with a blank or missing key is rejected; 0 catalogue row that can authorize a run
- [ ] Catalogue may include reserved slots for Claude Code / OpenCode without implementing those runners this INIT
- [ ] platform_admin can set per-Programme **default runner + model per lane** (at least: spec, implement, closeout, initiative)
- [ ] Twin/initiative (wave) start resolves an **effective** runner (+ model): caller value if supplied, else that programme’s default for the lane
- [ ] If effective runner is missing (caller omitted and no default) or not provisioned in the DB catalogue (or key missing), start is rejected with a named reason
- [ ] Process-global `CURSOR_API_KEY` must not authorize a product run when catalogue resolution fails or as a silent fallback

#### US-5 — Operate the programme under JWT `(CAP-05)`

**As a** tenant_admin, **I want** to onboard/deboard repos and run twin/initiative work using only my user JWT, **so that** day-to-day factory calls name me and my programme — not a shared service password.

**Acceptance criteria:**

- [ ] Repo connect/choose/onboard/deboard and twin/initiative start/read paths that 012/013 already shipped accept `tenant_admin` JWT for the matching Programme (Appendix C)
- [ ] The same calls **refuse** a programme service token and refuse a tenant bearer as soon as the JWT product edge is live (**W2** — no dual-auth window). `(Source: User-confirmed)` Dead code deletion is **W3**.
- [ ] Starting twin/initiative uses the effective platform-catalogue agent (caller or lane default); forge/git outbound uses **that programme’s** stored GitHub credential — never the caller JWT
- [ ] Webhooks remain signature-checked and are not switched to user JWT

#### US-6 — Fail closed when identity is wrong `(CAP-06)`

**As a** programme owner, **I want** unauthenticated or wrongly-scoped product calls to fail closed, **so that** “secret or open” cannot return as the default.

**Acceptance criteria:**

- [ ] Unauthenticated product calls (Appendix C) are refused
- [ ] A `tenant_admin` JWT cannot perform `platform_admin`-only actions (programme create, platform agent provision, attach tenant_admin, list-all programmes as admin)
- [ ] A `platform_admin` JWT cannot perform `tenant_admin`-only actions (repo lifecycle, twin/initiative start)
- [ ] A `tenant_admin` JWT for programme A cannot operate programme B’s repos or runs

#### US-7 — Delete the old doors and prove they are gone `(CAP-07, CAP-08)`

**As a** person running live checks, **I want** the old shared-secret doors removed and proven absent, **so that** dual-auth or “delete later” cannot claim success.

**Acceptance criteria:**

- [ ] Global programme service token auth is removed from product APIs (not left as a second accepted Bearer)
- [ ] Per-tenant bearer auth is removed from product APIs
- [ ] Open tenant register is removed
- [ ] Existing 012/013 lab tenants that depended on the old doors are **wiped**; re-onboard uses Programme + attach (OQ-3)
- [ ] Wipe while any run is in flight for that programme is **rejected** (0 wipe)
- [ ] Verify / live-check scripts prove: JWT happy path works; old programme token → refused; tenant bearer → refused; open register → gone; env Cursor does not authorize product start
- [ ] Teaching surfaces (verify scripts, README examples, as-built docs that instruct callers) teach JWT + per-programme GitHub + DB catalogue only
- [ ] Shipping JWT while the old programme token still works is a **failed** exit

### Non-Goals

| Non-goal | Why |
|----------|-----|
| Ops dashboard / login screens | Later (INIT-GATEFLOW-016 / follow-ons); login is API-only this INIT |
| Full IdP / SSO product line | Seed + Gateflow-issued JWTs only |
| Multiple programme **role types** | Only `tenant_admin` (multiple users of that role allowed) |
| `platform_admin` running twin or repo onboard | Explicit split (D11) |
| Implementing Claude Code / OpenCode runners | Provision slots only |
| One shared factory GitHub PAT for all programmes | Rejected — PAT is per programme |
| Agent keys on the programme row | Rejected — agents are platform-level DB catalogue (D19) |
| Changing pin explicit vs automated forge policy | Unchanged |
| Encrypting DB secrets | Plaintext accepted (same class as 012) |
| Dual-auth / “legacy still works” | Failed exit; greenfield / breaking |
| Migrating existing lab tenants in place | Wipe + re-onboard (OQ-3) |
| Runtime use of GitHub App credentials this INIT | Storage shape reserved only (OQ-2) |
| Keeping INIT-GATEFLOW-012 G3 (JWT parked; tenant bearer as product auth) | **Superseded** by this INIT |

### Assumptions

| ID | Assumption | Status | Dependent REQs |
|----|------------|--------|----------------|
| A1 | INIT-GATEFLOW-012/013 callables (repos, readiness, waves, runs, board, checkpoints, initiatives, forge authorize) remain the behavior substrate; this INIT changes auth + secret placement + Programme entity, not those behaviors’ core contracts | Confirmed by scope | REQ-23–REQ-28 |
| A2 | Dormant `AuthMiddleware` / JWT settings can be activated and role-shaped for `platform_admin` / `tenant_admin` without introducing a second JWT stack | Assumed — eng owns exact claim/schema design; product requires Gateflow-issued user JWT only | REQ-01–REQ-07 |
| A3 | Lab environments can wipe 012/013 tenant/secret rows and re-onboard without preserving old bearer tokens | Locked (OQ-3) | REQ-35, REQ-36 |
| A4 | Cursor remains the only implemented runner; other catalogue slots are name/place holders until a later INIT implements them | Confirmed by scope | REQ-19–REQ-22, REQ-40–REQ-41 |
| A5 | Webhooks stay outside the user-JWT product edge and continue signature verification | Confirmed by D5 | REQ-28 |

### Error table (product-normative)

| Situation | Result | Side effects |
|-----------|--------|--------------|
| Product call with missing/invalid/expired JWT | Refused (unauthorized) | 0 state change |
| Product call with programme service token after JWT edge live | Refused | 0 state change |
| Product call with tenant bearer after JWT edge live | Refused | 0 state change |
| Unauthenticated tenant register after cutover | Gone / refused | 0 create |
| Login with invalid credentials | Refused | 0 JWT |
| Validate-then-create with bad PAT or unreachable/malformed meta | Rejected, named reason | No Programme row; no child Tenant |
| Programme onboard includes agent keys | Rejected | 0 Programme create |
| Attach `tenant_admin` to unknown programme | Rejected, named reason | 0 attach |
| Duplicate attach of same `tenant_admin` identity | Idempotent success | No second conflicting row |
| Re-seed same `platform_admin` | Idempotent success | No conflicting duplicate admin |
| Provision agent with blank/missing key | Rejected, named reason | 0 usable catalogue credential |
| `tenant_admin` attempts platform_admin-only action | Refused | 0 state change |
| `platform_admin` attempts twin start or repo onboard | Refused | 0 state change |
| Twin/initiative start: effective runner unprovisioned or key missing in DB catalogue | Rejected, named reason | 0 run started |
| Twin/initiative start: caller omits runner/model and programme has no lane default | Rejected, named reason | 0 run started |
| Env `CURSOR_API_KEY` used as sole credential when catalogue resolution fails | Must not authorize | 0 run started |
| Cross-programme `tenant_admin` JWT used on another programme | Refused | 0 state change |
| Wipe while a run is in flight for that programme | Rejected, named reason | 0 wipe |

### Open questions

| ID | Open question | Status |
|----|----------------|--------|
| OQ-1 | Exact lab JWT minting mechanics | **Resolved** — seed script mints tokens **and** login API returns JWT (no UI) |
| OQ-2 | GitHub App per-programme runtime timing vs storage shape first | **Resolved** — storage shape only this INIT; PAT required; App fields reserved unused |
| OQ-3 | Existing 012/013 lab tenants: migrate, wipe, or re-onboard? | **Resolved** — wipe; re-onboard via new path |
| OQ-4 | Agent / runner obligation at start | **Resolved** — effective runner (+ model) always required (caller or programme per-lane default); must be provisioned in platform DB catalogue; env Cursor not a product path `(Source: User-confirmed)` |
| OQ-5 | Exact JWT claim schema (issuer/audience/role/programme binding) and password/credential store for login API | Open — engineering design; product only requires Gateflow-issued user JWT with role + programme binding for `tenant_admin` |
| OQ-6 | Whether health/internal/webhook paths remain on the JWT public allowlist exactly as today | Open — eng; product requires webhooks signature-based and product APIs JWT-only (Appendix C) |

`OQ-1`–`OQ-4` from the outline are resolved above and not carried as blocking opens.

---

## 3. Functional requirements

### Capabilities

| ID | Capability | Covers |
|----|------------|--------|
| CAP-01 | Seed `platform_admin` + mint/login JWT edge | REQ-01–REQ-07, REQ-43 |
| CAP-02 | Validate-then-create Programme (per-programme PAT) | REQ-08–REQ-14 |
| CAP-03 | Attach `tenant_admin` + list programmes | REQ-15–REQ-18, REQ-44, REQ-47 |
| CAP-04 | Platform agent DB catalogue + programme lane defaults | REQ-19–REQ-22, REQ-40–REQ-42, REQ-45 |
| CAP-05 | `tenant_admin` operates under JWT | REQ-23–REQ-28 |
| CAP-06 | Fail closed + refuse old doors | REQ-29–REQ-33 |
| CAP-07 | Dead-door deletion + wipe cutover | REQ-34–REQ-35, REQ-46 |
| CAP-08 | Prove absence + teaching-surface rewrite | REQ-36–REQ-39 |

### Requirements

| ID | Requirement | Outline | Condition | Observable result | Evidence |
|----|-------------|---------|-----------|--------------------|----------|
| REQ-01 | A seed script creates at least one `platform_admin` and can mint a Gateflow-issued user JWT for that user | D10, OQ-1 | Seed run | `platform_admin` exists; minted JWT accepted by product edge | unit + verify |
| REQ-02 | A login API (no UI) authenticates an existing user and returns a Gateflow-issued user JWT | OQ-1 | Login call with valid credentials | JWT returned; usable on product APIs | unit + verify |
| REQ-03 | Login with invalid credentials is refused; no JWT issued | Fail-closed | Bad login | Named unauthorized/invalid; 0 token | unit + verify |
| REQ-04 | Product APIs listed in Appendix C accept only Gateflow-issued user JWTs as the caller Bearer | D1 | Product call | Valid JWT authorized per role; non-JWT secrets not accepted as dual-auth | unit + verify |
| REQ-05 | Missing, malformed, expired, or wrong-issuer/audience JWTs are refused | D12 | Bad JWT | Unauthorized; 0 state change | unit + verify |
| REQ-06 | JWT identifies user and role; `tenant_admin` is bound to its Programme | D10, D17 | Authorized call | Role/programme binding enforced | unit + verify |
| REQ-07 | GitHub PAT and agent keys are never accepted as the caller’s API Bearer | D4 | Caller sends PAT/agent key as Bearer | Refused as product auth | unit + verify |
| REQ-08 | `platform_admin` can validate-then-create a **Programme** with meta location, workspace root, and **required** GitHub PAT | D13, D14, D15, D18 | Onboard call | On success: Programme exists with PAT + workspace + repo catalogue from validation | unit + verify |
| REQ-09 | Validation (PAT usability + meta clone/extract) runs before durable create | D13 | Onboard call | Create happens only after validation succeeds | unit + verify |
| REQ-10 | Validation failure leaves no Programme row and no child Tenant behind | Fail-closed | Bad PAT / bad meta | Named reason; 0 durable Programme/Tenant | unit + verify |
| REQ-11 | Successful create stores **that programme’s** PAT in plaintext on the Programme — not a factory-global GitHub credential | D14, D18 | Successful onboard | PAT readable only as that programme’s runtime secret; not returned on ordinary read APIs | unit + inspection |
| REQ-12 | Workspace root is set at programme onboard and used as the programme’s workspace root thereafter | D15 | Successful onboard | Workspace root persisted and used by subsequent lifecycle | unit + verify |
| REQ-13 | Agent keys supplied on programme onboard are rejected; no agent key is stored on the Programme | D14, D19 | Onboard with agent key | Rejected; 0 create | unit + verify |
| REQ-14 | GitHub App fields may exist as reserved storage shape; they are unused at runtime this INIT; PAT remains the required runtime GitHub credential | OQ-2 | Onboard / forge/git | PAT used; App materials not required and not runtime-active | unit + inspection |
| REQ-15 | `platform_admin` can create/attach a `tenant_admin` (Tenant child) for an onboarded Programme | D10, D17 | Attach call | `tenant_admin` can log in and receive JWT for that programme | unit + verify |
| REQ-16 | The only programme role type is `tenant_admin` | D17 | Role model | No additional programme role types ship | inspection |
| REQ-17 | `platform_admin` can list programmes | D11 | List call | Onboarded programmes visible | unit + verify |
| REQ-18 | `platform_admin` cannot onboard/deboard repos or start twin/initiative runs | D11 | platform_admin JWT on those calls | Refused | unit + verify |
| REQ-19 | `platform_admin` can provision Cursor into a platform-level **DB** agent catalogue | D19 | Provision call | Cursor present as a choose-able / default-able platform agent | unit + verify |
| REQ-20 | Catalogue may include reserved slots for Claude Code / OpenCode without implementing those runners | Scope-out | Provision/list | Slots representable; runners not required to execute | unit + inspection |
| REQ-21 | Twin/initiative start resolves an **effective** runner (+ model): caller if supplied, else Programme per-lane default | D19, OQ-4 | Start call | Effective runner/model recorded/used for that run | unit + verify |
| REQ-22 | If the effective runner is missing or not provisioned (or key missing) in the DB catalogue, start is rejected with a named reason | OQ-4 | Start with bad/missing effective runner | Rejected; 0 run | unit + verify |
| REQ-23 | `tenant_admin` JWT authorizes repo lifecycle calls (connect/choose/onboard/deboard) for its programme | D1, D9 | Repo lifecycle under JWT | Same 012/013 behaviors, new auth | unit + verify |
| REQ-24 | `tenant_admin` JWT authorizes twin/initiative start and Appendix C control-plane reads/actions for its programme | D1, D9 | Control-plane under JWT | Authorized for matching programme | unit + verify |
| REQ-25 | Outbound forge/git for a programme uses that programme’s stored GitHub credential — never the caller JWT | D4, D18 | Forge/git operation | Per-programme PAT used | unit + verify |
| REQ-26 | Twin/initiative start loads agent credentials **only** from the platform DB catalogue for the effective runner — never a programme-stored agent key and never env Cursor as product auth | D19 | Agent dispatch | Catalogue key used; env alone does not authorize | unit + verify |
| REQ-27 | Pin explicit vs automated forge authorize policy is unchanged | Scope-out | Authorize path | Behavior matches pre-INIT policy | inspection + unit |
| REQ-28 | Webhooks remain signature-checked and are not converted to user-JWT auth | D5 | Webhook ingress | Signature verification unchanged | unit + verify |
| REQ-29 | Unauthenticated product API calls (Appendix C) fail closed | D12 | No auth header | Refused | unit + verify |
| REQ-30 | Role mismatch fails closed (`tenant_admin` ↛ platform-only; `platform_admin` ↛ tenant-only) | D10, D11, D17 | Cross-role call | Refused | unit + verify |
| REQ-31 | Cross-programme `tenant_admin` access fails closed | D12 | JWT for A on B’s resources | Refused | unit + verify |
| REQ-32 | When the JWT product edge is live (**W2**), presenting the global programme service token is refused (no dual-auth window) | D1, D2, D16 | Call with `PROGRAMME_SERVICE_TOKEN` | Refused | unit + verify |
| REQ-33 | When the JWT product edge is live (**W2**), presenting a tenant bearer is refused | D3, D16 | Call with tenant bearer | Refused | unit + verify |
| REQ-34 | Open (unauthenticated) tenant register is removed | D3 | Unauthenticated register | Gone / refused; 0 create | unit + verify |
| REQ-35 | Existing 012/013 lab tenants/shared-secret rows are wiped at cutover; re-onboard uses CAP-02/CAP-03 | OQ-3 | Cutover | Old lab tenants gone; new Programme path works | verify + runbook |
| REQ-36 | Verify / live-check scripts prove JWT happy path for `platform_admin` and `tenant_admin` | Exit | Verify run | Scripts pass on JWT path | verify |
| REQ-37 | Verify / live-check scripts prove refusal of programme service token, tenant bearer, and open register | Exit, D16 | Negative verify | Named refusal / absence | verify |
| REQ-38 | Teaching surfaces (verify scripts, caller examples, docs that instruct product callers) describe JWT + per-programme GitHub + DB catalogue only | D6, D8 | Doc/script review | No instruction to use old product doors or env Cursor as product path | inspection |
| REQ-39 | Vision/ADR updates in prayog-meta supersede programme-token product-edge language to match JWT-only + secret split (and note 012 G3 / ADR-005/011 supersession) | D8 | Meta update | Product truth matches this INIT | inspection |
| REQ-40 | Platform agent catalogue is a durable **separate DB table**; provision persists runner identity + key there | D19 | Provision | Catalogue row exists; not env-only | unit + inspection |
| REQ-41 | Before twin/initiative start, Gateflow checks the DB catalogue for the effective runner and uses only that row’s credential; process-global `CURSOR_API_KEY` must not authorize a product run | OQ-4 | Start | Catalogue used; env fallback does not succeed | unit + verify |
| REQ-42 | Each Programme may store default runner + model per lane (spec, implement, closeout, initiative); if the start API omits runner and/or model, the lane default applies; caller-supplied values win; if both omit and no default → reject; defaults must themselves be provisioned catalogue runners | D19, OQ-4 | Start / set-defaults | Defaults applied correctly; reject when unresolved | unit + verify |
| REQ-43 | Invalid login is refused with a named outcome and issues no JWT (surfaces REQ-03 in US-1) | Fail-closed | Bad login | 0 token | unit + verify |
| REQ-44 | Attach `tenant_admin` to an unknown programme is rejected; 0 attach | Fail-closed | Bad attach | Named reason | unit + verify |
| REQ-45 | Provisioning a platform agent with blank or missing key is rejected | Fail-closed | Bad provision | 0 usable credential | unit + verify |
| REQ-46 | Wipe while any run is in flight for that programme is rejected; 0 wipe | Fail-closed | Wipe mid-run | Named reason; rows unchanged | unit + verify |
| REQ-47 | Re-seed of the same `platform_admin` is idempotent; re-attach of the same `tenant_admin` identity to the same programme is idempotent; multiple distinct `tenant_admin` identities on one programme are allowed | Efficiency | Seed / attach | Named idempotent behavior | unit + verify |

**Implementation notes (non-normative):** Activating `AuthMiddleware` by shrinking `public_paths` and replacing `Depends(verify_programme_service_token)` / `verify_tenant_bearer_token` with role-aware JWT dependencies is the natural as-built seam — not a second auth stack. Programme PAT storage moves off the 012 tenant column onto the new Programme entity. Platform agent credentials live in a dedicated catalogue table; process-global `CURSOR_API_KEY` is legacy and must not remain a product authorization path at exit (REQ-41). Module names are design detail.

---

## 4. Technical Specifications

### Architecture Overview *(design — not product vocabulary)*

```text
Seed / Login (CAP-01)
  → Gateflow-issued user JWT (platform_admin | tenant_admin)

Product edge (Appendix C)
  → JWT only → role + programme binding → handler
  → refuse programme service token | tenant bearer (W2)
  → delete dead doors (W3)

Validate-then-create (CAP-02)
  → receive PAT + meta + workspace
  → validate (PAT + meta repo-catalogue extract)
  → create Programme (PAT + workspace + repo catalogue)
  → Tenant is child attach target — never store agent keys on Programme

Attach tenant_admin + list (CAP-03)
  → platform_admin only

Platform agent DB catalogue (CAP-04)
  → separate table; platform_admin provisions Cursor (+ reserved slots)
  → programme per-lane default runner+model
  → start: resolve effective runner → load key from catalogue only
  → missing/unprovisioned → reject; env Cursor must not authorize

Operate (CAP-05)
  → tenant_admin JWT → repo lifecycle + twin/initiative
  → forge/git uses per-programme PAT
  → agent dispatch uses catalogue key for effective runner
  → webhooks stay signature-based

Fail closed + refuse (CAP-06) + delete + wipe (CAP-07) + prove (CAP-08)
```

### Integration Points

| System | Use |
|--------|-----|
| Gateflow JWT middleware / settings (existing, dormant on product paths) | Activated as the product edge |
| Programme meta repo (prayog-meta) | Read during validate-then-create for repo catalogue extract |
| Programme entity (new) | Per-programme PAT, workspace, lane defaults |
| Tenant (child) | `tenant_admin` attach / repo lifecycle scope |
| Platform agent catalogue table (new) | Runtime credential for effective runner |
| GitHub webhooks | Unchanged signature verification |
| prayog-meta vision/ADRs | Supporting doc updates for JWT-only edge + secret split + G3/ADR supersession |
| gateflow-ops | Not built; inherits callable auth story later |

### Security & Privacy

- **Product edge:** user JWT only. Shared-secret API doors refused in W2 and deleted in W3 (D1/D2/D3). Dual-auth is a failed exit; no interim dual-auth window. `(Source: User-confirmed)`
- **Runtime secrets stay runtime secrets:** GitHub PAT and agent keys are never the caller’s Bearer (D4).
- **Plaintext storage** of programme PAT and platform agent keys is an explicit, accepted risk (same class as INIT-GATEFLOW-012 G1).
- **Platform agent blast radius:** a provisioned Cursor key is shared across programmes that use it. Accepted and named; only `platform_admin` may provision.
- **Programme PAT blast radius:** still wide within one programme (same class as 012); explicit.
- **Lab wipe:** old tenant bearers and programme-token-era rows are destroyed rather than migrated.
- **Env Cursor:** not a product authorization path at exit (REQ-41).
- **Webhooks:** remain signature-based; not user-JWT.

### AI / agent evaluation

Not a new model product. Quality bar is contract/behavior correctness:

- Unit tests for role matrix, Programme validate-then-create, catalogue provision, lane-default resolution, env non-authorization, and negative auth paths.
- Verify / live-check scripts proving: seed/login JWT happy paths; `platform_admin` onboard + provision; `tenant_admin` repo + twin with effective agent; refusal of programme token / tenant bearer / open register.

---

## 5. Risks & Roadmap

### Delivery waves (product-normative)

| Wave | Intent | Exit REQs |
|------|--------|-----------|
| **W0** | Seed `platform_admin` + user JWT mint/login edge | REQ-01–REQ-07, REQ-43 |
| **W1** | Validate-then-create Programme + `tenant_admin` + DB agent catalogue + lane defaults | REQ-08–REQ-22, REQ-40–REQ-42, REQ-44–REQ-45, REQ-47 |
| **W2** | Cut over repo lifecycle + twin/initiative under JWT; **refuse** old doors (no dual-auth) | REQ-23–REQ-33 |
| **W3** | Dead-door deletion + wipe cutover | REQ-34–REQ-35, REQ-46 |
| **W4** | Prove absence + teaching-surface rewrite + meta vision/ADR supersession | REQ-36–REQ-39 |

### Technical risks

| Risk | Mitigation |
|------|------------|
| Dual-auth sneaks back (“JWT + keep programme token for scripts”) | D1/D2 + REQ-32/33 in W2 + exit verify negatives |
| Lab automation still sends programme API token | Debt purge + REQ-37 teaching/verify rewrite |
| Platform agent key blast radius across programmes | Named accepted risk; only `platform_admin` provisions |
| Programme PAT still wide within one programme | Same class as 012; explicit |
| Activating JWT middleware breaks health/webhook/internal paths | Keep non-product paths correctly allowlisted; OQ-6; Appendix C |
| Wipe surprises operators who expected migrate | OQ-3 locked wipe; cutover runbook in W3/W4 |
| Env-only `CURSOR_API_KEY` remains as a silent second path | REQ-41 + REQ-38; verify negatives |

### Phased rollout

- **MVP (this INIT, W0–W4):** JWT-only product edge; Programme validate-then-create; per-programme PAT; platform DB Cursor catalogue with choose-or-default-at-run; old doors refused then deleted and proven absent.
- **Later, explicit follow-ups:** ops/login UI; GitHub App runtime per programme; Claude Code / OpenCode runner implementations; encryption/secrets manager; full IdP/SSO if ever needed.

---

## 6. Locked decisions reference

### Carried forward from outline (D1–D19)

| ID | Decision |
|----|----------|
| D1 | Product APIs: Gateflow **user JWT only** — no dual-auth with programme service token |
| D2 | Removing global programme-token auth is **in-band**; JWT while old token works = failed exit |
| D3 | Open register + tenant bearer removed in this INIT |
| D4 | GitHub credentials are **runtime-only** — never API Bearer |
| D5 | Webhooks stay signature-based |
| D6 | Teaching surfaces rewritten in this delivery |
| D7 | No ops/login UI this INIT |
| D8 | Vision/ADRs superseded to match JWT-only product edge |
| D9 | New INIT on top of 012/013 behaviors |
| D10 | **platform_admin** seeded by script; receives programme details; onboards programme; then onboards **tenant_admin** |
| D11 | platform_admin may **list** programmes; does **not** onboard repos or run the twin |
| D12 | Product edge fail closed |
| D13 | Programme onboard is **validate-then-create** (clone meta, extract catalogue, then create) |
| D14 | **Programme** plaintext secrets = **GitHub only** (PAT required day one; App materials later, still per programme) — never agent keys on the programme |
| D15 | **Workspace root** set when onboarding the programme |
| D16 | Auth debt purge + prove absence are peer capabilities |
| D17 | Only **one** programme role: **tenant_admin** |
| D18 | GitHub credentials are **per programme** (programme provides PAT at onboard; platform_admin stores it linked to that programme) |
| D19 | **Code agents are platform-level**: platform_admin provisions the agent catalogue; tenant_admin **chooses** (or uses programme lane default) when running an initiative/twin — not at programme create |

### Resolved this Draft PRD (outline OQ-1–OQ-4 + discovery + review)

| ID | Resolution |
|----|------------|
| OQ-1 | Seed script mints JWTs **and** login API returns JWT (no UI) |
| OQ-2 | GitHub App = storage shape only; PAT required; App unused at runtime this INIT |
| OQ-3 | Wipe existing 012/013 lab tenants; re-onboard via new path |
| OQ-4 | Effective runner required (caller or per-lane programme default); DB catalogue provisioned; env Cursor not product path |
| Discovery #1 | Exit proof = verify / live-check scripts |
| Discovery #6 | No hard deadline / cross-INIT ship constraint |
| Entity model | **New Programme table**; Tenant child for `tenant_admin` `(Source: User-confirmed)` |
| Dual-auth window | None — refuse old doors in **W2**; delete in **W3** `(Source: User-confirmed)` |
| 012 G3 | **Superseded** — JWT is the product edge; tenant bearer removed |
| Lane defaults | Per-programme default runner+model for spec / implement / closeout / initiative; API omit → default |

---

## 7. Next steps

1. Impact map → programme sign-off → gateflow implementation waves per §5.
2. Do **not** implement Gateflow code from this Draft PRD alone — follow the full detailed-design process (spec → feasibility → technical review → plan → waves).
3. Schedule prayog-meta vision/ADR supersession (REQ-39), including ADR-005/011 follow-on, in the same delivery window as W4 teaching-surface rewrite.
4. Re-run `validate-requirements` in incremental mode against `prd/reports/Validation-Report-INIT-GATEFLOW-014.md`.

---

## Appendix A — Data model changes (illustrative — engineering owns exact schema)

| Change | Notes |
|--------|-------|
| User identity (`platform_admin` / `tenant_admin`) | Seeded + login-able; JWT subject/role claims |
| **Programme** table (new) | Workspace root; per-programme GitHub PAT (plaintext); optional reserved App fields; per-lane default runner+model |
| **Tenant** as child / attach target | `tenant_admin` binding; repo lifecycle scope — not the PAT owner after this INIT |
| Remove / stop minting tenant bearer as product auth | Wipe lab rows (OQ-3) |
| **Platform agent catalogue** table (new) | Cursor provisioned now; slots for other runners; keys plaintext at platform scope |
| No agent keys on Programme | Enforce on write |

## Appendix B — Target capability surface (illustrative — engineering owns exact routes)

| Capability | Notes |
|------------|-------|
| Seed `platform_admin` | Script; CAP-01 |
| Login → JWT | API only, no UI; CAP-01 |
| Validate-then-create Programme | `platform_admin`; CAP-02 |
| Attach `tenant_admin` | `platform_admin`; CAP-03 |
| List programmes | `platform_admin`; CAP-03 |
| Provision platform agent (DB catalogue) | `platform_admin`; CAP-04 |
| Set/read programme per-lane runner+model defaults | `platform_admin`; CAP-04 |
| Repo lifecycle + twin/initiative under JWT | `tenant_admin`; CAP-05; effective agent at start |
| *(removed)* Programme service token product auth | CAP-06 refuse / CAP-07 delete |
| *(removed)* Tenant bearer product auth | CAP-06 refuse / CAP-07 delete |
| *(removed)* Open tenant register | CAP-07 |

Exact routes, request/response shapes, and error body fields are engineering’s to design in the next stage — not fixed here.

## Appendix C — Product API families in scope (JWT + refuse matrix)

Families that must accept Gateflow-issued user JWT (role-gated) and must refuse programme service token / tenant bearer once the JWT edge is live:

| Family | Notes |
|--------|-------|
| Waves (twin / lane starts) | Including implement / spec / closeout-style starts |
| Runs | Status / list / forge authorize as applicable |
| Board | Ticket mutations/reads previously programme-token gated |
| Checkpoints | Previously programme-token gated |
| Initiatives | Reads/starts previously programme-token gated |
| Metrics | Previously programme-token gated |
| Repo connect / choose / onboard / deboard | Previously tenant-bearer gated |
| Programme onboard / list / attach / agent provision / lane defaults | `platform_admin` JWT |

**Excluded from user-JWT product edge (unchanged trust model):**

| Path class | Auth |
|------------|------|
| GitHub webhooks | Signature verification only |
| Health | Non-product |
| Internal | Non-product |

Exact mounts remain engineering design (OQ-6); this appendix is the product refuse/accept matrix.

---

## Exit gate

> **Exit:** User JWT only on product APIs (Appendix C); Programme entity with per-programme GitHub PAT (no agent keys on programme); platform **DB** agent catalogue with effective runner (caller or per-lane default); env Cursor not a product path; old programme service token, tenant bearer, and open register refused then gone; dual-auth or “delete later” is a **failed** exit. Verify scripts prove the happy path and the refusals.

---

## References

- Outline: [INIT-GATEFLOW-014-outline](./INIT-GATEFLOW-014-outline.md)
- Vision: [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md)
- Predecessors: [INIT-GATEFLOW-012](./INIT-GATEFLOW-012.md) (G3 **superseded**), [INIT-GATEFLOW-013](./INIT-GATEFLOW-013.md)
- Resolution: [Resolution-INIT-GATEFLOW-014](./reports/Resolution-INIT-GATEFLOW-014.md)
- Code evidence (verified this session against local `gateflow` / fleet graph): `src/app.py` (`public_paths`), `src/common/auth/middleware.py`, `src/api/v1/programme_token.py`, `src/api/v1/tenant_token.py`, `src/api/v1/tenant_routes.py`, `src/configs/cursor_agent_settings.py`, `src/configs/jwt_settings.py`, `src/database/postgres/schema/tenant_schema.py`
