# Plan: mkw-race-tracker

## Objective

A single-file command-line tool that, while a Retro Rewind race is running, attaches to the `RetroRewind.exe` process, reads the race state through the community-documented `Racedata`/`Raceinfo`/`MenuData` pointer chain, classifies each event as single-player offline, and appends one JSON Line per completed single-player race to a log file — replacing manual note-taking during Grand Prix and Time Trial sessions.

## Goal / Motive / Purpose

Eliminate manual note-taking by automatically capturing the data a player actually cares about for a single-player race record (which track, grid and finish position, each finish time, who drove which character/vehicle, and the mode) — nothing beyond that. The purpose is strictly single-player logging; every deferred item (VR delta, lobby occupancy, private-vs-worldwide) stays out of this plan to avoid scope creep.

## Chosen path

External process memory reading over Windows `ReadProcessMemory` (and `/proc/<pid>/mem` on Linux for the optional reader) using PyMemoryEditor, with struct offsets from `mkw-structures` (RESEARCH.md section 5 item 1) and the dereference + big-endian byte-swap pattern from the Dolphin autosplitter (RESEARCH.md section 5 item 2). This path is chosen over the network-capture tooling (mkw-ana, RESEARCH.md section 2 items 2-3) and the in-game Discord Rich Presence because only an external reader satisfies the "no game modification, logs the running RetroRewind.exe process" requirement. The single unresolved precondition is item 6.1 of RESEARCH.md: the Wiicompiled host address mapping must be confirmed empirically before the address math is locked.

## Target stack and tooling

- Language: Python 3.12 (stdlib only for logging/CLI; struct for PowerPC big-endian decoding).
- Memory-access layer: PyMemoryEditor v3.4.0 (MIT), `OpenProcess(name="RetroRewind.exe")`; use `read_process_memory_into` for the tight poll loop and typed/raw reads for the pointer chain, **opened with read-only access** (RESEARCH.md section 5 item 3). This tool is strictly a reader: it never writes to, suspends, injects into, or otherwise modifies `RetroRewind.exe`. The only writes the program performs are to its own log directory. PyMemoryEditor's `permission` is pinned to `PROCESS_VM_READ` on Windows; any write API is intentionally out of scope and must not be wired in. Fallback if a ctypes-native dependency is disallowed: pymem (Windows-only `ReadProcessMemory` ctypes wrapper, RESEARCH.md section 5 item 4), also read-only.
- Data model: offsets/types taken verbatim from `mkw-structures` `racedata.h`/`raceinfo.h`, validated against the working autosplitter's dereference (RESEARCH.md section 4 item "data is well-documented"; section 5 items 1-2). Track name resolution from the community "List of Identifiers" (tockdom.com), not bundled here.
- Output: JSON Lines appended to `./logs/<timestamp>.jsonl` after each logged race.
- Platform: Windows 10/11 64-bit primary (the `RetroRewind.exe` target). Linux reader optional and not required for MVP (CONCEPT.md scope boundary).

## Milestones and steps

Milestone 1 — Process discovery and attachment.
- Poll the process table for `RetroRewind.exe` by name (PyMemoryEditor `OpenProcess(name=...)`) and open it **read-only** (`PROCESS_VM_READ`); resolve PID/handle; retry until the process appears; close the handle and keep polling when it disappears.
- Never call `WriteProcessMemory`, `WriteProcessMemory`-equivalent, or any in-process injection. The program has no write path into `RetroRewind.exe`; all writes go only to the local log directory.
- Fail fast and message clearly if access is denied (permissions) rather than crash.

Milestone 2 — Address resolution (the unresolved precondition: RESEARCH.md section 6 item 1).
- Inspect the live `RetroRewind.exe` virtual memory layout for a readable region whose contents contain a known MEM1 marker (e.g., the game-code header at the 0x80000000 origin, `0x52535045` style pattern) to determine whether `0x809BD728` is directly dereferenceable or whether a host base offset must be computed once and reused.
- Implement the resolution once; do NOT hardcode a Dolphin-style literal mapping until validated.
- Document the chosen scheme (literal vs. base+offset) in code headers so later NTSC/region work is a configuration change, not a rewrite.

Milestone 3 — Pointer chain and struct reads.
- Read the three static instance pointers (`Racedata::sInstance`, `Raceinfo::sInstance`, `MenuData::sInstance`) at their Wii addresses (RESEARCH.md section 6 item 4 notes the MenuData offset is community-sourced, not in `mkw-structures`).
- Dereference `Racedata -> main.scenarios[0] -> settings` for `courseId`, `gameMode`, `localPlayerCount`; `RacedataScenario.players[12]` for each player's `characterId`/`vehicleId`.
- Dereference `Raceinfo -> players[]` (pointer array) -> each `RaceinfoPlayer` for `position`; follow `raceFinishTime` (Timer*) -> `minutes(seconds u8; milliseconds` (RESEARCH.md section 4 item 4).
- Byte-swap every multi-byte value (PowerPC big-endian to x86 little-endian) per the autosplitter pattern.

Milestone 4 — Race boundary detection and single-player classification.
- Use `MenuData`/scene state to distinguish menu vs. race and to detect start and end boundaries (RESEARCH.md section 6 item 4).
- Classify via `gameMode` + `localPlayerCount`: log only Grand Prix (mode 0) and Time Trial (mode 2) as single-player; treat VS/battle/online as non-single-player and do not log (CONCEPT.md section "Single-player vs. multi-player detection").

Milestone 5 — Result assembly and JSON Lines logging.
- On race end for a single-player race, assemble the record (timestamp, track ID + resolved name, mode, ordered player list with finish position, character, vehicle, finish time) and append one JSON Line **to the local log file only**.
- The program performs exactly two kinds of I/O against `RetroRewind.exe`: `ReadProcessMemory` (to read the pointer chain and struct fields), and `CloseHandle` on shutdown. No bytes are written to the target process under any code path.
- Fail gracefully: if expected addresses do not yield valid-looking data (e.g., out-of-range courseId/gameMode, null player pointers), skip the record and keep polling rather than crashing (CONCEPT.md "Known constraints").
- Validate output is well-formed JSON per line.

Milestone 6 — Region flag (configuration, not a new feature).
- PAL addresses are the baseline (RESEARCH.md section 6 item 3). Expose a `--region pal|ntscu|ntscj|ntsc-k` flag wired to the community offset alternatives; default PAL. NTSC offsets are NOT implemented until validated against mkwiiki address tables (out of scope for the single-player MVP).

## Key risks and unknowns

- Address-resolution mechanism for Wiicompiled is unverified (RESEARCH.md section 6 item 1). If `0x809BD738` is not directly readable in the host process, milestone 2 must discover a host base offset; the plan is ordered so this is resolved before any dereference work.
- Retro Rewind closed-source anti-cheat may treat external memory reads as suspicious (RESEARCH.md section 6 item 2); the read-only posture (no writes, no injection, `PROCESS_VM_READ` only) is the conservative stance, but impact and mitigation are unknown until tested against a live build.
- `MenuData`/`raceFinishTime` pointer validity windows are community-attested, not independently verified (RESEARCH.md section 6 item 4); race-end boundary detection may need tuning against the autosplitter's scene handling.
- NTSC offsets and the character/vehicle/track-name lookup tables are not validated here (RESEARCH.md section 6 item 3); deferred to a region-configuration task, not built into MVP.

## Handoff boundary

This plan is complete. Implementation (file scaffolding, dependency install, building milestones 1-6, testing against a live `RetroRewind.exe`) begins only on explicit approval. After implementation, a local code review is suggested as a separate step.
