# CONCEPT.md

## Project Name

fast-biomes

## Objective

A standalone, ultra-fast Minecraft seed finder that searches for seeds matching biome and structure criteria at speeds exceeding existing tools. It reuses the cubiomes library for mathematical seed-to-world simulation (GPL-3.0), and explores GPU acceleration and low-level optimization to beat cubiomes viewer's performance. The core workflow is: benchmark current speed on a rare criteria, then iteratively optimize (GPU, Rust, or both) while maintaining correctness by validating results against cubiomes.

## Technical Goals

- Search for seeds matching configurable biome and structure criteria (near spawn and globally).
- Benchmark against cubiomes viewer on a known rare criteria, then optimize to surpass it.
- Maintain correctness: all results validated against cubiomes output (or in-game verification as a secondary check).
- Explore GPU acceleration (via JAX or a Rust GPU compute layer) to parallelize seed evaluation.
- Explore Rust as a low-level alternative to C++ for the search hot path.
- Target a latest Minecraft version supported by cubiomes viewer, for baseline comparison.

## Existing Solutions Survey

### Existing Solutions

- **Cubiomes Viewer** (github.com/Cubitect/cubiomes-viewer). C++/Qt desktop app, GPL-3.0. Uses the cubiomes C library for fast mathematical seed search. Supports all vanilla structures, biome conditions, hierarchical criteria, Lua custom filters, and a map viewer. Searches millions of seeds/sec. This is the baseline to beat.
- **kaptainwutax Java libraries** (SeedUtils, MCUtils, FeatureUtils). Java, MIT. High-performance structure and biome simulation. Slower than cubiomes (tens of thousands of seeds/sec) but Java ecosystem.

### Uniqueness Verification

- Cubiomes Viewer is fast and complete but has not been GPU-accelerated or ported to a Rust core.
- No existing tool combines cubiomes-level seed math with GPU compute (JAX or Rust GPU) or a Rust search core for additional speedups over C++.

### Gap Analysis

- Speed headroom exists via GPU parallelization (thousands of seeds evaluated concurrently) or Rust low-level optimizations (zero-cost abstractions, SIMD) beyond cubiomes viewer's C++ implementation.
- The benchmark-and-iterate approach (measure against cubiomes, then optimize while validating correctness) is not packaged as a standalone workflow anywhere.

## Design Decisions

- **Reuse strategy**: Link against the cubiomes C library (GPL-3.0); derivatives inherit GPL-3.0.
- **Platform**: Standalone CLI tool / library (no GUI for v1; no modpack awareness).
- **Scope**: Biome and structure search near spawn and in the world, using math-based (non-generational) evaluation.
- **Minecraft version**: Latest version supported by cubiomes viewer (TBD which exact release at draft time; pinned during implementation).
- **Benchmark**: A known rare criteria that takes a meaningful amount of time under cubiomes viewer (see Benchmark Criteria below).

## Benchmark Criteria

The initial benchmark uses a rare combined criteria that should take a measurable search window under cubiomes viewer:

- Village within 150 blocks of world spawn.
- Ocean monument within 500 blocks of world spawn.
- Badlands biome (modified or standard) within 1500 blocks of world spawn.
- Stronghold within 1000 blocks of world spawn.

This combines structure, biome, and distance constraints, making it rare enough to benchmark meaningfully while remaining a valid Minecraft seed-finding target.

## Scope

### v1 (MVP)

- CLI seed finder reusing the cubiomes C library.
- Configurable criteria: structure + biome types, max distance from spawn.
- Configurable Minecraft version (latest cubiomes-supported release).
- Benchmark harness: measure seeds/sec and time-to-first-match against the benchmark criteria.
- Result validation: confirm every reported seed also satisfies the criteria under cubiomes viewer.
- Output: list of matching seeds with structure/biome positions and distances.

### Post-v1 (aspirational)

- GPU acceleration layer (JAX or Rust GPU compute) for massively parallel seed evaluation.
- Rust core as a low-level alternative or supplement to the cubiomes C library.
- CLI + lightweight TUI or GUI for interactive search.
- Broader biome/structure coverage and arbitrary compound criteria.
- In-game verification mode for the highest-confidence candidates.

## Constraints

- Math-based search is fast but assumes vanilla structure/biome placement algorithms; heavily customized world types may produce inaccurate results.
- GPU acceleration (JAX) requires translating cubiomes' C-based RNG/structure math into a vectorized, differentiable-friendly (or at least parallelizable) form — non-trivial and may require a custom implementation rather than direct cubiomes binding.
- Rust integration with the cubiomes C library requires FFI bindings; full Rust reimplementation of seed math is a larger effort.
- GPL-3.0: any distributed derivative of cubiomes/cubiomes viewer is share-alike.

## Unresolved Design Decisions

- **Technology path**: GPU-first (JAX) vs. low-level (Rust FFI to cubiomes) vs. a staged approach (Rust core first, then GPU)? Which approach to attempt first?
- **Benchmark criteria**: Is the proposed village + ocean monument + badlands + stronghold combo the right "rare seed" benchmark, or should it be simpler (e.g., a pure biome search)?
- **GPU approach**: Would require re-implementing cubioms RNG/structure math in a vectorized form (pure JAX or Rust GPU) rather than binding to the C library. A from-scratch GPU math implementation is explicitly acceptable; cubiomes reuse is preferred where feasible but not mandatory.
- **Output format**: Just a seed number to paste into Minecraft, or a richer export (seed + coordinates + book)?
- **Version target**: Pin to a specific Minecraft version once cubiomes viewer support is confirmed.

## Research & Speed Comparison

| Approach | Seeds/sec (approx) | Pros | Cons |
|---|---|---|---|
| Cubiomes Viewer (C++ math) | Millions | Fastest baseline; complete; reusable | C++/Qt; GPL-3.0; CPU-only |
| Java tool (kaptainwutax) | Tens of thousands | Java ecosystem; MIT | Slower; from-scratch search UI |
| JAX (GPU, vectorized) | Unknown (target: 10x+) | Massive parallelism; modern | Requires re-implementing seed math; binding challenges |
| Rust (FFI to cubiomes) | Millions+ (target: 1.5x+) | Low-level control; SIMD; safe | FFI overhead; partial reimplement |

## User Preferences

- Reuse existing tools/libraries (cubiomes) rather than reimplementing from scratch.
- Start with a benchmark, then iterate on speed while maintaining correctness.
- GPL-3.0 license is acceptable.
- Biome and structure search near spawn and globally.
- No modpack awareness for this concept.
- Rust and JAX are exploratory technology choices to evaluate.

---

This document is a **draft proposal**. It has not been validated against the user's intent and should be reviewed and confirmed before being treated as finalized or used to guide development.

**File location**: `concept/fast-biomes/CONCEPT.md`

Please review and confirm whether this accurately captures your intent. Key points I'd like you to verify or correct:

1. The benchmark criteria (village + ocean monument + badlands + stronghold near spawn) — good, or would you prefer a different rare criteria?
2. The technology exploration order — should we try Rust FFI first, JAX (pure GPU reimplement) first, or something else?
3. Is "biome + structure" search the right scope, or should v1 focus on one dimension?
