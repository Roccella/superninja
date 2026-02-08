# Super Ninja Hyper Mega Ultra

## Overview
Single-file HTML5 Canvas top-down action game with Bayonetta-style Witch Time.
Gamepad (PS5) + keyboard. All code in one file: `proto/superninja-static.html` (v51).

**Inspirations**: Bayonetta (Witch Time), Hotline Miami (camera), Crimsonland (top-down shooter)

## File Structure
- `proto/superninja-static.html` — current version (v51)
- `proto/superninja-stable-1.html` — stable backup (pre-camera, v36)
- `proto/prototype_v28.html` — early prototype

## Architecture
- Game state in `game` object
- Game loop: `gameLoop()` → `update(gamepad)` → `render()`
- No external dependencies, pure Canvas 2D
- Single HTML with inline CSS + JS, all assets drawn procedurally

## Camera & Game Area
- **Viewport**: Dynamic fullscreen
- **Game area**: 2000x1250px (16:10)
- **Camera**: Hotline Miami-style, 40px deadzone, 0.08 smoothing
- **Zoom**: 1.2x normal, 1.0x during Witch Time
- **Out-of-bounds**: Dark (#0a0a14) with green border (#4ecca3)

## Core Mechanics

### Player
- **Movement**: WASD / Left Stick. Base speed 3.5
- **Melee**: K / R1 — Hold to charge (max 42 frames). Combo system: every 3rd hit is a Combo Finisher (violet, 1.5x damage, can deflect lasers)
- **Dash**: O / L1 — Quick dodge, invincible during dash. Base: speed 18, duration 24 frames
- **Witch Time**: P / L2/R2 — Activates when charge bar full, or parry (dodge near charging enemy)
- **Shooting**: Auto-fire at visible enemies, independent aim from melee direction
- **Weapons**: SMG / Shotgun, auto-switches every 10s when enemies nearby
- **HP**: 100, regenerates 0.25/frame after 1.5s no damage

### Witch Time System
- Charge fills by melee hits (5% per hit, 100 max)
- Duration: 6s base (360 frames), upgradeable
- During Witch Time:
  - Enemies at 0.20x speed, player at 1x
  - Camera zooms out
  - Melee: +40 range, +0.3 rad arc
  - Bullet damage doubled
  - Kills extend duration + bar blinks magenta/white
  - Enemy corpses split → halves (suspended, 6-8s) → quarters (1.5s) with melee
  - Hit area for pieces: circular 360°, distance-based (generous)
- **Perfect Clear**: Kill ALL enemies → 3s freeze + wave animation + ALL 5 powerups +1

### Enemy Types
1. **Normal** (red circles): Fast, weak, melee attacks
2. **Big** (blue diamonds): Slower, more HP, laser attack (charge ~1.1s, fire ~0.13s, lineWidth 0.17)
3. **Boss** (gold pentagons): Slowest, high HP, laser + melee (charge ~2s, fire ~0.3s, lineWidth 0.28, size 66)

### Enemy Hit Effects
- Bounce/scale on hit: melee 1.35x squish, gun 1.18x squish
- Animation decay: 0.04/frame
- White flash on damage
- Spawn fade-in: 0.5s

### Laser System
- Big/Boss enemies charge and fire lasers
- Targeting line same thickness as melee charge rings (lineWidth: 2)
- **Deflect with charged melee or combo finisher** → redirects to random visible enemy
- Deflected damage: 8 (big), 15 (boss) — one-shots most enemies
- Hitting enemy cancels their laser charge

### Melee Combo System
- Every 3rd normal melee = **Combo Finisher**: violet (#aa44ff), 1.5x damage, 1.15x range, 1.1x arc, can deflect lasers
- Combo resets after ~0.75s of no attacks
- Charged attacks reset combo counter

### Melee Range
- Normal time: base - 15px
- Witch Time: base + 40px
- Charged melee: +40 bonus range

### Death Animation (SlashDeath Phases)
1. `flash`: Initial flash (can be cut)
2. `split`: Halves separating (can be cut, faster in WT)
3. `suspended`: Floating during Witch Time (can be cut)
4. `half`: Single half piece (can be cut into quarters)
5. `quarter`: Quarter piece (cannot cut further)
6. `explode`: Final explosion
7. `piece`: Legacy phase
- Visual opacity: halves 70%, quarters 55%
- Enemies killed by WT activation → halves directly in 'suspended' (ready to cut)

### Power-Up System (Slot Machine)
On level up, slot machine with 5 options (Dash/L1 to select after 1s, auto at 3s):
1. **MELEE** (🗡️): Damage + Range
2. **GUN** (🔫): Damage + Fire Rate
3. **DASH** (💨): Speed + Distance
4. **W. TIME** (⏱️): Duration + Enemy Slowdown
5. **REGEN** (❤️): HP Regen + Charge Speed

### XP & Levels
- XP per kill: normal=15, big=50, boss=150
- XP bar blinks on gain, intense blink on level up

## UI Elements
- **XP Bar**: Bottom, with level indicator
- **Health Bar**: Left side, with regen indicator
- **Witch Time Bar**: Right side, charge + active duration
- **Power-up Icons**: Center-top, white count numbers
- **Weapon Indicator**: Current weapon name
- **Targeting Reticle**: 4 arrow markers rotating around target with lock-in animation
- **Pause (H)**: Controls + power-up info
- **Slot Machine**: Floating circle next to player on level-up
- **Perfect Clear**: Text above center, icons below

## Visual Effects
- Screen shake on hits
- Particles for attacks, hits, deaths
- Ghost trails during dash
- Shockwaves on parry and boss attacks
- Circle wipe transition when Witch Time ends

## Technical Details

### Gamepad System
- High-frequency polling: `setInterval` at 0ms (~1000Hz) independent from game loop (60Hz)
- Accumulates events in `buttonsPressedThisFrame`/`buttonsReleasedThisFrame`
- Game loop copies to `buttonsJustPressed`/`buttonsJustReleased` — matches keyboard responsiveness
- Handles analog triggers: `btn.pressed || btn.value > 0.5`

### Key Game Objects
- `game.player`: Player state (includes `lastMeleeWasCharged`, `meleeComboCount`, `comboResetTimer`, `reticleAnim`, `reticleRotation`)
- `game.enemies`: Enemy array (with `spawnFadeIn`, `hitBounce`, `laserCharging/laserFiring`)
- `game.slashDeaths`: Death pieces (phases: flash→split→suspended→half→quarter→explode)
- `game.witchTime`: WT state (includes `circleWipe.expanding`)
- `game.witchTimeCharge`: Charge bar state
- `slotMachine`: Level-up system
- `powerUps`: Power-up levels

### Witch Time CircleWipe
- Activation expands circle wipe from player
- During expanding: `game.witchTime.circleWipe.expanding = true`
- `inWitchTime` is `false` during wipe (for enemy slowdown)
- Pieces use separate check including expanding wipe to stay alive

## Controls Summary

| Action | Keyboard | Gamepad (PS5) |
|--------|----------|---------------|
| Move | WASD | Left Stick |
| Melee | K (hold charge) | R1 |
| Dash | O | L1 |
| Witch Time | P / dodge | L2/R2 / dodge |
| Aim | auto | auto |
| Shoot | auto | auto |
| Weapon | auto | auto |
| Pause | H | Options |
