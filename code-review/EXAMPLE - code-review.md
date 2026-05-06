# Code review — reading-list — tag-filter-on-inbox

## Header

- **ID:** CR-007
- **Scope:** reading-list
- **Change:** tag-filter-on-inbox
- **Date:** 2026-01-14
- **Commit/branch:** `feat/inbox-tag-filter` (uncommitted on top of `9af2c10`)
- **Spec:** FD-012 — reading-list — Tag filter on the inbox view
- **Verdict:** Hold — one major to fix, the rest is minor cleanup. No blockers; behaviour is correct for the happy path.

## Summary

Adds a multi-select tag filter above the inbox list and rewrites `getInboxArticles()` to accept an optional `tagIds: string[]`. The query shape is right and the empty-state copy is good. What I don't love is the per-keystroke refetch on the search box and a quietly wrong fallback when the tag list fails to load.

## Spec compliance

| Scenario                                | Status | Notes                                               |
|-----------------------------------------|--------|-----------------------------------------------------|
| Filter narrows results by tag           | ✅     |                                                     |
| Multi-tag filter uses OR semantics      | ✅     |                                                     |
| Filter persists across reload           | ✅     | URL params wired correctly                          |
| Tag-load failure is visible             | ❌     | Falls through to empty-state copy — see Major below |
| Clearing the filter restores full inbox | —      | Not yet implemented (Phase 2 item)                  |

## Major

- `apps/reading-list/app/inbox/page.tsx:88` — when `getTags()` throws, `tags` is set to `[]` and the filter UI renders "no tags yet" — same string the empty state shows for a brand-new user. A logged-in power user with 40 tags will see the same screen as someone who's never tagged anything, with no way to tell something failed. Either surface the error inline (a one-line banner with a retry) or let the error bubble to the route's error boundary so the user knows to refresh.

## Minor

- `apps/reading-list/app/inbox/page.tsx:112-118` — the search input refetches on every keystroke. Debounce by 250ms (there's already a `useDebouncedValue` hook in `lib/hooks/`) — without it, typing "javascript" fires ten requests.
- `apps/reading-list/lib/db/articles.ts:45` — `tagIds.length === 0 ? undefined : tagIds` is fine but reads oddly next to the other filters which all use `??`. Match the surrounding style.
- `apps/reading-list/components/TagFilter.tsx` — the `Tag` type is redeclared locally instead of imported from `lib/types/tag.ts`. Drop the local copy.

## Nit

- `apps/reading-list/app/inbox/page.tsx` — "Filter by tag(s)" reads awkwardly. "Filter by tag" is fine; the multi-select affordance is obvious from the UI.

## Out of scope but worth noting

- `apps/reading-list/lib/db/articles.ts` has had `getInboxArticles`, `getArchivedArticles`, and `getStarredArticles` accumulate near-identical pagination + ordering code. Worth a follow-up to extract a shared `paginate()` helper — not in this PR.
