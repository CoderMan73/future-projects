# CONCEPT.md

## Project Name

terraria-seedfinder

## Objective

A seed-finding tool for Terraria (PC, Desktop 1.4.x) analogous to Cubitect/cubiomes and cubiomes-viewer: search a range of world seeds against user-defined criteria, list matches with the relevant feature coordinates/context, and preview the generated world. Primary focus is the search engine; a preview/viewer is wanted in the same tool but secondary.

## Target Platform

- PC / Desktop Edition only.
- Target version: latest stable (1.4.5.x as of research; the generation pass list was sourced from `Terraria.WorldGen.cs` `GenerateWorld()` at 1.4.4.9 per the wiki, with the note that the current desktop version is 1.4.5.8).
- No console, mobile, or old-gen console support.

## Technical Goals

- Search a configurable range of Terraria world seeds and return matches where seed-deterministic world-gen features satisfy user criteria.
- Criteria expressed relative to world spawn (horizontal center of the world) and/or in absolute world coordinates, e.g. structures/loot within a radius of spawn.
- List and report matching seeds with the relevant feature positions, distances, and a compact description.
- Optionally preview a matched seed's generated world (viewer), reusing the same world-gen reimplementation.
- Reuse existing reference material rather than reimplementing generation blind: primarily tModLoader's reconstructed Terraria source (see Research & Survey).

## Seed Model (from research)

- Terraria world seed format: `size.difficulty.evil.special.identifier`.
  - `size`: 1 (Small) / 2 (Medium) / 3 (Large).
  - `difficulty`: 1 (Classic) / 2 (Expert) / 3 (Master) / 4 (Journey).
  - `evil`: 1 (Corruption) / 2 (Crimson).
  - `special`: additive bitmask of special world seeds (0..511).
  - `identifier`: 32-bit integer (0..2147483647); if the user-entered identifier is non-numeric it is converted via CRC-32. The identifier is the actual random seed.
- The full parameter set (size, difficulty, evil, special seeds) must be fixed per search so results reproduce identically in-game.

## Research & Survey

A discovery pass was run against the Terraria wiki (World_generation, World_Seed) and the public tooling landscape.

### Existing Solutions

- **terraria.tools/seed-map** — A Terraria seed *viewer* for 1.4.5: enter any seed and explore the world it generates. "The real worldgen runs in your browser." This proves Terraria's world generation has been reimplemented/reconstructed and is runnable client-side, but it is a per-seed preview, not a multi-seed search-by-criteria tool. Strongest reference for the generation model; not a competitor as a *finder*.
- **tModLoader (open source)** — Provides access to reconstructed Terraria source (`Terraria.WorldGen.cs`), the primary reference for reverse-engineering the world-gen passes.
- **TerraMap / TEdit / MoreTerra** — World *viewers/editors* that open an already-generated `.wld` file. Not seed-based and not search tools.
- **Cubitect/cubiomes & cubiomes-viewer** — The Minecraft reference the user cited. C library + viewer doing fast math-based seed search.

### Uniqueness Verification

- No existing tool searches Terraria seeds by criteria across many seeds. The only client-side worldgen reimplementation found (terraria.tools) is a preview viewer, not a finder.

## Technical Direction

- **Reverse-engineering approach**: Reimplement the seed-deterministic generation passes of `Terraria.WorldGen.cs` `GenerateWorld()` in a host language, using tModLoader's reconstructed source as the reference. (This mirrors how cubiomes reimplements Minecraft generation, and how terraria.tools reimplements Terraria generation for its viewer.)
- **Search model**: Simulate Terraria's generation per seed. Unlike Minecraft (region-grid closed-form for structure positions), Terraria generation is a sequential pass sequence driven by .NET `System.Random`; most features are not reducible to a closed-form "position from seed" formula. Therefore the search engine runs a fast reimplementation of the relevant passes for each candidate seed and then tests criteria.
- **Scope of reimplementation**: Start by replicating only the passes needed for the targeted search features; extend passes as new criteria are added.

## Searchable Features (proposed for v1, pending wiki confirmation)

Seed-deterministic features available for criteria. Final v1 list to be confirmed by the user after reviewing the wiki.

- Dungeon entrance side (Left/Right) and approximate dungeon mouth position.
- Evil biome choice (Corruption vs Crimson) and chasm layout.
- Floating islands: count, horizontal positions, and loot tier (Sky Crate / Royal/Demonite/Beetle/Flamelash types, etc.).
- Pyramid presence and position (rare Pre-Hardmode structure).
- Living Trees (giant trees) presence/count/position.
- Jungle Temple (Lihzahrd Temple) approximate position.
- Underground Mushroom biome presence.
- Underground Houses presence/count.
- Demon/Crimson Altars placement and count.
- Hellforge count and position in The Underworld.
- Pre-Hardmode ore pair types (Copper/Tin, Iron/Lead, Silver/Tungsten, Gold/Platinum) and distribution.

## Scope

### v1 (MVP)

- World-gen reimplementation covering the passes needed for a small starter set of search features.
- Criteria: dungeon side, floating islands (presence/position/loot), evil biome/chasms, pyramids, living trees.
- Search over a configurable seed range; list matches with positions/distances from spawn.
- Command-line interface for running searches and printing results.
- Target one fixed parameter set (e.g., Large / Expert / Corruption, no special seeds) as the reference for development and testing.

### Post-v1 (not committed)

- Viewer/preview component (reuses the world-gen reimplementation to render a seed's map; like cubiomes-viewer).
- Additional criteria from the proposed feature list above.
- Configuration of size/difficulty/evil/special-seeds per search.
- GUI with an interactive map preview of structure positions.

## Constraints

- **Performance model**: Per-seed simulation of generation passes, not cubiomes-style closed-form math. Expected throughput is on the order of hundreds to low thousands of seeds/sec in a straight port, rather than cubiomes' millions/sec. Optimizing the hot paths of the reimplementation is a core part of the work.
- **32-bit identifier space**: The random seed is a signed 32-bit integer; CRC-32 is applied to non-numeric identifiers. Exhaustive search of 2^32 space is infeasible; tools must allow ranges, sampling, and/or bit-trick reduction where applicable.
- **Version coupling**: World-gen passes change between Terraria versions. The reimplementation must be version-keyed; v1 targets one specific version (above).
- **Legal**: Terraria's world generation is proprietary. The tool reimplements behavior from the open-source tModLoader reference and public wiki documentation; it will not include or redistribute Terraria source code, and will not enable playing Terraria without owning a copy.
- **False positives**: Replicating only the relevant passes means some criteria may need full-pass fidelity or post-checks; accuracy of each feature is bounded by how completely its pass is replicated.

## Unresolved Decisions

- **Host language / runtime**: Not chosen. C# is a natural fit (matches the reference source) but other languages may serve the per-seed throughput goal better.
- **UI toolkit / viewer**: Not chosen; deferred to post-v1.
- **Final v1 search feature set**: Proposed list above is a starting point; the user will confirm which features to include after reviewing the wiki.
- **Parameter flexibility**: v1 fixes one parameter set; when to generalize size/evil/special-seeds is undecided.
- **Seed search strategy**: Range enumeration vs. random sampling vs. bit-level reduction; depends on throughput measured during the v1 reimplementation.

## User Preferences (endorsed)

- Reverse-engineering via tModLoader tooling/reference, reimplementing generation rather than running the full game.
- Search engine is the priority; viewer is wanted but secondary.
- PC latest stable version; no other platforms.
- Do not publish Terraria source; do not make the game playable without owning a copy.
- Reuse existing reimplementation/reference material rather than reimplementing generation blind.

---

This document is a draft proposal. It has not been validated against the user's intent and should be reviewed and confirmed before being treated as finalized or used to guide development.

## Next Review Inputs (needed from the user)

1. Confirm the final v1 feature set from the "Searchable Features (proposed)" list (the user indicated they want to check the wiki first).
2. Confirm or defer the "host language / runtime" decision (currently: none chosen).
3. Confirm the fixed v1 parameter set (size/difficulty/evil/special seeds) to develop and test against.
