# Guardrails

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

## Architectural Principles

### Principle 1

CoE remains the primary source of truth for:

- Apps
- Flows
- Makers
- Owners
- Environments

Do not duplicate CoE inventory. Any proposed copy or new store requires a documented gap, data-ownership rationale, retention need, and reconciliation strategy.

---

### Principle 2

Build only missing capabilities:

- License inventory
- License analytics
- License optimization
- License reclamation

---

### Principle 3

No approval workflows.

The solution provides:

- Visibility
- Recommendations
- Bulk actions

It is not a business approval engine.

---

### Principle 4

Use only supported Microsoft APIs.

Preferred sources:

- Microsoft Graph
- Power Platform APIs
- Power Platform Admin Connectors
- Existing CoE Data

Listing a source does not approve it. Validate support status, permissions, licensing, throttling, data availability, and tenant compatibility before use. Undocumented endpoints and direct manipulation of CoE internals are prohibited.

---

### Principle 5

Safety first.

Never automatically remove licenses for:

- Service accounts
- Break-glass accounts
- Shared accounts
- Group-assigned licenses
- Critical app owners
- Critical flow owners

All remediation must be explicitly initiated by an authorized administrator, preceded by a dry run and dependency checks, and followed by an audit record. Missing or stale evidence must block automatic Safe classification.

---

### Principle 6

Every recommendation must be explainable.

An administrator should understand exactly why a user is a candidate.

The explanation must identify data sources, freshness, dependencies, assignment source, exceptions, uncertainty, and the rule that produced the recommendation.