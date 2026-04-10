# Cosmos Infographic Skill — Design Spec

> Skill per generare infografiche personalizzate nello stile visivo Cosmos di Grimoria.
> Approccio: Component Library + Composizione (Approccio B).

---

## 1. Skill Overview & Trigger

**Nome skill:** `cosmos-infographic`

**Trigger:** Si attiva quando l'utente chiede di creare un'infografica, un visual, una mappa visiva, o qualsiasi grafica nello stile Grimoria/Cosmos. Esempi:

- "Fammi un'infografica sui 12 profili Human Design"
- "Crea un visual per Substack sulla Sequenza di Attivazione Gene Keys"
- "Infografica per il profilo di [cliente] — Sole Scorpione, Gate 44, GK 28"
- "Visual per il pitch deck di Grimoria — i 3 tier di offerta"

**Input:** Prompt libero in linguaggio naturale. Parametri opzionali:

| Parametro | Opzioni | Default |
|-----------|---------|---------|
| **Formato** | "per Instagram" (1080x1350), "quadrato" (1080x1080), "per slide" (1920x1080) | 1080x1350 verticale |
| **Mood** | "dark mode" (Obsidian), "pearl/light", "alta densita di dati", "minimal" | Claude sceglie in base al contesto |
| **Destinazione** | "per Substack", "per cliente", "per pitch" | Informa il livello di rigore semantico |

**Output:** File HTML autocontenuto nella cartella del progetto.
Nome file: `infographic-YYYY-MM-DD-<slug>.html`

---

## 2. Cosmos Visual DNA — Sistema di Stile

### 2.1 Principi fondamentali

| Principio | Regola |
|-----------|--------|
| **Geometria sacra come struttura** | Cerchi, vesica piscis, simmetria assiale. Mai griglie rettangolari piatte — tutto e curvo, orbitale, concentrico |
| **Centri luminosi** | Ogni composizione ha almeno un punto focale radiante — un nucleo che attrae lo sguardo (radial-gradient con blur) |
| **Gradienti eterici** | Mai colori piatti. Tutto respira con sfumature: `radial-gradient`, `conic-gradient`, `box-shadow` con blur alto |
| **Il vuoto e sacro** | Minimo 30% dello spazio e respiro vuoto. Mai riempire tutto — il vuoto e parte del messaggio |
| **Doppio registro** | Due modalita: **Obsidian** (sfondo scuro, glow chiari) e **Pearl** (sfondo chiaro, ombre morbide). Claude sceglie in base al contesto |
| **Organico dentro il geometrico** | Le forme sono precise, ma i riempimenti sono morbidi, sfumati, luminescenti — mai bordi netti al 100% |

### 2.2 Palette

Eredita i design tokens Grimoria (`Grimoria Analisi e Branding/Brand_Identity_Master/design-tokens.json`) + estensioni Cosmos:

| Colore | HEX | Uso |
|--------|-----|-----|
| Obsidian | `#0E0E10` | Sfondo dark mode |
| Pearl | `#EDEAE4` | Sfondo light mode |
| Gold | `#B8924A` | Accenti esoterici, astrologia, titoli manifesto |
| Emerald | `#009E74` | Accenti tech/AI, label sistema |
| **Cosmos Blue** | `#4A6CF7` → `#7B5EA7` | Gradienti eterici freddi |
| **Cosmos Violet** | `#9B6DFF` → `#FF6FB5` | Aura, campi energetici, nuclei radianti |
| **Cosmos Amber** | `#C4A24D` → `#1A1A1A` | Singolarita calde, buchi neri dorati |

### 2.3 Tipografia

- **Titoli:** Playfair Display — mai per body
- **Body/Label:** Inter weight 300 — line-height 1.85 per leggibilita
- **Glifi esoterici:** Unicode symbols (☽ ☿ ♀ ♄ ⊕) in Inter weight 400, colore Gold

### 2.4 Anti-pattern

- Mai clipart o icone flat-style
- Mai bordi 1px solid netti su forme principali — solo glow e ombre
- Mai piu di 3 colori dominanti per infografica
- Mai testo sotto i 13px
- Mai layout a griglia rettangolare rigida — sempre flusso circolare/organico

---

## 3. Component Library — Primitive Visive

### 3.1 Primitive strutturali

| Primitiva | Descrizione | Reference Cosmos |
|-----------|-------------|------------------|
| **Orb** | Sfera radiante con gradiente eterico e nucleo luminoso. Scalabile. Puo contenere testo o simbolo al centro | csms-001, csms-003 |
| **Totem** | Composizione verticale di 2-4 orb allineati sull'asse centrale, con connessioni luminose tra loro | csms-001, csms-003 |
| **Aura Ring** | Cerchio con alone sfumato concentrico — usato per mappe circolari con label attorno al perimetro | csms-002 |
| **Vesica** | Due cerchi sovrapposti con zona di intersezione luminosa (vesica piscis). Per dualita, integrazioni, incroci | csms-006 |
| **Sacred Grid** | Griglia di punti/cerchi con pattern geometrico — per visualizzare sistemi, confronti, tassonomie | csms-005 |
| **Bioform** | Forma simmetrica organica generata da curve bezier sovrapposte con opacita variabile — per energia, campo, frequenza | csms-007 |
| **Tech Blueprint** | Linee sottili, cerchi concentrici con annotazioni, stile disegno tecnico su sfondo pearl/pergamena | csms-004 |

### 3.2 Primitive di contenuto

| Primitiva | Descrizione |
|-----------|-------------|
| **Zodiac Wheel** | Cerchio a 12 settori con glifi zodiacali posizionati correttamente. Evidenzia i segni rilevanti con glow Gold |
| **HD Bodygraph Mini** | Schema semplificato dei 9 centri Human Design, con centri definiti/aperti visualizzati come orb pieni/vuoti |
| **GK Spectrum Bar** | Barra orizzontale con 3 zone (Ombra → Dono → Siddhi) e gradiente dal rosso scuro al gold al bianco |
| **Data Label** | Etichetta minimalista: testo Inter 11px uppercase, tracking largo, colore Emerald o Gold |
| **Pull Quote** | Testo Playfair Display italic, colore Gold, centrato — per frasi manifesto o insight chiave |

### 3.3 Regole di composizione

1. **Massimo 3 primitive strutturali** per infografica
2. **Gerarchia per dimensione e luminosita** — l'elemento piu importante e il piu grande e il piu luminoso
3. **Asse centrale** — le composizioni seguono un asse verticale o un centro radiale, mai layout sparsi
4. **Breathing space** — minimo `48px` di padding tra primitive, minimo `64px` dai bordi
5. **Layer order** — sfondo gradiente → primitive strutturali → primitive di contenuto → testo

---

## 4. Semantic Engine — Intelligenza Esoterica

### 4.1 Fonti di conoscenza

La skill si appoggia alla knowledge base del progetto:

| Sistema | Fonte | Cosa sa |
|---------|-------|---------|
| **Astrologia** | `skills/astro-hd-gk-profiler/references/prompt-astrologica.md` | Posizioni zodiacali, case, aspetti, dignita planetarie |
| **Human Design** | `skills/astro-hd-gk-profiler/references/human-design-framework.md` | 9 centri, 64 gate, 6 profili, tipi energetici, canali |
| **Gene Keys** | `skills/astro-hd-gk-profiler/references/gene-keys-framework.md` | 64 chiavi, sequenze (Attivazione, Venere, Perla), spettro Ombra→Dono→Siddhi |
| **Sintesi** | `skills/astro-hd-gk-profiler/references/synthesis-framework.md` | Corrispondenze incrociate tra i 3 sistemi |

### 4.2 Regole semantiche

**Zodiac Wheel:**
- I 12 segni sono posizionati in senso antiorario partendo dall'Ariete a ore 9 (ascendente standard)
- I pianeti si posizionano nel grado corretto del segno
- Gli aspetti (congiunzione, opposizione, trigono, quadratura, sestile) si visualizzano come linee tra pianeti con colori diversi

**HD Bodygraph Mini:**
- I 9 centri rispettano la topologia del Bodygraph reale (Testa in alto, Radice in basso)
- Centro definito = Orb pieno con colore del centro (colori derivati da `human-design-framework.md`)
- Centro aperto = Orb vuoto con bordo sottile
- I Gate si posizionano nel centro corretto secondo la mappa gate→centro del framework

**GK Spectrum:**
- Ogni Gene Key ha il suo specifico trittico Ombra/Dono/Siddhi — la skill usa i nomi corretti, mai generici
- Esempio: Gene Key 28 = "Purposelessness → Totality → Immortality"

**Regola d'oro:** Se il prompt menziona dati specifici di un cliente, la skill li verifica contro i framework di riferimento prima di posizionarli. Se qualcosa non torna, chiede conferma.

---

## 5. Generation Workflow

### 5.1 Pipeline

```
Prompt utente
    ↓
[1] INTERPRET — Analisi del prompt
    → Identifica: soggetto, destinazione, formato, mood
    → Classifica: social / cliente / pitch
    ↓
[2] SELECT — Scelta primitive
    → Seleziona max 3 primitive strutturali adatte al contenuto
    → Seleziona primitive di contenuto necessarie
    → Sceglie registro: Obsidian o Pearl
    ↓
[3] ENRICH — Arricchimento semantico
    → Se ci sono dati esoterici: consulta i framework di riferimento
    → Posiziona simboli/dati secondo le regole semantiche
    → Se dati ambigui o incompleti: chiede all'utente
    ↓
[4] COMPOSE — Assemblaggio
    → Genera HTML con le primitive selezionate
    → Applica design tokens Grimoria + palette Cosmos
    → Rispetta regole di composizione (asse, breathing, gerarchia)
    ↓
[5] VERIFY — Controllo qualita
    → Anti-pattern check (no clipart, no bordi netti, no griglia piatta)
    → Verifica leggibilita (testo ≥13px, contrasto sufficiente)
    → Verifica accuratezza semantica dei dati esoterici
    ↓
[6] DELIVER — Output
    → Salva come infographic-YYYY-MM-DD-<slug>.html
    → Apre nel browser preview per verifica visiva
    → Chiede conferma all'utente
```

### 5.2 Esempi di flusso

**Prompt:** "Infografica per Instagram sui 4 tipi energetici Human Design"
- INTERPRET → social, verticale 1080x1350, educational
- SELECT → Totem (4 orb verticali, uno per tipo), Data Label per i nomi
- ENRICH → Manifestor, Generator, Projector, Reflector con colori semantici HD
- COMPOSE → Obsidian background, 4 orb con gradiente specifico per tipo
- DELIVER → `infographic-2026-04-10-4-tipi-hd.html`

**Prompt:** "Visual per il profilo di Maria — Sole Cancro, Luna Capricorno, Gate 2, GK 14"
- INTERPRET → cliente, default verticale, profilo personale
- SELECT → Vesica (dualita Sole/Luna) + HD Bodygraph Mini + GK Spectrum Bar
- ENRICH → Cancro a ore 3 sulla wheel, Capricorno a ore 9. Gate 2 nel centro G. GK 14: Compromise → Competence → Bounteousness
- COMPOSE → Pearl background (piu intimo per deliverable cliente), Gold accents
- DELIVER → `infographic-2026-04-10-profilo-maria.html`

---

## 6. Struttura file della skill

```
skills/cosmos-infographic/
├── SKILL.md                          # Skill definition + trigger + workflow
├── references/
│   ├── cosmos-visual-dna.md          # Sezione 2: regole visive, palette, anti-pattern
│   ├── component-library.md          # Sezione 3: primitive e regole composizione
│   └── semantic-rules.md             # Sezione 4: regole posizionamento esoteriche
└── templates/
    ├── base-obsidian.html            # Template base dark mode
    └── base-pearl.html               # Template base light mode
```

I framework esoterici NON vengono duplicati — la skill referenzia quelli in `skills/astro-hd-gk-profiler/references/`.
