# Starting a new Home Assistant integration repository

How to create the repository of a new Home Assistant custom integration from this template. For adopting the template in a repository that already exists, see "Adopting it in an existing repository" in the [README](../README.md).

## Prerequisites

- `git`, and a GitHub account that can create repositories.
- [uv](https://docs.astral.sh/uv/getting-started/installation/) and Copier 9.0.0 or newer (`uv tool install copier`). See "Installation" in the [README](../README.md).
- For the devcontainer: Docker and VS Code with the Dev Containers extension. The container forwards `GITHUB_PERSONAL_ACCESS_TOKEN` from your host to the GitHub MCP server (`remoteEnv` in `devcontainer.json`), so export it on the host if you use Claude Code in the container.

## 1. Create the repository

1. Create an empty repository on GitHub. Name it `ha-<domain>`, with underscores of the domain written as dashes: this is the default of `repo_name` in `copier.yml`.
2. Clone it and generate the project from the template, from the repository root. Always use the `gh:` address and not a local clone: the source is recorded in `.copier-answers.yml`, and `copier update` uses it later.

   ```bash
   git clone git@github.com:<owner>/ha-<domain>.git
   cd ha-<domain>
   copier copy gh:bastgau/ha-dev-environment .
   ```

3. Answer the questions (see `copier.yml`):

   | Variable                                                             | What to give                                                                                                  |
   | -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
   | `integration_domain`                                                 | Name of the `custom_components/<domain>` folder. Lowercase letters, digits and underscores, starting with a letter |
   | `integration_name`                                                   | Display name                                                                                                  |
   | `repo_name`                                                          | GitHub repository name, also the devcontainer workspace folder                                                |
   | `description`                                                        | One line, goes to `pyproject.toml`                                                                            |
   | `client_package`                                                     | Name of the API client package bundled in `custom_components/<domain>/`, or empty when there is none          |
   | `github_owner`, `author_name`, `author_email`                        | **Give the real values**: the defaults are placeholders (`janedoe`, Jane Doe)                                 |
   | `copyright_year`                                                     | Year of creation (default: the current year)                                                                  |
   | `coverage_fail_under`                                                | Coverage floor in percent (default 95). Lower it while there are no tests yet                                 |
   | `python_version`, `python_requires`                                  | Defaults `3.14` and `>=3.14.2,<3.15`. They must satisfy the `homeassistant` pin                               |

   Having a `client_package` enables the `client` lint and test scopes. Without one, there is a single `integration` scope.

4. Commit `.copier-answers.yml` along with the generated files: `copier update` needs it.

## 2. What the template generates

Checked in `template/` on `main`:

- **Environment**: `.devcontainer/` (Dockerfile, `devcontainer.json`, `.common-env`), `.envrc`, `.vscode/`, `pyproject.toml` (a `uv` project), `pyrightconfig.json`, `.yamllint`.
- **Scripts** (`scripts/`, see `template/scripts/README.md`): `setup`, `lint`, `test`, `develop`, `update_requirements`, `clean_bruno_headers`.
- **Quality gates**: `.pre-commit-config.yaml` and `.commitlintrc.mjs` (commit types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`; format `type: subject`, no body or footer).
- **GitHub**: workflows `lint`, `test`, `release`, `stale`, `sync-labels`, `template-drift`, `validate-for-hacs`, `validate-with-hassfest`; `dependabot.yml`; issue templates; `.github/labels.json`.
- **Claude Code**: `.claude/commands` (`create-develop-pull-request`, `create-main-pull-request`, `suggest-commit-title`) and `.mcp.json` (GitHub MCP server).
- **Tests**: `tests/__init__.py`, `tests/conftest.py` and `tests/test_integration_loads.py` are seeded once and then belong to the repository.
- `LICENSE` (MIT, from `author_name` and `copyright_year`).

Not generated (no such file in `template/`): the integration code itself (`custom_components/<domain>/`, with its `manifest.json`), a `README.md`, and a `hacs.json`. You write them.

## 3. Create the integration skeleton

Create `custom_components/<domain>/` with at least `__init__.py` and `manifest.json`. The `release` workflow rewrites the `version` key of `manifest.json` from the tag of the GitHub release, and zips that folder, so the folder name must be exactly `integration_domain`. `validate-with-hassfest` and `validate-for-hacs` check the result in CI, so `manifest.json` and the HACS metadata must satisfy them (not checked here: the exact required fields, see the Home Assistant and HACS documentation).

## 4. Start the development environment

1. Open the repository in VS Code and choose "Reopen in Container". `postCreateCommand` runs `scripts/setup`: `uv sync`, the `default_config` packages, the git hooks (`prek install`: pre-commit and commit-msg).
2. If `git` does not know your name and email yet, run `scripts/setup` once in a terminal (the question in the container gives up after a minute).
3. Check that everything is green, then commit:

   ```bash
   scripts/lint
   scripts/test
   ```

   If `scripts/test` fails on the coverage floor because the repository has few or no tests, lower it for now: `uv run copier update -d coverage_fail_under=0`, and raise it as tests are written.

4. `scripts/develop` starts Home Assistant against the gitignored `./config` (port 8123 is forwarded).

## 5. Configure GitHub

Done by hand on the new repository (the template cannot do it):

1. **Labels**: push to `main` and let `sync-labels.yml` create the 14 labels of `.github/labels.json`, or run it from Actions (Sync labels, Run workflow). It creates and updates but never deletes: remove the default GitHub labels (`bug`, `enhancement`, `question`, ...) by hand if you want the exact list.
2. **Branches**: the Claude commands target `develop` for feature pull requests and `main` for the release (`pr:release`), so create a `develop` branch. Whether `develop` or `main` is the default branch is your choice (not set by the template).
3. **Branch protection** (optional): the `lint` workflow is written to be usable as a required status check, even on pull requests that touch no Python file.
4. **Dependabot** is configured by `.github/dependabot.yml`; nothing to do besides enabling alerts if you want them.

## 6. First release

Releases are made from the GitHub UI: publishing a release with a tag such as `v0.1.0` triggers `release.yml`, which sets the manifest version, zips `custom_components/<domain>/` (without `*.md`, `*.bru` and `bruno/`) and attaches `<domain>.zip` to the release.

## 7. Staying in step with the template

- `template-drift.yml` runs every Monday and turns red when a newer template tag exists. Update with `uv run copier update`, resolve conflicts, run `scripts/lint` and `scripts/test`, commit.
- Template changes are published by tags `vX.Y.Z` on `ha-dev-environment`. Without a tag, Copier cannot tell what is newer.
- Do not edit the managed scripts and workflows in the project to change them everywhere: change them in `ha-dev-environment` and release. Local edits survive `copier update` (three-way merge) but can conflict.

## Quick checklist

- [ ] Repository created, `copier copy gh:bastgau/ha-dev-environment .` run with real `github_owner`, `author_name`, `author_email`
- [ ] `.copier-answers.yml` committed
- [ ] `custom_components/<domain>/` with `manifest.json` created
- [ ] Devcontainer opens, `scripts/lint` and `scripts/test` pass
- [ ] Labels synced, default GitHub labels removed, `develop` branch created
- [ ] CI green (lint, test, hassfest, HACS)
- [ ] First release `v0.1.0` published
