# PRD-002: Reading List

## Status
- [x] Open
- [ ] In-progress
- [ ] Partly implemented
- [ ] Done

## Overview

A personal reading-list app for saving articles to read later. Users paste a URL (or use
a browser bookmarklet), the server fetches the article's title + excerpt + estimated
reading time, and the article shows up in their inbox. Tags and an archive let a user
keep a few hundred articles organised without it becoming a swamp. Single-user MVP — no
sharing, no social features.

## Tech Stack

- **Frontend**: a server-rendered web framework with file-system routing and a
  component-based UI library
- **Runtime**: Node.js, deployed on a serverless host
- **Auth**: email magic-link only — no passwords, no OAuth providers in the MVP
- **Storage**: managed Postgres with row-level security; each user only sees their own
  rows
- **External services**: an article-extraction service that takes a URL and returns
  `{ title, excerpt, reading_time, lead_image_url }`

## Users

Single role — **owner**. Each account is one human; there are no shared lists or
collaborators. Anonymous access is limited to the marketing page; everything else is
behind the magic-link sign-in.

## Features

### 1. Save an article

- Save by pasting a URL into the inbox composer
- Save via browser bookmarklet (one-click on the current tab)
- Server fetches title, excerpt, reading time, and lead image
- Duplicate URLs for the same user are rejected with an inline message

### 2. Inbox

- List of unread articles, newest first
- Filter by tag (multi-select, OR semantics)
- Search across title + excerpt
- Mark as read (moves to archive)
- Star (pins to the top of the inbox)
- Delete

### 3. Tags

- Create, rename, delete tags
- Assign multiple tags per article
- Tag colours from a fixed palette (no custom colour picker in MVP)

### 4. Archive

- Read articles live here forever
- Same filters as the inbox
- Restore-to-inbox action

### 5. Account

- Sign in via magic link
- Sign out
- Delete account (cascades all articles + tags)

## Data Model

### user

| Field      | Type        | Details            |
|------------|-------------|--------------------|
| id         | uuid        | PK                 |
| email      | text        | unique, lowercased |
| created_at | timestamptz |                    |

### article

| Field                | Type        | Details                  |
|----------------------|-------------|--------------------------|
| id                   | uuid        | PK                       |
| user_id              | uuid        | FK → user.id             |
| url                  | text        | unique per user          |
| title                | text        |                          |
| excerpt              | text        |                          |
| reading_time_minutes | int         | nullable                 |
| lead_image_url       | text        | nullable                 |
| status               | enum        | `inbox` \| `archived`   |
| starred              | bool        | default false            |
| created_at           | timestamptz |                          |
| read_at              | timestamptz | nullable                 |

### tag

| Field   | Type | Details                                                      |
|---------|------|--------------------------------------------------------------|
| id      | uuid | PK                                                           |
| user_id | uuid | FK → user.id                                                 |
| name    | text | unique per user                                              |
| colour  | enum | `slate` \| `red` \| `amber` \| `green` \| `blue` \| `violet` |

### article_tags

| Field      | Type | Details                            |
|------------|------|------------------------------------|
| article_id | uuid | FK → article.id, on delete cascade |
| tag_id     | uuid | FK → tag.id, on delete cascade     |
|            |      | PK is (article_id, tag_id)         |

## UX Design

### Design Principles

- **Calm** — no notifications, no badges, no streaks. The list is the product.
- **Keyboard-first** — every common action has a shortcut; the mouse is optional.
- **Fast empty** — a fresh inbox renders instantly; the article fetch is async and
  swaps the placeholder card in when it resolves.

### Layout

```
+-------------------------------------------------------------------+
| Reading List                          [search...]   [+ add URL]   |
+----------------+--------------------------------------------------+
| Inbox      (24)| Filter: [tag ▼] [tag ▼]               sort: new  |
| Starred     (3)|--------------------------------------------------|
| Archive   (412)| ☆ The case for slow software         12 min · 2d |
| Tags           |   on engineering culture                          |
|   slate     12 |--------------------------------------------------|
|   green      8 |   How databases handle nulls          7 min · 5d |
|   blue       5 |   on databases                                    |
|                |--------------------------------------------------|
|                |   Why your build is slow              4 min · 1w |
+----------------+--------------------------------------------------+
```

### Key Interactions

- **Add URL** — the composer at the top of the inbox accepts a URL; on submit a
  placeholder card slots in immediately and is replaced when the extraction service
  responds. If extraction fails, the card shows the URL only with a "couldn't extract"
  note.
- **Tag filter** — the tag chips above the list are a multi-select; OR semantics across
  selected tags. Selection persists in the URL (`?tags=...`) so the back button works.
- **Bookmarklet** — saves the current tab's URL, then closes immediately. No
  confirmation popup; the saved article will be in the inbox next time the user opens
  the app.

## Implementation Plan

### Phase 1 — Auth + empty inbox

- Magic-link sign-in
- Empty inbox view
- Account deletion

### Phase 2 — Save + extract

- URL composer
- Article extraction service integration
- Inbox list rendering
- Star, mark-as-read, delete

### Phase 3 — Tags + archive

- Tag CRUD
- Tag assignment from the article card
- Archive view
- Tag filter on inbox + archive

### Phase 4 — Bookmarklet + search

- Bookmarklet hosted at a stable URL
- Title + excerpt search

## Open Questions

_All resolved._

## Decision Trail

| ✅ | Question                                 | Decision               | Why                                                                                                      |
|----|------------------------------------------|------------------------|----------------------------------------------------------------------------------------------------------|
| ✅ | Q1. Auth method?                         | Magic link only        | Removes password reset flows; the user base is single-user-per-account so OAuth provides no team benefit |
| ✅ | Q2. Extraction in-house or third-party?  | Third-party service    | Article extraction is a deep problem; not worth building in MVP                                          |
| ✅ | Q3. Tag colours custom or fixed palette? | Fixed 6-colour palette | Custom colour picker is UX cost we don't need; six is enough to differentiate                            |
| ✅ | Q4. Multi-tag filter semantics?          | OR                     | Matches user expectation; AND can ship later if asked for                                                |
| ✅ | Q5. Sharing / collaboration?             | Out of scope for MVP   | Keeps the schema and the UI simple; revisit after the product has users                                  |
