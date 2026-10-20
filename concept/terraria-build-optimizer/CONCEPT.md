# Concept: Terraria Build Optimizer

## Objective

Build a static, client-side web application that serves as an optimal build optimizer for Terraria. Given a progression gate (corresponding to the stages defined on the [Class Setups wiki guide](https://terraria.wiki.gg/wiki/Guide:Class_setups)) and a target stat, the tool determines the optimal combination of weapons, armor, accessories, buffs, furniture, and niche mechanics to maximize that stat.

The tool also supports a CLI variant for development, testing, and batch optimization runs.

## Goal / Motive / Purpose

Terraria has no formal class system, but the Class Setups wiki guide informally defines progression breakpoints with recommended gear. Existing community tools either focus purely on DPS or do not model niche mechanics (e.g., sitting in a chair to heal, campfire regeneration, furniture-based buffs). This tool fills the gap by modeling the full state space of player stats including survivability, buffs, furniture effects, and undocumented mechanics — all sourced from the decompiled game source code for maximum accuracy.

## Technical Goals

- Extract authoritative item, armor, accessory, weapon, wing, and furniture data from the tModLoader decompiled C# source via ILSpy (or pre-decompiled reference).
- Catalog all player stats relevant to build optimization, including niche ones (life regen from furniture, minecart stats, wing flight time/stats, pet/minion/sentry mechanics).
- Model progression gates matching the wiki's class setup stages: Gearing Up, Pre-Bosses, Pre-Skeletron, Pre-Wall of Flesh, Pre-mechanical bosses, Pre-Plantera, Pre-Golem, Pre-Lunatic Cultist, Pre-Moon Lord, Endgame.
- Model buff sources: potions, food, furniture (campfires, heart lanterns, star in a bottle, etc.), environmental (honey, water, etc.), and class-specific accessories.
- Model niche mechanics: chair-sitting damage reduction/healing, dryad's blessing proximity bonuses, loadout swapping, accessory reforging modifiers, banner bonuses.
- Implement a stat-optimization engine that computes maximum effective stat values given a gate, stat target, and equipped item set.
- Run entirely client-side as a static web app (pure TypeScript/JavaScript) deployable to GitHub Pages.
- Provide a CLI interface for batch computations and development testing.
- Support per-item modifier prefixes (e.g., Legendary, Demonic) in optimization.

## Restrictions and Constraints

- **No backend/server**: Must compile and run as a static site on GitHub Pages. All computation runs in-browser.
- **Data extraction approach**: Use tModLoader's decompiled source (available via ILSpy on `tModLoader.dll`) rather than decompiling raw Terraria binaries. This is cleaner and more maintainable.
- **Manual data refresh**: Between game updates, data must be re-extracted and re-pushed. No automated scheduled scraping.
- **Legal**: Decompiled source use is for data extraction only (reference/fair use); no redistribution of copyrighted game assets.
- **Non-goals**: Real-time simulation of combat, multiplayer interaction modeling, mod support (only vanilla Terraria), map/arena layout optimization beyond buffs.
- **Platform scope**: Desktop/1.4.x version of Terraria. Existing-item data only; no speculative new content.
- **Stat scope**: Stats tied to equipment, buffs, and furniture. Not modeling biome-dependent effects, NPC happiness, or world-generation-dependent bonuses.

## User Preferences

- **Data source**: Decompiled source code (tModLoader/ILSpy) for accuracy over wiki scraping.
- **Interface**: Web app first (GitHub Pages static hosting), CLI fallback for testing.
- **Language/Tooling**: TypeScript for the web app (framework TBD). Python acceptable for the data extraction and CLI tooling.
- **Progression gates**: Wiki Class Setups page structure as the natural gate breakpoints.
- **Update model**: Manual data extraction and push to GitHub per game update.

## Planning-time Questions

- Which decompilation output format is most parseable: ILSpy C# text, or the tModLoader GitHub decompiled repository directly (if available)?
- Does the Terraria/MonoGame source structure expose item stat definitions in a centralized data file (e.g., `ItemData`, `Buffs`, `Accessories`) or are stats scattered across many methods requiring heuristic extraction?
- Which MediaWiki API endpoints (action=query with pageprops/extensions) can reliably fetch the Class Setups progression gate structure and item lists?
- Can static JSON data (items, buffs, gates, modifiers) be generated at build time and bundled with the web app, or does the data size require code-splitting/lazy-loading per gate?
- What optimization algorithm is appropriate: exhaustive search (feasible if gate item pools are small), greedy selection, or constraint-satisfaction / knapsack-style optimization?

## User Questions

- **Stat coverage**: The existing `CompareItemStats` tModLoader mod lists core stats (damage, crit, defense, life regen, knockback, wing stats, tool power, etc.). Does this list cover the niches you want, or are there specific undocumented mechanics (e.g., sitting in a chair, dryad's blessing, minecart speed) that are must-haves? *(Resolved: source-code approach preferred, niche mechanics important — needs confirmation of priority order.)*
- **Modifier modeling**: Do you need all weapon/accessory prefixes modeled (50+ prefixes across accessories and weapons), or can the optimizer use a simplified modifier model (e.g., best-in-slot modifier only)? *(Unresolved.)*
- **Hybrid build support**: The wiki explicitly discusses hybrid classes (e.g., melee + summoner). Should the optimizer support multi-class hybrid builds, or focus on single-class optimal builds with optional summons? *(Unresolved.)*

## Status

This is a draft proposal. Review and confirm before it is treated as finalized.
