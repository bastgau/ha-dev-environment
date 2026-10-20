# ha-dev-environment

Shared development environment for bastgau's Home Assistant custom integrations (`ha-pi-hole-v6`, `ha-device-link-tools`, `ha-betaseries`, `ha-clouding`): devcontainer, `uv` project layout, lint/test scripts, GitHub workflows, pre-commit. It is a [Copier](https://copier.readthedocs.io) template.

## Use

```bash
copier copy gh:bastgau/ha-dev-environment .   # first time, from the repository root
copier update                                  # later: pull in template changes
```

Copier asks for the project variables (see `copier.yml`) and records the answers in `.copier-answers.yml`; commit that file. Local edits to generated files survive `copier update` (three-way merge).

| Variable                                                             | Purpose                                                                                                   |
| -------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| `integration_domain`, `integration_name`, `repo_name`, `description` | Identity of the integration                                                                               |
| `client_package`                                                     | Bundled API client under `custom_components/<domain>/` (optional) — enables the `client` lint/test scopes |
| `prettier_excludes`                                                  | Vendored or generated folders Prettier must skip                                                          |
| `ha_testing_config`, `strict_docstrings`                             | Per-project strictness switches                                                                           |

`.devcontainer/.common-env` is generated from these and is the single place scripts and workflows read them from.

## Adopting it in an existing repository

Run `copier copy gh:bastgau/ha-dev-environment .` with `--overwrite` (the files already exist), then review `git diff`: anything you want to keep from the old version goes back as a local edit. Always copy from the `gh:` address and not from a local clone: the source is recorded in `.copier-answers.yml` and `copier update` uses it later.

## Updating

`copier update` in the target repository replays the template changes since the version recorded in `.copier-answers.yml` and keeps your local edits (three-way merge). Resolve any conflict, run `scripts/lint` and `scripts/test`, commit.

Copier decides what is "newer" from **version tags** (`v0.1.0`, `v0.2.0`…) on this repository. Without a tag it falls back to `HEAD`, and `copier check-update` cannot tell anything. So: **tag a release (`vX.Y.Z`) after merging a template change**. Verified in a scratch copy: with `v0.1.0` recorded and `v0.2.0` published, `copier check-update --quiet` exits with status 2.

## Keeping the repositories in step

- Each target repository gets `.github/workflows/template-drift.yml`: every Monday it runs `copier check-update` and turns red when a newer template tag exists. While this repository is private, give it a `TEMPLATE_TOKEN` secret (a token with read access to it); the workflow does without once the repository is public.
- This repository's own CI (`check-template.yml`, which runs `scripts/check-template`) renders the template for a project with and without a client, parses the generated scripts and resolves the generated `pyproject.toml` with `uv lock`. It does not run the linters or the tests: they need real integration code.

## MCP servers

`template/.mcp.json` declares the GitHub MCP server shared by every repository; the token comes from `GITHUB_PERSONAL_ACCESS_TOKEN`, which the devcontainer forwards from your host (`remoteEnv`). Nothing secret is committed.

A server that only one repository needs: add its entry to that repository's own `.mcp.json`. `copier update` merges it with the template's changes and keeps it (checked on a scratch copy: a local `chrome-devtools` entry survived a template change to the `github` entry). A conflict can only happen if both sides edit the same lines; resolve it like any merge conflict. A server tied to your machine (a local address, a personal path) does not belong in the repository at all: add it with `claude mcp add --scope local` or `--scope user` instead.

## Layout

`template/` mirrors the target repository. Files ending in `.jinja` are rendered; the others are copied as they are. `.devcontainer/requirements-default-config.txt` is seeded once and never overwritten (regenerate it with `scripts/update_requirements`).

## Checked

Rendered for `ha-device-link-tools` (no client) and `ha-betaseries` (with client) in a scratch copy: `uv lock`, `scripts/lint` (all scopes) and `scripts/test` pass, and `copier update` applies a template change while keeping a local edit. Not checked: the GitHub workflows themselves, a devcontainer rebuild, Dependabot's `uv` support.
