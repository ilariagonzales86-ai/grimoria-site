# MurmurCast Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a `/murmurcast` section to grimoria-os that aggregates YouTube channels, podcast RSS feeds, and newsletters — transcribes with Whisper, summarizes with GPT-4o-mini, provides full-text search, and sends a daily email digest.

**Architecture:** Next.js Server Components read SQLite directly via better-sqlite3. A Node.js worker (run via `tsx watch`) polls a `jobs` table every 30s for async processing. Both processes start with `npm run dev` via `concurrently`.

**Tech Stack:** Next.js 16.2.1 · better-sqlite3 · OpenAI SDK (already installed as `openai`) · yt-dlp + ffmpeg (brew) · tsx · concurrently · node-cron · nodemailer · execa · rss-parser · vitest

**Spec:** `docs/superpowers/specs/2026-04-01-murmurcast-design.md`

---

> ⚠️ **Before writing any Next.js code:** Read `node_modules/next/dist/docs/` for current API. Next.js 16 has breaking changes. All request APIs are async (`await cookies()`, `await headers()`, `await params`). Middleware renamed to `proxy.ts`.

---

## File Map

New files (all paths relative to `grimoria-os/`):

```
lib/murmurcast/
  db.ts               - SQLite singleton, schema init, WAL mode
  jobs.ts             - Job queue: enqueue, claim (atomic), complete, fail, recover
  settings.ts         - Settings CRUD from settings table
  sources.ts          - Source CRUD
  episodes.ts         - Episode CRUD + status transitions
  transcribe.ts       - yt-dlp + ffmpeg + Whisper API + chunking (uses execa package)
  summarize.ts        - GPT-4o-mini summarization
  search.ts           - FTS5 query + snippet helpers
  digest.ts           - Digest generation + Nodemailer email
  types.ts            - All murmurcast TypeScript types

worker/
  murmurcast.ts       - Polling loop + node-cron + stale job recovery

app/murmurcast/
  layout.tsx          - Sidebar navigation wrapper
  page.tsx            - Dashboard
  sources/page.tsx    - Source management
  analyze/page.tsx    - Quick Analyze (Client Component)
  search/page.tsx     - Full-text search
  digests/page.tsx    - Digest archive
  settings/page.tsx   - API keys, email config

app/api/murmurcast/
  jobs/[id]/route.ts  - GET job status
  analyze/route.ts    - POST create analyze job

tests/murmurcast/
  db.test.ts
  jobs.test.ts
  settings.test.ts
  sources.test.ts
  episodes.test.ts
  search.test.ts
  summarize.test.ts
  digest.test.ts
  # transcribe.ts intentionally has NO unit test — it shells out to yt-dlp/ffmpeg + Whisper API.
  # Verified manually during Task 10 smoke test.
```

Modified files:
```
package.json   - add deps + update "dev" script
.gitignore     - add murmurcast.db
```

---

## Phase 1 — Foundation

### Task 1: Install dependencies + configure tooling

**Files:** `package.json`, `.gitignore`

- [ ] **Step 1: Install runtime dependencies**

```bash
cd grimoria-os
npm install better-sqlite3 node-cron nodemailer execa rss-parser
```

- [ ] **Step 2: Install dev dependencies**

```bash
npm install -D @types/better-sqlite3 @types/nodemailer tsx concurrently vitest @vitest/ui happy-dom
```

- [ ] **Step 3: Update package.json scripts and vitest config**

In `package.json`, replace the `"scripts"` block and add a `"vitest"` config:

```json
"scripts": {
  "dev": "concurrently \"next dev\" \"tsx watch worker/murmurcast.ts\"",
  "dev:next": "next dev",
  "dev:worker": "tsx watch worker/murmurcast.ts",
  "build": "next build",
  "start": "next start",
  "test": "vitest run",
  "test:watch": "vitest"
},
"vitest": {
  "environment": "happy-dom",
  "globals": true,
  "include": ["tests/**/*.test.ts"]
}
```

- [ ] **Step 4: Add murmurcast.db to .gitignore**

Append to `.gitignore`:
```
murmurcast.db
murmurcast.db-wal
murmurcast.db-shm
/tmp/murmurcast/
```

- [ ] **Step 5: Verify vitest works**

```bash
mkdir -p tests/murmurcast
echo 'import { test, expect } from "vitest"; test("sanity", () => expect(1+1).toBe(2));' > tests/murmurcast/sanity.test.ts
npm test
rm tests/murmurcast/sanity.test.ts
```

Expected: `✓ sanity`

- [ ] **Step 6: Commit**

```bash
git add package.json .gitignore
git commit -m "chore(murmurcast): install deps and configure vitest + dev scripts"
```

---

### Task 2: SQLite db layer + types

**Files:** `lib/murmurcast/types.ts`, `lib/murmurcast/db.ts`, `tests/murmurcast/db.test.ts`

- [ ] **Step 1: Write the failing test**

Create `tests/murmurcast/db.test.ts`:

```typescript
import { test, expect, beforeEach, vi } from 'vitest'

beforeEach(() => {
  process.env.MURMURCAST_DB_PATH = ':memory:'
  vi.resetModules()
})

test('creates all tables on init', async () => {
  const { getDb } = await import('@/lib/murmurcast/db')
  const db = getDb()
  const tables = db.prepare(
    "SELECT name FROM sqlite_master WHERE type='table' ORDER BY name"
  ).all() as { name: string }[]
  const names = tables.map(t => t.name)
  ;['sources','episodes','transcripts','summaries','jobs','digests','settings'].forEach(t =>
    expect(names).toContain(t)
  )
})

test('creates search_index FTS5 table', async () => {
  const { getDb } = await import('@/lib/murmurcast/db')
  const db = getDb()
  const r = db.prepare("SELECT name FROM sqlite_master WHERE name='search_index'").get()
  expect(r).toBeTruthy()
})

test('enables WAL mode', async () => {
  const { getDb } = await import('@/lib/murmurcast/db')
  const db = getDb()
  const { journal_mode } = db.prepare('PRAGMA journal_mode').get() as { journal_mode: string }
  expect(journal_mode).toBe('wal')
})

test('singleton: same instance on repeated calls', async () => {
  const { getDb } = await import('@/lib/murmurcast/db')
  expect(getDb()).toBe(getDb())
})
```

- [ ] **Step 2: Run → FAIL**

```bash
npm test -- tests/murmurcast/db.test.ts
```

Expected: FAIL (module not found)

- [ ] **Step 3: Create types.ts**

Create `lib/murmurcast/types.ts`:

```typescript
export type SourceType = 'youtube' | 'podcast' | 'newsletter_rss' | 'newsletter_paste'
export type EpisodeStatus =
  | 'pending' | 'transcribing' | 'transcribed'
  | 'summarizing' | 'summarized' | 'failed'
export type JobType =
  | 'fetch_source' | 'transcribe' | 'summarize'
  | 'ingest_newsletter' | 'digest'
export type JobStatus = 'pending' | 'processing' | 'done' | 'failed'

export interface Source {
  id: number; type: SourceType; name: string; url: string | null
  config: string; active: number; last_fetched_at: string | null; created_at: string
}
export interface Episode {
  id: number; source_id: number | null; external_id: string | null
  title: string; url: string | null; published_at: string | null
  duration_s: number; status: EpisodeStatus; is_quick_analyze: number; created_at: string
}
export interface Transcript {
  id: number; episode_id: number; content: string; model: string; created_at: string
}
export interface Summary {
  id: number; episode_id: number; content: string; key_points: string
  model: string; tokens_used: number; created_at: string
}
export interface Job {
  id: number; type: JobType; payload: string; status: JobStatus
  error: string | null; attempts: number
  created_at: string; updated_at: string; processed_at: string | null
}
export interface Digest {
  id: number; date: string; content: string; sent_at: string | null
  episode_ids: string; created_at: string
}
export interface Setting { key: string; value: string }
```

- [ ] **Step 4: Create db.ts**

Create `lib/murmurcast/db.ts`:

```typescript
import Database from 'better-sqlite3'
import path from 'path'

let _db: Database.Database | null = null

export function getDb(): Database.Database {
  if (_db) return _db
  const dbPath = process.env.MURMURCAST_DB_PATH
    ?? path.join(process.cwd(), 'murmurcast.db')
  _db = new Database(dbPath)
  _db.pragma('journal_mode = WAL')
  _db.pragma('foreign_keys = ON')
  initSchema(_db)
  return _db
}

function initSchema(db: Database.Database): void {
  db.exec(`
    CREATE TABLE IF NOT EXISTS sources (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      type TEXT NOT NULL, name TEXT NOT NULL, url TEXT,
      config TEXT NOT NULL DEFAULT '{}', active INTEGER NOT NULL DEFAULT 1,
      last_fetched_at TEXT, created_at TEXT NOT NULL DEFAULT (datetime('now'))
    );
    CREATE TABLE IF NOT EXISTS episodes (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      source_id INTEGER REFERENCES sources(id),
      external_id TEXT, title TEXT NOT NULL, url TEXT,
      published_at TEXT, duration_s INTEGER NOT NULL DEFAULT 0,
      status TEXT NOT NULL DEFAULT 'pending',
      is_quick_analyze INTEGER NOT NULL DEFAULT 0,
      created_at TEXT NOT NULL DEFAULT (datetime('now')),
      UNIQUE(source_id, external_id)
    );
    CREATE TABLE IF NOT EXISTS transcripts (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      episode_id INTEGER NOT NULL REFERENCES episodes(id),
      content TEXT NOT NULL, model TEXT NOT NULL DEFAULT 'whisper-1',
      created_at TEXT NOT NULL DEFAULT (datetime('now'))
    );
    CREATE TABLE IF NOT EXISTS summaries (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      episode_id INTEGER NOT NULL REFERENCES episodes(id),
      content TEXT NOT NULL, key_points TEXT NOT NULL DEFAULT '[]',
      model TEXT NOT NULL DEFAULT 'gpt-4o-mini',
      tokens_used INTEGER NOT NULL DEFAULT 0,
      created_at TEXT NOT NULL DEFAULT (datetime('now'))
    );
    CREATE TABLE IF NOT EXISTS jobs (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      type TEXT NOT NULL, payload TEXT NOT NULL DEFAULT '{}',
      status TEXT NOT NULL DEFAULT 'pending', error TEXT,
      attempts INTEGER NOT NULL DEFAULT 0,
      created_at TEXT NOT NULL DEFAULT (datetime('now')),
      updated_at TEXT NOT NULL DEFAULT (datetime('now')),
      processed_at TEXT
    );
    CREATE TABLE IF NOT EXISTS digests (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      date TEXT NOT NULL UNIQUE, content TEXT NOT NULL,
      sent_at TEXT, episode_ids TEXT NOT NULL DEFAULT '[]',
      created_at TEXT NOT NULL DEFAULT (datetime('now'))
    );
    CREATE TABLE IF NOT EXISTS settings (
      key TEXT PRIMARY KEY, value TEXT NOT NULL DEFAULT ''
    );
    CREATE VIRTUAL TABLE IF NOT EXISTS search_index USING fts5(
      episode_id UNINDEXED, title, summary_content, transcript_content
    );
    CREATE TRIGGER IF NOT EXISTS search_index_after_summary
    AFTER INSERT ON summaries BEGIN
      INSERT INTO search_index(episode_id, title, summary_content, transcript_content)
      SELECT NEW.episode_id, e.title, NEW.content, COALESCE(t.content, '')
      FROM episodes e
      LEFT JOIN transcripts t ON t.episode_id = e.id
      WHERE e.id = NEW.episode_id;
    END;
  `)
}
```

- [ ] **Step 5: Run → PASS**

```bash
npm test -- tests/murmurcast/db.test.ts
```

Expected: `✓ 4 tests`

- [ ] **Step 6: Commit**

```bash
git add lib/murmurcast/db.ts lib/murmurcast/types.ts tests/murmurcast/db.test.ts
git commit -m "feat(murmurcast): SQLite schema, WAL mode, FTS5 trigger"
```

---

### Task 3: Job queue

**Files:** `lib/murmurcast/jobs.ts`, `tests/murmurcast/jobs.test.ts`

- [ ] **Step 1: Write failing tests**

Create `tests/murmurcast/jobs.test.ts`:

```typescript
import { test, expect, beforeEach } from 'vitest'

beforeEach(() => { process.env.MURMURCAST_DB_PATH = ':memory:'; vi.resetModules() })

test('enqueueJob inserts pending job', async () => {
  const { getDb } = await import('@/lib/murmurcast/db')
  const { enqueueJob } = await import('@/lib/murmurcast/jobs')
  const db = getDb()
  const id = enqueueJob(db, 'transcribe', { episode_id: 1 })
  const job = db.prepare('SELECT * FROM jobs WHERE id = ?').get(id) as any
  expect(job.type).toBe('transcribe')
  expect(job.status).toBe('pending')
})

test('claimJob returns null when empty', async () => {
  const { getDb } = await import('@/lib/murmurcast/db')
  const { claimJob } = await import('@/lib/murmurcast/jobs')
  expect(claimJob(getDb())).toBeNull()
})

test('claimJob claims atomically and increments attempts', async () => {
  const { getDb } = await import('@/lib/murmurcast/db')
  const { enqueueJob, claimJob } = await import('@/lib/murmurcast/jobs')
  const db = getDb()
  enqueueJob(db, 'summarize', { episode_id: 2 })
  const job = claimJob(db)
  expect(job?.status).toBe('processing')
  expect(job?.attempts).toBe(1)
})

test('claimJob skips jobs with 3+ attempts', async () => {
  const { getDb } = await import('@/lib/murmurcast/db')
  const { enqueueJob, claimJob } = await import('@/lib/murmurcast/jobs')
  const db = getDb()
  const id = enqueueJob(db, 'transcribe', { episode_id: 1 })
  db.prepare('UPDATE jobs SET attempts = 3 WHERE id = ?').run(id)
  expect(claimJob(db)).toBeNull()
})

test('completeJob sets status done with timestamp', async () => {
  const { getDb } = await import('@/lib/murmurcast/db')
  const { enqueueJob, claimJob, completeJob } = await import('@/lib/murmurcast/jobs')
  const db = getDb()
  const id = enqueueJob(db, 'digest', { date: '2026-04-01' })
  claimJob(db)
  completeJob(db, id)
  const j = db.prepare('SELECT * FROM jobs WHERE id = ?').get(id) as any
  expect(j.status).toBe('done')
  expect(j.processed_at).not.toBeNull()
})

test('failJob sets status failed with error', async () => {
  const { getDb } = await import('@/lib/murmurcast/db')
  const { enqueueJob, claimJob, failJob } = await import('@/lib/murmurcast/jobs')
  const db = getDb()
  const id = enqueueJob(db, 'transcribe', { episode_id: 1 })
  claimJob(db)
  failJob(db, id, 'Whisper error')
  const j = db.prepare('SELECT * FROM jobs WHERE id = ?').get(id) as any
  expect(j.status).toBe('failed')
  expect(j.error).toBe('Whisper error')
})

test('recoverStaleJobs resets stuck processing jobs', async () => {
  const { getDb } = await import('@/lib/murmurcast/db')
  const { enqueueJob, recoverStaleJobs } = await import('@/lib/murmurcast/jobs')
  const db = getDb()
  const id = enqueueJob(db, 'transcribe', { episode_id: 1 })
  db.prepare(
    "UPDATE jobs SET status='processing', updated_at=datetime('now','-15 minutes') WHERE id=?"
  ).run(id)
  recoverStaleJobs(db)
  const j = db.prepare('SELECT * FROM jobs WHERE id = ?').get(id) as any
  expect(j.status).toBe('pending')
})
```

- [ ] **Step 2: Run → FAIL**

```bash
npm test -- tests/murmurcast/jobs.test.ts
```

- [ ] **Step 3: Implement jobs.ts**

Create `lib/murmurcast/jobs.ts`:

```typescript
import type Database from 'better-sqlite3'
import type { Job, JobType } from './types'

export function enqueueJob(
  db: Database.Database, type: JobType, payload: Record<string, unknown>
): number {
  return (db.prepare('INSERT INTO jobs (type, payload) VALUES (?, ?)').run(
    type, JSON.stringify(payload)
  ).lastInsertRowid) as number
}

export function claimJob(db: Database.Database): Job | null {
  return db.transaction((): Job | null => {
    const job = db.prepare(
      "SELECT * FROM jobs WHERE status='pending' AND attempts<3 ORDER BY created_at ASC LIMIT 1"
    ).get() as Job | undefined
    if (!job) return null
    db.prepare(
      "UPDATE jobs SET status='processing', attempts=attempts+1, updated_at=datetime('now') WHERE id=?"
    ).run(job.id)
    return { ...job, status: 'processing', attempts: job.attempts + 1 }
  })()
}

export function completeJob(db: Database.Database, id: number): void {
  db.prepare(
    "UPDATE jobs SET status='done', updated_at=datetime('now'), processed_at=datetime('now') WHERE id=?"
  ).run(id)
}

export function failJob(db: Database.Database, id: number, error: string): void {
  db.prepare(
    "UPDATE jobs SET status='failed', error=?, updated_at=datetime('now'), processed_at=datetime('now') WHERE id=?"
  ).run(error, id)
}

export function recoverStaleJobs(db: Database.Database): void {
  db.prepare(
    "UPDATE jobs SET status='pending', updated_at=datetime('now') WHERE status='processing' AND updated_at<datetime('now','-10 minutes')"
  ).run()
}
```

- [ ] **Step 4: Run → PASS**

```bash
npm test -- tests/murmurcast/jobs.test.ts
```

Expected: `✓ 7 tests`

- [ ] **Step 5: Commit**

```bash
git add lib/murmurcast/jobs.ts tests/murmurcast/jobs.test.ts
git commit -m "feat(murmurcast): atomic job queue with BEGIN IMMEDIATE transaction"
```

---

### Task 4: Settings

**Files:** `lib/murmurcast/settings.ts`, `tests/murmurcast/settings.test.ts`

- [ ] **Step 1: Write failing tests**

Create `tests/murmurcast/settings.test.ts`:

```typescript
import { test, expect, beforeEach, vi } from 'vitest'
beforeEach(() => { process.env.MURMURCAST_DB_PATH = ':memory:'; vi.resetModules() })

test('getSetting returns null for missing key', async () => {
  const { getDb } = await import('@/lib/murmurcast/db')
  const { getSetting } = await import('@/lib/murmurcast/settings')
  expect(getSetting(getDb(), 'missing')).toBeNull()
})
test('setSetting + getSetting round-trip', async () => {
  const { getDb } = await import('@/lib/murmurcast/db')
  const { getSetting, setSetting } = await import('@/lib/murmurcast/settings')
  const db = getDb()
  setSetting(db, 'openai_api_key', 'sk-test')
  expect(getSetting(db, 'openai_api_key')).toBe('sk-test')
})
test('setSetting updates existing key', async () => {
  const { getDb } = await import('@/lib/murmurcast/db')
  const { getSetting, setSetting } = await import('@/lib/murmurcast/settings')
  const db = getDb()
  setSetting(db, 'model', 'gpt-4o-mini')
  setSetting(db, 'model', 'gpt-4o')
  expect(getSetting(db, 'model')).toBe('gpt-4o')
})
test('getAllSettings returns all keys', async () => {
  const { getDb } = await import('@/lib/murmurcast/db')
  const { setSetting, getAllSettings } = await import('@/lib/murmurcast/settings')
  const db = getDb()
  setSetting(db, 'a', '1'); setSetting(db, 'b', '2')
  const all = getAllSettings(db)
  expect(all['a']).toBe('1'); expect(all['b']).toBe('2')
})
```

- [ ] **Step 2: Run → FAIL, implement, run → PASS**

Implement `lib/murmurcast/settings.ts`:

```typescript
import type Database from 'better-sqlite3'

export function getSetting(db: Database.Database, key: string): string | null {
  return ((db.prepare('SELECT value FROM settings WHERE key=?').get(key) as { value: string } | undefined)?.value) ?? null
}
export function setSetting(db: Database.Database, key: string, value: string): void {
  db.prepare('INSERT OR REPLACE INTO settings (key,value) VALUES (?,?)').run(key, value)
}
export function getAllSettings(db: Database.Database): Record<string, string> {
  return Object.fromEntries(
    (db.prepare('SELECT key,value FROM settings').all() as { key: string; value: string }[])
      .map(r => [r.key, r.value])
  )
}
```

```bash
npm test -- tests/murmurcast/settings.test.ts
```

Expected: `✓ 4 tests`

- [ ] **Step 3: Commit**

```bash
git add lib/murmurcast/settings.ts tests/murmurcast/settings.test.ts
git commit -m "feat(murmurcast): settings CRUD"
```

---

## Phase 2 — Sources, Episodes, Search

### Task 5: Sources + Episodes CRUD

**Files:** `lib/murmurcast/sources.ts`, `lib/murmurcast/episodes.ts`, `tests/murmurcast/sources.test.ts`, `tests/murmurcast/episodes.test.ts`

- [ ] **Step 1: Write failing tests for sources**

Create `tests/murmurcast/sources.test.ts`:

```typescript
import { test, expect, beforeEach, vi } from 'vitest'
beforeEach(() => { process.env.MURMURCAST_DB_PATH = ':memory:'; vi.resetModules() })

test('createSource inserts a source', async () => {
  const { getDb } = await import('@/lib/murmurcast/db')
  const { createSource, getSources } = await import('@/lib/murmurcast/sources')
  const db = getDb()
  createSource(db, { type: 'youtube', name: 'Test', url: 'https://youtube.com/@test', config: {} })
  expect(getSources(db)).toHaveLength(1)
})
test('getSources excludes inactive by default', async () => {
  const { getDb } = await import('@/lib/murmurcast/db')
  const { createSource, getSources, toggleSource } = await import('@/lib/murmurcast/sources')
  const db = getDb()
  const id = createSource(db, { type: 'podcast', name: 'P', url: 'u', config: {} })
  toggleSource(db, id, false)
  expect(getSources(db)).toHaveLength(0)
  expect(getSources(db, { includeInactive: true })).toHaveLength(1)
})
```

- [ ] **Step 2: Write failing tests for episodes**

Create `tests/murmurcast/episodes.test.ts`:

```typescript
import { test, expect, beforeEach, vi } from 'vitest'
beforeEach(() => { process.env.MURMURCAST_DB_PATH = ':memory:'; vi.resetModules() })

test('createEpisode inserts and getEpisodeById returns it', async () => {
  const { getDb } = await import('@/lib/murmurcast/db')
  const { createSource } = await import('@/lib/murmurcast/sources')
  const { createEpisode, getEpisodeById } = await import('@/lib/murmurcast/episodes')
  const db = getDb()
  const sid = createSource(db, { type: 'youtube', name: 'C', url: 'u', config: {} })
  const id = createEpisode(db, { source_id: sid, external_id: 'abc', title: 'Ep1', url: 'u' })
  expect(getEpisodeById(db, id)?.title).toBe('Ep1')
})
test('createEpisode ignores duplicate external_id', async () => {
  const { getDb } = await import('@/lib/murmurcast/db')
  const { createSource } = await import('@/lib/murmurcast/sources')
  const { createEpisode, getRecentEpisodes } = await import('@/lib/murmurcast/episodes')
  const db = getDb()
  const sid = createSource(db, { type: 'youtube', name: 'C', url: 'u', config: {} })
  createEpisode(db, { source_id: sid, external_id: 'same', title: 'E1', url: 'u' })
  createEpisode(db, { source_id: sid, external_id: 'same', title: 'E2', url: 'u' })
  expect(getRecentEpisodes(db, 10)).toHaveLength(1)
})
test('updateEpisodeStatus transitions', async () => {
  const { getDb } = await import('@/lib/murmurcast/db')
  const { createSource } = await import('@/lib/murmurcast/sources')
  const { createEpisode, updateEpisodeStatus, getEpisodeById } = await import('@/lib/murmurcast/episodes')
  const db = getDb()
  const sid = createSource(db, { type: 'youtube', name: 'C', url: 'u', config: {} })
  const id = createEpisode(db, { source_id: sid, external_id: 'x', title: 'E', url: 'u' })
  updateEpisodeStatus(db, id, 'transcribing')
  expect(getEpisodeById(db, id)?.status).toBe('transcribing')
})
test('newsletter_paste episode stores pasted content as transcript', async () => {
  // Verifies the ingest_newsletter path: transcript row is created with pasted content
  const { getDb } = await import('@/lib/murmurcast/db')
  const { createSource } = await import('@/lib/murmurcast/sources')
  const { createEpisode } = await import('@/lib/murmurcast/episodes')
  const db = getDb()
  const sid = createSource(db, { type: 'newsletter_paste', name: 'NL', url: null, config: {} })
  const id = createEpisode(db, { source_id: sid, external_id: null, title: 'Issue #1', url: null })
  db.prepare('INSERT INTO transcripts (episode_id, content, model) VALUES (?,?,?)').run(id, 'Pasted content here', 'paste')
  const t = db.prepare('SELECT content FROM transcripts WHERE episode_id=?').get(id) as { content: string }
  expect(t.content).toBe('Pasted content here')
})
```

- [ ] **Step 3: Implement sources.ts**

Create `lib/murmurcast/sources.ts`:

```typescript
import type Database from 'better-sqlite3'
import type { Source, SourceType } from './types'

interface CreateSourceInput {
  type: SourceType; name: string; url: string | null
  config: Record<string, unknown>
}

export function createSource(db: Database.Database, i: CreateSourceInput): number {
  return db.prepare('INSERT INTO sources (type,name,url,config) VALUES (?,?,?,?)')
    .run(i.type, i.name, i.url, JSON.stringify(i.config)).lastInsertRowid as number
}
export function getSources(db: Database.Database, opts: { includeInactive?: boolean } = {}): Source[] {
  return db.prepare(opts.includeInactive
    ? 'SELECT * FROM sources ORDER BY created_at DESC'
    : 'SELECT * FROM sources WHERE active=1 ORDER BY created_at DESC'
  ).all() as Source[]
}
export function getSourceById(db: Database.Database, id: number): Source | null {
  return (db.prepare('SELECT * FROM sources WHERE id=?').get(id) as Source) ?? null
}
export function toggleSource(db: Database.Database, id: number, active: boolean): void {
  db.prepare('UPDATE sources SET active=? WHERE id=?').run(active ? 1 : 0, id)
}
export function updateLastFetched(db: Database.Database, id: number): void {
  db.prepare("UPDATE sources SET last_fetched_at=datetime('now') WHERE id=?").run(id)
}
```

- [ ] **Step 4: Implement episodes.ts**

Create `lib/murmurcast/episodes.ts`:

```typescript
import type Database from 'better-sqlite3'
import type { Episode, EpisodeStatus } from './types'

interface CreateEpisodeInput {
  source_id: number | null; external_id: string | null
  title: string; url: string | null
  published_at?: string; duration_s?: number; is_quick_analyze?: boolean
}

export function createEpisode(db: Database.Database, i: CreateEpisodeInput): number {
  const r = db.prepare(
    'INSERT OR IGNORE INTO episodes (source_id,external_id,title,url,published_at,duration_s,is_quick_analyze) VALUES (?,?,?,?,?,?,?)'
  ).run(i.source_id, i.external_id, i.title, i.url, i.published_at ?? null, i.duration_s ?? 0, i.is_quick_analyze ? 1 : 0)
  if (r.changes === 0) {
    const e = db.prepare('SELECT id FROM episodes WHERE source_id=? AND external_id=?')
      .get(i.source_id, i.external_id) as { id: number } | undefined
    return e?.id ?? 0
  }
  return r.lastInsertRowid as number
}
export function getEpisodeById(db: Database.Database, id: number): Episode | null {
  return (db.prepare('SELECT * FROM episodes WHERE id=?').get(id) as Episode) ?? null
}
export function getRecentEpisodes(db: Database.Database, limit: number): Episode[] {
  return db.prepare('SELECT * FROM episodes ORDER BY created_at DESC LIMIT ?').all(limit) as Episode[]
}
export function updateEpisodeStatus(db: Database.Database, id: number, status: EpisodeStatus): void {
  db.prepare('UPDATE episodes SET status=? WHERE id=?').run(status, id)
}
```

- [ ] **Step 5: Run → PASS**

```bash
npm test -- tests/murmurcast/sources.test.ts tests/murmurcast/episodes.test.ts
```

Expected: `✓ 5 tests`

- [ ] **Step 6: Commit**

```bash
git add lib/murmurcast/sources.ts lib/murmurcast/episodes.ts \
        tests/murmurcast/sources.test.ts tests/murmurcast/episodes.test.ts
git commit -m "feat(murmurcast): sources and episodes CRUD with deduplication"
```

---

### Task 6: Search (FTS5)

**Files:** `lib/murmurcast/search.ts`, `tests/murmurcast/search.test.ts`

- [ ] **Step 1: Write failing tests**

Create `tests/murmurcast/search.test.ts`:

```typescript
import { test, expect, beforeEach, vi } from 'vitest'
beforeEach(() => { process.env.MURMURCAST_DB_PATH = ':memory:'; vi.resetModules() })

function seedEp(db: any, sourceId: number, title: string, summary: string, transcript: string) {
  const eid = db.prepare(
    'INSERT INTO episodes (source_id,external_id,title,url) VALUES (?,?,?,?)'
  ).run(sourceId, title, title, null).lastInsertRowid as number
  db.prepare('INSERT INTO transcripts (episode_id,content) VALUES (?,?)').run(eid, transcript)
  db.prepare('INSERT INTO summaries (episode_id,content,key_points) VALUES (?,?,?)').run(eid, summary, '[]')
  return eid
}

test('searchContent finds matching episode', async () => {
  const { getDb } = await import('@/lib/murmurcast/db')
  const { createSource } = await import('@/lib/murmurcast/sources')
  const { searchContent } = await import('@/lib/murmurcast/search')
  const db = getDb()
  const sid = createSource(db, { type: 'youtube', name: 'C', url: 'u', config: {} })
  seedEp(db, sid, 'AI future', 'AI is transforming', 'artificial intelligence')
  seedEp(db, sid, 'Cooking pasta', 'How to cook pasta', 'boil water')
  const results = searchContent(db, 'AI')
  expect(results).toHaveLength(1)
  expect(results[0].title).toBe('AI future')
})
test('searchContent returns empty on no match', async () => {
  const { getDb } = await import('@/lib/murmurcast/db')
  const { searchContent } = await import('@/lib/murmurcast/search')
  expect(searchContent(getDb(), 'xyznotfound99')).toHaveLength(0)
})
test('searchContent result has snippet', async () => {
  const { getDb } = await import('@/lib/murmurcast/db')
  const { createSource } = await import('@/lib/murmurcast/sources')
  const { searchContent } = await import('@/lib/murmurcast/search')
  const db = getDb()
  const sid = createSource(db, { type: 'youtube', name: 'C', url: 'u', config: {} })
  seedEp(db, sid, 'Test', 'summary with keyword here', 'transcript')
  const r = searchContent(db, 'keyword')
  expect(r[0].summary_snippet).toContain('keyword')
})
```

- [ ] **Step 2: Run → FAIL, implement, run → PASS**

Create `lib/murmurcast/search.ts`:

```typescript
import type Database from 'better-sqlite3'

export interface SearchResult {
  episode_id: number; title: string
  summary_snippet: string; transcript_snippet: string
}

export function searchContent(db: Database.Database, query: string, limit = 20): SearchResult[] {
  if (!query.trim()) return []
  try {
    // FTS5 column indexes: 0=episode_id (UNINDEXED), 1=title, 2=summary_content, 3=transcript_content
    return db.prepare(`
      SELECT episode_id, title,
        snippet(search_index,2,'<mark>','</mark>','…',20) AS summary_snippet,
        snippet(search_index,3,'<mark>','</mark>','…',20) AS transcript_snippet
      FROM search_index WHERE search_index MATCH ? ORDER BY rank LIMIT ?
    `).all(query, limit) as SearchResult[]
  } catch { return [] }
}
```

```bash
npm test -- tests/murmurcast/search.test.ts
```

Expected: `✓ 3 tests`

- [ ] **Step 3: Commit**

```bash
git add lib/murmurcast/search.ts tests/murmurcast/search.test.ts
git commit -m "feat(murmurcast): FTS5 search with snippet highlighting"
```

---

## Phase 3 — Processing Pipeline

### Task 7: Summarization

**Files:** `lib/murmurcast/summarize.ts`, `tests/murmurcast/summarize.test.ts`

- [ ] **Step 1: Write failing tests (mocked OpenAI)**

Create `tests/murmurcast/summarize.test.ts`:

```typescript
import { test, expect, vi } from 'vitest'

vi.mock('openai', () => ({
  default: vi.fn().mockImplementation(() => ({
    chat: { completions: { create: vi.fn().mockResolvedValue({
      choices: [{ message: { content: JSON.stringify({
        summary: 'Test summary', key_points: ['p1','p2']
      }) } }],
      usage: { total_tokens: 100 }
    }) } }
  }))
}))

test('buildSummarizePrompt includes transcript + title', async () => {
  const { buildSummarizePrompt } = await import('@/lib/murmurcast/summarize')
  const p = buildSummarizePrompt('transcript text', 'My Video')
  expect(p).toContain('transcript text')
  expect(p).toContain('My Video')
})
test('parseSummarizeResponse extracts fields', async () => {
  const { parseSummarizeResponse } = await import('@/lib/murmurcast/summarize')
  const r = parseSummarizeResponse(JSON.stringify({ summary: 'S', key_points: ['A'] }))
  expect(r.summary).toBe('S')
  expect(r.key_points).toEqual(['A'])
})
test('parseSummarizeResponse handles malformed JSON', async () => {
  const { parseSummarizeResponse } = await import('@/lib/murmurcast/summarize')
  const r = parseSummarizeResponse('plain text')
  expect(r.summary).toBe('plain text')
  expect(r.key_points).toEqual([])
})
```

- [ ] **Step 2: Run → FAIL, implement, run → PASS**

Create `lib/murmurcast/summarize.ts`:

```typescript
import OpenAI from 'openai'
import type Database from 'better-sqlite3'
import { getSetting } from './settings'
import { updateEpisodeStatus } from './episodes'

export function buildSummarizePrompt(transcript: string, title: string): string {
  return `Summarize this transcript from "${title}". Return JSON: {"summary":"...","key_points":["..."]}\n\nTRANSCRIPT:\n${transcript.slice(0,60000)}`
}

export function parseSummarizeResponse(raw: string): { summary: string; key_points: string[] } {
  try {
    const p = JSON.parse(raw)
    return { summary: p.summary ?? raw, key_points: Array.isArray(p.key_points) ? p.key_points : [] }
  } catch {
    return { summary: raw, key_points: [] }
  }
}

export async function summarizeEpisode(
  db: Database.Database, episodeId: number,
  transcriptContent: string, episodeTitle: string
): Promise<void> {
  const apiKey = getSetting(db, 'openai_api_key')
  if (!apiKey) throw new Error('openai_api_key not set in settings')
  const model = getSetting(db, 'gpt_model') ?? 'gpt-4o-mini'
  const openai = new OpenAI({ apiKey })
  updateEpisodeStatus(db, episodeId, 'summarizing')
  const res = await openai.chat.completions.create({
    model,
    messages: [{ role: 'user', content: buildSummarizePrompt(transcriptContent, episodeTitle) }],
    response_format: { type: 'json_object' }
  })
  const { summary, key_points } = parseSummarizeResponse(res.choices[0].message.content ?? '{}')
  db.prepare('INSERT INTO summaries (episode_id,content,key_points,tokens_used) VALUES (?,?,?,?)')
    .run(episodeId, summary, JSON.stringify(key_points), res.usage?.total_tokens ?? 0)
  updateEpisodeStatus(db, episodeId, 'summarized')
}
```

```bash
npm test -- tests/murmurcast/summarize.test.ts
```

Expected: `✓ 3 tests`

- [ ] **Step 3: Commit**

```bash
git add lib/murmurcast/summarize.ts tests/murmurcast/summarize.test.ts
git commit -m "feat(murmurcast): GPT-4o-mini summarization with JSON output"
```

---

### Task 8: Transcription (yt-dlp + Whisper)

**File:** `lib/murmurcast/transcribe.ts`

> No unit tests — this module shells out to yt-dlp/ffmpeg and calls Whisper API. Verified manually in Task 10.

- [ ] **Step 1: Verify system deps**

```bash
which yt-dlp && which ffmpeg
```

If missing: `brew install yt-dlp ffmpeg`

- [ ] **Step 2: Implement transcribe.ts**

Create `lib/murmurcast/transcribe.ts`. The module must:
1. Validate URL (only https://, only YouTube domains — use `new URL(url).hostname`)
2. Download audio using the `execa` package (NOT shell exec): `execa('yt-dlp', ['--extract-audio', '--audio-format', 'mp3', '-o', outputPath, url])`
3. Check file size — if > 24MB, split with ffmpeg: `execa('ffmpeg', ['-i', mp3, '-f', 'segment', '-segment_time', '1200', '-c', 'copy', chunkPattern])`
4. Send each chunk to OpenAI audio.transcriptions.create with model 'whisper-1'
5. Concatenate results
6. Save to `transcripts` table
7. Update episode status: transcribing → transcribed
8. Clean up /tmp/murmurcast/ after transcription

> **IMPORTANT:** Always use `execa` with array arguments (never template strings with user input) to prevent shell injection.

- [ ] **Step 3: Commit**

```bash
git add lib/murmurcast/transcribe.ts
git commit -m "feat(murmurcast): yt-dlp+Whisper transcription with 25MB chunking"
```

---

### Task 9: Digest generation + email

**Files:** `lib/murmurcast/digest.ts`, `tests/murmurcast/digest.test.ts`

- [ ] **Step 1: Write failing tests**

Create `tests/murmurcast/digest.test.ts`:

```typescript
import { test, expect } from 'vitest'

test('buildDigestHtml includes episode titles and date', async () => {
  const { buildDigestHtml } = await import('@/lib/murmurcast/digest')
  const html = buildDigestHtml('2026-04-01', [
    { title: 'Ep A', source: 'YT', summary: 'Sum A', key_points: ['p1'] },
  ])
  expect(html).toContain('Ep A')
  expect(html).toContain('Sum A')
  expect(html).toContain('2026-04-01')
  expect(html).toContain('p1')
})
test('buildDigestHtml handles empty episodes', async () => {
  const { buildDigestHtml } = await import('@/lib/murmurcast/digest')
  expect(buildDigestHtml('2026-04-01', [])).toContain('No new content')
})
```

- [ ] **Step 2: Run → FAIL, implement, run → PASS**

Create `lib/murmurcast/digest.ts` with:
- `buildDigestHtml(date, episodes)` → returns HTML string
- `generateAndSaveDigest(db, date)` → queries today's summarized episodes, calls buildDigestHtml, inserts into digests table (INSERT OR IGNORE), returns digest id
- `sendDigestEmail(db, digestId)` → reads smtp settings via getSetting, sends via nodemailer.createTransport, updates sent_at; silently returns if smtp_host not configured

```bash
npm test -- tests/murmurcast/digest.test.ts
```

Expected: `✓ 2 tests`

- [ ] **Step 3: Commit**

```bash
git add lib/murmurcast/digest.ts tests/murmurcast/digest.test.ts
git commit -m "feat(murmurcast): digest HTML generation and Nodemailer email"
```

---

## Phase 4 — Worker

### Task 10: Background worker

**File:** `worker/murmurcast.ts`

- [ ] **Step 1: Implement worker/murmurcast.ts**

The worker must:
1. Import `getDb` and all processing modules
2. On startup: call `recoverStaleJobs(db)` (reset stuck jobs)
3. Poll every 30s: call `claimJob(db)` → switch on `job.type` → call appropriate function → `completeJob` or `failJob`
4. Job type routing:
   - `transcribe` → `transcribeEpisode(db, episodeId, episode.url)` then enqueue `summarize`
   - `summarize` → `summarizeEpisode(db, episodeId, transcript.content, episode.title)`
   - `ingest_newsletter` → create episode with pasted content as transcript, enqueue `summarize`
   - `digest` → `generateAndSaveDigest(db, date)` then `sendDigestEmail(db, digestId)`
   - `fetch_source` → **explicitly deferred**: log `"[murmurcast] fetch_source not implemented — add YouTube/RSS logic when API key is configured"` and call `failJob(db, job.id, 'fetch_source not implemented')`. See Out of Scope (v1) section. Do NOT silently drop the job.
5. `node-cron` schedule `'0 7 * * *'`: enqueue digest job for today if not already in digests table
6. Handle `SIGTERM`/`SIGINT` for graceful shutdown

- [ ] **Step 2: Smoke test worker starts**

```bash
timeout 3 tsx worker/murmurcast.ts 2>&1 | head -5
```

Expected: `[worker] MurmurCast worker started` then clean exit.

- [ ] **Step 3: Commit**

```bash
git add worker/murmurcast.ts
git commit -m "feat(murmurcast): background worker with polling and daily cron"
```

---

## Phase 5 — UI

### Task 11: Sidebar layout

**File:** `app/murmurcast/layout.tsx`

- [ ] **Step 1: Read existing TabBar.tsx** to match navigation style.

- [ ] **Step 2: Implement layout.tsx** — sidebar with links to all 6 routes using Next.js `Link`. Dark background (`#0a0a0a`), border-right, Geist Sans font.

- [ ] **Step 3: Verify build**

```bash
npm run build 2>&1 | grep -E "(error|murmurcast)"
```

Expected: no errors.

- [ ] **Step 4: Commit**

```bash
git add app/murmurcast/layout.tsx
git commit -m "feat(murmurcast): sidebar navigation layout"
```

---

### Task 12: Settings page

**File:** `app/murmurcast/settings/page.tsx`

- [ ] **Step 1: Implement as Server Component** with inline Server Action `saveSettings`. Form fields for all keys in the `settings` table (openai_api_key, youtube_api_key, smtp config, email config, digest_time, gpt_model, max_episodes_per_fetch). Read current values via `getAllSettings(getDb())` for controlled defaults. Call `revalidatePath('/murmurcast/settings')` after save.

- [ ] **Step 2: Commit**

```bash
git add app/murmurcast/settings/page.tsx
git commit -m "feat(murmurcast): settings page"
```

---

### Task 13: Sources page

**File:** `app/murmurcast/sources/page.tsx`

- [ ] **Step 1: Implement as Server Component** with two Server Actions: `addSource` (calls `createSource` + enqueues `fetch_source` job — note: job will fail with "not implemented" in v1; this is expected) and `toggleActive`. List all sources using `getSources(db, { includeInactive: true })` (active + inactive) with type badge, name, URL, last_fetched_at, pause/resume button. `revalidatePath('/murmurcast/sources')` after each mutation.

- [ ] **Step 2: Commit**

```bash
git add app/murmurcast/sources/page.tsx
git commit -m "feat(murmurcast): sources management page"
```

---

### Task 14: Quick Analyze page + API routes

**Files:** `app/murmurcast/analyze/page.tsx`, `app/api/murmurcast/analyze/route.ts`, `app/api/murmurcast/jobs/[id]/route.ts`

- [ ] **Step 1: Create job status route**

Create `app/api/murmurcast/jobs/[id]/route.ts`:

```typescript
import { NextResponse } from 'next/server'
import { getDb } from '@/lib/murmurcast/db'
import type { Job } from '@/lib/murmurcast/types'

export async function GET(_req: Request, { params }: { params: Promise<{ id: string }> }) {
  const { id } = await params   // params is async in Next.js 16
  const db = getDb()
  const job = db.prepare('SELECT * FROM jobs WHERE id=?').get(parseInt(id)) as Job | undefined
  if (!job) return NextResponse.json({ error: 'not found' }, { status: 404 })
  return NextResponse.json({ id: job.id, status: job.status, error: job.error })
}
```

- [ ] **Step 2: Create analyze API route**

Create `app/api/murmurcast/analyze/route.ts`:

```typescript
import { NextResponse } from 'next/server'
import { getDb } from '@/lib/murmurcast/db'
import { createEpisode } from '@/lib/murmurcast/episodes'
import { enqueueJob } from '@/lib/murmurcast/jobs'

export async function POST(req: Request) {
  const { url } = await req.json() as { url?: string }
  if (!url?.startsWith('http')) return NextResponse.json({ error: 'Invalid URL' }, { status: 400 })
  const db = getDb()
  const episodeId = createEpisode(db, { source_id: null, external_id: null,
    title: `Quick Analyze — ${url.slice(-20)}`, url, is_quick_analyze: true })
  const jobId = enqueueJob(db, 'transcribe', { episode_id: episodeId })
  return NextResponse.json({ job_id: jobId, episode_id: episodeId })
}
```

- [ ] **Step 3: Implement analyze page (Client Component)**

Create `app/murmurcast/analyze/page.tsx` — mark `'use client'`. State: `url`, `jobId`, `status` (idle/pending/processing/done/failed), `error`, `episodeId`. On submit: POST to `/api/murmurcast/analyze`, then `setInterval` polling `/api/murmurcast/jobs/:id` every 2s until done/failed. Show status indicator and link to dashboard on completion.

- [ ] **Step 4: Commit**

```bash
git add app/api/murmurcast/analyze/route.ts \
        app/api/murmurcast/jobs/[id]/route.ts \
        app/murmurcast/analyze/page.tsx
git commit -m "feat(murmurcast): quick analyze with job status polling"
```

---

### Task 15: Search + Dashboard + Digests pages

**Files:** `app/murmurcast/search/page.tsx`, `app/murmurcast/page.tsx`, `app/murmurcast/digests/page.tsx`

- [ ] **Step 1: Implement search/page.tsx**

Server Component. Read `searchParams` (async in Next.js 16: `const { q } = await searchParams`). Call `searchContent(db, q)`. Render results with `dangerouslySetInnerHTML` for HTML `<mark>` snippet highlighting. Include a search `<form>` with `name="q"` input that submits via GET (no JS needed).

- [ ] **Step 2: Implement page.tsx (Dashboard)**

Server Component. Show: today's digest (if exists, inline HTML), pending job count badge, recent 20 summarized episodes with title + source name + summary excerpt (first 300 chars). Empty state: "Add sources or use Quick Analyze."

- [ ] **Step 3: Implement digests/page.tsx**

Server Component. Query last 30 digests ordered by date DESC. Render as `<details>/<summary>` accordion — summary shows date + sent status, expanded body shows full digest HTML via `dangerouslySetInnerHTML`.

- [ ] **Step 4: Commit**

```bash
git add app/murmurcast/search/page.tsx app/murmurcast/page.tsx \
        app/murmurcast/digests/page.tsx
git commit -m "feat(murmurcast): search, dashboard, and digest archive pages"
```

---

## Phase 6 — Integration

### Task 16: Final integration smoke test

- [ ] **Step 1: Run full test suite**

```bash
cd grimoria-os && npm test
```

Expected: all 29 tests pass.

- [ ] **Step 2: Start both processes**

```bash
npm run dev
```

Expected: Next.js starts on localhost + `[worker] MurmurCast worker started`

- [ ] **Step 3: Configure settings**

Open `http://localhost:3000/murmurcast/settings`. Enter OpenAI API key. Save.

- [ ] **Step 4: Quick Analyze smoke test**

1. Go to `/murmurcast/analyze`
2. Paste a short YouTube URL (< 5 min video for speed)
3. Click Analyze — watch status: pending → processing → done
4. Go to `/murmurcast` — result card should appear
5. Go to `/murmurcast/search` — search a word from the video — snippets should appear

- [ ] **Step 5: Commit**

```bash
git add -A
git commit -m "feat(murmurcast): complete MurmurCast feature integrated into grimoria-os"
```

---

## Out of Scope (v1)

- `fetch_source` YouTube Data API implementation (worker stub exists — implement when YouTube API key is configured)
- Newsletter RSS feed parsing (same pattern as podcast RSS)
- shadcn/ui polish — inline styles used for speed; refactor when desired
- Podcast RSS `fetch_source` implementation

---

## Test Coverage Summary

| Module | Tests |
|---|---|
| db.ts | 4 |
| jobs.ts | 7 |
| settings.ts | 4 |
| sources.ts | 2 |
| episodes.ts | 4 |
| search.ts | 3 |
| summarize.ts | 3 |
| digest.ts | 2 |
| **Total** | **29** |
