# Cosmos Infographic — Component Library

This is the CORE reference file for the Cosmos Infographic skill. When composing an infographic, read this file and copy the HTML/CSS blocks for each primitive you need. All code is copy-paste ready for a self-contained HTML file.

---

## CSS Variables (always include in `<style>` or `:root`)

```css
:root {
  --obsidian: #0E0E10;
  --pearl: #EDEAE4;
  --gold: #B8924A;
  --emerald: #009E74;
  --charcoal: #3A3530;
  --stone: #7A7670;
  --cosmos-blue: #4A6CF7;
  --cosmos-violet: #9B6DFF;
  --cosmos-violet-end: #FF6FB5;
  --cosmos-amber: #C4A24D;
}
```

---

## Structural Primitives

---

### 1. Orb

Radiating sphere with ethereal gradient and luminous core. Use as focal element for a single concept, archetype, or energy center.

**Variants:** `.orb-blue`, `.orb-violet`, `.orb-amber`, `.orb-gold`, `.orb-emerald`

**HTML:**
```html
<div class="orb orb-blue" style="--orb-size: 200px;">
  <div class="orb-core"></div>
  <div class="orb-label">Label Text</div>
</div>
```

**CSS:**
```css
/* ── Orb Base ── */
.orb {
  position: relative;
  width: var(--orb-size, 200px);
  height: var(--orb-size, 200px);
  border-radius: 50%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

/* Orb Core — luminous inner highlight */
.orb-core {
  position: absolute;
  top: 15%;
  left: 20%;
  width: 35%;
  height: 35%;
  border-radius: 50%;
  background: radial-gradient(circle at 40% 40%, rgba(255,255,255,0.55) 0%, rgba(255,255,255,0) 70%);
  pointer-events: none;
}

/* Orb Label */
.orb-label {
  position: relative;
  z-index: 2;
  font-family: 'Playfair Display', serif;
  font-size: calc(var(--orb-size, 200px) * 0.11);
  color: var(--pearl);
  text-align: center;
  line-height: 1.3;
  padding: 0 12%;
  text-shadow: 0 0 12px rgba(0,0,0,0.6);
}

/* ── Orb Variants ── */
.orb-blue {
  background: radial-gradient(circle at 35% 30%,
    rgba(100,140,255,0.9) 0%,
    rgba(74,108,247,0.75) 35%,
    rgba(30,50,160,0.55) 65%,
    rgba(14,14,16,0) 100%
  );
  box-shadow:
    0 0 40px 10px rgba(74,108,247,0.35),
    0 0 80px 20px rgba(74,108,247,0.18),
    inset 0 0 30px rgba(74,108,247,0.25);
}

.orb-violet {
  background: radial-gradient(circle at 35% 30%,
    rgba(185,150,255,0.9) 0%,
    rgba(155,109,255,0.75) 35%,
    rgba(80,40,180,0.55) 65%,
    rgba(14,14,16,0) 100%
  );
  box-shadow:
    0 0 40px 10px rgba(155,109,255,0.35),
    0 0 80px 20px rgba(155,109,255,0.18),
    inset 0 0 30px rgba(155,109,255,0.25);
}

.orb-amber {
  background: radial-gradient(circle at 35% 30%,
    rgba(220,185,100,0.9) 0%,
    rgba(196,162,77,0.75) 35%,
    rgba(110,75,20,0.55) 65%,
    rgba(14,14,16,0) 100%
  );
  box-shadow:
    0 0 40px 10px rgba(196,162,77,0.35),
    0 0 80px 20px rgba(196,162,77,0.18),
    inset 0 0 30px rgba(196,162,77,0.25);
}

.orb-gold {
  background: radial-gradient(circle at 35% 30%,
    rgba(237,222,180,0.92) 0%,
    rgba(184,146,74,0.8) 35%,
    rgba(100,70,10,0.55) 65%,
    rgba(14,14,16,0) 100%
  );
  box-shadow:
    0 0 40px 10px rgba(184,146,74,0.4),
    0 0 90px 25px rgba(184,146,74,0.2),
    inset 0 0 35px rgba(237,222,180,0.2);
}

.orb-emerald {
  background: radial-gradient(circle at 35% 30%,
    rgba(80,220,170,0.9) 0%,
    rgba(0,158,116,0.75) 35%,
    rgba(0,70,50,0.55) 65%,
    rgba(14,14,16,0) 100%
  );
  box-shadow:
    0 0 40px 10px rgba(0,158,116,0.35),
    0 0 80px 20px rgba(0,158,116,0.18),
    inset 0 0 30px rgba(0,158,116,0.25);
}

/* ── Pearl Mode overrides (reduced opacity for light backgrounds) ── */
.pearl-mode .orb-blue   { opacity: 0.72; }
.pearl-mode .orb-violet { opacity: 0.72; }
.pearl-mode .orb-amber  { opacity: 0.75; }
.pearl-mode .orb-gold   { opacity: 0.78; }
.pearl-mode .orb-emerald{ opacity: 0.72; }
```

---

### 2. Totem

Vertical stack of 2–4 Orbs on a central axis, connected by luminous gradient lines. Use to show hierarchical or sequential relationships between concepts.

**HTML:**
```html
<div class="totem">
  <div class="orb orb-violet" style="--orb-size: 120px;">
    <div class="orb-core"></div>
    <div class="orb-label">Crown</div>
  </div>
  <div class="totem-connector"></div>
  <div class="orb orb-blue" style="--orb-size: 160px;">
    <div class="orb-core"></div>
    <div class="orb-label">Center</div>
  </div>
  <div class="totem-connector"></div>
  <div class="orb orb-gold" style="--orb-size: 120px;">
    <div class="orb-core"></div>
    <div class="orb-label">Root</div>
  </div>
</div>
```

**CSS:**
```css
/* ── Totem ── */
.totem {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0;
}

.totem-connector {
  width: 2px;
  height: 40px;
  margin: -12px 0; /* Overlap orbs slightly */
  background: linear-gradient(
    to bottom,
    rgba(184,146,74,0) 0%,
    rgba(184,146,74,0.6) 30%,
    rgba(184,146,74,0.6) 70%,
    rgba(184,146,74,0) 100%
  );
  flex-shrink: 0;
  z-index: 1;
}
```

---

### 3. Aura Ring

Concentric circle with diffuse glow aura. Use for circular maps, wheel diagrams, and orbital layouts. Labels are positioned around the circumference via CSS transforms.

**HTML:**
```html
<div class="aura-ring" style="--ring-size: 400px;">
  <div class="aura-ring-inner">
    <!-- Optional center content -->
    <span class="aura-ring-center-label">Center</span>
  </div>
  <!-- Labels positioned around ring: set --angle for each -->
  <span class="aura-ring-label" style="--angle: 0deg;">North</span>
  <span class="aura-ring-label" style="--angle: 90deg;">East</span>
  <span class="aura-ring-label" style="--angle: 180deg;">South</span>
  <span class="aura-ring-label" style="--angle: 270deg;">West</span>
</div>
```

**CSS:**
```css
/* ── Aura Ring ── */
.aura-ring {
  position: relative;
  width: var(--ring-size, 400px);
  height: var(--ring-size, 400px);
  border-radius: 50%;
  background:
    radial-gradient(circle at 50% 50%,
      rgba(14,14,16,0) 35%,
      rgba(184,146,74,0.06) 50%,
      rgba(184,146,74,0.18) 60%,
      rgba(184,146,74,0.08) 70%,
      rgba(14,14,16,0) 80%
    );
  border: 1.5px solid rgba(184,146,74,0.25);
  box-shadow:
    0 0 30px 6px rgba(184,146,74,0.12),
    inset 0 0 30px 6px rgba(184,146,74,0.08);
  filter: drop-shadow(0 0 18px rgba(184,146,74,0.18));
  display: flex;
  align-items: center;
  justify-content: center;
}

/* Inner circle — secondary concentric ring */
.aura-ring-inner {
  width: 55%;
  height: 55%;
  border-radius: 50%;
  border: 1px solid rgba(184,146,74,0.15);
  display: flex;
  align-items: center;
  justify-content: center;
  background: radial-gradient(circle at 50% 40%,
    rgba(184,146,74,0.07) 0%,
    rgba(14,14,16,0) 70%
  );
}

.aura-ring-center-label {
  font-family: 'Playfair Display', serif;
  font-size: 14px;
  color: var(--gold);
  text-align: center;
}

/* Label positioned on ring circumference.
   Set --angle per element. translateY moves out to ring edge. */
.aura-ring-label {
  position: absolute;
  top: 50%;
  left: 50%;
  transform:
    translate(-50%, -50%)
    rotate(var(--angle, 0deg))
    translateY(calc(var(--ring-size, 400px) * -0.46))
    rotate(calc(-1 * var(--angle, 0deg)));
  font-family: 'Inter', sans-serif;
  font-size: 11px;
  font-weight: 500;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: var(--pearl);
  white-space: nowrap;
}
```

---

### 4. Vesica

Two overlapping circles forming a vesica piscis — a sacred geometry symbol of union, intersection, and emergence. The intersection zone contains a star symbol.

**HTML:**
```html
<div class="vesica" style="--vesica-size: 160px;">
  <div class="vesica-top">
    <span class="vesica-top-label">Intuition</span>
  </div>
  <div class="vesica-intersection">
    <span class="vesica-star">✦</span>
  </div>
  <div class="vesica-bottom">
    <span class="vesica-bottom-label">Instinct</span>
  </div>
</div>
```

**CSS:**
```css
/* ── Vesica Piscis ── */
.vesica {
  position: relative;
  width: var(--vesica-size, 160px);
  height: calc(var(--vesica-size, 160px) * 1.6);
  display: flex;
  flex-direction: column;
  align-items: center;
}

/* Top circle — blue/indigo gradient */
.vesica-top {
  position: absolute;
  top: 0;
  width: var(--vesica-size, 160px);
  height: var(--vesica-size, 160px);
  border-radius: 50%;
  background: radial-gradient(circle at 45% 38%,
    rgba(130,160,255,0.75) 0%,
    rgba(74,108,247,0.55) 45%,
    rgba(30,50,160,0.3) 75%,
    rgba(14,14,16,0) 100%
  );
  border: 1px solid rgba(74,108,247,0.35);
  box-shadow: 0 0 30px 6px rgba(74,108,247,0.2);
  display: flex;
  align-items: flex-start;
  justify-content: center;
  padding-top: 14px;
}

/* Bottom circle — gold gradient */
.vesica-bottom {
  position: absolute;
  bottom: 0;
  width: var(--vesica-size, 160px);
  height: var(--vesica-size, 160px);
  border-radius: 50%;
  background: radial-gradient(circle at 55% 62%,
    rgba(237,220,160,0.75) 0%,
    rgba(184,146,74,0.55) 45%,
    rgba(100,70,10,0.3) 75%,
    rgba(14,14,16,0) 100%
  );
  border: 1px solid rgba(184,146,74,0.35);
  box-shadow: 0 0 30px 6px rgba(184,146,74,0.2);
  display: flex;
  align-items: flex-end;
  justify-content: center;
  padding-bottom: 14px;
}

/* Intersection zone — luminous center */
.vesica-intersection {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: calc(var(--vesica-size, 160px) * 0.52);
  height: calc(var(--vesica-size, 160px) * 0.52);
  border-radius: 50%;
  background: radial-gradient(circle at 50% 50%,
    rgba(237,234,228,0.25) 0%,
    rgba(184,146,74,0.15) 50%,
    rgba(184,146,74,0) 100%
  );
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2;
}

.vesica-star {
  font-size: calc(var(--vesica-size, 160px) * 0.18);
  color: var(--pearl);
  text-shadow: 0 0 12px rgba(184,146,74,0.8), 0 0 28px rgba(184,146,74,0.4);
}

.vesica-top-label,
.vesica-bottom-label {
  font-family: 'Inter', sans-serif;
  font-size: 10px;
  font-weight: 500;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: var(--pearl);
  opacity: 0.85;
  position: relative;
  z-index: 3;
}
```

---

### 5. Sacred Grid

CSS grid of dot-nodes for taxonomy layouts. Active nodes glow with gold radial-gradient. Use for gene keys sets, planetary lists, or any categorical data.

**HTML:**
```html
<div class="sacred-grid" style="--grid-cols: 4; --grid-rows: 3;">
  <div class="grid-node grid-node-active">
    <div class="grid-dot"></div>
    <div class="grid-label">Gate 1</div>
  </div>
  <div class="grid-node">
    <div class="grid-dot"></div>
    <div class="grid-label">Gate 2</div>
  </div>
  <div class="grid-node">
    <div class="grid-dot"></div>
    <div class="grid-label">Gate 3</div>
  </div>
  <!-- repeat grid-node as needed -->
</div>
```

**CSS:**
```css
/* ── Sacred Grid ── */
.sacred-grid {
  display: grid;
  grid-template-columns: repeat(var(--grid-cols, 4), 1fr);
  grid-template-rows: repeat(var(--grid-rows, 3), auto);
  gap: 20px 16px;
  padding: 8px;
}

.grid-node {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  cursor: default;
}

.grid-dot {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: radial-gradient(circle at 40% 35%,
    rgba(122,118,112,0.5) 0%,
    rgba(58,53,48,0.4) 60%,
    rgba(14,14,16,0) 100%
  );
  border: 1px solid rgba(122,118,112,0.3);
  transition: box-shadow 0.3s ease;
}

.grid-label {
  font-family: 'Inter', sans-serif;
  font-size: 9px;
  font-weight: 500;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--stone);
  text-align: center;
}

/* Active node — gold glow */
.grid-node-active .grid-dot {
  width: 30px;
  height: 30px;
  background: radial-gradient(circle at 38% 33%,
    rgba(237,222,180,0.9) 0%,
    rgba(184,146,74,0.75) 40%,
    rgba(100,70,10,0.4) 70%,
    rgba(14,14,16,0) 100%
  );
  border: 1px solid rgba(184,146,74,0.5);
  box-shadow:
    0 0 12px 3px rgba(184,146,74,0.35),
    0 0 24px 6px rgba(184,146,74,0.15);
}

.grid-node-active .grid-label {
  color: var(--gold);
}
```

---

### 6. Bioform

SVG with symmetric organic forms using bezier curves and Gaussian blur layers. Use as background decoration or soft aura behind structural elements. Note: bezier curves (the `d` path values) should be adapted per infographic context — this is a template.

**HTML:**
```html
<div class="bioform-wrapper">
  <svg class="bioform" viewBox="0 0 400 400" xmlns="http://www.w3.org/2000/svg">
    <defs>
      <filter id="bioform-blur-1">
        <feGaussianBlur stdDeviation="18"/>
      </filter>
      <filter id="bioform-blur-2">
        <feGaussianBlur stdDeviation="10"/>
      </filter>
      <filter id="bioform-blur-3">
        <feGaussianBlur stdDeviation="5"/>
      </filter>
    </defs>

    <!-- Layer 1: Soft outer petals (most diffuse) -->
    <g filter="url(#bioform-blur-1)" opacity="0.35">
      <!-- Top petal -->
      <path d="M200,200 C160,140 140,80 200,40 C260,80 240,140 200,200Z"
            fill="rgba(155,109,255,0.6)"/>
      <!-- Bottom petal (mirror) -->
      <path d="M200,200 C240,260 260,320 200,360 C140,320 160,260 200,200Z"
            fill="rgba(74,108,247,0.6)"/>
      <!-- Left petal -->
      <path d="M200,200 C140,160 80,140 40,200 C80,260 140,240 200,200Z"
            fill="rgba(184,146,74,0.4)"/>
      <!-- Right petal (mirror) -->
      <path d="M200,200 C260,240 320,260 360,200 C320,140 260,160 200,200Z"
            fill="rgba(184,146,74,0.4)"/>
    </g>

    <!-- Layer 2: Mid organic form -->
    <g filter="url(#bioform-blur-2)" opacity="0.5">
      <path d="M200,200 C170,155 155,100 200,70 C245,100 230,155 200,200Z"
            fill="rgba(185,150,255,0.7)"/>
      <path d="M200,200 C230,245 245,300 200,330 C155,300 170,245 200,200Z"
            fill="rgba(100,140,255,0.7)"/>
    </g>

    <!-- Layer 3: Sharp inner form -->
    <g filter="url(#bioform-blur-3)" opacity="0.7">
      <path d="M200,200 C182,170 176,130 200,110 C224,130 218,170 200,200Z"
            fill="rgba(237,234,228,0.6)"/>
      <path d="M200,200 C218,230 224,270 200,290 C176,270 182,230 200,200Z"
            fill="rgba(237,234,228,0.6)"/>
    </g>

    <!-- Center node -->
    <circle cx="200" cy="200" r="8" fill="rgba(237,234,228,0.9)"
            filter="url(#bioform-blur-3)"/>
    <circle cx="200" cy="200" r="3" fill="#EDEAE4"/>
  </svg>
</div>
```

**CSS:**
```css
/* ── Bioform ── */
.bioform-wrapper {
  position: relative;
  display: inline-block;
}

.bioform {
  display: block;
  overflow: visible;
}
```

---

### 7. Tech Blueprint

SVG with concentric circles, dashed axis lines, and annotation points. Use for technical or analytical infographics, system diagrams, or architectural maps.

**HTML:**
```html
<div class="blueprint-wrapper">
  <svg class="blueprint" viewBox="0 0 400 400" xmlns="http://www.w3.org/2000/svg">
    <defs>
      <style>
        .bp-circle { fill: none; stroke: rgba(74,108,247,0.25); stroke-width: 1; }
        .bp-circle-accent { fill: none; stroke: rgba(74,108,247,0.45); stroke-width: 1.5; }
        .bp-axis { stroke: rgba(74,108,247,0.2); stroke-width: 1; stroke-dasharray: 4 6; }
        .bp-axis-main { stroke: rgba(74,108,247,0.35); stroke-width: 1; stroke-dasharray: 6 4; }
        .bp-dot { fill: rgba(184,146,74,0.8); }
        .bp-dot-ring { fill: none; stroke: rgba(184,146,74,0.5); stroke-width: 1; }
        .blueprint-annotation {
          font-family: 'Inter', sans-serif;
          font-size: 10px;
          font-weight: 500;
          letter-spacing: 0.15em;
          text-transform: uppercase;
          fill: rgba(237,234,228,0.6);
        }
      </style>
    </defs>

    <!-- Concentric circles -->
    <circle cx="200" cy="200" r="160" class="bp-circle"/>
    <circle cx="200" cy="200" r="120" class="bp-circle"/>
    <circle cx="200" cy="200" r="80"  class="bp-circle-accent"/>
    <circle cx="200" cy="200" r="40"  class="bp-circle"/>
    <circle cx="200" cy="200" r="8"   class="bp-circle-accent"/>

    <!-- Axis lines -->
    <line x1="40"  y1="200" x2="360" y2="200" class="bp-axis-main"/>
    <line x1="200" y1="40"  x2="200" y2="360" class="bp-axis-main"/>
    <line x1="87"  y1="87"  x2="313" y2="313" class="bp-axis"/>
    <line x1="313" y1="87"  x2="87"  y2="313" class="bp-axis"/>

    <!-- Annotation points (adapt positions per infographic) -->
    <g transform="translate(280, 120)">
      <circle r="4" class="bp-dot"/>
      <circle r="8" class="bp-dot-ring"/>
      <text x="12" y="4" class="blueprint-annotation">Node A</text>
    </g>
    <g transform="translate(120, 280)">
      <circle r="4" class="bp-dot"/>
      <circle r="8" class="bp-dot-ring"/>
      <text x="12" y="4" class="blueprint-annotation">Node B</text>
    </g>
    <g transform="translate(320, 200)">
      <circle r="4" class="bp-dot"/>
      <circle r="8" class="bp-dot-ring"/>
      <text x="12" y="4" class="blueprint-annotation">Node C</text>
    </g>
  </svg>
</div>
```

**CSS:**
```css
/* ── Tech Blueprint ── */
.blueprint-wrapper {
  position: relative;
  display: inline-block;
}

.blueprint {
  display: block;
  overflow: visible;
}
```

---

## Content Primitives

---

### 8. Zodiac Wheel

SVG circle divided into 12 sectors. Aries starts at 270° (9 o'clock position), sectors proceed counterclockwise. Zodiac glyphs appear on the outer ring; planet markers float inside.

**HTML:**
```html
<div class="zodiac-wrapper">
  <svg class="zodiac-wheel" viewBox="0 0 360 360" xmlns="http://www.w3.org/2000/svg">
    <defs>
      <filter id="planet-glow">
        <feDropShadow dx="0" dy="0" stdDeviation="3" flood-color="rgba(184,146,74,0.7)"/>
      </filter>
      <filter id="glyph-glow">
        <feDropShadow dx="0" dy="0" stdDeviation="2" flood-color="rgba(184,146,74,0.5)"/>
      </filter>
    </defs>

    <!-- Outer background ring -->
    <circle cx="180" cy="180" r="168" fill="none"
            stroke="rgba(184,146,74,0.15)" stroke-width="28"/>

    <!-- 12 sector dividers (every 30°, starting at 270°) -->
    <!-- Each line from center to edge at angle = 270 + (n*30) degrees -->
    <!-- Aries=270°, Taurus=300°, Gemini=330°, Cancer=0°, Leo=30°, Virgo=60°,
         Libra=90°, Scorpio=120°, Sagittarius=150°, Capricorn=180°,
         Aquarius=210°, Pisces=240° -->
    <g stroke="rgba(184,146,74,0.25)" stroke-width="1">
      <line x1="180" y1="180" x2="180" y2="12"/>       <!-- 270° -->
      <line x1="180" y1="180" x2="321" y2="50"/>        <!-- 300° -->
      <line x1="180" y1="180" x2="348" y2="180"/>       <!-- 330° (approx) -->
      <line x1="180" y1="180" x2="321" y2="310"/>       <!-- 0° (Cancer) -->
      <line x1="180" y1="180" x2="180" y2="348"/>       <!-- 30° -->
      <line x1="180" y1="180" x2="39"  y2="310"/>       <!-- 60° -->
      <line x1="180" y1="180" x2="12"  y2="180"/>       <!-- 90° -->
      <line x1="180" y1="180" x2="39"  y2="50"/>        <!-- 120° -->
      <line x1="180" y1="180" x2="101" y2="13"/>        <!-- 150° -->
      <line x1="180" y1="180" x2="258" y2="13"/>        <!-- 180° -->
      <line x1="180" y1="180" x2="323" y2="101"/>       <!-- 210° -->
      <line x1="180" y1="180" x2="323" y2="258"/>       <!-- 240° -->
    </g>

    <!-- Inner ring border -->
    <circle cx="180" cy="180" r="152" fill="none"
            stroke="rgba(184,146,74,0.2)" stroke-width="1"/>
    <!-- Inner chart area border -->
    <circle cx="180" cy="180" r="124" fill="none"
            stroke="rgba(184,146,74,0.12)" stroke-width="1"/>

    <!-- Zodiac glyphs on outer ring (r≈161, counterclockwise from Aries@270°) -->
    <!-- Each glyph centered in its 30° sector: sector_start + 15° -->
    <!-- Aries: 270+15=285°, Taurus: 315°, Gemini: 345°, Cancer: 15°,
         Leo: 45°, Virgo: 75°, Libra: 105°, Scorpio: 135°,
         Sagittarius: 165°, Capricorn: 195°, Aquarius: 225°, Pisces: 255° -->
    <text class="zodiac-glyph" filter="url(#glyph-glow)"
          x="179" y="25" text-anchor="middle">♈</text>  <!-- Aries 285° -->
    <text class="zodiac-glyph" filter="url(#glyph-glow)"
          x="293" y="63" text-anchor="middle">♉</text>  <!-- Taurus 315° -->
    <text class="zodiac-glyph" filter="url(#glyph-glow)"
          x="340" y="172" text-anchor="middle">♊</text> <!-- Gemini 345° -->
    <text class="zodiac-glyph" filter="url(#glyph-glow)"
          x="290" y="296" text-anchor="middle">♋</text> <!-- Cancer 15° -->
    <text class="zodiac-glyph" filter="url(#glyph-glow)"
          x="179" y="340" text-anchor="middle">♌</text> <!-- Leo 45° -->
    <text class="zodiac-glyph" filter="url(#glyph-glow)"
          x="66"  y="296" text-anchor="middle">♍</text> <!-- Virgo 75° -->
    <text class="zodiac-glyph" filter="url(#glyph-glow)"
          x="20"  y="172" text-anchor="middle">♎</text> <!-- Libra 105° -->
    <text class="zodiac-glyph" filter="url(#glyph-glow)"
          x="66"  y="63"  text-anchor="middle">♏</text> <!-- Scorpio 135° -->
    <text class="zodiac-glyph" filter="url(#glyph-glow)"
          x="120" y="22"  text-anchor="middle">♐</text> <!-- Sagittarius 165° -->
    <text class="zodiac-glyph" filter="url(#glyph-glow)"
          x="238" y="22"  text-anchor="middle">♑</text> <!-- Capricorn 195° -->
    <text class="zodiac-glyph" filter="url(#glyph-glow)"
          x="331" y="112" text-anchor="middle">♒</text> <!-- Aquarius 225° -->
    <text class="zodiac-glyph" filter="url(#glyph-glow)"
          x="331" y="242" text-anchor="middle">♓</text> <!-- Pisces 255° -->

    <!-- Planet markers (adapt x/y per chart; use class planet-marker) -->
    <g class="planet-marker" filter="url(#planet-glow)" transform="translate(210,90)">
      <circle r="7" fill="rgba(184,146,74,0.9)"/>
      <text font-family="'Inter',sans-serif" font-size="8" fill="#0E0E10"
            text-anchor="middle" dy="3">☉</text>
    </g>
    <g class="planet-marker" filter="url(#planet-glow)" transform="translate(140,265)">
      <circle r="7" fill="rgba(155,109,255,0.85)"/>
      <text font-family="'Inter',sans-serif" font-size="8" fill="#EDEAE4"
            text-anchor="middle" dy="3">☽</text>
    </g>
  </svg>
</div>
```

**CSS:**
```css
/* ── Zodiac Wheel ── */
.zodiac-wrapper {
  position: relative;
  display: inline-block;
}

.zodiac-wheel {
  display: block;
  overflow: visible;
}

.zodiac-glyph {
  font-family: 'Inter', sans-serif;
  font-size: 16px;
  fill: var(--gold);
}

.planet-marker {
  cursor: default;
}
```

---

### 9. HD Bodygraph Mini

SVG viewBox 200×340 with 9 circles representing Human Design centers in correct topological positions. Defined centers are filled orbs with color + drop-shadow; open centers are transparent with thin border. Channels connect centers as lines.

**Center coordinates:**
- Head: (100, 30), Ajna: (100, 75), Throat: (100, 120)
- G/Self: (100, 165), Heart/Ego: (145, 155)
- Spleen: (55, 210), Sacral: (100, 235)
- Solar Plexus: (145, 220), Root: (100, 300)

**HTML:**
```html
<div class="bodygraph-wrapper">
  <svg class="bodygraph-mini" viewBox="0 0 200 340" xmlns="http://www.w3.org/2000/svg">
    <defs>
      <filter id="center-glow">
        <feDropShadow dx="0" dy="0" stdDeviation="4" flood-color="rgba(184,146,74,0.6)"/>
      </filter>
      <filter id="center-glow-blue">
        <feDropShadow dx="0" dy="0" stdDeviation="4" flood-color="rgba(74,108,247,0.6)"/>
      </filter>
      <filter id="center-glow-violet">
        <feDropShadow dx="0" dy="0" stdDeviation="4" flood-color="rgba(155,109,255,0.6)"/>
      </filter>
    </defs>

    <!-- ── Channels (lines between centers — adapt per chart) ── -->
    <!-- Head → Ajna -->
    <line x1="100" y1="30"  x2="100" y2="75"  class="hd-channel"/>
    <!-- Ajna → Throat -->
    <line x1="100" y1="75"  x2="100" y2="120" class="hd-channel"/>
    <!-- Throat → G -->
    <line x1="100" y1="120" x2="100" y2="165" class="hd-channel"/>
    <!-- G → Heart -->
    <line x1="100" y1="165" x2="145" y2="155" class="hd-channel"/>
    <!-- Throat → Spleen (adapt as needed) -->
    <line x1="100" y1="120" x2="55"  y2="210" class="hd-channel hd-channel-defined"/>
    <!-- G → Sacral -->
    <line x1="100" y1="165" x2="100" y2="235" class="hd-channel"/>
    <!-- Sacral → Root -->
    <line x1="100" y1="235" x2="100" y2="300" class="hd-channel"/>
    <!-- Sacral → Solar Plexus -->
    <line x1="100" y1="235" x2="145" y2="220" class="hd-channel"/>

    <!-- ── Centers ── -->
    <!-- DEFINED center (example: Head) — filled orb -->
    <circle cx="100" cy="30" r="18" class="hd-center hd-center-defined-violet"
            filter="url(#center-glow-violet)"/>
    <text x="100" y="34" class="hd-center-label">HD</text>

    <!-- OPEN center (example: Ajna) — transparent with border -->
    <circle cx="100" cy="75" r="18" class="hd-center hd-center-open"/>
    <text x="100" y="79" class="hd-center-label hd-center-label-open">AJ</text>

    <!-- DEFINED center (example: Throat) — gold -->
    <circle cx="100" cy="120" r="20" class="hd-center hd-center-defined-gold"
            filter="url(#center-glow)"/>
    <text x="100" y="124" class="hd-center-label">TH</text>

    <!-- DEFINED center (example: G/Self) — blue -->
    <circle cx="100" cy="165" r="20" class="hd-center hd-center-defined-blue"
            filter="url(#center-glow-blue)"/>
    <text x="100" y="169" class="hd-center-label">G</text>

    <!-- OPEN center (example: Heart) -->
    <circle cx="145" cy="155" r="16" class="hd-center hd-center-open"/>
    <text x="145" y="159" class="hd-center-label hd-center-label-open">HT</text>

    <!-- OPEN center (example: Spleen) -->
    <circle cx="55"  cy="210" r="16" class="hd-center hd-center-open"/>
    <text x="55"  y="214" class="hd-center-label hd-center-label-open">SP</text>

    <!-- DEFINED center (example: Sacral) — gold -->
    <circle cx="100" cy="235" r="20" class="hd-center hd-center-defined-gold"
            filter="url(#center-glow)"/>
    <text x="100" y="239" class="hd-center-label">SC</text>

    <!-- OPEN center (example: Solar Plexus) -->
    <circle cx="145" cy="220" r="16" class="hd-center hd-center-open"/>
    <text x="145" y="224" class="hd-center-label hd-center-label-open">SX</text>

    <!-- DEFINED center (example: Root) — emerald -->
    <circle cx="100" cy="300" r="18" class="hd-center hd-center-defined-emerald"
            filter="url(#center-glow)"/>
    <text x="100" y="304" class="hd-center-label">RT</text>
  </svg>
</div>
```

**CSS:**
```css
/* ── HD Bodygraph Mini ── */
.bodygraph-wrapper {
  position: relative;
  display: inline-block;
}

.bodygraph-mini {
  display: block;
  overflow: visible;
}

/* Channels */
.hd-channel {
  stroke: rgba(122,118,112,0.3);
  stroke-width: 2;
  fill: none;
}
.hd-channel-defined {
  stroke: rgba(184,146,74,0.65);
  stroke-width: 3;
}

/* Center — base */
.hd-center {
  fill: none;
  stroke: none;
}

/* Open center */
.hd-center-open {
  fill: rgba(14,14,16,0.0);
  stroke: rgba(122,118,112,0.4);
  stroke-width: 1.5;
}

/* Defined center variants */
.hd-center-defined-gold {
  fill: url(#hd-grad-gold);
  /* Inline fallback */
  fill: radial-gradient(circle at 38% 35%,
    rgba(237,222,180,0.95) 0%,
    rgba(184,146,74,0.8) 40%,
    rgba(100,70,10,0.5) 75%,
    rgba(14,14,16,0) 100%
  );
  fill: rgba(184,146,74,0.75); /* CSS fallback for SVG */
}
.hd-center-defined-blue {
  fill: rgba(74,108,247,0.75);
}
.hd-center-defined-violet {
  fill: rgba(155,109,255,0.75);
}
.hd-center-defined-emerald {
  fill: rgba(0,158,116,0.75);
}

/* Center labels */
.hd-center-label {
  font-family: 'Inter', sans-serif;
  font-size: 8px;
  font-weight: 600;
  letter-spacing: 0.05em;
  fill: var(--pearl);
  text-anchor: middle;
}
.hd-center-label-open {
  fill: var(--stone);
}
```

---

### 10. GK Spectrum Bar

Horizontal gradient bar with three named zones: Shadow (dark red), Gift (warm gold), Siddhi (gold to pearl). Use for Gene Keys codon readings, shadow-to-gift spectrums, or any tri-zone gradient scale.

**HTML:**
```html
<div class="gk-spectrum">
  <div class="gk-bar"></div>
  <div class="gk-zones">
    <div class="gk-zone">
      <span class="gk-zone-label">Shadow</span>
      <span class="gk-zone-name playfair">Reticence</span>
    </div>
    <div class="gk-zone">
      <span class="gk-zone-label">Gift</span>
      <span class="gk-zone-name playfair">Expression</span>
    </div>
    <div class="gk-zone">
      <span class="gk-zone-label">Siddhi</span>
      <span class="gk-zone-name playfair">Word</span>
    </div>
  </div>
</div>
```

**CSS:**
```css
/* ── GK Spectrum Bar ── */
.gk-spectrum {
  display: flex;
  flex-direction: column;
  gap: 8px;
  width: 100%;
  max-width: 480px;
}

.gk-bar {
  height: 14px;
  border-radius: 30px;
  background: linear-gradient(
    to right,
    #3a1c1c 0%,
    #5a2a2a 28%,
    #5a4a20 45%,
    #B8924A 62%,
    #B8924A 78%,
    #EDEAE4 100%
  );
  box-shadow:
    0 0 16px 2px rgba(184,146,74,0.2),
    inset 0 1px 0 rgba(255,255,255,0.08);
}

.gk-zones {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  gap: 4px;
}

.gk-zone {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
}

.gk-zone-label {
  font-family: 'Inter', sans-serif;
  font-size: 9px;
  font-weight: 600;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  color: var(--stone);
}

/* First zone = Shadow */
.gk-zone:first-child .gk-zone-label { color: rgba(180,100,100,0.7); }
/* Last zone = Siddhi */
.gk-zone:last-child .gk-zone-label  { color: var(--pearl); opacity: 0.8; }
/* Middle zone = Gift */
.gk-zone:nth-child(2) .gk-zone-label { color: var(--gold); }

.gk-zone-name {
  font-family: 'Playfair Display', serif;
  font-size: 13px;
  font-style: italic;
  color: var(--pearl);
  text-align: center;
}
```

---

### 11. Data Label

Minimal typographic tag. Use to annotate charts, diagrams, or sections with category markers, archetype codes, or gate numbers.

**Variants:** `.data-label-gold`, `.data-label-emerald`

**HTML:**
```html
<!-- Gold variant -->
<span class="data-label data-label-gold">Gate 1</span>

<!-- Emerald variant -->
<span class="data-label data-label-emerald">Defined</span>
```

**CSS:**
```css
/* ── Data Label ── */
.data-label {
  display: inline-block;
  font-family: 'Inter', sans-serif;
  font-size: 11px;
  font-weight: 500;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  padding: 4px 12px;
  border-radius: 2px;
  border: 1px solid transparent;
  white-space: nowrap;
}

/* Gold variant */
.data-label-gold {
  color: var(--gold);
  border-color: rgba(184,146,74,0.3);
  background: rgba(184,146,74,0.06);
}

/* Emerald variant */
.data-label-emerald {
  color: var(--emerald);
  border-color: rgba(0,158,116,0.3);
  background: rgba(0,158,116,0.06);
}
```

---

### 12. Pull Quote

Large, centered italic quote in Playfair Display. Use for channel descriptions, archetype mottos, or thematic statements that anchor the infographic.

**HTML:**
```html
<blockquote class="pull-quote">
  "The gift of expression speaks the truth that silence cannot hold."
</blockquote>
```

**CSS:**
```css
/* ── Pull Quote ── */
.pull-quote {
  font-family: 'Playfair Display', serif;
  font-weight: 400;
  font-style: italic;
  font-size: clamp(20px, 2.5vw, 28px);
  color: var(--gold);
  text-align: center;
  max-width: 80%;
  margin: 0 auto;
  padding: 24px 0;
  line-height: 1.55;
  border: none;
  quotes: none;
}

.pull-quote::before,
.pull-quote::after {
  content: none;
}
```

---

## Composition Rules

1. **Max 3 structural primitives** per infographic (Orb, Totem, Aura Ring, Vesica, Sacred Grid, Bioform, Blueprint — pick at most 3).
2. **Hierarchy by size and luminosity** — the most important element should be largest and brightest.
3. **Central axis** — always establish a vertical or radial center that the composition anchors around.
4. **Breathing space** — 48px minimum between structural primitives; 64px minimum from infographic edges.
5. **Layer order (z-index discipline):**
   - Z-0: Background (`--obsidian` fill or pearl background)
   - Z-1: Bioform / decorative SVG elements
   - Z-2: Structural primitives (Orb, Totem, Aura Ring, etc.)
   - Z-3: Content primitives (Zodiac Wheel, Bodygraph, Spectrum Bar)
   - Z-4: Text, labels, data labels, pull quotes
6. **Grimoria signature** — always place at the bottom of every infographic:

```html
<div class="grimoria-signature">✦ GRIMORIA</div>
```

```css
.grimoria-signature {
  font-family: 'Inter', sans-serif;
  font-size: 9px;
  font-weight: 500;
  letter-spacing: 0.3em;
  text-transform: uppercase;
  color: var(--stone);
  text-align: center;
  padding-top: 16px;
}
```
