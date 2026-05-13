# Open Coding Tools

Reusable coding workflows, agent skills, and planning templates for shipping software with less guesswork.

This repo is my personal toolbox for working with AI coding agents: structured prompts, repeatable planning artefacts,
review rubrics, and templates that help turn vague ideas into reviewable work. The goal is simple: make the agent do the
boring scaffolding, keep the important decisions visible, and leave a clear paper trail for future me.

## The Harness Frame

The vocabulary in this repo follows Birgitta Böckeler's article [*Harness
Engineering*](https://martinfowler.com/articles/harness-engineering.html) (martinfowler.com). In her framing, an agent
is `Model + Harness`, and the harness is everything around the model that makes its output trustworthy. This repo is a
personal slice of that outer harness — the part the user owns, not the part the agent vendor ships.

| Term                         | What it means here                                                                                                 |
|------------------------------|--------------------------------------------------------------------------------------------------------------------|
| Harness                      | Everything around the model that steers, checks, and corrects its work — these skills, templates, and rules.       |
| Guide (feedforward)          | Anticipatory control that steers the agent **before** it acts — PRDs, FDs, test plans, fitness specs.              |
| Sensor (feedback)            | Observational control that catches issues **after** the agent acts — code review, dependency audit.                |
| Computational check          | Deterministic and fast — tests, linters, type checkers, `npm audit`, perf budgets. Run these as early as possible. |
| Inferential check            | Probabilistic and semantic — review prose, layer-fit judgement, necessity triage. Slower, richer, fallible.        |
| Maintainability harness      | Regulates internal code quality. Most mature today.                                                                |
| Architecture fitness harness | Enforces architectural characteristics with fitness functions (perf budgets, layering rules, a11y budgets).        |
| Behaviour harness            | Guides functional correctness — what the product is supposed to do.                                                |

The point of using Böckeler's vocabulary is to stop pretending these artefacts are rails or scaffolding and call them
what they are: controls in a feedforward/feedback system around a probabilistic worker.

## What's Inside

| Area                                                   | Use it when                                                                    | Main files                                                                                                              |
|--------------------------------------------------------|--------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------|
| [Architecture Fitness](architecture-fitness/README.md) | You want fitness functions for perf, layering, a11y, or other characteristics. | `architecture-fitness/architecture-fitness-skill.md`, `architecture-fitness/AF-000 - architecture-fitness TEMPLATE.md`  |
| [Code Review](code-review/README.md)                   | You want a sharp PR, diff, or branch review before merging.                    | `code-review/code-review-skill.md`, `code-review/CR-000 - code-review TEMPLATE.md`                                      |
| [Dependency Audit](dependency-audit/README.md)         | You want to find vulnerabilities or trim unused npm dependencies.              | `dependency-audit/dependency-audit-skill.md`, `dependency-audit/DA-000 - dependency-audit TEMPLATE.md`                  |
| [Feature Development](feature-development/README.md)   | You need to plan a specific feature or fix before code starts.                 | `feature-development/feature-development-skill.md`, `feature-development/FD-000 - feature-development TEMPLATE.md`      |
| [Product Requirements](product-requirements/README.md) | You want to turn a product idea into a complete PRD.                           | `product-requirements/product-requirements-skill.md`, `product-requirements/PRD-000 - product-requirements TEMPLATE.md` |
| [Testing Pyramid](testing-pyramid/README.md)           | You need to plan or audit test coverage at the right layer.                    | `testing-pyramid/testing-pyramid-skill.md`, `testing-pyramid/TP-000 - testing-pyramid TEMPLATE.md`                      |

## Repo Structure

```text
open-coding-tools/
  architecture-fitness/
    architecture-fitness-skill.md
    AF-000 - architecture-fitness TEMPLATE.md
    EXAMPLE - architecture-fitness.md
  code-review/
    code-review-skill.md
    CR-000 - code-review TEMPLATE.md
    EXAMPLE - code-review.md
  dependency-audit/
    dependency-audit-skill.md
    DA-000 - dependency-audit TEMPLATE.md
    EXAMPLE - dependency-audit.md
  feature-development/
    feature-development-skill.md
    FD-000 - feature-development TEMPLATE.md
    EXAMPLE - feature-development.md
  product-requirements/
    product-requirements-skill.md
    PRD-000 - product-requirements TEMPLATE.md
    EXAMPLE - product-requirements.md
  testing-pyramid/
    testing-pyramid-skill.md
    TP-000 - testing-pyramid TEMPLATE.md
    EXAMPLE - testing-pyramid.md
```

## How I Use This Repo

| Step | Action                                                                                         | Output                                              |
|-----:|------------------------------------------------------------------------------------------------|-----------------------------------------------------|
|    1 | Pick the workflow that matches the work: PRD, FD, fitness spec, code review, or test planning. | A clear starting point instead of a blank page.     |
|    2 | Copy or invoke the relevant skill prompt in the coding agent.                                  | The agent follows the same rules every time.        |
|    3 | Create a numbered markdown artefact from the matching template.                                | A durable spec, review, or test plan.               |
|    4 | Iterate in the file with checkboxes, decisions, and tables.                                    | Decisions are folded back into the source of truth. |
|    5 | Implement, review, or test from the signed-off artefact.                                       | Less context loss and fewer surprise requirements.  |

## Harness Map

Each step is labelled by **control type** (guide ↪ feedforward, sensor ↩ feedback) and **regulation category** (
maintainability / architecture fitness / behaviour).

```text
Idea
  |
  v
Product Requirements (PRD)            ↪ guide  |  behaviour
  |
  v
Feature Development (FD)              ↪ guide  |  behaviour + maintainability
  |
  +--> Architecture Fitness (AF)      ↪ guide + ↩ sensor  |  architecture fitness
  |
  v
Implementation
  |
  +--> Testing Pyramid plan/audit     ↪ guide  |  behaviour
  |
  v
Dependency Audit (cadence)            ↩ sensor |  maintainability
  |
  v
Code Review                           ↩ sensor |  maintainability + behaviour
  |
  v
Ship
```

## Skill Classification

| Skill                   | Control type   | Regulation category         | Computational checks it triggers / expects                 | Inferential checks it performs        | Lifecycle stage                 |
|-------------------------|----------------|-----------------------------|------------------------------------------------------------|---------------------------------------|---------------------------------|
| `prd-plan`              | guide          | behaviour                   | —                                                          | structured Q&A on product scope       | pre-design                      |
| `fd-plan`               | guide          | behaviour + maintainability | declares the gate (tests, types, lint, perf budget)        | structured Q&A on design + risks      | pre-implementation              |
| `architecture-fitness`  | guide + sensor | architecture fitness        | fitness-function suite (perf, bundle, layering, a11y, SLO) | architectural-characteristic review   | pre-implementation + continuous |
| `testing-pyramid`       | guide          | behaviour                   | the test plan it produces; lint of test files              | layer-fit and bloat review            | pre + post-implementation       |
| `dependency-audit`      | sensor         | maintainability             | `npm audit`, lockfile diff, bundle-size delta              | necessity / native-replacement triage | on cadence + pre-release        |
| `principal-code-review` | sensor         | maintainability + behaviour | typecheck, test, lint output the reviewer reads first      | severity-grouped semantic review      | pre-merge                       |

## Keep Quality Left

Böckeler's "keep quality left" point: spread checks across the lifecycle by cost, with the cheapest, most deterministic
ones running first. Concretely in this repo:

| Stage               | What runs                                                                      | Why                                                                    |
|---------------------|--------------------------------------------------------------------------------|------------------------------------------------------------------------|
| Pre-design          | `prd-plan` (guide)                                                             | Cheapest possible check — disagree on paper before anyone writes code. |
| Pre-implementation  | `fd-plan`, `architecture-fitness` (guide), `testing-pyramid` plan (guide)      | Steer the model before it generates; declare the gate up front.        |
| Author's local loop | The computational gate the FD declared (tests, types, lint, perf budget, a11y) | Deterministic, milliseconds-to-seconds, runs every save / commit.      |
| Pre-merge           | `principal-code-review` (sensor)                                               | Inferential pass after the computational gate is green.                |
| On cadence          | `dependency-audit`, `architecture-fitness` continuous checks (sensor)          | Catches drift the per-PR loop won't see.                               |

If a check is cheap and computational, it belongs as far left as it'll fit. The skills here exist to be precise about
*which* check belongs *where*.

## Skill Index

| Skill                   | What it enforces                                                                          | Best for                                                                  |
|-------------------------|-------------------------------------------------------------------------------------------|---------------------------------------------------------------------------|
| `prd-plan`              | Product-level requirements, users, features, data model, UX, phases, and decision trail.  | New products, side projects, and larger product changes.                  |
| `fd-plan`               | In-file Q&A, explicit decisions, risks tied to verification, and no code before sign-off. | Feature or fix designs where implementation details matter.               |
| `architecture-fitness`  | Fitness functions for architectural characteristics (perf, bundle, layering, a11y, SLOs). | Setting and policing perf budgets, layering rules, accessibility budgets. |
| `testing-pyramid`       | Coverage by behaviour and layer, avoiding slow or duplicate tests.                        | Test plans, test suite audits, and layer-picking decisions.               |
| `dependency-audit`      | Security-first then necessity audit; outputs a report before touching any files.          | Vulnerability triage, dep trimming, native-replacement checks.            |
| `principal-code-review` | Findings-first reviews grouped by severity, with direct merge guidance.                   | Pre-merge reviews, branch sanity checks, pasted diffs.                    |

## Naming Conventions

| Artefact             | Pattern                                  | Example                           |
|----------------------|------------------------------------------|-----------------------------------|
| Architecture fitness | `AF-XXX - <scope> - <characteristic>.md` | `AF-001 - web - bundle-budget.md` |
| Code review          | `CR-XXX - <scope> - <change-slug>.md`    | `CR-001 - web - auth-flow.md`     |
| Dependency audit     | `DA-XXX - <scope> - <topic>.md`          | `DA-001 - api - q1-deps.md`       |
| Feature design       | `FD-XXX - <scope> - <short-title>.md`    | `FD-002 - app - invite-flow.md`   |
| Product requirements | `PRD-XXX - <product-name>.md`            | `PRD-003 - motivation-map.md`     |
| Testing plan/audit   | `TP-XXX - <scope> - <topic>.md`          | `TP-004 - api - billing-audit.md` |

## Why These Exist

AI coding tools generate fast and probabilistically. Without a harness around the model, momentum produces work that
looks plausible but drifts from what was intended. These skills exist to be the user's outer harness — explicit
feedforward guides before the model acts, explicit feedback sensors after, and a clear story for which checks are
deterministic vs inferential. The frame is borrowed from [Birgitta Böckeler's *Harness
Engineering*](https://martinfowler.com/articles/harness-engineering.html); the artefacts here are how I apply it to my
own work.

In practice that means:

- **Decisions explicit**: judgement calls get recorded instead of disappearing into chat history.
- **Plans executable**: a developer should be able to pick up the artefact cold and know what to do next.
- **Reviews useful**: review output should focus on risk, behaviour, and merge readiness.
- **Tests proportional**: test the behaviour at the cheapest layer that proves the claim.
- **Context portable**: the important work lives in markdown files that can move between tools.

## Interpretable Context

This repo is also influenced by the paper [*Interpretable Context Methodology: Folder Structure as Agentic
Architecture*](https://arxiv.org/abs/2603.16021) by Jake Van Clief and David McDermott.

The point is not to let tools abstract away the work so completely that we stop understanding what is happening. If all
of the reasoning, planning, and project knowledge lives inside one vendor's hidden workflow, the result is lock-in and
lost learning. I want the opposite: plain folders, readable markdown, reusable templates, and context that a human or
agent can inspect without needing a specific platform.

That is why these tools are stored as files instead of magic buttons. The structure is part of the architecture:

| Principle               | What it means here                                                                     |
|-------------------------|----------------------------------------------------------------------------------------|
| Transparent context     | Plans, reviews, prompts, and decisions are visible in the repo.                        |
| Portable workflows      | The artefacts can move between Cursor, Claude, GitHub, or whatever comes next.         |
| Human-readable defaults | Markdown beats hidden state because it can be read, reviewed, versioned, and improved. |
| Learning stays exposed  | The process shows how good work gets planned, reviewed, and verified.                  |
| Tools assist the craft  | Agents should accelerate the work, not replace understanding of the work.              |

## About Me

I'm [Benjamin Doyle](https://www.imjackofitall.com/), a software developer in Australia who likes progressing code,
ideas, and team culture. I keep my skills broad, enjoy working across different stacks, and care a lot about
collaboration, practical tooling, and building things people actually want to use.

For the most up-to-date bio, side projects, and personal links:

[![GitHub](https://img.shields.io/badge/GitHub-imjackofitall%2Fben--doyle-181717?logo=github&logoColor=white)](https://github.com/imjackofitall/ben-doyle)
[![Website](https://img.shields.io/badge/Website-imjackofitall.com-4A90E2?logo=googlechrome&logoColor=white)](https://www.imjackofitall.com/)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-benjamin--doyle--aus-0A66C2?logo=linkedin&logoColor=white)](https://www.linkedin.com/in/benjamin-doyle-aus/)

Outside code, I'm into 3D printing, music production, running, hiking, skydiving, and restoring a 1974 VW Kombi
campervan, [DZDaisy](https://www.imjackofitall.com/). That mix of digital tools and hands-on projects is a big part of
how I think about building: useful, practical, and a bit playful.

## Notes

This is a living repo. The templates are intentionally opinionated because the value is in having a repeatable default.
When a workflow earns its keep, it can get its own folder, example, and README.
