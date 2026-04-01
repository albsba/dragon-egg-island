# Dragon Egg Island -- Game Design Spec

## Overview

**Dragon Egg Island** is a cooperative 2-player browser game where two best friends (Harper and Audrey) explore a magical island, discover hidden dragon eggs, watch them hatch, and build a shared dragon family together.

- **Tech:** Single HTML file, Canvas rendering, vanilla JavaScript
- **Target audience:** 6-year-olds (super simple controls, friendly visuals)
- **Controls:** Player 1 (Harper) = Arrow keys, Player 2 (Audrey) = WASD. Movement only -- no other buttons.
- **Art style:** Pixel art, chunky and colorful
- **Modes:** Adventure mode (goals/unlocks) + Free play mode
- **Approach:** Start with vanilla HTML/Canvas (Approach A). Migrate to Phaser.js in a future stage if needed for dragon riding/powers (Approach C).

## Characters

### Harper (Player 1)
- Light blue pixel art character with snowflake motif
- Controlled with Arrow keys (Up/Down/Left/Right)
- 4-directional movement

### Audrey (Player 2)
- Purple pixel art character
- Controlled with WASD keys
- 4-directional movement

Both players move smoothly in a tile-based world. No jumping, no attacking -- just walking/exploring.

## World Design

The island is made of distinct magical zones, connected so players walk between them. Each zone fits roughly one screen with soft scroll boundaries.

### Zones (in adventure mode unlock order)

1. **Flower Meadow** (starting zone)
   - Bright, colorful flowers and grass
   - Egg types: Fire, Nature
   - Mood: Cheerful, welcoming

2. **Heart Forest**
   - Pink trees, heart-shaped leaves, little streams
   - Egg types: Love
   - Mood: Warm, sweet

3. **Enchanted Forest**
   - Tall dark trees, glowing mushrooms, hidden paths
   - Egg types: Nature, Shadow
   - Mood: Mysterious, magical

4. **Crystal Cave**
   - Sparkling walls, underground pools, gem formations
   - Egg types: Star, Ice
   - Mood: Glittery, wondrous

5. **Snowy Mountain**
   - Snowdrifts, frozen waterfalls, icy paths
   - Egg types: Ice, Star
   - Mood: Cool, peaceful

6. **Volcano Ridge**
   - Warm orange glow, lava streams (safe -- no damage)
   - Egg types: Fire, Shadow
   - Mood: Exciting, dramatic

7. **Rainbow Falls** (secret final zone)
   - Waterfalls with rainbow mist, prismatic light
   - Egg types: Rainbow (exclusive)
   - Unlocked after collecting all other egg types
   - Mood: Magical, celebratory

## Dragon Eggs & Types

7 dragon types, each with a distinct color and personality:

| Type | Egg Color | Found In | Future Power (Stage 2+) |
|------|-----------|----------|------------------------|
| Fire | Orange/Red | Flower Meadow, Volcano Ridge | Melts ice blocks |
| Nature | Green | Flower Meadow, Enchanted Forest | Grows bridges |
| Love | Pink | Heart Forest | Heals/helps |
| Shadow | Dark Purple | Enchanted Forest, Volcano Ridge | Reveals hidden paths |
| Star | Gold | Crystal Cave, Snowy Mountain | Lights dark areas |
| Ice | Light Blue | Crystal Cave, Snowy Mountain | Freezes water to walk on |
| Rainbow | Rainbow gradient | Rainbow Falls | Special/all powers |

### Egg Behavior
- Eggs sit in the world, glowing with a soft pulse animation
- They bob gently up and down to attract attention
- Placed in discoverable spots -- behind trees, in corners, near landmarks
- Either player can walk over an egg to collect it

### Hatching
- When an egg is collected, a hatching animation plays:
  1. Egg wobbles side to side
  2. Cracks appear on the surface
  3. Egg breaks open, baby dragon pops out with sparkle effect
- The baby dragon is added to the shared collection
- Baby dragons follow the players around in a parade line behind them

## Collection & UI

### Shared Collection
- Both players contribute to one shared collection (cooperative, not competitive)
- Collection tracker displayed on screen showing:
  - Dragon type icons for each type discovered
  - Count of how many of each type they have
  - Grayed-out silhouettes for types not yet found

### Baby Dragon Parade
- Hatched dragons follow behind the players as small pixel sprites
- They bounce along in a line, creating a cute trailing effect
- The parade grows as more dragons are collected

## Game Modes

### Adventure Mode
- Start in Flower Meadow, other zones locked
- Collect a target number of eggs in the current zone to unlock the path to the next
- Unlocking plays a sparkle animation on the zone boundary
- Goal: Reach Rainbow Falls and hatch a Rainbow Dragon
- Progression: Flower Meadow → Heart Forest → Enchanted Forest → Crystal Cave → Snowy Mountain → Volcano Ridge → Rainbow Falls

### Free Play Mode
- All zones unlocked from the start
- Eggs respawn so they never run out
- No goals -- just explore, collect, and enjoy
- Accessible from the start screen

### Start Screen
- Simple title screen: "Dragon Egg Island"
- Two options: "Adventure" and "Free Play"
- Either player can select with their movement keys

## Visual Design

### Pixel Art Style
- Chunky, colorful pixel art (think 16x16 or 32x32 sprite scale)
- Characters and dragons are hand-drawn in code (SVG/Canvas pixel rendering)
- Each zone has its own distinct color palette

### Animations & Effects
- Character walk cycle (simple 2-frame animation)
- Egg glow/pulse (ambient)
- Egg wobble-crack-hatch sequence
- Baby dragon bounce (following players)
- Zone unlock sparkle
- Per-zone ambient particles:
  - Flower Meadow: floating petals
  - Heart Forest: drifting hearts
  - Enchanted Forest: floating fireflies
  - Crystal Cave: twinkling gems
  - Snowy Mountain: falling snowflakes
  - Volcano Ridge: rising embers
  - Rainbow Falls: prismatic sparkles

## Stage 1 Scope (What We Build Now)

### Included
- Two-player simultaneous movement (Arrow keys + WASD)
- Harper (light blue/snowflake) and Audrey (purple) pixel art characters
- Flower Meadow and Heart Forest zones (first 2 zones)
- Fire, Nature, and Love dragon eggs
- Egg collection mechanic (walk over to collect)
- Hatching animation
- Baby dragons following players
- Collection tracker UI
- Adventure mode with zone unlocking (Meadow → Heart Forest)
- Free play mode (both zones, eggs respawn)
- Simple start screen with mode selection
- Zone-specific ambient particles (petals, hearts)

### Deferred to Stage 2+
- Remaining 5 zones (Enchanted Forest, Crystal Cave, Snowy Mountain, Volcano Ridge, Rainbow Falls)
- Remaining 4 egg types (Shadow, Star, Ice, Rainbow)
- Dragon riding mechanic
- Dragon powers
- Sound effects and music
- Save/load progress
- Migration to Phaser.js (if needed for riding/powers)
- Additional egg types beyond the core 7 (the girls want more -- expandable)

## Technical Notes

- Single `index.html` file, no dependencies, no build step
- Open in any modern browser to play
- Canvas element for game rendering
- Game loop with requestAnimationFrame
- Tile-based world map defined as 2D arrays
- Collision detection for zone boundaries and egg pickup
- Keyboard input handling for two simultaneous players
- Code structured cleanly for future Phaser migration (separate game state, rendering, input)
