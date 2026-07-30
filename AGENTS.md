# Agent guide (drivestream-lab meta)

<!-- launchpad:harness-start -->
## Harness (managed by launchpad — do not edit)

Installed under **`.harness/skills/<skill>/`** (hub) mirrored to **`.agents/skills/`** and **`.claude/skills/`**:

- Community: `/prd` @ awesome-copilot
- Prayog PM bundle @ **v0.5.0-rc.2**: `/validate-requirements`, `/review-findings`, `/update-documents`, `/prd-impact-map`, `/commit-workspace`, `/open-draft-pr`, `/create-board-tickets`, `/prd`

Pin record: [`.harness-pin.yaml`](.harness-pin.yaml) (`profile: meta-pm`).

Re-sync after clone: `launchpad apply-harness --meta --apply`

### Delivery bootstrap

- Contract: **sdd-delivery/v2**
- Workflow: `prayog-skills/workflow.yaml`
- Pin record: `.harness-pin.yaml`
- Skill hub: `.harness/skills/`

When asked “what next?”, read the latest persistent handoff and the pinned
workflow, then explain the current stage, blockers, and next candidate. Do not
perform file or GitHub mutations unless the user explicitly authorizes them.

Development content skills only change the local workspace and record Forge
readiness. Branch/commit/push/PR/issue/label/merge happen only via forge skills.
<!-- launchpad:harness-end -->

PM workspace for **drivestream-lab** (`prayog-meta`).

## Repository truth

- Programme vision and planning: `planning/`
- PRDs: `prd/`
- Reports and impact maps: `prd/reports/`
- Service ownership: `config/service-catalog*.yaml`
- Programme/harness configuration: `config/`

Product decisions must be committed into PRD artifacts. Engineering decisions
are routed to engineering. Check existing branches and PRs before proposing a
new initiative PR.
