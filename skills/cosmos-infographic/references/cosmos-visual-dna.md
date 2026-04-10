# Cosmos Visual DNA

> Reference file for the Cosmos Infographic skill. Read this before generating any infographic.
> This document codifies the visual rules extracted from 7 Grimoria reference graphics.

---

## 1. Principi Fondamentali

### 1.1 Geometria Sacra come Struttura

Ogni layout si costruisce su geometria sacra, non su griglie rettangolari.

- Usa cerchi, vesica piscis, simmetria assiale come struttura portante
- Posiziona gli elementi lungo archi, spirali o cerchi concentrici
- Mai layout a griglia rettangolare (no CSS Grid con colonne uniformi come struttura primaria)
- Le forme di sfondo sono sempre circolari o organiche — mai rettangoli piatti

### 1.2 Centri Luminosi

Ogni infografica ha uno o più centri di luce che attraggono lo sguardo.

- `radial-gradient` con centro luminoso (color-stop 0% = colore chiaro, 100% = trasparente o scuro)
- `box-shadow` con blur 30–80px per creare alone luminoso attorno agli elementi chiave
- `filter: blur()` per sfumare elementi di sfondo e creare profondità
- La luce emana dall'interno verso l'esterno — mai bordi duri su elementi principali

### 1.3 Gradienti Eterici

I colori non sono mai piatti. Ogni forma usa almeno un gradiente.

- Usa `radial-gradient`, `conic-gradient`, `linear-gradient` — mai `background: color` solido sulle forme principali
- Minimo 2 color-stop per gradiente; preferisci 3–4 con transizioni morbide
- I gradienti trasmettono energia, profondità, vibrazione — non decorazione
- Abbina gradiente con `opacity` (0.3–0.8) per sovrapposizioni sottili

### 1.4 Il Vuoto è Sacro

Lo spazio vuoto non è mancanza — è respiro visivo intenzionale.

- Minimo 30% di spazio vuoto in ogni composizione
- Padding bordi: minimo 64px su ogni lato
- Distanza minima tra primitivi visivi: 48px
- Non riempire ogni area — la densità visiva alta è un anti-pattern

### 1.5 Doppio Registro — Obsidian e Pearl

Il sistema supporta due modalità visive, entrambe pienamente sviluppate.

**Obsidian (Dark):**
- Background: `#0E0E10` (`--obsidian`)
- I colori luminosi (Gold, Cosmos Violet, Cosmos Blue) brillano sul nero
- `box-shadow` con colori accesi per effetto glow
- Testo principale: Pearl (`#EDEAE4`)

**Pearl (Light):**
- Background: `#EDEAE4` (`--pearl`)
- Ombre morbide al posto dei glow (`box-shadow` con colori scuri a bassa opacità)
- `filter: drop-shadow()` sottile per profondità
- Testo principale: Obsidian (`#0E0E10`)

### 1.6 Organico Dentro il Geometrico

Le forme sono precise nella struttura, ma sempre ammorbidite nell'esecuzione.

- I cerchi e le forme geometriche hanno contorni precisi via CSS/SVG
- I fill sono morbidi: gradiente + opacità parziale, mai colore solido pieno
- Usa sempre `box-shadow` o `filter: blur()` per ammorbidire la percezione dei bordi
- Le forme si sovrappongono con `mix-blend-mode: screen` o `overlay` per fusione organica

---

## 2. Palette Colori

### 2.1 Colori Base (Grimoria Design Tokens)

| Nome | Hex | CSS Variable | Uso |
|------|-----|-------------|-----|
| Obsidian | `#0E0E10` | `--obsidian` | Dark bg, testo su Pearl |
| Pearl | `#EDEAE4` | `--pearl` | Light bg, testo su Obsidian |
| Pearl Deep | `#E0DDD7` | `--pearl-deep` | Superfici secondarie light |
| Gold | `#B8924A` | `--gold` | Accenti esoterici, CTA, astrologia |
| Emerald | `#009E74` | `--emerald` | Accenti Tech/AI, label di sistema |
| Charcoal | `#3A3530` | `--charcoal` | Testo scuro secondario |
| Stone | `#7A7670` | `--stone` | Body text, elementi muted |

### 2.2 Estensioni Cosmos

| Nome | Hex | CSS Variable | Uso |
|------|-----|-------------|-----|
| Cosmos Blue | `#4A6CF7` | `--cosmos-blue` | Gradienti freddi (start) |
| Cosmos Blue End | `#7B5EA7` | `--cosmos-blue-end` | Gradienti freddi (end) |
| Cosmos Violet | `#9B6DFF` | `--cosmos-violet` | Aura, campi energetici (start) |
| Cosmos Violet End | `#FF6FB5` | `--cosmos-violet-end` | Aura, nuclei radianti (end) |
| Cosmos Amber | `#C4A24D` | `--cosmos-amber` | Singolarità calde (start) |
| Cosmos Amber End | `#1A1A1A` | `--cosmos-amber-end` | Buchi neri dorati (end) |

### 2.3 Regole Colore

- **Mai** `#000000` puro — usa Obsidian (`#0E0E10`)
- **Mai** saturazione 100% — tutti i colori hanno calore o grigio incorporato
- **Max 3 colori dominanti** per infografica (+ neutri Obsidian/Pearl)
- **Gold** = esoterismo, astrologia, spiritualità
- **Emerald** = AI, tecnologia, sistemi
- **Cosmos Violet** = Gene Keys, aura, coscienza
- **Cosmos Blue** = Human Design, struttura, mente

---

## 3. Tipografia

### 3.1 Google Fonts Import

```html
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600&family=Playfair+Display:ital,wght@0,400;0,500;0,600;0,700;1,400&display=swap" rel="stylesheet">
```

```css
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600&family=Playfair+Display:ital,wght@0,400;0,500;0,600;0,700;1,400&display=swap');
```

### 3.2 Type Scale

| Role | Font | Weight | Size | Line-height | Letter-spacing | Uso |
|------|------|--------|------|-------------|----------------|-----|
| Display | Playfair Display | 700 | `clamp(40px, 5vw, 56px)` | 1.1 | -0.02em | Titolo principale, hero |
| H1 | Playfair Display | 600 | `clamp(28px, 3.5vw, 40px)` | 1.2 | -0.01em | Sezioni principali |
| H2 | Playfair Display | 500 | `clamp(22px, 2.5vw, 28px)` | 1.3 | normal | Sottosezioni |
| Pull Quote | Playfair Display | 400 italic | `clamp(20px, 2.5vw, 28px)` | 1.4 | normal | Citazioni, insight chiave |
| Body | Inter | 300 | 17px | 1.85 | 0.01em | Testo narrativo |
| Label | Inter | 600 | 12px | 1.5 | 0.3em uppercase | Etichette, categorie |
| Data Label | Inter | 500 | 11px | 1.4 | 0.15em uppercase | Valori numerici, dati |
| Caption | Inter | 300 | 13px | 1.5 | normal | Note, descrizioni brevi |

### 3.3 Regole Tipografiche

- **Playfair Display SOLO** per titoli (Display, H1, H2) e citazioni (Pull Quote)
- **Inter per tutto il resto** — label, body, caption, dati
- Body sempre weight **300** (light) — mai 400 per paragrafi lunghi
- Line-height minimo **1.85** per body — accessibilità ADHD
- Testo minimo **13px** — mai sotto questa soglia
- Uppercase SOLO per Label e Data Label, mai per body o titoli

---

## 4. Glifi Esoterici

I glifi Unicode sono parte del vocabolario visivo Cosmos. Usali con intenzione.

### 4.1 Pianeti Astrologici

| Glifo | Corpo | Colore |
|-------|-------|--------|
| ☉ | Sole | Gold `#B8924A` |
| ☽ | Luna | Gold `#B8924A` |
| ☿ | Mercurio | Gold `#B8924A` |
| ♀ | Venere | Gold `#B8924A` |
| ♂ | Marte | Gold `#B8924A` |
| ♃ | Giove | Gold `#B8924A` |
| ♄ | Saturno | Gold `#B8924A` |
| ♅ | Urano | Gold `#B8924A` |
| ♆ | Nettuno | Gold `#B8924A` |
| ♇ | Plutone | Gold `#B8924A` |

### 4.2 Segni Zodiacali

| Glifi | Colore |
|-------|--------|
| ♈ ♉ ♊ ♋ ♌ ♍ ♎ ♏ ♐ ♑ ♒ ♓ | Gold `#B8924A` |

### 4.3 Human Design

| Glifo | Elemento | Colore |
|-------|----------|--------|
| ⬡ | Centri HD | Emerald `#009E74` |
| △ | Canali | Emerald `#009E74` |

### 4.4 Gene Keys

| Glifo | Elemento | Colore |
|-------|----------|--------|
| ✦ | Chiavi | Cosmos Violet `#9B6DFF` |
| ◯ | Sfere | Cosmos Violet `#9B6DFF` |

### 4.5 Styling Glifi

```css
.glyph {
  font-family: 'Inter', sans-serif;
  font-weight: 400;
  font-size: 120%; /* 120–150% del testo circostante */
  line-height: 1;
}
```

---

## 5. Anti-Pattern — 10 Regole del MAI

Queste regole sono assolute. Nessuna eccezione.

| # | Anti-Pattern | Perché |
|---|-------------|--------|
| 1 | **Mai clipart o icone flat** | Rompono l'estetica organico-cosmica |
| 2 | **Mai bordi `1px solid` sulle forme principali** | Le forme devono avere luce, non contorni duri |
| 3 | **Mai più di 3 colori dominanti** | La palette deve respirare — la complessità viene dalla geometria |
| 4 | **Mai testo sotto 13px** | Accessibilità e leggibilità non negoziabili |
| 5 | **Mai layout a griglia rettangolare** | La struttura è circolare, assiale, organica |
| 6 | **Mai colori piatti sulle forme principali** | Ogni forma vuole un gradiente — i colori solidi sono morti |
| 7 | **Mai `#000000` puro** | Usa Obsidian `#0E0E10` — il nero assoluto è artificiale |
| 8 | **Mai riempire tutto lo spazio** | Minimo 30% vuoto — il respiro è intenzionale |
| 9 | **Mai font decorativi oltre Playfair Display** | Il sistema è bi-tipografico: Playfair + Inter, nient'altro |
| 10 | **Mai immagini esterne** | Tutto il visual è CSS/SVG inline — zero dipendenze esterne |

---

## Note d'Uso

Questo file è la fonte unica di verità per le decisioni visive del Cosmos Infographic skill.

Prima di generare qualsiasi infografica:
1. Verifica il registro (Obsidian o Pearl)
2. Scegli max 3 colori dominanti dalla palette
3. Pianifica la struttura geometrica (cerchi, archi, simmetria)
4. Applica i principi di vuoto e luce
5. Controlla gli anti-pattern prima di finalizzare
