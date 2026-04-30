# Feature Development Skill

A planning workflow for turning a feature or fix into a signed-off implementation design before code is written.

## Files

| File | Purpose |
|---|---|
| `feature-development-skill.md` | The agent skill prompt for iterating on feature design documents. |
| `FD-000 - feature-development TEMPLATE.md` | Starter template for new FD artefacts. |
| `EXAMPLE - feature-development.md` | Example feature design document. |

## When To Use It

| Trigger | Output |
|---|---|
| "Plan FD-003" | An iterated feature design document. |
| "Flesh out this fix before coding" | A structured plan with open questions and verification. |
| "I answered the questions in the FD" | The answers are folded back into the spec. |
| "Review this feature plan" | Gaps, contradictions, and next questions are surfaced in-file. |

## FD Shape

| Section | Purpose |
|---|---|
| Status | Tracks whether the FD is open, in progress, partly implemented, or done. |
| Reason | Why the work exists. |
| Problem Description | The behaviour, bug, or need being addressed. |
| Potential Solution | The proposed approach in concrete steps. |
| Files to Modify | Expected implementation touch points. |
| Verification | Checks that prove the plan worked. |
| Open Questions | In-file checkbox Q&A for unresolved decisions. |
| Decision Trail | Resolved decisions kept as a compact table. |

## Core Rules

- No code is written while the FD loop is active.
- User answers are folded into every affected section, not left as notes.
- Open questions use checkboxes with concrete options and a recommendation.
- Risks are documented with mitigations and verification.
- The plan is done only when questions are closed, the spec is self-consistent, and the user signs off.
