# Interface and API Specification

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

---

## Purpose and Scope

This specification defines the **supported interfaces** the accelerator uses, with enough precision to be **implementation-ready** for **WP0** (validation) and **WP1** (read-only Premium License Visibility). It also specifies the **reclamation** operation as forward-looking (WP3), gated and not provisioned during WP0/WP1.

It is **architecture documentation only** — no Power Platform assets, Dataverse schemas, or flows are generated. Every interface is subject to ADR-004 validation before adoption. CoE artifact names remain "(validate)" per `docs/core-and-audit-components-assessment.md`.

Interfaces in scope:

- **A — Microsoft Graph (licensing read)** — acquire premium assignments and SKUs (C1). *WP0 validate, WP1 use.*
- **B — Dataverse (CoE read adapter)** — read CoE Core + Audit tables for correlation/usage (C2/C3). *WP0 validate, WP1 use.*
- **C — Microsoft Graph (license reclamation)** — remove a direct license assignment (C6). *WP0 validate only; used in WP3.*

---

## Explicit Assumptions

- The tenant is a commercial Microsoft 365 cloud; Graph `v1.0` endpoints are available (confirm sovereign-cloud base URLs if applicable).
- Application (unattended) permissions are approvable by a tenant admin via admin consent.
- CoE Core + Audit are deployed; CoE data is readable via a Dataverse application user with a read-only security role.
- `licenseAssignmentStates` is available on users and reliably indicates group vs. direct assignment.
- The project runs in a **separate project-owned environment**; the CoE environment is read **cross-environment** via a service principal.

---

## Interface Inventory

| ID | Interface | Purpose | Auth | Least-privilege permission | WP |
| --- | --- | --- | --- | --- | --- |
| A1 | Graph `GET /subscribedSkus` | Tenant SKUs → SKU Reference, premium identification | App (client credentials) | `Organization.Read.All` | WP1 |
| A2 | Graph `GET /users?$select=id,userPrincipalName,assignedLicenses,licenseAssignmentStates` | Per-user assignments + direct/group source | App | `User.Read.All` | WP1 |
| A3 | Graph `GET /users/{id}/licenseDetails` (optional) | Service-plan detail per user | App | `User.Read.All` | WP1 |
| A4 | Graph `GET /groups/{id}` (optional) | Resolve group name for explainability | App | `GroupMember.Read.All` | WP2 |
| B1 | Dataverse Web API / FetchXML (CoE Maker) | Entra object ID ↔ user correlation anchor | App user (SPN) | Read-only role in CoE env | WP1 |
| B2 | Dataverse (CoE App/Flow/Environment/Connector) | Ownership, inventory, dependency context | App user (SPN) | Read-only role in CoE env | WP1/WP2 |
| B3 | Dataverse (CoE Audit usage/last-launched) | Usage evidence for inactivity | App user (SPN) | Read-only role in CoE env | WP1/WP2 |
| C1 | Graph `POST /users/{id}/assignLicenses` (removeLicenses) | Remove a **direct** license | App (separate identity) | `LicenseAssignment.ReadWrite.All` | WP3 |

---

## A — Microsoft Graph: Licensing Read (C1)

### A1 — SKU reference

- **Operation:** `GET https://graph.microsoft.com/v1.0/subscribedSkus`
- **Returns:** `skuId`, `skuPartNumber`, `servicePlans[]`, `prepaidUnits`, `consumedUnits`.
- **Use:** populate **SKU Reference**; flag in-scope **premium** SKUs; provide counts for opportunity sizing.
- **Permission:** `Organization.Read.All` (application).

### A2 — Per-user assignments and source

- **Operation:** `GET /users?$select=id,userPrincipalName,assignedLicenses,licenseAssignmentStates&$top=999` (page via `@odata.nextLink`).
- **Returns:** `assignedLicenses[].skuId`; `licenseAssignmentStates[]` with `skuId`, `state`, and **`assignedByGroup`** (GUID when group-assigned, null/empty when **direct**).
- **Direct vs. group (critical):** `assignedByGroup` empty → **Direct**; populated → **Group**. This is the authoritative signal and drives eligibility (only direct is reclaimable).
- **Use:** populate **License Assignment** (user object ID, SKU, assignment type, assignment source ref, observed as-of, correlation status).
- **Permission:** `User.Read.All` (application).
- **Incremental option:** `GET /users/delta` with `$select` to acquire only changes after the first full sync.

### A3 — License detail (optional)

- **Operation:** `GET /users/{id}/licenseDetails` → `servicePlans[]` per SKU (plan-level enabled/disabled). Use only if plan-level granularity is needed.

### Mapping to the data model

| Graph field | License Assignment / SKU Reference |
| --- | --- |
| `subscribedSkus.skuId` / `skuPartNumber` | SKU Reference.SKU Identifier / SKU Name |
| `user.id` | License Assignment.User Object ID (correlation anchor) |
| `userPrincipalName` | License Assignment.User UPN (PII, field-secured) |
| `assignedLicenses.skuId` | License Assignment.SKU (lookup) |
| `licenseAssignmentStates.assignedByGroup` | Assignment Type (Direct/Group) + Assignment Source Ref |
| request time | Observed As-of (freshness) |

### Cross-cutting (Graph read)

- **Throttling:** handle `429` with `Retry-After`; keep `$select` minimal; prefer delta; avoid unfiltered `$expand`.
- **Paging:** always follow `@odata.nextLink`.
- **Freshness:** stamp every acquisition with an as-of timestamp; never infer "no license" from a failed/partial page.

---

## B — Dataverse: CoE Read Adapter (C2/C3)

- **Pattern:** a **read-only adapter** issues Dataverse Web API (`GET` with `$select/$filter/$expand`) or FetchXML queries against CoE Core + Audit tables, keyed on **canonical IDs** (Entra object ID, environment/app/flow IDs). **No hard Dataverse lookups into CoE-managed tables**; names are "(validate)".
- **B1 Maker read:** retrieve maker records to resolve the **Entra object ID ↔ user** mapping (the correlation anchor). Blocking validation: the object ID must be populated (assessment area 4).
- **B2 Inventory/ownership read:** apps/flows/environments/connectors by owner for dependency context (C2/C4).
- **B3 Usage read:** app launches / unique users / last-launched (and flow usage where available) for inactivity evidence (C3); respect the usage-history window (G2); **missing usage ≠ zero usage**.
- **Auth:** a **service principal application user** in the **CoE environment** with a **least-privilege read-only security role** on the required CoE tables.
- **Throttling:** handle Dataverse service-protection limits (`429`, `Retry-After` / `x-ms-retry-after-ms`); page with `@odata.nextLink`; batch reads where appropriate.
- **Isolation:** reads only; never write to or modify CoE-managed components.

---

## C — Microsoft Graph: License Reclamation (C6 — WP3, validate in WP0)

- **Operation:** `POST /users/{id}/assignLicenses` with body `{ "addLicenses": [], "removeLicenses": ["{skuId}"] }`.
- **Direct-only:** removing a **group-assigned** license via this call **fails** — group assignments must be changed at the group (explicitly out of scope). Detect via `assignedByGroup` and **exclude** group-assigned from execution.
- **Permission:** `LicenseAssignment.ReadWrite.All` (granular, preferred) — on a **separate identity/connection** from the read path (execution separation). Not provisioned during WP0/WP1.
- **Dry run:** compute and present the exact intended `removeLicenses` change **without** calling the API; execution is a separate, confirmed, administrator-initiated step.
- **Idempotency/partial failure:** treat re-issue safely; capture per-user outcome; surface partial failures; write an immutable audit entry (C7).

---

## Dependencies

- Tenant admin to grant **admin consent** for application permissions (A1/A2; later C1).
- A **CoE-environment application user** with a read-only role (B1–B3).
- Resolution of **G1 (per-user usage granularity)** and **G2 (usage window)** from `docs/audit-components-gap-analysis.md` — if per-user usage is unavailable, B3 is supplemented or the inactivity definition adjusts.
- `docs/security-identity-design.md` for the identity, consent, and secret model.

## Risks

| Risk | Impact | Mitigation |
| --- | --- | --- |
| Admin consent delayed | Blocks A1/A2 (WP1) | Start consent in WP0; treat as a gate |
| Over-privileged app (write granted early) | Security exposure | Separate read vs. write identities; no write scopes in WP0/WP1 |
| `assignedByGroup` not reliably populated | Wrong eligibility | Validate on a sample; treat ambiguous as Review/Blocked |
| Graph/Dataverse throttling at scale | Slow/incomplete sync | Retry/backoff, delta, paging; disclose freshness |
| CoE schema/name drift | Adapter breakage | Confirm names during WP0; isolate in adapter |
| Sovereign cloud base URLs differ | Wrong endpoints | Confirm cloud and base URLs in WP0 |

## Acceptance Criteria (supports WP0/WP1)

- **WP0:** A1 and A2 return data in the target tenant under application permissions with admin consent; `assignedByGroup` correctly distinguishes direct vs. group on a validated sample; a CoE read via the SPN returns maker Entra object IDs; throttling/paging handled; C1 validated as supported but **not** consented for write.
- **WP1:** acquisition populates License Assignment + SKU Reference with direct/group distinction and freshness; CoE read correlates by Entra object ID; unmatched records are visible; no write/reclamation path is callable.

## Constraints Honored

- Documentation only; no assets generated. Only supported, validated interfaces (ADR-004). Read and write identities are separated; reclamation is direct-only, administrator-initiated, dry-run-first, and out of WP0/WP1 scope.
