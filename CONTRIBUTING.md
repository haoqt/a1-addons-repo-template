# Contributing

Thanks for improving `a1-addons-repo-template`.

## Ground rules

1. **Do not add OCA-specific defaults.** This template is an A1 fork; anything
   like `OCB` test flavor, codecov upload, makepot, stale-action belongs in
   the upstream OCA template, not here.
2. **Test render before you commit.** Smoke-test with
   `uv run copier copy . /tmp/probe --defaults --trust` and inspect the output
   tree. If you can, try both a legacy version (e.g. `--data odoo_version=12.0`)
   and a modern one (`--data odoo_version=19.0`).
3. **One variable = one purpose.** If a new toggle only affects one line of
   one file, prefer hard-coding the sensible default.

## Layout

```
copier.yml            Template variables, defaults, conditionals
src/                  Everything that gets rendered into the target repo
    .github/workflows/  CI templates
    …                 Jinja templates (.jinja) or plain files
```

Files whose *name* depends on the answers use Jinja in the filename, e.g.
`{% if odoo_version < 13 %}.flake8{% endif %}.jinja` — an empty result skips
the file entirely.

## Release

Bump the version in `pyproject.toml`, tag with `git tag vX.Y.Z`, push tag. No
package publish step (this template is consumed via `copier copy gh:a1/...`).
