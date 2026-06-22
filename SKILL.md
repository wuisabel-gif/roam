---
name: roam
description: Build immersive walkable 3D web experiences you roam through — from scroll-driven CSS-3D corridors (glowing cards you "walk" past, per-section hue shifts, flip-for-info, HUD, end-gate) up to true first-person Three.js rooms you explore with WASD/look, with GLB furniture, enterable rooms behind paintings, and tileable textures. No build step required. Use when someone asks for a 3D hallway, walkthrough, tunnel, museum/gallery walk, scrollytelling corridor, an immersive "walk through X" landing, OR a real explorable 3D room/lobby/suite/hotel/showroom. Brand-agnostic; works for any list of items or any themed interior.
---

# Roam — Walkable 3D (Corridors & Rooms)

Techniques for turning content into a space you move through — either a scroll-driven CSS-3D corridor of doorway cards, or a true first-person Three.js room you walk and look around. Both run from a single HTML file with no build step.

**Two flagship worked examples (both in the same GitHub folder, both original — copy freely):**
- `https://wuisabel-gif.github.io/kpop-crush/` — **CSS-3D corridor**: 21 K-pop groups as glowing doorways, scroll-up to walk, flip cards, HUD, end-gate into a catalogue, Spotify photos + playlist.
- `the Atrium reference build (not bundled in this public repo — full code is inline in references/recipes.md)` — **Three.js first-person room**: an Art-Deco hotel ("The Atrium") you explore with WASD + mouse-look, a portrait hall, GLB furniture (desk/lamp/sofa/armchairs), and **enterable rooms behind paintings** (a suite and a manager's office) via fade transitions. Single file, ES-module three over a CDN importmap, **no build**. This is the reference for everything in the Three.js sections below.

Copy patterns from `references/recipes.md`.

## When to reach for this
- "Make a 3D hallway / tunnel / corridor I walk down."
- "Turn this list into an immersive walkthrough."
- "A landing page where you walk past each [product / project / member / milestone]."
- Any gallery where *movement through space* should carry the narrative.

If the ask is a normal grid/list, this is overkill — use a flat layout instead.

## The mental model

The whole effect is **three nested layers** plus a **scroll-mapped camera**:

```
viewport   →  position:fixed, perspective: ~820px        (the eye)
  world    →  transform-style:preserve-3d, translateZ()  (moves toward you = walking)
    cards  →  translate3d(x, y, -depth) rotateY(angle)   (doorways on the walls)
```

You never move the cards. You move the `world` along Z, and the perspective makes near cards rush past while far cards stay small. Scroll position drives one CSS variable (`--hz`); a `requestAnimationFrame` loop eases the camera toward the scroll target so motion feels weighty, not jumpy.

## Engines — pick your renderer

The default here is **CSS 3D transforms**: zero dependencies, one HTML file, ideal for corridors of flat "doorway" cards. When you need *real* geometry, lighting, or VR, step up to a WebGL engine.

| Engine | Best for | Cost |
|---|---|---|
| **CSS 3D transforms** (default) | Card/doorway corridors, gallery walks, scrollytelling | none — one file |
| **A-Frame** (WebGL, declarative HTML tags) | True 3D rooms/worlds you move a camera through; VR-ready; readable entity markup | one `<script>` CDN tag |
| **Three.js** (WebGL, imperative JS) | Custom meshes, shaders, pixel-art / mosaic scenes, fine-grained control | a library + more code |

The same *concepts* carry across all three — items placed in depth, a camera you advance (by scroll or input), per-section colour, an intro, a HUD, an end handoff. Only the rendering primitive changes: CSS `translateZ` on a `world` div, vs. an A-Frame camera `rig` position, vs. a Three.js `camera.position.z`.

**Worked examples — one per engine:**
- `https://wuisabel-gif.github.io/kpop-crush/` — **CSS-3D** corridor (the default technique).
- `college-playbook-3d_14.html` — *"The College Playbook — Campus Walk"*, a true-3D walk built with **A-Frame** (WebGL). Copy the A-Frame recipe in `references/recipes.md`.
- `the Atrium reference build (not bundled in this public repo — full code is inline in references/recipes.md)` — **Three.js, single file, no build** (RECOMMENDED starting point for WebGL): first-person hotel with GLB furniture and rooms behind paintings, loaded via a CDN importmap. Copy the "Three.js single-file walkable rooms" recipe in `references/recipes.md`.
- `gallery-master/` — **Three.js, advanced/build-based** variant: Blender-baked lighting + Draco/KTX2 + `three-mesh-bvh` capsule collision + nipplejs. Reach for this only when you need baked GI or true mesh collision. Third-party: © Steve245270533, **GPL-3.0** — keep its `LICENSE` and credit the author if you reuse any of it; it needs a build (`pnpm install && pnpm dev`).

## Build order

Follow these steps. Each has a copy-paste recipe in `references/recipes.md`.

### 1. Scene scaffold
Create `viewport` (fixed, `perspective`, dark radial background + edge vignette via `::after`) and `world` (preserve-3d, `transform: translateZ(calc(var(--hz,0)*1px))`). Recipe: **Scene**.

### 2. Lay out the items in depth
For each item `i`, place a card at `z = START - i*SPACING`, alternating left/right walls (`translateX(±X)`), angled toward the viewer (`rotateY(±~26deg)`), with `backface-visibility:hidden`. Add a "ring" frame at each station so the corridor reads as architecture. Recipe: **Cards & rings**.

### 3. Drive the camera from scroll
A tall transparent **spacer** element gives the page its scroll length. Map scroll → progress → `--hz`. Decide direction:
- **scroll down = forward** (default): `p = scrollY / hallLen`.
- **scroll up = forward**: `p = 1 - scrollY/hallLen`, start at the bottom (`scrollTo(0, hallLen)` on load, set `history.scrollRestoration='manual'`).

Ease it in a rAF loop: `curZ += (targetZ - curZ) * 0.08`. Recipe: **Camera**.

### 4. Intro overlay
A full-screen `intro` (z-top) with eyebrow, big title, a one-line description, a primary CTA, and a controls guide. **Give it a radial dark scrim + `backdrop-filter: blur(3px)`** so text never fights the bright corridor behind it. Fade it out once the user starts moving. Recipe: **Intro**.

### 5. Flip-for-info cards (optional but great)
Wrap each card face in a `card-inner` (preserve-3d, `transition: transform .75s`); front = image/title, back = `rotateY(180deg)` info panel. Toggle a `.flipped` class on click; `stopPropagation` on any link inside the back so it doesn't re-flip. Recipe: **Flip cards**.

### 6. HUD
A fixed pill that names the item nearest the camera (compare `|curZ - (-card.z)|`, pick the min) and recolors to that section's hue. Recipe: **HUD**.

### 7. End-gate handoff
When progress ≈ 1, reveal a centered "Enter →" gate. Keep the *next* view (`display:none`) until the user clicks — don't auto-slide. On click: add a body class that hides the gallery (`display:none`) and the spacer, shows the next view, and `scrollTo(0,0)`. Add a "back" button to return. Recipe: **End-gate**.

### 8. Media (optional)
- **Photos:** fetch `https://open.spotify.com/oembed?url=spotify:artist:<id>` (CORS-open, no key) for real artist images; lazy-load via IntersectionObserver. Any oEmbed-style source works the same way. Recipe: **oEmbed photos**.
- **Audio:** embed a background player via the Spotify IFrame API. If items also have their own players, **coordinate so only one plays** — pausing one when another starts. Recipe: **Single-player audio**.

### 9. Accessibility & polish (do not skip)
- Honor `prefers-reduced-motion: reduce`: skip the rAF loop, render a static frame, disable the auto-animations.
- Keep all interactive accents on a per-section CSS hue variable (`--h` / `--foil-hue`) so re-theming is one number.
- Click-disable mockup anchors if it's a demo, not a real site.
- Test on a phone — the corridor should still read; reduce `perspective` slightly and card sizes for small screens.

## The dials (tune per project)

| Dial | Typical | Effect |
|---|---|---|
| `perspective` | 700–900px | Lower = more dramatic/fish-eye |
| `SPACING` | 900–1200px | Gap between stations (walk pacing) |
| side `X` | ±380–460px | Corridor width |
| card `rotateY` | ±20–32° | How much doorways face you |
| ease factor | 0.06–0.10 | Camera weight (lower = floatier) |
| `hallLen` | DEPTH × 0.55–0.7 | Scroll distance for the whole walk |

## Three.js path A (DEFAULT) — single-file walkable rooms, no build

When the brief is a *real room* you walk and look around in — a hotel, suite, office, lobby, showroom — and you want to ship **one HTML file with no toolchain**, use this path. It's what `the Atrium reference build (not bundled in this public repo — full code is inline in references/recipes.md)` is built on. Full code in `references/recipes.md` → **Three.js single-file walkable rooms**. The shape:

1. **Load three from a CDN importmap — no bundler.** A `<script type="importmap">` maps `"three"` and `"three/addons/"` to a CDN (unpkg/jsdelivr) at a pinned version; then `<script type="module">` does `import * as THREE from 'three'` and `import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js'`. This is the single biggest simplification over the build-based path. Recipe: **Importmap boot**.
2. **Shell from planes.** Build each room as 6 textured `PlaneGeometry` walls/floor/ceiling via one `mk(geo, mat, x,y,z, rx,ry)` helper. Textures load with a tiny `TX(path, repeatX, repeatY)` helper (`RepeatWrapping`). Recipe: **Room shell + texture helper**.
3. **Drop in GLB furniture with a `place()` helper.** Auto-scale each model to a target width, recentre it, and **rest it on the floor** (or on another surface) — `place(url, x, y, z, width, rotY, mode, tint?, rough?, metal?, texUrl?)`. Modes: `'floor'` (base sits at y) and `'center'` (centre at y, for wall-mounted pieces). Recipe: **GLB place() helper**.
4. **Handle untextured / mis-coloured models (you WILL hit this).** Free image-to-3D exports (Meshy/Tripo) often arrive grey or with no usable texture. Two fixes: **tint** (force a flat PBR colour — good for upholstery/wood) or **texture-wrap** (apply the original 2D reference PNG as the model's `map` — good when the model's silhouette matches a flat product image, e.g. a clock face). Recipe: **Fix grey models**.
5. **Rest props on a measured surface.** Don't guess a desk's height — load the desk first, read `new Box3().setFromObject(m).max.y` after scaling, then place the lamp/phone/clock at *that* height. Guessing is the #1 cause of props floating or sinking into furniture. Recipe: **Measure-then-stack**.
6. **Controls.** Pointer-lock + `WASD` (movement relative to yaw) on desktop; touch-drag look + a walk button on mobile. Clamp the player inside an axis-aligned box per room (`x/z` min/max with a margin) — no physics engine needed for boxy rooms. Recipe: **First-person controls + clamp**.
7. **Rooms behind paintings (the signature move).** Keep a `ROOMS` registry `{key:{x,w,l,ez}}`, each interior built far offstage (e.g. `x=300`, `x=340`). Raycast from screen centre; looking at a portrait with `userData.portal` and clicking **fades to black, teleports** the player into that room's coordinates, and sets `curRoom`. An exit marker fades back. Recipe: **Rooms behind paintings (portal + fade)**.
8. **Lighting without blowing out.** Small rooms over-expose fast. Keep a strict budget: one dim ambient/hemisphere, one lantern that follows the camera (dimmed further inside rooms), and a couple of low-intensity point/spot accents. If a wall reads white, cut the nearest light's intensity AND range first. Recipe: **Lighting budget**.
9. **Polish + fallback.** Flickering lantern/sconces for mood, `RoomEnvironment` PMREM for soft PBR reflections (keep `envMapIntensity` ~0.2 so it doesn't wash the scene), positional/ambient audio on first gesture, and always a non-3D fallback list for reduced-motion / low-power / screen-reader users.

## Three.js path B (ADVANCED) — baked lighting + mesh collision (build-based)

Step up to this only when you need **baked global illumination** or **true non-boxy collision** (curved walls, stairs, props you can't approximate with an AABB):

1. **Asset pipeline.** Model in Blender, **bake lighting into textures**, export **glb**; export a *second, low-poly invisible* **collision mesh**. Compress with Draco (geometry) + KTX2 (textures). A bundler (vite) handles the imports.
2. **Collision.** Build a `MeshBVH` (`three-mesh-bvh`) on the collision mesh; represent the player as a **capsule** (segment + radius); each frame apply gravity, move, then `shapecast` and push out of walls/floor.
3. **Controls/media.** Pointer-lock + WASD + jump; **nipplejs** joystick on mobile; `Reflector` mirror floor; `CSS3DRenderer` labels for crisp world-space text.
4. **Perf.** Cap `pixelRatio` ~2, Draco/KTX2 everything, frustum-cull.

A full reference for path B lives at `gallery-master/` (third-party, © Steve245270533, GPL-3.0) — study it, but write your own; if you reuse any of its code, keep its LICENSE and credit. For path A, read `the Atrium reference build (not bundled in this public repo — full code is inline in references/recipes.md)` directly — it's original, copy what you like.

## Failure modes to avoid

**CSS-3D corridor**
- **Text over busy 3D** → always scrim/blur the intro and any overlay copy.
- **Clipping into the first card** → start the camera a station *before* item 0 (`START` negative, e.g. -1500).
- **Two audio sources at once** → enforce single-player coordination.
- **Jank from rendering every frame** → throttle heavy canvas redraws (~14fps is plenty); the CSS transform itself can run every frame cheaply.
- **Hardcoded accent colors** → drive every glow/border/wash from one hue variable per section.
- **Mobile scroll start** → if "scroll-up = forward," the one-shot `scrollTo(0,hallLen)` can fail on mobile and strand the user at the end-gate; retry it on rAF/load/orientationchange until the user interacts, and set `touch-action:pan-y`.

**Three.js single-file rooms (all hit this session in `the-atrium`)**
- **`object.position = {...}` silently fails / throws** → three's `.position` is **read-only**; use `obj.position.set(x,y,z)`. Under `"use strict"` this `TypeError` aborts the whole scene build (symptom: stuck on the loading screen).
- **GLB props float above or sink into furniture** → you guessed the surface height. Measure it (`Box3.max.y`) after the host model loads, then stack. See **Measure-then-stack**.
- **Model renders as a flat grey blob** → it shipped untextured. Tint it a flat colour, or texture-wrap it with its 2D reference image. See **Fix grey models**.
- **Room is blown out white** → too many/too-bright lights in a small box. Cut the nearest light's intensity *and* range; dim the camera lantern further when inside a room.
- **Filenames with spaces or non-breaking spaces (U+00A0)** from AI asset tools → `encodeURI()` every asset URL before loading, and watch for `\302\240` in `ls -1b`.
- **Wall-mounted GLB sits on the floor** → use `mode:'center'` (centre at y), not `'floor'` (base at y).

## Aesthetic directions

The corridor is a stage — dress it however the subject wants:
- **Neon / holographic** — dark scene, glowing cards, per-section hue shifts, foil gradients (the kpop_crush look).
- **Architectural / museum** — pale walls, soft daylight, framed works, quiet type (a gallery walk).
- **Campus / world** — ground plane, sky, signage, walkable WebGL space (the college-playbook look).
- **Pixel-art / mosaic** — low-res textures, blocky geometry, chiptune; pairs naturally with the Three.js engine.

Keep one accent hue per section in a CSS/JS variable so the whole palette re-themes from a single number.

## Files
- `SKILL.md` — this file.
- `references/recipes.md` — copy-paste recipes: CSS-3D, A-Frame, Three.js single-file (no build), and Three.js advanced (build-based).
- `https://wuisabel-gif.github.io/kpop-crush/` — worked example, CSS-3D corridor (original).
- `the Atrium reference build (not bundled in this public repo — full code is inline in references/recipes.md)` — worked example, **Three.js single-file walkable rooms** (original, no build, rooms-behind-paintings).
- `college-playbook-3d_14.html` — worked example, A-Frame WebGL campus walk.
- `gallery-master/` — worked example, Three.js advanced/baked exhibition (third-party, © Steve245270533, GPL-3.0).
- `index.html` — showcase page that embeds the examples.
