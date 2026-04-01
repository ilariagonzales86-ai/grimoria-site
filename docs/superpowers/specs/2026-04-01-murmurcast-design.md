# MurmurCast Clone — Design Spec

**Date:** 2026-04-01
**Status:** Draft
**Project:** grimoria-os (Next.js) — new `/murmurcast` section

---

## Overview

A personal, local-only AI content digest tool inspired by murmurcast.com. Aggregates YouTube channels, podcast RSS feeds, and newsletters — transcribes audio/video via OpenAI Whisper, summarizes with GPT-4o-mini, delivers a daily email digest, and supports full-text search across all content.

**Scope:** Personal use only. No auth, no multi-tenancy, no billing. Runs locally via `npm run dev`.

---

## Decisions Made

| Question | Answer |
|---|---|
| Features | All 6: YouTube, Podcast, Newsletter, Daily Digest, Search, Quick Analyze |
| Users | Personal use only |
| Deployment | Local (Mac) |
| AI provider | OpenAI — Whisper (transcription) + GPT-4o-mini (summarization) |
| Framework | Integrated into grimoria-os (Next.js) |
| Database | SQLite (better-sqlite3) |
| Approach | Next.js + SQLite Job Queue + Node.js Worker |

---

## Architecture

### Components

```
grimoria-os/
├── app/murmurcast/          ← Next.js route group (UI)
├── lib/murmurcast/          ← Shared logic (db, types, utils)
├── worker/murmurcast.ts     ← Background worker script (run via tsx)
└── murmurcast.db            ← SQLite database (gitignored)
```

### Process Model

A single `npm run dev` starts two processes via `concurrently`. The `package.json` script:

```json
"scripts": {
  "dev": "concurrently \"next dev\" \"tsx watch worker/murmurcast.ts\"",
  "dev:next": "next dev",
  "dev:worker": "tsx watch worker/murmurcast.ts"
}
```

- **Next.js** (`next dev`) — web UI on localhost
- **Worker** (`tsx watch worker/murmurcast.ts`) — polls `jobs` table every 30s

`tsx` is used to run the TypeScript worker directly without a separate build step. Path aliases (`@/lib/...`) work in `tsx` when `tsconfig.json` is present at the project root.

### npm Dependencies

```
better-sqlite3       # SQLite driver (sync, no callback hell)
@types/better-sqlite3  # TypeScript types (devDependency)
tsx                  # TypeScript runner for worker (devDependency)
concurrently         # Run Next.js + worker in parallel (devDependency)
node-cron            # Cron scheduling for daily digest
nodemailer           # Email sending
@types/nodemailer    # TypeScript types (devDependency)
execa                # Shell command execution (yt-dlp, ffmpeg)
openai               # OpenAI SDK (Whisper + GPT)
rss-parser           # Parse podcast RSS feeds
```

### System Dependencies (one-time install)

```bash
brew install yt-dlp ffmpeg
```

### SQLite Configuration

- WAL mode (`PRAGMA journal_mode=WAL`) — allows concurrent reads (Next.js) + writes (Worker)
- FTS5 virtual table populated by SQL triggers on `transcripts` and `summaries`
- Database file: `grimoria-os/murmurcast.db` (added to `.gitignore`)

### API Keys & Secrets

All sensitive config is stored in SQLite `settings` table (plaintext — acceptable for local personal use). No `.env` file for secrets. The Settings UI page provides the input forms. Document what needs to be set in `README.murmurcast.md`.

---

## Database Schema

### `sources`
| Column | Type | Notes |
|---|---|---|
| id | INTEGER PK | |
| type | TEXT | `youtube` \| `podcast` \| `newsletter_rss` \| `newsletter_paste` |
| name | TEXT | Display name |
| url | TEXT | Channel URL / RSS feed URL (NULL for newsletter_paste) |
| config | TEXT | JSON — e.g. `{"youtube_channel_id": "UCxxx", "max_episodes": 5}` |
| active | INTEGER | 0 or 1 |
| last_fetched_at | TEXT | ISO timestamp |
| created_at | TEXT | ISO timestamp |

`newsletter_rss` = has a URL and is fetched like a podcast RSS.
`newsletter_paste` = no URL; content arrives via the paste form (no periodic fetch).

### `episodes`
| Column | Type | Notes |
|---|---|---|
| id | INTEGER PK | |
| source_id | INTEGER FK | → sources.id (NULL for quick-analyze one-offs) |
| external_id | TEXT | YouTube video ID / RSS guid / NULL — used for deduplication |
| title | TEXT | |
| url | TEXT | |
| published_at | TEXT | ISO timestamp |
| duration_s | INTEGER | Duration in seconds (0 if unknown) |
| status | TEXT | `pending` \| `transcribing` \| `transcribed` \| `summarizing` \| `summarized` \| `failed` |
| is_quick_analyze | INTEGER | 1 if created from Quick Analyze, 0 otherwise |
| created_at | TEXT | |

Deduplication: `UNIQUE(source_id, external_id)` — INSERT OR IGNORE on fetch.

### `transcripts`
| Column | Type | Notes |
|---|---|---|
| id | INTEGER PK | |
| episode_id | INTEGER FK | → episodes.id |
| content | TEXT | Full transcript (can be large) |
| model | TEXT | `whisper-1` |
| created_at | TEXT | |

### `summaries`
| Column | Type | Notes |
|---|---|---|
| id | INTEGER PK | |
| episode_id | INTEGER FK | → episodes.id |
| content | TEXT | AI-generated summary |
| key_points | TEXT | JSON array of strings |
| model | TEXT | `gpt-4o-mini` |
| tokens_used | INTEGER | For cost awareness |
| created_at | TEXT | |

### `jobs`
| Column | Type | Notes |
|---|---|---|
| id | INTEGER PK | |
| type | TEXT | `fetch_source` \| `transcribe` \| `summarize` \| `ingest_newsletter` \| `digest` |
| payload | TEXT | JSON (see payload schemas below) |
| status | TEXT | `pending` \| `processing` \| `done` \| `failed` |
| error | TEXT | Error message if failed |
| attempts | INTEGER | Retry counter (max 3) |
| created_at | TEXT | |
| updated_at | TEXT | |
| processed_at | TEXT | ISO timestamp when done/failed |

**Job payload schemas:**
- `fetch_source`: `{ source_id: number }`
- `transcribe`: `{ episode_id: number }`
- `summarize`: `{ episode_id: number }`
- `ingest_newsletter`: `{ source_id: number, content: string, title: string }`
- `digest`: `{ date: string }` — e.g. `"2026-04-01"`

### `digests`
| Column | Type | Notes |
|---|---|---|
| id | INTEGER PK | |
| date | TEXT | `YYYY-MM-DD` UNIQUE |
| content | TEXT | HTML/Markdown digest |
| sent_at | TEXT | NULL if not sent yet |
| episode_ids | TEXT | JSON array of episode IDs included |
| created_at | TEXT | |

### `settings`
| Column | Type | Notes |
|---|---|---|
| key | TEXT PK | Setting name |
| value | TEXT | Setting value |

Default keys:
- `openai_api_key`
- `youtube_api_key`
- `digest_email_to`
- `digest_email_from`
- `smtp_host` / `smtp_port` / `smtp_user` / `smtp_pass`
- `digest_time` (default: `"07:00"`)
- `gpt_model` (default: `"gpt-4o-mini"`)
- `max_episodes_per_fetch` (default: `"5"`)

### `search_index` (FTS5 virtual table)

Regular (non-contentless) FTS5 table — stores indexed text directly. Contentless (`content=''`) is NOT used because it does not support UPDATE, and because this is a personal tool where storage size is not a concern.

**Indexing strategy:** Only the `summaries` INSERT trigger fires. At the time a summary is inserted, the transcript is already present (transcription always completes before summarization). The single trigger joins to `transcripts` to capture both pieces in one INSERT.

```sql
CREATE VIRTUAL TABLE search_index USING fts5(
  episode_id UNINDEXED,
  title,
  summary_content,
  transcript_content
);

-- Single trigger: fires on summary insert (transcript already exists at this point)
-- Joins to transcripts to capture both summary + full transcript in one INSERT
CREATE TRIGGER search_index_after_summary
AFTER INSERT ON summaries BEGIN
  INSERT INTO search_index(episode_id, title, summary_content, transcript_content)
  SELECT NEW.episode_id, e.title, NEW.content, COALESCE(t.content, '')
  FROM episodes e
  LEFT JOIN transcripts t ON t.episode_id = e.id
  WHERE e.id = NEW.episode_id;
END;
```

No separate transcript trigger is needed because transcription always precedes summarization in the job pipeline.

Search query with snippet highlighting:
```sql
SELECT episode_id, title,
  snippet(search_index, 1, '<mark>', '</mark>', '…', 20) AS summary_snippet,
  snippet(search_index, 2, '<mark>', '</mark>', '…', 20) AS transcript_snippet
FROM search_index
WHERE search_index MATCH ?
ORDER BY rank;
```

---

## UI Structure

Route group: `app/murmurcast/`

| Route | Page | Description |
|---|---|---|
| `/murmurcast` | Dashboard | Today's digest, recent summaries feed, job status bar |
| `/murmurcast/sources` | Sources | List/add/toggle YouTube channels, podcast feeds, newsletters |
| `/murmurcast/analyze` | Quick Analyze | Paste YouTube URL → transcribe + summarize on demand |
| `/murmurcast/search` | Search | FTS5 search with snippet highlighting, filters by type/date |
| `/murmurcast/digests` | Digest Archive | Historical daily digests |
| `/murmurcast/settings` | Settings | OpenAI/YouTube API keys, email config, GPT model, fetch limits |

### Quick Analyze Data Model

Quick Analyze creates a standalone `episode` record with `source_id = NULL` and `is_quick_analyze = 1`. It does NOT create a `source`. Results appear in search and can optionally be promoted to a source via "Add to Sources" button.

### Newsletter Manual Paste

The Sources page includes a "Paste newsletter content" form for `newsletter_paste` sources. On submit, a Server Action creates an episode directly and enqueues an `ingest_newsletter` job (which runs summarize only — no audio to transcribe). `transcripts` entry is created with the pasted content as-is.

### Component Patterns

- **Server Components (default):** Dashboard, Sources list, Search results, Digest archive — read SQLite directly via better-sqlite3
- **Client Components (minimal):** Quick Analyze status polling, Search input (debounce), job progress bar
- **Server Actions:** Add source, trigger URL analysis, paste newsletter, save settings, toggle source active
- **Route Handler:** `GET /api/murmurcast/jobs/[id]` — job status polling for Quick Analyze

### Design System

- shadcn/ui components + Tailwind CSS
- Geist Sans / Geist Mono
- Dark mode
- zinc/slate palette, green accent for active/success states

---

## Worker Logic

### Job Claiming (atomic)

```sql
-- Atomic claim with BEGIN IMMEDIATE to prevent double-processing
BEGIN IMMEDIATE;
SELECT id, type, payload FROM jobs
WHERE status = 'pending' AND attempts < 3
ORDER BY created_at ASC LIMIT 1;
-- if found:
UPDATE jobs SET status = 'processing', attempts = attempts + 1, updated_at = now()
WHERE id = ?;
COMMIT;
```

Stale job recovery: on worker startup, reset `status = 'pending'` for any job stuck in `processing` for > 10 minutes (crashed mid-job). Detection uses `updated_at`: `WHERE status = 'processing' AND updated_at < datetime('now', '-10 minutes')`.

### Polling Loop

```
every 30s:
  1. recover stale processing jobs (> 10 min)
  2. claim one pending job
  3. execute job based on type
  4. mark done or failed (with error)
  5. update episode.status accordingly
```

### Episode Status Transitions

```
pending → transcribing  (when transcribe job claimed)
transcribing → transcribed  (when transcribe job done)
transcribed → summarizing  (when summarize job claimed)
summarizing → summarized  (when summarize job done)
any → failed  (when job fails after 3 attempts)
```

### fetch_source Logic

- YouTube: call YouTube Data API with `channelId` from `sources.config`, fetch videos newer than `last_fetched_at` (or last N if first fetch), insert with deduplication (`INSERT OR IGNORE`)
- Podcast RSS: parse feed with `rss-parser`, same incremental logic
- `newsletter_rss`: same as podcast RSS
- `newsletter_paste`: no periodic fetch; handled via `ingest_newsletter` job only

### Whisper File Size Limit (25 MB)

OpenAI Whisper API has a 25 MB limit. Strategy:
- After `yt-dlp` download, check file size
- If > 24 MB: split audio into 20-minute chunks using ffmpeg (`-segment_time 1200`)
- Transcribe each chunk sequentially, concatenate results
- Clean up `/tmp/murmurcast/` after transcription

### Daily Digest Cron

```
node-cron: '0 7 * * *'  (07:00 every day)
→ enqueue digest job for today if not already present
→ worker processes it: collect all summarized episodes from today
→ GPT-4o-mini: generate digest HTML from summaries
→ save to digests table
→ send via Nodemailer (if email configured in settings)
```

Digest is always viewable at `/murmurcast/digests` even without email configured.

---

## Input Validation

All URLs passed to `yt-dlp` and RSS parser must be validated before execution:
- Whitelist schemes: `https://`, `http://`
- For YouTube URLs: validate against known YouTube URL patterns before passing to `yt-dlp`
- Use `execa` (not `exec`) for subprocess calls — arguments are passed as arrays, not shell strings, preventing injection

---

## File Structure

```
grimoria-os/
├── app/
│   └── murmurcast/
│       ├── layout.tsx
│       ├── page.tsx             ← Dashboard
│       ├── sources/page.tsx
│       ├── analyze/page.tsx
│       ├── search/page.tsx
│       ├── digests/page.tsx
│       └── settings/page.tsx
├── lib/
│   └── murmurcast/
│       ├── db.ts                ← better-sqlite3 singleton + schema init + WAL
│       ├── jobs.ts              ← enqueue, claim (atomic), complete, fail, recover
│       ├── sources.ts           ← Source CRUD
│       ├── episodes.ts          ← Episode CRUD
│       ├── transcribe.ts        ← yt-dlp + ffmpeg + Whisper API + chunking
│       ├── summarize.ts         ← GPT-4o-mini summarization
│       ├── digest.ts            ← Digest generation + Nodemailer
│       ├── search.ts            ← FTS5 query + snippet helpers
│       └── settings.ts          ← Settings CRUD from DB
├── worker/
│   └── murmurcast.ts            ← Polling loop + cron + stale recovery
├── app/api/murmurcast/
│   └── jobs/[id]/route.ts       ← Job status polling
├── murmurcast.db                ← SQLite (gitignored)
└── README.murmurcast.md         ← Setup instructions (API keys, brew deps)
```

---

## Out of Scope (v1)

- Multi-user / auth
- Cloud deployment
- Mobile app
- Podcast discovery / recommendations
- Automatic newsletter forwarding via dedicated email address
- Whisper local model (Ollama)
