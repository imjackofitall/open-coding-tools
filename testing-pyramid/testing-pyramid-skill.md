---
name: testing-pyramid
description: Plan or audit a project's test coverage against the testing pyramid (unit / integration / e2e). Use when the user asks "what should we test", "is our coverage right", "are we over-testing", "are we missing tests", "what layer should this go in", or wants a review of an existing test suite for bloat or gaps. Outputs a coverage map, layer recommendations, and a concrete edit list — not a fresh test suite.
metadata:
  argument-hint: <feature/area to test, plan ID, or "audit" to review the existing suite>
---

# Testing Pyramid

## Harness role

This skill is a **guide (feedforward control)** in the **behaviour** harness, in the sense used by Birgitta Böckeler in [*Harness Engineering*](https://martinfowler.com/articles/harness-engineering.html). It runs **pre-implementation** (Mode A — Plan) and **post-implementation / on cadence** (Mode B — Audit).

- **Computational checks it produces:** the test plan itself becomes part of the deterministic gate — every behaviour in the surface-area table is a test that runs in the relevant CI tier.
- **Inferential checks it performs:** layer-fit judgement, bloat detection, gap detection, layer-pick decisions (Mode C).

This skill plans or audits a project's test coverage using the testing pyramid as the rubric. It does NOT write tests — it produces a coverage map, identifies gaps and bloat, and outputs a prioritised edit list. Implementation is a separate task.

The pyramid the skill uses:

```
        /\
       /e2e\          few — cross-page flows, real DB, real cookies
      /------\
     / inte-  \       some — route-handler logic, component renders
    / gration  \
   /------------\
  /     unit     \    many — pure functions, validators, reducers, hooks
 /----------------\
```

Heuristic: **if a test would pass with a stubbed-out async result, it doesn't belong in the e2e layer.**

## When to use

Trigger this skill when the user asks any of:

- "What tests should I write for X?"
- "Is our test coverage right?" / "Are we missing tests?"
- "Is this over-tested?" / "Why is CI so slow?"
- "What layer should this assertion go in?"
- "Audit the test suite" / "Review tests in `<path>`"
- "Plan tests for {feature or plan ID}"

Do NOT trigger this skill when:

- The user wants you to actually write the test code (use a normal task; cite this skill's output if one exists).
- The user is debugging a single failing test (just fix it).
- The user wants a code review of a single PR — that's a different task.

## Modes

The skill has three modes. Pick one based on the trigger phrase or ask if ambiguous.

### Mode A — **Plan**: design tests for a new feature / FD

Input: a feature description, a plan-file path, or "the changes on this branch."

Output: a markdown file at `.plans/testing-pyramid/TP-XXX - <scope> - <short topic>.md` (numbered sequentially — find the highest existing `TP-NNN` and increment) containing:

1. **Layer rubric** (the standard table — copy verbatim).
2. **Surface area** — list every behaviour the feature introduces, one row per behaviour, columns: `Behaviour`, `Layer`, `Test file (proposed)`, `Notes`.
3. **Open questions** — only the judgement calls (e.g. "do we mock translations or use the real provider?"). Use the in-file checkbox format defined below.
4. **Files to create / modify** with a Pass column (P1/P2/P3) for ordering.
5. **Out of scope** so the plan stays bounded.

### Mode B — **Audit**: review an existing test suite

Input: a test directory (e.g. `<project>/tests/`) or "this project".

Output: a markdown file at `.plans/testing-pyramid/TP-XXX - <scope> - audit.md` (numbered sequentially — find the highest existing `TP-NNN` and increment) containing:

1. **Pyramid shape** — counts by layer (e.g. unit: 18, integration: 0, e2e: 12) and a sentence on whether the shape is healthy. An inverted or hourglass pyramid is a finding.
2. **Bloat** — tests that are at the wrong layer, duplicates across layers, or assertions that prove nothing the next-layer-up doesn't already prove. Cite specific files and line ranges.
3. **Gaps** — behaviours with no test coverage at any layer, or critical paths covered only by flaky e2e specs.
4. **Flakes** — tests that have been quarantined, marked `.skip`, or have a history of intermittent failures (grep `it.skip`, `test.skip`, `// flaky`, `xit`, `xdescribe`).
5. **Concrete edits** ordered as Pass 1 (must), Pass 2 (should), Pass 3 (nice). Each cites a file path and a one-sentence rationale.

### Mode C — **Layer pick**: which layer for this one test?

Input: a description of one assertion ("does pasting an invite code with too few chars keep the button disabled?").

Output: an inline answer (≤120 words). Layer + the cheapest test that proves it + the file path it would live in. No markdown file written.

## Layer rubric (canonical)

| Layer | What it proves | When to reach for it | Speed | Example file path |
|-------|----------------|----------------------|-------|-------------------|
| **Unit** (`tests/unit/`) | Pure functions; validators; reducers; component branching with mocked deps; small hook logic | Behaviour fits a single module, no real DOM tree, no network. | <100ms | `tests/unit/<module>.test.ts` |
| **Integration** (`tests/integration/`) | Route-handler logic with real validators + mocked clients for external systems; component renders of a single page with mocked hooks; assertion of branching, error states, focus, accessibility | Behaviour spans 2–3 modules, no real browser or live network. | 100–500ms | `tests/integration/route-handlers/<route>.test.ts`, `tests/integration/components/<Component>.test.tsx` |
| **E2E** (`tests/e2e/`) | Cross-page flows touching DOM + cookies + redirects + DB — value is in the *between-pages* movement, not leaf logic | Genuinely end-to-end behaviour. The test would lose its point if you stubbed any single layer. | seconds | `tests/e2e/<flow>.spec.ts` |

## Questions the skill asks while planning

Cap one round at ~3 in-file questions. Skip questions whose answers are derivable from the codebase (config files, existing patterns). Ask only judgement calls.

### In-file question format (mandatory)

Every open question MUST use this exact shape — discrete checkbox options, a recommendation inline on the recommended option, plus an "Other" slot and a notes block. Free-prose questions without options are a failure mode; do not write them.

```markdown
#### QX. {short question title}
{One or two sentences explaining the question and why it matters.}

- [ ] Option A — {short description of what this option means}
- [ ] Option B — {short description} **(recommended — {one-line reason})**
- [ ] Option C — {short description}
- [ ] Other — notes:
  - _{your note here}_

**Notes / reasoning:**
- _{anything else worth recording about this pick}_
```

Rules:
- **Always offer concrete options.** Vague "how should we handle X?" is banned. If you can't think of two real options, the question isn't ready to ask.
- **Mark exactly one option `(recommended — …)`** with a one-line reason. Don't make the user derive your opinion.
- **Every question has an Other + Notes slot** so the user can override or annotate.
- **Questions are numbered** (Q1, Q1a, Q2, …) so chat can reference them.
- **Cap at ~3 per round.** Pick the questions that unblock the most other decisions.

### Common questions for testing plans

Use these as templates — adapt the options to the project, but keep the format above.

1. **i18n / translation strategy.** Mock the translation hook to identity, or wrap renders in the real translation provider with the real message catalogue? (Default: real provider when missing-key bugs would be silent.)
2. **Test runner config split.** One config covering unit + integration, or a sibling `*.integration.config.*`? (Default: split when CI parallelism matters or integration runs are slow.)
3. **Existing-test handling.** Leave existing tests, rename + rewrite in place, or delete and rewrite from scratch? (Default: rewrite in place unless they're entirely about removed behaviour.)
4. **E2E scope.** Smoke (one happy path per surface) or full (every UX branch)? (Default: smoke.)
5. **Outbound HTTP stubbing.** Network-interception library, recorded fixtures, or a contract tool? (Default: a network-level interceptor for consumer apps; contract tools only when you also own the provider.)
6. **DB strategy.** Module-mock the client, run a local DB container, or contract-stub the wire protocol? (Default: module-mock unless you have row-level-security / triggers worth exercising.)

## Bloat detection

Flag a test as bloat in the audit when:

- **Wrong layer.** A browser-driven e2e test that asserts a regex on rendered text from a single component — a component render would prove the same thing in 50ms. ("e2e spec asserting a single heading is visible" is the canonical example.)
- **Multi-layer duplicate.** The same assertion exists at unit + integration + e2e. Pick the cheapest layer that's still meaningful; delete the others.
- **Mock theatre.** A test that mocks every collaborator and asserts the mocks were called with the values you passed in. The behaviour under test is "the function calls its arguments" — delete it.
- **Setup-heavy / assertion-light.** Setup ≥ 5× the assertion lines. Either the test is testing setup, or the unit under test has too many seams. Flag for either deletion or a refactor question.
- **Implementation snapshot.** Snapshot tests on rendered HTML that drift on every legitimate copy change. The signal-to-noise is low; delete unless the snapshot covers a real invariant (token usage, ARIA structure).
- **Coverage-percentage tests.** Tests written purely to bump a coverage percentage with no behavioural claim. Delete.

## Sufficiency detection

Flag a behaviour as under-tested when ALL of these are true:

- It's user-visible OR it crosses a system boundary (HTTP, DB, auth).
- A bug here would be discovered by a user, not by the next test that runs.
- No layer currently asserts the behaviour. (Coverage tools count line execution, not assertions — read the actual tests.)

Critical-path checklist (use as a prompt, not a checklist gate):

- Authentication flow: signin/signup happy path, error path, OAuth callback.
- Authorisation: the cheapest "user A can't see user B's data" test.
- Money path: anything that creates, modifies, or charges a billing record.
- Data write path: the API route that creates the most-queried table row.
- Empty state: every page that has one. Empty states regress silently.
- Redirect chain: any flow with two or more redirects (auth callback is the usual culprit).

## Output format

### Mode A / Mode B output

A single markdown file at `.plans/testing-pyramid/TP-XXX - <app> - <topic|audit>.md` with these sections in order:

```markdown
# Test plan / audit — <subject>

## Pyramid shape (audit only)
{counts + one-sentence diagnosis}

## Layer rubric
{copy the canonical table verbatim}

## Surface area / Findings
{table per the mode's spec}

## Open questions
{checkbox blocks using the in-file question format above — REMOVE blocks once resolved; this section only contains live, unresolved questions}

## Decisions
{table populated as questions resolve — see Decisions table format below}

## Files to create / modify
| File | Pass | Type (create/modify/done) | Notes |

## Model routing

| Step | Model      | Reason                                                  |
| ---- | ---------- | ------------------------------------------------------- |
| 1    | **haiku**  | {one line — pure mechanical work}                       |
| 2    | **sonnet** | {one line — bounded judgement, well-specified}          |
| 3    | **opus**   | {one line — design / risky / cross-cutting / synthesis} |

If a sonnet/haiku step surfaces a non-trivial decision, escalate to the main session rather than guess.

## Out of scope
{bullets}
```

The file is the durable artefact. Chat updates between cycles are ≤120 words.

### Mode C output

Inline answer only. Format:

> **Layer:** <unit | integration | e2e>
> **Why:** <one sentence>
> **File:** `<path>` (existing or new)
> **Why not the next layer up:** <one sentence — what mocks would still apply>

## Model routing table (mandatory in every Mode A / Mode B plan)

The defer-to-smaller-models rule is theoretical until you write it down per step. **Every Mode A and Mode B plan
includes a Model routing table** (in the `## Model routing` section of the output) so the cheapest viable model is
picked deliberately rather than by reflex.

Template (shown in the output format above — fill in one row per pass of the "Files to create / modify" list):

```markdown
## Model routing

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
- **sonnet** — bounded judgement: apply a pattern the plan specifies, refactor following a recipe, fix lint errors with
  clear rules, write boilerplate from a spec, classify hits into a fixed set of buckets. The agent makes small calls
  inside well-defined rails.
- **opus** (= top-level session) — design, architecture, risky migrations, cross-cutting changes, synthesising multiple
  inputs, anything where getting it wrong wastes more than the model-cost saving. Don't delegate this.

**Rules:**

- **Each pass in "Files to create / modify" gets a row.** No row → no execution. If a pass has no row, the plan is incomplete.
- **Default towards cheaper.** If you're choosing between sonnet and opus and the work is well-specified by the plan,
  pick sonnet. Reserve opus for the parts where the plan doesn't fully prescribe the answer.
- **The escalation line is mandatory.** Sonnet/haiku subagents must know they can return without guessing — they will
  guess otherwise.
- **Re-evaluate after the design step.** Once the inventory/design step (usually opus) is done, the remaining steps are
  often more mechanical than first thought; downgrade them if so.

## Iteration loop (Mode A and Mode B)

Once the plan/audit file exists, this skill runs an in-file Q&A loop until the user signs off. No test code is written while the loop is running — the file is the artefact, chat is for terse status only.

### Beat 1 — Read current state
Read the full plan file end to end. Identify:
- Ticked checkboxes and any inline notes the user added under questions
- Any new prose the user inserted into Surface area / Findings / Files-to-modify (a direct edit is a decision too)
- Sections that have drifted from current reality (e.g. file paths that have moved, layer choices the codebase no longer supports)
- The current `## Open questions` and `## Decisions` sections — what's answered, what isn't

### Beat 2 — Process user answers (fold FIRST, then move)
For each question the user has resolved:
1. **Fold the answer into every section of the plan it affects** — Surface area rows, Files to create / modify, layer notes, prose. The plan must read consistently as if the decision was always there. Orphaned answers sitting in `## Open questions` while the body still hedges is the most common failure mode of this skill.
2. **Append the decision to the `## Decisions` table** (format below).
3. **Remove the resolved question block from `## Open questions`** — do not leave ticked checkboxes hanging around. The section should only ever contain *unresolved* questions.

The order matters: fold first, then record, then prune. Skipping the fold-in step leaves the plan contradictory.

Watch for answers that contradict things written in earlier cycles. Reconcile explicitly — never silently keep the old wording.

### Beat 3 — Keep progress live
Update `## Pyramid shape (current)` after each implementation pass — actual file counts per layer plus a one-line note on what's done and what remains.

For the `## Files to create / modify` table, mark rows as you complete them — flip the `Type` column from `create` / `modify` to `done`, or add a `✓` prefix. Don't delete rows; the table is a checklist the user reads to see what's left.

### Beat 4 — Raise new questions
Answers almost always raise new questions. Surface them **in the plan file** using the in-file question format above (NOT only in chat). Cap one round at ~3 questions — pick the ones that unblock the most other decisions. If you have more than three, hold the rest for the next cycle.

### Beat 5 — Report back and wait
Send the user a terse message (≤120 words) covering:
- What was folded in based on their answers
- What new questions were raised (by ID) and where in the file to find them
- One unilateral decision made if any — flag it explicitly so the user can override

Then stop. The user will either answer the new questions (another cycle begins) or sign off.

## Decisions table format

`## Decisions` is a **table**, not a list — columns: `✅`, `Question`, `Decision`, `Why`. Keep each `Why` cell to one sentence so the table scans fast. Order rows by when each question was raised (not alphabetically) — the order itself carries information about how the design evolved.

Example:

```markdown
## Decisions

| ✅ | Question | Decision | Why |
|----|----------|----------|-----|
| ✅ | Q1 — Translation strategy | Wrap component renders in the real translation provider with the real message catalogue | Missing-key bugs would otherwise be silent |
| ✅ | Q2 — Test runner config split | Single config covering unit + integration | CI parallelism not yet a bottleneck |
```

## Sign-off gate (Mode A and Mode B)

Don't declare the plan/audit done until ALL of the following are true:

1. Every question in the file is answered — no unticked checkboxes in `## Open questions`, no `{your note here}` placeholders the user was expected to fill.
2. The plan sections (Surface area / Findings, Files to create / modify, Pyramid shape, etc.) are self-consistent — you could hand the document to someone cold and they could execute it without re-asking the user.
3. **The `## Model routing` table is populated** — one row per pass of the "Files to create / modify" list, model picked
   deliberately from the rubric, escalation line present. See "Model routing table (mandatory in every Mode A / Mode B
   plan)". A plan without this table isn't signed off, no matter how good the rest looks.
4. The user has explicitly confirmed they want to proceed. Common sign-off phrases: "looks good, start writing", "go", "implement", "ship it", "approved". If the user's latest message doesn't clearly sign off, ask: "Is this ready to implement, or do you want another pass?" — one sentence, then stop.

Until all three are true, stay in the loop. Do not start writing test files. Do not even read the implementation source files unless they're needed to answer a planning question.

## Hard rules

- **No code edits.** Test files are written in a separate task; this skill produces the plan only.
- **Cite real file paths.** When recommending a layer or pointing at bloat, name the file. Don't say "some e2e test" — say `tests/e2e/basic-functionality.spec.ts:42`.
- **Don't prescribe coverage percentages.** They're the wrong metric. Talk in behaviours and risk.
- **Don't recommend snapshot tests.** Unless the snapshot is over a structural invariant (ARIA tree, token list) the user can defend.
- **One layer per behaviour.** If the surface-area or findings table lists the same behaviour at two layers, justify it in the Notes column or pick one.
- **Respect project-wide preferences in memory.** If memory says "no fallbacks / no deprecation comments," the audit must flag deprecation-comment cruft in tests as bloat.
- **Always fold answers into the main plan.** Orphaned answers in `## Open questions` while the surface-area / findings table is stale is the most common failure mode of this skill — guard against it.
- **Never hide questions from the user by asking them only in chat.** If it's a decision that shapes the plan, it goes in the file with options + a notes slot. Chat is for terse status updates between cycles.
- **Watch for project-wide feedback while iterating.** If the user shares something that clearly applies beyond this plan ("I prefer X over Y everywhere"), save it to auto-memory in the same cycle it's folded into the plan — don't wait for a separate invitation.
- **Keep update messages terse.** The plan file is the durable artefact. Chat messages between cycles should be ≤120 words and never repeat content that's already in the file.
- **Ask before guessing.** If an answer is ambiguous, raise it as a new question in the next cycle rather than picking unilaterally. If the decision is tiny and blocks progress, flag the unilateral pick explicitly in the chat summary so the user can override.
- **Defer to smaller models for routine reads.** When you need to read a known file,
  grep for a specific symbol, or fetch a single doc page, do it directly with `Read` /
  `Grep` / `WebFetch`. But anything that fans out — exploring an unfamiliar subsystem,
  finding all call-sites of a symbol, summarising a long doc, comparing multiple files —
  should be delegated to a subagent with `model: "haiku"` (or `"sonnet"` if the task
  needs reasoning). Reserve the top-level Opus session for the synthesis work that
  actually needs it: judging test layering, weighing tradeoffs, writing the final plan.
  Don't pay Opus rates for grep.
- **Every Mode A / Mode B plan has a Model routing table.** A plan whose `## Model routing` section is unfilled isn't
  signed off, even if every other section is complete. Use the template in "Model routing table (mandatory in every
  Mode A / Mode B plan)" above. The sign-off gate enforces this.

## Reference: suggested test layout

For a new project, mirror this shape:

```
<project>/
  tests/
    unit/                       # pure logic, validators, hooks (mocked deps)
      *.test.ts
    integration/                # route handlers + component renders
      route-handlers/
        *.test.ts
      components/
        *.test.tsx
    e2e/                        # browser-driven, cross-page only
      *.spec.ts
      global-setup.ts
      helpers/
  <unit-runner>.config.ts        # unit
  <integration-runner>.config.ts # integration (separate so CI can split)
  <e2e-runner>.config.ts
  test-setup.ts
  test-setup-components.ts       # shared render helpers
```

If a project deviates, that deviation is a finding in the audit unless there's a documented reason in the project's
conventions doc.
