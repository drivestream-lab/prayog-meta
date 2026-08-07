# INIT-GATEFLOW-012 — Programme registry and workspace/branch lifecycle (outline)

**Status:** outline · **Author:** programme PM · **Date:** 2026-08-07
**Vision:** [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md)
**Component:** GATEFLOW · **Type:** platform / execution substrate
**Predecessor:** INIT-GATEFLOW-010 (engineering-lane pin tip executor parity — this INIT extends that executor's workspace assumptions to multi-repo, multi-programme, cloud execution)
**Related, not blocking:** INIT-GATEFLOW-011 (Day-1 visibility) — independent read-only layer, no dependency either direction

> **Outline only.** Written in plain product language on purpose — this is the
> problem framing and scope lock, not the engineering contract. The Draft PRD
> (next step) will translate this into CAP-*/REQ-* tables, error tables, and
> acceptance criteria. Engineering detail routes to the impact map and the
> gateflow/prayog-skills spec PRs, same as every prior INIT.

---

## Document control

| Field | Value |
|---|---|
| Initiative ID | INIT-GATEFLOW-012 |
| Artifact | `prd/INIT-GATEFLOW-012-outline.md` (this outline); Draft PRD to follow at `./INIT-GATEFLOW-012.md` |
| Programme | prayog |
| Primary repos | drivestream-lab/gateflow (implementation); drivestream-lab/prayog-skills (new forge/contract actions) |
| Explicitly **not** touched this INIT | gateflow-ops (onboarding UI is a follow-on initiative — no screens built here; programme onboarding happens via direct API call in this INIT) |
| Target users | Programme admin/operator (registers programmes, credentials, repos); developer/engineer running waves (benefits transparently — no git plumbing to think about) |
| Depends on | INIT-GATEFLOW-010 (eng lanes proven and human-gated correctly) |

---

## 1. Problem statement

Today, Gateflow's contract (ADR-010) assumes someone else already did the git
setup: cloned the target repo, checked out the correct ref, and handed
Gateflow an absolute folder path to work in. Gateflow itself never runs
`git clone`, `git checkout`, or `git branch` — it trusts that folder is
already correct. `(Source: verified against gateflow code this session —
`ForgeClient` has no local clone/checkout method; `SpecWaveStartRequest`/
`WaveClosingStartRequest` require caller-supplied `workspace_path`)`

That assumption has two consequences as the programme scales toward running
on cloud, across multiple repos:

1. **No multi-programme, multi-repo story.** Gateflow's GitHub identity
   (`GithubSettings`) is a single, global setting for the whole service —
   there is no way to register a second programme with its own repos and its
   own credential without touching global config. "Which repo, whose
   credentials, where's the workspace" all have to be answered by something
   outside Gateflow today, and nothing exists that answers them.
2. **Branches never get cleaned up.** Every wave branch and spec branch ever
   created still exists in its target repo, merged or not — nothing has ever
   deleted one. Over time, a repo's branch list stops being a reliable signal
   of what's actually in flight — "branch explosion."

Without an owner for repo/branch lifecycle, moving Gateflow to run on cloud,
across many repos and possibly many programmes, doesn't have a clean seam —
someone still has to pre-clone and pre-checkout by hand, which doesn't scale
past today's single-repo reality.

---

## 2. Proposed solution (summary)

Give Gateflow the ability to behave like an independent developer toward any
repo it's told to operate on:

| What we're building | Solves |
|---|---|
| **Programme registry** — a Programme has a PAT credential, a repo list, a workspace root, and a board. Users attach to a Programme (not to individual repos); once attached, they can operate on any repo the Programme owns. Onboarding happens via direct API call this INIT — no UI. | "Which repo, whose credentials, where" has no owner today |
| **Repo clone/refresh** — Gateflow clones a repo the first time it's asked to work on it; refreshes (fetches) it in place on subsequent uses. | No pre-made folder assumption |
| **Branch create-or-reuse** — a new wave forks from the target repo's current `develop`; a continuation reuses that wave's existing branch/PR. | Matches how a human developer actually starts vs. resumes work |
| **Branch purge on close** — once a wave merges, Gateflow deletes its own branch. Built and tested this INIT, but **not activated live** — the pin/contract changes so Gateflow *can* honor it; the default flow doesn't trigger it yet. | Branch explosion — deliberately, cautiously |
| **Harness-readiness check** — before operating on a newly registered repo, verify it has the harness scaffolding installed; flag non-readiness if not. | Prevents Gateflow producing work that doesn't match a repo's actual conventions |
| **One-active-path-per-repo** — sequencing enforced strictly at the repo level (one initiative, one wave, at a time, per repo); different repos run fully in parallel. | Tightens an existing precondition rather than inventing new locking |

---

## 3. Locked product decisions

| ID | Decision |
|---|---|
| **D1** | Gateflow owns its own working-directory lifecycle per use case and makes its own git calls (clone, branch, delete) — it behaves like an independent developer toward the repos it operates on, not a caller-bound path. This reverses ADR-010's caller-supplied-workspace assumption for repos managed this way. |
| **D2** | A new wave (not a continuation) forks from the target repo's current `develop` tip. A continuation (re-entering an existing wave) reuses the existing wave's PR branch — no re-fork. |
| **D3** | Merge always targets `develop` (unchanged from today). |
| **D4** | Branch purge happens on wave close (post-merge), to control branch explosion and keep repo state predictable. |
| **D5** | Build this as a **capability, not an activated behavior**. Ship the workspace/branch lifecycle service, the new pin contract entries, and tests — but do not wire it into the live default flow yet. The one required live change now is the `workflow.yaml`/`delivery-contract.yaml` update so Gateflow's pin-reading code *can* honor the new node/action shape once turned on later — not a second spec pass. |
| **D6** | Before Gateflow operates on a repo, it checks the repo is harness-enabled (launchpad harness materialized). If not, it flags non-readiness rather than proceeding. |
| **D7** | One combined initiative, **backend only**. Programme registry (credentials, repo list, workspace root, board) and workspace/branch lifecycle ship together as data model + APIs — not a UI. Programme onboarding happens via direct API call this INIT. |
| **D8** | Per-programme GitHub credential is **PAT only** for now. GitHub App installation per programme (the existing "preferred" mode) is explicitly deferred. |
| **D9** | Access control boundary is **User ↔ Programme**, not User ↔ Repo. A user attached to a programme can operate on / manage any repo in that programme's repo list. No finer-grained per-repo ACL in this INIT — deliberately simple. |
| **D10** | Sequentiality is enforced at the repo level, as a **repo → initiative → wave chain — exactly one path active at a time per repo**. Across different repos, full parallelism is expected and fine. This tightens the existing `NO_CONCURRENT_RUN` precondition from wave/PR/issue-scoped to repo-scoped — not a new mechanism. Direct consequence: no locking or per-wave worktree isolation is needed — one shared working directory per repo is sufficient. |

---

## 4. Users and jobs-to-be-done

| User | Job to be done |
|---|---|
| **Programme admin/operator** | "I want to register my programme's PAT and repo list once, so Gateflow can start working on any of those repos without me hand-preparing a folder for every run." |
| **Developer/engineer running waves** | "I want Gateflow to figure out on its own whether I'm starting fresh or continuing, and to keep the repo's branch list from turning into a graveyard — without me thinking about git plumbing." |

---

## 5. Scope — in (only what we are building)

| Area | What we build |
|---|---|
| Programme registry | Data model + API to register a programme: PAT credential, repo list, workspace root, board reference |
| Repo clone | First-use clone of a registered repo into its shared working directory |
| Repo refresh | Fetch-in-place on subsequent use (not re-clone) |
| Branch create-or-reuse | New wave → fork from `develop`; continuation → reuse existing wave branch/PR |
| Branch purge (dormant) | Delete-branch capability, pin-declared, tested — not wired into the live default flow |
| Harness-readiness check | Verify a registered repo has harness scaffolding before Gateflow operates on it; flag non-readiness otherwise |
| Repo-level sequencing | Tighten `NO_CONCURRENT_RUN` from wave/PR/issue scope to repo scope |

---

## 6. Scope — out

| Out | Rationale |
|---|---|
| `gateflow-ops` onboarding UI ("Mission Control") | Follow-on initiative, after this one ships the APIs it would call |
| GitHub App installation per programme | PAT only this INIT (D8); App mode deferred |
| Per-repo ACL within a programme | D9 — User ↔ Programme is the only boundary this INIT builds |
| Activating branch purge live | D5 — capability ships dormant; activation is a separate, later decision |
| Any new concurrency/locking mechanism | D10 — repo-level exclusivity is enough; no per-wave isolation needed |
| Changing how merges happen | Merge stays human-only at `wave-signoff`, unchanged |

---

## 7. Capability walkthroughs (what "done" looks like, per capability)

### Programme registry
**Problem today:** Gateflow's GitHub identity is one global setting for the whole service — there's no way to onboard a second programme with its own repos/credential.
**What we build:** An API to register a programme (PAT, repo list, workspace root, board) and attach users to it.
**Done looks like:** A programme admin can register a brand-new programme and its repos via one API call, with no global config change.

### Repo clone/refresh
**Problem today:** Gateflow assumes a human already cloned and checked out the repo into a folder it's handed.
**What we build:** Gateflow clones a registered repo itself on first use, and refreshes it in place afterward.
**Done looks like:** A newly registered repo is usable by Gateflow without anyone manually preparing a folder.

### Branch create-or-reuse
**Problem today:** No distinction exists between "start fresh" and "continue existing work" at the git level — that's entirely on the caller today.
**What we build:** New wave forks from `develop`; continuation reuses the existing wave's branch/PR.
**Done looks like:** Gateflow picks the right branch behavior automatically, matching what a careful human developer would do.

### Branch purge (built, dormant)
**Problem today:** Every wave/spec branch ever created still exists, merged or not — no cleanup mechanism exists anywhere.
**What we build:** A branch-delete capability, tested and pin-declared, triggered on wave close — but not wired into the live default flow.
**Done looks like:** The capability exists and is provably correct in tests/verify scripts; zero live branch deletions happen until a separate activation decision is made.

### Harness-readiness check
**Problem today:** Nothing verifies a repo is set up for Gateflow to operate in before Gateflow starts working in it.
**What we build:** A pre-flight check for harness scaffolding on any newly registered repo.
**Done looks like:** A repo without the harness installed is flagged "not ready" rather than silently operated on.

### Repo-level sequencing
**Problem today:** The existing "no concurrent run" check only blocks a second run on the *same* wave/PR/issue — two different waves or initiatives could run concurrently on the same repo.
**What we build:** Broaden that check to repo scope: one initiative, one wave, at a time, per repo.
**Done looks like:** Two initiatives can never run on the same repo simultaneously; two initiatives on two different repos run with no conflict at all.

---

## 8. Delivery waves (proposed — refined in Draft PRD / plan)

| Wave | Intent |
|---|---|
| **W0** | Programme registry: data model + API (register programme — PAT, repo list, workspace root, board) — foundation for everything else |
| **W1** | Repo clone (first use) + refresh (subsequent use) — consumes W0's repo list/credential |
| **W2** | Branch create-or-reuse (new wave vs. continuation) — consumes W1's workspace |
| **W3** | Harness-readiness check — gate before operating on a newly cloned repo |
| **W4** | Repo-level sequencing — broaden `NO_CONCURRENT_RUN` to repo scope |
| **W5** | Branch purge capability — built + pin-declared, dormant (not activated) |

---

## 9. Success criteria (initiative exit)

1. A programme can be registered via API with PAT + repo list + workspace root + board, without touching global config.
2. First-use clone and subsequent refresh work without a human pre-preparing a folder.
3. A new wave forks from `develop`; a continuation reuses the existing wave branch — both provably correct.
4. A repo without harness scaffolding is flagged non-ready, not silently operated on.
5. Two initiatives cannot run concurrently on the same repo; two initiatives on different repos can, with no conflict.
6. The branch-purge capability exists, is tested, and is pin-declared — but issues zero live deletions until explicitly activated in a later decision.

---

## 10. Dependencies and non-goals for partners

| Partner | Expectation |
|---|---|
| **prayog-skills** | New forge actions (branch create/reuse/purge; possibly clone/refresh representation) added to `delivery-contract.yaml`/`workflow.yaml` — consumed by `gateflow`; this is a **contract change**, not consume-only, unlike INIT-GATEFLOW-011 |
| **gateflow-ops** | Not touched this INIT; the onboarding UI is a distinct, later initiative that will consume these APIs |
| **prayog-meta** | Hosts this PRD only; no process change |

---

## 11. Risks

| Risk | Mitigation |
|---|---|
| Reopening `delete_branch` reverses a boundary three prior INITs (008, 010, and `prayog-skills`' own gap list) deliberately closed | Mitigated by D5 — capability ships dormant, not activated |
| Credential storage for per-programme PATs isn't designed yet (plaintext env var today, for one global tenant) | Must be resolved as a locked decision before feasibility — flagged as `OQ-1` |
| Local git execution (clone/fetch) is new attack surface — different credential type (deploy key / fine-grained token) than `ForgeClient`'s REST/GraphQL auth | Needs explicit security review before implementation; flagged as `OQ-4` |
| Scope creep into `gateflow-ops` UI work | Explicit non-goal (§6); separate, later initiative |
| Board/project mapping ambiguity (one board per programme vs. per repo) | Flagged as `OQ-3`, locked in Draft PRD before implementation |

---

## 12. Open questions

| ID | Open question | Status |
|---|---|---|
| OQ-1 | How are per-programme PAT credentials stored securely (encrypted Postgres column vs. external secrets manager)? | Open — must lock before feasibility |
| OQ-2 | Does registering a programme's repo list eagerly verify PAT access via a GitHub API call, or is that deferred to first use? | Open |
| OQ-3 | How does a programme's "board" map onto the existing `BoardService` — one board per programme, or per repo within a programme? | Open |
| OQ-4 | Is local git execution (clone/fetch/checkout) new code inside Gateflow, or a distinct adapter — and what credential type (deploy key vs. fine-grained token) does it use? | Open |
| OQ-5 | Deployment topology: does Gateflow's compute have a persistent disk between runs (fetch-in-place works as-is), or is it scale-to-zero (needs a shared/remote cache)? | Open — leans toward persistent per current cloud target, not blocking |
| OQ-6 | Exactly how does "build but don't activate" (D5) get expressed in the pin — a node declared but never wired to a live outcome edge, or wired in behind a settings flag that defaults off? | Open — Draft PRD must lock one mechanism |

---

## 13. Next steps

1. Review this outline with PE / programme.
2. Expand into Draft PRD with full `CAP-*`/`REQ-*` tables spanning **gateflow** and **prayog-skills** (a contract change, not consume-only).
3. Impact map → meta Gate 1 → gateflow + prayog-skills spec/implement waves per §8.
4. Do **not** implement Gateflow or prayog-skills code from this outline alone — follow SDD (spec → feasibility → technical review → plan → waves), same as every prior INIT.
5. Flag to PE/programme explicitly that this reopens a boundary (`delete_branch`) that INIT-GATEFLOW-008 and INIT-GATEFLOW-010 deliberately closed — so it reads as a considered decision, not scope creep.
