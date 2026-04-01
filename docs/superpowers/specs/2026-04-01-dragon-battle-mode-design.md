# Dragon Battle Mode -- Design Spec

## Overview

**"The Shadow is corrupting Dragon Egg Island!"** A vertical shooter (Space Invaders-style) mode where players fly their collected dragons upward, auto-shooting to cure waves of corrupted creatures. Starts cute and easy, gets slowly darker and harder. No game over -- instant respawn, infinite play.

- **Integration:** New mode accessible from the main title screen alongside egg collecting. Also unlockable as Stage 3 in adventure mode, but can be jumped to directly.
- **Tech:** Same `index.html` file, new screen state (`state.screen = 'battle'`)
- **Players:** 1P, 2P same iPad, 2P two iPads -- uses the exact same player mode system
- **Controls:** D-pad left/right to move, auto-shoot upward, big glowing button for super move
- **Target:** 6-year-olds. Very easy. They basically can't lose.

## Dragon Selection

Before battle, each player picks a dragon. Uses existing pixel art dragon sprites scaled up.

| Dragon | Shot Type | Super Move | Shot Color |
|--------|-----------|------------|------------|
| Fire | Fireballs (fast, straight) | Flame Wave -- wall of fire across screen | #FF4500 |
| Ice | Ice shards (medium, slight spread of 3) | Freeze Blast -- freezes all enemies 3 sec | #4FC3F7 |
| Nature | Leaf shots (medium, pierce through enemies) | Vine Burst -- vines grab all enemies on screen | #43A047 |
| Love | Heart shots (slow, wide hitbox) | Heal Wave -- full hearts + cures all on screen | #E91E63 |
| Shadow | Shadow bolts (fast, narrow) | Shadow Cloak -- invincible 5 seconds | #5E35B1 |
| Star | Star shots (bounce off screen edges) | Supernova -- big explosion clears everything | #FFC107 |
| Rainbow | Rainbow beam (continuous stream) | Rainbow Wave -- all enemies become power-ups | #FF6B6B |

**Super meter:** Fills as enemies are cured. When full, the super button glows and pulses. Tap to unleash. Meter resets after use.

## Dragon Select Screen

- Show all 7 dragons in a row with names
- Player 1 picks first (left/right to browse, confirm to select)
- Player 2 picks second (can pick same dragon -- that's fine)
- In 1-player mode, just one selection
- Big pixel art preview of selected dragon
- Touch-friendly: tap a dragon to select

## Gameplay

### Screen Layout

- **Play area:** Full canvas width, most of canvas height
- **HUD at top:** Hearts, super meter, wave counter
- **Dragons at bottom:** ~3 tiles up from the bottom edge, move left/right only
- **Enemies:** Spawn at top, move downward in patterns
- **Auto-shoot:** Dragons fire upward constantly (~4 shots per second)

### 2-Player Co-op

Both dragons on screen simultaneously, side by side at the bottom. Each controlled by their respective d-pad (left d-pad / right d-pad, or arrows / WASD). Shared health pool or individual -- individual hearts, shared wave progress. They work together to clear waves.

### Movement

- Left/right only (no up/down in battle mode)
- Smooth pixel movement (not tile-based like egg collecting)
- Dragon stays within screen bounds
- Movement speed: ~200 pixels/second

## Enemies

All enemies are "corrupted" -- dark/purple tinted. When cured (shot enough), they flash bright, do a happy spin, and fly away. NOT destroyed -- cured. This is important for the tone.

### Wave 1-5: Cute & Easy

- **Shadow Puffs** -- dark purple clouds, drift straight down slowly, 1 hit
- **Corrupted Eggs** -- dark glowing eggs wobbling down, 1 hit
- **Dark Butterflies** -- gentle zigzag downward, 1 hit
- 3-5 enemies per wave, very slow movement

### Wave 6-10: Medium

- **Shadow Slimes** -- bounce side to side while descending, 2 hits
- **Corrupted Baby Dragons** -- swoop in arcs, 2 hits
- **Dark Mushrooms** -- drop and split into 2 smaller ones on first hit, 1 hit each after
- 5-8 enemies per wave, moderate speed
- Mix of easy and medium enemies

### Wave 11+: Harder (but still fair)

- **Shadow Knights** -- armored, 3 hits, move in formation
- **Storm Clouds** -- big, slow, 4 hits, shoot a slow dark droplet downward (easy to dodge)
- Enemy count increases gradually
- Speed increases very slowly
- Mix of all enemy types

### Mini-Boss (every 5 waves)

- **Corrupted Elder Dragon** -- large, takes 10 hits
- Simple predictable pattern: sweeps left-right, pauses, drops 3 slow projectiles, repeat
- Flashes and changes color with each hit so they can see progress
- When cured: dramatic happy animation, drops lots of hearts and power-ups

## Drops (from cured enemies)

Items float down slowly after an enemy is cured. Walk into them to collect.

- **Hearts** -- restore 1 health. Common (~30% drop rate)
- **Super Boost** -- fills super meter by 25%. Uncommon (~15%)
- **Speed Star** -- temporary speed boost for 5 seconds. Rare (~10%)
- **Shield Bubble** -- blocks next hit. Rare (~10%)

## Health & Respawn

- Each player starts with **5 hearts** (shown as heart icons in HUD)
- Getting hit by an enemy or projectile = lose 1 heart + 1 second invincibility flash
- Hearts drop frequently from enemies (tuned so they rarely run low)
- **If all hearts lost:**
  1. Cute dizzy stars animation on dragon (2 seconds)
  2. Dragon respawns in place with full 5 hearts
  3. 3 seconds of invincibility after respawn (flashing)
  4. Wave continues uninterrupted
  5. No score penalty, no game over
- **Secret easy mode:** If a player gets hit 3 times in one wave, secretly increase heart drop rate and slow enemies by 10%

## Projectiles

### Player shots
- Auto-fire upward at ~4 shots/second
- Each dragon type has visually distinct shots (fireballs, ice shards, hearts, etc.)
- Shots travel upward at ~400 pixels/second
- Disappear when hitting an enemy or leaving the screen

### Enemy projectiles (Wave 11+ only, Storm Clouds)
- Dark purple droplets
- Very slow (~100 pixels/second)
- Large and easy to see
- Only 1-2 on screen at a time

## Wave Flow

1. "Wave X" text appears briefly at top of screen
2. Enemies spawn from top in a pattern (rows, V-shape, random, etc.)
3. Players shoot to cure all enemies
4. When all cured: brief celebration ("Wave X cleared!" with sparkles)
5. 2-second pause
6. Next wave begins
7. Every 5th wave: mini-boss instead of normal enemies
8. Endless -- keeps going until they quit

## HUD

- **Top-left:** Player 1 hearts (heart icons) + dragon name
- **Top-right:** Player 2 hearts + dragon name (if 2-player)
- **Top-center:** Wave counter ("Wave 7")
- **Super meter:** Bar below hearts for each player, glows when full
- **Super button:** Large glowing button overlaid on screen (HTML element, not canvas), only visible when super is ready. Positioned above each player's d-pad.

## Visual Style

- Same pixel art style as egg collecting mode
- Background: dark starfield (simple, doesn't distract)
- Corrupted enemies: purple/dark tinted versions of cute shapes
- Curing animation: enemy flashes white → bright colors → happy face → flies up and away
- Player shots: colored streaks matching dragon type
- Super moves: full-screen flashy effects (appropriate to each dragon)
- Wave clear: sparkle explosion + brief text

## Integration with Main Game

### Title Screen

Add "Dragon Battle" as a third game type option after choosing player mode:
- Adventure (egg collecting)
- Free Play (egg collecting)
- Dragon Battle

### From Adventure Mode

After collecting all 7 dragon types and reaching Rainbow Falls, a "Dragon Battle" option appears -- "The Shadow is attacking! Defend the island!" But this is optional flavor. The mode is always accessible from the title screen regardless of progress.

### Shared Assets

Reuses: dragon sprites (scaled up), pixel art renderer, d-pad controls, touch handling, Firebase networking (for 2-iPad mode), particle effects.

## Controls Summary

| Input | Action |
|-------|--------|
| D-pad left/right | Move dragon |
| D-pad up/down | No effect in battle |
| Auto | Shoot upward continuously |
| Super button (touch) | Activate super move when meter full |
| Spacebar (keyboard) | Activate super move when meter full |

## Technical Notes

- Battle mode uses pixel-based movement (not tile-based like egg collecting)
- Enemy positions stored as floating point {x, y, vx, vy, hp, type}
- Collision detection: simple rectangle overlap (hitboxes)
- Shot pooling: reuse shot objects to avoid garbage collection
- Firebase sync for 2-iPad: same host/guest pattern. Host runs battle logic, syncs enemy positions + shot states. Guest sends input, renders received state.
- The index.html file is getting large (~1800+ lines). Battle mode code should be organized in its own clearly marked sections with comment banners.
