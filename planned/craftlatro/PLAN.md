# Plan: Craftlatro - Balatro inside a Minecraft Java Edition mod

> Derived from CONCEPT.md and grounded in RESEARCH.md (see `research-template.md` structure). Implementation starts only after explicit approval; relocation from `concept/` to `planned/` happens only on approval.

## Objective

A Fabric mod for Minecraft 1.26.2 that lets a player run the real Balatro game inside Minecraft. The player supplies their own Balatro.exe; the mod extracts its `.love`, serves a love.js package to an in-game Chromium browser, and surfaces it on a flat in-game screen with Minecraft controls mapped to the game. No Balatro code or assets are bundled or distributed.

## Goal / Motive / Purpose

Let players experience the genuine Balatro (1-to-1 game logic, all decks, stakes, jokers, blinds, profiles) from inside Minecraft Java Edition, by composing an existing browser-embed library (Rinku) with an existing real-game web runtime (love.js) rather than hand-rolling the rules. The purpose constrains scope: do not reimplement Balatro's logic, do not bundle Balatro assets, and ship only the integration glue (extraction/serving, in-game screen, input mapping).

## Chosen path

Embed a Chromium browser via Rinku (Fabric, 1.26.2) and render the love.js web port of Balatro on a flat in-game GUI screen. Rationale: this is the only path that runs Balatro's real Lua logic with 1-to-1 behavior at all (RESEARCH.md Section 2, candidates 1-3 and Section 4 Gaps 1-3). The LuaJ + from-scratch LÖVE API path (RESEARCH.md Section 5, candidate 7) is rejected because no Java-native LÖVE engine exists and it would reimplement the entire LÖVE API surface.

## Target stack and tooling

- Loader: Fabric, Minecraft 1.26.2 (matches Rinku v3.0.4 and WebGUI support; RESEARCH.md Section 2, candidate 1).
- Browser: Rinku v3.0.4 (Chromium 151.0.7922.34). Dependency `de.keksuccino:rinku-fabric:3.0.4-1.26.2` from Maven `https://keksuccino.github.io/maven/`. On Fabric the runtime is bundled into the consumer jar (RESEARCH.md Section 2, candidate 1).
- Runtime: love.js (2dengine player, MIT) loading the user's extracted `.love`. The 2dengine player runs a `.love` directly via `player.js?g=...` and is the maintained LÖVE 11.5 web player (RESEARCH.md Section 2, candidate 2).
- Balatro extraction: reuse the open love.js ports' Balatro.exe -> `.love` extraction flow from web-balatro and Balatro-Web-Port (RESEARCH.md Section 2, candidate 3). No Balatro code/assets shipped.
- Local serving: a minimal in-mod HTTP server (Java `com.sun.net.httpserver` or equivalent) to serve the generated love.js package to the embedded Chromium with COOP/COEP headers. Required because the 2dengine player will not run a `file://` page for the standard build (RESEARCH.md Section 4, Gap 5). Compatibility mode (`-c`) is a fallback if a server is not feasible.
- Build: Gradle (Fabric Loom), Java 21 (required for 1.26 by the Fabric/Rinku stack; RESEARCH.md Section 2, WebGUI reference supports Java 21+).
- Legal guardrails: only accept a user-supplied Balatro.exe; never bundle or extract Balatro code/assets into the repo; do not distribute the love.js LÖVE runtime itself with the mod jar if it crosses redistribution lines (serve it, do not ship it) (RESEARCH.md Section 2, candidate 3).

## Milestones and steps

### M1: Scaffolding and dependency integration
- Scaffold the Fabric 1.26.2 mod project with Loom.
- Add Rinku as a compile/dependency from the Keksuccino Maven (`de.keksuccino:rinku-fabric:3.0.4-1.26.2`) and verify it resolves and loads in a dev run (RESEARCH.md Section 2, candidate 1).
- Inspect Rinku's public API surface to confirm a flat GUI browser screen can be driven programmatically (RESEARCH.md Section 6, unknown area 1). Record the API contract in `docs/rinku-api-notes.md`.

### M2: Balatro.exe -> love.js package pipeline
- Implement the client-side file picker (chat link, keybind, command) to record the user's Balatro.exe path; no assets stored in the mod (CONCEPT.md File Handoff UX).
- Implement extraction of the `.love` from the user's Balatro.exe using the web-balatro / Balatro-Web-Port extraction approach, packaging it with the 2dengine love.js player into a served bundle (RESEARCH.md Section 2, candidate 3; Gap 2).
- Implement the local HTTP server to serve the bundle with COOP/COEP headers; load it in the embedded Rinku browser at `http://localhost:<port>` (RESEARCH.md Section 4, Gap 5).

### M3: In-game screen surface
- Implement a flat Fabric GUI screen that hosts the Rinku browser view (RESEARCH.md Section 2, WebGUI reference, candidate 1).
- Map Minecraft keyboard/mouse input to the embedded browser (RESEARCH.md Section 4, Gap 1).
- Defer the in-world 3D/computer-block surface (CONCEPT.md resolved decisions) to a later milestone; M3 targets the flat GUI only (RESEARCH.md Section 4, Gap 4).

### M4: Controls, UX, and fidelity checks
- Wire the chat "Click here to select balatro.exe" flow and keybind/command entrypoints to open the game screen (CONCEPT.md File Handoff UX).
- Verify save persistence across sessions via the love.js IndexedDB path; surface the mod's save handling to the player.
- Document the known fidelity gap: love.js RNG/shader/audio differs from native Balatro and seed parity is not guaranteed (RESEARCH.md Section 4, Gap 3); surface this in-game on first run.

### M5: Packaging and legal guardrail
- Ensure no Balatro code/assets are included in the mod jar or repo; add a build-time check that fails if any Balatro binary is staged.
- Confirm the Chromium runtime is fetched at runtime from Rinku's cache (not bundled by Craftlatro) so the ~150 MB download is Rinku's responsibility (RESEARCH.md Section 2, candidate 1).

## Key risks and unknowns

- Rinku API stability/exposure for a flat GUI screen: WebGUI proves Rinku is consumable but the exact public API must be confirmed (RESEARCH.md Section 6, unknown area 1). Mitigation: inspect Rinku source at M1 kickoff; switch to Browsermod-entrypoint reuse if a programmatic surface is unavailable.
- Local serving path: if `file://` cannot host the 2dengine player, a bundled local HTTP server is required (RESEARCH.md Section 6, unknown area 3). Mitigation: default to the local server; fall back to love.js compatibility mode (`-c`) if headers are problematic.
- Balatro version / LÖVE 11.5 parity: if the user's Balatro.exe targets a LÖVE version outside love.js 11.5 support, RNG/mod fidelity may break (RESEARCH.md Section 6, unknown area 2). Mitigation: detect and warn at extraction.
- Performance: love.js (WebGL) inside Chromium inside Minecraft (OpenGL) is an unverified double-render path with a ~150 MB Chromium runtime (RESEARCH.md Section 6, unknown area 6). Mitigation: cap browser FPS to 60 and add a render-distance/graphics toggle.
- Legal: bundling the love.js LÖVE runtime with the mod jar could cross redistribution lines. Mitigation: serve the runtime from a path the user opts into; keep the mod jar free of Balatro assets.

## Handoff boundary

This plan is complete at RESEARCH.md + PLAN.md and waits for explicit user approval before relocating `concept/craftlatro/` to `planned/craftlatro/`. Implementation (scaffolding, builds, code, PRs) begins only on a separate explicit request. The handoff boundary between planning and implementation is the M1 checklist in `docs/rinku-api-notes.md` plus the Rinku dependency resolution in a dev run.
