# AGENTS.md

Guide for working on ConfigMap components generated from
**vercel-configmap-template** — per-environment variable repos that deploy
as env vars onto a Vercel project. No application code.

## Overview

A ConfigMap component is the deployable projection of an app's *non-secret*
runtime configuration: `config/<env>/application-<env>.yaml` holds flat
`KEY: value` pairs that `cm-deploy` upserts into the target Vercel project
(via `POST /v10/projects/{project}/env?upsert=true`). The binding to the app
is declared in the app-model OAM (`needs:` on the app, so config lands
before it deploys) plus this repo's `.deploy/cd/{env}/cd.yml` — the ci_ids
matching the app it configures.

## Structure

- `config/<env>/application-<env>.yaml` — the only content that ships.
  Strictly flat scalars: `KEY: value` (uppercase env-var keys; `#` comments
  and quoted values OK).
- `version.txt` — the release version (semver). `release.yml` validates it,
  creates the tag + GitHub Release; manifest `tag:` must match.
- `.github/workflows/` — thin callers to `progmise/reusable-workflows@v1`
  (`cm-ci`, `cm-release`, `cm-deploy`). Keep them thin — never inline logic.

## Rules

- **Never commit secret values** — keys ending `_PASSWORD`, `_PASSWD`,
  `_SECRET`, `_TOKEN` or `PRIVATE_KEY` fail CI; secrets live in the platform
  credential store (Supabase `platform_secrets`/`platform_vars`).
- Keep keys in sync across every env file — `cm-config.py validate` enforces
  parity (missing/extra keys fail the PR).
- On any change to this repo, cut a release (`release.yml`) and bump the
  component's `version` in the app-model OAM — its PR validates the
  tag exists.
- Upsert semantics only: removing a key from the file does **not** delete
  the var from Vercel — clean stale vars in the Vercel dashboard by hand.

## Git workflow

Single `main` (protected). Work lands on `<type>/<snake_description>` + PR.

## Verify before done

```bash
python3 cm-config.py validate config   # script from reusable-workflows@v1
```
