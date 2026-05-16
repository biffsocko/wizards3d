# CLAUDE.md — Wizards & Warriors 3D

Working notes for anyone (human or AI) picking up this project.

## What this is

A 3D port of [Declan Murphy's Wizards & Warriors](https://github.com/declanjmurphy2013-dot/wizards), a single-file browser fighting game written by a 12-year-old. This port keeps the same characters, controls, and combat feel — just renders in 3D with Three.js.

**Audience matters:** the original was written by a kid in plain JS, in one file. This port aims to stay in that spirit. Declan may eventually inherit, extend, or learn from this code. That goal shapes a lot of the constraints below.

## Hard constraints — don't break these without asking

- **Single `index.html` file.** All HTML, CSS, and JS live in one file. No build step, no `package.json`, no `node_modules`, no bundler. If the project ever needs splitting, that's a conversation, not a refactor.
- **No transpilation / no TypeScript.** Plain modern JS only. A 12-year-old should be able to open the file and read it.
- **Three.js via ESM importmap, loaded from a CDN (currently unpkg).** No local copy of three.js, no npm install. If Three.js breaks the importmap path in a future version, pin the version rather than introducing a build step.
- **Stay close to the 2D version's combat logic.** Stat scaling, attack profiles (`meleeProfile`), stamina costs, super meter behavior, AI shape — these were tuned by Declan. Don't rebalance silently. If a 3D-specific change is needed, leave the 2D-equivalent value in a comment.
- **No external assets.** No `.glb` / `.png` / `.mp3` files. Fighters and environment are built from primitives in code, like the 2D version draws sprites with `fillRect` calls. If we ever want models, that's a separate decision.

## Run it

```bash
cd ~/src/wizards3d
python3 -m http.server 8000
# open http://localhost:8000/
```

Browsers block ES modules over `file://`, so a local server is required. Any static server works (`python3 -m http.server`, `npx serve`, `caddy file-server`, etc.).

## Mental model

The game is a 2.5D fighter — fighters move in the XY plane at `z = 0`, but rendered with a slightly-angled perspective camera, lighting, and shadows. Combat math is essentially identical to a 2D side-scrolling fighter; only the renderer is different.

Coordinate system:

- **+X** is to the right (player 1 starts at `x = -3`, player 2 at `x = +3`)
- **+Y** is up (floor at `y = 0`, jump apex ~`y = 3`)
- **+Z** points toward the camera; combat is always `z = 0`. Background pillars / trees sit at negative Z.
- **1 world unit ≈ 1 foot.** Fighters are ~2 units tall, arena is `±ARENA_W` (=8) wide.

The 2D version uses pixels per second; this port scales those values down with `SPEED_SCALE = 1/70`. When porting any value from the 2D code, divide pixel speeds by ~70 to get sensible Three.js units.

## Code layout (inside `index.html`)

The script section is organized into clearly-labeled blocks:

| Block | What's in it |
|---|---|
| `chars` array (~line 90) | Character roster — name, title, color, accent, weapon, stats. Source of truth; keep aligned with the 2D version |
| `D` (difficulty) and constants | `GRAVITY`, `JUMP_VEL`, `ARENA_W`, `SPEED_SCALE`, `MELEE_KINDS` |
| `initScene` | Three.js setup: scene, camera, renderer, lights, ground, pillars, trees, sky shader |
| `makeWeapon` / `buildFighterMesh` / `addHeadDetail` | Procedural fighter models built from `BoxGeometry` / `SphereGeometry` parented to a `THREE.Group` |
| `fighter` | Spawns a fighter state object + mesh; mirrors `fighter()` in the 2D version |
| `attack`, `block`, `jumpFighter`, `damage` | Combat verbs. Mirror their 2D namesakes |
| `meleeProfile` | Per-weapon-kind reach / damage / cooldown / lunge values |
| `phys`, `applyControl`, `ai` | Tick: cooldowns, gravity, friction, CPU logic |
| `resolve` | Hit detection — melee swing rectangle + projectile vs. fighter AABB |
| `animate` | Procedural limb animation (legs swing when running, weapon arm arcs on attack, shield raises on block) |
| `update`, `loop` | Main game loop; render via `renderer.render(scene, camera)` |
| `makeCards`, `bars`, `flash` | HTML overlay UI |
| `startGame`, `showMenu`, `endGame` | State transitions |

## Conventions

- **Functions named like the 2D version where possible.** A diff between this file and Declan's `index.html` should be readable.
- **`f` is the local name for "this fighter".** `t` for "target". `p1` / `p2` are globals.
- **`G` is the global per-match game state** (timer, fx list, run/over flags).
- **Don't add comments that just restate what code does.** The 2D version comments style — block headers + occasional explanation of why a value was picked — is the target.
- **No emojis in code or commit messages.**

## What's in v1 vs. deferred

See `README.md` for the player-facing scope summary. Internally:

**In v1:** rendering, all 20 characters, light/heavy attack, block, jump, stamina, super meter (charges but doesn't fire), CPU AI, win/lose, HUD, end screen.

**Deferred — don't preempt these:**

- Signature moves (C) — per-character specials with unique 3D effects (Lysandra ice barrage, Pip teleport, Kael demon strike, etc.). The 2D version's `sigMove()` is the reference.
- Ultimates (Space) — cinematic 1.55s super move scenes per character (`ULT` table in 2D version).
- Grab (Shift) and counter (B).
- Hidden combos (VXC, CVC, XXV).
- Multiplayer — PeerJS WebRTC + BroadcastChannel + 6-digit code. The 2D version's net code can port largely verbatim once combat is stable.
- Story mode, skins / crate shop, procedural audio (Web Audio API synth).

**Why deferred is meaningful:** adding any of these well requires a non-trivial chunk of work. Don't half-implement them. If you're touching `attack()` or `damage()` and tempted to "drop in" signature-move handling, stop and write it as a focused change.

## Common pitfalls

- **Don't rewrite combat math to use `THREE.Box3` / Raycaster** for hits. The 2D AABB overlap in `rectsOverlap()` is intentional — it matches the 2D version's behavior so balance carries over.
- **Don't add a build pipeline** to "make things cleaner". The single-file constraint is load-bearing.
- **Camera angle is fixed-ish for combat readability.** A free orbit camera would break the fighting-game feel. Subtle follow + zoom is fine; dramatic angle changes are not unless they're tied to ultimates.
- **Three.js version pin:** use `three@0.160.0` until there's a reason to bump. Newer versions occasionally rename or move things; pin protects against drift.
- **Memory:** `disposeMesh()` is called on fighters and projectiles when they leave the scene. If you add new persistent meshes, dispose them too — otherwise GPU memory leaks accumulate across matches.

## Relationship to the 2D repo

The 2D source of truth lives at <https://github.com/declanjmurphy2013-dot/wizards>. When in doubt about combat semantics or character flavor, read that file. If a 3D-side decision contradicts the 2D version, leave a comment explaining the deviation.
