# DnD VTT

A virtual tabletop (VTT) for running D&D campaigns. Built for **mixed play**: some sessions in person (map on a TV/monitor on the table, players on their phones), some online (remote players join from a browser link), and hybrid nights with both at once.

Primary goal: a tool for my own group, and a learning/portfolio project. Not competing head-on with Roll20/Foundry; differentiators worth leaning into are good in-person/table support, phone-friendly player views, and simplicity.

## Decisions made

- **Web technologies, not native.** Remote players must be able to join from a browser link with no install. A desktop wrapper (Tauri) may come later for the DM.
- **TypeScript everywhere.** No PHP/Laravel (not my preference). Client and server share types and game rules.
- **Vue 3, not React.**
- **PixiJS is used directly** as a plain TypeScript map engine (`MapEngine.ts`), mounted inside a Vue component. No Vue-Pixi binding, so the engine stays framework-independent and fast.
- **Event-driven game state from day one.** Every action (UI button or token drag) is an event like `token:move`, `dice:roll` or `fog:reveal`, processed by one central game-state module. Going online later means sending the same events over a WebSocket instead of rewriting the app.
- **Multiplayer model (later): server-authoritative.** The server validates events, rolls dice and filters hidden information (fog of war, hidden tokens, DM notes, secret rolls). Hidden data must never be sent to player clients.
- **Hosting should stay free or cheap.** Free tiers, or the DM's PC with a tunnel (Tailscale / Cloudflare Tunnel), before paying for a VPS.

## Stack

**Frontend (`apps/web`)**
- Vue 3 + TypeScript + Vite, `<script setup>` single-file components
- PixiJS (map, grid, tokens, fog) as a standalone engine
- Pinia for game state (shared by the UI panels and the map engine)
- Vue Router: `/dm`, `/table`, `/play/:roomCode`
- Tailwind CSS, plus PrimeVue or shadcn-vue for UI components
- VueUse utilities
- `@dice-roller/rpg-dice-roller` for dice notation (`2d6+3`, `4d6kh3`, advantage)
- `vite-plugin-pwa` so phones can install the player view

**Backend (`apps/server`)**, from milestone 3
- Node.js + Fastify (HTTP API: campaigns, uploads, auth)
- Socket.IO (realtime game rooms)
- Postgres + Drizzle ORM
- Better Auth (later)

**Shared (`packages/shared`)**
- Event types, Zod schemas for validating events, dice logic, game rules

## Project structure (pnpm monorepo)

```
apps/
  web/        # Vue + Vite + PixiJS (DM, table and player views)
  server/     # Fastify + Socket.IO game server
packages/
  shared/     # Types, events, dice, rules
pnpm-workspace.yaml
```

## Views

- **DM view:** full control, sees everything (hidden tokens, full map, notes)
- **Table/TV view:** the shared map only, no UI clutter, respects fog of war
- **Player view:** phone-friendly; character sheet, dice, moving their own token

## Roadmap

1. **Local prototype (frontend only):** upload a map image, adjustable square grid, add and drag tokens with snap-to-grid, pan/zoom, dice roller with roll log. Everything goes through events in the Pinia store.
2. **DM and table views:** fog of war controlled by the DM; two tabs synced locally (`BroadcastChannel`).
3. **Game server:** move state to Socket.IO rooms; players join from their phones with a room code.
4. **Persistence and accounts:** Postgres, saved campaigns, logins.
5. **Extras:** initiative tracker, HP and conditions on tokens, character sheets, monster stat blocks (SRD 5.1), handouts, measuring and AoE templates, music, Tauri desktop wrapper.

Later ideas: dynamic lighting and line of sight, campaign notes/wiki, encounter builder, session recaps, AI DM helpers.
