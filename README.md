# ha-dev-environment

Shared development environment for bastgau's Home Assistant custom integrations (`ha-pi-hole-v6`, `ha-device-link-tools`, `ha-betaseries`, `ha-clouding`): devcontainer, `uv` project layout, lint/test scripts, GitHub workflows, pre-commit. It is a [Copier](https://copier.readthedocs.io) template.

## Installation

Copier is a Python command line tool (this template needs Copier 9.0.0 or newer, see `_min_copier_version` in `copier.yml`) and needs `git`. The steps are the same on Linux, macOS and Windows with WSL.

1. Install [uv](https://docs.astral.sh/uv/getting-started/installation/), which the generated repositories use as well:

   ```bash
   curl -LsSf https://astral.sh/uv/install.sh | sh   # Linux, macOS, WSL
   # macOS with Homebrew: brew install uv
   ```

2. Install Copier as a tool, so that it does not touch any project environment:

   ```bash
   uv tool install copier
   copier --version
   ```

   `uvx --from copier copier <command>` runs it without installing anything. Any other way of installing a Python command line tool, such as `pipx install copier`, works too.

To upgrade Copier later: `uv tool upgrade copier`.

In a repository generated from this template, Copier is already there: it is part of the `lint` dependency group, so the devcontainer installs it (`scripts/setup`) and Dependabot keeps it up to date. Run it with `uv run copier update`. The steps above are only needed for the first `copier copy`.

## Use

```bash
copier copy gh:bastgau/ha-dev-environment .   # first time, from the repository root
copier update                                  # later: pull in template changes
```

Copier asks for the project variables (see `copier.yml`) and records the answers in `.copier-answers.yml`; commit that file. Local edits to generated files survive `copier update` (three-way merge).

| Variable                                                             | Purpose                                                                                                                                             |
| -------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `integration_domain`, `integration_name`, `repo_name`, `description` | Identity of the integration                                                                                                                         |
| `client_package`                                                     | Bundled API client under `custom_components/<domain>/` (optional) — enables the `client` lint/test scopes                                           |
| `github_owner`                                                       | GitHub account of the repository: project URLs in `pyproject.toml`, assignee of the `.claude/commands` pull request commands. Defaults to `janedoe` |
| `author_name`, `author_email`                                        | Author and maintainer in `pyproject.toml`; `author_name` is also the copyright holder of `LICENSE`. Default to Jane Doe, `jane.doe@example.com`     |
| `copyright_year`                                                     | First year of the `LICENSE` notice (MIT): the year the repository was created. Defaults to the current year                                         |

`.devcontainer/.common-env` is generated from these and is the single place scripts and workflows read them from.

## Adopting it in an existing repository

Run `copier copy gh:bastgau/ha-dev-environment .` with `--overwrite` (the files already exist), then review `git diff`: anything you want to keep from the old version goes back as a local edit. Always copy from the `gh:` address and not from a local clone: the source is recorded in `.copier-answers.yml` and `copier update` uses it later.

## Updating

The first `copier update` to a version that renders `LICENSE` asks for `copyright_year` (with `--defaults`, the current year is taken): give the year the repository was created, once, for instance `uv run copier update -d copyright_year=2024`. It is recorded in `.copier-answers.yml`. The same goes for `github_owner`, `author_name` and `author_email`, whose defaults are placeholders (`janedoe`, Jane Doe): give the real values on that first update, for instance `uv run copier update -d copyright_year=2024 -d github_owner=bastgau -d author_name="Bastien Gautier" -d author_email=contact@bastiengautier.com`. Where the existing `LICENSE` carries another year (2026 in `ha-custom-universal-media-player`, `ha-pi-hole-v6` and `ha-cup-component`), that update leaves a conflict in `LICENSE`: keep the `after updating` side. Creation years, from each repository's first commit: `ha-custom-universal-media-player` 2024, `ha-pi-hole-v6` and `ha-cup-component` 2025, `ha-clouding`, `ha-betaseries`, `ha-device-link-tools` and `ha-globe-radio` 2026.

`copier update` in the target repository replays the template changes since the version recorded in `.copier-answers.yml` and keeps your local edits (three-way merge). Resolve any conflict, run `scripts/lint` and `scripts/test`, commit.

Copier decides what is "newer" from **version tags** (`v0.1.0`, `v0.2.0`…) on this repository. Without a tag it falls back to `HEAD`, and `copier check-update` cannot tell anything. So: **tag a release (`vX.Y.Z`) after merging a template change**. Verified in a scratch copy: with `v0.1.0` recorded and `v0.2.0` published, `copier check-update --quiet` exits with status 2.

## Keeping the repositories in step

- Each target repository gets `.github/workflows/template-drift.yml`: every Monday it runs `copier check-update` and turns red when a newer template tag exists. This repository is public, so the workflow needs no token.

Fixed choices shared by every repository: Prettier skips `card/` and every `bruno/` folder and leaves `.copier-answers.yml` alone; `tests/testing_config` and `tests/typing.py` are excluded from the linters when they exist; a `stale.yml` workflow closes inactive issues; `.github/ISSUE_TEMPLATE` holds the issue templates (bug report, feature proposal, blank issues disabled), whose labels all come from the labels list below.

- This repository's own CI (`check-template.yml`, which runs `scripts/check-template`) runs [ShellCheck](https://www.shellcheck.net) over the bash scripts of `template/scripts/` and `scripts/`, renders the template for a project with and without a client, parses the generated scripts, resolves the generated `pyproject.toml` with `uv lock` and runs the commit-msg hook (commitlint, against `.commitlintrc.mjs`) on a valid and an invalid message. It does not run the linters or the tests: they need real integration code.
  ShellCheck is pinned through PyPI (`shellcheck-py`), so `scripts/check-template` reports locally what CI reports.

## Labels

`template/.github/labels.json` is the reference list of GitHub labels for every project: one `pr:<type>` label per commit type allowed by `.commitlintrc.mjs` (`feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`), `pr:release` for the pull request that releases `develop` to `main`, and the `issue:*` labels used by the issue templates and by the stale workflow. Each entry has a `name`, a `color` (six hexadecimal digits, no `#`) and a `description`.

Every repository gets the list as `.github/labels.json` and a `sync-labels.yml` workflow that creates the labels, or updates their colour and description, whenever the list changes or on demand (Actions, Sync labels, Run workflow). It never deletes a label. This repository applies the same list to itself.

To add or change a label: edit `template/.github/labels.json`, merge, release, then `copier update` in each repository.

## MCP servers

`template/.mcp.json` declares the GitHub MCP server shared by every repository; the token comes from `GITHUB_PERSONAL_ACCESS_TOKEN`, which the devcontainer forwards from your host (`remoteEnv`). Nothing secret is committed.

A server that only one repository needs: add its entry to that repository's own `.mcp.json`. `copier update` merges it with the template's changes and keeps it (checked on a scratch copy: a local `chrome-devtools` entry survived a template change to the `github` entry). A conflict can only happen if both sides edit the same lines; resolve it like any merge conflict. A server tied to your machine (a local address, a personal path) does not belong in the repository at all: add it with `claude mcp add --scope local` or `--scope user` instead.

## Layout

`template/` mirrors the target repository. Files ending in `.jinja` are rendered; the others are copied as they are. `.devcontainer/requirements-default-config.txt` is seeded once and never overwritten (regenerate it with `scripts/update_requirements`). `.github/ISSUE_TEMPLATE/bug_report.md` is seeded once too: it names the integration, then belongs to the repository, which can add its own fields (the version of a device, a compatibility note). So are `tests/__init__.py`, `tests/conftest.py` and `tests/test_integration_loads.py`, a first test that checks Home Assistant finds the integration and imports it: a repository that already has one of them, such as tests vendored from Home Assistant core, keeps its own.

## Checked

Rendered for `ha-device-link-tools` (no client) and `ha-betaseries` (with client) in a scratch copy: `uv lock`, `scripts/lint` (all scopes) and `scripts/test` pass, and `copier update` applies a template change while keeping a local edit. Not checked: the GitHub workflows themselves, a devcontainer rebuild, Dependabot's `uv` support.
