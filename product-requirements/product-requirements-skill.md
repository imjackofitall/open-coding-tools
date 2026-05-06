---
name: prd-plan
description: Iteratively plan a Product Requirements Document (PRD) with the user via in-file checkbox Q&A, folding answers into the spec until signed off. Use when the user asks to "plan PRD-XXX", "create a PRD for X", "work on PRD-003", "iterate on PRD-XXX", or to turn a rough product idea into a fully specified PRD. PRDs are numbered sequentially (PRD-001, PRD-002, …).
metadata:
  argument-hint: <PRD-XXX id, product name, or plan file path>
---

# PRD Plan Iteration

## Harness role

This skill is a **guide (feedforward control)** in the **behaviour** harness, in the sense used by Birgitta Böckeler in [*Harness Engineering*](https://martinfowler.com/articles/harness-engineering.html). It runs **pre-design**.

- **Computational checks it triggers:** none. PRDs are upstream of code; the deterministic gate is declared later, by the FDs that descend from this PRD.
- **Inferential checks it performs:** structured in-file Q&A on users, scope, data model, UX, and phasing — every question with concrete options and a recommendation.

This skill runs a tight, in-file question-and-answer loop that builds and refines a Product Requirements Document (PRD)
until the user has signed it off. No code is written while this skill is running. The goal is a spec where every product
decision is explicit, every open question is closed, and every user correction has been folded back into the relevant
section(s) of the document — not just recorded as an answer.

A PRD is a complete product spec: who uses it, what it does, what the data model looks like, how the UX works, and how
the build is phased. It is written once, up front, and is the authoritative reference for the entire product build.

## When to use

Invoke this skill when:

- The user says "plan PRD-XXX", "create a PRD for X", "let's work on PRD-XXX", "iterate on PRD-XXX", "flesh out
  PRD-XXX", or similar.
- The user has a rough product idea or an existing `.plans/product-requirements-document/` file they want to fully
  specify.
- The user says "I answered the open questions, review and update" — that's the middle of this loop; resume it.

Do NOT invoke this skill when:

- The user is asking a quick question about an existing PRD. Just read it and answer.
- The user has asked for code changes directly. This skill never writes code.

## PRD document structure

Every PRD lives under `.plans/product-requirements-document/PRD-XXX - {slug}.md`, where `XXX` is a zero-padded
sequential number (PRD-001, PRD-002, …). Check the existing files in `.plans/product-requirements-document/` and pick
the next number. A template lives at `.plans/product-requirements-document/001-TEMPLATE.md` — copy it as the starting
point for a new PRD. A complete PRD has these sections (in order):

```
# PRD-XXX: {Product Name}

## Status
- [x] Open
- [ ] In-progress
- [ ] Partly implemented
- [ ] Done

## Overview
One paragraph: what the product is, who it's for, and the core problem it solves.

## Tech Stack
Framework, database, auth, any third-party services. Match existing monorepo patterns unless there's a reason not to.

## Users
Who uses this? What role(s) do they have? Is there public access? Authentication approach?

## Features
One H3 per major feature area. Each feature area lists what users can do (CRUD verbs, filters, actions). No implementation detail — just capabilities.

## Data Model
One table per entity showing Field, Type, Details. Include enums with their allowed values. Note FK relationships.

## UX Design

### Design Principles
2–4 one-liners that define the feel and priorities of the UI.

### Layout
ASCII wireframe(s) for the primary view(s). One diagram per major screen or state.

### Key Interactions
Named subsections for the most complex or non-obvious interactions (e.g. slide-out panel, inline confirmation, email generator drawer).

## Implementation Plan
Numbered phases, each with a short title and a bullet list of what gets built. Phases should be independently deployable or at least independently testable.

### Model routing
Each Implementation Plan phase includes a Model routing table so the cheapest viable model is picked deliberately rather than by reflex.

Template (paste verbatim into each phase, fill in per-step):

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
- **sonnet** — bounded judgement: apply a pattern the PRD specifies, refactor following a recipe, fix lint errors with
  clear rules, write boilerplate from a spec, classify hits into a fixed set of buckets. The agent makes small calls
  inside well-defined rails.
- **opus** (= top-level session) — design, architecture, risky migrations, cross-cutting changes, synthesising multiple
  inputs, anything where getting it wrong wastes more than the model-cost saving. Don't delegate this.

**Rules:**

- **Each step in the phase gets a row.** No row → no execution. If a step has no row, the Implementation Plan is incomplete.
- **Default towards cheaper.** If you're choosing between sonnet and opus and the work is well-specified by the PRD,
  pick sonnet. Reserve opus for the parts where the PRD doesn't fully prescribe the answer.
- **The escalation line is mandatory.** Sonnet/haiku subagents must know they can return without guessing — they will
  guess otherwise.
- **Re-evaluate after the design step.** Once the inventory/design step (usually opus) is done, the remaining steps are
  often more mechanical than first thought; downgrade them if so.

## Open Questions
Active Q&A area (see format below). Moves to Decision Trail once all answered.

## Decision Trail
Table of resolved decisions (columns: ✅, Question, Decision, Why).
```

## Locating or creating the plan file

1. If the user passed a file path, read it.
2. If they passed an ID (e.g. `PRD-003`, `003`) or slug (e.g. `product-database`), glob for it under
   `.plans/product-requirements-document/` and pick the matching file. Ignore `*TEMPLATE*` files.
3. If no file exists yet:
    - Look at existing `PRD-XXX - *.md` files in `.plans/product-requirements-document/` and pick the next sequential
      number (zero-padded to three digits).
    - Read `.plans/product-requirements-document/001-TEMPLATE.md` and use it as the skeleton.
    - Create the new file at `.plans/product-requirements-document/PRD-XXX - {slug}.md`, filling in what you know from
      the user's description, and leaving sections as `_TBD_` placeholders where you need answers.
4. Confirm the file path and the assigned PRD number in your first reply so the user can correct you before you start
   editing.

## The iteration loop

Each cycle has four beats:

### Beat 1 — Read current state

Read the full PRD file. Identify:

- Which sections are still vague, incomplete, or contradicting each other
- What's answered vs open in the Q&A section
- Any checkboxes the user has ticked or notes they've added since the last read
- Any new requirements the user added directly into prose sections

### Beat 2 — Process user answers

If the user has answered questions since last time:

- Fold each answer into EVERY section of the PRD it affects — not just the questions area. If an answer changes the data
  model, rewrite the Data Model section. If it changes the layout, update the wireframe. If it changes what phases look
  like, update the Implementation Plan.
- A decision is "processed" only when the PRD reads consistently as if the decision was always there. Leaving a resolved
  answer orphaned in the Open Questions area while the main spec still says something contradictory is a failure mode —
  do not do it.
- Watch for answers that contradict things written earlier. Reconcile explicitly; never silently keep the old wording.
- Watch for answers that reveal a project-wide preference (e.g. a locale/spelling rule, a tooling convention). These
  belong in memory via the auto-memory system, not just in this one PRD.

### Beat 3 — Raise new questions

Surface new questions **in the PRD file** — NOT just in chat. The user works through these in their IDE and the PRD is
the durable artefact.

**In-file question format:**

For a question with discrete options, use checkboxes the user can tick directly in the file:

```markdown
#### QX. {short question title}

{One or two sentences explaining the question and why it matters for the product.}

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
  explain the tradeoff in one line.
- **Never ask the user to pick between options you haven't described.** Vague questions like "how should we handle X?"
  are a failure mode — always offer concrete options.
- **Don't ask questions whose answers are already derivable from the codebase.** Read existing apps first. Ask only for
  judgement calls (scope, UX direction, priorities, business rules), not facts you can grep for.
- **Questions have IDs** (Q1, Q1a, Q1b, Q2, ...) so conversation can reference them precisely.
- **Cap one round at ~3 questions.** More than that is a survey, not iteration. Pick the ones that unblock the most
  other decisions first.
- **PRD questions focus on product decisions**, not implementation detail. Good PRD questions: "Should this be
  single-user or multi-role?", "Is archive soft-delete or hard-delete?", "Should filters persist across sessions?". Bad
  PRD questions: "Which framework primitive should we use for this?", "Should this run on the server or the client?"

### Beat 4 — Report back and wait

Send the user a terse message (≤120 words) summarising:

- What you changed in the PRD based on their answers
- What new questions you raised (by ID) and where to find them in the file
- One unilateral decision you made if you had to make one — flag it so they can override

Then stop. The user will either answer the new questions (another cycle begins) or sign off.

## Managing the decision trail

- Under `## Open Questions`, only keep questions that are actually still open (unchecked or partially answered).
- Move resolved questions to `## Decision Trail` as a table: columns `✅ | Question | Decision | Why`. Keep each Why cell
  to one sentence.
- Order the table chronologically — the order carries information about how the design evolved.
- When a question is resolved, fold the answer into the main PRD sections FIRST, then move the question to the decision
  table. Never skip the fold-in step.

## Sign-off gate

You are finished ONLY when:

1. Every question in the file is answered (no unticked checkboxes, no `{your note here}` placeholders the user was
   expected to fill).
2. All PRD sections are complete and self-consistent — Overview, Tech Stack, Users, Features, Data Model, UX Design, and
   Implementation Plan all say consistent things. A developer could read this PRD cold and know what they're building.
3. **Every Implementation Plan phase has a populated Model routing table** — one row per step, model picked deliberately
   from the rubric, escalation line present. See "Model routing" under `## Implementation Plan`. A phase without this
   table isn't signed off, no matter how good the rest looks.
4. The user has explicitly confirmed they want to proceed. Common sign-off phrases: "looks good", "go", "approved", "
   ship it", "done". If the user's latest message doesn't clearly sign off, ask: "Is this PRD ready, or do you want
   another pass?" — one sentence, then stop.

Until all three are true, stay in the loop. Do not write any code. Do not read implementation files unless needed to
answer a planning question.

## Hard rules

- **No code edits during this skill.** Only edits to the PRD file and (where justified) the user's memory system.
- **Always fold answers into the main PRD sections.** Orphaned answers in the Open Questions area while the rest of the
  PRD is stale is the most common failure mode — guard against it.
- **Never hide questions from the user by asking them only in chat.** If it's a product decision, it goes in the file
  with a notes slot. Chat is for terse status updates between cycles.
- **Match the tech stack and patterns of the existing project** unless the user explicitly says otherwise. Read
  `CLAUDE.md` (or equivalent project conventions doc) before raising a tech-stack question.
- **Wireframes are mandatory.** Every PRD must have at least one ASCII wireframe for the primary view before sign-off.
  If none exists after the first cycle, add a skeleton one and ask the user to correct it.
- **Implementation Plan must have phases.** Each phase should be independently testable. If the user hasn't given phase
  breakdown, propose one based on the feature list — it's easier to react than to generate from scratch.
- **Respect project-wide preferences** stored in memory. Default to those preferences; only raise a question if there's
  genuine tension with the product's needs.
- **Keep update messages terse.** The PRD file is the durable artefact. Chat messages between cycles should be ≤120
  words and never repeat content that's already in the file.
- **Ask before guessing.** If an answer is ambiguous, raise it as a new question in the next cycle rather than picking
  unilaterally. If you must pick unilaterally because the decision is tiny and blocks progress, flag it explicitly in
  the chat summary so the user can override.
- **Defer to smaller models for routine reads.** When you need to read a known file, grep for a specific symbol, or
  fetch a single doc page during the planning loop, do it directly with `Read` / `Grep` / `WebFetch`. But anything that
  fans out — exploring an unfamiliar subsystem, finding all call-sites of a symbol, summarising a long doc, comparing
  multiple files — should be delegated to a subagent with `model: "haiku"` (or `"sonnet"` if the task needs reasoning).
  Reserve the top-level Opus session for the synthesis work that actually needs it: reconciling user answers with the
  PRD, judging tradeoffs, deciding question wording. The planning loop is mostly orchestration; don't pay Opus rates
  for grep.
- **Every Implementation Plan phase has a Model routing table.** A phase without per-step model assignments isn't signed
  off, even if every other section is complete. Use the template in "Model routing" under `## Implementation Plan`. The
  sign-off gate enforces this.

## Example cycle

Initial state: user says "plan a PRD for {some new product}". No file exists yet.

Cycle 1:

- Create `.plans/product-requirements-document/PRD-XXX - {slug}.md` with skeleton sections filled from the user's
  description.
- Identify three big unknowns: who the users are, the CRUD scope, and the boundary of one fuzzy feature area.
- Add Q1 (user roles), Q2 (CRUD scope), Q3 (the fuzzy feature's boundary).
- Send chat: "Created PRD skeleton at `.plans/product-requirements-document/PRD-XXX - {slug}.md`. Three questions at
  the bottom — Q1 is the main unlock, it shapes the auth model and most of the UX."
- Stop.

Cycle 2 (user ticks answers and adds notes):

- Read the file.
- Fold answers into Overview, Users, Features, and UX sections. Update the wireframe to reflect the layout the user
  described.
- Raise Q4 (a follow-on decision unblocked by the previous answers).
- Move Q1–Q3 to Decision Trail.
- Send chat: "Folded your answers into sections 3–5, updated wireframe. One new question: Q4 — take a look."
- Stop.

Cycle N (user says "looks good"):

- Verify sign-off gate.
- Confirm in chat: "PRD signed off. Ready to start building when you are."
- Exit the skill.
