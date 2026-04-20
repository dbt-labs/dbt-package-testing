# Design: Core + Fusion Reusable Workflows

## Goal

Update `dbt-package-testing` to provide two reusable GitHub Actions workflows:

- `run_tox.yml` — runs dbt Core integration tests (existing, updated)
- `run_tox_fusion.yml` — runs dbt Fusion integration tests (new)

Both support optional `pull_request_target` usage with GitHub Environment gating for fork PRs.

## Shared Interface

Both workflows accept these new inputs:

| Input | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `adapters` | string | no | `""` | Comma-separated adapter list. Falls back to reading `supported_adapters.env` from the calling repo. |
| `environment` | string | no | `""` | GitHub Environment name for job protection (e.g. `cloud-tests`). |
| `ref` | string | no | `""` | Git ref to checkout. Required for `pull_request_target` callers — pass `github.event.pull_request.head.sha`. |

All existing adapter credential inputs and secrets remain unchanged for backwards compatibility.

## Adapter Resolution

Both workflows use the same logic to determine which adapters to test:

```yaml
- name: "Get list of supported adapters"
  run: |
    if [ -n "$INPUT_ADAPTERS" ]; then
      echo "test_adapters=$INPUT_ADAPTERS" >> $GITHUB_OUTPUT
    else
      source supported_adapters.env
      echo "test_adapters=$SUPPORTED_ADAPTERS" >> $GITHUB_OUTPUT
    fi
  env:
    INPUT_ADAPTERS: ${{ inputs.adapters }}
```

## Changes to `run_tox.yml` (Core)

1. Add `adapters`, `environment`, and `ref` inputs.
2. Update `determine-supported-adapters` to prefer `adapters` input over `supported_adapters.env`.
3. Add `environment: ${{ inputs.environment }}` to the `run-tests` job.
4. Update checkout step: `ref: ${{ inputs.ref || github.ref }}`.

## New `run_tox_fusion.yml`

Same structure as `run_tox.yml` with these differences:

- **Install step**: installs Fusion via `curl -fsSL https://public.cdn.getdbt.com/fs/install/install.sh | sh` instead of `pip install dbt-<adapter>`.
- **Tox env**: runs `tox -e dbt_integration_fusion_${{ matrix.adapter }}` instead of `tox -e dbt_integration_${{ matrix.adapter }}`.
- **No postgres service container**: Fusion does not support postgres.
- **No adapter pip install**: Fusion bundles adapter support.

## Caller Example

A calling repo (e.g. `dbt-project-evaluator`) would have a single CI workflow:

```yaml
name: Package Integration Tests

on:
  push:
    branches: [main]
  pull_request_target:
  workflow_dispatch:

jobs:
  core-tests:
    uses: dbt-labs/dbt-package-testing/.github/workflows/run_tox.yml@main
    with:
      adapters: "snowflake,bigquery,redshift,databricks"
      environment: >-
        ${{ github.event_name == 'pull_request_target'
            && github.event.pull_request.head.repo.full_name != github.repository
            && 'cloud-tests' || '' }}
      ref: ${{ github.event.pull_request.head.sha || '' }}
      SNOWFLAKE_USER: ${{ vars.SNOWFLAKE_USER }}
      # ... other adapter vars ...
    secrets:
      SNOWFLAKE_ACCOUNT: ${{ secrets.SNOWFLAKE_ACCOUNT }}
      # ... other secrets ...

  fusion-tests:
    uses: dbt-labs/dbt-package-testing/.github/workflows/run_tox_fusion.yml@main
    with:
      adapters: "snowflake"
      environment: >-
        ${{ github.event_name == 'pull_request_target'
            && github.event.pull_request.head.repo.full_name != github.repository
            && 'cloud-tests' || '' }}
      ref: ${{ github.event.pull_request.head.sha || '' }}
      SNOWFLAKE_USER: ${{ vars.SNOWFLAKE_USER }}
      # ... other adapter vars ...
    secrets:
      SNOWFLAKE_ACCOUNT: ${{ secrets.SNOWFLAKE_ACCOUNT }}
      # ... other secrets ...
```

## Caller Requirements

- `tox.ini` with `[testenv:dbt_integration_<adapter>]` sections for Core
- `tox.ini` with `[testenv:dbt_integration_fusion_<adapter>]` sections for Fusion
- Either `supported_adapters.env` or pass `adapters` input explicitly
- A GitHub Environment (e.g. `cloud-tests`) configured with required reviewers if using `pull_request_target` with fork protection
