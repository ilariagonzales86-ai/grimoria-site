# Substack Notes Writer — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create a Claude Code skill that generates 5-7 Substack Notes with variants in the GrimorIa brand voice, with trend research and optional RAG integration.

**Architecture:** Single skill directory under `skills/substack-notes-writer/` with SKILL.md as the main logic file and 4 reference files distilled from existing brand/strategy documents. No runtime code — purely prompt-driven skill that orchestrates Claude Code tools (WebSearch, WebFetch, Apify MCP, grimoria-rag MCP).

**Tech Stack:** Claude Code skill system (SKILL.md + references/), MCP tools (grimoria-rag, Apify), WebSearch/WebFetch

**Spec:** `docs/superpowers/specs/2026-04-08-substack-notes-writer-design.md`

---

## File Structure

```
skills/substack-notes-writer/
├── SKILL.md                          # Main skill file — workflow, rules, generation logic
└── references/
    ├── brand-voice-compact.md        # Distilled brand voice: vocabulary, pillars, expressions, anti-patterns
    ├── portali-e-formati.md          # 6 Portali di Apertura + 5 Build in Public formats + FVIA workflow
    ├── algo-substack-2026.md         # Substack 2026 algorithm rules and best practices
    └── trend-research-prompts.md     # Query templates for WebSearch, Apify (X/Twitter)
```

Each file has one job:
- `SKILL.md` — orchestrates the 5-phase pipeline, never contains reference data
- `brand-voice-compact.md` — single source of truth for voice rules during generation
- `portali-e-formati.md` — single source of truth for Note structure and formats
- `algo-substack-2026.md` — algorithm constraints that shape formatting decisions
- `trend-research-prompts.md` — copy-pasteable query templates for research tools

---

### Task 1: Create `references/brand-voice-compact.md`

**Files:**
- Read: `Grimoria Analisi e Branding/02_Brand/brand-voice-guidelines.md`
- Create: `skills/substack-notes-writer/references/brand-voice-compact.md`

This file distills the ~500-line brand guidelines into ~150 operational lines. Only what the generation phase needs — no strategy, no rationale.

- [ ] **Step 1: Read the source brand guidelines**

Read the full file at `Grimoria Analisi e Branding/02_Brand/brand-voice-guidelines.md`. Extract:
1. The 4 pillar definitions with 1 example each (from Section 2)
2. The full Si/No vocabulary table (22 rows, from Section 5)
3. The forbidden words list (from Section 5)
4. The 5 signature expression categories with examples (from Section 5)
5. The authenticity test (4 questions, from Section 2 of the profiler skill)

- [ ] **Step 2: Create the compact reference file**

Create `skills/substack-notes-writer/references/brand-voice-compact.md`:

```markdown
# Brand Voice — Riferimento Operativo per Note Substack

> Estratto da: brand-voice-guidelines.md
> Uso: generazione e validazione Note Substack nel tono GrimorIa

---

## I 4 Pilastri del Tono

### 1. Calore con Spina Dorsale
Il pubblico si sente accolto — e da quello spazio sicuro riceve la verità scomoda.
**Esempio:** "Lo so che fa paura. E lo facciamo lo stesso."

### 2. Regale ma Mai Distante
Tono sovrano naturale — non arrogante. Registro elevato senza essere accademico.
**Esempio:** "Quello che sto per condividere con te non è per tutti. È per chi è pronto."

### 3. Shock Amorevole
Ribaltamenti di prospettiva detti con tale amore che la persona si sente svegliata, non attaccata.
**Esempio:** "E se il problema non fosse che non hai abbastanza disciplina, ma che stai usando la disciplina per la cosa sbagliata?"

### 4. Parla Poco, Colpisci Sempre
Non è quella che posta ogni giorno. È quella che quando parla, le persone salvano il post.
**Esempio:** "Smetti di cercare il permesso. Il permesso sei tu."

---

## Vocabolario: Sostituzioni Obbligatorie

| USA QUESTA | INVECE DI QUESTA |
|---|---|
| Trasformazione | Miglioramento |
| Profondità | Tips / Trucchi |
| Risveglio | Motivazione |
| Campo | Funnel |
| Iniziazione | Percorso step-by-step |
| Visione | Strategia (da sola) |
| Incarnare | Imparare |
| Magnetismo | Marketing |
| Sovranità | Empowerment |
| Frequenza | Mindset |
| Scossa amorevole | Kick motivazionale |
| Mondo / Universo | Offerta / Prodotto |
| Alchimia | Metodo |
| Brillare | Crescere / Scalare |
| Sacro | Importante |
| Le persone che servo / chi è pronto | Target / avatar / ideal client |
| Regalo / dono / invito | Lead magnet |
| Espandere / far brillare di più | Scalare / monetizzare |
| Cambio di frequenza | Mindset shift |
| Ciò che senti / dove fa male | Pain point |
| Verità, chiavi | Tips, trucchi |
| Visione architettata | Strategia |

## Parole Proibite (mai usare)

Funnel, lead magnet, target, scalare, monetizzare, KPI, ROI, metriche (nel linguaggio pubblico), hack, trucco, scorciatoia, grind, hustle, mindset shift, call to action (come termine), pain point, ideal client, avatar, empowerment, tips, motivazione, metodo, offerta, prodotto, importante.

---

## Espressioni Firma

**Per aprire:**
- "La verità scomoda è..."
- "Nessuno te lo dice, ma..."
- "Ti chiedo un minuto di onestà radicale."
- "Fermati un secondo. Respira. E ora leggimi."

**Per chiudere:**
- "Lo sai già. Lo senti."
- "Questo è il momento."
- "Se questo ti risuona, sai dove trovarmi."
- "La porta è aperta. Quando sei pronta, è per te."
- "Io ti vedo."

**Per lo shock amorevole:**
- "E se il problema fosse esattamente l'opposto di quello che pensi?"
- "Stai cercando la risposta nel posto sbagliato. La risposta sei tu."
- "La comfort zone non è comoda. È solo familiare."
- "Non hai bisogno di più informazioni. Hai bisogno di più coraggio."

**Per creare connessione emotiva:**
- "Lo so. Lo sento anche io."
- "Sei nel posto giusto. Anche se non lo sembra."
- "Non sei rotta. Sei in trasformazione."

---

## Anti-Pattern (rifiuta Note che contengono questi)

| Anti-pattern | Segnale | Motivo |
|---|---|---|
| Lista di consigli | "3 modi per...", "5 cose che..." | GrimorIa non dà tips. Apre varchi. |
| Motivazionale generico | "Credi in te stessa!", "Ce la puoi fare!" | Zero risonanza. Già sentito mille volte. |
| CTA mascherata | "Se vuoi saperne di più...", "Scopri come..." | Le Note non vendono. Mai. |
| Jargon spirituale vuoto | "Alza la tua vibrazione", "Manifesta abbondanza" | Spiritualità incarnata, non decorativa. |
| Flex travestito | "Ero al verde, ora fatturo 6 cifre" | Build in Essence, non Build in Ego. |
| Astro-generico | "Mercurio retrogrado, attenzione!" | Se parli di astro, parli del TUO chart specifico. |

---

## Test di Autenticità (ogni Nota deve passare 3 su 4)

1. La salverei se la leggessi nel feed di qualcun altro?
2. Funziona senza contesto (chi non ha letto la newsletter capisce)?
3. C'è una verità che il lettore sente ma non ha ancora nominato?
4. La pubblicherei anche con zero like?
```

- [ ] **Step 3: Verify the file is self-contained**

Read the created file. Check:
- All 22 vocabulary substitutions are present
- All 4 pillar definitions have exactly 1 example each
- Forbidden words list is complete
- All 5 expression categories are present
- Anti-pattern table has 6 entries
- Authenticity test has 4 questions
- No references to external files

- [ ] **Step 4: Commit**

```bash
git add skills/substack-notes-writer/references/brand-voice-compact.md
git commit -m "feat: add brand voice compact reference for substack-notes-writer skill"
```

---

### Task 2: Create `references/portali-e-formati.md`

**Files:**
- Read: `Grimoria Analisi e Branding/Substack/Framework_Note_Substack.md`
- Read: `Grimoria Analisi e Branding/Substack/Strategia_Substack_Build_In_Public_Grimoria_2026.md`
- Create: `skills/substack-notes-writer/references/portali-e-formati.md`

Merges the 6 Portali di Apertura + 5 Build in Public formats + FVIA extraction workflow into one operational file.

- [ ] **Step 1: Read both source documents**

Read `Framework_Note_Substack.md` (138 lines) — extract the 6 Portali with structure and 2 examples each.
Read `Strategia_Substack_Build_In_Public_Grimoria_2026.md` (240 lines) — extract the 5 Note formats with examples, and the FVIA-to-Notes workflow.

- [ ] **Step 2: Create the merged reference file**

Create `skills/substack-notes-writer/references/portali-e-formati.md`:

```markdown
# Portali di Apertura e Formati Note — Riferimento Operativo

> Fonti: Framework_Note_Substack.md, Strategia_Substack_Build_In_Public_Grimoria_2026.md
> Uso: selezione del portale e formato per ogni Nota generata

---

## I 6 Portali di Apertura

Ogni Nota si apre con uno di questi 6 portali. Il portale determina la prima frase.

### 1. Dichiarazione Sovrana
Pianta la bandiera. Nessun tentennamento. Dici quello che è vero per te.
**Struttura:** Una frase. Punto. Nessuna giustificazione.
**Pilastro:** Calore con Spina Dorsale
**Esempi:**
- Il tuo business non ha bisogno di un'altra strategia. Ha bisogno di te, incarnata.
- Ho smesso di costruire dalla testa. E tutto ha iniziato a funzionare.

### 2. Domanda che Spacca
Entra nella mente del lettore con una domanda che già si sta facendo — ma non ha ancora formulato.
**Struttura:** Una domanda. Nessuna risposta immediata. Il silenzio dopo è il contenuto.
**Pilastro:** Shock Amorevole
**Esempi:**
- E se il problema non fosse la mancanza di disciplina, ma il fatto che stai usando la disciplina per la cosa sbagliata?
- Da quanto tempo stai costruendo qualcosa che non ti appartiene?

### 3. Rovesciamento
Prendi una credenza accettata e mostra l'altro lato. Non per provocare — per svegliare.
**Struttura:** L'opposto della narrativa dominante. Detto con amore, non con arroganza.
**Pilastro:** Shock Amorevole + Regale ma Mai Distante
**Esempi:**
- Il business plan è il modo più elegante per ignorare chi sei davvero.
- Non hai bisogno di più clienti. Hai bisogno di più te.

### 4. Momento Esatto
Ancora il lettore a un istante preciso. Il corpo entra nella storia prima che la mente decida se leggere.
**Struttura:** Un dettaglio sensoriale specifico. Tempo, luogo, sensazione fisica.
**Pilastro:** Calore con Spina Dorsale
**Esempi:**
- Erano le 3 di notte quando ho capito che il business che avevo costruito non era mio.
- C'è stato un momento, seduta davanti al laptop a mezzanotte, in cui il corpo ha detto basta prima della mente.

### 5. Vulnerabilità Incarnata
Condividi qualcosa di vero su di te. Non per vittimismo — per creare il campo dove l'altro si sente autorizzato a essere umano.
**Struttura:** Una confessione in prima persona. Breve. Senza giustificazione né redenzione nella stessa frase.
**Pilastro:** Calore con Spina Dorsale + Parla Poco Colpisci Sempre
**Esempi:**
- Per anni ho costruito un business perfetto sulla carta e completamente disconnesso da chi sono.
- Non sono mai stata brava a seguire i piani degli altri. Solo che ci ho messo 15 anni a capire che non era un difetto.

### 6. Dettaglio che Spacca la Realtà
Un fatto, un dato, un'osservazione che il lettore non si aspetta. Il momento "non lo sapevo".
**Struttura:** Un dato specifico o un'osservazione insolita. Poi silenzio.
**Pilastro:** Shock Amorevole + Regale ma Mai Distante
**Esempi:**
- Il tuo Sole non è dove pensi che sia. La posizione vedica differisce di circa 24° da quella occidentale.
- C'è un canale nel tuo Human Design che determina come il tuo corpo comunica. Il mio dice: parla poco, colpisci sempre.

---

## I 5 Formati Build in Public ("Frammenti dal Cantiere")

Usare quando il contenuto documenta il processo di costruzione di GrimorIa.

### 1. Flash dal Laboratorio
Cosa ho fatto oggi. 2-3 righe.
**Esempio:** "Oggi ho costruito un agente AI che prende la mia Carta Natale e genera il piano editoriale del mese basato sui transiti. Non i transiti generici — i MIEI. Funziona. Vi racconto martedì come."

### 2. Decisione Nuda
Una scelta di business + il perché.
**Esempio:** "Ho deciso di non lanciare il paid tier di Grimoria prima dei 500 iscritti. Il mio Sacrale ha detto 'non ancora'. Il mercato mi dà ragione: chi lancia troppo presto vende ansia, non valore."

### 3. Numero Vero
Una metrica reale, senza hype.
**Esempio:** "Settimana 3 di Grimoria. 87 iscritti. Tasso di apertura: 68%. Commenti ricevuti: 12 di cui 4 erano 'come fai a scrivere esattamente quello che sto vivendo?' Non vendo ancora nulla. Il campo si sta costruendo."

### 4. Frase-Lama
Insight emerso dal fare. Massima densità.
**Esempio:** "Stavo costruendo un'automazione AI per l'onboarding clienti. Mi sono accorta che cercavo di automatizzare il calore umano. Il sistema non ha bisogno di replicare te — ha bisogno di liberarti per essere più te."

### 5. Dietro le Quinte con Screenshot
Un pezzo del tuo sistema, mostrato. Visuale + 2 righe di contesto.
**Esempio:** Un'immagine del tuo workspace Claude, o del prompt che hai usato, o della mappa astrologica che hai consultato per una decisione.

---

## Workflow: Da Newsletter (FVIA) a Note

Quando la skill riceve testo di una newsletter:

1. Identifica le 4 sezioni FVIA:
   - **F (Ferita):** Cosa stavo vivendo — il punto di partenza reale
   - **V (Verità):** Lo shock amorevole emerso dal fare
   - **I (Incarnazione):** Cosa è cambiato nel business — il prima/dopo concreto
   - **A (Accesso):** Il prompt, la domanda, il micro-sistema nato dall'esperienza

2. Per ogni sezione, estrai le frasi che superano il test "la salverei anche se non sapessi chi l'ha scritta?"

3. Per ogni frase, scegli il Portale che la esprime meglio

4. Riscrivi come Nota autonoma — deve funzionare senza contesto

5. Aggiungi 1-2 righe di espansione — non spiegare, approfondisci

---

## Regole Strutturali per Ogni Nota

1. Una frase di apertura (portale). Poi espandi in 2-3 righe max. Totale: 5-6 righe.
2. Mai CTA esplicita. Niente "leggi la newsletter", "link in bio", "clicca qui".
3. Per rimandare alla newsletter: "Ne parlo nella lettera di questa settimana." — basta.
4. Ogni Nota deve reggere da sola.
5. Vocabolario GrimorIa — stesse regole del brand voice compact.

---

## Calendario Tipo (per suggerimenti scheduling)

| Giorno | Tipo | Note |
|--------|------|------|
| Martedì | PUBBLICA newsletter | — |
| Mercoledì | Teaser dalla newsletter | Portale: qualsiasi. Formato: da FVIA. |
| Giovedì | Flash dal Laboratorio o Numero Vero | Portale: Momento Esatto o Dettaglio. |
| Venerdì | Frase-Lama o Decisione Nuda | Portale: Dichiarazione o Rovesciamento. |
| Weekend | Solo se esce naturale | Qualsiasi formato. |
```

- [ ] **Step 3: Verify completeness**

Check:
- All 6 Portali present with structure, pillar, and 2 examples each
- All 5 Build in Public formats present with 1 example each
- FVIA workflow has all 4 sections defined
- Structural rules match Framework_Note_Substack.md
- Calendar matches Strategia_Build_In_Public_2026.md

- [ ] **Step 4: Commit**

```bash
git add skills/substack-notes-writer/references/portali-e-formati.md
git commit -m "feat: add portali and formats reference for substack-notes-writer skill"
```

---

### Task 3: Create `references/algo-substack-2026.md`

**Files:**
- Create: `skills/substack-notes-writer/references/algo-substack-2026.md`

Based on trend research from the previous session. Contains algorithm rules and formatting best practices for Substack Notes in 2026.

- [ ] **Step 1: Create the algorithm reference file**

Create `skills/substack-notes-writer/references/algo-substack-2026.md`:

```markdown
# Algoritmo Substack Notes 2026 — Regole Operative

> Fonte: ricerca trend aprile 2026
> Uso: vincoli di formato e strategia di discovery per Note generate

---

## Come Funziona la Discovery su Substack Notes (2026)

1. **Substantive replies > likes.** L'algoritmo premia le Note che generano risposte di qualità, non solo cuori. Una Nota con 5 commenti sostantivi batte una con 50 like.

2. **1 Nota/giorno è il sweet spot.** Più di 1 al giorno diluisce la reach. Meno di 3 a settimana = invisibilità. Ideale: 3-5 Note/settimana.

3. **5-10 righe = formato ottimale.** Troppo corte (1-2 righe) = percepite come throwaway. Troppo lunghe (15+) = scroll-past. Il nostro target: 5-6 righe.

4. **Le immagini boostan la discovery.** Una Nota con immagine rilevante (non decorativa) ha ~40% più reach. Usare per il formato "Dietro le Quinte con Screenshot".

5. **Engagement nei primi 30 minuti è critico.** Le prime interazioni determinano la distribuzione. Pubblicare quando il proprio pubblico è attivo (per pubblico italiano: 8-9 AM o 12-13 PM).

6. **Le Note che parlano di processo > quelle che danno consigli.** "Oggi ho fatto X e ho scoperto Y" performa meglio di "Dovresti fare X per ottenere Y".

7. **Cross-posting da X/Twitter non funziona.** Note copiate da altri social vengono penalizzate. Il contenuto deve essere nativo Substack.

---

## Vincoli di Formato (hard rules)

- **Zero hashtag.** Substack non li usa. Inserirli segnala "non è di qui".
- **Zero emoji nel corpo testo.** Al massimo 1 come separatore visivo tra sezioni.
- **Nessun link nel corpo.** I link riducono la reach. Se serve rimandare alla newsletter, usare la formula "Ne parlo nella lettera di questa settimana." — Substack linka automaticamente il profilo.
- **No thread lunghi.** A differenza di X, i thread multi-nota non funzionano. Una Nota = un pensiero completo.

---

## Pattern di Contenuto che Performano (Q1-Q2 2026)

- **Mini-storie personali** con dettaglio sensoriale (tempo, luogo, sensazione)
- **Contrarian takes** detti con grazia (non rage bait)
- **Behind the scenes** di processi creativi o di business
- **Domande genuine** che invitano riflessione (non engagement bait tipo "Sei d'accordo?")
- **Dati reali** condivisi con vulnerabilità (numeri veri, non flexing)

## Pattern che Non Funzionano

- Liste di consigli ("3 cose che...")
- Motivazionale generico
- Cross-post da X/Twitter
- Note che chiedono esplicitamente interazione ("Cosa ne pensi? Commenta!")
- Contenuti troppo nicchia senza contesto
```

- [ ] **Step 2: Commit**

```bash
git add skills/substack-notes-writer/references/algo-substack-2026.md
git commit -m "feat: add substack 2026 algorithm reference for substack-notes-writer skill"
```

---

### Task 4: Create `references/trend-research-prompts.md`

**Files:**
- Create: `skills/substack-notes-writer/references/trend-research-prompts.md`

Query templates for the two research sources (WebSearch for Substack, Apify for X/Twitter).

- [ ] **Step 1: Create the trend research prompts file**

Create `skills/substack-notes-writer/references/trend-research-prompts.md`:

```markdown
# Template Query per Ricerca Trend — Riferimento Operativo

> Uso: la skill usa questi template per costruire le query nella Fase 2 (Ricerca Trend)

---

## WebSearch — Trend Substack

Query da eseguire con il tool `WebSearch`. Sostituire `{TEMA}` con l'argomento corrente.

### Query obbligatorie (sempre)

1. **Note virali nella nicchia:**
   `site:substack.com/notes "{TEMA}" 2026`

2. **Newsletter trending nella nicchia:**
   `substack.com best newsletters "{TEMA}" spirituality business AI 2026`

3. **Pattern di formato:**
   `substack notes viral format best practices 2026`

### Query opzionali (se il tema lo richiede)

4. **Tema + AI:**
   `"{TEMA}" AI automation personal business substack 2026`

5. **Tema + spiritualità/astrologia:**
   `"{TEMA}" astrology human design business substack`

---

## Apify — Trend X/Twitter

Query da eseguire con il tool `Apify` (call-actor: `apify/twitter-scraper` o equivalente).
Se Apify non è disponibile, usare `WebSearch` con `site:x.com` come fallback.

### Query obbligatorie (sempre)

1. **Conversazioni trending nella nicchia:**
   Cerca: `{TEMA} AND (business OR entrepreneur) AND (spiritual OR astrology OR "human design") min_faves:50`

2. **Conversazioni AI + business consapevole:**
   Cerca: `AI AND (coach OR consultant OR solopreneur) AND (authentic OR soul OR purpose) min_faves:30`

### Query opzionali

3. **Controversie/dibattiti attivi:**
   Cerca: `{TEMA} AND (unpopular opinion OR hot take OR controversial) min_faves:100`

---

## Fallback: Solo WebSearch (se Apify non disponibile)

Sostituire le query Apify con:

1. `site:x.com "{TEMA}" business spiritual 2026`
2. `site:x.com AI coach entrepreneur authentic 2026`

---

## Come Sintetizzare i Trend

Dopo la ricerca, sintetizza in questo formato (5-8 trend):

```
Trend: [descrizione in una riga]
Angolo GrimorIa: [come GrimorIa ne parlerebbe — quale portale/pilastro]
```

Esempio:
```
Trend: dibattito su AI che sostituisce il coach
Angolo GrimorIa: l'AI non sostituisce, amplifica chi sei già (Portale: Rovesciamento, Pilastro: Shock Amorevole)
```
```

- [ ] **Step 2: Commit**

```bash
git add skills/substack-notes-writer/references/trend-research-prompts.md
git commit -m "feat: add trend research prompts reference for substack-notes-writer skill"
```

---

### Task 5: Create `SKILL.md` — Frontmatter and Philosophy

**Files:**
- Create: `skills/substack-notes-writer/SKILL.md`

This is the main skill file. We build it in stages. This task creates the frontmatter (name, description, trigger phrases) and the philosophy section.

- [ ] **Step 1: Create SKILL.md with frontmatter and philosophy**

Create `skills/substack-notes-writer/SKILL.md`:

```markdown
---
name: substack-notes-writer
description: "Skill per generare Note Substack nel tono di voce GrimorIa. Usa questa skill OGNI VOLTA che l'utente chiede di scrivere Note per Substack, generare contenuti per Substack Notes, creare post brevi per Substack, o cercare trend per scrivere Note. Attiva anche quando l'utente dice 'scrivi note substack', 'genera note', 'note per questa settimana', 'note dalla newsletter', 'cerca trend e scrivi note', 'note su [tema]', 'contenuti substack', 'frammenti dal cantiere', o qualsiasi richiesta che implichi la creazione di Note brevi per Substack nel tono GrimorIa. NON usare per scrivere newsletter complete (usa il framework FVIA manualmente), per analisi di carte natali (usa astro-hd-gk-profiler), o per contenuti non-Substack (Instagram, email, sales page)."
---

# Substack Notes Writer

## Filosofia

**Le Note non sono contenuti. Sono varchi.**

Ogni Nota è una frase singola che apre un portale. Il lettore la legge e sente qualcosa muoversi nel corpo. Non "attenzione catturata" — risonanza attivata.

Questa skill non produce "post per i social". Produce frammenti di verità che reggono da soli, che il lettore salverebbe anche senza sapere chi li ha scritti, e che rispettano il principio GrimorIa: **parla poco, colpisci sempre.**

### Cosa produce questa skill

- **5-7 Note** per ogni esecuzione
- **2 varianti** per ogni Nota (A: intima, B: provocatoria)
- **Scheduling suggerito** (quale giorno per quale Nota)
- **Portale e formato** esplicitati per ogni Nota

### Cosa NON fa questa skill

- Non scrive newsletter (il framework FVIA è un processo separato)
- Non pubblica direttamente su Substack
- Non analizza carte natali (quello è il lavoro di astro-hd-gk-profiler)
```

- [ ] **Step 2: Commit**

```bash
git add skills/substack-notes-writer/SKILL.md
git commit -m "feat: add SKILL.md frontmatter and philosophy for substack-notes-writer"
```

---

### Task 6: Add Phase 1 (Input Analysis) to `SKILL.md`

**Files:**
- Modify: `skills/substack-notes-writer/SKILL.md`

- [ ] **Step 1: Append Phase 1 to SKILL.md**

Append after the philosophy section:

```markdown

---

## Workflow

### Fase 0: Rileva Modalità Input

Rileva automaticamente quale delle 3 modalità l'utente sta usando:

| Modalità | Come riconoscerla | Azione |
|----------|------------------|--------|
| **Tema libero** | L'utente passa un argomento o tema ("scrivi note su...", "note su come uso l'AI") | Vai a Fase 1A |
| **Da Newsletter** | L'utente passa testo lungo o dice "dalla newsletter" | Vai a Fase 1B |
| **Trend Search** | L'utente dice "cerca trend", "cosa funziona adesso", "trend" | Vai a Fase 2 direttamente |

Se non è chiaro, chiedi: "Vuoi che parta da un tema, dalla tua newsletter, o che cerchi trend attuali?"

### Fase 1A: Analisi Tema Libero

1. Estrai le keyword principali dal tema
2. Controlla se il tema si avvicina a uno dei 12 temi Build in Essence:
   1. Come ho scelto il posizionamento di Grimoria
   2. Il mio primo agente AI costruito sulla mia Carta Natale
   3. Perché ho messo il prezzo a 500 euro senza giustificarlo
   4. La settimana in cui ho cancellato tutto dal calendario
   5. Come uso Claude come Funzione Esecutiva Esterna
   6. Il prompt che ha cambiato il modo in cui preparo le sessioni
   7. Cosa succede quando lasci che il tuo Gene Key guidi il tono di voce
   8. I numeri reali del primo mese di Grimoria
   9. Ho automatizzato l'onboarding — cosa ho imparato su cosa NON automatizzare
   10. La decisione più controintuitiva che ho preso questo mese
   11. Come il transito di questa settimana ha cambiato la mia strategia editoriale
   12. 3 mesi di Build in Essence: cosa ha funzionato, cosa no
3. Se il tema è vicino a uno di questi, usa il formato Build in Public corrispondente
4. Se il tema tocca HD/GK/Astrologia E l'utente ha attivato `--rag`: vai a Fase 3
5. Procedi a Fase 4 (Generazione)

### Fase 1B: Analisi Newsletter (FVIA)

Leggi `references/portali-e-formati.md` — sezione "Workflow: Da Newsletter (FVIA) a Note".

1. Identifica le 4 sezioni FVIA nel testo:
   - **F (Ferita):** il punto di partenza reale
   - **V (Verità):** lo shock amorevole emerso dal fare
   - **I (Incarnazione):** cosa è cambiato nel business
   - **A (Accesso):** il prompt, la domanda, il micro-sistema
2. Per ogni sezione, estrai le frasi che superano il test: "La salverei anche se non sapessi chi l'ha scritta?"
3. Seleziona 5-7 frasi totali (almeno 1 per sezione FVIA)
4. Per ogni frase, pre-assegna il Portale di Apertura più naturale
5. Se il contenuto tocca HD/GK/Astrologia E l'utente ha attivato `--rag`: vai a Fase 3
6. Procedi a Fase 4 (Generazione)
```

- [ ] **Step 2: Commit**

```bash
git add skills/substack-notes-writer/SKILL.md
git commit -m "feat: add input analysis phases (1A/1B) to substack-notes-writer"
```

---

### Task 7: Add Phase 2 (Trend Research) to `SKILL.md`

**Files:**
- Modify: `skills/substack-notes-writer/SKILL.md`

- [ ] **Step 1: Append Phase 2 to SKILL.md**

Append after Phase 1B:

```markdown

### Fase 2: Ricerca Trend

Attiva **sempre** per modalità Trend Search. Per le altre modalità, attiva solo se l'utente lo chiede esplicitamente.

Leggi `references/trend-research-prompts.md` per i template delle query.

**Step 2.1 — Ricerca Substack (obbligatoria)**

Esegui le 3 query WebSearch obbligatorie da `trend-research-prompts.md`, sostituendo `{TEMA}` con il tema corrente. Per ogni risultato rilevante, usa `WebFetch` per leggere il contenuto.

**Step 2.2 — Ricerca X/Twitter (se Apify disponibile)**

Usa il tool MCP `Apify` con `call-actor` per eseguire le query X/Twitter da `trend-research-prompts.md`.

Se Apify non è disponibile: usa le query fallback WebSearch (`site:x.com`).

**Step 2.3 — Sintetizza i Trend**

Produci un brief di 5-8 trend nel formato:

```
Trend: [descrizione in una riga]
Angolo GrimorIa: [come GrimorIa ne parlerebbe — portale + pilastro suggeriti]
```

Mostra il brief all'utente prima di procedere. Chiedi: "Questi trend ti risuonano? Vuoi che ne escluda o aggiunga qualcuno?"

Dopo la conferma, procedi a Fase 4 (o Fase 3 se RAG attivo).
```

- [ ] **Step 2: Commit**

```bash
git add skills/substack-notes-writer/SKILL.md
git commit -m "feat: add trend research phase (2) to substack-notes-writer"
```

---

### Task 8: Add Phase 3 (RAG) to `SKILL.md`

**Files:**
- Modify: `skills/substack-notes-writer/SKILL.md`

- [ ] **Step 1: Append Phase 3 to SKILL.md**

Append after Phase 2:

```markdown

### Fase 3: Arricchimento RAG (condizionale)

Attiva SOLO SE:
- L'utente ha passato `--rag` o ha detto "usa la knowledge base"
- Il tema tocca esplicitamente HD, Gene Keys, Astrologia, transiti

Se il tool MCP `grimoria-rag` (`search_knowledge_base`) non è disponibile, salta questa fase con un avviso:
> "La knowledge base non è disponibile in questa sessione. Procedo con la generazione senza arricchimento RAG."

**Regole:**
- Massimo 2 query a `search_knowledge_base`
- Ogni query deve essere specifica, non generica
- Domain mapping: HD/GK → `domain: "human_design"`, Vedica → `domain: "jyotish"`

**Esempi di query mirate:**
- Se il tema è "come uso il mio canale 25-51 nel business" → `query: "Channel 25-51 business strategy", domain: "human_design"`
- Se il tema è "il mio Gene Key del Sole" → `query: "Gene Key [N] shadow gift siddhi", domain: "human_design"`
- Se il tema è "il transito di questa settimana" → `query: "[pianeta] transit [segno] business", domain: "human_design"`

**Output:** 1-3 insight specifici da intrecciare nelle Note. L'insight deve essere integrato nella prosa della Nota, NON appeso come "Secondo il mio HD..." — deve emergere naturalmente.

Procedi a Fase 4.
```

- [ ] **Step 2: Commit**

```bash
git add skills/substack-notes-writer/SKILL.md
git commit -m "feat: add RAG enrichment phase (3) to substack-notes-writer"
```

---

### Task 9: Add Phase 4 (Note Generation) to `SKILL.md`

**Files:**
- Modify: `skills/substack-notes-writer/SKILL.md`

This is the core generation logic — the longest section.

- [ ] **Step 1: Append Phase 4 to SKILL.md**

Append after Phase 3:

```markdown

### Fase 4: Generazione Note

Leggi `references/brand-voice-compact.md` e `references/portali-e-formati.md` PRIMA di generare.
Leggi `references/algo-substack-2026.md` per i vincoli di formato.

Per ogni Nota (genera 5-7 totali):

**Step 4.1 — Seleziona il Portale**

Scegli tra i 6 Portali di Apertura (da `portali-e-formati.md`) quello più adatto al contenuto. Distribuisci i portali: non usare lo stesso portale per più di 2 Note nella stessa batch.

**Step 4.2 — Seleziona il Formato (se Build in Public)**

Se il contenuto documenta il processo di costruzione di GrimorIa, assegna uno dei 5 formati Build in Public (da `portali-e-formati.md`). Altrimenti, il formato è "libero".

**Step 4.3 — Genera Variante A (intima)**

Scrivi la Nota con tono verso **Calore con Spina Dorsale** o **Vulnerabilità Incarnata**:

- Prima frase = portale selezionato (apre il varco)
- 2-3 righe di espansione — non spiegare, approfondisci
- Totale: 5-6 righe max
- Zero CTA
- Vocabolario da `brand-voice-compact.md`

**Step 4.4 — Genera Variante B (provocatoria)**

Riscrivi la stessa Nota con tono verso **Shock Amorevole** o **Regale ma Mai Distante**:

- Stesso nucleo di contenuto, angolo diverso
- Stessi vincoli strutturali (5-6 righe, zero CTA, vocabolario GrimorIa)

**Step 4.5 — Validazione Post-Generazione**

Per ogni Nota generata (entrambe le varianti), esegui questi check:

1. **Check vocabolario:** Cerca parole proibite (lista in `brand-voice-compact.md`). Se trovate, riscrivi la frase.
2. **Check anti-pattern:** Confronta con la tabella anti-pattern in `brand-voice-compact.md`. Se la Nota cade in un pattern vietato, riscrivila.
3. **Check lunghezza:** Se supera 6 righe, taglia. Non esiste "un po' più lunga".
4. **Check prima frase:** La prima frase deve funzionare da sola come post. Se ha bisogno di contesto, riscrivila.
5. **Check formato:** Zero hashtag, zero emoji nel corpo, nessun link esplicito, italiano naturale, nessun "tu dovresti".
6. **Test di autenticità (3 su 4 devono essere sì):**
   - La salverei se la leggessi nel feed di qualcun altro?
   - Funziona senza contesto?
   - C'è una verità non ancora nominata?
   - La pubblicherei anche con zero like?

Se una Nota non passa il test di autenticità (meno di 3 sì), scartala e generane una nuova.
```

- [ ] **Step 2: Commit**

```bash
git add skills/substack-notes-writer/SKILL.md
git commit -m "feat: add note generation phase (4) with validation to substack-notes-writer"
```

---

### Task 10: Add Phase 5 (Packaging and Output) to `SKILL.md`

**Files:**
- Modify: `skills/substack-notes-writer/SKILL.md`

- [ ] **Step 1: Append Phase 5 to SKILL.md**

Append after Phase 4:

```markdown

### Fase 5: Packaging e Output

Assembla l'output finale nel seguente formato. Presenta ALL'UTENTE (non salvare in file — il contenuto è per uso immediato).

```
## Note Generate — [Tema] — [Data di oggi]

### Nota 1 — Portale: [nome portale] | Formato: [nome formato o "libero"]

**Variante A (intima):**
[testo della nota, 5-6 righe]

**Variante B (provocatoria):**
[testo della nota, 5-6 righe]

**Scheduling suggerito:** [giorno] — [motivazione breve]
**Pilastro dominante:** [quale dei 4 pilastri]

---

[... ripeti per tutte le 5-7 note ...]

---

### Piano Pubblicazione Settimanale

| Giorno | Nota # | Portale | Formato | Motivazione |
|--------|--------|---------|---------|-------------|
| Mercoledì | #1 | [portale] | [formato] | Teaser post-newsletter |
| Giovedì | #3 | [portale] | [formato] | [motivazione] |
| Venerdì | #5 | [portale] | [formato] | Shock di chiusura settimana |

### Note per l'utente
- Scegli la variante (A o B) che senti più tua per ogni Nota
- Puoi mescolare: Variante A per la #1, Variante B per la #3, ecc.
- Se una Nota rimanda alla newsletter: "Ne parlo nella lettera di questa settimana."
- Se vuoi aggiungere un'immagine: usa il formato "Dietro le Quinte" con screenshot del tuo workspace
```

**Dopo aver presentato l'output**, chiedi:
> "Vuoi che modifichi qualche Nota? Posso riscrivere una variante, cambiare portale, o generarne di nuove su un angolo specifico."
```

- [ ] **Step 2: Commit**

```bash
git add skills/substack-notes-writer/SKILL.md
git commit -m "feat: add packaging and output phase (5) to substack-notes-writer"
```

---

### Task 11: End-to-End Validation

**Files:**
- Read: `skills/substack-notes-writer/SKILL.md`
- Read: all 4 files in `skills/substack-notes-writer/references/`

- [ ] **Step 1: Read the complete SKILL.md**

Read the full file. Verify:
- Frontmatter has name, description with trigger phrases
- Philosophy section present
- All 5 phases present and in order (0 → 1A/1B → 2 → 3 → 4 → 5)
- Each phase references the correct `references/` file
- No orphan references (every file mentioned in SKILL.md exists in references/)

- [ ] **Step 2: Cross-check references**

For each `references/` file:
- `brand-voice-compact.md` — referenced in Phase 4 (Steps 4.3, 4.4, 4.5)
- `portali-e-formati.md` — referenced in Phase 1B, Phase 4 (Steps 4.1, 4.2)
- `algo-substack-2026.md` — referenced in Phase 4 (header)
- `trend-research-prompts.md` — referenced in Phase 2

Verify no file is unreferenced and no reference points to a nonexistent file.

- [ ] **Step 3: Smoke test — simulate Tema Libero mode**

Mentally walk through the skill with input: "Scrivi note su come uso l'AI nel mio business"
- Fase 0: detects "Tema libero" → Fase 1A
- Fase 1A: keyword "AI nel business", matches theme #5 (Claude come Funzione Esecutiva Esterna) → Build in Public format
- Fase 2: skipped (user didn't ask for trends)
- Fase 3: skipped (no --rag, tema non tocca HD/GK/Astro)
- Fase 4: generates 5-7 Notes with portals and variants
- Fase 5: packages with scheduling

Confirm the flow is complete with no dead ends.

- [ ] **Step 4: Smoke test — simulate Da Newsletter mode**

Input: user pastes 500 words of FVIA newsletter text
- Fase 0: detects long text → "Da Newsletter" → Fase 1B
- Fase 1B: identifies FVIA sections, extracts 5-7 candidate phrases
- Fase 2: skipped
- Fase 3: skipped
- Fase 4: generates Notes from extracted phrases
- Fase 5: packages

Confirm flow is complete.

- [ ] **Step 5: Smoke test — simulate Trend Search mode with --rag**

Input: "Cerca trend e scrivi note --rag"
- Fase 0: detects "trend" → Fase 2
- Fase 2: WebSearch + Apify queries → trend brief → user confirmation
- Fase 3: RAG active → 2 queries to grimoria-rag
- Fase 4: generates Notes from trends + RAG insights
- Fase 5: packages

Confirm flow is complete.

- [ ] **Step 6: Final commit**

```bash
git add skills/substack-notes-writer/
git commit -m "feat: complete substack-notes-writer skill with all references and phases"
```

---

## Summary

| Task | What | Files | Est. |
|------|------|-------|------|
| 1 | Brand voice compact reference | `references/brand-voice-compact.md` | 3 min |
| 2 | Portali + formati reference | `references/portali-e-formati.md` | 3 min |
| 3 | Algorithm reference | `references/algo-substack-2026.md` | 2 min |
| 4 | Trend research prompts | `references/trend-research-prompts.md` | 2 min |
| 5 | SKILL.md frontmatter + philosophy | `SKILL.md` | 2 min |
| 6 | Phase 1 (input analysis) | `SKILL.md` | 3 min |
| 7 | Phase 2 (trend research) | `SKILL.md` | 2 min |
| 8 | Phase 3 (RAG) | `SKILL.md` | 2 min |
| 9 | Phase 4 (generation + validation) | `SKILL.md` | 3 min |
| 10 | Phase 5 (packaging + output) | `SKILL.md` | 2 min |
| 11 | End-to-end validation | All files | 5 min |
