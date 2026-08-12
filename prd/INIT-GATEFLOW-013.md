# INIT-GATEFLOW-013 — Programme-first onboarding, choosing which repos matter, and real readiness checks

**Status:** draft PRD · **Author:** programme PM · **Date:** 2026-08-08
**Outline:** [INIT-GATEFLOW-013-outline](./INIT-GATEFLOW-013-outline.md)
**Vision:** [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md)
**Component:** GATEFLOW · **Type:** onboarding experience
**Predecessor:** INIT-GATEFLOW-012 (tenant registry + workspace/branch lifecycle) — **hard depend, fully shipped**. All six waves are merged and human-approved; eng-side closure has already run.

> **Draft PRD** — Locked decisions D1–D11 (outline) carried forward unchanged.
> Primary delivery = **gateflow only**; no `prayog-skills` contract change is
> expected, consistent with how INIT-GATEFLOW-012 itself turned out (its own
> anticipated contract change was never needed either).
>
> **As-built baseline (Source: verified against gateflow, launchpad, and
> prayog-meta code this session).** `TenantRegisterRequest.repos` is
> currently **required, non-empty**
> (`src/models/tenant_models.py:34` — `Field(min_length=1)`), and
> `TenantService.register_tenant` explicitly rejects an empty list
> (`src/business_services/tenant_service.py:54–58`) — confirming today's
> "must hand-type at least one repo at setup" behavior is real code, not a
> hypothetical, and is exactly what D1/D11 retire. `tenant_repos.harness_verified`
> already exists and is already written by `TenantService.mark_harness_verified`
> / read by `TenantService.is_harness_verified`, keyed by `(org, repo)`
> (`src/business_services/tenant_service.py:176–206`) — confirming D9's
> "already-live gate" framing. `GithubPatProbe.verify_read_access` already
> exists as a per-call, caller-credential probe
> (`src/infra_services/github_pat_probe.py`) — the same mechanism this PRD
> reuses for CAP-03's repo-selection probe (REQ-11). `TenantGitWorkspaceClient.resolve_workspace`
> already exists as a generic `(org, repo)` clone-or-fetch mechanism
> (`src/infra_services/tenant_git_workspace_client.py`) — reused unchanged
> for both the programme connection (CAP-01) and repo setup (CAP-04). Launchpad's
> `status` command is real and already does a materially deeper check than
> today's file-presence gate — kit-version drift, pin freshness, and
> constitution/skills drift — via `--repo`/`--meta` and `--config-dir`
> (verified against `launchpad` code this session); `--client` resolves
> through an operator's local machine registry (`~/.config/launchpad/clients.yaml`),
> confirming it is unusable from a server process. `config/programme.yaml`
> and `config/service-catalog-drivestream-lab.yaml` are real files in
> `prayog-meta` today.

---

## Document control

| Field | Value |
|-------|-------|
| Initiative ID | INIT-GATEFLOW-013 |
| Programme | prayog |
| Primary repo | drivestream-lab/gateflow |
| Supporting repos | drivestream-lab/prayog-meta (config files — read-only consumer); drivestream-lab/launchpad (CLI, inspect-only — no product change) |
| Explicitly **not** touched this INIT | gateflow-ops (no UI); any Launchpad mutate/apply command |
| Target users | Tenant administrator (now connects to the programme and chooses repos, instead of typing them in); developer running day-to-day work (unaffected directly; benefits from a more trustworthy readiness check) |
| Depends on | INIT-GATEFLOW-012 — fully shipped. This INIT reuses its git-workspace client and credential-probe mechanisms unchanged, and takes over ownership of its harness-readiness gate (already live in production) |

---

## 1. Executive Summary

### Problem Statement

Registering a tenant today requires hand-typing at least one repo directly into the setup call — the code enforces this (`repos` is a required, non-empty field) — and there is no way to discover what a programme actually owns, no way to add only a subset of a large shared list, and no way to learn about a new repo the programme adds later without repeating setup from scratch. Separately, the readiness check that already gates every wave start today only confirms that a couple of setup files exist — a materially weaker guarantee than what the programme's own setup tool can already provide.

### Proposed Solution

Give every tenant a guided path to its repos, sourced entirely from the programme's own shared records — connect once, see the full list, choose what matters, and get set up automatically — and replace today's thin, file-presence readiness check with a real check run through the programme's own setup tool. From this initiative onward, **hand-typing a repo directly is retired entirely**, for new and existing tenants alike; every repo, the first one and every one after, arrives only through this connect-and-choose flow.

### Success Criteria

| KPI | Target | Measurement |
|-----|--------|-------------|
| **No more hand-typed repos** | Setting up a brand-new tenant no longer requires or accepts a repo list; every repo arrives via programme connection + selection | Unit + inspection of the registration path |
| **Programme-sourced discovery** | A connected tenant sees every repo the programme's shared records declare, with zero manual file-reading by the operator | Unit + verify (fixture programme repo) |
| **Selection integrity** | Every attempt to select a repo not on the current catalogue is rejected; every accepted selection is checked for read access before being added | Unit + verify (fixture: in-catalogue vs. out-of-catalogue attempts) |
| **Partial-success setup and checks** | One repo's setup or readiness-check failure never affects any other repo in the same batch; each gets its own named result | Unit + verify (mixed success/failure fixture) |
| **One trustworthy readiness answer** | Every repo added through this initiative has its readiness answered exclusively by the real check; repos that predate this initiative are provably untouched | Unit + verify + code guard (no write path touches pre-existing rows) |
| **Catalogue stays current** | A connected tenant can refresh and see a newly added programme repo without repeating setup | Unit + verify (fixture: catalogue grows between two refreshes) |

### Capability ↔ wave ↔ requirement map

| CAP | Wave | Capability | REQ |
|-----|------|------------|-----|
| CAP-01 | W0 | Connect to the programme | REQ-01–REQ-04, REQ-28 |
| CAP-02 | W0 | Read the shared catalogue | REQ-05–REQ-07 |
| CAP-03 | W1 | Choose repos (retires hand-typed setup) | REQ-08–REQ-13, REQ-26–REQ-27 |
| CAP-04 | W2 | Set up chosen repos | REQ-14–REQ-16 |
| CAP-05 | W3 | Real readiness check | REQ-17–REQ-20 |
| CAP-06 | W3 | One trustworthy readiness answer | REQ-21–REQ-23 |
| CAP-07 | W4 | Stay current | REQ-24–REQ-25 |

---

## 2. User Experience & Functionality

### User Personas

| Persona | Role | Primary need |
|---------|------|---------------|
| **Tenant administrator** | Sets up and grows a tenant's repo list | Connect to the programme once, see everything it owns, and choose exactly the repos that matter — without typing anything in by hand, and without missing new repos later |
| **Developer running day-to-day work** | Starts and runs waves on already-set-up repos | No change to how their work starts or runs; the readiness check that already stands between them and a wave start becomes a real answer, not a file-presence guess |

### User Stories & Acceptance Criteria

#### US-1 — Connect a tenant to its programme `(CAP-01)`

**As a** tenant administrator, **I want** to connect my tenant to our programme's shared records once, **so that** Gateflow always has an up-to-date copy of what the programme owns.

**Acceptance criteria:**

- [ ] Connecting clones/syncs the programme's shared repo using the exact same clone-or-fetch mechanism already used for every other repo — no second git mechanism is introduced
- [ ] Connecting authenticates with the tenant's existing access credential — no new credential type
- [ ] The same credential the tenant already uses for its other repos is reused to read the shared repo — no separate, narrower credential is created for this
- [ ] If the shared repo can't be reached (bad credential, network issue, not found), connecting fails with a specific, named reason and leaves nothing partial behind
- [ ] Connecting again while already connected refreshes/re-syncs the existing connection — it never creates a second one

#### US-2 — See everything the programme owns `(CAP-02)`

**As a** tenant administrator, **I want** to see every repo my programme's shared records declare, **so that** I never have to open a file myself to find out what exists.

**Acceptance criteria:**

- [ ] After connecting, Gateflow reads the programme's shared records and turns them into a list of repos an operator can see
- [ ] If those records don't look like what's expected (missing, malformed, wrong shape), the read fails with a specific reason rather than silently returning an empty or partial list
- [ ] The list always reflects the most recently synced copy of the programme's records — never a value frozen forever from the moment of first connecting

#### US-3 — Choose exactly the repos that matter `(CAP-03)`

**As a** tenant administrator, **I want** to select a subset of the programme's repos as my tenant's active list, **so that** I don't have to take everything the programme owns.

**Acceptance criteria:**

- [ ] A selection call saves a chosen subset of the current catalogue as the tenant's active repos
- [ ] Any attempt to select something not on the current catalogue is rejected outright — no exceptions
- [ ] A selection can be revisited and changed later; every change is checked against the *current* catalogue, not whatever it looked like at an earlier connect or refresh
- [ ] Selecting a repo the tenant doesn't already have runs the same read-access check already used at tenant setup today, before the repo is added — an inaccessible or mistyped-on-the-catalogue-side repo is caught immediately, not silently added
- [ ] Setting up a brand-new tenant no longer requires or accepts a starting repo list — every tenant begins with zero repos
- [ ] This selection step is the *only* way any tenant, new or already existing, ever gains a repo from this point forward — there is no remaining way to add one by typing it in directly
- [ ] Deselecting a repo removes it from the active list only — its local clone and stored readiness answer are left untouched
- [ ] Deselecting a repo with a wave currently running on it is rejected — it must be deselected only once no wave is in flight

#### US-4 — Get chosen repos ready to work on `(CAP-04)`

**As a** tenant administrator, **I want** every repo I choose to be automatically set up, **so that** choosing a repo and being able to actually work on it are the same moment, not two separate steps.

**Acceptance criteria:**

- [ ] Every newly selected repo is cloned (and, where it has sub-projects, those are set up too), using the exact same mechanism already used for repo setup — no second mechanism
- [ ] Setting up each selected repo happens independently — one repo's failure never blocks or delays any other repo in the same selection
- [ ] Each repo's setup result is reported on its own, with a specific reason on failure — never bundled into a single all-or-nothing outcome

#### US-5 — Get a real readiness check, not a file-presence guess `(CAP-05)`

**As a** tenant administrator, **I want** every chosen repo to go through the programme's own proper readiness check, **so that** "ready" means something real.

**Acceptance criteria:**

- [ ] For every selected, set-up repo, Gateflow asks the programme's own setup tool to run its full check, pointed at Gateflow's own synced copy of the programme's records
- [ ] The setup tool is never asked to fix, install, or change anything for any repo, or for the programme's shared repo itself — questions only
- [ ] Each repo's check runs and reports independently, matching the same one-at-a-time, partial-success pattern as setup
- [ ] If the setup tool itself can't run (missing, misconfigured), that's reported as its own distinct, named failure — never confused with "this specific repo failed its check"

#### US-6 — Trust one clear readiness answer `(CAP-06)`

**As a** developer, **I want** "is this repo ready" to mean one real thing, **so that** I never wonder which of two checks decided the answer.

**Acceptance criteria:**

- [ ] The real check's result becomes the stored readiness answer for every repo added through this initiative, replacing the simpler check that answers it today
- [ ] For a repo a tenant already had *before* this initiative shipped, its existing stored answer is left exactly as it is — never reset, never forced through a fresh check, never requiring the tenant to reconnect
- [ ] A stored readiness answer can be refreshed on demand, without needing to remove and re-select the repo

#### US-7 — Never lose track as the programme grows `(CAP-07)`

**As a** tenant administrator, **I want** to refresh my view of the programme's repos at any time, **so that** I find out about new ones without starting over.

**Acceptance criteria:**

- [ ] A connected tenant can refresh its copy of the programme's shared records at any time
- [ ] The catalogue of choices updates to reflect anything newly added by the programme since the last connect or refresh
- [ ] Refreshing never removes or changes an already-selected repo on its own — it only ever changes what's newly available to select

### Non-Goals

| Non-goal | Why |
|----------|-----|
| Any screens or dashboard | This initiative ships only what other systems can call; screens are a later initiative's job |
| Letting the programme's setup tool fix, install, or change anything | Inspect-only, by design — never a mutating call |
| Selecting a repo that isn't on the programme's shared list | Deliberately closed off — no way around the shared list |
| Keeping a way to add a repo by typing it in directly, for anyone | Retired outright — not offered as a fallback for any tenant, new or existing |
| Retroactively re-checking or resetting repos a tenant already had before this initiative shipped | Left exactly as-is — no forced reconnect, no forced re-check |
| Any finer-grained, per-repo permission model | INIT-GATEFLOW-012 already fixed the boundary at the tenant level; unchanged here |
| Changing how a wave actually runs once it starts | This initiative is about getting set up; wave-start mechanics are untouched |
| A separate, narrower credential just for reading the programme's shared records | Same credential as everything else |
| Relying on anything stored on an individual operator's own machine | The setup tool is always pointed at Gateflow's own synced copy of the programme's records |
| Reconciling this initiative's readiness check with INIT-GATEFLOW-016's onboarding scorecard | **Resolved by INIT-GATEFLOW-016 (OQ-2):** this initiative's existing catalogue (CAP-02), selection (CAP-03), and readiness (CAP-05/CAP-06) endpoints are composed client-side in gateflow-ops into a single pass/fail onboarding verdict — no new composite endpoint requested of this initiative; coordination point closed |

### Assumptions

| ID | Assumption | Status | Dependent REQs |
|----|------------|--------|----------------|
| A1 | The programme's shared repo is reachable through the same transport already used for every other repo — no new network path or protocol | Confirmed by scope | REQ-01, REQ-04 |
| A2 | The environment Gateflow runs in can be given the ability to run the programme's setup tool (a new runtime dependency, not previously required) | Assumed, needs an explicit provisioning decision (see OQ-2) | REQ-17, REQ-20 |
| A3 | The credential-probe mechanism already used at tenant setup today can be reused, unchanged, at the point a repo is newly selected | Confirmed by code — the same mechanism, a different call site | REQ-11 |
| A4 | This initiative's readiness-check write path only ever touches repos added through selection (CAP-03) going forward; it never re-touches a repo record that already existed before this initiative shipped | Confirmed by design (no code path is added that iterates or rewrites pre-existing repo records) | REQ-21, REQ-22 |

### Error table (product-normative)

| Situation | Result | Side effects |
|-----------|--------|---------------|
| Setting up a brand-new tenant with a repo list provided | Rejected | 0 rows written; repo list is no longer an accepted input |
| Connecting to the programme's shared repo fails (credential, network, not found) | Rejected, specific reason named | Nothing partial left behind |
| The programme's shared records don't look as expected | Rejected, specific reason named | No candidate list produced; nothing guessed |
| Selecting a repo not on the current catalogue | Rejected | 0 change to the tenant's active repo list |
| Selecting a repo that fails the read-access check | Rejected, specific reason named | 0 change to the tenant's active repo list |
| One repo fails setup while others in the same selection succeed | The failing repo is reported with its own reason | Other repos are unaffected and proceed normally |
| The programme's setup tool itself can't run | Rejected, distinct reason from any individual repo's check result | No repo in the batch is marked either ready or not-ready by this failure |
| Refreshing the programme's records fails | Rejected, specific reason named | Existing selections and existing readiness answers are untouched |
| Deselecting a repo with a wave currently in flight | Rejected | 0 change to the active list; repo remains selected |

### Open questions

| ID | Open question | Status |
|----|----------------|--------|
| OQ-1 | If a chosen repo was never checked, or its last check failed, should starting a wave on it be blocked outright, or allowed with a warning? | Open — must lock before implementation |
| OQ-2 | Where does the programme's setup tool actually run from in Gateflow's environment — built in ahead of time, or fetched when needed? | Open — needs a decision from whoever manages that environment |
| OQ-3 | Should refreshing the programme's records happen on a schedule, only when asked, or both? | Open |
| OQ-4 | Do repos with nested sub-projects need any extra credential handling beyond the tenant's existing credential? | Open |
| OQ-5 | What should happen if the setup tool available to Gateflow doesn't match what a particular repo expects? | Open — needs a compatibility policy |
| OQ-8 | What today actually depends on the old "set up a tenant with a repo list" behavior — scripts, runbooks, habits — that would break once it's retired? | Open — needs to be sized before implementation |
| OQ-9 *(new)* | For a repo that predates this initiative and was never brought in through programme connection, can an operator still ask for a fresh readiness check on it — and if so, does that fresh check use the old file-presence method (since there's no programme connection to point the real check at) or is a fresh check simply unavailable until that repo's tenant connects? | Open — a real gap the outline didn't resolve; must lock before implementation |

`OQ-6` and `OQ-7` from the outline are resolved (see §6) and not carried here as open.

---

## 3. Functional requirements

### Capabilities

| ID | Capability | Covers |
|----|------------|--------|
| CAP-01 | Connect to the programme | REQ-01–REQ-04, REQ-28 |
| CAP-02 | Read the shared catalogue | REQ-05–REQ-07 |
| CAP-03 | Choose repos (retires hand-typed setup) | REQ-08–REQ-13, REQ-26–REQ-27 |
| CAP-04 | Set up chosen repos | REQ-14–REQ-16 |
| CAP-05 | Real readiness check | REQ-17–REQ-20 |
| CAP-06 | One trustworthy readiness answer | REQ-21–REQ-23 |
| CAP-07 | Stay current | REQ-24–REQ-25 |

### Requirements

| ID | Requirement | Outline | Condition | Observable result | Evidence |
|----|-------------|---------|-----------|--------------------|----------|
| REQ-01 | A tenant can connect to its programme's shared repo (identified by org/repo/ref); connecting clones/syncs it using the same mechanism already used for any other repo | D2 | Connect call | Shared repo present in Gateflow's own synced copy | unit + verify |
| REQ-02 | Connecting authenticates with the tenant's existing access credential; no new credential type exists | D6 | Connect call | Same auth check as every other tenant call | unit |
| REQ-03 | The tenant's existing credential (already used for its other repos) is reused to read the shared repo; no separate, narrower credential is created | D7 | Connect call | Same credential used; no new credential field/table | unit + inspection |
| REQ-04 | Connect failure (credential, network, not found) is rejected with a specific, named reason; nothing partial is left behind | Fail-closed | Bad credential / unreachable / missing repo | Named reason; no partial local copy | unit + verify |
| REQ-05 | After connecting, Gateflow reads the programme's shared records from its synced copy and produces a list of candidate repos | D1 | Any read call after connect | List of candidates matches the shared records | unit + verify |
| REQ-06 | If the shared records don't match the expected shape, the read is rejected with a specific reason, not guessed at or partially returned | Fail-closed | Malformed/missing records | Named reason; no partial candidate list | unit + verify |
| REQ-07 | The candidate list reflects the most recently synced copy — not a value frozen at first connect | D4 | After a refresh (CAP-07) | List includes anything added since the prior sync | unit + verify |
| REQ-08 | An operator can select and save a subset of the current candidate list as the tenant's active repos | D4 | Selection call | Selected subset persisted | unit + verify |
| REQ-09 | Any repo not on the current candidate list is rejected from selection outright | D1 | Selection includes an out-of-catalogue repo | Rejected; 0 change to active list | unit + verify |
| REQ-10 | A selection can be changed later; every change is checked against the *current* candidate list | D4 | Re-selection after catalogue changes | New selection checked fresh, not against a stale list | unit + verify |
| REQ-11 | Selecting a repo the tenant doesn't already have runs the same read-access probe already used at tenant setup, before the repo is added | A3 | New repo selected | Probe runs; failure rejects the selection with a named reason | unit + verify |
| REQ-12 | Setting up a brand-new tenant no longer requires or accepts a starting repo list; a repo list provided at setup is rejected | D1, D11 | New tenant setup with a repo list provided | Rejected; tenant not created with any repos this way | unit + verify (regression against today's required-non-empty behavior) |
| REQ-13 | Selection (CAP-03) is the only way any tenant, new or already existing, gains a repo from this point forward | D1, D11 | Any attempt to add a repo any other way | No such path exists in the API surface | inspection + code guard |
| REQ-14 | Every newly selected repo is cloned (and sub-projects initialized, where present) using the exact same mechanism already used for repo setup | D2 | New repo selected | Repo present and usable, same as any tenant repo today | unit + verify |
| REQ-15 | Setting up each selected repo happens independently; one repo's failure never blocks or delays any other repo in the same selection | D5 | Mixed success/failure batch | Failing repo isolated; others proceed | unit + verify |
| REQ-16 | Each repo's setup result is reported individually, with a specific reason on failure | D5 | Any selection batch | Per-repo result, never a single bundled outcome | unit + verify |
| REQ-17 | For every selected, set-up repo, Gateflow asks the programme's setup tool to run its full readiness check, pointed at Gateflow's own synced copy of the programme's records (depends on A2 / `OQ-2` — setup-tool provisioning; see §2) | D3 | Repo set up | Real check runs, not a file-presence guess | unit + verify |
| REQ-18 | The setup tool is only ever asked questions — never asked to fix, install, or change anything, for any repo or the shared repo itself | D3 | Any check call | No mutating call is ever made | code guard + verify |
| REQ-19 | Each repo's check runs and reports independently, matching CAP-04's partial-success pattern | D5 | Mixed pass/fail batch | Failing repo isolated; others proceed | unit + verify |
| REQ-20 | If the setup tool itself can't run, that failure is reported distinctly from any individual repo's check result (depends on A2 / `OQ-2` — setup-tool provisioning; see §2) | Fail-closed | Tool unavailable/misconfigured | Distinct named reason; no repo wrongly marked either way | unit + verify |
| REQ-21 | The real check's result becomes the stored readiness answer for every repo added through this initiative, replacing today's simpler check | D9 | New repo, check completes | Stored answer reflects the real check | unit + verify |
| REQ-22 | A repo a tenant already had before this initiative shipped keeps its existing stored readiness answer untouched | D9 (resolves outline `OQ-6`) | Any repo predating this initiative | No write ever touches that repo's existing record | unit + code guard |
| REQ-23 | A stored readiness answer can be refreshed on demand without removing and re-selecting the repo | Efficiency | Refresh-check call | New check result replaces the stored answer in place | unit + verify |
| REQ-24 | A connected tenant can refresh its copy of the programme's shared records at any time | D4 | Refresh call | Synced copy updated | unit + verify |
| REQ-25 | Refreshing never changes an already-selected repo on its own — it only changes what's newly available to select | D4 | Refresh after catalogue growth | Existing selections untouched; new candidates appear | unit + verify |
| REQ-26 | A repo can be deselected from the tenant's active list; deselecting does not delete its local clone or alter its stored readiness answer — only active-list membership changes | D9 (consistency) | Deselect call | Clone and readiness record untouched; only list membership changes | unit + verify |
| REQ-27 | Deselecting a repo with a wave currently running on it is rejected | Fail-closed | Deselect attempted mid-wave | Rejected; repo remains selected | unit + verify |
| REQ-28 | A tenant has exactly one active programme connection at a time; connecting again refreshes/re-syncs that same connection rather than creating a second one | D1 | Connect call while already connected | Same connection record updated; no second record created | unit + verify |

**Implementation notes (non-normative):** REQ-01/REQ-14 may both be implemented as calls to
the same generic clone-or-fetch mechanism already used for tenant repos, varying only in
the `(org, repo)` argument. REQ-03/REQ-11 may reuse the same per-call, caller-credential
read-access probe already used at tenant setup, unchanged. REQ-12 is a tightening of an
existing, currently-required field on the tenant setup call, not a new validation rule
invented from nothing. Module/method names are design detail, not product vocabulary. See §4.

---

## 4. Technical Specifications

### Architecture Overview *(design — not product vocabulary)*

```text
Connect to the programme (CAP-01)
  → clone/sync the programme's shared repo (same mechanism as any tenant repo)

Read the shared catalogue (CAP-02)
  → parse the synced copy's shared records → candidate list

Choose repos (CAP-03)
  → validate selection against current candidate list
  → run the existing read-access probe on any newly selected repo
  → persist the active repo list
  → (retires: tenant setup accepting a repo list directly)

Set up chosen repos (CAP-04)
  → clone/prepare each newly selected repo, one at a time, independently
    (same mechanism as CAP-01's programme clone)

Real readiness check (CAP-05) + one trustworthy answer (CAP-06)
  → ask the programme's setup tool to check each repo, pointed at the
    synced programme copy — never at anything on an individual machine
  → store each result as that repo's readiness answer
  → repos predating this initiative are never touched by this write path

Stay current (CAP-07)
  → re-sync the programme's shared repo on demand
  → re-derive the candidate list from the fresh copy
```

### Integration Points

| System | Use |
|--------|-----|
| The programme's shared repo (prayog-meta) | Read-only source of the repo catalogue and related settings |
| The programme's setup tool (launchpad) | Asked, never told — read-only readiness checks per repo |
| Gateflow's existing repo clone/refresh mechanism | Reused unchanged for both the programme connection and every chosen repo |
| Gateflow's existing credential-probe mechanism | Reused unchanged for validating a newly selected repo's access |
| Gateflow's existing tenant access credential | Reused unchanged for every call this initiative adds — no new credential type |

### Security & Privacy

- The same credential a tenant already holds now reads a wider set of things (the programme's shared repo, in addition to its chosen app repos) — a larger blast radius than before if that credential is ever compromised. This is the same *kind* of risk already accepted for this tenant model; it's named here explicitly rather than left implicit.
- The programme's setup tool is never given the ability to change anything — every call this initiative makes to it is a question, never an instruction.
- Nothing this initiative reads or writes depends on anything stored on an individual operator's own machine.
- Repos that predate this initiative are never touched by any new write path this initiative introduces — their existing standing is preserved exactly.

### AI / agent evaluation

Not a new model product — no AI/LLM involvement in any capability here. Quality bar is
contract/behavior correctness: unit tests against fixture programmes and repos (valid/invalid
credential, in-catalogue/out-of-catalogue selections, pass/fail readiness checks), plus live
verify scripts proving at least one real connect, one real catalogue read, one real selection,
one real per-repo setup, and one real readiness check — matching the evaluation pattern used
for every prior initiative in this programme (verify scripts and units, not rubrics).

### Code-grounded implementation notes (non-normative, verified against `gateflow`/`launchpad` this session)

| Existing code | What it already does | What CAP-01–CAP-07 still need to add |
|---|---|---|
| `TenantRegisterRequest.repos` (`src/models/tenant_models.py`) | Required, non-empty (`Field(min_length=1)`) | Loosen to accept zero repos (or remove the field entirely) — REQ-12 |
| `TenantService.register_tenant` (`src/business_services/tenant_service.py`) | Explicitly rejects an empty `repos` list today | Remove that rejection; a tenant may be created with zero repos — REQ-12 |
| `TenantGitWorkspaceClient.resolve_workspace` | Already generic on `(org, repo)` — clone-or-fetch, one per-repo lock | Call it once more for the programme's own shared repo (CAP-01) and once per newly selected repo (CAP-04) — no new method needed |
| `GithubPatProbe.verify_read_access` | Already a per-call, caller-credential probe used at tenant setup | Call it again at selection time for a newly chosen repo (REQ-11) — same mechanism, new call site |
| `TenantService.mark_harness_verified` / `is_harness_verified` (`tenant_repos.harness_verified`) | Already the live read/write path for the readiness flag, keyed by `(org, repo)` | CAP-06's real-check result writes through this same path for newly selected repos; the write path must not be extended to iterate or touch rows that already existed before this initiative — REQ-22 |
| `LaunchpadClient.sync_harness` (file-presence check) | Already the thing that computes today's readiness answer | Not modified for pre-existing repos (REQ-22); whether it remains callable at all for a legacy, non-catalogue repo's force-recheck is `OQ-9`, not decided here |
| Launchpad `status --repo` / `--meta` via `--config-dir` | Already does a materially deeper check than file-presence | New call site inside CAP-05, pointed at Gateflow's own synced meta checkout |
| `config/programme.yaml`, `config/service-catalog-<org>.yaml` (prayog-meta) | Already real files with a defined shape | New parser needed inside CAP-02 — first consumer of these files from Gateflow |

---

## 5. Risks & Roadmap

### Delivery waves (product-normative)

| Wave | Intent | Exit REQs |
|------|--------|-----------|
| **W0** | Connect to the programme + read the shared catalogue | REQ-01–REQ-07, REQ-28 |
| **W1** | Choose repos — retires hand-typed tenant setup | REQ-08–REQ-13, REQ-26–REQ-27 |
| **W2** | Set up chosen repos | REQ-14–REQ-16 |
| **W3** | Real readiness check + one trustworthy answer | REQ-17–REQ-23 |
| **W4** | Stay current | REQ-24–REQ-25 |

### Technical risks

| Risk | Mitigation |
|------|------------|
| Retiring hand-typed tenant setup could break anything that currently depends on it (a script, a runbook, a habit) | `OQ-8` — must be sized before implementation, not discovered after the fact |
| The programme's setup tool needs to run somewhere it isn't installed today | `OQ-2` — needs an explicit decision from whoever owns Gateflow's deployment environment |
| A wave could start on a repo that was never checked or last failed | `OQ-1` — exact behavior (block vs. warn) must be locked before implementation |
| Whether a repo that predates this initiative can ever get a fresh real check without first connecting its tenant to the programme is genuinely undecided | `OQ-9` — must be locked before implementation; leaving it ambiguous risks confusing operators either way |
| The programme's shared records could change shape over time | Fail closed on anything unexpected (REQ-06), rather than guessing |
| The tenant's credential now reads a wider set of things than before | Named explicitly (§4 Security & Privacy) as the same kind of risk already accepted for this tenant model, not a new category |
| A mismatch between the setup tool's version and what a particular repo expects | `OQ-5` — needs a compatibility policy before implementation |

### Phased rollout

- **MVP (this INIT, W0–W4):** Programme connection, catalogue-driven selection (replacing hand-typed setup entirely), automatic setup, and a real, trustworthy readiness check.
- **Later, explicit follow-ups:** Screens/dashboard (a separate initiative); resolving `OQ-9` for legacy, non-catalogue repos if that population turns out to be non-trivial; scheduled (not just on-demand) catalogue refresh, if `OQ-3` lands that way.

---

## 6. Locked decisions reference

### Carried forward from outline (D1–D11)

| ID | Decision |
|----|----------|
| D1 | A tenant can only ever gain a repo by connecting to the programme and choosing it from the shared list — at any point, including first setup. Typing a repo in directly is retired. |
| D2 | Connecting to the programme and setting up any chosen repo both reuse the exact same mechanism already built for cloning repos. |
| D3 | The programme's setup tool is only ever asked questions — never asked to fix or change anything; always pointed at Gateflow's own synced copy, never at an individual machine. |
| D4 | Choosing repos is a deliberate, saved, revisitable decision, always checked against the current catalogue. |
| D5 | Setting up repos and checking their readiness both happen one repo at a time, independently, with partial success. |
| D6 | Connecting and choosing repos use the same access credential the tenant already has. |
| D7 | The same credential used for app repos is also used to read the programme's shared records. |
| D8 | A new initiative in its own right, built on top of INIT-GATEFLOW-012's completed work. |
| D9 | This initiative takes over the readiness check from INIT-GATEFLOW-012; repos predating this initiative keep their existing answer untouched. |
| D10 | No screens or dashboard this initiative. |
| D11 | Tenant setup itself is unchanged in mechanism, but no longer comes bundled with a hand-typed repo list — every repo arrives only through D1's connect-and-choose flow. |

### Resolved this Draft PRD (from outline OQ-6/OQ-7)

| ID | Resolution |
|----|------------|
| OQ-6 | Repos a tenant already had before this initiative shipped keep their existing readiness answer exactly as it is — not reset, not forced through the new check. |
| OQ-7 | The old, direct way of adding a repo does not stick around for anyone — it's retired outright for new and existing tenants alike, since D1/D11 mean there's no ongoing case of a tenant adding a repo without connecting to the programme first. |

---

## 7. Next steps

1. Review this Draft PRD with PE/programme — in particular `OQ-9` (legacy repos' access to a fresh real check) and `OQ-8` (what currently depends on hand-typed tenant setup), since neither was fully resolved at the outline stage.
2. Impact map → programme sign-off → gateflow implementation waves per §5. Primary delivery is gateflow only; no `prayog-skills` contract change is expected.
3. Do **not** implement Gateflow code from this Draft PRD alone — follow the full detailed-design process (spec → feasibility → technical review → plan → waves), the same as every prior initiative.
4. Communicate directly with the team behind INIT-GATEFLOW-012 that this initiative takes over their readiness check and retires their original repo-add path — INIT-GATEFLOW-012 is closed, so this is a direct conversation, not a document edit.
5. Flag `OQ-2` (where the setup tool runs from) to whoever owns Gateflow's deployment environment before implementation begins.
6. Separately, not blocking this initiative: INIT-GATEFLOW-012's engineering-side wrap-up is done; the matching wrap-up in this shared workspace is still outstanding.

---

## Appendix A — Data model changes (illustrative — engineering owns exact schema)

| Change | Notes |
|--------|-------|
| Tenant setup no longer accepts a repo list | Remove the field, or keep it optional and reject any non-empty value outright (matches REQ-12) — never silently accept a provided list |
| A new record of which programme (shared repo) a tenant is connected to | One programme connection per tenant; includes what was last synced and when |
| No new field needed to distinguish "old" vs. "new" repos | A repo's existing readiness answer, or lack of one, is itself the signal — this initiative's write path simply never touches a repo record that already existed |

## Appendix B — Target capability surface (illustrative — engineering owns exact routes)

| Capability | Notes |
|------------|-------|
| Connect a tenant to its programme | New; CAP-01 |
| Read the current catalogue | New; CAP-02, returns candidates, not yet-selected status included |
| Choose/update the active repo list | New; CAP-03, rejects out-of-catalogue attempts |
| Set up chosen repos | May be automatic on selection, or a distinct call; CAP-04 |
| Run/refresh the real readiness check | New; CAP-05/CAP-06, per-repo, on selection and on demand |
| Refresh the catalogue | New; CAP-07 |
| *(changed)* Tenant setup | Existing call from INIT-GATEFLOW-012; repo list input removed/loosened per REQ-12 |

Exact routes, request/response shapes, and error body fields are engineering's to design in the next stage — not fixed here.

---

## References

- Outline: [INIT-GATEFLOW-013-outline](./INIT-GATEFLOW-013-outline.md)
- Vision: [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md)
- Predecessor: [INIT-GATEFLOW-012](./INIT-GATEFLOW-012.md) (tenant registry + workspace/branch lifecycle — fully shipped; this initiative takes over its readiness check and retires its hand-typed repo-add path)
- Code evidence (verified this session against local `gateflow`/`launchpad` checkouts): `src/models/tenant_models.py`, `src/business_services/tenant_service.py`, `src/infra_services/tenant_git_workspace_client.py`, `src/infra_services/github_pat_probe.py`, `src/infra_services/launchpad_client.py`, `src/database/postgres/schema/tenant_schema.py`, `docs/specification/as-built/implementation-status.md`; `launchpad` CLI (`status`, `--config-dir`, `--client`, `apply-harness`); `prayog-meta/config/programme.yaml`, `prayog-meta/config/service-catalog-drivestream-lab.yaml`
