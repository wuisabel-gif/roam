# Roam

**Immersive 3D worlds you actually walk through** — a small, growing collection of browser-based walkable experiences.

> ⚠️ Early prototype. Expect rough edges as the worlds grow.

## The worlds

| World | What it is | Live |
|---|---|---|
| **Kpop Crush — The Hall of Eras** | Scroll-up to walk a neon corridor of 21 K-pop groups; flip a doorway for its info, then enter a full catalogue. Real photos + a background playlist. | https://wuisabel-gif.github.io/kpop-crush/ |
| **The College Playbook — Campus Walk** | A true-3D walkable campus built with A-Frame — a ground plane, lights, and signboards you walk up to. Ships with this site. | included here |
| **Backrooms — Level 0** | A first-person walk through the liminal "backrooms": endless yellow rooms, buzzing lights, that uncanny empty hum. | https://wuisabel-gif.github.io/backroom_level_0/ |

## Run it locally

It's a static site — no build step. Serve the folder and open `index.html`:

```
python3 -m http.server 8000
# then visit http://localhost:8000/
```

The Campus Walk loads from this repo; Kpop Crush and the Backrooms load from their own live sites.

## Tech

Plain HTML/CSS/JS. The landing page is a single `index.html` (animated corridor background, drifting light motes, an SVG logo, and a horizontally-scrolling gallery that launches each world inline). The walkable worlds themselves use CSS 3D transforms, A-Frame, and Three.js.

---

© 2026 [wuisabel-gif](https://github.com/wuisabel-gif) · all rights reserved
