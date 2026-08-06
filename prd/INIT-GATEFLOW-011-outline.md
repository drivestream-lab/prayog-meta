# INIT-GATEFLOW-011 — Day-1 visibility and GitHub reconcile (outline)

**Status:** outline · **Author:** programme PM · **Date:** 2026-08-06
**Vision:** [planning/gateflow-programme-vision.md](../planning/gateflow-programme-vision.md)
**Component:** GATEFLOW · **Type:** platform / delivery control plane
**Predecessor:** INIT-GATEFLOW-010 (engineering-lane pin tip executor parity — eng lanes run correctly end to end, but nothing about them is visible or independently checked)

> **Outline only.** Written in plain product language on purpose — this is the
> problem framing and scope lock, not the engineering contract. The Draft PRD
> (next step) will translate this into CAP-*/REQ-* tables, error tables, and
> acceptance criteria. Engineering detail routes to the impact map and the
> gateflow spec PR, same as every prior INIT.
>
> **SSOT:** remounted prayog-skills `workflow.yaml` + `delivery-contract.yaml`
> (`sdd-delivery/v2`). This INIT does not change the pin — it makes Gateflow
> able to *show* and *check* what the pin already governs.

---

## Document control

| Field | Value |
|---|---|
| Initiative ID | INIT-GATEFLOW-011 |
| Artifact | `prd/INIT-GATEFLOW-011-outline.md` (this outline); Draft PRD to follow at `./INIT-GATEFLOW-011.md` |
| Programme | prayog |
| Primary repo | drivestream-lab/gateflow |
| Supporting repos (read-only consumers, not touched) | prayog-meta (PRD/meta PR state, read only), prayog-skills (pin consume-only) |
| Explicitly **not** touched this INIT | gateflow-ops (no UI screens built here — this INIT only builds what a UI would call) |
| Target users | Developer/engineer running the delivery loop day to day; engineering lead / PE reviewing gates |
| Depends on | INIT-GATEFLOW-010 (eng lanes proven and human-gated correctly); **Wave -1 remount (§0) — hard prerequisite, gates spec-draft** |

---

## 0. Pre-flight — remount to current prayog-skills tip (prerequisite, not part of this INIT's build)

**Decision:** Before any spec work starts on INIT-GATEFLOW-011, `gateflow`, `gateflow-ops`, and `prayog-meta` are all remounted onto the current `prayog-skills` tip. Gateflow backfills onto current skills first; only then does `/spec-draft` for INIT-GATEFLOW-011 begin. This is a hard gate, not a nice-to-have — running spec work on a stale pin would produce artifacts against terminology and safeguards the SSOT has already retired.

**Why this is needed (evidence, checked 2026-08-06):** all three consumers are pinned to the *same* commit, and that commit is three commits behind the current `prayog-skills` tip on `features/rc-2`:

| Repo | Vendored commit | Current SSOT tip | Behind by |
|---|---|---|---|
| `gateflow` | `6561c7c` | `ebaa912` | 3 commits |
| `gateflow-ops` | `6561c7c` | `ebaa912` | 3 commits |
| `prayog-meta` | `6561c7c` | `ebaa912` | 3 commits |

`prayog-skills` tags `v0.5.0-rc.2` as a documented **mutable "retagged tip"** during the RC phase (its own CHANGELOG: *"Consumable tip for programmes remounting `agent_skills.ref: v0.5.0-rc.2` (resolve to the retagged tip SHA)"*). All three consumers still declare `ref: v0.5.0-rc.2` in `.harness-pin.yaml`, but their local submodule checkout never re-fetched the moved tag, so `launchpad status` reports no drift even though the tag has moved upstream.

**What the backfill picks up for every consumer:**

| Change already released on the current tip | Currently missing from all three | Why it matters |
|---|---|---|
| `live-verify` → `wave-acceptance` checkpoint rename; `wave-accepted` label as the human-approval signal | `gateflow`'s own vendored `workflow.yaml` still has node `live-verify`, not `wave-acceptance` | INIT-GATEFLOW-011's checkpoint read-out (Phase 5/9 reconcile) must speak the current terminology, not the retired one |
| `/verify` skill removed (folded into `pre-implement`'s live-smoke policy) | `.harness/skills/verify` and `.agents/skills/verify` still exist in `gateflow` and `gateflow-ops` | A developer or agent could still invoke a skill the SSOT considers deleted |
| ADR product-boundary lint (`scripts/adr_boundary_lint.py`) on the spec lane | Not present | Guards exactly the kind of ADR (e.g. "how do we read GitHub check-runs") this INIT will produce |
| `launchpad` command docs fixed (`sync-harness-*` → `apply-harness`/`status`) | Cosmetic only for these repos | Removes a dead-command trap for anyone following setup docs |

**Sequence (per repo, same shape each time):**

1. `launchpad reset-harness --repo <name> --apply` (or `--meta --apply` for `prayog-meta`) — clears the current harness materialization.
2. `launchpad apply-harness --repo <name> --apply` — remounts onto the current `v0.5.0-rc.2` tip; re-resolves `profiles/<profile>.yaml` at the new commit; drops `/verify`, adds any new skills.
3. `launchpad status --repo <name>` — confirms zero drift post-remount.
4. Spot-check each repo's own as-built/README for stale `live-verify` references and correct them.

**Order across repos:** `gateflow` first (primary engineering surface for this INIT), then `gateflow-ops` and `prayog-meta` (no ordering dependency between the latter two — they can remount in parallel with each other, just not before `gateflow`'s remount is verified clean).

**Exit condition for §0:** all three repos' `launchpad status` reports no drift, and `gateflow`'s vendored `workflow.yaml` shows `wave-acceptance` (not `live-verify`) before `/spec-draft` for INIT-GATEFLOW-011 is run.

---

## 1. Problem statement

Today, a developer can do every real step of the delivery loop — reviewing a spec, coding a wave, running verification, merging — but they can't do any of it *through* Gateflow, because Gateflow can't yet answer two basic questions:

1. **"What's going on right now?"** There is no list of initiatives, no view of which waves are done/ready/blocked, and no summary of why an automated run stopped or what to do about it. The work Gateflow already does behind the scenes (drafting specs, coding waves, closing out) is invisible the moment it finishes.
2. **"Did I actually finish what I think I finished?"** After a developer approves something in GitHub (a label, a review, a merge), there is no way to ask Gateflow to double-check that the approval is real, current, and complete before relying on it. Right now that trust is just assumed.

Without these two things, the "Gateflow runs automation → developer works in Cursor/GitHub → Gateflow checks GitHub → developer triggers next phase" loop we want for Day 1 cannot exist — there's no *front door* to start the loop, and no *safety check* to close it.

---

## 2. Proposed solution (summary)

Build the two missing pieces, and reuse the safety-check piece everywhere it's needed instead of rebuilding it per phase:

| What we're building | Solves |
|---|---|
| **A status-check capability** — Gateflow looks at a GitHub pull request right now and reports plainly whether the right approval/label/checks/reviews are actually there, on the current version of the code, or exactly what's missing | The "did I actually finish?" problem — reused at every checkpoint |
| **A visibility layer** — initiative list, wave map, run/checkpoint summaries, all in plain language | The "what's going on?" problem |

We are deliberately **not** building the visual screens themselves (buttons, pages) in this INIT — that belongs to a follow-on `gateflow-ops` initiative. This INIT builds the things a screen would need to call. We are also **not** touching anything on the product-management side (the original idea/requirements documents) — that stays a separate initiative.

---

## 3. Locked product decisions

| ID | Decision |
|---|---|
| **D1** | The status-check capability is read-only. Gateflow may look at GitHub and report back; it may never apply a label, approve, or merge on anyone's behalf. This was already a hard rule for the engineering lanes (INIT-GATEFLOW-010) and stays a hard rule here. |
| **D2** | Every status check must say *what version of the code* it checked and *when*, so a decision made five minutes ago can't be silently treated as still valid after new changes land. |
| **D3** | The status check must return **specific missing items**, not just pass/fail — e.g. "the acceptance label isn't on the PR yet" or "two required checks are still running" — so a developer knows exactly what to fix. |
| **D4** | Visibility (the initiative list, the wave map, the run summary) is a read-only reflection of state that already exists elsewhere (GitHub, the board, prior runs). This INIT does not invent new sources of truth. |
| **D5** | The product-management side of an initiative (the original PRD/requirements documents, and their own closeout) is explicitly out of scope. It depends on a different system and likely needs its own initiative. |
| **D6** | No screens. This INIT produces the things a screen calls, not the screen itself. |

---

## 4. Users and jobs-to-be-done

| User | Job to be done |
|---|---|
| **Developer/engineer** | "I want to open one place, see what's ready for me and what's waiting on me, do my review/coding/verification work, then come back and get a clear yes/no on whether I'm actually clear to move on." |
| **Engineering lead / PE** | "I want to trust that nothing moves forward on a stale or incomplete approval, and I want a clear record of when something was last checked and against what version." |

---

## 5. Scope — in (only what we are building)

Mapped to the Day-1 Human Developer Experience phases. Only phases that need something *built* are listed — phases that already work correctly as pure human/GitHub steps, or that belong to a different system, are called out in §6 (Scope — out) instead.

| Phase (Day-1 doc) | What we build |
|---|---|
| **Phase 1 — Initiative discovery** | A read-out of initiatives: name/description, whether the underlying idea is approved, which repos are affected, current stage in plain words, links to anything already in flight |
| **Phase 2 — Spec lane generation** | A read-out of what the automated spec pass produced: link to the resulting draft PR, what was generated, findings/open questions, and the exact next step (what to check out, what to run) |
| **Phase 5 — Reconcile spec completion** | The core status-check capability: confirm a spec PR's approval is real, current, and complete, or list exactly what's missing |
| **Phase 6 — Wave preparation** | A read-out of the wave map for an initiative: done / ready-to-start / blocked (and why) / active |
| **Phase 7 — Wave implementation** | A read-out of an in-progress wave's coding run, broken down by task rather than one spinner, with the resulting draft PR link |
| **Phase 9 — Reconcile wave acceptance** | The same status-check capability from Phase 5, applied to wave acceptance instead of spec approval — reuse, not rebuild |
| **Phase 10 — Automated wave closeout** | A read-out of what closeout added (lessons captured, records updated), plus a safeguard: if product code changed after a human accepted the wave, say so clearly instead of continuing silently |
| **Phase 11 — Wave approval and merge** | Reuse the Phase 5/9 status check to confirm the merge really happened, plus a nudge: if another wave is now unblocked, say so |
| **Phase 12 — Initiative completion eligibility** | A read-out of "all waves complete, ready to close" vs "waiting on wave N" |
| **Phase 13 — Application closure** | A plain-language summary of what a cleanup pass is about to delete (or kept) before/after it runs, plus reuse of the status check once its PR is merged |

---

## 6. Scope — out

| Out | Rationale |
|---|---|
| **Phase 3 — Specification engineering review** | Already works correctly, entirely human, in Cursor/GitHub. Nothing to build. |
| **Phase 4 — Specification approval and merge** | Already works correctly, entirely in GitHub. Gateflow must never participate in applying the approval or merging — that boundary stays. |
| **Phase 8 — Code and feature verification** | Already works correctly, entirely human, in Cursor/GitHub. Nothing to build. |
| **Phase 14 — Meta closure** | Depends on a different system (the product-management side) that Gateflow does not touch today. Recommend as its own, later initiative once that side is ready to be scoped. |
| **The actual UI screens/buttons** (`gateflow-ops`) | Follow-on initiative — this INIT builds what those screens will call, not the screens themselves. |
| **Any automatic label/approval/merge action** | Hard boundary, unchanged from INIT-GATEFLOW-010 — a human always makes that call in GitHub. |
| **Automatic progression on GitHub events (webhooks driving the loop)** | Day-1 is explicitly "developer asks Gateflow to check" — we should design so this can be added later without a rebuild, but we are not turning it on now. |
| **`prayog-meta` / product-management process changes** | Out of bounds for a gateflow-owned initiative. |
| **`launchpad` command-naming cleanup** | Already fixed upstream in `prayog-skills` (see §0) — no separate work needed here once remounted. |

---

## 7. Capability walkthroughs (what "done" looks like, per phase)

### Phase 1 — Initiative discovery
**Problem today:** No list exists. Knowing an initiative exists requires already knowing where to look.
**What we build:** A list of initiatives showing name, approval status, affected repos, and current stage, with a link into whatever's already running.
**Done looks like:** A developer with no prior context can correctly say what's ready and what's moving, unprompted.

### Phase 2 — Spec lane generation
**Problem today:** The automated spec pass runs correctly but produces no visible handoff.
**What we build:** A read-out of the result — PR link, findings, open questions, exact next step.
**Done looks like:** A developer goes from starting the spec lane to reviewing the right branch in Cursor within a minute.

### Phase 5 — Reconcile spec completion
**Problem today:** No way to confirm an approval is real, current, and complete before moving on.
**What we build:** A status check that inspects the live PR and reports pass, with what was checked and when, or a specific list of what's missing.
**Done looks like:** A developer can never accidentally proceed on a stale or incomplete approval.

### Phase 6 — Wave preparation
**Problem today:** No visual map of an initiative's waves.
**What we build:** A wave map: done / ready / blocked / active.
**Done looks like:** A developer sees the whole shape of remaining work at a glance.

### Phase 7 — Wave implementation
**Problem today:** The coding pass runs correctly but looks like a black box while running.
**What we build:** Task-by-task progress and a link to the resulting draft PR the moment it exists.
**Done looks like:** A developer can watch a wave unfold without leaving Gateflow.

### Phase 9 — Reconcile wave acceptance
**Problem today:** Same gap as Phase 5, at a more consequential moment (right before closeout runs).
**What we build:** Reuse of the Phase 5 status check, applied to wave acceptance evidence.
**Done looks like:** Closeout never starts on a stale or incomplete acceptance.

### Phase 10 — Automated wave closeout
**Problem today:** Closeout runs correctly but silently.
**What we build:** A read-out of what was added, plus a safeguard that flags (rather than hides) any post-acceptance code change.
**Done looks like:** A developer trusts closeout because they can see exactly what it touched.

### Phase 11 — Wave approval and merge
**Problem today:** Merge mechanics already work; nothing proactively surfaces the next unblocked wave.
**What we build:** Reuse the status check to confirm the merge, and surface "start next wave" right there when applicable.
**Done looks like:** Finishing one wave and starting the next feels like one continuous flow.

### Phase 12 — Initiative completion eligibility
**Problem today:** The "all waves done" check already runs correctly but is invisible.
**What we build:** A clear "ready to close" / "waiting on wave N" read-out.
**Done looks like:** No manual tallying of wave statuses required.

### Phase 13 — Application closure
**Problem today:** Cleanup mechanics work; no transparency into what gets deleted.
**What we build:** A plain-language before/after summary of what's deleted or kept, plus reuse of the status check once the cleanup PR merges.
**Done looks like:** A developer approves a cleanup PR fully informed, not trusting a black box.

---

## 8. Delivery waves (proposed — refined in Draft PRD / plan)

| Wave | Intent |
|---|---|
| **W0** | Build the status-check capability against a single PR (labels, checks, reviews) — read-only, no persistence yet |
| **W1** | Add the "last checked, what version" record-keeping, and the general checkpoint read-out that composes it with existing run/timeline data — this unlocks Phase 5, 9, 11 |
| **W2** | Add the initiative list/detail read-out using data Gateflow already owns (runs, board tickets) — unlocks most of Phase 1 |
| **W3** | Extend the initiative read-out with meta PR/approval state (read-only from prayog-meta) — completes Phase 1 |
| **W4** | Add the wave map read-out (done/ready/blocked/active) — unlocks Phase 6 and the "start next wave" nudge in Phase 11 |

---

## 9. Success criteria (initiative exit)

1. Given any spec or wave PR, a developer can ask Gateflow to check its status and get a specific, accurate, current answer — never a stale one.
2. Every status check records when it looked and at what version of the code.
3. A developer can list all initiatives and see approval state, affected repos, and current stage without asking a colleague.
4. A developer can see an initiative's wave map (done/ready/blocked/active) without piecing it together manually.
5. None of the above ever writes a label, approval, or merge to GitHub on its own.
6. Everything built here is consumable by a future `gateflow-ops` screen without further Gateflow changes.

---

## 10. Dependencies and non-goals for partners

| Partner | Expectation |
|---|---|
| **prayog-skills** | Consumed as-is at the current tip (post §0 remount); no pin/workflow changes needed for this INIT |
| **prayog-meta** | Read-only source for meta PR/approval state (Phase 1); no process changes requested |
| **gateflow-ops** | Not touched this INIT (beyond its own §0 remount); this INIT is the dependency that unblocks its future build |
| **launchpad** | Used for §0 remount only; not otherwise touched this INIT |

---

## 11. Risks

| Risk | Mitigation |
|---|---|
| Status check looks "stale" if GitHub state changes between check and use | Always check live at the moment of the click; never cache silently |
| Developers distrust automated cleanup (Phase 13) without transparency | Explicit before/after summary is part of scope, not a nice-to-have |
| Scope creep into meta closure (Phase 14) or UI screens | Explicit out-of-scope in §6; separate initiatives |
| Read-only meta PR bridge (Phase 1) misread as PM-process ownership | D5 and §6 make the boundary explicit |
| §0 remount skipped or done partially (e.g. only `gateflow`, not `gateflow-ops`/`prayog-meta`) | §0 exit condition requires all three clean before spec-draft starts |

---

## 12. Next steps

1. **Complete §0 remount first** — `gateflow`, `gateflow-ops`, `prayog-meta` onto the current `prayog-skills` tip; confirm `launchpad status` clean on all three. Nothing below starts until this is done.
2. Review this outline with PE / programme.
3. Expand into Draft PRD with full capability/requirement tables and error tables.
4. Impact map → meta Gate 1 → gateflow spec / implement waves per §8.
5. Do **not** implement Gateflow code from this outline alone — follow SDD (spec → feasibility → technical review → plan → waves), same as every prior INIT.
