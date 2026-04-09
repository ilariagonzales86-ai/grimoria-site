# Substack Notes Writer — Design Spec

> Skill monolitica per generare Note Substack nel tono di voce GrimorIa,
> con ricerca trend, integrazione RAG opzionale, e varianti per ogni Nota.

---

## 1. Architettura della Skill

### Struttura File

```
skills/substack-notes-writer/
├── SKILL.md                          # Skill principale (logica, workflow, regole)
├── substack-notes-writer.skill       # Registro skill (puntatore)
└── references/
    ├── brand-voice-compact.md        # Estratto compatto delle brand guidelines
    ├── portali-e-formati.md          # I 6 Portali + 5 formati Build in Public
    ├── algo-substack-2026.md         # Regole algoritmo Substack aggiornate
    └── trend-research-prompts.md     # Template per query di ricerca trend
```

### Input della Skill

La skill accetta 3 modalità di input, rilevate automaticamente:

| Modalità | Trigger | Esempio |
|----------|---------|---------|
| **Tema libero** | L'utente passa un argomento | "Scrivi note su come uso l'AI nel business" |
| **Da Newsletter** | L'utente passa testo o link | "Ecco la newsletter di questa settimana: [testo]" |
| **Trend Search** | L'utente chiede di cercare trend | "Cerca trend e scrivi note" |

### Output

Per ogni esecuzione:

- **5-7 Note** (ciascuna con **2 varianti**: una più intima, una più provocatoria)
- Ogni Nota: 5-6 righe max, nessuna CTA esplicita
- **Suggerimento di scheduling**: quale Nota pubblicare quale giorno (Mer/Gio/Ven)
- **Portale usato** indicato per ogni Nota (es. "Dichiarazione Sovrana")
- **Formato Build in Public** indicato dove applicabile

### Integrazione RAG (opzionale)

Quando l'utente attiva il flag `--rag` o il tema tocca HD/GK/Astrologia:

- Query a `grimoria-rag` (MCP: `search_knowledge_base`) per estrarre insight specifici
- L'insight RAG viene intrecciato nella Nota, non appeso come decorazione

---

## 2. Pipeline e Flusso di Generazione

### Il flusso in 5 fasi

```
INPUT → [1. Analisi] → [2. Ricerca] → [3. RAG] → [4. Generazione] → [5. Packaging] → OUTPUT
```

### Fase 1 — Analisi Input

La skill rileva automaticamente la modalità:

- **Tema libero**: estrae keyword principali, identifica quale dei 12 temi Build in Essence è più vicino (se nessuno, procede libero)
- **Da Newsletter**: scansiona il testo, estrae 5-7 frasi-chiave che superano il test "la salverei anche se non sapessi chi l'ha scritta?", identifica il FVIA di appartenenza (Ferita/Verità/Incarnazione/Accesso)
- **Trend Search**: salta direttamente alla Fase 2

### Fase 2 — Ricerca Trend

Attiva **sempre** per modalità Trend Search, **opzionale** per le altre due (l'utente può dire "senza trend" per saltare).

Due fonti in parallelo:

| Fonte | Tool | Cosa cerca |
|-------|------|------------|
| **Substack** | `WebSearch` + `WebFetch` su substack.com | Note virali nella nicchia spiritualità x business x AI, pattern di formato, temi caldi |
| **X/Twitter** | `Apify` (call-actor con Twitter scraper) | Conversazioni trending su imprenditorialità consapevole, AI x spiritualità, Human Design business |

**Output Fase 2**: un brief di 5-8 trend sintetizzati in una riga ciascuno, con angolo GrimorIa suggerito. Esempio:

> "Trend: dibattito su AI che sostituisce il coach → Angolo GrimorIa: l'AI non sostituisce, amplifica chi sei già"

### Fase 3 — RAG (condizionale)

Attiva solo se:

- L'utente passa `--rag` o dice "usa la knowledge base"
- Il tema tocca esplicitamente HD, Gene Keys, Astrologia, transiti

**Query**: la skill formula una domanda mirata a `grimoria-rag` basata sul tema. Non query generiche — query specifiche tipo:

> "Qual è il significato del Canale 25-51 in Human Design per il business?"

**Output Fase 3**: 1-3 insight specifici da intrecciare nelle Note, con citazione della fonte.

### Fase 4 — Generazione Note

Il cuore della skill. Per ogni Nota generata:

1. **Seleziona il Portale** — sceglie tra i 6 Portali di Apertura quello più adatto al contenuto
2. **Seleziona il Formato** — se il contenuto è Build in Public, assegna uno dei 5 formati (Flash dal Laboratorio, Decisione Nuda, Numero Vero, Frase-Lama, Dietro le Quinte)
3. **Genera la Nota** — applica le regole:
   - Prima frase = portale (apre un varco)
   - Espansione in 2-3 righe max
   - Totale: 5-6 righe (allineato con sweet spot algoritmo 2026: 5-10 righe)
   - Zero CTA esplicite
   - Vocabolario GrimorIa (mai: tips, hack, funnel, strategia, offerta)
4. **Genera Variante B** — stessa Nota, angolo diverso:
   - Variante A = più intima (Calore con Spina Dorsale)
   - Variante B = più provocatoria (Shock Amorevole)
5. **Test di qualità** — verifica automatica contro le 4 domande del framework:
   - La salverei se la leggessi nel feed di qualcun altro?
   - Funziona senza contesto?
   - C'è una verità non ancora nominata?
   - La pubblicherei anche con zero like?

### Fase 5 — Packaging

Assembla l'output finale:

```
## Note Generate — [Tema/Data]

### Nota 1 — [Portale: Dichiarazione Sovrana] [Formato: Frase-Lama]
**Variante A (intima):**
[testo nota]

**Variante B (provocatoria):**
[testo nota]

**Scheduling suggerito:** Mercoledì (teaser newsletter)

---
[... ripeti per 5-7 note ...]

### Piano Pubblicazione
| Giorno | Nota | Portale | Perché questo giorno |
|--------|------|---------|---------------------|
| Mer    | #1   | ...     | Teaser post-newsletter |
| Gio    | #3   | ...     | Flash dal Laboratorio   |
| Ven    | #5   | ...     | Shock di chiusura settimana |
```

---

## 3. Regole di Brand Voice e Guardrail

### Il filtro vocale: 3 livelli

Ogni Nota passa attraverso tre livelli di filtro prima di essere presentata.

#### Livello 1 — Vocabolario

Regole hard-coded nel file `references/brand-voice-compact.md`:

| Mai usare | Usa invece |
|-----------|------------|
| tips, trucchi | verità, chiavi |
| strategia | visione architettata |
| funnel | campo, ecosistema |
| target | le persone che risuonano |
| offerta, prodotto | mondo, esperienza, iniziazione |
| vendita | l'atto di aprire la porta |
| scalare, crescere (gergo) | espandere, radicarsi |
| mindset | coscienza, consapevolezza |
| empowerment | sovranità |
| hack | alchimia |
| monetizzazione | come fluisce la prosperità |

La skill esegue un check automatico post-generazione: se una parola proibita appare nella Nota, la riscrive prima di presentarla.

#### Livello 2 — Pilastri tonali

Ogni Nota deve incarnare almeno 1 dei 4 pilastri. La skill li assegna esplicitamente:

| Pilastro | Quando si attiva | Segnale nel testo |
|----------|-----------------|-------------------|
| **Calore con Spina Dorsale** | Vulnerabilità, momenti personali | Prima persona, dettaglio sensoriale, nessuna giustificazione |
| **Regale ma Mai Distante** | Dichiarazioni, posizionamento | Frasi brevi, tono sicuro, nessun "secondo me" |
| **Shock Amorevole** | Rovesciamenti, domande scomode | Controintuitivo, amorevole nella brutalità |
| **Parla Poco Colpisci Sempre** | Chiusure, frasi-lama | Massima densità, zero parole superflue |

La Variante A tende verso Calore con Spina Dorsale o Vulnerabilità Incarnata.
La Variante B tende verso Shock Amorevole o Regale ma Mai Distante.

#### Livello 3 — Anti-pattern

La skill rifiuta automaticamente Note che cadono in questi pattern:

| Anti-pattern | Esempio | Perché è vietato |
|-------------|---------|-----------------|
| **La lista di consigli** | "3 modi per..." | GrimorIa non dà tips. Apre varchi. |
| **Il motivazionale generico** | "Credi in te stessa!" | Già sentito mille volte. Zero risonanza. |
| **La CTA mascherata** | "Se vuoi saperne di più..." | Le Note non vendono. Mai. |
| **Il jargon spirituale vuoto** | "Alza la tua vibrazione" | Spiritualità incarnata, non decorativa. |
| **Il flex travestito da vulnerabilità** | "Ero al verde, ora faturo 6 cifre" | Build in Essence, non Build in Ego. |
| **L'astro-generico** | "Mercurio retrogrado, attenzione!" | Se parli di astro, parli del TUO chart, del TUO transito specifico. |

### Guardrail strutturali

Regole applicate a ogni Nota generata, senza eccezioni:

1. **Max 6 righe.** Se supera, taglia. Non esiste la Nota "un po' più lunga".
2. **Prima frase autonoma.** Deve funzionare come post a sé. Se serve contesto per capirla, riscrivila.
3. **Zero hashtag.** Algoritmo Substack 2026 non li usa.
4. **Zero emoji nel corpo.** Al massimo 1 emoji come separatore visivo, mai nel testo.
5. **Nessun link esplicito.** Se rimanda alla newsletter: "Ne parlo nella lettera di questa settimana." Substack linka il profilo automaticamente.
6. **Italiano naturale.** Niente anglicismi gratuiti. "Insight" sì (è entrato nell'uso), "mindset" mai.
7. **Nessun "tu dovresti".** GrimorIa non prescrive. Dichiara, domanda, mostra. Il lettore sceglie.

### Il file `references/brand-voice-compact.md`

Estratto operativo (non copia integrale) delle brand guidelines. Contiene:

- Tabella Si/No completa (22 sostituzioni)
- I 4 pilastri con esempio di frase per ciascuno
- Le 5 espressioni firma (apertura, chiusura, sales, shock, connessione)
- La checklist anti-pattern
- Il test di autenticità in 4 domande

Circa 150-200 righe — compatto per il contesto, ricco per la generazione.

---

## 4. Integrazione con Documenti Esistenti e Dipendenze

### Mappa delle dipendenze

```
substack-notes-writer/
    |
    +-- LEGGE DA (fonte di verita)
    |   +-- Grimoria Analisi e Branding/02_Brand/brand-voice-guidelines.md
    |   +-- Grimoria Analisi e Branding/Substack/Framework_Note_Substack.md
    |   +-- Grimoria Analisi e Branding/Substack/Strategia_Substack_Build_In_Public_Grimoria_2026.md
    |
    +-- DISTILLA IN (references/ compatti per il contesto)
    |   +-- references/brand-voice-compact.md    <- da brand-voice-guidelines.md
    |   +-- references/portali-e-formati.md      <- da Framework_Note + Strategia_Build_In_Public
    |   +-- references/algo-substack-2026.md     <- da ricerca trend (nuova)
    |
    +-- USA COME TOOL (MCP/runtime)
    |   +-- grimoria-rag (search_knowledge_base) <- per insight HD/GK/Astro
    |   +-- Apify (call-actor)                   <- per scraping X/Twitter
    |   +-- WebSearch + WebFetch                 <- per trend Substack
    |
    +-- COESISTE CON (altre skill)
        +-- skills/astro-hd-gk-profiler/         <- non si sovrappone, si complementa
```

### Relazione tra documenti fonte e references/

I file in `references/` sono estratti operativi, non copie. La skill deve funzionare leggendo solo i suoi `references/`, senza mai aprire i documenti originali durante l'esecuzione.

| Documento fonte | Cosa viene estratto | File reference |
|----------------|--------------------|--------------------|
| `brand-voice-guidelines.md` (~500 righe) | Tabella Si/No, 4 pilastri con esempi, espressioni firma, test autenticità | `brand-voice-compact.md` (~150 righe) |
| `Framework_Note_Substack.md` (138 righe) | I 6 Portali con struttura ed esempi, regole Note, test della Nota | `portali-e-formati.md` (~120 righe) |
| `Strategia_Build_In_Public_2026.md` (240 righe) | I 5 formati Build in Public con esempi, calendario tipo, FVIA -> Note workflow | Merge in `portali-e-formati.md` |
| Ricerca trend (sessione precedente) | Regole algoritmo 2026, sweet spot formato, best practice discovery | `algo-substack-2026.md` (~60 righe) |

### Perché distillare e non referenziare direttamente

1. **Token budget.** I documenti originali sommati fanno ~900 righe. I references compressi fanno ~330 righe. Meno della metà del contesto.
2. **Precisione.** I documenti originali contengono strategia, motivazioni, piani a 3 mesi. I references contengono solo le regole operative.
3. **Manutenibilità.** Se aggiorni le brand guidelines, aggiorni il compact in un punto solo.

### Coesistenza con `astro-hd-gk-profiler`

| | `astro-hd-gk-profiler` | `substack-notes-writer` |
|---|---|---|
| **Input** | Chart/immagini di Carta Natale, Bodygraph, Gene Keys | Tema, testo newsletter, o "cerca trend" |
| **Output** | Profilo strategico completo (documento lungo) | 5-7 Note brevi con varianti |
| **Quando** | Analisi profonda di una persona | Produzione contenuti settimanale |
| **RAG** | Non usa grimoria-rag | Usa grimoria-rag (opzionale) |

### Dipendenze runtime

| Dipendenza | Tipo | Obbligatoria? |
|-----------|------|---------------|
| `WebSearch` | Tool nativo Claude Code | Si (per trend Substack) |
| `WebFetch` | Tool nativo Claude Code | Si (per leggere contenuti) |
| `Apify` MCP (`call-actor`) | MCP configurato in `.mcp.json` | No — fallback su solo WebSearch se non disponibile |
| `grimoria-rag` MCP (`search_knowledge_base`) | MCP configurato in `.mcp.json` | No — attivo solo con flag `--rag` |

**Fallback strategy:** Se Apify non è disponibile, la Fase 2 (Ricerca Trend) usa solo WebSearch. Se grimoria-rag non è disponibile, la Fase 3 viene saltata con un avviso. La skill funziona sempre, anche con tool ridotti.
