# Dragon Egg Island -- Stage 1 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a playable 2-player cooperative dragon egg collecting game in a single HTML file, with two zones (Flower Meadow, Heart Forest), three egg types (Fire, Nature, Love), hatching animations, and baby dragon followers.

**Architecture:** Single `index.html` file with embedded CSS and JavaScript. Canvas element for game rendering. Game state managed in plain objects. Game loop via `requestAnimationFrame`. Tile-based world with two zones connected by a walkable path. Input handled via `keydown`/`keyup` event listeners tracking pressed keys for both players simultaneously.

**Tech Stack:** Vanilla HTML5, CSS, JavaScript, Canvas API. No dependencies. No build step.

---

## File Structure

```
harper-audrey-game/
├── index.html          # The entire game -- HTML + CSS + JS in one file
├── docs/
│   ├── ROADMAP.md
│   └── superpowers/
│       ├── specs/2026-04-01-dragon-egg-island-design.md
│       └── plans/2026-04-01-dragon-egg-island-stage1.md
```

Single file: `index.html`. All game code lives here. Sections are organized with clear comment banners:

1. **HTML/CSS** -- Canvas element, start screen overlay, collection tracker UI
2. **Constants** -- Tile size, colors, sprite data, zone maps, egg configs
3. **Sprite Renderer** -- Functions to draw pixel art characters, eggs, dragons on canvas
4. **Game State** -- Players, eggs, dragons, zones, mode, unlock status
5. **Input Handler** -- Keyboard tracking for two players
6. **Game Logic** -- Movement, collision, egg collection, hatching, zone transitions
7. **Particle System** -- Simple ambient particles per zone
8. **Rendering** -- Draw world, characters, eggs, dragons, UI each frame
9. **Game Loop** -- requestAnimationFrame loop tying it all together
10. **Initialization** -- Start screen, mode selection, game boot

---

### Task 1: Project Setup & Canvas Boilerplate

**Files:**
- Create: `index.html`

This task creates the HTML shell with a canvas element and a basic game loop that clears the screen each frame.

- [ ] **Step 1: Initialize git repo**

```bash
cd C:/projects/harper-audrey-game
git init
echo ".superpowers/" > .gitignore
git add .gitignore docs/
git commit -m "docs: add game design spec, roadmap, and implementation plan"
```

- [ ] **Step 2: Create index.html with canvas and game loop**

Create `index.html` with:

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Dragon Egg Island</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    background: #1a1a2e;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    overflow: hidden;
  }
  canvas {
    image-rendering: pixelated;
    image-rendering: crisp-edges;
    border: 3px solid #333;
    background: #2d5a27;
  }
</style>
</head>
<body>
<canvas id="game" width="800" height="600"></canvas>
<script>
// ============================================================
// CONSTANTS
// ============================================================
const CANVAS_W = 800;
const CANVAS_H = 600;
const TILE = 32;
const COLS = Math.floor(CANVAS_W / TILE); // 25
const ROWS = Math.floor(CANVAS_H / TILE); // 18 (with 2 rows for UI)

// ============================================================
// GAME STATE
// ============================================================
const state = {
  screen: 'title', // 'title' | 'playing'
  mode: null,      // 'adventure' | 'freeplay'
  time: 0,
};

// ============================================================
// CANVAS SETUP
// ============================================================
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');

// ============================================================
// GAME LOOP
// ============================================================
let lastTime = 0;

function gameLoop(timestamp) {
  const dt = (timestamp - lastTime) / 1000;
  lastTime = timestamp;
  state.time += dt;

  // Clear
  ctx.fillStyle = '#2d5a27';
  ctx.fillRect(0, 0, CANVAS_W, CANVAS_H);

  // Placeholder text
  ctx.fillStyle = '#fff';
  ctx.font = '24px monospace';
  ctx.textAlign = 'center';
  ctx.fillText('Dragon Egg Island', CANVAS_W / 2, CANVAS_H / 2);
  ctx.font = '14px monospace';
  ctx.fillText('Game loop running...', CANVAS_W / 2, CANVAS_H / 2 + 30);

  requestAnimationFrame(gameLoop);
}

requestAnimationFrame(gameLoop);
</script>
</body>
</html>
```

- [ ] **Step 3: Verify in browser**

Open `index.html` in a browser. You should see a dark green canvas with "Dragon Egg Island" and "Game loop running..." centered on screen.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add canvas boilerplate with game loop"
```

---

### Task 2: Pixel Art Sprite Renderer

**Files:**
- Modify: `index.html` (add after CONSTANTS section)

This task adds functions to draw pixel art sprites on the canvas. Sprites are defined as 2D arrays of color values. Each cell is one "pixel" scaled up to the tile size.

- [ ] **Step 1: Add sprite data for both characters**

Add after the CONSTANTS section in `index.html`:

```javascript
// ============================================================
// SPRITE DATA
// ============================================================
// Each sprite is an array of rows. Each row is an array of color strings.
// '' = transparent, any other string = fill color.
// Sprites are 10x12 "pixels", rendered scaled to fit tile size.

const SPRITES = {
  harper: [
    ['','','','#87CEEB','#87CEEB','#87CEEB','#87CEEB','','',''],
    ['','','#87CEEB','#87CEEB','#87CEEB','#87CEEB','#87CEEB','#87CEEB','',''],
    ['','','#87CEEB','#FFDAB9','#FFDAB9','#FFDAB9','#FFDAB9','#87CEEB','',''],
    ['','','#FFDAB9','#FFDAB9','#FFDAB9','#FFDAB9','#FFDAB9','#FFDAB9','',''],
    ['','','#FFDAB9','#333','','','#333','#FFDAB9','',''],
    ['','','#FFDAB9','','','','','#FFDAB9','',''],
    ['','','','#FFDAB9','#f9a','#f9a','#FFDAB9','','',''],
    ['','','#B0E0E6','#B0E0E6','#B0E0E6','#B0E0E6','#B0E0E6','#B0E0E6','',''],
    ['','#FFDAB9','#B0E0E6','#B0E0E6','#B0E0E6','#B0E0E6','#B0E0E6','#B0E0E6','#FFDAB9',''],
    ['','','#B0E0E6','#B0E0E6','#E0F7FA','#B0E0E6','#B0E0E6','#B0E0E6','',''],
    ['','','','#5b8','','','#5b8','','',''],
    ['','','','#654','','','#654','','',''],
  ],
  audrey: [
    ['','','','#9B59B6','#9B59B6','#9B59B6','#9B59B6','','',''],
    ['','','#9B59B6','#9B59B6','#9B59B6','#9B59B6','#9B59B6','#9B59B6','',''],
    ['','','#9B59B6','#FFDAB9','#FFDAB9','#FFDAB9','#FFDAB9','#9B59B6','',''],
    ['','','#FFDAB9','#FFDAB9','#FFDAB9','#FFDAB9','#FFDAB9','#FFDAB9','',''],
    ['','','#FFDAB9','#333','','','#333','#FFDAB9','',''],
    ['','','#FFDAB9','','','','','#FFDAB9','',''],
    ['','','','#FFDAB9','#f9a','#f9a','#FFDAB9','','',''],
    ['','','#8E44AD','#8E44AD','#8E44AD','#8E44AD','#8E44AD','#8E44AD','',''],
    ['','#FFDAB9','#8E44AD','#8E44AD','#8E44AD','#8E44AD','#8E44AD','#8E44AD','#FFDAB9',''],
    ['','','#8E44AD','#8E44AD','#D7BDE2','#8E44AD','#8E44AD','#8E44AD','',''],
    ['','','','#c6e','','','#c6e','','',''],
    ['','','','#654','','','#654','','',''],
  ],
};

function drawSprite(sprite, x, y, size) {
  const rows = sprite.length;
  const cols = sprite[0].length;
  const px = size / Math.max(rows, cols);
  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      if (sprite[r][c]) {
        ctx.fillStyle = sprite[r][c];
        ctx.fillRect(
          x + c * px,
          y + r * px,
          Math.ceil(px),
          Math.ceil(px)
        );
      }
    }
  }
}
```

- [ ] **Step 2: Add egg sprite data**

```javascript
// Egg sprites are 8x10 pixels
function makeEggSprite(color1, color2, shine) {
  return [
    ['','','',color1,color1,'','',''],
    ['','',color1,color1,color1,color1,'',''],
    ['',color1,color1,shine,color1,color1,color1,''],
    ['',color1,color1,shine,color1,color1,color1,''],
    [color2,color1,color1,color1,color1,color1,color1,color2],
    [color2,color2,color1,color1,color1,color1,color2,color2],
    ['',color2,color2,color1,color1,color2,color2,''],
    ['','',color2,color2,color2,color2,'',''],
  ];
}

const EGG_SPRITES = {
  fire:   makeEggSprite('#FF6B35', '#FF4500', '#FFD700'),
  nature: makeEggSprite('#66BB6A', '#43A047', '#A5D6A7'),
  love:   makeEggSprite('#FF69B4', '#E91E63', '#FFB6C1'),
};
```

- [ ] **Step 3: Add baby dragon sprite data**

```javascript
// Baby dragon sprites are 8x8 pixels
function makeDragonSprite(body, belly, eye) {
  return [
    ['','',body,body,body,'','',''],
    ['',body,eye,body,body,body,'',''],
    ['',body,body,body,belly,body,'',''],
    [body,body,body,belly,belly,body,'',''],
    ['',body,body,belly,body,body,body,''],
    ['','',body,body,body,'',body,''],
    ['','',body,'',body,'','',''],
    ['','',body,'',body,'','',''],
  ];
}

const DRAGON_SPRITES = {
  fire:   makeDragonSprite('#FF4500', '#FFD700', '#333'),
  nature: makeDragonSprite('#43A047', '#A5D6A7', '#333'),
  love:   makeDragonSprite('#E91E63', '#FFB6C1', '#333'),
};
```

- [ ] **Step 4: Test sprites by drawing them in the game loop**

Temporarily replace the placeholder text in the game loop with:

```javascript
// Test sprites
drawSprite(SPRITES.harper, 200, 250, TILE);
drawSprite(SPRITES.audrey, 280, 250, TILE);
drawSprite(EGG_SPRITES.fire, 370, 258, TILE * 0.75);
drawSprite(EGG_SPRITES.nature, 420, 258, TILE * 0.75);
drawSprite(EGG_SPRITES.love, 470, 258, TILE * 0.75);
drawSprite(DRAGON_SPRITES.fire, 370, 300, TILE * 0.6);
drawSprite(DRAGON_SPRITES.nature, 420, 300, TILE * 0.6);
drawSprite(DRAGON_SPRITES.love, 470, 300, TILE * 0.6);

ctx.fillStyle = '#fff';
ctx.font = '24px monospace';
ctx.textAlign = 'center';
ctx.fillText('Dragon Egg Island - Sprite Test', CANVAS_W / 2, 100);
```

Open in browser. You should see Harper (light blue), Audrey (purple), three eggs (orange, green, pink), and three baby dragons below.

- [ ] **Step 5: Remove test rendering, commit**

Remove the test sprite drawing code from the game loop (restore it to just clearing the screen). The sprites are ready to use but shouldn't render until the game is playing.

```bash
git add index.html
git commit -m "feat: add pixel art sprite renderer with characters, eggs, and dragons"
```

---

### Task 3: Input Handler

**Files:**
- Modify: `index.html` (add after SPRITE DATA section)

This task adds keyboard input tracking so both players can move simultaneously.

- [ ] **Step 1: Add input tracking**

```javascript
// ============================================================
// INPUT
// ============================================================
const keys = {};

window.addEventListener('keydown', (e) => {
  keys[e.key] = true;
  // Prevent arrow keys from scrolling the page
  if (['ArrowUp','ArrowDown','ArrowLeft','ArrowRight',' '].includes(e.key)) {
    e.preventDefault();
  }
});

window.addEventListener('keyup', (e) => {
  keys[e.key] = false;
});

// Helper: returns {dx, dy} for a player based on their keys
function getPlayerInput(up, down, left, right) {
  let dx = 0, dy = 0;
  if (keys[up]) dy = -1;
  if (keys[down]) dy = 1;
  if (keys[left]) dx = -1;
  if (keys[right]) dx = 1;
  return { dx, dy };
}
```

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "feat: add dual-player keyboard input handler"
```

---

### Task 4: Tile Map & Zone Rendering

**Files:**
- Modify: `index.html` (add zone data to CONSTANTS, add rendering functions)

This task creates the two zone tile maps and renders them. The game world is a grid of tiles. Each tile has a type (grass, flower, tree, water, path, heart-tree, etc.) that determines its color.

- [ ] **Step 1: Add tile type colors and zone maps**

Add to the CONSTANTS section:

```javascript
// Tile types and their colors
const TILE_COLORS = {
  grass:      '#4a8f3f',
  grass2:     '#3d7a34',
  flower_r:   '#e74c3c',
  flower_y:   '#f1c40f',
  flower_p:   '#e84393',
  tree_trunk: '#8B4513',
  tree_top:   '#2d7a1e',
  water:      '#3498db',
  path:       '#c4a265',
  rock:       '#888',
  heart_tree: '#c0392b',
  heart_leaf: '#ff69b4',
  bush:       '#27ae60',
  bridge:     '#a0522d',
};

// UI_ROWS at top for collection tracker
const UI_ROWS = 2;
const MAP_ROWS = ROWS - UI_ROWS; // 16 playable rows

// Zone maps: 2D arrays of tile type strings, COLS x MAP_ROWS
// 'E' marks possible egg spawn positions (rendered as the zone's base tile)
// Zones are COLS(25) wide x MAP_ROWS(16) tall

const ZONE_FLOWER_MEADOW = [
  // Row 0
  'tree_top,tree_top,grass,grass,flower_r,grass,grass,flower_y,grass,grass,grass,flower_p,grass,grass,grass,grass,flower_r,grass,grass,flower_y,grass,grass,grass,tree_top,tree_top'.split(','),
  'tree_trunk,tree_trunk,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,tree_trunk,tree_trunk'.split(','),
  'grass,grass,grass,flower_y,grass,grass,grass,grass,grass,flower_r,grass,grass,grass,grass,grass,flower_p,grass,grass,grass,grass,flower_y,grass,grass,grass,grass'.split(','),
  'grass,grass,grass,grass,grass,grass,bush,grass,grass,grass,grass,grass,flower_y,grass,grass,grass,grass,grass,bush,grass,grass,grass,grass,grass,grass'.split(','),
  'grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass'.split(','),
  'grass,flower_p,grass,grass,grass,grass,grass,grass,grass,grass,grass2,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,flower_r,grass,grass'.split(','),
  'grass,grass,grass,grass,grass,grass,grass,grass,grass,grass2,grass2,grass2,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass'.split(','),
  'grass,grass,grass,bush,grass,grass,grass,grass,grass,grass,grass2,grass,grass,grass,grass,flower_y,grass,grass,grass,grass,grass,bush,grass,grass,grass'.split(','),
  'grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass'.split(','),
  'grass,grass,grass,grass,grass,grass,grass,grass,grass,flower_r,grass,grass,grass,grass,flower_p,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass'.split(','),
  'grass,grass,flower_y,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,flower_y,grass,grass'.split(','),
  'grass,grass,grass,grass,grass,rock,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,rock,grass,grass,grass,grass,grass,grass'.split(','),
  'grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,flower_r,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass'.split(','),
  'grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass'.split(','),
  'grass,grass,flower_p,grass,grass,grass,grass,grass,path,path,path,path,path,path,path,path,path,grass,grass,grass,grass,grass,flower_p,grass,grass'.split(','),
  'tree_top,tree_top,grass,grass,grass,grass,grass,grass,path,path,path,path,path,path,path,path,path,grass,grass,grass,grass,grass,grass,tree_top,tree_top'.split(','),
];

const ZONE_HEART_FOREST = [
  'heart_leaf,heart_leaf,grass,grass,path,path,path,path,path,path,path,path,path,path,path,path,path,grass,grass,grass,grass,grass,grass,heart_leaf,heart_leaf'.split(','),
  'heart_tree,heart_tree,grass,grass,path,path,path,path,path,path,path,path,path,path,path,path,path,grass,grass,grass,grass,grass,grass,heart_tree,heart_tree'.split(','),
  'grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass'.split(','),
  'grass,grass,heart_leaf,grass,grass,grass,grass,grass,grass,flower_p,grass,grass,grass,grass,flower_p,grass,grass,grass,grass,grass,grass,heart_leaf,grass,grass,grass'.split(','),
  'grass,grass,heart_tree,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,heart_tree,grass,grass,grass'.split(','),
  'grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass'.split(','),
  'grass,grass,grass,grass,grass,grass,heart_leaf,grass,grass,grass,grass,grass,water,water,grass,grass,grass,grass,heart_leaf,grass,grass,grass,grass,grass,grass'.split(','),
  'grass,grass,grass,grass,grass,grass,heart_tree,grass,grass,grass,grass,water,water,water,water,grass,grass,grass,heart_tree,grass,grass,grass,grass,grass,grass'.split(','),
  'grass,grass,grass,flower_p,grass,grass,grass,grass,grass,grass,grass,water,water,water,water,grass,grass,grass,grass,grass,grass,flower_p,grass,grass,grass'.split(','),
  'grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,water,water,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass'.split(','),
  'grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass'.split(','),
  'grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass'.split(','),
  'grass,heart_leaf,grass,grass,grass,grass,grass,grass,grass,flower_p,grass,grass,grass,grass,grass,flower_p,grass,grass,grass,grass,grass,grass,heart_leaf,grass,grass'.split(','),
  'grass,heart_tree,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,heart_tree,grass,grass'.split(','),
  'grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass'.split(','),
  'grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass,grass'.split(','),
];

const ZONES = {
  meadow: { map: ZONE_FLOWER_MEADOW, name: 'Flower Meadow', eggs: ['fire', 'nature'] },
  heart:  { map: ZONE_HEART_FOREST, name: 'Heart Forest', eggs: ['love'] },
};

// Solid tiles players cannot walk through
const SOLID_TILES = ['tree_trunk', 'tree_top', 'heart_tree', 'heart_leaf', 'water', 'rock'];
```

- [ ] **Step 2: Add zone rendering function**

Add to the rendering section:

```javascript
// ============================================================
// RENDERING
// ============================================================
function drawTile(type, col, row) {
  const x = col * TILE;
  const y = (row + UI_ROWS) * TILE;

  if (TILE_COLORS[type]) {
    ctx.fillStyle = TILE_COLORS[type];
    ctx.fillRect(x, y, TILE, TILE);
  }

  // Add detail to certain tiles
  if (type === 'flower_r' || type === 'flower_y' || type === 'flower_p') {
    // Draw grass underneath, flower on top
    ctx.fillStyle = TILE_COLORS.grass;
    ctx.fillRect(x, y, TILE, TILE);
    ctx.fillStyle = TILE_COLORS[type];
    const cx = x + TILE / 2;
    const cy = y + TILE / 2;
    ctx.beginPath();
    ctx.arc(cx, cy, 4, 0, Math.PI * 2);
    ctx.fill();
    ctx.fillStyle = '#fff';
    ctx.beginPath();
    ctx.arc(cx, cy, 1.5, 0, Math.PI * 2);
    ctx.fill();
  }

  if (type === 'bush') {
    ctx.fillStyle = TILE_COLORS.grass;
    ctx.fillRect(x, y, TILE, TILE);
    ctx.fillStyle = TILE_COLORS.bush;
    ctx.beginPath();
    ctx.arc(x + TILE / 2, y + TILE / 2, TILE / 3, 0, Math.PI * 2);
    ctx.fill();
    ctx.fillStyle = '#2ecc71';
    ctx.beginPath();
    ctx.arc(x + TILE / 2 - 3, y + TILE / 2 - 3, TILE / 5, 0, Math.PI * 2);
    ctx.fill();
  }
}

function drawZone(zone) {
  const map = zone.map;
  for (let r = 0; r < map.length; r++) {
    for (let c = 0; c < map[r].length; c++) {
      drawTile(map[r][c], c, r);
    }
  }
}
```

- [ ] **Step 3: Test zone rendering**

In the game loop, replace the clear + placeholder with:

```javascript
// Clear
ctx.fillStyle = '#1a1a2e';
ctx.fillRect(0, 0, CANVAS_W, CANVAS_H);

// Draw current zone
drawZone(ZONES.meadow);
```

Open in browser. You should see the Flower Meadow with green grass, colored flowers, trees at corners, bushes, and a path at the bottom.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add tile-based zone maps and rendering for Flower Meadow and Heart Forest"
```

---

### Task 5: Player Movement & Collision

**Files:**
- Modify: `index.html` (add player state, movement logic)

This task makes both characters move around the zone with collision detection against solid tiles.

- [ ] **Step 1: Add player state to GAME STATE section**

```javascript
const players = [
  {
    name: 'Harper',
    x: 12, y: 7, // tile position (center of meadow)
    sprite: SPRITES.harper,
    color: '#87CEEB',
    keys: { up: 'ArrowUp', down: 'ArrowDown', left: 'ArrowLeft', right: 'ArrowRight' },
    moveTimer: 0,
  },
  {
    name: 'Audrey',
    x: 13, y: 7,
    sprite: SPRITES.audrey,
    color: '#9B59B6',
    keys: { up: 'w', down: 's', left: 'a', right: 'd' },
    moveTimer: 0,
  },
];

// Add to state:
state.currentZone = 'meadow';
state.unlockedZones = ['meadow'];
state.collection = {}; // { fire: 3, nature: 1, ... }
state.dragons = [];    // [{ type, followPlayer, offset }]
```

- [ ] **Step 2: Add movement logic**

```javascript
// ============================================================
// GAME LOGIC
// ============================================================
const MOVE_DELAY = 0.12; // seconds between moves (controls speed)

function isSolid(zoneKey, col, row) {
  const zone = ZONES[zoneKey];
  if (col < 0 || col >= COLS || row < 0 || row >= MAP_ROWS) return true;
  const tile = zone.map[row][col];
  return SOLID_TILES.includes(tile);
}

function isOccupiedByOtherPlayer(playerIndex, col, row) {
  for (let i = 0; i < players.length; i++) {
    if (i !== playerIndex && players[i].x === col && players[i].y === row) return true;
  }
  return false;
}

function updatePlayers(dt) {
  for (let i = 0; i < players.length; i++) {
    const p = players[i];
    p.moveTimer -= dt;
    if (p.moveTimer > 0) continue;

    const input = getPlayerInput(p.keys.up, p.keys.down, p.keys.left, p.keys.right);
    if (input.dx === 0 && input.dy === 0) continue;

    // Move one axis at a time (prefer horizontal)
    let nx = p.x + input.dx;
    let ny = p.y + input.dy;

    // Try combined move first
    if (!isSolid(state.currentZone, nx, ny) && !isOccupiedByOtherPlayer(i, nx, ny)) {
      p.x = nx;
      p.y = ny;
      p.moveTimer = MOVE_DELAY;
    }
    // Try horizontal only
    else if (input.dx !== 0 && !isSolid(state.currentZone, p.x + input.dx, p.y) && !isOccupiedByOtherPlayer(i, p.x + input.dx, p.y)) {
      p.x += input.dx;
      p.moveTimer = MOVE_DELAY;
    }
    // Try vertical only
    else if (input.dy !== 0 && !isSolid(state.currentZone, p.x, p.y + input.dy) && !isOccupiedByOtherPlayer(i, p.x, p.y + input.dy)) {
      p.y += input.dy;
      p.moveTimer = MOVE_DELAY;
    }
  }
}

function drawPlayers() {
  for (const p of players) {
    drawSprite(p.sprite, p.x * TILE, (p.y + UI_ROWS) * TILE, TILE);
  }
}
```

- [ ] **Step 3: Wire movement and rendering into the game loop**

Update the game loop to:

```javascript
function gameLoop(timestamp) {
  const dt = Math.min((timestamp - lastTime) / 1000, 0.1);
  lastTime = timestamp;
  state.time += dt;

  if (state.screen === 'playing') {
    updatePlayers(dt);
  }

  // Clear
  ctx.fillStyle = '#1a1a2e';
  ctx.fillRect(0, 0, CANVAS_W, CANVAS_H);

  if (state.screen === 'playing') {
    drawZone(ZONES[state.currentZone]);
    drawPlayers();
  }

  requestAnimationFrame(gameLoop);
}
```

Temporarily set `state.screen = 'playing'` and `state.mode = 'adventure'` for testing.

- [ ] **Step 4: Test in browser**

Open in browser. Both characters should appear in the meadow. Arrow keys move Harper (light blue). WASD moves Audrey (purple). Characters stop at trees, rocks, and water. They can't overlap each other.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add two-player movement with tile collision detection"
```

---

### Task 6: Zone Transitions

**Files:**
- Modify: `index.html` (add zone transition logic)

Players walk onto the path at the bottom of Flower Meadow to enter Heart Forest (and vice versa at the top of Heart Forest).

- [ ] **Step 1: Add zone connection data**

Add to CONSTANTS:

```javascript
// Zone connections: walking off one edge enters another zone
// { fromZone, edge ('top'|'bottom'), toZone, unlockRequired }
const ZONE_CONNECTIONS = [
  { from: 'meadow', edge: 'bottom', to: 'heart', unlockRequired: true },
  { from: 'heart', edge: 'top', to: 'meadow', unlockRequired: false },
];
```

- [ ] **Step 2: Add transition logic**

Add to GAME LOGIC section:

```javascript
function checkZoneTransition() {
  for (const conn of ZONE_CONNECTIONS) {
    if (state.currentZone !== conn.from) continue;

    if (conn.unlockRequired && !state.unlockedZones.includes(conn.to)) continue;

    for (const p of players) {
      const atEdge =
        (conn.edge === 'bottom' && p.y >= MAP_ROWS - 1) ||
        (conn.edge === 'top' && p.y <= 0);

      if (atEdge) {
        // Check player is on path tile
        const tile = ZONES[conn.from].map[p.y][p.x];
        if (tile === 'path') {
          transitionToZone(conn.to, conn.edge);
          return;
        }
      }
    }
  }
}

function transitionToZone(zoneKey, fromEdge) {
  state.currentZone = zoneKey;

  // Place players at the opposite edge
  for (const p of players) {
    if (fromEdge === 'bottom') {
      p.y = 1; // near top
    } else {
      p.y = MAP_ROWS - 2; // near bottom
    }
  }

  // Spawn eggs for new zone
  spawnEggsForZone(zoneKey);
}
```

- [ ] **Step 3: Add placeholder egg spawning (will be completed in Task 7)**

```javascript
function spawnEggsForZone(zoneKey) {
  // Placeholder -- filled in Task 7
}
```

- [ ] **Step 4: Call checkZoneTransition in game loop**

Add after `updatePlayers(dt)`:

```javascript
checkZoneTransition();
```

- [ ] **Step 5: Test zone transitions**

Temporarily unlock heart zone: `state.unlockedZones = ['meadow', 'heart']`. Walk Harper to the path at the bottom of the meadow. When she reaches the bottom row on a path tile, both players should appear at the top of Heart Forest. Walk back up the path to return to Flower Meadow.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add zone transitions via path tiles at map edges"
```

---

### Task 7: Egg Spawning & Collection

**Files:**
- Modify: `index.html` (egg state, spawning, collection, hatching)

This task places eggs in the world and lets players collect them by walking over them.

- [ ] **Step 1: Add egg state and spawning**

Add to GAME STATE:

```javascript
state.eggs = []; // [{ type, x, y, bobOffset, hatchTimer }]
state.hatching = []; // [{ type, x, y, timer, phase }] active hatch animations
```

Implement `spawnEggsForZone` (replace the placeholder):

```javascript
function spawnEggsForZone(zoneKey) {
  state.eggs = [];
  const zone = ZONES[zoneKey];
  const eggTypes = zone.eggs;

  // Find all grass tiles not near edges, not occupied by players
  const candidates = [];
  for (let r = 2; r < MAP_ROWS - 2; r++) {
    for (let c = 2; c < COLS - 2; c++) {
      const tile = zone.map[r][c];
      if (tile === 'grass' || tile === 'grass2') {
        candidates.push({ x: c, y: r });
      }
    }
  }

  // Shuffle and pick spots
  for (let i = candidates.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [candidates[i], candidates[j]] = [candidates[j], candidates[i]];
  }

  const count = state.mode === 'freeplay' ? 8 : 5;
  for (let i = 0; i < Math.min(count, candidates.length); i++) {
    const type = eggTypes[Math.floor(Math.random() * eggTypes.length)];
    state.eggs.push({
      type: type,
      x: candidates[i].x,
      y: candidates[i].y,
      bobOffset: Math.random() * Math.PI * 2,
    });
  }
}
```

- [ ] **Step 2: Add egg rendering with glow and bob animation**

```javascript
function drawEggs() {
  for (const egg of state.eggs) {
    const bob = Math.sin(state.time * 2 + egg.bobOffset) * 3;
    const px = egg.x * TILE + 4;
    const py = (egg.y + UI_ROWS) * TILE + bob;

    // Glow effect
    const glowAlpha = 0.3 + Math.sin(state.time * 3 + egg.bobOffset) * 0.15;
    ctx.fillStyle = `rgba(255, 255, 200, ${glowAlpha})`;
    ctx.beginPath();
    ctx.arc(egg.x * TILE + TILE / 2, (egg.y + UI_ROWS) * TILE + TILE / 2 + bob, TILE / 2, 0, Math.PI * 2);
    ctx.fill();

    // Draw egg sprite
    drawSprite(EGG_SPRITES[egg.type], px, py, TILE * 0.75);
  }
}
```

- [ ] **Step 3: Add egg collection logic**

```javascript
function checkEggCollection() {
  for (let i = state.eggs.length - 1; i >= 0; i--) {
    const egg = state.eggs[i];
    for (const p of players) {
      if (p.x === egg.x && p.y === egg.y) {
        // Start hatching animation
        state.hatching.push({
          type: egg.type,
          x: egg.x,
          y: egg.y,
          timer: 0,
          phase: 'wobble', // wobble -> crack -> hatch
        });
        state.eggs.splice(i, 1);
        break;
      }
    }
  }
}
```

- [ ] **Step 4: Add hatching animation**

```javascript
function updateHatching(dt) {
  for (let i = state.hatching.length - 1; i >= 0; i--) {
    const h = state.hatching[i];
    h.timer += dt;

    if (h.phase === 'wobble' && h.timer > 0.5) {
      h.phase = 'crack';
      h.timer = 0;
    } else if (h.phase === 'crack' && h.timer > 0.4) {
      h.phase = 'hatch';
      h.timer = 0;
    } else if (h.phase === 'hatch' && h.timer > 0.6) {
      // Hatching complete -- add dragon to collection
      state.collection[h.type] = (state.collection[h.type] || 0) + 1;

      // Add baby dragon follower
      const followPlayer = state.dragons.length % 2; // alternate which player they follow
      state.dragons.push({
        type: h.type,
        followPlayer: followPlayer,
        x: players[followPlayer].x,
        y: players[followPlayer].y,
      });

      state.hatching.splice(i, 1);

      // Check if zone should unlock
      checkUnlocks();

      // Respawn eggs in freeplay
      if (state.mode === 'freeplay' && state.eggs.length < 3) {
        spawnEggsForZone(state.currentZone);
      }
    }
  }
}

function drawHatching() {
  for (const h of state.hatching) {
    const px = h.x * TILE;
    const py = (h.y + UI_ROWS) * TILE;

    if (h.phase === 'wobble') {
      const wobble = Math.sin(h.timer * 20) * 3;
      drawSprite(EGG_SPRITES[h.type], px + 4 + wobble, py, TILE * 0.75);
    } else if (h.phase === 'crack') {
      drawSprite(EGG_SPRITES[h.type], px + 4, py, TILE * 0.75);
      // Draw cracks
      ctx.strokeStyle = '#333';
      ctx.lineWidth = 1;
      ctx.beginPath();
      ctx.moveTo(px + TILE / 2, py + 4);
      ctx.lineTo(px + TILE / 2 + 4, py + TILE / 2);
      ctx.lineTo(px + TILE / 2 - 2, py + TILE - 4);
      ctx.stroke();
    } else if (h.phase === 'hatch') {
      // Sparkle effect
      const sparkleCount = 6;
      for (let s = 0; s < sparkleCount; s++) {
        const angle = (s / sparkleCount) * Math.PI * 2 + h.timer * 5;
        const dist = h.timer * 30;
        const sx = px + TILE / 2 + Math.cos(angle) * dist;
        const sy = py + TILE / 2 + Math.sin(angle) * dist;
        const alpha = 1 - h.timer / 0.6;
        ctx.fillStyle = `rgba(255, 255, 100, ${alpha})`;
        ctx.fillRect(sx - 2, sy - 2, 4, 4);
      }
      // Draw baby dragon emerging
      const scale = h.timer / 0.6;
      drawSprite(DRAGON_SPRITES[h.type], px + 6, py + 4, TILE * 0.6 * scale);
    }
  }
}
```

- [ ] **Step 5: Wire egg logic into game loop**

After `checkZoneTransition()`:

```javascript
checkEggCollection();
updateHatching(dt);
```

After `drawPlayers()`:

```javascript
drawEggs();
drawHatching();
```

Call `spawnEggsForZone('meadow')` when the game starts (after setting screen to 'playing').

- [ ] **Step 6: Test egg collection**

Open in browser. Eggs should appear on the meadow, glowing and bobbing. Walk a character over an egg -- it should wobble, crack, then hatch with sparkles, revealing a baby dragon.

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "feat: add egg spawning, collection, and hatching animation"
```

---

### Task 8: Baby Dragon Followers

**Files:**
- Modify: `index.html` (dragon following logic and rendering)

Baby dragons follow behind the players in a cute bouncing parade.

- [ ] **Step 1: Add dragon following logic**

```javascript
function updateDragons(dt) {
  for (let i = 0; i < state.dragons.length; i++) {
    const d = state.dragons[i];
    const leader = i === 0
      ? players[d.followPlayer]
      : (state.dragons[i - 1].followPlayer === d.followPlayer ? state.dragons[i - 1] : players[d.followPlayer]);

    // Smoothly follow leader with delay
    const dx = leader.x - d.x;
    const dy = leader.y - d.y;
    const dist = Math.abs(dx) + Math.abs(dy);

    if (dist > 1.5) {
      d.x += Math.sign(dx) * dt * 5;
      d.y += Math.sign(dy) * dt * 5;
    }
  }
}

function drawDragons() {
  for (let i = 0; i < state.dragons.length; i++) {
    const d = state.dragons[i];
    const bounce = Math.sin(state.time * 4 + i * 0.8) * 2;
    drawSprite(
      DRAGON_SPRITES[d.type],
      d.x * TILE + 6,
      (d.y + UI_ROWS) * TILE + bounce + 4,
      TILE * 0.6
    );
  }
}
```

- [ ] **Step 2: Wire into game loop**

Add `updateDragons(dt)` after `updateHatching(dt)`.
Draw dragons after drawing players but before drawing eggs (so eggs render on top):

```javascript
drawZone(ZONES[state.currentZone]);
drawDragons();
drawPlayers();
drawEggs();
drawHatching();
```

- [ ] **Step 3: Test dragon followers**

Collect a few eggs. Baby dragons should appear and bounce along behind the players, creating a parade effect. Dragons alternate following Player 1 and Player 2.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add baby dragon followers that parade behind players"
```

---

### Task 9: Collection Tracker UI

**Files:**
- Modify: `index.html` (UI rendering in top rows)

Shows the shared dragon collection at the top of the screen.

- [ ] **Step 1: Add UI rendering**

```javascript
function drawUI() {
  // Background bar
  ctx.fillStyle = '#1a1a2e';
  ctx.fillRect(0, 0, CANVAS_W, UI_ROWS * TILE);

  // Zone name
  ctx.fillStyle = '#fff';
  ctx.font = 'bold 14px monospace';
  ctx.textAlign = 'left';
  ctx.fillText(ZONES[state.currentZone].name, 10, 20);

  // Dragon collection icons
  const allTypes = ['fire', 'nature', 'love'];
  const startX = 10;
  const y = 34;

  for (let i = 0; i < allTypes.length; i++) {
    const type = allTypes[i];
    const count = state.collection[type] || 0;
    const x = startX + i * 80;

    if (count > 0) {
      // Draw mini dragon
      drawSprite(DRAGON_SPRITES[type], x, y - 6, 20);
      ctx.fillStyle = '#fff';
      ctx.font = '12px monospace';
      ctx.textAlign = 'left';
      ctx.fillText('x' + count, x + 24, y + 8);
    } else {
      // Gray silhouette
      ctx.fillStyle = '#444';
      ctx.fillRect(x, y - 4, 16, 16);
      ctx.fillStyle = '#555';
      ctx.font = '12px monospace';
      ctx.fillText('?', x + 4, y + 8);
    }
  }

  // Player labels on right side
  ctx.textAlign = 'right';
  ctx.font = '11px monospace';
  ctx.fillStyle = players[0].color;
  ctx.fillText(players[0].name + ': Arrows', CANVAS_W - 10, 18);
  ctx.fillStyle = players[1].color;
  ctx.fillText(players[1].name + ': WASD', CANVAS_W - 10, 34);

  // Adventure mode: eggs needed to unlock
  if (state.mode === 'adventure') {
    const totalCollected = Object.values(state.collection).reduce((a, b) => a + b, 0);
    const needed = 5;
    ctx.textAlign = 'center';
    ctx.fillStyle = '#aaa';
    ctx.font = '11px monospace';
    if (!state.unlockedZones.includes('heart')) {
      ctx.fillText('Collect ' + totalCollected + '/' + needed + ' eggs to unlock Heart Forest', CANVAS_W / 2, CANVAS_H - 8);
    } else {
      ctx.fillText('Heart Forest unlocked!', CANVAS_W / 2, CANVAS_H - 8);
    }
  }

  // Divider line
  ctx.strokeStyle = '#444';
  ctx.beginPath();
  ctx.moveTo(0, UI_ROWS * TILE);
  ctx.lineTo(CANVAS_W, UI_ROWS * TILE);
  ctx.stroke();
}
```

- [ ] **Step 2: Wire into game loop**

Add `drawUI()` as the last draw call (draws on top of everything):

```javascript
drawZone(ZONES[state.currentZone]);
drawDragons();
drawPlayers();
drawEggs();
drawHatching();
drawUI();
```

- [ ] **Step 3: Test UI**

Open in browser. Top bar should show zone name, dragon collection with counts (or "?" for uncollected), player controls reference, and unlock progress at bottom.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add collection tracker UI with zone name and unlock progress"
```

---

### Task 10: Zone Unlocking

**Files:**
- Modify: `index.html` (unlock logic and sparkle effect)

- [ ] **Step 1: Add unlock checking**

```javascript
function checkUnlocks() {
  if (state.mode !== 'adventure') return;

  const totalCollected = Object.values(state.collection).reduce((a, b) => a + b, 0);

  if (totalCollected >= 5 && !state.unlockedZones.includes('heart')) {
    state.unlockedZones.push('heart');
    // Trigger unlock sparkle effect
    state.unlockEffect = { timer: 0, zone: 'heart' };
  }
}
```

- [ ] **Step 2: Add unlock sparkle animation**

```javascript
function updateUnlockEffect(dt) {
  if (!state.unlockEffect) return;
  state.unlockEffect.timer += dt;
  if (state.unlockEffect.timer > 2) {
    state.unlockEffect = null;
  }
}

function drawUnlockEffect() {
  if (!state.unlockEffect) return;
  const t = state.unlockEffect.timer;
  const alpha = Math.max(0, 1 - t / 2);

  // Sparkle across bottom path
  for (let i = 0; i < 15; i++) {
    const x = (8 + i) * TILE + Math.sin(t * 5 + i) * 5;
    const y = (MAP_ROWS - 1 + UI_ROWS) * TILE + Math.cos(t * 4 + i * 0.5) * 10;
    ctx.fillStyle = `rgba(255, 255, 100, ${alpha * (0.5 + Math.sin(t * 8 + i) * 0.5)})`;
    ctx.fillRect(x - 2, y - 2, 4, 4);
  }

  // Text announcement
  if (t < 1.5) {
    ctx.fillStyle = `rgba(255, 255, 255, ${alpha})`;
    ctx.font = 'bold 18px monospace';
    ctx.textAlign = 'center';
    ctx.fillText('Heart Forest unlocked!', CANVAS_W / 2, CANVAS_H / 2 - 20);
    ctx.font = '13px monospace';
    ctx.fillText('Walk to the path at the bottom!', CANVAS_W / 2, CANVAS_H / 2 + 5);
  }
}
```

- [ ] **Step 3: Wire into game loop**

Add `updateUnlockEffect(dt)` to update section.
Add `drawUnlockEffect()` after `drawUI()`.

- [ ] **Step 4: Test unlock flow**

Set `state.mode = 'adventure'`. Collect 5 eggs in Flower Meadow. A sparkle effect and "Heart Forest unlocked!" message should appear. Walk to the path at the bottom to transition to Heart Forest.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add zone unlock mechanic with sparkle celebration effect"
```

---

### Task 11: Particle System

**Files:**
- Modify: `index.html` (ambient particles per zone)

Simple particles that float around to give each zone personality.

- [ ] **Step 1: Add particle system**

```javascript
// ============================================================
// PARTICLES
// ============================================================
state.particles = [];
const MAX_PARTICLES = 30;

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
  };
  p.maxLife = p.life;

  if (zoneKey === 'meadow') {
    // Floating petals -- drift sideways and down gently
    p.vy = Math.random() * 10 + 5;
    p.vx = (Math.random() - 0.5) * 30;
    p.color = ['#e74c3c', '#f1c40f', '#e84393', '#fff'][Math.floor(Math.random() * 4)];
  } else if (zoneKey === 'heart') {
    // Drifting hearts -- float upward
    p.vy = -(Math.random() * 10 + 5);
    p.vx = (Math.random() - 0.5) * 15;
    p.color = '#ff69b4';
    p.isHeart = true;
  }

  state.particles.push(p);
}

function updateParticles(dt) {
  // Spawn new particles
  if (state.particles.length < MAX_PARTICLES && Math.random() < 0.1) {
    spawnParticle(state.currentZone);
  }

  // Update existing
  for (let i = state.particles.length - 1; i >= 0; i--) {
    const p = state.particles[i];
    p.x += p.vx * dt;
    p.y += p.vy * dt;
    p.life -= dt;

    if (p.life <= 0 || p.y < 0 || p.y > CANVAS_H || p.x < 0 || p.x > CANVAS_W) {
      state.particles.splice(i, 1);
    }
  }
}

function drawParticles() {
  for (const p of state.particles) {
    const alpha = Math.min(1, p.life / (p.maxLife * 0.3));
    ctx.globalAlpha = alpha * 0.6;

    if (p.isHeart) {
      // Draw tiny heart shape
      ctx.fillStyle = p.color;
      ctx.font = (p.size * 3) + 'px serif';
      ctx.fillText('\u2665', p.x, p.y);
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

- [ ] **Step 2: Wire into game loop**

Add `updateParticles(dt)` to update section.
Add `drawParticles()` after `drawZone()` but before `drawDragons()`:

```javascript
drawZone(ZONES[state.currentZone]);
drawParticles();
drawDragons();
drawPlayers();
drawEggs();
drawHatching();
drawUI();
drawUnlockEffect();
```

- [ ] **Step 3: Clear particles on zone transition**

In `transitionToZone()`, add:

```javascript
state.particles = [];
```

- [ ] **Step 4: Test particles**

Open in browser. Flower Meadow should have colorful petals drifting down. Transition to Heart Forest -- pink hearts should float upward.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add ambient particle system with petals and hearts per zone"
```

---

### Task 12: Start Screen & Mode Selection

**Files:**
- Modify: `index.html` (title screen rendering and input)

The start screen lets the players pick Adventure or Free Play mode.

- [ ] **Step 1: Add title screen rendering**

```javascript
function drawTitleScreen() {
  // Background
  ctx.fillStyle = '#1a1a2e';
  ctx.fillRect(0, 0, CANVAS_W, CANVAS_H);

  // Stars background
  for (let i = 0; i < 50; i++) {
    const sx = (Math.sin(i * 127.1) * 0.5 + 0.5) * CANVAS_W;
    const sy = (Math.cos(i * 311.7) * 0.5 + 0.5) * CANVAS_H;
    const twinkle = Math.sin(state.time * 2 + i) * 0.5 + 0.5;
    ctx.fillStyle = `rgba(255, 255, 255, ${twinkle * 0.5})`;
    ctx.fillRect(sx, sy, 2, 2);
  }

  // Title
  ctx.fillStyle = '#FFD700';
  ctx.font = 'bold 36px monospace';
  ctx.textAlign = 'center';
  ctx.fillText('Dragon Egg Island', CANVAS_W / 2, 150);

  // Subtitle
  ctx.fillStyle = '#aaa';
  ctx.font = '14px monospace';
  ctx.fillText('A cooperative dragon collecting adventure', CANVAS_W / 2, 185);

  // Draw characters
  drawSprite(SPRITES.harper, CANVAS_W / 2 - 80, 210, 48);
  drawSprite(SPRITES.audrey, CANVAS_W / 2 + 32, 210, 48);

  // Player names
  ctx.font = '12px monospace';
  ctx.fillStyle = '#87CEEB';
  ctx.fillText('Harper', CANVAS_W / 2 - 56, 275);
  ctx.fillStyle = '#9B59B6';
  ctx.fillText('Audrey', CANVAS_W / 2 + 56, 275);

  // Mode selection
  const adventureSelected = state.titleSelection === 0;
  const freeplaySelected = state.titleSelection === 1;

  // Adventure option
  ctx.fillStyle = adventureSelected ? '#FFD700' : '#666';
  ctx.font = (adventureSelected ? 'bold ' : '') + '20px monospace';
  ctx.fillText(adventureSelected ? '> Adventure <' : 'Adventure', CANVAS_W / 2, 340);
  ctx.font = '11px monospace';
  ctx.fillStyle = adventureSelected ? '#ccc' : '#555';
  ctx.fillText('Explore zones, collect eggs, unlock new areas!', CANVAS_W / 2, 360);

  // Free Play option
  ctx.fillStyle = freeplaySelected ? '#FFD700' : '#666';
  ctx.font = (freeplaySelected ? 'bold ' : '') + '20px monospace';
  ctx.fillText(freeplaySelected ? '> Free Play <' : 'Free Play', CANVAS_W / 2, 410);
  ctx.font = '11px monospace';
  ctx.fillStyle = freeplaySelected ? '#ccc' : '#555';
  ctx.fillText('All zones open, just explore and have fun!', CANVAS_W / 2, 430);

  // Controls hint
  ctx.fillStyle = '#555';
  ctx.font = '11px monospace';
  ctx.fillText('Use Arrow Keys or WASD to select, then press any key to start', CANVAS_W / 2, 510);

  // Egg decorations
  const eggTypes = ['fire', 'nature', 'love'];
  for (let i = 0; i < 3; i++) {
    const bob = Math.sin(state.time * 2 + i * 2) * 4;
    drawSprite(EGG_SPRITES[eggTypes[i]], 100 + i * 40, 480 + bob, 24);
    drawSprite(EGG_SPRITES[eggTypes[i]], CANVAS_W - 180 + i * 40, 480 + bob, 24);
  }
}
```

- [ ] **Step 2: Add title screen input handling**

Add to GAME STATE:

```javascript
state.titleSelection = 0; // 0 = adventure, 1 = freeplay
```

Add title screen input handler:

```javascript
function updateTitleScreen() {
  // Check for up/down to switch selection (from either player)
  if (keys['ArrowUp'] || keys['w']) {
    state.titleSelection = 0;
    keys['ArrowUp'] = false;
    keys['w'] = false;
  }
  if (keys['ArrowDown'] || keys['s']) {
    state.titleSelection = 1;
    keys['ArrowDown'] = false;
    keys['s'] = false;
  }

  // Enter/Space or right arrow to confirm
  if (keys['Enter'] || keys[' '] || keys['ArrowRight'] || keys['d']) {
    startGame(state.titleSelection === 0 ? 'adventure' : 'freeplay');
    keys['Enter'] = false;
    keys[' '] = false;
    keys['ArrowRight'] = false;
    keys['d'] = false;
  }
}

function startGame(mode) {
  state.screen = 'playing';
  state.mode = mode;
  state.currentZone = 'meadow';
  state.collection = {};
  state.dragons = [];
  state.eggs = [];
  state.particles = [];
  state.hatching = [];
  state.unlockEffect = null;

  if (mode === 'freeplay') {
    state.unlockedZones = ['meadow', 'heart'];
  } else {
    state.unlockedZones = ['meadow'];
  }

  // Reset player positions
  players[0].x = 11;
  players[0].y = 7;
  players[1].x = 13;
  players[1].y = 7;

  spawnEggsForZone('meadow');
}
```

- [ ] **Step 3: Update game loop to handle both screens**

```javascript
function gameLoop(timestamp) {
  const dt = Math.min((timestamp - lastTime) / 1000, 0.1);
  lastTime = timestamp;
  state.time += dt;

  if (state.screen === 'title') {
    updateTitleScreen();
    drawTitleScreen();
  } else if (state.screen === 'playing') {
    updatePlayers(dt);
    checkZoneTransition();
    checkEggCollection();
    updateHatching(dt);
    updateDragons(dt);
    updateParticles(dt);
    updateUnlockEffect(dt);

    ctx.fillStyle = '#1a1a2e';
    ctx.fillRect(0, 0, CANVAS_W, CANVAS_H);

    drawZone(ZONES[state.currentZone]);
    drawParticles();
    drawDragons();
    drawPlayers();
    drawEggs();
    drawHatching();
    drawUI();
    drawUnlockEffect();
  }

  requestAnimationFrame(gameLoop);
}
```

Remove the temporary `state.screen = 'playing'` setting. The game should start on the title screen.

- [ ] **Step 4: Test start screen**

Open in browser. Title screen should show with character sprites, mode selection, and decorative eggs. Arrow up/down or W/S switches selection. Enter/Space/Right starts the game.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add title screen with adventure and free play mode selection"
```

---

### Task 13: Final Polish & Integration Test

**Files:**
- Modify: `index.html` (cleanup, final tweaks)

- [ ] **Step 1: Add Harper's snowflake detail**

Harper's character should have a small snowflake accent. Add to `drawPlayers()`:

```javascript
function drawPlayers() {
  for (const p of players) {
    drawSprite(p.sprite, p.x * TILE, (p.y + UI_ROWS) * TILE, TILE);

    // Harper's snowflake accent
    if (p.name === 'Harper') {
      const sx = p.x * TILE + TILE - 6;
      const sy = (p.y + UI_ROWS) * TILE + 2;
      const twinkle = Math.sin(state.time * 3) * 0.3 + 0.7;
      ctx.fillStyle = `rgba(255, 255, 255, ${twinkle})`;
      ctx.font = '8px serif';
      ctx.fillText('\u2744', sx, sy + 8);
    }
  }
}
```

- [ ] **Step 2: Add Audrey's sparkle detail**

```javascript
    // Audrey's sparkle accent
    if (p.name === 'Audrey') {
      const sx = p.x * TILE + TILE - 6;
      const sy = (p.y + UI_ROWS) * TILE + 2;
      const twinkle = Math.sin(state.time * 4) * 0.3 + 0.7;
      ctx.fillStyle = `rgba(200, 150, 255, ${twinkle})`;
      ctx.font = '8px serif';
      ctx.fillText('\u2726', sx, sy + 8);
    }
```

- [ ] **Step 3: Full integration test**

Test the complete game flow:

1. Open `index.html` -- title screen shows with characters and mode options
2. Select "Adventure" with arrows or WASD, press Enter
3. Both characters appear in Flower Meadow
4. Arrow keys move Harper (light blue with snowflake), WASD moves Audrey (purple with sparkle)
5. Characters collide with trees, rocks, and each other
6. Eggs glow and bob on the grass
7. Walk over an egg -- it wobbles, cracks, and hatches with sparkles
8. Baby dragon appears and follows the player
9. Collection tracker updates at top of screen
10. Petals float in the meadow
11. After 5 eggs collected, "Heart Forest unlocked!" appears with sparkle effect
12. Walk to path at bottom to transition to Heart Forest
13. Pink hearts float upward, love eggs appear
14. Walk back up the path to return to Flower Meadow
15. Return to title (refresh page), select "Free Play" -- both zones accessible immediately

- [ ] **Step 4: Final commit**

```bash
git add index.html
git commit -m "feat: add character accents and complete Stage 1 integration"
```

---

## Summary

**13 tasks** building up the game layer by layer:

| Task | What it adds |
|------|-------------|
| 1 | Project setup, canvas, game loop |
| 2 | Pixel art sprite renderer (characters, eggs, dragons) |
| 3 | Dual-player keyboard input |
| 4 | Tile maps and zone rendering |
| 5 | Player movement with collision |
| 6 | Zone transitions via path tiles |
| 7 | Egg spawning, collection, and hatching animation |
| 8 | Baby dragon followers parade |
| 9 | Collection tracker UI |
| 10 | Zone unlock mechanic with celebration |
| 11 | Ambient particle system |
| 12 | Title screen with mode selection |
| 13 | Character accents and integration test |

After all tasks, opening `index.html` in a browser gives Harper and Audrey a complete, playable dragon egg collecting game.
