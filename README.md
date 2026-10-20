# ha-dev-environment

Shared development environment (devcontainer, scripts, CI, tooling) for bastgau's Home Assistant custom integrations:
`ha-pi-hole-v6`, `ha-device-link-tools`, `ha-betaseries`, `ha-clouding`.

`template/` mirrors the layout of a target repository. Files in it are generic: copy them as they are.
Per-project values (integration domain, optional bundled client package) live in the target's
`.devcontainer/.common-env`, never in the template.

Status: work in progress. Only the files identical across the repositories are here so far; the parametrised
ones (`scripts/lint`, `scripts/test`, `scripts/setup`, `devcontainer.json`, `pyproject.toml`, lint/test/release workflows,
`.pre-commit-config.yaml`) and the sync script come next, together with the move to uv.
