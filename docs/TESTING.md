<!-- generated-by: gsd-doc-writer -->
# Testing

## Test Framework and Setup

This role uses **Ansible playbooks** for integration testing, orchestrated by
[tox-lsr](https://github.com/linux-system-roles/tox-lsr) — a tox plugin from the
Linux System Roles project that drives test execution against QEMU virtual machines
and containers.

Static analysis is handled by **ansible-lint** (configured via `.ansible-lint`) and
**ansible-test sanity** (for collection-format validation).

Before running tests locally, install the required dependencies:

```bash
pip install git+https://github.com/linux-system-roles/tox-lsr@3.17.1
```

Download the tox-lsr configuration file:

```bash
curl -o ~/.config/linux-system-roles.json \
  https://raw.githubusercontent.com/linux-system-roles/linux-system-roles.github.io/master/download/linux-system-roles.json
```

## Running Tests

### Single test playbook via QEMU

Run one test playbook against a specific platform:

```bash
tox -e qemu-ansible-core-2-20 -- --image-name centos-9 tests/tests_default.yml
```

### All tests via container

Run all `tests/tests_*.yml` playbooks sequentially inside a container:

```bash
tox -e container-ansible-core-2-17 -- --image-name centos-9 tests/tests_default.yml
```

### Ansible-lint

```bash
tox -e ansible-lint
```

### Molecule (local Docker-based)

A `molecule/default` scenario is included for basic local verification. It runs
`tests/tests_default.yml` against CentOS 6, 7, and 8 containers. The driver
defaults to Docker but can be overridden via the `LSR_MOLECULE_DRIVER` environment
variable.

```bash
molecule test
```

## Writing New Tests

All test playbooks live in the `tests/` directory and must follow the naming
convention `tests_*.yml`. Each playbook targets `hosts: all` and uses shared task
files from `tests/tasks/`.

Shared helpers:

| File | Purpose |
|------|---------|
| `tests/tasks/install_and_check.yml` | Install PostgreSQL, verify service is running, check shared_buffers and effective_cache_size tuning |
| `tests/tasks/clean_instance.yml` | Uninstall PostgreSQL and clean up after a test block |
| `tests/tasks/run_role_with_clear_facts.yml` | Re-run the role with cleared cached facts (used in multi-step tests) |
| `tests/tasks/check_header.yml` | Verify SPDX license header presence |

The standard pattern for a new test playbook is:

```yaml
# SPDX-License-Identifier: MIT
---
- name: Describe what this test covers
  hosts: all
  tasks:
    - name: Skip if platform is not supported
      meta: end_host
      when: ansible_facts['os_family'] != 'RedHat'

    - name: Test FEAT-01 - brief description
      block:
        - name: Run role with specific variables
          include_tasks: tasks/run_role_with_clear_facts.yml
          vars:
            __sr_public: true
            postgresql_use_upstream_packages: true
            postgresql_version: "16"

        - name: Assert expected state
          assert:
            that:
              - some_condition
            fail_msg: "FEAT-01 failed: ..."

      rescue:
        - name: Fail with context
          fail:
            msg: "FEAT-01 test failed: {{ ansible_failed_result.msg }}"

      always:
        - name: Clean up
          include_tasks: tasks/clean_instance.yml
          tags: tests::cleanup
```

The `ansible-lint` profile `production` is applied to all `tests/tests_*.yml`
files. The skip list in `.ansible-lint` documents intentional rule exceptions.

## Coverage Requirements

No automated code coverage thresholds are configured. Test coverage is ensured
through the breadth of integration test playbooks, which exercise:

- Default installation (`tests_default.yml`)
- Multiple PostgreSQL versions (`tests_versions.yml`)
- TLS certificate integration (`tests_certificate.yml`, `tests_custom_certificate.yml`)
- Configuration file management (`tests_config_files.yml`)
- Extension lifecycle — packages, preloading, repository setup, validation
  (`tests_extensions_*.yml`)
- Upstream (PGDG) package installation and validation guards (`tests_upstream_validation.yml`)
- OSTree/bootc image build validation (`tests_bootc_e2e.yml`)
- Input file variable loading (`tests_input_file.yml`, `tests_include_vars_from_parent.yml`)

## CI Integration

Four GitHub Actions workflows drive automated testing:

| Workflow file | Trigger | What it runs |
|---------------|---------|--------------|
| `.github/workflows/qemu-kvm-integration-tests.yml` | push/PR to `main`, merge group | All `tests/tests_*.yml` playbooks against QEMU and container images for CentOS 9/10, Fedora 42/43, openSUSE Leap 15.6 |
| `.github/workflows/ansible-lint.yml` | push/PR to `main`, merge group | `ansible-lint` against the collection-converted role |
| `.github/workflows/ansible-test.yml` | push/PR to `main`, merge group | `ansible-test sanity` with ansible-core 2.17 |
| `.github/workflows/weekly_ci.yml` | Weekly schedule (Saturday 18:00 UTC) + manual | Periodic full CI run via a draft PR |

The integration test matrix covers both QEMU-based VMs and container images, including
bootc variants. Bootc images undergo a two-stage test: the role builds the image
inside a container, then the resulting QCOW2 disk image is booted under QEMU for
end-to-end validation.

CI skips a test matrix entry when the role's `meta/main.yml` `galaxy_tags` does not
list the target platform. Add the relevant platform tag to `meta/main.yml` to enable
a new platform in CI.

To skip CI on a commit, include `[citest_skip]` in the PR title or commit message.
