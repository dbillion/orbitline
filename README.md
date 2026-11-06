# Orbitline — Spaceport Mission Control

> **World 02** of the World Prompt Pack · Electric violet `#7C5CFF` · Dark-mode first

🌐 **Live:** https://dbillion.github.io/orbitline/

---

## What It Is

Orbitline is a live 3D spaceport mission-control dashboard for **Constellation Spaceport** on a tropical coast. It feels like a strategy game: everything moves autonomously, the data is realistic, and the UI is a frosted-glass overlay on top of a Three.js 3D scene.

---

## How the Polygons Were Built Into This File

Every 3D object in Orbitline is **procedurally generated from raw Three.js geometry primitives** — zero external model files (`.gltf`, `.obj`, etc.) were loaded. All geometry is defined as JavaScript inside `index.html`.

### Primitive palette

| Primitive | Three.js class | Used for |
|-----------|----------------|----------|
| Box | `BoxGeometry(w,h,d)` | Buildings, VAB, structural bars, crawlerway, rocket body panels |
| Cylinder | `CylinderGeometry(rt,rb,h,seg)` | Tower columns, propellant tanks, rocket fuselage, lighthouse, palm trunks |
| Sphere | `SphereGeometry(r,ws,hs)` | Propellant spheres, water tower tanks, antenna dishes, person heads |
| Cone | `ConeGeometry(r,h,seg)` | Rocket nose cone, palm frond leaves, gull bodies |
| Torus | `TorusGeometry(r,tube,rs,ts)` | Landing zone glowing rings |
| Plane | `PlaneGeometry(w,h,wSegs,hSegs)` | Ground, ocean (animated), sand beach |
| Points | `BufferGeometry` + `PointsMaterial` | Vapour venting particles, rocket exhaust smoke, stars, bioluminescent surf |

### Rocket assembly process

```
CylinderGeometry  → Stage 1 body (r=1.5, h=24)
CylinderGeometry  → Stage 2 body (r=1.4, h=16)
ConeGeometry      → Nose fairing (r=1.4, h=8)
BoxGeometry       → 4× fins, interstage ring
CylinderGeometry  → Engine bell (flared)
BoxGeometry       → 4× deployable landing legs (rotation.z animated on descent)
ConeGeometry      → Exhaust plume (inverted, MeshLambertMaterial emissive glow)
PointLight        → Flame light (dynamic colour/intensity during burn)
```

All parts are added to a `THREE.Group` so the entire rocket translates and rotates as one unit.

### Launch pad assembly

The lattice tower uses **8 horizontal cross-braces** and **4 vertical column boxes** offset at 45° angles to fake a truss structure — no actual lattice geometry:

```js
for(let i=0;i<8;i++){
  // vertical stack
  box(0.5, 4, 0.5) at y = 2+i*4   // riser
  if(i%2===0){
    box(8, 0.4, 0.4)  // X-axis crossbar
    box(0.4, 0.4, 8)  // Z-axis crossbar
  }
}
```

### Ocean animation

The ocean uses a subdivided `PlaneGeometry(400,120, 30,10)` with Gerstner-inspired vertex displacement in the render loop:

```js
oceanPos.setY(i,
  Math.sin(ox*0.3 + t) * 0.4 +
  Math.sin(oz*0.2 + t*1.3) * 0.3
);
```

The original vertex positions are cached in `oceanPosOrig` so the displacement is applied relative to rest position each frame — no GC allocation inside the loop.

---

## How the Mission Loop Works

```
Idle → Rollout (crawler carries rocket 50m along crawlerway)
     → Fueling (vapour vent particles, 60s)
     → Terminal Count (10-second countdown)
     → Ignition (exhaust cone + PointLight flame)
     → Liftoff + Ascent (parabolic trajectory, 25s)
     → Stage Separation (boosterGroup splits off)
     → Booster Flip (rotation.x += 0.025 per frame)
     → Re-entry Burn (descend toward landing zone)
     → Booster Landing (legs deploy, touch-down)
     → Reset → loop
```

---

## Performance Design

Lessons carried forward from **Gateflow** (World 01):

| Issue | Fix |
|-------|-----|
| `new THREE.Vector3()` in render loop → GC thrash | Pre-allocated `_v3`, `_v3b`, `_q` scratch objects |
| DOM updates every frame → reflow | Throttled to 500 ms with `domTimer` |
| Particle system allocating per frame | Pre-allocated `Float32Array`, mutate in-place each frame |
| Shadow map overdraw | 1024×1024 max, one `DirectionalLight` with shadow |
| Rain on WebGL canvas → GPU stall | Rain drawn on a separate 2D `<canvas>` overlay |
| Ocean subdivision jank | Updated only on even frames (`frameCount%2===0`) |

---

## Stack

- **Three.js r128** — local `js/three.min.js` (CDN fallback: `cdnjs.cloudflare.com`)
- **OrbitControls r128** — local `js/OrbitControls.js`
- **Vanilla CSS** — no Tailwind, no frameworks
- **Single `index.html`** — all CSS and JS inline, no build step

---

## Controls

| Input | Action |
|-------|--------|
| Left-drag | Pan camera |
| Right-drag | Orbit / rotate |
| Scroll / pinch | Zoom |
| Click object | Select → glass telemetry panel + camera glide |
| Area switcher | Fly to Pads / Assembly / Mission Control / Landing Zone / Coast |
| Search box | Type object name + Enter |
| ☀️ Clear / ⛈️ Monsoon / 🌫️ Fog | Weather system |
| 🌙 / ☀️ | Day / night toggle (stars, moon, bioluminescent surf) |

---

*Part of the [World Prompt Pack](https://x.com/sanjai.builds) — 17 interactive 3D strategy-game dashboards.*
