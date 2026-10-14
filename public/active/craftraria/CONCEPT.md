# Concept: Craftraria

## Objective
A Java Edition Minecraft mod that lets a player run the real Terraria from inside
Minecraft on an in-game GUI screen. It embeds a browser (Rinku) that hosts a wasm
port of the game, with the player supplying their own purchased copy. It is a
sibling app to Craftlatro with the same handoff pattern.

## Goal / Motive / Purpose
A second, independent app in the browser-embedded family, so Craftlatro reads as
an app rather than as the library itself. Terraria qualifies like Celeste: a
commercial C#/FNA game with no JVM runtime path, whose full-fidelity portable
build is a community wasm port (a decompile-and-recompile of the game to wasm,
published by velzie). It is also, per the requester, fun.

## Technical Goals
- Run genuine Terraria at 1-to-1 behavior: the wasm build already contains the
  game code, and the player's mounted assets complete it.
- Player supplies their own Terraria game directory (functionally the same
  handoff as an exe, per the requester); nothing copyrighted is bundled or
  distributed by the mod.
- Embedded browser via Rinku (the MCEF successor), rendering the wasm build on a
  flat GUI screen.
- Input bridging from Minecraft key events into the game; audio through the
  embedded browser.
- Persistent saves: worlds and players persist across sessions, with the exact
  mechanism (emscripten filesystem persistence versus exporting back to the
  player's directory) resolved at planning time.

## Restrictions and Constraints
- Legal: Terraria is a commercial product (Re-Logic). The player must own a copy;
  no game code or assets ship with the mod.
- The wasm port is a community project built by decompiling and recompiling the
  game; its license, maintenance status, and supported Terraria versions are
  unverified.
- The published port runs vanilla content. tModLoader under wasm is unproven and
  is out of scope for v1 unless the requester says otherwise.
- Platform: Java Edition only, client-side.
- Non-goals: bundled game files or assets, in-world displays (flat GUI screen
  only), multiplayer (untested in the port).
- Heavier runtime than Celeste: full game content size plus FNA-in-wasm
  performance cost, on top of Minecraft and Chromium.

## User Preferences
- Structure this exactly like Craftlatro: browser-embedded runtime,
  player-supplied game files, flat GUI screen.
- Treat the directory handoff as functionally the same thing as an exe handoff;
  no additional handoff complexity.

## Planning-time Questions
- Which Terraria versions the port supports, and how the directory picker works
  through CEF.
- Local serving approach: embedded HTTP server versus file://, and whether the
  build needs cross-origin isolation headers (COOP/COEP) for SharedArrayBuffer.
- Save persistence: emscripten filesystem (IndexedDB) versus exporting worlds and
  players back into the player's game directory.
- Loader and MC version: follow Craftlatro's stack (Fabric, MC 26.2) or target a
  broader matrix given Rinku's current support.
- Performance and memory budget of Chromium plus wasm alongside Minecraft.
- Gamepad support priority (Terraria plays better with a controller).

## User Questions
- Steam or GOG install, and is vanilla-only acceptable for v1?
- Is multiplayer ever in scope, or is this strictly single-player?
- What is the in-game identity of the surface (arcade cabinet, handheld,
  computer screen)?
- Keyboard first or gamepad first?

## Status
Validated (confirmed by requester). No development yet; awaiting an explicit
execute request before any builds or code.
