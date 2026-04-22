---
name: vedic-astrology-report
description: "Skill per generare il report mensile di astrologia vedica (Jyotish) di GrimorIa. Usa questa skill OGNI VOLTA che l'utente chiede di creare il report astrologico mensile, il report Jyotish, le previsioni vediche del mese, o dice 'genera il report astrologico', 'crea il report del mese', 'report Jyotish [mese]'. NON usare per analisi di carte natali individuali (usa astro-hd-gk-profiler), per note Substack (usa substack-notes-writer), o per previsioni personalizzate su richiesta."
---

# Vedic Astrology Report

## Cosa produce questa skill

Un report mensile completo di astrologia vedica (Jyotish) in italiano, salvato come sottopagina Notion nella pagina "Report Astro" di GrimorIa. Il report include:

- Tema karmico del mese
- Transiti planetari principali con date precise
- Eclissi e significato vedico
- Nakshatra attive
- Graha Yuddha e aspetti particolari
- Previsioni per i 12 lagna/rashi
- Periodi propizi e sfavorevoli con tabelle
- Consigli pratici (spiritualità, lavoro, relazioni, salute)
- Mantra del mese
- Fonti delle ricerche

---

## Workflow

### Step 1 — Data corrente

```bash
date '+%B %Y'   # es. "April 2026"
date '+%Y-%m'   # es. "2026-04"
```

### Step 2 — Ricerca web (query in inglese)

Esegui le seguenti 4 query WebSearch **in parallelo**, sostituendo `[MONTH]` e `[YEAR]` con i valori reali:

1. `vedic astrology [MONTH] [YEAR] planetary transits`
2. `jyotish [MONTH] [YEAR] predictions all signs`
3. `vedic astrology eclipse nakshatra [MONTH] [YEAR]`
4. `jyotish graha transit [MONTH] [YEAR]`

Se emergono eventi particolari (Graha Yuddha, congiunzioni rare, eclissi), esegui 1-2 ricerche aggiuntive mirate.

### Step 3 — Sintesi e redazione report

Redigi il report in italiano con i seguenti blocchi (in ordine):

| Sezione | Contenuto |
|---|---|
| **Tema del mese** | Titolo evocativo + paragrafo introduttivo |
| **Transiti planetari** | Surya, Chandra, Mangal, Budha, Shukra, Guru, Shani, Rahu/Ketu — con date precise |
| **Eclissi (Grahan)** | Se presenti: tipo, segno, nakshatra, sutak kaal, chi è più influenzato |
| **Nakshatra attive** | Tabella: Nakshatra / Segno / Evento / Significato |
| **Graha Yuddha** | Se presente: pianeti coinvolti, date, effetti fisici/pratici/spirituali |
| **12 Rashi** | Breve paragrafo per ognuno dei 12 segni lunari |
| **Periodi propizi/sfavorevoli** | Tabella con date, eventi e indicazioni |
| **Consigli pratici** | Spiritualità, lavoro, relazioni, salute |
| **Mantra del mese** | 1-2 mantra con traslitterazione e contesto |
| **Sintesi karmica** | Paragrafo conclusivo ad alto impatto |
| **Fonti** | Lista linkata di tutte le fonti WebSearch usate |

**Lingua e stile:** italiano fluente, termini tecnici in sanscrito/inglese dove appropriato (es. *neecha*, *sva-nakshatra*, Graha Yuddha). Tono: autorevole, spirituale, accessibile.

### Step 4 — Salvataggio su Notion

Crea una nuova sottopagina direttamente nella pagina **"Report Astro"** su Notion:

- **Parent page ID:** `34ab63787713809eb3b2c5c4560603a6`
- **Titolo:** `Report Jyotish — [Mese] [Anno]` (es. "Report Jyotish — Aprile 2026")
- **Icona:** 🪐
- **Contenuto:** il report formattato in Notion Markdown

```
Parent:
  type: page_id
  page_id: 34ab63787713809eb3b2c5c4560603a6
```

> **Nota:** NON salvare il report come file `.md` nel repository Git. La destinazione finale è sempre Notion.

### Step 5 — Conferma

Dopo la creazione, restituisci all'utente:
- Il link diretto alla pagina Notion creata
- Un breve riepilogo dei temi principali del mese (3-5 righe)

---

## Note tecniche

- Tutti i transiti usano **astrologia siderale (Lahiri Ayanamsha)**, non tropicale
- Le previsioni per i rashi seguono il **segno lunare (Janma Rashi)**, non solare
- Il sutak kaal per le eclissi solari è di 12 ore precedenti l'evento; per le eclissi lunari varia per tradizione
- Se il mese non contiene eclissi, omettere la sezione relativa
