# Phase 3: Validation & Safety - Context

**Gathered:** 2026-03-26
**Status:** Ready for planning

<domain>
## Phase Boundary

Add fail-fast validation for invalid upstream configurations (version minimum, RHEL package conflicts, OSTree guard, unsupported platform guard) and verify regression protection for the existing Red Hat package workflow. Also add a PGDG-aware version-lock check and create test playbooks for both failure cases and happy-path upstream installation.

</domain>

<decisions>
## Implementation Decisions

### Conflict Detection
- **D-01:** Fail if ANY `postgresql*` RPM (without PGDG version-number prefix like `postgresql16`) is installed when upstream flag is set. Catches server, libs, contrib -- not just postgresql-server.
- **D-02:** Also detect cross-version PGDG conflicts: fail if a PGDG package from a different major version is installed (e.g., postgresql15-server when requesting version 16).
- **D-03:** Conflict checks only run when `postgresql_use_upstream_packages` is true. The RHEL workflow is not modified with new failure modes.

### Error Message Style
- **D-04:** All upstream validation errors use actionable guidance: state what went wrong AND how to fix it. E.g., "Upstream packages require PostgreSQL 16 or higher. Set postgresql_version to 16, 17, or 18."
- **D-05:** Conflict detection errors list the specific conflicting packages found in `ansible_facts.packages`. E.g., "Found RHEL postgresql packages [postgresql-server-13.9] that conflict with upstream PGDG packages. Remove them first or set postgresql_use_upstream_packages: false."

### Validation Ordering
- **D-06:** All upstream guards (version minimum, RHEL conflict, cross-version PGDG conflict, OSTree guard, platform guard) run as a single early block before any action (repo setup, package install, etc.). Users see ALL problems at once rather than fixing one, re-running, and hitting the next.
- **D-07:** Create a separate task file `tasks/validate_upstream.yml` for all upstream validation checks. Include it conditionally in `tasks/main.yml` when upstream flag is set -- keeps main.yml clean.

### Version Lock for PGDG
- **D-08:** Add a parallel version-lock check for PGDG packages (looks for `postgresql{VER}-server` in `package_facts`). Keeps existing RHEL version-lock check untouched. Two separate checks, one per workflow.

### Platform Guard
- **D-09:** Fail with clear message if upstream flag is set on a non-RedHat platform (Fedora, SUSE). Message: "Upstream PGDG packages are only supported on RHEL 8/9/10 and derivatives."

### Test Strategy
- **D-10:** Create `tests/tests_upstream_validation.yml` that exercises each validation guard: bad version, RHEL conflict, OSTree, unsupported platform. Uses block/rescue pattern to assert expected failures (follows existing `tests_input_file.yml` rescue pattern).
- **D-11:** Include a happy-path test in the upstream test playbook: set `postgresql_use_upstream_packages: true` with version 16 on RHEL 9 and verify the service starts. First end-to-end upstream test.
- **D-12:** SAFE-05 regression verification: document that existing test playbooks (`tests_default`, `tests_versions`, `tests_config_files`, `tests_certificate`, `tests_input_file`) already exercise the RHEL workflow. Phase 3 verifies they pass without modification -- no new regression wrapper needed.

### Claude's Discretion
- Exact package name matching logic for conflict detection (regex vs list comparison)
- How to enumerate installed PGDG packages for cross-version check
- Task ordering within `validate_upstream.yml` (which guard runs first)
- How to mock OSTree and non-RHEL platforms in tests (may need conditional skips based on test environment)

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Phase 1 & 2 Artifacts
- `vars/main.yml` -- Base internal variables including `__postgresql_versions_el*` lists
- `vars/RedHat_9_pgdg.yml` -- Reference PGDG override file (version allowlists, package names)
- `tasks/set_vars.yml` -- PGDG cascade loading, OSTree detection, booted detection
- `tasks/main.yml` -- Existing version checks (lines 11-45), package_facts gather, upstream_repo include
- `defaults/main.yml` -- `postgresql_use_upstream_packages` variable

### Templates
- `templates/pg_hba.conf.j2` -- HBA template (SCRAM-SHA-256 conditional added in Phase 2)

### Existing Tests (regression reference)
- `tests/tests_default.yml` -- Default RHEL workflow test
- `tests/tests_versions.yml` -- Version handling tests
- `tests/tests_config_files.yml` -- Config file generation tests
- `tests/tests_certificate.yml` -- SSL cert tests
- `tests/tests_input_file.yml` -- SQL execution tests (has block/rescue pattern for expected failures)
- `tests/tasks/run_role_with_clear_facts.yml` -- Test helper for clean role invocation
- `tests/tasks/clean_instance.yml` -- Test cleanup helper

### Research
- `.planning/research/PITFALLS.md` -- Known gotchas with path/name differences
- `.planning/codebase/TESTING.md` -- Existing test patterns and conventions

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- `package_facts` module already gathered at line 8 of `tasks/main.yml` -- `ansible_facts.packages` available for conflict detection
- Existing version check pattern (lines 11-35) provides template for new upstream version validation
- `__postgresql_is_ostree` variable already set in `tasks/set_vars.yml` -- ready for OSTree guard
- `tests/tests_input_file.yml` block/rescue pattern for testing expected failures

### Established Patterns
- Version allowlists per-platform: `__postgresql_versions_el8`, `__postgresql_versions_el9`, `__postgresql_versions_el10`
- PGDG version allowlists in `_pgdg.yml` files already set to 16/17/18
- Conditional task inclusion: `include_tasks` with `when:` for feature-gated logic
- Test pattern: `tests_*.yml` prefix, `run_role_with_clear_facts.yml` for role invocation, block/always for cleanup

### Integration Points
- `tasks/main.yml`: New `include_tasks: validate_upstream.yml` after existing version checks, before `upstream_repo.yml`
- `tasks/validate_upstream.yml`: New file with all upstream guards
- `tests/tests_upstream_validation.yml`: New test playbook

</code_context>

<specifics>
## Specific Ideas

- Conflict detection should catch any `postgresql*` RPM (without version-number prefix) as RHEL packages, plus any `postgresql{OTHER_VER}-server` as cross-version PGDG conflicts
- Error messages must be actionable with specific package names listed
- All validations in one file (`validate_upstream.yml`) to fail fast with all problems visible
- Happy-path test targets RHEL 9 specifically as the primary platform
- Platform guard covers Fedora and SUSE explicitly in the error message

</specifics>

<deferred>
## Deferred Ideas

None -- discussion stayed within phase scope

</deferred>

---

*Phase: 03-validation-safety*
*Context gathered: 2026-03-26*
