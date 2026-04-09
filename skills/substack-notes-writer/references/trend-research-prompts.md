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
