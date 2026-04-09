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
