# Semantic Rules — Cosmos Infographic

This file defines HOW to position esoteric data correctly in infographics.
It does NOT explain meanings — for those, see `skills/astro-hd-gk-profiler/references/`.

---

## 1. Zodiac Wheel — Positioning Signs and Planets

### Sign Order Table (counterclockwise from 9 o'clock)

| Degrees | Sign | Glyph | SVG Angle |
|---------|------|-------|-----------|
| 0–30 | Aries | ♈ | 270° (9 o'clock) |
| 30–60 | Taurus | ♉ | 240° |
| 60–90 | Gemini | ♊ | 210° |
| 90–120 | Cancer | ♋ | 180° (6 o'clock) |
| 120–150 | Leo | ♌ | 150° |
| 150–180 | Virgo | ♍ | 120° |
| 180–210 | Libra | ♎ | 90° (3 o'clock) |
| 210–240 | Scorpio | ♏ | 60° |
| 240–270 | Sagittarius | ♐ | 30° |
| 270–300 | Capricorn | ♑ | 0° (12 o'clock) |
| 300–330 | Aquarius | ♒ | 330° |
| 330–360 | Pisces | ♓ | 300° |

> Direction: counterclockwise (standard western astrology convention on wheel).
> SVG angles are measured from the positive X-axis; adjust with `atan2` as needed.

### Planet Position Formula

```
angle_sign_start = SVG angle for the sign's 0° cusp (from table above)
angle_planet = angle_sign_start - (degree_in_sign / 30 * 30)

SVG coordinates:
  x = cx + r * cos(angle_planet * π / 180)
  y = cy + r * sin(angle_planet * π / 180)
```

Where `cx`, `cy` = center of wheel; `r` = orbit radius for planet layer.

### Planet Symbols Table

| Planet | Glyph | Color | Hex |
|--------|-------|-------|-----|
| Sun | ☉ | Gold | #B8924A |
| Moon | ☽ | Pearl / Silver | #D8D4C8 |
| Mercury | ☿ | Emerald | #2E7D5E |
| Venus | ♀ | Cosmos Violet | #7B5EA7 |
| Mars | ♂ | Deep Crimson | #8B2500 |
| Jupiter | ♃ | Cosmos Blue | #4A6CF7 |
| Saturn | ♄ | Charcoal | #4A4A4A |
| Uranus | ♅ | Cosmos Blue | #4A6CF7 |
| Neptune | ♆ | Cosmos Violet | #7B5EA7 |
| Pluto | ♇ | Obsidian + Gold glow | #1A1A1A (glow: #B8924A) |

### Aspects Table

| Aspect | Angle | Color | Line Style |
|--------|-------|-------|------------|
| Conjunction | 0° | Gold | solid 2px |
| Sextile | 60° | Emerald | dashed |
| Square | 90° | Deep Crimson | solid 1px |
| Trine | 120° | Cosmos Blue | solid 1.5px |
| Opposition | 180° | Cosmos Violet | solid 1px |

Aspect lines are drawn as chords inside the wheel, connecting the two planetary positions.

---

## 2. Human Design Bodygraph — Center Topology

### 9 Centers Position Table (SVG viewBox 200×340)

| Center | cx | cy | r | Color | Hex |
|--------|----|----|---|-------|-----|
| Head (Crown) | 100 | 30 | 18 | Gold | #B8924A |
| Ajna (Mind) | 100 | 75 | 18 | Cosmos Blue | #4A6CF7 |
| Throat | 100 | 120 | 18 | Brown / Ochre | #8B6914 |
| G Center (Self) | 100 | 165 | 18 | Gold | #B8924A |
| Heart / Ego | 145 | 155 | 14 | Red | #C0392B |
| Spleen | 55 | 210 | 16 | Brown | #8B6914 |
| Sacral | 100 | 235 | 18 | Red | #C0392B |
| Solar Plexus | 145 | 220 | 16 | Brown / Ochre | #8B6914 |
| Root | 100 | 300 | 18 | Red | #C0392B |

### Main Channels Table

| Channel | From Center | To Center | Gate Pair |
|---------|-------------|-----------|-----------|
| Channel 1–8 | G Center | Throat | Gate 1 ↔ Gate 8 |
| Channel 2–14 | G Center | Sacral | Gate 2 ↔ Gate 14 |
| Channel 3–60 | Sacral | Root | Gate 3 ↔ Gate 60 |
| Channel 5–15 | Sacral | G Center | Gate 5 ↔ Gate 15 |
| Channel 9–52 | Sacral | Root | Gate 9 ↔ Gate 52 |
| Channel 11–56 | Ajna | Throat | Gate 11 ↔ Gate 56 |
| Channel 17–62 | Ajna | Throat | Gate 17 ↔ Gate 62 |
| Channel 20–57 | Throat | Spleen | Gate 20 ↔ Gate 57 |
| Channel 26–44 | Heart / Ego | Spleen | Gate 26 ↔ Gate 44 |
| Channel 36–35 | Solar Plexus | Throat | Gate 36 ↔ Gate 35 |

> Gate→Center full mapping: consult `skills/astro-hd-gk-profiler/references/human-design-framework.md`

### Visualization Rules

- **Defined center** = filled circle, center color at opacity 0.7, drop-shadow glow effect
- **Open center** = transparent circle, thin border `rgba(237, 234, 228, 0.3)`
- **Active channel** = straight line between the two center coordinates, stroke `rgba(237, 234, 228, 0.4)`, stroke-width 2

---

## 3. Gene Keys — Spectrum and Sequences

### Triplet Format

```
GK [number]: [Shadow] → [Gift] → [Siddhi]
```

Example: GK 28: Purposelessness → Totality → Immortality

> For all correct triplet names, ALWAYS consult `skills/astro-hd-gk-profiler/references/gene-keys-framework.md`.
> Never guess or infer triplet names from context.

### Sequences Table

| Sequence | Spheres | Primary Use |
|----------|---------|-------------|
| Activation | Life's Work, Evolution, Radiance, Purpose | Business profiles |
| Venus | Attraction, IQ, EQ, SQ | Relationships, communication |
| Pearl | Vocation, Culture, Brand, Pearl | Prosperity, brand |

### GK Spectrum Bar Visualization

The spectrum bar runs left (Shadow) → center (Gift) → right (Siddhi):

| Zone | Label | Gradient Range |
|------|-------|----------------|
| Left | Shadow | dark red-brown `#3a1c1c` → `#5a2a2a` |
| Center | Gift | warm gold `#5a4a20` → `#B8924A` |
| Right | Siddhi | gold to pearl `#B8924A` → `#EDEAE4` |

Implementation: SVG `linearGradient` with three color stops at 0%, 50%, 100%.

---

## 4. Golden Rule — Data Verification Before Rendering

If the user provides specific data (planetary positions, HD gates, Gene Keys), the skill **VERIFIES** them against the reference frameworks BEFORE positioning.

### Verification Steps

1. Check planet/sign position against zodiac wheel table (Section 1)
2. Check gate→center assignment against `human-design-framework.md`
3. Check GK triplet names against `gene-keys-framework.md`

### On Data Mismatch

Example of invalid data: "Gate 44 in Sacral" — incorrect, Gate 44 belongs to the Spleen.

When a mismatch is detected, the skill:

1. Does **NOT** proceed silently
2. **ASKS** the user for confirmation, specifying what was flagged and why
3. Only generates the infographic **after** confirmation

> This rule exists to protect the integrity of the reading. Grimoria infographics are semantically precise — not just visually beautiful.
