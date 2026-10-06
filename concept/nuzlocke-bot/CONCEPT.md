# Project Name: Nuzlocke Assistant (Emerald)

## Overview

A staged, learning-oriented assistant that guides and plays Pokemon Emerald nuzlocke runs using a rule-based advisor and a search-based battle solver. Early stages reuse existing engines and datasets (no ROM required for battles). A future optional stage can hook into a GBA emulator.

## Goal / Success Outcome

- Stage 1: a windowed app that, given the player's current party and location, recommends which patch of grass to enter (and which encounter to expect/avoid) respecting the first-encounter-only lock rule, and generally what to do next.
- Stage 2: integrate a Showdown-backed battle simulator with an MCTS/move-solver that suggests moves during battles, and can export replays for "watch" mode.
- Stage 3 (optional): optionally drive a Pokemon Emerald GBA emulator for movement and battles.
- Learning intent: this is a learning-first project. If the user asks what a concept is, explain it in detail.

## Technical Goals

1. Reuse first, build second: use Pokemon Showdown locally as the Emerald-era battle engine, and PS data files for species/learnsets/moves/abilities.
2. No ROM/emulator dependency in Stages 1-2.
3. Rule-based reasoning for Stage 1; search/game-tree solver (MCTS) for Stage 2; RL only considered if Stage 2 is solid (not required).
4. Windowed GUI for end-use; CLI/tools allowed during development only.
5. Export Showdown replays for watch mode.

## Stages

### Stage 0 - Research & Foundation
- Inventory existing nuzlocke tooling: nuzlocke-generator, pokus, pokemon-showdown, usage-stat/AI data.
- Pull Gen 3 (Emerald) encounter tables per route/location.
- Set up local Showdown instance or library for battle resolution and replays.
- License check on all reused data/code (PS data files are CC/BSD).

### Stage 1 - Encounter/Route Advisor (Rule-Based + GUI)
- Core logic: per-route optimal patch-of-grass selection given current party slots and desired species, honoring the first-encounter-only locked rule.
- Output: windowed app presenting advice (no CLI interaction for the user; CLI allowed only as internal dev tool).
- Learning focus: nuzlocke rules, encounter tables, constraint reasoning, GUI (e.g. Tauri/Electron, or DearPyGui/flet).

### Stage 2 - Battle Integration (Search/MCTS Solver)
- Tie the advisor to a Showdown-backed battle simulator for Emerald rules.
- Build an MCTS team-evaluation/move-selection solver that suggests moves on demand.
- Export Showdown replays for "watch what happens" mode.
- Learning focus: battle simulation, game-tree search.

### Stage 3 - Emulator Drive (Optional / Far Off)
- Optionally hook an Emerald GBA emulator (e.g. mGBA) for memory reading and input automation.
- Movement may start optional; battles are the desired hook.
- Learning focus: memory reading, input automation, emulator tooling.

## Non-Goals

- LLM-based planning is explicitly out of scope.
- Full autonomous emulator wandering around the map is a far-off, optional goal, not Stage 1.

## Game Scope

- Pokemon Emerald (Gen 3) only for now.

## Reuse / Dependencies (to research)

- Pokemon Showdown: local battle engine + data files.
- Route encounter tables: community datasets / Bulbapedia.
- Existing nuzlocke tooling: for reference only (respect their licenses).
- Windowed UI: e.g. Tauri (Rust), or a light Python GUI, to be chosen in Stage 0.

## Constraints & Restrictions

- Emerald era mechanics and data only.
- First-encounter-only lock rule respected.
- No ROM redistribution or ROM dependency in Stages 1-2.
- License-aware reuse of PS data and third-party tooling.
- User must not be forced to use a CLI during normal use.

## Preferences

- Windowed GUI first.
- CLI/internal tools acceptable during development.
- Watch mode via replay export.
- Learning value of each stage is explicit.
