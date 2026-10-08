# mkw-race-tracker

**Status**: Draft proposal. This is a working document describing the proposed scope,
not a finalized specification. Feedback needed before any development begins.

## Objective

A real-time telemetry extractor that reads Mario Kart Wii race data from a running
Retro Rewind process on the user's PC and logs it automatically, eliminating manual
note-taking during single-player sessions.

The tool reads game memory externally (from a separate process). It does not modify the
game, inject code, or require rebuilding Retro Rewind. It targets the Retro Rewind
profile of Wiicompiled exclusively.

## What it tracks (core)

For each race, the tool captures:

1. **Track identification** — which course is being raced, read from
   `Racedata` (course ID in the settings struct). Verified against the known track
   order tables already used by the community.

2. **Race placements** — starting position (grid position before the race begins)
   and final finishing position (1st, 2nd, etc.), read from `Raceinfo` player data
   and the position ordering array.

3. **Individual finishing times** — each player's finish time (minutes, seconds,
   milliseconds), read from the `Timer` struct pointed to by each `RaceinfoPlayer`.

4. **Race type classification** — explicitly differentiating single-player races
   from multi-player/non-single-player races. This is read from the `gameMode` field
   in `RacedataSettings` and cross-checked against `localPlayerCount`. Modes that are
   strictly single-player (Grand Prix, Time Trial) log as "single-player"; VS, battle,
   and online modes log as "multi-player/non-single-player". This distinction gates
   which races are logged and how player data is interpreted.

These data points cover the core use case: logging per-race results for record-keeping,
personal stats, or content creation (showcase videos).

## What it does NOT track (future goals)

The following are explicitly deferred to a future phase:

1. **VR (Virtual Rating) delta** — requires tracking player rating pre- and post-race,
   detecting race-completion boundaries, and validating against Retro Rewind's Pulsar
   engine which modifies the rating calculation path.

2. **Lobby occupancy** — the number of players in an online lobby. The offline player
   count is trivially available from Racedata, but the online lobby state depends on
   Retro Rewind's custom network layer (RWFC server) which patches the vanilla network
   structures. This needs Retro Rewind-specific address validation.

3. **Private lobby vs. worldwide race differentiation** (long-term goal) — a feature
   that distinguishes between private friend lobbies and worldwide public races during
   online play. Private lobbies correspond to Friend Room game modes (mode 7: lobby,
   mode 8: private VS, mode 11: private battle), while worldwide public races correspond
   to public online modes (mode 9: public VS, mode 10: public battle). This requires
   address validation against Retro Rewind's network layer and is deferred.

## Target platform

Strictly the **Retro Rewind** profile of Wiicompiled. The tool detects and attaches to
the `RetroRewind.exe` process. The stock Wiicompiled build and non-Retro Rewind
configurations are out of scope.

Rationale: Retro Rewind is the actively developed mod distribution for MKWii on
Wiicompiled. It patches the game via Kamek/Pulsar, which is compiled statically into
the native binary. The base game's static addresses (Racedata, Raceinfo, MenuData) are
expected to be preserved, but the network layer is modified by Pulsar. Single-player
modes are unaffected by the network layer changes, making them reliable targets.

## Technical approach

### Memory access method

External process memory reading via the host OS API (`ReadProcessMemory` on Windows,
`/proc/<pid>/mem` on Linux). No code injection, no DLL, no in-game hooks.

### The address resolution trick

Wiicompiled maps the entire 4 GB Wii address space into the host process at a
fixed, deterministic base address. Every Wii address `0x80XXXXXX` corresponds to
exactly one host address with no ASLR. This means:

- The static pointers documented by the MKWii community (Racedata, Raceinfo) work
  directly — the addresses are the same as in Dolphin and on real hardware.
- No runtime base-resolution step (unlike Dolphin, which randomizes its RAM base).
- The tool simply adds a fixed offset to any known Wii address.

### Known addresses (PAL region; NTSC variants exist)

These are documented and proven by the community's gecko codes and the LiveSplit
autosplitter. The tool reads them as the foundation of the telemetry chain:

- `0x809BD728` — `Racedata::sInstance` (static pointer to race config)
- `0x809BD730` — `Raceinfo::sInstance` (static pointer to live race state)
- `0x809C1E38` — `MenuData::sInstance` / scene manager (detects menu vs. race)

### Data extraction chain

1. Open the RetroRewind process by name (`RetroRewind.exe`).
2. Read the 32-bit pointer at the flat address of each static instance.
3. Dereference through the known struct offsets to reach:
   - `Racedata` -> `main.scenarios[0].settings.courseId` (track)
   - `Racedata` -> `main.scenarios[0].settings.gameMode` (race type: single-player vs. multi-player)
   - `Racedata` -> `main.scenarios[0]` -> `localPlayerCount` (cross-check for single-player detection)
   - `Raceinfo` -> `players[]` array -> each player's `position` and `raceFinishTime`
   - `MenuData` / scene state -> detect race start and race end boundaries
4. Byte-swap all multi-byte values (PowerPC big-endian to x86 little-endian).
5. Log the captured data (JSON Lines format recommended).

### Single-player vs. multi-player detection

The `gameMode` field in `RacedataSettings` (offset 0x08 within the settings struct)
determines the race classification:

- Mode 0 (Grand Prix), Mode 2 (Time Trial): single-player campaigns
- Mode 1 (VS Race), Mode 3 (Battle), Mode 8 (Private VS): multi-player / local multiplayer
- Mode 7 (Friend Room Lobby), Mode 9 (Public VS), Mode 10 (Public Battle), Mode 11 (Private Battle): online
- Mode 12 (Award), Mode 13 (Credits): results/end sequences

The tool cross-references `gameMode` with `localPlayerCount` to confirm the
classification. Only single-player races (Grand Prix, Time Trial) are logged in the
core phase.

## Scope boundaries

- **Retro Rewind single-player only.** The tool targets Retro Rewind's Wiicompiled
  build and reads only offline single-player race data. Online race telemetry is a
  future goal.
- **No game file access.** No reading of ISO/GCM/RPZ images. The tool reads only
  the running process's memory.
- **No asset generation.** This is purely telemetry/logging. No connection to fal,
  ComfyUI, or any asset pipeline.
- **Windows primary.** The Retro Rewind process is a Windows native executable.
  Linux/macOS support for the reader tool is optional but not required for MVP.

## Output format

Log file (JSON Lines) written after each single-player race, containing:
- timestamp
- track ID and track name (resolved via known lookup table)
- race mode (Grand Prix, Time Trial, etc.)
- list of players with their finishing position, character, vehicle
- finishing times (parsed from Timer struct)

## Known constraints

- Addresses are PAL region. NTSC-U/J/K variants have known alternative offsets
  (documented by the community) and should be selected via a region flag.
- Retro Rewind memory layout is based on community documentation but has not been
  independently verified. The tool should fail gracefully if expected addresses don't
  contain valid-looking data.
- Process must be running. The tool can poll for the `RetroRewind.exe` process to appear.
