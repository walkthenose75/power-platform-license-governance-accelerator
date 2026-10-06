# Schema Architecture Review Prompt

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

## Purpose

Use this prompt to review data architecture and recommend boundaries. Do not generate Dataverse tables, columns, relationships, or other implementation artifacts.

## Required Review

- Map required information to existing CoE data before recommending new storage.
- Identify only licensing-specific data that is demonstrably absent from CoE.
- Define authoritative ownership, retention, freshness, and reconciliation responsibilities at a conceptual level.
- Avoid copying CoE application, flow, maker, owner, environment, or usage inventory.
- Require supported, stable identifiers for correlation.
- Identify risks from CoE version differences, stale data, identity mismatch, and group-based licensing.
- Reject unsupported interfaces and direct manipulation of CoE internals.

## Expected Output

Provide a conceptual information-domain assessment, duplication findings, risks, and recommendations for greater CoE reuse. Do not output a physical schema or deployable artifact.