# CONCEPT.md

## Project Name

easy-farms

## Objective

Create a Fabric mod for Minecraft 1.26.2 that converts popular, well-known auto-farm designs into single placeable "farm blocks." Each farm block, once placed, generates that farm's outputs over time on the server, letting players obtain farm yields without building the full structure in-world. To keep it survival-balanced and "not free," acquiring a farm block costs the same materials a player would spend building the equivalent farm; the tradeoff is convenience and the removal of build-space/validation constraints. The mod is data-driven: each farm is defined by a schematic plus a material tally plus output rates, so adding a farm (or letting users add their own) is a content task, not a code task. Original farm creators are credited (in-game and in source) for tested rates and designs, but no builds or assets are copied from them.

## Technical Goals

- Fabric mod targeting Minecraft 1.26.2, written in Java with Fabric API (aligned with the sibling procedural-tools concept's established loader/version preference).
- Server-side ticking block entities for each farm block so outputs generate correctly in single-player and multiplayer/LAN. This cannot be a client-only mod.
- Data-driven farm definitions: a farm block is backed by a schematic (structure .nbt / block palette) that provides (a) the build-material ingredient list and (b) the block's model/render. Output rates are declared per farm.
- Generic "Assembly Table" crafting container: the player deposits the schematic-derived materials (arbitrary quantities, item-by-item) and receives the farm block item. No hand-authored recipe per farm is required.
- Extensibility by users: adding a new farm = provide a schematic + material tally + rates (via datapack/schematic). Users can expand the mod without code.
- Farm block acquisition cost equals the build materials of the equivalent farm, derived from the schematic.
- No free outputs: players pay material cost up front; convenience and freed build space are the only rewards.
- In-game and source attribution to original farm video creators/designers for tested rates and designs; no copyrighted builds or assets reused.
- Initial farm types: basic mob farm, iron farm, gold farm.
- Itemizer tool mechanic: a low-cost craftable tool (sticks, iron, gold, ender pearl) that captures a villager on left-click into a "villager item," feeding the iron farm and enabling an automatic villager breeder that outputs "baby villager" items which age up over time.
- Guidebook: start with Patchouli (Fabric 1.26.2 build available) to document mechanics; Ponder is evaluated as a future enhancement (see Research).

## Scope

### v1 (MVP)

- Data-driven farm-block system: schematic-backed definition (material tally + model + rates).
- Assembly Table custom crafting container that consumes schematic-derived materials and outputs a farm block item.
- 3 farm blocks: mob farm, iron farm, gold farm. Each is a placeable block with a server-side ticking block entity that generates the farm's outputs over time at declared rates.
- The "itemizer" tool (craftable) that converts a targeted villager into a single-item stack on left-click.
- Baby-villager-item aging mechanic supporting an automatic villager breeder output (items that age up over time, matching villager maturation timing).
- Server (multiplayer/LAN) support required; client-only behavior explicitly avoided.
- Guidebook delivered via Patchouli.
- Personal use locally first, then public open-source release.

### Post-v1 (aspirational, not committed)

- Additional farm types (crop farms, experience farms, fluid/fish farms, etc.) added purely by dropping in schematics + rate definitions.
- More entity-capture targets beyond villagers.
- Evaluate Ponder as a richer 3D tutorial/guide layer (requires alignment/porting to 1.26.2; see Research).
- User-facing farm-editor or in-game schematic submission.

## Research & Existing Solutions

A discovery pass was run before drafting. The concept is partially redundant with existing mods; the notes below record what exists, how the user requested it be reused (study for code/ideas), and how easy-farms differs. GitHub sources are saved for possible code reuse but are not dependencies.

### Existing Solutions Survey

- Easy Mob Farm (Kaworru). CurseForge 563464 / Modrinth `mod/easy-mob-farm` / GitHub `MarkusBordihn/BOs-Easy-Mob-Farm`. License: code MIT. Loaders: Fabric, Forge, NeoForge, Quilt; **Fabric 1.26.2 supported** (e.g. `easy_mob_farm-fabric-26.2-*.jar`; 5.9M+ CurseForge downloads). Server-friendly ticked "Mob Farm" block. Farm types include a **Mob Farm**, an **Iron Golem Farm**, and **cooper/iron/gold/netherite farms** (changelog: "Added cooper, iron, gold and netherite mob farms"). Economy is capture-card-based (capture a mob into a card, process in a tiered farm; datapack-configurable loot tables), not build-material-cost. Per-user guidance, saved for possible code study only.
- Easy Villagers (Max Henkel / henkelmax). CurseForge 400514 / Modrinth `mod/easy-villagers` / GitHub `henkelmax/easy-villagers`. License: "All Rights Reserved"; NeoForge/Forge only (1.26.2 NeoForge build exists). 67M+ downloads. Provides a Villager-as-item capture, a **Breeder** (outputs baby-villager items), an **Incubator** (ages baby villagers), and an **Iron Farm Block** (places a villager to produce iron over time). This matches the desired itemizer + villager-breeder + aging mechanic almost verbatim.
- Easy Villagers — Fabric port. GitHub `S-BlackDragon/easy-villagers-fabric-port`. License: GPL-3.0. Fabric 1.21.1, "full feature parity" to the NeoForge original (same block set). The only Fabric implementation of the villager mechanic, but not yet on 1.26.2. Saved for possible code study.
- SimpleVillagers (`samolego/SimpleVillagers`). License: LGPL-3.0. Fabric 1.20; small subset (Breeder/Incubator/Converter/Iron Farm block). Saved as a lightweight reference only.
- Compact Iron Farm (Modrinth `project/bPDALpFM`). Single block "spawns 3-5 iron every 30 seconds," costs 3 beds/5 diamonds/3 iron; Minecraft 1.20.1, loader/license under-specified, ~438 downloads. The closest existing analogue to the "pay materials for a single-block farm" idea, but tiny/under-documented.
- FarmingBlock. NeoForge 1.21.1; requires FE/RF energy; crop auto-harvest only. Ruled out (power-gated, crops only, not Fabric).
- Patchouli (Fabric/Quilt). Vazkii; `VazkiiMods/Patchouli`. Custom license. **Fabric 1.26.1.2 build available** (`patchouli-fabric-26.1-94.jar`, beta). Data-driven guidebooks, multiblock visualization, recipe pages, advancement unlocking. Chosen as the v1 guidebook foundation.
- Ponder (Create). `Creators-of-Create/Ponder`. License: MIT. Fabric builds only up to 1.20.1/1.21.1; no 1.26.2 Fabric build. Interactive 3D tutorials. Pinned as a post-v1 enhancement; porting to 1.26.2 is uncertain.
- GuideME / PonderLib. Forge/NeoForge only. Ruled out for a Fabric mod.

### Uniqueness Verification

- Mob/iron/gold single-block farms on Fabric 1.26.2 are already shipped (Easy Mob Farm).
- The villager-capture/breeder/baby-villager-aging/iron-farm mechanics are already shipped (Easy Villagers + Fabric port).
- The combination — schematic-backed blocks whose model and cost derive from a loaded schematic, an Assembly Table that takes arbitrary materials, output rates declared per farm, all data-driven and user-extensible, under a "cost = build materials" economy with a Patchouli guidebook — is not implemented by any single existing mod. That integrative design is the residual differentiator.

### Gap Analysis

- No existing mod ties a farm block's model and material cost directly to a loaded schematic in a generic, user-extensible way; Easy Mob Farm and Easy Villagers author farms by code/config, not schematics.
- No existing mod uses the "acquisition cost = full build-material tally of the farm" economy.
- No existing mod combines the mob/iron/gold farm domain with the villager-capture/villager-breeder domain in one project.
- Easy Mob Farm is capture-card/"fun" oriented, not vanilla-esque; Easy Villagers covers only the villager niche; neither matches the build-material parity or the user-guided expansion model.
- Version skew: Easy Mob Farm supports Fabric 1.26.2; the Easy Villagers Fabric port is only at 1.21.1.

### Foundation Candidates (ranked)

1. Easy Mob Farm (Fabric 1.26.2, MIT) — reference for server-side ticked farm blocks, tiering, redstone/item-buffer patterns. Not a dependency; study only.
2. Easy Villagers Fabric port (GPL-3.0, 1.21.1) — reference for villager-as-item, breeder→baby-villager, incubator→aging, and in-block entity rendering. Study only.
3. Compact Iron Farm — reference for "materials in → single-block iron farm out" economy. Study only.
4. Patchouli (Fabric 26.1.2) — guidebook foundation (dependency acceptable; external, not bundled).
5. Ponder — richer 3D tutorials; not viable until/unless 1.26.2 Fabric support is confirmed or ported.

### Unknown Areas

- Easy Mob Farm's exact gold-farm mechanism (dedicated block vs. zombified-piglin capture-card producing gold) — confirmed only by changelog phrasing.
- Whether the Easy Villagers Fabric port reaches 1.26.2 Fabric (no Fabric 26.2 jar found).
- The exact schematic format and model-rendering approach for block models derived from a structure file on Fabric 1.26.2 (to be resolved in implementation planning).
- Whether Ponder gains 1.26.2 Fabric support upstream.

## Design Decisions

- Loader: Fabric (primary). No NeoForge or cross-loader abstraction.
- Target version: Minecraft 1.26.2 (user-stated; aligned with the sibling procedural-tools concept; Fabric API 0.152.1+26.2/0.161.0+26.2 available).
- Language: Java with Fabric API.
- Farm definition model: each farm = a schematic (block palette + counts → material cost + block model) + a declared output rate. Adding farms is data-driven, not code-driven.
- Cost model: the Assembly Table consumes the schematic-derived material tally; "cost = build materials" is computed from the schematic.
- Guidebook: Patchouli (Fabric 1.26.2 build available) for v1 documentation; Ponder evaluated as a future enhancement.
- Attribution: in-game credits screen plus README/source attribution to original farm creators for design and rates; no builds or assets copied from anyone (low licensing risk). Reference mod sources are consumed for study only, not as dependencies or redistributed assets.
- Entity capture: the "itemizer" serializes a villager into one item stack; treated as a reusable "entity-to-item" capture primitive rather than iron-farm-specific code.

## Constraints

- Must be server-side ticked; cannot be a client-only mod. Outputs must be server-authoritative.
- No free outputs: acquisition cost = build materials (derived from the schematic); convenience and freed space are the only rewards.
- No wholesale copying of copyrighted farm builds or assets; credit only.
- Deterministic, server-authoritative output generation at declared rates.
- Fabric 1.26.2 target (user-stated).
- Reference/research mods are not runtime dependencies; no code or assets are copied from them.

## Unresolved Design Decisions

- Exact material recipe/ingredient list for each farm block: how strictly the full build-material tally maps to the Assembly Table's required inputs (exact count vs. rounded/stack-friendly amounts).
- Output yield rate vs. the "real" farm's rates: whether to match exact rates or scale for balance; how rates are authored per schematic farm.
- Exact UI/mechanism for baby-villager aging (item NBT timer vs. an incubator block that consumes/ages them).
- How mob-farm blocks simulate spawns without requiring a valid spawn volume in the world (simulation vs. spawn-volume check).
- Exact UI/mechanism for the Assembly Table (generic ingredient slots vs. single material-type acceptance; handling of container items).
- Schematic format accepted for farm definitions and block model derivation (structure .nbt vs. custom format).
- Attribution presentation details: in-game screen vs. tooltip vs. README/source only.
- Whether farm blocks are upgradeable/infusable or purely craft-and-place.

## User Preferences

- Fabric over NeoForge / cross-loader for simplicity.
- 1.26.2 target (user-stated; aligned with the sibling procedural-tools concept).
- Server-side only; multiplayer/LAN support required.
- "Not free" survival balance: pay build-material cost (derived from the farm's schematic), gain convenience and build-space savings.
- Data-driven and user-extensible: new farms added by supplying a schematic + material tally + rates, not by writing code.
- Credit original farm creators (videos/designers) for tested rates and designs, without copying their builds.
- Study existing mods for ideas/code (Easy Mob Farm, Easy Villagers + Fabric port, Compact Iron Farm) but do not depend on or copy them.
- Guidebook via Patchouli to start; Ponder only if it can be brought to 1.26.2 (not a v1 blocker).
- Personal use locally first, then public open-source release.
- Low-cost "itemizer" tool (sticks, iron, gold, ender pearl) to capture villager(s) for the iron farm and villager breeder.

---

This document is a draft proposal. It has not been validated against the user's intent and should be reviewed and confirmed before being treated as finalized or used to guide development.
