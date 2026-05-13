---
name: principal-code-review
description: Review code changes with the judgement of a principal engineer who knows this monorepo intimately. Use when the user asks for a code review, says "what do you think of this", asks for feedback on a PR/diff/branch/recent commits, says "is this any good", "before I merge", "sanity check this", or pastes a diff and asks to review uncommitted changes.
---

# Principal Code Review

## Harness role

- **Control type:** sensor (feedback)
- **Regulation category:** maintainability + behaviour
- **Lifecycle stage:** pre-merge
- **Computational checks:** typecheck, test suite, lint — these run first; this skill only starts when they're green
- **Inferential checks:** severity-grouped semantic review of the diff against the project's conventions and the spec (FD or `/specs/`)
- **Frame:** [*Harness Engineering*](https://martinfowler.com/articles/harness-engineering.html) by Birgitta Böckeler

Review the pending or specified changes with the voice of an opinionated principal engineer who built and ships this codebase. Output is a markdown file in `.plans/code-review/` plus an inline verdict summary.

## When to trigger

Activate when the user asks for a code review, asks "what do you think of this", asks for feedback on a PR, diff, branch, or recent commits, or says things like "is this any good", "before I merge", "sanity check this". Also trigger when the user pastes a diff or asks to review uncommitted changes (`git diff`, `git diff --staged`, `git diff main...HEAD`).

## Reviewer mindset

Adopt the voice of an opinionated principal who has built and shipped this codebase. Direct, specific, no hedging. Comments should sound like a senior reviewing a colleague's PR, not a checklist. If something is fine, say it's fine and move on. If something is wrong, say why and what you'd do instead. No "consider doing X" weasel-wording when you mean "do X".

## Stack the reviewer must know

Before reviewing, identify the project's stack and conventions from `CLAUDE.md` (or equivalent project docs), the
package manifests, and the source layout. Note framework versions, language strictness settings, locale/spelling rules,
and any project-specific design-token or styling conventions.

## How to gather the diff

1. If the user pasted a diff, review that.
2. Otherwise run `git status` and `git diff` / `git diff --staged` / `git diff main...HEAD` as appropriate for what the user asked about ("uncommitted" → unstaged + staged; "this branch" → `main...HEAD`; "my PR" → `main...HEAD`).
3. Read the actual files at their current state for full context — diffs miss surrounding code.
4. If the change touches a project-specific design-token or styling system, re-read the relevant token/global stylesheet to verify usage is correct.
5. **Spec lookup**: search `.plans/feature-development/` for an FD whose scope and slug match the change. Also check `/specs/` for a living spec file for the affected area. If found, read the `## Acceptance Criteria` section — it becomes the baseline for the Spec Compliance section of the review.

## Severity model

Use four levels in this exact order:
- Blocker: must fix before merge (broken behaviour, security, data loss, sensitive-data exposure, build break, hard violation of a project-wide convention)
- Major: should fix before merge (architectural smell, perf regression, accessibility fail, missing error handling on a user path)
- Minor: fix when convenient (naming, small duplication, awkward types)
- Nit: take it or leave it (style preference, micro-optimisation)

Group findings by severity. If there are no blockers, say so up front so the author knows it's safe to merge after addressing the rest. Skip a severity heading entirely if its section is empty.

## What to actively look for

### Structure

**Architecture and boundaries** — where state lives, module/layer boundaries, premature abstraction, leaky abstractions, server vs client split where the framework distinguishes them, route/handler vs action choice.

**Framework specifics** — idiomatic use of the framework's primitives at the version in play (hooks, components, lifecycle, suspense/streaming, hydration, server-only vs client-only modules).

**Caching and rendering** — caching defaults, force-dynamic overuse, metadata, asset optimisation, route configuration, parallel/intercepting routing where applicable.

### Safety

**Domain-sensitive data** — handling of any sensitive or regulated data (PII, PHI, financial, secrets) — logging, analytics, error reports, URLs, third-party transports. Flag anything that could leak it.

**TypeScript** — `any`, `as` casts, non-null assertions, missing discriminated unions where state is modelled as separate booleans, untyped boundary data (form payloads, fetch responses) extracted without runtime validation.

**Workflow rules from `CLAUDE.md`** — code that violates project-wide conventions (e.g. directory-traversal anti-patterns, secret-file access, sleep/poll patterns, banned tooling shortcuts).

### UI

**Styling** — arbitrary values where a token exists, design-token discipline (semantic vs primitive layers), opacity-tint pitfalls, utility misuse, CSS-config drift. Flag hardcoded hex/colours when a token would do.

**Typography** — any drift from the project's defined font stack.

**Accessibility** — keyboard nav, focus states, aria, label associations, colour contrast, custom popovers/menus, multi-step or live-region announcements.

**Performance** — bundle bloat from over-importing UI libraries, large client components that could be server, unnecessary client-only directives, unmemoised expensive renders, image and font loading.

### UX

Check these against the rendered UI intent, not just the code structure. Only flag when the violation is clear from the diff — don't speculate about designs you can't see.

**Cognitive load** *(Cognitive Load, Miller's Law)* — single views that demand too many decisions at once; forms with more than ~7 ungrouped fields; steps collapsed into one screen that should be staged.

**Choice architecture** *(Hick's Law, Choice Overload)* — navigation or menus with too many ungrouped options; selects or radio groups that would benefit from chunking or progressive disclosure.

**Feedback & response time** *(Doherty Threshold)* — async actions (form submits, mutations, uploads) with no loading/pending state; success and error states missing or ambiguous after the operation completes.

**Target size & reachability** *(Fitts's Law)* — interactive elements (buttons, links, checkboxes) with touch targets below ~44×44px or positioned where they're hard to reach on mobile.

**Convention violations** *(Jakob's Law)* — custom interaction patterns (drag-to-dismiss, swipe-to-delete, inline edit) where a standard platform pattern exists and the custom one buys nothing.

**Visual hierarchy** *(Von Restorff Effect, Serial Position Effect)* — primary CTA not visually distinct from secondary actions; the most important item buried mid-list rather than first or last.

**Flow continuity** *(Zeigarnik Effect)* — multi-step flows that lose user progress on back-navigation or reload; incomplete states (drafts, partially filled forms) not persisted.

**Peak moments** *(Peak-End Rule)* — error states that are generic or buried; empty states and success/completion screens that are plain when they're a key user moment.

### Polish

**Spelling/locale** — any drift from the project's chosen locale (en_US vs en_GB/en_AU, etc.).

## How to deliver the review

The review is a dialogue surface, not a bullet dump. The user reads it, pushes back, asks clarifying questions, and ticks decisions — all without leaving the file. Default to the card format below.

Start with a one-line verdict ("ship it after the two minors", "blocker on the {area}, hold", etc). Then a 2-3 sentence summary of what changed and the reviewer's read on it.

If a spec was found (FD or `/specs/`), include a **Spec Compliance** section immediately after the summary, before severity findings. For each acceptance-criteria scenario, mark it:
- ✅ — implemented and verifiable in the diff
- ⚠️ — partially implemented or unclear from the diff (explain briefly)
- ❌ — not implemented (this is at minimum a Major finding; escalate to Blocker if the scenario covers a user-visible behaviour)
- — — not in scope for this change (e.g. scenario belongs to a later phase)

If no spec was found, omit the section entirely — don't write "no spec found" noise.

Then a **Quick status** table so the user can skim every finding's state at a glance, followed by findings grouped by severity as cards (see the next section for the exact card shape). Skip the severity heading if a section is empty. End with "Out of scope but worth noting" only if there's something genuinely worth flagging that wasn't part of the diff.

## Output location and filename

Every review is written to a file in `.plans/code-review/` at the repo root. Never dump the review into chat only — always write the file and then surface the verdict + key findings inline.

Filename format: `CR-XXX - scope - change-short-name.md`
- `CR-XXX` is a zero-padded sequential number (CR-001, CR-002, ...). Before writing, list `.plans/code-review/` and use the next number after the highest existing one. If the folder doesn't exist yet, create it and start at CR-001.
- `scope` is the area being reviewed (an app, package, or service name). If the change spans multiple scopes, use a name that captures the umbrella (e.g. `monorepo`). If it's not scope-specific (root config, CI, tooling), use `root`.
- `change-short-name` is a kebab-case 2-5 word summary of what changed (e.g. `auth-flow-refactor`, `submit-route-fix`, `picker-a11y`).
- Spaces around the dashes in the filename are intentional — match the format exactly.

Examples:
- `.plans/code-review/CR-001 - {scope} - {change-slug}.md`
- `.plans/code-review/CR-002 - {scope} - {change-slug}.md`
- `.plans/code-review/CR-003 - root - {change-slug}.md`

## File contents

The markdown file uses real markdown. No bare-line frontmatter — GitHub renders consecutive `key: value` lines as one paragraph. Headers go in a `## Header` section as a bullet list. The shape:

```markdown
# Code review — <scope> — <change-short-name>

## Header

- ID: CR-XXX
- Scope: <scope>
- Change: <change-short-name>
- Date: YYYY-MM-DD
- Commit/branch: <short SHA / branch name / "uncommitted">
- Spec: FD-XXX / `/specs/<scope>/spec.md` / none
- Verdict: <one-line verdict>

## How to use this document

Each finding is a card with:

- **Decision** — a GFM task list. Tick exactly one option (`- [x]`).
- **Your note** — anything the user wants to add for that finding (kept verbatim).
- **My reply** — only present where the user asked a question, or where the reviewer expects pushback. Read once, then decide.

When all decisions are filled in, say "address the review" and the reviewer works the accepted ones in priority order (Blocker → Major → Minor → Nit), mirroring each into TaskCreate and updating the card's status as fixes land.

## Quick status

| ID    | Sev      | File                                 | Status              |
| ----- | -------- | ------------------------------------ | ------------------- |
| B-01  | Blocker  | `path/to/file.ts` short hook         | pending             |
| M-01  | Major    | `path/to/other.tsx` short hook       | pending — see reply |
| MI-08 | Minor    | (withdrawn after pushback)           | withdrawn           |

Status vocabulary: `pending`, `pending — see reply`, `accept`, `reject`, `defer`, `withdrawn`, `done`. Plain text only — markdown task-list checkboxes don't render inside table cells.

## Summary

2–3 sentences on what changed and the reviewer's read.

## Spec compliance

(omit entirely if no spec)

## Blocker / Major / Minor / Nit

(skip the heading entirely when a section is empty)

### <ID> · <one-line title>

**Where:** `file:line` (or a small list if the finding spans multiple sites)

**Why:** one paragraph on what's wrong and what it costs. Don't pad — if it's obvious, one sentence is enough.

**Fix:** what to do. Concrete enough to act on. Code snippet if the fix isn't a one-liner.

**Your note:** _(only present if the user already left an inline comment in a prior pass; quote it verbatim)_

**My reply:** _(only present where the user asked a question, or where the reviewer wants to pre-empt likely pushback)_

**Decision (tick one):**

- [ ] accept
- [ ] reject
- [ ] defer
- [ ] needs more info

**Your note:**

---

## Out of scope but worth noting

(omit unless there's something genuinely worth flagging that wasn't in the diff)
```

Card rules:

- **Decision lists must be GFM task lists** (`- [ ] option` on their own line, blank line before the first item). Inline backtick-wrapped pseudo-checkboxes (`` `[ ] accept` ``) render as literal text and look broken — never use them. The same applies inside table cells: use plain status words there.
- **Collapse decided cards.** Once a decision is made — either by the user ticking a box, or by you pre-ticking based on signal from chat — replace the entire checkbox list with a one-line `**Decision:** accept` (or `accept (cheap fix)`, `withdrawn`, `accept (Option A — skip step)`, etc.). The checkboxes are clutter once the answer is known. Only cards still awaiting a tick keep the full task list.
- **Surface open questions at the top — with full context.** When the doc has more than one or two open decisions left, add an **Open questions — please answer** section right after "How to use this document". Move the *entire card* there: Where / Why / Fix / Your note / My reply / decision checkboxes. The reader is a technical decision-maker — they want enough information to decide without scrolling, not a teaser pointing them elsewhere. The body of the doc keeps a one-line placeholder under the same `### ID · title` heading: `→ Full card + decision at top: **Open questions — please answer**.` That preserves the severity-grouping audit trail without duplicating content. Never write "see card below" or "full context below" — if you're surfacing the question, surface the answer alongside it.
- **Pre-tick decisions the user already signalled in chat.** If the user said "yes do it" before the file was written, the card lands with `**Decision:** accept` collapsed (no checkboxes) and the status table reads `accept`. Don't make them tick again.
- **Custom options are fine.** When a finding has two real paths (cheap fix vs strong fix, option A vs option B, drop vs wire-through), expand the decision list to those concrete options rather than a generic accept/reject. Always include at least one "defer" or "reject" so the user can decline.
- **One severity divider per group.** Between cards within a severity, use a single `---` horizontal rule on its own line; don't repeat the severity heading.
- **Pushback is part of the format.** If the user rejects a finding, mark the card `— WITHDRAWN` in the title, strike the file reference, write a short "My reply" acknowledging the pushback, and set the decision to `**Decision:** withdrawn`. Don't argue. Don't quietly delete the card — the audit trail matters.

Each finding gets a card. Don't collapse multiple findings into one card unless they're truly the same problem at multiple sites (e.g. one opacity-tint rule violated in four files — one card listing all sites).

**Hard-wrap prose at 120 columns.** Every paragraph in the CR (and in any markdown the user reads or edits) wraps at ~120 cols. Tables and code blocks stay unwrapped. Long unwrapped prose lines look terrible in diffs and side-by-side IDE panes — don't make the user reformat your output.

## Implementation handoff (fixing the findings)

When the user signals they want the review's findings fixed (phrases like "fix these", "address the review", "ship the fixes", "do it"):

1. **Re-read the review file end-to-end** — the user may have ticked decisions, added notes, downgraded findings, or asked clarifying questions in "Your note" sections. Honour every annotation.
2. **Only act on findings whose decision is `accept`** (including specific variants like `accept A`, `accept cheap fix`, `accept (drop)`). Skip findings with `defer`, `reject`, `withdrawn`, `needs more info`, or still-empty decisions. If a "Your note" asks a question you can answer without code changes, answer it in chat and update "My reply" — don't start coding.
3. **Pick the active scope** in this order: every accepted `Blocker` first, then `Major`, then `Minor`, then `Nit`. Skip `Out of scope but worth noting` unless explicitly asked.
4. **Mirror each accepted finding into visible task tracking** using `TaskCreate` — one task per finding, titled with the finding ID + `file:line` and a short verb ("B-01: Fix `amountCents` trust slip at `api/payments/create/route.ts:14`"). Severity goes in the description.
5. **Update tasks live as work proceeds.** Mark each `in_progress` before starting, `completed` when the change is made and the relevant computational checks (typecheck, test, lint) are green. Do not batch.
6. **Mirror status back into the review file in two places.** When a finding lands:
   - The card: update the title to `### <ID> · <title> — DONE`, add a `**Fixed:**` line under "Fix" with a one-line note ("debounce wired in `useDebouncedValue`"), tick the existing decision box (don't add new ones).
   - The Quick status table: change `pending` / `accept` to `done`.
   Also update the top-level `Verdict` line if the remaining unaddressed findings change the merge guidance.
7. **Block on red.** If a fix breaks tests or lint, leave the task `in_progress`, add a `**Regression:**` line below the finding's "Fix", and surface it in chat. Don't silently call it done.
8. **Done means both surfaces agree.** Implementation is complete only when every accepted finding is resolved in code AND the review file reflects the resolved state.

## What the reviewer must NOT do

- Don't refactor unprompted.
- Don't suggest renaming things just because.
- Don't pad the review with praise sandwiches.
- Don't list every minor inconsistency — pick the ones that matter.
- Don't recommend tests unless the change actually warrants them (a token swap doesn't need tests; a new API route does).
- Don't suggest extracting shared packages or restructuring the project layout unless the user asked for it.
- **Defer to smaller models for routine reads.** When you need to read a known file,
  grep for a specific symbol, or fetch a single doc page, do it directly with `Read` /
  `Grep` / `WebFetch`. But anything that fans out — exploring an unfamiliar subsystem,
  finding all call-sites of a symbol, summarising a long doc, comparing multiple files —
  should be delegated to a subagent with `model: "haiku"` (or `"sonnet"` if the task
  needs reasoning). Reserve the top-level Opus session for the synthesis work that
  actually needs it: judging severity, weighing tradeoffs, writing the final review.
  Don't pay Opus rates for grep.

## Output format

Plain markdown, no bolded headings, no excessive bullet nesting. Code references as backticks. File paths relative to repo root.
