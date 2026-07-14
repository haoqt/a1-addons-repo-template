# Changelog

All notable changes to `odoo-project-github-template` are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)
and this project adheres to [Semantic Versioning](https://semver.org/).

## [1.1.1] — 2026-07-14

### Fixed

- **`.pylintrc`: drop `manifest-required-authors`.** Real projects
  vendor 3rd-party addons (Cybrosys, iPredict, Sensible, Port Cities,
  OCA…) that carry their own author names. Enforcing an author
  allow-list floods CI with `manifest-required-author` warnings that
  can only be silenced by patching upstream manifests. The setting is
  removed from `[ODOOLINT]`; a NOTE explains when to turn it back on
  (first-party-only repos). Matches BELGO baseline.

## [1.1.0] — 2026-07-14

Removed ruff. Standardised the lint stack on
**black + isort + pyupgrade + flake8 + prettier + OCA odoo-module hooks +
pylint-odoo** to match the A1 BELGO project baseline.

### Removed

- `use_ruff` and `additional_ruff_rules` copier variables.
- `.ruff.toml` template file. Existing repos: `copier update --trust`
  will drop the file; if you had local edits, save them elsewhere first.
- Ruff branch of `.pre-commit-config.yaml.jinja`.

### Added

- `pyupgrade` and OCA `oca-checks-odoo-module` + `oca-checks-po` hooks
  (previously only enabled behind `use_ruff=no`).

### Changed

- `.flake8` and `.isort.cfg` now always render (previously only when
  `not use_ruff and odoo_version < 17.0`).

## [1.0.1] — 2026-07-09

Patch release focused on making the generated CI actually go green
end-to-end on Odoo 18 / 19 Enterprise. Six template bugs surfaced by
the first real render on `haoqt/my-new-repo` are fixed.

### Fixed

- **`test.yml`: pip cache path.** `actions/setup-python@v5`'s built-in
  `cache: pip` scans the workspace root, but the workflow checks out
  the target repo into a subdirectory (so `../odoo` can sit next to
  it). setup-python hard-errored with:
  ```
  No file matched to [**/requirements.txt or **/pyproject.toml]
  ```
  Replaced with an explicit `actions/cache@v4` step keyed off
  `<repo>/requirements.txt`; missing requirements.txt is now a
  no-op instead of a fatal error.

- **`.pylintrc`: obsolete pylint codes.** Removed `E0501`, `E0502`,
  `E0503`, `W0110`, `W0312`, `W0623` from the `enable=` list —
  modern pylint (>=3.0) no longer knows them and warned
  `unknown-option-value` on every run. Also dropped `C0103`
  (false-positive on Odoo model names) and `C0301` (line-too-long
  is now solely ruff's job).

- **`test.yml`: preflight `ODOO_ENTERPRISE_TOKEN`.** When the secret
  was missing, the enterprise clone failed ~30s into the job with a
  terse `fatal: Authentication failed`. Added a preflight step that
  fails within seconds with an actionable error containing the
  `gh secret set` command and the "switch to community" fallback.

- **`test.yml`: Odoo system build dependencies.** Odoo's
  `requirements.txt` pulls source distributions of `python-ldap`,
  `lxml`, `Pillow`, `cryptography`, `psycopg2` on Ubuntu runners.
  Without native build deps the job died with:
  ```
  Modules/common.h:15:10: fatal error: lber.h: No such file or directory
  ```
  Added an `Install Odoo system dependencies` step upfront that
  installs `libldap2-dev`, `libsasl2-dev`, `libxml2-dev`,
  `libxslt1-dev`, `libjpeg-dev`, `zlib1g-dev`, `libtiff-dev`,
  `libffi-dev`, `libssl-dev`, `libpq-dev`, `libpng-dev`,
  `libfreetype-dev`, `build-essential`. Absorbed the previous
  `include_wkhtmltopdf` step to avoid two `apt-get update` runs.

- **`.pylintrc`: E1101 no-member false-positives on Odoo.** Odoo
  composes model classes at runtime (base + `_inherit` siblings),
  so pylint's static analyser cannot resolve inherited methods and
  fires `E1101` on valid `super()._some_odoo_method()` calls. Local
  runs happened to pass because Odoo wasn't on `PYTHONPATH`; CI
  runs against the full Odoo tree surfaced the false positives.
  Removed `E1101` from the enable list.

### Changed

- **Answers file style: `""` for empty strings.** Copier's
  `to_nice_yaml` filter serialises empty strings as `''`. The
  answers-file template now post-processes the output to swap
  `: ''` for `: ""` (matching A1 YAML style). Non-empty
  single-quoted values (e.g. `target_branch: '18.0'`) are
  unaffected because the pattern requires the quotes to be
  adjacent.

## [1.0.0] — 2026-07-09

Initial release. Copier template for A1 Odoo addon repositories.

- Supports Odoo **10.0 → 19.0**, Community and Enterprise.
- Configurable `target_branch`, independent from `odoo_version`.
- `ruff` for Odoo ≥17 or `black`/`isort`/`flake8` for older,
  wired into `pre-commit`.
- `pylint-odoo` with version-aware pin: `9.3.4` (Odoo ≥17),
  `8.0.19` (14-16), `3.5.0` (12-13), `1.6.2` (≤11).
- GitHub Actions: test matrix + pre-commit gate; license
  allow-list check in CI.
- Multi-license `LICENSE` render (OPL-1 default, LGPL-3,
  AGPL-3, or other proprietary).
