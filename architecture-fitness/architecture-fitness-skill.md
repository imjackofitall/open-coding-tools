---
name: architecture-fitness
description: Plan and police architectural characteristics with fitness functions — perf budgets, bundle ceilings, layering rules, accessibility budgets, error-budget SLOs. Use when the user asks to "audit our architectural characteristics", "set a perf budget", "add an arch-fitness check", "are our layering rules enforced", or "what should we fail the build over". Outputs an AF-XXX markdown spec; only modifies enforcement config (lint rules, CI scripts, budget files) with explicit go-ahead.
metadata:
  argument-hint: <app path, scope, or "audit" to review existing characteristics>
---

# Architecture Fitness

## Harness role

- **Control type:** guide (feedforward) + sensor (feedback)
- **Regulation category:** architecture fitness
- **Lifecycle stage:** pre-implementation + continuous
- **Computational checks:** the fitness functions this skill plans — perf budgets (Lighthouse, custom probes), bundle ceilings (`size-limit`), layering rules (ESLint `no-restricted-imports`, dependency-cruiser), accessibility budgets (axe-core), error-budget SLOs (Prom alerts). Each one runs pre-commit, in CI, or continuously
- **Inferential checks:** characteristic selection, threshold-setting, ownership, and enforcement-state judgement (enforced / gap / accepted gap / out of scope)
- **Frame:** [*Harness Engineering*](https://martinfowler.com/articles/harness-engineering.html) by Birgitta Böckeler

This skill names a project's architectural characteristics and turns each one into a **fitness function** — a deterministic, automated check that fails the build when the characteristic regresses. It produces a single Markdown spec at `.plans/architecture-fitness/AF-XXX - <scope> - <characteristic-or-audit>.md` (numbered sequentially — find the highest existing `AF-NNN` and increment, start at AF-001 if the folder is empty).

## When to use

Trigger this skill when the user asks any of:

- "Audit our architectural characteristics" / "what arch checks do we have?"
- "Set a perf budget" / "set a bundle-size ceiling"
- "Add a layering rule" / "are our boundaries enforced?"
- "Add an accessibility budget" / "fail the build on a11y regressions"
- "Define an error-budget / SLO check"
- "What should we fail the build over?"

Do NOT trigger this skill when:

- The user wants to fix a single perf bug — that's an FD or a direct task.
- The user wants a code review — that's `principal-code-review`.
- The user wants a test plan — that's `testing-pyramid`.

## Modes

### Mode A — Plan: design fitness functions for a named scope
Input: an app or package path, plus optionally a list of characteristics to start with. Output: a fitness spec covering each characteristic, with thresholds, fitness functions, and where each one runs.

### Mode B — Audit: review what's already enforced
Input: a scope and (optionally) a list of suspected characteristics. Output: the same spec shape, but with each row tagged by enforcement state — `enforced` (with the function cited), `documented but not enforced` (a gap), or `missing` (no awareness yet).

## The fitness spec

Every spec has these sections, in order:

```markdown
# AF-XXX — <scope> — <topic or "characteristics audit">

AF-XXX
Scope: <app path or "root">
Date: <YYYY-MM-DD>
Status: <open | in progress | live>

## Overview

One paragraph: what this scope is, what kind of system it is (UI / API / batch / data), and which characteristics matter
most for it.

## Characteristics

| ID  | Characteristic   | Why it matters                         | Threshold                  | Fitness function                                 | Where it runs                   | Enforcement state          | Owner |
|-----|------------------|----------------------------------------|----------------------------|--------------------------------------------------|---------------------------------|----------------------------|-------|
| AF1 | LCP on /booking  | Conversion drops past 2.5s             | LCP ≤ 2.5s p75 (mobile)    | Lighthouse CI assertion in `lighthouserc.json`   | CI (per PR) + nightly synthetic | enforced ✓ / gap / missing | Web   |
| AF2 | Route JS bundle  | First-load cost on mobile              | ≤ 180KB gzipped per route  | `next build` + size-limit config                 | pre-commit + CI                 | …                          | Web   |
| AF3 | Layering         | UI must not import server-only modules | zero violations            | ESLint `no-restricted-imports` boundaries config | pre-commit + CI                 | …                          | Web   |
| AF4 | a11y on /booking | Legal + ethical floor                  | axe: 0 serious, 0 critical | Playwright + axe-core in e2e suite               | CI                              | …                          | Web   |
| AF5 | API error budget | Reliability SLO                        | ≤ 0.1% 5xx over 7d rolling | Prom alert on the SLO recording rule             | continuous (Prometheus)         | …                          | API   |

## Open questions

{checkbox blocks using the in-file question format below — only live, unresolved questions}

## Decisions

| ✅   | Question | Decision | Why |
|-----|----------|----------|-----|

## Out of scope

{bullets — characteristics deliberately not addressed this round, with one-line reason}
```

## Picking characteristics

Use these as starting prompts, not a checklist gate. Add the ones the scope's risk profile actually needs.

- **Performance** — LCP, INP, TBT on critical routes; API p95 / p99 latency; cold-start time for serverless.
- **Bundle / payload** — per-route JS budget, total CSS budget, image-optimisation budget.
- **Layering / dependency direction** — UI must not import server-only modules; domain must not import infrastructure; no module reaches across feature folders. Enforced via ESLint `no-restricted-imports`, dependency-cruiser, or `eslint-plugin-boundaries`.
- **Accessibility** — axe-core thresholds on critical views; keyboard-nav e2e; colour-contrast budget.
- **Security** — secrets-scanning, dependency-audit cadence (delegate the actual audit to the `dependency-audit` skill), SAST findings ceiling.
- **Reliability** — error-budget SLOs (5xx rate, latency p99); synthetic uptime; flaky-test ceiling.
- **Data integrity** — schema-migration safety check; FK-orphan check; backup-restore drill cadence.
- **Cost** — peak monthly cloud spend per service; query cost ceiling for hot endpoints.
- **Build hygiene** — CI wall-clock budget; bundle-of-bundles drift; type-check time.

For each candidate: name it, name the threshold, name the fitness function (or admit there isn't one yet — that's a gap, not a fitness function).

## Fitness-function quality bar

A row in the table is only a fitness function if all of these are true:

1. **Deterministic.** Same input, same answer. "Looks fast" doesn't count; a number from Lighthouse does.
2. **Automated.** Runs without a human pressing go. A doc that says "remember to run X" is not a fitness function.
3. **Threshold is a number.** "Reasonable" is not a threshold. Pick a number; you can always change it.
4. **Failure has consequences.** Either it fails CI, or it pages someone, or it blocks a deployment. A check whose failure is ignored is theatre — call it that and remove it.
5. **Cheap to run where it lives.** Pre-commit fitness functions must finish in seconds. CI ones can take minutes. Continuous ones run on a schedule, not a PR.

If a candidate fails any of those, either reshape it until it passes or move it to "Out of scope" with a one-line reason.

## In-file question format (mandatory)

Use this exact shape for any judgement call — discrete options, one marked recommended, an Other slot, and a notes block. No free-prose questions.

```markdown
#### QX. {short question title}
{One or two sentences explaining the question and why it matters.}

- [ ] Option A — {short description}
- [ ] Option B — {short description} **(recommended — {one-line reason})**
- [ ] Option C — {short description}
- [ ] Other — notes:
  - _{your note here}_

**Notes / reasoning:**
- _{anything else worth recording about this pick}_
```

Cap one round at ~3 questions. Skip anything derivable from the codebase or existing config.

### Common architecture-fitness questions

1. **Threshold strictness.** Set the bar at current production p75 ("don't regress"), or at the target we want ("force improvement")? Default: don't regress; set a separate stretch target with a reminder to revisit.
2. **Failure mode.** Hard-fail CI, soft-warn-only for one sprint, or warn-then-fail after N days? Default: warn first, then fail after the team has had a sprint to clear existing violations.
3. **Where to run.** Pre-commit (cheap, friction), CI per PR (medium), or nightly only (cheapest, slowest signal)? Default: per PR for fast checks, nightly for synthetic / external probes.
4. **Owner.** Single team, rotating on-call, or a platform team? Default: the team whose code most often changes the relevant surface.

## Iteration loop

Once the spec file exists, run an in-file Q&A loop until sign-off. No enforcement-config edits beyond what the user explicitly approves.

### Beat 1 — Read current state
Read the spec end to end. Note ticked checkboxes, inline notes, and any direct edits the user made to the Characteristics table.

### Beat 2 — Process answers (fold FIRST, then move)
For each resolved question:
1. Fold the answer into the Characteristics row(s) it affects (threshold, fitness function, where it runs, enforcement state, owner).
2. Append the decision to `## Decisions`.
3. Remove the resolved block from `## Open questions`.

### Beat 3 — Keep progress live
Update the `Status:` header. Update each row's `Enforcement state` column as wiring lands (`gap` → `enforced ✓`).

### Beat 4 — Raise new questions
Cap at ~3 per round. New characteristics often surface new threshold questions; add them in-file.

### Beat 5 — Report back and wait
Send a terse chat message (≤120 words): what was folded, what new questions exist (by ID), and any unilateral pick (flagged so the user can override). Then stop.

## Implementation handoff (wiring the fitness functions)

When the user signals the spec is ready to enforce (phrases like "wire these up", "ship the checks", "enforce them", "make them fail the build"):

1. **Re-read the spec end-to-end** — thresholds and ownership may have moved since the last cycle.
2. **Pick the active scope** by walking `## Characteristics` top-to-bottom and selecting every row whose `Enforcement state` is `gap`. Skip rows already `enforced ✓` and rows marked `accepted gap` or `out of scope`.
3. **Mirror each chosen row into visible task tracking** using `TaskCreate` — one task per characteristic, titled with the AF ID and the fitness function (`AF4 — wire axe-core in tests/e2e/a11y.spec.ts`). Description carries the threshold.
4. **Update tasks live as work proceeds.** Mark each `in_progress` before starting, `completed` only when (a) the fitness function exists in config / CI / the alert system, (b) the threshold is encoded as a number, and (c) a deliberate failing run was observed at least once (proving it actually fails the build, not just runs). Do not batch.
5. **Mirror status back into the spec.** Flip the row's `Enforcement state` from `gap` to `enforced ✓` and cite the config path inline. Update `Status:` in the header (`open` → `in progress` → `live`).
6. **Default to warn-only first if violations exist today.** Land the function in non-blocking mode, get the existing violations to zero, then flip to fail. The task isn't `completed` until it actually fails on regression — explicitly track the flip date in the row's Notes.
7. **Block on red.** A check that "runs" but never fails is theatre. If you can't make it fail on a deliberate regression, leave the task `in_progress` and surface that in chat.
8. **Done means both surfaces agree.** Implementation is complete only when every chosen row is `enforced ✓` AND every task is `completed`.

## Sign-off gate

Don't declare the spec live until ALL of:

1. Every characteristic has a threshold (a number), a fitness function (one of: a config file, a CI step, an alert rule), and an owner.
2. Every characteristic's `Enforcement state` is one of `enforced ✓`, `accepted gap (deferred to YYYY-Q)`, or `out of scope (with reason)`. No silent gaps.
3. Every question in `## Open questions` is answered.
4. The user has explicitly confirmed ("ship it", "approved", "go ahead").

Until then, stay in the loop.

## Hard rules

- **Don't enable a check that hard-fails CI without explicit user approval.** Land the function in warn-only mode first if the team has existing violations.
- **Don't add fitness functions for characteristics nobody is willing to fail a build over.** That's documentation, not a control.
- **Cite real config paths.** A row that says "ESLint enforces this" without naming the rule and config file is a gap, not an enforcement.
- **A perf budget is a number, not "fast enough".** Same for bundle, a11y, and SLOs.
- **Don't double up with other skills.** Dependency-vulnerability cadence belongs to `dependency-audit`; layer-fit per-test belongs to `testing-pyramid`. This skill *names the characteristic* and points at where it's enforced — it doesn't re-implement those workflows.
- **Australian English in the report** (colour, organised, behaviour, prioritise).
- **Keep update messages terse.** The spec file is the durable artefact; chat updates are ≤120 words.
