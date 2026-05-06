# Dependency audit — apps/booking-web

## Header

- **ID:** DA-003
- **Scope:** apps/booking-web
- **Date:** 2026-04-22
- **Node engine:** 22.4.0 (`.nvmrc`)
- **Status:** done

## Phase 1 — Security

### Fixed (non-breaking)

| Package        | Advisory          | Version delta       | Notes                                                |
|----------------|-------------------|---------------------|------------------------------------------------------|
| `tar-fs`       | GHSA-pq67-2wwv-3xjx | `2.1.1` → `2.1.3` | Transitive via `puppeteer-core`; `npm audit fix` cleared it |
| `semver`       | GHSA-c2qf-rxjj-qqgw | `7.5.4` → `7.6.3` | Direct dev dep                                       |

### Pending user approval (force / breaking)

#### `webpack-dev-server`

- **Current → proposed:** `4.15.1` → `5.2.1`
- **Breaking changes:** dropped Node 14/16 support (we're on 22, fine); `setupMiddlewares` API replaces deprecated `before`/`after` hooks; default host changed from `localhost` to `0.0.0.0`.
- **Affected files in this repo:** `apps/booking-web/dev-server.config.ts` (uses the `before` hook), `package.json` script `dev:proxy`.
- **Risk:** low — one config file, one option to migrate, dev-only dependency. Recommend approve.

### Unfixable / no upstream fix

| Package    | Advisory             | Mitigation or carry-over                                                |
|------------|----------------------|-------------------------------------------------------------------------|
| `request`  | GHSA-p8p7-x288-28g6  | Deprecated; carry to Phase 2 — replace with native `fetch` (Q1).        |

## Phase 2 — Necessity

| Package         | Classification         | Rationale                                                  | Suggested action                                                             | Effort |
|-----------------|------------------------|------------------------------------------------------------|------------------------------------------------------------------------------|--------|
| `next`          | KEEP                   | Framework.                                                 | —                                                                            | —      |
| `react`         | KEEP                   | Framework.                                                 | —                                                                            | —      |
| `axios`         | REPLACE WITH NATIVE    | Used in 4 call sites; Node 22 has global `fetch`.          | Swap to `fetch` (`lib/api/client.ts`, 3 server-action files); see Q2.        | M      |
| `request`       | REMOVE                 | Used only by `scripts/seed-db.ts`; replaced by `fetch`.    | Delete after the seed script is migrated.                                    | S      |
| `lodash`        | REPLACE WITH NATIVE    | 6 call sites, all `groupBy` and `keyBy`. ES `Object.groupBy` is in Node 22. | Inline `Object.groupBy` and `Map.groupBy`; remove dep.                  | S      |
| `dotenv`        | REPLACE WITH NATIVE    | Node 22 has `--env-file`.                                  | Update `dev` and `start` scripts in `package.json`.                          | S      |
| `uuid`          | REPLACE WITH NATIVE    | One call site in `lib/ids.ts`. Node 22 has `crypto.randomUUID`. | Inline; drop dep.                                                       | S      |
| `moment`        | REPLACE WITH LIGHTER   | 22 call sites, three of them timezone-aware. `dayjs` already in tree.  | Migrate remaining `moment` usages to `dayjs`; consolidate. See Q3. | L      |
| `dayjs`         | KEEP (after consolidation) | Lighter, already used in 8 places.                     | Becomes the only date library.                                               | —      |
| `chalk`         | KEEP                   | Used widely in scripts; `util.styleText` doesn't yet cover all our cases. | Revisit when we drop Node 22 support floor.                       | —      |
| `is-odd`        | REMOVE                 | Used in zero places (verified across src, configs, scripts, CI). | Drop.                                                                  | S      |
| `body-parser`   | REMOVE                 | Express 4.18 has it built in; the import was removed in #842 but the dep stayed. | Drop.                                                          | S      |

## Open questions

_All resolved; see Decisions._

## Decisions

| ✅ | Question                                       | Decision                                              | Why                                                                  |
|----|------------------------------------------------|-------------------------------------------------------|----------------------------------------------------------------------|
| ✅ | Q1 — `request` replacement                     | Replace with native `fetch`                           | Node 22; one script's worth of work; closes the unfixable advisory.  |
| ✅ | Q2 — `axios` migration aggressiveness          | Migrate all 4 sites this round                        | All four are simple GETs/POSTs; no interceptors; cheap to do at once.|
| ✅ | Q3 — Date library consolidation timing         | Phase the `moment` → `dayjs` migration over 2 PRs     | 22 call sites is too big for one PR; risk-bounded by phasing.        |
| ✅ | Q4 — `webpack-dev-server` v5 force-fix         | Approve                                               | Dev-only, one config file, low blast radius.                         |

## Prioritised action list

1. Approve `webpack-dev-server` v5 fix and migrate `dev-server.config.ts` (closes Phase 1).
2. Drop `is-odd` and `body-parser` (zero-effort wins).
3. Replace `dotenv` and `uuid` with native APIs (small, mechanical).
4. Replace `lodash` with `Object.groupBy` / `Map.groupBy` and remove dep.
5. Migrate `axios` → `fetch` (one PR for all 4 sites).
6. Migrate `request` usage in the seed script and remove the dep.
7. Phase the `moment` → `dayjs` consolidation across 2 PRs.

## Out of scope

- Monorepo root devDependencies — separate audit (DA-004).
- `apps/admin-web` — not requested this round.
- TypeScript major upgrade — tracked in FD-031, not a dependency-hygiene concern.
