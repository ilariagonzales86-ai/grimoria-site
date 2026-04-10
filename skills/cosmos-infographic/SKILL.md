---
name: cosmos-infographic
description: "Skill per generare infografiche e visual nel sistema visivo Cosmos di GrimorIa. Usa questa skill OGNI VOLTA che l'utente chiede di creare un'infografica, un visual, una mappa visiva, una grafica per Instagram o Substack, un diagramma di Human Design o Gene Keys o astrologia, un visual per pitch o presentazione client, una chart esoterica, una mappa del profilo energetico, un poster visivo, un visual da condividere sui social o da mostrare a un cliente. Attiva anche quando l'utente dice 'crea un'infografica', 'fammi un visual', 'voglio una grafica', 'diagramma HD', 'mappa visiva del profilo GK', 'visual per Instagram', 'grafica per Substack', 'infografica zodiacale', 'visual per pitch', 'crea un poster', 'grafica del corpo grafico', 'mappa astrologica visiva', 'visual per il cliente', o qualsiasi richiesta che implichi tradurre dati esoterici, profilazioni energetiche o contenuti strategici in una composizione visiva. NON usare per: analisi del tema natale o del bodygraph (usa astro-hd-gk-profiler), produzione video (usa remotion-video-pipeline), editing di immagini esistenti, o qualsiasi richiesta che non richieda la creazione di un nuovo file HTML visivo."
---

# Cosmos Infographic

## Filosofia

**Il vuoto è sacro. La geometria è linguaggio. La luce emerge dal centro.**

Il sistema visivo Cosmos non è un tema grafico — è una grammatica. Ogni infografica è costruita come si costruisce un mandala: dalla radice verso la periferia, con ogni elemento che porta significato e non solo decorazione.

I principi fondamentali:

- **Il vuoto respira:** lo spazio vuoto non è assenza. È tensione, potenziale, pausa necessaria prima della rivelazione. Non riempire mai una composizione per paura del silenzio.
- **I centri luminosi:** ogni infografica ha uno o più punti di massima densità luminosa. Da lì si irradia tutto il resto. Nessuna composizione è piatta — c'è sempre una gerarchia energetica.
- **L'organico dentro il geometrico:** le forme sacre (cerchio, vesica, griglia) contengono elementi vivi: gradienti che respirano, curvature che evocano corpi e galassie, texture che sembrano materia vivente.
- **Mai piatto, mai generico:** anche il più semplice data label nel sistema Cosmos porta una vibrazione specifica. Il font, il peso, il colore, il posizionamento — tutto è semantico.

Il mood di default è **Obsidian** (oscurità profonda, centri luminosi, palette fredda-calda). Il mood **Pearl** (luce lattea, geometria eterea) si usa per destinazioni client o pitch di alto livello.

---

## Workflow

### Step 0: Leggi i File di Riferimento

Prima di comporre qualsiasi infografica, leggi questi file:

| File | Contenuto | Quando Leggerlo |
|------|-----------|-----------------|
| `references/cosmos-visual-dna.md` | Il DNA visivo completo: palette, tipografia, texture, regole compositive | Sempre, prima di ogni infografica |
| `references/component-library.md` | Tutti i primitivi strutturali e di contenuto con codice HTML/SVG | Per selezionare e costruire i componenti |
| `references/semantic-rules.md` | Regole di accuratezza semantica per dati astrologici, HD e GK | Ogni volta che il contenuto include dati esoterici |

Se il contenuto richiede verifica di dati astrologici, Human Design o Gene Keys, consulta anche:
- `skills/astro-hd-gk-profiler/references/prompt-astrologica.md`
- `skills/astro-hd-gk-profiler/references/human-design-framework.md`
- `skills/astro-hd-gk-profiler/references/gene-keys-framework.md`
- `skills/astro-hd-gk-profiler/references/synthesis-framework.md`

---

### Step 1: INTERPRETA — Analisi del Prompt

Analizza la richiesta dell'utente e identifica i seguenti parametri. Se non esplicitati, usa i default indicati.

| Parametro | Come rilevarlo | Default |
|-----------|---------------|---------|
| **Soggetto** | Cosa deve rappresentare l'infografica (profilo HD, carta astrologica, concetto strategico, quote, dati...) | Contenuto principale della richiesta |
| **Formato** | Dimensioni e orientamento | 1080×1350 px verticale (Instagram/Substack) |
| **Mood** | Obsidian (scuro) o Pearl (chiaro) | Claude sceglie in base al contesto: Obsidian per social, Pearl per client/pitch |
| **Destinazione** | social / client / pitch | social |
| **Lingua** | Italiano o inglese per i testi nell'infografica | Italiano |

**Formati disponibili:**
- `1080×1350` — verticale, Instagram feed / Substack cover (default)
- `1080×1080` — quadrato, Instagram carosello / standalone
- `1920×1080` — slide orizzontale, pitch / presentazione / Notion

**Livello di rigore semantico per destinazione:**
- `social` — priorità estetica e impatto immediato; dati esoterici possono essere evocativi
- `client` — dati devono essere precisi e verificati; stile sobrio ed elevato
- `pitch` — massima precisione; ogni dato deve essere difendibile; layout ispirato a strategic deck

Se manca il soggetto o è ambiguo, chiedi prima di procedere.

---

### Step 2: SELEZIONA — Scegli i Primitivi

Scegli **massimo 3 primitivi strutturali** e i **primitivi di contenuto** necessari. Meno è più.

**Primitivi strutturali disponibili:**

| Primitivo | Quando usarlo |
|-----------|--------------|
| **Orb** | Fulcro energetico centrale; per profili personali, concetti singoli, portali |
| **Totem** | Sequenza verticale di elementi; per journey, fasi, progressioni |
| **Aura Ring** | Anello o aureola attorno a un centro; per enfatizzare un tema o personaggio |
| **Vesica** | Intersezione di due cerchi; per integrazioni, duali, punti di incontro tra sistemi |
| **Sacred Grid** | Griglia geometrica sacra (Fiore della Vita, griglia esagonale...); per mappe, sistemi complessi |
| **Bioform** | Forma organica, fluida, non geometrica; per energie morbide, transizioni, flussi |
| **Tech Blueprint** | Linee sottili, schema tecnico; per sistemi, architetture, framework strategici |

**Primitivi di contenuto disponibili:**

| Primitivo | Contenuto | Quando usarlo |
|-----------|-----------|--------------|
| **Zodiac Wheel** | Ruota zodiacale semplificata | Profili astrologici, transiti, carte natali visualizzate |
| **HD Bodygraph Mini** | Schema corporeo HD ridotto | Qualsiasi contenuto Human Design |
| **GK Spectrum Bar** | Barra Ombra-Dono-Siddhi per un Gene Key | Contenuto Gene Keys |
| **Data Label** | Etichetta con dato + titolo + valore | KPI, numeri, statistiche, highlight di dati |
| **Pull Quote** | Citazione ampia con stile tipografico forte | Frasi chiave, titoli evocativi, headline |

**Guida alla selezione per tipo di contenuto:**

| Tipo di contenuto | Primitivi strutturali consigliati | Primitivi di contenuto consigliati |
|-------------------|----------------------------------|-----------------------------------|
| Profilo Human Design | Orb + Aura Ring | HD Bodygraph Mini + Data Label |
| Profilo Gene Keys | Totem o Vesica | GK Spectrum Bar + Pull Quote |
| Carta astrologica visiva | Sacred Grid + Orb | Zodiac Wheel + Data Label |
| Concetto strategico / insight | Orb o Bioform | Pull Quote + Data Label |
| Framework / sistema | Tech Blueprint + Sacred Grid | Data Label + Pull Quote |
| Profilo integrato (HD+GK+Astro) | Vesica + Totem | Tutte e 3 le chart mini + Pull Quote |
| Quote / frase evocativa | Orb o Bioform | Pull Quote |
| Pitch / slide strategica | Tech Blueprint | Data Label + Pull Quote |

---

### Step 3: ARRICCHISCI — Verifica Semantica

Per contenuti che includono dati astrologici, HD o GK:

1. **Verifica l'accuratezza:** Consulta i file di riferimento di `astro-hd-gk-profiler` per confermare la correttezza tecnica dei dati (segni, porte, canali, Gene Keys, linee, sfere).
2. **Applica le regole semantiche:** Leggi `references/semantic-rules.md` per le convenzioni visive specifiche (es. come rappresentare i centri HD definiti vs indefiniti, come colorare i gradi zodiacali).
3. **Non inventare dati:** Se l'utente non ha fornito dati precisi e il contenuto è di tipo `client` o `pitch`, chiedi conferma prima di procedere.
4. **Per `social`:** i dati evocativi (es. "Gene Key 22 — Grazia") sono accettabili senza conferma tecnica se il soggetto non è una persona specifica.

---

### Step 4: COMPONI — Assembla l'HTML

Leggi il template base da `templates/` e inserisci i primitivi selezionati. Segui queste regole compositive:

**Struttura dell'HTML output:**
- Un singolo file `.html` self-contained
- CSS inline nel tag `<style>` nell'`<head>`
- SVG inline nel `<body>` (nessuna immagine esterna)
- Google Fonts caricato via `<link>` nell'`<head>`
- Nessuna dipendenza esterna oltre a Google Fonts

**Regole di composizione:**

1. **Asse centrale:** ogni infografica ha un asse verticale implicito. I centri luminosi principali si allineano sull'asse o si bilanciamo simmetricamente.
2. **Gerarchia in 3 livelli:** Livello 1 = elemento dominante (il soggetto principale), Livello 2 = elementi secondari di supporto, Livello 3 = texture e dettagli decorativi.
3. **Spazio respirante:** tra ogni sezione di contenuto, includi spazio vuoto equivalente ad almeno 1/5 dell'altezza del contenuto soprastante.
4. **Ordine dei layer (dal basso all'alto):**
   - Background (gradiente o campo di colore)
   - Texture geometrica / Sacred Grid sottile
   - Primitivi strutturali (Orb, Totem, ecc.)
   - Primitivi di contenuto (chart, label, quote)
   - Testo primario e titolo
   - Accenti luminosi e highlight

**Palette Obsidian (default):**
- Background: `#050510` o `#0A0A1A`
- Centri luminosi: da `#8B5CF6` (viola profondo) a `#E2D9FF` (lavanda chiara)
- Accenti caldi: `#F59E0B` (oro) o `#EC4899` (rosa cosmico)
- Testo primario: `#F8F7FF`
- Testo secondario: `#9CA3AF`

**Palette Pearl (client/pitch):**
- Background: `#FAFAF8` o `#F5F0EC`
- Centri luminosi: `#E8E0F0` con accento `#7C3AED`
- Testo primario: `#1A1A2E`
- Accenti: `#B8860B` (oro antico) o `#6B21A8` (viola profondo)

---

### Step 5: VERIFICA — Checklist Anti-Pattern

Prima di consegnare, esegui questi check:

**Check anti-pattern visivi:**
- [ ] Nessun elemento "galleggia" isolato senza relazione compositiva con gli altri
- [ ] Il testo non è troppo piccolo (minimo 14px per testi di supporto, 20px per body, 32px+ per titoli)
- [ ] Nessun gradiente piatto o monocromatico — i centri luminosi devono avere profondità
- [ ] Nessuna composizione simmetrica perfetta e robotica — c'è sempre una tensione dinamica
- [ ] Non più di 3 font weights nella stessa infografica
- [ ] Nessun elemento che "sborda" oltre il canvas senza intenzione compositiva

**Check leggibilità:**
- [ ] Il contrasto testo/sfondo è sufficiente (WCAG AA come minimo)
- [ ] Il titolo principale è leggibile anche a dimensione ridotta (anteprima social)
- [ ] La gerarchia visiva è chiara: si capisce immediatamente qual è l'elemento principale

**Check accuratezza semantica (per contenuti esoterici):**
- [ ] I simboli zodiacali usati corrispondono ai segni menzionati
- [ ] I numeri di porte/canali HD sono corretti
- [ ] I Gene Keys sono numerati correttamente (1-64)
- [ ] I colori dei centri HD riflettono le convenzioni standard (se pertinente)
- [ ] Nessun dato inventato presentato come preciso in contesto client/pitch

---

### Step 6: CONSEGNA

1. **Salva il file** con naming convention: `infographic-YYYY-MM-DD-[slug].html`
   - Esempio: `infographic-2026-04-10-profilo-hd-generator.html`
   - Slug: 2-4 parole kebab-case che descrivono il contenuto
   - Salva nella cartella dell'utente o nella working directory corrente

2. **Apri in anteprima** nel browser (usa `open [filename].html` su macOS) per verifica visiva.

3. **Presenta all'utente** con:
   - Il percorso del file salvato
   - Il formato e il mood usato
   - Una breve nota (1-2 righe) su scelte compositive non ovvie

4. **Chiedi conferma:** "Vuoi modificare qualcosa? Posso cambiare il mood, il formato, i colori, o aggiungere / rimuovere elementi."

---

## Nota Tecnica: Formato Output HTML

Ogni infografica è un file HTML self-contained con le seguenti caratteristiche tecniche:

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <link href="https://fonts.googleapis.com/css2?family=Cinzel:wght@400;600;700&family=Inter:wght@300;400;500&display=swap" rel="stylesheet">
  <style>
    /* Tutto il CSS è inline qui — nessun file esterno */
    /* Dimensioni canvas definite come variabili CSS */
    :root {
      --canvas-w: 1080px;
      --canvas-h: 1350px; /* o 1080px o 1920px/1080px */
    }
  </style>
</head>
<body>
  <!-- SVG inline — nessuna immagine esterna -->
  <!-- Tutti i grafici, icone e decorazioni sono SVG generati nel markup -->
</body>
</html>
```

**Font system:**
- Titoli e headline: `Cinzel` (serif geometrico, evoca incisioni su pietra)
- Body e data label: `Inter` (sans-serif precisa, leggibilità massima)
- Varianti peso: Cinzel 400/600/700 — Inter 300/400/500

**SVG vs CSS per le forme:**
- Forme complesse (Zodiac Wheel, Bodygraph, Sacred Grid): SVG inline
- Forme semplici (cerchi, anelli, linee): CSS border-radius + box-shadow + filter
- Gradienti e glow: CSS `radial-gradient`, `conic-gradient`, `filter: blur()`

---

## File di Riferimento

| File | Contenuto | Quando Leggerlo |
|------|-----------|-----------------|
| `references/cosmos-visual-dna.md` | DNA visivo completo: palette Obsidian e Pearl, tipografia, texture, pattern geometrici, regole della luce | Sempre, prima di comporre |
| `references/component-library.md` | Codice HTML/SVG/CSS per ogni primitivo strutturale e di contenuto | Per costruire i componenti selezionati |
| `references/semantic-rules.md` | Convenzioni di accuratezza per dati astrologici, HD e GK nel visivo | Ogni volta che il contenuto è esoterico |
| `skills/astro-hd-gk-profiler/references/human-design-framework.md` | Chiavi interpretative HD: tipi, autorità, centri, canali, porte | Verifica accuratezza dati HD |
| `skills/astro-hd-gk-profiler/references/gene-keys-framework.md` | Chiavi interpretative GK: numeri, linee, sfere, sequenze | Verifica accuratezza dati GK |
| `skills/astro-hd-gk-profiler/references/prompt-astrologica.md` | Framework astrologico per business | Verifica accuratezza dati astrologici |
| `skills/astro-hd-gk-profiler/references/synthesis-framework.md` | Corrispondenze tra sistemi HD/GK/Astro | Infografiche che integrano più sistemi |
