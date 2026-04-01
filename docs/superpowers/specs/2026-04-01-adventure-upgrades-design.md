# Adventure Mode Upgrades -- Design Spec

## Overview

Three upgrades to adventure mode: dragon riding for faster exploration, hidden secret rooms in each zone, and a simple quest system with stars.

## Dragon Riding

- A "RIDE" button (HTML overlay, like the swap button) appears when the player has at least one dragon in their collection
- Tap to mount the most recently collected dragon
- While riding: move 2x faster, dragon sprite replaces the character sprite (drawn larger), character sprite sits on top
- Tap RIDE again to dismount (or it could say "WALK" when mounted)
- The mounted dragon is removed from the parade while being ridden
- Each dragon type has a visual flair while riding:
  - Fire: small flame trail behind
  - Ice: frost sparkles
  - Nature: leaf particles
  - Love: heart trail
  - Shadow: dark wisps
  - Star: golden sparkles
  - Rainbow: rainbow trail
- Riding is purely cosmetic/speed -- no gameplay gates (keep it simple for 6-year-olds)
- Touch: RIDE button positioned bottom-center, between the two d-pads
- Works in both adventure and free play modes

## Hidden Secrets

### Secret tiles (discoverable by exploring)
- 2-3 tiles per zone that LOOK solid (a specific bush, cracked rock, unusual mushroom) but are actually walkable
- Walking into them triggers a "secret found!" sparkle effect and teleports the player to a small hidden room
- Hidden room: a small 10x8 area with a unique background color and 1-2 rare eggs plus sparkle effects
- Walking to the edge of the hidden room returns to the main zone
- Secret tiles are visually SLIGHTLY different from normal solid tiles (a tiny sparkle every few seconds, slightly different shade) so observant players can spot them

### Secret paths (earned by collecting)
- After collecting all eggs currently spawned in a zone, a secret path appears somewhere in the zone
- The path sparkles gently at first
- After 30 seconds, it starts glowing brightly and pulsing so they can't miss it
- Walking on it leads to a bonus area with extra rare eggs
- In free play mode, secret paths appear after collecting 5 eggs in a zone

### Hidden rooms
- Each zone has 1 unique hidden room
- Small area (10 cols x 8 rows) with themed decoration
- Contains 1-2 eggs of a type matching the zone
- Flower Meadow secret: a tiny garden with golden flowers
- Heart Forest secret: a heart-shaped clearing with a pond
- Enchanted Forest secret: a fairy circle with glowing mushrooms
- Crystal Cave secret: a gem vault with crystal walls
- Snowy Mountain secret: an ice castle room
- Volcano Ridge secret: a magma chamber with gold
- Rainbow Falls secret: a prismatic shrine

## Quest System

### Quest list
- 3 quests active at a time, shown at top-right of screen (small text, doesn't obscure gameplay)
- Quests auto-complete when conditions are met
- Completing a quest: star burst animation, quest text flashes gold, new quest replaces it
- Stars are counted and displayed on the title screen ("12 stars earned!")

### Quest types (randomly assigned)
- "Collect 3 [type] eggs" -- collect 3 of a specific dragon type
- "Visit [zone name]" -- walk into a specific zone
- "Find a secret room" -- discover any hidden room
- "Hatch 5 dragons" -- hatch 5 eggs total
- "Collect 2 different dragon types" -- have at least 2 types in collection
- "Walk to Heart Forest and back" -- visit heart forest then return to meadow
- "Collect an egg in [zone]" -- collect any egg in a specific zone
- "Build a dragon parade of 4" -- have 4+ dragons following you

### Quest state
- `state.quests` = array of 3 active quests: `[{ type, target, progress, completed }]`
- `state.stars` = total stars earned (persists in localStorage if possible)
- New quests are randomly generated when one completes
- Quests that reference locked zones are skipped (only generate quests for unlocked zones)

## Player Modes

All features work in 1P, 2P same iPad, and 2P online:
- Dragon riding: each player has their own RIDE button and can independently ride/walk
- Secrets: either player can discover secret tiles or paths
- Quests: shared quest progress (co-op, both players contribute)

## Technical Notes

- RIDE button: HTML overlay like super/swap buttons
- Hidden rooms: separate small tile maps loaded when entering a secret
- Quest UI: drawn on canvas as part of drawUI()
- Star count: saved to localStorage('dragonIslandStars')
- All new features are purely additive -- no changes to existing mechanics
