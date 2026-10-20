# ha-dev-environment

Shared development environment for bastgau's Home Assistant custom integrations (`ha-pi-hole-v6`, `ha-device-link-tools`, `ha-betaseries`, `ha-clouding`): devcontainer, `uv` project layout, lint/test scripts, GitHub workflows, pre-commit. It is a [Copier](https://copier.readthedocs.io) template.

## Use

```bash
copier copy gh:bastgau/ha-dev-environment .   # first time, from the repository root
copier update                                  # later: pull in template changes
```

Copier asks for the project variables (see `copier.yml`) and records the answers in `.copier-answers.yml`; commit that file. Local edits to generated files survive `copier update` (three-way merge).

| Variable                                                       | Purpose                                                       |
| -------------------------------------------------------------- | ------------------------------------------------------------- |
| `integration_domain`, `integration_name`, `repo_name`, `description` | Identity of the integration                                   |
| `client_package`                                               | Bundled API client under `custom_components/<domain>/` (optional) — enables the `client` lint/test scopes |
| `prettier_excludes`                                            | Vendored or generated folders Prettier must skip              |
| `ha_testing_config`, `strict_docstrings`, `lenient_tests`      | Per-project strictness switches                               |

`.devcontainer/.common-env` is generated from these and is the single place scripts and workflows read them from.

## Layout

`template/` mirrors the target repository. Files ending in `.jinja` are rendered; the others are copied as they are. `.devcontainer/requirements-default-config.txt` is seeded once and never overwritten (regenerate it with `scripts/update_requirements`).

## Checked

Rendered for `ha-device-link-tools` (no client) and `ha-betaseries` (with client) in a scratch copy: `uv lock`, `scripts/lint` (all scopes) and `scripts/test` pass, and `copier update` applies a template change while keeping a local edit. Not checked: the GitHub workflows themselves, a devcontainer rebuild, Dependabot's `uv` support.
