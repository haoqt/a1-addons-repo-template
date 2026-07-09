# a1-addons-repo-template

A [Copier](https://copier.readthedocs.io/) template for bootstrapping and
maintaining Odoo addon repositories at A1. Supports Odoo **10.0 → 19.0**,
Community and Enterprise editions, with modern lint tooling (ruff,
pylint-odoo, pre-commit) and CI baked in.

Modelled after
[OCA/oca-addons-repo-template](https://github.com/OCA/oca-addons-repo-template),
with OCA-specific options (OCB testing, codecov, makepot, stale-action…)
stripped out and replaced by A1 defaults.

---

## Requirements

- Python 3.10+
- [Copier](https://copier.readthedocs.io/) 9+
  ```bash
  pipx install copier         # or:  uv tool install copier
  ```

---

## Quick start

### 1. Create a new addon repository

```bash
copier copy --trust gh:a1/a1-addons-repo-template my-new-repo
cd my-new-repo
git init && git add -A && git commit -m "Initial commit from a1-addons-repo-template"
```

Or from a local checkout (useful during template development):

```bash
copier copy --trust /Users/visnu/Odoo/Onnet/SourceCode/a1-addons-repo-template ~/my-new-repo
```

Copier will prompt for each variable interactively. To skip prompts and
supply values on the CLI, use `--defaults` and `--data key=value`:

```bash
copier copy --trust --defaults \
    --data odoo_version=18.0 \
    --data odoo_edition=enterprise \
    --data repo_slug=server-tools \
    --data repo_name="A1 Server Tools" \
    --data repo_description="Server-side utilities." \
    gh:a1/a1-addons-repo-template ~/server-tools
```

### 2. Update an existing repository

Whenever the template evolves, pull the changes in with three-way merge:

```bash
cd my-repo
copier update --trust
```

Answers from the previous run live in `.copier-answers.yml`. **Do not
delete that file** — it is what makes `copier update` work. Commit it.

---

## Configuration variables

Full source: [`copier.yml`](./copier.yml). Grouped here for easy reference.

### Odoo target

| Variable        | Type   | Default        | Notes                                                                      |
| --------------- | ------ | -------------- | -------------------------------------------------------------------------- |
| `odoo_version`  | float  | `18.0`         | One of `10.0` … `19.0`. Drives Python version, lint tooling defaults.      |
| `odoo_edition`  | string | `enterprise`   | `community` or `enterprise`. Enterprise CI clones `odoo/enterprise` too.   |

### Repository identity

| Variable            | Type   | Default                                       |
| ------------------- | ------ | --------------------------------------------- |
| `org_slug`          | string | `a1`                                          |
| `org_name`          | string | `A1`                                          |
| `repo_slug`         | string | *(required)* e.g. `server-tools`              |
| `repo_name`         | string | *(required)* human-readable                   |
| `repo_website`      | string | `https://github.com/{{org_slug}}/{{repo_slug}}` |
| `repo_description`  | string | *(required)* short paragraph                  |
| `default_license`   | choice | `OPL-1` (also: `LGPL-3`, `AGPL-3`, `Other proprietary`) |

### Git branch / CI triggers

`odoo_version` is **independent** from the branch name. Pick Odoo 18 and
still name the branch `prod`.

| Variable             | Type      | Default              | Notes                              |
| -------------------- | --------- | -------------------- | ---------------------------------- |
| `target_branch`      | string    | `{{ odoo_version }}` | Primary branch that triggers CI.   |
| `extra_ci_branches`  | list[str] | `[]`                 | Additional branches for CI.        |

Example — Odoo 17 on `main` + `staging`:

```bash
copier copy … \
    --data odoo_version=17.0 \
    --data target_branch=main \
    --data 'extra_ci_branches=["staging"]'
```

### Lint tooling

`ruff`, `pylint-odoo` and pre-commit are all opt-out toggles with
version-aware defaults.

| Variable                    | Type      | Default (behaviour)                                                   |
| --------------------------- | --------- | --------------------------------------------------------------------- |
| `use_pre_commit`            | bool      | `yes` — generates `.pre-commit-config.yaml`                           |
| `use_ruff`                  | bool      | `yes` for Odoo ≥ 17, `no` for older (falls back to black/isort/flake8) |
| `additional_ruff_rules`     | list[str] | `[]` — extra rule codes on top of the template defaults               |
| `use_pylint_odoo`           | bool      | `yes` — generates `.pylintrc` + wires pylint into pre-commit and CI   |
| `pylint_odoo_version`       | string    | auto: `9.3.4` (Odoo ≥17) / `8.0.19` (14-16) / `3.5.0` (12-13) / `1.6.2` (≤11) |

#### Customising ruff

The generated `.ruff.toml` starts with a sensible base
(`E, F, W, I, UP, B, C4, SIM`). Add project-specific rules at scaffold time:

```bash
copier copy … --data 'additional_ruff_rules=["N", "PL", "RUF"]'
```

or re-render with `copier update` after editing `.copier-answers.yml`:

```yaml
# .copier-answers.yml
additional_ruff_rules:
    - N        # pep8-naming
    - RUF      # ruff-specific
    - S        # bandit
```

Then run `copier update --trust`. If you need deeper changes (custom
`per-file-ignores`, `format` options, etc.), edit `.ruff.toml` directly
after generation — Copier’s three-way merge will preserve your edits on
subsequent `copier update` runs.

#### Customising pylint-odoo

The generated `.pylintrc` enables the `odoolint` category plus a curated
list of pylint core checks, and configures `[ODOOLINT]` with:

- `manifest-required-authors=<org_name>`
- `license-allowed=OPL-1,LGPL-3,AGPL-3,Other proprietary`
- `valid-odoo-versions=<odoo_version>`

Two ways to customise:

1. **Change the version pin** (e.g. to a hotfix release):
   ```bash
   copier copy … --data pylint_odoo_version=9.4.0
   # or edit .copier-answers.yml → copier update
   ```
2. **Edit `.pylintrc` after generation.** The file is under your control
   once rendered; Copier will not overwrite local edits (three-way merge).
   Common tweaks:
   - Silence a noisy check: append its code to `disable=` under
     `[MESSAGES CONTROL]`.
   - Ignore a directory: add to `ignore=` under `[MASTER]`.

Turn it off entirely:

```bash
copier copy … --data use_pylint_odoo=false
```

### CI knobs

| Variable                                | Type   | Default | Purpose                                                                            |
| --------------------------------------- | ------ | ------- | ---------------------------------------------------------------------------------- |
| `postgres_image`                        | string | `""`    | Override the default PG image (e.g. `pgvector/pgvector:pg16`).                     |
| `include_wkhtmltopdf`                   | bool   | `no`    | Install wkhtmltopdf on the runner (needed for PDF report tests).                   |
| `enable_checklog_odoo`                  | bool   | `yes` for Odoo ≥18 else `no` | Fail CI on Odoo `WARNING`/`ERROR` log lines.                       |
| `github_check_license`                  | bool   | `yes`   | CI checks each `__manifest__.py` license against the allow-list.                   |
| `github_enforce_dev_status_compatibility` | bool | `yes`   | CI checks `development_status` vs dependencies.                                    |
| `github_ci_extra_env`                   | dict   | `{}`    | Extra env vars injected into the test job. E.g. `{"MY_VAR": "1"}`.                 |
| `rebel_module_groups`                   | list   | `[]`    | Modules that must be tested in isolation. Group with commas: `["a,b", "c"]`.       |

Enterprise editions also need a repo secret **`ODOO_ENTERPRISE_TOKEN`**
(a GitHub PAT with `repo` scope on `odoo/enterprise`).

---

## What you get

Every generated repository ships with:

```
├── .editorconfig
├── .gitattributes
├── .gitignore
├── .pre-commit-config.yaml       # ruff (17+) OR black+isort+flake8 (≤16)
├── .ruff.toml                    # only when use_ruff
├── .flake8, .isort.cfg           # only when NOT use_ruff and Odoo < 17
├── .pylintrc                     # only when use_pylint_odoo
├── LICENSE                       # per default_license
├── README.md
├── .copier-answers.yml           # DO NOT delete — needed for `copier update`
└── .github/
    └── workflows/
        ├── pre-commit.yml
        └── test.yml
```

---

## Recipes

### Community, older Odoo, custom branch

```bash
copier copy --trust --defaults \
    --data odoo_version=15.0 \
    --data odoo_edition=community \
    --data target_branch=main \
    --data repo_slug=warehouse-utils \
    --data repo_name="A1 Warehouse Utils" \
    --data repo_description="Barcode & logistics helpers." \
    gh:a1/a1-addons-repo-template ~/warehouse-utils
```

### Enterprise, stricter lint, extra ruff rules

```bash
copier copy --trust --defaults \
    --data odoo_version=19.0 \
    --data odoo_edition=enterprise \
    --data 'additional_ruff_rules=["N", "RUF", "S"]' \
    --data enable_checklog_odoo=true \
    --data repo_slug=account-a1 \
    --data repo_name="A1 Account Extensions" \
    --data repo_description="Chart-of-accounts and reporting patches." \
    gh:a1/a1-addons-repo-template ~/account-a1
```

### Skip pylint-odoo, keep ruff only

```bash
copier copy --trust --defaults \
    --data use_pylint_odoo=false \
    --data repo_slug=... --data repo_name=... --data repo_description=... \
    gh:a1/a1-addons-repo-template ~/dest
```

### Running lints locally

```bash
pipx install pre-commit
pre-commit install                # git hook for staged files
pre-commit run --all-files        # full-repo lint pass
```

pylint-odoo alone:

```bash
pip install "pylint-odoo==$(grep pylint_odoo_version .copier-answers.yml | awk '{print $2}')"
pylint --rcfile=.pylintrc your_module/**/*.py
```

---

## Local development on the template itself

```bash
# Smoke-test rendering into a scratch dir
copier copy --trust --defaults \
    --data repo_slug=probe --data repo_name=Probe --data repo_description=probe \
    . /tmp/probe

# Try older Odoo + community
copier copy --trust --defaults \
    --data odoo_version=12.0 --data odoo_edition=community \
    --data repo_slug=probe --data repo_name=Probe --data repo_description=probe \
    . /tmp/probe-12
```

Iterate on `copier.yml` and `src/*.jinja`, then re-render. Files with
Jinja in the filename (e.g. `{% if use_pylint_odoo %}.pylintrc{% endif %}.jinja`)
disappear entirely when the condition is false — a convenient way to gate
whole files.

---

## License

MIT — see [`LICENSE`](./LICENSE). Repositories *generated by* this template
default to `OPL-1`; override per invocation with `--data default_license=...`.
