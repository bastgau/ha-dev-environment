# Starting a new Home Assistant integration repository

How to create the repository of a new Home Assistant custom integration from this template. For adopting the template in a repository that already exists, see "Adopting it in an existing repository" in the [README](../README.md).

## Prerequisites

- `git`, and a GitHub account that can create repositories.
- [uv](https://docs.astral.sh/uv/getting-started/installation/) and Copier 9.0.0 or newer (`uv tool install copier`). See "Installation" in the [README](../README.md).
- For the devcontainer: Docker and VS Code with the Dev Containers extension. The container forwards `GITHUB_PERSONAL_ACCESS_TOKEN` from your host to the GitHub MCP server (`remoteEnv` in `devcontainer.json`), so export it on the host if you use Claude Code in the container.
- Optional, for the GitHub settings below from a terminal: the [GitHub CLI](https://cli.github.com) (`gh`), authenticated. Everything can also be done in the GitHub web interface.

## 1. Create the repository

1. Create an empty repository on GitHub. Name it `ha-<domain>`, with underscores of the domain written as dashes: this is the default of `repo_name` in `copier.yml`.
2. Clone it and generate the project from the template, from the repository root. Always use the `gh:` address and not a local clone: the source is recorded in `.copier-answers.yml`, and `copier update` uses it later.

   ```bash
   git clone git@github.com:<owner>/ha-<domain>.git
   cd ha-<domain>
   copier copy gh:bastgau/ha-dev-environment .
   ```

3. Answer the questions (see `copier.yml`):

   | Variable                                      | What to give                                                                                                       |
   | --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
   | `integration_domain`                          | Name of the `custom_components/<domain>` folder. Lowercase letters, digits and underscores, starting with a letter |
   | `integration_name`                            | Display name                                                                                                       |
   | `repo_name`                                   | GitHub repository name, also the devcontainer workspace folder                                                     |
   | `description`                                 | One line, goes to `pyproject.toml`. Reuse it as the GitHub description (step 5)                                    |
   | `client_package`                              | Name of the API client package bundled in `custom_components/<domain>/`, or empty when there is none               |
   | `github_owner`, `author_name`, `author_email` | **Give the real values**: the defaults are placeholders (`janedoe`, Jane Doe)                                      |
   | `copyright_year`                              | Year of creation (default: the current year)                                                                       |
   | `coverage_fail_under`                         | Coverage floor in percent (default 95). Lower it while there are no tests yet                                      |
   | `python_version`, `python_requires`           | Defaults `3.14` and `>=3.14.2,<3.15`. They must satisfy the `homeassistant` pin                                    |

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

Also generated, once (`_skip_if_exists` in `copier.yml`: a repository that already has them keeps its own, and later `copier update` runs never touch them):

- `README.md`: the badges of `ha-feedparser` (maintainer, Python, Home Assistant, release, HACS and hassfest validation), installation through HACS or by hand with the "Add Repository" and "Add Integration" buttons, a configuration placeholder, development, license. The "Add Integration" button only works once the integration has a config flow.
- `hacs.json` (`name`, `render_readme`).
- The integration skeleton `custom_components/<domain>/`: `__init__.py` (a docstring and `DOMAIN`) and `manifest.json` (`domain`, `name`, `codeowners`, `documentation`, `issue_tracker`, `version`, and placeholders to adjust: `config_flow` false, `integration_type` `service`, `iot_class` `cloud_polling`).

## 3. Adjust the generated integration

The skeleton makes `scripts/lint` and `scripts/test` pass as it is; the first test checks Home Assistant finds it. Then make it yours:

1. In `manifest.json`: set `iot_class`, `integration_type`, `requirements` and, with a config flow, `config_flow` (and add `config_flow.py`). Keep `version` as it is: the `release` workflow rewrites it from the tag of the GitHub release and zips the folder, so the folder name must stay exactly `integration_domain`.
2. In `README.md`: fill in the "Configuration" section.
3. Write the integration code in `custom_components/<domain>/`.

Checked on a rendered project: `scripts/lint` (all of `integration`, `tests`, `repo`) passes and `scripts/test` passes (1 test, 100 % coverage). Not run here: `validate-with-hassfest` and `validate-for-hacs` (they run in GitHub Actions), so whether they accept the generated `manifest.json` and `hacs.json` as they are is not verified: check the first CI run.

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

## 5. Configure the GitHub repository

Done by hand on the new repository (the template cannot do it). Once the generated files are pushed:

### 5.1 Description

Set the repository description (the "About" box), the same line as the `description` variable:

```bash
gh repo edit <owner>/ha-<domain> --description "<one-line description>"
```

Or: repository page, gear icon next to "About", Description.

### 5.2 Topics

Add topics so the repository can be found. Base set, taken from the topics of `ha-feedparser`: `hacs`, `hacs-integration`, `home-assistant`, `homeassistant`. Add the ones specific to the service or device (`ha-feedparser` has `feedparser`, `rss`, `rss-feed`, `rss-reader`). Topics are lowercase, with letters, digits and dashes.

```bash
gh repo edit <owner>/ha-<domain> \
  --add-topic hacs --add-topic hacs-integration \
  --add-topic home-assistant --add-topic homeassistant \
  --add-topic <specific-topic>
```

Or: gear icon next to "About", Topics.

Not checked: whether `validate-for-hacs` requires a description or topics. HACS's own documentation says what a custom repository needs.

### 5.3 Labels (pull requests and issues)

`.github/labels.json` holds the 14 reference labels: 8 `pr:*` (`feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `release`) and 6 `issue:*` (`bug`, `enhancement`, `triage`, `stale`, `question`, `documentation`). The `sync-labels.yml` workflow creates them, or updates their colour and description:

1. It runs on its own when a push changes `.github/labels.json` or `.github/workflows/sync-labels.yml`, so the push of the generated files normally triggers it (assumption from the `paths` filter, not run on a new repository).
2. Otherwise run it by hand: Actions, "Sync labels", "Run workflow", or `gh workflow run sync-labels.yml --repo <owner>/ha-<domain>`.
3. Check the result: Issues, Labels, or `gh api repos/<owner>/ha-<domain>/labels --paginate --jq '.[].name'`.

The workflow never deletes a label. A new repository comes with GitHub's default labels (`bug`, `enhancement`, `documentation`, `question`, `good first issue`, `help wanted`, `wontfix`, `duplicate`, `invalid`): delete them if you want exactly the 14 reference labels.

```bash
for l in bug enhancement documentation question "good first issue" "help wanted" wontfix duplicate invalid; do
  gh label delete "$l" --repo <owner>/ha-<domain> --yes
done
```

The issue templates, the stale workflow and the `.claude/commands` pull request commands use these labels, so sync them before opening the first issue or pull request.

### 5.4 Branches and protection

The Claude commands target `develop` for feature pull requests and `main` for the release (`pr:release`), so create a `develop` branch. Whether `develop` or `main` is the default branch is your choice (not set by the template). Branch protection is optional: the `lint` workflow is written to be usable as a required status check, even on pull requests that touch no Python file.

### 5.5 Dependabot

Configured by `.github/dependabot.yml`; nothing to do besides enabling alerts if you want them.

## 6. First release

Releases are made from the GitHub UI: publishing a release with a tag such as `v0.1.0` triggers `release.yml`, which sets the manifest version, zips `custom_components/<domain>/` (without `*.md`, `*.bru` and `bruno/`) and attaches `<domain>.zip` to the release.

## 7. Staying in step with the template

- `template-drift.yml` runs every Monday and turns red when a newer template tag exists. Update with `uv run copier update`, resolve conflicts, run `scripts/lint` and `scripts/test`, commit.
- Template changes are published by tags `vX.Y.Z` on `ha-dev-environment`. Without a tag, Copier cannot tell what is newer.
- Do not edit the managed scripts and workflows in the project to change them everywhere: change them in `ha-dev-environment` and release. Local edits survive `copier update` (three-way merge) but can conflict.

## Quick checklist

- [ ] Repository created, `copier copy gh:bastgau/ha-dev-environment .` run with real `github_owner`, `author_name`, `author_email`
- [ ] `.copier-answers.yml` committed
- [ ] Generated `manifest.json` adjusted (`iot_class`, `integration_type`, `requirements`), `README.md` completed
- [ ] Devcontainer opens, `scripts/lint` and `scripts/test` pass
- [ ] Description set
- [ ] Topics added
- [ ] Labels synced (14 `pr:*` and `issue:*`), default GitHub labels removed
- [ ] `develop` branch created
- [ ] CI green (lint, test, hassfest, HACS)
- [ ] First release `v0.1.0` published
