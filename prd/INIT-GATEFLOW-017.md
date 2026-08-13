# INIT-GATEFLOW-017: One human, many programmes, one login

| Field | Value |
|-------|-------|
| Initiative | INIT-GATEFLOW-017 |
| Status | Draft PRD |
| Brief | `prd/INIT-GATEFLOW-017-outline.md` |
| Promoted from | `prd/reports/INIT-GATEFLOW-017-prd-think-3.md` (C3, user-authorized 2026-08-13) |

## 1. Job and outcome

- **Job:** When `platform_admin` looks after programmes, a **human** is already an **identity** in the factory (name, email, password) with role `tenant_admin`, including with **zero programmes**. `platform_admin` grants or refuses that identity’s entry to a programme without minting a second login. `platform_admin` can answer “who can enter programme X?” without opening a delivery screen. A `tenant_admin` signs in once, enters each programme they were granted, and runs that programme (include catalogue repos, waves, metrics).
- **Why now:** INIT-GATEFLOW-014’s create-login-and-bind-to-one-programme is the identity model today. This INIT **removes** that feature. INIT-GATEFLOW-016 still describes “invite a teammate.” The lab is not live; the 014 bind is dead and is deleted, not wrapped.
- **If we do nothing:** Another programme still means another login. `platform_admin` cannot see who can enter which programme without a delivery screen or the database. Invite and create+bind stay as two membership stories. The job we locked cannot exist.
- **Kill assumption:** Fails if 014 create+bind still creates membership, or if we ship identity screens while **one human = one programme**, or if invite still creates entry.

**OST — one outcome chosen.** Desired outcome: one human, many programmes, one login; only `platform_admin` grants or refuses. Forks not chosen: keep 014 1:1 bind and only add a list; let `tenant_admin` grant entry; a separate IdP/SSO. Chosen solution: **enter identity, then grant programme** (two acts) plus delete 014 bind and purge invite.

## 2. Locked decisions

1. **Job** is one human, many programmes, one login. An identity exists without a programme. `platform_admin` grants or refuses entry. `tenant_admin` does not grant. There is no invite.
2. **Roles:** only `platform_admin` and `tenant_admin`. Entering a human always creates role `tenant_admin`. It never creates `platform_admin`. Seeded `platform_admin` remains the factory seed.
3. **Identity** is name, email, and password. Email is the unique login. `platform_admin` sets the password at entry.
4. **Two acts:** (1) `platform_admin` enters identity — no programme. (2) `platform_admin` grants that existing identity entry to a programme — no new login, no new password. A second programme is act 2 again.
5. **014 identity bind is removed and deleted.** Creating a `tenant_admin` by minting a login and binding it to exactly one programme in one act is gone. Not a compatibility path. Not dual-model.
6. **Many programmes:** one identity may be granted more than one programme.
7. **How they work:** after sign-in, a `tenant_admin` sees only programmes they were granted, **enters** one, and does 013/016 work there. They may enter another they were granted. Existing rule: **one active wave per repo**.
8. **No invite.** Purge 016 “invite a teammate” and the historic handle-attach that cannot sign in. One membership story only.
9. **Detach:** `platform_admin` removes entry on that programme. Identity remains. Other grants remain. Not a programme wipe.
10. **Suspend / unsuspend.** Suspend: they cannot sign in and cannot keep acting, including reads on an open sign-in. Unsuspend: they can sign in again; grants unchanged.
11. **Set password:** `platform_admin` sets a new password. Previous sign-in for that identity must stop working.
12. **In-flight work:** detach or suspend does **not** cancel waves already running. That identity cannot **start or authorize** further work (detach: in that programme; suspend: anywhere). After suspend, an open sign-in cannot continue any product act, including reads. After detach, that programme is not enterable (reads included).
13. **Out this INIT:** delete identity; change email after entry; self-serve password; extra programme roles; a separate IdP/SSO; rewriting 016 delivery except removing invite. Programme wipe stays the existing 014 wipe act.
14. **Zero programmes:** that identity can sign in and see they belong to no programme. No catalogue include, waves, or metrics until a grant.
15. **Surfaces:** `platform_admin` enters humans, grants/detaches, suspends, sets password, and onboards programmes (meta repo → parse/store → show catalogue) in the console. `tenant_admin` uses the console for delivery by entering a programme they were granted. `platform_admin` does not include repos or run waves. `tenant_admin` does not see other identities or the factory identity list.
16. **Nomenclature:** say `platform_admin`, `tenant_admin`, identity, human, programme. Do not say ops, factory operator, lab operator, or invite as a product act.

## 3. In scope / non-goals

### In scope

- Identity: `platform_admin` enters name, email, password; role `tenant_admin`; unique email; factory list; find by name or email.
- Membership: grant from the existing identity list; detach; many programmes per identity; visibility of who can enter which programme (`platform_admin` only).
- Sign-in once; enter each granted programme; keep 013/016 work (catalogue include, waves, metrics).
- Suspend, unsuspend, `platform_admin`-set password with previous sign-in stopped.
- Delete 014 create+bind. Purge invite / handle-attach that is not a login.
- Fail closed: unknown identity cannot be granted; unknown programme cannot be granted; entry without name or password is refused; `tenant_admin` cannot grant; `platform_admin` cannot run delivery; an identity cannot enter a programme they were not granted.

### Non-goals (load-bearing)

- Invite / `tenant_admin` “add a teammate.”
- Delete identity; change email after entry.
- Self-serve password change.
- Extra programme roles beyond `tenant_admin`.
- A separate identity provider or SSO.
- Rewriting 016 Mission Control delivery (repos, waves, metrics, board, scorecard) except removing invite.
- Programme wipe / un-onboard of a programme (stays the existing wipe act).
- `platform_admin` including repos, starting or authorizing waves, or operating programme metrics as `tenant_admin`.
- Encrypting stored secrets (014 plaintext-accepted posture unchanged unless engineering raises it).
- A company people directory product or display-name-as-SSO.

## 4. Actors

| Actor | Can actually | Cannot / OQ |
|-------|----------------|-------------|
| `platform_admin` | Onboard a programme and its meta repo; parse, store, and show the repo catalogue; enter identities (name, email, password); find identities; grant/detach programme entry; suspend/unsuspend; set password; see who can enter which programme | Include repos, start/authorize waves, or operate metrics as delivery. Cannot be created by entering a human (Lock 2). |
| `tenant_admin` | Sign in with email + password; see programmes they were granted; enter one and include repos, run waves, look at metrics; do the same in another programme they were granted | Enter identities; grant/detach; invite; see or manage the factory identity list; see other identities on a programme; enter a programme they were not granted. |
| Identity with zero programmes | Sign in; see that they belong to no programme | Start delivery. |
| Suspended identity | Identity: nothing product-facing until unsuspended. `platform_admin` may still grant or detach (REQ-24). | Sign in or continue a current sign-in. |

## 5. Capabilities

| ID | Capability | Journeys covered | Notes |
|----|------------|------------------|-------|
| CAP-01 | Enter and find identities | J1, J6 | Name + email + password; unique email; role `tenant_admin` |
| CAP-02 | Grant and refuse programme entry | J1, J5, J10, J13 | Grant from existing identity; detach; many programmes; who-can-enter |
| CAP-03 | Suspend, unsuspend, and set password | J4, J8, J9 | Sign-in and further acts (including reads) stop; waves already running do not cancel |
| CAP-04 | `tenant_admin` works every programme they were granted | J2, J3, J14 | One sign-in; enter a programme; keep 013/016 delivery |
| CAP-05 | One membership story; 014 bind and invite gone | J7, J11, J12 | Delete create+bind; purge invite |
| CAP-06 | Programme onboard and catalogue stay as they are | J1 | Keep 014 onboard; catalogue show; include remains `tenant_admin` |

## 6. Requirements

| ID | CAP | Requirement (WHAT) | Condition / event | Observable result | Evidence |
|----|-----|--------------------|-------------------|-------------------|----------|
| REQ-01 | CAP-01 | `platform_admin` enters a human with name, email, and a password, with no programme required | Valid entry by `platform_admin` | Identity exists on the factory list with that name and email; role is `tenant_admin`; they can later sign in with that email and password; they belong to no programme yet | unit / live |
| REQ-02 | CAP-01 | Email is the sign-in identifier and is unique in the factory | Second entry uses an email that already exists | Refused (duplicate email); no second identity; first identity unchanged | unit / live |
| REQ-03 | CAP-01 | Login identifier must be an email | Entry or sign-in identifier is empty, has no `@`, or has no domain part | Refused (not an email); no identity created (entry) / no sign-in (sign-in) | unit |
| REQ-04 | CAP-01 | `platform_admin` can list factory identities and find an identity by name or email | `platform_admin` searches the entered list | Identities whose name contains the query (case-insensitive) or whose email equals the query (case-insensitive exact) are shown; no match shows an empty result, not an error | live / inspection |
| REQ-05 | CAP-01 | Entering a human creates `tenant_admin`, not `platform_admin` | Identity entry succeeds | New identity cannot perform `platform_admin` acts; seeded `platform_admin` is unchanged | unit / live |
| REQ-06 | CAP-02 | `platform_admin` grants an existing factory identity entry to a programme | Identity exists; programme exists; not already granted that programme | Identity may enter that programme after sign-in; no new login is created; password is not collected | unit / live |
| REQ-07 | CAP-02 | Grant of an identity already granted that programme is idempotent | Repeat grant of the same identity to the same programme | Success; still one grant; identity unchanged | unit |
| REQ-08 | CAP-02 | An identity that has not been entered cannot be granted | Grant names an email/identity that is not on the factory list | Refused (unknown identity); no grant | unit / live |
| REQ-09 | CAP-02 | One identity may be granted more than one programme | Grant the same identity a second programme | Both grants exist; one email; one password; they can enter either programme | unit / live |
| REQ-10 | CAP-02 | `platform_admin` detaches an identity from a programme | Detach of an existing grant | That grant is gone; identity remains; other grants remain; programme remains | unit / live |
| REQ-11 | CAP-02 | `platform_admin` can see who can enter a given programme, and which programmes a given identity can enter, without opening a delivery screen | `platform_admin` opens identity or programme membership | Membership is visible as identities ↔ programmes | live / inspection |
| REQ-12 | CAP-03 | Suspended identity cannot sign in and cannot continue product acts | `platform_admin` suspends an identity | Sign-in refused (suspended); any current sign-in cannot continue any product act, including reads; grants unchanged | unit / live |
| REQ-13 | CAP-03 | Unsuspend restores sign-in; grants unchanged | `platform_admin` unsuspends | Identity can sign in and work every programme they are still granted | unit / live |
| REQ-14 | CAP-03 | `platform_admin` sets a new password; previous sign-in stops working | Password set for that identity | They can sign in only with the new password; prior sign-in cannot continue | unit / live |
| REQ-15 | CAP-03 | Detach or suspend does not cancel waves already running | Detach or suspend while a wave is in flight on that programme’s repo | In-flight wave continues; that identity cannot start or authorize further work (detach: in that programme; suspend: anywhere). After detach, that programme is not enterable (reads included). After suspend, an open sign-in cannot continue any product act, including reads. | unit / live |
| REQ-16 | CAP-04 | After sign-in, a `tenant_admin` sees only programmes they were granted and reaches delivery by entering one | Sign-in with at least one grant | They can include repos, run waves, and look at metrics in that programme; they cannot enter a programme they were not granted (not granted) | live |
| REQ-17 | CAP-04 | The same identity can run delivery in two programmes they were granted | Identity is granted two programmes with different repos | They can start work in both; existing **one active wave per repo** still holds | live |
| REQ-18 | CAP-04 | A signed-in identity with zero programmes cannot run delivery | Sign-in with no programme grant | They are signed in; no programme delivery is available | live |
| REQ-19 | CAP-04 | `tenant_admin` can still include catalogue repos, run waves, and look at metrics in a programme they were granted | Granted identity enters that programme | 013/016 delivery acts succeed; invite is absent | live / inspection |
| REQ-20 | CAP-05 | There is no invite act | `tenant_admin` or any actor attempts invite / historic teammate-attach | Act is gone; no membership created that way | live / inspection |
| REQ-21 | CAP-05 | Membership is not created by minting a login bound to one programme in the same act | Actor attempts 014-style create+bind (new email + password as grant) | Refused or impossible; identity entry and grant remain separate; that 014 act is gone | unit / live |
| REQ-22 | CAP-05 | `tenant_admin` cannot enter identities, grant, detach, suspend, or set password | `tenant_admin` attempts those acts | Refused (wrong actor); 0 state change | unit / live |
| REQ-23 | CAP-05 | `platform_admin` cannot include repos, start or authorize waves, or operate programme metrics | `platform_admin` attempts those acts | Refused (wrong actor); 0 state change | unit / live |
| REQ-24 | CAP-02 | A suspended identity may still be granted or detached; suspend only blocks sign-in and product acts (including reads) | Grant or detach while suspended | Membership changes; they still cannot sign in until unsuspended | unit |
| REQ-25 | CAP-01 | Name is required at identity entry and is shown on the factory list | Entry without a name | Refused (missing name); no identity | unit |
| REQ-26 | CAP-04 | `tenant_admin` does not see other identities or the factory identity list | Signed-in `tenant_admin` opens delivery | No factory identity list; no roster of other identities on the programme | live / inspection |
| REQ-27 | CAP-06 | `platform_admin` still onboards a programme and its meta repo, stores the parsed catalogue, and can show that catalogue | Programme onboard by `platform_admin` | Catalogue is visible to `platform_admin`; include-into-delivery remains a `tenant_admin` act after grant (REQ-19) | live / inspection |
| REQ-28 | CAP-02 | Grant or detach naming a programme that is not onboarded is refused | Grant or detach names a programme that does not exist | Refused (unknown programme); 0 membership change | unit / live |
| REQ-29 | CAP-01 | Entry without a password is refused | Entry without a password | Refused (missing password); no identity | unit |
| REQ-30 | CAP-01 | After set, password is never returned on list, search, or membership views | `platform_admin` lists, searches, or views membership | Password is not shown or returned | live / inspection |

## 7. Negative and failure paths

| REQ | Condition | Required behavior | Why it matters |
|-----|-----------|-------------------|----------------|
| REQ-02 | Duplicate email | Refuse (duplicate email); existing identity untouched | Two logins for one human is the failure mode this INIT exists to stop |
| REQ-03 | Identifier is empty, has no `@`, or has no domain part | Refuse (not an email) | Lock 3: login is email |
| REQ-05 | Entry succeeds | New identity cannot perform `platform_admin` acts; seeded `platform_admin` unchanged | Lock 2: entering a human is not `platform_admin` |
| REQ-08 | Grant of unknown identity | Refuse (unknown identity) | Enter-then-grant; no silent create |
| REQ-10 | Detach last programme | Identity remains; zero programmes; they can still sign in (unless suspended) | Identity ≠ grant |
| REQ-12 | Suspend while a sign-in is open | Current sign-in cannot continue any product act, including reads | “Cannot sign in” is not enough if the tab stays live |
| REQ-15 | Detach during an in-flight wave | Wave continues; they cannot start/authorize in that programme; that programme is not enterable (reads included) | Waves belong to the programme/repo, not the identity |
| REQ-16 | Enter a programme they were not granted | Refused (not granted) / not available | Grant is the gate |
| REQ-18 | Zero programmes | Signed in; no delivery | Entry-without-programme is real |
| REQ-20 | Invite | Gone | Lock 8 |
| REQ-21 | Create+bind grant | Gone | Lock 5 |
| REQ-22 | `tenant_admin` grants entry | Refuse (wrong actor) | Lock 1 |
| REQ-23 | `platform_admin` runs delivery | Refuse (wrong actor) | Role split |
| REQ-24 | Grant or detach while suspended | Membership changes; they still cannot sign in | Suspend is not a membership freeze |
| REQ-25 | Entry without a name | Refuse (missing name); no identity | Name is required |
| REQ-26 | `tenant_admin` looks for other identities | Not shown | Lock 15 |
| REQ-28 | Grant or detach of a programme that is not onboarded | Refuse (unknown programme); 0 membership change | Programme must exist before grant |
| REQ-29 | Entry without a password | Refuse (missing password); no identity | Password is required at entry |
| REQ-30 | List, search, or membership view returns or displays a password | Not shown; 0 leak | Password is write-only after set |

## 8. Contract seeds (semantic)

| ID | Provider | Consumer | Logical operation | Field meaning | Invariants | Errors |
|----|----------|----------|-------------------|---------------|------------|--------|
| CTR-01 | gateflow | gateflow-ops | Enter identity; list/search identities; suspend; unsuspend; set password | Identity: name, email (login), suspended or not, role `tenant_admin`. Password is set, never read back (REQ-30). | Email unique. Entry does not create a programme grant. Entry is not `platform_admin`. | Duplicate email; not an email; missing name; missing password; actor is not `platform_admin` |
| CTR-02 | gateflow | gateflow-ops | Grant programme entry; detach; list who can enter a programme; list programmes an identity can enter | Grant: this identity may enter this programme as `tenant_admin`. | Identity must already exist. Grant does not set password. Many grants per identity. Detach is not programme wipe. | Unknown identity; unknown programme; actor is not `platform_admin` |
| CTR-03 | gateflow | gateflow-ops | Sign-in | Email + password → a current sign-in as that identity | Suspended identity cannot obtain a sign-in. After suspend, an open sign-in cannot continue any product act, including reads. After password set, prior sign-in is unusable. | Invalid credentials; suspended |
| CTR-04 | gateflow | gateflow-ops | Resolve which programmes this sign-in may enter | Only grants of the signed-in identity | `platform_admin` sign-in does not become delivery. `tenant_admin` cannot enter a non-granted programme. `tenant_admin` does not receive the factory identity list. | Not granted; wrong actor for the act |

## 9. NFR applicability

| Area | Requirement or N/A rationale |
|------|------------------------------|
| Security | Only `platform_admin` performs identity and grant acts (REQ-22). Delivery stays `tenant_admin` (REQ-23). Password is never shown after set (REQ-30). Suspend and password-set end the prior sign-in (REQ-12, REQ-14). |
| Reliability | Membership change must not cancel in-flight waves (REQ-15). Enter/grant/detach either take effect or refuse with a named reason; no half-created identity (REQ-02, REQ-08, REQ-28, REQ-29). |
| Performance / capacity | N/A — lab-scale identity list; no programme volume target this INIT. |
| Observability | Named refusal reasons for duplicate email, unknown identity, unknown programme, not granted, suspended, wrong actor, missing name, missing password, not an email. `platform_admin` can inspect membership without a delivery screen (REQ-11). |
| Privacy / data handling | Name and email are factory identity. Password is write-only after set (REQ-30). `tenant_admin` does not see other identities (REQ-26). |
| Migration / compatibility | 014 create+bind and 016 invite are not product paths (REQ-20, REQ-21). That 014 bind is deleted, not left as compatibility (Lock 5, OQ-01 default). 014 PRD update is OQ-04. |
| Rollback / recovery | Unsuspend reverses suspend. Detach reverses grant. Password-set cannot restore the old password (`platform_admin` sets a new one). No delete-identity this INIT. |
| Operations / support | `platform_admin` enters humans and grants entry in the console (Lock 15). Support path for a locked-out `tenant_admin` is `platform_admin`-set password or unsuspend, not self-serve. |

## 10. Assumptions

| ID | Assumption | Status | Dependent REQs | Default if false |
|----|------------|--------|----------------|------------------|
| A1 | Programme onboard from meta, catalogue parse/store/show, and `platform_admin` seeing that catalogue stay as they are except identity/grant | accepted | REQ-23, REQ-27 | This INIT does not redesign programme create |
| A2 | One active wave per repo remains true and is not reopened here | accepted | REQ-17 | If false, delivery concurrency is a different INIT |
| A3 | Seeded `platform_admin` remains the only factory-admin creation path | accepted | REQ-05 | Do not add “enter a `platform_admin`” |
| A4 | Name is a display label, not unique | accepted | REQ-01, REQ-04, REQ-25 | If unique names are required, that is a new REQ |
| A5 | At least one human will need a second programme before membership data is treated as production-precious | accepted (kill line) | REQ-09, REQ-17 | Stop; do not ship screens on 1:1 bind |
| A6 | Enter-then-grant is acceptable `platform_admin` cost (extra step vs 014 one-call) | accepted | REQ-01, REQ-06 | If `platform_admin` refuses the split, the INIT has failed its own job — do not collapse grant back into create |

## 11. Open questions

| ID | Question | Owner | Blocking | Required-by | Default if deferred |
|----|----------|-------|----------|-------------|---------------------|
| OQ-01 | Cutover of any leftover 014 bind rows in a lab database (lab is not live; bind is dead and deleted) | PE | no | spec-draft | Wipe leftover bind rows; no dual model; no compatibility create+bind. Product truth is REQ-21. |
| OQ-02 | After this INIT is promoted, 016’s invite stories in `prd/INIT-GATEFLOW-016.md` need a document update | PM | no | later | Track as follow-on `/update-documents`; 017 product truth is “no invite” (REQ-20). |
| OQ-03 | May `platform_admin` grant the seeded `platform_admin` email as a `tenant_admin` on a programme? | PM | no | spec-draft | No — entering a human creates `tenant_admin` only (REQ-05); seeded `platform_admin` is not grantable as `tenant_admin`. |
| OQ-04 | After this INIT is promoted, 014’s create+bind (REQ-15) and programme-bound JWT (REQ-06, REQ-31) in `prd/INIT-GATEFLOW-014.md` need a document update | PM | no | later | Track as follow-on `/update-documents`; 017 product truth is REQ-21 / REQ-09 / REQ-16; no dual model. JWT programme-binding is engineering, not leftover 014 product. |

## 12. Journeys

**J1 — `platform_admin` stands up a programme with an existing identity (main).** `platform_admin` enters identity (name, email, password) if needed → onboards programme from meta and sees catalogue → finds identity on the factory list → grants entry. Edge: already granted → idempotent. Abandon: unknown email or unknown programme → refuse grant.

**J2 — `tenant_admin` works two programmes.** Identity is granted P1 and P2 (different repos) → signs in once → enters P1, includes a repo, starts a wave → enters P2, starts a wave. Edge: same repo already has a wave → existing one-wave-per-repo rule. Abandon: not granted P2 → cannot enter.

**J3 — Zero programmes (brief omitted).** Identity entered, never granted → signs in → no programme delivery. Edge: `platform_admin` later grants → J1 continues. Abandon: they leave.

**J4 — Suspended `tenant_admin` (brief omitted).** `platform_admin` suspends → sign-in refused; open sign-in cannot continue any product act, including reads; in-flight wave continues. Edge: unsuspend → they work remaining grants; grant or detach while suspended → membership changes (REQ-24). Abandon: left suspended.

**J5 — Detach during a wave (brief omitted).** Wave running on a repo in P1 → `platform_admin` detaches the identity from P1 → wave continues; they cannot enter P1 (reads included); they cannot start/authorize in P1; other programmes unchanged.

**J6 — Duplicate email.** `platform_admin` enters a second human with the same email → refused (duplicate email). Edge: entry without a name or without a password → refused; identifier empty / no `@` / no domain → refused.

**J7 — Invite is gone (brief asked for teammate; T1 purged it).** `tenant_admin` looks for invite / teammate-attach → not present; membership unchanged.

**J8 — Password set.** `platform_admin` sets a new password → prior sign-in dead; identity signs in with the new password only. Edge: factory list and membership views do not show the password (REQ-30).

**J9 — Unsuspend.** `platform_admin` unsuspends → sign-in works; grants as they were.

**J10 — Who can enter X.** `platform_admin`, without opening delivery, sees identities on a programme and programmes for an identity. Inspection: passwords are not shown (REQ-30).

**J11 — `tenant_admin` tries to grant entry.** `tenant_admin` tries grant/enter-identity → refused (wrong actor).

**J12 — 014 create+bind.** Actor tries to create a login and bind a programme in one act → not a product path; that act is gone. Edge: entering a human creates `tenant_admin` only; seeded `platform_admin` unchanged (REQ-05).

**J13 — Detach last programme.** Only grant removed → identity remains; zero programmes; can still sign in (J3).

**J14 — `tenant_admin` looks for a roster.** They do not see other identities or the factory list (REQ-26).

## 13. Domain terms

| Term | Meaning in this INIT | Collision with existing product? |
|------|----------------------|----------------------------------|
| Identity | A factory record: name, email (login), password. Role `tenant_admin`. Exists with zero or more programme grants. | 014 “user identity” was created as a programme bind. This INIT splits them. |
| Human | The being `platform_admin` enters. Not a second entity besides identity. | Brief said “people directory” — not this. |
| Email | The unique sign-in identifier. Empty, no `@`, or no domain part is not an email. | 014 login identifier was not required to be an email. |
| Name | Display label `platform_admin` uses on the factory list. Not unique. Find is case-insensitive contains. | 014 had no name. |
| Grant | The fact that this identity may enter this programme as `tenant_admin`. Created by grant, removed by detach. | Not 014 create+bind. Not 016 invite. |
| Detach | Remove a grant. Not programme wipe. | New. |
| Suspend | Identity cannot sign in and cannot keep acting, including reads. Grants stay. | New. |
| Unsuspend | Reverse suspend. | New. |
| Invite | Not a term. Purged. | 016 teammate-attach. |
| `platform_admin` | Factory role: programmes, identities, grants, suspend, password. Does not run delivery. | Same hat as 014; identity model is not. |
| `tenant_admin` | Delivery role for programmes they were granted. | 014 bound this role to one programme at login create. |
| Programme onboard | `platform_admin` creates the programme from meta and shows catalogue. | Keep; do not confuse with identity entry or repo include. |
| Identity entry | `platform_admin` creates the identity (this INIT). | |
| Repo include | `tenant_admin` includes a catalogue repo in delivery (013/016). | |
| Factory list | The list of entered identities `platform_admin` searches. Name find is case-insensitive contains; email find is case-insensitive exact. | Not a directory product. Not historic handle-attach. |
