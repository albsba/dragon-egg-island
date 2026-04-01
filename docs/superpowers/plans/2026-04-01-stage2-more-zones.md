# Stage 2: More Zones & Dragons Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add 5 new zones (Enchanted Forest, Crystal Cave, Snowy Mountain, Volcano Ridge, Rainbow Falls), 4 new dragon types (Shadow, Star, Ice, Rainbow), progressive zone unlocking, and zone-specific particle effects -- expanding the game from 2 zones to 7.

**Architecture:** Extend the existing single `index.html` file. Add new sprite data, zone maps, particle types, and update the unlock/progression system. All patterns already exist in the code -- this is additive work following established conventions.

**Tech Stack:** Same as Stage 1 -- vanilla HTML5 Canvas, JavaScript, no dependencies.

---

## File Structure

```
harper-audrey-game/
├── index.html          # Only file modified -- add zones, sprites, particles, unlock logic
```

---

### Task 1: Add New Dragon & Egg Sprites

**Files:**
- Modify: `index.html` (SPRITE DATA section -- EGG_SPRITES and DRAGON_SPRITES objects)

Add 4 new egg types and 4 new baby dragon types using the existing `makeEggSprite` and `makeDragonSprite` factory functions.

- [ ] **Step 1: Add new egg sprites**

Add to the `EGG_SPRITES` object:

```javascript
const EGG_SPRITES = {
  fire:   makeEggSprite('#FF6B35', '#FF4500', '#FFD700'),
  nature: makeEggSprite('#66BB6A', '#43A047', '#A5D6A7'),
  love:   makeEggSprite('#FF69B4', '#E91E63', '#FFB6C1'),
  shadow: makeEggSprite('#7E57C2', '#5E35B1', '#B39DDB'),
  star:   makeEggSprite('#FFD700', '#FFC107', '#FFF9C4'),
  ice:    makeEggSprite('#81D4FA', '#4FC3F7', '#E0F7FA'),
  rainbow: makeEggSprite('#FF6B6B', '#4ECDC4', '#FFE66D'),
};
```

- [ ] **Step 2: Add new dragon sprites**

Add to the `DRAGON_SPRITES` object:

```javascript
const DRAGON_SPRITES = {
  fire:    makeDragonSprite('#FF4500', '#FFD700', '#333'),
  nature:  makeDragonSprite('#43A047', '#A5D6A7', '#333'),
  love:    makeDragonSprite('#E91E63', '#FFB6C1', '#333'),
  shadow:  makeDragonSprite('#5E35B1', '#B39DDB', '#EDE7F6'),
  star:    makeDragonSprite('#FFC107', '#FFF9C4', '#333'),
  ice:     makeDragonSprite('#4FC3F7', '#E0F7FA', '#333'),
  rainbow: makeDragonSprite('#FF6B6B', '#FFE66D', '#333'),
};
```

- [ ] **Step 3: Verify sprites render**

Temporarily draw the new sprites in the game loop to visually check they look right. Then remove test code.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add shadow, star, ice, and rainbow egg and dragon sprites"
```

---

### Task 2: Add Enchanted Forest Zone

**Files:**
- Modify: `index.html` (CONSTANTS section -- add zone map, add to ZONES object)

The Enchanted Forest has tall dark trees, glowing mushrooms, and hidden paths. Egg types: Nature, Shadow.

- [ ] **Step 1: Add new tile types for Enchanted Forest**

Add to `TILE_COLORS`:

```javascript
dark_tree_top:  '#1a5c14',
dark_tree_trunk:'#5D4037',
mushroom:       '#E040FB',
moss:           '#558B2F',
```

Add `'dark_tree_top'`, `'dark_tree_trunk'`, `'mushroom'` to `SOLID_TILES` array.

- [ ] **Step 2: Create ZONE_ENCHANTED_FOREST map**

Create a 16-row x 25-col map. Design: dark trees scattered around edges and interior creating winding paths, glowing mushrooms (small solid obstacles), moss patches for ground variety, grass as base tile. Leave open grass areas for eggs to spawn.

```javascript
const ZONE_ENCHANTED_FOREST = [
  ['dark_tree_top','dark_tree_top','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','dark_tree_top','dark_tree_top'],
  ['dark_tree_trunk','dark_tree_trunk','grass','moss','grass','grass','dark_tree_top','dark_tree_top','grass','grass','grass','grass','grass','grass','grass','grass','grass','dark_tree_top','dark_tree_top','grass','grass','moss','grass','dark_tree_trunk','dark_tree_trunk'],
  ['grass','grass','grass','grass','grass','grass','dark_tree_trunk','dark_tree_trunk','grass','grass','moss','grass','grass','grass','moss','grass','grass','dark_tree_trunk','dark_tree_trunk','grass','grass','grass','grass','grass','grass'],
  ['grass','moss','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','mushroom','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','moss','grass'],
  ['grass','grass','grass','dark_tree_top','dark_tree_top','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','dark_tree_top','dark_tree_top','grass','grass','grass','grass'],
  ['grass','grass','grass','dark_tree_trunk','dark_tree_trunk','grass','grass','moss','grass','grass','grass','grass','grass','grass','grass','grass','moss','grass','grass','dark_tree_trunk','dark_tree_trunk','grass','grass','grass','grass'],
  ['grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass'],
  ['grass','mushroom','grass','grass','grass','grass','grass','grass','grass','grass','moss','grass','grass','grass','moss','grass','grass','grass','grass','grass','grass','grass','mushroom','grass','grass'],
  ['grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass'],
  ['grass','grass','grass','grass','grass','grass','dark_tree_top','dark_tree_top','grass','grass','grass','grass','grass','grass','grass','grass','grass','dark_tree_top','dark_tree_top','grass','grass','grass','grass','grass','grass'],
  ['grass','moss','grass','grass','grass','grass','dark_tree_trunk','dark_tree_trunk','grass','grass','grass','mushroom','grass','grass','grass','grass','grass','dark_tree_trunk','dark_tree_trunk','grass','grass','grass','moss','grass','grass'],
  ['grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass'],
  ['grass','grass','mushroom','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','mushroom','grass','grass','grass'],
  ['grass','grass','grass','grass','moss','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','grass','moss','grass','grass','grass','grass','grass'],
  ['dark_tree_top','dark_tree_top','grass','grass','grass','grass','grass','grass','path','path','path','path','path','path','path','path','path','grass','grass','grass','grass','grass','grass','dark_tree_top','dark_tree_top'],
  ['dark_tree_trunk','dark_tree_trunk','grass','grass','grass','grass','grass','grass','path','path','path','path','path','path','path','path','path','grass','grass','grass','grass','grass','grass','dark_tree_trunk','dark_tree_trunk'],
];
```

- [ ] **Step 3: Add to ZONES object and update drawTile**

```javascript
ZONES.enchanted = { map: ZONE_ENCHANTED_FOREST, name: 'Enchanted Forest', eggs: ['nature', 'shadow'] };
```

Add mushroom rendering to `drawTile()` -- draw grass background, then a small purple circle with a white dot (similar to flower rendering but purple):

```javascript
} else if (type === 'mushroom') {
  ctx.fillStyle = TILE_COLORS['grass'];
  ctx.fillRect(x, y, TILE, TILE);
  // stem
  ctx.fillStyle = '#BCAAA4';
  ctx.fillRect(x + TILE/2 - 2, y + TILE/2, 4, TILE/2);
  // cap
  ctx.fillStyle = TILE_COLORS['mushroom'];
  ctx.beginPath();
  ctx.arc(x + TILE/2, y + TILE/2, TILE * 0.25, Math.PI, 0);
  ctx.fill();
  // spots
  ctx.fillStyle = '#fff';
  ctx.beginPath();
  ctx.arc(x + TILE/2, y + TILE/2 - 2, 2, 0, Math.PI * 2);
  ctx.fill();
}
```

Add moss rendering -- just a darker green variant of grass:

```javascript
} else if (type === 'moss') {
  ctx.fillStyle = TILE_COLORS['moss'];
  ctx.fillRect(x, y, TILE, TILE);
}
```

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add Enchanted Forest zone with mushrooms and shadow eggs"
```

---

### Task 3: Add Crystal Cave Zone

**Files:**
- Modify: `index.html` (CONSTANTS section)

Sparkling walls, underground pools, gem formations. Egg types: Star, Ice.

- [ ] **Step 1: Add tile types**

Add to `TILE_COLORS`:

```javascript
cave_wall:  '#4a4a5e',
cave_floor: '#6b6b80',
crystal:    '#E1BEE7',
gem_blue:   '#64B5F6',
gem_pink:   '#F48FB1',
cave_water: '#1565C0',
```

Add `'cave_wall'`, `'crystal'` to `SOLID_TILES`.

- [ ] **Step 2: Create ZONE_CRYSTAL_CAVE map**

A cave with walls around edges, open floor in center, crystals as obstacles, a small dark pool, gems scattered as decoration. Path at top for entrance from Enchanted Forest.

```javascript
const ZONE_CRYSTAL_CAVE = [
  ['cave_wall','cave_wall','cave_wall','cave_wall','path','path','path','path','path','path','path','path','path','path','path','path','path','cave_wall','cave_wall','cave_wall','cave_wall','cave_wall','cave_wall','cave_wall','cave_wall'],
  ['cave_wall','cave_wall','cave_floor','cave_floor','path','path','path','path','path','path','path','path','path','path','path','path','path','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_wall','cave_wall'],
  ['cave_wall','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_wall'],
  ['cave_wall','cave_floor','cave_floor','crystal','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','gem_blue','cave_floor','cave_floor','cave_floor','gem_pink','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','crystal','cave_floor','cave_floor','cave_wall'],
  ['cave_wall','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_wall'],
  ['cave_wall','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','crystal','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','crystal','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_wall'],
  ['cave_wall','cave_floor','gem_pink','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_water','cave_water','cave_water','cave_water','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','gem_blue','cave_floor','cave_floor','cave_wall'],
  ['cave_wall','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_water','cave_water','cave_water','cave_water','cave_water','cave_water','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_wall'],
  ['cave_wall','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_water','cave_water','cave_water','cave_water','cave_water','cave_water','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_wall'],
  ['cave_wall','cave_floor','cave_floor','cave_floor','cave_floor','crystal','cave_floor','cave_floor','cave_floor','cave_floor','cave_water','cave_water','cave_water','cave_water','cave_floor','cave_floor','cave_floor','cave_floor','crystal','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_wall'],
  ['cave_wall','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_wall'],
  ['cave_wall','cave_floor','gem_blue','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','gem_pink','cave_floor','cave_floor','cave_wall'],
  ['cave_wall','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','crystal','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','crystal','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_wall'],
  ['cave_wall','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_wall'],
  ['cave_wall','cave_wall','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','path','path','path','path','path','path','path','path','path','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_floor','cave_wall','cave_wall'],
  ['cave_wall','cave_wall','cave_wall','cave_wall','cave_wall','cave_wall','cave_wall','cave_wall','path','path','path','path','path','path','path','path','path','cave_wall','cave_wall','cave_wall','cave_wall','cave_wall','cave_wall','cave_wall','cave_wall'],
];
```

- [ ] **Step 3: Add to ZONES and update drawTile**

```javascript
ZONES.crystal = { map: ZONE_CRYSTAL_CAVE, name: 'Crystal Cave', eggs: ['star', 'ice'] };
```

Add cave tile rendering to `drawTile()`:

```javascript
} else if (type === 'crystal') {
  ctx.fillStyle = TILE_COLORS['cave_floor'];
  ctx.fillRect(x, y, TILE, TILE);
  ctx.fillStyle = TILE_COLORS['crystal'];
  // Diamond shape
  ctx.beginPath();
  ctx.moveTo(x + TILE/2, y + 4);
  ctx.lineTo(x + TILE - 6, y + TILE/2);
  ctx.lineTo(x + TILE/2, y + TILE - 4);
  ctx.lineTo(x + 6, y + TILE/2);
  ctx.closePath();
  ctx.fill();
  // Shine
  ctx.fillStyle = '#fff';
  ctx.beginPath();
  ctx.arc(x + TILE/2 - 3, y + TILE/2 - 3, 2, 0, Math.PI * 2);
  ctx.fill();
} else if (type === 'gem_blue' || type === 'gem_pink') {
  ctx.fillStyle = TILE_COLORS['cave_floor'];
  ctx.fillRect(x, y, TILE, TILE);
  ctx.fillStyle = TILE_COLORS[type];
  ctx.beginPath();
  ctx.arc(x + TILE/2, y + TILE/2, 4, 0, Math.PI * 2);
  ctx.fill();
  ctx.fillStyle = '#fff';
  ctx.beginPath();
  ctx.arc(x + TILE/2 - 1, y + TILE/2 - 1, 1.5, 0, Math.PI * 2);
  ctx.fill();
} else if (type === 'cave_water') {
  ctx.fillStyle = TILE_COLORS['cave_water'];
  ctx.fillRect(x, y, TILE, TILE);
}
```

Add `'cave_water'` to `SOLID_TILES`.

Update `spawnEggsForZone` to also allow `cave_floor` as valid spawn tiles:

```javascript
if (tile === 'grass' || tile === 'grass2' || tile === 'cave_floor') {
```

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add Crystal Cave zone with star and ice eggs"
```

---

### Task 4: Add Snowy Mountain Zone

**Files:**
- Modify: `index.html` (CONSTANTS section)

Snowdrifts, frozen waterfalls, icy paths. Egg types: Ice, Star.

- [ ] **Step 1: Add tile types**

Add to `TILE_COLORS`:

```javascript
snow:       '#f0f0f0',
snow2:      '#dde4e8',
ice:        '#bbdefb',
pine_top:   '#1B5E20',
pine_trunk: '#5D4037',
frozen_water:'#90CAF9',
snow_rock:  '#B0BEC5',
```

Add `'pine_top'`, `'pine_trunk'`, `'snow_rock'`, `'frozen_water'` to `SOLID_TILES`.

- [ ] **Step 2: Create ZONE_SNOWY_MOUNTAIN map**

Snow-covered terrain, pine trees, frozen water patches, rocks. Path at top.

```javascript
const ZONE_SNOWY_MOUNTAIN = [
  ['pine_top','pine_top','snow','snow','path','path','path','path','path','path','path','path','path','path','path','path','path','snow','snow','snow','snow','snow','snow','pine_top','pine_top'],
  ['pine_trunk','pine_trunk','snow','snow','path','path','path','path','path','path','path','path','path','path','path','path','path','snow','snow','snow','snow','snow','snow','pine_trunk','pine_trunk'],
  ['snow','snow','snow','snow','snow','snow','snow','snow2','snow','snow','snow','snow','snow','snow','snow','snow2','snow','snow','snow','snow','snow','snow','snow','snow','snow'],
  ['snow','snow2','snow','snow','snow','pine_top','pine_top','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','pine_top','pine_top','snow','snow','snow','snow2','snow','snow'],
  ['snow','snow','snow','snow','snow','pine_trunk','pine_trunk','snow','snow','snow2','snow','snow','snow','snow','snow2','snow','snow','pine_trunk','pine_trunk','snow','snow','snow','snow','snow','snow'],
  ['snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow'],
  ['snow','snow','snow','snow_rock','snow','snow','snow','snow','snow','frozen_water','frozen_water','frozen_water','frozen_water','frozen_water','frozen_water','frozen_water','snow','snow','snow','snow','snow','snow_rock','snow','snow','snow'],
  ['snow','snow','snow','snow','snow','snow','snow','snow','frozen_water','frozen_water','frozen_water','frozen_water','frozen_water','frozen_water','frozen_water','frozen_water','frozen_water','snow','snow','snow','snow','snow','snow','snow','snow'],
  ['snow','snow','snow','snow','snow','snow','snow','snow','frozen_water','frozen_water','frozen_water','frozen_water','frozen_water','frozen_water','frozen_water','frozen_water','frozen_water','snow','snow','snow','snow','snow','snow','snow','snow'],
  ['snow','snow2','snow','snow','snow','snow','snow','snow','snow','frozen_water','frozen_water','frozen_water','frozen_water','frozen_water','frozen_water','frozen_water','snow','snow','snow','snow','snow','snow','snow2','snow','snow'],
  ['snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow'],
  ['snow','snow','snow','pine_top','pine_top','snow','snow','snow','snow2','snow','snow','snow','snow','snow','snow','snow2','snow','snow','snow','snow','pine_top','pine_top','snow','snow','snow'],
  ['snow','snow','snow','pine_trunk','pine_trunk','snow','snow','snow','snow','snow','snow','snow_rock','snow','snow_rock','snow','snow','snow','snow','snow','snow','pine_trunk','pine_trunk','snow','snow','snow'],
  ['snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow','snow'],
  ['pine_top','pine_top','snow','snow','snow','snow','snow','snow','path','path','path','path','path','path','path','path','path','snow','snow','snow','snow','snow','snow','pine_top','pine_top'],
  ['pine_trunk','pine_trunk','snow','snow','snow','snow','snow','snow','path','path','path','path','path','path','path','path','path','snow','snow','snow','snow','snow','snow','pine_trunk','pine_trunk'],
];
```

- [ ] **Step 3: Add to ZONES and update drawTile and spawnEggsForZone**

```javascript
ZONES.snowy = { map: ZONE_SNOWY_MOUNTAIN, name: 'Snowy Mountain', eggs: ['ice', 'star'] };
```

Add snow/ice rendering (snow is white, snow2 is slight grey, ice is blue-tinted, pine trees are dark green). Add `'snow'` and `'snow2'` to the egg spawn check:

```javascript
if (tile === 'grass' || tile === 'grass2' || tile === 'cave_floor' || tile === 'snow' || tile === 'snow2') {
```

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add Snowy Mountain zone with ice and star eggs"
```

---

### Task 5: Add Volcano Ridge Zone

**Files:**
- Modify: `index.html` (CONSTANTS section)

Warm glow, lava streams (safe), dramatic rocks. Egg types: Fire, Shadow.

- [ ] **Step 1: Add tile types**

Add to `TILE_COLORS`:

```javascript
volcanic_rock: '#4E342E',
lava:          '#FF6F00',
lava2:         '#E65100',
magma_rock:    '#3E2723',
ash:           '#616161',
ember:         '#BF360C',
```

Add `'volcanic_rock'`, `'lava'`, `'lava2'`, `'magma_rock'` to `SOLID_TILES`.

- [ ] **Step 2: Create ZONE_VOLCANO_RIDGE map**

Dark rocky terrain with lava rivers, ash ground as walkable surface.

```javascript
const ZONE_VOLCANO_RIDGE = [
  ['volcanic_rock','volcanic_rock','ash','ash','path','path','path','path','path','path','path','path','path','path','path','path','path','ash','ash','ash','ash','ash','ash','volcanic_rock','volcanic_rock'],
  ['volcanic_rock','volcanic_rock','ash','ash','path','path','path','path','path','path','path','path','path','path','path','path','path','ash','ash','ash','ash','ash','ash','volcanic_rock','volcanic_rock'],
  ['ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash'],
  ['ash','ash','ash','magma_rock','ash','ash','ash','ash','ash','ash','ash','ember','ash','ember','ash','ash','ash','ash','ash','ash','magma_rock','ash','ash','ash','ash'],
  ['ash','ash','ash','ash','ash','ash','ash','ash','lava','lava','ash','ash','ash','ash','ash','lava','lava','ash','ash','ash','ash','ash','ash','ash','ash'],
  ['ash','ember','ash','ash','ash','ash','ash','ash','lava','lava','ash','ash','ash','ash','ash','lava','lava','ash','ash','ash','ash','ash','ember','ash','ash'],
  ['ash','ash','ash','ash','ash','ash','ash','ash','lava','lava','lava','ash','ash','ash','lava','lava','lava','ash','ash','ash','ash','ash','ash','ash','ash'],
  ['ash','ash','ash','ash','ash','magma_rock','ash','ash','ash','lava','lava','lava','lava','lava','lava','lava','ash','ash','ash','magma_rock','ash','ash','ash','ash','ash'],
  ['ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','lava','lava','lava','lava','lava','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash'],
  ['ash','ash','ember','ash','ash','ash','ash','ash','ash','ash','ash','lava','lava','lava','ash','ash','ash','ash','ash','ash','ash','ash','ember','ash','ash'],
  ['ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash'],
  ['ash','ash','ash','ash','ash','ash','magma_rock','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','magma_rock','ash','ash','ash','ash','ash','ash'],
  ['ash','ash','ash','ash','ember','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ember','ash','ash','ash','ash'],
  ['ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash','ash'],
  ['volcanic_rock','volcanic_rock','ash','ash','ash','ash','ash','ash','path','path','path','path','path','path','path','path','path','ash','ash','ash','ash','ash','ash','volcanic_rock','volcanic_rock'],
  ['volcanic_rock','volcanic_rock','ash','ash','ash','ash','ash','ash','path','path','path','path','path','path','path','path','path','ash','ash','ash','ash','ash','ash','volcanic_rock','volcanic_rock'],
];
```

- [ ] **Step 3: Add to ZONES, update drawTile and spawnEggsForZone**

```javascript
ZONES.volcano = { map: ZONE_VOLCANO_RIDGE, name: 'Volcano Ridge', eggs: ['fire', 'shadow'] };
```

Add lava rendering to `drawTile()` -- lava should animate/pulse:

```javascript
} else if (type === 'lava' || type === 'lava2') {
  const pulse = Math.sin(state.time * 3 + col * 0.5 + row * 0.3) * 0.15;
  const r = type === 'lava' ? 255 : 230;
  const g = type === 'lava' ? Math.floor(111 + pulse * 50) : Math.floor(81 + pulse * 40);
  ctx.fillStyle = `rgb(${r},${g},0)`;
  ctx.fillRect(x, y, TILE, TILE);
} else if (type === 'ember') {
  ctx.fillStyle = TILE_COLORS['ash'];
  ctx.fillRect(x, y, TILE, TILE);
  ctx.fillStyle = TILE_COLORS['ember'];
  ctx.beginPath();
  ctx.arc(x + TILE/2, y + TILE/2, 3, 0, Math.PI * 2);
  ctx.fill();
}
```

Add `'ash'` to the egg spawn check:

```javascript
if (tile === 'grass' || tile === 'grass2' || tile === 'cave_floor' || tile === 'snow' || tile === 'snow2' || tile === 'ash') {
```

Note: `drawTile` needs access to `state.time` for lava animation. Since `state` is defined after rendering functions, either move the lava rendering to a separate function called from the game loop, or reference `state.time` directly (it will be available at runtime since `drawTile` is only called during the game loop after state exists). No code changes needed -- JS hoisting handles this since `state` is declared with `const` in the same scope.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add Volcano Ridge zone with animated lava and fire/shadow eggs"
```

---

### Task 6: Add Rainbow Falls Zone (Secret Final Zone)

**Files:**
- Modify: `index.html` (CONSTANTS section)

Waterfalls with rainbow mist, prismatic light. Only Rainbow eggs. Unlocked after collecting all 6 other dragon types.

- [ ] **Step 1: Add tile types**

Add to `TILE_COLORS`:

```javascript
rainbow_grass:  '#81C784',
rainbow_water:  '#4DD0E1',
rainbow_rock:   '#B0BEC5',
rainbow_flower: '#FF80AB',
sparkle_tile:   '#FFF176',
```

Add `'rainbow_rock'`, `'rainbow_water'` to `SOLID_TILES`.

- [ ] **Step 2: Create ZONE_RAINBOW_FALLS map**

A magical zone with rainbow water features, sparkling tiles, colorful flowers. Path at top only (no exit at bottom -- it's the final zone).

```javascript
const ZONE_RAINBOW_FALLS = [
  ['rainbow_rock','rainbow_rock','rainbow_grass','rainbow_grass','path','path','path','path','path','path','path','path','path','path','path','path','path','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_rock','rainbow_rock'],
  ['rainbow_rock','rainbow_grass','rainbow_grass','rainbow_grass','path','path','path','path','path','path','path','path','path','path','path','path','path','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_rock'],
  ['rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass'],
  ['rainbow_grass','rainbow_grass','sparkle_tile','rainbow_grass','rainbow_grass','rainbow_flower','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_flower','rainbow_grass','rainbow_grass','sparkle_tile','rainbow_grass','rainbow_grass','rainbow_grass'],
  ['rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_water','rainbow_water','rainbow_water','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_water','rainbow_water','rainbow_water','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass'],
  ['rainbow_grass','rainbow_flower','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_water','rainbow_water','rainbow_water','rainbow_water','rainbow_water','rainbow_grass','rainbow_water','rainbow_water','rainbow_water','rainbow_water','rainbow_water','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_flower','rainbow_grass','rainbow_grass'],
  ['rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_water','rainbow_water','rainbow_water','rainbow_water','rainbow_water','rainbow_water','rainbow_water','rainbow_water','rainbow_water','rainbow_water','rainbow_water','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass'],
  ['rainbow_grass','rainbow_grass','rainbow_grass','sparkle_tile','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_water','rainbow_water','rainbow_water','rainbow_water','rainbow_water','rainbow_water','rainbow_water','rainbow_water','rainbow_water','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','sparkle_tile','rainbow_grass','rainbow_grass','rainbow_grass'],
  ['rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_water','rainbow_water','rainbow_water','rainbow_water','rainbow_water','rainbow_water','rainbow_water','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass'],
  ['rainbow_grass','rainbow_grass','rainbow_flower','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_water','rainbow_water','rainbow_water','rainbow_water','rainbow_water','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_flower','rainbow_grass','rainbow_grass','rainbow_grass'],
  ['rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass'],
  ['rainbow_grass','sparkle_tile','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','sparkle_tile','rainbow_grass','rainbow_grass'],
  ['rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_flower','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_flower','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_flower','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_flower','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass'],
  ['rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass'],
  ['rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass'],
  ['rainbow_rock','rainbow_rock','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_grass','rainbow_rock','rainbow_rock'],
];
```

- [ ] **Step 3: Add to ZONES, update drawTile and spawnEggsForZone**

```javascript
ZONES.rainbow = { map: ZONE_RAINBOW_FALLS, name: 'Rainbow Falls', eggs: ['rainbow'] };
```

Add rainbow tile rendering:

```javascript
} else if (type === 'rainbow_grass') {
  ctx.fillStyle = TILE_COLORS['rainbow_grass'];
  ctx.fillRect(x, y, TILE, TILE);
} else if (type === 'rainbow_water') {
  // Animated rainbow shimmer
  const hue = ((state.time * 50) + col * 20 + row * 20) % 360;
  ctx.fillStyle = `hsl(${hue}, 70%, 60%)`;
  ctx.fillRect(x, y, TILE, TILE);
} else if (type === 'rainbow_flower') {
  ctx.fillStyle = TILE_COLORS['rainbow_grass'];
  ctx.fillRect(x, y, TILE, TILE);
  const hue2 = ((state.time * 30) + col * 40) % 360;
  ctx.fillStyle = `hsl(${hue2}, 80%, 70%)`;
  ctx.beginPath();
  ctx.arc(x + TILE/2, y + TILE/2, TILE * 0.28, 0, Math.PI * 2);
  ctx.fill();
  ctx.fillStyle = '#fff';
  ctx.beginPath();
  ctx.arc(x + TILE/2, y + TILE/2, TILE * 0.1, 0, Math.PI * 2);
  ctx.fill();
} else if (type === 'sparkle_tile') {
  ctx.fillStyle = TILE_COLORS['rainbow_grass'];
  ctx.fillRect(x, y, TILE, TILE);
  const sparkle = Math.sin(state.time * 5 + col + row) * 0.5 + 0.5;
  ctx.fillStyle = `rgba(255, 255, 255, ${sparkle * 0.6})`;
  ctx.fillRect(x + TILE/2 - 2, y + TILE/2 - 2, 4, 4);
}
```

Add `'rainbow_grass'` to egg spawn check:

```javascript
if (tile === 'grass' || tile === 'grass2' || tile === 'cave_floor' || tile === 'snow' || tile === 'snow2' || tile === 'ash' || tile === 'rainbow_grass') {
```

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add Rainbow Falls secret zone with animated rainbow water"
```

---

### Task 7: Update Zone Connections & Progressive Unlocking

**Files:**
- Modify: `index.html` (ZONE_CONNECTIONS, checkUnlocks, startGame)

Wire all 7 zones together in a linear chain with progressive unlocking.

- [ ] **Step 1: Update ZONE_CONNECTIONS**

Replace the existing `ZONE_CONNECTIONS` array:

```javascript
const ZONE_CONNECTIONS = [
  { from: 'meadow',    edge: 'bottom', to: 'heart',     unlockRequired: true },
  { from: 'heart',     edge: 'top',    to: 'meadow',    unlockRequired: false },
  { from: 'heart',     edge: 'bottom', to: 'enchanted',  unlockRequired: true },
  { from: 'enchanted', edge: 'top',    to: 'heart',     unlockRequired: false },
  { from: 'enchanted', edge: 'bottom', to: 'crystal',   unlockRequired: true },
  { from: 'crystal',   edge: 'top',    to: 'enchanted', unlockRequired: false },
  { from: 'crystal',   edge: 'bottom', to: 'snowy',     unlockRequired: true },
  { from: 'snowy',     edge: 'top',    to: 'crystal',   unlockRequired: false },
  { from: 'snowy',     edge: 'bottom', to: 'volcano',   unlockRequired: true },
  { from: 'volcano',   edge: 'top',    to: 'snowy',     unlockRequired: false },
  { from: 'volcano',   edge: 'bottom', to: 'rainbow',   unlockRequired: true },
  { from: 'rainbow',   edge: 'top',    to: 'volcano',   unlockRequired: false },
];
```

- [ ] **Step 2: Update checkUnlocks for progressive unlocking**

Replace the existing `checkUnlocks()`:

```javascript
function checkUnlocks() {
  if (state.mode !== 'adventure') return;

  const total = Object.values(state.collection).reduce((a, b) => a + b, 0);
  const types = Object.keys(state.collection).length;

  const unlocks = [
    { zone: 'heart',     condition: total >= 3 },
    { zone: 'enchanted', condition: total >= 8 },
    { zone: 'crystal',   condition: total >= 14 },
    { zone: 'snowy',     condition: total >= 20 },
    { zone: 'volcano',   condition: total >= 27 },
    { zone: 'rainbow',   condition: types >= 6 }, // must have all 6 non-rainbow types
  ];

  for (const u of unlocks) {
    if (u.condition && !state.unlockedZones.includes(u.zone)) {
      state.unlockedZones.push(u.zone);
      state.unlockEffect = { timer: 0, zone: u.zone };
    }
  }
}
```

- [ ] **Step 3: Update drawUnlockEffect to show zone name**

Replace hardcoded "Heart Forest" with the actual zone name:

```javascript
function drawUnlockEffect() {
  if (!state.unlockEffect) return;
  const t = state.unlockEffect.timer;
  const alpha = Math.max(0, 1 - t / 2);
  const zoneName = ZONES[state.unlockEffect.zone].name;

  for (let i = 0; i < 15; i++) {
    const x = (8 + i) * TILE + Math.sin(t * 5 + i) * 5;
    const y = (MAP_ROWS - 1 + UI_ROWS) * TILE + Math.cos(t * 4 + i * 0.5) * 10;
    ctx.fillStyle = `rgba(255, 255, 100, ${alpha * (0.5 + Math.sin(t * 8 + i) * 0.5)})`;
    ctx.fillRect(x - 2, y - 2, 4, 4);
  }

  if (t < 1.5) {
    ctx.fillStyle = `rgba(255, 255, 255, ${alpha})`;
    ctx.font = 'bold 18px monospace';
    ctx.textAlign = 'center';
    ctx.fillText(zoneName + ' unlocked!', CANVAS_W / 2, CANVAS_H / 2 - 20);
    ctx.font = '13px monospace';
    ctx.fillText('Walk to the path at the bottom!', CANVAS_W / 2, CANVAS_H / 2 + 5);
  }
}
```

- [ ] **Step 4: Update drawUI unlock progress**

Replace the hardcoded Heart Forest progress text with dynamic next-zone info:

```javascript
if (state.mode === 'adventure') {
  const total = Object.values(state.collection).reduce((a, b) => a + b, 0);
  const types = Object.keys(state.collection).length;
  const nextUnlocks = [
    { zone: 'heart', need: 3, msg: 'Collect 3 eggs' },
    { zone: 'enchanted', need: 8, msg: 'Collect 8 eggs' },
    { zone: 'crystal', need: 14, msg: 'Collect 14 eggs' },
    { zone: 'snowy', need: 20, msg: 'Collect 20 eggs' },
    { zone: 'volcano', need: 27, msg: 'Collect 27 eggs' },
    { zone: 'rainbow', need: 6, msg: 'Collect all 6 dragon types', useTypes: true },
  ];

  ctx.textAlign = 'center';
  ctx.fillStyle = '#aaa';
  ctx.font = '11px monospace';

  let shown = false;
  for (const u of nextUnlocks) {
    if (!state.unlockedZones.includes(u.zone)) {
      const current = u.useTypes ? types : total;
      ctx.fillText(u.msg + ' to unlock ' + ZONES[u.zone].name + ' (' + current + '/' + u.need + ')', CANVAS_W / 2, CANVAS_H - 8);
      shown = true;
      break;
    }
  }
  if (!shown) {
    ctx.fillText('All zones unlocked! Find the Rainbow Dragon!', CANVAS_W / 2, CANVAS_H - 8);
  }
}
```

- [ ] **Step 5: Update startGame for freeplay**

In `startGame()`, update freeplay to unlock all zones:

```javascript
if (mode === 'freeplay') {
  state.unlockedZones = ['meadow', 'heart', 'enchanted', 'crystal', 'snowy', 'volcano', 'rainbow'];
} else {
  state.unlockedZones = ['meadow'];
}
```

- [ ] **Step 6: Update drawUI collection tracker to show all 7 types**

Replace the `allTypes` array and adjust spacing:

```javascript
const allTypes = ['fire', 'nature', 'love', 'shadow', 'star', 'ice', 'rainbow'];
const startX = 10;
const y = 34;

for (let i = 0; i < allTypes.length; i++) {
  const type = allTypes[i];
  const count = state.collection[type] || 0;
  const x = startX + i * 55; // tighter spacing for 7 items
  // ... rest same but with tighter spacing
}
```

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "feat: add progressive zone unlocking chain through all 7 zones"
```

---

### Task 8: Add Zone-Specific Particle Effects

**Files:**
- Modify: `index.html` (PARTICLES section -- spawnParticle function)

Each zone gets unique ambient particles.

- [ ] **Step 1: Update spawnParticle for all zones**

Replace the `spawnParticle` function to handle all 7 zones:

```javascript
function spawnParticle(zoneKey) {
  const p = {
    x: Math.random() * CANVAS_W,
    y: Math.random() * CANVAS_H,
    vx: (Math.random() - 0.5) * 20,
    vy: -Math.random() * 15 - 5,
    life: Math.random() * 3 + 2,
    maxLife: 0,
    size: Math.random() * 3 + 2,
    type: zoneKey,
    isHeart: false,
    isSnowflake: false,
    isEmber: false,
    isStar: false,
    isRainbow: false,
  };
  p.maxLife = p.life;

  if (zoneKey === 'meadow') {
    p.vy = Math.random() * 10 + 5;
    p.vx = (Math.random() - 0.5) * 30;
    p.color = ['#e74c3c', '#f1c40f', '#e84393', '#fff'][Math.floor(Math.random() * 4)];
  } else if (zoneKey === 'heart') {
    p.vy = -(Math.random() * 10 + 5);
    p.vx = (Math.random() - 0.5) * 15;
    p.color = '#ff69b4';
    p.isHeart = true;
  } else if (zoneKey === 'enchanted') {
    // Fireflies -- small glowing dots that drift slowly
    p.vx = (Math.random() - 0.5) * 10;
    p.vy = (Math.random() - 0.5) * 10;
    p.color = '#FFEB3B';
    p.size = Math.random() * 2 + 1;
  } else if (zoneKey === 'crystal') {
    // Twinkling gems -- sparkle and fade
    p.vx = (Math.random() - 0.5) * 5;
    p.vy = -(Math.random() * 5 + 2);
    p.color = ['#E1BEE7', '#64B5F6', '#F48FB1', '#fff'][Math.floor(Math.random() * 4)];
    p.isStar = true;
    p.size = Math.random() * 2 + 1;
  } else if (zoneKey === 'snowy') {
    // Snowflakes -- drift down gently
    p.vy = Math.random() * 15 + 8;
    p.vx = (Math.random() - 0.5) * 20;
    p.color = '#fff';
    p.isSnowflake = true;
    p.size = Math.random() * 2 + 2;
  } else if (zoneKey === 'volcano') {
    // Rising embers -- float upward
    p.vy = -(Math.random() * 20 + 10);
    p.vx = (Math.random() - 0.5) * 15;
    p.color = ['#FF6F00', '#FF8F00', '#FFD600'][Math.floor(Math.random() * 3)];
    p.isEmber = true;
    p.size = Math.random() * 2 + 1;
  } else if (zoneKey === 'rainbow') {
    // Prismatic sparkles -- all colors, drift in all directions
    p.vx = (Math.random() - 0.5) * 20;
    p.vy = (Math.random() - 0.5) * 20;
    p.isRainbow = true;
    p.size = Math.random() * 3 + 2;
  }

  state.particles.push(p);
}
```

- [ ] **Step 2: Update drawParticles for new types**

Replace the `drawParticles` function:

```javascript
function drawParticles() {
  for (const p of state.particles) {
    const alpha = Math.min(1, p.life / (p.maxLife * 0.3));
    ctx.globalAlpha = alpha * 0.6;

    if (p.isHeart) {
      ctx.fillStyle = p.color;
      ctx.font = (p.size * 3) + 'px serif';
      ctx.textAlign = 'left';
      ctx.fillText('\u2665', p.x, p.y);
    } else if (p.isSnowflake) {
      ctx.fillStyle = p.color;
      ctx.font = (p.size * 3) + 'px serif';
      ctx.textAlign = 'left';
      ctx.fillText('\u2744', p.x, p.y);
    } else if (p.isStar) {
      ctx.fillStyle = p.color;
      ctx.font = (p.size * 3) + 'px serif';
      ctx.textAlign = 'left';
      ctx.fillText('\u2726', p.x, p.y);
    } else if (p.isEmber) {
      ctx.fillStyle = p.color;
      ctx.globalAlpha = alpha * 0.8;
      ctx.fillRect(p.x - p.size/2, p.y - p.size/2, p.size, p.size);
    } else if (p.isRainbow) {
      const hue = (state.time * 100 + p.x + p.y) % 360;
      ctx.fillStyle = `hsl(${hue}, 80%, 70%)`;
      ctx.beginPath();
      ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
      ctx.fill();
    } else {
      ctx.fillStyle = p.color;
      ctx.beginPath();
      ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
      ctx.fill();
    }
  }
  ctx.globalAlpha = 1;
}
```

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add zone-specific particles -- fireflies, gems, snow, embers, rainbow sparkles"
```

---

### Task 9: Add Path Tiles to Heart Forest Bottom

**Files:**
- Modify: `index.html` (ZONE_HEART_FOREST map)

Heart Forest currently only has a path at the top (entrance from meadow). We need a path at the bottom too for the exit to Enchanted Forest.

- [ ] **Step 1: Update ZONE_HEART_FOREST rows 14-15**

Replace the last 2 rows of ZONE_HEART_FOREST to add path tiles at the bottom:

Row 14: same as current but cols 8-16 become `'path'`
Row 15: same as current but cols 8-16 become `'path'`

```javascript
// row 14 - replace to add path
['heart_leaf','heart_leaf','grass','grass','grass','grass','grass','grass','path','path','path','path','path','path','path','path','path','grass','grass','grass','grass','grass','heart_leaf','heart_leaf','grass'],
// row 15 - replace to add path  
['heart_tree','heart_tree','grass','grass','grass','flower_p','grass','grass','path','path','path','path','path','path','path','path','path','grass','grass','flower_p','grass','grass','heart_tree','heart_tree','grass'],
```

- [ ] **Step 2: Similarly update Enchanted Forest, Crystal Cave, Snowy Mountain, Volcano Ridge**

All zones need paths at both top (entrance) and bottom (exit), EXCEPT Rainbow Falls which only has a path at top. Verify each zone map has `'path'` tiles at:
- Top: rows 0-1, approximately cols 4-16 (or 8-16)
- Bottom: rows 14-15, approximately cols 8-16

Check each zone and add paths where missing. The maps created in Tasks 2-6 should already have paths at top and bottom -- verify and fix any that don't.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add connecting paths to all zones for full progression chain"
```

---

### Task 10: Integration Test & Push

**Files:**
- Modify: `index.html` (any final fixes)

- [ ] **Step 1: Full playthrough test**

Open `index.html` in a browser and verify:

1. Title screen works, both modes selectable
2. **Flower Meadow**: Fire and Nature eggs spawn, petals float
3. Collect 3 eggs → "Heart Forest unlocked!" with sparkle
4. **Heart Forest**: Love eggs, floating hearts, path at bottom
5. Collect 8 total → "Enchanted Forest unlocked!"
6. **Enchanted Forest**: Nature and Shadow eggs, firefly particles, mushrooms visible
7. Collect 14 total → "Crystal Cave unlocked!"
8. **Crystal Cave**: Star and Ice eggs, gem sparkle particles, crystal obstacles
9. Collect 20 total → "Snowy Mountain unlocked!"
10. **Snowy Mountain**: Ice and Star eggs, snowflake particles
11. Collect 27 total → "Volcano Ridge unlocked!"
12. **Volcano Ridge**: Fire and Shadow eggs, rising embers, animated lava
13. Collect all 6 types → "Rainbow Falls unlocked!"
14. **Rainbow Falls**: Rainbow eggs only, prismatic sparkles, rainbow water animation
15. Collection tracker shows all 7 dragon types
16. Free Play mode has all zones accessible
17. Touch controls still work on iPad

- [ ] **Step 2: Fix any issues found**

Address any bugs or visual issues discovered during testing.

- [ ] **Step 3: Final commit and push**

```bash
git add index.html
git commit -m "feat: complete Stage 2 -- all 7 zones playable with full progression"
git push origin main
```

After pushing, the new version will auto-deploy to GitHub Pages at `https://albsba.github.io/dragon-egg-island/`

---

## Summary

| Task | What it adds |
|------|-------------|
| 1 | Shadow, Star, Ice, Rainbow egg & dragon sprites |
| 2 | Enchanted Forest zone (nature, shadow) |
| 3 | Crystal Cave zone (star, ice) |
| 4 | Snowy Mountain zone (ice, star) |
| 5 | Volcano Ridge zone (fire, shadow) |
| 6 | Rainbow Falls secret zone (rainbow) |
| 7 | Progressive zone unlocking chain + UI updates |
| 8 | Zone-specific particle effects |
| 9 | Connecting paths between all zones |
| 10 | Integration test and deploy |
