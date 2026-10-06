# Automation Architecture Review Prompt

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

## Purpose

Use this prompt to assess automation responsibilities and recommend safe boundaries. Do not generate cloud flows, desktop flows, connectors, expressions, or implementation artifacts.

## Required Review

- Identify existing CoE automation that can be reused, extended, or surfaced.
- Avoid reimplementing inventory synchronization, ownership discovery, environment discovery, or governance processes already supplied by CoE.
- Require validation of support status, permissions, throttling, retry behavior, and tenant availability for proposed operations.
- Do not introduce business approval workflows.
- Keep remediation explicitly administrator-initiated and require dry-run, dependency, exception, assignment-source, authorization, and audit controls.
- Ensure partial failures are visible and actionable.
- Exclude automatic or unattended license removal.

## Expected Output

Provide automation responsibility boundaries, reuse recommendations, safety controls, failure risks, and validation needs. Do not output flow definitions or deployable artifacts.