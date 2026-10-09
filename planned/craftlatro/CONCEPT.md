# Craftlatro

## Objective
A Java Edition Minecraft mod (Forge or Fabric) that lets a player run the real
Balatro game from inside Minecraft. Open an interactable computer or item, and
the actual Balatro game becomes playable on an in-game screen.

## Technical Goals
- Run Balatro's genuine game logic with 1-to-1 behavior, matching the
  web-balatro and Telatro web ports. Not a hand-rolled clone of the rules.
- The player supplies their own Balatro.exe; the mod extracts and runs it. No
  copyrighted assets or source are bundled or distributed.
- Surface the game on an in-game Minecraft screen/UI element.
- Avoid large-scale reimplementation of Balatro's jokers, decks, stakes, and
  blind logic.

## Recommended Approach
Discovery (see DISCOVERY.md) changes the read on path 1: an in-game Chromium
browser on the target stack (Fabric, Minecraft 26.2) already exists and is
supported.

1. Embed a Chromium browser via Rinku (LGPL-2.1, Fabric + NeoForge + Forge, MC 26.2,
   actively maintained) and render the existing web-balatro/Telatro love.js build
   on an in-game screen. Least reimplementation of game logic. The OpenGL/LWJGL
   integration is already provided by these libraries. Caveat: a Chromium runtime
   is a heavy download (~150 MB) and love.js is slower than native and does not
   match native RNG/ shaders.
2. Run Balatro's Lua directly in Java via a Lua runtime (LuaJ) with a
   re-implemented LÖVE API surface. No Java-native LÖVE engine exists (love.js is
   the only engine port and it targets the web, not the JVM), so this path
   reimplements LÖVE's graphics/audio/input/filesystem API from scratch.

Recommendation: start with path 1 (Rinku + web-balatro/Telatro love.js build).
Browsermod is a reference app showing the exact "browser as an in-game screen"
pattern. Switch to path 2 only if a non-browser integrated path is required.

## Restrictions and Constraints
- Legal: must comply with Balatro's proprietary license. Only user-supplied
  Balatro.exe is accepted as input. No Balatro code or assets in this repo.
- Platform: Java Edition only. Not Bedrock (C++ add-ons).
- Target fidelity: 1-to-1 with original Balatro (all decks, stakes, jokers,
  blinds, profiles).
- No execution or builds until this concept is validated.

## User Preferences (confirmed)
- Java Edition mod.
- User supplies own Balatro.exe.
- Favors whichever path is easier; leans toward embedding the web port.
- Wants true 1-to-1 behavior, like the web ports achieve.

## Resolved Decisions
- Loader: Fabric, Minecraft 26.2 (Rinku and Browsermod both provide Fabric 26.2).
- Execution path: embed the web port via Rinku + love.js (path 1).
- In-game surface: a flat GUI screen (first implementation); in-world 3D display
  deferred.

## File Handoff UX (decided)
On entering a world, if the player has not supplied a Balatro.exe, the mod sends a
client-only chat message:
  [CraftLatro] Balatro won't work without the game. Click here to select balatro.exe.
Clicking it (and a keybind, and a command) opens a file picker for the player to
locate their Balatro.exe. The mod records the path and uses it to build/serve the
love.js package to the embedded browser when the game screen is opened. No Balatro
assets are bundled by the mod.

## Status
Validated (confirmed by requester). No development yet — awaiting an explicit
execute request before any builds or code.
