# CONCEPT.md

## Project Name

modpack-seedfinder

## Objective

A seed-finding tool for Minecraft Java Edition that helps players locate seeds where desirable vanilla structures (e.g., villages, strongholds, ruined portals, ocean monuments) spawn near world spawn, with the goal of being usable against specific NeoForge modpacks. The user's primary use case is pre-generating a world: pick a seed, start a new world with that seed, and land near a village or other wanted structure at spawn.

## Technical Goals

- Find seeds where specified vanilla structures are within a configurable distance of world spawn (default ~500-1000 blocks).
- Run efficiently enough to search a meaningful range of seeds in minutes, not hours or days.
- Reuse existing, well-tested seed-search infrastructure rather than reimplementing structure-placement algorithms from scratch.
- Optionally support a lightweight in-game verification step that confirms structure presence using the modpack's actual world generation.
- Be targetable to a specific modpack (world type, biome source, structure config) so results match that pack's behavior.

## Existing Solutions Survey

A discovery pass was run before drafting. The concept partially overlaps with existing tools; notes below record what exists, how to reuse it, and how modpack-seedfinder differs. GitHub sources are saved for possible code/library reuse.

### Existing Solutions

- **Cubiomes Viewer** (github.com/Cubitect/cubiomes-viewer). C++/Qt desktop app, GPL-3.0. Implements fast mathematical seed search using the cubiomes C library. Supports all vanilla structures, biome conditions, hierarchical criteria, Lua custom filters, and a map viewer. Searches millions of seeds/sec. Does not understand modded structures or custom world types beyond what cubiomes can configure. This is the strongest reuse candidate for the search engine.
- **SeedcrackerX** (github.com/19MisterX98/SeedcrackerX). Fabric mod, MIT. *Cracks* (recovers) the seed of an already-running world by observing structures and brute-forcing. Does not search by criteria. Internally uses kaptainwutax Java libraries. Not directly relevant as a seed-finder, but its underlying libraries are a reuse candidate for a Java-based approach.
- **SeedFinderMod** (github.com/crackedMagnet/SeedFinderMod). Fabric mod, CC0-1.0. Adds a "Seed Finder" world type that searches seeds in-game by criteria, using the cubiomes library for vanilla structure math. Generates worlds in new dimensions for review. Closest in spirit to this concept but Fabric-only and vanilla-structures-only.
- **kaptainwutax Java libraries** (SeedUtils, MCUtils, FeatureUtils). Java, MIT. High-performance simulation of Minecraft structure placement and loot for seed-finding. Used by SeedcrackerX. Reuse candidate for a Java-based standalone tool.

### Uniqueness Verification

- Cubiomes Viewer is a complete, fast, vanilla-structure seed finder but does not target specific modpacks and is C++/Qt (outside the Minecraft Java ecosystem).
- SeedFinderMod is in-game (always mod-aware) but slow (world creation per seed), Fabric-only, and does not support NeoForge modpacks out of the box.
- No existing tool combines fast math-based searching with modpack-aware configuration and NeoForge compatibility.

### Gap Analysis

- No existing standalone tool lets a user search seeds for vanilla structures near spawn tailored to a specific modpack's world type / biome config.
- Reusable building blocks exist: the cubiomes C library (via Cubiomes Viewer) and the kaptainwutax Java libraries, both implementing the same mathematical structure-placement algorithms.

### Foundation Candidates (ranked)

1. **Cubiomes Viewer (fork/extend)** — strongest reuse; already does math-based search; GPL-3.0 license requires share-alike if distributed.
2. **kaptainwutax Java libraries (build new tool)** — Java ecosystem, MIT license; more from-scratch work to build the search UI and config; speeds of tens of thousands of seeds/sec.
3. **SeedFinderMod (extend to NeoForge)** — most modpack-aware; slow per-seed; CC0 license; Fabric-to-NeoForge port is non-trivial.

## Design Decisions

- **Loader/platform**: Standalone desktop tool (Option A or B above). Not an in-game mod for the primary search, due to speed constraints.
- **Target modpacks**: Java Edition NeoForge modpacks.
- **Structure scope**: Vanilla structures only (village, stronghold, ruined portal, ocean monument, pillager outpost, woodland mansion, igloo, swamp hut, desert temple, jungle temple, shipwreck). Modded structures out of scope without in-game verification.
- **Search method**: Mathematical/algorithmic position computation (cubiomes or kaptainwutax libraries), no chunk generation during search.
- **Modpack awareness**: Configure biome source and world type in the tool to match the target modpack, so structure-spawn conditions and spawn point computation reflect the pack.
- **Verification**: Optional lightweight NeoForge in-game mod (Option C) that takes a candidate seed, creates a world, and confirms structure placement. Used only for final candidates, not the full search.

## Scope

### v1 (MVP)

- Standalone seed search tool reusing existing structure-placement math (cubiomes or kaptainwutax libraries).
- Configurable structure criteria: structure type + max distance from spawn.
- Configurable Minecraft version (primary target: 1.26.x, matching sibling concepts).
- Configurable world type / biome source to match a target modpack.
- List and display matching seeds with structure positions and distances.
- Support for at least: village, stronghold, ocean monument, ruined portal.
- Target one specific modpack (TBD) as the reference for development and testing.

### Post-v1 (aspirational, not committed)

- In-game NeoForge verification mod for confirming candidate seeds against actual modpack generation.
- Modded structure support (requires in-game verification or reverse-engineering of mod placement algorithms).
- GUI with map preview of structure positions.
- Import/export integration with Minecraft launcher / modpack profiles.

## Constraints

- Math-based search is fast but assumes vanilla structure-placement algorithms. Modpacks that override structure placement or use heavily customized world types may produce inaccurate results.
- In-game verification is always accurate but slow (minutes to hours per seed).
- v1 targets one specific modpack; generalization to "arbitrary modpacks" is a post-v1 goal.
- Speed tradeoff: C++ (Cubiomes Viewer) is ~100x faster than Java (kaptainwutax); both are vastly faster than in-game.

## Unresolved Design Decisions

- Target modpack: which specific NeoForge modpack to develop and test against (drives biome source / world type config).
- Tool approach: fork/extend Cubiomes Viewer (C++/Qt, GPL-3.0) vs. build a new Java tool (kaptainwutax, MIT)?
- Structure list: beyond village, which structures are priority (stronghold for speedrunning, ocean monument, etc.)?
- Distance thresholds: what counts as "near spawn" (user suggested a few hundred to a few thousand blocks)?
- In-game verification: is a separate NeoForge mod worth building, or is math-based accuracy sufficient for the target modpack?
- Output format: just a seed number to paste into Minecraft, or a richer export (seed + structure coordinates + book)?

## Research & Speed Comparison

| Approach | Seeds/sec (approx) | Pros | Cons |
|---|---|---|---|
| Cubiomes Viewer (C++ math) | Millions | Fastest; complete GUI; reusable | C++/Qt; GPL-3.0; modpack config is manual |
| Java tool (kaptainwutax) | Tens of thousands | Java ecosystem; MIT license; can share code | More from-scratch; slower than C++ |
| In-game NeoForge mod | 5-20 (locate API) | Always mod-accurate; can verify | Too slow for large searches |
| In-game NeoForge mod (chunk gen) | 1-10 / minute | Most accurate | Impractical for searching |

## User Preferences

- Reuse existing tools/libraries rather than reimplementing from scratch.
- Speed is a priority: math-based search, not full chunk generation.
- Target NeoForge modpacks on Java Edition.
- Focus on vanilla structures near spawn (village is the primary example).
- In-game verification is a nice-to-have, not a main priority.
- Scope to a specific modpack for development/testing (to be chosen).

---

This document is a draft proposal. It has not been validated against the user's intent and should be reviewed and confirmed before being treated as finalized or used to guide development.
