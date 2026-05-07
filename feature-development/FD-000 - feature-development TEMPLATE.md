# FD-XXX — {scope} — {short title}

## Status
- [x] Open
- [ ] In-progress
- [ ] Partly implemented
- [ ] Done

## Reason

## Problem Description

## Potential Solution

1.
2.
3.

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

## Acceptance Criteria

_One scenario block per key behaviour. Populated during planning; referenced by the testing-pyramid and code-review skills._

#### Scenario: {scenario name}
- **Given** {precondition}
- **When** {action}
- **Then** {outcome}

## Files to Modify

-

## Computational gate

_The deterministic checks that must be green before this work is considered done. Keep quality left — the cheaper and earlier the better. Delete rows that don't apply; add rows for any project-specific budget the change touches._

| Check               | Where it runs            | Threshold / pass condition                  |
|---------------------|--------------------------|---------------------------------------------|
| Type check          | local + CI               | zero errors in touched packages             |
| Lint                | pre-commit + CI          | zero errors, zero new warnings              |
| Unit + integration  | CI (and local pre-push)  | all green; no `.skip` added                 |
| E2E (if affected)   | CI                       | affected specs green                        |
| Perf budget         | (fill in or delete)      | (e.g. LCP ≤ 2.5s on the touched route)      |
| Bundle ceiling      | (fill in or delete)      | (e.g. route bundle ≤ 180KB gzipped)         |
| Accessibility       | (fill in or delete)      | (e.g. axe: zero serious/critical on view X) |

## Verification

1.
