# Customer Deployment Inventory (Template)

> **Do not commit tenant IDs, secrets, UPNs, connection strings, or sensitive configuration to this public repository.** Record real values in a private location (private fork, internal wiki, or secret store). This file is a **template** with placeholders; keep it generic here.

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

---

## Purpose

This is the **WP0 record** that replaces every "(validate)" assumption with a **confirmed CoE artifact** and resolves the blocking gaps. Complete it during WP0 validation (see `docs/core-and-audit-components-assessment.md` and `docs/audit-components-gap-analysis.md`). Legend: `[ ]` open · `[x]` confirmed · `[!]` deviates (note).

---

## 1. CoE Solutions and Versions

| Item | Expected | Confirmed value | Status |
| --- | --- | --- | --- |
| Core Components solution | `CenterofExcellenceCoreComponents` | `CenterofExcellenceCoreComponents` (managed) | `[x]` |
| Audit Components solution | `CenterOfExcellenceAuditComponents` | `CenterofExcellenceAuditComponents` — friendly name "Center of Excellence - Governance Components" (managed) | `[x]` |
| Core Components version | — | `4.50.9` | `[x]` |
| Audit Components version | — | `3.27.7` | `[x]` |
| Inventory mode | Cloud-flow inventory vs. Data Export | `<record>` | `[ ]` |
| Publisher prefix | `admin_` (validate) | `<record>` | `[ ]` |

## 2. Confirmed Core Tables and Key Columns

Replace expected names with confirmed logical names/columns. Mark PII.

| Capability | Expected table | Confirmed logical name | Key columns confirmed | Status |
| --- | --- | --- | --- | --- |
| Environment | `admin_environment` | `admin_environment` ✓ | confirmed | `[x]` |
| Power Apps App | `admin_app` | `admin_app` ✓ (`admin_applastlaunchedon`, `admin_launchesinthepast30days`, `admin_usespremiumapi`, `admin_appowner`) | confirmed | `[x]` |
| Flow | `admin_flow` | `admin_flow` ✓ (+ `admin_flowactiondetail`) | confirmed | `[x]` |
| Maker | `admin_makeruser` | `admin_maker` ✓ (also `admin_powerplatformuser`); UPN `admin_userprincipalname`, email `admin_useremail` | confirmed | `[x]` |
| Connector | `admin_connector` | `admin_connector` ✓ | confirmed | `[x]` |
| Connection Reference | `admin_connectionreference` | `admin_connectionreference` ✓ (+ `admin_connectionreferenceidentity`) | confirmed | `[x]` |

## 3. Confirmed Audit Tables and Usage Signals

| Signal | Expected source | Confirmed | Status |
| --- | --- | --- | --- |
| App launch events | Audit log table | `admin_auditlog` — **per-user** (`admin_userid`, `admin_userupn`), per-app (`admin_appid`, `admin_appname`), premium flag (`admin_appispremium`), timestamp (`admin_creationtime`) ✓ | `[x]` |
| Unique users | Aggregate on app | `admin_app.admin_appsharedusers`; per-user derivable from `admin_auditlog` | `[x]` |
| Last launched / last run | App/Flow columns | `admin_app.admin_applastlaunchedon`, `admin_launchesinthepast30days` ✓ | `[x]` |
| Flow usage | Flow usage data | `admin_flow` (+ `admin_flowactiondetail`); per-user flow usage weaker than apps (validate when populated) | `[~]` |

## 4. Blocking Gap Resolution

| Gap | Question | Resolution recorded | Status |
| --- | --- | --- | --- |
| **G1 per-user usage granularity** | Does CoE Audit retain per-user launch detail? | **Resolved (schema): `admin_auditlog` retains per-user, per-app, premium-flagged, timestamped launches → Reuse As-Is.** | `[x]` |
| **G2 usage-history window** | What is the retention/window length? | Pending data — `admin_auditlog` currently **empty (0 rows)** in Contoso - Dev; window TBD until CoE Audit sync populates. | `[!]` |
| G3 flow usage completeness | Is flow usage captured per user? | Flow usage weaker than app launches; revisit when populated. | `[~]` |
| Maker Entra object ID populated | Is the correlation anchor present? | No dedicated AAD object-id column; **correlate via UPN** (`admin_userprincipalname` / auditlog `admin_userupn` ↔ Graph `userPrincipalName`). `admin_makerid` is the Dataverse PK. | `[x]` |
| Business-criticality classification | Is CoE criticality populated? (Q-COE-3) | TBD — check when inventory populated. | `[ ]` |
| Last successful inventory/audit sync | Fresh? Any failed flows? | **CoE inventory empty** (0 apps/flows/environments, 1 maker, 0 auditlogs) — sync not yet populated in Contoso - Dev. | `[!]` |

## 5. Microsoft Graph and Consent

| Item | Value | Status |
| --- | --- | --- |
| Cloud / Graph base URL | `<commercial / sovereign>` | `[ ]` |
| `Organization.Read.All` consent | `<granted by / date>` | `[ ]` |
| `User.Read.All` consent | `<granted by / date>` | `[ ]` |
| `assignedByGroup` reliable (direct vs. group) | `<validated on sample>` | `[ ]` |
| Reclamation scope (`LicenseAssignment.ReadWrite.All`) | **Not provisioned until WP3** | `[ ]` |

## 6. Identities (names only — no secrets)

| Identity | Purpose | Scope | Status |
| --- | --- | --- | --- |
| SPN-Read-Graph | Licensing read | Organization.Read.All, User.Read.All | `[ ]` |
| SPN-Read-CoE | CoE Core+Audit read | Read-only role in CoE env | `[ ]` |
| Project app user | Write project tables | Project solution role | `[ ]` |
| SPN-Reclaim-Graph | Reclamation (WP3) | LicenseAssignment.ReadWrite.All | `[ ]` (WP3) |

## 7. Project Environment and Solution

| Item | Value | Status |
| --- | --- | --- |
| Dev environment | `Contoso - Dev` (target; build deferred) | `[ ]` |
| Project solution + publisher | `<name/prefix>` | `[ ]` |
| Environment variables / connection references | `<list>` | `[ ]` |
| Four Dataverse roles + FLS | Executive/Analyst/Administrator/Auditor | `[ ]` |

## WP0 Exit Gate

- `[ ]` Admin consent granted for read scopes; reclamation scope **not** provisioned.
- `[ ]` G1 and G2 resolved and recorded.
- `[ ]` Maker Entra object ID confirmed populated.
- `[ ]` CoE reads validated via SPN-Read-CoE; Graph reads validated via SPN-Read-Graph.
- `[ ]` All WP1-relevant "(validate)" names replaced with confirmed values.

When every gate item is `[x]`, WP0 is complete and WP1 may start.
