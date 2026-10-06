# Architecture Decisions

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

## ADR-001

Decision:

This solution will be designed as a CoE Toolkit extension.

Rationale:

The customer already has CoE deployed and intends to continue using it.

Impact:

Discover, surface, and reuse existing inventory rather than rebuilding or copying it. Any exception requires a documented CoE capability gap.

Status:

Accepted

---

## ADR-002

Decision:

Approval workflows will not be included.

Rationale:

The accelerator is not a business approval engine. Remediation safety is provided through authorization, explicit administrator initiation, dependency and exception checks, dry-run evidence, and auditing.

Impact:

No approval workflow components are designed or introduced. A future change requires a charter-level scope decision rather than an implicit phase expansion.

Status:

Accepted

---

## ADR-003

Decision:

An existing CoE experience will be extended or surfaced where it can meet the requirement. A separate model-driven app may be considered only for documented licensing-specific gaps that cannot be met through CoE reuse.

Rationale:

The project charter requires reuse before creation of a new user experience.

Impact:

CoE discovery and extension feasibility are prerequisites. This decision does not authorize a parallel governance application.

Status:

Superseded

---

## ADR-004

Decision:

Candidate interfaces are not approved until their support status, permissions, licensing, throttling, data availability, and tenant compatibility are validated.

Rationale:

Naming Microsoft Graph, Power Platform APIs, or admin connectors does not guarantee that a specific operation is supported or available.

Impact:

Architecture and backlog items must record interface validation evidence. Undocumented endpoints and direct manipulation of CoE internals are prohibited.

Status:

Accepted

---

## ADR-005

Decision:

Customer-facing authentication must minimize credential creation. The handoff unmanaged solution should use **delegated, connection-reference-based authentication** — the Dataverse connector for CoE reads, and a delegated Microsoft Entra-protected connection for Graph licensing read and reclamation — so the importing administrator signs in rather than creating an app registration. Service principals / application permissions are **optional** (for unattended or pipeline scenarios) and are used for the project's own build, not required of the customer.

Rationale:

The deliverable is an unmanaged solution a customer imports; requiring them to create Entra app registrations and grant application consent adds friction and may be blocked by their permissions or policies.

Impact:

Primary customer authentication is delegated connections. The delegated Graph license-read (and reclamation) path's feasibility depends on the signed-in administrator's directory role and must be validated (ADR-004). Application-permission guidance remains documented as an optional alternative. The project's own build in a development environment may use a disposable service principal.

Status:

Accepted