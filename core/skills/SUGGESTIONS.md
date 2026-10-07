# Skills pipeline — suggestions tracker

Original feedback captured below. Status updated after skill/doc changes.

## Completed in skills/docs

- [x] **Ask where to store feature requests** — `gh-issue-creator`, `rh-qs-discovery` (contrib vs [rfe-creator](https://github.com/opendatahub-io/rfe-creator) vs [strat-creator](https://github.com/opendatahub-io/strat-creator))
- [x] **Verify step for quickstarts** — new `rh-qs-verify-deploy`; pipeline doc updated; docs run after verification
- [x] **Agent permissions / guardrails** — `rh-qs-secure` + `references/agent-permissions.md` (Helm/Makefile only, no kubectl)
- [x] **Documentation after verification** — `rh-qs-document` trigger updated; `NEW_QUICKSTART_SKILLS.md` flow reordered
- [x] **Security split** — `rh-qs-secure`: cluster access vs application security (`application-security.md`)
- [x] **Deployment sub-agent** — documented in `rh-qs-deploy` and `agent-permissions.md`
- [x] **Helm-only cluster ops (no oc in agent path)** — deploy, document, scaffold, helm reference guides
- [x] **Version bump skill** — `rh-qs-bump-versions`
- [x] **MinIO conversion → `rh-qs-debug-and-deploy`** — `rh-qs-minio-conversion` step 7 recommends debug-and-deploy (deploy + test + debug/fix for conversion gaps, e.g. Loki bundled MinIO) instead of verify-deploy alone

## Not completed (needs more time / separate work)

- [ ] **Integrate rfe-creator / strat-creator scripts** — documented routing only; no automation wired to those repos (partners may lack access)
- [ ] **Custom MCP server for OpenShift** — documented as future pattern with tool allowlist; no MCP implementation in this repo
- [ ] **Implement `make verify-deploy` in each quickstart template** — skill defines contract; template repo / ai-quickstart-template still needs the target added
- [ ] **Dedicated OpenShift user/SA provisioning** — guidance in `rh-qs-secure`; no IaC or bootstrap script
- [ ] **Prompt guard skill as standalone** — covered under application security + Llama Stack notes; not a separate skill file
- [ ] **Audit existing OpenShift skills in the wild** — not inventoried; link when a canonical list exists

## Original notes (archived)

<details>
<summary>Raw suggestion list</summary>

- Ask the user where they want to put the feature requests and store issues.
  - Use https://github.com/opendatahub-io/rfe-creator
  - Partners may not have this but lets take a look
- https://github.com/opendatahub-io/strat-creator
- Biggest Gap: Validation of the actual quickstart. Maybe a verify step?
- What permissions we want to give to the Agent? Guard rails with minimal access.
- Move document back and have verification step, then double check documentation.
- MCP Server to bridge Claude to OpenShift — tool allowlist; no kubectl workaround.
- Security split: basic cluster vs application secure; prompt guard; user context.
- Deployment: sub-agent; docs after successful deployment.
- No OC Command executed. Only via helm chart.
- Skill for updating versions / any package.

</details>
