# Playlistdle

## Objective

A web-based variant of the daily music guessing game "Heardle." Instead of a fixed daily song, Playlistdle lets a user supply a Spotify playlist and plays a guessing game from only those tracks. Players hear an incrementally longer clip of a song's intro and must guess the song title and artist before running out of attempts.

## Outcome

A playable web application with two modes:

1. **Daily/Shareable mode**: A player provides a Spotify playlist URL. The app generates a link that, when opened by anyone, always selects the same song for that day from the given playlist. Multiple people can use the same link to play the same daily puzzle.
2. **Endless mode**: The same game flow but draws a random song from the provided playlist each round, allowing continuous play.

## Technical Goals

- Fetch a user-supplied Spotify playlist's track list via the Spotify Web API.
- Use Spotify 30-second preview clips (MP3) as the audio source.
- Present an increasing-length snippet of the song's intro on each guessing attempt (e.g., 0s, 2s, 5s, 10s, 15s, full 30s).
- Allow the player to guess the song title and/or artist, with a limited number of attempts.
- Persist the current playlist and selected song via the shareable URL (query parameter for playlist ID or URL).
- Ensure the same playlist + date combination yields the same daily song consistently.

## Constraints

- No leaderboards or multiplayer features.
- Web-based only (browser playable, no native app).
- Relies on Spotify 30-second preview clips being available for all tracks in the playlist (fallback handling for tracks without previews should be considered).

## Open Decisions

- Tech stack: undecided. Suggestions (neutral options):
  - JavaScript/TypeScript with React or Vite.
  - Plain HTML/CSS/JavaScript.
- Authentication: Spotify Web API can serve public playlist tracks without authentication for many endpoints, but official access requires a client credentials flow. This needs a decision on whether to proxy requests through a backend or use a client-side approach.

## Status

Validated. Concept finalized.