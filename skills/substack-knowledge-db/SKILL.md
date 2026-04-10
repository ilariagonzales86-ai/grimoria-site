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

## Modalità: Ingest

### Quando si attiva

L'utente dice "ingest", "aggiorna", "scarica nuovi post", "aggiorna il database", "fetch newsletter".

Parametri opzionali rilevabili dal messaggio:
- `--full` o "ingest completo" → `maxPostsPerNewsletter: 20` (invece di 5)
- Nome autore specifico → ingestisce solo quell'autore
- "tutti" o nessuna specifica → ingestisce tutti gli autori attivi

### Workflow

**Fase 1 — Preparazione**

1. Leggi `data/feeds.json`
2. Filtra solo i feed con `active: true`
3. Se l'utente ha specificato un autore, filtra anche per quello
4. Per ogni autore, determina `startDate`:
   - Se esiste `data/profiles/{author_id}.json`, usa `last_ingested` come startDate
   - Se non esiste, usa `2024-01-01`
5. Comunica all'utente: "Avvio ingestione per N autori. Costo stimato: ~$X.XX"

**Fase 2 — Fetch via Apify**

Per ogni autore, chiama `mcp__Apify__call-actor`:

```
Actor: automation-lab/substack-scraper
Input:
  urls: [autore.url]
  maxPostsPerNewsletter: 5 (o 20 se --full)
  includeContent: true
  includeComments: false
  includePublicationInfo: true
  contentType: "newsletter"
  startDate: (calcolato in Fase 1)
  onlyFree: true
```

Dopo la chiamata, usa `mcp__Apify__get-actor-output` con il `datasetId` restituito per ottenere i risultati completi.

**Se Apify non è disponibile**: avvisa l'utente e offri il fallback manuale:
> "Apify non è disponibile. Puoi incollare il testo di un post e lo ingerisco direttamente come JSON."

**Fase 3 — Deduplicazione e salvataggio raw**

Per ogni post restituito da Apify:

1. Genera lo slug dal titolo: lowercase, sostituisci spazi con `-`, rimuovi caratteri speciali, tronca a 50 char
2. Genera l'ID: `{author_id}_{published_YYYY-MM-DD}_{slug}`
3. Genera il filename: `data/posts/{id}.json`
4. Se il file esiste già → skip (post già ingestito)
5. Se il file non esiste → crea il JSON raw:

```json
{
  "id": "GENERATED_ID",
  "author_id": "FROM_FEEDS",
  "author_name": "FROM_FEEDS",
  "title": "FROM_APIFY",
  "url": "FROM_APIFY_canonical_url",
  "published": "FROM_APIFY_post_date_AS_YYYY-MM-DD",
  "ingested": "NOW_ISO_TIMESTAMP",
  "word_count": "FROM_APIFY",
  "type": "FROM_APIFY",
  "is_free": true,
  "reactions": "FROM_APIFY_reactions_total",
  "comments_count": "FROM_APIFY_comment_count",
  "tags": [],
  "summary": "",
  "key_insights": [],
  "content_preview": "FIRST_3000_CHARS_OF_BODY_TEXT",
  "grimoria_angle": ""
}
```

**Fase 4 — Arricchimento Claude**

Per ogni post nuovo (non skippato), arricchisci i campi vuoti:

**Prompt di arricchimento** (usa internamente, non mostrare all'utente):

> Analizza questo post di newsletter e genera:
>
> 1. **tags**: 3-5 tag dal vocabolario condiviso (vedi sezione "Vocabolario tag condiviso" sopra). Solo tag rilevanti.
> 2. **summary**: Sintesi in italiano, 3-5 righe. Cattura l'essenza, non parafrasare. Tono neutro-informativo.
> 3. **key_insights**: 2-4 insight principali, una frase ciascuno. Solo insight non ovvi e azionabili.
> 4. **grimoria_angle**: In 1-2 frasi, come questo contenuto si collega alla visione GrimorIa (AI come amplificatore dell'unicità, posizionamento autentico, spiritualità incarnata nel business). Se non c'è un collegamento naturale, scrivi "Collegamento indiretto — utile come benchmark/ispirazione".
>
> Post:
> Titolo: {title}
> Autore: {author_name}
> Contenuto: {content_preview}

Aggiorna il file JSON del post con i campi arricchiti.

**Fase 5 — Rigenerazione profili**

Per ogni autore che ha ricevuto nuovi post:

1. Leggi TUTTI i file in `data/posts/` che iniziano con `{author_id}_`
2. Leggi il feed entry da `data/feeds.json` per nome e note

**Prompt di generazione profilo** (usa internamente):

> Basandoti su questi {N} post di {author_name}, genera un profilo autore:
>
> 1. **bio**: Bio sintetica 2-3 righe. Chi è, cosa scrive, qual è la sua voce.
> 2. **recurring_themes**: 3-5 temi ricorrenti nei post (non tag, ma temi discorsivi).
> 3. **stance**: La posizione/prospettiva dell'autore in una frase.
> 4. **avg_reactions**: Media reactions tra tutti i post.
> 5. **top_post**: Il post con più reactions (titolo + numero).
> 6. **relevance_for_grimoria**: Come questo autore è rilevante per la visione GrimorIa. Sii specifico.
>
> Post disponibili:
> {per ogni post: titolo, data, summary, tags, reactions}

3. Scrivi (o sovrascrivi) `data/profiles/{author_id}.json` con:

```json
{
  "id": "author_id",
  "name": "author_name",
  "bio": "GENERATED",
  "recurring_themes": ["GENERATED"],
  "stance": "GENERATED",
  "total_posts": N,
  "avg_reactions": CALCULATED,
  "top_post": "GENERATED",
  "last_ingested": "TODAY_YYYY-MM-DD",
  "relevance_for_grimoria": "GENERATED"
}
```

**Fase 6 — Report**

Presenta all'utente un report finale:

```
## Ingest completato — [data]

| Autore | Nuovi post | Totale | Top post |
|--------|-----------|--------|----------|
| Ethan Mollick | 3 | 15 | "AI Agents Are Here" (342 ❤) |
| ... | ... | ... | ... |

**Totale**: N nuovi post da M autori
**Costo Apify stimato**: ~$X.XX
**Prossimo ingest consigliato**: [data +7 giorni]
```
