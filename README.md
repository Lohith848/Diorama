# Monsoon Works

A looping pixel-art diorama of a Tamil Nadu village rain factory, made with Three.js r160 and a hand-written palette-snap post pass.
A copper kettle boils the sea, a bell tower hangs the steam as a cloud, a wringer squeezes it into a jug, and a scooter courier carries the
jug to a dry paddy. Then the works cat sits on the safety lever and everything goes wrong, on schedule.

- 56 s seamless loop, locked camera (azimuth 45, elevation 33), 64 palette colours, no textures, no imported assets
- 4 systems (Sea Kettle, Cloud Bell, Wringer, Scooter Dispatch), 4 people and one cat, 1432 meshes
- Every frame is a pure function of the time `t`, so any moment can be reproduced exactly

## Run it

- **Just look:** open `monsoon-works.html`. It is one self-contained file and works over `file://`.
- **Develop:** `npm install`, then `node build.mjs` (add `--min` to minify), then open `index.html`.
  `build.mjs` bundles `src/main.js` with esbuild and writes `index.html` (loads `dist/app.js`), `monsoon-works.html` (everything inlined)
  and `dist/monsoon-works.artifact.html` (the same page without the html/head/body tags, for hosts that wrap it).

## URL switches

| switch | effect |
| --- | --- |
| `?t=30.9` | freeze on one frame (seconds, 0 to 56) |
| `?from=28` | start the loop at that time |
| `?grey=1` | flat grey blockout, no post pass |
| `?scale=1` | render at 1.0 instead of 1.25 (pixelated upscale) |
| `?dither=0`, `?ink=0`, `?snap=0` | switch one stage of the post pass off |

Switches need a query string, so they do not exist in a hosted copy that only accepts a hash.

Viewers who ask their system for reduced motion open on the establishing frame (t = 6 s). A click, tap or Space plays and pauses the loop.

From the browser console, `window.__diorama` exposes `freeze(t)`, `play()`, `info()`, `world`, `camera`, `scene` and `THREE`.

## What happens (56 s)

| t (s) | beat |
| --- | --- |
| 0 to 12 | normal output: shovelling, cranking, cloud swelling, paddy drying |
| 12 to 19.6 | the cat wakes, crosses the works and sits on the safety lever |
| 19.6 to 30 | pressure climbs, the stoker and the operator stop |
| 30 | the valve pops, the cat is flung, the cloud bursts over the yard |
| 31 to 36 | the courier corks the jug (30.9), hooks the bungee (32.3), mounts (33.8) and rides out (35.6) |
| 40 to 46 | the farmer pulls the cork (41.0), the cloud climbs out of the jug and rains on the paddy (shower 43.6 to 50.4, paddy greens 44.2 to 48) |
| 46 to 56 | the soaked courier rides home (51.2) and dismounts (51.5 to 52.9), the cat is asleep again (52.4), the farmer is back on his stool (55.7) |

The full beat sheet, cast sheet, set coordinates and deviations are in `DESIGN.md`.

## Layout

```
src/main.js            renderer, resize, loop, URL switches
src/config.js          LOOP, SLAB, RENDER and shared easing helpers
src/core/              palette (64 colours), palette-name materials, geometry helpers, PixelPost (MSAA target, ink, dither, palette snap)
src/scene/world.js     builds everything, one update(t) for the whole world
src/scene/beats.js     the timeline table and the signals derived from it
src/scene/set.js       slab, works floor, paddy, road, pad, ground height
src/scene/{kettle,tower,wringer,scooter,fx,cat}.js   machines, effects, cat
src/scene/rig.js, motion.js                           clothed IK person rig and the motion toolkit
src/scene/{stoker,operator,courier,farmer,cast}.js    the four people and their choreography
qa/                    headless Chrome checks (below)
shots/                 beat frames, strips, seam frames and the contact sheet
DESIGN.md  CREDITS.md  refs/SOURCES.md  TODO.md
_old/                  superseded early code, not built
```

## Checks (`qa/`)

All scripts drive headless Chrome with SwiftShader through `playwright-core`. Run them after `node build.mjs`.

| command | what it does |
| --- | --- |
| `node qa/shoot.mjs --t=6,30.9 [--vp=1280x800] [--page=monsoon-works.html]` | frozen frames into `shots/`, prints console errors |
| `node qa/follow.mjs --who=farmer --t=40,41,42` | close-up crops that follow one character |
| `python3 qa/strip.py out.png a.png b.png ... --cols=9` | tile crops into a labelled strip |
| `python3 qa/sheet.py out.png 2 0.6 a.png b.png ...` | captioned contact sheet |
| `node qa/sweep.mjs` | every 0.05 s: NaN check, closest approach between actors, facing against motion |
| `node qa/contacts.mjs` | largest miss of every palm or boot against its anchor in each contact window |
| `node qa/eval.mjs "D.world.parts.cast.cat.group.position.y" --t=17.7` | evaluate one expression inside the page |

## Credits

Everything on screen is hand-built from primitives and flat palette colours. Look, camera and post-pass recipe follow the
`diorama` skill v3.8.0 by Nehal Kanak (MIT).
