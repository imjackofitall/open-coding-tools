# Testing Pyramid Skill

A planning and audit workflow for deciding what to test, where to test it, and which tests are not pulling their weight.

## Files

| File | Purpose |
|---|---|
| `testing-pyramid-skill.md` | The agent skill prompt for planning or auditing test coverage. |
| `TP-000 - testing-pyramid TEMPLATE.md` | Starter template for test plans and audits. |
| `EXAMPLE - testing-pyramid.md` | Example testing pyramid artefact. |

## When To Use It

| Trigger | Output |
|---|---|
| "What tests should I write for this?" | A behaviour-by-behaviour test plan. |
| "Audit our tests" | Findings on shape, gaps, bloat, flakes, and concrete edits. |
| "Is this over-tested?" | Layer recommendations and duplicate coverage callouts. |
| "What layer should this assertion live in?" | A short unit, integration, or e2e recommendation. |

## Layer Rubric

| Layer | What it proves | Reach for it when | Speed |
|---|---|---|---|
| Unit | Pure functions, validators, reducers, small hook or component branching. | The behaviour fits one module and does not need real network or browser state. | Fastest |
| Integration | Route-handler logic, component renders, error states, accessibility branches. | The behaviour spans a few modules but does not need a real browser flow. | Medium |
| E2E | Cross-page flows with real browser, cookies, redirects, and data boundaries. | The value is in the full journey, not a leaf assertion. | Slowest |

## Output Modes

| Mode | Use it for | Output |
|---|---|---|
| Plan | New feature or planned change. | `TP-XXX - <scope> - <topic>.md` |
| Audit | Existing test suite. | `TP-XXX - <scope> - audit.md` |
| Layer pick | One assertion or behaviour. | Inline answer only. |

## Core Rules

- Test behaviours, not coverage percentages.
- Use the cheapest layer that proves the claim.
- Avoid e2e tests that would still pass with stubbed async results.
- Flag duplicate, setup-heavy, snapshot-heavy, or assertion-light tests as bloat.
- Produce a concrete edit list rather than writing test code during the planning loop.
