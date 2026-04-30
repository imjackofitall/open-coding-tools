# Open Coding Tools

Reusable coding workflows, agent skills, and planning templates for shipping software with less guesswork.

This repo is my personal toolbox for working with AI coding agents: structured prompts, repeatable planning artefacts, review rubrics, and templates that help turn vague ideas into reviewable work. The goal is simple: make the agent do the boring scaffolding, keep the important decisions visible, and leave a clear paper trail for future me.

## What's Inside

| Area | Use it when | Main files |
|---|---|---|
| [Code Review](code-review/README.md) | You want a sharp PR, diff, or branch review before merging. | `code-review/code-review-skill.md`, `code-review/CR-000 - code-review TEMPLATE.md` |
| [Feature Development](feature-development/README.md) | You need to plan a specific feature or fix before code starts. | `feature-development/feature-development-skill.md`, `feature-development/FD-000 - feature-development TEMPLATE.md` |
| [Product Requirements](product-requirements/README.md) | You want to turn a product idea into a complete PRD. | `product-requirements/product-requirements-skill.md`, `product-requirements/PRD-000 - product-requirements TEMPLATE.md` |
| [Testing Pyramid](testing-pyramid/README.md) | You need to plan or audit test coverage at the right layer. | `testing-pyramid/testing-pyramid-skill.md`, `testing-pyramid/TP-000 - testing-pyramid TEMPLATE.md` |

## Repo Structure

```text
open-coding-tools/
  code-review/
    code-review-skill.md
    CR-000 - code-review TEMPLATE.md
    EXAMPLE - code-review.md
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

| Step | Action | Output |
|---:|---|---|
| 1 | Pick the workflow that matches the work: PRD, FD, code review, or test planning. | A clear starting point instead of a blank page. |
| 2 | Copy or invoke the relevant skill prompt in the coding agent. | The agent follows the same rules every time. |
| 3 | Create a numbered markdown artefact from the matching template. | A durable spec, review, or test plan. |
| 4 | Iterate in the file with checkboxes, decisions, and tables. | Decisions are folded back into the source of truth. |
| 5 | Implement, review, or test from the signed-off artefact. | Less context loss and fewer surprise requirements. |

## Workflow Map

```text
Idea
  |
  v
Product Requirements (PRD)
  |
  v
Feature Development (FD)
  |
  v
Implementation
  |
  +--> Testing Pyramid plan/audit
  |
  v
Code Review
  |
  v
Ship
```

## Skill Index

| Skill | What it enforces | Best for |
|---|---|---|
| `principal-code-review` | Findings-first reviews grouped by severity, with direct merge guidance. | Pre-merge reviews, branch sanity checks, pasted diffs. |
| `fd-plan` | In-file Q&A, explicit decisions, risks tied to verification, and no code before sign-off. | Feature or fix designs where implementation details matter. |
| `prd-plan` | Product-level requirements, users, features, data model, UX, phases, and decision trail. | New products, side projects, and larger product changes. |
| `testing-pyramid` | Coverage by behaviour and layer, avoiding slow or duplicate tests. | Test plans, test suite audits, and layer-picking decisions. |

## Naming Conventions

| Artefact | Pattern | Example |
|---|---|---|
| Code review | `CR-XXX - <scope> - <change-slug>.md` | `CR-001 - web - auth-flow.md` |
| Feature design | `FD-XXX - <scope> - <short-title>.md` | `FD-002 - app - invite-flow.md` |
| Product requirements | `PRD-XXX - <product-name>.md` | `PRD-003 - motivation-map.md` |
| Testing plan/audit | `TP-XXX - <scope> - <topic>.md` | `TP-004 - api - billing-audit.md` |

## Why These Exist

AI coding tools are great at momentum, but momentum needs rails. These templates help keep:

- **Decisions explicit**: judgement calls get recorded instead of disappearing into chat history.
- **Plans executable**: a developer should be able to pick up the artefact cold and know what to do next.
- **Reviews useful**: review output should focus on risk, behaviour, and merge readiness.
- **Tests proportional**: test the behaviour at the cheapest layer that proves the claim.
- **Context portable**: the important work lives in markdown files that can move between tools.

## Interpretable Context

This repo is also influenced by the paper [*Interpretable Context Methodology: Folder Structure as Agentic Architecture*](https://arxiv.org/abs/2603.16021) by Jake Van Clief and David McDermott.

The point is not to let tools abstract away the work so completely that we stop understanding what is happening. If all of the reasoning, planning, and project knowledge lives inside one vendor's hidden workflow, the result is lock-in and lost learning. I want the opposite: plain folders, readable markdown, reusable templates, and context that a human or agent can inspect without needing a specific platform.

That is why these tools are stored as files instead of magic buttons. The structure is part of the architecture:

| Principle | What it means here |
|---|---|
| Transparent context | Plans, reviews, prompts, and decisions are visible in the repo. |
| Portable workflows | The artefacts can move between Cursor, Claude, GitHub, or whatever comes next. |
| Human-readable defaults | Markdown beats hidden state because it can be read, reviewed, versioned, and improved. |
| Learning stays exposed | The process shows how good work gets planned, reviewed, and verified. |
| Tools assist the craft | Agents should accelerate the work, not replace understanding of the work. |

## About Me

I'm [Benjamin Doyle](https://www.imjackofitall.com/), a software developer in Australia who likes progressing code, ideas, and team culture. I keep my skills broad, enjoy working across different stacks, and care a lot about collaboration, practical tooling, and building things people actually want to use.

For the most up-to-date bio, side projects, and personal links, start with my GitHub profile repo or website:

| Link | What you'll find |
|---|---|
| [GitHub profile repo](https://github.com/imjackofitall/ben-doyle) | Current profile README, side projects, contact links, and personal background. |
| [imjackofitall.com](https://www.imjackofitall.com/) | Portfolio, client work, blog, product links, and socials. |
| [LinkedIn](https://www.linkedin.com/in/benjamin-doyle-aus/) | Professional background and work history. |

Outside code, I'm into 3D printing, music production, running, hiking, skydiving, and restoring a 1974 VW Kombi campervan, [DZDaisy](https://www.imjackofitall.com/). That mix of digital tools and hands-on projects is a big part of how I think about building: useful, practical, and a bit playful.

## Notes

This is a living repo. The templates are intentionally opinionated because the value is in having a repeatable default. When a workflow earns its keep, it can get its own folder, example, and README.