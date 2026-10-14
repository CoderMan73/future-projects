# Plan: Craftleste (genuine Celeste inside a Minecraft Java Edition mod)

> Derived from CONCEPT.md and grounded in RESEARCH.md (same folder). Implementation starts only after explicit approval; relocation from `concept/` to `planned/` happens only on approval.

## Objective

A Fabric mod for Minecraft 26.2 that runs the genuine Celeste (2018) from inside Minecraft on an in-game GUI screen. The player supplies their own Celeste.exe (FNA build) plus the Content folder; the mod copies them into the Minecraft instance, fetches the Webleste wasm runtime at runtime, serves both over a local cross-origin-isolated HTTP server, and renders the game in an embedded Chromium browser (Rinku) on a flat, framed GUI window with Minecraft input mapped through. No Celeste code, assets, FMOD binaries, or Webleste build files ship in the jar or the repo.

## Goal / Motive / Purpose

Per CONCEPT.md: establish the browser-embedded-runtime pattern as a product line by shipping a second, independent app in the craftlatro family, keeping the apps distinct from the library itself, and doing it because Celeste is fun. The purpose constrains scope, and every milestone below ties back to it:

1. The genuine game runs, loaded from the player's own copy; the mod is integration glue, not an engine.
2. Nothing copyrighted is bundled or distributed; every runtime component is fetched at runtime.
3. The surface is the flat GUI screen, structured exactly like craftlatro, so the two projects stay reviewable against each other.
4. The result doubles as the proof that craftlatro's browser-as-a-screen pattern generalizes, which is the precondition for any later shared-library extraction ([CL] ROADMAP.md Backlog, Packaging/reuse, owner follow-up 2026-10-13).

## Chosen path

Embed Rinku (Chromium) on a flat Fabric GUI screen, host the Webleste stub-loader wasm port as the game runtime, serve the player's Celeste assets and the port from a local COOP/COEP HTTP server, and bridge disk files into the browser's OPFS with a mod-owned bootstrap page. Rationale: this is the only path that runs genuine Celeste at full fidelity, since no Java-native Celeste engine exists and the wasm port is the sole full-fidelity portable build (RESEARCH.md sections 2, 4, 5, candidate 1). It reuses craftlatro's proven answers for every hard part of the pipeline (isolation headers, cache busting, paint rate, parking, save store), so the plan's real work is the Celeste-specific intake and bootstrap, not new architecture. Rejected: reimplementing the game in Java or via LuaJ (out of scope per CONCEPT.md and [CL] ROADMAP non-goals), bundling the port or its FMOD layer (license undefined, RESEARCH.md unknown 5), and depending on CEF file dialogs for asset intake (unverified, RESEARCH.md unknown 1; the bootstrap-page fallback needs no dialog at all).

## Target stack and tooling

- Minecraft `26.2`, Fabric Loader `0.19.5`, fabric-api `0.161.0+26.2`, Fabric Loom `1.18-SNAPSHOT`; Java toolchain 25 for compilation, mirroring craftlatro's proven build ([CL] gradle.properties, [CL] build.gradle, [CL] src/main/resources/fabric.mod.json).
- Embedding: `de.keksuccino:rinku-fabric:3.0.4-26.2` from `https://keksuccino.github.io/maven/` (RESEARCH.md section 5 candidate 3; the proven pin from [CL] AGENTS.md; 3.0.5 exists in source for a later bump).
- Config: Cloth Config `26.2.155` (hard dependency, LGPLv3) plus Mod Menu `20.0.3` compile-only for the settings entrypoint, exactly as proven in [CL] build.gradle.
- Runtime: Webleste, `MercuryWorkshop/celeste-wasm` branch `threads-v2`, pinned to a specific commit (no tags exist), fetched at runtime into `<instance>/craftleste/game` on the LoveJsDownloader pattern ([CL] `downloader/LoveJsDownloader.java`), with the served directory version-stamped so a runtime change cannot be served from a stale browser cache (the `440f734` lesson, [CL] AGENTS.md Gotchas).
- Game files: Celeste 1.4.0.0, FNA build, player-supplied. Intake accepts either the executable path or the Content directory and treats them as the same thing (CONCEPT.md user preference), with the Content folder resolved from the executable when only the exe is given. Validation checks the port's own contract: `Content/{Dialog,Effects,FMOD,Graphics,Maps,Monocle,Overworld,Tutorials}` (RESEARCH.md gap 2, fs.tsx `hasContent`).
- Serving: in-mod `com.sun.net.httpserver` on localhost with COOP/COEP/CORP headers and `application/wasm` MIME handling, reusing the shape of [CL] `server/LocalHttpServer.java`.
- Conventions carried over from craftlatro: Conventional Commits with one change per commit; single-line version in gradle.properties (`MAJOR.MINOR.PATCH+MCVER`); CHANGELOG.md (Keep a Changelog); release-only CI attaching the jar ([CL] docs/commits-and-releases.md, [CL] .github/workflows/release.yml); instance-local paths under a single paths class ([CL] `CraftlatroPaths.java`); AGENTS.md, ROADMAP.md, docs/, ai-docs/ structure from the first commit.
- Dev-environment rules recorded in AGENTS.md from day one: PowerShell 5.1 chains with `;` (no `&&`), existing files are CRLF, close all streams before replacing files, the AWT clipboard is unavailable (use GLFW), CI is Linux (exec bit on gradlew, committed wrapper jar) ([PM] F3, [CL] AGENTS.md).

## Milestones and steps

### M1: Scaffolding and dependency integration

- Initialize the repo with craftlatro's skeleton: AGENTS.md first (tech stack, verification ladder, Windows notes, boundaries), ROADMAP.md, .gitignore, gradle.properties (`version=0.1.0+26.2`), build.gradle with the Rinku/Cloth/Mod Menu dependencies, fabric.mod.json (`id: craftleste`, GPL-3.0, client-side), LICENSE.
- Verify the Rinku dependency resolves and loads in a dev run; record the flat-GUI API contract in `docs/rinku-api-notes.md`, seeded from [CL] docs/rinku-api-notes.md (it is source-verified and version-matched) rather than re-derived.
- Probe 1 (unknown 1): a page in the real client that reports whether `showDirectoryPicker`, `navigator.storage.getDirectory`, and `navigator.getGamepads` exist and work inside Rinku's browser. This decides the intake mechanism (picker vs bootstrap-only) and the gamepad question early.
- M1 exit: a screen that renders any URL through Rinku, verified by a user-driven `./gradlew runClient` (verification ladder rung 4; an agent cannot open browser screens, [CL] ai-docs/notes/2026-10-12-lovejs-pipeline-proven.md).

### M2: Webleste runtime acquisition, local serving, asset intake

- Implement `WeblesteDownloader`: fetch the pinned `threads-v2` build from its official origin into `<instance>/craftleste/game`, only when missing, with a recorded pin and version in ai-docs.
- Implement the local HTTP server: COOP/COEP/CORP on every response, `application/wasm` MIME, directory-traversal guard, support for the port's split-wasm chunked fetch (numbered chunks appended to the default URI, RESEARCH.md gap 3 / dotnet.ts).
- Implement Celeste intake: accept exe or Content directory, resolve and validate the Content contract, copy into the game directory, record the path in config. No game files enter the repo or jar.
- Implement the bootstrap page (mod-owned): fetch the player's assets from the local server and write them into OPFS at the paths the loader mounts (`/libsdl`), then navigate to the port's index. This is the no-file-dialog path chosen to close unknown 1.
- Probe 2: a standalone harness that runs the OPFS-write logic against a directory of test files before shipping (the standalone-probe pattern, [PM] P1), and a threaded-wasm probe page served through the local server to retire unknown 2.
- M2 exit: the port's frontend boots to its Play button with the player's assets present in OPFS, verified by a user-driven run and a stats/log paste.

### M3: In-game screen surface

- Implement `BrowserScreen` plus the framed window (title bar, drag, resize, fullscreen) mirroring craftlatro's shipped surface, with the game area at the port's fixed 16:9 inside the window (RESEARCH.md unknown 7).
- Input bridging: full mouse and keyboard mapping per the proven table ([CL] docs/rinku-api-notes.md); gamepad support only if Probe 1 answered yes, otherwise documented keyboard-first (RESEARCH.md gap 8).
- Browser lifecycle: create the browser with the two-arg overload only, size through the viewport on the same frame, park when detached (mute, 5 fps, shrink), release on world exit, set the paint rate explicitly and cap it to Minecraft's measured fps ([CL] AGENTS.md Gotchas, lovejs-pipeline-proven note).
- Extend the stats/diagnostics report (`/craftleste stats` analog) with the port's own memory watch (`WatchMemoryUsage`, dotnet.ts) and a paint-rate counter, so runtime claims are measurements, not impressions ([PM] P2).
- M1/M2 unknown 7 resolution: if the game canvas does not follow window resizes, land craftlatro's notice-strip-plus-player-reload behavior rather than pinning the surface (pinning crashed the client natively, [CL] AGENTS.md Gotchas).

### M4: Controls, UX, config, and fidelity

- Commands and keybinds mirroring craftlatro's set (`open`, `pick`, `load`, `warn`, `reload`, `render`, `fps`, `stats`, `save export/import/list/clear`), with the initial keybind as a toggle.
- Cloth Config settings screen (Browser / Craftleste categories) plus the Mod Menu entrypoint; instance-local config, save, and game directories via a single `CraftlestePaths`-style class.
- First-run warning screen covering: the FNA-version and 1.4.0.0 requirement, the memory budget, the "mostly-complete" fidelity caveat, and that Everest is not part of the mod's promise (RESEARCH.md gaps 7 and 10).
- M4 exit: a fresh-install pass driven by the user: first runtime download, Chromium fetch, asset intake, and a full open/play/close cycle.

### M5: Saves, packaging, and guardrails

- Save persistence through the browser profile store, plus portable export/import/clear on the SaveStore pattern: zip the profile storage directories, stage imports, apply before CEF starts, back up before replacing, and awareness that browser storage can evict data ([CL] save/SaveStore.java, RESEARCH.md gap 5).
- Packaging guardrail: verify the built jar contains only mod classes, `fabric.mod.json`, assets, and license; fail the build if a Celeste or Webleste binary is staged; confirm the Chromium runtime remains Rinku's runtime fetch ([CL] ROADMAP.md M5).
- Release conventions: CHANGELOG entry per release, version bumped only in gradle.properties, release-only CI workflow copied from craftlatro's proven workflow, Ask First on every tag and release ([CL] docs/commits-and-releases.md).
- M5 exit: `./gradlew clean build check` green plus a user-verified play session; a 0.x release is cut only on explicit approval.

### Process rules for every milestone (from the craftlatro post-mortem)

- The verification ladder is named in every claim: compile, standalone probe, stats report, user-driven run. A claim of "verified" resting on reading source is not on the ladder ([PM] F2).
- At most one browser-affecting change per user verification run, with a statement of exactly what to check ([PM] F5).
- Local sources before web search: `E:\Coding_Projects\MCSourceCode` (fabric-api/26.2, vanilla-minecraft/26.2, cloth-config, Rinku) for API and version questions ([PM] F1).
- Decisions with the evidence that would settle them are written to `ai-docs/decisions/` before code ([PM] P4); a stale cache or runtime layer is suspected before re-diagnosing a shipped fix ([PM] process, `440f734`).
- Ask First: creating a remote repository, pushing tags or releases, and adding dependencies with non-open runtimes.

## Key risks and unknowns

- Webleste licensing is undefined (no license file) and its audio layer embeds proprietary FMOD object code (RESEARCH.md unknown 5). Mitigation: runtime download from the official origin only, nothing hosted, nothing bundled; an upstream license inquiry is recommended before 1.0.
- The three embedded-browser unknowns: file pickers in OSR, threaded wasm under OSR, and gamepad support (RESEARCH.md unknowns 1 to 3). Each has a named probe in M1/M2 and a fallback (bootstrap page without pickers; keyboard-first input).
- Memory: 600 MB to 1.6 GB for the port alone, plus Minecraft plus the roughly 150 MB Chromium runtime (RESEARCH.md gap 6). Mitigation: measure with the stats report at M3/M4 before promising anything; render scale is the lever.
- Fidelity: the port is self-described as mostly/near complete, so first-run warning plus honest measurement replace the "1-to-1" promise (RESEARCH.md gap 7).
- Version drift: no upstream tags, so a commit pin plus a version-stamped served directory; record the pin in ai-docs and bump deliberately (RESEARCH.md gap 9).
- Save durability: browser storage can be wiped by the browser itself, so export/import is a first-class feature, not an afterthought (RESEARCH.md gap 5).
- Version matrix: stay on MC 26.2 with craftlatro; Rinku publishes per-MC artifacts and NeoForge builds exist in source, but any broadening is a dependency-validation task after v1 ([CL] ROADMAP.md Backlog, Portability).

## Handoff boundary

This plan is complete at RESEARCH.md plus PLAN.md and waits for explicit approval before the folder moves from `concept/` to `planned/`. Implementation (scaffolding, builds, code, PRs) begins only on a separate explicit request. The first implementation gate is M1's exit criterion: a screen rendering any URL through Rinku in a user-driven `runClient`, with the API contract recorded in `docs/rinku-api-notes.md`.
