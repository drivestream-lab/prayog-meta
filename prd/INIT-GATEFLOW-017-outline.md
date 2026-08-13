# INIT-GATEFLOW-017 — People and programmes (working title)

**Status:** outline · **greenfield** · **Not a PRD** · **Date:** 2026-08-13  
**How to use:** Stakeholder brief for `/prd-think` / `/grilling`. Nothing here is locked. The grill must produce job, non-goals, and decisions. Do **not** invoke `/prd`. Do **not** treat this file as a table of contents to fill.

**Initiative id:** INIT-GATEFLOW-017 (id only — scope is not decided)

---

## The ask (as we heard it)

Ops is stumbling. Standing up another lab still means **another login**. There is no one place to see **who can get into which programme**. Programme admins keep asking for a way to **add a teammate** themselves.

Someone said: *“We need a people directory and a platform console. And fix sign-in so one person can work two labs.”*

That sentence is a **proxy**. It is not the product. Challenge it.

---

## What already exists (facts, not decisions)

These are as-built. The grill decides what we do about them.

- **INIT-GATEFLOW-014 (delivered):** Creating a programme admin **creates a login and binds it to exactly one programme in one call**, and may start their session. A person has no display name. The same login **cannot** belong to two programmes.
- **INIT-GATEFLOW-016:** Mission Control — onboard repos, waves, runs, scorecard, forge, board. It has talked about **“invite a teammate.”** That work is not shipped as a decided design here.
- **INIT-GATEFLOW-013 (delivered):** Repo catalogue / onboard / deboard — today this is programme-admin work.
- **Two codebases:** `gateflow` (orchestrator / identity APIs) and `gateflow-ops` (ops UI). How work splits is not decided in this file.
- **Lab is not live.** A breaking change is possible; that does not mean we must break.
- **INIT-012 `tenant_users`** exists in history. Whether it is the people directory is **not** assumed.

Vision notes live in `planning/gateflow-programme-vision.md` if you need programme context. Do not copy a section number into a requirement without reading it.

---

## Who seems to hurt (unconfirmed)

| Voice | What they said | Treat as |
|---|---|---|
| Ops / “platform” | Cannot see all people or all programmes in one place; cannot look after the factory without opening delivery screens | Hypothesis |
| Programme admin | Wants to add someone to *their* lab without waiting on ops | Hypothesis — may be the wrong product |
| Someone on two labs | Tired of a second email / second password | Hypothesis — may not be real this horizon |

Roles (`platform_admin`, `tenant_admin`, person-with-no-programme) are **names in play**, not a decided model.
