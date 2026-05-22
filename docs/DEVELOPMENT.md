<!-- generated-by: gsd-doc-writer -->
# Development

## Local setup

1. Fork and clone the repository:

   ```bash
   git clone https://github.com/linux-system-roles/postgresql.git
   cd postgresql
   ```

2. Install Python (the project pins `3.12`) and tox:

   ```bash
   pip install tox
   ```

3. Install `tox-lsr`, the Linux System Roles tox plugin:

   ```bash
   pip install --user "git+https://github.com/linux-system-roles/tox-lsr@main"
   ```

4. Download the tox-lsr configuration file:

   ```bash
   curl -o ~/.config/linux-system-roles.json \
     https://raw.githubusercontent.com/linux-system-roles/linux-system-roles.github.io/master/download/linux-system-roles.json
   ```

5. (RHEL/CentOS only) Enable EPEL and install the `standard-test-roles-inventory-qemu` package before running QEMU tests.

No `.env` file or compilation step is required — the role is pure YAML and Jinja2.

## Lint and check commands

This project uses `tox-lsr` to standardize all checks. The key tox environments are:

| Command | Description |
|---|---|
| `TOXENV=collection lsr_ci_runtox` | Convert the role to collection format (prerequisite for ansible-lint) |
| `tox -e ansible-lint` | Run ansible-lint with the `production` profile against the role |
| `tox -e yamllint` | Run yamllint across all YAML files (ignores `.tox/`) |
| `tox -e markdownlint` | Run markdownlint against all `*.md` files except `CHANGELOG.md` |
| `tox -e codespell` | Run codespell spell-checker (config in `.codespellrc`) |

Individual tools and their config file locations:

- **ansible-lint** — `.ansible-lint` (profile: `production`)
- **yamllint** — `.yamllint.yml`
- **markdownlint** — `.markdownlint.yaml`
- **codespell** — `.codespellrc` (skips `.pandoc_template.html5` and `.README.html`)

## Running integration tests locally

Integration tests use QEMU virtual machines managed by `tox-lsr`. Ensure the prerequisites in the Local setup section are met, then:

```bash
# Run a single test playbook against a specific image
tox -e qemu-ansible-core-2-20 -- --image-name fedora-43 tests/tests_default.yml

# Run against CentOS 9 with an older Ansible core
tox -e qemu-ansible-core-2-16 -- --image-name centos-9 tests/tests_default.yml
```

Container-based tests (no KVM required) follow the same pattern using `container-ansible-core-*` tox environments:

```bash
tox -e container-ansible-core-2-16 -- --image-name centos-9 tests/tests_default.yml
```

See [contributing.md](../contributing.md) and the [tox-lsr qemu docs](https://github.com/linux-system-roles/tox-lsr#qemu-testing) for the full range of options.

## Code style

- **ansible-lint** enforces the `production` profile, which includes FQCN module names, task naming, and standard Ansible idioms. Run it via `tox -e ansible-lint`.
- **yamllint** enforces YAML syntax consistency. Config is minimal (only ignores `.tox/`).
- **markdownlint** enforces Markdown style for all `*.md` files. Config is in `.markdownlint.yaml`.
- **codespell** checks spelling in source files. Words to ignore are listed in `.codespell_ignores`.

CI enforces all of these checks on every pull request and push to `main`.

## Branch conventions

No formal branch naming convention is documented in this repository. The default and integration branch is `main`.

Based on the commit lint configuration, branch names are not constrained, but commit message types (and therefore PR titles) must follow Conventional Commits:

Valid types: `build`, `chore`, `ci`, `docs`, `feat`, `fix`, `perf`, `refactor`, `revert`, `style`, `test`, `tests`

Examples:
- `feat: add support for TimescaleDB extension`
- `fix: correct PostGIS package naming for PostgreSQL 18`
- `docs: update CONFIGURATION.md with new variables`

## PR process

PR titles are linted against Conventional Commits format by the `PR Title Lint` CI workflow on every opened, synchronize, reopened, or edited event.

When submitting a pull request:

- Fill in the PR description template fields: **Enhancement**, **Reason**, **Result**, and any **Issue Tracker Tickets** (Jira or BZ).
- Ensure all CI checks pass — ansible-lint, yamllint, markdownlint, codespell, and integration tests all run automatically.
- Add or update test playbooks in `tests/` for any changed role behavior.
- The PR title must follow Conventional Commits format (`type: subject` in lowercase, no trailing period).
- Use `[citest_skip]` in the PR title to bypass integration tests when only making documentation or non-functional changes.

For general contribution guidelines, see [contributing.md](../contributing.md) and the upstream [Linux System Roles contribution guide](https://linux-system-roles.github.io/contribute.html).
