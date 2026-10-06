# Application Experience Review Prompt

## Project Charter Alignment

This document must be interpreted in conjunction with `docs/project-charter.md`.

The goal is not to build a new governance platform.

The goal is to extend the customer's existing CoE investment with licensing intelligence, optimization analytics, dependency analysis, and safe remediation capabilities while minimizing duplication of existing CoE functionality.

All recommendations, designs, user stories, schemas, flows, dashboards, and implementation decisions in this document must comply with the principles defined in the project charter.

## Purpose

Use this prompt to assess user-experience needs and recommend CoE reuse. Do not generate an application, forms, views, commands, pages, or implementation artifacts.

## Required Review

- Determine whether existing CoE experiences can surface licensing intelligence and recommendations.
- Recommend a separate experience only for documented requirements that CoE cannot satisfy.
- Avoid recreating CoE inventory, administration, governance, or maker-management experiences.
- Ensure recommendations expose evidence, freshness, dependencies, assignment source, exceptions, and uncertainty.
- Keep remediation controls restricted to authorized administrators and separate from business approval workflows.
- Require dry-run visibility and clear partial-failure outcomes.

## Expected Output

Provide experience architecture recommendations, reuse options, accessibility and security risks, and any justified gaps. Do not output application configuration or deployable artifacts.