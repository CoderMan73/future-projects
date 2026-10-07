# Project Name: Team Perfection Tool (Emerald)

## Status

Draft proposal. Confirm with user before treating as validated.

## Overview

An evolutionary optimizer that "perfects" a Pokémon Emerald team by starting from an initial team, evaluating it against an enemy team (or pool of enemy teams), and iteratively mutating team composition and individual Pokémon builds to maximize win-rate. When one or more candidate teams reach 100% win-rate, further improvement continues via secondary metrics (e.g., total HP lost, KOs taken on your side, battles won in fewer turns).

This is a re-scoped and Emerald-specialized version of the broader `02-team-perfection-tool` idea (which targeted Gen 6+ competitive play). It shares the workspace with `03-enhanced-agent-intelligence` (a separate battle-AI project) and `concept/nuzlocke-bot` (which reuses Pokémon Showdown as an Emerald-era battle engine).

## Goal / Success Outcome

- Given an initial team and a target enemy (fixed team, or a pool of enemy teams with the same constraints), autonomously evolve the best possible Emerald team through mutation.
- Output: the highest-fitness team found, its win-rate statistics, and a summary of which mutations led to improvements.
- Two team pools supported:
  1. **In-game obtainable**: Pokémon, moves, items, abilities, natures, and IVs/EVs that are legitimately obtainable at a chosen point/progress in Pokémon Emerald (e.g., pre-badge-1, post-Hoenn, pre-E4, post-game).
  2. **Competitive viable**: a curated pool of competitively relevant Emerald Pokémon/builds (for optimization without story-legality constraints).
- Enemy modes:
  1. **Fixed**: a single specified enemy team.
  2. **Pool**: multiple enemy teams drawn from the same constraint model as the evolving team.

## Technical Goals

1. Mutation-based evolutionary optimizer. Starting from a seed team, generate offspring by mutating one or more dimensions, evaluate fitness, and select toward higher win-rate.
2. Mutation scope covers all team dimensions: Pokémon species, moves (4 per Pokémon), ability, held item, nature, IVs (0-31 per stat), and EVs (0-510 total, 252 per stat cap in Gen 3).
3. Fitness function:
   - Primary metric: win-rate over a fixed number of battles (100 per evaluation, as discussed).
   - Secondary metrics (applied only when win-rate plateaus at 100%, or as tiebreakers): total HP lost across the team, number of own-side KOs, number of turns elapsed.
4. Battle resolution reuses an existing engine rather than reimplementing Emerald mechanics. The leading option is Pokémon Showdown configured for Gen 3 / Emerald rules (consistent with `concept/nuzlocke-bot`), accessed via a Rust Showdown host or the `pokemon-showdown-rs` crate.
5. The player's own team is controlled by a bot during battle (not a human). The default uses Showdown's built-in bot (e.g., RuleBot or a simple heuristic strategy). The `03-enhanced-agent-intelligence` battle AI is explicitly out of scope for this project to keep concerns separate; it may optionally be swapped in later as an alternate controller.

## Implementation Preference

- Lean Rust for performance (running 100 battles x many candidates x many generations) and consistency with the sibling projects' Rust direction (`concept/nuzlocke-bot` and `03-enhanced-agent-intelligence` both lean Rust).
- Showdown is integrated as the Emerald-era battle engine/data source.
- Python is rejected in favor of Rust, accepting slower initial prototyping.

## Open Questions / Undecided

- **Enemy pool generation**: For pool mode, enemy teams are randomly generated within the same constraint model as the evolving team. Whether this is fully random or drawn from a curated set is not yet decided.
- **Evolutionary strategy specifics**: Population size, mutation rates, selection method (e.g., (1+\lambda), (\mu,\lambda)), and elitism policy are not yet decided.

## Constraints & Restrictions

- Pokémon Emerald (Generation 3) mechanics and data only.
- No ROM redistribution or ROM dependency. Battles are resolved through an existing battle engine/data source (Pokémon Showdown or a Rust port), not an emulator.
- In-game obtainable mode must respect what is actually accessible at the chosen progression point in Emerald (story flags, TM availability, etc.).
- Reuse existing data and engines first; do not reimplement damage calculation or battle mechanics from scratch.
- The `03-enhanced-agent-intelligence` battle AI is out of scope; use Showdown's default bot for the player's side.

## Reuse / Dependencies (to research)

- **Pokémon Showdown**: as the Emerald-era battle engine and data source (species, moves, abilities, items, learnsets). Consistent with `concept/nuzlocke-bot`.
- **Gen 3 data files**: from Showdown's `data/` or PokeAPI, filtered to Gen 3 legality.
- **Rust Showdown integration**: `pokemon-showdown-rs` crate or a Rust host running Showdown's JS engine, to be confirmed during setup.
- **In-game obtainability data**: community datasets / Bulbapedia for Emerald progression-gated Pokémon, TMs, and items.

## Non-Goals

- Not a general-purpose competitive team builder for later generations (stay Emerald-focused).
- Not a ROM hacking or emulator automation tool.
- Does not build or integrate the `03-enhanced-agent-intelligence` battle AI; that project remains separate. The player-side bot is a Showdown default bot (e.g., RuleBot).
- LLM-based planning is out of scope.
