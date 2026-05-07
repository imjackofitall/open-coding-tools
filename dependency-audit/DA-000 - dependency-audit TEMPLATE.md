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

## Model routing

| Step | Model      | Reason                                                    |
| ---- | ---------- | --------------------------------------------------------- |
| 1    | **haiku**  | _{one line — pure mechanical work}_                       |
| 2    | **sonnet** | _{one line — bounded judgement, well-specified}_          |
| 3    | **opus**   | _{one line — design / risky / cross-cutting / synthesis}_ |

If a sonnet/haiku step surfaces a non-trivial decision, escalate to the main session rather than guess.

**Rubric:**
- **haiku** — pure mechanical (commands, greps, counts, single-line edits)
- **sonnet** — bounded judgement (apply a pattern, refactor following a recipe, fix lint with clear rules)
- **opus** — design / architecture / risky / cross-cutting / synthesis

## Out of scope

-
