# Discovery: Browser-in-Minecraft + 1-to-1 Balatro port

## 1. Executive summary

The proposed idea is **partially unique with strong reusable foundations**. A real
Balatro *port* (running Balatro's actual Lua code) inside Minecraft does not exist
yet. The two ingredients the idea needs already exist separately and are reusable:

- A way to run the real Balatro game in a browser: the **love.js** LÖVE-to-web
  engine (used by web-balatro, Telatro, and Balatro-Web-Port). Balatro is a LÖVE
  game, so its shipped `Balatro.exe` contains the real game logic as Lua.
- A way to embed a real Chromium browser inside Minecraft on the target stack
  (Java Edition, **Fabric, Minecraft 26.2**): the **Rinku** library and the
  **Browsermod** reference app.

No single mod combines these to run the real Balatro inside Minecraft. The only
existing "Balatro in Minecraft" is a from-scratch reimplementation minigame
(Modrinth 1+mod), which does not satisfy the 1-to-1 / use-real-game requirement.

## 2. Existing solutions survey

### Browser-in-Minecraft (Chromium embed)

- **Rinku** (library). "Library mod for rendering a fully controllable
  Chromium-based browser in Minecraft." Supports Minecraft **26.2** on
  Fabric, NeoForge, and Forge. Modern CEF/Chromium, uncapped FPS (60 default,
  supports 120+), in-game settings UI, performance/compatibility improvements.
  LGPL-2.1. Last release v3.0.4 for MC 26.2 on Fabric, 27 Aug 2026.
  https://www.curseforge.com/minecraft/mc-mods/rinku
  https://modrinth.com/mod/rinku
  Rinku is the maintained successor to MCEF (Keksuccino's fork). This is the
  primary foundation candidate.

- **Browsermod** (end-user browser app). "Sophisticated, Chromium-based web
  browser integration for Minecraft." Renders as a Minecraft screen. Features:
  multi-tab, Picture-in-Picture, fullscreen browser, chat-link interceptor,
  persistent config, server admin commands. Uses Rinku/MCEF under the hood.
  Available for Minecraft **26.2 on Fabric** (browsermod-0.4-26.2.jar, 1 Jul 2026).
  https://www.curseforge.com/minecraft/mc-mods/browsermod
  Excellent reference for the exact integration pattern wanted (browser on an
  in-game screen). Note: 0.3 could stream the browser onto in-world blocks
  singleplayer; 0.4 dropped that and focuses on the browser screen.

- **Client Web Displays (CWD)**. "Client-side in-world Chromium displays."
  Fabric, MIT. In-world 3D displays with spatial audio; client-side (works on any
  server). Latest build is for Minecraft **26.1.2** (not 26.2 yet), beta, ~1.7k
  downloads.
  https://www.curseforge.com/minecraft/mc-mods/cwd
  Useful for the "display on a block/screen surface" vision, but version gap to
  26.2 is currently a blocker.

- **MCEF (CinemaMod/mcef)**. The original Minecraft Chromium Embedded Framework.
  Fabric and NeoForge. Last Fabric build targets Minecraft 1.21.4; effectively
  superseded by Rinku. LGPL-2.1.
  https://github.com/CinemaMod/mcef
  https://www.curseforge.com/minecraft/mc-mods/mcef
  Reference/legacy only.

- **Rinku / MCEF forks** (heibaiya-dev/mcef-1.21.4, meowdding/mcef, forks).
  Unmaintained or version-pinned forks. Not recommended as a dependency target.

### Balatro 1-to-1 web ports (LÖVE / love.js)

- **web-balatro** (W0W53R). "Real Balatro running on a LÖVE js runtime." Uses
  love.js. User supplies Balatro.exe; the build extracts the game and runs it in
  the browser via the LÖVE wasm engine. Save import/export; mod support via
  Lovely dump. 35 stars, MIT.
  https://github.com/W0W53R/web-balatro
  Confirms the mechanism: extract Balatro's .love from Balatro.exe, run on
  love.js.

- **Balatro-Web-Port** (ytrewq000). Full game port using love.js, with save
  import/export and multiple mods. Open-source build script (build.ps1 extracts
  Balatro.exe). Includes a disclaimer: Balatro is LocalThunk's trademark, no
  endorsement, no redistribution.
  https://github.com/ytrewq000/Balatro-Web-Port
  Confirms the accepted legal pattern: user supplies Balatro.exe; no Balatro
  code/assets are distributed by the port.

- **Telatro** (tomcat). "A full web port of the hit gambling game Balatro...
  ad-free, tracker-free, embeddable." Closed-source; creator is tomcat
  (self-taught full-stack dev, html/css/js/python/C++/rust).
  https://telatro.tomcat.sh
  Mechanism is not documented (source closed). Treated as equivalent to the
  open love.js ports: extract the .love from Balatrol.exe, run on love.js. So a
  "1-to-1" result comes for free because it runs the real engine, not a clone.

- **love.js (2dengine / Davidobot)**. The standard LÖVE-to-web runtime. Runs
  LÖVE 11.5 apps as WebAssembly (Emscripten build of the C++ LÖVE engine).
  Game code stays plain Lua; no build step for the game itself.
  https://github.com/2dengine/love.js
  Limitations: slower than native (no LuaJIT), WebGL shaders differ, audio
  requires a user gesture, ~150 MB for the bundled LÖVE runtime, in-browser
  package cache via IndexedDB.

- **LÖVE engine** (love2d/love). The target engine itself: C++ (SDL2 + OpenGL +
  Lua/LuaJIT), embeds the game as a .love zip containing Lua + assets.
  Dependencies: SDL3, OpenGL/Vulkan/Metal, OpenAL, Lua/LuaJIT/LLVM-lua, FreeType,
  harfbuzz, ModPlug, Vorbisfile, Theora.
  https://github.com/love2d/love
  Confirms why a .love can be extracted from Balatro.exe and re-run.

### Existing "Balatro in Minecraft"

- **BALATRO** (1+mod, Modrinth). "My Take on Balatro in Minecraft! ... Jokers ...
  Shop after each Round." A from-scratch Minecraft minigame inspired by Balatro.
  Fabric + Forge + NeoForge + Quilt, client-side, Minecraft 1.21.5, licensed ARR.
  https://modrinth.com/datapack/balatro/version/1+mod
  This is a reimplementation minigame, not the real Balatro code. Does not meet
  the 1-to-1 / use-real-game requirement.

- **Minecraft full deck** (Nexus). A card skin/reskin pack (SteamModded/Lovely)
  that retextures Balatro's cards with Minecraft imagery. Not Minecraft-side at
  all.
  https://www.nexusmods.com/balatro/mods/580

### Lua-in-JVM (path #2 option)

- **LuaJ** is the main Lua 5.2/5.3 VM for the JVM, but it does not implement LÖVE.
- **No Java-native LÖVE runtime exists.** love.js is the only engine port and it
  targets the web (WebAssembly/WebGL/DOM), not the JVM. A "LuaJ + re-implemented
  LÖVE API in Java" path would have to reimplement LÖVE's graphics, audio,
  shader, input, and filesystem API surface from scratch, with no reusable engine
  core.

## 3. Uniqueness verification

Claim: running the real Balatro inside Minecraft does not already exist.

Evidence:
- The browser-in-Minecraft ecosystem (Rinku, Browsermod, CWD, MCEF) embeds
  Chromium but ships no Balatro integration. Their READMEs list generic browsing,
  HUDs, and in-world displays, not Balatro.
- The only mod literally named for Balatro in Minecraft is the 1+mod datapack,
  which is a self-contained Minecraft minigame (no Balatro code), on 1.21.5, ARR.
- The love.js Balatro ports (web-balatro, ytrewq000, Telatro) are web-only; none
  target Minecraft.

Confidence: high. The idea is unique in its specific form (real Balatro, embedded
in a Java Minecraft mod), but it composes two existing, reusable subsystems.

## 4. Gap analysis

1. No mod wires "user supplies Balatro.exe" together with "embed in Minecraft
   screen." The missing integration is end-to-end: extract/build a love.js package
   from the user's Balatro.exe and serve it to the in-game Chromium on 26.2
   Fabric, with input mapped to Minecraft controls.

2. love.js needs the user's Balatro.exe uploaded to the web port to build the
   .love. Inside an in-game browser this handoff is a UX gap: the file must reach
   the love.js loader, either via a local file URL, a bundled helper, or a
   generated local page. No existing mod handles this.

3. Telatro is closed-source, so its exact build/packaging pipeline is inferred
   from the open love.js ports (web-balatro, ytrewq000). Confidence is high but
   not verified against Telatro itself.

4. love.js does not match native RNG, and WebGL shader/audio behavior differs;
   seed parity with the user's Balatro version is not guaranteed. Known limitation
   accepted by the existing ports.

5. Browsermod's in-world block streaming was dropped in 0.4; CWD still does
   in-world displays but only on 26.1.2. If the goal is "Balatro playing on a
   Minecraft computer block," this surface question is still open.

## 5. Foundation candidates (ranked)

1. Rinku (library) + Browsermod (reference app). Best foundation for the
   embed path. Fabric 26.2 confirmed, modern Chromium, actively maintained,
   LGPL-2.1. Browsermod shows the exact "browser as a Minecraft screen" pattern
   wanted.
   https://www.curseforge.com/minecraft/mc-mods/rinku

2. love.js (2dengine). Required runtime to host web-balatro/Telatro in the
   embedded browser. Not a Minecraft mod; it is a dependency of the web port.
   https://github.com/2dengine/love.js

3. Client Web Displays. Useful if the vision is a 3D in-world screen rather than a
   flat GUI screen. Version gap: 26.1.2 only, beta. Candidate to watch/rebuild
   for 26.2.
   https://www.curseforge.com/minecraft/mc-mods/cwd

4. web-balatro / ytrewq000 (open love.js builds). Reusable as the embedded
   Balatro target, including their Balatro.exe-extraction build flow.
   https://github.com/W0W53R/web-balatro
   https://github.com/ytrewq000/Balatro-Web-Port

5. MCEF (CinemaMod). Legacy; Fabric tops out at 1.21.4. Reference only; Rinku
   supersedes it.
   https://github.com/CinemaMod/mcef

6. LuaJ + from-scratch LÖVE API (path #2). Weak foundation; no reusable engine
   core. Only viable if a non-browser integrated path is required.

## 6. Unknown areas

- Whether Telatro's packaging/build differs from the open love.js ports in any
  way that matters to reproduction.
- Whether the user's installed Balatro version matches the love.js LÖVE 11.5
  target (RNG and mod compatibility vary by version).
- Performance of running love.js (WebGL) inside Chromium inside Minecraft
  (OpenGL): double rendering path and memory overhead are unverified.
- Whether Browsermod exposes a programmatic API suitable for an item- or
  computer-triggered screen, versus only keybindings/commands.
