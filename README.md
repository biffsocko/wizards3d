# Wizards & Warriors 3D

A 3D port of [Declan Murphy's Wizards & Warriors](https://github.com/declanjmurphy2013-dot/wizards) browser fighting game, built with Three.js. Same 20 characters, same controls, same combat feel — just in 3D on a 2.5D combat plane (fighters move on a fixed XY plane, rendered with depth, lighting, and shadows).

Single-file, no build step. Drop `index.html` into a browser (via a local web server).

## Running

```bash
cd wizards3d
python3 -m http.server 8000
```

Then open <http://localhost:8000/>.

Browsers block ES modules over `file://`, so a local server is required.

## Controls

| Key | Action |
|---|---|
| Arrow Keys / WASD | Move / jump |
| Z | Block |
| V | Quick attack |
| X | Heavy attack |
| Esc | Return to menu (mid-fight) |
| P | Pause |

## What's in v1

- All 20 characters from the 2D game, with their stats, weapon kinds, and accent colors
- Procedural 3D fighter models (box-based, character-flavored headwear, capes, weapons)
- 2.5D combat plane with side-angle camera that follows and zooms based on fighter distance
- Movement, jump, block, light + heavy attacks
- Stamina, super meter, cooldowns, tired state, guard break
- Projectile attacks for ranged kinds (staff / wand / flask)
- CPU AI with 4 difficulty modes
- HP / timer / win conditions
- Procedural limb animation (run cycle, attack arc, block stance, cape sway)

## Deferred

- Signature moves (C key) — per-character specials
- Ultimates (Space) — cinematic super moves
- Grab (Shift) and counter (B)
- Hidden combos (VXC, CVC, XXV)
- Multiplayer (PeerJS WebRTC, 6-digit join codes) — present in 2D version
- Story mode, skins / crate shop, procedural audio

## Credits

- Original 2D game: Declan Murphy ([declanjmurphy2013-dot/wizards](https://github.com/declanjmurphy2013-dot/wizards))
- 3D port: Tom Murphy
