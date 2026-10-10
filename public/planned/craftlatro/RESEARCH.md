# Research: Craftlatro - Balatro inside a Minecraft Java Edition mod

> Discovery run per `dont-reinvent-the-wheel` snapshot at `references/dont-reinvent-the-wheel.md` (snapshotted 2026-10-09T17:43:30Z). Scope: browser-in-Minecraft Chromium embeds + real Balatro (love.js) ports. Source: CONCEPT.md and prior DISCOVERY.md in `concept/craftlatro/`.

## 1. Executive summary

The idea is **partially unique with strong reusable foundations**. A real Balatro port (running Balatro's actual Lua code) inside a Minecraft Java mod does not exist. The two required ingredients already exist separately and are composable:

- Host a real Chromium browser inside Minecraft on the target stack (Fabric, Minecraft 1.26.2): the Rinku library.
- Run the real Balatro game logic in a browser: the love.js LÖVE-to-web engine, used by web-balatro and Balatro-Web-Port to extract the user's Balatro.exe and run it.

Confidence: high that no single mod combines these. The only existing "Balatro in Minecraft" is a from-scratch reimplementation minigame (1+mod), which does not satisfy the 1-to-1 / use-real-game requirement.

## 2. Existing solutions survey

### Browser-in-Minecraft (Chromium embed)

- Rinku (library). "Library mod for rendering a Chromium-based browser in Minecraft." Fabric, NeoForge, Forge, Minecraft 1.26.2. Modern CEF/Chromium (pinned to 151.0.7922.34 per Keksuccino/jcef-mcef), uncapped FPS (60 default, 120+ when set), in-game settings UI, performance/compatibility improvements. LGPL-2.1. v3.0.4 for 1.26.2 Fabric released 27 Aug 2026; v3.0.5 for 1.26.3 released 19 Sep 2026. Published to Maven `https://keksuccino.github.io/maven/` as `de.keksuccino:rinku-fabric:<v>-<mc>`. On Fabric the runtime is bundled into consumers; NeoForge needs Rinku declared separately. Chromium runtime downloads on first launch (~150 MB; jcefmaven/jcefbuild CEF 151 linux-amd64 release is 151 MB). Maintained successor to MCEF. Confirmed library API: third-party mods (e.g., WebGUI) mixin-target Rinku classes and depend on it as a library. https://github.com/Keksuccino/Rinku ; https://www.curseforge.com/minecraft/mc-mods/rinku

- WebGUI (end-user embed app). "Embed a real Chromium browser via Rinku in the game client... display any web page as a transparent HUD overlay... custom main menu page." Supports Minecraft 1.20.1, 1.21.1, 1.21.11, 1.26.1.2, 1.26.2; Fabric + NeoForge; Rinku required (bundled on Fabric, separate on NeoForge). "WebGUI's mixins target its classes" confirms Rinku exposes a consumable Java API. ~150 MB Chromium download on first launch. Licensed ARR (all rights reserved). Closest structural reference to Craftlatro's flat-GUI-screen surface. https://webgui.space/guide/getting-started.html

- Browsermod (end-user browser app). CC0-1.0. "Sophisticated Chromium web browser integration... Multi-Tab, PiP, fullscreen, chat-link interceptor, persistence." Fabric + NeoForge, Minecraft 1.26.2 (browsermod-0.4-1.26.2.jar, Jul 2026). Depends on MCEF [Keksuccino's Fork] (Rinku lineage). Keybindings only: B opens the browser screen, P toggles PiP, F12 toggles fullscreen. No documented programmatic API for other mods to trigger its screen. "Stream Browser ingame on blocks... only version 0.3 higher versions not supported" (in-world streaming dropped in 0.4). https://github.com/Mcjunky33/BrowserMod ; https://www.curseforge.com/minecraft/mc-mods/browsermod

- Client Web Displays (CWD). Fabric, MIT. "Client-side in-world Chromium displays." Spatial audio, rich window config. Latest build 1.0-beta-1 for Minecraft 1.26.1.2 (Jun 3 2026) only; no 1.26.2 build. Version gap is a blocker for the 1.26.2 target. https://www.curseforge.com/minecraft/mc-mods/cwd

- MCEF (CinemaMod). Original Minecraft CEF. Fabric and NeoForge. Last Fabric build targets 1.21.4; superseded by Rinku. LGPL-2.1. Reference/legacy only. https://github.com/CinemaMod/mcef

- WebDisplays / WebDisplays Unofficial (Fabric). Legacy in-world web screens via MCEF. WebDisplays tops out at 1.20.1; the unofficial Fabric port sits at 1.21.4 beta. Not on 1.26.2. Reference only. https://github.com/CinemaMod/webdisplays ; https://www.curseforge.com/minecraft/mc-mods/webdisplays-unofficial-fabric

### Balatro 1-to-1 web ports (LÖVE / love.js)

- love.js (2dengine). MIT, 188 stars. The maintained LÖVE-to-web player: runs a `.love` file directly with no build step via `<script src='player.js?g=mygame.love'></script>`. Targets LÖVE 11.5. Requires a web server and COOP/COEP (`same-origin`/`require-corp`) headers for the standard (pthread) build; a compatibility build (`-c`) runs without pthreads and works on hosts that do not send COOP/COEP. IndexedDB caches packaged games; ~150 MB LÖVE runtime. Limitations: slower than native (no LuaJIT), WebGL shaders differ, audio requires a user gesture, no LuaSocket. https://github.com/2dengine/love.js

- love.js (Davidobot). MIT, 848 stars. The Emscripten builder that compiles LÖVE C++ to the WebAssembly runtime the 2dengine player loads. Forked from TannerRogalsky/love.js; updates Emscripten + LÖVE to 11.5. This is the build toolchain, not the in-game target. https://github.com/Davidobot/love.js

- LÖVE engine (love2d/love). The target engine: C++ (SDL2/3 + OpenGL + Lua/LuaJIT + OpenAL + FreeType etc.), embeds a game as a `.love` zip (Lua + assets). Balatro.exe is an LÖVE game packaged as an .exe, so the `.love` can be extracted and re-run on love.js. Confirms why the ports work. https://github.com/love2d/love

- web-balatro (W0W53R). MIT, 35 stars. "Real Balatro running on a LÖVE js runtime." User supplies Balatro.exe; the build extracts the `.love` and runs it in the browser via love.js. Save import/export; mod support via Lovely dump. "Make Portable" produces a 150 MiB zip ("everything needed to run Balatro in 3 files"). Confirms the extract-.love-from-Balatro.exe flow. https://github.com/W0W53R/web-balatro

- Balatro-Web-Port (ytrewq000). Build script (build.ps1) extracts Balatro.exe, generates a love.js web package, includes save import/export and mods. Disclaimer: Balatro is LocalThunk's trademark, no endorsement, no redistribution by the port. Confirms the accepted legal pattern (user supplies Balatro.exe; no Balatro code/assets distributed). https://github.com/ytrewq000/Balatro-Web-Port

- Telastro (tomcat). Closed-source. "A full web port of Balatro... ad-free, tracker-free, embeddable." Source not published; treated as equivalent to the open love.js ports (extract `.love` from Balatro.exe, run on love.js). Mechanism not independently verifiable. https://telatro.tomcat.sh

- balatro-browser (luluxe008). MIT fork of web-balatro tuned for mobile. Same extract-.love + love.js mechanism. https://github.com/luluxe008/balatro-browser

### Existing "Balatro in Minecraft"

- BALATRO (1+mod, Modrinth). A from-scratch Minecraft minigame inspired by Balatro (jokers, shop, rounds). Fabric + Forge + NeoForge + Quilt, client-side, Minecraft 1.21.5, licensed All Rights Reserved. This is a reimplementation, not the real Balatro code. Does not meet the 1-to-1 / use-real-game requirement. https://modrinth.com/datapack/balatro/version/1+mod

### Lua-in-JVM (path #2)

- LuaJ is the main Lua 5.2/5.3 VM for the JVM but does not implement LÖVE. No Java-native LÖVE runtime exists: love.js is the only engine port and it targets the web (WebAssembly/WebGL/DOM), not the JVM. A "LuaJ + re-implemented LÖVE API in Java" path would reimplement LÖVE's graphics, audio, shader, input, and filesystem API surface from scratch with no reusable engine core. Not recommended.

## 3. Uniqueness verification

Claim: running the real Balatro inside a Minecraft Java mod does not already exist.

Evidence:
- The browser-in-Minecraft ecosystem (Rinku, WebGUI, Browsermod, CWD, MCEF) embeds Chromium but ships no Balatro integration; their READMEs describe generic browsing, HUDs, video, and in-world displays, not Balatro.
- The only mod literally named for Balatro in Minecraft is the 1+mod datapack, a self-contained Minecraft minigame (no Balatro code), on 1.21.5, ARR.
- The love.js Balatro ports (web-balatro, Balatro-Web-Port, Telastro, balatro-browser) are web-only; none target Minecraft.
- WebGUI (the closest embed reference) supports generic URLs/HUD overlays only; no Balatro-specific integration.

Confidence: high. The specific form (real Balatro, embedded in a Java Minecraft mod) is unique, but it composes two existing, reusable subsystems (Rinku for the browser; love.js + an extractor for the game logic).

## 4. Gap analysis

1. No mod wires "user supplies Balatro.exe" together with "embed in Minecraft screen." The missing integration is end-to-end: extract the `.love` from the user's Balatro.exe, serve a love.js package to the in-game Chromium on 1.26.2 Fabric, and map Minecraft controls to the browser.

2. love.js needs the user's Balatro.exe to build/serve the `.love`. Inside an in-game browser this is a UX gap: the file must reach the love.js loader via a local file URL, a bundled helper, or a generated local page. No existing mod handles Balatro.exe -> in-game love.js handoff.

3. love.js does not match native RNG, and WebGL shader/audio behavior differs from native LÖVE; seed/frame parity with the user's Balatro version is not guaranteed. This limitation is accepted by the existing web ports and is inherent to the love.js approach.

4. In-world 3D screen is open. Browsermod's in-world block streaming was dropped in 0.4 (too buggy); CWD still does in-world displays but only on 1.26.1.2. If the goal expands to "Balatro playing on a Minecraft computer block," a 1.26.2-capable in-world display path is unresolved.

5. love.js player requires a web server (the 2dengine player will not run from a local `file://` page for the standard build; it needs COOP/COEP headers). An embedded Chromium inside Minecraft may relax the `file://` restriction, but the robust path is a local HTTP server bundled in the mod serving the love.js package with correct headers.

6. The user's installed Balatro version must match the love.js LÖVE 11.5 target (RNG and mod compatibility vary by version). Not yet verified against Balatro's shipping LÖVE/Lua version.

## 5. Foundation candidates (ranked)

1. Rinku (library) + WebGUI pattern (reference). Best foundation for the embed path. Fabric 1.26.2 confirmed, modern Chromium 151.x, actively maintained, LGPL-2.1. Maven coords `de.keksuccino:rinku-fabric:<v>-<mc>` at `https://keksuccino.github.io/maven/`. WebGUI proves the library API is consumable (other mods mixin-target Rinku classes) and confirms the ~150 MB Chromium download. https://github.com/Keksuccino/Rinku

2. 2dengine/love.js (player, MIT) + Davidobot/love.js (Emscripten build toolchain, MIT). Required runtime to host web-balatro/Telastro in the embedded browser. The 2dengine player runs a `.love` directly via `?g=` and needs a local web server (see Gap 5). https://github.com/2dengine/love.js ; https://github.com/Davidobot/love.js

3. web-balatro / Balatro-Web-Port (open builds, MIT). Reusable as the embedded Balatro target, including their Balatro.exe -> `.love` extraction flow and the "Make Portable" packaging. https://github.com/W0W53R/web-balatro ; https://github.com/ytrewq000/Balatro-Web-Port

4. Client Web Displays. Useful only if the vision is a 3D in-world screen rather than a flat GUI screen. Version gap (1.26.1.2 only) is a current blocker. https://www.curseforge.com/minecraft/mc-mods/cwd

5. Browsermod (CC0). Reference for the "browser on an in-game screen" pattern and keybindings, but no programmatic API for item/computer-triggered screens. https://github.com/Mcjunky33/BrowserMod

6. MCEF (CinemaMod). Legacy; Fabric tops out at 1.21.4. Reference only; Rinku supersedes it. https://github.com/CinemaMod/mcef

7. LuaJ + from-scratch LÖVE API (path #2). Weak foundation; no reusable engine core. Only viable if a non-browser integrated path is required.

## 6. Unknown areas

- Whether Rinku's public Java API is sufficiently documented/stable to drive a flat GUI browser screen from Craftlatro (WebGUI proves the API is consumable via mixins, but the exact surface and version stability need verification during implementation). Resolved: confirm by reading Rinku source/API at kickoff.

- Whether the user's installed Balatro version matches the love.js LÖVE 11.5 target (RNG and mod compatibility vary by version). Resolution: detect Balatro.exe version at extraction time and warn if outside supported LÖVE 11.5 range.

- Whether the embedded Chromium inside Minecraft can load a love.js package from `file://` (avoiding a bundled local HTTP server), or whether a local server with COOP/COEP headers is required. Resolution: prototype load path during implementation; default to a bundled local server as the robust path.

- Whether Browsermod or another existing mod exposes a programmatic entrance to open a URL/screen on demand (item or keybinding) that Craftlatro could reuse instead of integrating Rinku directly. Resolution: inspect Browsermod/WebGUI entrypoints during implementation; fallback to direct Rinku integration.

- Whether Telastro's packaging/build differs from the open love.js ports in any way that affects reproduction. Confidence is high but not independently verified against Telastro's closed source.

- Performance of running love.js (WebGL) inside Chromium inside Minecraft (OpenGL): the double-render path and ~150 MB Chromium runtime memory overhead are unverified for real gameplay.
