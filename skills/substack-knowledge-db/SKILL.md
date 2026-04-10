---
name: substack-knowledge-db
description: "Skill per gestire il database di conoscenza delle newsletter Substack. Usa questa skill OGNI VOLTA che l'utente chiede di: ingerire/scaricare/aggiornare i post delle newsletter Substack, interrogare il database delle newsletter ('cosa dice X su Y?', 'chi parla di Z?', 'trend della settimana'), gestire i feed seguiti (aggiungere autore, rimuovere, lista feed, stats). NON usare per scrivere Note Substack (usa substack-notes-writer), per analisi carte natali (usa astro-hd-gk-profiler), o per il Magazine HTML (substack_summarizer)."
---

# Substack Knowledge DB

## Scopo

Database di conoscenza costruito dalle newsletter Substack seguite da Ilaria. Ogni autore diventa un profilo con temi ricorrenti, stance, e rilevanza per GrimorIa. Ogni post viene arricchito con summary, tag, insight chiave, e angolo GrimorIa.

## Come funziona

Tre modalità operative, rilevate automaticamente:

| Modalità | Trigger dell'utente | Cosa fa |
|----------|-------------------|---------|
| **Ingest** | "ingest", "aggiorna", "scarica nuovi post", "aggiorna il database" | Scarica post da Substack via Apify, arricchisce con Claude, salva JSON |
| **Query** | Domanda sul database: "cosa dice X su Y?", "chi parla di Z?", "trend" | Legge profili e post, sintetizza risposta con citazioni |
| **Manage** | "aggiungi autore", "rimuovi", "lista feed", "stats" | Gestisce il registry dei feed in feeds.json |

### Rilevamento modalità

1. Se il messaggio contiene "ingest", "aggiorna", "scarica", "download", "fetch" → **Ingest**
2. Se il messaggio contiene "aggiungi", "rimuovi", "lista feed", "stats", "gestisci" → **Manage**
3. Altrimenti → **Query** (default: qualsiasi domanda sul database)

Se non è chiaro, chiedi: "Vuoi che aggiorni il database (ingest), che cerchi informazioni (query), o che gestisca i feed (manage)?"

---

## Percorsi file

Tutti i percorsi sono relativi a `skills/substack-knowledge-db/`:

| File | Scopo |
|------|-------|
| `data/feeds.json` | Registry degli autori seguiti (URL, tag, stato) |
| `data/posts/{author_id}_{YYYY-MM-DD}_{slug}.json` | Singolo post (raw + arricchito) |
| `data/profiles/{author_id}.json` | Profilo autore (rigenerato dopo ogni ingest) |

---

## Schema dati

### feeds.json

```json
{
  "feeds": [
    {
      "id": "slug-autore",
      "name": "Nome Display",
      "url": "https://newsletter.substack.com",
      "tags": ["tag1", "tag2"],
      "notes": "Note libere",
      "added": "YYYY-MM-DD",
      "active": true
    }
  ]
}
```

### Post JSON

Campi **raw** (da Apify):

| Campo | Tipo | Fonte |
|-------|------|-------|
| `id` | string | `{author_id}_{published}_{slug}` |
| `author_id` | string | Da feeds.json |
| `author_name` | string | Da feeds.json |
| `title` | string | Apify `title` |
| `url` | string | Apify `url` o `canonical_url` |
| `published` | string | Apify `post_date` → formato YYYY-MM-DD |
| `ingested` | string | Timestamp ISO dell'ingestione |
| `word_count` | int | Apify `word_count` |
| `type` | string | Apify `type` (newsletter/podcast/thread) |
| `is_free` | bool | Apify `audience` == "everyone" |
| `reactions` | int | Apify `reactions.❤` o totale reactions |
| `comments_count` | int | Apify `comment_count` |
| `content_preview` | string | Primi 3000 char del body text (HTML → testo) |

Campi **arricchiti** (da Claude):

| Campo | Tipo | Cosa contiene |
|-------|------|---------------|
| `tags` | string[] | 3-5 tag tematici dal vocabolario condiviso |
| `summary` | string | Sintesi 3-5 righe in italiano |
| `key_insights` | string[] | 2-4 insight principali, una frase ciascuno |
| `grimoria_angle` | string | Come si collega alla visione GrimorIa (1-2 frasi) |

### Profile JSON

```json
{
  "id": "slug-autore",
  "name": "Nome Display",
  "bio": "Bio sintetica 2-3 righe",
  "recurring_themes": ["tema1", "tema2", "tema3"],
  "stance": "Posizione dell'autore in una frase",
  "total_posts": 12,
  "avg_reactions": 280,
  "top_post": "Titolo Post (N reactions)",
  "last_ingested": "YYYY-MM-DD",
  "relevance_for_grimoria": "Rilevanza per GrimorIa in 1-2 frasi"
}
```

---

## Configurazione Apify

**Actor**: `automation-lab/substack-scraper`

**Parametri standard per ingest**:

```json
{
  "urls": ["URL_NEWSLETTER"],
  "maxPostsPerNewsletter": 5,
  "includeContent": true,
  "includeComments": false,
  "includePublicationInfo": true,
  "contentType": "newsletter",
  "startDate": "YYYY-MM-DD",
  "onlyFree": true
}
```

**Mapping output Apify → Post JSON**:

L'output di Apify contiene campi come `title`, `subtitle`, `post_date`, `canonical_url`, `body_html`, `word_count`, `comment_count`, `reactions`, `audience`, `type`. Mappa questi campi allo schema Post JSON sopra.

Per estrarre il testo dal body HTML: rimuovi tutti i tag HTML, conserva solo il testo. Tronca a 3000 caratteri per `content_preview`.

---

## Vocabolario tag condiviso

Quando assegni tag ai post, usa SOLO tag da questa lista (aggiungi nuovi solo se strettamente necessario):

**Macro-categorie**: AI, business, marketing, spirituality, consciousness, personal-growth, leadership, branding, entrepreneurship, technology

**Sotto-categorie**: agents, automation, coding, content-marketing, creativity, decision-making, deepfakes, education, emerging-tech, future-of-work, geopolitics, growth, innovation, intentional-tech, media, mindfulness, personal-brand, productivity, purpose, research, SEO, small-business, startup, strategy, tools, tutorials, women-in-business
