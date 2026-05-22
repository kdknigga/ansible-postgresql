<!-- generated-by: gsd-doc-writer -->
# Configuration Reference

This document describes all role variables available for the `linux-system-roles.postgresql` Ansible role.

## Environment Variables

This is an Ansible role and does not use OS-level environment variables. All configuration is passed as
Ansible variables in your playbook or inventory. See the sections below for the complete variable reference.

## Role Variables

### `postgresql_version`

The major version of the PostgreSQL server to install.

| OS Family | Supported Versions |
|---|---|
| RHEL/CentOS/AlmaLinux/Rocky 8 | `10`, `12`, `13`, `15`, `16` |
| RHEL/CentOS/AlmaLinux/Rocky 9 | `13`, `15`, `16`, `18` |
| RHEL/CentOS/AlmaLinux/Rocky 10 | `16`, `18` |
| Fedora | All versions above |

**Default:** `16` on RHEL 10, `13` on all other supported platforms.

```yaml
postgresql_version: "16"
```

Once PostgreSQL is installed, you cannot change this value to upgrade or downgrade the server. The role
will fail if the requested version does not match the installed version.

---

### `postgresql_password`

Password for the `postgres` database superuser. When unset, the database is accessible only from the
`postgres` system account via a UNIX socket.

**Default:** `null` (no password set)

It is strongly recommended to encrypt this value using Ansible Vault:

```yaml
postgresql_password: !vault |
      $ANSIBLE_VAULT;1.2;AES256;dev
      ....
```

Not available when running the role in container builds (buildah connection).

Once set, the same password must be supplied on every subsequent role run that accesses the database
(including `postgresql_input_file` operations). Changing this value does not change the existing password.

---

### `postgresql_server_conf`

Additional settings appended to `postgresql.conf`. These key/value pairs are written to
`/etc/postgresql/system-roles.conf`, which is included at the end of the main
`/var/lib/pgsql/data/postgresql.conf`. Because it is appended last, these settings override any earlier
defaults.

**Default:** undefined (no extra settings)

```yaml
postgresql_server_conf:
  ssl: on
  shared_buffers: 128MB
  huge_pages: try
```

Each key in the dict becomes a `key = value` line in the generated conf file. The file is regenerated
on every role run, so any manual edits are overwritten.

---

### `postgresql_pg_hba_conf`

Replacement content for `/var/lib/pgsql/data/pg_hba.conf`. When defined, the entire default
`pg_hba.conf` is replaced by the entries in this list.

**Default:** undefined (upstream default `pg_hba.conf` is preserved)

Each list item is a dict with these keys:

| Key | Required | Description |
|---|---|---|
| `type` | Yes | Connection type: `local`, `host`, `hostssl`, `hostnossl` |
| `database` | Yes | Target database name or `all` |
| `user` | Yes | Database user name or `all` |
| `auth_method` | Yes | Authentication method (e.g., `md5`, `peer`, `ident`, `scram-sha-256`) |
| `address` | No | IP address/CIDR (required for `host*` types) |
| `options` | No | Additional method-specific options |

```yaml
postgresql_pg_hba_conf:
  - type: local
    database: all
    user: all
    auth_method: peer
  - type: host
    database: all
    user: all
    address: '127.0.0.1/32'
    auth_method: md5
  - type: host
    database: all
    user: all
    address: '::1/128'
    auth_method: md5
```

This file is regenerated on every role run; any manual edits are overwritten.

---

### `postgresql_ssl_enable`

Enables SSL/TLS for the PostgreSQL server. When `true`, `ssl = on` is written to the internal role
conf file (`/etc/postgresql/system-roles-internal.conf`). A certificate and private key must also be
provided via `postgresql_cert_name` or `postgresql_certificates`.

**Default:** `false`

```yaml
postgresql_ssl_enable: true
```

---

### `postgresql_cert_name`

Absolute path prefix for an existing certificate and private key on the managed node. The role expects:
- `<path>.crt` — server certificate
- `<path>.key` — private key

**Default:** `null`

```yaml
postgresql_cert_name: /etc/certs/server
```

---

### `postgresql_certificates`

List of certificate definitions passed to the `fedora.linux_system_roles.certificate` role to
automatically generate certificates for the PostgreSQL server. Uses the same dict format as that role.

**Default:** `[]` (no certificates generated)

```yaml
postgresql_certificates:
  - name: postgresql_cert
    dns: ['localhost', 'www.example.com']
    ca: self-sign
```

---

### `postgresql_server_tuning`

When `true`, the role automatically tunes `shared_buffers` and `effective_cache_size` based on system
RAM:

- `shared_buffers` = total RAM / 4
- `effective_cache_size` = total RAM / 2

These values are written to `/etc/postgresql/system-roles-internal.conf`.

**Default:** `true`, except when the Ansible connection is `buildah` (container build), where it
defaults to `false` because build-time RAM does not reflect the deployment environment.

```yaml
postgresql_server_tuning: false
```

---

### `postgresql_input_file`

Path to a SQL script file on the Ansible controller. The role copies and executes this script against
the PostgreSQL instance after the server is started.

**Default:** undefined (no SQL script executed)

```yaml
postgresql_input_file: "/tmp/mypath/file.sql"
```

Not available when running the role in container builds (buildah connection).

---

### `postgresql_use_upstream_packages`

When `true`, installs PostgreSQL from the official PGDG upstream repository instead of the OS
Appstream packages.

**Default:** `false`

```yaml
postgresql_use_upstream_packages: true
```

Constraints when `true`:
- Only supported on RHEL 8/9/10 and derivatives (AlmaLinux, CentOS, Rocky). Not supported on Fedora
  or OSTree/bootc systems.
- Requires `postgresql_version: "16"` or higher.
- Existing RHEL-shipped `postgresql*` packages must be removed before enabling this option.
- Only one PGDG major version may be active at a time.

---

### `postgresql_extensions`

List of PostgreSQL extensions to install from PGDG packages. Requires
`postgresql_use_upstream_packages: true`.

**Default:** `[]`

Each list item is a dict:

| Key | Required | Description |
|---|---|---|
| `name` | Yes | Extension name (see supported extensions below) |
| `state` | No | `present` (default) or `absent` |
| `version` | No | PostGIS stream version prefix (e.g., `'33'`). Only valid for `postgis`. |
| `shared_preload` | No | Boolean override for `shared_preload_libraries` inclusion. The role manages this automatically for most extensions. |

**PGDG-packaged extensions** (require a separate package install):

| Name | Notes |
|---|---|
| `postgis` | Not available for PostgreSQL 13. Default stream is PostGIS 3.5. Use `version` to pin a specific stream (e.g., `'33'` → `postgis33_16`). Not available on RHEL 10 for PG 17 via PGDG. |
| `pgvector` | |
| `timescaledb` | Not available on RHEL 10. Does not support the `version` key. |
| `pg_cron` | |
| `pgaudit` | |

**Contrib-bundled extensions** (included in `postgresql{ver}-contrib`; no separate package installed):

| Name | Notes |
|---|---|
| `pg_stat_statements` | Requires preloading; managed automatically. |
| `auto_explain` | Requires preloading; managed automatically. |

**Extensions that require `shared_preload_libraries`:** `timescaledb`, `pg_stat_statements`, `pg_cron`,
`pgaudit`, `auto_explain`. The role adds these to `shared_preload_libraries` automatically. Use
`shared_preload: false` to suppress this behavior for a specific extension.

```yaml
postgresql_use_upstream_packages: true
postgresql_extensions:
  - name: postgis
  - name: pgvector
    state: present
  - name: timescaledb
  - name: postgis
    version: '33'
  - name: pg_stat_statements
```

## Generated Configuration Files

The role manages these configuration files on the target host:

| File | Template | Purpose |
|---|---|---|
| `/var/lib/pgsql/data/postgresql.conf` | (upstream default) | Main PostgreSQL config; role appends `include_if_exists` lines |
| `/etc/postgresql/system-roles-internal.conf` | `postgresql-internal.conf.j2` | Auto-tuning, SSL flag, `shared_preload_libraries` |
| `/etc/postgresql/system-roles.conf` | `postgresql.conf.j2` | User-supplied settings from `postgresql_server_conf` |
| `/var/lib/pgsql/data/pg_hba.conf` | `pg_hba.conf.j2` | Client authentication; only written when `postgresql_pg_hba_conf` is defined |

The internal conf file (`system-roles-internal.conf`) is always generated. The user conf file
(`system-roles.conf`) is only generated when `postgresql_server_conf` is defined. Both are linked
into `postgresql.conf` via `include_if_exists` directives.

## Per-Environment Overrides

This role does not use `.env` files. To apply different settings per environment, use standard Ansible
mechanisms:

- **Inventory group variables** — place variables in `group_vars/production/postgresql.yml` vs.
  `group_vars/staging/postgresql.yml`
- **Host variables** — override per host in `host_vars/<hostname>/postgresql.yml`
- **Ansible Vault** — encrypt sensitive values like `postgresql_password` per environment

## Idempotence Notes

- **Password:** Setting `postgresql_password` for the first time works. Changing it on a subsequent
  run does not update the database password. You must manage password changes manually via `psql`.
- **Config files:** `postgresql_server_conf` and `postgresql_pg_hba_conf` are fully regenerated on
  every run; manual edits are overwritten.
- **Version:** Once installed, `postgresql_version` cannot be changed via the role. Attempts to
  change the version will cause the role to fail.
- **SSL:** Reflects the state of the most recent role run.
- **Server tuning:** Reflects the state of the most recent role run.
