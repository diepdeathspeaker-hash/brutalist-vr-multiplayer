# BRUTALISTICK WEBXR (work-in-progress port)

Standalone WebXR physics-combat sandbox for the **Meta Quest browser** (Three.js + Rapier + Vite), with a desktop test mode.
Fictional cartoon violence (blood, dismemberment). Medieval arena theme.

## Run
Three ways (pick one):
1. **Vercel (recommended):** the folder that contains `package.json` must be the repository root / Vercel "Root Directory". Framework preset *Vite*; Vercel runs `npm install` + `npm run build` and serves `dist/`.
2. **Dev:** `npm install && npm run dev`.
3. **No-build static hosting:** upload the project folder as-is (keep `index.html`, `src/`, `public/` assets at `models/`, `audio/`, `textures/` next to index.html -> copy `public/*` to the root). `index.html` has an import map that loads three.js and Rapier from the jsdelivr CDN, so no build is needed. It must be served over http(s), never opened from file://.
```bash
npm install
npm run dev        # desktop test: http://localhost:5173   (npm test = slicer unit test)
npm run build      # outputs dist/
```
### Quest
WebXR needs HTTPS. Easiest: deploy to Vercel (below) and open the URL in the Quest Browser, then press **Enter VR**.
For local testing: `npm run dev`, then use `adb reverse tcp:5173 tcp:5173` and open `http://localhost:5173` in the Quest browser
(localhost counts as a secure context).

### Vercel
Push the folder to a Git repo and import it in Vercel (framework preset: Vite; `vercel.json` is included), or run `npx vercel --prod`.

## Controls
**VR:** left stick move · right stick snap-turn · grip = grab (physical, throwable, two-handed on weapons) · right trigger presses the
wrist menu on your left controller (spawn NPC/sword/dagger/mace/crate/barrel, reset).
Punch/throw things at NPCs for blunt damage. Stab = thrust the blade tip into a body part; slash = swing fast.
**Desktop:** WASD move · mouse look · hold LMB + move = swing hand · E grab/release · G sword · V dagger · M mace · N NPC · C crate · R reset.

## Gameplay
Terminal (east side of the arena, x=3.6): aim the controller ray and pull the trigger (desktop: look at it and press **Q**).
Tabs: Missions (accept/claim), Shop (Machete $250, Glock $500, Shotgun $900), Spawn, Settings. Money persists in localStorage.
Guns: grab with grip, pull trigger to fire (desktop: E then F). Shotgun at close range can split a head.
Details of what is/isn't verified: see `AUDIT.md`.

## Weapons (all in the shop except sword/dagger/mace)
Melee: kitchen knife, combat knife, machete, axe, sword, dagger (sharp - progressive slicing) · bat, crowbar, hammer, mace (blunt, knockback by mass).
Firearms: Glock, revolver, Desert Eagle, SMG, assault rifle, AK, shotgun, double barrel, sniper - each with its own mag size, fire rate, auto/semi, recoil, sound.
### Reloading (VR)
Grip-grab the gun. **A/X button** = eject magazine (mag guns) / open cylinder (revolver) / unload (shotguns, sniper).
Reach to your **hip pouches** (left/right, ~0.7 m below your head) and squeeze to pull ammo for the gun you last held, then push it into the gun's well/port:
mag guns need the old mag ejected first; revolver needs the cylinder open + speedloader; shotguns take shells one at a time; sniper takes a clip.
Ammo comes from your reserve (buy packs in the terminal AMMO tab). Desktop: **T** eject, **R** automatic reload sequence.
### Progressive slicing
A sharp blade cuts only as far as it has actually travelled through the part (perpendicular to the blade, faster + sharper = faster). A groove is carved while you cut,
the blade drags against your hand, and the part separates only when you finish the stroke. Pull out early = partial wound (re-enter the same wound within ~4 s to continue). Stabs (thrust along the blade) embed the weapon.
## Arena Mode
Terminal ARENA tab: set enemies per wave, START ARENA, then START WAVE for each wave. Enemies chase and punch you (player health on the scoreboard on the north wall); wave rewards go to your money. END ARENA returns to the sandbox.
## Measuring performance on a Quest (honest procedure)
No Quest numbers are shipped because none have been measured. To measure: open the game in the Quest browser, enter VR, go to the terminal -> SETTINGS -> turn the Performance panel ON (shows fps, frame ms, p95, physics ms, NPC ms, bodies, draws, triangles, refresh rate)
and press RUN BENCH (20 s, 8 NPCs). Results are shown, logged (`[perf] benchmark`) and saved to localStorage (`brutalistick_perf`). For GPU/CPU detail use the Meta Quest Developer Hub + OVR Metrics Tool.
Tuning knobs: VR render scale (Settings), NPC cap (14) and far-NPC half-rate update in `main.js`, decal/debris pools in `blood/impacts.js`.

## Online multiplayer
See **MULTIPLAYER.md** (server setup on Render/any Node host, Vercel config, room codes, two-headset test plan, what was and was not tested).

## Graphics (RTX-style approximations - NOT hardware ray tracing)
Tier is chosen automatically (Quest browser -> **quest**, otherwise **high**; override in Settings > Graphics: AUTO/QUEST/HIGH/ULTRA).
- all tiers: ACES filmic tone mapping, image-based reflections from a generated arena environment map (metal weapons, wet floor, blood), PBR stone/wall materials with generated normal + roughness maps, baked floor ambient occlusion, soft blob AO under characters/props, flickering torches, dust motes, exponential fog.
- quest: 1024 shadow map, no post-processing (WebXR cannot run full-screen post passes), standard materials.
- high (desktop): + GTAO screen-space ambient occlusion, bloom, 2048 shadows, clearcoat (wet look) NPC materials.
- ultra (desktop): + 4096 shadows, 2 shadow-casting torches, stronger AO/bloom, pixel ratio up to 2. Desktop tiers auto-downgrade after ~4 s of slow frames.
No screen-space reflections or ray-traced GI are implemented.

## Status (honest)
Nothing here has been run in a browser or on a Quest — the build environment had no network, so `npm install`, the Vite build and the
physics tuning are **untested**. Only the mesh slicer has an automated test (`npm test`: slicer, bullet-hole geometry, money/missions/shop/ammo logic, static boot checks, and a stubbed smoke test of every system via `sh test/smoke/run.sh`).
See the checklist in the chat reply / `docs` for per-feature status. Tuning constants for the ragdoll are at the top of `src/npcs/npc.js`.

## Credits / licences
- `public/models/sword.glb` — "Medieval Sword - Free to Use" by ENOMIC (Sketchfab), **CC-BY-NC-4.0** (non-commercial only; textures downscaled to 1024px).
- Audio (`public/audio`) and blood decal sheet (`public/textures`) came from your gameassets.zip — check their licences before publishing.
- `als_ragdoll.glb` (samllea1, CC-BY-4.0) was provided but is not used; NPCs are procedural.
