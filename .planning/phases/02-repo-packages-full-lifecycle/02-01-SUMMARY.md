---
phase: 02-repo-packages-full-lifecycle
plan: 01
subsystem: infra
tags: [pgdg, rpm, dnf, postgresql, upstream, appstream]

# Dependency graph
requires:
  - phase: 01-abstraction-variable-layer
    provides: PGDG variable override files (_pgdg.yml) and set_vars conditional loading
provides:
  - tasks/upstream_repo.yml with PGDG repo RPM install, AppStream disable, unused repo disable
  - contrib package in all PGDG package lists
  - Conditional wiring in tasks/main.yml before package install
affects: [02-repo-packages-full-lifecycle]

# Tech tracking
tech-stack:
  added: [pgdg-redhat-repo RPM, dnf module disable, dnf config-manager]
  patterns: [conditional include_tasks with d(false) | bool, difference filter for repo disable loop]

key-files:
  created: [tasks/upstream_repo.yml]
  modified: [tasks/main.yml, vars/RedHat_8_pgdg.yml, vars/RedHat_9_pgdg.yml, vars/RedHat_10_pgdg.yml]

key-decisions:
  - "AppStream disable uses dnf module disable for RHEL 8/9, excludepkgs for RHEL 10"
  - "PGDG repo list hardcoded as pgdg13-18 for unused repo disable loop"

patterns-established:
  - "PGDG repo tasks isolated in separate file, conditionally included"
  - "difference filter pattern for disabling all-except-active repos"

requirements-completed: [REPO-02, REPO-03, REPO-04, REPO-05, REPO-06, REPO-07]

# Metrics
duration: 2min
completed: 2026-03-26
---

# Phase 02 Plan 01: PGDG Upstream Repo Setup Summary

**PGDG repo RPM install, AppStream module/excludepkgs disable, unused repo disable, and contrib package for RHEL 8/9/10**

## Performance

- **Duration:** 2 min
- **Started:** 2026-03-26T15:30:50Z
- **Completed:** 2026-03-26T15:32:53Z
- **Tasks:** 2
- **Files modified:** 5

## Accomplishments
- Created tasks/upstream_repo.yml with 4 PGDG repo setup tasks (install RPM, disable AppStream module on RHEL 8/9, exclude AppStream packages on RHEL 10, disable unused version repos)
- Added postgresql-contrib package to all three PGDG variable override files
- Wired upstream_repo.yml into tasks/main.yml with conditional guard before package install

## Task Commits

Each task was committed atomically:

1. **Task 1: Create upstream_repo.yml and add contrib to _pgdg files** - `09732c3` (feat)
2. **Task 2: Wire upstream_repo.yml into tasks/main.yml** - `9be7599` (feat)

## Files Created/Modified
- `tasks/upstream_repo.yml` - PGDG repo setup: RPM install, AppStream disable, unused repo disable
- `tasks/main.yml` - Added conditional include_tasks for upstream_repo.yml before package install
- `vars/RedHat_8_pgdg.yml` - Added contrib package to PGDG package list
- `vars/RedHat_9_pgdg.yml` - Added contrib package to PGDG package list
- `vars/RedHat_10_pgdg.yml` - Added contrib package to PGDG package list

## Decisions Made
- AppStream disable uses `dnf module disable` for RHEL 8/9 (module streams) and `dnf config-manager --setopt=appstream.excludepkgs` for RHEL 10 (no module streams)
- PGDG repo list hardcoded as pgdg13 through pgdg18 for the unused repo disable loop, using Jinja difference filter against active version
- Changed_when set to check for "Nothing to do" in stderr for AppStream module disable idempotency

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered
- ansible-lint exit code 1 due to pre-existing role-name metadata issue (galaxy_info in meta/main.yml), not related to our changes
- yamllint passes clean on all files

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness
- PGDG repo setup complete, ready for Plan 02 (initdb and service lifecycle for upstream)
- The upstream_repo.yml tasks are conditional on postgresql_use_upstream_packages flag

---
*Phase: 02-repo-packages-full-lifecycle*
*Completed: 2026-03-26*
