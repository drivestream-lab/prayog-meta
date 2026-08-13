# INIT-GATEFLOW-013 — Programme-first onboarding, choosing which repos matter, and real readiness checks (outline)

**Status:** outline · **Author:** programme PM · **Date:** 2026-08-08
**Vision:** [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md)
**Component:** GATEFLOW · **Type:** onboarding experience
**Predecessor:** INIT-GATEFLOW-012 (tenant registration and the clone/branch/run lifecycle) — **already delivered in full**, and this initiative builds directly on top of it.
**Related, not blocking:** INIT-GATEFLOW-016 (the operations dashboard, formerly INIT-GATEFLOW-004 — retired) — will eventually display what this initiative builds; Launchpad (the programme's own setup/readiness tool) — this initiative only asks it questions, never lets it make changes

> **Outline only.** Written in plain product language on purpose — this is the
> problem framing and scope lock, not the engineering contract. The Draft PRD
> (next step) will translate this into detailed requirements, error handling,
> and acceptance criteria.

---

## Document control

| Field | Value |
|---|---|
| Initiative ID | INIT-GATEFLOW-013 |
| Artifact | `prd/INIT-GATEFLOW-013-outline.md` (this outline); Draft PRD to follow at `./INIT-GATEFLOW-013.md` |
| Programme | prayog |
| Primary repo | drivestream-lab/gateflow |
| Supporting repos | prayog-meta (the programme's shared list of repos and settings — we only read from it, never change it); launchpad (the programme's own setup/readiness tool — we only ask it questions, never ask it to change anything) |
| Explicitly **not** touched this INIT | The operations dashboard (a later initiative will build screens on top of what we deliver here); anything that changes or repairs a repo's setup — this initiative only checks and reports |
| Target users | The person who administers a tenant (now picks repos from a list instead of typing them by hand); developers running day-to-day work — unaffected, except that the readiness check behind the scenes gets more trustworthy |
| Depends on | INIT-GATEFLOW-012's tenant setup and its repo clone/refresh mechanism, both already delivered. This initiative also takes over responsibility for the readiness check INIT-GATEFLOW-012 already put in place, which today runs live in production. |

---

## 1. Problem statement

INIT-GATEFLOW-012 gave every tenant a credential, a list of repos, a place to work, and a working clone/branch/run lifecycle. It also put a first readiness check in place: before a wave of work can start on a repo, Gateflow checks that the repo has the basic setup files present, and refuses to start if not. That check already runs in production today.

Four things are still missing or thin once a programme has more than a handful of repos:

1. **Where do candidate repos come from?** Registering a tenant today means typing in every repo by hand, one at a time — including the very first repo a brand-new tenant ever gets. Nothing helps the operator discover what the programme actually owns.
2. **How does an operator pick which repos matter?** A programme's shared repo list may be long; a given tenant may only care about some of them. Today it's all-or-nothing — either type in everything you want, or nothing.
3. **How does Gateflow stay current as the programme grows?** Once a tenant is set up, its repo list is frozen at whatever was typed in at the time. If the programme adds a new repo next month, nothing tells the tenant it exists.
4. **Is the existing readiness check actually checking enough?** Today's check only confirms that some setup files exist — it doesn't verify the repo is genuinely healthy, current, or free of drift. The programme's own setup tool already knows how to do a much deeper check, but running that full check on every single wave would be slow and mostly unnecessary.

Without a better onboarding experience, every new repo still needs a person to already know about it and type it in by hand. And the readiness check that exists today only answers "does a file exist," not "is this repo actually in good shape" — a much weaker promise than what's already possible.

---

## 2. Proposed solution (summary)

Give tenants a guided way to discover and choose their repos from the programme's own records — the *only* way to get a repo at all, going forward — and make the existing readiness check meaningfully better:

| What we're building | Solves |
|---|---|
| **Connect to the programme** — a tenant links itself to the programme's shared records once. | Today there's no notion of "which programme a tenant belongs to" beyond whatever repos were typed in |
| **See the full list** — Gateflow reads the programme's own repo list and shows every repo it could work on. | Nobody has to discover repos by asking around or reading a config file themselves |
| **Choose what matters** — the operator picks a subset of that list as the tenant's active repos; that choice is saved and can be changed later. | All-or-nothing manual entry doesn't scale as a programme grows |
| **Set everything up** — every chosen repo gets cloned and made ready to work on automatically, using the same mechanism already built for this. | Choosing a repo and having it actually usable are two different things today |
| **A real readiness check** — instead of just checking that a file exists, Gateflow asks the programme's own setup tool to do a proper readiness check on each chosen repo, and reports each result separately. | Today's check is a thin, one-file guess |
| **One trustworthy readiness answer** — the result of that real check becomes the single, official answer to "is this repo ready," replacing today's thinner check. | Two different checks giving two different answers to the same question |
| **Stay current** — a tenant can refresh its view of the programme's repo list at any time and see anything new without starting over. | Today, once set up, a tenant never finds out about new repos on its own |

---

## 3. Locked product decisions

| ID | Decision |
|---|---|
| **D1** | A tenant can only ever gain a repo by connecting to the programme first and choosing it from the shared list — **at any point, including the very first time a tenant is set up**. Typing a repo in directly is retired, not just discouraged. |
| **D2** | Setting up the programme's own records and setting up any chosen repo both use the exact same mechanism already built for cloning repos — nothing new or separate is invented for "the programme is special." |
| **D3** | The programme's setup tool is only ever *asked questions* by this initiative — never asked to fix or change anything. It's pointed at the programme's own copy of its records, never at anything stored on an individual person's computer. |
| **D4** | Choosing which repos matter is a deliberate, saved decision by the operator — not automatic, not "everything the programme owns." That choice can be revisited any time, and is always checked against the programme's *current* list, not a stale copy. |
| **D5** | Setting up repos and checking their readiness both happen **one repo at a time, independently** — one repo having a problem never blocks the others, and each repo gets its own clear result. |
| **D6** | Connecting to the programme and choosing repos both use the same access credential the tenant already has — no new kind of login or credential is introduced. |
| **D7** | The same credential a tenant already uses to work on its repos is also used to read the programme's shared records — no separate, more limited credential is created just for that. This matches a choice already made and accepted in INIT-GATEFLOW-012 (one credential, one shared risk). |
| **D8** | This is a new initiative in its own right, not an extension of INIT-GATEFLOW-012. INIT-GATEFLOW-012's own work is complete; this initiative adds a discovery-and-readiness layer on top of it. |
| **D9** | **This initiative takes over the readiness check from INIT-GATEFLOW-012.** The real, programme-tool-based check becomes the one true answer, replacing the current file-presence check, for every repo a tenant works with going forward — since D1/D11 mean every repo now arrives through the programme connection, there is no ongoing case of a tenant having a repo without ever having connected. The one thing this doesn't automatically answer is what happens to **repos a tenant already had before this initiative existed** — those were added the old way, before it was retired, and are grandfathered as-is rather than reset (see §12, `OQ-6`). This is a handover of responsibility, communicated directly to the team that built INIT-GATEFLOW-012 — their original plans and documents aren't rewritten. |
| **D10** | No dashboard or screens are built in this initiative — everything ships as something other systems can call. Screens are a later initiative's job. |
| **D11** | Setting up a tenant itself — its credential, its workspace, its basic account — still happens through the same simple setup call INIT-GATEFLOW-012 built. What changes is that call **no longer comes bundled with a hand-typed repo list**. From this initiative onward, a tenant is set up empty of repos, and every repo it ever works on — the first one and every one after — arrives only by connecting to the programme and choosing from its list (D1). This tightens INIT-GATEFLOW-012's original setup call; it does not add a second, optional path alongside it. |

---

## 4. Users and jobs-to-be-done

| User | Job to be done |
|---|---|
| **Tenant administrator** | "I want to connect to our programme once, see every repo it owns, and pick the ones I actually care about — without typing each one in by hand, and without missing it when a new one gets added later." |
| **Developer doing day-to-day work** | Unaffected — the way their work gets started and run doesn't change. They benefit only indirectly: the readiness check standing between them and starting a wave becomes a real, trustworthy answer instead of a thin guess. |

---

## 5. Scope — in (only what we are building)

| Area | What we build |
|---|---|
| Connect to the programme | A tenant links to the programme's shared records once, and Gateflow keeps its own current copy of them |
| See the full list | Read the programme's shared repo list and turn it into a list of choices |
| Choose what matters | Let the operator pick and save a subset of that list; always checked against the current list |
| Set everything up | Clone and prepare every chosen repo, using the existing mechanism |
| A real readiness check | Ask the programme's setup tool to check each chosen repo, one at a time |
| One trustworthy readiness answer | The real check's result becomes the single record of "is this repo ready," replacing the thinner check |
| Stay current | Let a tenant refresh and see new repos without starting over |

---

## 6. Scope — out

| Out | Rationale |
|---|---|
| Any screens or dashboard | A later initiative's job — this one only builds what other systems can call |
| Letting the programme's setup tool fix or change anything | We only ever ask it questions this initiative — never ask it to repair or install anything |
| Choosing a repo that isn't on the programme's own list | Deliberately closed off — no way around the shared list this initiative |
| Letting a tenant add a repo without connecting to the programme, at any point — including first setup | D1/D11 — retired, not just discouraged; there is no "add a repo directly" path left anywhere, for new or existing tenants |
| Retroactively re-checking or resetting repos a tenant already had before this initiative shipped | D9/`OQ-6` — those are grandfathered as-is; this initiative doesn't force existing tenants to reconnect or re-earn their standing |
| Any finer-grained permissions per repo | INIT-GATEFLOW-012 already decided access is per-tenant, not per-repo — this initiative doesn't change that |
| Changing how day-to-day work actually runs | This initiative is about getting set up, not about how work happens once it's running |
| A separate, more limited credential just for reading the programme's records | Same credential as everything else — no new credential type |
| Relying on anything stored on an individual person's computer | The programme's setup tool is always pointed at Gateflow's own copy of the records |

---

## 7. Capability walkthroughs (what "done" looks like, per capability)

### Connect to the programme
**Problem today:** A tenant has no sense of "which programme it belongs to" beyond whatever repos were typed in for it.
**What we build:** A one-time connection that keeps Gateflow's own current copy of the programme's shared records.
**Done looks like:** A tenant connects once, and Gateflow always has an up-to-date copy of the programme's records from then on.

### See the full list
**Problem today:** Nobody reads the programme's shared repo list automatically — a person has to open it and copy repos in by hand.
**What we build:** A way to turn that shared list into a set of choices an operator can see.
**Done looks like:** An operator sees every repo the programme owns, without opening a file themselves.

### Choose what matters
**Problem today:** Setting up a tenant is all-or-nothing and manual — there's no "pick a few repos out of many," and no repo, not even the first one, arrives any other way today.
**What we build:** A selection step that saves a chosen subset and always checks it against the current shared list. This becomes the *only* way any tenant, new or existing, ever gains a repo.
**Done looks like:** An operator picks exactly the repos they want, can change their mind later, and can never accidentally pick — or type in — something the programme doesn't actually own.

### Set everything up
**Problem today:** Choosing a repo and having Gateflow actually able to work on it are two separate things — nothing connects the two today.
**What we build:** Automatic setup of every chosen repo, using the same mechanism already built for this.
**Done looks like:** Every chosen repo is ready to work on, with no second way of setting up repos anywhere in the system.

### A real readiness check
**Problem today:** The readiness check Gateflow runs today only confirms a couple of setup files exist — it can't tell if a repo is actually current, correctly configured, or drifting out of sync.
**What we build:** A call to the programme's own setup tool to do its full, proper readiness check on each chosen repo.
**Done looks like:** Every chosen repo gets a real, trustworthy readiness verdict — not a guess based on one file's existence.

### One trustworthy readiness answer
**Problem today:** The record of "is this repo ready" already exists and already controls whether work can start — today it's set by the thin, file-only check.
**What we build:** The real check's result becomes that record's value, taking over from the thin check.
**Done looks like:** "Is this repo ready" reflects a real, current check for every repo added from now on — since every repo now arrives through the programme connection, there's no ongoing gap. Repos a tenant already had before this initiative shipped are left exactly as they were, deliberately, not silently reset.

### Stay current
**Problem today:** A tenant's view of what the programme owns is frozen at whatever it looked like the last time it was read.
**What we build:** A refresh option that re-reads the programme's records on demand.
**Done looks like:** When the programme adds a new repo, a tenant sees it the next time they refresh — without repeating the whole setup process.

---

## 8. Delivery waves (proposed — refined in Draft PRD / plan)

| Wave | Intent |
|---|---|
| **W0** | Connect to the programme + keep a current copy of its records + turn that into a list of choices |
| **W1** | Let an operator choose and save which repos matter — reject anything not on the programme's list |
| **W2** | Set up every chosen repo automatically |
| **W3** | Run the real readiness check on each chosen repo, and make it the one trustworthy answer — with a clear plan for the switch-over |
| **W4** | Let a tenant refresh and discover newly added repos |

---

## 9. Success criteria (initiative exit)

1. A tenant can connect to its programme and see every repo it owns; a repo can no longer be typed in by hand, at any point, for a new or existing tenant.
2. An operator can choose and save a subset of repos, and that choice is always checked against the programme's current list.
3. Every chosen repo gets set up automatically, using the one existing mechanism for doing so — no second way of setting up a repo exists anywhere.
4. The real readiness check runs on every chosen repo, and a problem with one repo never affects the others — each gets its own clear result.
5. "Is this repo ready" is answered exclusively by the real check for every repo added from this initiative onward; repos a tenant already had before this initiative shipped are deliberately left as they were, not silently reset.
6. When the programme adds a new repo, a tenant sees it the next time they check, without repeating the whole setup process.

---

## 10. Dependencies and non-goals for partners

| Partner | Expectation |
|---|---|
| **prayog-meta** | We only read its shared repo list and settings — nothing about how that's maintained changes because of this initiative |
| **launchpad** | We only ever ask its existing readiness tool questions — nothing about the tool itself changes, and we never ask it to fix or install anything |
| **The future operations dashboard** | Not touched this initiative; it's a later initiative's job to build screens on top of what we deliver here |
| **INIT-GATEFLOW-016** (formerly INIT-GATEFLOW-004, retired) | Not touched by this initiative; the coordination point flagged in INIT-GATEFLOW-012's own notes is now resolved — INIT-GATEFLOW-016 composes this initiative's existing catalogue/selection/readiness endpoints client-side, no new endpoint requested of 013 |
| **The team behind INIT-GATEFLOW-012** | A direct heads-up on two things: this initiative takes over responsibility for their readiness check (a real, currently-used capability), and it also retires their original way of adding a repo to a tenant (typing it in directly) in favor of the programme-connection flow. Their existing documents aren't changed; how existing tenants' already-added repos are treated (`OQ-6`) is locked as "left as-is," not decided unilaterally to mean something more disruptive. |

---

## 11. Risks

| Risk | Mitigation |
|---|---|
| The programme's shared records could change shape or move over time | Lock down exactly what we expect to read, and refuse to proceed if it doesn't look right, rather than guessing |
| The programme's setup tool has to run somewhere it isn't already installed | Needs an explicit decision from whoever manages that environment before building — flagged as an open question |
| A tenant could start work on a repo that was never checked, or last failed its check | Exact behavior (block it, or allow it with a warning) needs to be locked before building — flagged as an open question |
| This initiative takes over a check that's already running and already deciding whether real work can start today | Locked (`OQ-6`): repos a tenant already had before this initiative shipped keep their existing readiness answer, untouched — no forced re-check, no forced reconnect |
| Retiring the direct-add path could break anyone or anything that depends on adding a repo the old way (e.g. a script, a runbook, an existing habit) | Needs an explicit look at what currently calls the old setup-with-repos path before it's turned off, flagged as a new risk to size in the Draft PRD |
| The same credential now reads both the programme's records and every chosen repo — one leaked credential now has a wider reach than before | Same kind of risk already accepted in INIT-GATEFLOW-012; the wider reach is called out plainly in the next, more detailed document rather than left unsaid |
| The version of the programme's setup tool available to Gateflow might not match what a particular repo expects | Needs an explicit compatibility policy before building — flagged as an open question |

---

## 12. Open questions

| ID | Open question | Status |
|---|---|---|
| OQ-1 | If a chosen repo was never checked, or its last check failed, should starting work on it be blocked outright, or allowed with a warning? | Open — must lock before the detailed plan |
| OQ-2 | Where does the programme's setup tool actually run from — is it built into Gateflow's environment ahead of time, or fetched when needed? | Open — needs a decision from whoever manages that environment |
| OQ-3 | Should refreshing the programme's repo list happen automatically on a schedule, only when a person asks for it, or both? | Open |
| OQ-4 | Do repos with nested sub-projects need any extra credential handling, or does the existing credential already cover them? | Open |
| OQ-5 | What should happen if the version of the programme's setup tool available to Gateflow doesn't match what a particular repo expects? | Open — needs a compatibility policy before building |
| OQ-6 | For repos a tenant already had before this initiative shipped, what happens to their existing readiness answer? | **Resolved** — left exactly as-is; not reset, not forced through the new check (D9). New repos never have this ambiguity, since D1/D11 mean every new repo comes through the programme connection from day one. |
| OQ-7 | Does the old, direct-add way of getting a repo need to stick around for any tenant? | **Resolved** — no. D1/D11 retire it outright, for new and existing tenants alike; there's no ongoing case where a tenant can add a repo without connecting to the programme. |
| OQ-8 *(new)* | What today actually calls the old "set up a tenant with a repo list" path — scripts, runbooks, habits — that would break once it's retired? | Open — needs to be sized before the Draft PRD locks the exact cutover date/mechanism |

---

## 13. Next steps

1. Review this outline with PE / programme — especially that this initiative **retires** INIT-GATEFLOW-012's original way of adding a repo (typing it in directly), for new and existing tenants alike, not just adds an alternative alongside it.
2. Expand into a Draft PRD with full detailed requirements. Primary delivery stays within Gateflow alone — consistent with how INIT-GATEFLOW-012 turned out, no changes are expected to the shared skills/process repository; the programme's records and setup tool already exist as they are.
3. Impact map → programme sign-off → implementation waves per §8.
4. Do **not** start building from this outline alone — this still needs to go through the full detailed-design process, the same as every prior initiative.
5. Communicate directly with the team behind INIT-GATEFLOW-012 that this initiative takes over their readiness check and retires their original repo-add path. Since that initiative is already wrapped up, this is a direct conversation, not an edit to their documents.
6. Flag the "where does the setup tool run from" question (`OQ-2`) to whoever manages Gateflow's environment — it needs a real decision, not an assumption.
7. Before locking the exact cutover mechanism in the Draft PRD, check what currently depends on the old repo-add path (`OQ-8`) — scripts, runbooks, anything that would break the moment it's retired.
8. Separately, not blocking this initiative: INIT-GATEFLOW-012's own wrap-up on the engineering side is done; the matching wrap-up here in the programme's shared workspace is still outstanding.
