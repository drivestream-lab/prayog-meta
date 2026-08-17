# INIT-GATEFLOW-020 — Live OpenCode runner via existing LiteLLM (outline)

**Status:** outline · **Author:** programme PM · **Date:** 2026-08-17  
**Draft PRD:** [INIT-GATEFLOW-020.md](./INIT-GATEFLOW-020.md)  
**Vision:** [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md) (§6 AgentRunner / ModelGateway slots · §9 H2 multi-runner · H3 LiteLLM)  
**Component:** GATEFLOW · **Type:** AgentRunner + consumed ModelGateway  
**Predecessor:** INIT-GATEFLOW-014 (JWT + platform agent catalogue — delivered); INIT-GATEFLOW-019 (operator lane start + `GET /runners` picker — delivered in app repos)  
**Related, not blocking:** INIT-GATEFLOW-016/017/018 ops console (picker already binds `GET /runners`); Pi / Claude Code runners (later INITs)

> **Outline** — problem framing and scope lock. Detailed requirements live in the
> Draft PRD.
>
> **Hard rule for this initiative:** LiteLLM is a **consumed upstream** already
> deployed on DriveStream local IaC. Configuring models in the LiteLLM UI is
> **out of scope**. Gateflow reads `GET /v1/models` and runs OpenCode against
> that endpoint. Inventing an OpenCode model enum is a failed design.

---

## Document control

| Field | Value |
|---|---|
| Initiative ID | INIT-GATEFLOW-020 |
| Artifact | `prd/INIT-GATEFLOW-020-outline.md` (this outline); Draft PRD at [`./INIT-GATEFLOW-020.md`](./INIT-GATEFLOW-020.md) |
| Programme | prayog |
| Primary repo | drivestream-lab/gateflow |
| Supporting | drivestream-lab/gateflow-ops (no LiteLLM calls; lane dialog already consumes `GET /runners`); DriveStream `iac-local` LiteLLM (read-only dependency — not edited this INIT) |
| Explicitly **not** touched this INIT | LiteLLM compose / UI / `config.yaml` `model_list` / provider keys; Pi; Claude Code; `opencode serve`; Cursor-through-LiteLLM; ops provision or refresh-models UI; prompt packages / `prayog-skills`; ForgeClient; writing `opencode.json` into tenant repos |
| Target users | Delivery operator (`tenant_admin`) — pick OpenCode + a LiteLLM model on a lane start; lab PE — set global LiteLLM URL/key in Gateflow `.env` |
| Depends on | INIT-GATEFLOW-014 catalogue + JWT (delivered); INIT-GATEFLOW-019 runner picker (delivered in apps); local IaC LiteLLM at host `http://localhost:4000` / compose `http://litellm:8000/v1` |

---

## 1. Problem statement

Operators can already choose a runner and model on Spec / Implement / Closeout.
Only **Cursor** is live. `RunOrchestrator` hard-fails when `runner != cursor`.
`OpenCodeAgentRunner` is an honest stub. `GET /api/v1/runners` returns the
Cursor enum only.

A factory picker cannot invent OpenCode models. The list must come from the
**already-deployed LiteLLM** (DriveStream `iac-local`: host
`http://localhost:4000`, in-compose `http://litellm:8000/v1`, models managed in
the Admin UI / Postgres — `config.yaml` `model_list` is empty by design).

Without this initiative, “support OpenCode” stays a stub, and ops has no honest
model list for that runner.

---

## 2. Proposed solution (summary)

| What we're building | Solves |
|---|---|
| **CAP-01 — Live OpenCodeAgentRunner** | `opencode run` in the workspace; same `run_skill` contract as Cursor |
| **CAP-02 — Dispatcher** | Route by `resolved.runner`; drop Cursor-only hard-fail |
| **CAP-03 — LiteLLM-backed list for ops** | `GET /runners` OpenCode row from `GET {LITELLM_BASE_URL}/v1/models`; cache 10 minutes in gateflow |
| **CAP-04 — Isolated hop policy** | Per-hop OpenCode config aimed at LiteLLM `/v1`; allow handoff root only; deny git commit/push/`gh` |
| **CAP-05 — Prove-it** | One short packaged-skill hop writes a valid baton; stage records `runner` + `model_id` |

**Unchanged on purpose:** prompt packages, handoff envelope, ForgeClient,
wave-start gates, LiteLLM IaC, Cursor path.

---

## 3. Locked product decisions

| ID | Decision |
|---|---|
| **D1** | Initiative id is **INIT-GATEFLOW-020** (018/019 already used in app repos). |
| **D2** | LiteLLM is a **consumed upstream**. This INIT does not deploy, compose, or operate the proxy. |
| **D3** | **Configuring models in LiteLLM is out of scope.** Empty catalog ⇒ empty OpenCode picker and OpenCode start 422. |
| **D4** | Model list is fetched from LiteLLM and returned to ops **only** via existing `GET /api/v1/runners`. Ops never calls LiteLLM. |
| **D5** | Cache that list **10 minutes** in **gateflow** (not ops). Cache key includes the credential (or a hash). TTL applies to a successful fetch. |
| **D6** | LiteLLM down: serve **last good** list if one exists (stale, with `fetched_at`). Never succeeded ⇒ fail closed. No hardcoded OpenCode enum fallback. |
| **D7** | Wave start validates `model_id` against the **cached** list. No second live LiteLLM call at start. No ops “refresh models” control. |
| **D8** | LiteLLM connection is **global config in Gateflow `.env`** (`LITELLM_BASE_URL`, and the virtual key as a global env — not a programme setting, not a catalogue field for the URL). Host default `http://localhost:4000/v1`; compose `http://litellm:8000/v1`. Do not use the IaC master key as a documented product secret; a virtual key in `.env` is the lab credential. |
| **D9** | Picker model ids are prefixed **`litellm/<LiteLLM model_name>`** so they never collide with `cursor/…`. Strip the prefix only if the proxy/OpenCode hop requires the bare name. |
| **D10** | Cursor stays Cursor-native (existing enum + Cursor catalogue credential). Not routed through LiteLLM this INIT. |
| **D11** | Invoke is **`opencode run`** (`--dir`, `--model`, `--agent build`, `--auto`). Not `opencode serve`. Not the JS SDK. |
| **D12** | Content skills still must not commit or push. OpenCode deny rules enforce that. Forge writes stay ForgeClient. |
| **D13** | No new ops screens. Lane dialog already binds `GET /runners`. Agent-catalogue provision UI is out of scope. |
| **D14** | Do not write `opencode.json` into the tenant workspace. Inject hop policy via `OPENCODE_CONFIG_CONTENT` (or equivalent env) for that process only. |

---

## 4. Users and jobs-to-be-done

| User | Job to be done |
|---|---|
| **Delivery operator (`tenant_admin`)** | "I pick OpenCode and a model from the list we actually have, start a lane, and get a real hop — not a stub." |
| **Lab PE** | "I point Gateflow at the LiteLLM we already run by setting `.env`. I add/remove models in the LiteLLM UI, not in this INIT." |
| **Factory** | "Same packaged prompt and handoff baton as Cursor; different harness; tokens go through the existing gateway." |

---

## 5. Scope — in

| Area | What we build |
|---|---|
| OpenCode adapter | Live `OpenCodeAgentRunner`; mark `opencode` `implemented=true` |
| Dispatcher | `RunOrchestrator` + SlotValidator for any live runner, not only `cursor` |
| Global LiteLLM env | Read `.env` (`LITELLM_BASE_URL` + virtual key); fail closed if missing when an OpenCode hop or OpenCode list is requested |
| Runners API | OpenCode models from LiteLLM `/v1/models`; 10-minute cache; Cursor enum unchanged |
| Hop isolation | Gateway base URL, handoff `external_directory` = `GATEFLOW_HANDOFF_ROOT` only, deny `git commit` / `git push` / `gh`; disable OpenCode skill discovery / autoupdate / share |
| Worker | Pin `opencode` version; fail closed if the binary is missing |
| Proof | Unit tests + one short live packaged-skill prove-it |

## 6. Scope — out

| Area | Why |
|---|---|
| LiteLLM deploy, compose, UI, `model_list`, provider keys | Already IaC; D2–D3 |
| Pi, Claude Code, `opencode serve` | Next runner INITs / later optimization |
| Cursor-through-LiteLLM | Cursor SDK path stays Cursor-native (D10) |
| Streaming console, cost ingest, OpenCode stats | Not required to prove the hop |
| Tenant-repo `opencode.json` | Would leak factory policy into git (D14) |
| Ops provision / refresh-models UI | Picker is enough; D7 / D13 |
| Prompt-package or pin edits | Same rendered message as Cursor |
| Authorize→resume walker | Unrelated |

---

## 7. Integration sketch (PRD will refine)

```
ops lane dialog
  → GET /api/v1/runners
       cursor: existing enum
       opencode: cache 10m ← GET $LITELLM_BASE_URL/v1/models (Bearer from .env)

wave start runner=opencode model_id=litellm/<name>
  → OpenCodeAgentRunner
       opencode run --dir $WORKSPACE --model … --auto --agent build
       OPENCODE_CONFIG_CONTENT → baseURL $LITELLM_BASE_URL
  → overwrite $GATEFLOW_HANDOFF_ROOT/{run_id}/handoff.md
  → HandoffReader (unchanged)
```

**Lab prerequisite (not a CAP):** LiteLLM UI already has ≥1 chat model; the `.env`
virtual key can list it (`curl http://localhost:4000/v1/models` non-empty).

---

## 8. Success (outline-level)

- `GET /runners` includes `opencode` with models from LiteLLM (cached ≤10 min).
- Lane start with `runner=opencode` enqueues; the stub path is gone.
- Prove-it hop writes a valid handoff baton; stage has `runner=opencode` and the chosen `model_id`.
- Cursor hops unchanged.
- No LiteLLM IaC change in the delivery.
- LiteLLM URL and key are read from Gateflow `.env` only.

---

## 9. Risks to carry into the PRD

- Baton path is **outside** the workspace — `external_directory` must allow only `GATEFLOW_HANDOFF_ROOT`; `--auto` without that pin is too wide.
- Empty LiteLLM catalog looks like a Gateflow bug; docs must say models are configured in the IaC UI, out of this INIT.
- OpenCode extra params vs Chat Completions — local proxy already has `drop_params: true`; still verify on prove-it.
- Worker must ship a pinned `opencode` binary; missing binary = fail closed.
- Global `.env` key is lab-wide (all programmes share the gateway). Per-programme LiteLLM keys are a later INIT if needed.

---

## 10. Next steps

1. Outline locked (D1–D14); Draft PRD at [`INIT-GATEFLOW-020.md`](./INIT-GATEFLOW-020.md).
2. Impact map (gateflow primary; gateflow-ops monitor — picker consumer only).
3. Gate 1 / eng spec in `drivestream-lab/gateflow`.
