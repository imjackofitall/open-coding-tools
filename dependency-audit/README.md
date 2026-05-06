# Dependency Audit Skill

A two-phase audit workflow for an app's npm dependencies — security first (`npm audit`, fix path, breaking-change triage), then necessity (keep / replace-native / replace-lighter / remove / consolidate).

## Harness role

- **Control type:** sensor (feedback)
- **Regulation category:** maintainability
- **Lifecycle stage:** on cadence (and pre-release when a release is dependency-sensitive)
- See the [harness frame](../readme.md#the-harness-frame) for vocabulary. Computational checks (`npm audit`, lockfile diff, `npm view` size data) run first; the necessity / replacement triage is the inferential half.

## Files

| File                                          | Purpose                                                              |
|-----------------------------------------------|----------------------------------------------------------------------|
| `dependency-audit-skill.md`                   | The agent skill prompt for running the two-phase audit.              |
| `DA-000 - dependency-audit TEMPLATE.md`       | Starter template for an audit report.                                |
| `EXAMPLE - dependency-audit.md`               | Example output covering both phases for a small app.                 |

## When To Use It

| Trigger                                          | Output                                                              |
|--------------------------------------------------|---------------------------------------------------------------------|
| "Audit our dependencies"                         | A two-phase report; security first, necessity second.               |
| "Run `npm audit` and tell me what's actionable"  | Security findings grouped by severity with a fix path per advisory. |
| "What packages can we drop?" / "Is `<x>` still used?" | A grep-backed classification of every dep.                     |
| "Are we using anything we could replace with native Node?" | Native-replacement candidates with call sites and migration cost. |

## Audit Shape

| Section                          | Purpose                                                                          |
|----------------------------------|----------------------------------------------------------------------------------|
| Header                           | Scope, date, Node engine, status.                                                |
| Phase 1 — Security               | Fixed (non-breaking), pending user approval (force / breaking), unfixable.       |
| Phase 2 — Necessity              | One row per dep, classified KEEP / REPLACE WITH NATIVE / REPLACE WITH LIGHTER / REMOVE / CONSOLIDATE. |
| Open questions / Decisions       | In-file Q&A loop; resolved decisions move to a compact table.                    |
| Prioritised action list          | Highest impact-to-effort first.                                                  |
| Out of scope                     | Anything deliberately not audited this round.                                    |

## Core Rules

- Security audit runs to completion before necessity classification starts.
- `npm audit fix --force` is per-package, never bulk.
- A package is only `REMOVE` after greps across source, configs, scripts, and CI prove it's unused.
- Native-replacement recommendations require verifying the project's Node engine.
- The report file is the durable artefact; chat updates between cycles stay terse (≤120 words).
