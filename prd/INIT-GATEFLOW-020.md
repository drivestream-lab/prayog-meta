# INIT-GATEFLOW-020: Honest wave-start OpenCode

| Field | Value |
|-------|-------|
| Initiative | INIT-GATEFLOW-020 |
| Status | Draft PRD |
| Brief | `prd/INIT-GATEFLOW-020-outline.md` |
| Promoted from | `prd/reports/INIT-GATEFLOW-020-prd-think-2.md` (C2, user-authorized 2026-08-17) |
| Quality | `prd/reports/INIT-GATEFLOW-020-prd-quality.md` (C2 handover: yes) |
| Predecessor / supersede | INIT-GATEFLOW-003 **for OpenCode only** (Claude / Pi / unknown stay 003 fail-closed) |

## 1. Job and outcome

- **Job:** A `tenant_admin` (or any start caller) can begin a Spec / Implement / Closeout **lane** only with a **resolved runner and model**, get a real OpenCode hop when that pair is OpenCode + a LiteLLM-served model, and after a **failed** lane start a **new** lane with a different pair. Done-in-the-world: no stub, no start without a model, no silent harness swap.
- **Why now:** OpenCode is already choosable and still dies `(Source: User-confirmed)`; start paths besides ops (programme default, caller) would inherit that lie unless honesty is a start rule.
- **If we do nothing:** Picker, API, and defaults can name OpenCode without a valid model; a failed OpenCode lane has no honest recovery except “try Cursor in your head.”
- **Kill assumption:** Fails if a lane can be accepted without both runner and model, if OpenCode is accepted with a model not in the remembered LiteLLM-derived set (including default/API), if a failed wave is rewritten to another runner in place, or if OpenCode remains a stub.

## 2. Locked decisions

1. OpenCode is the first live non-Cursor runner. This INIT **supersedes INIT-GATEFLOW-003 for OpenCode only**. LiteLLM already in IaC is its inference gateway. This INIT does not operate LiteLLM. Prove-it on both runners is a later INIT.
2. Every lane start must resolve to a **runner and a model**. Missing either after resolution ⇒ refuse.
3. Programme lane default or caller may **supply** the pair. Defaults do not skip honesty.
4. Honesty is a **start** rule for every channel (ops picker, API, default): OpenCode is accepted only if the model is in Gateflow’s remembered LiteLLM-derived OpenCode set.
5. That pair is **wave-scoped**. This INIT does not switch runner or model per hop.
6. Empty LiteLLM catalog, missing gateway process configuration, or never-succeeded list ⇒ OpenCode cannot start. No invented enum. No silent Cursor substitute.
7. If a lane fails for any reason, the operator may start **again** with another runner + model. The failed wave is not mutated to a different harness.
8. Models are listed per runner. OpenCode’s set is LiteLLM-derived; Cursor’s is Cursor-native. LiteLLM ids must stay shareable with a later LiteLLM-consuming runner. Recorded identity is `runner` + `model_id`.
9. IaC puts LiteLLM endpoint and credential in Gateflow process configuration (lab `.env`). Not a screen. Not `tenant_admin`. No OpenCode vendor key in the 014 catalogue.
10. Stub is removed; hop actually runs OpenCode through LiteLLM. Hop must not deliver via `git commit` / `git push` / `gh`. Baton only where Gateflow already expects it. Policy not written into tenant git.
11. Cursor path unchanged when the caller **explicitly** starts Cursor with a Cursor model.
12. Remember-interval and credential-rotation vs remembered set are engineering / `OQ-*`, not invented REQs.

## 3. In scope / non-goals

### In scope

- Mandatory resolved runner+model on lane start (all channels).
- Remembered LiteLLM-derived OpenCode set; ops consumes existing surface; start is accepted or refused against that remembered set.
- Live OpenCode runner; remove stub; dispatch live runners; not-live still fail closed.
- Fail-closed OpenCode when gateway missing, catalog empty, model not in set, worker cannot run OpenCode.
- New-start recovery after failure (different pair).
- Hop isolation and stage `runner` + `model_id`.

### Non-goals (load-bearing)

- LiteLLM deploy/UI/models/compose; IaC master key as product secret.
- Pi, Claude, OpenCode-as-server, Cursor-through-LiteLLM, generic ModelGateway slot.
- Per-hop runner switching; resume-same-wave with a new harness; mixed OpenCode+Cursor hops in one wave.
- New ops screens, refresh-models, provision UI.
- Prompt-package / pin / ForgeClient edits; OpenCode policy files in tenant git.
- Streaming, cost ingest; per-programme LiteLLM keys.
- Live packaged-skill prove-it (later INIT).
- Forcing `platform_admin` to provision an OpenCode catalogue key.

## 4. Actors

| Actor | Can actually | Cannot / OQ |
|-------|----------------|-------------|
| `tenant_admin` | Supply or confirm runner+model; start a lane; after failure, start a **new** lane with another pair; start Cursor explicitly | Change an in-flight wave’s runner; configure LiteLLM; see the gateway credential; skip the model |
| Start caller (API) | Same start rules as ops | A weaker honesty bar than ops |
| IaC team | Supply process configuration; manage LiteLLM models | Gateflow product persona |
| `platform_admin` | Cursor catalogue as 014 | OpenCode vendor key as gateway |
| gateflow-ops | Present per-runner models Gateflow exposes | Call LiteLLM; mutate a failed wave’s runner |

## 5. Capabilities

| ID | Capability | Journeys covered | Notes |
|----|------------|------------------|-------|
| CAP-01 | Mandatory wave-start pair | J1, J2, J3, J7 | Runner **and** model after resolution |
| CAP-02 | Honest OpenCode start on every channel | J1, J2, J3, J4 | Remembered LiteLLM set; no ops-only honesty; unauthorized credential fail-closed (REQ-23) |
| CAP-03 | Recover by new start | J5 | Failed wave unchanged; new pair allowed |
| CAP-04 | Live OpenCode hop | J1, J6 | Stub gone; traffic through configured LiteLLM |
| CAP-05 | Isolated hop + fail closed | J6, J7 | No git/`gh`; no Cursor substitute |

## 6. Requirements

| ID | CAP | Requirement (WHAT) | Condition / event | Observable result | Evidence |
|----|-----|--------------------|-------------------|-------------------|----------|
| REQ-01 | CAP-01 | Lane start requires runner and model | Start is requested and after default/caller resolution either runner or model is absent | Start is refused; operator-visible reason that both are required | unit |
| REQ-02 | CAP-01 | Defaults may supply the pair | Programme lane default or caller provides runner and model | Start proceeds to honesty checks; defaults are not ignored | unit |
| REQ-03 | CAP-02 | OpenCode start is gated by the remembered set | Start names OpenCode (ops, API, or default) with a model not in the remembered OpenCode set, including a Cursor model id | Start refused; no hop | unit |
| REQ-04 | CAP-02 | OpenCode set is LiteLLM-derived | Gateway is configured and a list fetch has succeeded | Existing runners-and-models surface shows OpenCode with that set; ops does not call LiteLLM; no new ops screen. Depends on A-02. | unit / inspection |
| REQ-05 | CAP-02 | Empty or unconfigured OpenCode cannot start | Endpoint/credential missing, LiteLLM catalog empty, or list never succeeded | OpenCode not startable; no hardcoded OpenCode enum; visible reason | unit |
| REQ-06 | CAP-02 | Stale remembered set still gates start | LiteLLM is down after a successful fetch | Start must accept that model if it is on the remembered OpenCode set, and must refuse it if it is not | unit |
| REQ-07 | CAP-01 | Wave keeps the start pair | A wave has accepted runner R and model M | Orchestrated hops in that wave use R and M; no per-hop change this INIT | unit |
| REQ-08 | CAP-03 | Failure does not rewrite the wave’s harness | A lane/wave has failed | That wave’s runner and model stay as started; Gateflow does not switch them to another runner | unit |
| REQ-09 | CAP-03 | Operator may start again with another pair | After a failure, `tenant_admin` starts a new lane with a different live runner and a model valid for that runner | New start is allowed subject to REQ-01–REQ-05; prior failed wave is not required to change. Depends on `OQ-04`. | unit |
| REQ-10 | CAP-03 | Explicit Cursor after OpenCode failure | Previous OpenCode lane failed; caller starts Cursor with a Cursor model | Cursor hop runs as today; not a hidden fallback of the failed wave | unit |
| REQ-11 | CAP-04 | OpenCode is live | Start accepted for OpenCode with a model in the remembered set and the worker can run OpenCode | Hop runs OpenCode through the configured LiteLLM; not a stub; not success-without-run. Depends on A-03. | unit |
| REQ-12 | CAP-04 | Same skill and handoff contract as Cursor | OpenCode hop runs a packaged orchestrated skill | Skill message and handoff baton shape match Cursor hops | unit / inspection |
| REQ-13 | CAP-04 | Stage records the pair | OpenCode hop runs | Stage has runner OpenCode and the chosen model id | unit |
| REQ-14 | CAP-04 | Cursor explicit path unchanged | Caller starts Cursor with a Cursor model as today | Cursor catalogue and hop unchanged | unit |
| REQ-15 | CAP-05 | Not-live runners fail closed | Start names Claude, Pi, or unknown / not-live | Refused; no silent OpenCode or Cursor substitute | unit |
| REQ-16 | CAP-05 | No silent Cursor substitute | OpenCode cannot start (REQ-05, REQ-23) or an OpenCode hop cannot reach LiteLLM | That start or hop fails; Cursor runs only if the caller starts Cursor (REQ-10) | unit |
| REQ-17 | CAP-05 | Worker cannot run OpenCode | OpenCode would run and the worker cannot | Fail closed; not stub success | unit |
| REQ-18 | CAP-05 | No git/`gh` delivery | OpenCode hop attempts `git commit`, `git push`, or `gh` | Not the delivery success path; ForgeClient remains forge | unit |
| REQ-19 | CAP-05 | Baton location; no tenant policy file | OpenCode hop writes handoff | Baton only where Gateflow already expects it; no OpenCode policy file added to tenant git | unit / inspection |
| REQ-20 | CAP-02 | OpenCode model ids are LiteLLM identities | OpenCode models are listed and recorded | Recorded `model_id` strings equal the remembered set; no second OpenCode-only namespace | inspection |
| REQ-21 | CAP-02 | LiteLLM credential not shown | Operator or ops views start, picker, or run detail | Gateway credential is not exposed | unit / inspection |
| REQ-22 | CAP-04 | No OpenCode vendor key in 014 catalogue | OpenCode is choosable | Cursor catalogue unchanged; `platform_admin` does not store LiteLLM as an OpenCode agent key | inspection |
| REQ-23 | CAP-02 | Unauthorized LiteLLM credential fails closed | LiteLLM endpoint and credential are present but LiteLLM refuses the list or inference (unauthorized) | OpenCode is not startable and/or the hop fails; no stub success; no Cursor substitute | unit |

## 7. Negative and failure paths

| REQ | Condition | Required behavior | Why it matters |
|-----|-----------|-------------------|----------------|
| REQ-01 | Model omitted after resolution | Refuse start | “Runner only” was explicitly rejected |
| REQ-03 | Default OpenCode + stale/foreign model | Refuse | Defaults must not be a back door |
| REQ-05 | Empty catalog / missing env | OpenCode not startable | IaC owns models; Gateflow must not invent them |
| REQ-06 | LiteLLM down after a successful fetch; model on or off the remembered set | Must accept if on the set; must refuse if not | Stale set still gates start; no invented enum |
| REQ-08 | Wave failed | Do not retarget that wave to Cursor or another model | Mid-wave harness change is out of scope |
| REQ-09 / REQ-10 | Operator wants another harness | New start with a valid pair | Recovery without mutating history |
| REQ-12 | OpenCode hop runs a packaged skill | Skill message and baton match Cursor hops | Same contract, different harness |
| REQ-15 | Not-live runner | Refuse | 003 honesty (Claude / Pi / unknown) |
| REQ-16 | OpenCode hop/proxy fail | Fail that hop; no auto-Cursor | Substitute is the anti-pattern |
| REQ-17 | Binary/runtime missing | Fail closed | Else “integrated” is a stub |
| REQ-18 | Skill tries to forge via git/`gh` | Must not be success | ForgeClient owns forge |
| REQ-19 | Hop writes handoff or policy | Baton only where Gateflow expects it; no OpenCode policy file in tenant git | Isolation |
| REQ-20 | OpenCode models listed/recorded | `model_id` strings equal the remembered LiteLLM set; no OpenCode-only namespace | Shareable later; testable identity |
| REQ-21 | Operator or ops views start, picker, or run detail | LiteLLM credential is not shown | Shared IaC secret |
| REQ-22 | OpenCode is choosable | No OpenCode vendor key in 014 catalogue | Gateway is process config, not Cursor-style provision |
| REQ-23 | Credential present but LiteLLM refuses list or inference | Fail closed; no stub; no Cursor substitute | Unauthorized ≠ missing |

## 8. Contract seeds (semantic)

| ID | Provider | Consumer | Logical operation | Field meaning | Invariants | Errors |
|----|----------|----------|-------------------|---------------|------------|--------|
| CTR-01 | gateflow | gateflow-ops and other start callers | Accept or refuse lane start | Start carries resolved runner + model; OpenCode model must be in remembered OpenCode set | Pair mandatory; same honesty on every channel | Missing pair; foreign model; OpenCode not startable; not-live runner |
| CTR-02 | gateflow | gateflow-ops | List runners and per-runner models | OpenCode set = last successful LiteLLM list (possibly stale); Cursor set unchanged | Ops does not call LiteLLM | Unconfigured; empty; never succeeded; unauthorized |
| CTR-03 | LiteLLM (IaC) | gateflow | List configured models | Model identity is what LiteLLM serves | No Gateflow OpenCode enum | Unreachable; empty; unauthorized |
| CTR-04 | LiteLLM (IaC) | OpenCode hop | Inference for the start model | Same identity as remembered OpenCode set | Wave pair unchanged mid-hop | Proxy down; unknown model; auth |

## 9. NFR applicability

| Area | Requirement or N/A rationale |
|------|------------------------------|
| Security | REQ-18, REQ-19, REQ-21. Isolation; credential not on the product edge. |
| Reliability | REQ-01, REQ-05, REQ-08, REQ-16, REQ-17, REQ-23. Fail closed; no in-place harness swap. |
| Performance / capacity | N/A as a product number. Remember-interval is `OQ-02`. |
| Observability | REQ-13. Stage pair is the audit trail for retries (new waves, not rewritten ones). |
| Privacy / data handling | REQ-21. |
| Migration / compatibility | REQ-02, REQ-14, REQ-22. 014 defaults still resolve a pair; Cursor catalogue unchanged. This INIT supersedes 003 for OpenCode only. |
| Rollback / recovery | Product recovery is **new start** (REQ-09), not rollback of the failed wave’s runner. Cursor remains if started explicitly. |
| Operations / support | IaC owns LiteLLM. Support must distinguish “pair missing,” “OpenCode not startable,” and “hop failed — start again.” |

## 10. Assumptions

| ID | Assumption | Status | Dependent REQs | Default if false |
|----|------------|--------|----------------|------------------|
| A-01 | Callers besides ops (014 default, API) already exist and must obey the same start rule | accepted | REQ-01–REQ-03 | Honesty would be ops-only — rejected in this grill |
| A-02 | Existing ops picker can show a second runner’s models `(Source: User-confirmed)` | accepted | REQ-04 | If picker cannot bind OpenCode’s set: this INIT still must not add a new ops screen (follow-up INIT / PE defect — not an open product question). |
| A-03 | OpenCode can talk to this lab LiteLLM | unverified | REQ-11, CTR-04 | Later prove-it INIT; no empty success |
| A-04 | “Start again” is allowed by current wave/initiative product (new lane start) | unverified | REQ-09 | If product forbids a second start, `OQ-04` blocks recovery |
| A-05 | 014 env-Cursor ban does not forbid IaC gateway process configuration `(Source: User-confirmed)` | accepted | REQ-22 | Reopen catalogue-vs-env |

## 11. Open questions

| ID | Question | Owner | Blocking | Required-by | Default if deferred |
|----|----------|-------|----------|-------------|---------------------|
| OQ-01 | On LiteLLM credential rotation, discard remembered set immediately? | PM | no | spec-draft | Keep until remember-interval; hop may fail auth |
| OQ-02 | Remember-interval duration | PE | no | feasibility | Engineering |
| OQ-04 | After a failed wave, is a second lane start on the same initiative already allowed, or is that a hidden gap? | PE | yes | spec-draft | If not allowed, recovery (REQ-09) cannot ship without that start rule existing or a follow-up INIT |
| OQ-05 | Which INIT owns live prove-it on OpenCode and Cursor together? | PM | no | later | Not this INIT |
| OQ-06 | Does “new start” create a new wave id on the same initiative (expected) or a new initiative? | PE | no | spec-draft | Same initiative, new wave — if as-built differs, document as-built, do not invent resume-with-new-runner |

## 12. Journeys

**J1 — Happy OpenCode lane** (brief)  
Actor: `tenant_admin`. Trigger: start with OpenCode + LiteLLM model. Main: pair present; model in remembered set; hop runs; baton valid. Edge: default supplied the pair; packaged skill and baton match Cursor (REQ-12); recorded `model_id` is the LiteLLM identity from the remembered set (REQ-20). Abandon: cancel before start.

**J2 — Start without a model** (**omitted from brief**)  
Actor: caller. Trigger: runner present, model missing after resolution. Main: refuse (REQ-01). Edge: default has runner OpenCode but no model. Abandon: supply a model from that runner’s set.

**J3 — Default OpenCode, empty gateway** (**omitted from brief**)  
Actor: caller. Trigger: programme default is OpenCode; catalog empty or env missing. Main: refuse; no Cursor substitute. Edge: caller then starts Cursor explicitly (J5). Abandon: wait for IaC to add models.

**J4 — API start, no ops** (**omitted from brief**)  
Actor: API caller. Trigger: start OpenCode with a Cursor model id. Main: refuse (REQ-03). Same rule as picker. Abandon: pass a model from the OpenCode set.

**J5 — Failed lane, try another pair** (**stated this run; not in brief**)  
Actor: `tenant_admin`. Trigger: OpenCode (or any) lane failed. Main: failed wave keeps its pair; operator starts a **new** lane with another live runner + valid model (e.g. Cursor). Edge: second start still subject to mandatory pair and honesty. Abandon: stop; do not rewrite the failed wave.

**J6 — Isolation / missing runtime** (brief risk only)  
Hop must not forge via git/`gh`; worker cannot run OpenCode ⇒ fail closed. Baton only where Gateflow expects it; no tenant OpenCode policy file (REQ-19); LiteLLM credential not shown (REQ-21).

**J7 — Not-live runner named** (003, not in this brief)  
Refuse Claude/Pi/unknown; operator may new-start a live pair.

## 13. Domain terms

| Term | Meaning in this INIT | Collision with existing product? |
|------|----------------------|----------------------------------|
| Lane start pair | Resolved runner **and** model required to accept a wave | Stronger than “runner optional / model later” |
| Wave-scoped pair | That pair applies to the whole wave’s orchestrated hops | Not per-skill profiles (later) |
| Remembered OpenCode set | Last successful LiteLLM-derived model list | Not 014 agent catalogue |
| New start | After failure, a separate lane start with another pair | Not resume-with-new-runner; not in-place rewrite |
| LiteLLM gateway | IaC inference proxy in process configuration | Not vision ModelGateway slot |
| Live runner | Stub removed; hop actually runs | Not merely listed |
