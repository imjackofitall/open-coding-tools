# AF-001 — booking-web — characteristics audit

AF-001
Scope: apps/booking-web
Date: 2026-05-06
Status: in progress

## Overview

`booking-web` is the customer-facing Next.js app for the booking flow. The dominant characteristics for this scope are page performance on mobile (LCP-bound conversion impact), keep-the-bundle-small (cold visits on flaky networks), accessibility (regulated industry, AA floor), and layering integrity (server-only modules must not leak into the client bundle). Reliability SLOs sit upstream in the API service and are tracked there.

## Characteristics

| ID  | Characteristic                | Why it matters                                          | Threshold                       | Fitness function                                                              | Where it runs                  | Enforcement state                              | Owner |
|-----|-------------------------------|---------------------------------------------------------|---------------------------------|-------------------------------------------------------------------------------|--------------------------------|------------------------------------------------|-------|
| AF1 | LCP on `/booking/[slug]`      | Conversion drops sharply past 2.5s on mobile.           | LCP ≤ 2.5s p75 (mobile)         | Lighthouse CI assertion: `lighthouse/lighthouserc.json` `categories.performance ≥ 0.85` and `audits.largest-contentful-paint ≤ 2500`. | CI per PR (preview URL) + nightly synthetic on prod. | enforced ✓                                     | Web   |
| AF2 | Route JS budget               | Cold first-load on mid-range mobile is the bottleneck.  | ≤ 180KB gzipped per route chunk | `size-limit` config at `apps/booking-web/.size-limit.cjs`, run in CI.          | pre-push hook + CI per PR.     | enforced ✓                                     | Web   |
| AF3 | Server-only import boundary   | Importing server-only code in client bundle leaks secrets and bloats payload. | zero violations | ESLint `no-restricted-imports` rule on `apps/booking-web/eslint.config.ts` blocking `@/server/**` from `app/**` client components. | pre-commit + CI.               | enforced ✓                                     | Web   |
| AF4 | Accessibility on `/booking/*` | Regulated industry; AA is a hard floor.                 | axe: 0 serious, 0 critical      | Playwright + `@axe-core/playwright` over the booking flow, in `tests/e2e/a11y.spec.ts`. | CI per PR.                     | gap — currently warn-only; flips to fail 2026-06-01. | Web   |
| AF5 | Image weight                  | Hero images dominate first paint.                       | ≤ 220KB total above-the-fold    | Custom `scripts/check-image-budget.ts` reading the `next/image` manifest.      | CI per PR.                     | accepted gap (deferred to 2026-Q3) — flagged but not yet wired. | Web   |
| AF6 | CI wall-clock                 | Slow CI kills flow; >12 min and people stop watching.   | ≤ 12 min p75 over 14 days       | GitHub Actions duration recording + a weekly job that opens an issue on regression. | continuous (GitHub Actions).   | gap — recording exists, issue automation missing. | Web   |

## Open Questions

#### Q1. AF4 enforcement timing

When does the a11y check flip from warn-only to fail-CI? Sooner forces fixes; later avoids merge friction while the team clears the existing violations.

- [ ] Option A — Flip on 2026-05-20 (next sprint end)
- [x] Option B — Flip on 2026-06-01 once the current backlog of 4 serious violations is cleared **(recommended — gives a real deadline without blocking unrelated PRs)**
- [ ] Option C — Flip when violation count hits zero, no fixed date
- [ ] Other — notes:
  - _none_

**Notes / reasoning:**
- _Option C tends to slip; a fixed date is what shipped the bundle budget last quarter._

## Decisions

| ✅ | Question | Decision | Why |
|----|----------|----------|-----|
| ✅ | Should AF1 use Lighthouse score or raw LCP? | Both — score ≥ 0.85 catches regressions broadly; raw LCP ≤ 2500 is the sharper signal. | Score alone hides single-metric regressions; raw alone churns on small jitter. |
| ✅ | Should AF3 use ESLint or dependency-cruiser? | ESLint `no-restricted-imports`. | Already in the local lint pass; dep-cruiser would add a second tool for one rule. |

## Out of Scope

- Backend SLOs — owned by the API service's own AF spec (AF-002, separate scope).
- SEO / metadata characteristics — tracked in the marketing-site fitness spec, not this app.
- Build-cache hit rate — interesting but no current pain; revisit if CI wall-clock breaches AF6.
