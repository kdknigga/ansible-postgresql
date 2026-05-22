<!-- generated-by: gsd-doc-writer -->
# Getting Started

This guide walks you through the prerequisites and steps needed to use the
`linux-system-roles.postgresql` Ansible role for the first time.

## Prerequisites

Before using this role you need the following tools installed on your Ansible
control node:

- **Ansible** `>= 2.9` (ansible-core `>= 2.16` recommended for full feature
  support; tested up to ansible-core 2.20)
- **Python** `>= 3.12` (the `.python-version` file pins `home_ansible_py312`)
- **git** — to clone the role repository

The **managed node** (the host where PostgreSQL will be installed) must be one
of the supported platforms:

| Platform | Versions |
|---|---|
| RHEL / CentOS / AlmaLinux / Rocky | 8, 9, 10 |
| Fedora | all current releases |
| openSUSE Leap | 15.6+ |

The role installs PostgreSQL packages via the OS package manager (AppStream /
Zypper) or via the upstream PGDG repository. No manual package installation is
required on the managed node.

## Installation Steps

### 1. Clone the repository

```bash
git clone https://github.com/linux-system-roles/postgresql.git
cd postgresql
```

### 2. Install required Ansible collections

The role depends on three Ansible collections. Install them with:

```bash
ansible-galaxy collection install -vv -r meta/collection-requirements.yml
```

This installs:

- `ansible.posix`
- `fedora.linux_system_roles`
- `community.general >= 8.2.0` (required for `dnf_config_manager` support)

### 3. Create a minimal playbook

Create a file named `playbook.yml` in your working directory:

```yaml
---
- name: Deploy PostgreSQL
  hosts: all
  vars:
    postgresql_version: "16"
  roles:
    - linux-system-roles.postgresql
```

Replace `postgresql_version` with the version you want to install. Supported
versions are `10`, `12`, `13`, `15`, `16`, `17`, and `18` (availability depends on
your OS version — see `docs/CONFIGURATION.md` for the full compatibility
matrix).

### 4. Create an inventory file

Create a file named `inventory` listing your managed node(s):

```ini
[db_servers]
my-db-host.example.com
```

For a quick local test you can use `localhost`:

```ini
[db_servers]
localhost ansible_connection=local
```

## First Run

Run the playbook against your inventory:

```bash
ansible-playbook -i inventory playbook.yml
```

A successful run ends with a `PLAY RECAP` showing `failed=0`. PostgreSQL is
then installed, initialized, and started on the managed node. The `postgres`
OS user can connect via a UNIX socket without a password.

To verify the server is running on the managed node:

```bash
sudo -u postgres psql -c "SELECT version();"
```

## Common Setup Issues

### Wrong Ansible version

The role requires Ansible `>= 2.9`. If you see a syntax error about unsupported
keywords, upgrade Ansible:

```bash
pip install --upgrade ansible-core
```

### Missing collections

If the playbook fails with `ERROR! couldn't resolve module/action`, run the
collection install command again:

```bash
ansible-galaxy collection install -vv -r meta/collection-requirements.yml
```

### PostgreSQL version not available on the target OS

Not every PostgreSQL version is available on every OS release. If you get a
"no package found" error, check the supported version matrix in
`docs/CONFIGURATION.md` and adjust `postgresql_version` accordingly.

### Version cannot be changed after install

Once the PostgreSQL server is installed, you cannot change `postgresql_version`
to upgrade or downgrade. If you need a different version, provision a fresh
host and re-run the role.

### Container / buildah environments

When connecting via a Buildah connection, server tuning (`postgresql_server_tuning`)
defaults to `false` and features that require a live systemd runtime (such as
setting `postgresql_password` and running `postgresql_input_file`) are not
available.

## Next Steps

- **Configuration reference** — See `docs/CONFIGURATION.md` for the full list
  of role variables, including SSL setup, pg_hba.conf customization, and
  PostgreSQL extensions.
- **Architecture** — See `docs/ARCHITECTURE.md` for an overview of how the
  role is structured internally.
- **Examples** — The `examples/` directory contains ready-to-run playbooks for
  common scenarios (basic install, SSL with certificate role, running an SQL
  input file).
- **Contributing** — See `contributing.md` for guidelines on running tests
  locally and submitting pull requests.
