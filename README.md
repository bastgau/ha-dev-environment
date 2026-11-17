# ha-dev-environment

Shared development environment for bastgau's Home Assistant custom integrations: devcontainer, `uv` project layout, lint/test scripts, GitHub workflows, pre-commit. It is a [Copier](https://copier.readthedocs.io) template.

## Installation

The installation, from a console on your host system, and the creation of a new integration repository are described in [docs/new-integration.md](docs/new-integration.md).

## Use

```bash
uvx --from copier copier copy gh:bastgau/ha-dev-environment .   # first time, from the repository root
uvx --from copier copier update                                  # later: pull in template changes
```

Copier asks for the project variables (see `copier.yml`) and records the answers in `.copier-answers.yml`; commit that file. Local edits to generated files survive `copier update` (three-way merge).

| Variable                                                             | Purpose                                                                                                                                             |
| -------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `integration_domain`, `integration_name`, `repo_name`, `description` | Identity of the integration                                                                                                                         |
| `client_package`                                                     | Bundled API client under `custom_components/<domain>/` (optional) — enables the `client` lint/test scopes                                           |
| `github_owner`                                                       | GitHub account of the repository: project URLs in `pyproject.toml`, assignee of the `.claude/commands` pull request commands. Defaults to `janedoe` |
| `author_name`, `author_email`                                        | Author and maintainer in `pyproject.toml`; `author_name` is also the copyright holder of `LICENSE`. Default to Jane Doe, `jane.doe@example.com`     |
| `copyright_year`                                                     | First year of the `LICENSE` notice (MIT): the year the repository was created. Defaults to the current year                                         |
| `has_card`                                                           | Ship a Lovelace card (default `false`): placeholder card, `card.py` that serves it, its test and manifest `after_dependencies`                      |
| `coverage_fail_under`                                                | Minimum coverage, in percent, `scripts/test` and CI require of each test scope. Defaults to 95                                                      |
| `ha_minimum_version`                                                 | Minimum Home Assistant version supported (the `homeassistant` key of `hacs.json`), shown as a warning in the bug report. Defaults to 2026.10.0      |

`.devcontainer/.common-env` is generated from these and is the single place scripts and workflows read them from.

## Adopting it in an existing repository

Run `uvx --from copier copier copy gh:bastgau/ha-dev-environment .` with `--overwrite` (the files already exist), then review `git diff`: anything you want to keep from the old version goes back as a local edit. Always copy from the `gh:` address and not from a local clone: the source is recorded in `.copier-answers.yml` and `copier update` uses it later.

## Updating

`uvx --from copier copier update`, from the root of the target repository, replays the template changes since the version recorded in `.copier-answers.yml` and keeps your local edits (three-way merge). Resolve any conflict, run `scripts/lint` and `scripts/test`, commit.

An update can ask for variables that did not exist when the repository was generated (see `copier.yml`; with `--defaults`, each takes its default). Give the real values once with `-d name=value`: they are recorded in `.copier-answers.yml`.

- `copyright_year`, `github_owner`, `author_name` and `author_email`: the defaults are placeholders (the current year, `janedoe`, Jane Doe). For instance `uvx --from copier copier update -d copyright_year=<year> -d github_owner=<owner> -d author_name="<name>" -d author_email=<email>`. Where the existing `LICENSE` carries another year, the update leaves a conflict in `LICENSE`: keep the `after updating` side.
- `coverage_fail_under` (95 by default): a repository with few tests or none fails that floor in CI as soon as `tests/` exists. Give it a floor it meets for now, for instance `-d coverage_fail_under=0`, and raise it as tests are written (`-d coverage_fail_under=95`, or edit `.copier-answers.yml` then update again).

Copier decides what is "newer" from **version tags** (`v0.1.0`, `v0.2.0`…) on this repository. Without a tag it falls back to `HEAD`, and `copier check-update` cannot tell anything. So: **tag a release (`vX.Y.Z`) after merging a template change**. Verified in a scratch copy: with `v0.1.0` recorded and `v0.2.0` published, `copier check-update --quiet` exits with status 2.

## Keeping the repositories in step

- Each target repository gets `.github/workflows/template-drift.yml`: every Monday it runs `copier check-update` and turns red when a newer template tag exists. This repository is public, so the workflow needs no token.
- Each target repository also gets `.github/workflows/template-update.yml`: when a newer template tag exists, it opens a pull request `template-update/<tag>` against `develop` with `copier update` applied, and closes the previous one. Its token is `GITHUB_TOKEN`, so the repository must allow Actions to create pull requests.

Fixed choices shared by every repository: Prettier skips `card/` and every `bruno/` folder and leaves `.copier-answers.yml` alone; `tests/testing_config` and `tests/typing.py` are excluded from the linters when they exist; a `stale.yml` workflow closes inactive issues; `.github/ISSUE_TEMPLATE` holds the issue templates (bug report and feature proposal as issue forms, blank issues disabled), whose labels all come from the labels list below.

- This repository's own CI (`check-template.yml`, which runs `scripts/check-template`) runs [ShellCheck](https://www.shellcheck.net) over the bash scripts of `template/scripts/` and `scripts/`, renders the template for a project with and without a client, parses the generated scripts, resolves the generated `pyproject.toml` with `uv lock` and runs the commit-msg hook (commitlint, against `.commitlintrc.mjs`) on a valid and an invalid message. It does not run the linters or the tests: they need real integration code.
  ShellCheck is pinned through PyPI (`shellcheck-py`), so `scripts/check-template` reports locally what CI reports.

## Labels

`template/.github/labels.json` is the reference list of GitHub labels for every project: one `pr:<type>` label per commit type allowed by `.commitlintrc.mjs` (`feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`), `pr:release` for the pull request that releases `develop` to `main`, and the `issue:*` labels used by the issue templates and by the stale workflow. Each entry has a `name`, a `color` (six hexadecimal digits, no `#`) and a `description`.

Every repository gets the list as `.github/labels.json` and a `sync-labels.yml` workflow that creates the labels, or updates their colour and description, whenever the list changes or on demand (Actions, Sync labels, Run workflow). It never deletes a label. This repository applies the same list to itself.

To add or change a label: edit `template/.github/labels.json`, merge, release, then `copier update` in each repository.

## Repository settings

`template/.github/repo-settings.json` is the reference for the GitHub settings of every repository (default branch, wiki, merge options, what Actions may do, security features): a snapshot of `ha-pi-hole-v6`. Every repository gets it as `.github/repo-settings.json`, with `scripts/sync-repo-settings`, which compares a repository with it and aligns it on demand. It needs `gh` (`repo` scope) and `jq`.

```bash
scripts/sync-repo-settings diff  --from .github/repo-settings.json bastgau/ha-clouding bastgau/ha-globe-radio
scripts/sync-repo-settings apply --from .github/repo-settings.json bastgau/ha-clouding           # dry-run
scripts/sync-repo-settings apply --from .github/repo-settings.json bastgau/ha-clouding --yes     # writes
```

`apply` only changes the settings that differ, and does nothing without `--yes`. The default branch is reported but never renamed. Settings GitHub does not expose for a repository (branch protection and rulesets on a private repository of a free plan) are out of scope. List keys to skip, one per line, in `.github/repo-settings.ignore`.

`copier copy` and `copier update` run `apply` as a dry-run on the repository named by the `origin` remote, so the differences are listed and nothing is written. The task is skipped when there is no GitHub remote yet.

To change the reference: edit `template/.github/repo-settings.json`, or regenerate it from another repository with `scripts/sync-repo-settings snapshot <owner>/<repo> > template/.github/repo-settings.json`. Merge, release, then `copier update` in each repository.

## MCP servers

`template/.mcp.json` declares the GitHub MCP server shared by every repository; the token is read from the `GITHUB_PERSONAL_ACCESS_TOKEN` environment variable, which the devcontainer takes from `.devcontainer/.env`. That file is seeded once with the placeholder `xxxxx` (replace it with your token) and is gitignored, so nothing secret is committed.

A server that only one repository needs: add its entry to that repository's own `.mcp.json`. `copier update` merges it with the template's changes and keeps it (checked on a scratch copy: a local `chrome-devtools` entry survived a template change to the `github` entry). A conflict can only happen if both sides edit the same lines; resolve it like any merge conflict. A server tied to your machine (a local address, a personal path) does not belong in the repository at all: add it with `claude mcp add --scope local` or `--scope user` instead.

## Layout

The template also ships eight Claude Code agents in `.claude/agents/` (`ha-architect`, `ha-developer`, `ha-test-writer`, `ha-reviewer`, `ha-ci-quality-guardian`, `ha-release-manager`, `ha-lookup`, `ha-webdesigner`), generic and read from a project `CLAUDE.md` that you write, and the brand tooling the web designer uses in `docs/brand/`. They are kept in step by `copier update` (improve them here, not in the repositories); see [docs/new-integration.md](docs/new-integration.md) for the pipeline.

`template/` mirrors the target repository. Files ending in `.jinja` are rendered; the others are copied as they are. `.devcontainer/requirements-default-config.txt` is seeded once and never overwritten (regenerate it with `scripts/update_requirements`). `.github/.issue-extra.yml` is seeded once too, then belongs to the repository: it lists the blocks the bug report adds to the template's own fields (the version of a device, a compatibility note), through `_external_data`. So are `tests/__init__.py`, `tests/conftest.py` and `tests/test_integration_loads.py`, a first test that checks Home Assistant finds the integration and imports it: a repository that already has one of them, such as tests vendored from Home Assistant core, keeps its own. Also seeded once, so that a new project starts complete and an existing one keeps its own: `README.md` (the badges, the "Add Repository" and "Add Integration" buttons, installation through HACS or by hand, development, license), `hacs.json`, the integration skeleton `custom_components/<domain>/__init__.py` and `manifest.json`, and with a `client_package` the client package and its first test (`__init__.py`, `test_integration_loads.py` in `client_tests_path`), and with `has_card` the card registration `card.py`, the placeholder card in `frontend/` and `tests/test_card.py` (`config_flow` false, `integration_type` `service`, `iot_class` `cloud_polling` as placeholders to adjust, version `0.0.0` rewritten by the release workflow).

## Checked

Rendered for a project without a client and for one with a client in a scratch copy: `uv lock`, `scripts/lint` (all scopes) and `scripts/test` pass, and `copier update` applies a template change while keeping a local edit. Not checked: the GitHub workflows themselves, a devcontainer rebuild, Dependabot's `uv` support.
