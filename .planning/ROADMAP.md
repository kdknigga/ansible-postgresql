# Roadmap: Upstream PostgreSQL Packages Support

## Overview

This roadmap delivers PGDG upstream package support for the ansible-postgresql role in three phases. Phase 1 extracts hardcoded values into variables and creates the upstream variable layer with the feature flag -- a zero-risk refactor that establishes the abstraction all subsequent work depends on. Phase 2 wires up the complete upstream lifecycle: repo installation, package installation, initdb, service management, config layering, SSL, password, and SQL execution. Phase 3 hardens the implementation with version validation, conflict detection, OSTree guards, and regression testing to ensure the existing Red Hat workflow remains untouched.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [ ] **Phase 1: Abstraction & Variable Layer** - Extract hardcoded values into variables, create upstream variable files, add feature flag
- [ ] **Phase 2: Repo, Packages & Full Lifecycle** - Install PGDG repo and packages, wire up complete upstream lifecycle
- [ ] **Phase 3: Validation & Safety** - Version validation, conflict detection, OSTree guard, regression testing

## Phase Details

### Phase 1: Abstraction & Variable Layer
**Goal**: The role's task files operate on variables instead of hardcoded values for service name, binary paths, data directory, and setup command -- and upstream-specific variable files exist to override these when the feature flag is set
**Depends on**: Nothing (first phase)
**Requirements**: REPO-01, PATH-01, PATH-02, PATH-03, PATH-04
**Success Criteria** (what must be TRUE):
  1. Running the role with default settings (no upstream flag) produces identical behavior to the current codebase -- all existing tests pass unchanged
  2. `postgresql_use_upstream_packages` variable exists in defaults/main.yml with default value `false`
  3. Handler, service tasks, initdb tasks, and psql tasks reference `__postgresql_service_name`, `__postgresql_bin_dir`, `__postgresql_data_dir`, and `__postgresql_setup_cmd_path` variables instead of hardcoded values
  4. Upstream variable files (`vars/RedHat_{8,9,10}_pgdg.yml` with derivative symlinks) exist and contain correct PGDG paths, package names, and service names
  5. Setting the upstream flag on a RHEL system loads the upstream variable overrides via set_vars.yml
**Plans**: 2 plans

Plans:
- [x] 01-01-PLAN.md -- Extract hardcoded values into __postgresql_* variables with downstream defaults
- [ ] 01-02-PLAN.md -- Add feature flag, create PGDG variable files, wire conditional loading

### Phase 2: Repo, Packages & Full Lifecycle
**Goal**: Users can install and run upstream PostgreSQL from PGDG packages on RHEL 8/9/10 with full parity to the Red Hat package workflow -- repo setup, package installation, database initialization, service management, config files, SSL certificates, password management, and SQL execution all work end-to-end
**Depends on**: Phase 1
**Requirements**: REPO-02, REPO-03, REPO-04, REPO-05, REPO-06, REPO-07, PATH-05, PATH-06, PATH-07, PATH-08, SAFE-04
**Success Criteria** (what must be TRUE):
  1. Running the role with `postgresql_use_upstream_packages: true` and `postgresql_version: "16"` on a clean RHEL 9 system results in a running PostgreSQL 16 service installed from PGDG packages
  2. The PGDG repo RPM is installed, AppStream postgresql module is disabled on RHEL 8/9, and unused PGDG version repos are disabled
  3. Config file layering (postgresql.conf includes, pg_hba.conf, SSL certs) writes to the correct upstream data directory
  4. Password management (ALTER USER) and SQL file execution work using the upstream psql binary path
  5. SCRAM-SHA-256 is used as the default authentication method for upstream installs
**Plans**: TBD

Plans:
- [ ] 02-01: TBD
- [ ] 02-02: TBD
- [ ] 02-03: TBD

### Phase 3: Validation & Safety
**Goal**: The role fails fast with clear errors on invalid configurations and the existing Red Hat package workflow has verified regression protection
**Depends on**: Phase 2
**Requirements**: SAFE-01, SAFE-02, SAFE-03, SAFE-05
**Success Criteria** (what must be TRUE):
  1. Role fails with a clear error message when upstream flag is set with a PostgreSQL version below 16
  2. Role fails with a clear error message when upstream flag is set but RHEL-shipped PostgreSQL packages are already installed on the system
  3. Role fails with a clear error message when upstream flag is set on an OSTree/bootc system
  4. All existing integration tests pass without modification when `postgresql_use_upstream_packages` is false (no regressions)
**Plans**: TBD

Plans:
- [ ] 03-01: TBD

## Progress

**Execution Order:**
Phases execute in numeric order: 1 -> 2 -> 3

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Abstraction & Variable Layer | 0/2 | Not started | - |
| 2. Repo, Packages & Full Lifecycle | 0/3 | Not started | - |
| 3. Validation & Safety | 0/1 | Not started | - |
