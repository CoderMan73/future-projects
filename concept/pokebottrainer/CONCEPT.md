# Project Name: PokeBotTrainer

## Status

Draft proposal. Confirm with user before treating as validated.

## Overview

An extension of the existing StockFeebas codebase (Phases 0–5 complete) that transforms it into a modular, plug-and-play framework for training specialized Pokémon battle AI agents on user-defined, arbitrary fixed teams. Rather than a single generalist agent, the project focuses on fast per-team specialization: given a Showdown-formatted team and a selected battle format, the system produces a battle-ready agent tailored to that specific roster.

The intended use case is as a library and CLI tool for developers, including as a fitness evaluator inside evolutionary team-optimization loops where many candidate teams are generated and the bot's win rate decides whether to keep each mutation.

## Goal / Success Outcome

- Given a Showdown-formatted team and a user-selected battle format, automatically produce a battle agent that plays that team effectively.
- Training from a generalist base model to team-specific proficiency is fast enough to be used inside an evolutionary optimization loop (many candidate teams evaluated per hour).
- The framework is usable as a Python library/CLI by other developers, with clear documentation and minimal setup.
- Architecture is modular so that future throughput improvements (poke-engine integration, process parallelism, custom fast environments) can be swapped in without rewriting the training or team-loading logic.

## Technical Goals

1. Team-specific training pipeline: ingest a fixed team definition, instantiate a team-locked environment, and run a focused training or fine-tuning loop.
2. Generalist-to-specialist workflow: support loading a pretrained generalist policy and fast fine-tuning on the target team, with a path for full scratch training if no base model is available.
3. Modular simulation backend: define a simulation interface so the current Showdown-based backend can be replaced later by a faster engine (poke-engine, custom fast env, etc.) without changing the training or team code.
4. Format-aware configuration: team and format inputs are explicit and validated before training begins.
5. Developer-friendly interface: Python package with a simple CLI entry point and programmatic API for loading teams, configuring formats, launching training, and exporting a trained agent.
6. Early consolidation step: before any feature work begins, identify and migrate or replicate every needed component from the existing StockFeebas codebase into this project's repository, so that StockFeebas can be retired and is not required as a live dependency.

## Implementation Preference

- Extend the existing StockFeebas repository and Python codebase rather than starting from scratch.
- Keep the current Showdown/poke-env backend as the default for v1, but abstract simulation access behind an interface to enable later acceleration.
- Python-first for accessibility; performance-sensitive acceleration backends may be Rust or JAX-based in future versions.

## Open Questions / Undecided

- **Training speed target**: What is the acceptable wall-clock time to specialize on one team? Minutes per team? Tens of minutes? This affects whether fine-tuning from a generalist base is required, and how aggressive the acceleration roadmap must be.
- **Generalist base model**: Does the project ship a pretrained generalist model, or does the user bring their own? If shipped, what format coverage should it cover (all National Dex, specific formats)?
- **Evaluation protocol**: Should training stop based on a win-rate threshold, a fixed number of battles, or convergence metrics? Who defines the opponent pool during training (fixed baselines, league of past checkpoints, random teams)?
- **Acceleration backend**: When switching away from Showdown, should the first target be poke-engine integration, process parallelism, or a custom incremental fast environment?

## Constraints & Restrictions

- Each training run targets exactly one fixed team of 6 Pokémon. Team changes require a new training run; dynamic roster management during training is out of scope.
- Teams are provided in Showdown team format.
- The battle format is selected explicitly by the user per training run.
- v1 does not require custom engine acceleration; throughput improvements are treated as replaceable backend modules.
- The framework is intended for local/research use, not live server deployment.

## Non-Goals

- Not a standalone battle engine or battle mechanic reimplementation.
- Not a generalist-only agent that never specializes.
- Not a GUI or web tool for casual users; developer CLI/library only.
- Not a team builder or team-analysis dashboard (team content is provided as input, not generated).

## Reuse / Dependencies

- **StockFeebas**: existing environment wrapper, observation encoder, action space, PPO trainer, league-based self-play, checkpoint manager, and baseline RuleBot are the starting implementation surface.
- **Pokémon Showdown / poke-env**: default simulation backend for v1.
- **poke-engine**: candidate future backend for accelerated training.
- **Showdown team format**: standard team definition input.
