# Elderan Calculators — CLAUDE.md

Fan-made single-page web app for the MMORPG **Elderan Online**.
Hosted on GitHub Pages. No build step, no framework, no dependencies beyond two Google Fonts loaded via CDN.

---

## File Structure

```
index.html   — the entire app (HTML + CSS + JS, ~1173 lines)
CLAUDE.md    — this file
```

Everything lives in `index.html`. CSS is in a `<style>` block in `<head>`. JavaScript is in a `<script>` block at the end of `<body>`.

---

## Architecture

### Routing
Hash-based SPA routing. Pages are `<section class="page">` elements toggled with `.active`. Navigation updates `location.hash` via `history.replaceState` (wrapped in try/catch for sandboxed environments).

```
#home   → page-home
#rune   → page-rune
#mana   → page-mana
```

`showPage(name)` handles all routing. The site title and nav tabs call it. Home cards navigate via `data-goto` attribute.

### Persistence
`localStorage` is used to persist user edits to data tables.
- Rune calculator: key `elderan-rune-calc-v2`
- Mana calculator: key `elderan-mana-calc-v1`

On load, saved data is merged with defaults so new default entries are always picked up. Reset buttons call `localStorage.removeItem` and reload from defaults.

---

## Calculators

### 1. Rune Table Calculator (`#rune`)

**Purpose:** Estimate how many runes a rune table produces in 8h or 24h, and the total gold cost.

**Inputs:**
| Input | Description |
|---|---|
| Rune | Dropdown filtered by selected vocation |
| Duration | 8h or 24h toggle |
| Vocation | Knight / Paladin / Druid / Sorcerer |
| Promotion | None / Promotion I / Promotion II (replaces base regen, not additive) |
| Starting Spirit | Spirit already accumulated before the table starts |
| Magic Carpet | +1 spirit regen/sec toggle |
| Rune Stone | 5% cheaper spirit cost toggle |

**Formula:**
```
regen      = vocation_regen_at_promo_tier + (magicCarpet ? 1 : 0)
spiritCost = rune.spirit * (runeStone ? 0.95 : 1)
total      = startingSpirit + regen * duration_seconds
runesProduced = floor(total / spiritCost)
charges    = runesProduced * rune.charges
blanksCost = runesProduced * rune.blankCost
totalGold  = tableCost[duration] + blanksCost
costPerCharge = ceil(totalGold / charges)
```

**Outputs:** Blank Runes Required (or "Spell Casts" if blankCost === 0), Charges Obtained, gold breakdown, cost per charge.

**Rune Data:**
| Rune | Spirit | Charges | Blank Cost (gp) | Vocations |
|---|---|---|---|---|
| Hmm | 70 | 5 | 15 | All |
| Ih | 60 | 1 | 10 | Knight, Paladin, Druid |
| Uh | 90 | 1 | 20 | Knight, Paladin, Druid |
| Freeze Missile | 120 | 5 | 20 | Druid, Sorcerer |
| GFB / Ava | 110 | 2 | 20 | Druid, Sorcerer |
| Explosion | 180 | 3 | 30 | All |
| SD | 180 | 1 | 30 | Sorcerer |
| Ice SD | 180 | 1 | 30 | Druid, Sorcerer |
| Twin SD | 280 | 1 | 30 | Sorcerer |
| Elven Arrow | 60 | 15 | 0 | Paladin |
| Golden Arrow | 90 | 13 | 0 | Paladin |
| Titan Bolt | 110 | 15 | 0 | Paladin |
| Exp Arrow | 325 | 5 | 0 | Paladin |

**Vocation Spirit Regen (per second):**
| Vocation | Base | Promotion I | Promotion II |
|---|---|---|---|
| Knight | 0.2 | Elite Knight 0.4 | Noble Knight 0.5 |
| Paladin | 0.33 | Royal Paladin 0.66 | Runic Paladin 0.75 |
| Druid | 0.4 | Elder Druid 1.0 | Ancient Druid 1.25 |
| Sorcerer | 0.4 | Master Sorcerer 1.0 | Mystical Sorcerer 1.25 |

**Table Rental Costs:** 8h = 45,000 gp · 24h = 125,000 gp

---

### 2. Mana Cost Calculator (`#mana`)

**Purpose:** Calculate the gold cost per spell cast and per target hit, based on a chosen mana regen method.

**Inputs:**
| Input | Description |
|---|---|
| Vocation | Filters spell list |
| Mana Regen Method | Dropdown of 11 options |
| Spell | Filtered by vocation; shows mana cost and AoE/ST tag |
| Avg Targets Hit | Only shown for AoE spells; defaults to 1 |

**Formula:**
```
costPerCast   = spell.mana * regenMethod.gpPerMana
costPerTarget = ceil(costPerCast / targets)   // targets=1 for single-target
```

**Outputs:** Gold per Cast, Gold per Target (hidden for single-target spells since it equals cost per cast).

**Mana Regen Methods:**
| Method | Avg Mana | Price/Cast (gp) | gp/Mana |
|---|---|---|---|
| Small Mana | 100 | 65 | 0.65 ⭐ |
| Big Mana | 150 | 135 | 0.90 |
| Small Keg | 200 | 203 | 1.02 |
| Big Keg | 200 | 206.6 | 1.03 |
| Enchanted Mana Pot (24h) | 1300 | 1275 | 0.98 |
| Big Enchanted Mana Pot (24h) | 2000 | 1975 | 0.99 |
| Enchanted Mana Pot (8h) | 1300 | 1400 | 1.08 |
| Big Enchanted Mana Pot (8h) | 2000 | 2100 | 1.05 |
| Velvet Boots | 14400 | 10000 | 0.69 |
| Life Ring (npc price) | 1600 | 900 | 0.56 ⭐ |
| Ring of Healing (npc price) | 1800 | 2000 | 1.11 |

⭐ = best value options (highlighted in UI)

**Spells by Vocation:**

*Paladin:* Holy Grenade (25, ST), Throwing Knife (30, ST), Holy Rain (150, AoE), Hail of Arrows (165, AoE), Bullseye (225, ST), Divine Healing (190, AoE)

*Druid:* Elemental Strikes (20, ST), Spore Flick (30, AoE), Poison Wave (220, AoE), Poison Lash (105, AoE), Ice Wave (340, AoE), Poison Storm (950, AoE), Greater Poison Lash (215, AoE), Frost Storm (1050, AoE), Redemption (100, ST), Enhanced Heal Friend (130, ST), Heal Friend (80, ST)

*Knight:* Blunt Strike (50, ST), Ground Stomp (140, AoE), Frontal Strike (250, AoE), Strong Berserk (350, AoE), Executioner (320, ST), Physical Shockwave (650, AoE)

*Sorcerer:* Elemental Strikes (20, ST), Cinder Snap (30, AoE), Magic Bullet (30, ST), Energy Beam (100, AoE), Fire Wave (250, AoE), Great Energy Beam (200, AoE), Hell Chain (85, AoE), Tempest Chain (110, AoE), Energy Wave (350, AoE), Thunder Outburst (705, AoE), Ultimate Explosion (850, AoE), Greater Hell Chain (160, AoE), Death Wave (400, AoE), Greater Tempest Chain (225, AoE), Armagedon (1080, AoE), Chain of the Unhollowed (245, AoE)

---

## UI Patterns

- **Cards** (`.card`) have a double-border effect via `::before` pseudo-element.
- **Button rows** (`.btn-row`) use `.active` class for selected state — gold highlight.
- **Toggles** (`.toggle`) are `<label>` wrappers with a hidden `<input type="checkbox">` and a custom `.box` element animated with `::after`.
- **Setup sections** are collapsible via `.setup-content.open` toggled by a button. State is local to the DOM, not persisted.
- **Stat boxes** (`.stat-box`) use a 2-column grid inside `.result-stats`. The second box is hidden for single-target spells in the mana calculator.

---

## Design Tokens

| Token | Value | Usage |
|---|---|---|
| `--bg` | `#0d0a06` | Page background |
| `--bg-card` | `#181109` | Card backgrounds |
| `--bg-elevated` | `#241a0f` | Button rows, toggles |
| `--bg-input` | `#0f0a04` | Inputs, setup rows |
| `--gold` | `#c9a876` | Labels, accents |
| `--gold-bright` | `#f0d496` | Results, active states, title |
| `--gold-dim` | `#8a7556` | Borders on active elements |
| `--burgundy` | `#722929` | Reset button border |
| `--text` | `#e8dcc4` | Body text |
| `--text-muted` | `#968670` | Descriptions, metadata |
| `--border` | `#3d2f1f` | Default borders |
| `--border-bright` | `#6b4f30` | Hover borders |

**Fonts:** Cinzel (headings, labels, numbers) · Crimson Pro (body, inputs)

---

## Adding a New Calculator

1. Add a new `<button data-page="mypage">` in `<nav class="tabs">`.
2. Add a `<section class="page" id="page-mypage">` with the calculator UI.
3. Add a card in `#page-home .calc-grid` with `data-goto="mypage"`.
4. Add `"mypage"` to the `PAGES` array in the routing JS.
5. Add a `DEFAULT_*_DATA` object, a storage key, and `render*All()` / `calculate*()` functions following the same pattern as the existing calculators.
6. Call `render*All()` at the bottom of the script alongside `renderAll()` and `renderManaAll()`.

---

*Made by Langak · Fan-made tool for Elderan Online*
