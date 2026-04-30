---
name: fd-plan
description: Iteratively plan a feature/fix design (FD) document with the user via in-file checkbox Q&A, folding answers into the spec until signed off. Use when the user asks you to "plan FD-XXX", "work on FD-XXX", "iterate on FD-XXX", or to turn a rough plan document into a fully specified one before any code is written.
metadata:
  argument-hint: <FD-id or plan file path>
---

# FD Plan Iteration

This skill runs a tight, in-file question-and-answer loop that refines a plan document (an "FD" — feature design / fix
design) until the user has signed it off. No code is written while this skill is running. The goal is to reach a spec
where every decision is explicit, every open question is closed, and every user correction has been folded back into the
relevant section(s) of the document — not just recorded as an answer.

## When to use

Invoke this skill when:

- The user says "plan FD-XXX", "let's work on FD-XXX", "iterate on FD-XXX", "flesh out FD-XXX", "review my answers in
  FD-XXX", or similar.
- A plan document exists under `.plans/feature-development/FD-XXX - ....md` and the user wants to refine it before
  implementing.
- The user says "I answered the open questions, review and update" — that's the middle of this loop, resume it.

Do NOT invoke this skill when:

- The user has asked for code changes directly. This skill never writes code.
- The user is asking a quick question about an existing FD. Just read it and answer.

## Locating the plan file

1. If the user passed an argument that looks like a file path, read that path.
2. If they passed an ID (e.g. `FD-005`, `005`), glob for it under `.plans/feature-development/` and pick the matching
   file. Ignore `*TEMPLATE*` files.
3. If nothing matches or there are multiple candidates, ask the user which file they meant. Do not guess.
4. Always confirm the file path in your first reply so the user can correct you before you start editing.

## The iteration loop

Each cycle of the loop has four beats:

### Beat 1 — Read current state

Read the full plan file. Identify:

- Which sections of the spec are still vague (hand-wavy, TODO-ish, or contradicting each other)
- The current "Open questions" / "Decisions" / similar section — what's answered, what isn't
- Any checkboxes the user has ticked or notes they've added since the last read
- Any new requirements the user added directly into prose sections

### Beat 2 — Process user answers

If the user has answered questions since last time:

- Fold each answer into EVERY section of the spec it affects — not just the questions area. If an answer changes the
  label rules, rewrite the Label Rules section. If it changes data fetching, rewrite the Data section. If it changes
  click behaviour, rewrite the Click Handler section.
- A decision is "processed" only when the spec reads consistently as if the decision was always there. Leaving a
  resolved answer orphaned in the Open Questions area while the main spec still says something contradictory is a
  failure mode — do not do it.
- Watch for answers that contradict things you wrote earlier in this skill's run. Reconcile explicitly; never silently
  keep the old wording.
- Watch for answers that reveal a project-wide preference (e.g. a coding-style rule, a tooling convention, a copy/locale
  rule). These belong in memory via the auto-memory system, not just in this one FD.

### Beat 2b — Populate Acceptance Criteria

After processing user answers, update the `## Acceptance Criteria` section. Each scenario block uses Given-When-Then
format and covers one observable behaviour:

```markdown
#### Scenario: {scenario name}
- **Given** {precondition}
- **When** {action}
- **Then** {outcome}
```

Rules:
- One scenario per key behaviour — not one per code path.
- Write scenarios a tester could execute against the running app, not against the implementation.
- Scenarios must be consistent with the rest of the spec. If an answer changes a behaviour, rewrite its scenario.
- These scenarios feed the `testing-pyramid` skill and the `principal-code-review` spec compliance check — keep them
  precise enough to be a checklist.

### Beat 3 — Raise new questions

Answers almost always raise new questions. Surface them **in the plan file** using the in-file format below — NOT just
in chat. The user works through these in their IDE and the plan is the durable artefact.

**In-file question format:**

For a question with discrete options, use checkboxes the user can tick directly in the file:

```markdown
#### QX. {short question title}

{One or two sentences explaining the question and why it matters.}

- [ ] Option A — {short description of what this option means and when it's right}
- [ ] Option B — {short description}
- [ ] Option C — {short description}
- [ ] Other — notes:
    - _{your note here}_

**Notes / reasoning:**

- _{anything you want me to know about the pick}_
```

Rules for writing questions:

- **Every question gets a "Notes" slot.** The user can always fill it with an alternative or a reason.
- **Give your current recommendation inline.** If you think Option B is right, say so in Option B's description and
  explain the tradeoff in one line. The user is busy; don't make them derive your opinion from scratch.
- **Never ask the user to pick between options you haven't described.** Vague questions like "how should we handle X?"
  are a failure mode — always offer concrete options.
- **Don't ask questions whose answers are already derivable from the codebase.** Read the code first. Ask the user only
  for judgement calls (UX choices, priorities, scope), not facts you could grep for.
- **Questions have IDs** (Q1, Q1a, Q1b, Q2, ...) so subsequent conversation can reference them precisely.
- **Cap one round at ~3 questions.** More than that and you're not iterating, you're running a survey. If you have more,
  pick the ones that unblock the most other decisions first.

### Beat 4 — Report back and wait

Send the user a terse message (≤120 words) summarising:

- What you changed in the spec based on their answers
- What new questions you raised (by ID) and where to find them in the file
- One unilateral decision you made if you had to make one — flag it so they can override

Then stop. The user will either answer the new questions (another cycle begins) or sign off.

## Managing the decision trail

Over multiple cycles the plan accumulates answered questions. Keep them visible but compact so the document doesn't
bloat:

- Under `## Open questions`, only keep questions that are actually still open (unchecked or partially answered).
  Everything resolved moves to a "Decisions" or "Decision trail" section.
- Use a **table format** for resolved decisions — the user has expressed this preference (columns: ✅, Question,
  Decision, Why). Keep each Why cell to one sentence so the table scans fast.
- Order the table by when each question was raised, not alphabetically. The order itself carries information about how
  the design evolved.
- When a question is resolved, fold the answer into the main spec FIRST, then move the question to the decisions table.
  Never skip the fold-in step.

## Risks section — always a table, always linked to tests

Once the spec is fleshed out enough that you're naming specific files, libraries, and framework primitives, you are ALSO
responsible for surfacing risks and new issues the chosen approach would introduce. These are not questions — they are
footguns you discovered during investigation that the user should see before implementation starts.

Surface them as a `## Risks & new issues surfaced by this investigation` section with a **table**, not a list of prose
paragraphs. Columns:

| ID | Risk | Mitigation | Verification |
|----|------|------------|--------------|

Rules:

- **IDs are `R1`, `R2`, …** and are referenced elsewhere in the doc (e.g. from Files to Modify, from Decisions, from the
  Verification section).
- **Each row has at least one test ID** in the Verification column (`V1`, `V2`, …). If a risk genuinely cannot be
  tested (e.g. "pre-existing limitation, documented baseline"), write `V_ (informational)` and add a corresponding entry
  in the Verification section that records the known baseline.
- **Verification section mirrors the table**: every `Vn` referenced in the risks column must exist as a subsection under
  `## Verification`, with the list of checks for that test. The Verification section header should say "Every risk Rn
  below has at least one test. Test IDs are tagged with the risks they cover."
- **Mitigations are concrete actions**, not "be careful" — point at a specific code comment, an e2e test, a config
  flag, or a resolved question (e.g. "Resolved by Qb option 2").
- **Order by severity then discovery order.** Highest-blast-radius risks first (compile-time blockers, data loss,
  security). Informational / accepted-tradeoff risks last.
- **A risk that's resolved by an answered question cites that question** in the Mitigation cell ("Resolved by Qb (option
  2)") so the audit trail is readable.
- **Don't repeat content** between the risks table and the Verification tests — the table is the index, the Verification
  section has the actual steps.

When to add risks vs when to raise questions: if the risk requires a user decision, raise it as a numbered question (Qa,
Qb, …). If the risk is a gotcha with a clear mitigation, it goes in the table. A question can graduate to a risk once
the user answers it — keep the risk row (referencing the answered question) so the audit trail stays intact.

Watch for risks in these categories:

- **Config/flag prerequisites** (framework feature requires opt-in).
- **Silent coexistence issues** (new system + old system; what's the invalidation boundary?).
- **UX regressions from the refactor itself** (removing a lift breaks a live hint).
- **Cache-staleness windows** (SWR bought you speed, but users might see stale state for N seconds).
- **Cross-boundary cancellation/error propagation** (server actions, transitions, error boundaries).
- **State preservation across rollbacks** (optimistic updates, transitions, navigation).
- **Performance cliffs under cold cache / rate limits**.
- **Backward compatibility with saved user state** (password managers, sessionStorage, URL params).

## Sign-off gate

You are finished ONLY when:

1. Every question in the file is answered (no unticked checkboxes in open questions, no `{your note here}` placeholders
   the user was expected to fill).
2. The spec sections (Problem, Data, Label rules, Click handler, Files to modify, Verification, etc.) are
   self-consistent — you could hand the document to someone cold and they could implement it.
3. The `## Acceptance Criteria` section has at least one scenario per key user-visible behaviour.
4. The user has explicitly confirmed they want to proceed. Common sign-off phrases: "looks good, start coding", "go", "
   implement it", "ship it", "approved". If the user's latest message doesn't clearly sign off, ask: "Is this ready to
   implement, or do you want another pass?" — one sentence, then stop.

Until all four are true, stay in the loop. Do not write any code. Do not start edits to the implementation files. Do
not even read the implementation files unless you need them to answer a planning question.

## Spec graduation (post sign-off)

Once the user signs off, offer to graduate the acceptance criteria to a living spec file:

> "Want me to graduate the acceptance criteria to `/specs/{scope}/spec.md`? This creates a permanent, readable record
> of what {feature} is supposed to do — the `principal-code-review` skill will reference it automatically."

If the user agrees, write `/specs/{scope}/spec.md` with this structure (omit planning artefacts — decisions, risks, open
questions all stay in the FD):

```markdown
# {scope} — {feature short title}

> Source: FD-XXX — last updated YYYY-MM-DD

## Purpose

{one paragraph from the FD's Problem Description / Solution Overview}

## Acceptance Criteria

{paste the Acceptance Criteria section from the FD verbatim}
```

Rules for the spec file:
- Keep it short — purpose + acceptance criteria only. The FD has the reasoning.
- If a `/specs/{scope}/spec.md` already exists, merge the new scenarios in; do not overwrite the whole file.
- Update the spec when an FD changes existing behaviour, not for internal refactors.
- Link back to the FD number in the `> Source:` line so readers can find the decision trail.

## Hard rules

- **No code edits during this skill.** Only edits to the plan file and (where justified) to the user's memory system.
- **Always fold answers into the main spec.** Orphaned answers in an "Open questions" area while the rest of the spec is
  stale is the most common failure mode of this skill — guard against it.
- **Never hide questions from the user by asking them only in chat.** If it's a decision that shapes the spec, it goes
  in the file with a notes slot. Chat is for terse status updates between cycles.
- **Respect project-wide preferences** stored in memory. If a question touches one of them, default to the memory's
  answer and only raise the question if there's a real tension.
- **Watch for project-wide feedback** while iterating. If the user tells you something that clearly applies beyond this
  FD ("I prefer X over Y everywhere"), save it to auto-memory in the same cycle you're folding it into the spec — don't
  wait for a separate invitation.
- **Keep update messages terse.** The plan file is the durable artefact. Chat messages between cycles should be ≤120
  words and never repeat content that's already in the file.
- **Ask before guessing.** If an answer is ambiguous, raise it as a new question in the next cycle rather than picking
  unilaterally. If you must pick unilaterally because the decision is tiny and blocks progress, flag it explicitly in
  the chat summary so the user can override.
- **Risks are a table, not prose, and every row cites a test.** See "Risks section" above. Failing to tie each risk to a
  `Vn` test ID in the Verification section is a failure mode — the whole point of documenting the risk is to ensure it
  gets tested.

## Example cycle

Initial state: `.plans/feature-development/FD-XXX - {app} - {short slug}.md` exists with a rough problem statement, no
open questions, no data section.

Cycle 1:

- Read the file.
- Spec is sparse; a key sub-system (e.g. where state is persisted) isn't specified.
- Add an "Open questions" section with Q1 (the primary unlock — pick from a small set of concrete options), Q2 (a scope
  question that depends on Q1), Q3 (a follow-on edge case). Each has checkbox options + notes slots + your
  recommendation inline.
- Send a chat message: "Drafted 3 questions at the bottom of FD-XXX. Q1 is the main unlock — tick one and I'll build the
  rest around your choice."
- Stop.

Cycle 2 (user ticks Q1's recommended option and adds a note on Q2):

- Read the file.
- Fold the chosen option into the relevant spec section, update "Files to Modify" to mention the affected modules,
  update Verification to include the new checks.
- Process Q2's note as a partial answer; raise Q2a as a follow-up sub-question.
- Move Q1 to a new "Decisions" table.
- Send a chat message: "Folded the Q1 decision into sections 3 and 5. Raised Q2a as a follow-up — take a look."
- Stop.

Cycle N (user says "looks good, ship it"):

- Verify sign-off gate conditions.
- Confirm in chat: "Signed off. Switching out of planning mode — want me to start the implementation now?"
- Exit the skill. Implementation is a separate task.
