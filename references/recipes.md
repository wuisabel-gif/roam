# Roam — Recipes

Copy-paste building blocks for both tracks: the **CSS-3D corridor** (sections below up to the A-Frame recipe) and the **Three.js single-file walkable rooms** (the last section). Adapt names/values; keep the structure. In the corridor track every accent reads from a per-section hue var `--h` (an HSL hue number, e.g. `276`).

---

## Scene

```html
<div class="viewport"><div class="world" id="world"></div></div>
<div class="floorglow"></div>
<div class="spacer" id="spacer" aria-hidden="true"></div>
```

```css
.viewport{position:fixed;inset:0;overflow:hidden;z-index:1;perspective:820px;perspective-origin:50% 47%;
  transform-style:preserve-3d;
  background:radial-gradient(120% 90% at 50% 38%, #1a1230 0%, #0c0820 42%, #060410 78%, #040208 100%);}
.viewport::after{content:"";position:absolute;inset:0;pointer-events:none;z-index:40;
  background:radial-gradient(120% 80% at 50% 46%, transparent 40%, rgba(4,2,10,.5) 78%, rgba(3,1,7,.92) 100%);}
.world{position:absolute;inset:0;transform-style:preserve-3d;will-change:transform;
  transform:translateZ(calc(var(--hz,0) * 1px));}
.floorglow{position:fixed;left:0;right:0;bottom:0;height:34vh;z-index:30;pointer-events:none;
  background:linear-gradient(180deg,transparent,rgba(150,90,230,.12));
  -webkit-mask-image:linear-gradient(180deg,transparent,#000 60%);mask-image:linear-gradient(180deg,transparent,#000 60%);}
.spacer{position:relative;width:100%;}   /* height set by JS = scroll length */
```

---

## Cards & rings

```css
.ring{position:absolute;left:50%;top:50%;width:1240px;height:760px;margin:-380px 0 0 -620px;border-radius:10px;
  transform-style:preserve-3d;border:2px solid hsl(var(--h) 85% 62% / .42);
  box-shadow:0 0 60px hsl(var(--h) 85% 55% / .28), inset 0 0 90px hsl(var(--h) 80% 40% / .14);}
.card{position:absolute;left:50%;top:50%;width:300px;height:460px;transform-style:preserve-3d;backface-visibility:hidden;cursor:pointer;}
.card .sign{position:absolute;left:50%;top:-52px;transform:translateX(-50%);white-space:nowrap;
  font:900 30px/1 Georgia,serif;color:hsl(var(--h) 90% 82%);
  text-shadow:0 0 8px hsl(var(--h) 90% 60%),0 0 22px hsl(var(--h) 90% 55% /.8),0 0 44px hsl(var(--h) 85% 50% /.6);}
.card .frame{position:absolute;inset:0;border-radius:14px;overflow:hidden;display:flex;align-items:center;justify-content:center;
  background:linear-gradient(180deg,hsl(var(--h) 40% 16%),hsl(var(--h) 45% 9%));border:2px solid hsl(var(--h) 85% 65% /.7);
  box-shadow:0 0 40px hsl(var(--h) 85% 55% /.5), inset 0 0 50px hsl(var(--h) 70% 40% /.35);}
.card .frame img{position:absolute;inset:0;width:100%;height:100%;object-fit:cover;opacity:0;transition:opacity .6s ease;}
.card .frame img.loaded{opacity:1;}
.card .spill{position:absolute;left:-30%;right:-30%;bottom:-90px;height:120px;border-radius:50%;
  background:radial-gradient(ellipse at 50% 40%,hsl(var(--h) 85% 60% /.55),transparent 70%);filter:blur(14px);}
```

```js
var SPACING=1050, START=-1500;
var ITEMS=[ /* {name, hue, ...} */ ];
var world=document.getElementById('world');
ITEMS.forEach(function(it,i){
  var z=START - i*SPACING, side=(i%2===0)?-1:1, x=side*440, rotY=side*-26, y=(i%2===0)?-14:22;
  var ring=document.createElement('div'); ring.className='ring'; ring.style.setProperty('--h',it.hue);
  ring.style.transform='translate(-50%,-50%) translate3d(0,0,'+z+'px)'; world.appendChild(ring);
  var card=document.createElement('div'); card.className='card'; card.style.setProperty('--h',it.hue);
  card.style.transform='translate(-50%,-50%) translate3d('+x+'px,'+y+'px,'+z+'px) rotateY('+rotY+'deg)';
  card.innerHTML='<div class="sign">'+it.name+'</div><div class="frame"></div><div class="spill"></div>';
  world.appendChild(card); it._z=z;
});
```

---

## Camera (scroll-driven, eased)

```js
var DEPTH=(ITEMS.length+1.4)*SPACING, hallLen;
var spacer=document.getElementById('spacer');
function vh(){return window.innerHeight;}
function sizeSpacer(){ hallLen=DEPTH*0.62; spacer.style.height=(hallLen+vh())+'px'; }
sizeSpacer();
var reduce=matchMedia('(prefers-reduced-motion:reduce)').matches;
var curZ=0,tgtZ=0;
function apply(z){ world.style.setProperty('--hz', z.toFixed(1)); /* + updateHud(z), lazyPhotos(z) */ }

// DOWN = forward:  var p = Math.min(1,Math.max(0, window.scrollY/hallLen));
// UP   = forward (used in the example):
function onScroll(){
  var p=Math.min(1,Math.max(0, 1 - window.scrollY/hallLen));
  tgtZ=p*DEPTH;
  if(reduce){ curZ=tgtZ; apply(curZ); }
}
addEventListener('scroll',onScroll,{passive:true});
addEventListener('resize',function(){ sizeSpacer(); onScroll(); });

// For UP-forward, start at the bottom:
if('scrollRestoration' in history){ try{history.scrollRestoration='manual';}catch(e){} }
requestAnimationFrame(function(){ scrollTo(0,hallLen); onScroll(); curZ=tgtZ; if(reduce) apply(curZ); });
if(!reduce){ requestAnimationFrame(function loop(){ requestAnimationFrame(loop); curZ+=(tgtZ-curZ)*0.08; apply(curZ); }); }
```

---

## Intro (with scrim — critical for legibility)

```css
.intro{position:fixed;inset:0;z-index:80;display:flex;flex-direction:column;align-items:center;justify-content:center;
  text-align:center;padding:24px;transition:opacity .8s ease;
  background:radial-gradient(125% 82% at 50% 46%, rgba(7,5,16,.93) 0%, rgba(7,5,16,.82) 40%, rgba(7,5,16,.52) 72%, rgba(7,5,16,.12) 100%);
  -webkit-backdrop-filter:blur(3px);backdrop-filter:blur(3px);}
.intro > *{position:relative;}
body.moving .intro{opacity:0;pointer-events:none;}
.intro h1{font:900 clamp(2.6rem,8vw,5.6rem)/.95 Georgia,serif;letter-spacing:-.02em;
  background:linear-gradient(100deg,#A6C44E,#E054A0,#9B6BE0,#3FB6CF,#2FB58A);
  -webkit-background-clip:text;background-clip:text;color:transparent;}
.intro .guide{display:flex;flex-wrap:wrap;justify-content:center;gap:8px;margin:6px auto 18px;max-width:580px;}
.intro .gi{display:inline-flex;align-items:center;gap:8px;font:10.5px ui-monospace,monospace;letter-spacing:.1em;text-transform:uppercase;
  color:rgba(255,255,255,.85);background:rgba(12,8,26,.6);border:1px solid hsl(280 70% 65% /.35);backdrop-filter:blur(6px);padding:8px 13px;border-radius:999px;}
```

Fade trigger: in `onScroll`, `document.body.classList.toggle('moving', p>0.02)`.

---

## Flip cards

```html
<div class="card"><div class="card-inner">
  <div class="cface cfront"><div class="frame"><img></div></div>
  <div class="cface cback"><div class="bk">
    <div class="bk-name">Name</div>
    <dl><div><dt>Debut</dt><dd>…</dd></div><div><dt>More</dt><dd>…</dd></div></dl>
    <a class="bk-link" href="…" target="_blank" rel="noopener">▶ Link</a>
  </div></div>
</div><div class="spill"></div></div>
```

```css
.card-inner{position:absolute;inset:0;transform-style:preserve-3d;transition:transform .75s cubic-bezier(.22,1,.36,1);}
.card.flipped .card-inner{transform:rotateY(180deg);}
.cface{position:absolute;inset:0;backface-visibility:hidden;-webkit-backface-visibility:hidden;border-radius:14px;}
.cback{transform:rotateY(180deg);display:flex;align-items:center;justify-content:center;padding:22px;color:#fff;
  background:linear-gradient(180deg,hsl(var(--h) 46% 15%),hsl(var(--h) 52% 8%));border:2px solid hsl(var(--h) 85% 65% /.7);}
```

```js
card.addEventListener('click',function(){ card.classList.toggle('flipped'); });
var link=card.querySelector('.bk-link'); if(link) link.addEventListener('click',function(e){ e.stopPropagation(); });
```

---

## HUD

```js
var hud=document.getElementById('hud'), hudName=document.getElementById('hudName');
function updateHud(z){
  var best=0,d=1e9; ITEMS.forEach(function(it,i){var dd=Math.abs(z-(-it._z));if(dd<d){d=dd;best=i;}});
  var it=ITEMS[best];
  if(hudName.textContent!==it.name){ hudName.textContent=it.name; hud.style.setProperty('--h',it.hue); }
}
```

---

## End-gate handoff

```css
.gallery .endgate{position:fixed;inset:0;z-index:70;display:flex;align-items:center;justify-content:center;
  opacity:0;pointer-events:none;transition:opacity .6s ease;text-align:center;padding:24px;}
.gallery .endgate.show{opacity:1;pointer-events:auto;}
.nextview{display:none;position:relative;z-index:2;background:#fff;}      /* the view you hand off to */
body.entered .gallery, body.entered .spacer{display:none;}
body.entered .nextview{display:block;}
```

```js
// in onScroll: endgate.classList.toggle('show', p>=0.985);
function enterNext(){ document.body.classList.add('entered'); scrollTo(0,0); }
document.getElementById('enterBtn').addEventListener('click', enterNext);
// back button: document.body.classList.remove('entered'); scrollTo(0, Math.round(hallLen*0.5));
```

---

## oEmbed photos (lazy)

```js
var io=new IntersectionObserver(function(es){es.forEach(function(e){
  if(e.isIntersecting){ var el=e.target,id=el.dataset.id,img=el.querySelector('img');
    if(id&&img&&!el.__done){ el.__done=1; loadPhoto(id,img); } io.unobserve(el); }
});},{rootMargin:'300px 0px'});
function loadPhoto(id,img){
  fetch('https://open.spotify.com/oembed?url=spotify:artist:'+id)
    .then(function(r){return r.ok?r.json():null;})
    .then(function(d){if(d&&d.thumbnail_url){img.onload=function(){img.classList.add('loaded');};img.src=d.thumbnail_url;}})
    .catch(function(){});
}
// Or: load only when the card is within ~2600px of the camera (see kpop_crush ensurePhotos()).
```

---

## Single-player audio coordination

Background playlist via the Spotify IFrame API; per-item players are plain embeds. Rule: **opening an item player pauses the background; the background starting closes any open item.**

```html
<div id="bg-embed"></div>
<script>
  window.__spPlaying=false; window.__ducked=false; window.__wasPlaying=false;
  window.__duck=function(){ if(!__ducked){__wasPlaying=!!__spPlaying;__ducked=true;} if(__spc&&__spPlaying){try{__spc.pause();}catch(e){}} };
  window.__restore=function(){ if(!__ducked)return; __ducked=false; if(__spc&&__wasPlaying){try{__spc.resume();}catch(e){}} __wasPlaying=false; };
  window.onSpotifyIframeApiReady=function(API){
    API.createController(document.getElementById('bg-embed'),
      {uri:'spotify:playlist:PLAYLIST_ID',width:'100%',height:'80'},function(c){
        window.__spc=c;
        c.addListener('playback_update',function(e){
          var playing=!(e&&e.data&&e.data.isPaused);
          if(playing && !__spPlaying && !__ducked && window.__closeOpenItem) window.__closeOpenItem();
          window.__spPlaying=playing;
        });
      });
  };
</script>
<script src="https://open.spotify.com/embed/iframe-api/v1" async></script>
```

In the item open/close handlers: call `window.__duck()` on open and (deferred, so switching items doesn't blip) `window.__restore()` on close; expose `window.__closeOpenItem`.

Caveats to state to users: audio needs a click to start (autoplay is blocked), and full Spotify tracks require being logged into Spotify in the browser (else 30-sec previews).

---

## Reduced motion checklist
- Skip the rAF camera loop; set `--hz` directly from scroll.
- Disable looping background animations (auras, tickers, equalizers).
- Render a usable static frame (park the camera a little way into the hall).
- Everything must remain reachable by plain scrolling.

---

## A-Frame walkable scene (WebGL path)

For true 3D rooms/worlds (the `college-playbook-3d_14.html` style). Same concepts as the CSS path — items in depth, a camera you advance, per-section colour — but rendered by WebGL via A-Frame's declarative tags. One CDN script, no build.

```html
<script src="https://aframe.io/releases/1.5.0/aframe.min.js"></script>

<a-scene background="color:#0a0712" fog="type:exponential;color:#0a0712;density:0.03">
  <a-assets>
    <img id="p0" crossorigin="anonymous" src="…">
    <img id="p1" crossorigin="anonymous" src="…">
  </a-assets>

  <!-- ground + ambient/point light -->
  <a-plane rotation="-90 0 0" width="40" height="200" position="0 0 -90" color="#120a22"></a-plane>
  <a-entity light="type:ambient;intensity:0.6"></a-entity>
  <a-entity light="type:point;intensity:0.8;color:#9B6BE0" position="0 4 -10"></a-entity>

  <!-- camera rig: move THIS down -Z to walk forward -->
  <a-entity id="rig" position="0 1.6 6">
    <a-camera look-controls="pointerLockEnabled:false" wasd-controls="enabled:false"></a-camera>
  </a-entity>

  <!-- one glowing doorway per item, alternating walls, receding in Z -->
  <!-- built in JS below -->
</a-scene>
```

```js
// place doorways
var SPACING=6, items=[{img:'#p0',hue:'#9B6BE0'},{img:'#p1',hue:'#3FB6CF'}];
var scene=document.querySelector('a-scene');
items.forEach(function(it,i){
  var side=(i%2===0)?-1:1, x=side*2.4, z=-4 - i*SPACING;
  var d=document.createElement('a-entity');
  d.setAttribute('geometry','primitive:plane;width:1.6;height:2.3');
  d.setAttribute('material','src:'+it.img+';side:double;emissive:'+it.hue+';emissiveIntensity:0.25');
  d.setAttribute('position', x+' 1.6 '+z);
  d.setAttribute('rotation','0 '+(side*-22)+' 0');           // face the corridor
  // a glowing frame behind it:
  var f=document.createElement('a-entity');
  f.setAttribute('geometry','primitive:plane;width:1.85;height:2.55');
  f.setAttribute('material','color:'+it.hue+';opacity:0.35;transparent:true;side:double');
  f.setAttribute('position', x+' 1.6 '+(z-0.02));
  f.setAttribute('rotation','0 '+(side*-22)+' 0');
  scene.appendChild(f); scene.appendChild(d);
});

// scroll = walk: map page scroll to the rig's Z (mirror of the CSS camera)
var rig=document.getElementById('rig'), DEPTH=items.length*SPACING+8;
document.body.style.height=(DEPTH*120+window.innerHeight)+'px';  // a tall scroll spacer
var reduce=matchMedia('(prefers-reduced-motion:reduce)').matches;
var curZ=6, tgtZ=6;
addEventListener('scroll',function(){
  var p=Math.min(1,Math.max(0, window.scrollY/(document.body.scrollHeight-innerHeight)));
  tgtZ = 6 - p*DEPTH;                       // forward = decreasing Z
  if(reduce){ curZ=tgtZ; rig.object3D.position.z=curZ; }
},{passive:true});
if(!reduce){ (function loop(){ requestAnimationFrame(loop);
  curZ += (tgtZ-curZ)*0.08; rig.object3D.position.z=curZ; })(); }
```

Notes:
- **Click/gaze to inspect:** add `cursor="rayOrigin:mouse"` to `<a-scene>` and an `a-entity` with `cursor` for gaze; listen for `click` on a doorway to open an info panel (`<a-entity text>` or an HTML overlay).
- **VR for free:** A-Frame ships a headset/AR button; the same scene works in a headset.
- **Per-section colour:** drive `emissive` / light colour from the item's hue, exactly like `--h` in the CSS path.
- **Reduced motion / accessibility:** WebGL scenes are hard for screen readers — always offer a non-3D fallback list of the same items, and disable auto-motion under `prefers-reduced-motion`.

---

## Three.js walkable exhibition (WebGL room)

For a *real room* you walk and look around in — museum / showroom / gallery. All original skeleton code; standard Three.js + three-mesh-bvh patterns. Build it as a Vite project.

### 0. Asset pipeline (decide before coding)
1. Model the room in **Blender**. **Bake** lights/GI into textures — the browser renders no real-time global illumination, so baked maps are what make it look good and run fast.
2. Export **two** glb files: the **visual** room (detailed, baked) and a **collision** mesh (simplified, invisible — boxes/planes roughly tracing walls + floor). You move against the collision mesh.
3. Compress: **Draco** for geometry, **KTX2/basis** for textures. Keeps the download small.

```bash
npm i three three-mesh-bvh nipplejs
# build tool: vite
```

### 1. Core (renderer, camera, loop)
```js
import * as THREE from 'three';
const renderer = new THREE.WebGLRenderer({ antialias:true });
renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
renderer.setSize(innerWidth, innerHeight);
renderer.outputColorSpace = THREE.SRGBColorSpace;
renderer.toneMapping = THREE.ACESFilmicToneMapping;
document.body.appendChild(renderer.domElement);

const scene = new THREE.Scene();
scene.fog = new THREE.FogExp2(0x0a0712, 0.02);
const camera = new THREE.PerspectiveCamera(70, innerWidth/innerHeight, 0.1, 200);
addEventListener('resize', ()=>{ camera.aspect=innerWidth/innerHeight; camera.updateProjectionMatrix(); renderer.setSize(innerWidth, innerHeight); });

const clock = new THREE.Clock();
function loop(){ requestAnimationFrame(loop);
  const dt = Math.min(clock.getDelta(), 0.05);   // clamp to avoid tunnelling
  update(dt); renderer.render(scene, camera);
}
loop();
```

### 2. Load the glb (visual + collision)
```js
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader.js';
const draco = new DRACOLoader(); draco.setDecoderPath('https://www.gstatic.com/draco/v1/decoders/');
const gltf = new GLTFLoader(); gltf.setDRACOLoader(draco);

let collider;                       // BVH-backed collision mesh
gltf.load(VISUAL_URL, g => scene.add(g.scene));
gltf.load(COLLISION_URL, g => { collider = buildCollider(g.scene); });   // not added to scene (invisible)
```

### 3. Collision via three-mesh-bvh (capsule vs BVH)
```js
import { MeshBVH, acceleratedRaycast } from 'three-mesh-bvh';
THREE.Mesh.prototype.raycast = acceleratedRaycast;

function buildCollider(root){
  const geoms=[]; root.updateMatrixWorld(true);
  root.traverse(o=>{ if(o.isMesh){ const g=o.geometry.clone(); g.applyMatrix4(o.matrixWorld); geoms.push(g); } });
  // merge geoms (BufferGeometryUtils.mergeGeometries) → one geometry
  const merged = mergeGeometries(geoms);
  merged.boundsTree = new MeshBVH(merged);
  return new THREE.Mesh(merged);
}

// player = capsule (segment start/end + radius); resolve against BVH each frame
const player = { pos:new THREE.Vector3(0,1.6,6), vel:new THREE.Vector3(), onGround:false, r:0.35, h:1.6 };
const seg = new THREE.Line3(), tri = new THREE.Triangle(), tmp = new THREE.Vector3(), box = new THREE.Box3();
function movePlayer(dt){
  player.vel.y += -18 * dt;                         // gravity
  player.pos.addScaledVector(player.vel, dt);
  // capsule segment in world space
  seg.start.copy(player.pos); seg.end.copy(player.pos); seg.end.y -= (player.h - player.r); seg.start.y -= player.r;
  box.makeEmpty(); box.expandByPoint(seg.start); box.expandByPoint(seg.end); box.expandByScalar(player.r);
  player.onGround=false;
  collider.geometry.boundsTree.shapecast({
    intersectsBounds: b => b.intersectsBox(box),
    intersectsTriangle: t => {
      const dist = t.closestPointToSegment(seg, tmp, /*out*/new THREE.Vector3());
      if(dist < player.r){ const depth=player.r-dist; /* push capsule out along normal; if mostly-up contact → onGround=true, vel.y=0 */ }
    }
  });
}
```
(Use the official three-mesh-bvh "character controller" example as your template; the shape is exactly this — segment-vs-triangle shapecast + push-out + ground check.)

### 4. First-person controls (desktop) + jump
```js
let yaw=0, pitch=0;
renderer.domElement.addEventListener('click', ()=> renderer.domElement.requestPointerLock());
addEventListener('mousemove', e=>{ if(document.pointerLockElement){ yaw -= e.movementX*0.0022; pitch = Math.max(-1.3, Math.min(1.3, pitch - e.movementY*0.0022)); }});
const keys={}; addEventListener('keydown',e=>{keys[e.code]=1; if(e.code==='Space'&&player.onGround) player.vel.y=7;}); addEventListener('keyup',e=>{keys[e.code]=0;});
function applyInput(dt){
  camera.rotation.set(pitch, yaw, 0, 'YXZ');
  const f=(keys.KeyW?1:0)-(keys.KeyS?1:0), s=(keys.KeyD?1:0)-(keys.KeyA?1:0), spd=4;
  const dir=new THREE.Vector3(s,0,-f).applyEuler(new THREE.Euler(0,yaw,0)).normalize().multiplyScalar(spd*dt);
  player.pos.x+=dir.x; player.pos.z+=dir.z;
  camera.position.copy(player.pos);
}
```

### 5. Artwork boards + look-to-inspect
```js
const ray = new THREE.Raycaster(); const boards=[];
ARTWORKS.forEach(a=>{
  const tex = new THREE.TextureLoader().load(a.img); tex.colorSpace=THREE.SRGBColorSpace;
  const m = new THREE.Mesh(new THREE.PlaneGeometry(a.w, a.h), new THREE.MeshBasicMaterial({map:tex}));
  m.position.set(a.x,a.y,a.z); m.rotation.y=a.ry; m.userData=a; scene.add(m); boards.push(m);
});
function checkGaze(){
  ray.setFromCamera({x:0,y:0}, camera);                 // center of screen
  const hit = ray.intersectObjects(boards)[0];
  showPanel(hit && hit.distance < 5 ? hit.object.userData : null);   // HTML overlay: title/author/blurb
}
```

### 6. Crisp labels in 3D (optional) — CSS3DRenderer
Render real HTML (sharp text, links) positioned in the 3D world, layered over the WebGL canvas:
```js
import { CSS3DRenderer, CSS3DObject } from 'three/examples/jsm/renderers/CSS3DRenderer.js';
const css = new CSS3DRenderer(); css.setSize(innerWidth, innerHeight);
css.domElement.style.cssText='position:fixed;inset:0;pointer-events:none';
document.body.appendChild(css.domElement);
// per label: new CSS3DObject(divEl), position/scale into the scene; render css.render(scene,camera) in the loop too.
```

### 7. Mirror floor (optional) — Reflector
```js
import { Reflector } from 'three/examples/jsm/objects/Reflector.js';
const floor = new Reflector(new THREE.PlaneGeometry(40,40), { textureWidth:1024, textureHeight:1024, color:0x222233 });
floor.rotateX(-Math.PI/2); scene.add(floor);
```

### 8. Mobile — nipplejs joystick + touch look
```js
import nipplejs from 'nipplejs';
const stick = nipplejs.create({ zone:document.getElementById('joy'), mode:'static', position:{left:'70px',bottom:'70px'} });
let move={x:0,y:0}; stick.on('move',(_,d)=>{ move.x=Math.cos(d.angle.radian)*d.force; move.y=Math.sin(d.angle.radian)*d.force; }); stick.on('end',()=>move={x:0,y:0});
// feed move.x/move.y into applyInput; use a second touch on the right half of the screen for look (track touchmove deltas → yaw/pitch).
```

### 9. Audio (starts on first gesture — autoplay is blocked)
```js
const listener = new THREE.AudioListener(); camera.add(listener);
const sound = new THREE.Audio(listener);
addEventListener('pointerdown', ()=>{ if(!sound.isPlaying){ new THREE.AudioLoader().load(TRACK, b=>{ sound.setBuffer(b); sound.setLoop(true); sound.setVolume(0.4); sound.play(); }); } }, {once:true});
// For per-artwork sound use THREE.PositionalAudio attached to the board mesh.
```

### 10. Performance + fallback
- Cap `pixelRatio` at 2; Draco geometry + KTX2 textures; dispose unused assets.
- One merged collision mesh with a BVH beats many colliders.
- Frustum culling is on by default — keep meshes separate enough to benefit.
- **Always** render a plain HTML list of the works (links + text) as a `<noscript>` / reduced-motion / low-GPU fallback. WebGL is invisible to assistive tech.

> Reference implementation to study (don't copy): a third-party Three.js exhibition (Steve245270533, GPL-3.0 — not bundled here) — © Steve245270533, GPL-3.0. Keep its LICENSE and credit if you reuse any code.

---

# Three.js single-file walkable rooms (no build)

The DEFAULT WebGL path. One HTML file, three over a CDN importmap, no bundler. This is exactly how `the Atrium reference build (not bundled in this public repo — full code is inline below)` works — an original, copy-freely reference. All snippets below are distilled from it.

## Importmap boot (no bundler)

```html
<script type="importmap">
{ "imports": {
    "three": "https://unpkg.com/three@0.150.1/build/three.module.js",
    "three/addons/": "https://unpkg.com/three@0.150.1/examples/jsm/"
} }
</script>
<script type="module">
  import * as THREE from 'three';
  import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
  import { RoomEnvironment } from 'three/addons/environments/RoomEnvironment.js';
  // ...everything else goes here. No build step, no npm.
</script>
```
Pin the version. unpkg and jsdelivr both work. `three/addons/` must keep its trailing slash.

## Room shell + texture helper

```js
// repeatable texture loader — encodeURI handles spaces / non-breaking spaces in AI-exported filenames
function TX(path, rx, ry){ var t=new THREE.TextureLoader().load(encodeURI("assets/2d_art/"+path));
  t.wrapS=t.wrapT=THREE.RepeatWrapping; t.repeat.set(rx,ry); return t; }
// one-liner mesh placer (note: rx/ry are rotations, applied only if truthy)
function mk(g,m,px,py,pz,rx,ry){ var o=new THREE.Mesh(g,m); o.position.set(px,py,pz);
  if(rx)o.rotation.x=rx; if(ry)o.rotation.y=ry; scene.add(o); return o; }

// a 6-plane room at centre X, width W, height H, length L
var wall=new THREE.MeshStandardMaterial({map:TX("texture/wallpaper.png",4,2.4),roughness:.85});
var floor=new THREE.MeshStandardMaterial({map:TX("texture/floor.png",3,3),roughness:.9});
var ceil=new THREE.MeshStandardMaterial({map:TX("texture/ceiling.png",2,2),roughness:1});
mk(new THREE.PlaneGeometry(W,L), floor, X,0,0, -Math.PI/2,0);
mk(new THREE.PlaneGeometry(W,L), ceil,  X,H,0,  Math.PI/2,0);
mk(new THREE.PlaneGeometry(L,H), wall, X-W/2,H/2,0, 0, Math.PI/2);
mk(new THREE.PlaneGeometry(L,H), wall, X+W/2,H/2,0, 0,-Math.PI/2);
mk(new THREE.PlaneGeometry(W,H), wall, X,H/2,-L/2, 0,0);
mk(new THREE.PlaneGeometry(W,H), wall, X,H/2, L/2, 0,Math.PI);
```
For tileable textures, generate **square, seamless, flat-lit, no-shadow** swatches (the prompt words that matter). Product-shot PNGs on a grey background do NOT tile.

## GLB place() helper (auto-scale, recentre, rest on a surface)

```js
// mode 'floor' = rest model base at y; 'center' = centre model at y (wall-mounted pieces).
// tn = flat tint colour; texUrl = wrap the model in a 2D image instead.
function place(url,x,y,z,tw,rotY,mode,tn,rough,metal,texUrl){
  new GLTFLoader().load(encodeURI(url), function(g){
    var m=g.scene; m.rotation.y=rotY||0;
    var b=new THREE.Box3().setFromObject(m), sz=new THREE.Vector3(); b.getSize(sz);
    m.scale.setScalar(tw/(sz.x||1));                 // fit to target width
    b.setFromObject(m); var c=new THREE.Vector3(); b.getCenter(c);
    var py = mode==='center' ? (y-c.y) : (y-b.min.y); // centre, or sit base at y
    m.position.set(x-c.x, py, z-c.z);
    if(tn!=null) tint(m,tn,rough,metal);
    if(texUrl) wrap(m,texUrl,rough,metal);
    scene.add(m);
  }, undefined, function(e){ console.warn('glb fail',url,e); });
}
```

## Fix grey models (tint or texture-wrap)

```js
// TINT — force a flat PBR colour (upholstery, painted wood). Nulls the map so the colour shows.
function tint(m,hex,rough,metal){ m.traverse(function(o){ if(o.isMesh&&o.material){
  (Array.isArray(o.material)?o.material:[o.material]).forEach(function(mm){
    if(mm.color)mm.color.setHex(hex); if('map' in mm)mm.map=null;
    if('roughness' in mm)mm.roughness=(rough==null?.6:rough);
    if('metalness' in mm)mm.metalness=(metal==null?.1:metal); mm.needsUpdate=true; }); } }); }

// WRAP — apply the original 2D reference image as the model's map (clock face, framed art).
// Works when the model's UVs/silhouette match a flat front-on product image.
function wrap(m,texUrl,rough,metal){ var tt=new THREE.TextureLoader().load(encodeURI(texUrl));
  tt.colorSpace=THREE.SRGBColorSpace; m.traverse(function(o){ if(o.isMesh&&o.material){
  (Array.isArray(o.material)?o.material:[o.material]).forEach(function(mm){
    if('map' in mm)mm.map=tt; if(mm.color)mm.color.setHex(0xffffff);
    if('roughness' in mm)mm.roughness=(rough==null?.55:rough);
    if('metalness' in mm)mm.metalness=(metal==null?.2:metal); mm.needsUpdate=true; }); } }); }
```
Rule of thumb: tint solid-colour objects (sofa, chairs, desk); texture-wrap things whose detail IS a flat image (a clock face, a framed painting). Tinting flattens multi-colour detail (gold trim, leather inlay) to one colour — if you need that detail back, re-export the model WITH textures.

## Measure-then-stack (props on a desk)

```js
// Load the host (desk) first; rest props on its MEASURED top, never a guessed height.
new GLTFLoader().load(encodeURI(DESK), function(g){
  var m=g.scene, b=new THREE.Box3().setFromObject(m), sz=new THREE.Vector3(); b.getSize(sz);
  m.scale.setScalar(2.6/(sz.x||1));
  b.setFromObject(m); var c=new THREE.Vector3(); b.getCenter(c);
  m.position.set(X-c.x, -b.min.y, Zdesk-c.z); tint(m,0x3a2616,.55,.18); scene.add(m);
  b.setFromObject(m); var topY=b.max.y;            // <-- the real desktop height
  place(LAMP,  X-0.85, topY, Zdesk-0.1, 0.6, 0,'floor', 0x9c7c3c,.35,.85);
  place(CLOCK, X+0.85, topY, Zdesk-0.1, 0.42, Math.PI,'floor', null,.5,.35, CLOCK_IMG);
});
```

## First-person controls + per-room clamp

```js
var player={pos:new THREE.Vector3(0,1.6,4), yaw:0, pitch:0}, keys={}, locked=false;
addEventListener('keydown',e=>keys[e.code]=1); addEventListener('keyup',e=>keys[e.code]=0);
renderer.domElement.addEventListener('click',()=>renderer.domElement.requestPointerLock());
document.addEventListener('pointerlockchange',()=>locked=document.pointerLockElement===renderer.domElement);
addEventListener('mousemove',e=>{ if(!locked)return;
  player.yaw-=e.movementX*0.0022;
  player.pitch=Math.max(-1.2,Math.min(1.2,player.pitch-e.movementY*0.0022)); });

function tick(){
  var sp=0.06, fwd=(keys.KeyW?1:0)-(keys.KeyS?1:0), str=(keys.KeyD?1:0)-(keys.KeyA?1:0);
  player.pos.x += (-Math.sin(player.yaw)*fwd + Math.cos(player.yaw)*str)*sp;
  player.pos.z += (-Math.cos(player.yaw)*fwd - Math.sin(player.yaw)*str)*sp;
  // clamp inside the current room's AABB — no physics engine needed for boxy rooms
  if(curRoom){ var r=ROOMS[curRoom], mg=0.4;
    player.pos.x=Math.max(r.x-r.w/2+mg, Math.min(r.x+r.w/2-mg, player.pos.x));
    player.pos.z=Math.max(-r.l/2+mg,    Math.min(r.l/2-mg,    player.pos.z)); }
  camera.position.copy(player.pos);
  camera.rotation.set(player.pitch, player.yaw, 0, 'YXZ');
  renderer.render(scene,camera); requestAnimationFrame(tick);
}
```
Mobile: touch-drag on a full-screen `div` updates yaw/pitch; a hold-to-walk button sets a `walking` flag that drives forward motion.

## Rooms behind paintings (portal + fade)

```js
// each interior is built offstage at its own X so rooms never overlap
var ROOMS={ suite64:{x:300,w:8,l:9,ez:9/2-1.2}, office:{x:340,w:7,l:8,ez:8/2-1.2} };
var curRoom=null, returnPos=new THREE.Vector3(), returnYaw=0;
var fade=document.createElement('div'); fade.id='fade'; document.body.appendChild(fade); // black, opacity-transition

function enterRoom(key){ var r=ROOMS[key]; if(!r) return; fade.classList.add('on');
  setTimeout(function(){
    returnPos.copy(player.pos); returnYaw=player.yaw;       // remember the hall
    player.pos.set(r.x,1.6,r.ez); player.yaw=0; player.pitch=0; curRoom=key;
    scene.environment=roomEnv;                              // soft PBR only inside rooms
    setTimeout(()=>fade.classList.remove('on'),80);
  },520); }
function exitRoom(){ fade.classList.add('on');
  setTimeout(function(){ player.pos.copy(returnPos); player.yaw=returnYaw; player.pitch=0;
    curRoom=null; scene.environment=null; setTimeout(()=>fade.classList.remove('on'),80); },520); }

// portrait planes carry userData.portal; an exit plane carries userData.exit
var ray=new THREE.Raycaster();
function interact(){ ray.setFromCamera({x:0,y:0},camera);
  var hit=ray.intersectObjects(boards)[0]; if(!hit||hit.distance>6.5) return;
  var u=hit.object.userData; if(u.exit) exitRoom(); else if(u.portal) enterRoom(u.portal); }
```
The hall portraits live in one `ART` array; tag the two you want enterable with `portal:"suite64"` / `portal:"office"`. Reorder the array to change which painting is "first."

## Lighting budget (don't blow out small rooms)

```js
scene.add(new THREE.HemisphereLight(0x223530,0x000000,0.22));   // faint base
var lantern=new THREE.SpotLight(0xffe1a6,30,24,0.62,0.5,1.3);   // follows the camera
// per frame: lantern follows camera, dimmer inside rooms, with a flicker
lantern.position.copy(camera.position);
lantern.target.position.copy(camera.position).addScaledVector(dir,5);
lantern.intensity = 30 * flick * (curRoom ? 0.3 : 1);
// room accents stay LOW: point lights ~0.8–1.8 with short range (3.5–5.5), not 3+ at range 12
```
If a wall reads white: cut the nearest light's **intensity and range** first; only then touch materials. With `RoomEnvironment`, keep `envMapIntensity ≈ 0.2` on every material or it washes out.

```js
// soft reflections without an HDR file:
var pmrem=new THREE.PMREMGenerator(renderer);
var roomEnv=pmrem.fromScene(new RoomEnvironment(),0.04).texture; // assign to scene.environment inside rooms
```

> Reference to copy from (original, MIT-spirit — it's yours): `the Atrium reference build (not bundled in this public repo — full code is inline below)`.
