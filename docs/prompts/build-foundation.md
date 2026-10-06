# Foundation Architecture Review Prompt

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

## Purpose

Use this prompt only to assess foundation architecture and recommend a reuse-first approach. Do not generate Power Platform components or implementation artifacts.

## Required Review

- Inventory the relevant capabilities already present in the customer's CoE deployment.
- Identify licensing, analytics, dependency, and remediation gaps that CoE does not meet.
- Recommend how existing CoE data and experiences can be surfaced or extended.
- Reject parallel governance and inventory capabilities unless a documented gap justifies them.
- Identify security, ALM, versioning, data ownership, and operational risks.
- Require validation evidence for every proposed interface.
- Keep remediation administrator-initiated, explainable, dependency-aware, and auditable.
- Do not introduce business approval workflows.

## Expected Output

Provide architecture recommendations, assumptions, risks, decisions requiring validation, and CoE reuse opportunities. Do not produce schemas, flows, applications, dashboards, tables, or code.