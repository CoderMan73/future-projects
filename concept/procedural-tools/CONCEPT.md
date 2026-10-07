# CONCEPT.md

## Project Name

procedural-tools

## Objective

Create a Fabric mod for Minecraft that procedurally generates a massive tree of swords. Each sword is derived from a root sword (vanilla base) through a sequence of seeded mutation steps. The mod focuses on v1 as a creative-mode showcase of the tree, plus an in-game visualization tool for exploring sword genealogy.

## Technical Goals

- Generate thousands of swords at build-time using a baked-in, unchangeable seed so players can share and compare their mod versions.
- Use Fabric datagen to create item registrations, model JSON, and lang entries from the generation pass.
- Each sword derives from a parent sword through mutation operations (add, subtract, multiply, divide) applied to a subset of stats: attack damage, attack speed, durability, and knockback.
- Vanilla swords are root nodes. Any generated sword can be used as a mutation base up to 3 times, creating a wide branching tree of life structure.
- Provide a creative-mode tab containing every generated sword.
- Include an in-game genealogy tool that visualizes the evolution tree and shows which swords descended from which parents.

## Scope

### v1 (MVP)

- Sword mutation generation pass (stat mutations only; no special effects, enchants, or abilities).
- Root nodes: all vanilla sword materials (wood, stone, iron, gold, diamond, netherite).
- Mutation limit: each sword can serve as a parent at most 3 times.
- Creative tab with all generated swords.
- In-game genealogy visualization item/tool.
- No server support. No survival crafting. No obtaining swords outside creative mode.

### Post-v1 (aspirational, not committed)

- Additional tools (pickaxes, axes, shovels) using the same system.
- Cool effects, enchantment-like abilities, or custom attributes.
- Survival-friendly generation using world-placed materials or special workstations.
- Curated balancing or rarity tiers.

## Design Decisions

- **Loader**: Fabric (primary). No Architectury or cross-loader abstraction in v1.
- **Target version**: Minecraft 1.26.2 (user-stated target).
- **Language**: Java with Fabric API.
- **Generation timing**: Build-time generation using a Fabric data generator (or equivalent build-time code) that emits registrations and assets before the game starts.
- **Seed**: Single baked-in constant. Unchangeable by players or server operators.
- **Mutation model**: Stat-only mutations in v1. Operations applied: add, subtract, multiply, divide against base stats.
- **Visualization**: In-game genealogy viewer. Exact UI form is a design decision captured during implementation, not pre-committed here.

## Constraints

- Seed must be baked into the mod and unchangeable.
- No server support in v1.
- No survival crafting or world-gen materials in v1.
- Mutation logic should be deterministic given the seed.
- Generated swords are obtainable only via creative mode in v1.

## Unresolved Design Decisions

- Exact UI mechanism for the genealogy visualization.
- Whether swords carry lineage NBT metadata on the item stack.
- Max tree depth versus max total sword count.
- How the datagen pass integrates with Fabric's build pipeline for 1.26.2.

## User Preferences

- Fabric over NeoForge/Fabric cross-loader for simplicity.
- Seed baked in, unchangeable, no server complexity.
- "Tree of life" branching model with per-sword parent limit of 3.
- Stat mutations only for v1, with cooler content planned for later.
- In-game genealogy visualization preferred over external tools.

---

This document is a draft proposal. It has not been validated against the user's intent and should be reviewed and confirmed before being treated as finalized or used to guide development.
