# Test plan / audit — reading-list

## Pyramid shape (current)

Unit: 22 (5 files) · Integration: 14 (4 files: 2 route handlers + extraction-client +
tag-filter component) · E2E: 2 hermetic specs. Suite runs in ~1.6s; e2e in ~4s. Healthy
shape — no inversion, no hourglass.

## Layer rubric

| Layer | What it proves | When to reach for it | Speed | Example file path |
|---|---|---|---|---|
| **Unit** (`tests/unit/`) | Pure functions; validators; reducers; component branching with mocked deps; small hook logic | Behaviour fits a single module, no real DOM tree, no network. | <100ms | `tests/unit/url-validator.test.ts` |
| **Integration** (`tests/integration/`) | Route-handler logic with real validators + mocked external clients; component renders of a single page with mocked hooks; assertion of branching, error states, focus, accessibility | Behaviour spans 2–3 modules, no real browser or live network. | 100–500ms | `tests/integration/route-handlers/api-articles-create.test.ts` |
| **E2E** (`tests/e2e/`) | Cross-page flows touching DOM + cookies + DB — value is in the *between-pages* movement | Genuinely end-to-end. The test would lose its point if any layer were stubbed. | seconds | `tests/e2e/save-article-happy-path.spec.ts` |

## Surface area / Findings

### Unit

| Behaviour | Layer | Test file (proposed) | Notes |
|---|---|---|---|
| URL validator (accepts http/https, rejects javascript:, mailto:, empty) | unit | `tests/unit/url-validator.test.ts` | Pure function, no setup |
| Tag-name normaliser (trim, lowercase, max 32 chars, reject empty) | unit | `tests/unit/tag-normaliser.test.ts` | Pure |
| Reading-time formatter (`12 min`, `< 1 min`, `1 hr+`) | unit | `tests/unit/reading-time.test.ts` | Boundary cases at 0, 1, 60 |
| `useDebouncedValue` hook | unit | `tests/unit/use-debounced-value.test.ts` | jsdom env |
| Inbox sort comparator (starred first, then created_at desc) | unit | `tests/unit/inbox-sort.test.ts` | Pure |

### Integration

| Behaviour | Layer | Test file (proposed) | Notes |
|---|---|---|---|
| `POST /api/articles` — happy path, duplicate URL, extraction failure, missing auth | integration | `tests/integration/route-handlers/api-articles-create.test.ts` | Mock the extraction client at the module boundary |
| `GET /api/articles` — tag filter, search, pagination | integration | `tests/integration/route-handlers/api-articles-list.test.ts` | Module-mock the DB client |
| Extraction client — parses well-formed response, surfaces 4xx/5xx, times out at 5s | integration | `tests/integration/extraction-client.test.ts` | Stub the upstream HTTP at the network boundary |
| `<TagFilter>` — toggling chips emits the right query, "clear" resets, error state renders banner | integration | `tests/integration/components/TagFilter.test.tsx` | Real component, mocked `getTags()` |

### E2E (smoke only)

| Behaviour | Layer | Test file (proposed) | Notes |
|---|---|---|---|
| Save an article from the composer → see it in the inbox → mark as read → see it in the archive | e2e | `tests/e2e/save-article-happy-path.spec.ts` | One spec covers the whole save→read loop. Extraction stubbed at the network boundary |
| Magic-link sign-in flow (request link → land on callback → see inbox) | e2e | `tests/e2e/magic-link-sign-in.spec.ts` | Genuinely cross-page; integration can't prove the redirect chain |

Everything else (per-field error messages, the empty-state rendering, "save button is
disabled while extracting") is proven cheaper at integration. Don't add it to e2e.

## Open questions

_None — all resolved._

## Decisions

| ✅ | Question | Decision | Why |
|---|---|---|---|
| ✅ | Q1 — Extraction client stubbing | Network-level interceptor in `tests/msw/handlers.ts` | Same code path as production fetch; doubles as a contract record |
| ✅ | Q2 — DB stubbing strategy | Module-mock the typed client | Cheaper than spinning a real DB; no row-level-security branches worth exercising in tests |
| ✅ | Q3 — Test runner config split | Single config covers unit + integration | CI is fast enough; revisit if integration grows past ~5s |
| ✅ | Q4 — E2E scope | Two specs: save-article happy path + magic-link sign-in | Smoke only; everything else proven at integration |

## Files to create / modify

| File | Pass | Type (create/modify/done) | Notes |
|---|---|---|---|
| `apps/reading-list/package.json` | P1 | ✓ done | Test deps + `test`/`test:unit`/`test:integration` scripts |
| `apps/reading-list/test-setup.ts` | P1 | ✓ done | jest-dom + interceptor lifecycle |
| `apps/reading-list/tests/msw/{server,handlers}.ts` | P1 | ✓ done | Default empty handlers |
| `apps/reading-list/tests/unit/*.test.ts` | P1 | ✓ done | 5 files, 22 tests |
| `apps/reading-list/tests/integration/route-handlers/*.test.ts` | P2 | ✓ done | 2 files, 9 tests |
| `apps/reading-list/tests/integration/extraction-client.test.ts` | P2 | ✓ done | 4 tests, contract for the upstream extractor |
| `apps/reading-list/tests/integration/components/TagFilter.test.tsx` | P2 | ✓ done | 4 tests including the error-banner branch (covers FD-012 R2 / V3) |
| `apps/reading-list/tests/e2e/save-article-happy-path.spec.ts` | P3 | ✓ done | Stubbed extraction; runs against a hermetic DB seed |
| `apps/reading-list/tests/e2e/magic-link-sign-in.spec.ts` | P3 | ✓ done | Magic-link token short-circuited via a test-only env flag |

## Out of scope

- Visual regression / snapshot tests on rendered HTML.
- Tests for the marketing site — separate codebase, separate plan.
- Coverage thresholds in CI. Behaviour-first; add a floor only after the suite settles.
- Real-DB integration against a local Postgres — deferred until row-level-security
  branches exist that are worth exercising.
