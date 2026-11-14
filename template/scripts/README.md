# Development scripts

Managed by [ha-dev-environment](https://github.com/bastgau/ha-dev-environment): change them there, then run `uvx --from copier copier update`.

Every script resolves the repository root from its own location and reads the project variables (integration domain, optional bundled client, Prettier exclusions) from `.devcontainer/.common-env`, so it can be run from anywhere.

| Script                | What it does                                         | Modifies files                                                     |
| --------------------- | ---------------------------------------------------- | ------------------------------------------------------------------ |
| `setup`               | Builds the development environment                   | yes — `.venv`, `.git/config`, `.git/hooks`, `~/.vimrc`, global npm |
| `lint`                | Runs the five CI checks, plus Prettier               | yes — the `repo` scope reformats in place                          |
| `test`                | Runs pytest with the coverage gates                  | no                                                                 |
| `develop`             | Starts Home Assistant against `./config`             | yes — `./config` (gitignored)                                      |
| `update_requirements` | Regenerates the `default_config` requirements        | yes — `.devcontainer/requirements-default-config.txt`              |
| `clean_bruno_headers` | Strips noisy response headers from Bruno examples    | yes — the client's `bruno/*.bru` (none with `--check`)             |
| `sync-repo-settings`  | Compares GitHub repository settings with a reference | no — GitHub settings, only with `--yes`                            |

`_common` is sourced by the others; it is not meant to be run.

## `setup`

Asks for the Git name and email that `git config` does not already have (VS Code usually copies them from the host) and records them in the repository's `.git/config`, which survives a container rebuild. When the devcontainer runs it (`postCreateCommand`), the question gives up after a minute without an answer, so that it never blocks the creation of the container: if git does not know you yet, run `scripts/setup` once in a terminal.

Then `uv sync` (the groups declared in `pyproject.toml`), then the `default_config` packages on top. Those are kept out of `uv.lock` on purpose: they only exist so a local `hass` can boot, and tracking them would flood Dependabot. The consequence is that **every other script runs `uv run --no-sync`** — a sync would uninstall them. Then `prek install` writes two git hooks: `.git/hooks/pre-commit` runs `.pre-commit-config.yaml` over the staged files, and `.git/hooks/commit-msg` checks the commit message against `.commitlintrc.mjs` (commitlint). The hook environments are built on the first commit. Also installs Prettier globally (pinned to the pre-commit hook's version) and a `~/.vimrc`.

## `lint`

```bash
scripts/lint [scope ...]     # default: all
```

| Scope         | Paths                                                       | Checks                                                                                                                                                                                     |
| ------------- | ----------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `client`      | `custom_components/<domain>/<client>` (if there is one)     | ruff check, ruff format, pylint, pyright, docstring-linter                                                                                                                                 |
| `integration` | `custom_components/<domain>/*.py`                           | same five                                                                                                                                                                                  |
| `tests`       | `tests/`                                                    | same five                                                                                                                                                                                  |
| `js`          | the `.js` / `.mjs` files                                    | `prettier --check` (no write), then `node --check` and ESLint (what CI runs)                                                                                                               |
| `repo`        | `**/*.{json,yaml,yml,md,js,mjs}` minus the excluded folders | Prettier (**writes**), then `node --check` and ESLint on the `.js` / `.mjs` files under `custom_components/` and `tests/` (pinned `npx eslint`, `eslint.config.mjs`, skipped without node) |

`.github/workflows/lint.yml` asks `scripts/lint --list-scopes` which scopes exist, so **a green run here predicts a green CI run** by construction. Every check runs even after one fails.

## `test`

```bash
scripts/test [client|integration|all]
```

Each scope has its own coverage floor, `COVERAGE_FAIL_UNDER` in `.devcontainer/.common-env` (95 % by default, see `coverage_fail_under` in `.copier-answers.yml`), and covers what it owns on its own. Without a bundled client there is a single `integration` scope. To iterate on one test, bypass the gate: `uv run --no-sync pytest tests/test_x.py -q`.

## `update_requirements`

Walks the manifests reachable from `default_config` and rewrites the generated block of `.devcontainer/requirements-default-config.txt`. Run it **whenever `homeassistant` is upgraded**, review the diff, then run `scripts/setup`.

## `clean_bruno_headers`

Only does something when the bundled client ships a Bruno collection in `custom_components/<domain>/<client>/bruno/`; otherwise it says so and exits 0. Bruno records the real response headers of an example (timestamps, Cloudflare metadata, key and token fingerprints), which must not be committed. `--check` reports without writing and exits 1; the pre-commit hook runs it that way whenever a `.bru` file is staged.

## `sync-repo-settings`

```bash
scripts/sync-repo-settings snapshot <owner>/<repo>                                # settings as JSON
scripts/sync-repo-settings diff  --from .github/repo-settings.json <owner>/<repo>
scripts/sync-repo-settings apply --from .github/repo-settings.json <owner>/<repo> [--yes]
```

Reads the GitHub settings of a repository (default branch, wiki, merge options, Actions permissions, security features) and compares them with `.github/repo-settings.json`. `apply` is a dry-run unless `--yes` is given, and only touches what differs. The default branch is reported, never renamed. Needs `gh` and `jq`; it does not use `.devcontainer/.common-env`. Keys listed in `.github/repo-settings.ignore` (one per line) are skipped. Exit status 1 when a repository differs.
