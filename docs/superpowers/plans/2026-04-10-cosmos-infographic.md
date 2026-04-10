# Cosmos Infographic Skill — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create a Claude Code skill that generates personalized infographics in the Grimoria/Cosmos visual style, with semantic knowledge of Astrology, Human Design, and Gene Keys.

**Architecture:** Component Library + Composition. The skill defines visual primitives (Orb, Totem, Vesica, etc.) as CSS/SVG code blocks, stores them in reference files, and instructs Claude to compose them into self-contained HTML infographics based on free-form prompts. Semantic rules reference the existing `astro-hd-gk-profiler` knowledge base without duplication.

**Tech Stack:** Claude Code Skill (Markdown), HTML5, CSS3 (gradients, filters, blur), inline SVG, Google Fonts (Playfair Display, Inter)

**Spec:** `docs/superpowers/specs/2026-04-10-cosmos-infographic-design.md`

---

## File Structure

```
skills/cosmos-infographic/
├── SKILL.md                          # Skill definition, trigger, full workflow
├── references/
│   ├── cosmos-visual-dna.md          # Visual style system: palette, typography, principles, anti-patterns
│   ├── component-library.md          # All primitives with complete CSS/SVG code
│   └── semantic-rules.md             # Esoteric positioning rules for Zodiac, HD, GK
└── templates/
    ├── base-obsidian.html            # Dark mode base template with shared CSS
    └── base-pearl.html               # Light mode base template with shared CSS
```

**Dependencies (read-only, not modified):**
- `skills/astro-hd-gk-profiler/references/human-design-framework.md`
- `skills/astro-hd-gk-profiler/references/gene-keys-framework.md`
- `skills/astro-hd-gk-profiler/references/prompt-astrologica.md`
- `skills/astro-hd-gk-profiler/references/synthesis-framework.md`
- `Grimoria Analisi e Branding/Brand_Identity_Master/design-tokens.json`

---

### Task 1: Create skill directory and SKILL.md

**Files:**
- Create: `skills/cosmos-infographic/SKILL.md`

- [ ] **Step 1: Create directory structure**

```bash
mkdir -p skills/cosmos-infographic/references skills/cosmos-infographic/templates
```

- [ ] **Step 2: Write SKILL.md with frontmatter, philosophy, and full workflow**

Create `skills/cosmos-infographic/SKILL.md` with this content:

```markdown
---
name: cosmos-infographic
description: "Skill per generare infografiche personalizzate nello stile visivo Cosmos di Grimoria. Usa questa skill OGNI VOLTA che l'utente chiede di creare un'infografica, un visual, una mappa visiva, un diagramma, o qualsiasi grafica nello stile Grimoria/Cosmos. Attiva anche quando l'utente dice 'fammi un'infografica', 'crea un visual', 'grafica per Instagram', 'visual per Substack', 'infografica per il profilo di [nome]', 'visual per il pitch', 'mappa visiva', 'diagramma HD/GK/astro', o qualsiasi richiesta che implichi la creazione di contenuto grafico. Supporta contenuti esoterici intelligenti (posizionamento semanticamente corretto di segni zodiacali, gate HD, Gene Keys). NON usare per analisi testuali di carte natali (usa astro-hd-gk-profiler), per video (usa remotion-video-pipeline), o per editing di immagini esistenti."
---

# Cosmos Infographic

## Filosofia

**La geometria sacra e il linguaggio visivo dell'anima.**

Ogni infografica Cosmos non e una "grafica" — e un campo visivo. Chi la guarda sente qualcosa prima di leggere qualcosa. Lo stile nasce dall'intersezione di arte generativa, geometria sacra e luxury esoteric design.

Principi inviolabili:
- **Il vuoto e sacro** — minimo 30% di spazio vuoto. Il respiro e parte del messaggio.
- **Centri luminosi** — ogni composizione ha almeno un nucleo radiante che attrae lo sguardo.
- **Organico dentro il geometrico** — forme precise con riempimenti morbidi, sfumati, luminescenti.
- **Mai piatto** — tutto respira con gradienti eterici. Zero colori solidi su forme principali.

---

## Workflow

### Step 0: Leggi i Riferimenti

Prima di generare qualsiasi infografica, leggi SEMPRE:
1. `references/cosmos-visual-dna.md` — regole visive, palette, anti-pattern
2. `references/component-library.md` — primitive CSS/SVG disponibili
3. `references/semantic-rules.md` — regole di posizionamento esoterico (solo se il contenuto include dati astro/HD/GK)

### Step 1: INTERPRET — Analisi del Prompt

Dall'input dell'utente, identifica:

| Parametro | Come rilevarlo | Default |
|-----------|---------------|---------|
| **Soggetto** | Il tema principale dell'infografica | — (obbligatorio) |
| **Formato** | "per Instagram" = 1080x1350, "quadrato" = 1080x1080, "per slide" = 1920x1080 | 1080x1350 verticale |
| **Mood** | "dark" / "obsidian" = scuro, "pearl" / "light" = chiaro | Claude sceglie in base al contesto |
| **Destinazione** | "per Substack" / "per cliente" / "per pitch" | Informa rigore semantico |

Regole per la scelta automatica del mood:
- **Obsidian** (dark): contenuti social, post educational, pitch, contenuti "potenti"
- **Pearl** (light): deliverable per clienti, profili personali, contenuti intimi

### Step 2: SELECT — Scelta Primitive

Consulta `references/component-library.md` e seleziona:
- **Max 3 primitive strutturali** tra: Orb, Totem, Aura Ring, Vesica, Sacred Grid, Bioform, Tech Blueprint
- **Primitive di contenuto** necessarie: Zodiac Wheel, HD Bodygraph Mini, GK Spectrum Bar, Data Label, Pull Quote

Guida alla selezione:

| Contenuto | Primitive strutturali consigliate |
|-----------|----------------------------------|
| Lista/gerarchia (es. 4 tipi HD) | Totem (verticale) o Sacred Grid |
| Dualita/confronto (es. Sole vs Luna) | Vesica |
| Mappa circolare (es. wheel zodiacale) | Aura Ring |
| Singolo concetto/energia | Orb singolo con Bioform di sfondo |
| Dati tecnici/blueprint | Tech Blueprint |
| Profilo cliente completo | Totem + primitive di contenuto specifiche |

### Step 3: ENRICH — Arricchimento Semantico

Se il contenuto include dati esoterici:
1. Consulta `references/semantic-rules.md` per le regole di posizionamento
2. Per dati astrologici: verifica posizioni contro `skills/astro-hd-gk-profiler/references/prompt-astrologica.md`
3. Per dati HD: verifica gate e centri contro `skills/astro-hd-gk-profiler/references/human-design-framework.md`
4. Per dati GK: verifica trittici contro `skills/astro-hd-gk-profiler/references/gene-keys-framework.md`

**Regola d'oro:** se un dato non torna o e ambiguo, CHIEDI conferma all'utente. Mai inventare posizioni.

### Step 4: COMPOSE — Assemblaggio HTML

1. Parti dal template base appropriato (`templates/base-obsidian.html` o `templates/base-pearl.html`)
2. Inserisci le primitive selezionate seguendo le regole di composizione:
   - **Asse centrale** — tutto allineato su asse verticale o centro radiale
   - **Gerarchia per luminosita** — elemento principale = piu grande e luminoso
   - **Breathing space** — minimo 48px tra primitive, 64px dai bordi
   - **Layer order** — sfondo → primitive strutturali → primitive di contenuto → testo
3. Applica il CSS delle primitive da `references/component-library.md`
4. Sostituisci contenuti placeholder con i dati reali

### Step 5: VERIFY — Controllo Qualita

Prima di consegnare, verifica:

**Anti-pattern check:**
- [ ] Nessun clipart o icona flat
- [ ] Nessun bordo 1px solid netto su forme principali
- [ ] Max 3 colori dominanti
- [ ] Nessun testo sotto 13px
- [ ] Nessun layout a griglia rettangolare rigida

**Leggibilita:**
- [ ] Contrasto testo/sfondo sufficiente
- [ ] Font caricate correttamente (Google Fonts link presente)
- [ ] Spaziatura adeguata tra elementi

**Accuratezza semantica** (se applicabile):
- [ ] Segni zodiacali nella posizione corretta
- [ ] Centri HD nella topologia corretta
- [ ] Trittici GK con nomi specifici (mai generici)

### Step 6: DELIVER — Output

1. Salva il file come `infographic-YYYY-MM-DD-<slug>.html` nella directory del progetto
2. Apri nel browser preview per verifica visiva
3. Mostra screenshot all'utente
4. Chiedi: "Ti piace? Vuoi modifiche?"

---

## Formato Output

Il file HTML e COMPLETAMENTE autocontenuto:
- CSS inline nel `<style>` tag
- SVG inline nel body
- Google Fonts caricati via `<link>` tag
- Nessuna dipendenza esterna (niente JS frameworks, niente immagini esterne)
- Dimensioni impostate via CSS su `body` e `.canvas`
```

- [ ] **Step 3: Commit**

```bash
git add skills/cosmos-infographic/SKILL.md
git commit -m "feat: add cosmos-infographic skill definition"
```

---

### Task 2: Create Cosmos Visual DNA reference

**Files:**
- Create: `skills/cosmos-infographic/references/cosmos-visual-dna.md`

- [ ] **Step 1: Write the visual DNA reference with palette, typography, principles**

Create `skills/cosmos-infographic/references/cosmos-visual-dna.md` with this content:

```markdown
# Cosmos Visual DNA

> Riferimento visivo per tutte le infografiche Cosmos. Questo file codifica le regole estratte dalle 7 reference grafiche Cosmos (csms-001 → csms-007).

---

## Principi Fondamentali

### 1. Geometria Sacra come Struttura
Cerchi, vesica piscis, simmetria assiale. Mai griglie rettangolari piatte — tutto e curvo, orbitale, concentrico.

### 2. Centri Luminosi
Ogni composizione ha almeno un punto focale radiante. Implementazione:
- `radial-gradient` con centro luminoso e bordi sfumati
- `box-shadow` con blur alto (30-80px) per glow
- `filter: blur()` su layer di sfondo per effetto eterico

### 3. Gradienti Eterici
Mai colori piatti. Tutto respira con sfumature:
- `radial-gradient` per sfere e nuclei
- `conic-gradient` per rotazioni e wheel
- `linear-gradient` per barre e transizioni
- Sempre almeno 2 color-stop per gradiente

### 4. Il Vuoto e Sacro
Minimo 30% dello spazio totale e vuoto (no contenuto, no decorazione).
- Padding minimo dal bordo: `64px`
- Spaziatura minima tra primitive: `48px`
- Se in dubbio, togli un elemento — non aggiungerne uno

### 5. Doppio Registro

**Obsidian (Dark Mode):**
- Sfondo: `#0E0E10`
- Glow: colori chiari con opacita (bianco, gold, viola)
- Testo primario: `#EDEAE4` (Pearl)
- Effetto: luminosita su sfondo profondo

**Pearl (Light Mode):**
- Sfondo: `#EDEAE4`
- Ombre: morbide, scure, con blur alto
- Testo primario: `#0E0E10` (Obsidian)
- Effetto: leggerezza con profondita sottile

### 6. Organico Dentro il Geometrico
Le forme geometriche sono precise nei contorni, ma i riempimenti sono morbidi, sfumati, luminescenti. Mai bordi netti al 100% su forme principali — usare sempre `box-shadow` o `filter: blur()` per ammorbidire.

---

## Palette Colori

### Colori Base (da design-tokens.json Grimoria)

| Nome | HEX | CSS Variable | Uso |
|------|-----|-------------|-----|
| Obsidian | `#0E0E10` | `--obsidian` | Sfondo dark, testo su pearl |
| Pearl | `#EDEAE4` | `--pearl` | Sfondo light, testo su obsidian |
| Pearl Deep | `#E0DDD7` | `--pearl-deep` | Superfici secondarie light |
| Gold | `#B8924A` | `--gold` | Accenti esoterici, CTA, astrologia |
| Emerald | `#009E74` | `--emerald` | Accenti tech/AI, label sistema |
| Charcoal | `#3A3530` | `--charcoal` | Testo secondario scuro |
| Stone | `#7A7670` | `--stone` | Testo body, elementi muted |

### Colori Cosmos (estensioni per infografiche)

| Nome | Valore | CSS Variable | Uso |
|------|--------|-------------|-----|
| Cosmos Blue | `#4A6CF7` | `--cosmos-blue` | Gradienti freddi (start) |
| Cosmos Blue End | `#7B5EA7` | `--cosmos-blue-end` | Gradienti freddi (end) |
| Cosmos Violet | `#9B6DFF` | `--cosmos-violet` | Aura, campi energetici (start) |
| Cosmos Violet End | `#FF6FB5` | `--cosmos-violet-end` | Aura, nuclei radianti (end) |
| Cosmos Amber | `#C4A24D` | `--cosmos-amber` | Singolarita calde (start) |
| Cosmos Amber End | `#1A1A1A` | `--cosmos-amber-end` | Buchi neri dorati (end) |

### Regole Colore
- Mai `#000000` puro — sempre Obsidian `#0E0E10`
- Mai colori saturi al 100% — sempre leggermente desaturati o con opacita
- Max 3 colori dominanti per infografica (1 sfondo + 2 accenti)
- Gold = esoterismo, astrologia, luxury
- Emerald = AI, sistema, tech
- Cosmos Blue/Violet = energia, aura, campo

---

## Tipografia

### Font Stack

```css
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600&family=Playfair+Display:ital,wght@0,400;0,500;0,600;0,700;1,400&display=swap');
```

### Scala Tipografica

| Ruolo | Font | Weight | Size | Line-height | Letter-spacing | Uso |
|-------|------|--------|------|-------------|----------------|-----|
| **Display** | Playfair Display | 700 | `clamp(40px, 5vw, 56px)` | 1.1 | -0.02em | Titolo hero |
| **H1** | Playfair Display | 600 | `clamp(28px, 3.5vw, 40px)` | 1.2 | -0.01em | Titolo sezione |
| **H2** | Playfair Display | 500 | `clamp(22px, 2.5vw, 28px)` | 1.3 | normal | Sottotitolo |
| **Pull Quote** | Playfair Display | 400 italic | `clamp(20px, 2.5vw, 28px)` | 1.4 | normal | Citazioni, manifesto |
| **Body** | Inter | 300 | 17px | 1.85 | 0.01em | Testo corpo |
| **Label** | Inter | 600 | 12px | 1.5 | 0.3em (uppercase) | Etichette tech |
| **Data Label** | Inter | 500 | 11px | 1.4 | 0.15em (uppercase) | Dati, categorie |
| **Caption** | Inter | 300 | 13px | 1.5 | normal | Note, didascalie |

### Regole Tipografia
- Playfair Display: SOLO titoli e citazioni — MAI per body
- Inter: tutto il resto
- Body weight SEMPRE 300 (light) per eleganza
- Line-height minimo 1.85 per body (ADHD-friendly)
- Testo minimo: 13px (Caption). Mai sotto.

---

## Glifi Esoterici

Usa Unicode symbols, non icone:

| Contesto | Simboli | Colore |
|----------|---------|--------|
| Astrologia | ☉ ☽ ☿ ♀ ♂ ♃ ♄ ♅ ♆ ♇ | Gold |
| Segni zodiacali | ♈ ♉ ♊ ♋ ♌ ♍ ♎ ♏ ♐ ♑ ♒ ♓ | Gold |
| Human Design | ⬡ (centri), △ (canali) | Emerald |
| Gene Keys | ✦ (chiavi), ◯ (sfere) | Cosmos Violet |
| Decorativi | ✦ ◉ ⊕ ☆ | Varia |

Font per glifi: Inter weight 400, size 120-150% del testo circostante.

---

## Anti-Pattern — Cosa NON Fare Mai

1. **Mai clipart o icone flat-style** — solo geometria sacra, glifi Unicode, e forme CSS/SVG
2. **Mai bordi 1px solid netti** su forme principali — solo glow (`box-shadow`) e ombre
3. **Mai piu di 3 colori dominanti** per infografica
4. **Mai testo sotto i 13px**
5. **Mai layout a griglia rettangolare rigida** — sempre flusso circolare/organico/assiale
6. **Mai colori piatti** su forme principali — sempre gradienti
7. **Mai `#000000`** — usare Obsidian `#0E0E10`
8. **Mai riempire tutto lo spazio** — minimo 30% vuoto
9. **Mai font decorativi** oltre a Playfair Display — solo Inter per tutto il non-titolo
10. **Mai immagini esterne** — tutto deve essere CSS/SVG inline
```

- [ ] **Step 2: Commit**

```bash
git add skills/cosmos-infographic/references/cosmos-visual-dna.md
git commit -m "feat: add Cosmos Visual DNA reference — palette, typography, principles"
```

---

### Task 3: Create Component Library reference

**Files:**
- Create: `skills/cosmos-infographic/references/component-library.md`

- [ ] **Step 1: Write the component library with all 12 primitives and their CSS/SVG code**

Create `skills/cosmos-infographic/references/component-library.md` with this content:

```markdown
# Component Library — Primitive Visive Cosmos

> Ogni primitiva e un blocco CSS/SVG autocontenuto. Claude seleziona e compone queste primitive per costruire ogni infografica.

---

## Primitive Strutturali

### 1. Orb

Sfera radiante con gradiente eterico e nucleo luminoso. Scalabile. Puo contenere testo o simbolo al centro.

**Varianti:** `orb-blue`, `orb-violet`, `orb-amber`, `orb-gold`, `orb-emerald`

```html
<div class="orb orb-violet" style="--orb-size: 200px;">
  <div class="orb-core">
    <span class="orb-label">♏</span>
  </div>
</div>
```

```css
.orb {
  width: var(--orb-size, 200px);
  height: var(--orb-size, 200px);
  border-radius: 50%;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
}

.orb-core {
  width: 60%;
  height: 60%;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2;
}

.orb-label {
  font-family: 'Inter', sans-serif;
  font-weight: 400;
  font-size: calc(var(--orb-size, 200px) * 0.15);
  color: var(--pearl);
}

/* --- Obsidian mode --- */
.orb-violet {
  background: radial-gradient(circle at 50% 50%,
    rgba(155, 109, 255, 0.6) 0%,
    rgba(255, 111, 181, 0.3) 40%,
    rgba(155, 109, 255, 0.05) 70%,
    transparent 100%);
  box-shadow:
    0 0 60px rgba(155, 109, 255, 0.3),
    0 0 120px rgba(155, 109, 255, 0.1);
}

.orb-blue {
  background: radial-gradient(circle at 50% 50%,
    rgba(74, 108, 247, 0.6) 0%,
    rgba(123, 94, 167, 0.3) 40%,
    rgba(74, 108, 247, 0.05) 70%,
    transparent 100%);
  box-shadow:
    0 0 60px rgba(74, 108, 247, 0.3),
    0 0 120px rgba(74, 108, 247, 0.1);
}

.orb-amber {
  background: radial-gradient(circle at 50% 50%,
    rgba(196, 162, 77, 0.7) 0%,
    rgba(196, 162, 77, 0.2) 40%,
    rgba(26, 26, 26, 0.1) 70%,
    transparent 100%);
  box-shadow:
    0 0 60px rgba(196, 162, 77, 0.3),
    0 0 120px rgba(196, 162, 77, 0.1);
}

.orb-gold {
  background: radial-gradient(circle at 50% 50%,
    rgba(184, 146, 74, 0.7) 0%,
    rgba(184, 146, 74, 0.3) 40%,
    rgba(184, 146, 74, 0.05) 70%,
    transparent 100%);
  box-shadow:
    0 0 60px rgba(184, 146, 74, 0.3),
    0 0 120px rgba(184, 146, 74, 0.1);
}

.orb-emerald {
  background: radial-gradient(circle at 50% 50%,
    rgba(0, 158, 116, 0.6) 0%,
    rgba(0, 158, 116, 0.2) 40%,
    rgba(0, 158, 116, 0.05) 70%,
    transparent 100%);
  box-shadow:
    0 0 60px rgba(0, 158, 116, 0.3),
    0 0 120px rgba(0, 158, 116, 0.1);
}

/* --- Pearl mode overrides --- */
.pearl-mode .orb-violet {
  background: radial-gradient(circle at 50% 50%,
    rgba(155, 109, 255, 0.4) 0%,
    rgba(255, 111, 181, 0.2) 40%,
    rgba(155, 109, 255, 0.03) 70%,
    transparent 100%);
  box-shadow:
    0 0 40px rgba(155, 109, 255, 0.15),
    0 0 80px rgba(155, 109, 255, 0.05);
}

.pearl-mode .orb-label {
  color: var(--obsidian);
}
```

---

### 2. Totem

Composizione verticale di 2-4 orb allineati sull'asse centrale, con connessioni luminose tra loro.

```html
<div class="totem">
  <div class="orb orb-blue" style="--orb-size: 160px;">
    <div class="orb-core"><span class="orb-label">☉</span></div>
  </div>
  <div class="totem-connector"></div>
  <div class="orb orb-violet" style="--orb-size: 200px;">
    <div class="orb-core"><span class="orb-label">☽</span></div>
  </div>
  <div class="totem-connector"></div>
  <div class="orb orb-amber" style="--orb-size: 140px;">
    <div class="orb-core"><span class="orb-label">♄</span></div>
  </div>
</div>
```

```css
.totem {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0;
}

.totem-connector {
  width: 2px;
  height: 40px;
  background: linear-gradient(to bottom,
    rgba(155, 109, 255, 0.4),
    rgba(155, 109, 255, 0.05));
  margin: -20px 0;
  z-index: 1;
}
```

---

### 3. Aura Ring

Cerchio con alone sfumato concentrico — per mappe circolari con label attorno al perimetro.

```html
<div class="aura-ring" style="--ring-size: 400px;">
  <div class="aura-ring-glow"></div>
  <div class="aura-ring-core"></div>
  <!-- Labels positioned with transform: rotate + translateY -->
  <div class="aura-label" style="--angle: 0deg;">Experiencing new things</div>
  <div class="aura-label" style="--angle: 45deg;">Taking it all in</div>
  <div class="aura-label" style="--angle: 90deg;">Making a difference</div>
  <!-- ... more labels ... -->
</div>
```

```css
.aura-ring {
  width: var(--ring-size, 400px);
  height: var(--ring-size, 400px);
  border-radius: 50%;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
}

.aura-ring-glow {
  position: absolute;
  inset: 0;
  border-radius: 50%;
  background: radial-gradient(circle at 50% 50%,
    rgba(155, 109, 255, 0.5) 0%,
    rgba(74, 108, 247, 0.3) 30%,
    rgba(255, 111, 181, 0.2) 50%,
    rgba(155, 109, 255, 0.1) 70%,
    transparent 100%);
  filter: blur(8px);
}

.aura-ring-core {
  width: 20%;
  height: 20%;
  border-radius: 50%;
  background: radial-gradient(circle, white 0%, rgba(155, 109, 255, 0.6) 60%, transparent 100%);
  z-index: 2;
}

.aura-label {
  position: absolute;
  top: 50%;
  left: 50%;
  transform:
    rotate(var(--angle))
    translateY(calc(var(--ring-size, 400px) * -0.42))
    rotate(calc(-1 * var(--angle)));
  font-family: 'Inter', sans-serif;
  font-weight: 300;
  font-size: 13px;
  color: var(--pearl);
  text-align: center;
  white-space: nowrap;
}
```

---

### 4. Vesica

Due cerchi sovrapposti con zona di intersezione luminosa (vesica piscis).

```html
<div class="vesica" style="--vesica-size: 300px;">
  <div class="vesica-circle vesica-top">
    <span class="orb-label">☉ Cancro</span>
  </div>
  <div class="vesica-circle vesica-bottom">
    <span class="orb-label">☽ Capricorno</span>
  </div>
  <div class="vesica-intersection">
    <span class="vesica-star">✦</span>
  </div>
</div>
```

```css
.vesica {
  position: relative;
  width: var(--vesica-size, 300px);
  height: calc(var(--vesica-size, 300px) * 1.5);
  display: flex;
  flex-direction: column;
  align-items: center;
}

.vesica-circle {
  width: var(--vesica-size, 300px);
  height: var(--vesica-size, 300px);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  position: absolute;
}

.vesica-top {
  top: 0;
  background: radial-gradient(circle,
    rgba(74, 108, 247, 0.4) 0%,
    rgba(123, 94, 167, 0.2) 50%,
    transparent 80%);
}

.vesica-bottom {
  bottom: 0;
  background: radial-gradient(circle,
    rgba(196, 162, 77, 0.4) 0%,
    rgba(196, 162, 77, 0.15) 50%,
    transparent 80%);
}

.vesica-intersection {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 60%;
  height: 30%;
  background: radial-gradient(ellipse,
    rgba(255, 255, 255, 0.3) 0%,
    rgba(255, 111, 181, 0.2) 50%,
    transparent 100%);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 3;
}

.vesica-star {
  font-size: 24px;
  color: var(--gold);
}
```

---

### 5. Sacred Grid

Griglia di punti/cerchi con pattern geometrico — per sistemi, confronti, tassonomie.

```html
<div class="sacred-grid" style="--grid-cols: 4; --grid-rows: 3;">
  <div class="grid-node grid-node-active">
    <div class="grid-dot"></div>
    <span class="grid-label">Manifestor</span>
  </div>
  <div class="grid-node">
    <div class="grid-dot"></div>
    <span class="grid-label">Generator</span>
  </div>
  <!-- ... more nodes ... -->
</div>
```

```css
.sacred-grid {
  display: grid;
  grid-template-columns: repeat(var(--grid-cols, 4), 1fr);
  grid-template-rows: repeat(var(--grid-rows, 3), 1fr);
  gap: 48px;
  justify-items: center;
  align-items: center;
}

.grid-node {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
}

.grid-dot {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: rgba(237, 234, 228, 0.2);
  border: 1px solid rgba(237, 234, 228, 0.1);
}

.grid-node-active .grid-dot {
  width: 32px;
  height: 32px;
  background: radial-gradient(circle, var(--gold) 0%, rgba(184, 146, 74, 0.3) 60%, transparent 100%);
  box-shadow: 0 0 20px rgba(184, 146, 74, 0.4);
  border: none;
}

.grid-label {
  font-family: 'Inter', sans-serif;
  font-weight: 500;
  font-size: 11px;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  color: var(--stone);
}
```

---

### 6. Bioform

Forma simmetrica organica generata da SVG con curve bezier sovrapposte e opacita variabile.

```html
<svg class="bioform" viewBox="0 0 400 400" width="400" height="400">
  <defs>
    <filter id="bioform-blur">
      <feGaussianBlur in="SourceGraphic" stdDeviation="4"/>
    </filter>
  </defs>
  <!-- Layer 1: outer petals -->
  <g opacity="0.3" filter="url(#bioform-blur)">
    <path d="M200,50 C280,100 320,180 200,200 C80,180 120,100 200,50Z"
          fill="rgba(155,109,255,0.4)"/>
    <path d="M200,350 C280,300 320,220 200,200 C80,220 120,300 200,350Z"
          fill="rgba(255,111,181,0.4)"/>
  </g>
  <!-- Layer 2: inner form -->
  <g opacity="0.5">
    <path d="M200,100 C260,140 280,180 200,200 C120,180 140,140 200,100Z"
          fill="rgba(74,108,247,0.5)"/>
    <path d="M200,300 C260,260 280,220 200,200 C120,220 140,260 200,300Z"
          fill="rgba(196,162,77,0.5)"/>
  </g>
  <!-- Center node -->
  <circle cx="200" cy="200" r="12" fill="rgba(255,255,255,0.8)"/>
</svg>
```

```css
.bioform {
  position: absolute;
  opacity: 0.6;
  z-index: 0;
}
```

Note: Le curve bezier del Bioform devono essere adattate al contenuto specifico. Questo e un template di partenza — Claude genera forme simmetriche uniche per ogni infografica mantenendo la simmetria verticale.

---

### 7. Tech Blueprint

Linee sottili, cerchi concentrici con annotazioni — stile disegno tecnico.

```html
<svg class="tech-blueprint" viewBox="0 0 600 800" width="600" height="800">
  <!-- Concentric circles -->
  <circle cx="300" cy="400" r="250" fill="none" stroke="var(--charcoal)" stroke-width="0.5" opacity="0.3"/>
  <circle cx="300" cy="400" r="180" fill="none" stroke="var(--charcoal)" stroke-width="0.5" opacity="0.3"/>
  <circle cx="300" cy="400" r="100" fill="none" stroke="var(--charcoal)" stroke-width="0.5" opacity="0.3"/>
  <!-- Axis lines -->
  <line x1="50" y1="400" x2="550" y2="400" stroke="var(--charcoal)" stroke-width="0.5" opacity="0.2" stroke-dasharray="4,4"/>
  <line x1="300" y1="100" x2="300" y2="700" stroke="var(--charcoal)" stroke-width="0.5" opacity="0.2" stroke-dasharray="4,4"/>
  <!-- Annotation points -->
  <circle cx="300" cy="150" r="4" fill="var(--gold)"/>
  <text x="315" y="153" class="blueprint-annotation">Testa</text>
</svg>
```

```css
.tech-blueprint {
  width: 100%;
  height: auto;
}

.blueprint-annotation {
  font-family: 'Inter', sans-serif;
  font-weight: 500;
  font-size: 10px;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  fill: var(--stone);
}
```

---

## Primitive di Contenuto

### Zodiac Wheel

```html
<div class="zodiac-wheel" style="--wheel-size: 360px;">
  <svg viewBox="0 0 360 360" class="zodiac-svg">
    <!-- Outer ring -->
    <circle cx="180" cy="180" r="170" fill="none" stroke="rgba(184,146,74,0.2)" stroke-width="1"/>
    <circle cx="180" cy="180" r="140" fill="none" stroke="rgba(184,146,74,0.1)" stroke-width="0.5"/>
    <!-- 12 sector dividers (30deg each) -->
    <!-- Aries starts at 270deg (ore 9), counterclockwise -->
    <line x1="180" y1="180" x2="10" y2="180" stroke="rgba(184,146,74,0.15)" stroke-width="0.5"/>
    <!-- ... 11 more lines at 30deg intervals ... -->
    <!-- Zodiac glyphs positioned on outer ring -->
    <!-- Aries at 270deg -->
    <text x="25" y="185" text-anchor="middle" class="zodiac-glyph">♈</text>
    <!-- Taurus at 240deg -->
    <!-- ... etc counterclockwise ... -->
    <!-- Planet markers -->
    <circle cx="50" cy="130" r="6" fill="var(--gold)" class="planet-marker"/>
    <text x="50" y="115" text-anchor="middle" class="planet-label">☉</text>
  </svg>
</div>
```

```css
.zodiac-wheel {
  width: var(--wheel-size, 360px);
  height: var(--wheel-size, 360px);
  position: relative;
}

.zodiac-svg {
  width: 100%;
  height: 100%;
}

.zodiac-glyph {
  font-family: 'Inter', sans-serif;
  font-weight: 400;
  font-size: 16px;
  fill: var(--gold);
}

.planet-marker {
  filter: drop-shadow(0 0 8px rgba(184, 146, 74, 0.5));
}

.planet-label {
  font-family: 'Inter', sans-serif;
  font-size: 12px;
  fill: var(--gold);
}
```

### HD Bodygraph Mini

```html
<div class="bodygraph-mini">
  <svg viewBox="0 0 200 340" width="200" height="340">
    <!-- 9 centers in correct topology -->
    <!-- Head (top) -->
    <circle cx="100" cy="30" r="18" class="hd-center hd-center-open"/>
    <!-- Ajna -->
    <circle cx="100" cy="75" r="18" class="hd-center hd-center-defined" style="--center-color: #4A6CF7"/>
    <!-- Throat -->
    <circle cx="100" cy="120" r="18" class="hd-center hd-center-open"/>
    <!-- G Center -->
    <circle cx="100" cy="165" r="18" class="hd-center hd-center-defined" style="--center-color: #B8924A"/>
    <!-- Heart/Ego -->
    <circle cx="145" cy="155" r="14" class="hd-center hd-center-open"/>
    <!-- Spleen -->
    <circle cx="55" cy="210" r="16" class="hd-center hd-center-open"/>
    <!-- Sacral -->
    <circle cx="100" cy="235" r="18" class="hd-center hd-center-defined" style="--center-color: #C0392B"/>
    <!-- Solar Plexus -->
    <circle cx="145" cy="220" r="16" class="hd-center hd-center-open"/>
    <!-- Root -->
    <circle cx="100" cy="300" r="18" class="hd-center hd-center-defined" style="--center-color: #C0392B"/>
    <!-- Channels as lines between centers -->
    <line x1="100" y1="93" x2="100" y2="102" class="hd-channel"/>
  </svg>
</div>
```

```css
.hd-center {
  transition: all 0.3s;
}

.hd-center-defined {
  fill: var(--center-color, #B8924A);
  opacity: 0.7;
  filter: drop-shadow(0 0 12px var(--center-color));
}

.hd-center-open {
  fill: transparent;
  stroke: rgba(237, 234, 228, 0.3);
  stroke-width: 1;
}

.hd-channel {
  stroke: rgba(237, 234, 228, 0.4);
  stroke-width: 2;
}
```

### GK Spectrum Bar

```html
<div class="gk-spectrum">
  <div class="gk-zone gk-shadow">
    <span class="gk-zone-label">Ombra</span>
    <span class="gk-zone-name">Purposelessness</span>
  </div>
  <div class="gk-zone gk-gift">
    <span class="gk-zone-label">Dono</span>
    <span class="gk-zone-name">Totality</span>
  </div>
  <div class="gk-zone gk-siddhi">
    <span class="gk-zone-label">Siddhi</span>
    <span class="gk-zone-name">Immortality</span>
  </div>
</div>
```

```css
.gk-spectrum {
  display: flex;
  width: 100%;
  max-width: 500px;
  height: 60px;
  border-radius: 30px;
  overflow: hidden;
}

.gk-zone {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 2px;
}

.gk-shadow {
  background: linear-gradient(135deg, #3a1c1c 0%, #5a2a2a 100%);
}

.gk-gift {
  background: linear-gradient(135deg, #5a4a20 0%, #B8924A 100%);
}

.gk-siddhi {
  background: linear-gradient(135deg, #B8924A 0%, #EDEAE4 100%);
}

.gk-zone-label {
  font-family: 'Inter', sans-serif;
  font-weight: 600;
  font-size: 9px;
  letter-spacing: 0.3em;
  text-transform: uppercase;
  color: rgba(237, 234, 228, 0.6);
}

.gk-zone-name {
  font-family: 'Playfair Display', serif;
  font-weight: 500;
  font-size: 13px;
  color: var(--pearl);
}
```

### Data Label

```html
<span class="data-label data-label-gold">Gate 44 — Alertness</span>
<span class="data-label data-label-emerald">AI Strategy Layer</span>
```

```css
.data-label {
  font-family: 'Inter', sans-serif;
  font-weight: 500;
  font-size: 11px;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  padding: 4px 12px;
  border-radius: 2px;
}

.data-label-gold {
  color: var(--gold);
  border: 1px solid rgba(184, 146, 74, 0.3);
}

.data-label-emerald {
  color: var(--emerald);
  border: 1px solid rgba(0, 158, 116, 0.3);
}
```

### Pull Quote

```html
<blockquote class="pull-quote">
  Strategy meets Soul.
</blockquote>
```

```css
.pull-quote {
  font-family: 'Playfair Display', serif;
  font-weight: 400;
  font-style: italic;
  font-size: clamp(20px, 2.5vw, 28px);
  line-height: 1.4;
  color: var(--gold);
  text-align: center;
  max-width: 80%;
  margin: 0 auto;
  padding: 24px 0;
}
```

---

## Regole di Composizione

1. **Max 3 primitive strutturali** per infografica — meno e sempre meglio
2. **Gerarchia per dimensione e luminosita** — elemento piu importante = piu grande e luminoso
3. **Asse centrale** — tutto allineato su asse verticale o centro radiale
4. **Breathing space** — minimo `48px` tra primitive, `64px` dai bordi
5. **Layer order:**
   - Z-0: sfondo (gradiente o colore solido)
   - Z-1: Bioform o decorazioni di sfondo (opacity 0.3-0.6)
   - Z-2: primitive strutturali (Orb, Totem, Vesica, etc.)
   - Z-3: primitive di contenuto (Wheel, Bodygraph, Spectrum)
   - Z-4: testo (titoli, label, caption)
6. **Firma Grimoria** — ogni infografica include in basso a destra o al centro-basso:
   ```html
   <div class="cosmos-signature">
     <span class="signature-glyph">✦</span>
     <span class="signature-text">GRIMORIA</span>
   </div>
   ```
```

- [ ] **Step 2: Commit**

```bash
git add skills/cosmos-infographic/references/component-library.md
git commit -m "feat: add component library — 12 visual primitives with CSS/SVG code"
```

---

### Task 4: Create Semantic Rules reference

**Files:**
- Create: `skills/cosmos-infographic/references/semantic-rules.md`

- [ ] **Step 1: Write semantic rules for Zodiac, HD, and GK positioning**

Create `skills/cosmos-infographic/references/semantic-rules.md` with this content:

```markdown
# Regole Semantiche — Posizionamento Esoterico

> Questo file definisce COME posizionare correttamente i dati astrologici, Human Design e Gene Keys nelle infografiche. Per il SIGNIFICATO dei dati, consulta i framework in `skills/astro-hd-gk-profiler/references/`.

---

## 1. Zodiac Wheel — Posizionamento Segni e Pianeti

### Ordine dei segni (senso antiorario da ore 9)

| Posizione (gradi) | Segno | Glifo | Angolo sul cerchio |
|--------------------|-------|-------|-------------------|
| 0-30 | Ariete | ♈ | 270° (ore 9) |
| 30-60 | Toro | ♉ | 240° |
| 60-90 | Gemelli | ♊ | 210° |
| 90-120 | Cancro | ♋ | 180° (ore 6) |
| 120-150 | Leone | ♌ | 150° |
| 150-180 | Vergine | ♍ | 120° |
| 180-210 | Bilancia | ♎ | 90° (ore 3) |
| 210-240 | Scorpione | ♏ | 60° |
| 240-270 | Sagittario | ♐ | 30° |
| 270-300 | Capricorno | ♑ | 0° (ore 12) |
| 300-330 | Acquario | ♒ | 330° |
| 330-360 | Pesci | ♓ | 300° |

### Calcolo posizione pianeta

Per posizionare un pianeta a N gradi nel segno:
1. Trova l'angolo iniziale del segno dalla tabella
2. Il grado 0 del segno e al bordo iniziale, il grado 30 al bordo finale
3. `angolo_pianeta = angolo_segno - (grado_nel_segno * 30 / 30)`
4. Converti in coordinate SVG: `x = cx + r * cos(angolo)`, `y = cy + r * sin(angolo)`

### Simboli planetari

| Pianeta | Glifo | Colore |
|---------|-------|--------|
| Sole | ☉ | Gold |
| Luna | ☽ | Pearl/Silver |
| Mercurio | ☿ | Emerald |
| Venere | ♀ | Cosmos Violet |
| Marte | ♂ | Rosso scuro `#8B2500` |
| Giove | ♃ | Cosmos Blue |
| Saturno | ♄ | Charcoal |
| Urano | ♅ | Cosmos Blue |
| Nettuno | ♆ | Cosmos Violet |
| Plutone | ♇ | Obsidian con glow Gold |

### Aspetti (linee tra pianeti)

| Aspetto | Angolo | Colore linea | Stile |
|---------|--------|-------------|-------|
| Congiunzione | 0° | Gold | Solida, 2px |
| Sestile | 60° | Emerald | Tratteggiata |
| Quadratura | 90° | `#8B2500` | Solida, 1px |
| Trigono | 120° | Cosmos Blue | Solida, 1.5px |
| Opposizione | 180° | Cosmos Violet | Solida, 1px |

---

## 2. Human Design Bodygraph — Topologia dei Centri

### Posizioni dei 9 centri (coordinate relative in viewBox 200x340)

| Centro | cx | cy | Raggio | Colore (definito) |
|--------|----|----|--------|-------------------|
| Testa (Crown) | 100 | 30 | 18 | `#B8924A` (Gold) |
| Ajna (Mind) | 100 | 75 | 18 | `#4A6CF7` (Cosmos Blue) |
| Gola (Throat) | 100 | 120 | 18 | `#8B6914` (Marrone/Ocra) |
| G Center (Self) | 100 | 165 | 18 | `#B8924A` (Gold) |
| Cuore/Ego (Heart) | 145 | 155 | 14 | `#C0392B` (Rosso) |
| Milza (Spleen) | 55 | 210 | 16 | `#8B6914` (Marrone) |
| Sacrale (Sacral) | 100 | 235 | 18 | `#C0392B` (Rosso) |
| Plesso Solare (Solar Plexus) | 145 | 220 | 16 | `#8B6914` (Marrone/Ocra) |
| Radice (Root) | 100 | 300 | 18 | `#C0392B` (Rosso) |

### Canali principali (connessioni tra centri)

I canali si disegnano come linee tra i centri che collegano. I piu comuni:

| Canale | Da → A | Gate |
|--------|--------|------|
| 64-47 | Testa → Ajna | Confusione → Comprensione |
| 61-24 | Testa → Ajna | Mistero → Razionalizzazione |
| 63-4 | Testa → Ajna | Dubbio → Formulazione |
| 17-62 | Ajna → Gola | Opinione → Dettaglio |
| 43-23 | Ajna → Gola | Insight → Assimilazione |
| 20-57 | Gola → Milza | Contemplazione → Intuizione |
| 34-20 | Sacrale → Gola | Potere → Contemplazione |

### Mappa Gate → Centro

Per posizionare un gate nel centro corretto, consulta `skills/astro-hd-gk-profiler/references/human-design-framework.md`. I 64 gate sono distribuiti nei 9 centri. Quando l'utente specifica un gate, Claude verifica in quale centro si trova prima di posizionarlo.

### Visualizzazione

- **Centro definito**: cerchio pieno con colore del centro, opacita 0.7, `drop-shadow` glow
- **Centro aperto**: cerchio vuoto con bordo sottile `rgba(237, 234, 228, 0.3)`
- **Canale attivo**: linea tra i due centri, colore `rgba(237, 234, 228, 0.4)`, stroke-width 2

---

## 3. Gene Keys — Spettro e Sequenze

### Formato Trittico

Ogni Gene Key ha 3 frequenze. La skill usa SEMPRE i nomi specifici:

Formato: `GK [numero]: [Ombra] → [Dono] → [Siddhi]`

Esempio: `GK 28: Purposelessness → Totality → Immortality`

Per trovare i nomi corretti, consulta `skills/astro-hd-gk-profiler/references/gene-keys-framework.md`.

### Sequenze Gene Keys

| Sequenza | Sfere | Uso tipico |
|----------|-------|-----------|
| **Activation** | Life's Work, Evolution, Radiance, Purpose | Profili business, posizionamento |
| **Venus** | Attraction, IQ, EQ, SQ | Relazioni, comunicazione |
| **Pearl** | Vocation, Culture, Brand, Pearl | Prosperita, brand |

### Visualizzazione GK Spectrum Bar

La barra e divisa in 3 zone con gradiente:
- **Ombra** (sinistra): sfondo scuro rosso-marrone `#3a1c1c` → `#5a2a2a`
- **Dono** (centro): sfondo warm gold `#5a4a20` → `#B8924A`
- **Siddhi** (destra): sfondo luminoso gold → pearl `#B8924A` → `#EDEAE4`

La transizione visiva comunica la trasformazione dalla frequenza bassa a quella alta.

---

## 4. Regola d'Oro

**Se l'utente fornisce dati specifici (posizioni planetarie, gate, chiavi), la skill li VERIFICA contro i framework di riferimento prima di posizionarli nel visual.**

Se i dati non corrispondono (es. "Gate 44 nel centro Sacrale" — sbagliato, Gate 44 e nel centro Milza), la skill:
1. NON procede silenziosamente
2. CHIEDE conferma: "Il Gate 44 appartiene al centro Milza, non al Sacrale. Procedo con la posizione corretta?"
3. Solo dopo conferma, genera l'infografica
```

- [ ] **Step 2: Commit**

```bash
git add skills/cosmos-infographic/references/semantic-rules.md
git commit -m "feat: add semantic rules — zodiac, HD bodygraph, GK positioning"
```

---

### Task 5: Create base Obsidian template

**Files:**
- Create: `skills/cosmos-infographic/templates/base-obsidian.html`

- [ ] **Step 1: Write the dark mode base template with all shared CSS**

Create `skills/cosmos-infographic/templates/base-obsidian.html` with the complete HTML template. This is the starting scaffold that Claude customizes for each infographic:

```html
<!DOCTYPE html>
<html lang="it">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Cosmos Infographic — [TITOLO]</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600&family=Playfair+Display:ital,wght@0,400;0,500;0,600;0,700;1,400&display=swap" rel="stylesheet">
  <style>
    /* === COSMOS DESIGN SYSTEM — OBSIDIAN MODE === */
    :root {
      --obsidian: #0E0E10;
      --pearl: #EDEAE4;
      --pearl-deep: #E0DDD7;
      --gold: #B8924A;
      --emerald: #009E74;
      --charcoal: #3A3530;
      --stone: #7A7670;
      --cosmos-blue: #4A6CF7;
      --cosmos-blue-end: #7B5EA7;
      --cosmos-violet: #9B6DFF;
      --cosmos-violet-end: #FF6FB5;
      --cosmos-amber: #C4A24D;
      --cosmos-amber-end: #1A1A1A;
    }

    * { margin: 0; padding: 0; box-sizing: border-box; }

    body {
      background: var(--obsidian);
      color: var(--pearl);
      font-family: 'Inter', sans-serif;
      font-weight: 300;
      -webkit-font-smoothing: antialiased;
    }

    .canvas {
      width: 1080px;
      height: 1350px; /* Default: vertical social. Change for other formats */
      margin: 0 auto;
      position: relative;
      overflow: hidden;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      padding: 64px;
    }

    /* === TYPOGRAPHY === */
    .display { font-family: 'Playfair Display', serif; font-weight: 700; font-size: clamp(40px, 5vw, 56px); line-height: 1.1; letter-spacing: -0.02em; color: var(--pearl); text-align: center; }
    .h1 { font-family: 'Playfair Display', serif; font-weight: 600; font-size: clamp(28px, 3.5vw, 40px); line-height: 1.2; letter-spacing: -0.01em; color: var(--gold); text-align: center; }
    .h2 { font-family: 'Playfair Display', serif; font-weight: 500; font-size: clamp(22px, 2.5vw, 28px); line-height: 1.3; color: var(--pearl); text-align: center; }
    .body-text { font-family: 'Inter', sans-serif; font-weight: 300; font-size: 17px; line-height: 1.85; letter-spacing: 0.01em; color: var(--stone); }
    .label { font-family: 'Inter', sans-serif; font-weight: 600; font-size: 12px; line-height: 1.5; letter-spacing: 0.3em; text-transform: uppercase; color: var(--emerald); }
    .caption { font-family: 'Inter', sans-serif; font-weight: 300; font-size: 13px; line-height: 1.5; color: rgba(237, 234, 228, 0.5); }

    /* === SHARED PRIMITIVE CSS === */
    /* Orb */
    .orb { border-radius: 50%; position: relative; display: flex; align-items: center; justify-content: center; width: var(--orb-size, 200px); height: var(--orb-size, 200px); }
    .orb-core { width: 60%; height: 60%; border-radius: 50%; display: flex; align-items: center; justify-content: center; z-index: 2; }
    .orb-label { font-family: 'Inter', sans-serif; font-weight: 400; font-size: calc(var(--orb-size, 200px) * 0.15); color: var(--pearl); }
    .orb-violet { background: radial-gradient(circle at 50% 50%, rgba(155,109,255,0.6) 0%, rgba(255,111,181,0.3) 40%, rgba(155,109,255,0.05) 70%, transparent 100%); box-shadow: 0 0 60px rgba(155,109,255,0.3), 0 0 120px rgba(155,109,255,0.1); }
    .orb-blue { background: radial-gradient(circle at 50% 50%, rgba(74,108,247,0.6) 0%, rgba(123,94,167,0.3) 40%, rgba(74,108,247,0.05) 70%, transparent 100%); box-shadow: 0 0 60px rgba(74,108,247,0.3), 0 0 120px rgba(74,108,247,0.1); }
    .orb-amber { background: radial-gradient(circle at 50% 50%, rgba(196,162,77,0.7) 0%, rgba(196,162,77,0.2) 40%, rgba(26,26,26,0.1) 70%, transparent 100%); box-shadow: 0 0 60px rgba(196,162,77,0.3), 0 0 120px rgba(196,162,77,0.1); }
    .orb-gold { background: radial-gradient(circle at 50% 50%, rgba(184,146,74,0.7) 0%, rgba(184,146,74,0.3) 40%, rgba(184,146,74,0.05) 70%, transparent 100%); box-shadow: 0 0 60px rgba(184,146,74,0.3), 0 0 120px rgba(184,146,74,0.1); }
    .orb-emerald { background: radial-gradient(circle at 50% 50%, rgba(0,158,116,0.6) 0%, rgba(0,158,116,0.2) 40%, rgba(0,158,116,0.05) 70%, transparent 100%); box-shadow: 0 0 60px rgba(0,158,116,0.3), 0 0 120px rgba(0,158,116,0.1); }

    /* Totem */
    .totem { display: flex; flex-direction: column; align-items: center; gap: 0; }
    .totem-connector { width: 2px; height: 40px; background: linear-gradient(to bottom, rgba(155,109,255,0.4), rgba(155,109,255,0.05)); margin: -20px 0; z-index: 1; }

    /* Pull Quote */
    .pull-quote { font-family: 'Playfair Display', serif; font-weight: 400; font-style: italic; font-size: clamp(20px, 2.5vw, 28px); line-height: 1.4; color: var(--gold); text-align: center; max-width: 80%; margin: 0 auto; padding: 24px 0; }

    /* Data Label */
    .data-label { font-family: 'Inter', sans-serif; font-weight: 500; font-size: 11px; letter-spacing: 0.15em; text-transform: uppercase; padding: 4px 12px; border-radius: 2px; }
    .data-label-gold { color: var(--gold); border: 1px solid rgba(184,146,74,0.3); }
    .data-label-emerald { color: var(--emerald); border: 1px solid rgba(0,158,116,0.3); }

    /* GK Spectrum */
    .gk-spectrum { display: flex; width: 100%; max-width: 500px; height: 60px; border-radius: 30px; overflow: hidden; }
    .gk-zone { flex: 1; display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 2px; }
    .gk-shadow { background: linear-gradient(135deg, #3a1c1c 0%, #5a2a2a 100%); }
    .gk-gift { background: linear-gradient(135deg, #5a4a20 0%, #B8924A 100%); }
    .gk-siddhi { background: linear-gradient(135deg, #B8924A 0%, #EDEAE4 100%); }
    .gk-zone-label { font-family: 'Inter', sans-serif; font-weight: 600; font-size: 9px; letter-spacing: 0.3em; text-transform: uppercase; color: rgba(237,234,228,0.6); }
    .gk-zone-name { font-family: 'Playfair Display', serif; font-weight: 500; font-size: 13px; color: var(--pearl); }

    /* Signature */
    .cosmos-signature { position: absolute; bottom: 32px; display: flex; align-items: center; gap: 8px; }
    .signature-glyph { font-size: 14px; color: var(--gold); }
    .signature-text { font-family: 'Inter', sans-serif; font-weight: 600; font-size: 9px; letter-spacing: 0.3em; text-transform: uppercase; color: rgba(237,234,228,0.3); }
  </style>
</head>
<body>
  <div class="canvas">

    <!-- === CONTENT GOES HERE === -->
    <!-- Claude assembles primitives from component-library.md -->

    <!-- Signature -->
    <div class="cosmos-signature">
      <span class="signature-glyph">✦</span>
      <span class="signature-text">GRIMORIA</span>
    </div>
  </div>
</body>
</html>
```

- [ ] **Step 2: Open in browser preview and verify the base template renders correctly**

Open `skills/cosmos-infographic/templates/base-obsidian.html` in the browser. Expected: dark background (#0E0E10), only the Grimoria signature visible at the bottom center.

- [ ] **Step 3: Commit**

```bash
git add skills/cosmos-infographic/templates/base-obsidian.html
git commit -m "feat: add base Obsidian (dark) HTML template"
```

---

### Task 6: Create base Pearl template

**Files:**
- Create: `skills/cosmos-infographic/templates/base-pearl.html`

- [ ] **Step 1: Write the light mode base template**

Create `skills/cosmos-infographic/templates/base-pearl.html`. This is identical to the Obsidian template except for the mode-specific overrides. Key differences:

- `body` background: `var(--pearl)` instead of `var(--obsidian)`
- `body` color: `var(--obsidian)` instead of `var(--pearl)`
- `.canvas` gets class `pearl-mode`
- Add Pearl-mode overrides after the shared CSS:

```css
/* === PEARL MODE OVERRIDES === */
body {
  background: var(--pearl);
  color: var(--obsidian);
}

.display { color: var(--obsidian); }
.h2 { color: var(--obsidian); }
.body-text { color: var(--charcoal); }
.caption { color: rgba(14, 14, 16, 0.4); }
.orb-label { color: var(--obsidian); }

.orb-violet { background: radial-gradient(circle at 50% 50%, rgba(155,109,255,0.4) 0%, rgba(255,111,181,0.2) 40%, rgba(155,109,255,0.03) 70%, transparent 100%); box-shadow: 0 0 40px rgba(155,109,255,0.15), 0 0 80px rgba(155,109,255,0.05); }
.orb-blue { background: radial-gradient(circle at 50% 50%, rgba(74,108,247,0.4) 0%, rgba(123,94,167,0.2) 40%, rgba(74,108,247,0.03) 70%, transparent 100%); box-shadow: 0 0 40px rgba(74,108,247,0.15), 0 0 80px rgba(74,108,247,0.05); }
.orb-amber { background: radial-gradient(circle at 50% 50%, rgba(196,162,77,0.5) 0%, rgba(196,162,77,0.15) 40%, rgba(26,26,26,0.05) 70%, transparent 100%); box-shadow: 0 0 40px rgba(196,162,77,0.15), 0 0 80px rgba(196,162,77,0.05); }

.hd-center-open { stroke: rgba(14, 14, 16, 0.2); }
.hd-channel { stroke: rgba(14, 14, 16, 0.3); }

.signature-text { color: rgba(14, 14, 16, 0.2); }
```

Copy the full Obsidian template, apply these overrides, and change the `<title>` to include "Pearl Mode".

- [ ] **Step 2: Open in browser preview and verify the Pearl template renders correctly**

Expected: Pearl background (#EDEAE4), Grimoria signature visible in dark text at bottom.

- [ ] **Step 3: Commit**

```bash
git add skills/cosmos-infographic/templates/base-pearl.html
git commit -m "feat: add base Pearl (light) HTML template"
```

---

### Task 7: Smoke Test — Generate a test infographic

**Files:**
- Create: `infographic-2026-04-10-test-4-tipi-hd.html` (test output, can be deleted after)

- [ ] **Step 1: Using the skill workflow, generate a test infographic**

Prompt: "Infografica per Instagram sui 4 tipi energetici Human Design"

Follow the SKILL.md workflow:
1. INTERPRET: social, 1080x1350, educational, Obsidian mode
2. SELECT: Totem (4 orb verticali) + Data Label per nomi tipo
3. ENRICH: Manifestor (Gold), Generator (Rosso), Projector (Blue), Reflector (Pearl/Silver)
4. COMPOSE: Start from base-obsidian.html, insert 4 Orbs in Totem layout with Data Labels
5. VERIFY: anti-pattern check, leggibilita, no semantic errors
6. DELIVER: save as `infographic-2026-04-10-test-4-tipi-hd.html`

Expected result: A dark vertical infographic with 4 glowing orbs stacked vertically, each with the HD type name below, Grimoria signature at bottom.

- [ ] **Step 2: Open in browser preview and take screenshot**

Verify visually that:
- Dark background renders
- 4 orbs are visible with distinct colors
- Typography is correct (Playfair for title, Inter for labels)
- Minimum 30% white space
- Grimoria signature visible
- No flat colors, no hard borders

- [ ] **Step 3: Commit test infographic (or delete if only for validation)**

```bash
git add infographic-2026-04-10-test-4-tipi-hd.html
git commit -m "test: smoke test infographic — 4 HD types in Totem layout"
```

---

### Task 8: Final commit and skill registration

**Files:**
- Modify: `skills/cosmos-infographic/SKILL.md` (if any adjustments from smoke test)

- [ ] **Step 1: Verify all files are committed**

```bash
git status
git log --oneline -8
```

Expected: 7 commits from Tasks 1-7, all files tracked.

- [ ] **Step 2: Test skill trigger by reading SKILL.md**

Read the SKILL.md and verify:
- Frontmatter `name` and `description` are correct
- Trigger words cover all expected inputs
- Workflow references all 3 reference files
- Output format is documented

- [ ] **Step 3: Final commit if any adjustments were made**

```bash
git add -A skills/cosmos-infographic/
git commit -m "feat: finalize cosmos-infographic skill — all references and templates complete"
```
