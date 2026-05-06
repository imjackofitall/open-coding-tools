---
name: fd-plan
description: Iteratively plan a feature/fix design (FD) document with the user via in-file checkbox Q&A, folding answers into the spec until signed off. Use when the user asks you to "plan FD-XXX", "work on FD-XXX", "iterate on FD-XXX", or to turn a rough plan document into a fully specified one before any code is written.
metadata:
  argument-hint: <FD-id or plan file path>
---

# FD Plan Iteration

## Harness role

- **Control type:** guide (feedforward)
- **Regulation category:** behaviour + maintainability
- **Lifecycle stage:** pre-implementation
- **Computational checks:** the FD declares the gate in its `## Computational gate` block — tests, type checks, lint at minimum, plus any perf budget, bundle ceiling, or accessibility budget the work touches. Keep quality left: name the cheap deterministic checks before generation starts
- **Inferential checks:** structured in-file Q&A on design, scope, and risk, with a recommendation per question and a notes slot
- **Frame:** [*Harness Engineering*](https://martinfowler.com/articles/harness-engineering.html) by Birgitta Böckeler

This skill runs a tight, in-file question-and-answer loop that refines a plan document (an "FD" — feature design / fix
design) until the user has signed it off. No implementation code is written while this skill is running. The goal is to
reach a spec where every decision is explicit, every open question is closed, and every user correction has been folded
back into the relevant section(s) of the document — not just recorded as an answer.

The FD is also a **living record**, not just a planning artefact. Once implementation begins (in a separate task), the
FD continues to evolve: each phase that ships gets an Implementation Notes block appended, capturing what actually
changed, deviations from the plan, and follow-ups. See "Phases and the implementation template" below — that template
applies during implementation, not during planning, but the FD's structure must accommodate it from day one.

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

After processing user answers, update acceptance criteria. **Placement depends on FD shape:** flat FDs use a top-level
`## Acceptance Criteria` section; phased FDs put criteria inside each `### Phase N` block (under `#### Acceptance
criteria`) so each phase reads as a self-contained slice. Same Given-When-Then format either way. Each scenario block
uses:

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

**Placement depends on FD shape.** Flat FDs put a single `## Risks & new issues surfaced by this investigation` section
near the bottom. Phased FDs put a `#### Risks & verification` block inside each `### Phase N` so risks travel with the
phase they apply to. Same table format either way; IDs stay unique across the whole FD (Phase 2's first risk continues
the numbering from Phase 1's last).

Surface as a **table**, not a list of prose paragraphs. Columns:

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

## Phases and the implementation template

Most FDs deliver a single feature in one go. Some span multiple visible-to-the-user phases (e.g. ship Phase 1 minimal,
review, then ship Phase 2 logos). When an FD has phases, surface them as the dominant structure of the document — a
top-level `## Phases` section with each phase as a `###` heading.

### When to use phases

Add a `## Phases` section if any of the following is true:

- The user asks for "phased" delivery, "ship X first, then Y", or "let me see X before we do Y".
- A decision question's answer is "do X now, defer Y" (Q1's resolution, etc.).
- The work crosses a natural review boundary (a render the user wants to eyeball before more code lands).
- Total scope is large enough that landing it in one PR is risky.

If none of these apply, keep the FD flat — no `## Phases` section, no per-phase blocks. Don't manufacture phases for
ceremony. A flat FD with `## Solution Overview` + `## Files to Modify` is fine for single-shot work.

### Per-phase structure

Each phase is a `### Phase N — {short title}` heading. The phase block is **self-contained**: everything scoped to that
phase (acceptance scenarios, risks, verification, implementation notes) lives inside it, not scattered across top-level
sections. This is a deliberate readability rule — the user expects related things grouped together. Skim a phase
top-to-bottom and you have the full picture.

```markdown
### Phase N — {short title}

**Status**
- [ ] Not started
- [ ] In progress
- [ ] Shipped YYYY-MM-DD
- [ ] Deferred

#### Plan

{Tight summary of what this phase ships, what it doesn't, and why this slice. Pipeline steps, file targets, behaviours.
Riffing-level concise — same density as a flat FD's Solution Overview.}

#### Acceptance criteria

##### Scenario: {phase-scoped scenario name}
- **Given** {precondition}
- **When** {action}
- **Then** {outcome}

(One scenario per key behaviour for THIS phase.)

#### Risks & verification

| ID | Risk | Mitigation | Verification |
| --- | --- | --- | --- |
| Rn | {phase-scoped risk} | {concrete mitigation} | Vn |

##### Vn — {short title} (Rn)
- [ ] {concrete check, ticked when run}

#### Implementation notes

_(filled in during/after implementation — leave empty until then)_
```

Cross-phase content (Problem Description, the Decisions table, Open Questions) stays at the top of the FD as global
sections. Anything phase-scoped belongs inside the phase block.

**Status uses a checkbox list, not prose.** Mirrors the top-of-FD overall status block; consistent visual language
across the document.

**Verification entries use checkboxes** (`- [ ] {check}`) so reviewers can tick them off as they run them.

**Risk and verification IDs are unique across the whole FD**, even though they live inside phase blocks (so R3 in Phase
1, R4 in Phase 2 — never two R3s). Phase-2 IDs continue from where Phase 1 left off.

### The implementation template

Once a phase ships, fill in its `#### Implementation notes` block using this template. It is intentionally more verbose
than the Plan block — a reviewer with no session context should be able to reconstruct the change from these notes
alone.

```markdown
#### Implementation notes

##### Files touched

- `path/to/file.ts` — {what changed, in one line. Reference the function or section if non-obvious.}
- `path/to/other.ts` (new) — {what it is and why.}

(No new dependencies / Added dep `foo@1.2`. / No env vars added. / No DB changes.)

##### Deviations from the plan

- **{Heading of the deviation}.** {One short paragraph: what the plan said, what was actually done, why the change was
  made, whether it was reversible. If a deviation should have looped back to the user but didn't, flag that explicitly.}

(Use `_None._` if the implementation matched the plan exactly.)

##### How to test (manual, this session)

1. {Concrete step a reviewer can run.}
2. {Next step.}
3. {What to look for to confirm the phase is good.}

##### Verification status

- V1 — {pass / fail / pending user manual test}.
- V2 — …

##### Follow-ups deferred

- {Anything the implementation surfaced that didn't ship in this phase. If big enough to warrant its own FD, say so.}

(Use `_None._` if nothing was deferred.)
```

Verbosity rule: **Plan blocks stay terse, Implementation notes get verbose.** During planning, riff at the Plan
density. During implementation, the notes are the durable record of what actually happened.

### Cost discipline during implementation

The same defer-to-smaller-models rule from the planning loop applies once you're implementing the FD's phases — and
arguably matters more, because implementation work fans out across many files. Concretely:

- **Reading the codebase to confirm a file path or current behaviour** → `Explore` subagent (Haiku-class).
- **Generating boilerplate** (migrations, route handlers from a template, test scaffolds) → `general-purpose` subagent
  with `model: "haiku"` or `"sonnet"`, given the relevant spec slice as context.
- **Drafting Implementation-notes blocks** (Files touched, How-to-test, Verification status) → `general-purpose`
  subagent with `model: "haiku"`; the top-level session reviews and edits.
- **Reserve Opus for**: applying user feedback to the spec, resolving spec/code contradictions, security-sensitive
  edits, and anything where getting it wrong wastes more than the model-cost saving.

Cheapest path: top-level session decides *what* to do, subagent does it, top-level session reviews the diff. Don't
skip the review — small models will sometimes drift — but do skip the manual-typing toil.

#### Model routing table (mandatory in every Plan / phase Plan)

The defer-to-smaller-models rule above is theoretical until you write it down per step. **Every Plan block — flat or
phased — includes a Model routing table** so the cheapest viable model is picked deliberately rather than by reflex.

Template (paste verbatim into the Plan block, fill in per-step):

```markdown
#### Model routing

| Step | Model      | Reason                                                  |
| ---- | ---------- | ------------------------------------------------------- |
| 1    | **haiku**  | {one line — pure mechanical work}                       |
| 2    | **sonnet** | {one line — bounded judgement, well-specified}          |
| 3    | **opus**   | {one line — design / risky / cross-cutting / synthesis} |

If a sonnet/haiku step surfaces a non-trivial decision, escalate to the main session rather than guess.
```

**Rubric for picking the model:**

- **haiku** — pure mechanical: run a command, grep, count lines, file moves, single-line edits, install a dep, confirm
  a number, paste known content. No judgement required; if the agent has to choose between options, it's not haiku-class.
- **sonnet** — bounded judgement: apply a pattern the FD specifies, refactor following a recipe, fix lint errors with
  clear rules, write boilerplate from a spec, classify hits into a fixed set of buckets. The agent makes small calls
  inside well-defined rails.
- **opus** (= top-level session) — design, architecture, risky migrations, cross-cutting changes, synthesising multiple
  inputs, anything where getting it wrong wastes more than the model-cost saving. Don't delegate this.

**Rules:**

- **Each step in the Plan gets a row.** No row → no execution. If a step has no row, the Plan is incomplete.
- **Default towards cheaper.** If you're choosing between sonnet and opus and the work is well-specified by the FD,
  pick sonnet. Reserve opus for the parts where the FD doesn't fully prescribe the answer.
- **The escalation line is mandatory.** Sonnet/haiku subagents must know they can return without guessing — they will
  guess otherwise.
- **Re-evaluate after the design step.** Once the inventory/design step (usually opus) is done, the remaining steps are
  often more mechanical than first thought; downgrade them if so.

### When implementation notes get written

Implementation notes are NOT written by the planning loop. They're written:

- During the implementation task, immediately after each phase's code lands (or in tight increments as code lands).
- Or retroactively, when the user asks ("update the FD with what you did", "add implementation notes for Phase N").

The planning skill's job is to (a) decide whether the FD has phases, (b) lay out the per-phase Plan blocks, and (c)
leave the Implementation notes placeholder so the structure is ready when implementation starts.

### Updating the Decisions table during implementation

If a unilateral decision was made during implementation (e.g. "coalesce horizontal runs in the EPS writer"), append it
to the existing Decisions table with an `(impl)` tag in the Question column so the audit trail captures decisions made
outside the planning loop. Keep the table chronological — implementation decisions go at the bottom.

## Sign-off gate

You are finished ONLY when:

1. Every question in the file is answered (no unticked checkboxes in open questions, no `{your note here}` placeholders
   the user was expected to fill).
2. The spec sections (Problem, Data, Label rules, Click handler, Files to modify, Verification, etc.) are
   self-consistent — you could hand the document to someone cold and they could implement it.
3. The `## Acceptance Criteria` section has at least one scenario per key user-visible behaviour.
4. **Every Plan block (flat or per-phase) has a populated Model routing table** — one row per step, model picked
   deliberately from the rubric, escalation line present. See "Model routing table (mandatory in every Plan / phase
   Plan)". A Plan without this table isn't signed off, no matter how good the rest looks.
5. The user has explicitly confirmed they want to proceed. Common sign-off phrases: "looks good, start coding", "go", "
   implement it", "ship it", "approved". If the user's latest message doesn't clearly sign off, ask: "Is this ready to
   implement, or do you want another pass?" — one sentence, then stop.

Until all five are true, stay in the loop. Do not write any code. Do not start edits to the implementation files. Do
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
  Implementation-notes updates to the FD happen in a separate task, not during the planning loop.
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
- **Defer to smaller models for routine reads.** When you need to read a known file, grep for a specific symbol, or
  fetch a single doc page during the planning loop, do it directly with `Read` / `Grep` / `WebFetch`. But anything that
  fans out — exploring an unfamiliar subsystem, finding all call-sites of a symbol, summarising a long doc, comparing
  multiple files — should be delegated to a subagent with `model: "haiku"` (or `"sonnet"` if the task needs reasoning).
  Reserve the top-level Opus session for the synthesis work that actually needs it: reconciling user answers with the
  spec, judging risk severity, deciding question wording. The planning loop is mostly orchestration; don't pay Opus
  rates for grep.
- **Every Plan block has a Model routing table.** A Plan section without per-step model assignments isn't signed off,
  even if every other section is complete. Use the template in "Model routing table (mandatory in every Plan / phase
  Plan)" above. The sign-off gate enforces this.

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
- If the FD has phases, confirm each phase has a `#### Plan` block populated and an empty `#### Implementation notes`
  placeholder ready.
- Confirm in chat: "Signed off. Switching out of planning mode — want me to start the implementation now?"
- Exit the skill. Implementation is a separate task.

Post-implementation (separate task, not part of the planning loop):

- After Phase N's code lands, append the Implementation notes block to that phase using the template above.
- Files touched, deviations, test instructions, verification status, follow-ups.
- Update the Status line on Phase N to `✅ Shipped YYYY-MM-DD`.
- If a unilateral implementation decision was made, log it in the Decisions table tagged `(impl)`.
- Update the top-of-FD Status checkboxes (`In-progress`, `Partly implemented`, `Done`) to match reality.
