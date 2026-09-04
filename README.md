# Hi there 👋, we are kitconcept, a Plone Agency from Bonn and Barcelona

We love Open Source and contribute to [Plone](https://plone.org) and other Open Source projects on a daily basis.

This repository holds the composite actions and reusable workflows shared by our
Plone monorepo projects. They assume a repository laid out with a `backend/` and a
`frontend/` folder, each exposing a `Makefile`.

Everything here is referenced at `@main`, so changes land on every consumer as soon
as they are merged.

## Deprecation

> [!WARNING]
> Everything in this repository except [`deploy.yml`](#deployyml) is **deprecated**.
> Use [`plone/meta`](https://github.com/plone/meta) instead, referenced at `@2.x`.
> The deprecated workflows and actions still run, but each one now emits a warning
> annotation and a note in the job summary. They will be removed.
>
> `deploy.yml` is **not** deprecated: `plone/meta` has no Docker Swarm deployment
> counterpart, so it remains the supported way to deploy.

### Migrating

| This repository | Replacement in `plone/meta@2.x` |
| --- | --- |
| `.github/actions/setup_backend` | `.github/actions/setup_backend_uv` |
| `.github/actions/setup_frontend` | `.github/actions/setup_frontend` |
| `.github/workflows/backend-lint.yml` | `.github/workflows/backend-lint.yml` |
| `.github/workflows/backend-test.yml` | `.github/workflows/backend-pytest.yml` |
| `.github/workflows/backend-coverage.yml` | `.github/workflows/backend-pytest-coverage.yml` |
| `.github/workflows/frontend-lint.yml` | `.github/workflows/frontend-code.yml` |
| `.github/workflows/frontend-test.yml` | `.github/workflows/frontend-unit.yml` |
| `.github/workflows/frontend-i18n.yml` | `.github/workflows/frontend-i18n.yml` |
| `.github/workflows/docs.yml` | `.github/workflows/docs-build.yml` |
| `.github/workflows/image-build.yml` | `.github/workflows/container-image-build-push.yml` |
| `.github/workflows/deploy.yml` | none, keep using this one |

The replacements are equivalent in purpose but not always drop-in:

- `plone/meta` defaults `working-directory` to `.` rather than `backend` or
  `frontend`, so a monorepo has to pass it explicitly.
- The backend test and coverage workflows here run `make test` and
  `make test-coverage`; the `plone/meta` ones invoke `pytest` directly.
- `container-image-build-push.yml` takes an additional required `push` input.
- `backend-lint.yml` and `docs-build.yml` in `plone/meta` accept extra inputs, such
  as pinned tool versions and optional Vale checks.


## Composite actions

### `setup_backend`

> [!WARNING]
> Deprecated. Use `plone/meta/.github/actions/setup_backend_uv@2.x` instead.

Installs `uv`, restores the `uv` cache and installs Plone plus the project package by
running `make install` in the working directory.

`kitconcept/meta/.github/actions/setup_backend@main`

#### Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `python-version` | yes | `3.14` | Python version passed to `uv` |
| `plone-version` | yes | `6.2.1` | Plone version, exported as `PLONE_VERSION` and used in the cache key |
| `working-directory` | no | `backend` | Directory the `make install` runs in |

#### Outputs

None.

#### Requirements

A `Makefile` in the working directory with an `install` target. It receives
`PYTHON_VERSION` and `PLONE_VERSION` in the environment.

### `setup_frontend`

> [!WARNING]
> Deprecated. Use `plone/meta/.github/actions/setup_frontend@2.x` instead.

Sets up Node.js, enables `corepack`, restores the `pnpm` store cache and installs the
project dependencies by running `make install` in the working directory.

`kitconcept/meta/.github/actions/setup_frontend@main`

#### Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `node-version` | yes | `24.x` | Node.js version |
| `plone-version` | yes | `6.2.1` | Declared but currently not used by any step |
| `working-directory` | no | `frontend` | Directory the `make install` runs in |

#### Outputs

None. The action exports `STORE_PATH` to the job environment, pointing at the `pnpm`
store directory.

#### Requirements

A `Makefile` in the working directory with an `install` target, and a
`packageManager` entry so `corepack` can provision `pnpm`.

## Reusable Workflows

| Workflow | Purpose | Status |
| --- | --- | --- |
| [`backend-lint.yml`](#backend-lintyml) | Lint and check metadata of the Python codebase | Deprecated |
| [`backend-test.yml`](#backend-testyml) | Run the backend test suite | Deprecated |
| [`backend-coverage.yml`](#backend-coverageyml) | Run the backend suite with coverage reporting | Deprecated |
| [`frontend-lint.yml`](#frontend-lintyml) | Lint the frontend codebase | Deprecated |
| [`frontend-test.yml`](#frontend-testyml) | Run the frontend test suite | Deprecated |
| [`frontend-i18n.yml`](#frontend-i18nyml) | Check that translations are up to date | Deprecated |
| [`docs.yml`](#docsyml) | Build the documentation and check for broken links | Deprecated |
| [`image-build.yml`](#image-buildyml) | Build and publish a container image | Deprecated |
| [`deploy.yml`](#deployyml) | Deploy a stack to a Docker Swarm cluster | Supported |

None of these workflows declare `outputs`.

### `backend-lint.yml`

> [!WARNING]
> Deprecated. Use `plone/meta/.github/workflows/backend-lint.yml@2.x` instead.

Runs `ruff format --diff`, `ruff check --diff`, `zpretty --check src`, `pyroma` and
`check-python-versions`, then writes a summary. Each check runs even if an earlier one
failed, so a single run reports every problem.

#### Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `python-version` | yes | — | Python version passed to `uv` |
| `plone-version` | yes | — | Declared but currently not used by any step |
| `working-directory` | no | `backend` | Directory the checks run in |

#### Example

```yaml
jobs:
  lint:
    uses: kitconcept/meta/.github/workflows/backend-lint.yml@main
    with:
      python-version: "3.14"
      plone-version: "6.2.1"
```

### `backend-test.yml`

> [!WARNING]
> Deprecated. Use `plone/meta/.github/workflows/backend-pytest.yml@2.x` instead.

Sets up the backend and runs `make test`.

#### Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `python-version` | yes | — | Python version |
| `plone-version` | yes | — | Plone version |
| `working-directory` | no | `backend` | Directory the tests run in |

#### Requirements

A `test` target in the backend `Makefile`.

### `backend-coverage.yml`

> [!WARNING]
> Deprecated. Use `plone/meta/.github/workflows/backend-pytest-coverage.yml@2.x` instead.

Sets up the backend, runs `make test-coverage` and appends a Markdown coverage report
to the job summary.

#### Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `python-version` | yes | — | Python version |
| `plone-version` | yes | — | Plone version |
| `working-directory` | no | `backend` | Directory the tests run in |

#### Requirements

A `test-coverage` target in the backend `Makefile`, and `coverage` available through
`uv run`.

### `frontend-lint.yml`

> [!WARNING]
> Deprecated. Use `plone/meta/.github/workflows/frontend-code.yml@2.x` instead.

Sets up the frontend and runs `make lint`.

#### Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `node-version` | yes | — | Node.js version |
| `working-directory` | no | `frontend` | Directory the lint runs in |

### `frontend-test.yml`

> [!WARNING]
> Deprecated. Use `plone/meta/.github/workflows/frontend-unit.yml@2.x` instead.

Sets up the frontend and runs `make test`.

#### Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `node-version` | yes | — | Node.js version |
| `working-directory` | no | `frontend` | Directory the tests run in |

### `frontend-i18n.yml`

> [!WARNING]
> Deprecated. Use `plone/meta/.github/workflows/frontend-i18n.yml@2.x` instead.

Sets up the frontend and runs `make ci-i18n`, which fails when the translation files
are out of sync with the source.

#### Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `node-version` | yes | — | Node.js version |
| `working-directory` | no | `frontend` | Directory the check runs in |

### `docs.yml`

> [!WARNING]
> Deprecated. Use `plone/meta/.github/workflows/docs-build.yml@2.x` instead.

Installs the documentation dependencies with `uv`, runs `make linkcheckbroken` and
then `make build`.

#### Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `python-version` | no | `3.14` | Python version passed to `uv` |
| `working-directory` | no | `docs` | Directory the documentation build runs in |

#### Requirements

A `Makefile` in the working directory with `install`, `linkcheckbroken` and `build`
targets.

### `image-build.yml`

> [!WARNING]
> Deprecated. Use `plone/meta/.github/workflows/container-image-build-push.yml@2.x` instead.

Builds a container image with Buildx and pushes it to a registry, using a registry
backed build cache. The image is tagged with the value of `base-tag` and with the
tags derived by `docker/metadata-action`. QEMU is set up before the build, so
`platforms` may name architectures other than the runner's.

#### Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `base-tag` | yes | — | Tag applied to the image, also used as the cache destination suffix |
| `working-directory` | yes | — | Build context |
| `image-name-prefix` | yes | — | First part of the image name |
| `image-name-suffix` | yes | — | Second part of the image name, joined with a dash |
| `image-cache-suffix` | no | `buildcache` | Prefix for the registry cache tag |
| `platforms` | no | `linux/amd64` | Target platforms for the build, comma separated, for example `linux/amd64,linux/arm64` |
| `dockerfile` | no | `Dockerfile` | Dockerfile path, relative to `working-directory` |
| `registry` | no | `ghcr.io` | Registry to log into |
| `build-args` | no | `""` | Build arguments forwarded to the build |
| `cache-key` | no | `${{ github.ref_name }}` | Suffix for the cache source tag |

#### Secrets

| Name | Required | Description |
| --- | --- | --- |
| `username` | yes | Registry user |
| `password` | yes | Registry password or token |

#### Example

```yaml
jobs:
  build:
    uses: kitconcept/meta/.github/workflows/image-build.yml@main
    with:
      base-tag: latest
      working-directory: backend
      image-name-prefix: ghcr.io/kitconcept/my-project
      image-name-suffix: backend
    secrets:
      username: ${{ github.actor }}
      password: ${{ secrets.GITHUB_TOKEN }}
```

### `deploy.yml`

Deploys a stack to a Docker Swarm cluster over SSH using
[`kitconcept/docker-stack-deploy`](https://github.com/kitconcept/docker-stack-deploy),
with a deploy timeout of 480 seconds. The job runs in the GitHub environment named by
the `environment` input, so it picks up that environment's protection rules and
secrets.

#### Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `registry` | yes | — | Registry holding the images |
| `username` | yes | — | Registry user |
| `tag` | yes | — | Image tag to deploy, passed to the stack as a parameter |
| `environment` | yes | — | GitHub environment the job runs in |
| `stack-name` | yes | — | Name of the Swarm stack |
| `stack-file` | yes | — | Path to the stack compose file |

#### Secrets

| Name | Required | Description |
| --- | --- | --- |
| `password` | yes | Registry password or token |
| `remote-host` | yes | Cluster host to deploy to |
| `remote-port` | yes | SSH port |
| `remote-user` | yes | SSH user |
| `remote-private-key` | yes | SSH private key |
| `env-file` | no | Contents of an environment file for the stack |

#### Example

```yaml
jobs:
  deploy:
    uses: kitconcept/meta/.github/workflows/deploy.yml@main
    with:
      registry: ghcr.io
      username: ${{ github.actor }}
      tag: ${{ github.ref_name }}
      environment: production
      stack-name: my-project
      stack-file: devops/stacks/production.yml
    secrets:
      password: ${{ secrets.GITHUB_TOKEN }}
      remote-host: ${{ secrets.DEPLOY_HOST }}
      remote-port: ${{ secrets.DEPLOY_PORT }}
      remote-user: ${{ secrets.DEPLOY_USER }}
      remote-private-key: ${{ secrets.DEPLOY_SSH_KEY }}
```
