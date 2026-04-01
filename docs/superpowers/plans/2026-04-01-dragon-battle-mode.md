# Dragon Battle Mode Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a vertical shooter ("Dragon Battle") mode to the existing Dragon Egg Island game where players fly dragons, auto-shoot to cure corrupted enemies in waves, pick from 7 dragon types with unique abilities, and use super moves -- all with the existing 1P/2P/2-iPad player modes.

**Architecture:** Extend the existing `index.html` with new battle-specific sections. Battle mode uses pixel-based movement (not tile-based), its own update/draw loop branch, and reuses the existing player mode, input, d-pad, and Firebase systems. New state lives under `state.battle`. Title screen gets a third game type option.

**Tech Stack:** Same -- vanilla HTML5 Canvas, JavaScript, Firebase for 2-iPad mode. No new dependencies.

---

## File Structure

```
harper-audrey-game/
├── index.html    # Add ~600-800 lines of battle mode code in new sections
```

New code sections to add (with comment banners):

1. **BATTLE CONSTANTS** -- dragon configs, enemy types, wave definitions
2. **BATTLE STATE** -- positions, projectiles, enemies, wave tracking
3. **BATTLE LOGIC** -- movement, shooting, collision, waves, spawning, drops
4. **BATTLE RENDERING** -- draw battlefield, dragons, shots, enemies, HUD, effects
5. **BATTLE UI** -- super button HTML overlay, dragon select screen

The game loop already branches on `state.screen`. Battle mode adds `'dragonSelect'` and `'battle'` screen states.

---

### Task 1: Battle Constants & Dragon Configs

**Files:**
- Modify: `index.html` (add new section after PARTICLES, before CANVAS SETUP)

Define all dragon types, enemy types, and wave configurations.

- [ ] **Step 1: Add BATTLE CONSTANTS section**

Add after the PARTICLES section and before CANVAS SETUP:

```javascript
// ============================================================
// BATTLE CONSTANTS
// ============================================================
const DRAGON_CONFIGS = {
  fire:    { name: 'Fire Dragon',    color: '#FF4500', shotSpeed: 500, shotRate: 4, shotType: 'single',  superName: 'Flame Wave' },
  ice:     { name: 'Ice Dragon',     color: '#4FC3F7', shotSpeed: 400, shotRate: 3.5, shotType: 'spread', superName: 'Freeze Blast' },
  nature:  { name: 'Nature Dragon',  color: '#43A047', shotSpeed: 400, shotRate: 3.5, shotType: 'pierce', superName: 'Vine Burst' },
  love:    { name: 'Love Dragon',    color: '#E91E63', shotSpeed: 300, shotRate: 3, shotType: 'wide',    superName: 'Heal Wave' },
  shadow:  { name: 'Shadow Dragon',  color: '#5E35B1', shotSpeed: 500, shotRate: 4.5, shotType: 'single', superName: 'Shadow Cloak' },
  star:    { name: 'Star Dragon',    color: '#FFC107', shotSpeed: 400, shotRate: 3.5, shotType: 'bounce', superName: 'Supernova' },
  rainbow: { name: 'Rainbow Dragon', color: '#FF6B6B', shotSpeed: 350, shotRate: 6, shotType: 'beam',    superName: 'Rainbow Wave' },
};

const ENEMY_TYPES = {
  // Wave 1-5: Cute & Easy
  shadowPuff:     { name: 'Shadow Puff',     hp: 1, speed: 40,  points: 10, color: '#9575CD', size: 24, pattern: 'straight' },
  corruptedEgg:   { name: 'Corrupted Egg',   hp: 1, speed: 35,  points: 10, color: '#7E57C2', size: 20, pattern: 'wobble' },
  darkButterfly:  { name: 'Dark Butterfly',   hp: 1, speed: 45,  points: 15, color: '#AB47BC', size: 18, pattern: 'zigzag' },
  // Wave 6-10: Medium
  shadowSlime:    { name: 'Shadow Slime',     hp: 2, speed: 50,  points: 25, color: '#7B1FA2', size: 22, pattern: 'bounce' },
  corruptedDragon:{ name: 'Corrupted Dragon', hp: 2, speed: 55,  points: 30, color: '#6A1B9A', size: 24, pattern: 'swoop' },
  darkMushroom:   { name: 'Dark Mushroom',    hp: 1, speed: 45,  points: 20, color: '#8E24AA', size: 20, pattern: 'straight', splits: true },
  // Wave 11+: Harder
  shadowKnight:   { name: 'Shadow Knight',    hp: 3, speed: 35,  points: 40, color: '#4A148C', size: 26, pattern: 'formation' },
  stormCloud:     { name: 'Storm Cloud',      hp: 4, speed: 25,  points: 50, color: '#311B92', size: 32, pattern: 'straight', shoots: true },
  // Mini-boss
  elderDragon:    { name: 'Elder Dragon',     hp: 10, speed: 30, points: 200, color: '#1A0033', size: 48, pattern: 'boss' },
};

const WAVE_DEFS = [
  // Wave 1-5: easy
  { enemies: ['shadowPuff','shadowPuff','shadowPuff'] },
  { enemies: ['shadowPuff','corruptedEgg','shadowPuff','corruptedEgg'] },
  { enemies: ['darkButterfly','darkButterfly','shadowPuff','corruptedEgg'] },
  { enemies: ['shadowPuff','darkButterfly','corruptedEgg','darkButterfly','shadowPuff'] },
  { enemies: ['elderDragon'] }, // mini-boss wave 5
  // Wave 6-10: medium
  { enemies: ['shadowSlime','shadowSlime','shadowPuff','shadowPuff','darkButterfly'] },
  { enemies: ['corruptedDragon','corruptedDragon','shadowSlime','shadowPuff'] },
  { enemies: ['darkMushroom','darkMushroom','shadowSlime','corruptedDragon'] },
  { enemies: ['shadowSlime','corruptedDragon','darkMushroom','shadowSlime','darkButterfly','darkButterfly'] },
  { enemies: ['elderDragon'] }, // mini-boss wave 10
  // Wave 11+: harder (we generate these dynamically after wave 10)
];

const BATTLE_PLAYER_SPEED = 200;
const BATTLE_PLAYER_Y = CANVAS_H - 80; // dragon position near bottom
const SHOT_SPEED = 400;
const MAX_HEARTS = 5;
const RESPAWN_TIME = 2;
const INVINCIBLE_TIME = 3;
const SUPER_METER_MAX = 100;

const DROP_TYPES = {
  heart:      { color: '#FF5252', chance: 0.30, symbol: '\u2665' },
  superBoost: { color: '#FFD700', chance: 0.15, symbol: '\u2605' },
  speedStar:  { color: '#00E5FF', chance: 0.10, symbol: '\u2736' },
  shield:     { color: '#69F0AE', chance: 0.10, symbol: '\u25CE' },
};
```

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "feat: add battle mode constants -- dragon configs, enemy types, wave definitions"
```

---

### Task 2: Battle State & Dragon Select Screen

**Files:**
- Modify: `index.html` (GAME STATE section + TITLE SCREEN section)

Add battle state initialization and dragon selection screen.

- [ ] **Step 1: Add battle state**

Add to the GAME STATE section, after the existing state properties:

```javascript
state.battle = {
  wave: 0,
  enemies: [],       // [{ type, x, y, vx, vy, hp, maxHp, timer, curing, cureTimer }]
  shots: [],         // [{ x, y, vx, vy, type, playerId, pierce }]
  drops: [],         // [{ type, x, y, vy }]
  effects: [],       // [{ type, x, y, timer, maxTimer, ... }]
  waveState: 'waiting', // 'waiting', 'active', 'clearing', 'bossIntro'
  waveTimer: 0,
  playerDragons: [null, null], // dragon type keys chosen by each player
  battlePlayers: [
    { x: CANVAS_W / 2 - 40, hearts: MAX_HEARTS, superMeter: 0, shotTimer: 0, respawnTimer: 0, invTimer: 0, shieldActive: false, speedBoost: 0, alive: true },
    { x: CANVAS_W / 2 + 40, hearts: MAX_HEARTS, superMeter: 0, shotTimer: 0, respawnTimer: 0, invTimer: 0, shieldActive: false, speedBoost: 0, alive: true },
  ],
  hitStreak: [0, 0],  // hits taken this wave per player (for secret easy mode)
  superActive: [null, null], // active super move per player { type, timer }
};

state.dragonSelectPlayer = 0; // which player is selecting
state.dragonSelectIndex = 0;  // cursor position in dragon list
```

- [ ] **Step 2: Add "Dragon Battle" to title screen game type selection**

In the `drawTitleScreen` function, find the `gametype` step rendering. Add a third option "Dragon Battle". Update `state.titleStep === 'gametype'` rendering to show 3 options:

```javascript
// In the gametype step drawing code:
const options = ['Adventure', 'Free Play', 'Dragon Battle'];
const descriptions = [
  'Explore zones, collect eggs, unlock new areas!',
  'All zones open, just explore and have fun!',
  'Fly your dragon and cure corrupted creatures!'
];
```

Update `updateTitleScreen()` for the gametype step to allow selection index 0-2. When index 2 is confirmed, set `state.screen = 'dragonSelect'`.

Update the touch buttons: add a `btn-battle` button to `#title-touch-btns`.

- [ ] **Step 3: Add dragon select screen rendering**

```javascript
function drawDragonSelect() {
  ctx.fillStyle = '#1a1a2e';
  ctx.fillRect(0, 0, CANVAS_W, CANVAS_H);

  // Stars background (same as title)
  for (let i = 0; i < 50; i++) {
    const sx = (Math.sin(i * 127.1) * 0.5 + 0.5) * CANVAS_W;
    const sy = (Math.cos(i * 311.7) * 0.5 + 0.5) * CANVAS_H;
    const twinkle = Math.sin(state.time * 2 + i) * 0.5 + 0.5;
    ctx.fillStyle = `rgba(255, 255, 255, ${twinkle * 0.5})`;
    ctx.fillRect(sx, sy, 2, 2);
  }

  const currentPlayer = state.dragonSelectPlayer;
  const playerName = currentPlayer === 0 ? players[0].name : players[1].name;

  ctx.fillStyle = '#FFD700';
  ctx.font = 'bold 24px monospace';
  ctx.textAlign = 'center';
  ctx.fillText(playerName + ', choose your dragon!', CANVAS_W / 2, 60);

  // Draw dragon options in a row
  const dragonKeys = Object.keys(DRAGON_CONFIGS);
  const startX = CANVAS_W / 2 - (dragonKeys.length * 50) / 2;

  for (let i = 0; i < dragonKeys.length; i++) {
    const key = dragonKeys[i];
    const config = DRAGON_CONFIGS[key];
    const x = startX + i * 50 + 25;
    const y = 150;
    const selected = i === state.dragonSelectIndex;

    // Highlight box
    if (selected) {
      ctx.strokeStyle = '#FFD700';
      ctx.lineWidth = 2;
      ctx.strokeRect(x - 22, y - 22, 44, 44);
    }

    // Draw dragon sprite
    drawSprite(DRAGON_SPRITES[key], x - 15, y - 15, 30);

    // Name below
    ctx.fillStyle = selected ? '#FFD700' : '#888';
    ctx.font = '9px monospace';
    ctx.fillText(config.name.split(' ')[0], x, y + 30);
  }

  // Selected dragon details
  const selKey = dragonKeys[state.dragonSelectIndex];
  const selConfig = DRAGON_CONFIGS[selKey];

  // Large preview
  drawSprite(DRAGON_SPRITES[selKey], CANVAS_W / 2 - 40, 220, 80);

  ctx.fillStyle = selConfig.color;
  ctx.font = 'bold 18px monospace';
  ctx.fillText(selConfig.name, CANVAS_W / 2, 330);

  ctx.fillStyle = '#ccc';
  ctx.font = '13px monospace';
  ctx.fillText('Shot: ' + selConfig.shotType, CANVAS_W / 2, 355);
  ctx.fillText('Super: ' + selConfig.superName, CANVAS_W / 2, 375);

  // Hint
  ctx.fillStyle = '#555';
  ctx.font = '11px monospace';
  ctx.fillText('Left/Right to browse, Enter to select', CANVAS_W / 2, CANVAS_H - 40);

  // Show player 1's choice if selecting player 2
  if (currentPlayer === 1) {
    const p1Key = state.battle.playerDragons[0];
    ctx.fillStyle = '#666';
    ctx.font = '11px monospace';
    ctx.fillText(players[0].name + ' chose: ' + DRAGON_CONFIGS[p1Key].name, CANVAS_W / 2, CANVAS_H - 60);
  }
}
```

- [ ] **Step 4: Add dragon select input handling**

```javascript
function updateDragonSelect() {
  const dragonKeys = Object.keys(DRAGON_CONFIGS);

  if (keys['ArrowLeft'] || keys['a']) {
    state.dragonSelectIndex = (state.dragonSelectIndex - 1 + dragonKeys.length) % dragonKeys.length;
    keys['ArrowLeft'] = false;
    keys['a'] = false;
  }
  if (keys['ArrowRight'] || keys['d']) {
    state.dragonSelectIndex = (state.dragonSelectIndex + 1) % dragonKeys.length;
    keys['ArrowRight'] = false;
    keys['d'] = false;
  }
  if (keys['Enter'] || keys[' ']) {
    const selKey = dragonKeys[state.dragonSelectIndex];
    state.battle.playerDragons[state.dragonSelectPlayer] = selKey;
    keys['Enter'] = false;
    keys[' '] = false;

    if (state.activePlayerCount === 1 || state.dragonSelectPlayer === 1) {
      // All players have selected, start battle
      startBattle();
    } else {
      // Player 2 selects next
      state.dragonSelectPlayer = 1;
      state.dragonSelectIndex = 0;
    }
  }
  if (keys['Escape'] || keys['ArrowLeft'] && state.dragonSelectPlayer === 0) {
    // Go back to title -- only if haven't started selecting
  }
}
```

- [ ] **Step 5: Add startBattle function**

```javascript
function startBattle() {
  state.screen = 'battle';
  state.battle.wave = 0;
  state.battle.enemies = [];
  state.battle.shots = [];
  state.battle.drops = [];
  state.battle.effects = [];
  state.battle.waveState = 'waiting';
  state.battle.waveTimer = 0;
  state.battle.hitStreak = [0, 0];
  state.battle.superActive = [null, null];

  // Reset battle players
  const centerX = CANVAS_W / 2;
  state.battle.battlePlayers[0] = {
    x: state.activePlayerCount === 2 ? centerX - 40 : centerX,
    hearts: MAX_HEARTS, superMeter: 0, shotTimer: 0,
    respawnTimer: 0, invTimer: 0, shieldActive: false, speedBoost: 0, alive: true,
  };
  state.battle.battlePlayers[1] = {
    x: centerX + 40,
    hearts: MAX_HEARTS, superMeter: 0, shotTimer: 0,
    respawnTimer: 0, invTimer: 0, shieldActive: false, speedBoost: 0, alive: true,
  };
}
```

- [ ] **Step 6: Wire into game loop**

Add to the game loop:

```javascript
} else if (state.screen === 'dragonSelect') {
  updateDragonSelect();
  drawDragonSelect();
} else if (state.screen === 'battle') {
  // Will be filled in next tasks
  ctx.fillStyle = '#1a1a2e';
  ctx.fillRect(0, 0, CANVAS_W, CANVAS_H);
  ctx.fillStyle = '#fff';
  ctx.font = '24px monospace';
  ctx.textAlign = 'center';
  ctx.fillText('Battle mode starting...', CANVAS_W / 2, CANVAS_H / 2);
}
```

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "feat: add dragon select screen and battle mode title option"
```

---

### Task 3: Battle Player Movement & Auto-Shoot

**Files:**
- Modify: `index.html` (add BATTLE LOGIC and BATTLE RENDERING sections)

Players move left/right and auto-fire upward.

- [ ] **Step 1: Add battle player update logic**

Add a new BATTLE LOGIC section:

```javascript
// ============================================================
// BATTLE LOGIC
// ============================================================
function updateBattlePlayers(dt) {
  const activePlayers = getActivePlayers();

  for (let i = 0; i < activePlayers.length; i++) {
    const p = activePlayers[i];
    const pi = state.activePlayerCount === 1 ? state.activePlayer : i;
    const bp = state.battle.battlePlayers[pi];

    if (!bp.alive) {
      bp.respawnTimer -= dt;
      if (bp.respawnTimer <= 0) {
        bp.alive = true;
        bp.hearts = MAX_HEARTS;
        bp.invTimer = INVINCIBLE_TIME;
      }
      continue;
    }

    // Invincibility timer
    if (bp.invTimer > 0) bp.invTimer -= dt;
    if (bp.speedBoost > 0) bp.speedBoost -= dt;

    // Movement
    const input = getPlayerInput(p.keys.up, p.keys.down, p.keys.left, p.keys.right);
    const speed = bp.speedBoost > 0 ? BATTLE_PLAYER_SPEED * 1.5 : BATTLE_PLAYER_SPEED;
    bp.x += input.dx * speed * dt;
    bp.x = Math.max(20, Math.min(CANVAS_W - 20, bp.x));

    // Auto-shoot
    const dragonKey = state.battle.playerDragons[pi];
    const config = DRAGON_CONFIGS[dragonKey];
    bp.shotTimer -= dt;
    if (bp.shotTimer <= 0) {
      bp.shotTimer = 1 / config.shotRate;
      spawnShots(pi, bp.x, BATTLE_PLAYER_Y, config);
    }
  }
}

function spawnShots(playerId, x, y, config) {
  const baseShot = { playerId, vy: -config.shotSpeed, type: config.shotType, color: config.color };

  if (config.shotType === 'single') {
    state.battle.shots.push({ ...baseShot, x, y, vx: 0, pierce: false });
  } else if (config.shotType === 'spread') {
    state.battle.shots.push({ ...baseShot, x: x - 8, y, vx: -30, pierce: false });
    state.battle.shots.push({ ...baseShot, x, y, vx: 0, pierce: false });
    state.battle.shots.push({ ...baseShot, x: x + 8, y, vx: 30, pierce: false });
  } else if (config.shotType === 'pierce') {
    state.battle.shots.push({ ...baseShot, x, y, vx: 0, pierce: true });
  } else if (config.shotType === 'wide') {
    state.battle.shots.push({ ...baseShot, x, y, vx: 0, pierce: false, wide: true });
  } else if (config.shotType === 'bounce') {
    state.battle.shots.push({ ...baseShot, x, y, vx: 0, pierce: false, bounces: 2 });
  } else if (config.shotType === 'beam') {
    state.battle.shots.push({ ...baseShot, x, y, vx: 0, pierce: true, beam: true });
  }
}

function updateShots(dt) {
  for (let i = state.battle.shots.length - 1; i >= 0; i--) {
    const s = state.battle.shots[i];
    s.x += (s.vx || 0) * dt;
    s.y += s.vy * dt;

    // Bounce off edges
    if (s.bounces && s.bounces > 0) {
      if (s.x <= 5 || s.x >= CANVAS_W - 5) {
        s.vx = -(s.vx || 50);
        s.bounces--;
      }
    }

    // Remove if off screen
    if (s.y < -10 || s.y > CANVAS_H + 10 || s.x < -10 || s.x > CANVAS_W + 10) {
      state.battle.shots.splice(i, 1);
    }
  }
}
```

- [ ] **Step 2: Add battle rendering**

```javascript
// ============================================================
// BATTLE RENDERING
// ============================================================
function drawBattleBackground() {
  ctx.fillStyle = '#0a0a1a';
  ctx.fillRect(0, 0, CANVAS_W, CANVAS_H);

  // Scrolling stars
  for (let i = 0; i < 60; i++) {
    const sx = (Math.sin(i * 127.1) * 0.5 + 0.5) * CANVAS_W;
    const sy = ((Math.cos(i * 311.7) * 0.5 + 0.5) * CANVAS_H + state.time * 20 * ((i % 3) + 1)) % CANVAS_H;
    const brightness = 0.3 + (i % 3) * 0.2;
    ctx.fillStyle = `rgba(255, 255, 255, ${brightness})`;
    ctx.fillRect(sx, sy, (i % 3 === 0) ? 2 : 1, (i % 3 === 0) ? 2 : 1);
  }
}

function drawBattlePlayers() {
  for (let pi = 0; pi < state.activePlayerCount; pi++) {
    const playerIdx = state.activePlayerCount === 1 ? state.activePlayer : pi;
    const bp = state.battle.battlePlayers[playerIdx];
    const dragonKey = state.battle.playerDragons[playerIdx];

    if (!bp.alive) {
      // Dizzy stars animation
      const t = bp.respawnTimer;
      for (let s = 0; s < 3; s++) {
        const angle = state.time * 5 + s * (Math.PI * 2 / 3);
        const sx = bp.x + Math.cos(angle) * 15;
        const sy = BATTLE_PLAYER_Y + Math.sin(angle) * 10 - 10;
        ctx.fillStyle = '#FFD700';
        ctx.font = '10px serif';
        ctx.textAlign = 'center';
        ctx.fillText('\u2605', sx, sy);
      }
      continue;
    }

    // Invincibility flash
    if (bp.invTimer > 0 && Math.floor(state.time * 10) % 2 === 0) continue;

    // Shield bubble
    if (bp.shieldActive) {
      ctx.strokeStyle = 'rgba(105, 240, 174, 0.5)';
      ctx.lineWidth = 2;
      ctx.beginPath();
      ctx.arc(bp.x, BATTLE_PLAYER_Y + 5, 22, 0, Math.PI * 2);
      ctx.stroke();
    }

    // Dragon sprite (scaled up)
    drawSprite(DRAGON_SPRITES[dragonKey], bp.x - 18, BATTLE_PLAYER_Y - 12, 36);
  }
}

function drawBattleShots() {
  for (const s of state.battle.shots) {
    const config = Object.values(DRAGON_CONFIGS).find(c => c.color === s.color) || {};

    if (s.beam) {
      // Continuous beam
      ctx.fillStyle = s.color;
      ctx.globalAlpha = 0.6;
      ctx.fillRect(s.x - 2, s.y, 4, -20);
      ctx.globalAlpha = 1;
      ctx.fillRect(s.x - 1, s.y, 2, -16);
    } else if (s.wide) {
      // Wide heart shot
      ctx.fillStyle = s.color;
      ctx.font = '12px serif';
      ctx.textAlign = 'center';
      ctx.fillText('\u2665', s.x, s.y);
    } else {
      // Standard projectile
      ctx.fillStyle = s.color;
      ctx.beginPath();
      ctx.arc(s.x, s.y, 3, 0, Math.PI * 2);
      ctx.fill();
      // Trail
      ctx.fillStyle = s.color;
      ctx.globalAlpha = 0.3;
      ctx.beginPath();
      ctx.arc(s.x, s.y + 6, 2, 0, Math.PI * 2);
      ctx.fill();
      ctx.globalAlpha = 1;
    }
  }
}
```

- [ ] **Step 3: Wire into game loop**

Replace the battle placeholder in the game loop:

```javascript
} else if (state.screen === 'battle') {
  if (!state.isGuest) {
    updateBattlePlayers(dt);
    updateShots(dt);
  }

  drawBattleBackground();
  drawBattleShots();
  drawBattlePlayers();
}
```

- [ ] **Step 4: Test**

Open in browser. Select Dragon Battle from title, pick a dragon, enter battle. You should see a starfield background with a dragon at the bottom that moves left/right and auto-fires colored projectiles upward. No enemies yet.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add battle player movement, auto-shoot, and rendering"
```

---

### Task 4: Enemy Spawning, Movement & Collision

**Files:**
- Modify: `index.html` (BATTLE LOGIC section)

Enemies spawn in waves, move in patterns, and can be hit by player shots.

- [ ] **Step 1: Add wave spawning**

```javascript
function startNextWave() {
  state.battle.wave++;
  state.battle.waveState = 'active';
  state.battle.waveTimer = 0;
  state.battle.hitStreak = [0, 0];

  const waveNum = state.battle.wave;
  let enemiesToSpawn;

  if (waveNum <= WAVE_DEFS.length) {
    enemiesToSpawn = WAVE_DEFS[waveNum - 1].enemies;
  } else {
    // Generate dynamically for waves beyond defined ones
    enemiesToSpawn = generateWave(waveNum);
  }

  // Spawn enemies across the top
  const spacing = CANVAS_W / (enemiesToSpawn.length + 1);
  for (let i = 0; i < enemiesToSpawn.length; i++) {
    const typeName = enemiesToSpawn[i];
    const typeConfig = ENEMY_TYPES[typeName];
    state.battle.enemies.push({
      type: typeName,
      x: spacing * (i + 1),
      y: -typeConfig.size - (i * 20), // stagger entry
      hp: typeConfig.hp,
      maxHp: typeConfig.hp,
      speed: typeConfig.speed * (1 + (waveNum - 1) * 0.03), // 3% faster per wave
      size: typeConfig.size,
      color: typeConfig.color,
      pattern: typeConfig.pattern,
      timer: Math.random() * Math.PI * 2,
      curing: false,
      cureTimer: 0,
      shoots: typeConfig.shoots || false,
      splits: typeConfig.splits || false,
      shootTimer: 0,
    });
  }
}

function generateWave(waveNum) {
  const enemies = [];
  const count = Math.min(5 + Math.floor(waveNum / 2), 12);
  const isBoss = waveNum % 5 === 0;

  if (isBoss) {
    return ['elderDragon'];
  }

  const pool = waveNum < 15
    ? ['shadowSlime', 'corruptedDragon', 'darkMushroom', 'shadowKnight']
    : ['shadowSlime', 'corruptedDragon', 'darkMushroom', 'shadowKnight', 'stormCloud'];

  for (let i = 0; i < count; i++) {
    enemies.push(pool[Math.floor(Math.random() * pool.length)]);
  }
  return enemies;
}
```

- [ ] **Step 2: Add enemy movement patterns**

```javascript
function updateEnemies(dt) {
  for (let i = state.battle.enemies.length - 1; i >= 0; i--) {
    const e = state.battle.enemies[i];
    e.timer += dt;

    // Curing animation
    if (e.curing) {
      e.cureTimer += dt;
      if (e.cureTimer > 0.8) {
        // Drop items
        spawnDrop(e.x, e.y);
        // Add cure effect
        state.battle.effects.push({ type: 'cure', x: e.x, y: e.y, timer: 0, maxTimer: 0.5 });
        // Add super meter to all active players
        for (let pi = 0; pi < state.activePlayerCount; pi++) {
          const idx = state.activePlayerCount === 1 ? state.activePlayer : pi;
          state.battle.battlePlayers[idx].superMeter = Math.min(SUPER_METER_MAX,
            state.battle.battlePlayers[idx].superMeter + 8);
        }
        state.battle.enemies.splice(i, 1);
        continue;
      }
      continue; // don't move while curing
    }

    // Movement patterns
    const baseSpeed = e.speed;
    switch (e.pattern) {
      case 'straight':
        e.y += baseSpeed * dt;
        break;
      case 'wobble':
        e.y += baseSpeed * dt;
        e.x += Math.sin(e.timer * 3) * 30 * dt;
        break;
      case 'zigzag':
        e.y += baseSpeed * dt;
        e.x += Math.sin(e.timer * 4) * 60 * dt;
        break;
      case 'bounce':
        e.y += baseSpeed * dt;
        e.x += Math.cos(e.timer * 2) * 80 * dt;
        if (e.x < 20) e.x = 20;
        if (e.x > CANVAS_W - 20) e.x = CANVAS_W - 20;
        break;
      case 'swoop':
        e.y += baseSpeed * dt * 0.7;
        e.x += Math.sin(e.timer * 1.5) * 100 * dt;
        break;
      case 'formation':
        e.y += baseSpeed * dt;
        e.x += Math.sin(e.timer * 2 + i * 0.5) * 40 * dt;
        break;
      case 'boss':
        // Sweep left-right, stay near top
        if (e.y < 80) {
          e.y += baseSpeed * dt;
        } else {
          e.x += Math.sin(e.timer * 0.8) * 120 * dt;
        }
        break;
    }

    // Clamp to screen
    e.x = Math.max(e.size / 2, Math.min(CANVAS_W - e.size / 2, e.x));

    // Enemy shooting (Storm Clouds only)
    if (e.shoots) {
      e.shootTimer -= dt;
      if (e.shootTimer <= 0) {
        e.shootTimer = 3; // shoot every 3 seconds
        state.battle.shots.push({
          x: e.x, y: e.y + e.size / 2,
          vx: 0, vy: 100,
          type: 'enemy', color: '#7C4DFF', playerId: -1,
          pierce: false, isEnemy: true,
        });
      }
    }

    // Remove if off bottom of screen
    if (e.y > CANVAS_H + 50) {
      state.battle.enemies.splice(i, 1);
    }
  }
}
```

- [ ] **Step 3: Add shot-enemy collision**

```javascript
function checkBattleCollisions() {
  // Shots hitting enemies
  for (let si = state.battle.shots.length - 1; si >= 0; si--) {
    const s = state.battle.shots[si];
    if (s.isEnemy) continue; // enemy shots don't hit enemies

    for (let ei = state.battle.enemies.length - 1; ei >= 0; ei--) {
      const e = state.battle.enemies[ei];
      if (e.curing) continue;

      const hitW = s.wide ? 16 : 6;
      const dx = Math.abs(s.x - e.x);
      const dy = Math.abs(s.y - e.y);

      if (dx < (e.size / 2 + hitW / 2) && dy < (e.size / 2 + 6)) {
        // Hit!
        e.hp--;
        state.battle.effects.push({ type: 'hit', x: s.x, y: s.y, timer: 0, maxTimer: 0.2 });

        if (e.hp <= 0) {
          // Start cure animation
          e.curing = true;
          e.cureTimer = 0;

          // Split mushrooms
          if (e.splits) {
            for (let si2 = 0; si2 < 2; si2++) {
              state.battle.enemies.push({
                type: e.type, x: e.x + (si2 === 0 ? -15 : 15), y: e.y,
                hp: 1, maxHp: 1, speed: e.speed * 1.2, size: e.size * 0.7,
                color: e.color, pattern: 'wobble', timer: Math.random() * Math.PI * 2,
                curing: false, cureTimer: 0, shoots: false, splits: false, shootTimer: 0,
              });
            }
          }
        }

        // Remove shot unless it pierces
        if (!s.pierce) {
          state.battle.shots.splice(si, 1);
        }
        break;
      }
    }
  }

  // Enemies/enemy shots hitting players
  for (let pi = 0; pi < state.activePlayerCount; pi++) {
    const idx = state.activePlayerCount === 1 ? state.activePlayer : pi;
    const bp = state.battle.battlePlayers[idx];
    if (!bp.alive || bp.invTimer > 0) continue;

    // Check enemy body collision
    for (const e of state.battle.enemies) {
      if (e.curing) continue;
      const dx = Math.abs(bp.x - e.x);
      const dy = Math.abs(BATTLE_PLAYER_Y - e.y);
      if (dx < (e.size / 2 + 12) && dy < (e.size / 2 + 12)) {
        hitBattlePlayer(idx);
        break;
      }
    }

    // Check enemy projectiles
    for (let si = state.battle.shots.length - 1; si >= 0; si--) {
      const s = state.battle.shots[si];
      if (!s.isEnemy) continue;
      const dx = Math.abs(bp.x - s.x);
      const dy = Math.abs(BATTLE_PLAYER_Y - s.y);
      if (dx < 14 && dy < 14) {
        hitBattlePlayer(idx);
        state.battle.shots.splice(si, 1);
        break;
      }
    }
  }
}

function hitBattlePlayer(playerIdx) {
  const bp = state.battle.battlePlayers[playerIdx];

  if (bp.shieldActive) {
    bp.shieldActive = false;
    bp.invTimer = 0.5;
    state.battle.effects.push({ type: 'shieldBreak', x: bp.x, y: BATTLE_PLAYER_Y, timer: 0, maxTimer: 0.3 });
    return;
  }

  bp.hearts--;
  bp.invTimer = 1;
  state.battle.hitStreak[playerIdx]++;

  state.battle.effects.push({ type: 'playerHit', x: bp.x, y: BATTLE_PLAYER_Y, timer: 0, maxTimer: 0.3 });

  if (bp.hearts <= 0) {
    bp.alive = false;
    bp.respawnTimer = RESPAWN_TIME;
  }
}
```

- [ ] **Step 4: Add wave management to game loop**

```javascript
function updateBattleWaves(dt) {
  if (state.battle.waveState === 'waiting') {
    state.battle.waveTimer += dt;
    if (state.battle.waveTimer > 2) {
      startNextWave();
    }
  } else if (state.battle.waveState === 'active') {
    if (state.battle.enemies.length === 0) {
      state.battle.waveState = 'clearing';
      state.battle.waveTimer = 0;
    }
  } else if (state.battle.waveState === 'clearing') {
    state.battle.waveTimer += dt;
    if (state.battle.waveTimer > 2) {
      state.battle.waveState = 'waiting';
      state.battle.waveTimer = 0;
    }
  }

  // Secret easy mode: if hit 3+ times this wave, boost drops and slow enemies
  for (let pi = 0; pi < state.activePlayerCount; pi++) {
    const idx = state.activePlayerCount === 1 ? state.activePlayer : pi;
    if (state.battle.hitStreak[idx] >= 3) {
      // Already applied for this wave
    }
  }
}
```

- [ ] **Step 5: Update game loop battle branch**

```javascript
} else if (state.screen === 'battle') {
  if (!state.isGuest) {
    updateBattlePlayers(dt);
    updateShots(dt);
    updateEnemies(dt);
    checkBattleCollisions();
    updateBattleWaves(dt);
    updateBattleEffects(dt);
    updateDrops(dt);
  }

  drawBattleBackground();
  drawBattleEnemies();
  drawBattleShots();
  drawBattlePlayers();
  drawDrops();
  drawBattleEffects();
  drawBattleHUD();
}
```

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add enemy spawning, movement patterns, wave system, and collision"
```

---

### Task 5: Enemy Rendering & Cure Animation

**Files:**
- Modify: `index.html` (BATTLE RENDERING section)

- [ ] **Step 1: Add enemy rendering**

```javascript
function drawBattleEnemies() {
  for (const e of state.battle.enemies) {
    if (e.curing) {
      // Cure animation: flash white, then bright, then happy face flies up
      const t = e.cureTimer / 0.8;
      ctx.globalAlpha = 1 - t * 0.5;

      // Flash bright colors
      const hue = (t * 360) % 360;
      ctx.fillStyle = `hsl(${hue}, 80%, 70%)`;
      ctx.beginPath();
      ctx.arc(e.x, e.y - t * 30, e.size / 2 * (1 + t * 0.5), 0, Math.PI * 2);
      ctx.fill();

      // Happy face
      if (t > 0.3) {
        ctx.fillStyle = '#333';
        ctx.font = (e.size * 0.4) + 'px monospace';
        ctx.textAlign = 'center';
        ctx.fillText(':)', e.x, e.y - t * 30 + 4);
      }

      ctx.globalAlpha = 1;
      continue;
    }

    // HP flash when damaged
    const damaged = e.hp < e.maxHp;
    const flash = damaged && Math.sin(state.time * 15) > 0;

    ctx.fillStyle = flash ? '#fff' : e.color;

    // Draw based on enemy type
    if (e.type === 'shadowPuff') {
      // Cloud shape
      ctx.beginPath();
      ctx.arc(e.x, e.y, e.size / 2, 0, Math.PI * 2);
      ctx.fill();
      ctx.beginPath();
      ctx.arc(e.x - 8, e.y + 2, e.size / 3, 0, Math.PI * 2);
      ctx.fill();
      ctx.beginPath();
      ctx.arc(e.x + 8, e.y + 2, e.size / 3, 0, Math.PI * 2);
      ctx.fill();
    } else if (e.type === 'corruptedEgg') {
      drawSprite(EGG_SPRITES.shadow, e.x - e.size / 2, e.y - e.size / 2, e.size);
    } else if (e.type === 'darkButterfly') {
      // Simple butterfly: body + wings
      ctx.fillRect(e.x - 1, e.y - 5, 3, 10);
      const wingFlap = Math.sin(state.time * 8) * 0.3;
      ctx.beginPath();
      ctx.ellipse(e.x - 8, e.y, 7, 5, wingFlap, 0, Math.PI * 2);
      ctx.fill();
      ctx.beginPath();
      ctx.ellipse(e.x + 8, e.y, 7, 5, -wingFlap, 0, Math.PI * 2);
      ctx.fill();
    } else if (e.type === 'shadowSlime') {
      // Blobby shape
      const squish = Math.sin(state.time * 4) * 2;
      ctx.beginPath();
      ctx.ellipse(e.x, e.y, e.size / 2 + squish, e.size / 2 - squish, 0, 0, Math.PI * 2);
      ctx.fill();
      // Eyes
      ctx.fillStyle = '#EDE7F6';
      ctx.fillRect(e.x - 5, e.y - 3, 3, 3);
      ctx.fillRect(e.x + 3, e.y - 3, 3, 3);
    } else if (e.type === 'corruptedDragon') {
      drawSprite(DRAGON_SPRITES.shadow, e.x - e.size / 2, e.y - e.size / 2, e.size);
    } else if (e.type === 'darkMushroom') {
      // Mushroom
      ctx.fillStyle = flash ? '#fff' : '#5D4037';
      ctx.fillRect(e.x - 2, e.y, 5, e.size / 3);
      ctx.fillStyle = flash ? '#fff' : e.color;
      ctx.beginPath();
      ctx.arc(e.x, e.y, e.size / 2.5, Math.PI, 0);
      ctx.fill();
    } else if (e.type === 'shadowKnight') {
      // Armored square with visor
      ctx.fillRect(e.x - e.size / 2, e.y - e.size / 2, e.size, e.size);
      ctx.fillStyle = '#EDE7F6';
      ctx.fillRect(e.x - 6, e.y - 2, 12, 4);
    } else if (e.type === 'stormCloud') {
      // Big dark cloud
      ctx.beginPath();
      ctx.arc(e.x, e.y, e.size / 2, 0, Math.PI * 2);
      ctx.fill();
      ctx.beginPath();
      ctx.arc(e.x - 12, e.y + 4, e.size / 3, 0, Math.PI * 2);
      ctx.fill();
      ctx.beginPath();
      ctx.arc(e.x + 12, e.y + 4, e.size / 3, 0, Math.PI * 2);
      ctx.fill();
    } else if (e.type === 'elderDragon') {
      // Big boss - draw multiple sprites layered
      const bossFlash = e.hp <= e.maxHp / 2 ? Math.sin(state.time * 6) * 0.2 : 0;
      ctx.globalAlpha = 0.8 + bossFlash;
      drawSprite(DRAGON_SPRITES.shadow, e.x - e.size / 2, e.y - e.size / 2, e.size);
      ctx.globalAlpha = 1;
      // HP bar
      const hpPct = e.hp / e.maxHp;
      ctx.fillStyle = '#333';
      ctx.fillRect(e.x - 25, e.y - e.size / 2 - 10, 50, 6);
      ctx.fillStyle = hpPct > 0.5 ? '#4CAF50' : hpPct > 0.25 ? '#FF9800' : '#F44336';
      ctx.fillRect(e.x - 25, e.y - e.size / 2 - 10, 50 * hpPct, 6);
    } else {
      // Fallback circle
      ctx.beginPath();
      ctx.arc(e.x, e.y, e.size / 2, 0, Math.PI * 2);
      ctx.fill();
    }
  }
}
```

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "feat: add enemy rendering with unique shapes and cure animation"
```

---

### Task 6: Drops, Effects & Battle HUD

**Files:**
- Modify: `index.html` (BATTLE LOGIC + BATTLE RENDERING sections)

- [ ] **Step 1: Add drop spawning and collection**

```javascript
function spawnDrop(x, y) {
  // Secret easy mode: boost heart drops if struggling
  const heartBoost = state.battle.hitStreak.some(h => h >= 3) ? 0.2 : 0;

  for (const [typeName, dropConfig] of Object.entries(DROP_TYPES)) {
    const chance = typeName === 'heart' ? dropConfig.chance + heartBoost : dropConfig.chance;
    if (Math.random() < chance) {
      state.battle.drops.push({
        type: typeName,
        x: x + (Math.random() - 0.5) * 20,
        y: y,
        vy: 40,
        color: dropConfig.color,
        symbol: dropConfig.symbol,
      });
      return; // Only one drop per enemy
    }
  }
}

function updateDrops(dt) {
  for (let i = state.battle.drops.length - 1; i >= 0; i--) {
    const d = state.battle.drops[i];
    d.y += d.vy * dt;

    // Collect if player touches it
    for (let pi = 0; pi < state.activePlayerCount; pi++) {
      const idx = state.activePlayerCount === 1 ? state.activePlayer : pi;
      const bp = state.battle.battlePlayers[idx];
      if (!bp.alive) continue;

      if (Math.abs(bp.x - d.x) < 20 && Math.abs(BATTLE_PLAYER_Y - d.y) < 20) {
        applyDrop(d.type, idx);
        state.battle.drops.splice(i, 1);
        break;
      }
    }

    // Remove if off screen
    if (d.y > CANVAS_H + 20) {
      state.battle.drops.splice(i, 1);
    }
  }
}

function applyDrop(type, playerIdx) {
  const bp = state.battle.battlePlayers[playerIdx];
  switch (type) {
    case 'heart':
      bp.hearts = Math.min(MAX_HEARTS, bp.hearts + 1);
      break;
    case 'superBoost':
      bp.superMeter = Math.min(SUPER_METER_MAX, bp.superMeter + 25);
      break;
    case 'speedStar':
      bp.speedBoost = 5;
      break;
    case 'shield':
      bp.shieldActive = true;
      break;
  }
  state.battle.effects.push({ type: 'collect', x: bp.x, y: BATTLE_PLAYER_Y - 20, timer: 0, maxTimer: 0.4 });
}
```

- [ ] **Step 2: Add effects update and rendering**

```javascript
function updateBattleEffects(dt) {
  for (let i = state.battle.effects.length - 1; i >= 0; i--) {
    state.battle.effects[i].timer += dt;
    if (state.battle.effects[i].timer >= state.battle.effects[i].maxTimer) {
      state.battle.effects.splice(i, 1);
    }
  }
}

function drawBattleEffects() {
  for (const e of state.battle.effects) {
    const t = e.timer / e.maxTimer;

    if (e.type === 'cure') {
      // Sparkle burst
      for (let i = 0; i < 8; i++) {
        const angle = (i / 8) * Math.PI * 2;
        const dist = t * 30;
        const sx = e.x + Math.cos(angle) * dist;
        const sy = e.y + Math.sin(angle) * dist;
        ctx.globalAlpha = 1 - t;
        ctx.fillStyle = `hsl(${(i * 45 + state.time * 200) % 360}, 80%, 70%)`;
        ctx.fillRect(sx - 2, sy - 2, 4, 4);
      }
      ctx.globalAlpha = 1;
    } else if (e.type === 'hit') {
      ctx.globalAlpha = 1 - t;
      ctx.fillStyle = '#fff';
      ctx.beginPath();
      ctx.arc(e.x, e.y, 6 + t * 10, 0, Math.PI * 2);
      ctx.fill();
      ctx.globalAlpha = 1;
    } else if (e.type === 'playerHit') {
      ctx.globalAlpha = 1 - t;
      ctx.strokeStyle = '#FF5252';
      ctx.lineWidth = 2;
      ctx.beginPath();
      ctx.arc(e.x, e.y, 15 + t * 20, 0, Math.PI * 2);
      ctx.stroke();
      ctx.globalAlpha = 1;
    } else if (e.type === 'collect') {
      ctx.globalAlpha = 1 - t;
      ctx.fillStyle = '#FFD700';
      ctx.font = '12px monospace';
      ctx.textAlign = 'center';
      ctx.fillText('+', e.x, e.y - t * 15);
      ctx.globalAlpha = 1;
    }
  }
}

function drawDrops() {
  for (const d of state.battle.drops) {
    const bob = Math.sin(state.time * 5 + d.x) * 2;
    ctx.fillStyle = d.color;
    ctx.font = '14px serif';
    ctx.textAlign = 'center';
    ctx.fillText(d.symbol, d.x, d.y + bob);
  }
}
```

- [ ] **Step 3: Add battle HUD**

```javascript
function drawBattleHUD() {
  // Wave indicator
  ctx.fillStyle = '#fff';
  ctx.font = 'bold 14px monospace';
  ctx.textAlign = 'center';
  ctx.fillText('Wave ' + state.battle.wave, CANVAS_W / 2, 20);

  // Wave clear message
  if (state.battle.waveState === 'clearing') {
    const alpha = Math.min(1, state.battle.waveTimer * 2);
    ctx.globalAlpha = alpha;
    ctx.fillStyle = '#FFD700';
    ctx.font = 'bold 22px monospace';
    ctx.fillText('Wave ' + state.battle.wave + ' cleared!', CANVAS_W / 2, CANVAS_H / 2 - 20);
    // Sparkles
    for (let i = 0; i < 8; i++) {
      const angle = (i / 8) * Math.PI * 2 + state.time * 3;
      const dist = 40 + Math.sin(state.time * 5 + i) * 10;
      ctx.fillStyle = `hsl(${i * 45}, 80%, 70%)`;
      ctx.fillRect(CANVAS_W / 2 + Math.cos(angle) * dist - 2, CANVAS_H / 2 - 20 + Math.sin(angle) * dist - 2, 4, 4);
    }
    ctx.globalAlpha = 1;
  }

  // Wave waiting countdown
  if (state.battle.waveState === 'waiting' && state.battle.wave > 0) {
    ctx.fillStyle = '#aaa';
    ctx.font = '14px monospace';
    ctx.fillText('Next wave...', CANVAS_W / 2, CANVAS_H / 2);
  }

  // Per-player HUD
  for (let pi = 0; pi < state.activePlayerCount; pi++) {
    const idx = state.activePlayerCount === 1 ? state.activePlayer : pi;
    const bp = state.battle.battlePlayers[idx];
    const dragonKey = state.battle.playerDragons[idx];
    const config = DRAGON_CONFIGS[dragonKey];
    const isLeft = pi === 0;
    const hudX = isLeft ? 10 : CANVAS_W - 10;
    const align = isLeft ? 'left' : 'right';

    // Player name
    ctx.fillStyle = config.color;
    ctx.font = '11px monospace';
    ctx.textAlign = align;
    ctx.fillText(players[idx].name + ' - ' + config.name.split(' ')[0], hudX, 18);

    // Hearts
    let heartsStr = '';
    for (let h = 0; h < MAX_HEARTS; h++) {
      heartsStr += h < bp.hearts ? '\u2665 ' : '\u2661 ';
    }
    ctx.fillStyle = bp.hearts <= 1 ? '#FF5252' : '#FF8A80';
    ctx.font = '12px serif';
    ctx.fillText(heartsStr, hudX, 35);

    // Super meter bar
    const barX = isLeft ? 10 : CANVAS_W - 110;
    const barW = 100;
    ctx.fillStyle = '#333';
    ctx.fillRect(barX, 42, barW, 8);
    const meterPct = bp.superMeter / SUPER_METER_MAX;
    const meterColor = meterPct >= 1 ? `hsl(${(state.time * 200) % 360}, 80%, 60%)` : config.color;
    ctx.fillStyle = meterColor;
    ctx.fillRect(barX, 42, barW * meterPct, 8);

    if (meterPct >= 1) {
      ctx.fillStyle = '#FFD700';
      ctx.font = '9px monospace';
      ctx.textAlign = 'center';
      ctx.fillText('SUPER READY!', barX + barW / 2, 41);
    }
  }
}
```

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add drops, battle effects, and HUD with hearts and super meter"
```

---

### Task 7: Super Moves & Super Button

**Files:**
- Modify: `index.html` (BATTLE LOGIC + HTML for super button)

- [ ] **Step 1: Add super button HTML**

Add after the d-pad HTML elements:

```html
<button id="super-btn-left" class="super-btn super-btn-left" style="display:none;">SUPER!</button>
<button id="super-btn-right" class="super-btn super-btn-right" style="display:none;">SUPER!</button>
```

Add CSS:

```css
.super-btn {
  position: fixed;
  width: 80px;
  height: 80px;
  border-radius: 50%;
  border: 3px solid #FFD700;
  background: rgba(255, 215, 0, 0.2);
  color: #FFD700;
  font-family: monospace;
  font-size: 14px;
  font-weight: bold;
  z-index: 20;
  touch-action: manipulation;
  display: none;
}

.super-btn.ready {
  background: rgba(255, 215, 0, 0.4);
  animation: superPulse 0.8s ease-in-out infinite;
}

.super-btn-left { bottom: 200px; left: 30px; }
.super-btn-right { bottom: 200px; right: 30px; }

@keyframes superPulse {
  0%, 100% { transform: scale(1); box-shadow: 0 0 10px rgba(255, 215, 0, 0.3); }
  50% { transform: scale(1.1); box-shadow: 0 0 25px rgba(255, 215, 0, 0.6); }
}
```

- [ ] **Step 2: Add super move logic**

```javascript
function activateSuper(playerIdx) {
  const bp = state.battle.battlePlayers[playerIdx];
  if (bp.superMeter < SUPER_METER_MAX || !bp.alive) return;

  bp.superMeter = 0;
  const dragonKey = state.battle.playerDragons[playerIdx];

  switch (dragonKey) {
    case 'fire':
      // Flame Wave: damage all enemies on screen
      for (const e of state.battle.enemies) {
        if (!e.curing) { e.hp -= 3; if (e.hp <= 0) { e.curing = true; e.cureTimer = 0; } }
      }
      state.battle.effects.push({ type: 'flameWave', x: CANVAS_W / 2, y: CANVAS_H / 2, timer: 0, maxTimer: 1 });
      break;

    case 'ice':
      // Freeze Blast: freeze all enemies for 3 seconds
      state.battle.superActive[playerIdx] = { type: 'freeze', timer: 3 };
      state.battle.effects.push({ type: 'freezeBlast', x: CANVAS_W / 2, y: CANVAS_H / 2, timer: 0, maxTimer: 0.5 });
      break;

    case 'nature':
      // Vine Burst: damage and hold all enemies
      for (const e of state.battle.enemies) {
        if (!e.curing) { e.hp -= 2; if (e.hp <= 0) { e.curing = true; e.cureTimer = 0; } }
      }
      state.battle.superActive[playerIdx] = { type: 'vines', timer: 2 };
      state.battle.effects.push({ type: 'vineBurst', x: CANVAS_W / 2, y: CANVAS_H / 2, timer: 0, maxTimer: 0.8 });
      break;

    case 'love':
      // Heal Wave: full hearts + cure all
      for (let pi2 = 0; pi2 < state.activePlayerCount; pi2++) {
        const idx2 = state.activePlayerCount === 1 ? state.activePlayer : pi2;
        state.battle.battlePlayers[idx2].hearts = MAX_HEARTS;
      }
      for (const e of state.battle.enemies) {
        if (!e.curing) { e.curing = true; e.cureTimer = 0; }
      }
      state.battle.effects.push({ type: 'healWave', x: CANVAS_W / 2, y: CANVAS_H / 2, timer: 0, maxTimer: 1 });
      break;

    case 'shadow':
      // Shadow Cloak: invincible 5 seconds
      bp.invTimer = 5;
      state.battle.effects.push({ type: 'shadowCloak', x: bp.x, y: BATTLE_PLAYER_Y, timer: 0, maxTimer: 0.5 });
      break;

    case 'star':
      // Supernova: clear everything
      for (const e of state.battle.enemies) {
        if (!e.curing) { e.curing = true; e.cureTimer = 0; }
      }
      state.battle.effects.push({ type: 'supernova', x: CANVAS_W / 2, y: CANVAS_H / 2, timer: 0, maxTimer: 1.2 });
      break;

    case 'rainbow':
      // Rainbow Wave: all enemies become drops
      for (const e of state.battle.enemies) {
        if (!e.curing) {
          spawnDrop(e.x, e.y);
          spawnDrop(e.x, e.y); // double drops
          e.curing = true;
          e.cureTimer = 0;
        }
      }
      state.battle.effects.push({ type: 'rainbowWave', x: CANVAS_W / 2, y: CANVAS_H / 2, timer: 0, maxTimer: 1 });
      break;
  }
}

function updateSuperActive(dt) {
  for (let pi = 0; pi < 2; pi++) {
    const sa = state.battle.superActive[pi];
    if (!sa) continue;
    sa.timer -= dt;

    if (sa.type === 'freeze') {
      // Enemies don't move (handled in updateEnemies)
    }

    if (sa.timer <= 0) {
      state.battle.superActive[pi] = null;
    }
  }
}
```

In `updateEnemies`, add freeze check at the top of the loop body:

```javascript
// At the start of the enemy update loop, after curing check:
const frozen = state.battle.superActive.some(sa => sa && sa.type === 'freeze');
const vined = state.battle.superActive.some(sa => sa && sa.type === 'vines');
if (frozen || vined) continue; // skip movement
```

- [ ] **Step 3: Wire super buttons**

```javascript
// Super button touch/click handlers
function setupSuperButtons() {
  const leftBtn = document.getElementById('super-btn-left');
  const rightBtn = document.getElementById('super-btn-right');

  if (leftBtn) {
    leftBtn.addEventListener('touchstart', function(e) { e.preventDefault(); activateSuper(0); }, { passive: false });
    leftBtn.addEventListener('click', function() { activateSuper(0); });
  }
  if (rightBtn) {
    rightBtn.addEventListener('touchstart', function(e) { e.preventDefault(); activateSuper(1); }, { passive: false });
    rightBtn.addEventListener('click', function() { activateSuper(1); });
  }
}

// Keyboard super (spacebar for P1, 'e' for P2)
// Add to keydown handler:
// if (e.key === ' ' && state.screen === 'battle') activateSuper(0);
// if (e.key === 'e' && state.screen === 'battle') activateSuper(1);
```

Add to the game loop's battle update section: `updateSuperActive(dt);`

Add super button visibility logic -- show/hide and add `ready` class based on meter:

```javascript
function updateSuperButtons() {
  const leftBtn = document.getElementById('super-btn-left');
  const rightBtn = document.getElementById('super-btn-right');
  const inBattle = state.screen === 'battle';

  if (leftBtn) {
    const showLeft = inBattle && isTouchDevice && state.activePlayerCount >= 1;
    leftBtn.style.display = showLeft ? 'block' : 'none';
    if (showLeft) {
      const idx = state.activePlayerCount === 1 ? state.activePlayer : 0;
      const ready = state.battle.battlePlayers[idx].superMeter >= SUPER_METER_MAX;
      leftBtn.classList.toggle('ready', ready);
    }
  }

  if (rightBtn) {
    const showRight = inBattle && isTouchDevice && state.activePlayerCount === 2;
    rightBtn.style.display = showRight ? 'block' : 'none';
    if (showRight) {
      const ready = state.battle.battlePlayers[1].superMeter >= SUPER_METER_MAX;
      rightBtn.classList.toggle('ready', ready);
    }
  }
}
```

Call `updateSuperButtons()` in the game loop after drawing.

- [ ] **Step 4: Add super move effect rendering**

Add to `drawBattleEffects`:

```javascript
// Inside drawBattleEffects, add cases:
} else if (e.type === 'flameWave') {
  ctx.globalAlpha = (1 - t) * 0.4;
  ctx.fillStyle = '#FF4500';
  ctx.fillRect(0, CANVAS_H * t * 0.3, CANVAS_W, CANVAS_H * 0.3);
  ctx.globalAlpha = 1;
} else if (e.type === 'freezeBlast') {
  ctx.globalAlpha = (1 - t) * 0.3;
  ctx.fillStyle = '#B3E5FC';
  ctx.fillRect(0, 0, CANVAS_W, CANVAS_H);
  ctx.globalAlpha = 1;
} else if (e.type === 'supernova') {
  const radius = t * Math.max(CANVAS_W, CANVAS_H);
  ctx.globalAlpha = (1 - t) * 0.5;
  ctx.fillStyle = '#FFD700';
  ctx.beginPath();
  ctx.arc(e.x, e.y, radius, 0, Math.PI * 2);
  ctx.fill();
  ctx.globalAlpha = 1;
} else if (e.type === 'healWave') {
  ctx.globalAlpha = (1 - t) * 0.3;
  ctx.fillStyle = '#FF69B4';
  ctx.fillRect(0, 0, CANVAS_W, CANVAS_H);
  for (let h = 0; h < 10; h++) {
    const hx = (Math.sin(h * 73.1) * 0.5 + 0.5) * CANVAS_W;
    const hy = (Math.cos(h * 131.7) * 0.5 + 0.5) * CANVAS_H;
    ctx.fillStyle = `rgba(255, 105, 180, ${(1 - t) * 0.5})`;
    ctx.font = '20px serif';
    ctx.textAlign = 'center';
    ctx.fillText('\u2665', hx, hy);
  }
  ctx.globalAlpha = 1;
} else if (e.type === 'rainbowWave') {
  for (let r = 0; r < 12; r++) {
    const angle = (r / 12) * Math.PI * 2 + state.time * 2;
    const dist = t * 300;
    const rx = e.x + Math.cos(angle) * dist;
    const ry = e.y + Math.sin(angle) * dist;
    ctx.globalAlpha = (1 - t) * 0.6;
    ctx.fillStyle = `hsl(${r * 30}, 80%, 60%)`;
    ctx.beginPath();
    ctx.arc(rx, ry, 8 + t * 5, 0, Math.PI * 2);
    ctx.fill();
  }
  ctx.globalAlpha = 1;
}
```

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add super moves with unique effects per dragon and touch button"
```

---

### Task 8: Dragon Select Touch Support & D-Pad Visibility

**Files:**
- Modify: `index.html` (touch button handling, d-pad show/hide)

- [ ] **Step 1: Add touch support for dragon select**

Add touch-friendly dragon selection: show dragon options as tappable buttons, or allow tapping left/right side of screen to browse and center to confirm.

Add dragon select touch overlay HTML:

```html
<div id="dragon-select-touch" style="display:none; position: fixed; bottom: 20px; left: 0; right: 0; z-index: 20; text-align: center;">
  <button id="dragon-prev" class="room-btn" style="font-size: 24px; padding: 15px 25px;">&#9664;</button>
  <button id="dragon-confirm" class="room-btn" style="font-size: 18px; padding: 15px 30px; border-color: #FFD700; color: #FFD700;">Select!</button>
  <button id="dragon-next" class="room-btn" style="font-size: 24px; padding: 15px 25px;">&#9654;</button>
</div>
```

Wire touch events:
```javascript
const prevBtn = document.getElementById('dragon-prev');
const nextBtn = document.getElementById('dragon-next');
const confirmBtn = document.getElementById('dragon-confirm');

if (prevBtn) prevBtn.addEventListener('touchstart', function(e) {
  e.preventDefault();
  keys['ArrowLeft'] = true;
  setTimeout(() => keys['ArrowLeft'] = false, 50);
}, { passive: false });

if (nextBtn) nextBtn.addEventListener('touchstart', function(e) {
  e.preventDefault();
  keys['ArrowRight'] = true;
  setTimeout(() => keys['ArrowRight'] = false, 50);
}, { passive: false });

if (confirmBtn) confirmBtn.addEventListener('touchstart', function(e) {
  e.preventDefault();
  keys['Enter'] = true;
  setTimeout(() => keys['Enter'] = false, 50);
}, { passive: false });
```

- [ ] **Step 2: Update d-pad and overlay visibility**

Create a function that manages visibility of all overlays based on game state:

```javascript
function updateBattleUI() {
  const dragonSelectUI = document.getElementById('dragon-select-touch');
  if (dragonSelectUI) {
    dragonSelectUI.style.display = (state.screen === 'dragonSelect' && isTouchDevice) ? 'block' : 'none';
  }

  // D-pads: show in battle mode, hide in dragon select
  // (existing d-pad visibility logic should also check for battle mode)
}
```

Call `updateBattleUI()` in the game loop.

- [ ] **Step 3: Add keyboard super activation**

In the existing keydown handler, add:

```javascript
if (e.key === ' ' && state.screen === 'battle') {
  const idx = state.activePlayerCount === 1 ? state.activePlayer : 0;
  activateSuper(idx);
  e.preventDefault();
}
if ((e.key === 'e' || e.key === 'E') && state.screen === 'battle') {
  activateSuper(1);
}
```

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add dragon select touch controls and keyboard super activation"
```

---

### Task 9: Firebase Sync for Battle Mode

**Files:**
- Modify: `index.html` (Firebase sync section)

Battle mode needs to sync through Firebase for 2-iPad play, using the same host/guest pattern.

- [ ] **Step 1: Add battle state to Firebase sync**

Find the existing Firebase sync functions (where host sends state and guest receives it). Extend the compressed state to include battle data when in battle mode:

```javascript
// In the host sync function, add battle state:
if (state.screen === 'battle' || state.screen === 'dragonSelect') {
  syncData.scr = state.screen;
  syncData.bat = {
    w: state.battle.wave,
    ws: state.battle.waveState,
    wt: state.battle.waveTimer,
    pd: state.battle.playerDragons,
    bp: state.battle.battlePlayers.map(bp => ({
      x: bp.x, h: bp.hearts, sm: bp.superMeter, a: bp.alive,
      rt: bp.respawnTimer, it: bp.invTimer, sh: bp.shieldActive, sb: bp.speedBoost,
    })),
    en: state.battle.enemies.map(e => ({
      tp: e.type, x: e.x, y: e.y, hp: e.hp, mhp: e.maxHp,
      c: e.curing, ct: e.cureTimer, tm: e.timer,
    })),
    sh: state.battle.shots.map(s => ({
      x: s.x, y: s.y, vx: s.vx, vy: s.vy, c: s.color, ie: s.isEnemy || false,
    })),
    dr: state.battle.drops.map(d => ({
      tp: d.type, x: d.x, y: d.y, c: d.color, sy: d.symbol,
    })),
    sa: state.battle.superActive,
  };
}
```

On the guest side, decompress and apply battle state:

```javascript
// In the guest state receive handler:
if (data.scr) state.screen = data.scr;
if (data.bat) {
  state.battle.wave = data.bat.w;
  state.battle.waveState = data.bat.ws;
  state.battle.waveTimer = data.bat.wt;
  state.battle.playerDragons = data.bat.pd;
  // ... decompress battlePlayers, enemies, shots, drops, superActive
}
```

- [ ] **Step 2: Handle dragon select for online mode**

In online mode, only the host navigates the dragon select. The guest waits. When host starts battle, the guest's dragon is set by the host (host picks for both, or a negotiation screen -- simplest: host picks both for now, guest auto-assigned player 2 dragon).

Actually simpler: host selects dragon for player 1, then the screen syncs to guest who selects dragon for player 2 on their own device. Add a `dragonSelectPlayer` to the sync data. When it's player 2's turn, the guest's input is used.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add Firebase sync for battle mode state"
```

---

### Task 10: Integration Test & Deploy

**Files:**
- Modify: `index.html` (any fixes)

- [ ] **Step 1: Full playtest checklist**

1. Title → pick player mode → pick game type "Dragon Battle"
2. Dragon select: browse dragons, see stats, select
3. 2P: second player selects after first
4. Battle starts, dragons at bottom auto-shoot
5. Wave 1: Shadow Puffs drift down, 1 hit cures them with happy animation
6. Hearts drop from cured enemies
7. Wave clear celebration between waves
8. Super meter fills, button glows when ready
9. Super move triggers with flashy effect
10. Getting hit: lose heart, invincibility flash
11. Losing all hearts: dizzy stars, respawn with full health
12. Wave 5: Elder Dragon mini-boss with HP bar
13. Waves keep going, slowly harder
14. 1P mode works with single d-pad
15. 2P same iPad works with both d-pads + super buttons
16. Touch controls work on iPad
17. Going back to title from battle works cleanly

- [ ] **Step 2: Fix any issues found**

- [ ] **Step 3: Commit and push**

```bash
git add index.html
git commit -m "feat: complete Dragon Battle mode with all enemies, supers, and multiplayer"
git push origin main
```

---

## Summary

| Task | What it adds |
|------|-------------|
| 1 | Battle constants -- dragon configs, enemy types, wave definitions |
| 2 | Dragon select screen + battle mode in title menu |
| 3 | Player movement, auto-shoot, projectile rendering |
| 4 | Enemy spawning, movement patterns, wave system, collision detection |
| 5 | Enemy rendering with unique shapes, cure animation |
| 6 | Drops, battle effects, HUD with hearts and super meter |
| 7 | Super moves (7 unique) with effects and touch button |
| 8 | Dragon select touch controls, keyboard super, UI visibility |
| 9 | Firebase sync for 2-iPad battle mode |
| 10 | Integration test and deploy |
