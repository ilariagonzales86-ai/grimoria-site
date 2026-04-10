# Substack Knowledge DB — Design Spec

> Skill per costruire e interrogare un database di conoscenza estratto dalle newsletter Substack seguite da Ilaria. Indicizza post per autore e tema, genera profili autore evolutivi, e permette query trasversali.

---

## 1. Architettura

### Struttura file

```
skills/substack-knowledge-db/
├── SKILL.md                     # Skill Claude Code (logica 3 modalità)
├── substack-knowledge-db.skill  # Registro skill (trigger description)
├── ingest.py                    # Script Python fetch + parse RSS
├── requirements.txt             # feedparser, beautifulsoup4, requests
├── setup.sh                     # Bootstrap venv + install deps
├── data/
│   ├── feeds.json               # Registry autori (dinamico)
│   ├── posts/                   # Un JSON per post
│   └── profiles/                # Un JSON per autore (rigenerato dopo ingest)
```

### Approccio scelto

**Skill pura + JSON locale.** Nessun vector DB, nessun embedding, nessuna dipendenza su Ollama o API esterne per lo storage. Lo script Python fa fetch + parsing meccanico, Claude fa arricchimento intelligente (summary, tag, insights).

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
      "url": "https://oneusefulthing.substack.com/feed",
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
| `url` | string | URL feed RSS Substack |
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
  "tags": ["AI", "agents", "automation"],
  "summary": "Sintesi di 3-5 righe generata da Claude al momento dell'ingest",
  "key_insights": ["insight 1", "insight 2", "insight 3"],
  "content_preview": "Primi 2000 caratteri del post originale",
  "grimoria_angle": "Come si collega alla visione GrimorIa"
}
```

Due fasi di scrittura:

1. **Raw** (da `ingest.py`): `id`, `author_id`, `author_name`, `title`, `url`, `published`, `ingested`, `content_preview`
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
[ingest.py]                          [Claude / SKILL.md]
    │                                       │
    ├─ Legge feeds.json                     │
    ├─ Fetch RSS × N feed attivi            │
    ├─ Parse HTML → testo pulito            │
    ├─ Deduplicazione (file exists?)        │
    ├─ Salva raw JSON in posts/             │
    │                                       │
    └─ stdout report ──────────────────────>│
                                            ├─ Legge i raw JSON nuovi
                                            ├─ Per ogni post: genera summary, tags,
                                            │  key_insights, grimoria_angle
                                            ├─ Aggiorna i JSON con campi arricchiti
                                            └─ Rigenera profiles/ per autori toccati
```

**Parametri**:

| Parametro | Default | Descrizione |
|-----------|---------|-------------|
| Max post per feed | 3 | Quanti post recenti prendere dal RSS |
| `--full` | false | Prende fino a 10 post per feed (primo ingest) |

**Deduplicazione**: controlla se esiste già `posts/{autore}_{data}_{slug}.json`. Se sì, skip.

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

**Strategia di caricamento**: la skill non legge tutti i file ogni volta. Prima filtra per profili/tag, poi carica solo i post pertinenti alla query.

### Modalità 3: `manage`

**Trigger**: "aggiungi autore", "rimuovi", "lista feed", "stats"

| Comando | Azione |
|---------|--------|
| "Aggiungi [URL substack]" | Fetch feed per estrarre nome, genera slug, chiede tag all'utente, aggiunge a feeds.json |
| "Rimuovi [nome]" | Mette `active: false` (non cancella post) |
| "Lista feed" | Tabella: nome, tag, n. post, ultimo ingest, stato |
| "Modifica tag di [autore]" | Aggiorna tags in feeds.json |
| "Stats" | Totale post, autori attivi, distribuzione per tag, ultimo ingest globale |

---

## 4. Script Python `ingest.py`

### Responsabilità

Solo fetch e parsing meccanico. Non fa arricchimento AI.

**Input**: path a `feeds.json`, path a `posts/`, max post per feed
**Output**: JSON raw salvati in `posts/`, report su stdout

### Dipendenze

```
feedparser
beautifulsoup4
requests
```

### Comportamento

1. Legge `feeds.json`, filtra `active: true`
2. Per ogni feed:
   - Fetch RSS con `feedparser`
   - Per ogni entry (fino a `max_posts`):
     - Genera slug dal titolo
     - Genera `id` = `{author_id}_{published}_{slug}`
     - Se file esiste in `posts/` → skip
     - Altrimenti: parse HTML con BeautifulSoup, estrai testo, salva raw JSON
3. Stampa report: "N nuovi post da M autori"

### Setup

`setup.sh` crea un venv locale nella skill:

```bash
#!/usr/bin/env bash
SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
VENV="$SCRIPT_DIR/venv"
if [ ! -d "$VENV" ]; then
    python3 -m venv "$VENV"
fi
"$VENV/bin/pip" install --quiet -r "$SCRIPT_DIR/requirements.txt"
```

---

## 5. Pre-popolazione da substack_summarizer

Al primo setup, `feeds.json` viene pre-popolato con i 22 feed già configurati in `substack_summarizer/main.py`:

```
oneusefulthing, simonsinek, theaiexchange, koopingshung,
the-founders-corner, cristina735772, atemplejar, aiblewmymind,
lorenabordonaba, aimeetsgirlboss, aibutintimate, artificialcorner,
claudecodemasterclass, emergingai, ruben, ninaschick,
insidesmallgiants, mkt1, natesnewsletter, theslowai,
tenspeed, intelligencebriefing, augmentedmind
```

I tag iniziali vengono assegnati da Claude durante la creazione del registry, basandosi sul nome/URL del feed.

---

## 6. Limitazioni note e future estensioni

### Limitazioni

- **Niente ricerca semantica**: query tipo "chi parla del rapporto tra intuizione e decisioni?" dipendono dalla qualità dei tag e dalla capacità di Claude di ragionare sui profili. Funziona bene ma non è vector search.
- **Scala**: con ~220 post (22 autori × 10 post) i file pesano ~500KB totali. Gestibile. Oltre i 500 post potrebbe servire un indice di lookup.
- **Feed RSS limitati**: Substack espone solo gli ultimi ~10-20 post nel feed RSS. Non è possibile fare backfill storico completo.

### Possibili estensioni future (non in scope)

- Migrazione a ChromaDB se il volume cresce oltre 500 post
- Ingest automatico via cron/scheduled task
- Integrazione con `substack-notes-writer` per suggerire temi basati su trend dal DB
- Export del database come pagina HTML consultabile (stile Grimoria Magazine)
