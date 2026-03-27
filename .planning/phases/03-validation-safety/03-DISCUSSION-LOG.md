# Phase 3: Validation & Safety - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md -- this log preserves the alternatives considered.

**Date:** 2026-03-26
**Phase:** 03-validation-safety
**Areas discussed:** Conflict detection, Error message style, Validation ordering, Test strategy, Version lock behavior, Regression test scope, Guard for non-RHEL distros

---

## Conflict detection

| Option | Description | Selected |
|--------|-------------|----------|
| postgresql-server RPM | Only fail if the RHEL-shipped postgresql-server package is installed | |
| Any postgresql RPM | Fail if ANY package named 'postgresql*' (without version-number prefix) is installed | ✓ |
| You decide | Claude picks the best approach | |

**User's choice:** Any postgresql RPM
**Notes:** More aggressive but prevents partial conflicts

| Option | Description | Selected |
|--------|-------------|----------|
| No, only RHEL packages | Existing version-lock check already catches mismatched installed versions | |
| Yes, cross-version PGDG too | Fail if any PGDG package from a different major version is installed | ✓ |
| You decide | Claude picks based on existing version-lock logic | |

**User's choice:** Yes, cross-version PGDG too
**Notes:** Prevents multi-instance confusion

| Option | Description | Selected |
|--------|-------------|----------|
| No, upstream-only | Only check for conflicts when upstream flag is true | ✓ |
| Yes, bidirectional | Also fail if PGDG packages detected when running default RHEL workflow | |

**User's choice:** No, upstream-only
**Notes:** Don't add new failure modes to the default RHEL path

---

## Error message style

| Option | Description | Selected |
|--------|-------------|----------|
| Actionable guidance | Include what went wrong AND how to fix it | ✓ |
| Match existing style | Keep messages terse like existing checks | |
| You decide | Claude picks a balanced style | |

**User's choice:** Actionable guidance

| Option | Description | Selected |
|--------|-------------|----------|
| Yes, list packages | Error lists specific conflicting packages found | ✓ |
| No, generic message | Generic conflict message without specifics | |

**User's choice:** Yes, list packages
**Notes:** Helps users debug

---

## Validation ordering

| Option | Description | Selected |
|--------|-------------|----------|
| Single early block | Group all upstream guards together, fail fast before any action | ✓ |
| Natural placement | Each guard fires at its logical point in the task flow | |
| You decide | Claude picks based on task flow | |

**User's choice:** Single early block
**Notes:** User sees ALL problems at once

| Option | Description | Selected |
|--------|-------------|----------|
| Separate file | Create tasks/validate_upstream.yml with all upstream guards | ✓ |
| Inline in main.yml | Add checks directly in main.yml | |

**User's choice:** Separate file
**Notes:** Keeps main.yml clean (already 155+ lines)

---

## Test strategy

| Option | Description | Selected |
|--------|-------------|----------|
| Yes, failure test playbook | Create tests/tests_upstream_validation.yml with block/rescue pattern | ✓ |
| No, manual verification only | Just verify existing tests pass | |
| You decide | Claude decides based on scope | |

**User's choice:** Yes, failure test playbook

| Option | Description | Selected |
|--------|-------------|----------|
| Yes, happy path too | Include test that sets upstream flag with version 16 and verifies service starts | ✓ |
| No, failures only | Phase 3 is about validation/safety only | |
| You decide | Claude decides based on what's practical | |

**User's choice:** Yes, happy path too

| Option | Description | Selected |
|--------|-------------|----------|
| RHEL 9 | Most common deployment target, has module streams | ✓ |
| RHEL 10 | Latest platform, no module streams | |
| All three (8, 9, 10) | Comprehensive but larger test matrix | |

**User's choice:** RHEL 9

---

## Version lock behavior

| Option | Description | Selected |
|--------|-------------|----------|
| Add parallel PGDG check | New check for postgresql{VER}-server packages, keeps existing check untouched | ✓ |
| Unified check | Refactor existing version-lock to handle both naming conventions | |
| You decide | Claude picks safest approach | |

**User's choice:** Add parallel PGDG check
**Notes:** Two separate checks, one per workflow

---

## Regression test scope

| Option | Description | Selected |
|--------|-------------|----------|
| Document which tests cover it | Note existing tests as the regression suite, verify they pass unchanged | ✓ |
| Create a regression wrapper | Create tests/tests_regression.yml that explicitly runs key tests | |
| You decide | Claude picks pragmatic approach | |

**User's choice:** Document which tests cover it
**Notes:** Existing tests ARE the regression suite

---

## Guard for non-RHEL distros

| Option | Description | Selected |
|--------|-------------|----------|
| Explicit fail with message | Fail if upstream flag set on non-RedHat platform | ✓ |
| Silent no-op | Cascade finds no _pgdg.yml, continues with defaults | |
| You decide | Claude picks safest approach | |

**User's choice:** Explicit fail with message
**Notes:** Prevents silent misconfigurations

---

## Claude's Discretion

- Exact package name matching logic for conflict detection
- How to enumerate installed PGDG packages for cross-version check
- Task ordering within validate_upstream.yml
- How to mock OSTree and non-RHEL platforms in tests

## Deferred Ideas

None -- discussion stayed within phase scope
