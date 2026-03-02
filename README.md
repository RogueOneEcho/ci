# CI

Shared GitHub Actions workflows for [RogueOneEcho](https://github.com/RogueOneEcho) projects.

## Calling Convention

Each consuming repo has four workflow files. The three trigger files call the local `ci.yml`, which in turn calls the shared workflow. This keeps repo-specific inputs (e.g. `config-path`) in one place.

- [`ci.yml`](examples/ci/ci.yml) — Local wrapper that calls the shared workflow with repo-specific inputs
- [`ci-on-push.yml`](examples/ci/ci-on-push.yml) — Runs CI on every push. Keeps the actions cache warm on `main`
- [`ci-on-pr-approved.yml`](examples/ci/ci-on-pr-approved.yml) — Runs CI when a PR is approved. Gates auto-merge via branch protection
- [`ci-on-pr-labeled.yml`](examples/ci/ci-on-pr-labeled.yml) — Runs CI on demand without approving. Removes the label after

```
ci-on-push.yml        ──┐
ci-on-pr-approved.yml ──┼──▶ ci.yml ──▶ RogueOneEcho/ci/rust-lib.yml@v1
ci-on-pr-labeled.yml  ──┘
```

## Workflows

### [`rust-lib.yml`](.github/workflows/rust-lib.yml)

CI for Rust library crates

Used by:

- [flat_db](https://github.com/RogueOneEcho/flat_db)
- [gazelle_api](https://github.com/RogueOneEcho/gazelle_api)
- [logging](https://github.com/RogueOneEcho/logging)

Examples:

- [`minimal.yml`](examples/rust-lib/minimal.yml) — Default usage
- [`with-config.yml`](examples/rust-lib/with-config.yml) — Write a config file before tests
- [`no-publish.yml`](examples/rust-lib/no-publish.yml) — Skip crates.io publish
- [`workspace.yml`](examples/rust-lib/workspace.yml) — Workspace with multiple manifests

### [`docker.yml`](.github/workflows/docker.yml)

CI for Docker-only projects. Multi-arch builds (amd64/arm64), SBOM scanning, cosign attestation, and GHCR publishing.

Used by:

- [caddy-cloudflare](https://github.com/RogueOneEcho/caddy-cloudflare)
- [backup](https://github.com/RogueOneEcho/backup)
- [preflight](https://github.com/RogueOneEcho/preflight)

Examples:

- [`minimal.yml`](examples/docker/minimal.yml) — Default build context
- [`with-build-context.yml`](examples/docker/with-build-context.yml) — Custom build context subdirectory

### [`rust-bin.yml`](.github/workflows/rust-bin.yml)

CI for Rust binary crates with optional Docker builds.

Used by:

- [caesura](https://github.com/RogueOneEcho/caesura)

Examples:

- [`minimal.yml`](examples/rust-bin/minimal.yml) — Single-target binary with Docker
- [`cross-platform.yml`](examples/rust-bin/cross-platform.yml) — Multi-target matrix with Docker
- [`with-extra-jobs.yml`](examples/rust-bin/with-extra-jobs.yml) — Using version output for post-CI jobs

### [`ff-release.yml`](.github/workflows/ff-release.yml)

Fast-forward the `release` branch to a tagged commit on `main`. Must be dispatched on the `release` branch so that chained CI runs against `release`. Validates the commit is on `main`, has a version tag, CI has passed, and fast-forward is possible.

Because `github.token` pushes don't trigger workflows, the caller must chain CI as a dependent job (see example).

Examples:

- [`release.yml`](examples/ff-release/release.yml) — Dispatch workflow that fast-forwards then runs CI

## Rulesets

### [`main.json`](rulesets/main.json)

Branch protection ruleset for `main`. Apply to a repo with:

```sh
gh api repos/RogueOneEcho/{repo}/rulesets -X POST --input rulesets/main.json
```

### [`release.json`](rulesets/release.json)

Branch protection ruleset for `release`. No bypass — requires CI and git-tag to pass (from `main`) before pushing. Apply to a repo with:

```sh
gh api repos/RogueOneEcho/{repo}/rulesets -X POST --input rulesets/release.json
```
