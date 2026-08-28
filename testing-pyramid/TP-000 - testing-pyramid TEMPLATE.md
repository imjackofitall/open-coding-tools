# Test plan / audit — {scope}

## Pyramid shape (current)

{Audit only. Counts per layer + one-sentence diagnosis.}

## Layer rubric

| Layer                                  | What it proves | When to reach for it | Speed     | Example file path                                |
|----------------------------------------|----------------|----------------------|-----------|--------------------------------------------------|
| **Unit** (`tests/unit/`)               |                |                      | <100ms    | `tests/unit/<module>.test.ts`                    |
| **Integration** (`tests/integration/`) |                |                      | 100–500ms | `tests/integration/<area>/<name>.test.ts`        |
| **E2E** (`tests/e2e/`)                 |                |                      | seconds   | `tests/e2e/<flow>.spec.ts`                       |

## Surface area / Findings

### Unit

| Behaviour | Layer | Test file (proposed)        | Notes |
|-----------|-------|-----------------------------|-------|
|           | unit  | `tests/unit/<name>.test.ts` |       |

### Integration

| Behaviour | Layer       | Test file (proposed)                | Notes |
|-----------|-------------|-------------------------------------|-------|
|           | integration | `tests/integration/<name>.test.ts`  |       |

### E2E

| Behaviour | Layer | Test file (proposed)        | Notes |
|-----------|-------|-----------------------------|-------|
|           | e2e   | `tests/e2e/<name>.spec.ts`  |       |

## Open questions

#### Q1. {short question title}

{One or two sentences.}

- [ ] Option A — {description}
- [ ] Option B — {description} **(recommended — {one-line reason})**
- [ ] Other — notes:
  - _{your note here}_

**Notes / reasoning:**
- _{anything else}_

## Decisions

| ✅ | Question | Decision | Why |
|----|----------|----------|-----|

## Files to create / modify

| File | Pass | Type (create/modify/done) | Notes |
|------|------|---------------------------|-------|

## Out of scope

-
