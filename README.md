## Understanding dbt-package-testing

This repo provides reusable GitHub Actions workflows for testing dbt packages against supported adapters:

- **`run_tox.yml`** — runs integration tests using dbt Core
- **`run_tox_fusion.yml`** — runs integration tests using the dbt Fusion engine

For more info on how to test packages, refer to:
- [Integration tests](https://docs.getdbt.com/guides/building-packages?step=4) for new package maintainers.
- [Test package with Fusion](https://docs.getdbt.com/guides/fusion-package-compat?step=5#test-package-with-fusion) to test your package with the dbt Fusion engine to ensure compatibility.

## Setup

To use these workflows, your package repo needs a few things: a dbt project for integration tests, a `profiles.yml` wired to environment variables, a `tox.ini` to run the tests, and a caller workflow.

### 1. Integration test project

Your repo needs an `integration_tests/` directory containing a dbt project with:

- `dbt_project.yml`
- `profiles.yml` — defines a target for each adapter, using `env_var()` to read credentials from the environment. You can copy [`integration_tests/profiles.yml`](integration_tests/profiles.yml) from this repo as a starting point — it includes targets for all supported adapters.

For example, a Snowflake target in `profiles.yml`:

```yaml
integration_tests:
  target: snowflake
  outputs:
    snowflake:
      type: "snowflake"
      account: "{{ env_var('SNOWFLAKE_ACCOUNT') }}"
      user: "{{ env_var('SNOWFLAKE_USER') }}"
      password: "{{ env_var('DBT_ENV_SECRET_SNOWFLAKE_PASS') }}"
      role: "{{ env_var('SNOWFLAKE_ROLE') }}"
      database: "{{ env_var('SNOWFLAKE_DATABASE') }}"
      warehouse: "{{ env_var('SNOWFLAKE_WAREHOUSE') }}"
      schema: "{{ env_var('SNOWFLAKE_SCHEMA') }}"
      threads: 10
```

### 2. `tox.ini`

Your package needs a `tox.ini` at the repo root. The workflows run `tox -e dbt_integration_<adapter>` (Core) or `tox -e dbt_integration_fusion_<adapter>` (Fusion engine), so the environment names must follow this convention.

The `[testenv]` section must include a `passenv` list with all the environment variables your adapters need. Tox isolates the environment by default, so without `passenv` the credentials set by the workflow won't reach dbt. See the [`tox.ini`](tox.ini) in this repo for a complete example.

**For dbt Core**, create `[testenv:dbt_integration_<adapter>]` sections:

```ini
[testenv]
passenv =
    SNOWFLAKE_ACCOUNT
    SNOWFLAKE_USER
    DBT_ENV_SECRET_SNOWFLAKE_PASS
    SNOWFLAKE_ROLE
    SNOWFLAKE_DATABASE
    SNOWFLAKE_WAREHOUSE
    SNOWFLAKE_SCHEMA
    # ... add all env vars for your adapters

[testenv:dbt_integration_snowflake]
changedir = integration_tests
allowlist_externals =
    dbt
skip_install = true
commands =
    dbt deps --target snowflake
    dbt build -x --target snowflake --full-refresh
```

**For the dbt Fusion engine**, create `[testenv:dbt_integration_fusion_<adapter>]` sections:

```ini
[testenv:dbt_integration_fusion_snowflake]
changedir = integration_tests
allowlist_externals =
    dbt
skip_install = true
commands =
    dbt deps --target snowflake
    dbt build -x --target snowflake --full-refresh
```

**About `--static-analysis=off`:** By default, you should _not_ set this flag — the dbt Fusion engine's static analysis provides valuable validation. However, you may need to add `--static-analysis=off` to your commands if:

- Your integration tests don't have all sources available at parse time
- Your package uses SQL functions or patterns not yet supported by the dbt Fusion engine's static analyzer

If you do need it, add it to the dbt commands in your tox environment:

```ini
commands =
    dbt deps --target snowflake
    dbt build -x --target snowflake --full-refresh --static-analysis=off
```

### 3. Adapter list

The workflows need to know which adapters to test. You can either:

- **Pass the `adapters` input** explicitly in your workflow call (e.g. `adapters: "snowflake,bigquery"`) — recommended
- **Create a `supported_adapters.env` file** at the repo root as a fallback:
  ```
  SUPPORTED_ADAPTERS=snowflake,bigquery,redshift,databricks
  ```

### 4. GitHub secrets and variables

The workflows pass credentials to dbt via environment variables. You need to configure these in your GitHub repo settings under **Settings > Secrets and variables > Actions**:

- **Secrets** — for sensitive values like passwords and tokens (e.g. `SNOWFLAKE_ACCOUNT`, `DBT_ENV_SECRET_SNOWFLAKE_PASS`, `BIGQUERY_KEYFILE_JSON`)
- **Variables** — for non-sensitive values like hostnames and roles (e.g. `SNOWFLAKE_USER`, `SNOWFLAKE_ROLE`, `SNOWFLAKE_DATABASE`)

The names must match what your `profiles.yml` expects via `env_var()` and what you pass as inputs/secrets in the caller workflow. See the [`ci.yml`](.github/workflows/ci.yml) example for the full list of inputs and secrets each workflow accepts.

### 5. Caller workflow

Create a `.github/workflows/ci.yml` in your package repo that calls the reusable workflows. See [`ci.yml`](.github/workflows/ci.yml) in this repo for a complete example.

**Key inputs:**

| Input | Description |
|-------|-------------|
| `adapters` | Comma-separated adapter list (falls back to `supported_adapters.env`) |
| `environment` | GitHub Environment name for fork PR protection (see below) |
| `ref` | Git ref to checkout — required for `pull_request_target` callers |

All adapter-specific inputs (e.g. `SNOWFLAKE_USER`, `BIGQUERY_PROJECT`) and secrets (e.g. `SNOWFLAKE_ACCOUNT`, `DBT_ENV_SECRET_SNOWFLAKE_PASS`) are optional — only include the ones for adapters you're testing.

### 6. Fork PR protection (optional)

If your workflow uses `pull_request_target` to run CI on fork PRs (which need access to secrets), you can gate execution behind a GitHub Environment:

1. Create a GitHub Environment in your repo settings (e.g. `cloud-tests`) with required reviewers.
2. Pass the environment name conditionally in your workflow call:

```yaml
environment: >-
  ${{ github.event_name == 'pull_request_target'
      && github.event.pull_request.head.repo.full_name != github.repository
      && 'cloud-tests' || '' }}
ref: ${{ github.event.pull_request.head.sha || '' }}
```

This requires a maintainer to approve the workflow run before fork PRs get access to secrets.

## Reporting bugs and contributing code

- Want to report a bug or request a feature? Let us know by opening [an issue](https://github.com/dbt-labs/dbt-package-testing/issues/new)

## Code of Conduct

Everyone interacting in the dbt project's codebases, issue trackers, chat rooms, and mailing lists is expected to follow the [dbt Code of Conduct](https://community.getdbt.com/code-of-conduct).
