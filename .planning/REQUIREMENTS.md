# Requirements: Upstream PostgreSQL Packages Support

**Defined:** 2026-03-26
**Core Value:** Users can install upstream PostgreSQL packages with full parity by setting a single flag

## v1 Requirements

Requirements for initial release. Each maps to roadmap phases.

### Repository & Package Management

- [x] **REPO-01**: Role accepts `postgresql_use_upstream_packages` boolean variable (default: false) that switches to upstream PGDG packages
- [ ] **REPO-02**: Role installs PGDG repo RPM (`pgdg-redhat-repo-latest.noarch.rpm`) when upstream flag is set
- [ ] **REPO-03**: Role disables AppStream postgresql module (`dnf module disable postgresql`) on RHEL 8/9 when upstream flag is set
- [ ] **REPO-04**: Role installs upstream packages using PGDG naming convention (`postgresql{VER}-server`, `postgresql{VER}`) when upstream flag is set
- [ ] **REPO-05**: Role installs contrib package (`postgresql{VER}-contrib`) when upstream flag is set
- [ ] **REPO-06**: Role disables unused PGDG version repos to avoid confusion when upstream flag is set
- [ ] **REPO-07**: Upstream packages work on RHEL 8, 9, 10 and derivatives (AlmaLinux, CentOS, Rocky)

### Path & Service Abstraction

- [x] **PATH-01**: Role uses upstream binary path (`/usr/pgsql-{VER}/bin/`) for psql, initdb, and setup commands when upstream flag is set
- [x] **PATH-02**: Role uses upstream data directory (`/var/lib/pgsql/{VER}/data/`) when upstream flag is set
- [x] **PATH-03**: Role uses upstream service name (`postgresql-{VER}`) for service enable/start/restart when upstream flag is set
- [x] **PATH-04**: Role uses upstream setup command path (`/usr/pgsql-{VER}/bin/postgresql-{VER}-setup`) for initdb when upstream flag is set
- [ ] **PATH-05**: Config file layering (postgresql.conf includes internal + user conf) works correctly with upstream data directory paths
- [ ] **PATH-06**: SSL/TLS certificate management works correctly with upstream paths
- [ ] **PATH-07**: Password management (ALTER USER, pg_hba md5 switch) works correctly with upstream psql binary path
- [ ] **PATH-08**: SQL file execution works correctly with upstream psql binary path

### Validation & Safety

- [ ] **SAFE-01**: Role validates that requested version is supported for upstream packages (minimum PG 16)
- [ ] **SAFE-02**: Role fails with clear error if RHEL postgresql packages are detected when upstream flag is set (conflict detection)
- [ ] **SAFE-03**: Role fails with clear error if `postgresql_use_upstream_packages` is set on OSTree/bootc systems
- [ ] **SAFE-04**: Role uses SCRAM-SHA-256 authentication instead of md5 when upstream flag is set (PG 16+ default)
- [ ] **SAFE-05**: Existing Red Hat package workflow is unchanged when upstream flag is false (no regressions)

## v2 Requirements

Deferred to future release. Tracked but not in current roadmap.

### Extensions

- **EXT-01**: Role accepts list of extension packages to install (`postgresql_extensions: [postgis, pgvector]`)
- **EXT-02**: Role maps extension names to PGDG RPM naming conventions

### Multi-Instance

- **MULTI-01**: Role supports managing multiple PostgreSQL versions side-by-side on one host
- **MULTI-02**: Role orchestrates pg_upgrade between major versions

### Platform Expansion

- **PLAT-01**: Upstream package support for Fedora
- **PLAT-02**: Upstream package support for SUSE

## Out of Scope

| Feature | Reason |
|---------|--------|
| Fedora PGDG packages | Keep first pass focused on RHEL family; different packaging quirks |
| SUSE PGDG packages | Different package ecosystem (zypper); separate packaging track |
| OSTree/bootc with PGDG | Untested interaction with rhel_rpm_ostree; explicitly blocked |
| Multiple PG instances | Requires rethinking single-instance architecture; separate milestone |
| pg_upgrade orchestration | Complex, data-destructive if wrong; separate playbook territory |
| Custom PGDG mirror URLs | Niche case; users can pre-configure repos manually |
| Non-free PGDG packages | Users can enable manually if needed |
| PostgreSQL < 16 from upstream | Older versions nearing EOL; minimum PG 16 per project scope |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| REPO-01 | Phase 1 | Complete |
| REPO-02 | Phase 2 | Pending |
| REPO-03 | Phase 2 | Pending |
| REPO-04 | Phase 2 | Pending |
| REPO-05 | Phase 2 | Pending |
| REPO-06 | Phase 2 | Pending |
| REPO-07 | Phase 2 | Pending |
| PATH-01 | Phase 1 | Complete |
| PATH-02 | Phase 1 | Complete |
| PATH-03 | Phase 1 | Complete |
| PATH-04 | Phase 1 | Complete |
| PATH-05 | Phase 2 | Pending |
| PATH-06 | Phase 2 | Pending |
| PATH-07 | Phase 2 | Pending |
| PATH-08 | Phase 2 | Pending |
| SAFE-01 | Phase 3 | Pending |
| SAFE-02 | Phase 3 | Pending |
| SAFE-03 | Phase 3 | Pending |
| SAFE-04 | Phase 2 | Pending |
| SAFE-05 | Phase 3 | Pending |

**Coverage:**
- v1 requirements: 20 total
- Mapped to phases: 20
- Unmapped: 0

---
*Requirements defined: 2026-03-26*
*Last updated: 2026-03-26 after roadmap creation*
