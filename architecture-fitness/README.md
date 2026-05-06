# Architecture Fitness Skill

A workflow for naming a project's architectural characteristics and turning each one into a **fitness function** — a deterministic, repeatable check that fails when the characteristic regresses.

## Harness role

- **Control type:** guide (feedforward) + sensor (feedback)
- **Regulation category:** architecture fitness
- **Lifecycle stage:** pre-implementation + continuous
- See the [harness frame](../readme.md#the-harness-frame) for vocabulary. The fitness functions this skill plans are computational checks (perf budgets, bundle ceilings, layering rules, accessibility budgets, error-budget SLOs) wired in pre-commit, in CI, or as continuous synthetic checks. Choosing *which* characteristic matters and *what* the threshold should be is the inferential half.

## Files

| File                                                  | Purpose                                                                |
|-------------------------------------------------------|------------------------------------------------------------------------|
| `architecture-fitness-skill.md`                       | The agent skill prompt for planning, naming, and policing fitness functions. |
| `AF-000 - architecture-fitness TEMPLATE.md`           | Starter template for a fitness-spec artefact.                          |
| `EXAMPLE - architecture-fitness.md`                   | Example output covering perf, bundle, layering, and a11y characteristics. |

## When To Use It

| Trigger                                                  | Output                                                                  |
|----------------------------------------------------------|-------------------------------------------------------------------------|
| "Audit our architectural characteristics"                | A fitness spec naming each characteristic, threshold, and check.        |
| "Set a perf budget for the booking app"                  | A fitness function (where it runs, what it fails on) for that budget.   |
| "Are our layering rules actually enforced?"              | A spec entry mapping the rule to an enforcement mechanism (or a gap).   |
| "Add an accessibility budget"                            | A fitness function for a11y on the named view(s).                       |

## Fitness Spec Shape

| Section                       | Purpose                                                                |
|-------------------------------|------------------------------------------------------------------------|
| Status                        | Tracks whether the spec is open, in progress, or live.                 |
| Scope                         | The app, package, or system the characteristics apply to.              |
| Characteristics               | One row per characteristic — why it matters, threshold, fitness function, where it runs, owner. |
| Open Questions                | Threshold and trade-off decisions still needing user input.            |
| Decision Trail                | Resolved decisions in a compact table.                                 |

## Core Rules

- A characteristic without a fitness function is a wish, not a control.
- Every fitness function names where it runs (pre-commit / CI / continuous) and what failure looks like.
- Thresholds are written as numbers, not adjectives.
- Don't add a characteristic the team isn't willing to fail a build over.
- The spec is ready only when every characteristic has a function, a threshold, and an owner.
