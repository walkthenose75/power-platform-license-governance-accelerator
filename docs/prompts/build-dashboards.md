# Analytics Experience Review Prompt

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

## Purpose

Use this prompt to assess analytics requirements and recommend reuse-first reporting architecture. Do not generate dashboards, reports, queries, semantic models, or implementation artifacts.

## Required Review

- Inventory relevant CoE dashboards, reports, and measures before proposing new analytics.
- Prefer extending or surfacing existing CoE reporting.
- Limit new recommendations to licensing intelligence, optimization, dependency, and remediation gaps.
- Do not recreate CoE application, flow, maker, owner, environment, usage, or governance reporting.
- Require clear source, freshness, assumptions, uncertainty, and estimated-versus-realized savings labels.
- Require validated supported interfaces for data acquisition.

## Expected Output

Provide reporting architecture recommendations, metric definitions at a conceptual level, CoE reuse opportunities, and data-quality risks. Do not output a dashboard or deployable artifact.