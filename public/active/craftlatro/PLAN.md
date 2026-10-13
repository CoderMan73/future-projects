# Plan: Craftlatro - Balatro inside a Minecraft Java Edition mod

> Derived from CONCEPT.md and grounded in RESEARCH.md (see `research-template.md` structure). Implementation starts only after explicit approval; relocation from `concept/` to `planned/` happens only on approval.

## Objective

A Fabric mod for Minecraft 1.26.2 that lets a player run the real Balatro game inside Minecraft. The player supplies their own Balatro.exe; the mod extracts its `.love`, serves a love.js package to an in-game Chromium browser, and surfaces it on a flat in-game screen with Minecraft controls mapped to the game. No Balatro code or assets are bundled or distributed.

## Goal / Motive / Purpose

Let players experience the genuine Balatro (1-to-1 game logic, all decks, stakes, jokers, blinds, profiles) from inside Minecraft Java Edition, by composing an existing browser-embed library (Rinku) with an existing real-game web runtime (love.js) rather than hand-rolling the rules. The purpose constrains scope: do not reimplement Balatro's logic, do not bundle Balatro assets, and ship only the integration glue (extraction/serving, in-game screen, input mapping).

## Chosen path

Embed a Chromium browser via Rinku (Fabric, 1.26.2) and render the love.js web port of Balatro on a flat in-game GUI screen. Rationale: this is the only path that runs Balatro's real Lua logic with 1-to-1 behavior at all (RESEARCH.md Section 2, candidates 1-3 and Section 4 Gaps 1-3). The LuaJ + from-scratch LÃ–VE API path (RESEARCH.md Section 5, candidate 7) is rejected because no Java-native LÃ–VE engine exists and it would reimplement the entire LÃ–VE API surface.

## Target stack and tooling

- Loader: Fabric, Minecraft 1.26.2 (matches Rinku v3.0.4 and WebGUI support; RESEARCH.md Section 2, candidate 1).
- Browser: Rinku v3.0.4 (Chromium 151.0.7922.34). Dependency `de.keksuccino:rinku-fabric:3.0.4-1.26.2` from Maven `https://keksuccino.github.io/maven/`. On Fabric the runtime is bundled into the consumer jar (RESEARCH.md Section 2, candidate 1).
- Runtime: love.js (2dengine player, MIT) loading the user's extracted `.love`. The 2dengine player runs a `.love` directly via `player.js?g=...` and is the maintained LÃ–VE 11.5 web player (RESEARCH.md Section 2, candidate 2).
- Balatro extraction: reuse the open love.js ports' Balatro.exe -> `.love` extraction flow from web-balatro and Balatro-Web-Port (RESEARCH.md Section 2, candidate 3). No Balatro code/assets shipped.
- Local serving: a minimal in-mod HTTP server (Java `com.sun.net.httpserver` or equivalent) to serve the generated love.js package to the embedded Chromium with COOP/COEP headers. Required because the 2dengine player will not run a `file://` page for the standard build (RESEARCH.md Section 4, Gap 5). Compatibility mode (`-c`) is a fallback if a server is not feasible.
- Build: Gradle (Fabric Loom), Java 21 (required for 1.26 by the Fabric/Rinku stack; RESEARCH.md Section 2, WebGUI reference supports Java 21+).
- Legal guardrails: only accept a user-supplied Balatro.exe; never bundle or extract Balatro code/assets into the repo; do not distribute the love.js LÃ–VE runtime itself with the mod jar if it crosses redistribution lines (serve it, do not ship it) (RESEARCH.md Section 2, candidate 3).

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
- Balatro version / LÃ–VE 11.5 parity: if the user's Balatro.exe targets a LÃ–VE version outside love.js 11.5 support, RNG/mod fidelity may break (RESEARCH.md Section 6, unknown area 2). Mitigation: detect and warn at extraction.
- Performance: love.js (WebGL) inside Chromium inside Minecraft (OpenGL) is an unverified double-render path with a ~150 MB Chromium runtime (RESEARCH.md Section 6, unknown area 6). Mitigation: cap browser FPS to 60 and add a render-distance/graphics toggle.
- Legal: bundling the love.js LÃ–VE runtime with the mod jar could cross redistribution lines. Mitigation: serve the runtime from a path the user opts into; keep the mod jar free of Balatro assets.

## Handoff boundary

This plan is complete at RESEARCH.md + PLAN.md and waits for explicit user approval before relocating `concept/craftlatro/` to `planned/craftlatro/`. Implementation (scaffolding, builds, code, PRs) begins only on a separate explicit request. The handoff boundary between planning and implementation is the M1 checklist in `docs/rinku-api-notes.md` plus the Rinku dependency resolution in a dev run.

---

## Status as of 2026-10-12: M1-M5 complete, concept proven

Verified by a full `./gradlew runClient` session against a user-supplied Balatro.exe. All milestones
are complete; no architectural work remains open.

| Milestone | Status | Evidence |
|---|---|---|
| M1 Scaffolding + Rinku | Complete | Rinku 3.0.4 loads, flat-GUI browser API confirmed usable |
| M2 Extraction + serving | Complete | `.love` extraction, local HTTP server with COOP/COEP, IndexedDB-safe serving |
| M3 In-game screen | Complete | `BrowserScreen` blits the Rinku texture, MC input mapped through |
| M4 Controls / UX / fidelity | Complete | keybind `B`, commands, chat link, persisted fidelity warning |
| M5 Packaging + guardrails | Complete | jar is 70 KB, classes + Lua shims only; no assets, no runtime |

### What M2/M3/M4 actually required
Four independent compounding bugs had to be fixed; the first was misread as architectural failure
and triggered a pivot decision that turned out to be wrong. Full record in the repo at
`ai-docs/notes/2026-10-12-lovejs-pipeline-proven.md`, and the pivot ADR is marked resolved.

1. **ZIP data descriptors** — Java `ZipOutputStream` writes data descriptors that PhysFS (compiled C)
   cannot parse; repacked `.love` as STORED with explicit sizes.
2. **love.js IndexedDB cache** — `Player.cache = true` keys only on `?g=`, so `Cache-Control` was
   useless; versioned the `?g` parameter by `game.love` mtime.
3. **Offscreen paint rate never set** — surface ran at Chromium's default, well below Minecraft's own
   framerate limit; now set explicitly and auto-capped to Minecraft's measured fps.
4. **Screen size never propagated** — `Screen`'s own `width`/`height` only refresh on rebuild; now read
   from `Window` and re-checked every frame.

### Remaining work (parked, not blocking)
- **Optimization only**: render scale, frame rate, parked-surface policy. Largest lever is
  `/craftlatro render` (chromium cost scales with pixel count).
- **UX**: no user-facing build/run guide; default-`render scale` value should be re-confirmed against
  the persisted config.
- **Testing**: none configured yet. `./gradlew check` passes but has no test sources; a client-side
  harness is the main piece of missing engineering.
- **Deferred by design**: in-world 3D/computer-block surface (CONCEPT.md resolved decisions).

### Handoff notes for the next session
Start by reading `AGENTS.md` and `ai-docs/notes/2026-10-12-lovejs-pipeline-proven.md` — the latter
contains the runtime command table and the architecture rationale for `BalatroBrowser` as a
screen-agnostic singleton. Do not re-litigate the love.js approach.

---

## Status as of 2026-10-13 (evening): 0.9.0, UX pass, and the resolution thread

Version is now `0.9.0+26.2` (gradle.properties). Full record of the day in
`ai-docs/notes/2026-10-13-session-handoff.md`; the resolution work and its crash are in
`ai-docs/notes/2026-10-13-lovejs-canvas-sizes-at-page-load.md`.

### Shipped this session
- **Window**: framed window with title bar, drag, resize, aspect lock, blur, full-screen toggle;
  full screen now fills the viewport edge to edge (`a9459d0`).
- **Settings**: Cloth Config screen with Browser / Craftlatro categories; keybind category is
  Craftlatro's own; `Pause on Browser open` and `Open key is a toggle` live in Browser > General.
- **Save data**: export/import/list exist (previous session); **`/craftlatro save clear [confirm]`**
  erases the game's save data and restarts the page (new).
- **Resolution**: render scale now runs 25-200%, capped so the surface never exceeds the window's own
  full-screen size; the game still only picks a value up at page load, so the notice strip below
  exists.
- **Config location**: settings, save exports and the game cache moved out of `~/.craftlatro` into the
  Minecraft instance (`config/craftlatro.properties`, `config/craftlatro/saves/`,
  `<instance>/craftlatro/game`), migrated automatically on first read (`CraftlatroPaths`).
- **Notice strip**: when the window settles somewhere >10% off the size the page loaded at, a strip
  along the top offers Reload page / Ignore rather than reloading by itself.

### The one thing that must not be re-attempted blindly
A browser surface **larger than the window** crashed the client natively, twice, deterministically:
`EXCEPTION_ACCESS_VIOLATION` in `jemalloc.dll` under Rinku's `AsyncResourceLeaseManager.drain`,
~1s into page load (`run/hs_err_pid15784.log`, `hs_err_pid4532.log`). Every surface at or below the
window's own size has run safely across all sessions. Rinku is a prebuilt dependency, so the bug is
not fixable here; it is worth reporting upstream. This is why render scale above 100% is capped and
why the notice offers a reload instead of forcing one.

### Unverified — needs the player's hands, an agent cannot do these
- The notice strip end to end (appearance at >10% change, Reload page, Ignore, narrow-window layout).
- Render scale above 100% (does the canvas really re-create at the higher size after a reload).
- `/craftlatro save clear` (both the live-page and no-session paths) and the first-run path migration
  from `~/.craftlatro` into the instance.
- The first-run pipeline test (fresh instance, no cached Chromium) has still never been done.

### Parked work (see ROADMAP.md Backlog for the full list and the reasoning)
Browser volume slider (blocked on PCM capture and an audio output, not on design), review of whether
the command tree still earns its keep, save import/export testing, a code review, the library
extraction proven by porting a second game, visual polish, README and a
GitHub Pages landing page (gated on a remote repo, an Ask First boundary). Everything on that list is
the gate the owner drew for 1.0, with 0.9.0 as the current build.

### Releases and CI (2026-10-13, late)

Two releases are published, both from `main`, both with the jar produced by the Gradle wrapper:

| Release | Tag | Content |
|---|---|---|
| 0.9.0 | `v0.9.0` (main@`d9cd48c`) | the whole UX pass — window, settings, saves, resolution notice, instance-local paths |
| 0.9.1 | `v0.9.1` (main@`8e5df84`) | CI repaired (it had never been green) and release automation added; no gameplay changes |

CI now runs **only when a release is published** (`.github/workflows/release.yml`): it builds the
tagged source and attaches the jar, so a push or a merge costs no runner time and the local
`./gradlew clean build check` is the only gate between a commit and a tag. A failed release workflow
leaves the release published without a jar, never with a broken one. The conventions it was all
codified under live in `docs/commits-and-releases.md`
(Conventional Commits, one-line versioning in `gradle.properties`, the release runbook) and
`CHANGELOG.md` (Keep a Changelog). The standing boundary from AGENTS.md still applies: pushing tags
and cutting releases is Ask First, and this session's two releases were each authorized by an explicit
message.
