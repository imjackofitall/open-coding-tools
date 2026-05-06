# FD-012 — reading-list — Tag filter on the inbox view

## Status

In progress — Phase 1 complete, Phase 2 (filter UI) underway.

## Progress

### Phase 1 — Schema + query plumbing

- [x] Add `article_tags` join table migration
- [x] Backfill from existing comma-separated `tags` column
- [x] Extend `getInboxArticles()` to accept `tagIds: string[]`
- [x] Drop the legacy `articles.tags` text column

### Phase 2 — Filter UI

- [x] `TagFilter` multi-select component
- [ ] Wire `TagFilter` into the inbox page; persist selection in URL params
- [ ] Update empty-state copy when a filter returns zero rows
- [ ] Verification (V2)

## Context

Power users have asked for a way to narrow the inbox to a single topic. Today the only
filter is "unread / archived", and a 200-article inbox is unreadable without grouping.
Tags already exist on the article record; this work makes them filterable.

## Reference

The product behaviour is specified in PRD-002 (reading-list). When this FD disagrees with
the PRD, fix the FD.

## Current State (research findings)

- Tags are stored today as a comma-separated `articles.tags` text column. There are
  ~3,400 articles across all users; the median article has 2 tags, the max has 11.
- `getInboxArticles()` already supports a `search: string` filter via `ilike` on title +
  excerpt — the new filter slots in next to it.
- No existing component does multi-select; the closest is `Combobox` in
  `components/ui/`, which is single-select. Multi-select is new UI.

## Solution Overview

Two phases. Phase 1 is a one-shot schema rewrite; Phase 2 is the user-facing filter.

### Phase 1 — Schema + query plumbing

Replace the comma-separated text column with an `article_tags(article_id, tag_id)` join
table. Backfill once, then drop the legacy column. `getInboxArticles()` grows an optional
`tagIds: string[]` argument; an empty array is treated as "no filter".

### Phase 2 — Filter UI

A `TagFilter` multi-select sits above the inbox list. Selection is persisted in
`?tags=id1,id2` so the URL is shareable and the back button works. Zero-result states
show a "no articles match these tags — clear filter" affordance.

## Data Model

### Migration

```sql
create table public.article_tags (
  article_id uuid not null references public.articles(id) on delete cascade,
  tag_id uuid not null references public.tags(id) on delete cascade,
  primary key (article_id, tag_id)
);

insert into public.article_tags (article_id, tag_id)
select a.id, t.id
from public.articles a
cross join lateral unnest(string_to_array(a.tags, ',')) as raw(name)
join public.tags t on t.name = trim(raw.name) and t.user_id = a.user_id;

alter table public.articles drop column tags;
```

## Acceptance Criteria

#### Scenario: Filter narrows results by tag
- **Given** a user has articles with varying tags
- **When** they select one tag in TagFilter
- **Then** only articles linked to that tag appear in the inbox list

#### Scenario: Multi-tag filter uses OR semantics
- **Given** a user selects two tags
- **When** the filter is applied
- **Then** articles matching either tag are shown (not only articles with both)

#### Scenario: Filter persists across reload
- **Given** a user has selected tags in the filter
- **When** they reload the page
- **Then** the same tags remain selected (URL params survive the reload)

#### Scenario: Tag-load failure is visible
- **Given** the tag service returns an error
- **When** the inbox page loads
- **Then** an inline error banner is shown — not the empty-inbox copy

#### Scenario: Clearing the filter restores full inbox
- **Given** a user has one or more tags active
- **When** they clear the filter
- **Then** all inbox articles are visible again

## Files to Modify

| Area       | Files                                                                                     |
|------------|-------------------------------------------------------------------------------------------|
| migration  | `supabase/migrations/20260112000000_article_tags.sql`                                     |
| db query   | `apps/reading-list/lib/db/articles.ts`, `apps/reading-list/lib/db/tags.ts`               |
| inbox page | `apps/reading-list/app/inbox/page.tsx`                                                    |
| filter UI  | `apps/reading-list/components/TagFilter.tsx` (new), `components/ui/MultiSelect.tsx` (new) |

## Verification

Every risk row below cites a `Vn` test ID.

### V1 — Migration backfill is lossless

- After running the migration, `select count(*) from article_tags` equals the total
  number of comma-separated tag occurrences across the old `articles.tags` column.
- Spot-check three random users: their tag set after migration matches before.

### V2 — Inbox filter narrows results correctly

- Selecting one tag returns only articles linked to that tag.
- Selecting two tags returns articles linked to *either* (OR semantics, per Q1).
- Clearing the filter restores the full inbox.
- URL reflects the selection (`?tags=...`) and survives a page reload.

### V3 — Tag-load failure is visible

- Force `getTags()` to throw; the inbox page renders an inline "couldn't load tags"
  banner, not the empty state.

## Risks & new issues surfaced by this investigation

| ID | Risk                                                                                                    | Mitigation                                                                                              | Verification |
|----|---------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------|--------------|
| R1 | Backfill silently drops tags whose `tags` row was deleted but whose name still appears in `articles.tags` | Pre-flight query lists orphans before the migration runs; if non-zero, create the missing tag rows first | V1           |
| R2 | Tag-load failure renders the same UI as a brand-new account, hiding the failure                         | Inline error banner (resolved by Q2 option B)                                                           | V3           |
| R3 | Per-keystroke search refetch combines badly with the new filter — N tag toggles × M search keystrokes   | Debounce search input at 250ms (existing `useDebouncedValue`)                                           | V2           |

## Open Questions

_(none — signed off, ready to implement Phase 2)_

## Decisions

| ✅ | Question                        | Decision                                            | Why                                                                               |
|----|---------------------------------|-----------------------------------------------------|-----------------------------------------------------------------------------------|
| ✅ | Q1 — Multi-tag filter semantics | OR (article matches any selected tag)               | Matches user expectation from competing readers; AND is a follow-up if asked for  |
| ✅ | Q2 — Tag-load failure UI        | Inline banner with retry, distinct from empty state | Empty state is a real product state; conflating it with "fetch failed" hides bugs |
| ✅ | Q3 — Filter persistence         | URL params (`?tags=...`)                            | Shareable, back-button friendly, no extra storage                                 |

## Status — signed off

User signed off on 2026-01-10. Phase 1 shipped 2026-01-12. Phase 2 in progress.
