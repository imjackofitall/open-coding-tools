# Dependency audit — {scope}

## Header

- **ID:** DA-XXX
- **Scope:** {app path or `root`}
- **Date:** YYYY-MM-DD
- **Node engine:** {from `engines`, `.nvmrc`, or `.node-version`}
- **Status:** Phase 1 in progress

## Phase 1 — Security

### Fixed (non-breaking)

| Package | Advisory | Version delta | Notes |
|---------|----------|---------------|-------|
|         |          |               |       |

### Pending user approval (force / breaking)

#### {package-name}

- **Current → proposed:** `x.y.z` → `a.b.c`
- **Breaking changes:** {summarise from real changelog / release notes}
- **Affected files in this repo:** `path/one.ts`, `path/two.tsx`
- **Risk:** low / medium / high — {one-sentence reason}

### Unfixable / no upstream fix

| Package | Advisory | Mitigation or carry-over |
|---------|----------|--------------------------|
|         |          |                          |

## Phase 2 — Necessity

| Package | Classification                                                           | Rationale    | Suggested action                | Effort    |
|---------|--------------------------------------------------------------------------|--------------|---------------------------------|-----------|
|         | KEEP / REPLACE WITH NATIVE / REPLACE WITH LIGHTER / REMOVE / CONSOLIDATE | one sentence | concrete diff or migration step | S / M / L |

## Open questions

_Use the in-file question format from `dependency-audit-skill.md`. Cap at ~3 per round._

## Decisions

| ✅   | Question | Decision | Why |
|-----|----------|----------|-----|

## Prioritised action list

1.

## Out of scope

-
