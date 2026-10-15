# vercel-configmap-template

Template repo for **ConfigMap components** on the progmise platform — the
Vercel equivalent of Gluon's `*-cm` repos. Holds per-environment variable
files; deploying a version upserts each key as an environment variable on the
target Vercel project. No application code.

## Layout

```
config/
  pre/application-pre.yaml      # flat KEY: value pairs (env-var mode)
  pro/application-pro.yaml
version.txt                     # release version (semver, bumped per release)
.github/workflows/              # thin callers → reusable-workflows cm-* @v1
```

One folder per manifest environment (`config/<env>/application-<env>.yaml`);
add a new env by creating `config/<env>/application-<env>.yaml` and an
environment with that name in `deploy-manifest`.

## How it deploys

`deploy.yml` (manual dispatch or via `deploy-manifest`'s orchestrator) takes
`version`, `environment`, `ci_id` and `project`. The orchestrator derives
`project` from the manifest's `infrastructures[].properties.project` — the
Vercel project this ConfigMap feeds. `cm-deploy` then upserts every key:

- `pro` → Vercel target `production`; any other env → `preview`.
- Keys removed from the file are **not** deleted from the project (upsert
  only — manually managed vars are preserved).

## Rules

- **Never put secrets here** — keys ending `_PASSWORD`, `_SECRET`, `_TOKEN`
  or `PRIVATE_KEY` fail CI. Secrets belong in the platform credential store.
- Keep the key set identical across all env files (CI enforces parity).
- Flat `KEY: value` scalars only — no nested YAML or lists.

## Flow

PR → `ci.yml` (config validation) → merge → `release.yml` (tag + GitHub
Release — version lives in `version.txt`) → `deploy.yml` / orchestrator
dispatches per `ci_id` → env vars upserted into the target Vercel project.

Registering a CM in `deploy-manifest` looks like:

```yaml
  - name: loans-api-cm
    repo: progmise/loans-api-cm
    tag: "0.1.0"
    type: configmap              # no docker image — the deployable unit is config/
    needs: []
    infra: [loans-api-pre, loans-api-pro]   # the app's infra ids
  - name: loans-api
    ...
    needs: [loans-api-cm]        # config lands before the app deploys
```

Required repo secrets/vars: `VERCEL_TOKEN` (secret), `VERCEL_ORG_ID` (var).
