---
name: dependency-audit
description: Audit an app's npm dependencies in two phases — security (npm audit, fix path, breaking-change triage) then necessity (keep / replace-native / replace-lighter / remove / consolidate). Use when the user asks to "audit dependencies", "check for vulns", "trim node_modules", "do we still need X", "what can we drop from package.json", or "review deps in apps/<app>". Outputs a markdown report; only modifies package.json/package-lock.json with explicit go-ahead.
metadata:
  argument-hint: <app path, package.json path, or "root" for the workspace root>
---

# Dependency Audit

## Harness role

This skill is a **sensor (feedback control)** in the **maintainability** harness, in the sense used by Birgitta Böckeler in [*Harness Engineering*](https://martinfowler.com/articles/harness-engineering.html). It runs **on cadence** (and pre-release when a release is dependency-sensitive).

- **Computational checks it triggers:** `npm audit --json`, `npm audit fix` (no-force), `npm view <pkg> dist.unpackedSize`, lockfile diff, codebase greps for actual call sites.
- **Inferential checks it performs:** necessity / native-replacement triage, migration-cost judgement, classification of every dep into KEEP / REPLACE / REMOVE / CONSOLIDATE.

This skill audits one project's npm dependencies — security first, necessity second — and produces a single markdown report. It does NOT speculatively upgrade, install, or refactor source code. Phase 1 must finish before Phase 2 starts.

The output is a markdown file at `.plans/dependency-audit/DA-XXX - <scope> - <short topic>.md` (numbered sequentially — find the highest existing `DA-NNN` and increment, start at DA-001 if the folder is empty).

## When to use

Trigger this skill when the user asks any of:

- "Audit our dependencies" / "audit deps in `apps/<app>`"
- "Run `npm audit` and tell me what's actionable"
- "What packages can we drop?" / "Is `<x>` still used?"
- "Are we using anything we could replace with native Node?"
- "Trim `package.json`" / "shrink `node_modules`"

Do NOT trigger when:

- The user wants a single targeted upgrade ("bump Next to 16.2") — just do that.
- The user wants a full lockfile refresh / `npm update` sweep — that's a different ask.
- The user is debugging a specific install failure — fix the install, don't audit.

## Scope resolution

Before running anything, pin the scope:

- If the user named an app (`apps/booking-ui`), audit that app's `package.json` only. The monorepo root is out of scope unless they say so.
- If the user said "root" or didn't specify and there's no obvious target, audit `package.json` at the repo root.
- If `package.json` has `workspaces`, ask which workspace to audit before continuing — auditing all of them at once produces an unreadable report.

If unclear after one read of `package.json`, ask a single in-file question (see format below). Don't guess.

## Phase 1 — Security audit

Run in this order. Do not skip.

1. Run `npm audit --json` (from the correct working directory; in this monorepo run `--workspace=<name>` from the root rather than `cd`-ing into the app dir) and parse the output.
2. Summarise findings grouped by severity (critical, high, moderate, low). For each, list:
   - Package
   - Vulnerability (CVE / advisory ID + one-line description)
   - Fix path (version that resolves it, or "no fix")
   - Direct or transitive — and if transitive, which top-level dep pulls it in
3. Apply non-breaking fixes: run `npm audit fix` (without `--force`). Show the diff to `package.json` and `package-lock.json` and wait for go-ahead before committing.
4. For anything requiring `--force` (breaking change), STOP. Per package, list:
   - Current version → proposed version
   - Breaking changes from the actual changelog / release notes (fetch them; do not guess)
   - Files in this repo that touch the affected API (grep the imports)
   - Risk assessment: low / medium / high, with one-sentence reason
   Wait for go-ahead **per package**. Do not bulk-apply force fixes.
5. If a vulnerability has no fix available, note it, check whether a maintained fork or alternative exists, and flag it for Phase 2.

Phase 1 is done when every advisory is either resolved, queued for user approval, or explicitly carried into Phase 2.

## Phase 2 — Necessity review

Walk every entry in `dependencies` and `devDependencies`. Classify each into exactly one bucket:

### KEEP
Genuinely needed; no reasonable native or lighter alternative. One-sentence justification.

### REPLACE WITH NATIVE
Modern Node / browser APIs cover this now. Common candidates:

| Package                | Native replacement                                          | Note                                                                     |
|------------------------|-------------------------------------------------------------|--------------------------------------------------------------------------|
| `lodash`, `underscore` | ES methods (you-don't-need-lodash patterns)                 | Check actual usage — heavy use is a migration cost flag, not a free swap |
| `moment`, `dayjs`      | `Intl.DateTimeFormat`, `Temporal` (where available), `Date` | `Temporal` polyfill needed if not Node 22+                               |
| `axios`, `node-fetch`  | global `fetch`                                              | Node 18+                                                                 |
| `uuid` (v4)            | `crypto.randomUUID()`                                       | Node 19+                                                                 |
| `dotenv`               | `node --env-file`                                           | Node 20.6+                                                               |
| `chalk`, `colors`      | `util.styleText`                                            | Node 21+; only for simple cases                                          |
| `rimraf`, `mkdirp`     | `fs.rm` / `fs.mkdir` with `{ recursive: true }`             | Node 14+                                                                 |
| `body-parser`          | built into Express 4.16+                                    |                                                                          |
| `qs`, `querystring`    | `URLSearchParams`                                           |                                                                          |
| `glob` (in scripts)    | `fs.glob` (Node 22) or `import.meta.glob` (Vite)            |                                                                          |

For each candidate: grep the codebase for actual usage, list the call sites, and propose a concrete diff. Don't suggest a swap if usage is heavy or non-trivial without flagging the migration cost. **Always verify the project's Node version (`engines`, `.nvmrc`, `.node-version`) supports the native API before recommending it.**

### REPLACE WITH LIGHTER
A meaningfully smaller, better-maintained alternative exists. Cite bundle-size data (bundlephobia or `npm view <pkg> dist.unpackedSize`) and download trend. Don't recommend swaps under ~20% size win unless maintenance health is the driver.

### REMOVE
Unused. Verify by grepping `import` / `require` / dynamic `import()` across the entire repo, including:

- Source files (`.ts`, `.tsx`, `.js`, `.jsx`, `.vue`, `.mjs`, `.cjs`)
- Config files (`vite.config.*`, `next.config.*`, `tailwind.config.*`, `postcss.config.*`, `eslint.config.*`, `.eslintrc.*`, `prettier.config.*`, `vitest.config.*`, `playwright.config.*`, `tsconfig.*`)
- `package.json` `scripts` (CLI usage)
- GitHub Actions / CI workflow files
- Husky / lint-staged configs

Don't trust `package.json` alone. If you can't prove a package is unused, classify it as KEEP with a "usage unclear" note rather than recommending removal.

### CONSOLIDATE
CONSOLIDATE: Multiple packages doing the same job (e.g. both `moment` and `date-fns`, or `axios` and `node-fetch`). Pick one, migrate the other.

## Output format

A single Markdown file at `.plans/dependency-audit/DA-XXX - <scope> - <topic>.md` with these sections in order:

```markdown
# Dependency audit — <scope>

DA-XXX
Scope: <app path or "root">
Date: <YYYY-MM-DD>
Node engine: <from package.json engines / .nvmrc>
Status: <Phase 1 in progress | Phase 1 complete | Phase 2 in progress | done>

## Phase 1 — Security

### Fixed (non-breaking)

{list — package, advisory, version delta}

### Pending user approval (force / breaking)

{one block per package as specified above}

### Unfixable / no upstream fix

{list — package, advisory, mitigation or carry-over to Phase 2}

## Phase 2 — Necessity

| Package | Classification                                                           | Rationale    | Suggested action                | Effort    |
|---------|--------------------------------------------------------------------------|--------------|---------------------------------|-----------|
| ...     | KEEP / REPLACE WITH NATIVE / REPLACE WITH LIGHTER / REMOVE / CONSOLIDATE | one sentence | concrete diff or migration step | S / M / L |

## Open questions

{checkbox blocks using the in-file question format below — only live, unresolved questions}

## Decisions

| ✅   | Question | Decision | Why |
|-----|----------|----------|-----|

## Prioritised action list

1. {highest impact-to-effort first}
2. ...

## Model routing

| Step | Model      | Reason                                                  |
| ---- | ---------- | ------------------------------------------------------- |
| 1    | **haiku**  | {one line — pure mechanical work}                       |
| 2    | **sonnet** | {one line — bounded judgement, well-specified}          |
| 3    | **opus**   | {one line — design / risky / cross-cutting / synthesis} |

If a sonnet/haiku step surfaces a non-trivial decision, escalate to the main session rather than guess.

## Out of scope

{bullets — e.g. "monorepo root deps not audited this round"}
```

### Model routing table (mandatory in every DA report)

The defer-to-smaller-models rule is theoretical until you write it down per step. **Every DA report includes a Model
routing table** (in the `## Model routing` section above) so the cheapest viable model is picked deliberately rather
than by reflex.

Template (shown in the output format above — fill in one row per action-list step):

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
- **sonnet** — bounded judgement: apply a pattern the report specifies, refactor following a recipe, fix lint errors with
  clear rules, write boilerplate from a spec, classify hits into a fixed set of buckets. The agent makes small calls
  inside well-defined rails.
- **opus** (= top-level session) — design, architecture, risky migrations, cross-cutting changes, synthesising multiple
  inputs, anything where getting it wrong wastes more than the model-cost saving. Don't delegate this.

**Rules:**

- **Each step in the prioritised action list gets a row.** No row → no execution. If a step has no row, the report is incomplete.
- **Default towards cheaper.** If you're choosing between sonnet and opus and the work is well-specified by the report,
  pick sonnet. Reserve opus for the parts where the report doesn't fully prescribe the answer.
- **The escalation line is mandatory.** Sonnet/haiku subagents must know they can return without guessing — they will
  guess otherwise.
- **Re-evaluate after the design step.** Once the inventory/design step (usually opus) is done, the remaining steps are
  often more mechanical than first thought; downgrade them if so.

## In-file question format (mandatory)

Use this exact shape for any judgement call — discrete options, one marked recommended, an Other slot, and a notes block. No free-prose questions.

```markdown
#### QX. {short question title}
{One or two sentences explaining the question and why it matters.}

- [ ] Option A — {short description}
- [ ] Option B — {short description} **(recommended — {one-line reason})**
- [ ] Option C — {short description}
- [ ] Other — notes:
  - _{your note here}_

**Notes / reasoning:**
- _{anything else worth recording about this pick}_
```

Cap one round at ~3 questions. Skip anything derivable from the codebase or Node version.

### Common dependency-audit questions

1. **Force-fix risk threshold.** Auto-apply low-risk force fixes, or require approval on every `--force`? (Default: approval on every force fix.)
2. **Native swap aggressiveness.** Recommend native swaps only when usage is ≤5 call sites, or always recommend and let me decide? (Default: always recommend, flag migration cost.)
3. **Lockfile churn tolerance.** Apply transitive-only `npm audit fix` updates that move many lockfile lines, or hold them for a dedicated lockfile-refresh PR? (Default: apply if no breaking changes.)
4. **Workspaces.** Audit only the named app, or include hoisted root devDependencies that affect it? (Default: named app only.)

## Iteration loop

Once the report file exists, run an in-file Q&A loop until sign-off. No code or `package.json` edits beyond what Phase 1 step 3 produces.

### Beat 1 — Read current state
Read the report file end to end. Note ticked checkboxes, inline notes, and any direct edits the user made to the Phase 2 table.

### Beat 2 — Process answers (fold FIRST, then move)
For each resolved question:
1. Fold the answer into every section it affects (Phase 2 classifications, prioritised action list, prose).
2. Append the decision to `## Decisions`.
3. Remove the resolved block from `## Open questions`.

### Beat 3 — Keep progress live
Update `Status:` in the header. In the Phase 2 table, mark rows as `done ✓` once the suggested action has been taken — don't delete them.

### Beat 4 — Raise new questions
New answers usually raise new questions. Surface them **in the file** (not chat-only), capped at ~3 per round.

### Beat 5 — Report back and wait
Send a terse chat message (≤120 words): what was folded, what new questions exist (by ID), and any unilateral pick (flagged so the user can override). Then stop.

## Sign-off gate

Don't declare the audit complete until ALL of:

1. Every Phase 1 advisory is either fixed, explicitly approved-and-applied, or carried over to Phase 2 with a documented reason.
2. Every package in `dependencies` and `devDependencies` has a Phase 2 classification.
3. Every question in `## Open questions` is answered.
4. **The `## Model routing` table is populated** — one row per prioritised action-list step, model picked deliberately
   from the rubric, escalation line present. See "Model routing table (mandatory in every DA report)". An unfilled
   routing table means the report isn't ready to act on.
5. The user has explicitly confirmed ("ship it", "approved", "go ahead").

Until then, stay in the loop.

## Hard rules

- **Don't modify code outside `package.json` and `package-lock.json` without asking.** Native-API swaps require explicit approval per package.
- **Don't run `npm install` or `npm update` speculatively.** Only `npm audit`, `npm audit fix` (no force), and `npm view` for size data without explicit go-ahead.
- **Don't run `npm audit fix --force` without per-package approval.** Bulk force fixes are banned.
- **Verify Node engine before recommending native APIs.** No `node --env-file` advice for a Node 18 project.
- **Cite real call sites.** "Used in 3 places" is not a citation — name the files.
- **Don't recommend a swap on usage > ~20 call sites without flagging migration cost in the Effort column.**
- **Don't classify a package as REMOVE on `package.json` evidence alone.** Grep config + scripts + CI first.
- **Australian English in the report** (colour, organised, analyse, licence, behaviour, prioritise).
- **Never `cd` into app dirs in this monorepo.** Use `npm --workspace=<name> ...` from the root, or the named root scripts.
- **Don't read `.env` files.** If a config check needs an env var, grep `process.env` usage instead.
- **Stay terse between cycles.** The report file is the durable artefact; chat updates are ≤120 words.
- **Defer to smaller models for routine reads.** When you need to read a known file,
  grep for a specific symbol, or fetch a single doc page, do it directly with `Read` /
  `Grep` / `WebFetch`. But anything that fans out — exploring an unfamiliar subsystem,
  finding all call-sites of a symbol, summarising a long doc, comparing multiple files —
  should be delegated to a subagent with `model: "haiku"` (or `"sonnet"` if the task
  needs reasoning). Reserve the top-level Opus session for the synthesis work that
  actually needs it: package selection, migration-cost assessment, writing the final
  report. Don't pay Opus rates for grep.
- **Every DA report has a Model routing table.** A report whose `## Model routing` section is unfilled isn't signed
  off, even if every other section is complete. Use the template in "Model routing table (mandatory in every DA
  report)" above. The sign-off gate enforces this.
