# Security and Identity Design

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

---

## Purpose and Scope

This document specifies the **security and identity design** for the accelerator, implementation-ready for **WP0** (identities, consent, roles) and **WP1** (read-only operation). Write/reclamation identity is specified but **not provisioned** until WP3 (least privilege by phase).

It is **architecture documentation only** — no Power Platform assets, Dataverse schemas, or flows are generated. It pairs with `docs/interface-api-specification.md` and `docs/data-model.md`.

---

## Explicit Assumptions

- Application (unattended) access is used for data acquisition; a tenant admin can grant **admin consent**.
- The project runs in a **separate project-owned environment and solution**; CoE is read cross-environment.
- Azure Key Vault (or equivalent supported secret store) is available for credentials.
- The four Dataverse roles from `docs/data-model.md` are the authorization model.

---

## Identity Architecture (Separation of Duties)

Three distinct identities enforce least privilege and execution separation:

| Identity | Purpose | Scope / role | Phase |
| --- | --- | --- | --- |
| **SPN-Read-Graph** | Acquire licensing (A1/A2/A3) | Graph app permissions: `Organization.Read.All`, `User.Read.All` (+ `GroupMember.Read.All` in WP2) | WP0/WP1 |
| **SPN-Read-CoE** | Read CoE Core + Audit (B1–B3) | Application user in the **CoE environment** with a **read-only** security role on required CoE tables | WP0/WP1 |
| **SPN-Reclaim-Graph** | Remove direct licenses (C1) | Graph `LicenseAssignment.ReadWrite.All` — **separate** app registration | **WP3 only** |

- **Read and write are different identities.** WP0/WP1 provision only the two read identities; the reclamation identity's write scope is **not consented** until WP3 and security sign-off.
- The project-environment application user that **writes project-owned tables** holds a least-privilege role scoped to the project solution only (no CoE write).

## Authentication and Secrets

- **Certificate-preferred** credentials for service principals (over client secrets); if secrets are used, short rotation.
- Store credentials in **Azure Key Vault**; reference via **environment variables / connection references** — never in source (already enforced; secret scan clean).
- Document a **rotation** schedule and owner; no secrets in logs or audit content.

## Admin Consent and Least-Privilege Justification

| Permission | Why needed | Phase | Approver |
| --- | --- | --- | --- |
| `Organization.Read.All` | Read tenant SKUs (A1) | WP0/WP1 | Tenant admin |
| `User.Read.All` | Read per-user assignments + source (A2/A3) | WP0/WP1 | Tenant admin |
| `GroupMember.Read.All` | Resolve group-assignment names (A4) | WP2 | Tenant admin |
| CoE read-only security role | Read CoE Core + Audit (B1–B3) | WP0/WP1 | CoE/env admin |
| `LicenseAssignment.ReadWrite.All` | Remove direct licenses (C1) | **WP3** | Security + tenant admin |

- Record consent in the **Customer Deployment Inventory**; no broader scopes (for example, no `Directory.ReadWrite.All`) are requested.

## Dataverse Security Model

From `docs/data-model.md` (organization-owned tables):

- **Four roles:** Executive (read dashboards), Analyst (read + review status; **no execute**), Administrator (full + execute reclamation + manage exceptions/reference), Auditor (read-only across all incl. audit).
- **Field-level security:** PII (UPN, display/identity names) and financial (cost/estimate) restricted per role.
- **Execution separation:** only **Administrator** can create a Reclamation Action; Analysts cannot execute.
- **Immutable audit:** **no role** has Update/Delete on Audit Log Entry (append-only).
- **Native Dataverse auditing** enabled on Reclamation Action, Protected Identity/Exception, License Assignment status, and Optimization Candidate classification.

### WP0/WP1 posture

- Provision all four roles in WP0, but WP1 is **read-only**: no reclamation privilege is exercised, and the reclamation identity/scope is absent until WP3.

## Data Protection and Privacy

- **PII minimization:** store **Entra object IDs**; snapshot UPN/display names only where needed for usability, protected by field-level security.
- **Retention** per `docs/data-model.md`; immutable audit retained per policy; operational data purged on a rolling window.
- **Policy preservation:** DLP, tenant, environment, and conditional-access policies are honored; no control is weakened to enable a feature.
- A lightweight **privacy/DPIA note** (identified as a missing doc) should cover user/licensing data handling before WP2/WP3.

## Dependencies

- Tenant admin (consent), CoE/env admin (CoE read role), security owner (reclamation sign-off before WP3).
- Key Vault (or equivalent) for secrets.
- `docs/interface-api-specification.md` (permissions per operation).

## Risks

| Risk | Impact | Mitigation |
| --- | --- | --- |
| Over-privileged or shared identity | Security exposure / breaks separation | Three scoped identities; no write in WP0/WP1 |
| Secret leakage | Credential compromise | Key Vault, certificates, rotation, no secrets in source/logs |
| Consent delays | Blocks acquisition | Start consent in WP0; gate owner |
| FLS/role gaps expose PII or allow unintended execution | Privacy/safety breach | Enforce FLS + execution separation; test in WP1 |
| Conditional access blocks SPN | Acquisition fails | Validate SPN access under CA in WP0 |

## Acceptance Criteria (supports WP0/WP1)

- **WP0:** SPN-Read-Graph and SPN-Read-CoE exist with only read scopes and admin consent recorded; the four Dataverse roles exist; FLS profiles defined; reclamation identity **not** provisioned; secrets in Key Vault; consent and identities recorded in the Customer Deployment Inventory.
- **WP1:** acquisition and CoE reads succeed under least-privilege read identities; Analyst role cannot create a Reclamation Action; PII fields are field-secured; no write path to Graph or CoE exists.

## Constraints Honored

- Documentation only; no assets generated. Least privilege and separation of duties; read/write identities separated; reclamation scope deferred to WP3; no CoE-managed component modified; no approval workflow and no automatic removal introduced.
