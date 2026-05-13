# Code review — {scope} — {change short name}

## Header

- ID: CR-XXX
- Scope: {app, package, or service name; or `root` for tooling/CI}
- Change: {kebab-case short name}
- Date: YYYY-MM-DD
- Commit/branch: {short SHA, branch name, or `uncommitted`}
- Spec: FD-XXX / `/specs/{scope}/spec.md` / none
- Verdict: {one-line verdict}

## How to use this document

Each finding is a card. Decisions collapse to a one-line `**Decision:**` once made — checkbox lists only appear on
cards still awaiting an answer, and the open questions are surfaced near the top with full context so you don't have
to scroll.

When everything is decided, say "address the review" and I'll work the accepted findings in priority order
(Blocker → Major → Minor → Nit), mirroring each into TaskCreate and updating the card + status table as fixes land.

## Open questions — please answer

_Only present when one or more cards still need a decision and they outnumber what's comfortable to scan inline.
Move the entire card (Where / Why / Fix / Your note / My reply / decision checkboxes) here. The body of the doc keeps
a one-line placeholder under the same `### ID · title` heading: `→ Full card + decision at top: **Open questions —
please answer**.` Never write "see card below" or "full context below" — surface the answer alongside the question._

_Delete this whole section if there are no open questions or only one trivial one._

---

### {ID} · {one-line title}

**Where:** `file:line`

**Why:** {short problem statement}

**Fix:** {what to do; code snippet if not a one-liner}

**Your note:** _"{verbatim quote from the user's inline note}"_

**My reply:** {answer to the user's question or pre-empt of likely pushback}

**Decision (tick one):**

- [ ] accept
- [ ] reject
- [ ] defer

---

## Quick status

| ID    | Sev      | File                                          | Status                |
|-------|----------|-----------------------------------------------|-----------------------|
| B-01  | Blocker  | `path/to/file.ts` — short hook                | pending               |
| M-01  | Major    | `path/to/file.tsx` — short hook               | pending — see reply   |
| MI-01 | Minor    | `path/to/file.tsx` — short hook               | accept                |
| MI-02 | Minor    | `path/to/file.tsx` — short hook               | withdrawn             |
| N-01  | Nit      | `path/to/file.ts` — short hook                | accept                |

_Status vocabulary: `pending`, `pending — see reply`, `accept`, `reject`, `defer`, `withdrawn`, `done`. Plain text
only — markdown task-list checkboxes don't render inside table cells._

## Summary

_2–3 sentences: what changed and the reviewer's read on it._

## Spec compliance

_Omit this section entirely if no spec was found._

| Scenario        | Status            | Notes |
|-----------------|-------------------|-------|
| {scenario name} | ✅ / ⚠️ / ❌ / — |       |

---

## Blocker

_Skip the heading entirely if empty. One card per finding._

### B-01 · {one-line title}

**Where:** `file:line` (or a small list if the finding spans multiple sites)

**Why:** {one paragraph on what's wrong and what it costs. Don't pad — if it's obvious, one sentence is enough.}

**Fix:** {what to do. Concrete enough to act on. Code snippet if the fix isn't a one-liner.}

**Decision:** accept

---

## Major

_Skip the heading entirely if empty._

### M-01 · {one-line title}

**Where:** `file:line`

**Why:** {short problem statement}

**Your note:** _"{verbatim quote, if the user has annotated}"_

**My reply:** {only present where the user asked a question, or where the reviewer pre-empts likely pushback}

**Fix:** {what to do}

**Decision (tick one):**

- [ ] accept
- [ ] reject
- [ ] defer

---

### M-02 · {one-line title — example of a multi-path decision}

**Where:** `file:line`

**Why:** {short problem statement}

**Fix:** {what to do}

**Decision (tick one):**

- [ ] accept cheap fix (description)
- [ ] accept stronger fix (description)
- [ ] reject
- [ ] defer

---

## Minor

_Skip the heading entirely if empty._

### MI-01 · {one-line title — example of a card that points to the top open-questions section}

→ Full card + decision at top: **Open questions — please answer**.

---

## Nit

_Skip the heading entirely if empty._

### N-01 · {one-line title — example of an already-accepted card}

**Where:** `file:line`

**Why:** {short problem statement}

**Fix:** {what to do}

**Decision:** accept

---

## Out of scope but worth noting

_Skip the heading entirely if empty._

- {one-line observation that wasn't in the diff but is worth flagging}

---

## Card-shape rules (reference)

- **Decision lists are GFM task lists** (`- [ ] option` on their own line, blank line before the first item). Never
  inline backtick-wrapped pseudo-checkboxes (`` `[ ] accept` ``) — they render as literal text.
- **Plain text in table cells.** Markdown task-list syntax doesn't render inside table cells; use words like `pending`,
  `accept`, `withdrawn` in the Quick status table.
- **Collapse decided cards.** Once a decision is made — either by the user ticking or by the reviewer pre-ticking
  based on chat signal — replace the checkbox list with a single line `**Decision:** accept` (or `withdrawn`, `accept
  (Option A — skip step)`, etc.). The checkboxes are clutter once the answer is known.
- **Pre-tick decisions the user already signalled in chat.** If they said "yes do it" before the file was written, the
  card lands with `**Decision:** accept` collapsed (no checkboxes) and the status table reads `accept`. Don't make
  them tick again.
- **Custom options for multi-path findings.** When a finding has two real paths (cheap fix vs strong fix, option A vs
  option B, drop vs wire-through), expand the decision list to those concrete options. Always include at least one
  `reject` or `defer` so the user can decline.
- **Withdrawn findings keep a card.** If the user pushes back, mark the title `— WITHDRAWN`, strike the file
  reference (`~~file:line~~`), write a short "My reply" acknowledging, and set `**Decision:** withdrawn`. Don't argue.
  Don't quietly delete — the audit trail matters.
- **Surface open questions at the top, with full context.** When the doc has more than one or two open decisions,
  move the *entire card* into an `## Open questions — please answer` section right after `How to use`. Body keeps a
  one-line `→ Full card + decision at top: **Open questions — please answer**.` placeholder under the same
  `### ID · title` heading. Never write "see card below" or "full context below" — surface the answer alongside the
  question.
- **One `---` divider between cards.**
- **Hard-wrap prose at 120 columns.** Tables and code blocks stay unwrapped.
- **Don't collapse multiple findings into one card** unless they're truly the same problem at multiple sites (e.g.
  one opacity-tint rule violated in four files — one card listing all sites).
