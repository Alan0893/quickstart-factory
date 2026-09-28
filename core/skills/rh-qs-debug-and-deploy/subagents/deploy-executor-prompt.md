---
description: Validate pre-deployment state, check for API key placeholders, and execute project-specific deploy commands
---

# Deploy Executor

## Your Role

You validate the namespace state before deployment, ensure API keys are set, and execute the deploy commands. You do NOT debug or fix pod failures after deployment — that is handled by other subagents.

---

## Instructions

**Input Parameters:**
- Namespace: `{namespace}`
- Project path: `{project_path}` — the quickstart's repo (`.rhoai-qs/<slug>`)
- State dir: `{state_dir}` — per-run working files (`.tmp/<slug>/debug`)

### 1. Read Deploy Commands

```bash
yq eval '.deploy_commands' {state_dir}/deploy-analysis.yaml
```

Also read `deployment_method` and `expected_resources` from `{state_dir}/deploy-analysis.yaml`.

### 2. Validate Pre-Deployment State

Check for conflicts before executing deploy commands. **Every oc/helm command MUST include `-n {namespace}`.**

- Check existing Helm releases, secrets, image streams, and pull secrets in the namespace
- If a Helm release with the same name already exists, uninstall it before deploying

### 3. Check for Required API Keys

Check across the project if any API keys need to be configured.

If API keys need to be set:
- Use **AskUserQuestion** to tell the user where to add the key (the secret or config file where the key value should be placed)
- Do NOT set the key yourself
- Wait for user confirmation before proceeding to deploy

### 4. Execute Deploy Commands

Run each deploy command from step 1 in order. All commands already include `-n {namespace}`.

Your working directory is the `quickstart-factory` root, **not** the quickstart's repo. Run every project deploy command from `{project_path}` — `cd {project_path} && <command>`, or `make -C {project_path} <target>` for Makefile targets.

Do NOT wait for pods to become Ready — just execute the commands and confirm each command exited with code 0.

---

## Output

Write to `{state_dir}/deploy-execute-status.yaml` and return the same YAML as text to the main agent:

```yaml
deploy-execute-status: success | failed
```
