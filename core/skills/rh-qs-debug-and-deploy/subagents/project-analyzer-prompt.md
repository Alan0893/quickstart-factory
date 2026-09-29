---
description: Analyze project structure before deployment — map dependencies, expected resources, deploy method, and exact deploy commands
---

# Project Pre-Deployment Analysis

## Your Role

You are investigating an AI Quickstart that already has OpenShift/RHOAI deploy configuration in place. Your job is to understand its structure, map what will be deployed, discover the exact deployment commands, and build a dependency graph — all BEFORE any deployment happens.

Your output will be used to:
1. **Deploy correctly** — main agent runs the deploy commands you discover
2. **Know what to expect** — health scanner compares actual vs expected resources
3. **Debug in order** — dependency graph determines which resources to fix first

This is a **pure analysis task** — you are NOT deploying anything.

---

## Instructions

**Input Parameters:**
- Project directory: `{project_path}` — the quickstart's repo (`.rhoai-qs/<slug>`); all discovery below happens inside it
- State dir: `{state_dir}` — per-run working files (`.tmp/<slug>/debug`); your output goes here
- Target namespace: `{namespace}`

**General guideline:** The shell commands shown in the steps below (grep, find, etc.) are examples for your analysis work. If a command doesn't work or doesn't produce the expected output, use alternative approaches to get the information you need.

### 1. Find Deployment Instructions

Search the project for deployment documentation — READMEs, deployment guides, Makefiles, deploy scripts, or any other files that describe how to deploy.

Read each relevant file found and extract:
- Exact deployment commands (helm install, oc apply, or custom scripts)
- Deployment order if specified
- Required prerequisites (secrets, configmaps to create first)
- Any values files or parameters needed

### 2. Identify Deployment Method

Based on the deployment documentation and project structure you discovered in step 1, identify the deployment method (Helm, Kustomize, raw manifests, custom scripts, etc.). Read the relevant chart, manifest, or configuration files to understand the deployment setup.

### 3. Map Expected Resources

For Helm charts — inspect templates to list all Kubernetes resources:
```bash
# List all template files and extract resource kinds
grep -r "kind:" {chart_path}/templates/ | sort -u
```

For raw manifests — read each YAML file:
```bash
# Extract resource names and kinds
grep -rE "^kind:|^  name:" {manifests_path}/ | head -40
```

Build a complete list of expected resources: Deployments, StatefulSets, Services, PVCs, Routes, ConfigMaps, Secrets, Jobs.

### 4. Map Dependencies

Identify inter-service dependencies by analyzing:
- Environment variables referencing other services (e.g., `POSTGRES_HOST`, `REDIS_URL`)
- Init containers waiting for dependencies
- Service references in configuration
- Explicit dependency documentation in project docs

```bash
# Find env vars referencing services
grep -rE "value:.*(-svc|_HOST|_URL|_ADDR|_PORT)" {chart_path}/templates/ 2>/dev/null || true
grep -rE "value:.*(-svc|_HOST|_URL|_ADDR|_PORT)" {manifests_path}/ 2>/dev/null || true

# Find init containers
grep -A5 "initContainers:" {chart_path}/templates/*.yaml 2>/dev/null || true
```

### 5. Identify Resource Requirements

Check for GPU, storage, and other special requirements:
```bash
# GPU requirements
grep -rE "nvidia.com/gpu|gpu:" {chart_path}/templates/ 2>/dev/null || true

# Storage requirements
grep -rE "storage:|storageClassName:|accessModes:" {chart_path}/templates/ 2>/dev/null || true

# Security contexts
grep -rE "securityContext:|runAsNonRoot:|fsGroup:" {chart_path}/templates/ 2>/dev/null || true
```

### 6. Build Dependency Order

Create a dependency order with leaves first (resources that have no dependencies):
- Level 0: Resources with no dependencies (databases, caches)
- Level 1: Resources depending only on Level 0
- Level 2: Resources depending on Level 0 + Level 1
- etc.

### 7. Detect Mutually-Exclusive Deployment Options

A project may document more than one way to deploy to OpenShift — in-cluster model serving vs. an external inference endpoint, GPU vs. CPU-only, a full stack vs. a minimal profile, etc. These usually show up as alternative values files, `--set` flags, Makefile targets, or "Option A / Option B" sections in the deployment docs.

Treat something as a deployment option only when **all** of these hold:

- The paths are **mutually exclusive** — running both is wrong or meaningless.
- Each path is a **complete, deployable** OpenShift path on its own.
- Choosing between them changes **what runs in the cluster**, not just where or under what name.

These are **not** deployment options — do not record them:

- Namespace or release-name differences
- Command ordering, or optional add-ons that can coexist with the base install
- Local-development paths. This skill deploys to OpenShift only, so if `{project_path}/pipeline/architecture-spec.yaml` has `deployment_mode: both`, ignore the `compose` path entirely and consider only the Helm/OpenShift paths.

If you find two or more qualifying paths, record each one under `deployment_options` with its own `deploy_commands`, and set `recommended: true` on the one the project documents as primary. Also seed the top-level `deploy_commands` with the recommended option's commands, so the file is valid even before a selection is made — the main agent overwrites it once the user chooses.

If you find exactly one path, omit `deployment_options`, set `selected_option_id: null`, and fill `deploy_commands` as usual.

---

## Output

Read the output schema from:
```
./output-templates/deploy-analysis-template.md
```

Write the analysis to `{state_dir}/deploy-analysis.yaml` following that schema.

**Critical fields:**
- `deploy_commands` — the EXACT commands for this project (not generic templates). Replace `<namespace>` placeholder with actual namespace.
- `deployment_options` — only when two or more mutually-exclusive OpenShift paths exist (see step 7). Omit the field otherwise; do not emit a single-entry list.
- `expected_resources` — complete list of all K8s resources that should exist after deployment
- `dependency_order` — levels from leaves to root
- `deploy_instructions_source` — where you found the deploy commands

**Important:**
- All oc/helm commands you include must use `-n {namespace}` explicitly.
- Prefer `oc` commands over `kubectl` in all deploy commands.
