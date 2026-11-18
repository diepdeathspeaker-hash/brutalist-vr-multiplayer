# Feature status (code review + automated tests only — NOT run in a browser or on a Quest)

Legend: **T** = covered by Node tests · **S** = executed under the stubbed smoke test (no undefined names / wiring errors, physics & rendering are fake) · **U** = implemented but never run for real.

| Feature | Status |
|---|---|
| Loading manager / boot stages | T+S, U in browser |
| NPC active ragdoll (stand, wander, stumble, fall, get up) | S, **U** (tuning unknown) |
| Real mesh slicing (closed halves, red caps) | T (geometry) |
| **Real holes**: layered anatomy (skin, flesh, red cavity lining, bone, organs), holes carved by true region shape with real walls | T (geometry), S (carving on every NPC part), **U** visually |
| Cut faces: proper loop triangulation, skin/flesh/bone rings | T |
| **Progressive slicing** (progress by travelled distance, groove carving, drag, resume, no auto-finish) | S, **U** — the maths/feel needs your hands-on tuning (`cutStep` in weapon.js) |
| Bullet perforation (entry/exit holes) | T (geometry) |
| 19 weapons: 9 firearms, 10 melee, all priced / spawnable / data-driven | S, U |
| Ammo: mags, reserve pool, shop ammo, physical reload (eject, pouch, insert) | S, **U** |
| Arena Mode (waves, hostile NPCs, health, rewards, scoreboard, start-wave button) | S (wave/reward flow), U |
| NPC speech (formant voice + bubbles), reactions, footsteps; player footsteps; torch crackle | S, U — voice is synthesized gibberish with readable subtitles, not recorded speech |
| Bullet marks on walls/floors (pooled), debris, impact sounds | S, U |
| Hats & glasses as physical bodies on the head | S, U |
| Performance monitor, benchmark, LOD, caps | S, U — **no Quest measurements exist** |
| Money / missions / shop logic | T |

Known gaps: no grenades/explosives, melee weapons have no individual swing animations (physics only), surface variation is limited to pitch/volume (one floor material), clothing is vertex-coloured (not separate garments).

## Update: cutting/visual fixes (root causes found by code inspection - the testing video was not available to me)
1. *Clear spots*: bullet/slice holes only deleted triangles of a hollow one-layer shell, so you saw through to the world behind. Now every part has interior layers (flesh, red cavity lining, bone, organs) built on first damage, and holes get real wall geometry.
2. *Rectangular holes / red objects sticking out*: the cut-cap code fanned triangles from the centroid of ALL cut points, which produced huge stray shapes on any mesh that already had holes; holes also removed whole large (quad-sized) triangles. Caps are now built from real loops (ear-clipped) per layer, holes are carved on a subdivided mesh in the shape of the weapon, with skirts built from the actual hole boundary.
3. *Slice treated as a projectile*: slashes reused the bullet-hole routine (round tube) and the ray 'stab' test fired on any swing with the tip leading. Now blades have their own cross-section (bladeW x bladeT x kerf) swept along the real path, stabs need a true thrust (and only stab-capable blades), machete/axe never stab.
4. *Cutting too hard*: the blade was sampled once per physics step, so fast swings jumped over thin limbs. It is now swept in sub-steps (<=2 cm) and needs less travel.
5. *Accessories*: hats/glasses use fixed (non-swinging) joints and are re-hosted onto whichever head piece holds them.
Graphics are approximations (see README), not ray tracing. Not verified visually: everything above is U apart from the tests listed.

## Update: online multiplayer (see MULTIPLAYER.md)
| Feature | Status |
|---|---|
| Authoritative server (rooms, codes, limits, ownership, damage/death/respawn, scores, economy) | **T** (36 checks, real sockets) |
| Client: connect, create/join, reconnect, latency, interpolation | **T** (15 checks) |
| Game integration (host announces world, NPC/pose streaming, remote spawn/wounds, damage/death/respawn, reconnect rebuild) | **S+real server** (15 checks, 3D/physics stubbed) |
| Player avatars (head/torso/arms/hands/legs, IK, crouch, same layered cuttable body) | S, **U** visually |
| Player-vs-player progressive cutting via the same NPC cutting code + events | S (event path), **U** feel; concurrent cuts can diverge |
| Shared grabbing/throwing/hand-off, NPC replication (walking = FK, ragdoll = part snapshots) | **U** |
| PvP arena, co-op arena waves/rewards, shared sandbox, host-only room settings | server logic **T**, client UI S/U |
| Co-op revive teammates | **Missing** |
| Real two-headset test, bandwidth/latency numbers | **Not done** (test plan provided) |
