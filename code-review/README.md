# Code Review Skill

A principal-engineer style review workflow for PRs, branches, staged changes, unstaged changes, and pasted diffs.

## Harness role

- **Control type:** sensor (feedback)
- **Regulation category:** maintainability + behaviour
- **Lifecycle stage:** pre-merge
- See the [harness frame](../readme.md#the-harness-frame) for vocabulary. Computational checks (typecheck, test, lint) run first; this skill is the inferential pass over the diff.

## Files

| File                               | Purpose                                                                              |
|------------------------------------|--------------------------------------------------------------------------------------|
| `code-review-skill.md`             | The agent skill prompt that defines how reviews are gathered, judged, and delivered. |
| `CR-000 - code-review TEMPLATE.md` | Starter structure for a written code review artefact.                                |
| `EXAMPLE - code-review.md`         | Example output showing the intended shape of a review.                               |

## When To Use It

| Trigger                            | Output                                               |
|------------------------------------|------------------------------------------------------|
| "Review this PR"                   | A findings-first review with merge guidance.         |
| "Sanity check this branch"         | A review against the branch diff.                    |
| "What do you think of this diff?"  | Direct feedback on the pasted or current diff.       |
| "Before I merge..."                | Risk-focused review grouped by severity.             |

## Review Shape

| Section      | What belongs there                                            |
|--------------|---------------------------------------------------------------|
| Verdict      | One clear line: ship, hold, or ship after specific fixes.     |
| Summary      | Short context on what changed and the reviewer's read.        |
| Blocker      | Must-fix issues before merge.                                 |
| Major        | Issues that should be fixed before merge.                     |
| Minor        | Worth fixing, but not necessarily merge-blocking.             |
| Nit          | Optional polish.                                              |
| Out of scope | Relevant notes that should not block this change.             |

## Review Principles

- Lead with findings, not praise.
- Cite specific files and lines where possible.
- Focus on behaviour, risk, architecture, security, accessibility, performance, and test gaps.
- Do not refactor, rename, or broaden scope during the review.
- Avoid checklist noise; only raise issues that matter.
