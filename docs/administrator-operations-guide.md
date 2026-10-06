# Administrator — Setup and Operations Guide

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

---

## About This Guide

This is the **instruction manual** for an administrator who imports and operates the License Governance Accelerator. The accelerator is delivered as an **unmanaged solution** you import into your own environment, configure, operate, and may modify.

It is **documentation only** — importing and configuring real components is performed by you in your tenant. Interface and permission details are in `docs/interface-api-specification.md`; security in `docs/security-identity-design.md`; recommendation logic in `docs/recommendation-rules-specification.md`.

## What the Accelerator Does

- Reads **licensing entitlements** (premium assignments, SKUs, direct vs. group) from **Microsoft Graph**.
- Correlates them to your existing **CoE Core** inventory/ownership and **CoE Audit** usage.
- Surfaces potentially reclaimable premium licenses with **explainable Safe / Review / Blocked** recommendations.
- Lets an administrator **safely reclaim** eligible, directly-assigned licenses with a **dry run**, guardrails, and an **immutable audit** — never automatically, never via an approval workflow.

## Prerequisites (Before Import)

1. **CoE Core Components and Audit Components** deployed and syncing in your tenant (data is reasonably fresh).
2. A **Microsoft Entra app registration** (read) with admin consent: `Organization.Read.All`, `User.Read.All` (add `GroupMember.Read.All` for group-name resolution). See `docs/security-identity-design.md`.
3. A **CoE-environment application user** with a read-only role to read CoE tables.
4. An authoritative **protected-identity source** (service / break-glass / shared / critical accounts).
5. Appropriate admin rights to assign security roles and grant consent.

> The accelerator has **no managed dependency on CoE** (it correlates via canonical IDs, not hard lookups), so import does not fail if your CoE schema differs. CoE is a **runtime data prerequisite**, not a solution dependency.

## Import and Setup

1. **Import** the unmanaged solution into your target environment (for example, a development or governance environment).
2. **Set connection references** — the Microsoft Graph connection (via your read service principal) and the CoE read connection.
3. **Set environment variables** — usage window (N days), in-scope premium SKUs, cost basis for estimates, CoE environment reference, and freshness limits (see Configuration Reference).
4. **Assign security roles** — map people to the four roles (below).
5. **Populate the protected-identity list** — service, break-glass, shared, and critical accounts that must never be reclaimed.
6. **Run a first acquisition** and confirm licensing data and correlation appear, with freshness shown.

## Security Roles

| Role | Who | Can do |
| --- | --- | --- |
| Executive | Leadership / finance | View dashboards only |
| Analyst | Platform/BU admins | View candidates, evidence, dry run; **cannot execute** reclamation |
| Administrator | Authorized admins | Everything, including execute reclamation and manage exceptions |
| Auditor | Security / compliance | Read-only across all, including the audit trail |

## Operating the Accelerator

1. **Acquire licensing** — refresh premium assignments/SKUs from Graph; direct vs. group is captured.
2. **Review visibility** — see premium counts, direct vs. group split, and "no recent usage evidence."
3. **Review candidates** — open a candidate to read its **Safe / Review / Blocked** classification with evidence, dependencies, assignment source, exceptions, freshness, and the governing rule.
4. **Dry run** — preview exactly what a reclamation would change; nothing is removed.
5. **Reclaim safely** — an Administrator initiates a single reclamation; protected, group-assigned, and stale-evidence candidates are blocked; confirm; an audit entry is written.
6. **Bounded bulk** (if enabled) — select a bounded set; per-item dry run, exclusions, and audit apply.
7. **Read the audit** — every action is recorded (actor, evidence, action, outcome) and is immutable.
8. **Dashboards** — operational triage and executive value (estimated vs. realized), extending your CoE Power BI.

## Configuration Reference (Environment Variables)

| Setting | Purpose | Notes |
| --- | --- | --- |
| Usage window (N) | Inactivity look-back | Must be ≤ your CoE usage-history window |
| In-scope premium SKUs | Which SKUs are governed | From your tenant's SKUs |
| Cost basis | Estimate calculation | Estimates are directional, not guaranteed |
| Freshness limit | Max evidence age for Safe | Beyond this, evidence is treated as stale |
| CoE environment reference | Where to read CoE data | Read-only |

## Safety Guardrails (What It Will and Won't Do)

- **Never** removes a license automatically or on a schedule — every reclamation is administrator-initiated.
- **Never** reclaims protected identities or group-assigned licenses.
- **Never** classifies as Safe when evidence is missing or stale.
- **No approval workflow** — safety comes from authorization, dry run, dependency checks, and audit.
- **Never** modifies CoE-managed components.

## Troubleshooting

| Symptom | Likely cause | Action |
| --- | --- | --- |
| No licensing data | Consent missing or app registration misconfigured | Confirm admin consent and connection reference |
| Everything "Blocked — insufficient evidence" | Usage window > CoE retention, or sync stale | Align N to the window; check CoE sync health |
| Many unmatched users | Maker Entra object ID not populated in CoE | Validate CoE maker data (correlation anchor) |
| Group-assigned users not actionable | Expected | Group licenses are changed at the group, outside this tool |
| Throttling/slow refresh | Graph/Dataverse limits | Retry/backoff; run during off-peak; use incremental |

## Maintenance and Modification

- Refresh acquisition on a cadence that matches your decision needs; always check freshness before acting.
- Keep the **protected-identity list** current; rotate service-principal credentials per policy.
- Because this is an **unmanaged solution you own**, you may modify or extend it — keep your components in your own solution and avoid modifying CoE-managed assets.

## Assumptions

- You operate the solution in your tenant and are responsible for its configuration and use.
- CoE Core + Audit are present and syncing; Graph read access is consented.

## Risks

| Risk | Mitigation |
| --- | --- |
| Operating before validating safety | Run `docs/validation-safety-test-checklist.md` first |
| Acting on stale data | Freshness is disclosed; Safe is blocked on stale evidence |
| Over-privileged access | Assign least-privilege roles; separate analyst from administrator |

## Dependencies

- `docs/interface-api-specification.md`, `docs/security-identity-design.md`, `docs/recommendation-rules-specification.md`, `docs/validation-safety-test-checklist.md`, `docs/data-handling-privacy-note.md`.

## Acceptance Criteria

- An administrator can import the solution, configure connections/variables/roles, run acquisition, review explainable candidates, perform a dry run, safely reclaim a single eligible license, and read the audit — using only this guide.

## Constraints Honored

- Documentation only; no assets generated. The accelerator extends CoE, builds only the licensing layer, performs no automatic removal and no approval workflow, and does not modify CoE-managed components.
