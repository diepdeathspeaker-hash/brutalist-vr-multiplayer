# Online multiplayer: setup, use, testing

## How it works (short)
* `server/` is a **zero-dependency Node server** (Node 18+): HTTP for the built game + a WebSocket endpoint `/ws`. Vercel (static hosting) cannot hold WebSocket connections, so the multiplayer server must run on a Node host.
* The room **host** (whoever created it) simulates the NPCs and all un-held objects. Whoever **holds** an object simulates it (the server arbitrates ownership: two players can never control one object; weapons can be handed between players).
* Wounds, cuts and pops are tiny events relayed by the server in order (and replayed to late joiners); every client re-applies them to the same deterministic geometry with the same slicer, so everyone sees the same holes and severed parts. No meshes are ever sent.
* The **server decides** health, damage (clamped, rate-limited, reach-checked, rules-checked), deaths, respawns, spawn protection, scores, PvP matches, room settings, and every player's money / weapons / ammo / mission rewards. The server does not simulate physics; clients report what their blade or bullet hit.

## Option A - one service does everything (simplest, recommended)
1. Push this folder to GitHub. 2. On https://render.com: New > Blueprint (it reads `render.yaml`) or New > Web Service with Build `npm install && npm run build`, Start `npm start`.
3. Open the Render URL in the Quest Browser. That's it: the page is HTTPS, the socket is automatically WSS, no configuration needed.
(Fly.io / Railway / any VPS: `Dockerfile` is included, or run `npm install && npm run build && npm start`; put TLS in front, e.g. the platform's HTTPS.)
Note: free tiers sleep when idle; the first connection after a pause can take ~30-60 s.

## Option B - game on Vercel, server elsewhere
1. Deploy the server as in Option A (you can ignore the game it also serves). 2. In Vercel > Settings > Environment Variables add `VITE_MP_URL = wss://YOUR-SERVER/ws` and redeploy.
(Or open the game once with `?server=wss://YOUR-SERVER/ws`; it is remembered on that device.)

## Local testing (no Quest needed)
```bash
npm install && npm run build && npm start      # http://localhost:8080 serves the game and the server
```
Open it in two browser tabs/windows (desktop mode). Tab 1: terminal (look at the kiosk, press Q) > ONLINE > CREATE ROOM. Tab 2: ONLINE > JOIN ROOM > spell the 5-letter code.
Dev server: `npm run dev` + `npm run server`, then open `http://localhost:5173/?server=ws://localhost:8080/ws`.

## Using it in VR
Terminal (east side of the arena) > **ONLINE**. *Create room* gives a 5-letter code; friends choose *Join room* and tap the letters with the controller ray. The room page lists players (kills/deaths/health), lets the host switch **Shared Sandbox / PvP Arena / Co-op Arena**, and has **Room settings** (host only): player damage, PvP, player mesh cutting, friendly fire, spawn protection (+seconds), weapon sharing, respawn timer, max players, PvP score/time limit. *Leave room* restores your single-player sandbox. Your outfit/skin are chosen on the same page before connecting.
* If your connection drops, the game reconnects automatically (same player slot kept for 60 s). Money/weapons/ammo belong to your device id (stored in this browser; clearing site data = new account).

## Two-headset test plan (please run this; it has NOT been run on real headsets)
1. Both open the same URL; A creates a room, B joins with the code. Both appear in the player list; you see each other's head, torso, arms, hands, legs and walking.
2. A grabs the machete from the table and throws it: B sees the same machete fly. B picks up a dropped weapon; A hands B a weapon (grab it from A's hand).
3. Both cut the same NPC; B sees A's progressive cut, severed parts, hats falling. 4. Switch to PvP Arena: cut each other; check death, countdown, respawn with protection, scoreboard.
5. Turn *Player mesh cutting* off and confirm cuts on players are refused. 6. Co-op: host starts wave 1 (ARENA tab > START WAVE), clear it, check both get the reward. 7. Buy a weapon on B only; A's money is unchanged; die and respawn, the weapon is still yours.
8. Turn Wi-Fi off on B for 10 s, turn it back on: B should reconnect. 9. Settings > Performance panel ON: read ping, in/out KB/s, fps.

## What was tested here (honest)
* Server: 36 end-to-end checks over real WebSockets (rooms, codes, limits, ownership races, hand-off, damage rules/clamps/rate limits, death/respawn/protection, PvP match, co-op rewards, economy isolation, late join replay, reconnect, host migration).
* Network client: 15 checks (latency/clock sync, auto-reconnect, errors, interpolation maths).
* Game <-> server integration: 15 checks run with the REAL game code against the REAL server (3D and physics stubbed): room creation, world announcement, NPC/pose streaming, remote avatar creation, remote spawn, remote wounds as real geometry, damage/death/respawn, reconnect + world rebuild, leave.
* NOT tested: real Quest hardware, real physics replication quality, real latency/bandwidth, Quest browser WebSocket behaviour, concurrent cuts on one body part by two players, host migration of live NPC simulation, long sessions.
## Known limits
* Two players cutting the SAME part at the same instant can diverge slightly (events apply in arrival order, no rollback). * Clients report hits; the server clamps/rate-limits/reach-checks but cannot see physics, so a modified client could cheat within those limits. Mission progress for hits/severs is client-reported (rate-limited); kills are attributed by the host's NPC deaths.
* Reviving teammates in co-op is not implemented (respawn timer instead). * Avatars: 4 outfits x 3 skins, procedural walk/crouch from headset height (no full-body tracking). * Accounts have no login: they are tied to a random id in the browser.
