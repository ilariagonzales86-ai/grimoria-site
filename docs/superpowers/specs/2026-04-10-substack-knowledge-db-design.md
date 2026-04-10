# Substack Knowledge DB — Design Spec

> Skill per costruire e interrogare un database di conoscenza estratto dalle newsletter Substack seguite da Ilaria. Indicizza post per autore e tema, genera profili autore evolutivi, e permette query trasversali.

---

## 1. Architettura

### Struttura file

```
skills/substack-knowledge-db/
├── SKILL.md                     # Skill Claude Code (logica 3 modalità)
├── substack-knowledge-db.skill  # Registro skill (trigger description)
├── data/
│   ├── feeds.json               # Registry autori (dinamico)
│   ├── posts/                   # Un JSON per post
│   └── profiles/                # Un JSON per autore (rigenerato dopo ingest)
```

### Approccio scelto

**Skill pura Claude Code + JSON locale + Apify per ingestione.** Nessun vector DB, nessun embedding, nessuna dipendenza su Ollama. Apify (`automation-lab/substack-scraper`) fa il fetch dei post via API JSON di Substack. Claude arricchisce i dati (summary, tag, insights) e li salva come JSON.

### Perché Apify invece di feedparser/RSS

1. **Archivio illimitato**: RSS espone solo gli ultimi ~10-20 post. Apify scarica l'intero archivio.
2. **Contenuto completo**: HTML pieno del post, non summary troncato.
3. **Metadati ricchi**: reactions, commenti, word count, tipo (newsletter/podcast/thread), paywall status.
4. **Filtri nativi**: per data, tipo contenuto, solo free posts.
5. **Zero dipendenze locali**: niente Python venv, niente librerie. Solo MCP call da Claude.
6. **99.5% success rate**, usa API JSON pubbliche di Substack (no browser, no proxy).
7. **Costo contenuto**: ~$0.002/post con contenuto. Full ingest 22 autori × 10 post = ~$0.44.

### Perché JSON e non ChromaDB

1. I post Substack hanno già struttura (titolo, autore, data, contenuto) — non servono embeddings per filtrarli.
2. Claude compensa la mancanza di ricerca semantica ragionando sui profili e tag.
3. Zero dipendenze runtime: niente Ollama, niente ChromaDB server.
4. File leggibili e ispezionabili manualmente.

---

## 2. Schema dati

### `feeds.json` — Registry autori

```json
{
  "feeds": [
    {
      "id": "ethan-mollick",
      "name": "Ethan Mollick",
      "url": "https://oneusefulthing.substack.com",
      "tags": ["AI", "education", "future-of-work"],
      "notes": "Prof Wharton, voce autorevole su AI nel lavoro",
      "added": "2026-04-10",
      "active": true
    }
  ]
}
```

Campi:

| Campo | Tipo | Descrizione |
|-------|------|-------------|
| `id` | string | Slug univoco, derivato dal nome (lowercase, kebab-case) |
| `name` | string | Nome display dell'autore |
| `url` | string | URL homepage della newsletter Substack (non RSS) |
| `tags` | string[] | Tag tematici dell'autore (vocabolario condiviso) |
| `notes` | string | Note libere sull'autore |
| `added` | string | Data di aggiunta (YYYY-MM-DD) |
| `active` | bool | Se `false`, non viene ingestito ma i post restano consultabili |

### Post JSON — `posts/{autore}_{YYYY-MM-DD}_{slug}.json`

```json
{
  "id": "ethan-mollick_2026-04-08_ai-agents-are-here",
  "author_id": "ethan-mollick",
  "author_name": "Ethan Mollick",
  "title": "AI Agents Are Here",
  "url": "https://oneusefulthing.substack.com/p/ai-agents-are-here",
  "published": "2026-04-08",
  "ingested": "2026-04-10T14:30:00",
  "word_count": 2400,
  "type": "newsletter",
  "is_free": true,
  "reactions": 342,
  "comments_count": 56,
  "tags": ["AI", "agents", "automation"],
  "summary": "Sintesi di 3-5 righe generata da Claude al momento dell'ingest",
  "key_insights": ["insight 1", "insight 2", "insight 3"],
  "content_preview": "Primi 3000 caratteri del post originale (testo pulito da HTML)",
  "grimoria_angle": "Come si collega alla visione GrimorIa"
}
```

Due fasi di scrittura:

1. **Raw** (da Apify): `id`, `author_id`, `author_name`, `title`, `url`, `published`, `ingested`, `word_count`, `type`, `is_free`, `reactions`, `comments_count`, `content_preview`
2. **Arricchito** (da Claude): `tags`, `summary`, `key_insights`, `grimoria_angle`

### Profile JSON — `profiles/{autore_slug}.json`

```json
{
  "id": "ethan-mollick",
  "name": "Ethan Mollick",
  "bio": "Bio sintetica generata dall'analisi dei post",
  "recurring_themes": ["AI agency", "human-AI collaboration", "education"],
  "stance": "Ottimista pragmatico sull'AI, focus su esperimenti concreti",
  "total_posts": 12,
  "avg_reactions": 280,
  "top_post": "AI Agents Are Here (342 reactions)",
  "last_ingested": "2026-04-10",
  "relevance_for_grimoria": "Voce forte su AI come amplificatore, non sostituto — allineato con visione GrimorIa"
}
```

I profili vengono **rigenerati** dopo ogni ingest (non incrementali). Claude rilegge tutti i post dell'autore e riscrive il profilo.

---

## 3. Le 3 modalità

### Modalità 1: `ingest`

**Trigger**: "ingest", "aggiorna", "scarica nuovi post"

**Flusso**:

```
[Apify MCP: call-actor]              [Claude / SKILL.md]
    │                                       │
    ├─ Legge feeds.json                     │
    ├─ Per ogni autore attivo:              │
    │   chiama automation-lab/              │
    │   substack-scraper con:               │
    │   - url newsletter                    │
    │   - maxPostsPerNewsletter             │
    │   - includeContent: true              │
    │   - startDate (ultimo ingest)         │
    │                                       │
    └─ Risultati JSON ────────────────────>│
                                            ├─ Deduplicazione (file exists?)
                                            ├─ Per ogni post nuovo:
                                            │  - Estrai testo da HTML
                                            │  - Genera summary, tags,
                                            │    key_insights, grimoria_angle
                                            │  - Salva JSON in posts/
                                            └─ Rigenera profiles/ per autori toccati
```

**Parametri Apify per ogni call**:

| Parametro | Valore |
|-----------|--------|
| `urls` | `[autore.url]` |
| `maxPostsPerNewsletter` | 5 (default) o 20 (con `--full`) |
| `includeContent` | `true` |
| `includeComments` | `false` (risparmio costi) |
| `includePublicationInfo` | `true` (solo al primo ingest) |
| `contentType` | `"newsletter"` |
| `startDate` | Data ultimo ingest dell'autore (o `2024-01-01` al primo) |
| `onlyFree` | `true` |

**Strategia di chiamata**: un `call-actor` per autore. Con 22 autori × 5 post = ~110 post = ~$0.25.

**Deduplicazione**: controlla se esiste già `posts/{autore}_{data}_{slug}.json`. Se sì, skip. Il filtro `startDate` su Apify riduce a monte i duplicati.

### Modalità 2: `query`

**Trigger**: domanda libera sul database — "cosa dice X su Y?", "chi parla di Z?", "trend della settimana"

**Flusso**:

1. Analizza la domanda → identifica: autore specifico? tema? periodo?
2. Carica dati rilevanti:
   - Autore specifico → profile + suoi post
   - Tema → scansiona profiles per tag match, poi post rilevanti
   - Trend/panoramica → ultimi N post trasversali
3. Sintetizza risposta con citazioni (autore + titolo + link)

**Query supportate**:

| Tipo | Esempio | Strategia |
|------|---------|-----------|
| Per autore | "Cosa dice Mollick sull'AI agency?" | Carica profile + post di Mollick, filtra per tema |
| Per tema | "Chi parla di branding?" | Scansiona profiles per tag, lista autori con snippet |
| Trend | "Trend della settimana" | Post ultimi 7gg, raggruppa per tema |
| Comparazione | "Confronta X e Y sull'AI" | Carica profili + post di entrambi, genera confronto |
| Per GrimorIa | "Insight per la newsletter di questa settimana" | Seleziona 3-5 post più rilevanti, suggerisce angoli |
| Per engagement | "Quali post hanno avuto più reactions?" | Ordina per reactions, mostra top 10 |

**Strategia di caricamento**: la skill non legge tutti i file ogni volta. Prima filtra per profili/tag, poi carica solo i post pertinenti alla query.

### Modalità 3: `manage`

**Trigger**: "aggiungi autore", "rimuovi", "lista feed", "stats"

| Comando | Azione |
|---------|--------|
| "Aggiungi [URL substack]" | Valida URL, chiama Apify per publication info, estrae nome autore, chiede tag all'utente, aggiunge a feeds.json |
| "Rimuovi [nome]" | Mette `active: false` (non cancella post) |
| "Lista feed" | Tabella: nome, tag, n. post, ultimo ingest, stato |
| "Modifica tag di [autore]" | Aggiorna tags in feeds.json |
| "Stats" | Totale post, autori attivi, distribuzione per tag, ultimo ingest globale, costo stimato |

---

## 4. Apify Actor: `automation-lab/substack-scraper`

### Perché questo Actor

- **99.5% success rate**, 59 utenti totali, 34 attivi mensili
- Usa API JSON pubbliche di Substack — no browser rendering, no proxy
- Supporta sia subdomain (`newsletter.substack.com`) che custom domain (`www.lennysnewsletter.com`)
- Archivio illimitato con filtro per data
- Output strutturato: titolo, contenuto HTML, reactions, commenti, word count, tipo, paywall status

### Costi stimati

| Scenario | Post | Costo |
|----------|------|-------|
| Ingest settimanale (5 post × 22 autori) | 110 | ~$0.25 |
| Primo ingest full (20 post × 22 autori) | 440 | ~$1.00 |
| Singolo autore, 10 post | 10 | ~$0.03 |

Costo per post con contenuto: $0.0023 (tier FREE). Il costo scende con tier superiori.

### Invocazione MCP

La skill chiama `mcp__Apify__call-actor` con:

```json
{
  "actorId": "automation-lab/substack-scraper",
  "input": {
    "urls": ["https://oneusefulthing.substack.com"],
    "maxPostsPerNewsletter": 5,
    "includeContent": true,
    "includeComments": false,
    "includePublicationInfo": true,
    "contentType": "newsletter",
    "startDate": "2026-04-01",
    "onlyFree": true
  }
}
```

### Fallback

Se Apify non è disponibile (MCP non configurato, errore, quota esaurita):

1. La skill avvisa l'utente
2. Offre fallback manuale: "Incolla il testo di un post e lo ingerisco direttamente"
3. Non c'è fallback automatico su RSS — Apify è il metodo primario

---

## 5. Pre-popolazione da substack_summarizer

Al primo setup, `feeds.json` viene pre-popolato con i 22 feed già configurati in `substack_summarizer/main.py`:

| # | ID | URL |
|---|-----|-----|
| 1 | ethan-mollick | oneusefulthing.substack.com |
| 2 | simon-sinek | simonsinek.substack.com |
| 3 | the-ai-exchange | theaiexchange.substack.com |
| 4 | kooping-shung | koopingshung.substack.com |
| 5 | founders-corner | www.the-founders-corner.com |
| 6 | cristina | cristina735772.substack.com |
| 7 | a-temple-jar | atemplejar.substack.com |
| 8 | aiblew-my-mind | aiblewmymind.substack.com |
| 9 | lorena-bordonaba | lorenabordonaba.substack.com |
| 10 | ai-meets-girlboss | aimeetsgirlboss.substack.com |
| 11 | ai-but-intimate | aibutintimate.substack.com |
| 12 | artificial-corner | artificialcorner.com |
| 13 | claude-code-masterclass | newsletter.claudecodemasterclass.com |
| 14 | emerging-ai | emergingai.substack.com |
| 15 | ruben | ruben.substack.com |
| 16 | nina-schick | ninaschick.substack.com |
| 17 | inside-small-giants | www.insidesmallgiants.com |
| 18 | mkt1 | newsletter.mkt1.co |
| 19 | nate | natesnewsletter.substack.com |
| 20 | the-slow-ai | theslowai.substack.com |
| 21 | ten-speed | newsletter.tenspeed.io |
| 22 | intelligence-briefing | intelligencebriefing.substack.com |
| 23 | augmented-mind | augmentedmind.substack.com |

I tag iniziali vengono assegnati da Claude durante la creazione del registry, basandosi sulla publication info ottenuta da Apify al primo ingest.

---

## 6. Limitazioni note e future estensioni

### Limitazioni

- **Niente ricerca semantica**: query tipo "chi parla del rapporto tra intuizione e decisioni?" dipendono dalla qualità dei tag e dalla capacità di Claude di ragionare sui profili. Funziona bene ma non è vector search.
- **Scala**: con ~440 post (22 autori × 20 post) i file pesano ~1.5MB totali. Gestibile. Oltre i 1000 post potrebbe servire un indice di lookup.
- **Costo Apify**: ~$0.25/settimana per ingest regolare. Contenuto ma non zero.
- **Solo post gratuiti**: il flag `onlyFree` esclude contenuti paywalled. Per includere post a pagamento serve autenticazione Substack (non supportata dall'Actor).

### Possibili estensioni future (non in scope)

- Migrazione a ChromaDB se il volume cresce oltre 1000 post
- Ingest automatico via scheduled task / cron
- Integrazione con `substack-notes-writer` per suggerire temi basati su trend dal DB
- Export del database come pagina HTML consultabile (stile Grimoria Magazine)
- Inclusione commenti (Apify lo supporta, disabilitato per costi)
