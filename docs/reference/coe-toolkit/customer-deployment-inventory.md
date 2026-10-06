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
| Core Components solution | `CenterofExcellenceCoreComponents` | `<record>` | `[ ]` |
| Audit Components solution | `CenterOfExcellenceAuditComponents` | `<record>` | `[ ]` |
| Core Components version | — | `<record>` | `[ ]` |
| Audit Components version | — | `<record>` | `[ ]` |
| Inventory mode | Cloud-flow inventory vs. Data Export | `<record>` | `[ ]` |
| Publisher prefix | `admin_` (validate) | `<record>` | `[ ]` |

## 2. Confirmed Core Tables and Key Columns

Replace expected names with confirmed logical names/columns. Mark PII.

| Capability | Expected table | Confirmed logical name | Key columns confirmed | Status |
| --- | --- | --- | --- | --- |
| Environment | `admin_environment` | `<record>` | id, type, region | `[ ]` |
| Power Apps App | `admin_app` | `<record>` | id, owner, environment, type, last launched | `[ ]` |
| Flow | `admin_flow` | `<record>` | id, owner, environment, state | `[ ]` |
| Maker | `admin_makeruser` | `<record>` | **Entra object ID**, UPN (PII) | `[ ]` |
| Connector | `admin_connector` | `<record>` | id, tier/premium | `[ ]` |
| Connection Reference | `admin_connectionreference` | `<record>` | connector, owner, environment | `[ ]` |

## 3. Confirmed Audit Tables and Usage Signals

| Signal | Expected source | Confirmed | Status |
| --- | --- | --- | --- |
| App launch events | Audit log table | `<record>` | `[ ]` |
| Unique users | Aggregate on app | `<record>` | `[ ]` |
| Last launched / last run | App/Flow columns | `<record>` | `[ ]` |
| Flow usage | Flow usage data | `<record>` | `[ ]` |

## 4. Blocking Gap Resolution

| Gap | Question | Resolution recorded | Status |
| --- | --- | --- | --- |
| **G1 per-user usage granularity** | Does CoE Audit retain per-user launch detail? | `<Reuse / Supplement + detail>` | `[ ]` |
| **G2 usage-history window** | What is the retention/window length? | `<N days>` | `[ ]` |
| G3 flow usage completeness | Is flow usage captured per user? | `<record>` | `[ ]` |
| Maker Entra object ID populated | Is the correlation anchor present? | `<yes/no + coverage>` | `[ ]` |
| Business-criticality classification | Is CoE criticality populated? (Q-COE-3) | `<reuse / project-owned>` | `[ ]` |
| Last successful inventory/audit sync | Fresh? Any failed flows? | `<record>` | `[ ]` |

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
