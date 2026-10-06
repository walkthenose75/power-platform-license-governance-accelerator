# Validation and Safety Test Checklist

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

---

## Purpose

This is a **self-serve safety checklist**. Because the accelerator can **remove premium licenses from people**, run these checks and confirm every result **before trusting reclamation** in your environment. It is deliberately lightweight (a focused safety verification, not a full QA test plan).

It is **documentation only** — you execute these checks in your own environment.

## How to Use

- Run in a **non-production / safe environment** or with **disposable test identities** wherever a reclamation path is exercised.
- Always **dry-run first**; never execute reclamation against real users during validation.
- Record the outcome per row: `[ ]` not run · `[x]` pass · `[!]` fail (stop and investigate).

## Safety Test Scenarios

| # | Scenario | Steps | Expected result | Result |
| --- | --- | --- | --- | --- |
| S1 | Protected identity never reclaimed | Mark a test account as service/break-glass/shared/critical; run analysis | Classified **Blocked**; reclamation command disabled; reason shown | `[ ]` |
| S2 | Group-assigned never actionable | Use a user with a **group-assigned** premium license | **Blocked**; controlling group shown; cannot execute | `[ ]` |
| S3 | Missing/stale evidence never Safe | Use a user with no usage data / stale evidence | **Blocked — insufficient evidence**; not Safe | `[ ]` |
| S4 | Active usage excluded | Use a user with recent premium activity | Shown **Active**; not a reclamation candidate | `[ ]` |
| S5 | Dry-run correctness | Dry-run a Safe candidate | Shows the exact intended change; **nothing is removed** | `[ ]` |
| S6 | Execution separation | Sign in as **Analyst**; attempt reclamation | **Not permitted** (no execute privilege) | `[ ]` |
| S7 | Immutable audit | Execute a reclamation (disposable identity); try to edit/delete the audit entry | Entry written (actor/evidence/action/outcome); **cannot** be edited or deleted | `[ ]` |
| S8 | No automatic removal | Observe over a sync/refresh cycle without administrator action | **No license is removed** unattended | `[ ]` |
| S9 | No approval workflow | Inspect the reclamation path | No approval/sign-off step exists; safety is authorization + dry run + audit | `[ ]` |
| S10 | Direct vs. group correctness | Compare a known direct vs. known group assignment | Correctly distinguished (`assignedByGroup`) | `[ ]` |
| S11 | Unmatched records surfaced | Introduce a user not matchable to CoE | Shown as **unmatched**, not silently dropped | `[ ]` |
| S12 | Partial failure surfaced | Simulate a failed reclamation in a bounded set | Per-item failure is visible; batch does not report false success | `[ ]` |
| S13 | Estimate labeled | Review the opportunity figure | Shown as an **estimate** with assumptions and freshness disclosed | `[ ]` |

## Functional Sanity Checks

| # | Check | Expected | Result |
| --- | --- | --- | --- |
| F1 | Acquisition | Premium assignments and SKUs load with freshness shown | `[ ]` |
| F2 | Correlation | Licensed users match CoE makers via Entra object ID | `[ ]` |
| F3 | Visibility reconciles | Counts (total, direct/group, no-recent-usage) are internally consistent | `[ ]` |
| F4 | Explanation completeness | Each candidate shows evidence, dependencies, source, exceptions, freshness, rule | `[ ]` |

## Sign-Off

| Gate | Criteria | Signed off by | Date |
| --- | --- | --- | --- |
| Safety | S1–S13 all pass | `<security owner>` | `<date>` |
| Function | F1–F4 all pass | `<admin owner>` | `<date>` |

Reclamation should be enabled for real users **only after the Safety gate passes**.

## Assumptions

- A safe test environment or disposable identities are available for reclamation checks.
- Protected-identity and group-assignment test data can be arranged.

## Risks

| Risk | Mitigation |
| --- | --- |
| Validation run against real users | Use disposable identities; dry-run only during validation |
| Skipping validation before go-live | Make the Safety gate a hard prerequisite to enabling reclamation |

## Dependencies

- `docs/recommendation-rules-specification.md` (expected classifications), `docs/security-identity-design.md` (roles/immutability), `docs/administrator-operations-guide.md` (operation).

## Acceptance Criteria

- All S1–S13 safety scenarios pass and are signed off before reclamation is enabled for real users; F1–F4 confirm basic function.

## Constraints Honored

- Documentation only; no assets generated. Validates safety-first behavior: no automatic removal, no approval workflow, protected/group/stale never Safe, immutable audit, and execution separation.
