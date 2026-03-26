---
phase: 01-abstraction-variable-layer
plan: 02
subsystem: infra
tags: [ansible, postgresql, pgdg, upstream, variables]

requires:
  - phase: 01-abstraction-variable-layer/01
    provides: "__postgresql_service_name, __postgresql_bin_dir, __postgresql_setup_cmd_path variables"
provides:
  - "postgresql_use_upstream_packages feature flag in defaults/main.yml"
  - "PGDG variable override files for RHEL 8/9/10 with derivative symlinks"
  - "Conditional PGDG loading in set_vars.yml"
affects: [02-upstream-lifecycle]

tech-stack:
  added: []
  patterns: ["conditional variable cascade for upstream/downstream package switching"]

key-files:
  created:
    - vars/RedHat_8_pgdg.yml
    - vars/RedHat_9_pgdg.yml
    - vars/RedHat_10_pgdg.yml
  modified:
    - defaults/main.yml
    - tasks/set_vars.yml

key-decisions:
  - "PGDG version allowlists set to 16/17/18 across all RHEL versions"
  - "Single include_vars task with d(false) filter rather than loop for PGDG loading"

patterns-established:
  - "PGDG override pattern: _pgdg.yml suffix files loaded after platform vars when feature flag true"
  - "Derivative symlink pattern extended: AlmaLinux/CentOS/Rocky symlinks for _pgdg files"

requirements-completed: [REPO-01, PATH-01, PATH-02, PATH-03, PATH-04]

duration: 2min
completed: 2026-03-26
---

# Phase 01 Plan 02: PGDG Feature Flag and Upstream Variable Overrides Summary

**Feature flag postgresql_use_upstream_packages with PGDG override files for RHEL 8/9/10 and conditional cascade loading in set_vars.yml**

## Performance

- **Duration:** 2 min
- **Started:** 2026-03-26T23:51:49Z
- **Completed:** 2026-03-26T23:53:33Z
- **Tasks:** 2
- **Files modified:** 5 (+ 9 symlinks created)

## Accomplishments
- Added postgresql_use_upstream_packages feature flag defaulting to false in defaults/main.yml
- Created 3 PGDG variable override files with versioned paths, service names, and package names for RHEL 8/9/10
- Created 9 derivative symlinks (AlmaLinux, CentOS, Rocky x 8, 9, 10)
- Wired conditional second cascade pass in set_vars.yml that loads _pgdg files only when feature flag is true

## Task Commits

Each task was committed atomically:

1. **Task 1: Add feature flag and create PGDG variable files with symlinks** - `802de98` (feat)
2. **Task 2: Add second cascade pass in set_vars.yml for PGDG loading** - `cb92dd6` (feat)

## Files Created/Modified
- `defaults/main.yml` - Added postgresql_use_upstream_packages: false
- `vars/RedHat_8_pgdg.yml` - PGDG overrides for RHEL 8 (versioned paths, packages, service name)
- `vars/RedHat_9_pgdg.yml` - PGDG overrides for RHEL 9
- `vars/RedHat_10_pgdg.yml` - PGDG overrides for RHEL 10
- `vars/*_{8,9,10}_pgdg.yml` - 9 derivative symlinks for AlmaLinux, CentOS, Rocky
- `tasks/set_vars.yml` - Added PGDG include_vars task with d(false) safe conditional

## Decisions Made
- PGDG version allowlists set to ["16", "17", "18"] across all RHEL versions (current PGDG-supported versions)
- Used single include_vars task with `d(false) | bool` filter rather than a loop, since only one _pgdg file matches per platform

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered
None

## User Setup Required
None - no external service configuration required.

## Next Phase Readiness
- Abstraction variable layer fully complete (Plans 01 + 02)
- Feature flag + override files + cascade loading all in place
- Ready for Phase 02 (upstream lifecycle) to implement repo management and package installation
- No blockers

## Known Stubs
None - all variables have concrete values and the conditional loading mechanism is fully wired.

---
*Phase: 01-abstraction-variable-layer*
*Completed: 2026-03-26*
