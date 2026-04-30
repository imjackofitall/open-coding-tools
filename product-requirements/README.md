# Product Requirements Skill

A PRD workflow for turning a rough product idea into a complete product specification that can guide the whole build.

## Files

| File | Purpose |
|---|---|
| `product-requirements-skill.md` | The agent skill prompt for creating and iterating on PRDs. |
| `PRD-000 - product-requirements TEMPLATE.md` | Starter template for new product requirements documents. |
| `EXAMPLE - product-requirements.md` | Example PRD output. |

## When To Use It

| Trigger | Output |
|---|---|
| "Create a PRD for this product" | A numbered PRD with the core sections scaffolded. |
| "Plan PRD-002" | An existing PRD is iterated through in-file Q&A. |
| "Turn this product idea into a spec" | Product scope, users, features, data, UX, and phases. |
| "I answered the PRD questions" | The answers are folded back into the PRD and decision trail. |

## PRD Shape

| Section | Purpose |
|---|---|
| Overview | What the product is, who it is for, and the problem it solves. |
| Tech Stack | Framework, runtime, auth, storage, and services. |
| Users | Roles, access model, and authentication expectations. |
| Features | Capabilities grouped by product area. |
| Data Model | Entities, fields, types, relationships, and enums. |
| UX Design | Principles, layout, wireframes, and key interactions. |
| Implementation Plan | Phased build plan that can be tested incrementally. |
| Open Questions | Product decisions still needing user input. |
| Decision Trail | Resolved decisions in a compact table. |

## Core Rules

- A PRD is product-level, not a code-level implementation plan.
- Every PRD should include at least one ASCII wireframe for the primary experience.
- Questions are asked in the file with concrete options and notes slots.
- Resolved answers are folded into the PRD before being moved to the decision trail.
- The PRD is ready only when it is complete, self-consistent, and signed off.
