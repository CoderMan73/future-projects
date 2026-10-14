# Concept: Craftleste

## Objective
A Java Edition Minecraft mod that lets a player run the real Celeste from inside
Minecraft on an in-game GUI screen. It embeds a browser (Rinku) that hosts a wasm
port of the game, with the player supplying their own purchased copy. It is a
sibling app to Craftlatro, not a new engine integration: same player-supplied
files, browser runs the genuine game.

## Goal / Motive / Purpose
Craftlatro on its own reads as a wrapper around a browser library. A second,
independent app in the same family establishes the browser-embedded-runtime
pattern as the product line, keeping the apps distinct from the library itself.
Celeste qualifies for the same reason Balatro did: it is a C#/FNA game with no
practical JVM path, so the only full-fidelity portable build is the wasm one
(Webleste, by Mercury Workshop). It is also, per the requester, fun.

## Technical Goals
- Run genuine Celeste at 1-to-1 behavior; the wasm build loads the player's own
  Celeste.exe at runtime rather than reimplementing the game.
- Player supplies their own Celeste.exe plus the Content folder; nothing
  copyrighted is bundled or distributed by the mod.
- Embedded browser via Rinku (the MCEF successor), rendering the wasm build on a
  flat GUI screen.
- Input bridging from Minecraft key events into the game; audio through the
  embedded browser.
- Persistent saves across sessions, with the exact mechanism (in-game saves
  versus save states) resolved at planning time.
- Everest mod support is technically available (the port runs a mod-patched
  exe); v1 scope to be confirmed.

## Restrictions and Constraints
- Legal: Celeste is a commercial product (Extremely OK Games / Matt Makes
  Games). The player must own a copy; no game code or assets ship with the mod.
- The wasm port (Webleste / celeste-wasm) is a community project, not an
  upstream release; its license, maintenance status, and supported Celeste
  versions are unverified.
- The port depends on proprietary FMOD emscripten builds; redistribution
  constraints on that toolchain are unknown.
- Platform: Java Edition only, client-side.
- Non-goals: bundled game files or assets, in-world displays (flat GUI screen
  only, matching Craftlatro's first surface), multiplayer.
- Memory: the Celeste wasm port uses roughly 600 MB of RAM, on top of Minecraft
  and Chromium.

## User Preferences
- Structure this exactly like Craftlatro: browser-embedded runtime,
  player-supplied game files, flat GUI screen.
- No extra complexity in the handoff: whether a directory or an exe is selected,
  treat it as the same thing.
- The mod is a product first (a second app for the browser-library line) and a
  fun one.

## Planning-time Questions
- Which Celeste editions and versions the Webleste stub loader supports, and how
  the player's assets are mounted through CEF's file dialog.
- Local serving approach: embedded HTTP server versus file://, and whether the
  build needs cross-origin isolation headers (COOP/COEP) for SharedArrayBuffer.
- Save persistence mechanism: emscripten IDBFS/IndexedDB versus exporting saves
  back into the player's game folder.
- Everest scope for v1: vanilla only, or documented mod-patched exe support.
- Loader and MC version: follow Craftlatro's stack (Fabric, MC 26.2) or target a
  broader matrix given Rinku's current support.
- Audio routing, frame pacing against Minecraft's render loop, and the Chromium
  plus wasm memory budget inside the MC process.
- Gamepad support: in scope for v1 or deferred (Celeste expects a controller).

## User Questions
- Which Celeste edition do you own and play (Steam, GOG, itch)?
- Is Everest/mod support part of the v1 promise, or vanilla-only until later?
- What is the in-game identity of the surface (arcade cabinet, handheld,
  computer screen)?
- Keyboard first or gamepad first?

## Status
Validated (confirmed by requester). No development yet; awaiting an explicit
execute request before any builds or code.
