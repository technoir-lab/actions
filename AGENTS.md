# Repository Guidelines

## Project Structure & Module Organization

This repository publishes two composite GitHub Actions: `setup-kotlin-multiplatform/` caches Kotlin/Native toolchains, and `code-review/` runs maintainer-requested Claude reviews. Each directory contains `action.yml` and a usage `README.md`. CI, release, and review workflows live in `.github/workflows/`; lint configuration is `.github/actionlint.yaml`.

## Build, Test, and Development Commands

No build or local application runtime is required. Run checks from the repository root:

- `actionlint`: lint GitHub workflow files with the repository configuration.
- `git diff --check`: check tracked changes for whitespace errors.

Update the relevant action README when changing inputs, outputs, defaults, or examples.

## Coding Style & Naming Conventions

Use two-space YAML indentation, `action.yml` for action metadata, and `.yaml` for workflows. Name action directories, input keys, and step IDs in lowercase kebab-case, such as `cache-read-only`. Declare `shell: bash` for composite `run` steps and quote shell variables. Pin external `uses:` dependencies to exact version tags, following existing files.

## Testing Guidelines

CI tests Kotlin/Native cache behavior on `ubuntu-26.04`, `macos-26`, and `windows-2025` by restoring and checking a marker in `~/.konan`. Extend descriptively named CI steps when changing cache behavior or outputs. Preserve existing CI job names: the `main` ruleset requires them as status checks, and each `Checks` job name includes its runner label, so renaming a job or changing a runner needs a matching update to the ruleset's required checks. There is no standalone test framework or coverage threshold. `actionlint` checks workflows; verify composite-action behavior separately in GitHub Actions.

## Commit & Pull Request Guidelines

Use concise imperative commit subjects, as in `Add code-review action`. Use descriptive branches without AI harness prefixes such as `codex/`. Never add `Co-Authored-By` trailers.

PR descriptions should explain the behavior change, link relevant issues, and state the checks performed in the sections that describe the change rather than in a section of their own. Assign review and audit findings stable, unique IDs such as `I1`, `I2`, and `I3`.

## Security & Releases

Keep OAuth tokens in GitHub Actions secrets. Pass event data through `env` before using it in shell scripts. Preserve the code-review action's permission checks and default-branch checkout; never execute pull request code with review credentials.

All actions share one release tag. Reference releases by action directory and exact tag, and keep README examples consistent with the intended release.
