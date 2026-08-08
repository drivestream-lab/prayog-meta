# INIT-GATEFLOW-012 — Tenant registry and workspace/branch lifecycle

**Status:** draft PRD · **Author:** programme PM · **Date:** 2026-08-07  
**Outline:** [INIT-GATEFLOW-012-outline](./INIT-GATEFLOW-012-outline.md)  
**Vision:** [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md)  
**Component:** GATEFLOW · **Type:** platform / execution substrate  
**Predecessor:** INIT-GATEFLOW-010 (engineering-lane pin tip executor parity — this INIT extends that executor's workspace assumptions to multi-repo, multi-tenant, cloud execution)  
**Related, not blocking:** INIT-GATEFLOW-011 (Day-1 visibility) — independent read-only layer, no dependency either direction

> **Draft PRD** — Locked decisions D1–D10 (outline) carried forward; G1–G5
> added below from Draft PRD review, including three decisions locked
> directly with the programme PM this session (credential storage, git
> credential scope, auth boundary mechanism — see G1–G3). Primary delivery =
> **gateflow only**; `gateflow-ops` (no UI this INIT) and `prayog-skills` (a
> genuine **contract change**, not consume-only — see §4) are supporting
> repos. This INIT reverses one piece of ADR-010 (caller-supplied
> `workspace_path`) by design (D1) — that reversal is scoped to Tenant-
> registered repos only; unregistered/legacy callers are unaffected (§2
> Non-Goals).
>
> **As-built baseline (Source: verified against gateflow code this
> session).** Findings below changed the shape of this Draft PRD versus the
> outline and are cited inline with file paths. Headline findings: (1)
> `ImplementWaveStartRequest.workspace_path` is already `Optional[str]` — the
> **only** wave-start request that skips `_require_existing_directory`
> (`src/business_services/wave_start_service.py`). When omitted, Gateflow
> does not fail closed: `RunOrchestrator.process_job` silently falls back to
> **Gateflow's own process working directory**
> (`workspace_path = context.workspace_path or str(Path.cwd())`,
> `src/business_services/run_orchestrator.py`) as the coding agent's
> workspace — a real, already-**active** (and dangerous) behavior today, not
> a passive gap; (2) `ForgeClient` has no clone/checkout/delete-branch method
> today (`src/infra_services/forge_client.py`) — confirms D1's premise; (3) a
> JWT `AuthMiddleware` with `user_id`/`tenant_id`/`owner_id` claims is wired
> into `app.py` but **every** real `/api/v1/*` route is listed in
> `public_paths`, so it gates nothing today, and its model docstring/fields
> (`device_id`, `credential_id`, `catalog_id`) trace to an unrelated product
> template — confirmed dead for this purpose (G3); this INIT's new "Tenant"
> entity name is chosen independently and does **not** reuse or activate
> this dormant field; (4) `LaunchpadClient` is a real, already-wired
> **stub** (`sync_harness` — "W1: log + no-op") that only checks the path
> exists — already invoked inside `RunOrchestrator.process_job` (the
> **asynchronous worker phase**, after a run is enqueued) — the natural
> extension point for CAP-04, not new infrastructure; (5) the real
> concurrency check for the API lane-start route lives in `WaveStartService`,
> which calls `RunRepository.find_active_run` **directly** — **not** via
> `TriggerRouter`, a separate class serving only the webhook/legacy
> trigger-authorization ingress — and this check already scopes active-run
> conflicts to `org`+`repo` **plus** a narrower key (PR/issue/initiative+wave),
> confirming two different waves on the *same* repo can run concurrently
> today, which is exactly the gap D10/CAP-05 closes; (6) the "board" today is
> an org GitHub Project v2 (`project_number`/`project_owner`), always
> caller-supplied per call (`src/models/board_models.py`) — resolving OQ-3 as
> a default-value problem, not a new board concept.

---

## Document control

| Field | Value |
|-------|-------|
| Initiative ID | INIT-GATEFLOW-012 |
| Programme | prayog |
| Primary repos | drivestream-lab/gateflow (implementation); drivestream-lab/prayog-skills (new forge action types + pin shape — **contract change**) |
| Explicitly **not** touched this INIT | gateflow-ops (onboarding UI is a follow-on initiative); JWT `AuthMiddleware`/`tenant_id` scaffolding (dormant, unrelated — parked, not repurposed — G3) |
| Target users | Tenant admin/operator (registers tenants, credentials, repos); developer/engineer running waves (benefits transparently) |
| Credential storage | **Plaintext Postgres column** on the new Tenant table — no encryption/secrets-manager this INIT; explicit, PM-accepted risk, not a security recommendation (**G1**, resolves OQ-1) |
| Git credential scope | **Single per-tenant PAT** authenticates both `ForgeClient` platform API calls **and** local git clone/fetch — no separate deploy key this INIT (**G2**, resolves OQ-4) |
| User↔Tenant auth | **New** tenant-scoped bearer-token mechanism, extending the existing `verify_programme_service_token` shared-secret pattern (now per-tenant instead of one global token) — the dormant JWT middleware is explicitly parked, not reused (**G3**, resolves the auth half of D9) |
| Depends on | INIT-GATEFLOW-010 (eng lanes proven and human-gated correctly) |
| As-built baseline | 2026-08-07 · findings verified directly against local `gateflow` checkout this session (see banner above) |

---

## 1. Executive Summary

### Problem Statement

Gateflow's contract (ADR-010) assumes someone else already did the git setup
— cloned the repo, checked out the ref, handed Gateflow an absolute folder.
Gateflow itself never runs `git clone`/`checkout`/`branch`; it trusts the
folder is already correct. This is not hypothetical: `ImplementWaveStartRequest.workspace_path`
is already optional in code today, and is the one wave-start request that
skips the existing-directory check every other lane enforces. When omitted,
Gateflow does not fail closed — `RunOrchestrator.process_job` silently falls
back to its own process's current working directory
(`workspace_path = context.workspace_path or str(Path.cwd())`) as the
coding agent's workspace: an active, already-present behavior today, not a
passive gap. *(Source: verified — `src/models/wave_start_models.py`,
`src/business_services/wave_start_service.py`,
`src/business_services/run_orchestrator.py`.)*

Two consequences block running Gateflow on cloud, across many repos and
tenants:

1. **No multi-tenant, multi-repo story.** `GithubSettings` is one global
   env-var credential for the whole service. There is no way to register a
   second tenant with its own repos and credential without touching
   global config — nothing in code answers "which repo, whose credentials,
   where's the workspace." *(Source: `src/configs/github_settings.py`.)*
2. **Branches never get cleaned up.** `ForgeClient` has `ensure_branch_from_base`
   but no delete-branch method at all — every wave/spec branch ever created
   still exists, merged or not. *(Source: `src/infra_services/forge_client.py`
   method inventory, verified this session.)*

### Proposed Solution

Give Gateflow the ability to behave like an independent developer toward any
repo it is told to operate on, scoped by a new **Tenant registry**:

```text
Tenant (PAT + repo list + workspace root + board default)
  └─ Repo clone (first use) / refresh (subsequent use) — new WorkspacePrep behavior
      └─ Branch create-or-reuse (new wave → fork develop; continuation → reuse)
          └─ Harness-readiness gate (before Gateflow operates on a repo)
              └─ Repo-level sequencing (NO_CONCURRENT_RUN broadened org+repo → repo-scoped)
                  └─ Branch purge (built, pin-declared — dormant, not wired live)
```

Tenant onboarding happens via **direct API call** this INIT — no UI. The
one required *live* change is the `workflow.yaml`/`delivery-contract.yaml`
shape so Gateflow's pin-reading code **can** honor the new node/action
vocabulary once activated later (D5) — not a second spec pass on
`prayog-skills`.

### Success Criteria

| KPI | Target | Measurement |
|-----|--------|-------------|
| **Tenant self-service onboarding** | A tenant admin registers a tenant (PAT, repo list, workspace root, board default) via one API call, 0 global config edits | Unit + manual UAT |
| **Workspace self-sufficiency** | Implement-start with `workspace_path` omitted succeeds end to end for a Tenant-registered repo (first-use clone; subsequent refresh) — 0 "pre-clone by hand" steps, and the current `Path.cwd()` fallback is never reached | Unit + verify (fixture repo) |
| **Correct branch behavior** | New wave forks from live `develop` tip; continuation reuses the existing wave PR branch — both provably correct, 0 accidental re-forks or stray branches | Unit + verify (fresh vs. resumed run fixtures) |
| **Harness gate accuracy** | A repo without harness scaffolding is flagged non-ready before any coding hop dispatches; a harness-enabled repo is never falsely blocked | Unit + verify (positive/negative fixture repos) |
| **Repo-level exclusivity** | Two different initiatives/waves on the **same** repo cannot both hold an ACTIVE run; two initiatives on **different** repos run with 0 cross-repo blocking; the concurrency gate always runs before any shared-workspace mutation for that repo | Unit against `find_active_run` scoping change + verify |
| **Purge capability provably dormant** | `delete_branch` capability passes its own unit/verify suite; **0** live branch deletions occur via any default flow this INIT ships | Unit + verify + code guard (route/outcome-edge audit) |
| **Contract hygiene** | `prayog-skills` pin declares the new forge action / node shape; gateflow parses it with 0 BROKEN nodes; `authorization` remains `explicit` on every new mutate-capable node | Unit against remounted pin |

### Capability ↔ wave ↔ requirement map

| CAP | Wave | Capability | REQ |
|-----|------|------------|-----|
| CAP-01 | W0 | Tenant registry (data + API + credential + user attach + read/list) | REQ-01–REQ-09, REQ-32 |
| CAP-02 | W1 | Repo clone (first use) + refresh (subsequent use) | REQ-10–REQ-15 |
| CAP-03 | W2 | Branch create-or-reuse | REQ-16–REQ-19 |
| CAP-04 | W3 | Harness-readiness check | REQ-20–REQ-22 |
| CAP-05 | W4 | Repo-level sequencing | REQ-23–REQ-25 |
| CAP-06 | W5 | Branch purge (built, pin-declared, dormant) | REQ-26–REQ-29 |
| — | cross-cutting (lands once; see §3 note) | Pin/contract shape (prayog-skills) | REQ-30–REQ-31 |

---

## 2. User Experience & Functionality

### User Personas

| Persona | Role | Primary need |
|---------|------|---------------|
| **Tenant admin/operator** | Registers tenants, credentials, repos | Register a PAT + repo list once via API; never touch global env config for a second tenant |
| **Developer/engineer running waves** | Starts implement/spec/closeout lanes | Gateflow figures out fresh-vs-continuation on its own; branch list doesn't turn into a graveyard; no git plumbing to think about |

### User Stories & Acceptance Criteria

#### US-1 — Register a tenant without touching global config `(CAP-01)`

**As a** tenant admin, **I want** to register a tenant's PAT, repo
list, workspace root, and board default via one API call, **so that**
Gateflow can start working on any of those repos without a second env
deploy.

**Acceptance criteria:**

- [ ] `POST` tenant-registration accepts: `name`, `pat` (raw string — see
      G1), `repos[]` (org/repo pairs), `workspace_root` (absolute path),
      `board` default (`project_owner`, `project_number` — optional,
      overridable per-call same as today's `BoardTicketCreateRequest`
      precedent)
- [ ] Missing/malformed required fields → **400**, 0 rows written
- [ ] Success returns a `tenant_id` and the tenant's own bearer token
      (G3) — never echoes the PAT back in the response body
- [ ] Registering a second tenant requires **0** edits to `GithubSettings`
      or any other global env-backed `Settings` class
- [ ] The PAT is persisted in a dedicated Postgres column on the Tenant
      row, in plaintext (**G1** — explicit, accepted risk; not a security
      recommendation; see §4 Security & Privacy for the carried-forward risk
      note)

#### US-2 — Attach a user to a tenant `(CAP-01)`

**As a** tenant admin, **I want** to attach a developer/engineer to a
tenant, **so that** they can operate on any repo the tenant owns
without per-repo grants.

**Acceptance criteria:**

- [ ] Attach endpoint records a user identity (email or handle) against a
      `tenant_id` for audit/attribution
- [ ] A user attached to a tenant can call any lane-start/board/checkpoint
      route scoped to that tenant's registered repos; no finer-grained
      per-repo check exists (**D9** — deliberately simple)
- [ ] A user with **no** attachment to a tenant, presenting that
      tenant's token incorrectly or omitting it, is rejected **401**
      before any lane logic runs — same enforcement point/shape as today's
      `verify_programme_service_token` (G3), now keyed per-tenant instead
      of one global secret

#### US-3 — Eager credential verification at registration `(CAP-01)`

**As a** tenant admin, **I want** Gateflow to verify the PAT can read a
newly registered repo at registration time, **so that** a typo or
insufficient scope surfaces immediately, not on the first real wave start.

**Acceptance criteria:**

- [ ] Registration calls a read-only GitHub check (e.g. repo metadata fetch)
      per registered repo using the submitted PAT before the Tenant row is
      committed
- [ ] Any repo failing that check → **422**, 0 rows written, response names
      exactly which repo(s) failed and why (not a single boolean) — same
      itemized-failure shape as INIT-GATEFLOW-011's checkpoint error
      convention
- [ ] This resolves **OQ-2** — verification is eager, not deferred to first
      use (PM judgment call, this Draft PRD; **G4**)

#### US-4 — Workspace self-sufficiency on first use `(CAP-02)`

**As a** developer, **I want** Gateflow to clone a Tenant-registered repo
itself the first time it's asked to work on it, **so that** I never
hand-prepare a folder.

**Acceptance criteria:**

- [ ] When `ImplementWaveStartRequest.workspace_path` is omitted **and** the
      request's `org`/`repo` resolve to a Tenant-registered repo, Gateflow
      clones that repo under the Tenant's registered `workspace_root`, at a
      deterministic, discoverable path (exact on-disk scheme is
      non-normative — see §4 implementation notes), before dispatching
      `pre-implement` — replacing today's `Path.cwd()` fallback
      (`RunOrchestrator.process_job`)
- [ ] Clone uses the Tenant's stored PAT for git auth (**G2** — same
      credential as `ForgeClient`, no separate deploy key this INIT)
- [ ] If `workspace_path` is explicitly supplied (legacy/unregistered-repo
      path), Gateflow's existing caller-supplied-path behavior is unchanged
      — this capability is additive, not a breaking change to today's
      contract (Non-Goal — see below)
- [ ] Clone failure (auth, network, repo not found) → **422** on the
      lane-start call, 0 enqueue; reason names the repo and the underlying
      git/HTTP failure class, not a generic message

#### US-5 — Workspace self-sufficiency on repeat use `(CAP-02)`

**As a** developer, **I want** Gateflow to refresh (not re-clone) a repo it
has already cloned, **so that** repeat waves on the same repo start from a
current base without repeated full clones.

**Acceptance criteria:**

- [ ] If the deterministic workspace path already exists and is a valid git
      checkout of the expected remote, Gateflow fetches in place (no
      re-clone)
- [ ] If the path exists but is **not** a valid checkout of the expected
      remote (corrupted / wrong remote), Gateflow fails closed with a named
      reason rather than silently operating on the wrong tree — **422**, 0
      enqueue
- [ ] Refresh happens before branch resolution (CAP-03), so branch
      create-or-reuse always sees a current `develop` tip

#### US-6 — New wave forks from live develop `(CAP-03)`

**As a** developer, **I want** a brand-new wave to fork from the target
repo's current `develop` tip, **so that** it matches what a careful human
developer would do.

**Acceptance criteria:**

- [ ] "New wave" = no existing run/PR found for this `initiative_id`+`wave_id`
      (reuses `RunRepository` lookup shape already used by
      `find_active_run`/dual-identity resolution)
- [ ] Fork point = target repo's current `develop` tip at fork time (not a
      cached/stale SHA)
- [ ] Resulting head branch name follows the existing deterministic
      convention (`build_wave_head_branch` — `feature/{INIT}-{wn}-{slug}`,
      `src/models/pr_branch_naming.py`) — this INIT does not invent a second
      naming scheme
- [ ] Branch creation reuses `ForgeClient.ensure_branch_from_base` — no new
      git-ref-creation code path
- [ ] If the target repo has no `develop` branch, or
      `ForgeClient.ensure_branch_from_base` fails (network/permission/API
      error), the wave-start call fails with **422**, naming the underlying
      failure — no partial branch is left behind

#### US-7 — Continuation reuses the existing branch `(CAP-03)`

**As a** developer, **I want** re-entering an existing wave (e.g. closeout
after implement, or a retry) to reuse that wave's existing PR branch, **so
that** Gateflow never accidentally re-forks and orphans work.

**Acceptance criteria:**

- [ ] "Continuation" = an existing run/PR is found for this
      `initiative_id`+`wave_id` (same detection as US-6, opposite branch)
- [ ] Gateflow resolves the existing head ref via the same
      `branch_slug_from_head_ref` precedent already used for closeout intake
      (`src/models/pr_branch_naming.py`) — no re-fork, no new branch created
- [ ] Continuation on a repo Gateflow has not yet cloned locally still
      succeeds — CAP-02's clone-then-checkout-existing-branch path is
      exercised, not just the new-wave path
- [ ] If the previously-recorded head ref no longer exists on the remote
      (PR closed, branch deleted out-of-band), continuation fails with
      **422** naming "continuation branch not found on remote" rather than
      silently re-forking or crashing

#### US-8 — Repo flagged not harness-ready is never silently operated on `(CAP-04)`

**As a** developer, **I want** Gateflow to check a newly registered repo has
the harness installed before it starts working in it, **so that** Gateflow
never produces work that doesn't match the repo's actual conventions.

**Acceptance criteria:**

- [ ] Check runs after clone/refresh (CAP-02), before any coding hop
      dispatch, for any repo not yet marked harness-verified for this
      Tenant
- [ ] Reuses/extends `LaunchpadClient` — today a real, wired **stub**
      (`sync_harness`: "W1: log + no-op", checks only that the path exists,
      already invoked inside `RunOrchestrator.process_job`,
      `src/infra_services/launchpad_client.py`) — this INIT makes the check
      real (e.g. `.harness-pin.yaml` / `.harness/` presence), not a new infra
      slot
- [ ] Non-ready repo → lane-start **422**, reason names what's missing (e.g.
      "`.harness-pin.yaml` not found"), 0 enqueue
- [ ] A harness-ready repo is marked verified so repeat waves don't re-check
      on every single run (cache with an explicit re-check path, not an
      unconditional check every call)

#### US-9 — One active path per repo `(CAP-05)`

**As a** developer, **I want** the "no concurrent run" rule to block a
second initiative/wave on the **same repo**, not just the same PR/issue/wave,
**so that** two waves never collide on one shared working directory.

**Acceptance criteria:**

- [ ] `RunRepository.find_active_run`, called directly by `WaveStartService`
      (not `TriggerRouter`, which serves only the webhook/legacy ingress),
      rejects a new run whenever an ACTIVE run already exists for the same
      `org`+`repo`, **regardless** of whether
      `initiative_id`/`wave_id`/`pr_number`/`issue_number` match —
      broadened from today's narrower key match
      (`src/database/postgres/repository/run_store_repository.py`,
      `src/business_services/wave_start_service.py`)
- [ ] This check executes **synchronously** in `WaveStartService`, before
      the run is enqueued — strictly before CAP-02 (clone/refresh) or
      CAP-03 (branch resolve) ever touch the shared per-repo workspace,
      both of which execute later, inside `RunOrchestrator.process_job`
      (the asynchronous worker phase) `(Source: User-confirmed)`
- [ ] Two initiatives on **two different** repos are never blocked by each
      other — this is a scoping change, not a new lock/queue mechanism (D10)
- [ ] Rejected start → **PreconditionFailure** with `precondition_id ==
      NO_CONCURRENT_RUN` (existing enum value, `WavePreconditionIdType`), not
      a new failure code
- [ ] No new worktree/isolation mechanism is introduced — one shared working
      directory per repo remains sufficient (direct consequence of D10)

#### US-10 — Branch purge exists, tested, and dormant `(CAP-06)`

**As a** tech lead, **I want** the branch-delete capability to exist and be
provably correct without deleting anything live this INIT, **so that**
reopening this boundary is a considered, staged decision.

**Acceptance criteria:**

- [ ] `ForgeClient` gains a `delete_branch(owner, repo, branch)` method (does
      not exist today — confirmed by method inventory this session) using
      the existing `_git_ref_update_path` (`DELETE .../git/refs/heads/{branch}`)
      shape, not a new transport
- [ ] `prayog-skills` pin declares a new forge action type for branch delete
      (contract change — see REQ-30/REQ-31) with `authorization: explicit`
      (never `automated`) — even once activated later, a delete always
      requires an explicit authorize call, never an automatic hop (**G5**,
      resolves OQ-6)
- [ ] This INIT wires **0** live outcome edges to that action — no
      wave-close path in the default walker calls it; "dormant" is enforced
      structurally (no reachable edge), not by a runtime settings flag
      (**G5**)
- [ ] Unit + verify prove the method deletes the correct branch and fails
      closed on a missing/protected branch — proven in isolation, never
      exercised end-to-end via any live wave-close flow this INIT ships

### Non-Goals

| Non-goal | Why |
|----------|-----|
| `gateflow-ops` onboarding UI ("Mission Control") | Follow-on initiative, after this one ships the APIs it would call |
| GitHub App installation per tenant | PAT only this INIT (D8); App mode deferred |
| Per-repo ACL within a tenant | D9 — User ↔ Tenant is the only boundary this INIT builds |
| Activating branch purge live | D5/G5 — capability ships dormant; activation is a separate, later decision |
| Any new concurrency/locking mechanism (worktrees, per-wave isolation) | D10 — repo-level exclusivity is enough |
| Changing how merges happen | Merge stays human-only at `wave-signoff`, unchanged |
| Encrypting/rotating the stored PAT, or any secrets-manager integration | **G1** — explicit scope cut this INIT, not solved here; carried as a risk, not silently deferred |
| A separate git-workspace credential distinct from the tenant PAT | **G2** — single PAT serves both `ForgeClient` and local git this INIT |
| Repurposing the dormant JWT `AuthMiddleware`/`tenant_id` scaffolding | **G3** — unrelated to this INIT's Tenant model; left exactly as-is (still gates nothing, still in `public_paths`) |
| Breaking the existing caller-supplied `workspace_path` path for unregistered repos | CAP-02 is additive: explicit `workspace_path` still works unchanged (US-4) |
| Overlap with `INIT-GATEFLOW-004`'s onboarding "scorecard" categories ("Harness posture," "GitHub/Forge access") | Not resolved this INIT — CAP-04 (harness-readiness) and US-3 (eager PAT verify) may eventually back those scorecard categories, or remain independent; flagged for cross-initiative coordination, not decided here |

### Assumptions

| ID | Assumption | Status | Dependent REQs |
|----|------------|--------|----------------|
| A1 | A Tenant's registered repos are on the same GitHub host reachable by `ForgeClient` today (`api_base_url`) — no new GitHub Enterprise/self-hosted transport this INIT | Confirmed by scope | REQ-01, REQ-10 |
| A2 | Deployment target has a **persistent disk** between runs, so fetch-in-place (CAP-02 refresh) is valid without a shared/remote cache — resolves **OQ-5** per the outline's own lean ("not blocking") | Assumed, not yet infra-confirmed | REQ-11, REQ-13 |
| A3 | `LaunchpadClient.sync_harness`'s current no-op stub is the intended integration point for real harness verification (CAP-04), not a placeholder for a different mechanism | Confirmed by code shape (already injected into the DI container, already invoked inside `RunOrchestrator.process_job`) | REQ-20–REQ-22 |
| A4 | `RunRepository.find_active_run`'s existing `org`+`repo` WHERE clause (already present before the narrower key filter) means broadening to repo-scope is a query change, not a new table/index | Confirmed by code (`run_store_repository.py`) | REQ-23 |
| A5 | The Tenant's default board (`project_owner`/`project_number`) follows the exact same caller-override precedent `BoardTicketCreateRequest` already establishes — no new board data model | Confirmed by code (`src/models/board_models.py`) | REQ-08 |

### Error table (product-normative)

| Situation | HTTP / run | Side effects |
|-----------|------------|---------------|
| Tenant registration missing/malformed required fields | **400** | 0 rows written |
| Tenant registration: PAT fails eager per-repo verification (US-3) | **422** | 0 rows written; itemized per-repo failure |
| Lane-start for a user not attached to the target tenant | **401** | 0 enqueue |
| Implement-start, `workspace_path` omitted, repo not Tenant-registered | **422** | 0 enqueue — this INIT does not guess a workspace for unregistered repos |
| Clone fails (auth/network/not-found) on first use | **422** | 0 enqueue; reason names repo + failure class |
| Refresh finds an existing path that is not a valid checkout of the expected remote | **422** | 0 enqueue; fail closed, never silently overwrite |
| Branch fork from `develop` fails (missing branch, API/network/permission error) | **422** | 0 enqueue; reason names the underlying failure; no partial branch left behind |
| Continuation's recorded head ref no longer exists on the remote (PR closed / branch deleted out-of-band) | **422** | 0 enqueue; reason: "continuation branch not found on remote" |
| Harness-readiness check fails for a newly registered/cloned repo | **422** | 0 enqueue; reason names missing harness artifact |
| Two starts target the same `org`+`repo` while one is ACTIVE | **PreconditionFailure** (`NO_CONCURRENT_RUN`) | 0 enqueue for the second request |
| `delete_branch` called directly (unit/verify only — no live route this INIT) on a missing/protected branch | fail closed | No partial state; error names the branch |

Exact problem+json / OpenAPI error body field names → **OQ-01** (deferred to
gateflow OpenAPI pass, same deferral pattern as INIT-GATEFLOW-010/011).

### Open questions

| ID | Open question | Status |
|----|----------------|--------|
| OQ-01 | Exact problem+json / OpenAPI error body field names for the new Tenant/workspace/branch routes | Open — deferred to gateflow OpenAPI pass |
| OQ-7 *(new)* | Exact eager-verification GitHub call for US-3 (repo metadata read vs. a scoped permissions probe) — engineering routing, not a product decision | Open — routes to gateflow implementation plan |
| OQ-8 *(new)* | Whether the deterministic clone path (`{workspace_root}/{org}/{repo}`) needs per-wave subpaths to avoid collision with a *future* worktree-isolation initiative, given D10 explicitly rules out isolation **this** INIT | Open — flagged for whoever revisits D10 later, not blocking this INIT |

**Resolved from outline (not carried above):** OQ-1 → G1, OQ-2 → G4,
OQ-3 → REQ-08/A5 (§3), OQ-4 → G2, OQ-5 → A2, OQ-6 → G5. All six of the
outline's original open questions are accounted for; none were silently
dropped.

---

## 3. Functional requirements

### Capabilities

| ID | Capability | Covers |
|----|------------|--------|
| CAP-01 | Tenant registry (data model + API + credential + user attach + read/list) | REQ-01–REQ-09, REQ-32 |
| CAP-02 | Repo clone (first use) + refresh (subsequent use) | REQ-10–REQ-15 |
| CAP-03 | Branch create-or-reuse | REQ-16–REQ-19 |
| CAP-04 | Harness-readiness check | REQ-20–REQ-22 |
| CAP-05 | Repo-level sequencing | REQ-23–REQ-25 |
| CAP-06 | Branch purge (built, pin-declared, dormant) | REQ-26–REQ-29 |
| — | Pin/contract shape (prayog-skills) | REQ-30–REQ-31 |

**Note on the cross-cutting row:** REQ-30 covers three separable pin/contract
additions — a Tenant-aware workspace-prep node shape (backs W1), a branch
create-or-reuse node shape (backs W2), and the branch-delete forge action
type (backs W5). The `prayog-skills` contract PR lands **once**, covering
all three shapes together; it is not a W5-only deliverable even though §5's
wave table shows its exit REQs at W5 (where the *last* of the three shapes —
branch-delete — is proven). W1 and W2 exit implicitly depend on that same
contract PR landing early enough to unblock them.

### Requirements

| ID | Requirement | Outline / gap | Condition | Observable result | Evidence |
|----|-------------|----------------|-----------|--------------------|----------|
| REQ-01 | Tenant registration accepts `name`, `pat`, `repos[]`, `workspace_root`, optional `board` default; rejects missing/malformed → **400**, 0 rows | D7 | Registration call | 400 + 0 rows on bad input; 201/200 + `tenant_id` on success | unit |
| REQ-02 | PAT persisted in a dedicated Postgres column, plaintext, never echoed in any response body | D8, **G1** | Registration success | Column present in schema; response body has no `pat` field | unit + inspection |
| REQ-03 | Registration issues a tenant-scoped bearer token distinct from the global `PROGRAMME_SERVICE_TOKEN` | D9, **G3** | Registration success | Token returned once; verifiable on subsequent calls | unit |
| REQ-04 | User-attach endpoint records identity against `tenant_id`; a user with no attachment presenting the wrong/no token → **401** before lane logic | D9 | Attach + subsequent call | 401 on mismatch; success on attached user | unit + verify |
| REQ-05 | No per-repo ACL exists within a tenant — any attached user may act on any of the tenant's registered repos | D9 | Any attached-user call | No repo-scoped permission check exists in code | inspection |
| REQ-06 | Registration eagerly verifies PAT read access per registered repo before commit; failing repo(s) → **422**, 0 rows, itemized per-repo reason | **G4** (resolves OQ-2) | Registration call | 422 with named failing repo(s) on bad PAT/scope | unit + verify |
| REQ-07 | `workspace_root` is validated as an absolute path (reuses the existing `_require_absolute_workspaces`-style validator pattern from `wave_start_models.py`) | Consistency | Registration call | Non-absolute path → 400 | unit |
| REQ-08 | Tenant's default `board` (`project_owner`/`project_number`) follows the exact override precedent `BoardTicketCreateRequest` already establishes | A5 (resolves OQ-3) | Board-touching call with no explicit override | Tenant default applied; explicit per-call value still wins | unit |
| REQ-09 | Registering a second tenant requires 0 edits to `GithubSettings` or any other global env-backed Settings class | D7 | Second registration | No deploy/env change required | inspection + manual UAT |
| REQ-10 | Implement-start with `workspace_path` omitted **and** `org`/`repo` Tenant-registered resolves a deterministic, discoverable workspace for the repo before `pre-implement` dispatch, replacing the current `Path.cwd()` fallback in `RunOrchestrator.process_job` (exact path scheme is non-normative — see §4) | D1 | Omitted `workspace_path`, registered repo | Workspace exists at a deterministic, discoverable path pre-dispatch; `Path.cwd()` fallback is never reached | unit + verify |
| REQ-11 | Clone/refresh authenticates git operations with the Tenant's stored PAT (**G2** — same credential as `ForgeClient`) | D8, **G2** | Any clone/refresh | Git auth uses Tenant PAT; no separate deploy-key path exists | unit + inspection |
| REQ-12 | Explicit `workspace_path` on any wave-start request is honored exactly as today — CAP-02 is additive, never overrides a caller-supplied path | Non-goal | `workspace_path` present | Behavior identical to pre-INIT-012 code path | unit (regression) |
| REQ-13 | If the deterministic path already exists and is a valid checkout of the expected remote, Gateflow fetches in place; no re-clone (depends on A2 — persistent disk; see §2 Assumptions) | D1 | Repeat use, valid existing checkout | Fetch executed; no `git clone` invoked | unit + verify |
| REQ-14 | If the deterministic path exists but is not a valid checkout of the expected remote, refresh fails closed (**422**), naming the mismatch | Fail-closed | Corrupted/mismatched path | 422; workspace untouched | unit + verify |
| REQ-15 | `workspace_path` omitted **and** repo not Tenant-registered → **422**, 0 enqueue (Gateflow does not guess a workspace) | Fail-closed | Unregistered repo, no path | 422; no clone attempted | unit + verify |
| REQ-16 | "New wave" (no existing run/PR for `initiative_id`+`wave_id`) forks the head branch from the target repo's **current** `develop` tip via `ForgeClient.ensure_branch_from_base` | D2 | New wave start | Head branch created from live `develop` SHA at fork time | unit + verify |
| REQ-17 | Head branch name uses the existing deterministic convention (`build_wave_head_branch`, `src/models/pr_branch_naming.py`) — no second naming scheme introduced | Consistency | New wave start | Branch name matches `feature/{INIT}-{wn}-{slug}` | unit |
| REQ-18 | "Continuation" (existing run/PR found for `initiative_id`+`wave_id`) resolves and reuses the existing head ref via `branch_slug_from_head_ref`; 0 new branches created | D2 | Continuation start | No `ensure_branch_from_base` create-path invoked; existing branch checked out | unit + verify |
| REQ-19 | Continuation succeeds even when Gateflow has not previously cloned the repo locally (CAP-02 clone-then-checkout-existing-branch composes with CAP-03) | Composition | Continuation on fresh workspace | Clone + checkout of existing branch both succeed in one flow | verify |
| REQ-20 | Harness-readiness check runs after clone/refresh, before any coding-hop dispatch, for any repo not yet marked harness-verified for the Tenant | D6 | Repo not yet verified | Check executes before `pre-implement`/`spec-draft` dispatch | unit + verify |
| REQ-21 | Check extends `LaunchpadClient.sync_harness` to a real check (e.g. `.harness-pin.yaml`/`.harness/` presence) rather than the current path-exists-only stub | A3, D6 | Any harness check | Non-ready repo (harness artifacts absent) fails; harness-enabled repo passes | unit + verify (positive + negative fixture) |
| REQ-22 | A harness-ready repo is cached as verified so repeat waves don't re-check every call; an explicit re-check path exists | Efficiency | Repeat wave on verified repo | No redundant check call; re-check available on demand | unit |
| REQ-23 | `find_active_run`, called directly by `WaveStartService` (not `TriggerRouter`, which serves only the webhook/legacy ingress), rejects a new run whenever an ACTIVE run exists for the same `org`+`repo`, regardless of PR/issue/initiative+wave match | D10 | Second start, same repo, different wave | `NO_CONCURRENT_RUN` failure — broadened from today's narrower key | unit + verify |
| REQ-24 | Two initiatives on two different repos are never blocked by each other under the broadened check | D10 | Two starts, different repos | Both authorized; no cross-repo blocking | unit + verify |
| REQ-25 | No new worktree/lock/isolation mechanism is introduced; one shared working directory per repo remains sufficient | D10 | Any repo | No new isolation infra present in code | inspection |
| REQ-26 | `ForgeClient.delete_branch(owner, repo, branch)` is added, using the existing `_git_ref_update_path` DELETE shape | D4 | Unit call | Branch deleted on success; correct API path/method used | unit |
| REQ-27 | `delete_branch` fails closed (raises, no partial state) on a missing or protected branch | Fail-closed | Missing/protected branch | Named error; no silent no-op | unit |
| REQ-28 | `prayog-skills` pin declares a branch-delete forge action type with `authorization: explicit` — never `automated` | D5, **G5** | Pin remount | Node parses; `authorization` value is `explicit` | unit (pin parse) |
| REQ-29 | 0 live outcome edges in the default walker route to the branch-delete action this INIT; capability is unreachable via any shipped default flow | D5, **G5** | Any wave-close walk this INIT ships | No call to `delete_branch` occurs in end-to-end verify | verify + code guard |
| REQ-30 | `prayog-skills` `workflow.yaml`/`delivery-contract.yaml` gains the new node/action shape (Tenant-aware workspace prep, branch create-or-reuse, branch-delete) so Gateflow's pin-reading code can parse it — lands as one contract PR covering all three shapes (see Capabilities note above) | D5, D1 | Pin remount | 0 BROKEN nodes on remounted pin | unit (pin parse) |
| REQ-31 | This is a genuine **contract change** to `prayog-skills` (not consume-only) — tracked distinctly from every prior INIT's "pin consume-only" posture | Outline §10 | Design record | prayog-skills spec PR exists for the new shape, reviewed as a contract change | inspection |
| REQ-32 | Tenant registry supports read/list: `GET /api/v1/tenants` and `GET /api/v1/tenants/{tenant_id}` return the tenant's registered repo list, `workspace_root`, and board default — never the PAT | Gap closed this Draft PRD (`review-findings`, VF-12) | Read call | 200 with tenant fields present; `pat` field absent from every response | unit + verify |

**Implementation notes (non-normative):** Gateflow may implement REQ-02/REQ-03 via
`ForgeActionType.update_board_status`, `NodeForgePolicy` status retention, and
`BoardService.update_ticket_status` — module names are design detail, not
product vocabulary. See §4.

### Locked decisions from this session (G1–G5)

| ID | Decision | Resolves |
|----|----------|----------|
| **G1** | Per-tenant PAT is stored in a dedicated Postgres column, in **plaintext** — no application-level encryption, no secrets-manager integration this INIT. This is an explicit, PM-accepted risk for the pilot scope, not a security recommendation. Carried into §4 Security & Privacy and §5 Technical risks so it is visible, not silently dropped. | OQ-1 |
| **G2** | The single per-tenant PAT authenticates **both** `ForgeClient` platform API calls **and** local git clone/fetch. No separate deploy-key/fine-grained-token credential is introduced this INIT, even though the programme vision doc's original architecture anticipated a separate credential for local git. | OQ-4 |
| **G3** | The User↔Tenant access boundary (D9) is built as a **new**, clean mechanism extending today's real, live pattern — a shared-secret bearer token, now minted per-tenant instead of one global `PROGRAMME_SERVICE_TOKEN`. The existing JWT `AuthMiddleware`/`AuthContext` (`user_id`/`tenant_id`/`owner_id`) is confirmed dormant (every `/api/v1/*` route is in `public_paths`) and traces to an unrelated product template — it is explicitly **parked**, not repurposed, by this INIT. | D9 (auth mechanism) |
| **G4** | Tenant registration **eagerly** verifies PAT read access per registered repo at registration time (fail fast), rather than deferring verification to first use. | OQ-2 |
| **G5** | "Build but don't activate" (D5) is expressed **structurally**: the pin declares the branch-delete action/node with `authorization: explicit`, but this INIT wires **zero** live outcome edges to it — dormancy is enforced by unreachability in the graph, not by a runtime settings flag that could be flipped without a spec change. | OQ-6 |

---

## 4. Technical Specifications

### Architecture Overview *(design — not product vocabulary)*

```text
Tenant registration (synchronous request handling)
  POST /api/v1/tenants                              (CAP-01 — new)
    → eager per-repo PAT check (G4) → Tenant row (plaintext PAT, G1) → tenant token (G3)
  POST /api/v1/tenants/{tenant_id}/users             (CAP-01 — new)
  GET  /api/v1/tenants, /api/v1/tenants/{tenant_id}  (CAP-01 — new, REQ-32; never returns the PAT)

Existing lane-start route (unchanged route, extended behavior — POST /api/v1/waves/implement/start):

  Synchronous request-handling phase (WaveStartService, before enqueue — unchanged from today):
    → RunRepository.find_active_run (repo-scoped NO_CONCURRENT_RUN — CAP-05, existing service, broadened query)
    → enqueue job

  Asynchronous worker phase (RunOrchestrator.process_job, after enqueue —
  same phase LaunchpadClient.sync_harness already runs in today; concurrency
  gate above has already passed before any of the following can run):
    workspace_path omitted + repo Tenant-registered
      → WorkspacePrep.clone_or_refresh(tenant, org, repo)   (CAP-02 — new; replaces today's Path.cwd() fallback)
      → HarnessReadinessCheck (extends LaunchpadClient.sync_harness)  (CAP-04 — new)
      → BranchResolver.create_or_reuse(initiative_id, wave_id)  (CAP-03 — new; ForgeClient.ensure_branch_from_base / branch_slug_from_head_ref — existing)
      → pre-implement → … (unchanged)

ForgeClient (existing infra service)
  + delete_branch(owner, repo, branch)          (CAP-06 — new method; dormant per G5)

prayog-skills workflow.yaml / delivery-contract.yaml (contract change — lands
as one PR covering all three shapes; see §3 note on REQ-30/REQ-31)
  + Tenant-aware workspace-prep node shape
  + branch create-or-reuse node shape
  + branch-delete forge action type (authorization: explicit)
```

### Integration Points

| System | Use |
|--------|-----|
| PostgreSQL (existing RunStore instance) | New `tenants` table (PAT column, repo list, workspace root, board default); new `tenant_users` join table |
| `ForgeClient` (existing) | Extended with `delete_branch`; unchanged for everything else |
| `LaunchpadClient` (existing stub) | Extended from path-exists-only to real harness-artifact check |
| Local git (new transport for Gateflow) | Clone/fetch via Tenant PAT (G2) — first local git execution inside Gateflow; distinct code path from `ForgeClient`'s REST/GraphQL, same credential |
| Remounted `prayog-skills` `workflow.yaml`/`delivery-contract.yaml` | New node/action shape (contract change, not consume-only) |
| Tenant-scoped bearer token (new, G3) | Auth for tenant-registration, user-attach, tenant read/list, and (going forward) lane routes scoped to that tenant |

### Security & Privacy

- **Carried-forward risk (G1):** the per-tenant PAT is stored in
  plaintext in Postgres. Anyone with read access to that table/column has
  the credential. This is an explicit, accepted scope cut for this INIT's
  pilot posture — not a security best practice, and not resolved by this
  INIT. Any production hardening (encryption at rest, secrets-manager
  migration, access audit on the table) is future work, tracked as a
  Technical risk (§5), not silently assumed away.
- **Carried-forward risk (G2):** the same PAT used for `ForgeClient` platform
  writes is also used for local git clone/fetch — a compromise of the local
  git credential is a compromise of the platform-write credential too (no
  blast-radius separation this INIT).
- The dormant JWT `AuthMiddleware` remains exactly as-is (still in
  `public_paths` for every `/api/v1/*` route) — this INIT neither fixes nor
  builds on it (G3).
- `delete_branch` (CAP-06) is a genuinely new destructive capability inside
  `ForgeClient`. It ships with `authorization: explicit` at the pin level and
  zero live outcome edges (G5) specifically so its blast radius is "provably
  correct in tests" only, this INIT.
- Content skills still must not treat local `gh` as automate success —
  `ForgeClient`/local-git-via-Tenant-PAT only (carried from
  INIT-GATEFLOW-009/010/011).

### AI / agent evaluation

Not a new model product. Agent quality bar = contract/orchestration correctness
(unit tests against fixture Tenants/repos with valid/invalid PAT,
harness-present/absent fixtures, fresh-vs-continuation fixtures) plus live
verify scripts proving at least one real clone, one real branch fork, one
real branch reuse, and one real repo-scoped concurrency rejection — matching
the INIT-GATEFLOW-009/010/011 evaluation pattern (verify scripts and units,
not rubrics).

### Code-grounded implementation notes (non-normative, verified against `gateflow` this session)

| Existing code | What it already does | What CAP-01–CAP-06 still need to add |
|---|---|---|
| `src/models/wave_start_models.py` — `ImplementWaveStartRequest.workspace_path` | Already `Optional[str]`; already the one wave-start request that skips `_require_existing_directory` | Wire the None-branch to CAP-02 clone/refresh (async worker phase, after the concurrency gate) instead of letting it fall through to `Path.cwd()` |
| `src/business_services/run_orchestrator.py` (`RunOrchestrator.process_job`) | Already runs `LaunchpadClient.sync_harness` **after** enqueue (the async worker phase); falls back to `Path.cwd()` when `workspace_path` is empty | CAP-02/CAP-03/CAP-04 hook into this same async phase, replacing the `Path.cwd()` fallback; the concurrency gate (CAP-05) has already run earlier, synchronously, in `WaveStartService` |
| CAP-02 workspace path scheme | — | Deterministic on-disk path: `{tenant.workspace_root}/{org}/{repo}` — this exact scheme is an implementation detail, not product-normative (see REQ-10/REQ-15) |
| `src/infra_services/forge_client.py` | `ensure_branch_from_base`, `_git_ref_get_path`/`_git_ref_update_path` shapes | Add `delete_branch` using the existing DELETE-on-`_git_ref_update_path` shape — no new transport |
| `src/models/pr_branch_naming.py` | Deterministic `build_wave_head_branch` / `branch_slug_from_head_ref` already used by closeout intake | Reuse directly for CAP-03 — do not invent a second naming function |
| `src/infra_services/launchpad_client.py` (`LaunchpadClient`) | Real, DI-wired stub; `sync_harness` only checks path existence ("W1: log + no-op") | Extend to a real harness-artifact check (CAP-04) — this is the intended extension point, not new infra |
| `src/database/postgres/repository/run_store_repository.py` (`find_active_run`) | Already filters `org`+`repo` first, then narrows by PR/issue/initiative+wave | Drop or broaden the narrowing filter so `org`+`repo` alone determines the conflict (CAP-05) |
| `src/business_services/trigger_router.py` (`TriggerRouter.authorize_and_check`) | Separate webhook/legacy-ingress precondition check; also calls `find_active_run` internally, but is **not** the code path used by the API lane-start route this INIT extends | Not part of CAP-05's wiring for the API path — do not treat as an alternate caller (corrects this Draft PRD's earlier mischaracterization) |
| `src/models/board_models.py` (`BoardTicketCreateRequest`) | `project_number`/`project_owner` already caller-supplied, no config-derived default | Tenant registry supplies a default; call-time override logic is unchanged |
| `src/api/v1/programme_token.py` (`verify_programme_service_token`) | Single global shared-secret check against `ProgrammeAuthSettings` | Generalize to a per-`tenant_id` token lookup (G3) — same dependency shape, new lookup source |
| `src/common/auth/middleware.py` (`AuthMiddleware`), `src/models/auth_models.py` | Wired into `app.py` but every real route is in `public_paths`; model fields/docstring trace to an unrelated product | Leave untouched (G3) — do not extend or repurpose |
| `src/configs/github_settings.py` (`GithubSettings`) | Single global env-var GitHub credential for the whole service | Tenant registry is additive — `GithubSettings` may remain for any non-Tenant-scoped path; not replaced this INIT |

---

## 5. Risks & Roadmap

### Delivery waves (product-normative)

| Wave | Intent | Exit REQs |
|------|--------|-----------|
| **W0** | Tenant registry: data model + API (register, attach user, eager PAT verify, read/list) | REQ-01–REQ-09, REQ-32 |
| **W1** | Repo clone (first use) + refresh (subsequent use) — consumes W0's repo list/credential | REQ-10–REQ-15 |
| **W2** | Branch create-or-reuse (new wave vs. continuation) — consumes W1's workspace | REQ-16–REQ-19 |
| **W3** | Harness-readiness check — gate before operating on a newly cloned repo | REQ-20–REQ-22 |
| **W4** | Repo-level sequencing — broaden `NO_CONCURRENT_RUN` to repo scope | REQ-23–REQ-25 |
| **W5** | Branch purge capability — built + pin-declared, dormant | REQ-26–REQ-31 |

**Note on REQ-30/REQ-31 (W5 exit):** the `prayog-skills` contract PR itself
covers three shapes spanning W1 (workspace-prep), W2 (branch create-or-reuse),
and W5 (branch-delete) — it is a single cross-cutting deliverable (§3
Capabilities note) landed once, not W5-only work. W5's "REQ-26–REQ-31" exit
range reflects where the *last* of the three shapes (branch-delete) is
proven; W1 and W2 exit implicitly depend on the same contract PR landing
early enough to unblock them.

### Technical risks

| Risk | Mitigation |
|------|------------|
| Plaintext PAT in Postgres (**G1**) is a real credential-exposure surface | Explicit, visible, accepted for this INIT's pilot scope; flagged for a follow-up hardening initiative, not silently carried as if resolved |
| Single PAT spans both `ForgeClient` writes and local git (**G2**) — no blast-radius separation | Same acceptance as G1; a compromised PAT already had platform-write scope regardless |
| Reopening `delete_branch` reverses a boundary INIT-GATEFLOW-008/010 and `prayog-skills`' own gap list deliberately closed | Mitigated by D5/G5 — capability ships dormant, zero live outcome edges, `authorization: explicit` even once activated |
| Local git execution (clone/fetch) is new attack surface inside Gateflow, not previously present | Reuses the Tenant PAT already scoped for this Tenant (G2); no new credential type to secure separately, but no isolation from platform-write scope either (see risk above) |
| Deployment topology assumption (persistent disk, A2/OQ-5) unconfirmed | Outline's own lean is toward persistent; not a blocker per outline, but flagged for infra confirmation before W1 delivery freeze |
| Broadening `NO_CONCURRENT_RUN` to repo scope could surface latent double-starts in existing tenant usage that today's narrower key silently allowed | Unit test against real historical run patterns before flipping the query; verify script asserts the new rejection shape explicitly |
| Scope creep into `gateflow-ops` UI or GitHub App installation mode | Explicit non-goals (§2); separate, later initiatives |
| Overlap with `INIT-GATEFLOW-004`'s onboarding scorecard categories going unreconciled | Flagged as a Non-Goal (§2) for cross-initiative coordination, not silently ignored |

### Phased rollout

- **MVP (this INIT, W0–W5):** Tenant registry + workspace/branch
  lifecycle capability, dormant purge, contract shape change.
- **Later (explicit follow-ups, not this INIT):** PAT encryption/secrets-
  manager migration (resolves the G1 risk); separate git-workspace
  credential (resolves the G2 risk); `gateflow-ops` onboarding UI consuming
  CAP-01's API; branch-purge activation decision (a separate, later
  decision per D5); GitHub App installation mode per tenant (D8 follow-
  on); reconciling CAP-04/US-3 with INIT-GATEFLOW-004's scorecard.

---

## 6. Locked decisions reference

### Carried forward from outline (D1–D10)

| ID | Decision |
|----|----------|
| D1 | Gateflow owns its own working-directory lifecycle per use case and makes its own git calls (clone, branch, delete) — reverses ADR-010's caller-supplied-workspace assumption for Tenant-registered repos |
| D2 | New wave forks from `develop` tip; continuation reuses the existing wave's PR branch — no re-fork |
| D3 | Merge always targets `develop` (unchanged) |
| D4 | Branch purge happens on wave close (post-merge) — capability, see D5 |
| D5 | Ship as a **capability, not an activated behavior** — pin/contract change only; live default flow unchanged this INIT |
| D6 | Harness-readiness check before Gateflow operates on a repo; flag non-readiness rather than proceed |
| D7 | One combined initiative, **backend only** — Tenant registry + workspace/branch lifecycle, no UI; onboarding via direct API call |
| D8 | Per-tenant GitHub credential is **PAT only**; GitHub App per tenant deferred |
| D9 | Access boundary is **User ↔ Tenant**, not User ↔ Repo — no per-repo ACL |
| D10 | Sequentiality enforced at the **repo** level — one active path per repo; full parallelism across different repos; no new locking/worktree isolation needed |

### Added in this Draft PRD (G1–G5)

| ID | Decision |
|----|----------|
| G1 | Per-tenant PAT stored **plaintext** in a Postgres column — no encryption/secrets-manager this INIT; explicit accepted risk (resolves OQ-1) |
| G2 | Single per-tenant PAT serves **both** `ForgeClient` API auth and local git clone/fetch — no separate deploy key this INIT (resolves OQ-4) |
| G3 | User↔Tenant auth is a **new**, per-tenant extension of the existing shared-secret bearer-token pattern; the dormant JWT `AuthMiddleware`/`tenant_id` scaffolding is explicitly parked, not reused |
| G4 | Tenant registration **eagerly** verifies PAT read access per repo at registration time (resolves OQ-2) |
| G5 | "Build but don't activate" (D5) is enforced **structurally** — pin declares the action with `authorization: explicit`; zero live outcome edges wired this INIT (resolves OQ-6) |

---

## 7. Next steps

1. Review this Draft PRD with PE/programme — confirm G1–G5 (especially G1's
   plaintext-PAT risk acceptance) are read and owned, not just defaulted.
2. Impact map → meta Gate 1 → **gateflow** implementation waves (§5) **and**
   a **`prayog-skills` spec/contract PR** for the new node/action shape
   (REQ-30/REQ-31) — this is a genuine contract change, routed like a real
   spec pass, not a consume-only catch-up.
3. Do **not** implement Gateflow or prayog-skills code from this Draft PRD
   alone — follow SDD (spec → feasibility → technical review → plan →
   waves), same as every prior INIT.
4. Flag to PE/programme explicitly that this reopens the `delete_branch`
   boundary INIT-GATEFLOW-008 and INIT-GATEFLOW-010 deliberately closed, and
   that it introduces the first local-git-credential surface inside
   Gateflow — both are considered decisions (D5/G5, G2), not scope creep,
   but both deserve a second pair of eyes before the `prayog-skills` spec PR
   is opened.

---

## Appendix A — New data model (illustrative — engineering owns exact schema)

| Table | Key columns | Notes |
|-------|-------------|-------|
| `tenants` | `id`, `name`, `pat` (plaintext, **G1**), `workspace_root`, `board_project_owner`, `board_project_number`, `created_at` | New table; follows existing `PostgresBaseModel`/`Mapped[...]` schema conventions (`src/database/postgres/schema/*.py`) |
| `tenant_repos` | `tenant_id`, `org`, `repo` | Repo list — one tenant may own many repos (D7) |
| `tenant_users` | `tenant_id`, `user_identity` (email/handle) | Attach records; no per-repo grant column (D9) |

## Appendix B — Target API surface (new routes only)

| Route | Capability | Notes |
|-------|------------|-------|
| `POST /api/v1/tenants` | CAP-01 | Registration; eager per-repo PAT check (G4) |
| `POST /api/v1/tenants/{tenant_id}/users` | CAP-01 | Attach a user (D9) |
| `GET /api/v1/tenants` | CAP-01 | List registered tenants (REQ-32) — never returns the PAT |
| `GET /api/v1/tenants/{tenant_id}` | CAP-01 | Tenant detail: repo list, `workspace_root`, board default (REQ-32) — never returns the PAT |
| *(no new route)* — `POST /api/v1/waves/implement/start` behavior extended | CAP-02, CAP-03, CAP-04, CAP-05 | `workspace_path` becomes optional-and-self-sufficient for Tenant-registered repos; existing route, extended behavior |

Exact response schema / problem+json field names → **OQ-01** (deferred to
gateflow OpenAPI pass, same deferral as INIT-GATEFLOW-010/011).

---

## References

- Outline: [INIT-GATEFLOW-012-outline](./INIT-GATEFLOW-012-outline.md)
- Vision: [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md)
- Predecessor: [INIT-GATEFLOW-010](./INIT-GATEFLOW-010.md) (eng-lane pin tip executor parity — this INIT extends that executor's workspace assumptions)
- Related, independent: [INIT-GATEFLOW-011](./INIT-GATEFLOW-011.md) (Day-1 visibility — no dependency either direction)
- Related, unreconciled overlap (§2 Non-Goals, §5 risks): [INIT-GATEFLOW-004](./INIT-GATEFLOW-004.md) (Mission Control onboarding scorecard)
- Validation / resolution history: [Validation-Report-INIT-GATEFLOW-012](./reports/Validation-Report-INIT-GATEFLOW-012.md), [Resolution-INIT-GATEFLOW-012](./reports/Resolution-INIT-GATEFLOW-012.md)
- Code evidence (verified this session against local `drivestream-lab/gateflow` checkout): `src/models/wave_start_models.py`, `src/business_services/wave_start_service.py`, `src/business_services/run_orchestrator.py`, `src/infra_services/forge_client.py`, `src/models/pr_branch_naming.py`, `src/infra_services/launchpad_client.py`, `src/database/postgres/repository/run_store_repository.py`, `src/business_services/trigger_router.py`, `src/models/board_models.py`, `src/configs/github_settings.py`, `src/configs/programme_auth_settings.py`, `src/api/v1/programme_token.py`, `src/common/auth/middleware.py`, `src/models/auth_models.py`
