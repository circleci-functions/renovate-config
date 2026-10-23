# renovate-config

Shared [Renovate config presets](https://docs.renovatebot.com/config-presets/) for the
`circleci-functions` org.

`default.json` is the org-level preset. Every repo in the org should extend it.

## Usage

Add a `renovate.json` to the root of your repo:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "local>circleci-functions/renovate-config"
  ]
}
```

## What the preset does

- Runs weekly, on Sundays (UTC).
- Waits 7 days after a release before proposing it, and skips releases whose CI is still
  pending. Security updates are exempt and are labelled `security`.
- Batches minor and patch updates into a single **all non-major dependencies** PR.
- Puts `go` (the `go.mod` directive) and `cimg/go` in one **Go** PR, so the declared
  toolchain and the CI image never drift apart.
- Gives `golangci-lint` its own PR, since lint upgrades tend to introduce new failures.
- Proposes `github.com/circleci-functions/*` updates immediately, in their own PR, so
  `toolkit` releases reach the `setup-*` repos without waiting for the weekly batch.
- Runs `go mod tidy` after updating.
- Ignores `vendor/`, `node_modules/`, `testdata/` and `internal/testing/` — test fixtures
  are pinned on purpose.

## Renovate annotation comments

The preset can update versions in files Renovate doesn't otherwise understand, such as
pipeline parameters in `.circleci/config.yml`, via a trailing comment:

```yaml
parameters:
  node-version:
    type: string
    default: "26.4.0" # renovate: datasource=node-version depName=node
```

The version must be quoted, and both `#` and `//` comments work. See the
[datasource list](https://docs.renovatebot.com/modules/datasource/) for available values.

These optional fields may follow `depName`, in this order:

- `packageName` — defaults to `depName`. See
  [depName vs packageName](https://docs.renovatebot.com/faq/#whats-the-difference-between-depname-and-packagename).
- `versioning` — a custom [versioning](https://docs.renovatebot.com/modules/versioning/) module.
- `extractVersion` — when the version in your code doesn't match the datasource, e.g.
  `^v(?<version>\d+\.\d+)`.
- `registryUrl` — for datasources that need an explicit registry.

To track a digest alongside the version, use `renovate-digest:` and a `version@digest` value.

## Making changes

`.circleci/config.yml` validates `default.json` with `renovate-config-validator --strict`.
To check locally:

```sh
RENOVATE_GLOBAL_EXTENDS='["global:safeEnv"]' \
  npx --yes --package renovate renovate-config-validator --strict --no-global default.json
```

Changes here apply to every repo in the org on the next Renovate run.
